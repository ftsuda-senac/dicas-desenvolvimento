# Logs e Observabilidade com Spring Boot

Este documento reune informações e exemplos práticos sobre logging e observabilidade em aplicações Spring Boot 3.x:

- uso do SLF4J como fachada de logging e boas práticas de uso;
- integração com Log4j2 substituindo o Logback padrão;
- configurações recomendadas para ambientes de desenvolvimento e produção;
- uso do MDC (Mapped Diagnostic Context) para enriquecer logs com contexto;
- logs estruturados em JSON para ingestão em sistemas de observabilidade;
- rastreamento distribuído com `@WithSpan` do OpenTelemetry e `@Observed` do Micrometer;
- infraestrutura de observabilidade com Prometheus, OpenTelemetry Collector, Loki, Tempo e Grafana.

Os exemplos seguem um estilo compatível com Spring Boot 3.x e Jakarta EE.

---

## 1. SLF4J — A fachada de logging

### 1.1. O que é o SLF4J

O SLF4J (Simple Logging Facade for Java) é uma **fachada** (API), não uma implementação. O código da aplicação programa contra a API do SLF4J, e em tempo de execução a implementação concreta (Logback, Log4j2, JUL) e escolhida pelo classpath.

```
Código da aplicação
       |
     SLF4J API
       |
  +-----------+-----------+
  |           |           |
Logback    Log4j2       JUL
```

No Spring Boot 3.x o Logback e a implementação padrão. Para trocar por Log4j2 veja a seção 2.

### 1.2. Dependência

O `spring-boot-starter` já inclui o SLF4J transitivamente via `spring-boot-starter-logging`. não é necessário adicionar dependência extra.

### 1.3. Uso básico

Sempre declare o logger como `private static final` e use a classe atual como parâmetro.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class PedidoService {

    private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

    public Pedido criar(CriarPedidoRequest request) {
        log.debug("Iniciando criação de pedido para cliente {}", request.clienteId());

        Pedido pedido = pedidoRepository.save(new Pedido(request));

        log.info("Pedido criado com sucesso: id={}, clienteId={}, total={}",
                pedido.getId(), pedido.getClienteId(), pedido.getTotal());

        return pedido;
    }
}
```

**Regras de nível:**

| Nível   | Quando usar                                                        |
|---------|--------------------------------------------------------------------|
| `TRACE` | Detalhes muito finos, fluxo linha a linha. Nunca em produção.      |
| `DEBUG` | informações de diagnostico. Habilitado em desenvolvimento.         |
| `INFO`  | Eventos de negócio relevantes: pedido criado, usuário autenticado. |
| `WARN`  | Situações suspeitas que não causam falha imediata.                 |
| `ERROR` | Falhas que exigem atenção: exceções não tratadas, dados inválidos. |

### 1.4. Evitar concatenação de strings

O SLF4J usa parametrização lazy — a mensagem só e montada se o nível estiver habilitado.

```java
// Ruim: monta a string mesmo que DEBUG esteja desabilitado
log.debug("Processando item: " + item.toString());

// Correto: parametrização lazy
log.debug("Processando item: {}", item);

// Para objetos custosos, use isDebugEnabled()
if (log.isDebugEnabled()) {
    log.debug("Estado completo: {}", objetoCostoso.toDetalhes());
}
```

### 1.5. Logging de exceções

Passe o `Throwable` sempre como último argumento — o SLF4J imprime o stack trace automaticamente.

```java
try {
    integracaoExterna.enviar(payload);
} catch (IntegracaoException e) {
    log.error("Falha ao enviar payload para integração externa: pedidoId={}", pedidoId, e);
    throw new ProcessamentoException("Falha na integração", e);
}
```

### 1.6. Lombok @Slf4j

Com Lombok, elimine o boilerplate da declaração do logger:

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class PedidoService {

    public void processar(Long pedidoId) {
        log.info("Processando pedido: id={}", pedidoId);
    }
}
```

---

## 2. integração com Log4j2

### 2.1. Por que usar Log4j2

O Log4j2 oferece:

- **Melhor desempenho**: uso de buffers assíncronos e menor contenção.
- **configuração mais rica**: suporte nativo a JSON, YAML, XML e properties.
- **Appenders modernos**: RollingFile, Kafka, JDBC, Syslog, SMTP.
- **Lookup plugins**: resolução de variáveis em tempo de execução.
- **Async loggers**: descarga de I/O para threads de background sem latência.

