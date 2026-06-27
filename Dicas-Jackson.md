# Jackson — Guia Completo: Serialização e Desserialização JSON

> **Baseline principal:** Jackson 2.17+ · Spring Boot 3.5 · Java 21+
>
> **Notas de compatibilidade:** quando uma seção exigir Jackson 3.x / Spring Boot 4, isso será indicado explicitamente com a tag **`[Jackson 3 / SB4]`**.

---

## Sumário

- [Jackson 2 vs Jackson 3 — Diferenças e Migração](#jackson-2-vs-jackson-3--diferenças-e-migração)
    - [JsonMapper — O Novo Entry Point do Jackson 3](#jsonmapper--o-novo-entry-point-do-jackson-3)

1. [Visão Geral e Arquitetura](#1-visão-geral-e-arquitetura)
    - [1.1 Módulos Principais](#11-módulos-principais)
    - [1.2 Ciclo de Vida: Serialização e Desserialização](#12-ciclo-de-vida-serialização-e-desserialização)
2. [Configuração e Dependências](#2-configuração-e-dependências)
    - [2.1 Dependências Maven — Spring Boot 3](#21-dependências-maven--spring-boot-3)
    - [2.2 Dependências Maven — Spring Boot 4 / Jackson 3](#22-dependências-maven--spring-boot-4--jackson-3)
    - [2.3 ObjectMapper — Configuração Programática](#23-objectmapper--configuração-programática)
    - [2.4 application.yml — Propriedades Jackson no Spring Boot](#24-applicationyml--propriedades-jackson-no-spring-boot)
3. [Anotações Essenciais](#3-anotações-essenciais)
    - [3.1 @JsonProperty — Renomear Campos](#31-jsonproperty--renomear-campos)
    - [3.2 @JsonIgnore e @JsonIgnoreProperties](#32-jsonignore-e-jsonignoreproperties)
    - [3.3 @JsonAlias — Múltiplos Nomes na Leitura](#33-jsonalias--múltiplos-nomes-na-leitura)
    - [3.4 @JsonInclude — Controlar Campos Nulos e Vazios](#34-jsoninclude--controlar-campos-nulos-e-vazios)
    - [3.5 @JsonFormat — Formatação de Datas e Números](#35-jsonformat--formatação-de-datas-e-números)
    - [3.6 @JsonCreator e @JsonValue — Controlar Construção e Serialização](#36-jsoncreator-e-jsonvalue--controlar-construção-e-serialização)
    - [3.7 @JsonUnwrapped — Achatar Objetos Aninhados](#37-jsonunwrapped--achatar-objetos-aninhados)
    - [3.8 @JsonAnyGetter e @JsonAnySetter — Propriedades Dinâmicas](#38-jsonanygetter-e-jsonanysetter--propriedades-dinâmicas)
    - [3.9 @JsonRawValue — JSON Embutido sem Escape](#39-jsonrawvalue--json-embutido-sem-escape)
    - [3.10 @JsonRootName — Envelope Raiz](#310-jsonrootname--envelope-raiz)
    - [3.11 @JsonView — Visões Parciais do Objeto](#311-jsonview--visões-parciais-do-objeto)
    - [3.12 @JsonManagedReference e @JsonBackReference — Referências Bidirecionais](#312-jsonmanagedreference-e-jsonbackreference--referências-bidirecionais)
4. [Serialização e Desserialização Customizadas](#4-serialização-e-desserialização-customizadas)
    - [4.1 Serializer Customizado](#41-serializer-customizado)
    - [4.2 Deserializer Customizado](#42-deserializer-customizado)
    - [4.3 @JsonSerialize e @JsonDeserialize — Aplicação Pontual](#43-jsonserialize-e-jsondeserialize--aplicação-pontual)
    - [4.4 StdConverter — Conversão Simples](#44-stdconverter--conversão-simples)
    - [4.5 Boolean como 0/1 — Integração com APIs Legadas](#45-boolean-como-01--integração-com-apis-legadas)
5. [Polimorfismo — @JsonTypeInfo e @JsonSubTypes](#5-polimorfismo--jsontypeinfo-e-jsonsubtypes)
    - [5.1 Configuração Básica](#51-configuração-básica)
    - [5.2 Tipos de Inclusão de Tipo](#52-tipos-de-inclusão-de-tipo)
    - [5.3 Polimorfismo em Coleções](#53-polimorfismo-em-coleções)
6. [Módulos Jackson](#6-módulos-jackson)
    - [6.1 JavaTimeModule — Java 8+ Date/Time](#61-javatimemodule--java-8-datetime)
    - [6.2 ParameterNamesModule — Sem @JsonCreator](#62-parameternamesmodule--sem-jsoncreator)
    - [6.3 JavaOptionalModule / Jdk8Module](#63-javaoptionalmodule--jdk8module)
    - [6.4 AfterburnerModule / BlackbirdModule — Performance](#64-afterburnermodule--blackbirdmodule--performance)
7. [Java Records e Jackson](#7-java-records-e-jackson)
    - [7.1 Suporte Nativo a Records (Jackson 2.12+)](#71-suporte-nativo-a-records-jackson-212)
    - [7.2 Records com Anotações](#72-records-com-anotações)
    - [7.3 Records como DTOs — Padrão Recomendado](#73-records-como-dtos--padrão-recomendado)
8. [JsonNode — Manipulação de JSON Dinâmico](#8-jsonnode--manipulação-de-json-dinâmico)
    - [8.1 Leitura com JsonNode](#81-leitura-com-jsonnode)
    - [8.2 Construção de JSON Dinâmico](#82-construção-de-json-dinâmico)
    - [8.3 JsonNode vs Map — Quando Usar Cada Um](#83-jsonnode-vs-map--quando-usar-cada-um)
9. [Integração com Spring Boot](#9-integração-com-spring-boot)
    - [9.1 Auto-configuração do Jackson](#91-auto-configuração-do-jackson)
    - [9.2 Jackson2ObjectMapperBuilderCustomizer](#92-jackson2objectmapperbuildercustomizer)
    - [9.3 @JsonComponent — Registrar Customizações como Beans](#93-jsoncomponent--registrar-customizações-como-beans)
    - [9.4 @JsonView em Controllers REST](#94-jsonview-em-controllers-rest)
    - [9.5 MixIn — Anotações sem Modificar Terceiros](#95-mixin--anotações-sem-modificar-terceiros)
10. [Casos de Uso Avançados](#10-casos-de-uso-avançados)
    - [10.1 Desserialização de Tipos Genéricos — TypeReference](#101-desserialização-de-tipos-genéricos--typereference)
    - [10.2 Tree Model — readTree e convertValue](#102-tree-model--readtree-e-convertvalue)
    - [10.3 Streaming API — JsonParser e JsonGenerator](#103-streaming-api--jsonparser-e-jsongenerator)
    - [10.4 InjectableValues — Injetar Dependências no Deserializer](#104-injectablevalues--injetar-dependências-no-deserializer)
    - [10.5 JSON Merge Patch — Atualização Parcial (PATCH)](#105-json-merge-patch--atualização-parcial-patch)
    - [10.6 @JsonView Dinâmico por Role do Usuário Logado](#106-jsonview-dinâmico-por-role-do-usuário-logado)
11. [Tratamento de Erros de Desserialização](#11-tratamento-de-erros-de-desserialização)
12. [Testes com Jackson](#12-testes-com-jackson)
13. [Boas Práticas e Armadilhas Comuns](#13-boas-práticas-e-armadilhas-comuns)

---

> **Como navegar este material:** para uma primeira leitura, priorize as seções 2, 3, 7, 9 e a tabela comparativa Jackson 2 vs 3. As seções 4, 5, 8 e 10 funcionam melhor como consulta e aprofundamento.

---

## Jackson 2 vs Jackson 3 — Diferenças e Migração

Jackson 3 representa uma reescrita parcial com foco em modernidade, desempenho e eliminação de legado acumulado desde 2012. Spring Boot 4 adotará Jackson 3 como baseline.

### Tabela Comparativa

| Aspecto | Jackson 2.x | Jackson 3.x |
|---|---|---|
| **Java mínimo** | Java 8 | Java 17 |
| **Namespace** | `com.fasterxml.jackson` | `tools.jackson` *(novo pacote raiz)* |
| **Módulo core** | `jackson-core`, `jackson-databind`, `jackson-annotations` | Mesma estrutura, porém `jackson-databind` mais enxuto |
| **Suporte a Records** | Parcial (desde 2.12, via reflexão) | Nativo e completo |
| **Suporte a sealed classes** | Limitado / manual | Nativo para polimorfismo |
| **`@JsonKey`** | Não existe | Novo: controla serialização de chaves de Map |
| **`MapperFeature.DEFAULT_VIEW_INCLUSION`** | `true` por padrão | `false` por padrão — campos sem `@JsonView` são omitidos |
| **`ObjectMapper` mutável** | Sim (thread-unsafe após configuração tardia) | `ObjectMapper` imutável; mutações via `ObjectMapper.rebuild()` |
| **Entry point principal** | `new ObjectMapper()` | `JsonMapper.builder().build()` — subtipo especializado para JSON |
| **`MapperBuilder` fluente** | Não existe (`ObjectMapper` configurado por setters) | `JsonMapper.builder()` retorna `JsonMapper.Builder` com API fluente completa |
| **Mappers por formato** | Subclasses: `XmlMapper`, `CsvMapper`, `YAMLMapper` | Mesma estrutura; `JsonMapper` é o mapper de JSON explícito (não confundir com `ObjectMapper` genérico) |
| **Módulo Kotlin** | `jackson-module-kotlin` | Integração melhorada com coroutines |
| **Módulo afterburner** | `jackson-module-afterburner` (bytecode) | Substituído por `jackson-module-blackbird` (MethodHandles) |
| **APIs descontinuadas** | Muitas presentes por retrocompatibilidade | Removidas: `ObjectMapper.defaultPrettyPrintingWriter()`, `getJsonFactory()`, `ObjectMapper.configure(Feature, boolean)` para features de parser/generator |
| **Leitura de propriedade desconhecida** | Falha com `FAIL_ON_UNKNOWN_PROPERTIES=true` (padrão `false` no Spring) | Mesmo comportamento, configuração mais explícita |
| **Spring Boot** | Spring Boot 3.x | Spring Boot 4.x |

### APIs Removidas no Jackson 3 — Guia de Migração

```java
// ─── Jackson 2 → Jackson 3: API do ObjectMapper ───────────────────────────────

// ANTES (Jackson 2) — obsoleto
ObjectMapper mapper = new ObjectMapper();
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false); // OK em 2
JsonFactory factory = mapper.getJsonFactory();                              // ❌ removido em 3

// DEPOIS (Jackson 3) — imutável após construção
ObjectMapper mapper = JsonMapper.builder()
    .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
    .build();
// Para modificar uma instância existente:
ObjectMapper novaInstancia = mapper.rebuild()
    .enable(SerializationFeature.INDENT_OUTPUT)
    .build();

// ─── Namespace (Jackson 3) ────────────────────────────────────────────────────
// Jackson 2: import com.fasterxml.jackson.databind.ObjectMapper;
// Jackson 3: import tools.jackson.databind.ObjectMapper;
//
// ⚠️ ATENÇÃO: Spring Boot 4 gerencia esse import via auto-configuração.
//    Na prática você raramente precisa trocar imports manualmente quando usa Spring Boot.
```

### `JsonMapper` — O Novo Entry Point do Jackson 3

**`[Jackson 3 / SB4]`**

No Jackson 2, `ObjectMapper` era usado diretamente para JSON, XML, YAML e qualquer outro formato. No Jackson 3, cada formato ganhou um subtipo dedicado de `ObjectMapper`. `JsonMapper` é o mapper oficial para JSON e substitui `new ObjectMapper()` como ponto de entrada padrão.

```
ObjectMapper (abstrato — base para todos os formatos)
├── JsonMapper       → JSON               (tools.jackson.databind.json.JsonMapper)
├── XmlMapper        → XML                (tools.jackson.dataformat.xml.XmlMapper)
├── CsvMapper        → CSV                (tools.jackson.dataformat.csv.CsvMapper)
├── YAMLMapper       → YAML               (tools.jackson.dataformat.yaml.YAMLMapper)
└── CBORMapper       → CBOR (binário)     (tools.jackson.dataformat.cbor.CBORMapper)
```

#### Criação com `JsonMapper.builder()` — API Fluente

```java
import tools.jackson.databind.json.JsonMapper;             // [Jackson 3]
import tools.jackson.databind.SerializationFeature;
import tools.jackson.databind.DeserializationFeature;
import tools.jackson.databind.MapperFeature;
import tools.jackson.databind.json.JsonMapper.Builder;

// ─── Configuração completa via builder ────────────────────────────────────────
JsonMapper mapper = JsonMapper.builder()

    // ── Serialização ──────────────────────────────────────────────────────────
    .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
    .enable(SerializationFeature.INDENT_OUTPUT)              // pretty print
    .serializationInclusion(JsonInclude.Include.NON_NULL)    // omite nulos

    // ── Desserialização ───────────────────────────────────────────────────────
    .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
    .enable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY)

    // ── Mapper features ───────────────────────────────────────────────────────
    .enable(MapperFeature.ACCEPT_CASE_INSENSITIVE_PROPERTIES)
    .enable(MapperFeature.DEFAULT_VIEW_INCLUSION)            // padrão=false em J3; habilita para compatibilidade

    // ── Módulos ───────────────────────────────────────────────────────────────
    .addModule(new JavaTimeModule())
    .addModule(new BlackbirdModule())                        // performance

    // ── Naming strategy ───────────────────────────────────────────────────────
    .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)

    .build();  // ← ObjectMapper imutável a partir daqui

// Tipo concreto: tools.jackson.databind.json.JsonMapper
// É também um ObjectMapper — pode ser injetado onde ObjectMapper é esperado
```

#### `rebuild()` — Criar Variantes sem Mutar a Instância Original

No Jackson 3, `ObjectMapper` é **imutável após a construção**. Para criar uma versão modificada:

```java
// ─── Instância base compartilhada (bean Spring, por ex.) ──────────────────────
JsonMapper mapperBase = JsonMapper.builder()
    .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
    .addModule(new JavaTimeModule())
    .build();

// ─── Variante com pretty print — não afeta mapperBase ─────────────────────────
JsonMapper mapperPretty = (JsonMapper) mapperBase.rebuild()
    .enable(SerializationFeature.INDENT_OUTPUT)
    .build();

// ─── Variante com SNAKE_CASE — sem modules duplicados ─────────────────────────
JsonMapper mapperSnake = (JsonMapper) mapperBase.rebuild()
    .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
    .build();
```

> **Por que `rebuild()` em vez de `copy()`?**
> `ObjectMapper.copy()` ainda existe no Jackson 3 mas retorna uma instância mutável (legado de compatibilidade). `rebuild()` retorna um `MapperBuilder` tipado e é a abordagem idiomática do Jackson 3.

#### `JsonMapper` no Spring Boot 4

O Spring Boot 4 usa `JsonMapper` internamente, mas a API de customização via `Jackson2ObjectMapperBuilderCustomizer` é mantida por compatibilidade. Para configurações avançadas, prefira registrar um `@Bean` do tipo `JsonMapper`:

```java
@Configuration
public class JacksonConfig {

    @Bean
    @Primary
    public JsonMapper jsonMapper() {
        return JsonMapper.builder()
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
            .serializationInclusion(JsonInclude.Include.NON_NULL)
            .addModule(new JavaTimeModule())
            .addModule(new BlackbirdModule())
            .build();
    }
}
```

> Como `JsonMapper extends ObjectMapper`, injetar `ObjectMapper` ou `JsonMapper` como dependência funciona da mesma forma. Prefira `JsonMapper` nos próprios beans de configuração para deixar explícito que o formato é JSON.

#### Verificar Tipo em Runtime

```java
ObjectMapper mapper = context.getBean(ObjectMapper.class);

System.out.println(mapper.getClass().getName());
// Spring Boot 3 (Jackson 2): com.fasterxml.jackson.databind.ObjectMapper
// Spring Boot 4 (Jackson 3): tools.jackson.databind.json.JsonMapper

// Verificação de instância
boolean isJsonMapper = mapper instanceof JsonMapper;  // true em Jackson 3 com Spring Boot 4
```

#### Diferença Prática: `ObjectMapper` vs `JsonMapper` no Jackson 3

```java
// Ambos funcionam em Jackson 3 para JSON puro:
ObjectMapper omGenerico = new ObjectMapper();           // ainda válido, mas sem tipo explícito
JsonMapper   jmJson     = JsonMapper.builder().build(); // ✅ preferido — semântica clara

// Para outros formatos, os subtipos especializados são obrigatórios:
XmlMapper  xmlMapper  = XmlMapper.builder().build();    // requer jackson-dataformat-xml
YAMLMapper yamlMapper = YAMLMapper.builder().build();   // requer jackson-dataformat-yaml
```

| Situação | Jackson 2 | Jackson 3 |
|---|---|---|
| Criar mapper JSON | `new ObjectMapper()` | `JsonMapper.builder().build()` |
| Modificar após criação | `mapper.configure(...)` diretamente | `mapper.rebuild()....build()` |
| Injetar no Spring | `@Bean ObjectMapper` | `@Bean JsonMapper` (ou `ObjectMapper`) |
| Obter `JsonFactory` | `mapper.getJsonFactory()` ❌ removido em J3 | `mapper.tokenStreamFactory()` |

### `DEFAULT_VIEW_INCLUSION` — Quebra de Comportamento Importante

```java
// Cenário: campo sem @JsonView numa classe que usa @JsonView em outros campos

public class Produto {
    @JsonView(Views.Publico.class)
    public String nome;

    public BigDecimal custoInterno; // sem @JsonView
}

// Jackson 2: custoInterno APARECE quando nenhuma view é ativa (DEFAULT_VIEW_INCLUSION=true)
// Jackson 3: custoInterno NÃO APARECE (DEFAULT_VIEW_INCLUSION=false por padrão)

// Migração: habilitar explicitamente se o comportamento antigo for necessário
ObjectMapper mapper = JsonMapper.builder()
    .enable(MapperFeature.DEFAULT_VIEW_INCLUSION) // restaura comportamento do Jackson 2
    .build();
```

---

## 1. Visão Geral e Arquitetura

Jackson é a biblioteca de processamento JSON mais utilizada no ecossistema Java. O Spring Boot a usa internamente para serialização/desserialização em REST APIs, clientes HTTP, cache e mensageria.

### 1.1 Módulos Principais

```
jackson-core          → Streaming API: JsonParser, JsonGenerator (baixo nível)
jackson-annotations   → @JsonProperty, @JsonIgnore, @JsonView, etc.
jackson-databind      → ObjectMapper: ponte entre JSON e objetos Java
```

**Módulos opcionais registrados automaticamente pelo Spring Boot:**

| Módulo | Artefato Maven | Função |
|---|---|---|
| `JavaTimeModule` | `jackson-datatype-jsr310` | `LocalDate`, `Instant`, `ZonedDateTime` |
| `Jdk8Module` | `jackson-datatype-jdk8` | `Optional<T>`, `OptionalInt`, etc. |
| `ParameterNamesModule` | `jackson-module-parameter-names` | Construtores sem `@JsonCreator` |
| `BlackbirdModule` | `jackson-module-blackbird` | Performance via MethodHandles (substitui Afterburner) |

### 1.2 Ciclo de Vida: Serialização e Desserialização

```
Serialização (Java → JSON):
  Objeto Java → BeanSerializer → JsonGenerator → OutputStream

Desserialização (JSON → Java):
  InputStream → JsonParser → BeanDeserializer → Objeto Java
```

---

## 2. Configuração e Dependências

### 2.1 Dependências Maven — Spring Boot 3

```xml
<!-- O starter web já inclui jackson-databind, jackson-core e jackson-annotations -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Java 8+ Date/Time — incluído via spring-boot-starter-web no SB 3 -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>

<!-- Optional, OptionalInt, etc. -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
</dependency>

<!-- Construtores sem @JsonCreator (requer -parameters no compilador) -->
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-parameter-names</artifactId>
</dependency>

<!-- Performance — substitui afterburner em Java 11+ -->
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-blackbird</artifactId>
</dependency>

<!-- YAML como formato alternativo (ex.: leitura de config files) -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>
```

```xml
<!-- pom.xml: habilitar -parameters para ParameterNamesModule funcionar -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <parameters>true</parameters> <!-- obrigatório para records e @JsonCreator implícito -->
    </configuration>
</plugin>
```

### 2.2 Dependências Maven — Spring Boot 4 / Jackson 3

**`[Jackson 3 / SB4]`**

```xml
<!-- Spring Boot 4 gerencia a versão do Jackson 3 automaticamente -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Jackson 3 unifica jsr310 e jdk8 num único módulo -->
<dependency>
    <groupId>tools.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>

<!-- Blackbird é o padrão de performance no Jackson 3 -->
<dependency>
    <groupId>tools.jackson.module</groupId>
    <artifactId>jackson-module-blackbird</artifactId>
</dependency>
```

### 2.3 ObjectMapper — Configuração Programática

```java
// ─── Jackson 2 — Builder via Jackson2ObjectMapperBuilder (Spring) ─────────────
@Bean
public ObjectMapper objectMapper() {
    return Jackson2ObjectMapperBuilder.json()
        .featuresToDisable(
            SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,
            DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES
        )
        .featuresToEnable(
            MapperFeature.ACCEPT_CASE_INSENSITIVE_PROPERTIES
        )
        .serializationInclusion(JsonInclude.Include.NON_NULL)
        .modules(new JavaTimeModule(), new ParameterNamesModule(), new Jdk8Module())
        .build();
}
```

```java
// ─── Jackson 3 — JsonMapper.builder() ────────────────────────────────────────
// [Jackson 3 / SB4]
@Bean
public ObjectMapper objectMapper() {
    return JsonMapper.builder()
        .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .enable(MapperFeature.ACCEPT_CASE_INSENSITIVE_PROPERTIES)
        .serializationInclusion(JsonInclude.Include.NON_NULL)
        .addModule(new JavaTimeModule())
        .build();
}
```

### 2.4 application.yml — Propriedades Jackson no Spring Boot

```yaml
spring:
  jackson:
    # Nomenclatura de propriedades: SNAKE_CASE, LOWER_CAMEL_CASE, etc.
    property-naming-strategy: SNAKE_CASE

    # Inclusão de valores: NON_NULL, NON_EMPTY, NON_ABSENT, ALWAYS
    default-property-inclusion: non_null

    # Serialização de datas como string ISO-8601 (não timestamp numérico)
    serialization:
      write-dates-as-timestamps: false
      indent-output: false       # true apenas em desenvolvimento

    # Desserialização
    deserialization:
      fail-on-unknown-properties: false   # Spring Boot já desabilita por padrão
      accept-single-value-as-array: true  # "foo" → ["foo"] automaticamente

    # Locale e timezone
    locale: pt_BR
    time-zone: America/Sao_Paulo

    # Features de parser/gerador
    parser:
      allow-comments: true            # permite // e /* */ no JSON
    generator:
      write-bigdecimal-as-plain: true # 1E+2 → 100
```

---

## 3. Anotações Essenciais

### 3.1 `@JsonProperty` — Renomear Campos

```java
public class ProdutoDTO {

    @JsonProperty("product_name")   // serializa/desserializa como "product_name"
    private String nome;

    @JsonProperty(value = "preco_unitario", required = true) // campo obrigatório na leitura
    private BigDecimal preco;

    @JsonProperty(access = JsonProperty.Access.READ_ONLY) // ignorado na desserialização
    private Instant criadoEm;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY) // ignorado na serialização
    private String senha;
}
```

**Serialização:**
```json
{
  "product_name": "Cadeira Ergonômica",
  "preco_unitario": 899.90,
  "criado_em": "2025-03-17T10:00:00Z"
}
```

**Desserialização — `senha` é lida mas `criadoEm` é ignorada:**
```json
{ "product_name": "Cadeira", "preco_unitario": 499.00, "senha": "abc123" }
```

### 3.2 `@JsonIgnore` e `@JsonIgnoreProperties`

```java
// ─── @JsonIgnore: campo específico ────────────────────────────────────────────
public class Usuario {

    private String nome;

    @JsonIgnore                 // nunca serializa nem desserializa
    private String senhaHash;
}

// ─── @JsonIgnoreProperties: múltiplos campos na classe ───────────────────────
@JsonIgnoreProperties({"senhaHash", "salt", "tentativasLogin"})
public class UsuarioPublicoDTO {
    private Long id;
    private String nome;
    private String email;
    // senhaHash, salt, tentativasLogin ignorados mesmo que existam no JSON de entrada
}

// ─── ignoreUnknown: tolerante a campos extras no JSON ─────────────────────────
@JsonIgnoreProperties(ignoreUnknown = true)
public class EventoExterno {
    private String tipo;
    private String payload;
    // campos desconhecidos do JSON de entrada são silenciosamente ignorados
}
```

### 3.3 `@JsonAlias` — Múltiplos Nomes na Leitura

```java
public class ClienteDTO {

    @JsonAlias({"client_name", "clientName", "nome_cliente"})
    private String nome; // aceita qualquer um dos nomes na leitura; serializa como "nome"

    @JsonProperty("email_address")
    @JsonAlias({"email", "e_mail"})
    private String email; // serializa como "email_address"; aceita "email" ou "e_mail" na leitura
}
```

**Entrada aceita:**
```json
{ "client_name": "Ana Lima", "e_mail": "ana@email.com" }
{ "clientName": "Ana Lima", "email": "ana@email.com"   }
{ "nome_cliente": "Ana Lima", "email_address": "ana@email.com" }
```

**Saída serializada:**
```json
{ "nome": "Ana Lima", "email_address": "ana@email.com" }
```

### 3.4 `@JsonInclude` — Controlar Campos Nulos e Vazios

```java
// ─── Nível de classe ──────────────────────────────────────────────────────────
@JsonInclude(JsonInclude.Include.NON_NULL)   // omite campos null
public class RespostaDTO {
    private String nome;
    private String descricao; // null → omitido
    private List<String> tags; // null → omitido; [] → incluído
}

// ─── Nível de campo — configuração granular ───────────────────────────────────
public class ProdutoDTO {

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String descricao;     // omite se null

    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    private List<String> tags;    // omite se null OU vazio ([])

    @JsonInclude(JsonInclude.Include.NON_ABSENT)
    private Optional<String> sku; // omite se null OU Optional.empty()

    @JsonInclude(JsonInclude.Include.NON_DEFAULT)
    private int estoque;          // omite se igual ao valor default do tipo (0 para int)
}
```

**Exemplo com `NON_NULL`:**
```java
ProdutoDTO p = new ProdutoDTO();
p.setNome("Teclado");
p.setDescricao(null);
p.setTags(List.of());
```

```json
// NON_NULL em descricao:
{ "nome": "Teclado", "tags": [] }

// NON_EMPTY em tags (lista vazia omitida):
{ "nome": "Teclado" }
```

### 3.5 `@JsonFormat` — Formatação de Datas e Números

```java
public class PedidoDTO {

    @JsonFormat(shape = JsonFormat.Shape.STRING,
                pattern = "dd/MM/yyyy",
                locale = "pt-BR",
                timezone = "America/Sao_Paulo")
    private LocalDate dataPedido;

    @JsonFormat(shape = JsonFormat.Shape.STRING,
                pattern = "dd/MM/yyyy HH:mm:ss",
                timezone = "America/Sao_Paulo")
    private LocalDateTime criadoEm;

    @JsonFormat(shape = JsonFormat.Shape.STRING,
                pattern = "yyyy-MM-dd'T'HH:mm:ssXXX") // ISO com offset
    private ZonedDateTime expiracaoToken;

    @JsonFormat(shape = JsonFormat.Shape.NUMBER)  // serializa enum como índice numérico
    private StatusPedido status;

    @JsonFormat(shape = JsonFormat.Shape.STRING)  // serializa enum como nome (padrão)
    private TipoPagamento tipoPagamento;
}
```

**Resultado:**
```json
{
  "data_pedido": "17/03/2025",
  "criado_em": "17/03/2025 10:30:00",
  "expiracao_token": "2025-03-17T10:30:00-03:00",
  "status": 1,
  "tipo_pagamento": "CARTAO_CREDITO"
}
```

### 3.6 `@JsonCreator` e `@JsonValue` — Controlar Construção e Serialização

```java
// ─── @JsonValue: controla como o objeto é serializado ────────────────────────
public enum StatusPedido {
    PENDENTE("P"), APROVADO("A"), CANCELADO("C");

    private final String codigo;

    StatusPedido(String codigo) { this.codigo = codigo; }

    @JsonValue                          // serializa como o codigo, não como o nome
    public String getCodigo() { return codigo; }

    @JsonCreator                        // desserializa a partir do codigo
    public static StatusPedido fromCodigo(String codigo) {
        return Arrays.stream(values())
            .filter(s -> s.codigo.equals(codigo))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Status inválido: " + codigo));
    }
}
```

**JSON serializado:** `"A"` em vez de `"APROVADO"`

```java
// ─── @JsonCreator em construtor — imutável sem setters ───────────────────────
public class Coordenada {
    private final double lat;
    private final double lon;

    @JsonCreator
    public Coordenada(
            @JsonProperty("latitude")  double lat,
            @JsonProperty("longitude") double lon) {
        this.lat = lat;
        this.lon = lon;
    }
    // getters omitidos — sem setters, objeto imutável
}
```

**Entrada:**
```json
{ "latitude": -23.5505, "longitude": -46.6333 }
```

```java
// ─── @JsonCreator com factory method ─────────────────────────────────────────
public class Moeda {
    private final String codigo;  // "BRL", "USD"

    private Moeda(String codigo) { this.codigo = codigo; }

    @JsonCreator
    public static Moeda de(String codigo) {
        return new Moeda(codigo.toUpperCase());
    }

    @JsonValue
    public String getCodigo() { return codigo; }
}
```

### 3.7 `@JsonUnwrapped` — Achatar Objetos Aninhados

```java
public class Endereco {
    private String rua;
    private String cidade;
    private String cep;
    // getters/setters
}

public class ClienteDTO {
    private String nome;

    @JsonUnwrapped                          // achata os campos de Endereco no nível raiz
    private Endereco endereco;

    @JsonUnwrapped(prefix = "entrega_")    // com prefixo para evitar colisão
    private Endereco enderecoEntrega;
}
```

**Resultado — sem aninhamento:**
```json
{
  "nome": "João Silva",
  "rua": "Rua das Flores, 100",
  "cidade": "São Paulo",
  "cep": "01310-100",
  "entrega_rua": "Av. Paulista, 1000",
  "entrega_cidade": "São Paulo",
  "entrega_cep": "01311-000"
}
```

### 3.8 `@JsonAnyGetter` e `@JsonAnySetter` — Propriedades Dinâmicas

```java
public class Configuracao {
    private String nome;
    private Map<String, Object> propriedades = new HashMap<>();

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }

    @JsonAnyGetter                          // serializa o Map como propriedades no nível raiz
    public Map<String, Object> getPropriedades() {
        return propriedades;
    }

    @JsonAnySetter                          // campos desconhecidos vão para o Map
    public void setPropriedade(String chave, Object valor) {
        propriedades.put(chave, valor);
    }
}
```

**Entrada:**
```json
{
  "nome": "Config Prod",
  "timeout": 30,
  "max_conexoes": 100,
  "debug": false
}
```

**Objeto resultante:** `nome="Config Prod"`, `propriedades={"timeout":30, "max_conexoes":100, "debug":false}`

**Saída serializada:**
```json
{
  "nome": "Config Prod",
  "timeout": 30,
  "max_conexoes": 100,
  "debug": false
}
```

### 3.9 `@JsonRawValue` — JSON Embutido sem Escape

```java
public class RespostaAPI {
    private String status;

    @JsonRawValue                           // inclui como JSON literal — sem escapar aspas
    private String dadosJson;              // ex.: {"chave":"valor"}
}

RespostaAPI r = new RespostaAPI();
r.setStatus("ok");
r.setDadosJson("{\"total\": 42, \"itens\": [1, 2, 3]}");
```

**Saída:**
```json
{
  "status": "ok",
  "dados_json": { "total": 42, "itens": [1, 2, 3] }
}
```

> ⚠️ **Segurança:** nunca use `@JsonRawValue` com dados vindos de usuários sem sanitização — isso pode injetar JSON malformado na resposta.

### 3.10 `@JsonRootName` — Envelope Raiz

```java
@JsonRootName("produto")
public class ProdutoDTO {
    private String nome;
    private BigDecimal preco;
}
```

```java
// Requer habilitar WRAP_ROOT_VALUE no ObjectMapper
ObjectMapper mapper = new ObjectMapper();
mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
```

**Saída:**
```json
{
  "produto": {
    "nome": "Mesa de Escritório",
    "preco": 1299.90
  }
}
```

### 3.11 `@JsonView` — Visões Parciais do Objeto

```java
// ─── Definição das views ──────────────────────────────────────────────────────
public class Views {
    public static class Publico {}
    public static class Resumido extends Publico {}
    public static class Completo extends Resumido {}
    public static class Admin extends Completo {}
}

// ─── Entidade com views ───────────────────────────────────────────────────────
public class Funcionario {

    @JsonView(Views.Publico.class)
    private Long id;

    @JsonView(Views.Publico.class)
    private String nome;

    @JsonView(Views.Resumido.class)
    private String departamento;

    @JsonView(Views.Completo.class)
    private BigDecimal salario;

    @JsonView(Views.Admin.class)
    private String cpf;

    private String senhaHash; // sem @JsonView → nunca aparece (padrão Jackson 3)
                              // Jackson 2: aparece quando nenhuma view é ativa
}
```

**View `Publico`:**
```json
{ "id": 1, "nome": "Maria Oliveira" }
```

**View `Resumido`:**
```json
{ "id": 1, "nome": "Maria Oliveira", "departamento": "Engenharia" }
```

**View `Admin`:**
```json
{ "id": 1, "nome": "Maria Oliveira", "departamento": "Engenharia", "salario": 12000.00, "cpf": "123.456.789-00" }
```

### 3.12 `@JsonManagedReference` e `@JsonBackReference` — Referências Bidirecionais

```java
// Problema: Pedido → List<Item> → Pedido → ... (loop infinito)

public class Pedido {
    private Long id;

    @JsonManagedReference               // lado "pai" — serializado normalmente
    private List<ItemPedido> itens;
}

public class ItemPedido {
    private Long id;
    private String descricao;

    @JsonBackReference                  // lado "filho" — omitido na serialização
    private Pedido pedido;
}
```

**Saída — sem loop:**
```json
{
  "id": 1,
  "itens": [
    { "id": 10, "descricao": "Produto A" },
    { "id": 11, "descricao": "Produto B" }
  ]
}
```

> **Alternativa recomendada:** use DTOs sem referências circulares em vez de `@JsonManagedReference`. DTOs são mais explícitos e seguros em produção.

---

## 4. Serialização e Desserialização Customizadas

### 4.1 Serializer Customizado

```java
// Serializa CPF no formato "###.###.###-##"
public class CpfSerializer extends StdSerializer<String> {

    public CpfSerializer() { super(String.class); }

    @Override
    public void serialize(String cpf, JsonGenerator gen, SerializerProvider provider)
            throws IOException {
        if (cpf == null) {
            gen.writeNull();
            return;
        }
        // "12345678900" → "123.456.789-00"
        String formatado = cpf.replaceAll("(\\d{3})(\\d{3})(\\d{3})(\\d{2})",
                                          "$1.$2.$3-$4");
        gen.writeString(formatado);
    }
}
```

### 4.2 Deserializer Customizado

```java
// Aceita CPF com ou sem formatação e normaliza para apenas dígitos
public class CpfDeserializer extends StdDeserializer<String> {

    public CpfDeserializer() { super(String.class); }

    @Override
    public String deserialize(JsonParser p, DeserializationContext ctx)
            throws IOException {
        String valor = p.getText();
        if (valor == null) return null;

        String soDigitos = valor.replaceAll("[^\\d]", "");
        if (soDigitos.length() != 11) {
            throw new JsonParseException(p, "CPF inválido: " + valor);
        }
        return soDigitos;
    }
}
```

### 4.3 `@JsonSerialize` e `@JsonDeserialize` — Aplicação Pontual

```java
public class PessoaDTO {

    @JsonSerialize(using = CpfSerializer.class)
    @JsonDeserialize(using = CpfDeserializer.class)
    private String cpf;

    // Usando converters para tipos específicos
    @JsonSerialize(using = MoneySerializer.class)
    private BigDecimal salario;

    // Serialização de coleções com serializer de elementos
    @JsonSerialize(contentUsing = LocalDateSerializer.class)
    @JsonDeserialize(contentUsing = LocalDateDeserializer.class)
    private List<LocalDate> datasImportantes;

    // Serialização de chaves de Map
    @JsonSerialize(keyUsing = StatusKeySerializer.class)
    @JsonDeserialize(keyUsing = StatusKeyDeserializer.class)
    private Map<StatusPedido, Integer> contagemPorStatus;
}
```

### 4.4 `StdConverter` — Conversão Simples

```java
// Mais simples que StdSerializer quando a conversão é tipo A → tipo B
public class CentavosParaReaisConverter extends StdConverter<Long, BigDecimal> {
    @Override
    public BigDecimal convert(Long centavos) {
        return centavos == null ? null :
               BigDecimal.valueOf(centavos).movePointLeft(2);
    }
}

public class ReaisParaCentavosConverter extends StdConverter<BigDecimal, Long> {
    @Override
    public Long convert(BigDecimal reais) {
        return reais == null ? null :
               reais.movePointRight(2).longValueExact();
    }
}

// Uso:
public class PagamentoDTO {
    @JsonSerialize(converter = ReaisParaCentavosConverter.class)
    @JsonDeserialize(converter = CentavosParaReaisConverter.class)
    private BigDecimal valor; // internamente em reais; no JSON em centavos
}
```

**Entrada JSON:** `{ "valor": 15099 }` (centavos)
**Valor no objeto:** `BigDecimal("150.99")`
**Saída JSON:** `{ "valor": 15099 }`

### 4.5 Boolean como 0/1 — Integração com APIs Legadas

Muitos sistemas legados (bancos, ERPs, mainframes) representam booleanos como `0` e `1` em vez de `true`/`false`. Jackson permite tratar essa conversão de duas formas: **por campo** (anotação) ou **global** (módulo registrado no Spring Boot).

#### Serializer e Deserializer

```java
// Serializa: true → 1, false → 0
public class BooleanToIntSerializer extends JsonSerializer<Boolean> {

    @Override
    public void serialize(Boolean value, JsonGenerator gen, SerializerProvider provider)
            throws IOException {
        gen.writeNumber(value != null && value ? 1 : 0);
    }
}
```

```java
// Desserializa: aceita 1/0, true/false e variações
public class IntToBooleanDeserializer extends JsonDeserializer<Boolean> {

    @Override
    public Boolean deserialize(JsonParser p, DeserializationContext ctx)
            throws IOException {
        return switch (p.getText().trim()) {
            case "1", "true",  "TRUE"  -> true;
            case "0", "false", "FALSE" -> false;
            default -> throw new JsonParseException(
                p, "Valor inválido para boolean: " + p.getText()
            );
        };
    }
}
```

#### Uso por Campo (granular)

```java
public record ProdutoDTO(
    String nome,

    @JsonSerialize(using = BooleanToIntSerializer.class)
    @JsonDeserialize(using = IntToBooleanDeserializer.class)
    boolean ativo,

    @JsonSerialize(using = BooleanToIntSerializer.class)
    @JsonDeserialize(using = IntToBooleanDeserializer.class)
    boolean emEstoque
) {}
```

#### Registro Global via Spring Boot (recomendado)

Spring Boot detecta automaticamente qualquer `@Bean` do tipo `com.fasterxml.jackson.databind.Module` e o registra no `ObjectMapper` global:

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Module booleanAsIntModule() {
        var module = new SimpleModule("BooleanAsInt");

        module.addSerializer(Boolean.class,   new BooleanToIntSerializer());
        module.addSerializer(boolean.class,   new BooleanToIntSerializer()); // primitivo
        module.addDeserializer(Boolean.class, new IntToBooleanDeserializer());
        module.addDeserializer(boolean.class, new IntToBooleanDeserializer());

        return module;
    }
}
```

#### Resultado

```json
// Serialização: true → 1, false → 0
{ "nome": "Notebook", "ativo": 1, "emEstoque": 0 }

// Desserialização: aceita 1/0 e true/false como entrada
{ "nome": "Notebook", "ativo": 1, "emEstoque": 0 }       → ativo=true, emEstoque=false
{ "nome": "Notebook", "ativo": true, "emEstoque": false } → também funciona
```

| Abordagem | Quando usar |
|---|---|
| **Por campo** | Apenas alguns campos específicos precisam do comportamento (integração pontual com legado) |
| **Global** | Toda a aplicação conversa com um sistema que usa `0`/`1` como convenção booleana |

#### Contraparte no Frontend — JavaScript/TypeScript

Quando a API serializa booleanos como `0`/`1`, o frontend precisa tratar a conversão na ida e na volta. Duas abordagens nativas sem dependências externas:

**`JSON.parse` com reviver — desserialização (entrada):**

```typescript
// Converte campos 0/1 para boolean na leitura do JSON
function parseWithBooleans<T>(
  json: string,
  booleanFields: Set<string>
): T {
  return JSON.parse(json, (key, value) => {
    if (booleanFields.has(key) && (value === 0 || value === 1)) {
      return value === 1;
    }
    return value;
  });
}

// Uso:
const produto = parseWithBooleans<Produto>(
  '{"nome":"Notebook","ativo":1,"emEstoque":0}',
  new Set(["ativo", "emEstoque"])
);
// → { nome: "Notebook", ativo: true, emEstoque: false }
```

**`JSON.stringify` com replacer — serialização (saída):**

```typescript
// Converte campos boolean para 0/1 na escrita do JSON
function stringifyWithBoolAsInt<T extends object>(
  value: T,
  booleanFields: Set<string>
): string {
  return JSON.stringify(value, (key, val) => {
    if (booleanFields.has(key) && typeof val === "boolean") {
      return val ? 1 : 0;
    }
    return val;
  });
}

// Uso:
const json = stringifyWithBoolAsInt(
  { nome: "Notebook", ativo: true, emEstoque: false },
  new Set(["ativo", "emEstoque"])
);
// → '{"nome":"Notebook","ativo":1,"emEstoque":0}'
```

**Fetch wrapper combinando ambas as conversões:**

```typescript
const BOOL_FIELDS = new Set(["ativo", "emEstoque", "habilitado"]);

const BOOL_REPLACER = (key: string, value: unknown): unknown =>
  BOOL_FIELDS.has(key) && typeof value === "boolean" ? (value ? 1 : 0) : value;

async function fetchApi<T>(url: string, options: RequestInit = {}): Promise<T> {
  const res = await fetch(url, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);

  const text = await res.text();
  return JSON.parse(text, (key, value) => {
    if (BOOL_FIELDS.has(key) && (value === 0 || value === 1)) {
      return value === 1;
    }
    return value;
  });
}

// POST com conversão automática na ida e na volta
async function postApi<T>(url: string, body: unknown): Promise<T> {
  return fetchApi<T>(url, {
    method: "POST",
    body: JSON.stringify(body, BOOL_REPLACER),
  });
}

// Uso:
const produto = await fetchApi<Produto>("/api/produtos/1");
// → { nome: "Notebook", ativo: true, emEstoque: false }

await postApi("/api/produtos", { nome: "Notebook", ativo: true, emEstoque: false });
// Corpo enviado: { "nome": "Notebook", "ativo": 1, "emEstoque": 0 }
```

| Jackson (Java) | JavaScript/TypeScript nativo |
|---|---|
| `@JsonSerialize` por campo | `JSON.stringify` com replacer |
| `@JsonDeserialize` por campo | `JSON.parse` com reviver |
| `SimpleModule` global no Boot | Fetch wrapper com replacer + reviver |

---

## 5. Polimorfismo — `@JsonTypeInfo` e `@JsonSubTypes`

### 5.1 Configuração Básica

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,        // discriminador baseado em nome
    include = JsonTypeInfo.As.PROPERTY, // como propriedade no JSON
    property = "tipo"                   // nome da propriedade discriminadora
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = PagamentoCartao.class,     name = "cartao"),
    @JsonSubTypes.Type(value = PagamentoBoleto.class,     name = "boleto"),
    @JsonSubTypes.Type(value = PagamentoPix.class,        name = "pix")
})
public abstract class Pagamento {
    private BigDecimal valor;
    private LocalDateTime dataPagamento;
    // getters/setters
}

public class PagamentoCartao extends Pagamento {
    private String ultimos4Digitos;
    private int parcelas;
}

public class PagamentoPix extends Pagamento {
    private String chave;
    private String txId;
}
```

**JSON de entrada/saída:**
```json
{ "tipo": "pix", "valor": 150.00, "chave": "email@exemplo.com", "tx_id": "xyz123" }
{ "tipo": "cartao", "valor": 500.00, "ultimos_4_digitos": "1234", "parcelas": 3 }
```

**Desserialização polimórfica:**
```java
Pagamento p = mapper.readValue(json, Pagamento.class);
// p será instância de PagamentoPix ou PagamentoCartao conforme "tipo"
```

### 5.2 Tipos de Inclusão de Tipo

```java
// PROPERTY — campo separado (mais comum)
@JsonTypeInfo(use = Id.NAME, include = As.PROPERTY, property = "tipo")
// → { "tipo": "pix", "valor": 100 }

// WRAPPER_OBJECT — envelope externo
@JsonTypeInfo(use = Id.NAME, include = As.WRAPPER_OBJECT)
// → { "pix": { "valor": 100 } }

// EXISTING_PROPERTY — usa um campo existente como discriminador
@JsonTypeInfo(use = Id.NAME, include = As.EXISTING_PROPERTY, property = "canal")
// → { "canal": "pix", "valor": 100 }  (campo "canal" já existe na classe)

// CLASS — nome completo da classe (evite em APIs públicas — expõe estrutura interna)
@JsonTypeInfo(use = Id.CLASS)
// → { "@class": "com.exemplo.PagamentoPix", "valor": 100 }
```

### 5.3 Polimorfismo em Coleções

```java
List<Pagamento> pagamentos = mapper.readValue(
    "[{\"tipo\":\"pix\",\"valor\":50},{\"tipo\":\"boleto\",\"valor\":200}]",
    new TypeReference<List<Pagamento>>() {}
);
// → [PagamentoPix, PagamentoBoleto]
```

**`[Jackson 3 / SB4]` — Sealed Classes como Alternativa:**
```java
// Jackson 3 reconhece sealed classes automaticamente para polimorfismo
@JsonTypeInfo(use = Id.NAME, property = "tipo")
public sealed interface Notificacao
    permits NotificacaoEmail, NotificacaoSMS, NotificacaoPush {}

public record NotificacaoEmail(String destinatario, String assunto) implements Notificacao {}
public record NotificacaoSMS(String telefone, String mensagem) implements Notificacao {}
```

---

## 6. Módulos Jackson

### 6.1 `JavaTimeModule` — Java 8+ Date/Time

```java
// Registro (Spring Boot faz automaticamente se jackson-datatype-jsr310 está no classpath)
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS); // ISO-8601, não epoch
```

**Tipos suportados e formatos padrão:**

| Tipo Java | JSON padrão | Com `WRITE_DATES_AS_TIMESTAMPS=false` |
|---|---|---|
| `LocalDate` | `[2025,3,17]` | `"2025-03-17"` |
| `LocalDateTime` | `[2025,3,17,10,30,0]` | `"2025-03-17T10:30:00"` |
| `ZonedDateTime` | `1742205000.0` | `"2025-03-17T10:30:00-03:00"` |
| `Instant` | `1742205000.0` | `"2025-03-17T13:30:00Z"` |
| `Duration` | `3600.0` | `"PT1H"` |
| `YearMonth` | `[2025,3]` | `"2025-03"` |

```java
// Formatação customizada por campo
public class EventoDTO {
    @JsonFormat(pattern = "dd/MM/yyyy")
    private LocalDate dataEvento;

    @JsonFormat(pattern = "dd/MM/yyyy HH:mm", timezone = "America/Sao_Paulo")
    private LocalDateTime inicio;

    @JsonFormat(shape = JsonFormat.Shape.NUMBER)  // força epoch mesmo com módulo ativo
    private Instant timestampAuditoria;
}
```

### 6.2 `ParameterNamesModule` — Sem `@JsonCreator`

```java
// Sem o módulo: Jackson não consegue descobrir nomes de parâmetros em runtime
// Com o módulo + -parameters: funciona automaticamente
public class ProdutoDTO {
    private final String nome;
    private final BigDecimal preco;

    // @JsonCreator seria obrigatório sem ParameterNamesModule
    public ProdutoDTO(String nome, BigDecimal preco) {
        this.nome = nome;
        this.preco = preco;
    }
}
```

> ⚠️ **Pré-requisito:** compilar com `-parameters` (ver seção 2.1).

### 6.3 `JavaOptionalModule` / `Jdk8Module`

```java
// Com Jdk8Module: Optional é tratado como valor presente ou ausente
public class RespostaDTO {
    private Optional<String> descricao; // presente → valor; vazio → campo omitido/null
}

// Optional<String> com valor "foo":
// { "descricao": "foo" }

// Optional.empty():
// { "descricao": null }  (ou omitido com NON_ABSENT)
```

### 6.4 `AfterburnerModule` / `BlackbirdModule` — Performance

```java
// Jackson 2: AfterburnerModule usa geração de bytecode (ASM)
// Jackson 2.12+ / Java 11+: BlackbirdModule usa MethodHandles (sem geração de bytecode)

// Ambos otimizam acesso a campos/métodos sem reflexão direta
// Ganho típico: 20-40% de throughput em serialização de grandes volumes

// Jackson 2:
mapper.registerModule(new AfterburnerModule()); // ou BlackbirdModule

// Jackson 3: BlackbirdModule é o padrão recomendado
// [Jackson 3 / SB4]
mapper.registerModule(new BlackbirdModule());
```

---

## 7. Java Records e Jackson

### 7.1 Suporte Nativo a Records (Jackson 2.12+)

```java
// Records são suportados nativamente desde Jackson 2.12
// O construtor canônico é detectado automaticamente como @JsonCreator
public record ProdutoDTO(
    Long id,
    String nome,
    BigDecimal preco,
    String categoria
) {}
```

**Entrada:** `{ "id": 1, "nome": "Monitor", "preco": 1500.00, "categoria": "Informatica" }`
**Saída:** `{ "id": 1, "nome": "Monitor", "preco": 1500.00, "categoria": "Informatica" }`

### 7.2 Records com Anotações

```java
public record ClienteDTO(

    @JsonProperty("customer_id")
    Long id,

    @JsonProperty("full_name")
    String nomeCompleto,

    @JsonIgnore
    String senhaHash,

    @JsonFormat(pattern = "dd/MM/yyyy")
    LocalDate dataNascimento,

    @JsonInclude(JsonInclude.Include.NON_NULL)
    String telefone

) {}
```

**Saída (telefone=null):**
```json
{
  "customer_id": 42,
  "full_name": "Carlos Mendes",
  "data_nascimento": "15/08/1990"
}
```

### 7.3 Records como DTOs — Padrão Recomendado

```java
// ─── Padrão: record por operação, não por entidade ─────────────────────────────

// Criação
public record CriarProdutoRequest(
    @NotBlank String nome,
    @Positive BigDecimal preco,
    @NotNull String categoriaId
) {}

// Resposta
public record ProdutoResponse(
    Long id,
    String nome,
    BigDecimal preco,
    String categoria,
    @JsonFormat(pattern = "dd/MM/yyyy HH:mm") LocalDateTime criadoEm
) {
    // Fábrica estática para construir a partir da entidade
    public static ProdutoResponse de(Produto p) {
        return new ProdutoResponse(p.getId(), p.getNome(), p.getPreco(),
                                   p.getCategoria().getNome(), p.getCriadoEm());
    }
}

// Atualização parcial — campos Optional indicam "não enviado"
public record AtualizarProdutoRequest(
    Optional<String> nome,
    Optional<BigDecimal> preco
) {}
```

---

## 8. `JsonNode` — Manipulação de JSON Dinâmico

### 8.1 Leitura com `JsonNode`

```java
String json = """
    {
      "pedido": {
        "id": 123,
        "itens": [
          {"produto": "Mouse", "quantidade": 2},
          {"produto": "Teclado", "quantidade": 1}
        ],
        "total": 350.50
      }
    }
    """;

ObjectMapper mapper = new ObjectMapper();
JsonNode root = mapper.readTree(json);

// Navegação
int id = root.path("pedido").path("id").asInt();               // 123
double total = root.path("pedido").path("total").asDouble();   // 350.50
String produto = root.at("/pedido/itens/0/produto").asText();  // "Mouse" (JSON Pointer)

// Verificações
boolean temDesconto = root.path("pedido").has("desconto");     // false
boolean idNumerico  = root.path("pedido").path("id").isNumber(); // true

// Iteração sobre array
JsonNode itens = root.path("pedido").path("itens");
for (JsonNode item : itens) {
    String p = item.path("produto").asText();
    int q = item.path("quantidade").asInt();
    System.out.println(p + " x " + q);
}
// Mouse x 2
// Teclado x 1
```

### 8.2 Construção de JSON Dinâmico

```java
ObjectMapper mapper = new ObjectMapper();
ObjectNode root = mapper.createObjectNode();

root.put("status", "sucesso");
root.put("codigo", 200);
root.put("timestamp", Instant.now().toString());

ObjectNode dados = root.putObject("dados");
dados.put("id", 42);
dados.put("nome", "Produto X");

ArrayNode tags = dados.putArray("tags");
tags.add("eletrônico").add("oferta").add("novo");

String json = mapper.writeValueAsString(root);
```

**Saída:**
```json
{
  "status": "sucesso",
  "codigo": 200,
  "timestamp": "2025-03-17T13:00:00Z",
  "dados": {
    "id": 42,
    "nome": "Produto X",
    "tags": ["eletrônico", "oferta", "novo"]
  }
}
```

### 8.3 `JsonNode` vs `Map` — Quando Usar Cada Um

| Critério | `JsonNode` | `Map<String, Object>` |
|---|---|---|
| Estrutura desconhecida em runtime | ✅ Ideal | ✅ Funciona |
| Navegação aninhada | ✅ `.path()`, `.at()` | ⚠️ Cast manual necessário |
| Modificação do JSON | ✅ `ObjectNode.put()` | ✅ `map.put()` |
| Conversão para tipo forte | ✅ `mapper.treeToValue()` | ✅ `mapper.convertValue()` |
| Performance com grandes volumes | ✅ Melhor | ⚠️ Overhead de boxing |
| Interoperabilidade com código existente | ⚠️ Jackson-específico | ✅ Java padrão |

---

## 9. Integração com Spring Boot

### 9.1 Auto-configuração do Jackson

O Spring Boot configura automaticamente o `ObjectMapper` via `JacksonAutoConfiguration` e `JacksonProperties`. As configurações padrão do Spring Boot para Jackson são:

- `FAIL_ON_UNKNOWN_PROPERTIES` = `false` (tolera campos extras)
- `MapperFeature.DEFAULT_VIEW_INCLUSION` = `true` (Jackson 2; muda para `false` no Jackson 3)
- Módulos registrados automaticamente se presentes no classpath: `JavaTimeModule`, `Jdk8Module`, `ParameterNamesModule`
- Naming strategy: camelCase (padrão Jackson)

### 9.2 `Jackson2ObjectMapperBuilderCustomizer`

```java
// Personalização sem substituir o bean principal — abordagem recomendada no Spring Boot
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jacksonCustomizer() {
        return builder -> builder
            // Datas como ISO-8601
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            // Ignora campos desconhecidos
            .featuresToDisable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
            // Snake_case para todas as propriedades
            .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
            // Omitir nulls globalmente
            .serializationInclusion(JsonInclude.Include.NON_NULL)
            // Aceitar número como boolean (ex.: 0/1)
            .featuresToEnable(MapperFeature.ALLOW_COERCION_OF_SCALARS)
            // Registrar serializer global para BigDecimal
            .serializerByType(BigDecimal.class, new BigDecimalSerializer());
    }
}
```

```java
// Serializer para BigDecimal com precisão controlada
public class BigDecimalSerializer extends StdSerializer<BigDecimal> {
    public BigDecimalSerializer() { super(BigDecimal.class); }

    @Override
    public void serialize(BigDecimal value, JsonGenerator gen, SerializerProvider provider)
            throws IOException {
        gen.writeNumber(value.setScale(2, RoundingMode.HALF_UP).toPlainString());
    }
}
```

### 9.3 `@JsonComponent` — Registrar Customizações como Beans

```java
// Spring Boot detecta @JsonComponent e registra automaticamente no ObjectMapper
@JsonComponent
public class StatusPedidoJsonComponent {

    public static class Serializer extends JsonSerializer<StatusPedido> {
        @Override
        public void serialize(StatusPedido status, JsonGenerator gen,
                              SerializerProvider provider) throws IOException {
            gen.writeStartObject();
            gen.writeStringField("codigo", status.getCodigo());
            gen.writeStringField("descricao", status.getDescricao());
            gen.writeEndObject();
        }
    }

    public static class Deserializer extends JsonDeserializer<StatusPedido> {
        @Override
        public StatusPedido deserialize(JsonParser p, DeserializationContext ctx)
                throws IOException {
            JsonNode node = p.getCodec().readTree(p);
            String codigo = node.get("codigo").asText();
            return StatusPedido.fromCodigo(codigo);
        }
    }
}
```

**Entrada:** `{ "status": { "codigo": "A", "descricao": "Aprovado" } }`
**Saída:** `{ "status": { "codigo": "A", "descricao": "Aprovado" } }`

### 9.4 `@JsonView` em Controllers REST

```java
@RestController
@RequestMapping("/funcionarios")
public class FuncionarioController {

    // View pública — retorna apenas id e nome
    @GetMapping
    @JsonView(Views.Publico.class)
    public List<Funcionario> listar() {
        return service.listarTodos();
    }

    // View completa — retorna todos os campos da view Completo
    @GetMapping("/{id}")
    @JsonView(Views.Completo.class)
    public Funcionario buscar(@PathVariable Long id) {
        return service.buscarPorId(id);
    }

    // View admin — apenas para usuários com role ADMIN
    @GetMapping("/{id}/admin")
    @PreAuthorize("hasRole('ADMIN')")
    @JsonView(Views.Admin.class)
    public Funcionario buscarAdmin(@PathVariable Long id) {
        return service.buscarPorId(id);
    }
}
```

### 9.5 `MixIn` — Anotações sem Modificar Terceiros

```java
// Cenário: adicionar anotações Jackson a classe de biblioteca de terceiros
// sem modificar o código-fonte original

// Classe de terceiro (não pode ser modificada)
public class EnderecoExterno {
    public String logradouro;
    public String numero;
    public String complemento;
    public String cep;
    public String municipio;
    public String uf;
}

// MixIn: interface/classe abstrata com as anotações desejadas
@JsonIgnoreProperties(ignoreUnknown = true)
abstract class EnderecoExternoMixIn {

    @JsonProperty("street")
    abstract String logradouro;

    @JsonProperty("zip_code")
    abstract String cep;

    @JsonProperty("city")
    abstract String municipio;

    @JsonIgnore
    abstract String complemento;
}

// Registro do MixIn
@Bean
public Jackson2ObjectMapperBuilderCustomizer mixin() {
    return builder -> builder.mixIn(EnderecoExterno.class, EnderecoExternoMixIn.class);
}
```

---

## 10. Casos de Uso Avançados

### 10.1 Desserialização de Tipos Genéricos — `TypeReference`

```java
ObjectMapper mapper = new ObjectMapper();

// ─── Sem TypeReference: tipo genérico perdido em runtime (type erasure) ────────
List<ProdutoDTO> errado = mapper.readValue(json, List.class); // ❌ retorna List<Map>

// ─── Com TypeReference: preserva tipo genérico ────────────────────────────────
List<ProdutoDTO> produtos = mapper.readValue(json,
    new TypeReference<List<ProdutoDTO>>() {});  // ✅

Map<String, ProdutoDTO> mapaPorid = mapper.readValue(json,
    new TypeReference<Map<String, ProdutoDTO>>() {});

// ─── JavaType — alternativa programática ──────────────────────────────────────
JavaType tipo = mapper.getTypeFactory()
    .constructCollectionType(List.class, ProdutoDTO.class);
List<ProdutoDTO> produtos2 = mapper.readValue(json, tipo);

// Genérico aninhado
JavaType tipoAninhado = mapper.getTypeFactory()
    .constructParametricType(ResponseWrapper.class, ProdutoDTO.class);
// → ResponseWrapper<ProdutoDTO>
```

### 10.2 Tree Model — `readTree` e `convertValue`

```java
ObjectMapper mapper = new ObjectMapper();

// JsonNode → tipo forte
JsonNode node = mapper.readTree(json);
ProdutoDTO produto = mapper.treeToValue(node.path("produto"), ProdutoDTO.class);

// Objeto Java → JsonNode (sem serializar para String)
JsonNode nodeFromObject = mapper.valueToTree(produto);

// Mapa → POJO sem passar por String
Map<String, Object> mapa = Map.of("nome", "Teclado", "preco", 299.90);
ProdutoDTO dto = mapper.convertValue(mapa, ProdutoDTO.class);

// POJO → Mapa
Map<String, Object> mapaConvertido = mapper.convertValue(dto,
    new TypeReference<Map<String, Object>>() {});
```

### 10.3 Streaming API — `JsonParser` e `JsonGenerator`

```java
// Streaming é a API de mais baixo nível — use apenas para alto volume ou parsing parcial

// ─── Escrita via JsonGenerator ─────────────────────────────────────────────────
StringWriter sw = new StringWriter();
try (JsonGenerator gen = mapper.createGenerator(sw)) {
    gen.writeStartObject();
    gen.writeStringField("nome", "Produto A");
    gen.writeNumberField("preco", 99.90);
    gen.writeArrayFieldStart("tags");
    gen.writeString("oferta");
    gen.writeString("novo");
    gen.writeEndArray();
    gen.writeEndObject();
}
// → {"nome":"Produto A","preco":99.9,"tags":["oferta","novo"]}

// ─── Leitura via JsonParser — parsing eficiente de grandes arquivos ─────────
try (JsonParser parser = mapper.createParser(inputStream)) {
    while (parser.nextToken() != JsonToken.END_OBJECT) {
        String campo = parser.currentName();
        parser.nextToken();
        if ("nome".equals(campo)) {
            System.out.println("Nome: " + parser.getText());
        }
    }
}
```

### 10.4 `InjectableValues` — Injetar Dependências no Deserializer

```java
// Caso: deserializer precisa de um repositório para validar ou enriquecer dados
public class CategoriaDeserializer extends StdDeserializer<Categoria> {
    public CategoriaDeserializer() { super(Categoria.class); }

    @Override
    public Categoria deserialize(JsonParser p, DeserializationContext ctx)
            throws IOException {
        String id = p.getText();
        // Recupera repositório injetado
        CategoriaRepository repo =
            (CategoriaRepository) ctx.findInjectableValue("categoriaRepository", null, null);
        return repo.findById(Long.parseLong(id))
            .orElseThrow(() -> new JsonParseException(p, "Categoria não encontrada: " + id));
    }
}

// Configuração no Spring Boot
@Bean
public Jackson2ObjectMapperBuilderCustomizer injectableValues(CategoriaRepository repo) {
    return builder -> {
        InjectableValues.Std iv = new InjectableValues.Std();
        iv.addValue("categoriaRepository", repo);
        builder.postConfigurer(m -> m.setInjectableValues(iv));
    };
}
```

### 10.5 JSON Merge Patch — Atualização Parcial (PATCH)

```java
// Cenário: PATCH deve atualizar apenas os campos enviados
// Campos ausentes = "não alterar"; campos com null = "apagar"

@PatchMapping("/{id}")
public ProdutoResponse atualizarParcial(
        @PathVariable Long id,
        @RequestBody JsonNode patch) {         // recebe JSON livre

    Produto existente = service.buscarPorId(id);

    // Converte entidade atual para ObjectNode
    ObjectNode base = mapper.valueToTree(existente);

    // Aplica o patch campo a campo (JSON Merge Patch — RFC 7396)
    base.setAll((ObjectNode) patch);

    // Converte de volta para entidade
    Produto atualizado = mapper.treeToValue(base, Produto.class);
    return ProdutoResponse.de(service.salvar(atualizado));
}
```

**Entrada PATCH:**
```json
{ "preco": 899.90 }   // apenas preço é alterado; nome, categoria etc. permanecem
```

### 10.6 `@JsonView` Dinâmico por Role do Usuário Logado

O `@JsonView` no controller é estático — a view é fixada em tempo de compilação. Para
selecionar a view **em runtime** com base nas roles do usuário autenticado, é preciso
serializar manualmente com o `ObjectMapper` ou usar um `ResponseBodyAdvice` que intercepta
todas as respostas antes de escrever no `HttpServletResponse`.

A abordagem com `ResponseBodyAdvice` é a mais transparente: os controllers continuam
retornando objetos normalmente, e a seleção de view fica centralizada numa única classe.

#### Definição das Views e do DTO

```java
// ─── Views hierárquicas ────────────────────────────────────────────────────────
public class Views {
    public static class Publico {}                          // qualquer usuário
    public static class Autenticado extends Publico {}      // usuário logado
    public static class Gerente extends Autenticado {}      // role GERENTE
    public static class Admin extends Gerente {}            // role ADMIN
}

// ─── DTO com campos segmentados por view ──────────────────────────────────────
public class FuncionarioResponse {

    @JsonView(Views.Publico.class)
    private Long id;

    @JsonView(Views.Publico.class)
    private String nome;

    @JsonView(Views.Publico.class)
    private String departamento;

    @JsonView(Views.Autenticado.class)
    private String email;           // visível apenas para usuários logados

    @JsonView(Views.Gerente.class)
    private BigDecimal salario;     // visível para gerentes e admins

    @JsonView(Views.Gerente.class)
    private String matricula;

    @JsonView(Views.Admin.class)
    private String cpf;             // visível somente para admins

    @JsonView(Views.Admin.class)
    private LocalDate dataAdmissao;

    // getters/setters ou record equivalente
}
```

#### Mapeamento de Role → Classe de View

```java
// ─── Componente que resolve a view correta para o usuário logado ───────────────
@Component
public class JsonViewResolver {

    // Ordem importa: do mais privilegiado para o menos — usa o primeiro match
    private static final List<Map.Entry<String, Class<?>>> ROLE_VIEW_MAP = List.of(
        Map.entry("ROLE_ADMIN",    Views.Admin.class),
        Map.entry("ROLE_GERENTE",  Views.Gerente.class),
        Map.entry("ROLE_USER",     Views.Autenticado.class)
    );

    /**
     * Retorna a view mais privilegiada que o usuário possui.
     * Usuários não autenticados recebem Views.Publico.
     */
    public Class<?> resolverParaUsuarioAtual() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        if (auth == null || !auth.isAuthenticated()
                || auth instanceof AnonymousAuthenticationToken) {
            return Views.Publico.class;
        }

        return auth.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .flatMap(role -> ROLE_VIEW_MAP.stream()
                .filter(e -> e.getKey().equals(role))
                .map(Map.Entry::getValue))
            .findFirst()                           // primeiro match = mais privilegiado
            .orElse(Views.Autenticado.class);      // logado mas sem role mapeada
    }
}
```

#### `ResponseBodyAdvice` — Interceptação Transparente

```java
/**
 * Intercepta todas as respostas anotadas com @DynamicJsonView e
 * reescreve o corpo usando a view calculada para o usuário logado.
 *
 * Funciona com @RestController e @RestControllerAdvice.
 */
@RestControllerAdvice
public class DynamicJsonViewAdvice implements ResponseBodyAdvice<Object> {

    private final ObjectMapper mapper;
    private final JsonViewResolver viewResolver;

    public DynamicJsonViewAdvice(ObjectMapper mapper, JsonViewResolver viewResolver) {
        this.mapper = mapper;
        this.viewResolver = viewResolver;
    }

    /** Ativa apenas em métodos/controllers marcados com @DynamicJsonView */
    @Override
    public boolean supports(MethodParameter returnType,
                            Class<? extends HttpMessageConverter<?>> converterType) {
        return MappingJackson2HttpMessageConverter.class.isAssignableFrom(converterType)
            && (returnType.hasMethodAnnotation(DynamicJsonView.class)
                || AnnotationUtils.findAnnotation(
                       returnType.getContainingClass(), DynamicJsonView.class) != null);
    }

    @Override
    public Object beforeBodyWrite(Object body,
                                  MethodParameter returnType,
                                  MediaType selectedContentType,
                                  Class<? extends HttpMessageConverter<?>> selectedConverterType,
                                  ServerHttpRequest request,
                                  ServerHttpResponse response) {
        if (body == null) return null;

        Class<?> view = viewResolver.resolverParaUsuarioAtual();

        try {
            // Serializa com a view calculada e devolve bytes já prontos
            // O converter não vai re-serializar pois recebe byte[]
            byte[] json = mapper.writerWithView(view).writeValueAsBytes(body);
            response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
            return new String(json, StandardCharsets.UTF_8);
        } catch (JsonProcessingException e) {
            throw new HttpMessageConversionException("Erro ao serializar com view dinâmica", e);
        }
    }
}
```

#### Anotação Marker `@DynamicJsonView`

```java
/**
 * Marca um controller ou método para ter a serialização
 * JSON roteada dinamicamente conforme as roles do usuário.
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DynamicJsonView {}
```

#### Uso no Controller

```java
@RestController
@RequestMapping("/funcionarios")
@DynamicJsonView        // todas as respostas deste controller usam view dinâmica
public class FuncionarioController {

    private final FuncionarioService service;

    @GetMapping
    public List<FuncionarioResponse> listar() {
        // Sem @JsonView estático — a view é resolvida pelo advice em runtime
        return service.listarTodos();
    }

    @GetMapping("/{id}")
    public FuncionarioResponse buscar(@PathVariable Long id) {
        return service.buscarPorId(id);
    }

    // Endpoint específico com view sempre pública — sobrescreve o @DynamicJsonView da classe
    @GetMapping("/publico")
    @DynamicJsonView     // continua usando o advice, mas viewResolver retorna Publico
    public List<FuncionarioResponse> listarPublico() {
        return service.listarAtivos();
    }
}
```

#### Comportamento por Perfil

| Usuário | Authorities | View Aplicada | Campos Visíveis |
|---|---|---|---|
| Não autenticado | — | `Views.Publico` | `id`, `nome`, `departamento` |
| Funcionário comum | `ROLE_USER` | `Views.Autenticado` | + `email` |
| Gerente | `ROLE_GERENTE` | `Views.Gerente` | + `salario`, `matricula` |
| Admin | `ROLE_ADMIN` | `Views.Admin` | + `cpf`, `dataAdmissao` |

**Resposta para `ROLE_USER`:**
```json
{
  "id": 7,
  "nome": "Fernanda Costa",
  "departamento": "Financeiro",
  "email": "fernanda@empresa.com"
}
```

**Resposta para `ROLE_ADMIN`:**
```json
{
  "id": 7,
  "nome": "Fernanda Costa",
  "departamento": "Financeiro",
  "email": "fernanda@empresa.com",
  "salario": 8500.00,
  "matricula": "FIN-0042",
  "cpf": "987.654.321-00",
  "data_admissao": "15/06/2019"
}
```

#### Teste da Lógica de Resolução de View

```java
@ExtendWith(MockitoExtension.class)
class JsonViewResolverTest {

    private final JsonViewResolver resolver = new JsonViewResolver();

    @Test
    void usuarioNaoAutenticadoRecebeViewPublica() {
        SecurityContextHolder.clearContext();
        assertThat(resolver.resolverParaUsuarioAtual()).isEqualTo(Views.Publico.class);
    }

    @Test
    void usuarioComRoleAdminRecebeViewAdmin() {
        autenticar("ROLE_ADMIN");
        assertThat(resolver.resolverParaUsuarioAtual()).isEqualTo(Views.Admin.class);
    }

    @Test
    void usuarioComRoleGerenteNaoRecebeViewAdmin() {
        autenticar("ROLE_GERENTE");
        assertThat(resolver.resolverParaUsuarioAtual()).isEqualTo(Views.Gerente.class);
    }

    @Test
    void usuarioComRoleUserRecebeViewAutenticado() {
        autenticar("ROLE_USER");
        assertThat(resolver.resolverParaUsuarioAtual()).isEqualTo(Views.Autenticado.class);
    }

    @Test
    void usuarioLogadoSemRoleMapeadaRecebeViewAutenticado() {
        autenticar("ROLE_VISUALIZADOR"); // role sem mapeamento explícito
        assertThat(resolver.resolverParaUsuarioAtual()).isEqualTo(Views.Autenticado.class);
    }

    private void autenticar(String... roles) {
        List<GrantedAuthority> authorities = Arrays.stream(roles)
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
        Authentication auth = new UsernamePasswordAuthenticationToken(
            "usuario", "senha", authorities);
        SecurityContextHolder.getContext().setAuthentication(auth);
    }
}
```

#### Teste de Integração com `@WebMvcTest`

```java
@WebMvcTest(FuncionarioController.class)
class FuncionarioControllerJsonViewTest {

    @Autowired MockMvc mockMvc;
    @MockBean FuncionarioService service;

    @BeforeEach
    void setUp() {
        FuncionarioResponse f = new FuncionarioResponse();
        f.setId(7L);
        f.setNome("Fernanda Costa");
        f.setDepartamento("Financeiro");
        f.setEmail("fernanda@empresa.com");
        f.setSalario(new BigDecimal("8500.00"));
        f.setCpf("987.654.321-00");
        given(service.buscarPorId(7L)).willReturn(f);
    }

    @Test
    @WithAnonymousUser
    void anonimVeApenasViewPublica() throws Exception {
        mockMvc.perform(get("/funcionarios/7"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.nome").value("Fernanda Costa"))
            .andExpect(jsonPath("$.email").doesNotExist())   // Autenticado
            .andExpect(jsonPath("$.salario").doesNotExist()) // Gerente
            .andExpect(jsonPath("$.cpf").doesNotExist());    // Admin
    }

    @Test
    @WithMockUser(roles = "USER")
    void roleUserVeEmailMasNaoSalario() throws Exception {
        mockMvc.perform(get("/funcionarios/7"))
            .andExpect(jsonPath("$.email").value("fernanda@empresa.com"))
            .andExpect(jsonPath("$.salario").doesNotExist());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void adminVeTodosCampos() throws Exception {
        mockMvc.perform(get("/funcionarios/7"))
            .andExpect(jsonPath("$.email").value("fernanda@empresa.com"))
            .andExpect(jsonPath("$.salario").value(8500.00))
            .andExpect(jsonPath("$.cpf").value("987.654.321-00"));
    }
}
```

> **Alternativa sem `ResponseBodyAdvice`:** injete `JsonViewResolver` diretamente no controller e serialize manualmente com `mapper.writerWithView(view).writeValueAsString(obj)`, retornando `ResponseEntity<String>` com `Content-Type: application/json`. A abordagem com advice é preferível por manter os controllers limpos.

---

## 11. Tratamento de Erros de Desserialização

```java
// ─── Exceções comuns ──────────────────────────────────────────────────────────
// JsonParseException       → JSON malformado (chave sem aspas, vírgula extra etc.)
// JsonMappingException     → tipo incompatível ou campo obrigatório ausente
// UnrecognizedPropertyException → campo desconhecido com FAIL_ON_UNKNOWN_PROPERTIES=true
// InvalidDefinitionException    → problema na configuração da classe/anotação

// ─── Handler global no Spring Boot ───────────────────────────────────────────
@RestControllerAdvice
public class JsonErrorHandler {

    @ExceptionHandler(HttpMessageNotReadableException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleJsonError(HttpMessageNotReadableException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Corpo da requisição inválido");

        Throwable causa = ex.getCause();
        if (causa instanceof UnrecognizedPropertyException upe) {
            problem.setDetail("Campo desconhecido: '" + upe.getPropertyName() + "'");
        } else if (causa instanceof InvalidFormatException ife) {
            problem.setDetail("Valor inválido para o campo '" +
                ife.getPath().stream()
                    .map(ref -> ref.getFieldName())
                    .collect(Collectors.joining(".")) + "': " + ife.getValue());
        } else if (causa instanceof JsonParseException jpe) {
            problem.setDetail("JSON malformado: " + jpe.getOriginalMessage());
        } else {
            problem.setDetail("Não foi possível ler o corpo da requisição");
        }
        return problem;
    }
}
```

**Resposta para campo com tipo errado:**
```json
{
  "type": "about:blank",
  "title": "Corpo da requisição inválido",
  "status": 400,
  "detail": "Valor inválido para o campo 'preco': 'abc'"
}
```

---

## 12. Testes com Jackson

```java
// ─── Teste unitário de serialização ──────────────────────────────────────────
@Test
void deveSerializarProdutoCorreto() throws Exception {
    ObjectMapper mapper = new ObjectMapper()
        .registerModule(new JavaTimeModule())
        .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    ProdutoResponse produto = new ProdutoResponse(
        1L, "Monitor 4K", new BigDecimal("2999.90"), "Informatica",
        LocalDateTime.of(2025, 3, 17, 10, 0, 0));

    String json = mapper.writeValueAsString(produto);

    assertThat(json).contains("\"nome\":\"Monitor 4K\"");
    assertThat(json).contains("\"preco\":2999.90");
    assertThat(json).contains("\"criado_em\":\"17/03/2025 10:00\""); // se @JsonFormat aplicado
}

// ─── Teste com JsonPath (mais expressivo) ─────────────────────────────────────
// Dependência: com.jayway.jsonpath:json-path-assert
@Test
void deveConterCamposEsperados() throws Exception {
    String json = mapper.writeValueAsString(produto);

    assertThat(json).satisfies(j -> {
        DocumentContext ctx = JsonPath.parse(j);
        assertThat((Integer) ctx.read("$.id")).isEqualTo(1);
        assertThat((String) ctx.read("$.nome")).isEqualTo("Monitor 4K");
        assertThat((Double) ctx.read("$.preco")).isEqualTo(2999.90);
    });
}

// ─── Teste de desserialização ─────────────────────────────────────────────────
@Test
void deveDesserializarJsonParaDTO() throws Exception {
    String json = """
        {
          "id": 1,
          "nome": "Monitor 4K",
          "preco": 2999.90
        }
        """;

    ProdutoResponse dto = mapper.readValue(json, ProdutoResponse.class);

    assertThat(dto.id()).isEqualTo(1L);
    assertThat(dto.nome()).isEqualTo("Monitor 4K");
    assertThat(dto.preco()).isEqualByComparingTo("2999.90");
}

// ─── Teste de controller com MockMvc ─────────────────────────────────────────
@WebMvcTest(ProdutoController.class)
class ProdutoControllerTest {

    @Autowired MockMvc mockMvc;
    @MockBean ProdutoService service;

    @Test
    void deveRetornarProdutoJson() throws Exception {
        given(service.buscarPorId(1L))
            .willReturn(new ProdutoResponse(1L, "Monitor", new BigDecimal("1500"), "Info", LocalDateTime.now()));

        mockMvc.perform(get("/produtos/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.nome").value("Monitor"))
            .andExpect(jsonPath("$.preco").value(1500.0));
    }

    @Test
    void deveAceitarJsonNaEntrada() throws Exception {
        String body = """
            { "nome": "Teclado", "preco": 299.90, "categoria_id": "informatica" }
            """;

        mockMvc.perform(post("/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(body))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"));
    }
}
```

---

## 13. Boas Práticas e Armadilhas Comuns

### Boas Práticas

| Prática | Motivo |
|---|---|
| Use **DTOs/records** em vez de entidades JPA diretamente | Evita exposição acidental de dados, loops de serialização e acoplamento |
| Configure `FAIL_ON_UNKNOWN_PROPERTIES=false` em APIs clientes | APIs externas evoluem; campos novos não devem quebrar sua aplicação |
| Use `WRITE_DATES_AS_TIMESTAMPS=false` | Timestamps numéricos dependem de timezone e são ilegíveis |
| Prefira `Jackson2ObjectMapperBuilderCustomizer` a criar um `@Bean ObjectMapper` | Evita sobrescrever as configurações automáticas do Spring Boot |
| Use `@JsonComponent` para serializers que dependem de beans Spring | Permite injeção de dependências via Spring |
| Compile com `-parameters` ao usar `ParameterNamesModule` | Necessário para descoberta de nomes de parâmetros em runtime |
| Use `TypeReference` ou `JavaType` para tipos genéricos | Evita `ClassCastException` silenciosa por type erasure |

### Armadilhas Comuns

```java
// ❌ ERRADO: ObjectMapper não é thread-safe após configuração
ObjectMapper mapper = new ObjectMapper();
// ... usar em diferentes threads sem sincronização

// ✅ CERTO: configure uma vez, use como singleton (Spring gerencia isso)
@Bean
public ObjectMapper objectMapper() { /* configure aqui */ }

// ─────────────────────────────────────────────────────────────────────────────

// ❌ ERRADO: criar novo ObjectMapper a cada request (muito lento)
@GetMapping("/{id}")
public String buscar(@PathVariable Long id) throws Exception {
    ObjectMapper mapper = new ObjectMapper(); // recria toda configuração
    return mapper.writeValueAsString(service.buscar(id));
}

// ✅ CERTO: injetar o ObjectMapper configurado pelo Spring
@RestController
public class ProdutoController {
    private final ObjectMapper mapper; // injetado pelo Spring

    // ─── ou simplesmente retornar o objeto — Spring serializa automaticamente ──
    @GetMapping("/{id}")
    public ProdutoResponse buscar(@PathVariable Long id) {
        return service.buscar(id); // Jackson serializa internamente via MappingJackson2HttpMessageConverter
    }
}

// ─────────────────────────────────────────────────────────────────────────────

// ❌ ERRADO: @JsonManagedReference em entidades JPA com lazy loading
@Entity
public class Pedido {
    @OneToMany(fetch = FetchType.LAZY)
    @JsonManagedReference  // pode disparar LazyInitializationException fora de transação
    private List<Item> itens;
}

// ✅ CERTO: use DTOs que carregam apenas os dados necessários
public record PedidoResponse(Long id, List<ItemResponse> itens) {}

// ─────────────────────────────────────────────────────────────────────────────

// ❌ ERRADO: ignorar que @JsonView(Views.Admin.class) no Jackson 3 oculta campos sem view
public class Produto {
    @JsonView(Views.Admin.class) String dadosSensiveis;
    String nome; // Jackson 3: omitido pois DEFAULT_VIEW_INCLUSION=false
}

// ✅ CERTO: anote todos os campos relevantes com a view correta
public class Produto {
    @JsonView(Views.Publico.class) String nome; // explícito
    @JsonView(Views.Admin.class)   String dadosSensiveis;
}
```

### Checklist de Configuração Jackson no Spring Boot

- [ ] `WRITE_DATES_AS_TIMESTAMPS` desabilitado
- [ ] `FAIL_ON_UNKNOWN_PROPERTIES` desabilitado (consumidor de APIs externas)
- [ ] `JavaTimeModule` registrado (incluído via `spring-boot-starter-web`)
- [ ] Compilado com `-parameters` (para `ParameterNamesModule` e records)
- [ ] `property-naming-strategy` definida explicitamente (`SNAKE_CASE` ou padrão camelCase)
- [ ] `default-property-inclusion` definida (evitar nulos indesejados na resposta)
- [ ] DTOs sem referências circulares (sem `@JsonManagedReference` em entidades JPA)
- [ ] `@JsonView` aplicado a todos os campos relevantes (especialmente ao migrar para Jackson 3)
- [ ] Handlers de `HttpMessageNotReadableException` configurados no `@ControllerAdvice`
