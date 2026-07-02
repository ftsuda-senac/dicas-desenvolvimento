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

### 7.8. Stack ELK — Elasticsearch, Logstash e Kibana

A stack **ELK** é uma alternativa consolidada à stack Loki/Promtail/Grafana para gerenciamento de logs. Enquanto o Loki indexa apenas labels (mais leve e barato), o Elasticsearch indexa o conteúdo completo dos logs, permitindo buscas full-text mais poderosas — ao custo de maior consumo de recursos.

```
Aplicação Spring Boot
    │
    ├── logs JSON ──► Filebeat ──► Logstash ──► Elasticsearch
    │                   (coleta)    (transforma)    (armazena/indexa)
    │                                                     │
    │                                                  Kibana
    │                                          (visualização/dashboards)
    │
    └── Alternativa simplificada:
        logs JSON ──► Filebeat ──────────────► Elasticsearch
                       (sem Logstash)              │
                                                Kibana
```

| Componente | Função | Equivalente na stack Grafana |
|------------|--------|------------------------------|
| **Elasticsearch** | Armazenamento e busca full-text de logs | Loki |
| **Logstash** | Pipeline de ingestão: parsing, enriquecimento, roteamento | Promtail / Alloy |
| **Kibana** | Visualização, dashboards, alertas | Grafana |
| **Filebeat** | Agente leve de coleta de logs (substitui Logstash na coleta) | Promtail |

#### 7.8.1. Quando usar ELK vs Loki/Grafana

| Critério | ELK | Loki + Grafana |
|----------|-----|----------------|
| **Busca full-text** | Excelente (Elasticsearch indexa tudo) | Limitada (indexa labels, filtra texto com grep) |
| **Consumo de recursos** | Alto (RAM e disco) | Baixo (índice compacto) |
| **Custo operacional** | Mais complexo de operar | Mais simples e leve |
| **Escala** | Horizontal com shards/réplicas | Horizontal com chunks/compactor |
| **Dashboards de log** | Kibana (Discover, Lens) | Grafana (Explore, LogQL) |
| **Correlação com métricas/traces** | Requer integração extra (APM, Jaeger) | Nativa com Prometheus e Tempo |
| **Caso de uso ideal** | Logs de múltiplas origens, compliance, auditoria, busca ad-hoc complexa | Microsserviços cloud-native já usando Prometheus/Grafana |

#### 7.8.2. docker-compose.yml — Stack ELK

```yaml
version: "3.9"

services:

  # ── Elasticsearch ─────────────────────────────────────────────────
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.1
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false    # desabilitar em dev; habilitar em produção
      - xpack.security.http.ssl.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m  # ajuste conforme memória disponível
      - cluster.name=observabilidade
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks: [elk]
    healthcheck:
      test: ["CMD-SHELL", "curl -s http://localhost:9200/_cluster/health | grep -q '\"status\":\"green\"\\|\"status\":\"yellow\"'"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ── Logstash ──────────────────────────────────────────────────────
  logstash:
    image: docker.elastic.co/logstash/logstash:8.14.1
    container_name: logstash
    ports:
      - "5044:5044"    # Beats input
      - "5000:5000"    # TCP input
      - "9600:9600"    # API de monitoramento
    volumes:
      - ./elk/logstash/pipeline:/usr/share/logstash/pipeline:ro
      - ./elk/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
    environment:
      - LS_JAVA_OPTS=-Xms256m -Xmx256m
    networks: [elk]
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ── Kibana ────────────────────────────────────────────────────────
  kibana:
    image: docker.elastic.co/kibana/kibana:8.14.1
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - xpack.security.enabled=false
    networks: [elk]
    depends_on:
      elasticsearch:
        condition: service_healthy

  # ── Filebeat (agente de coleta) ───────────────────────────────────
  filebeat:
    image: docker.elastic.co/beats/filebeat:8.14.1
    container_name: filebeat
    user: root
    volumes:
      - ./elk/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/log:/var/log:ro
      - filebeat_data:/usr/share/filebeat/data
    networks: [elk]
    depends_on:
      elasticsearch:
        condition: service_healthy

volumes:
  elasticsearch_data:
  filebeat_data:

networks:
  elk:
    driver: bridge
```

#### 7.8.3. elk/logstash/config/logstash.yml

```yaml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]

pipeline.workers: 2
pipeline.batch.size: 125
pipeline.batch.delay: 50
```

#### 7.8.4. elk/logstash/pipeline/logstash.conf

