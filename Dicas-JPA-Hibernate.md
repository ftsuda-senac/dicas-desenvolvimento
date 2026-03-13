# Guia Avançado de JPA/Hibernate com Spring Boot

> Compilação de boas práticas, padrões e recursos avançados de JPA/Hibernate para aplicações Spring Boot com PostgreSQL.

---

## Sumário

1. [Consulta Eficiente de IDs em Batch](#1-consulta-eficiente-de-ids-em-batch)
2. [Herança JPA — Estratégias de Mapeamento](#2-herança-jpa--estratégias-de-mapeamento)
3. [Abordagem Mista com @SecondaryTable](#3-abordagem-mista-com-secondarytable)
4. [Resolução Polimórfica de Tipos](#4-resolução-polimórfica-de-tipos)
5. [Agrupamento por Tipo com Streams](#5-agrupamento-por-tipo-com-streams)
6. [Enum como Discriminator](#6-enum-como-discriminator)
7. [Discriminator Acessível via Getter](#7-discriminator-acessível-via-getter)
8. [ManyToMany e OneToMany na Mesma Tabela Intermediária](#8-manytomany-e-onetomany-na-mesma-tabela-intermediária)
9. [Cascade vs OrphanRemoval](#9-cascade-vs-orphanremoval)
10. [Cascade ALL Bidirecional — Problemas](#10-cascade-all-bidirecional--problemas)
11. [Recursos Pouco Conhecidos do JPA/Hibernate](#11-recursos-pouco-conhecidos-do-jpahibernate)
12. [Soft Delete Nativo (@SoftDelete)](#12-soft-delete-nativo-softdelete)
13. [Ordenação de Coleções (@OrderBy, @OrderColumn, @SortNatural)](#13-ordenação-de-coleções-orderby-ordercolumn-sortnatural)
14. [@ElementCollection vs @OneToMany](#14-elementcollection-vs-onetomany)
15. [@EmbeddedId vs @IdClass — PKs Compostas](#15-embeddedid-vs-idclass--pks-compostas)
16. [Recursos Adicionais do JPA/Hibernate](#16-recursos-adicionais-do-jpahibernate)
17. [OffsetDateTime e Conversão de Timezone](#17-offsetdatetime-e-conversão-de-timezone)
18. [Metamodel Estático do JPA](#18-metamodel-estático-do-jpa)
19. [Segurança de Queries JPQL com Campos Renomeados](#19-segurança-de-queries-jpql-com-campos-renomeados)
20. [@NamedQuery — Vantagens e Limitações](#20-namedquery--vantagens-e-limitações)
21. [@Modifying — flushAutomatically e clearAutomatically](#21-modifying--flushautomatically-e-clearautomatically)
22. [Remoção Eficiente de Itens em @OneToMany](#22-remoção-eficiente-de-itens-em-onetomany)
23. [Dígitos Verificadores com Value Objects e AttributeConverter](#23-dígitos-verificadores-com-value-objects-e-attributeconverter)
24. [Libs para Validação de Documentos Brasileiros](#24-libs-para-validação-de-documentos-brasileiros)
25. [@Column com AttributeConverter de Objetos Complexos](#25-column-com-attributeconverter-de-objetos-complexos)
26. [Records como @Embeddable](#26-records-como-embeddable)
27. [equals/hashCode em Entidades JPA](#27-equalshashcode-em-entidades-jpa)
28. [equals/hashCode em Hierarquias (@MappedSuperclass)](#28-equalshashcode-em-hierarquias-mappedsuperclass)
29. [getReferenceById — Proxy sem SELECT](#29-getreferencebyid--proxy-sem-select)
30. [Unique Constraints Compostas](#30-unique-constraints-compostas)
31. [Functions, Stored Procedures, Views e Materialized Views](#31-functions-stored-procedures-views-e-materialized-views)

---

## 1. Consulta Eficiente de IDs em Batch

### Validação pura — COUNT e comparação

```java
@Query("SELECT COUNT(e.id) FROM Entity e WHERE e.id IN :ids")
long countByIdIn(@Param("ids") Collection<Long> ids);
```

```java
if (repository.countByIdIn(ids) != ids.size()) {
    throw new EntityNotFoundException("Um ou mais IDs não encontrados");
}
```

### Identificar quais IDs faltam — projeção de IDs

```java
@Query("SELECT e.id FROM Entity e WHERE e.id IN :ids")
Set<Long> findExistingIds(@Param("ids") Collection<Long> ids);
```

```java
Set<Long> existing = repository.findExistingIds(ids);
Set<Long> missing = new HashSet<>(ids);
missing.removeAll(existing);
if (!missing.isEmpty()) {
    throw new EntityNotFoundException("IDs não encontrados: " + missing);
}
```

### PostgreSQL — `= ANY` para listas grandes

```java
@Query(value = "SELECT id FROM entity WHERE id = ANY(:ids)", nativeQuery = true)
Set<Long> findExistingIds(@Param("ids") Long[] ids);
```

O Hibernate 6.x já otimiza com `hibernate.query.in_list_padding` e suporte a arrays nativos do PostgreSQL.

### Antipattern: N+1 em loop

```java
// NUNCA faça isso
ids.forEach(id -> repository.findById(id)
    .orElseThrow(() -> new EntityNotFoundException(id)));
```

---

## 2. Herança JPA — Estratégias de Mapeamento

### SINGLE_TABLE (default)

Uma tabela com coluna discriminadora. Sem JOINs para queries polimórficas.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo", discriminatorType = DiscriminatorType.STRING)
public abstract class Pagamento {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private BigDecimal valor;
    private LocalDateTime criadoEm;
}

@Entity
@DiscriminatorValue("PIX")
public class PagamentoPix extends Pagamento {
    private String chave;
    private String txid;
}
```

**Prós:** query polimórfica sem JOIN, melhor performance para leitura. **Contras:** colunas dos subtipos ficam `NULL` — compensar com Bean Validation e check constraints condicionais.

```sql
ALTER TABLE pagamento
    ADD CONSTRAINT chk_pix_chave CHECK (tipo <> 'PIX' OR chave IS NOT NULL);
```

### JOINED

Uma tabela-base + uma tabela por subtipo. Queries polimórficas fazem `LEFT JOIN` em todas as subtabelas.

**Prós:** schema normalizado, `NOT NULL` real nos subtipos. **Contras:** N JOINs em queries polimórficas.

### TABLE_PER_CLASS

Uma tabela independente por classe concreta. Queries polimórficas viram `UNION ALL`.

**Prós:** tabelas autossuficientes. **Contras:** `UNION ALL` lento, não pode usar `IDENTITY`, FKs polimórficas impossíveis.

### @MappedSuperclass

Para reuso de campos sem relação polimórfica (campos de auditoria, por exemplo).

```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private LocalDateTime criadoEm;
}
```

### Comparação

| Critério | SINGLE_TABLE | JOINED | TABLE_PER_CLASS |
|---|---|---|---|
| Query polimórfica | Sem JOIN | N JOINs | UNION ALL |
| Query por subtipo | `WHERE tipo = ?` | JOIN simples | Direto na tabela |
| `NOT NULL` nos subtipos | Não | Sim | Sim |
| FK para a hierarquia | Simples | Simples | Impossível |
| Inserção | 1 INSERT | 2 INSERTs | 1 INSERT |

**Recomendação:** `SINGLE_TABLE` para a maioria dos casos. `JOINED` quando subtipos têm muitas colunas e integridade no DDL é requisito forte.

---

## 3. Abordagem Mista com @SecondaryTable

A spec JPA não permite misturar estratégias, mas `@SecondaryTable` atinge o mesmo efeito. Mantém `SINGLE_TABLE` e move subtipos complexos para tabela separada.

```java
@Entity
@DiscriminatorValue("CARTAO")
@SecondaryTable(
    name = "pagamento_cartao",
    pkJoinColumns = @PrimaryKeyJoinColumn(name = "pagamento_id")
)
public class PagamentoCartao extends Pagamento {

    @Column(table = "pagamento_cartao", nullable = false)
    private String bandeira;

    @Column(table = "pagamento_cartao", nullable = false)
    private Integer parcelas;
}
```

**Regra prática:** subtipos com 2–3 campos ficam inline. Subtipos com 5+ campos vão para `@SecondaryTable`. Queries polimórficas geram apenas 1 `LEFT JOIN` (para o subtipo separado), não N como `JOINED`.

| Cenário | JOINED (5 subtipos) | SINGLE_TABLE + @SecondaryTable (1) |
|---|---|---|
| Query polimórfica | 5 LEFT JOINs | 1 LEFT JOIN |
| Buscar Pix por ID | 1 JOIN | 0 JOINs |
| Buscar Cartão por ID | 1 JOIN | 1 JOIN |
| INSERT de Pix | 2 INSERTs | 1 INSERT |

---

## 4. Resolução Polimórfica de Tipos

O Hibernate lê a coluna discriminadora e instancia a classe concreta automaticamente:

```java
List<Pagamento> pagamentos = pagamentoRepository.findAll();
pagamentos.get(0).getClass(); // PagamentoPix.class
pagamentos.get(1).getClass(); // PagamentoCartao.class
```

Pattern matching funciona normalmente:

```java
for (Pagamento p : pagamentos) {
    String descricao = switch (p) {
        case PagamentoPix pix       -> "Pix para " + pix.getChave();
        case PagamentoCartao cartao -> cartao.getBandeira() + " em " + cartao.getParcelas() + "x";
        case PagamentoBoleto boleto -> "Boleto venc. " + boleto.getVencimento();
        default                     -> "Pagamento #" + p.getId();
    };
}
```

Repositories específicos por subtipo também funcionam:

```java
public interface PagamentoPixRepository extends JpaRepository<PagamentoPix, Long> {
    List<PagamentoPix> findByChave(String chave);
    // Hibernate gera: WHERE tipo = 'PIX' AND chave = ?
}
```

---

## 5. Agrupamento por Tipo com Streams

### Com `groupingBy` na classe

```java
Map<Class<? extends Pagamento>, List<Pagamento>> porTipo = pagamentoRepository.findAll()
    .stream()
    .collect(Collectors.groupingBy(Pagamento::getClass));
```

### Com enum discriminador (recomendado)

```java
Map<TipoPagamento, List<Pagamento>> porTipo = pagamentoRepository.findAll()
    .stream()
    .collect(Collectors.groupingBy(Pagamento::getTipo));
```

### Com record para tipagem completa e pattern matching

```java
public record PagamentosAgrupados(
    List<PagamentoPix> pix,
    List<PagamentoCartao> cartao,
    List<PagamentoBoleto> boleto
) {
    public static PagamentosAgrupados of(List<Pagamento> pagamentos) {
        var pix = new ArrayList<PagamentoPix>();
        var cartao = new ArrayList<PagamentoCartao>();
        var boleto = new ArrayList<PagamentoBoleto>();

        for (Pagamento p : pagamentos) {
            switch (p) {
                case PagamentoPix px    -> pix.add(px);
                case PagamentoCartao c  -> cartao.add(c);
                case PagamentoBoleto b  -> boleto.add(b);
                default -> { }
            }
        }

        return new PagamentosAgrupados(
            Collections.unmodifiableList(pix),
            Collections.unmodifiableList(cartao),
            Collections.unmodifiableList(boleto)
        );
    }
}
```

---

## 6. Enum como Discriminator

O JPA só aceita `STRING`, `INTEGER` e `CHAR` como `DiscriminatorType`. Use constantes compartilhadas para manter consistência:

```java
public enum TipoPagamento {

    PIX(Discriminator.PIX),
    CARTAO(Discriminator.CARTAO),
    BOLETO(Discriminator.BOLETO);

    private final String discriminator;

    TipoPagamento(String discriminator) {
        this.discriminator = discriminator;
    }

    public static final class Discriminator {
        public static final String PIX = "PIX";
        public static final String CARTAO = "CARTAO";
        public static final String BOLETO = "BOLETO";
        private Discriminator() {}
    }
}
```

```java
@Entity
@DiscriminatorValue(TipoPagamento.Discriminator.PIX)
public class PagamentoPix extends Pagamento { }
```

### Teste de consistência

```java
@Test
void discriminatorDeveCorresponderAoEnum() {
    var mapeamentos = Map.of(
        PagamentoPix.class,    TipoPagamento.PIX,
        PagamentoCartao.class, TipoPagamento.CARTAO,
        PagamentoBoleto.class, TipoPagamento.BOLETO
    );

    mapeamentos.forEach((classe, tipo) -> {
        String discriminator = classe.getAnnotation(DiscriminatorValue.class).value();
        assertThat(discriminator).isEqualTo(tipo.name());
    });
}
```

---

## 7. Discriminator Acessível via Getter

Mapeie a coluna discriminadora como campo read-only:

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo", discriminatorType = DiscriminatorType.STRING)
public abstract class Pagamento {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "tipo", insertable = false, updatable = false)
    @Enumerated(EnumType.STRING)
    private TipoPagamento tipo;

    public TipoPagamento getTipo() {
        return tipo;
    }
}
```

O `insertable = false, updatable = false` evita conflito com o mecanismo de discriminação do Hibernate. Subclasses herdam o getter sem precisar de override.

---

## 8. ManyToMany e OneToMany na Mesma Tabela Intermediária

Quando a tabela intermediária precisa de atributos próprios, promova-a a entidade:

```java
@Entity
public class Matricula {

    @EmbeddedId
    private MatriculaId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("cursoId")
    private Curso curso;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("alunoId")
    private Aluno aluno;

    private BigDecimal nota;
    private OffsetDateTime dataMatricula;

    @Enumerated(EnumType.STRING)
    private StatusMatricula status;
}
```

**Na prática, prefira só `@OneToMany`** e adicione método de conveniência:

```java
@Entity
public class Curso {

    @OneToMany(mappedBy = "curso", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Matricula> matriculas = new HashSet<>();

    public Set<Aluno> getAlunos() {
        return matriculas.stream()
            .map(Matricula::getAluno)
            .collect(Collectors.toUnmodifiableSet());
    }
}
```

---

## 9. Cascade vs OrphanRemoval

`Cascade` propaga operações do `EntityManager`. `OrphanRemoval` reage à desassociação de um filho do pai.

| Ação | `cascade = ALL` | `orphanRemoval = true` |
|---|---|---|
| `em.persist(curso)` | Persiste matrículas | — |
| `em.merge(curso)` | Atualiza matrículas | — |
| `em.remove(curso)` | Remove matrículas | Remove matrículas |
| `curso.getMatriculas().remove(m)` | Nada (ou seta NULL) | **Deleta `m` do banco** |
| `curso.getMatriculas().clear()` | Nada (ou seta NULL) | **Deleta todas do banco** |

Para composições fortes (filho sem sentido sem o pai), use ambos:

```java
@OneToMany(mappedBy = "curso", cascade = CascadeType.ALL, orphanRemoval = true)
private Set<Matricula> matriculas = new HashSet<>();
```

---

## 10. Cascade ALL Bidirecional — Problemas

`cascade = ALL` no lado `@ManyToOne` é quase sempre um bug:

- **REMOVE em cascata:** ao remover uma matrícula, propaga `REMOVE` para o curso, que deleta todas as matrículas, que propagam para todos os alunos — efeito dominó.
- **MERGE inesperado:** propaga `MERGE` para o pai, podendo sobrescrever dados com versões desatualizadas.

**Regra:** cascade segue a direção de dependência. Pai cascateia para filhos, nunca o contrário.

```java
@Entity
public class Matricula {
    @ManyToOne(fetch = FetchType.LAZY) // SEM cascade
    @JoinColumn(name = "curso_id", nullable = false)
    private Curso curso;
}
```

---

## 11. Recursos Pouco Conhecidos do JPA/Hibernate

### @Formula — campo calculado sem coluna

```java
@Formula("preco - desconto")
private BigDecimal precoFinal;

@Formula("(SELECT COUNT(*) FROM avaliacao a WHERE a.produto_id = id)")
private Long totalAvaliacoes;
```

### @SQLRestriction — filtro automático (substitui @Where deprecado no Hibernate 6.3+)

```java
@Entity
@SQLRestriction("deletado_em IS NULL")
public class Produto {
    private OffsetDateTime deletadoEm;
}
```

### @Filter — filtro dinâmico para multi-tenancy

```java
@Entity
@FilterDef(name = "tenantFilter", parameters = @ParamDef(name = "tenantId", type = Long.class))
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
public class Documento { }
```

### @NaturalId — cache e lookup por chave natural

```java
@NaturalId
@Column(nullable = false, unique = true)
private String ra;
```

### @DynamicUpdate — UPDATE só dos campos alterados

```java
@Entity
@DynamicUpdate
public class Pedido { }
```

### AttributeConverter — tipo custom transparente

```java
@Converter(autoApply = true)
public class CpfConverter implements AttributeConverter<Cpf, String> {
    @Override
    public String convertToDatabaseColumn(Cpf cpf) {
        return cpf == null ? null : cpf.valor();
    }
    @Override
    public Cpf convertToEntityAttribute(String valor) {
        return valor == null ? null : new Cpf(valor);
    }
}
```

### @CreationTimestamp e @UpdateTimestamp

```java
@MappedSuperclass
public abstract class BaseEntity {
    @CreationTimestamp
    @Column(updatable = false)
    private OffsetDateTime criadoEm;

    @UpdateTimestamp
    private OffsetDateTime atualizadoEm;
}
```

### @BatchSize — controle de N+1 sem JOIN FETCH

```java
@OneToMany(mappedBy = "curso")
@BatchSize(size = 25)
private Set<Matricula> matriculas;
```

### Projeções com interface

```java
public interface CursoResumo {
    Long getId();
    String getNome();
    long getTotalMatriculas();
}
```

### @Immutable — entidade read-only (sem dirty checking)

```java
@Entity
@Immutable
public class VwRelatorioMatriculas { }
```

---

## 12. Soft Delete Nativo (@SoftDelete)

Introduzido no Hibernate 6.4 (Spring Boot 3.3+):

```java
@Entity
@SoftDelete
public class Aluno { }
```

- `em.remove()` gera `UPDATE SET deleted = true` em vez de `DELETE`.
- Todas as queries adicionam `WHERE deleted = false` automaticamente.
- Funciona com `orphanRemoval` e `@ManyToMany`.

### Com timestamp

```java
@Entity
@SoftDelete(columnName = "deletado_em", converter = DeletedAtConverter.class)
public class Aluno { }
```

```java
public class DeletedAtConverter implements AttributeConverter<Boolean, OffsetDateTime> {
    @Override
    public OffsetDateTime convertToDatabaseColumn(Boolean deleted) {
        return deleted ? OffsetDateTime.now() : null;
    }
    @Override
    public Boolean convertToEntityAttribute(OffsetDateTime deletedAt) {
        return deletedAt != null;
    }
}
```

### Unique constraints com soft delete — partial index

```sql
CREATE UNIQUE INDEX uq_aluno_ra_ativo ON aluno (ra) WHERE deleted = false;
```

### Consultar deletados — query nativa

```java
@Query(value = "SELECT * FROM aluno WHERE deleted = true", nativeQuery = true)
List<Aluno> findAllDeletados();
```

---

## 13. Ordenação de Coleções (@OrderBy, @OrderColumn, @SortNatural)

### @OrderBy (JPA) — ORDER BY no SQL

```java
@OneToMany(mappedBy = "curso")
@OrderBy("criadoEm DESC")
private List<Matricula> matriculas = new ArrayList<>();
```

### @OrderColumn (JPA) — posição persistida no banco

```java
@OneToMany(mappedBy = "prova", cascade = CascadeType.ALL, orphanRemoval = true)
@OrderColumn(name = "posicao")
private List<Questao> questoes = new ArrayList<>();
```

### @SortNatural / @SortComparator (Hibernate) — ordenação em memória

```java
@OneToMany(mappedBy = "curso")
@SortNatural
private SortedSet<Matricula> matriculas = new TreeSet<>();
```

| Aspecto | @OrderBy | @OrderColumn | @SortNatural |
|---|---|---|---|
| Onde ordena | SQL | SQL (coluna posição) | Java (memória) |
| Persiste posição | Não | Sim | Não |
| Tipo de coleção | List, Set | List apenas | SortedSet, SortedMap |
| Custo | Baixo (índice) | Alto em reordenação | Proporcional ao tamanho |

---

## 14. @ElementCollection vs @OneToMany

### @ElementCollection — sem identidade própria

```java
@ElementCollection
@CollectionTable(name = "aluno_telefone", joinColumns = @JoinColumn(name = "aluno_id"))
@Column(name = "telefone")
private Set<String> telefones = new HashSet<>();
```

### Problema de performance

Ao modificar qualquer elemento, o Hibernate faz **delete-all + reinsert** (sem `@OrderColumn`), pois não há ID para fazer UPDATE pontual.

### Quando usar

- `@ElementCollection`: Value Objects pequenos e raramente modificados.
- `@OneToMany`: precisa de identidade, Repository, queries JPQL diretas, ou performance de UPDATE.

---

## 15. @EmbeddedId vs @IdClass — PKs Compostas

### @EmbeddedId — PK como Value Object

```java
@Embeddable
public record MatriculaId(Long cursoId, Long alunoId) implements Serializable {}

@Entity
public class Matricula {
    @EmbeddedId
    private MatriculaId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("cursoId")
    private Curso curso;
}
```

JPQL: `m.id.cursoId`. Spring Data: `findByIdCursoId()`.

### @IdClass — campos @Id separados

```java
@Entity
@IdClass(MatriculaId.class)
public class Matricula {
    @Id @ManyToOne(fetch = FetchType.LAZY)
    private Curso curso;

    @Id @ManyToOne(fetch = FetchType.LAZY)
    private Aluno aluno;
}
```

JPQL: `m.curso.id`. Spring Data: `findByCursoId()`.

**Recomendação:** `@EmbeddedId` quando a PK é um conceito de domínio. `@IdClass` quando a PK é consequência técnica do relacionamento.

---

## 16. Recursos Adicionais do JPA/Hibernate

### @Version — Optimistic Locking

```java
@Version
private Integer versao;
```

### @EntityGraph — controle de fetch por query

```java
@EntityGraph(attributePaths = {"matriculas", "professor"})
List<Curso> findByNomeContaining(String nome);
```

### @Fetch(FetchMode.SUBSELECT)

Resolve N+1 em uma única subselect, independente do tamanho.

### @ColumnTransformer — transformação no SQL

```java
@ColumnTransformer(
    read = "pgp_sym_decrypt(cpf_enc, current_setting('app.encryption_key'))",
    write = "pgp_sym_encrypt(?, current_setting('app.encryption_key'))"
)
private String cpf;
```

### @Subselect — mapear query como entidade read-only

```java
@Entity
@Immutable
@Subselect("SELECT c.id, c.nome, COUNT(m.aluno_id) AS total FROM curso c LEFT JOIN matricula m ...")
@Synchronize({"curso", "matricula"})
public class CursoEstatistica { }
```

### Hibernate Envers — auditoria com histórico

```java
@Entity
@Audited
public class Pedido { }
```

### @Generated — campos populados pelo banco

```java
@Generated
@Column(name = "numero", insertable = false, updatable = false)
private String numero;
```

### Spring Data Specifications — queries dinâmicas tipadas

```java
public static Specification<Pedido> comStatus(StatusPedido status) {
    return (root, query, cb) -> cb.equal(root.get("status"), status);
}
```

### @DynamicInsert — INSERT sem campos nulos (respeita DEFAULTs)

```java
@Entity
@DynamicInsert
public class Configuracao { }
```

---

## 17. OffsetDateTime e Conversão de Timezone

O PostgreSQL `timestamptz` converte tudo para UTC internamente e retorna no timezone da sessão.

### Configuração recomendada — forçar UTC

```yaml
spring:
  datasource:
    hikari:
      connection-init-sql: SET timezone = 'UTC'
  jpa:
    properties:
      hibernate:
        jdbc:
          time_zone: UTC
```

### Mapeamento correto

```java
@Column(nullable = false, updatable = false, columnDefinition = "TIMESTAMPTZ DEFAULT NOW()")
@CreationTimestamp
private OffsetDateTime criadoEm;
```

**Regra de ouro:** persista em UTC, converta na camada de apresentação.

### Fluxo completo

```
Java (OffsetDateTime -03:00)
  → Hibernate (jdbc.time_zone=UTC) converte para UTC
    → PostgreSQL armazena UTC (offset descartado)
      → Leitura retorna no timezone da sessão
        → Conversão para usuário na apresentação
```

---

## 18. Metamodel Estático do JPA

### Configuração

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <scope>provided</scope>
</dependency>
```

### Uso em Specifications (type-safe)

```java
public static Specification<Matricula> comStatus(StatusMatricula status) {
    return (root, query, cb) -> cb.equal(root.get(Matricula_.status), status);
}

public static Specification<Matricula> doCurso(Long cursoId) {
    return (root, query, cb) -> cb.equal(
        root.get(Matricula_.curso).get(Curso_.id), cursoId
    );
}
```

### SpecBuilder fluente com metamodel

```java
public class SpecBuilder<T> {
    private final List<Specification<T>> specs = new ArrayList<>();

    public <V> SpecBuilder<T> equalIfPresent(SingularAttribute<T, V> attr, V value) {
        if (value != null) {
            specs.add((root, query, cb) -> cb.equal(root.get(attr), value));
        }
        return this;
    }

    public Specification<T> build() {
        return specs.stream().reduce(Specification.where(null), Specification::and);
    }
}
```

---

## 19. Segurança de Queries JPQL com Campos Renomeados

| Tipo | Validação | Quando detecta |
|---|---|---|
| Derived query (`findByNota`) | Automática | Startup |
| `@Query` JPQL | Automática | Startup |
| `@NamedQuery` | Automática | Startup |
| Criteria com metamodel | Compilação | Build |
| Criteria com string | Manual | Runtime |
| `@Query` nativa | Manual | Runtime (teste) |

### Teste que valida o startup completo

```java
@SpringBootTest
class ApplicationContextTest {
    @Test
    void contextDevSubirSemErros() {
        // Se qualquer JPQL referencia campo inexistente, o contexto não sobe
    }
}
```

---

## 20. @NamedQuery — Vantagens e Limitações

### Vantagens

- Pré-compilação no startup.
- Validação antecipada de campos.
- Reutilização entre services via `entityManager.createNamedQuery()`.

### Na prática com Spring Data

O `@Query` no repository faz a mesma validação e fica mais próximo do uso. `@NamedQuery` é útil quando a mesma JPQL é compartilhada entre componentes que não passam pelo repository.

### Prioridade de resolução do Spring Data

1. `@Query` no método (maior prioridade)
2. `@NamedQuery` com nome matching (`Entidade.nomeDoMetodo`)
3. Derived query pelo nome do método

---

## 21. @Modifying — flushAutomatically e clearAutomatically

### flushAutomatically = true

Chama `entityManager.flush()` **antes** do bulk update — persiste mudanças pendentes.

### clearAutomatically = true

Chama `entityManager.clear()` **depois** do bulk update — esvazia o persistence context.

### Padrão seguro

```java
@Modifying(flushAutomatically = true, clearAutomatically = true)
@Query("UPDATE Matricula m SET m.status = 'CANCELADA' WHERE m.criadoEm < :limite")
int cancelarVencidas(@Param("limite") OffsetDateTime limite);
```

**Fluxo:** flush → execute → clear → próximas leituras buscam do banco.

**Cuidado:** `clear()` descarta **todas** as entidades gerenciadas, não apenas as afetadas.

---

## 22. Remoção Eficiente de Itens em @OneToMany

| Abordagem | Queries | Carrega coleção | Cascade/Lifecycle |
|---|---|---|---|
| `collection.remove()` | SELECT all + DELETE | Sim (tudo) | Sim |
| Bulk JPQL `DELETE` | 1 DELETE | Não | Não |
| `repository.delete(entity)` | 1 SELECT + 1 DELETE | Não | Sim |
| `getReference` + `remove` | 1 DELETE | Não | Sim |

### Recomendação

`repository.delete(entity)` — carrega só o filho, respeita lifecycle, não carrega a coleção inteira. Para performance máxima sem cascatas, bulk JPQL.

### Antipattern

Carregar o pai e acessar a coleção inteira só para remover um filho — evitar em coleções que podem crescer.

---

## 23. Dígitos Verificadores com Value Objects e AttributeConverter

### Value Object com validação no construtor

```java
public record Cpf(String valor) {
    private static final int[] PESOS_PRIMEIRO = {10, 9, 8, 7, 6, 5, 4, 3, 2};
    private static final int[] PESOS_SEGUNDO  = {11, 10, 9, 8, 7, 6, 5, 4, 3, 2};

    public Cpf {
        Objects.requireNonNull(valor, "CPF não pode ser nulo");
        String digitos = valor.replaceAll("\\D", "");
        if (digitos.length() != 11) throw new IllegalArgumentException("CPF deve ter 11 dígitos");
        if (digitos.chars().distinct().count() == 1) throw new IllegalArgumentException("CPF repetidos");
        if (!validarDigitos(digitos)) throw new IllegalArgumentException("CPF inválido: " + valor);
        valor = digitos;
    }

    private static boolean validarDigitos(String digitos) {
        int primeiro = Modulo11.calcularDigito(digitos, PESOS_PRIMEIRO);
        int segundo = Modulo11.calcularDigito(digitos, PESOS_SEGUNDO);
        return digitos.charAt(9) - '0' == primeiro && digitos.charAt(10) - '0' == segundo;
    }
}
```

### Módulo 11 genérico

```java
public final class Modulo11 {
    private Modulo11() {}

    public static int calcularDigito(String digitos, int[] pesos) {
        int soma = 0;
        for (int i = 0; i < pesos.length; i++) {
            soma += (digitos.charAt(i) - '0') * pesos[i];
        }
        int resto = soma % 11;
        return resto < 2 ? 0 : 11 - resto;
    }
}
```

### Bean Validation customizado

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = CpfValidator.class)
public @interface CpfValido {
    String message() default "CPF inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### Arquitetura

```
Request (String) → DTO + @CpfValido → new Cpf(valor) → Entity → CpfConverter → PostgreSQL (VARCHAR)
```

---

## 24. Libs para Validação de Documentos Brasileiros

### Apache Commons Validator

Possui `ModulusCheckDigit` abstrata com `MODULUS_11`, mas implementações são para ISBN, ISSN, EC Number — não para documentos brasileiros.

### Caelum Stella (recomendado para produção)

```xml
<dependency>
    <groupId>br.com.caelum.stella</groupId>
    <artifactId>caelum-stella-core</artifactId>
    <version>2.2.0</version>
</dependency>
```

- Validadores prontos: `CPFValidator`, `CNPJValidator`, NIT, RENAVAM, Título Eleitoral.
- `DigitoPara`: fluent interface genérica para Módulo 11.
- Bean Validation: `@CPF`, `@CNPJ` (módulo `caelum-stella-bean-validation`).
- Geração de valores válidos para testes: `generateRandomValid()`.

---

## 25. @Column com AttributeConverter de Objetos Complexos

As propriedades do `@Column` referem-se ao **tipo convertido** (String, Long no banco), não ao objeto Java.

```java
@Column(nullable = false, unique = true, length = 14, columnDefinition = "CHAR(14)")
private Cnpj cnpj; // persiste como CHAR(14) via CnpjConverter

@Column(nullable = false, precision = 19, scale = 2)
private Dinheiro valor; // converter para BigDecimal
```

### CHAR vs VARCHAR

- **Tamanho fixo** (CPF, CNPJ): `columnDefinition = "CHAR(n)"`.
- **Tamanho variável** (Email): `length = n` (VARCHAR padrão).

### Múltiplas colunas → @Embeddable

`AttributeConverter` é sempre 1 objeto → 1 coluna. Para múltiplas colunas, use `@Embeddable` com `@AttributeOverrides`.

---

## 26. Records como @Embeddable

Hibernate 6+ suporta nativamente. Usa o construtor canônico do record:

```java
@Embeddable
public record Endereco(
    @Column(nullable = false, length = 200) String logradouro,
    @Column(nullable = false, length = 100) String cidade,
    @Column(nullable = false, length = 2, columnDefinition = "CHAR(2)") String uf,
    @Column(nullable = false, length = 8, columnDefinition = "CHAR(8)") String cep
) {}
```

### Composição de records aninhados

```java
@Embeddable
public record Endereco(
    String logradouro,
    @AttributeOverride(name = "valor", column = @Column(name = "cep", columnDefinition = "CHAR(8)"))
    Cep cep
) {}
```

### Vantagens sobre classes

- Imutabilidade garantida.
- `equals/hashCode` automático por todos os campos.
- Construtor canônico com compact constructor para validação.

### Limitação

Se todas as colunas do embeddable forem `NULL` no banco, o Hibernate retorna `null` (não um record com valores nulos).

---

## 27. equals/hashCode em Entidades JPA

### Abordagem recomendada: equals por id + hashCode fixo

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Matricula that)) return false;
    return id != null && id.equals(that.id);
}

@Override
public int hashCode() {
    return getClass().hashCode();
}
```

- Duas entidades transient (id = null) nunca são iguais.
- `hashCode` fixo garante consistência em `HashSet` antes e depois do `persist`.
- Usar `instanceof` (não `getClass()`) para funcionar com proxies Hibernate.

### Abordagem alternativa: natural key

Quando existe chave natural imutável (atribuída antes do persist):

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Aluno that)) return false;
    return ra != null && ra.equals(that.ra);
}

@Override
public int hashCode() {
    return Objects.hash(ra);
}
```

### O que NÃO fazer

- `Objects.hash(id)` — muda após persist, quebra HashSet.
- `getClass() != o.getClass()` — quebra com proxies Hibernate.
- Usar todos os campos — quebra ao alterar qualquer atributo.

---

## 28. equals/hashCode em Hierarquias (@MappedSuperclass)

Centralizar na classe-base com `Hibernate.getClass()` para segurança com proxies:

```java
@MappedSuperclass
public abstract class BaseEntity implements Serializable {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof BaseEntity that)) return false;

        Class<?> thisType = Hibernate.getClass(this);
        Class<?> thatType = Hibernate.getClass(that);
        if (thisType != thatType) return false;

        return getId() != null && getId().equals(that.getId());
    }

    @Override
    public int hashCode() {
        return 31; // constante fixa para TODA a hierarquia
    }
}
```

- `Hibernate.getClass()` resolve o tipo real mesmo com proxy.
- Subclasses **não** sobrescrevem — herdam de `BaseEntity`.
- Exceção: subclasse com natural key pode sobrescrever quando performance de Set importa.

---

## 29. getReferenceById — Proxy sem SELECT

Retorna um proxy não inicializado. SELECT só acontece ao acessar campo que não seja id.

### Uso principal: setar FK sem carregar

```java
@Transactional
public Matricula criar(CriarMatriculaRequest request) {
    var matricula = new Matricula();
    matricula.setCurso(cursoRepository.getReferenceById(request.cursoId()));
    matricula.setAluno(alunoRepository.getReferenceById(request.alunoId()));
    return repository.save(matricula);
    // 1 INSERT, 0 SELECTs
}
```

### Deletar sem carregar

```java
cursoRepository.delete(cursoRepository.getReferenceById(id));
// 1 DELETE, 0 SELECTs
```

### Quando evitar

- Precisa validar existência ou acessar dados → use `findById`.
- Fora de `@Transactional` → `LazyInitializationException` ao acessar campos.

| Aspecto | findById | getReferenceById |
|---|---|---|
| Retorno | `Optional<T>` | `T` (proxy) |
| SELECT imediato | Sim | Não |
| Entidade não existe | `Optional.empty()` | Exception (no acesso) |

---

## 30. Unique Constraints Compostas

### @UniqueConstraint — sempre com nome legível

```java
@Table(
    name = "matricula",
    uniqueConstraints = @UniqueConstraint(
        name = "uk_matricula_curso_aluno",
        columnNames = {"curso_id", "aluno_id"}
    )
)
public class Matricula { }
```

### Tratamento de violação por nome

```java
@ExceptionHandler(DataIntegrityViolationException.class)
public ProblemDetail handleDuplicado(DataIntegrityViolationException ex) {
    if (ex.getMostSpecificCause().getMessage().contains("uk_matricula_curso_aluno")) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, "Aluno já matriculado");
    }
    return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, "Registro duplicado");
}
```

### Com soft delete — partial index via Flyway

```sql
CREATE UNIQUE INDEX uk_matricula_curso_aluno_ativo
    ON matricula (curso_id, aluno_id) WHERE deleted = false;
```

### Case-insensitive — functional index via Flyway

```sql
CREATE UNIQUE INDEX uk_aluno_email_lower ON aluno (LOWER(email));
```

### Validação preventiva + constraint como defesa

```java
if (repository.existsByCursoIdAndAlunoId(cursoId, alunoId)) {
    throw new MatriculaDuplicadaException(cursoId, alunoId);
}
```

Verificação otimista — constraint é indispensável como última defesa contra race conditions.

---

## 31. Functions, Stored Procedures, Views e Materialized Views

### Views — @Entity + @Immutable

```java
@Entity
@Immutable
@Table(name = "vw_curso_estatistica")
public class CursoEstatistica {
    @Id private Long id;
    private String nome;
    private Long totalAlunos;
}
```

### Materialized Views — mesmo + refresh

```java
@Modifying
@Query(value = "REFRESH MATERIALIZED VIEW CONCURRENTLY mv_ranking_alunos", nativeQuery = true)
void refresh();
```

### Functions — query nativa, @Formula, ou registro no JPQL

```java
@Formula("calcular_media_ponderada(id)")
private BigDecimal mediaPonderada;
```

### Stored Procedures — @Procedure ou EntityManager

```java
@Procedure(procedureName = "fechar_matriculas_vencidas")
Long fecharMatriculasVencidas(@Param("p_data_limite") OffsetDateTime dataLimite);
```

### Configuração de variáveis de sessão PostgreSQL (para @ColumnTransformer)

```yaml
spring:
  datasource:
    hikari:
      connection-init-sql: SET app.encryption_key = '${app.encryption-key}'
```

Para isolamento por transação, use `SET LOCAL` via `DataSource` wrapper ou aspecto.

| Recurso | Mapeamento JPA | Escrita | Refresh |
|---|---|---|---|
| View | @Entity + @Immutable | Não | Automático |
| Materialized View | @Entity + @Immutable | Não | Manual |
| Function (escalar) | @Formula, query nativa | Não | — |
| Stored Procedure | @Procedure, EntityManager | Sim | — |

---

## Referências

- [Hibernate ORM 6 User Guide](https://docs.jboss.org/hibernate/orm/6.6/userguide/html_single/Hibernate_User_Guide.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Vlad Mihalcea — High-Performance Java Persistence](https://vladmihalcea.com/)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Caelum Stella](https://github.com/caelum/caelum-stella)
- [Apache Commons Validator](https://commons.apache.org/proper/commons-validator/)
