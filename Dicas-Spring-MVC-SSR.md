# Spring MVC — Guia SSR com Thymeleaf

> Derivado de **Dicas-Spring-MVC-REST-SSR.md** — contém apenas o conteúdo referente
> a SSR (Server-Side Rendering) com Thymeleaf.

## Sumário

1. [Arquitetura do Spring MVC](#1-arquitetura-do-spring-mvc)
    - [1.1 Ciclo de Vida de uma Requisição](#11-ciclo-de-vida-de-uma-requisição)
    - [1.2 Componentes Principais](#12-componentes-principais)
    - [1.3 Fluxo Visual — SSR (Thymeleaf)](#13-fluxo-visual--ssr-thymeleaf)
    - [1.5 Diferença Fundamental: REST vs MVC SSR](#15-diferença-fundamental-rest-vs-mvc-ssr)
2. [Configuração Base](#2-configuração-base)
    - [2.1 Dependências Maven](#21-dependências-maven)
    - [2.2 Configuração MVC Centralizada](#22-configuração-mvc-centralizada)
    - [2.3 application.yml — Configuração Recomendada](#23-applicationyml--configuração-recomendada)
3. [Anotações do Controller — Referência Rápida](#3-anotações-do-controller--referência-rápida)
    - [3.1 Anotações de Classe — Definição do Controller](#31-anotações-de-classe--definição-do-controller)
    - [3.2 Anotações de Método — Mapeamento de Requisições](#32-anotações-de-método--mapeamento-de-requisições)
    - [3.3 Anotações de Método — Controle da Resposta](#33-anotações-de-método--controle-da-resposta)
    - [3.4 Anotações de Parâmetro — Captura de Dados da Requisição](#34-anotações-de-parâmetro--captura-de-dados-da-requisição)
    - [3.5 Anotações de Parâmetro — Objetos Especiais do Spring MVC](#35-anotações-de-parâmetro--objetos-especiais-do-spring-mvc)
    - [3.6 Anotações de Classe/Método — @ControllerAdvice](#36-anotações-de-classemétodo--controlleradvice)
    - [3.7 Visão Consolidada — Contexto por Anotação](#37-visão-consolidada--contexto-por-anotação)
    - [3.8 Tipos de Retorno dos Métodos de Controller](#38-tipos-de-retorno-dos-métodos-de-controller)
4. [Controllers MVC com Thymeleaf (SSR)](#4-controllers-mvc-com-thymeleaf-ssr)
    - [4.1 Controller MVC Clássico](#41-controller-mvc-clássico)
    - [4.2 Form Object (Separado do Domain/DTO)](#42-form-object-separado-do-domaindto)
    - [4.3 Templates Thymeleaf](#43-templates-thymeleaf)
    - [4.4 Convertendo ConstraintViolationException do Service em BindingResult](#44-convertendo-constraintviolationexception-do-service-em-bindingresult)
    - [4.5 Atributos de Modelo Compartilhados com @ModelAttribute](#45-atributos-de-modelo-compartilhados-com-modelattribute)
    - [4.6 @SessionAttributes e @SessionAttribute](#46-sessionattributes-e-sessionattribute)
5. [Bean Validation — @Valid vs @Validated](#5-bean-validation--valid-vs-validated)
    - [5.1 Diferença Conceitual](#51-diferença-conceitual)
    - [5.2 Exemplo Prático de Grupos de Validação](#52-exemplo-prático-de-grupos-de-validação)
    - [5.3 Validação em Serviços com @Validated](#53-validação-em-serviços-com-validated)
    - [5.4 Cascata de Validação com @Valid](#54-cascata-de-validação-com-valid)
    - [5.5 Constraint Customizada](#55-constraint-customizada)
    - [5.6 Constraint com Acesso a Banco (Spring Bean)](#56-constraint-com-acesso-a-banco-spring-bean)
    - [5.7 Atributo payload nas Constraints](#57-atributo-payload-nas-constraints)
6. [InitBinder](#6-initbinder)
    - [6.1 Usos Comuns](#61-usos-comuns)
    - [6.2 PropertyEditor Customizado](#62-propertyeditor-customizado)
    - [6.3 Validator Programático com @InitBinder](#63-validator-programático-com-initbinder)
7. [Converters e Formatters](#7-converters-e-formatters)
    - [7.1 Diferenças entre os Tipos](#71-diferenças-entre-os-tipos)
    - [7.2 Converter — String para Enum Genérico](#72-converter--string-para-enum-genérico)
    - [7.3 Converter — ID para Entidade JPA](#73-converter--id-para-entidade-jpa)
    - [7.4 Formatter — Moeda Brasileira com Locale](#74-formatter--moeda-brasileira-com-locale)
    - [7.5 Formatter para LocalDate Brasileiro](#75-formatter-para-localdate-brasileiro)
    - [7.6 Registro dos Converters/Formatters](#76-registro-dos-convertersformatters)
8. [Tratamento de Erros](#8-tratamento-de-erros)
    - [8.1 Hierarquia de Exceções](#81-hierarquia-de-exceções)
    - [8.2 Tratamento de Erros em SSR (Páginas de Erro Thymeleaf)](#82-tratamento-de-erros-em-ssr-páginas-de-erro-thymeleaf)
9. [Recursos Avançados e Pouco Explorados](#9-recursos-avançados-e-pouco-explorados)
    - [9.1 HandlerInterceptor — Auditoria e Métricas](#91-handlerinterceptor--auditoria-e-métricas)
    - [9.2 @ModelAttribute Global com @ControllerAdvice](#92-modelattribute-global-com-controlleradvice)
    - [9.3 HandlerMethodArgumentResolver — Argumento Customizado](#93-handlermethodargumentresolver--argumento-customizado)
    - [9.4 @RequestScope e @SessionScope Beans](#94-requestscope-e-sessionscope-beans)
    - [9.5 Flash Attributes — Dados entre Redirects (PRG Pattern)](#95-flash-attributes--dados-entre-redirects-prg-pattern)
    - [9.6 Controller Assíncrono — CompletableFuture, Callable e DeferredResult](#96-controller-assíncrono--completablefuture-callable-e-deferredresult)
    - [9.7 Acesso a Recursos do Servlet — HttpServletRequest, HttpServletResponse e RequestContextHolder](#97-acesso-a-recursos-do-servlet--httpservletrequest-httpservletresponse-e-requestcontextholder)
    - [9.8 Integração com Spring Security](#98-integração-com-spring-security)
10. [Alternativas ao Thymeleaf](#10-alternativas-ao-thymeleaf)
    - [10.1 Comparativo de Engines de Template](#101-comparativo-de-engines-de-template)
    - [10.2 FreeMarker](#102-freemarker)
    - [10.3 JTE (Java Template Engine) — Recomendado para Performance](#103-jte-java-template-engine--recomendado-para-performance)
    - [10.4 Quando Usar Cada Template Engine](#104-quando-usar-cada-template-engine)
11. [Boas Práticas e Checklist](#11-boas-práticas-e-checklist)
    - [11.1 Diagrama de Fluxo de Decisão](#111-diagrama-de-fluxo-de-decisão)
    - [11.2 Checklist — SSR Controllers](#112-checklist--ssr-controllers)
    - [11.3 Checklist — Validação](#113-checklist--validação)
    - [11.4 Anti-patterns a Evitar](#114-anti-patterns-a-evitar)
    - [11.5 Mensagens de Validação i18n — Integração com messages.properties](#115-mensagens-de-validação-i18n--integração-com-messagesproperties)
12. [ETag e Cache HTTP](#12-etag-e-cache-http)
    - [12.1 Cache HTTP em Views SSR](#121-cache-http-em-views-ssr)
    - [12.2 Resumo: Quando Usar Cada Estratégia](#122-resumo-quando-usar-cada-estratégia)
13. [Upload de Arquivos](#13-upload-de-arquivos)
    - [13.1 Configuração](#131-configuração)
    - [13.2 Controller de Upload](#132-controller-de-upload)
    - [13.3 Service — Estratégias de Armazenamento](#133-service--estratégias-de-armazenamento)
    - [13.4 Download de Arquivos](#134-download-de-arquivos)
    - [13.5 Upload via Fetch API (JavaScript)](#135-upload-via-fetch-api-javascript)
14. [Internacionalização (i18n)](#14-internacionalização-i18n)
    - [14.1 Estratégias de Resolução de Locale](#141-estratégias-de-resolução-de-locale)
    - [14.2 Configuração Completa](#142-configuração-completa)
    - [14.3 Arquivos de Mensagens](#143-arquivos-de-mensagens)
    - [14.4 i18n em Controllers REST](#144-i18n-em-controllers-rest)
    - [14.5 i18n em Templates Thymeleaf](#145-i18n-em-templates-thymeleaf)
    - [14.6 Controller SSR — enviando chave em vez de texto](#146-controller-ssr--enviando-chave-em-vez-de-texto)
    - [14.7 i18n em Respostas JSON — MessageSourceAccessor](#147-i18n-em-respostas-json--messagesourceaccessor)
    - [14.8 Timezone — Integração com i18n](#148-timezone--integração-com-i18n)
    - [14.9 LocaleContextHolder — acesso ao Locale fora do Controller](#149-localecontextholder--acesso-ao-locale-fora-do-controller)
15. [Customização do ErrorController](#15-customização-do-errorcontroller)
    - [15.1 Como o Fluxo de Erro Funciona](#151-como-o-fluxo-de-erro-funciona)
    - [15.2 Customizando o BasicErrorController](#152-customizando-o-basicerrorcontroller)
    - [15.3 Templates de Erro Thymeleaf](#153-templates-de-erro-thymeleaf)
    - [15.4 Configuração via application.yml](#154-configuração-via-applicationyml)
16. [@ResponseStatus em Classes de Exceção](#16-responsestatus-em-classes-de-exceção)
    - [16.1 Uso Básico](#161-uso-básico)
    - [16.2 @ResponseStatus vs @ExceptionHandler — Quando Usar Cada Um](#162-responsestatus-vs-exceptionhandler--quando-usar-cada-um)
    - [16.3 Precedência com @ControllerAdvice](#163-precedência-com-controlleradvice)
17. [MultiValueMap e Form Data](#17-multivaluemap-e-form-data)
    - [17.1 MultiValueMap — Múltiplos Valores por Chave](#171-multivaluemap--múltiplos-valores-por-chave)
    - [17.2 Form Data com Checkboxes (multi-select)](#172-form-data-com-checkboxes-multi-select)
    - [17.3 LinkedMultiValueMap — Construção Programática](#173-linkedmultivaluemap--construção-programática)
18. [RedirectView e UrlBasedViewResolver](#18-redirectview-e-urlbasedviewresolver)
    - [18.1 RedirectView — Redirect Programático com Controle Total](#181-redirectview--redirect-programático-com-controle-total)
    - [18.2 UrlBasedViewResolver — Configuração Avançada de View Resolution](#182-urlbasedviewresolver--configuração-avançada-de-view-resolution)
    - [18.3 Redirect 301 Permanente — SEO e Mudança de URL](#183-redirect-301-permanente--seo-e-mudança-de-url)
    - [18.4 Forward Interno — Compartilhamento de Handlers](#184-forward-interno--compartilhamento-de-handlers)
19. [Testes](#19-testes)
    - [19.1 Visão Geral — Pirâmide de Testes no Spring MVC](#191-visão-geral--pirâmide-de-testes-no-spring-mvc)
    - [19.2 Teste Unitário — Service sem Spring](#192-teste-unitário--service-sem-spring)
    - [19.3 @WebMvcTest — Slice Test da Camada Web](#193-webmvctest--slice-test-da-camada-web)
    - [19.4 @WebMvcTest com Spring Security](#194-webmvctest-com-spring-security)
    - [19.5 @SpringBootTest — Teste de Integração](#195-springboottest--teste-de-integração)
    - [19.6 Configuração de Contexto de Teste](#196-configuração-de-contexto-de-teste)
    - [19.7 Testando Upload, CORS e SSE](#197-testando-upload-cors-e-sse)
20. [Tópicos Relevantes Não Cobertos Neste Documento](#20-tópicos-relevantes-não-cobertos-neste-documento)
    - [20.1 Tópicos Ausentes — Alta Relevância](#201-tópicos-ausentes--alta-relevância)
    - [20.2 Tópicos Ausentes — Relevância Moderada](#202-tópicos-ausentes--relevância-moderada)
    - [20.3 Tópicos Ausentes — Relevância Menor mas Notáveis](#203-tópicos-ausentes--relevância-menor-mas-notáveis)
    - [20.4 Resumo por Prioridade](#204-resumo-por-prioridade)
- [Referências e Créditos](#referências-e-créditos)

---

## 1. Arquitetura do Spring MVC

> **Escopo deste documento — Spring MVC (Servlet Stack)**
>
> Este documento cobre exclusivamente o **Spring MVC**, o módulo web baseado na
> Servlet API (`spring-webmvc`). Ele opera sobre um modelo de concorrência
> **thread-per-request** (bloqueante) e é a stack padrão do
> `spring-boot-starter-web`.
>
> O Spring Framework oferece uma segunda stack web — **Spring WebFlux**
> (`spring-webflux`, `spring-boot-starter-webflux`) — baseada em programação
> reativa com Project Reactor (`Mono<T>`, `Flux<T>`). WebFlux opera de forma
> não-bloqueante sobre Netty (ou Servlet 3.1+ assíncrono) e é adequado para
> cenários de alta concorrência com I/O intensivo.
>
> As duas stacks **não coexistem** na mesma aplicação Spring Boot: a presença de
> `spring-boot-starter-webflux` no classpath sem `spring-boot-starter-web`
> ativa o WebFlux; a presença de ambos mantém o MVC como padrão.
>
> | | Spring MVC | Spring WebFlux |
> |---|---|---|
> | Modelo de execução | Thread-per-request (bloqueante) | Event loop (não-bloqueante) |
> | API de retorno | `ResponseEntity<T>`, `String`, `ModelAndView` | `Mono<T>`, `Flux<T>` |
> | Servidor padrão | Tomcat (Servlet) | Netty |
> | Starter (Boot 3.x) | `spring-boot-starter-web` | `spring-boot-starter-webflux` |
> | Starter (Boot 4.x) | `spring-boot-starter-webmvc` ¹ | `spring-boot-starter-webflux` |
> | Java 21+ Virtual Threads | ✅ Recomendado — simplifica throughput | Não necessário |
> | **Coberto neste doc** | ✅ Sim | ❌ Não |
>
> ¹ No Spring Boot 4 o starter foi renomeado de `spring-boot-starter-web` para
> `spring-boot-starter-webmvc`, tornando explícito que ele ativa a stack MVC
> (Servlet). O nome antigo ainda funciona como alias para compatibilidade, mas
> o nome canônico nos novos projetos é `spring-boot-starter-webmvc`.

### 1.1 Ciclo de Vida de uma Requisição

```mermaid
sequenceDiagram
    participant C as Cliente (Browser / API)
    participant DS as DispatcherServlet
    participant HM as HandlerMapping
    participant HI as HandlerInterceptor
    participant HC as HandlerAdapter
    participant CT as Controller
    participant VR as ViewResolver
    participant V as View (Thymeleaf)

    C->>DS: HTTP Request
    DS->>HM: getHandler(request)
    HM-->>DS: HandlerExecutionChain
    DS->>HI: preHandle()
    HI-->>DS: true (continua)
    DS->>HC: handle(request, response, handler)
    HC->>CT: método anotado
    CT-->>HC: ModelAndView / @ResponseBody
    HC-->>DS: ModelAndView
    DS->>HI: postHandle()
    alt REST (@ResponseBody)
        DS->>C: JSON/XML serializado
    else SSR (View Name)
        DS->>VR: resolveViewName()
        VR-->>DS: View
        DS->>V: render(model, request, response)
        V-->>C: HTML renderizado
    end
    DS->>HI: afterCompletion()
```

### 1.2 Componentes Principais

```mermaid
graph TB
    subgraph "Camada Web"
        DS["DispatcherServlet<br/>Front Controller"]
        HM["HandlerMapping<br/>RequestMappingHandlerMapping"]
        HA["HandlerAdapter<br/>RequestMappingHandlerAdapter"]
        VR["ViewResolver<br/>ThymeleafViewResolver"]
    end

    subgraph "Suporte"
        HI["HandlerInterceptor"]
        EH["ExceptionHandler<br/>@ControllerAdvice"]
        MC["MessageConverter<br/>JSON/XML/Form"]
        DT["DataBinder<br/>InitBinder + Formatters"]
    end

    subgraph "Controllers"
        RC["@RestController<br/>REST APIs"]
        VC["@Controller<br/>SSR + Thymeleaf"]
    end

    DS --> HM --> HA
    HA --> RC
    HA --> VC
    HA --> DT
    DS --> HI
    DS --> EH
    HA --> MC
    VC --> VR
```

### 1.3 Fluxo Visual — SSR (Thymeleaf)

O diagrama abaixo representa o ciclo completo de uma requisição SSR, com cada componente e o fluxo de dados entre eles — padrão clássico de Front Controller.

```mermaid
flowchart LR
    Browser(["🌐 Browser<br>HTTP client"])

    subgraph SERVLET ["  Servlet container (e.g. Tomcat)  "]
        direction LR
        DS["DispatcherServlet<br>Front controller"]
        CTRL["@Controller<br>Handles request"]
        VR["ViewResolver"]
        VIEW["View template<br>Thymeleaf / JTE"]

        DS -- "① delegate request" --> CTRL
        CTRL -. "② model" .-> DS
        DS -- "③ view name + model" --> VR
        VR -- "④ render(model)" --> VIEW
        VIEW -. "⑤ HTML renderizado" .-> DS
    end

    MODEL(["Service /<br>Repository<br>MODEL"])

    Browser -- "HTTP request" --> DS
    DS -- "HTML response" --> Browser
    CTRL -- "call" --> MODEL
    MODEL -. "data" .-> CTRL

    style DS fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style CTRL fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style VR fill:#F1EFE8,stroke:#5F5E5A,color:#2C2C2A
    style VIEW fill:#EAF3DE,stroke:#3B6D11,color:#27500A
    style MODEL fill:#FAECE7,stroke:#993C1D,color:#4A1B0C
    style Browser fill:#F1EFE8,stroke:#5F5E5A,color:#2C2C2A
```

**Passos do fluxo SSR:**

| Passo | Ação |
|-------|------|
| ① DS → @Controller | Delega via `HandlerMapping` ao handler correto |
| ② @Controller → DS | Retorna `ModelAndView` com dados e nome da view |
| ③ DS → ViewResolver | Resolve o nome lógico para uma `View` concreta |
| ④ ViewResolver → Thymeleaf | Template engine renderiza o HTML com o `Model` |
| ⑤ View → DS | HTML pronto é enviado ao browser |

### 1.5 Diferença Fundamental: REST vs MVC SSR

| Aspecto | REST (`@RestController`) | SSR (`@Controller` + Thymeleaf) |
|---|---|---|
| Retorno | Objeto serializado (JSON/XML) | Nome da view ou `ModelAndView` |
| Cliente | SPA, mobile, outro serviço | Browser (requisição completa) |
| Estado | Stateless (JWT/OAuth2) | Session ou stateless |
| Formulários | Não aplicável | `@ModelAttribute` + BindingResult |
| Validação | `@Valid` no `@RequestBody` | `@Valid` no `@ModelAttribute` |
| Redirect | `ResponseEntity` com Location | `"redirect:/caminho"` |

---



## 2. Configuração Base

### 2.1 Dependências Maven

A tabela abaixo resume o que cada starter ativa automaticamente via
auto-configuração do Spring Boot — sem nenhuma linha de código adicional:

| Starter | Auto-configurações ativadas |
|---|---|
| `spring-boot-starter-web` | `DispatcherServlet`, `Jackson`, `Tomcat`, `CharacterEncodingFilter`, `HiddenHttpMethodFilter`, recursos estáticos, `ContentNegotiationStrategy` |
| `spring-boot-starter-validation` | `LocalValidatorFactoryBean` (Bean Validation), `MethodValidationPostProcessor` |
| `spring-boot-starter-thymeleaf` | `ThymeleafViewResolver`, `SpringTemplateEngine`, `ClassLoaderTemplateResolver` |
| `spring-boot-starter-data-jpa` | `PageableHandlerMethodArgumentResolver` (Pageable em controllers), `SortHandlerMethodArgumentResolver` |
| `springdoc-openapi-starter-webmvc-ui` | Endpoint `/api-docs`, `/swagger-ui.html`, `OpenApiWebMvcResource` |
| `spring-boot-starter-actuator` | Endpoints `/actuator/*`, métricas, health |

```xml
<!-- REST APIs -->
<!-- ✅ Auto-configura: DispatcherServlet, Jackson ObjectMapper, Tomcat, CORS básico -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- ✅ Auto-configura: LocalValidatorFactoryBean e MethodValidationPostProcessor -->
<!-- ⚠️  NÃO conecta o validador ao MessageSource do Spring (ver seção 13.6) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- SSR com Thymeleaf -->
<!-- ✅ Auto-configura: ThymeleafViewResolver, SpringTemplateEngine, templates em /templates/ -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!-- ❌ Não auto-configurado: precisa ser registrado manualmente como bean TemplateEngine -->
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>

<!-- ❌ Não auto-configurado: requer declaração explícita em ThymeleafConfig -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>

<!-- Webjars para SSR (Bootstrap, Font Awesome) -->
<!-- ✅ Auto-configura: /webjars/** mapeado automaticamente pelo ResourceHandlerRegistry -->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-lite</artifactId>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>5.3.8</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>font-awesome</artifactId>
    <version>7.2.0</version>
</dependency>
```

### 2.2 Configuração MVC Centralizada

> **`@EnableWebMvc` vs `WebMvcConfigurer`**
>
> O Spring Boot auto-configura todo o Spring MVC via `WebMvcAutoConfiguration`.
> Ao usar `WebMvcConfigurer` (sem `@EnableWebMvc`), você *adiciona* comportamento
> sem quebrar a auto-configuração — é a abordagem recomendada.
> `@EnableWebMvc` **desativa** a auto-configuração do Boot e exige que tudo seja
> configurado manualmente (sem defaults de Jackson, sem `ResourceHandlers`, etc.).
> Use `@EnableWebMvc` apenas se precisar de controle absoluto sobre a stack MVC.

```java
@Configuration
// @EnableWebMvc  ← EVITE: desativa WebMvcAutoConfiguration e todos os seus defaults
//                  Use apenas se precisar substituir completamente a stack MVC.
//                  Com Spring Boot, prefira apenas WebMvcConfigurer sem esta anotação.
public class WebMvcConfig implements WebMvcConfigurer {

    // ─── Recursos estáticos ──────────────────────────────────────────────────
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/")
                .setCacheControl(CacheControl.maxAge(1, TimeUnit.HOURS).cachePublic());

        // Webjars com versionamento automático
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .resourceChain(true)
                .addResolver(new WebJarsResourceResolver());
    }

    // ─── CORS global ─────────────────────────────────────────────────────────
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("https://app.example.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }

    // ─── Interceptors ────────────────────────────────────────────────────────
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuditInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/health");

        registry.addInterceptor(new LocaleChangeInterceptor())
                .addPathPatterns("/**");
    }

    // ─── Resolução de locale ──────────────────────────────────────────────────
    @Bean
    public LocaleResolver localeResolver() {
        CookieLocaleResolver resolver = new CookieLocaleResolver("APP_LOCALE");
        resolver.setDefaultLocale(new Locale("pt", "BR"));
        return resolver;
    }

    // ─── Formatters e Converters (seção 7) ───────────────────────────────────
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToMoneyConverter());
        registry.addFormatter(new BrazilianDateFormatter());
        registry.addConverterFactory(new StringToEnumConverterFactory());
    }

    // ─── Content Negotiation ──────────────────────────────────────────────────
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer
            .favorParameter(true)             // ?format=json
            .parameterName("format")
            .ignoreAcceptHeader(false)
            .defaultContentType(MediaType.APPLICATION_JSON)
            .mediaType("json", MediaType.APPLICATION_JSON)
            .mediaType("xml", MediaType.APPLICATION_XML);
    }

    // ─── View Controllers — redirecionamentos e views sem lógica ────────────
    //
    // addViewControllers registra mapeamentos diretos URL→view ou URL→redirect
    // sem precisar de um @Controller. Ideal para:
    //   - Páginas estáticas (sobre, termos de uso, manutenção)
    //   - Redirects permanentes de URLs antigas
    //   - Respostas de status sem corpo (503 em manutenção)
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // Renderiza uma view Thymeleaf sem nenhum controller ou model
        registry.addViewController("/").setViewName("home");
        registry.addViewController("/login").setViewName("auth/login");
        registry.addViewController("/sobre").setViewName("institucional/sobre");
        registry.addViewController("/termos").setViewName("institucional/termos");

        // Redirect permanente (301) — troca de URL sem perder SEO
        registry.addRedirectViewController("/home", "/")
                .setPermanent(true);

        // Redirect temporário (302) — padrão quando omitido setPermanent
        registry.addRedirectViewController("/admin", "/admin/dashboard");

        // Status puro — sem body, sem view (ex.: modo manutenção)
        // Útil combinado com um filtro que bloqueia as demais rotas
        registry.addStatusController("/health/ping", HttpStatus.OK);
    }

    // ─── Async / Virtual Threads ──────────────────────────────────────────────
    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        configurer.setDefaultTimeout(30_000L);
        // Com Virtual Threads (Java 21+), o Executor já é configurado automaticamente
        // pelo Spring Boot quando spring.threads.virtual.enabled=true
    }
}
```

### 2.3 application.yml — Configuração Recomendada

Cada propriedade abaixo está marcada com o que o Spring Boot faz por padrão
quando a propriedade **não** é declarada:

```yaml
# ─── Servidor — EmbeddedWebServerFactoryCustomizerAutoConfiguration ───────────
server:
  # ✅ Default: 8080
  port: 8080

  servlet:
    # ✅ Default: "" (raiz — sem prefixo)
    # Prefixo global aplicado a TODOS os endpoints, incluindo Actuator e Swagger.
    # Ex.: context-path: /app  →  http://localhost:8080/app/api/v1/produtos
    # ⚠️  Diferente de spring.mvc.servlet.path, que só afeta o DispatcherServlet.
    context-path: /

spring:
  # ─── MVC — WebMvcAutoConfiguration ──────────────────────────────────────────
  mvc:
    # ✅ Default: false — lança NoHandlerFoundException mapeável pelo @ControllerAdvice
    throw-exception-if-no-handler-found: true

    # ✅ Default: /** (todos os recursos estáticos em /static, /public, /resources, /META-INF/resources)
    static-path-pattern: /static/**

    # ✅ Defaults: nenhum formato pré-configurado (datas serializam como timestamp)
    format:
      date: yyyy-MM-dd
      date-time: yyyy-MM-dd'T'HH:mm:ss

  # ─── Thymeleaf — ThymeleafAutoConfiguration ──────────────────────────────────
  thymeleaf:
    cache: false            # ✅ Default: true — SEMPRE false em dev, true em prod
    mode: HTML              # ✅ Default: HTML
    encoding: UTF-8         # ✅ Default: UTF-8
    prefix: classpath:/templates/  # ✅ Default: classpath:/templates/
    suffix: .html           # ✅ Default: .html
    servlet:
      content-type: text/html;charset=UTF-8  # ✅ Default: text/html;charset=UTF-8

  # ─── Jackson — JacksonAutoConfiguration ──────────────────────────────────────
  jackson:
    # ✅ Default: ALWAYS (inclui nulls) — non_null é recomendado para APIs limpas
    default-property-inclusion: non_null
    serialization:
      write-dates-as-timestamps: false  # ✅ Default: true — false para ISO 8601
      indent-output: false              # ✅ Default: false
    deserialization:
      fail-on-unknown-properties: false # ✅ Default: false (Boot 2.3+)
    time-zone: America/Sao_Paulo        # ✅ Default: UTC

  # ─── Multipart — MultipartAutoConfiguration ──────────────────────────────────
  servlet:
    multipart:
      enabled: true           # ✅ Default: true
      max-file-size: 10MB     # ✅ Default: 1MB — ajuste conforme necessidade
      max-request-size: 50MB  # ✅ Default: 10MB

  # ─── Virtual Threads — TomcatVirtualThreadsWebServerFactoryCustomizer ────────
  threads:
    virtual:
      enabled: true           # ✅ Default: false — habilitar em prod com Java 21+

  # ─── MessageSource — MessageSourceAutoConfiguration ──────────────────────────
  messages:
    basename: messages        # ✅ Default: messages (lê messages*.properties)
    encoding: UTF-8           # ✅ Default: UTF-8
    cache-duration: 1s        # ✅ Default: sem cache (recarrega a cada acesso em dev)
    use-code-as-default-message: false  # ✅ Default: false — lança exceção se chave não existe

  # ─── Spring Data Web — SpringDataWebAutoConfiguration ────────────────────────
  data:
    web:
      pageable:
        default-page-size: 20      # ✅ Default: 20
        max-page-size: 100         # ✅ Default: 2000 — SEMPRE reduzir em produção
        one-indexed-parameters: false  # ✅ Default: false (página começa em 0)

```

---



## 3. Anotações do Controller — Referência Rápida

Esta seção apresenta as principais anotações do Spring MVC usadas em controllers,
agrupadas por categoria. Para cada anotação são indicados: onde pode ser aplicada
(classe, método ou parâmetro), o contexto de uso (REST, SSR ou ambos) e uma
breve descrição.

**Legenda de contexto:**
- **Geral** — aplicável a REST e SSR sem restrição
- **REST** — voltada para APIs que retornam JSON/XML
- **SSR** — voltada para controllers que renderizam templates (ex.: Thymeleaf)

---

### 3.1 Anotações de Classe — Definição do Controller

| Anotação | Contexto | Alvo | Descrição |
|----------|----------|------|-----------|
| `@Controller` | Geral | Classe | Marca a classe como controller Spring MVC. Métodos podem retornar nomes de views ou `@ResponseBody`. |
| `@RestController` | REST | Classe | Atalho para `@Controller` + `@ResponseBody`. Todos os métodos serializam o retorno para JSON/XML. |
| `@RequestMapping` | Geral | Classe / Método | Define o prefixo de URL para todos os métodos do controller. Na classe, estabelece a raiz do path. |
| `@Validated` | Geral | Classe | Habilita validação de parâmetros simples (`@PathVariable`, `@RequestParam`) via Bean Validation. Necessário para `@NotNull`, `@Min` etc. fora de `@RequestBody`. |
| `@SessionAttributes` | SSR | Classe | Mantém atributos do `Model` na sessão HTTP entre requisições. Útil em formulários multi-etapa. |
| `@CrossOrigin` | REST | Classe / Método | Habilita CORS para o controller ou método específico. Equivalente pontual à configuração global de CORS. |

```java
// REST — retorno sempre serializado para JSON
@RestController
@RequestMapping("/api/v1/produtos")
@Validated
@CrossOrigin(origins = "https://meusite.com")
public class ProdutoController { ... }

// SSR — retorno é nome de view (ex.: "produtos/lista")
@Controller
@RequestMapping("/produtos")
@SessionAttributes("produtoForm")
public class ProdutoMvcController { ... }
```

---

### 3.2 Anotações de Método — Mapeamento de Requisições

| Anotação | Contexto | Alvo | Descrição |
|----------|----------|------|-----------|
| `@GetMapping` | Geral | Método | Atalho para `@RequestMapping(method = GET)`. |
| `@PostMapping` | Geral | Método | Atalho para `@RequestMapping(method = POST)`. |
| `@PutMapping` | REST | Método | Atalho para `@RequestMapping(method = PUT)`. Substitui o recurso inteiro. |
| `@PatchMapping` | REST | Método | Atalho para `@RequestMapping(method = PATCH)`. Atualização parcial. |
| `@DeleteMapping` | REST | Método | Atalho para `@RequestMapping(method = DELETE)`. |
| `@RequestMapping` | Geral | Método | Mapeamento genérico — use quando precisar de mais de um método HTTP ou configurar `consumes`/`produces`. |

```java
@GetMapping                                  // GET /api/v1/produtos
@GetMapping("/{id}")                         // GET /api/v1/produtos/{id}
@PostMapping(consumes = "application/json")  // POST com body JSON
@PutMapping("/{id}")                         // PUT /api/v1/produtos/{id}
@PatchMapping("/{id}")                       // PATCH /api/v1/produtos/{id}
@DeleteMapping("/{id}")                      // DELETE /api/v1/produtos/{id}

// Quando precisar de mais controle:
@RequestMapping(value = "/export", method = {GET, HEAD},
                produces = "text/csv")
```

---

### 3.3 Anotações de Método — Controle da Resposta

| Anotação | Contexto | Alvo | Descrição |
|----------|----------|------|-----------|
| `@ResponseBody` | REST | Método / Classe | Serializa o retorno do método para o corpo da resposta HTTP (JSON, XML etc). Implícito em `@RestController`. |
| `@ResponseStatus` | Geral | Método / Classe | Define o status HTTP padrão da resposta. Em classes de exceção, elimina a necessidade de `@ExceptionHandler`. |
| `@ModelAttribute` | SSR | Método | Método cujo retorno é adicionado ao `Model` antes de qualquer handler do controller ser chamado. |
| `@InitBinder` | Geral | Método | Inicializa o `WebDataBinder` para o controller — registra `PropertyEditors`, `Validators` e formatadores customizados. |
| `@ExceptionHandler` | Geral | Método | Captura exceções lançadas pelo controller (ou por toda a aplicação se em `@ControllerAdvice`). |

```java
// Status customizado — sem precisar de ResponseEntity
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public ProdutoResponse criar(@RequestBody @Valid ProdutoRequest req) { ... }

// Método que pré-popula o Model para todas as views do controller
@ModelAttribute("categorias")
public List<Categoria> popularCategorias() {
    return categoriaService.listarAtivas();
}

// Tratamento de erro local (só para este controller)
@ExceptionHandler(ProdutoNaoEncontradoException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public ProblemDetail handleNotFound(ProdutoNaoEncontradoException ex) { ... }
```

---

### 3.4 Anotações de Parâmetro — Captura de Dados da Requisição

| Anotação | Contexto | Alvo | Descrição |
|----------|----------|------|-----------|
| `@PathVariable` | Geral | Parâmetro | Captura segmento de URL: `/produtos/{id}` → `@PathVariable Long id`. |
| `@RequestParam` | Geral | Parâmetro | Captura parâmetro de query string ou form data: `?page=0`. Aceita `required`, `defaultValue`. |
| `@RequestBody` | REST | Parâmetro | Desserializa o corpo da requisição (JSON/XML) para o tipo do parâmetro. |
| `@ModelAttribute` | SSR | Parâmetro | Faz o binding de form data (HTML form) para um objeto Java. |
| `@RequestHeader` | Geral | Parâmetro | Captura um header HTTP específico: `Authorization`, `Accept-Language` etc. |
| `@CookieValue` | Geral | Parâmetro | Captura o valor de um cookie pelo nome. |
| `@RequestPart` | REST | Parâmetro | Captura uma parte de requisição `multipart/form-data` (arquivo ou JSON). |
| `@Valid` / `@Validated` | Geral | Parâmetro | Aciona o Bean Validation no objeto recebido. `@Valid` para cascata; `@Validated` para grupos. |
| `@SessionAttribute` | SSR | Parâmetro | Recupera um atributo específico da sessão HTTP. |
| `@RequestAttribute` | Geral | Parâmetro | Recupera um atributo do `HttpServletRequest` (definido por filtro ou interceptor). |

```java
@GetMapping("/{id}")
public ResponseEntity<ProdutoResponse> buscar(
        @PathVariable Long id) { ... }

@GetMapping
public Page<ProdutoResponse> listar(
        @RequestParam(defaultValue = "0")  int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(required = false)    String busca) { ... }

@PostMapping
public ResponseEntity<ProdutoResponse> criar(
        @RequestBody @Valid ProdutoRequest request) { ... }

// SSR — binding de formulário HTML
@PostMapping
public String salvar(
        @ModelAttribute @Valid ProdutoForm form,
        BindingResult binding) { ... }

@GetMapping("/export")
public ResponseEntity<byte[]> exportar(
        @RequestHeader("Accept-Language") String lang,
        @CookieValue(name = "APP_LOCALE", required = false) String locale) { ... }

@PostMapping("/upload")
public ResponseEntity<String> upload(
        @RequestPart("arquivo") MultipartFile arquivo,
        @RequestPart("dados")   @Valid ProdutoRequest dados) { ... }
```

---

### 3.5 Anotações de Parâmetro — Objetos Especiais do Spring MVC

Esses tipos são injetados automaticamente pelo Spring MVC como parâmetros de
método — sem necessidade de anotação.

| Tipo | Contexto | Descrição |
|------|----------|-----------|
| `HttpServletRequest` | Geral | Acesso direto ao request HTTP (headers, cookies, attributes). Prefira as anotações acima quando possível. |
| `HttpServletResponse` | Geral | Acesso direto à resposta HTTP. Útil para streaming ou cookies programáticos. |
| `HttpSession` | Geral | Acesso direto à sessão HTTP. Prefira `@SessionAttributes`/`@SessionAttribute` em SSR; use diretamente apenas quando precisar de controle explícito (invalidar sessão, iterar atributos). |
| `BindingResult` | Geral | Resultado do Bean Validation (`@Valid`/`@Validated`). Deve ser declarado imediatamente após o parâmetro validado. |
| `Model` / `ModelMap` | SSR | Mapa de atributos enviados à view. Alternativa a `ModelAndView`. |
| `RedirectAttributes` | SSR | Atributos para redirect (flash attributes). Disponível na próxima requisição. |
| `Locale` | Geral | Locale resolvido pelo `LocaleResolver` para a requisição atual. |
| `TimeZone` / `ZoneId` | Geral | TimeZone resolvido pelo `LocaleResolver`. |
| `Principal` | Geral | Usuário autenticado (Spring Security ou container). |
| `@AuthenticationPrincipal` | Geral | Extrai o objeto de usuário do `SecurityContext` diretamente como parâmetro. |
| `Pageable` | REST | Parâmetros de paginação do Spring Data (`page`, `size`, `sort`) desserializados automaticamente. |
| `UriComponentsBuilder` | REST | Constrói URIs de forma programática e tipada. Injetado com o base URL do request atual. Use `ServletUriComponentsBuilder` para herdar scheme/host/port/context-path automaticamente. |

```java
@PostMapping
public String salvar(
        @ModelAttribute @Valid ProdutoForm form,
        BindingResult binding,           // ← imediatamente após @ModelAttribute/@RequestBody
        Model model,
        RedirectAttributes redirectAttrs,
        Locale locale) {

    if (binding.hasErrors()) {
        model.addAttribute("categorias", categoriaService.listar());
        return "produtos/formulario";
    }
    produtoService.criar(form);
    redirectAttrs.addFlashAttribute("mensagem", "produto.criado");
    return "redirect:/produtos";
}
```

---

### 3.6 Anotações de Classe/Método — `@ControllerAdvice`

Usadas em classes anotadas com `@ControllerAdvice` ou `@RestControllerAdvice`
para comportamento global (toda a aplicação ou um subconjunto de controllers).

| Anotação | Contexto | Alvo | Descrição |
|----------|----------|------|-----------|
| `@ControllerAdvice` | SSR | Classe | Intercepta controllers SSR globalmente. Pode conter `@ExceptionHandler`, `@ModelAttribute` e `@InitBinder`. |
| `@RestControllerAdvice` | REST | Classe | Atalho para `@ControllerAdvice` + `@ResponseBody`. Respostas de erro são serializadas para JSON. |
| `@ExceptionHandler` | Geral | Método | Dentro de `@ControllerAdvice`, captura exceções de todos os controllers. |
| `@ModelAttribute` | SSR | Método | Dentro de `@ControllerAdvice`, adiciona atributos ao `Model` globalmente (ex.: usuário logado, configurações). |
| `@InitBinder` | Geral | Método | Dentro de `@ControllerAdvice`, inicializa `WebDataBinder` para todos os controllers. |

```java
// Tratamento de erros global — REST
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ProblemDetail handleNotFound(RecursoNaoEncontradoException ex, Locale locale) { ... }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) { ... }
}

// Dados globais para todas as views SSR
@ControllerAdvice
public class GlobalModelAdvice {

    @ModelAttribute("usuarioLogado")
    public UsuarioDTO popularUsuario(@AuthenticationPrincipal UserDetails user) {
        return usuarioService.toDTO(user);
    }
}
```

---

### 3.7 Visão Consolidada — Contexto por Anotação

```
GERAL (REST + SSR)
├── Classe:    @RequestMapping  @Validated  @CrossOrigin
├── Método:    @GetMapping  @PostMapping  @PutMapping  @PatchMapping  @DeleteMapping
│              @ResponseStatus  @ExceptionHandler  @InitBinder
└── Parâmetro: @PathVariable  @RequestParam  @RequestHeader  @CookieValue
               @Valid  @Validated  @RequestAttribute

REST
├── Classe:    @RestController
├── Método:    @ResponseBody
└── Parâmetro: @RequestBody  @RequestPart

SSR (Thymeleaf / Templates)
├── Classe:    @Controller  @SessionAttributes
├── Método:    @ModelAttribute (pré-popula Model)
└── Parâmetro: @ModelAttribute (binding de form)  @SessionAttribute
```

---

### 3.8 Tipos de Retorno dos Métodos de Controller

Esta seção apresenta os principais tipos que um método de controller pode retornar,
com indicação do contexto de uso, quando preferir cada um e exemplos práticos.

| Tipo de Retorno | Contexto | Descrição |
|-----------------|----------|-----------|
| `String` | SSR | Nome da view a renderizar (`"produtos/lista"`) ou redirect (`"redirect:/produtos"`). O tipo mais simples para SSR. |
| `ModelAndView` | SSR | Encapsula nome da view **e** os atributos do model em um único objeto. Útil quando ambos são construídos condicionalmente. |
| `ResponseEntity<T>` | REST | Controle total sobre status HTTP, headers e corpo da resposta. O tipo mais completo para REST. |
| `T` (objeto direto) | REST | Em `@RestController`, qualquer objeto retornado é serializado para JSON/XML automaticamente (implica `@ResponseBody`). |
| `void` | Geral | O método escreve a resposta diretamente no `HttpServletResponse`, ou para SSR com `@ResponseStatus`. |
| `View` | SSR | Implementação de `View` retornada diretamente — permite controle total da renderização sem passar pelo `ViewResolver`. |
| `RedirectView` | SSR | Especialização de `View` para redirects HTTP. Permite definir status code, propagação de parâmetros e encoding. |
| `HttpEntity<T>` | REST | Versão simplificada de `ResponseEntity` — headers + corpo, sem controle de status. |
| `ResponseBodyEmitter` | REST | Streaming de múltiplos objetos para a resposta. Base de `SseEmitter`. |
| `StreamingResponseBody` | REST | Escrita assíncrona e incremental no `OutputStream` da resposta (ex.: download de arquivos grandes). |
| `Callable<T>` / `CompletableFuture<T>` | REST | Processamento assíncrono — libera a thread do Servlet durante a execução. |

---

#### `String` — retorno SSR mais simples

```java
@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    // Retorna nome de view — Spring resolve para templates/produtos/lista.html
    @GetMapping
    public String listar(Model model) {
        model.addAttribute("produtos", produtoService.listar());
        return "produtos/lista";                   // view name
    }

    // Redirect após POST (padrão PRG — Post/Redirect/Get)
    @PostMapping
    public String salvar(@ModelAttribute @Valid ProdutoForm form,
                         BindingResult binding,
                         RedirectAttributes ra) {
        if (binding.hasErrors()) return "produtos/formulario"; // re-exibe form
        produtoService.criar(form);
        ra.addFlashAttribute("mensagem", "produto.criado");
        return "redirect:/produtos";               // redirect HTTP 302
    }

    // Forward interno (não cria novo request)
    @GetMapping("/legado/{id}")
    public String forward(@PathVariable Long id) {
        return "forward:/produtos/" + id;          // forward para outra rota
    }
}
```

---

#### `ModelAndView` — view e model juntos

```java
// Preferir quando view e atributos são construídos em lógica condicional
@GetMapping("/{id}/editar")
public ModelAndView editar(@PathVariable Long id) {
    var mav = new ModelAndView("produtos/formulario"); // view name

    produtoService.buscarPorId(id).ifPresentOrElse(
        p -> {
            mav.addObject("form", ProdutoForm.de(p));
            mav.addObject("categorias", categoriaService.listar());
        },
        () -> {
            mav.setViewName("redirect:/produtos");
            mav.addObject("erro", "produto.nao.encontrado");
        }
    );

    return mav;
}

// Com status HTTP explícito
@GetMapping("/erro")
public ModelAndView paginaErro() {
    var mav = new ModelAndView("erros/generico");
    mav.setStatus(HttpStatus.INTERNAL_SERVER_ERROR);
    mav.addObject("mensagem", "Erro inesperado.");
    return mav;
}
```

> **Prefira `String` + `Model`** para o caso simples — `ModelAndView` agrega valor
> quando o nome da view ou os atributos dependem de lógica condicional.

---

#### `ResponseEntity<T>` — controle total da resposta REST

```java
@RestController
@RequestMapping("/api/v1/produtos")
public class ProdutoController {

    // GET — 200 OK com body / 404 Not Found sem body
    @GetMapping("/{id}")
    public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) {
        return produtoService.buscarPorId(id)
                .map(ResponseEntity::ok)                         // 200 + body
                .orElse(ResponseEntity.notFound().build());      // 404 sem body
    }

    // POST — 201 Created com Location header e body
    @PostMapping
    public ResponseEntity<ProdutoResponse> criar(@RequestBody @Valid ProdutoRequest req) {
        ProdutoResponse criado = produtoService.criar(req);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(criado.id())
                .toUri();
        return ResponseEntity
                .created(location)                               // 201 + Location
                .body(criado);
    }

    // PUT — 200 OK ou 404 Not Found
    @PutMapping("/{id}")
    public ResponseEntity<ProdutoResponse> atualizar(
            @PathVariable Long id,
            @RequestBody @Valid ProdutoRequest req) {
        return produtoService.atualizar(id, req)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE — 204 No Content
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> excluir(@PathVariable Long id) {
        produtoService.excluir(id);
        return ResponseEntity.noContent().build();               // 204
    }

    // Headers customizados
    @GetMapping("/{id}/export")
    public ResponseEntity<byte[]> exportar(@PathVariable Long id) {
        byte[] csv = produtoService.exportarCsv(id);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION,
                        "attachment; filename=\"produto-" + id + ".csv\"")
                .contentType(MediaType.parseMediaType("text/csv"))
                .body(csv);
    }
}
```

> **Prefira `T` direto** quando o status for sempre 200 OK e não houver headers extras —
> `ResponseEntity` agrega valor quando status, headers ou ausência de body variam.

---

#### `View` e `RedirectView` — controle programático de views SSR

```java
@Controller
public class ExportController {

    // View customizada — renderização própria, sem ViewResolver
    @GetMapping("/relatorio/pdf")
    public View gerarPdf() {
        return new AbstractView() {
            @Override
            protected void renderMergedOutputModel(
                    Map<String, Object> model,
                    HttpServletRequest req,
                    HttpServletResponse res) throws Exception {
                res.setContentType("application/pdf");
                // escreve bytes do PDF diretamente no response
            }
        };
    }
}

@Controller
public class ProdutoMvcController {

    // RedirectView — mais controle que "redirect:/url"
    @PostMapping
    public RedirectView salvar(@ModelAttribute @Valid ProdutoForm form,
                               BindingResult binding) {
        if (binding.hasErrors()) {
            // ⚠️ RedirectView não suporta re-exibir form com erros diretamente
            // Retorne String nesse caso: return "produtos/formulario"
        }
        Long novoId = produtoService.criar(form).getId();

        var rv = new RedirectView("/produtos/" + novoId);
        rv.setStatusCode(HttpStatus.SEE_OTHER);          // 303 em vez de 302
        rv.setExposeModelAttributes(false);              // não passa model como query params
        rv.setContextRelative(true);                     // URL relativa ao context-path
        return rv;
    }

    // Redirect permanente 301 (SEO — mudança de URL definitiva)
    @GetMapping("/produto/{id}")          // URL antiga
    public RedirectView redirectLegado(@PathVariable Long id) {
        var rv = new RedirectView("/produtos/" + id);
        rv.setStatusCode(HttpStatus.MOVED_PERMANENTLY);  // 301
        return rv;
    }
}
```

| | `"redirect:/url"` (String) | `RedirectView` |
|-|---------------------------|----------------|
| Status code | Sempre 302 | Configurável (301, 302, 303, 307…) |
| `exposeModelAttributes` | Configurável globalmente | Configurável por redirect |
| Simplicidade | Alta | Média |
| Quando usar | Casos comuns | Status != 302 ou controle fino |

---

#### Visão consolidada — tipos de retorno por contexto

```
REST (@RestController / @ResponseBody)
├── T                    — objeto serializado, sempre 200 OK
├── ResponseEntity<T>    — status + headers + body configuráveis  ← preferido para REST
├── HttpEntity<T>        — headers + body (sem controle de status)
├── void                 — escreve direto no HttpServletResponse
├── StreamingResponseBody— streaming de bytes (downloads)
└── Callable<T> /
    CompletableFuture<T> — processamento assíncrono

SSR (@Controller sem @ResponseBody)
├── String               — nome de view, "redirect:/url" ou "forward:/url"  ← preferido para SSR
├── ModelAndView         — view + atributos juntos (lógica condicional)
├── View                 — implementação customizada de renderização
├── RedirectView         — redirect com controle de status (301, 303…)
└── void                 — escreve direto no HttpServletResponse
```


---



## 4. Controllers MVC com Thymeleaf (SSR)

> **Sintaxe alternativa HTML5 — `data-th-*`**
>
> Todos os atributos `th:*` do Thymeleaf possuem uma forma equivalente no padrão
> `data-th-*` (ex.: `data-th-text`, `data-th-href`, `data-th-field`), que é
> tecnicamente válida segundo a especificação HTML5 — qualquer atributo prefixado
> com `data-` é permitido pelo padrão e ignorado pelo browser.
>
> ```html
> <!-- th:* — sintaxe canônica do Thymeleaf (mais comum) -->
> <span th:text="${produto.nome}">Nome</span>
>
> <!-- data-th-* — equivalente 100%, aderente ao HTML5 -->
> <span data-th-text="${produto.nome}">Nome</span>
> ```
>
> A diferença é apenas sintática: o comportamento em tempo de execução é idêntico.
> A forma `th:*` é a mais usada na documentação e na comunidade; `data-th-*` é
> preferida quando validadores HTML5 estritos (linters, ferramentas de QA) são
> exigidos no projeto, pois `th:text` tecnicamente não é um atributo HTML válido
> fora do namespace Thymeleaf.

### 4.1 Controller MVC Clássico

```java
@Controller
@RequestMapping("/produtos")
@Slf4j
public class ProdutoMvcController {

    private final ProdutoService produtoService;
    private final CategoriaService categoriaService;

    public ProdutoMvcController(ProdutoService produtoService,
                                 CategoriaService categoriaService) {
        this.produtoService = produtoService;
        this.categoriaService = categoriaService;
    }

    // ─── GET /produtos ────────────────────────────────────────────────────────
    @GetMapping
    public String listar(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String busca,
            Model model) {

        var pageable = PageRequest.of(page, size, Sort.by("nome"));
        model.addAttribute("produtos", produtoService.listar(busca, pageable));
        model.addAttribute("busca", busca);
        return "produtos/lista";   // → templates/produtos/lista.html
    }

    // ─── GET /produtos/novo ───────────────────────────────────────────────────
    @GetMapping("/novo")
    public String exibirFormulario(Model model) {
        model.addAttribute("produto", new ProdutoForm());
        model.addAttribute("categorias", categoriaService.listarTodas());
        return "produtos/formulario";
    }

    // ─── POST /produtos ───────────────────────────────────────────────────────
    @PostMapping
    public String salvar(
            @ModelAttribute("produto") @Valid ProdutoForm form,
            BindingResult bindingResult,
            Model model,
            RedirectAttributes redirectAttrs) {

        // Sempre verificar BindingResult ANTES de usar o form
        if (bindingResult.hasErrors()) {
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";   // Volta ao form com erros
        }

        try {
            produtoService.criar(form);
            redirectAttrs.addFlashAttribute("mensagem",
                "Produto criado com sucesso!");
            redirectAttrs.addFlashAttribute("tipoMensagem", "success");
            return "redirect:/produtos";

        } catch (BusinessException e) {
            bindingResult.rejectValue("sku", "produto.sku.duplicado",
                "SKU já cadastrado no sistema");
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";
        }
    }

    // ─── GET /produtos/{id}/editar ────────────────────────────────────────────
    @GetMapping("/{id}/editar")
    public String exibirEdicao(@PathVariable Long id, Model model) {
        var produto = produtoService.buscarPorId(id)
                .orElseThrow(() -> new ResourceNotFoundException("Produto", id));

        model.addAttribute("produto", ProdutoForm.from(produto));
        model.addAttribute("categorias", categoriaService.listarTodas());
        model.addAttribute("editando", true);
        return "produtos/formulario";
    }

    // ─── POST /produtos/{id} (PUT simulado via _method) ───────────────────────
    @PostMapping("/{id}")
    public String atualizar(
            @PathVariable Long id,
            @ModelAttribute("produto") @Valid ProdutoForm form,
            BindingResult bindingResult,
            Model model,
            RedirectAttributes redirectAttrs) {

        if (bindingResult.hasErrors()) {
            model.addAttribute("categorias", categoriaService.listarTodas());
            model.addAttribute("editando", true);
            return "produtos/formulario";
        }

        produtoService.atualizar(id, form);
        redirectAttrs.addFlashAttribute("mensagem", "Produto atualizado!");
        redirectAttrs.addFlashAttribute("tipoMensagem", "success");
        return "redirect:/produtos";
    }

    // ─── POST /produtos/{id}/excluir ──────────────────────────────────────────
    @PostMapping("/{id}/excluir")
    public String excluir(@PathVariable Long id, RedirectAttributes redirectAttrs) {
        produtoService.excluir(id);
        redirectAttrs.addFlashAttribute("mensagem", "Produto excluído!");
        redirectAttrs.addFlashAttribute("tipoMensagem", "warning");
        return "redirect:/produtos";
    }
}
```

### 4.2 Form Object (Separado do Domain/DTO)

```java
// Form object: representa o estado do formulário HTML, com coerção de tipos
public class ProdutoForm {

    @NotBlank(message = "Nome é obrigatório")
    @Size(max = 200)
    private String nome;

    @NotBlank
    @Size(max = 2000)
    private String descricao;

    // String no form para aceitar formatação do usuário (ex: "1.299,90")
    // O Formatter/Converter fará a coerção para BigDecimal
    @NotBlank
    private String preco;

    @NotNull
    @Min(0)
    private Integer estoque;

    @NotNull
    private Long categoriaId;

    // Factory method para preencher a partir da entidade (para edição)
    public static ProdutoForm from(Produto produto) {
        var form = new ProdutoForm();
        form.nome = produto.getNome();
        form.descricao = produto.getDescricao();
        form.preco = produto.getPreco().toPlainString();
        form.estoque = produto.getEstoque();
        form.categoriaId = produto.getCategoria().getId();
        return form;
    }

    // Getters e Setters (necessários para binding MVC)
    // ...
}
```

### 4.3 Templates Thymeleaf

#### Layout Base (`templates/layout/base.html`)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security"
      lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title layout:title-pattern="$CONTENT_TITLE - $DECORATOR_TITLE">Minha App</title>
    <!-- CSRF token para formulários AJAX -->
    <meta name="_csrf" th:content="${_csrf.token}">
    <meta name="_csrf_header" th:content="${_csrf.headerName}">
    <!-- Bootstrap via Webjars (versão resolvida automaticamente) -->
    <link rel="stylesheet" th:href="@{/webjars/bootstrap/css/bootstrap.min.css}">
    <link rel="stylesheet" th:href="@{/static/css/app.css}">
</head>
<body>
<nav th:replace="~{layout/navbar :: navbar}"></nav>

<!-- Mensagens Flash -->
<div class="container mt-3" th:if="${mensagem}">
    <div class="alert"
         th:classappend="'alert-' + ${tipoMensagem ?: 'info'}"
         th:text="${mensagem}"
         role="alert">
    </div>
</div>

<!-- Conteúdo da página filha -->
<main class="container mt-4" layout:fragment="content">
</main>

<script th:src="@{/webjars/bootstrap/js/bootstrap.bundle.min.js}"></script>
<th:block layout:fragment="scripts"></th:block>
</body>
</html>
```

#### Formulário de Produto (`templates/produtos/formulario.html`)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/base}">
<head>
    <title th:text="${editando} ? 'Editar Produto' : 'Novo Produto'">Produto</title>
</head>
<body>
<div layout:fragment="content">
    <h1 class="mb-4" th:text="${editando} ? 'Editar Produto' : 'Novo Produto'"></h1>

    <!-- Erros globais do BindingResult -->
    <div th:if="${#fields.hasGlobalErrors()}" class="alert alert-danger">
        <ul class="mb-0">
            <li th:each="err : ${#fields.globalErrors()}" th:text="${err}"></li>
        </ul>
    </div>

    <!--
        th:action: URL dinâmica para criar ou atualizar
        th:object: vincula o form ao @ModelAttribute "produto"
    -->
    <form th:action="${editando} ? @{/produtos/{id}(id=${produto.id})} : @{/produtos}"
          th:object="${produto}"
          method="post"
          enctype="application/x-www-form-urlencoded"
          novalidate>

        <!-- Campo nome com exibição de erro inline -->
        <div class="mb-3">
            <label for="nome" class="form-label">Nome <span class="text-danger">*</span></label>
            <input type="text"
                   id="nome"
                   th:field="*{nome}"
                   th:errorclass="is-invalid"
                   class="form-control">
            <div class="invalid-feedback" th:errors="*{nome}"></div>
        </div>

        <!-- Preço com placeholder de formato -->
        <div class="mb-3">
            <label for="preco" class="form-label">Preço <span class="text-danger">*</span></label>
            <div class="input-group">
                <span class="input-group-text">R$</span>
                <input type="text"
                       id="preco"
                       th:field="*{preco}"
                       th:errorclass="is-invalid"
                       class="form-control"
                       placeholder="0,00">
                <div class="invalid-feedback" th:errors="*{preco}"></div>
            </div>
        </div>

        <!-- Select de categoria com opção vazia -->
        <div class="mb-3">
            <label for="categoriaId" class="form-label">Categoria <span class="text-danger">*</span></label>
            <select id="categoriaId"
                    th:field="*{categoriaId}"
                    th:errorclass="is-invalid"
                    class="form-select">
                <option value="">Selecione...</option>
                <option th:each="cat : ${categorias}"
                        th:value="${cat.id}"
                        th:text="${cat.nome}">
                </option>
            </select>
            <div class="invalid-feedback" th:errors="*{categoriaId}"></div>
        </div>

        <!-- Estoque -->
        <div class="mb-3">
            <label for="estoque" class="form-label">Estoque</label>
            <input type="number"
                   id="estoque"
                   th:field="*{estoque}"
                   th:errorclass="is-invalid"
                   class="form-control"
                   min="0">
            <div class="invalid-feedback" th:errors="*{estoque}"></div>
        </div>

        <!-- Descrição -->
        <div class="mb-3">
            <label for="descricao" class="form-label">Descrição</label>
            <textarea id="descricao"
                      th:field="*{descricao}"
                      th:errorclass="is-invalid"
                      class="form-control"
                      rows="4">
            </textarea>
            <div class="invalid-feedback" th:errors="*{descricao}"></div>
        </div>

        <div class="d-flex gap-2">
            <button type="submit" class="btn btn-primary">
                <i class="fas fa-save"></i>
                <span th:text="${editando} ? 'Atualizar' : 'Salvar'">Salvar</span>
            </button>
            <a th:href="@{/produtos}" class="btn btn-outline-secondary">Cancelar</a>
        </div>
    </form>
</div>
</html>
```

#### Habilitando PUT/DELETE em Formulários HTML

O HTML padrão suporta apenas `GET` e `POST`. Para simular `PUT`/`DELETE`:

```java
// Em WebMvcConfig
@Bean
public HiddenHttpMethodFilter hiddenHttpMethodFilter() {
    return new HiddenHttpMethodFilter();
}
```

```html
<!-- No template: simula PUT -->
<form method="post" th:action="@{/produtos/{id}(id=${produto.id})}">
    <input type="hidden" name="_method" value="PUT">
    <!-- campos... -->
</form>
```

### 4.4 Convertendo `ConstraintViolationException` do Service em `BindingResult`

Quando um `@Service` anotado com `@Validated` lança `ConstraintViolationException`,
essa exceção **não é capturada automaticamente** pelo mecanismo de `BindingResult`
do Spring MVC — ela existe no service via proxy AOP, fora do ciclo de binding do
controller. Se não tratada, propagaria como 500 ou seria capturada por um
`@ControllerAdvice` genérico, sem vincular os erros aos campos do formulário.

O objetivo desta seção é mostrar como **converter** as violações de constraint para
erros de `BindingResult`, fazendo com que os `th:errors` e `th:errorclass` do
Thymeleaf funcionem normalmente — sem nenhuma alteração nos templates.

#### Por que isso acontece

```mermaid
flowchart LR
    subgraph Controller["Controller"]
        C1["@Valid @ModelAttribute<br>↓<br>BindingResult populado<br>automaticamente pelo MVC"]
        C2["produtoService.criar(form)<br>↓<br>ConstraintViolationException<br>lançada pelo AOP Proxy<br>do @Validated"]
    end
    subgraph Service["Service (@Validated)"]
        S1["proxy AOP intercepta<br>chamada ao método"]
    end

    C1 -->|"erros resolvidos<br>automaticamente"| TH["Thymeleaf<br>th:errors funciona ✓"]
    C2 --> S1
    S1 -->|ConstraintViolationException| C2
    C2 -->|"sem conversão:<br>exceção propaga"| EX["500 / @ControllerAdvice<br>th:errors NÃO funciona ✗"]

    style EX fill:#FCEBEB,stroke:#A32D2D,color:#501313
    style TH fill:#EAF3DE,stroke:#3B6D11,color:#27500A
```

#### Utilitário de conversão

Centralizar a conversão em um componente reutilizável evita duplicar o código em
todos os controllers.

```java
/**
 * Converte as violações de constraint lançadas por @Validated em services
 * para erros reconhecidos pelo BindingResult — e, consequentemente, pelo Thymeleaf.
 */
@Component
public class ConstraintViolationConverter {

    /**
     * Transfere cada {@link ConstraintViolation} para o {@link BindingResult},
     * vinculando-o ao campo correto do form object.
     *
     * O propertyPath retornado pelo violation tem o formato completo do caminho
     * AOP: "criar.form.nome", "criar.form.preco" etc.
     * O método extrai apenas o último nó com Kind=PROPERTY, que corresponde
     * ao nome do campo no form: "nome", "preco" etc.
     */
    public void convert(ConstraintViolationException ex, BindingResult result) {
        for (ConstraintViolation<?> violation : ex.getConstraintViolations()) {
            String field   = extractFieldName(violation.getPropertyPath());
            String code    = violation.getConstraintDescriptor()
                                      .getAnnotation()
                                      .annotationType()
                                      .getSimpleName(); // "NotBlank", "Size", ...
            String message = violation.getMessage();

            if (field.isEmpty()) {
                // Violação de classe (cross-field constraint) — erro global
                result.reject(code, message);
            } else {
                result.rejectValue(field, code, message);
            }
        }
    }

    /**
     * Extrai o nome do campo a partir do PropertyPath completo.
     *
     * "criar.form.nome"          → "nome"
     * "atualizar.request.preco"  → "preco"
     * "criar.form"               → "" (violação de classe — sem campo)
     */
    private String extractFieldName(Path propertyPath) {
        Path.Node lastNode = null;
        for (Path.Node node : propertyPath) {
            lastNode = node;
        }
        if (lastNode == null) return "";

        // ElementKind.PROPERTY identifica um campo do objeto;
        // outros kinds (METHOD, PARAMETER) representam o contexto AOP
        return lastNode.getKind() == ElementKind.PROPERTY
                ? lastNode.getName()
                : "";
    }
}
```

#### Controller usando o conversor

```java
@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    private final ProdutoService               produtoService;
    private final CategoriaService             categoriaService;
    private final ConstraintViolationConverter cvConverter;

    // ─── POST /produtos ───────────────────────────────────────────────────────
    @PostMapping
    public String salvar(
            @ModelAttribute("produto") @Valid ProdutoForm form,
            BindingResult bindingResult,              // deve vir LOGO após @ModelAttribute
            Model model,
            RedirectAttributes redirectAttrs) {

        // 1. Erros de binding (@Valid no controller) — tratar antes do service
        if (bindingResult.hasErrors()) {
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";
        }

        try {
            produtoService.criar(form);
            redirectAttrs.addFlashAttribute("mensagem", "Produto criado com sucesso!");
            redirectAttrs.addFlashAttribute("tipoMensagem", "success");
            return "redirect:/produtos";

        } catch (ConstraintViolationException ex) {
            // 2. Violações do @Validated no service → converter para BindingResult
            cvConverter.convert(ex, bindingResult);
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";   // retorna ao form normalmente

        } catch (BusinessException ex) {
            // 3. Regras de negócio explícitas (ex: SKU duplicado)
            bindingResult.rejectValue("sku", "produto.sku.duplicado", ex.getMessage());
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";
        }
    }

    // ─── POST /produtos/{id} ─────────────────────────────────────────────────
    @PostMapping("/{id}")
    public String atualizar(
            @PathVariable Long id,
            @ModelAttribute("produto") @Valid ProdutoForm form,
            BindingResult bindingResult,
            Model model,
            RedirectAttributes redirectAttrs) {

        if (bindingResult.hasErrors()) {
            model.addAttribute("categorias", categoriaService.listarTodas());
            model.addAttribute("editando", true);
            return "produtos/formulario";
        }

        try {
            produtoService.atualizar(id, form);
            redirectAttrs.addFlashAttribute("mensagem", "Produto atualizado!");
            return "redirect:/produtos";

        } catch (ConstraintViolationException ex) {
            cvConverter.convert(ex, bindingResult);
            model.addAttribute("categorias", categoriaService.listarTodas());
            model.addAttribute("editando", true);
            return "produtos/formulario";
        }
    }
}
```

#### Service com `@Validated` — origem das violações

```java
@Service
@Validated   // ativa o proxy AOP que lança ConstraintViolationException
public class ProdutoService {

    /**
     * As constraints aqui representam invariantes de domínio que valem
     * independentemente de onde o service é chamado (controller, job, outro service).
     * O proxy AOP valida ANTES de entrar no corpo do método.
     */
    public ProdutoResponse criar(@Valid @NotNull ProdutoForm form) {
        // Se qualquer constraint for violada, o proxy lança
        // ConstraintViolationException antes de chegar aqui
        return ProdutoResponse.from(produtoRepository.save(toEntity(form)));
    }

    public ProdutoResponse atualizar(Long id, @Valid @NotNull ProdutoForm form) {
        var produto = produtoRepository.findById(id)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));
        update(produto, form);
        return ProdutoResponse.from(produtoRepository.save(produto));
    }
}
```

#### Template Thymeleaf — sem nenhuma alteração

O `BindingResult` preenchido pelo conversor é idêntico ao preenchido pelo `@Valid`
do MVC. O Thymeleaf usa o mesmo mecanismo em ambos os casos: `th:errors`,
`th:errorclass` e `#fields.hasErrors()` funcionam sem qualquer modificação:

```html
<!-- templates/produtos/formulario.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/base}">
<body>
<section layout:fragment="content">

<form th:action="${editando} ? @{/produtos/{id}(id=${produto.id})} : @{/produtos}"
      th:object="${produto}"
      method="post"
      novalidate>

    <!-- ─── Erros globais (violações cross-field ou BusinessException sem campo) -->
    <div th:if="${#fields.hasGlobalErrors()}" class="alert alert-danger">
        <ul class="mb-0">
            <li th:each="err : ${#fields.globalErrors()}" th:text="${err}"></li>
        </ul>
    </div>

    <!-- ─── Campo: nome ───────────────────────────────────────────────────────
         th:errorclass="is-invalid" é adicionado automaticamente quando
         BindingResult tem erros para o campo "nome" — de qualquer origem:
         @Valid no controller OU ConstraintViolationConverter do service. -->
    <div class="mb-3">
        <label for="nome" class="form-label">Nome *</label>
        <input type="text" id="nome"
               th:field="*{nome}"
               th:errorclass="is-invalid"
               class="form-control">
        <div class="invalid-feedback" th:errors="*{nome}"></div>
    </div>

    <!-- ─── Campo: preco ──────────────────────────────────────────────────── -->
    <div class="mb-3">
        <label for="preco" class="form-label">Preço *</label>
        <input type="text" id="preco"
               th:field="*{preco}"
               th:errorclass="is-invalid"
               class="form-control" placeholder="0,00">
        <div class="invalid-feedback" th:errors="*{preco}"></div>
    </div>

    <!-- ─── Campo: estoque ────────────────────────────────────────────────── -->
    <div class="mb-3">
        <label for="estoque" class="form-label">Estoque</label>
        <input type="number" id="estoque"
               th:field="*{estoque}"
               th:errorclass="is-invalid"
               class="form-control">
        <div class="invalid-feedback" th:errors="*{estoque}"></div>
    </div>

    <!-- ─── Campo: sku ────────────────────────────────────────────────────── -->
    <div class="mb-3">
        <label for="sku" class="form-label">SKU</label>
        <input type="text" id="sku"
               th:field="*{sku}"
               th:errorclass="is-invalid"
               class="form-control">
        <div class="invalid-feedback" th:errors="*{sku}"></div>
    </div>

    <!-- ─── Select: categoria ─────────────────────────────────────────────── -->
    <div class="mb-3">
        <label for="categoriaId" class="form-label">Categoria *</label>
        <select id="categoriaId"
                th:field="*{categoriaId}"
                th:errorclass="is-invalid"
                class="form-select">
            <option value="">Selecione...</option>
            <option th:each="cat : ${categorias}"
                    th:value="${cat.id()}"
                    th:text="${cat.nome()}">
            </option>
        </select>
        <div class="invalid-feedback" th:errors="*{categoriaId}"></div>
    </div>

    <div class="d-flex gap-2 mt-4">
        <button type="submit" class="btn btn-primary">Salvar</button>
        <a th:href="@{/produtos}" class="btn btn-outline-secondary">Cancelar</a>
    </div>
</form>

</section>
</body>
</html>
```

#### Alternativa: `@ControllerAdvice` para centralização

Para evitar o bloco `try/catch` repetido em vários controllers, a exceção pode ser
capturada em um `@ControllerAdvice`. A limitação é que o `BindingResult` **não é
acessível** fora do escopo do controller — os erros precisam ser transportados como
flash attribute e exibidos em um bloco genérico, sem vinculação campo a campo:

```java
@ControllerAdvice(annotations = Controller.class)
public class ConstraintViolationMvcAdvice {

    @ExceptionHandler(ConstraintViolationException.class)
    public String handleConstraintViolation(
            ConstraintViolationException ex,
            HttpServletRequest request,
            RedirectAttributes redirectAttrs) {

        var erros = ex.getConstraintViolations().stream()
                .map(v -> new FieldErrorInfo(
                        extractFieldName(v.getPropertyPath()),
                        v.getMessage()))
                .toList();

        redirectAttrs.addFlashAttribute("errosValidacao", erros);

        // Redireciona para a URL de origem (Referer) para re-exibir o formulário
        String referer = request.getHeader(HttpHeaders.REFERER);
        return "redirect:" + (referer != null ? referer : "/");
    }

    private String extractFieldName(Path path) {
        Path.Node last = null;
        for (Path.Node n : path) last = n;
        return (last != null && last.getKind() == ElementKind.PROPERTY)
                ? last.getName() : "";
    }

    public record FieldErrorInfo(String campo, String mensagem) {}
}
```

```html
<!-- Template — exibição via lista genérica (abordagem @ControllerAdvice) -->
<div th:if="${errosValidacao != null and !#lists.isEmpty(errosValidacao)}"
     class="alert alert-danger">
    <strong>Corrija os erros abaixo:</strong>
    <ul class="mb-0 mt-1">
        <li th:each="err : ${errosValidacao}"
            th:text="${err.campo() != '' ? err.campo() + ': ' : ''} + ${err.mensagem()}">
        </li>
    </ul>
</div>
```

#### Resumo: quando usar cada abordagem

| | `try/catch` no controller | `@ControllerAdvice` |
|---|---|---|
| `th:errors` vinculado ao campo | ✅ Sim | ❌ Não |
| `th:errorclass` automático | ✅ Sim | ❌ Não |
| Form mantém valores digitados | ✅ Sim (model intacto) | ⚠️ Perde no redirect |
| Centralização do tratamento | ❌ Repetido por controller | ✅ Um único lugar |
| **Recomendação** | ✅ Formulários com campos | Somente lista genérica de erros |

### 4.5 Atributos de Modelo Compartilhados com @ModelAttribute

```java
@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    // Executado ANTES de todos os métodos do controller.
    // Útil para dados comuns a múltiplas views (ex: listas de select).
    @ModelAttribute("categorias")
    public List<CategoriaResponse> categorias() {
        return categoriaService.listarTodas();
    }

    @ModelAttribute("usuario")
    public UsuarioInfo usuarioLogado(Authentication auth) {
        return (UsuarioInfo) auth.getPrincipal();
    }
}
```

---

### 4.6 `@SessionAttributes` e `@SessionAttribute`

#### `@SessionAttributes` — manter atributos do model na sessão

`@SessionAttributes` é uma anotação de **classe** que instrui o Spring MVC a
persistir determinados atributos do `Model` na `HttpSession` entre requisições do
mesmo controller. É a solução nativa do Spring MVC para fluxos de múltiplas etapas
(wizards) sem precisar manipular `HttpSession` diretamente.

```java
/**
 * Wizard de criação de pedido em três etapas.
 *
 * @SessionAttributes mantém "pedidoWizard" na sessão enquanto o fluxo
 * não for concluído ou cancelado — sem nenhum acesso direto à HttpSession.
 *
 * IMPORTANTE: funciona apenas para atributos criados pelo MESMO controller.
 * Para ler atributos de sessão criados externamente, use @SessionAttribute (singular).
 */
@Controller
@RequestMapping("/pedidos/novo")
@SessionAttributes("pedidoWizard")          // ← nome(s) do(s) atributo(s) a persistir
public class PedidoWizardController {

    private final ProdutoService   produtoService;
    private final EnderecoService  enderecoService;
    private final PedidoService    pedidoService;

    // ─── Inicializa o objeto de sessão (chamado apenas na PRIMEIRA request) ───
    //
    // @ModelAttribute de classe é executado antes de qualquer handler method.
    // O Spring só chama este método se "pedidoWizard" ainda não existir no Model
    // (nem na sessão) — evita sobrescrever o estado acumulado entre etapas.
    @ModelAttribute("pedidoWizard")
    public PedidoWizard inicializarWizard() {
        return new PedidoWizard();
    }

    // ─── Etapa 1: seleção de produtos ────────────────────────────────────────
    @GetMapping("/etapa-1")
    public String etapa1(Model model) {
        model.addAttribute("produtos", produtoService.listarAtivos());
        return "pedidos/wizard/etapa1";
    }

    @PostMapping("/etapa-1")
    public String processarEtapa1(
            @ModelAttribute("pedidoWizard") PedidoWizard wizard, // ← vem da sessão
            @Valid EtapaItensForm form,
            BindingResult binding,
            Model model) {

        if (binding.hasErrors()) {
            model.addAttribute("produtos", produtoService.listarAtivos());
            return "pedidos/wizard/etapa1";
        }

        wizard.setItens(form.getItens());   // acumula estado no objeto de sessão
        return "redirect:/pedidos/novo/etapa-2";
    }

    // ─── Etapa 2: endereço de entrega ────────────────────────────────────────
    @GetMapping("/etapa-2")
    public String etapa2(
            @ModelAttribute("pedidoWizard") PedidoWizard wizard,
            Model model) {

        model.addAttribute("enderecos", enderecoService.listarDoCliente());
        model.addAttribute("subtotal", wizard.calcularSubtotal());
        return "pedidos/wizard/etapa2";
    }

    @PostMapping("/etapa-2")
    public String processarEtapa2(
            @ModelAttribute("pedidoWizard") PedidoWizard wizard,
            @Valid EtapaEnderecoForm form,
            BindingResult binding,
            Model model) {

        if (binding.hasErrors()) {
            model.addAttribute("enderecos", enderecoService.listarDoCliente());
            return "pedidos/wizard/etapa2";
        }

        wizard.setEnderecoEntregaId(form.getEnderecoId());
        return "redirect:/pedidos/novo/etapa-3";
    }

    // ─── Etapa 3: resumo e confirmação ───────────────────────────────────────
    @GetMapping("/etapa-3")
    public String etapa3(
            @ModelAttribute("pedidoWizard") PedidoWizard wizard,
            Model model) {

        model.addAttribute("resumo", pedidoService.calcularResumo(wizard));
        return "pedidos/wizard/etapa3";
    }

    @PostMapping("/confirmar")
    public String confirmar(
            @ModelAttribute("pedidoWizard") PedidoWizard wizard,
            SessionStatus sessionStatus,         // ← injeta o status da sessão
            RedirectAttributes redirectAttrs) {

        var pedido = pedidoService.criar(wizard);

        // OBRIGATÓRIO ao final do fluxo: limpa os atributos de @SessionAttributes
        // da sessão. Sem isso, o wizard persiste indefinidamente e interferirá
        // na próxima tentativa de criação de pedido.
        sessionStatus.setComplete();

        redirectAttrs.addFlashAttribute("mensagem",
                "Pedido #" + pedido.getNumero() + " criado com sucesso!");
        return "redirect:/pedidos/" + pedido.getId();
    }

    @GetMapping("/cancelar")
    public String cancelar(SessionStatus sessionStatus, RedirectAttributes redirectAttrs) {
        sessionStatus.setComplete();    // limpa a sessão ao cancelar também
        redirectAttrs.addFlashAttribute("mensagem", "Criação de pedido cancelada.");
        return "redirect:/pedidos";
    }
}
```

**Form object acumulador do wizard:**

```java
/**
 * Objeto de sessão que acumula o estado entre as etapas do wizard.
 * Deve ser serializável se a sessão for distribuída (Redis, Hazelcast).
 */
public class PedidoWizard implements Serializable {

    private List<ItemWizard> itens      = new ArrayList<>();
    private Long   enderecoEntregaId;
    private String observacao;

    public BigDecimal calcularSubtotal() {
        return itens.stream()
                .map(i -> i.precoUnitario().multiply(BigDecimal.valueOf(i.quantidade())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    // getters / setters
}
```

**Template da etapa 1 — acesso ao wizard acumulado:**

```html
<!-- templates/pedidos/wizard/etapa1.html -->
<form th:action="@{/pedidos/novo/etapa-1}" method="post">

    <!-- Progresso do wizard -->
    <div class="d-flex gap-2 mb-4">
        <span class="badge bg-primary">1. Produtos</span>
        <span class="badge bg-secondary">2. Endereço</span>
        <span class="badge bg-secondary">3. Confirmação</span>
    </div>

    <!-- Lista de produtos para seleção -->
    <div th:each="produto : ${produtos}" class="form-check mb-2">
        <input class="form-check-input" type="checkbox"
               th:name="'itens[' + ${produtoStat.index} + '].produtoId'"
               th:value="${produto.id()}"
               th:id="'prod-' + ${produto.id()}">
        <label class="form-check-label" th:for="'prod-' + ${produto.id()}">
            <span th:text="${produto.nome()}">Produto</span>
            — R$ <span th:text="${produto.preco()}">0,00</span>
        </label>
    </div>

    <!-- Botões de navegação do wizard -->
    <div class="d-flex justify-content-between mt-4">
        <a th:href="@{/pedidos/novo/cancelar}" class="btn btn-outline-secondary">
            Cancelar
        </a>
        <button type="submit" class="btn btn-primary">
            Próximo →
        </button>
    </div>
</form>
```

---

#### `@SessionAttribute` (singular) — ler atributo de sessão externo

`@SessionAttribute` (singular, sem `s`) é um **parâmetro de método** que lê um
atributo já existente na `HttpSession` — tipicamente criado por outro controller,
filtro ou interceptor. Não gerencia ciclo de vida; apenas lê.

```java
@Controller
@RequestMapping("/checkout")
public class CheckoutController {

    /**
     * Lê o carrinho de compras da sessão, criado pelo CarrinhoController.
     *
     * required = true  (default): lança exceção se o atributo não existir.
     * required = false           : injeta null se ausente — use Optional ou null-check.
     */
    @GetMapping
    public String exibir(
            @SessionAttribute("carrinho") CarrinhoSession carrinho,
            Model model) {

        model.addAttribute("itens",    carrinho.getItens());
        model.addAttribute("subtotal", carrinho.calcularTotal());
        return "checkout/resumo";
    }

    @GetMapping("/pagamento")
    public String pagamento(
            @SessionAttribute(value = "carrinho", required = false) CarrinhoSession carrinho,
            Model model) {

        if (carrinho == null || carrinho.estaVazio()) {
            return "redirect:/carrinho";
        }
        model.addAttribute("total", carrinho.calcularTotal());
        return "checkout/pagamento";
    }
}
```

#### Comparativo: `@SessionAttributes` vs `@SessionAttribute` vs `@SessionScope`

| | `@SessionAttributes` | `@SessionAttribute` | `@SessionScope` |
|---|---|---|---|
| Nível | Classe do controller | Parâmetro de método | Bean Spring |
| Cria atributo na sessão | ✅ Via `@ModelAttribute` | ❌ Apenas lê | ✅ Auto-gerenciado |
| Escopo | Mesmo controller | Qualquer controller | Toda a aplicação |
| Finalização | `SessionStatus.setComplete()` | N/A | Fim da sessão HTTP |
| Uso típico | Wizards / multi-step forms | Ler dado de sessão externo | Carrinho, preferências |
| Serialização necessária | Se sessão distribuída | Se sessão distribuída | Se sessão distribuída |

---



## 5. Bean Validation — @Valid vs @Validated

### 5.1 Diferença Conceitual

```mermaid
graph TB
    subgraph "@Valid — javax/jakarta.validation"
        V1["Parte da especificação Bean Validation<br/>(Jakarta Validation 3.x)"]
        V2["Valida o objeto e seus filhos<br/>(Cascata com @Valid no campo)"]
        V3["NÃO suporta grupos de validação<br/>como parâmetro da anotação"]
    end

    subgraph "@Validated — Spring"
        VA1["Extensão Spring de @Valid"]
        VA2["Suporta grupos de validação<br/>@Validated(CadastroGroup.class)"]
        VA3["Habilita validação de parâmetros<br/>de método no @Service/@Controller"]
        VA4["Processado via AOP Proxy<br/>(MethodValidationPostProcessor)"]
    end

    V1 --> V2 --> V3
    VA1 --> VA2 --> VA3 --> VA4
```

### 5.2 Exemplo Prático de Grupos de Validação

```java
// ─── Definição de grupos ──────────────────────────────────────────────────────
//
// Por que extends Default?
//
// Quando @Validated(Cadastro.class) é ativado, APENAS as constraints do grupo
// Cadastro são avaliadas. Constraints sem grupo explícito pertencem ao grupo
// Default, mas NÃO são executadas automaticamente quando um grupo específico
// é informado — a menos que o grupo herde de Default.
//
// Ao fazer `interface Cadastro extends Default`, o Bean Validation inclui
// automaticamente todas as constraints do grupo Default na mesma passagem.
// Isso evita repetir `groups = {Cadastro.class, Default.class}` em cada campo.
//
// Referência: https://stackoverflow.com/a/35359965
//
public interface ValidationGroups {
    interface Cadastro  extends Default {}  // herda Default: valida campos sem grupo também
    interface Edicao    extends Default {}  // herda Default: idem
    interface PatchGroup extends Default {} // herda Default: idem
}

// ─── DTO com grupos ───────────────────────────────────────────────────────────
public class ClienteRequest {

    // Obrigatório apenas no cadastro — campos sem grupo rodam via herança de Default
    @NotBlank(groups = ValidationGroups.Cadastro.class,
              message = "CPF é obrigatório no cadastro")
    @CPF(groups = {ValidationGroups.Cadastro.class, ValidationGroups.Edicao.class})
    private String cpf;

    @NotBlank(groups = {ValidationGroups.Cadastro.class, ValidationGroups.Edicao.class})
    @Email  // ← sem grupo = Default; executado em Cadastro e Edicao via herança
    private String email;

    @NotBlank(groups = {ValidationGroups.Cadastro.class, ValidationGroups.Edicao.class})
    @Size(min = 2, max = 100)  // ← sem grupo = Default; executado em todos os grupos
    private String nome;

    // Sem grupo = Default; roda em Cadastro, Edicao e PatchGroup via herança
    @Size(max = 20)
    private String telefone;

    // Validação de elementos dentro do genérico (TYPE_USE — Jakarta BV 2.0+)
    // Cada tag da lista é validada individualmente: @NotBlank e @Size por elemento
    @NotEmpty(groups = ValidationGroups.Cadastro.class)
    private List<@NotBlank @Size(max = 50) String> tags;
}

// ─── Controller usando grupos ─────────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/clientes")
public class ClienteController {

    @PostMapping
    public ResponseEntity<ClienteResponse> criar(
            // @Validated com grupo: aplica apenas as regras de Cadastro
            @RequestBody @Validated(ValidationGroups.Cadastro.class) ClienteRequest request) {
        // ...
    }

    @PutMapping("/{id}")
    public ResponseEntity<ClienteResponse> atualizar(
            @PathVariable Long id,
            @RequestBody @Validated(ValidationGroups.Edicao.class) ClienteRequest request) {
        // ...
    }

    @PatchMapping("/{id}")
    public ResponseEntity<ClienteResponse> atualizarParcial(
            @PathVariable Long id,
            @RequestBody @Validated(ValidationGroups.PatchGroup.class) ClienteRequest request) {
        // ...
    }
}
```

### 5.3 Validação em Serviços com @Validated

```java
// Habilitar validação de método em Services
@Service
@Validated  // Fundamental: sem isso, as anotações nos parâmetros são ignoradas
public class ClienteService {

    // Valida o parâmetro de entrada e o retorno
    public @NotNull ClienteResponse criar(@Valid @NotNull ClienteRequest request) {
        // ...
    }

    // Valida apenas parâmetros escalares (sem objeto wrapper)
    public ClienteResponse buscarPorCpf(
            @NotBlank @CPF String cpf) {   // Funciona com @Validated na classe
        // ...
    }

    // Valida coleção de elementos
    public List<ClienteResponse> criarLote(
            @NotEmpty @Valid List<ClienteRequest> requests) {
        // ...
    }
}
```

### 5.4 Cascata de Validação com @Valid

```java
public class PedidoRequest {

    @NotNull
    @Valid           // ← Cascata: valida os campos internos de EnderecoRequest
    private EnderecoRequest enderecoEntrega;

    @NotEmpty
    @Valid           // ← Cascata em coleção: valida cada ItemRequest
    private List<ItemRequest> itens;
}

public class EnderecoRequest {
    @NotBlank private String cep;
    @NotBlank private String logradouro;
    @NotBlank private String numero;
    @Size(max = 8) private String complemento;
    @NotBlank private String cidade;
    @NotBlank @Size(min = 2, max = 2) private String uf;
}
```

### 5.5 Constraint Customizada

```java
// ─── Anotação ─────────────────────────────────────────────────────────────────
@Target({FIELD, PARAMETER, ANNOTATION_TYPE})
@Retention(RUNTIME)
@Constraint(validatedBy = CpfValidator.class)
@Documented
public @interface CPF {
    String message() default "{br.com.app.validation.cpf.invalido}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// ─── Implementação ────────────────────────────────────────────────────────────
public class CpfValidator implements ConstraintValidator<CPF, String> {

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null || value.isBlank()) return true; // @NotNull cuida de null

        var digits = value.replaceAll("\\D", "");
        if (digits.length() != 11 || digits.chars().distinct().count() == 1) {
            return false;
        }

        return verificarDigitos(digits);
    }

    private boolean verificarDigitos(String digits) {
        int sum = 0;
        for (int i = 0; i < 9; i++) sum += (digits.charAt(i) - '0') * (10 - i);
        int r1 = sum % 11 < 2 ? 0 : 11 - (sum % 11);
        if (r1 != (digits.charAt(9) - '0')) return false;

        sum = 0;
        for (int i = 0; i < 10; i++) sum += (digits.charAt(i) - '0') * (11 - i);
        int r2 = sum % 11 < 2 ? 0 : 11 - (sum % 11);
        return r2 == (digits.charAt(10) - '0');
    }
}
```

### 5.6 Constraint com Acesso a Banco (Spring Bean)

```java
@Target(FIELD)
@Retention(RUNTIME)
@Constraint(validatedBy = EmailUnicoValidator.class)
public @interface EmailUnico {
    String message() default "E-mail já cadastrado";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Spring injeta dependências normalmente no validator
@Component
public class EmailUnicoValidator implements ConstraintValidator<EmailUnico, String> {

    private final ClienteRepository repository;

    public EmailUnicoValidator(ClienteRepository repository) {
        this.repository = repository;
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext ctx) {
        if (email == null) return true;
        return !repository.existsByEmailIgnoreCase(email);
    }
}
```

---

### 5.7 Atributo `payload` nas Constraints

O atributo `payload` presente em toda anotação de constraint (`Class<? extends Payload>[]`)
é uma extensão point da especificação Jakarta Bean Validation: permite **anexar
metadados** a uma violação em tempo de definição da constraint, sem alterar a lógica
de validação. Esses metadados ficam disponíveis em `ConstraintViolation.unwrap()` ou
via `ConstraintDescriptor` e podem ser lidos por quem processa as violações.

#### Usos práticos do `payload`

```mermaid
flowchart LR
    PL["payload = Severity.Error.class
 payload = Severity.Warning.class
 payload = HttpStatus.Conflict.class
 payload = Sensitive.class"]

    PL -->|"lido no @ControllerAdvice"| A["Mapear HTTP status
conforme o payload"]
    PL -->|"lido no handler MVC"| B["Separar erros críticos
de avisos não-bloqueantes"]
    PL -->|"lido no audit log"| C["Omitir campo sensível
do log de auditoria"]
    PL -->|"lido no cliente"| D["Códigos de erro
machine-readable"]
```

#### 1. Severidade — separar erros bloqueantes de avisos

```java
// ─── Definição dos marcadores de severidade ───────────────────────────────────
//
// Os payloads são interfaces marcadoras que estendem Payload.
// A especificação define Error e Warning como exemplos; Warning é convenção,
// não bloqueia o fluxo por padrão — quem decide o que fazer com cada
// severidade é o código consumidor.
public interface Severity {
    // Erro bloqueante: impede a operação
    interface Error   extends Payload {}

    // Aviso: operação prossegue mas cliente deve ser alertado
    interface Warning extends Payload {}

    // Informacional: sugestão de melhoria, não obrigatória
    interface Info    extends Payload {}
}

// ─── Uso nas constraints ──────────────────────────────────────────────────────
public record ProdutoRequest(

    @NotBlank(
        message  = "Nome é obrigatório",
        payload  = { Severity.Error.class }    // bloqueia — campo obrigatório
    )
    String nome,

    @Size(
        min     = 10,
        message = "Descrição muito curta — recomendamos pelo menos 10 caracteres",
        payload = { Severity.Warning.class }   // não bloqueia — apenas aviso
    )
    String descricao,

    @DecimalMax(
        value   = "9999.99",
        message = "Preço acima do limite recomendado para esta categoria",
        payload = { Severity.Warning.class }   // aviso, não erro
    )
    @DecimalMin(
        value   = "0.01",
        message = "Preço deve ser positivo",
        payload = { Severity.Error.class }     // bloqueia
    )
    BigDecimal preco
) {}
```

```java
// ─── Leitura da severidade no @ControllerAdvice ───────────────────────────────
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ValidationErrorResponse> handleConstraintViolation(
            ConstraintViolationException ex) {

        var errors   = new ArrayList<FieldError>();
        var warnings = new ArrayList<FieldError>();

        for (ConstraintViolation<?> v : ex.getConstraintViolations()) {
            var payloads = v.getConstraintDescriptor().getPayload();
            var entry    = new FieldError(
                    extractField(v.getPropertyPath()), v.getMessage());

            // Separa por severidade via payload
            if (payloads.stream().anyMatch(Severity.Error.class::isAssignableFrom)) {
                errors.add(entry);
            } else if (payloads.stream().anyMatch(Severity.Warning.class::isAssignableFrom)) {
                warnings.add(entry);
            }
        }

        // Se há erros bloqueantes: 422; se só avisos: 200 com warnings no body
        HttpStatus status = errors.isEmpty()
                ? HttpStatus.OK
                : HttpStatus.UNPROCESSABLE_ENTITY;

        return ResponseEntity.status(status)
                .body(new ValidationErrorResponse(errors, warnings));
    }

    private String extractField(Path path) {
        Path.Node last = null;
        for (Path.Node n : path) last = n;
        return last != null ? last.getName() : "";
    }

    public record FieldError(String campo, String mensagem) {}

    public record ValidationErrorResponse(
            List<FieldError> errors,
            List<FieldError> warnings
    ) {}
}
```

#### 2. Código de erro machine-readable — contrato com o cliente

```java
// ─── Payloads como códigos de erro ────────────────────────────────────────────
public interface ErrorCode {
    interface DuplicateValue  extends Payload {}
    interface InvalidFormat   extends Payload {}
    interface OutOfRange      extends Payload {}
    interface RequiredField   extends Payload {}
    interface BusinessRule    extends Payload {}
}

// ─── Constraints com código de erro ──────────────────────────────────────────
public record ClienteRequest(

    @NotBlank(
        message = "CPF é obrigatório",
        payload = { Severity.Error.class, ErrorCode.RequiredField.class }
    )
    @CPF(
        message = "CPF inválido",
        payload = { Severity.Error.class, ErrorCode.InvalidFormat.class }
    )
    String cpf,

    @EmailUnico(
        message = "E-mail já cadastrado",
        payload = { Severity.Error.class, ErrorCode.DuplicateValue.class }
    )
    String email
) {}

// ─── Handler que expõe os códigos de erro no JSON da resposta ─────────────────
@ExceptionHandler(ConstraintViolationException.class)
@ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
public ProblemDetail handleWithErrorCodes(ConstraintViolationException ex) {
    var violations = ex.getConstraintViolations().stream()
            .map(v -> {
                // Lê TODOS os payloads da violação
                var payloadNames = v.getConstraintDescriptor().getPayload().stream()
                        .map(Class::getSimpleName)   // "Error", "RequiredField", ...
                        .toList();

                return Map.of(
                    "campo",   extractField(v.getPropertyPath()),
                    "message", v.getMessage(),
                    "codes",   payloadNames           // cliente pode usar para i18n
                );
            })
            .toList();

    var pd = ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY, "Dados inválidos");
    pd.setProperty("violations", violations);
    return pd;
}
```

#### 4. Payload de sensibilidade — omitir campos do log de auditoria

```java
// ─── Marcador de dado sensível ────────────────────────────────────────────────
public interface Sensitive extends Payload {}

// ─── Constraints marcando campos sensíveis ────────────────────────────────────
public record LoginRequest(
    @NotBlank String username,

    @NotBlank(
        message = "Senha é obrigatória",
        payload = { Sensitive.class }   // informa ao audit log: não logar este campo
    )
    @Size(min = 8, message = "Senha muito curta", payload = { Sensitive.class })
    String password,

    @NotBlank(
        message = "Token obrigatório",
        payload = { Sensitive.class }   // token também sensível
    )
    String mfaToken
) {}

// ─── Audit interceptor que respeita o payload de sensibilidade ────────────────
@Component
public class ValidationAuditLogger {

    private static final Logger log = LoggerFactory.getLogger(ValidationAuditLogger.class);

    public void logViolations(ConstraintViolationException ex, String operacao) {
        ex.getConstraintViolations().forEach(v -> {
            boolean isSensitive = v.getConstraintDescriptor()
                    .getPayload()
                    .stream()
                    .anyMatch(Sensitive.class::isAssignableFrom);

            if (isSensitive) {
                // Loga apenas o campo — nunca o valor
                log.warn("[{}] Violação em campo sensível: campo={}",
                        operacao,
                        extractField(v.getPropertyPath()));
            } else {
                log.warn("[{}] Violação: campo={}, valor={}, mensagem={}",
                        operacao,
                        extractField(v.getPropertyPath()),
                        v.getInvalidValue(),
                        v.getMessage());
            }
        });
    }
}
```

#### 5. `payload` em SSR — separar erros por severidade no Thymeleaf

```java
// ─── Extensão do ConstraintViolationConverter (seção 4.4) para leitura do payload ─
@Component
public class ConstraintViolationConverter {

    /**
     * Versão que separa erros bloqueantes de avisos no BindingResult.
     * Erros (Severity.Error) → rejectValue → bloqueiam o envio do form.
     * Avisos (Severity.Warning) → adicionados ao Model como lista separada
     * para exibição não-bloqueante no template.
     */
    public List<FieldWarning> convertWithSeverity(
            ConstraintViolationException ex,
            BindingResult result) {

        var warnings = new ArrayList<FieldWarning>();

        for (ConstraintViolation<?> v : ex.getConstraintViolations()) {
            var payloads = v.getConstraintDescriptor().getPayload();
            String field = extractFieldName(v.getPropertyPath());
            String code  = v.getConstraintDescriptor().getAnnotation()
                             .annotationType().getSimpleName();

            boolean isWarning = payloads.stream()
                    .anyMatch(Severity.Warning.class::isAssignableFrom);

            if (isWarning) {
                warnings.add(new FieldWarning(field, v.getMessage()));
            } else {
                // Severity.Error ou sem payload → rejeita o campo normalmente
                if (field.isEmpty()) result.reject(code, v.getMessage());
                else                 result.rejectValue(field, code, v.getMessage());
            }
        }
        return warnings; // controller adiciona ao Model para exibição no template
    }

    public record FieldWarning(String campo, String mensagem) {}
}
```

```java
// ─── Controller usando a separação de severidade ──────────────────────────────
@PostMapping
public String salvar(
        @ModelAttribute("produto") @Valid ProdutoForm form,
        BindingResult bindingResult,
        Model model,
        RedirectAttributes redirectAttrs) {

    if (bindingResult.hasErrors()) {
        model.addAttribute("categorias", categoriaService.listarTodas());
        return "produtos/formulario";
    }

    try {
        produtoService.criar(form);
        redirectAttrs.addFlashAttribute("mensagem", "Produto criado com sucesso!");
        return "redirect:/produtos";

    } catch (ConstraintViolationException ex) {
        var warnings = cvConverter.convertWithSeverity(ex, bindingResult);

        if (bindingResult.hasErrors()) {
            // Há erros bloqueantes — retorna ao form
            model.addAttribute("warnings",   warnings);
            model.addAttribute("categorias", categoriaService.listarTodas());
            return "produtos/formulario";
        } else {
            // Apenas avisos — persiste e notifica
            redirectAttrs.addFlashAttribute("warnings", warnings);
            redirectAttrs.addFlashAttribute("mensagem", "Produto criado com avisos.");
            return "redirect:/produtos";
        }
    }
}
```

```html
<!-- Template: exibição de avisos não-bloqueantes separados dos erros ──────── -->

<!-- Bloco de avisos (Severity.Warning) — exibido mesmo quando o form foi salvo -->
<div th:if="${warnings != null and !#lists.isEmpty(warnings)}"
     class="alert alert-warning">
    <strong>Atenção:</strong>
    <ul class="mb-0 mt-1">
        <li th:each="w : ${warnings}"
            th:text="${w.campo() != '' ? w.campo() + ': ' : ''} + ${w.mensagem()}">
        </li>
    </ul>
</div>

<!-- Erros bloqueantes: exibidos normalmente via th:errors (sem mudança) -->
<div class="mb-3">
    <label for="descricao" class="form-label">Descrição</label>
    <textarea id="descricao" th:field="*{descricao}"
              th:errorclass="is-invalid"
              class="form-control" rows="3"></textarea>
    <!-- "Descrição muito curta" virá como warning, não como th:errors -->
    <div class="invalid-feedback" th:errors="*{descricao}"></div>
</div>
```

#### Resumo dos casos de uso do `payload`

| Caso de uso | Payload | Quem consome |
|---|---|---|
| Severidade de erro | `Severity.Error` / `.Warning` / `.Info` | `@ControllerAdvice`, converter SSR |
| Código de erro para o cliente | `ErrorCode.DuplicateValue` etc. | `@ControllerAdvice` (JSON) |
| Campo sensível / PII | `Sensitive` | Audit logger, log interceptor |
| Mapeamento de HTTP status | `HttpStatus.Conflict.class` (payload custom) | `@ControllerAdvice` |
| Classificação para monitoramento | `Payload` de domínio próprio | Métricas, alertas |

---



## 6. InitBinder

`@InitBinder` é executado antes do binding de cada request no controller. Permite registrar editores, formatters e configurações de binding específicas por controller.

### 6.1 Usos Comuns

```java
@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    /**
     * @InitBinder sem parâmetro: aplica a TODOS os @ModelAttribute do controller
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {

        // ── 1. Trimming automático de Strings ────────────────────────────────
        binder.registerCustomEditor(String.class,
            new StringTrimmerEditor(true)); // true = converter string vazia para null

        // ── 2. Formato de datas brasileiro ────────────────────────────────────
        var dateEditor = new CustomDateEditor(
            new SimpleDateFormat("dd/MM/yyyy"), true); // true = aceita null
        binder.registerCustomEditor(Date.class, dateEditor);

        // ── 3. Formatação de moeda brasileira (campo preco) ───────────────────
        binder.registerCustomEditor(BigDecimal.class, "preco",
            new BigDecimalBrazilianEditor());

        // ── 4. Campos proibidos (segurança: evitar mass assignment) ───────────
        binder.setDisallowedFields("id", "criadoEm", "atualizadoEm", "versao");

        // ── 5. Campos permitidos (whitelist — mais seguro) ────────────────────
        // binder.setAllowedFields("nome", "descricao", "preco", "estoque", "categoriaId");
    }

    /**
     * @InitBinder com nome: aplica apenas ao @ModelAttribute "produto"
     */
    @InitBinder("produto")
    public void initBinderProduto(WebDataBinder binder) {
        // Validação máxima de tamanho do objeto antes do binding
        binder.setValidator(new ProdutoFormValidator());
        // Configuração de campos obrigatórios no nível do binder
        binder.setRequiredFields("nome", "preco");
    }
}
```

### 6.2 PropertyEditor Customizado

```java
/**
 * Converte String no formato brasileiro "1.299,90" para BigDecimal
 */
public class BigDecimalBrazilianEditor extends PropertyEditorSupport {

    private static final NumberFormat FORMAT =
        NumberFormat.getNumberInstance(new Locale("pt", "BR"));

    @Override
    public void setAsText(String text) {
        if (text == null || text.isBlank()) {
            setValue(null);
            return;
        }
        try {
            // Remove espaços e converte
            setValue(new BigDecimal(FORMAT.parse(text.trim()).toString()));
        } catch (ParseException e) {
            throw new IllegalArgumentException(
                "Valor monetário inválido: '" + text + "'");
        }
    }

    @Override
    public String getAsText() {
        var value = (BigDecimal) getValue();
        return value == null ? "" : FORMAT.format(value);
    }
}
```

### 6.3 Validator Programático com @InitBinder

```java
@Component
public class ProdutoFormValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return ProdutoForm.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        var form = (ProdutoForm) target;

        // Regra de negócio: preço de promoção deve ser menor que preço original
        if (form.getPrecoPromocional() != null &&
            form.getPreco() != null &&
            form.getPrecoPromocional().compareTo(form.getPreco()) >= 0) {

            errors.rejectValue("precoPromocional",
                "produto.precoPromocional.invalido",
                "Preço promocional deve ser menor que o preço original");
        }

        // Validação de estoque mínimo por categoria
        if ("PERECIVEL".equals(form.getTipoCategoria()) && form.getEstoque() > 1000) {
            errors.rejectValue("estoque",
                "produto.estoque.perecivel",
                "Produtos perecíveis não podem ter estoque acima de 1.000 unidades");
        }
    }
}
```

---



## 7. Converters e Formatters

### 7.1 Diferenças entre os Tipos

```mermaid
graph TB
    subgraph "Spring Type Conversion System"
        C["Converter&lt;S, T&gt;<br/>Conversão simples entre tipos<br/>Sem locale (genérico)"]
        CF["ConverterFactory&lt;S, R&gt;<br/>Fabrica Converters para<br/>uma família de tipos (ex: enums)"]
        GC["GenericConverter<br/>Múltiplos pares S→T<br/>Com suporte a TypeDescriptor"]
        F["Formatter&lt;T&gt;<br/>Formato texto ↔ objeto<br/>Com suporte a Locale"]
        PE["PropertyEditor<br/>Legado (java.beans)<br/>Para binding em @InitBinder"]
    end

    C --> |"Registrado em"| FR[FormatterRegistry]
    CF --> FR
    GC --> FR
    F --> FR
    PE --> |"Registrado em"| WDB[WebDataBinder]
```

| | `Converter<S,T>` | `Formatter<T>` | `PropertyEditor` |
|---|---|---|---|
| Locale | ❌ Não | ✅ Sim | ❌ Não |
| Threads | Thread-safe | Thread-safe | ⚠️ Não (stateful) |
| Uso ideal | Conversão de tipos | Apresentação de dados | Legado / @InitBinder |
| Registro | `FormatterRegistry` | `FormatterRegistry` | `WebDataBinder` |

### 7.2 Converter — String para Enum Genérico

```java
/**
 * Converte qualquer String para qualquer Enum.
 * Aceita o nome do enum case-insensitive.
 */
public class StringToEnumConverterFactory
        implements ConverterFactory<String, Enum<?>> {

    @Override
    @SuppressWarnings({"unchecked", "rawtypes"})
    public <T extends Enum<?>> Converter<String, T> getConverter(Class<T> targetType) {
        return source -> {
            if (source == null || source.isBlank()) return null;
            // Busca case-insensitive
            return (T) Arrays.stream(targetType.getEnumConstants())
                    .filter(e -> e.name().equalsIgnoreCase(source.trim()))
                    .findFirst()
                    .orElseThrow(() -> new IllegalArgumentException(
                        "Valor inválido '" + source + "' para " + targetType.getSimpleName()));
        };
    }
}
```

### 7.3 Converter — ID para Entidade JPA

```java
/**
 * Permite receber o ID de uma entidade em um formulário e
 * obter a entidade completa automaticamente no binding.
 *
 * Uso: <select th:field="*{categoriaId}">
 * O MVC converte automaticamente Long → Categoria
 */
@Component
public class IdToCategoriaConverter implements Converter<Long, Categoria> {

    private final CategoriaRepository repository;

    public IdToCategoriaConverter(CategoriaRepository repository) {
        this.repository = repository;
    }

    @Override
    public Categoria convert(@NonNull Long source) {
        return repository.findById(source)
                .orElseThrow(() -> new ResourceNotFoundException("Categoria", source));
    }
}
```

### 7.4 Formatter — Moeda Brasileira com Locale

```java
/**
 * Formatter para BigDecimal no formato monetário brasileiro.
 * Responde ao Locale pt-BR.
 */
@Component
public class BrazilianMoneyFormatter implements Formatter<BigDecimal> {

    @Override
    public BigDecimal parse(String text, Locale locale) {
        if (text == null || text.isBlank()) return null;
        try {
            var format = NumberFormat.getNumberInstance(
                locale != null ? locale : new Locale("pt", "BR"));
            ((DecimalFormat) format).setParseBigDecimal(true);
            return (BigDecimal) format.parse(text.trim().replace("R$", "").trim());
        } catch (ParseException e) {
            throw new IllegalArgumentException("Valor monetário inválido: " + text);
        }
    }

    @Override
    public String print(BigDecimal object, Locale locale) {
        if (object == null) return "";
        return NumberFormat.getCurrencyInstance(
            locale != null ? locale : new Locale("pt", "BR"))
            .format(object);
    }
}
```

### 7.5 Formatter para LocalDate Brasileiro

```java
@Component
public class BrazilianDateFormatter implements Formatter<LocalDate> {

    private static final DateTimeFormatter BR_FORMAT =
        DateTimeFormatter.ofPattern("dd/MM/yyyy");
    private static final DateTimeFormatter ISO_FORMAT =
        DateTimeFormatter.ISO_LOCAL_DATE;

    @Override
    public LocalDate parse(String text, Locale locale) {
        if (text == null || text.isBlank()) return null;
        // Aceita tanto dd/MM/yyyy quanto yyyy-MM-dd
        if (text.contains("/")) {
            return LocalDate.parse(text, BR_FORMAT);
        }
        return LocalDate.parse(text, ISO_FORMAT);
    }

    @Override
    public String print(LocalDate object, Locale locale) {
        if (object == null) return "";
        // Formato para exibição pt-BR, ISO para API
        Locale effectiveLocale = locale != null ? locale : Locale.getDefault();
        return effectiveLocale.getLanguage().equals("pt")
            ? object.format(BR_FORMAT)
            : object.format(ISO_FORMAT);
    }
}
```

### 7.6 Registro dos Converters/Formatters

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    private final IdToCategoriaConverter idToCategoriaConverter;
    private final BrazilianMoneyFormatter brazilianMoneyFormatter;
    private final BrazilianDateFormatter brazilianDateFormatter;

    public WebMvcConfig(IdToCategoriaConverter idToCategoriaConverter,
                        BrazilianMoneyFormatter brazilianMoneyFormatter,
                        BrazilianDateFormatter brazilianDateFormatter) {
        this.idToCategoriaConverter = idToCategoriaConverter;
        this.brazilianMoneyFormatter = brazilianMoneyFormatter;
        this.brazilianDateFormatter = brazilianDateFormatter;
    }

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(idToCategoriaConverter);
        registry.addFormatter(brazilianMoneyFormatter);
        registry.addFormatter(brazilianDateFormatter);
        registry.addConverterFactory(new StringToEnumConverterFactory());
    }
}
```

---



## 8. Tratamento de Erros

### 8.1 Hierarquia de Exceções

```mermaid
graph TB
    BE[BusinessException<br/>abstract] --> RNF[ResourceNotFoundException<br/>404]
    BE --> DC[DuplicateEntityException<br/>409]
    BE --> BRE[BusinessRuleException<br/>422]
    BE --> AU[AccessDeniedException<br/>403]

    RuntimeException --> BE
    RuntimeException --> VE[ValidationException<br/>400 - Bean Validation]
```

### 8.2 Tratamento de Erros em SSR (Páginas de Erro Thymeleaf)

```java
// Para controllers @Controller (SSR), use @ControllerAdvice sem "Rest"
@ControllerAdvice(annotations = Controller.class)
public class MvcExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleNotFound(ResourceNotFoundException ex, Model model) {
        model.addAttribute("mensagem", ex.getMessage());
        model.addAttribute("codigo", 404);
        return "erros/404";   // → templates/erros/404.html
    }

    @ExceptionHandler(AccessDeniedException.class)
    public String handleForbidden(AccessDeniedException ex, Model model) {
        model.addAttribute("mensagem", "Você não tem permissão para acessar este recurso.");
        return "erros/403";
    }
}
```

---


## 9. Recursos Avançados e Pouco Explorados

### 9.1 HandlerInterceptor — Auditoria e Métricas

```java
@Component
@Slf4j
public class AuditInterceptor implements HandlerInterceptor {

    private final AuditService auditService;

    public AuditInterceptor(AuditService auditService) {
        this.auditService = auditService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) throws Exception {

        if (!(handler instanceof HandlerMethod method)) return true;

        if (method.hasMethodAnnotation(RequiresAudit.class)) {
            request.setAttribute("AUDIT_START_TIME", System.nanoTime());
            request.setAttribute("AUDIT_USER", SecurityUtils.getCurrentUser());
        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler, Exception ex) throws Exception {

        var startTime = (Long) request.getAttribute("AUDIT_START_TIME");
        if (startTime == null) return;

        var duration = (System.nanoTime() - startTime) / 1_000_000;
        var user = (String) request.getAttribute("AUDIT_USER");

        auditService.registrar(AuditEvent.builder()
            .usuario(user)
            .endpoint(request.getMethod() + " " + request.getRequestURI())
            .statusCode(response.getStatus())
            .duracaoMs(duration)
            .ip(getClientIp(request))
            .build());
    }

    private String getClientIp(HttpServletRequest request) {
        var xff = request.getHeader("X-Forwarded-For");
        return xff != null ? xff.split(",")[0].trim() : request.getRemoteAddr();
    }
}
```

### 9.2 @ModelAttribute Global com @ControllerAdvice

```java
/**
 * Adiciona dados comuns a todos os models de todos os controllers SSR.
 * Útil para dados de navegação, usuário logado, configurações.
 */
@ControllerAdvice(annotations = Controller.class)
public class GlobalModelAttributeAdvice {

    private final AppConfigService configService;

    @ModelAttribute("appConfig")
    public AppConfig appConfig() {
        return configService.getConfig();
    }

    @ModelAttribute("usuarioLogado")
    public Optional<UsuarioInfo> usuarioLogado(Authentication authentication) {
        if (authentication == null || !authentication.isAuthenticated()) {
            return Optional.empty();
        }
        return Optional.ofNullable((UsuarioInfo) authentication.getPrincipal());
    }

    @ModelAttribute("anoAtual")
    public int anoAtual() {
        return LocalDate.now().getYear();
    }
}
```

### 9.3 HandlerMethodArgumentResolver — Argumento Customizado

```java
/**
 * Resolve automaticamente o usuário logado como parâmetro de método.
 * Uso: public ResponseEntity<?> meuMetodo(@CurrentUser UsuarioInfo usuario)
 *
 * Elimina o boilerplate de injetar Authentication em todos os métodos.
 */
@Component
public class CurrentUserArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUser.class)
            && parameter.getParameterType().equals(UsuarioInfo.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                   ModelAndViewContainer mavContainer,
                                   NativeWebRequest webRequest,
                                   WebDataBinderFactory binderFactory) {

        var auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) return null;
        return auth.getPrincipal() instanceof UsuarioInfo u ? u : null;
    }
}

// Registro em WebMvcConfigurer
@Override
public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(currentUserArgumentResolver);
}

// Uso limpo no Controller (sem Authentication como parâmetro)
@GetMapping("/meu-perfil")
public ResponseEntity<PerfilResponse> meuPerfil(@CurrentUser UsuarioInfo usuario) {
    return ResponseEntity.ok(perfilService.buscar(usuario.getId()));
}
```

### 9.4 @RequestScope e @SessionScope Beans

```java
// Bean scoped por request — contexto da requisição corrente
@Component
@RequestScope
public class RequestContext {
    private String correlationId = UUID.randomUUID().toString();
    private String usuarioId;
    private String tenantId;
    // getters/setters
}

// Bean scoped por sessão — carrinho de compras, preferências
@Component
@SessionScope
public class CarrinhoSession {
    private final Map<Long, Integer> itens = new LinkedHashMap<>();

    public void adicionar(Long produtoId, int quantidade) {
        itens.merge(produtoId, quantidade, Integer::sum);
    }

    public Map<Long, Integer> getItens() {
        return Collections.unmodifiableMap(itens);
    }
}

// Injeção normal — Spring usa proxy para resolver o scope correto
@Controller
public class CarrinhoController {

    private final CarrinhoSession carrinho;

    public CarrinhoController(CarrinhoSession carrinho) {
        this.carrinho = carrinho;
    }
}
```

### 9.5 Flash Attributes — Dados entre Redirects (PRG Pattern)

```java
/**
 * PRG Pattern (Post-Redirect-Get):
 * POST → processa → redirect → GET → exibe resultado.
 * Flash attributes sobrevivem ao redirect e são removidos após a próxima view.
 */
@PostMapping
public String processarFormulario(RedirectAttributes redirectAttrs) {
    var resultado = service.processar();

    // Sobrevive ao redirect, removido automaticamente após exibição
    redirectAttrs.addFlashAttribute("sucesso", "Operação realizada com sucesso!");
    redirectAttrs.addFlashAttribute("resultado", resultado);

    // Vai na query string: /sucesso?id=42
    redirectAttrs.addAttribute("id", resultado.getId());

    return "redirect:/sucesso";
}
```

### 9.6 Controller Assíncrono — `CompletableFuture`, `Callable` e `DeferredResult`

O Spring MVC suporta retornos assíncronos no controller sem bloquear a thread do
Servlet. O container libera a thread imediatamente e a resposta é enviada quando o
resultado ficar disponível — essencial para operações de I/O pesado, integrações
externas e processamento paralelo.

#### Comparativo dos mecanismos

```mermaid
flowchart TD
    subgraph Mecanismos["Mecanismos de resposta assíncrona"]
        direction TB
        VT["Virtual Threads
@Async + spring.threads.virtual.enabled=true
Mais simples — sem mudança no controller"]
        CF["CompletableFuture<ResponseEntity<T>>
Controle total do pipeline assíncrono
Composição e transformações encadeadas"]
        CA["Callable<ResponseEntity<T>>
Mais simples — delegado ao TaskExecutor
Sem composição encadeada"]
        DR["DeferredResult<ResponseEntity<T>>
Resultado preenchido por outro thread
Ideal para eventos externos (SSE, filas)"]
    end

    VT -->|"Recomendado para
nova stack Java 21+"| USE
    CF -->|"I/O paralelo
múltiplas dependências"| USE
    CA -->|"Tarefa única
sem composição"| USE
    DR -->|"Resultado externo
long-polling"| USE[Escolha conforme o caso de uso]
```

#### Virtual Threads — a forma mais simples (Java 21+ / Spring Boot 3.2+)

Com Virtual Threads habilitados, o controller continua **síncrono** no código mas
não bloqueia threads de plataforma. É a abordagem recomendada para a maioria dos casos:

```java
// application.yml
// spring:
//   threads:
//     virtual:
//       enabled: true    ← habilita Virtual Threads no Tomcat automaticamente

// O controller continua idêntico — sem nenhuma mudança
@GetMapping("/{id}")
public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) {
    // Mesmo que esta chamada bloqueie 200ms, a thread virtual é suspensa
    // e a thread de plataforma fica livre para outras requisições
    return ResponseEntity.ok(produtoService.buscar(id));
}
```

#### `CompletableFuture` — composição assíncrona

```java
@RestController
@RequestMapping("/api/v1/produtos")
public class ProdutoAsyncController {

    private final ProdutoService produtoService;
    private final EstoqueService estoqueService;
    private final PrecoService precoService;

    // ─── Retorno simples com CompletableFuture ────────────────────────────────
    //
    // Spring detecta o CompletableFuture e libera a thread do Servlet.
    // Quando o future completar, a resposta é escrita na conexão original.
    @GetMapping("/{id}")
    @Operation(summary = "Buscar produto (assíncrono)")
    public CompletableFuture<ResponseEntity<ProdutoResponse>> buscar(
            @PathVariable Long id) {

        return produtoService.buscarAsync(id)
                .thenApply(ResponseEntity::ok)
                .exceptionally(ex -> {
                    if (ex.getCause() instanceof RecursoNaoEncontradoException) {
                        return ResponseEntity.notFound().build();
                    }
                    return ResponseEntity.internalServerError().build();
                });
    }

    // ─── Múltiplas chamadas paralelas combinadas ───────────────────────────────
    //
    // As três chamadas executam em paralelo; a resposta aguarda todas.
    // Sem CompletableFuture, seriam sequenciais: 300ms + 150ms + 200ms = 650ms.
    // Com paralelismo: max(300, 150, 200) = 300ms.
    @GetMapping("/{id}/detalhes-completos")
    @Operation(summary = "Detalhes completos — busca paralela")
    public CompletableFuture<ResponseEntity<ProdutoDetalhesResponse>> detalhes(
            @PathVariable Long id) {

        var produtoFuture  = produtoService.buscarAsync(id);           // ~300ms
        var estoqueFuture  = estoqueService.buscarPorProdutoAsync(id); // ~150ms
        var precoFuture    = precoService.buscarHistoricoAsync(id);    // ~200ms

        return CompletableFuture.allOf(produtoFuture, estoqueFuture, precoFuture)
                .thenApply(_ -> {
                    var resposta = new ProdutoDetalhesResponse(
                            produtoFuture.join(),
                            estoqueFuture.join(),
                            precoFuture.join()
                    );
                    return ResponseEntity.ok(resposta);
                });
    }

    // ─── CompletableFuture com timeout explícito ──────────────────────────────
    @GetMapping("/{id}/preco-externo")
    public CompletableFuture<ResponseEntity<PrecoExternoResponse>> precoExterno(
            @PathVariable Long id) {

        return precoService.consultarFornecedorAsync(id)
                .orTimeout(5, TimeUnit.SECONDS)           // ← timeout de 5s
                .thenApply(ResponseEntity::ok)
                .exceptionally(ex -> {
                    if (ex.getCause() instanceof TimeoutException) {
                        return ResponseEntity.status(HttpStatus.GATEWAY_TIMEOUT)
                                .build();
                    }
                    return ResponseEntity.status(HttpStatus.BAD_GATEWAY).build();
                });
    }
}
```

#### Serviço assíncrono com `@Async`

```java
@Service
public class ProdutoService {

    private final ProdutoRepository produtoRepository;

    // ─── @Async transforma o retorno em CompletableFuture automaticamente ──────
    //
    // Spring executa o método num thread separado do pool definido em @Async.
    // O chamador (controller) recebe um CompletableFuture já em andamento.
    @Async("asyncExecutor")
    public CompletableFuture<ProdutoResponse> buscarAsync(Long id) {
        var produto = produtoRepository.findById(id)
                .map(ProdutoResponse::from)
                .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));
        return CompletableFuture.completedFuture(produto);
    }

    @Async("asyncExecutor")
    public CompletableFuture<List<ProdutoResponse>> listarAsync(ProdutoFiltros filtros,
                                                                  Pageable pageable) {
        return CompletableFuture.completedFuture(
                produtoRepository.findAll(filtros.toSpec(), pageable)
                        .map(ProdutoResponse::from)
                        .toList()
        );
    }
}
```

#### Configuração do `TaskExecutor` para `@Async`

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    // ─── Executor dedicado para operações assíncronas da aplicação ────────────
    @Bean("asyncExecutor")
    public Executor asyncExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");

        // Propaga o SecurityContext para threads filhas (@Async + Spring Security)
        executor.setTaskDecorator(new DelegatingSecurityContextTaskDecorator(
                new ContextPropagatingTaskDecorator()));

        // Política de rejeição: lança exceção quando fila está cheia
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    // ─── Handler global para exceções não capturadas em @Async ───────────────
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            LoggerFactory.getLogger(AsyncConfig.class)
                    .error("Erro assíncrono em {}: {}", method.getName(), ex.getMessage(), ex);
    }
}
```

> **Nota sobre Virtual Threads e `@Async`:** com `spring.threads.virtual.enabled=true`,
> o Spring Boot substitui o `SimpleAsyncTaskExecutor` padrão por um executor de
> Virtual Threads automaticamente. Nesse cenário, `@Async` com Virtual Threads
> elimina a necessidade de configurar pool sizes — cada tarefa recebe uma Virtual
> Thread própria sem custo de bloqueio.

#### `Callable<T>` — delegação simples ao `TaskExecutor`

```java
// Callable é a forma mais simples de assincronia sem CompletableFuture.
// Spring MVC executa o Callable num AsyncTaskExecutor e libera a thread do Servlet.
@GetMapping("/processamento-pesado")
public Callable<ResponseEntity<RelatorioResponse>> processamentoPesado(
        @RequestParam String periodo) {

    // O Callable é executado em outro thread — a thread do Servlet é liberada aqui
    return () -> {
        var relatorio = relatorioService.gerarCompleto(periodo); // pode demorar
        return ResponseEntity.ok(relatorio);
    };
}
```

#### `DeferredResult<T>` — resultado produzido por outro componente

```java
// DeferredResult é preenchido por um thread externo (event listener, fila, etc.)
// O Servlet aguarda sem bloquear thread. Ideal para long-polling e integração com
// sistemas de mensageria.
@Component
public class CotacaoController {

    // Mapa de cotações pendentes: tickerSymbol → lista de DeferredResults aguardando
    private final Map<String, List<DeferredResult<ResponseEntity<CotacaoResponse>>>> pendentes
            = new ConcurrentHashMap<>();

    @GetMapping("/api/v1/cotacoes/{ticker}/aguardar")
    @Operation(summary = "Aguarda a próxima atualização de cotação (long-polling)")
    public DeferredResult<ResponseEntity<CotacaoResponse>> aguardarCotacao(
            @PathVariable String ticker) {

        // Timeout de 30s — se não houver atualização, retorna 204
        var result = new DeferredResult<ResponseEntity<CotacaoResponse>>(30_000L,
                ResponseEntity.noContent().build());

        pendentes.computeIfAbsent(ticker, _ -> new CopyOnWriteArrayList<>()).add(result);
        result.onCompletion(() -> pendentes.getOrDefault(ticker, List.of()).remove(result));

        return result;
    }

    // Chamado por um @EventListener ou consumidor de fila quando chega nova cotação
    @EventListener
    public void onNovaCotacao(CotacaoAtualizadaEvent evento) {
        var resultados = pendentes.remove(evento.ticker());
        if (resultados != null) {
            var response = ResponseEntity.ok(CotacaoResponse.from(evento));
            resultados.forEach(r -> r.setResult(response)); // notifica todos os clientes
        }
    }
}
```

#### Tratamento de erros assíncronos no `@ControllerAdvice`

```java
// Exceções lançadas dentro de CompletableFuture são envolvidas em
// CompletionException — o Spring MVC desempacota automaticamente a causa
// antes de passar ao @ExceptionHandler.
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Este handler captura tanto exceções síncronas quanto as desempacotadas
    // de CompletableFuture (Spring MVC faz o unwrap de CompletionException)
    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ProblemDetail handleNotFound(RecursoNaoEncontradoException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    // AsyncRequestTimeoutException: lançada quando o timeout do DeferredResult
    // ou Callable expira sem que o resultado tenha sido preenchido
    @ExceptionHandler(AsyncRequestTimeoutException.class)
    @ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
    public ProblemDetail handleAsyncTimeout(AsyncRequestTimeoutException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.SERVICE_UNAVAILABLE,
                "Operação expirou — tente novamente");
    }
}
```

#### Configuração do timeout global de async no MVC

```java
// Em WebMvcConfigurer — define o timeout padrão para Callable e DeferredResult
@Override
public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    configurer.setDefaultTimeout(30_000L); // 30 segundos

    // Quando Virtual Threads estão habilitadas, o Spring Boot já configura
    // um VirtualThreadTaskExecutor automaticamente — não é necessário setar aqui.
    // Para pool de threads convencional:
    // configurer.setTaskExecutor(asyncExecutor());
}
```

#### Teste de endpoint assíncrono

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
class ProdutoAsyncControllerIT {

    @Autowired
    private RestTestClient restTestClient;

    @MockitoBean
    private ProdutoService produtoService;

    @Test
    @DisplayName("GET /{id} assíncrono → 200 quando produto existe")
    void buscar_Async_Returns200() {
        when(produtoService.buscarAsync(1L))
                .thenReturn(CompletableFuture.completedFuture(
                        new ProdutoResponse(1L, "Notebook", new BigDecimal("3499.99"),
                                "Informática", LocalDateTime.now(), LocalDateTime.now())));

        restTestClient.get()
                .uri("/api/v1/produtos/1")
                .exchange()
                .expectStatus().isOk()
                .expectBody(ProdutoResponse.class)
                .value(p -> assertThat(p.nome()).isEqualTo("Notebook"));
    }

    @Test
    @DisplayName("GET /{id}/preco-externo → 504 quando timeout")
    void buscarPrecoExterno_Timeout_Returns504() {
        when(produtoService.consultarFornecedorAsync(1L))
                .thenReturn(CompletableFuture.failedFuture(new TimeoutException()));

        restTestClient.get()
                .uri("/api/v1/produtos/1/preco-externo")
                .exchange()
                .expectStatus().isEqualTo(HttpStatus.GATEWAY_TIMEOUT);
    }
}
```

---

### 9.7 Acesso a Recursos do Servlet — `HttpServletRequest`, `HttpServletResponse` e `RequestContextHolder`

O Spring MVC expõe os objetos do Servlet diretamente como parâmetros de método
nos controllers. Para camadas mais internas (services, componentes) que não têm
acesso direto ao contexto da requisição, o `RequestContextHolder` fornece acesso
estático thread-safe.

#### Injeção direta nos controllers

```java
@RestController
@RequestMapping("/api/v1/exemplos")
public class ExemplosServletController {

    // ─── HttpServletRequest — dados brutos da requisição ─────────────────────
    @GetMapping("/request-info")
    public Map<String, Object> requestInfo(HttpServletRequest request) {
        return Map.of(
            "method",      request.getMethod(),
            "uri",         request.getRequestURI(),
            "queryString", Objects.requireNonNullElse(request.getQueryString(), ""),
            "remoteAddr",  request.getRemoteAddr(),
            "serverName",  request.getServerName(),
            "headers",     Collections.list(request.getHeaderNames())
                               .stream()
                               .collect(Collectors.toMap(
                                   h -> h,
                                   request::getHeader
                               ))
        );
    }

    // ─── HttpServletResponse — manipulação direta da resposta ────────────────
    @GetMapping("/download")
    public void download(HttpServletResponse response) throws IOException {
        response.setContentType("text/csv");
        response.setHeader(HttpHeaders.CONTENT_DISPOSITION,
                           "attachment; filename=\"dados.csv\"");
        response.setCharacterEncoding("UTF-8");

        try (var writer = response.getWriter()) {
            writer.write("id,nome,preco\n");
            writer.write("1,Produto A,99.90\n");
        }
        // Não retorna nada — a resposta já foi escrita diretamente
    }

    // ─── HttpSession — acesso direto ou criação lazy ──────────────────────────
    @GetMapping("/sessao")
    public Map<String, Object> sessao(HttpSession session) {
        // false = não cria sessão se não existir
        // Para acesso sem criar: request.getSession(false)
        return Map.of(
            "sessionId", session.getId(),
            "isNew",     session.isNew(),
            "maxInactive", session.getMaxInactiveInterval()
        );
    }

    // ─── WebRequest — abstração portável (funciona com Servlet e Portlet) ────
    @GetMapping("/web-request")
    public String webRequest(WebRequest webRequest,
                              NativeWebRequest nativeWebRequest) {
        // WebRequest: API portável do Spring
        String param = webRequest.getParameter("q");

        // NativeWebRequest: acesso ao objeto nativo quando necessário
        HttpServletRequest nativo = nativeWebRequest.getNativeRequest(HttpServletRequest.class);

        return param;
    }

    // ─── Principal — usuário autenticado via interface padrão Java EE ─────────
    @GetMapping("/principal")
    public String principal(Principal principal) {
        // Funciona com qualquer mecanismo de autenticação (Basic, JWT, OIDC...)
        return principal != null ? principal.getName() : "anônimo";
    }

    // ─── Locale — idioma/region do cliente ───────────────────────────────────
    @GetMapping("/locale")
    public String locale(Locale locale) {
        // Resolvido pelo LocaleResolver configurado (Accept-Language, cookie, session)
        return locale.toLanguageTag();  // ex: "pt-BR"
    }

    // ─── InputStream / OutputStream direto ───────────────────────────────────
    @PostMapping(value = "/raw-body", consumes = MediaType.APPLICATION_OCTET_STREAM_VALUE)
    public void rawBody(InputStream body, OutputStream out) throws IOException {
        // Leitura e escrita direta nos streams da requisição/resposta
        body.transferTo(out);
    }
}
```

#### `RequestContextHolder` — acesso fora do controller

O `RequestContextHolder` permite acessar o contexto da requisição corrente em
qualquer camada da aplicação — útil em services, interceptors e componentes que
não recebem o request por parâmetro.

```java
// ─── Utilitário de acesso ao contexto da requisição ──────────────────────────
//
// ATENÇÃO: use com parcimônia. Passar HttpServletRequest como parâmetro
// é mais explícito e testável. Use RequestContextHolder apenas quando
// não há acesso direto ao contexto (ex: dentro de um @Component genérico).
//
@Component
public class RequestContextUtils {

    /**
     * Retorna o HttpServletRequest da requisição corrente, ou null se chamado
     * fora do escopo de uma requisição HTTP (ex: thread de background).
     */
    public static HttpServletRequest currentRequest() {
        var attrs = RequestContextHolder.getRequestAttributes();
        if (attrs instanceof ServletRequestAttributes sra) {
            return sra.getRequest();
        }
        return null;
    }

    public static HttpServletResponse currentResponse() {
        var attrs = RequestContextHolder.getRequestAttributes();
        if (attrs instanceof ServletRequestAttributes sra) {
            return sra.getResponse();  // pode ser null se ainda não resolvido
        }
        return null;
    }

    public static HttpSession currentSession(boolean create) {
        var attrs = RequestContextHolder.getRequestAttributes();
        if (attrs instanceof ServletRequestAttributes sra) {
            return sra.getRequest().getSession(create);
        }
        return null;
    }

    /** Header da requisição corrente — útil para ler X-Tenant-Id, X-Correlation-Id etc. */
    public static String header(String name) {
        var req = currentRequest();
        return req != null ? req.getHeader(name) : null;
    }

    /** IP real do cliente, considerando proxies reversos. */
    public static String clientIp() {
        var req = currentRequest();
        if (req == null) return null;
        var forwarded = req.getHeader("X-Forwarded-For");
        if (forwarded != null && !forwarded.isBlank()) {
            return forwarded.split(",")[0].trim();
        }
        return req.getRemoteAddr();
    }
}

// ─── Uso em um @Service (sem receber request como parâmetro) ─────────────────
@Service
public class AuditService {

    public void registrar(String evento) {
        String ip          = RequestContextUtils.clientIp();
        String correlation = RequestContextUtils.header("X-Correlation-Id");
        String uri         = Optional.ofNullable(RequestContextUtils.currentRequest())
                                     .map(HttpServletRequest::getRequestURI)
                                     .orElse("unknown");

        log.info("Evento={} IP={} Correlation={} URI={}", evento, ip, correlation, uri);
    }
}
```

#### Propagação para threads assíncronas

```java
// RequestContextHolder é ThreadLocal — NÃO funciona em threads filhas por padrão.
// Para propagar o contexto em chamadas assíncronas:

@Configuration
public class AsyncConfig {

    @Bean
    public TaskExecutor asyncExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        // Propaga RequestAttributes e SecurityContext para threads do pool
        executor.setTaskDecorator(new RequestContextTaskDecorator());
        return executor;
    }
}

// Decorator que copia o contexto da thread pai para a thread filha
public class RequestContextTaskDecorator implements TaskDecorator {

    @Override
    public Runnable decorate(Runnable runnable) {
        // Captura o contexto da thread chamadora (request thread)
        var requestAttrs  = RequestContextHolder.getRequestAttributes();
        var securityCtx   = SecurityContextHolder.getContext();

        return () -> {
            try {
                RequestContextHolder.setRequestAttributes(requestAttrs);
                SecurityContextHolder.setContext(securityCtx);
                runnable.run();
            } finally {
                RequestContextHolder.resetRequestAttributes();
                SecurityContextHolder.clearContext();
            }
        };
    }
}
```

---

### 9.8 Integração com Spring Security

#### Recuperando o usuário autenticado no Controller

O Spring Security oferece três formas de acessar o usuário no controller, da
mais simples à mais poderosa:

```java
@RestController
@RequestMapping("/api/v1/perfil")
public class PerfilController {

    // ─── Forma 1: @AuthenticationPrincipal (recomendada) ─────────────────────
    //
    // Injeta diretamente o principal retornado pelo UserDetailsService.
    // Mais limpa: sem cast manual, sem acoplamento ao SecurityContextHolder.
    //
    @GetMapping
    public ResponseEntity<PerfilResponse> meuPerfil(
            @AuthenticationPrincipal UserDetails userDetails) {

        return ResponseEntity.ok(perfilService.buscar(userDetails.getUsername()));
    }

    // ─── Forma 1b: @AuthenticationPrincipal com tipo customizado ──────────────
    //
    // Quando UserDetailsService retorna uma implementação própria (UsuarioPrincipal),
    // o Spring faz o cast automaticamente — sem ClassCastException.
    //
    @GetMapping("/completo")
    public ResponseEntity<PerfilResponse> perfilCompleto(
            @AuthenticationPrincipal UsuarioPrincipal principal) {
        // UsuarioPrincipal é seu UserDetails customizado com dados extras
        return ResponseEntity.ok(perfilService.buscarCompleto(principal.getId()));
    }

    // ─── Forma 1c: @AuthenticationPrincipal com SpEL ─────────────────────────
    //
    // Extrai propriedade diretamente do principal usando Spring Expression Language.
    //
    @GetMapping("/id")
    public Long meuId(
            @AuthenticationPrincipal(expression = "id") Long userId) {
        return userId;
    }

    // ─── Forma 2: Authentication como parâmetro ───────────────────────────────
    //
    // Acesso ao objeto Authentication completo: principal, credentials,
    // authorities e detalhes adicionais. Útil quando se precisa das authorities.
    //
    @GetMapping("/roles")
    public List<String> minhasRoles(Authentication authentication) {
        return authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .toList();
    }

    // ─── Forma 3: SecurityContextHolder (acesso estático) ────────────────────
    //
    // Útil em camadas que não têm Authentication por parâmetro (services, utils).
    // Funciona na mesma thread da requisição (ThreadLocal).
    //
    @GetMapping("/direto")
    public String acessoDireto() {
        var auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()
                || auth instanceof AnonymousAuthenticationToken) {
            return "anônimo";
        }
        return auth.getName();
    }

    // ─── @CurrentSecurityContext — acesso ao SecurityContext completo ─────────
    //
    // Injeta o SecurityContext inteiro, não apenas o Authentication.
    // Raro, mas útil para lógica que precisa do contexto completo.
    //
    @GetMapping("/security-context")
    public String securityContext(
            @CurrentSecurityContext SecurityContext ctx) {
        var auth = ctx.getAuthentication();
        return auth != null ? auth.getName() : "anônimo";
    }
}
```

#### UserDetails customizado — boas práticas

```java
// ─── Principal customizado com dados do domínio ───────────────────────────────
public class UsuarioPrincipal implements UserDetails {

    private final Long id;
    private final String email;
    private final String nome;
    private final String senha;
    private final Collection<? extends GrantedAuthority> authorities;

    public UsuarioPrincipal(Usuario usuario) {
        this.id          = usuario.getId();
        this.email       = usuario.getEmail();
        this.nome        = usuario.getNome();
        this.senha       = usuario.getSenhaHash();
        this.authorities = usuario.getPerfis().stream()
                .map(p -> new SimpleGrantedAuthority("ROLE_" + p.name()))
                .toList();
    }

    // ─── Getters do domínio (não fazem parte da interface UserDetails) ────────
    public Long getId()    { return id; }
    public String getNome(){ return nome; }

    // ─── Interface UserDetails ────────────────────────────────────────────────
    @Override public String getUsername()               { return email; }
    @Override public String getPassword()               { return senha; }
    @Override public Collection<? extends GrantedAuthority> getAuthorities() { return authorities; }
    @Override public boolean isAccountNonExpired()      { return true; }
    @Override public boolean isAccountNonLocked()       { return true; }
    @Override public boolean isCredentialsNonExpired()  { return true; }
    @Override public boolean isEnabled()                { return true; }
}

// ─── UserDetailsService que retorna o tipo customizado ────────────────────────
@Service
public class UsuarioDetailsService implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return usuarioRepository.findByEmail(email)
                .map(UsuarioPrincipal::new)
                .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado: " + email));
    }
}
```

#### Uso do SecurityContextHolder em services

```java
// ─── Componente utilitário de segurança para uso em qualquer camada ───────────
@Component
public class SecurityUtils {

    /**
     * Retorna o usuário autenticado ou lança exceção se não houver autenticação.
     * Convenção: retorna Optional para não forçar tratamento de exceção em
     * endpoints públicos onde o usuário pode não estar autenticado.
     */
    public Optional<UsuarioPrincipal> usuarioAtual() {
        return Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
                .filter(auth -> auth.isAuthenticated()
                        && !(auth instanceof AnonymousAuthenticationToken))
                .map(Authentication::getPrincipal)
                .filter(UsuarioPrincipal.class::isInstance)
                .map(UsuarioPrincipal.class::cast);
    }

    public UsuarioPrincipal usuarioAtualOuErro() {
        return usuarioAtual()
                .orElseThrow(() -> new AccessDeniedException("Usuário não autenticado"));
    }

    public Long idUsuarioAtual() {
        return usuarioAtual().map(UsuarioPrincipal::getId).orElse(null);
    }

    public boolean temRole(String role) {
        return usuarioAtual()
                .map(u -> u.getAuthorities().stream()
                        .anyMatch(a -> a.getAuthority().equals("ROLE_" + role)))
                .orElse(false);
    }
}

// ─── Uso em serviços de domínio ───────────────────────────────────────────────
@Service
public class PedidoService {

    private final SecurityUtils securityUtils;

    public PedidoResponse criar(PedidoRequest request) {
        // Sem parâmetro extra no método — contexto capturado internamente
        var usuario = securityUtils.usuarioAtualOuErro();

        var pedido = new Pedido();
        pedido.setCliente(usuario.getId());
        // ...
        return PedidoResponse.from(pedido);
    }
}
```

#### Thymeleaf + Spring Security — templates seguros

Para usar as expressões de segurança no Thymeleaf, o artefato
`thymeleaf-extras-springsecurity6` deve estar no classpath. O namespace
`sec:` fica disponível automaticamente após a detecção da dependência.

```xml
<!-- pom.xml — necessário para o namespace sec: no Thymeleaf -->
<!-- ✅ Auto-configurado pelo ThymeleafSecurityDialect quando no classpath -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
    <!-- Versão gerenciada pelo Spring Boot BOM — não declare explicitamente -->
</dependency>
```

```html
<!-- templates/fragmentos/navbar.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<body>

<!-- ─── Exibição condicional por autenticação ─────────────────────────────── -->
<nav>
    <!-- Bloco visível apenas para usuários NÃO autenticados -->
    <div sec:authorize="!isAuthenticated()">
        <a th:href="@{/login}" class="btn btn-outline-primary">Entrar</a>
        <a th:href="@{/cadastro}" class="btn btn-primary">Cadastrar</a>
    </div>

    <!-- Bloco visível apenas para usuários autenticados -->
    <div sec:authorize="isAuthenticated()">
        <!-- Exibe o username (retorno de Authentication.getName()) -->
        <span>Olá, <strong sec:authentication="name">Usuário</strong>!</span>

        <!-- Exibe propriedade do principal customizado via SpEL -->
        <span sec:authentication="principal.nome">Nome</span>

        <!-- Exibe o e-mail (username configurado no UserDetailsService) -->
        <span sec:authentication="principal.username">e-mail</span>

        <!-- Avatar com iniciais do nome -->
        <span th:text="${#strings.substring(#authentication.principal.nome, 0, 1)}">A</span>

        <!-- Logout com CSRF token obrigatório -->
        <form th:action="@{/logout}" method="post">
            <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
            <button type="submit" class="btn btn-link">Sair</button>
        </form>
    </div>
</nav>

<!-- ─── Controle por role ──────────────────────────────────────────────────── -->
<aside>
    <!-- Visível apenas para ADMIN -->
    <a sec:authorize="hasRole('ADMIN')" th:href="@{/admin}">Painel Admin</a>

    <!-- Visível para ADMIN ou GERENTE -->
    <a sec:authorize="hasAnyRole('ADMIN', 'GERENTE')" th:href="@{/relatorios}">
        Relatórios
    </a>

    <!-- Visível para quem tem permissão específica (não role) -->
    <button sec:authorize="hasAuthority('PRODUTO_EDITAR')"
            th:onclick="|window.location='${@{/produtos/novo}}'|">
        Novo Produto
    </button>
</aside>

<!-- ─── Acesso a propriedades do principal em templates ───────────────────── -->
<div sec:authorize="isAuthenticated()" class="user-info">
    <!-- authentication.principal devolve o objeto UserDetails (ou customizado) -->
    <p>
        <span class="label">Nome:</span>
        <span sec:authentication="principal.nome">-</span>
    </p>
    <p>
        <span class="label">E-mail:</span>
        <span sec:authentication="principal.username">-</span>
    </p>
    <p>
        <span class="label">Perfis:</span>
        <!-- Lista de GrantedAuthority como string separado por vírgula -->
        <span th:text="${#authentication.principal.authorities}">-</span>
    </p>
</div>

<!-- ─── Expressões SpEL avançadas ─────────────────────────────────────────── -->
<!-- hasPermission() requer PermissionEvaluator customizado -->
<button sec:authorize="hasPermission(#pedido, 'cancelar')">Cancelar pedido</button>

<!-- Checar role E estar em rota específica -->
<a sec:authorize="hasRole('ADMIN') and isFullyAuthenticated()">Área restrita</a>

<!-- ─── Uso no controlador SSR para popular model com dados do usuário ─────── -->
<!--
    No controller, você pode adicionar dados do usuário ao model explicitamente:

    @GetMapping("/dashboard")
    public String dashboard(@AuthenticationPrincipal UsuarioPrincipal principal,
                             Model model) {
        model.addAttribute("usuario", principal);
        return "dashboard";
    }

    Ou usar @ModelAttribute global no @ControllerAdvice para disponibilizar
    o usuário em TODAS as views sem repetição:

    @ControllerAdvice
    public class SecurityModelAdvice {
        @ModelAttribute("usuarioLogado")
        public UsuarioPrincipal usuarioLogado(
                @AuthenticationPrincipal UsuarioPrincipal principal) {
            return principal;  // null quando não autenticado
        }
    }

    Então no template: th:text="${usuarioLogado?.nome}"
-->
</body>
</html>
```

#### @ControllerAdvice global para dados de segurança nas views SSR

```java
// ─── Disponibiliza dados do usuário em TODAS as views Thymeleaf ───────────────
//
// Alternativa ao sec:authentication do Thymeleaf quando se precisa de
// propriedades do domínio que não estão na interface UserDetails.
//
@ControllerAdvice
public class SecurityModelAdvice {

    /**
     * Injeta o usuário autenticado no model de TODAS as requisições MVC.
     * Retorna null se não autenticado — templates usam ${usuarioLogado?.nome}.
     */
    @ModelAttribute("usuarioLogado")
    public UsuarioPrincipal usuarioLogado(
            @AuthenticationPrincipal UsuarioPrincipal principal) {
        return principal;
    }

    /** Disponibiliza a lista de roles para lógica condicional nos templates. */
    @ModelAttribute("roles")
    public Set<String> roles(Authentication authentication) {
        if (authentication == null) return Set.of();
        return authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.toSet());
    }
}
```

```html
<!-- Uso nas views sem nenhum @sec adicional ──────────────────────────────── -->
<div th:if="${usuarioLogado != null}">
    <p th:text="${usuarioLogado.nome}">Nome</p>
    <p th:text="${usuarioLogado.email}">Email</p>

    <!-- roles vem do @ModelAttribute("roles") -->
    <a th:if="${roles.contains('ROLE_ADMIN')}" th:href="@{/admin}">Admin</a>
</div>
```

#### Diagrama — fluxo de resolução do usuário autenticado

```mermaid
flowchart TD
    REQ["HTTP Request<br>(JWT / Session / Basic)"] --> FILTER["Spring Security FilterChain"]
    FILTER --> SC["SecurityContextHolder<br>ThreadLocal&lt;SecurityContext&gt;"]
    SC --> AUTH["Authentication<br>(principal, authorities, credentials)"]

    subgraph Controller ["Controller / Service"]
        AP["@AuthenticationPrincipal<br>injeta o principal diretamente"]
        APARAM["Authentication param<br>injeta o objeto completo"]
        SCH["SecurityContextHolder<br>.getContext().getAuthentication()"]
        CSC["@CurrentSecurityContext<br>injeta o SecurityContext"]
    end

    AUTH --> AP
    AUTH --> APARAM
    AUTH --> SCH
    AUTH --> CSC

    subgraph Thymeleaf ["Thymeleaf (SSR)"]
        SECNS["sec:authentication=&quot;name&quot;<br>sec:authorize=&quot;hasRole(...)&quot;"]
        MODEL["Model attribute<br>@ControllerAdvice"]
    end

    AUTH --> SECNS
    AUTH --> MODEL

    style SC fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style FILTER fill:#FAECE7,stroke:#993C1D,color:#4A1B0C
    style AP fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style APARAM fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style SCH fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style CSC fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style SECNS fill:#EAF3DE,stroke:#3B6D11,color:#27500A
    style MODEL fill:#EAF3DE,stroke:#3B6D11,color:#27500A
```

---


## 10. Alternativas ao Thymeleaf

### 10.1 Comparativo de Engines de Template

```mermaid
graph TB
    subgraph "Engines SSR para Spring MVC"
        TH["Thymeleaf 3.x<br/>★★★★★<br/>HTML válido, dialeto rico,<br/>integração Spring perfeita"]
        FM["FreeMarker<br/>★★★★☆<br/>Alto desempenho,<br/>sintaxe própria (*.ftlh)"]
        MU["Mustache<br/>★★★☆☆<br/>Logic-less, simples,<br/>portável entre linguagens"]
        JTE["JTE (Java Template Engine)<br/>★★★★☆<br/>Type-safe, compilado,<br/>excelente performance"]
    end
```

### 10.2 FreeMarker

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

```yaml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    suffix: .ftlh
    charset: UTF-8
    cache: false
    settings:
      number_format: ",##0.##"
      date_format: dd/MM/yyyy
```

```html
<!-- templates/produtos/lista.ftlh -->
<#list produtos as produto>
    <p>${produto.nome} — R$ ${produto.preco?string("0.00")}</p>
</#list>

<!-- Macro reutilizável (equivale a fragment do Thymeleaf) -->
<#macro campo label nome>
    <div class="mb-3">
        <label>${label}</label>
        <input type="text" name="${nome}" value="${(form[nome])!}">
    </div>
</#macro>
<@campo label="Nome" nome="nome"/>
```

### 10.3 JTE (Java Template Engine) — Recomendado para Performance

JTE compila os templates para bytecode Java, oferecendo type-safety em tempo de compilação e desempenho superior ao Thymeleaf.

```xml
<dependency>
    <groupId>gg.jte</groupId>
    <artifactId>jte-spring-boot-starter-3</artifactId>
    <version>3.1.12</version>
</dependency>
```

```java
// src/main/jte/produtos/lista.jte
@import br.com.app.dto.ProdutoResponse
@import java.util.List

@param List<ProdutoResponse> produtos
@param String busca

<!DOCTYPE html>
<html>
<body>
<h1>Produtos</h1>
@for(var produto : produtos)
    <div class="card">
        <h3>${produto.nome()}</h3>   <%-- Erro de compilação se campo não existir! --%>
        <p>R$ ${produto.preco()}</p>
    </div>
@endfor

@if(produtos.isEmpty())
    <p>Nenhum produto encontrado.</p>
@endif
</body>
</html>
```

### 10.4 Quando Usar Cada Template Engine

| Cenário | Recomendação |
|---|---|
| Projeto Spring Boot padrão | **Thymeleaf** — melhor ecossistema, Spring Security integrado |
| Alta performance / muitas requests | **JTE** — compilado, type-safe |
| Templates de e-mail | **Thymeleaf** — dialeto rico para HTML de e-mail |
| Equipe com Java forte, sem UX | **JTE** — erros em tempo de compilação |
| Templates portáveis (Java + Node) | **Mustache** |
| Geração de documentos (PDF, XML) | **FreeMarker** — mais controle sobre output |

---



## 11. Boas Práticas e Checklist

### 11.1 Diagrama de Fluxo de Decisão

```mermaid
flowchart TD
    A[Novo Endpoint] --> B{REST ou SSR?}

    B -->|REST| C[Criar DTO com Records]
    B -->|SSR| D[Criar Form Object + Template]

    C --> E[Adicionar anotações Bean Validation]
    D --> E

    E --> F[Criar Controller com mapeamento correto]
    F --> G{Requer conversão<br/>de tipos?}

    G -->|Sim| H[Implementar Converter<br/>ou Formatter]
    G -->|Não| I[Implementar lógica de serviço]
    H --> I

    I --> J[Configurar tratamento de erros<br/>@ControllerAdvice]
    J --> K{REST?}

    K -->|Sim| L[Documentar com OpenAPI]
    K -->|Não| M[Testar formulário com<br/>cenários de erro]

    L --> N[Escrever testes]
    M --> N

    N --> O[@WebMvcTest + @SpringBootTest<br/>RestTestClient ou REST Assured]
```

### 11.2 Checklist — SSR Controllers

- [ ] Usar `@Controller` (não `@RestController`)
- [ ] Criar Form Objects separados das entidades/DTOs
- [ ] Sempre checar `bindingResult.hasErrors()` ANTES de usar o form
- [ ] Usar `RedirectAttributes.addFlashAttribute()` para mensagens pós-redirect (PRG)
- [ ] Usar `@ModelAttribute` para dados comuns (select options, etc.)
- [ ] Configurar `HiddenHttpMethodFilter` para PUT/DELETE em forms HTML
- [ ] Aplicar `StringTrimmerEditor` via `@InitBinder`
- [ ] Definir `setAllowedFields` para prevenir mass assignment

### 11.3 Checklist — Validação

- [ ] Bean Validation nas camadas Controller E Service (`@Validated`)
- [ ] Usar grupos de validação para criar/atualizar com regras diferentes
- [ ] Constraints customizadas para regras de domínio (CPF, CNPJ, etc.)
- [ ] `@Valid` em objetos aninhados e coleções para cascata
- [ ] Mensagens de erro externalizadas em `messages.properties`
- [ ] `@EmailUnico`, `@CpfUnico` com acesso ao repositório via Spring

### 11.4 Anti-patterns a Evitar

```java
// ❌ Expondo entidade JPA diretamente
@GetMapping("/{id}")
public Produto buscar(@PathVariable Long id) { ... }

// ✅ DTO como contrato da API
@GetMapping("/{id}")
public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) { ... }

// ❌ Lógica de negócio no Controller
@PostMapping
public ResponseEntity<?> criar(@RequestBody ProdutoRequest req) {
    if (produtoRepository.existsBySku(req.sku())) {
        return ResponseEntity.badRequest().body("SKU duplicado");
    }
    produtoRepository.save(new Produto(req.nome(), req.preco()));
    // ...
}

// ✅ Controller delega para Service
@PostMapping
public ResponseEntity<ProdutoResponse> criar(
        @RequestBody @Valid ProdutoCreateRequest req,
        UriComponentsBuilder uriBuilder) {
    var produto = produtoService.criar(req);
    var location = uriBuilder.path("/api/v1/produtos/{id}")
            .buildAndExpand(produto.id()).toUri();
    return ResponseEntity.created(location).body(produto);
}

// ❌ Verificar BindingResult DEPOIS de usar o form
@PostMapping
public String salvar(@ModelAttribute @Valid ProdutoForm form,
                     BindingResult result, Model model) {
    produtoService.salvar(form);  // ERRO: usa o form antes de checar!
    if (result.hasErrors()) return "formulario";
    return "redirect:/lista";
}

// ✅ Verificar BindingResult PRIMEIRO
@PostMapping
public String salvar(@ModelAttribute @Valid ProdutoForm form,
                     BindingResult result, Model model) {
    if (result.hasErrors()) return "formulario";  // Verifica ANTES
    produtoService.salvar(form);
    return "redirect:/lista";
}

// ❌ @InitBinder sem whitelist (vulnerável a mass assignment)
@InitBinder
public void init(WebDataBinder binder) {
    binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
    // Esqueceu setAllowedFields!
}

// ✅ Sempre proteger com whitelist
@InitBinder
public void init(WebDataBinder binder) {
    binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
    binder.setAllowedFields("nome", "descricao", "preco", "estoque", "categoriaId");
}
```

### 11.5 Mensagens de Validação i18n — Integração com messages.properties

Por padrão o Bean Validation resolve mensagens no arquivo
`ValidationMessages.properties` (padrão Jakarta) ou dentro das próprias
anotações. Para usar o sistema de mensagens do Spring (`messages.properties`)
— unificando as traduções de validação com as demais mensagens da aplicação —
é necessário conectar o `MessageSource` ao validador.

#### O que o Spring Boot faz automaticamente

| Comportamento | Auto-configurado? |
|---|---|
| `MessageSource` lendo `classpath:messages*.properties` | ✅ `MessageSourceAutoConfiguration` |
| Validador padrão (`javax.validation`) | ✅ `ValidationAutoConfiguration` |
| Validador integrado ao `MessageSource` do Spring | ❌ **Requer configuração manual** |
| Interpolação `{min}`, `{max}`, `{value}` nos arquivos `.properties` | ✅ Funcionam após integração |

> **Por que não é automático?** O `ValidationAutoConfiguration` registra um
> `LocalValidatorFactoryBean`, mas sem apontar para o `MessageSource` da
> aplicação. O Spring Boot *evita* sobrescrever o bean do usuário, por isso
> a integração precisa ser declarada explicitamente.

#### Convenções de nomenclatura de chaves

O Bean Validation procura mensagens na seguinte ordem de prioridade:

1. `{chave}` literal definida no atributo `message` da constraint
2. `{NomeDaConstraint.nomeDoTipo.nomeDoCampo}` — ex.: `NotBlank.clienteRequest.cpf`
3. `{NomeDaConstraint.nomeDoCampo}` — ex.: `NotBlank.cpf`
4. `{NomeDaConstraint.tipoPrimitivo}` — ex.: `NotBlank.java.lang.String`
5. `{NomeDaConstraint}` — ex.: `NotBlank`

```properties
# src/main/resources/messages.properties
# ─── Chaves explícitas referenciadas com {chave} nas anotações ────────────────
produto.nome.obrigatorio=Nome do produto é obrigatório
produto.nome.tamanho=Nome deve ter entre {min} e {max} caracteres
produto.preco.minimo=Preço deve ser maior que {value}
produto.estoque.invalido=Estoque não pode ser negativo

# ─── Constraints customizadas ─────────────────────────────────────────────────
br.com.app.validation.cpf.invalido=CPF inválido
br.com.app.validation.email.unico=E-mail já cadastrado no sistema

# ─── Chaves por convenção de nome (sem precisar declarar message=) ────────────
# Bean Validation resolve automaticamente pelo padrão: ConstraintName.campo
NotBlank.clienteRequest.nome=Nome é obrigatório
Size.clienteRequest.nome=Nome deve ter entre {min} e {max} caracteres
Email.clienteRequest.email=E-mail inválido

# ─── Fallback genérico por tipo de constraint ─────────────────────────────────
NotBlank=Campo obrigatório
NotNull=Campo obrigatório
Size=Tamanho inválido: deve ter entre {min} e {max} caracteres
```

```java
// ─── Configuração necessária para integrar MessageSource ao validador ─────────
//
// ⚠️  Spring Boot NÃO faz isso automaticamente.
//     Sem este bean, mensagens como {produto.nome.obrigatorio} ficam
//     literalmente na resposta em vez de serem resolvidas.
//
@Configuration
public class ValidationConfig {

    // MessageSource já é auto-configurado pelo Spring Boot a partir de
    // messages.properties — este bean é necessário apenas para wiring manual.
    @Bean
    public LocalValidatorFactoryBean validator(MessageSource messageSource) {
        var factory = new LocalValidatorFactoryBean();
        // Aponta o validador para o MessageSource da aplicação
        factory.setValidationMessageSource(messageSource);
        return factory;
    }
}
```

```java
// ─── Uso nas constraints: referencias com {chave} ─────────────────────────────
public record ProdutoRequest(

    // Referência explícita a chave do messages.properties
    @NotBlank(message = "{produto.nome.obrigatorio}")
    @Size(min = 2, max = 200, message = "{produto.nome.tamanho}")
    String nome,

    // Sem message= : Bean Validation usa a convenção de nomes (NotNull.preco,
    // NotNull.java.math.BigDecimal ou NotNull — nessa ordem de prioridade)
    @NotNull
    @Positive
    BigDecimal preco,

    // Interpolação de atributos da própria anotação: {min}, {max}, {value}
    // funcionam dentro dos messages.properties após a integração acima
    @Size(min = 3, max = 50)  // {min}=3 e {max}=50 ficam disponíveis no template
    String sku
) {}
```

```yaml
# application.yml — configuração do MessageSource (auto-configurado pelo Boot)
# Estas propriedades são gerenciadas por MessageSourceAutoConfiguration.
spring:
  messages:
    basename: messages          # padrão; separe com vírgula para múltiplos arquivos
    encoding: UTF-8             # padrão UTF-8
    cache-duration: 1s          # 0 = sem cache (útil em desenvolvimento)
    use-code-as-default-message: false  # false = lança exceção se chave não existir
```



## 12. ETag e Cache HTTP

### 12.1 Cache HTTP em Views SSR

```java
// Para páginas Thymeleaf com dados relativamente estáveis
@Controller
@RequestMapping("/catalogo")
public class CatalogoMvcController {

    @GetMapping("/vitrine")
    public String vitrine(Model model, HttpServletResponse response) {
        // Adiciona headers de cache à resposta HTML
        response.setHeader(HttpHeaders.CACHE_CONTROL,
                "public, max-age=300, must-revalidate");

        model.addAttribute("produtos", catalogoService.destaques());
        return "catalogo/vitrine";
    }
}
```

```yaml
# application.yml — Cache-Control para recursos estáticos (auto-configurado pelo Boot)
spring:
  web:
    resources:
      cache:
        period: 3600          # segundos — padrão 0 (sem cache)
        cachecontrol:
          max-age: 3600
          cache-public: true
          must-revalidate: true
```

### 12.2 Resumo: Quando Usar Cada Estratégia

| Recurso | Estratégia recomendada | Headers |
|---|---|---|
| Dados que mudam raramente (categorias, config) | `max-age` longo + `public` | `Cache-Control: public, max-age=3600` |
| Dados por usuário | `max-age` curto + `private` | `Cache-Control: private, max-age=60` |
| Dados voláteis mas verificáveis | `no-cache` + ETag | `ETag: "abc"`, `Cache-Control: no-cache` |
| Dados em tempo real (estoque, preço ao vivo) | `no-store` | `Cache-Control: no-store` |
| Assets com hash no nome | `immutable` | `Cache-Control: public, max-age=31536000, immutable` |
| API paginada | ETag por página + `max-age` curto | `ETag: "page-0-hash"` |

---



## 13. Upload de Arquivos

### 13.1 Configuração

```yaml
# application.yml — MultipartAutoConfiguration (✅ auto-configurado pelo Boot)
spring:
  servlet:
    multipart:
      enabled: true             # ✅ Default: true
      max-file-size: 10MB       # ✅ Default: 1MB — tamanho máximo por arquivo
      max-request-size: 50MB    # ✅ Default: 10MB — tamanho total da requisição
      file-size-threshold: 2KB  # ✅ Default: 0 — acima disso grava em disco temporário
      location: /tmp/uploads    # ✅ Default: diretório temporário do SO
```

### 13.2 Controller de Upload

```java
@RestController
@RequestMapping("/api/v1/arquivos")
public class ArquivoController {

    private final ArquivoService arquivoService;

    // ─── Upload simples — arquivo único ──────────────────────────────────────
    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @Operation(summary = "Upload de arquivo único")
    public ResponseEntity<ArquivoResponse> upload(
            @RequestParam("arquivo") MultipartFile arquivo) {

        validarArquivo(arquivo);
        var response = arquivoService.salvar(arquivo);

        return ResponseEntity.created(
                URI.create("/api/v1/arquivos/" + response.id()))
                .body(response);
    }

    // ─── Upload com metadados — @RequestPart ─────────────────────────────────
    // @RequestPart permite enviar JSON + arquivo na mesma requisição multipart
    @PostMapping(value = "/com-metadados", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<ArquivoResponse> uploadComMetadados(
            @RequestPart("arquivo")   MultipartFile arquivo,
            @RequestPart("metadados") @Valid ArquivoMetadadosRequest metadados) {

        validarArquivo(arquivo);
        return ResponseEntity.ok(arquivoService.salvarComMetadados(arquivo, metadados));
    }

    // ─── Upload múltiplo ──────────────────────────────────────────────────────
    @PostMapping(value = "/multiplos", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<List<ArquivoResponse>> uploadMultiplo(
            @RequestParam("arquivos") List<MultipartFile> arquivos) {

        if (arquivos.size() > 10) {
            throw new NegocioException("Máximo de 10 arquivos por requisição");
        }
        arquivos.forEach(this::validarArquivo);

        return ResponseEntity.ok(arquivoService.salvarTodos(arquivos));
    }

    // ─── Validação do arquivo ─────────────────────────────────────────────────
    private void validarArquivo(MultipartFile arquivo) {
        if (arquivo.isEmpty()) {
            throw new NegocioException("Arquivo não pode ser vazio");
        }
        if (arquivo.getSize() > 10 * 1024 * 1024) { // 10 MB
            throw new NegocioException("Arquivo excede o tamanho máximo de 10 MB");
        }

        // Validação de MIME type real (não confia apenas na extensão)
        String contentType = detectarMimeType(arquivo);
        var permitidos = Set.of("image/jpeg", "image/png", "image/webp",
                                "application/pdf", "text/csv");
        if (!permitidos.contains(contentType)) {
            throw new NegocioException(
                    "Tipo de arquivo não permitido: " + contentType);
        }
    }

    private String detectarMimeType(MultipartFile arquivo) {
        try {
            // Apache Tika ou Files.probeContentType são mais seguros que
            // confiar no Content-Type declarado pelo cliente
            var tika = new Tika();
            return tika.detect(arquivo.getInputStream());
        } catch (IOException e) {
            throw new NegocioException("Não foi possível verificar o tipo do arquivo");
        }
    }
}
```

### 13.3 Service — Estratégias de Armazenamento

```java
@Service
public class ArquivoService {

    private final Path storageDir;

    public ArquivoService(@Value("${app.storage.dir:uploads}") String dir) {
        this.storageDir = Paths.get(dir).toAbsolutePath().normalize();
        try {
            Files.createDirectories(storageDir);
        } catch (IOException e) {
            throw new IllegalStateException("Não foi possível criar diretório de uploads", e);
        }
    }

    // ─── Estratégia 1: disco local ────────────────────────────────────────────
    public ArquivoResponse salvarLocalmente(MultipartFile arquivo) {
        // Nunca usar o nome original diretamente — risco de path traversal
        String extensao  = StringUtils.getFilenameExtension(arquivo.getOriginalFilename());
        String nomeSeguro = UUID.randomUUID() + (extensao != null ? "." + extensao : "");
        Path destino = storageDir.resolve(nomeSeguro);

        try {
            Files.copy(arquivo.getInputStream(), destino,
                    StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new NegocioException("Erro ao salvar arquivo: " + e.getMessage());
        }

        return new ArquivoResponse(
                UUID.randomUUID().toString(),
                nomeSeguro,
                arquivo.getSize(),
                arquivo.getContentType(),
                "/api/v1/arquivos/" + nomeSeguro);
    }

    // ─── Estratégia 2: S3 / MinIO via Spring Cloud AWS ───────────────────────
    public ArquivoResponse salvarS3(MultipartFile arquivo, S3Template s3Template) {
        String chave = "uploads/" + UUID.randomUUID() + "/"
                + arquivo.getOriginalFilename();
        try {
            var upload = s3Template.upload(
                    "meu-bucket", chave,
                    arquivo.getInputStream(),
                    ObjectMetadata.builder()
                            .contentType(arquivo.getContentType())
                            .contentLength(arquivo.getSize())
                            .build());

            return new ArquivoResponse(
                    chave,
                    arquivo.getOriginalFilename(),
                    arquivo.getSize(),
                    arquivo.getContentType(),
                    upload.url().toString());
        } catch (IOException e) {
            throw new NegocioException("Erro ao enviar para S3: " + e.getMessage());
        }
    }
}
```

### 13.4 Download de Arquivos

```java
@GetMapping("/{nomeArquivo:.+}")
public ResponseEntity<Resource> download(@PathVariable String nomeArquivo) {
    Path caminho = storageDir.resolve(nomeArquivo).normalize();

    // Proteção contra path traversal: garante que o arquivo está dentro do storageDir
    if (!caminho.startsWith(storageDir)) {
        throw new RecursoNaoEncontradoException("Arquivo", nomeArquivo);
    }

    Resource resource = new FileSystemResource(caminho);
    if (!resource.exists() || !resource.isReadable()) {
        throw new RecursoNaoEncontradoException("Arquivo", nomeArquivo);
    }

    String contentType;
    try {
        contentType = Files.probeContentType(caminho);
    } catch (IOException e) {
        contentType = MediaType.APPLICATION_OCTET_STREAM_VALUE;
    }

    return ResponseEntity.ok()
            .contentType(MediaType.parseMediaType(contentType))
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    ContentDisposition.attachment()
                            .filename(nomeArquivo, StandardCharsets.UTF_8)
                            .build()
                            .toString())
            .body(resource);
}
```

### 13.5 Upload via Fetch API (JavaScript)

#### Upload simples com arquivo único

```html
<!-- templates/arquivos/upload.html (Thymeleaf) -->
<form id="uploadForm">
    <input type="file" id="arquivo" name="arquivo" accept="image/*,.pdf,.csv">
    <div id="progress" style="display:none">
        <div id="progressBar" style="width:0%;height:8px;background:#0d6efd;transition:width .2s"></div>
        <span id="progressText">0%</span>
    </div>
    <button type="submit" class="btn btn-primary">Enviar</button>
</form>
<div id="resultado"></div>

<script>
document.getElementById('uploadForm').addEventListener('submit', async (e) => {
    e.preventDefault();

    const arquivo = document.getElementById('arquivo').files[0];
    if (!arquivo) return alert('Selecione um arquivo');

    const formData = new FormData();
    formData.append('arquivo', arquivo);          // nome deve corresponder ao @RequestParam

    const csrfToken  = document.querySelector('meta[name="_csrf"]')?.content;
    const csrfHeader = document.querySelector('meta[name="_csrf_header"]')?.content;

    const headers = {};
    if (csrfToken && csrfHeader) {
        headers[csrfHeader] = csrfToken;          // CSRF para apps SSR com Spring Security
    }

    try {
        // XMLHttpRequest para acompanhar progresso (Fetch API não suporta upload progress)
        await uploadComProgresso('/api/v1/arquivos', formData, headers);
    } catch (err) {
        document.getElementById('resultado').innerHTML =
            `<div class="alert alert-danger">Erro: ${err.message}</div>`;
    }
});

function uploadComProgresso(url, formData, headers) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();

        xhr.upload.addEventListener('progress', (e) => {
            if (e.lengthComputable) {
                const pct = Math.round((e.loaded / e.total) * 100);
                document.getElementById('progress').style.display = 'block';
                document.getElementById('progressBar').style.width = pct + '%';
                document.getElementById('progressText').textContent = pct + '%';
            }
        });

        xhr.addEventListener('load', () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                const data = JSON.parse(xhr.responseText);
                document.getElementById('resultado').innerHTML =
                    `<div class="alert alert-success">
                        Arquivo enviado: <a href="${data.url}">${data.nome}</a>
                     </div>`;
                resolve(data);
            } else {
                const err = JSON.parse(xhr.responseText);
                reject(new Error(err.detail || 'Erro no upload'));
            }
        });

        xhr.addEventListener('error', () => reject(new Error('Falha na conexão')));

        xhr.open('POST', url);
        Object.entries(headers).forEach(([k, v]) => xhr.setRequestHeader(k, v));
        xhr.send(formData);
    });
}
</script>
```

#### Upload com múltiplos arquivos e pré-visualização

```html
<input type="file" id="arquivos" multiple accept="image/*">
<div id="preview" class="d-flex flex-wrap gap-2 my-3"></div>
<button id="btnEnviar" class="btn btn-primary" disabled>Enviar todos</button>

<script>
const input     = document.getElementById('arquivos');
const preview   = document.getElementById('preview');
const btnEnviar = document.getElementById('btnEnviar');
const CSRF      = document.querySelector('meta[name="_csrf"]')?.content;
const CSRF_HDR  = document.querySelector('meta[name="_csrf_header"]')?.content;

input.addEventListener('change', () => {
    preview.innerHTML = '';
    [...input.files].forEach(file => {
        if (!file.type.startsWith('image/')) return;
        const reader = new FileReader();
        reader.onload = (e) => {
            const img = document.createElement('img');
            img.src = e.target.result;
            img.style.cssText = 'width:80px;height:80px;object-fit:cover;border-radius:4px';
            img.title = file.name;
            preview.appendChild(img);
        };
        reader.readAsDataURL(file);
    });
    btnEnviar.disabled = input.files.length === 0;
});

btnEnviar.addEventListener('click', async () => {
    const formData = new FormData();
    [...input.files].forEach(f => formData.append('arquivos', f)); // mesmo @RequestParam

    const headers = {};
    if (CSRF && CSRF_HDR) headers[CSRF_HDR] = CSRF;

    const res = await fetch('/api/v1/arquivos/multiplos', {
        method: 'POST',
        headers,
        body: formData
        // NÃO definir Content-Type — o browser define com o boundary correto
    });

    if (!res.ok) {
        const err = await res.json();
        alert('Erro: ' + (err.detail ?? res.statusText));
        return;
    }

    const arquivos = await res.json();
    console.log('Enviados:', arquivos);
});
</script>
```

#### Upload de arquivo com JSON (usando `@RequestPart`)

```javascript
// Fetch API: envia arquivo + JSON na mesma requisição multipart
async function uploadComMetadados(arquivo, metadados) {
    const formData = new FormData();

    // Parte 1: arquivo binário
    formData.append('arquivo', arquivo);

    // Parte 2: JSON como Blob com Content-Type explícito
    // Necessário para que o Spring MVC deserialize o JSON corretamente com @RequestPart
    formData.append(
        'metadados',
        new Blob([JSON.stringify(metadados)], { type: 'application/json' })
    );

    const res = await fetch('/api/v1/arquivos/com-metadados', {
        method: 'POST',
        headers: {
            [document.querySelector('meta[name="_csrf_header"]').content]:
             document.querySelector('meta[name="_csrf"]').content
        },
        body: formData
    });

    if (!res.ok) throw new Error(await res.text());
    return res.json();
}

// Exemplo de uso
uploadComMetadados(
    document.getElementById('arquivo').files[0],
    { titulo: 'Foto do produto', descricao: 'Vista frontal', publica: true }
).then(r => console.log('Arquivo salvo:', r));
```

#### Tratamento de erros de upload no `@ControllerAdvice`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Arquivo maior que spring.servlet.multipart.max-file-size
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    @ResponseStatus(HttpStatus.PAYLOAD_TOO_LARGE)
    public ProblemDetail handleMaxSize(MaxUploadSizeExceededException ex) {
        return ProblemDetail.forStatusAndDetail(
                HttpStatus.PAYLOAD_TOO_LARGE,
                "Arquivo excede o tamanho máximo permitido");
    }

    // Arquivo corrompido ou leitura falhou
    @ExceptionHandler(MultipartException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleMultipart(MultipartException ex) {
        return ProblemDetail.forStatusAndDetail(
                HttpStatus.BAD_REQUEST,
                "Requisição multipart inválida: " + ex.getMessage());
    }
}
```

---



## 14. Internacionalização (i18n)

### 14.1 Estratégias de Resolução de Locale

```mermaid
flowchart TD
    REQ["HTTP Request"] --> LR

    subgraph LR["LocaleResolver (escolha uma)"]
        AH["AcceptHeaderLocaleResolver<br>Lê Accept-Language: pt-BR, en<br>Sem estado — não persiste<br>Default do Spring Boot"]
        CK["CookieLocaleResolver<br>Armazena locale em cookie<br>Persiste entre sessões<br>Ideal para SPAs e SSR"]
        SS["SessionLocaleResolver<br>Armazena locale na HttpSession<br>Persiste na sessão<br>Ideal para SSR autenticado"]
        FX["FixedLocaleResolver<br>Locale fixo — nunca muda<br>útil em APIs internas"]
    end

    LR -->|LocaleChangeInterceptor| CTRL["Controller / Template"]
```

### 14.2 Configuração Completa

```java
@Configuration
public class I18nConfig implements WebMvcConfigurer {

    // ─── 1. LocaleResolver — escolha conforme o tipo de aplicação ─────────────

    // Para SSR com usuário logado: SessionLocaleResolver
    @Bean
    public LocaleResolver localeResolver() {
        var resolver = new CookieLocaleResolver("APP_LOCALE");
        resolver.setDefaultLocale(new Locale("pt", "BR"));
        resolver.setDefaultTimeZone(TimeZone.getTimeZone("America/Sao_Paulo"));
        resolver.setCookieMaxAge(Duration.ofDays(365));
        resolver.setCookieHttpOnly(true);
        resolver.setCookieSecure(true); // apenas HTTPS em produção
        return resolver;
    }

    // Para REST APIs stateless: AcceptHeaderLocaleResolver
    // @Bean
    // public LocaleResolver localeResolver() {
    //     var resolver = new AcceptHeaderLocaleResolver();
    //     resolver.setDefaultLocale(new Locale("pt", "BR"));
    //     resolver.setSupportedLocales(List.of(
    //         new Locale("pt", "BR"),
    //         Locale.ENGLISH
    //     ));
    //     return resolver;
    // }

    // ─── 2. LocaleChangeInterceptor — muda locale via query param ─────────────
    // GET /qualquer-rota?lang=en  → muda para inglês
    // GET /qualquer-rota?lang=pt-BR → muda para português
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        var interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang");
        return interceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }

    // ─── 3. MessageSource — carrega os arquivos de mensagens ──────────────────
    // ✅ Spring Boot auto-configura via MessageSourceAutoConfiguration
    // Declare apenas para customizar (charset, cache, múltiplos basenames)
    @Bean
    public MessageSource messageSource() {
        var source = new ReloadableResourceBundleMessageSource();
        source.setBasenames(
            "classpath:messages",        // messages_pt_BR.properties, messages_en.properties
            "classpath:validation-messages" // separado para mensagens de validação
        );
        source.setDefaultEncoding("UTF-8");
        source.setDefaultLocale(new Locale("pt", "BR"));
        source.setCacheSeconds(3600);    // 0 = sem cache (dev), 3600 (prod)
        source.setUseCodeAsDefaultMessage(false);
        return source;
    }

    // ─── 4. Conectar MessageSource ao Bean Validation ─────────────────────────
    // ⚠️ NÃO automático — necessário para {chave} nas mensagens de constraint
    @Bean
    public LocalValidatorFactoryBean validator(MessageSource messageSource) {
        var factory = new LocalValidatorFactoryBean();
        factory.setValidationMessageSource(messageSource);
        return factory;
    }
}
```

```yaml
# application.yml — MessageSourceAutoConfiguration
spring:
  messages:
    basename: messages             # ✅ Default: messages
    encoding: UTF-8               # ✅ Default: UTF-8
    cache-duration: 3600s         # ✅ Default: sem cache
    use-code-as-default-message: false
    fallback-to-system-locale: true  # tenta locale do SO se não encontrar o arquivo
```

### 14.3 Arquivos de Mensagens

```
src/main/resources/
├── messages.properties           ← fallback (pt-BR, idioma padrão)
├── messages_pt_BR.properties     ← português do Brasil
├── messages_en.properties        ← inglês
├── messages_es.properties        ← espanhol (opcional)
└── validation-messages.properties← mensagens de constraint (todas as línguas)
```

```properties
# messages_pt_BR.properties
# ─── Títulos e navegação ──────────────────────────────────────────────────────
app.titulo=Minha Aplicação
app.nav.home=Início
app.nav.produtos=Produtos
app.nav.sair=Sair

# ─── Mensagens de feedback ────────────────────────────────────────────────────
produto.criado=Produto "{0}" cadastrado com sucesso!
produto.atualizado=Produto atualizado.
produto.removido=Produto removido.
produto.nao.encontrado=Produto com ID {0} não encontrado.

# ─── Labels de formulário ─────────────────────────────────────────────────────
form.campo.nome=Nome
form.campo.preco=Preço
form.campo.estoque=Estoque em estoque
form.botao.salvar=Salvar
form.botao.cancelar=Cancelar

# ─── Paginação ────────────────────────────────────────────────────────────────
paginacao.anterior=Anterior
paginacao.proximo=Próximo
paginacao.total=Mostrando {0} a {1} de {2} registros
```

```properties
# messages_en.properties
app.titulo=My Application
app.nav.home=Home
app.nav.produtos=Products
app.nav.sair=Sign out

produto.criado=Product "{0}" created successfully!
produto.atualizado=Product updated.
produto.removido=Product removed.
produto.nao.encontrado=Product with ID {0} not found.

form.campo.nome=Name
form.campo.preco=Price
form.campo.estoque=Stock
form.botao.salvar=Save
form.botao.cancelar=Cancel

paginacao.anterior=Previous
paginacao.proximo=Next
paginacao.total=Showing {0} to {1} of {2} records
```

```properties
# validation-messages.properties (sem sufixo de locale — único arquivo para todas as línguas
# OU criar validation-messages_pt_BR.properties e validation-messages_en.properties)

# Convenção Jakarta Bean Validation: ConstraintName.objectName.fieldName
NotBlank.produtoRequest.nome=Nome do produto é obrigatório
Size.produtoRequest.nome=Nome deve ter entre {min} e {max} caracteres

# Fallback por tipo de constraint
NotBlank=Campo obrigatório
NotNull=Campo obrigatório
Size=Deve ter entre {min} e {max} caracteres
Min=Valor mínimo: {value}
Max=Valor máximo: {value}
Email=E-mail inválido
Positive=Deve ser um número positivo
DecimalMin=Valor mínimo: {value}

# Constraints customizadas
br.com.app.validation.cpf.invalido=CPF inválido
br.com.app.validation.email.unico=E-mail já cadastrado
```

### 14.4 i18n em Controllers REST

```java
@RestController
@RequestMapping("/api/v1/produtos")
public class ProdutoController {

    private final MessageSource messageSource;

    // ─── Usando MessageSource diretamente ─────────────────────────────────────
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void excluir(@PathVariable Long id, Locale locale) {
        produtoService.excluir(id);
        // locale é injetado pelo LocaleResolver — sem acoplamento ao request
    }

    // ─── Mensagem localizada em ProblemDetail ─────────────────────────────────
    @GetMapping("/{id}")
    public ResponseEntity<ProdutoResponse> buscar(
            @PathVariable Long id,
            Locale locale) {

        return produtoService.buscarPorId(id)
                .map(ResponseEntity::ok)
                .orElseThrow(() -> {
                    String msg = messageSource.getMessage(
                            "produto.nao.encontrado",
                            new Object[]{id},
                            locale);
                    return new RecursoNaoEncontradoException(msg);
                });
    }
}
```

```java
// ─── @ControllerAdvice localizando mensagens de erro ─────────────────────────
@RestControllerAdvice
public class GlobalExceptionHandler {

    private final MessageSource messageSource;

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ProblemDetail handleNotFound(
            RecursoNaoEncontradoException ex,
            Locale locale) {

        // A mensagem já vem localizada da exceção OU buscamos aqui
        var pd = ProblemDetail.forStatusAndDetail(
                HttpStatus.NOT_FOUND, ex.getMessage());
        pd.setTitle(messageSource.getMessage(
                "error.not.found.title", null, "Not Found", locale));
        return pd;
    }
}
```

### 14.5 i18n em Templates Thymeleaf

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/base}">
<body>
<section layout:fragment="content">

<!-- ─── #{chave} — resolve mensagem do MessageSource para o locale atual ─────
     Equivalente a messageSource.getMessage("produto.criado", null, locale)     -->
<h1 th:text="#{app.nav.produtos}">Produtos</h1>

<!-- ─── #{chave(param1, param2)} — mensagem com parâmetros ──────────────────
     Equivale ao {0}, {1} nos arquivos .properties                              -->
<p th:text="#{paginacao.total(${page.number * page.size + 1},
                               ${page.number * page.size + page.numberOfElements},
                               ${page.totalElements})}">
    Mostrando 1 a 20 de 150 registros
</p>

<!-- ─── Mensagem flash localizada ────────────────────────────────────────────
     O controller envia a chave (não o texto) como flash attribute             -->
<div th:if="${mensagemChave}" class="alert alert-success">
    <!-- Resolve a chave com parâmetros opcionais -->
    <span th:text="${mensagemArgs != null}
                    ? #{__${mensagemChave}__(__${mensagemArgs}__)}
                    : #{__${mensagemChave}__}">
    </span>
</div>

<!-- ─── Labels de formulário com fallback ────────────────────────────────────
     #{chave,default='texto'} — exibe o default se a chave não existir         -->
<label th:text="#{form.campo.nome,default='Nome'}">Nome</label>

<!-- ─── Seletor de idioma ────────────────────────────────────────────────────
     LocaleChangeInterceptor intercepta o parâmetro lang e altera o locale     -->
<div class="dropdown">
    <button class="btn btn-sm btn-outline-secondary dropdown-toggle">
        Idioma
    </button>
    <ul class="dropdown-menu">
        <li><a class="dropdown-item" th:href="@{/(lang=pt-BR)}">🇧🇷 Português</a></li>
        <li><a class="dropdown-item" th:href="@{/(lang=en)}">🇺🇸 English</a></li>
        <li><a class="dropdown-item" th:href="@{/(lang=es)}">🇪🇸 Español</a></li>
    </ul>
</div>

<!-- ─── Formatação de data/número com o locale atual ─────────────────────────
     Thymeleaf usa automaticamente o locale resolvido pelo LocaleResolver       -->
<td th:text="${#temporals.format(produto.criadoEm, 'dd/MM/yyyy HH:mm')}"></td>
<td th:text="${#numbers.formatDecimal(produto.preco, 1, 'POINT', 2, 'COMMA')}"></td>

<!-- ─── Acesso programático ao locale atual ──────────────────────────────────
     #locale é o objeto java.util.Locale resolvido para a requisição atual      -->
<span th:text="${#locale.language}">pt</span>
<span th:text="${#locale.country}">BR</span>
<span th:text="${#locale}">pt_BR</span>

</section>
</body>
</html>
```

### 14.6 Controller SSR — enviando chave em vez de texto

```java
// Padrão recomendado para SSR: o controller envia CHAVES, o template resolve
@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    @PostMapping
    public String salvar(@ModelAttribute @Valid ProdutoForm form,
                         BindingResult binding,
                         RedirectAttributes redirectAttrs) {
        if (binding.hasErrors()) return "produtos/formulario";

        var produto = produtoService.criar(form);

        // Envia a CHAVE da mensagem e os argumentos separados
        // O template Thymeleaf resolve com o locale do usuário
        redirectAttrs.addFlashAttribute("mensagemChave", "produto.criado");
        redirectAttrs.addFlashAttribute("mensagemArgs", produto.getNome());

        return "redirect:/produtos";
    }
}
```

### 14.7 i18n em Respostas JSON — `MessageSourceAccessor`

```java
/**
 * Wrapper conveniente sobre MessageSource — atalho para evitar
 * passar Locale explicitamente em todo lugar.
 * Usa o Locale do LocaleContextHolder (thread-local do Spring MVC).
 */
@Configuration
public class I18nConfig {

    @Bean
    public MessageSourceAccessor messageSourceAccessor(MessageSource messageSource) {
        return new MessageSourceAccessor(messageSource, new Locale("pt", "BR"));
    }
}

// Uso em services e components sem precisar injetar Locale
@Service
public class NotificacaoService {

    private final MessageSourceAccessor messages;

    public String getTextoEmail(String chave, Object... args) {
        // Usa o Locale do LocaleContextHolder — respeita o locale da requisição
        return messages.getMessage(chave, args);
    }

    public void enviarBoasVindas(Usuario usuario) {
        String assunto = messages.getMessage("email.boas.vindas.assunto",
                new Object[]{usuario.getNome()});
        String corpo   = messages.getMessage("email.boas.vindas.corpo",
                new Object[]{usuario.getNome()});
        emailService.enviar(usuario.getEmail(), assunto, corpo);
    }
}
```

### 14.8 Timezone — Integração com i18n

```java
// LocaleContextHolder carrega Locale E TimeZone — útil para formatação
@Component
public class DateTimeFormatter {

    public String formatarParaUsuario(LocalDateTime dateTime) {
        TimeZone tz = LocaleContextHolder.getTimeZone();
        ZoneId zoneId = tz != null ? tz.toZoneId() : ZoneOffset.UTC;

        DateTimeFormatter fmt = DateTimeFormatter
                .ofLocalizedDateTime(FormatStyle.MEDIUM)
                .withLocale(LocaleContextHolder.getLocale())
                .withZone(zoneId);

        return fmt.format(dateTime.atZone(ZoneId.systemDefault())
                .withZoneSameInstant(zoneId));
    }
}
```


### 14.9 `LocaleContextHolder` — acesso ao Locale fora do Controller

`LocaleContextHolder` é um utilitário estático do Spring que armazena o **Locale e o
TimeZone da requisição atual** em uma variável `ThreadLocal`. O `DispatcherServlet`
popula esse holder automaticamente ao início de cada requisição — usando o
`LocaleResolver` configurado — e o limpa ao final.

```
HTTP Request
    │
    ▼
DispatcherServlet
    ├─ chama LocaleResolver.resolveLocale(request)
    ├─ armazena resultado em LocaleContextHolder (ThreadLocal)
    │
    ▼                              ▼                         ▼
@Controller                   @Service                  @Component
  Locale locale               LocaleContextHolder        MessageSourceAccessor
  (injetado pelo MVC)         .getLocale()               (usa o holder internamente)
```

#### Métodos principais

| Método | Descrição |
|--------|-----------|
| `getLocale()` | Retorna o `Locale` da thread atual (nunca `null` — usa `Locale.getDefault()` como fallback) |
| `getTimeZone()` | Retorna o `TimeZone` da thread atual (pode ser `null` se não configurado) |
| `setLocale(Locale)` | Define o Locale para a thread atual (útil em testes e tarefas assíncronas) |
| `setTimeZone(TimeZone)` | Define o TimeZone para a thread atual |
| `getLocaleContext()` | Retorna o `LocaleContext` completo (contém Locale e TimeZone juntos) |
| `setLocaleContext(LocaleContext)` | Define `LocaleContext` na thread atual |
| `resetLocaleContext()` | Remove o contexto da thread (evita memory leak fora do MVC) |

```java
// Imports
import org.springframework.context.i18n.LocaleContextHolder;
import java.util.Locale;
import java.util.TimeZone;
```

#### Uso na camada de serviço

O principal benefício é acessar o Locale **sem precisar propagá-lo por parâmetro**
através de todas as camadas.

```java
// ❌ Abordagem sem LocaleContextHolder — polui a assinatura dos métodos
@Service
public class ProdutoService {
    public String getNomeProduto(Long id, Locale locale) { ... }
    public void validarEstoque(Long id, int qtd, Locale locale) { ... }
    public void enviarConfirmacao(Pedido pedido, Locale locale) { ... }
}

// ✅ Com LocaleContextHolder — Locale transparente na camada de serviço
@Service
public class ProdutoService {

    private final MessageSource messageSource;

    public String getMensagemEstoqueInsuficiente(int disponivel) {
        // Locale resolvido automaticamente — sem parâmetro extra
        Locale locale = LocaleContextHolder.getLocale();
        return messageSource.getMessage(
            "estoque.insuficiente",
            new Object[]{disponivel},
            locale);
    }
}
```

#### Propagação manual em código assíncrono

> **Atenção:** `ThreadLocal` não é propagado automaticamente para novas threads.
> Em métodos `@Async`, `CompletableFuture`, `ExecutorService` e similares,
> é preciso capturar e restaurar o contexto manualmente.

```java
@Service
public class RelatorioService {

    // ─── @Async — contexto NÃO é propagado automaticamente ───────────────────
    @Async
    public CompletableFuture<String> gerarRelatorioAsync(Long id) {
        // ⚠️ LocaleContextHolder.getLocale() aqui retornaria o locale padrão do SO,
        // NÃO o locale da requisição que disparou o @Async.
        Locale locale = LocaleContextHolder.getLocale(); // ← ERRADO em @Async
        ...
    }

    // ─── Solução: capturar antes de delegar à outra thread ────────────────────
    public CompletableFuture<String> gerarRelatorioComLocale(Long id) {
        // 1. Captura o contexto NA thread da requisição
        LocaleContext contexto = LocaleContextHolder.getLocaleContext();
        Locale localeAtual = LocaleContextHolder.getLocale();

        return CompletableFuture.supplyAsync(() -> {
            // 2. Restaura o contexto NA nova thread
            LocaleContextHolder.setLocaleContext(contexto);
            try {
                return processarRelatorio(id, localeAtual);
            } finally {
                // 3. Limpa para evitar memory leak no pool de threads
                LocaleContextHolder.resetLocaleContext();
            }
        });
    }
}
```

#### Configurando em testes

```java
@SpringBootTest
class ProdutoServiceTest {

    @Autowired
    private ProdutoService produtoService;

    @Test
    void deveMostrarMensagemEmIngles() {
        // Define o locale para este teste
        LocaleContextHolder.setLocale(Locale.ENGLISH);
        try {
            String msg = produtoService.getMensagemEstoqueInsuficiente(0);
            assertThat(msg).contains("out of stock");
        } finally {
            // Sempre restaurar após o teste
            LocaleContextHolder.resetLocaleContext();
        }
    }

    @Test
    void deveMostrarMensagemEmPortugues() {
        LocaleContextHolder.setLocale(new Locale("pt", "BR"));
        try {
            String msg = produtoService.getMensagemEstoqueInsuficiente(0);
            assertThat(msg).contains("sem estoque");
        } finally {
            LocaleContextHolder.resetLocaleContext();
        }
    }
}
```

#### Locale e TimeZone juntos — `LocaleContext` e `TimeZoneAwareLocaleContext`

```java
@Component
public class ContextoUsuarioHelper {

    /**
     * Retorna um resumo do contexto de localização da thread atual.
     * Útil para logging e diagnóstico.
     */
    public String descreverContexto() {
        Locale locale = LocaleContextHolder.getLocale();
        TimeZone tz   = LocaleContextHolder.getTimeZone();

        return String.format("Locale: %s | TimeZone: %s",
            locale,
            tz != null ? tz.getID() : "não definido (usando UTC)");
    }

    /**
     * Cria um LocaleContext com Locale e TimeZone para uso em threads assíncronas.
     * SimpleTimeZoneAwareLocaleContext implementa TimeZoneAwareLocaleContext.
     */
    public LocaleContext capturarContextoAtual() {
        return new SimpleTimeZoneAwareLocaleContext(
            LocaleContextHolder.getLocale(),
            LocaleContextHolder.getTimeZone()
        );
    }
}
```

#### Quando usar `LocaleContextHolder` vs injetar `Locale` no método

| Situação | Recomendação |
|----------|-------------|
| Controller (REST ou MVC) | Injetar `Locale locale` no método — mais explícito e testável |
| Camada de serviço (`@Service`) | `LocaleContextHolder.getLocale()` — evita poluir a API |
| Componente de infraestrutura (`@Component`) | `LocaleContextHolder.getLocale()` ou `MessageSourceAccessor` |
| `@Async` / `CompletableFuture` | Capturar antes, restaurar dentro da nova thread |
| `@Scheduled` | Definir locale explicitamente — não há requisição HTTP |
| Testes unitários | `LocaleContextHolder.setLocale(...)` + `resetLocaleContext()` no `@AfterEach` |


---


## 15. Customização do `ErrorController`

O Spring Boot registra automaticamente um `BasicErrorController` que serve o
endpoint `/error` — ponto de chegada de todos os erros não tratados por um
`@ControllerAdvice` (ex.: 404 gerado antes do `DispatcherServlet`, exceções em
filtros, erros de Tomcat). Esta seção cobre como personalizar esse comportamento.

### 15.1 Como o Fluxo de Erro Funciona

```mermaid
flowchart LR
    REQ["Request"] --> F["Filters / Security"]
    F -->|exceção no filtro| EC["/error<br>BasicErrorController"]
    F --> DS["DispatcherServlet"]
    DS -->|404 — handler não encontrado| EC
    DS --> CTRL["@Controller / @RestController"]
    CTRL -->|exceção não mapeada| CA["@ControllerAdvice<br>GlobalExceptionHandler"]
    CA -->|exceção não mapeada| EC
    EC -->|browser| VIEW["Whitelabel Error Page<br>ou template error/*.html"]
    EC -->|cliente REST| JSON["JSON ProblemDetail"]
```

> A regra prática é: `@ControllerAdvice` para a esmagadora maioria dos casos;
> `ErrorController` apenas para erros que **escapam** do `DispatcherServlet`.

### 15.2 Customizando o `BasicErrorController`

#### Abordagem 1 — `ErrorAttributes` customizado

A forma mais simples: substitui apenas o **conteúdo** da resposta de erro sem
reimplementar o controller.

```java
/**
 * Substitui o DefaultErrorAttributes do Spring Boot.
 * Controla quais campos aparecem na resposta JSON de /error
 * e adiciona informações customizadas.
 */
@Component
public class AppErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest,
                                                   ErrorAttributeOptions options) {
        // Começa com os atributos padrão (timestamp, status, error, path...)
        Map<String, Object> attrs = super.getErrorAttributes(webRequest, options);

        // Remove campos verbosos que não devem ser expostos ao cliente
        attrs.remove("exception");   // classe da exceção interna
        attrs.remove("trace");       // stack trace

        // Adiciona campos customizados
        attrs.put("api_version", "v1");
        attrs.put("docs", "https://api.empresa.com.br/docs/erros");

        // Recupera a exceção original para enriquecer a resposta
        Throwable ex = getError(webRequest);
        if (ex instanceof RecursoNaoEncontradoException e) {
            attrs.put("recurso", e.getRecurso());
            attrs.put("identificador", e.getId());
        }

        return attrs;
    }
}
```

#### Abordagem 2 — `ErrorController` completo

Reimplementa todo o endpoint `/error`, separando a resposta para clientes REST
e para o browser (SSR):

```java
/**
 * Substitui o BasicErrorController do Spring Boot.
 *
 * Registrar este bean faz o Spring Boot desabilitar o BasicErrorController
 * automaticamente — não é necessário excluir nenhuma auto-configuração.
 */
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class AppErrorController implements ErrorController {

    private final ErrorAttributes errorAttributes;

    public AppErrorController(ErrorAttributes errorAttributes) {
        this.errorAttributes = errorAttributes;
    }

    // ─── Resposta JSON para clientes REST ─────────────────────────────────────
    @RequestMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public ResponseEntity<Map<String, Object>> errorJson(HttpServletRequest request) {
        var attrs   = getErrorAttributes(request);
        var status  = HttpStatus.valueOf((int) attrs.getOrDefault("status", 500));

        // Formata no padrão RFC 9457 (ProblemDetail)
        var body = new LinkedHashMap<String, Object>();
        body.put("type",     "https://api.empresa.com.br/erros/" + status.value());
        body.put("title",    status.getReasonPhrase());
        body.put("status",   status.value());
        body.put("detail",   attrs.getOrDefault("message", "Erro interno"));
        body.put("instance", attrs.get("path"));
        body.put("timestamp", Instant.now());

        return ResponseEntity.status(status).body(body);
    }

    // ─── Resposta HTML para o browser (SSR) ───────────────────────────────────
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request) {
        var attrs  = getErrorAttributes(request);
        var status = HttpStatus.valueOf((int) attrs.getOrDefault("status", 500));

        // Tenta view específica por status (error/404.html, error/500.html)
        // com fallback para error/error.html genérico
        String viewName = switch (status) {
            case NOT_FOUND            -> "error/404";
            case FORBIDDEN            -> "error/403";
            case INTERNAL_SERVER_ERROR-> "error/500";
            default                   -> "error/error";
        };

        var mav = new ModelAndView(viewName);
        mav.setStatus(status);
        mav.addObject("status",  status.value());
        mav.addObject("message", attrs.getOrDefault("message", status.getReasonPhrase()));
        mav.addObject("path",    attrs.get("path"));
        return mav;
    }

    private Map<String, Object> getErrorAttributes(HttpServletRequest request) {
        var webRequest = new ServletWebRequest(request);
        return errorAttributes.getErrorAttributes(webRequest,
                ErrorAttributeOptions.of(
                        ErrorAttributeOptions.Include.MESSAGE,
                        ErrorAttributeOptions.Include.BINDING_ERRORS
                ));
    }
}
```

### 15.3 Templates de Erro Thymeleaf

O Spring Boot resolve automaticamente templates em `templates/error/` pelo
código de status — sem nenhuma configuração adicional.

```
src/main/resources/templates/
└── error/
    ├── 400.html   ← Bad Request
    ├── 403.html   ← Forbidden
    ├── 404.html   ← Not Found
    ├── 500.html   ← Internal Server Error
    └── error.html ← fallback genérico (qualquer outro status)
```

```html
<!-- templates/error/404.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/base}">
<body>
<section layout:fragment="content" class="text-center py-5">
    <h1 class="display-1 fw-bold text-muted">404</h1>
    <h2 class="mb-3">Página não encontrada</h2>
    <p class="text-muted mb-4">
        O endereço <code th:text="${path}"></code> não existe ou foi movido.
    </p>
    <a th:href="@{/}" class="btn btn-primary">Voltar ao início</a>
</section>
</body>
</html>
```

### 15.4 Configuração via `application.yml`

```yaml
server:
  error:
    path: /error                  # ✅ Default: /error
    include-message: always       # ✅ Default: never — expõe getMessage() na resposta
    include-binding-errors: always# ✅ Default: never — expõe erros de validação
    include-stacktrace: never     # ✅ Default: never — NUNCA expor em produção
    include-exception: false      # ✅ Default: false — oculta classe da exceção

    # Whitelabel: página padrão do Spring Boot quando não há template de erro
    whitelabel:
      enabled: false              # ✅ Default: true — desabilitar quando usar templates próprios
```

---



## 16. `@ResponseStatus` em Classes de Exceção

`@ResponseStatus` aplicado diretamente a uma classe de exceção instrui o Spring
MVC a retornar um HTTP status específico sempre que essa exceção for lançada —
sem necessidade de um `@ExceptionHandler` dedicado.

### 16.1 Uso Básico

```java
// ─── Exceção com status fixo ──────────────────────────────────────────────────
@ResponseStatus(HttpStatus.NOT_FOUND)
public class RecursoNaoEncontradoException extends RuntimeException {
    public RecursoNaoEncontradoException(String mensagem) {
        super(mensagem);
    }
}

// ─── Exceção com código de erro personalizado (reason) ────────────────────────
//
// reason: texto fixo que substitui a mensagem da exceção no body.
// Use apenas quando a mensagem de erro pode ser exposta ao cliente.
@ResponseStatus(
    value  = HttpStatus.CONFLICT,
    reason = "Registro duplicado"
)
public class DuplicidadeException extends RuntimeException {
    public DuplicidadeException(String entidade, Object chave) {
        super("Já existe um(a) " + entidade + " com a chave: " + chave);
    }
}

// ─── Uso no controller — sem nenhum try/catch ─────────────────────────────────
@GetMapping("/{id}")
public ProdutoResponse buscar(@PathVariable Long id) {
    return produtoService.buscarPorId(id)
            .orElseThrow(() ->
                new RecursoNaoEncontradoException("Produto " + id + " não encontrado"));
            // → Spring retorna automaticamente 404
}

@PostMapping
public ResponseEntity<ProdutoResponse> criar(@RequestBody @Valid ProdutoRequest req) {
    if (produtoService.skuJaExiste(req.sku())) {
        throw new DuplicidadeException("Produto", req.sku());
        // → Spring retorna automaticamente 409 Conflict com body "Registro duplicado"
    }
    // ...
}
```

### 16.2 `@ResponseStatus` vs `@ExceptionHandler` — Quando Usar Cada Um

```java
// ─── @ResponseStatus: adequado para exceções simples ─────────────────────────
//
// ✅ Use quando:
//   - A resposta de erro é apenas o status HTTP + mensagem simples
//   - Não há lógica de tratamento (log, enriquecimento, ProblemDetail detalhado)
//   - A exceção é de domínio e carrega a mensagem diretamente

@ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
public class RegraDeNegocioException extends RuntimeException {
    public RegraDeNegocioException(String mensagem) { super(mensagem); }
}

// ─── @ExceptionHandler: necessário para respostas ricas ──────────────────────
//
// ✅ Use quando:
//   - Precisa de ProblemDetail com campos extras (violations, links, correlationId)
//   - Precisa logar a exceção
//   - Precisa de lógica condicional na resposta (ex: detalhe diferente por ambiente)
//   - Múltiplas exceções mapeadas para o mesmo formato de resposta

@ExceptionHandler(ConstraintViolationException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public ProblemDetail handleConstraintViolation(ConstraintViolationException ex) {
    var pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Dados inválidos");
    // enriquece com a lista de violações...
    return pd;
}
```

### 16.3 Precedência com `@ControllerAdvice`

Quando uma exceção tem `@ResponseStatus` **e** existe um `@ExceptionHandler`
compatível no `@ControllerAdvice`, o **`@ExceptionHandler` vence** — a anotação
`@ResponseStatus` na classe da exceção é ignorada.

```java
// Esta exceção tem @ResponseStatus(404) na classe...
@ResponseStatus(HttpStatus.NOT_FOUND)
public class RecursoNaoEncontradoException extends RuntimeException { /* ... */ }

// ...mas este handler vence, pois @ExceptionHandler tem precedência:
@ExceptionHandler(RecursoNaoEncontradoException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)           // ainda precisa declarar aqui
public ProblemDetail handle(RecursoNaoEncontradoException ex, HttpServletRequest req) {
    var pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    pd.setInstance(URI.create(req.getRequestURI()));
    return pd;                                   // retorna 404 com ProblemDetail
}
```

| Cenário | Quem controla a resposta |
|---|---|
| Exceção com `@ResponseStatus`, sem `@ExceptionHandler` | `@ResponseStatus` na classe |
| Exceção com `@ResponseStatus` + `@ExceptionHandler` compatível | `@ExceptionHandler` (vence) |
| Exceção sem `@ResponseStatus`, sem `@ExceptionHandler` | Spring MVC → 500 |
| Exceção sem `@ResponseStatus` + `@ExceptionHandler` | `@ExceptionHandler` |

---



## 17. `MultiValueMap` e Form Data

### 17.1 `MultiValueMap` — Múltiplos Valores por Chave

`MultiValueMap<K, V>` é uma extensão de `Map` do Spring que associa **uma ou mais
valores** a cada chave. É o tipo usado internamente pelo MVC para representar
parâmetros de query, headers e form data — onde um mesmo campo pode aparecer
múltiplas vezes (ex.: `?tag=java&tag=spring`).

```java
@RestController
@RequestMapping("/api/v1/exemplos")
public class MultiValueMapController {

    // ─── @RequestParam com lista — forma mais comum ───────────────────────────
    // GET /api/v1/exemplos/busca?tag=java&tag=spring&tag=mvc
    @GetMapping("/busca")
    public List<ProdutoResponse> buscar(
            @RequestParam List<String> tag,          // lista de valores do param "tag"
            @RequestParam(required = false) String q) {
        return produtoService.buscarPorTags(tag, q);
    }

    // ─── MultiValueMap completo — quando os parâmetros são dinâmicos ──────────
    // Todos os query params em um único mapa — útil para proxy/forwarding
    @GetMapping("/todos-params")
    public Map<String, List<String>> todosParams(
            @RequestParam MultiValueMap<String, String> params) {
        // params.get("tag")      → ["java", "spring"]
        // params.getFirst("tag") → "java"
        // params.toSingleValueMap() → Map<String, String> (pega o primeiro de cada)
        return params;
    }

    // ─── @RequestHeader com MultiValueMap ─────────────────────────────────────
    @GetMapping("/headers")
    public Map<String, List<String>> headers(
            @RequestHeader MultiValueMap<String, String> headers) {
        return headers;
    }
}
```

### 17.2 Form Data com Checkboxes (`multi-select`)

O caso de uso mais comum de `MultiValueMap` em SSR é o binding de checkboxes e
selects múltiplos em um form HTML — onde cada checkbox marcado envia o mesmo
nome de campo com valores diferentes.

```java
// ─── Form object com lista para binding de checkboxes ─────────────────────────
public class FiltroProdutoForm {

    // Lista recebe um valor por checkbox marcado
    private List<Long> categoriaIds = new ArrayList<>();

    // Lista de Strings para checkboxes de tags
    private List<String> tags = new ArrayList<>();

    // Integer para select múltiplo de estoque mínimo
    private Integer estoqueMinimo;

    // getters/setters necessários para o binding MVC
    public List<Long> getCategoriaIds()          { return categoriaIds; }
    public void setCategoriaIds(List<Long> ids)  { this.categoriaIds = ids; }
    public List<String> getTags()                { return tags; }
    public void setTags(List<String> tags)       { this.tags = tags; }
    // ...
}

@Controller
@RequestMapping("/produtos")
public class ProdutoMvcController {

    @GetMapping("/filtrar")
    public String exibirFiltro(Model model) {
        model.addAttribute("filtro",     new FiltroProdutoForm());
        model.addAttribute("categorias", categoriaService.listarTodas());
        model.addAttribute("tagsDisponiveis", tagService.listarTodas());
        return "produtos/filtro";
    }

    @PostMapping("/filtrar")
    public String aplicarFiltro(
            @ModelAttribute("filtro") FiltroProdutoForm filtro,
            Model model) {
        // filtro.getCategoriaIds() contém os IDs das checkboxes marcadas
        // filtro.getTags() contém as tags selecionadas
        model.addAttribute("produtos",
                produtoService.filtrar(filtro.getCategoriaIds(), filtro.getTags(),
                                       filtro.getEstoqueMinimo()));
        model.addAttribute("categorias", categoriaService.listarTodas());
        model.addAttribute("tagsDisponiveis", tagService.listarTodas());
        return "produtos/filtro";
    }
}
```

```html
<!-- templates/produtos/filtro.html -->
<form th:action="@{/produtos/filtrar}" th:object="${filtro}" method="post">

    <!-- ─── Checkboxes — cada checkbox tem o mesmo "name" ─────────────────────
         th:field gera name="categoriaIds" e value="${cat.id()}" para cada item.
         O MVC coleta todos os valores marcados na lista categoriaIds do form.  -->
    <fieldset class="mb-3">
        <legend class="fw-semibold">Categorias</legend>
        <div th:each="cat : ${categorias}" class="form-check form-check-inline">
            <input type="checkbox"
                   th:field="*{categoriaIds}"
                   th:value="${cat.id()}"
                   class="form-check-input"
                   th:id="'cat-' + ${cat.id()}">
            <label class="form-check-label"
                   th:for="'cat-' + ${cat.id()}"
                   th:text="${cat.nome()}">Categoria</label>
        </div>
    </fieldset>

    <!-- ─── Checkboxes de Strings ─────────────────────────────────────────────
         Para listas de String, th:field gera value igual ao texto da tag.     -->
    <fieldset class="mb-3">
        <legend class="fw-semibold">Tags</legend>
        <div th:each="tag : ${tagsDisponiveis}" class="form-check form-check-inline">
            <input type="checkbox"
                   th:field="*{tags}"
                   th:value="${tag}"
                   class="form-check-input"
                   th:id="'tag-' + ${tag}">
            <label class="form-check-label"
                   th:for="'tag-' + ${tag}"
                   th:text="${tag}">Tag</label>
        </div>
    </fieldset>

    <!-- ─── Select múltiplo ───────────────────────────────────────────────────
         multiple="true" permite selecionar vários itens com Ctrl/Cmd+clique.
         O binding é idêntico ao dos checkboxes — lista de valores.            -->
    <div class="mb-3">
        <label class="form-label fw-semibold">Selecionar categorias (alternativa)</label>
        <select th:field="*{categoriaIds}" class="form-select" multiple size="4">
            <option th:each="cat : ${categorias}"
                    th:value="${cat.id()}"
                    th:text="${cat.nome()}">
            </option>
        </select>
        <small class="text-muted">Ctrl/Cmd + clique para selecionar múltiplos</small>
    </div>

    <button type="submit" class="btn btn-primary">Filtrar</button>
</form>
```

### 17.3 `LinkedMultiValueMap` — Construção Programática

```java
// Construção manual de MultiValueMap — útil em testes ou ao montar requests
MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
params.add("tag", "java");
params.add("tag", "spring");
params.add("tag", "mvc");
params.set("pagina", "0");       // set substitui todos os valores da chave

// Uso com RestClient / TestRestTemplate
restClient.get()
        .uri(uriBuilder -> uriBuilder
                .path("/api/v1/produtos/busca")
                .queryParams(params)
                .build())
        .retrieve()
        .body(new ParameterizedTypeReference<List<ProdutoResponse>>() {});
```

---



## 18. `RedirectView` e `UrlBasedViewResolver`

### 18.1 `RedirectView` — Redirect Programático com Controle Total

```java
@Controller
@RequestMapping("/legacy")
public class LegacyRedirectController {

    // ─── String de redirect — forma mais simples (preferível na maioria dos casos)
    @GetMapping("/produtos")
    public String redirectSimples() {
        return "redirect:/api/v1/produtos";  // 302 por padrão
    }

    // ─── RedirectView — quando precisar de controle fino ─────────────────────
    @GetMapping("/produto/{id}")
    public RedirectView redirectComControle(@PathVariable Long id) {
        var rv = new RedirectView();

        rv.setUrl("/produtos/" + id);

        // Status code: 301 (permanente) ou 302 (temporário, default)
        rv.setStatusCode(HttpStatus.MOVED_PERMANENTLY);

        // false = preserva o contexto da app no path (recomendado: true)
        rv.setContextRelative(true);

        // false = não expõe os atributos do model como query params
        rv.setExposeModelAttributes(false);

        return rv;
    }

    // ─── RedirectView com query params via Model ──────────────────────────────
    //
    // Atributos adicionados ao Model (com exposeModelAttributes=true, default)
    // são automaticamente anexados como query params na URL de destino
    @PostMapping("/busca-legada")
    public RedirectView redirectComParams(@RequestParam String q, Model model) {
        model.addAttribute("query",  q);        // → /busca?query=valor
        model.addAttribute("origem", "legacy"); // → /busca?query=valor&origem=legacy

        var rv = new RedirectView("/busca");
        rv.setExposeModelAttributes(true);      // default: true
        return rv;
    }

    // ─── ModelAndView com RedirectView ────────────────────────────────────────
    @GetMapping("/painel")
    public ModelAndView redirectMav() {
        var rv  = new RedirectView("/admin/dashboard", true); // contextRelative=true
        var mav = new ModelAndView(rv);
        mav.addObject("source", "legacy_painel");
        return mav;
    }
}
```

### 18.2 `UrlBasedViewResolver` — Configuração Avançada de View Resolution

O Spring Boot auto-configura o `ThymeleafViewResolver`, que tem precedência sobre
o `UrlBasedViewResolver`. Este é relevante quando se usa FreeMarker, Mustache,
JSP ou uma engine customizada onde o resolver precisa ser configurado manualmente.

```java
@Configuration
public class ViewResolverConfig {

    // ─── Encadeamento de resolvers por ordem de precedência ───────────────────
    //
    // O Spring MVC tenta cada resolver em ordem (menor order = maior prioridade).
    // O primeiro que retornar uma View não-nula é usado.

    // 1. Thymeleaf: order=1 (auto-configurado pelo Boot com mais alta prioridade)
    //    Resolve: qualquer nome sem prefixo especial

    // 2. Redirects e Forwards: sempre resolvidos antes de qualquer ViewResolver
    //    "redirect:/rota"  → RedirectView (302)
    //    "redirect:301:/rota" → RedirectView (301) — Spring 6+
    //    "forward:/rota"   → InternalResourceView (forward interno)

    // 3. Resolver customizado para relatórios PDF (exemplo)
    @Bean
    public ViewResolver pdfViewResolver() {
        var resolver = new UrlBasedViewResolver();
        resolver.setOrder(2);                     // após Thymeleaf
        resolver.setViewClass(PdfView.class);     // view customizada
        resolver.setPrefix("classpath:/relatorios/");
        resolver.setSuffix(".jrxml");             // JasperReports, por exemplo
        // Só resolve nomes com prefixo "pdf:" no controller:
        // return "pdf:relatorio-vendas";
        return resolver;
    }

    // ─── InternalResourceViewResolver (JSP — legado) ─────────────────────────
    // Necessário apenas em projetos que ainda usam JSP
    // @Bean
    // public InternalResourceViewResolver jspViewResolver() {
    //     var resolver = new InternalResourceViewResolver();
    //     resolver.setOrder(3);
    //     resolver.setPrefix("/WEB-INF/views/");
    //     resolver.setSuffix(".jsp");
    //     return resolver;
    // }
}
```

### 18.3 Redirect 301 Permanente — SEO e Mudança de URL

```java
@Controller
public class SeoRedirectController {

    // ─── Redirect 301 via String (Spring 6+) ─────────────────────────────────
    @GetMapping("/blog/{slug}")
    public String blogPost(@PathVariable String slug) {
        return "redirect:301:/artigos/" + slug;   // sintaxe Spring 6+
    }

    // ─── Redirect 301 via RedirectView (Spring 5 e anterior) ─────────────────
    @GetMapping("/noticias/{id}")
    public RedirectView noticiaLegada(@PathVariable Long id) {
        var rv = new RedirectView("/conteudo/noticias/" + id);
        rv.setStatusCode(HttpStatus.MOVED_PERMANENTLY);
        return rv;
    }

    // ─── Redirects em massa via addViewControllers (WebMvcConfigurer) ─────────
    // Para redirects estáticos sem lógica, prefira addViewControllers (seção 2.2)
    // em vez de um controller dedicado — mais performático e sem instância de bean
}
```

### 18.4 Forward Interno — Compartilhamento de Handlers

```java
@Controller
@RequestMapping("/compatibilidade")
public class ForwardController {

    // Forward: transfere a requisição para outro handler INTERNAMENTE
    // O browser não sabe que houve um forward — a URL não muda
    // Diferente do redirect, o request original é preservado (incluindo body e attrs)
    @GetMapping("/produto-antigo/{codigo}")
    public String forwardParaNovo(@PathVariable String codigo,
                                   HttpServletRequest request) {
        // Preserva o código como atributo para o handler destino
        request.setAttribute("codigoLegado", codigo);
        return "forward:/api/v1/produtos/por-codigo/" + codigo;
    }

    // Forward com prefixo explícito — equivalente ao return "forward:..."
    @GetMapping("/rota-alternativa")
    public ModelAndView forwardMav() {
        return new ModelAndView("forward:/rota-principal");
    }
}
```
---



## 19. Testes

### 19.1 Visão Geral — Pirâmide de Testes no Spring MVC

```mermaid
graph TB
    A["@SpringBootTest<br/>Integração completa<br/>contexto real + banco (Testcontainers)<br/>Mais lento / mais fidelidade"]
    B["@WebMvcTest<br/>Slice test — só a camada web<br/>MockMvc / RestTestClient<br/>Rápido / sem banco"]
    C["@DataJpaTest / @JdbcTest<br/>Slice test — só a camada de dados<br/>Banco em memória ou Testcontainers"]
    D["Unitário puro<br/>new Service(mockRepo)<br/>Sem Spring / instantâneo"]
    D --> C --> B --> A
```

**Dependências de teste — Spring Boot 3.x vs Spring Boot 4.x:**

> **Spring Boot 4 / Spring Framework 7 — mudanças importantes nas dependências de teste:**
>
> - **JUnit 6 (Jupiter 6):** o Spring Boot 4 usa JUnit 6 (`org.junit.api.*`).
>   O pacote mudou de `org.junit.jupiter.api` para `org.junit.api` — todos os
>   imports precisam ser atualizados.
> - **`spring-boot-starter-webmvc` explícito:** no Spring Boot 4, o starter web
>   foi renomeado. Para testes que precisam da stack MVC completa, declare
>   `spring-boot-starter-webmvc` explicitamente (ver seção 1).
> - **`RestTestClient` nativo:** o `RestTestClient` passou a ser incluído
>   automaticamente pelo `spring-boot-starter-test` no Boot 4 — sem dependência
>   adicional.

```xml
<!-- ─── Spring Boot 3.x ──────────────────────────────────────────────────── -->

<!-- Inclui: JUnit 5 (org.junit.jupiter.api.*), Mockito, AssertJ,
             Hamcrest, JsonPath, Spring Test, Spring Boot Test, MockMvc -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- ─── Spring Boot 4.x — diferenças relevantes ─────────────────────────── -->

<!-- Inclui: JUnit 6 (org.junit.api.*), Mockito, AssertJ,
             Spring Test, Spring Boot Test, RestTestClient (nativo)  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- Boot 4: declarar explicitamente para testes @WebMvcTest e @SpringBootTest
     que precisam da stack MVC (DispatcherServlet, MessageConverters, etc.) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
    <scope>test</scope>
</dependency>

<!-- ─── Comuns a Boot 3 e Boot 4 ─────────────────────────────────────────── -->

<!-- Spring Security Test — @WithMockUser, @WithUserDetails, SecurityMockMvcRequestPostProcessors -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- Testcontainers — banco real em container para testes de integração -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

**JUnit 5 vs JUnit 6 — mudança de pacote:**

```java
// ─── JUnit 5 (Spring Boot 3.x) ────────────────────────────────────────────────
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

// ─── JUnit 6 (Spring Boot 4.x) — pacote raiz mudou de jupiter para api ────────
import org.junit.api.Test;
import org.junit.api.BeforeEach;
import org.junit.api.DisplayName;
import org.junit.api.extension.ExtendWith;
import org.junit.api.params.ParameterizedTest;
import org.junit.api.params.provider.ValueSource;

// As anotações e comportamentos são os mesmos — apenas o pacote mudou.
// No IntelliJ IDEA: use "Migrate Packages" ou busca/substituição global:
//   org.junit.jupiter.api  →  org.junit.api
```

---

### 19.2 Teste Unitário — Service sem Spring

O teste mais rápido: instancia a classe diretamente, injeta mocks via construtor.
Não carrega nenhum contexto Spring.

#### Ciclo de vida dos métodos de setup e teardown

```java
// JUnit 6 (Boot 4) — pacote org.junit.api.*
// JUnit 5 (Boot 3) — pacote org.junit.jupiter.api.*
//
// As anotações abaixo funcionam igualmente em ambas as versões;
// apenas o pacote de import muda.
//
// ─── Ordem de execução por teste ──────────────────────────────────────────────
//
//  @BeforeAll   → uma vez antes de TODOS os testes da classe
//  @BeforeEach  → antes de CADA teste
//     @Test     → o próprio teste
//  @AfterEach   → após CADA teste
//  @AfterAll    → uma vez após TODOS os testes da classe
//
// @BeforeAll e @AfterAll precisam ser static (por padrão) porque são chamados
// antes de qualquer instância ser criada. Podem ser não-static com
// @TestInstance(Lifecycle.PER_CLASS) — uma única instância para toda a classe.

@ExtendWith(MockitoExtension.class)
@DisplayName("ProdutoService — ciclo de vida completo")
class ProdutoServiceLifecycleTest {

    // ─── @BeforeAll — executado UMA VEZ antes de todos os testes ─────────────
    // Usado para recursos caros de inicializar que podem ser compartilhados:
    // conexões, servidores externos, dados de referência read-only.
    // Deve ser static (a menos que @TestInstance(PER_CLASS) seja usado).
    @BeforeAll
    static void configurarAmbiente() {
        // Exemplos de uso real:
        //   - Iniciar um servidor mock de e-mail (GreenMail)
        //   - Carregar fixtures de dados estáticos de arquivos
        //   - Configurar propriedades de sistema necessárias ao teste
        System.setProperty("app.test.modo", "unitario");
    }

    // ─── @BeforeEach — executado antes de CADA teste ──────────────────────────
    // Reinicia o estado que pode ser "poluído" por um teste anterior.
    // Cada teste parte de um estado limpo e previsível.
    @Mock private ProdutoRepository produtoRepository;
    @Mock private CategoriaService  categoriaService;
    @InjectMocks private ProdutoService produtoService;

    @BeforeEach
    void prepararCenario() {
        // Com @ExtendWith(MockitoExtension) os mocks já são reiniciados automaticamente
        // a cada teste — @BeforeEach é útil para preparar dados de teste reutilizáveis
        // ou configurar comportamentos padrão comuns a vários testes.
        when(categoriaService.buscarPorId(1L))
                .thenReturn(new Categoria(1L, "Informática"));
    }

    // ─── @AfterEach — executado após CADA teste ───────────────────────────────
    // Limpa recursos alocados no @BeforeEach ou durante o próprio teste:
    // arquivos temporários, conexões abertas, estados estáticos modificados.
    @AfterEach
    void limparCenario() {
        // Exemplos de uso real:
        //   - Deletar arquivos temporários criados pelo teste
        //   - Resetar contadores estáticos
        //   - Fechar streams ou conexões abertas no @BeforeEach
        // Com Mockito puro (@ExtendWith) os mocks já são resetados automaticamente —
        // @AfterEach só é necessário para recursos externos.
    }

    // ─── @AfterAll — executado UMA VEZ após todos os testes ──────────────────
    // Libera recursos inicializados no @BeforeAll.
    // Deve ser static (mesma restrição do @BeforeAll).
    @AfterAll
    static void liberarAmbiente() {
        // Exemplos de uso real:
        //   - Parar um servidor mock de e-mail iniciado no @BeforeAll
        //   - Remover propriedades de sistema configuradas no @BeforeAll
        //   - Liberar conexões de banco compartilhadas (em testes que não usam Testcontainers)
        System.clearProperty("app.test.modo");
    }

    // ─── Testes que usam o estado preparado no @BeforeEach ────────────────────
    @Test
    @DisplayName("buscarPorId retorna response quando produto existe")
    void buscarPorId_WhenExists_ReturnsResponse() {
        var produto = new Produto(1L, "Notebook", new BigDecimal("3499.99"));
        when(produtoRepository.findById(1L)).thenReturn(Optional.of(produto));

        var result = produtoService.buscarPorId(1L);

        assertThat(result.id()).isEqualTo(1L);
        assertThat(result.nome()).isEqualTo("Notebook");
        verify(produtoRepository).findById(1L);
        verifyNoMoreInteractions(produtoRepository, categoriaService);
    }

    @Test
    @DisplayName("buscarPorId lança exceção quando produto não existe")
    void buscarPorId_WhenNotFound_ThrowsException() {
        when(produtoRepository.findById(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> produtoService.buscarPorId(99L))
                .isInstanceOf(RecursoNaoEncontradoException.class)
                .hasMessageContaining("99");
    }

    // ─── ArgumentCaptor — inspecionar o objeto passado ao mock ───────────────
    @Test
    @DisplayName("criar persiste produto com dados corretos")
    void criar_ValidRequest_PersistsCorrectly() {
        var request = new ProdutoRequest("Notebook", new BigDecimal("3499.99"), 1L);
        var saved   = new Produto(42L, "Notebook", new BigDecimal("3499.99"));
        when(produtoRepository.save(any(Produto.class))).thenReturn(saved);

        var result = produtoService.criar(request);

        var captor = ArgumentCaptor.forClass(Produto.class);
        verify(produtoRepository).save(captor.capture());
        assertThat(captor.getValue().getNome()).isEqualTo("Notebook");
        assertThat(result.id()).isEqualTo(42L);
    }
}
```

#### `@TestInstance(Lifecycle.PER_CLASS)` — `@BeforeAll` não-estático

```java
// Por padrão o JUnit cria uma nova instância da classe para cada @Test
// (PER_METHOD), tornando @BeforeAll/@AfterAll obrigatoriamente static.
//
// Com @TestInstance(PER_CLASS), uma única instância é usada para todos os testes:
//   - @BeforeAll e @AfterAll podem ser métodos de instância (não-static)
//   - Permite compartilhar estado entre testes (use com cuidado — pode gerar
//     dependência entre testes se o estado for modificado)
//   - Útil quando @BeforeAll precisa acessar campos de instância (ex.: mocks)
@ExtendWith(MockitoExtension.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@DisplayName("ProdutoService — PER_CLASS lifecycle")
class ProdutoServicePerClassTest {

    @Mock private ProdutoRepository produtoRepository;
    @InjectMocks private ProdutoService produtoService;

    // Sem static — acessa this.produtoRepository normalmente
    @BeforeAll
    void carregarDadosCompartilhados() {
        // Configura stub permanente que vale para todos os testes da classe
        when(produtoRepository.count()).thenReturn(100L);
    }

    @AfterAll
    void gerarRelatorioDeCobertura() {
        // Executado após o último teste — pode acessar campos de instância
        System.out.println("Testes concluídos. Total de produtos no mock: "
                + produtoRepository.count());
    }

    @Test
    void contagem_RetornaValorConfigurado() {
        assertThat(produtoService.contarTodos()).isEqualTo(100L);
    }
}
```

**Ciclo de vida dos testes — `@BeforeAll`, `@BeforeEach`, `@AfterEach`, `@AfterAll`:**

```java
// ─── JUnit 5 (Boot 3.x) ───────────────────────────────────────────────────────
import org.junit.jupiter.api.*;

// ─── JUnit 6 (Boot 4.x) ───────────────────────────────────────────────────────
import org.junit.api.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("PedidoService — ciclo de vida de testes")
class PedidoServiceLifecycleTest {

    // ─── @BeforeAll ────────────────────────────────────────────────────────────
    // Executado UMA VEZ antes de todos os testes da classe.
    // DEVE ser static (a menos que a classe use @TestInstance(PER_CLASS)).
    // Uso típico: inicializar recursos caros compartilhados entre testes
    // (ex: conexão de banco, servidor embarcado, dados de fixtures fixos).
    @BeforeAll
    static void configurarAmbiente() {
        // Exemplo: criar diretório temporário, carregar arquivo de configuração
        System.setProperty("app.test.mode", "true");
    }

    // ─── @BeforeEach ───────────────────────────────────────────────────────────
    // Executado antes de CADA teste.
    // Uso típico: estado inicial limpo por teste (mocks resetados, dados frescos).
    // O Mockito já reseta os mocks automaticamente com MockitoExtension —
    // use @BeforeEach para inicializar outros objetos ou estado adicional.
    @BeforeEach
    void prepararCadaTeste() {
        // Reset de estado que não é gerenciado pelo Mockito
        CarrinhoContexto.limpar();
    }

    @Test
    @DisplayName("confirmar pedido dispara email de confirmação")
    void confirmar_PedidoValido_EnviaEmail() {
        // ...
    }

    @Test
    @DisplayName("confirmar pedido lança exceção se estoque insuficiente")
    void confirmar_SemEstoque_LancaExcecao() {
        // ...
    }

    // ─── @AfterEach ────────────────────────────────────────────────────────────
    // Executado após CADA teste, mesmo que o teste tenha falhado.
    // Uso típico: limpar recursos criados pelo teste, fechar conexões temporárias,
    // verificar ausência de interações inesperadas (verifyNoMoreInteractions).
    @AfterEach
    void limparAposCadaTeste() {
        // Garante que nenhum mock foi chamado de forma não verificada
        // (substitui Mockito.validateMockitoUsage() que é mais verboso)
        CarrinhoContexto.limpar();
    }

    // ─── @AfterAll ─────────────────────────────────────────────────────────────
    // Executado UMA VEZ após todos os testes da classe, mesmo após falhas.
    // DEVE ser static (a menos que use @TestInstance(PER_CLASS)).
    // Uso típico: liberar recursos globais abertos no @BeforeAll.
    @AfterAll
    static void tearDown() {
        System.clearProperty("app.test.mode");
    }
}

// ─── @TestInstance(PER_CLASS) — permite @BeforeAll/@AfterAll não-static ────────
//
// Por padrão o JUnit cria uma nova instância da classe para CADA teste.
// Com PER_CLASS, uma única instância é usada em todos os testes da classe.
// Vantagem: @BeforeAll e @AfterAll podem ser de instância (não precisam ser static),
//            o que permite injetar dependências normalmente nesses métodos.
// Atenção: o estado da instância persiste entre testes — mocks e campos podem
//          ser "contaminados" por testes anteriores; use @BeforeEach para resetar.
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@DisplayName("PedidoController — integração (PER_CLASS)")
class PedidoControllerIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:18")
            .withReuse(true);

    @Autowired
    private PedidoRepository pedidoRepository;

    @Autowired
    private RestTestClient restTestClient;

    // ─── Não-static: possível apenas com PER_CLASS ─────────────────────────────
    @BeforeAll
    void carregarDadosBase() {
        // Inserção de dados de referência usados por todos os testes da classe.
        // Como é PER_CLASS, este método roda uma única vez — mais eficiente.
        pedidoRepository.saveAll(DadosBase.pedidosPadrao());
    }

    @BeforeEach
    void limparPedidosVariaveis() {
        // Remove apenas pedidos criados nos testes individuais;
        // os dados base do @BeforeAll são preservados.
        pedidoRepository.deleteByOrigemTeste(true);
    }

    @Test
    @DisplayName("GET /api/v1/pedidos → retorna lista paginada")
    void listar_RetornaPaginaComPedidosBase() {
        restTestClient.get()
                .uri("/api/v1/pedidos?page=0&size=10")
                .exchange()
                .expectStatus().isOk()
                .expectBody()
                .jsonPath("$.content").isNotEmpty();
    }

    // ─── @AfterAll não-static — possível com PER_CLASS ────────────────────────
    @AfterAll
    void limparDadosBase() {
        pedidoRepository.deleteAll();
    }
}
```

**Resumo do ciclo de vida:**

```
┌─────────────────────────────────────────────────────────────────────┐
│  @BeforeAll (static)  ─── uma vez por CLASSE (antes de tudo)        │
│                                                                       │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │  @BeforeEach  ─── antes de CADA @Test                        │   │
│   │  @Test        ─── execução do teste                          │   │
│   │  @AfterEach   ─── após CADA @Test (mesmo se falhou)          │   │
│   └──────────────────────────────────────────────────────────────┘   │
│   (repetido para cada método @Test da classe)                         │
│                                                                       │
│  @AfterAll (static)   ─── uma vez por CLASSE (após tudo)             │
└─────────────────────────────────────────────────────────────────────┘
```

---

### 19.3 `@WebMvcTest` — Slice Test da Camada Web

Carrega apenas o slice MVC (controllers, filters, converters, security web).
**Não carrega** services, repositories nem o banco — estes devem ser mockados.

#### 19.3.2 Controller MVC SSR com Thymeleaf

```java
@WebMvcTest(ProdutoMvcController.class)
@DisplayName("ProdutoMvcController — slice SSR")
class ProdutoMvcControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockitoBean private ProdutoService   produtoService;
    @MockitoBean private CategoriaService categoriaService;

    @BeforeEach
    void setUp() {
        when(categoriaService.listarTodas()).thenReturn(List.of(
                new CategoriaResponse(1L, "TI"),
                new CategoriaResponse(2L, "Eletrônicos")
        ));
    }

    @Test
    @DisplayName("GET /produtos → view lista com atributos corretos")
    void listar_ReturnsListViewWithModel() throws Exception {
        var page = new PageImpl<>(List.of(
                new ProdutoResponse(1L, "Notebook", new BigDecimal("3499.99"),
                        "TI", LocalDateTime.now(), LocalDateTime.now())
        ));
        when(produtoService.listar(any(), any())).thenReturn(page);

        mockMvc.perform(get("/produtos"))
                .andExpect(status().isOk())
                .andExpect(view().name("produtos/lista"))        // verifica a view
                .andExpect(model().attributeExists("produtos"))  // verifica o model
                .andExpect(model().attribute("produtos", page))
                .andExpect(content().contentTypeCompatibleWith(MediaType.TEXT_HTML))
                .andExpect(xpath("//table").exists());           // verifica o HTML
    }

    @Test
    @DisplayName("GET /produtos/novo → view formulário com form vazio")
    void exibirFormulario_ReturnsFormView() throws Exception {
        mockMvc.perform(get("/produtos/novo"))
                .andExpect(status().isOk())
                .andExpect(view().name("produtos/formulario"))
                .andExpect(model().attributeExists("produtoForm"))
                .andExpect(model().attributeExists("categorias"));
    }

    @Test
    @DisplayName("POST /produtos → redireciona após criação válida (PRG)")
    void salvar_ValidForm_RedirectsWithFlash() throws Exception {
        var produto = new Produto(1L, "Notebook", new BigDecimal("3499.99"));
        when(produtoService.criar(any())).thenReturn(produto);
        when(produtoService.nomeJaExiste(any())).thenReturn(false);

        mockMvc.perform(post("/produtos")
                        .with(csrf())                          // CSRF obrigatório com Spring Security
                        .param("nome",        "Notebook")
                        .param("preco",       "3499.99")
                        .param("categoriaId", "1"))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrlPattern("/produtos/*"))
                .andExpect(flash().attributeExists("mensagem")); // verifica flash attribute
    }

    @Test
    @DisplayName("POST /produtos → retorna form com erros quando inválido")
    void salvar_InvalidForm_ReturnsFormWithErrors() throws Exception {
        mockMvc.perform(post("/produtos")
                        .with(csrf())
                        .param("nome", "")      // nome vazio — viola @NotBlank
                        .param("preco", "")
                        .param("categoriaId", "1"))
                .andExpect(status().isOk())
                .andExpect(view().name("produtos/formulario"))
                .andExpect(model().hasErrors())
                .andExpect(model().attributeHasFieldErrors("produtoForm", "nome", "preco"));
    }
}
```

---

### 19.4 `@WebMvcTest` com Spring Security

```java
// Por padrão, @WebMvcTest aplica a configuração de Security do projeto.
// Use as anotações do spring-security-test para simular autenticação.

@WebMvcTest(PedidoController.class)
@DisplayName("PedidoController — autenticação e autorização")
class PedidoControllerSecurityTest {

    @Autowired private MockMvc       mockMvc;
    @MockitoBean private PedidoService pedidoService;

    // ─── @WithMockUser — simula usuário autenticado com roles ────────────────
    @Test
    @WithMockUser(username = "joao@empresa.com", roles = {"USER"})
    @DisplayName("GET /api/v1/pedidos → 200 para usuário autenticado")
    void listar_AuthenticatedUser_Returns200() throws Exception {
        when(pedidoService.listar(any())).thenReturn(Page.empty());

        mockMvc.perform(get("/api/v1/pedidos"))
                .andExpect(status().isOk());
    }

    // ─── Sem autenticação → 401 ───────────────────────────────────────────────
    @Test
    @DisplayName("GET /api/v1/pedidos → 401 sem autenticação")
    void listar_Unauthenticated_Returns401() throws Exception {
        mockMvc.perform(get("/api/v1/pedidos"))
                .andExpect(status().isUnauthorized());
    }

    // ─── Papel insuficiente → 403 ────────────────────────────────────────────
    @Test
    @WithMockUser(roles = {"USER"})
    @DisplayName("DELETE /api/v1/pedidos/{id} → 403 para role USER")
    void excluir_InsufficientRole_Returns403() throws Exception {
        mockMvc.perform(delete("/api/v1/pedidos/1").with(csrf()))
                .andExpect(status().isForbidden());
    }

    // ─── @WithUserDetails — carrega o UserDetails real do UserDetailsService ─
    @Test
    @WithUserDetails(value = "admin@empresa.com",
                     userDetailsServiceBeanName = "usuarioDetailsService")
    @DisplayName("DELETE → 204 para usuário ADMIN real")
    void excluir_AdminUser_Returns204() throws Exception {
        doNothing().when(pedidoService).excluir(1L);

        mockMvc.perform(delete("/api/v1/pedidos/1").with(csrf()))
                .andExpect(status().isNoContent());
    }

    // ─── Simular principal customizado (UsuarioDetails) ──────────────────────
    @Test
    @DisplayName("GET /meus-pedidos → usa o ID do usuário autenticado")
    void meusPedidos_UsesAuthenticatedUserId() throws Exception {
        var principal = new UsuarioDetails(42L, "João", "joao@email.com",
                "tenant-01", List.of(new SimpleGrantedAuthority("ROLE_USER")));

        mockMvc.perform(get("/api/v1/pedidos/meus")
                        .with(user(principal)))           // SecurityMockMvcRequestPostProcessors
                .andExpect(status().isOk());

        verify(pedidoService).listarPorCliente(eq(42L), any());
    }
}
```

---

### 19.5 `@SpringBootTest` — Teste de Integração

Carrega o contexto Spring completo. Use com `Testcontainers` para banco real.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@DisplayName("ProdutoController — integração")
class ProdutoControllerIT {

    // Testcontainers: sobe PostgreSQL real em Docker
    @Container
    static PostgreSQLContainer<?> postgres =
            new PostgreSQLContainer<>("postgres:17")
                    .withReuse(true);

    // Conecta o contexto Spring ao container PostgreSQL
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url",      postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    // RestTestClient com RANDOM_PORT — injeta o cliente HTTP real
    @Autowired
    private RestTestClient restTestClient;

    @Autowired
    private ProdutoRepository produtoRepository;

    @BeforeEach
    void setUp() {
        produtoRepository.deleteAll();
    }

    @Test
    @DisplayName("CRUD completo: criar → buscar → atualizar → deletar")
    void crudCompleto() {
        // CREATE
        var created = restTestClient.post()
                .uri("/api/v1/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue("""
                        {"nome": "Notebook Dell", "preco": 3499.99, "categoriaId": 1}
                        """)
                .exchange()
                .expectStatus().isCreated()
                .expectBody(ProdutoResponse.class)
                .returnResult()
                .getResponseBody();

        assertThat(created).isNotNull();
        Long id = created.id();

        // READ
        restTestClient.get()
                .uri("/api/v1/produtos/{id}", id)
                .exchange()
                .expectStatus().isOk()
                .expectBody()
                .jsonPath("$.nome").isEqualTo("Notebook Dell");

        // UPDATE
        restTestClient.put()
                .uri("/api/v1/produtos/{id}", id)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue("""
                        {"nome": "Notebook Dell Atualizado", "preco": 3299.99, "categoriaId": 1}
                        """)
                .exchange()
                .expectStatus().isOk()
                .expectBody()
                .jsonPath("$.nome").isEqualTo("Notebook Dell Atualizado");

        // DELETE
        restTestClient.delete()
                .uri("/api/v1/produtos/{id}", id)
                .exchange()
                .expectStatus().isNoContent();

        // VERIFY DELETED
        restTestClient.get()
                .uri("/api/v1/produtos/{id}", id)
                .exchange()
                .expectStatus().isNotFound();
    }

    // ─── Teste de validação end-to-end ────────────────────────────────────────
    @Test
    @DisplayName("POST com dados inválidos → 400 com erros por campo")
    void criar_InvalidData_Returns400WithFieldErrors() {
        restTestClient.post()
                .uri("/api/v1/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue("""
                        {"nome": "", "preco": -1, "categoriaId": null}
                        """)
                .exchange()
                .expectStatus().isBadRequest()
                .expectBody()
                .jsonPath("$.errors.nome").isNotEmpty()
                .jsonPath("$.errors.preco").isNotEmpty()
                .jsonPath("$.errors.categoriaId").isNotEmpty();
    }
}
```

---

### 19.6 Configuração de Contexto de Teste

```java
// ─── @TestConfiguration — beans extras apenas nos testes ─────────────────────
@TestConfiguration
public class TestSecurityConfig {

    // Substitui a SecurityFilterChain de produção durante os testes
    @Bean
    @Primary
    public SecurityFilterChain testSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
        return http.build();
    }
}

// Uso: importar apenas onde necessário
@WebMvcTest(ProdutoController.class)
@Import(TestSecurityConfig.class)
class ProdutoControllerTest { /* ... */ }

// ─── @ActiveProfiles — ativar perfil de teste ─────────────────────────────────
@SpringBootTest
@ActiveProfiles("test")   // carrega application-test.yml
class ProdutoServiceIT { /* ... */ }

// ─── @Sql — executar scripts SQL antes/depois dos testes ──────────────────────
@SpringBootTest
@Sql(scripts = "/sql/setup-produtos.sql",
     executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
@Sql(scripts = "/sql/cleanup.sql",
     executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
class RelatorioServiceIT { /* ... */ }

// ─── @Transactional em testes — rollback automático ──────────────────────────
@SpringBootTest
@Transactional   // cada teste é executado em transação que faz rollback no final
class ProdutoRepositoryTest {

    @Autowired private ProdutoRepository repository;

    @Test
    void salvar_PersisteProduto() {
        repository.save(new Produto("Notebook", new BigDecimal("3499.99")));
        // rollback automático ao final do teste — banco fica limpo
        assertThat(repository.count()).isEqualTo(1);
    }
}

// ─── @DirtiesContext — reinicia contexto após teste que modifica estado global ─
@SpringBootTest
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
class IntegracaoComEfeitos { /* ... */ }
```

---

### 19.7 Testando Upload, CORS e SSE

```java
@WebMvcTest(ArquivoController.class)
class ArquivoControllerTest {

    @Autowired  private MockMvc       mockMvc;
    @MockitoBean private ArquivoService arquivoService;

    // ─── Upload de arquivo ────────────────────────────────────────────────────
    @Test
    @WithMockUser
    void upload_ArquivoValido_Returns201() throws Exception {
        var arquivo = new MockMultipartFile(
                "arquivo",                         // nome do parâmetro (@RequestParam)
                "foto.jpg",                        // nome original do arquivo
                MediaType.IMAGE_JPEG_VALUE,        // content type
                "conteudo-binario".getBytes());    // bytes do arquivo

        when(arquivoService.salvar(any()))
                .thenReturn(new ArquivoResponse("id-1", "foto.jpg", 100L,
                        "image/jpeg", "/api/v1/arquivos/id-1"));

        mockMvc.perform(multipart("/api/v1/arquivos").file(arquivo)
                        .with(csrf()))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.nome").value("foto.jpg"));
    }

    // ─── CORS — verifica headers na resposta ──────────────────────────────────
    @Test
    void cors_AllowedOrigin_RetornaHeaders() throws Exception {
        mockMvc.perform(options("/api/v1/arquivos")
                        .header("Origin",                        "https://app.empresa.com.br")
                        .header("Access-Control-Request-Method", "POST"))
                .andExpect(status().isOk())
                .andExpect(header().string("Access-Control-Allow-Origin",
                        "https://app.empresa.com.br"))
                .andExpect(header().exists("Access-Control-Allow-Methods"));
    }
}

// ─── SSE — Server-Sent Events ─────────────────────────────────────────────────
@WebMvcTest(EventoController.class)
class EventoControllerTest {

    @Autowired  private MockMvc         mockMvc;
    @MockitoBean private EventoBroadcaster broadcaster;

    @Test
    @WithMockUser
    void stream_RetornaTextEventStream() throws Exception {
        var emitter = new SseEmitter();
        when(broadcaster.subscribe(any(), any())).thenReturn(emitter);

        mockMvc.perform(get("/api/v1/eventos/stream")
                        .param("usuarioId", "1")
                        .accept(MediaType.TEXT_EVENT_STREAM_VALUE))
                .andExpect(status().isOk())
                .andExpect(content().contentTypeCompatibleWith(
                        MediaType.TEXT_EVENT_STREAM_VALUE));
    }
}
```

---



## 20. Tópicos Relevantes Não Cobertos Neste Documento

Assuntos relacionados ao Spring MVC ainda ausentes neste documento, ordenados por relevância prática.

### 20.1 Tópicos Ausentes — Alta Relevância

Introduzido no Spring 6, é a forma moderna de declarar clients HTTP (similar ao Feign) usando interfaces anotadas com `@GetExchange`, `@PostExchange` etc., resolvidos por `HttpServiceProxyFactory`. Direto ao território do Spring MVC e completamente ausente.


### 20.2 Tópicos Ausentes — Relevância Moderada

**3. Endpoints funcionais — `RouterFunction` / WebMvc.fn**
Alternativa ao `@Controller` introduzida no Spring 5, disponível no MVC via `WebMvcConfigurer.addRouterFunctions()`. Não substitui `@Controller` no dia a dia mas é relevante para cenários de roteamento dinâmico ou bibliotecas internas.

### 20.3 Tópicos Ausentes — Relevância Menor mas Notáveis

**4. `WebMvcTest` + `MockMvcRestDocumentation`** — geração de documentação a partir dos testes (Spring REST Docs)

**5. Virtual Threads — seção dedicada** — mencionado em vários lugares, mas sem consolidar os impactos no MVC (thread locals, `@Async`, `SecurityContextHolder`, `TransactionSynchronizationManager`)

### 20.4 Resumo por Prioridade

| Prioridade | Tópico | Justificativa |
|---|---|---|
| 🟡 Média | WebMvc.fn | Nicho, mas parte oficial da stack MVC |
| 🟢 Baixa | `MockMvcRestDocumentation` | Útil para gerar documentação a partir dos testes |
| 🟢 Baixa | Virtual Threads | Merece consolidação dos impactos no MVC e em contexto assíncrono |

---


## Referências e Créditos

### Documentação Oficial

| Recurso | URL |
|---|---|
| Spring MVC Reference | https://docs.spring.io/spring-framework/reference/web/webmvc.html |
| Spring Boot Web Reference | https://docs.spring.io/spring-boot/reference/web/servlet.html |
| Spring Framework 7 — API Versioning | https://docs.spring.io/spring-framework/reference/web/webmvc-versioning.html |
| Spring Security Reference | https://docs.spring.io/spring-security/reference/ |
| Thymeleaf Documentation | https://www.thymeleaf.org/documentation.html |
| Thymeleaf Layout Dialect | https://ultraq.github.io/thymeleaf-layout-dialect/ |
| Thymeleaf + Spring Security | https://github.com/thymeleaf/thymeleaf-extras-springsecurity |
| Jakarta Bean Validation 3.0 | https://beanvalidation.org/3.0/ |
| JTE — Java Template Engine | https://jte.gg/ |
| JUnit 5 User Guide | https://junit.org/junit5/docs/current/user-guide/ |
| Mockito Documentation | https://site.mockito.org/ |
| Testcontainers for Java | https://java.testcontainers.org/ |

### Especificações e RFCs

| Especificação | URL |
|---|---|
| RFC 8594 — Sunset HTTP Header | https://www.rfc-editor.org/rfc/rfc8594 |
| Jakarta EE 10 Platform | https://jakarta.ee/specifications/platform/10/ |
| WHATWG HTML — `data-*` attributes | https://html.spec.whatwg.org/multipage/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes |

### Referências Complementares

| Recurso | URL |
|---|---|
| Baeldung — Spring MVC | https://www.baeldung.com/spring-mvc |
| Baeldung — Spring Security | https://www.baeldung.com/security-spring |
| Spring Boot API Versioning (Baeldung) | https://www.baeldung.com/spring-api-versioning |
| Validation Groups — Stack Overflow | https://stackoverflow.com/a/35359965 |
| Spring Blog — API Versioning in Spring | https://spring.io/blog/2025/09/16/api-versioning-in-spring/ |

---

### Créditos

Este documento foi elaborado de forma colaborativa entre:

- **[@ftsuda-senac](https://github.com/ftsuda-senac)** — autor das perguntas, revisões e direcionamentos que moldaram o conteúdo e a estrutura do documento.

- **[Claude Sonnet 4.6](https://www.anthropic.com/claude)** (Anthropic) — modelo de linguagem utilizado para geração, estruturação e revisão do conteúdo técnico ao longo de toda a conversa.