```ruby
input {
  # Receber logs do Filebeat
  beats {
    port => 5044
  }

  # Receber logs via TCP (alternativa para envio direto da aplicação)
  tcp {
    port => 5000
    codec => json_lines
  }
}

filter {
  # Parsear JSON se o log vier em formato estruturado
  if [message] =~ /^\{/ {
    json {
      source => "message"
      target => "log"
      remove_field => ["message"]
    }
  }

  # Extrair campos do MDC para o nível raiz
  if [log][mdc] {
    ruby {
      code => '
        mdc = event.get("[log][mdc]")
        if mdc.is_a?(Hash)
          mdc.each { |k, v| event.set(k, v) }
        end
      '
    }
  }

  # Enriquecer com informações do host
  mutate {
    add_field => {
      "environment" => "${ENVIRONMENT:dev}"
    }
    remove_field => ["host", "agent", "ecs", "input", "log.offset"]
  }

  # Parsear timestamp do log (se presente)
  if [log][@timestamp] {
    date {
      match => ["[log][@timestamp]", "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", "ISO8601"]
      target => "@timestamp"
    }
  }

  # Anonimizar dados sensíveis
  mutate {
    gsub => [
      "[log][message]", "\d{3}\.\d{3}\.\d{3}-\d{2}", "***.***.***-**",
      "[log][message]", "\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}", "****-****-****-****"
    ]
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{[log][service]}-%{+YYYY.MM.dd}"
    # Em produção com segurança habilitada:
    # user => "${ES_USER}"
    # password => "${ES_PASSWORD}"
  }

  # Debug: descomentar para ver os eventos processados no console
  # stdout { codec => rubydebug }
}
```

#### 7.8.5. elk/filebeat/filebeat.yml

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    json.keys_under_root: true      # campos JSON no nível raiz
    json.add_error_key: true         # adiciona campo de erro se JSON inválido
    json.message_key: message        # campo usado como "message"
    json.overwrite_keys: true

    # Multiline: agrupar stack traces Java
    multiline.pattern: '^\{'
    multiline.negate: true
    multiline.match: after

    fields:
      service: pedidos-service
      environment: dev
    fields_under_root: true

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - drop_fields:
      fields: ["agent.ephemeral_id", "agent.id", "agent.version", "ecs.version"]
      ignore_missing: true

# Enviar para o Logstash
output.logstash:
  hosts: ["logstash:5044"]
  bulk_max_size: 1024

# Alternativa: enviar diretamente ao Elasticsearch (sem Logstash)
# output.elasticsearch:
#   hosts: ["http://elasticsearch:9200"]
#   index: "logs-pedidos-service-%{+yyyy.MM.dd}"

# ILM (Index Lifecycle Management) — gerenciamento automático de retenção
setup.ilm.enabled: true
setup.ilm.rollover_alias: "logs-pedidos-service"
setup.ilm.pattern: "{now/d}-000001"

# Dashboards prontos do Filebeat no Kibana
setup.kibana:
  host: "http://kibana:5601"
setup.dashboards.enabled: true
```

#### 7.8.6. Envio direto da aplicação via Log4j2 (sem Filebeat)

Em cenários onde não é possível usar Filebeat, o Log4j2 pode enviar logs diretamente ao Logstash via TCP/Socket:

```xml
<!-- log4j2-spring.xml — appender para Logstash via TCP -->
<Configuration status="WARN">
    <Appenders>
        <Socket name="Logstash" host="localhost" port="5000" protocol="TCP">
            <JsonTemplateLayout eventTemplateUri="classpath:log4j2-json-template.json"/>
        </Socket>

        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d %-5level [%thread] %logger{36} - %msg%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <AsyncLogger name="com.exemplo" level="INFO" additivity="false">
            <AppenderRef ref="Logstash"/>
            <AppenderRef ref="Console"/>
        </AsyncLogger>

        <Root level="WARN">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

Alternativa com Logback usando o `logstash-logback-encoder`:

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>requestId</includeMdcKeyName>
            <includeMdcKeyName>userId</includeMdcKeyName>
            <includeMdcKeyName>traceId</includeMdcKeyName>
            <includeMdcKeyName>spanId</includeMdcKeyName>
            <customFields>{"service":"pedidos-service"}</customFields>
        </encoder>
        <!-- Reconexão automática -->
        <reconnectionDelay>5 seconds</reconnectionDelay>
        <!-- Buffer em disco para não perder logs se o Logstash cair -->
        <ringBufferSize>8192</ringBufferSize>
    </appender>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d %-5level [%X{requestId}] [%thread] %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="LOGSTASH"/>
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

#### 7.8.7. Index Lifecycle Management (ILM)

O ILM do Elasticsearch gerencia automaticamente a retenção e o ciclo de vida dos índices de log:

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1d"
          },
          "set_priority": { "priority": 100 }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 },
          "set_priority": { "priority": 50 }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": { "priority": 0 },
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

```
Ciclo de vida dos índices:

Hot (0-7d)        Warm (7-30d)       Cold (30-90d)      Delete (90d+)
┌──────────┐     ┌──────────┐       ┌──────────┐       ┌──────────┐
│ Escrita  │     │ Somente  │       │ Congelado│       │ Removido │
│ ativa    │ ──► │ leitura  │  ──►  │ (frozen) │  ──►  │          │
│ SSD      │     │ Compacto │       │ HDD      │       │          │
└──────────┘     └──────────┘       └──────────┘       └──────────┘
```

#### 7.8.8. Consultas no Kibana (KQL e Lucene)

No Kibana, use o **Discover** para explorar logs e o **Lens** para criar dashboards.

**KQL (Kibana Query Language):**

