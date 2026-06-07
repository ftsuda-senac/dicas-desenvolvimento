# Spring Data REST e Spring HATEOAS — Visão Geral Abrangente

> **Baseline principal:** Spring Boot 3.5 · Spring Data REST 4.x · Spring HATEOAS 2.x · Java 21+
>
> **Pré-requisito:** familiaridade com Spring MVC REST (ver `Dicas-Spring-MVC-REST-SSR.md`) e Spring Data JPA (ver `Dicas-JPA-Hibernate-Modelagem-Entities.md`).

---

## Sumário

- [Conceitos Fundamentais — REST Maduro e HATEOAS](#conceitos-fundamentais--rest-maduro-e-hateoas)

1. [Spring Data REST — Visão Geral](#1-spring-data-rest--visão-geral)
   - [1.1 O Que é Spring Data REST](#11-o-que-é-spring-data-rest)
   - [1.2 Dependências e Configuração Mínima](#12-dependências-e-configuração-mínima)
   - [1.3 Como Funciona — Endpoints Gerados Automaticamente](#13-como-funciona--endpoints-gerados-automaticamente)
   - [1.4 application.yml — Configuração Recomendada](#14-applicationyml--configuração-recomendada)
2. [Spring HATEOAS — Visão Geral](#2-spring-hateoas--visão-geral)
   - [2.1 O Que é Spring HATEOAS](#21-o-que-é-spring-hateoas)
   - [2.2 Dependência e Auto-configuração](#22-dependência-e-auto-configuração)
   - [2.3 Modelo de Objetos — Link, EntityModel, CollectionModel, PagedModel](#23-modelo-de-objetos--link-entitymodel-collectionmodel-pagedmodel)
3. [Links e Hypermedia com Spring HATEOAS](#3-links-e-hypermedia-com-spring-hateoas)
   - [3.1 Construindo Links com WebMvcLinkBuilder](#31-construindo-links-com-webmvclinkbuilder)
   - [3.2 EntityModel — Recurso Individual com Links](#32-entitymodel--recurso-individual-com-links)
   - [3.3 CollectionModel — Coleção de Recursos](#33-collectionmodel--coleção-de-recursos)
   - [3.4 PagedModel — Coleção Paginada](#34-pagedmodel--coleção-paginada)
   - [3.5 Affordances — Descrever Ações Possíveis](#35-affordances--descrever-ações-possíveis)
4. [RepresentationModelAssembler — Converter Entidade em Model](#4-representationmodelassembler--converter-entidade-em-model)
   - [4.1 Interface RepresentationModelAssembler](#41-interface-representationmodelassembler)
   - [4.2 RepresentationModelAssemblerSupport](#42-representationmodelassemblersupport)
   - [4.3 Assembler com Paginação](#43-assembler-com-paginação)
5. [Spring Data REST — Customizações](#5-spring-data-rest--customizações)
   - [5.1 RepositoryRestConfigurer — Configuração Central](#51-repositoryrestconfigurer--configuração-central)
   - [5.2 @RepositoryRestResource — Customizar o Repositório](#52-repositoryrestresource--customizar-o-repositório)
   - [5.3 @RestResource — Customizar Métodos Individuais](#53-restresource--customizar-métodos-individuais)
   - [5.4 Projeções — @Projection](#54-projeções--projection)
   - [5.5 Excerpts — Projeção Padrão em Coleções](#55-excerpts--projeção-padrão-em-coleções)
   - [5.6 Validação Integrada com Bean Validation](#56-validação-integrada-com-bean-validation)
6. [Event Handlers — Ciclo de Vida dos Recursos](#6-event-handlers--ciclo-de-vida-dos-recursos)
   - [6.1 Eventos Disponíveis](#61-eventos-disponíveis)
   - [6.2 @RepositoryEventHandler](#62-repositoryeventhandler)
   - [6.3 ApplicationListener Genérico](#63-applicationlistener-genérico)
7. [Customizando Respostas com ResourceProcessor](#7-customizando-respostas-com-resourceprocessor)
8. [Segurança com Spring Security](#8-segurança-com-spring-security)
   - [8.1 Protegendo Repositórios](#81-protegendo-repositórios)
   - [8.2 Filtragem por Usuário Autenticado](#82-filtragem-por-usuário-autenticado)
9. [HAL Explorer — Interface Visual para APIs HAL](#9-hal-explorer--interface-visual-para-apis-hal)
10. [Testes](#10-testes)
    - [10.1 Testando APIs com Spring Data REST](#101-testando-apis-com-spring-data-rest)
    - [10.2 Testando Assemblers HATEOAS Isoladamente](#102-testando-assemblers-hateoas-isoladamente)
    - [10.3 Verificando Links em Respostas JSON](#103-verificando-links-em-respostas-json)
11. [Cenários de Uso: Spring Data REST vs Controller Manual](#11-cenários-de-uso-spring-data-rest-vs-controller-manual)
12. [Integração com Projetos Spring MVC Existentes](#12-integração-com-projetos-spring-mvc-existentes)
    - [12.1 O Problema — Dois Sistemas de Roteamento no Mesmo Contexto](#121-o-problema--dois-sistemas-de-roteamento-no-mesmo-contexto)
    - [12.2 Separação de URLs — base-path Obrigatório](#122-separação-de-urls--base-path-obrigatório)
    - [12.3 Conflito de ObjectMapper e HttpMessageConverters](#123-conflito-de-objectmapper-e-httpmessageconverters)
    - [12.4 Conflito com @ControllerAdvice e Tratamento de Erros](#124-conflito-com-controlleradvice-e-tratamento-de-erros)
    - [12.5 Conflito de Configuração CORS](#125-conflito-de-configuração-cors)
    - [12.6 Integração com Spring Security Já Configurado](#126-integração-com-spring-security-já-configurado)
    - [12.7 Serialização Jackson — Datas, Enums e Campos Ignorados](#127-serialização-jackson--datas-enums-e-campos-ignorados)
    - [12.8 Paginação — Conflito de Parâmetros com SpringDoc/Swagger](#128-paginação--conflito-de-parâmetros-com-springdocswagger)
    - [12.9 Checklist de Integração](#129-checklist-de-integração)
13. [Boas Práticas e Checklist](#13-boas-práticas-e-checklist)

---

## Conceitos Fundamentais — REST Maduro e HATEOAS

### Modelo de Maturidade de Richardson

O Modelo de Maturidade de Richardson define quatro níveis de maturidade para APIs REST:

| Nível | Nome            | Descrição                                                   |
|-------|-----------------|-------------------------------------------------------------|
| 0     | POX (Plain Old XML) | Um único endpoint para tudo (estilo RPC sobre HTTP)     |
| 1     | Recursos        | URLs separadas por recurso (`/produtos`, `/clientes`)       |
| 2     | Verbos HTTP     | Uso correto de GET, POST, PUT, DELETE + status codes        |
| 3     | Hypermedia (HATEOAS) | Respostas incluem links que guiam o cliente           |

> A maioria das APIs "REST" do mercado para no **Nível 2**. O Nível 3 (HATEOAS) é o REST "completo" conforme definido por Roy Fielding.

### O Que é HATEOAS?

**HATEOAS** (Hypermedia As The Engine Of Application State) é a constraint do REST que determina que o servidor deve guiar o cliente através de links nas respostas — o cliente não precisa conhecer URLs antecipadamente.

```json
// Exemplo de resposta HATEOAS — o cliente sabe o que pode fazer a seguir
{
  "id": 1,
  "nome": "Notebook Dell",
  "preco": 3500.00,
  "_links": {
    "self":      { "href": "/api/produtos/1" },
    "produtos":  { "href": "/api/produtos" },
    "categoria": { "href": "/api/categorias/5" },
    "comprar":   { "href": "/api/pedidos", "type": "POST" }
  }
}
```

**Benefícios práticos:**
- Cliente resiliente a mudanças de URL
- API autodocumentada através dos links
- Controle de fluxo do lado do servidor (ex: ocultar "comprar" se estoque = 0)

### HAL — Hypertext Application Language

O **HAL** (`application/hal+json`) é o formato de hypermedia mais popular no ecossistema Spring. Convenções:

- `_links` — objeto com links relacionados ao recurso
- `_embedded` — coleções de recursos embutidos
- Cada link tem `href` obrigatório e pode ter `rel`, `type`, `templated`, etc.

```json
// Coleção HAL com recursos embutidos
{
  "_embedded": {
    "produtos": [
      { "id": 1, "nome": "Notebook", "_links": { "self": { "href": "/api/produtos/1" } } },
      { "id": 2, "nome": "Mouse",    "_links": { "self": { "href": "/api/produtos/2" } } }
    ]
  },
  "_links": {
    "self":  { "href": "/api/produtos?page=0&size=20" },
    "first": { "href": "/api/produtos?page=0&size=20" },
    "next":  { "href": "/api/produtos?page=1&size=20" },
    "last":  { "href": "/api/produtos?page=4&size=20" }
  },
  "page": { "size": 20, "totalElements": 95, "totalPages": 5, "number": 0 }
}
```

---

## 1. Spring Data REST — Visão Geral

### 1.1 O Que é Spring Data REST

**Spring Data REST** é um módulo que expõe automaticamente repositórios Spring Data como uma API REST hipermídia (HAL por padrão), sem necessidade de escrever controllers manualmente.

Com uma interface `JpaRepository`, você obtém gratuitamente:
- CRUD completo via HTTP (GET, POST, PUT, PATCH, DELETE)
- Paginação e ordenação
- Navegação por associações
- Busca por query methods do repositório
- Respostas no formato HAL com links automáticos

**Quando usar:**
- Prototipação rápida e MVPs
- APIs administrativas internas
- Microserviços com lógica de negócio mínima

**Quando NÃO usar:**
- Lógica de negócio complexa em cada operação
- Transformações de DTO não triviais
- Controle fino sobre o contrato da API

### 1.2 Dependências e Configuração Mínima

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Data REST (já inclui Spring HATEOAS) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>

    <!-- JPA + banco de dados -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- HAL Explorer (UI opcional para explorar a API) -->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-rest-hal-explorer</artifactId>
    </dependency>
</dependencies>
```

```java
// Entidade JPA
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String nome;

    @Positive
    private BigDecimal preco;

    @ManyToOne
    private Categoria categoria;

    // getters e setters
}

// Repositório — basta isso para ter uma API REST completa!
@RepositoryRestResource(collectionResourceRel = "produtos", path = "produtos")
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    List<Produto> findByNomeContainingIgnoreCase(@Param("nome") String nome);

    Page<Produto> findByCategoriaId(@Param("categoriaId") Long categoriaId, Pageable pageable);
}
```

Com esse código, o Spring Data REST expõe automaticamente:

| Método | URL                         | Ação                              |
|--------|-----------------------------|-----------------------------------|
| GET    | `/produtos`                 | Listar todos (paginado)           |
| GET    | `/produtos/{id}`            | Buscar por ID                     |
| POST   | `/produtos`                 | Criar                             |
| PUT    | `/produtos/{id}`            | Substituir completo               |
| PATCH  | `/produtos/{id}`            | Atualizar parcialmente            |
| DELETE | `/produtos/{id}`            | Excluir                           |
| GET    | `/produtos/search`          | Listar query methods disponíveis  |
| GET    | `/produtos/search/findByNomeContainingIgnoreCase?nome=note` | Busca customizada |

### 1.3 Como Funciona — Endpoints Gerados Automaticamente

O Spring Data REST descobre todos os repositórios no classpath, determina o tipo de entidade e expõe os endpoints. O URL base padrão é `/` (raiz), configurável via `spring.data.rest.base-path`.

```
GET /
{
  "_links": {
    "produtos":   { "href": "http://localhost:8080/api/produtos{?page,size,sort}" },
    "categorias": { "href": "http://localhost:8080/api/categorias{?page,size,sort}" },
    "profile":    { "href": "http://localhost:8080/api/profile" }
  }
}
```

O endpoint `/profile` segue o padrão **ALPS** (Application-Level Profile Semantics) e descreve a estrutura de todos os recursos.

### 1.4 application.yml — Configuração Recomendada

```yaml
spring:
  data:
    rest:
      # Prefixo de todos os endpoints REST do Spring Data REST
      base-path: /api

      # Tamanho padrão da página
      default-page-size: 20

      # Máximo de itens por página (evita abuso)
      max-page-size: 100

      # Parâmetro de paginação na URL
      page-param-name: page
      limit-param-name: size
      sort-param-name: sort

      # HAL é o padrão — HAL_FORMS adiciona suporte a affordances
      default-media-type: application/hal+json

      # true = IDs das entidades são omitidos na resposta (apenas links)
      # false = IDs aparecem no corpo (mais conveniente para clientes simples)
      return-body-on-create: true
      return-body-on-update: true

      # Detecta apenas repositórios anotados com @RepositoryRestResource
      # ALL = todos os repositórios (padrão); ANNOTATED = somente anotados;
      # DEFAULT = excluindo os anotados com exported=false
      detection-strategy: default
```

---

## 2. Spring HATEOAS — Visão Geral

### 2.1 O Que é Spring HATEOAS

**Spring HATEOAS** é a biblioteca de suporte a hypermedia do ecossistema Spring. Ela fornece:

- Abstrações para construir respostas hipermídia (`EntityModel`, `CollectionModel`, `PagedModel`)
- `WebMvcLinkBuilder` para criar links type-safe apontando para controllers
- Suporte a múltiplos media types: **HAL**, **HAL-FORMS**, **Collection+JSON**, **UBER**
- Serialização/desserialização automática dos modelos hypermídia

Spring Data REST usa Spring HATEOAS internamente. Para controllers escritos manualmente, você pode usar Spring HATEOAS diretamente.

### 2.2 Dependência e Auto-configuração

```xml
<!-- Usar Spring HATEOAS sem Spring Data REST -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

> `spring-boot-starter-data-rest` já inclui `spring-boot-starter-hateoas`. Não adicione os dois.

A auto-configuração registra os `HttpMessageConverter`s para os media types HAL e HAL-FORMS e habilita o suporte a `@EnableHypermediaSupport` automaticamente.

### 2.3 Modelo de Objetos — Link, EntityModel, CollectionModel, PagedModel

```
RepresentationModel          ← classe base (possui _links)
    ├── EntityModel<T>        ← recurso único (wraps T + links)
    ├── CollectionModel<T>    ← coleção de recursos + links
    └── PagedModel<T>         ← coleção paginada + metadados de paginação
```

| Classe             | Quando usar                                          |
|--------------------|------------------------------------------------------|
| `EntityModel<T>`   | Resposta de GET por ID, POST, PUT                   |
| `CollectionModel<T>` | Lista simples sem paginação                        |
| `PagedModel<T>`    | Lista paginada (com `Page` do Spring Data)           |
| `RepresentationModel` | Modelo customizado que estende diretamente       |

---

## 3. Links e Hypermedia com Spring HATEOAS

### 3.1 Construindo Links com WebMvcLinkBuilder

`WebMvcLinkBuilder` cria links type-safe referenciando métodos de controllers — se o URL do método mudar, o link é atualizado automaticamente.

```java
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

// ─── Link simples para um método do controller ────────────────────────────────
Link selfLink = linkTo(methodOn(ProdutoController.class).buscarPorId(1L)).withSelfRel();
// Resultado: { "self": { "href": "http://localhost:8080/api/produtos/1" } }

// ─── Link com rel customizado ─────────────────────────────────────────────────
Link listLink = linkTo(methodOn(ProdutoController.class).listar(Pageable.unpaged()))
    .withRel("produtos");
// Resultado: { "produtos": { "href": "http://localhost:8080/api/produtos" } }

// ─── Link para recurso relacionado (associação) ───────────────────────────────
Link categoriaLink = linkTo(methodOn(CategoriaController.class)
    .buscarPorId(produto.getCategoria().getId()))
    .withRel("categoria");

// ─── Link com template de URL (para buscas com parâmetros opcionais) ──────────
Link searchLink = linkTo(ProdutoController.class).slash("search").withRel("search");

// ─── Link literal (quando não há controller mapeado) ─────────────────────────
Link externalLink = Link.of("https://docs.example.com/api").withRel("documentation");
```

### 3.2 EntityModel — Recurso Individual com Links

```java
@RestController
@RequestMapping("/api/produtos")
public class ProdutoController {

    private final ProdutoService service;

    @GetMapping("/{id}")
    public EntityModel<ProdutoResponse> buscarPorId(@PathVariable Long id) {
        ProdutoResponse dto = service.buscarPorId(id);

        return EntityModel.of(dto,
            linkTo(methodOn(ProdutoController.class).buscarPorId(id))
                .withSelfRel(),
            linkTo(methodOn(ProdutoController.class).listar(Pageable.unpaged()))
                .withRel("produtos"),
            linkTo(methodOn(CategoriaController.class).buscarPorId(dto.categoriaId()))
                .withRel("categoria")
        );
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public EntityModel<ProdutoResponse> criar(@RequestBody @Valid ProdutoCriacaoRequest req) {
        ProdutoResponse criado = service.criar(req);

        return EntityModel.of(criado,
            linkTo(methodOn(ProdutoController.class).buscarPorId(criado.id()))
                .withSelfRel()
        );
    }
}
```

**Resposta JSON gerada:**
```json
{
  "id": 1,
  "nome": "Notebook Dell",
  "preco": 3500.00,
  "_links": {
    "self":      { "href": "http://localhost:8080/api/produtos/1" },
    "produtos":  { "href": "http://localhost:8080/api/produtos" },
    "categoria": { "href": "http://localhost:8080/api/categorias/5" }
  }
}
```

### 3.3 CollectionModel — Coleção de Recursos

```java
@GetMapping
public CollectionModel<EntityModel<ProdutoResponse>> listarTodos() {
    List<EntityModel<ProdutoResponse>> produtos = service.listarTodos()
        .stream()
        .map(dto -> EntityModel.of(dto,
            linkTo(methodOn(ProdutoController.class).buscarPorId(dto.id()))
                .withSelfRel()))
        .toList();

    return CollectionModel.of(produtos,
        linkTo(methodOn(ProdutoController.class).listarTodos()).withSelfRel()
    );
}
```

### 3.4 PagedModel — Coleção Paginada

Para paginação, use `PagedModel` com `PagedResourcesAssembler` injetado automaticamente pelo Spring:

```java
@GetMapping
public PagedModel<EntityModel<ProdutoResponse>> listar(
        Pageable pageable,
        PagedResourcesAssembler<ProdutoResponse> assembler) {

    Page<ProdutoResponse> pagina = service.listar(pageable);

    // O assembler gera automaticamente os links first/prev/self/next/last
    return assembler.toModel(pagina,
        dto -> EntityModel.of(dto,
            linkTo(methodOn(ProdutoController.class).buscarPorId(dto.id()))
                .withSelfRel())
    );
}
```

**Resposta JSON gerada (HAL):**
```json
{
  "_embedded": {
    "produtoResponseList": [
      { "id": 1, "nome": "Notebook", "_links": { "self": { "href": "/api/produtos/1" } } }
    ]
  },
  "_links": {
    "first": { "href": "/api/produtos?page=0&size=20" },
    "self":  { "href": "/api/produtos?page=0&size=20" },
    "next":  { "href": "/api/produtos?page=1&size=20" },
    "last":  { "href": "/api/produtos?page=4&size=20" }
  },
  "page": {
    "size": 20,
    "totalElements": 95,
    "totalPages": 5,
    "number": 0
  }
}
```

> **Customizando o nome do campo `_embedded`:** Por padrão, o Spring HATEOAS usa o nome simples da classe. Para personalizar, anote o DTO com `@Relation`:
> ```java
> @Relation(collectionRelation = "produtos", itemRelation = "produto")
> public record ProdutoResponse(Long id, String nome, BigDecimal preco) {}
> ```

### 3.5 Affordances — Descrever Ações Possíveis

**Affordances** descrevem as ações que podem ser realizadas sobre um recurso (quais métodos HTTP e quais campos aceitar). Funcionam com o media type `application/hal-forms+json`.

```java
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

@GetMapping("/{id}")
public EntityModel<ProdutoResponse> buscarPorId(@PathVariable Long id) {
    ProdutoResponse dto = service.buscarPorId(id);

    // Link self com affordances de atualização e deleção
    Link selfComAffordances = linkTo(methodOn(ProdutoController.class).buscarPorId(id))
        .withSelfRel()
        .andAffordance(afford(methodOn(ProdutoController.class).atualizar(id, null)))
        .andAffordance(afford(methodOn(ProdutoController.class).excluir(id)));

    return EntityModel.of(dto, selfComAffordances);
}
```

**Resposta com HAL-FORMS:**
```json
{
  "id": 1,
  "nome": "Notebook",
  "_links": { "self": { "href": "/api/produtos/1" } },
  "_templates": {
    "atualizar": {
      "method": "PUT",
      "properties": [
        { "name": "nome",  "required": true,  "type": "text" },
        { "name": "preco", "required": true,  "type": "number" }
      ]
    },
    "excluir": { "method": "DELETE" }
  }
}
```

---

## 4. RepresentationModelAssembler — Converter Entidade em Model

### 4.1 Interface RepresentationModelAssembler

`RepresentationModelAssembler<T, D>` é uma interface de conversão:
- `T` — tipo de entrada (entidade ou DTO)
- `D extends RepresentationModel<D>` — tipo de saída (modelo hypermídia)

```java
@Component
public class ProdutoAssembler
        implements RepresentationModelAssembler<Produto, EntityModel<ProdutoResponse>> {

    @Override
    public EntityModel<ProdutoResponse> toModel(Produto produto) {
        ProdutoResponse dto = new ProdutoResponse(
            produto.getId(),
            produto.getNome(),
            produto.getPreco()
        );

        return EntityModel.of(dto,
            linkTo(methodOn(ProdutoController.class).buscarPorId(produto.getId()))
                .withSelfRel(),
            linkTo(methodOn(ProdutoController.class).listar(Pageable.unpaged()))
                .withRel("produtos")
        );
    }
}
```

**Usando o assembler no controller:**
```java
@RestController
@RequestMapping("/api/produtos")
@RequiredArgsConstructor
public class ProdutoController {

    private final ProdutoService service;
    private final ProdutoAssembler assembler;

    @GetMapping("/{id}")
    public EntityModel<ProdutoResponse> buscarPorId(@PathVariable Long id) {
        Produto produto = service.buscarEntidadePorId(id);
        return assembler.toModel(produto);
    }

    @GetMapping
    public CollectionModel<EntityModel<ProdutoResponse>> listar() {
        List<Produto> produtos = service.listarTodos();
        return assembler.toCollectionModel(produtos); // método default da interface
    }
}
```

### 4.2 RepresentationModelAssemblerSupport

`RepresentationModelAssemblerSupport<T, D>` é uma classe abstrata que simplifica a criação de modelos ao adicionar automaticamente o link `self` baseado no controller informado:

```java
@Component
public class ProdutoAssembler
        extends RepresentationModelAssemblerSupport<Produto, EntityModel<ProdutoResponse>> {

    public ProdutoAssembler() {
        // Informa o controller e o tipo de modelo — o link "self" é gerado automaticamente
        super(ProdutoController.class, (Class<EntityModel<ProdutoResponse>>) (Class<?>) EntityModel.class);
    }

    @Override
    public EntityModel<ProdutoResponse> toModel(Produto produto) {
        // createModelWithId adiciona automaticamente o link self
        // Requer que o controller tenha um método @GetMapping("/{id}") com @PathVariable Long id
        EntityModel<ProdutoResponse> model = createModelWithId(produto.getId(), produto);

        // Adicione links extras manualmente
        model.add(linkTo(methodOn(CategoriaController.class)
            .buscarPorId(produto.getCategoria().getId()))
            .withRel("categoria"));

        return model;
    }

    @Override
    protected EntityModel<ProdutoResponse> instantiateModel(Produto produto) {
        // Converte Produto → ProdutoResponse e wraps em EntityModel
        return EntityModel.of(new ProdutoResponse(
            produto.getId(),
            produto.getNome(),
            produto.getPreco(),
            produto.getCategoria().getNome()
        ));
    }
}
```

### 4.3 Assembler com Paginação

```java
@GetMapping
public PagedModel<EntityModel<ProdutoResponse>> listar(
        Pageable pageable,
        PagedResourcesAssembler<Produto> pagedAssembler) {

    Page<Produto> pagina = service.listar(pageable);

    // Delega ao pagedAssembler usando o assembler de entidade individual
    return pagedAssembler.toModel(pagina, assembler);
}
```

---

## 5. Spring Data REST — Customizações

### 5.1 RepositoryRestConfigurer — Configuração Central

```java
@Configuration
public class DataRestConfig implements RepositoryRestConfigurer {

    @Override
    public void configureRepositoryRestConfiguration(
            RepositoryRestConfiguration config,
            CorsRegistry cors) {

        // ── Expor IDs das entidades no JSON ──────────────────────────────────
        // Por padrão, Spring Data REST omite o campo "id" (usa apenas links).
        // Expor IDs facilita integração com clientes mais simples.
        config.exposeIdsFor(Produto.class, Categoria.class, Pedido.class);

        // ── Desabilitar DELETE globalmente ────────────────────────────────────
        config.getExposureConfiguration()
            .forDomainType(Produto.class)
            .withItemExposure((metadata, httpMethods) -> httpMethods.disable(HttpMethod.DELETE))
            .withCollectionExposure((metadata, httpMethods) -> httpMethods.disable(HttpMethod.DELETE));

        // ── CORS para endpoints Spring Data REST ──────────────────────────────
        cors.addMapping("/api/**")
            .allowedOrigins("https://meu-frontend.com")
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
            .allowedHeaders("*");

        // ── Alterar base path programaticamente ──────────────────────────────
        config.setBasePath("/api");

        // ── Tamanho padrão de página ──────────────────────────────────────────
        config.setDefaultPageSize(20);
        config.setMaxPageSize(100);
    }

    @Override
    public void configureConversionService(ConfigurableConversionService conversionService) {
        // Registrar converters customizados usados na desserialização
        conversionService.addConverter(new StringToMoedaConverter());
    }

    @Override
    public void configureJacksonObjectMapper(ObjectMapper objectMapper) {
        // Customizar Jackson para todos os endpoints Spring Data REST
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }
}
```

### 5.2 @RepositoryRestResource — Customizar o Repositório

```java
// ── Repositório totalmente customizado ───────────────────────────────────────
@RepositoryRestResource(
    collectionResourceRel = "produtos",     // nome da chave em _embedded
    itemResourceRel = "produto",            // nome do rel em links individuais
    path = "produtos",                      // segmento de URL
    exported = true                         // false = não expõe este repositório
)
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    // ── Query method exposto como /api/produtos/search/porNome?nome=... ───────
    @RestResource(path = "porNome", rel = "porNome")
    Page<Produto> findByNomeContainingIgnoreCase(
        @Param("nome") String nome,
        Pageable pageable
    );

    // ── Query method NÃO exposto (exported = false) ──────────────────────────
    @RestResource(exported = false)
    List<Produto> findByAtivoFalse();
}

// ── Repositório não exposto (apenas para uso interno) ────────────────────────
@RepositoryRestResource(exported = false)
public interface AuditoriaRepository extends JpaRepository<Auditoria, Long> { }
```

### 5.3 @RestResource — Customizar Métodos Individuais

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Desabilitar apenas o DELETE mantendo os demais
    @Override
    @RestResource(exported = false)
    void deleteById(Long id);

    @Override
    @RestResource(exported = false)
    void delete(Pedido pedido);

    // Customizar a URL de um finder
    @RestResource(path = "recentes", rel = "recentes",
                  description = @Description("Pedidos dos últimos 30 dias"))
    List<Pedido> findByDataCriacaoAfter(@Param("data") LocalDate data);
}
```

### 5.4 Projeções — @Projection

**Projeções** permitem retornar uma visão customizada da entidade, sem expor todos os campos ou incluindo campos calculados e relacionamentos inline.

```java
// ── Projeção simples — subconjunto de campos ──────────────────────────────────
@Projection(name = "resumo", types = { Produto.class })
public interface ProdutoResumo {
    Long getId();
    String getNome();
    BigDecimal getPreco();
    // getNome() da entidade Categoria (join automático)
    String getCategoriaNome();  // ⚠️ precisa que o campo na entidade se chame "categoriaNome"
                                 //    ou que use @Value abaixo
}

// ── Projeção com SpEL — campos calculados e acesso a relacionamentos ──────────
@Projection(name = "detalhada", types = { Produto.class })
public interface ProdutoDetalhada {
    Long getId();
    String getNome();
    BigDecimal getPreco();

    // SpEL: acessa o relacionamento @ManyToOne diretamente
    @Value("#{target.categoria.nome}")
    String getNomeCategoria();

    // SpEL: campo calculado
    @Value("#{target.preco * 1.1}")
    BigDecimal getPrecoComImposto();

    // Inclui o objeto completo do relacionamento inline (evita link externo)
    Categoria getCategoria();
}
```

**Usando projeções na URL:**
```
GET /api/produtos/1?projection=resumo
GET /api/produtos?projection=detalhada
```

**Usando projeções no repositório (para buscas):**
```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    // Retorna projeção diretamente em queries
    List<ProdutoResumo> findByCategoria(Categoria categoria);

    @Query("SELECT p FROM Produto p WHERE p.preco < :preco")
    List<ProdutoDetalhada> findBaratos(@Param("preco") BigDecimal preco);
}
```

### 5.5 Excerpts — Projeção Padrão em Coleções

Por padrão, ao listar uma coleção, o Spring Data REST usa a entidade completa. **Excerpts** permitem definir qual projeção é usada automaticamente nas listagens:

```java
@RepositoryRestResource(
    collectionResourceRel = "produtos",
    path = "produtos",
    excerptProjection = ProdutoResumo.class   // projeção padrão na coleção
)
public interface ProdutoRepository extends JpaRepository<Produto, Long> { }
```

Agora `GET /api/produtos` retorna `ProdutoResumo` automaticamente, sem precisar de `?projection=resumo`. O recurso individual (`GET /api/produtos/1`) ainda retorna a entidade completa.

### 5.6 Validação Integrada com Bean Validation

O Spring Data REST respeita Bean Validation automaticamente nas operações de escrita (POST/PUT/PATCH), desde que as anotações estejam na entidade:

```java
@Entity
public class Produto {
    @Id @GeneratedValue
    private Long id;

    @NotBlank(message = "O nome é obrigatório")
    @Size(min = 2, max = 100)
    private String nome;

    @NotNull
    @Positive(message = "O preço deve ser positivo")
    private BigDecimal preco;
}
```

Também é possível registrar validators customizados via `RepositoryRestConfigurer`:

```java
@Configuration
public class DataRestConfig implements RepositoryRestConfigurer {

    @Autowired
    private ProdutoValidator produtoValidator;

    @Override
    public void configureValidatingRepositoryEventListener(
            ValidatingRepositoryEventListener validatingListener) {
        // Roda o validator antes de salvar (beforeCreate e beforeSave)
        validatingListener.addValidator("beforeCreate", produtoValidator);
        validatingListener.addValidator("beforeSave", produtoValidator);
    }
}
```

---

## 6. Event Handlers — Ciclo de Vida dos Recursos

### 6.1 Eventos Disponíveis

O Spring Data REST publica eventos do ciclo de vida antes e depois de cada operação:

| Evento                  | Momento                                         |
|-------------------------|-------------------------------------------------|
| `BeforeCreateEvent`     | Antes de salvar uma nova entidade (POST)        |
| `AfterCreateEvent`      | Depois de salvar uma nova entidade              |
| `BeforeSaveEvent`       | Antes de atualizar (PUT/PATCH)                  |
| `AfterSaveEvent`        | Depois de atualizar                             |
| `BeforeDeleteEvent`     | Antes de excluir (DELETE)                       |
| `AfterDeleteEvent`      | Depois de excluir                               |
| `BeforeLinkSaveEvent`   | Antes de salvar uma associação                  |
| `AfterLinkSaveEvent`    | Depois de salvar uma associação                 |
| `BeforeLinkDeleteEvent` | Antes de excluir uma associação                 |
| `AfterLinkDeleteEvent`  | Depois de excluir uma associação                |

### 6.2 @RepositoryEventHandler

```java
@Component
@RepositoryEventHandler  // registra este bean como handler de eventos
public class ProdutoEventHandler {

    private final AuditoriaService auditoria;
    private final EstoqueService estoque;

    // ── Rodando lógica antes de criar ─────────────────────────────────────────
    @HandleBeforeCreate
    public void beforeCreate(Produto produto) {
        // Normalizar dados, calcular campos derivados, etc.
        produto.setNome(produto.getNome().trim());
        produto.setDataCriacao(LocalDateTime.now());
        estoque.reservarEspaco(produto);
    }

    // ── Auditoria após criar ──────────────────────────────────────────────────
    @HandleAfterCreate
    public void afterCreate(Produto produto) {
        auditoria.registrar("PRODUTO_CRIADO", produto.getId());
    }

    // ── Validação antes de salvar (atualização) ───────────────────────────────
    @HandleBeforeSave
    public void beforeSave(Produto produto) {
        if (produto.getPreco().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Preço deve ser positivo");
        }
    }

    // ── Limpeza após excluir ──────────────────────────────────────────────────
    @HandleAfterDelete
    public void afterDelete(Produto produto) {
        auditoria.registrar("PRODUTO_EXCLUIDO", produto.getId());
    }
}
```

### 6.3 ApplicationListener Genérico

Para casos onde um único listener deve tratar múltiplas entidades:

```java
@Component
public class AuditoriaUniversalListener
        implements ApplicationListener<RepositoryEvent> {

    private final AuditoriaService auditoria;

    @Override
    public void onApplicationEvent(RepositoryEvent event) {
        Object entidade = event.getSource();

        if (event instanceof AfterCreateEvent) {
            auditoria.registrar("CRIACAO", entidade.getClass().getSimpleName(), entidade);
        } else if (event instanceof AfterDeleteEvent) {
            auditoria.registrar("EXCLUSAO", entidade.getClass().getSimpleName(), entidade);
        }
    }
}
```

---

## 7. Customizando Respostas com ResourceProcessor

`ResourceProcessor<T>` permite interceptar e modificar qualquer modelo hypermídia antes de ser serializado — o equivalente a um `ResponseBodyAdvice` específico para HATEOAS.

```java
@Component
public class ProdutoResourceProcessor
        implements ResourceProcessor<EntityModel<Produto>> {

    @Override
    public EntityModel<Produto> process(EntityModel<Produto> model) {
        Produto produto = model.getContent();

        // Adicionar link condicional — "comprar" só disponível se há estoque
        if (produto != null && produto.getEstoque() > 0) {
            model.add(linkTo(methodOn(PedidoController.class)
                .criar(null))
                .withRel("comprar"));
        }

        // Adicionar link para imagem do produto
        if (produto != null && produto.getImagemUrl() != null) {
            model.add(Link.of(produto.getImagemUrl()).withRel("imagem"));
        }

        return model;
    }
}
```

> `ResourceProcessor` funciona tanto com **Spring Data REST** quanto com controllers Spring MVC que retornam `EntityModel`.

---

## 8. Segurança com Spring Security

### 8.1 Protegendo Repositórios

```java
@RepositoryRestResource
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    // ── Método protegido por anotação Spring Security ──────────────────────────
    @Override
    @PreAuthorize("hasRole('ADMIN')")
    void deleteById(Long id);

    @Override
    @PreAuthorize("hasRole('ADMIN')")
    void delete(Produto produto);

    // Leitura pública, escrita restrita
    @Override
    @PreAuthorize("hasAnyRole('ADMIN', 'EDITOR')")
    <S extends Produto> S save(S produto);
}
```

**Configuração de segurança para endpoints Spring Data REST:**
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // habilita @PreAuthorize nos repositórios
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                // Leitura pública
                .requestMatchers(HttpMethod.GET, "/api/**").permitAll()
                // Modificação requer autenticação
                .requestMatchers(HttpMethod.POST, "/api/**").authenticated()
                .requestMatchers(HttpMethod.PUT, "/api/**").authenticated()
                .requestMatchers(HttpMethod.PATCH, "/api/**").authenticated()
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .csrf(csrf -> csrf.disable())  // APIs stateless tipicamente desabilitam CSRF
            .build();
    }
}
```

### 8.2 Filtragem por Usuário Autenticado

```java
@RepositoryRestResource
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Retorna apenas pedidos do usuário autenticado
    @Query("SELECT p FROM Pedido p WHERE p.usuario.email = :#{authentication.name}")
    @PreAuthorize("isAuthenticated()")
    Page<Pedido> findAll(Pageable pageable);

    // SpEL com referência ao contexto de segurança
    @Query("SELECT p FROM Pedido p WHERE p.usuario.email = :email")
    Page<Pedido> findByUsuarioEmail(
        @Param("email") @Value("#{authentication.name}") String email,
        Pageable pageable
    );
}
```

---

## 9. HAL Explorer — Interface Visual para APIs HAL

O **HAL Explorer** é uma interface web que permite navegar pela API HAL interativamente, sem precisar de ferramentas externas como Postman.

**Dependência:**
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-rest-hal-explorer</artifactId>
</dependency>
```

Após adicionar a dependência, acesse `http://localhost:8080/api` (ou o `base-path` configurado) no navegador. O Spring Boot redireciona automaticamente para o HAL Explorer.

**Funcionalidades:**
- Navegação por links hypermídia clicáveis
- Formulários automáticos para POST/PUT/PATCH (via HAL-FORMS)
- Visualização formatada de respostas JSON
- Suporte a autenticação via header customizado

---

## 10. Testes

### 10.1 Testando APIs com Spring Data REST

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class ProdutoRestApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ProdutoRepository repository;

    @Test
    void deveListarProdutos() throws Exception {
        repository.save(new Produto("Notebook", new BigDecimal("3500.00")));

        mockMvc.perform(get("/api/produtos")
                .accept(MediaTypes.HAL_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$._embedded.produtos").isArray())
            .andExpect(jsonPath("$._embedded.produtos[0].nome").value("Notebook"))
            .andExpect(jsonPath("$._embedded.produtos[0]._links.self.href").exists())
            .andExpect(jsonPath("$._links.self.href").exists())
            .andExpect(jsonPath("$.page.totalElements").value(1));
    }

    @Test
    void deveCriarProduto() throws Exception {
        String payload = """
            {
              "nome": "Mouse Logitech",
              "preco": 150.00
            }
            """;

        mockMvc.perform(post("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.nome").value("Mouse Logitech"))
            .andExpect(jsonPath("$._links.self.href").exists());
    }

    @Test
    void deveBuscarPorNomeViaSearchEndpoint() throws Exception {
        repository.save(new Produto("Notebook Dell", new BigDecimal("4000.00")));
        repository.save(new Produto("Notebook HP", new BigDecimal("3500.00")));

        mockMvc.perform(get("/api/produtos/search/porNome")
                .param("nome", "Dell")
                .accept(MediaTypes.HAL_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$._embedded.produtos").isArray())
            .andExpect(jsonPath("$._embedded.produtos", hasSize(1)))
            .andExpect(jsonPath("$._embedded.produtos[0].nome").value("Notebook Dell"));
    }

    @Test
    void deveRetornar404ParaProdutoInexistente() throws Exception {
        mockMvc.perform(get("/api/produtos/9999")
                .accept(MediaTypes.HAL_JSON))
            .andExpect(status().isNotFound());
    }
}
```

### 10.2 Testando Assemblers HATEOAS Isoladamente

```java
class ProdutoAssemblerTest {

    // Habilitar construção de links em contexto de teste
    @BeforeEach
    void setUp() {
        WebMvcLinkBuilderFactory factory = new WebMvcLinkBuilderFactory();
        // Simula requisição para resolver links relativos
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setServerName("localhost");
        request.setServerPort(8080);
        RequestContextHolder.setRequestAttributes(new ServletRequestAttributes(request));
    }

    @AfterEach
    void tearDown() {
        RequestContextHolder.resetRequestAttributes();
    }

    @Test
    void deveCriarModelComLinksCorretos() {
        ProdutoAssembler assembler = new ProdutoAssembler();
        Produto produto = new Produto(1L, "Notebook", new BigDecimal("3500.00"));

        EntityModel<ProdutoResponse> model = assembler.toModel(produto);

        assertThat(model.getContent()).isNotNull();
        assertThat(model.getContent().nome()).isEqualTo("Notebook");

        assertThat(model.getLink("self"))
            .isPresent()
            .hasValueSatisfying(link ->
                assertThat(link.getHref()).contains("/api/produtos/1"));

        assertThat(model.getLink("produtos")).isPresent();
    }
}
```

### 10.3 Verificando Links em Respostas JSON

```java
@Test
void deveConterLinksHypermidiaCompletos() throws Exception {
    Produto salvo = repository.save(new Produto("Teclado", new BigDecimal("250.00")));

    mockMvc.perform(get("/api/produtos/{id}", salvo.getId())
            .accept(MediaTypes.HAL_JSON))
        .andExpect(status().isOk())
        // Verifica link self
        .andExpect(jsonPath("$._links.self.href")
            .value(containsString("/api/produtos/" + salvo.getId())))
        // Verifica link de coleção
        .andExpect(jsonPath("$._links.produtos.href")
            .value(containsString("/api/produtos")))
        // Verifica que o link self é um URI válido (não template)
        .andExpect(jsonPath("$._links.self.templated").doesNotExist());
}

@Test
void deveConterLinksDePaginacao() throws Exception {
    // Salvar mais de uma página de produtos
    IntStream.range(0, 25).forEach(i ->
        repository.save(new Produto("Produto " + i, BigDecimal.TEN)));

    mockMvc.perform(get("/api/produtos")
            .param("size", "10")
            .param("page", "1")
            .accept(MediaTypes.HAL_JSON))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$._links.first.href").exists())
        .andExpect(jsonPath("$._links.prev.href").exists())
        .andExpect(jsonPath("$._links.self.href").exists())
        .andExpect(jsonPath("$._links.next.href").exists())
        .andExpect(jsonPath("$._links.last.href").exists());
}
```

---

## 11. Cenários de Uso: Spring Data REST vs Controller Manual

| Critério                         | Spring Data REST            | Controller Manual + HATEOAS   |
|----------------------------------|-----------------------------|-------------------------------|
| Velocidade de desenvolvimento    | Alta (zero boilerplate)     | Média                          |
| Controle sobre o contrato da API | Baixo                       | Total                          |
| Lógica de negócio por operação   | Via EventHandlers (limitado)| Direta no Service              |
| DTOs customizados                | Via Projeções (limitado)    | Total flexibilidade            |
| Validação customizada            | Via Validators + anotações  | Total via @Valid + Service     |
| Segurança por método             | Via @PreAuthorize           | Via @PreAuthorize ou SecurityConfig |
| Testabilidade                    | Testes de integração diretos| Unitário + integração          |
| Consistência com outros endpoints| Difícil de misturar         | Uniforme                       |

**Regra prática:**
- Use **Spring Data REST** para recursos simples de CRUD, admin interno, prototipação
- Use **Controller Manual + Spring HATEOAS** para APIs públicas, lógica de negócio complexa
- **Não misture** os dois estilos para o mesmo recurso — gera inconsistência

---

## 12. Integração com Projetos Spring MVC Existentes

Adicionar Spring Data REST (ou Spring HATEOAS puro) a um projeto que já usa Spring MVC com `@RestController` exige atenção a vários pontos de conflito. Esta seção cobre os problemas mais comuns e como resolvê-los.

### 12.1 O Problema — Dois Sistemas de Roteamento no Mesmo Contexto

Spring MVC e Spring Data REST coexistem no mesmo `ApplicationContext` e no mesmo `DispatcherServlet`, mas usam mecanismos de roteamento diferentes:

- **Spring MVC**: `@RequestMapping` registrado em `HandlerMapping` com prioridade alta
- **Spring Data REST**: `RepositoryRestHandlerMapping` registrado com prioridade mais baixa

**Ordem de resolução de requisições** (do mais prioritário ao menos):

```
1. RequestMappingHandlerMapping   ← @RequestMapping/@GetMapping do seu @RestController
2. RepositoryRestHandlerMapping   ← endpoints do Spring Data REST
3. BasePathAwareHandlerMapping    ← /profile, /explorer (HAL Explorer)
```

Se um `@RestController` mapear `GET /api/produtos`, ele vence sobre o endpoint gerado pelo Spring Data REST — sem aviso, sem erro. Por isso, **separação de URLs é obrigatória**.

### 12.2 Separação de URLs — base-path Obrigatório

O `base-path` do Spring Data REST isola os endpoints gerados automaticamente de qualquer URL usada pelos seus controllers.

```yaml
# application.yml — isola Spring Data REST em /api/data
spring:
  data:
    rest:
      base-path: /api/data   # todos os endpoints SDR ficam em /api/data/**
```

```java
// Seus controllers MVC existentes — mantêm o prefixo original
@RestController
@RequestMapping("/api")          // permanece em /api/**
public class ProdutoController { ... }

// Spring Data REST gera automaticamente em /api/data/produtos
// Seus controllers ficam em /api/produtos — sem conflito
```

> **Convenção recomendada para projetos mistos:**
> - Controllers manuais: `/api/**`
> - Spring Data REST: `/api/data/**` ou `/internal/**`

**Verificando conflitos existentes com o Actuator:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: mappings   # GET /actuator/mappings lista todos os endpoints registrados
```

### 12.3 Conflito de ObjectMapper e HttpMessageConverters

Ao adicionar `spring-boot-starter-data-rest`, o Spring Data REST registra seus próprios `HttpMessageConverter`s para os media types HAL (`application/hal+json`) e HAL-FORMS. Isso **não afeta** o `ObjectMapper` padrão usado pelos seus `@RestController`s — os dois convivem.

O problema ocorre quando você **customizou o `ObjectMapper`** globalmente:

```java
// ⚠️ Configuração que pode causar comportamento inesperado no Spring Data REST
@Bean
public Jackson2ObjectMapperBuilderCustomizer customizer() {
    return builder -> builder
        .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        .modules(new JavaTimeModule())
        .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE); // ← PROBLEMA
}
```

Se você definir `SNAKE_CASE` globalmente, os campos `_links`, `_embedded` e `page` do HAL **também serão afetados**, corrompendo o formato das respostas do Spring Data REST.

**Solução — customizar o ObjectMapper apenas para Spring Data REST:**
```java
@Configuration
public class DataRestConfig implements RepositoryRestConfigurer {

    @Override
    public void configureJacksonObjectMapper(ObjectMapper objectMapper) {
        // Customizações aplicadas APENAS aos endpoints do Spring Data REST
        // Não afeta seus @RestControllers
        objectMapper.configure(
            DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

        // NÃO altere propertyNamingStrategy aqui — quebraria o formato HAL
    }
}
```

**Customizando o ObjectMapper dos seus controllers sem afetar o SDR:**
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // Adiciona converter customizado apenas para application/json
        // Não interfere com os converters HAL registrados pelo Spring Data REST
        MappingJackson2HttpMessageConverter converter =
            new MappingJackson2HttpMessageConverter();
        converter.setObjectMapper(meuObjectMapperCustomizado());
        converters.add(0, converter);
    }
}
```

### 12.4 Conflito com @ControllerAdvice e Tratamento de Erros

O `@ControllerAdvice` global dos seus controllers **também é acionado** para exceções lançadas em endpoints do Spring Data REST, o que pode gerar respostas inconsistentes.

**Problema típico:** seu `@ControllerAdvice` retorna um formato de erro customizado, mas o Spring Data REST espera (e os clientes esperam) o formato padrão `application/problem+json`.

```java
// ⚠️ Este handler é acionado para TODOS os controllers, incluindo Spring Data REST
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<MeuErroCustomizado> handleAll(Exception ex) {
        // Isso sobrescreve o tratamento de erro padrão do Spring Data REST!
        return ResponseEntity.status(500).body(new MeuErroCustomizado(ex.getMessage()));
    }
}
```

**Solução 1 — restringir o `@ControllerAdvice` aos seus packages:**
```java
// Aplica apenas aos controllers do seu pacote, não ao Spring Data REST
@RestControllerAdvice(basePackages = "com.empresa.meuapp.controller")
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problem);
    }
}
```

**Solução 2 — excluir os beans do Spring Data REST com `assignableTypes`:**
```java
// Aplica a tudo EXCETO aos handlers do Spring Data REST
@RestControllerAdvice(assignableTypes = {
    ProdutoController.class,
    CategoriaController.class,
    PedidoController.class
})
public class GlobalExceptionHandler { ... }
```

**Solução 3 — verificar a origem da requisição no handler:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Prefixo base do Spring Data REST — vem do application.yml
    @Value("${spring.data.rest.base-path:/api/data}")
    private String dataRestBasePath;

    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handleAll(Exception ex, HttpServletRequest request) {
        // Deixa o Spring Data REST tratar suas próprias exceções
        if (request.getRequestURI().startsWith(dataRestBasePath)) {
            throw ex; // re-lança para o handler padrão do SDR
        }
        return ResponseEntity.status(500)
            .body(new MeuErroCustomizado(ex.getMessage()));
    }
}
```

### 12.5 Conflito de Configuração CORS

Projetos Spring MVC costumam configurar CORS via `WebMvcConfigurer`. Essa configuração **não se aplica automaticamente** aos endpoints do Spring Data REST — eles usam um mecanismo próprio.

```java
// ⚠️ Esta configuração NÃO cobre os endpoints /api/data/** do Spring Data REST
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")           // cobre seus controllers
            .allowedOrigins("https://meu-frontend.com")
            .allowedMethods("*");
        // /api/data/** NÃO está coberto aqui
    }
}
```

**Solução — configurar CORS nos dois lugares:**
```java
// CORS para seus controllers (WebMvcConfigurer)
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://meu-frontend.com")
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE");
    }
}

// CORS para Spring Data REST (RepositoryRestConfigurer)
@Configuration
public class DataRestConfig implements RepositoryRestConfigurer {

    @Override
    public void configureRepositoryRestConfiguration(
            RepositoryRestConfiguration config, CorsRegistry cors) {
        cors.addMapping("/api/data/**")          // base-path do Spring Data REST
            .allowedOrigins("https://meu-frontend.com")
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE");
    }
}
```

> Se você usa Spring Security, o filtro `CorsFilter` do Security precisa cobrir todos os paths — veja a seção 8.

### 12.6 Integração com Spring Security Já Configurado

Ao adicionar Spring Data REST em um projeto com Spring Security configurado, os novos endpoints (`/api/data/**`) ficam **bloqueados por padrão**, pois a regra `.anyRequest().authenticated()` (ou `.denyAll()`) cobre tudo.

```java
// SecurityConfig existente — os endpoints do Spring Data REST
// também serão barrados pelo anyRequest().authenticated()
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .anyRequest().authenticated()   // ← também bloqueia /api/data/**
        )
        .build();
}
```

**Solução — adicionar regras explícitas para o base-path do Spring Data REST:**
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(auth -> auth
            // ── Seus endpoints existentes ─────────────────────────────────
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers(HttpMethod.GET, "/api/**").authenticated()

            // ── Endpoints do Spring Data REST ────────────────────────────
            .requestMatchers(HttpMethod.GET,    "/api/data/**").authenticated()
            .requestMatchers(HttpMethod.POST,   "/api/data/**").hasRole("EDITOR")
            .requestMatchers(HttpMethod.PUT,    "/api/data/**").hasRole("EDITOR")
            .requestMatchers(HttpMethod.PATCH,  "/api/data/**").hasRole("EDITOR")
            .requestMatchers(HttpMethod.DELETE, "/api/data/**").hasRole("ADMIN")

            .anyRequest().authenticated()
        )
        .build();
}
```

**Ponto de atenção — HAL Explorer e `/profile`:**

O Spring Data REST registra dois endpoints especiais que também precisam de permissão:
- `GET /api/data` — raiz da API (lista todos os recursos disponíveis)
- `GET /api/data/profile` — metadados ALPS
- `GET /api/data/explorer` — HAL Explorer (se no classpath)

```java
// Permitir acesso público à raiz e ao profile (sem dados sensíveis)
.requestMatchers(HttpMethod.GET, "/api/data", "/api/data/profile").permitAll()
```

### 12.7 Serialização Jackson — Datas, Enums e Campos Ignorados

O Spring Data REST usa um `ObjectMapper` separado, configurado pelo `HalConfiguration`. Configurações feitas via `application.properties` no namespace `spring.jackson.*` se aplicam ao ObjectMapper padrão **e** ao do Spring Data REST (pois ambos partem do mesmo `JacksonProperties`).

```yaml
# Estas configurações afetam AMBOS os ObjectMappers
spring:
  jackson:
    serialization:
      write-dates-as-timestamps: false   # datas como ISO-8601 string
      write-enums-using-to-string: true  # enum usa toString() em vez de name()
    deserialization:
      fail-on-unknown-properties: false
    default-property-inclusion: non_null
```

**Problema com `@JsonIgnore` em entidades JPA:**

Se você anotou um campo com `@JsonIgnore` para omiti-lo dos seus controllers, o Spring Data REST também vai ignorar o campo — o que pode ser desejado ou não:

```java
@Entity
public class Usuario {
    @JsonIgnore                   // ← oculta nos controllers E no Spring Data REST
    private String senha;

    @JsonProperty(access = READ_ONLY)  // ← alternativa mais granular: lê mas não escreve
    private String email;
}
```

**Problema com relacionamentos bidirecionais (`@JsonManagedReference` / `@JsonBackReference`):**

Spring Data REST serializa associações como **links** (não inline), então `@JsonManagedReference` e `@JsonBackReference` podem gerar comportamento inesperado. Prefira:

```java
@Entity
public class Pedido {
    @ManyToOne
    @JoinColumn(name = "cliente_id")
    // Não use @JsonBackReference aqui — Spring Data REST substitui por link
    // Use projeções (@Projection) se precisar do objeto inline
    private Cliente cliente;
}
```

### 12.8 Paginação — Conflito de Parâmetros com SpringDoc/Swagger

Se o projeto já usa SpringDoc OpenAPI, a integração com Spring Data REST exige a dependência adicional `springdoc-openapi-data-rest`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.x.x</version>
</dependency>

<!-- Integração SpringDoc + Spring Data REST -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-data-rest</artifactId>
    <version>2.x.x</version>
</dependency>
```

Sem a dependência de integração, o Swagger não documenta os endpoints do Spring Data REST — e os parâmetros de paginação (`page`, `size`, `sort`) podem não aparecer corretamente nos endpoints dos seus controllers que usam `Pageable`.

**Ponto de atenção — nomes dos parâmetros de paginação:**

O Spring Data REST permite renomear os parâmetros de paginação via `application.yml`. Se você fizer isso e também usar `Pageable` nos seus controllers, os nomes devem ser consistentes:

```yaml
spring:
  data:
    rest:
      page-param-name: pagina    # ← renomeia ?page para ?pagina
      limit-param-name: tamanho  # ← renomeia ?size para ?tamanho
      sort-param-name: ordenar   # ← renomeia ?sort para ?ordenar
```

> **Atenção:** renomear parâmetros no Spring Data REST **não afeta** os controllers MVC que recebem `Pageable` — eles continuam usando `page`/`size`/`sort`. Evite renomear para não gerar inconsistência.

### 12.9 Checklist de Integração

Use este checklist ao adicionar Spring Data REST ou Spring HATEOAS em um projeto Spring MVC existente:

**Configuração obrigatória:**
- [ ] Definir `spring.data.rest.base-path` diferente do prefixo dos controllers MVC
- [ ] Verificar conflitos de URL com `GET /actuator/mappings`
- [ ] Anotar com `@RepositoryRestResource(exported = false)` repositórios que **não** devem ser expostos
- [ ] Configurar CORS separadamente em `RepositoryRestConfigurer` (não apenas em `WebMvcConfigurer`)

**Spring Security:**
- [ ] Adicionar regras de autorização explícitas para o `base-path` do Spring Data REST
- [ ] Liberar acesso a `/api/data` (raiz) e `/api/data/profile` se necessário
- [ ] Habilitar `@EnableMethodSecurity` se usar `@PreAuthorize` nos repositórios

**Jackson e serialização:**
- [ ] Não usar `PropertyNamingStrategy.SNAKE_CASE` globalmente (quebra o formato HAL)
- [ ] Revisar uso de `@JsonIgnore` em entidades — afeta respostas do Spring Data REST também
- [ ] Evitar `@JsonManagedReference`/`@JsonBackReference` em entidades expostas pelo SDR — usar projeções

**Tratamento de erros:**
- [ ] Restringir `@ControllerAdvice` ao package dos seus controllers (evitar sobrescrever o handler do SDR)
- [ ] Testar respostas de erro (404, 400, 409) dos endpoints do Spring Data REST para garantir formato correto

**SpringDoc/Swagger:**
- [ ] Adicionar `springdoc-openapi-starter-data-rest` se usar SpringDoc
- [ ] Não renomear parâmetros de paginação do Spring Data REST (mantém consistência com os controllers MVC)

---

## 13. Boas Práticas e Checklist

### Design da API

- [ ] Definir um `base-path` explícito (`/api`) para separar do restante da aplicação
- [ ] Expor IDs (`exposeIdsFor`) apenas quando os clientes realmente precisam
- [ ] Usar `@Relation` nos DTOs para controlar os nomes dos campos `_embedded`
- [ ] Desabilitar operações desnecessárias com `@RestResource(exported = false)` ou `withItemExposure`
- [ ] Usar Projeções para evitar over-fetching em coleções (Excerpts)

### HATEOAS

- [ ] Preferir `WebMvcLinkBuilder.methodOn()` a links literais — evita URLs hardcoded
- [ ] Centralizar lógica de links em `RepresentationModelAssembler` (evitar duplicação nos controllers)
- [ ] Anotar DTOs com `@Relation` para nomes semanticamente corretos em `_embedded`
- [ ] Incluir link `self` em todos os recursos individuais
- [ ] Incluir links de paginação (`first`/`prev`/`next`/`last`) via `PagedResourcesAssembler`

### Segurança

- [ ] Configurar CORS explicitamente via `RepositoryRestConfigurer`
- [ ] Proteger operações de escrita com `@PreAuthorize` nos métodos do repositório
- [ ] Usar `STATELESS` session management para APIs JSON
- [ ] Validar CSRF somente se a API é consumida por browsers (geralmente desabilitado em APIs)

### Performance

- [ ] Usar Projeções para evitar joins desnecessários em consultas de coleção
- [ ] Configurar `max-page-size` para evitar consultas excessivamente grandes
- [ ] Adicionar índices nas colunas usadas em `findBy` query methods
- [ ] Usar `@EntityGraph` ou `fetch = EAGER` com cuidado em projeções com relacionamentos

### Testes

- [ ] Testar endpoints Spring Data REST com `@SpringBootTest` + banco de dados real (H2/Testcontainers)
- [ ] Verificar presença dos links `_links.self`, `_links.next`, etc. nos testes
- [ ] Testar projeções e excerpts separadamente
- [ ] Testar event handlers (`@HandleBeforeCreate`, etc.) via integração

---

> **Leitura complementar:**
> - `Dicas-Spring-MVC-REST-SSR.md` — Controllers REST manuais com Spring MVC
> - `Dicas-JPA-Hibernate-Modelagem-Entities.md` — Modelagem de entidades JPA
> - `Dicas-Spring-Security.md` — Configuração detalhada de segurança