### 2.2. Dependências

Exclua o Logback e inclua o Log4j2:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>

    <!-- Para async loggers de alto desempenho -->
    <dependency>
        <groupId>com.lmax</groupId>
        <artifactId>disruptor</artifactId>
        <version>3.4.4</version>
    </dependency>
</dependencies>
```

### 2.3. Configuração log4j2.xml — Desenvolvimento

Crie o arquivo em `src/main/resources/log4j2.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">

    <Properties>
        <Property name="LOG_PATTERN">
            %d{yyyy-MM-dd HH:mm:ss.SSS} %highlight{%-5level} [%thread] %cyan{%logger{36}} - %msg%n%throwable
        </Property>
    </Properties>

    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </Console>
    </Appenders>

    <Loggers>
        <Logger name="com.exemplo" level="DEBUG" additivity="false">
            <AppenderRef ref="Console"/>
        </Logger>

        <Logger name="org.springframework" level="INFO" additivity="false">
            <AppenderRef ref="Console"/>
        </Logger>

        <Logger name="org.hibernate.SQL" level="DEBUG" additivity="false">
            <AppenderRef ref="Console"/>
        </Logger>

        <Root level="WARN">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>

</Configuration>
```

### 2.4. Configuração log4j2.xml — produção com JSON e arquivo rotativo

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">

    <Properties>
        <Property name="LOG_DIR">/var/log/app</Property>
        <Property name="APP_NAME">minha-aplicacao</Property>
    </Properties>

    <Appenders>
        <!-- Saída em JSON para coleta pelo agente de logs -->
        <Console name="JsonConsole" target="SYSTEM_OUT">
            <JsonTemplateLayout eventTemplateUri="classpath:log4j2-json-template.json"/>
        </Console>

        <!-- Arquivo rotativo por tamanho e data -->
        <RollingFile name="RollingFile"
                     fileName="${LOG_DIR}/${APP_NAME}.log"
                     filePattern="${LOG_DIR}/${APP_NAME}-%d{yyyy-MM-dd}-%i.log.gz">
            <JsonTemplateLayout eventTemplateUri="classpath:log4j2-json-template.json"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="100MB"/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- Async logger para maior throughput -->
        <AsyncLogger name="com.exemplo" level="INFO" additivity="false">
            <AppenderRef ref="JsonConsole"/>
            <AppenderRef ref="RollingFile"/>
        </AsyncLogger>

        <Root level="WARN">
            <AppenderRef ref="JsonConsole"/>
        </Root>
    </Loggers>

</Configuration>
```

### 2.5. Template JSON personalizado

Crie `src/main/resources/log4j2-json-template.json`:

```json
{
    "@timestamp": {
        "$resolver": "timestamp",
        "pattern": {
            "format": "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",
            "timeZone": "UTC"
        }
    },
    "level": {
        "$resolver": "level",
        "field": "name"
    },
    "thread": {
        "$resolver": "thread",
        "field": "name"
    },
    "logger": {
        "$resolver": "logger",
        "field": "name"
    },
    "message": {
        "$resolver": "message",
        "stringified": true
    },
    "mdc": {
        "$resolver": "mdc"
    },
    "exception": {
        "$resolver": "exception",
        "field": "stackTrace",
        "stackTrace": {
            "stringified": true
        }
    },
    "service": "${APP_NAME:-unknown}"
}
```

### 2.6. Configuração por perfil com Spring

Use `log4j2-spring.xml` (com `-spring`) para habilitar o Spring Profile lookup:

```xml
<!-- src/main/resources/log4j2-spring.xml -->
<Configuration status="WARN">
    <SpringProfile name="dev">
        <!-- configuração de desenvolvimento -->
    </SpringProfile>
    <SpringProfile name="prod">
        <!-- configuração de produção -->
    </SpringProfile>
</Configuration>
```

Ou referencie arquivos distintos no `application.yml`:

```yaml
logging:
  config: classpath:log4j2-${spring.profiles.active}.xml
```

---

## 3. Configurações recomendadas

### 3.1. application.yml — Nível por pacote