```kql
# Todos os erros do serviço
log.level: "ERROR" and service: "pedidos-service"

# Logs de um trace específico
traceId: "4bf92f3577b34da6a3ce929d0e0e4736"

# Exceções de um tipo específico
log.exception.stackTrace: *NullPointerException*

# Logs de um endpoint com erro
log.logger: *PedidoController* and log.level: "ERROR"

# Busca por intervalo de tempo e usuário
userId: "user-456" and @timestamp >= "2024-05-15T10:00:00" and @timestamp <= "2024-05-15T11:00:00"
```

**Lucene (para buscas mais avançadas):**

```lucene
# Busca fuzzy (tolera erros de digitação)
log.message: pedido~2

# Busca por proximidade
log.message: "pedido criado"~5

# Busca com wildcard e range
log.level: ERROR AND log.message: /Connection.*timeout/
    AND @timestamp:[now-1h TO now]

# Agregação por campo (usado no Lens para dashboards)
service: "pedidos-service" AND NOT log.logger: *actuator*
```

#### 7.8.9. Alertas no Kibana

O Kibana oferece alertas nativos via **Rules** (Stack Management → Rules):

```
Kibana Alerting
    │
    ├── Elasticsearch query rule
    │   └── Dispara quando uma query retorna resultados acima do limiar
    │
    ├── Log threshold rule
    │   └── Dispara quando a contagem de logs atinge um limiar
    │
    └── Actions (conectores)
        ├── Slack
        ├── E-mail
        ├── PagerDuty
        ├── Webhook
        └── Microsoft Teams
```

Exemplo de regra — alertar quando houver mais de 10 erros em 5 minutos:

1. **Stack Management → Rules → Create rule**
2. **Rule type:** Elasticsearch query
3. **Query:**
   ```json
   {
     "bool": {
       "must": [
         { "term": { "log.level": "ERROR" } },
         { "term": { "service": "pedidos-service" } },
         { "range": { "@timestamp": { "gte": "now-5m" } } }
       ]
     }
   }
   ```
4. **Threshold:** quando `count` > 10
5. **Action:** enviar para Slack com mensagem customizada

#### 7.8.10. Elastic APM — Traces com a stack ELK

Para correlacionar logs e traces dentro do ecossistema Elastic (sem depender do Grafana Tempo), use o **Elastic APM**:

```xml
<!-- Dependência do agente APM para Spring Boot -->
<dependency>
    <groupId>co.elastic.apm</groupId>
    <artifactId>apm-agent-attach</artifactId>
    <version>1.49.0</version>
</dependency>
```

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        ElasticApmAttacher.attach();
        SpringApplication.run(Application.class, args);
    }
}
```

```properties
# src/main/resources/elasticapm.properties
service_name=pedidos-service
server_urls=http://localhost:8200
environment=dev
application_packages=com.exemplo
log_ecs_reformatting=SHADE
enable_log_correlation=true
```

Adicione o APM Server ao `docker-compose.yml`:

```yaml
  apm-server:
    image: docker.elastic.co/apm/apm-server:8.14.1
    container_name: apm-server
    ports:
      - "8200:8200"
    environment:
      - output.elasticsearch.hosts=["http://elasticsearch:9200"]
      - apm-server.kibana.host=http://kibana:5601
    networks: [elk]
    depends_on:
      elasticsearch:
        condition: service_healthy
```

Com essa configuração, o Kibana exibe logs, métricas e traces na aba **Observability → APM**, com correlação automática entre os três pilares — equivalente ao que o Grafana oferece com Loki + Prometheus + Tempo.

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

---

## 11. Monitoramento e Alertas

A observabilidade só é completa quando anomalias geram alertas automáticos e dashboards facilitam a investigação. Esta seção cobre a configuração de alertas com Prometheus Alertmanager e Grafana, definição de SLIs/SLOs, dashboards operacionais e boas práticas de resposta a incidentes.

### 11.1. Arquitetura de alertas

```
Prometheus  ──► Alertmanager ──► Canais de notificação
(avalia regras)   (agrupa,         ├── Slack
                   silencia,       ├── E-mail (SMTP)
                   roteia)         ├── PagerDuty / OpsGenie
                                   └── Webhooks

Grafana     ──► Grafana Alerting ──► Contact Points
(regras sobre       (unified         ├── Slack
 qualquer            alerting)       ├── E-mail
 datasource)                         └── Microsoft Teams
```

O Prometheus avalia regras de alerta periodicamente e envia alertas disparados ao **Alertmanager**, que é responsável por agrupar, silenciar, inibir e rotear as notificações. O Grafana também possui seu próprio sistema de alertas (Grafana Alerting), que pode consultar qualquer datasource (Prometheus, Loki, etc.).

### 11.2. Prometheus Alertmanager

#### 11.2.1. Adicionando o Alertmanager ao docker-compose

Adicione ao `docker-compose.yml` da seção 7.1:

```yaml
  alertmanager:
    image: prom/alertmanager:v0.27.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./observabilidade/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    command:
      - "--config.file=/etc/alertmanager/alertmanager.yml"
      - "--storage.path=/alertmanager"
    networks: [obs]
