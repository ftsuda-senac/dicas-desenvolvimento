# Performance e Otimização de Recursos — Java e Spring Boot

> **Objetivo:** Cobrir de forma abrangente as técnicas de otimização de performance em aplicações Java/Spring Boot, desde o nível de código até configurações de JVM, sistema operacional e containers Docker, incluindo ferramentas de profiling, tracing, benchmarking e testes de carga — com exemplos práticos voltados para Spring Boot 3.x e Java 21+.
>
> **Tópicos abordados:**
>
> - Otimização de código Java e Spring Boot: lazy loading, N+1, projeções, batch processing
> - Connection pooling com HikariCP e tuning de pool de threads
> - Cache com Spring Cache, Caffeine e Redis, incluindo estratégias de invalidação
> - Processamento assíncrono com `@Async`, Virtual Threads e Structured Concurrency
> - Configurações de JVM: garbage collectors (G1, ZGC, Shenandoah), flags de memória e ergonomics
> - GraalVM Native Image: considerações de performance
> - Configuração de sistema operacional: Ubuntu e Alpine Linux, Docker e on-premise
> - Profiling com JFR, JMC e async-profiler
> - Observabilidade com Micrometer, Prometheus, Grafana e OpenTelemetry
> - Benchmarking com JMH
> - Testes de carga com Gatling, k6 e JMeter
>
> Os exemplos seguem Spring Boot 3.x, Java 21+ e Jakarta EE.

---

## Sumário

**Parte 1 — Otimização no Nível de Código**