```yaml
logging:
  level:
    root: WARN
    com.exemplo: INFO
    com.exemplo.dominio.pedido: DEBUG
    org.springframework.web: INFO
    org.springframework.security: INFO
    org.hibernate.SQL: DEBUG            # SQL gerado pelo Hibernate
    org.hibernate.orm.jdbc.bind: TRACE  # Parâmetros das queries
    io.lettuce.core: WARN
  # Arquivo de saída (Logback padrão)
  file:
    name: /var/log/app/aplicação.log
  pattern:
    console: "%d{HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n"
```

### 3.2. Variáveis de ambiente para produção

Exporte níveis de log sem redeploy:

```bash
# Ajuste temporário em produção via variável de ambiente
LOGGING_LEVEL_COM_EXEMPLO_DOMINIO_PEDIDO=DEBUG java -jar app.jar
```

### 3.3. Ajuste dinâmico via Actuator

Com `spring-boot-starter-actuator` habilitado:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: loggers, health, info
  endpoint:
    loggers:
      enabled: true
```

Altere o nivel em tempo de execucao via HTTP:

```bash
# Ver nível atual de um logger
curl http://localhost:8080/actuator/loggers/com.exemplo.dominio.pedido

# Alterar para DEBUG temporariamente
curl -X POST http://localhost:8080/actuator/loggers/com.exemplo.dominio.pedido \
     -H "Content-Type: application/json" \
     -d '{"configuredLevel": "DEBUG"}'

# Resetar para o nível configurado
curl -X POST http://localhost:8080/actuator/loggers/com.exemplo.dominio.pedido \
     -H "Content-Type: application/json" \
     -d '{"configuredLevel": null}'
```

### 3.4. O que não logar

- Senhas, tokens, números de cartão, dados pessoais (LGPD/GDPR).
- Stack traces completos em nível INFO — use ERROR ou WARN.
- Parâmetros de entrada brutos sem sanitização.

Use uma classe utilitária para mascarar dados sensíveis:

```java
public final class LogMascarador {

    private static final Pattern CARTAO = Pattern.compile("\\d{4}[- ]?\\d{4}[- ]?\\d{4}[- ]?(\\d{4})");
    private static final Pattern CPF   = Pattern.compile("\\d{3}\\.?\\d{3}\\.?\\d{3}-?\\d{2}");

    private LogMascarador() {}

    public static String mascarar(String valor) {
        if (valor == null) return null;
        return CARTAO.matcher(CPF.matcher(valor).replaceAll("***.***.***-**"))
                     .replaceAll("****-****-****-$1");
    }
}

// Uso:
log.info("Processando pagamento: cartao={}", LogMascarador.mascarar(numeroCartao));
```

---

## 4. MDC — Mapped Diagnostic Context

### 4.1. O que é o MDC

O MDC é um mapa de chave-valor associado ao `ThreadLocal` corrente. Valores inseridos no MDC aparecem automaticamente em todos os logs emitidos pela mesma thread, sem precisar passar o contexto explicitamente para cada método.

```
Requisição HTTP
    │
    ├── MDC.put("requestId", "abc-123")
    ├── MDC.put("userId", "user-456")
    │
    ├── PedidoController.criar(...)   → log imprime requestId e userId
    ├── PedidoService.criar(...)      → log imprime requestId e userId
    └── PedidoRepository.save(...)    → log imprime requestId e userId
```

### 4.2. Filter para popular o MDC por requisição

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class MdcFilter extends OncePerRequestFilter {

    private static final Logger log = LoggerFactory.getLogger(MdcFilter.class);

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        String requestId = Optional
                .ofNullable(request.getHeader("X-Request-ID"))
                .orElse(UUID.randomUUID().toString());

        try {
            MDC.put("requestId",  requestId);
            MDC.put("httpMethod", request.getMethod());
            MDC.put("uri",        request.getRequestURI());
            MDC.put("remoteAddr", request.getRemoteAddr());

            response.setHeader("X-Request-ID", requestId);

            log.debug("Requisicao recebida");
            chain.doFilter(request, response);
            log.debug("Requisicao concluida: status={}", response.getStatus());

        } finally {
            MDC.clear(); // SEMPRE limpe o MDC ao final
        }
    }
}
```