```

Atualize o Prometheus para apontar ao Alertmanager:

```yaml
  prometheus:
    # ... configuração existente ...
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.retention.time=15d"
      - "--web.enable-lifecycle"  # permite reload via POST /-/reload
```

#### 11.2.2. observabilidade/alertmanager.yml

```yaml
global:
  resolve_timeout: 5m
  smtp_from: "alertas@empresa.com"
  smtp_smarthost: "smtp.empresa.com:587"
  smtp_auth_username: "alertas@empresa.com"
  smtp_auth_password: "${SMTP_PASSWORD}"
  smtp_require_tls: true

route:
  receiver: "equipe-dev"
  group_by: ["alertname", "service"]
  group_wait: 30s        # espera para agrupar alertas do mesmo grupo
  group_interval: 5m     # intervalo entre notificações do mesmo grupo
  repeat_interval: 4h    # reenvio se o alerta continuar ativo

  routes:
    - match:
        severity: critical
      receiver: "oncall"
      group_wait: 10s
      repeat_interval: 1h

    - match:
        severity: warning
      receiver: "equipe-dev"
      repeat_interval: 4h

    - match:
        alertname: DeadManSwitch
      receiver: "heartbeat"
      repeat_interval: 1m

receivers:
  - name: "equipe-dev"
    slack_configs:
      - api_url: "${SLACK_WEBHOOK_URL}"
        channel: "#alertas-dev"
        title: '{{ .GroupLabels.alertname }}'
        text: >-
          {{ range .Alerts }}
          *{{ .Labels.severity | toUpper }}* — {{ .Annotations.summary }}
          {{ .Annotations.description }}
          {{ end }}
        send_resolved: true
    email_configs:
      - to: "equipe-dev@empresa.com"
        send_resolved: true

  - name: "oncall"
    slack_configs:
      - api_url: "${SLACK_WEBHOOK_URL}"
        channel: "#oncall"
        title: 'CRITICAL: {{ .GroupLabels.alertname }}'
        send_resolved: true
    pagerduty_configs:
      - service_key: "${PAGERDUTY_SERVICE_KEY}"
        severity: critical

  - name: "heartbeat"
    webhook_configs:
      - url: "http://healthcheck-service:8080/ping"

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ["alertname", "service"]
```

### 11.3. Regras de alerta do Prometheus

#### 11.3.1. Configuração no prometheus.yml

Adicione a referência aos arquivos de regras:

```yaml
# observabilidade/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

rule_files:
  - "/etc/prometheus/rules/*.yml"

scrape_configs:
  # ... configurações existentes ...
```

Monte o diretório de regras no container do Prometheus:

```yaml
  prometheus:
    volumes:
      - ./observabilidade/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./observabilidade/rules:/etc/prometheus/rules:ro
      - prometheus_data:/prometheus
```

#### 11.3.2. observabilidade/rules/aplicacao.yml

```yaml
groups:
  - name: aplicacao.regras
    interval: 15s
    rules:

      # ── Disponibilidade ─────────────────────────────────────────────
      - alert: AplicacaoIndisponivel
        expr: up{job="pedidos-service"} == 0
        for: 1m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Aplicação pedidos-service está indisponível"
          description: >
            A instância {{ $labels.instance }} não responde ao scrape
            do Prometheus há mais de 1 minuto.
          runbook: "https://wiki.empresa.com/runbooks/aplicacao-indisponivel"

      # ── Latência HTTP ───────────────────────────────────────────────
      - alert: LatenciaAltaP95
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_server_requests_seconds_bucket{
              job="pedidos-service",
              uri!~"/actuator.*"
            }[5m])) by (le, uri)
          ) > 2
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Latência P95 acima de 2s no endpoint {{ $labels.uri }}"
          description: >
            O percentil 95 de latência do endpoint {{ $labels.uri }}
            está em {{ $value | printf "%.2f" }}s (limiar: 2s) nos últimos 5 minutos.

      - alert: LatenciaCriticaP99
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_server_requests_seconds_bucket{
              job="pedidos-service",
              uri!~"/actuator.*"
            }[5m])) by (le, uri)
          ) > 5
        for: 3m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Latência P99 acima de 5s no endpoint {{ $labels.uri }}"
          description: >
            O percentil 99 atingiu {{ $value | printf "%.2f" }}s.
            Possível degradação grave de performance.

      # ── Taxa de erros HTTP ──────────────────────────────────────────
      - alert: TaxaErrosAlta
        expr: |
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service",
            status=~"5..",
            uri!~"/actuator.*"
          }[5m]))
          /
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service",
            uri!~"/actuator.*"
          }[5m]))
          > 0.05
        for: 5m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Taxa de erros 5xx acima de 5%"
          description: >
            {{ $value | printf "%.1f" }}% das requisições retornaram
            erro 5xx nos últimos 5 minutos.

      # ── Throughput (queda abrupta) ──────────────────────────────────
      - alert: QuedaDeTrafego
        expr: |
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service"
          }[5m]))
          < 0.1
            and
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service"
          }[5m] offset 1h))
          > 1
        for: 10m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Queda abrupta de tráfego detectada"
          description: >
            O tráfego atual é {{ $value | printf "%.2f" }} req/s,
            mas há 1 hora era significativamente maior.
            Possível problema de rede ou load balancer.

      # ── Dead Man Switch (prova que o Prometheus está funcionando) ───
      - alert: DeadManSwitch
        expr: vector(1)
        labels:
          severity: none
        annotations:
          summary: "Heartbeat do Prometheus — alerta sempre ativo"
