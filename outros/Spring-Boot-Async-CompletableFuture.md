# Spring Boot Web — Async, CompletableFuture e Transações

> Stack de referência: Java 21+, Spring Boot 3.5+, Spring Security 6, PostgreSQL  
> Autor: ftsuda-senac

---

## Sumário

1. [Propagação de Contexto entre Threads](#1-propagação-de-contexto-entre-threads)
2. [Padrões de Controller Async](#2-padrões-de-controller-async)
3. [Progresso Periódico](#3-progresso-periódico)
4. [Async com Transações](#4-async-com-transações)
5. [Timeout por Operação](#5-timeout-por-operação)
6. [Tratamento de Exceções em Chains](#6-tratamento-de-exceções-em-chains)
7. [Configuração Avançada do Pool](#7-configuração-avançada-do-pool)
8. [Testes de Código Async](#8-testes-de-código-async)
9. [Observabilidade](#9-observabilidade)
10. [@Scheduled com @Async](#10-scheduled-com-async)
11. [@Cacheable com @Async](#11-cacheable-com-async)
12. [Resiliência com Resilience4j](#12-resiliência-com-resilience4j)
13. [Quando Ir Além do Async Local](#13-quando-ir-além-do-async-local)

---

## 1. Propagação de Contexto entre Threads

### O Problema

`SecurityContextHolder`, `RequestContextHolder` e `MDC` armazenam dados por thread via `ThreadLocal`. Quando `@Async` ou `CompletableFuture.supplyAsync()` despacha a tarefa para o pool de threads, esses contextos simplesmente não existem na thread destino.

`InheritableThreadLocal` não resolve o problema com pools porque as threads do pool são criadas uma única vez e reutilizadas — elas não são "filhas" da thread da requisição.

### Abordagem Principal — `TaskDecorator`

O decorator captura o contexto na thread da requisição **antes** de delegar ao pool, e o restaura **dentro** da thread async.

```java
// ContextPropagatingTaskDecorator.java
public class ContextPropagatingTaskDecorator implements TaskDecorator {

    @Override
    public Runnable decorate(Runnable runnable) {
        // Captura na thread da requisição
        SecurityContext securityContext = SecurityContextHolder.getContext();
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        Map<String, String> mdcContext = MDC.getCopyOfContextMap();

        return () -> {
            try {
                SecurityContextHolder.setContext(securityContext);
                if (requestAttributes != null) {
                    RequestContextHolder.setRequestAttributes(requestAttributes, true);
                }
                if (mdcContext != null) {
                    MDC.setContextMap(mdcContext);
                }
                runnable.run();
            } finally {
                // Limpeza obrigatória — threads de pool são reutilizadas
                SecurityContextHolder.clearContext();
                RequestContextHolder.resetRequestAttributes();
                MDC.clear();
            }
        };
    }
}
```

```java
// AsyncConfig.java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setTaskDecorator(new ContextPropagatingTaskDecorator());

        int cores = Runtime.getRuntime().availableProcessors();
        executor.setCorePoolSize(cores);
        executor.setMaxPoolSize(cores * 2);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-ctx-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new SimpleAsyncUncaughtExceptionHandler();
    }
}
```

### Abordagem Moderna — Micrometer Context Propagation (Spring Boot 3.x+)

O Spring Security 6+ registra automaticamente um `SecurityContextThreadLocalAccessor` no `ContextRegistry` do Micrometer. O decorator abaixo propaga automaticamente todos os `ThreadLocal`s registrados, sem precisar atualizá-lo manualmente quando novas bibliotecas são adicionadas.

```java
// MicrometerContextTaskDecorator.java
public class MicrometerContextTaskDecorator implements TaskDecorator {

    private final ContextSnapshotFactory snapshotFactory =
            ContextSnapshotFactory.builder().build();

    @Override
    public Runnable decorate(Runnable runnable) {
        ContextSnapshot snapshot = snapshotFactory.captureAll();
        return () -> {
            try (ContextSnapshot.Scope scope = snapshot.setThreadLocals()) {
                runnable.run();
            }
        };
    }
}
```

Para habilitar, troque o decorator em `AsyncConfig`:

```java
executor.setTaskDecorator(new MicrometerContextTaskDecorator());
```

### Reforço para `CompletableFuture` — `DelegatingSecurityContextAsyncTaskExecutor`

Para chains de `.thenApply()`, `.thenCompose()`, etc., o wrapper do Spring Security garante que o `SecurityContext` seja propagado também nas continuations.

```java
// No AsyncConfig — envolve o executor configurado
@Override
public Executor getAsyncExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setTaskDecorator(new ContextPropagatingTaskDecorator());
    // ... demais configurações
    executor.initialize();

    return new DelegatingSecurityContextAsyncTaskExecutor(executor);
}
```

Para `CompletableFuture` construídos manualmente fora do `@Async`:

```java
ExecutorService secureExecutor = DelegatingSecurityContextExecutorService
        .create(taskExecutor.getThreadPoolExecutor());

CompletableFuture.supplyAsync(() -> processData(), secureExecutor)
    .thenApplyAsync(result -> enrichResult(result), secureExecutor);
```

### Atenção Crítica — `HttpServletRequest` Não É Thread-Safe

O `RequestContextHolder.getRequestAttributes()` envolve o `HttpServletRequest` real. Se a task async sobreviver ao ciclo de vida da requisição HTTP (fire-and-forget), o container pode reciclar esse objeto antes da thread async terminar.

Para esses cenários, copie os dados necessários antes do dispatch:

```java
// RequestContext.java — cópia segura dos dados da requisição
public record RequestContext(
        String username,
        String tenantId,
        String correlationId
) {
    public static RequestContext capture() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return new RequestContext(
                auth != null ? auth.getName() : "anonymous",
                MDC.get("tenantId"),
                MDC.get("correlationId")
        );
    }
}
```

### Virtual Threads (Java 21+)

Com `spring.threads.virtual.enabled=true` (Spring Boot 3.2+), as threads virtuais são usadas para tasks async. O problema do `ThreadLocal` **persiste** — virtual threads criadas pelo executor não herdam o contexto da thread da requisição. O `TaskDecorator` continua funcionando da mesma forma.

> **Atenção:** `InheritableThreadLocal` é explicitamente desencorajado com virtual threads pelo Project Loom — pode causar vazamento de contexto e overhead inesperado.

---

## 2. Padrões de Controller Async

### Fire-and-Forget

O controller retorna `202 Accepted` imediatamente. A task roda em background sem que nenhuma thread aguarde o resultado. **Exceções nunca chegam ao controller** — precisam ser tratadas internamente no serviço ou no `AsyncUncaughtExceptionHandler`.

```java
// EmailRequest.java
public record EmailRequest(
        @NotBlank String to,
        @NotBlank String subject,
        @NotBlank String body
) {}
```

```java
// EmailService.java
@Slf4j
@Service
public class EmailService {

    // void + @Async = fire-and-forget puro
    // Exceções vão para AsyncUncaughtExceptionHandler — nunca para o caller
    @Async
    public void sendAsync(EmailRequest request, RequestContext ctx) {
        MDC.put("correlationId", ctx.correlationId());
        MDC.put("username", ctx.username());
        try {
            log.info("Enviando e-mail para {}", request.to());
            doSend(request);
            log.info("E-mail enviado com sucesso para {}", request.to());
        } catch (Exception ex) {
            // Trate aqui — sem try/catch a exceção é silenciosa para o cliente
            log.error("Falha ao enviar e-mail para {}: {}", request.to(), ex.getMessage(), ex);
        } finally {
            MDC.clear();
        }
    }

    private void doSend(EmailRequest request) { /* integração real */ }
}
```

```java
// EmailController.java
@Slf4j
@RestController
@RequestMapping("/api/v1/emails")
@RequiredArgsConstructor
public class EmailController {

    private final EmailService emailService;

    @PostMapping
    public ResponseEntity<Void> send(
            @RequestBody @Valid EmailRequest request,
            HttpServletRequest httpRequest
    ) {
        // Captura ANTES do async — após o método retornar o request pode ser reciclado
        RequestContext ctx = RequestContext.capture();

        emailService.sendAsync(request, ctx);

        URI statusUri = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/status/{correlationId}")
                .buildAndExpand(ctx.correlationId())
                .toUri();

        return ResponseEntity.accepted()
                .location(statusUri)
                .build();
    }
}
```

### Async com Retorno

O Spring MVC suporta `CompletableFuture<ResponseEntity<T>>` nativamente. A servlet thread é liberada imediatamente; quando o `CompletableFuture` completa, o Spring despacha a resposta sem bloquear o pool HTTP.

```java
// ReportDto.java
public record ReportDto(
        Long id,
        String content,
        String generatedBy,
        Instant generatedAt
) {}
```

```java
// ReportService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class ReportService {

    @Async
    public CompletableFuture<ReportDto> generateAsync(Long reportId) {
        // SecurityContext disponível graças ao TaskDecorator
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        String username = auth.getName();

        log.info("Iniciando geração do relatório {} para {}", reportId, username);
        try {
            ReportDto report = buildReport(reportId, username);
            return CompletableFuture.completedFuture(report);
        } catch (Exception ex) {
            // failedFuture propaga para o .exceptionally() do controller
            return CompletableFuture.failedFuture(ex);
        }
    }

    private ReportDto buildReport(Long id, String username) {
        return new ReportDto(id, "conteúdo...", username, Instant.now());
    }
}
```

```java
// ReportController.java
@Slf4j
@RestController
@RequestMapping("/api/v1/reports")
@RequiredArgsConstructor
public class ReportController {

    private final ReportService reportService;

    // Servlet thread liberada imediatamente — resposta enviada quando o future completar
    @GetMapping("/{id}")
    public CompletableFuture<ResponseEntity<ReportDto>> get(@PathVariable Long id) {
        return reportService.generateAsync(id)
                .thenApply(ResponseEntity::ok)
                .exceptionally(ex -> {
                    Throwable cause = ex.getCause() != null ? ex.getCause() : ex;
                    log.error("Erro ao gerar relatório {}: {}", id, cause.getMessage(), cause);

                    if (cause instanceof ReportNotFoundException) {
                        return ResponseEntity.notFound().build();
                    }
                    return ResponseEntity.internalServerError().build();
                });
    }
}
```

### Timeout Global

Sem timeout, um `CompletableFuture` pode prender a conexão indefinidamente.

```yaml
# application.yml
spring:
  mvc:
    async:
      request-timeout: 30s
```

```java
// MvcConfig.java — timeout por endpoint via WebMvcConfigurer
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        configurer.setDefaultTimeout(30_000); // ms
    }
}
```

### Comparação — Fire-and-Forget vs Async com Retorno

| | Fire-and-forget (`void`) | Async com retorno (`CompletableFuture`) |
|---|---|---|
| Thread do caller aguarda? | Não | Não — servlet thread é liberada |
| Exceção chega ao caller? | Nunca | Sim, via `.exceptionally()` |
| HTTP response | `202 Accepted` imediato | Aguarda o future completar |
| Timeout aplicável? | Não | Sim — `request-timeout` |
| Uso típico | Notificações, auditoria, eventos | Consultas pesadas, agregações, I/O |

---

## 3. Progresso Periódico

### SSE com `SseEmitter`

Servidor envia eventos em tempo real via push. Ideal quando o cliente aguarda ativamente o resultado com baixa latência.

```java
// ProgressEvent.java
public record ProgressEvent(
        int percentage,
        String message,
        Instant timestamp
) {}
```

```java
// ReportService.java
@Slf4j
@Service
public class ReportService {

    // void + @Async: o "retorno" são os eventos SSE, não o valor do método
    @Async
    public void generateWithProgressAsync(Long reportId, SseEmitter emitter, RequestContext ctx) {
        MDC.put("correlationId", ctx.correlationId());
        try {
            progress(emitter, 10, "Validando parâmetros");
            validateParams(reportId);

            progress(emitter, 35, "Buscando dados");
            var data = fetchData(reportId);

            progress(emitter, 75, "Processando relatório");
            var report = buildReport(reportId, data, ctx.username());

            // Evento final com o payload completo
            emitter.send(SseEmitter.event()
                    .name("result")
                    .data(report));
            emitter.complete();

        } catch (UncheckedIOException ex) {
            // Cliente desconectou — não há o que fazer
            log.warn("Cliente desconectou durante geração do relatório {}", reportId);
        } catch (Exception ex) {
            log.error("Erro ao gerar relatório {}", reportId, ex);
            sendError(emitter, ex.getMessage());
            emitter.completeWithError(ex);
        } finally {
            MDC.clear();
        }
    }

    private void progress(SseEmitter emitter, int pct, String message) {
        try {
            emitter.send(SseEmitter.event()
                    .name("progress")
                    .data(new ProgressEvent(pct, message, Instant.now())));
        } catch (IOException ex) {
            // Lança para interromper a task — não faz sentido continuar processando
            throw new UncheckedIOException("Cliente desconectou", ex);
        }
    }

    private void sendError(SseEmitter emitter, String message) {
        try {
            emitter.send(SseEmitter.event()
                    .name("error")
                    .data(Map.of("message", message)));
        } catch (IOException ignored) {}
    }
}
```

```java
// ReportController.java
@Slf4j
@RestController
@RequestMapping("/api/v1/reports")
@RequiredArgsConstructor
public class ReportController {

    private final ReportService reportService;

    @GetMapping(value = "/{id}/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter stream(@PathVariable Long id) {
        SseEmitter emitter = new SseEmitter(120_000L); // timeout do processo (2 min)

        emitter.onTimeout(() -> {
            log.warn("SSE timeout para relatório {}", id);
            emitter.complete();
        });
        emitter.onError(ex -> log.error("SSE erro para relatório {}", id, ex));

        RequestContext ctx = RequestContext.capture();
        reportService.generateWithProgressAsync(id, emitter, ctx);

        return emitter;
    }
}
```

**Consumo no cliente:**

```javascript
const source = new EventSource(`/api/v1/reports/${id}/stream`);

source.addEventListener('progress', (e) => {
    const { percentage, message } = JSON.parse(e.data);
    updateProgressBar(percentage, message);
});

source.addEventListener('result', (e) => {
    const report = JSON.parse(e.data);
    renderReport(report);
    source.close();
});

source.addEventListener('error', (e) => {
    const { message } = JSON.parse(e.data);
    showError(message);
    source.close();
});
```

---

### Polling com Store de Progresso

Cliente consulta um endpoint periodicamente. Mais resiliente a reconexões e adequado para jobs longos ou clientes que não suportam SSE.

```java
// JobProgress.java
public record JobProgress(
        String jobId,
        int percentage,
        String message,
        JobState state,
        Object result,
        String error,
        Instant updatedAt
) {
    public enum JobState { PENDING, RUNNING, COMPLETED, FAILED }

    public boolean isTerminal() {
        return state == JobState.COMPLETED || state == JobState.FAILED;
    }
}
```

```java
// ProgressStore.java
// Caffeine para TTL automático — em ambiente distribuído, use Redis
@Component
public class ProgressStore {

    private final Cache<String, JobProgress> cache = Caffeine.newBuilder()
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .maximumSize(10_000)
            .build();

    public void init(String jobId) {
        put(jobId, 0, "Aguardando início...", JobProgress.JobState.PENDING, null, null);
    }

    public void update(String jobId, int pct, String message) {
        put(jobId, pct, message, JobProgress.JobState.RUNNING, null, null);
    }

    public void complete(String jobId, Object result) {
        put(jobId, 100, "Concluído", JobProgress.JobState.COMPLETED, result, null);
    }

    public void fail(String jobId, String error) {
        put(jobId, -1, error, JobProgress.JobState.FAILED, null, error);
    }

    public Optional<JobProgress> get(String jobId) {
        return Optional.ofNullable(cache.getIfPresent(jobId));
    }

    private void put(String jobId, int pct, String msg,
                     JobProgress.JobState state, Object result, String error) {
        cache.put(jobId, new JobProgress(jobId, pct, msg, state, result, error, Instant.now()));
    }
}
```

```java
// ReportService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class ReportService {

    private final ProgressStore progressStore;

    @Async
    public void generateAsync(String jobId, ReportRequest request, RequestContext ctx) {
        MDC.put("correlationId", ctx.correlationId());
        try {
            progressStore.update(jobId, 10, "Validando parâmetros");
            validateParams(request);

            progressStore.update(jobId, 35, "Buscando dados");
            var data = fetchData(request);

            progressStore.update(jobId, 75, "Processando relatório");
            var report = buildReport(request, data, ctx.username());

            progressStore.complete(jobId, report);
            log.info("Relatório {} finalizado", jobId);

        } catch (Exception ex) {
            log.error("Erro ao gerar relatório {}", jobId, ex);
            progressStore.fail(jobId, ex.getMessage());
        } finally {
            MDC.clear();
        }
    }
}
```

```java
// ReportController.java
@Slf4j
@RestController
@RequestMapping("/api/v1/reports")
@RequiredArgsConstructor
public class ReportController {

    private final ReportService reportService;
    private final ProgressStore progressStore;

    @PostMapping
    public ResponseEntity<Map<String, String>> start(@RequestBody @Valid ReportRequest request) {
        String jobId = UUID.randomUUID().toString();

        progressStore.init(jobId);
        reportService.generateAsync(jobId, request, RequestContext.capture());

        URI statusUri = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/status/{jobId}")
                .buildAndExpand(jobId)
                .toUri();

        return ResponseEntity.accepted()
                .location(statusUri)
                .body(Map.of("jobId", jobId));
    }

    @GetMapping("/status/{jobId}")
    public ResponseEntity<JobProgress> status(@PathVariable String jobId) {
        return progressStore.get(jobId)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}
```

**Consumo no cliente:**

```javascript
const { jobId } = await fetch('/api/v1/reports', {
    method: 'POST',
    body: JSON.stringify(payload)
}).then(r => r.json());

const poll = setInterval(async () => {
    const progress = await fetch(`/api/v1/reports/status/${jobId}`).then(r => r.json());

    updateProgressBar(progress.percentage, progress.message);

    if (progress.state === 'COMPLETED') {
        renderReport(progress.result);
        clearInterval(poll);
    }
    if (progress.state === 'FAILED') {
        showError(progress.error);
        clearInterval(poll);
    }
}, 2000);
```

### Comparação — SSE vs Polling

| | SSE (`SseEmitter`) | Polling |
|---|---|---|
| Latência do progresso | Imediata (push) | Até N segundos |
| Conexão HTTP | Mantida aberta durante todo o job | Uma por consulta |
| Reconexão automática | Sim, via `EventSource` (sem replay) | Sim, naturalmente |
| Job outlives a conexão | Não — perde o progresso se cair | Sim — store persiste independente |
| Múltiplos clientes monitorando | Requer um emitter por conexão | Qualquer cliente lê o mesmo store |
| Adequado para jobs > 2 min | Arriscado (timeouts de proxy/LB) | Sim |
| Suporte no cliente | Apenas browsers modernos (`EventSource`) | Qualquer cliente HTTP |

---

## 4. Async com Transações

### Por Que Transações Não Propagam entre Threads

`TransactionSynchronizationManager` usa `ThreadLocal` para vincular a conexão JDBC ativa à thread corrente. Quando `@Async` despacha para o pool, a nova thread começa sem nenhuma conexão — e não há como transferir uma conexão JDBC entre threads porque a própria especificação JDBC não permite.

A abordagem correta é que **cada thread gerencie sua própria transação**.

> Quando um método `@Async @Transactional` é chamado de dentro de outro `@Transactional`, a transação da thread chamadora **não é herdada**. `Propagation.REQUIRED` (padrão) age como `REQUIRES_NEW` nesse contexto porque a thread do pool não enxerga a transação da thread pai.

### Padrão 1 — `@Async @Transactional` na Mesma Operação

Os proxies são aplicados nessa ordem: `@Async` intercepta primeiro → despacha para o pool → na nova thread, `@Transactional` abre a transação → executa → commit/rollback.

```java
// OrderService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Async
    @Transactional(readOnly = true)         // transação abre NA thread do pool
    public CompletableFuture<List<OrderDto>> findByCustomerAsync(Long customerId) {
        List<Order> orders = orderRepository.findByCustomerId(customerId);

        // Map para DTO DENTRO da transação — lazy collections ainda estão abertas aqui
        return CompletableFuture.completedFuture(
                orders.stream().map(OrderDto::from).toList()
        );
    }

    @Async
    @Transactional
    public CompletableFuture<OrderDto> processOrderAsync(Long orderId, OrderCommand command) {
        Order order = orderRepository.findById(orderId)
                .orElseThrow(() -> new OrderNotFoundException(orderId));

        order.apply(command);               // dirty checking garante UPDATE no commit

        return CompletableFuture.completedFuture(OrderDto.from(order));
    }
}
```

### Padrão 2 — Leituras Paralelas + Escrita Consolidada

Múltiplos repositórios consultados em paralelo; cada método é uma transação independente.

```java
// SalesQueryService.java
@Service
@RequiredArgsConstructor
public class SalesQueryService {

    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;

    @Async
    @Transactional(readOnly = true)
    public CompletableFuture<List<OrderSummary>> findOrderSummariesAsync(
            LocalDate from, LocalDate to) {
        return CompletableFuture.completedFuture(
                orderRepository.findByPeriod(from, to).stream()
                        .map(OrderSummary::from)
                        .toList()
        );
    }

    @Async
    @Transactional(readOnly = true)
    public CompletableFuture<List<ProductSummary>> findTopProductsAsync(
            LocalDate from, LocalDate to, int limit) {
        return CompletableFuture.completedFuture(
                productRepository.findTopSellers(from, to, limit).stream()
                        .map(ProductSummary::from)
                        .toList()
        );
    }
}
```

A escrita deve estar em um **bean separado** para que o proxy Spring esteja ativo:

```java
// SalesReportPersistence.java
@Component
@RequiredArgsConstructor
public class SalesReportPersistence {

    private final SalesReportRepository reportRepository;

    @Transactional
    public SalesReportDto save(ReportData data) {
        SalesReport report = SalesReport.builder()
                .period(data.period())
                .orderCount(data.orderSummaries().size())
                .topProducts(data.productSummaries())
                .generatedAt(Instant.now())
                .build();
        return SalesReportDto.from(reportRepository.save(report));
    }
}
```

```java
// SalesReportService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class SalesReportService {

    private final SalesQueryService queryService;
    private final SalesReportPersistence persistence; // bean injetado — proxy Spring ativo

    @Async
    public CompletableFuture<SalesReportDto> generateAsync(SalesReportRequest request) {
        // Dispara as duas leituras em paralelo — cada uma abre sua própria transação
        CompletableFuture<List<OrderSummary>> ordersFuture =
                queryService.findOrderSummariesAsync(request.from(), request.to());

        CompletableFuture<List<ProductSummary>> productsFuture =
                queryService.findTopProductsAsync(request.from(), request.to(), 10);

        return CompletableFuture.allOf(ordersFuture, productsFuture)
                .thenApply(v -> new ReportData(
                        request.period(),
                        ordersFuture.join(),
                        productsFuture.join()
                ))
                // persistence é bean injetado → @Transactional funciona corretamente
                .thenApply(data -> persistence.save(data));
    }
}
```

### Armadilha 1 — `LazyInitializationException`

Retornar entidades JPA antes de mapear para DTO fecha a sessão Hibernate antes que o `thenApply` execute o mapeamento.

```java
// ERRADO — a transação fecha quando o método retorna
@Async
@Transactional(readOnly = true)
public CompletableFuture<List<Order>> findOrdersAsync(Long customerId) {
    List<Order> orders = orderRepository.findByCustomerId(customerId);
    return CompletableFuture.completedFuture(orders); // sessão encerra aqui
}

// Na chamada — sessão já fechada, lazy load explode
.thenApply(orders -> orders.stream()
        .map(OrderDto::from) // LazyInitializationException se Order tiver @OneToMany LAZY
        .toList())
```

```java
// CORRETO — mapeamento ocorre dentro do escopo da transação
@Async
@Transactional(readOnly = true)
public CompletableFuture<List<OrderDto>> findOrdersAsync(Long customerId) {
    return CompletableFuture.completedFuture(
            orderRepository.findByCustomerId(customerId).stream()
                    .map(OrderDto::from)   // sessão ativa — lazy load funciona
                    .toList()
    );
}
```

### Armadilha 2 — Auto-invocação Quebra `@Transactional`

Chamar um método `@Transactional` via `this` dentro do mesmo bean bypassa o proxy do Spring.

```java
// ERRADO — this.save() bypassa o proxy, @Transactional é ignorado silenciosamente
@Service
public class ReportService {

    @Async
    public CompletableFuture<ReportDto> generateAsync(ReportRequest req) {
        return buildData(req)
                .thenApply(data -> this.save(data)); // proxy ignorado — sem transação!
    }

    @Transactional  // nunca executado via this
    public ReportDto save(ReportData data) { ... }
}
```

**Solução preferida — extrair para bean dedicado** (conforme Padrão 2 acima).

**Alternativa — auto-injeção com `@Lazy`:**

```java
// CORRETO — auto-injeção do proxy do próprio bean
@Service
public class ReportService {

    @Lazy
    @Autowired
    private ReportService self;

    @Async
    public CompletableFuture<ReportDto> generateAsync(ReportRequest req) {
        return buildData(req)
                .thenApply(data -> self.save(data)); // via proxy — @Transactional ativo
    }

    @Transactional
    public ReportDto save(ReportData data) { ... }
}
```

### Resumo — Async com Transações

| Cenário | Solução |
|---|---|
| Operação única async + transação | `@Async @Transactional` no mesmo método |
| Leitura paralela de múltiplos repos | Um `@Async @Transactional(readOnly=true)` por método |
| Escrita após agregação paralela | Bean separado com `@Transactional` chamado via proxy |
| Lazy loading em contexto async | Mapear para DTO dentro do método `@Transactional` |
| Escrita no mesmo bean do orquestrador | Auto-injeção com `@Lazy` ou extrair para bean dedicado |

---

## 5. Timeout por Operação

O `request-timeout` do MVC encerra toda a requisição HTTP. Para impor limites por chamada individual dentro de uma chain, use os métodos da própria `CompletableFuture`.

```java
// orTimeout — lança CompletionException(TimeoutException) se não completar no prazo
CompletableFuture<ReportDto> future = reportService.generateAsync(reportId)
        .orTimeout(10, TimeUnit.SECONDS);

// completeOnTimeout — completa com valor padrão em vez de lançar exceção
CompletableFuture<ReportDto> future = reportService.generateAsync(reportId)
        .completeOnTimeout(ReportDto.empty(), 10, TimeUnit.SECONDS);
```

> **Atenção:** `orTimeout()` e `completeOnTimeout()` **não interrompem** a thread em execução. A computação continua rodando no pool; apenas o resultado é descartado ou substituído pelo valor de timeout. Para parar o trabalho efetivamente, use cancelamento cooperativo.

### Cancelamento Cooperativo

```java
// ReportService.java
@Async
public CompletableFuture<ReportDto> generateAsync(Long reportId, AtomicBoolean cancelled) {
    try {
        var step1 = fetchData(reportId);
        if (cancelled.get()) return CompletableFuture.failedFuture(new CancellationException());

        var step2 = processData(step1);
        if (cancelled.get()) return CompletableFuture.failedFuture(new CancellationException());

        return CompletableFuture.completedFuture(buildReport(step2));
    } catch (CancellationException ex) {
        log.info("Relatório {} cancelado", reportId);
        return CompletableFuture.failedFuture(ex);
    }
}
```

```java
// ReportController.java
@GetMapping("/{id}")
public CompletableFuture<ResponseEntity<ReportDto>> get(@PathVariable Long id) {
    AtomicBoolean cancelled = new AtomicBoolean(false);

    return reportService.generateAsync(id, cancelled)
            .orTimeout(10, TimeUnit.SECONDS)
            .thenApply(ResponseEntity::ok)
            .exceptionally(ex -> {
                Throwable cause = ex instanceof CompletionException ce ? ce.getCause() : ex;
                if (cause instanceof TimeoutException || cause instanceof CancellationException) {
                    cancelled.set(true); // sinaliza a thread async para encerrar
                    return ResponseEntity.status(HttpStatus.GATEWAY_TIMEOUT).build();
                }
                return ResponseEntity.internalServerError().build();
            });
}
```

---

## 6. Tratamento de Exceções em Chains

### Unwrapping — `CompletionException` e `ExecutionException`

Exceptions lançadas dentro de um `CompletableFuture` são sempre embrulhadas:
- `.join()` lança `CompletionException` — use `.getCause()` para a causa real
- `.get()` lança `ExecutionException` — use `.getCause()` para a causa real
- `.exceptionally()` recebe `CompletionException` — mesma regra

```java
// Utilitário para unwrap seguro — reutilize em todos os pontos de tratamento
static Throwable unwrap(Throwable ex) {
    return (ex instanceof CompletionException || ex instanceof ExecutionException)
            && ex.getCause() != null ? ex.getCause() : ex;
}
```

```java
// Uso correto em .exceptionally()
.exceptionally(ex -> {
    Throwable cause = unwrap(ex);
    log.error("Falha: {}", cause.getMessage(), cause);

    return switch (cause) {
        case ReportNotFoundException e -> ResponseEntity.notFound().<ReportDto>build();
        case ValidationException e     -> ResponseEntity.badRequest().<ReportDto>build();
        default                        -> ResponseEntity.internalServerError().<ReportDto>build();
    };
})
```

### Comparação — `exceptionally`, `handle` e `whenComplete`

| Método | Chamado quando | Altera o resultado? | Uso típico |
|---|---|---|---|
| `.exceptionally(fn)` | Apenas em exception | Sim — retorna novo valor | Fallback, mapeamento de erro para HTTP status |
| `.handle(fn)` | Sempre (sucesso ou falha) | Sim — retorna novo valor | Transformação condicional de resultado |
| `.whenComplete(fn)` | Sempre | Não — efeito colateral | Logging, métricas, limpeza de recursos |

```java
// handle — trata sucesso e falha em um único estágio
.handle((result, ex) -> {
    if (ex != null) {
        Throwable cause = unwrap(ex);
        log.error("Falha ao gerar relatório", cause);
        return ReportDto.failed(cause.getMessage());
    }
    return result;
})

// whenComplete — side effect sem alterar o resultado (não substitui exceptionally)
.whenComplete((result, ex) -> {
    if (ex != null) {
        meterRegistry.counter("report.error").increment();
    } else {
        meterRegistry.counter("report.success").increment();
    }
})
```

### Re-lançar sem Perder o Tipo Original

```java
// ERRADO — embrulha desnecessariamente, dificulta o instanceof posterior
.handle((result, ex) -> {
    if (ex != null) throw new RuntimeException(ex);
    return result;
})

// CORRETO — propaga sem embrulhar quando já é RuntimeException
.handle((result, ex) -> {
    if (ex != null) {
        Throwable cause = unwrap(ex);
        if (cause instanceof RuntimeException re) throw re;
        throw new CompletionException(cause);
    }
    return result;
})
```

---

## 7. Configuração Avançada do Pool

### Política de Rejeição

Quando a fila (`queueCapacity`) está cheia e o pool está no máximo (`maxPoolSize`), novas tarefas são rejeitadas. O Spring lança `TaskRejectedException` (wrapping `RejectedExecutionException`).

| `RejectedExecutionHandler` | Comportamento | Quando usar |
|---|---|---|
| `AbortPolicy` (padrão) | Lança `TaskRejectedException` | Rejeição deve ser tratada explicitamente pelo caller |
| `CallerRunsPolicy` | Executa na thread do caller — bloqueia | Backpressure natural; limita a taxa de submissão |
| `DiscardPolicy` | Descarta silenciosamente | Tarefas opcionais (métricas secundárias, heartbeats) |
| `DiscardOldestPolicy` | Remove a mais antiga da fila e retenta | Apenas o estado mais recente importa (ex.: snapshots de sensor) |

```java
// AsyncConfig.java
executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
```

> **Atenção com `CallerRunsPolicy`:** a servlet thread passa a executar a task diretamente até o fim. Sob carga alta, isso bloqueia o pool HTTP e pode parecer um bug de regressão de performance.

### Múltiplos Executors Nomeados

Pools separados evitam que tarefas lentas (relatórios, I/O externo) bloqueiem tarefas rápidas (notificações, auditoria).

```java
// AsyncConfig.java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    // Executor padrão para @Async sem nome explícito
    @Override
    public Executor getAsyncExecutor() {
        return buildExecutor("async-default-", 4, 8, 100,
                new ThreadPoolExecutor.AbortPolicy());
    }

    // Pool para relatórios pesados — I/O-bound, fila pequena para detectar sobrecarga cedo
    @Bean("reportExecutor")
    public Executor reportExecutor() {
        return buildExecutor("async-report-", 2, 8, 20,
                new ThreadPoolExecutor.CallerRunsPolicy());
    }

    // Pool para notificações — alta concorrência, descarte aceitável
    @Bean("notificationExecutor")
    public Executor notificationExecutor() {
        return buildExecutor("async-notif-", 4, 16, 500,
                new ThreadPoolExecutor.DiscardPolicy());
    }

    private Executor buildExecutor(String prefix, int core, int max, int queue,
                                   RejectedExecutionHandler rejectionHandler) {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setTaskDecorator(new ContextPropagatingTaskDecorator());
        executor.setCorePoolSize(core);
        executor.setMaxPoolSize(max);
        executor.setQueueCapacity(queue);
        executor.setRejectedExecutionHandler(rejectionHandler);
        executor.setThreadNamePrefix(prefix);
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        executor.initialize();
        return executor;
    }
}
```

```java
// Serviços referenciando o executor pelo nome do bean
@Async("reportExecutor")
public CompletableFuture<ReportDto> generateAsync(Long reportId) { ... }

@Async("notificationExecutor")
public void sendAsync(Notification notification) { ... }
```

### Dimensionamento do Pool

| Tipo de tarefa | `corePoolSize` | `maxPoolSize` | `queueCapacity` |
|---|---|---|---|
| CPU-bound | `nCPU` | `nCPU` | Pequena (10–50) |
| I/O-bound (HTTP, DB, disco) | `nCPU * 2` | `nCPU * 4–8` | Moderada (100–500) |
| Fire-and-forget (alta vazão) | `nCPU` | `nCPU * 2` | Grande (1 000+) |

> **Regra prática para I/O-bound:** prefira fila pequena e `max` alto. Threads adicionais são baratas; mensagens presas na fila atrasam o progresso percebido e dificultam a detecção de sobrecarga.

---

## 8. Testes de Código Async

### Regra Fundamental

Métodos `@Async` só funcionam via proxy Spring. Instanciar o serviço com `new` ou invocar `this.method()` dentro do próprio bean **bypassa o proxy** e executa sincronamente — sem dispatch ao pool. Sempre injete o bean via contexto Spring nos testes de integração.

### AssertJ — Assertions Diretas em `CompletableFuture`

```java
@SpringBootTest
class ReportServiceTest {

    @Autowired
    ReportService reportService; // proxy Spring — @Async funciona

    @Test
    void deveGerarRelatorio() {
        CompletableFuture<ReportDto> future = reportService.generateAsync(1L);

        assertThat(future)
                .succeedsWithin(5, TimeUnit.SECONDS)
                .satisfies(dto -> {
                    assertThat(dto.id()).isEqualTo(1L);
                    assertThat(dto.generatedBy()).isNotBlank();
                });
    }

    @Test
    void deveFalharParaIdInexistente() {
        CompletableFuture<ReportDto> future = reportService.generateAsync(999L);

        assertThat(future)
                .failsWithin(5, TimeUnit.SECONDS)
                .withThrowableOfType(ExecutionException.class)
                .havingCause()
                .isInstanceOf(ReportNotFoundException.class);
    }
}
```

### Awaitility — Fire-and-Forget

Para métodos `void` que não retornam `CompletableFuture`, aguarde o efeito colateral com Awaitility.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
class EmailServiceTest {

    @Autowired
    EmailService emailService;

    @MockitoBean
    EmailSender emailSender;

    @Test
    void deveEnviarEmailEmBackground() {
        EmailRequest request = new EmailRequest("dest@exemplo.com", "Assunto", "Corpo");
        RequestContext ctx = new RequestContext("user1", "tenant-a", "corr-123");

        emailService.sendAsync(request, ctx);

        // Aguarda até 5s para que o envio async seja concluído
        await().atMost(5, TimeUnit.SECONDS)
               .untilAsserted(() ->
                   verify(emailSender).send(argThat(r -> r.to().equals("dest@exemplo.com")))
               );
    }
}
```

### Executor Síncrono — Testes Unitários Rápidos

Substitua o pool por `SyncTaskExecutor` para testes unitários onde o Spring context é carregado mas a assincronia não é o foco — sem `Awaitility`, sem timeouts, execução determinística.

```java
// AsyncTestConfig.java
@TestConfiguration
public class AsyncTestConfig implements AsyncConfigurer {

    // Executa a task na thread chamadora, de forma síncrona
    @Override
    public Executor getAsyncExecutor() {
        return new SyncTaskExecutor();
    }
}
```

```java
@SpringBootTest
@Import(AsyncTestConfig.class)  // substitui AsyncConfig de produção
class EmailServiceFastTest {

    @Autowired
    EmailService emailService;

    @MockitoBean
    EmailSender emailSender;

    @Test
    void deveLogarEnvio() {
        // Com SyncTaskExecutor não precisa de Awaitility — execução é imediata
        emailService.sendAsync(
                new EmailRequest("a@b.com", "S", "B"),
                new RequestContext("user", "t1", "corr-1"));

        verify(emailSender).send(any());
    }
}
```

> **Atenção:** `SyncTaskExecutor` ignora o `TaskDecorator`. Se o teste valida comportamento que depende de `SecurityContext` ou `MDC`, use um `ThreadPoolTaskExecutor` com `corePoolSize = 1` em vez disso.

---

## 9. Observabilidade

### Métricas do Executor via Actuator

Registre o `ThreadPoolExecutor` no `MeterRegistry` para expor métricas via `/actuator/metrics`:

```java
// AsyncConfig.java
@Bean("reportExecutor")
public Executor reportExecutor(MeterRegistry registry) {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(2);
    executor.setMaxPoolSize(8);
    executor.setQueueCapacity(20);
    executor.setThreadNamePrefix("async-report-");
    executor.initialize();

    // Registra com a tag pool=report
    ExecutorServiceMetrics.monitor(
            registry,
            executor.getThreadPoolExecutor(),
            "report",
            List.of(Tag.of("pool", "report")));
    return executor;
}
```

Métricas disponíveis:

| Métrica | Descrição |
|---|---|
| `executor.active` | Threads executando tasks no momento |
| `executor.pool.size` | Total de threads criadas no pool |
| `executor.queued` | Tasks aguardando na fila |
| `executor.completed` | Tasks concluídas (contador cumulativo) |
| `executor.pool.core` | `corePoolSize` configurado |
| `executor.pool.max` | `maxPoolSize` configurado |

```bash
# Exemplo de consulta
curl /actuator/metrics/executor.queued?tag=pool:report
```

> **Alerta útil:** configure alertas quando `executor.queued` ultrapassar 80% de `queueCapacity`. Rejeições começam somente quando a fila está 100% cheia — o alerta antecipa o problema antes de erros chegarem ao usuário.

### Rastreamento Distribuído (Micrometer Tracing)

O `MicrometerContextTaskDecorator` (seção 1) propaga automaticamente o trace context. Dois cenários que exigem atenção adicional:

**Fire-and-forget:** o span da requisição HTTP fecha antes da task async terminar. Crie um span filho para o trabalho em background:

```java
// EmailService.java
@Autowired
private Tracer tracer;

@Async
public void sendAsync(EmailRequest request, RequestContext ctx) {
    Span span = tracer.nextSpan()
            .name("email.send")
            .tag("email.to", request.to())
            .start();
    try (Tracer.SpanInScope ignored = tracer.withSpan(span)) {
        MDC.put("correlationId", ctx.correlationId());
        doSend(request);
    } catch (Exception ex) {
        span.error(ex);
        throw ex;
    } finally {
        span.end();
        MDC.clear();
    }
}
```

**Chains com `thenApplyAsync`:** em continuations executadas em threads diferentes do pool, o trace ID é propagado corretamente apenas quando o executor configurado usa o `MicrometerContextTaskDecorator`. Um executor custom passado diretamente ao `thenApplyAsync()` sem decorator **perde o trace**.

---

## 10. `@Scheduled` com `@Async`

Por padrão, `@Scheduled` usa uma única thread (scheduler thread). Tasks longas bloqueiam toda a fila de agendamentos. Adicionando `@Async`, cada execução é despachada para o pool, liberando a thread do scheduler imediatamente.

```java
// ScheduledConfig.java
@Configuration
@EnableScheduling
public class ScheduledConfig {

    @Bean
    public ThreadPoolTaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(2);  // suporta múltiplos @Scheduled simultâneos
        scheduler.setThreadNamePrefix("scheduler-");
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        return scheduler;
    }
}
```

```java
// SyncService.java
@Slf4j
@Service
public class SyncService {

    @Scheduled(fixedDelay = 60_000)  // próxima execução começa 60s após o FIM
    @Async("syncExecutor")            // scheduler thread liberada imediatamente
    public void syncData() {
        log.info("Sincronizando dados...");
        doHeavySync();
    }
}
```

### `fixedRate` vs `fixedDelay` com `@Async`

| | `fixedDelay` | `fixedRate` |
|---|---|---|
| Próximo disparo | 60s após o FIM da execução | 60s após o INÍCIO, independente do término |
| Sem `@Async` | Sobreposição impossível | Sobreposição se task > intervalo (aguarda na fila do scheduler) |
| Com `@Async` | Sobreposição impossível (disparo só após fim) | **Sobreposição possível** — cada disparo vira task independente no pool |

> **Prefira `fixedDelay` com `@Async`** para evitar sobreposição sem locks adicionais. Se precisar de `fixedRate` com garantia de execução única, use um guard:

```java
private final AtomicBoolean running = new AtomicBoolean(false);

@Scheduled(fixedRate = 60_000)
@Async("syncExecutor")
public void syncData() {
    if (!running.compareAndSet(false, true)) {
        log.warn("Sincronização já em andamento — execução ignorada");
        return;
    }
    try {
        doHeavySync();
    } finally {
        running.set(false);
    }
}
```

---

## 11. `@Cacheable` com `@Async`

### Comportamento por Versão do Spring

| Versão | Comportamento com retorno `CompletableFuture` |
|---|---|
| Spring 6.0 / Boot 3.0–3.1 | Armazena o `CompletableFuture` em si, não o valor resolvido — cache não funciona como esperado |
| Spring 6.1 / Boot 3.2+ | Suporte nativo: aguarda o future completar e armazena o valor resolvido |

### Spring Boot 3.2+ — Comportamento Nativo

```java
@Async
@Cacheable(value = "reports", key = "#reportId")
public CompletableFuture<ReportDto> generateAsync(Long reportId) {
    // Spring 6.1+ armazena o ReportDto resolvido, não o CompletableFuture
    return CompletableFuture.completedFuture(buildReport(reportId));
}
```

### Spring Boot 3.1 ou Anterior — Bean Dedicado de Cache

Separe a camada de cache (síncrona) da camada async para garantir que o `@Cacheable` opere sobre o valor concreto:

```java
// ReportCacheService.java
@Service
public class ReportCacheService {

    @Cacheable(value = "reports", key = "#reportId")
    public ReportDto getOrGenerate(Long reportId) {
        return buildReport(reportId);  // armazena ReportDto diretamente
    }
}

// ReportService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class ReportService {

    private final ReportCacheService cacheService;

    @Async
    public CompletableFuture<ReportDto> generateAsync(Long reportId) {
        // lookup e store do cache ocorrem na thread do pool — funciona corretamente
        return CompletableFuture.completedFuture(cacheService.getOrGenerate(reportId));
    }
}
```

> **Regra geral:** nunca coloque `@Async` e `@Cacheable` no mesmo método em versões anteriores ao Spring Boot 3.2 — o comportamento é incorreto e silencioso. O cache armazena o wrapper `CompletableFuture`, que nunca reaproveita a computação.

---

## 12. Resiliência com Resilience4j

### Onde Aplicar as Anotações

`@Async` atua como proxy mais externo: despacha a chamada ao pool e retorna imediatamente. `@CircuitBreaker`, `@Retry` e `@TimeLimiter` aplicados **no mesmo método** executam dentro da thread do pool, que é o comportamento correto. O problema é arquitetural: o método async acumula responsabilidades e o fallback do `@CircuitBreaker` também precisa retornar `CompletableFuture`.

**Padrão mais limpo** — separe dispatch de resiliência:

```java
// ExternalApiService.java
@Slf4j
@Service
@RequiredArgsConstructor
public class ExternalApiService {

    // Despacha para o pool — sem anotações de resiliência aqui
    @Async
    public CompletableFuture<ApiResult> callAsync(String param) {
        return CompletableFuture.completedFuture(callWithResilience(param));
    }

    // Resiliência na camada síncrona — proxy Spring ativo via bean injetado
    @CircuitBreaker(name = "externalApi", fallbackMethod = "fallback")
    @Retry(name = "externalApi")
    public ApiResult callWithResilience(String param) {
        return externalClient.call(param);
    }

    // Assinatura: mesmos parâmetros + Throwable ao final
    public ApiResult fallback(String param, Throwable ex) {
        log.warn("Fallback ativado para param={}: {}", param, ex.getMessage());
        return ApiResult.empty();
    }
}
```

> **Atenção:** `callWithResilience` precisa ser chamado via proxy — portanto, deve estar em um bean separado ou via auto-injeção com `@Lazy` (a armadilha de auto-invocação da seção 4 se aplica aqui também).

### `TimeLimiter` do Resilience4j

Alternativa ao `.orTimeout()` quando você precisa de métricas automáticas e fallback integrado ao circuit breaker:

```yaml
# application.yml
resilience4j:
  timelimiter:
    instances:
      externalApi:
        timeout-duration: 10s
        cancel-running-future: true  # chama future.cancel(true) ao expirar
```

```java
@Async
@TimeLimiter(name = "externalApi", fallbackMethod = "timeoutFallback")
@CircuitBreaker(name = "externalApi")
public CompletableFuture<ApiResult> callAsync(String param) {
    return CompletableFuture.supplyAsync(() -> externalClient.call(param));
}

public CompletableFuture<ApiResult> timeoutFallback(String param, TimeoutException ex) {
    log.warn("Timeout ao chamar API para param={}", param);
    return CompletableFuture.completedFuture(ApiResult.empty());
}
```

> **`cancel-running-future: true`** chama `future.cancel(true)`, que envia `interrupt()` à thread se ela estiver bloqueada em I/O. Para computações CPU-bound, o cancelamento cooperativo com `AtomicBoolean` (seção 5) ainda é necessário.

---

## 13. Quando Ir Além do Async Local

`@Async` e `CompletableFuture` processam tarefas dentro do mesmo processo JVM. Se um dos sinais abaixo aparecer, o custo de manter a solução local supera o custo de adotar mensageria:

| Sinal | Problema com async local | Solução recomendada |
|---|---|---|
| Jobs > 2–3 min | Timeouts de proxy/LB encerram a conexão | Mensageria + polling ou SSE de status |
| Retry com persistência | Falha no processo descarta tarefas na fila em memória | Outbox pattern + RabbitMQ / Kafka |
| Múltiplas instâncias (pods) | Pool in-memory não é compartilhado entre pods | Fila externa compartilhada |
| Auditoria de execuções | `CompletableFuture` não deixa rastro persistente | Spring Batch / Quartz com job store no banco |
| Rate limit no consumidor externo | Pool async não respeita limites da API destino | Fila com consumer throttled |

### Outbox Pattern — Ponte Transacional para Mensageria

Garante que o evento seja publicado exatamente uma vez, mesmo que o broker esteja indisponível no momento da escrita:

```java
// OrderService.java
@Transactional
public Order placeOrder(OrderCommand command) {
    Order order = orderRepository.save(Order.from(command));

    // Persiste o evento na mesma transação — nunca perdido se o processo cair
    outboxRepository.save(OutboxEvent.of("order.placed", order.getId(), order));

    return order;  // broker ainda não foi chamado
}
```

```java
// OutboxPublisher.java — relay assíncrono para o broker
@Slf4j
@Service
@RequiredArgsConstructor
public class OutboxPublisher {

    private final OutboxRepository outboxRepository;
    private final MessageBroker messageBroker;

    @Scheduled(fixedDelay = 5_000)
    @Transactional
    public void publishPending() {
        outboxRepository.findUnpublished(Limit.of(100)).forEach(event -> {
            try {
                messageBroker.publish(event.topic(), event.payload());
                event.markPublished();
            } catch (Exception ex) {
                log.warn("Falha ao publicar evento {}: {}", event.id(), ex.getMessage());
                // próxima execução retenta — evento permanece não publicado
            }
        });
    }
}
```

> O outbox pattern combina `@Transactional` (garantia de persistência) com `@Scheduled` (relay periódico) para eliminar a janela de falha entre escrever no banco e publicar no broker. Em ambiente multi-pod, use `@SchedulerLock` (ShedLock) para que apenas um pod execute o relay por vez.

---

## Referência Rápida

### Propagação de Contexto

| Cenário | Solução |
|---|---|
| Spring Boot 3.x, stack padrão | `TaskDecorator` manual (Security + RequestAttributes + MDC) |
| Spring Boot 3.x com Micrometer Tracing | `MicrometerContextTaskDecorator` (propaga tudo automaticamente) |
| Chains de `CompletableFuture` com Security | `DelegatingSecurityContextAsyncTaskExecutor` como wrapper adicional |
| Fire-and-forget (async outlives request) | Copie os dados necessários em um `record` antes do dispatch |
| Virtual Threads (Java 21+) | `TaskDecorator` — mesmo padrão, evite `InheritableThreadLocal` |

### Timeout e Cancelamento

| Cenário | Solução |
|---|---|
| Limite por operação individual | `.orTimeout()` ou `.completeOnTimeout()` no `CompletableFuture` |
| Interromper a thread ao expirar (I/O-bound) | Resilience4j `TimeLimiter` com `cancel-running-future: true` |
| Interromper computação CPU-bound | `AtomicBoolean` cooperativo verificado entre os passos |
| Limite global da requisição HTTP | `spring.mvc.async.request-timeout` |

### Exceções

| Cenário | Solução |
|---|---|
| Identificar a causa real em `.exceptionally()` | `unwrap(ex)` — verifica `CompletionException` / `ExecutionException` |
| Transformar sucesso e erro juntos | `.handle((result, ex) -> ...)` |
| Logging/métricas sem alterar o resultado | `.whenComplete((result, ex) -> ...)` |
| Exceção em fire-and-forget | `AsyncUncaughtExceptionHandler` ou try/catch interno |

### Pool e Resiliência

| Cenário | Solução |
|---|---|
| Isolamento de cargas distintas | Múltiplos `@Bean` de executor com `@Async("nomeDoExecutor")` |
| Sobrecarga de fila — comportamento desejado | `CallerRunsPolicy` (backpressure) ou `AbortPolicy` (erro explícito) |
| Circuit breaker + retry em chamadas externas | `@CircuitBreaker` / `@Retry` no método síncrono interno, não no `@Async` |
| `@Scheduled` sem sobreposição com jobs longos | `fixedDelay` + `@Async` ou `AtomicBoolean` guard com `fixedRate` |
| `@Cacheable` com retorno `CompletableFuture` | Spring Boot 3.2+: nativo; anterior: bean de cache síncrono separado |

### Quando Escalar

| Sinal | Próximo passo |
|---|---|
| Job > 2–3 min | Mensageria + endpoint de status (polling ou SSE) |
| Retry persistente após reinício | Outbox pattern + broker (RabbitMQ, Kafka) |
| Múltiplos pods | Fila externa compartilhada + `@SchedulerLock` no relay |