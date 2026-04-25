# Spring — Java, Spring Framework e Camada de Serviço

> **Baseline principal:** Spring Boot 3.5 · Spring Framework 6.x · Java 21+
>
> Este documento reúne os fundamentos de Java, Spring Framework e as boas práticas
> da camada de serviço — base compartilhada pelos guias
> [REST](Dicas-Spring-MVC-REST.md) e [SSR](Dicas-Spring-MVC-SSR.md).

---

## Sumário

1. [Java — Recursos da Linguagem Relevantes para o Documento](#1-java-recursos-da-linguagem-relevantes-para-o-documento)
2. [Spring Framework: IoC, DI e Anotações Essenciais](#2-base-spring-framework-ioc-di-e-anotações-essenciais)
3. [Boas Práticas na Camada de Serviço (`@Service`)](#3-boas-práticas-na-camada-de-serviço-service)
    - [3.1 Responsabilidades e Estrutura](#31-responsabilidades-e-estrutura)
    - [3.2 `@Transactional` — Padrões e Armadilhas](#32-transactional-padrões-e-armadilhas)
    - [3.3 Mapeamento DTO ↔ Entidade](#33-mapeamento-dto-entidade)
    - [3.4 Validação no Service — Invariantes de Domínio](#34-validação-no-service-invariantes-de-domínio)
    - [3.5 Tratamento de Exceções no Service](#35-tratamento-de-exceções-no-service)
    - [3.6 Services Stateless — Evitar Estado em Campos](#36-services-stateless-evitar-estado-em-campos)
    - [3.7 Checklist — Boas Práticas do `@Service`](#37-checklist-boas-práticas-do-service)
4. [Classes Utilitárias do Spring](#4-classes-utilitárias-do-spring)
    - [4.1 `StringUtils` — Validação e Manipulação de Strings](#41-stringutils-validação-e-manipulação-de-strings)
    - [4.2 `Assert` — Pré-condições e Invariantes](#42-assert-pré-condições-e-invariantes)
    - [4.3 `CollectionUtils` — Operações Null-Safe em Coleções](#43-collectionutils-operações-null-safe-em-coleções)
    - [4.4 `ObjectUtils` — Operações Null-Safe em Objetos](#44-objectutils-operações-null-safe-em-objetos)
    - [4.5 `ClassUtils` — Introspecção de Classes](#45-classutils-introspecção-de-classes)
    - [4.6 `BeanUtils` — Cópia de Propriedades entre Objetos](#46-beanutils-cópia-de-propriedades-entre-objetos)
    - [4.7 `ReflectionUtils` — Reflexão Sem Checked Exceptions](#47-reflectionutils-reflexão-sem-checked-exceptions)
    - [4.8 `StopWatch` — Medição de Desempenho](#48-stopwatch-medição-de-desempenho)
    - [4.9 `AntPathMatcher` — Correspondência de Padrões de Caminho](#49-antpathmatcher-correspondência-de-padrões-de-caminho)
    - [4.10 `FileCopyUtils` — Operações de I/O com Cópia Automática](#410-filecopyutils-operações-de-io-com-cópia-automática)
    - [4.11 `MultiValueMap` e `LinkedMultiValueMap` — Mapas com Múltiplos Valores por Chave](#411-multivaluemap-e-linkedmultivaluemap--mapas-com-múltiplos-valores-por-chave)
    - [4.12 `LinkedCaseInsensitiveMap` — Mapa com Chaves Case-Insensitive](#412-linkedcaseinsensitivemap--mapa-com-chaves-case-insensitive)
    - [4.13 `DigestUtils` — Cálculo de Digests MD5](#413-digestutils--cálculo-de-digests-md5)
    - [4.14 `SerializationUtils` — Serialização e Clone de Objetos](#414-serializationutils--serialização-e-clone-de-objetos)
    - [4.15 `MethodInvoker` — Invocação Declarativa de Métodos](#415-methodinvoker--invocação-declarativa-de-métodos)
    - [4.16 `PathMatcher` e `RouteMatcher` — Interfaces de Correspondência](#416-pathmatcher-e-routematcher--interfaces-de-correspondência)
    - [4.17 `ResourceUtils` — Resolução de Recursos no Classpath e Sistema de Arquivos](#417-resourceutils--resolução-de-recursos-no-classpath-e-sistema-de-arquivos)
    - [4.18 `MimeTypeUtils` — Tipos MIME](#418-mimetypeutils--tipos-mime)
    - [4.19 `NumberUtils` — Conversão e Parsing de Números](#419-numberutils--conversão-e-parsing-de-números)
    - [4.20 `ConcurrentReferenceHashMap` — Cache Thread-Safe com Referências Fracas/Suaves](#420-concurrentreferencehashmap--cache-thread-safe-com-referências-fracassuaves)
    - [4.21 `ExponentialBackOff` e `FixedBackOff` — Estratégias de Retry com Backoff](#421-exponentialbackoff-e-fixedbackoff--estratégias-de-retry-com-backoff)
    - [4.22 `IdGenerator` e Implementações — Geração de UUIDs](#422-idgenerator-e-implementações--geração-de-uuids)
    - [4.23 `PropertyPlaceholderHelper` — Resolução de Placeholders em Strings](#423-propertyplaceholderhelper--resolução-de-placeholders-em-strings)
    - [4.24 `SocketUtils` / `TestSocketUtils` — Portas TCP Disponíveis em Testes](#424-socketutils--testSocketutils--portas-tcp-disponíveis-em-testes)
    - [4.25 `DataSize` — Representação de Tamanhos de Dados](#425-datasize--representação-de-tamanhos-de-dados)
    - [4.26 Resumo das Classes Utilitárias](#426-resumo-das-classes-utilitárias)

---

## 1. Java — Recursos da Linguagem Relevantes para o Documento

Esta seção cobre recursos do Java puro (sem Spring) que aparecem com frequência
ao longo do documento. O objetivo é servir como referência rápida e nivelamento
de conhecimento antes das seções de framework.

### 1.1 `Optional<T>` — Representação Explícita de Ausência de Valor

`Optional<T>` é um container que pode conter ou não um valor não-nulo. Substitui
o retorno de `null` em métodos que podem legitimamente não ter resultado, tornando
a ausência explícita no contrato do método.

```java
// ─── Criação ──────────────────────────────────────────────────────────────────
Optional<String> comValor  = Optional.of("Spring MVC");        // nunca aceita null
Optional<String> vazio     = Optional.empty();
Optional<String> talvezNull = Optional.ofNullable(valorQuePodeSerNull); // aceita null

// ─── Verificação e acesso ─────────────────────────────────────────────────────
boolean presente = comValor.isPresent();   // true
boolean ausente  = vazio.isEmpty();        // true (Java 11+)

String valor     = comValor.get();         // ⚠️ lança NoSuchElementException se vazio
String seguro    = comValor.orElse("default");          // retorna "default" se vazio
String calculado = vazio.orElseGet(() -> calcular());   // avaliação lazy
String obrigatorio = vazio.orElseThrow(
    () -> new RecursoNaoEncontradoException("Produto", 42L)); // lança exceção customizada

// ─── Transformação — map e flatMap ────────────────────────────────────────────
Optional<Integer> tamanho = comValor.map(String::length);        // Optional<Integer>(10)
Optional<String>  upper   = comValor.map(String::toUpperCase);   // Optional<String>("SPRING MVC")

// flatMap: quando a função de mapeamento já retorna Optional (evita Optional<Optional<T>>)
Optional<Endereco> endereco = clienteOpt.flatMap(Cliente::getEnderecoOpcional);

// ─── Filtragem ────────────────────────────────────────────────────────────────
Optional<String> longo = comValor.filter(s -> s.length() > 5);  // presente se atende o predicado

// ─── Execução condicional (sem if-null explícito) ─────────────────────────────
comValor.ifPresent(s -> System.out.println("Valor: " + s));

// ifPresentOrElse (Java 9+): ação se presente, ação alternativa se vazio
comValor.ifPresentOrElse(
    s  -> System.out.println("Encontrado: " + s),
    () -> System.out.println("Não encontrado")
);

// or (Java 9+): substitui por outro Optional se vazio
Optional<Produto> resultado = repositorio.findById(id)
        .or(() -> repositorio.findByCodigoLegado(id));  // tenta fallback

// stream (Java 9+): converte Optional em Stream de 0 ou 1 elemento
List<String> nomes = optionals.stream()
        .flatMap(Optional::stream)     // remove os vazios e desembala os presentes
        .toList();
```

**Uso em repositórios e controllers (padrão no documento):**

```java
// ─── Repositório retorna Optional ─────────────────────────────────────────────
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    Optional<Produto> findBySkuIgnoreCase(String sku);
}

// ─── Service usa orElseThrow para transformar ausência em exceção de domínio ──
@Service
public class ProdutoService {

    public ProdutoResponse buscarPorId(Long id) {
        return produtoRepository.findById(id)
                .map(ProdutoResponse::from)                   // transforma se presente
                .orElseThrow(() ->                            // lança se vazio
                        new RecursoNaoEncontradoException("Produto", id));
    }

    public Optional<ProdutoResponse> buscarPorSku(String sku) {
        // Retorna Optional quando a ausência é um resultado válido (não um erro)
        return produtoRepository.findBySkuIgnoreCase(sku)
                .map(ProdutoResponse::from);
    }
}

// ─── Controller trata Optional com map/orElseThrow ou orElse ──────────────────
@GetMapping("/{id}")
public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) {
    return produtoService.buscarPorSku("ABC-123")
            .map(ResponseEntity::ok)                          // 200 se presente
            .orElse(ResponseEntity.notFound().build());       // 404 se vazio
}
```

**O que evitar com `Optional`:**

```java
// ❌ NÃO use Optional como campo de classe — use null com @Nullable
public class Produto {
    private Optional<String> descricao;  // ← errado; Optional não é Serializable seguro
}

// ❌ NÃO use Optional como parâmetro de método — use sobrecarga ou @Nullable
public void processar(Optional<Filtro> filtro) { }  // ← verboso, desnecessário

// ❌ NÃO chame get() sem verificar — prefira orElse/orElseThrow
String s = optional.get();  // ← lança NoSuchElementException se vazio

// ✅ Padrão correto: Optional apenas como tipo de retorno de métodos
public Optional<Produto> findBySku(String sku) { /* ... */ }
```

---

### 1.2 Interfaces — Contratos, Implementações Múltiplas e Padrões do Java

Interfaces definem **contratos** sem implementação (exceto `default` e `static`).
São a base do polimorfismo, do princípio de Segregação de Interfaces (ISP) e da
Inversão de Dependência (DIP) — dois dos cinco princípios SOLID.

```java
// ─── Interface básica — define o contrato ─────────────────────────────────────
public interface Exporter {
    byte[] exportar(Long relatorioId);

    // default: implementação opcional — subclasses podem sobrescrever
    default String getNomeArquivo(Long relatorioId) {
        return "relatorio-" + relatorioId + "." + getExtensao();
    }

    // método abstrato que toda implementação deve fornecer
    String getExtensao();

    // static: utilitário da interface, não herdado
    static boolean formatoSuportado(String formato) {
        return Set.of("pdf", "csv", "xlsx").contains(formato.toLowerCase());
    }
}

// ─── Múltiplas implementações do mesmo contrato ───────────────────────────────
@Component("pdfExporter")
public class PdfExporter implements Exporter {
    @Override public byte[] exportar(Long id) { /* gera PDF */ return new byte[0]; }
    @Override public String getExtensao()      { return "pdf"; }
}

@Component("csvExporter")
public class CsvExporter implements Exporter {
    @Override public byte[] exportar(Long id) { /* gera CSV */ return new byte[0]; }
    @Override public String getExtensao()      { return "csv"; }
}

// ─── Uso polimórfico: o código depende da interface, não da implementação ──────
@Service
public class RelatorioService {
    // Depende de Exporter, não de PdfExporter ou CsvExporter.
    // Trocar a implementação não exige nenhuma alteração aqui.
    private final Exporter exporter;

    public RelatorioService(@Qualifier("pdfExporter") Exporter exporter) {
        this.exporter = exporter;
    }
}
```

**Interfaces funcionais e lambdas — visão rápida:**

```java
// Interfaces com exatamente UM método abstrato podem ser usadas com lambdas.
// Ver seção 1.5 para o catálogo completo de java.util.function.

// Supplier<T>: sem parâmetros, retorna T
Supplier<Produto>  supplier  = () -> new Produto("Notebook", BigDecimal.TEN);

// Consumer<T>: recebe T, sem retorno
Consumer<String>   consumer  = nome -> System.out.println("Olá, " + nome);

// Function<T,R>: recebe T, retorna R
Function<Produto, String> nomeFn = produto -> produto.getNome().toUpperCase();

// Predicate<T>: recebe T, retorna boolean
Predicate<Produto> ativo = produto -> produto.isAtivo();

// BiFunction<T,U,R>: recebe T e U, retorna R
BiFunction<String, BigDecimal, Produto> factory =
        (nome, preco) -> new Produto(nome, preco);

// ─── Uso em Streams — composição funcional ────────────────────────────────────
List<ProdutoResponse> ativos = produtos.stream()
        .filter(Predicate.not(Produto::isRemovido))   // Predicate
        .filter(p -> p.getEstoque() > 0)              // lambda Predicate
        .map(ProdutoResponse::from)                   // Function — method reference
        .sorted(Comparator.comparing(ProdutoResponse::nome))
        .toList();                                    // Java 16+ — lista imutável

// ─── Interface funcional customizada ─────────────────────────────────────────
@FunctionalInterface
public interface RelatorioGenerator<T> {
    byte[] gerar(T dados, Locale locale);

    // default e static são permitidos em @FunctionalInterface
    default String cabecalho() { return "Relatório"; }
}

// Uso como lambda:
RelatorioGenerator<List<Produto>> gen =
        (dados, locale) -> pdfService.renderizar(dados, locale);
```

**Interfaces no contexto Spring MVC (usadas ao longo do documento):**

| Interface | Onde aparece | Propósito |
|---|---|---|
| `WebMvcConfigurer` | Seção 2.2 | Personalizar o Spring MVC sem substituir a auto-configuração |
| `HandlerInterceptor` | Seção 11.1 | Interceptar requisições antes/depois do controller |
| `Converter<S,T>` | Seção 7 | Converter tipos no binding de parâmetros |
| `Formatter<T>` | Seção 7 | Formatar/parsear tipos com locale |
| `ErrorController` | Seção 18 | Personalizar o endpoint `/error` |
| `CorsConfigurationSource` | Seção 14 | Prover configuração CORS dinâmica |
| `UserDetails` | Seção 11.12 | Contrato do usuário autenticado no Spring Security |

---

### 1.3 POJO, `class` e `record` — Modelagem de Dados

**POJO (Plain Old Java Object)** é um objeto Java sem herança obrigatória de
frameworks, sem anotações intrusivas, representando apenas dados ou comportamento
de domínio.

#### 1.3.1 POJO com `class` — mutável, com getters/setters

```java
// ─── POJO clássico — necessário quando o objeto precisa ser mutável ───────────
// Casos de uso: form objects (seção 4.2), entidades JPA, objetos acumuladores
public class ProdutoForm {

    // Campos privados — encapsulamento
    private String     nome;
    private BigDecimal preco;
    private Integer    estoque;
    private Long       categoriaId;
    private boolean    ativo = true;   // valor padrão

    // Construtor padrão obrigatório para frameworks de binding (Thymeleaf, Jackson)
    public ProdutoForm() {}

    // Construtor de conveniência
    public ProdutoForm(String nome, BigDecimal preco) {
        this.nome  = nome;
        this.preco = preco;
    }

    // Getters e setters — necessários para binding MVC e serialização Jackson
    public String getNome()              { return nome; }
    public void setNome(String nome)     { this.nome = nome; }

    public BigDecimal getPreco()                 { return preco; }
    public void setPreco(BigDecimal preco)        { this.preco = preco; }

    public Integer getEstoque()                  { return estoque; }
    public void setEstoque(Integer estoque)       { this.estoque = estoque; }

    public Long getCategoriaId()                 { return categoriaId; }
    public void setCategoriaId(Long categoriaId) { this.categoriaId = categoriaId; }

    public boolean isAtivo()                     { return ativo; }
    public void setAtivo(boolean ativo)          { this.ativo = ativo; }

    // equals, hashCode, toString (gerados pela IDE ou Lombok @Data)
    @Override public String toString() {
        return "ProdutoForm{nome='%s', preco=%s}".formatted(nome, preco);
    }
}
```

#### 1.3.2 `record` — POJO imutável conciso (Java 16+)

`record` gera automaticamente: construtor canônico, getters (sem prefixo `get`),
`equals`, `hashCode` e `toString`. É imutável por design — ideal para DTOs,
value objects e respostas de API.

```java
// ─── Record básico ────────────────────────────────────────────────────────────
// Equivale a uma class com 4 campos final, construtor, getters, equals, hashCode, toString
public record ProdutoResponse(
        Long       id,
        String     nome,
        BigDecimal preco,
        String     categoria
) {}

// Acesso: produto.id(), produto.nome(), produto.preco() — sem prefixo "get"
ProdutoResponse p = new ProdutoResponse(1L, "Notebook", new BigDecimal("3499.99"), "TI");
System.out.println(p.nome());    // "Notebook"
System.out.println(p);           // ProdutoResponse[id=1, nome=Notebook, ...]

// ─── Record com anotações de validação ────────────────────────────────────────
public record ProdutoRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 200)
    String nome,

    @NotNull
    @Positive
    BigDecimal preco,

    @NotNull
    Long categoriaId
) {}

// ─── Record com construtor compacto — validação customizada ───────────────────
public record Intervalo(LocalDate inicio, LocalDate fim) {

    // Construtor compacto: sem parâmetros explícitos — acessa diretamente os campos
    // Executado antes da atribuição dos campos
    public Intervalo {
        if (inicio.isAfter(fim)) {
            throw new IllegalArgumentException(
                    "Início (%s) não pode ser após o fim (%s)".formatted(inicio, fim));
        }
        // Normalização: campos são atribuídos automaticamente após o bloco
    }
}

// ─── Record com método factory e método de instância ─────────────────────────
public record PaginaInfo(int numero, int tamanho, long total) {

    // Método factory estático
    public static PaginaInfo of(Pageable pageable, long total) {
        return new PaginaInfo(pageable.getPageNumber(), pageable.getPageSize(), total);
    }

    // Método de instância — records podem ter métodos
    public int totalPaginas() {
        return tamanho == 0 ? 0 : (int) Math.ceil((double) total / tamanho);
    }

    public boolean temProxima() {
        return numero < totalPaginas() - 1;
    }
}

// ─── Record com implementação de interface ────────────────────────────────────
public record CoordenadaGPS(double latitude, double longitude) implements Serializable {

    // Construtor compacto com validação
    public CoordenadaGPS {
        if (latitude  < -90  || latitude  > 90)  throw new IllegalArgumentException("Latitude inválida");
        if (longitude < -180 || longitude > 180)  throw new IllegalArgumentException("Longitude inválida");
    }

    public double distanciaKm(CoordenadaGPS outro) {
        // Haversine simplificado
        double r = 6371;
        double dLat = Math.toRadians(outro.latitude  - this.latitude);
        double dLon = Math.toRadians(outro.longitude - this.longitude);
        double a = Math.sin(dLat/2) * Math.sin(dLat/2)
                 + Math.cos(Math.toRadians(this.latitude))
                 * Math.cos(Math.toRadians(outro.latitude))
                 * Math.sin(dLon/2) * Math.sin(dLon/2);
        return r * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    }
}
```

**Comparativo `class` vs `record`:**

| Característica | `class` (POJO) | `record` |
|---|---|---|
| Imutabilidade | ❌ Mutável por padrão | ✅ Imutável por padrão |
| Boilerplate | Alto (getters, setters, equals...) | ✅ Zero — tudo gerado |
| Getters | `getNome()` | `nome()` (sem prefixo) |
| Setters | Sim | ❌ Não (campos `final`) |
| Herança | Pode herdar e ser herdada | ❌ `final` implícito — não pode ser herdada |
| Binding MVC/Thymeleaf | ✅ Form objects (mutável) | ⚠️ Apenas leitura (sem setter) |
| Jackson / JSON | ✅ Com configuração | ✅ Funciona sem configuração extra |
| JPA `@Entity` | ✅ Necessário | ❌ Não use — JPA exige mutabilidade |
| **Uso típico** | Form objects, entidades JPA | DTOs de request/response, value objects |

---

### 1.4 Text Blocks — Strings Multilinha (Java 15+)

Text blocks (`"""..."""`) permitem escrever strings multilinha sem concatenação,
escapamentos excessivos ou quebras de legibilidade. São especialmente úteis para
JSON, SQL, HTML e outros conteúdos estruturados inline.

```java
// ─── String tradicional vs text block ────────────────────────────────────────

// Antes (Java 14 e anterior) — ilegível, propenso a erro
String jsonAntigo = "{\n" +
                    "  \"nome\": \"Notebook\",\n" +
                    "  \"preco\": 3499.99,\n" +
                    "  \"ativo\": true\n" +
                    "}";

// Agora (Java 15+) — legível, indentação preservada automaticamente
String json = """
        {
          "nome": "Notebook",
          "preco": 3499.99,
          "ativo": true
        }
        """;
// A indentação do fechamento """ define o nível de corte da indentação:
// qualquer espaço comum a todas as linhas é removido automaticamente.
// O \n final existe porque """ fechando está em linha própria.

// ─── Uso em testes — payloads de request ─────────────────────────────────────
restTestClient.post()
        .uri("/api/v1/produtos")
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue("""
                {
                  "nome": "Notebook Dell",
                  "preco": 3499.99,
                  "categoriaId": 1
                }
                """)
        .exchange()
        .expectStatus().isCreated();

// ─── SQL multilinha — muito mais legível ─────────────────────────────────────
String sql = """
        SELECT p.id, p.nome, p.preco, c.nome AS categoria
        FROM produtos p
        JOIN categorias c ON c.id = p.categoria_id
        WHERE p.ativo = true
          AND p.estoque > :estoqueMinimo
        ORDER BY p.nome
        LIMIT :limite
        """;

// ─── HTML inline (ex: templates de e-mail simples) ───────────────────────────
String html = """
        <html>
          <body>
            <h1>Pedido confirmado</h1>
            <p>Olá, %s! Seu pedido <strong>#%d</strong> foi confirmado.</p>
          </body>
        </html>
        """.formatted(nomeCliente, numeroPedido);
// .formatted() é o equivalente de String.format() em text blocks

// ─── Interpolação de variáveis com formatted() ───────────────────────────────
String mensagem = """
        Produto: %s
        Preço:   R$ %.2f
        Estoque: %d unidades
        """.formatted(produto.getNome(), produto.getPreco(), produto.getEstoque());

// ─── Controle preciso de nova linha e espaços ─────────────────────────────────
// \  no final da linha: continua na mesma linha (sem \n)
String umaLinha = """
        Esta é uma string \
        que continua \
        na mesma linha.
        """;  // "Esta é uma string que continua na mesma linha."

// \s no final da linha: preserva espaços à direita (que seriam removidos)
String comEspacos = """
        col1     \s
        col2     \s
        """;

// ─── Uso nos exemplos do documento ───────────────────────────────────────────
// Text blocks aparecem principalmente em:
//   - Payloads de teste (seção 3)
//   - Mensagens Javadoc de constraints (seção 5)
//   - Templates de e-mail em serviços de notificação
//   - Queries SQL nativas com JdbcClient
```

---

### 1.5 Programação Funcional — `java.util.function`, Composição e Method References

O pacote `java.util.function` define as interfaces funcionais padrão do Java.
Todas possuem exatamente **um método abstrato** (`@FunctionalInterface`) e por
isso aceitam lambdas e method references como implementações.

#### 1.5.1 Catálogo das interfaces principais

```java
// ════════════════════════════════════════════════════════════════════════════════
// GRUPO 1 — Transformação (recebe algo, retorna algo diferente)
// ════════════════════════════════════════════════════════════════════════════════

// Function<T, R> — recebe T, produz R
// Método abstrato: R apply(T t)
Function<String, Integer>    tamanho  = String::length;        // "abc" → 3
Function<Produto, String>    nomeFn   = Produto::getNome;      // Produto → "Notebook"
Function<String, LocalDate>  parseData = LocalDate::parse;    // "2024-01-15" → LocalDate

// BiFunction<T, U, R> — recebe T e U, produz R
// Método abstrato: R apply(T t, U u)
BiFunction<String, Integer, String> repetir =
        (s, n) -> s.repeat(n);                                 // ("AB", 3) → "ABABAB"
BiFunction<BigDecimal, BigDecimal, BigDecimal> soma =
        BigDecimal::add;                                       // (1.0, 2.0) → 3.0

// UnaryOperator<T> — especialização de Function<T,T> (entrada e saída do mesmo tipo)
// Método abstrato: T apply(T t)
UnaryOperator<String>     maiusculo = String::toUpperCase;     // "abc" → "ABC"
UnaryOperator<BigDecimal> dobrar    = v -> v.multiply(BigDecimal.TWO);

// BinaryOperator<T> — especialização de BiFunction<T,T,T>
// Método abstrato: T apply(T t1, T t2)
BinaryOperator<BigDecimal> maior    = BigDecimal::max;         // (3, 5) → 5
BinaryOperator<String>     concat   = String::concat;          // ("a","b") → "ab"

// ════════════════════════════════════════════════════════════════════════════════
// GRUPO 2 — Produção (sem entrada, retorna algo)
// ════════════════════════════════════════════════════════════════════════════════

// Supplier<T> — sem parâmetros, fornece T (lazy evaluation)
// Método abstrato: T get()
Supplier<Instant>  agora      = Instant::now;                  // () → Instant.now()
Supplier<Produto>  novoProduto = Produto::new;                 // () → new Produto()
Supplier<List<String>> listaVazia = ArrayList::new;            // () → new ArrayList<>()

// Uso típico: valor default lazy (só avaliado se necessário)
String nome = optional.orElseGet(() -> gerarNomeDefault());    // Supplier<String>

// ════════════════════════════════════════════════════════════════════════════════
// GRUPO 3 — Consumo (recebe algo, sem retorno — side effect)
// ════════════════════════════════════════════════════════════════════════════════

// Consumer<T> — recebe T, sem retorno (void)
// Método abstrato: void accept(T t)
Consumer<String>  log    = System.out::println;
Consumer<Produto> salvar = produtoRepository::save;
Consumer<Pedido>  enviar = emailService::enviarConfirmacao;

// BiConsumer<T, U> — recebe T e U, sem retorno
// Método abstrato: void accept(T t, U u)
BiConsumer<String, Integer> logComNivel =
        (msg, nivel) -> logger.log(nivel, msg);
BiConsumer<Map<String, Object>, String> remover =
        (mapa, chave) -> mapa.remove(chave);

// Uso típico: forEach, ifPresent, peek
produtos.forEach(salvar);                                      // Consumer
optional.ifPresent(log);                                       // Consumer
stream.peek(p -> logger.debug("Processando: {}", p.getId())); // Consumer

// ════════════════════════════════════════════════════════════════════════════════
// GRUPO 4 — Predicados (recebe algo, retorna boolean)
// ════════════════════════════════════════════════════════════════════════════════

// Predicate<T> — recebe T, retorna boolean
// Método abstrato: boolean test(T t)
Predicate<String>  naoVazio   = s -> !s.isBlank();
Predicate<Produto> emEstoque  = p -> p.getEstoque() > 0;
Predicate<Pedido>  confirmado = p -> p.getStatus() == Status.CONFIRMADO;

// BiPredicate<T, U> — recebe T e U, retorna boolean
// Método abstrato: boolean test(T t, U u)
BiPredicate<String, Integer> tamanhoMinimo =
        (s, min) -> s.length() >= min;

// ════════════════════════════════════════════════════════════════════════════════
// GRUPO 5 — Versões primitivas (evitam boxing/unboxing — mais performáticas)
// ════════════════════════════════════════════════════════════════════════════════

// IntFunction<R>, LongFunction<R>, DoubleFunction<R>
IntFunction<String>    intToStr  = Integer::toString;       // int → String
LongFunction<Instant>  epochToTs = Instant::ofEpochMilli;  // long → Instant

// ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
ToIntFunction<String>    strlen  = String::length;          // String → int
ToDoubleFunction<Produto> preco  = p -> p.getPreco().doubleValue();

// IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
IntUnaryOperator    dobrarInt  = n -> n * 2;                // int → int
DoubleUnaryOperator arredondar = Math::floor;               // double → double

// IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
IntBinaryOperator    somaInt = Integer::sum;                // (int, int) → int
DoubleBinaryOperator maiorD  = Math::max;                   // (double, double) → double

// IntSupplier, LongSupplier, DoubleSupplier
IntSupplier    contador = atomicInt::getAndIncrement;       // () → int
DoubleSupplier random   = Math::random;                     // () → double

// IntConsumer, LongConsumer, DoubleConsumer
IntConsumer    printInt  = System.out::println;             // int → void
DoubleConsumer logDouble = d -> logger.debug("val={}", d);  // double → void

// IntPredicate, LongPredicate, DoublePredicate
IntPredicate  positivo = n -> n > 0;                        // int → boolean
LongPredicate par      = n -> n % 2 == 0;                   // long → boolean
```

#### 1.5.2 Composição de funções

Todas as interfaces funcionais do `java.util.function` oferecem métodos `default`
para **encadear e combinar** funções sem variáveis intermediárias.

```java
// ─── Function: andThen e compose ─────────────────────────────────────────────

Function<String, String>  trim      = String::trim;
Function<String, String>  maiusculo = String::toUpperCase;
Function<String, Integer> tamanho   = String::length;

// andThen: aplica this, depois a função argumento
// trim → maiusculo → tamanho: "  abc  " → "ABC" → 3
Function<String, Integer> pipeline = trim.andThen(maiusculo).andThen(tamanho);
int resultado = pipeline.apply("  abc  ");  // 3

// compose: aplica a função argumento ANTES de this
// maiusculo.compose(trim) ≡ trim.andThen(maiusculo)
Function<String, String> trimThenMaiusculo = maiusculo.compose(trim);

// ─── Predicate: and, or, negate ───────────────────────────────────────────────

Predicate<Produto> ativo    = p -> p.isAtivo();
Predicate<Produto> emEstoque = p -> p.getEstoque() > 0;
Predicate<Produto> barato   = p -> p.getPreco().compareTo(new BigDecimal("100")) < 0;

// and: ambos devem ser verdadeiros
Predicate<Produto> disponivel = ativo.and(emEstoque);

// or: pelo menos um deve ser verdadeiro
Predicate<Produto> interessante = barato.or(emEstoque);

// negate: inverte o resultado
Predicate<Produto> indisponivel = disponivel.negate();

// Predicate.not (Java 11+) — inverso de um method reference
Predicate<String> naoVazio = Predicate.not(String::isBlank);

// Uso em stream com composição
List<Produto> catalogo = produtos.stream()
        .filter(ativo.and(emEstoque).and(Predicate.not(Produto::isRemovido)))
        .toList();

// ─── Consumer: andThen ────────────────────────────────────────────────────────

Consumer<Pedido> salvar  = pedidoRepository::save;
Consumer<Pedido> auditar = auditService::registrar;
Consumer<Pedido> notificar = emailService::enviarConfirmacao;

// Executa salvar → auditar → notificar em sequência
Consumer<Pedido> processarPedido = salvar.andThen(auditar).andThen(notificar);
processarPedido.accept(pedido);

// ─── BinaryOperator como acumulador em reduce ─────────────────────────────────

BinaryOperator<BigDecimal> soma  = BigDecimal::add;
BinaryOperator<String>     concat = (a, b) -> a + ", " + b;

BigDecimal totalPrecos = precos.stream()
        .reduce(BigDecimal.ZERO, soma);           // 0 + p1 + p2 + ...

String nomesCSV = nomes.stream()
        .reduce("", concat);                       // "" + n1 + ", " + n2 + ...

// Function.identity() — retorna o próprio argumento (útil em collectors)
Map<Long, Produto> porId = produtos.stream()
        .collect(Collectors.toMap(Produto::getId, Function.identity()));
```

#### 1.5.3 Method references — quatro formas

```java
// ════════════════════════════════════════════════════════════════════════════════
// FORMA 1: Classe::métodoEstático
// Equivale a: (args) -> Classe.metodoEstatico(args)
// ════════════════════════════════════════════════════════════════════════════════
Function<String, Integer>  parseInt    = Integer::parseInt;      // s -> Integer.parseInt(s)
Function<String, String>   valueOf     = String::valueOf;        // s -> String.valueOf(s)
Function<Double, Double>   sqrt        = Math::sqrt;             // d -> Math.sqrt(d)
Predicate<String>          isNullFn    = Objects::isNull;        // s -> Objects.isNull(s)
BiFunction<Object,Object,Boolean> eq   = Objects::equals;        // (a,b) -> Objects.equals(a,b)

// ════════════════════════════════════════════════════════════════════════════════
// FORMA 2: instância::método
// Equivale a: (args) -> instancia.metodo(args)
// ════════════════════════════════════════════════════════════════════════════════
Consumer<String>  printer   = System.out::println;               // s -> System.out.println(s)
Consumer<Produto> salvar    = produtoRepository::save;           // p -> produtoRepository.save(p)
Supplier<Instant> agora     = Instant::now;                      // () -> Instant.now()
// (Instant::now é estático — comporta-se como Forma 1 aqui)

// ════════════════════════════════════════════════════════════════════════════════
// FORMA 3: Classe::métodoDeInstância
// O primeiro argumento da lambda vira o receptor (this) do método
// Equivale a: (obj, args) -> obj.metodo(args)
// ════════════════════════════════════════════════════════════════════════════════
Function<String, String>      upper    = String::toUpperCase;    // s -> s.toUpperCase()
Function<String, Integer>     length   = String::length;         // s -> s.length()
Function<Produto, String>     getNome  = Produto::getNome;       // p -> p.getNome()
BiFunction<String,String,Boolean> startsWith = String::startsWith; // (s,p) -> s.startsWith(p)
Predicate<String>             vazio    = String::isEmpty;         // s -> s.isEmpty()
ToIntFunction<String>         tamanho  = String::length;         // s -> s.length() (int primitivo)

// ════════════════════════════════════════════════════════════════════════════════
// FORMA 4: Classe::new (construtor)
// Equivale a: (args) -> new Classe(args)
// ════════════════════════════════════════════════════════════════════════════════
Supplier<ArrayList<String>>         listaVazia = ArrayList::new;   // () -> new ArrayList<>()
Function<String, StringBuilder>     sbFactory  = StringBuilder::new; // s -> new StringBuilder(s)
BiFunction<String,BigDecimal,Produto> prodFactory =
        Produto::new;                                               // (n,p) -> new Produto(n,p)

// ─── Uso prático dos quatro tipos em conjunto ─────────────────────────────────
List<String> nomes = List.of("  Ana  ", "  Bob  ", "");

List<String> resultado = nomes.stream()
        .map(String::trim)             // Forma 3: instance method
        .filter(Predicate.not(String::isEmpty))  // Forma 3: instance method
        .map(String::toUpperCase)      // Forma 3: instance method
        .sorted(String::compareTo)     // Forma 3: instance method (BiFunction)
        .collect(Collectors.toList()); // ["ANA", "BOB"]
```

#### 1.5.4 Interfaces funcionais customizadas e usos no Spring MVC

```java
// ─── @FunctionalInterface — garante que a interface tem exatamente 1 método abs.
// O compilador emite erro se alguém adicionar um segundo método abstrato.

@FunctionalInterface
public interface Validator<T> {
    ValidationResult validate(T obj);

    // Composição: encadeia validadores
    default Validator<T> andThen(Validator<T> other) {
        return obj -> {
            ValidationResult r = this.validate(obj);
            return r.isValid() ? other.validate(obj) : r;
        };
    }
}

// Uso:
Validator<ProdutoRequest> nomeValido  = req -> req.nome() != null && !req.nome().isBlank()
        ? ValidationResult.ok()
        : ValidationResult.erro("Nome obrigatório");

Validator<ProdutoRequest> precoValido = req -> req.preco() != null && req.preco().signum() > 0
        ? ValidationResult.ok()
        : ValidationResult.erro("Preço deve ser positivo");

Validator<ProdutoRequest> validarProduto = nomeValido.andThen(precoValido);
ValidationResult resultado = validarProduto.validate(request);

// ─── Tipos funcionais recorrentes no Spring Framework ─────────────────────────

// HandlerFunction<T> (WebMvc.fn) — Function<ServerRequest, ServerResponse>
// RouterFunction<T>  (WebMvc.fn) — encadeia rotas como predicados

// Comparator — interface funcional com método estático e helpers
Comparator<Produto> porPreco = Comparator.comparing(Produto::getPreco);
Comparator<Produto> porNome  = Comparator.comparing(Produto::getNome);
Comparator<Produto> ordem    = porPreco.reversed()           // maior preço primeiro
                                       .thenComparing(porNome); // desempate por nome

produtos.sort(ordem);
List<Produto> ordenados = produtos.stream().sorted(ordem).toList();

// Runnable — () -> void (sem parâmetros, sem retorno)
// Muito comum com @Async e CompletableFuture
Runnable tarefa = () -> processarRelatorio(id);
CompletableFuture.runAsync(tarefa, executor);

// Callable<T> — () throws Exception -> T (com checagem de exceção)
// Usado em @Async e controllers assíncronos (seção 11.9)
Callable<RelatorioResponse> callable = () -> relatorioService.gerarCompleto(periodo);
```

#### 1.5.5 Resumo — guia de escolha rápida

| Preciso... | Interface | Assinatura |
|---|---|---|
| Transformar `T` em `R` | `Function<T,R>` | `R apply(T)` |
| Transformar `T` em `T` | `UnaryOperator<T>` | `T apply(T)` |
| Transformar `T` e `U` em `R` | `BiFunction<T,U,R>` | `R apply(T,U)` |
| Combinar dois `T` em `T` | `BinaryOperator<T>` | `T apply(T,T)` |
| Produzir `T` sem entrada | `Supplier<T>` | `T get()` |
| Consumir `T` sem retorno | `Consumer<T>` | `void accept(T)` |
| Consumir `T` e `U` sem retorno | `BiConsumer<T,U>` | `void accept(T,U)` |
| Testar `T` → boolean | `Predicate<T>` | `boolean test(T)` |
| Testar `T` e `U` → boolean | `BiPredicate<T,U>` | `boolean test(T,U)` |
| Executar sem parâmetros / retorno | `Runnable` | `void run()` |
| Produzir `T` com exceção checada | `Callable<T>` | `T call() throws Exception` |
| Transformar `int` → `R` (sem boxing) | `IntFunction<R>` | `R apply(int)` |
| Transformar `T` → `int` (sem boxing) | `ToIntFunction<T>` | `int applyAsInt(T)` |
| Operar `int` → `int` | `IntUnaryOperator` | `int applyAsInt(int)` |

---

### 1.6 Outros Recursos Java Usados no Documento

Referência rápida de recursos Java modernos que aparecem nos exemplos das seções
numeradas.

```java
// ─── var — inferência de tipo local (Java 10+) ────────────────────────────────
// O compilador infere o tipo; não reduz type-safety — é açúcar sintático.
var produtos  = produtoRepository.findAll();          // List<Produto>
var pageable  = PageRequest.of(0, 20, Sort.by("nome")); // PageRequest
var resultado = produtoService.criar(request);         // tipo inferido do retorno

// ─── Pattern matching para instanceof (Java 16+) ─────────────────────────────
// Elimina o cast manual após verificação de tipo
Object obj = obterObjeto();

// Antes:
if (obj instanceof String) {
    String s = (String) obj;  // cast redundante
    System.out.println(s.length());
}

// Agora:
if (obj instanceof String s) {              // declara 's' já como String
    System.out.println(s.length());
}

// ─── switch expression (Java 14+) ────────────────────────────────────────────
// Retorna valor diretamente; sem fall-through; cobre todos os casos
HttpStatus status = switch (tipoErro) {
    case "NOT_FOUND"   -> HttpStatus.NOT_FOUND;
    case "FORBIDDEN"   -> HttpStatus.FORBIDDEN;
    case "CONFLICT"    -> HttpStatus.CONFLICT;
    default            -> HttpStatus.INTERNAL_SERVER_ERROR;
};

// switch com bloco (yield para retornar de bloco multi-linha)
String descricao = switch (produto.getTipo()) {
    case FISICO   -> "Produto físico — requer envio";
    case DIGITAL  -> {
        var link = gerarLink(produto);
        yield "Download: " + link;           // yield retorna de dentro do bloco
    }
    case SERVICO  -> "Prestação de serviço";
};

// ─── Sealed classes (Java 17+) — hierarquias fechadas ────────────────────────
// Restringe quais classes podem implementar/estender; potencializa o switch
public sealed interface Resultado<T>
        permits Resultado.Sucesso, Resultado.Erro {

    record Sucesso<T>(T valor)              implements Resultado<T> {}
    record Erro<T>(String mensagem, int codigo) implements Resultado<T> {}
}

// switch exaustivo sobre sealed interface — o compilador garante todos os casos
String texto = switch (resultado) {
    case Resultado.Sucesso<ProdutoResponse> s -> "Criado: " + s.valor().nome();
    case Resultado.Erro<ProdutoResponse>    e -> "Erro %d: %s".formatted(e.codigo(), e.mensagem());
    // Sem default necessário — sealed garante exaustividade
};

// ─── Method references — atalho para lambdas que delegam a um método ─────────
// Classe::métodoEstático
Function<String, Integer> parse  = Integer::parseInt;
// instância::método
Consumer<String>          printer = System.out::println;
// Classe::métodoDeInstância (receptor é o primeiro parâmetro)
Function<Produto, String> nome   = Produto::getNome;
// Classe::new (construtor)
Function<Long, Produto>   ctor   = Produto::new;

// ─── Stream API — processamento declarativo de coleções ───────────────────────
List<ProdutoResponse> resultado = produtos.stream()
        .filter(p  -> p.isAtivo())                           // filtra
        .map(ProdutoResponse::from)                          // transforma
        .sorted(Comparator.comparing(ProdutoResponse::nome)) // ordena
        .limit(10)                                           // limita
        .toList();                                           // coleta

// Operações terminais úteis
long total           = stream.count();
Optional<T> primeiro = stream.findFirst();
boolean todos        = stream.allMatch(predicado);
boolean algum        = stream.anyMatch(predicado);
Map<K, List<T>> agrupado = stream.collect(Collectors.groupingBy(classificador));
```

---

## 2. Base — Spring Framework: IoC, DI e Anotações Essenciais

Esta seção apresenta os fundamentos do Spring Framework que sustentam todo o
Spring MVC. O entendimento destes conceitos é pré-requisito para as seções
numeradas do documento.

### 2.1 Inversão de Controle e Injeção de Dependência

O Spring Framework é construído sobre dois pilares:

- **IoC (Inversion of Control):** o container Spring cria e gerencia o ciclo de
  vida dos objetos em vez de o código criá-los com `new`.
- **DI (Dependency Injection):** o container injeta as dependências nos objetos
  em vez de eles as buscarem ativamente.

#### 2.1.1 Sem IoC — acoplamento manual (o problema)

```java
// ════════════════════════════════════════════════════════════════════════════════
// CENÁRIO: sistema de pedidos com repositório, serviço e controller
// SEM IoC — todas as dependências criadas e gerenciadas manualmente pelo código.
// ════════════════════════════════════════════════════════════════════════════════

// ─── Repositório ──────────────────────────────────────────────────────────────
public class PedidoRepository {

    // A conexão com o banco é criada DENTRO do repositório.
    // Ninguém de fora controla como ela é criada ou qual implementação é usada.
    private final DataSource dataSource;

    public PedidoRepository() {
        // Acoplamento forte: o repositório decide qual banco usar e como conectar.
        // Para trocar de banco ou usar um banco diferente em testes, é necessário
        // alterar o código desta classe — viola o Princípio Aberto/Fechado (OCP).
        var ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://localhost:5432/loja");
        ds.setUsername("admin");
        ds.setPassword("senha123");    // ← senha hardcoded no código
        this.dataSource = ds;
    }

    public Optional<Pedido> findById(Long id) {
        // usa dataSource...
        return Optional.empty();
    }
}

// ─── Serviço ──────────────────────────────────────────────────────────────────
public class PedidoService {

    // O serviço cria o repositório com "new" — ele está no controle da criação.
    // Problema 1: não há como substituir PedidoRepository por uma implementação
    //             de teste sem alterar esta classe.
    // Problema 2: se PedidoRepository mudar seu construtor, PedidoService
    //             também precisará ser alterado — alto acoplamento.
    private final PedidoRepository pedidoRepository = new PedidoRepository();

    // O serviço também cria o EmailService, que por sua vez pode criar
    // outras dependências — a cadeia de "new" cresce indefinidamente.
    private final EmailService emailService = new EmailService();

    public PedidoResponse buscar(Long id) {
        return pedidoRepository.findById(id)
                .map(PedidoResponse::from)
                .orElseThrow(() -> new RuntimeException("Pedido não encontrado: " + id));
    }

    public void confirmar(Long id) {
        var pedido = pedidoRepository.findById(id).orElseThrow();
        // ...lógica de confirmação...
        emailService.enviarConfirmacao(pedido);
    }
}

// ─── Controller ───────────────────────────────────────────────────────────────
public class PedidoController {

    // O controller também cria o serviço com "new".
    // Se PedidoService precisar de novos colaboradores no futuro,
    // PedidoController precisará ser atualizado — mesmo que não use esses colaboradores.
    private final PedidoService pedidoService = new PedidoService();

    public PedidoResponse buscar(Long id) {
        return pedidoService.buscar(id);
    }
}

// ─── Criação no ponto de uso ──────────────────────────────────────────────────
public class Main {
    public static void main(String[] args) {
        // Cada vez que precisar de um controller, todo o grafo de objetos é
        // recriado do zero: controller → service → repository → datasource.
        // Não há singleton: cada "new PedidoController()" cria conexões novas.
        // Não há gerenciamento de ciclo de vida: conexões nunca são fechadas.
        var controller = new PedidoController();
        controller.buscar(1L);
    }
}

// ─── Teste unitário — quase impossível ───────────────────────────────────────
class PedidoServiceTest {
    @Test
    void buscar_deveRetornarPedido() {
        // Não há como criar um PedidoService com um repositório "fake":
        //   - PedidoRepository é instanciado DENTRO de PedidoService
        //   - O teste vai tentar conectar ao banco de produção
        //   - Único recurso: subclassear e sobrescrever (Mockito estático, PowerMock)
        //     — frágil, verboso e com baixo valor didático
        var service = new PedidoService(); // ← conecta ao banco de produção!
        // ...
    }
}
```

#### 2.1.2 Com IoC e DI — o Spring assume o controle

```java
// ════════════════════════════════════════════════════════════════════════════════
// MESMO CENÁRIO — com Spring IoC e injeção de dependência por construtor.
// O container cria, configura e injeta tudo. O código só declara do que precisa.
// ════════════════════════════════════════════════════════════════════════════════

// ─── Repositório ──────────────────────────────────────────────────────────────
@Repository   // ← detectado pelo component scan; registrado como bean singleton
public class PedidoRepository {

    private final JdbcClient jdbcClient;

    // O construtor declara a dependência — o Spring a injeta.
    // PedidoRepository não sabe COMO o JdbcClient foi criado nem qual banco usa.
    // Em teste, o Spring (ou o teste) injeta um JdbcClient conectado ao H2/Testcontainers.
    public PedidoRepository(JdbcClient jdbcClient) {
        this.jdbcClient = jdbcClient;
    }

    public Optional<Pedido> findById(Long id) {
        return jdbcClient.sql("SELECT * FROM pedidos WHERE id = :id")
                .param("id", id)
                .query(Pedido.class)
                .optional();
    }
}

// ─── Serviço ──────────────────────────────────────────────────────────────────
@Service   // ← detectado pelo component scan; registrado como bean singleton
public class PedidoService {

    // Campos final: imutáveis após construção, nunca serão null.
    private final PedidoRepository pedidoRepository;
    private final EmailService     emailService;

    // O Spring injeta as implementações corretas automaticamente.
    // Para trocar EmailService por uma versão de teste, basta registrar
    // outra implementação no contexto de teste — nenhuma alteração aqui.
    public PedidoService(PedidoRepository pedidoRepository,
                         EmailService emailService) {
        this.pedidoRepository = pedidoRepository;
        this.emailService     = emailService;
    }

    public PedidoResponse buscar(Long id) {
        return pedidoRepository.findById(id)
                .map(PedidoResponse::from)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Pedido", id));
    }

    public void confirmar(Long id) {
        var pedido = pedidoRepository.findById(id).orElseThrow();
        // ...lógica de confirmação...
        emailService.enviarConfirmacao(pedido);
    }
}

// ─── Controller ───────────────────────────────────────────────────────────────
@RestController   // ← detectado pelo component scan; registrado como bean singleton
@RequestMapping("/api/v1/pedidos")
public class PedidoController {

    private final PedidoService pedidoService;

    // O Spring injeta o MESMO bean PedidoService que foi criado anteriormente —
    // singleton por padrão. Não há nova instância, não há nova conexão com banco.
    public PedidoController(PedidoService pedidoService) {
        this.pedidoService = pedidoService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<PedidoResponse> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(pedidoService.buscar(id));
    }
}

// ─── DataSource e JdbcClient declarados uma única vez — @Configuration ────────
// O Spring Boot auto-configura isso a partir do application.yml.
// Configuração manual é necessária apenas para múltiplos datasources.
@Configuration
public class DataConfig {

    // Este bean é criado UMA vez e injetado em TODOS os repositórios.
    // O pool de conexões (HikariCP) é gerenciado pelo container — fechado
    // corretamente quando o contexto é encerrado (@PreDestroy implícito).
    @Bean
    public DataSource dataSource(
            @Value("${spring.datasource.url}")      String url,
            @Value("${spring.datasource.username}") String username,
            @Value("${spring.datasource.password}") String password) {
        var ds = new HikariDataSource();
        ds.setJdbcUrl(url);       // lido do application.yml — sem hardcode
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}

// ─── Teste unitário — limpo e simples ────────────────────────────────────────
class PedidoServiceTest {

    // Mockito cria implementações "fake" das dependências em memória.
    // Nenhum banco de dados, nenhum servidor de email — teste é instantâneo.
    @Mock
    private PedidoRepository pedidoRepository;

    @Mock
    private EmailService emailService;

    private PedidoService pedidoService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        // Construtor com injeção: o teste passa as implementações fake diretamente.
        // Isso só é possível porque o serviço recebe as dependências via construtor
        // e não as cria internamente com "new".
        pedidoService = new PedidoService(pedidoRepository, emailService);
    }

    @Test
    void buscar_QuandoPedidoExiste_RetornaResponse() {
        var pedidoMock = new Pedido(1L, "CONFIRMADO");
        when(pedidoRepository.findById(1L)).thenReturn(Optional.of(pedidoMock));

        var result = pedidoService.buscar(1L);

        assertThat(result.id()).isEqualTo(1L);
        verify(pedidoRepository).findById(1L);
        verifyNoInteractions(emailService);   // email não foi enviado em busca
    }
}
```

#### 2.1.3 Comparativo direto

| Aspecto | Sem IoC (`new`) | Com IoC (Spring) |
|---|---|---|
| Criação de objetos | Cada classe cria suas dependências com `new` | Container cria e gerencia todos os objetos |
| Acoplamento | Alto — mudança em A força mudança em B | Baixo — classes dependem de interfaces |
| Ciclo de vida | Manual — sem garantia de fechamento | Gerenciado — `@PostConstruct` / `@PreDestroy` |
| Singleton | Impossível sem código extra | Automático (padrão para todos os beans) |
| Testabilidade | Difícil — dependências hardcoded | Fácil — qualquer implementação pode ser injetada |
| Configuração de ambiente | Hardcoded no código | Externalizada em `application.yml` / variáveis de ambiente |
| Substituição de implementação | Requer alteração do código | Registrar novo bean / `@Profile` / `@ConditionalOn*` |

```mermaid
flowchart LR
    subgraph Container["ApplicationContext (IoC Container)"]
        direction TB
        BF["BeanFactory<br>(criação lazy)"]
        AC["ApplicationContext<br>(criação eager + eventos + i18n)"]
        BF --> AC
    end

    subgraph Fontes["Fontes de definição de beans"]
        CF["@Configuration<br>@Bean"]
        CP["@Component<br>@Service<br>@Repository<br>@Controller"]
        XM["XML / Groovy<br>(legado)"]
    end

    Fontes --> Container
    Container -->|"injeta via construtor<br>(recomendado)"| A["@Service<br>ProdutoService"]
    Container -->|"injeta via construtor"| B["@Controller<br>ProdutoController"]
    A --> B
```

### 2.2 `@Configuration` e `proxyBeanMethods`

`@Configuration` marca uma classe como **fonte de definições de beans** para o
container. Por padrão, o Spring cria um **proxy CGLIB** da classe para garantir
que chamadas entre métodos `@Bean` dentro da mesma classe retornem sempre a
**mesma instância** (comportamento de singleton).

```java
// ─── proxyBeanMethods = true (DEFAULT) ───────────────────────────────────────
//
// O Spring cria um proxy CGLIB da classe.
// Chamadas entre métodos @Bean são interceptadas:
//   dataSource() chamado por jdbcClient() retorna o MESMO bean do container.
// Use quando um @Bean chama outro @Bean da mesma classe.
@Configuration   // equivalente a @Configuration(proxyBeanMethods = true)
public class DataConfig {

    @Bean
    public DataSource dataSource() {
        var ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        return ds;
    }

    @Bean
    public JdbcClient jdbcClient() {
        // Chama dataSource() — o proxy intercepta e retorna o bean singleton,
        // NÃO cria uma segunda instância de HikariDataSource.
        return JdbcClient.create(dataSource());
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        // Mesma chamada interceptada — mesma instância de DataSource
        return new DataSourceTransactionManager(dataSource());
    }
}

// ─── proxyBeanMethods = false ─────────────────────────────────────────────────
//
// Sem proxy CGLIB: a classe é instanciada diretamente.
// Benefícios: startup mais rápido, menor uso de memória, melhor para GraalVM native.
// Restrição: métodos @Bean NÃO podem chamar outros @Bean da mesma classe
//            (cada chamada criaria uma nova instância, quebrando o singleton).
// Use quando os beans são independentes entre si — ex.: configurações simples.
@Configuration(proxyBeanMethods = false)
public class WebConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return JsonMapper.builder()
                .findAndAddModules()
                .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .build();
    }

    @Bean
    public ModelMapper modelMapper() {
        var mm = new ModelMapper();
        mm.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);
        return mm;
    }
    // objectMapper() e modelMapper() são independentes — sem chamadas cruzadas.
    // proxyBeanMethods = false é seguro aqui e melhora a performance de startup.
}
```

**Resumo:**

| | `proxyBeanMethods = true` (default) | `proxyBeanMethods = false` |
|---|---|---|
| Proxy CGLIB gerado | ✅ Sim | ❌ Não |
| Chamadas entre `@Bean` retornam singleton | ✅ Sim | ❌ Não (nova instância a cada chamada) |
| Performance de startup | Ligeiramente mais lenta | ✅ Mais rápida |
| Compatível com GraalVM Native | ⚠️ Limitado | ✅ Preferível |
| **Use quando** | Beans dependem uns dos outros na config | Beans são declarados independentemente |

### 2.3 `@Bean`

`@Bean` declara que um método de uma classe `@Configuration` produz um bean
gerenciado pelo container. É a forma de registrar objetos de terceiros (que não
podem receber `@Component`) ou quando a criação exige lógica.

```java
@Configuration(proxyBeanMethods = false)
public class InfraConfig {

    // ─── Bean com nome explícito ──────────────────────────────────────────────
    // Por padrão o nome é o nome do método. name= sobrescreve.
    @Bean(name = "primaryDataSource")
    public DataSource dataSource(
            @Value("${spring.datasource.url}") String url,
            @Value("${spring.datasource.username}") String username,
            @Value("${spring.datasource.password}") String password) {
        var ds = new HikariDataSource();
        ds.setJdbcUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }

    // ─── Escopo do bean ───────────────────────────────────────────────────────
    // @Scope("singleton") é o padrão — um único bean compartilhado.
    // Outros escopos: prototype (nova instância por injeção), request, session.
    @Bean
    @Scope("prototype")
    public EmailBuilder emailBuilder() {
        return new EmailBuilder();  // nova instância a cada ponto de injeção
    }

    // ─── @Primary — bean preferido quando há múltiplas implementações ─────────
    @Bean
    @Primary
    public CacheManager redisCacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.builder(factory).build();
    }

    @Bean
    public CacheManager caffeineCacheManager() {
        return new CaffeineCacheManager();
    }

    // ─── @Qualifier — identifica bean específico quando há múltiplos ──────────
    // (usado na injeção, ver seção 2.5)

    // ─── Lifecycle callbacks ──────────────────────────────────────────────────
    @Bean(initMethod = "connect", destroyMethod = "disconnect")
    public ExternalApiClient apiClient() {
        return new ExternalApiClient("https://api.externa.com.br");
    }

    // Alternativa: @PostConstruct / @PreDestroy na própria classe do bean
}
```

### 2.3.1 Escopos de Beans (`@Scope`)

O escopo de um bean define **quantas instâncias** o container cria e **por quanto
tempo** cada instância vive. O Spring oferece dois escopos universais (disponíveis
em qualquer contexto) e quatro escopos web (requerem um `WebApplicationContext`).

#### Escopos universais

| Escopo | Valor em `@Scope` | Instâncias | Ciclo de vida |
|---|---|---|---|
| **Singleton** | `"singleton"` | **1** por container | Nasce com o contexto, morre com ele |
| **Prototype** | `"prototype"` | **1 nova por solicitação** | Nasce a cada `getBean()` / injeção; destruição é responsabilidade do chamador |

`singleton` é o **padrão**; não precisa ser declarado explicitamente.

```java
@Component                        // singleton implícito
public class ServicoCalculo { }

@Component
@Scope("prototype")               // nova instância a cada ponto de injeção
public class RelatorioBuilder { }
```

#### Escopos web

Disponíveis apenas quando o contexto é um `WebApplicationContext` (aplicações
Spring MVC / Spring Boot com servlet container).

| Escopo | Valor em `@Scope` | Instâncias | Ciclo de vida |
|---|---|---|---|
| **Request** | `"request"` | 1 por **requisição HTTP** | Nasce no início do request, morre ao final |
| **Session** | `"session"` | 1 por **sessão HTTP** | Nasce na criação da sessão, morre no timeout/invalidação |
| **Application** | `"application"` | 1 por **`ServletContext`** | Análogo ao singleton, mas vinculado ao `ServletContext` (partilhado por todos os `ApplicationContext` da mesma app web) |
| **WebSocket** | `"websocket"` | 1 por **sessão WebSocket** | Nasce na abertura da conexão, morre no fechamento |

#### `proxyMode` — por que é necessário nos escopos web

Quando um bean de escopo curto (ex.: `request`) é injetado em um bean de escopo
mais longo (ex.: `singleton`), surge um problema estrutural: o singleton é criado
**uma única vez** pelo container. Nesse momento, o Spring tentaria injetar a
instância `request`-scoped — mas ainda não existe nenhuma requisição HTTP em
curso. Mesmo que existisse, a referência ficaria **fixada** para sempre naquele
singleton, vazando dados entre requisições diferentes.

A solução é o **scoped proxy**: em vez de injetar o bean real, o Spring injeta
um objeto proxy com a mesma interface. O proxy sabe como localizar a instância
correta para o contexto ativo (requisição ou sessão atual) a cada chamada de
método.

```
Singleton  →  [Proxy]  →  instância real do request/session atual
                ↑
        redireciona cada chamada ao bean real do contexto ativo
```

O atributo `proxyMode` de `@Scope` controla **como** o proxy é gerado:

| Valor | Mecanismo | Quando usar |
|---|---|---|
| `ScopedProxyMode.NO` | Sem proxy (padrão) | Bean nunca é injetado em escopo mais longo |
| `ScopedProxyMode.TARGET_CLASS` | Proxy CGLIB (subclasse) | Bean é uma **classe concreta** — caso mais comum |
| `ScopedProxyMode.INTERFACES` | Proxy JDK (interface) | Bean implementa uma **interface** e você prefere proxy dinâmico |
| `ScopedProxyMode.DEFAULT` | Depende da configuração do container | Delega a decisão ao contexto |

```java
// ── Bean de escopo request com proxy CGLIB (classe concreta) ─────────────────
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ContextoRequisicao {
    private String usuarioId;
    // getters / setters
}

// ── Bean de escopo session com proxy CGLIB ────────────────────────────────────
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class CarrinhoCompras {
    private final List<Item> itens = new ArrayList<>();
    // getters / setters
}

// ── Singleton que injeta os dois beans de escopo mais curto ───────────────────
@Service
public class PedidoService {

    // O Spring injeta proxies aqui — nunca o bean real diretamente.
    // Cada chamada a carrinho.getItens() ou contexto.getUsuarioId()
    // é redirecionada pelo proxy à instância real da requisição/sessão ativa.
    private final CarrinhoCompras carrinho;
    private final ContextoRequisicao contexto;

    public PedidoService(CarrinhoCompras carrinho, ContextoRequisicao contexto) {
        this.carrinho = carrinho;
        this.contexto = contexto;
    }

    public Pedido fecharPedido() {
        // proxy resolve carrinho e contexto para a requisição HTTP atual
        return new Pedido(contexto.getUsuarioId(), carrinho.getItens());
    }
}
```

> **Regra prática:** use sempre `TARGET_CLASS` para beans de escopo `request`,
> `session` ou `websocket` que precisem ser injetados em singletons. `INTERFACES`
> só vale a pena quando o bean já implementa uma interface e você quer evitar a
> dependência do CGLIB (rara no Spring Boot, que inclui CGLIB por padrão).
> Consulte a seção 2.3.3 para o detalhamento completo do problema e das soluções.

#### Escopos do Spring Batch (dependência separada)

| Escopo | Ativação | Ciclo de vida |
|---|---|---|
| `@StepScope` | `spring-batch-core` | 1 instância por execução de **step** |
| `@JobScope` | `spring-batch-core` | 1 instância por execução de **job** |

Esses escopos permitem que parâmetros de job/step sejam resolvidos via SpEL em
tempo de execução: `@Value("#{jobParameters['arquivo']}")`.

#### Escopos customizados

É possível registrar escopos próprios via
`ConfigurableBeanFactory.registerScope(name, scope)` e usá-los normalmente com
`@Scope("meuEscopo")`. Exemplos de terceiros: `refreshScope` (Spring Cloud
Config), `tenantScope` (soluções multi-tenant).

#### Resumo — quando usar cada escopo

| Necessidade | Escopo recomendado |
|---|---|
| Serviços sem estado compartilhado (regra geral) | `singleton` |
| Objeto com estado mutável que não pode ser compartilhado | `prototype` |
| Dados específicos de uma requisição (locale, usuário autenticado) | `request` |
| Carrinho de compras, preferências do usuário na sessão | `session` |
| Cache ou configuração global de uma app web | `application` |
| Estado de uma conexão WebSocket específica | `websocket` |

---

### 2.3.2 Injetando um Bean Prototype dentro de um Singleton

Por padrão, todos os beans Spring são **singleton**: o container cria uma única
instância e a reutiliza. Um bean **prototype** gera uma nova instância a cada
vez que é solicitado ao container. O problema surge quando você injeta um
prototype dentro de um singleton via construtor: a injeção ocorre **uma única
vez** na criação do singleton, e o campo nunca muda — o prototype passa a se
comportar como singleton, anulando o objetivo do escopo.

```java
// ════════════════════════════════════════════════════════════════════════════════
// PROBLEMA: injeção direta via construtor faz o prototype virar singleton
// ════════════════════════════════════════════════════════════════════════════════

// Bean prototype — cada solicitação deveria entregar uma nova instância
@Component
@Scope("prototype")
public class RelatorioBuilder {

    private final List<String> linhas = new ArrayList<>();

    public void adicionar(String linha) { linhas.add(linha); }
    public String gerar() { return String.join("\n", linhas); }
}

// Bean singleton — criado UMA vez pelo container
@Service
public class RelatorioService {

    // ❌ ERRADO: esta instância é injetada UMA única vez na criação do singleton.
    //    Todas as chamadas a gerarRelatorio() compartilham o MESMO RelatorioBuilder,
    //    acumulando linhas de relatórios anteriores — bug de estado compartilhado.
    private final RelatorioBuilder builder;

    public RelatorioService(RelatorioBuilder builder) {
        this.builder = builder;   // ← injetado uma vez, nunca substituído
    }

    public String gerarRelatorio(List<String> dados) {
        dados.forEach(builder::adicionar);
        return builder.gerar();   // linhas de chamadas anteriores vazam aqui
    }
}
```

#### Solução 1 — `ObjectProvider<T>` (recomendada)

`ObjectProvider<T>` é a forma idiomática moderna do Spring para obter beans
prototype sob demanda. É injetado como qualquer outro bean e respeita toda a
infraestrutura do container (proxies, perfis, qualificadores).

```java
// ════════════════════════════════════════════════════════════════════════════════
// SOLUÇÃO 1: ObjectProvider<T> — forma recomendada (Spring 4.3+)
// ════════════════════════════════════════════════════════════════════════════════

@Service
public class RelatorioService {

    // ObjectProvider é injetado como singleton, mas cada chamada a getObject()
    // pede uma NOVA instância de RelatorioBuilder ao container.
    private final ObjectProvider<RelatorioBuilder> builderProvider;

    public RelatorioService(ObjectProvider<RelatorioBuilder> builderProvider) {
        this.builderProvider = builderProvider;
    }

    public String gerarRelatorio(List<String> dados) {
        // ✅ Nova instância a cada chamada — sem estado compartilhado entre relatórios
        RelatorioBuilder builder = builderProvider.getObject();
        dados.forEach(builder::adicionar);
        return builder.gerar();
    }
}
```

#### Solução 2 — `@Lookup` (injeção de método)

`@Lookup` instrui o Spring a sobrescrever o método anotado em uma subclasse
CGLIB, fazendo-o retornar sempre uma nova instância do prototype. É útil quando
a classe não pode depender de tipos Spring (ex.: frameworks ou legado).

```java
// ════════════════════════════════════════════════════════════════════════════════
// SOLUÇÃO 2: @Lookup — método sobrescrito por proxy CGLIB
// ════════════════════════════════════════════════════════════════════════════════

@Service
public abstract class RelatorioService {   // ← precisa ser abstrata ou não-final

    public String gerarRelatorio(List<String> dados) {
        // novoBuilder() é interceptado pelo CGLIB e retorna uma nova instância
        RelatorioBuilder builder = novoBuilder();
        dados.forEach(builder::adicionar);
        return builder.gerar();
    }

    // O Spring sobrescreve este método; o corpo pode ficar vazio ou lançar exceção.
    // O nome do bean pode ser especificado em @Lookup("beanName") se necessário.
    @Lookup
    protected abstract RelatorioBuilder novoBuilder();
}
```

#### Solução 3 — `ApplicationContext` (acesso direto ao container)

Funciona, mas aumenta o acoplamento ao Spring e dificulta testes. Prefira
`ObjectProvider` nas situações onde as soluções anteriores são viáveis.

```java
// ════════════════════════════════════════════════════════════════════════════════
// SOLUÇÃO 3: ApplicationContext — funciona, mas acopla ao Spring
// ════════════════════════════════════════════════════════════════════════════════

@Service
public class RelatorioService implements ApplicationContextAware {

    private ApplicationContext ctx;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.ctx = ctx;
    }

    public String gerarRelatorio(List<String> dados) {
        // getBean() sempre retorna nova instância para beans prototype
        RelatorioBuilder builder = ctx.getBean(RelatorioBuilder.class);
        dados.forEach(builder::adicionar);
        return builder.gerar();
    }
}
```

**Comparativo:**

| Abordagem | Acoplamento ao Spring | Testabilidade | Quando usar |
|---|---|---|---|
| `ObjectProvider<T>` | Baixo (API Spring) | ✅ Fácil (mock/stub do provider) | Caso geral — forma recomendada |
| `@Lookup` | Nenhum na interface pública | ✅ Boa (pode sobrescrever o método) | Quando a classe não deve referenciar tipos Spring |
| `ApplicationContext` | Alto | ⚠️ Mais trabalhoso (mock do contexto) | Código legado ou utilitários genéricos |

> **Regra prática:** se um bean precisa ser **prototype**, cada singleton que o
> utiliza deve solicitar uma **nova instância por chamada**, nunca armazenar o
> prototype em um campo. Use `ObjectProvider` como padrão.

### 2.3.3 Injetando Beans de Escopo `request` ou `session` em Beans de Escopo Mais Amplo

Os escopos web do Spring criam instâncias vinculadas ao ciclo de vida de uma
requisição HTTP (`request`) ou de uma sessão de usuário (`session`). O escopo
`application` é equivalente a um singleton no contexto do `ServletContext`.

```
Hierarquia de escopos (do mais curto ao mais longo):
  request  <  session  <  application ≈ singleton
```

O problema é análogo ao do prototype: um bean de escopo mais amplo (ex.:
`singleton`) é criado uma única vez e guarda uma referência fixa. Se o bean
injetado é de escopo mais curto (ex.: `session`), cada usuário deveria ter sua
própria instância — mas o singleton não troca a referência automaticamente.

```java
// ════════════════════════════════════════════════════════════════════════════════
// PROBLEMA: injeção direta de bean session em singleton
// ════════════════════════════════════════════════════════════════════════════════

@Component
@Scope("session")
public class CarrinhoCompras {
    private final List<Item> itens = new ArrayList<>();
    public void adicionar(Item item) { itens.add(item); }
    public List<Item> getItens() { return Collections.unmodifiableList(itens); }
}

@Service
public class PedidoService {

    // ❌ ERRADO: o Spring injeta o CarrinhoCompras do PRIMEIRO usuário
    //    que ativou este bean. Todos os usuários subsequentes veem o
    //    carrinho desse primeiro usuário — vazamento grave de dados.
    private final CarrinhoCompras carrinho;

    public PedidoService(CarrinhoCompras carrinho) {
        this.carrinho = carrinho;   // fixado na criação do singleton
    }
}
```

#### Solução 1 — Scoped Proxy (recomendada para escopos web)

O Spring cria um **proxy** que tem a mesma interface do bean alvo. O singleton
recebe o proxy (que é "eterno"), e cada acesso ao proxy é redirecionado à
instância real do escopo correto (request/session do contexto atual).

```java
// ════════════════════════════════════════════════════════════════════════════════
// SOLUÇÃO 1: ScopedProxyMode — proxy que delega ao bean do escopo correto
// ════════════════════════════════════════════════════════════════════════════════

// ─── Bean de escopo session com proxy ─────────────────────────────────────────
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
// TARGET_CLASS → proxy CGLIB (para classes concretas)
// INTERFACES   → proxy JDK  (para interfaces — requer que o bean implemente uma)
public class CarrinhoCompras {

    private final List<Item> itens = new ArrayList<>();

    public void adicionar(Item item) { itens.add(item); }
    public List<Item> getItens() { return Collections.unmodifiableList(itens); }
    public void limpar() { itens.clear(); }
}

// ─── Bean request com proxy ───────────────────────────────────────────────────
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ContextoRequisicao {

    private String usuarioLogado;
    private String ipOrigem;

    // getters e setters omitidos
}

// ─── Singleton recebe o proxy — injeção via construtor funciona normalmente ───
@Service
public class PedidoService {

    // ✅ CORRETO: carrinho é um proxy. Cada chamada a carrinho.getItens()
    //    é redirecionada ao CarrinhoCompras da sessão HTTP ativa no momento.
    //    Usuários diferentes veem carrinhos diferentes — sem vazamento.
    private final CarrinhoCompras carrinho;
    private final ContextoRequisicao contexto;

    public PedidoService(CarrinhoCompras carrinho,
                         ContextoRequisicao contexto) {
        this.carrinho = carrinho;   // proxy session-scoped
        this.contexto = contexto;   // proxy request-scoped
    }

    public PedidoResponse fecharPedido() {
        // Em cada chamada, o proxy resolve o bean real da sessão/request corretas
        List<Item> itens = carrinho.getItens();
        String usuario   = contexto.getUsuarioLogado();
        // ...lógica de negócio...
        carrinho.limpar();
        return new PedidoResponse(usuario, itens);
    }
}
```

#### Declarando via `@Bean` (em `@Configuration`)

```java
@Configuration
public class WebBeansConfig {

    // ─── Session-scoped via @Bean ──────────────────────────────────────────────
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION,
           proxyMode = ScopedProxyMode.TARGET_CLASS)
    public CarrinhoCompras carrinhoCompras() {
        return new CarrinhoCompras();
    }

    // ─── Request-scoped via @Bean ─────────────────────────────────────────────
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST,
           proxyMode = ScopedProxyMode.TARGET_CLASS)
    public ContextoRequisicao contextoRequisicao() {
        return new ContextoRequisicao();
    }

    // ─── Application-scoped (equivale a singleton no ServletContext) ──────────
    // Pode ser injetado diretamente em qualquer bean — sem proxy necessário,
    // pois dura tanto quanto o singleton (ou mais).
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_APPLICATION)
    public ConfiguracoesGlobais configuracoesGlobais() {
        return new ConfiguracoesGlobais();
    }
}
```

#### Solução 2 — `ObjectProvider<T>` (alternativa sem proxy)

Quando não é possível ou desejável adicionar o proxy ao bean de destino (ex.:
bean de biblioteca de terceiros), `ObjectProvider` resolve a instância correta
no momento da chamada, sem necessidade de proxy na definição do bean.

```java
// ════════════════════════════════════════════════════════════════════════════════
// SOLUÇÃO 2: ObjectProvider — resolução tardia sem proxy na definição do bean
// ════════════════════════════════════════════════════════════════════════════════

// Bean session SEM proxyMode — definição simples
@Component
@Scope("session")
public class PreferenciasUsuario {
    private Locale idioma = Locale.of("pt", "BR");
    // getters e setters
}

@Service
public class NotificacaoService {

    private final ObjectProvider<PreferenciasUsuario> prefsProvider;

    public NotificacaoService(ObjectProvider<PreferenciasUsuario> prefsProvider) {
        this.prefsProvider = prefsProvider;
    }

    public void enviar(String mensagem) {
        // getObject() resolve a instância da sessão ativa no momento da chamada
        PreferenciasUsuario prefs = prefsProvider.getObject();
        String mensagemLocalizada = traduzir(mensagem, prefs.getIdioma());
        // ...
    }
}
```

**Comparativo:**

| Abordagem | Configuração no bean alvo | Transparência para o consumidor | Quando usar |
|---|---|---|---|
| `ScopedProxyMode.TARGET_CLASS` | `proxyMode` na definição | ✅ Total — injeção normal via construtor | Caso geral; bean é uma classe concreta |
| `ScopedProxyMode.INTERFACES` | `proxyMode` na definição | ✅ Total — injeção normal via construtor | Bean implementa interface; proxy JDK suficiente |
| `ObjectProvider<T>` | Nenhuma mudança no bean | ⚠️ Requer chamada a `getObject()` | Bean de terceiros sem `proxyMode` |

> **Regra de ouro dos escopos:** você pode injetar um bean de escopo **mais
> longo** em um de escopo **mais curto** diretamente (ex.: singleton em
> request). O inverso — escopo **mais curto** em **mais longo** — exige sempre
> um **proxy** ou `ObjectProvider`, pois o bean receptor sobrevive ao ciclo de
> vida do bean injetado.

### 2.4 `@Component` e Especializações (Stereotype Annotations)

`@Component` é a anotação base de **detecção automática**: o Spring varre o
classpath por componentes anotados e os registra como beans. As demais são
especializações semânticas — todas se comportam como `@Component`, mas comunicam
a intenção e habilitam comportamentos adicionais.

```mermaid
graph TD
    C["@Component<br>Bean genérico detectado<br>pelo component scan"]
    C --> CTR["@Controller<br>Web MVC: processa requests<br>retorna view name ou ModelAndView"]
    CTR --> RCT["@RestController<br>= @Controller + @ResponseBody<br>REST: retorna objeto serializado<br>como JSON/XML"]
    C --> SVC["@Service<br>Camada de negócio<br>Habilita @Transactional por AOP"]
    C --> REP["@Repository<br>Camada de persistência<br>Traduz exceções para DataAccessException<br>Habilita @Transactional por AOP"]
    C --> CFG["@Configuration<br>Fonte de beans @Bean<br>(+ proxy CGLIB por padrão)"]
```

```java
// ─── @Component — uso genérico ────────────────────────────────────────────────
// Para utilitários, helpers, e componentes que não se encaixam nas outras camadas
@Component
public class CpfFormatter {
    public String formatar(String cpf) { /* ... */ return cpf; }
    public String desformatar(String cpf) { /* ... */ return cpf; }
}

// ─── @Service — camada de negócio ────────────────────────────────────────────
// Semanticamente indica "lógica de aplicação/domínio".
// @Transactional na classe ou método funciona aqui via AOP proxy.
@Service
@Transactional(readOnly = true)   // padrão de leitura; sobrescrito nos métodos de escrita
public class ProdutoService {

    private final ProdutoRepository produtoRepository;

    public ProdutoService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    public Optional<ProdutoResponse> buscarPorId(Long id) {
        return produtoRepository.findById(id).map(ProdutoResponse::from);
    }

    @Transactional   // escrita: sobrescreve o readOnly=true da classe
    public ProdutoResponse criar(ProdutoRequest request) {
        var produto = Produto.from(request);
        return ProdutoResponse.from(produtoRepository.save(produto));
    }
}

// ─── @Repository — camada de dados ───────────────────────────────────────────
// Habilita tradução automática de exceções de infraestrutura
// (SQLException, JPA exceptions) para a hierarquia DataAccessException do Spring.
// Permite que a camada de serviço trate exceções sem acoplamento ao JPA/JDBC.
@Repository
public class ProdutoJdbcRepository {

    private final JdbcClient jdbcClient;

    public ProdutoJdbcRepository(JdbcClient jdbcClient) {
        this.jdbcClient = jdbcClient;
    }

    public Optional<Produto> findById(Long id) {
        return jdbcClient.sql("SELECT * FROM produtos WHERE id = :id")
                .param("id", id)
                .query(Produto.class)
                .optional();
    }
}
// Nota: interfaces que estendem JpaRepository / CrudRepository já são detectadas
// como @Repository pelo Spring Data — não é necessário declarar a anotação.

// ─── @Controller e @RestController — camada web ───────────────────────────────
// Ver seções 3 e 4 para exemplos completos.
// A diferença fundamental:
//   @Controller       → retorna nome de view (String) ou ModelAndView (SSR)
//   @RestController   → retorna objeto serializado para o corpo HTTP (REST)
```

### 2.5 `@Autowired` — Injeção de Dependência e a Boa Prática Atual

`@Autowired` instrui o container a injetar um bean no ponto anotado. Pode ser
aplicada em **campos**, **setters** ou **construtores**.

```java
// ─── EVITAR: injeção por campo (@Autowired no campo) ─────────────────────────
//
// ❌ Problemas:
//   - Impossibilita testes unitários sem o container (não dá para usar "new")
//   - Oculta as dependências — a assinatura da classe não as declara
//   - Viola o princípio da imutabilidade: o campo não pode ser "final"
//   - Reflation acoplada ao Spring (não funciona fora do contexto Spring)
@Service
public class ProdutoServiceRuim {
    @Autowired                          // ← EVITAR
    private ProdutoRepository repository;

    @Autowired                          // ← EVITAR
    private CategoriaService categoriaService;
}

// ─── EVITAR: injeção por setter ───────────────────────────────────────────────
//
// ❌ Permite que dependências obrigatórias sejam omitidas (NullPointerException)
// ✅ Aceitável apenas para dependências OPCIONAIS (@Autowired(required = false))
@Service
public class ProdutoServiceSetter {
    private ProdutoRepository repository;

    @Autowired                          // ← EVITAR para dependências obrigatórias
    public void setRepository(ProdutoRepository repository) {
        this.repository = repository;
    }
}

// ─── RECOMENDADO: injeção por construtor, sem @Autowired ─────────────────────
//
// ✅ Desde o Spring 4.3: quando há UM único construtor, @Autowired é opcional.
//    O Spring injeta automaticamente — a anotação é redundante.
// ✅ Vantagens:
//   - Campos podem ser "final" — imutabilidade garantida
//   - Dependências explícitas na assinatura do construtor
//   - Classe pode ser instanciada com "new" nos testes (sem container)
//   - Código mais limpo e autoexplicativo
@Service
public class ProdutoService {

    private final ProdutoRepository   produtoRepository;   // final!
    private final CategoriaService    categoriaService;    // final!
    private final ApplicationEventPublisher eventPublisher; // final!

    // @Autowired DESNECESSÁRIO — Spring injeta automaticamente (único construtor)
    public ProdutoService(ProdutoRepository produtoRepository,
                          CategoriaService categoriaService,
                          ApplicationEventPublisher eventPublisher) {
        this.produtoRepository = produtoRepository;
        this.categoriaService  = categoriaService;
        this.eventPublisher    = eventPublisher;
    }
}

// ─── @Qualifier — quando há múltiplas implementações do mesmo tipo ────────────
@Service
public class NotificacaoService {

    private final CacheManager cacheManager;

    // @Qualifier seleciona pelo nome do bean quando @Primary não é suficiente
    public NotificacaoService(
            @Qualifier("caffeineCacheManager") CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
}

// ─── @Autowired(required = false) — dependência opcional ─────────────────────
// Único caso em que setter injection é aceitável
@Component
public class MetricasCollector {

    private MeterRegistry meterRegistry;  // pode ser null se Micrometer não estiver no classpath

    @Autowired(required = false)
    public void setMeterRegistry(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void registrar(String evento) {
        if (meterRegistry != null) {
            meterRegistry.counter(evento).increment();
        }
    }
}

// ─── Dependência opcional com injeção por construtor ─────────────────────────
//
// Com construtor, há três formas de declarar uma dependência opcional,
// sem precisar recorrer a @Autowired(required = false) em setter.

// Forma 1: Optional<T> — mais explícita e idiomática (Java 8+)
// O container injeta Optional.empty() se o bean não estiver registrado.
// Vantagem: a assinatura do construtor comunica claramente a opcionalidade.
@Service
public class NotificacaoService {

    private final EmailService         emailService;
    private final Optional<SmsService> smsService;   // ausente → Optional.empty()

    public NotificacaoService(EmailService emailService,
                               Optional<SmsService> smsService) {
        this.emailService = emailService;
        this.smsService   = smsService;
    }

    public void notificar(String destinatario, String mensagem) {
        emailService.enviar(destinatario, mensagem);
        smsService.ifPresent(sms -> sms.enviar(destinatario, mensagem));
    }
}

// Forma 2: @Nullable — null-safe sem wrapper
// O container injeta null se o bean não estiver registrado.
// Vantagem: sem overhead do Optional, idiomático com JSpecify (projeto já usa @NullMarked).
// Atenção: requer null-check explícito no uso — certifique-se de tratar o null.
@Service
public class RelatorioService {

    private final ProdutoRepository      produtoRepository;
    private final @Nullable AuditService auditService;   // null se não registrado

    public RelatorioService(ProdutoRepository produtoRepository,
                             @Nullable AuditService auditService) {
        this.produtoRepository = produtoRepository;
        this.auditService      = auditService;
    }

    public RelatorioResponse gerar(Long id) {
        var relatorio = produtoRepository.findById(id).orElseThrow();
        if (auditService != null) {
            auditService.registrar("relatorio.gerado", id);
        }
        return RelatorioResponse.from(relatorio);
    }
}

// Forma 3: ObjectProvider<T> — mais poderoso; resolve lazy, múltiplos beans e protótipos
//
// ObjectProvider é a forma mais flexível:
//   getIfAvailable()  — retorna null (sem exceção) se o bean não existir
//   ifAvailable(cb)   — executa o callback apenas se o bean existir
//   stream()          — itera sobre TODAS as implementações registradas
//   getObject(args)   — cria nova instância (útil para beans @Prototype)
//
// Vantagem extra: o bean alvo só é resolvido/criado quando getIfAvailable()
// ou ifAvailable() é chamado — comportamento lazy implícito.
@Service
public class ExportacaoService {

    private final ProdutoRepository          produtoRepository;
    private final ObjectProvider<PdfExporter> pdfExporterProvider;

    public ExportacaoService(ProdutoRepository produtoRepository,
                              ObjectProvider<PdfExporter> pdfExporterProvider) {
        this.produtoRepository   = produtoRepository;
        this.pdfExporterProvider = pdfExporterProvider;
    }

    public byte[] exportarPdf(Long id) {
        var produto = produtoRepository.findById(id).orElseThrow();

        // Lança NoSuchBeanDefinitionException se PdfExporter não estiver registrado
        // — use quando o bean é obrigatório mas de resolução lazy
        // return pdfExporterProvider.getObject().exportar(produto);

        // Retorna null sem exceção se PdfExporter não estiver disponível
        PdfExporter exporter = pdfExporterProvider.getIfAvailable();
        if (exporter == null) {
            throw new NegocioException("Exportação PDF não está habilitada neste ambiente");
        }
        return exporter.exportar(produto);
    }

    public void notificarTodosExportadores(Long id) {
        var produto = produtoRepository.findById(id).orElseThrow();
        // stream() itera sobre TODOS os beans do tipo Exporter registrados
        pdfExporterProvider.stream().forEach(e -> e.exportar(produto));
    }
}

// ─── Comparativo: quando usar cada forma de dependência opcional ──────────────
//
// | Abordagem              | Quando preferir                                      |
// |------------------------|------------------------------------------------------|
// | Optional<T>            | Forma mais legível — comunicação explícita na API    |
// | @Nullable              | Sem overhead; projeto já usa JSpecify/@NullMarked    |
// | ObjectProvider<T>      | Lazy, múltiplas implementações, beans @Prototype     |
// | @Autowired(req=false)  | Apenas para setter — nunca no construtor             |

// ─── @Lazy — inicialização tardia do bean ─────────────────────────────────────
//
// Por padrão o Spring inicializa todos os beans singleton na subida do contexto
// (eager initialization). @Lazy adia a criação do bean para o momento em que
// ele é acessado pela primeira vez.
//
// ONDE pode ser aplicado:
//   - Na declaração do bean (@Component, @Service, @Bean): define o bean como lazy.
//   - No ponto de injeção (construtor, setter): injeta um proxy que inicializa
//     o bean real somente na primeira chamada ao proxy — mesmo que o bean alvo
//     não seja lazy por definição.
//
// QUANDO usar:
//   ✅ Beans pesados raramente usados (ex.: cliente de integração, gerador de PDF)
//   ✅ Acelerar o startup em desenvolvimento (todos os beans lazy via property)
//   ✅ Quebrar dependências circulares (solução paliativa — prefira refatorar)
//   ❌ Beans críticos ao funcionamento da aplicação — erros de configuração
//      só aparecem na primeira chamada, não no startup
//   ❌ Beans que precisam de validação de configuração no startup

// Caso 1: bean marcado como lazy em sua própria declaração
@Service
@Lazy   // o Spring não cria esta instância na subida — apenas quando for injetada/acessada
public class RelatorioExcelService {

    // Construtor pesado: carrega templates, inicializa Apache POI, etc.
    public RelatorioExcelService() {
        // simulação de inicialização cara
    }

    public byte[] gerar(Long relatorioId) {
        // gera o Excel...
        return new byte[0];
    }
}

// Caso 2: @Lazy no ponto de injeção — injeta um proxy CGLIB do bean alvo
// O bean real só é criado na primeira chamada a qualquer método do proxy.
// Útil quando o bean A depende de B mas raramente usa B em runtime.
@Service
public class PedidoService {

    private final PedidoRepository      pedidoRepository;
    private final RelatorioExcelService relatorioService;  // raramente usado

    public PedidoService(
            PedidoRepository pedidoRepository,
            @Lazy RelatorioExcelService relatorioService) {
            // ↑ proxy injetado na construção; RelatorioExcelService só é instanciado
            //   quando relatorioService.gerar(...) for chamado pela primeira vez
        this.pedidoRepository = pedidoRepository;
        this.relatorioService = relatorioService;
    }

    public PedidoResponse buscar(Long id) {
        // relatorioService.gerar() não é chamado aqui — o bean real ainda não existe
        return pedidoRepository.findById(id).map(PedidoResponse::from).orElseThrow();
    }

    public byte[] exportarRelatorio(Long pedidoId) {
        // Primeira chamada a relatorioService: o proxy inicializa o bean real agora
        return relatorioService.gerar(pedidoId);
    }
}

// Caso 3: lazy global via application.yml — útil em desenvolvimento
// Reduz o tempo de startup consideravelmente em projetos grandes.
// ⚠️  Não recomendado em produção: erros de configuração só aparecem em runtime.
//
// spring:
//   main:
//     lazy-initialization: true  # torna TODOS os beans lazy por padrão
//
// Para forçar beans críticos como eager mesmo com lazy global:
@Service
@Lazy(false)   // eager mesmo quando spring.main.lazy-initialization=true
public class SecurityAuditService {
    // bean crítico que deve falhar no startup se mal configurado
}
```

### 2.6 Component Scan — Como o Spring Detecta os Beans

```java
// A anotação @SpringBootApplication combina três anotações:
//   @SpringBootConfiguration  → @Configuration especializado
//   @EnableAutoConfiguration  → ativa os starters / auto-config
//   @ComponentScan            → varre o pacote da classe e subpacotes

@SpringBootApplication   // escaneia br.com.empresa.app e TODOS os subpacotes
public class MeuAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(MeuAppApplication.class, args);
    }
}

// ─── Para escanear pacotes adicionais ou customizar o scan ────────────────────
@SpringBootApplication(
    scanBasePackages = {
        "br.com.empresa.app",
        "br.com.empresa.shared"   // pacotes externos ao projeto principal
    }
)
public class MeuAppApplication { /* ... */ }

// ─── @ComponentScan em @Configuration separado ────────────────────────────────
@Configuration
@ComponentScan(
    basePackages = "br.com.empresa.plugins",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = TestComponent.class   // exclui beans de teste
    )
)
public class PluginConfig { }
```

### 2.7 Boas Práticas Resumidas

```java
// ✅ DO — Injeção por construtor sem @Autowired
@Service
public class PedidoService {
    private final PedidoRepository repository;
    private final EstoqueService    estoqueService;

    public PedidoService(PedidoRepository repository, EstoqueService estoqueService) {
        this.repository      = repository;
        this.estoqueService  = estoqueService;
    }
}

// ✅ DO — @Configuration(proxyBeanMethods = false) quando beans não se chamam
@Configuration(proxyBeanMethods = false)
public class AppConfig {
    @Bean public ObjectMapper objectMapper() { /* ... */ }
    @Bean public ModelMapper modelMapper()   { /* ... */ }
}

// ✅ DO — @Configuration (proxy ativo) quando beans dependem uns dos outros
@Configuration
public class DataConfig {
    @Bean DataSource dataSource() { /* ... */ }
    @Bean JdbcClient jdbcClient() { return JdbcClient.create(dataSource()); }
}

// ✅ DO — @Qualifier para desambiguar múltiplas implementações
@Service
public class RelatorioService {
    public RelatorioService(@Qualifier("pdfExporter") Exporter exporter) { /* ... */ }
}

// ❌ DON'T — @Autowired em campo
@Service
public class Ruim {
    @Autowired  // ← não faça
    private Repositorio rep;
}

// ❌ DON'T — @Autowired(required = true) explícito em construtor (redundante)
@Service
public class Redundante {
    @Autowired  // ← desnecessário com Spring 4.3+ e único construtor
    public Redundante(Servico servico) { /* ... */ }
}
```

---

### 2.8 `ApplicationContext` — Acesso Programático ao Container

O `ApplicationContext` é a interface central do IoC container. Em situações onde
a injeção de dependência declarativa não é possível (factories estáticos, código
legado, integrações dinâmicas), é possível acessar beans programaticamente.

> **Regra de ouro:** prefira sempre a injeção por construtor (seção 2.5). Acesso
> programático ao `ApplicationContext` é um escape hatch — use com parcimônia e
> documente o motivo.

#### 2.8.1 Injetando o próprio `ApplicationContext`

```java
// O ApplicationContext é um bean como qualquer outro — pode ser injetado.
// Use quando precisar de acesso dinâmico ao container dentro de um bean Spring.
@Component
public class BeanLocator {

    private final ApplicationContext context;

    public BeanLocator(ApplicationContext context) {
        this.context = context;
    }

    // ─── Buscar bean pelo tipo ─────────────────────────────────────────────────
    public <T> T getBean(Class<T> tipo) {
        // Lança NoSuchBeanDefinitionException se não houver bean do tipo
        // Lança NoUniqueBeanDefinitionException se houver mais de um
        return context.getBean(tipo);
    }

    // ─── Buscar bean pelo nome e tipo (evita ambiguidade) ─────────────────────
    public <T> T getBean(String nome, Class<T> tipo) {
        return context.getBean(nome, tipo);
    }

    // ─── Verificar se um bean existe antes de buscá-lo ────────────────────────
    public boolean contemBean(String nome) {
        return context.containsBean(nome);
    }

    // ─── Listar todos os beans de um tipo (ex: todos os EventListener) ─────────
    public <T> Map<String, T> getBeansDoTipo(Class<T> tipo) {
        return context.getBeansOfType(tipo);
    }

    // ─── Verificar se o bean é singleton ou prototype ─────────────────────────
    public boolean eSingleton(String nome) {
        return context.isSingleton(nome);
    }
}
```

#### 2.8.2 `ApplicationContextAware` — receber o contexto via callback

Útil quando o bean não pode usar injeção por construtor (ex.: classes instanciadas
por frameworks externos como JPA, Jackson ou em testes de integração legados).

```java
/**
 * Implementar ApplicationContextAware faz o Spring chamar setApplicationContext()
 * logo após construir e configurar o bean, antes de qualquer @PostConstruct.
 *
 * Prefira injeção por construtor quando possível — esta abordagem é um fallback
 * para situações onde o bean é criado fora do controle direto do container.
 */
@Component
public class SpringContextHolder implements ApplicationContextAware {

    // Static para que o contexto fique acessível sem referência ao bean
    // (ex.: chamado de código estático legado)
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        SpringContextHolder.applicationContext = ctx;
    }

    // ─── Ponto de acesso estático — use apenas quando injeção não for viável ───
    public static ApplicationContext getContext() {
        if (applicationContext == null) {
            throw new IllegalStateException(
                "ApplicationContext não foi inicializado. " +
                "Certifique-se de que SpringContextHolder é um bean Spring.");
        }
        return applicationContext;
    }

    public static <T> T getBean(Class<T> tipo) {
        return getContext().getBean(tipo);
    }

    public static <T> T getBean(String nome, Class<T> tipo) {
        return getContext().getBean(nome, tipo);
    }
}

// Uso em código legado ou estático que não pode receber injeção:
public class UtilitarioLegado {

    public static void processarPedido(Long id) {
        // Apenas quando não há outra alternativa — documente o motivo
        var service = SpringContextHolder.getBean(PedidoService.class);
        service.confirmar(id);
    }
}
```

#### 2.8.3 Acesso programático a beans com nome dinâmico

```java
/**
 * Caso de uso: estratégia dinâmica onde o nome do bean é determinado em runtime
 * (ex.: Strategy Pattern com seleção por enum ou configuração).
 */
@Service
public class ExportacaoDispatcher {

    private final ApplicationContext context;

    public ExportacaoDispatcher(ApplicationContext context) {
        this.context = context;
    }

    /**
     * Seleciona o bean exportador conforme o formato solicitado.
     * Os beans são nomeados por convenção: "pdfExporter", "csvExporter", etc.
     *
     * Alternativa mais type-safe: Map<String, Exporter> injetado por construtor
     * (o Spring popula o Map com todos os beans do tipo Exporter automaticamente).
     */
    public byte[] exportar(Long relatorioId, String formato) {
        String nomeDoBean = formato.toLowerCase() + "Exporter"; // "pdfExporter"

        if (!context.containsBean(nomeDoBean)) {
            throw new NegocioException(
                    "Formato de exportação não suportado: " + formato);
        }

        Exporter exporter = context.getBean(nomeDoBean, Exporter.class);
        return exporter.exportar(relatorioId);
    }
}

// ─── Alternativa preferível: injeção de Map<String, Exporter> ─────────────────
// O Spring injeta automaticamente todos os beans do tipo Exporter,
// usando o nome do bean como chave — sem acesso direto ao ApplicationContext.
@Service
public class ExportacaoDispatcherV2 {

    // Spring popula o map: {"pdfExporter" → PdfExporter, "csvExporter" → CsvExporter}
    private final Map<String, Exporter> exporters;

    public ExportacaoDispatcherV2(Map<String, Exporter> exporters) {
        this.exporters = exporters;
    }

    public byte[] exportar(Long relatorioId, String formato) {
        Exporter exporter = exporters.get(formato.toLowerCase() + "Exporter");
        if (exporter == null) {
            throw new NegocioException("Formato não suportado: " + formato);
        }
        return exporter.exportar(relatorioId);
    }
}
```

#### 2.8.4 Publicação e escuta de eventos via `ApplicationContext`

O `ApplicationContext` também é um **event bus** interno — permite comunicação
desacoplada entre beans sem dependência direta.

```java
// ─── Definição do evento ──────────────────────────────────────────────────────
// Recomendado: Record (imutável, sem herança de ApplicationEvent necessária — Spring 4.2+)
public record PedidoConfirmadoEvent(Long pedidoId, String clienteEmail, Instant ocorridoEm) {}

// ─── Publicação do evento no Service ─────────────────────────────────────────
@Service
public class PedidoService {

    private final PedidoRepository         pedidoRepository;
    private final ApplicationEventPublisher eventPublisher;   // interface mais estreita que ApplicationContext

    public PedidoService(PedidoRepository pedidoRepository,
                          ApplicationEventPublisher eventPublisher) {
        this.pedidoRepository = pedidoRepository;
        this.eventPublisher   = eventPublisher;
    }

    @Transactional
    public void confirmar(Long id) {
        var pedido = pedidoRepository.findById(id).orElseThrow();
        pedido.confirmar();
        pedidoRepository.save(pedido);

        // Publica o evento APÓS o commit — listeners são chamados de forma síncrona
        // por padrão; use @TransactionalEventListener para garantir pós-commit.
        eventPublisher.publishEvent(
            new PedidoConfirmadoEvent(pedido.getId(), pedido.getClienteEmail(), Instant.now())
        );
    }
}

// ─── Listener do evento ───────────────────────────────────────────────────────
@Component
public class PedidoConfirmadoListener {

    private final EmailService emailService;

    public PedidoConfirmadoListener(EmailService emailService) {
        this.emailService = emailService;
    }

    // @TransactionalEventListener: só executa após o commit da transação que publicou o evento.
    // Evita enviar e-mail se a transação do Service for revertida (rollback).
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onPedidoConfirmado(PedidoConfirmadoEvent evento) {
        emailService.enviarConfirmacao(evento.clienteEmail(), evento.pedidoId());
    }

    // @Async: processa em thread separada para não bloquear a transação principal
    @EventListener
    @Async
    public void onPedidoConfirmadoAsync(PedidoConfirmadoEvent evento) {
        // notificações secundárias, analytics, etc.
    }
}
```

> **`ApplicationContext` vs `ApplicationEventPublisher`:** sempre injete a interface
> mais estreita. Se o bean só precisa publicar eventos, declare
> `ApplicationEventPublisher` — não `ApplicationContext`. Isso deixa a intenção
> clara e facilita os testes (mock mais simples).

---

## 3. Boas Práticas na Camada de Serviço (`@Service`)

A camada de serviço é o coração da aplicação: concentra as regras de negócio,
coordena operações entre repositórios e garante a consistência dos dados. Esta
seção consolida os padrões e armadilhas mais importantes do `@Service` no
ecossistema Spring Boot.

### 3.1 Responsabilidades e Estrutura

```java
// ─── Responsabilidades de um @Service bem definido ────────────────────────────
//
// ✅ Deve fazer:
//   - Implementar regras e invariantes de negócio
//   - Coordenar repositórios e outros serviços
//   - Converter entidades JPA ↔ DTOs (ou delegar a um Mapper)
//   - Publicar eventos de domínio via ApplicationEventPublisher
//   - Gerenciar transações (@Transactional)
//
// ❌ Não deve fazer:
//   - Conhecer HttpServletRequest, HttpSession ou qualquer detalhe HTTP
//   - Conter lógica de apresentação (formatação, i18n, redirecionamentos)
//   - Lançar diretamente respostas HTTP (ResponseEntity, ModelAndView)
//   - Armazenar estado de requisição em campos (services são singletons)

// ─── Interface de serviço: quando vale a pena ─────────────────────────────────
//
// Crie uma interface quando:
//   - Há (ou pode haver) múltiplas implementações (ex.: NotificacaoService
//     com EmailNotificacaoService e SmsNotificacaoService)
//   - O serviço precisa ser mockado em muitos testes (a interface facilita)
//   - Segue arquitetura hexagonal (ports & adapters)
//
// Não crie interface quando:
//   - Existe apenas uma implementação e ela nunca mudará
//     → IServicoXxx + ServicoXxxImpl é over-engineering na maioria dos projetos
//   - O serviço é interno / de suporte (ex.: CacheWarmupService)

public interface NotificacaoService {
    void enviar(String destinatario, String mensagem, TipoNotificacao tipo);
}

@Service
@Primary
public class EmailNotificacaoService implements NotificacaoService {
    @Override
    public void enviar(String destinatario, String mensagem, TipoNotificacao tipo) {
        // implementação via e-mail
    }
}

@Service
@Profile("sms-enabled")
public class SmsNotificacaoService implements NotificacaoService {
    @Override
    public void enviar(String destinatario, String mensagem, TipoNotificacao tipo) {
        // implementação via SMS
    }
}
```

### 3.2 `@Transactional` — Padrões e Armadilhas

#### 10.2.1 Padrão `readOnly` + escrita explícita

```java
@Service
// readOnly=true na classe: otimiza leituras (sem dirty checking, hint para o banco)
// e serve como documentação — qualquer método de escrita precisa sobrescrever.
@Transactional(readOnly = true)
public class ProdutoService {

    private final ProdutoRepository produtoRepository;
    private final CategoriaService  categoriaService;
    private final ApplicationEventPublisher eventPublisher;

    public ProdutoService(ProdutoRepository produtoRepository,
                          CategoriaService categoriaService,
                          ApplicationEventPublisher eventPublisher) {
        this.produtoRepository = produtoRepository;
        this.categoriaService  = categoriaService;
        this.eventPublisher    = eventPublisher;
    }

    // ─── Leitura — herda readOnly=true da classe ──────────────────────────────
    public ProdutoResponse buscarPorId(Long id) {
        return produtoRepository.findById(id)
                .map(ProdutoResponse::from)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));
    }

    public Page<ProdutoResponse> listar(ProdutoFiltros filtros, Pageable pageable) {
        return produtoRepository.findAll(filtros.toSpec(), pageable)
                .map(ProdutoResponse::from);
    }

    // ─── Escrita — sobrescreve com readOnly=false (default) ───────────────────
    @Transactional
    public ProdutoResponse criar(ProdutoRequest request) {
        categoriaService.verificarExistencia(request.categoriaId()); // valida antes

        var produto = Produto.from(request);
        produto = produtoRepository.save(produto);

        // Publica evento APÓS salvar, mas ANTES do commit — @TransactionalEventListener
        // garante que o listener só executa após o commit ter sucesso
        eventPublisher.publishEvent(new ProdutoCriadoEvent(produto.getId()));

        return ProdutoResponse.from(produto);
    }

    @Transactional
    public ProdutoResponse atualizar(Long id, ProdutoRequest request) {
        var produto = produtoRepository.findById(id)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));

        produto.atualizar(request); // lógica no próprio domínio
        // Não é necessário chamar save() — dirty checking do JPA detecta a mudança
        // e persiste automaticamente no commit da transação

        return ProdutoResponse.from(produto);
    }

    @Transactional
    public void remover(Long id) {
        var produto = produtoRepository.findById(id)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));

        if (produto.temPedidosAtivos()) {
            throw new NegocioException("Produto com pedidos ativos não pode ser removido");
        }
        produtoRepository.delete(produto);
    }
}
```

#### 10.2.2 Propagação — quando usar cada modo

```java
@Service
@Transactional(readOnly = true)
public class PedidoService {

    // ─── REQUIRED (default) — participa da transação existente ou cria nova ────
    // 99% dos casos: o comportamento correto para operações de negócio.
    @Transactional
    public void confirmar(Long id) {
        // executa dentro da mesma transação se chamado de outro @Transactional
    }

    // ─── REQUIRES_NEW — sempre cria uma nova transação independente ───────────
    // Use quando a operação DEVE persistir mesmo se a transação chamadora fizer
    // rollback. Exemplo clássico: log de auditoria ou registro de tentativa.
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarTentativaDePagamento(Long pedidoId, String motivo) {
        // Esta gravação persiste MESMO que o pagamento falhe e a tx principal
        // seja revertida — é o comportamento desejado para auditoria
        auditRepository.save(new AuditEntry(pedidoId, motivo, Instant.now()));
    }

    // ─── SUPPORTS — executa com transação se houver, sem transação se não ──────
    // Útil para operações de leitura que podem ser chamadas tanto dentro
    // quanto fora de um contexto transacional.
    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
    public Optional<PedidoResponse> buscarOpcional(Long id) {
        return pedidoRepository.findById(id).map(PedidoResponse::from);
    }

    // ─── NOT_SUPPORTED — suspende a transação existente ──────────────────────
    // Raramente necessário. Use quando a operação tem efeitos colaterais que
    // não devem ser revertidos com a transação (ex.: enviar mensagem a broker).
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void publicarEvento(DomainEvent evento) {
        // executado sem transação — evita o broker ser enrolado no rollback
        messageBroker.publish(evento);
    }
}
```

#### 10.2.3 `rollbackFor` — checked exceptions não fazem rollback por padrão

```java
@Service
@Transactional(readOnly = true)
public class ImportacaoService {

    // ⚠️ ARMADILHA: por padrão @Transactional só faz rollback em
    // RuntimeException (unchecked). Se o método lançar uma checked exception,
    // a transação é COMMITADA mesmo com erro!

    // ❌ Errado: IOException é checked — a transação vai commitar parcialmente
    @Transactional
    public void importarArquivo(MultipartFile arquivo) throws IOException {
        var linha1 = processar(arquivo.getInputStream());  // salva no banco
        throw new IOException("Erro na linha 2");          // tx COMMITA mesmo assim!
    }

    // ✅ Correto: rollbackFor inclui checked exceptions explicitamente
    @Transactional(rollbackFor = Exception.class)
    public void importarArquivoCorreto(MultipartFile arquivo) throws IOException {
        var linha1 = processar(arquivo.getInputStream());  // salva no banco
        throw new IOException("Erro na linha 2");          // tx faz ROLLBACK agora
    }

    // ✅ Alternativa: envolver em RuntimeException (unchecked)
    @Transactional
    public void importarArquivoAlternativo(MultipartFile arquivo) {
        try {
            processar(arquivo.getInputStream());
        } catch (IOException e) {
            throw new NegocioException("Falha na importação: " + e.getMessage(), e);
            // NegocioException extends RuntimeException → rollback automático
        }
    }

    // noRollbackFor: rollback NÃO ocorre para esta exceção específica
    // Use quando a exceção é esperada e não indica falha transacional
    @Transactional(noRollbackFor = RegistroDuplicadoException.class)
    public void importarComIdempotencia(List<Produto> produtos) {
        for (var p : produtos) {
            try {
                produtoRepository.save(p);
            } catch (RegistroDuplicadoException e) {
                // Produto já existe — ignora e continua (sem rollback)
                log.warn("Produto {} já cadastrado, pulando", p.getSku());
            }
        }
    }
}
```

#### 10.2.4 Self-invocation — a armadilha silenciosa

```java
@Service
@Transactional(readOnly = true)
public class RelatorioService {

    // ⚠️ SELF-INVOCATION: quando um método do bean chama outro método do MESMO
    // bean, a chamada bypassa o proxy AOP do Spring. O @Transactional do método
    // chamado é completamente IGNORADO — nenhuma nova transação é criada.

    // ❌ Errado: gerarRelatorio() chama gerarPDF() diretamente
    @Transactional
    public RelatorioResponse gerarRelatorio(Long id) {
        var dados = buscarDados(id);
        // Esta chamada é this.gerarPDF(dados) — bypass do proxy!
        // O @Transactional(propagation = REQUIRES_NEW) abaixo NÃO é ativado
        var pdf = gerarPDF(dados);  // ← proxy bypassado!
        return new RelatorioResponse(dados, pdf);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public byte[] gerarPDF(DadosRelatorio dados) {
        // Este @Transactional só funciona se chamado de FORA do bean
        return pdfService.gerar(dados);
    }

    // ✅ Solução 1: extrair gerarPDF para um serviço separado
    // PdfService é um @Component diferente — o proxy funciona normalmente

    // ✅ Solução 2: injetar o próprio bean (auto-injeção)
    // Spring injeta o proxy, não o objeto direto
    private RelatorioService self;  // inicializado abaixo

    @Autowired
    public void setSelf(RelatorioService self) { this.self = self; }

    @Transactional
    public RelatorioResponse gerarRelatorioCorreto(Long id) {
        var dados = buscarDados(id);
        // Chama via proxy — @Transactional(REQUIRES_NEW) será respeitado
        var pdf = self.gerarPDF(dados);  // ← via proxy
        return new RelatorioResponse(dados, pdf);
    }

    // ✅ Solução 3 (preferível): ApplicationContext para resolver o proxy
    // — ver seção 2.8 para detalhes
}
```

### 3.3 Mapeamento DTO ↔ Entidade

```java
// ─── Opção 1: método factory estático no Record (simples, sem dependência) ────
//
// Vantagem: zero overhead, zero dependência, código próximo do tipo.
// Limitação: lógica de mapeamento fica no DTO — viola SRP se ficar complexo.
public record ProdutoResponse(Long id, String nome, BigDecimal preco, String categoria) {

    public static ProdutoResponse from(Produto produto) {
        return new ProdutoResponse(
                produto.getId(),
                produto.getNome(),
                produto.getPreco(),
                produto.getCategoria().getNome()
        );
    }
}

// ─── Opção 2: Mapper dedicado — para mapeamentos complexos ────────────────────
//
// Usar quando: múltiplos campos calculados, listas aninhadas, lógica condicional.
// MapStruct gera a implementação em tempo de compilação — zero overhead em runtime.
@Mapper(componentModel = "spring")  // gera ProdutoMapperImpl como @Component
public interface ProdutoMapper {

    @Mapping(source = "categoria.nome", target = "categoria")
    ProdutoResponse toResponse(Produto produto);

    @Mapping(target = "id",         ignore = true)
    @Mapping(target = "criadoEm",   ignore = true)
    @Mapping(target = "categoria",  ignore = true)   // definido no service
    Produto toEntity(ProdutoRequest request);

    List<ProdutoResponse> toResponseList(List<Produto> produtos);
}

// ─── Onde fazer a conversão: no service (recomendado) ─────────────────────────
//
// ✅ Service converte entidade → DTO antes de retornar ao controller
// ✅ Controller nunca recebe nem retorna entidades JPA diretamente
// ❌ Não exponha entidades JPA ao controller — lazy loading fora da transação
//    gera LazyInitializationException (OSIV desabilitado, como recomendado)
@Service
@Transactional(readOnly = true)
public class ProdutoService {

    private final ProdutoRepository produtoRepository;
    private final ProdutoMapper     produtoMapper;

    public List<ProdutoResponse> listar() {
        // Entidades são convertidas para DTO DENTRO da transação
        return produtoMapper.toResponseList(produtoRepository.findAll());
        // Após o retorno, a transação fecha — sem risco de lazy loading externo
    }
}
```

### 3.4 Validação no Service — Invariantes de Domínio

```java
// O @Valid no controller valida o formato do input (Bean Validation).
// O service deve validar as REGRAS DE NEGÓCIO que o Bean Validation não cobre.

@Service
@Validated   // habilita Bean Validation nos parâmetros do service (via AOP)
@Transactional(readOnly = true)
public class PedidoService {

    // ─── @Validated no service: invariantes de domínio via Bean Validation ────
    // Útil para garantir contratos que valem independentemente de quem chama
    // (controller, job agendado, outro service, teste)
    @Transactional
    public PedidoResponse criar(@Valid @NotNull PedidoRequest request) {
        // Se request violar constraints, ConstraintViolationException é lançada
        // pelo proxy AOP antes de entrar neste método
        return processarPedido(request);
    }

    // ─── Regras de negócio explícitas (não cobertas pelo Bean Validation) ──────
    @Transactional
    public void confirmarPagamento(Long pedidoId, PagamentoRequest pagamento) {
        var pedido = pedidoRepository.findById(pedidoId)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Pedido", pedidoId));

        // Regra 1: status
        if (pedido.getStatus() != StatusPedido.AGUARDANDO_PAGAMENTO) {
            throw new NegocioException(
                    "Pedido #" + pedidoId + " não está aguardando pagamento. " +
                    "Status atual: " + pedido.getStatus());
        }

        // Regra 2: prazo
        if (pedido.getCriadoEm().isBefore(Instant.now().minus(24, ChronoUnit.HOURS))) {
            pedido.cancelar(MotivoCancelamento.PRAZO_EXPIRADO);
            throw new NegocioException("Prazo para pagamento expirou. Pedido cancelado.");
        }

        // Regra 3: valor
        if (pagamento.valor().compareTo(pedido.getTotal()) != 0) {
            throw new NegocioException(
                    "Valor do pagamento (R$ " + pagamento.valor() + ") " +
                    "diverge do total do pedido (R$ " + pedido.getTotal() + ")");
        }

        pedido.confirmarPagamento(pagamento);
    }
}
```

### 3.5 Tratamento de Exceções no Service

```java
// ─── Hierarquia de exceções de domínio ────────────────────────────────────────
//
// Crie exceções próprias que estendem RuntimeException — serão tratadas
// centralizadamente no @ControllerAdvice (seção 11).
// Não capture e relance desnecessariamente.

public abstract class AppException extends RuntimeException {
    protected AppException(String message)                   { super(message); }
    protected AppException(String message, Throwable cause)  { super(message, cause); }
}

@ResponseStatus(HttpStatus.NOT_FOUND)
public class RecursoNaoEncontradoException extends AppException {
    public RecursoNaoEncontradoException(String recurso, Object id) {
        super(recurso + " com id '" + id + "' não encontrado");
    }
}

@ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
public class NegocioException extends AppException {
    public NegocioException(String message) { super(message); }
}

@ResponseStatus(HttpStatus.CONFLICT)
public class ConflitoDeDadosException extends AppException {
    public ConflitoDeDadosException(String message) { super(message); }
}

// ─── Boas práticas de exceção no service ──────────────────────────────────────
@Service
@Transactional(readOnly = true)
public class ClienteService {

    // ✅ Lança exceção de domínio específica — o @ControllerAdvice trata
    public ClienteResponse buscarPorCpf(String cpf) {
        return clienteRepository.findByCpf(cpf)
                .map(ClienteResponse::from)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", cpf));
    }

    // ✅ Wrapping de exceção de infraestrutura em exceção de domínio
    @Transactional
    public ClienteResponse criar(ClienteRequest request) {
        if (clienteRepository.existsByCpf(request.cpf())) {
            throw new ConflitoDeDadosException(
                    "CPF " + request.cpf() + " já cadastrado");
        }
        try {
            var cliente = clienteRepository.save(Cliente.from(request));
            return ClienteResponse.from(cliente);
        } catch (DataIntegrityViolationException e) {
            // Converte exceção de infraestrutura (JPA) em exceção de domínio
            throw new ConflitoDeDadosException("Dados duplicados: " + e.getMessage());
        }
    }

    // ❌ Anti-pattern: capturar e relanças sem agregar valor
    public ClienteResponse buscarRuim(Long id) {
        try {
            return clienteRepository.findById(id)
                    .map(ClienteResponse::from)
                    .orElseThrow();
        } catch (Exception e) {
            throw new RuntimeException(e);  // ← não faça: perde o tipo, perde o contexto
        }
    }

    // ❌ Anti-pattern: logar E relançar (o @ControllerAdvice vai logar de novo)
    public void removerRuim(Long id) {
        try {
            clienteRepository.deleteById(id);
        } catch (Exception e) {
            log.error("Erro ao remover cliente", e);  // ← vai ser logado duas vezes
            throw e;
        }
    }
}
```

### 3.6 Services Stateless — Evitar Estado em Campos

```java
// @Service é um singleton — a MESMA instância atende TODAS as requisições.
// Armazenar estado de requisição em campos gera condições de corrida.

// ❌ Errado: campo de instância armazena estado da requisição
@Service
public class RelatorioServiceRuim {
    private Long usuarioAtualId;   // ← PERIGO: compartilhado entre threads!
    private String filtroAtual;    // ← PERIGO: condição de corrida!

    public void setContexto(Long userId, String filtro) {
        this.usuarioAtualId = userId;   // Thread A seta userId=1
        this.filtroAtual    = filtro;   // Thread B seta userId=2 antes de usar
    }

    public RelatorioResponse gerar() {
        // usuarioAtualId pode ser de outra thread!
        return relatorioRepository.buscar(usuarioAtualId, filtroAtual);
    }
}

// ✅ Correto: estado passado como parâmetro — stateless por design
@Service
@Transactional(readOnly = true)
public class RelatorioService {

    public RelatorioResponse gerar(Long usuarioId, String filtro) {
        // Cada chamada tem seu próprio escopo — sem estado compartilhado
        return relatorioRepository.buscar(usuarioId, filtro);
    }
}

// ✅ Alternativa para contexto de requisição: @RequestScope bean (seção 12.6)
// O Spring gerencia um bean por requisição — sem campos no Service
@Component
@RequestScope
public class RequestContext {
    private Long usuarioId;
    private String tenantId;
    // getters/setters
}
```

### 3.7 Checklist — Boas Práticas do `@Service`

| Prática | Justificativa |
|---|---|
| `@Transactional(readOnly = true)` na classe | Otimiza leituras; força declaração explícita de métodos de escrita |
| `@Transactional` sem `readOnly` nos métodos de escrita | Garante persistência; dirty checking ativo |
| `rollbackFor = Exception.class` quando há checked exceptions | Evita commit parcial silencioso |
| Retornar DTOs, nunca entidades JPA | Evita `LazyInitializationException` com `open-in-view: false` |
| Mapeamento DTO↔entidade dentro da transação | Lazy loading seguro dentro da transação aberta |
| Sem estado em campos — parâmetros para tudo | Singleton thread-safe |
| Exceções de domínio (`extends RuntimeException`) | Tratamento centralizado no `@ControllerAdvice` |
| Não logar E relançar | Evita log duplicado; deixa o `@ControllerAdvice` logar |
| Evitar self-invocation para métodos `@Transactional` | Proxy bypassa `@Transactional` em chamadas internas |
| `@Validated` para invariantes de domínio | Contratos válidos independente do chamador |

---

## 4. Classes Utilitárias do Spring

O Spring Framework disponibiliza um conjunto de classes utilitárias no pacote
`org.springframework.util` (e subpacotes) que cobrem necessidades comuns de
desenvolvimento — validação de parâmetros, manipulação de strings, coleções,
reflexão, I/O e medição de desempenho. Como essas classes fazem parte do
`spring-core` (dependência transitiva de todo projeto Spring Boot), estão sempre
disponíveis sem dependências adicionais.

> **Quando usar:** prefira as classes abaixo em vez de reimplementar lógica
> equivalente com Apache Commons, Guava ou código manual — menos dependências,
> menos código, comportamento consistente com o ecossistema Spring.

---

### 4.1 `StringUtils` — Validação e Manipulação de Strings

`org.springframework.util.StringUtils` oferece operações null-safe para strings,
mais robustas que `String.isEmpty()` pois tratam espaços em branco corretamente.

```java
import org.springframework.util.StringUtils;

// ─── Verificação de conteúdo ──────────────────────────────────────────────────
StringUtils.hasText(null)    // false — null
StringUtils.hasText("")      // false — string vazia
StringUtils.hasText("  ")    // false — só espaços em branco
StringUtils.hasText("João")  // true

StringUtils.hasLength(null)  // false — null
StringUtils.hasLength("")    // false — string vazia
StringUtils.hasLength(" ")   // true — tem comprimento (não verifica whitespace)

// ─── Manipulação ──────────────────────────────────────────────────────────────
StringUtils.trimAllWhitespace("  Spring  Boot  ") // "SpringBoot"
StringUtils.capitalize("spring")                  // "Spring"
StringUtils.uncapitalize("Spring")               // "spring"
StringUtils.delete("Hello World", "World")        // "Hello "
StringUtils.replace("foo bar", "bar", "baz")      // "foo baz"

// ─── Arquivos e caminhos ──────────────────────────────────────────────────────
StringUtils.getFilename("src/main/resources/app.yml")     // "app.yml"
StringUtils.getFilenameExtension("app.yml")               // "yml"
StringUtils.stripFilenameExtension("app.yml")             // "app"
StringUtils.cleanPath("src/../resources/./app.yml")       // "resources/app.yml"

// ─── Divisão e junção ─────────────────────────────────────────────────────────
// tokenizeToStringArray: faz trim e ignora tokens vazios automaticamente
String[] partes = StringUtils.tokenizeToStringArray("a, b, ,c", ",", true, true);
// ["a", "b", "c"]

String csv = StringUtils.arrayToCommaDelimitedString(new String[]{"a", "b", "c"});
// "a,b,c"

// ─── Uso em validação de serviço ──────────────────────────────────────────────
@Service
public class ClienteService {
    public ClienteResponse criar(ClienteRequest request) {
        if (!StringUtils.hasText(request.nome())) {
            throw new NegocioException("Nome é obrigatório");
        }
        // ...
    }
}
```

---

### 4.2 `Assert` — Pré-condições e Invariantes

`org.springframework.util.Assert` valida argumentos e invariantes de domínio.
Lança `IllegalArgumentException` (para parâmetros inválidos) ou
`IllegalStateException` (para estado interno inconsistente) ao falhar.

```java
import org.springframework.util.Assert;

// ─── Validação de argumentos ──────────────────────────────────────────────────
Assert.notNull(pedido, "Pedido não pode ser nulo");
Assert.hasText(nome, "Nome é obrigatório");
Assert.isTrue(valor > 0, "Valor deve ser positivo");
Assert.notEmpty(itens, "Pedido deve ter ao menos um item");
Assert.isNull(id, "ID deve ser nulo para criação");

// ─── Com mensagem via Supplier (lazy — evita concatenação desnecessária) ──────
Assert.notNull(pedido, () -> "Pedido " + pedidoId + " não encontrado");

// ─── Estado interno — lança IllegalStateException ────────────────────────────
Assert.state(conexao.isOpen(), "Conexão deve estar aberta");

// ─── Uso típico no topo de um método de serviço ───────────────────────────────
@Service
public class PedidoService {

    @Transactional
    public PedidoResponse criar(PedidoRequest request) {
        Assert.notNull(request, "Request não pode ser nulo");
        Assert.notEmpty(request.itens(), "Pedido deve ter ao menos um item");
        Assert.isTrue(
            request.itens().stream().allMatch(i -> i.quantidade() > 0),
            "Quantidade de todos os itens deve ser positiva"
        );
        // ...
    }
}
```

> **`Assert` vs Bean Validation:** use Bean Validation (`@NotNull`, `@Size`) para
> validar entrada de usuário na camada de controle. Use `Assert` para proteger
> invariantes dentro do domínio — pré-condições que nunca deveriam ser violadas
> por código correto.

---

### 4.3 `CollectionUtils` — Operações Null-Safe em Coleções

`org.springframework.util.CollectionUtils` oferece verificações e transformações
de coleções que funcionam corretamente com `null`, sem lançar `NullPointerException`.

```java
import org.springframework.util.CollectionUtils;

// ─── Verificação null-safe ────────────────────────────────────────────────────
CollectionUtils.isEmpty(null)           // true
CollectionUtils.isEmpty(List.of())      // true
CollectionUtils.isEmpty(List.of("a"))   // false

// ─── Acesso a elementos ───────────────────────────────────────────────────────
CollectionUtils.firstElement(List.of("a", "b", "c"))   // "a"
CollectionUtils.lastElement(List.of("a", "b", "c"))    // "c"
CollectionUtils.firstElement(Collections.emptyList())  // null (sem exceção)

// ─── Verificação de intersecção ───────────────────────────────────────────────
List<String> permissoes  = List.of("ADMIN", "USER");
List<String> necessarias = List.of("ADMIN");
CollectionUtils.containsAny(permissoes, necessarias)   // true

// ─── Busca de primeiro elemento comum ─────────────────────────────────────────
String match = CollectionUtils.findFirstMatch(permissoes, necessarias); // "ADMIN"

// ─── Conversão array → coleção existente ─────────────────────────────────────
List<String> destino = new ArrayList<>();
CollectionUtils.mergeArrayIntoCollection(new String[]{"x", "y"}, destino);

// ─── Uso em serviço ───────────────────────────────────────────────────────────
public List<ProdutoResponse> buscarPorIds(List<Long> ids) {
    if (CollectionUtils.isEmpty(ids)) return List.of();
    return produtoRepository.findAllById(ids)
            .stream().map(ProdutoResponse::from).toList();
}
```

---

### 4.4 `ObjectUtils` — Operações Null-Safe em Objetos

`org.springframework.util.ObjectUtils` oferece métodos null-safe para operações
que normalmente lançariam `NullPointerException`, incluindo suporte a arrays.

```java
import org.springframework.util.ObjectUtils;

// ─── Verificação de vazio (funciona para null, strings, arrays e coleções) ─────
ObjectUtils.isEmpty(null)              // true
ObjectUtils.isEmpty("")                // true
ObjectUtils.isEmpty(new int[]{})       // true
ObjectUtils.isEmpty(List.of())         // true
ObjectUtils.isEmpty("texto")           // false
ObjectUtils.isEmpty(new int[]{1, 2})  // false

// ─── Comparação null-safe ─────────────────────────────────────────────────────
ObjectUtils.nullSafeEquals(null, null) // true
ObjectUtils.nullSafeEquals("a", null)  // false
ObjectUtils.nullSafeEquals(null, "a")  // false
ObjectUtils.nullSafeEquals("a", "a")   // true

// ─── toString null-safe ───────────────────────────────────────────────────────
ObjectUtils.nullSafeToString(null)            // "null" (String, não NullPointerException)
ObjectUtils.nullSafeToString(new int[]{1, 2}) // "{1, 2}"

// ─── Verificação de tipo ──────────────────────────────────────────────────────
ObjectUtils.isArray(new String[]{"a"})        // true
ObjectUtils.isArray(List.of("a"))             // false

// ─── Identificar se é um tipo simples (primitivo, String, Number, Date…) ──────
ObjectUtils.isSimpleValueType(String.class)   // true
ObjectUtils.isSimpleValueType(Long.class)     // true
ObjectUtils.isSimpleValueType(Produto.class)  // false
```

---

### 4.5 `ClassUtils` — Introspecção de Classes

`org.springframework.util.ClassUtils` facilita operações dinâmicas de carregamento
e introspecção de classes, úteis em código genérico e configurações condicionais.

```java
import org.springframework.util.ClassUtils;

// ─── Verificar se uma classe está disponível no classpath ─────────────────────
if (ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", null)) {
    // Jackson disponível — configurar serialização JSON
}

// ─── Carregar classe pelo nome ────────────────────────────────────────────────
Class<?> clazz = ClassUtils.forName("com.example.Produto", null);

// ─── Obter nome curto (sem pacote) ────────────────────────────────────────────
ClassUtils.getShortName(ProdutoService.class)          // "ProdutoService"
ClassUtils.getShortName("com.example.ProdutoService")  // "ProdutoService"

// ─── Identificar tipos ────────────────────────────────────────────────────────
ClassUtils.isPrimitiveOrWrapper(int.class)      // true
ClassUtils.isPrimitiveOrWrapper(Integer.class)  // true
ClassUtils.isPrimitiveOrWrapper(String.class)   // false

ClassUtils.isPrimitiveArray(int[].class)                        // true
ClassUtils.isAssignable(Number.class, Integer.class)            // true

// ─── ClassLoader padrão da aplicação ─────────────────────────────────────────
ClassLoader loader = ClassUtils.getDefaultClassLoader();
```

---

### 4.6 `BeanUtils` — Cópia de Propriedades entre Objetos

`org.springframework.beans.BeanUtils` copia propriedades entre objetos por
reflexão, usando convenção JavaBeans (getters/setters). Útil para mapeamento
simples DTO ↔ entidade quando MapStruct seria excessivo.

```java
import org.springframework.beans.BeanUtils;

// ─── Cópia simples — copia propriedades com mesmo nome e tipo ─────────────────
ProdutoRequest request = new ProdutoRequest("Caneta", new BigDecimal("2.50"));
Produto produto = new Produto();
BeanUtils.copyProperties(request, produto);
// produto.nome = "Caneta", produto.preco = 2.50

// ─── Cópia ignorando campos específicos ───────────────────────────────────────
// Útil em PATCH: preserva campos não enviados pelo cliente
BeanUtils.copyProperties(request, produto, "id", "criadoEm");

// ─── Instanciar classe com construtor padrão ──────────────────────────────────
Produto novoProduto = BeanUtils.instantiateClass(Produto.class);

// ─── Obter PropertyDescriptors (para iterar propriedades) ────────────────────
PropertyDescriptor[] props = BeanUtils.getPropertyDescriptors(Produto.class);
for (PropertyDescriptor pd : props) {
    System.out.println(pd.getName() + " : " + pd.getPropertyType().getSimpleName());
}
```

> **Limitações de `BeanUtils.copyProperties`:**
> - Copia apenas propriedades com nome **e tipo** idênticos entre origem e destino.
> - Não realiza deep copy de objetos aninhados.
> - Para mapeamentos complexos (tipos diferentes, transformações, aninhamentos),
>   prefira **MapStruct** (seção 3.3) — type-safe e sem overhead de reflexão em runtime.

---

### 4.7 `ReflectionUtils` — Reflexão Sem Checked Exceptions

`org.springframework.util.ReflectionUtils` simplifica a API de reflexão do Java,
eliminando o tratamento verboso de checked exceptions (`NoSuchFieldException`,
`IllegalAccessException`, etc.).

```java
import org.springframework.util.ReflectionUtils;

// ─── Acesso a campos (incluindo privados) ─────────────────────────────────────
Field campo = ReflectionUtils.findField(Produto.class, "nome");
ReflectionUtils.makeAccessible(campo);                     // torna acessível
Object valor = ReflectionUtils.getField(campo, produto);   // lê o valor
ReflectionUtils.setField(campo, produto, "Novo Nome");     // escreve o valor

// ─── Invocação de métodos ─────────────────────────────────────────────────────
Method metodo = ReflectionUtils.findMethod(
    Produto.class, "calcularDesconto", BigDecimal.class);
ReflectionUtils.makeAccessible(metodo);
Object resultado = ReflectionUtils.invokeMethod(metodo, produto, new BigDecimal("0.1"));

// ─── Iteração sobre campos (percorre também superclasses) ────────────────────
ReflectionUtils.doWithFields(Produto.class, field -> {
    ReflectionUtils.makeAccessible(field);
    System.out.println(field.getName() + " = " + field.get(produto));
});

// ─── Iteração sobre métodos com filtro ───────────────────────────────────────
ReflectionUtils.doWithMethods(Produto.class,
    method -> System.out.println(method.getName()),
    method -> method.getParameterCount() == 0 && Modifier.isPublic(method.getModifiers()));
```

> **Quando usar `ReflectionUtils`:** em código de infraestrutura, serializers,
> plugins ou testes. Em código de negócio, reflexão geralmente indica design
> problemático — prefira polimorfismo ou interfaces.

---

### 4.8 `StopWatch` — Medição de Desempenho

`org.springframework.util.StopWatch` mede o tempo de execução de múltiplas tarefas
em sequência, útil para profiling rápido e logs de diagnóstico sem dependência de
biblioteca externa.

```java
import org.springframework.util.StopWatch;

StopWatch sw = new StopWatch("Importação de Produtos");

sw.start("Leitura do CSV");
var linhas = lerCsv(arquivo);
sw.stop();

sw.start("Validação");
var validos = validar(linhas);
sw.stop();

sw.start("Persistência");
produtoRepository.saveAll(validos);
sw.stop();

// ─── Resumo formatado ─────────────────────────────────────────────────────────
log.info(sw.prettyPrint());
// StopWatch 'Importação de Produtos': 1,234 ms
// ----------------------------------------
// ms     %     Task name
// ----------------------------------------
// 00045  04%   Leitura do CSV
// 00089  07%   Validação
// 01100  89%   Persistência

log.info("Total: {} ms", sw.getTotalTimeMillis());
log.info("Última tarefa: {} ms", sw.getLastTaskTimeMillis());
```

---

### 4.9 `AntPathMatcher` — Correspondência de Padrões de Caminho

`org.springframework.util.AntPathMatcher` implementa a notação Ant para matching
de caminhos — a mesma usada internamente pelo Spring MVC e Spring Security para
mapeamento de rotas.

```java
import org.springframework.util.AntPathMatcher;

AntPathMatcher matcher = new AntPathMatcher();

// ─── Símbolos da notação Ant ──────────────────────────────────────────────────
// ?  → qualquer caractere único
// *  → qualquer sequência dentro de um segmento (sem /)
// ** → qualquer sequência de segmentos (zero ou mais /)

matcher.match("/api/produtos/*",   "/api/produtos/42")         // true
matcher.match("/api/produtos/**",  "/api/produtos/42/itens")   // true
matcher.match("/api/*/itens",      "/api/pedidos/itens")       // true
matcher.match("/api/?rodutos",     "/api/produtos")            // true

// ─── Extração de variáveis de template ───────────────────────────────────────
Map<String, String> vars = matcher.extractUriTemplateVariables(
    "/api/{recurso}/{id}", "/api/produtos/42");
// {"recurso": "produtos", "id": "42"}

// ─── Verificar se string é um padrão ─────────────────────────────────────────
matcher.isPattern("/api/produtos/*")   // true
matcher.isPattern("/api/produtos/42")  // false

// ─── Uso em filtros de autorização customizados ───────────────────────────────
List<String> rotasPublicas = List.of("/api/auth/**", "/actuator/health");
boolean isPublica = rotasPublicas.stream()
        .anyMatch(padrao -> matcher.match(padrao, request.getRequestURI()));
```

---

### 4.10 `FileCopyUtils` — Operações de I/O com Cópia Automática

`org.springframework.util.FileCopyUtils` lida com leitura e escrita de arquivos e
streams, garantindo o fechamento automático dos recursos sem necessidade de
`try-with-resources` manual.

```java
import org.springframework.util.FileCopyUtils;

// ─── Ler arquivo como bytes ───────────────────────────────────────────────────
byte[] bytes = FileCopyUtils.copyToByteArray(new File("dados.csv"));

// ─── Ler InputStream como bytes (fecha o stream ao final) ────────────────────
byte[] bytes = FileCopyUtils.copyToByteArray(inputStream);

// ─── Ler Reader como String ───────────────────────────────────────────────────
String conteudo = FileCopyUtils.copyToString(
        new InputStreamReader(inputStream, StandardCharsets.UTF_8));

// ─── Copiar entre streams ─────────────────────────────────────────────────────
int bytesCopiados = FileCopyUtils.copy(inputStream, outputStream);

// ─── Copiar bytes para arquivo ────────────────────────────────────────────────
FileCopyUtils.copy(bytes, new File("saida.csv"));

// ─── Uso em download de arquivo em controller ─────────────────────────────────
@GetMapping("/relatorios/{id}/download")
public ResponseEntity<byte[]> download(@PathVariable Long id) throws IOException {
    var arquivo = relatorioService.gerarArquivo(id);
    byte[] conteudo = FileCopyUtils.copyToByteArray(arquivo);
    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"relatorio-" + id + ".csv\"")
            .contentType(MediaType.TEXT_PLAIN)
            .body(conteudo);
}
```

---

---

### 4.11 `MultiValueMap` e `LinkedMultiValueMap` — Mapas com Múltiplos Valores por Chave

`org.springframework.util.MultiValueMap<K,V>` estende `Map<K, List<V>>` e permite
que uma mesma chave tenha múltiplos valores associados — o mesmo modelo dos
parâmetros de query HTTP, headers e dados de formulário. A implementação padrão é
`LinkedMultiValueMap`, que preserva a ordem de inserção.

```java
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

// ─── Criação e adição de valores ──────────────────────────────────────────────
MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
params.add("cor", "vermelho");
params.add("cor", "azul");           // mesma chave, novo valor
params.add("cor", "verde");
params.add("tamanho", "M");

// ─── Leitura ──────────────────────────────────────────────────────────────────
String primeira = params.getFirst("cor");      // "vermelho"
List<String> todas = params.get("cor");        // ["vermelho", "azul", "verde"]

// ─── set() substitui todos os valores da chave ────────────────────────────────
params.set("cor", "preto");                    // ["preto"] — descarta anteriores

// ─── addAll() adiciona uma lista inteira ──────────────────────────────────────
params.addAll("tamanho", List.of("G", "GG"));

// ─── Converter para Map<K, V> com apenas o primeiro valor de cada chave ───────
Map<String, String> simples = params.toSingleValueMap();   // cópia
Map<String, String> visao   = params.asSingleValueMap();   // visão (live)

// ─── Uso no RestClient / WebClient ───────────────────────────────────────────
MultiValueMap<String, String> queryParams = new LinkedMultiValueMap<>();
queryParams.add("status", "ATIVO");
queryParams.add("status", "PENDENTE");

String uri = UriComponentsBuilder.fromPath("/api/pedidos")
        .queryParams(queryParams)
        .build().toUriString();
// /api/pedidos?status=ATIVO&status=PENDENTE

// ─── Uso em envio de formulário (application/x-www-form-urlencoded) ───────────
MultiValueMap<String, String> formData = new LinkedMultiValueMap<>();
formData.add("username", "joao");
formData.add("roles", "USER");
formData.add("roles", "ADMIN");

restClient.post().uri("/login")
        .contentType(MediaType.APPLICATION_FORM_URLENCODED)
        .body(formData)
        .retrieve().toBodilessEntity();
```

---

### 4.12 `LinkedCaseInsensitiveMap` — Mapa com Chaves Case-Insensitive

`org.springframework.util.LinkedCaseInsensitiveMap<V>` estende `LinkedHashMap`
para oferecer busca, verificação e remoção de chaves sem distinção de maiúsculas/
minúsculas, preservando a ordem de inserção e o case original das chaves.
Usado internamente pelo Spring nos `HttpHeaders`.

```java
import org.springframework.util.LinkedCaseInsensitiveMap;

// ─── Criação ──────────────────────────────────────────────────────────────────
LinkedCaseInsensitiveMap<String> headers = new LinkedCaseInsensitiveMap<>();

// ─── Inserção com case variado ────────────────────────────────────────────────
headers.put("Content-Type",   "application/json");
headers.put("AUTHORIZATION",  "Bearer token123");
headers.put("accept-language","pt-BR");

// ─── Busca case-insensitive ───────────────────────────────────────────────────
headers.get("content-type")    // "application/json"
headers.get("Authorization")   // "Bearer token123"
headers.get("Accept-Language") // "pt-BR"

// ─── containsKey também é case-insensitive ────────────────────────────────────
headers.containsKey("CONTENT-TYPE")   // true
headers.containsKey("authorization")  // true

// ─── keySet() preserva o case original ───────────────────────────────────────
headers.keySet().forEach(System.out::println);
// Content-Type
// AUTHORIZATION
// accept-language

// ─── Com Locale específico (importante para idiomas com regras de case especiais) ─
var mapa = new LinkedCaseInsensitiveMap<String>(16, Locale.ROOT);
mapa.put("İstanbul", "Turquia");     // case seguro independente de Locale
```

---

### 4.13 `DigestUtils` — Cálculo de Digests MD5

`org.springframework.util.DigestUtils` fornece métodos estáticos para calcular
digests MD5 de bytes ou streams. Útil para gerar checksums, ETags ou fingerprints
de conteúdo.

> **Atenção:** MD5 não é adequado para hashing de senhas (use BCrypt/Argon2) nem
> para verificação de integridade contra adversários (use SHA-256+). Use MD5
> apenas para checksums não-criptográficos (cache, ETag, detecção de duplicatas).

```java
import org.springframework.util.DigestUtils;

// ─── Hash como array de bytes ─────────────────────────────────────────────────
byte[] hash = DigestUtils.md5Digest("Spring Boot".getBytes(StandardCharsets.UTF_8));

// ─── Hash como String hexadecimal ────────────────────────────────────────────
String hex = DigestUtils.md5DigestAsHex("Spring Boot".getBytes(StandardCharsets.UTF_8));
// "f2b8e92bc0a0de3c40fbb38d30d3b087"

// ─── Hash de InputStream (stream NÃO é fechado automaticamente) ──────────────
try (var is = new FileInputStream("arquivo.bin")) {
    String hexHash = DigestUtils.md5DigestAsHex(is);
}

// ─── Append em StringBuilder ──────────────────────────────────────────────────
var sb = new StringBuilder("ETag: \"");
DigestUtils.appendMd5DigestAsHex("conteudo".getBytes(), sb);
sb.append("\"");
// ETag: "9a0364b9e99bb480dd25e1f0284c8555"

// ─── Geração de ETag em controller ───────────────────────────────────────────
@GetMapping("/produtos/{id}")
public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) {
    var produto = produtoService.buscarPorId(id);
    String etag = "\"" + DigestUtils.md5DigestAsHex(
            produto.toString().getBytes(StandardCharsets.UTF_8)) + "\"";
    return ResponseEntity.ok().eTag(etag).body(produto);
}
```

---

### 4.14 `SerializationUtils` — Serialização e Clone de Objetos

`org.springframework.util.SerializationUtils` oferece serialização Java padrão e
clonagem profunda de objetos `Serializable`.

> **Aviso de segurança:** `deserialize()` foi **depreciado no Spring 6.0** por
> vulnerabilidade de Remote Code Execution (RCE) ao desserializar dados de fontes
> não confiáveis. Prefira JSON/XML para persistência e transferência de dados.
> O método `clone()` (adicionado no Spring 6.0) é seguro pois opera sobre dados
> produzidos pelo próprio processo.

```java
import org.springframework.util.SerializationUtils;

// ─── Serializar objeto em bytes ───────────────────────────────────────────────
Produto produto = new Produto("Caneta", new BigDecimal("2.50"));
byte[] bytes = SerializationUtils.serialize(produto);   // Produto deve implementar Serializable

// ─── Clone profundo via serialização (Spring 6.0+) ───────────────────────────
// Cria uma cópia independente do objeto — todos os objetos aninhados são copiados
Produto copia = SerializationUtils.clone(produto);
copia.setNome("Lápis");       // não afeta o original

// ─── Uso típico: cópia de configuração para evitar mutação compartilhada ──────
@Service
public class ConfigService {
    private final AppConfig config;

    public AppConfig getConfigSnapshot() {
        // Retorna cópia para que chamadores não alterem o estado interno
        return SerializationUtils.clone(config);
    }
}
```

---

### 4.15 `MethodInvoker` — Invocação Declarativa de Métodos

`org.springframework.util.MethodInvoker` invoca métodos estáticos ou de instância
de forma programática e declarativa, com resolução de sobrecarga baseada nos
argumentos fornecidos.

```java
import org.springframework.util.MethodInvoker;

// ─── Método estático ──────────────────────────────────────────────────────────
MethodInvoker invoker = new MethodInvoker();
invoker.setTargetClass(Math.class);
invoker.setTargetMethod("max");
invoker.setArguments(5, 10);
invoker.prepare();
int resultado = (int) invoker.invoke();   // 10

// ─── Método de instância ──────────────────────────────────────────────────────
MethodInvoker invoker2 = new MethodInvoker();
invoker2.setTargetObject(produtoService);
invoker2.setTargetMethod("buscarPorId");
invoker2.setArguments(42L);
invoker2.prepare();
ProdutoResponse produto = (ProdutoResponse) invoker2.invoke();

// ─── Forma concisa para estático (nome completo com classe) ──────────────────
MethodInvoker invoker3 = new MethodInvoker();
invoker3.setStaticMethod("java.lang.Integer.parseInt");
invoker3.setArguments("42");
invoker3.prepare();
int valor = (int) invoker3.invoke();   // 42

// ─── Invoke pode ser chamado múltiplas vezes após prepare() ──────────────────
// prepare() é custoso (reflexão); invoke() é barato.
// Crie um invoker por método; reutilize invoke() para chamadas repetidas.
```

---

### 4.16 `PathMatcher` e `RouteMatcher` — Interfaces de Correspondência

O Spring define duas interfaces complementares para correspondência de caminhos:

- **`PathMatcher`** (`org.springframework.util`) — trabalha diretamente com `String`;
  implementação padrão: `AntPathMatcher` (seção 4.9).
- **`RouteMatcher`** (`org.springframework.util`) — trabalha com objetos `Route`
  pré-analisados, mais eficiente quando um padrão é comparado repetidamente contra
  muitas rotas. Implementação padrão: `SimpleRouteMatcher`.

```java
import org.springframework.util.PathMatcher;
import org.springframework.util.AntPathMatcher;
import org.springframework.util.RouteMatcher;
import org.springframework.util.SimpleRouteMatcher;

// ─── PathMatcher — uso genérico baseado em String ────────────────────────────
PathMatcher pathMatcher = new AntPathMatcher();
pathMatcher.match("/api/**", "/api/produtos/42")          // true
pathMatcher.isPattern("/api/**")                          // true
pathMatcher.isPattern("/api/produtos")                    // false
pathMatcher.combine("/api", "/produtos/{id}")             // "/api/produtos/{id}"

Map<String, String> vars = pathMatcher.extractUriTemplateVariables(
    "/api/{recurso}/{id}", "/api/produtos/42");
// {recurso=produtos, id=42}

// ─── RouteMatcher — eficiente para múltiplas rotas (ex: WebSocket, mensageria) ─
RouteMatcher routeMatcher = new SimpleRouteMatcher(new AntPathMatcher());

// Parsear a rota uma vez e reutilizar em múltiplas comparações
RouteMatcher.Route route = routeMatcher.parseRoute("/api/produtos/42");

routeMatcher.match("/api/**",            route)   // true
routeMatcher.match("/api/produtos/{id}", route)   // true
routeMatcher.isPattern("/api/**")                 // true

Map<String, String> vars2 = routeMatcher.matchAndExtract("/api/{recurso}/{id}", route);
// {recurso=produtos, id=42}

// ─── Quando usar cada interface ───────────────────────────────────────────────
// PathMatcher  → filtros, autorização, operações pontuais de matching
// RouteMatcher → roteamento de mensagens WebSocket/STOMP, event handlers
//               (a rota é parseada uma vez; o padrão é checado muitas vezes)
```

---

### 4.17 `ResourceUtils` — Resolução de Recursos no Classpath e Sistema de Arquivos

`org.springframework.util.ResourceUtils` resolve caminhos de recursos para
`java.io.File` ou `java.net.URL`, suportando os prefixos `classpath:` e `file:`.

> **Preferência:** para novo código, use a abstração `Resource` do Spring
> (`ResourceLoader`, `ClassPathResource`, `FileSystemResource`) — mais uniforme
> e testável. `ResourceUtils` é útil em código utilitário ou de configuração onde
> a injeção de `ResourceLoader` seria excessiva.

```java
import org.springframework.util.ResourceUtils;

// ─── Constantes de prefixo ────────────────────────────────────────────────────
ResourceUtils.CLASSPATH_URL_PREFIX   // "classpath:"
ResourceUtils.FILE_URL_PREFIX        // "file:"

// ─── Resolver arquivo do classpath ────────────────────────────────────────────
File props = ResourceUtils.getFile("classpath:application.properties");

// ─── Resolver arquivo do sistema de arquivos ──────────────────────────────────
File dados = ResourceUtils.getFile("file:/var/data/importacao.csv");

// ─── Verificar tipo de URL ────────────────────────────────────────────────────
URL url = new URL("jar:file:/app/lib/core.jar!/META-INF/config.xml");
ResourceUtils.isJarURL(url)          // true
ResourceUtils.isFileURL(url)         // false

// ─── Extrair URL do JAR a partir de URL de recurso interno ───────────────────
URL jarFileUrl = ResourceUtils.extractJarFileURL(url);
// file:/app/lib/core.jar

// ─── Verificar se string é uma URL válida ────────────────────────────────────
ResourceUtils.isUrl("classpath:config.xml")     // true
ResourceUtils.isUrl("file:/data/file.txt")      // true
ResourceUtils.isUrl("/relative/path.txt")       // false

// ─── Converter localização para URI com espaços codificados ──────────────────
URI uri = ResourceUtils.toURI("file:/caminho com espaços/arquivo.txt");
// file:/caminho%20com%20espa%C3%A7os/arquivo.txt

// ─── Uso em inicialização de aplicação ───────────────────────────────────────
@PostConstruct
public void carregarDados() throws IOException {
    File csv = ResourceUtils.getFile("classpath:dados/seed.csv");
    Files.lines(csv.toPath()).forEach(this::processarLinha);
}
```

---

### 4.18 `MimeTypeUtils` — Tipos MIME

`org.springframework.util.MimeTypeUtils` fornece constantes para tipos MIME comuns
e métodos para parsing, validação e ordenação de tipos MIME.

```java
import org.springframework.util.MimeTypeUtils;
import org.springframework.util.MimeType;

// ─── Constantes pré-definidas (MimeType e String _VALUE) ─────────────────────
MimeTypeUtils.APPLICATION_JSON          // application/json
MimeTypeUtils.APPLICATION_XML           // application/xml
MimeTypeUtils.APPLICATION_OCTET_STREAM  // application/octet-stream
MimeTypeUtils.TEXT_HTML                 // text/html
MimeTypeUtils.TEXT_PLAIN                // text/plain
MimeTypeUtils.TEXT_XML                  // text/xml
MimeTypeUtils.IMAGE_PNG                 // image/png
MimeTypeUtils.IMAGE_JPEG                // image/jpeg
MimeTypeUtils.ALL                       // */*

// ─── Parsing de um tipo MIME (resultado é cacheado) ──────────────────────────
MimeType json   = MimeTypeUtils.parseMimeType("application/json");
MimeType custom = MimeTypeUtils.parseMimeType("application/vnd.empresa+json;version=2");
MimeType comCharset = MimeTypeUtils.parseMimeType("text/html;charset=UTF-8");
// Lança InvalidMimeTypeException se a string for inválida

// ─── Parsing de lista separada por vírgula ────────────────────────────────────
List<MimeType> tipos = MimeTypeUtils.parseMimeTypes(
    "application/json, text/html;q=0.9, */*;q=0.8");

// ─── Tokenizar (respeita parâmetros entre aspas) ─────────────────────────────
List<String> tokens = MimeTypeUtils.tokenize(
    "text/html;charset=UTF-8, application/json");
// ["text/html;charset=UTF-8", "application/json"]

// ─── Ordenar por especificidade (RFC 7231) ────────────────────────────────────
List<MimeType> accept = MimeTypeUtils.parseMimeTypes("*/*, text/*, text/html");
MimeTypeUtils.sortBySpecificity(accept);
// [text/html, text/*, */*]  — mais específico primeiro

// ─── toString de coleção ─────────────────────────────────────────────────────
String header = MimeTypeUtils.toString(
    List.of(MimeTypeUtils.APPLICATION_JSON, MimeTypeUtils.TEXT_HTML));
// "application/json, text/html"

// ─── Gerar boundary para requisições multipart ───────────────────────────────
String boundary = MimeTypeUtils.generateMultipartBoundaryString();
// Ex: "----WebKitFormBoundary7MA4YWxkTrZu0gW"
```

---

### 4.19 `NumberUtils` — Conversão e Parsing de Números

`org.springframework.util.NumberUtils` converte entre tipos numéricos e faz parse
de strings para qualquer subtipo de `Number` do JDK.

```java
import org.springframework.util.NumberUtils;

// ─── Tipos suportados ─────────────────────────────────────────────────────────
// Byte, Short, Integer, Long, BigInteger, Float, Double, BigDecimal

// ─── Converter entre tipos numéricos ─────────────────────────────────────────
Integer intVal = NumberUtils.convertNumberToTargetClass(100L, Integer.class);
BigDecimal bd  = NumberUtils.convertNumberToTargetClass(42, BigDecimal.class);
// Lança IllegalArgumentException se o tipo alvo não for suportado

// ─── Parse de String (decimal) ────────────────────────────────────────────────
Integer n1 = NumberUtils.parseNumber("123", Integer.class);
Double  n2 = NumberUtils.parseNumber("  45.67  ", Double.class);  // trim automático
Long    n3 = NumberUtils.parseNumber("9876543210", Long.class);
BigDecimal n4 = NumberUtils.parseNumber("123.456789", BigDecimal.class);

// ─── Parse hexadecimal (prefixos 0x, 0X ou #) ────────────────────────────────
Integer hex = NumberUtils.parseNumber("0x1F", Integer.class);  // 31
Integer hex2 = NumberUtils.parseNumber("#FF", Integer.class);  // 255

// ─── Parse com NumberFormat (locale-aware) ────────────────────────────────────
// Útil para strings numéricas em formatos locais (vírgula como decimal, etc.)
NumberFormat nf = NumberFormat.getInstance(Locale.GERMAN);
Double german = NumberUtils.parseNumber("1.234,56", Double.class, nf);
// 1234.56

NumberFormat ptBr = NumberFormat.getInstance(new Locale("pt", "BR"));
Double brl = NumberUtils.parseNumber("1.234,56", Double.class, ptBr);
// 1234.56
```

---

### 4.20 `ConcurrentReferenceHashMap` — Cache Thread-Safe com Referências Fracas/Suaves

`org.springframework.util.ConcurrentReferenceHashMap<K,V>` é uma variante
thread-safe de `ConcurrentHashMap` que utiliza **referências soft ou weak** para
chaves e/ou valores, permitindo que o garbage collector recupere memória
automaticamente sob pressão.

```java
import org.springframework.util.ConcurrentReferenceHashMap;
import org.springframework.util.ConcurrentReferenceHashMap.ReferenceType;

// ─── SOFT (padrão) — libera entradas só sob pressão de memória ───────────────
ConcurrentReferenceHashMap<String, Relatorio> cache =
        new ConcurrentReferenceHashMap<>();

// ─── WEAK — libera entradas assim que não há mais referências fortes ─────────
ConcurrentReferenceHashMap<String, Sessao> sessions =
        new ConcurrentReferenceHashMap<>(16, ReferenceType.WEAK);

// ─── Configuração completa ────────────────────────────────────────────────────
var map = new ConcurrentReferenceHashMap<String, byte[]>(
    1024,          // capacidade inicial
    0.75f,         // load factor
    16,            // nível de concorrência (segmentos)
    ReferenceType.SOFT
);

// ─── API idêntica ao ConcurrentHashMap ───────────────────────────────────────
map.put("chave", dados);
byte[] val = map.get("chave");          // pode ser null se o GC coletou a entrada
map.computeIfAbsent("chave", k -> carregar(k));
map.putIfAbsent("chave", dados);

// ─── Forçar limpeza de entradas já coletadas pelo GC ─────────────────────────
map.purgeUnreferencedEntries();

// ─── Diferença entre SOFT e WEAK ─────────────────────────────────────────────
// SOFT: o GC só coleta quando a JVM está com pouca memória
//       → ideal para cache de objetos caros de recriar (relatórios, imagens)
// WEAK: o GC coleta na próxima passagem sem garantias de retenção
//       → ideal para metadata associado a objetos externos (listeners, wrappers)

// ─── Atenção: get() pode retornar null mesmo após put() ──────────────────────
// Sempre use computeIfAbsent() para garantir que a entrada existe ao acessar
```

> **Quando usar:** cache de dados caros de calcular, onde a perda de entradas
> sob pressão de memória é aceitável. Para cache com política de evicção explícita
> (TTL, LRU), prefira Caffeine ou Spring Cache com `@Cacheable`.

---

### 4.21 `ExponentialBackOff` e `FixedBackOff` — Estratégias de Retry com Backoff

`org.springframework.util.backoff` define a interface `BackOff` e duas
implementações para controlar o intervalo entre tentativas de retry.

```java
import org.springframework.util.backoff.BackOff;
import org.springframework.util.backoff.BackOffExecution;
import org.springframework.util.backoff.ExponentialBackOff;
import org.springframework.util.backoff.FixedBackOff;

// ════════════════════════════════════════════════════════════════════════════════
// ExponentialBackOff — intervalo cresce exponencialmente a cada tentativa
// ════════════════════════════════════════════════════════════════════════════════
ExponentialBackOff expBackOff = new ExponentialBackOff();
expBackOff.setInitialInterval(1_000);   // 1 s na 1ª tentativa
expBackOff.setMultiplier(2.0);          // dobra a cada falha: 1s, 2s, 4s, 8s…
expBackOff.setMaxInterval(30_000);      // teto de 30 s
expBackOff.setMaxElapsedTime(120_000);  // desiste após 2 min no total
expBackOff.setMaxAttempts(10);          // ou após 10 tentativas

// Sequência resultante: 1000, 2000, 4000, 8000, 16000, 30000, 30000… (ms)

// ════════════════════════════════════════════════════════════════════════════════
// FixedBackOff — intervalo constante (mais simples)
// ════════════════════════════════════════════════════════════════════════════════
FixedBackOff fixedBackOff = new FixedBackOff(2_000, 3);
// 2 s entre tentativas, máximo 3 tentativas

// ════════════════════════════════════════════════════════════════════════════════
// Uso com BackOffExecution — controle manual do loop de retry
// ════════════════════════════════════════════════════════════════════════════════
BackOffExecution execution = expBackOff.start();

long waitMs;
while ((waitMs = execution.nextBackOff()) != BackOffExecution.STOP) {
    try {
        chamarApiExterna();
        return;                          // sucesso — sai do loop
    } catch (TransientException e) {
        log.warn("Falha temporária, próxima tentativa em {} ms", waitMs);
        Thread.sleep(waitMs);
    }
}
throw new ServiceUnavailableException("API indisponível após todas as tentativas");

// ════════════════════════════════════════════════════════════════════════════════
// Uso com Spring AMQP (RabbitMQ) — configuração de retry listener
// ════════════════════════════════════════════════════════════════════════════════
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
        ConnectionFactory connectionFactory) {

    var factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);

    var retryTemplate = new RetryTemplate();
    retryTemplate.setBackOffPolicy(new ExponentialBackOffPolicy());
    factory.setRetryTemplate(retryTemplate);
    return factory;
}
```

| Estratégia | Intervalo | Uso típico |
|---|---|---|
| `FixedBackOff` | Constante | Retry simples com intervalo previsível |
| `ExponentialBackOff` | Cresce exponencialmente | APIs externas, filas — evita thundering herd |

---

### 4.22 `IdGenerator` e Implementações — Geração de UUIDs

`org.springframework.util.IdGenerator` é uma interface funcional para geração de
UUIDs. O Spring fornece três implementações prontas com perfis de desempenho
diferentes.

```java
import org.springframework.util.IdGenerator;
import org.springframework.util.JdkIdGenerator;
import org.springframework.util.AlternativeJdkIdGenerator;
import org.springframework.util.SimpleIdGenerator;

// ─── JdkIdGenerator — delega para UUID.randomUUID() (padrão JDK) ─────────────
// Usa SecureRandom a cada chamada; mais lento sob alta concorrência
IdGenerator jdk = new JdkIdGenerator();
UUID id1 = jdk.generateId();

// ─── AlternativeJdkIdGenerator — SecureRandom apenas para seed; Random para geração ─
// Mais rápido que JdkIdGenerator em cenários de alta concorrência
IdGenerator alt = new AlternativeJdkIdGenerator();
UUID id2 = alt.generateId();   // recomendado para produção com alto volume de IDs

// ─── SimpleIdGenerator — UUID sequencial (00000000-0000-0000-0000-000000000001…) ─
// Determinístico; útil apenas em testes
IdGenerator simple = new SimpleIdGenerator();
UUID id3 = simple.generateId();   // 00000000-0000-0000-0000-000000000001
UUID id4 = simple.generateId();   // 00000000-0000-0000-0000-000000000002

// ─── Injeção via Spring ───────────────────────────────────────────────────────
@Configuration
public class IdConfig {
    @Bean
    public IdGenerator idGenerator() {
        return new AlternativeJdkIdGenerator();   // troca para SimpleIdGenerator em testes
    }
}

@Service
public class PedidoService {
    private final IdGenerator idGenerator;

    public PedidoService(IdGenerator idGenerator) {
        this.idGenerator = idGenerator;
    }

    @Transactional
    public PedidoResponse criar(PedidoRequest request) {
        var pedido = Pedido.from(request);
        pedido.setCorrelationId(idGenerator.generateId());
        return PedidoResponse.from(pedidoRepository.save(pedido));
    }
}
```

| Implementação | Algoritmo | Desempenho | Quando usar |
|---|---|---|---|
| `JdkIdGenerator` | `UUID.randomUUID()` | Médio | Uso geral |
| `AlternativeJdkIdGenerator` | SecureRandom seed + Random | Alto | Alto volume, produção |
| `SimpleIdGenerator` | Sequencial | Muito alto | Testes unitários |

---

### 4.23 `PropertyPlaceholderHelper` — Resolução de Placeholders em Strings

`org.springframework.util.PropertyPlaceholderHelper` resolve placeholders do tipo
`${chave}` (ou formato customizado) em strings arbitrárias, similar ao que o Spring
faz internamente ao processar `@Value` e `application.properties`.

```java
import org.springframework.util.PropertyPlaceholderHelper;

// ─── Configuração básica ──────────────────────────────────────────────────────
var helper = new PropertyPlaceholderHelper("${", "}");

// ─── Resolver com Properties ──────────────────────────────────────────────────
Properties props = new Properties();
props.setProperty("app.nome",    "SistemaLoja");
props.setProperty("app.versao",  "2.1.0");
props.setProperty("app.env",     "producao");

String template = "Bem-vindo ao ${app.nome} v${app.versao} [${app.env}]";
String resultado = helper.replacePlaceholders(template, props);
// "Bem-vindo ao SistemaLoja v2.1.0 [producao]"

// ─── Resolver com lambda (PlaceholderResolver) ────────────────────────────────
String msg = helper.replacePlaceholders(
    "OS: ${os.name}, Usuário: ${user.name}",
    System::getProperty   // method reference para System.getProperty(String)
);

// ─── Com valor padrão (separador :) ──────────────────────────────────────────
var helperComDefault = new PropertyPlaceholderHelper("${", "}", ":", null, true);
String porta = helperComDefault.replacePlaceholders(
    "Porta: ${server.port:8080}", new Properties());
// "Porta: 8080" (chave ausente usa o default)

// ─── Ignorar placeholders não resolvidos (ignoreUnresolvablePlaceholders=true) ─
var lenient = new PropertyPlaceholderHelper("${", "}", null, null, true);
String out = lenient.replacePlaceholders(
    "Valor: ${definido} e ${nao.definido}", props);
// "Valor: <valor> e ${nao.definido}"  — placeholder não encontrado é mantido

// ─── Caso de uso: geração de mensagens de notificação com template ─────────────
@Service
public class NotificacaoService {
    private static final PropertyPlaceholderHelper HELPER =
            new PropertyPlaceholderHelper("${", "}");

    public String renderizar(String template, Map<String, String> dados) {
        Properties props = new Properties();
        props.putAll(dados);
        return HELPER.replacePlaceholders(template, props);
    }
}
// template: "Olá ${nome}, seu pedido #${pedidoId} foi confirmado."
// dados:    {nome=Maria, pedidoId=1234}
```

---

### 4.24 `SocketUtils` / `TestSocketUtils` — Portas TCP Disponíveis em Testes

`SocketUtils` foi **removido no Spring 6.0** (depreciado desde 5.3.16).
O substituto é `TestSocketUtils` em `org.springframework.test.util`, disponível
no módulo `spring-test`.

> **Aviso:** encontrar uma porta livre e tentar usá-la logo em seguida é inerentemente
> sujeito a condição de corrida. A abordagem recomendada é deixar o servidor
> escolher a própria porta (porta 0) e consultar qual porta foi atribuída após
> a inicialização.

```java
// ─── TestSocketUtils (substituto, Spring 6.0+) — apenas para testes ──────────
import org.springframework.test.util.TestSocketUtils;

// Encontra uma porta TCP aleatória disponível no range [1024, 65535]
int porta = TestSocketUtils.findAvailableTcpPort();

// ─── Abordagem preferida: porta 0 (o SO escolhe) ─────────────────────────────
// Em testes de integração com Spring Boot, configure porta=0 e leia a porta real:

// application-test.yml
// server:
//   port: 0

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegracaoTest {

    @LocalServerPort              // injeta a porta atribuída pelo SO
    private int porta;

    @Test
    void deveResponder() {
        RestAssured.given()
                .baseUri("http://localhost:" + porta)
                .get("/actuator/health")
                .then().statusCode(200);
    }
}

// ─── Para servidores embutidos (ex: WireMock em testes) ──────────────────────
WireMockServer wireMock = new WireMockServer(
    WireMockConfiguration.options().dynamicPort());
wireMock.start();
int portaWireMock = wireMock.port();   // consulta após inicializar
```

---

### 4.25 `DataSize` — Representação de Tamanhos de Dados

`org.springframework.util.unit.DataSize` representa tamanhos de dados de forma
imutável e type-safe, usando prefixos binários (potências de 2). Integra-se
diretamente ao sistema de binding de propriedades do Spring Boot.

```java
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

// ─── Criação via factory methods ─────────────────────────────────────────────
DataSize cincoMb  = DataSize.ofMegabytes(5);
DataSize doisGb   = DataSize.ofGigabytes(2);
DataSize quinzeKb = DataSize.ofKilobytes(15);
DataSize generico = DataSize.of(512, DataUnit.KILOBYTES);

// ─── Parse de string ──────────────────────────────────────────────────────────
DataSize s1 = DataSize.parse("10MB");     // 10 megabytes
DataSize s2 = DataSize.parse("1GB");      // 1 gigabyte
DataSize s3 = DataSize.parse("2048");     // 2048 bytes (sem unidade = bytes)
DataSize s4 = DataSize.parse("20", DataUnit.KILOBYTES);  // 20 KB (unidade padrão)

// ─── Conversão entre unidades ─────────────────────────────────────────────────
DataSize tamanho = DataSize.ofMegabytes(5);
tamanho.toBytes();       // 5_242_880
tamanho.toKilobytes();   // 5_120
tamanho.toMegabytes();   // 5
tamanho.toGigabytes();   // 0

// ─── Utilitários ──────────────────────────────────────────────────────────────
tamanho.isNegative()     // false
tamanho.toString()       // "5MB"
tamanho.compareTo(DataSize.ofMegabytes(3))  // > 0

// ─── Binding em @ConfigurationProperties ─────────────────────────────────────
@ConfigurationProperties(prefix = "app.storage")
public class StorageProperties {
    private DataSize maxUploadSize    = DataSize.ofMegabytes(10);
    private DataSize maxCacheSize     = DataSize.ofGigabytes(1);
    private DataSize chunkSize        = DataSize.ofKilobytes(64);

    // getters / setters
}

// application.yml:
// app.storage:
//   max-upload-size: 50MB
//   max-cache-size: 4GB
//   chunk-size: 128KB

// ─── Validação de tamanho de arquivo em serviço ───────────────────────────────
@Service
public class UploadService {
    private final StorageProperties props;

    public void validar(MultipartFile arquivo) {
        long limiteBytes = props.getMaxUploadSize().toBytes();
        if (arquivo.getSize() > limiteBytes) {
            throw new NegocioException(
                "Arquivo excede o limite de " + props.getMaxUploadSize());
        }
    }
}
```

| Unidade | Constante `DataUnit` | Bytes equivalentes |
|---|---|---|
| Byte | `DataUnit.BYTES` | 1 |
| Kilobyte | `DataUnit.KILOBYTES` | 1.024 |
| Megabyte | `DataUnit.MEGABYTES` | 1.048.576 |
| Gigabyte | `DataUnit.GIGABYTES` | 1.073.741.824 |
| Terabyte | `DataUnit.TERABYTES` | 1.099.511.627.776 |

---

### 4.26 Resumo das Classes Utilitárias

| Classe | Pacote | Finalidade Principal |
|---|---|---|
| `StringUtils` | `org.springframework.util` | Validação e manipulação de strings null-safe |
| `Assert` | `org.springframework.util` | Pré-condições e invariantes (lança exceção se falhar) |
| `CollectionUtils` | `org.springframework.util` | Operações null-safe em coleções |
| `ObjectUtils` | `org.springframework.util` | Operações null-safe em objetos e arrays |
| `ClassUtils` | `org.springframework.util` | Introspecção e carregamento dinâmico de classes |
| `BeanUtils` | `org.springframework.beans` | Cópia de propriedades e instanciação de beans |
| `ReflectionUtils` | `org.springframework.util` | Reflexão sem checked exceptions |
| `StopWatch` | `org.springframework.util` | Medição de tempo de execução por tarefa |
| `AntPathMatcher` | `org.springframework.util` | Matching de caminhos com notação Ant |
| `FileCopyUtils` | `org.springframework.util` | Cópia/leitura de arquivos e streams |
| `MultiValueMap` / `LinkedMultiValueMap` | `org.springframework.util` | Map com múltiplos valores por chave |
| `LinkedCaseInsensitiveMap` | `org.springframework.util` | Map String com busca case-insensitive |
| `DigestUtils` | `org.springframework.util` | Cálculo de hash MD5 (checksum, ETag) |
| `SerializationUtils` | `org.springframework.util` | Serialização Java e clone profundo |
| `MethodInvoker` | `org.springframework.util` | Invocação declarativa de métodos por reflexão |
| `PathMatcher` | `org.springframework.util` | Interface para matching de caminhos (impl: `AntPathMatcher`) |
| `RouteMatcher` / `SimpleRouteMatcher` | `org.springframework.util` | Matching eficiente de rotas pré-analisadas |
| `ResourceUtils` | `org.springframework.util` | Resolução de recursos `classpath:` e `file:` |
| `MimeTypeUtils` | `org.springframework.util` | Constantes e parsing de tipos MIME |
| `NumberUtils` | `org.springframework.util` | Conversão e parsing de números |
| `ConcurrentReferenceHashMap` | `org.springframework.util` | Cache thread-safe com referências soft/weak |
| `ExponentialBackOff` | `org.springframework.util.backoff` | Retry com intervalo crescente exponencialmente |
| `FixedBackOff` | `org.springframework.util.backoff` | Retry com intervalo fixo |
| `IdGenerator` / `AlternativeJdkIdGenerator` | `org.springframework.util` | Geração de UUIDs |
| `PropertyPlaceholderHelper` | `org.springframework.util` | Resolução de placeholders `${chave}` em strings |
| `TestSocketUtils` | `org.springframework.test.util` | Porta TCP aleatória disponível (somente testes) |
| `DataSize` | `org.springframework.util.unit` | Representação type-safe de tamanhos de dados |

---