```

#### 11.3.3. observabilidade/rules/jvm.yml

```yaml
groups:
  - name: jvm.regras
    interval: 15s
    rules:

      # ── Memória Heap ────────────────────────────────────────────────
      - alert: HeapUsageAlto
        expr: |
          jvm_memory_used_bytes{area="heap", job="pedidos-service"}
          /
          jvm_memory_max_bytes{area="heap", job="pedidos-service"}
          > 0.85
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Uso de heap acima de 85%"
          description: >
            A JVM está usando {{ $value | printf "%.0f" }}% do heap máximo.
            Instância: {{ $labels.instance }}.

      - alert: HeapUsageCritico
        expr: |
          jvm_memory_used_bytes{area="heap", job="pedidos-service"}
          /
          jvm_memory_max_bytes{area="heap", job="pedidos-service"}
          > 0.95
        for: 2m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Uso de heap acima de 95% — risco de OutOfMemoryError"
          description: >
            Heap em {{ $value | printf "%.0f" }}%. Considere
            aumentar -Xmx ou investigar vazamento de memória.

      # ── Garbage Collection ──────────────────────────────────────────
      - alert: GCExcessivo
        expr: |
          sum(rate(jvm_gc_pause_seconds_sum{
            job="pedidos-service"
          }[5m])) by (instance)
          /
          sum(rate(jvm_gc_pause_seconds_count{
            job="pedidos-service"
          }[5m])) by (instance)
          > 0.5
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Tempo médio de pausa do GC acima de 500ms"
          description: >
            A JVM está gastando tempo excessivo em GC.
            Tempo médio de pausa: {{ $value | printf "%.2f" }}s.

      - alert: GCFrequente
        expr: |
          rate(jvm_gc_pause_seconds_count{
            job="pedidos-service",
            action="end of major GC"
          }[5m]) > 0.1
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Full GC muito frequente"
          description: >
            Full GC ocorrendo {{ $value | printf "%.2f" }} vezes/s.
            Investigue vazamento de memória ou aumente o heap.

      # ── Threads ─────────────────────────────────────────────────────
      - alert: ThreadsAlto
        expr: jvm_threads_live_threads{job="pedidos-service"} > 300
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Número de threads acima de 300"
          description: >
            {{ $value }} threads ativas. Possível vazamento de threads
            ou pool mal configurado.

      # ── Connection Pool (HikariCP) ──────────────────────────────────
      - alert: ConnectionPoolEsgotado
        expr: |
          hikaricp_connections_pending{job="pedidos-service"} > 5
        for: 2m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Pool de conexões com fila de espera"
          description: >
            {{ $value }} threads aguardando conexão no HikariCP.
            Possível esgotamento do pool ou queries lentas bloqueando conexões.

      - alert: ConnectionPoolUsageAlto
        expr: |
          hikaricp_connections_active{job="pedidos-service"}
          /
          hikaricp_connections_max{job="pedidos-service"}
          > 0.8
        for: 5m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Uso do connection pool acima de 80%"
          description: >
            {{ $value | printf "%.0f" }}% das conexões estão em uso.
            Considere aumentar o pool ou otimizar queries.
```

#### 11.3.4. observabilidade/rules/negocio.yml

```yaml
groups:
  - name: negocio.regras
    interval: 30s
    rules:

      # ── Métricas de negócio ─────────────────────────────────────────
      - alert: PedidosZeradosPorMuitoTempo
        expr: |
          rate(pedidos_criados_total{servico="pedidos"}[15m]) == 0
            and
          hour() >= 8 and hour() <= 22
        for: 15m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Nenhum pedido criado nos últimos 15 minutos (horário comercial)"
          description: >
            Esperava-se atividade de criação de pedidos entre 8h e 22h
            mas nenhum pedido foi registrado.

      - alert: TaxaCancelamentosAlta
        expr: |
          rate(pedidos_cancelados_total[1h])
          /
          rate(pedidos_criados_total[1h])
          > 0.3
        for: 30m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Taxa de cancelamentos acima de 30%"
          description: >
            {{ $value | printf "%.0f" }}% dos pedidos estão sendo cancelados.
            Investigar possíveis problemas de UX, estoque ou pagamento.
```

### 11.4. Grafana Alerting

O Grafana 10+ oferece **Unified Alerting**, que permite criar regras de alerta diretamente sobre qualquer datasource, sem depender exclusivamente do Prometheus Alertmanager.

#### 11.4.1. Configuração de Contact Points

No `docker-compose.yml`, adicione variáveis ao Grafana:

```yaml
  grafana:
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_FEATURE_TOGGLES_ENABLE: traceqlEditor
      GF_UNIFIED_ALERTING_ENABLED: "true"
      GF_ALERTING_ENABLED: "false"  # desabilita o legacy alerting