### 4.3. Propagar MDC para threads assíncronas

O MDC e vinculado ao `ThreadLocal` — threads filhas não herdam o contexto automaticamente.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setTaskDecorator(new MdcTaskDecorator()); // propaga o MDC
        executor.initialize();
        return executor;
    }
}
```

```java
public class MdcTaskDecorator implements TaskDecorator {

    @Override
    public Runnable decorate(Runnable runnable) {
        Map<String, String> contextMap = MDC.getCopyOfContextMap();
        return () -> {
            try {
                if (contextMap != null) {
                    MDC.setContextMap(contextMap);
                }
                runnable.run();
            } finally {
                MDC.clear();
            }
        };
    }
}
```

### 4.4. Incluir MDC no padrão de log

**Logback (logback-spring.xml):**

```xml
<pattern>%d{HH:mm:ss.SSS} %-5level [%X{requestId}] [%thread] %logger{36} - %msg%n</pattern>
```

**Log4j2 (pattern):**

```
%d{HH:mm:ss.SSS} %-5level [%X{requestId}] [%thread] %logger{36} - %msg%n
```

**Log4j2 (JSON template)** — o campo `mdc` no template da seção 2.5 já inclui todos os valores automaticamente.

### 4.5. Enriquecer o MDC com dados do usuário autenticado

```java
@Component
public class SecurityMdcFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        try {
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth != null && auth.isAuthenticated()
                    && !(auth instanceof AnonymousAuthenticationToken)) {
                MDC.put("userId", auth.getName());
                if (auth.getPrincipal() instanceof UserDetails ud) {
                    // adicione atributos customizados do seu UserDetails
                }
            }
            chain.doFilter(request, response);
        } finally {
            MDC.remove("userId");
        }
    }
}
```

---

## 5. Logs Estruturados em JSON

### 5.1. Por que JSON

Sistemas como Loki, Elasticsearch e Splunk ingerem JSON nativamente. Com logs em texto livre, o parser precisa extrair campos via regex — processo frágil e custoso.

Com JSON:

```json
{
    "@timestamp": "2024-05-15T10:23:45.123Z",
    "level": "INFO",
    "logger": "com.exemplo.dominio.pedido.PedidoService",
    "message": "Pedido criado com sucesso",
    "mdc": {
        "requestId": "abc-123",
        "userId": "user-456",
        "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
        "spanId": "00f067aa0ba902b7"
    },
    "pedidoId": 789,
    "total": 150.00,
    "service": "pedidos-service"
}
```

### 5.2. Adicionar campos customizados ao log

Combine o MDC com um objeto de contexto de log:

```java
@Slf4j
@Service
public class PedidoService {

    public Pedido criar(CriarPedidoRequest request) {
        Pedido pedido = pedidoRepository.save(new Pedido(request));

        // Adiciona campos estruturados ao MDC para este bloco
        MDC.put("pedidoId", String.valueOf(pedido.getId()));
        MDC.put("total", pedido.getTotal().toPlainString());
        try {
            log.info("Pedido criado com sucesso");
        } finally {
            MDC.remove("pedidoId");
            MDC.remove("total");
        }

        return pedido;
    }
}
```

### 5.3. Log4j2 com campos extras via StructuredDataMessage

```java
import org.apache.logging.log4j.message.StringMapMessage;

// StringMapMessage serializa campos diretamente no JSON
logger.info(new StringMapMessage()
        .with("event", "pedido.criado")
        .with("pedidoId", pedido.getId())
        .with("clienteId", pedido.getClienteId())
        .with("total", pedido.getTotal()));
```

### 5.4. Logback com logstash-logback-encoder

Se estiver usando Logback, o `logstash-logback-encoder` e a solução mais popular:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<!-- logback-spring.xml -->
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <includeMdcKeyName>requestId</includeMdcKeyName>
        <includeMdcKeyName>userId</includeMdcKeyName>
        <includeMdcKeyName>traceId</includeMdcKeyName>
        <includeMdcKeyName>spanId</includeMdcKeyName>
        <customFields>{"service":"pedidos-service","environment":"${spring.profiles.active}"}</customFields>
    </encoder>
</appender>
```

Com o encoder, campos estruturados podem ser adicionados via `StructuredArguments`:

```java
import net.logstash.logback.argument.StructuredArguments;

log.info("Pedido criado",
    StructuredArguments.keyValue("pedidoId", pedido.getId()),
    StructuredArguments.keyValue("total", pedido.getTotal()));
```

---

## 6. Rastreamento Distribuído

### 6.1. Dependências

```xml
<dependencies>
    <!-- Micrometer Tracing -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-otel</artifactId>
    </dependency>

    <!-- Exportador OTLP para OpenTelemetry Collector -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-otlp</artifactId>
    </dependency>

    <!-- Propagação de contexto automática no Spring MVC e WebClient -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing</artifactId>
    </dependency>

    <!-- Actuator (expõe métricas para Prometheus) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Prometheus registry -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### 6.2. Configuração

```yaml
management:
  tracing:
    sampling:
      probability: 1.0   # 100% em dev; use 0.1 a 0.5 em produção
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces
    metrics:
      export:
        url: http://localhost:4318/v1/metrics
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  metrics:
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active:default}

spring:
  application:
    name: pedidos-service
```

### 6.3. correlação de traceId com logs

Com `micrometer-tracing`, o `traceId` e `spanId` são propagados automaticamente para o MDC:

```
# saída de log com rastreamento
INFO [pedidos-service,4bf92f3577b34da6,a3ce929d0e0e4736] c.e.d.p.PedidoService - Pedido criado: id=789
#                     ^traceId              ^spanId
```

Para aparecer no padrão de log do Logback:

```xml
<pattern>%d %-5level [%X{traceId},%X{spanId}] %logger{36} - %msg%n</pattern>
```

### 6.4. @WithSpan — Criar spans personalizados

A anotação `@WithSpan` cria um span de rastreamento para o método anotado, visível no Grafana Tempo.

```java
import io.opentelemetry.instrumentation.annotations.WithSpan;
import io.opentelemetry.instrumentation.annotations.SpanAttribute;

@Service
public class PedidoService {

    @WithSpan("pedido.criar")
    public Pedido criar(@SpanAttribute("clienteId") Long clienteId,
                        CriarPedidoRequest request) {
        // Este método aparece como um span no trace
        Pedido pedido = pedidoRepository.save(new Pedido(request));
        return pedido;
    }
}
```

Para adicionar atributos ao span corrente programaticamente:

```java
import io.opentelemetry.api.trace.Span;

@WithSpan("pagamento.processar")
public void processarPagamento(Pedido pedido) {
    Span span = Span.current();
    span.setAttribute("pedidoId", pedido.getId());
    span.setAttribute("total", pedido.getTotal().doubleValue());
    span.setAttribute("formaPagamento", pedido.getFormaPagamento().name());

    // logica de pagamento...
}
```

### 6.5. @Observed — métricas e rastreamento automáticos

A anotação `@Observed` do Micrometer gera automaticamente:
- Um timer (duração da operação)
- Um span de rastreamento
- Um contador de erros

```java
import io.micrometer.observation.annotation.Observed;

@Service
public class PedidoService {

    @Observed(
        name = "pedido.criar",
        contextualName = "criar pedido",
        lowCardinalityKeyValues = {"tipo", "novo"}
    )
    public Pedido criar(CriarPedidoRequest request) {
        return pedidoRepository.save(new Pedido(request));
    }
}
```

Para que `@Observed` funcione, adicione o aspect ao contexto Spring:

```java
@Configuration
public class ObservabilidadeConfig {

    @Bean
    ObservedAspect observedAspect(ObservationRegistry registry) {
        return new ObservedAspect(registry);
    }
}
```

### 6.6. ObservationRegistry para instrumentação manual

```java
@Service
@RequiredArgsConstructor
public class PedidoService {

    private final ObservationRegistry observationRegistry;
    private final PedidoRepository pedidoRepository;