1. [Visão Geral — Onde Investir em Performance](#1-visão-geral--onde-investir-em-performance)
2. [Otimização de Queries JPA/Hibernate](#2-otimização-de-queries-jpahibernate)
3. [Connection Pooling com HikariCP](#3-connection-pooling-com-hikaricp)
4. [Cache — Spring Cache, Caffeine e Redis](#4-cache--spring-cache-caffeine-e-redis)
5. [Otimização de Serialização JSON (Jackson)](#5-otimização-de-serialização-json-jackson)
6. [HTTP Client — RestClient, WebClient e Connection Pooling](#6-http-client--restclient-webclient-e-connection-pooling)
7. [Processamento Assíncrono e Paralelo](#7-processamento-assíncrono-e-paralelo)
8. [Virtual Threads e Structured Concurrency](#8-virtual-threads-e-structured-concurrency)
9. [Otimização de Collections e Algoritmos](#9-otimização-de-collections-e-algoritmos)
10. [Lazy Initialization e Startup Otimizado](#10-lazy-initialization-e-startup-otimizado)

**Parte 2 — Configuração de Propriedades e Infraestrutura**

11. [Tuning do Spring Boot — application.yml](#11-tuning-do-spring-boot--applicationyml)
12. [Thread Pool Tuning — Tomcat, Undertow e Jetty](#12-thread-pool-tuning--tomcat-undertow-e-jetty)
13. [Tuning de Banco de Dados — PostgreSQL](#13-tuning-de-banco-de-dados--postgresql)
14. [GraalVM Native Image — Considerações de Performance](#14-graalvm-native-image--considerações-de-performance)

**Parte 3 — Configuração da JVM**

15. [Memória e Garbage Collection — Fundamentos](#15-memória-e-garbage-collection--fundamentos)
16. [Garbage Collectors — G1, ZGC e Shenandoah](#16-garbage-collectors--g1-zgc-e-shenandoah)
17. [JVM Flags e Ergonomics](#17-jvm-flags-e-ergonomics)
18. [JVM em Containers — Limites de CPU e Memória](#18-jvm-em-containers--limites-de-cpu-e-memória)

**Parte 4 — Configuração do Sistema Operacional**

19. [Linux Kernel Tuning — Ubuntu e Alpine](#19-linux-kernel-tuning--ubuntu-e-alpine)
20. [Docker — Otimização de Imagens e Runtime](#20-docker--otimização-de-imagens-e-runtime)
21. [Limites de Recursos em Docker e Kubernetes](#21-limites-de-recursos-em-docker-e-kubernetes)

**Parte 5 — Profiling, Tracing e Observabilidade**

22. [Profiling com JFR e JMC](#22-profiling-com-jfr-e-jmc)
23. [async-profiler — CPU e Allocation Profiling](#23-async-profiler--cpu-e-allocation-profiling)
24. [Observabilidade com Micrometer, Prometheus e Grafana](#24-observabilidade-com-micrometer-prometheus-e-grafana)
25. [Distributed Tracing com OpenTelemetry](#25-distributed-tracing-com-opentelemetry)
26. [Spring Boot Actuator — Endpoints de Performance](#26-spring-boot-actuator--endpoints-de-performance)

**Parte 6 — Benchmarking e Testes de Carga**

27. [Microbenchmarks com JMH](#27-microbenchmarks-com-jmh)
28. [Testes de Carga — Gatling, k6 e JMeter](#28-testes-de-carga--gatling-k6-e-jmeter)

**Parte 7 — Boas Práticas**

29. [Boas Práticas e Checklist de Performance](#29-boas-práticas-e-checklist-de-performance)
30. [Anti-Patterns de Performance](#30-anti-patterns-de-performance)
31. [Referências e Leitura Complementar](#31-referências-e-leitura-complementar)

---

**Parte 1 — Otimização no Nível de Código**


## 1. Visão Geral — Onde Investir em Performance

Antes de otimizar qualquer coisa, é fundamental entender **onde** o tempo e gasto. A maioria dos problemas de performance em aplicações web esta concentrada em poucas areas — e quase sempre o banco de dados é a rede são os maiores viloes.

### 1.1 Pirâmide de Otimização

A piramide abaixo mostra as camadas de otimização, da base (maior impacto) ao topo (menor impacto). Investir na base traz retorno muito maior do que micro-otimizações de código:

```
                    /\
                   /  \
                  / Cod \        <-- Algoritmos, Collections, Streams
                 /  igo  \
                /----------\
               / Config &   \    <-- application.yml, pool sizes, cache
              / Properties   \
             /----------------\
            /   JVM Tuning     \  <-- GC, heap, JIT, flags
           /                    \
          /----------------------\
         /  SO / Infra / Rede     \ <-- CPU, RAM, disco, rede, containers
        /--------------------------\
```

**Regra de ouro:** otimizações na base da piramide (infraestrutura, JVM) afetam **toda** a aplicação. Otimizacoes no topo (código) afetam apenas o trecho modificado.

### 1.2 Numeros de Latência que Todo Desenvolvedor Deve Conhecer

Estes números ajudam a entender a **ordem de grandeza** das operações. São aproximados e variam por hardware, mas as proporções se mantém:

| Operação                              | Latência Aproximada | Comparação       |
|---------------------------------------|---------------------|------------------|
| Referência L1 cache                   | ~1 ns               | 1x (baseline)    |
| Referência L2 cache                   | ~4 ns               | 4x               |
| Referência L3 cache                   | ~10 ns              | 10x              |
| Referência memória RAM                | ~100 ns             | 100x             |
| Leitura SSD (NVMe, 4KB)              | ~10 us              | 10.000x          |
| Leitura HDD (seek)                   | ~5 ms               | 5.000.000x       |
| Round-trip rede local (datacenter)    | ~500 us             | 500.000x         |
| Round-trip rede internet (continente) | ~50-150 ms          | 50.000.000x      |
| Query simples PostgreSQL (local)      | ~1-5 ms             | 1.000.000x       |
| Query complexa com JOINs             | ~10-100 ms          | 10.000.000x      |
| Chamada REST para outro serviço       | ~5-50 ms            | 5.000.000x       |
| Handshake TLS                         | ~5-30 ms            | 5.000.000x       |

**Conclusão prática:** uma única query SQL mal escrita ou uma chamada de rede desnecessária custa **milhões de vezes mais** do que qualquer otimização de código em memória.

### 1.3 Principio Fundamental: Meca Antes de Otimizar

> *"Premature optimization is the root of all evil"* — Donald Knuth

Nunca otimize com base em intuição. O fluxo correto e:

```
  [1. Medir]  -->  [2. Identificar gargalo]  -->  [3. Otimizar]  -->  [4. Medir novamente]
       ^                                                                      |
       |______________________________________________________________________|
                              (repetir até atingir meta)
```

**Ferramentas essenciais para medição:**

| Ferramenta           | Uso                                      |
|----------------------|------------------------------------------|
| Spring Boot Actuator | Métricas da aplicação, health checks      |
| Micrometer           | Coleta de métricas (timer, counter, gauge)|
| Prometheus + Grafana | Armazenamento e visualização de métricas  |
| JFR (Flight Recorder)| Profiling de CPU, memória, locks          |
| async-profiler       | Profiling de CPU e alocação (low overhead)|
| EXPLAIN ANALYZE      | Análise de plano de execução SQL          |
| p6spy / datasource-proxy | Log de queries SQL com tempo         |
| JMH                  | Microbenchmarks de métodos Java           |

```java
// Exemplo: medindo tempo de execução com Micrometer
@RestController
@RequiredArgsConstructor
public class ProdutoController {

    private final ProdutoService produtoService;
    private final MeterRegistry meterRegistry;

    @GetMapping("/produtos")
    public List<ProdutoDTO> listar() {
        return meterRegistry.timer("produtos.listar")
                .record(() -> produtoService.listarTodos());
    }
}
```

> Para detalhes sobre logging, métricas e observabilidade, consulte [Dicas-Logs-Observabilidade.md](Dicas-Logs-Observabilidade.md).

---

## 2. Otimização de Queries JPA/Hibernate

O Hibernate é uma ferramenta poderosa, mas queries mal configuradas são a causa número um de problemas de performance em aplicações Spring Boot. Esta seção cobre as otimizações mais impactantes.

### 2.1 O Problema N+1 — Detecção e Correção

O problema N+1 ocorre quando o Hibernate executa 1 query para buscar a entidade principal e **N queries adicionais** para carregar os relacionamentos de cada registro.

**Exemplo do problema — código que gera N+1:**

```java
@Entity
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDate data;

    @ManyToOne(fetch = FetchType.LAZY)
    private Cliente cliente;

    @OneToMany(mappedBy = "pedido", fetch = FetchType.LAZY)
    private List<ItemPedido> itens;

    // getters/setters
}
```

```java
// PROBLEMA: gera N+1 queries!
@Service
public class PedidoService {

    @Autowired
    private PedidoRepository pedidoRepository;

    public List<PedidoResumoDTO> listarPedidos() {
        List<Pedido> pedidos = pedidoRepository.findAll(); // 1 query

        return pedidos.stream()
                .map(p -> new PedidoResumoDTO(
                        p.getId(),
                        p.getData(),
                        p.getCliente().getNome(),  // +1 query por pedido (N queries)
                        p.getItens().size()         // +1 query por pedido (N queries)
                ))
                .toList();
    }
}
```

Se houver 100 pedidos, o código acima executa **1 + 100 + 100 = 201 queries!**

**Solução 1 — JOIN FETCH na query JPQL:**

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    @Query("""
        SELECT p FROM Pedido p
        JOIN FETCH p.cliente
        JOIN FETCH p.itens
        WHERE p.data >= :dataInicio
        """)
    List<Pedido> findPedidosComClienteEItens(@Param("dataInicio") LocalDate dataInicio);
}
```

> **Atenção:** não é possível usar `JOIN FETCH` em duas coleções (`@OneToMany`) simultaneamente — o Hibernate lança `MultipleBagFetchException`. Para esse cenário, use `@BatchSize` ou queries separadas.

**Solução 2 — @EntityGraph (declarativo):**

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    @EntityGraph(attributePaths = {"cliente", "itens"})
    @Query("SELECT p FROM Pedido p WHERE p.data >= :dataInicio")
    List<Pedido> findPedidosComGrafo(@Param("dataInicio") LocalDate dataInicio);
}
```

**Solução 3 — @BatchSize (carregamento em lotes):**

```java
@Entity
public class Pedido {

    @OneToMany(mappedBy = "pedido", fetch = FetchType.LAZY)
    @BatchSize(size = 50) // Carrega itens em lotes de 50
    private List<ItemPedido> itens;
}
```

Com `@BatchSize(size = 50)`, ao inves de 100 queries individuais, o Hibernate executa apenas **2 queries** (100 / 50 = 2 lotes) usando cláusula `IN`.

**Configuração global de batch size (recomendado):**

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 50
```

### 2.2 Projeções para Leitura

Carregar entidades completas quando você precisa apenas de alguns campos é um desperdício de memória e processamento. Use projeções.

**Projeção baseada em interface (interface-based projection):**

```java
// Projecao — apenas os campos necessários
public interface PedidoResumoProjection {
    Long getId();
    LocalDate getData();
    String getClienteNome(); // Resolve automaticamente: cliente.nome

    @Value("#{target.itens.size()}")
    int getQuantidadeItens();
}

public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Retorna apenas os campos definidos na projeção
    List<PedidoResumoProjection> findByDataAfter(LocalDate data);
}
```

**Projeção baseada em DTO (mais performática — recomendada para leitura):**

```java
// DTO record (Java 17+)
public record PedidoResumoDTO(
        Long id,
        LocalDate data,
        String clienteNome,
        BigDecimal valorTotal
) {}

public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    @Query("""
        SELECT new com.example.dto.PedidoResumoDTO(
            p.id,
            p.data,
            c.nome,
            SUM(i.preco * i.quantidade)
        )
        FROM Pedido p
        JOIN p.cliente c
        JOIN p.itens i
        WHERE p.data >= :dataInicio
        GROUP BY p.id, p.data, c.nome
        """)
    List<PedidoResumoDTO> findResumoPedidos(@Param("dataInicio") LocalDate dataInicio);
}
```

**Comparação de performance:**

| Abordagem               | Dados Trafegados | First-Level Cache | Overhead |
|--------------------------|------------------|-------------------|----------|
| Entidade completa        | Todos os campos  | Sim               | Alto     |
| Interface projection     | Campos selecionados | Sim (proxy)    | Médio    |
| DTO projection (JPQL)    | Campos selecionados | Não             | Baixo    |
| Native query + DTO       | Campos selecionados | Não             | Mínimo   |

### 2.3 Paginação Eficiente — Keyset vs Offset

**Por que OFFSET degrada performance:**

A paginação com `OFFSET` obriga o banco a **percorrer e descartar** todas as linhas anteriores. Na página 1000, o banco lê 10.000 linhas para retornar apenas 10.

```sql
-- OFFSET: lento para paginas grandes
SELECT * FROM pedido ORDER BY id LIMIT 10 OFFSET 99990;
-- O banco lê 100.000 linhas e descarta 99.990!
```

```
  Página 1:    OFFSET 0     --> lê 10 linhas            (rápido)
  Página 100:  OFFSET 990   --> lê 1.000, descarta 990  (ok)
  Página 10000: OFFSET 99990 --> lê 100.000, descarta 99.990 (LENTO!)
```

**Paginação com Offset (simples, mas degrada):**

```java
// Paginacao offset — padrão do Spring Data
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    Page<PedidoResumoDTO> findByStatus(StatusPedido status, Pageable pageable);
}

// No controller
@GetMapping("/pedidos")
public Page<PedidoResumoDTO> listar(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    return pedidoRepository.findByStatus(
            StatusPedido.ATIVO,
            PageRequest.of(page, size, Sort.by("id").descending())
    );
}
```

**Paginação com Keyset (cursor-based — performática):**

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Keyset pagination: usa o último ID como cursor
    @Query("""
        SELECT p FROM Pedido p
        WHERE p.status = :status
        AND p.id < :ultimoId
        ORDER BY p.id DESC
        """)
    List<Pedido> findByStatusComCursor(
            @Param("status") StatusPedido status,
            @Param("ultimoId") Long ultimoId,
            Pageable pageable
    );

    // Primeira página (sem cursor)
    @Query("""
        SELECT p FROM Pedido p
        WHERE p.status = :status
        ORDER BY p.id DESC
        """)
    List<Pedido> findByStatusPrimeiraPagina(
            @Param("status") StatusPedido status,
            Pageable pageable
    );
}
```

```java
// Controller com keyset pagination
@GetMapping("/pedidos")
public CursorPageResponse<PedidoResumoDTO> listar(
        @RequestParam(required = false) Long cursor,
        @RequestParam(defaultValue = "20") int size) {

    Pageable pageable = PageRequest.of(0, size);

    List<Pedido> pedidos = (cursor == null)
            ? pedidoRepository.findByStatusPrimeiraPagina(StatusPedido.ATIVO, pageable)
            : pedidoRepository.findByStatusComCursor(StatusPedido.ATIVO, cursor, pageable);

    Long proximoCursor = pedidos.isEmpty()
            ? null
            : pedidos.getLast().getId();

    List<PedidoResumoDTO> dtos = pedidos.stream()
            .map(PedidoResumoDTO::fromEntity)
            .toList();

    return new CursorPageResponse<>(dtos, proximoCursor, pedidos.size() == size);
}

public record CursorPageResponse<T>(
        List<T> items,
        Long nextCursor,
        boolean hasMore
) {}
```

```sql
-- Keyset: sempre rápido, independente da "página"
SELECT * FROM pedido WHERE status = 'ATIVO' AND id < 50000 ORDER BY id DESC LIMIT 10;
-- Usa índice, sem scan desnecessário
```

**Comparação Offset vs Keyset:**

| Aspecto              | Offset              | Keyset (Cursor)        |
|----------------------|---------------------|------------------------|
| Performance          | Degrada com páginas | Constante              |
| "Ir para página X"   | Sim                 | Não                    |
| Dados inseridos/removidos | Pode pular/repetir | Consistente         |
| Implementação        | Simples (Spring)    | Mais código            |
| Uso recomendado      | Admin panels        | APIs publicas, feeds   |

### 2.4 Bulk Operations — INSERT e UPDATE em Lote

Por padrão, o Hibernate executa um INSERT por entidade. Para inserções em massa, isso e extremamente lento.

**Configuração para batch insert/update:**

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
          batch_versioned_data: true
        order_inserts: true
        order_updates: true

  # PostgreSQL: reescreve batches em multi-row INSERT (muito mais rápido)
  datasource:
    url: jdbc:postgresql://localhost:5432/app?rewriteBatchedInserts=true
```

**Código para inserção em lote com flush periódico:**

```java
@Service
@RequiredArgsConstructor
public class ProdutoImportService {

    private final EntityManager entityManager;
    private static final int BATCH_SIZE = 50;

    @Transactional
    public int importarProdutos(List<ProdutoCSV> produtosCSV) {
        int count = 0;

        for (ProdutoCSV csv : produtosCSV) {
            Produto produto = new Produto();
            produto.setNome(csv.nome());
            produto.setPreco(csv.preco());
            produto.setCategoria(csv.categoria());

            entityManager.persist(produto);
            count++;

            // Flush e limpa o persistence context a cada lote
            if (count % BATCH_SIZE == 0) {
                entityManager.flush();
                entityManager.clear(); // Libera memória
            }
        }

        // Flush dos registros restantes
        entityManager.flush();
        entityManager.clear();

        return count;
    }
}
```

**Alternativa mais performática — JDBC direto para inserções massivas:**

```java
@Repository
@RequiredArgsConstructor
public class ProdutoBatchRepository {

    private final JdbcTemplate jdbcTemplate;

    public void insertBatch(List<Produto> produtos) {
        String sql = "INSERT INTO produto (nome, preco, categoria) VALUES (?, ?, ?)";

        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Produto p = produtos.get(i);
                ps.setString(1, p.getNome());
                ps.setBigDecimal(2, p.getPreco());
                ps.setString(3, p.getCategoria().name());
            }

            @Override
            public int getBatchSize() {
                return produtos.size();
            }
        });
    }
}
```

### 2.5 Read-Only Transactions

Marcar transações como somente leitura permite ao Hibernate desativar o **dirty checking** (verificação de mudanças), economizando CPU e memória.

```java
@Service
@RequiredArgsConstructor
public class RelatorioService {

    private final PedidoRepository pedidoRepository;

    // Hibernate pula o dirty checking — mais rápido e usa menos memória
    @Transactional(readOnly = true)
    public List<PedidoResumoDTO> gerarRelatorio(LocalDate inicio, LocalDate fim) {
        return pedidoRepository.findResumoPorPeriodo(inicio, fim);
    }
}
```

**Beneficios de `readOnly = true`:**

| Benefício                           | Descrição                                    |
|-------------------------------------|----------------------------------------------|
| Sem dirty checking                  | Hibernate não compara snapshots              |
| Sem flush automático                | Não tenta sincronizar com o banco            |
| Hint para connection pool           | Pode rotear para read réplica                |
| Menor consumo de memória            | Não mantém snapshot das entidades            |

**Dica:** configure o routing para read réplica automaticamente:

```java
@Configuration
public class ReadOnlyRoutingConfig {

    @Bean
    public DataSource routingDataSource(
            @Qualifier("primaryDataSource") DataSource primary,
            @Qualifier("replicaDataSource") DataSource replica) {

        var routing = new AbstractRoutingDataSource() {
            @Override
            protected Object determineCurrentLookupKey() {
                return TransactionSynchronizationManager
                        .isCurrentTransactionReadOnly() ? "replica" : "primary";
            }
        };

        routing.setTargetDataSources(Map.of(
                "primary", primary,
                "replica", replica
        ));
        routing.setDefaultTargetDataSource(primary);

        return routing;
    }
}
```

### 2.6 Índices — Estratégia e Criação

Índices são a otimização de banco de dados com maior impacto. Um índice bem posicionado pode transformar uma query de 5 segundos em 5 milissegundos.

**Criando índices com JPA:**

```java
@Entity
@Table(
    name = "pedido",
    indexes = {
        @Index(name = "idx_pedido_status_data", columnList = "status, data DESC"),
        @Index(name = "idx_pedido_cliente", columnList = "cliente_id"),
        @Index(name = "idx_pedido_numero", columnList = "numero", unique = true)
    }
)
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    private StatusPedido status;

    private LocalDate data;

    @Column(unique = true)
    private String numero;

    @ManyToOne(fetch = FetchType.LAZY)
    private Cliente cliente;
}
```

**Covering index (índice que cobre a query inteira):**

```sql
-- PostgreSQL: covering index evita acesso a tabela (Index Only Scan)
CREATE INDEX idx_pedido_covering ON pedido (status, data DESC)
    INCLUDE (id, numero, cliente_id);

-- A query abaixo será resolvida SOMENTE pelo índice:
SELECT id, numero, cliente_id FROM pedido
WHERE status = 'ATIVO' ORDER BY data DESC LIMIT 20;
```

**Partial index (índice para subconjunto de dados):**

```sql
-- Indexa apenas pedidos ativos — índice menor é mais rápido
CREATE INDEX idx_pedido_ativo ON pedido (data DESC)
    WHERE status = 'ATIVO';
```

**Usando EXPLAIN ANALYZE para diagnóstico:**

```sql
EXPLAIN ANALYZE
SELECT p.id, p.numero, c.nome
FROM pedido p
JOIN cliente c ON c.id = p.cliente_id
WHERE p.status = 'ATIVO'
AND p.data >= '2024-01-01'
ORDER BY p.data DESC
LIMIT 20;
```

Resultado típico **sem índice**:
```
Seq Scan on pedido  (cost=0.00..25000.00 rows=50000 width=48)
                     (actual time=0.020..450.123 rows=50000 loops=1)
Planning Time: 0.150 ms
Execution Time: 523.456 ms
```

Resultado **com índice composto**:
```
Index Scan using idx_pedido_status_data on pedido
                     (cost=0.42..15.30 rows=20 width=48)
                     (actual time=0.025..0.089 rows=20 loops=1)
Planning Time: 0.200 ms
Execution Time: 0.132 ms
```

### 2.7 Hibernate Statistics e Logging de Queries Lentas

**Habilitando estatisticas do Hibernate:**

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        generate_statistics: true

logging:
  level:
    org.hibernate.stat: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.orm.jdbc.bind: TRACE  # Mostra parâmetros (Hibernate 6+)
```

**Interceptando queries lentas com datasource-proxy:**

```java
@Configuration
public class DataSourceProxyConfig {

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        var realDataSource = properties.initializeDataSourceBuilder().build();

        var queryListener = new QueryExecutionListener() {
            private static final Logger log =
                    LoggerFactory.getLogger("SlowQueryLogger");

            @Override
            public void afterQuery(ExecutionInfo execInfo,
                                   List<QueryInfo> queryInfoList) {
                long thresholdMs = 500; // Log queries acima de 500ms

                if (execInfo.getElapsedTime() > thresholdMs) {
                    for (QueryInfo qi : queryInfoList) {
                        log.warn("Query lenta ({}ms): {}",
                                execInfo.getElapsedTime(),
                                qi.getQuery());
                    }
                }
            }
        };

        return ProxyDataSourceBuilder.create(realDataSource)
                .name("slow-query-detector")
                .listener(queryListener)
                .countQuery()
                .build();
    }
}
```

**Dependência necessária:**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>net.ttddyy</groupId>
    <artifactId>datasource-proxy</artifactId>
    <version>1.10</version>
</dependency>
```

> Para detalhes sobre queries JPA, mapeamentos e relacionamentos, consulte [Dicas-JPA-Hibernate-Queries.md](Dicas-JPA-Hibernate-Queries.md).

---

## 3. Connection Pooling com HikariCP

O HikariCP é o connection pool padrão do Spring Boot é um dos mais rapidos disponíveis. Uma configuração adequada é essencial para evitar gargalos de conexão.

### 3.1 Como o Connection Pool Funciona

Abrir e fechar conexões com o banco é uma operação **cara** (handshake TCP, autenticação, alocação de memória no servidor). O connection pool mantém conexões já abertas e as reutiliza:

```
  Sem pool:
  Request 1 --> [Abrir conexão] --> Query --> [Fechar conexão]
  Request 2 --> [Abrir conexão] --> Query --> [Fechar conexão]    (lento!)
  Request 3 --> [Abrir conexão] --> Query --> [Fechar conexão]

  Com pool (HikariCP):
                    +----------------------------------+
                    |         HikariCP Pool            |
                    |   +------+  +------+  +------+   |
  Request 1 ------>|   |Conn 1|  |Conn 2|  |Conn 3|   |------> PostgreSQL
  Request 2 ------>|   |      |  |      |  |      |   |
  Request 3 ------>|   |(busy)|  |(idle)|  |(idle)|   |
                    |   +------+  +------+  +------+   |
  Request 4 -----X |   [aguarda liberação...]          |
                    +----------------------------------+
```

### 3.2 Configuração Recomendada para Produção

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/app?rewriteBatchedInserts=true
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver

    hikari:
      # === Tamanho do Pool ===
      # Formula: pool_size = (core_count * 2) + spindle_count
      # Para SSD: spindle_count = 1. Ex: 4 cores -> (4*2)+1 = 9
      # Na prática, entre 10-20 é suficiente para a maioria das aplicações
      maximum-pool-size: 10
      minimum-idle: 5            # Conexoes ociosas mantidas prontas

      # === Timeouts ===
      connection-timeout: 10000  # 10s - tempo max para obter conexão do pool
      idle-timeout: 300000       # 5min - tempo max de conexão ociosa
      max-lifetime: 1800000      # 30min - tempo max de vida de uma conexão
      keepalive-time: 60000      # 1min - envia query de keepalive (evita timeout do firewall)

      # === Validacao ===
      validation-timeout: 5000   # 5s - tempo max para validar conexão

      # === Debug ===
      leak-detection-threshold: 30000  # 30s - alerta se conexão não for devolvida
      pool-name: AppHikariPool

      # === Performance ===
      auto-commit: false         # Desativar auto-commit (Spring gerência transações)
      data-source-properties:
        prepStmtCacheSize: 250
        prepStmtCacheSqlLimit: 2048
        useServerPrepStmts: true  # PostgreSQL: prepared statements no servidor
```

**Formula para dimensionar o pool:**

```
  Conexões necessárias = Threads simultâneas que usam banco

  Para Spring Boot com Tomcat padrão (200 threads):
  - NEM TODAS as threads usam banco ao mesmo tempo
  - Pool de 10-20 conexões atende a maioria dos cenários
  - NUNCA coloque pool_size = thread_count (desperdiçado)

  Para Virtual Threads (milhares de threads):
  - Pool AINDA deve ser pequeno (10-20)
  - O pool limita concorrência no banco (que é o recurso escasso)
```

### 3.3 Métricas do HikariCP com Micrometer

O HikariCP exporta métricas automaticamente quando o Micrometer esta no classpath.

**Dependencias:**

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Métricas mais importantes para monitorar:**

| Métrica Prometheus                          | Descrição                         | Alerta Sugerido         |
|---------------------------------------------|-----------------------------------|-------------------------|
| `hikaricp_connections_active`               | Conexoes em uso                   | > 80% do max por 5min   |
| `hikaricp_connections_idle`                 | Conexoes ociosas                  | —                       |
| `hikaricp_connections_pending`              | Threads aguardando conexão        | > 0 por mais de 10s     |
| `hikaricp_connections_timeout_total`        | Total de timeouts                 | Qualquer incremento     |
| `hikaricp_connections_creation_seconds`     | Tempo para criar conexão          | p95 > 1s                |
| `hikaricp_connections_usage_seconds`        | Tempo de uso da conexão           | p95 > 5s (possível leak)|

**Queries Grafana úteis:**

```promql
# Porcentagem de uso do pool
hikaricp_connections_active{pool="AppHikariPool"}
  / hikaricp_connections_max{pool="AppHikariPool"} * 100

# Threads esperando por conexão (crítico se > 0 por muito tempo)
hikaricp_connections_pending{pool="AppHikariPool"}

# Taxa de timeouts por minuto
rate(hikaricp_connections_timeout_total{pool="AppHikariPool"}[5m]) * 60
```

### 3.4 Diagnosticando Connection Leaks

Um **connection leak** ocorre quando o código obtém uma conexão do pool mas não a devolve. Com o tempo, o pool esgota é a aplicação trava.

**Configuração para detectar leaks:**

```yaml
spring:
  datasource:
    hikari:
      leak-detection-threshold: 30000  # Alerta após 30 segundos
```

**Código que causa leak (ERRADO):**

```java
// ERRADO: conexão nunca e devolvida se ocorrer exceção!
@Service
public class RelatorioService {

    @Autowired
    private DataSource dataSource;

    public List<Map<String, Object>> gerarRelatorio() {
        try {
            Connection conn = dataSource.getConnection();  // Obtem conexão
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM pedido");

            List<Map<String, Object>> result = new ArrayList<>();
            while (rs.next()) {
                // processa...
                result.add(Map.of("id", rs.getLong("id")));
            }
            conn.close();  // Se houver exceção ANTES, a conexão vaza!
            return result;

        } catch (SQLException e) {
            throw new RuntimeException(e);
            // Conexao não foi fechada! LEAK!
        }
    }
}
```

**Código correto (try-with-resources):**

```java
@Service
public class RelatorioService {

    @Autowired
    private DataSource dataSource;

    public List<Map<String, Object>> gerarRelatorio() {
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM pedido")) {

            List<Map<String, Object>> result = new ArrayList<>();
            while (rs.next()) {
                result.add(Map.of("id", rs.getLong("id")));
            }
            return result;

        } catch (SQLException e) {
            throw new RuntimeException(e);
            // Conexao fechada automaticamente pelo try-with-resources
        }
    }
}
```

**Melhor ainda — use Spring JdbcTemplate ou JPA (gerenciam conexões automaticamente):**

```java
@Service
@RequiredArgsConstructor
public class RelatorioService {

    private final JdbcTemplate jdbcTemplate;

    public List<Map<String, Object>> gerarRelatorio() {
        // JdbcTemplate obtém e devolve a conexão automaticamente
        return jdbcTemplate.queryForList("SELECT id, numero FROM pedido");
    }
}
```

### 3.5 Connection Pool para Múltiplas Datasources

Em cenários de read/write split, você pode configurar datasources separadas:

```java
@Configuration
public class MultiDataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSourceProperties primaryDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    @Primary
    public DataSource primaryDataSource(
            @Qualifier("primaryDataSourceProperties")
            DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.replica")
    public DataSourceProperties replicaDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource replicaDataSource(
            @Qualifier("replicaDataSourceProperties")
            DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }
}
```

```yaml
# application.yml
spring:
  datasource:
    primary:
      url: jdbc:postgresql://primary-host:5432/app
      username: ${DB_PRIMARY_USER}
      password: ${DB_PRIMARY_PASSWORD}
      hikari:
        maximum-pool-size: 10
        pool-name: PrimaryPool

    replica:
      url: jdbc:postgresql://réplica-host:5432/app
      username: ${DB_REPLICA_USER}
      password: ${DB_REPLICA_PASSWORD}
      hikari:
        maximum-pool-size: 15       # Mais conexões para leitura
        pool-name: ReplicaPool
        read-only: true
```

---

## 4. Cache — Spring Cache, Caffeine e Redis

Cache é uma das otimizações com maior impacto: eliminar completamente a necessidade de reprocessar ou rebuscar dados. Mas cache mal implementado causa bugs dificeis de rastrear.

### 4.1 Quando Usar Cache

Nem tudo deve ser cacheado. Use este fluxograma para decidir:

```
  Dado muda com frequência?
       |
       +-- SIM --> Dado é consultado com muita frequência?
       |               |
       |               +-- SIM --> Cache com TTL curto (segundos)
       |               |
       |               +-- NÃO --> NÃO use cache
       |
       +-- NÃO --> Dado é caro de obter? (query lenta, API externa, cálculo pesado)
                       |
                       +-- SIM --> Cache com TTL longo (minutos/horas)
                       |
                       +-- NÃO --> Dado é consultado frequentemente?
                                       |
                                       +-- SIM --> Cache com TTL médio
                                       |
                                       +-- NÃO --> NÃO use cache

  Dica: Cache ideal = leitura frequente + escrita rara + tolerância a dados levemente stale
```

**Exemplos práticos de quando cachear:**

| Cenário                          | Cachear? | TTL Sugerido |
|----------------------------------|----------|--------------|
| Configurações do sistema         | Sim      | 10-30 min    |
| Catalogo de produtos             | Sim      | 5-15 min     |
| Lista de estados/cidades         | Sim      | 24 horas     |
| Resultado de busca personalizada | Talvez   | 30-60 seg    |
| Saldo bancario                   | Não      | —            |
| Dados de sessão do usuario       | Sim      | Tempo sessão |
| Resposta de API externa (CEP)    | Sim      | 24 horas     |
| Dashboard com métricas           | Sim      | 30-60 seg    |

### 4.2 Spring Cache Abstraction

O Spring oferece uma abstraction de cache via anotações, desacoplando o código da implementação específica (Caffeine, Redis, etc.).

**Habilitando cache:**

```java
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Usando anotações de cache:**

```java
@Service
@RequiredArgsConstructor
public class ProdutoService {

    private final ProdutoRepository produtoRepository;

    // @Cacheable: retorna do cache se existir, senão executa o método
    @Cacheable(value = "produtos", key = "#id")
    public ProdutoDTO buscarPorId(Long id) {
        return produtoRepository.findById(id)
                .map(ProdutoDTO::fromEntity)
                .orElseThrow(() -> new ProdutoNaoEncontradoException(id));
    }

    // Cache com chave composta
    @Cacheable(value = "produtos-busca", key = "#categoria + '-' + #pagina")
    public List<ProdutoDTO> buscarPorCategoria(String categoria, int pagina) {
        var pageable = PageRequest.of(pagina, 20);
        return produtoRepository.findByCategoria(categoria, pageable)
                .map(ProdutoDTO::fromEntity)
                .getContent();
    }

    // @CachePut: atualiza o cache após execução do método
    @CachePut(value = "produtos", key = "#result.id")
    @CacheEvict(value = "produtos-busca", allEntries = true)
    public ProdutoDTO atualizar(Long id, ProdutoUpdateRequest request) {
        Produto produto = produtoRepository.findById(id)
                .orElseThrow(() -> new ProdutoNaoEncontradoException(id));
        produto.setNome(request.nome());
        produto.setPreco(request.preco());
        return ProdutoDTO.fromEntity(produtoRepository.save(produto));
    }

    // @CacheEvict: remove entrada do cache
    @CacheEvict(value = {"produtos", "produtos-busca"}, allEntries = true)
    public void excluir(Long id) {
        produtoRepository.deleteById(id);
    }

    // Cache condicional: só cacheia se o resultado não for vazio
    @Cacheable(value = "produtos", key = "#id",
               unless = "#result == null")
    public ProdutoDTO buscarOpcional(Long id) {
        return produtoRepository.findById(id)
                .map(ProdutoDTO::fromEntity)
                .orElse(null);
    }
}
```

### 4.3 Caffeine — Cache Local de Alta Performance

Caffeine é o cache local mais rápido para Java, com políticas de evicção baseadas em Window TinyLfu (melhor hit rate que LRU).

**Dependência:**

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Configuração via application.yml:**

```yaml
# application.yml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=10m,recordStats
    cache-names:
      - produtos
      - categorias
      - configuracoes
```

**Configuração programática (mais controle por cache):**

```java
@Configuration
@EnableCaching
public class CaffeineCacheConfig {

    @Bean
    public CacheManager cacheManager() {
        var caffeineCacheManager = new CaffeineCacheManager();

        // Configuracao padrão
        caffeineCacheManager.setCaffeine(
                Caffeine.newBuilder()
                        .maximumSize(500)
                        .expireAfterWrite(Duration.ofMinutes(10))
                        .recordStats() // Habilita métricas
        );

        return caffeineCacheManager;
    }

    // Alternativa: caches com configurações diferentes
    @Bean
    public CacheManager multiCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();

        cacheManager.setCaches(List.of(
                buildCache("produtos", 1000, Duration.ofMinutes(15)),
                buildCache("categorias", 100, Duration.ofHours(1)),
                buildCache("configuracoes", 50, Duration.ofHours(24)),
                buildCache("usuarios-ativos", 5000, Duration.ofMinutes(5))
        ));

        return cacheManager;
    }

    private CaffeineCache buildCache(String name, int maxSize, Duration ttl) {
        return new CaffeineCache(name,
                Caffeine.newBuilder()
                        .maximumSize(maxSize)
                        .expireAfterWrite(ttl)
                        .recordStats()
                        .build());
    }
}
```

**Monitorando estatisticas do Caffeine:**

```java
@Component
@RequiredArgsConstructor
public class CacheMetricsExporter {

    private final CacheManager cacheManager;
    private final MeterRegistry meterRegistry;

    @Scheduled(fixedRate = 60_000)
    public void exportCacheMetrics() {
        cacheManager.getCacheNames().forEach(name -> {
            Cache cache = cacheManager.getCache(name);
            if (cache != null && cache.getNativeCache() instanceof
                    com.github.benmanes.caffeine.cache.Cache<?, ?> caffeineCache) {

                CacheStats stats = caffeineCache.stats();

                meterRegistry.gauge("cache.hit.rate", Tags.of("cache", name),
                        stats, CacheStats::hitRate);
                meterRegistry.gauge("cache.size", Tags.of("cache", name),
                        caffeineCache, com.github.benmanes.caffeine.cache.Cache::estimatedSize);
                meterRegistry.gauge("cache.eviction.count", Tags.of("cache", name),
                        stats, CacheStats::evictionCount);
            }
        });
    }
}
```

### 4.4 Redis — Cache Distribuido

Enquanto o Caffeine e local (por instância), o Redis é um cache **distribuido** — compartilhado entre todas as instâncias da aplicação.

**Quando usar Redis vs Caffeine:**

| Aspecto                   | Caffeine (Local)     | Redis (Distribuido)        |
|---------------------------|----------------------|----------------------------|
| Latência de acesso        | ~100 ns (nanoseg)    | ~1-2 ms (rede)             |
| Compartilhado entre pods  | Não                  | Sim                        |
| Sobrevive ao restart      | Não                  | Sim                        |
| Capacidade                | Limitada pela heap   | GBs/TBs (servidor externo) |
| Complexidade operacional  | Nenhuma              | Requer servidor Redis      |
| Caso de uso               | Hot data, 1 instância| Multi-instância, sessões   |

**Dependência:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**Configuração do RedisCacheManager:**

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {

        // Serializacao JSON (legivel e compatível entre versoes)
        var jsonSerializer = new GenericJackson2JsonRedisSerializer();

        var defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair
                                .fromSerializer(jsonSerializer))
                .disableCachingNullValues()
                .prefixCacheNameWith("app::");

        // TTLs diferentes por cache
        Map<String, RedisCacheConfiguration> cacheConfigs = Map.of(
                "produtos",
                defaultConfig.entryTtl(Duration.ofMinutes(30)),

                "configuracoes",
                defaultConfig.entryTtl(Duration.ofHours(24)),

                "sessoes",
                defaultConfig.entryTtl(Duration.ofMinutes(30))
        );

        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(defaultConfig)
                .withInitialCacheConfigurations(cacheConfigs)
                .transactionAware() // Commit do cache após commit da transação
                .build();
    }
}
```

```yaml
# application.yml
spring:
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 16
          max-idle: 8
          min-idle: 4
          max-wait: 2000ms
```

### 4.5 Cache de Dois Niveis — Caffeine + Redis

O padrão L1 (local) + L2 (distribuido) combina a velocidade do Caffeine com a consistência do Redis:

```
  Request --> [L1: Caffeine]  HIT --> Retorna (< 1ms)
                   |
                  MISS
                   |
              [L2: Redis]    HIT --> Retorna + popula L1 (~2ms)
                   |
                  MISS
                   |
              [Banco/API]    --> Retorna + popula L2 + L1 (~50ms)
```

```java
@Component
public class TwoLevelCacheManager implements CacheManager {

    private final CacheManager caffeineCacheManager;
    private final CacheManager redisCacheManager;

    public TwoLevelCacheManager(
            RedisConnectionFactory redisConnectionFactory) {

        // L1: Caffeine — rápido, pouca capacidade
        this.caffeineCacheManager = new CaffeineCacheManager();
        ((CaffeineCacheManager) this.caffeineCacheManager).setCaffeine(
                Caffeine.newBuilder()
                        .maximumSize(500)
                        .expireAfterWrite(Duration.ofMinutes(5))
        );

        // L2: Redis — mais lento, grande capacidade
        var redisConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair
                                .fromSerializer(new GenericJackson2JsonRedisSerializer()));

        this.redisCacheManager = RedisCacheManager
                .builder(redisConnectionFactory)
                .cacheDefaults(redisConfig)
                .build();
    }

    @Override
    public Cache getCache(String name) {
        Cache caffeineCache = caffeineCacheManager.getCache(name);
        Cache redisCache = redisCacheManager.getCache(name);

        if (caffeineCache == null || redisCache == null) {
            return null;
        }

        return new TwoLevelCache(name, caffeineCache, redisCache);
    }

    @Override
    public Collection<String> getCacheNames() {
        Set<String> names = new HashSet<>();
        names.addAll(caffeineCacheManager.getCacheNames());
        names.addAll(redisCacheManager.getCacheNames());
        return names;
    }
}

/**
 * Cache que consulta L1 (Caffeine) primeiro, depois L2 (Redis).
 */
public class TwoLevelCache implements Cache {

    private final String name;
    private final Cache l1Cache; // Caffeine
    private final Cache l2Cache; // Redis

    public TwoLevelCache(String name, Cache l1, Cache l2) {
        this.name = name;
        this.l1Cache = l1;
        this.l2Cache = l2;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public Object getNativeCache() {
        return l1Cache.getNativeCache();
    }

    @Override
    public ValueWrapper get(Object key) {
        // Tenta L1 primeiro
        ValueWrapper value = l1Cache.get(key);
        if (value != null) {
            return value;
        }

        // Fallback para L2
        value = l2Cache.get(key);
        if (value != null) {
            // Popula L1 para proximas chamadas
            l1Cache.put(key, value.get());
        }

        return value;
    }

    @Override
    public void put(Object key, Object value) {
        l1Cache.put(key, value);
        l2Cache.put(key, value);
    }

    @Override
    public void evict(Object key) {
        l1Cache.evict(key);
        l2Cache.evict(key);
    }

    @Override
    public void clear() {
        l1Cache.clear();
        l2Cache.clear();
    }

    // ... demais métodos do interface Cache
}
```

### 4.6 HTTP Caching — Cache-Control e ETags

Além do cache no servidor, é possível cachear respostas HTTP no lado do cliente/CDN:

**Cache-Control headers:**

```java
@RestController
@RequestMapping("/api/produtos")
@RequiredArgsConstructor
public class ProdutoController {

    private final ProdutoService produtoService;

    @GetMapping("/{id}")
    public ResponseEntity<ProdutoDTO> buscarPorId(@PathVariable Long id) {
        ProdutoDTO produto = produtoService.buscarPorId(id);

        return ResponseEntity.ok()
                .cacheControl(CacheControl.maxAge(Duration.ofMinutes(5))
                        .mustRevalidate())
                .body(produto);
    }

    // Dados que raramente mudam — cache agressivo
    @GetMapping("/categorias")
    public ResponseEntity<List<CategoriaDTO>> listarCategorias() {
        List<CategoriaDTO> categorias = produtoService.listarCategorias();

        return ResponseEntity.ok()
                .cacheControl(CacheControl.maxAge(Duration.ofHours(24))
                        .cachePublic()) // CDN pode cachear
                .body(categorias);
    }
}
```

**ETags com ShallowEtagHeaderFilter:**

```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean<ShallowEtagHeaderFilter> etagFilter() {
        FilterRegistrationBean<ShallowEtagHeaderFilter> registration =
                new FilterRegistrationBean<>();
        registration.setFilter(new ShallowEtagHeaderFilter());
        registration.addUrlPatterns("/api/*");
        registration.setName("etagFilter");
        registration.setOrder(1);
        return registration;
    }
}
```

O `ShallowEtagHeaderFilter` calcula um hash MD5 do body da resposta e adiciona o header `ETag`. Se o cliente enviar `If-None-Match` com o mesmo ETag, o servidor retorna `304 Not Modified` sem body — economizando banda.

### 4.7 Estrategias de Invalidação

| Estratégia      | Como Funciona                                        | Consistência | Complexidade | Uso Ideal                    |
|-----------------|------------------------------------------------------|--------------|--------------|------------------------------|
| Cache-Aside     | App lê do cache; se miss, lê do DB e popula cache    | Eventual     | Baixa        | Maioria dos cenários         |
| Read-Through    | Cache lê do DB automaticamente quando há miss        | Eventual     | Média        | Quando cache gerência leitura|
| Write-Through   | App escreve no cache E no DB simultaneamente         | Forte        | Média        | Dados críticos               |
| Write-Behind    | App escreve no cache; cache escreve no DB em batch   | Eventual     | Alta         | Alta vazao de escrita        |
| TTL (Time-Based)| Dados expiram automaticamente após um tempo          | Eventual     | Baixa        | Dados que toleram stale      |

**Cache-Aside (padrão mais comum no Spring):**

```java
// Cache-Aside é exatamente o que @Cacheable faz:
@Cacheable("produtos")
public ProdutoDTO buscar(Long id) {
    // So executa se não estiver no cache
    return repository.findById(id).map(ProdutoDTO::fromEntity).orElse(null);
}

@CacheEvict(value = "produtos", key = "#id")
public void atualizar(Long id, ProdutoUpdateRequest req) {
    // Atualiza DB e remove do cache
    // Próxima leitura buscara do DB e populara o cache novamente
}
```

> Para detalhes sobre configurações avancadas do Spring Boot, consulte [Spring-Boot-Avancado.md](Spring-Boot-Avancado.md).

---

## 5. Otimização de Serialização JSON (Jackson)

A serialização/desserialização JSON é executada em **cada request e response**. Em APIs com alto throughput, o Jackson pode se tornar um gargalo mensurável.

### 5.1 Jackson e Performance — @JsonView

`@JsonView` permite retornar diferentes "visoes" de um mesmo DTO, evitando criar múltiplas classes:

```java
// Definicao de views
public class Views {
    public static class Resumo {}
    public static class Completo extends Resumo {}
    public static class Admin extends Completo {}
}

// DTO com views
public class ProdutoDTO {

    @JsonView(Views.Resumo.class)
    private Long id;

    @JsonView(Views.Resumo.class)
    private String nome;

    @JsonView(Views.Resumo.class)
    private BigDecimal preco;

    @JsonView(Views.Completo.class)
    private String descricao;

    @JsonView(Views.Completo.class)
    private List<String> imagens;

    @JsonView(Views.Admin.class)
    private BigDecimal custoInterno;

    @JsonView(Views.Admin.class)
    private LocalDateTime dataCriacao;
}

@RestController
@RequestMapping("/api/produtos")
public class ProdutoController {

    // Retorna apenas id, nome, preco (payload menor)
    @GetMapping
    @JsonView(Views.Resumo.class)
    public List<ProdutoDTO> listar() {
        return produtoService.listarTodos();
    }

    // Retorna tudo exceto campos admin
    @GetMapping("/{id}")
    @JsonView(Views.Completo.class)
    public ProdutoDTO detalhe(@PathVariable Long id) {
        return produtoService.buscarPorId(id);
    }

    // Retorna todos os campos
    @GetMapping("/{id}/admin")
    @JsonView(Views.Admin.class)
    @PreAuthorize("hasRole('ADMIN')")
    public ProdutoDTO detalheAdmin(@PathVariable Long id) {
        return produtoService.buscarPorId(id);
    }
}
```

### 5.2 Evitar Serialização Desnecessaria

**@JsonIgnore e @JsonIgnoreProperties:**

```java
@Entity
public class Usuario {

    private Long id;
    private String nome;
    private String email;

    @JsonIgnore // Nunca serializar senha!
    private String senhaHash;

    @JsonIgnore // Evita referência circular e dados internos
    @OneToMany(mappedBy = "usuario")
    private List<Auditoria> logs;
}

// Ou no DTO (preferido — não acopla Entity ao Jackson)
@JsonIgnoreProperties({"senhaHash", "logs"})
public record UsuarioResponse(
        Long id,
        String nome,
        String email
) {
    public static UsuarioResponse fromEntity(Usuario u) {
        return new UsuarioResponse(u.getId(), u.getNome(), u.getEmail());
    }
}
```

**Padrão DTO (recomendado para performance e segurança):**

```java
// Em vez de retornar a entidade (que pode ter dezenas de campos e lazy-loads),
// retorne um DTO enxuto:
public record PedidoListagemDTO(
        Long id,
        String numero,
        LocalDate data,
        String statusDisplay,
        BigDecimal valorTotal
) {
    public static PedidoListagemDTO fromEntity(Pedido p) {
        return new PedidoListagemDTO(
                p.getId(),
                p.getNumero(),
                p.getData(),
                p.getStatus().getDescricao(),
                p.getValorTotal()
        );
    }
}
```

### 5.3 Streaming API para Payloads Grandes

Para respostas com milhares de itens, gerar o JSON inteiro em memória pode causar `OutOfMemoryError`. A Streaming API do Jackson escreve diretamente no output stream:

```java
@RestController
@RequiredArgsConstructor
public class ExportController {

    private final ProdutoRepository produtoRepository;
    private final ObjectMapper objectMapper;

    @GetMapping(value = "/api/export/produtos",
                produces = MediaType.APPLICATION_JSON_VALUE)
    public void exportarProdutos(HttpServletResponse response) throws IOException {
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);

        try (var generator = objectMapper.getFactory()
                .createGenerator(response.getOutputStream())) {

            generator.writeStartArray();

            // Stream do banco — processa um registro por vez, sem carregar tudo em memória
            produtoRepository.findAllAsStream().forEach(produto -> {
                try {
                    generator.writeObject(ProdutoDTO.fromEntity(produto));
                    generator.flush(); // Envia ao cliente progressivamente
                } catch (IOException e) {
                    throw new UncheckedIOException(e);
                }
            });

            generator.writeEndArray();
        }
    }
}

// No repositorio:
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    @Query("SELECT p FROM Produto p")
    @QueryHints(@QueryHint(name = HINT_FETCH_SIZE, value = "100"))
    Stream<Produto> findAllAsStream();
}
```

> **Importante:** use `Stream` dentro de uma `@Transactional(readOnly = true)` e feche o stream após o uso.

### 5.4 Configurações de Performance do ObjectMapper

**Módulo Blackbird (substituto do Afterburner para Java 11+):**

O módulo Blackbird usa `java.lang.invoke.MethodHandle` para acelerar a serialização/desserialização em até 30%:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-blackbird</artifactId>
</dependency>
```

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jacksonCustomizer() {
        return builder -> builder
                .modulesToInstall(new BlackbirdModule())
                .featuresToDisable(
                        // Desabilitar features não usadas para ganhar performance
                        SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,
                        DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES
                )
                .featuresToEnable(
                        // Acelerar parsing de strings
                        JsonParser.Feature.USE_FAST_DOUBLE_PARSER,
                        JsonParser.Feature.USE_FAST_BIG_NUMBER_PARSER
                );
    }
}
```

> Para detalhes sobre configurações Jackson, anotações e customizações, consulte [Dicas-Jackson.md](Dicas-Jackson.md).

---

## 6. HTTP Client — RestClient, WebClient e Connection Pooling

Chamadas HTTP para serviços externos são frequentemente o maior gargalo em arquiteturas de microservicos. A escolha e configuração do client HTTP impacta diretamente a performance.

### 6.1 RestClient vs WebClient vs RestTemplate

| Aspecto                | RestTemplate (legado)       | RestClient (Spring 6.1+)    | WebClient (reativo)         |
|------------------------|-----------------------------|-----------------------------|-----------------------------|
| API                    | Sincrona, verbosa           | Sincrona, fluent API        | Assincrona/reativa          |
| Status                 | Em manutenção               | Recomendado (bloqueante)    | Recomendado (não-bloqueante)|
| Connection pooling     | Configuravel                | Configuravel                | Netty (não-bloqueante)      |
| Streaming              | Limitado                    | Sim                         | Sim (Flux)                  |
| Requer WebFlux         | Não                         | Não                         | Sim                         |
| Uso com Virtual Threads| Bom                         | Excelente                   | Desnecessario               |

**Com Virtual Threads, o `RestClient` síncrono se torna a escolha natural** — você obtém concorrência sem a complexidade da programação reativa.

### 6.2 Connection Pooling com Apache HttpClient 5

Por padrão, o `RestClient` cria uma nova conexão HTTP por request. Para alto throughput, configure um connection pool:

```java
@Configuration
public class HttpClientConfig {

    @Bean
    public RestClient restClient(RestClient.Builder builder) {
        // Connection pool com Apache HttpClient 5
        var connectionManager = PoolingHttpClientConnectionManagerBuilder.create()
                .setMaxConnTotal(100)            // Total de conexões no pool
                .setMaxConnPerRoute(20)          // Max conexões por host
                .setDefaultConnectionConfig(
                        ConnectionConfig.custom()
                                .setConnectTimeout(Timeout.ofSeconds(5))
                                .setSocketTimeout(Timeout.ofSeconds(30))
                                .build())
                .build();

        var httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .setDefaultRequestConfig(
                        RequestConfig.custom()
                                .setConnectionRequestTimeout(Timeout.ofSeconds(5))
                                .setResponseTimeout(Timeout.ofSeconds(30))
                                .build())
                .evictExpiredConnections()        // Remove conexões expiradas
                .evictIdleConnections(
                        TimeValue.ofMinutes(5))  // Remove conexões ociosas
                .build();

        var requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);

        return builder
                .requestFactory(requestFactory)
                .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
                .build();
    }
}
```

```xml
<!-- Dependencia necessária -->
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
</dependency>
```

### 6.3 Timeouts — Conectividade e Leitura

**Sempre configure timeouts!** Sem timeout, uma chamada a um serviço lento pode travar sua thread indefinidamente.

```java
@Service
@RequiredArgsConstructor
public class CepService {

    private final RestClient restClient;

    public EnderecoDTO buscarCep(String cep) {
        return restClient.get()
                .uri("https://viacep.com.br/ws/{cep}/json/", cep)
                .retrieve()
                .onStatus(HttpStatusCode::is4xxClientError, (req, resp) -> {
                    throw new CepInvalidoException(cep);
                })
                .body(EnderecoDTO.class);
    }
}
```

**Timeouts por request (quando necessário sobrescrever os defaults):**

```java
@Bean
public RestClient restClientComTimeoutCustom() {
    var requestFactory = new HttpComponentsClientHttpRequestFactory();
    requestFactory.setConnectTimeout(Duration.ofSeconds(3));
    requestFactory.setReadTimeout(Duration.ofSeconds(10));

    return RestClient.builder()
            .requestFactory(requestFactory)
            .baseUrl("https://api.externa.com")
            .build();
}
```

### 6.4 Retry e Circuit Breaker

Para resiliência em chamadas externas, combine retry com circuit breaker. Tratamos brevemente aqui — o assunto completo esta no documento referenciado.

```java
// Exemplo básico com Spring Retry
@Service
public class PagamentoExternoService {

    private final RestClient restClient;

    @Retryable(
            retryFor = {ResourceAccessException.class, HttpServerErrorException.class},
            maxAttempts = 3,
            backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public PagamentoResponse processar(PagamentoRequest request) {
        return restClient.post()
                .uri("/pagamentos")
                .body(request)
                .retrieve()
                .body(PagamentoResponse.class);
    }

    @Recover
    public PagamentoResponse fallback(Exception ex, PagamentoRequest request) {
        log.error("Falha após retries para pagamento: {}", request.id(), ex);
        throw new PagamentoIndisponivelException(ex);
    }
}
```

> Para detalhes sobre Circuit Breaker (Resilience4j), retry avancado e patterns de resiliência, consulte [Spring-Boot-Avancado.md](Spring-Boot-Avancado.md).

---

## 7. Processamento Assíncrono e Paralelo

Operações demoradas que não precisam bloquear a resposta HTTP devem ser executadas de forma assíncrona: envio de emails, geração de relatórios, notificações push, etc.

### 7.1 @Async com ThreadPoolTaskExecutor

**Configuração do pool de threads:**

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);         // Threads mantidas ativas
        executor.setMaxPoolSize(20);         // Max threads sob carga
        executor.setQueueCapacity(100);      // Fila antes de criar mais threads
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(
                new ThreadPoolExecutor.CallerRunsPolicy()); // Se fila cheia, executa na thread chamadora
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        executor.initialize();
        return executor;
    }
}
```

**Usando @Async:**

```java
@Service
@RequiredArgsConstructor
public class NotificacaoService {

    private final EmailSender emailSender;
    private final PushNotificationClient pushClient;

    // Executa em thread separada — não bloqueia o caller
    @Async("taskExecutor")
    public void enviarNotificacaoPedido(Pedido pedido) {
        emailSender.enviar(
                pedido.getCliente().getEmail(),
                "Pedido confirmado",
                "Seu pedido #" + pedido.getNumero() + " foi confirmado!"
        );

        pushClient.enviar(
                pedido.getCliente().getDeviceToken(),
                "Pedido " + pedido.getNumero() + " confirmado"
        );
    }

    // @Async com retorno — retorna CompletableFuture
    @Async("taskExecutor")
    public CompletableFuture<RelatorioDTO> gerarRelatorioAsync(
            LocalDate inicio, LocalDate fim) {
        // Processamento demorado...
        RelatorioDTO relatorio = processarRelatorio(inicio, fim);
        return CompletableFuture.completedFuture(relatorio);
    }
}
```

```java
// No controller — fire-and-forget
@PostMapping("/pedidos")
public ResponseEntity<PedidoDTO> criarPedido(@RequestBody PedidoRequest request) {
    Pedido pedido = pedidoService.criar(request);

    // Envio de notificação em background — resposta retorna imediatamente
    notificacaoService.enviarNotificacaoPedido(pedido);

    return ResponseEntity.status(HttpStatus.CREATED)
            .body(PedidoDTO.fromEntity(pedido));
}
```

> **Atenção:** `@Async` só funciona quando chamado de **fora** da classe. Chamadas internas (self-invocation) ignoram o proxy e executam de forma síncrona.

### 7.2 CompletableFuture — Composição de Tarefas

Para executar múltiplas chamadas em paralelo e combinar os resultados:

```java
@Service
@RequiredArgsConstructor
public class DashboardService {

    private final PedidoService pedidoService;
    private final ClienteService clienteService;
    private final ProdutoService produtoService;
    private final Executor taskExecutor;

    public DashboardDTO montarDashboard() {
        // Executa 3 consultas em paralelo
        CompletableFuture<Long> totalPedidos = CompletableFuture
                .supplyAsync(pedidoService::contarPedidosMes, taskExecutor);

        CompletableFuture<Long> totalClientes = CompletableFuture
                .supplyAsync(clienteService::contarClientesAtivos, taskExecutor);

        CompletableFuture<List<ProdutoDTO>> maisVendidos = CompletableFuture
                .supplyAsync(() -> produtoService.listarMaisVendidos(10), taskExecutor);

        // Aguarda todos completarem (bloqueante, ok com Virtual Threads)
        CompletableFuture.allOf(totalPedidos, totalClientes, maisVendidos).join();

        return new DashboardDTO(
                totalPedidos.join(),
                totalClientes.join(),
                maisVendidos.join()
        );
    }

    // Exemplo com timeout e fallback
    public DashboardDTO montarDashboardComFallback() {
        CompletableFuture<Long> totalPedidos = CompletableFuture
                .supplyAsync(pedidoService::contarPedidosMes, taskExecutor)
                .orTimeout(5, TimeUnit.SECONDS)
                .exceptionally(ex -> {
                    log.warn("Falha ao contar pedidos: {}", ex.getMessage());
                    return -1L; // Valor de fallback
                });

        CompletableFuture<Long> totalClientes = CompletableFuture
                .supplyAsync(clienteService::contarClientesAtivos, taskExecutor)
                .orTimeout(5, TimeUnit.SECONDS)
                .exceptionally(ex -> -1L);

        CompletableFuture.allOf(totalPedidos, totalClientes).join();

        return new DashboardDTO(
                totalPedidos.join(),
                totalClientes.join(),
                List.of()
        );
    }
}
```

### 7.3 @Scheduled com Pool Dedicado

O `@Scheduled` usa **uma única thread** por padrão. Se uma tarefa travar, todas as outras param.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        // Pool dedicado para tarefas agendadas
        taskRegistrar.setScheduler(scheduledExecutor());
    }

    @Bean(destroyMethod = "shutdown")
    public ScheduledExecutorService scheduledExecutor() {
        return Executors.newScheduledThreadPool(5,
                new CustomizableThreadFactory("scheduled-"));
    }
}

@Service
@RequiredArgsConstructor
public class TarefasAgendadas {

    private final CacheManager cacheManager;
    private final RelatorioService relatorioService;

    // Limpa cache expirado a cada hora
    @Scheduled(cron = "0 0 * * * *")
    public void limparCache() {
        cacheManager.getCacheNames()
                .forEach(name -> {
                    Cache cache = cacheManager.getCache(name);
                    if (cache != null) cache.clear();
                });
    }

    // Gera relatório diario as 6h
    @Scheduled(cron = "0 0 6 * * *")
    public void gerarRelatorioDiario() {
        relatorioService.gerarEEnviarPorEmail(LocalDate.now().minusDays(1));
    }

    // Health check a cada 30 segundos
    @Scheduled(fixedRate = 30_000)
    public void verificarServicosExternos() {
        // verificação...
    }
}
```

### 7.4 Spring Events Assincronos

Events desacoplam o produtor do consumidor. Combinados com `@Async`, o consumidor executa em thread separada:

```java
// Evento
public record PedidoConfirmadoEvent(
        Long pedidoId,
        String clienteEmail,
        BigDecimal valorTotal,
        LocalDateTime timestamp
) {}

// Produtor — publica o evento
@Service
@RequiredArgsConstructor
public class PedidoService {

    private final ApplicationEventPublisher eventPublisher;
    private final PedidoRepository pedidoRepository;

    @Transactional
    public Pedido confirmar(Long pedidoId) {
        Pedido pedido = pedidoRepository.findById(pedidoId)
                .orElseThrow();
        pedido.setStatus(StatusPedido.CONFIRMADO);
        pedidoRepository.save(pedido);

        // Publica evento — listeners assincronos processam em background
        eventPublisher.publishEvent(new PedidoConfirmadoEvent(
                pedido.getId(),
                pedido.getCliente().getEmail(),
                pedido.getValorTotal(),
                LocalDateTime.now()
        ));

        return pedido;
    }
}

// Consumidor 1 — envia email
@Component
@RequiredArgsConstructor
public class EmailPedidoListener {

    private final EmailSender emailSender;

    @Async("taskExecutor")
    @EventListener
    public void onPedidoConfirmado(PedidoConfirmadoEvent event) {
        emailSender.enviar(event.clienteEmail(),
                "Pedido Confirmado",
                "Valor: R$ " + event.valorTotal());
    }
}

// Consumidor 2 — atualiza estoque
@Component
@RequiredArgsConstructor
public class EstoqueListener {

    private final EstoqueService estoqueService;

    @Async("taskExecutor")
    @EventListener
    public void onPedidoConfirmado(PedidoConfirmadoEvent event) {
        estoqueService.reservar(event.pedidoId());
    }
}

// Consumidor 3 — gera nota fiscal (transacional — executa após commit)
@Component
@RequiredArgsConstructor
public class NotaFiscalListener {

    private final NotaFiscalService notaFiscalService;

    @Async("taskExecutor")
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onPedidoConfirmado(PedidoConfirmadoEvent event) {
        notaFiscalService.gerar(event.pedidoId());
    }
}
```

> Para detalhes sobre processamento assíncrono e CompletableFuture, consulte [Spring-Boot-Async-CompletableFuture.md](outros/Spring-Boot-Async-CompletableFuture.md).

---

## 8. Virtual Threads e Structured Concurrency

Virtual Threads (Project Loom) são uma das maiores mudanças na plataforma Java desde generics. Permitem escalabilidade massiva sem programação reativa.

### 8.1 Virtual Threads no Spring Boot 3.2+

**Habilitando Virtual Threads:**

```yaml
# application.yml — uma única propriedade!
spring:
  threads:
    virtual:
      enabled: true
```

Com esta propriedade, o Spring Boot configura automaticamente:
- Tomcat para usar Virtual Threads (uma virtual thread por request)
- `@Async` para usar Virtual Threads
- `@Scheduled` para usar Virtual Threads
- Spring MVC para processar requests em Virtual Threads

**O que muda na prática:**

```
  Threads de Plataforma (antes):
  - Tomcat: 200 threads (padrão)
  - Cada thread: ~1MB de stack
  - 200 requests simultâneos MAX
  - Request bloqueado em I/O = thread desperdiçada

  Virtual Threads (depois):
  - Milhões de virtual threads possíveis
  - Cada virtual thread: ~1KB
  - Request bloqueado em I/O = virtual thread "estaciona", plataforma thread fica livre
  - Throughput limitado pelo recurso externo (banco, rede), não pelas threads
```

**Quando Virtual Threads ajudam:**
- Aplicações I/O-bound (maioria das apps Spring Boot)
- Muitas chamadas a banco de dados
- Muitas chamadas HTTP para outros serviços
- Alto número de requests simultâneos

**Quando Virtual Threads NÃO ajudam:**
- Processamento CPU-bound (cálculo pesado, compressão)
- Aplicações que já usam programação reativa (WebFlux)
- Aplicações com poucas conexões simultâneas

### 8.2 Impacto nas Aplicações Spring Boot

Com Virtual Threads, você pode escrever código **síncrono e bloqueante** com performance equivalente ao código reativo:

```java
// Codigo sincrono que ESCALA com Virtual Threads
// Nenhuma mudança necessária no código existente!
@Service
@RequiredArgsConstructor
public class PedidoService {

    private final PedidoRepository pedidoRepository;     // Bloqueia em I/O
    private final RestClient clienteClient;              // Bloqueia em I/O
    private final RestClient estoqueClient;              // Bloqueia em I/O

    public PedidoDetalheDTO buscarDetalhes(Long id) {
        Pedido pedido = pedidoRepository.findById(id)    // DB call (bloqueante)
                .orElseThrow();

        ClienteDTO cliente = clienteClient.get()         // HTTP call (bloqueante)
                .uri("/clientes/{id}", pedido.getClienteId())
                .retrieve()
                .body(ClienteDTO.class);

        EstoqueDTO estoque = estoqueClient.get()         // HTTP call (bloqueante)
                .uri("/estoque/pedido/{id}", id)
                .retrieve()
                .body(EstoqueDTO.class);

        return new PedidoDetalheDTO(pedido, cliente, estoque);
    }
}
```

> **Sem** Virtual Threads: cada chamada bloqueante trava uma thread de plataforma (custo ~1MB cada).
> **Com** Virtual Threads: cada chamada bloqueante "estaciona" a virtual thread (custo ~1KB), liberando a thread de plataforma para atender outros requests.

### 8.3 Pinning — Detecção e Correção

**Pinning** ocorre quando uma virtual thread executa código `synchronized`, impedindo que a thread de plataforma subjacente seja reutilizada. Isso anula os benefícios das virtual threads.

**Detectando pinning:**

```bash
# JVM flag para detectar pinning (Java 21+)
java -Djdk.tracePinnedThreads=short -jar app.jar

# Ou com mais detalhes
java -Djdk.tracePinnedThreads=full -jar app.jar
```

Saida de exemplo quando há pinning:
```
Thread[#42,ForkJoinPool-1-worker-3,5,CarrierThreads]
    java.base/java.lang.VirtualThread$VStackContinuation.onPinned(VirtualThread.java:180)
    com.example.service.LegacyService.processarSynchronized(LegacyService.java:25)
    <== monitors:1
```

**Corrigindo pinning — trocar `synchronized` por `ReentrantLock`:**

```java
// ANTES: causa pinning em Virtual Threads
public class ContadorLegado {

    private int count = 0;

    public synchronized void incrementar() { // <-- synchronized causa pinning
        count++;
        salvarNoBanco(count); // I/O dentro de bloco synchronized = muito ruim!
    }

    public synchronized int getCount() {
        return count;
    }
}

// DEPOIS: compatível com Virtual Threads
public class ContadorModerno {

    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void incrementar() {
        lock.lock();
        try {
            count++;
            salvarNoBanco(count); // I/O com ReentrantLock = sem pinning
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

> **Nota:** `synchronized` em blocos curtos sem I/O (ex: acesso a variável) causa pinning mínimo e geralmente não é problema. O caso crítico e `synchronized` + operação de I/O.

### 8.4 Structured Concurrency

Structured Concurrency (preview no Java 21+) organiza tarefas concorrentes em escopos bem definidos, garantindo que todas as subtarefas completem (ou sejam canceladas) antes que o escopo se encerre.

```java
// Structured Concurrency — tarefas paralelas com tratamento de erro coordenado
@Service
public class DashboardService {

    private final PedidoService pedidoService;
    private final ClienteService clienteService;
    private final EstoqueService estoqueService;

    public DashboardDTO montarDashboard() throws Exception {
        // ShutdownOnFailure: se qualquer tarefa falhar, cancela as demais
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {

            // Lanca 3 tarefas em paralelo (cada uma em sua virtual thread)
            Subtask<Long> pedidos = scope.fork(pedidoService::contarPedidosMes);
            Subtask<Long> clientes = scope.fork(clienteService::contarAtivos);
            Subtask<List<ProdutoDTO>> produtos = scope.fork(
                    () -> estoqueService.listarBaixoEstoque(10));

            // Aguarda todas completarem
            scope.join();

            // Propaga exceção se alguma falhou
            scope.throwIfFailed();

            return new DashboardDTO(
                    pedidos.get(),
                    clientes.get(),
                    produtos.get()
            );
        }
    }

    // ShutdownOnSuccess: retorna o PRIMEIRO resultado bem-sucedido
    public CotacaoDTO buscarMelhorCotacao(String produto) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<CotacaoDTO>()) {

            // Consulta 3 fornecedores em paralelo — usa a primeira resposta
            scope.fork(() -> fornecedorA.cotar(produto));
            scope.fork(() -> fornecedorB.cotar(produto));
            scope.fork(() -> fornecedorC.cotar(produto));

            scope.join();

            return scope.result(); // Retorna o primeiro resultado
        }
    }
}
```

### 8.5 Scoped Values vs ThreadLocal

`ThreadLocal` é problemático com Virtual Threads: cada virtual thread mantém sua própria cópia, e com milhões de virtual threads o consumo de memória explode. Além disso, `ThreadLocal` é mutável e propenso a memory leaks.

```java
// PROBLEMA: ThreadLocal com Virtual Threads
public class ContextoRequisicao {

    // Com 1 milhao de virtual threads, cria 1 milhao de cópias!
    private static final ThreadLocal<String> tenantId = new ThreadLocal<>();

    public static void setTenantId(String id) {
        tenantId.set(id); // Mutavel — difícil de rastrear quem altera
    }

    public static String getTenantId() {
        return tenantId.get();
    }

    // Esqueceu de chamar remove()? Memory leak!
    public static void clear() {
        tenantId.remove();
    }
}
```

**ScopedValue (preview no Java 21+) — alternativa imutável e eficiente:**

```java
// ScopedValue: imutável, com escopo bem definido, eficiente com Virtual Threads
public class ContextoRequisicao {

    public static final ScopedValue<String> TENANT_ID = ScopedValue.newInstance();
    public static final ScopedValue<UserInfo> USUARIO = ScopedValue.newInstance();

    // Uso:
    public static void processarRequisicao(String tenant, UserInfo user) {
        ScopedValue.runWhere(TENANT_ID, tenant, () ->
            ScopedValue.runWhere(USUARIO, user, () -> {
                // Dentro deste bloco, TENANT_ID e USUARIO estao disponiveis
                executarLogicaDeNegocio();
                // Ao sair do bloco, os valores são automaticamente limpos
            })
        );
    }
}

// Acessando o valor em qualquer camada dentro do escopo
@Repository
public class PedidoRepository {

    public List<Pedido> findAll() {
        String tenant = ContextoRequisicao.TENANT_ID.get();
        // Usa o tenant para filtrar queries...
        return jdbcTemplate.query(
                "SELECT * FROM pedido WHERE tenant_id = ?",
                mapper, tenant
        );
    }
}
```

**Comparação ThreadLocal vs ScopedValue:**

| Aspecto                  | ThreadLocal            | ScopedValue (preview)       |
|--------------------------|------------------------|-----------------------------|
| Mutabilidade             | Mutavel (set/get)      | Imutavel (bind once)        |
| Escopo                   | Toda a vida da thread  | Bloco de código definido    |
| Memory leak              | Possivel (sem remove)  | Impossivel (auto-cleanup)   |
| Performance c/ V.Threads | Ruim (1 cópia/thread)  | Excelente (compartilhado)   |
| Herança em subtarefas    | InheritableThreadLocal | Automático c/ Structured C. |

> Para detalhes sobre threads, concorrência e programação paralela em Java, consulte [Java-Avancado-Threads-Concorrência.md](outros/Java-Avancado-Threads-Concorrência.md).

---

## 9. Otimização de Collections e Algoritmos

A escolha da estrutura de dados correta pode transformar um algoritmo de O(n) para O(1). Embora o impacto seja menor que otimizações de I/O, em loops frequentes ou coleções grandes a diferença e significativa.

### 9.1 Escolha da Estrutura de Dados

| Estrutura         | get/contains | add (fim) | add (meio) | remove    | Ordenada | Obs                           |
|-------------------|-------------|-----------|------------|-----------|----------|-------------------------------|
| `ArrayList`       | O(1) índice | O(1)*     | O(n)       | O(n)      | Inserção | * amortizado                  |
| `LinkedList`      | O(n)        | O(1)      | O(1)**     | O(1)**    | Inserção | ** se tiver o iterator        |
| `HashSet`         | O(1)        | O(1)      | —          | O(1)      | Não      | Sem duplicatas                |
| `LinkedHashSet`   | O(1)        | O(1)      | —          | O(1)      | Inserção | HashSet + ordem               |
| `TreeSet`         | O(log n)    | O(log n)  | —          | O(log n)  | Natural  | Ordenado (Red-Black Tree)     |
| `HashMap`         | O(1)        | O(1)      | —          | O(1)      | Não      | Chave-valor                   |
| `LinkedHashMap`   | O(1)        | O(1)      | —          | O(1)      | Inserção | HashMap + ordem               |
| `TreeMap`         | O(log n)    | O(log n)  | —          | O(log n)  | Chave    | Ordenado por chave            |
| `EnumSet`         | O(1)        | O(1)      | —          | O(1)      | Enum     | Bit vector — extremamente rápido |
| `EnumMap`          | O(1)        | O(1)      | —          | O(1)      | Enum     | Array interno — rápido e compacto |

**EnumSet e EnumMap — coleções especializadas e ultrarapidas:**

```java
// EnumSet: implementado como bit vector — O(1) para tudo, quase zero de memória
public enum Permissao {
    LEITURA, ESCRITA, EXCLUSAO, ADMIN, EXPORTACAO, IMPORTACAO
}

// Em vez de Set<Permissão> = new HashSet<>()
Set<Permissao> permissoes = EnumSet.of(Permissao.LEITURA, Permissao.ESCRITA);
permissoes.add(Permissao.EXPORTACAO);
boolean temAdmin = permissoes.contains(Permissao.ADMIN); // O(1), internamente: bit & mask

// EnumMap: array indexado pelo ordinal do enum — mais rápido que HashMap
Map<Permissao, String> descricoes = new EnumMap<>(Permissao.class);
descricoes.put(Permissao.LEITURA, "Pode ler dados");
descricoes.put(Permissao.ESCRITA, "Pode criar e editar dados");
```

### 9.2 Pre-alocação de Capacidade

Quando você sabe (ou estima) o tamanho final da coleção, pré-alocar evita realocações internas:

```java
// RUIM: ArrayList começa com capacidade 10 e faz resize multiplas vezes
List<String> nomes = new ArrayList<>(); // capacidade inicial: 10
for (int i = 0; i < 10_000; i++) {
    nomes.add("Nome " + i); // Resize em 10, 15, 22, 33, 49... (fator 1.5x)
}

// BOM: pré-aloca capacidade exata
List<String> nomes = new ArrayList<>(10_000); // Nenhum resize necessário

// HashMap: considere o load factor (0.75 padrão)
// Para 1000 elementos sem rehash: capacidade = 1000 / 0.75 = 1334
Map<Long, ProdutoDTO> mapa = new HashMap<>(1334);

// Ou use a factory method que calcula automaticamente (Java 19+)
Map<Long, ProdutoDTO> mapa = HashMap.newHashMap(1000);
```

**Impacto mensurável:**

```java
// Benchmark com JMH
@Benchmark
public List<Integer> semPreAlocacao() {
    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < 100_000; i++) {
        list.add(i);
    }
    return list;
}

@Benchmark
public List<Integer> comPreAlocacao() {
    List<Integer> list = new ArrayList<>(100_000);
    for (int i = 0; i < 100_000; i++) {
        list.add(i);
    }
    return list;
}

// Resultado típico:
// semPreAlocacao:  ~1.8 ms/op
// comPreAlocacao:  ~0.9 ms/op  (2x mais rápido)
```

### 9.3 Streams — Considerações de Performance

**Parallel Streams — quando usar é quando evitar:**

```java
// USAR parallel stream: coleções GRANDES + operações CPU-bound
List<BigDecimal> precos = produtos.stream()       // 1 milhao de itens
        .parallel()                                // Divide entre CPU cores
        .map(p -> calcularDescontoComplexo(p))     // CPU-bound
        .toList();

// NÃO USAR parallel stream:
// 1. Colecoes pequenas (overhead de fork/join > ganho)
List<String> nomes = clientes.stream()             // 50 itens
        .parallel()                                // PIOR que sequencial!
        .map(Cliente::getNome)
        .toList();

// 2. Operacoes I/O (todas as threads do ForkJoinPool ficam bloqueadas)
List<String> resultados = urls.stream()
        .parallel()                                // PERIGO: bloqueia pool comum
        .map(this::chamarApi)                      // I/O bloqueante
        .toList();
```

**Streams primitivos — evitando autoboxing:**

```java
// RUIM: boxing/unboxing em cada operação
List<Integer> numeros = List.of(1, 2, 3, 4, 5);
int soma = numeros.stream()
        .mapToInt(Integer::intValue) // Unboxing
        .sum();

// BOM: use IntStream, LongStream, DoubleStream diretamente
int soma = IntStream.rangeClosed(1, 1_000_000)
        .filter(n -> n % 2 == 0)
        .sum(); // Sem boxing — operações em primitivos

// Para listas existentes
double media = pedidos.stream()
        .mapToDouble(p -> p.getValorTotal().doubleValue()) // Converte uma vez
        .average()
        .orElse(0.0);
```

**Collectors otimizados:**

```java
// toList() (Java 16+) é mais eficiente que Collectors.toList()
// Retorna lista imutável, sem overhead de ArrayList
var nomes = clientes.stream()
        .map(Cliente::getNome)
        .toList(); // Preferir sobre Collectors.toList()

// Para mapas, use toUnmodifiableMap quando não precisar de mutabilidade
var clientesPorId = clientes.stream()
        .collect(Collectors.toUnmodifiableMap(
                Cliente::getId,
                Function.identity()
        ));
```

### 9.4 String Handling

Strings são imutáveis em Java. Concatenação repetida cria objetos intermediarios descartaveis.

```java
// RUIM: cada += cria um novo objeto String
String resultado = "";
for (String item : itens) {   // 10.000 itens
    resultado += item + ", ";  // Cria ~20.000 objetos temporarios!
}

// BOM: StringBuilder (mutavel, pré-alocavel)
StringBuilder sb = new StringBuilder(itens.size() * 20); // Estimar tamanho
for (String item : itens) {
    if (!sb.isEmpty()) sb.append(", ");
    sb.append(item);
}
String resultado = sb.toString(); // Uma única alocação final

// MELHOR: String.join ou Collectors.joining
String resultado = String.join(", ", itens);

// Com stream + transformação
String resultado = pedidos.stream()
        .map(Pedido::getNumero)
        .collect(Collectors.joining(", ", "[", "]"));
// Resultado: [PED001, PED002, PED003]
```

**String.intern() — cuidado com uso indiscriminado:**

```java
// intern() armazena a string no String Pool (PermGen/Metaspace)
// Util APENAS quando há muitas strings identicas de fontes externas
String status = resultSet.getString("status").intern();

// PERIGO: não use intern() com strings dinâmicas/unicas
// Isso polui o String Pool e causa overhead no GC
String naoFacaIsso = ("usuario-" + id).intern(); // Ruim!
```

---

## 10. Lazy Initialization e Startup Otimizado

O tempo de startup é crítico em ambientes com autoscaling (Kubernetes), serverless, e pipelines de CI/CD. Reduzir o tempo de inicialização melhora a experiência de desenvolvimento é a resiliência em produção.

### 10.1 Spring Boot Lazy Initialization

**Habilitando lazy initialization global:**

```yaml
# application.yml
spring:
  main:
    lazy-initialization: true
```

Com esta propriedade, os beans **só são criados quando são usados pela primeira vez**, em vez de todos na inicialização.

**Pros e contras:**

| Aspecto                | Lazy ON                          | Lazy OFF (padrão)             |
|------------------------|----------------------------------|-------------------------------|
| Tempo de startup       | Muito mais rápido                | Mais lento                    |
| Primeiro request       | Mais lento (cria beans sob demanda)| Normal                      |
| Detecção de erros      | Erro só aparece no primeiro uso  | Falha rápida no startup       |
| Uso de memória         | Menor (beans não usados não existem)| Maior                      |
| Recomendado para       | Desenvolvimento, testes          | Produção                      |

**@Lazy seletivo (recomendado para produção):**

```java
// Apenas beans especificos são lazy
@Service
public class RelatorioService {

    // RelatorioGerador e pesado para criar — inicializa apenas quando necessário
    @Lazy
    @Autowired
    private RelatorioGerador relatorioGerador;

    public void gerarSeNecessario(boolean gerar) {
        if (gerar) {
            relatorioGerador.gerar(); // Bean criado aqui, na primeira chamada
        }
    }
}

// Ou com @Lazy na classe
@Service
@Lazy
public class RelatorioGerador {
    // So e instanciado quando algum outro bean o requisitar
}
```

**Combinando: lazy global + eager seletivo:**

```yaml
spring:
  main:
    lazy-initialization: true  # Tudo lazy por padrão
```

```java
// Beans criticos que devem inicializar no startup (mesmo com lazy global)
@Service
@Lazy(false) // Forca eager initialization
public class CacheWarmupService {

    @PostConstruct
    public void warmup() {
        // Pre-carrega dados essenciais
    }
}
```

### 10.2 Spring AOT e Class Data Sharing (CDS)

**Application Class Data Sharing (AppCDS) com Spring Boot 3.3+:**

O AppCDS permite que a JVM pré-processe e compartilhe metadados de classes, reduzindo significativamente o tempo de startup.

**Gerando o arquivo CDS automaticamente (Spring Boot 3.3+):**

```bash
# Passo 1: gerar o arquivo de classes (training run)
java -Dspring.context.exit=onRefresh \
     -XX:ArchiveClassesAtExit=app-cds.jsa \
     -jar app.jar

# Passo 2: executar com CDS
java -XX:SharedArchiveFile=app-cds.jsa \
     -jar app.jar
```

**Usando o plugin do Spring Boot (mais simples):**

```bash
# Maven — processo automatizado
./mvnw spring-boot:run -Dspring-boot.run.arguments="--spring.context.exit=onRefresh" \
    -Dspring-boot.run.jvmArguments="-XX:ArchiveClassesAtExit=app-cds.jsa"

# Próxima execução com CDS
./mvnw spring-boot:run \
    -Dspring-boot.run.jvmArguments="-XX:SharedArchiveFile=app-cds.jsa"
```

**Resultados tipicos:**

| Métrica              | Sem CDS   | Com AppCDS | Melhoria      |
|----------------------|-----------|------------|---------------|
| Tempo de startup     | ~3.5s     | ~2.1s      | ~40% mais rápido |
| Uso de memória (RSS) | ~250MB    | ~210MB     | ~16% menos     |

**Spring AOT (Ahead-of-Time processing):**

```xml
<!-- pom.xml — habilitar AOT processing -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>process-aot</id>
            <goals>
                <goal>process-aot</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

```bash
# Build com AOT
./mvnw clean package -Pnative

# Executar com AOT (sem native image, mas com otimizações AOT)
java -Dspring.aot.enabled=true -jar app.jar
```

### 10.3 Redução de Classpath

Cada dependência no classpath aumenta o tempo de startup (class scanning, auto-configuration).

**Excluir starters não utilizados:**

```xml
<!-- Se você usa apenas REST API, não precisa de template engines -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Se não usa Tomcat WebSocket -->
        <exclusion>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-websocket</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**Undertow vs Tomcat — servidor embarcado mais leve:**

```xml
<!-- Trocar Tomcat por Undertow (menor footprint de memória) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

**Comparação de servidores embarcados:**

| Aspecto           | Tomcat           | Undertow         | Jetty            |
|-------------------|------------------|------------------|------------------|
| Memória (idle)    | ~120MB           | ~90MB            | ~100MB           |
| Startup           | ~2.0s            | ~1.7s            | ~1.8s            |
| Throughput (RPS)  | Alto             | Muito alto       | Alto             |
| Maturidade        | Muito alta       | Alta             | Alta             |
| Comunidade        | Muito grande     | Grande           | Grande           |
| Padrão Spring Boot| Sim              | Não              | Não              |

**Desabilitando auto-configurations desnecessarias:**

```java
@SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class,         // Se não usa banco relacional
        MongoAutoConfiguration.class,              // Se não usa MongoDB
        SecurityAutoConfiguration.class,           // Se não usa Spring Security
        MailSenderAutoConfiguration.class           // Se não envia emails
})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Verificando quais auto-configurations estao ativas:**

```yaml
# application.yml
logging:
  level:
    org.springframework.boot.autoconfigure: DEBUG

# Ou via Actuator
management:
  endpoints:
    web:
      exposure:
        include: conditions
```

```bash
# Endpoint que mostra o que foi auto-configurado é o que não foi
curl http://localhost:8080/actuator/conditions | jq '.contexts.application.positiveMatches | keys'
```


---

**Parte 2 — Configuração de Propriedades e Infraestrutura**

## 11. Tuning do Spring Boot — application.yml

### 11.1 Configurações de Servidor

O Spring Boot embarca um servidor web (Tomcat por padrão) que aceita configurações via `application.yml`. Em produção, os valores padrão raramente são adequados para cargas reais. Um `application-prod.yml` bem ajustado evita saturação de threads, reduz latência e economiza banda com compressão.

```yaml
# application-prod.yml
server:
  port: 8080

  tomcat:
    # --- Pool de threads ---
    threads:
      max: 200            # máximo de worker threads (default: 200)
      min-spare: 20       # threads mantidas vivas mesmo ociosas (default: 10)

    # --- Conexões ---
    max-connections: 8192  # conexões simultâneas que o conector aceita (default: 8192)
    accept-count: 100      # fila de backlog do OS quando max-connections é atingido (default: 100)

    # --- Timeouts ---
    connection-timeout: 5s           # tempo máximo para estabelecer a conexão
    keep-alive-timeout: 30s          # tempo que uma conexão keep-alive fica ociosa
    max-keep-alive-requests: 200     # requisições por conexão keep-alive antes de fechar

    # --- Outros ---
    max-http-form-post-size: 5MB     # limite de POST para formulários
    max-swallow-size: 5MB            # tamanho máximo que o Tomcat "engole" de body não consumido

  # --- Compressão HTTP ---
  compression:
    enabled: true
    min-response-size: 1024                                # em bytes (default: 2048)
    mime-types: application/json,application/xml,text/html,text/xml,text/plain,application/javascript,text/css

  # --- Session ---
  servlet:
    session:
      timeout: 30m
```

Diagrama do fluxo de uma requisição no Tomcat embarcado:

```
  Cliente HTTP
       |
       v
  +-----------+
  |  Acceptor  |  (1-2 threads, aceita conexões TCP)
  +-----------+
       |
       v
  +-----------+
  |  Poller    |  (NIO selector, monitora I/O)
  +-----------+
       |
       v
  +----------------+
  |  Worker Pool   |  (server.tomcat.threads.max)
  |  Thread 1      |
  |  Thread 2      |  --> Executa Filter → Servlet → Controller
  |  ...           |
  |  Thread N      |
  +----------------+
       |
       v
  Resposta ao cliente
```

Relação entre os parâmetros de conexão:

| Parâmetro | Significado | Efeito de valor baixo | Efeito de valor alto |
|---|---|---|---|
| `threads.max` | Worker threads no pool | Requests enfileiradas, latência sobe | Mais memória, context switching |
| `threads.min-spare` | Threads pré-criadas | Cold start nas primeiras requests | Memória ociosa reservada |
| `max-connections` | Conexões TCP simultâneas | Rejeição de conexões sob carga | Mais file descriptors usados |
| `accept-count` | Backlog da fila do OS | Conexões recusadas antes de chegar ao app | Fila longa aumenta latência percebida |

### 11.2 Configurações de JPA/Hibernate para Performance

O Hibernate oferece dezenas de propriedades que impactam diretamente a performance. As configurações abaixo representam o conjunto mínimo recomendado para produção.

```yaml
# application-prod.yml
spring:
  jpa:
    # CRÍTICO: desabilitar Open Session in View
    open-in-view: false

    properties:
      hibernate:
        # --- Batching de statements ---
        jdbc:
          batch_size: 25                          # agrupa INSERTs/UPDATEs em lotes de 25
          batch_versioned_data: true               # permite batch mesmo com @Version

        order_inserts: true                        # reordena INSERTs para maximizar batching
        order_updates: true                        # reordena UPDATEs para maximizar batching

        # --- Otimização de queries IN ---
        query:
          in_clause_parameter_padding: true         # arredonda parâmetros de IN (1,2,4,8,16...)
                                                    # melhora cache de planos de execução do DB

        # --- Autocommit ---
        connection:
          provider_disables_autocommit: true        # evita roundtrip SET autocommit=0 por transação
                                                    # REQUER que o pool (HikariCP) configure auto-commit=false

        # --- Geração de estatísticas (desabilitar em prod) ---
        generate_statistics: false

        # --- Cache de segundo nível ---
        cache:
          use_second_level_cache: false             # habilitar apenas se necessário
          use_query_cache: false

        # --- Logging SQL (desabilitar em prod) ---
        show_sql: false
        format_sql: false

    hibernate:
      ddl-auto: validate                            # NUNCA use update/create em produção

  # --- HikariCP (pool de conexões) ---
  datasource:
    hikari:
      maximum-pool-size: 20                         # regra: (2 * núcleos CPU) + disco_spindles
      minimum-idle: 5
      idle-timeout: 300000                           # 5 min (ms)
      max-lifetime: 1800000                          # 30 min (ms)
      connection-timeout: 30000                      # 30s (ms)
      auto-commit: false                             # ESSENCIAL para provider_disables_autocommit
      data-source-properties:
        prepStmtCacheSize: 250                       # cache de prepared statements
        prepStmtCacheSqlLimit: 2048
        useServerPrepStmts: true                     # prepared statements no servidor
```

**Por que `open-in-view: false`?**

O padrão do Spring Boot é `true`, o que mantém a Session do Hibernate aberta durante toda a requisição HTTP, incluindo a renderização da view. Isso causa:

1. **Queries N+1 escondidas** — lazy loading dispara SELECTs durante serialização JSON sem que o desenvolvedor perceba.
2. **Conexão com o banco presa** — a conexão do pool fica alocada desde o início do controller até o fim da resposta HTTP.
3. **Problemas de diagnóstico** — erros de `LazyInitializationException` que deveriam aparecer no service ficam mascarados.

```java
// Com open-in-view=false, resolver relacionamentos explicitamente:
@Service
@Transactional(readOnly = true)
public class PedidoService {

    public PedidoDTO buscarComItens(Long id) {
        Pedido pedido = pedidoRepository.findByIdComItens(id)
            .orElseThrow(() -> new EntityNotFoundException("Pedido não encontrado"));

        return PedidoDTO.fromEntity(pedido); // conversão dentro da transação
    }
}
```

**Efeito do `in_clause_parameter_padding`:**

```
-- Sem padding: cada quantidade de IDs gera um plano diferente
SELECT * FROM produto WHERE id IN (?, ?)            -- plano 1
SELECT * FROM produto WHERE id IN (?, ?, ?)         -- plano 2
SELECT * FROM produto WHERE id IN (?, ?, ?, ?)      -- plano 3

-- Com padding: arredonda para potência de 2, reutiliza planos
SELECT * FROM produto WHERE id IN (?, ?)                      -- 2 IDs → 2
SELECT * FROM produto WHERE id IN (?, ?, ?, ?)                -- 3 IDs → 4
SELECT * FROM produto WHERE id IN (?, ?, ?, ?, ?, ?, ?, ?)    -- 5-8 IDs → 8
```

> Para detalhes sobre queries e fetch strategies, consulte [Dicas-JPA-Hibernate-Queries.md](../Dicas-JPA-Hibernate-Queries.md).

### 11.3 Configurações de Jackson

O Jackson é o serializador JSON padrão do Spring Boot. Configurações adequadas evitam overhead desnecessário.

```yaml
# application-prod.yml
spring:
  jackson:
    serialization:
      write-dates-as-timestamps: false                   # datas como string ISO, não long
      fail-on-empty-beans: false                         # não lança exceção para beans vazios
    deserialization:
      fail-on-unknown-properties: false                  # ignora campos desconhecidos
      adjust-dates-to-context-time-zone: false
    default-property-inclusion: non_null                  # não serializa campos null
    time-zone: UTC
    locale: pt-BR
```

Para alto volume de serialização, o módulo **Blackbird** substitui reflection por `MethodHandles`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-blackbird</artifactId>
</dependency>
```

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer blackbirdCustomizer() {
        return builder -> builder.modulesToInstall(new BlackbirdModule());
    }
}
```

> Para detalhes sobre configuração e módulos do Jackson, consulte [Dicas-Jackson.md](../Dicas-Jackson.md).

### 11.4 Configurações de Logging para Produção

Em produção, o nível de log deve ser restritivo para reduzir I/O de disco. O appender assíncrono evita que threads da aplicação bloqueiem na escrita em disco.

```yaml
# application-prod.yml
logging:
  level:
    root: WARN
    com.empresa.meuprojeto: INFO
    org.springframework: WARN
    org.hibernate.SQL: WARN
    org.hibernate.type.descriptor.sql.BasicBinder: WARN
    com.zaxxer.hikari: WARN
    org.apache.catalina: WARN
```

Configuração do Logback com appender assíncrono e rotação:

```xml
<!-- src/main/resources/logback-spring.xml -->
<configuration>

    <property name="LOG_PATH" value="${LOG_PATH:-/var/log/app}" />
    <property name="LOG_FILE" value="${LOG_FILE:-application}" />
    <property name="MAX_FILE_SIZE" value="100MB" />
    <property name="MAX_HISTORY" value="30" />
    <property name="TOTAL_SIZE_CAP" value="3GB" />

    <!-- Console (dev) -->
    <springProfile name="!prod">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} %highlight(%-5level) [%thread] %cyan(%logger{36}) - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

    <!-- Produção -->
    <springProfile name="prod">

        <!-- Appender síncrono: arquivo com rotação -->
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_PATH}/${LOG_FILE}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${LOG_FILE}.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
                <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
                <maxHistory>${MAX_HISTORY}</maxHistory>
                <totalSizeCap>${TOTAL_SIZE_CAP}</totalSizeCap>
            </rollingPolicy>
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>

        <!-- Appender assíncrono: não bloqueia threads da aplicação -->
        <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
            <queueSize>1024</queueSize>
            <discardingThreshold>0</discardingThreshold>
            <neverBlock>true</neverBlock>
            <includeCallerData>false</includeCallerData>
            <appender-ref ref="FILE" />
        </appender>

        <!-- JSON estruturado para ingestão em ELK/Loki -->
        <appender name="JSON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_PATH}/${LOG_FILE}-json.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${LOG_FILE}-json.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
                <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>
                <maxHistory>${MAX_HISTORY}</maxHistory>
                <totalSizeCap>${TOTAL_SIZE_CAP}</totalSizeCap>
            </rollingPolicy>
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>traceId</includeMdcKeyName>
                <includeMdcKeyName>spanId</includeMdcKeyName>
                <includeMdcKeyName>userId</includeMdcKeyName>
            </encoder>
        </appender>

        <appender name="ASYNC_JSON" class="ch.qos.logback.classic.AsyncAppender">
            <queueSize>1024</queueSize>
            <discardingThreshold>0</discardingThreshold>
            <neverBlock>true</neverBlock>
            <includeCallerData>false</includeCallerData>
            <appender-ref ref="JSON_FILE" />
        </appender>

        <root level="WARN">
            <appender-ref ref="ASYNC_FILE" />
            <appender-ref ref="ASYNC_JSON" />
        </root>

        <logger name="com.empresa.meuprojeto" level="INFO" />
    </springProfile>

</configuration>
```

Parâmetros do `AsyncAppender` e seus trade-offs:

| Parâmetro | Default | Efeito |
|---|---|---|
| `queueSize` | 256 | Tamanho da fila interna. Fila maior absorve picos, mas consome mais memória |
| `discardingThreshold` | 20% | Quando a fila atinge esse % de ocupação, descarta TRACE/DEBUG. `0` = nunca descarta |
| `neverBlock` | false | `true` = nunca bloqueia a thread; descarta se a fila encher |
| `includeCallerData` | false | `true` captura classe/método/linha. Muito custoso; evitar em prod |

> Para detalhes sobre logging e observabilidade, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

---

## 12. Thread Pool Tuning — Tomcat, Undertow e Jetty

### 12.1 Modelo de Threading dos Servidores Embedded

Todos os servidores embedded do Spring Boot seguem um modelo similar: threads aceitadoras recebem conexões TCP e as delegam para um pool de worker threads.

```
                            Tomcat / Undertow / Jetty
                            ========================

  Conexão TCP            +---------------------------+
  do cliente  ---------> | Acceptor Thread(s)        |
                         | (1-2 threads dedicadas)   |
                         +---------------------------+
                                    |
                                    v
                         +---------------------------+
                         | I/O Selector / Poller     |
                         | (NIO event loop)          |
                         +---------------------------+
                                    |
                                    v
                         +---------------------------+
                         |     Worker Thread Pool     |
                         |                           |
                         |  +-----+  +-----+        |
                         |  | T-1 |  | T-2 |  ...   |  --> Filter chain
                         |  +-----+  +-----+        |  --> DispatcherServlet
                         |  +-----+  +-----+        |  --> Controller
                         |  | T-3 |  | T-N |        |  --> Service / Repository
                         |  +-----+  +-----+        |
                         |                           |
                         +---------------------------+
```

Comparação dos valores padrão:

| Parâmetro | Tomcat | Undertow | Jetty |
|---|---|---|---|
| Worker threads (max) | 200 | 64 (I/O) + 8×cores (workers) | 200 |
| Worker threads (min) | 10 | — | 8 |
| Max connections | 8192 | ilimitado (I/O threads) | 65536 |
| Accept queue (backlog) | 100 | 1000 | 0 (OS default) |
| Connection timeout | 60s | 60s | 30s |
| Modelo I/O | NIO (non-blocking) | XNIO (non-blocking) | NIO (non-blocking) |

### 12.2 Sizing do Thread Pool

O dimensionamento do pool depende da natureza da carga. A **Lei de Little** fornece a base teórica:

```
Threads necessárias = Throughput desejado (req/s) × Latência média (s)
```

Para 500 req/s com latência média de 200ms:

```
Threads = 500 × 0.2 = 100
```

| Tipo de carga | Fórmula | Exemplo (8 cores) |
|---|---|---|
| CPU-bound | `N_threads = N_cores + 1` | 9 threads |
| I/O-bound | `N_threads = N_cores × (1 + W/C)` | 8 × (1 + 10) = 88 threads |
| Misto | Medir com profiling | Depende da proporção |

Onde: `W` = tempo de espera (I/O, rede, banco), `C` = tempo de computação (CPU).

```java
@Component
public class TomcatTuning implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    @Value("${app.server.threads.max:200}")
    private int maxThreads;

    @Value("${app.server.threads.min-spare:20}")
    private int minSpare;

    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.addConnectorCustomizers(connector -> {
            var protocol = (AbstractProtocol<?>) connector.getProtocolHandler();
            protocol.setMaxThreads(maxThreads);
            protocol.setMinSpareThreads(minSpare);
            protocol.setMaxConnections(8192);
            protocol.setAcceptCount(100);
            protocol.setKeepAliveTimeout(30000);
            protocol.setMaxKeepAliveRequests(200);
            protocol.setConnectionTimeout(5000);
        });
    }
}
```

### 12.3 Undertow como Alternativa Leve

O Undertow (Red Hat/WildFly) tem menor footprint de memória. A troca é simples:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

```yaml
server:
  undertow:
    threads:
      io: 4                     # I/O threads (default: cores)
      worker: 64                # worker threads (default: 8 × cores)
    buffer-size: 1024
    direct-buffers: true
```

Comparação de performance (valores aproximados, REST simples):

| Métrica | Tomcat | Undertow | Jetty |
|---|---|---|---|
| Throughput (req/s) | ~18.000 | ~20.500 | ~17.500 |
| Latência p99 | ~12ms | ~9ms | ~13ms |
| Memória RSS (idle) | ~180MB | ~140MB | ~170MB |
| Memória RSS (carga) | ~320MB | ~260MB | ~310MB |
| Startup time | ~2.1s | ~1.8s | ~2.3s |

| Cenário | Recomendação |
|---|---|
| API REST de alta concorrência | Undertow |
| Aplicação com WebSocket intensivo | Undertow |
| Aplicação legada com Servlets/JSP | Tomcat |
| Microsserviços com muitas instâncias | Undertow |

---

## 13. Tuning de Banco de Dados — PostgreSQL

### 13.1 Parâmetros Essenciais do postgresql.conf

As configurações abaixo assumem um servidor dedicado com 16 GB de RAM e discos SSD.

```properties
# =============================================================================
# postgresql.conf — Tuning para produção (servidor 16 GB RAM, SSD)
# =============================================================================

# --- Memória ---
shared_buffers = 4GB                  # 25% da RAM total
effective_cache_size = 12GB           # 75% da RAM (informa o planner, não aloca)
work_mem = 32MB                       # POR OPERAÇÃO de sort/hash join
maintenance_work_mem = 512MB          # VACUUM, CREATE INDEX
wal_buffers = 64MB

# --- WAL ---
wal_level = replica
checkpoint_timeout = 15min
max_wal_size = 4GB
min_wal_size = 1GB
checkpoint_completion_target = 0.9

# --- Planner ---
random_page_cost = 1.1                # SSD (default 4.0 é para HDD)
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
default_statistics_target = 200

# --- Conexões ---
max_connections = 100                 # cada conexão ~10MB RAM
superuser_reserved_connections = 3

# --- Logging ---
log_min_duration_statement = 500      # loga queries > 500ms
log_statement = 'ddl'
log_temp_files = 0                    # indica work_mem baixo
log_checkpoints = on
log_lock_waits = on

# --- Autovacuum ---
autovacuum_max_workers = 4
autovacuum_naptime = 30s
autovacuum_vacuum_scale_factor = 0.05
autovacuum_analyze_scale_factor = 0.02
```

```
+--------------------------------------------+
|  Servidor com 16 GB RAM, SSD, 8 cores      |
+--------------------------------------------+
| shared_buffers     = 4GB   (25% de 16GB)   |
| effective_cache_size = 12GB (75% de 16GB)  |
| work_mem           = 32MB  (cuidado!)      |
| maintenance_work_mem = 512MB               |
| random_page_cost   = 1.1   (SSD)           |
| max_connections    = 100                    |
| max_parallel_workers = 8   (= cores)       |
+--------------------------------------------+
```

### 13.2 Análise de Queries com EXPLAIN ANALYZE

O `EXPLAIN ANALYZE` executa a query e mostra o plano real com tempos.

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT p.nome, c.nome AS categoria
FROM produto p
JOIN categoria c ON c.id = p.categoria_id
WHERE p.preco > 100.00
  AND p.ativo = true
ORDER BY p.nome;
```

Exemplo de saída:

```
Sort  (cost=1245..1250 rows=1823 width=64) (actual time=12.3..12.9 rows=1500 loops=1)
  Sort Key: p.nome
  Sort Method: quicksort  Memory: 180kB
  Buffers: shared hit=892 read=45
  ->  Hash Join  (cost=45..1150 rows=1823 width=64) (actual time=0.9..10.2 rows=1500 loops=1)
        ->  Index Scan using idx_produto_ativo_preco on produto p
              Index Cond: ((preco > 100.00) AND (ativo = true))
              Buffers: shared hit=850 read=45
        ->  Hash  (cost=30..30 rows=50 width=20)
              ->  Seq Scan on categoria c  Buffers: shared hit=40
Planning Time: 0.234 ms
Execution Time: 13.123 ms
```

**Tipos de scan:**

| Tipo de Scan | Quando o planner escolhe | Performance |
|---|---|---|
| **Seq Scan** | Tabela pequena ou retorna >5-10% das linhas | O(n) |
| **Index Scan** | Poucas linhas e índice adequado | O(log n) + leituras aleatórias |
| **Index Only Scan** | Todos os campos estão no índice (covering) | O(log n) sem acesso à tabela |
| **Bitmap Index Scan** | Volume intermediário (1-10%) | Combina índice + acesso sequencial |

### 13.3 pg_stat_statements para Queries Lentas

```properties
# postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
pg_stat_statements.track_planning = on
```

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top 10 queries mais lentas (tempo total acumulado)
SELECT
    round(total_exec_time::numeric, 2) AS total_ms,
    calls,
    round(mean_exec_time::numeric, 2) AS media_ms,
    rows,
    round((100.0 * total_exec_time / sum(total_exec_time) OVER ())::numeric, 2) AS pct_total,
    left(query, 120) AS query_resumida
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 13.4 Connection Limits e pgBouncer

Cada conexão no PostgreSQL cria um **processo dedicado** (~5-10 MB de RAM).

```
  Sem pgBouncer                              Com pgBouncer
  ========================                   =======================

  App 1 (pool=20) --+                        App 1 (pool=20) --+
  App 2 (pool=20) --+--> PostgreSQL          App 2 (pool=20) --+--> pgBouncer --> PostgreSQL
  App 3 (pool=20) --+    max_conn=100        App 3 (pool=20) --+    pool=30       max_conn=50
  App 4 (pool=20) --+    80 conn ociosas!    App 4 (pool=20) --+    multiplexação
  App 5 (pool=20) --+                        App 5 (pool=20) --+
```

**Modos de pooling do pgBouncer:**

| Modo | Descrição | Uso ideal |
|---|---|---|
| `session` | Conexão reservada para toda a sessão | Apps com session state |
| `transaction` | Conexão reservada durante a transação | **Maioria das apps web** |
| `statement` | Conexão devolvida após cada statement | Autocommit simples (raro) |

```ini
# pgbouncer.ini
[databases]
meubanco = host=127.0.0.1 port=5432 dbname=meubanco

[pgbouncer]
listen_port = 6432
pool_mode = transaction
default_pool_size = 30
max_client_conn = 500
max_db_connections = 50
server_idle_timeout = 600
query_timeout = 30
```

**Configuração Spring Boot para pgBouncer:**

```yaml
spring:
  datasource:
    url: jdbc:postgresql://pgbouncer-host:6432/meubanco
    hikari:
      maximum-pool-size: 20
      auto-commit: false
      data-source-properties:
        prepareThreshold: 0   # desabilitar prepared statements do servidor (incompatível com pgBouncer transaction mode)
```

| Cenário | pgBouncer necessário? |
|---|---|
| 1 app, pool < 50 conexões | Não. HikariCP é suficiente |
| Múltiplas apps somando > 100 conexões | Sim |
| Serverless com muitas instâncias | Sim |
| Microsserviços (10+ instâncias) | Sim |

> Para detalhes sobre modelagem de queries, consulte [Dicas-JPA-Hibernate-Queries.md](../Dicas-JPA-Hibernate-Queries.md).

---

## 14. GraalVM Native Image — Considerações de Performance

### 14.1 Startup vs Throughput

| Métrica | JVM (JIT) | Native Image (AOT) | Diferença |
|---|---|---|---|
| **Startup time** | ~2.5s | ~0.08s | ~30× mais rápido |
| **Time to first request** | ~3.0s | ~0.10s | ~30× mais rápido |
| **Peak throughput** (req/s) | ~45.000 | ~32.000 | ~30% menor |
| **RSS memória (idle)** | ~180MB | ~45MB | ~4× menor |
| **RSS memória (carga)** | ~350MB | ~120MB | ~3× menor |
| **Tamanho do artefato** | ~25MB (JAR) | ~80MB (binário) | ~3× maior |
| **Tempo de build** | ~10s | ~3-5min | ~20-30× mais lento |

**Por que o throughput nativo é menor?**

```
JVM (JIT) — Otimização progressiva:
  t=0s        t=5s         t=30s        t=60s
  Interpreter --> C1 JIT   --> C2 JIT   --> Peak
  (lento)        (rápido)     (otimizado)  (máximo)
                              Usa profiling runtime:
                              hot paths, inlining, escape analysis

Native Image (AOT) — Performance constante:
  t=0s        t=5s         t=30s        t=60s
  Compilado   --> Mesma     --> Mesma    --> Mesma performance
  (já otimizado)
  Não melhora em runtime.
```

### 14.2 Profile-Guided Optimization (PGO)

```bash
# Passo 1: build com instrumentação
mvn -Pnative native:compile -Dgraalvm.native-image.options="--pgo-instrument"

# Executar com carga representativa
./target/minha-app &
wrk -t4 -c50 -d120s http://localhost:8080/api/produtos
kill $!

# Passo 2: recompilar com o perfil
mvn -Pnative native:compile -Dgraalvm.native-image.options="--pgo=default.iprof"
```

| Métrica | Sem PGO | Com PGO | Ganho |
|---|---|---|---|
| Peak throughput (req/s) | ~32.000 | ~39.000 | ~22% |
| Latência p99 | ~8ms | ~5ms | ~37% |

### 14.3 Quando Usar Native Image

| Tipo de aplicação | Recomendação | Justificativa |
|---|---|---|
| **Serverless / FaaS** | **Native Image** | Cold start é crítico |
| **CLI tools** | **Native Image** | Startup instantâneo |
| **Microsserviços leves** (CRUD) | **Native Image** | Menor footprint por instância |
| **Microsserviços pesados** (ML) | **JVM** | Throughput de pico importa mais |
| **Monolitos long-running** | **JVM** | JIT otimiza progressivamente |
| **Batch processing** | **JVM** | Processamento longo; JIT maximiza throughput |
| **APIs com auto-scaling agressivo** | **Native Image** | Scale-up rápido sem cold start |

```
Cenário: API REST com auto-scaling (10-100 pods)

JVM:
  - Startup: 3s, warmup: ~30s, memória/pod: 350MB
  - Total (50 pods): 17.5GB RAM

Native Image:
  - Startup: 0.08s, performance total em <1s, memória/pod: 120MB
  - Total (50 pods): 6GB RAM
  - Economia: ~65% de RAM
```

> Para detalhes sobre compilação e configuração de hints, consulte [GraalVM-Native-Image-Spring-Boot.md](GraalVM-Native-Image-Spring-Boot.md).

---

**Parte 3 — Configuração da JVM**

## 15. Memória e Garbage Collection — Fundamentos

A JVM gerencia memória automaticamente, mas isso não significa que o desenvolvedor pode ignorar como essa gestão funciona. Compreender o modelo de memória, o ciclo de vida dos objetos e as métricas de Garbage Collection é essencial para diagnosticar problemas de desempenho, evitar pausas excessivas e dimensionar corretamente os recursos em produção.

### 15.1 Modelo de Memória da JVM

A memória da JVM é dividida em regiões com propósitos distintos. O diagrama abaixo mostra a visão geral:

```
+-----------------------------------------------------------------------+
|                         PROCESSO DA JVM                               |
|                                                                       |
|  +--------------------------------------------------------------+    |
|  |                         HEAP                                  |    |
|  |                                                               |    |
|  |  +-------------------------+  +----------------------------+  |    |
|  |  |      Young Generation   |  |       Old Generation       |  |    |
|  |  |                         |  |                            |  |    |
|  |  |  +-------+ +---------+ |  |  Objetos que sobreviveram  |  |    |
|  |  |  | Eden  | |Survivor | |  |  múltiplos ciclos de GC.   |  |    |
|  |  |  |       | | S0 | S1 | |  |  Coletados por Major GC.   |  |    |
|  |  |  | Novos | |         | |  |                            |  |    |
|  |  |  | objs  | | Cópia   | |  |                            |  |    |
|  |  |  | aqui  | | entre   | |  |                            |  |    |
|  |  |  |       | | S0<->S1 | |  |                            |  |    |
|  |  |  +-------+ +---------+ |  +----------------------------+  |    |
|  |  +-------------------------+                                  |    |
|  +--------------------------------------------------------------+    |
|                                                                       |
|  +------------------+  +------------------+  +-------------------+    |
|  |    Metaspace      |  |     Stack        |  |  Direct Memory    |    |
|  |                   |  |  (por thread)    |  |                   |    |
|  |  Metadados de     |  |  Frames de       |  |  ByteBuffers      |    |
|  |  classes, method  |  |  métodos,        |  |  alocados fora    |    |
|  |  tables, constant |  |  variáveis       |  |  do heap via      |    |
|  |  pool. Cresce     |  |  locais,         |  |  allocateDirect() |    |
|  |  conforme classes |  |  referências.    |  |  ou Netty/NIO.    |    |
|  |  são carregadas.  |  |  Tamanho fixo    |  |                   |    |
|  |                   |  |  por thread.     |  |                   |    |
|  +------------------+  +------------------+  +-------------------+    |
|                                                                       |
|  +-------------------------------------------------------------------+|
|  | Code Cache: código compilado pelo JIT (C1/C2)                      |
|  +-------------------------------------------------------------------+|
+-----------------------------------------------------------------------+
```

**O que vive em cada região:**

| Região | Conteúdo | Controlada por |
|--------|----------|----------------|
| **Eden** | Objetos recém-criados. A maioria morre aqui (objetos de curta duração). | `-Xmn`, `-XX:NewRatio` |
| **Survivor (S0/S1)** | Objetos que sobreviveram a pelo menos um Minor GC. Copiados entre S0 e S1 a cada ciclo. | `-XX:SurvivorRatio` |
| **Old Generation** | Objetos promovidos após sobreviverem N ciclos de Minor GC (threshold configurável). | `-Xmx` (limite do heap total) |
| **Metaspace** | Metadados de classes carregadas, tabelas de métodos, constant pool. Substituiu o PermGen desde o Java 8. | `-XX:MetaspaceSize`, `-XX:MaxMetaspaceSize` |
| **Stack** | Um stack por thread: frames de métodos, variáveis locais, referências. Estouro causa `StackOverflowError`. | `-Xss` |
| **Direct Memory** | Buffers alocados fora do heap, usados por NIO, Netty, e frameworks que fazem I/O de alta performance. | `-XX:MaxDirectMemorySize` |
| **Code Cache** | Código nativo gerado pelo compilador JIT. Se ficar cheio, a compilação JIT para. | `-XX:ReservedCodeCacheSize` |

> **Observação:** A partir do Java 8, o PermGen foi removido e substituído pelo Metaspace, que cresce dinamicamente na memória nativa. É recomendável definir um limite máximo com `-XX:MaxMetaspaceSize` para evitar crescimento descontrolado em aplicações que carregam classes dinamicamente (como frameworks de proxy e classloaders customizados).

### 15.2 Ciclo de Vida dos Objetos

Cada objeto Java percorre um caminho previsível pela memória do heap:

```
  Alocação              Minor GC              Promoção            Major GC
     |                     |                     |                    |
     v                     v                     v                    v
  +-------+          +-----------+          +-----------+       +-----------+
  | Eden  |--vivo?-->| Survivor  |--idade-->| Old Gen   |--GC-->| Liberado  |
  |       |   sim    | S0 <-> S1 |  >= N    |           |       | ou mantido|
  +-------+          +-----------+          +-----------+       +-----------+
     |                     |
     | morto?              | morto?
     v                     v
  Liberado              Liberado
```

**Tipos de coleta:**

| Tipo | O que coleta | Quando ocorre | Impacto |
|------|-------------|---------------|---------|
| **Minor GC** (Young GC) | Apenas Young Generation (Eden + Survivors). Rápido porque a maioria dos objetos já está morta. | Eden cheio | Pausa curta (ms a dezenas de ms) |
| **Major GC** (Old GC) | Old Generation. Mais lento porque objetos vivem mais. | Old Gen atingindo threshold de ocupação | Pausa moderada a longa |
| **Full GC** | Todo o heap (Young + Old + Metaspace). O mais custoso. | Falha de alocação, `System.gc()`, Metaspace cheia | Pausa longa (centenas de ms a segundos) |

**Promoção de objetos (Young para Old):**

Um objeto é promovido para a Old Generation quando:
1. **Idade:** sobreviveu a N ciclos de Minor GC (padrão: 15 ciclos, configurável com `-XX:MaxTenuringThreshold`).
2. **Tamanho:** o objeto é maior que metade do Survivor space (alocado diretamente na Old Gen).
3. **Survivor cheio:** não há espaço no Survivor para copiar objetos sobreviventes.

**Promoção prematura** é um dos problemas mais comuns de GC. Quando objetos de vida curta são promovidos para a Old Gen antes de morrer, a Old Gen enche mais rápido, forçando Major GCs frequentes. Causas típicas:

- Survivor spaces subdimensionados;
- Burst de alocação que lotam o Eden e o Survivor simultaneamente;
- Objetos que sobrevivem "por pouco" mais que o threshold de idade.

```java
// Exemplo de alocação que pode causar promoção prematura
public List<RelatorioPDF> gerarRelatoriosMensal(List<Cliente> clientes) {
    // Se clientes tem 100.000 registros, este método aloca muitos objetos
    // temporários que podem sobreviver ao Minor GC se o processamento
    // de cada cliente for lento o suficiente
    return clientes.stream()
        .map(c -> {
            byte[] dados = buscarDadosBanco(c);     // aloca buffer temporário
            String html = renderizarTemplate(dados); // aloca String grande
            return converterParaPDF(html);            // aloca resultado final
        })
        .collect(Collectors.toList());
}

// Melhor: processar em lotes para reduzir pressão simultânea no heap
public void gerarRelatoriosMensalEmLotes(List<Cliente> clientes) {
    Lists.partition(clientes, 500).forEach(lote -> {
        List<RelatorioPDF> pdfs = lote.stream()
            .map(this::processarCliente)
            .collect(Collectors.toList());
        salvarEmDisco(pdfs);
        // pdfs elegíveis para GC após esta iteração
    });
}
```

### 15.3 Métricas de GC

Monitorar o Garbage Collector é essencial para identificar gargalos de desempenho. As métricas-chave são:

**Métricas principais:**

| Métrica | O que mede | Valor aceitável |
|---------|-----------|-----------------|
| **Pause Time** | Duração de cada pausa de GC (stop-the-world) | < 200 ms para apps web, < 10 ms para low-latency |
| **Throughput** | % do tempo gasto executando código da aplicação (vs. GC) | > 95% (idealmente > 99%) |
| **Allocation Rate** | Velocidade de alocação de novos objetos no Eden | Depende da app; picos podem indicar problema |
| **Promotion Rate** | Volume de objetos promovidos de Young para Old por segundo | Deve ser baixo; alto indica promoção prematura |
| **Heap After GC** | Ocupação do heap logo após o GC | Deve ser estável; crescimento contínuo indica memory leak |

**Habilitando GC Logging (Java 11+):**

A JVM moderna usa o sistema Unified Logging com `-Xlog`:

```bash
# Logging básico de GC
java -Xlog:gc*:file=gc.log:time,level,tags -jar app.jar

# Logging detalhado com rotação de arquivos
java \
  -Xlog:gc*,gc+phases=debug,gc+age=trace:file=gc.log:time,uptime,level,tags:filecount=5,filesize=100m \
  -jar app.jar

# Logging para stdout (útil em containers)
java -Xlog:gc*:stdout:time,level,tags -jar app.jar
```

**Anatomia dos parâmetros de `-Xlog`:**

```
-Xlog:<seletores>:<saida>:<decoradores>:<opções>

Seletores:    gc*                       -> todos os tags que começam com gc
              gc+phases=debug           -> fases do GC com nível debug
              gc+age=trace              -> informação de idade dos objetos

Saída:        file=gc.log               -> grava em arquivo
              stdout                    -> saída padrão

Decoradores:  time                      -> timestamp absoluto
              uptime                    -> tempo desde startup
              level                     -> nível do log (info, debug, trace)
              tags                      -> tags do log

Opções:       filecount=5               -> manter até 5 arquivos
              filesize=100m             -> rotacionar a cada 100 MB
```

**Lendo um GC log — exemplo de saída:**

```
[2024-01-15T14:30:01.234+0000][info][gc] GC(42) Pause Young (Normal) (G1 Evacuation Pause)
[2024-01-15T14:30:01.234+0000][info][gc] GC(42)   Eden: 512M->0B  Survivors: 64M->64M  Old: 1024M->1040M
[2024-01-15T14:30:01.234+0000][info][gc] GC(42)   Metaspace: 128M->128M
[2024-01-15T14:30:01.234+0000][info][gc] GC(42) Pause Young (Normal) 1600M->1104M(2048M) 12.345ms
```

**Como interpretar:**

| Campo | Significado |
|-------|-------------|
| `GC(42)` | Número sequencial do evento de GC |
| `Pause Young (Normal)` | Tipo: Minor GC (coleta da Young Generation) |
| `Eden: 512M->0B` | Eden tinha 512 MB, foi totalmente coletado |
| `Survivors: 64M->64M` | Survivor manteve 64 MB (objetos sobreviventes) |
| `Old: 1024M->1040M` | Old Gen cresceu 16 MB (promoção) |
| `1600M->1104M(2048M)` | Heap total: de 1600 MB para 1104 MB, com máximo de 2048 MB |
| `12.345ms` | Duração da pausa (stop-the-world) |

**Sinais de alerta nos logs:**

```
# Full GC — quase sempre indica problema
[gc] GC(99) Pause Full (Ergonomics) 1800M->1750M(2048M) 2345.678ms
                                     ^^^^^^^^^^ quase não liberou memória
                                                             ^^^^^^^^^^ pausa longa

# GC frequente com pouca liberação — possível memory leak
[gc] GC(100) Pause Young 2040M->2020M(2048M) 45.123ms
[gc] GC(101) Pause Young 2040M->2025M(2048M) 52.456ms
[gc] GC(102) Pause Full  2040M->2030M(2048M) 3456.789ms
                          ^^^^^^^^^^^^ heap quase cheio, GC não libera espaço
```

**Ferramentas para análise de GC logs:**

- **GCViewer** (open source): visualização gráfica de logs de GC.
- **GCEasy** (gceasy.io): análise online que gera relatórios com recomendações.
- **JDK Mission Control (JMC)**: ferramenta oficial da Oracle, integra com Flight Recorder.
- **VisualVM**: monitoramento em tempo real do heap, threads e GC.

```bash
# Usar jstat para monitoramento rápido em tempo real
# Mostra estatísticas do GC a cada 1 segundo, 10 amostras
jstat -gcutil <PID> 1000 10

#   S0     S1     E      O      M     CCS    YGC   YGCT    FGC  FGCT   CGC  CGCT    GCT
#  0.00  98.12  45.67  62.34  97.89  95.12   142   1.234    2  0.456    12  0.089   1.779
#
# S0/S1 = Survivor   E = Eden   O = Old   M = Metaspace
# YGC/YGCT = Young GC count/time   FGC/FGCT = Full GC count/time
```

---

## 16. Garbage Collectors — G1, ZGC e Shenandoah

A escolha do Garbage Collector impacta diretamente a latência, o throughput e o consumo de recursos da aplicação. A JVM oferece vários coletores, cada um otimizado para cenários diferentes.

### 16.1 G1GC — O Coletor Padrão

O G1 (Garbage-First) é o coletor padrão desde o Java 9. Ele foi projetado para oferecer um equilíbrio entre throughput e latência, com pausas previsíveis.

**Como o G1 funciona:**

Diferente dos coletores tradicionais que dividem o heap em regiões contíguas (Young e Old), o G1 divide o heap em centenas ou milhares de **regiões** (regions) de tamanho igual:

```
+-----+-----+-----+-----+-----+-----+-----+-----+
|  E  |  E  |  S  |  O  |  O  |  H  |  E  | Vazio|
+-----+-----+-----+-----+-----+-----+-----+-----+
|  O  |  O  |  E  |  E  | Vazio|  O  |  S  |  O  |
+-----+-----+-----+-----+-----+-----+-----+-----+
|  E  | Vazio|  O  |  O  |  O  |  H  |  H  | Vazio|
+-----+-----+-----+-----+-----+-----+-----+-----+

E = Eden    S = Survivor    O = Old    H = Humongous (objeto grande)
Vazio = região livre

Cada região tem tamanho fixo (1 MB a 32 MB, determinado automaticamente)
```

**Fases do G1:**

1. **Young-only phase:** coleta apenas regiões Eden e Survivor (equivalente ao Minor GC).
2. **Concurrent marking:** marca objetos vivos na Old Gen **concorrentemente** (sem parar a aplicação).
3. **Mixed collections:** após a marcação, o G1 seleciona as regiões Old com maior proporção de lixo (daí o nome "Garbage-First") e as coleta junto com as regiões Young.
4. **Full GC (fallback):** ocorre apenas quando as fases anteriores não conseguem liberar espaço suficiente. Deve ser evitado.

**Flags essenciais para G1:**

```bash
java \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:G1HeapRegionSize=16m \
  -XX:InitiatingHeapOccupancyPercent=45 \
  -XX:G1ReservePercent=10 \
  -XX:ParallelGCThreads=8 \
  -XX:ConcGCThreads=2 \
  -jar app.jar
```

| Flag | Padrão | Descrição |
|------|--------|-----------|
| `-XX:MaxGCPauseMillis` | 200 | Meta de pausa máxima (em ms). O G1 ajusta o tamanho da Young Gen para atingir essa meta. |
| `-XX:G1HeapRegionSize` | auto | Tamanho de cada região (1 MB a 32 MB, potência de 2). Regiões maiores beneficiam objetos grandes. |
| `-XX:InitiatingHeapOccupancyPercent` | 45 | % de ocupação do heap que dispara o concurrent marking. Reduzir para iniciar o marking mais cedo. |
| `-XX:G1ReservePercent` | 10 | % do heap reservado como "buffer de segurança" para evitar Full GC durante evacuação. |
| `-XX:ParallelGCThreads` | auto | Threads para fases paralelas (stop-the-world). Padrão: número de CPUs (até 8) + 5/8 do restante. |
| `-XX:ConcGCThreads` | auto | Threads para fases concorrentes. Padrão: ParallelGCThreads / 4. |

**Quando usar G1:**

- Heaps de **4 GB a 16 GB** (funciona com heaps maiores, mas ZGC pode ser melhor).
- Aplicações que precisam de **pausas previsíveis** sem sacrificar muito throughput.
- Cenário **general-purpose**: APIs REST, aplicações web, microsserviços.

**Conjunto completo de flags de produção para G1:**

```bash
java \
  -server \
  -Xms4g -Xmx4g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:G1HeapRegionSize=16m \
  -XX:InitiatingHeapOccupancyPercent=45 \
  -XX:G1ReservePercent=10 \
  -XX:ParallelGCThreads=4 \
  -XX:ConcGCThreads=1 \
  -XX:+ParallelRefProcEnabled \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:+UseStringDeduplication \
  -Xlog:gc*:file=/var/log/app/gc.log:time,level,tags:filecount=5,filesize=100m \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/app/heapdump.hprof \
  -jar app.jar
```

### 16.2 ZGC — Ultra-Low Latency

O ZGC (Z Garbage Collector) é um coletor de baixíssima latência, projetado para manter pausas abaixo de **1 milissegundo**, independentemente do tamanho do heap. Está em produção desde o Java 15 e recebeu suporte a gerações (ZGenerational) no Java 21.

**Como o ZGC funciona:**

O ZGC realiza quase todo o trabalho de coleta **concorrentemente** com a aplicação, usando duas técnicas fundamentais:

1. **Colored Pointers (Ponteiros Coloridos):** o ZGC armazena metadados de GC diretamente nos bits não utilizados dos ponteiros de 64 bits. Cada ponteiro carrega informações sobre o estado do objeto (marcado, remapeado, etc.), eliminando a necessidade de estruturas de dados auxiliares.

2. **Load Barriers (Barreiras de Leitura):** quando a aplicação lê uma referência de objeto, uma barreira de leitura verifica se o ponteiro está atualizado. Se o objeto foi movido pelo GC, a barreira corrige o ponteiro transparentemente. Isso permite que o GC mova objetos sem parar a aplicação.

```
Ponteiro de 64 bits com cores (ZGC):

  +--+--+--+--+----------------------------------------------+
  |M0|M1|Rm|Fn|            Endereço do Objeto (42 bits)       |
  +--+--+--+--+----------------------------------------------+
   |  |  |  |
   |  |  |  +-- Finalizable: objeto tem finalizador pendente
   |  |  +----- Remapped: ponteiro já foi atualizado
   |  +-------- Marked 1: marcado na fase de marcação atual
   +----------- Marked 0: marcado na fase de marcação anterior

O ZGC alterna entre M0 e M1 a cada ciclo de GC.
```

**Habilitando o ZGC:**

```bash
# Java 17 (não-geracional)
java -XX:+UseZGC -jar app.jar

# Java 21+ (geracional — recomendado)
java -XX:+UseZGC -XX:+ZGenerational -jar app.jar

# Java 23+ (geracional é o padrão, não-geracional requer flag explícita)
java -XX:+UseZGC -jar app.jar
```

> **Importante:** A partir do Java 21, sempre use `-XX:+ZGenerational`. O ZGC geracional oferece melhor throughput e menor uso de memória porque coleta objetos de curta duração com mais frequência, sem precisar escanear todo o heap.

**Flags essenciais para ZGC:**

```bash
java \
  -XX:+UseZGC \
  -XX:+ZGenerational \
  -XX:SoftMaxHeapSize=6g \
  -Xms8g -Xmx8g \
  -jar app.jar
```

| Flag | Padrão | Descrição |
|------|--------|-----------|
| `-XX:SoftMaxHeapSize` | (não definido) | Limite "suave" de heap. O ZGC tenta manter o uso abaixo desse valor, mas pode ultrapassá-lo se necessário. Útil para deixar memória disponível para direct memory e cache do SO. |
| `-XX:ZCollectionInterval` | 0 (desabilitado) | Intervalo máximo em segundos entre coletas. Útil para forçar coletas periódicas em aplicações com carga variável. |
| `-XX:ZFragmentationLimit` | 25 | % de fragmentação que dispara compactação. |
| `-XX:ZUncommitDelay` | 300 | Segundos antes de devolver memória não utilizada ao SO. |

**Quando usar ZGC:**

- Aplicações **sensíveis a latência**: trading, jogos, APIs com SLA rigoroso de p99.
- Heaps **grandes** (16 GB a terabytes) onde pausas de G1 se tornam problemáticas.
- Cenários onde pausas de **menos de 1 ms** são necessárias.

**Conjunto completo de flags de produção para ZGC:**

```bash
java \
  -server \
  -Xms16g -Xmx16g \
  -XX:+UseZGC \
  -XX:+ZGenerational \
  -XX:SoftMaxHeapSize=12g \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:ConcGCThreads=2 \
  -Xlog:gc*:file=/var/log/app/gc.log:time,level,tags:filecount=5,filesize=100m \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/app/heapdump.hprof \
  -jar app.jar
```

### 16.3 Shenandoah — Low Latency para OpenJDK

O Shenandoah é um coletor de baixa latência desenvolvido pela Red Hat, disponível no OpenJDK (mas **não no Oracle JDK**). Assim como o ZGC, ele realiza compactação concorrente para minimizar pausas.

**Diferenças em relação ao ZGC:**

| Aspecto | ZGC | Shenandoah |
|---------|-----|------------|
| Mecanismo de concorrência | Colored pointers + load barriers | Brooks pointers + read/write barriers |
| Overhead de memória | ~3-5% (metadados nos ponteiros) | ~5-10% (forwarding pointer por objeto) |
| Disponibilidade | Oracle JDK + OpenJDK | Apenas OpenJDK |
| Suporte a gerações | Sim (Java 21+) | Experimental |
| Maturidade | Produção desde Java 15 | Produção desde Java 12 (OpenJDK) |
| Heaps muito grandes (TB) | Excelente | Bom |

**Habilitando o Shenandoah:**

```bash
# Requer OpenJDK (AdoptOpenJDK, Amazon Corretto, Red Hat, etc.)
java -XX:+UseShenandoahGC -jar app.jar

# Com flags de tuning
java \
  -XX:+UseShenandoahGC \
  -XX:ShenandoahGCHeuristics=adaptive \
  -XX:+AlwaysPreTouch \
  -XX:+DisableExplicitGC \
  -jar app.jar
```

**Quando usar Shenandoah:**

- Já utiliza OpenJDK e precisa de **baixa latência** sem migrar para Oracle JDK.
- Heaps de **4 GB a 64 GB** com requisitos de pausa similares ao ZGC.
- Alternativa ao ZGC quando a distribuição OpenJDK utilizada não inclui ZGC.

### 16.4 Comparação entre Coletores

| Característica | Serial | Parallel | G1 | ZGC | Shenandoah |
|---------------|--------|----------|----|----|------------|
| **Pausa alvo** | N/A (simples) | Alto throughput | < 200 ms | < 1 ms | < 10 ms |
| **Heap recomendado** | < 256 MB | 1-8 GB | 4-16 GB | 16 GB - TB | 4-64 GB |
| **Throughput** | Bom (single-thread) | Excelente | Bom | Bom (geracional) | Bom |
| **Overhead de CPU** | Mínimo | Moderado (picos) | Moderado | Alto (concorrência) | Alto (concorrência) |
| **Maturidade** | Desde Java 1.0 | Desde Java 1.4 | Desde Java 9 (padrão) | Produção desde Java 15 | Produção desde Java 12 |
| **Concorrente** | Não | Não | Parcialmente | Quase totalmente | Quase totalmente |
| **Compactação** | Stop-the-world | Stop-the-world | Stop-the-world | Concorrente | Concorrente |
| **Caso de uso** | Apps CLI, containers mínimos | Batch, ETL, cálculos | General-purpose | Low-latency, heaps grandes | Low-latency (OpenJDK) |
| **Flag** | `-XX:+UseSerialGC` | `-XX:+UseParallelGC` | `-XX:+UseG1GC` | `-XX:+UseZGC` | `-XX:+UseShenandoahGC` |

### 16.5 Como Escolher o GC

Use o fluxograma abaixo para guiar a escolha:

```
                    +---------------------------+
                    |   Qual é a prioridade?    |
                    +---------------------------+
                       /          |          \
                      /           |           \
                     v            v            v
              +-----------+ +-----------+ +------------------+
              | Throughput| | Latência  | | Mínimo recurso   |
              | máximo    | | mínima    | | (container tiny) |
              +-----------+ +-----------+ +------------------+
                   |              |                |
                   v              v                v
              +-----------+  +--------+     +----------+
              | Parallel  |  | Heap   |     | Serial   |
              | GC        |  | size?  |     | GC       |
              +-----------+  +--------+     +----------+
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
              +-----------+          +-------------+
              | < 16 GB   |          | >= 16 GB    |
              +-----------+          +-------------+
                    |                       |
                    v                       v
              +-----------+          +-------------+
              | G1GC      |          | Oracle JDK? |
              | (padrão)  |          +-------------+
              +-----------+            /         \
                                      /           \
                                     v             v
                               +---------+   +------------+
                               | Sim     |   | Não        |
                               | -> ZGC  |   | -> ZGC ou  |
                               +---------+   | Shenandoah |
                                              +------------+

Notas:
- Para a maioria das aplicações Spring Boot, o G1GC padrão é suficiente.
- Só mude de GC se tiver evidência (métricas) de que as pausas ou
  o throughput do GC atual são insatisfatórios.
- Teste sempre com carga representativa antes de mudar em produção.
```

> **Regra de ouro:** Não troque de GC prematuramente. Comece com G1 (o padrão), monitore as métricas de GC em produção, e só migre para ZGC ou Shenandoah se os dados mostrarem que as pausas do G1 estão impactando os SLAs da aplicação.

---

## 17. JVM Flags e Ergonomics

As flags da JVM controlam desde a alocação de memória até o comportamento do compilador JIT. Configurá-las corretamente é a diferença entre uma aplicação estável em produção e uma que sofre com pausas de GC, OutOfMemoryError ou degradação progressiva.

### 17.1 Flags de Memória

As flags de memória definem os limites de cada região:

```bash
java \
  -Xms4g \                           # Heap inicial
  -Xmx4g \                           # Heap máximo
  -XX:MetaspaceSize=256m \            # Threshold inicial para GC do Metaspace
  -XX:MaxMetaspaceSize=512m \         # Limite máximo do Metaspace
  -XX:MaxDirectMemorySize=512m \      # Limite de Direct Memory (ByteBuffers)
  -Xss512k \                         # Tamanho do stack por thread
  -jar app.jar
```

| Flag | Descrição | Valor recomendado |
|------|-----------|-------------------|
| `-Xms` | Tamanho **inicial** do heap. A JVM aloca esta quantidade ao iniciar. | Igual ao `-Xmx` em produção |
| `-Xmx` | Tamanho **máximo** do heap. A JVM nunca alocará mais que isso. | 50-75% da memória do container |
| `-XX:MetaspaceSize` | Threshold que dispara o primeiro GC do Metaspace. Não é o tamanho inicial. | 256m para apps Spring Boot |
| `-XX:MaxMetaspaceSize` | Limite máximo do Metaspace. Sem esse limite, pode consumir toda a memória nativa. | 512m (ajustar conforme necessidade) |
| `-XX:MaxDirectMemorySize` | Limite de memória alocada fora do heap (NIO, Netty). Padrão: igual ao `-Xmx`. | Definir explicitamente para evitar surpresas |
| `-Xss` | Tamanho do stack por thread. Mais threads = mais memória total de stacks. | 512k (padrão: 1m; reduzir se muitas threads) |

**Por que `-Xms` deve ser igual a `-Xmx` em produção:**

```
Quando Xms < Xmx:
1. A JVM inicia com heap menor (Xms)
2. Conforme a aplicação aloca memória, o heap cresce
3. O crescimento do heap causa pausas adicionais (o SO precisa alocar páginas)
4. Após Full GC, a JVM pode DIMINUIR o heap (se ergonomics decidir)
5. Esse ciclo de crescer/encolher causa instabilidade nas pausas de GC

Quando Xms = Xmx:
1. A JVM aloca todo o heap no startup
2. O tamanho do heap nunca muda
3. Pausas de GC são mais previsíveis
4. Com -XX:+AlwaysPreTouch, as páginas são "tocadas" no startup,
   evitando page faults durante a execução
```

### 17.2 JVM Ergonomics

A JVM possui um sistema de **ergonomics** que detecta automaticamente as características do hardware e ajusta configurações como GC, tamanho de heap e número de threads. Esse mecanismo funciona bem em servidores bare-metal, mas pode causar problemas em containers.

**O que o ergonomics detecta automaticamente:**

| Configuração | Regra automática |
|-------------|-----------------|
| GC padrão | G1 se >= 2 CPUs e >= 1792 MB de RAM; senão Serial |
| `-Xmx` | 1/4 da memória física (máximo ~32 GB com compressed oops) |
| `-Xms` | 1/64 da memória física |
| `ParallelGCThreads` | Número de CPUs (até 8) + 5/8 das CPUs excedentes |
| `ConcGCThreads` | ParallelGCThreads / 4 |

**O problema em containers (antes do Java 10):**

Antes do Java 10, a JVM ignorava os limites de cgroup do container e enxergava os recursos do **host físico**:

```
Host: 64 GB RAM, 16 CPUs
Container: docker run --memory=2g --cpus=2

Sem UseContainerSupport (Java 8u131 a Java 9):
  JVM vê: 64 GB RAM, 16 CPUs
  Xmx automático: 64 GB / 4 = 16 GB (!!!)
  ParallelGCThreads: 13 (!!!)
  -> Container é OOMKilled imediatamente

Com UseContainerSupport (Java 10+, habilitado por padrão):
  JVM vê: 2 GB RAM, 2 CPUs
  Xmx automático: 2 GB / 4 = 512 MB
  ParallelGCThreads: 2
  -> Comportamento correto
```

**Flags de container support:**

```bash
# Java 10+ (habilitado por padrão, não precisa definir)
-XX:+UseContainerSupport

# Se precisar forçar o número de CPUs visíveis
-XX:ActiveProcessorCount=2

# Alternativa ao Xmx: definir como % da memória do container
-XX:MaxRAMPercentage=75.0
-XX:InitialRAMPercentage=75.0
-XX:MinRAMPercentage=50.0    # usado quando a memória total é < 256 MB
```

> **Observação:** Se a aplicação roda em Java 8, atualize para pelo menos **Java 8u191**, que incluiu o suporte a containers via `-XX:+UseContainerSupport`. Versões anteriores do Java 8 (de u131 a u190) suportavam apenas as flags experimentais `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`, que foram removidas posteriormente.

### 17.3 JIT Compiler Flags

O compilador Just-In-Time (JIT) da JVM compila bytecode para código nativo em tempo de execução. O sistema de compilação em camadas (tiered compilation) usa dois compiladores:

- **C1 (Client):** compilação rápida com otimizações leves. Usado para código executado poucas vezes.
- **C2 (Server):** compilação lenta com otimizações agressivas. Usado para "hot methods" (métodos executados milhares de vezes).

```bash
java \
  -XX:+TieredCompilation \
  -XX:ReservedCodeCacheSize=512m \
  -XX:CompileThreshold=5000 \
  -jar app.jar
```

| Flag | Padrão | Descrição |
|------|--------|-----------|
| `-XX:+TieredCompilation` | true (Java 8+) | Habilita compilação em camadas (C1 -> C2). Desabilitar para reduzir uso de memória em apps de curta duração. |
| `-XX:ReservedCodeCacheSize` | 240 MB (tiered) | Tamanho máximo do cache de código compilado. Se esgotar, a JVM para de compilar e o desempenho degrada silenciosamente. |
| `-XX:CompileThreshold` | 10000 (C2) | Número de invocações/iterações antes de compilar um método com C2. Valores menores = warm-up mais rápido, mas mais CPU no startup. |
| `-XX:-TieredCompilation` | — | Desabilita tiered; útil para ferramentas CLI que executam por poucos segundos. |

**Monitorando a compilação JIT:**

```bash
# Ver quais métodos estão sendo compilados
java -XX:+PrintCompilation -jar app.jar

# Saída:
#   123   1       3       java.lang.String::hashCode (55 bytes)
#   125   2       4       java.lang.String::equals (81 bytes)
#              tier (1-4)
#              3 = C1 full
#              4 = C2

# Verificar se o code cache está cheio
jcmd <PID> Compiler.codecache
```

> **Dica:** Se os logs mostram `CodeCache is full. Compiler has been disabled`, aumente o `-XX:ReservedCodeCacheSize`. Em aplicações Spring Boot com muitas dependências, 512 MB é um valor seguro.

### 17.4 Diagnostic Flags

Flags de diagnóstico ajudam a investigar problemas em produção sem interromper a aplicação:

```bash
java \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/app/heapdump.hprof \
  -XX:+ExitOnOutOfMemoryError \
  -XX:NativeMemoryTracking=summary \
  -XX:+UnlockDiagnosticVMOptions \
  -XX:+DebugNonSafepoints \
  -jar app.jar
```

| Flag | Descrição |
|------|-----------|
| `-XX:+HeapDumpOnOutOfMemoryError` | Gera um heap dump automaticamente quando ocorre `OutOfMemoryError`. Essencial para diagnóstico post-mortem. |
| `-XX:HeapDumpPath=<caminho>` | Diretório ou arquivo onde o heap dump será salvo. Certifique-se de que há espaço em disco (o dump tem o tamanho do heap). |
| `-XX:+ExitOnOutOfMemoryError` | Encerra a JVM imediatamente após `OutOfMemoryError`. Evita que a aplicação fique em estado inconsistente. Em containers, o orquestrador (Kubernetes) reinicia o pod. |
| `-XX:+CrashOnOutOfMemoryError` | Similar ao anterior, mas gera um crash dump adicional com informações da JVM. |
| `-XX:NativeMemoryTracking=summary` | Rastreia alocações de memória nativa da JVM. Útil para diagnosticar memory leaks fora do heap. Use `jcmd <PID> VM.native_memory summary` para consultar. |
| `-XX:+UnlockDiagnosticVMOptions` | Habilita flags de diagnóstico avançadas (necessário para algumas flags). |
| `-XX:+DebugNonSafepoints` | Melhora a precisão de profilers (async-profiler, JFR) ao registrar informação de debug em pontos além dos safepoints. |

**Consultando Native Memory Tracking:**

```bash
# Habilitar no startup
java -XX:NativeMemoryTracking=summary -jar app.jar

# Consultar em tempo real
jcmd <PID> VM.native_memory summary

# Saída:
# Total: reserved=4567MB, committed=2345MB
#   - Java Heap:    reserved=2048MB, committed=2048MB
#   - Class:        reserved=1100MB, committed=156MB     # Metaspace
#   - Thread:       reserved=256MB,  committed=256MB      # Stacks
#   - Code:         reserved=250MB,  committed=89MB       # Code Cache
#   - GC:           reserved=198MB,  committed=198MB      # Estruturas do GC
#   - Internal:     reserved=12MB,   committed=12MB
#   - Symbol:       reserved=24MB,   committed=24MB       # String table
#   - Direct:       reserved=128MB,  committed=128MB      # Direct ByteBuffers

# Comparar com baseline (detectar leaks)
jcmd <PID> VM.native_memory baseline
# ... aguardar algum tempo ...
jcmd <PID> VM.native_memory summary.diff
```

### 17.5 Script de Produção Completo

Os scripts abaixo consolidam todas as flags recomendadas em configurações prontas para produção.

**Script shell para aplicação com G1GC:**

```bash
#!/bin/bash
# run-g1.sh — Script de produção com G1GC
# Uso: ./run-g1.sh [heap_size] [app_jar]
#   Exemplo: ./run-g1.sh 4g myapp.jar

HEAP_SIZE="${1:-4g}"
APP_JAR="${2:-app.jar}"
LOG_DIR="/var/log/app"

mkdir -p "${LOG_DIR}"

exec java \
  -server \
  \
  # === Memória === \
  -Xms${HEAP_SIZE} -Xmx${HEAP_SIZE} \
  -XX:MetaspaceSize=256m \
  -XX:MaxMetaspaceSize=512m \
  -XX:MaxDirectMemorySize=256m \
  -Xss512k \
  \
  # === Garbage Collector (G1) === \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:G1HeapRegionSize=16m \
  -XX:InitiatingHeapOccupancyPercent=45 \
  -XX:+ParallelRefProcEnabled \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:+UseStringDeduplication \
  \
  # === JIT === \
  -XX:+TieredCompilation \
  -XX:ReservedCodeCacheSize=512m \
  \
  # === GC Logging === \
  -Xlog:gc*,gc+phases=debug:file=${LOG_DIR}/gc.log:time,uptime,level,tags:filecount=5,filesize=100m \
  \
  # === Diagnóstico === \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=${LOG_DIR}/heapdump.hprof \
  -XX:+ExitOnOutOfMemoryError \
  -XX:NativeMemoryTracking=summary \
  \
  -jar "${APP_JAR}"
```

**Script shell para aplicação com ZGC:**

```bash
#!/bin/bash
# run-zgc.sh — Script de produção com ZGC (Java 21+)
# Uso: ./run-zgc.sh [heap_size] [app_jar]

HEAP_SIZE="${1:-16g}"
SOFT_MAX="${2:-12g}"
APP_JAR="${3:-app.jar}"
LOG_DIR="/var/log/app"

mkdir -p "${LOG_DIR}"

exec java \
  -server \
  \
  # === Memória === \
  -Xms${HEAP_SIZE} -Xmx${HEAP_SIZE} \
  -XX:MetaspaceSize=256m \
  -XX:MaxMetaspaceSize=512m \
  -XX:MaxDirectMemorySize=512m \
  -Xss512k \
  \
  # === Garbage Collector (ZGC Generational) === \
  -XX:+UseZGC \
  -XX:+ZGenerational \
  -XX:SoftMaxHeapSize=${SOFT_MAX} \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:ConcGCThreads=2 \
  \
  # === JIT === \
  -XX:+TieredCompilation \
  -XX:ReservedCodeCacheSize=512m \
  \
  # === GC Logging === \
  -Xlog:gc*:file=${LOG_DIR}/gc.log:time,uptime,level,tags:filecount=5,filesize=100m \
  \
  # === Diagnóstico === \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=${LOG_DIR}/heapdump.hprof \
  -XX:+ExitOnOutOfMemoryError \
  -XX:NativeMemoryTracking=summary \
  \
  -jar "${APP_JAR}"
```

**Dockerfile ENTRYPOINT com G1GC:**

```dockerfile
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S app && adduser -S app -G app
WORKDIR /app

COPY target/*.jar app.jar
RUN mkdir -p /var/log/app && chown app:app /var/log/app

USER app

ENTRYPOINT ["java", \
  "-server", \
  "-Xms2g", "-Xmx2g", \
  "-XX:+UseG1GC", \
  "-XX:MaxGCPauseMillis=200", \
  "-XX:+ParallelRefProcEnabled", \
  "-XX:+DisableExplicitGC", \
  "-XX:+AlwaysPreTouch", \
  "-XX:+UseStringDeduplication", \
  "-XX:MetaspaceSize=128m", \
  "-XX:MaxMetaspaceSize=256m", \
  "-Xlog:gc*:stdout:time,level,tags", \
  "-XX:+HeapDumpOnOutOfMemoryError", \
  "-XX:HeapDumpPath=/var/log/app/heapdump.hprof", \
  "-XX:+ExitOnOutOfMemoryError", \
  "-jar", "app.jar"]
```

**Dockerfile ENTRYPOINT com ZGC (Java 21+):**

```dockerfile
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S app && adduser -S app -G app
WORKDIR /app

COPY target/*.jar app.jar
RUN mkdir -p /var/log/app && chown app:app /var/log/app

USER app

ENV JAVA_OPTS_HEAP="-Xms4g -Xmx4g -XX:SoftMaxHeapSize=3g"
ENV JAVA_OPTS_GC="-XX:+UseZGC -XX:+ZGenerational -XX:+DisableExplicitGC -XX:+AlwaysPreTouch"
ENV JAVA_OPTS_META="-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m"
ENV JAVA_OPTS_LOG="-Xlog:gc*:stdout:time,level,tags"
ENV JAVA_OPTS_DIAG="-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/app/heapdump.hprof -XX:+ExitOnOutOfMemoryError"

ENTRYPOINT ["sh", "-c", \
  "exec java -server $JAVA_OPTS_HEAP $JAVA_OPTS_GC $JAVA_OPTS_META $JAVA_OPTS_LOG $JAVA_OPTS_DIAG -jar app.jar"]
```

> **Dica:** No Dockerfile com ZGC, o uso de variáveis de ambiente permite sobrescrever flags sem reconstruir a imagem: `docker run -e JAVA_OPTS_HEAP="-Xms8g -Xmx8g -XX:SoftMaxHeapSize=6g" myapp:latest`.

---

## 18. JVM em Containers — Limites de CPU e Memória

A execução da JVM em containers Docker e Kubernetes traz desafios específicos de dimensionamento de recursos. A JVM precisa operar dentro dos limites do container sem ser encerrada pelo sistema operacional, e o desenvolvedor precisa entender como a memória é distribuída entre as diversas regiões da JVM.

### 18.1 Como a JVM Vê os Recursos do Container

A partir do Java 10 (e backport para Java 8u191+), a JVM detecta automaticamente os limites de CPU e memória definidos pelo cgroup do container.

**CPU — `availableProcessors` vs cgroup:**

```java
// O que a aplicação enxerga
int cpus = Runtime.getRuntime().availableProcessors();

// Em um container com --cpus=2 em um host de 16 CPUs:
// Java 8 (sem container support): cpus = 16  (ERRADO)
// Java 10+:                       cpus = 2   (CORRETO)
```

O número de CPUs visíveis afeta diretamente:

- **ParallelGCThreads** e **ConcGCThreads** (threads do GC);
- **ForkJoinPool.commonPool()** (tamanho do pool padrão para parallel streams);
- **Thread pools automáticos** de frameworks (Tomcat, Netty, etc.);
- **CompilerThreads** (threads do JIT).

**Sobrescrevendo o número de CPUs:**

```bash
# Forçar a JVM a enxergar N processadores (independente do cgroup)
java -XX:ActiveProcessorCount=4 -jar app.jar
```

Isso é útil quando:
- O container tem CPU limit fracionário (ex: `--cpus=1.5`) e a JVM arredonda para 1;
- O orquestrador define `cpu.shares` em vez de `cpu.quota`, e a JVM não consegue inferir o limite;
- É necessário controlar precisamente o número de threads de GC.

**Memória — como a JVM detecta o limite:**

```bash
# Container com 4 GB de memória
docker run --memory=4g myapp:latest

# A JVM (Java 10+) detecta:
# - Memória total disponível: 4 GB (do cgroup, não do host)
# - Xmx automático (se não definido): 4 GB / 4 = 1 GB
# - MaxRAMPercentage (se definido): 4 GB * 75% = 3 GB
```

### 18.2 Memória: JVM Heap vs Container Memory Limit

O erro mais comum ao rodar a JVM em containers é configurar o `-Xmx` igual ao limite de memória do container. O heap é apenas **uma parte** da memória total consumida pela JVM:

```
+-----------------------------------------------------------------------+
|                    MEMÓRIA DO CONTAINER (4 GB)                        |
|                                                                       |
|  +----------------------------------------------+                    |
|  |              JVM Heap (-Xmx)                  |  ~2.5 - 3 GB      |
|  |          (objetos Java, arrays, etc.)         |                    |
|  +----------------------------------------------+                    |
|                                                                       |
|  +------------------+  +-----------------------+                      |
|  | Metaspace        |  | Thread Stacks          |                     |
|  | ~128-256 MB      |  | threads * Xss          |                     |
|  | (classes, métodos)|  | ex: 200 * 512KB = 100MB|                    |
|  +------------------+  +-----------------------+                      |
|                                                                       |
|  +------------------+  +-----------------------+                      |
|  | Code Cache       |  | Direct Memory          |                     |
|  | ~64-250 MB       |  | (NIO, Netty buffers)   |                     |
|  | (JIT compiled)   |  | ~64-256 MB             |                     |
|  +------------------+  +-----------------------+                      |
|                                                                       |
|  +------------------+  +-----------------------+                      |
|  | GC Overhead      |  | Memória do SO           |                    |
|  | ~5-10% do heap   |  | (libc, /proc, etc.)    |                     |
|  | (card tables,    |  | ~50-100 MB             |                     |
|  |  remembered sets) |  |                        |                     |
|  +------------------+  +-----------------------+                      |
+-----------------------------------------------------------------------+
```

**Fórmula para calcular o `-Xmx`:**

```
Container Memory = Heap + Metaspace + Threads + CodeCache + DirectMemory + GC + SO

Exemplo com container de 4 GB:
  Metaspace:      256 MB
  Threads:        200 threads * 512 KB = 100 MB
  Code Cache:     250 MB
  Direct Memory:  128 MB
  GC Overhead:    ~200 MB (G1, com heap de 3 GB)
  SO/libc:        100 MB
  --------------------------
  Total não-heap:  ~1034 MB ≈ 1 GB

  Xmx = 4 GB - 1 GB = 3 GB (75% do container)
```

**Regra prática:**

```
Xmx = container_memory_limit * 0.75
```

Essa regra funciona para a maioria dos casos. Para aplicações que usam muita Direct Memory (Netty, Kafka), reduza para 60-65%.

**Usando `MaxRAMPercentage` como alternativa:**

```bash
# Em vez de definir Xmx absolutamente, definir como % da memória visível
java \
  -XX:MaxRAMPercentage=75.0 \
  -XX:InitialRAMPercentage=75.0 \
  -jar app.jar

# Vantagem: funciona com qualquer tamanho de container sem recalcular
# Container de 2 GB -> Xmx = 1.5 GB
# Container de 8 GB -> Xmx = 6 GB
```

| Abordagem | Vantagem | Desvantagem |
|-----------|----------|-------------|
| `-Xmx` fixo | Controle preciso, previsível | Precisa recalcular ao mudar o container |
| `-XX:MaxRAMPercentage` | Adaptável a qualquer tamanho | Menos controle; pode alocar demais se a app usar muita memória nativa |

### 18.3 OOMKilled — Diagnóstico e Prevenção

Existem dois tipos de "Out of Memory" em containers, e é crucial distingui-los:

**Java OOM vs Container OOMKilled:**

| Aspecto | Java `OutOfMemoryError` | Container OOMKilled |
|---------|------------------------|---------------------|
| **Quem detecta** | A JVM | O kernel do Linux (cgroup) |
| **Causa** | Heap cheio (não consegue alocar objeto) | Processo excedeu o memory limit do container |
| **Heap dump** | Sim (com `-XX:+HeapDumpOnOutOfMemoryError`) | Não (processo é morto com SIGKILL) |
| **Exit code** | Depende da aplicação | 137 (128 + 9 = SIGKILL) |
| **Mensagem** | `java.lang.OutOfMemoryError: Java heap space` | `OOMKilled` no Docker / Kubernetes |
| **Diagnóstico** | Analisar heap dump | Verificar métricas de memória do container |

**Diagnosticando OOMKilled:**

```bash
# Docker — verificar se o container foi OOMKilled
docker inspect <container_id> --format='{{.State.OOMKilled}}'
# true

# Docker — ver o exit code
docker inspect <container_id> --format='{{.State.ExitCode}}'
# 137

# Kubernetes — verificar o status do pod
kubectl describe pod <pod-name>
# Last State:  Terminated
#   Reason:    OOMKilled
#   Exit Code: 137

# Kubernetes — ver eventos
kubectl get events --field-selector involvedObject.name=<pod-name>

# Linux — verificar logs do kernel (dmesg)
dmesg | grep -i "oom\|killed"
# [12345.678] Memory cgroup out of memory: Killed process 1234 (java)
#             total-vm:5242880kB, anon-rss:4194304kB, file-rss:12345kB
```

**Cenários comuns de OOMKilled e soluções:**

```
Cenário 1: Xmx muito alto
  Container: 4 GB
  -Xmx4g -> Heap consome 4 GB, não sobra para Metaspace/threads/SO
  Solução: -Xmx3g (75% do container)

Cenário 2: Memory leak nativa (Direct Memory)
  Heap estável em 2 GB, mas container cresce até 4 GB
  Causa: ByteBuffers não liberados (Netty, NIO)
  Solução: -XX:MaxDirectMemorySize=256m + investigar leak

Cenário 3: Muitas threads
  200 threads * 1 MB (Xss padrão) = 200 MB fora do heap
  Solução: -Xss512k (se a aplicação não usa recursão profunda)

Cenário 4: Metaspace descontrolado
  Frameworks com geração dinâmica de classes (Groovy, proxies CGLIB)
  Solução: -XX:MaxMetaspaceSize=256m
```

**Dockerfile com configuração completa para evitar OOMKilled:**

```dockerfile
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S app && adduser -S app -G app
WORKDIR /app

COPY target/*.jar app.jar
RUN mkdir -p /var/log/app && chown app:app /var/log/app

USER app

# Configuração segura para container de 4 GB
ENTRYPOINT ["java", \
  "-server", \
  \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:InitialRAMPercentage=75.0", \
  \
  "-XX:MetaspaceSize=128m", \
  "-XX:MaxMetaspaceSize=256m", \
  "-XX:MaxDirectMemorySize=256m", \
  "-Xss512k", \
  \
  "-XX:+UseG1GC", \
  "-XX:MaxGCPauseMillis=200", \
  "-XX:+AlwaysPreTouch", \
  "-XX:+DisableExplicitGC", \
  \
  "-Xlog:gc*:stdout:time,level,tags", \
  \
  "-XX:+HeapDumpOnOutOfMemoryError", \
  "-XX:HeapDumpPath=/var/log/app/heapdump.hprof", \
  "-XX:+ExitOnOutOfMemoryError", \
  "-XX:NativeMemoryTracking=summary", \
  \
  "-jar", "app.jar"]
```

**Kubernetes deployment com limites corretos:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        image: myapp:latest
        resources:
          requests:
            memory: "3Gi"      # Request = o mínimo garantido
            cpu: "1000m"
          limits:
            memory: "4Gi"      # Limit = máximo antes de OOMKill
            cpu: "2000m"
        env:
        - name: JAVA_OPTS
          value: >-
            -Xms3g -Xmx3g
            -XX:MaxMetaspaceSize=256m
            -XX:MaxDirectMemorySize=256m
            -XX:+UseG1GC
            -XX:+ExitOnOutOfMemoryError
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
          initialDelaySeconds: 15
          periodSeconds: 5
```

**Checklist de prevenção de OOMKilled:**

| Verificação | Como verificar |
|------------|---------------|
| `-Xmx` <= 75% do memory limit | Comparar valores no Dockerfile/YAML |
| `-XX:MaxMetaspaceSize` definido | Verificar nas flags da JVM |
| `-XX:MaxDirectMemorySize` definido | Verificar nas flags da JVM |
| `-Xss` reduzido se muitas threads | Verificar com `jcmd <PID> Thread.print` |
| `-XX:+ExitOnOutOfMemoryError` ativo | Verificar nas flags da JVM |
| `-XX:NativeMemoryTracking=summary` ativo | Consultar com `jcmd <PID> VM.native_memory` |
| Monitoramento de memória do container | Grafana + Prometheus com `container_memory_usage_bytes` |

> Para detalhes sobre Docker, Docker Compose e configuração de containers Kubernetes, consulte [Docker-Compose-Kubernetes.md](outros/Docker-Compose-Kubernetes.md).

---

**Parte 4 — Configuração do Sistema Operacional**

## 19. Linux Kernel Tuning — Ubuntu e Alpine

Aplicações Spring Boot em produção dependem diretamente de parâmetros do sistema operacional para atingir desempenho e estabilidade adequados. Um kernel mal configurado pode causar recusa de conexões, latência em I/O e swapping excessivo — problemas que nenhuma otimização de JVM resolve sozinha.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Camadas de Tuning                                │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │           Aplicação (Spring Boot / JVM Flags)                 │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │           JVM (Heap, GC, JIT)                                 │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │           Sistema Operacional (kernel params, limites)        │  │ ◄── Esta seção
│  ├───────────────────────────────────────────────────────────────┤  │
│  │           Hardware / Hypervisor / Container Runtime           │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 19.1 Limites de File Descriptors

Cada conexão TCP, arquivo aberto, socket Unix e pipe consome um **file descriptor (FD)**. Uma aplicação Spring Boot com pool de conexões de banco, conexões HTTP de clientes e comunicação com serviços externos pode facilmente consumir milhares de FDs simultaneamente.

**Cálculo aproximado de FDs para uma aplicação Spring Boot:**

```
FDs necessários ≈ conexões_clientes
                 + pool_conexões_banco (HikariCP: default 10)
                 + pool_conexões_redis
                 + conexões_HTTP_saída (WebClient/RestClient)
                 + arquivos_log
                 + JARs_carregados (~50-100)
                 + overhead_JVM (~200-300)

Exemplo: 2000 clientes + 20 DB + 10 Redis + 50 HTTP + 5 logs + 100 JARs + 300 JVM
       = ~2485 FDs mínimos
```

**Verificando o limite atual:**

```bash
# Limite do processo atual
ulimit -n

# Limites soft e hard separadamente
ulimit -Sn   # soft limit (limite efetivo)
ulimit -Hn   # hard limit (teto máximo)

# Limites de um processo em execução (PID da JVM)
cat /proc/$(pgrep -f "spring-boot")/limits | grep "open files"
```

#### Configuração no Ubuntu (22.04 / 24.04)

```bash
# /etc/security/limits.conf
# <domain>  <type>  <item>   <value>
*            soft    nofile   65536
*            hard    nofile   65536
root         soft    nofile   65536
root         hard    nofile   65536

# Usuário específico da aplicação (recomendado em produção)
spring-app   soft    nofile   65536
spring-app   hard    nofile   65536
```

```bash
# /etc/pam.d/common-session — garantir que o PAM carregue os limites
session required pam_limits.so

# /etc/sysctl.conf — limite global do sistema
fs.file-max = 2097152
fs.nr_open  = 1048576
```

```bash
# Para serviços gerenciados pelo systemd, adicionar no unit file:
# /etc/systemd/system/spring-boot-app.service
[Service]
LimitNOFILE=65536
LimitNPROC=4096
```

```bash
# Aplicar sem reiniciar
sudo sysctl -p
sudo systemctl daemon-reload
```

#### Configuração no Alpine Linux

O Alpine usa **BusyBox** e **OpenRC** em vez de systemd, portanto a configuração difere:

```bash
# /etc/conf.d/spring-boot-app (arquivo de configuração do serviço OpenRC)
rc_ulimit="-n 65536"

# Ou diretamente no script de inicialização /etc/init.d/spring-boot-app
start() {
    ulimit -n 65536
    ebegin "Starting Spring Boot Application"
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/spring-boot.pid \
        --exec /usr/bin/java -- -jar /opt/app/app.jar
    eend $?
}
```

```bash
# /etc/sysctl.conf (funciona igual ao Ubuntu)
fs.file-max = 2097152
```

> **Importante:** Em containers Docker, o limite de FDs é herdado do host. Use `--ulimit nofile=65536:65536` no `docker run` ou configure no `docker-compose.yml`.

### 19.2 TCP Tuning

O stack TCP do Linux possui dezenas de parâmetros ajustáveis. Para aplicações Spring Boot que recebem muitas conexões simultâneas, os mais impactantes são o tamanho da fila de backlog, reutilização de portas e timeouts.

```
┌──────────────────────────────────────────────────────────────────┐
│                 Fluxo de Conexão TCP                             │
│                                                                  │
│  Cliente ──SYN──► ┌─────────────┐                                │
│                   │  SYN Queue   │ ◄── net.ipv4.tcp_max_syn_backlog
│  Cliente ◄─SYN+ACK─┤  (half-open) │                              │
│                   └──────┬──────┘                                │
│  Cliente ──ACK──►        ▼                                       │
│                   ┌─────────────┐                                │
│                   │ Accept Queue │ ◄── net.core.somaxconn         │
│                   │  (completed) │                                │
│                   └──────┬──────┘                                │
│                          ▼                                       │
│                   Application accept()                           │
│                   (Tomcat/Netty thread)                           │
└──────────────────────────────────────────────────────────────────┘
```

#### Configuração completa — /etc/sysctl.d/99-spring-boot-tcp.conf

```bash
# =============================================================
# TCP Tuning para aplicações Spring Boot em produção
# =============================================================

# --- Backlog e filas de conexão ---
# Tamanho máximo da fila de conexões aceitas (accept queue).
# Default: 4096. Para aplicações com muitas conexões simultâneas, aumentar.
# Deve ser >= server.tomcat.accept-count no Spring Boot.
net.core.somaxconn = 8192

# Tamanho máximo da fila de half-open connections (SYN recebido, esperando ACK).
net.ipv4.tcp_max_syn_backlog = 8192

# --- Reutilização de portas ---
# Permitir reutilização de sockets em TIME_WAIT para novas conexões.
# Essencial em servidores que abrem/fecham muitas conexões (microservices).
net.ipv4.tcp_tw_reuse = 1

# Tempo (segundos) que o socket fica em FIN_WAIT_2 antes de ser liberado.
# Default: 60. Reduzir libera portas mais rapidamente.
net.ipv4.tcp_fin_timeout = 15

# --- Keep-alive ---
# Tempo (segundos) antes de enviar o primeiro probe de keep-alive.
# Default: 7200 (2 horas). Reduzir para detectar conexões mortas mais rápido.
net.ipv4.tcp_keepalive_time = 300

# Intervalo (segundos) entre probes de keep-alive.
# Default: 75.
net.ipv4.tcp_keepalive_intvl = 30

# Número de probes antes de considerar a conexão morta.
# Default: 9.
net.ipv4.tcp_keepalive_probes = 5

# --- Buffers de rede ---
# Tamanho máximo do buffer de recebimento por socket (bytes).
net.core.rmem_max = 16777216

# Tamanho máximo do buffer de envio por socket (bytes).
net.core.wmem_max = 16777216

# Buffers TCP: min, default, max (bytes).
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# --- Fila de pacotes ---
# Comprimento da fila de pacotes que aguardam processamento.
# Default: 1000. Aumentar para tráfego intenso.
net.core.netdev_max_backlog = 5000
```

```bash
# Aplicar configurações
sudo sysctl --system
# Ou especificamente:
sudo sysctl -p /etc/sysctl.d/99-spring-boot-tcp.conf
```

**Alinhamento com Spring Boot — application.yml:**

```yaml
server:
  tomcat:
    # Deve ser <= net.core.somaxconn do kernel
    accept-count: 4096
    # Máximo de conexões simultâneas
    max-connections: 8192
    threads:
      max: 200
      min-spare: 20
  # Keep-alive no nível HTTP
  connection-timeout: 20000
```

### 19.3 Memory Tuning

A JVM possui seu próprio gerenciamento de memória, mas o kernel Linux ainda interfere por meio de swapping, alocação de páginas e políticas de overcommit.

#### Swappiness

```bash
# vm.swappiness controla a agressividade do kernel em mover páginas para swap.
# Escala: 0 (evita swap ao máximo) a 100 (swap agressivo).
# Para aplicações Java: usar valor baixo (10).
# Razão: a JVM gerencia sua própria memória com GC — swap de heap causa
# stop-the-world extremamente longos e imprevisíveis.

# /etc/sysctl.d/99-spring-boot-memory.conf
vm.swappiness = 10
```

**Por que `vm.swappiness = 10` e nao `0`?**

| Valor | Comportamento | Uso |
|-------|---------------|-----|
| `0` | Swap apenas se memória completamente esgotada | Risco de OOM Killer |
| `10` | Swap mínimo, mantém margem de segurança | **Recomendado para JVM** |
| `60` | Comportamento padrão do Linux | Ambientes genéricos |
| `100` | Swap agressivo | Nunca para Java |

#### Max Map Count

```bash
# Número máximo de regiões de mapeamento de memória (mmap) por processo.
# A JVM usa mmap extensivamente para: class loading, JIT compiled code,
# memory-mapped files, NIO buffers.
# Default: 65530. Insuficiente para aplicações grandes.
vm.max_map_count = 262144
```

#### Transparent Huge Pages (THP)

Transparent Huge Pages é um recurso do kernel que automaticamente agrupa páginas de 4KB em páginas de 2MB. Embora beneficie algumas workloads, causa problemas sérios para a JVM:

- **Latência de alocação:** a compactação de páginas pode causar stalls de dezenas de milissegundos
- **Fragmentação:** o GC da JVM aloca/libera memória em padrões que conflitam com THP
- **GC pause spikes:** medições mostram aumento de 10-30% nas pausas do G1GC com THP habilitado

```bash
# Verificar estado atual
cat /sys/kernel/mm/transparent_hugepage/enabled
# Saída: [always] madvise never
#         ^^^^^^^^ se "always" estiver selecionado, desabilitar

# Desabilitar THP em runtime
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# Persistir a configuração — Ubuntu (via GRUB)
# /etc/default/grub
GRUB_CMDLINE_LINUX="transparent_hugepage=never"
# Depois: sudo update-grub && sudo reboot

# Persistir a configuração — Alpine (via sysfs no boot)
# /etc/local.d/thp-disable.start
#!/bin/sh
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
# chmod +x /etc/local.d/thp-disable.start
# rc-update add local default
```

#### Configuração completa de memória

```bash
# /etc/sysctl.d/99-spring-boot-memory.conf

# Swappiness baixo para JVM
vm.swappiness = 10

# Mapeamentos de memória (mmap) suficientes para JVM + Elasticsearch/Lucene
vm.max_map_count = 262144

# Overcommit: 0 = heurístico (default), 1 = sempre permitir, 2 = nunca
# Para produção com JVM, manter o default (0) ou usar 2 com ratio adequado
vm.overcommit_memory = 0

# Porcentagem de memória que pode ser usada por dirty pages antes de
# forçar write-back. Reduzir para I/O mais previsível.
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
```

### 19.4 Diferenças entre Ubuntu e Alpine

A escolha da distribuição base impacta diretamente o comportamento da JVM, especialmente em ambientes containerizados.

#### Comparação Geral

| Aspecto | Ubuntu (Jammy/Noble) | Alpine Linux |
|---------|---------------------|--------------|
| **Biblioteca C** | glibc | musl libc |
| **Tamanho base** | ~78 MB (minimal) | ~7 MB |
| **Gerenciador de pacotes** | apt | apk |
| **Init system** | systemd | OpenRC |
| **Shell padrão** | bash | ash (BusyBox) |
| **Comunidade/suporte** | Muito ampla | Focada em containers |
| **Ciclo de releases** | LTS (5 anos) | ~6 meses, suporte limitado |

#### Impacto no JVM

| Aspecto | glibc (Ubuntu) | musl (Alpine) |
|---------|----------------|---------------|
| **Compatibilidade JVM** | Total — JVMs são testadas primariamente com glibc | Boa, mas com ressalvas |
| **Alocação de memória** | `malloc` otimizado com arenas por thread | `malloc` mais simples, single-threaded |
| **Desempenho de threads** | Thread-local storage otimizado | TLS mais lento em workloads com muitas threads |
| **DNS resolution** | nsswitch.conf completo, suporta mDNS, LDAP | Resolver próprio — **não suporta `search` e `ndots` do `/etc/resolv.conf` corretamente em versões antigas** |
| **Stack size default** | 8 MB | 128 KB — pode causar `StackOverflowError` |
| **Desempenho geral JVM** | Baseline (100%) | ~95-98% em benchmarks típicos |

#### Problemas conhecidos com Alpine/musl

**1. DNS Resolution em Kubernetes:**

```
# musl resolve DNS de forma diferente do glibc:
# - Não respeita "search" domains corretamente (versões < 1.2.4)
# - Pode falhar com "ndots:5" padrão do Kubernetes
# - Queries A e AAAA são serializadas (não paralelas como glibc)

# Sintoma: lentidão ou falha na resolução de nomes de serviços K8s
# Ex: "my-service.my-namespace.svc.cluster.local" falha, mas IP funciona

# Mitigação: usar FQDN completo ou ajustar ndots
# No deployment K8s:
spec:
  dnsConfig:
    options:
      - name: ndots
        value: "2"
```

**2. Stack Size:**

```bash
# Alpine usa stack de 128KB por default (musl).
# Java threads com recursão profunda podem falhar com StackOverflowError.
# Solução: definir -Xss explicitamente
java -Xss512k -jar app.jar
```

**3. Fontes e Locale:**

```dockerfile
# Alpine não inclui fontes por padrão — necessário para geração de PDFs/imagens
RUN apk add --no-cache fontconfig ttf-dejavu
```

#### Quando usar cada distribuição

| Cenário | Recomendação | Justificativa |
|---------|-------------|---------------|
| Produção com alta carga | **Ubuntu/Debian** | glibc mais estável para JVM sob pressão |
| Imagens mínimas sem restrição de performance | **Alpine** | Tamanho ~5x menor |
| Kubernetes com service discovery | **Ubuntu/Debian** | DNS resolution mais confiável |
| Aplicações com muitas threads (WebFlux) | **Ubuntu/Debian** | malloc e TLS otimizados para multi-thread |
| Ambientes de CI/CD (build efêmero) | **Alpine** | Download e startup mais rápidos |
| GraalVM Native Image | **Alpine (distroless)** | Binário estático não depende de libc |

> Para detalhes sobre GraalVM Native Image com diferentes distribuições, consulte [GraalVM-Native-Image-Spring-Boot.md](outros/GraalVM-Native-Image-Spring-Boot.md).

---

## 20. Docker — Otimização de Imagens e Runtime

Otimizar imagens Docker para aplicações Spring Boot reduz tempo de deploy, consumo de banda, superfície de ataque e tempo de startup. Esta seção foca nas decisões que geram maior impacto.

### 20.1 Imagens Base para Java

A escolha da imagem base define tamanho, segurança, performance e compatibilidade da JVM.

#### Comparação de Imagens Base

| Imagem | JDK/JRE | Base OS | Tamanho (JRE 21) | Mantenedor | Notas |
|--------|---------|---------|-------------------|------------|-------|
| `eclipse-temurin:21-jre-jammy` | OpenJDK (Adoptium) | Ubuntu 22.04 | ~220 MB | Eclipse Foundation | Padrão da comunidade, muito testada |
| `eclipse-temurin:21-jre-alpine` | OpenJDK (Adoptium) | Alpine 3.x | ~100 MB | Eclipse Foundation | Menor, usa musl |
| `amazoncorretto:21-alpine` | Amazon Corretto | Alpine 3.x | ~110 MB | Amazon | Otimizações AWS, bom suporte LTS |
| `amazoncorretto:21` | Amazon Corretto | Amazon Linux 2023 | ~250 MB | Amazon | Ideal para workloads AWS |
| `bellsoft/liberica-openjre-alpine:21` | Liberica JRE | Alpine 3.x | ~90 MB | BellSoft | Uma das menores com JRE completo |
| `bellsoft/liberica-openjre-debian:21` | Liberica JRE | Debian Slim | ~190 MB | BellSoft | Alternativa glibc compacta |
| `azul/zulu-openjdk-alpine:21-jre` | Azul Zulu | Alpine 3.x | ~105 MB | Azul Systems | Boa performance, TCK certified |
| `gcr.io/distroless/java21-debian12` | OpenJDK | Distroless | ~210 MB | Google | Sem shell — máxima segurança |

```
Tamanho comparativo (JRE 21, aproximado):

bellsoft/liberica-alpine  ████████░░░░░░░░░░░░░░░░░  ~90 MB
eclipse-temurin-alpine    ██████████░░░░░░░░░░░░░░░ ~100 MB
azul/zulu-alpine          ██████████░░░░░░░░░░░░░░░ ~105 MB
amazoncorretto-alpine     ███████████░░░░░░░░░░░░░░ ~110 MB
bellsoft/liberica-debian   ███████████████████░░░░░░ ~190 MB
distroless/java21         █████████████████████░░░░ ~210 MB
eclipse-temurin-jammy     ██████████████████████░░░ ~220 MB
amazoncorretto (AL2023)   █████████████████████████ ~250 MB
```

#### Recomendação por cenário

| Cenário | Imagem recomendada |
|---------|-------------------|
| Produção geral | `eclipse-temurin:21-jre-jammy` |
| Produção com foco em segurança | `gcr.io/distroless/java21-debian12` |
| Produção na AWS | `amazoncorretto:21` |
| Imagem menor possível | `bellsoft/liberica-openjre-alpine:21` |
| Desenvolvimento local | `eclipse-temurin:21-jdk-jammy` |

### 20.2 Multi-Stage Build para Spring Boot

Multi-stage builds separam o ambiente de compilação do ambiente de execução, resultando em imagens finais menores e mais seguras.

```dockerfile
# ===========================================================
# Estágio 1: Build com Maven e cache de dependências
# ===========================================================
FROM eclipse-temurin:21-jdk-jammy AS builder

WORKDIR /app

# Copiar apenas os arquivos de definição de dependências primeiro.
# Isso permite que o Docker faça cache da camada de download de dependências,
# que só será invalidada quando pom.xml mudar.
COPY pom.xml .
COPY .mvn/ .mvn/
COPY mvnw .
RUN chmod +x mvnw

# Download de dependências (camada cacheada)
RUN ./mvnw dependency:go-offline -B

# Copiar código-fonte (invalida cache quando código muda)
COPY src/ src/

# Build do projeto (skip tests — devem rodar no CI antes do build da imagem)
RUN ./mvnw package -DskipTests -B

# ===========================================================
# Estágio 2: Runtime — imagem mínima
# ===========================================================
FROM eclipse-temurin:21-jre-jammy AS runtime

# Criar usuário não-root para segurança
RUN groupadd --system spring && \
    useradd --system --gid spring --shell /bin/false spring

WORKDIR /app

# Copiar apenas o JAR do estágio de build
COPY --from=builder /app/target/*.jar app.jar

# Mudar para usuário não-root
USER spring:spring

# Porta padrão do Spring Boot
EXPOSE 8080

# JVM flags otimizadas para container
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-XX:InitialRAMPercentage=50.0", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-jar", "app.jar"]
```

> **Alternativa com Spring Boot Plugin:** O comando `./mvnw spring-boot:build-image` usa Cloud Native Buildpacks (Paketo) para gerar uma imagem OCI otimizada automaticamente, sem necessidade de Dockerfile. Ideal para equipes que preferem convenção sobre configuração.

```bash
# Build com Buildpacks (sem Dockerfile)
./mvnw spring-boot:build-image \
    -Dspring-boot.build-image.imageName=myapp:latest
```

### 20.3 Spring Boot Layered JARs

A partir do Spring Boot 2.3+, o JAR pode ser dividido em camadas que correspondem à frequência de mudança. Isso permite que o Docker faça cache das camadas que raramente mudam (dependências, framework), reconstruindo apenas a camada da aplicação.

```
┌─────────────────────────────────────────────────────┐
│           Layered JAR — Ordem das Camadas            │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │  dependencies        (~50 MB)  — muda raramente│  │  ◄── Cache Docker
│  ├────────────────────────────────────────────────┤  │
│  │  spring-boot-loader  (~0.5 MB) — muda raramente│  │  ◄── Cache Docker
│  ├────────────────────────────────────────────────┤  │
│  │  snapshot-dependencies (~5 MB) — muda às vezes │  │  ◄── Cache parcial
│  ├────────────────────────────────────────────────┤  │
│  │  application          (~2 MB)  — muda sempre   │  │  ◄── Reconstruída
│  └────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**Extraindo as camadas do JAR:**

```bash
# Listar camadas disponíveis
java -Djarmode=layertools -jar app.jar list
# Saída:
# dependencies
# spring-boot-loader
# snapshot-dependencies
# application

# Extrair camadas em diretórios separados
java -Djarmode=layertools -jar app.jar extract
```

**Dockerfile otimizado com Layered JARs:**

```dockerfile
# ===========================================================
# Estágio 1: Build
# ===========================================================
FROM eclipse-temurin:21-jdk-jammy AS builder
WORKDIR /app
COPY pom.xml .
COPY .mvn/ .mvn/
COPY mvnw .
RUN chmod +x mvnw
RUN ./mvnw dependency:go-offline -B
COPY src/ src/
RUN ./mvnw package -DskipTests -B

# ===========================================================
# Estágio 2: Extração das camadas
# ===========================================================
FROM eclipse-temurin:21-jdk-jammy AS extractor
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# ===========================================================
# Estágio 3: Runtime com camadas otimizadas
# ===========================================================
FROM eclipse-temurin:21-jre-jammy AS runtime

RUN groupadd --system spring && \
    useradd --system --gid spring --shell /bin/false spring

WORKDIR /app

# Cada COPY cria uma camada Docker separada.
# Camadas que mudam raramente ficam primeiro (melhor cache hit).
COPY --from=extractor /app/dependencies/ ./
COPY --from=extractor /app/spring-boot-loader/ ./
COPY --from=extractor /app/snapshot-dependencies/ ./
COPY --from=extractor /app/application/ ./

USER spring:spring
EXPOSE 8080

ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "org.springframework.boot.loader.launch.JarLauncher"]
```

**Impacto no tempo de build e push:**

| Cenário | Build tradicional | Layered JARs |
|---------|-------------------|--------------|
| Mudança apenas no código | Reconstrói ~55 MB | Reconstrói ~2 MB |
| Mudança em dependência SNAPSHOT | Reconstrói ~55 MB | Reconstrói ~7 MB |
| Mudança no pom.xml | Reconstrói ~55 MB | Reconstrói ~55 MB |
| Push para registry (código) | ~55 MB upload | ~2 MB upload |

### 20.4 Otimizações de Runtime

#### Limitação de CPU e Memória

```bash
# docker run com limites explícitos
docker run -d \
    --name spring-app \
    --cpus="2.0" \
    --memory="1g" \
    --memory-swap="1g" \
    -p 8080:8080 \
    myapp:latest
```

> **Importante:** `--memory-swap` igual a `--memory` desabilita swap no container, comportamento recomendado para JVM.

#### Health Checks com Spring Boot Actuator

```dockerfile
# No Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1
```

```yaml
# application.yml — configurar Actuator para health checks
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

#### Graceful Shutdown

Graceful shutdown permite que a aplicação finalize requisições em andamento antes de encerrar, evitando erros 502/503 durante deploys.

```yaml
# application.yml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

#### Docker Compose completo para produção

```yaml
# docker-compose.yml
services:
  spring-app:
    image: myapp:latest
    build:
      context: .
      dockerfile: Dockerfile
    container_name: spring-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - JAVA_OPTS=-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -XX:+UseG1GC
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/mydb
      - SPRING_DATASOURCE_USERNAME=app_user
      - SPRING_DATASOURCE_PASSWORD_FILE=/run/secrets/db_password
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 1024M
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health/liveness"]
      interval: 30s
      timeout: 5s
      start_period: 60s
      retries: 3
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - app-network
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres-data:
    driver: local

networks:
  app-network:
    driver: bridge
```

> Para detalhes sobre Docker Compose, volumes, networking e Kubernetes, consulte [Docker-Compose-Kubernetes.md](outros/Docker-Compose-Kubernetes.md).

---

## 21. Limites de Recursos em Docker e Kubernetes

Configurar limites de recursos corretamente é essencial para evitar contenção de CPU, OOM Kills e comportamento imprevisível. A JVM moderna (11+) reconhece limites de container via `UseContainerSupport` (habilitado por padrão), mas o alinhamento entre limites do orquestrador e flags da JVM ainda requer atenção.

```
┌─────────────────────────────────────────────────────────────────────┐
│           Alinhamento JVM ↔ Container                               │
│                                                                     │
│  Container Memory Limit: 1024 MB                                    │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  JVM Heap (MaxRAMPercentage=75%) ──────── ~768 MB             │  │
│  │  ┌──────────────────────────────────────────────────────────┐ │  │
│  │  │  Metaspace, Thread Stacks, Native ──── ~200 MB           │ │  │
│  │  │  Code Cache, Direct Buffers            (estimativa)      │ │  │
│  │  └──────────────────────────────────────────────────────────┘ │  │
│  │  Margem livre (evitar OOM Kill) ────────── ~56 MB             │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ⚠ Se Heap + Non-Heap > Container Limit → OOM Kill (SIGKILL)       │
│  ⚠ MaxRAMPercentage > 80% → risco alto de OOM Kill                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 21.1 Docker Compose — Resource Limits

No Docker Compose v3+, limites de recursos são definidos sob `deploy.resources`. Para aplicação sem Docker Swarm, é necessário usar `docker compose up` (v2 do Compose CLI) que respeita estas configurações mesmo em modo standalone.

```yaml
# docker-compose.yml — Foco em resource limits + JVM alignment
services:
  api-service:
    image: myapp:latest
    environment:
      # JVM flags alinhadas com os limites do container
      JAVA_OPTS: >-
        -XX:+UseContainerSupport
        -XX:MaxRAMPercentage=75.0
        -XX:InitialRAMPercentage=50.0
        -XX:+UseG1GC
        -XX:MaxGCPauseMillis=200
        -Xss512k
        -XX:+HeapDumpOnOutOfMemoryError
        -XX:HeapDumpPath=/tmp/heapdump.hprof
    deploy:
      resources:
        limits:
          # Limite máximo — container é morto (OOM Kill) se exceder
          cpus: '2.0'
          memory: 1024M
        reservations:
          # Reserva garantida — scheduler garante estes recursos
          cpus: '0.5'
          memory: 512M
      replicas: 2
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  worker-service:
    image: myworker:latest
    environment:
      JAVA_OPTS: >-
        -XX:+UseContainerSupport
        -XX:MaxRAMPercentage=75.0
        -XX:+UseZGC
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 768M
        reservations:
          cpus: '0.25'
          memory: 384M
```

**Cálculo de memória JVM para container de 1024 MB:**

```
Container Limit:           1024 MB
├── MaxRAMPercentage=75% →  768 MB (Heap máximo)
├── Metaspace:             ~100 MB (Spring Boot típico)
├── Thread Stacks:          ~50 MB (100 threads × 512KB)
├── Code Cache:             ~48 MB (default ReservedCodeCacheSize)
├── Direct Buffers:         ~10 MB (NIO, Netty)
└── Overhead (GC, JNI):    ~48 MB
                           -------
    Total estimado:        ~1024 MB  ← No limite! Use 70% para margem
```

> **Recomendação:** Para containers de 1 GB, usar `MaxRAMPercentage=70.0` é mais seguro. Para 2 GB+, `75.0` funciona bem.

### 21.2 Kubernetes — Requests e Limits

Kubernetes usa `requests` e `limits` para scheduling e enforcement de recursos. Entender a diferença é fundamental para evitar throttling de CPU e OOM Kills.

| Conceito | Descrição | Consequência se insuficiente |
|----------|-----------|------------------------------|
| **requests.cpu** | CPU garantida para o pod; usada pelo scheduler para alocação | Pod não é schedulado (Pending) |
| **requests.memory** | Memória garantida; scheduler usa para decidir onde alocar | Pod não é schedulado (Pending) |
| **limits.cpu** | Máximo de CPU; exceder causa **throttling** (não kill) | Latência alta, timeouts |
| **limits.memory** | Máximo de memória; exceder causa **OOM Kill** (SIGKILL) | Pod reiniciado, CrashLoopBackOff |

#### QoS Classes

O Kubernetes atribui uma Quality of Service class a cada pod com base na configuração de resources:

```
┌──────────────────────────────────────────────────────────────────┐
│                     QoS Classes                                   │
│                                                                   │
│  Guaranteed ──► requests == limits (CPU e memória)                │
│                 Última prioridade para eviction                   │
│                 Ideal para aplicações críticas                    │
│                                                                   │
│  Burstable ───► requests < limits (pelo menos um definido)       │
│                 Prioridade intermediária para eviction             │
│                 Bom trade-off custo/estabilidade                  │
│                                                                   │
│  BestEffort ──► Nenhum request ou limit definido                  │
│                 Primeira prioridade para eviction                 │
│                 NÃO usar em produção                              │
└──────────────────────────────────────────────────────────────────┘
```

#### Deployment Manifest Completo

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  namespace: production
  labels:
    app: spring-app
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      terminationGracePeriodSeconds: 45
      containers:
        - name: spring-app
          image: registry.example.com/myapp:1.0.0
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod,k8s"
            - name: JAVA_OPTS
              value: >-
                -XX:+UseContainerSupport
                -XX:MaxRAMPercentage=75.0
                -XX:InitialRAMPercentage=50.0
                -XX:+UseG1GC
                -XX:MaxGCPauseMillis=200
                -XX:+HeapDumpOnOutOfMemoryError
                -XX:HeapDumpPath=/tmp/heapdump.hprof
          resources:
            # QoS Class: Burstable (requests < limits)
            requests:
              cpu: "500m"       # 0.5 vCPU garantido
              memory: "768Mi"   # 768 MB garantidos
            limits:
              cpu: "2000m"      # Pode usar até 2 vCPUs (burst)
              memory: "1024Mi"  # Máximo 1 GB (OOM Kill se exceder)
          # Probes detalhadas na seção 21.3
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: http
            initialDelaySeconds: 45
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 20
```

**Dica sobre CPU throttling:** No Kubernetes, `limits.cpu` usa CFS (Completely Fair Scheduler) do Linux. Um pod com `limits.cpu: "1000m"` que tenta usar mais de 1 vCPU não é morto — é **throttled**, o que se manifesta como latência inesperada. Monitorar `container_cpu_cfs_throttled_seconds_total` no Prometheus ajuda a identificar se o limite de CPU está subdimensionado.

```yaml
# Para aplicações Java que sofrem throttling de CPU frequente,
# considerar QoS Guaranteed (requests == limits):
resources:
  requests:
    cpu: "2000m"
    memory: "1024Mi"
  limits:
    cpu: "2000m"      # requests == limits
    memory: "1024Mi"  # → QoS Guaranteed
```

### 21.3 Liveness, Readiness e Startup Probes

Probes são mecanismos do Kubernetes para verificar a saúde do container. Configuração incorreta é uma das causas mais comuns de instabilidade em deploys de aplicações Spring Boot.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Ciclo de Vida das Probes                          │
│                                                                     │
│  Container Start                                                    │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────┐     Período: initialDelaySeconds                   │
│  │ Startup     │     + (periodSeconds × failureThreshold)           │
│  │ Probe       │──── Permite JVM warmup sem interferência           │
│  │ (gate)      │     das outras probes                              │
│  └──────┬──────┘                                                    │
│         │ sucesso                                                   │
│         ▼                                                           │
│  ┌─────────────┐  ┌──────────────┐                                  │
│  │ Liveness    │  │ Readiness    │   Rodam em paralelo              │
│  │ Probe       │  │ Probe        │   após startup probe passar      │
│  │             │  │              │                                   │
│  │ Falha →     │  │ Falha →      │                                  │
│  │ RESTART     │  │ Remove do    │                                  │
│  │ container   │  │ Service      │                                  │
│  └─────────────┘  │ (sem tráfego)│                                  │
│                   └──────────────┘                                   │
└─────────────────────────────────────────────────────────────────────┘
```

| Probe | Endpoint Actuator | Objetivo | Ação em falha |
|-------|-------------------|----------|---------------|
| **Startup** | `/actuator/health/liveness` | Esperar a aplicação inicializar | Reinicia container se não responder no tempo |
| **Liveness** | `/actuator/health/liveness` | Verificar se o processo está saudável | Reinicia container |
| **Readiness** | `/actuator/health/readiness` | Verificar se pode receber tráfego | Remove do Service (para de receber requests) |

#### Configuração do Spring Boot Actuator para Probes

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true
      group:
        # Liveness: verifica se a aplicação está "viva"
        # Não deve incluir dependências externas (DB, Redis)
        liveness:
          include: livenessState
        # Readiness: verifica se está pronta para receber tráfego
        # Inclui dependências necessárias para processar requests
        readiness:
          include: readinessState,db,redis
      show-details: when-authorized
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

> **Importante:** O endpoint de liveness **nunca** deve verificar dependências externas (banco, cache, mensageria). Se o banco estiver fora, o liveness falha, o container reinicia, mas o banco continua fora — gerando um loop infinito de reinicializações (CrashLoopBackOff). Dependências externas devem ser verificadas apenas no readiness probe.

#### Manifest Completo com Probes Otimizadas

```yaml
# k8s/deployment-with-probes.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      # Tempo para graceful shutdown (deve ser > spring.lifecycle.timeout-per-shutdown-phase)
      terminationGracePeriodSeconds: 45
      containers:
        - name: spring-app
          image: registry.example.com/myapp:1.0.0
          ports:
            - containerPort: 8080
              name: http
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod,k8s"
            - name: JAVA_OPTS
              value: >-
                -XX:+UseContainerSupport
                -XX:MaxRAMPercentage=75.0
                -XX:+UseG1GC
          resources:
            requests:
              cpu: "500m"
              memory: "768Mi"
            limits:
              cpu: "2000m"
              memory: "1024Mi"

          # --- STARTUP PROBE ---
          # Gate para as outras probes. Permite que a JVM inicialize
          # sem ser morta prematuramente pelo liveness probe.
          # Budget total: initialDelay + (period × failure)
          #             = 10 + (5 × 20) = 110 segundos para startup
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 20
            # successThreshold é sempre 1 para startup e liveness

          # --- LIVENESS PROBE ---
          # Verifica se o processo Java está responsivo.
          # Só roda após startup probe ter sucesso.
          # NÃO verificar dependências externas aqui.
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: http
            # initialDelaySeconds não é necessário com startup probe,
            # mas pode ser incluído como segurança adicional
            initialDelaySeconds: 0
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 3
            # Total até restart: 15 × 3 = 45 segundos de falhas

          # --- READINESS PROBE ---
          # Verifica se a aplicação pode processar requests.
          # Inclui checagem de DB, Redis, etc.
          # Falha → pod removido do Service, sem restart.
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: http
            initialDelaySeconds: 0
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
            successThreshold: 1
            # Pod removido do Service após 30s de falhas
            # Retorna ao Service após 1 sucesso

          lifecycle:
            preStop:
              exec:
                # Espera antes de receber SIGTERM para permitir que o
                # Kubernetes atualize as regras de iptables/IPVS
                command: ["sh", "-c", "sleep 5"]
```

**Erros comuns na configuração de probes:**

| Erro | Consequência | Solução |
|------|-------------|---------|
| `initialDelaySeconds` muito curto sem startup probe | Container reiniciado durante startup da JVM | Usar startup probe com budget de ~90-120s |
| Liveness probe verifica DB/Redis | CrashLoopBackOff quando dependência cai | Verificar dependências apenas no readiness |
| `timeoutSeconds` muito baixo (1s) | Falsos positivos durante GC pause | Usar 3-5 segundos |
| `failureThreshold: 1` no liveness | Um spike de latência mata o container | Usar `failureThreshold: 3` |
| `terminationGracePeriodSeconds` < tempo de shutdown | Requests em andamento são interrompidas | Alinhar com `spring.lifecycle.timeout-per-shutdown-phase` |
| Sem `preStop` hook | Requests chegam durante shutdown | Adicionar `sleep 5` no preStop |

> Para detalhes completos sobre Docker, Docker Compose e Kubernetes, consulte [Docker-Compose-Kubernetes.md](outros/Docker-Compose-Kubernetes.md).

---

**Parte 5 — Profiling, Tracing e Observabilidade**

## 22. Profiling com JFR e JMC

O Java Flight Recorder (JFR) e o JDK Mission Control (JMC) formam o kit de profiling oficial da JVM. São ferramentas de nível production-grade, com overhead quase zero, que permitem coletar e analisar dados detalhados sobre CPU, memória, I/O, threads e GC sem impacto perceptível na aplicação.

### 22.1 Java Flight Recorder (JFR) — Conceitos

O JFR é um framework de coleta de eventos embutido diretamente na JVM (desde o JDK 11 como open source). Ele grava eventos em um buffer circular em memória e, opcionalmente, persiste em disco.

**O que o JFR registra:**

| Categoria           | Eventos coletados                                                    |
|----------------------|----------------------------------------------------------------------|
| CPU                  | Amostras de stack (CPU sampling), compilação JIT, class loading      |
| Memória              | Alocações por thread/TLAB, promoções para Old Gen, object count      |
| Garbage Collector    | Pausas de GC, fases, causa, tempo total                              |
| Threads              | Contention de locks (`MonitorWait`, `MonitorEnter`), thread sleep    |
| I/O                  | Leitura/escrita de arquivos, operações de socket (latência e bytes)  |
| Exceções             | Exceções lançadas (com stack trace), erros da JVM                    |
| Compilação           | Métodos compilados pelo JIT, inlining, deoptimizações                |
| Sistema operacional  | Uso de CPU do processo/host, memória física, carga do sistema        |

**Overhead quase zero:** o JFR foi projetado pela Oracle/Sun para rodar continuamente em produção. O overhead típico é inferior a 1% de CPU e memória, pois os eventos são gravados via buffers thread-local sem locks globais.

```
┌─────────────────────────────────────────────────┐
│                    JVM (HotSpot)                │
│                                                  │
│  Thread 1 ──► [Buffer Local] ──┐                │
│  Thread 2 ──► [Buffer Local] ──┤                │
│  Thread 3 ──► [Buffer Local] ──┼──► Global Ring │
│  GC Thread ──► [Buffer Local] ──┘    Buffer     │
│                                       │          │
│                                       ▼          │
│                              Arquivo .jfr        │
└─────────────────────────────────────────────────┘
```

### 22.2 Habilitando JFR

Existem duas formas principais de iniciar uma gravação JFR: via flags da JVM na inicialização ou via `jcmd` em tempo de execução.

**Opção 1 — Flag da JVM na inicialização:**

```bash
# Gravação com duração fixa de 60 segundos
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr \
     -jar minha-app.jar

# Gravação com perfil detalhado (profile inclui mais eventos)
java -XX:StartFlightRecording=duration=120s,filename=recording.jfr,settings=profile \
     -jar minha-app.jar
```

Os dois perfis padrão são:

| Perfil      | Arquivo                        | Uso recomendado            |
|-------------|--------------------------------|----------------------------|
| `default`   | `default.jfc`                  | Produção (overhead mínimo) |
| `profile`   | `profile.jfc`                  | Desenvolvimento/debug      |

**Opção 2 — Comandos `jcmd` em tempo de execução:**

```bash
# Descobrir o PID do processo Java
jcmd -l

# Iniciar gravação (sem parar a aplicação)
jcmd <PID> JFR.start name=minha-gravacao duration=60s filename=recording.jfr

# Verificar gravações ativas
jcmd <PID> JFR.check

# Parar uma gravação manualmente
jcmd <PID> JFR.stop name=minha-gravacao

# Dump de uma gravação contínua (sem parar)
jcmd <PID> JFR.dump name=minha-gravacao filename=snapshot.jfr
```

**Opção 3 — Programática (útil para testes de performance):**

```java
import jdk.jfr.Configuration;
import jdk.jfr.Recording;
import java.nio.file.Path;

public class JfrProgramatico {

    public void gravarPerfil() throws Exception {
        Configuration config = Configuration.getConfiguration("profile");

        try (Recording recording = new Recording(config)) {
            recording.start();

            // Executa a operação que deseja analisar
            executarOperacaoCritica();

            recording.stop();
            recording.dump(Path.of("operacao-critica.jfr"));
        }
    }
}
```

### 22.3 JFR em Produção — Continuous Recording

Em produção, a prática recomendada é manter uma gravação contínua rodando permanentemente. O JFR usa um buffer circular que descarta dados antigos automaticamente, sem crescimento ilimitado de disco.

```bash
# Iniciar gravação contínua na inicialização da JVM
java -XX:StartFlightRecording=disk=true,maxsize=500m,maxage=1d,dumponexit=true,filename=producao.jfr \
     -jar minha-app.jar
```

| Parâmetro      | Descrição                                                          |
|----------------|--------------------------------------------------------------------|
| `disk=true`    | Persiste eventos em disco (não apenas em memória)                  |
| `maxsize=500m` | Tamanho máximo do arquivo — dados antigos são sobrescritos          |
| `maxage=1d`    | Retém no máximo 1 dia de dados                                     |
| `dumponexit`   | Grava o buffer ao encerrar a JVM (útil em crashes)                 |

**Dump sob demanda quando um incidente ocorre:**

```bash
# Incidente detectado — gerar dump do buffer atual
jcmd <PID> JFR.dump name=1 filename=/tmp/incidente-$(date +%Y%m%d-%H%M%S).jfr

# Dump apenas dos últimos 5 minutos
jcmd <PID> JFR.dump name=1 maxage=5m filename=/tmp/ultimos-5min.jfr
```

Essa abordagem permite que a equipe colete dados retroativos do momento exato de um problema sem precisar reproduzi-lo.

### 22.4 JDK Mission Control (JMC) — Análise

O JDK Mission Control (JMC) é a ferramenta gráfica oficial para analisar arquivos `.jfr`. Ele pode ser baixado separadamente em [https://jdk.java.net/jmc/](https://jdk.java.net/jmc/).

**Visões principais do JMC:**

| Visão                  | O que mostra                                                       | Para que serve                          |
|------------------------|--------------------------------------------------------------------|-----------------------------------------|
| **Hot Methods**        | Métodos que mais consomem CPU (ordenados por amostras)              | Identificar hotspots de CPU             |
| **Memory**             | Alocações por classe/método, pressão de alocação                   | Encontrar memory leaks e churn          |
| **Thread Contention**  | Locks, tempo de espera por monitor, deadlocks                      | Diagnosticar contenção de threads       |
| **I/O**                | Operações de arquivo e socket (latência, bytes transferidos)        | Identificar I/O lento                   |
| **GC**                 | Pausas de GC, frequência, memória recuperada, causa                | Tuning de GC                            |
| **Exceptions**         | Exceções lançadas (mesmo as capturadas), frequência                | Encontrar exceções excessivas           |

**Como identificar CPU hotspots:**

1. Abra o arquivo `.jfr` no JMC.
2. Navegue até **Method Profiling** (ou **Hot Methods**).
3. Ordene por **Sample Count** — os métodos no topo são os que mais consomem CPU.
4. Clique em um método para ver o stack trace completo.
5. Verifique se o hotspot está no código da aplicação ou em uma biblioteca.

**Como identificar memory leaks:**

1. Na visão **Memory**, observe o painel **Allocation by Class**.
2. Classes com alocação desproporcional indicam possível vazamento.
3. Na visão **GC**, verifique se o heap usado após Full GC está crescendo ao longo do tempo — isso confirma um leak.
4. Cruze com **Allocation by Thread/Stack Trace** para encontrar o código responsável.

### 22.5 JFR Event Streaming (Java 14+)

A partir do Java 14, o JFR suporta streaming de eventos em tempo real via `RecordingStream`. Isso permite que a aplicação reaja imediatamente a eventos da JVM sem precisar analisar um arquivo `.jfr` depois.

```java
import jdk.jfr.consumer.RecordingStream;
import java.time.Duration;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class JfrMonitor {

    private static final Logger log = LoggerFactory.getLogger(JfrMonitor.class);

    public void iniciarMonitoramento() {
        RecordingStream stream = new RecordingStream();

        // Habilitar eventos de GC
        stream.enable("jdk.GarbageCollection")
              .withThreshold(Duration.ofMillis(100));

        // Habilitar eventos de CPU
        stream.enable("jdk.CPULoad")
              .withPeriod(Duration.ofSeconds(5));

        // Habilitar contenção de monitor
        stream.enable("jdk.JavaMonitorEnter")
              .withThreshold(Duration.ofMillis(50));

        // Alerta para pausas de GC longas
        stream.onEvent("jdk.GarbageCollection", event -> {
            Duration pausa = event.getDuration();
            String causa = event.getString("cause");
            String nome = event.getString("name");

            if (pausa.toMillis() > 200) {
                log.warn("GC longo detectado: tipo={}, causa={}, duração={}ms",
                        nome, causa, pausa.toMillis());
                // Aqui poderia enviar alerta via Slack, PagerDuty etc.
            }
        });

        // Monitorar carga de CPU
        stream.onEvent("jdk.CPULoad", event -> {
            float jvmUser = event.getFloat("jvmUser");
            float jvmSystem = event.getFloat("jvmSystem");
            float machineTotal = event.getFloat("machineTotal");

            if (jvmUser > 0.8f) {
                log.warn("CPU alta na JVM: user={:.1f}%, system={:.1f}%, machine={:.1f}%",
                        jvmUser * 100, jvmSystem * 100, machineTotal * 100);
            }
        });

        // Monitorar contenção de locks
        stream.onEvent("jdk.JavaMonitorEnter", event -> {
            Duration bloqueio = event.getDuration();
            if (bloqueio.toMillis() > 100) {
                log.warn("Contenção de lock: classe={}, duração={}ms",
                        event.getClass("monitorClass").getName(),
                        bloqueio.toMillis());
            }
        });

        // Iniciar em thread separada (não bloqueia)
        stream.startAsync();
        log.info("JFR Event Streaming iniciado");
    }
}
```

**Integração com Spring Boot:**

```java
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class JfrStreamingInitializer {

    private final JfrMonitor jfrMonitor;

    public JfrStreamingInitializer(JfrMonitor jfrMonitor) {
        this.jfrMonitor = jfrMonitor;
    }

    @EventListener(ApplicationReadyEvent.class)
    public void onReady() {
        jfrMonitor.iniciarMonitoramento();
    }
}
```

---

## 23. async-profiler — CPU e Allocation Profiling

O async-profiler é uma ferramenta de profiling de baixo nível para a JVM que complementa o JFR, oferecendo visibilidade sobre frames nativos e eliminando o viés de safepoint que afeta outros profilers.

### 23.1 O que é o async-profiler

O async-profiler é um profiler de amostragem open source que coleta stack traces sem depender de safepoints da JVM. Isso o torna mais preciso que profilers tradicionais (incluindo o JFR em algumas situações) para identificar hotspots reais.

**Comparação com JFR:**

| Característica              | JFR                              | async-profiler                         |
|-----------------------------|----------------------------------|----------------------------------------|
| Frames nativos (C/C++)      | Limitado                         | Completo (perf_events do Linux)        |
| Viés de safepoint           | Presente (amostra em safepoints) | Ausente (amostra a qualquer momento)   |
| Allocation profiling        | Sim (TLAB-based)                 | Sim (mais preciso para objetos grandes)|
| Flame graph nativo          | Não (precisa converter)          | Sim (geração direta em HTML/SVG)       |
| Overhead                    | < 1%                             | < 1-2%                                 |
| Plataforma                  | Todas (JVM built-in)             | Linux e macOS (requer perf_events)     |
| Integração com ferramentas  | JMC, IntelliJ                    | Flame graphs, IntelliJ                 |

**O que é o viés de safepoint?** Profilers tradicionais só conseguem amostrar o stack quando a JVM está em um safepoint (ponto seguro para GC). Isso significa que código entre safepoints — como loops intensivos com JIT — pode ser invisível. O async-profiler usa sinais POSIX (`SIGPROF`) para amostrar em qualquer ponto da execução, eliminando esse viés.

```
Profiler tradicional (com viés de safepoint):
  ──────●──────────────────●──────────────────●──────
       SP1               SP2               SP3
       ↑ amostra          ↑ amostra          ↑ amostra
       (perde tudo entre safepoints)

async-profiler (sem viés):
  ──●───●──●────●───●──●───●────●───●──●────●───●──
    ↑   ↑  ↑    ↑   ↑  ↑   ↑    ↑   ↑  ↑    ↑   ↑
    (amostra em qualquer momento da execução)
```

### 23.2 Instalação e Uso Básico

**Download e instalação (Linux):**

```bash
# Baixar a versão mais recente
wget https://github.com/async-profiler/async-profiler/releases/download/v3.0/async-profiler-3.0-linux-x64.tar.gz
tar xzf async-profiler-3.0-linux-x64.tar.gz
cd async-profiler-3.0-linux-x64

# Permitir acesso aos perf_events (necessário em kernels recentes)
echo 1 | sudo tee /proc/sys/kernel/perf_event_paranoid
echo 0 | sudo tee /proc/sys/kernel/kptr_restrict
```

**CPU profiling — identificar métodos que mais consomem CPU:**

```bash
# Descobrir PID do processo Java
jcmd -l

# Profiling de CPU por 30 segundos, gerando flame graph HTML
./asprof -d 30 -f cpu-profile.html <PID>

# Profiling de CPU com saída em formato texto (top methods)
./asprof -d 30 -o flat <PID>

# Exemplo de saída flat:
#    ns  percent  samples  top
# -----  -------  -------  ---
#  4.2s   42.00%      420  com.example.service.PedidoService.calcularFrete
#  1.8s   18.00%      180  java.util.regex.Pattern.compile
#  1.2s   12.00%      120  com.fasterxml.jackson.databind.ObjectMapper.writeValueAsString
```

**Allocation profiling — identificar onde objetos são alocados:**

```bash
# Profiling de alocações de memória (rastreia new/malloc)
./asprof -d 30 -e alloc -f alloc-profile.html <PID>

# Allocation profiling com threshold (apenas alocações > 512KB)
./asprof -d 30 -e alloc --alloc 512k -f alloc-grandes.html <PID>
```

**Lock profiling — identificar contenção de locks:**

```bash
# Profiling de contenção de locks
./asprof -d 30 -e lock -f lock-profile.html <PID>
```

### 23.3 Flame Graphs

Flame graphs são a forma mais eficiente de visualizar dados de profiling. Cada barra horizontal representa um método no stack trace, e a largura indica o tempo relativo gasto naquele método.

**Como ler um flame graph:**

```
┌─────────────────────────────────────────────────────────────┐
│                    calcularFrete (42%)                       │  ← HOTSPOT
├──────────────────────┬──────────────────────────────────────┤
│ consultarTabela (20%)│     aplicarDesconto (22%)            │
├──────────┬───────────┼─────────────┬────────────────────────┤
│ query(12)│ parse (8) │ validate(10)│  BigDecimal.mult (12)  │
├──────────┴───────────┴─────────────┴────────────────────────┤
│                  processarPedido (100%)                      │  ← BASE
└─────────────────────────────────────────────────────────────┘

Leitura:
- Eixo X (largura): tempo relativo (NÃO é cronológico)
- Eixo Y (altura): profundidade do stack (base = entry point)
- Barras mais largas = métodos que mais consomem recursos
- Platô no topo = método folha que executa o trabalho real
```

**Regras práticas para identificar problemas:**

1. **Platôs largos no topo**: o método está fazendo trabalho pesado diretamente (loop, cálculo, I/O).
2. **Torres estreitas e altas**: recursão profunda — pode indicar algoritmo ineficiente.
3. **Muitas barras do mesmo pacote/classe**: uma classe está dominando a execução.
4. **Frames de `java.util.regex`**: regex sendo compilado repetidamente (deveria ser `static final`).
5. **Frames de serialização (Jackson/Gson)**: serialização excessiva, considerar cache ou DTOs mais leves.

**Gerando flame graphs:**

```bash
# Flame graph interativo em HTML (recomendado — permite zoom e busca)
./asprof -d 60 -f profile.html <PID>

# Flame graph em SVG
./asprof -d 60 -f profile.svg <PID>

# Flame graph reverso (mostra callers em vez de callees)
./asprof -d 60 --reverse -f profile-reverso.html <PID>

# Filtrar por pacote específico
./asprof -d 60 --include 'com.example.*' -f profile-app.html <PID>
```

### 23.4 Wall-Clock Profiling

O CPU profiling padrão só amostra threads que estão efetivamente usando a CPU. Para aplicações I/O-bound (chamadas HTTP, queries de banco, esperas em filas), muitas threads passam a maior parte do tempo bloqueadas e ficam invisíveis no CPU profiling.

O wall-clock profiling amostra todas as threads, independente do estado — running, sleeping, waiting, blocked.

```bash
# Wall-clock profiling (todas as threads, todos os estados)
./asprof -d 30 -e wall -f wall-profile.html <PID>

# Wall-clock apenas de threads específicas (filtrando por nome)
./asprof -d 30 -e wall -t -f wall-profile.html <PID>
```

**Quando usar cada modo:**

| Cenário                           | Modo recomendado  | Por quê                                      |
|-----------------------------------|-------------------|----------------------------------------------|
| Cálculos pesados, loops           | `-e cpu`          | Foco no tempo real de CPU                     |
| Chamadas HTTP/REST externas       | `-e wall`         | Thread fica em WAITING durante I/O            |
| Queries de banco lentas           | `-e wall`         | Thread fica bloqueada esperando resposta      |
| Leitura/escrita de arquivos       | `-e wall`         | I/O de disco bloqueia a thread                |
| Aplicação com alta latência       | `-e wall` primeiro| Descobre se o gargalo é CPU ou espera         |
| Contenção de locks/synchronized   | `-e lock`         | Mede tempo em `BLOCKED` por monitors          |
| Alocação excessiva de objetos     | `-e alloc`        | Rastreia onde objetos são criados             |

**Exemplo prático — diagnosticando latência alta:**

```bash
# 1. Começar com wall-clock para visão geral
./asprof -d 60 -e wall -f 1-wall.html <PID>

# 2. Se o flame graph mostrar CPU dominante, detalhar com CPU profiling
./asprof -d 60 -e cpu -f 2-cpu.html <PID>

# 3. Se mostrar muita espera em I/O, verificar alocações
./asprof -d 60 -e alloc -f 3-alloc.html <PID>

# 4. Se mostrar contenção de locks, detalhar
./asprof -d 60 -e lock -f 4-lock.html <PID>
```

---

## 24. Observabilidade com Micrometer, Prometheus e Grafana

O Micrometer é a biblioteca de métricas padrão do Spring Boot 3.x, funcionando como uma fachada (similar ao SLF4J para logs) que abstrai backends de monitoramento como Prometheus, Datadog, New Relic e CloudWatch. Combinado com Prometheus para coleta e Grafana para visualização, forma a stack de observabilidade mais adotada no ecossistema Java.

### 24.1 Métricas Automáticas do Spring Boot

O Spring Boot Actuator com Micrometer expõe automaticamente dezenas de métricas sem código adicional. Basta incluir as dependências:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  metrics:
    tags:
      application: ${spring.application.name}
```

**Métricas automáticas mais importantes para performance:**

| Métrica                              | Tipo      | O que indica                                           |
|--------------------------------------|-----------|--------------------------------------------------------|
| `jvm.memory.used`                    | Gauge     | Memória heap/non-heap em uso                           |
| `jvm.memory.max`                     | Gauge     | Memória máxima configurada                             |
| `jvm.gc.pause`                       | Timer     | Duração das pausas de GC (por causa e geração)         |
| `jvm.gc.memory.promoted`            | Counter   | Bytes promovidos para Old Gen                          |
| `jvm.threads.live`                   | Gauge     | Número de threads ativas                               |
| `jvm.threads.peak`                   | Gauge     | Pico de threads desde o início da JVM                  |
| `jvm.classes.loaded`                 | Gauge     | Classes carregadas atualmente                          |
| `http.server.requests`              | Timer     | Latência e contagem de requisições HTTP (por URI, método, status) |
| `hikaricp.connections.active`        | Gauge     | Conexões de banco em uso no momento                    |
| `hikaricp.connections.idle`          | Gauge     | Conexões ociosas no pool                               |
| `hikaricp.connections.pending`       | Gauge     | Threads esperando por uma conexão                      |
| `hikaricp.connections.timeout`       | Counter   | Conexões que deram timeout (pool esgotado)             |
| `hikaricp.connections.creation`      | Timer     | Tempo para criar novas conexões físicas                |
| `spring.data.repository.invocations`| Timer     | Latência de chamadas a repositórios Spring Data        |
| `process.cpu.usage`                  | Gauge     | Uso de CPU do processo (0.0 a 1.0)                     |
| `system.cpu.usage`                   | Gauge     | Uso de CPU do sistema (0.0 a 1.0)                      |
| `disk.total` / `disk.free`          | Gauge     | Espaço total e livre em disco                          |

### 24.2 Métricas Customizadas para Performance

Além das métricas automáticas, é essencial criar métricas específicas para operações críticas do negócio.

**Timer — medir latência de operações:**

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

@Service
public class PedidoService {

    private final Timer processamentoTimer;
    private final Timer integracaoExternaTimer;

    public PedidoService(MeterRegistry registry) {
        this.processamentoTimer = Timer.builder("pedido.processamento")
                .description("Tempo de processamento de pedidos")
                .tag("tipo", "novo")
                .publishPercentiles(0.5, 0.95, 0.99)       // P50, P95, P99
                .publishPercentileHistogram()                // histograma para agregação no Prometheus
                .serviceLevelObjectives(
                    Duration.ofMillis(100),
                    Duration.ofMillis(500),
                    Duration.ofSeconds(1))                   // buckets SLO
                .register(registry);

        this.integracaoExternaTimer = Timer.builder("integracao.externa")
                .description("Tempo de chamadas a APIs externas")
                .tag("servico", "pagamento")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(registry);
    }

    public Pedido processar(CriarPedidoRequest request) {
        return processamentoTimer.record(() -> {
            Pedido pedido = criarPedido(request);

            // Medir sub-operação separadamente
            integracaoExternaTimer.record(() -> {
                chamarGatewayPagamento(pedido);
            });

            return pedido;
        });
    }
}
```

**DistributionSummary — medir tamanhos e quantidades:**

```java
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class RelatorioService {

    private final DistributionSummary tamanhoPayload;
    private final DistributionSummary itensporPedido;

    public RelatorioService(MeterRegistry registry) {
        this.tamanhoPayload = DistributionSummary.builder("http.response.payload.size")
                .description("Tamanho dos payloads de resposta em bytes")
                .baseUnit("bytes")
                .publishPercentiles(0.5, 0.95, 0.99)
                .publishPercentileHistogram()
                .maximumExpectedValue(10_000_000.0)   // 10 MB
                .register(registry);

        this.itensporPedido = DistributionSummary.builder("pedido.itens.quantidade")
                .description("Quantidade de itens por pedido")
                .publishPercentiles(0.5, 0.95)
                .register(registry);
    }

    public byte[] gerarRelatorio(RelatorioRequest request) {
        byte[] relatorio = processarRelatorio(request);
        tamanhoPayload.record(relatorio.length);
        return relatorio;
    }

    public Pedido criarPedido(List<Item> itens) {
        itensporPedido.record(itens.size());
        // ...
    }
}
```

**Counter e Gauge — contadores e medidores:**

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

@Component
public class FilaProcessamento {

    private final BlockingQueue<Tarefa> fila = new LinkedBlockingQueue<>(1000);
    private final Counter tarefasProcessadas;
    private final Counter tarefasRejeitadas;

    public FilaProcessamento(MeterRegistry registry) {
        // Gauge acompanha o tamanho da fila em tempo real
        Gauge.builder("fila.processamento.tamanho", fila, BlockingQueue::size)
                .description("Tarefas pendentes na fila de processamento")
                .register(registry);

        this.tarefasProcessadas = Counter.builder("fila.processamento.total")
                .description("Total de tarefas processadas")
                .tag("resultado", "sucesso")
                .register(registry);

        this.tarefasRejeitadas = Counter.builder("fila.processamento.total")
                .description("Total de tarefas rejeitadas")
                .tag("resultado", "rejeitada")
                .register(registry);
    }

    public boolean adicionar(Tarefa tarefa) {
        boolean aceita = fila.offer(tarefa);
        if (!aceita) {
            tarefasRejeitadas.increment();
        }
        return aceita;
    }

    public void processar() {
        Tarefa tarefa = fila.poll();
        if (tarefa != null) {
            executar(tarefa);
            tarefasProcessadas.increment();
        }
    }
}
```

### 24.3 Prometheus — Scraping e Alertas

O Prometheus coleta métricas fazendo scrape periódico no endpoint `/actuator/prometheus` da aplicação.

**Configuração do Prometheus (`prometheus.yml`):**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert-rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "alertmanager:9093"

scrape_configs:
  - job_name: "spring-boot-app"
    metrics_path: "/actuator/prometheus"
    scrape_interval: 10s
    static_configs:
      - targets:
          - "app1:8080"
          - "app2:8080"
        labels:
          environment: "production"

  # Service discovery com Kubernetes
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

**Regras de alerta (`alert-rules.yml`):**

```yaml
groups:
  - name: spring-boot-performance
    rules:
      # Alerta: P99 de latência HTTP acima de 500ms
      - alert: HighP99Latency
        expr: |
          histogram_quantile(0.99,
            rate(http_server_requests_seconds_bucket{status!~"5.."}[5m])
          ) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "P99 de latência acima de 500ms"
          description: "Aplicação {{ $labels.application }} com P99={{ $value | humanizeDuration }} no endpoint {{ $labels.uri }}"

      # Alerta: taxa de erro HTTP 5xx acima de 1%
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m])) by (application)
            /
            sum(rate(http_server_requests_seconds_count[5m])) by (application)
          ) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Taxa de erro acima de 1%"
          description: "Aplicação {{ $labels.application }} com taxa de erro={{ $value | humanizePercentage }}"

      # Alerta: pool de conexões HikariCP esgotando
      - alert: HikariCPPoolExhausted
        expr: |
          hikaricp_connections_pending{} > 5
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Pool de conexões HikariCP com fila de espera"
          description: "{{ $labels.pool }} com {{ $value }} threads aguardando conexão"

      # Alerta: pausas de GC longas
      - alert: LongGCPauses
        expr: |
          rate(jvm_gc_pause_seconds_sum[5m])
          /
          rate(jvm_gc_pause_seconds_count[5m]) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pausas de GC acima de 500ms em média"
          description: "Aplicação {{ $labels.application }} com GC médio={{ $value | humanizeDuration }}"

      # Alerta: heap acima de 90%
      - alert: HighHeapUsage
        expr: |
          jvm_memory_used_bytes{area="heap"}
          /
          jvm_memory_max_bytes{area="heap"} > 0.9
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Uso de heap acima de 90%"
          description: "Aplicação {{ $labels.application }} com {{ $value | humanizePercentage }} de heap utilizado"
```

### 24.4 Grafana — Dashboards de Performance

O Grafana permite criar dashboards que combinam métricas do Prometheus com alertas visuais. Dois métodos amplamente adotados para monitoramento de microsserviços são o **RED** e o **USE**.

**Método RED (para serviços request-driven):**

| Sigla | Significado | Métrica                                        |
|-------|-------------|------------------------------------------------|
| **R** | Rate        | Taxa de requisições por segundo                |
| **E** | Errors      | Taxa de erros                                  |
| **D** | Duration    | Latência (percentis P50, P95, P99)             |

**Método USE (para recursos de infraestrutura):**

| Sigla | Significado  | Recurso                                         |
|-------|--------------|------------------------------------------------|
| **U** | Utilization  | % de uso (CPU, memória, pool de conexões)       |
| **S** | Saturation   | Fila de espera (threads pending, queue size)    |
| **E** | Errors       | Erros do recurso (timeout, rejeições)           |

**Queries PromQL para painéis do Grafana:**

```promql
# ---- RED: Request Rate (requisições por segundo) ----
sum(rate(http_server_requests_seconds_count{application="$app"}[5m])) by (uri, method)

# ---- RED: Error Rate (% de erros) ----
sum(rate(http_server_requests_seconds_count{application="$app", status=~"5.."}[5m]))
/
sum(rate(http_server_requests_seconds_count{application="$app"}[5m]))

# ---- RED: Duration (latência P50, P95, P99) ----
histogram_quantile(0.50, sum(rate(http_server_requests_seconds_bucket{application="$app"}[5m])) by (le, uri))
histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket{application="$app"}[5m])) by (le, uri))
histogram_quantile(0.99, sum(rate(http_server_requests_seconds_bucket{application="$app"}[5m])) by (le, uri))

# ---- USE: JVM Heap Utilization ----
jvm_memory_used_bytes{application="$app", area="heap"}
/
jvm_memory_max_bytes{application="$app", area="heap"}

# ---- USE: GC Pauses (duração média por intervalo) ----
rate(jvm_gc_pause_seconds_sum{application="$app"}[5m])
/
rate(jvm_gc_pause_seconds_count{application="$app"}[5m])

# ---- USE: GC Pause Rate (pausas por minuto) ----
rate(jvm_gc_pause_seconds_count{application="$app"}[5m]) * 60

# ---- USE: Thread Count ----
jvm_threads_live_threads{application="$app"}

# ---- USE: HikariCP Utilization ----
hikaricp_connections_active{application="$app"}
/
hikaricp_connections_max{application="$app"}

# ---- USE: HikariCP Saturation (threads esperando conexão) ----
hikaricp_connections_pending{application="$app"}

# ---- USE: HikariCP Errors (timeouts de conexão) ----
rate(hikaricp_connections_timeout_total{application="$app"}[5m])

# ---- Métricas customizadas ----
# Latência P99 do processamento de pedidos
histogram_quantile(0.99, sum(rate(pedido_processamento_seconds_bucket{application="$app"}[5m])) by (le))

# Taxa de tarefas rejeitadas na fila
rate(fila_processamento_total{application="$app", resultado="rejeitada"}[5m])
```

**Painéis recomendados para um dashboard de performance:**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Dashboard: Application Performance                   │
├──────────────────────┬──────────────────────┬───────────────────────────┤
│  Request Rate (rps)  │  Error Rate (%)      │  P99 Latency (ms)        │
│  ████████████████    │  ▁▁▁▁▂▁▁▁▁▁         │  ▁▁▁▃▁▁▁▁▁▁             │
├──────────────────────┴──────────────────────┴───────────────────────────┤
│                        Latency Heatmap (by endpoint)                    │
│  /api/pedidos    ░░░░░░▓▓▓▓████░░░░░░░░                                │
│  /api/produtos   ░░░░░░░░░▓▓░░░░░░░░░░                                │
│  /api/clientes   ░░░░░░░░▓░░░░░░░░░░░░                                │
├──────────────────────┬──────────────────────┬───────────────────────────┤
│  JVM Heap Used (MB)  │  GC Pause Time (ms)  │  Live Threads            │
│  ████████████████    │  ▁▁▁▃▁▁▁▁▃▁         │  ════════════════════     │
├──────────────────────┴──────────────────────┴───────────────────────────┤
│                    HikariCP Connection Pool                             │
│  Active: 8/20 ████████░░░░░░░░░░░░  │  Pending: 0  │  Timeouts: 0     │
└─────────────────────────────────────────────────────────────────────────┘
```

> Para detalhes sobre logging estruturado e infraestrutura de observabilidade com Loki e Tempo, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

---

## 25. Distributed Tracing com OpenTelemetry

Em arquiteturas de microsserviços, uma única requisição do usuário pode atravessar dezenas de serviços. O distributed tracing permite acompanhar essa requisição de ponta a ponta, medindo a latência de cada etapa e identificando gargalos.

### 25.1 Conceito de Distributed Tracing

O distributed tracing funciona com três conceitos fundamentais:

- **Trace**: representa o caminho completo de uma requisição, do início ao fim. Cada trace tem um ID único (trace ID).
- **Span**: representa uma operação individual dentro do trace (ex.: uma chamada HTTP, uma query de banco). Cada span tem um span ID, um parent span ID e timestamps de início/fim.
- **Context Propagation**: mecanismo que transmite o trace ID e o span ID entre serviços, tipicamente via headers HTTP (`traceparent`, `tracestate`).

```
Requisição do usuário
│
▼
┌──────────────────────────────────────────────────────────────────┐
│ Trace ID: abc123                                                 │
│                                                                  │
│ ┌──────────────────────────────────────────────────────────────┐ │
│ │ Span A: API Gateway (12ms)                                   │ │
│ │ span_id=s1, parent=none                                      │ │
│ │  ┌────────────────────────────────────────────────────────┐   │ │
│ │  │ Span B: Pedido Service (45ms)                         │   │ │
│ │  │ span_id=s2, parent=s1                                 │   │ │
│ │  │  ┌──────────────────────┐  ┌────────────────────────┐ │   │ │
│ │  │  │ Span C: DB Query    │  │ Span D: Pagamento Svc  │ │   │ │
│ │  │  │ (8ms)               │  │ (30ms)                  │ │   │ │
│ │  │  │ span_id=s3          │  │ span_id=s4              │ │   │ │
│ │  │  │ parent=s2           │  │ parent=s2               │ │   │ │
│ │  │  └──────────────────────┘  │  ┌──────────────────┐  │ │   │ │
│ │  │                            │  │ Span E: Gateway   │  │ │   │ │
│ │  │                            │  │ API (25ms)        │  │ │   │ │
│ │  │                            │  │ span_id=s5        │  │ │   │ │
│ │  │                            │  │ parent=s4         │  │ │   │ │
│ │  │                            │  └──────────────────┘  │ │   │ │
│ │  │                            └────────────────────────┘ │   │ │
│ │  └────────────────────────────────────────────────────────┘   │ │
│ └──────────────────────────────────────────────────────────────┘ │
│                                                                  │
│ Timeline:                                                        │
│ |--A(12ms)--------------------------------------------------|   │
│    |--B(45ms)-----------------------------------------------|   │
│       |--C(8ms)--|  |--D(30ms)-----------------------------|   │
│                        |--E(25ms)------------------------|       │
└──────────────────────────────────────────────────────────────────┘
```

O diagrama mostra que o Span D (Pagamento Service) domina a latência total. Dentro dele, o Span E (chamada ao gateway de pagamento) é o verdadeiro gargalo.

### 25.2 Configuração com Micrometer Tracing + OTLP

O Spring Boot 3.x integra tracing via Micrometer Tracing, que suporta exportação no formato OpenTelemetry Protocol (OTLP).

**Dependências:**

```xml
<dependencies>
    <!-- Actuator (inclui Micrometer) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Micrometer Tracing com bridge OpenTelemetry -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-otel</artifactId>
    </dependency>

    <!-- Exportador OTLP (envia traces para Jaeger, Tempo, etc.) -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-otlp</artifactId>
    </dependency>
</dependencies>
```

**Configuração (`application.yml`):**

```yaml
management:
  tracing:
    sampling:
      probability: 1.0        # 1.0 = 100% dos traces (desenvolvimento)
                               # 0.1 = 10% dos traces (produção)
  otlp:
    tracing:
      endpoint: http://otel-collector:4318/v1/traces

# Incluir trace ID nos logs automaticamente
logging:
  pattern:
    correlation: "[${spring.application.name:},%mdc{traceId:-},%mdc{spanId:-}] "
```

**Configuração por ambiente (perfis Spring):**

```yaml
# application-dev.yml
management:
  tracing:
    sampling:
      probability: 1.0   # 100% em desenvolvimento — ver todos os traces

---
# application-prod.yml
management:
  tracing:
    sampling:
      probability: 0.05  # 5% em produção — balancear custo vs visibilidade
```

**Propagação automática:** o Spring Boot propaga automaticamente o contexto de tracing via headers HTTP (`W3C Trace Context`) quando se usa `RestClient`, `WebClient` ou `RestTemplate` com bean gerenciado pelo Spring.

```java
// RestClient criado via builder do Spring — propagação automática
@Bean
public RestClient restClient(RestClient.Builder builder) {
    return builder
            .baseUrl("http://pagamento-service:8080")
            .build();
}
```

### 25.3 Instrumentação Automática vs Manual

**Instrumentação automática com `@Observed`:**

A anotação `@Observed` do Micrometer cria spans automaticamente para métodos anotados, além de registrar métricas de latência.

```java
import io.micrometer.observation.annotation.Observed;
import org.springframework.stereotype.Service;

@Service
public class PedidoService {

    // Cria span + timer automaticamente
    @Observed(name = "pedido.processar",
              contextualName = "processar-pedido",
              lowCardinalityKeyValues = {"tipo", "novo"})
    public Pedido processar(CriarPedidoRequest request) {
        Pedido pedido = criarPedido(request);
        calcularFrete(pedido);
        reservarEstoque(pedido);
        return pedido;
    }

    @Observed(name = "pedido.calcular-frete",
              contextualName = "calcular-frete")
    public void calcularFrete(Pedido pedido) {
        // Cria span filho automaticamente
        // ...
    }
}
```

Para que `@Observed` funcione, é necessário registrar o `ObservedAspect`:

```java
import io.micrometer.observation.ObservationRegistry;
import io.micrometer.observation.aop.ObservedAspect;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ObservationConfig {

    @Bean
    ObservedAspect observedAspect(ObservationRegistry registry) {
        return new ObservedAspect(registry);
    }
}
```

**Instrumentação manual com Observation API:**

Para cenários mais complexos onde se precisa de controle fino sobre spans — por exemplo, adicionar atributos condicionais ou criar spans dentro de loops.

```java
import io.micrometer.observation.Observation;
import io.micrometer.observation.ObservationRegistry;
import org.springframework.stereotype.Service;

@Service
public class IntegracaoService {

    private final ObservationRegistry observationRegistry;
    private final RestClient restClient;

    public IntegracaoService(ObservationRegistry observationRegistry,
                             RestClient restClient) {
        this.observationRegistry = observationRegistry;
        this.restClient = restClient;
    }

    public RespostaPagamento processarPagamento(Pedido pedido) {
        // Criar span manual com atributos customizados
        return Observation.createNotStarted("integracao.pagamento", observationRegistry)
                .lowCardinalityKeyValue("gateway", pedido.getGateway().name())
                .lowCardinalityKeyValue("moeda", pedido.getMoeda())
                .highCardinalityKeyValue("pedido.id", pedido.getId().toString())
                .observe(() -> {
                    RespostaPagamento resposta = restClient.post()
                            .uri("/api/pagamentos")
                            .body(new PagamentoRequest(pedido))
                            .retrieve()
                            .body(RespostaPagamento.class);

                    // Adicionar resultado ao span
                    if (resposta != null && !resposta.isAprovado()) {
                        Observation.Event.of("pagamento.recusado",
                                "Pagamento recusado: " + resposta.getMotivo());
                    }

                    return resposta;
                });
    }

    public List<Resultado> processarLote(List<Item> itens) {
        return Observation.createNotStarted("lote.processamento", observationRegistry)
                .lowCardinalityKeyValue("tamanho.faixa", categorizarTamanho(itens.size()))
                .observe(() -> {
                    List<Resultado> resultados = new ArrayList<>();

                    for (Item item : itens) {
                        // Span filho para cada item
                        Resultado resultado = Observation.createNotStarted(
                                    "lote.item", observationRegistry)
                                .highCardinalityKeyValue("item.id", item.getId().toString())
                                .observe(() -> processarItem(item));

                        resultados.add(resultado);
                    }

                    return resultados;
                });
    }

    private String categorizarTamanho(int tamanho) {
        if (tamanho <= 10) return "pequeno";
        if (tamanho <= 100) return "medio";
        return "grande";
    }
}
```

### 25.4 Identificando Gargalos em Sistemas Distribuídos

**Usando traces para encontrar serviços lentos:**

Em ferramentas como Jaeger, Tempo ou Zipkin, a visualização de um trace mostra a cascata de spans com suas durações. Os padrões mais comuns de gargalo são:

| Padrão no trace                         | Diagnóstico                                   | Ação                                       |
|-----------------------------------------|-----------------------------------------------|--------------------------------------------|
| Span de DB query muito longo            | Query lenta ou falta de índice                | Analisar com EXPLAIN, criar índice         |
| Span de chamada HTTP externa longo      | Serviço downstream lento ou rede instável     | Adicionar timeout, circuit breaker, cache  |
| Muitos spans sequenciais para o mesmo   | Problema N+1 em chamadas de serviço           | Implementar batch endpoint                 |
| Gap grande entre spans (tempo sem span) | Processamento interno pesado não instrumentado| Adicionar spans manuais para investigar    |
| Span de serialização/deserialização     | Payload grande ou serialização ineficiente    | Reduzir payload, usar streaming            |

**Correlacionando traces com logs via trace ID:**

A combinação de traces com logs permite navegar do trace para a linha exata do log e vice-versa.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.stereotype.Service;

@Service
public class PedidoService {

    private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

    public Pedido processar(CriarPedidoRequest request) {
        // O trace ID já está no MDC automaticamente (Spring Boot 3.x)
        log.info("Iniciando processamento do pedido para cliente={}",
                request.clienteId());

        // Em ferramentas como Grafana, é possível clicar no trace ID
        // do log e navegar diretamente para o trace no Tempo/Jaeger
        try {
            Pedido pedido = criarPedido(request);
            log.info("Pedido criado: id={}, total={}", pedido.getId(), pedido.getTotal());
            return pedido;
        } catch (Exception e) {
            // O trace ID no log ajuda a encontrar o trace exato do erro
            log.error("Erro ao processar pedido para cliente={}: {}",
                    request.clienteId(), e.getMessage(), e);
            throw e;
        }
    }
}
```

**Exemplo de log com trace ID correlacionado:**

```
2024-03-15 14:23:45.123 [minha-app,abc123def456,span789] INFO  PedidoService - Iniciando processamento do pedido para cliente=42
2024-03-15 14:23:45.234 [minha-app,abc123def456,span790] INFO  PedidoService - Pedido criado: id=1001, total=299.90
```

No Grafana, é possível configurar data links que conectam logs no Loki com traces no Tempo usando o trace ID, permitindo navegação bidirecional.

> Para detalhes sobre configuração de logs estruturados, MDC e infraestrutura com Loki e Tempo, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

---

## 26. Spring Boot Actuator — Endpoints de Performance

O Spring Boot Actuator expõe endpoints HTTP que fornecem informações operacionais sobre a aplicação em execução. Para diagnóstico de performance, os endpoints de thread dump, heap dump e métricas são particularmente valiosos.

### 26.1 Endpoints Essenciais

**Configuração para habilitar endpoints de diagnóstico:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus, threaddump, heapdump, caches, scheduledtasks, env, conditions
  endpoint:
    health:
      show-details: when-authorized    # detalhes apenas para autenticados
    heapdump:
      enabled: true                    # desabilitar em produção se não necessário
```

| Endpoint                     | Método | Descrição                                                    | Uso em performance                           |
|------------------------------|--------|--------------------------------------------------------------|----------------------------------------------|
| `/actuator/health`           | GET    | Estado de saúde da aplicação e suas dependências             | Verificar conectividade com DB, Redis, disco |
| `/actuator/metrics`          | GET    | Lista todas as métricas disponíveis                          | Descobrir métricas para monitoramento        |
| `/actuator/metrics/{nome}`   | GET    | Valor atual de uma métrica específica                        | Consultar métricas pontuais                  |
| `/actuator/prometheus`       | GET    | Todas as métricas em formato Prometheus                      | Scraping pelo Prometheus                     |
| `/actuator/threaddump`       | GET    | Stack trace de todas as threads ativas                       | Diagnosticar deadlocks e contenção           |
| `/actuator/heapdump`         | GET    | Download do heap dump (arquivo `.hprof`)                     | Diagnosticar memory leaks                    |
| `/actuator/caches`           | GET    | Caches registrados e seus nomes                              | Verificar e gerenciar caches                 |
| `/actuator/scheduledtasks`   | GET    | Tarefas agendadas (`@Scheduled`) e suas configurações        | Verificar cron expressions e intervalos      |

**Consultando métricas via endpoint:**

```bash
# Listar todas as métricas disponíveis
curl http://localhost:8080/actuator/metrics | jq '.names[]' | sort

# Consultar métrica específica
curl http://localhost:8080/actuator/metrics/jvm.memory.used | jq

# Filtrar por tag (ex.: apenas heap)
curl "http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap" | jq

# Verificar latência HTTP por endpoint
curl "http://localhost:8080/actuator/metrics/http.server.requests?tag=uri:/api/pedidos&tag=method:GET" | jq
```

### 26.2 Thread Dump — Diagnosticando Deadlocks e Contenção

O thread dump mostra o estado e o stack trace de todas as threads da JVM em um instante específico. É a ferramenta principal para diagnosticar deadlocks, contenção de locks e threads bloqueadas.

**Obtendo o thread dump:**

```bash
# Via Actuator (formato JSON)
curl http://localhost:8080/actuator/threaddump | jq

# Via Actuator (formato texto, mais legível)
curl -H "Accept: text/plain" http://localhost:8080/actuator/threaddump

# Via jstack (alternativa sem Actuator)
jstack <PID> > threaddump.txt

# Via jcmd
jcmd <PID> Thread.print > threaddump.txt
```

**Estados das threads e o que significam:**

| Estado            | Significado                                                       | Preocupação                                |
|-------------------|-------------------------------------------------------------------|--------------------------------------------|
| `RUNNABLE`        | Thread executando ou pronta para executar                         | Normal se não houver muitas                |
| `BLOCKED`         | Esperando adquirir um lock (`synchronized`)                       | Alta — indica contenção de lock            |
| `WAITING`         | Esperando indefinidamente (`Object.wait()`, `LockSupport.park()`) | Verificar se é intencional                 |
| `TIMED_WAITING`   | Esperando com timeout (`Thread.sleep()`, `Lock.tryLock(timeout)`) | Normal para pools e schedulers             |

**Padrões problemáticos comuns:**

```
# PADRÃO 1: Deadlock (duas threads esperando pelo lock uma da outra)
"http-nio-8080-exec-1" BLOCKED
   waiting to lock <0x000000076ab00208> (a com.example.ServiceA)
   which is held by "http-nio-8080-exec-2"

"http-nio-8080-exec-2" BLOCKED
   waiting to lock <0x000000076ab00310> (a com.example.ServiceB)
   which is held by "http-nio-8080-exec-1"

# PADRÃO 2: Contenção de lock (muitas threads BLOCKED no mesmo monitor)
"http-nio-8080-exec-1" BLOCKED
   waiting to lock <0x000000076ab00208> (a com.example.CacheService)
"http-nio-8080-exec-2" BLOCKED
   waiting to lock <0x000000076ab00208> (a com.example.CacheService)
"http-nio-8080-exec-3" BLOCKED
   waiting to lock <0x000000076ab00208> (a com.example.CacheService)
→ Solução: substituir synchronized por ConcurrentHashMap ou ReentrantReadWriteLock

# PADRÃO 3: Pool de conexões esgotado (threads esperando por conexão)
"http-nio-8080-exec-5" TIMED_WAITING
   at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:181)
   - waiting on <0x000000076cd00100> (a java.util.concurrent.SynchronousQueue)
→ Solução: aumentar pool, verificar queries lentas, verificar connection leak

# PADRÃO 4: Thread starvation (todas as threads do pool ocupadas)
"http-nio-8080-exec-1" RUNNABLE (em operação I/O lenta)
"http-nio-8080-exec-2" RUNNABLE (em operação I/O lenta)
...todos os 200 threads ocupados...
→ Solução: usar async para I/O, aumentar pool, adicionar timeout
```

**Análise automática de deadlocks com jstack:**

```bash
# jstack detecta deadlocks automaticamente
jstack <PID> | grep -A 30 "Found.*deadlock"

# Exemplo de saída:
# Found one Java-level deadlock:
# =============================
# "http-nio-8080-exec-1":
#   waiting to lock monitor 0x00007f8b2c003a18
#   which is held by "http-nio-8080-exec-2"
# "http-nio-8080-exec-2":
#   waiting to lock monitor 0x00007f8b2c003b28
#   which is held by "http-nio-8080-exec-1"
```

### 26.3 Heap Dump — Diagnosticando Memory Leaks

O heap dump é um snapshot de todos os objetos na memória da JVM. É o recurso definitivo para diagnosticar memory leaks e entender o consumo de memória.

**Obtendo o heap dump:**

```bash
# Via Actuator (download do arquivo .hprof)
curl -o heapdump.hprof http://localhost:8080/actuator/heapdump

# Via jcmd
jcmd <PID> GC.heap_dump /tmp/heapdump.hprof

# Gerar heap dump automaticamente em OutOfMemoryError
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/log/app/heapdump.hprof \
     -jar minha-app.jar
```

> **Cuidado em produção:** o heap dump pausa a JVM durante a geração e o arquivo pode ter vários gigabytes. Gere apenas quando necessário e em horários de baixo tráfego.

**Analisando com Eclipse MAT (Memory Analyzer Tool):**

O Eclipse MAT é a ferramenta mais usada para analisar heap dumps. Passos:

1. Baixe o Eclipse MAT em [https://eclipse.dev/mat/](https://eclipse.dev/mat/).
2. Abra o arquivo `.hprof`.
3. Execute o **Leak Suspects Report** (gerado automaticamente na abertura).
4. Use a visão **Dominator Tree** para ver os maiores objetos retidos.
5. Use **Histogram** para ver a contagem e tamanho por classe.
6. Use **Path to GC Roots** (excluindo referências fracas) para entender por que um objeto não é coletado.

**Padrões comuns de memory leak:**

| Padrão                              | Sintoma no MAT                                           | Causa típica                                    |
|--------------------------------------|----------------------------------------------------------|-------------------------------------------------|
| Coleção crescendo sem limite         | `ArrayList` ou `HashMap` com milhões de entradas         | Cache sem eviction, lista que nunca é limpa     |
| Objetos de sessão acumulando         | `ConcurrentHashMap` grande em `HttpSession`              | Sessões sem timeout, atributos pesados          |
| Listeners não removidos              | Muitas instâncias de `EventListener` retidas             | `addEventListener` sem `removeEventListener`    |
| ThreadLocal não limpo                | Objetos retidos por `ThreadLocal` em threads de pool     | Faltou `remove()` no finally                    |
| String duplicadas                    | Milhares de `String` idênticas                           | Usar `String.intern()` ou cache de strings      |
| ClassLoader leak                     | Classes carregadas repetidamente                         | Redeploy sem limpar referências                 |

**Comparando heap dumps para identificar crescimento:**

```bash
# Gerar dois heap dumps com intervalo
jcmd <PID> GC.heap_dump /tmp/heap1.hprof
# Aguardar e reproduzir o cenário de leak
jcmd <PID> GC.heap_dump /tmp/heap2.hprof
```

No Eclipse MAT, use **Compare Baskets** para comparar os dois dumps e identificar quais classes tiveram crescimento de instâncias entre os dois snapshots.

**Alternativa com VisualVM:**

O VisualVM (incluído no JDK ou disponível separadamente) oferece uma interface mais simples para análise rápida:

```bash
# Abrir VisualVM e conectar ao processo
jvisualvm

# Ou com o VisualVM standalone
visualvm --jdkhome /path/to/jdk
```

No VisualVM, a aba **Sampler > Memory** mostra alocações em tempo real, e a aba **Monitor** mostra o heap ao longo do tempo — útil para confirmar se o heap usado após GC está crescendo (indicador de leak).

### 26.4 Protegendo Endpoints em Produção

Os endpoints do Actuator expõem informações sensíveis e operações potencialmente perigosas (como heap dump e shutdown). Em produção, é essencial protegê-los com Spring Security.

```java
import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class ActuatorSecurityConfig {

    /**
     * Regras de segurança específicas para endpoints do Actuator.
     * @Order(1) garante que essa cadeia é avaliada antes da cadeia principal.
     */
    @Bean
    @Order(1)
    public SecurityFilterChain actuatorSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher(EndpointRequest.toAnyEndpoint())
            .authorizeHttpRequests(auth -> auth
                // health e info são públicos (usados por load balancers e k8s probes)
                .requestMatchers(EndpointRequest.to("health", "info"))
                    .permitAll()
                // prometheus é acessado pelo Prometheus scraper (pode restringir por IP)
                .requestMatchers(EndpointRequest.to("prometheus"))
                    .permitAll()
                // endpoints sensíveis exigem role ACTUATOR
                .requestMatchers(EndpointRequest.to(
                        "threaddump", "heapdump", "env", "configprops", "beans"))
                    .hasRole("ACTUATOR")
                // todos os demais endpoints exigem autenticação
                .anyRequest()
                    .authenticated()
            )
            .httpBasic(basic -> {}); // Basic Auth para acesso programático

        return http.build();
    }
}
```

**Configuração complementar (`application-prod.yml`):**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus, threaddump, heapdump, caches
      # Usar porta separada para Actuator (não expor na porta pública)
      base-path: /actuator
  server:
    port: 9090    # Porta separada para management (não acessível externamente)
  endpoint:
    shutdown:
      enabled: false    # NUNCA habilitar shutdown via HTTP em produção
    heapdump:
      enabled: true
    threaddump:
      enabled: true
  health:
    # Livenessstate e readinessstate para Kubernetes
    probes:
      enabled: true
```

**Usando porta separada (management port):**

```yaml
# A porta de management fica acessível apenas na rede interna
# O load balancer/ingress expõe apenas a porta 8080
management:
  server:
    port: 9090
    address: 0.0.0.0
```

```
┌─────────────────────────────────────────────────┐
│              Kubernetes / Docker                 │
│                                                  │
│  Internet ──► Ingress ──► :8080 (API pública)   │
│                                                  │
│  Rede interna:                                   │
│    Prometheus ──► :9090/actuator/prometheus      │
│    kubectl ──► :9090/actuator/health             │
│    SRE team ──► :9090/actuator/threaddump        │
└─────────────────────────────────────────────────┘
```

Essa separação garante que endpoints de diagnóstico não sejam acessíveis pela internet, mesmo em caso de misconfiguration do ingress.

> Para detalhes sobre Actuator, health indicators e métricas customizadas com Micrometer, consulte [Spring-Boot-Avancado.md](../Spring-Boot-Avancado.md).

---

**Parte 6 — Benchmarking e Testes de Carga**

## 27. Microbenchmarks com JMH

### 27.1 O que é JMH e Quando Usar

**JMH (Java Microbenchmark Harness)** é a ferramenta oficial da OpenJDK para medir o desempenho de trechos isolados de código Java. Antes de recorrer a ela, é importante entender por que abordagens ingênuas falham.

**Por que `System.currentTimeMillis()` não é confiável:**

| Problema | Explicação |
|---|---|
| **JVM warmup** | O JIT compiler otimiza bytecode em tempo de execução; as primeiras execuções são interpretadas e mais lentas. Medir sem aquecer a JVM captura performance não-representativa. |
| **Dead code elimination** | O compilador JIT pode remover código cujo resultado nunca é utilizado, fazendo o benchmark medir "nada". |
| **Constant folding** | Se o compilador detecta que o resultado é previsível, ele substitui o cálculo pelo valor constante em tempo de compilação. |
| **Resolução do relógio** | `System.currentTimeMillis()` tem resolução de ~15 ms no Windows, insuficiente para operações de nanossegundos. |
| **GC interference** | Pausas do Garbage Collector podem distorcer medições individuais. |

O JMH contorna todos esses problemas com:
- **Warmup iterations** configuráveis para estabilizar o JIT.
- **Blackhole** — consome resultados para evitar dead code elimination.
- **Forks** — executa cada benchmark em uma JVM separada para isolar efeitos de compilação.
- **Estatísticas robustas** — calcula média, desvio padrão e percentis.

**Quando usar JMH:**
- Comparar duas implementações de um mesmo algoritmo.
- Avaliar impacto de diferentes estruturas de dados.
- Medir overhead de serialização/desserialização.
- Validar hipóteses antes de otimizar código em produção.

**Quando NÃO usar JMH:**
- Para medir performance de ponta a ponta de uma aplicação (use testes de carga — Seção 28).
- Para monitorar performance em produção (use métricas e profiling).

### 27.2 Configuração do Projeto

Adicione as dependências do JMH ao `pom.xml`. Recomenda-se criar um módulo separado ou um profile Maven dedicado para benchmarks:

```xml
<properties>
    <jmh.version>1.37</jmh.version>
</properties>

<dependencies>
    <!-- JMH Core -->
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-core</artifactId>
        <version>${jmh.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- JMH Annotation Processor -->
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-generator-annprocess</artifactId>
        <version>${jmh.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.6.0</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals><goal>shade</goal></goals>
                    <configuration>
                        <finalName>benchmarks</finalName>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>org.openjdk.jmh.Main</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Para executar os benchmarks:

```bash
mvn clean package -DskipTests                               # Compilar JAR
java -jar target/benchmarks.jar                              # Executar todos
java -jar target/benchmarks.jar "ListIterationBenchmark"     # Benchmark específico
java -jar target/benchmarks.jar -rf json -rff results.json   # Exportar JSON
```

### 27.3 Escrevendo Benchmarks

#### Anotações Principais

| Anotação | Função |
|---|---|
| `@Benchmark` | Marca o método como um benchmark. |
| `@BenchmarkMode` | Define o que medir: `Throughput` (ops/s), `AverageTime`, `SampleTime`, `SingleShotTime`. |
| `@OutputTimeUnit` | Unidade de tempo nos resultados (`NANOSECONDS`, `MICROSECONDS`, `MILLISECONDS`, `SECONDS`). |
| `@State` | Define escopo do estado: `Scope.Benchmark` (compartilhado), `Scope.Thread` (por thread). |
| `@Setup` | Método executado antes dos benchmarks (similar a `@BeforeEach` em testes). |
| `@TearDown` | Método executado após os benchmarks. |
| `@Fork` | Número de JVMs separadas para executar o benchmark. |
| `@Warmup` | Configuração das iterações de aquecimento. |
| `@Measurement` | Configuração das iterações de medição. |
| `@Param` | Permite parametrizar benchmarks com diferentes valores. |

#### Exemplo 1 — ArrayList vs LinkedList: Iteração e Acesso

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import java.util.*;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Benchmark)
@Fork(value = 2, jvmArgs = {"-Xms512m", "-Xmx512m"})
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 5, time = 1)
public class ListIterationBenchmark {

    @Param({"100", "10000", "1000000"})
    private int size;

    private List<Integer> arrayList;
    private List<Integer> linkedList;

    @Setup(Level.Trial)
    public void setup() {
        arrayList = new ArrayList<>(size);
        linkedList = new LinkedList<>();
        var random = new Random(42);
        for (int i = 0; i < size; i++) {
            int value = random.nextInt();
            arrayList.add(value);
            linkedList.add(value);
        }
    }

    @Benchmark
    public void iterateArrayList(Blackhole bh) {
        for (Integer value : arrayList) {
            bh.consume(value); // Blackhole evita dead code elimination
        }
    }

    @Benchmark
    public void iterateLinkedList(Blackhole bh) {
        for (Integer value : linkedList) {
            bh.consume(value);
        }
    }

    @Benchmark
    public void randomAccessArrayList(Blackhole bh) {
        var random = new Random(123);
        for (int i = 0; i < 100; i++) {
            bh.consume(arrayList.get(random.nextInt(size)));
        }
    }

    @Benchmark
    public void randomAccessLinkedList(Blackhole bh) {
        var random = new Random(123);
        for (int i = 0; i < 100; i++) {
            bh.consume(linkedList.get(random.nextInt(size)));
        }
    }

    // Permite executar via main (alternativa ao JAR)
    public static void main(String[] args) throws Exception {
        org.openjdk.jmh.Main.main(args);
    }
}
```

#### Exemplo 2 — Comparando Estratégias de Serialização

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@State(Scope.Benchmark)
@Fork(2)
@Warmup(iterations = 5, time = 2)
@Measurement(iterations = 5, time = 2)
public class SerializationBenchmark {

    private ObjectMapper objectMapper;
    private Gson gson;
    private ProdutoDTO produto;
    private String jsonString;

    public record ProdutoDTO(
        Long id,
        String nome,
        String descricao,
        double preco,
        int estoque,
        String categoria
    ) {}

    @Setup(Level.Trial)
    public void setup() throws Exception {
        objectMapper = new ObjectMapper();
        gson = new Gson();
        produto = new ProdutoDTO(
            1L, "Notebook Gamer", "Intel i7, 16GB RAM, RTX 4060",
            5499.90, 42, "Eletrônicos"
        );
        jsonString = objectMapper.writeValueAsString(produto);
    }

    // --- Serialização (Objeto -> JSON) ---

    @Benchmark
    public void jacksonSerialize(Blackhole bh) throws Exception {
        bh.consume(objectMapper.writeValueAsString(produto));
    }

    @Benchmark
    public void gsonSerialize(Blackhole bh) {
        bh.consume(gson.toJson(produto));
    }

    // --- Desserialização (JSON -> Objeto) ---

    @Benchmark
    public void jacksonDeserialize(Blackhole bh) throws Exception {
        bh.consume(objectMapper.readValue(jsonString, ProdutoDTO.class));
    }

    @Benchmark
    public void gsonDeserialize(Blackhole bh) {
        bh.consume(gson.fromJson(jsonString, ProdutoDTO.class));
    }
}
```

### 27.4 Benchmark de Código Spring Boot

Quando é necessário medir código que depende do contexto Spring (beans, repositórios, caches), o benchmark precisa inicializar o `ApplicationContext`. Isso tem custo significativo, por isso é recomendado usar `@Setup(Level.Trial)` para inicializar o contexto uma única vez.

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ConfigurableApplicationContext;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@State(Scope.Benchmark)
@Fork(1)  // Fork reduzido: inicializar Spring é custoso
@Warmup(iterations = 3, time = 2)
@Measurement(iterations = 5, time = 2)
public class SpringServiceBenchmark {

    private ConfigurableApplicationContext context;
    private ProdutoService produtoService;
    private ProdutoRepository produtoRepository;

    @Setup(Level.Trial)
    public void setup() {
        // Inicializa o contexto Spring Boot completo
        context = SpringApplication.run(Application.class,
            "--spring.profiles.active=benchmark",
            "--spring.jpa.hibernate.ddl-auto=create-drop",
            "--spring.datasource.url=jdbc:h2:mem:benchmark"
        );
        produtoService = context.getBean(ProdutoService.class);
        produtoRepository = context.getBean(ProdutoRepository.class);

        // Seed de dados para o benchmark
        for (int i = 0; i < 1000; i++) {
            produtoRepository.save(new Produto("Produto " + i, 10.0 + i));
        }
    }

    @TearDown(Level.Trial)
    public void tearDown() {
        if (context != null) {
            context.close();
        }
    }

    @Benchmark
    public void buscarProdutosPorCategoria(Blackhole bh) {
        bh.consume(produtoService.buscarPorCategoria("Eletrônicos"));
    }

    @Benchmark
    public void buscarProdutoComCache(Blackhole bh) {
        // Mede o tempo com cache (segunda chamada em diante)
        bh.consume(produtoService.buscarPorId(1L));
    }

    @Benchmark
    public void buscarProdutoSemCache(Blackhole bh) {
        // Evita cache usando IDs diferentes a cada chamada
        long id = (System.nanoTime() % 1000) + 1;
        bh.consume(produtoService.buscarPorIdSemCache(id));
    }
}
```

**Benchmark de serialização no contexto Spring** — comparando o `ObjectMapper` configurado pelo Spring (com `JavaTimeModule`, etc.) contra um `ObjectMapper` vanilla e medindo `writeValueAsBytes` vs `writeValueAsString`:

```java
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@State(Scope.Benchmark)
@Fork(1)
@Warmup(iterations = 3, time = 2)
@Measurement(iterations = 5, time = 2)
public class SpringSerializationBenchmark {

    private ConfigurableApplicationContext context;
    private ObjectMapper springObjectMapper;
    private ObjectMapper vanillaObjectMapper;
    private ProdutoDTO produto;

    @Setup(Level.Trial)
    public void setup() {
        context = SpringApplication.run(Application.class,
            "--spring.profiles.active=benchmark");
        springObjectMapper = context.getBean(ObjectMapper.class);
        vanillaObjectMapper = new ObjectMapper();
        produto = new ProdutoDTO(1L, "Monitor 4K", "32 polegadas IPS",
            2899.90, 15, "Periféricos", LocalDateTime.now());
    }

    @TearDown(Level.Trial)
    public void tearDown() { if (context != null) context.close(); }

    @Benchmark
    public void serializeSpringObjectMapper(Blackhole bh) throws Exception {
        bh.consume(springObjectMapper.writeValueAsString(produto));
    }

    @Benchmark
    public void serializeVanillaObjectMapper(Blackhole bh) throws Exception {
        bh.consume(vanillaObjectMapper.writeValueAsString(produto));
    }

    @Benchmark
    public void serializeSpringToBytes(Blackhole bh) throws Exception {
        bh.consume(springObjectMapper.writeValueAsBytes(produto));
    }
}
```

### 27.5 Interpretando Resultados

Ao executar um benchmark, o JMH produz uma saída como esta:

```
Benchmark                                    (size)  Mode  Cnt        Score        Error  Units
ListIterationBenchmark.iterateArrayList         100  avgt    5       78,234 ±      2,105  ns/op
ListIterationBenchmark.iterateArrayList       10000  avgt    5     8102,456 ±    156,789  ns/op
ListIterationBenchmark.iterateLinkedList         100  avgt    5      142,567 ±      5,432  ns/op
ListIterationBenchmark.iterateLinkedList       10000  avgt    5    45678,123 ±   2345,678  ns/op
ListIterationBenchmark.randomAccessArrayList    100  avgt    5      456,789 ±     12,345  ns/op
ListIterationBenchmark.randomAccessLinkedList    100  avgt    5     3456,789 ±    234,567  ns/op
```

**Como ler cada coluna:**

| Coluna | Significado |
|---|---|
| **Benchmark** | Nome completo do método (Classe.método). |
| **(size)** | Valor do parâmetro `@Param` utilizado nesta execução. |
| **Mode** | Modo de medição (`avgt` = tempo médio, `thrpt` = throughput). |
| **Cnt** | Número de iterações de medição. |
| **Score** | Valor medido (menor = melhor para `avgt`, maior = melhor para `thrpt`). |
| **Error** | Margem de erro (intervalo de confiança de 99,9%). |
| **Units** | Unidade: `ns/op` (nanossegundos por operação), `ops/s` (operações por segundo). |

**Percentis** — para análise de distribuição, use `@BenchmarkMode(Mode.SampleTime)`:

```java
@BenchmarkMode(Mode.SampleTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
```

Isso gera histogramas com percentis (p50, p90, p95, p99, p99.9), úteis para identificar outliers causados por GC pauses.

**`@CompilerControl`** — controle fino sobre otimizações do JIT:

```java
import org.openjdk.jmh.annotations.CompilerControl;

@CompilerControl(CompilerControl.Mode.DONT_INLINE)
private int metodoQueNaoDeveSerInlined(int x) {
    return x * x + 1;
}
```

Valores possíveis:
- `DONT_INLINE` — impede inlining do método.
- `INLINE` — força inlining.
- `EXCLUDE` — exclui o método da compilação JIT (interpreta sempre).

**Armadilhas comuns:**

1. **Medir sem warmup** — as primeiras iterações usam código interpretado, não otimizado pelo JIT.
2. **Não usar Blackhole** — o JIT remove código cujo resultado não é utilizado.
3. **Fork insuficiente** — sem fork, efeitos de compilação de um benchmark contaminam outro.
4. **Comparar benchmarks de sessões diferentes** — variações de hardware, carga do SO e temperatura da CPU afetam resultados. Execute todos os benchmarks na mesma sessão.
5. **Extrapolar micro para macro** — um método 2x mais rápido em microbenchmark pode ter impacto negligível na aplicação real se não está no hot path.

---

## 28. Testes de Carga — Gatling, k6 e JMeter

### 28.1 Por que Fazer Testes de Carga

Testes de carga simulam múltiplos usuários acessando a aplicação simultaneamente, permitindo:

- **Encontrar pontos de ruptura** — em qual volume de requisições a aplicação começa a degradar ou falhar.
- **Validar SLAs** — confirmar que o tempo de resposta p95 está dentro do acordado (ex.: < 200ms).
- **Planejamento de capacidade** — determinar quantas instâncias/pods são necessários para a carga esperada.
- **Detectar memory leaks** — em testes prolongados (soak), vazamentos de memória se manifestam.
- **Validar autoscaling** — confirmar que o HPA do Kubernetes escala corretamente sob carga.

**Tipos de teste de carga:**

| Tipo | Objetivo | Padrão de Carga | Duração Típica |
|---|---|---|---|
| **Load Test** | Validar performance sob carga esperada | Ramp-up gradual até carga nominal | 15–60 min |
| **Stress Test** | Encontrar ponto de ruptura | Ramp-up até além da capacidade | 15–30 min |
| **Soak Test** | Detectar degradação ao longo do tempo (memory leaks, connection leaks) | Carga constante e moderada | 2–8 horas |
| **Spike Test** | Validar comportamento sob picos repentinos | Salto abrupto de carga | 5–15 min |
| **Breakpoint Test** | Determinar capacidade máxima absoluta | Incremento contínuo até falha | Até falhar |

### 28.2 Gatling

**Gatling** é uma ferramenta de teste de carga escrita em Scala, mas que a partir da versão 4 oferece um **Java DSL** completo. Gera relatórios HTML detalhados automaticamente.

#### Configuração com Maven

```xml
<dependencies>
    <dependency>
        <groupId>io.gatling.highcharts</groupId>
        <artifactId>gatling-charts-highcharts</artifactId>
        <version>3.11.5</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>io.gatling</groupId>
            <artifactId>gatling-maven-plugin</artifactId>
            <version>4.9.6</version>
        </plugin>
    </plugins>
</build>
```

#### Simulação em Java (Gatling 4 Java DSL)

```java
package com.exemplo.simulations;

import io.gatling.javaapi.core.*;
import io.gatling.javaapi.http.*;
import java.time.Duration;
import static io.gatling.javaapi.core.CoreDsl.*;
import static io.gatling.javaapi.http.HttpDsl.*;

public class ProdutoApiSimulation extends Simulation {

    // Configuração do protocolo HTTP
    HttpProtocolBuilder httpProtocol = http
        .baseUrl("http://localhost:8080")
        .acceptHeader("application/json")
        .contentTypeHeader("application/json")
        .userAgentHeader("Gatling/LoadTest");

    // Feeder — dados dinâmicos para cada usuário virtual
    FeederBuilder<String> categoriaFeeder = csv("categorias.csv").random();

    // Cenário: fluxo de navegação típico
    ScenarioBuilder cenarioNavegacao = scenario("Navegação de Produtos")
        .exec(
            http("Listar Produtos")
                .get("/api/produtos")
                .queryParam("page", "0")
                .queryParam("size", "20")
                .check(status().is(200))
                .check(jsonPath("$.content").exists())
        )
        .pause(Duration.ofSeconds(1), Duration.ofSeconds(3))  // Think time
        .feed(categoriaFeeder)
        .exec(
            http("Buscar por Categoria")
                .get("/api/produtos/categoria/#{categoria}")
                .check(status().is(200))
                .check(jsonPath("$[0].id").saveAs("produtoId"))
        )
        .pause(Duration.ofMillis(500), Duration.ofSeconds(2))
        .exec(
            http("Detalhe do Produto")
                .get("/api/produtos/#{produtoId}")
                .check(status().is(200))
                .check(jsonPath("$.nome").exists())
        );

    // Cenário: operações de escrita
    ScenarioBuilder cenarioCompra = scenario("Fluxo de Compra")
        .exec(
            http("Criar Pedido")
                .post("/api/pedidos")
                .body(StringBody("{\"clienteId\": #{randomInt(1,100)}, \"itens\": [{\"produtoId\": #{randomInt(1,50)}, \"quantidade\": 1}]}"))
                .check(status().is(201))
                .check(jsonPath("$.id").saveAs("pedidoId"))
        )
        .pause(1)
        .exec(http("Consultar Pedido").get("/api/pedidos/#{pedidoId}").check(status().is(200)));

    {
        setUp(
            cenarioNavegacao.injectOpen(
                rampUsers(80).during(Duration.ofMinutes(2)),
                constantUsersPerSec(10).during(Duration.ofMinutes(5))
            ),
            cenarioCompra.injectOpen(
                rampUsers(20).during(Duration.ofMinutes(2)),     // 20% escrita
                constantUsersPerSec(2).during(Duration.ofMinutes(5))
            )
        )
        .protocols(httpProtocol)
        .assertions(
            global().responseTime().percentile3().lt(500),    // p95 < 500ms
            global().responseTime().percentile4().lt(1000),   // p99 < 1s
            global().successfulRequests().percent().gt(99.0),
            global().requestsPerSec().gte(50.0)
        );
    }
}
```

#### Executando e Lendo o Relatório

```bash
# Executar via Maven (relatório HTML gerado em target/gatling/)
mvn gatling:test

# Executar simulação específica
mvn gatling:test -Dgatling.simulationClass=com.exemplo.simulations.ProdutoApiSimulation
```

O relatório HTML inclui: requisições totais, taxa de sucesso, histograma de tempos de resposta, percentis (p50, p75, p95, p99) ao longo do tempo, gráfico de usuários ativos e throughput (req/s).

### 28.3 k6

**k6** (da Grafana Labs) é uma ferramenta moderna de teste de carga escrita em Go, com scripts em JavaScript/TypeScript. É leve, fácil de integrar em CI/CD e tem excelente saída para terminal.

#### Instalação

```bash
# macOS / Linux (Homebrew)
brew install k6

# Windows (Chocolatey)
choco install k6

# Docker
docker run --rm -i grafana/k6 run - <script.js
```

#### Script de Teste com Stages e Thresholds

```javascript
import http from 'k6/http';
import { check, group, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const listaTrend = new Trend('lista_produtos_duration');

export const options = {
  stages: [
    { duration: '1m',  target: 20  },  // Ramp-up
    { duration: '3m',  target: 50  },  // Ramp-up
    { duration: '5m',  target: 50  },  // Sustenta
    { duration: '2m',  target: 100 },  // Spike
    { duration: '3m',  target: 100 },  // Sustenta pico
    { duration: '2m',  target: 0   },  // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.01'],
    errors: ['rate<0.05'],
    lista_produtos_duration: ['p(95)<300'],
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:8080';

export function setup() {
  const loginRes = http.post(`${BASE_URL}/api/auth/login`, JSON.stringify({
    email: 'loadtest@exemplo.com', senha: 'senha123',
  }), { headers: { 'Content-Type': 'application/json' } });
  return { token: loginRes.json('token') };
}

export default function (data) {
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${data.token}`,
  };

  group('Navegação', () => {
    const listRes = http.get(`${BASE_URL}/api/produtos?page=0&size=20`, { headers });
    listaTrend.add(listRes.timings.duration);
    check(listRes, {
      'lista: status 200': (r) => r.status === 200,
      'lista: tem conteúdo': (r) => r.json('content') !== null,
    }) || errorRate.add(1);

    sleep(Math.random() * 2 + 1);

    const produtoId = Math.floor(Math.random() * 50) + 1;
    const detailRes = http.get(`${BASE_URL}/api/produtos/${produtoId}`, { headers });
    check(detailRes, {
      'detalhe: status 200': (r) => r.status === 200,
      'detalhe: tem nome': (r) => r.json('nome') !== undefined,
    }) || errorRate.add(1);

    sleep(Math.random() * 1.5 + 0.5);
  });

  group('Busca', () => {
    const categorias = ['Eletrônicos', 'Livros', 'Roupas', 'Alimentos'];
    const cat = categorias[Math.floor(Math.random() * categorias.length)];
    const res = http.get(`${BASE_URL}/api/produtos/busca?categoria=${encodeURIComponent(cat)}`, { headers });
    check(res, { 'busca: status 200': (r) => r.status === 200 }) || errorRate.add(1);
    sleep(Math.random() * 2 + 1);
  });
}
```

#### Executando o k6

```bash
k6 run load-test.js                                          # Executar script
k6 run -e BASE_URL=https://staging.exemplo.com load-test.js  # URL customizada
k6 run --out json=results.json load-test.js                  # Exportar JSON
k6 run --out influxdb=http://localhost:8086/k6 load-test.js  # Integração Grafana
k6 run --vus 10 --duration 30s load-test.js                  # Override rápido
```

### 28.4 JMeter

**Apache JMeter** é a ferramenta mais antiga e com maior base de usuários. Possui GUI para construção de planos de teste, mas deve ser executada via **CLI em produção e CI/CD** (a GUI consome recursos excessivos e distorce resultados). Use JMeter quando a equipe já tem experiência ou quando precisa testar protocolos não-HTTP (JDBC, LDAP, FTP, SMTP, JMS).

```bash
# Executar plano de teste sem GUI (-n = non-GUI, obrigatório para CI/CD)
jmeter -n -t plano-de-teste.jmx -l resultados.jtl -e -o relatorio-html/

# Passando propriedades customizadas
jmeter -n -t plano.jmx -Jthreads=100 -Jrampup=60 -Jduration=300
```

### 28.5 Comparação entre Ferramentas

| Critério | **Gatling** | **k6** | **JMeter** |
|---|---|---|---|
| **Linguagem dos scripts** | Java / Scala / Kotlin | JavaScript / TypeScript | XML (GUI) / Groovy |
| **Protocolos** | HTTP, WebSocket, SSE, JMS | HTTP, WebSocket, gRPC | HTTP, JDBC, LDAP, FTP, SMTP, JMS, TCP |
| **Relatórios** | HTML embutido (excelente) | Terminal + JSON/CSV (integrável com Grafana) | HTML gerado via CLI, plugins |
| **CI/CD** | Maven/Gradle plugin nativo | CLI leve, ideal para CI | CLI disponível, mas pesado |
| **Consumo de recursos** | Médio (~200-400 MB) | Muito baixo (~50-100 MB) | Alto (~500 MB+) |
| **Curva de aprendizado** | Média (DSL específico) | Baixa (JavaScript puro) | Baixa (GUI) / Média (CLI + Groovy) |
| **Scripting** | Code-first, versionável | Code-first, versionável | XML (GUI), difícil de versionar |
| **Escalabilidade** | Boa (assíncrono, Akka) | Excelente (Go runtime) | Média (threads Java) |
| **Melhor para** | Equipes Java, relatórios ricos | CI/CD, DevOps, equipes ágeis | Testes multi-protocolo, equipes legadas |

### 28.6 Integrando Testes de Carga no CI/CD

Exemplo de workflow GitHub Actions executando testes de carga com **k6** após deploy em staging:

```yaml
name: Load Tests

on:
  workflow_dispatch:
    inputs:
      target_url:
        description: 'URL alvo para o teste de carga'
        required: true
        default: 'https://staging.exemplo.com'
  workflow_run:                            # Executa após deploy em staging
    workflows: ["Deploy to Staging"]
    types: [completed]

jobs:
  load-test:
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'workflow_dispatch' || github.event.workflow_run.conclusion == 'success' }}

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Instalar k6
        run: |
          curl -sL https://github.com/grafana/k6/releases/download/v0.54.0/k6-v0.54.0-linux-amd64.tar.gz \
            | tar xz --strip-components=1
          sudo mv k6 /usr/local/bin/

      - name: Aguardar aplicação estar saudável
        run: |
          for i in $(seq 1 30); do
            curl -sf "${{ inputs.target_url || 'https://staging.exemplo.com' }}/actuator/health" && exit 0
            echo "Tentativa $i/30..." && sleep 10
          done
          echo "Timeout: aplicação não respondeu" && exit 1

      - name: Executar teste de carga
        run: |
          k6 run \
            -e BASE_URL="${{ inputs.target_url || 'https://staging.exemplo.com' }}" \
            --out json=results.json \
            --summary-export=summary.json \
            tests/load/load-test.js

      - name: Upload dos resultados
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: load-test-results-${{ github.run_number }}
          path: |
            results.json
            summary.json
```

> Para detalhes sobre CI/CD com GitHub Actions, consulte [CI-CD-Github-Actions-Gitlab.md](CI-CD-Github-Actions-Gitlab.md).

---

**Parte 7 — Boas Práticas**

## 29. Boas Práticas e Checklist de Performance

### 29.1 Checklist de Código

- [ ] Consultas N+1 resolvidas com `JOIN FETCH`, `@EntityGraph` ou projeções
- [ ] Projeções (DTOs ou interfaces) usadas em vez de entidades completas para leituras
- [ ] `@BatchSize` ou `hibernate.default_batch_fetch_size` configurado (50-100)
- [ ] `spring.jpa.open-in-view=false` explícito em `application.yml`
- [ ] Queries de relatórios e dashboards usam `@QueryHints(readOnly = true)`
- [ ] `StringBuilder` ou `StringJoiner` em vez de concatenação em loops
- [ ] Streams paralelos usados apenas em coleções grandes (> 10.000 elementos) e operações CPU-bound
- [ ] Collections inicializadas com capacidade estimada (`new ArrayList<>(expectedSize)`)
- [ ] `Optional` não usado como parâmetro de método — apenas como retorno
- [ ] Pagination implementada no banco (não em memória) para listagens grandes
- [ ] Índices de banco criados para colunas usadas em `WHERE`, `JOIN` e `ORDER BY`
- [ ] Serialização Jackson configurada: `ObjectMapper` reutilizado como singleton, `@JsonView` para diferentes representações
- [ ] Regex compilado uma vez e reutilizado (`Pattern.compile()` em constante `static final`)
- [ ] Operações de bulk (INSERT/UPDATE/DELETE em lote) usando `saveAll()` com `batch_size`
- [ ] `@Transactional(readOnly = true)` em métodos que apenas leem dados

### 29.2 Checklist de Configuração

- [ ] HikariCP com `maximum-pool-size` calculado: `(core_count * 2) + disco_spindles`
- [ ] `hikari.minimum-idle` igual a `maximum-pool-size` (pool de tamanho fixo)
- [ ] `hikari.connection-timeout` configurado (padrão 30s, ajustar conforme SLA)
- [ ] `hikari.max-lifetime` configurado (< timeout do banco/proxy, padrão 30 min)
- [ ] `hikari.leak-detection-threshold` habilitado em staging (2000-5000ms)
- [ ] Compressão HTTP habilitada (`server.compression.enabled=true`)
- [ ] `server.compression.min-response-size` ajustado (padrão 2048 bytes)
- [ ] Logging assíncrono configurado (AsyncAppender no Logback ou async loggers no Log4j2)
- [ ] Nível de log em produção: `WARN` para frameworks, `INFO` para código da aplicação
- [ ] Cache de segundo nível do Hibernate habilitado se apropriado (com Caffeine ou Redis)
- [ ] `spring.jackson.default-property-inclusion=non_null` para reduzir payload JSON
- [ ] Actuator com endpoints filtrados em produção (expor apenas `/health`, `/metrics`, `/info`)
- [ ] Virtual Threads habilitadas no Spring Boot 3.2+ se aplicável (`spring.threads.virtual.enabled=true`)
- [ ] Tomcat `max-threads` ajustado conforme carga esperada (padrão 200)
- [ ] `server.tomcat.accept-count` ajustado (fila de conexões quando threads estão ocupadas)

### 29.3 Checklist de JVM

- [ ] GC escolhido intencionalmente: G1 (padrão), ZGC (baixa latência) ou Shenandoah
- [ ] `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/dumps/` configurado
- [ ] GC logging habilitado: `-Xlog:gc*:file=gc.log:time,level,tags`
- [ ] `-Xms` e `-Xmx` com o mesmo valor (evita resizing do heap)
- [ ] Heap dimensionado para deixar ~30% livre após Full GC
- [ ] `-XX:MaxMetaspaceSize` configurado para detectar class loader leaks
- [ ] `-XX:+UseStringDeduplication` considerado (efetivo com G1/ZGC quando há muitas Strings duplicadas)
- [ ] JVM flags de diagnóstico desabilitadas em produção (ex.: `-XX:+PrintCompilation`)
- [ ] JFR (Java Flight Recorder) habilitado em produção com overhead mínimo
- [ ] Versão da JVM atualizada (LTS: 21 ou 25, com patches de segurança)
- [ ] `-XX:+UseContainerSupport` (padrão desde Java 10) — JVM respeita limits do container

### 29.4 Checklist de Infraestrutura

- [ ] Container com `resources.limits` e `resources.requests` definidos (CPU e memória)
- [ ] `requests` de memória iguais a `limits` (evita OOM kill por overcommit)
- [ ] Liveness probe configurada (`/actuator/health/liveness`) com `initialDelaySeconds` adequado
- [ ] Readiness probe configurada (`/actuator/health/readiness`) para evitar tráfego antes do startup
- [ ] Startup probe configurada para aplicações com inicialização lenta
- [ ] Limite de file descriptors (FD) aumentado no container (`ulimit -n 65536`)
- [ ] TCP keepalive configurado para conexões de longa duração
- [ ] DNS caching ajustado (`networkaddress.cache.ttl=30` em `java.security` ou via flags JVM)
- [ ] Graceful shutdown habilitado (`server.shutdown=graceful`, `spring.lifecycle.timeout-per-shutdown-phase=30s`)
- [ ] Volume persistente para logs e heap dumps (não efêmero)
- [ ] Network policy restringindo tráfego entre pods (princípio do menor privilégio)
- [ ] Ingress com rate limiting e timeouts configurados
- [ ] HPA (Horizontal Pod Autoscaler) configurado com métrica de CPU e/ou custom metrics

### 29.5 Checklist de Observabilidade

- [ ] Métricas exportadas para Prometheus (ou equivalente) via Micrometer
- [ ] Dashboards criados: latência (p50, p95, p99), throughput, taxa de erro, saturação
- [ ] Alertas configurados: latência p99 > threshold, error rate > 1%, pod restart, OOM
- [ ] Tracing distribuído habilitado (OpenTelemetry com exportador para Jaeger/Tempo)
- [ ] `traceId` e `spanId` incluídos nos logs para correlação
- [ ] Health checks customizados para dependências críticas (banco, cache, filas)
- [ ] SLIs (Service Level Indicators) definidos e medidos automaticamente
- [ ] Logs estruturados em JSON com campos padronizados (timestamp, level, service, traceId)
- [ ] Log rotation configurado (por tamanho e por data)
- [ ] Métricas de negócio instrumentadas (pedidos/min, receita/hora, registros/dia)
- [ ] Dashboards de JVM: heap usage, GC pauses, thread count, class loading
- [ ] Alertas de saturação: connection pool usage > 80%, thread pool usage > 80%, disk > 85%

> Para detalhes sobre logs e observabilidade, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

---

## 30. Anti-Patterns de Performance

### 30.1 Premature Optimization

**Problema:** Otimizar código sem evidência de que ele é um gargalo. Substituir `ArrayList` por arrays primitivos, usar bitwise operations em vez de multiplicação, ou complicar arquitetura para "performance" sem medição. Isso aumenta complexidade, dificulta manutenção e frequentemente otimiza trechos que representam < 1% do tempo total.

**Solução:** Meça primeiro com profiling (async-profiler, JFR) e métricas de produção. Otimize apenas hot paths comprovados. Siga a máxima: *"Measure, don't guess."*

### 30.2 N+1 Queries Ignoradas

**Problema:** Acessar um relacionamento `@OneToMany` ou `@ManyToOne` com `FetchType.LAZY` dentro de um loop gera uma query adicional para cada entidade. Listar 100 pedidos com seus itens pode gerar 101 queries (1 para pedidos + 100 para itens de cada pedido).

**Solução:** Use `JOIN FETCH` em JPQL, `@EntityGraph` ou projeções DTO. Habilite `hibernate.generate_statistics=true` em desenvolvimento para detectar o problema. Configure `spring.jpa.properties.hibernate.session_metrics_log=true`.

> Para detalhes sobre N+1 e fetch strategies, consulte [Dicas-JPA-Hibernate-Queries.md](../Dicas-JPA-Hibernate-Queries.md).

### 30.3 Ausência de Connection Pool Tuning

**Problema:** Usar valores padrão do HikariCP sem considerar a carga real. O `maximum-pool-size` padrão é 10, que pode ser insuficiente para aplicações com alta concorrência, ou excessivo para bancos com limite baixo de conexões. Conexões ociosas desperdiçam recursos no banco; poucas conexões causam timeouts.

**Solução:** Calcule o pool size ideal: `conexões = (core_count * 2) + disco_spindles`. Monitore a métrica `hikaricp.connections.usage` no Micrometer. Configure `leak-detection-threshold` em staging para detectar conexões não devolvidas ao pool.

### 30.4 spring.jpa.open-in-view=true em Produção

**Problema:** O padrão `OSIV` (Open Session In View) mantém a sessão do Hibernate aberta durante toda a requisição HTTP, incluindo a renderização da view. Isso permite lazy loading na camada de apresentação, mas mantém uma conexão de banco ocupada durante todo o ciclo de vida da requisição — incluindo chamadas a serviços externos, serialização JSON e I/O de rede. Sob carga, o connection pool se esgota.

**Solução:** Configure `spring.jpa.open-in-view=false` e resolva todas as dependências de dados na camada de serviço com `JOIN FETCH`, `@EntityGraph` ou projeções. O Spring Boot 3.x já emite um warning quando OSIV está habilitado.

### 30.5 Thread Pool Excessivamente Grande

**Problema:** Configurar centenas ou milhares de threads no Tomcat (`server.tomcat.threads.max=1000`) na esperança de suportar mais requisições. Cada thread consome ~1 MB de stack, e com muitas threads o SO gasta mais tempo em context switching do que executando trabalho útil. O throughput cai em vez de subir.

**Solução:** Mantenha o thread pool proporcional ao número de cores. Para I/O-bound, `threads = cores * (1 + wait_time/service_time)`. Considere Virtual Threads (Java 21+) para workloads com muito I/O. Para CPU-bound, mais threads que cores é desperdício.

### 30.6 Cache sem Estratégia de Invalidação

**Problema:** Adicionar `@Cacheable` em métodos sem definir TTL, estratégia de evicção ou invalidação. O cache cresce indefinidamente (memory leak), serve dados obsoletos por tempo indeterminado e não há visibilidade sobre hit rate ou tamanho.

**Solução:** Sempre defina TTL (`expireAfterWrite`), tamanho máximo (`maximumSize`) e monitore hit rate via Micrometer (`cache.gets{result=hit}`). Use `@CacheEvict` nos métodos de escrita. Para caches distribuídos (Redis), planeje a estratégia de invalidação: write-through, write-behind ou invalidação explícita.

### 30.7 Ignorar GC Pauses

**Problema:** Não monitorar pausas do Garbage Collector em produção. Pausas de GC longas (> 200ms) causam picos de latência, timeouts em health checks e até remoção do pod pelo Kubernetes. A aplicação parece "travar" periodicamente sem causa aparente.

**Solução:** Habilite GC logging (`-Xlog:gc*`). Monitore a métrica `jvm.gc.pause` no Micrometer. Considere ZGC ou Shenandoah para aplicações sensíveis a latência. Dimensione o heap adequadamente — heap muito pequeno causa GC frequente, muito grande causa pausas longas no Full GC do G1.

### 30.8 Container sem Resource Limits

**Problema:** Executar containers sem `resources.limits` no Kubernetes. Sem limits, um container pode consumir toda a memória do nó, causando OOM kill em outros pods. Sem CPU limits, um container pode monopolizar o processador, degradando vizinhos.

**Solução:** Sempre defina `resources.requests` e `resources.limits`. Para memória, `requests == limits` (evita overcommit). Para CPU, `requests` pode ser menor que `limits` (permite burst). A JVM deve respeitar os limits: `-XX:MaxRAMPercentage=75.0` para deixar margem para o SO e processos auxiliares.

### 30.9 Logging Síncrono em Produção

**Problema:** Usar o appender síncrono padrão do Logback em produção. Cada chamada `log.info()` bloqueia a thread até a escrita no disco/rede ser concluída. Em cenários de alta volumetria, o logging se torna gargalo — a thread do Tomcat fica esperando I/O de disco em vez de processar requisições.

**Solução:** Configure `AsyncAppender` no Logback com `queue-size` adequado e `discardingThreshold` para descartar logs de nível DEBUG/TRACE sob pressão. Ou migre para Log4j2 com Async Loggers (baseados em LMAX Disruptor), que oferecem throughput ~10x superior ao Logback assíncrono.

> Para detalhes sobre logging assíncrono e configuração, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

---

## 31. Referências e Leitura Complementar

### Documentação Oficial

- [Spring Boot — Production-ready Features](https://docs.spring.io/spring-boot/reference/actuator/) — Actuator, métricas e health checks
- [Spring Boot — Performance](https://docs.spring.io/spring-boot/how-to/class-data-sharing.html) — CDS, AOT e otimizações de startup
- [HikariCP — GitHub](https://github.com/brettwooldridge/HikariCP) — Configuração, tuning e pool sizing
- [JMH — OpenJDK](https://github.com/openjdk/jmh) — Java Microbenchmark Harness
- [JMH Samples](https://github.com/openjdk/jmh/tree/master/jmh-samples/src/main/java/org/openjdk/jmh/samples) — Exemplos oficiais de benchmarks
- [ZGC — Oracle](https://docs.oracle.com/en/java/javase/21/gctuning/z-garbage-collector.html) — Tuning do Z Garbage Collector
- [G1 Garbage Collector — Oracle](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html) — Tuning do G1GC
- [Gatling — Documentação](https://docs.gatling.io/) — Testes de carga com Java/Scala DSL
- [k6 — Documentação](https://grafana.com/docs/k6/latest/) — Testes de carga com JavaScript
- [async-profiler — GitHub](https://github.com/async-profiler/async-profiler) — CPU e allocation profiler para JVM
- [Micrometer — Documentação](https://micrometer.io/docs) — Fachada de métricas para JVM
- [OpenTelemetry Java](https://opentelemetry.io/docs/languages/java/) — Traces, métricas e logs distribuídos
- [Virtual Threads — JEP 444](https://openjdk.org/jeps/444) — Threads virtuais (Java 21)

### Documentos Relacionados neste Repositório

> Para logs, métricas, traces e observabilidade, consulte [Dicas-Logs-Observabilidade.md](../Dicas-Logs-Observabilidade.md).

> Para recursos avançados do Spring Boot (Actuator, cache, batch, resiliência), consulte [Spring-Boot-Avancado.md](../Spring-Boot-Avancado.md).

> Para queries JPA, N+1, projeções e fetch strategies, consulte [Dicas-JPA-Hibernate-Queries.md](../Dicas-JPA-Hibernate-Queries.md).

> Para threads, concorrência e programação assíncrona em Java, consulte [Java-Avancado-Threads-Concorrencia.md](Java-Avancado-Threads-Concorrencia.md).

> Para compilação nativa e otimização de startup com GraalVM, consulte [GraalVM-Native-Image-Spring-Boot.md](GraalVM-Native-Image-Spring-Boot.md).

> Para containers, Docker Compose e Kubernetes, consulte [Docker-Compose-Kubernetes.md](Docker-Compose-Kubernetes.md).

> Para programação assíncrona com `@Async` e CompletableFuture, consulte [Spring-Boot-Async-CompletableFuture.md](Spring-Boot-Async-CompletableFuture.md).

> Para comparativo de desempenho entre Rust, Golang e Java, consulte [Rust-Golang-vs-Java-Desempenho.md](Rust-Golang-vs-Java-Desempenho.md).

> Para serialização JSON e configuração do Jackson, consulte [Dicas-Jackson.md](../Dicas-Jackson.md).

---

> **Documento produzido e revisado com apoio de agentes de IA. Última atualização: Julho/2026.**