```

Configure Contact Points via provisioning:

```yaml
# observabilidade/grafana-alerting.yml
apiVersion: 1

contactPoints:
  - orgId: 1
    name: "slack-equipe-dev"
    receivers:
      - uid: slack-dev
        type: slack
        settings:
          url: "${SLACK_WEBHOOK_URL}"
          recipient: "#alertas-dev"
          title: |
            {{ `{{ .CommonLabels.alertname }}` }}
          text: |
            {{ `{{ range .Alerts }}` }}
            **{{ `{{ .Labels.severity | toUpper }}` }}** — {{ `{{ .Annotations.summary }}` }}
            {{ `{{ .Annotations.description }}` }}
            {{ `{{ end }}` }}

  - orgId: 1
    name: "email-equipe"
    receivers:
      - uid: email-dev
        type: email
        settings:
          addresses: "equipe-dev@empresa.com"

policies:
  - orgId: 1
    receiver: "slack-equipe-dev"
    group_by: ["grafana_folder", "alertname"]
    group_wait: 30s
    group_interval: 5m
    repeat_interval: 4h
    routes:
      - receiver: "email-equipe"
        matchers:
          - severity = critical
        repeat_interval: 1h
```

Monte o arquivo no Grafana:

```yaml
  grafana:
    volumes:
      - ./observabilidade/grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml:ro
      - ./observabilidade/grafana-alerting.yml:/etc/grafana/provisioning/alerting/alerting.yml:ro
      - grafana_data:/var/lib/grafana
```

#### 11.4.2. Alertas sobre Logs (Loki)

O Grafana permite criar alertas diretamente sobre consultas LogQL — algo que o Prometheus sozinho não faz:

```logql
# Alerta: mais de 10 erros por minuto
sum(count_over_time({job="pedidos-service"} | json | level = "ERROR" [1m])) > 10

# Alerta: exceção específica detectada
count_over_time({job="pedidos-service"} |= "OutOfMemoryError" [5m]) > 0

# Alerta: erro de conexão com banco
count_over_time({job="pedidos-service"} |= "ConnectionPoolTimeoutException" [5m]) > 3
```

No Grafana UI: **Alerting → Alert rules → New alert rule** → selecione Loki como datasource e insira a query.

#### 11.4.3. Alertas sobre Traces (Tempo)

```traceql
# Alerta: traces com erro em endpoint crítico
{ .http.route = "/pedidos" && status = error } | count() > 5

# Alerta: latência extrema
{ .http.route = "/pagamentos" && duration > 10s } | count() > 0
```

### 11.5. SLIs, SLOs e Error Budgets

#### 11.5.1. Conceitos

```
SLI (Service Level Indicator)
  └── Métrica que mede o comportamento do serviço
      Ex.: "proporção de requisições com latência < 500ms"

SLO (Service Level Objective)
  └── Meta sobre o SLI
      Ex.: "99.5% das requisições devem ter latência < 500ms em 30 dias"

SLA (Service Level Agreement)
  └── Contrato formal com consequências financeiras
      Ex.: "Garantimos 99.9% de disponibilidade; abaixo disso, créditos são aplicados"

Error Budget
  └── Margem de erro permitida = 1 - SLO
      Ex.: SLO 99.5% → Error Budget = 0.5% → ~3.6h de indisponibilidade/mês
```

| SLO | Error Budget (30 dias) | Significado prático |
|-----|------------------------|---------------------|
| 99% | 7h 12min | ~14 min/dia de margem |
| 99.5% | 3h 36min | ~7 min/dia de margem |
| 99.9% | 43min 12s | ~1.4 min/dia de margem |
| 99.95% | 21min 36s | margem muito apertada |

#### 11.5.2. Regras de SLO no Prometheus

```yaml
# observabilidade/rules/slo.yml
groups:
  - name: slo.regras
    interval: 30s
    rules:

      # ── Recording rules para SLIs ──────────────────────────────────

      # SLI de disponibilidade: proporção de requisições não-5xx
      - record: sli:disponibilidade:rate5m
        expr: |
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service",
            status!~"5..",
            uri!~"/actuator.*"
          }[5m]))
          /
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service",
            uri!~"/actuator.*"
          }[5m]))

      # SLI de latência: proporção de requisições abaixo de 500ms
      - record: sli:latencia:rate5m
        expr: |
          sum(rate(http_server_requests_seconds_bucket{
            job="pedidos-service",
            le="0.5",
            uri!~"/actuator.*"
          }[5m]))
          /
          sum(rate(http_server_requests_seconds_count{
            job="pedidos-service",
            uri!~"/actuator.*"
          }[5m]))

      # ── Alertas de Error Budget ─────────────────────────────────────

      # Burn rate alto: consumindo error budget 14x mais rápido que o normal
      # (em 1h consome o que seria aceitável em ~14h)
      - alert: ErrorBudgetBurnRateAlto
        expr: |
          (1 - sli:disponibilidade:rate5m) > (14 * (1 - 0.995))
        for: 5m
        labels:
          severity: critical
          service: pedidos-service
        annotations:
          summary: "Error budget sendo consumido rapidamente"
          description: >
            A taxa de erro atual consumirá o error budget mensal
            em menos de 3 dias. SLI atual: {{ $value | printf "%.4f" }}.

      # Burn rate moderado: consumindo 3x mais rápido que o normal
      - alert: ErrorBudgetBurnRateModerado
        expr: |
          (1 - sli:disponibilidade:rate5m) > (3 * (1 - 0.995))
        for: 30m
        labels:
          severity: warning
          service: pedidos-service
        annotations:
          summary: "Error budget em consumo moderado"
          description: >
            A taxa de erro está acima do esperado para manter o SLO de 99.5%.