    public Pedido criar(CriarPedidoRequest request) {
        return Observation.createNotStarted("pedido.criar", observationRegistry)
                .lowCardinalityKeyValue("tipo", "novo")
                .highCardinalityKeyValue("clienteId", String.valueOf(request.clienteId()))
                .observe(() -> pedidoRepository.save(new Pedido(request)));
    }
}
```

> **Alta vs Baixa cardinalidade**: use `lowCardinalityKeyValue` para valores com poucas variantes (tipo, status, região) e `highCardinalityKeyValue` para valores de alta variação (IDs) — estes últimos são incluídos apenas no trace, não nas métricas, para evitar explosão de series temporais no Prometheus.

---

## 7. Infraestrutura de Observabilidade

A pilha recomendada para coletar e visualizar logs, métricas e traces:

```
Aplicação Spring Boot
    │
    ├── logs JSON ──────────────► Promtail/Alloy ──► Loki ──────────┐
    │                                                                 │
    ├── métricas (/actuator/prometheus) ◄──── Prometheus scrape      │
    │                                              │                  │
    └── traces (OTLP) ──────────────────► OTel Collector ──► Tempo  │
                                                   │                  │
                                              Prometheus              │
                                                   │                  │
                                               Grafana ◄─────────────┘
                                    (dashboards, alertas, correlação)
```

### 7.1. docker-compose.yml

```yaml
version: "3.9"

services:

  # ── Banco de series temporais para métricas ──────────────────────
  prometheus:
    image: prom/prometheus:v2.51.2
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./observabilidade/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.retention.time=15d"
    networks: [obs]

  # ── Banco de logs ─────────────────────────────────────────────────
  loki:
    image: grafana/loki:2.9.7
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./observabilidade/loki.yml:/etc/loki/config.yaml:ro
      - loki_data:/loki
    command: -config.file=/etc/loki/config.yaml
    networks: [obs]

  # ── Agente de coleta de logs ──────────────────────────────────────
  promtail:
    image: grafana/promtail:2.9.7
    container_name: promtail
    volumes:
      - /var/log:/var/log:ro
      - ./observabilidade/promtail.yml:/etc/promtail/config.yml:ro
    command: -config.file=/etc/promtail/config.yml
    networks: [obs]
    depends_on: [loki]

  # ── Banco de traces distribuídos ──────────────────────────────────
  tempo:
    image: grafana/tempo:2.4.1
    container_name: tempo
    ports:
      - "3200:3200"   # HTTP API
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    volumes:
      - ./observabilidade/tempo.yml:/etc/tempo.yaml:ro
      - tempo_data:/tmp/tempo
    command: -config.file=/etc/tempo.yaml
    networks: [obs]

  # ── OpenTelemetry Collector (roteador central) ────────────────────
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.99.0
    container_name: otel-collector
    ports:
      - "4319:4318"   # OTLP HTTP (recebe da aplicação)
      - "8888:8888"   # métricas internas do collector
    volumes:
      - ./observabilidade/otel-collector.yml:/etc/otelcol/config.yaml:ro
    command: ["--config=/etc/otelcol/config.yaml"]
    networks: [obs]
    depends_on: [tempo, prometheus]

  # ── Grafana (visualização) ────────────────────────────────────────
  grafana:
    image: grafana/grafana:10.4.2
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_FEATURE_TOGGLES_ENABLE: traceqlEditor
    volumes:
      - ./observabilidade/grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml:ro
      - grafana_data:/var/lib/grafana
    networks: [obs]
    depends_on: [prometheus, loki, tempo]

volumes:
  prometheus_data:
  loki_data:
  tempo_data:
  grafana_data:

networks:
  obs:
    driver: bridge
```

### 7.2. observabilidade/prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "pedidos-service"
    metrics_path: "/actuator/prometheus"
    static_configs:
      - targets: ["host.docker.internal:8080"]
    # Para ambientes com service discovery (Kubernetes):
    # kubernetes_sd_configs:
    #   - role: pod

  - job_name: "otel-collector"
    static_configs:
      - targets: ["otel-collector:8888"]
```

### 7.3. observabilidade/loki.yml

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  max_chunk_age: 1h
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  tsdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/index_cache
  filesystem:
    directory: /loki/chunks

limits_config:
  retention_period: 30d
  max_query_series: 10000
```

### 7.4. observabilidade/promtail.yml

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: "app-logs"
    static_configs:
      - targets: ["localhost"]
        labels:
          job: "pedidos-service"
          environment: "dev"
          __path__: /var/log/app/*.log

    pipeline_stages:
      # Interpreta cada linha como JSON
      - json:
          expressions:
            level:    level
            traceId:  "mdc.traceId"
            spanId:   "mdc.spanId"
            userId:   "mdc.userId"
            requestId: "mdc.requestId"

      # Promove campos JSON como labels do Loki
      - labels:
          level:
          traceId:
          spanId:
          userId:

      # Usa o timestamp do proprio log
      - timestamp:
          source: "@timestamp"
          format: RFC3339Milli
```

