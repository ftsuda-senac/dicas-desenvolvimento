# Revisão do arquivo Dicas-e-Tutoriais-Spring.md

Documento com sugestões de correção e melhoria para o arquivo `Dicas-e-Tutoriais-Spring.md`, gerado em 2026-06-07.

Dividido em duas partes:
1. [Boas práticas gerais e adequação para iniciantes](#parte-1--boas-práticas-gerais-e-adequação-para-iniciantes)
2. [Mudanças para aderência ao Spring Boot 4.0.x e Java 25](#parte-2--mudanças-para-aderência-ao-spring-boot-40x-e-java-25)

---

## Parte 1 — Boas práticas gerais e adequação para iniciantes

### 1. Informar a versão-alvo logo no início

O documento não especifica para qual versão do Spring Boot foi escrito. Isso é crítico para iniciantes: comportamentos, nomes de propriedades e APIs mudaram bastante entre Spring Boot 2.x e 3.x. Uma seção introdutória indicando "Este guia foi escrito com base no Spring Boot 3.x / Java 17+" evita confusão imediata.

---

### 2. Erro tipográfico no comentário do MVC

```java
// @WebMvcConfig /* NÃO INCLUIR ESSA ANOTAÇÃO */
```

O nome correto da anotação é `@EnableWebMvc`, não `@WebMvcConfig`. Para iniciantes esse erro pode causar confusão sobre qual anotação evitar.

---

### 3. Sumário incompleto

A seção **"Erros frequentes"** existe no documento mas não aparece no sumário. Iniciantes que leram a lista de conteúdos não saberão que essa seção existe.

---

### 4. Exemplo do `SecurityFilterChain` sem contexto de configuração

O exemplo mostrado exibe apenas o `@Bean` isolado, sem o `@Configuration` e o `@EnableWebSecurity`. Para um iniciante, não fica claro onde esse método deve ficar:

```java
// Sugestão: mostrar o contexto completo
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception { ... }
}
```

---

### 5. Ausência de exemplo básico de `@RestController`

O documento menciona `@Controller` e `@RestController` na seção MVC, mas não mostra um exemplo mínimo de endpoint. Para iniciantes, um snippet com `@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestBody` e `ResponseEntity` é o "hello world" da stack. Sem isso, o documento pressupõe conhecimento que o público-alvo talvez não tenha.

---

### 6. `@Transactional(readOnly = true)` não mencionado

A seção de transações orienta bem o uso do `@Transactional`, mas omite o padrão `readOnly = true` em métodos de leitura. É uma prática amplamente recomendada que melhora desempenho (evita flush do Hibernate, permite otimizações do banco) e sinaliza intenção:

```java
@Transactional(readOnly = true)
public List<Pedido> listar() { ... }
```

---

### 7. `@Modifying` não citado para queries de escrita com `@Query`

A seção de repositórios mostra exemplos de `@Query` apenas para leitura. Iniciantes frequentemente precisam fazer `UPDATE` ou `DELETE` via JPQL e ficam confusos sem ver o par `@Modifying` + `@Transactional`:

```java
@Modifying
@Transactional
@Query("UPDATE Pedido p SET p.status = :status WHERE p.id = :id")
void atualizarStatus(@Param("id") Long id, @Param("status") String status);
```

---

### 8. Migração `javax` → `jakarta` não explicada

O documento usa `jakarta.persistence` e `jakarta.validation` corretamente para Spring Boot 3.x, mas não explica a mudança. Para iniciantes que encontram tutoriais antigos com `javax.*`, isso é fonte de erros de compilação confusos. Uma nota sobre a migração Jakarta EE 9+ seria valiosa.

---

### 9. Bean Validation: ausência de exemplo de anotações comuns

A referência ao Bean Validation aponta para um link externo, mas não mostra nenhum exemplo inline. Para iniciantes, um snippet básico com `@NotBlank`, `@NotNull`, `@Size`, `@Valid` em um DTO e no controller seria muito mais acessível:

```java
public record NovoPedidoDto(
    @NotBlank String descricao,
    @Positive BigDecimal valor
) {}

@PostMapping
ResponseEntity<PedidoDto> criar(@Valid @RequestBody NovoPedidoDto dto) { ... }
```

---

### 10. Recomendação ambígua entre ModelMapper e MapStruct

ModelMapper aparece primeiro na lista de ferramentas de mapeamento. Na prática atual, **MapStruct é amplamente preferido** em projetos Spring Boot por ser baseado em geração de código em tempo de compilação (mais seguro, mais rápido, sem reflexão em runtime). A lista deveria indicar MapStruct como `[Recomendado]` e explicar brevemente o critério de escolha.

---

### 11. Ausência de `application.yml` como alternativa

O documento apresenta apenas `application.properties`. Para iniciantes, vale ao menos mencionar que `application.yml` é uma alternativa equivalente e bastante comum em projetos Spring Boot, pois encontrarão os dois formatos em tutoriais e projetos open source.

---

### 12. Virtual Threads não mencionados (Spring Boot 3.2+ / Java 21)

Para um guia atualizado, vale mencionar o suporte a Virtual Threads (introduzido no Spring Boot 3.2 com Java 21) como uma forma simples de aumentar throughput em aplicações MVC bloqueantes com uma única propriedade:

```properties
spring.threads.virtual.enabled=true
```

É especialmente relevante pois contrasta com a complexidade do WebFlux para o mesmo objetivo de escalabilidade.

---

### 13. Seção "Arquitetura" mistura dois níveis de detalhe

A seção "Arquitetura" apresenta conceitos como DDD e Onion Architecture — que são avançados — junto com anotações básicas como `@Component`. Para iniciantes, essa mistura pode gerar desorientação. Seria mais útil separar explicitamente o que é "fundamento obrigatório" do que é "aprofundamento opcional".

---

### 14. Links com `/current/` podem ficar desatualizados

As referências que usam `/current/` na URL da documentação do Spring apontam sempre para a versão mais recente, o que pode mudar o conteúdo ao longo do tempo. Para um guia pedagógico, é melhor orientar o leitor a substituir `current` pela versão do projeto — essa orientação deveria aparecer junto ao primeiro link, não só no final da seção.

---

### Resumo de prioridade (Parte 1)

| Prioridade | Item |
|---|---|
| Alta | #2 (erro de nome de anotação), #3 (sumário incompleto), #4 (contexto do SecurityFilterChain), #5 (exemplo básico de controller) |
| Média | #1 (versão-alvo), #6 (readOnly), #7 (@Modifying), #8 (jakarta), #9 (Bean Validation) |
| Baixa | #10 (MapStruct), #11 (yml), #12 (Virtual Threads), #13 (organização), #14 (links) |

---

## Parte 2 — Mudanças para aderência ao Spring Boot 4.0.x e Java 25

### Geral (cabeçalho / introdução)

- **Declarar versão-alvo**: indicar explicitamente Spring Boot 4.0.x, Spring Framework 7.0, Java 17 mínimo, Java 25 recomendado.
- **Adicionar aviso sobre `javax` → `jakarta`**: o documento já usa `jakarta.*` em alguns pontos, mas não avisa o leitor sobre a troca — crítico para quem vem de tutoriais mais antigos ou de Spring Boot 2.x.

---

### Sumário

- Adicionar entrada para a seção **"Erros frequentes"** (já existe no corpo mas está ausente do sumário).

---

### Arquitetura / Core e Configuração

Sem mudanças de breaking change nessa seção. O conteúdo sobre DI via construtor e `@Configuration`/`@Bean` permanece válido.

---

### MVC

| Item no documento | Mudança necessária |
|---|---|
| Comentário `@WebMvcConfig` | Corrigir para `@EnableWebMvc` (erro tipográfico preexistente) |
| Tratamento de erros com `ProblemDetail` | Sem mudança — RFC 7807 continua válido |
| — | **Adicionar**: suporte nativo a versionamento de API com `@GetMapping(version = "1.1+")` e as propriedades `spring.mvc.apiversion.*`, novo recurso do Spring MVC 7 |

---

### Data JPA

| Item no documento | Mudança necessária |
|---|---|
| Imports `jakarta.persistence` | Já correto para Boot 3.x/4.x — manter, mas adicionar nota explicativa sobre a mudança histórica |
| Exemplos com `@Query` | Adicionar exemplo com `@Modifying` para queries de escrita (UPDATE/DELETE) |
| Hibernate em anotações | **Adicionar aviso**: Hibernate 7 (usado no Boot 4.0) removeu `session.save()`, `session.update()` e `session.saveOrUpdate()` — usar apenas `entityManager.persist()` / `entityManager.merge()` da API padrão JPA |
| `@NativeQuery` | Já presente no documento — compatível com Boot 4.0 |
| — | **Adicionar**: menção ao recurso *AOT Repositories* do Boot 4.0, que compila query methods em código-fonte na build para suporte a GraalVM native image |

---

### Security

Esta é a seção com mais breaking changes:

| Item no documento | Mudança necessária |
|---|---|
| Exemplo de `SecurityFilterChain` | Adicionar `@Configuration` e `@EnableWebSecurity` ao exemplo |
| `WebSecurityConfigurerAdapter` | Incluir aviso explícito de que foi **removido** no Spring Security 7 (não só depreciado) |
| `authorizeRequests()` | Se aparecer em qualquer exemplo futuro, substituir por `authorizeHttpRequests()` — o método antigo foi **removido** |
| `antMatchers()` | Substituir por `requestMatchers()` — foi **removido** |
| CSRF | **Adicionar ponto de atenção**: no Spring Security 7, a proteção CSRF está habilitada por padrão para **todos** os endpoints, inclusive APIs REST. APIs stateless devem desabilitar explicitamente com `.csrf(csrf -> csrf.disable())` |
| JWT / OAuth2 | Conteúdo permanece válido em linhas gerais |

---

### Configuração e Properties

| Item no documento | Mudança necessária |
|---|---|
| `spring.jpa.open-in-view=false` | Permanece válido e recomendado |
| `spring.jpa.show-sql=true` | Permanece válido |
| `spring.jackson.*` | **Atenção**: Spring Boot 4.0 adota **Jackson 3**, com group ID `tools.jackson` no lugar de `com.fasterxml.jackson`. As propriedades `spring.jackson.*` seguem válidas, mas qualquer customização via `Jackson2ObjectMapperBuilderCustomizer` deve ser migrada para `JsonMapperBuilderCustomizer` |
| Links com `/current/` nas URLs da documentação Spring | Substituir `current` pela versão concreta `4.0` ou orientar explicitamente o leitor |
| — | **Adicionar** propriedade renomeada: `management.tracing.enabled` → `management.tracing.export.enabled` |
| — | **Adicionar nota**: `spring.devtools.livereload.enabled` agora é `false` por padrão no Boot 4.0 (era `true`) |
| — | **Adicionar nota**: `management.endpoint.health.probes.enabled` agora é `true` por padrão (útil para deploys em Kubernetes) |

---

### Ecossistema, testes e operação

| Item no documento | Mudança necessária |
|---|---|
| `@MockBean` / `@SpyBean` | **Renomear**: foram substituídos por `@MockitoBean` e `@MockitoSpyBean` no Spring Boot 4.0 (anotações antigas removidas) |
| Testes com MockMVC | **Adicionar**: `@AutoConfigureMockMvc` agora é obrigatório explicitamente; Boot 4.0 não auto-configura MockMvc por padrão via `@SpringBootTest` |
| Observabilidade | **Adicionar**: `spring-boot-starter-opentelemetry` como starter dedicado no Boot 4.0 |
| — | **Adicionar**: Virtual Threads finalizados no Java 21, melhorias significativas de pinning e throughput no Java 25 — habilitação via `spring.threads.virtual.enabled=true` |

---

### Java complementar

Adicionar recursos finalizados no Java 25 relevantes para código Spring:

- **Flexible Constructor Bodies** (JEP 495): `super()` / `this()` não precisa mais ser o primeiro statement — útil em construtores de beans com herança.
- **Scoped Values** (JEP 506): alternativa imutável e mais segura ao `ThreadLocal` para propagação de contexto (relevante para Spring Security `SecurityContextHolder` e rastreamento).
- **Compact Object Headers** (JEP 519): redução do tamanho de cabeçalho de objetos de 96/128 bits para 64 bits — melhora de memória em aplicações com muitos beans/entidades.
- **Module Import Declarations** (JEP 511): `import module java.sql` importa todos os pacotes de um módulo de uma vez — simplifica exemplos com muitos imports Jakarta.

---

### Erros frequentes / Troubleshooting

Adicionar os seguintes novos erros frequentes relativos ao Boot 4.0:

- **`[Atenção]`** Usar `@MockBean` / `@SpyBean` (removidos) em vez de `@MockitoBean` / `@MockitoSpyBean`.
- **`[Atenção]`** Usar imports `javax.*` em projetos Boot 4.0 (Jakarta EE 11 — apenas `jakarta.*` suportado).
- **`[Atenção]`** Usar `session.save()` / `session.update()` da API Hibernate — removidos no Hibernate 7; usar `entityManager.persist()` / `entityManager.merge()`.
- **`[Atenção]`** Esperar que APIs REST funcionem sem CSRF token no Boot 4.0 — proteção CSRF habilitada por padrão para todos os endpoints.
- **`[Atenção]`** Usar `authorizeRequests()` / `antMatchers()` — removidos no Spring Security 7.
- **`[Atenção]`** Configurar Jackson 3 com `Jackson2ObjectMapperBuilderCustomizer` (Boot 3.x) — substituída por `JsonMapperBuilderCustomizer` no Boot 4.0.

---

### Resumo de prioridade (Parte 2)

| Prioridade | Item |
|---|---|
| **Alta** (quebra em runtime ou compilação) | CSRF por padrão no Security 7, `@MockBean` → `@MockitoBean`, remoção de `authorizeRequests()`/`antMatchers()`, Hibernate 7 removeu `session.save()`, Jackson 3 muda group ID |
| **Média** (comportamento silenciosamente diferente) | DevTools live reload off por padrão, health probes on por padrão, `management.tracing.enabled` renomeado, `@AutoConfigureMockMvc` obrigatório |
| **Baixa** (novas funcionalidades a documentar) | API versioning no MVC, AOT Repositories, `spring-boot-starter-opentelemetry`, Scoped Values, Virtual Threads melhorias Java 25 |