```

### 11.6. Dashboards Operacionais

#### 11.6.1. Método RED para serviços

O método **RED** (Rate, Errors, Duration) é o padrão para dashboards de serviços. Cada serviço deve ter um dashboard com estes três painéis principais:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Pedidos Service — RED                        │
├──────────────────────┬──────────────────────┬──────────────────┤
│   Rate (req/s)       │   Errors (%)         │  Duration (ms)   │
│   ████████▓▓░░       │   ██░░░░░░░░         │  P50: 45ms       │
│   23.5 req/s         │   2.1%               │  P95: 230ms      │
│                      │                      │  P99: 890ms      │
├──────────────────────┴──────────────────────┴──────────────────┤
│                    Status por Endpoint                          │
│  POST /pedidos      ████████████████ 12.3 req/s   1.2% erros   │
│  GET  /pedidos/{id} ██████████ 8.1 req/s          0.1% erros   │
│  GET  /pedidos      ████ 3.1 req/s                0.0% erros   │
└─────────────────────────────────────────────────────────────────┘
```

Queries PromQL para o dashboard RED:

```promql
# Rate — requisições por segundo
sum(rate(http_server_requests_seconds_count{
  job="pedidos-service",
  uri!~"/actuator.*"
}[5m])) by (method, uri)

# Errors — percentual de erros 5xx
sum(rate(http_server_requests_seconds_count{
  job="pedidos-service",
  status=~"5..",
  uri!~"/actuator.*"
}[5m]))
/
sum(rate(http_server_requests_seconds_count{
  job="pedidos-service",
  uri!~"/actuator.*"
}[5m])) * 100

# Duration — percentis de latência
histogram_quantile(0.50,
  sum(rate(http_server_requests_seconds_bucket{
    job="pedidos-service",
    uri!~"/actuator.*"
  }[5m])) by (le))

histogram_quantile(0.95, ...)  # P95
histogram_quantile(0.99, ...)  # P99
```

#### 11.6.2. Método USE para infraestrutura

O método **USE** (Utilization, Saturation, Errors) é ideal para recursos de infraestrutura (CPU, memória, disco, pool de conexões):

```promql
# Utilização de CPU da JVM
system_cpu_usage{job="pedidos-service"}
process_cpu_usage{job="pedidos-service"}

# Utilização de Heap
jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"}

# Saturação do Connection Pool (HikariCP)
hikaricp_connections_active / hikaricp_connections_max

# Fila de espera (saturação)
hikaricp_connections_pending
```

#### 11.6.3. Dashboard de SLO

```
┌─────────────────────────────────────────────────────────────────┐
│                    SLO Dashboard — 30 dias                      │
├───────────────────────────────┬─────────────────────────────────┤
│  Disponibilidade              │  Latência (P95 < 500ms)         │
│  SLO: 99.5%                   │  SLO: 99.0%                     │
│  Atual: 99.72%  ✅            │  Atual: 98.85%  ⚠️              │
│  Error Budget restante: 62%   │  Error Budget restante: 0%      │
│  ████████████████▓▓▓░░░░░░    │  ██████████████████████████████  │
├───────────────────────────────┴─────────────────────────────────┤
│  Histórico de Error Budget (últimos 30 dias)                    │
│  100% ┤██████████████████▓▓▓▓▓▓▓▓▓▓▓░░░░                       │
│   50% ┤                                                         │
│    0% ┤─────────────────────────────────────── dias              │
│       1    5    10    15    20    25    30                       │
└─────────────────────────────────────────────────────────────────┘
```

Queries para o dashboard de SLO:

```promql
# Error budget restante (%) nos últimos 30 dias
(
  1 - (
    (1 - (
      sum_over_time(sli:disponibilidade:rate5m[30d])
      / count_over_time(sli:disponibilidade:rate5m[30d])
    ))
    / (1 - 0.995)
  )
) * 100
```

### 11.7. Métricas customizadas com Micrometer

#### 11.7.1. Gauge para monitorar estado

```java
@Component
public class FilaMetrics {

    private final AtomicInteger filaPendente = new AtomicInteger(0);

    public FilaMetrics(MeterRegistry registry) {
        Gauge.builder("fila.pedidos.pendentes", filaPendente, AtomicInteger::get)
                .description("Quantidade de pedidos na fila de processamento")
                .tag("tipo", "processamento")
                .register(registry);
    }

    public void incrementar() { filaPendente.incrementAndGet(); }
    public void decrementar() { filaPendente.decrementAndGet(); }
}
```