### 7.5. observabilidade/tempo.yml

```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

ingester:
  max_block_duration: 5m

compactor:
  compaction:
    block_retention: 48h

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/blocks
    wal:
      path: /tmp/tempo/wal
```

### 7.6. observabilidade/otel-collector.yml

```yaml
receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  batch:
    timeout: 5s
    send_batch_size: 1000
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
  resource:
    attributes:
      - action: insert
        key: service.namespace
        value: produção

exporters:
  otlp/tempo:
    endpoint: tempo:4317
    tls:
      insecure: true
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: otel
  debug:
    verbosity: basic

service:
  pipelines:
    traces:
      receivers:  [otlp]
      processors: [memory_limiter, batch, resource]
      exporters:  [otlp/tempo]
    metrics:
      receivers:  [otlp]
      processors: [memory_limiter, batch]
      exporters:  [prometheus]
```

### 7.7. observabilidade/grafana-datasources.yml

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    uid: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true
    jsonData:
      exemplarTraceIdDestinations:
        - name: traceID
          datasourceUid: tempo

  - name: Loki
    type: loki
    uid: loki
    url: http://loki:3100
    access: proxy
    jsonData:
      derivedFields:
        - datasourceUid: tempo
          matcherRegex: '"traceId":"(\w+)"'
          name: TraceID
          url: "$${__value.raw}"

  - name: Tempo
    type: tempo
    uid: tempo
    url: http://tempo:3200
    access: proxy
    jsonData:
      tracesToLogsV2:
        datasourceUid: loki
        filterByTraceID: true
        filterBySpanID: false
      serviceMap:
        datasourceUid: prometheus
      nodeGraph:
        enabled: true
      lokiSearch:
        datasourceUid: loki
```

---

## 8. Correlação entre Logs, métricas e Traces no Grafana

### 8.1. Fluxo de correlação

Com a configuração acima, o Grafana permite navegar entre as três fontes:

1. **métricas (Prometheus)** → detectar anomalia (ex.: latência alta no endpoint `/pedidos`)
2. **Traces (Tempo)** → abrir o trace correspondente ao período de anomalia (exemplares nos gráficos)
3. **Logs (Loki)** → a partir do traceId no Tempo, saltar para os logs relacionados

### 8.2. Exemplares no Prometheus

Habilite o envio de exemplares (liga métricas a traces):

```yaml
management:
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true  # habilita histograma com exemplares
```

### 8.3. LogQL — Consultas no Loki

```logql
# Todos os logs de erro do serviço pedidos-service
{job="pedidos-service"} | json | level = "ERROR"

# Logs de um trace especifico
{job="pedidos-service"} | json | mdc_traceId = "4bf92f3577b34da6a3ce929d0e0e4736"

# Contagem de erros por minuto
sum(rate({job="pedidos-service"} | json | level = "ERROR" [1m])) by (logger)

# Latência extraída dos logs (se logar duração)
{job="pedidos-service"}
    | json
    | level = "INFO"
    | unwrap duracao_ms
    | quantile_over_time(0.95, [5m]) by (uri)
```

### 8.4. TraceQL — Consultas no Tempo

```traceql
# Traces com erro
{ status = error }

# Traces lentos do endpoint de pedidos
{ .http.route = "/pedidos" && duration > 500ms }

# Traces de um usuário especifico
{ .userId = "user-456" }

# Spans com atributo customizado
{ span.pedidoId = "789" }
```

---

## 9. Boas Práticas Consolidadas

### 9.1. Estrutura de mensagem de log

```java
// padrão recomendado: evento + campos relevantes
log.info("Pedido criado: id={}, clienteId={}, total={}, itens={}",
         pedido.getId(), pedido.getClienteId(),
         pedido.getTotal(), pedido.getItens().size());

