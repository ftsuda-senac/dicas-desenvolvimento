# System Design — Guia Abrangente

> **Objetivo:** Cobrir os principais conceitos, padrões e decisões arquiteturais envolvidos no design de sistemas escaláveis, resilientes e mantíveis, com exemplos práticos usando Java e Spring Boot 3.x.

---

## Sumário

1. [Fundamentos de System Design](#1-fundamentos-de-system-design)
2. [Escalabilidade](#2-escalabilidade)
3. [Load Balancing](#3-load-balancing)
4. [Caching](#4-caching)
5. [Banco de Dados — Modelagem e Estratégias](#5-banco-de-dados--modelagem-e-estratégias)
6. [Replicação e Sharding](#6-replicação-e-sharding)
7. [Mensageria e Filas](#7-mensageria-e-filas)
8. [Design de APIs](#8-design-de-apis)
9. [Microsserviços](#9-microsserviços)
10. [Resiliência — Circuit Breaker, Retry e Bulkhead](#10-resiliência--circuit-breaker-retry-e-bulkhead)
11. [CQRS e Event Sourcing](#11-cqrs-e-event-sourcing)
12. [Service Discovery e API Gateway](#12-service-discovery-e-api-gateway)
13. [Teorema CAP e Consistência Eventual](#13-teorema-cap-e-consistência-eventual)
14. [Rate Limiting e Throttling](#14-rate-limiting-e-throttling)
15. [Segurança em Sistemas Distribuídos](#15-segurança-em-sistemas-distribuídos)
16. [Observabilidade em Sistemas Distribuídos](#16-observabilidade-em-sistemas-distribuídos)
17. [Checklist de System Design](#17-checklist-de-system-design)
18. [Estudos de Caso](#18-estudos-de-caso)
    - [18.1 Sistema de Pagamentos](#181-sistema-de-pagamentos)
    - [18.2 Fila de Atendimento com Senha de Chamada](#182-fila-de-atendimento-com-senha-de-chamada)
    - [18.3 Sistema de Notificações Multicanal](#183-sistema-de-notificações-multicanal)
    - [18.4 Sistema de Busca com Elasticsearch](#184-sistema-de-busca-com-elasticsearch)
    - [18.5 Sistema de Agendamento e Reservas](#185-sistema-de-agendamento-e-reservas)
    - [18.6 Upload e Processamento Assíncrono de Arquivos](#186-upload-e-processamento-assíncrono-de-arquivos)
    - [18.7 Plataforma de Vídeos — Envio, Processamento e Streaming](#187-plataforma-de-vídeos--envio-processamento-e-streaming)
    - [18.8 Gestão de Identidade com Keycloak — Multi-Tenant e SSO](#188-gestão-de-identidade-com-keycloak--multi-tenant-e-sso)
    - [18.9 Chat em Tempo Real](#189-chat-em-tempo-real)
    - [18.10 Geolocalização e Rastreamento em Tempo Real](#1810-geolocalização-e-rastreamento-em-tempo-real)
    - [18.11 Feed de Atividades e Timeline](#1811-feed-de-atividades-e-timeline)
    - [18.12 Webhooks para Clientes Externos](#1812-webhooks-para-clientes-externos)
    - [18.13 Relatórios Pesados e Exportação de Dados](#1813-relatórios-pesados-e-exportação-de-dados)
    - [18.14 Carrinho de Compras e Checkout](#1814-carrinho-de-compras-e-checkout)
    - [18.15 Encurtador de URLs](#1815-encurtador-de-urls)
    - [18.16 Chatbot com Integração de Agentes de IA](#1816-chatbot-com-integração-de-agentes-de-ia)
    - [18.17 Rastreabilidade com Blockchain](#1817-rastreabilidade-com-blockchain)
    - [18.18 Rede Social](#1818-rede-social)

---

## 1. Fundamentos de System Design

### 1.1 O que é System Design

System Design é o processo de definir a arquitetura, componentes, módulos, interfaces e dados de um sistema para satisfazer requisitos funcionais e não-funcionais. É a ponte entre os requisitos do negócio e o código que será escrito.

```
Requisitos de Negócio
        │
        ▼
  System Design ──────────────────────────────────────────────────────┐
  (Arquitetura, Componentes, Protocolos, Dados)                       │
        │                                                              │
        ▼                                                              ▼
Requisitos Funcionais                              Requisitos Não-Funcionais
- O que o sistema faz                              - Escalabilidade
- Casos de uso, APIs                               - Disponibilidade
- Fluxos de dados                                  - Latência / Throughput
                                                   - Consistência
                                                   - Segurança
```

### 1.2 Requisitos Funcionais vs. Não-Funcionais

| Categoria | Exemplos |
|-----------|----------|
| **Funcionais** | Cadastrar usuário, realizar pagamento, enviar notificação |
| **Escalabilidade** | Suportar 1 milhão de usuários simultâneos |
| **Disponibilidade** | SLA de 99,99% (≈ 52 min/ano de downtime) |
| **Latência** | Resposta em < 200ms para 95% das requisições (P95) |
| **Throughput** | Processar 50.000 transações por segundo |
| **Consistência** | Dados sempre consistentes ou eventualmente consistentes |
| **Durabilidade** | Nenhuma mensagem ou transação deve ser perdida |
| **Segurança** | Autenticação, autorização, criptografia em trânsito e em repouso |

### 1.3 Estimativas de Capacidade (Back-of-the-envelope)

Antes de desenhar o sistema, estime as ordens de magnitude:

```
Usuários ativos diários (DAU): 10 milhões
Requisições por usuário/dia:   10
Total requisições/dia:         100 milhões
Requisições por segundo (RPS): 100M / 86.400s ≈ 1.160 RPS (média)
Pico (10x médio):              ≈ 11.600 RPS

Tamanho médio do payload:      1 KB
Largura de banda de entrada:   11.600 * 1 KB ≈ 11,3 MB/s
Armazenamento (1 ano):         100M req/dia * 1 KB * 365 ≈ 36,5 TB
```

**Referências de latência para memorizar:**

| Operação | Latência aproximada |
|----------|---------------------|
| Leitura de cache L1 | 1 ns |
| Leitura de cache L2 | 10 ns |
| Acesso à RAM | 100 ns |
| Leitura de SSD (4 KB) | 150 µs |
| Round-trip na mesma região (rede) | 0,5 ms |
| Leitura de HDD (seek) | 10 ms |
| Round-trip entre regiões (ex: SP → NY) | 150 ms |

---

## 2. Escalabilidade

### 2.1 Escalabilidade Vertical vs. Horizontal

```
Vertical (Scale Up)                  Horizontal (Scale Out)
┌─────────────────┐                  ┌───────┐ ┌───────┐ ┌───────┐
│  Servidor maior │                  │  S1   │ │  S2   │ │  S3   │
│  (mais CPU/RAM) │                  │       │ │       │ │       │
└─────────────────┘                  └───────┘ └───────┘ └───────┘
                                          └────────┬────────┘
                                               Load Balancer
```

| Critério | Vertical | Horizontal |
|----------|----------|------------|
| Custo crescimento | Exponencial | Linear |
| Limite superior | Sim (hardware máximo) | Praticamente ilimitado |
| Complexidade | Baixa | Alta (estado distribuído) |
| Ponto único de falha | Sim | Não (com LB) |
| Ideal para | Banco de dados (inicial) | APIs stateless, workers |

### 2.2 Stateless vs. Stateful

Para escalar horizontalmente, as instâncias da aplicação devem ser **stateless**: nenhum estado de sessão é armazenado na memória da instância.

```
❌ Stateful — não escala bem
┌─────────┐        ┌─────────┐
│ Req #1  │──────► │ App S1  │ ← sessão do usuário A fica aqui
│ Req #2  │──────► │ App S1  │   Req #3 no S2 não vai encontrar!
└─────────┘        └─────────┘

✅ Stateless — escala facilmente
┌─────────┐     ┌──────┐     ┌─────────┐
│ Req #1  │────►│  LB  │────►│ App S1  │
│ Req #2  │────►│      │────►│ App S2  │  Estado no Redis
│ Req #3  │────►│      │────►│ App S3  │  (compartilhado)
└─────────┘     └──────┘     └─────────┘
```

**Configuração de sessão distribuída com Spring Session + Redis:**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  session:
    store-type: redis
    timeout: 30m
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: 6379
```

```java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig {
    // Spring Session substitui automaticamente o HttpSession padrão
}
```

### 2.3 Padrão de Auto-Scaling

Em ambientes cloud (AWS, GCP, Azure), a aplicação deve ser projetada para inicializar e encerrar rapidamente (cloud-native). Com Spring Boot:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        // Desativa banner para inicialização mais rápida
        app.setBannerMode(Banner.Mode.OFF);
        // Lazy initialization reduz o tempo de startup
        app.setLazyInitialization(true);
        app.run(args);
    }
}
```

```yaml
spring:
  main:
    lazy-initialization: true  # beans criados somente quando necessários

# Graceful shutdown — aguarda requisições em andamento
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

---

## 3. Load Balancing

### 3.1 Algoritmos de Balanceamento

| Algoritmo | Descrição | Quando usar |
|-----------|-----------|-------------|
| **Round Robin** | Distribui sequencialmente | Instâncias homogêneas |
| **Weighted Round Robin** | Considera peso de cada instância | Instâncias heterogêneas |
| **Least Connections** | Envia para a instância com menos conexões ativas | Requisições de duração variável |
| **IP Hash** | Hash do IP do cliente determina a instância | Necessidade de afinidade de sessão |
| **Random** | Escolha aleatória | Simples e funciona bem em escala |

### 3.2 Camadas de Load Balancing

```
Camada 4 (Transport — TCP/UDP)
  └── Roteamento baseado em IP e porta
  └── Mais rápido, sem inspecionar o conteúdo HTTP
  └── Ex: AWS NLB, HAProxy (modo TCP)

Camada 7 (Application — HTTP/HTTPS)
  └── Roteamento baseado em URL, headers, cookies
  └── Suporta SSL termination, compressão, cache
  └── Ex: AWS ALB, NGINX, Traefik
```

### 3.3 Health Check para Load Balancer

```java
// Spring Boot Actuator expõe /actuator/health automaticamente
// Configurar para incluir dependências críticas
```

```yaml
management:
  endpoint:
    health:
      show-details: when_authorized
      probes:
        enabled: true   # /actuator/health/liveness e /actuator/health/readiness
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
    redis:
      enabled: true
    db:
      enabled: true
```

```java
// Health check customizado
@Component
public class FilaProcessamentoHealthIndicator implements HealthIndicator {

    private final FilaService filaService;

    public FilaProcessamentoHealthIndicator(FilaService filaService) {
        this.filaService = filaService;
    }

    @Override
    public Health health() {
        long pendentes = filaService.contarMensagensPendentes();
        if (pendentes > 10_000) {
            return Health.down()
                    .withDetail("mensagens_pendentes", pendentes)
                    .withDetail("limite", 10_000)
                    .build();
        }
        return Health.up()
                .withDetail("mensagens_pendentes", pendentes)
                .build();
    }
}
```

---

## 4. Caching

### 4.1 Estratégias de Cache

```
Cache-Aside (Lazy Loading) — mais comum
  1. App busca no cache
  2. Se miss: busca no DB, popula cache, retorna dado
  3. Se hit: retorna direto do cache

Write-Through
  1. App escreve no cache E no DB simultaneamente
  2. Cache sempre consistente com DB
  3. Latência de escrita maior

Write-Behind (Write-Back)
  1. App escreve apenas no cache
  2. Cache grava no DB de forma assíncrona
  3. Risco de perda de dados se cache cair

Read-Through
  1. App busca sempre no cache
  2. Cache busca no DB automaticamente em caso de miss
  3. App nunca acessa o DB diretamente
```

### 4.2 Cache com Spring Boot + Caffeine (local) e Redis (distribuído)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<!-- Cache local de alta performance -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
<!-- Cache distribuído -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=5m,recordStats
```

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(Duration.ofMinutes(5))
                .recordStats());
        return manager;
    }
}
```

```java
@Service
public class ProdutoService {

    private final ProdutoRepository repository;

    // Cache-aside: cacheia o resultado da busca
    @Cacheable(value = "produtos", key = "#id", unless = "#result == null")
    public ProdutoResponse buscarPorId(Long id) {
        return repository.findById(id)
                .map(ProdutoResponse::from)
                .orElse(null);
    }

    // Atualiza o cache ao salvar
    @CachePut(value = "produtos", key = "#result.id()")
    public ProdutoResponse salvar(CriarProdutoRequest request) {
        Produto produto = repository.save(new Produto(request));
        return ProdutoResponse.from(produto);
    }

    // Invalida o cache ao deletar
    @CacheEvict(value = "produtos", key = "#id")
    public void deletar(Long id) {
        repository.deleteById(id);
    }

    // Invalida todo o cache de produtos
    @CacheEvict(value = "produtos", allEntries = true)
    @Scheduled(fixedRate = 300_000)
    public void limparCache() {
        // executado a cada 5 minutos
    }
}
```

### 4.3 Cache Distribuído com Redis

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(
                                new GenericJackson2JsonRedisSerializer()));

        Map<String, RedisCacheConfiguration> configs = Map.of(
                "produtos", defaultConfig.entryTtl(Duration.ofMinutes(30)),
                "sessoes",  defaultConfig.entryTtl(Duration.ofHours(1)),
                "config",   defaultConfig.entryTtl(Duration.ofHours(24))
        );

        return RedisCacheManager.builder(factory)
                .cacheDefaults(defaultConfig)
                .withInitialCacheConfigurations(configs)
                .build();
    }
}
```

### 4.4 Cache Stampede (Thundering Herd) — Problema e Solução

```
Problema: Cache expira para uma chave popular.
Centenas de threads vão ao DB simultaneamente.

Solução 1: Mutex Lock (Dog-Pile Lock)
Apenas uma thread recalcula; as demais aguardam.

Solução 2: Probabilistic Early Expiration
Recalcular antes de expirar com probabilidade crescente.
```

```java
@Service
public class CacheComLockService {

    private final StringRedisTemplate redis;
    private final ProdutoRepository repository;
    private static final Duration LOCK_TTL = Duration.ofSeconds(5);

    public ProdutoResponse buscarComLock(Long id) {
        String cacheKey = "produto:" + id;
        String lockKey  = "lock:produto:" + id;

        // Tenta ler do cache
        String cached = redis.opsForValue().get(cacheKey);
        if (cached != null) {
            return deserializar(cached);
        }

        // Tenta adquirir lock
        Boolean locked = redis.opsForValue()
                .setIfAbsent(lockKey, "1", LOCK_TTL);

        if (Boolean.TRUE.equals(locked)) {
            try {
                // Somente quem tem o lock vai ao DB
                ProdutoResponse resultado = repository.findById(id)
                        .map(ProdutoResponse::from)
                        .orElseThrow();
                redis.opsForValue().set(cacheKey, serializar(resultado), Duration.ofMinutes(10));
                return resultado;
            } finally {
                redis.delete(lockKey);
            }
        } else {
            // Aguarda brevemente e tenta novamente do cache
            try { Thread.sleep(50); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            return buscarComLock(id);
        }
    }
}
```

---

## 5. Banco de Dados — Modelagem e Estratégias

### 5.1 SQL vs. NoSQL

```
SQL (Relacional)                    NoSQL
┌──────────────────────────┐        ┌──────────────────────────────────┐
│ PostgreSQL, MySQL,        │        │ Document:  MongoDB, CouchDB       │
│ Oracle, SQL Server        │        │ Key-Value: Redis, DynamoDB        │
│                           │        │ Wide-Col:  Cassandra, HBase       │
│ ✅ ACID garantido          │        │ Graph:     Neo4j, Amazon Neptune   │
│ ✅ Joins e relações        │        │                                   │
│ ✅ Schema rígido           │        │ ✅ Escala horizontal facilmente    │
│ ✅ Consultas complexas     │        │ ✅ Schema flexível                 │
│ ❌ Sharding complexo       │        │ ✅ Alta disponibilidade            │
│ ❌ Escala vertical         │        │ ❌ Consistência eventual           │
└──────────────────────────┘        │ ❌ Sem joins nativos               │
                                    └──────────────────────────────────┘
```

**Critério de escolha:**

| Cenário | Recomendação |
|---------|--------------|
| Transações financeiras, pedidos | SQL (ACID) |
| Perfis de usuário, catálogos | MongoDB (documentos flexíveis) |
| Sessões, rate limiting, cache | Redis (key-value) |
| Feeds de atividade, grafos sociais | Neo4j ou DynamoDB |
| Métricas de séries temporais | TimescaleDB ou InfluxDB |
| Logs e busca full-text | Elasticsearch |

### 5.2 Índices e Performance

```sql
-- Índice simples
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);

-- Índice composto — ordem importa (seletividade maior primeiro)
CREATE INDEX idx_pedidos_status_data ON pedidos(status, criado_em DESC);

-- Índice parcial — apenas registros relevantes
CREATE INDEX idx_pedidos_pendentes ON pedidos(criado_em)
    WHERE status = 'PENDENTE';

-- Índice cobrindo (covering index) — evita acesso à tabela
CREATE INDEX idx_pedidos_cobrindo ON pedidos(cliente_id, status)
    INCLUDE (total, criado_em);
```

```java
// Spring Data JPA — índices via anotações
@Entity
@Table(
    name = "pedidos",
    indexes = {
        @Index(name = "idx_pedidos_cliente",     columnList = "cliente_id"),
        @Index(name = "idx_pedidos_status_data", columnList = "status, criado_em DESC")
    }
)
public class Pedido {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "cliente_id", nullable = false)
    private Long clienteId;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private StatusPedido status;

    @Column(name = "criado_em", nullable = false)
    private Instant criadoEm;
}
```

### 5.3 N+1 Problem e Estratégias de Fetch

```java
// ❌ N+1: 1 query para listar pedidos + N queries para buscar clientes
List<Pedido> pedidos = pedidoRepository.findAll();
pedidos.forEach(p -> System.out.println(p.getCliente().getNome())); // N queries!

// ✅ JOIN FETCH: 1 única query
@Query("SELECT p FROM Pedido p JOIN FETCH p.cliente WHERE p.status = :status")
List<Pedido> findByStatusComCliente(@Param("status") StatusPedido status);

// ✅ EntityGraph: declarativo e reutilizável
@EntityGraph(attributePaths = {"cliente", "itens", "itens.produto"})
List<Pedido> findByStatus(StatusPedido status);

// ✅ DTO Projection: retorna apenas os campos necessários
@Query("""
    SELECT new br.com.exemplo.dto.PedidoResumo(
        p.id, p.criadoEm, p.total, c.nome
    )
    FROM Pedido p
    JOIN p.cliente c
    WHERE p.status = :status
    """)
List<PedidoResumo> findResumoByStatus(@Param("status") StatusPedido status);
```

### 5.4 Connection Pool com HikariCP

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:5432/${DB_NAME:app}
    username: ${DB_USER}
    password: ${DB_PASS}
    hikari:
      maximum-pool-size: 20          # máximo de conexões no pool
      minimum-idle: 5                # mínimo de conexões ociosas
      idle-timeout: 300000           # 5 min: remove conexões ociosas
      connection-timeout: 30000      # 30s: aguarda conexão disponível
      max-lifetime: 1800000          # 30 min: recicla conexões antigas
      keepalive-time: 60000          # ping para evitar timeout do DB
      pool-name: HikariPool-App
      # Fórmula de pool size: connections = (core_count * 2) + effective_spindle_count
      # Para 4 cores com SSD: (4 * 2) + 1 = 9 (use entre 10 e 20)
```

---

## 6. Replicação e Sharding

### 6.1 Replicação de Banco de Dados

```
Primary-Replica (Leader-Follower)

     Escrita          Leitura
       │                │
       ▼                ▼
  ┌─────────┐      ┌─────────┐
  │ Primary │─────►│ Replica │
  │  (R/W)  │      │  (R)    │
  └─────────┘      └─────────┘
                   ┌─────────┐
              ────►│ Replica │
                   │  (R)    │
                   └─────────┘

Vantagens:
- Alta disponibilidade para leituras
- Réplicas podem servir como failover
- Backups sem impactar o Primary

Desvantagem:
- Replication lag: réplica pode estar levemente desatualizada
```

**Roteamento leitura/escrita com Spring:**

```java
public enum DataSourceType { PRIMARY, REPLICA }

public class RoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly()
                ? DataSourceType.REPLICA
                : DataSourceType.PRIMARY;
    }
}
```

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.replica")
    public DataSource replicaDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public DataSource routingDataSource(
            @Qualifier("primaryDataSource") DataSource primary,
            @Qualifier("replicaDataSource") DataSource replica) {

        RoutingDataSource routing = new RoutingDataSource();
        routing.setTargetDataSources(Map.of(
                DataSourceType.PRIMARY, primary,
                DataSourceType.REPLICA, replica
        ));
        routing.setDefaultTargetDataSource(primary);
        return routing;
    }
}
```

```java
@Service
@Transactional
public class PedidoService {

    // Escrita vai para Primary
    public Pedido criar(CriarPedidoRequest req) { ... }

    // Leitura vai para Replica (readOnly = true)
    @Transactional(readOnly = true)
    public List<Pedido> listar() { ... }
}
```

### 6.2 Sharding (Particionamento Horizontal)

```
Sharding por Range:          Sharding por Hash:
usuário_id 1..1M → Shard A   hash(usuário_id) % 4 → Shard 0..3
usuário_id 1M..2M → Shard B
                             ✅ Distribuição uniforme
❌ Hot spots possíveis        ❌ Range queries ineficientes
```

**Sharding lógico com Spring Data JPA:**

```java
@Service
public class ShardRouter {

    private static final int TOTAL_SHARDS = 4;
    private final Map<Integer, PedidoRepository> shards;

    public PedidoRepository resolverShard(Long clienteId) {
        int shard = Math.abs(clienteId.hashCode()) % TOTAL_SHARDS;
        return shards.get(shard);
    }

    public Pedido salvar(Pedido pedido) {
        PedidoRepository repo = resolverShard(pedido.getClienteId());
        return repo.save(pedido);
    }
}
```

---

## 7. Mensageria e Filas

### 7.1 Por que usar Mensageria

```
Sem mensageria (acoplamento síncrono):
  Serviço A ──HTTP──► Serviço B ──HTTP──► Serviço C
  - Falha em C impacta A e B
  - Latência acumulada
  - Impossível escalar independentemente

Com mensageria (desacoplamento assíncrono):
  Serviço A ──publish──► Fila/Tópico ──consume──► Serviço B
                                     ──consume──► Serviço C
  - A não conhece B e C
  - B e C podem cair e se recuperar
  - Backpressure natural
```

### 7.2 Point-to-Point vs. Publish-Subscribe

```
Point-to-Point (Queue):           Publish-Subscribe (Topic):
Producer ──► Queue ──► Consumer   Publisher ──► Topic ──► Sub A
                                                      ──► Sub B
Cada mensagem consumida por        Cada mensagem entregue a
exatamente UM consumidor.          TODOS os assinantes.

Casos: pedidos, pagamentos         Casos: eventos de domínio,
       tarefas de background               notificações
```

### 7.3 Kafka com Spring Boot

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: ${KAFKA_SERVERS:localhost:9092}
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all            # confirmação de todos os réplicas
      retries: 3
      properties:
        enable.idempotence: true  # evita duplicatas em retries
    consumer:
      group-id: ${spring.application.name}
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false  # commit manual para garantir processamento
      properties:
        spring.json.trusted.packages: "br.com.exemplo.*"
    listener:
      ack-mode: MANUAL_IMMEDIATE
```

```java
// Evento de domínio
public record PedidoCriadoEvent(
        Long pedidoId,
        Long clienteId,
        BigDecimal total,
        Instant criadoEm
) {}
```

```java
// Producer
@Service
@Slf4j
public class PedidoEventPublisher {

    private final KafkaTemplate<String, PedidoCriadoEvent> kafka;
    private static final String TOPICO = "pedidos.criados";

    public void publicar(PedidoCriadoEvent evento) {
        kafka.send(TOPICO, evento.pedidoId().toString(), evento)
                .whenComplete((result, ex) -> {
                    if (ex != null) {
                        log.error("Falha ao publicar evento pedidoId={}", evento.pedidoId(), ex);
                    } else {
                        log.info("Evento publicado: pedidoId={}, offset={}",
                                evento.pedidoId(),
                                result.getRecordMetadata().offset());
                    }
                });
    }
}
```

```java
// Consumer com tratamento de erro e DLQ
@Component
@Slf4j
public class PedidoCriadoConsumer {

    private final NotificacaoService notificacaoService;

    @KafkaListener(
            topics = "pedidos.criados",
            groupId = "notificacao-service",
            containerFactory = "kafkaListenerContainerFactory"
    )
    public void processar(
            ConsumerRecord<String, PedidoCriadoEvent> record,
            Acknowledgment ack) {
        try {
            PedidoCriadoEvent evento = record.value();
            log.info("Processando pedidoId={}", evento.pedidoId());
            notificacaoService.notificarCliente(evento);
            ack.acknowledge(); // commit somente após processamento com sucesso
        } catch (Exception e) {
            log.error("Erro ao processar mensagem offset={}", record.offset(), e);
            // Não faz ack — mensagem será reprocessada ou vai para DLQ
            throw e;
        }
    }
}
```

```java
// Configuração de Dead Letter Queue (DLQ) e retry
@Configuration
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object>
    kafkaListenerContainerFactory(
            ConsumerFactory<String, Object> consumerFactory,
            KafkaTemplate<String, Object> template) {

        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory);
        factory.setConcurrency(3); // número de threads por consumer group

        // Retry com backoff exponencial
        var retryTopicConfig = RetryTopicConfiguration.builder()
                .maxAttempts(3)
                .fixedBackOff(1000)
                .exponentialBackoff(1000, 2, 10_000)
                .includeTopic("pedidos.criados")
                .dltSuffix(".dlq")
                .create(template);

        return factory;
    }
}
```

### 7.4 Outbox Pattern — Garantia de Entrega Transacional

```
Problema: salvar no DB e publicar no Kafka em operações separadas
         pode resultar em inconsistência se uma falhar.

Solução Outbox:
1. Na mesma transação DB, persiste a entidade + um registro na tabela outbox.
2. Um processo separado (relay) lê a outbox e publica no Kafka.
3. Marca o registro como publicado.
```

```java
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String aggregateType;    // ex: "Pedido"
    private String aggregateId;      // ex: "123"
    private String eventType;        // ex: "PedidoCriado"

    @Column(columnDefinition = "jsonb")
    private String payload;

    private Instant criadoEm = Instant.now();
    private boolean publicado = false;
    private Instant publicadoEm;
}
```

```java
@Service
@Transactional
public class PedidoService {

    private final PedidoRepository pedidoRepository;
    private final OutboxEventRepository outboxRepository;
    private final ObjectMapper mapper;

    public Pedido criar(CriarPedidoRequest request) {
        // 1. Salva o pedido
        Pedido pedido = pedidoRepository.save(new Pedido(request));

        // 2. Na mesma transação, salva o evento na outbox
        OutboxEvent evento = new OutboxEvent();
        evento.setAggregateType("Pedido");
        evento.setAggregateId(pedido.getId().toString());
        evento.setEventType("PedidoCriado");
        evento.setPayload(mapper.writeValueAsString(PedidoCriadoEvent.from(pedido)));
        outboxRepository.save(evento);

        return pedido; // transação commitada aqui — ambos persistidos ou nenhum
    }
}
```

```java
// Relay que publica eventos da outbox para o Kafka
@Component
@Slf4j
public class OutboxRelay {

    private final OutboxEventRepository outboxRepository;
    private final KafkaTemplate<String, String> kafka;

    @Scheduled(fixedDelay = 500) // a cada 500ms
    @Transactional
    public void processar() {
        List<OutboxEvent> pendentes = outboxRepository.findTop100ByPublicadoFalse();
        for (OutboxEvent evento : pendentes) {
            String topico = "dominio." + evento.getAggregateType().toLowerCase()
                          + "." + evento.getEventType().toLowerCase();
            kafka.send(topico, evento.getAggregateId(), evento.getPayload());
            evento.setPublicado(true);
            evento.setPublicadoEm(Instant.now());
        }
    }
}
```

---

## 8. Design de APIs

### 8.1 REST — Boas Práticas

```
Recursos como substantivos, não verbos:
  ✅ GET    /api/v1/pedidos
  ✅ GET    /api/v1/pedidos/{id}
  ✅ POST   /api/v1/pedidos
  ✅ PUT    /api/v1/pedidos/{id}
  ✅ PATCH  /api/v1/pedidos/{id}
  ✅ DELETE /api/v1/pedidos/{id}
  ❌ POST   /api/v1/criarPedido
  ❌ GET    /api/v1/getPedidos

Relacionamentos:
  GET  /api/v1/pedidos/{id}/itens
  POST /api/v1/pedidos/{id}/itens

Ações que não são CRUD:
  POST /api/v1/pedidos/{id}/cancelar
  POST /api/v1/pedidos/{id}/confirmar
```

**Status HTTP semânticos:**

| Situação | Status |
|----------|--------|
| Criação com sucesso | 201 Created + Location header |
| Operação assíncrona aceita | 202 Accepted |
| Sem conteúdo (DELETE, PUT que não retorna) | 204 No Content |
| Redirecionamento permanente | 301 Moved Permanently |
| Requisição inválida (validação) | 400 Bad Request |
| Token ausente ou inválido | 401 Unauthorized |
| Token válido mas sem permissão | 403 Forbidden |
| Recurso não encontrado | 404 Not Found |
| Método HTTP não suportado | 405 Method Not Allowed |
| Conflito (ex: e-mail duplicado) | 409 Conflict |
| Payload muito grande | 413 Content Too Large |
| Rate limit atingido | 429 Too Many Requests |
| Erro interno do servidor | 500 Internal Server Error |
| Serviço indisponível (manutenção) | 503 Service Unavailable |

```java
@RestController
@RequestMapping("/api/v1/pedidos")
@Validated
public class PedidoController {

    private final PedidoService service;
    private final PedidoMapper mapper;

    @PostMapping
    public ResponseEntity<PedidoResponse> criar(
            @RequestBody @Valid CriarPedidoRequest request) {
        Pedido pedido = service.criar(request);
        URI location = URI.create("/api/v1/pedidos/" + pedido.getId());
        return ResponseEntity.created(location).body(mapper.toResponse(pedido));
    }

    @GetMapping("/{id}")
    public PedidoResponse buscar(@PathVariable Long id) {
        return service.buscar(id);
    }

    @PatchMapping("/{id}/cancelar")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void cancelar(@PathVariable Long id) {
        service.cancelar(id);
    }
}
```

### 8.2 Versionamento de API

```java
// Opção 1: Version na URL (mais visível, mais comum)
@RequestMapping("/api/v1/pedidos")
@RequestMapping("/api/v2/pedidos")

// Opção 2: Version no Header (mais RESTful)
@GetMapping(headers = "X-API-Version=2")

// Opção 3: Content Negotiation (Accept header)
@GetMapping(produces = "application/vnd.empresa.v2+json")
```

### 8.3 Paginação Padronizada

```java
@GetMapping
public Page<PedidoResponse> listar(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "criadoEm,desc") String sort) {

    Pageable pageable = PageableHandlerMethodArgumentResolver
            .createPageable(page, size, sort);
    return service.listar(pageable).map(mapper::toResponse);
}
```

```json
// Resposta paginada
{
  "content": [...],
  "page": {
    "size": 20,
    "number": 0,
    "totalElements": 350,
    "totalPages": 18
  }
}
```

### 8.4 Tratamento de Erros Padronizado (RFC 9457 — Problem Details)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public ProblemDetail handleNotFound(RecursoNaoEncontradoException ex,
                                        HttpServletRequest request) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
                HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Recurso não encontrado");
        problem.setType(URI.create("https://api.empresa.com.br/errors/not-found"));
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Erro de validação");
        List<String> erros = ex.getBindingResult().getFieldErrors()
                .stream()
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .toList();
        problem.setProperty("erros", erros);
        return problem;
    }
}
```

---

## 9. Microsserviços

### 9.1 Quando Usar Microsserviços

```
Monolito Modular              →  Microsserviços
(ponto de partida)               (quando necessário)

Migrar quando:
- Times diferentes precisam implantar de forma independente
- Serviços têm requisitos de escala muito distintos
- Domínios têm modelos de dados completamente separados
- O monolito se tornou difícil de manter e implantar

Não migrar apenas porque:
- É a tendência do mercado
- O time é pequeno (< 10 devs)
- A complexidade operacional é proibitiva
```

### 9.2 Decomposição por Domínio (DDD)

```
Bounded Contexts → Microsserviços

┌──────────────────────────────────────────────────────────────┐
│                    E-commerce Platform                       │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Catálogo   │  │  Pedidos    │  │     Pagamento        │  │
│  │  Service    │  │  Service    │  │     Service          │  │
│  │             │  │             │  │                      │  │
│  │ - Produto   │  │ - Pedido    │  │ - Transação          │  │
│  │ - Categoria │  │ - Item      │  │ - Gateway            │  │
│  │ - Estoque   │  │ - Entrega   │  │ - Reembolso          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Usuários   │  │Notificações │  │    Busca / Search    │  │
│  │  Service    │  │  Service    │  │    Service           │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 9.3 Comunicação entre Serviços

```
Síncrona (HTTP/gRPC):              Assíncrona (Mensageria):
- Simples de implementar           - Desacoplamento total
- Retorno imediato                 - Melhor resiliência
- Dependência em runtime           - Processamento eventual
- Cascata de falhas                - Maior complexidade

Use síncrono quando:               Use assíncrono quando:
  resultado é necessário            operação pode ser eventual
  imediatamente (consulta)          (notificações, relatórios,
                                     eventos de domínio)
```

```java
// Comunicação síncrona com RestClient (Spring Boot 3.2+)
@Service
public class CatalogoClient {

    private final RestClient restClient;

    public CatalogoClient(@Value("${servicos.catalogo.url}") String baseUrl) {
        this.restClient = RestClient.builder()
                .baseUrl(baseUrl)
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();
    }

    public ProdutoResponse buscarProduto(Long produtoId) {
        return restClient.get()
                .uri("/api/v1/produtos/{id}", produtoId)
                .retrieve()
                .onStatus(HttpStatusCode::is4xxClientError,
                        (req, res) -> { throw new ProdutoNaoEncontradoException(produtoId); })
                .body(ProdutoResponse.class);
    }
}
```

### 9.4 Saga Pattern — Transações Distribuídas

```
Problema: Transações ACID não funcionam entre microsserviços.

Saga Orquestrada:           Saga Coreografada:
  ┌──────────────┐            Serviço A publica evento
  │  Orquestrador│            Serviço B escuta e age,
  │  (Saga)      │            depois publica seu evento
  └──────┬───────┘            Serviço C escuta e age...
         │ chama
    S1 → S2 → S3              Sem coordenador central.
                              Mais simples, mais difícil de
    Compensação em caso        rastrear o fluxo completo.
    de falha: S3⁻¹ S2⁻¹ S1⁻¹
```

```java
// Saga de criação de pedido — orquestrada
@Service
@Slf4j
public class CriarPedidoSaga {

    private final EstoqueClient estoqueClient;
    private final PagamentoClient pagamentoClient;
    private final EntregaClient entregaClient;
    private final PedidoRepository pedidoRepository;

    @Transactional
    public Pedido executar(CriarPedidoRequest request) {
        Pedido pedido = null;
        ReservaEstoque reserva = null;
        AutorizacaoPagamento autorizacao = null;

        try {
            // Passo 1: Criar pedido local
            pedido = pedidoRepository.save(new Pedido(request));

            // Passo 2: Reservar estoque
            reserva = estoqueClient.reservar(ReservaRequest.from(pedido));

            // Passo 3: Autorizar pagamento
            autorizacao = pagamentoClient.autorizar(AutorizarRequest.from(pedido));

            // Passo 4: Agendar entrega
            entregaClient.agendar(AgendarEntregaRequest.from(pedido));

            pedido.confirmar();
            return pedidoRepository.save(pedido);

        } catch (Exception e) {
            log.error("Falha na saga de criação de pedido", e);
            // Compensações
            if (autorizacao != null) pagamentoClient.cancelar(autorizacao.id());
            if (reserva != null)     estoqueClient.cancelar(reserva.id());
            if (pedido != null)      pedidoRepository.delete(pedido);
            throw new PedidoNaoPodeSerCriadoException(e);
        }
    }
}
```

---

## 10. Resiliência — Circuit Breaker, Retry e Bulkhead

### 10.1 Padrões de Resiliência

```
Retry:           Tenta novamente após falha transitória.
Timeout:         Limita o tempo de espera de uma operação.
Circuit Breaker: Abre o circuito após N falhas consecutivas.
Bulkhead:        Isola pools de recursos para evitar cascata.
Rate Limiter:    Limita a taxa de chamadas a um serviço.
Fallback:        Retorna resposta padrão quando o serviço falha.
```

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      catalogo-service:
        sliding-window-type: COUNT_BASED
        sliding-window-size: 10
        failure-rate-threshold: 50         # abre com 50% de falhas
        wait-duration-in-open-state: 30s   # aguarda 30s antes de tentar (HALF-OPEN)
        permitted-number-of-calls-in-half-open-state: 3
        register-health-indicator: true

  retry:
    instances:
      catalogo-service:
        max-attempts: 3
        wait-duration: 500ms
        exponential-backoff-multiplier: 2  # 500ms → 1s → 2s
        retry-exceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException

  timelimiter:
    instances:
      catalogo-service:
        timeout-duration: 2s

  bulkhead:
    instances:
      catalogo-service:
        max-concurrent-calls: 20
        max-wait-duration: 100ms
```

```java
@Service
public class ProdutoService {

    private final CatalogoClient catalogoClient;

    // Aplica circuit breaker + retry + timeout em cadeia
    @CircuitBreaker(name = "catalogo-service", fallbackMethod = "produtoFallback")
    @Retry(name = "catalogo-service")
    @TimeLimiter(name = "catalogo-service")
    @Bulkhead(name = "catalogo-service")
    public CompletableFuture<ProdutoResponse> buscarProduto(Long id) {
        return CompletableFuture.supplyAsync(() -> catalogoClient.buscarProduto(id));
    }

    // Fallback: retorna dados em cache ou resposta degradada
    private CompletableFuture<ProdutoResponse> produtoFallback(Long id, Exception ex) {
        log.warn("Circuit breaker aberto para produto {}. Usando fallback.", id, ex);
        return CompletableFuture.completedFuture(ProdutoResponse.indisponivel(id));
    }
}
```

### 10.2 Estados do Circuit Breaker

```
                ┌──────────────────────────────────────────┐
                │                                          │
         ┌──────▼──────┐    N falhas     ┌───────────────┐ │
         │   CLOSED    │───────────────►│     OPEN      │ │
         │ (operando)  │                │ (bloqueado)   │ │
         └─────────────┘                └───────┬───────┘ │
                ▲                               │          │
                │          timeout              │          │
                │          esgotado             ▼          │
                │                    ┌──────────────────┐  │
                └────────────────────│   HALF-OPEN      │──┘
                  (se sucesso)       │ (testando)       │
                                     └──────────────────┘
                                       (se falha → OPEN)
```

---

## 11. CQRS e Event Sourcing

### 11.1 CQRS (Command Query Responsibility Segregation)

```
CQRS separa modelos de leitura e escrita:

     Commands (escrita)           Queries (leitura)
           │                            │
           ▼                            ▼
    ┌─────────────┐             ┌─────────────────┐
    │   Command   │             │   Query Model   │
    │   Handler   │             │   (otimizado    │
    │             │             │    para leitura)│
    └──────┬──────┘             └────────┬────────┘
           │ escreve                     │ lê
           ▼                            ▼
    ┌─────────────┐             ┌─────────────────┐
    │  Write DB   │────sync───►│    Read DB      │
    │ (Postgres)  │  ou async  │ (Elasticsearch  │
    └─────────────┘            │  / Redis / etc) │
                               └─────────────────┘
```

```java
// Command — representa uma intenção de mudança
public record CriarPedidoCommand(
        Long clienteId,
        List<ItemCommand> itens,
        EnderecoCommand enderecoEntrega
) {}

// Command Handler — processa o command
@Component
public class CriarPedidoCommandHandler {

    private final PedidoRepository repository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public Long handle(CriarPedidoCommand cmd) {
        Pedido pedido = Pedido.criar(cmd);
        repository.save(pedido);
        // Publica evento para atualizar o modelo de leitura
        eventPublisher.publishEvent(new PedidoCriadoEvent(pedido));
        return pedido.getId();
    }
}
```

```java
// Query Handler — acessa modelo de leitura otimizado
@Component
public class PedidoQueryHandler {

    private final PedidoReadRepository readRepository; // ex: Elasticsearch ou view desnormalizada

    public PedidoDetalheView handle(BuscarPedidoQuery query) {
        return readRepository.findById(query.pedidoId())
                .orElseThrow(() -> new PedidoNaoEncontradoException(query.pedidoId()));
    }

    public Page<PedidoResumoView> handle(ListarPedidosQuery query) {
        return readRepository.findByCliente(query.clienteId(), query.pageable());
    }
}
```

### 11.2 Event Sourcing

```
Estado tradicional:            Event Sourcing:
Armazena o estado atual.       Armazena a sequência de eventos.

DB: { status: ENTREGUE }       Event Store:
                               1. PedidoCriado    (t=0)
                               2. PagamentoConfirmado (t=1)
                               3. PedidoExpedido  (t=2)
                               4. PedidoEntregue  (t=3)

Estado atual = replay de todos os eventos.
Benefícios: auditoria completa, time-travel, retroatividade.
```

```java
// Aggregate com Event Sourcing
@Aggregate
public class Pedido {

    private Long id;
    private StatusPedido status;
    private List<ItemPedido> itens;
    private final List<DomainEvent> eventos = new ArrayList<>();

    // Construtor privado — estado só muda via eventos
    private Pedido() {}

    // Factory method — gera o evento, não muda estado diretamente
    public static Pedido criar(CriarPedidoCommand cmd) {
        Pedido pedido = new Pedido();
        pedido.apply(new PedidoCriadoEvent(
                UUID.randomUUID(), cmd.clienteId(), cmd.itens(), Instant.now()
        ));
        return pedido;
    }

    public void confirmarPagamento(String transacaoId) {
        if (this.status != StatusPedido.AGUARDANDO_PAGAMENTO) {
            throw new EstadoInvalidoException("Pedido não está aguardando pagamento");
        }
        apply(new PagamentoConfirmadoEvent(this.id, transacaoId, Instant.now()));
    }

    // Aplica o evento — única forma de mudar estado
    private void apply(DomainEvent evento) {
        this.eventos.add(evento);
        handle(evento);
    }

    // Reconstitui estado a partir do evento
    private void handle(PedidoCriadoEvent e) {
        this.id = e.pedidoId();
        this.itens = e.itens().stream().map(ItemPedido::from).toList();
        this.status = StatusPedido.AGUARDANDO_PAGAMENTO;
    }

    private void handle(PagamentoConfirmadoEvent e) {
        this.status = StatusPedido.PAGAMENTO_CONFIRMADO;
    }

    // Reconstitui o aggregate a partir do histórico de eventos
    public static Pedido reconstituir(List<DomainEvent> historico) {
        Pedido pedido = new Pedido();
        historico.forEach(pedido::handle);
        return pedido;
    }

    public List<DomainEvent> getEventosPendentes() {
        return Collections.unmodifiableList(eventos);
    }
}
```

---

## 12. Service Discovery e API Gateway

### 12.1 Service Discovery

```
Problema: Com dezenas de microsserviços escalando dinamicamente,
          como cada serviço sabe o endereço dos outros?

Client-side Discovery:         Server-side Discovery:
  Serviço A consulta           Load Balancer ou API Gateway
  o Registry e chama           consulta o Registry e roteia.
  diretamente.
  Ex: Eureka + Ribbon          Ex: AWS ALB + ECS Service Discovery
```

**Spring Cloud Eureka:**

```xml
<!-- Servidor Eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

```yaml
# application.yml do servidor Eureka
server:
  port: 8761
spring:
  application:
    name: service-registry
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

```xml
<!-- Cliente Eureka (microsserviço) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml do microsserviço
spring:
  application:
    name: pedidos-service
eureka:
  client:
    service-url:
      defaultZone: http://service-registry:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
```

```java
// Load-balanced RestClient usando nome do serviço (não IP)
@Configuration
public class ClientConfig {

    @Bean
    @LoadBalanced  // Spring Cloud LoadBalancer resolve o nome via Eureka
    public RestClient.Builder restClientBuilder() {
        return RestClient.builder();
    }
}

@Service
public class CatalogoClient {

    private final RestClient restClient;

    public CatalogoClient(RestClient.Builder builder) {
        this.restClient = builder
                .baseUrl("http://catalogo-service") // nome no Eureka
                .build();
    }
}
```

### 12.2 API Gateway com Spring Cloud Gateway

```
Clientes (Web, Mobile, Parceiros)
          │
          ▼
  ┌───────────────────┐
  │    API Gateway    │
  │                   │
  │ - Roteamento      │
  │ - Autenticação    │
  │ - Rate Limiting   │
  │ - Observabilidade │
  │ - CORS            │
  └──────┬────────────┘
         │
   ┌─────┼──────────┐
   ▼     ▼          ▼
 [S1]  [S2]  ...  [Sn]
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true           # registra rotas automaticamente via Eureka
          lower-case-service-id: true
      routes:
        - id: pedidos-service
          uri: lb://pedidos-service   # lb:// usa load balancer
          predicates:
            - Path=/api/v1/pedidos/**
          filters:
            - StripPrefix=0
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
            - name: CircuitBreaker
              args:
                name: pedidos-cb
                fallbackUri: forward:/fallback/pedidos

        - id: catalogo-service
          uri: lb://catalogo-service
          predicates:
            - Path=/api/v1/produtos/**
          filters:
            - AddRequestHeader=X-Gateway-Request, true
```

```java
// Filtro global de autenticação JWT no Gateway
@Component
public class JwtAuthFilter implements GlobalFilter, Ordered {

    private final JwtValidator jwtValidator;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().value();

        // Rotas públicas não precisam de JWT
        if (path.startsWith("/api/v1/auth/") || path.startsWith("/actuator/")) {
            return chain.filter(exchange);
        }

        String authHeader = request.getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        String token = authHeader.substring(7);
        return jwtValidator.validar(token)
                .flatMap(claims -> {
                    ServerHttpRequest mutated = request.mutate()
                            .header("X-User-Id", claims.getSubject())
                            .header("X-User-Roles", String.join(",", claims.getRoles()))
                            .build();
                    return chain.filter(exchange.mutate().request(mutated).build());
                })
                .onErrorResume(e -> {
                    exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                    return exchange.getResponse().setComplete();
                });
    }

    @Override
    public int getOrder() { return -1; }
}
```

---

## 13. Teorema CAP e Consistência Eventual

### 13.1 Teorema CAP

```
Um sistema distribuído pode garantir no máximo 2 das 3 propriedades:

        Consistency (C)
        Todos os nós vêem os mesmos dados ao mesmo tempo.
               /\
              /  \
             /    \
            /      \
           /  CAP   \
          /  Theorem \
         /────────────\
        /              \
Availability (A) ──── Partition Tolerance (P)
Toda requisição recebe  O sistema continua
resposta (sem timeout). operando mesmo com
                        falhas de rede.

Em redes reais, P é obrigatório (partições acontecem).
A escolha real é entre C e A durante uma partição.

CP Systems:  HBase, Zookeeper, etcd — consistência acima de disponibilidade.
AP Systems:  Cassandra, DynamoDB, CouchDB — disponibilidade acima de consistência.
CA Systems:  PostgreSQL standalone — consistência + disponibilidade (sem distribuição).
```

### 13.2 Níveis de Consistência

```
Consistência Forte:
  Toda leitura reflete a escrita mais recente.
  Ex: PostgreSQL com leitura do Primary.

Consistência Eventual:
  Todas as réplicas convergem para o mesmo valor, eventualmente.
  Ex: Cassandra, DynamoDB, DNS.

Causal Consistency:
  Operações causalmente relacionadas são vistas na ordem correta.
  Ex: MongoDB com sessões causalmente consistentes.

Read-your-writes:
  Após escrever, você sempre lê o que escreveu.
  Ex: Sessão sticky em réplica.

Monotonic Reads:
  Uma vez que você leu um valor, nunca vê um valor mais antigo.
```

### 13.3 Idempotência — Fundamental em Sistemas Distribuídos

```
Uma operação idempotente produz o mesmo resultado
se executada uma ou múltiplas vezes.

POST /pedidos          → NÃO idempotente (cria um novo pedido a cada vez)
PUT /pedidos/{id}      → Idempotente (substituição total)
DELETE /pedidos/{id}   → Idempotente (deletar o que já foi deletado = 404 ou 204)
```

```java
@RestController
@RequestMapping("/api/v1/pagamentos")
public class PagamentoController {

    private final PagamentoService service;

    // Idempotency Key — cliente gera um UUID único por tentativa
    @PostMapping
    public ResponseEntity<PagamentoResponse> processar(
            @RequestHeader("Idempotency-Key") UUID idempotencyKey,
            @RequestBody @Valid ProcessarPagamentoRequest request) {

        return service.processarComIdempotencia(idempotencyKey, request);
    }
}
```

```java
@Service
@Transactional
public class PagamentoService {

    private final IdempotencyRepository idempotencyRepo;
    private final PagamentoRepository pagamentoRepo;

    public ResponseEntity<PagamentoResponse> processarComIdempotencia(
            UUID idempotencyKey, ProcessarPagamentoRequest request) {

        // Verifica se a requisição já foi processada
        return idempotencyRepo.findById(idempotencyKey)
                .map(registro -> ResponseEntity.ok(registro.getResponse()))
                .orElseGet(() -> {
                    // Processa e salva o resultado
                    PagamentoResponse response = processar(request);
                    idempotencyRepo.save(new IdempotencyRecord(idempotencyKey, response));
                    return ResponseEntity.status(HttpStatus.CREATED).body(response);
                });
    }
}
```

---

## 14. Rate Limiting e Throttling

### 14.1 Algoritmos de Rate Limiting

```
Token Bucket:                   Leaky Bucket:
Tokens adicionados a taxa R.    Fila com vazão constante.
Cada requisição consome 1.      Bursts absorvidos, saída uniforme.
Permite bursts até capacidade.

Fixed Window:                   Sliding Window Log:
Contador por janela fixa.       Log de timestamps.
Ex: 100 req/min.                Mais preciso, mais memória.
Problema: borda da janela.

Sliding Window Counter:
Interpolação entre janelas.
Melhor equilíbrio.
```

### 14.2 Rate Limiting com Bucket4j + Redis

```xml
<dependency>
    <groupId>com.bucket4j</groupId>
    <artifactId>bucket4j-redis</artifactId>
</dependency>
```

```java
@Component
public class RateLimiterService {

    private final RedisTemplate<String, byte[]> redisTemplate;

    // Limite por IP: 100 req/min com burst de 20
    public boolean permitir(String identificador) {
        String chave = "rate:" + identificador;

        ProxyManager<String> proxyManager = Bucket4jRedis.casBasedBuilder(redisTemplate)
                .build();

        BucketConfiguration config = BucketConfiguration.builder()
                .addLimit(Bandwidth.builder()
                        .capacity(100)
                        .refillGreedy(100, Duration.ofMinutes(1))
                        .initialTokens(100)
                        .build())
                .addLimit(Bandwidth.builder()
                        .capacity(20)          // burst: máximo 20 por segundo
                        .refillGreedy(20, Duration.ofSeconds(1))
                        .build())
                .build();

        Bucket bucket = proxyManager.builder().build(chave, () -> config);
        return bucket.tryConsume(1);
    }
}
```

```java
@Component
@Order(1)
public class RateLimitFilter extends OncePerRequestFilter {

    private final RateLimiterService rateLimiter;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {

        String ip = obterIpReal(request);

        if (!rateLimiter.permitir(ip)) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.setHeader("Retry-After", "60");
            response.setHeader("X-RateLimit-Limit", "100");
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.getWriter().write("""
                    {"status":429,"title":"Too Many Requests",
                     "detail":"Limite de requisições atingido. Tente novamente em 60 segundos."}
                    """);
            return;
        }
        chain.doFilter(request, response);
    }

    private String obterIpReal(HttpServletRequest request) {
        String forwarded = request.getHeader("X-Forwarded-For");
        if (forwarded != null && !forwarded.isBlank()) {
            return forwarded.split(",")[0].trim();
        }
        return request.getRemoteAddr();
    }
}
```

---

## 15. Segurança em Sistemas Distribuídos

### 15.1 Autenticação e Autorização com JWT

```
Fluxo OAuth2 + JWT em microsserviços:

  Cliente ──► Auth Service ──► JWT
     │                          │
     └──── JWT no header ───────┤
                                ▼
                         API Gateway valida JWT
                                │
                                ▼
                         Microsserviço recebe
                         claims no header
                         (X-User-Id, X-User-Roles)
```

```yaml
# Microsserviço — valida JWT emitido pelo Authorization Server
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${AUTH_SERVER_URL}/realms/minha-app
          jwk-set-uri: ${AUTH_SERVER_URL}/realms/minha-app/protocol/openid-connect/certs
```

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/actuator/health/**").permitAll()
                        .requestMatchers("/api/v1/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2
                        .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtConverter()))
                )
                .build();
    }

    private JwtAuthenticationConverter jwtConverter() {
        JwtGrantedAuthoritiesConverter grantedConverter = new JwtGrantedAuthoritiesConverter();
        grantedConverter.setAuthoritiesClaimName("roles");
        grantedConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(grantedConverter);
        return converter;
    }
}
```

### 15.2 Segurança na Comunicação entre Serviços (mTLS)

```
mTLS (Mutual TLS): ambos os lados apresentam certificados.
Usado em service mesh (Istio, Linkerd) para autenticação serviço-a-serviço.

  Serviço A ──(certificado A)──► Serviço B
  Serviço A ◄──(certificado B)── Serviço B
  Ambos validam o certificado do outro.
```

### 15.3 Secrets Management

```java
// Nunca coloque secrets em application.yml ou variáveis de ambiente em texto simples.
// Use um gerenciador de secrets: AWS Secrets Manager, HashiCorp Vault, Azure Key Vault.

// Spring Cloud Vault
@Configuration
@EnableConfigurationProperties
public class VaultConfig {
    // application.yml configura spring.cloud.vault.uri e autenticação
    // @Value("${db.password}") injeta automaticamente do Vault
}
```

```yaml
spring:
  config:
    import: vault://
  cloud:
    vault:
      uri: ${VAULT_ADDR:http://vault:8200}
      authentication: KUBERNETES
      kubernetes:
        role: minha-app
      kv:
        enabled: true
        default-context: minha-app
```

---

## 16. Observabilidade em Sistemas Distribuídos

### 16.1 Os Três Pilares da Observabilidade

```
         Logs                Métricas              Traces
    ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
    │ Eventos com  │    │ Medições     │    │ Rastro de uma    │
    │ timestamp e  │    │ numéricas    │    │ requisição entre │
    │ contexto     │    │ ao longo do  │    │ todos os serviços│
    │              │    │ tempo        │    │                  │
    │ "O que       │    │ "Quantos     │    │ "Por que a req.  │
    │  aconteceu?" │    │  erros/s?"   │    │  demorou 3s?"    │
    └──────────────┘    └──────────────┘    └──────────────────┘
         Loki                Prometheus           Jaeger/Tempo
```

### 16.2 Rastreamento Distribuído com OpenTelemetry

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

```yaml
management:
  tracing:
    sampling:
      probability: 0.1  # amostra 10% das requisições em produção

  otlp:
    tracing:
      endpoint: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://otel-collector:4318}/v1/traces
```

```java
@Service
public class PedidoService {

    private final Tracer tracer;
    private final PedidoRepository repository;
    private final CatalogoClient catalogoClient;

    @WithSpan("PedidoService.criar")  // cria um span filho automaticamente
    public Pedido criar(CriarPedidoRequest request) {
        Span span = tracer.currentSpan();
        span.tag("pedido.clienteId", request.clienteId().toString());

        // O trace ID propaga automaticamente para o CatalogoClient via HTTP header
        // e para o Kafka via record header
        Pedido pedido = repository.save(new Pedido(request));

        span.tag("pedido.id", pedido.getId().toString());
        return pedido;
    }
}
```

### 16.3 Métricas com Micrometer + Prometheus

```java
@Service
public class PedidoService {

    private final MeterRegistry meterRegistry;
    private final Counter pedidosCriados;
    private final Counter pedidosFalharam;
    private final Timer latenciaCriacao;

    public PedidoService(MeterRegistry meterRegistry) {
        this.meterRegistry    = meterRegistry;
        this.pedidosCriados   = Counter.builder("pedidos.criados.total")
                .description("Total de pedidos criados com sucesso")
                .register(meterRegistry);
        this.pedidosFalharam  = Counter.builder("pedidos.falhas.total")
                .description("Total de pedidos que falharam")
                .register(meterRegistry);
        this.latenciaCriacao  = Timer.builder("pedidos.criacao.duracao")
                .description("Latência de criação de pedido")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry);
    }

    public Pedido criar(CriarPedidoRequest request) {
        return latenciaCriacao.record(() -> {
            try {
                Pedido pedido = processarCriacao(request);
                pedidosCriados.increment();
                meterRegistry.gauge("pedido.total", pedido.getTotal().doubleValue());
                return pedido;
            } catch (Exception e) {
                pedidosFalharam.increment(Tags.of("motivo", e.getClass().getSimpleName()));
                throw e;
            }
        });
    }
}
```

```yaml
# Expõe métricas no formato Prometheus
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics
  endpoint:
    prometheus:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${APP_ENV:local}
```

### 16.4 Stack de Observabilidade com Docker Compose

```yaml
# docker-compose.observabilidade.yml
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"

  tempo:
    image: grafana/tempo:latest
    ports:
      - "3200:3200"
      - "4317:4317"  # gRPC OTLP
      - "4318:4318"  # HTTP OTLP

  grafana:
    image: grafana/grafana:latest
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/provisioning:/etc/grafana/provisioning

  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    volumes:
      - ./otel-collector.yml:/etc/otel/config.yaml
    ports:
      - "4317:4317"
      - "4318:4318"
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['pedidos-service:8080', 'catalogo-service:8081']
```

---

## 17. Checklist de System Design

### 17.1 Fase de Levantamento de Requisitos

```
□ Requisitos funcionais definidos e priorizados
□ Estimativas de escala calculadas (DAU, RPS, armazenamento)
□ SLA/SLO definidos (disponibilidade, latência P99, RPO/RTO)
□ Restrições identificadas (budget, prazo, tecnologias existentes)
□ Casos de uso de borda mapeados (picos, falhas, migrações)
```

### 17.2 Fase de Design de Alto Nível

```
□ Componentes principais identificados (APIs, DB, Cache, Fila)
□ Fluxo de dados entre componentes documentado
□ Pontos únicos de falha (SPOF) identificados e mitigados
□ Estratégia de particionamento de dados definida
□ Estratégia de cache definida (onde, o quê, TTL, invalidação)
□ Comunicação síncrona vs. assíncrona decidida por operação
□ Protocolo de API definido (REST, gRPC, GraphQL)
□ Estratégia de versionamento de API definida
```

### 17.3 Fase de Design Detalhado

```
□ Schema do banco de dados modelado (índices, constraints)
□ Estratégia de migração de dados definida (Flyway/Liquibase)
□ Modelo de consistência escolhido (forte vs. eventual)
□ Idempotência implementada em operações críticas
□ Outbox Pattern para garantia de entrega transacional
□ Circuit Breaker configurado para dependências externas
□ Rate Limiting definido por endpoint e por cliente
□ Estratégia de paginação padronizada
□ Tratamento de erros padronizado (Problem Details RFC 9457)
```

### 17.4 Fase de Segurança

```
□ Autenticação e autorização implementadas (OAuth2/JWT)
□ Comunicação criptografada (TLS 1.2+) em todos os canais
□ Secrets nunca em código ou variáveis de ambiente em texto (usar Vault)
□ Validação de entrada em todas as fronteiras do sistema
□ Auditoria de operações sensíveis implementada
□ Rate limiting aplicado para prevenção de abuso
□ Política de CORS configurada corretamente
```

### 17.5 Fase de Operações e Observabilidade

```
□ Health checks expostos (/liveness, /readiness)
□ Logs estruturados em JSON com correlation ID
□ Métricas de negócio e técnicas instrumentadas
□ Rastreamento distribuído configurado
□ Alertas definidos para SLOs críticos
□ Dashboards de operação criados
□ Runbook documentado para incidentes comuns
□ Estratégia de backup e recuperação de desastres definida
□ Processo de deploy com zero downtime (blue-green, canary, rolling)
```

### 17.6 Armadilhas Comuns (Anti-Patterns)

| Anti-Pattern | Descrição | Solução |
|--------------|-----------|---------|
| **Distributed Monolith** | Microsserviços que se comunicam sincronamente em cadeia e fazem deploy junto | Desacoplar com mensageria; equipes e deploys independentes |
| **Chatty APIs** | Dezenas de chamadas pequenas entre serviços por requisição | Aggregation pattern; GraphQL; BFF (Backend for Frontend) |
| **Shared Database** | Múltiplos serviços compartilham o mesmo banco de dados | Um banco por serviço; comunicação via API ou eventos |
| **No Bulkhead** | Um serviço lento esgota o pool de threads e derruba todos | Bulkhead pattern; pool de threads separado por dependência |
| **Ignore Idempotency** | Retries causam duplicidade (pagamentos duplos, e-mails repetidos) | Idempotency Key; verificação antes de processar |
| **Synchronous Everything** | Tudo síncrono gera acoplamento e cascata de falhas | Assincronizar operações não críticas com mensageria |
| **No Observability** | Impossível diagnosticar problemas em produção | Logs + métricas + traces desde o início |
| **Premature Optimization** | Otimizações sem dados, antes de ter problemas reais | Meça primeiro; otimize com evidências |

---

## 18. Estudos de Caso

Esta seção aplica os conceitos anteriores em cenários concretos e recorrentes no desenvolvimento de software. Cada estudo de caso apresenta: contexto, requisitos-chave, decisões arquiteturais e implementação com Java e Spring Boot.

---

### 18.1 Sistema de Pagamentos

#### Contexto

Um e-commerce precisa aceitar pagamentos via cartão de crédito, Pix e boleto, integrando-se a um gateway externo (ex: Stripe, PagSeguro, Adyen). O maior risco é cobrar o cliente duas vezes ou perder uma transação aprovada.

#### Requisitos-Chave

```
Funcionais:
  - Processar pagamento (cartão, Pix, boleto)
  - Reembolsar total ou parcialmente
  - Receber confirmação assíncrona do gateway (webhook)
  - Conciliar transações ao final do dia

Não-Funcionais:
  - Nunca cobrar duas vezes o mesmo pedido (idempotência)
  - Tempo de resposta < 3s para cartão, Pix instantâneo
  - Disponibilidade 99,9% — falha de pagamento = perda de venda
  - Auditoria completa de cada tentativa e mudança de status
```

#### Arquitetura

```
  Cliente
    │
    ▼
API Gateway ──JWT──► Pagamento Service
                           │
              ┌────────────┼────────────────┐
              ▼            ▼                ▼
       PostgreSQL     Kafka (eventos)   Gateway Client
       (transações)        │            (Stripe/Pix)
                           │                │
                    ┌──────┴────┐           │ webhook
                    ▼           ▼           ▼
             Notificação   Pedido       Pagamento
             Service       Service      Webhook Handler
```

#### Decisões Arquiteturais

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Idempotência | Idempotency Key no header | Retries do cliente não geram cobranças duplas |
| Garantia de entrega | Outbox Pattern | Transação e evento publicados atomicamente |
| Confirmação assíncrona | Webhook do gateway → fila | Gateway confirma por Pix/boleto minutos depois |
| Auditoria | Event Sourcing na entidade `Transacao` | Histórico completo imutável de cada mudança |
| Consistência | Forte para débito; eventual para notificações | Cobrança errada é inaceitável; e-mail pode atrasar |

#### Implementação

```java
// Entidade com máquina de estados explícita
@Entity
@Table(name = "transacoes")
public class Transacao {

    @Id private UUID id;
    private UUID pedidoId;
    private UUID idempotencyKey;
    private BigDecimal valor;

    @Enumerated(EnumType.STRING)
    private StatusTransacao status; // PENDENTE → PROCESSANDO → APROVADA | RECUSADA | ESTORNADA

    private String gatewayTransacaoId;
    private String gatewayResposta;
    private Instant criadaEm;
    private Instant atualizadaEm;

    public void aprovar(String gatewayId, String resposta) {
        if (this.status != StatusTransacao.PROCESSANDO) {
            throw new TransicaoInvalidaException(this.status, StatusTransacao.APROVADA);
        }
        this.status = StatusTransacao.APROVADA;
        this.gatewayTransacaoId = gatewayId;
        this.gatewayResposta = resposta;
        this.atualizadaEm = Instant.now();
    }

    public void recusar(String motivo) {
        if (this.status != StatusTransacao.PROCESSANDO) {
            throw new TransicaoInvalidaException(this.status, StatusTransacao.RECUSADA);
        }
        this.status = StatusTransacao.RECUSADA;
        this.gatewayResposta = motivo;
        this.atualizadaEm = Instant.now();
    }
}
```

```java
@Service
@Transactional
@Slf4j
public class PagamentoService {

    private final TransacaoRepository transacaoRepo;
    private final OutboxEventRepository outboxRepo;
    private final GatewayClient gatewayClient;

    public TransacaoResponse processar(ProcessarPagamentoRequest request) {
        // 1. Idempotência: retorna resultado anterior se já foi processado
        return transacaoRepo.findByIdempotencyKey(request.idempotencyKey())
                .map(TransacaoResponse::from)
                .orElseGet(() -> criarEProcessar(request));
    }

    private TransacaoResponse criarEProcessar(ProcessarPagamentoRequest request) {
        // 2. Persiste em PENDENTE
        Transacao transacao = new Transacao(request);
        transacaoRepo.save(transacao);

        try {
            // 3. Chama o gateway (com circuit breaker e timeout — ver seção 10)
            transacao.setStatus(StatusTransacao.PROCESSANDO);
            GatewayResponse resp = gatewayClient.cobrar(GatewayRequest.from(transacao));

            // 4. Atualiza status conforme resposta
            if (resp.aprovado()) {
                transacao.aprovar(resp.gatewayId(), resp.mensagem());
            } else {
                transacao.recusar(resp.mensagem());
            }

        } catch (GatewayIndisponivelException e) {
            // Gateway fora do ar: status fica PENDENTE para reprocessamento
            log.warn("Gateway indisponível para transacao={}", transacao.getId(), e);
        }

        transacaoRepo.save(transacao);

        // 5. Outbox: publica evento na mesma transação de banco
        outboxRepo.save(OutboxEvent.pagamento(transacao));

        return TransacaoResponse.from(transacao);
    }
}
```

```java
// Webhook do gateway — Pix e boleto confirmam de forma assíncrona
@RestController
@RequestMapping("/webhooks/gateway")
@Slf4j
public class GatewayWebhookController {

    private final PagamentoWebhookService webhookService;

    @PostMapping
    @ResponseStatus(HttpStatus.OK)
    public void receber(
            @RequestHeader("X-Gateway-Signature") String assinatura,
            @RequestBody String payload) {

        // 1. Valida assinatura HMAC para garantir que veio do gateway
        webhookService.validarAssinatura(payload, assinatura);

        // 2. Processa de forma assíncrona — responde 200 imediatamente
        //    (gateways reenviam o webhook se não receberem 200 em < 5s)
        webhookService.processarAsync(payload);
    }
}
```

```java
@Service
@Slf4j
public class PagamentoWebhookService {

    private final TransacaoRepository transacaoRepo;
    private final HmacValidator hmacValidator;
    private final ApplicationEventPublisher events;

    @Async
    @Transactional
    public void processarAsync(String payload) {
        GatewayWebhookEvent evento = deserializar(payload);

        transacaoRepo.findByGatewayTransacaoId(evento.gatewayId())
                .ifPresentOrElse(
                        t -> {
                            t.aprovar(evento.gatewayId(), evento.mensagem());
                            transacaoRepo.save(t);
                            events.publishEvent(new PagamentoConfirmadoEvent(t));
                            log.info("Pagamento confirmado via webhook: transacaoId={}", t.getId());
                        },
                        () -> log.warn("Webhook recebido para transacao desconhecida: {}", evento.gatewayId())
                );
    }
}
```

---

### 18.2 Fila de Atendimento com Senha de Chamada

#### Contexto

Um sistema de atendimento ao público (farmácia, banco, clínica, cartório) precisa gerenciar senhas, chamar clientes nas guichês e exibir o painel de chamadas em tempo real. Múltiplos atendentes trabalham em paralelo e o painel deve atualizar instantaneamente para todos os monitores.

#### Requisitos-Chave

```
Funcionais:
  - Emitir senha com tipo (Normal, Preferencial, Gestante)
  - Atendente chamar próxima senha do seu tipo/guichê
  - Chamar senha específica (rechamada)
  - Painel exibir últimas chamadas em tempo real
  - Relatório de tempo médio de espera e atendimento

Não-Funcionais:
  - Painel atualiza em < 1s após chamada
  - Suportar centenas de senhas emitidas por hora
  - Funcionar offline (sem internet) em cenário de falha de rede
  - Ordem FIFO garantida por prioridade (Preferencial > Normal)
```

#### Arquitetura

```
  Totem de Emissão          Guichê (Atendente)      Painel (Monitor)
         │                          │                       │
         │ POST /senhas              │ POST /chamadas         │ GET /chamadas (SSE)
         ▼                          ▼                       ▼
  ┌──────────────────────────────────────────────────────────────┐
  │                      Fila Service (Spring Boot)              │
  │                                                              │
  │  ┌─────────────────────────────────────────────────────┐    │
  │  │              Fila em Memória (Redis Sorted Set)      │    │
  │  │   Score = prioridade * 10^12 + timestamp_emissao     │    │
  │  │   Preferencial (score alto) sai antes de Normal      │    │
  │  └─────────────────────────────────────────────────────┘    │
  │                                                              │
  │  ┌──────────────────┐    ┌──────────────────────────────┐   │
  │  │   PostgreSQL     │    │   SSE / WebSocket            │   │
  │  │   (histórico,    │    │   (push de chamadas          │   │
  │  │    relatórios)   │    │    para painéis)             │   │
  │  └──────────────────┘    └──────────────────────────────┘   │
  └──────────────────────────────────────────────────────────────┘
```

#### Decisões Arquiteturais

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Estrutura da fila | Redis Sorted Set por prioridade + timestamp | FIFO dentro de cada prioridade; atomicidade com ZPOPMIN |
| Push para painel | Server-Sent Events (SSE) | Unidirecional servidor→cliente; mais simples que WebSocket para este caso |
| Persistência | PostgreSQL para histórico; Redis para fila ativa | Redis rápido para a fila operacional; Postgres para relatórios |
| Numeração de senhas | Sequência diária por tipo (P001, N001, G001) | Reinicia à meia-noite; fácil de chamar verbalmente |

#### Implementação

```java
// Entidades de domínio
public enum TipoSenha {
    PREFERENCIAL(100),
    GESTANTE(90),
    NORMAL(50);

    private final int prioridade;
    TipoSenha(int prioridade) { this.prioridade = prioridade; }
    public int getPrioridade() { return prioridade; }
}

public record SenhaEmitida(
        String codigo,        // ex: "P042"
        TipoSenha tipo,
        Instant emitidaEm,
        String guicheDestino  // null = qualquer guichê
) {}

public record Chamada(
        String senhacodigo,
        String guiche,
        Instant chamadaEm,
        String atendente
) {}
```

```java
@Service
@Slf4j
public class FilaAtendimentoService {

    private final StringRedisTemplate redis;
    private final SenhaRepository senhaRepo;
    private final SenhaSequenciaService sequencia;
    private final ApplicationEventPublisher events;

    private static final String FILA_KEY = "fila:atendimento";

    // Score garante FIFO por prioridade:
    // senhas preferenciais têm score maior, saem primeiro do Sorted Set
    private double calcularScore(TipoSenha tipo, Instant emissao) {
        // Prioridade ocupa os dígitos mais significativos
        // Timestamp ocupa os menos significativos (invertido para FIFO)
        long maxTs = Long.MAX_VALUE;
        long tsInvertido = maxTs - emissao.toEpochMilli();
        return tipo.getPrioridade() * 1e15 + tsInvertido;
    }

    @Transactional
    public SenhaEmitida emitir(TipoSenha tipo) {
        String codigo = sequencia.proxima(tipo); // ex: "P043", "N102"
        Instant agora = Instant.now();

        // Persiste no banco
        Senha senha = senhaRepo.save(new Senha(codigo, tipo, agora));

        // Enfileira no Redis com score de prioridade
        double score = calcularScore(tipo, agora);
        redis.opsForZSet().add(FILA_KEY, codigo, score);

        log.info("Senha emitida: codigo={}, tipo={}", codigo, tipo);
        return new SenhaEmitida(codigo, tipo, agora, null);
    }

    @Transactional
    public Optional<Chamada> chamarProxima(String guiche, String atendente) {
        // ZPOPMAX retorna e remove atomicamente o elemento de maior score
        Set<String> resultado = redis.opsForZSet().popMax(FILA_KEY, 1)
                .stream()
                .map(ZSetOperations.TypedTuple::getValue)
                .collect(Collectors.toSet());

        if (resultado.isEmpty()) {
            log.debug("Fila vazia para guiche={}", guiche);
            return Optional.empty();
        }

        String codigo = resultado.iterator().next();
        Instant agora = Instant.now();

        // Persiste chamada no banco
        senhaRepo.findByCodigo(codigo).ifPresent(s -> {
            s.chamar(guiche, atendente, agora);
            senhaRepo.save(s);
        });

        Chamada chamada = new Chamada(codigo, guiche, agora, atendente);

        // Publica evento para notificar painéis via SSE
        events.publishEvent(new NovaChamadaEvent(chamada));

        log.info("Senha chamada: codigo={}, guiche={}, atendente={}", codigo, guiche, atendente);
        return Optional.of(chamada);
    }

    public Optional<Chamada> rechamar(String codigo, String guiche, String atendente) {
        Chamada chamada = new Chamada(codigo, guiche, Instant.now(), atendente);
        events.publishEvent(new NovaChamadaEvent(chamada));
        return Optional.of(chamada);
    }

    public long quantidadeNaFila() {
        Long total = redis.opsForZSet().size(FILA_KEY);
        return total != null ? total : 0;
    }
}
```

```java
// SSE — empurra chamadas em tempo real para os painéis
@RestController
@RequestMapping("/api/v1/fila")
public class FilaController {

    private final FilaAtendimentoService filaService;
    private final SseEmitterManager sseManager;

    @PostMapping("/senhas")
    @ResponseStatus(HttpStatus.CREATED)
    public SenhaEmitida emitir(@RequestParam TipoSenha tipo) {
        return filaService.emitir(tipo);
    }

    @PostMapping("/chamadas")
    public ResponseEntity<Chamada> chamar(
            @RequestParam String guiche,
            @RequestParam String atendente) {
        return filaService.chamarProxima(guiche, atendente)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.noContent().build());
    }

    @PostMapping("/chamadas/{codigo}/rechamar")
    public Chamada rechamar(@PathVariable String codigo,
                            @RequestParam String guiche,
                            @RequestParam String atendente) {
        return filaService.rechamar(codigo, guiche, atendente)
                .orElseThrow();
    }

    // Painel se conecta aqui e recebe eventos via SSE
    @GetMapping(value = "/painel", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter conectarPainel() {
        SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);
        sseManager.registrar(emitter);
        return emitter;
    }

    @GetMapping("/status")
    public Map<String, Object> status() {
        return Map.of("na_fila", filaService.quantidadeNaFila());
    }
}
```

```java
// Gerencia conexões SSE ativas e distribui eventos
@Component
@Slf4j
public class SseEmitterManager {

    private final CopyOnWriteArrayList<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    public void registrar(SseEmitter emitter) {
        emitters.add(emitter);
        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));
        emitter.onError(e -> emitters.remove(emitter));
        log.info("Painel conectado. Total de painéis: {}", emitters.size());
    }

    @EventListener
    public void onNovaChamada(NovaChamadaEvent event) {
        List<SseEmitter> mortos = new ArrayList<>();
        for (SseEmitter emitter : emitters) {
            try {
                emitter.send(SseEmitter.event()
                        .name("chamada")
                        .data(event.chamada()));
            } catch (IOException e) {
                mortos.add(emitter);
            }
        }
        emitters.removeAll(mortos);
    }
}
```

```java
// Sequência diária de senhas com Redis
@Service
public class SenhaSequenciaService {

    private final StringRedisTemplate redis;

    public String proxima(TipoSenha tipo) {
        String prefixo = switch (tipo) {
            case PREFERENCIAL -> "P";
            case GESTANTE     -> "G";
            case NORMAL       -> "N";
        };
        String chave = "seq:senha:" + prefixo + ":" + LocalDate.now();

        // Incremento atômico no Redis — sem condição de corrida
        Long numero = redis.opsForValue().increment(chave);
        // Expira à meia-noite + 5 minutos
        redis.expire(chave, Duration.ofHours(24).plusMinutes(5));

        return "%s%03d".formatted(prefixo, numero); // P001, N042, G007
    }
}
```

---

### 18.3 Sistema de Notificações Multicanal

#### Contexto

Uma plataforma precisa enviar notificações via e-mail, SMS e push notification (mobile). Cada usuário tem preferências de canal. O volume pode atingir milhões de notificações por dia durante campanhas ou eventos de negócio (ex: Black Friday).

#### Requisitos-Chave

```
Funcionais:
  - Enviar por e-mail, SMS e push (Android/iOS via FCM/APNS)
  - Respeitar preferências e opt-out do usuário por canal
  - Templates configuráveis com variáveis dinâmicas
  - Agendamento de envio (ex: enviar amanhã às 9h)
  - Relatório de entrega (enviado, entregue, lido, falhou)

Não-Funcionais:
  - Throughput de 10.000 notificações/minuto
  - Retry com backoff exponencial em falhas transitórias
  - Deduplicação: o usuário não recebe a mesma mensagem duas vezes
  - Sem perda de mensagens mesmo com falhas no serviço
```

#### Arquitetura

```
  Serviços do Domínio          Notificação Service           Provedores
  (Pedidos, Pagamento...)             │
         │                           │
         │ evento de domínio         ▼
         └─────────────► ┌──────────────────────┐
                         │  Kafka: notificacoes  │
                         └──────────┬───────────┘
                                    │ consumer group
                                    ▼
                         ┌──────────────────────┐
                         │  NotificacaoService  │
                         │  - resolve template  │
                         │  - verifica prefs    │
                         │  - deduplicação      │
                         └──────┬───────────────┘
                                │
               ┌────────────────┼────────────────┐
               ▼                ▼                 ▼
        ┌──────────┐    ┌──────────────┐   ┌───────────┐
        │  E-mail  │    │     SMS      │   │   Push    │
        │  Worker  │    │    Worker    │   │  Worker   │
        │(JavaMail)│    │  (Twilio/   │   │  (FCM/    │
        └──────────┘    │  AWS SNS)   │   │   APNS)   │
                        └─────────────┘   └───────────┘
```

```java
// Comando unificado de notificação
public record EnviarNotificacaoCommand(
        UUID destinatarioId,
        String templateId,           // ex: "pedido-confirmado"
        Map<String, String> variaveis,
        Set<CanalNotificacao> canaisPreferidos,
        Instant agendarPara          // null = imediato
) {}

public enum CanalNotificacao { EMAIL, SMS, PUSH }
```

```java
@Service
@Slf4j
public class NotificacaoOrquestradorService {

    private final UsuarioPreferenciasClient preferenciasClient;
    private final TemplateService templateService;
    private final DeduplicacaoService dedup;
    private final Map<CanalNotificacao, CanalNotificacaoService> canais;

    @KafkaListener(topics = "notificacoes.pendentes", groupId = "notificacao-service")
    public void processar(EnviarNotificacaoCommand cmd, Acknowledgment ack) {
        try {
            // 1. Deduplicação — evita reenvio por retry do Kafka
            if (dedup.jaProcessado(cmd.destinatarioId(), cmd.templateId())) {
                log.info("Notificação duplicada ignorada: usuario={}, template={}",
                        cmd.destinatarioId(), cmd.templateId());
                ack.acknowledge();
                return;
            }

            // 2. Busca preferências do usuário
            PreferenciasUsuario prefs = preferenciasClient.buscar(cmd.destinatarioId());

            // 3. Resolve canais ativos (interseção entre pedido e preferências)
            Set<CanalNotificacao> canaisAtivos = cmd.canaisPreferidos().stream()
                    .filter(prefs::aceitaCanal)
                    .collect(Collectors.toSet());

            if (canaisAtivos.isEmpty()) {
                log.info("Usuário {} não aceita nenhum canal para template {}",
                        cmd.destinatarioId(), cmd.templateId());
                ack.acknowledge();
                return;
            }

            // 4. Renderiza o template
            MensagemRenderizada mensagem = templateService.renderizar(
                    cmd.templateId(), cmd.variaveis(), prefs.idioma());

            // 5. Envia por cada canal ativo de forma independente
            canaisAtivos.parallelStream().forEach(canal -> {
                try {
                    canais.get(canal).enviar(prefs, mensagem);
                } catch (Exception e) {
                    log.error("Falha ao enviar via {}: usuario={}", canal, cmd.destinatarioId(), e);
                    // Falha em um canal não impede os outros
                }
            });

            dedup.marcarComoProcessado(cmd.destinatarioId(), cmd.templateId());
            ack.acknowledge();

        } catch (Exception e) {
            log.error("Erro ao processar notificação", e);
            // Não faz ack — mensagem vai para retry / DLQ
            throw e;
        }
    }
}
```

```java
// Canal de e-mail com template Thymeleaf e retry
@Service
@Slf4j
public class EmailCanalService implements CanalNotificacaoService {

    private final JavaMailSender mailSender;
    private final TemplateEngine thymeleaf;
    private final NotificacaoLogRepository logRepo;

    @Override
    @Retryable(
            retryFor = MailException.class,
            maxAttempts = 3,
            backoff = @Backoff(delay = 2000, multiplier = 2)
    )
    public void enviar(PreferenciasUsuario prefs, MensagemRenderizada msg) {
        MimeMessage mime = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mime, true, "UTF-8");

        helper.setTo(prefs.email());
        helper.setSubject(msg.assunto());
        helper.setText(msg.corpoHtml(), true);

        mailSender.send(mime);
        logRepo.registrar(prefs.usuarioId(), CanalNotificacao.EMAIL, StatusEnvio.ENTREGUE);
        log.info("E-mail enviado: usuario={}", prefs.usuarioId());
    }

    @Recover
    public void onFalha(MailException e, PreferenciasUsuario prefs, MensagemRenderizada msg) {
        log.error("E-mail falhou após retries: usuario={}", prefs.usuarioId(), e);
        logRepo.registrar(prefs.usuarioId(), CanalNotificacao.EMAIL, StatusEnvio.FALHOU);
    }
}
```

---

### 18.4 Sistema de Busca com Elasticsearch

#### Contexto

Um marketplace com centenas de milhares de produtos precisa de busca full-text com filtros por categoria, faixa de preço e avaliação, ordenação por relevância e sugestão de termos (autocomplete). O banco de dados relacional não comporta esse tipo de consulta com performance adequada.

#### Requisitos-Chave

```
Funcionais:
  - Busca por texto com stemming em português (plural, conjugações)
  - Filtros (facets): categoria, preço, marca, avaliação
  - Ordenação: relevância, menor preço, mais vendidos
  - Autocomplete ao digitar
  - Destaque dos termos buscados no resultado (highlighting)

Não-Funcionais:
  - Latência < 100ms para buscas
  - Índice sempre sincronizado com o banco de dados
  - Escala horizontal para bilhões de documentos
```

#### Arquitetura — CQRS aplicado à busca

```
  Write Path:                          Read Path:
  API → ProdutoService                 API → BuscaService
         │                                      │
         │ salva                                │ busca
         ▼                                      ▼
    PostgreSQL ─── Outbox/CDC ──► Kafka ──► Elasticsearch
                                  (indexacao async)
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```java
// Documento indexado no Elasticsearch
@Document(indexName = "produtos")
@Setting(settingPath = "elasticsearch/produto-settings.json")
public class ProdutoDocumento {

    @Id private String id;

    @Field(type = FieldType.Text, analyzer = "portuguese")
    private String nome;

    @Field(type = FieldType.Text, analyzer = "portuguese")
    private String descricao;

    @Field(type = FieldType.Keyword)
    private String categoria;

    @Field(type = FieldType.Keyword)
    private String marca;

    @Field(type = FieldType.Double)
    private BigDecimal preco;

    @Field(type = FieldType.Float)
    private float avaliacao;

    @Field(type = FieldType.Integer)
    private int totalVendas;

    @Field(type = FieldType.Boolean)
    private boolean ativo;

    // Campo para autocomplete
    @CompletionField(maxInputLength = 100)
    private Completion sugestao;
}
```

```json
// elasticsearch/produto-settings.json
{
  "analysis": {
    "analyzer": {
      "portuguese": {
        "type": "custom",
        "tokenizer": "standard",
        "filter": ["lowercase", "portuguese_stop", "portuguese_stemmer", "asciifolding"]
      }
    },
    "filter": {
      "portuguese_stop": {
        "type": "stop",
        "stopwords": "_portuguese_"
      },
      "portuguese_stemmer": {
        "type": "stemmer",
        "language": "light_portuguese"
      }
    }
  }
}
```

```java
@Service
public class BuscaService {

    private final ElasticsearchOperations esOps;

    public BuscaResultado buscar(BuscaRequest request) {
        // Query principal: busca por texto + filtros
        Query query = NativeQuery.builder()
                .withQuery(q -> q
                        .bool(b -> {
                            // Texto: busca em nome (peso 3x) e descrição (peso 1x)
                            if (StringUtils.hasText(request.texto())) {
                                b.must(m -> m.multiMatch(mm -> mm
                                        .query(request.texto())
                                        .fields("nome^3", "descricao")
                                        .fuzziness("AUTO")  // tolerância a erros de digitação
                                        .operator(Operator.And)));
                            }

                            // Filtros
                            b.filter(f -> f.term(t -> t.field("ativo").value(true)));

                            if (request.categoria() != null) {
                                b.filter(f -> f.term(t -> t.field("categoria").value(request.categoria())));
                            }
                            if (request.precoMin() != null || request.precoMax() != null) {
                                b.filter(f -> f.range(r -> {
                                    r.field("preco");
                                    if (request.precoMin() != null) r.gte(JsonData.of(request.precoMin()));
                                    if (request.precoMax() != null) r.lte(JsonData.of(request.precoMax()));
                                    return r;
                                }));
                            }
                            return b;
                        })
                )
                // Agregações para facets
                .withAggregation("por_categoria", Aggregation.of(a -> a
                        .terms(t -> t.field("categoria").size(20))))
                .withAggregation("faixa_preco", Aggregation.of(a -> a
                        .range(r -> r.field("preco")
                                .ranges(List.of(
                                        AggregationRange.of(ar -> ar.to(50.0)),
                                        AggregationRange.of(ar -> ar.from(50.0).to(200.0)),
                                        AggregationRange.of(ar -> ar.from(200.0)))))))
                // Highlighting
                .withHighlightQuery(new HighlightQuery(
                        new Highlight(List.of(new HighlightField("nome"), new HighlightField("descricao"))),
                        ProdutoDocumento.class))
                // Paginação e ordenação
                .withPageable(PageRequest.of(request.pagina(), request.tamanho(),
                        resolverOrdenacao(request.ordenacao())))
                .build();

        SearchHits<ProdutoDocumento> hits = esOps.search(query, ProdutoDocumento.class);
        return BuscaResultado.from(hits);
    }

    public List<String> autocomplete(String prefixo) {
        SuggestBuilder suggest = new SuggestBuilder()
                .addSuggestion("sugestoes", SuggestBuilders
                        .completionSuggestion("sugestao")
                        .prefix(prefixo)
                        .size(10));
        // ... executa e retorna sugestões
        return List.of();
    }

    private Sort resolverOrdenacao(String ordenacao) {
        return switch (ordenacao != null ? ordenacao : "relevancia") {
            case "menor_preco"   -> Sort.by(Sort.Order.asc("preco"));
            case "maior_preco"   -> Sort.by(Sort.Order.desc("preco"));
            case "mais_vendidos" -> Sort.by(Sort.Order.desc("totalVendas"));
            default              -> Sort.unsorted(); // relevância do Elasticsearch
        };
    }
}
```

```java
// Indexação assíncrona via evento Kafka (mantém ES sincronizado com Postgres)
@Component
@Slf4j
public class ProdutoIndexador {

    private final ProdutoDocumentoRepository esRepo;
    private final ProdutoRepository pgRepo;

    @KafkaListener(topics = "catalogo.produtos.alterados", groupId = "indexador-es")
    public void indexar(ProdutoAlteradoEvent evento, Acknowledgment ack) {
        pgRepo.findById(evento.produtoId()).ifPresent(produto -> {
            ProdutoDocumento doc = ProdutoDocumento.from(produto);
            esRepo.save(doc);
            log.info("Produto indexado no ES: id={}", produto.getId());
        });
        ack.acknowledge();
    }
}
```

---

### 18.5 Sistema de Agendamento e Reservas

#### Contexto

Uma plataforma de serviços (clínica, barbearia, academia) precisa gerenciar agenda de profissionais, permitir que clientes reservem horários disponíveis e enviar lembretes automáticos. O principal desafio técnico é evitar que dois clientes reservem o mesmo horário (condição de corrida).

#### Requisitos-Chave

```
Funcionais:
  - Configurar disponibilidade do profissional (dias/horários/duração do serviço)
  - Listar slots disponíveis para um profissional e data
  - Reservar um slot (somente se disponível)
  - Cancelar agendamento com prazo mínimo (ex: 2h antes)
  - Lembrete automático 24h e 1h antes do horário

Não-Funcionais:
  - Sem double-booking: garantia de exclusividade do slot
  - Slots expiram se não confirmados em 10 minutos (reserva temporária)
  - Alta leitura (listagem de horários) vs. baixa escrita (reserva)
```

#### Decisão Crítica — Evitar Double-Booking

```
Abordagem 1: Pessimistic Lock (SELECT FOR UPDATE)
  - Bloqueia o registro durante a transação
  - Seguro, mas reduz throughput em alta concorrência
  - Adequado para serviços com poucos agendamentos simultâneos

Abordagem 2: Optimistic Lock (@Version)
  - Não bloqueia; detecta conflito no commit
  - Maior throughput; requer retry na aplicação
  - Adequado para alta concorrência com baixa taxa de conflito

Abordagem 3: Reserva Temporária com Redis + Lua Script
  - Reserva atômica no Redis com TTL de 10 min
  - Apenas um cliente "trava" o slot; os outros recebem 409
  - Ideal para sistemas de reserva de alta demanda (ingressos, voos)
```

```java
@Entity
@Table(name = "slots_agenda",
       uniqueConstraints = @UniqueConstraint(columnNames = {"profissional_id", "inicio"}))
public class SlotAgenda {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "profissional_id", nullable = false)
    private Long profissionalId;

    @Column(nullable = false)
    private LocalDateTime inicio;

    @Column(nullable = false)
    private LocalDateTime fim;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private StatusSlot status; // DISPONIVEL, RESERVADO_TEMP, AGENDADO, CANCELADO

    @ManyToOne(fetch = FetchType.LAZY)
    private Agendamento agendamento;

    @Version  // Optimistic Lock
    private Long versao;
}
```

```java
@Service
@Slf4j
public class AgendamentoService {

    private final SlotAgendaRepository slotRepo;
    private final AgendamentoRepository agendamentoRepo;
    private final StringRedisTemplate redis;
    private final ApplicationEventPublisher events;

    private static final Duration TTL_RESERVA_TEMP = Duration.ofMinutes(10);

    /**
     * Reserva temporária com Redis — evita double-booking em alta concorrência.
     * A reserva permanente é confirmada em seguida via confirmacaoReserva().
     */
    public ReservaTemporariaResponse reservarTemporariamente(
            Long slotId, Long clienteId) {

        String lockKey = "reserva:slot:" + slotId;
        String valor   = clienteId.toString();

        // SET NX EX — atômica no Redis: só funciona se a chave não existe
        Boolean adquiriu = redis.opsForValue()
                .setIfAbsent(lockKey, valor, TTL_RESERVA_TEMP);

        if (!Boolean.TRUE.equals(adquiriu)) {
            String donoAtual = redis.opsForValue().get(lockKey);
            throw new SlotIndisponivelException(slotId,
                    "Slot está sendo reservado por outro cliente");
        }

        log.info("Reserva temporária adquirida: slot={}, cliente={}", slotId, clienteId);
        return new ReservaTemporariaResponse(
                slotId, TTL_RESERVA_TEMP, Instant.now().plus(TTL_RESERVA_TEMP));
    }

    @Transactional
    public Agendamento confirmarReserva(ConfirmarReservaRequest request) {
        String lockKey = "reserva:slot:" + request.slotId();

        // Verifica se a reserva temporária ainda pertence a este cliente
        String dono = redis.opsForValue().get(lockKey);
        if (!request.clienteId().toString().equals(dono)) {
            throw new ReservaExpiradaException(request.slotId());
        }

        // Busca e verifica o slot no banco
        SlotAgenda slot = slotRepo.findById(request.slotId())
                .orElseThrow(() -> new SlotNaoEncontradoException(request.slotId()));

        if (slot.getStatus() != StatusSlot.DISPONIVEL) {
            redis.delete(lockKey); // libera o lock Redis
            throw new SlotIndisponivelException(request.slotId(), slot.getStatus().name());
        }

        // Cria o agendamento e reserva o slot
        Agendamento agendamento = agendamentoRepo.save(
                new Agendamento(slot, request.clienteId(), request.servico()));
        slot.agendar(agendamento);
        slotRepo.save(slot);

        // Remove o lock Redis — slot agora está AGENDADO no banco
        redis.delete(lockKey);

        // Agenda lembretes automáticos
        events.publishEvent(new AgendamentoCriadoEvent(agendamento));

        log.info("Agendamento confirmado: id={}, slot={}, cliente={}",
                agendamento.getId(), slot.getId(), request.clienteId());
        return agendamento;
    }

    @Transactional
    public void cancelar(Long agendamentoId, Long solicitanteId) {
        Agendamento ag = agendamentoRepo.findById(agendamentoId)
                .orElseThrow();

        // Regra de negócio: cancelamento mínimo 2h antes
        if (ag.getSlot().getInicio().isBefore(LocalDateTime.now().plusHours(2))) {
            throw new CancelamentoForaDoPrazoException(agendamentoId);
        }

        ag.cancelar(solicitanteId);
        ag.getSlot().liberar();
        agendamentoRepo.save(ag);
        slotRepo.save(ag.getSlot());

        events.publishEvent(new AgendamentoCanceladoEvent(ag));
    }
}
```

```java
// Lembretes automáticos agendados com Spring Scheduling
@Component
@Slf4j
public class LembretesAgendamentosJob {

    private final AgendamentoRepository agendamentoRepo;
    private final ApplicationEventPublisher events;

    // Executa a cada 15 minutos — verifica agendamentos próximos
    @Scheduled(fixedRate = 900_000)
    @Transactional(readOnly = true)
    public void enviarLembretes() {
        LocalDateTime agora = LocalDateTime.now();

        // Lembrete de 24h
        LocalDateTime janela24hInicio = agora.plusHours(23).plusMinutes(45);
        LocalDateTime janela24hFim    = agora.plusHours(24).plusMinutes(15);

        agendamentoRepo
                .findByStatusAndSlotInicioBetween(StatusAgendamento.CONFIRMADO,
                        janela24hInicio, janela24hFim)
                .forEach(ag -> {
                    if (!ag.isLembrete24hEnviado()) {
                        events.publishEvent(new LembreteAgendamentoEvent(ag, "24h"));
                        ag.marcarLembrete24h();
                        agendamentoRepo.save(ag);
                    }
                });

        // Lembrete de 1h
        LocalDateTime janela1hInicio = agora.plusMinutes(45);
        LocalDateTime janela1hFim    = agora.plusMinutes(75);

        agendamentoRepo
                .findByStatusAndSlotInicioBetween(StatusAgendamento.CONFIRMADO,
                        janela1hInicio, janela1hFim)
                .forEach(ag -> {
                    if (!ag.isLembrete1hEnviado()) {
                        events.publishEvent(new LembreteAgendamentoEvent(ag, "1h"));
                        ag.marcarLembrete1h();
                        agendamentoRepo.save(ag);
                    }
                });

        log.debug("Job de lembretes executado às {}", agora);
    }
}
```

---

### 18.6 Upload e Processamento Assíncrono de Arquivos

#### Contexto

Uma plataforma precisa receber uploads de arquivos grandes (planilhas, imagens, vídeos) e processá-los de forma assíncrona: redimensionar imagens, validar e importar registros de planilhas, transcodificar vídeos. O cliente não deve aguardar o processamento completo na mesma requisição HTTP.

#### Arquitetura — Upload Direto para Object Storage

```
Abordagem 1 (via servidor — simples mas ineficiente):
  Cliente ──► API ──► Disco/Object Storage

Abordagem 2 (presigned URL — recomendada para arquivos grandes):
  1. Cliente solicita presigned URL à API
  2. API gera URL temporária no S3/GCS/MinIO
  3. Cliente faz upload diretamente para o storage
  4. Storage notifica API via webhook ou SQS
  5. API enfileira o processamento

  Cliente ──► API (1) ──► S3 (3)
  Cliente ──────────────────► S3 (upload direto) (2)
                         S3 ──► SQS/SNS (4)
                         SQS ──► API ──► Kafka ──► Worker (5)
```

```java
@RestController
@RequestMapping("/api/v1/uploads")
public class UploadController {

    private final UploadService uploadService;

    // Passo 1: cliente solicita URL de upload
    @PostMapping("/presigned-url")
    public PresignedUrlResponse gerarPresignedUrl(
            @RequestBody @Valid SolicitarUploadRequest request,
            @AuthenticationPrincipal Jwt jwt) {

        return uploadService.gerarPresignedUrl(
                jwt.getSubject(), request.nomeArquivo(), request.tipoConteudo());
    }

    // Passo 2: storage notifica conclusão do upload
    @PostMapping("/confirmacao")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void confirmarUpload(@RequestBody ConfirmacaoUploadRequest request) {
        uploadService.enfileirarProcessamento(request.arquivoKey(), request.metadados());
    }

    // Cliente consulta status do processamento
    @GetMapping("/{jobId}/status")
    public ProcessamentoStatusResponse consultarStatus(@PathVariable UUID jobId) {
        return uploadService.consultarStatus(jobId);
    }
}
```

```java
@Service
@Slf4j
public class UploadService {

    private final S3Presigner s3Presigner;
    private final KafkaTemplate<String, ProcessarArquivoCommand> kafka;
    private final ProcessamentoJobRepository jobRepo;

    @Value("${aws.s3.bucket}")
    private String bucket;

    public PresignedUrlResponse gerarPresignedUrl(
            String usuarioId, String nomeArquivo, String tipoConteudo) {

        String key = "uploads/%s/%s/%s".formatted(
                usuarioId, UUID.randomUUID(), nomeArquivo);

        PutObjectPresignRequest presignRequest = PutObjectPresignRequest.builder()
                .signatureDuration(Duration.ofMinutes(15))
                .putObjectRequest(r -> r
                        .bucket(bucket)
                        .key(key)
                        .contentType(tipoConteudo)
                        .build())
                .build();

        PresignedPutObjectRequest presigned = s3Presigner.presignPutObject(presignRequest);

        // Cria o job de processamento em AGUARDANDO
        ProcessamentoJob job = jobRepo.save(new ProcessamentoJob(usuarioId, key));

        return new PresignedUrlResponse(
                presigned.url().toString(), key, job.getId(),
                Duration.ofMinutes(15));
    }

    public void enfileirarProcessamento(String arquivoKey, Map<String, String> metadados) {
        ProcessarArquivoCommand cmd = new ProcessarArquivoCommand(
                arquivoKey, detectarTipo(arquivoKey), metadados);
        kafka.send("arquivos.para-processar", arquivoKey, cmd);
        log.info("Arquivo enfileirado para processamento: key={}", arquivoKey);
    }
}
```

```java
// Worker de processamento de imagens
@Component
@Slf4j
public class ImagemProcessamentoWorker {

    private final S3Client s3;
    private final ProcessamentoJobRepository jobRepo;

    @KafkaListener(
            topics = "arquivos.para-processar",
            groupId = "imagem-worker",
            containerFactory = "kafkaListenerContainerFactory"
    )
    public void processar(ProcessarArquivoCommand cmd, Acknowledgment ack) {
        if (cmd.tipo() != TipoArquivo.IMAGEM) {
            ack.acknowledge();
            return;
        }

        ProcessamentoJob job = jobRepo.findByArquivoKey(cmd.arquivoKey())
                .orElseThrow();
        job.iniciar();
        jobRepo.save(job);

        try {
            // Baixa do S3
            byte[] original = s3.getObjectAsBytes(r -> r.bucket("meu-bucket").key(cmd.arquivoKey()))
                    .asByteArray();

            // Processa com Thumbnailator
            List<VarianteImagem> variantes = List.of(
                    new VarianteImagem("thumbnail", 150, 150),
                    new VarianteImagem("medium",    800, 600),
                    new VarianteImagem("large",    1920, 1080)
            );

            for (VarianteImagem v : variantes) {
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                Thumbnails.of(new ByteArrayInputStream(original))
                        .size(v.largura(), v.altura())
                        .keepAspectRatio(true)
                        .outputFormat("webp")
                        .toOutputStream(baos);

                String varianteKey = cmd.arquivoKey().replace("uploads/", "processado/")
                        + "/" + v.nome() + ".webp";
                s3.putObject(r -> r.bucket("meu-bucket").key(varianteKey)
                        .contentType("image/webp"), RequestBody.fromBytes(baos.toByteArray()));
            }

            job.concluir(variantes.stream()
                    .map(v -> v.nome() + ".webp").collect(Collectors.joining(",")));
            jobRepo.save(job);
            ack.acknowledge();
            log.info("Imagem processada: key={}", cmd.arquivoKey());

        } catch (Exception e) {
            job.falhar(e.getMessage());
            jobRepo.save(job);
            log.error("Falha ao processar imagem: key={}", cmd.arquivoKey(), e);
            throw new RuntimeException(e);
        }
    }
}
```

---

### 18.7 Plataforma de Vídeos — Envio, Processamento e Streaming

#### Contexto

Uma plataforma de vídeos como YouTube ou Netflix precisa lidar com três desafios completamente distintos: **ingestão** (o criador envia um arquivo de vídeo bruto), **processamento** (transcodificar para múltiplas resoluções e formatos) e **entrega** (o espectador assiste com qualidade adaptativa, independente da velocidade de internet e do dispositivo). Cada etapa tem requisitos de escala e latência radicalmente diferentes.

#### Estimativas de Capacidade (YouTube scale)

```
Uploads:
  500 horas de vídeo enviadas por minuto
  Tamanho médio do arquivo bruto: 2 GB (1h em 1080p)
  Taxa de ingestão: ~500 * 2 GB/min ≈ 1 TB/min de entrada

Armazenamento (após transcodificação, 4 variantes por vídeo):
  1 hora de vídeo original = ~2 GB
  Após transcodificar (1080p + 720p + 480p + 360p + audio) ≈ 3,5 GB total
  1 bilhão de vídeos * 3,5 GB ≈ 3,5 EB (exabytes) de armazenamento

Reprodução:
  2 bilhões de usuários ativos/mês; pico de ~80 milhões simultâneos
  Bitrate médio de streaming: 2 Mbps (720p)
  Banda de saída: 80M * 2 Mbps = 160 Tbps → entregue via CDN
```

#### Arquitetura Geral

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PLATAFORMA DE VÍDEOS                        │
│                                                                      │
│  ┌──────────────┐   ┌──────────────────┐   ┌─────────────────────┐  │
│  │   Upload     │   │  Transcodificação│   │     Streaming       │  │
│  │  Service     │   │     Pipeline     │   │     Service         │  │
│  │              │   │                  │   │                     │  │
│  │ - Presigned  │   │ - Segmentação    │   │ - HLS/DASH manifest │  │
│  │   URL        │   │ - FFmpeg workers │   │ - Bitrate adaptativo│  │
│  │ - Chunked    │   │ - Multi-resolução│   │ - Token de acesso   │  │
│  │   upload     │   │ - Thumbnail      │   │   (conteúdo pago)   │  │
│  │ - Resumable  │   │ - Legenda (ML)   │   │                     │  │
│  └──────┬───────┘   └────────┬─────────┘   └──────────┬──────────┘  │
│         │                    │                         │             │
│         │      Kafka         │                         │             │
│         └────────────────────┘                         │             │
│                    │                                   │             │
│             ┌──────▼──────┐                    ┌───────▼──────────┐  │
│             │  Metadados  │                    │    CDN           │  │
│             │  Service    │                    │  (CloudFront,    │  │
│             │ (PostgreSQL │                    │   Akamai)        │  │
│             │  + Redis)   │                    └──────────────────┘  │
│             └─────────────┘                                          │
└─────────────────────────────────────────────────────────────────────┘
```

#### Etapa 1 — Ingestão: Upload Resumável por Chunks

Vídeos podem ter vários gigabytes. Uma conexão interrompida não deve exigir reenvio do arquivo inteiro. O protocolo **TUS** (ou implementação própria) divide o arquivo em chunks e permite retomar de onde parou.

```
Upload Resumável:

  Chunk 1 (10 MB) ──► OK  (offset: 10 MB)
  Chunk 2 (10 MB) ──► OK  (offset: 20 MB)
  Chunk 3 (10 MB) ──► FALHA (rede caiu)
  ...retoma mais tarde...
  Chunk 3 (10 MB) ──► OK  (offset: 30 MB) ← retoma do offset salvo
  Chunk N         ──► OK  (upload concluído → notifica processamento)
```

```java
@RestController
@RequestMapping("/api/v1/videos/uploads")
@Slf4j
public class VideoUploadController {

    private final VideoUploadService uploadService;

    // Passo 1: iniciar sessão de upload — retorna upload ID e URL
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public IniciarUploadResponse iniciar(
            @RequestBody @Valid IniciarUploadRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        return uploadService.iniciarSessao(jwt.getSubject(), request);
    }

    // Passo 2: enviar chunk (Content-Range: bytes 0-10485759/2147483648)
    @PutMapping("/{uploadId}")
    public ResponseEntity<UploadProgressResponse> enviarChunk(
            @PathVariable UUID uploadId,
            @RequestHeader("Content-Range") String contentRange,
            @RequestHeader("Content-Length") long contentLength,
            InputStream body) {

        ChunkInfo chunk = ChunkInfo.parse(contentRange, contentLength);
        UploadProgressResponse progress = uploadService.processarChunk(uploadId, chunk, body);

        // 308 Resume Incomplete enquanto não terminar; 200 quando concluir
        HttpStatus status = progress.completo() ? HttpStatus.OK : HttpStatus.PERMANENT_REDIRECT;
        return ResponseEntity.status(status)
                .header("Range", "bytes=0-" + (progress.bytesRecebidos() - 1))
                .body(progress);
    }

    // Cliente consulta o progresso (retomada após falha)
    @GetMapping("/{uploadId}")
    public UploadProgressResponse consultarProgresso(@PathVariable UUID uploadId) {
        return uploadService.consultarProgresso(uploadId);
    }
}
```

```java
@Service
@Slf4j
public class VideoUploadService {

    private final S3Client s3;
    private final UploadSessaoRepository sessaoRepo;
    private final KafkaTemplate<String, VideoRecebidoEvent> kafka;

    @Value("${aws.s3.bucket.raw}")
    private String bucketRaw;

    public IniciarUploadResponse iniciarSessao(String autorId, IniciarUploadRequest req) {
        String videoKey = "raw/%s/%s/%s".formatted(
                autorId, UUID.randomUUID(), req.nomeArquivo());

        // Inicia Multipart Upload no S3 — permite enviar partes de 5 MB a 5 GB
        CreateMultipartUploadResponse mpu = s3.createMultipartUpload(r -> r
                .bucket(bucketRaw)
                .key(videoKey)
                .contentType(req.tipoConteudo())
                .metadata(Map.of(
                        "autor-id",    autorId,
                        "titulo",      req.titulo(),
                        "tamanho-total", String.valueOf(req.tamanhoBytes()))));

        UploadSessao sessao = new UploadSessao(
                mpu.uploadId(), videoKey, autorId, req.tamanhoBytes(),
                req.titulo(), req.descricao());
        sessaoRepo.save(sessao);

        log.info("Sessão de upload iniciada: videoKey={}, autor={}", videoKey, autorId);
        return new IniciarUploadResponse(sessao.getId(), videoKey, mpu.uploadId());
    }

    public UploadProgressResponse processarChunk(UUID sessaoId, ChunkInfo chunk, InputStream dados) {
        UploadSessao sessao = sessaoRepo.findById(sessaoId).orElseThrow();

        // Envia parte para o S3 (parte mínima: 5 MB, exceto a última)
        int numeroParte = chunk.numeroParte(); // 1-based
        UploadPartResponse resp = s3.uploadPart(r -> r
                .bucket(bucketRaw)
                .key(sessao.getVideoKey())
                .uploadId(sessao.getS3UploadId())
                .partNumber(numeroParte)
                .contentLength(chunk.tamanho()),
                RequestBody.fromInputStream(dados, chunk.tamanho()));

        sessao.registrarParte(numeroParte, resp.eTag(), chunk.tamanhoAcumulado());
        sessaoRepo.save(sessao);

        if (sessao.estaCompleto()) {
            concluirUpload(sessao);
        }

        log.debug("Chunk {}/{} recebido: sessao={}", chunk.numeroParte(),
                sessao.totalPartes(), sessaoId);
        return UploadProgressResponse.from(sessao);
    }

    private void concluirUpload(UploadSessao sessao) {
        // Finaliza o Multipart Upload no S3
        List<CompletedPart> partes = sessao.getPartes().stream()
                .map(p -> CompletedPart.builder()
                        .partNumber(p.numero()).eTag(p.etag()).build())
                .toList();

        s3.completeMultipartUpload(r -> r
                .bucket(bucketRaw)
                .key(sessao.getVideoKey())
                .uploadId(sessao.getS3UploadId())
                .multipartUpload(m -> m.parts(partes)));

        sessao.concluir();
        sessaoRepo.save(sessao);

        // Notifica pipeline de transcodificação
        kafka.send("videos.recebidos",
                sessao.getVideoKey(),
                new VideoRecebidoEvent(
                        sessao.getId(),
                        sessao.getVideoKey(),
                        sessao.getAutorId(),
                        sessao.getTitulo(),
                        sessao.getDescricao(),
                        Instant.now()));

        log.info("Upload concluído e enfileirado para transcodificação: key={}",
                sessao.getVideoKey());
    }
}
```

#### Etapa 2 — Processamento: Transcodificação e Segmentação

O arquivo bruto deve ser convertido para múltiplas resoluções e segmentado no formato **HLS** (HTTP Live Streaming) ou **DASH**, permitindo ao player trocar de qualidade automaticamente conforme a banda disponível.

```
Arquivo bruto (ex: video.mp4 — 1080p, H.264, 4 GB)
         │
         ▼
  ┌──────────────────────────────────────────────────────────┐
  │                  Pipeline de Transcodificação            │
  │                                                          │
  │  Paralelamente (worker por resolução):                   │
  │                                                          │
  │  Worker A ──► 1080p (H.264/AAC) → segmentos .ts + .m3u8 │
  │  Worker B ──► 720p              → segmentos .ts + .m3u8 │
  │  Worker C ──► 480p              → segmentos .ts + .m3u8 │
  │  Worker D ──► 360p              → segmentos .ts + .m3u8 │
  │  Worker E ──► Thumbnail (frame 10s, 30s, 60s...)        │
  │  Worker F ──► Extração de áudio → legenda automática     │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
         │
         ▼
  Object Storage (S3/GCS):
    /videos/{videoId}/hls/
      ├── master.m3u8           ← playlist mestre (lista as variantes)
      ├── 1080p/
      │   ├── playlist.m3u8    ← playlist da resolução
      │   ├── seg000.ts
      │   ├── seg001.ts
      │   └── ...
      ├── 720p/ ...
      ├── 480p/ ...
      └── 360p/ ...
```

```java
// Orquestrador da pipeline de transcodificação
@Component
@Slf4j
public class TranscodificacaoPipeline {

    private final S3Client s3;
    private final VideoMetadataRepository metadataRepo;
    private final ApplicationEventPublisher events;
    private final MeterRegistry metrics;

    @Value("${aws.s3.bucket.raw}")    private String bucketRaw;
    @Value("${aws.s3.bucket.hls}")    private String bucketHls;
    @Value("${ffmpeg.bin.path:/usr/bin/ffmpeg}") private String ffmpegPath;

    @KafkaListener(topics = "videos.recebidos", groupId = "transcodificacao-pipeline")
    public void processar(VideoRecebidoEvent evento, Acknowledgment ack) {
        Instant inicio = Instant.now();
        log.info("Iniciando transcodificação: videoId={}", evento.uploadSessaoId());

        // Cria/atualiza metadados com status PROCESSANDO
        Video video = metadataRepo.findByUploadSessaoId(evento.uploadSessaoId())
                .orElseGet(() -> metadataRepo.save(Video.criar(evento)));
        video.iniciarProcessamento();
        metadataRepo.save(video);

        Path arquivoBruto = null;
        try {
            // 1. Baixa o arquivo bruto do S3 para disco temporário
            arquivoBruto = baixarArquivoBruto(evento.videoKey());

            // 2. Extrai metadados técnicos (duração, codec, resolução original)
            MetadadosTecnicos meta = extrairMetadados(arquivoBruto);
            video.atualizarMetadados(meta);
            metadataRepo.save(video);

            // 3. Transcodifica cada resolução em paralelo
            List<ResolucaoAlvo> resolucoes = resolverResolucoes(meta.alturaOriginal());
            List<CompletableFuture<SegmentoHLS>> futures = resolucoes.stream()
                    .map(res -> transcodificarAsync(arquivoBruto, video.getId(), res))
                    .toList();

            // 4. Aguarda todas as transcodificações
            List<SegmentoHLS> segmentos = futures.stream()
                    .map(CompletableFuture::join)
                    .toList();

            // 5. Gera a master playlist HLS que referencia todas as variantes
            String masterPlaylistKey = gerarMasterPlaylist(video.getId(), segmentos);

            // 6. Gera thumbnails em paralelo com a transcodificação
            List<String> thumbnails = gerarThumbnails(arquivoBruto, video.getId(), meta.duracao());

            // 7. Marca como publicado
            video.publicar(masterPlaylistKey, thumbnails, meta);
            metadataRepo.save(video);

            events.publishEvent(new VideoPublicadoEvent(video));
            metrics.timer("video.transcodificacao.duracao")
                    .record(Duration.between(inicio, Instant.now()));

            ack.acknowledge();
            log.info("Transcodificação concluída: videoId={}, duração={}s",
                    video.getId(), meta.duracao());

        } catch (Exception e) {
            video.falhar(e.getMessage());
            metadataRepo.save(video);
            log.error("Falha na transcodificação: videoId={}", video.getId(), e);
            throw new RuntimeException(e); // mensagem vai para DLQ

        } finally {
            if (arquivoBruto != null) {
                try { Files.deleteIfExists(arquivoBruto); } catch (IOException ignored) {}
            }
        }
    }

    @Async("transcodificacaoExecutor")
    private CompletableFuture<SegmentoHLS> transcodificarAsync(
            Path entrada, UUID videoId, ResolucaoAlvo res) {

        String prefixo = "videos/%s/hls/%s/".formatted(videoId, res.nome());
        Path dirSaida  = Files.createTempDirectory("hls-" + res.nome());

        try {
            // Invoca FFmpeg para segmentar em HLS
            // -c:v libx264 -crf 23 -preset fast -c:a aac -b:a 128k
            // -hls_time 6 -hls_playlist_type vod -hls_segment_filename seg%03d.ts
            ProcessBuilder pb = new ProcessBuilder(
                    ffmpegPath, "-i", entrada.toString(),
                    "-vf", "scale=-2:" + res.altura(),
                    "-c:v", "libx264", "-crf", String.valueOf(res.crf()),
                    "-preset", "fast",
                    "-c:a", "aac", "-b:a", "128k",
                    "-hls_time", "6",
                    "-hls_playlist_type", "vod",
                    "-hls_segment_filename", dirSaida + "/seg%03d.ts",
                    dirSaida + "/playlist.m3u8"
            );
            pb.redirectErrorStream(true);
            Process proc = pb.start();
            int exitCode = proc.waitFor();
            if (exitCode != 0) {
                throw new TranscodificacaoException("FFmpeg falhou com código " + exitCode);
            }

            // Upload de todos os segmentos e da playlist para o S3
            try (Stream<Path> arquivos = Files.list(dirSaida)) {
                arquivos.forEach(f -> s3.putObject(
                        r -> r.bucket(bucketHls).key(prefixo + f.getFileName()),
                        RequestBody.fromFile(f)));
            }

            log.info("Resolução {} transcodificada: videoId={}", res.nome(), videoId);
            return CompletableFuture.completedFuture(
                    new SegmentoHLS(res, prefixo + "playlist.m3u8", res.bandwidth()));

        } finally {
            // Remove diretório temporário
            FileSystemUtils.deleteRecursively(dirSaida.toFile());
        }
    }

    private String gerarMasterPlaylist(UUID videoId, List<SegmentoHLS> segmentos) {
        // Gera o arquivo master.m3u8 que lista todas as variantes
        StringBuilder m3u8 = new StringBuilder("#EXTM3U\n#EXT-X-VERSION:3\n\n");
        for (SegmentoHLS seg : segmentos) {
            m3u8.append("#EXT-X-STREAM-INF:BANDWIDTH=%d,RESOLUTION=%s\n%s\n\n"
                    .formatted(seg.bandwidth(), seg.resolucao().dimensoes(),
                               seg.playlistKey()));
        }

        String masterKey = "videos/%s/hls/master.m3u8".formatted(videoId);
        s3.putObject(
                r -> r.bucket(bucketHls).key(masterKey).contentType("application/x-mpegURL"),
                RequestBody.fromString(m3u8.toString()));

        return masterKey;
    }

    private List<String> gerarThumbnails(Path video, UUID videoId, int duracaoSegundos) {
        List<String> urls = new ArrayList<>();
        // Captura frames a cada 10% da duração (máximo 10 frames)
        int intervalo = Math.max(duracaoSegundos / 10, 1);
        for (int t = 0; t < duracaoSegundos; t += intervalo) {
            Path thumb = Files.createTempFile("thumb-", ".jpg");
            new ProcessBuilder(ffmpegPath,
                    "-ss", String.valueOf(t), "-i", video.toString(),
                    "-vframes", "1", "-vf", "scale=480:-1", "-q:v", "5",
                    thumb.toString())
                    .start().waitFor();

            String key = "videos/%s/thumbnails/thumb_%05d.jpg".formatted(videoId, t);
            s3.putObject(r -> r.bucket(bucketHls).key(key).contentType("image/jpeg"),
                    RequestBody.fromFile(thumb));
            urls.add(key);
            Files.deleteIfExists(thumb);
        }
        return urls;
    }

    private List<ResolucaoAlvo> resolverResolucoes(int alturaOriginal) {
        // Só gera resoluções menores ou iguais à original
        return Stream.of(
                new ResolucaoAlvo("1080p", 1080, 23, 5_000_000),
                new ResolucaoAlvo("720p",   720, 23, 2_800_000),
                new ResolucaoAlvo("480p",   480, 24, 1_400_000),
                new ResolucaoAlvo("360p",   360, 25,   800_000)
        ).filter(r -> r.altura() <= alturaOriginal).toList();
    }
}
```

```yaml
# Pool dedicado para workers de transcodificação (CPU-intensivo)
spring:
  task:
    execution:
      pool:
        core-size: 4         # 1 thread por resolução em paralelo
        max-size: 8
        queue-capacity: 50
      thread-name-prefix: transcodificacao-
```

#### Etapa 3 — Streaming: Entrega com Bitrate Adaptativo (ABR)

O player do cliente baixa o arquivo `master.m3u8`, verifica a banda disponível e solicita segmentos `.ts` da resolução mais adequada, podendo mudar a cada segmento (a cada ~6 segundos).

```
Fluxo HLS Adaptativo:

  Player                API Gateway / CDN              S3 (Origin)
    │                         │                            │
    │── GET /videos/{id} ────►│                            │
    │◄── 200 metadados ───────│                            │
    │                         │                            │
    │── GET master.m3u8 ─────►│── (cache miss) ───────────►│
    │◄── #EXTM3U ... ─────────│◄──────────────────────────│
    │                         │                            │
    │  [mede banda: 4 Mbps → escolhe 720p]                │
    │── GET 720p/seg000.ts ──►│── (cache hit) ────────────►│
    │◄── segmento (6s) ───────│◄── (cache ou origin) ─────│
    │── GET 720p/seg001.ts ──►│                            │
    │  [banda caiu → muda para 480p]                       │
    │── GET 480p/seg002.ts ──►│                            │
```

```java
// Endpoint de streaming — serve URL assinada para acesso autenticado
@RestController
@RequestMapping("/api/v1/videos")
public class VideoStreamingController {

    private final VideoMetadataRepository metadataRepo;
    private final VideoAcessoService acessoService;
    private final S3Presigner presigner;

    @Value("${aws.s3.bucket.hls}")   private String bucketHls;
    @Value("${cdn.base-url}")        private String cdnBaseUrl;

    // Retorna metadados + URL da master playlist para iniciar o player
    @GetMapping("/{videoId}")
    public VideoPlayerResponse obterVideo(
            @PathVariable UUID videoId,
            @AuthenticationPrincipal Jwt jwt) {

        Video video = metadataRepo.findById(videoId)
                .filter(v -> v.getStatus() == StatusVideo.PUBLICADO)
                .orElseThrow(() -> new VideoNaoEncontradoException(videoId));

        // Para conteúdo pago: verifica assinatura ou compra
        if (video.isPremium()) {
            acessoService.verificarAcesso(jwt.getSubject(), videoId);
        }

        // Gera token de streaming (JWT de curta duração para o CDN)
        String streamingToken = acessoService.gerarTokenStreaming(
                jwt.getSubject(), videoId, Duration.ofHours(4));

        // URL da master playlist via CDN com token no query string
        // O CDN verifica o token antes de servir os segmentos
        String playlistUrl = "%s/%s?token=%s".formatted(
                cdnBaseUrl, video.getMasterPlaylistKey(), streamingToken);

        return new VideoPlayerResponse(
                video.getId(),
                video.getTitulo(),
                video.getDescricao(),
                playlistUrl,
                video.getThumbnailPrincipal(),
                video.getDuracaoSegundos(),
                video.getResolucoes()
        );
    }

    // Gera URL pré-assinada para segmento específico (sem CDN, acesso direto ao S3)
    // Usado em ambientes onde CDN não está disponível
    @GetMapping("/{videoId}/segments/{resolucao}/{segmento}")
    public ResponseEntity<Void> obterSegmento(
            @PathVariable UUID videoId,
            @PathVariable String resolucao,
            @PathVariable String segmento,
            @RequestParam String token) {

        acessoService.verificarTokenStreaming(token, videoId);

        String key = "videos/%s/hls/%s/%s".formatted(videoId, resolucao, segmento);
        PresignedGetObjectRequest presigned = presigner.presignGetObject(r -> r
                .signatureDuration(Duration.ofMinutes(5))
                .getObjectRequest(g -> g.bucket(bucketHls).key(key)));

        return ResponseEntity.status(HttpStatus.FOUND)
                .location(presigned.url().toURI())
                .build();
    }
}
```

```java
// Controle de acesso e token de streaming
@Service
public class VideoAcessoService {

    private final AssinaturaRepository assinaturaRepo;
    private final CompraRepository compraRepo;
    private final JwtEncoder jwtEncoder;
    private final JwtDecoder jwtDecoder;

    public void verificarAcesso(String usuarioId, UUID videoId) {
        boolean temAcesso = assinaturaRepo.existeAssinaturaAtiva(usuarioId)
                || compraRepo.existeCompraPorVideo(usuarioId, videoId);
        if (!temAcesso) {
            throw new AcessoNegadoException("Conteúdo premium requer assinatura ou compra");
        }
    }

    public String gerarTokenStreaming(String usuarioId, UUID videoId, Duration validade) {
        JwtClaimsSet claims = JwtClaimsSet.builder()
                .subject(usuarioId)
                .claim("video_id", videoId.toString())
                .claim("tipo", "streaming")
                .issuedAt(Instant.now())
                .expiresAt(Instant.now().plus(validade))
                .build();
        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }

    public void verificarTokenStreaming(String token, UUID videoId) {
        Jwt jwt = jwtDecoder.decode(token);
        String tokenVideoId = jwt.getClaimAsString("video_id");
        if (!videoId.toString().equals(tokenVideoId)) {
            throw new TokenStreamingInvalidoException();
        }
    }
}
```

#### Etapa 4 — Metadados, Busca e Recomendações

```java
@Entity
@Table(name = "videos")
public class Video {

    @Id private UUID id;
    private UUID uploadSessaoId;
    private String autorId;
    private String titulo;

    @Column(columnDefinition = "text")
    private String descricao;

    @Enumerated(EnumType.STRING)
    private StatusVideo status; // AGUARDANDO, PROCESSANDO, PUBLICADO, FALHOU, REMOVIDO

    private String masterPlaylistKey;
    private int duracaoSegundos;
    private boolean premium;

    @ElementCollection
    @CollectionTable(name = "video_thumbnails")
    private List<String> thumbnails;

    @ElementCollection
    @CollectionTable(name = "video_resolucoes")
    @Enumerated(EnumType.STRING)
    private List<ResolucaoAlvo> resolucoes;

    // Métricas de engajamento
    private long totalVisualizacoes;
    private long totalLikes;
    private long totalComentarios;
    private double taxaRetencaoMedia; // porcentagem do vídeo assistida em média

    private Instant criadoEm;
    private Instant publicadoEm;

    // Índices para feed e busca
    @Column(name = "tags")
    @JdbcTypeCode(SqlTypes.ARRAY)
    private String[] tags;

    public void publicar(String playlistKey, List<String> thumbs, MetadadosTecnicos meta) {
        this.status            = StatusVideo.PUBLICADO;
        this.masterPlaylistKey = playlistKey;
        this.thumbnails        = thumbs;
        this.duracaoSegundos   = meta.duracao();
        this.resolucoes        = meta.resolucoes();
        this.publicadoEm       = Instant.now();
    }
}
```

```java
// Registro de visualizações — alta taxa de escrita, não deve bloquear o streaming
@Service
public class VisualizacaoService {

    private final StringRedisTemplate redis;
    private final KafkaTemplate<String, VisualizacaoEvent> kafka;

    // Registra visualização de forma assíncrona (não bloqueia o player)
    @Async
    public void registrar(UUID videoId, String usuarioId, String sessionId) {
        String dedupKey = "viz:dedup:%s:%s".formatted(videoId, sessionId);

        // Evita contar múltiplas views da mesma sessão em 30 minutos
        Boolean nova = redis.opsForValue().setIfAbsent(dedupKey, "1", Duration.ofMinutes(30));
        if (!Boolean.TRUE.equals(nova)) return;

        // Incrementa contador em Redis (rápido) — sincroniza com DB periodicamente
        redis.opsForValue().increment("viz:count:" + videoId);

        // Publica evento para analytics e recomendação
        kafka.send("videos.visualizacoes",
                videoId.toString(),
                new VisualizacaoEvent(videoId, usuarioId, Instant.now()));
    }

    // Job que sincroniza contadores do Redis para o PostgreSQL a cada minuto
    @Scheduled(fixedRate = 60_000)
    public void sincronizarContadores() {
        // Lê chaves "viz:count:*" do Redis e persiste no banco em batch
        // ...
    }
}
```

#### Decisões Arquiteturais Consolidadas

| Decisão | Escolha | Motivação |
|---------|---------|-----------|
| Upload de arquivos grandes | Multipart Upload S3 + Chunked (resumável) | Tolerância a falhas de rede; evita reenvio do arquivo inteiro |
| Pipeline de transcodificação | Workers Kafka + FFmpeg por resolução em paralelo | Cada resolução é independente; falha em uma não cancela as demais |
| Formato de streaming | HLS com segmentos de 6s | Suporte universal (iOS, Android, browsers); ABR nativo |
| Armazenamento de segmentos | S3 como origin + CDN na borda | S3 para durabilidade; CDN absorve 99% das requisições de leitura |
| Controle de acesso a segmentos | JWT de curta duração + token no CDN | Conteúdo premium protegido sem custo de verificação por segmento |
| Contagem de visualizações | Redis (tempo real) + Kafka (analytics) + sync para DB | Redis absorve picos; DB para relatórios; sem perda de dados |
| Recomendação | Evento de visualização → serviço de ML separado | Desacoplado; pode usar modelos diferentes sem afetar o streaming |
| Thumbnails | Gerados durante a transcodificação, múltiplos pontos | Permite preview ao passar o mouse (como YouTube) sem requisição extra |

---

### 18.8 Gestão de Identidade com Keycloak — Multi-Tenant e SSO

#### Contexto

Uma plataforma SaaS B2B precisa gerenciar identidades de múltiplos clientes (tenants) de forma isolada, permitir que cada tenant configure seu próprio provedor de identidade (SSO com Google Workspace, Azure AD, SAML corporativo), aplicar políticas de acesso por papel e recurso, e integrar todos os microsserviços sem que cada um implemente autenticação própria.

#### Conceitos-Chave do Keycloak

```
Realm:      Unidade de isolamento — um realm por tenant.
            Usuários, papéis, clientes e políticas são totalmente
            separados entre realms. Um usuário do Realm A não existe no Realm B.

Client:     Aplicação registrada no Keycloak (frontend, backend, microsserviço).
            Cada client tem seu client_id, segredo e fluxo de autenticação.

Role:       Papel atribuído a um usuário dentro de um realm ou client.
            Realm roles: globais no realm.
            Client roles: específicas de uma aplicação.

Scope:      Conjunto de permissões que um client pode solicitar no token.

Group:      Agrupa usuários para aplicar papéis e atributos em massa.

Identity Provider (IdP): Provedor externo (Google, Azure AD, SAML).
            O Keycloak atua como broker — autentica via IdP externo
            e emite seu próprio token JWT para as aplicações.

Fine-Grained Authorization (UMA): Políticas baseadas em recursos,
            escopos e condições (ex: "somente o dono pode editar").
```

#### Arquitetura Multi-Tenant

```
Estratégia 1 — Um Realm por Tenant (recomendada para SaaS B2B):

  ┌──────────────────────────────────────────────────────────┐
  │                        Keycloak                          │
  │                                                          │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
  │  │ Realm: acme  │  │ Realm: globo │  │ Realm: xpto  │   │
  │  │              │  │              │  │              │   │
  │  │ Usuários A   │  │ Usuários B   │  │ Usuários C   │   │
  │  │ Papéis A     │  │ Papéis B     │  │ Papéis C     │   │
  │  │ IdP: Azure   │  │ IdP: Google  │  │ IdP: SAML    │   │
  │  │ Políticas A  │  │ Políticas B  │  │ Políticas C  │   │
  │  └──────────────┘  └──────────────┘  └──────────────┘   │
  └──────────────────────────────────────────────────────────┘

  Vantagens:
  - Isolamento total de dados e configurações
  - Cada tenant configura seu próprio IdP/SSO
  - Políticas de senha e MFA independentes por tenant
  - Auditoria e logs segregados

  Desvantagem:
  - Overhead de gerenciar N realms (mitigado com automação via Admin API)

Estratégia 2 — Realm único com Groups por Tenant:
  - Mais simples de operar
  - Menos isolamento — não recomendada para dados sensíveis
  - Adequada quando os tenants são departamentos da mesma empresa
```

```
Fluxo de Autenticação (Authorization Code + PKCE):

  Browser/App             Keycloak (Realm: acme)         Resource Server
       │                          │                            │
       │── GET /authorize ───────►│                            │
       │   (client_id, scope,      │                            │
       │    redirect_uri, PKCE)    │                            │
       │◄── login page ───────────│                            │
       │── credenciais ──────────►│                            │
       │   (ou redirect para IdP) │                            │
       │◄── authorization_code ───│                            │
       │── POST /token ──────────►│                            │
       │   (code, code_verifier)   │                            │
       │◄── access_token (JWT) ───│                            │
       │    refresh_token          │                            │
       │                           │                            │
       │── GET /api/recurso ───────────────────────────────────►│
       │   Authorization: Bearer <access_token>                 │
       │◄── 200 OK ────────────────────────────────────────────│
```

#### Provisionamento Automático de Tenants via Admin API

Em um SaaS, quando um novo cliente contrata o produto, o onboarding deve criar o realm, configurar papéis padrão, registrar os clients e (opcionalmente) integrar o IdP do cliente — tudo de forma programática.

```java
@Service
@Slf4j
public class TenantProvisionamentoService {

    private final Keycloak keycloakAdmin;
    private final TenantRepository tenantRepo;

    @Value("${keycloak.server-url}")
    private String keycloakServerUrl;

    /**
     * Cria um realm isolado para o novo tenant com:
     * - Papéis padrão (ADMIN, GERENTE, OPERADOR, LEITURA)
     * - Client para o frontend SPA
     * - Client para comunicação backend (client credentials)
     * - Políticas de senha e sessão
     */
    @Transactional
    public TenantProvisionado provisionar(NovoTenantRequest req) {
        String realmId = req.slug(); // ex: "acme-corp"

        // 1. Cria o realm
        RealmRepresentation realm = new RealmRepresentation();
        realm.setRealm(realmId);
        realm.setDisplayName(req.nomeExibicao());
        realm.setEnabled(true);
        realm.setDefaultLocale("pt-BR");
        realm.setSupportedLocales(Set.of("pt-BR", "en"));

        // Política de senhas
        realm.setPasswordPolicy(
                "length(8) and upperCase(1) and digits(1) and specialChars(1) and notUsername");

        // Configuração de sessão
        realm.setAccessTokenLifespan(300);          // 5 min
        realm.setSsoSessionMaxLifespan(28800);       // 8h
        realm.setRefreshTokenMaxReuse(0);
        realm.setRevokeRefreshToken(true);

        keycloakAdmin.realms().create(realm);
        RealmResource realmResource = keycloakAdmin.realm(realmId);

        // 2. Cria papéis de negócio
        criarPapeisNegocio(realmResource);

        // 3. Registra client para o frontend SPA (Authorization Code + PKCE)
        String clientIdFrontend = criarClientFrontend(realmResource, req);

        // 4. Registra client para microsserviços (Client Credentials)
        ClientCredentials credenciais = criarClientBackend(realmResource, req);

        // 5. Cria usuário administrador inicial do tenant
        criarAdminInicial(realmResource, req);

        // 6. Persiste no banco da plataforma
        Tenant tenant = tenantRepo.save(new Tenant(realmId, req, credenciais));

        log.info("Tenant provisionado: realm={}", realmId);
        return new TenantProvisionado(tenant.getId(), realmId,
                keycloakServerUrl + "/realms/" + realmId, credenciais);
    }

    private void criarPapeisNegocio(RealmResource realm) {
        List<String> papeis = List.of("ADMIN_TENANT", "GERENTE", "OPERADOR", "LEITURA");
        papeis.forEach(papel -> {
            RoleRepresentation role = new RoleRepresentation();
            role.setName(papel);
            role.setDescription("Papel padrão: " + papel);
            realm.roles().create(role);
        });

        // Hierarquia: ADMIN_TENANT inclui todos os outros
        RoleRepresentation admin = realm.roles().get("ADMIN_TENANT").toRepresentation();
        List<RoleRepresentation> subordinados = papeis.stream()
                .filter(p -> !p.equals("ADMIN_TENANT"))
                .map(p -> realm.roles().get(p).toRepresentation())
                .toList();
        realm.roles().get("ADMIN_TENANT").addComposites(subordinados);
    }

    private String criarClientFrontend(RealmResource realm, NovoTenantRequest req) {
        ClientRepresentation client = new ClientRepresentation();
        client.setClientId(req.slug() + "-frontend");
        client.setName(req.nomeExibicao() + " — Frontend");
        client.setPublicClient(true);        // SPA não guarda segredo
        client.setStandardFlowEnabled(true); // Authorization Code
        client.setDirectAccessGrantsEnabled(false);
        client.setRedirectUris(List.of(req.frontendUrl() + "/*"));
        client.setWebOrigins(List.of(req.frontendUrl()));
        client.setAttributes(Map.of("pkce.code.challenge.method", "S256"));

        realm.clients().create(client);
        return client.getClientId();
    }

    private ClientCredentials criarClientBackend(RealmResource realm, NovoTenantRequest req) {
        ClientRepresentation client = new ClientRepresentation();
        client.setClientId(req.slug() + "-backend");
        client.setName(req.nomeExibicao() + " — Backend");
        client.setPublicClient(false);
        client.setServiceAccountsEnabled(true);  // Client Credentials Flow
        client.setStandardFlowEnabled(false);
        client.setSecret(UUID.randomUUID().toString());

        realm.clients().create(client);

        // Atribui papel para o service account do client
        ClientResource clientResource = realm.clients()
                .findByClientId(client.getClientId()).stream()
                .map(c -> realm.clients().get(c.getId()))
                .findFirst().orElseThrow();

        UserResource serviceAccount = realm.users()
                .get(clientResource.getServiceAccountUser().getId());
        serviceAccount.roles().realmLevel()
                .add(List.of(realm.roles().get("OPERADOR").toRepresentation()));

        return new ClientCredentials(client.getClientId(), client.getSecret());
    }

    private void criarAdminInicial(RealmResource realm, NovoTenantRequest req) {
        UserRepresentation user = new UserRepresentation();
        user.setUsername(req.adminEmail());
        user.setEmail(req.adminEmail());
        user.setFirstName(req.adminNome());
        user.setEnabled(true);
        user.setEmailVerified(false);

        // Senha temporária — usuário é forçado a trocar no primeiro acesso
        CredentialRepresentation senha = new CredentialRepresentation();
        senha.setType(CredentialRepresentation.PASSWORD);
        senha.setValue(gerarSenhaTemporaria());
        senha.setTemporary(true);
        user.setCredentials(List.of(senha));

        realm.users().create(user);

        // Atribui papel ADMIN_TENANT
        String userId = realm.users().search(req.adminEmail()).get(0).getId();
        realm.users().get(userId).roles().realmLevel()
                .add(List.of(realm.roles().get("ADMIN_TENANT").toRepresentation()));

        log.info("Admin inicial criado: email={}, realm={}", req.adminEmail(), realm);
    }
}
```

#### Integração de IdP Externo por Tenant (SSO Corporativo)

Cada tenant pode configurar seu próprio provedor de identidade. O Keycloak atua como broker: o usuário autentica no IdP do tenant (ex: Azure AD da ACME Corp) e o Keycloak emite um token JWT para as aplicações da plataforma.

```
Fluxo de SSO com Identity Broker:

  Usuário          Keycloak (Realm: acme)      Azure AD (ACME)
     │                     │                        │
     │── login ───────────►│                        │
     │◄── botão "Login     │                        │
     │    com ACME SSO" ───│                        │
     │── clica ───────────►│── redirect SAML/OIDC ─►│
     │                     │                        │── autenticação
     │                     │◄── assertion/token ────│   corporativa
     │                     │                        │   (MFA, AD Groups)
     │                     │── mapeia grupos AD      │
     │                     │   para papéis Keycloak  │
     │◄── JWT Keycloak ────│                        │
```

```java
@Service
@Slf4j
public class IdpConfiguracaoService {

    private final Keycloak keycloakAdmin;

    /**
     * Configura SSO via OIDC (ex: Google Workspace, Azure AD com OIDC)
     */
    public void configurarOidc(String realmId, ConfigurarOidcRequest req) {
        IdentityProviderRepresentation idp = new IdentityProviderRepresentation();
        idp.setAlias(req.alias());                  // ex: "azure-acme"
        idp.setDisplayName(req.nomeExibicao());     // ex: "Login com ACME"
        idp.setProviderId("oidc");
        idp.setEnabled(true);
        idp.setTrustEmail(true);                    // confia no e-mail verificado pelo IdP
        idp.setFirstBrokerLoginFlowAlias("first broker login");

        idp.setConfig(Map.of(
                "authorizationUrl",   req.authorizationEndpoint(),
                "tokenUrl",           req.tokenEndpoint(),
                "userInfoUrl",        req.userinfoEndpoint(),
                "jwksUrl",            req.jwksUri(),
                "clientId",           req.clientId(),
                "clientSecret",       req.clientSecret(),
                "defaultScope",       "openid email profile",
                "validateSignature",  "true",
                "useJwksUrl",         "true"
        ));

        keycloakAdmin.realm(realmId).identityProviders().create(idp);

        // Mapeador: sincroniza o e-mail do IdP com o atributo do usuário Keycloak
        IdentityProviderMapperRepresentation emailMapper = new IdentityProviderMapperRepresentation();
        emailMapper.setName("email-mapper");
        emailMapper.setIdentityProviderMapper("oidc-user-attribute-idp-mapper");
        emailMapper.setIdentityProviderAlias(req.alias());
        emailMapper.setConfig(Map.of(
                "syncMode",     "INHERIT",
                "claim",        "email",
                "user.attribute", "email"
        ));

        keycloakAdmin.realm(realmId).identityProviders()
                .get(req.alias()).addMapper(emailMapper);

        log.info("IdP OIDC configurado: realm={}, alias={}", realmId, req.alias());
    }

    /**
     * Configura SSO via SAML 2.0 (ex: ADFS, Okta, PingFederate)
     */
    public void configurarSaml(String realmId, ConfigurarSamlRequest req) {
        IdentityProviderRepresentation idp = new IdentityProviderRepresentation();
        idp.setAlias(req.alias());
        idp.setDisplayName(req.nomeExibicao());
        idp.setProviderId("saml");
        idp.setEnabled(true);
        idp.setTrustEmail(true);

        idp.setConfig(Map.of(
                "singleSignOnServiceUrl",    req.ssoUrl(),
                "singleLogoutServiceUrl",    req.sloUrl(),
                "nameIDPolicyFormat",        "urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress",
                "signingCertificate",        req.certificadoBase64(),
                "validateSignature",         "true",
                "wantAuthnRequestsSigned",   "false",
                "postBindingResponse",       "true",
                "postBindingAuthnRequest",   "true"
        ));

        keycloakAdmin.realm(realmId).identityProviders().create(idp);

        // Mapeador de grupos SAML → papéis Keycloak
        // ex: grupo "TI-Admins" no AD → papel "ADMIN_TENANT" no Keycloak
        req.mapeamentoGrupos().forEach((grupoAd, papelKeycloak) -> {
            IdentityProviderMapperRepresentation groupMapper = new IdentityProviderMapperRepresentation();
            groupMapper.setName("group-" + grupoAd.toLowerCase());
            groupMapper.setIdentityProviderMapper("saml-role-idp-mapper");
            groupMapper.setIdentityProviderAlias(req.alias());
            groupMapper.setConfig(Map.of(
                    "syncMode",         "INHERIT",
                    "attribute.value",  grupoAd,
                    "role",             papelKeycloak
            ));
            keycloakAdmin.realm(realmId).identityProviders()
                    .get(req.alias()).addMapper(groupMapper);
        });

        log.info("IdP SAML configurado: realm={}, alias={}", realmId, req.alias());
    }
}
```

#### Integração dos Microsserviços com Spring Security

Cada microsserviço valida o token JWT emitido pelo realm do tenant correspondente. O realm é resolvido dinamicamente a partir de um header ou subdomínio.

```yaml
# Configuração estática (realm fixo — para serviços internos)
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${KEYCLOAK_URL}/realms/plataforma-interna
          jwk-set-uri: ${KEYCLOAK_URL}/realms/plataforma-interna/protocol/openid-connect/certs
```

```java
// Resolução dinâmica de realm (multi-tenant)
// O tenant é identificado pelo subdomínio ou pelo header X-Tenant-ID
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class MultiTenantSecurityConfig {

    @Value("${keycloak.server-url}")
    private String keycloakServerUrl;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/actuator/health/**", "/api/v1/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2
                        .authenticationManagerResolver(multiTenantResolver())
                )
                .build();
    }

    @Bean
    public AuthenticationManagerResolver<HttpServletRequest> multiTenantResolver() {
        // Cache de JwtDecoder por realm — evita buscar JWKS a cada requisição
        Map<String, JwtDecoder> decoders = new ConcurrentHashMap<>();

        return request -> {
            String tenantId = resolverTenantId(request);
            JwtDecoder decoder = decoders.computeIfAbsent(tenantId, this::criarDecoder);
            return new JwtAuthenticationProvider(decoder)::authenticate;
        };
    }

    private String resolverTenantId(HttpServletRequest request) {
        // Estratégia 1: header explícito
        String header = request.getHeader("X-Tenant-ID");
        if (header != null && !header.isBlank()) return header;

        // Estratégia 2: subdomínio (acme.plataforma.com → "acme")
        String host = request.getServerName();
        if (host.contains(".")) return host.split("\\.")[0];

        throw new TenantNaoIdentificadoException("Tenant não identificado na requisição");
    }

    private JwtDecoder criarDecoder(String tenantId) {
        String issuerUri = keycloakServerUrl + "/realms/" + tenantId;
        return JwtDecoders.fromIssuerLocation(issuerUri);
    }
}
```

```java
// Extrator de contexto do tenant a partir do JWT
@Component
public class TenantContextHolder {

    private static final ThreadLocal<TenantContext> context = new ThreadLocal<>();

    public static void set(TenantContext ctx) { context.set(ctx); }
    public static TenantContext get()         { return context.get(); }
    public static void clear()               { context.remove(); }
}

public record TenantContext(
        String tenantId,
        String usuarioId,
        String email,
        Set<String> papeis
) {
    public static TenantContext from(Jwt jwt) {
        String issuer  = jwt.getIssuer().toString();
        String tenantId = issuer.substring(issuer.lastIndexOf('/') + 1);

        @SuppressWarnings("unchecked")
        List<String> roles = (List<String>) ((Map<?, ?>) jwt.getClaim("realm_access"))
                .getOrDefault("roles", List.of());

        return new TenantContext(
                tenantId,
                jwt.getSubject(),
                jwt.getClaimAsString("email"),
                new HashSet<>(roles)
        );
    }
}
```

```java
// Filtro que popula o TenantContext em cada requisição
@Component
@Order(Ordered.HIGHEST_PRECEDENCE + 10)
public class TenantContextFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {
        try {
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth instanceof JwtAuthenticationToken jwtAuth) {
                TenantContextHolder.set(TenantContext.from(jwtAuth.getToken()));
            }
            chain.doFilter(request, response);
        } finally {
            TenantContextHolder.clear(); // CRÍTICO: evitar vazamento entre threads
        }
    }
}
```

#### Autorização Baseada em Papéis e Recursos

```java
// Controle de acesso declarativo com @PreAuthorize
@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioController {

    private final UsuarioService service;

    @GetMapping
    @PreAuthorize("hasRole('ADMIN_TENANT') or hasRole('GERENTE')")
    public Page<UsuarioResponse> listar(Pageable pageable) {
        return service.listar(TenantContextHolder.get().tenantId(), pageable);
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN_TENANT')")
    @ResponseStatus(HttpStatus.CREATED)
    public UsuarioResponse criar(@RequestBody @Valid CriarUsuarioRequest request) {
        return service.criar(TenantContextHolder.get().tenantId(), request);
    }

    @PutMapping("/{usuarioId}/papeis")
    @PreAuthorize("hasRole('ADMIN_TENANT')")
    public void atribuirPapeis(
            @PathVariable String usuarioId,
            @RequestBody @Valid AtribuirPapeisRequest request) {
        service.atribuirPapeis(TenantContextHolder.get().tenantId(), usuarioId, request);
    }

    @DeleteMapping("/{usuarioId}")
    @PreAuthorize("hasRole('ADMIN_TENANT') or #usuarioId == authentication.name")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void remover(@PathVariable String usuarioId) {
        service.remover(TenantContextHolder.get().tenantId(), usuarioId);
    }
}
```

```java
// Serviço que gerencia usuários via Keycloak Admin API
@Service
@Slf4j
public class UsuarioService {

    private final Keycloak keycloakAdmin;

    public UsuarioResponse criar(String tenantId, CriarUsuarioRequest req) {
        RealmResource realm = keycloakAdmin.realm(tenantId);

        UserRepresentation user = new UserRepresentation();
        user.setUsername(req.email());
        user.setEmail(req.email());
        user.setFirstName(req.nome());
        user.setLastName(req.sobrenome());
        user.setEnabled(true);

        // Atributos customizados do tenant
        user.setAttributes(Map.of(
                "tenant_id",   List.of(tenantId),
                "departamento", List.of(req.departamento()),
                "cargo",       List.of(req.cargo())
        ));

        Response resp = realm.users().create(user);
        if (resp.getStatus() == 409) {
            throw new UsuarioDuplicadoException(req.email());
        }

        String userId = extrairIdDaResponse(resp);

        // Atribui papéis
        List<RoleRepresentation> papeis = req.papeis().stream()
                .map(papel -> realm.roles().get(papel).toRepresentation())
                .toList();
        realm.users().get(userId).roles().realmLevel().add(papeis);

        // Envia e-mail de definição de senha (fluxo nativo do Keycloak)
        realm.users().get(userId)
                .executeActionsEmail(List.of("UPDATE_PASSWORD", "VERIFY_EMAIL"));

        log.info("Usuário criado: email={}, tenant={}", req.email(), tenantId);
        return buscarPorId(tenantId, userId);
    }

    public void atribuirPapeis(String tenantId, String usuarioId,
                                AtribuirPapeisRequest req) {
        RealmResource realm = keycloakAdmin.realm(tenantId);
        UserResource user = realm.users().get(usuarioId);

        // Remove papéis de negócio atuais
        List<String> papeisDenegocio = List.of("ADMIN_TENANT", "GERENTE", "OPERADOR", "LEITURA");
        List<RoleRepresentation> atuais = user.roles().realmLevel().listAll().stream()
                .filter(r -> papeisDenegocio.contains(r.getName()))
                .toList();
        user.roles().realmLevel().remove(atuais);

        // Atribui novos papéis
        List<RoleRepresentation> novos = req.papeis().stream()
                .map(p -> realm.roles().get(p).toRepresentation())
                .toList();
        user.roles().realmLevel().add(novos);

        log.info("Papéis atualizados: userId={}, tenant={}, papeis={}",
                usuarioId, tenantId, req.papeis());
    }

    private String extrairIdDaResponse(Response resp) {
        String location = resp.getHeaderString("Location");
        return location.substring(location.lastIndexOf('/') + 1);
    }
}
```

#### Gerenciamento de Sessões e Logout Global (SSO Logout)

```java
// Logout do usuário em todos os devices e aplicações do realm
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {

    private final Keycloak keycloakAdmin;

    // Logout global: invalida todas as sessões do usuário no realm
    @PostMapping("/logout-global")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void logoutGlobal(@AuthenticationPrincipal Jwt jwt) {
        String tenantId = TenantContextHolder.get().tenantId();
        String usuarioId = jwt.getSubject();

        keycloakAdmin.realm(tenantId).users()
                .get(usuarioId).logout(); // revoga todas as sessões ativas

        log.info("Logout global executado: userId={}, tenant={}", usuarioId, tenantId);
    }

    // Desabilitar usuário imediatamente (ex: demissão, comprometimento de conta)
    @PostMapping("/usuarios/{usuarioId}/desabilitar")
    @PreAuthorize("hasRole('ADMIN_TENANT')")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void desabilitar(@PathVariable String usuarioId) {
        String tenantId = TenantContextHolder.get().tenantId();
        RealmResource realm = keycloakAdmin.realm(tenantId);

        // Desabilita a conta
        UserRepresentation user = realm.users().get(usuarioId).toRepresentation();
        user.setEnabled(false);
        realm.users().get(usuarioId).update(user);

        // Encerra imediatamente todas as sessões ativas
        realm.users().get(usuarioId).logout();

        log.info("Usuário desabilitado e sessões encerradas: userId={}, tenant={}",
                usuarioId, tenantId);
    }
}
```

#### Eventos do Keycloak — Auditoria e Reação a Mudanças

```java
// Listener de eventos Keycloak via webhook ou SPI
// Keycloak pode publicar eventos em uma fila — consumimos com Spring
@Component
@Slf4j
public class KeycloakEventConsumer {

    private final AuditoriaRepository auditoriaRepo;
    private final NotificacaoService notificacaoService;

    @KafkaListener(topics = "keycloak.events", groupId = "auditoria-service")
    public void processar(KeycloakEventMessage evento) {
        // Persiste para auditoria
        auditoriaRepo.save(new RegistroAuditoria(
                evento.realmId(),
                evento.userId(),
                evento.type(),
                evento.ipAddress(),
                evento.timestamp(),
                evento.details()
        ));

        // Reage a eventos relevantes
        switch (evento.type()) {
            case "LOGIN_ERROR" -> {
                int tentativas = auditoriaRepo.contarErrosRecentes(
                        evento.userId(), Duration.ofMinutes(15));
                if (tentativas >= 5) {
                    // Notifica admin do tenant sobre possível ataque de força bruta
                    notificacaoService.alertarAdmin(evento.realmId(),
                            "Múltiplas tentativas de login falhas para: " + evento.userId());
                }
            }
            case "UPDATE_PASSWORD" ->
                log.info("Senha atualizada: userId={}, tenant={}",
                        evento.userId(), evento.realmId());

            case "REGISTER" ->
                // Dispara onboarding do novo usuário
                notificacaoService.enviarBoasVindas(evento.realmId(), evento.userId());
        }
    }
}
```

#### Infraestrutura — Keycloak em Produção

```yaml
# docker-compose para ambiente de desenvolvimento
services:
  keycloak:
    image: quay.io/keycloak/keycloak:24.0
    command: start-dev --import-realm
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ${KC_ADMIN_PASSWORD}
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: ${KC_DB_USER}
      KC_DB_PASSWORD: ${KC_DB_PASSWORD}
      KC_HOSTNAME: ${KC_HOSTNAME:localhost}
      KC_HTTP_ENABLED: "true"
      KC_PROXY: edge           # TLS termina no nginx/load balancer
      KC_CACHE: ispn           # Infinispan para cluster multi-nó
      KC_CACHE_STACK: kubernetes
    ports:
      - "8080:8080"
    volumes:
      - ./keycloak/realms:/opt/keycloak/data/import  # realm base para dev
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: ${KC_DB_USER}
      POSTGRES_PASSWORD: ${KC_DB_PASSWORD}
    volumes:
      - kc_postgres_data:/var/lib/postgresql/data

volumes:
  kc_postgres_data:
```

```yaml
# Dependência Spring Boot
# pom.xml (equivalente)
# spring-boot-starter-oauth2-resource-server  →  valida tokens
# spring-boot-starter-oauth2-client           →  flows de login (frontend BFF)
# keycloak-admin-client                       →  Admin API
```

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <!-- Admin API do Keycloak -->
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-admin-client</artifactId>
        <version>24.0.3</version>
    </dependency>
</dependencies>
```

```java
// Bean do cliente Admin API
@Configuration
public class KeycloakAdminConfig {

    @Bean
    public Keycloak keycloakAdmin(
            @Value("${keycloak.server-url}")    String serverUrl,
            @Value("${keycloak.admin.realm}")   String adminRealm,
            @Value("${keycloak.admin.client-id}") String clientId,
            @Value("${keycloak.admin.client-secret}") String clientSecret) {

        return KeycloakBuilder.builder()
                .serverUrl(serverUrl)
                .realm(adminRealm)           // "master" realm para administração
                .grantType(OAuth2Constants.CLIENT_CREDENTIALS)
                .clientId(clientId)
                .clientSecret(clientSecret)
                .build();
    }
}
```

#### Decisões Arquiteturais Consolidadas

| Decisão | Escolha | Motivação |
|---------|---------|-----------|
| Estratégia multi-tenant | Um realm por tenant | Isolamento total de usuários, papéis, políticas e IdPs |
| Provisionamento de tenant | Admin API programática | Onboarding automatizado sem intervenção manual |
| Resolução de tenant na API | Header `X-Tenant-ID` ou subdomínio | Flexível; funciona com SPA e apps mobile |
| Cache de JwtDecoder | `ConcurrentHashMap` por realm | Evita fetch do JWKS a cada requisição; TTL implícito por restart |
| Propagação de contexto | `ThreadLocal` limpo no `finally` | Evita vazamento de contexto entre requisições no pool de threads |
| SSO com IdP externo | Keycloak como identity broker | Aplicações só conhecem o Keycloak; troca de IdP não afeta o código |
| Logout global | `realm.users().get(id).logout()` | Revoga todas as sessões ativas em todos os clients do realm |
| Auditoria de eventos | Keycloak Event SPI → Kafka | Eventos centralizados; desacoplado; permite análise de segurança em tempo real |
| MFA | Configurado por realm via políticas | Cada tenant pode exigir TOTP/WebAuthn conforme sua política de segurança |

---

### 18.9 Chat em Tempo Real

#### Contexto

Uma plataforma de comunicação (estilo WhatsApp ou Slack) precisa entregar mensagens em tempo real entre usuários, garantir que nenhuma mensagem seja perdida mesmo quando o destinatário está offline, manter histórico persistente, exibir status de presença (online/digitando/ausente) e escalar para milhões de conexões simultâneas.

#### Requisitos-Chave

```
Funcionais:
  - Mensagens em tempo real entre dois usuários ou em grupos
  - Status de entrega: enviada, entregue, lida (double check)
  - Presença: online, ausente, digitando...
  - Histórico de mensagens paginado
  - Suporte a texto, imagens e arquivos (link para o objeto no S3)

Não-Funcionais:
  - Latência de entrega < 100ms na mesma região
  - Garantia de entrega: mensagens offline entregues ao reconectar
  - Ordenação garantida dentro de uma conversa
  - Escala para 1 milhão de conexões WebSocket simultâneas por nó
```

#### Arquitetura

```
  Cliente A            Chat Service (nó 1)      Chat Service (nó 2)         Cliente B
     │                        │                        │                        │
     │── WS connect ─────────►│                        │◄─── WS connect ────────│
     │                   sessão A                  sessão B                     │
     │                        │                        │                        │
     │── msg para B ─────────►│                        │                        │
     │                        │── pub Kafka ───────────►│                        │
     │                        │   (sala/conversa_id)    │── push WS ────────────►│
     │                        │                        │                        │
     │                   PostgreSQL (histórico persistido por ambos os nós)
     │                   Redis (presença, sessões WebSocket por usuário)
```

**Problema central:** cliente A está conectado ao nó 1 e cliente B ao nó 2. A mensagem precisa cruzar nós. O Kafka (ou Redis Pub/Sub) faz o roteamento entre instâncias.

#### Decisões Arquiteturais

| Decisão | Escolha | Motivação |
|---------|---------|-----------|
| Protocolo | WebSocket (STOMP sobre WS) | Bidirecional; baixo overhead vs. long polling |
| Roteamento entre nós | Kafka por tópico de conversa | Particionamento por `conversa_id` garante ordem |
| Persistência de mensagens | PostgreSQL com particionamento por mês | Histórico durável; partições facilitam archiving |
| Presença e sessões | Redis com TTL e heartbeat | Expiração automática quando conexão cai |
| Entrega offline | Fila pessoal por usuário no Kafka | Mensagens acumulam enquanto offline; entregues ao reconectar |
| Ordenação | Sequence number monotônico por conversa | Detecta lacunas e permite reordenação no cliente |

#### Implementação

```java
// Configuração WebSocket com STOMP
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // Tópicos que o servidor publica — clientes se inscrevem
        registry.enableSimpleBroker("/topic", "/queue");
        // Prefixo para mensagens enviadas pelo cliente ao servidor
        registry.setApplicationDestinationPrefixes("/app");
        // Fila pessoal (ponto a ponto): /user/{userId}/queue/mensagens
        registry.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/chat")
                .setAllowedOriginPatterns("*")
                .withSockJS(); // fallback para navegadores sem WS nativo
    }

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration reg) {
        reg.setMessageSizeLimit(128 * 1024)   // 128 KB por mensagem
           .setSendBufferSizeLimit(512 * 1024) // buffer de envio por sessão
           .setSendTimeLimit(20_000);          // timeout de envio
    }
}
```

```java
// Mensagem de domínio
public record MensagemChat(
        UUID id,
        UUID conversaId,
        String remetenteId,
        String conteudo,
        TipoMensagem tipo,       // TEXTO, IMAGEM, ARQUIVO, SISTEMA
        long sequencia,          // número de sequência dentro da conversa
        Instant enviadaEm,
        StatusMensagem status    // ENVIADA, ENTREGUE, LIDA
) {}
```

```java
@Controller
@Slf4j
public class ChatController {

    private final MensagemService mensagemService;
    private final PresencaService presencaService;
    private final SimpMessagingTemplate messagingTemplate;

    // Cliente envia mensagem: /app/chat/mensagem
    @MessageMapping("/chat/mensagem")
    public void enviar(
            @Payload EnviarMensagemRequest request,
            @AuthenticationPrincipal Jwt jwt) {

        MensagemChat mensagem = mensagemService.persistirEPublicar(
                request, jwt.getSubject());

        // Confirma ao remetente que a mensagem foi recebida pelo servidor
        messagingTemplate.convertAndSendToUser(
                jwt.getSubject(), "/queue/acks",
                new AckMensagem(mensagem.id(), mensagem.sequencia(), StatusMensagem.ENVIADA));
    }

    // Cliente notifica que leu mensagens: /app/chat/leitura
    @MessageMapping("/chat/leitura")
    public void marcarLida(
            @Payload MarcarLidaRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        mensagemService.marcarComoLida(request.conversaId(), request.ate(), jwt.getSubject());
    }

    // Cliente notifica que está digitando: /app/chat/digitando
    @MessageMapping("/chat/digitando")
    public void digitando(
            @Payload DigitandoEvent event,
            @AuthenticationPrincipal Jwt jwt) {

        // Propaga para os outros participantes da conversa
        mensagemService.obterParticipantes(event.conversaId())
                .stream()
                .filter(id -> !id.equals(jwt.getSubject()))
                .forEach(participanteId ->
                        messagingTemplate.convertAndSendToUser(
                                participanteId, "/queue/digitando",
                                new DigitandoNotificacao(jwt.getSubject(), event.conversaId())));
    }
}
```

```java
@Service
@Slf4j
public class MensagemService {

    private final MensagemRepository mensagemRepo;
    private final ConversaRepository conversaRepo;
    private final KafkaTemplate<String, MensagemChat> kafka;
    private final SimpMessagingTemplate messagingTemplate;

    @Transactional
    public MensagemChat persistirEPublicar(EnviarMensagemRequest req, String remetenteId) {
        Conversa conversa = conversaRepo.findById(req.conversaId()).orElseThrow();

        // Sequence number atômico por conversa (evita lacunas)
        long seq = conversaRepo.incrementarSequencia(req.conversaId());

        MensagemChat msg = new MensagemChat(
                UUID.randomUUID(), req.conversaId(), remetenteId,
                req.conteudo(), req.tipo(), seq, Instant.now(), StatusMensagem.ENVIADA);

        mensagemRepo.save(Mensagem.from(msg));

        // Publica no Kafka — partição = conversaId (garante ordem)
        kafka.send("chat.mensagens", req.conversaId().toString(), msg);

        return msg;
    }

    @KafkaListener(topics = "chat.mensagens", groupId = "chat-delivery")
    public void entregarMensagem(MensagemChat msg, Acknowledgment ack) {
        List<String> participantes = obterParticipantes(msg.conversaId());

        for (String participanteId : participantes) {
            if (participanteId.equals(msg.remetenteId())) continue;

            // Tenta entregar via WebSocket (se online)
            try {
                messagingTemplate.convertAndSendToUser(
                        participanteId, "/queue/mensagens", msg);
                log.debug("Mensagem entregue via WS: para={}, msgId={}", participanteId, msg.id());
            } catch (Exception e) {
                // Usuário offline — mensagem já está no histórico; será carregada ao reconectar
                log.debug("Usuário offline, mensagem persistida: userId={}", participanteId);
            }
        }
        ack.acknowledge();
    }
}
```

```java
// Presença — online, ausente, digitando
@Service
public class PresencaService {

    private final StringRedisTemplate redis;
    private static final Duration TTL_PRESENCA = Duration.ofSeconds(30);

    // Heartbeat do cliente a cada 20s mantém presença ativa
    public void registrarOnline(String userId) {
        redis.opsForValue().set("presenca:" + userId, "online", TTL_PRESENCA);
    }

    public void registrarOffline(String userId) {
        redis.delete("presenca:" + userId);
        redis.opsForValue().set(
                "presenca:ultimo-acesso:" + userId,
                Instant.now().toString());
    }

    public StatusPresenca obter(String userId) {
        String status = redis.opsForValue().get("presenca:" + userId);
        if ("online".equals(status)) return StatusPresenca.ONLINE;

        String ultimoAcesso = redis.opsForValue().get("presenca:ultimo-acesso:" + userId);
        return new StatusPresenca(StatusPresenca.Tipo.OFFLINE,
                ultimoAcesso != null ? Instant.parse(ultimoAcesso) : null);
    }
}
```

```java
// Interceptor WebSocket — registra conexão/desconexão de usuários
@Component
public class WebSocketPresencaInterceptor implements ChannelInterceptor {

    private final PresencaService presencaService;

    @Override
    public void postSend(Message<?> message, MessageChannel channel, boolean sent) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        String userId = obterUserId(accessor);
        if (userId == null) return;

        switch (accessor.getCommand()) {
            case CONNECT    -> presencaService.registrarOnline(userId);
            case DISCONNECT -> presencaService.registrarOffline(userId);
            default         -> {}
        }
    }
}
```

```java
// Histórico paginado — carregado ao abrir a conversa
@RestController
@RequestMapping("/api/v1/conversas")
public class ConversaController {

    private final MensagemRepository mensagemRepo;

    // Paginação por cursor (sequência) — mais eficiente que offset para histórico
    @GetMapping("/{conversaId}/mensagens")
    public List<MensagemChat> historico(
            @PathVariable UUID conversaId,
            @RequestParam(required = false) Long antes,   // cursor: sequência anterior
            @RequestParam(defaultValue = "50") int limite) {

        return antes != null
                ? mensagemRepo.findByConversaIdAndSequenciaLessThan(conversaId, antes,
                        PageRequest.of(0, limite, Sort.by("sequencia").descending()))
                : mensagemRepo.findByConversaIdOrderBySequenciaDesc(conversaId,
                        PageRequest.of(0, limite));
    }
}
```

---

### 18.10 Geolocalização e Rastreamento em Tempo Real

#### Contexto

Uma plataforma de mobilidade (estilo Uber ou iFood) precisa rastrear a posição de entregadores/motoristas em tempo real, exibir profissionais próximos ao cliente, calcular ETAs e notificar o cliente com a posição atualizada durante o trajeto.

#### Requisitos-Chave

```
Funcionais:
  - Entregador publica sua posição a cada 5 segundos
  - Cliente vê entregadores disponíveis no raio de X km
  - Cliente acompanha posição do seu entregador em tempo real
  - Plataforma calcula ETA baseado na posição atual e rota

Não-Funcionais:
  - 100.000 entregadores atualizando posição a cada 5s
    = 20.000 atualizações/segundo
  - Busca por proximidade em < 50ms
  - Histórico de rotas para auditoria e análise
```

#### Arquitetura

```
  App Entregador              Location Service             App Cliente
       │                            │                           │
       │── PUT posição (5s) ───────►│                           │
       │                            │── Redis GEO update        │
       │                            │── Kafka pub               │
       │                            │── PostgreSQL (histórico)   │
       │                            │                           │
       │                            │◄── GET /proximos ─────────│
       │                            │    (lat, lng, raio)       │
       │                            │── Redis GEO radius ──────►│
       │                            │                           │
       │                            │◄── SSE /rastreamento ─────│
       │                            │    (conversaId pedido)    │
       │                            │── push SSE posição ──────►│
```

#### Por que Redis GEO?

```
Redis GEOADD armazena coordenadas como score em um Sorted Set,
codificado com Geohash de 52 bits.

Operações disponíveis:
  GEOADD   entregadores <lng> <lat> <id>   → O(log N)
  GEODIST  entregadores A B km             → O(log N)
  GEOSEARCH entregadores FROMLONLAT <lng> <lat>
            BYRADIUS 5 km ASC COUNT 20     → O(N+log M)

Para 100.000 entregadores, GEOSEARCH em raio de 5km retorna
em < 5ms — inviável com SQL convencional.
```

#### Implementação

```java
@Service
@Slf4j
public class LocationService {

    private final StringRedisTemplate redis;
    private final KafkaTemplate<String, PosicaoEvent> kafka;
    private final PosicaoHistoricoRepository historicoRepo;
    private final SseEmitterManager sseManager;

    private static final String GEO_KEY_DISPONIVEIS = "geo:entregadores:disponiveis";
    private static final String GEO_KEY_PREFIX_PEDIDO = "geo:entregador:pedido:";

    @Async
    public void atualizarPosicao(AtualizarPosicaoRequest req) {
        // 1. Atualiza posição no Redis GEO (para busca por proximidade)
        redis.opsForGeo().add(
                GEO_KEY_DISPONIVEIS,
                new RedisGeoCommands.GeoLocation<>(
                        req.entregadorId(),
                        new Point(req.longitude(), req.latitude())));

        // 2. Se em entrega ativa, atualiza chave específica do pedido (para rastreamento)
        if (req.pedidoId() != null) {
            redis.opsForGeo().add(
                    GEO_KEY_PREFIX_PEDIDO + req.pedidoId(),
                    new RedisGeoCommands.GeoLocation<>(
                            req.entregadorId(),
                            new Point(req.longitude(), req.latitude())));

            // Empurra via SSE para o cliente que está acompanhando este pedido
            sseManager.publicar("rastreamento:" + req.pedidoId(),
                    new PosicaoAtualizada(req.entregadorId(), req.latitude(),
                            req.longitude(), Instant.now()));
        }

        // 3. Publica no Kafka para histórico e analytics
        kafka.send("location.posicoes", req.entregadorId(),
                new PosicaoEvent(req.entregadorId(), req.pedidoId(),
                        req.latitude(), req.longitude(), Instant.now()));
    }

    public List<EntregadorProximo> buscarProximos(
            double latitude, double longitude, double raioKm, int limite) {

        GeoResults<RedisGeoCommands.GeoLocation<String>> results =
                redis.opsForGeo().search(
                        GEO_KEY_DISPONIVEIS,
                        GeoReference.fromCoordinate(longitude, latitude),
                        new Distance(raioKm, RedisGeoCommands.DistanceUnit.KILOMETERS),
                        RedisGeoCommands.GeoSearchCommandArgs.newGeoSearchArgs()
                                .includeDistance()
                                .includeCoordinates()
                                .sortAscending()
                                .limit(limite));

        return results.getContent().stream()
                .map(r -> new EntregadorProximo(
                        r.getContent().getName(),
                        r.getContent().getPoint().getY(),  // latitude
                        r.getContent().getPoint().getX(),  // longitude
                        r.getDistance().getValue()))
                .toList();
    }

    // Remove entregador da listagem quando fica indisponível
    public void marcarIndisponivel(String entregadorId) {
        redis.opsForGeo().remove(GEO_KEY_DISPONIVEIS, entregadorId);
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/location")
public class LocationController {

    private final LocationService locationService;
    private final SseEmitterManager sseManager;

    // Entregador atualiza posição (chamado a cada 5s pelo app)
    @PutMapping("/posicao")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void atualizar(
            @RequestBody @Valid AtualizarPosicaoRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        request = request.comEntregadorId(jwt.getSubject());
        locationService.atualizarPosicao(request);
    }

    // Cliente busca entregadores disponíveis próximos
    @GetMapping("/proximos")
    public List<EntregadorProximo> buscarProximos(
            @RequestParam double lat,
            @RequestParam double lng,
            @RequestParam(defaultValue = "5.0") double raioKm,
            @RequestParam(defaultValue = "20") int limite) {
        return locationService.buscarProximos(lat, lng, raioKm, limite);
    }

    // Cliente se inscreve no rastreamento em tempo real do seu pedido via SSE
    @GetMapping(value = "/rastreamento/{pedidoId}",
                produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter rastrear(
            @PathVariable UUID pedidoId,
            @AuthenticationPrincipal Jwt jwt) {

        // Valida que o JWT pertence ao dono do pedido
        locationService.verificarAcessoPedido(pedidoId, jwt.getSubject());

        SseEmitter emitter = new SseEmitter(Duration.ofHours(2).toMillis());
        sseManager.registrar("rastreamento:" + pedidoId, emitter);
        return emitter;
    }
}
```

```java
// Persistência do histórico de rotas para auditoria e análise
@Component
public class PosicaoHistoricoConsumer {

    private final PosicaoHistoricoRepository repository;

    @KafkaListener(topics = "location.posicoes", groupId = "historico-location")
    public void persistir(PosicaoEvent evento, Acknowledgment ack) {
        // Usa tabela particionada por data no PostgreSQL com PostGIS
        repository.save(new PosicaoHistorico(
                evento.entregadorId(),
                evento.pedidoId(),
                evento.latitude(),
                evento.longitude(),
                evento.timestamp()));
        ack.acknowledge();
    }
}
```

```sql
-- Schema PostgreSQL com PostGIS para histórico e queries espaciais
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE posicao_historico (
    id          BIGSERIAL,
    entregador_id VARCHAR(36) NOT NULL,
    pedido_id   UUID,
    coordenada  GEOGRAPHY(POINT, 4326) NOT NULL,  -- lon/lat WGS84
    registrado_em TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (id, registrado_em)
) PARTITION BY RANGE (registrado_em);

-- Índice espacial para queries de proximidade no histórico
CREATE INDEX idx_posicao_geo ON posicao_historico USING GIST (coordenada);
CREATE INDEX idx_posicao_entregador ON posicao_historico (entregador_id, registrado_em DESC);

-- Partição mensal (criada automaticamente via pg_partman ou job)
CREATE TABLE posicao_historico_2025_01
    PARTITION OF posicao_historico
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

---

### 18.11 Feed de Atividades e Timeline

#### Contexto

Uma rede social ou plataforma colaborativa (estilo LinkedIn, Twitter/X ou Instagram) precisa montar o feed de cada usuário com as publicações das pessoas e páginas que ele segue. O maior desafio arquitetural é a escala de escrita: quando uma celebridade com 10 milhões de seguidores publica algo, como entregar essa publicação para todos os feeds?

#### O Problema do Fan-Out

```
Fan-out on Write (Push):
  Ao publicar, escreve imediatamente no feed de cada seguidor.
  Leitura é instantânea (feed pré-computado).
  Problema: 1 publicação × 10M seguidores = 10M escritas simultâneas.

Fan-out on Read (Pull):
  Ao publicar, salva apenas uma vez.
  Ao ler o feed, agrega publicações de todos que o usuário segue.
  Problema: seguir 500 pessoas = 500 queries por carregamento de feed.

Solução Híbrida (usada por Twitter/X e Instagram):
  - Usuários comuns (< 10K seguidores): fan-out on write.
  - Celebridades (> 10K seguidores): fan-out on read sob demanda.
  - Feed = cache (fan-out write) + merge ao ler (celebrities).
```

```
Publicação de post:

  Autor (10K seg.)                   Autor (10M seg.)
       │ fan-out write                     │ apenas persiste
       ▼                                   ▼
  Feed Redis de                      Post Storage
  cada seguidor                           │
  (write imediato)               ao ler o feed do seguidor:
                                   feed cache + merge
                                   com posts de celebridades
```

#### Implementação

```java
@Service
@Slf4j
public class PostService {

    private final PostRepository postRepo;
    private final SeguidorRepository seguidorRepo;
    private final KafkaTemplate<String, PostPublicadoEvent> kafka;

    @Transactional
    public Post publicar(PublicarPostRequest req, String autorId) {
        Post post = postRepo.save(new Post(autorId, req.conteudo(), req.midia()));

        // Determina estratégia de fan-out conforme número de seguidores
        long totalSeguidores = seguidorRepo.contarSeguidores(autorId);
        FanOutStrategy estrategia = totalSeguidores > 10_000
                ? FanOutStrategy.ON_READ   // celebridade
                : FanOutStrategy.ON_WRITE; // usuário comum

        kafka.send("feed.posts-publicados",
                autorId,
                new PostPublicadoEvent(post.getId(), autorId,
                        post.getConteudo(), post.getCriadoEm(),
                        estrategia, totalSeguidores));

        log.info("Post publicado: id={}, autor={}, estrategia={}", post.getId(), autorId, estrategia);
        return post;
    }
}
```

```java
@Component
@Slf4j
public class FeedFanOutConsumer {

    private final StringRedisTemplate redis;
    private final SeguidorRepository seguidorRepo;

    private static final String FEED_KEY = "feed:usuario:";
    private static final int MAX_FEED_SIZE = 1000; // máximo de posts no cache por usuário

    @KafkaListener(topics = "feed.posts-publicados", groupId = "feed-fanout")
    public void processar(PostPublicadoEvent evento, Acknowledgment ack) {
        if (evento.estrategia() == FanOutStrategy.ON_WRITE) {
            fanOutWrite(evento);
        }
        // ON_READ: nenhuma ação aqui — o post já está no banco
        ack.acknowledge();
    }

    private void fanOutWrite(PostPublicadoEvent evento) {
        // Busca seguidores em lotes para não sobrecarregar a memória
        int pagina = 0;
        int tamanhoPagina = 500;
        List<String> seguidores;

        do {
            seguidores = seguidorRepo.buscarSeguidores(
                    evento.autorId(), pagina++, tamanhoPagina);

            // Pipeline Redis: todas as escritas em lote (reduz round-trips)
            redis.executePipelined((RedisCallback<Object>) conn -> {
                for (String seguidorId : seguidores) {
                    String feedKey = FEED_KEY + seguidorId;
                    // ZADD feed:<userId> <timestamp> <postId>
                    conn.zSetCommands().zAdd(
                            feedKey.getBytes(),
                            (double) evento.criadoEm().toEpochMilli(),
                            evento.postId().toString().getBytes());
                    // Mantém apenas os 1000 posts mais recentes
                    conn.zSetCommands().zRemRange(feedKey.getBytes(), 0, -(MAX_FEED_SIZE + 1));
                }
                return null;
            });

            log.debug("Fan-out write: postId={}, pagina={}, seguidores={}",
                    evento.postId(), pagina - 1, seguidores.size());

        } while (seguidores.size() == tamanhoPagina);
    }
}
```

```java
@Service
public class FeedService {

    private final StringRedisTemplate redis;
    private final PostRepository postRepo;
    private final SeguidorRepository seguidorRepo;

    private static final String FEED_KEY = "feed:usuario:";
    private static final String CELEB_KEY = "celebridades:seguidas:";

    public List<PostResponse> obterFeed(String usuarioId, long cursorTs, int limite) {
        // 1. Busca IDs do feed pré-computado (fan-out write) via Redis ZREVRANGEBYSCORE
        Set<String> idsFanOut = redis.opsForZSet().reverseRangeByScore(
                FEED_KEY + usuarioId,
                Double.NEGATIVE_INFINITY,
                cursorTs > 0 ? cursorTs - 1 : Double.MAX_VALUE,
                0, limite);

        List<UUID> postIds = idsFanOut.stream().map(UUID::fromString).toList();

        // 2. Merge com posts de celebridades (fan-out on read)
        List<String> celebridades = redis.opsForList().range(CELEB_KEY + usuarioId, 0, -1);
        if (celebridades != null && !celebridades.isEmpty()) {
            List<Post> postsCelebs = postRepo.findByCelebridadesApos(
                    celebridades, cursorTs > 0 ? Instant.ofEpochMilli(cursorTs) : Instant.now(),
                    limite);
            // Merge e ordena por timestamp
            List<UUID> idsCelebs = postsCelebs.stream().map(Post::getId).toList();
            postIds = merge(postIds, idsCelebs, limite);
        }

        if (postIds.isEmpty()) return List.of();

        // 3. Busca os dados completos dos posts (com cache Redis por post)
        return postRepo.findAllById(postIds).stream()
                .sorted(Comparator.comparing(Post::getCriadoEm).reversed())
                .map(PostResponse::from)
                .toList();
    }

    private List<UUID> merge(List<UUID> a, List<UUID> b, int limite) {
        return Stream.concat(a.stream(), b.stream())
                .distinct()
                .limit(limite)
                .toList();
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/feed")
public class FeedController {

    private final FeedService feedService;
    private final PostService postService;

    // Leitura do feed com paginação por cursor (timestamp)
    @GetMapping
    public List<PostResponse> obter(
            @RequestParam(defaultValue = "0") long antes,
            @RequestParam(defaultValue = "20") int limite,
            @AuthenticationPrincipal Jwt jwt) {
        return feedService.obterFeed(jwt.getSubject(), antes, Math.min(limite, 50));
    }

    @PostMapping("/posts")
    @ResponseStatus(HttpStatus.CREATED)
    public PostResponse publicar(
            @RequestBody @Valid PublicarPostRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        return PostResponse.from(postService.publicar(request, jwt.getSubject()));
    }
}
```

---

### 18.12 Webhooks para Clientes Externos

#### Contexto

Uma plataforma SaaS (estilo Stripe, GitHub ou PagSeguro) precisa notificar sistemas de terceiros quando eventos relevantes ocorrem (pagamento confirmado, pedido atualizado, build concluído). O cliente externo registra uma URL e a plataforma faz chamadas HTTP para ela. O desafio é garantir a entrega com retry, segurança da autenticidade e rastreabilidade.

#### Requisitos-Chave

```
Funcionais:
  - Cliente registra URLs de destino por tipo de evento
  - Plataforma entrega o evento via HTTP POST para cada URL
  - Retry automático com backoff exponencial em caso de falha
  - Portal para visualizar tentativas, status e reenvio manual
  - Segurança: assinatura HMAC para autenticação do payload

Não-Funcionais:
  - Entrega best-effort com garantia via retry (até 24h)
  - Timeout de 10s por tentativa — receptor deve responder rápido
  - Desativação automática de endpoints que falham por > 3 dias
  - Sem bloquear o fluxo principal da aplicação
```

#### Arquitetura

```
  Serviço de Domínio          Webhook Service              Cliente Externo
       │                            │                            │
       │ evento de domínio          │                            │
       └──► Kafka ─────────────────►│                            │
                             ┌──────┴──────┐                     │
                             │  Dispatcher │── POST + HMAC ──────►│
                             │             │◄── 2xx OK ───────────│
                             └──────┬──────┘                     │
                                    │ falhou?                     │
                             ┌──────▼──────┐                     │
                             │ Retry Queue │── retentar ─────────►│
                             │ (Kafka DLQ  │   (exponential       │
                             │  + schedule)│    backoff)          │
                             └─────────────┘
                                    │
                             ┌──────▼──────┐
                             │  PostgreSQL │
                             │ (tentativas,│
                             │  assinaturas│
                             │  endpoints) │
                             └─────────────┘
```

#### Implementação

```java
@Entity
@Table(name = "webhook_endpoints")
public class WebhookEndpoint {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String clienteId;
    private String url;

    @Column(nullable = false)
    private String segredoHmac;   // gerado na criação, nunca exposto na API

    @ElementCollection
    @CollectionTable(name = "webhook_endpoint_eventos")
    private Set<String> tiposEvento;  // ex: {"pagamento.confirmado", "pedido.criado"}

    private boolean ativo;
    private int falhasConsecutivas;
    private Instant ultimaFalhaEm;
    private Instant criadoEm;
}

@Entity
@Table(name = "webhook_entregas")
public class WebhookEntrega {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private UUID endpointId;
    private String tipoEvento;

    @Column(columnDefinition = "jsonb")
    private String payload;

    private int tentativa;
    private StatusEntrega status;  // PENDENTE, ENTREGUE, FALHOU, DESISTIU
    private int httpStatus;
    private String respostaCorpo;
    private long duracaoMs;
    private Instant agendadaPara;
    private Instant processadaEm;
}
```

```java
@Service
@Slf4j
public class WebhookDispatcherService {

    private final WebhookEndpointRepository endpointRepo;
    private final WebhookEntregaRepository entregaRepo;
    private final RestClient restClient;
    private final HmacService hmacService;
    private final KafkaTemplate<String, WebhookEntregaCommand> kafka;

    private static final int MAX_TENTATIVAS = 8;
    // Backoff: 30s, 1m, 5m, 30m, 2h, 6h, 12h, 24h
    private static final List<Duration> BACKOFF = List.of(
            Duration.ofSeconds(30), Duration.ofMinutes(1),
            Duration.ofMinutes(5), Duration.ofMinutes(30),
            Duration.ofHours(2), Duration.ofHours(6),
            Duration.ofHours(12), Duration.ofHours(24));

    // Recebe eventos de domínio e cria entregas para cada endpoint cadastrado
    @KafkaListener(topics = "dominio.eventos", groupId = "webhook-dispatcher")
    public void despachar(EventoDominioMessage evento, Acknowledgment ack) {
        List<WebhookEndpoint> endpoints = endpointRepo
                .findAtivosByTipoEvento(evento.tipo());

        for (WebhookEndpoint endpoint : endpoints) {
            WebhookEntrega entrega = new WebhookEntrega();
            entrega.setEndpointId(endpoint.getId());
            entrega.setTipoEvento(evento.tipo());
            entrega.setPayload(evento.payload());
            entrega.setTentativa(1);
            entrega.setStatus(StatusEntrega.PENDENTE);
            entrega.setAgendadaPara(Instant.now());
            entregaRepo.save(entrega);

            kafka.send("webhooks.para-entregar", entrega.getId().toString(),
                    new WebhookEntregaCommand(entrega.getId()));
        }
        ack.acknowledge();
    }

    @KafkaListener(topics = "webhooks.para-entregar", groupId = "webhook-executor")
    public void executar(WebhookEntregaCommand cmd, Acknowledgment ack) {
        WebhookEntrega entrega = entregaRepo.findById(cmd.entregaId()).orElseThrow();
        WebhookEndpoint endpoint = endpointRepo.findById(entrega.getEndpointId()).orElseThrow();

        Instant inicio = Instant.now();
        try {
            // Gera assinatura HMAC-SHA256 do payload
            String assinatura = hmacService.assinar(
                    entrega.getPayload(), endpoint.getSegredoHmac());

            // Entrega com timeout de 10s
            ResponseEntity<String> resp = restClient.post()
                    .uri(endpoint.getUrl())
                    .header("Content-Type", "application/json")
                    .header("X-Webhook-Event",     entrega.getTipoEvento())
                    .header("X-Webhook-Delivery",  entrega.getId().toString())
                    .header("X-Webhook-Timestamp", String.valueOf(Instant.now().getEpochSecond()))
                    .header("X-Webhook-Signature", "sha256=" + assinatura)
                    .body(entrega.getPayload())
                    .retrieve()
                    .toEntity(String.class);

            // Qualquer 2xx é considerado sucesso
            entrega.setHttpStatus(resp.getStatusCode().value());
            entrega.setRespostaCorpo(truncar(resp.getBody(), 1024));
            entrega.setStatus(StatusEntrega.ENTREGUE);
            entrega.setDuracaoMs(Duration.between(inicio, Instant.now()).toMillis());
            entregaRepo.save(entrega);

            // Reseta contador de falhas do endpoint
            endpoint.setFalhasConsecutivas(0);
            endpointRepo.save(endpoint);

            log.info("Webhook entregue: entregaId={}, url={}, status={}",
                    entrega.getId(), endpoint.getUrl(), resp.getStatusCode().value());

        } catch (Exception e) {
            int falhasConsec = endpoint.getFalhasConsecutivas() + 1;
            endpoint.setFalhasConsecutivas(falhasConsec);
            endpoint.setUltimaFalhaEm(Instant.now());

            // Desativa endpoint após 3 dias de falhas consecutivas (~ tentativa 7+)
            if (entrega.getTentativa() >= MAX_TENTATIVAS) {
                entrega.setStatus(StatusEntrega.DESISTIU);
                if (falhasConsec >= 7) {
                    endpoint.setAtivo(false);
                    log.warn("Endpoint desativado por falhas persistentes: id={}, url={}",
                            endpoint.getId(), endpoint.getUrl());
                }
            } else {
                // Agenda reentrega com backoff exponencial
                Duration espera = BACKOFF.get(Math.min(entrega.getTentativa() - 1, BACKOFF.size() - 1));
                WebhookEntrega reentrega = new WebhookEntrega();
                reentrega.setEndpointId(entrega.getEndpointId());
                reentrega.setTipoEvento(entrega.getTipoEvento());
                reentrega.setPayload(entrega.getPayload());
                reentrega.setTentativa(entrega.getTentativa() + 1);
                reentrega.setStatus(StatusEntrega.PENDENTE);
                reentrega.setAgendadaPara(Instant.now().plus(espera));
                entregaRepo.save(reentrega);

                entrega.setStatus(StatusEntrega.FALHOU);
                log.warn("Webhook falhou, reagendado em {}: entregaId={}, tentativa={}",
                        espera, entrega.getId(), entrega.getTentativa());
            }

            entrega.setDuracaoMs(Duration.between(inicio, Instant.now()).toMillis());
            entregaRepo.save(entrega);
            endpointRepo.save(endpoint);
        }
        ack.acknowledge();
    }
}
```

```java
// HMAC-SHA256 — receptor verifica autenticidade do payload
@Service
public class HmacService {

    public String assinar(String payload, String segredo) throws Exception {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(segredo.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
        byte[] hash = mac.doFinal(payload.getBytes(StandardCharsets.UTF_8));
        return HexFormat.of().formatHex(hash);
    }

    // Receptor usa este método para validar a assinatura recebida
    public boolean verificar(String payload, String segredo, String assinaturaRecebida) throws Exception {
        String esperada = assinar(payload, segredo);
        // Comparação em tempo constante — evita timing attack
        return MessageDigest.isEqual(
                esperada.getBytes(StandardCharsets.UTF_8),
                assinaturaRecebida.getBytes(StandardCharsets.UTF_8));
    }
}
```

```java
// Portal de gerenciamento para o cliente externo
@RestController
@RequestMapping("/api/v1/webhooks")
public class WebhookPortalController {

    private final WebhookEndpointRepository endpointRepo;
    private final WebhookEntregaRepository entregaRepo;
    private final WebhookDispatcherService dispatcher;

    @PostMapping("/endpoints")
    @ResponseStatus(HttpStatus.CREATED)
    public WebhookEndpointResponse registrar(
            @RequestBody @Valid RegistrarEndpointRequest req,
            @AuthenticationPrincipal Jwt jwt) {
        WebhookEndpoint endpoint = new WebhookEndpoint();
        endpoint.setClienteId(jwt.getSubject());
        endpoint.setUrl(req.url());
        endpoint.setTiposEvento(req.tiposEvento());
        endpoint.setSegredoHmac(UUID.randomUUID().toString().replace("-", ""));
        endpoint.setAtivo(true);
        endpointRepo.save(endpoint);
        // Retorna o segredo apenas na criação — nunca mais
        return WebhookEndpointResponse.comSegredo(endpoint);
    }

    // Histórico de entregas de um endpoint
    @GetMapping("/endpoints/{endpointId}/entregas")
    public Page<WebhookEntregaResponse> historico(
            @PathVariable UUID endpointId,
            Pageable pageable,
            @AuthenticationPrincipal Jwt jwt) {
        verificarProprietario(endpointId, jwt.getSubject());
        return entregaRepo.findByEndpointId(endpointId, pageable)
                .map(WebhookEntregaResponse::from);
    }

    // Reenvio manual de uma entrega específica
    @PostMapping("/entregas/{entregaId}/reenviar")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void reenviar(
            @PathVariable UUID entregaId,
            @AuthenticationPrincipal Jwt jwt) {
        WebhookEntrega entrega = entregaRepo.findById(entregaId).orElseThrow();
        verificarProprietario(entrega.getEndpointId(), jwt.getSubject());
        dispatcher.reenviarManualmente(entregaId);
    }
}
```

---

### 18.13 Relatórios Pesados e Exportação de Dados

#### Contexto

Uma plataforma de gestão precisa gerar relatórios analíticos complexos (vendas por período, inadimplência, ranking de produtos) e exportá-los em PDF e Excel. Essas queries podem levar de 10 segundos a vários minutos, inviabilizando uma resposta síncrona na API.

#### Requisitos-Chave

```
Funcionais:
  - Solicitar relatório com filtros (período, categoria, filial)
  - Acompanhar progresso da geração
  - Baixar o arquivo gerado (PDF, XLSX, CSV)
  - Histórico de relatórios gerados pelo usuário
  - Agendamento recorrente (ex: toda segunda às 8h)

Não-Funcionais:
  - Queries pesadas executadas em réplica (sem impactar produção)
  - Geração não bloqueia a API — responde imediatamente com jobId
  - Arquivos disponíveis por 7 dias (expiração automática no S3)
  - Limite de relatórios simultâneos por usuário (throttling)
```

#### Arquitetura

```
  Cliente               API               Worker               Storage
    │                    │                   │                    │
    │── POST /relatorios►│                   │                    │
    │◄── 202 + jobId ────│                   │                    │
    │                    │── Kafka ──────────►│                    │
    │                    │                   │── query na réplica  │
    │                    │                   │── gera PDF/XLSX     │
    │                    │                   │── upload ──────────►│ S3
    │                    │                   │── atualiza job      │
    │── GET /relatorios  │                   │   (status=PRONTO)   │
    │   /{jobId} ───────►│                   │                    │
    │◄── 200 + URL ──────│                   │                    │
    │── GET URL ─────────────────────────────────────────────────►│
    │◄── download arquivo────────────────────────────────────────│
```

#### Implementação

```java
@Entity
@Table(name = "relatorio_jobs")
public class RelatorioJob {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String solicitanteId;
    private String tipo;             // ex: "VENDAS_PERIODO", "RANKING_PRODUTOS"

    @Column(columnDefinition = "jsonb")
    private String filtrosJson;

    private String formato;          // PDF, XLSX, CSV
    private StatusRelatorio status;  // AGUARDANDO, PROCESSANDO, PRONTO, FALHOU

    @Column(name = "progresso_pct")
    private int progressoPct;        // 0..100 para mostrar barra de progresso

    private String arquivoKey;       // chave no S3
    private String arquivoUrl;       // URL pré-assinada (gerada no download)
    private String mensagemErro;
    private Instant solicitadoEm;
    private Instant concluidoEm;
    private Instant expiraEm;        // arquivo expira em 7 dias
}
```

```java
@Service
@Slf4j
public class RelatorioService {

    private final RelatorioJobRepository jobRepo;
    private final KafkaTemplate<String, GerarRelatorioCommand> kafka;
    private final S3Client s3;

    @Value("${aws.s3.bucket.relatorios}")
    private String bucketRelatorios;

    @Transactional
    public RelatorioJobResponse solicitar(SolicitarRelatorioRequest req, String usuarioId) {
        // Throttling: máximo 3 relatórios simultâneos por usuário
        long emAndamento = jobRepo.contarEmAndamento(usuarioId);
        if (emAndamento >= 3) {
            throw new LimiteRelatoriosAtingidoException(usuarioId);
        }

        RelatorioJob job = new RelatorioJob();
        job.setSolicitanteId(usuarioId);
        job.setTipo(req.tipo());
        job.setFiltrosJson(serializarFiltros(req.filtros()));
        job.setFormato(req.formato());
        job.setStatus(StatusRelatorio.AGUARDANDO);
        job.setProgressoPct(0);
        job.setSolicitadoEm(Instant.now());
        job.setExpiraEm(Instant.now().plus(Duration.ofDays(7)));
        jobRepo.save(job);

        kafka.send("relatorios.solicitar", job.getId().toString(),
                new GerarRelatorioCommand(job.getId()));

        return RelatorioJobResponse.from(job);
    }

    public RelatorioJobResponse consultar(UUID jobId, String usuarioId) {
        RelatorioJob job = jobRepo.findById(jobId).orElseThrow();
        verificarProprietario(job, usuarioId);

        // Gera URL pré-assinada apenas quando o relatório está pronto
        if (job.getStatus() == StatusRelatorio.PRONTO && job.getArquivoUrl() == null) {
            String url = gerarUrlDownload(job.getArquivoKey());
            job.setArquivoUrl(url);
            jobRepo.save(job);
        }

        return RelatorioJobResponse.from(job);
    }

    private String gerarUrlDownload(String key) {
        return s3Presigner.presignGetObject(r -> r
                .signatureDuration(Duration.ofHours(1))
                .getObjectRequest(g -> g.bucket(bucketRelatorios).key(key)))
                .url().toString();
    }
}
```

```java
@Component
@Slf4j
public class RelatorioWorker {

    private final RelatorioJobRepository jobRepo;
    private final DataSource replicaDataSource;  // lê da réplica
    private final RelatorioRenderizadorFactory renderizadorFactory;
    private final S3Client s3;

    @Value("${aws.s3.bucket.relatorios}")
    private String bucketRelatorios;

    @KafkaListener(topics = "relatorios.solicitar", groupId = "relatorio-worker")
    public void processar(GerarRelatorioCommand cmd, Acknowledgment ack) {
        RelatorioJob job = jobRepo.findById(cmd.jobId()).orElseThrow();
        job.setStatus(StatusRelatorio.PROCESSANDO);
        job.setProgressoPct(5);
        jobRepo.save(job);

        try {
            // 1. Executa query na réplica (evita impacto no banco de produção)
            List<Map<String, Object>> dados = executarQuery(job);
            atualizarProgresso(job, 50);

            // 2. Renderiza no formato solicitado
            RelatorioRenderizador renderizador = renderizadorFactory.criar(job.getFormato());
            byte[] arquivo = renderizador.renderizar(job.getTipo(), dados, job.getFiltros());
            atualizarProgresso(job, 85);

            // 3. Faz upload para S3
            String key = "relatorios/%s/%s/%s.%s".formatted(
                    job.getSolicitanteId(),
                    LocalDate.now(),
                    job.getId(),
                    job.getFormato().toLowerCase());

            s3.putObject(
                    r -> r.bucket(bucketRelatorios).key(key)
                            .contentType(renderizador.contentType())
                            .expires(job.getExpiraEm()),
                    RequestBody.fromBytes(arquivo));

            job.setArquivoKey(key);
            job.setStatus(StatusRelatorio.PRONTO);
            job.setProgressoPct(100);
            job.setConcluidoEm(Instant.now());
            jobRepo.save(job);
            ack.acknowledge();

            log.info("Relatório gerado: jobId={}, tipo={}, tamanho={}KB",
                    job.getId(), job.getTipo(), arquivo.length / 1024);

        } catch (Exception e) {
            job.setStatus(StatusRelatorio.FALHOU);
            job.setMensagemErro(e.getMessage());
            jobRepo.save(job);
            log.error("Falha ao gerar relatório: jobId={}", job.getId(), e);
            throw new RuntimeException(e);
        }
    }

    private void atualizarProgresso(RelatorioJob job, int pct) {
        job.setProgressoPct(pct);
        jobRepo.save(job);
    }

    private List<Map<String, Object>> executarQuery(RelatorioJob job) {
        try (Connection conn = replicaDataSource.getConnection()) {
            String sql = QueryCatalogo.obter(job.getTipo());
            FiltrosRelatorio filtros = deserializarFiltros(job.getFiltrosJson());
            // Executa query parametrizada na réplica
            return JdbcTemplate.queryForList(conn, sql, filtros.toParams());
        }
    }
}
```

```java
// Renderizadores por formato
@Component
public class ExcelRenderizador implements RelatorioRenderizador {

    @Override
    public byte[] renderizar(String tipo, List<Map<String, Object>> dados,
                              FiltrosRelatorio filtros) throws IOException {
        try (Workbook wb = new SXSSFWorkbook(100); // streaming — baixo uso de memória
             ByteArrayOutputStream out = new ByteArrayOutputStream()) {

            Sheet sheet = wb.createSheet("Relatório");

            // Cabeçalho
            Row header = sheet.createRow(0);
            CellStyle headerStyle = criarEstiloCabecalho(wb);
            List<String> colunas = new ArrayList<>(dados.get(0).keySet());
            for (int i = 0; i < colunas.size(); i++) {
                Cell cell = header.createCell(i);
                cell.setCellValue(colunas.get(i));
                cell.setCellStyle(headerStyle);
            }

            // Dados
            for (int r = 0; r < dados.size(); r++) {
                Row row = sheet.createRow(r + 1);
                Map<String, Object> registro = dados.get(r);
                for (int c = 0; c < colunas.size(); c++) {
                    preencherCelula(row.createCell(c), registro.get(colunas.get(c)));
                }
            }

            // Auto-ajusta largura das colunas
            for (int i = 0; i < colunas.size(); i++) sheet.autoSizeColumn(i);

            wb.write(out);
            return out.toByteArray();
        }
    }

    @Override public String contentType() { return "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"; }
    @Override public String extensao()    { return "xlsx"; }
}
```

---

### 18.14 Carrinho de Compras e Checkout

#### Contexto

Um e-commerce precisa de um carrinho de compras que persista entre sessões, aplique cupons e promoções, reserve estoque temporariamente durante o checkout, e transfira a responsabilidade de pagamento sem perder os dados do pedido se o usuário abandonar o fluxo.

#### Requisitos-Chave

```
Funcionais:
  - Adicionar, remover e atualizar itens do carrinho
  - Carrinho persiste mesmo se o usuário fechar o navegador
  - Aplicar cupom de desconto com validação de regras
  - Reservar estoque ao iniciar o checkout (por 15 min)
  - Liberar reserva se o checkout for abandonado
  - Calcular frete por CEP

Não-Funcionais:
  - Carrinho disponível em qualquer dispositivo (sincronizado)
  - Operações de carrinho em < 30ms (Redis)
  - Sem oversell: estoque garantido durante o checkout
```

#### Arquitetura

```
Estados do carrinho:

  ABERTO ──► CHECKOUT_INICIADO ──► PEDIDO_CRIADO
    │               │
    │               └── timeout 15 min → volta para ABERTO
    │                   (estoque liberado)
    │
    └── abandono → persiste indefinidamente (recuperação por e-mail)
```

```java
// Carrinho armazenado no Redis como Hash
@Service
@Slf4j
public class CarrinhoService {

    private final StringRedisTemplate redis;
    private final ProdutoClient produtoClient;
    private final EstoqueClient estoqueClient;
    private final CupomService cupomService;
    private final ObjectMapper mapper;

    private static final String CARRINHO_KEY = "carrinho:";
    private static final Duration TTL_CARRINHO = Duration.ofDays(30);
    private static final Duration TTL_RESERVA  = Duration.ofMinutes(15);

    public CarrinhoResponse obter(String usuarioId) {
        String key = CARRINHO_KEY + usuarioId;
        Map<Object, Object> campos = redis.opsForHash().entries(key);
        if (campos.isEmpty()) return CarrinhoResponse.vazio(usuarioId);
        return deserializar(campos);
    }

    @Transactional
    public CarrinhoResponse adicionarItem(String usuarioId, AdicionarItemRequest req) {
        String key = CARRINHO_KEY + usuarioId;

        // Valida se o produto existe e tem estoque disponível (cache)
        ProdutoResponse produto = produtoClient.buscarProduto(req.produtoId());
        if (!produto.disponivel()) throw new ProdutoIndisponivelException(req.produtoId());

        // Incrementa quantidade se já existe; adiciona se não existe
        String itemJson = redis.opsForHash().get(key, req.produtoId().toString()).toString();
        ItemCarrinho item;
        if (itemJson != null) {
            item = mapper.readValue(itemJson, ItemCarrinho.class);
            item = item.comQuantidade(item.quantidade() + req.quantidade());
        } else {
            item = new ItemCarrinho(req.produtoId(), produto.nome(),
                    produto.preco(), req.quantidade(), produto.imagemUrl());
        }

        redis.opsForHash().put(key, req.produtoId().toString(), mapper.writeValueAsString(item));
        redis.expire(key, TTL_CARRINHO);

        return calcularTotais(usuarioId);
    }

    public CarrinhoResponse removerItem(String usuarioId, Long produtoId) {
        redis.opsForHash().delete(CARRINHO_KEY + usuarioId, produtoId.toString());
        return calcularTotais(usuarioId);
    }

    public CarrinhoResponse aplicarCupom(String usuarioId, String codigoCupom) {
        CarrinhoResponse carrinho = obter(usuarioId);
        CupomValidado cupom = cupomService.validar(codigoCupom, usuarioId,
                carrinho.subtotal());
        redis.opsForHash().put(CARRINHO_KEY + usuarioId, "cupom",
                mapper.writeValueAsString(cupom));
        return calcularTotais(usuarioId);
    }

    // Inicia checkout: reserva estoque e retorna token de checkout
    @Transactional
    public IniciarCheckoutResponse iniciarCheckout(String usuarioId) {
        CarrinhoResponse carrinho = obter(usuarioId);
        if (carrinho.itens().isEmpty()) throw new CarrinhoVazioException();

        // Reserva estoque de todos os itens atomicamente
        ReservaEstoque reserva = estoqueClient.reservar(
                ReservaRequest.from(carrinho.itens(), TTL_RESERVA));

        // Salva ID da reserva no carrinho
        redis.opsForHash().put(CARRINHO_KEY + usuarioId, "reservaId", reserva.id().toString());

        // Token de checkout com TTL — expirou = reserva liberada
        String checkoutToken = UUID.randomUUID().toString();
        redis.opsForValue().set(
                "checkout:token:" + checkoutToken,
                usuarioId,
                TTL_RESERVA);

        // Job agendado para liberar reserva se não concluída em 15 min
        agendarLiberacaoReserva(reserva.id(), TTL_RESERVA);

        log.info("Checkout iniciado: usuario={}, reservaId={}", usuarioId, reserva.id());
        return new IniciarCheckoutResponse(checkoutToken, reserva.id(),
                carrinho, TTL_RESERVA);
    }

    // Limpa o carrinho após pedido criado
    @Transactional
    public void limparAposPedido(String usuarioId) {
        redis.delete(CARRINHO_KEY + usuarioId);
        log.info("Carrinho limpo após pedido: usuario={}", usuarioId);
    }

    private CarrinhoResponse calcularTotais(String usuarioId) {
        Map<Object, Object> campos = redis.opsForHash().entries(CARRINHO_KEY + usuarioId);
        List<ItemCarrinho> itens = campos.entrySet().stream()
                .filter(e -> !e.getKey().toString().startsWith("cupom")
                          && !e.getKey().toString().startsWith("reserva"))
                .map(e -> deserializarItem(e.getValue().toString()))
                .toList();

        BigDecimal subtotal = itens.stream()
                .map(i -> i.preco().multiply(BigDecimal.valueOf(i.quantidade())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);

        // Aplica desconto do cupom se presente
        String cupomJson = (String) campos.get("cupom");
        BigDecimal desconto = BigDecimal.ZERO;
        if (cupomJson != null) {
            CupomValidado cupom = deserializarCupom(cupomJson);
            desconto = cupom.calcularDesconto(subtotal);
        }

        return new CarrinhoResponse(usuarioId, itens, subtotal, desconto,
                subtotal.subtract(desconto));
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/carrinho")
public class CarrinhoController {

    private final CarrinhoService carrinhoService;
    private final PedidoService pedidoService;

    @GetMapping
    public CarrinhoResponse obter(@AuthenticationPrincipal Jwt jwt) {
        return carrinhoService.obter(jwt.getSubject());
    }

    @PostMapping("/itens")
    public CarrinhoResponse adicionar(
            @RequestBody @Valid AdicionarItemRequest req,
            @AuthenticationPrincipal Jwt jwt) {
        return carrinhoService.adicionarItem(jwt.getSubject(), req);
    }

    @DeleteMapping("/itens/{produtoId}")
    public CarrinhoResponse remover(
            @PathVariable Long produtoId,
            @AuthenticationPrincipal Jwt jwt) {
        return carrinhoService.removerItem(jwt.getSubject(), produtoId);
    }

    @PostMapping("/cupom")
    public CarrinhoResponse aplicarCupom(
            @RequestBody @Valid AplicarCupomRequest req,
            @AuthenticationPrincipal Jwt jwt) {
        return carrinhoService.aplicarCupom(jwt.getSubject(), req.codigo());
    }

    @PostMapping("/checkout")
    public IniciarCheckoutResponse iniciarCheckout(
            @AuthenticationPrincipal Jwt jwt) {
        return carrinhoService.iniciarCheckout(jwt.getSubject());
    }

    // Confirma o pedido após pagamento autorizado
    @PostMapping("/checkout/confirmar")
    public ResponseEntity<PedidoResponse> confirmar(
            @RequestHeader("X-Checkout-Token") String token,
            @RequestBody @Valid ConfirmarCheckoutRequest req,
            @AuthenticationPrincipal Jwt jwt) {

        PedidoResponse pedido = pedidoService.criarDesdoCarrinho(
                jwt.getSubject(), token, req);
        carrinhoService.limparAposPedido(jwt.getSubject());
        URI location = URI.create("/api/v1/pedidos/" + pedido.id());
        return ResponseEntity.created(location).body(pedido);
    }
}
```

---

### 18.15 Encurtador de URLs

#### Contexto

Um serviço de encurtamento de URLs (estilo bit.ly ou tinyurl) recebe uma URL longa e retorna um código curto de 7 caracteres. Ao acessar o código, o serviço redireciona para a URL original. É um problema clássico de system design por envolver hashing, alta taxa de leitura, cache e geração de IDs únicos distribuídos.

#### Estimativas

```
Escritas (criações):    100 URLs/s = 8,6 milhões/dia
Leituras (redirects):   10.000/s   (100x mais que escritas)
Tamanho médio da URL:   200 bytes
Armazenamento (5 anos): 8,6M/dia × 365 × 5 × 200 bytes ≈ 3,1 TB

Leitura altamente cacheável: a mesma URL curta é acessada
muitas vezes — hit rate de cache > 95% esperado.
```

#### Geração do Código Curto

```
Opção 1 — Hash (MD5/SHA-256) + truncar:
  hash("https://exemplo.com/pagina") → "5f4dcc3b" (primeiros 7 chars)
  Problema: colisões possíveis.
  Solução: verificar unicidade e tentar próximo bloco do hash.

Opção 2 — ID numérico + Base62 (recomendada):
  ID auto-incremento (1, 2, 3...)
  1         → "000001"
  2.176.782.336 → "ZZZZZZ" (7 chars Base62 = 62^7 ≈ 3,5 trilhões de URLs)

  Base62: 0-9 + a-z + A-Z (62 caracteres — sem caracteres especiais na URL)
  Vantagem: sem colisão, ordenado, reversível.

Opção 3 — Snowflake ID + Base62:
  IDs únicos distribuídos (timestamp + worker id + sequência)
  Sem coordenação central — escala horizontalmente.
```

#### Arquitetura

```
  Cliente              API (N instâncias)            Redis (cache)
    │                         │                           │
    │── POST /urls ──────────►│                           │
    │◄── { "curta": "xK3mN9p" }                          │
    │                         │── INSERT PostgreSQL        │
    │                         │── SET Redis (TTL 24h)      │
    │                         │                           │
    │── GET /xK3mN9p ────────►│── GET Redis ─────────────►│
    │◄── 301/302 Redirect ────│◄── URL original ───────────│
    │   Location: <url longa> │   (cache hit: > 95%)       │
```

#### Implementação

```java
// Gerador de ID baseado em Snowflake (distribuído, sem coordenação)
@Component
public class SnowflakeIdGenerator {

    private final long workerId;
    private long sequencia = 0L;
    private long ultimoTimestamp = -1L;

    private static final long EPOCH          = 1700000000000L; // Nov 2023
    private static final long WORKER_BITS    = 10L;
    private static final long SEQUENCIA_BITS = 12L;
    private static final long MAX_WORKER     = ~(-1L << WORKER_BITS);  // 1023
    private static final long MAX_SEQUENCIA  = ~(-1L << SEQUENCIA_BITS); // 4095

    public SnowflakeIdGenerator(@Value("${app.worker-id:0}") long workerId) {
        if (workerId > MAX_WORKER) throw new IllegalArgumentException("workerId inválido");
        this.workerId = workerId;
    }

    public synchronized long gerar() {
        long agora = System.currentTimeMillis();
        if (agora == ultimoTimestamp) {
            sequencia = (sequencia + 1) & MAX_SEQUENCIA;
            if (sequencia == 0) agora = aguardarProximoMs(ultimoTimestamp);
        } else {
            sequencia = 0L;
        }
        ultimoTimestamp = agora;
        return ((agora - EPOCH) << (WORKER_BITS + SEQUENCIA_BITS))
                | (workerId << SEQUENCIA_BITS)
                | sequencia;
    }

    private long aguardarProximoMs(long ts) {
        long agora = System.currentTimeMillis();
        while (agora <= ts) agora = System.currentTimeMillis();
        return agora;
    }
}
```

```java
// Conversão entre ID numérico e código Base62
@Component
public class Base62Codec {

    private static final String ALFABETO =
            "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private static final int BASE = 62;
    private static final int TAMANHO_CODIGO = 7;

    public String codificar(long id) {
        StringBuilder sb = new StringBuilder();
        while (id > 0) {
            sb.append(ALFABETO.charAt((int)(id % BASE)));
            id /= BASE;
        }
        // Padding à esquerda com '0' até 7 caracteres
        while (sb.length() < TAMANHO_CODIGO) sb.append('0');
        return sb.reverse().toString();
    }

    public long decodificar(String codigo) {
        long id = 0;
        for (char c : codigo.toCharArray()) {
            id = id * BASE + ALFABETO.indexOf(c);
        }
        return id;
    }
}
```

```java
@Service
@Slf4j
public class UrlEncurtadoraService {

    private final UrlRepository urlRepo;
    private final StringRedisTemplate redis;
    private final SnowflakeIdGenerator idGenerator;
    private final Base62Codec codec;

    private static final Duration TTL_CACHE   = Duration.ofHours(24);
    private static final String   REDIRECT_KEY = "url:";

    @Transactional
    public UrlCurtaResponse encurtar(EncurtarUrlRequest req, String usuarioId) {
        // Idempotência: se a mesma URL longa já foi encurtada pelo mesmo usuário,
        // retorna o código existente
        return urlRepo.findByUrlLongaEUsuario(req.urlLonga(), usuarioId)
                .map(UrlCurtaResponse::from)
                .orElseGet(() -> criarNova(req, usuarioId));
    }

    private UrlCurtaResponse criarNova(EncurtarUrlRequest req, String usuarioId) {
        long id = idGenerator.gerar();
        String codigo = codec.codificar(id);

        UrlCurta url = new UrlCurta();
        url.setId(id);
        url.setCodigo(codigo);
        url.setUrlLonga(req.urlLonga());
        url.setUsuarioId(usuarioId);
        url.setCriadaEm(Instant.now());
        url.setExpiraEm(req.expiraEm()); // null = nunca expira
        url.setAtiva(true);

        urlRepo.save(url);
        redis.opsForValue().set(REDIRECT_KEY + codigo, req.urlLonga(), TTL_CACHE);

        log.info("URL encurtada: codigo={}, destino={}", codigo, req.urlLonga());
        return UrlCurtaResponse.from(url);
    }

    public String resolver(String codigo) {
        // 1. Tenta o cache Redis (hot path — > 95% das requisições terminam aqui)
        String cached = redis.opsForValue().get(REDIRECT_KEY + codigo);
        if (cached != null) return cached;

        // 2. Cache miss — busca no banco
        UrlCurta url = urlRepo.findByCodigoAtiva(codigo)
                .orElseThrow(() -> new UrlNaoEncontradaException(codigo));

        if (url.getExpiraEm() != null && url.getExpiraEm().isBefore(Instant.now())) {
            throw new UrlExpiradaException(codigo);
        }

        // Repõe no cache
        redis.opsForValue().set(REDIRECT_KEY + codigo, url.getUrlLonga(), TTL_CACHE);
        return url.getUrlLonga();
    }
}
```

```java
@RestController
public class UrlRedirectController {

    private final UrlEncurtadoraService service;
    private final AnalyticsService analytics;

    // Criação da URL curta
    @PostMapping("/api/v1/urls")
    @ResponseStatus(HttpStatus.CREATED)
    public UrlCurtaResponse encurtar(
            @RequestBody @Valid EncurtarUrlRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        return service.encurtar(request, jwt.getSubject());
    }

    // Redirect — rota raiz para minimizar o path da URL curta
    // Ex: https://meu.link/xK3mN9p
    @GetMapping("/{codigo}")
    public ResponseEntity<Void> redirecionar(
            @PathVariable String codigo,
            HttpServletRequest request) {

        String destino = service.resolver(codigo);

        // Registra acesso de forma assíncrona (não atrasa o redirect)
        analytics.registrarAcessoAsync(codigo, request.getRemoteAddr(),
                request.getHeader("User-Agent"), request.getHeader("Referer"));

        // 301 Moved Permanently → browser cacheia o redirect (zero latência nas próximas vezes)
        // 302 Found → browser sempre consulta o servidor (permite expiração e analytics precisos)
        return ResponseEntity.status(HttpStatus.FOUND)  // 302 para analytics
                .location(URI.create(destino))
                .build();
    }

    // Analytics de uma URL
    @GetMapping("/api/v1/urls/{codigo}/stats")
    public UrlStatsResponse stats(
            @PathVariable String codigo,
            @AuthenticationPrincipal Jwt jwt) {
        return analytics.obterStats(codigo, jwt.getSubject());
    }
}
```

```java
// Analytics assíncrono — não atrasa o redirect
@Service
public class AnalyticsService {

    private final KafkaTemplate<String, AcessoUrlEvent> kafka;
    private final StringRedisTemplate redis;

    @Async
    public void registrarAcessoAsync(String codigo, String ip,
                                      String userAgent, String referer) {
        // Contador em Redis (rápido)
        redis.opsForValue().increment("stats:clicks:" + codigo);

        // Evento para analytics detalhado (país, dispositivo, etc.)
        kafka.send("urls.acessos", codigo,
                new AcessoUrlEvent(codigo, ip, userAgent, referer, Instant.now()));
    }

    public UrlStatsResponse obterStats(String codigo, String usuarioId) {
        Long clicks = redis.opsForValue()
                .get("stats:clicks:" + codigo) != null
                ? Long.parseLong(redis.opsForValue().get("stats:clicks:" + codigo))
                : 0L;
        // Dados detalhados viriam do banco (top países, dispositivos, etc.)
        return new UrlStatsResponse(codigo, clicks);
    }
}
```

---

### 18.16 Chatbot com Integração de Agentes de IA

#### Contexto

Uma empresa de e-commerce ou serviços financeiros precisa de um chatbot de atendimento ao cliente capaz de responder dúvidas sobre produtos, consultar status de pedidos, abrir chamados e escalar para um atendente humano quando necessário. O bot deve usar um LLM (Large Language Model) para linguagem natural, ter acesso a dados reais da empresa via **Tool Use** (function calling), e manter contexto da conversa sem "esquecer" o que foi dito.

#### Conceitos-Chave

```
LLM (Large Language Model):
  Modelo de linguagem que gera respostas em linguagem natural.
  Ex: Claude (Anthropic), GPT-4 (OpenAI), Gemini (Google).

RAG (Retrieval-Augmented Generation):
  Antes de chamar o LLM, busca documentos relevantes (FAQ, manuais)
  em um banco vetorial e os injeta no contexto do prompt.
  Resultado: respostas baseadas em dados da empresa, não só no
  conhecimento geral do modelo.

Tool Use (Function Calling):
  O LLM pode "chamar funções" da aplicação para buscar dados reais.
  Ex: "qual o status do pedido 123?" → LLM chama buscarPedido(123)
  → recebe o resultado → responde com linguagem natural.

Memória de Conversa:
  O histórico de mensagens é enviado junto com cada nova pergunta,
  permitindo que o LLM mantenha contexto da conversa.

Human Handoff (Escalada):
  Quando o bot detecta frustração, pergunta fora do escopo ou
  solicitação explícita, a conversa é transferida para um humano.
```

#### Arquitetura

```
  Cliente (Web/App)              Chatbot Service                  LLM (Claude API)
        │                               │                               │
        │── POST /chat/mensagem ────────►│                               │
        │                               │── 1. RAG: busca docs ────────►│
        │                               │   relevantes (pgvector)       │
        │                               │                               │
        │                               │── 2. Monta prompt:            │
        │                               │   system + histórico +        │
        │                               │   docs RAG + pergunta         │
        │                               │                               │
        │                               │── 3. POST /messages ─────────►│
        │                               │                               │── analisa
        │                               │                               │── precisa
        │                               │                               │   de tool?
        │                               │◄── tool_use: buscarPedido ────│
        │                               │── 4. executa a ferramenta     │
        │                               │   (busca no banco)            │
        │                               │── 5. retorna resultado ───────►│
        │                               │◄── resposta final (streaming) ─│
        │◄── SSE chunks da resposta ────│                               │
```

#### Implementação

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>sdk</artifactId>
    <version>0.9.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

```yaml
anthropic:
  api-key: ${ANTHROPIC_API_KEY}
  model: claude-opus-4-6
  max-tokens: 2048

spring:
  ai:
    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 1536   # dimensões do modelo de embedding
```

```java
// Definição das ferramentas (tools) disponíveis para o LLM
@Component
public class ChatbotTools {

    private final PedidoRepository pedidoRepo;
    private final ProdutoRepository produtoRepo;
    private final ChamadoService chamadoService;

    // O LLM chama esta ferramenta quando o usuário pergunta sobre um pedido
    @Tool(name = "buscar_pedido",
          description = "Busca status e detalhes de um pedido pelo número. "
                      + "Use quando o usuário perguntar sobre entrega, status ou detalhes do pedido.")
    public PedidoInfo buscarPedido(
            @ToolParam(description = "Número do pedido (somente dígitos)") String numeroPedido) {
        return pedidoRepo.findByNumero(numeroPedido)
                .map(PedidoInfo::from)
                .orElse(PedidoInfo.naoEncontrado(numeroPedido));
    }

    @Tool(name = "buscar_produto",
          description = "Busca informações de um produto: preço, disponibilidade, especificações.")
    public ProdutoInfo buscarProduto(
            @ToolParam(description = "Nome ou código do produto") String termoBusca) {
        return produtoRepo.buscarParaChatbot(termoBusca)
                .stream().findFirst()
                .map(ProdutoInfo::from)
                .orElse(ProdutoInfo.naoEncontrado(termoBusca));
    }

    @Tool(name = "abrir_chamado",
          description = "Abre um chamado de suporte quando o usuário relata um problema "
                      + "que não pode ser resolvido automaticamente.")
    public ChamadoInfo abrirChamado(
            @ToolParam(description = "Descrição detalhada do problema") String descricao,
            @ToolParam(description = "Número do pedido relacionado, se houver") String numeroPedido) {
        return chamadoService.abrir(descricao, numeroPedido);
    }

    @Tool(name = "escalar_para_humano",
          description = "Transfere a conversa para um atendente humano. "
                      + "Use quando: o usuário pedir explicitamente, "
                      + "estiver frustrado, ou o problema for complexo demais.")
    public EscaladaInfo escalarParaHumano(
            @ToolParam(description = "Motivo da escalada") String motivo) {
        return new EscaladaInfo(true, "Aguarde, estou transferindo para um atendente.", motivo);
    }
}
```

```java
// Serviço principal do chatbot — orquestra RAG + LLM + Tool Use
@Service
@Slf4j
public class ChatbotService {

    private final AnthropicClient anthropic;
    private final VectorStore vectorStore;
    private final ConversaRepository conversaRepo;
    private final ChatbotTools tools;

    @Value("${anthropic.model}")
    private String model;

    private static final String SYSTEM_PROMPT = """
            Você é um assistente de atendimento ao cliente da Loja Exemplo.
            Seja sempre cordial, objetivo e empático.

            Diretrizes:
            - Responda apenas sobre assuntos relacionados à loja e seus produtos.
            - Se não souber a resposta, use as ferramentas disponíveis para buscar.
            - Para problemas que não puder resolver, abra um chamado ou escale para humano.
            - Nunca invente informações — se não tiver certeza, diga que vai verificar.
            - Responda sempre em português do Brasil.
            """;

    public Flux<String> responder(String conversaId, String usuarioId,
                                   String mensagemUsuario) {
        // 1. Recupera ou cria a conversa
        Conversa conversa = conversaRepo.findById(conversaId)
                .orElseGet(() -> conversaRepo.save(new Conversa(conversaId, usuarioId)));

        if (conversa.isEscaladaParaHumano()) {
            return Flux.just("Esta conversa foi transferida para um atendente humano. "
                    + "Por favor, aguarde.");
        }

        // 2. RAG — busca documentos relevantes para enriquecer o contexto
        List<Document> docsRelevantes = vectorStore.similaritySearch(
                SearchRequest.query(mensagemUsuario).withTopK(3).withSimilarityThreshold(0.7));

        String contextoDocs = docsRelevantes.stream()
                .map(Document::getContent)
                .collect(Collectors.joining("\n\n---\n\n"));

        // 3. Monta histórico de mensagens
        List<MessageParam> historico = conversa.getMensagens().stream()
                .map(m -> MessageParam.builder()
                        .role(m.isPeloBot() ? MessageParam.Role.ASSISTANT : MessageParam.Role.USER)
                        .content(m.getConteudo())
                        .build())
                .toList();

        // 4. Prepara a mensagem atual com contexto RAG injetado
        String promptComContexto = contextoDocs.isBlank()
                ? mensagemUsuario
                : """
                  Contexto da base de conhecimento:
                  %s

                  Pergunta do cliente:
                  %s
                  """.formatted(contextoDocs, mensagemUsuario);

        List<MessageParam> mensagens = new ArrayList<>(historico);
        mensagens.add(MessageParam.builder()
                .role(MessageParam.Role.USER)
                .content(promptComContexto)
                .build());

        // 5. Chama o LLM com streaming + ferramentas disponíveis
        return Flux.create(sink -> {
            anthropic.messages()
                    .createStreaming(MessageCreateParamsBuilder.builder()
                            .model(model)
                            .maxTokens(2048)
                            .system(SYSTEM_PROMPT)
                            .messages(mensagens)
                            .tools(construirDefinicaoTools())
                            .toolChoice(ToolChoiceAuto.builder().build())
                            .build())
                    .subscribe(
                            event -> processarEvento(event, sink, conversa,
                                    mensagemUsuario),
                            sink::error,
                            sink::complete);
        });
    }

    private void processarEvento(RawMessageStreamEvent event,
                                  FluxSink<String> sink,
                                  Conversa conversa,
                                  String mensagemUsuario) {
        if (event instanceof RawContentBlockDeltaEvent delta) {
            // Streaming: cada chunk de texto é enviado ao cliente
            if (delta.delta() instanceof TextDelta textDelta) {
                sink.next(textDelta.text());
            }
        } else if (event instanceof RawMessageStopEvent) {
            // Mensagem completa — persiste no histórico
            conversa.adicionarMensagem(mensagemUsuario, false);
            // A resposta completa será salva ao acumular os chunks
            conversaRepo.save(conversa);
        }
    }

    // Indexação da base de conhecimento (FAQ, manuais, políticas)
    public void indexarDocumento(String titulo, String conteudo, String categoria) {
        Document doc = new Document(conteudo, Map.of(
                "titulo",    titulo,
                "categoria", categoria,
                "indexadoEm", Instant.now().toString()));
        vectorStore.add(List.of(doc));
        log.info("Documento indexado: titulo={}, categoria={}", titulo, categoria);
    }
}
```

```java
// Controller com streaming via SSE
@RestController
@RequestMapping("/api/v1/chat")
public class ChatbotController {

    private final ChatbotService chatbotService;
    private final ConversaRepository conversaRepo;

    // Endpoint de streaming — resposta chega em chunks (Server-Sent Events)
    @PostMapping(value = "/conversas/{conversaId}/mensagens",
                 produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> enviarMensagem(
            @PathVariable String conversaId,
            @RequestBody @Valid EnviarMensagemRequest request,
            @AuthenticationPrincipal Jwt jwt) {

        return chatbotService.responder(conversaId, jwt.getSubject(),
                request.mensagem())
                .doOnError(e -> log.error("Erro no chatbot: conversaId={}", conversaId, e));
    }

    // Cria nova conversa
    @PostMapping("/conversas")
    @ResponseStatus(HttpStatus.CREATED)
    public ConversaResponse iniciar(@AuthenticationPrincipal Jwt jwt) {
        String conversaId = UUID.randomUUID().toString();
        return new ConversaResponse(conversaId, jwt.getSubject(), Instant.now());
    }

    // Histórico da conversa
    @GetMapping("/conversas/{conversaId}")
    public ConversaDetalheResponse historico(
            @PathVariable String conversaId,
            @AuthenticationPrincipal Jwt jwt) {
        return conversaRepo.findById(conversaId)
                .map(ConversaDetalheResponse::from)
                .orElseThrow();
    }
}
```

```java
// Pipeline de indexação — ingere documentos na base vetorial
@Service
@Slf4j
public class DocumentoIndexadorService {

    private final ChatbotService chatbotService;
    private final EmbeddingModel embeddingModel;

    // Indexa FAQ completo da empresa
    @Transactional(readOnly = true)
    public void reindexarFaq(List<FaqItem> itens) {
        itens.forEach(item -> chatbotService.indexarDocumento(
                item.getPergunta(),
                "Pergunta: %s\nResposta: %s".formatted(item.getPergunta(), item.getResposta()),
                "faq"));
        log.info("FAQ reindexado: {} itens", itens.size());
    }

    // Indexa política de devolução, termos de uso, manual do produto, etc.
    public void indexarArquivoTexto(String titulo, String categoria, String conteudo) {
        // Divide textos longos em chunks com sobreposição (sliding window)
        List<String> chunks = dividirEmChunks(conteudo, 500, 50);
        chunks.forEach(chunk -> chatbotService.indexarDocumento(titulo, chunk, categoria));
        log.info("Arquivo indexado: titulo={}, chunks={}", titulo, chunks.size());
    }

    private List<String> dividirEmChunks(String texto, int tamanho, int sobreposicao) {
        List<String> chunks = new ArrayList<>();
        String[] palavras = texto.split("\\s+");
        for (int i = 0; i < palavras.length; i += tamanho - sobreposicao) {
            int fim = Math.min(i + tamanho, palavras.length);
            chunks.add(String.join(" ", Arrays.copyOfRange(palavras, i, fim)));
            if (fim == palavras.length) break;
        }
        return chunks;
    }
}
```

#### Fluxo Completo com Tool Use

```
Usuário: "Cadê meu pedido 98765?"

1. RAG busca docs sobre "rastreamento de pedidos" → injeta no prompt

2. LLM recebe:
   [system: instruções]
   [user: "Cadê meu pedido 98765?"]
   [tools: buscar_pedido, abrir_chamado, escalar_para_humano, ...]

3. LLM decide chamar tool: buscar_pedido("98765")

4. Aplicação executa: pedidoRepo.findByNumero("98765")
   → { status: "EM_TRANSPORTE", previsao: "2025-04-02", rastreio: "BR123..." }

5. LLM recebe o resultado e gera resposta:
   "Seu pedido #98765 está a caminho! 🚚 Ele está em transporte e
    a previsão de entrega é 02/04. Código de rastreio: BR123..."

6. Resposta chega em streaming ao cliente (chunk a chunk)
```

#### Decisões Arquiteturais

| Decisão | Escolha | Motivação |
|---------|---------|-----------|
| LLM | Claude via Anthropic API | Qualidade de raciocínio, Tool Use nativo, contexto longo |
| Memória de conversa | Histórico persistido no banco, enviado a cada chamada | LLMs são stateless — o estado fica na aplicação |
| Base de conhecimento | pgvector (PostgreSQL) com embeddings | Sem serviço externo extra; busca semântica eficiente |
| Streaming da resposta | SSE com Flux (WebFlux) | Experiência de digitação em tempo real; reduz percepção de latência |
| Tool Use | Ferramentas definidas no código, chamadas pelo LLM | LLM decide *quando* e *quais* ferramentas usar conforme o contexto |
| Fallback humano | Ferramenta `escalar_para_humano` chamada pelo próprio LLM | O modelo detecta frustração ou complexidade e transfere proativamente |
| Rate limiting | Por usuário e por conversa | Custos do LLM são por token — abuso pode gerar custos altos |
| Cache de respostas | Cache por hash do prompt para perguntas idênticas | FAQs muito repetidas (ex: "qual o prazo de entrega?") não chamam o LLM |

---

### 18.17 Rastreabilidade com Blockchain

#### Contexto

Uma cadeia de suprimentos (agronegócio, farmácia, luxo) ou uma plataforma de certificados digitais precisa garantir **imutabilidade** e **auditabilidade** de registros sem que uma única parte controle os dados. Blockchain resolve o problema de confiança entre participantes que não se conhecem e não querem depender de um intermediário centralizado.

#### Quando Usar (e Quando Não Usar) Blockchain

```
✅ USE blockchain quando:
  - Múltiplas organizações independentes precisam compartilhar dados
  - Nenhuma parte pode ser o custódio único (conflito de interesses)
  - Imutabilidade e não-repúdio são requisitos de negócio ou legais
  - Auditoria por terceiros sem acesso ao banco interno
  - Tokenização de ativos (certificados, títulos, direitos de propriedade)

❌ NÃO use blockchain quando:
  - Apenas uma organização controla os dados → use DB com auditoria
  - Precisa de atualizações frequentes e rápidas → latência e custo de gas
  - Os dados são confidenciais e não podem ser públicos → use DB privado
  - O problema real é de processos, não de tecnologia
  - Um banco de dados + assinatura digital resolve igualmente

Regra prática:
  "Se você pode substituir o blockchain por um banco de dados
   com auditoria e assinatura digital, provavelmente deve."
```

#### Caso de Uso: Rastreabilidade de Medicamentos na Cadeia Farmacêutica

```
Participantes: Fabricante → Distribuidora → Farmácia → Anvisa (regulador)
Problema:      Cada ator tem seu próprio sistema. Falsificações existem
               porque não há registro único e imutável de cada lote.
Solução:       Cada movimentação de um lote registrada em um smart contract.
               Qualquer participante (e a Anvisa) pode auditar sem depender
               de outro participante.

Fluxo:
  Fabricante ──► contrato.registrarLote(loteId, composição, validade)
  Distribuidora ─► contrato.registrarTransferencia(loteId, destino, data)
  Farmácia ──────► contrato.registrarRecebimento(loteId, quantidade)
  Consumidor ────► contrato.verificar(loteId) → histórico completo
  Anvisa ─────────► contrato.verificar(loteId) → auditoria sem intermediário
```

#### Arquitetura Híbrida (Blockchain + Banco Tradicional)

```
  Aplicação Spring Boot
        │
        ├── PostgreSQL (dados completos, operacionais, privados)
        │   - dados sensíveis (preço, fornecedor, cliente)
        │   - consultas rápidas e complexas
        │   - dados que podem ser corrigidos
        │
        └── Blockchain Ethereum / Polygon (dados de rastreabilidade)
            - hash dos documentos (prova de existência)
            - eventos de custódia (quem tinha o quê e quando)
            - apenas dados que precisam ser públicos e imutáveis
            - smart contracts com lógica de negócio imutável

Regra de ouro da arquitetura híbrida:
  Blockchain armazena o HASH e os EVENTOS.
  O banco armazena os DADOS COMPLETOS.
  O hash no blockchain prova que o dado no banco não foi alterado.
```

#### Smart Contract em Solidity

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * Rastreabilidade de lotes farmacêuticos.
 * Cada movimentação é registrada como evento imutável na chain.
 */
contract RastreabilidadeMedicamentos {

    // Papéis dos participantes
    mapping(address => bool) public fabricantes;
    mapping(address => bool) public distribuidoras;
    mapping(address => bool) public farmacias;
    address public regulador; // Anvisa

    // Estado atual de cada lote
    struct Lote {
        string loteId;
        address custodiante;    // quem tem o lote agora
        EstadoLote estado;
        bytes32 hashDados;      // hash SHA-256 dos dados completos no DB off-chain
        uint256 criadoEm;
    }

    enum EstadoLote {
        FABRICADO,
        EM_TRANSPORTE,
        NO_DISTRIBUIDOR,
        EM_TRANSPORTE_FINAL,
        NA_FARMACIA,
        DISPENSADO,
        RECOLHIDO     // recall
    }

    mapping(string => Lote) public lotes;

    // Eventos — ficam imutavelmente registrados na blockchain
    event LoteRegistrado(
        string indexed loteId,
        address fabricante,
        bytes32 hashDados,
        uint256 timestamp
    );
    event CustodiaTransferida(
        string indexed loteId,
        address de,
        address para,
        EstadoLote novoEstado,
        uint256 timestamp
    );
    event LoteRecolhido(
        string indexed loteId,
        address autoridade,
        string motivo,
        uint256 timestamp
    );

    modifier somenteFabricante() {
        require(fabricantes[msg.sender], "Apenas fabricantes");
        _;
    }
    modifier somenteCustodiante(string memory loteId) {
        require(lotes[loteId].custodiante == msg.sender, "Apenas o custodiante atual");
        _;
    }
    modifier somenteRegulador() {
        require(msg.sender == regulador, "Apenas o regulador");
        _;
    }

    constructor() {
        regulador = msg.sender;
    }

    function registrarLote(
        string memory loteId,
        bytes32 hashDados      // hash dos dados completos no banco off-chain
    ) external somenteFabricante {
        require(lotes[loteId].criadoEm == 0, "Lote ja registrado");

        lotes[loteId] = Lote({
            loteId:      loteId,
            custodiante: msg.sender,
            estado:      EstadoLote.FABRICADO,
            hashDados:   hashDados,
            criadoEm:    block.timestamp
        });

        emit LoteRegistrado(loteId, msg.sender, hashDados, block.timestamp);
    }

    function transferirCustodia(
        string memory loteId,
        address destino,
        EstadoLote novoEstado
    ) external somenteCustodiante(loteId) {
        address anterior = lotes[loteId].custodiante;
        lotes[loteId].custodiante = destino;
        lotes[loteId].estado = novoEstado;

        emit CustodiaTransferida(loteId, anterior, destino, novoEstado, block.timestamp);
    }

    function recolherLote(
        string memory loteId,
        string memory motivo
    ) external somenteRegulador {
        lotes[loteId].estado = EstadoLote.RECOLHIDO;
        emit LoteRecolhido(loteId, msg.sender, motivo, block.timestamp);
    }

    function verificarLote(string memory loteId)
        external view returns (Lote memory) {
        return lotes[loteId];
    }
}
```

#### Integração Spring Boot com Web3j

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>core</artifactId>
    <version>4.12.2</version>
</dependency>
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>contracts</artifactId>
    <version>4.12.2</version>
</dependency>
```

```yaml
blockchain:
  rpc-url:          ${BLOCKCHAIN_RPC_URL}          # ex: Infura, Alchemy, nó próprio
  chain-id:         137                            # Polygon Mainnet (menores taxas)
  contrato-address: ${CONTRATO_RASTREABILIDADE}
  wallet-path:      ${WALLET_KEYSTORE_PATH}
  wallet-password:  ${WALLET_PASSWORD}
  gas-price-gwei:   50
  gas-limit:        300000
```

```java
@Configuration
public class BlockchainConfig {

    @Value("${blockchain.rpc-url}")
    private String rpcUrl;

    @Value("${blockchain.chain-id}")
    private long chainId;

    @Bean
    public Web3j web3j() {
        return Web3j.build(new HttpService(rpcUrl));
    }

    @Bean
    public Credentials credentials(
            @Value("${blockchain.wallet-path}") String walletPath,
            @Value("${blockchain.wallet-password}") String password) throws Exception {
        return WalletUtils.loadCredentials(password, walletPath);
    }

    @Bean
    public RastreabilidadeMedicamentos contratoRastreabilidade(
            Web3j web3j,
            Credentials credentials,
            @Value("${blockchain.contrato-address}") String address,
            @Value("${blockchain.gas-price-gwei}") long gasPriceGwei,
            @Value("${blockchain.gas-limit}") long gasLimit) {

        ContractGasProvider gasProvider = new StaticGasProvider(
                BigInteger.valueOf(gasPriceGwei).multiply(BigInteger.TEN.pow(9)),
                BigInteger.valueOf(gasLimit));

        return RastreabilidadeMedicamentos.load(address, web3j, credentials, gasProvider);
    }
}
```

```java
@Service
@Slf4j
public class RastreabilidadeService {

    private final RastreabilidadeMedicamentos contrato;
    private final LoteRepository loteRepo;
    private final Web3j web3j;

    /**
     * Registra um novo lote: persiste dados completos no DB e o hash no blockchain.
     * Arquitetura híbrida: blockchain guarda a prova; DB guarda os dados.
     */
    @Transactional
    public LoteRegistradoResponse registrarLote(RegistrarLoteRequest req) {
        // 1. Persiste dados completos no PostgreSQL
        Lote lote = loteRepo.save(new Lote(req));

        // 2. Calcula hash SHA-256 dos dados canônicos do lote
        String dadosJson = serializarDadosCanonicos(lote);
        byte[] hashBytes = calcularSha256(dadosJson);
        byte[] hash32 = Arrays.copyOf(hashBytes, 32); // bytes32 no Solidity

        lote.setHashOnChain(HexFormat.of().formatHex(hash32));
        loteRepo.save(lote);

        try {
            // 3. Registra na blockchain (transação assíncrona — leva alguns segundos)
            TransactionReceipt receipt = contrato.registrarLote(
                    lote.getLoteId(),
                    hash32
            ).send();

            lote.setTxHash(receipt.getTransactionHash());
            lote.setBlockNumber(receipt.getBlockNumber().longValue());
            loteRepo.save(lote);

            log.info("Lote registrado na blockchain: loteId={}, txHash={}",
                    lote.getLoteId(), receipt.getTransactionHash());

        } catch (Exception e) {
            // Falha na blockchain não deve reverter o registro no DB
            // Job de reprocessamento tentará novamente
            lote.setStatusBlockchain(StatusBlockchain.PENDENTE);
            loteRepo.save(lote);
            log.error("Falha ao registrar na blockchain: loteId={}", lote.getLoteId(), e);
        }

        return LoteRegistradoResponse.from(lote);
    }

    /**
     * Transfere custódia de um lote para outro participante.
     */
    @Transactional
    public void transferirCustodia(TransferirCustodiaRequest req) {
        Lote lote = loteRepo.findByLoteId(req.loteId()).orElseThrow();

        // Atualiza no DB
        lote.setCustodiante(req.destinoAddress());
        lote.setEstado(req.novoEstado());
        loteRepo.save(lote);

        // Registra na blockchain
        try {
            TransactionReceipt receipt = contrato.transferirCustodia(
                    req.loteId(),
                    req.destinoAddress(),
                    BigInteger.valueOf(req.novoEstado().ordinal())
            ).send();

            log.info("Custódia transferida on-chain: loteId={}, de={}, para={}, tx={}",
                    req.loteId(), req.deAddress(), req.destinoAddress(),
                    receipt.getTransactionHash());

        } catch (Exception e) {
            lote.setStatusBlockchain(StatusBlockchain.PENDENTE_TRANSFERENCIA);
            loteRepo.save(lote);
            throw new BlockchainException("Falha ao registrar transferência", e);
        }
    }

    /**
     * Verifica integridade de um lote: compara hash no DB com hash na blockchain.
     */
    public VerificacaoIntegridade verificarIntegridade(String loteId) {
        Lote lote = loteRepo.findByLoteId(loteId).orElseThrow();

        try {
            // Busca hash registrado na blockchain (chamada de leitura — gratuita)
            RastreabilidadeMedicamentos.Lote loteOnChain = contrato
                    .verificarLote(loteId).send();

            String hashOnChain = HexFormat.of().formatHex(loteOnChain.hashDados);
            String hashNoDb    = lote.getHashOnChain();
            boolean integro    = hashOnChain.equals(hashNoDb);

            if (!integro) {
                log.error("ALERTA: Integridade comprometida! loteId={}, hashDB={}, hashChain={}",
                        loteId, hashNoDb, hashOnChain);
            }

            return new VerificacaoIntegridade(loteId, integro, hashOnChain,
                    hashNoDb, loteOnChain.estado.intValue());

        } catch (Exception e) {
            throw new BlockchainException("Falha ao verificar integridade", e);
        }
    }

    /**
     * Histórico completo de eventos de um lote lidos diretamente da blockchain.
     * Imutável — nenhuma parte pode alterar.
     */
    public List<EventoLote> buscarHistoricoOnChain(String loteId) {
        List<EventoLote> eventos = new ArrayList<>();

        // Filtra eventos de registro
        contrato.loteRegistradoEventFlowable(
                DefaultBlockParameterName.EARLIEST,
                DefaultBlockParameterName.LATEST)
                .filter(e -> e.loteId.equals(loteId))
                .map(e -> new EventoLote("REGISTRADO", e.fabricante,
                        null, Instant.ofEpochSecond(e.timestamp.longValue())))
                .blockingSubscribe(eventos::add);

        // Filtra eventos de transferência de custódia
        contrato.custodiaTransferidaEventFlowable(
                DefaultBlockParameterName.EARLIEST,
                DefaultBlockParameterName.LATEST)
                .filter(e -> e.loteId.equals(loteId))
                .map(e -> new EventoLote("TRANSFERENCIA", e.de, e.para,
                        Instant.ofEpochSecond(e.timestamp.longValue())))
                .blockingSubscribe(eventos::add);

        return eventos.stream()
                .sorted(Comparator.comparing(EventoLote::timestamp))
                .toList();
    }

    private byte[] calcularSha256(String dados) {
        try {
            return MessageDigest.getInstance("SHA-256")
                    .digest(dados.getBytes(StandardCharsets.UTF_8));
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
// Job de reprocessamento para registros pendentes na blockchain
@Component
@Slf4j
public class BlockchainSyncJob {

    private final LoteRepository loteRepo;
    private final RastreabilidadeService service;

    @Scheduled(fixedDelay = 60_000) // a cada minuto
    public void sincronizarPendentes() {
        List<Lote> pendentes = loteRepo
                .findByStatusBlockchain(StatusBlockchain.PENDENTE);

        for (Lote lote : pendentes) {
            try {
                // Reprocessa o registro que falhou anteriormente
                service.reenviarParaBlockchain(lote);
                log.info("Lote sincronizado: loteId={}", lote.getLoteId());
            } catch (Exception e) {
                log.warn("Falha ao sincronizar lote: loteId={}", lote.getLoteId(), e);
            }
        }
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/rastreabilidade")
public class RastreabilidadeController {

    private final RastreabilidadeService service;

    @PostMapping("/lotes")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("hasRole('FABRICANTE')")
    public LoteRegistradoResponse registrar(
            @RequestBody @Valid RegistrarLoteRequest request) {
        return service.registrarLote(request);
    }

    @PostMapping("/lotes/{loteId}/transferencia")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @PreAuthorize("hasRole('FABRICANTE') or hasRole('DISTRIBUIDOR')")
    public void transferir(
            @PathVariable String loteId,
            @RequestBody @Valid TransferirCustodiaRequest request) {
        service.transferirCustodia(request.comLoteId(loteId));
    }

    // Qualquer participante (inclusive regulador) pode verificar
    @GetMapping("/lotes/{loteId}/verificar")
    public VerificacaoIntegridade verificar(@PathVariable String loteId) {
        return service.verificarIntegridade(loteId);
    }

    // Histórico imutável lido diretamente da blockchain
    @GetMapping("/lotes/{loteId}/historico")
    public List<EventoLote> historico(@PathVariable String loteId) {
        return service.buscarHistoricoOnChain(loteId);
    }
}
```

#### Outros Casos de Uso de Blockchain

```
Certificados Digitais (diplomas, laudos, contratos):
  - Hash do documento registrado na blockchain
  - QR Code no documento aponta para verificação on-chain
  - Instituição emissora não pode alegar que não emitiu (não-repúdio)
  - Qualquer pessoa verifica autenticidade sem depender da instituição

NFT para Ativos do Mundo Real (RWA — Real World Assets):
  - Tokenização de imóveis, obras de arte, recebíveis
  - Smart contract transfere propriedade com registro imutável
  - Elimina cartório intermediário para certos tipos de transferência

DeFi (Finanças Descentralizadas):
  - Empréstimos colateralizados sem banco intermediário
  - Exchange descentralizada (AMM — Automated Market Maker)
  - Stablecoins lastreadas em ativos

DAO (Organização Autônoma Descentralizada):
  - Governança on-chain: membros votam em propostas via tokens
  - Execução automática via smart contract se proposta aprovada
```

#### Decisões Arquiteturais

| Decisão | Escolha | Motivação |
|---------|---------|-----------|
| Blockchain | Polygon (EVM-compatible) | Compatível com Ethereum; taxas de gas ~100x menores; 2s de finalidade |
| Arquitetura | Híbrida (blockchain + PostgreSQL) | Dados completos no DB (privados, rápidos); apenas hash e eventos on-chain |
| Integração Java | Web3j | Biblioteca madura para JVM; gera wrappers Java do contrato Solidity |
| Persistência de falhas | StatusBlockchain.PENDENTE + job de retry | Blockchain pode estar congestionada; aplicação não pode depender de disponibilidade 100% |
| Leitura de histórico | Event logs da blockchain (gratuitos) | Eventos são a forma canônica de histórico em Ethereum; imutáveis e indexáveis |
| Dados off-chain | IPFS ou S3 com hash armazenado on-chain | Blockchain não é adequada para grandes volumes de dados |
| Verificação de integridade | Comparação de hash DB vs. hash on-chain | Prova criptográfica de que o dado no banco não foi alterado desde o registro |

---

### 18.18 Rede Social

#### Contexto

Uma rede social (estilo Instagram, LinkedIn ou Twitter/X) é um dos sistemas mais completos do ponto de vista arquitetural: combina grafo de relacionamentos, feed, mídia, busca, notificações, mensagens e moderação de conteúdo. Este estudo de caso foca nos componentes **específicos e centrais** de uma rede social que ainda não foram cobertos nos estudos anteriores, referenciando-os quando aplicável.

#### Mapa dos Componentes e Referências

```
┌───────────────────────────────────────────────────────────────────┐
│                         REDE SOCIAL                               │
│                                                                   │
│  ┌─────────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │  Grafo Social   │   │  Feed / Timeline  │   │   Busca       │  │
│  │  (novo — §A)    │   │  (ver §18.11)     │   │  (ver §18.4)  │  │
│  └─────────────────┘   └──────────────────┘   └───────────────┘  │
│                                                                   │
│  ┌─────────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │  Privacidade e  │   │  Trending /      │   │  Notificações │  │
│  │  Visibilidade   │   │  Hashtags        │   │  (ver §18.3)  │  │
│  │  (novo — §B)    │   │  (novo — §C)     │   └───────────────┘  │
│  └─────────────────┘   └──────────────────┘                      │
│                                                                   │
│  ┌─────────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │  Moderação de   │   │  Pessoas que vc  │   │  Mídia/Upload │  │
│  │  Conteúdo       │   │  pode conhecer   │   │  (ver §18.6)  │  │
│  │  (novo — §D)    │   │  (novo — §E)     │   └───────────────┘  │
│  └─────────────────┘   └──────────────────┘                      │
│                                                                   │
│  ┌─────────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │  Mensagens      │   │  Autenticação    │   │  Vídeos       │  │
│  │  (ver §18.9)    │   │  (ver §18.8)     │   │  (ver §18.7)  │  │
│  └─────────────────┘   └──────────────────┘   └───────────────┘  │
└───────────────────────────────────────────────────────────────────┘
```

#### Estimativas de Capacidade

```
DAU (usuários ativos/dia):    50 milhões
Publicações/dia:              10 milhões  → 116 posts/s
Leituras de feed/dia:         500 milhões → 5.800 req/s
Interações (likes/comentários): 200 milhões/dia → 2.300/s
Seguidores médios por usuário: 200
Celebridades (> 1M seguidores): ~10.000 contas

Armazenamento (posts texto):  10M posts/dia × 500 bytes = 5 GB/dia
Fotos:                        3M fotos/dia × 300 KB     = 900 GB/dia
```

---

#### A — Grafo de Relacionamentos

O núcleo diferencial de uma rede social é o grafo: quem segue quem, amizades mútuas, grau de separação. Esse grafo alimenta o feed, as recomendações e o modelo de privacidade.

##### Modelo de Dados do Grafo

```
Tipos de relacionamento:

  Direcionado (follow):      A → segue → B   (Instagram, Twitter)
    B não precisa seguir A de volta.
    Assimétrico: A vê posts de B; B não vê posts de A.

  Bidirecional (amizade):    A ↔ amigos ↔ B   (Facebook, LinkedIn)
    Requer aceitação de convite.
    Simétrico: ambos vêem posts um do outro.

  Misto (LinkedIn):          A → segue → B (sem amizade)
                             A ↔ conectado ↔ B (amizade profissional)
```

```sql
-- Tabela de relacionamentos (adjacency list — escala melhor que matriz)
CREATE TABLE relacionamentos (
    seguidor_id   BIGINT NOT NULL,
    seguido_id    BIGINT NOT NULL,
    tipo          VARCHAR(20) NOT NULL DEFAULT 'FOLLOW',
                  -- FOLLOW, AMIZADE_PENDENTE, AMIZADE_CONFIRMADA, BLOQUEADO
    criado_em     TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (seguidor_id, seguido_id)
);

-- Índices para os dois sentidos de navegação do grafo
CREATE INDEX idx_rel_seguidor ON relacionamentos (seguidor_id, tipo);
CREATE INDEX idx_rel_seguido  ON relacionamentos (seguido_id,  tipo);

-- Contadores desnormalizados (evitam COUNT(*) a cada requisição)
ALTER TABLE perfis
    ADD COLUMN total_seguindo  INT NOT NULL DEFAULT 0,
    ADD COLUMN total_seguidores INT NOT NULL DEFAULT 0;
```

```java
@Entity
@Table(name = "relacionamentos",
       uniqueConstraints = @UniqueConstraint(
               columnNames = {"seguidor_id", "seguido_id"}))
public class Relacionamento {

    @EmbeddedId
    private RelacionamentoId id;  // { seguidorId, seguidoId }

    @Enumerated(EnumType.STRING)
    private TipoRelacionamento tipo;

    private Instant criadoEm = Instant.now();
}
```

```java
@Service
@Slf4j
public class GrafoSocialService {

    private final RelacionamentoRepository relacionamentoRepo;
    private final PerfilRepository perfilRepo;
    private final StringRedisTemplate redis;
    private final ApplicationEventPublisher events;

    // Cache Redis de seguidores por usuário (para fan-out do feed — §18.11)
    private static final String SEGUIDORES_KEY = "grafo:seguidores:";
    private static final String SEGUINDO_KEY   = "grafo:seguindo:";

    @Transactional
    public void seguir(String seguidorId, String seguidoId) {
        if (seguidorId.equals(seguidoId))
            throw new AutoSeguimentoException();

        // Verifica se está bloqueado
        if (relacionamentoRepo.existsBloqueio(seguidoId, seguidorId))
            throw new UsuarioBloqueadoException(seguidoId);

        Relacionamento rel = new Relacionamento(seguidorId, seguidoId, TipoRelacionamento.FOLLOW);
        relacionamentoRepo.save(rel);

        // Atualiza contadores desnormalizados
        perfilRepo.incrementarSeguindo(seguidorId);
        perfilRepo.incrementarSeguidores(seguidoId);

        // Invalida caches
        redis.delete(SEGUIDORES_KEY + seguidoId);
        redis.delete(SEGUINDO_KEY   + seguidorId);

        // Evento para fan-out do feed (§18.11) e notificação (§18.3)
        events.publishEvent(new NovoSeguimentoEvent(seguidorId, seguidoId));

        log.info("Seguimento: {} → {}", seguidorId, seguidoId);
    }

    @Transactional
    public void deixarDeSeguir(String seguidorId, String seguidoId) {
        relacionamentoRepo.deleteById(new RelacionamentoId(seguidorId, seguidoId));
        perfilRepo.decrementarSeguindo(seguidorId);
        perfilRepo.decrementarSeguidores(seguidoId);
        redis.delete(SEGUIDORES_KEY + seguidoId);
        redis.delete(SEGUINDO_KEY   + seguidorId);
    }

    @Transactional
    public void bloquear(String bloqueadorId, String bloqueadoId) {
        // Remove seguimentos em ambos os sentidos
        relacionamentoRepo.deleteById(new RelacionamentoId(bloqueadorId, bloqueadoId));
        relacionamentoRepo.deleteById(new RelacionamentoId(bloqueadoId, bloqueadorId));

        // Registra bloqueio
        Relacionamento bloqueio = new Relacionamento(
                bloqueadorId, bloqueadoId, TipoRelacionamento.BLOQUEADO);
        relacionamentoRepo.save(bloqueio);

        redis.delete(SEGUIDORES_KEY + bloqueadorId);
        redis.delete(SEGUIDORES_KEY + bloqueadoId);
    }

    // Seguidores em cache Redis — lista usada pelo fan-out do feed
    public List<String> listarSeguidores(String usuarioId) {
        String key = SEGUIDORES_KEY + usuarioId;
        List<String> cached = redis.opsForList().range(key, 0, -1);
        if (cached != null && !cached.isEmpty()) return cached;

        List<String> seguidores = relacionamentoRepo
                .findSeguidoresByUsuario(usuarioId);
        if (!seguidores.isEmpty()) {
            redis.opsForList().rightPushAll(key, seguidores);
            redis.expire(key, Duration.ofMinutes(30));
        }
        return seguidores;
    }

    // Amigos em comum — BFS de profundidade 1 nos dois sentidos
    public List<String> amigosEmComum(String usuarioA, String usuarioB) {
        Set<String> seguindoA = new HashSet<>(
                relacionamentoRepo.findSeguindoByUsuario(usuarioA));
        Set<String> seguindoB = new HashSet<>(
                relacionamentoRepo.findSeguindoByUsuario(usuarioB));
        seguindoA.retainAll(seguindoB); // interseção
        return new ArrayList<>(seguindoA);
    }

    // Grau de separação entre dois usuários (BFS — máximo 3 níveis)
    public int grauDeSeparacao(String origem, String destino) {
        if (origem.equals(destino)) return 0;

        Set<String> visitados = new HashSet<>();
        Queue<String> fila = new LinkedList<>();
        fila.add(origem);
        visitados.add(origem);

        for (int grau = 1; grau <= 3; grau++) {
            int tamanhoNivel = fila.size();
            for (int i = 0; i < tamanhoNivel; i++) {
                String atual = fila.poll();
                List<String> vizinhos = relacionamentoRepo.findSeguindoByUsuario(atual);
                for (String vizinho : vizinhos) {
                    if (vizinho.equals(destino)) return grau;
                    if (visitados.add(vizinho)) fila.add(vizinho);
                }
            }
        }
        return -1; // não conectados dentro de 3 graus
    }
}
```

---

#### B — Modelo de Privacidade e Visibilidade

Cada post, story ou perfil tem uma política de visibilidade que precisa ser verificada eficientemente antes de exibir qualquer conteúdo.

```
Níveis de visibilidade:

  PUBLICO        → qualquer pessoa, autenticada ou não
  SEGUIDORES     → apenas quem o autor segue e é seguido de volta (ou só seguidores)
  AMIGOS_AMIGOS  → seguidores + seguidores dos seguidores (2° grau)
  LISTA_CUSTOM   → lista específica de usuários definida pelo autor
  SOMENTE_EU     → rascunho ou post privado
```

```java
@Entity
@Table(name = "posts")
public class Post {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String autorId;

    @Column(columnDefinition = "text")
    private String conteudo;

    @Enumerated(EnumType.STRING)
    private Visibilidade visibilidade;

    // Preenchido apenas quando visibilidade = LISTA_CUSTOM
    @ElementCollection
    @CollectionTable(name = "post_lista_acesso")
    private Set<String> listaAcesso;

    private boolean arquivado;
    private Instant criadoEm;
    private Instant expiraEm;  // para stories com expiração de 24h
}
```

```java
@Service
public class PrivacidadeService {

    private final GrafoSocialService grafoService;
    private final RelacionamentoRepository relacionamentoRepo;
    private final StringRedisTemplate redis;

    /**
     * Verifica se um usuário pode ver um post específico.
     * Chamado antes de exibir qualquer conteúdo no feed ou perfil.
     */
    public boolean podeVer(String visualizadorId, Post post) {
        String autorId = post.getAutorId();

        // Autor sempre vê seu próprio conteúdo
        if (autorId.equals(visualizadorId)) return true;

        // Bloqueio em qualquer direção impede visualização
        if (relacionamentoRepo.existeBloqueioEmQualquerDirecao(autorId, visualizadorId))
            return false;

        return switch (post.getVisibilidade()) {
            case PUBLICO        -> true;
            case SOMENTE_EU     -> false;
            case SEGUIDORES     -> relacionamentoRepo
                    .existsBySeguidorIdAndSeguidoId(visualizadorId, autorId);
            case AMIGOS_AMIGOS  -> estaA2Graus(visualizadorId, autorId);
            case LISTA_CUSTOM   -> post.getListaAcesso().contains(visualizadorId);
        };
    }

    /**
     * Filtra uma lista de posts removendo os que o usuário não pode ver.
     * Usa cache Redis para evitar queries repetidas do grafo.
     */
    public List<Post> filtrarVisiveis(String visualizadorId, List<Post> posts) {
        // Precarga: busca em lote os relacionamentos para evitar N+1
        Set<String> autores = posts.stream()
                .map(Post::getAutorId)
                .collect(Collectors.toSet());

        Set<String> autoresSeguidos = relacionamentoRepo
                .findAutoresSeguidos(visualizadorId, autores);
        Set<String> autoresBloqueados = relacionamentoRepo
                .findBloqueios(visualizadorId, autores);

        return posts.stream()
                .filter(p -> !autoresBloqueados.contains(p.getAutorId()))
                .filter(p -> switch (p.getVisibilidade()) {
                    case PUBLICO        -> true;
                    case SEGUIDORES     -> autoresSeguidos.contains(p.getAutorId());
                    case SOMENTE_EU     -> p.getAutorId().equals(visualizadorId);
                    case AMIGOS_AMIGOS  -> estaA2Graus(visualizadorId, p.getAutorId());
                    case LISTA_CUSTOM   -> p.getListaAcesso().contains(visualizadorId);
                })
                .toList();
    }

    private boolean estaA2Graus(String a, String b) {
        String cacheKey = "privacidade:2graus:%s:%s".formatted(a, b);
        String cached = redis.opsForValue().get(cacheKey);
        if (cached != null) return "1".equals(cached);

        boolean resultado = !amigosEmComum(a, b).isEmpty();
        redis.opsForValue().set(cacheKey, resultado ? "1" : "0", Duration.ofMinutes(5));
        return resultado;
    }

    private List<String> amigosEmComum(String a, String b) {
        return grafoService.amigosEmComum(a, b);
    }
}
```

---

#### C — Trending Topics e Hashtags

Detectar o que está em alta em tempo real exige contagem de menções em uma janela deslizante — não um contador simples que nunca reseta.

```
Problema com contador simples:
  #Carnaval tem 10M menções históricas e 100 novas/hora agora.
  #EleiçõesHoje tem 0 menções históricas e 50.000 novas/hora.
  Qual está em alta? #EleiçõesHoje — mas o contador simples diria #Carnaval.

Solução — Sliding Window com Redis:
  Conta menções por janela de tempo (última hora, últimas 24h).
  Usa Redis Sorted Set com score = timestamp, expirando membros antigos.
```

```java
@Service
@Slf4j
public class HashtagService {

    private final StringRedisTemplate redis;
    private final HashtagRepository hashtagRepo;

    private static final String TRENDING_1H  = "trending:1h";
    private static final String TRENDING_24H = "trending:24h";

    /**
     * Registra uso de uma hashtag e incrementa contadores de trending.
     * Chamado ao publicar um post com hashtags.
     */
    @Async
    public void registrarUso(List<String> hashtags, String postId) {
        Instant agora = Instant.now();
        long tsMs = agora.toEpochMilli();

        redis.executePipelined((RedisCallback<Object>) conn -> {
            for (String tag : hashtags) {
                String tagNorm = tag.toLowerCase().strip();

                // Adiciona evento ao Sorted Set — score = timestamp
                // Cada membro = "postId:timestamp" para unicidade
                String membro = postId + ":" + tsMs;
                conn.zSetCommands().zAdd(("ht:eventos:" + tagNorm).getBytes(),
                        (double) tsMs, membro.getBytes());

                // Incrementa contador de trending (janela 1h e 24h)
                conn.hashCommands().hIncrBy(TRENDING_1H.getBytes(),
                        tagNorm.getBytes(), 1);
                conn.hashCommands().hIncrBy(TRENDING_24H.getBytes(),
                        tagNorm.getBytes(), 1);
            }
            return null;
        });
    }

    /**
     * Job a cada 5 min: remove eventos expirados e recalcula ranking.
     * Garante que os contadores reflitam apenas a janela atual.
     */
    @Scheduled(fixedRate = 300_000)
    public void recalcularTrending() {
        long corteMs1h  = Instant.now().minus(Duration.ofHours(1)).toEpochMilli();
        long corteMs24h = Instant.now().minus(Duration.ofHours(24)).toEpochMilli();

        // Para cada hashtag ativa, remove eventos fora da janela e reconta
        Set<String> hashtagsAtivas = hashtagRepo.findHashtagsAtivas();
        for (String tag : hashtagsAtivas) {
            String eventosKey = "ht:eventos:" + tag;

            // Remove eventos mais antigos que 1h
            Long removidos1h = redis.opsForZSet()
                    .removeRangeByScore(eventosKey, 0, corteMs1h);

            // Recontagem precisa da janela 1h (score acumulado pode ter viés)
            Long count1h = redis.opsForZSet()
                    .count(eventosKey, corteMs1h, Double.MAX_VALUE);

            if (count1h != null && count1h > 0) {
                redis.opsForHash().put(TRENDING_1H, tag, count1h.toString());
            } else {
                redis.opsForHash().delete(TRENDING_1H, tag);
            }
        }

        log.debug("Trending recalculado: {} hashtags ativas", hashtagsAtivas.size());
    }

    /**
     * Retorna as N hashtags mais mencionadas na última hora.
     */
    public List<TrendingHashtag> buscarTrending(int limite, Duration janela) {
        String key = janela.toHours() <= 1 ? TRENDING_1H : TRENDING_24H;

        // Ordena o hash por valor (contagem) decrescente
        return redis.opsForHash().entries(key).entrySet().stream()
                .map(e -> new TrendingHashtag(
                        e.getKey().toString(),
                        Long.parseLong(e.getValue().toString())))
                .sorted(Comparator.comparingLong(TrendingHashtag::mencoes).reversed())
                .limit(limite)
                .toList();
    }

    // Extrai hashtags de um texto: "#java #spring #microservices"
    public List<String> extrairHashtags(String texto) {
        return Arrays.stream(texto.split("\\s+"))
                .filter(p -> p.startsWith("#") && p.length() > 1)
                .map(p -> p.replaceAll("[^#\\w]", "").toLowerCase())
                .filter(p -> p.length() >= 2 && p.length() <= 50)
                .distinct()
                .toList();
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/trending")
public class TrendingController {

    private final HashtagService hashtagService;

    @GetMapping
    public List<TrendingHashtag> trending(
            @RequestParam(defaultValue = "10") int limite,
            @RequestParam(defaultValue = "1") int horas) {
        return hashtagService.buscarTrending(
                Math.min(limite, 50),
                Duration.ofHours(horas));
    }
}
```

---

#### D — Moderação de Conteúdo

Uma rede social precisa remover conteúdo ilegal, discurso de ódio e spam sem censurar conteúdo legítimo. A abordagem mais escalável combina **detecção automática** (ML) com **revisão humana** para casos borderline.

```
Pipeline de moderação:

  Post publicado
       │
       ▼
  Detector automático (ML)
       │
  ┌────┴──────────────────────────────┐
  │                                   │
  ▼                                   ▼
Score baixo                     Score alto
(conteúdo ok)                   (provável violação)
  │                                   │
  ▼                                   ▼
Publicado                      Fila de revisão humana
  │                                 │         │
  │             Score altíssimo     ▼         ▼
  │             (conteúdo ilegal) Aprovado  Removido
  │                   │
  │                   ▼
  │             Remoção automática imediata
  │
  └── Denúncia de usuário → re-análise
```

```java
@Entity
@Table(name = "denuncias")
public class Denuncia {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String denuncianteId;
    private Long postId;
    private Long comentarioId;       // null se a denúncia é sobre um post

    @Enumerated(EnumType.STRING)
    private MotivoDenuncia motivo;   // SPAM, NUDEZ, VIOLENCIA, ODIO, DESINFORMACAO, OUTRO

    private String descricao;

    @Enumerated(EnumType.STRING)
    private StatusDenuncia status;   // PENDENTE, EM_REVISAO, APROVADA, REJEITADA

    private String revisorId;
    private String decisaoJustificativa;
    private Instant criadaEm;
    private Instant resolvidaEm;
}
```

```java
@Service
@Slf4j
public class ModeracaoService {

    private final PostRepository postRepo;
    private final DenunciaRepository denunciaRepo;
    private final ModeracaoIAClient moderacaoIA;
    private final KafkaTemplate<String, ModeracaoEvent> kafka;
    private final NotificacaoService notificacaoService;

    private static final double SCORE_REMOCAO_AUTO   = 0.95;
    private static final double SCORE_REVISAO_HUMANA = 0.70;
    private static final int    DENUNCIAS_REVISAO    = 5;

    /**
     * Análise automática ao publicar — detecta violações antes de exibir.
     */
    @Async
    public void analisarPost(Long postId) {
        Post post = postRepo.findById(postId).orElseThrow();

        AnaliseModeracaoResponse analise = moderacaoIA.analisar(
                post.getConteudo(), post.getMidiaUrls());

        post.setScoreModeracao(analise.score());
        post.setCategoriaViolacao(analise.categoria());

        if (analise.score() >= SCORE_REMOCAO_AUTO) {
            // Conteúdo claramente ilegal: remove imediatamente sem revisão
            removerPost(post, "AUTO", "Remoção automática: score=" + analise.score());
            notificacaoService.notificarUsuario(post.getAutorId(),
                    "Seu post foi removido por violar as Diretrizes da Comunidade.");
            log.warn("Post removido automaticamente: id={}, score={}",
                    postId, analise.score());

        } else if (analise.score() >= SCORE_REVISAO_HUMANA) {
            // Caso borderline: enfileira para revisão humana mas mantém publicado
            post.setStatusModeracao(StatusModeracao.EM_REVISAO);
            postRepo.save(post);
            kafka.send("moderacao.fila-revisao", postId.toString(),
                    new ModeracaoEvent(postId, analise, "AUTO"));
            log.info("Post enviado para revisão humana: id={}, score={}", postId, analise.score());

        } else {
            post.setStatusModeracao(StatusModeracao.APROVADO);
            postRepo.save(post);
        }
    }

    /**
     * Recebe denúncia de usuário.
     * Após N denúncias do mesmo post, envia para revisão humana automaticamente.
     */
    @Transactional
    public void registrarDenuncia(RegistrarDenunciaRequest req, String denuncianteId) {
        // Impede denúncias duplicadas
        if (denunciaRepo.existsByDenuncianteIdAndPostId(denuncianteId, req.postId()))
            throw new DenunicaJaRegistradaException();

        Denuncia denuncia = new Denuncia(denuncianteId, req);
        denunciaRepo.save(denuncia);

        long totalDenuncias = denunciaRepo.countByPostId(req.postId());

        // Escalada automática com base no volume de denúncias
        if (totalDenuncias >= DENUNCIAS_REVISAO) {
            Post post = postRepo.findById(req.postId()).orElseThrow();
            if (post.getStatusModeracao() != StatusModeracao.EM_REVISAO) {
                post.setStatusModeracao(StatusModeracao.EM_REVISAO);
                postRepo.save(post);
                kafka.send("moderacao.fila-revisao", req.postId().toString(),
                        new ModeracaoEvent(req.postId(), null, "DENUNCIA_USUARIOS"));
            }
        }

        log.info("Denúncia registrada: post={}, motivo={}, total={}",
                req.postId(), req.motivo(), totalDenuncias);
    }

    /**
     * Moderador humano toma decisão sobre um post em revisão.
     */
    @Transactional
    public void decidirRevisao(DecidirRevisaoRequest req, String revisorId) {
        Post post = postRepo.findById(req.postId()).orElseThrow();

        if (req.decisao() == DecisaoModeracao.REMOVER) {
            removerPost(post, revisorId, req.justificativa());
            notificacaoService.notificarUsuario(post.getAutorId(),
                    "Seu post foi removido após revisão por um moderador.");
        } else {
            post.setStatusModeracao(StatusModeracao.APROVADO);
            postRepo.save(post);
        }

        // Atualiza denúncias relacionadas
        denunciaRepo.resolverPorPost(req.postId(), revisorId,
                req.decisao(), req.justificativa());

        log.info("Revisão decidida: post={}, decisao={}, revisor={}",
                req.postId(), req.decisao(), revisorId);
    }

    private void removerPost(Post post, String autorDecisao, String justificativa) {
        post.setStatusModeracao(StatusModeracao.REMOVIDO);
        post.setRemovidoPor(autorDecisao);
        post.setRemocaoJustificativa(justificativa);
        post.setRemovidoEm(Instant.now());
        postRepo.save(post);

        // Registra para auditoria e possível apelação
        kafka.send("moderacao.remocoes", post.getId().toString(),
                new PostRemovidoEvent(post.getId(), autorDecisao, justificativa));
    }
}
```

```java
// Fila de revisão para moderadores humanos
@RestController
@RequestMapping("/api/v1/moderacao")
@PreAuthorize("hasRole('MODERADOR')")
public class ModeracaoController {

    private final ModeracaoService moderacaoService;
    private final PostRepository postRepo;

    // Lista posts aguardando revisão humana (ordenados por score de risco)
    @GetMapping("/fila")
    public Page<PostModeracaoView> filaRevisao(
            @RequestParam(required = false) String categoria,
            Pageable pageable) {
        return postRepo.findEmRevisao(categoria, pageable);
    }

    @PostMapping("/posts/{postId}/decisao")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void decidir(
            @PathVariable Long postId,
            @RequestBody @Valid DecidirRevisaoRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        moderacaoService.decidirRevisao(
                request.comPostId(postId), jwt.getSubject());
    }

    // Usuário registra denúncia
    @PostMapping("/denuncias")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("isAuthenticated()")
    public void denunciar(
            @RequestBody @Valid RegistrarDenunciaRequest request,
            @AuthenticationPrincipal Jwt jwt) {
        moderacaoService.registrarDenuncia(request, jwt.getSubject());
    }
}
```

---

#### E — Pessoas que Você Pode Conhecer (Recomendação de Conexões)

```
Algoritmos de recomendação de conexões:

  1. Amigos em comum (BFS 2° grau):
     Quem você segue também segue X → sugira X.
     O(seguindo × seguidores_do_seguindo) — eficiente até ~1.000 conexões.

  2. Informações de perfil (escola, empresa, cidade):
     Usuários com mesma empresa → alta probabilidade de se conhecer.

  3. Contatos importados (agenda do celular):
     Usuário autoriza acesso à agenda → hash dos telefones → encontra matches.

  4. Collaborative Filtering:
     "Usuários similares a você seguem X" — baseado em histórico de follows.

  Estratégia na prática: combinar 1 + 2 e recalcular offline periodicamente.
```

```java
@Service
@Slf4j
public class RecomendacaoConexoesService {

    private final RelacionamentoRepository relacionamentoRepo;
    private final PerfilRepository perfilRepo;
    private final StringRedisTemplate redis;

    private static final String SUGESTOES_KEY = "sugestoes:conexoes:";
    private static final int    MAX_SUGESTOES = 20;

    /**
     * Retorna sugestões de conexão para um usuário.
     * Combina amigos em comum + similaridade de perfil.
     */
    public List<SugestaoConexao> sugerirConexoes(String usuarioId) {
        // Tenta o cache (recalculado em background periodicamente)
        List<String> cached = redis.opsForList()
                .range(SUGESTOES_KEY + usuarioId, 0, MAX_SUGESTOES - 1);
        if (cached != null && !cached.isEmpty()) {
            return enriquecerComPerfis(cached, usuarioId);
        }

        List<SugestaoConexao> sugestoes = calcularSugestoes(usuarioId);
        cachearSugestoes(usuarioId, sugestoes);
        return sugestoes;
    }

    private List<SugestaoConexao> calcularSugestoes(String usuarioId) {
        Perfil perfilUsuario = perfilRepo.findById(usuarioId).orElseThrow();

        // Já segue ou bloqueou — excluir das sugestões
        Set<String> jaConectados = new HashSet<>(
                relacionamentoRepo.findTodosRelacionados(usuarioId));
        jaConectados.add(usuarioId);

        Map<String, Integer> scoreAmigosEmComum = new HashMap<>();

        // Estratégia 1: amigos de amigos (BFS profundidade 2)
        List<String> seguindo = relacionamentoRepo.findSeguindoByUsuario(usuarioId);
        for (String amigo : seguindo) {
            List<String> amigosDoAmigo = relacionamentoRepo.findSeguindoByUsuario(amigo);
            for (String candidato : amigosDoAmigo) {
                if (!jaConectados.contains(candidato)) {
                    scoreAmigosEmComum.merge(candidato, 1, Integer::sum);
                }
            }
        }

        // Estratégia 2: mesmo empregador ou mesma cidade
        List<String> porPerfil = perfilRepo
                .findSimilaresPorEmpresaOuCidade(
                        perfilUsuario.getEmpresa(),
                        perfilUsuario.getCidade(),
                        jaConectados,
                        50);

        // Combina scores
        Map<String, Double> scoresFinal = new HashMap<>();
        scoreAmigosEmComum.forEach((id, amigos) ->
                scoresFinal.merge(id, amigos * 10.0, Double::sum));
        porPerfil.forEach(id ->
                scoresFinal.merge(id, 5.0, Double::sum));

        return scoresFinal.entrySet().stream()
                .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
                .limit(MAX_SUGESTOES)
                .map(e -> new SugestaoConexao(
                        e.getKey(),
                        scoreAmigosEmComum.getOrDefault(e.getKey(), 0),
                        e.getValue()))
                .toList();
    }

    private void cachearSugestoes(String usuarioId, List<SugestaoConexao> sugestoes) {
        if (sugestoes.isEmpty()) return;
        String key = SUGESTOES_KEY + usuarioId;
        List<String> ids = sugestoes.stream().map(SugestaoConexao::usuarioId).toList();
        redis.delete(key);
        redis.opsForList().rightPushAll(key, ids);
        redis.expire(key, Duration.ofHours(6));
    }

    // Job offline — recalcula sugestões para usuários ativos em batch
    @Scheduled(cron = "0 0 3 * * *") // todo dia às 3h
    public void recalcularEmBatch() {
        List<String> usuariosAtivos = perfilRepo.findUsuariosAtivos(Duration.ofDays(7));
        log.info("Recalculando sugestões para {} usuários ativos", usuariosAtivos.size());
        usuariosAtivos.parallelStream().forEach(id -> {
            try {
                List<SugestaoConexao> sugestoes = calcularSugestoes(id);
                cachearSugestoes(id, sugestoes);
            } catch (Exception e) {
                log.warn("Falha ao recalcular sugestões para userId={}", id, e);
            }
        });
    }

    private List<SugestaoConexao> enriquecerComPerfis(
            List<String> ids, String usuarioId) {
        Set<String> jaConectados = new HashSet<>(
                relacionamentoRepo.findTodosRelacionados(usuarioId));
        return perfilRepo.findAllById(ids).stream()
                .filter(p -> !jaConectados.contains(p.getId()))
                .map(p -> new SugestaoConexao(p.getId(), 0, 0.0))
                .toList();
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/conexoes")
public class ConexoesController {

    private final GrafoSocialService grafoService;
    private final RecomendacaoConexoesService recomendacaoService;

    @PostMapping("/seguir/{usuarioId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void seguir(@PathVariable String usuarioId,
                       @AuthenticationPrincipal Jwt jwt) {
        grafoService.seguir(jwt.getSubject(), usuarioId);
    }

    @DeleteMapping("/seguir/{usuarioId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deixarDeSeguir(@PathVariable String usuarioId,
                               @AuthenticationPrincipal Jwt jwt) {
        grafoService.deixarDeSeguir(jwt.getSubject(), usuarioId);
    }

    @PostMapping("/bloquear/{usuarioId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void bloquear(@PathVariable String usuarioId,
                         @AuthenticationPrincipal Jwt jwt) {
        grafoService.bloquear(jwt.getSubject(), usuarioId);
    }

    @GetMapping("/sugestoes")
    public List<SugestaoConexao> sugestoes(@AuthenticationPrincipal Jwt jwt) {
        return recomendacaoService.sugerirConexoes(jwt.getSubject());
    }

    @GetMapping("/{usuarioA}/comuns/{usuarioB}")
    public List<String> amigosEmComum(
            @PathVariable String usuarioA,
            @PathVariable String usuarioB) {
        return grafoService.amigosEmComum(usuarioA, usuarioB);
    }
}
```

#### Decisões Arquiteturais Consolidadas

| Componente | Decisão | Motivação |
|------------|---------|-----------|
| **Grafo de relacionamentos** | Adjacency list no PostgreSQL + cache Redis | SQL suficiente até bilhões de arestas; cache elimina queries repetidas |
| **Grafo complexo (futuro)** | Migrar para Neo4j ao precisar de queries multi-hop | `MATCH (a)-[:SEGUE*1..3]-(b)` é trivial em Cypher, custoso em SQL |
| **Contadores de seguidores** | Desnormalizados na tabela de perfis | Evitam `COUNT(*)` a cada visualização de perfil |
| **Privacidade** | Verificação em batch com precarga de relacionamentos | Evita N+1 ao filtrar feed — 1 query por lote, não 1 por post |
| **Trending** | Redis Sorted Set por janela deslizante + job de recálculo | Contadores simples acumulam historicamente; Sorted Set por timestamp permite janelas precisas |
| **Moderação** | ML para score + revisão humana para casos borderline | 100% automático tem falsos positivos; 100% humano não escala |
| **Recomendações** | BFS 2° grau + perfil + cache offline | Cálculo online seria muito lento; resultado recalculado a cada 6h é suficientemente fresco |
| **Fan-out do feed** | Ver §18.11 (híbrido write/read por tamanho do grafo) | Celebridades com 10M seguidores não fazem fan-out on write |
| **Busca de usuários** | Ver §18.4 (Elasticsearch) | Busca por nome parcial, bio, localização — inviável no PostgreSQL sem índice full-text |
| **Notificações** | Ver §18.3 (multicanal) + WebSocket para in-app | Push imediato no app; e-mail/SMS para usuários offline |

---

> **Referências:**
> - *Designing Data-Intensive Applications* — Martin Kleppmann
> - *System Design Interview* (Vol. 1 e 2) — Alex Xu
> - *Building Microservices* (2ª ed.) — Sam Newman
> - *Release It!* — Michael Nygard
> - *Domain-Driven Design* — Eric Evans
> - Documentação oficial: [spring.io](https://spring.io), [resilience4j.readme.io](https://resilience4j.readme.io), [opentelemetry.io](https://opentelemetry.io)