#### 11.7.2. DistributionSummary para valores de negócio

```java
@Component
public class PedidoMetricsAvancado {

    private final DistributionSummary valorPedidos;

    public PedidoMetricsAvancado(MeterRegistry registry) {
        valorPedidos = DistributionSummary.builder("pedidos.valor")
                .description("Distribuição do valor dos pedidos")
                .baseUnit("BRL")
                .publishPercentiles(0.5, 0.75, 0.95, 0.99)
                .publishPercentileHistogram()
                .minimumExpectedValue(10.0)
                .maximumExpectedValue(10000.0)
                .register(registry);
    }

    public void registrarValor(BigDecimal valor) {
        valorPedidos.record(valor.doubleValue());
    }
}
```

#### 11.7.3. FunctionCounter para métricas derivadas

```java
@Component
public class CacheMetrics {

    public CacheMetrics(MeterRegistry registry, CacheManager cacheManager) {
        Cache produtosCache = cacheManager.getCache("produtos");
        if (produtosCache != null) {
            FunctionCounter.builder("cache.hits", produtosCache,
                    c -> c.getNativeCache().stats().hitCount())
                    .tag("cache", "produtos")
                    .register(registry);

            FunctionCounter.builder("cache.misses", produtosCache,
                    c -> c.getNativeCache().stats().missCount())
                    .tag("cache", "produtos")
                    .register(registry);
        }
    }
}
```

### 11.8. Alertas com Spring Boot Actuator

#### 11.8.1. Health checks customizados

Health checks são a base para alertas de disponibilidade em ambientes Kubernetes e de monitoramento:

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(3)) {
                return Health.up()
                        .withDetail("database", "PostgreSQL")
                        .withDetail("validationTime", "< 3s")
                        .build();
            }
        } catch (SQLException e) {
            return Health.down()
                    .withDetail("error", e.getMessage())
                    .build();
        }
        return Health.down().build();
    }
}
```

```java
@Component
public class IntegracaoExternaHealthIndicator implements HealthIndicator {

    private final RestClient restClient;

    public IntegracaoExternaHealthIndicator(RestClient.Builder builder) {
        this.restClient = builder
                .baseUrl("http://servico-externo:8080")
                .build();
    }

    @Override
    public Health health() {
        try {
            ResponseEntity<Void> response = restClient.get()
                    .uri("/health")
                    .retrieve()
                    .toBodilessEntity();

            if (response.getStatusCode().is2xxSuccessful()) {
                return Health.up()
                        .withDetail("servico", "integracao-externa")
                        .build();
            }
            return Health.down()
                    .withDetail("status", response.getStatusCode().value())
                    .build();
        } catch (Exception e) {
            return Health.down()
                    .withDetail("error", e.getMessage())
                    .build();
        }
    }
}
```

#### 11.8.2. Configuração de health groups

```yaml
management:
  endpoint:
    health:
      show-details: when-authorized
      group:
        liveness:
          include: livenessState
        readiness:
          include: readinessState, db, redis
        startup:
          include: livenessState, readinessState
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
```

Em Kubernetes, aponte as probes para os grupos:

```yaml
# deployment.yaml (trecho)
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /actuator/health/startup
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 30
```

### 11.9. Boas Práticas de Monitoramento e Alertas

#### 11.9.1. Sintomas vs Causas

Alerte sobre **sintomas** (o que o usuário percebe), não sobre **causas** (detalhes internos):

| Tipo | Exemplo | Recomendação |
|------|---------|--------------|
| **Sintoma** | "Taxa de erros 5xx acima de 5%" | Alerta principal |
| **Sintoma** | "Latência P95 acima de 2s" | Alerta principal |
| **Causa** | "CPU acima de 80%" | Dashboard, não alerta |
| **Causa** | "Heap acima de 85%" | Alerta com `for` longo |

#### 11.9.2. Reduzir ruído

- Use `for` adequado: evite alertar por picos momentâneos (mínimo 2-5 minutos para warnings).
- Agrupe alertas por serviço no Alertmanager (`group_by`).
- Configure `inhibit_rules` para suprimir warnings quando um critical do mesmo serviço já está ativo.
- Revise alertas trimestralmente: remova os que nunca disparam ou que são ignorados.

#### 11.9.3. Checklist de monitoramento

- [ ] Cada serviço tem dashboard RED (Rate, Errors, Duration).
- [ ] Recursos de infraestrutura monitorados com USE (Utilization, Saturation, Errors).
- [ ] SLIs definidos e SLOs documentados.
- [ ] Error budget rastreado e visível no dashboard.
- [ ] Alertas de severidade `critical` têm runbook vinculado.
- [ ] Alertas testados periodicamente (simular falha e verificar notificação).
- [ ] Health checks de liveness e readiness configurados para Kubernetes.
- [ ] Contact points validados (Slack, e-mail, PagerDuty).
- [ ] Retenção de dados configurada (Prometheus: 15-30d, Loki: 30-90d, Tempo: 7-14d).