// Para eventos de negócio críticos, use um campo de evento estruturado
MDC.put("event", "pedido.criado");
log.info("Pedido criado com sucesso");
MDC.remove("event");
```

### 9.2. Não abuse do MDC

O MDC é um mapa global da thread. Abusar pode gerar confusão em fluxos complexos:

```java
// RUIM: MDC com dados de vida longa sem limpeza
MDC.put("pedidoId", pedido.getId().toString()); // e se o método lançar exceção?

// BOM: use try-finally para garantir limpeza
MDC.put("pedidoId", pedido.getId().toString());
try {
    processarPagamento(pedido);
} finally {
    MDC.remove("pedidoId");
}
```

### 9.3. Cheklist de revisão de logging

- [ ] Nenhum dado pessoal ou senha nos logs.
- [ ] Mensagens em inglês ou português — escolha um padrão por projeto.
- [ ] Parâmetros e não concatenação de strings.
- [ ] Throwable passado como último argumento nos logs de erro.
- [ ] MDC limpo ao final de cada requisição/tarefa.
- [ ] Nível correto: DEBUG para diagnostico, INFO para negócio, ERROR para falhas.
- [ ] Logs de entrada e saída em endpoints críticos.
- [ ] Correlation ID (`requestId`, `traceId`) presente em todos os logs.

### 9.4. Health check e métricas customizadas

```java
@Component
public class PedidoMetrics {

    private final Counter pedidosCriados;
    private final Counter pedidosCancelados;
    private final Timer  tempoProcessamento;

    public PedidoMetrics(MeterRegistry registry) {
        pedidosCriados = Counter.builder("pedidos.criados.total")
                .description("Total de pedidos criados")
                .tag("servico", "pedidos")
                .register(registry);

        pedidosCancelados = Counter.builder("pedidos.cancelados.total")
                .description("Total de pedidos cancelados")
                .register(registry);

        tempoProcessamento = Timer.builder("pedidos.processamento.duracao")
                .description("Duracao do processamento de pedido")
                .publishPercentileHistogram()
                .register(registry);
    }

    public void registrarCriacao() {
        pedidosCriados.increment();
    }

    public <T> T medirProcessamento(Supplier<T> operacao) {
        return tempoProcessamento.record(operacao);
    }
}
```

---

## 10. Exemplo completo — PedidoController instrumentado

```java
@RestController
@RequestMapping("/pedidos")
@RequiredArgsConstructor
@Slf4j
public class PedidoController {

    private final PedidoService   pedidoService;
    private final PedidoMetrics   pedidoMetrics;

    @PostMapping
    @Observed(name = "pedido.criar.http", contextualName = "POST /pedidos")
    public ResponseEntity<PedidoResponse> criar(@Valid @RequestBody CriarPedidoRequest request) {

        log.info("Solicitacao de criacao de pedido recebida: clienteId={}", request.clienteId());

        Pedido pedido = pedidoMetrics.medirProcessamento(() -> pedidoService.criar(request));
        pedidoMetrics.registrarCriacao();

        log.info("Pedido retornado ao cliente: id={}", pedido.getId());
        return ResponseEntity.status(HttpStatus.CREATED).body(PedidoResponse.from(pedido));
    }
}
```

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class PedidoService {

    private final PedidoRepository    pedidoRepository;
    private final EstoqueClient       estoqueClient;
    private final ObservationRegistry observationRegistry;

    @WithSpan("pedido.criar.service")
    public Pedido criar(CriarPedidoRequest request) {
        log.debug("Validando estoque para {} itens", request.itens().size());

        estoqueClient.validar(request.itens()); // span filho automatico se usar WebClient/RestClient

        Pedido pedido = pedidoRepository.save(new Pedido(request));

        Span.current().setAttribute("pedidoId", pedido.getId());

        log.info("Pedido salvo: id={}, clienteId={}, total={}",
                pedido.getId(), pedido.getClienteId(), pedido.getTotal());

        return pedido;
    }
}
```

Com essa configuração, cada requisição a `POST /pedidos`:

1. Ganha um `requestId` e `traceId` no MDC (via filtros).
2. Emite um log INFO estruturado em JSON com todos os campos de contexto.
3. Gera métricas de contagem e latência no Prometheus.
4. Registra um trace com spans hierárquicos no Tempo.
5. Permite correlação direta entre logs, métricas e traces no Grafana.