# Spring Data — JDBC, R2DBC, MongoDB, Redis e Elasticsearch

> **Baseline principal:** Spring Boot 3.5 · Java 21+  
> **Pré-requisito:** familiaridade com Spring IoC/DI e Spring Data JPA (ver `Dicas-JPA-Hibernate-Modelagem-Entities.md` e `Dicas-JPA-Hibernate-Queries.md`).

---

## Sumário

**Parte 1 — Spring Data JDBC**

1. [Spring Data JDBC — Visão Geral e Diferenças em Relação ao JPA](#1-spring-data-jdbc--visão-geral-e-diferenças-em-relação-ao-jpa)
2. [Dependência e Configuração](#2-dependência-e-configuração-jdbc)
3. [Entidades e Anotações](#3-entidades-e-anotações-jdbc)
4. [Repositórios](#4-repositórios-jdbc)
5. [Relacionamentos — Agregados e AggregateReference](#5-relacionamentos--agregados-e-aggregatereference)
6. [Consultas Customizadas](#6-consultas-customizadas-jdbc)
7. [Eventos de Ciclo de Vida](#7-eventos-de-ciclo-de-vida-jdbc)
8. [Testes com Spring Data JDBC](#8-testes-com-spring-data-jdbc)

**Parte 2 — Spring Data R2DBC**

9. [Spring Data R2DBC — Visão Geral e Programação Reativa](#9-spring-data-r2dbc--visão-geral-e-programação-reativa)
10. [Dependência e Configuração](#10-dependência-e-configuração-r2dbc)
11. [Entidades e Repositórios Reativos](#11-entidades-e-repositórios-reativos)
12. [Consultas Customizadas com R2DBC](#12-consultas-customizadas-com-r2dbc)
13. [Transações Reativas](#13-transações-reativas)
14. [Testes com Spring Data R2DBC](#14-testes-com-spring-data-r2dbc)

**Parte 3 — Spring Data MongoDB**

15. [Spring Data MongoDB — Visão Geral](#15-spring-data-mongodb--visão-geral)
16. [Dependência e Configuração](#16-dependência-e-configuração-mongodb)
17. [Mapeamento de Documentos](#17-mapeamento-de-documentos)
18. [Repositórios MongoDB](#18-repositórios-mongodb)
19. [MongoTemplate e Consultas Avançadas](#19-mongotemplate-e-consultas-avançadas)
20. [Aggregation Pipeline](#20-aggregation-pipeline)
21. [Índices e Texto Completo](#21-índices-e-texto-completo)
22. [GridFS — Armazenamento de Arquivos](#22-gridfs--armazenamento-de-arquivos)
23. [Change Streams](#23-change-streams)
24. [Testes com Spring Data MongoDB](#24-testes-com-spring-data-mongodb)

**Parte 4 — Spring Data Redis**

25. [Spring Data Redis — Visão Geral](#25-spring-data-redis--visão-geral)
26. [Dependência e Configuração](#26-dependência-e-configuração-redis)
27. [RedisTemplate — Operações Básicas](#27-redistemplate--operações-básicas)
28. [Spring Cache com Redis](#28-spring-cache-com-redis)
29. [Redis Repositories](#29-redis-repositories)
30. [Pub/Sub com Redis](#30-pubsub-com-redis)
31. [Lua Scripts, Pipeline e Transações](#31-lua-scripts-pipeline-e-transações)
32. [Testes com Spring Data Redis](#32-testes-com-spring-data-redis)

**Parte 5 — Spring Data Elasticsearch**

33. [Spring Data Elasticsearch — Visão Geral](#33-spring-data-elasticsearch--visão-geral)
34. [Dependência e Configuração](#34-dependência-e-configuração-elasticsearch)
35. [Mapeamento de Documentos Elasticsearch](#35-mapeamento-de-documentos-elasticsearch)
36. [Repositórios Elasticsearch](#36-repositórios-elasticsearch)
37. [ElasticsearchOperations e Consultas Avançadas](#37-elasticsearchoperations-e-consultas-avançadas)
38. [Testes com Spring Data Elasticsearch](#38-testes-com-spring-data-elasticsearch)

---

## Parte 1 — Spring Data JDBC

## 1. Spring Data JDBC — Visão Geral e Diferenças em Relação ao JPA

Spring Data JDBC é uma alternativa mais simples e explícita ao JPA/Hibernate. Não há lazy loading, sessão de persistência (contexto de persistência), cache de primeiro nível nem geração automática de DDL por padrão.

| Característica           | Spring Data JPA / Hibernate   | Spring Data JDBC             |
|--------------------------|-------------------------------|------------------------------|
| Lazy loading             | Sim (padrão em coleções)      | Não — sempre eager           |
| Contexto de persistência | Sim (EntityManager / Session) | Não — stateless              |
| DDL automático           | Sim (`ddl-auto`)              | Via scripts SQL ou Flyway/Liquibase |
| Suporte a herança        | Completo (`@Inheritance`)     | Limitado                     |
| Complexidade             | Alta                          | Baixa                        |
| Transparência SQL        | Baixa (Hibernate gera)        | Alta (queries explícitas)    |
| Domain-Driven Design     | Possível, mas trabalhoso      | Incentivado nativamente      |
| Eventos de ciclo de vida | `@PrePersist`, `@PostLoad`…   | `BeforeSave`, `AfterLoad`…   |

**Quando usar Spring Data JDBC:**
- Modelos de domínio com limites bem definidos (agregados DDD).
- Quando transparência total de SQL é prioritária.
- Microsserviços com esquemas simples.
- Quando lazy loading seria um problema (N+1).

---

## 2. Dependência e Configuração JDBC

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meudb
    username: postgres
    password: senha
  sql:
    init:
      mode: always          # executa schema.sql e data.sql na inicialização
      schema-locations: classpath:schema.sql
```

Diferente do JPA, a criação de tabelas não é automática — use `schema.sql` em `src/main/resources`:

```sql
-- schema.sql
CREATE TABLE IF NOT EXISTS pedido (
    id          BIGSERIAL PRIMARY KEY,
    cliente_id  BIGINT    NOT NULL,
    total       NUMERIC(10,2),
    criado_em   TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS item_pedido (
    id          BIGSERIAL PRIMARY KEY,
    pedido      BIGINT    NOT NULL REFERENCES pedido(id) ON DELETE CASCADE,
    produto_id  BIGINT    NOT NULL,
    quantidade  INT       NOT NULL,
    preco       NUMERIC(10,2) NOT NULL
);
```

---

## 3. Entidades e Anotações JDBC

```java
import org.springframework.data.annotation.*;
import org.springframework.data.relational.core.mapping.*;

@Table("pedido")
public class Pedido {

    @Id
    private Long id;

    private Long clienteId;

    private BigDecimal total;

    @CreatedDate
    private LocalDateTime criadoEm;

    @LastModifiedDate
    private LocalDateTime atualizadoEm;

    @Version
    private Long versao;

    // coleção embutida no agregado — Spring Data JDBC gerencia automaticamente
    @MappedCollection(idColumn = "pedido", keyColumn = "posicao")
    private List<ItemPedido> itens = new ArrayList<>();

    // construtor, getters, setters (ou record/Lombok)
}

@Table("item_pedido")
public class ItemPedido {

    @Id
    private Long id;

    private Long produtoId;
    private int quantidade;
    private BigDecimal preco;
}
```

**Anotações principais:**

| Anotação | Descrição |
|---|---|
| `@Table("nome")` | Nome da tabela (opcional se seguir convenção snake_case) |
| `@Id` | Chave primária |
| `@Column("nome")` | Mapeamento de coluna customizado |
| `@MappedCollection` | Coleção de filhos dentro do agregado; `idColumn` = FK na tabela filha |
| `@Embedded` | Objeto embutido mapeado em colunas da mesma tabela |
| `@CreatedDate` / `@LastModifiedDate` | Preenchidos automaticamente com `@EnableJdbcAuditing` |
| `@Version` | Controle otimista de concorrência |
| `@Transient` | Campo não persistido |

Para habilitar auditoria:

```java
@Configuration
@EnableJdbcAuditing
public class JdbcConfig {}
```

---

## 4. Repositórios JDBC

```java
public interface PedidoRepository extends CrudRepository<Pedido, Long> {

    // métodos derivados do nome funcionam igual ao JPA
    List<Pedido> findByClienteId(Long clienteId);

    Optional<Pedido> findByIdAndClienteId(Long id, Long clienteId);

    @Query("SELECT * FROM pedido WHERE total > :minimo ORDER BY criado_em DESC")
    List<Pedido> findComTotalAcimaDe(@Param("minimo") BigDecimal minimo);

    @Query("SELECT * FROM pedido WHERE criado_em >= :desde")
    Page<Pedido> findRecentes(@Param("desde") LocalDateTime desde, Pageable pageable);

    @Modifying
    @Query("UPDATE pedido SET total = :total WHERE id = :id")
    void atualizarTotal(@Param("id") Long id, @Param("total") BigDecimal total);

    long countByClienteId(Long clienteId);
}
```

`PagingAndSortingRepository` e `ListCrudRepository` também estão disponíveis:

```java
public interface PedidoRepository
        extends ListCrudRepository<Pedido, Long>,
                PagingAndSortingRepository<Pedido, Long> { ... }
```

**Importante:** `save()` no Spring Data JDBC faz `INSERT` quando `id == null` e `UPDATE` completo (todos os campos) quando `id != null`. Não existe merge parcial como no JPA — coleções filhas são sempre substituídas integralmente.

---

## 5. Relacionamentos — Agregados e AggregateReference

Spring Data JDBC segue o modelo de **agregados do DDD**. Uma entidade pertence a exatamente um agregado e é persistida junto com ele.

### Coleção de filhos (1:N dentro do agregado)

```java
@Table("pedido")
public class Pedido {
    @Id private Long id;

    @MappedCollection(idColumn = "pedido")
    private Set<ItemPedido> itens = new LinkedHashSet<>();

    public void adicionarItem(ItemPedido item) { itens.add(item); }
}
```

Ao salvar `pedido`, o Spring Data JDBC:
1. Faz `DELETE FROM item_pedido WHERE pedido = ?` (remove os antigos).
2. Faz `INSERT` de cada item da coleção atual.

### Referência a outro agregado (AggregateReference)

Para referenciar outra raiz de agregado sem carregar o objeto inteiro:

```java
import org.springframework.data.jdbc.core.mapping.AggregateReference;

@Table("pedido")
public class Pedido {
    @Id private Long id;

    // guarda apenas o ID de Cliente, sem eager/lazy loading
    private AggregateReference<Cliente, Long> cliente;
}

// uso
Pedido p = new Pedido();
p.setCliente(AggregateReference.to(42L));
pedidoRepository.save(p);

// leitura
Long clienteId = p.getCliente().getId();
```

### Objeto embutido (@Embedded)

```java
@Table("endereco_entrega")
public class EnderecoEntrega {
    @Id private Long id;

    @Embedded(onEmpty = Embedded.OnEmpty.USE_NULL)
    private Endereco endereco;
}

public record Endereco(String rua, String cidade, String cep) {}
```

As colunas `rua`, `cidade`, `cep` ficam diretamente na tabela `endereco_entrega`.

---

## 6. Consultas Customizadas JDBC

### NamedParameterJdbcTemplate

Para queries complexas não suportadas pelos repositórios:

```java
@Repository
@RequiredArgsConstructor
public class PedidoQueryRepository {

    private final NamedParameterJdbcTemplate jdbc;

    public List<RelatorioPedido> relatorio(Long clienteId, LocalDate inicio, LocalDate fim) {
        String sql = """
                SELECT p.id, p.total, c.nome AS cliente_nome,
                       COUNT(ip.id) AS qtd_itens
                  FROM pedido p
                  JOIN cliente c ON c.id = p.cliente_id
                  JOIN item_pedido ip ON ip.pedido = p.id
                 WHERE p.cliente_id = :clienteId
                   AND p.criado_em::date BETWEEN :inicio AND :fim
                 GROUP BY p.id, p.total, c.nome
                """;

        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("clienteId", clienteId)
                .addValue("inicio", inicio)
                .addValue("fim", fim);

        return jdbc.query(sql, params, (rs, rowNum) -> new RelatorioPedido(
                rs.getLong("id"),
                rs.getBigDecimal("total"),
                rs.getString("cliente_nome"),
                rs.getInt("qtd_itens")
        ));
    }
}
```

### RowMapper customizado

```java
@Component
public class PedidoRowMapper implements RowMapper<Pedido> {
    @Override
    public Pedido mapRow(ResultSet rs, int rowNum) throws SQLException {
        Pedido p = new Pedido();
        p.setId(rs.getLong("id"));
        p.setTotal(rs.getBigDecimal("total"));
        // ...
        return p;
    }
}
```

---

## 7. Eventos de Ciclo de Vida JDBC

Spring Data JDBC publica eventos no `ApplicationContext` em vez de usar anotações de callback na entidade:

```java
@Component
public class PedidoEventListener {

    @EventListener
    void antesDeGravar(BeforeSaveEvent<Pedido> event) {
        Pedido pedido = event.getEntity();
        pedido.recalcularTotal();
    }

    @EventListener
    void aposCarregar(AfterLoadEvent<Pedido> event) {
        // executado após SELECT
    }

    @EventListener
    void aposGravar(AfterSaveEvent<Pedido> event) {
        // auditoria, notificações, etc.
    }

    @EventListener
    void antesDeExcluir(BeforeDeleteEvent<Pedido> event) {
        // validações antes do DELETE
    }
}
```

Eventos disponíveis: `BeforeConvertEvent`, `BeforeSaveEvent`, `AfterSaveEvent`, `BeforeDeleteEvent`, `AfterDeleteEvent`, `AfterLoadEvent`.

---

## 8. Testes com Spring Data JDBC

```java
@DataJdbcTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class PedidoRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres =
            new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configureDataSource(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired PedidoRepository repository;

    @Test
    void deveSalvarERecuperarPedido() {
        Pedido pedido = new Pedido();
        pedido.setClienteId(1L);
        pedido.setTotal(new BigDecimal("150.00"));
        pedido.adicionarItem(new ItemPedido(10L, 2, new BigDecimal("75.00")));

        Pedido salvo = repository.save(pedido);

        assertThat(salvo.getId()).isNotNull();
        assertThat(salvo.getItens()).hasSize(1);

        Pedido recuperado = repository.findById(salvo.getId()).orElseThrow();
        assertThat(recuperado.getTotal()).isEqualByComparingTo("150.00");
    }
}
```

---

## Parte 2 — Spring Data R2DBC

## 9. Spring Data R2DBC — Visão Geral e Programação Reativa

Spring Data R2DBC oferece acesso reativo a bancos relacionais, usando Project Reactor (`Mono`/`Flux`). É a escolha correta quando o stack inteiro é reativo (Spring WebFlux).

| Aspecto | Spring Data JDBC | Spring Data R2DBC |
|---|---|---|
| I/O | Bloqueante | Não-bloqueante (reativo) |
| Stack alvo | Spring MVC | Spring WebFlux |
| Driver | JDBC | R2DBC (ex: `r2dbc-postgresql`) |
| Contexto | Thread-per-request | Event Loop |
| Relacionamentos | Parcialmente automáticos | Manuais (sem joins automáticos) |
| Transações | `@Transactional` | `@Transactional` (reativo) |

**Quando usar R2DBC:**
- APIs de alta concorrência com muitas conexões simultâneas.
- Stack totalmente reativo (WebFlux + R2DBC + Redis reativo).
- Streaming de resultados em tempo real.

---

## 10. Dependência e Configuração R2DBC

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/meudb
    username: postgres
    password: senha
    pool:
      initial-size: 5
      max-size: 20
  sql:
    init:
      mode: always
```

Configuração programática (opcional):

```java
@Configuration
@EnableR2dbcAuditing
public class R2dbcConfig extends AbstractR2dbcConfiguration {

    @Override
    public ConnectionFactory connectionFactory() {
        return ConnectionFactories.get(
            ConnectionFactoryOptions.builder()
                .option(DRIVER, "postgresql")
                .option(HOST, "localhost")
                .option(PORT, 5432)
                .option(DATABASE, "meudb")
                .option(USER, "postgres")
                .option(PASSWORD, "senha")
                .build()
        );
    }
}
```

---

## 11. Entidades e Repositórios Reativos

```java
@Table("produto")
public class Produto {

    @Id
    private Long id;

    private String nome;

    private BigDecimal preco;

    @Column("estoque_qtd")
    private int estoque;

    @CreatedDate
    private LocalDateTime criadoEm;

    @Version
    private Long versao;
}
```

```java
public interface ProdutoRepository extends ReactiveCrudRepository<Produto, Long> {

    Flux<Produto> findByPrecoLessThanEqual(BigDecimal preco);

    Mono<Produto> findByNomeIgnoreCase(String nome);

    Flux<Produto> findByEstoqueGreaterThan(int minimo);

    @Query("SELECT * FROM produto WHERE nome ILIKE '%' || :termo || '%' ORDER BY preco")
    Flux<Produto> buscarPorTermo(@Param("termo") String termo);

    @Query("SELECT COUNT(*) FROM produto WHERE estoque = 0")
    Mono<Long> contarSemEstoque();
}
```

Uso em um Service:

```java
@Service
@RequiredArgsConstructor
public class ProdutoService {

    private final ProdutoRepository repository;

    public Flux<Produto> listarDisponiveis(BigDecimal precoMax) {
        return repository.findByPrecoLessThanEqual(precoMax)
                .filter(p -> p.getEstoque() > 0);
    }

    public Mono<Produto> criarOuAtualizar(Produto produto) {
        return repository.save(produto);
    }

    public Mono<Void> excluir(Long id) {
        return repository.findById(id)
                .switchIfEmpty(Mono.error(new EntityNotFoundException("Produto não encontrado")))
                .flatMap(repository::delete);
    }
}
```

---

## 12. Consultas Customizadas com R2DBC

### DatabaseClient — API de baixo nível

```java
@Repository
@RequiredArgsConstructor
public class ProdutoQueryRepository {

    private final DatabaseClient db;

    public Flux<Map<String, Object>> relatorioEstoque() {
        return db.sql("""
                    SELECT categoria, SUM(estoque_qtd) AS total_estoque,
                           AVG(preco) AS preco_medio
                      FROM produto
                     GROUP BY categoria
                     ORDER BY total_estoque DESC
                    """)
                .fetch()
                .all();
    }

    public Mono<Produto> buscarComLock(Long id) {
        return db.sql("SELECT * FROM produto WHERE id = :id FOR UPDATE")
                .bind("id", id)
                .map((row, meta) -> {
                    Produto p = new Produto();
                    p.setId(row.get("id", Long.class));
                    p.setNome(row.get("nome", String.class));
                    p.setPreco(row.get("preco", BigDecimal.class));
                    return p;
                })
                .one();
    }
}
```

### R2dbcEntityTemplate — API de nível médio

```java
@Repository
@RequiredArgsConstructor
public class ProdutoTemplateRepository {

    private final R2dbcEntityTemplate template;

    public Flux<Produto> buscarBaratos(BigDecimal limite) {
        return template.select(Produto.class)
                .matching(query(where("preco").lessThanOrEquals(limite)
                        .and("estoque_qtd").greaterThan(0)))
                .all();
    }

    public Mono<Long> contarPorCategoria(String categoria) {
        return template.count(
                query(where("categoria").is(categoria)),
                Produto.class
        );
    }
}
```

---

## 13. Transações Reativas

```java
@Service
@RequiredArgsConstructor
public class TransferenciaProdutoService {

    private final ProdutoRepository produtoRepo;
    private final MovimentacaoRepository movimentacaoRepo;

    // @Transactional funciona com contexto de transação reativo
    @Transactional
    public Mono<Void> transferirEstoque(Long origemId, Long destinoId, int quantidade) {
        return produtoRepo.findById(origemId)
                .flatMap(origem -> {
                    if (origem.getEstoque() < quantidade) {
                        return Mono.error(new EstoqueInsuficienteException());
                    }
                    origem.setEstoque(origem.getEstoque() - quantidade);
                    return produtoRepo.save(origem);
                })
                .flatMap(origem -> produtoRepo.findById(destinoId))
                .flatMap(destino -> {
                    destino.setEstoque(destino.getEstoque() + quantidade);
                    return produtoRepo.save(destino);
                })
                .flatMap(destino -> movimentacaoRepo.save(
                        new Movimentacao(origemId, destinoId, quantidade)))
                .then();
    }
}
```

Controle manual com `TransactionalOperator`:

```java
@Service
@RequiredArgsConstructor
public class PagamentoService {

    private final TransactionalOperator txOperator;
    private final PedidoRepository pedidoRepo;

    public Mono<Pedido> processarPagamento(Long pedidoId) {
        return pedidoRepo.findById(pedidoId)
                .flatMap(this::cobrar)
                .flatMap(pedidoRepo::save)
                .as(txOperator::transactional);
    }

    private Mono<Pedido> cobrar(Pedido pedido) { /* ... */ return Mono.just(pedido); }
}
```

---

## 14. Testes com Spring Data R2DBC

```java
@DataR2dbcTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class ProdutoRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres =
            new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.r2dbc.url", () ->
                "r2dbc:postgresql://" + postgres.getHost() + ":" +
                postgres.getMappedPort(5432) + "/" + postgres.getDatabaseName());
        registry.add("spring.r2dbc.username", postgres::getUsername);
        registry.add("spring.r2dbc.password", postgres::getPassword);
    }

    @Autowired ProdutoRepository repository;

    @Test
    void deveSalvarProduto() {
        Produto p = new Produto();
        p.setNome("Notebook");
        p.setPreco(new BigDecimal("3500.00"));
        p.setEstoque(10);

        StepVerifier.create(repository.save(p).then(repository.count()))
                .expectNextMatches(count -> count > 0)
                .verifyComplete();
    }
}
```

---

## Parte 3 — Spring Data MongoDB

## 15. Spring Data MongoDB — Visão Geral

MongoDB é um banco de documentos orientado a JSON. Spring Data MongoDB oferece mapeamento de classes Java para documentos BSON, repositórios com derivação de métodos e suporte a aggregation pipeline.

**Quando usar MongoDB:**
- Estruturas de dados variáveis ou com esquema flexível.
- Documentos com profundidade moderada e dados naturalmente aninhados.
- Cargas com muita leitura e modelos desnormalizados.
- Prototipação rápida sem DDL.

---

## 16. Dependência e Configuração MongoDB

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/meudb
      auto-index-creation: true    # cria índices declarados com @Indexed
```

Configuração programática avançada:

```java
@Configuration
@EnableMongoAuditing
public class MongoConfig extends AbstractMongoClientConfiguration {

    @Override
    protected String getDatabaseName() { return "meudb"; }

    @Override
    public MongoClient mongoClient() {
        return MongoClients.create("mongodb://localhost:27017");
    }

    @Bean
    public MongoCustomConversions mongoCustomConversions() {
        return new MongoCustomConversions(List.of(
                new MoneyWriteConverter(),
                new MoneyReadConverter()
        ));
    }
}
```

---

## 17. Mapeamento de Documentos

```java
import org.springframework.data.mongodb.core.mapping.*;
import org.springframework.data.annotation.*;

@Document(collection = "produtos")
public class Produto {

    @Id
    private String id;          // String ou ObjectId — MongoDB gera automaticamente

    @Field("nome_produto")      // nome do campo no documento
    private String nome;

    private BigDecimal preco;

    @Indexed(unique = true)
    private String sku;

    @Indexed
    private String categoria;

    @DBRef                      // referência a outro documento (lazy ou eager)
    private Fornecedor fornecedor;

    private List<String> tags = new ArrayList<>();

    private Especificacoes especificacoes;   // documento aninhado

    @CreatedDate
    private Instant criadoEm;

    @LastModifiedDate
    private Instant atualizadoEm;

    @Version
    private Long versao;
}

// Objeto aninhado — não precisa de @Document
public class Especificacoes {
    private String peso;
    private String dimensoes;
    private Map<String, String> atributos = new HashMap<>();
}
```

### Índices compostos e TTL

```java
@Document(collection = "sessoes")
@CompoundIndex(name = "usuario_expiracao", def = "{'usuarioId': 1, 'expiracao': 1}")
public class Sessao {

    @Id
    private String id;

    private String usuarioId;

    @Indexed(expireAfterSeconds = 3600)  // TTL — MongoDB remove automaticamente
    private Date expiracao;

    private Map<String, Object> dados = new HashMap<>();
}
```

---

## 18. Repositórios MongoDB

```java
public interface ProdutoRepository extends MongoRepository<Produto, String> {

    // derivação de método
    List<Produto> findByCategoria(String categoria);

    Optional<Produto> findBySku(String sku);

    List<Produto> findByPrecoLessThanEqual(BigDecimal preco);

    List<Produto> findByTagsContaining(String tag);

    // consulta por campo aninhado
    List<Produto> findByEspecificacoesPeso(String peso);

    // @Query com expressão MongoDB
    @Query("{ 'preco': { $gte: ?0, $lte: ?1 }, 'categoria': ?2 }")
    List<Produto> findByPrecoBetweenECategoria(
            BigDecimal min, BigDecimal max, String categoria);

    // projeção — retorna apenas campos selecionados
    @Query(value = "{ 'categoria': ?0 }", fields = "{ 'nome': 1, 'preco': 1, 'sku': 1 }")
    List<ProdutoResumo> findResumoByCategoria(String categoria);

    // update
    @Update("{ '$inc': { 'estoque': ?1 } }")
    @Query("{ '_id': ?0 }")
    void incrementarEstoque(String id, int quantidade);

    Page<Produto> findByCategoriaOrderByPrecoAsc(String categoria, Pageable pageable);

    long countByCategoria(String categoria);
}

public interface ProdutoResumo {
    String getNome();
    BigDecimal getPreco();
    String getSku();
}
```

---

## 19. MongoTemplate e Consultas Avançadas

```java
@Repository
@RequiredArgsConstructor
public class ProdutoQueryRepository {

    private final MongoTemplate mongo;

    public List<Produto> buscarComFiltros(String categoria, BigDecimal precoMax, List<String> tags) {
        Criteria criteria = new Criteria();

        if (categoria != null) criteria.and("categoria").is(categoria);
        if (precoMax != null) criteria.and("preco").lte(precoMax);
        if (!tags.isEmpty()) criteria.and("tags").in(tags);

        Query query = new Query(criteria)
                .with(Sort.by("preco").ascending())
                .limit(50);

        return mongo.find(query, Produto.class);
    }

    public UpdateResult aumentarPrecoDaCategoria(String categoria, double percentual) {
        Query query = new Query(Criteria.where("categoria").is(categoria));
        Update update = new Update().multiply("preco", 1 + percentual / 100);
        return mongo.updateMulti(query, update, Produto.class);
    }

    public Produto buscarEAtualizarEstoque(String sku, int delta) {
        Query query = new Query(Criteria.where("sku").is(sku));
        Update update = new Update().inc("estoque", delta);
        // retorna o documento APÓS a modificação
        return mongo.findAndModify(
                query, update,
                FindAndModifyOptions.options().returnNew(true),
                Produto.class
        );
    }
}
```

---

## 20. Aggregation Pipeline

```java
@Repository
@RequiredArgsConstructor
public class RelatorioRepository {

    private final MongoTemplate mongo;

    public List<RelatorioCategorias> totalPorCategoria() {
        GroupOperation grupo = group("categoria")
                .count().as("quantidadeProdutos")
                .avg("preco").as("precoMedio")
                .sum("estoque").as("totalEstoque");

        SortOperation ordenar = sort(Sort.by(Sort.Direction.DESC, "quantidadeProdutos"));

        ProjectionOperation projetar = project()
                .andExpression("_id").as("categoria")
                .andExpression("quantidadeProdutos").as("quantidadeProdutos")
                .andExpression("precoMedio").as("precoMedio")
                .andExpression("totalEstoque").as("totalEstoque")
                .andExclude("_id");

        Aggregation agg = newAggregation(grupo, ordenar, projetar);

        return mongo.aggregate(agg, "produtos", RelatorioCategorias.class).getMappedResults();
    }

    // Exemplo com $lookup (join)
    public List<ProdutoComFornecedor> produtosComFornecedor(String categoria) {
        MatchOperation filtro = match(Criteria.where("categoria").is(categoria));

        LookupOperation lookup = LookupOperation.newLookup()
                .from("fornecedores")
                .localField("fornecedorId")
                .foreignField("_id")
                .as("fornecedor");

        UnwindOperation unwind = unwind("fornecedor", true); // true = preserveNullAndEmpty

        Aggregation agg = newAggregation(filtro, lookup, unwind);

        return mongo.aggregate(agg, "produtos", ProdutoComFornecedor.class).getMappedResults();
    }
}
```

---

## 21. Índices e Texto Completo

```java
@Document(collection = "artigos")
@TextIndexed(weights = { @TextIndexed.Weight(key = "titulo", weight = 3) })
public class Artigo {

    @Id private String id;

    @TextIndexed(weight = 3)
    private String titulo;

    @TextIndexed
    private String conteudo;

    @Indexed
    private String autorId;

    @Indexed
    private List<String> categorias;
}
```

Busca por texto completo:

```java
public interface ArtigoRepository extends MongoRepository<Artigo, String> {
    // busca por texto completo (usa índice @TextIndexed)
    List<Artigo> findByTituloContainingOrConteudoContaining(String t1, String t2);
}

// Alternativa com TextCriteria via MongoTemplate
@RequiredArgsConstructor
public class ArtigoSearchService {

    private final MongoTemplate mongo;

    public List<Artigo> buscarPorTexto(String termo) {
        TextCriteria textCriteria = TextCriteria.forDefaultLanguage().matchingPhrase(termo);
        Query query = TextQuery.queryText(textCriteria).sortByScore();
        return mongo.find(query, Artigo.class);
    }
}
```

---

## 22. GridFS — Armazenamento de Arquivos

GridFS divide arquivos grandes em chunks e armazena metadados.

```java
@Service
@RequiredArgsConstructor
public class ArquivoService {

    private final GridFsTemplate gridFsTemplate;
    private final GridFsOperations gridFsOperations;

    public String salvarArquivo(MultipartFile file) throws IOException {
        DBObject metadata = new BasicDBObject();
        metadata.put("tipo", file.getContentType());
        metadata.put("tamanho", file.getSize());

        ObjectId id = gridFsTemplate.store(
                file.getInputStream(),
                file.getOriginalFilename(),
                file.getContentType(),
                metadata
        );
        return id.toString();
    }

    public GridFsResource recuperarArquivo(String id) {
        GridFSFile arquivo = gridFsTemplate.findOne(
                new Query(Criteria.where("_id").is(new ObjectId(id)))
        );
        if (arquivo == null) throw new ArquivoNaoEncontradoException(id);
        return gridFsOperations.getResource(arquivo);
    }

    public void excluirArquivo(String id) {
        gridFsTemplate.delete(new Query(Criteria.where("_id").is(new ObjectId(id))));
    }
}
```

---

## 23. Change Streams

Change Streams permitem reação a mudanças no banco em tempo real:

```java
@Component
@RequiredArgsConstructor
public class ProdutoChangeStreamListener {

    private final MongoTemplate mongo;

    @EventListener(ApplicationReadyEvent.class)
    public void observarMudancas() {
        MessageListenerContainer container = new DefaultMessageListenerContainer(mongo);
        container.start();

        ChangeStreamRequest<Produto> request = ChangeStreamRequest.builder(
                (ChangeStreamEvent<Produto> event) -> {
                    Produto produto = event.getBody();
                    OperationType tipo = event.getOperationType();
                    System.out.printf("Mudança [%s]: %s%n", tipo, produto);
                })
                .collection("produtos")
                .filter(newAggregation(match(where("operationType").is("update"))))
                .fullDocumentLookup(FullDocument.UPDATE_LOOKUP)
                .build();

        container.register(request, Produto.class);
    }
}
```

---

## 24. Testes com Spring Data MongoDB

```java
@DataMongoTest
@Testcontainers
class ProdutoRepositoryTest {

    @Container
    static MongoDBContainer mongo = new MongoDBContainer("mongo:7.0");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongo::getReplicaSetUrl);
    }

    @Autowired ProdutoRepository repository;

    @BeforeEach
    void limpar() { repository.deleteAll(); }

    @Test
    void deveSalvarEBuscarPorCategoria() {
        Produto p = new Produto();
        p.setNome("Mouse");
        p.setSku("MSE-001");
        p.setCategoria("Periféricos");
        p.setPreco(new BigDecimal("89.90"));

        repository.save(p);

        List<Produto> resultado = repository.findByCategoria("Periféricos");
        assertThat(resultado).hasSize(1)
                .first().extracting(Produto::getNome).isEqualTo("Mouse");
    }
}
```

---

## Parte 4 — Spring Data Redis

## 25. Spring Data Redis — Visão Geral

Redis é um store de dados em memória, multi-estrutura (strings, hashes, lists, sets, sorted sets, streams). Spring Data Redis oferece:

- `RedisTemplate` e `StringRedisTemplate` para operações de baixo nível.
- Spring Cache (`@Cacheable`, `@CachePut`, `@CacheEvict`) com Redis como backend.
- Redis Repositories para persistência orientada a objetos.
- Pub/Sub para mensageria leve.

**Quando usar Redis:**
- Cache de resultados de queries custosas.
- Sessões de usuário.
- Rate limiting e contadores.
- Filas/leaderboards com sorted sets.
- Pub/Sub para notificações em tempo real.

---

## 26. Dependência e Configuração Redis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: ""
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 10
          max-idle: 5
          min-idle: 2
```

Configuração do `RedisTemplate` com serialização JSON:

```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        StringRedisSerializer strSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer jsonSerializer =
                new GenericJackson2JsonRedisSerializer();

        template.setKeySerializer(strSerializer);
        template.setHashKeySerializer(strSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    public RedisCacheConfiguration cacheConfig(ObjectMapper objectMapper) {
        Jackson2JsonRedisSerializer<Object> serializer =
                new Jackson2JsonRedisSerializer<>(objectMapper, Object.class);

        return RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(
                                new StringRedisSerializer()))
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(serializer))
                .disableCachingNullValues();
    }

    @Bean
    public RedisCacheManager cacheManager(
            RedisConnectionFactory factory,
            RedisCacheConfiguration cacheConfig) {

        Map<String, RedisCacheConfiguration> configs = Map.of(
                "produtos", cacheConfig.entryTtl(Duration.ofHours(1)),
                "sessoes",  cacheConfig.entryTtl(Duration.ofMinutes(30))
        );

        return RedisCacheManager.builder(factory)
                .cacheDefaults(cacheConfig)
                .withInitialCacheConfigurations(configs)
                .build();
    }
}
```

---

## 27. RedisTemplate — Operações Básicas

```java
@Service
@RequiredArgsConstructor
public class RedisOperacoesService {

    private final RedisTemplate<String, Object> redis;
    private final StringRedisTemplate stringRedis;

    // -------- String / Value --------

    public void salvarComExpiracao(String chave, Object valor, Duration ttl) {
        redis.opsForValue().set(chave, valor, ttl);
    }

    public Object buscar(String chave) {
        return redis.opsForValue().get(chave);
    }

    public Long incrementarContador(String chave) {
        return redis.opsForValue().increment(chave);
    }

    // -------- Hash --------

    public void salvarHash(String chave, String campo, Object valor) {
        redis.opsForHash().put(chave, campo, valor);
    }

    public Map<Object, Object> buscarHash(String chave) {
        return redis.opsForHash().entries(chave);
    }

    // -------- List --------

    public void empurrarNaFila(String chave, Object valor) {
        redis.opsForList().rightPush(chave, valor);
    }

    public Object retirarDaFila(String chave) {
        return redis.opsForList().leftPop(chave);
    }

    // -------- Set --------

    public void adicionarAoSet(String chave, Object... valores) {
        redis.opsForSet().add(chave, valores);
    }

    public boolean pertenceAoSet(String chave, Object valor) {
        return Boolean.TRUE.equals(redis.opsForSet().isMember(chave, valor));
    }

    // -------- Sorted Set (ZSet) — ex: leaderboard --------

    public void atualizarPontuacao(String ranking, String membro, double pontos) {
        redis.opsForZSet().add(ranking, membro, pontos);
    }

    public Set<Object> topN(String ranking, int n) {
        return redis.opsForZSet().reverseRange(ranking, 0, n - 1);
    }

    // -------- Chaves --------

    public boolean existeChave(String chave) {
        return Boolean.TRUE.equals(redis.hasKey(chave));
    }

    public void excluirChave(String chave) {
        redis.delete(chave);
    }

    public void definirExpiracao(String chave, Duration ttl) {
        redis.expire(chave, ttl);
    }
}
```

---

## 28. Spring Cache com Redis

```java
@Service
@CacheConfig(cacheNames = "produtos")
@RequiredArgsConstructor
public class ProdutoCacheService {

    private final ProdutoRepository repository;

    @Cacheable(key = "#id")
    public Produto buscarPorId(Long id) {
        return repository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("Produto " + id));
    }

    @CachePut(key = "#produto.id")
    public Produto salvar(Produto produto) {
        return repository.save(produto);
    }

    @CacheEvict(key = "#id")
    public void excluir(Long id) {
        repository.deleteById(id);
    }

    @CacheEvict(allEntries = true)
    public void limparCache() { /* invalida todo o cache "produtos" */ }

    // cache com condição — só cacheia se preço > 100
    @Cacheable(key = "#categoria", condition = "#result.size() > 0",
               unless = "#result.isEmpty()")
    public List<Produto> buscarPorCategoria(String categoria) {
        return repository.findByCategoria(categoria);
    }
}
```

---

## 29. Redis Repositories

Mapeiam objetos Java como hashes no Redis, com suporte a TTL e índices secundários:

```java
@RedisHash(value = "sessao", timeToLive = 1800)  // TTL em segundos
public class Sessao {

    @Id
    private String id;

    @Indexed          // cria índice secundário para busca por campo
    private String usuarioId;

    private Map<String, Object> atributos = new HashMap<>();

    private Instant criadaEm;
}

public interface SessaoRepository extends CrudRepository<Sessao, String> {
    List<Sessao> findByUsuarioId(String usuarioId);
}
```

```java
@Service
@RequiredArgsConstructor
public class SessaoService {

    private final SessaoRepository repository;

    public Sessao criarSessao(String usuarioId) {
        Sessao sessao = new Sessao();
        sessao.setId(UUID.randomUUID().toString());
        sessao.setUsuarioId(usuarioId);
        sessao.setCriadaEm(Instant.now());
        return repository.save(sessao);
    }

    public List<Sessao> sessoesDoUsuario(String usuarioId) {
        return repository.findByUsuarioId(usuarioId);
    }
}
```

Habilitar suporte a repositórios Redis:

```java
@Configuration
@EnableRedisRepositories
public class RedisRepositoryConfig {}
```

---

## 30. Pub/Sub com Redis

```java
// Publisher
@Service
@RequiredArgsConstructor
public class EventoPublisher {

    private final RedisTemplate<String, Object> redis;

    public void publicar(String canal, Object mensagem) {
        redis.convertAndSend(canal, mensagem);
    }
}

// Subscriber
@Component
public class EventoSubscriber implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String canal = new String(message.getChannel());
        String corpo = new String(message.getBody());
        System.out.printf("Mensagem no canal [%s]: %s%n", canal, corpo);
    }
}

// Configuração do listener container
@Configuration
@RequiredArgsConstructor
public class PubSubConfig {

    private final RedisConnectionFactory factory;

    @Bean
    public RedisMessageListenerContainer listenerContainer(EventoSubscriber subscriber) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(factory);
        container.addMessageListener(subscriber, new PatternTopic("eventos.*"));
        return container;
    }
}
```

---

## 31. Lua Scripts, Pipeline e Transações

### Lua Scripts — operações atômicas

```java
@Component
public class RateLimiter {

    private final RedisTemplate<String, String> redis;

    private final RedisScript<Long> script = RedisScript.of("""
            local current = redis.call('INCR', KEYS[1])
            if current == 1 then
                redis.call('EXPIRE', KEYS[1], ARGV[1])
            end
            return current
            """, Long.class);

    public RateLimiter(StringRedisTemplate redis) {
        this.redis = redis;
    }

    public boolean isPermitido(String chave, int limite, int janelasegundos) {
        Long contagem = redis.execute(
                script,
                List.of("rate:" + chave),
                String.valueOf(janelasegundos)
        );
        return contagem != null && contagem <= limite;
    }
}
```

### Pipeline — envio em lote

```java
@Service
@RequiredArgsConstructor
public class BulkCacheService {

    private final RedisTemplate<String, Object> redis;

    public void salvarEmLote(Map<String, Object> dados) {
        redis.executePipelined((RedisCallback<Object>) connection -> {
            dados.forEach((k, v) -> {
                byte[] key = redis.getKeySerializer().serialize(k);
                byte[] val = redis.getValueSerializer().serialize(v);
                connection.stringCommands().setEx(key, 3600, val);
            });
            return null;
        });
    }
}
```

### Transações Redis (MULTI/EXEC)

```java
@Service
@RequiredArgsConstructor
public class TransacaoRedisService {

    private final RedisTemplate<String, Object> redis;

    public void transferirSaldo(String origem, String destino, double valor) {
        redis.setEnableTransactionSupport(true);
        try {
            redis.multi();
            redis.opsForValue().decrement(origem, (long) valor);
            redis.opsForValue().increment(destino, (long) valor);
            redis.exec();
        } catch (Exception e) {
            redis.discard();
            throw e;
        }
    }
}
```

---

## 32. Testes com Spring Data Redis

```java
@SpringBootTest
@Testcontainers
class RedisOperacoesTest {

    @Container
    static GenericContainer<?> redisContainer =
            new GenericContainer<>("redis:7.2-alpine").withExposedPorts(6379);

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redisContainer::getHost);
        registry.add("spring.data.redis.port",
                () -> redisContainer.getMappedPort(6379).toString());
    }

    @Autowired RedisTemplate<String, Object> redis;
    @Autowired SessaoRepository sessaoRepository;

    @Test
    void deveSalvarERecuperarHash() {
        redis.opsForValue().set("chave:1", "valor", Duration.ofMinutes(1));
        assertThat(redis.opsForValue().get("chave:1")).isEqualTo("valor");
    }

    @Test
    void deveSalvarSessaoComTTL() {
        Sessao sessao = new Sessao();
        sessao.setId("sess-001");
        sessao.setUsuarioId("user-42");

        sessaoRepository.save(sessao);

        assertThat(sessaoRepository.findById("sess-001")).isPresent();
        assertThat(sessaoRepository.findByUsuarioId("user-42")).hasSize(1);
    }
}
```

---

## Parte 5 — Spring Data Elasticsearch

## 33. Spring Data Elasticsearch — Visão Geral

Elasticsearch é um motor de busca e análise distribuído, construído sobre Apache Lucene. Spring Data Elasticsearch oferece mapeamento de documentos, repositórios com suporte a queries DSL e integração com `ElasticsearchOperations`.

**Quando usar Elasticsearch:**
- Busca full-text com relevância e scoring.
- Faceted search (filtros dinâmicos por categorias).
- Análise de logs (ELK Stack).
- Buscas geoespaciais.
- Auto-complete e sugestões de pesquisa.

---

## 34. Dependência e Configuração Elasticsearch

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic
    password: senha
    connection-timeout: 5s
    socket-timeout: 30s
```

Configuração programática avançada:

```java
@Configuration
public class ElasticsearchConfig extends ElasticsearchConfiguration {

    @Override
    public ClientConfiguration clientConfiguration() {
        return ClientConfiguration.builder()
                .connectedTo("localhost:9200")
                .withBasicAuth("elastic", "senha")
                .withConnectTimeout(Duration.ofSeconds(5))
                .withSocketTimeout(Duration.ofSeconds(30))
                .build();
    }
}
```

---

## 35. Mapeamento de Documentos Elasticsearch

```java
import org.springframework.data.elasticsearch.annotations.*;

@Document(indexName = "produtos", createIndex = true)
@Setting(settingPath = "elasticsearch/produto-settings.json")  // analyzer customizado
public class ProdutoEs {

    @Id
    private String id;

    @MultiField(
        mainField = @Field(type = FieldType.Text, analyzer = "portuguese"),
        otherFields = {
            @InnerField(suffix = "keyword", type = FieldType.Keyword)
        }
    )
    private String nome;

    @Field(type = FieldType.Text, analyzer = "portuguese")
    private String descricao;

    @Field(type = FieldType.Keyword)
    private String categoria;

    @Field(type = FieldType.Double)
    private BigDecimal preco;

    @Field(type = FieldType.Integer)
    private int estoque;

    @Field(type = FieldType.Keyword)
    private List<String> tags = new ArrayList<>();

    @GeoPointField
    private GeoPoint localizacao;

    @Field(type = FieldType.Date, format = DateFormat.date_time)
    private Instant criadoEm;
}
```

Arquivo `src/main/resources/elasticsearch/produto-settings.json`:

```json
{
  "analysis": {
    "analyzer": {
      "portuguese": {
        "tokenizer": "standard",
        "filter": ["lowercase", "portuguese_stemmer", "asciifolding"]
      }
    },
    "filter": {
      "portuguese_stemmer": {
        "type": "stemmer",
        "language": "portuguese"
      }
    }
  }
}
```

---

## 36. Repositórios Elasticsearch

```java
public interface ProdutoEsRepository
        extends ElasticsearchRepository<ProdutoEs, String> {

    // busca full-text derivada de método
    List<ProdutoEs> findByNome(String nome);

    // busca por keyword exato
    List<ProdutoEs> findByCategoria(String categoria);

    // range de preço com paginação
    Page<ProdutoEs> findByPrecoBetween(BigDecimal min, BigDecimal max, Pageable pageable);

    // busca em múltiplos campos com @Query (formato Elasticsearch Query DSL)
    @Query("""
            {
              "multi_match": {
                "query": "?0",
                "fields": ["nome^3", "descricao", "tags"],
                "type": "best_fields",
                "fuzziness": "AUTO"
              }
            }
            """)
    SearchHits<ProdutoEs> buscarFullText(String termo, Pageable pageable);

    // contagem
    long countByCategoria(String categoria);

    // existência
    boolean existsByNomeAndCategoria(String nome, String categoria);
}
```

Uso em Service:

```java
@Service
@RequiredArgsConstructor
public class BuscaProdutoService {

    private final ProdutoEsRepository repository;

    public List<ProdutoEs> buscar(String termo, String categoria, Pageable pageable) {
        if (categoria != null && !categoria.isBlank()) {
            return repository.findByCategoria(categoria);
        }
        SearchHits<ProdutoEs> hits = repository.buscarFullText(termo, pageable);
        return hits.stream()
                .map(SearchHit::getContent)
                .toList();
    }
}
```

---

## 37. ElasticsearchOperations e Consultas Avançadas

```java
@Repository
@RequiredArgsConstructor
public class ProdutoEsQueryRepository {

    private final ElasticsearchOperations esOps;

    // -------- Query DSL --------

    public SearchHits<ProdutoEs> buscaCompleta(
            String termo, String categoria,
            BigDecimal precoMin, BigDecimal precoMax,
            Pageable pageable) {

        BoolQuery.Builder bool = new BoolQuery.Builder();

        if (termo != null && !termo.isBlank()) {
            bool.must(MultiMatchQuery.of(mm -> mm
                    .query(termo)
                    .fields("nome^3", "descricao", "tags")
                    .fuzziness("AUTO"))._toQuery());
        }

        if (categoria != null) {
            bool.filter(TermQuery.of(t -> t.field("categoria").value(categoria))._toQuery());
        }

        if (precoMin != null || precoMax != null) {
            RangeQuery.Builder range = new RangeQuery.Builder().field("preco");
            if (precoMin != null) range.gte(JsonData.of(precoMin));
            if (precoMax != null) range.lte(JsonData.of(precoMax));
            bool.filter(range.build()._toQuery());
        }

        NativeQuery query = NativeQuery.builder()
                .withQuery(bool.build()._toQuery())
                .withPageable(pageable)
                .withSort(Sort.by("_score").descending())
                .build();

        return esOps.search(query, ProdutoEs.class);
    }

    // -------- Aggregations --------

    public Map<String, Long> contarPorCategoria() {
        NativeQuery query = NativeQuery.builder()
                .withAggregation("por_categoria",
                        Aggregation.of(a -> a.terms(t -> t.field("categoria").size(50))))
                .withMaxResults(0)    // só nos interessa a agregação, não os hits
                .build();

        SearchHits<ProdutoEs> resultado = esOps.search(query, ProdutoEs.class);

        ElasticsearchAggregations agregacoes =
                (ElasticsearchAggregations) resultado.getAggregations();

        return agregacoes.get("por_categoria")
                .aggregation()
                .getAggregate()
                .sterms()
                .buckets().array().stream()
                .collect(Collectors.toMap(
                        bucket -> bucket.key().stringValue(),
                        StringTermsBucket::docCount
                ));
    }

    // -------- Highlight --------

    public SearchHits<ProdutoEs> buscarComDestaque(String termo) {
        NativeQuery query = NativeQuery.builder()
                .withQuery(MultiMatchQuery.of(mm -> mm
                        .query(termo).fields("nome", "descricao"))._toQuery())
                .withHighlightQuery(new HighlightQuery(
                        new Highlight(List.of(
                                new HighlightField("nome"),
                                new HighlightField("descricao")
                        )), ProdutoEs.class))
                .build();

        return esOps.search(query, ProdutoEs.class);
    }

    // -------- Index Management --------

    public void reindexarProdutos(List<ProdutoEs> produtos) {
        IndexOperations indexOps = esOps.indexOps(ProdutoEs.class);
        indexOps.refresh();

        List<IndexQuery> queries = produtos.stream()
                .map(p -> new IndexQueryBuilder()
                        .withId(p.getId())
                        .withObject(p)
                        .build())
                .toList();

        esOps.bulkIndex(queries, ProdutoEs.class);
        indexOps.refresh();
    }

    // -------- Suggest / Auto-complete --------

    public List<String> sugestoes(String prefixo) {
        NativeQuery query = NativeQuery.builder()
                .withQuery(MatchPhrasePrefixQuery.of(mp -> mp
                        .field("nome")
                        .query(prefixo))._toQuery())
                .withMaxResults(10)
                .build();

        return esOps.search(query, ProdutoEs.class)
                .stream()
                .map(hit -> hit.getContent().getNome())
                .distinct()
                .toList();
    }
}
```

---

## 38. Testes com Spring Data Elasticsearch

```java
@SpringBootTest
@Testcontainers
class ProdutoEsRepositoryTest {

    @Container
    static ElasticsearchContainer elastic =
            new ElasticsearchContainer(
                    DockerImageName.parse("docker.elastic.co/elasticsearch/elasticsearch:8.13.0"))
                    .withEnv("xpack.security.enabled", "false");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.elasticsearch.uris", elastic::getHttpHostAddress);
    }

    @Autowired ProdutoEsRepository repository;
    @Autowired ElasticsearchOperations esOps;

    @BeforeEach
    void setUp() {
        IndexOperations ops = esOps.indexOps(ProdutoEs.class);
        if (ops.exists()) ops.delete();
        ops.createWithMapping();
    }

    @Test
    void deveSalvarEBuscarPorCategoria() {
        ProdutoEs produto = new ProdutoEs();
        produto.setNome("Teclado Mecânico");
        produto.setCategoria("Periféricos");
        produto.setPreco(new BigDecimal("350.00"));
        repository.save(produto);

        // aguarda indexação
        esOps.indexOps(ProdutoEs.class).refresh();

        List<ProdutoEs> resultado = repository.findByCategoria("Periféricos");
        assertThat(resultado).hasSize(1)
                .first().extracting(ProdutoEs::getNome).isEqualTo("Teclado Mecânico");
    }

    @Test
    void deveBuscarFullText() {
        ProdutoEs p1 = criarProduto("Notebook Gamer", "Notebooks", "3500.00");
        ProdutoEs p2 = criarProduto("Notebook Corporativo", "Notebooks", "4200.00");
        repository.saveAll(List.of(p1, p2));
        esOps.indexOps(ProdutoEs.class).refresh();

        SearchHits<ProdutoEs> hits = repository.buscarFullText(
                "notebook gamer", PageRequest.of(0, 10));

        assertThat(hits.getTotalHits()).isEqualTo(2);
        // o gamer deve aparecer primeiro (maior score)
        assertThat(hits.getSearchHit(0).getContent().getNome()).contains("Gamer");
    }

    private ProdutoEs criarProduto(String nome, String categoria, String preco) {
        ProdutoEs p = new ProdutoEs();
        p.setNome(nome);
        p.setCategoria(categoria);
        p.setPreco(new BigDecimal(preco));
        return p;
    }
}
```

---

## Visão Comparativa — Quando Usar Cada Tecnologia

| Critério | Spring Data JDBC | Spring Data R2DBC | Spring Data MongoDB | Spring Data Redis | Spring Data Elasticsearch |
|---|---|---|---|---|---|
| Tipo de dados | Relacional | Relacional | Documentos JSON | Chave-valor/estruturas | Documentos (busca) |
| Modelo de acesso | Síncrono | Reativo | Síncrono ou reativo | Síncrono ou reativo | Síncrono |
| Stack alvo | Spring MVC | Spring WebFlux | Ambos | Ambos | Spring MVC |
| Forte em | DDD, transparência SQL | Alta concorrência | Esquemas flexíveis | Cache, sessões, rate limiting | Busca full-text, análise |
| Transações | ACID completo | ACID (reativo) | Multi-documento (limitado) | MULTI/EXEC (limitado) | Não |
| Escala horizontal | Com sharding externo | Com sharding externo | Nativa (sharding) | Redis Cluster | Nativa (shards/replicas) |
| Persistência | Durável | Durável | Durável | Opcional (AOF/RDB) | Durável (índices invertidos) |
| Caso de uso típico | CRUD transacional | Streaming de eventos | Catálogos, CMS | Cache de API, sessões JWT | E-commerce search, logs |
