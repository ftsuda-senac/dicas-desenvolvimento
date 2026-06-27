# Testes para Backend em Aplicações Spring Boot — Guia Abrangente

> **Objetivo:** Cobrir de forma prática e didática todas as camadas de teste em aplicações Spring Boot — desde testes unitários com JUnit 5 e Mockito até testes de carga, segurança e end-to-end — com exemplos de código, configurações e boas práticas para cada cenário.

---

## Sumário

1. [Fundamentos e Pirâmide de Testes](#1-fundamentos-e-pirâmide-de-testes)
2. [JUnit 5 e 6 — Plataforma e Recursos](#2-junit-5-e-6--plataforma-e-recursos)
3. [Mockito — Mocks, Stubs e Verificações](#3-mockito--mocks-stubs-e-verificações)
4. [Testes Unitários em Spring Boot](#4-testes-unitários-em-spring-boot)
5. [MockMvc — Testes de Controllers](#5-mockmvc--testes-de-controllers)
6. [REST Assured — Testes Fluentes de API](#6-rest-assured--testes-fluentes-de-api)
7. [@DataJpaTest — Testes de Repositórios](#7-datajpatest--testes-de-repositórios)
8. [Testes de Migrations — Flyway e Liquibase](#8-testes-de-migrations--flyway-e-liquibase)
9. [Testes de Integração com @SpringBootTest](#9-testes-de-integração-com-springboottest)
10. [WireMock — Mock de Serviços HTTP Externos](#10-wiremock--mock-de-serviços-http-externos)
11. [RestClient e RestTestClient (Spring Boot 4)](#11-restclient-e-resttestclient-spring-boot-4)
12. [Testcontainers — Infraestrutura Real nos Testes](#12-testcontainers--infraestrutura-real-nos-testes)
13. [Contract Testing — Pact e Spring Cloud Contract](#13-contract-testing--pact-e-spring-cloud-contract)
14. [Testes Reativos — WebFlux e WebTestClient](#14-testes-reativos--webflux-e-webtestclient)
15. [Testes End-to-End com Selenium](#15-testes-end-to-end-com-selenium)
16. [Testes End-to-End com Playwright](#16-testes-end-to-end-com-playwright)
17. [BDD com Cucumber](#17-bdd-com-cucumber)
18. [Testes de Carga e Performance](#18-testes-de-carga-e-performance)
19. [Testes de Segurança — Vulnerabilidades e Penetração](#19-testes-de-segurança--vulnerabilidades-e-penetração)
20. [Property-Based Testing com jqwik](#20-property-based-testing-com-jqwik)
21. [Snapshot e Approval Testing](#21-snapshot-e-approval-testing)
22. [ArchUnit — Testes de Arquitetura](#22-archunit--testes-de-arquitetura)
23. [Testes de Observabilidade](#23-testes-de-observabilidade)
24. [Cobertura de Código e Mutation Testing](#24-cobertura-de-código-e-mutation-testing)
25. [Organização, Boas Práticas e CI/CD](#25-organização-boas-práticas-e-cicd)

---

## 1. Fundamentos e Pirâmide de Testes

### 1.1 A Pirâmide de Testes

A pirâmide de testes orienta a proporção ideal entre os diferentes tipos de teste em uma aplicação:

```
            ╱╲
           ╱  ╲          E2E / UI
          ╱ E2E╲         (poucos, lentos, frágeis)
         ╱──────╲
        ╱        ╲       Integração
       ╱Integration╲     (quantidade média, velocidade média)
      ╱──────────────╲
     ╱                ╲   Unitários
    ╱    Unit Tests     ╲  (muitos, rápidos, estáveis)
   ╱────────────────────╲
```

| Camada | Quantidade | Velocidade | Custo de Manutenção | Confiança |
|--------|-----------|------------|---------------------|-----------|
| **Unitários** | ~70% | Milissegundos | Baixo | Lógica isolada |
| **Integração** | ~20% | Segundos | Médio | Componentes juntos |
| **E2E** | ~10% | Minutos | Alto | Sistema completo |

### 1.2 Tipos de teste no contexto Spring Boot

| Tipo | O que testa | Ferramentas |
|------|------------|-------------|
| **Unitário** | Classes isoladas (Service, Util) | JUnit 5, Mockito |
| **Slice Test** | Fatia da aplicação (Web, JPA, JSON) | `@WebMvcTest`, `@DataJpaTest`, `@JsonTest` |
| **Integração** | Múltiplos componentes reais juntos | `@SpringBootTest`, Testcontainers |
| **E2E** | Fluxo completo via navegador/API | Selenium, Playwright, RestAssured |
| **Carga** | Comportamento sob stress | Gatling, JMeter, k6 |
| **Segurança** | Vulnerabilidades e pentest | OWASP ZAP, Spring Security Test |

### 1.3 Dependências Maven essenciais

```xml
<dependencies>
    <!-- Spring Boot Starter Test (JUnit 5, Mockito, AssertJ, MockMvc, JSONPath) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Spring Security Test -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Testcontainers -->
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

    <!-- H2 para testes que não precisam de banco real -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 2. JUnit 5 e 6 — Plataforma e Recursos

### 2.1 Arquitetura do JUnit 5/6

O JUnit 5 (e sua evolução, o JUnit 6) é composto por três módulos independentes:

```
┌─────────────────────────────────────────────────┐
│              JUnit Platform                      │
│  (Motor de execução — integra IDEs e build tools)│
├─────────────────────┬───────────────────────────┤
│   JUnit Jupiter     │    JUnit Vintage          │
│   (API JUnit 5/6)   │    (Compat. JUnit 3/4)    │
│                     │    ⚠ Depreciado no JUnit 6│
└─────────────────────┴───────────────────────────┘
```

| Módulo | Função |
|--------|--------|
| **JUnit Platform** | Motor de descoberta e execução de testes; integra com Maven Surefire, Gradle, IDEs |
| **JUnit Jupiter** | API de programação e extensão (`@Test`, `@BeforeEach`, `@ExtendWith`) — mesmos pacotes no JUnit 5 e 6 |
| **JUnit Vintage** | Permite rodar testes JUnit 3 e 4 (depreciado no JUnit 6; removido no Spring Boot 4) |

> **JUnit 6:** A partir do JUnit 6.0, todos os três módulos usam **versionamento unificado** (mesmo número de versão), simplificando a gestão de dependências.

### 2.2 Anotações fundamentais

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Calculadora de Descontos")
class DescontoServiceTest {

    private DescontoService service;

    @BeforeAll
    static void setupGlobal() {
        // Executado uma vez antes de todos os testes da classe
    }

    @BeforeEach
    void setup() {
        service = new DescontoService();
    }

    @Test
    @DisplayName("Deve aplicar 10% para compras acima de R$100")
    void deveAplicarDescontoPorcentagem() {
        BigDecimal resultado = service.calcular(new BigDecimal("200.00"), TipoDesconto.PERCENTUAL_10);
        assertEquals(new BigDecimal("180.00"), resultado);
    }

    @Test
    @DisplayName("Não deve aceitar valor negativo")
    void naoDeveAceitarValorNegativo() {
        assertThrows(IllegalArgumentException.class,
            () -> service.calcular(new BigDecimal("-10"), TipoDesconto.PERCENTUAL_10));
    }

    @Test
    @Disabled("Aguardando definição da regra de negócio")
    void deveAplicarDescontoProgressivo() {
        // TODO: implementar quando regra estiver definida
    }

    @AfterEach
    void tearDown() {
        // Limpeza após cada teste
    }

    @AfterAll
    static void tearDownGlobal() {
        // Limpeza final
    }
}
```

### 2.3 Assertions avançadas

```java
@Test
void assertionsAvancadas() {
    Produto produto = new Produto("Notebook", new BigDecimal("3500.00"), 5);

    // Assertions agrupadas — todas executam mesmo que uma falhe
    assertAll("Validação do produto",
        () -> assertNotNull(produto.getNome()),
        () -> assertEquals("Notebook", produto.getNome()),
        () -> assertTrue(produto.getPreco().compareTo(BigDecimal.ZERO) > 0),
        () -> assertTrue(produto.getEstoque() >= 0)
    );

    // Timeout
    assertTimeout(Duration.ofSeconds(2), () -> {
        service.processarPedidoPesado();
    });

    // Mensagem customizada
    assertEquals(5, produto.getEstoque(),
        () -> "Estoque deveria ser 5, mas era " + produto.getEstoque());
}
```

### 2.4 Testes parametrizados

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class ValidadorCpfTest {

    private final ValidadorCpf validador = new ValidadorCpf();

    @ParameterizedTest(name = "CPF {0} deve ser válido")
    @ValueSource(strings = {"529.982.247-25", "111.444.777-35"})
    void deveAceitarCpfValido(String cpf) {
        assertTrue(validador.isValido(cpf));
    }

    @ParameterizedTest(name = "CPF {0} deve ser inválido")
    @NullAndEmptySource
    @ValueSource(strings = {"000.000.000-00", "123", "abc.def.ghi-jk"})
    void deveRejeitarCpfInvalido(String cpf) {
        assertFalse(validador.isValido(cpf));
    }

    @ParameterizedTest(name = "Desconto de {1}% sobre {0} = {2}")
    @CsvSource({
        "100.00, 10, 90.00",
        "200.00, 15, 170.00",
        "50.00,   0, 50.00"
    })
    void deveCalcularDesconto(BigDecimal valor, int percentual, BigDecimal esperado) {
        assertEquals(esperado, service.aplicarDesconto(valor, percentual));
    }

    @ParameterizedTest
    @MethodSource("fornecerPedidos")
    void deveValidarPedido(Pedido pedido, boolean esperado) {
        assertEquals(esperado, validador.isValido(pedido));
    }

    static Stream<Arguments> fornecerPedidos() {
        return Stream.of(
            Arguments.of(new Pedido(List.of(new Item("A", 1)), "12345-678"), true),
            Arguments.of(new Pedido(List.of(), "12345-678"), false),
            Arguments.of(new Pedido(List.of(new Item("A", 1)), null), false)
        );
    }
}
```

### 2.5 Testes dinâmicos e aninhados

```java
@DisplayName("Serviço de Pedidos")
class PedidoServiceTest {

    @Nested
    @DisplayName("Quando o pedido é válido")
    class PedidoValido {

        @Test
        @DisplayName("Deve criar pedido com status PENDENTE")
        void deveCriarComStatusPendente() {
            Pedido pedido = service.criar(pedidoValido);
            assertEquals(StatusPedido.PENDENTE, pedido.getStatus());
        }

        @Test
        @DisplayName("Deve calcular total com frete")
        void deveCalcularTotalComFrete() {
            Pedido pedido = service.criar(pedidoValido);
            assertTrue(pedido.getTotal().compareTo(pedido.getSubtotal()) > 0);
        }
    }

    @Nested
    @DisplayName("Quando o pedido é inválido")
    class PedidoInvalido {

        @Test
        @DisplayName("Deve rejeitar pedido sem itens")
        void deveRejeitarSemItens() {
            assertThrows(PedidoInvalidoException.class,
                () -> service.criar(pedidoSemItens));
        }
    }

    // Testes dinâmicos — gerados em runtime
    @TestFactory
    @DisplayName("Validações de status de pedido")
    Collection<DynamicTest> testesDeTransicaoDeStatus() {
        return List.of(
            dynamicTest("PENDENTE -> CONFIRMADO", () ->
                assertTrue(StatusPedido.PENDENTE.podeTransicionarPara(StatusPedido.CONFIRMADO))),
            dynamicTest("CONFIRMADO -> ENVIADO", () ->
                assertTrue(StatusPedido.CONFIRMADO.podeTransicionarPara(StatusPedido.ENVIADO))),
            dynamicTest("CANCELADO -> CONFIRMADO (inválido)", () ->
                assertFalse(StatusPedido.CANCELADO.podeTransicionarPara(StatusPedido.CONFIRMADO)))
        );
    }
}
```

### 2.6 Extensões do JUnit 5

```java
// Extensão customizada para medir tempo de execução
public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    @Override
    public void beforeTestExecution(ExtensionContext context) {
        context.getStore(Namespace.GLOBAL).put("start", System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) {
        long start = context.getStore(Namespace.GLOBAL).get("start", long.class);
        long duration = System.currentTimeMillis() - start;
        System.out.printf("[%s] executou em %d ms%n", context.getDisplayName(), duration);
    }
}

// Uso
@ExtendWith(TimingExtension.class)
class MeuServiceTest {
    // Todos os testes terão tempo medido
}
```

### 2.7 JUnit 5 vs JUnit 4 — Tabela de migração

| JUnit 4 | JUnit 5 / 6 | Observação |
|---------|-------------|-----------|
| `@Test` (org.junit) | `@Test` (org.junit.jupiter.api) | Pacote diferente |
| `@Before` / `@After` | `@BeforeEach` / `@AfterEach` | Renomeado |
| `@BeforeClass` / `@AfterClass` | `@BeforeAll` / `@AfterAll` | Renomeado |
| `@Ignore` | `@Disabled` | Renomeado |
| `@RunWith` | `@ExtendWith` | Modelo de extensão unificado |
| `@Rule` / `@ClassRule` | `@ExtendWith` + `@RegisterExtension` | Rules substituídas por extensões |
| `Assert.assertEquals` | `Assertions.assertEquals` | Mensagem é o último parâmetro |
| `@Category` | `@Tag` | Categorização de testes |
| `expected = Exception.class` | `assertThrows(Exception.class, ...)` | Lambda-based |

### 2.8 JUnit 6 — Novidades e Migração

O **JUnit 6.0.0** foi lançado em **setembro de 2025** e é a base de testes do **Spring Boot 4**. O Spring Boot 4 e o Spring Framework 7 exigem exclusivamente JUnit Jupiter 6 — o suporte ao JUnit 4 foi completamente removido.

A migração do JUnit 5.x para o 6.0 é consideravelmente mais simples do que a transição do JUnit 4 para o 5 — apenas APIs depreciadas há mais de dois anos foram removidas, tornando o JUnit 6 um *drop-in replacement* na maioria dos casos.

#### Principais novidades do JUnit 6

| Novidade | Descrição |
|----------|-----------|
| **Java 17 como baseline** | Versão mínima do Java passa a ser 17 (era 8 no JUnit 5) |
| **Versionamento unificado** | Platform, Jupiter e Vintage agora usam o **mesmo número de versão** |
| **Null-safety com JSpecify** | Anotações `@Nullable`, `@NonNull`, `@NullMarked` em toda a API |
| **Suporte a Kotlin coroutines** | Métodos `@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll` podem ser `suspend` |
| **FastCSV** | `@CsvSource` e `@CsvFileSource` usam FastCSV (substitui univocity-parsers, mais rápido, RFC 4180) |
| **Display names aprimorados** | `@ParameterizedClass` e `@ParameterizedTest` usam formatação consistente `name = value` |
| **Modelo de cancelamento** | Cancelamento adequado para pipelines CI — testes podem ser interrompidos de forma limpa |

#### O que foi removido no JUnit 6

| Removido | Alternativa |
|----------|------------|
| `junit-platform-runner` | Use suporte nativo da IDE ou Maven Surefire/Gradle |
| `junit-platform-jfr` (Flight Recorder) | Removido sem substituto direto |
| `ReflectionSupport.loadClass()` | Usar `Class.forName()` diretamente |
| `MethodOrderer.Alphanumeric` | Usar `MethodOrderer.MethodName` |
| `getOrComputeIfAbsent` (Store) | Usar `computeIfAbsent` |
| **JUnit Vintage** | **Depreciado** (ainda funciona, mas emite aviso em discovery) |

#### Migração do JUnit 5 para JUnit 6

```xml
<!-- Antes (JUnit 5) -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.11.x</version>
    <scope>test</scope>
</dependency>

<!-- Depois (JUnit 6) — mesma estrutura de pacotes -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>6.0.3</version>
    <scope>test</scope>
</dependency>
```

As anotações e classes de assertion **permanecem no mesmo pacote** (`org.junit.jupiter.api`) — os imports não mudam. Para projetos grandes, a receita [OpenRewrite JUnit5to6Migration](https://docs.openrewrite.org/recipes/java/testing/junit6/junit5to6migration) automatiza a atualização de dependências e remoção de APIs depreciadas.

```java
// Exemplo com suspend function (Kotlin) — novidade do JUnit 6
class ProdutoServiceKotlinTest {

    @Test
    suspend fun `deve buscar produto assíncrono`() {
        val produto = produtoService.buscarAsync("SKU-001")
        assertNotNull(produto)
        assertEquals("Widget", produto.nome)
    }

    @BeforeEach
    suspend fun setup() {
        repository.limparAsync()
    }
}
```

---

## 3. Mockito — Mocks, Stubs e Verificações

### 3.1 Conceitos fundamentais

```
┌──────────────────────────────────────────────────────────┐
│                   Tipos de Test Doubles                   │
├─────────┬────────────────────────────────────────────────┤
│ Dummy   │ Objeto passado mas nunca usado (preenche param)│
│ Stub    │ Retorna valores pré-definidos                  │
│ Mock    │ Verifica se métodos foram chamados corretamente│
│ Spy     │ Objeto real com comportamento parcial mockado  │
│ Fake    │ Implementação simplificada (ex: banco in-memory│
└─────────┴────────────────────────────────────────────────┘
```

### 3.2 Criando mocks e stubs

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {

    @Mock
    private PedidoRepository pedidoRepository;

    @Mock
    private EstoqueService estoqueService;

    @Mock
    private NotificacaoService notificacaoService;

    @InjectMocks
    private PedidoService pedidoService;

    @Test
    void deveCriarPedidoQuandoEstoqueDisponivel() {
        // Arrange — configurar stubs
        var item = new ItemPedido("PROD-001", 2);
        var pedidoRequest = new PedidoRequest(List.of(item), "cliente@email.com");

        when(estoqueService.verificarDisponibilidade("PROD-001", 2)).thenReturn(true);
        when(pedidoRepository.save(any(Pedido.class))).thenAnswer(invocation -> {
            Pedido p = invocation.getArgument(0);
            p.setId(1L);
            return p;
        });

        // Act
        Pedido resultado = pedidoService.criar(pedidoRequest);

        // Assert
        assertNotNull(resultado.getId());
        assertEquals(StatusPedido.PENDENTE, resultado.getStatus());

        // Verify — verificar interações
        verify(estoqueService).verificarDisponibilidade("PROD-001", 2);
        verify(pedidoRepository).save(any(Pedido.class));
        verify(notificacaoService).enviar(eq("cliente@email.com"), anyString());
        verifyNoMoreInteractions(estoqueService);
    }

    @Test
    void deveLancarExcecaoQuandoEstoqueInsuficiente() {
        var item = new ItemPedido("PROD-001", 100);
        var request = new PedidoRequest(List.of(item), "cliente@email.com");

        when(estoqueService.verificarDisponibilidade("PROD-001", 100)).thenReturn(false);

        assertThrows(EstoqueInsuficienteException.class,
            () -> pedidoService.criar(request));

        verify(pedidoRepository, never()).save(any());
        verify(notificacaoService, never()).enviar(anyString(), anyString());
    }
}
```

### 3.3 Argument Matchers e Captors

```java
@Test
void deveCapturarArgumentosSalvos() {
    var request = new PedidoRequest(List.of(new ItemPedido("A", 1)), "test@mail.com");
    when(estoqueService.verificarDisponibilidade(anyString(), anyInt())).thenReturn(true);
    when(pedidoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

    pedidoService.criar(request);

    // ArgumentCaptor para inspecionar o objeto salvo
    ArgumentCaptor<Pedido> captor = ArgumentCaptor.forClass(Pedido.class);
    verify(pedidoRepository).save(captor.capture());

    Pedido pedidoSalvo = captor.getValue();
    assertEquals(StatusPedido.PENDENTE, pedidoSalvo.getStatus());
    assertNotNull(pedidoSalvo.getDataCriacao());
    assertEquals(1, pedidoSalvo.getItens().size());
}

@Test
void deveUsarMatchersCustomizados() {
    // Matcher customizado com argThat
    when(pedidoRepository.findByFilters(argThat(filtro ->
        filtro.getStatus() == StatusPedido.PENDENTE &&
        filtro.getDataInicio() != null
    ))).thenReturn(List.of(new Pedido()));

    var filtro = new PedidoFiltro(StatusPedido.PENDENTE, LocalDate.now(), null);
    List<Pedido> resultado = pedidoService.buscarPorFiltros(filtro);

    assertFalse(resultado.isEmpty());
}
```

### 3.4 Spy — Objeto real com mock parcial

```java
@Test
void deveUsarSpyParaMockParcial() {
    // Spy cria um wrapper sobre o objeto real
    List<String> listaReal = new ArrayList<>();
    List<String> listaSpy = spy(listaReal);

    // Métodos reais funcionam
    listaSpy.add("elemento");
    assertEquals(1, listaSpy.size());

    // Pode sobrescrever comportamento específico
    doReturn(100).when(listaSpy).size();
    assertEquals(100, listaSpy.size());
}

@ExtendWith(MockitoExtension.class)
class AuditoriaServiceTest {

    @Spy
    private AuditoriaService auditoriaService = new AuditoriaService();

    @Test
    void deveRegistrarAuditoriaMasNaoEnviarEmail() {
        // doNothing para métodos void no spy
        doNothing().when(auditoriaService).enviarEmailAuditoria(anyString());

        auditoriaService.registrarAcao("LOGIN", "usuario1");

        verify(auditoriaService).registrarAcao("LOGIN", "usuario1");
        verify(auditoriaService).enviarEmailAuditoria(anyString());
    }
}
```

### 3.5 BDD Mockito — Given/When/Then

```java
import static org.mockito.BDDMockito.*;

@Test
void deveBuscarClientePorId_BddStyle() {
    // Given
    var clienteEsperado = new Cliente(1L, "Maria", "maria@mail.com");
    given(clienteRepository.findById(1L)).willReturn(Optional.of(clienteEsperado));

    // When
    Cliente resultado = clienteService.buscarPorId(1L);

    // Then
    then(clienteRepository).should().findById(1L);
    assertThat(resultado.getNome()).isEqualTo("Maria");
}
```

### 3.6 Mockando métodos estáticos e construtores (Mockito 5+)

```java
@Test
void deveMockarMetodoEstatico() {
    try (MockedStatic<LocalDateTime> mockedTime = mockStatic(LocalDateTime.class)) {
        LocalDateTime fixedTime = LocalDateTime.of(2025, 1, 15, 10, 30);
        mockedTime.when(LocalDateTime::now).thenReturn(fixedTime);

        Pedido pedido = pedidoService.criar(request);

        assertEquals(fixedTime, pedido.getDataCriacao());
    }
    // Fora do try, LocalDateTime.now() volta ao comportamento real
}

@Test
void deveMockarConstrutor() {
    try (MockedConstruction<HttpClient> mocked = mockConstruction(HttpClient.class,
            (mock, context) -> {
                when(mock.send(any())).thenReturn(new Response(200, "OK"));
            })) {

        // Qualquer new HttpClient() dentro deste bloco será um mock
        var resultado = integracaoService.chamarApiExterna();
        assertEquals("OK", resultado);
    }
}
```

---

## 4. Testes Unitários em Spring Boot

### 4.1 Testando Services sem Spring Context

A abordagem preferida para testes unitários é **não carregar o contexto Spring** — usar Mockito diretamente:

```java
@ExtendWith(MockitoExtension.class)
class ClienteServiceTest {

    @Mock
    private ClienteRepository clienteRepository;

    @Mock
    private EnderecoService enderecoService;

    @InjectMocks
    private ClienteService clienteService;

    @Test
    void deveCriarClienteComEnderecoValidado() {
        var dto = new ClienteDTO("João", "joao@mail.com", "01310-100");
        var endereco = new Endereco("Av Paulista", "São Paulo", "SP");

        when(enderecoService.buscarPorCep("01310-100")).thenReturn(endereco);
        when(clienteRepository.existsByEmail("joao@mail.com")).thenReturn(false);
        when(clienteRepository.save(any(Cliente.class))).thenAnswer(inv -> {
            Cliente c = inv.getArgument(0);
            c.setId(1L);
            return c;
        });

        Cliente resultado = clienteService.criar(dto);

        assertAll(
            () -> assertEquals("João", resultado.getNome()),
            () -> assertEquals("Av Paulista", resultado.getEndereco().getRua()),
            () -> assertNotNull(resultado.getId())
        );
    }

    @Test
    void deveLancarExcecaoParaEmailDuplicado() {
        var dto = new ClienteDTO("João", "existente@mail.com", "01310-100");
        when(clienteRepository.existsByEmail("existente@mail.com")).thenReturn(true);

        var ex = assertThrows(BusinessException.class,
            () -> clienteService.criar(dto));

        assertEquals("Email já cadastrado", ex.getMessage());
        verify(enderecoService, never()).buscarPorCep(anyString());
    }
}
```

### 4.2 Mockando Repository — Cenários comuns

```java
@ExtendWith(MockitoExtension.class)
class ProdutoServiceTest {

    @Mock
    private ProdutoRepository produtoRepository;

    @InjectMocks
    private ProdutoService produtoService;

    // --- findById: presente vs ausente ---

    @Test
    void deveBuscarProdutoPorId() {
        var produto = new Produto(1L, "Notebook", new BigDecimal("3500.00"), Categoria.ELETRONICOS, 10);
        when(produtoRepository.findById(1L)).thenReturn(Optional.of(produto));

        ProdutoDTO resultado = produtoService.buscarPorId(1L);

        assertEquals("Notebook", resultado.nome());
        verify(produtoRepository).findById(1L);
    }

    @Test
    void deveLancarExcecaoQuandoProdutoNaoEncontrado() {
        when(produtoRepository.findById(999L)).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class,
            () -> produtoService.buscarPorId(999L));
    }

    // --- save: capturar e validar o objeto persistido ---

    @Test
    void deveSalvarProdutoComDadosCorretos() {
        var request = new CriarProdutoDTO("Monitor", "1800.00", "ELETRONICOS", 15);
        when(produtoRepository.save(any(Produto.class))).thenAnswer(inv -> {
            Produto p = inv.getArgument(0);
            p.setId(10L);
            return p;
        });

        ProdutoDTO resultado = produtoService.criar(request);

        ArgumentCaptor<Produto> captor = ArgumentCaptor.forClass(Produto.class);
        verify(produtoRepository).save(captor.capture());

        Produto salvo = captor.getValue();
        assertEquals("Monitor", salvo.getNome());
        assertEquals(new BigDecimal("1800.00"), salvo.getPreco());
        assertEquals(Categoria.ELETRONICOS, salvo.getCategoria());
        assertEquals(15, salvo.getEstoque());
    }

    // --- findAll / queries com lista vazia ---

    @Test
    void deveRetornarListaVaziaQuandoNaoHaProdutos() {
        when(produtoRepository.findByCategoria(Categoria.ELETRONICOS))
            .thenReturn(Collections.emptyList());

        List<ProdutoDTO> resultado = produtoService.listarPorCategoria("ELETRONICOS");

        assertTrue(resultado.isEmpty());
    }

    // --- delete: verificar que foi chamado ---

    @Test
    void deveRemoverProduto() {
        var produto = new Produto(1L, "Mouse", new BigDecimal("60"), Categoria.PERIFERICOS, 20);
        when(produtoRepository.findById(1L)).thenReturn(Optional.of(produto));
        doNothing().when(produtoRepository).delete(produto);

        produtoService.remover(1L);

        verify(produtoRepository).findById(1L);
        verify(produtoRepository).delete(produto);
    }

    @Test
    void naoDeveRemoverProdutoInexistente() {
        when(produtoRepository.findById(999L)).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class,
            () -> produtoService.remover(999L));

        verify(produtoRepository, never()).delete(any());
    }

    // --- existsBy: verificação de unicidade ---

    @Test
    void deveRejeitarProdutoComNomeDuplicado() {
        when(produtoRepository.existsByNome("Notebook")).thenReturn(true);
        var request = new CriarProdutoDTO("Notebook", "3500.00", "ELETRONICOS", 5);

        assertThrows(BusinessException.class,
            () -> produtoService.criar(request));

        verify(produtoRepository, never()).save(any());
    }

    // --- Paginação ---

    @Test
    void deveRetornarProdutosPaginados() {
        var pageable = PageRequest.of(0, 10, Sort.by("nome"));
        var produtos = List.of(
            new Produto(1L, "Mouse", new BigDecimal("60"), Categoria.PERIFERICOS, 20));
        var page = new PageImpl<>(produtos, pageable, 1);

        when(produtoRepository.findAll(pageable)).thenReturn(page);

        Page<ProdutoDTO> resultado = produtoService.listarPaginado(pageable);

        assertEquals(1, resultado.getTotalElements());
        assertEquals("Mouse", resultado.getContent().get(0).nome());
    }
}
```

### 4.3 Mockando serviços externos (APIs HTTP)

Quando o Service depende de uma API externa (via `RestClient`, `WebClient`, ou `RestTemplate`), o teste unitário mocka o client HTTP:

```java
@ExtendWith(MockitoExtension.class)
class PagamentoServiceTest {

    @Mock
    private PagamentoGatewayClient gatewayClient;

    @Mock
    private PedidoRepository pedidoRepository;

    @Mock
    private PagamentoRepository pagamentoRepository;

    @InjectMocks
    private PagamentoService pagamentoService;

    @Test
    void deveProcessarPagamentoComSucesso() {
        var pedido = new Pedido(1L, StatusPedido.PENDENTE, new BigDecimal("200.00"));
        when(pedidoRepository.findById(1L)).thenReturn(Optional.of(pedido));

        var gatewayResponse = new GatewayResponse("TXN-12345", StatusGateway.APPROVED, "Aprovado");
        when(gatewayClient.cobrar(any(GatewayRequest.class))).thenReturn(gatewayResponse);
        when(pagamentoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

        PagamentoDTO resultado = pagamentoService.processar(1L, MetodoPagamento.CARTAO_CREDITO);

        assertAll(
            () -> assertEquals("TXN-12345", resultado.transacaoId()),
            () -> assertEquals(StatusPagamento.APROVADO, resultado.status())
        );

        // Verifica que o pedido foi atualizado
        ArgumentCaptor<Pedido> pedidoCaptor = ArgumentCaptor.forClass(Pedido.class);
        verify(pedidoRepository).save(pedidoCaptor.capture());
        assertEquals(StatusPedido.PAGO, pedidoCaptor.getValue().getStatus());
    }

    @Test
    void deveTratarRecusaDoGateway() {
        var pedido = new Pedido(1L, StatusPedido.PENDENTE, new BigDecimal("200.00"));
        when(pedidoRepository.findById(1L)).thenReturn(Optional.of(pedido));

        var gatewayResponse = new GatewayResponse(null, StatusGateway.DECLINED, "Saldo insuficiente");
        when(gatewayClient.cobrar(any())).thenReturn(gatewayResponse);

        var ex = assertThrows(PagamentoRecusadoException.class,
            () -> pagamentoService.processar(1L, MetodoPagamento.CARTAO_CREDITO));

        assertEquals("Saldo insuficiente", ex.getMessage());
        verify(pedidoRepository, never()).save(any());
    }

    @Test
    void deveTratarTimeoutDoGateway() {
        var pedido = new Pedido(1L, StatusPedido.PENDENTE, new BigDecimal("200.00"));
        when(pedidoRepository.findById(1L)).thenReturn(Optional.of(pedido));
        when(gatewayClient.cobrar(any()))
            .thenThrow(new ResourceAccessException("Connection timed out"));

        assertThrows(IntegracaoException.class,
            () -> pagamentoService.processar(1L, MetodoPagamento.CARTAO_CREDITO));

        // Verificar que pagamento ficou com status de erro para reprocessamento
        ArgumentCaptor<Pagamento> captor = ArgumentCaptor.forClass(Pagamento.class);
        verify(pagamentoRepository).save(captor.capture());
        assertEquals(StatusPagamento.ERRO_COMUNICACAO, captor.getValue().getStatus());
    }

    @Test
    void deveReprocessarPagamentoComRetentativa() {
        var pedido = new Pedido(1L, StatusPedido.PENDENTE, new BigDecimal("200.00"));
        when(pedidoRepository.findById(1L)).thenReturn(Optional.of(pedido));

        // Primeira chamada falha, segunda funciona
        when(gatewayClient.cobrar(any()))
            .thenThrow(new ResourceAccessException("Timeout"))
            .thenReturn(new GatewayResponse("TXN-999", StatusGateway.APPROVED, "OK"));
        when(pagamentoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

        PagamentoDTO resultado = pagamentoService.processarComRetry(1L, MetodoPagamento.PIX);

        assertEquals(StatusPagamento.APROVADO, resultado.status());
        verify(gatewayClient, times(2)).cobrar(any());
    }
}
```

### 4.4 Mockando serviço de envio de e-mail (SMTP)

```java
@ExtendWith(MockitoExtension.class)
class NotificacaoServiceTest {

    @Mock
    private JavaMailSender mailSender;

    @Mock
    private TemplateEngine templateEngine;

    @Mock
    private ClienteRepository clienteRepository;

    @InjectMocks
    private NotificacaoService notificacaoService;

    @Captor
    private ArgumentCaptor<MimeMessage> mimeMessageCaptor;

    @Test
    void deveEnviarEmailDeConfirmacaoDePedido() {
        var cliente = new Cliente(1L, "Maria", "maria@mail.com");
        var pedido = new Pedido(100L, StatusPedido.CONFIRMADO, new BigDecimal("350.00"));
        when(clienteRepository.findById(1L)).thenReturn(Optional.of(cliente));

        MimeMessage mimeMessage = new MimeMessage((Session) null);
        when(mailSender.createMimeMessage()).thenReturn(mimeMessage);
        when(templateEngine.process(eq("email/pedido-confirmado"), any(Context.class)))
            .thenReturn("<html><body>Pedido #100 confirmado</body></html>");
        doNothing().when(mailSender).send(any(MimeMessage.class));

        notificacaoService.notificarPedidoConfirmado(1L, pedido);

        verify(mailSender).send(any(MimeMessage.class));
        verify(templateEngine).process(eq("email/pedido-confirmado"), argThat(ctx ->
            ctx.getVariable("nomeCliente").equals("Maria") &&
            ctx.getVariable("numeroPedido").equals(100L)
        ));
    }

    @Test
    void naoDeveLancarExcecaoSeEnvioDeEmailFalhar() {
        var cliente = new Cliente(1L, "Maria", "maria@mail.com");
        when(clienteRepository.findById(1L)).thenReturn(Optional.of(cliente));
        when(mailSender.createMimeMessage()).thenReturn(new MimeMessage((Session) null));
        when(templateEngine.process(anyString(), any())).thenReturn("<html></html>");
        doThrow(new MailSendException("SMTP error")).when(mailSender).send(any(MimeMessage.class));

        // Não deve propagar — e-mail é best-effort
        assertDoesNotThrow(() ->
            notificacaoService.notificarPedidoConfirmado(1L, new Pedido()));

        verify(mailSender).send(any(MimeMessage.class));
    }
}
```

### 4.5 Mockando serviço de armazenamento (S3/MinIO)

```java
@ExtendWith(MockitoExtension.class)
class ArquivoServiceTest {

    @Mock
    private S3Client s3Client;

    @Mock
    private ArquivoRepository arquivoRepository;

    @InjectMocks
    private ArquivoService arquivoService;

    @Test
    void deveRealizarUploadParaS3() {
        byte[] conteudo = "conteúdo do arquivo".getBytes();
        var multipart = new MockMultipartFile("file", "relatorio.pdf",
            "application/pdf", conteudo);

        when(s3Client.putObject(any(PutObjectRequest.class), any(RequestBody.class)))
            .thenReturn(PutObjectResponse.builder().eTag("etag-123").build());
        when(arquivoRepository.save(any())).thenAnswer(inv -> {
            Arquivo a = inv.getArgument(0);
            a.setId(1L);
            return a;
        });

        ArquivoDTO resultado = arquivoService.upload(multipart, "documentos");

        assertAll(
            () -> assertNotNull(resultado.id()),
            () -> assertTrue(resultado.url().contains("documentos/")),
            () -> assertEquals("relatorio.pdf", resultado.nomeOriginal()),
            () -> assertEquals("application/pdf", resultado.contentType())
        );

        // Verifica parâmetros enviados ao S3
        ArgumentCaptor<PutObjectRequest> s3Captor = ArgumentCaptor.forClass(PutObjectRequest.class);
        verify(s3Client).putObject(s3Captor.capture(), any(RequestBody.class));

        PutObjectRequest s3Request = s3Captor.getValue();
        assertEquals("meu-bucket", s3Request.bucket());
        assertTrue(s3Request.key().startsWith("documentos/"));
        assertEquals("application/pdf", s3Request.contentType());
    }

    @Test
    void deveTratarErroDeUploadNoS3() {
        var multipart = new MockMultipartFile("file", "foto.jpg", "image/jpeg", new byte[100]);

        when(s3Client.putObject(any(PutObjectRequest.class), any(RequestBody.class)))
            .thenThrow(S3Exception.builder().message("Access Denied").build());

        assertThrows(ArmazenamentoException.class,
            () -> arquivoService.upload(multipart, "fotos"));

        verify(arquivoRepository, never()).save(any());
    }

    @Test
    void deveDeletarArquivoDoS3EBanco() {
        var arquivo = new Arquivo(1L, "documentos/abc-123.pdf", "relatorio.pdf");
        when(arquivoRepository.findById(1L)).thenReturn(Optional.of(arquivo));
        when(s3Client.deleteObject(any(DeleteObjectRequest.class)))
            .thenReturn(DeleteObjectResponse.builder().build());

        arquivoService.remover(1L);

        verify(s3Client).deleteObject(argThat(req ->
            req.bucket().equals("meu-bucket") &&
            req.key().equals("documentos/abc-123.pdf")));
        verify(arquivoRepository).delete(arquivo);
    }
}
```

### 4.6 Mockando cache (Redis / Spring Cache)

```java
@ExtendWith(MockitoExtension.class)
class CatalogoServiceTest {

    @Mock
    private ProdutoRepository produtoRepository;

    @Mock
    private StringRedisTemplate redisTemplate;

    @Mock
    private ValueOperations<String, String> valueOps;

    @InjectMocks
    private CatalogoService catalogoService;

    private final ObjectMapper objectMapper = new ObjectMapper();

    @BeforeEach
    void setup() {
        when(redisTemplate.opsForValue()).thenReturn(valueOps);
    }

    @Test
    void deveRetornarDoCacheQuandoDisponivel() throws Exception {
        var produto = new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS");
        String json = objectMapper.writeValueAsString(produto);
        when(valueOps.get("produto:1")).thenReturn(json);

        ProdutoDTO resultado = catalogoService.buscarPorId(1L);

        assertEquals("Notebook", resultado.nome());
        verify(produtoRepository, never()).findById(any());
    }

    @Test
    void deveBuscarNoBancoEPopularCacheQuandoCacheMiss() throws Exception {
        when(valueOps.get("produto:1")).thenReturn(null);  // cache miss

        var entity = new Produto(1L, "Notebook", new BigDecimal("3500.00"), Categoria.ELETRONICOS, 10);
        when(produtoRepository.findById(1L)).thenReturn(Optional.of(entity));

        ProdutoDTO resultado = catalogoService.buscarPorId(1L);

        assertEquals("Notebook", resultado.nome());
        verify(produtoRepository).findById(1L);
        verify(valueOps).set(eq("produto:1"), anyString(), eq(Duration.ofMinutes(30)));
    }

    @Test
    void deveInvalidarCacheAoAtualizarProduto() {
        var entity = new Produto(1L, "Notebook", new BigDecimal("3500"), Categoria.ELETRONICOS, 10);
        when(produtoRepository.findById(1L)).thenReturn(Optional.of(entity));
        when(produtoRepository.save(any())).thenReturn(entity);
        when(redisTemplate.delete("produto:1")).thenReturn(true);

        catalogoService.atualizar(1L, new AtualizarProdutoDTO("Notebook Pro", "4200.00"));

        verify(redisTemplate).delete("produto:1");
        verify(produtoRepository).save(any());
    }
}
```

### 4.7 Mockando múltiplos repositórios (Service com lógica transacional)

```java
@ExtendWith(MockitoExtension.class)
class TransferenciaServiceTest {

    @Mock
    private ContaRepository contaRepository;

    @Mock
    private TransferenciaRepository transferenciaRepository;

    @Mock
    private LimiteService limiteService;

    @InjectMocks
    private TransferenciaService transferenciaService;

    @Test
    void deveRealizarTransferenciaComSucesso() {
        var origem = new Conta(1L, "Maria", new BigDecimal("1000.00"));
        var destino = new Conta(2L, "João", new BigDecimal("500.00"));

        when(contaRepository.findByIdWithLock(1L)).thenReturn(Optional.of(origem));
        when(contaRepository.findByIdWithLock(2L)).thenReturn(Optional.of(destino));
        when(limiteService.verificarLimiteDiario(1L, new BigDecimal("200.00"))).thenReturn(true);
        when(transferenciaRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

        TransferenciaDTO resultado = transferenciaService.transferir(1L, 2L, new BigDecimal("200.00"));

        assertEquals(StatusTransferencia.CONCLUIDA, resultado.status());

        // Verificar saldos atualizados
        ArgumentCaptor<Conta> contaCaptor = ArgumentCaptor.forClass(Conta.class);
        verify(contaRepository, times(2)).save(contaCaptor.capture());

        List<Conta> contasSalvas = contaCaptor.getAllValues();
        Conta origemSalva = contasSalvas.stream().filter(c -> c.getId().equals(1L)).findFirst().get();
        Conta destinoSalvo = contasSalvas.stream().filter(c -> c.getId().equals(2L)).findFirst().get();

        assertEquals(new BigDecimal("800.00"), origemSalva.getSaldo());
        assertEquals(new BigDecimal("700.00"), destinoSalvo.getSaldo());
    }

    @Test
    void deveRejeitarTransferenciaSemSaldo() {
        var origem = new Conta(1L, "Maria", new BigDecimal("50.00"));
        var destino = new Conta(2L, "João", new BigDecimal("500.00"));

        when(contaRepository.findByIdWithLock(1L)).thenReturn(Optional.of(origem));
        when(contaRepository.findByIdWithLock(2L)).thenReturn(Optional.of(destino));

        assertThrows(SaldoInsuficienteException.class,
            () -> transferenciaService.transferir(1L, 2L, new BigDecimal("100.00")));

        verify(transferenciaRepository, never()).save(any());
        verify(contaRepository, never()).save(any());
    }

    @Test
    void deveRejeitarTransferenciaAcimaDoLimiteDiario() {
        var origem = new Conta(1L, "Maria", new BigDecimal("50000.00"));
        var destino = new Conta(2L, "João", new BigDecimal("500.00"));

        when(contaRepository.findByIdWithLock(1L)).thenReturn(Optional.of(origem));
        when(contaRepository.findByIdWithLock(2L)).thenReturn(Optional.of(destino));
        when(limiteService.verificarLimiteDiario(1L, new BigDecimal("20000.00"))).thenReturn(false);

        assertThrows(LimiteExcedidoException.class,
            () -> transferenciaService.transferir(1L, 2L, new BigDecimal("20000.00")));

        verify(transferenciaRepository, never()).save(any());
    }

    @Test
    void naoDevePermitirTransferenciaParaMesmaConta() {
        assertThrows(BusinessException.class,
            () -> transferenciaService.transferir(1L, 1L, new BigDecimal("100.00")));

        verify(contaRepository, never()).findByIdWithLock(any());
    }
}
```

### 4.8 Mockando serviço externo com RestClient / WebClient

```java
@ExtendWith(MockitoExtension.class)
class CepServiceTest {

    @Mock
    private RestClient restClient;

    @Mock
    private RestClient.RequestHeadersUriSpec requestHeadersUriSpec;

    @Mock
    private RestClient.RequestHeadersSpec requestHeadersSpec;

    @Mock
    private RestClient.ResponseSpec responseSpec;

    @InjectMocks
    private CepService cepService;

    @Test
    void deveBuscarEnderecoPorCep() {
        var endereco = new EnderecoViaCep("01310-100", "Avenida Paulista", "", "Bela Vista", "São Paulo", "SP");

        when(restClient.get()).thenReturn(requestHeadersUriSpec);
        when(requestHeadersUriSpec.uri("/ws/{cep}/json", "01310100")).thenReturn(requestHeadersSpec);
        when(requestHeadersSpec.retrieve()).thenReturn(responseSpec);
        when(responseSpec.body(EnderecoViaCep.class)).thenReturn(endereco);

        Endereco resultado = cepService.buscar("01310-100");

        assertEquals("Avenida Paulista", resultado.getLogradouro());
        assertEquals("São Paulo", resultado.getCidade());
        assertEquals("SP", resultado.getUf());
    }

    @Test
    void deveLancarExcecaoParaCepInexistente() {
        when(restClient.get()).thenReturn(requestHeadersUriSpec);
        when(requestHeadersUriSpec.uri("/ws/{cep}/json", "00000000")).thenReturn(requestHeadersSpec);
        when(requestHeadersSpec.retrieve()).thenReturn(responseSpec);
        when(responseSpec.body(EnderecoViaCep.class))
            .thenThrow(new HttpClientErrorException(HttpStatus.NOT_FOUND));

        assertThrows(CepNotFoundException.class,
            () -> cepService.buscar("00000-000"));
    }
}
```

> **Dica:** Para APIs externas complexas com muitas interações de `RestClient`, prefira usar `MockRestServiceServer` (seção 8.2) ou WireMock em testes de integração — evita o mock verboso da cadeia de chamadas fluentes.

### 4.9 Testando Mappers e DTOs

```java
class ProdutoMapperTest {

    private final ProdutoMapper mapper = new ProdutoMapper();

    @Test
    void deveConverterEntityParaDTO() {
        var entity = new Produto(1L, "Notebook", new BigDecimal("3500.00"),
                                 Categoria.ELETRONICOS, 10);

        ProdutoDTO dto = mapper.toDTO(entity);

        assertAll(
            () -> assertEquals(1L, dto.id()),
            () -> assertEquals("Notebook", dto.nome()),
            () -> assertEquals("R$ 3.500,00", dto.precoFormatado()),
            () -> assertEquals("ELETRONICOS", dto.categoria())
        );
    }

    @Test
    void deveConverterDTOParaEntity() {
        var dto = new CriarProdutoDTO("Mouse", "59.90", "PERIFERICOS", 50);

        Produto entity = mapper.toEntity(dto);

        assertEquals(new BigDecimal("59.90"), entity.getPreco());
        assertNull(entity.getId());
    }
}
```

### 4.10 Testando Validações (Bean Validation)

```java
class ClienteDTOValidationTest {

    private static Validator validator;

    @BeforeAll
    static void setup() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void deveAceitarDTOValido() {
        var dto = new ClienteDTO("Maria", "maria@email.com", "01310-100");
        Set<ConstraintViolation<ClienteDTO>> violations = validator.validate(dto);
        assertTrue(violations.isEmpty());
    }

    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = {"  ", "ab"})
    void deveRejeitarNomeInvalido(String nome) {
        var dto = new ClienteDTO(nome, "maria@email.com", "01310-100");
        Set<ConstraintViolation<ClienteDTO>> violations = validator.validate(dto);

        assertFalse(violations.isEmpty());
        assertTrue(violations.stream()
            .anyMatch(v -> v.getPropertyPath().toString().equals("nome")));
    }
}
```

---

## 5. MockMvc — Testes de Controllers

### 5.1 @WebMvcTest — Slice test de Controller

`@WebMvcTest` carrega apenas a camada web (controllers, filters, converters), sem Services ou Repositories reais:

```java
@WebMvcTest(ProdutoController.class)
class ProdutoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProdutoService produtoService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void deveListarProdutos() throws Exception {
        var produtos = List.of(
            new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS"),
            new ProdutoDTO(2L, "Mouse", "R$ 59,90", "PERIFERICOS")
        );
        when(produtoService.listarTodos()).thenReturn(produtos);

        mockMvc.perform(get("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(2)))
            .andExpect(jsonPath("$[0].nome").value("Notebook"))
            .andExpect(jsonPath("$[1].categoria").value("PERIFERICOS"));
    }

    @Test
    void deveCriarProduto() throws Exception {
        var request = new CriarProdutoDTO("Teclado", "149.90", "PERIFERICOS", 30);
        var response = new ProdutoDTO(3L, "Teclado", "R$ 149,90", "PERIFERICOS");

        when(produtoService.criar(any(CriarProdutoDTO.class))).thenReturn(response);

        mockMvc.perform(post("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(3))
            .andExpect(jsonPath("$.nome").value("Teclado"));
    }

    @Test
    void deveRetornar400ParaRequestInvalido() throws Exception {
        var requestInvalido = new CriarProdutoDTO("", "-10", null, -1);

        mockMvc.perform(post("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(requestInvalido)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").isArray())
            .andExpect(jsonPath("$.errors", hasSize(greaterThan(0))));

        verify(produtoService, never()).criar(any());
    }

    @Test
    void deveRetornar404ParaProdutoInexistente() throws Exception {
        when(produtoService.buscarPorId(999L))
            .thenThrow(new ResourceNotFoundException("Produto não encontrado"));

        mockMvc.perform(get("/api/produtos/{id}", 999))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.message").value("Produto não encontrado"));
    }
}
```

### 5.2 Testando paginação e filtros

```java
@Test
void deveRetornarProdutosPaginados() throws Exception {
    var page = new PageImpl<>(
        List.of(new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS")),
        PageRequest.of(0, 10, Sort.by("nome")),
        1
    );
    when(produtoService.listarPaginado(any(Pageable.class))).thenReturn(page);

    mockMvc.perform(get("/api/produtos")
            .param("page", "0")
            .param("size", "10")
            .param("sort", "nome,asc"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.content", hasSize(1)))
        .andExpect(jsonPath("$.totalElements").value(1))
        .andExpect(jsonPath("$.totalPages").value(1));
}

@Test
void deveFiltrarProdutosPorCategoria() throws Exception {
    when(produtoService.buscarPorCategoria("ELETRONICOS"))
        .thenReturn(List.of(new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS")));

    mockMvc.perform(get("/api/produtos")
            .param("categoria", "ELETRONICOS"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].categoria").value("ELETRONICOS"));
}
```

### 5.3 Testando upload de arquivos

```java
@Test
void deveRealizarUploadDeImagem() throws Exception {
    MockMultipartFile arquivo = new MockMultipartFile(
        "imagem",
        "produto.jpg",
        MediaType.IMAGE_JPEG_VALUE,
        "conteudo-fake-da-imagem".getBytes()
    );

    when(produtoService.salvarImagem(eq(1L), any())).thenReturn("/uploads/produto.jpg");

    mockMvc.perform(multipart("/api/produtos/{id}/imagem", 1)
            .file(arquivo))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.url").value("/uploads/produto.jpg"));
}
```

### 5.4 Testando controllers com Spring Security

```java
@WebMvcTest(AdminController.class)
@Import(SecurityConfig.class)
class AdminControllerSecurityTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UsuarioService usuarioService;

    @Test
    void deveNegarAcessoSemAutenticacao() throws Exception {
        mockMvc.perform(get("/api/admin/usuarios"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser(roles = "USER")
    void deveNegarAcessoParaUsuarioSemRoleAdmin() throws Exception {
        mockMvc.perform(get("/api/admin/usuarios"))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser(username = "admin", roles = "ADMIN")
    void devePermitirAcessoParaAdmin() throws Exception {
        when(usuarioService.listarTodos()).thenReturn(List.of());

        mockMvc.perform(get("/api/admin/usuarios"))
            .andExpect(status().isOk());
    }

    @Test
    void deveAutenticarComJwtValido() throws Exception {
        // Teste com SecurityMockMvcRequestPostProcessors
        mockMvc.perform(get("/api/admin/usuarios")
                .with(jwt().authorities(new SimpleGrantedAuthority("ROLE_ADMIN"))))
            .andExpect(status().isOk());
    }

    @Test
    @WithMockUser(username = "admin", roles = "ADMIN")
    void deveValidarCsrfEmMutacoes() throws Exception {
        mockMvc.perform(post("/api/admin/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"nome\":\"Novo Admin\"}")
                .with(csrf()))
            .andExpect(status().isCreated());
    }
}
```

### 5.5 Anotação customizada para contexto de segurança

```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithAdminUserSecurityContextFactory.class)
public @interface WithAdminUser {
    String username() default "admin";
}

public class WithAdminUserSecurityContextFactory
        implements WithSecurityContextFactory<WithAdminUser> {

    @Override
    public SecurityContext createSecurityContext(WithAdminUser annotation) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        UserDetails principal = User.withUsername(annotation.username())
            .password("password")
            .roles("ADMIN", "USER")
            .build();
        context.setAuthentication(
            new UsernamePasswordAuthenticationToken(principal, null, principal.getAuthorities()));
        return context;
    }
}

// Uso simplificado nos testes
@Test
@WithAdminUser
void deveListarUsuariosComoAdmin() throws Exception {
    mockMvc.perform(get("/api/admin/usuarios"))
        .andExpect(status().isOk());
}
```

---

## 6. REST Assured — Testes Fluentes de API

### 6.1 Conceito e configuração

REST Assured oferece uma DSL fluente no estilo Given/When/Then para testar APIs REST, com validação poderosa de JSON via JsonPath e XML via XmlPath.

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>spring-mock-mvc</artifactId>
    <scope>test</scope>
</dependency>
```

### 6.2 Testes com servidor real (@SpringBootTest)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProdutoRestAssuredTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setup() {
        RestAssured.port = port;
        RestAssured.basePath = "/api";
    }

    @Test
    void deveListarProdutos() {
        given()
            .accept(ContentType.JSON)
        .when()
            .get("/produtos")
        .then()
            .statusCode(200)
            .contentType(ContentType.JSON)
            .body("$", hasSize(greaterThan(0)))
            .body("[0].nome", notNullValue())
            .body("[0].preco", notNullValue());
    }

    @Test
    void deveCriarProduto() {
        String requestBody = """
            {
                "nome": "Teclado Mecânico",
                "preco": "349.90",
                "categoria": "PERIFERICOS",
                "estoque": 25
            }
            """;

        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("/produtos")
        .then()
            .statusCode(201)
            .header("Location", containsString("/api/produtos/"))
            .body("id", notNullValue())
            .body("nome", equalTo("Teclado Mecânico"))
            .body("categoria", equalTo("PERIFERICOS"));
    }

    @Test
    void deveBuscarProdutoPorIdComValidacaoDetalhada() {
        given()
            .pathParam("id", 1)
        .when()
            .get("/produtos/{id}")
        .then()
            .statusCode(200)
            .body("id", equalTo(1))
            .body("nome", not(emptyString()))
            .body("preco", matchesPattern("\\d+\\.\\d{2}"))
            .body("estoque", greaterThanOrEqualTo(0));
    }

    @Test
    void deveRetornar404ParaProdutoInexistente() {
        given()
            .pathParam("id", 999)
        .when()
            .get("/produtos/{id}")
        .then()
            .statusCode(404)
            .body("message", equalTo("Produto não encontrado"))
            .body("timestamp", notNullValue());
    }

    @Test
    void deveValidarPaginacao() {
        given()
            .queryParam("page", 0)
            .queryParam("size", 5)
            .queryParam("sort", "nome,asc")
        .when()
            .get("/produtos")
        .then()
            .statusCode(200)
            .body("content", hasSize(lessThanOrEqualTo(5)))
            .body("totalElements", greaterThanOrEqualTo(0))
            .body("pageable.pageSize", equalTo(5))
            .body("pageable.pageNumber", equalTo(0));
    }
}
```

### 6.3 REST Assured com autenticação

```java
@Test
void deveAcessarEndpointProtegidoComJWT() {
    // Obter token
    String token = given()
        .contentType(ContentType.JSON)
        .body("""
            {"email": "admin@mail.com", "password": "senha123"}
            """)
    .when()
        .post("/auth/login")
    .then()
        .statusCode(200)
        .extract().path("token");

    // Usar token
    given()
        .header("Authorization", "Bearer " + token)
    .when()
        .get("/admin/usuarios")
    .then()
        .statusCode(200)
        .body("$", hasSize(greaterThan(0)));
}

@Test
void deveNegarAcessoSemToken() {
    given()
    .when()
        .get("/admin/usuarios")
    .then()
        .statusCode(401);
}
```

### 6.4 REST Assured com Spring MockMvc (sem servidor)

```java
@WebMvcTest(ProdutoController.class)
class ProdutoRestAssuredMockMvcTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProdutoService produtoService;

    @BeforeEach
    void setup() {
        RestAssuredMockMvc.mockMvc(mockMvc);
    }

    @Test
    void deveListarProdutos() {
        when(produtoService.listarTodos()).thenReturn(List.of(
            new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS")));

        RestAssuredMockMvc.given()
            .accept(ContentType.JSON)
        .when()
            .get("/api/produtos")
        .then()
            .statusCode(200)
            .body("[0].nome", equalTo("Notebook"));
    }
}
```

### 6.5 Extraindo e reutilizando dados entre requisições

```java
@Test
void deveRealizarFluxoCompletoCRUD() {
    // Create — extrair ID
    int produtoId = given()
        .contentType(ContentType.JSON)
        .body("""
            {"nome": "Mouse", "preco": "89.90", "categoria": "PERIFERICOS", "estoque": 50}
            """)
    .when()
        .post("/produtos")
    .then()
        .statusCode(201)
        .extract().path("id");

    // Read
    given()
        .pathParam("id", produtoId)
    .when()
        .get("/produtos/{id}")
    .then()
        .statusCode(200)
        .body("nome", equalTo("Mouse"));

    // Update
    given()
        .contentType(ContentType.JSON)
        .pathParam("id", produtoId)
        .body("""
            {"nome": "Mouse Gamer", "preco": "149.90"}
            """)
    .when()
        .put("/produtos/{id}")
    .then()
        .statusCode(200)
        .body("nome", equalTo("Mouse Gamer"));

    // Delete
    given()
        .pathParam("id", produtoId)
    .when()
        .delete("/produtos/{id}")
    .then()
        .statusCode(204);
}
```

### 6.6 REST Assured vs MockMvc vs RestTestClient

| Característica | MockMvc | REST Assured | RestTestClient (Boot 4) |
|---------------|---------|-------------|------------------------|
| **Estilo** | Builder pattern | Given/When/Then (BDD) | Fluente encadeado |
| **Servidor** | Sem servidor (mock) | Com ou sem servidor | Com ou sem servidor |
| **JsonPath** | `jsonPath("$.nome")` | `body("nome", equalTo(...))` | `jsonPath("$.nome")` |
| **Logging** | Manual | `.log().all()` nativo | Manual |
| **Ecossistema** | Spring nativo | Independente de framework | Spring nativo |
| **Uso ideal** | Slice tests Spring | Testes de API expressivos | Spring Boot 4+ |

---

## 7. @DataJpaTest — Testes de Repositórios

### 7.1 Configuração básica

`@DataJpaTest` carrega apenas a camada JPA (EntityManager, Repositories), configura um banco in-memory e aplica rollback automático após cada teste:

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ProdutoRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private ProdutoRepository produtoRepository;

    @Test
    void deveSalvarERecuperarProduto() {
        var produto = new Produto("Notebook", new BigDecimal("3500.00"),
                                   Categoria.ELETRONICOS, 10);
        entityManager.persistAndFlush(produto);

        Optional<Produto> encontrado = produtoRepository.findById(produto.getId());

        assertTrue(encontrado.isPresent());
        assertEquals("Notebook", encontrado.get().getNome());
    }

    @Test
    void deveBuscarPorCategoria() {
        entityManager.persist(new Produto("Notebook", new BigDecimal("3500"), Categoria.ELETRONICOS, 5));
        entityManager.persist(new Produto("Mouse", new BigDecimal("60"), Categoria.PERIFERICOS, 20));
        entityManager.persist(new Produto("Monitor", new BigDecimal("1200"), Categoria.ELETRONICOS, 8));
        entityManager.flush();

        List<Produto> eletronicos = produtoRepository.findByCategoria(Categoria.ELETRONICOS);

        assertEquals(2, eletronicos.size());
        assertTrue(eletronicos.stream().allMatch(p -> p.getCategoria() == Categoria.ELETRONICOS));
    }
}
```

### 7.2 Testando queries customizadas (JPQL, Native, QueryDSL)

```java
@DataJpaTest
class PedidoRepositoryTest {

    @Autowired
    private TestEntityManager em;

    @Autowired
    private PedidoRepository pedidoRepository;

    @Test
    void deveBuscarPedidosPorPeriodo() {
        var cliente = em.persist(new Cliente("Maria", "maria@mail.com"));
        em.persist(criarPedido(cliente, LocalDate.of(2025, 1, 10), StatusPedido.CONFIRMADO));
        em.persist(criarPedido(cliente, LocalDate.of(2025, 1, 20), StatusPedido.ENVIADO));
        em.persist(criarPedido(cliente, LocalDate.of(2025, 2, 5), StatusPedido.PENDENTE));
        em.flush();

        List<Pedido> pedidosJaneiro = pedidoRepository.findByPeriodo(
            LocalDate.of(2025, 1, 1),
            LocalDate.of(2025, 1, 31)
        );

        assertEquals(2, pedidosJaneiro.size());
    }

    @Test
    void deveCalcularTotalVendasPorCliente() {
        var cliente = em.persist(new Cliente("João", "joao@mail.com"));
        em.persist(criarPedidoComTotal(cliente, new BigDecimal("100.00")));
        em.persist(criarPedidoComTotal(cliente, new BigDecimal("250.00")));
        em.flush();

        // Query customizada: @Query("SELECT SUM(p.total) FROM Pedido p WHERE p.cliente.id = :clienteId")
        BigDecimal total = pedidoRepository.calcularTotalPorCliente(cliente.getId());

        assertEquals(new BigDecimal("350.00"), total);
    }

    @Test
    void deveRetornarRelatorioDeVendas() {
        // Testando projeção (interface projection)
        var c1 = em.persist(new Cliente("Maria", "maria@mail.com"));
        var c2 = em.persist(new Cliente("João", "joao@mail.com"));
        em.persist(criarPedidoComTotal(c1, new BigDecimal("500.00")));
        em.persist(criarPedidoComTotal(c1, new BigDecimal("300.00")));
        em.persist(criarPedidoComTotal(c2, new BigDecimal("150.00")));
        em.flush();

        List<RelatorioVendaProjection> relatorio = pedidoRepository.gerarRelatorioVendas();

        assertEquals(2, relatorio.size());
        assertEquals("Maria", relatorio.get(0).getClienteNome());
        assertEquals(new BigDecimal("800.00"), relatorio.get(0).getTotalVendas());
    }
}
```

### 7.3 Testando Specifications (consultas dinâmicas)

```java
@DataJpaTest
class ProdutoSpecificationTest {

    @Autowired
    private ProdutoRepository produtoRepository;

    @Autowired
    private TestEntityManager em;

    @BeforeEach
    void setup() {
        em.persist(new Produto("Notebook Dell", new BigDecimal("4500"), Categoria.ELETRONICOS, 5));
        em.persist(new Produto("Mouse Logitech", new BigDecimal("80"), Categoria.PERIFERICOS, 50));
        em.persist(new Produto("Monitor Samsung", new BigDecimal("1800"), Categoria.ELETRONICOS, 10));
        em.flush();
    }

    @Test
    void deveFiltrarPorNomeECategoria() {
        Specification<Produto> spec = ProdutoSpecification.builder()
            .comNome("Note")
            .comCategoria(Categoria.ELETRONICOS)
            .build();

        List<Produto> resultado = produtoRepository.findAll(spec);

        assertEquals(1, resultado.size());
        assertEquals("Notebook Dell", resultado.get(0).getNome());
    }

    @Test
    void deveFiltrarPorFaixaDePreco() {
        Specification<Produto> spec = ProdutoSpecification.builder()
            .comPrecoEntre(new BigDecimal("100"), new BigDecimal("2000"))
            .build();

        List<Produto> resultado = produtoRepository.findAll(spec);

        assertEquals(1, resultado.size());
        assertEquals("Monitor Samsung", resultado.get(0).getNome());
    }
}
```

### 7.4 Testando auditoria (Auditing)

```java
@DataJpaTest
@EnableJpaAuditing
class AuditoriaTest {

    @Autowired
    private TestEntityManager em;

    @Autowired
    private ProdutoRepository produtoRepository;

    @Test
    void devePreencherCamposDeAuditoria() {
        var produto = new Produto("Teclado", new BigDecimal("200"), Categoria.PERIFERICOS, 15);
        Produto salvo = produtoRepository.saveAndFlush(produto);

        assertNotNull(salvo.getCreatedAt());
        assertNotNull(salvo.getUpdatedAt());

        salvo.setPreco(new BigDecimal("180"));
        produtoRepository.saveAndFlush(salvo);

        assertTrue(salvo.getUpdatedAt().isAfter(salvo.getCreatedAt()));
    }
}
```

---

## 8. Testes de Migrations — Flyway e Liquibase

### 8.1 Por que testar migrations

Migrations são código que altera o schema do banco de dados. Bugs em migrations podem corromper dados em produção de forma irreversível. Testes garantem que:

- Migrations executam sem erros na ordem correta
- O schema resultante é compatível com as entidades JPA
- Migrations podem ser revertidas (rollback)
- Dados existentes não são corrompidos

### 8.2 Testando Flyway Migrations

```java
@SpringBootTest
@Testcontainers
class FlywayMigrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private Flyway flyway;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    void todasMigrationsDevemExecutarComSucesso() {
        // Flyway já executou automaticamente via Spring Boot
        MigrationInfoService info = flyway.info();

        // Verificar que não há migrations pendentes
        assertEquals(0, info.pending().length,
            "Existem migrations pendentes: " + Arrays.toString(info.pending()));

        // Verificar que nenhuma falhou
        Arrays.stream(info.applied()).forEach(migration ->
            assertNotEquals("FAILED", migration.getState().name(),
                "Migration falhou: " + migration.getDescription()));
    }

    @Test
    void schemaDoBancoDeveSerCompativelComEntidades() {
        // Verifica que tabelas esperadas existem
        List<String> tabelas = jdbcTemplate.queryForList(
            "SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'",
            String.class);

        assertAll(
            () -> assertTrue(tabelas.contains("clientes"), "Tabela 'clientes' não encontrada"),
            () -> assertTrue(tabelas.contains("pedidos"), "Tabela 'pedidos' não encontrada"),
            () -> assertTrue(tabelas.contains("produtos"), "Tabela 'produtos' não encontrada")
        );
    }

    @Test
    void migrationDevePreservarDadosExistentes() {
        // Simula cenário: inserir dados como estavam antes da migration
        jdbcTemplate.execute("""
            INSERT INTO clientes (id, nome, email, created_at)
            VALUES (1, 'Maria', 'maria@mail.com', NOW())
            """);

        // Executar migration incremental (simula adição de nova coluna)
        flyway.migrate();

        // Verificar que dados existentes não foram afetados
        String nome = jdbcTemplate.queryForObject(
            "SELECT nome FROM clientes WHERE id = 1", String.class);
        assertEquals("Maria", nome);
    }
}
```

### 8.3 Testando migration específica isoladamente

```java
@Testcontainers
class MigrationV3AdicionarColunaTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Test
    void deveAdicionarColunaComValorDefault() {
        Flyway flyway = Flyway.configure()
            .dataSource(postgres.getJdbcUrl(), postgres.getUsername(), postgres.getPassword())
            .target(MigrationVersion.fromVersion("3"))  // Até a migration V3
            .load();

        flyway.migrate();

        JdbcTemplate jdbc = new JdbcTemplate(
            new DriverManagerDataSource(postgres.getJdbcUrl(),
                postgres.getUsername(), postgres.getPassword()));

        // Verificar que a nova coluna existe com valor default
        List<Map<String, Object>> colunas = jdbc.queryForList("""
            SELECT column_name, column_default, is_nullable
            FROM information_schema.columns
            WHERE table_name = 'pedidos' AND column_name = 'prioridade'
            """);

        assertFalse(colunas.isEmpty());
        assertEquals("'NORMAL'", colunas.get(0).get("column_default").toString().trim());
    }
}
```

### 8.4 Testando Liquibase Migrations

```java
@SpringBootTest
@Testcontainers
@TestPropertySource(properties = {
    "spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml"
})
class LiquibaseMigrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private SpringLiquibase liquibase;

    @Test
    void todasMigrationsDevemExecutarComSucesso() {
        // Verificar que changelogs foram aplicados
        Integer count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM databasechangelog", Integer.class);

        assertTrue(count > 0, "Nenhum changelog foi aplicado");
    }

    @Test
    void deveGerarRollbackSemErros() throws Exception {
        // Verificar que rollback scripts são válidos
        Database database = DatabaseFactory.getInstance()
            .findCorrectDatabaseImplementation(
                new JdbcConnection(dataSource.getConnection()));

        Liquibase liquibase = new Liquibase(
            "db/changelog/db.changelog-master.yaml",
            new ClassLoaderResourceAccessor(), database);

        // Dry-run do rollback — verifica SQL gerado sem executar
        StringWriter output = new StringWriter();
        liquibase.rollback(1, output);

        assertFalse(output.toString().isEmpty(),
            "Rollback SQL não deveria estar vazio");
    }
}
```

### 8.5 Validando schema JPA vs banco real

```java
@SpringBootTest
@Testcontainers
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=validate"  // Valida sem alterar
})
class SchemaValidationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Test
    void schemaDoFlywayDeveSerCompativelComEntidadesJPA() {
        // Se o contexto subiu sem erro, o schema é compatível
        // Se houver incompatibilidade, o Spring falha ao iniciar
        // com SchemaManagementException
        assertTrue(true, "Schema validado com sucesso — compatível com entidades JPA");
    }
}
```

---

## 9. Testes de Integração com @SpringBootTest

### 9.1 Teste de integração completo

`@SpringBootTest` carrega o contexto completo da aplicação:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class PedidoIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PedidoRepository pedidoRepository;

    @Autowired
    private ClienteRepository clienteRepository;

    @BeforeEach
    void setup() {
        pedidoRepository.deleteAll();
        clienteRepository.deleteAll();
    }

    @Test
    void deveRealizarFluxoCompletoDeCriacaoDePedido() {
        // 1. Criar cliente
        var clienteRequest = new ClienteDTO("Maria", "maria@test.com", "01310-100");
        ResponseEntity<Cliente> clienteResponse = restTemplate.postForEntity(
            "/api/clientes", clienteRequest, Cliente.class);

        assertEquals(HttpStatus.CREATED, clienteResponse.getStatusCode());
        Long clienteId = clienteResponse.getBody().getId();

        // 2. Criar pedido
        var pedidoRequest = new PedidoRequest(
            clienteId,
            List.of(new ItemPedido("PROD-001", 2, new BigDecimal("50.00")))
        );
        ResponseEntity<PedidoResponse> pedidoResponse = restTemplate.postForEntity(
            "/api/pedidos", pedidoRequest, PedidoResponse.class);

        assertEquals(HttpStatus.CREATED, pedidoResponse.getStatusCode());
        assertEquals(new BigDecimal("100.00"), pedidoResponse.getBody().getTotal());

        // 3. Verificar no banco
        List<Pedido> pedidos = pedidoRepository.findAll();
        assertEquals(1, pedidos.size());
        assertEquals(StatusPedido.PENDENTE, pedidos.get(0).getStatus());
    }

    @Test
    void deveRetornarErroAoCriarPedidoComClienteInexistente() {
        var request = new PedidoRequest(999L, List.of(new ItemPedido("A", 1, BigDecimal.TEN)));

        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/pedidos", request, ErrorResponse.class);

        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
    }
}
```

### 9.2 Profiles e configuração de teste

```yaml
# src/test/resources/application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate.format_sql: true
  mail:
    host: localhost
    port: 3025

logging:
  level:
    org.springframework.test: DEBUG
    org.hibernate.SQL: DEBUG
```

### 9.3 Testando publicação de eventos

```java
@SpringBootTest
class EventoIntegrationTest {

    @Autowired
    private PedidoService pedidoService;

    @MockBean
    private ApplicationEventPublisher eventPublisher;

    @Test
    void devePublicarEventoAoCriarPedido() {
        var request = criarPedidoRequest();

        pedidoService.criar(request);

        verify(eventPublisher).publishEvent(argThat(event ->
            event instanceof PedidoCriadoEvent &&
            ((PedidoCriadoEvent) event).getStatus() == StatusPedido.PENDENTE
        ));
    }
}
```

### 9.4 Testando Listeners de Eventos Spring (@EventListener)

#### 9.4.1 Listener síncrono — teste unitário com Mockito

```java
// O listener que será testado
@Component
public class PedidoEventListener {

    private final EstoqueService estoqueService;
    private final NotificacaoService notificacaoService;
    private final AuditoriaService auditoriaService;

    public PedidoEventListener(EstoqueService estoqueService,
                                NotificacaoService notificacaoService,
                                AuditoriaService auditoriaService) {
        this.estoqueService = estoqueService;
        this.notificacaoService = notificacaoService;
        this.auditoriaService = auditoriaService;
    }

    @EventListener
    public void onPedidoConfirmado(PedidoConfirmadoEvent evento) {
        estoqueService.reservar(evento.getItens());
        notificacaoService.notificarCliente(evento.getClienteId(), "Pedido confirmado!");
        auditoriaService.registrar("PEDIDO_CONFIRMADO", evento.getPedidoId());
    }

    @EventListener(condition = "#evento.valor.compareTo(T(java.math.BigDecimal).valueOf(1000)) > 0")
    public void onPedidoAltoValor(PedidoConfirmadoEvent evento) {
        auditoriaService.registrar("PEDIDO_ALTO_VALOR", evento.getPedidoId());
    }
}

// Teste unitário (sem Spring Context — rápido)
@ExtendWith(MockitoExtension.class)
class PedidoEventListenerTest {

    @Mock
    private EstoqueService estoqueService;

    @Mock
    private NotificacaoService notificacaoService;

    @Mock
    private AuditoriaService auditoriaService;

    @InjectMocks
    private PedidoEventListener listener;

    @Test
    void deveReservarEstoqueENotificarCliente() {
        var itens = List.of(new ItemPedido("PROD-001", 2), new ItemPedido("PROD-002", 1));
        var evento = new PedidoConfirmadoEvent(100L, 1L, itens, new BigDecimal("350.00"));

        listener.onPedidoConfirmado(evento);

        verify(estoqueService).reservar(itens);
        verify(notificacaoService).notificarCliente(1L, "Pedido confirmado!");
        verify(auditoriaService).registrar("PEDIDO_CONFIRMADO", 100L);
    }

    @Test
    void deveRegistrarAuditoriaParaPedidoAltoValor() {
        var evento = new PedidoConfirmadoEvent(200L, 1L, List.of(), new BigDecimal("5000.00"));

        listener.onPedidoAltoValor(evento);

        verify(auditoriaService).registrar("PEDIDO_ALTO_VALOR", 200L);
    }

    @Test
    void devePropararExcecaoQuandoEstoqueIndisponivel() {
        var evento = new PedidoConfirmadoEvent(100L, 1L,
            List.of(new ItemPedido("PROD-999", 100)), new BigDecimal("50000.00"));

        doThrow(new EstoqueInsuficienteException("PROD-999"))
            .when(estoqueService).reservar(anyList());

        assertThrows(EstoqueInsuficienteException.class,
            () -> listener.onPedidoConfirmado(evento));

        // Notificação e auditoria NÃO devem executar se o estoque falhou
        verify(notificacaoService, never()).notificarCliente(anyLong(), anyString());
    }
}
```

#### 9.4.2 Listener assíncrono — teste de integração

```java
// Listener assíncrono
@Component
public class PagamentoEventListener {

    private final RelatorioService relatorioService;
    private final EmailService emailService;

    @Async
    @EventListener
    public void onPagamentoAprovado(PagamentoAprovadoEvent evento) {
        relatorioService.atualizarResumoFinanceiro(evento.getValor());
        emailService.enviarRecibo(evento.getClienteEmail(), evento.getTransacaoId());
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onPagamentoCommitted(PagamentoAprovadoEvent evento) {
        emailService.enviarConfirmacaoFinal(evento.getClienteEmail());
    }
}

// Teste de integração — verifica que o evento chega ao listener via Spring
@SpringBootTest
class PagamentoEventListenerIntegrationTest {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @MockBean
    private RelatorioService relatorioService;

    @MockBean
    private EmailService emailService;

    @Test
    void deveProcessarEventoAssincronamente() {
        var evento = new PagamentoAprovadoEvent(
            "TXN-123", new BigDecimal("500.00"), "cliente@mail.com");

        eventPublisher.publishEvent(evento);

        // timeout() é essencial para listeners @Async
        verify(relatorioService, timeout(5000))
            .atualizarResumoFinanceiro(new BigDecimal("500.00"));
        verify(emailService, timeout(5000))
            .enviarRecibo("cliente@mail.com", "TXN-123");
    }

    @Test
    @Transactional
    void deveExecutarAposCommitDaTransacao() {
        var evento = new PagamentoAprovadoEvent(
            "TXN-456", new BigDecimal("100.00"), "user@mail.com");

        eventPublisher.publishEvent(evento);

        // @TransactionalEventListener só executa após o commit
        // Dentro de @Transactional no teste, o commit não ocorre (rollback)
        verify(emailService, never()).enviarConfirmacaoFinal(anyString());
    }
}
```

#### 9.4.3 Testando com ApplicationEvents (capture automática)

```java
// Spring Boot 3.1+ — captura automática de eventos publicados
@SpringBootTest
class PedidoServiceEventCaptureTest {

    @Autowired
    private PedidoService pedidoService;

    @Autowired
    private ApplicationEvents applicationEvents;

    @Test
    @RecordApplicationEvents
    void devePublicarEventoCorreto() {
        var request = criarPedidoRequest();

        pedidoService.criar(request);

        // Captura todos os eventos publicados do tipo específico
        long count = applicationEvents
            .stream(PedidoCriadoEvent.class)
            .filter(e -> e.getStatus() == StatusPedido.PENDENTE)
            .count();

        assertEquals(1, count);
    }

    @Test
    @RecordApplicationEvents
    void naoDevePublicarEventoEmCasoDeErro() {
        var requestInvalido = new PedidoRequest(999L, List.of());

        assertThrows(BusinessException.class,
            () -> pedidoService.criar(requestInvalido));

        long count = applicationEvents.stream(PedidoCriadoEvent.class).count();
        assertEquals(0, count);
    }
}
```

### 9.5 Testando Listeners JMS (ActiveMQ / Artemis)

#### 9.5.1 Configuração de dependências para testes JMS

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-jakarta-server</artifactId>
    <scope>test</scope>
</dependency>
<!-- Ou com Testcontainers -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <scope>test</scope>
</dependency>
```

#### 9.5.2 Listener JMS que será testado

```java
@Component
public class PedidoJmsListener {

    private final PedidoService pedidoService;
    private final AuditoriaService auditoriaService;
    private final JmsTemplate jmsTemplate;

    public PedidoJmsListener(PedidoService pedidoService,
                              AuditoriaService auditoriaService,
                              JmsTemplate jmsTemplate) {
        this.pedidoService = pedidoService;
        this.auditoriaService = auditoriaService;
        this.jmsTemplate = jmsTemplate;
    }

    @JmsListener(destination = "pedido.criado", containerFactory = "jmsListenerContainerFactory")
    public void onPedidoCriado(PedidoMessage mensagem) {
        try {
            pedidoService.processar(mensagem.getPedidoId());
            auditoriaService.registrar("PEDIDO_PROCESSADO", mensagem.getPedidoId());

            // Publica resposta na fila de confirmação
            var confirmacao = new ConfirmacaoMessage(mensagem.getPedidoId(), StatusProcessamento.SUCESSO);
            jmsTemplate.convertAndSend("pedido.confirmacao", confirmacao);
        } catch (Exception e) {
            var erro = new ConfirmacaoMessage(mensagem.getPedidoId(), StatusProcessamento.ERRO);
            jmsTemplate.convertAndSend("pedido.erro", erro);
            throw e;  // Re-throw para JMS retry/DLQ
        }
    }

    @JmsListener(destination = "estoque.atualizado")
    public void onEstoqueAtualizado(EstoqueMessage mensagem) {
        pedidoService.atualizarDisponibilidade(
            mensagem.getProdutoId(), mensagem.getQuantidade());
    }
}
```

#### 9.5.3 Teste unitário do listener JMS (sem broker)

```java
@ExtendWith(MockitoExtension.class)
class PedidoJmsListenerTest {

    @Mock
    private PedidoService pedidoService;

    @Mock
    private AuditoriaService auditoriaService;

    @Mock
    private JmsTemplate jmsTemplate;

    @InjectMocks
    private PedidoJmsListener listener;

    @Test
    void deveProcessarMensagemDePedidoComSucesso() {
        var mensagem = new PedidoMessage(100L, "cliente@mail.com", List.of("PROD-001"));

        listener.onPedidoCriado(mensagem);

        verify(pedidoService).processar(100L);
        verify(auditoriaService).registrar("PEDIDO_PROCESSADO", 100L);

        // Verifica que mensagem de confirmação foi enviada
        ArgumentCaptor<ConfirmacaoMessage> captor =
            ArgumentCaptor.forClass(ConfirmacaoMessage.class);
        verify(jmsTemplate).convertAndSend(eq("pedido.confirmacao"), captor.capture());

        ConfirmacaoMessage confirmacao = captor.getValue();
        assertEquals(100L, confirmacao.getPedidoId());
        assertEquals(StatusProcessamento.SUCESSO, confirmacao.getStatus());
    }

    @Test
    void deveEnviarMensagemDeErroQuandoProcessamentoFalha() {
        var mensagem = new PedidoMessage(200L, "user@mail.com", List.of("PROD-999"));
        doThrow(new BusinessException("Produto indisponível"))
            .when(pedidoService).processar(200L);

        assertThrows(BusinessException.class,
            () -> listener.onPedidoCriado(mensagem));

        // Verifica que mensagem de erro foi publicada na fila de erro
        ArgumentCaptor<ConfirmacaoMessage> captor =
            ArgumentCaptor.forClass(ConfirmacaoMessage.class);
        verify(jmsTemplate).convertAndSend(eq("pedido.erro"), captor.capture());
        assertEquals(StatusProcessamento.ERRO, captor.getValue().getStatus());

        // Auditoria NÃO deve ter sido registrada
        verify(auditoriaService, never()).registrar(anyString(), anyLong());
    }

    @Test
    void deveAtualizarDisponibilidadeDeEstoque() {
        var mensagem = new EstoqueMessage("PROD-001", 50);

        listener.onEstoqueAtualizado(mensagem);

        verify(pedidoService).atualizarDisponibilidade("PROD-001", 50);
    }
}
```

#### 9.5.4 Teste de integração JMS com broker embarcado (Artemis)

```java
@SpringBootTest
@TestPropertySource(properties = {
    "spring.artemis.mode=embedded",
    "spring.artemis.embedded.enabled=true"
})
class PedidoJmsIntegrationTest {

    @Autowired
    private JmsTemplate jmsTemplate;

    @MockBean
    private PedidoService pedidoService;

    @MockBean
    private AuditoriaService auditoriaService;

    @Test
    void deveReceberEProcessarMensagemDaFila() {
        var mensagem = new PedidoMessage(100L, "test@mail.com", List.of("PROD-001"));

        // Envia mensagem para a fila — o listener real consome
        jmsTemplate.convertAndSend("pedido.criado", mensagem);

        // Aguarda o listener processar (assíncrono)
        await().atMost(Duration.ofSeconds(10))
               .untilAsserted(() -> {
                   verify(pedidoService).processar(100L);
                   verify(auditoriaService).registrar("PEDIDO_PROCESSADO", 100L);
               });
    }

    @Test
    void devePublicarConfirmacaoAposProcessamento() {
        var mensagem = new PedidoMessage(300L, "user@mail.com", List.of("PROD-001"));

        jmsTemplate.convertAndSend("pedido.criado", mensagem);

        // Verifica que a mensagem de confirmação chegou na fila de saída
        await().atMost(Duration.ofSeconds(10))
               .untilAsserted(() -> {
                   ConfirmacaoMessage confirmacao = (ConfirmacaoMessage)
                       jmsTemplate.receiveAndConvert("pedido.confirmacao");
                   assertNotNull(confirmacao);
                   assertEquals(300L, confirmacao.getPedidoId());
                   assertEquals(StatusProcessamento.SUCESSO, confirmacao.getStatus());
               });
    }
}
```

#### 9.5.5 Teste de integração JMS com Testcontainers (ActiveMQ)

```java
@SpringBootTest
@Testcontainers
class JmsTestcontainersTest {

    @Container
    static GenericContainer<?> activemq = new GenericContainer<>("apache/activemq-artemis:2.37.0")
        .withExposedPorts(61616, 8161)
        .withEnv("ARTEMIS_USER", "admin")
        .withEnv("ARTEMIS_PASSWORD", "admin")
        .waitingFor(Wait.forLogMessage(".*AMQ241004.*", 1));

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.artemis.broker-url",
            () -> "tcp://localhost:" + activemq.getMappedPort(61616));
        registry.add("spring.artemis.user", () -> "admin");
        registry.add("spring.artemis.password", () -> "admin");
    }

    @Autowired
    private JmsTemplate jmsTemplate;

    @Autowired
    private PedidoRepository pedidoRepository;

    @Test
    void deveProcessarMensagemComBrokerReal() {
        // Preparar dados no banco
        var pedido = pedidoRepository.save(new Pedido(StatusPedido.PENDENTE, new BigDecimal("200.00")));

        var mensagem = new PedidoMessage(pedido.getId(), "test@mail.com", List.of("PROD-001"));
        jmsTemplate.convertAndSend("pedido.criado", mensagem);

        // Verificar no banco que o pedido foi processado
        await().atMost(Duration.ofSeconds(15))
               .untilAsserted(() -> {
                   Pedido atualizado = pedidoRepository.findById(pedido.getId()).orElseThrow();
                   assertEquals(StatusPedido.PROCESSADO, atualizado.getStatus());
               });
    }
}
```

### 9.6 Testando Listeners Kafka (@KafkaListener)

#### 9.6.1 Teste unitário do listener Kafka

```java
@ExtendWith(MockitoExtension.class)
class PagamentoKafkaListenerTest {

    @Mock
    private PagamentoService pagamentoService;

    @Mock
    private Acknowledgment acknowledgment;

    @InjectMocks
    private PagamentoKafkaListener listener;

    @Test
    void deveProcessarEventoDePagamento() {
        var evento = new PagamentoEvent("TXN-001", 100L, new BigDecimal("250.00"),
            StatusPagamento.APROVADO, Instant.now());

        listener.onPagamento(evento, acknowledgment);

        verify(pagamentoService).confirmar("TXN-001", 100L);
        verify(acknowledgment).acknowledge();  // Commit manual do offset
    }

    @Test
    void naoDeveCommitarOffsetEmCasoDeErro() {
        var evento = new PagamentoEvent("TXN-002", 200L, new BigDecimal("100.00"),
            StatusPagamento.APROVADO, Instant.now());
        doThrow(new RuntimeException("Falha ao processar"))
            .when(pagamentoService).confirmar("TXN-002", 200L);

        assertThrows(RuntimeException.class,
            () -> listener.onPagamento(evento, acknowledgment));

        verify(acknowledgment, never()).acknowledge();  // Offset NÃO commitado
    }
}
```

#### 9.6.2 Teste de integração Kafka com EmbeddedKafka

```java
@SpringBootTest
@EmbeddedKafka(
    partitions = 1,
    topics = {"pagamento-events", "notificacao-events"},
    brokerProperties = {"listeners=PLAINTEXT://localhost:9092"}
)
@TestPropertySource(properties = {
    "spring.kafka.bootstrap-servers=${spring.embedded.kafka.brokers}",
    "spring.kafka.consumer.auto-offset-reset=earliest"
})
class KafkaListenerIntegrationTest {

    @Autowired
    private KafkaTemplate<String, PagamentoEvent> kafkaTemplate;

    @Autowired
    private PedidoRepository pedidoRepository;

    @Test
    void deveProcessarEventoDoKafka() {
        var pedido = pedidoRepository.save(new Pedido(StatusPedido.PAGO, new BigDecimal("300.00")));
        var evento = new PagamentoEvent("TXN-100", pedido.getId(),
            new BigDecimal("300.00"), StatusPagamento.APROVADO, Instant.now());

        kafkaTemplate.send("pagamento-events", evento);

        await().atMost(Duration.ofSeconds(15))
               .untilAsserted(() -> {
                   Pedido atualizado = pedidoRepository.findById(pedido.getId()).orElseThrow();
                   assertEquals(StatusPedido.CONFIRMADO, atualizado.getStatus());
               });
    }
}
```

#### 9.6.3 Teste de integração Kafka com Testcontainers

```java
@SpringBootTest
@Testcontainers
class KafkaTestcontainersTest {

    @Container
    @ServiceConnection
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.6.0"))
        .withKraft();  // Kafka sem Zookeeper

    @Autowired
    private KafkaTemplate<String, PagamentoEvent> kafkaTemplate;

    @MockBean
    private NotificacaoService notificacaoService;

    @Test
    void deveConsumeEPublicarRespostaNoKafkaReal() {
        var evento = new PagamentoEvent("TXN-500", 1L,
            new BigDecimal("1500.00"), StatusPagamento.APROVADO, Instant.now());

        kafkaTemplate.send("pagamento-events", evento);

        await().atMost(Duration.ofSeconds(20))
               .untilAsserted(() ->
                   verify(notificacaoService).notificarPagamentoConfirmado(1L, "TXN-500")
               );
    }
}
```

### 9.7 Padrões e boas práticas para testes de Listeners

| Prática | Descrição |
|---------|-----------|
| **Unitário primeiro** | Teste a lógica do listener com Mockito, sem broker — é rápido e cobre a maioria dos cenários |
| **`timeout()` para async** | Use `verify(mock, timeout(5000))` ou Awaitility para listeners assíncronos |
| **Testar idempotência** | Envie a mesma mensagem 2x e verifique que o resultado é o mesmo |
| **Testar DLQ** | Simule erro e verifique que a mensagem vai para a Dead Letter Queue |
| **Embedded para CI** | Use `@EmbeddedKafka` ou Artemis embarcado para CI rápido |
| **Testcontainers para fidelidade** | Use Testcontainers quando o comportamento do broker real importa |
| **Capturar ArgumentCaptor** | Use `ArgumentCaptor` para verificar mensagens publicadas em filas de resposta |
| **Não testar o framework** | Não teste se `@JmsListener` / `@KafkaListener` registra o listener — teste a lógica dentro dele |

### 9.8 Testando Scheduling e Async

```java
@SpringBootTest
@EnableAsync
class AsyncServiceTest {

    @Autowired
    private RelatorioService relatorioService;

    @Test
    void deveGerarRelatorioAssincrono() throws Exception {
        CompletableFuture<Relatorio> futuro = relatorioService.gerarAsync(
            LocalDate.of(2025, 1, 1),
            LocalDate.of(2025, 1, 31)
        );

        Relatorio relatorio = futuro.get(10, TimeUnit.SECONDS);

        assertNotNull(relatorio);
        assertFalse(relatorio.getDados().isEmpty());
    }
}

@SpringBootTest
class ScheduledTaskTest {

    @SpyBean
    private LimpezaService limpezaService;

    @Test
    void deveLimparRegistrosExpirados() {
        // Aguarda execução do @Scheduled
        await().atMost(Duration.ofSeconds(65))
               .untilAsserted(() ->
                   verify(limpezaService, atLeastOnce()).limparExpirados()
               );
    }
}
```

---

## 10. WireMock — Mock de Serviços HTTP Externos

### 10.1 Conceito

WireMock cria um servidor HTTP falso que simula APIs externas durante testes de integração — mais realista que `MockRestServiceServer` pois intercepta chamadas HTTP reais (não apenas o client mockado).

```
┌─────────────┐       HTTP real        ┌──────────────┐
│  Sua App    │ ──────────────────────► │  WireMock    │
│  (Service)  │ ◄────────────────────── │  Server      │
└─────────────┘     Resposta mockada   │  (localhost)  │
                                        └──────────────┘
```

### 10.2 Configuração

```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock-standalone</artifactId>
    <version>3.10.0</version>
    <scope>test</scope>
</dependency>
<!-- Ou com Spring Boot starter -->
<dependency>
    <groupId>org.wiremock.integrations</groupId>
    <artifactId>wiremock-spring-boot</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
```

### 10.3 Uso básico com JUnit 5

```java
@WireMockTest(httpPort = 8089)
class CepServiceWireMockTest {

    @Autowired
    private CepService cepService;  // Configurado para apontar para localhost:8089

    @Test
    void deveBuscarEnderecoPorCep() {
        stubFor(get(urlPathEqualTo("/ws/01310100/json"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {
                        "cep": "01310-100",
                        "logradouro": "Avenida Paulista",
                        "bairro": "Bela Vista",
                        "localidade": "São Paulo",
                        "uf": "SP"
                    }
                    """)));

        Endereco endereco = cepService.buscar("01310-100");

        assertEquals("Avenida Paulista", endereco.getLogradouro());
        assertEquals("SP", endereco.getUf());

        // Verificar que a requisição foi feita
        verify(getRequestedFor(urlPathEqualTo("/ws/01310100/json")));
    }

    @Test
    void deveTratarTimeoutDoServicoExterno() {
        stubFor(get(urlPathMatching("/ws/.*/json"))
            .willReturn(aResponse()
                .withFixedDelay(5000)  // Simula timeout
                .withStatus(200)));

        assertThrows(IntegracaoException.class,
            () -> cepService.buscar("01310-100"));
    }

    @Test
    void deveTratarErro500DoServicoExterno() {
        stubFor(get(urlPathMatching("/ws/.*/json"))
            .willReturn(aResponse()
                .withStatus(500)
                .withBody("Internal Server Error")));

        assertThrows(IntegracaoException.class,
            () -> cepService.buscar("01310-100"));
    }
}
```

### 10.4 Cenários avançados

```java
@WireMockTest(httpPort = 8089)
class PagamentoGatewayWireMockTest {

    @Test
    void deveRealizarRetryAposFalhaTemporaria() {
        // Primeira chamada: erro 503
        stubFor(post(urlEqualTo("/api/cobrar"))
            .inScenario("retry")
            .whenScenarioStateIs(Scenario.STARTED)
            .willReturn(aResponse().withStatus(503))
            .willSetStateTo("segunda-tentativa"));

        // Segunda chamada: sucesso
        stubFor(post(urlEqualTo("/api/cobrar"))
            .inScenario("retry")
            .whenScenarioStateIs("segunda-tentativa")
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {"transacaoId": "TXN-001", "status": "APPROVED"}
                    """)));

        var resultado = gatewayService.cobrarComRetry(new CobrancaRequest(BigDecimal.TEN));

        assertEquals("TXN-001", resultado.getTransacaoId());
        verify(2, postRequestedFor(urlEqualTo("/api/cobrar")));
    }

    @Test
    void deveEnviarHeadersCorretos() {
        stubFor(post(urlEqualTo("/api/cobrar"))
            .withHeader("Authorization", matching("Bearer .*"))
            .withHeader("X-Idempotency-Key", matching(".+"))
            .withRequestBody(matchingJsonPath("$.valor"))
            .willReturn(okJson("""
                {"transacaoId": "TXN-002", "status": "APPROVED"}
                """)));

        gatewayService.cobrar(new CobrancaRequest(new BigDecimal("100.00")));

        verify(postRequestedFor(urlEqualTo("/api/cobrar"))
            .withHeader("Authorization", containing("Bearer"))
            .withHeader("Content-Type", equalTo("application/json"))
            .withRequestBody(matchingJsonPath("$.valor", equalTo("100.00"))));
    }

    @Test
    void deveSimularRespostaLentaParaTestarCircuitBreaker() {
        stubFor(post(urlEqualTo("/api/cobrar"))
            .willReturn(aResponse()
                .withStatus(200)
                .withFixedDelay(10000)));  // 10 segundos

        // Circuit breaker deve abrir após timeout
        assertThrows(CircuitBreakerException.class,
            () -> gatewayService.cobrar(new CobrancaRequest(BigDecimal.TEN)));
    }
}
```

### 10.5 WireMock com Spring Boot (integração automática)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@WireMockTest(httpPort = 8089)
@TestPropertySource(properties = {
    "servico.cep.url=http://localhost:8089",
    "servico.pagamento.url=http://localhost:8089"
})
class IntegracaoExternaWireMockTest {

    @Autowired
    private PedidoService pedidoService;

    @Test
    void deveCriarPedidoComValidacaoDeCepExterna() {
        // Mock do serviço de CEP
        stubFor(get(urlPathEqualTo("/ws/01310100/json"))
            .willReturn(okJson("""
                {"cep":"01310-100","logradouro":"Av Paulista","localidade":"São Paulo","uf":"SP"}
                """)));

        // Mock do gateway de pagamento
        stubFor(post(urlEqualTo("/api/cobrar"))
            .willReturn(okJson("""
                {"transacaoId":"TXN-100","status":"APPROVED"}
                """)));

        var request = new PedidoRequest(1L, "01310-100",
            List.of(new ItemPedido("PROD-001", 2)));

        PedidoResponse response = pedidoService.criarComPagamento(request);

        assertNotNull(response.getId());
        assertEquals(StatusPedido.PAGO, response.getStatus());
    }
}
```

### 10.6 WireMock vs MockRestServiceServer

| Característica | MockRestServiceServer | WireMock |
|---------------|----------------------|----------|
| **Nível** | Mock do RestClient/RestTemplate | Servidor HTTP real |
| **Verificação** | No nível do client Java | No nível HTTP (headers, body, URL) |
| **Cenários** | Simples (req/resp linear) | Complexos (retry, cenários stateful, delays) |
| **Fidelidade** | Média (pula camada HTTP) | Alta (HTTP real) |
| **Performance** | Rápido (sem rede) | Ligeiramente mais lento |
| **Uso ideal** | Testes unitários do client | Testes de integração, circuit breaker |

---

## 11. RestClient e RestTestClient (Spring Boot 4)

### 11.1 RestClient (Spring Boot 3.2+)

O `RestClient` foi introduzido no Spring Framework 6.1 como uma API fluente e síncrona para chamadas HTTP, substituindo gradualmente o `RestTemplate`:

```java
@Service
public class ProdutoExternoService {

    private final RestClient restClient;

    public ProdutoExternoService(RestClient.Builder builder) {
        this.restClient = builder
            .baseUrl("https://api.fornecedor.com")
            .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }

    public ProdutoExterno buscarPorSku(String sku) {
        return restClient.get()
            .uri("/produtos/{sku}", sku)
            .retrieve()
            .body(ProdutoExterno.class);
    }

    public List<ProdutoExterno> listarPorCategoria(String categoria) {
        return restClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/produtos")
                .queryParam("categoria", categoria)
                .build())
            .retrieve()
            .body(new ParameterizedTypeReference<>() {});
    }

    public ProdutoExterno criar(ProdutoExternoRequest request) {
        return restClient.post()
            .uri("/produtos")
            .contentType(MediaType.APPLICATION_JSON)
            .body(request)
            .retrieve()
            .body(ProdutoExterno.class);
    }
}
```

### 11.2 Testando RestClient com MockRestServiceServer

```java
@ExtendWith(MockitoExtension.class)
class ProdutoExternoServiceTest {

    private ProdutoExternoService service;
    private MockRestServiceServer mockServer;

    @BeforeEach
    void setup() {
        RestClient.Builder builder = RestClient.builder();
        mockServer = MockRestServiceServer.bindTo(builder).build();
        service = new ProdutoExternoService(builder);
    }

    @Test
    void deveBuscarProdutoPorSku() {
        String responseJson = """
            {
                "sku": "SKU-001",
                "nome": "Widget Premium",
                "preco": 29.99
            }
            """;

        mockServer.expect(requestTo("/produtos/SKU-001"))
            .andExpect(method(HttpMethod.GET))
            .andRespond(withSuccess(responseJson, MediaType.APPLICATION_JSON));

        ProdutoExterno produto = service.buscarPorSku("SKU-001");

        assertEquals("Widget Premium", produto.getNome());
        assertEquals(new BigDecimal("29.99"), produto.getPreco());
        mockServer.verify();
    }

    @Test
    void deveTratarErro404() {
        mockServer.expect(requestTo("/produtos/SKU-999"))
            .andRespond(withResourceNotFound());

        assertThrows(ResourceNotFoundException.class,
            () -> service.buscarPorSku("SKU-999"));
    }

    @Test
    void deveTratarErro500() {
        mockServer.expect(requestTo("/produtos/SKU-001"))
            .andRespond(withServerError());

        assertThrows(IntegracaoException.class,
            () -> service.buscarPorSku("SKU-001"));
    }
}
```

### 11.3 Spring Boot 4 — RestTestClient e novidades

O Spring Boot 4 (baseado no Spring Framework 7 e JUnit 6) introduz o `RestTestClient` como **substituto moderno do `TestRestTemplate`** (que será depreciado). Ele oferece uma API fluente unificada que funciona tanto com `MockMvc` quanto com servidor real.

**Mudança importante:** no Spring Boot 4, os clientes HTTP de teste **não são mais auto-configurados**. É necessário usar `@AutoConfigureRestTestClient` explicitamente:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class ProdutoApiTest {

    @Autowired
    private RestTestClient restTestClient;

    @Test
    void deveListarProdutos() {
        restTestClient.get()
            .uri("/api/produtos")
            .accept(MediaType.APPLICATION_JSON)
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$").isArray()
            .jsonPath("$[0].nome").isNotEmpty();
    }

    @Test
    void deveCriarProduto() {
        var request = new CriarProdutoDTO("Teclado", "149.90", "PERIFERICOS", 30);

        restTestClient.post()
            .uri("/api/produtos")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(request)
            .exchange()
            .expectStatus().isCreated()
            .expectHeader().exists("Location")
            .expectBody(ProdutoDTO.class)
            .value(produto -> {
                assertNotNull(produto.id());
                assertEquals("Teclado", produto.nome());
            });
    }

    @Test
    void deveRetornar404ParaProdutoInexistente() {
        restTestClient.get()
            .uri("/api/produtos/{id}", 999)
            .exchange()
            .expectStatus().isNotFound()
            .expectBody()
            .jsonPath("$.message").isEqualTo("Produto não encontrado");
    }

    @Test
    void deveAtualizarProduto() {
        var update = new AtualizarProdutoDTO("Teclado Mecânico", "249.90");

        restTestClient.put()
            .uri("/api/produtos/{id}", 1)
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(update)
            .exchange()
            .expectStatus().isOk()
            .expectBody(ProdutoDTO.class)
            .value(p -> assertEquals("Teclado Mecânico", p.nome()));
    }
}
```

### 11.4 RestTestClient com AssertJ (Spring Boot 4)

O `RestTestClient` oferece duas formas de verificar respostas — assertions encadeadas (built-in) e integração com **AssertJ**:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class ProdutoAssertJTest {

    @Autowired
    private RestTestClient restTestClient;

    @Test
    void deveVerificarComAssertJ() {
        restTestClient.get()
            .uri("/api/produtos/1")
            .exchange()
            .assertThat()  // Entra no modo AssertJ
            .hasStatusOk()
            .bodyJson()
            .extractingPath("$.nome")
            .asString()
            .isEqualTo("Notebook");
    }

    @Test
    void deveVerificarListaComAssertJ() {
        restTestClient.get()
            .uri("/api/produtos")
            .exchange()
            .assertThat()
            .hasStatusOk()
            .bodyJson()
            .extractingPath("$")
            .asArray()
            .hasSizeGreaterThan(0);
    }
}
```

### 11.5 RestTestClient com MockMvc (sem servidor real)

O `RestTestClient` também pode operar sobre o `MockMvc`, permitindo a mesma API fluente em slice tests:

```java
@WebMvcTest(ProdutoController.class)
@AutoConfigureRestTestClient
class ProdutoControllerRestTestClientTest {

    @Autowired
    private RestTestClient restTestClient;  // Opera sobre MockMvc internamente

    @MockBean
    private ProdutoService produtoService;

    @Test
    void deveListarProdutos() {
        when(produtoService.listarTodos()).thenReturn(List.of(
            new ProdutoDTO(1L, "Notebook", "R$ 3.500,00", "ELETRONICOS")));

        restTestClient.get()
            .uri("/api/produtos")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(ProdutoDTO.class)
            .hasSize(1);
    }
}
```

### 11.6 Pausa automática de contextos (Spring Framework 7)

Uma das melhorias mais impactantes do Spring Framework 7 é a **pausa automática de contextos em cache**: contextos de aplicação que não estão sendo usados ativamente são pausados e reiniciados automaticamente quando necessário. Isso reduz significativamente o consumo de memória em suítes de teste com muitas classes `@SpringBootTest`.

### 11.7 Comparação: TestRestTemplate vs WebTestClient vs RestTestClient

| Característica | TestRestTemplate | WebTestClient | RestTestClient (Boot 4) |
|---------------|-----------------|---------------|------------------------|
| **Estilo** | Imperativo | Reativo/Fluente | Fluente (síncrono) |
| **Stack** | Servlet (MVC) | Servlet e Reativo | Servlet e Reativo |
| **API** | `getForEntity()`, `postForEntity()` | `get().uri().exchange()` | `get().uri().exchange()` |
| **Assertions** | Manual com `assertEquals` | `.expectStatus()`, `.expectBody()` | `.expectStatus()`, `.expectBody()`, **AssertJ** |
| **MockMvc** | Não | Sim (desde Boot 3.x) | **Sim** (mesma API, sem servidor) |
| **Auto-configuração** | Automática (Boot 3.x) | Automática (Boot 3.x) | **`@AutoConfigureRestTestClient`** (explícita) |
| **Recomendação** | Legacy / Boot 2.x–3.x | Boot 3.x (reativo) | **Boot 4+ (preferencial)** |

### 11.8 Migrando de TestRestTemplate para RestTestClient

```java
// ❌ Antes (TestRestTemplate — Spring Boot 3.x)
@Test
void antigo_deveListarProdutos() {
    ResponseEntity<List<ProdutoDTO>> response = restTemplate.exchange(
        "/api/produtos",
        HttpMethod.GET,
        null,
        new ParameterizedTypeReference<List<ProdutoDTO>>() {}
    );
    assertEquals(HttpStatus.OK, response.getStatusCode());
    assertFalse(response.getBody().isEmpty());
}

// ✅ Depois (RestTestClient — Spring Boot 4)
@Test
void novo_deveListarProdutos() {
    restTestClient.get()
        .uri("/api/produtos")
        .exchange()
        .expectStatus().isOk()
        .expectBodyList(ProdutoDTO.class)
        .hasSize(greaterThan(0));
}
```

---

## 12. Testcontainers — Infraestrutura Real nos Testes

### 12.1 Conceito e arquitetura

Testcontainers sobe containers Docker reais durante a execução dos testes, garantindo que a aplicação seja testada contra a mesma infraestrutura usada em produção:

```
┌────────────────────────────────────────────┐
│              JVM (Testes)                   │
│  ┌────────────────────────────────────┐    │
│  │         Teste de Integração        │    │
│  │    @SpringBootTest + Testcontainers│    │
│  └────────────┬───────────────────────┘    │
│               │ JDBC / HTTP / AMQP         │
├───────────────┼────────────────────────────┤
│    Docker     │                             │
│  ┌──────────┐ │ ┌──────────┐ ┌──────────┐  │
│  │PostgreSQL│ │ │  Redis   │ │  Kafka   │  │
│  │  16.x    │◄┘ │  7.x    │ │  3.x     │  │
│  └──────────┘   └──────────┘ └──────────┘  │
└────────────────────────────────────────────┘
```

### 12.2 Configuração com Spring Boot 3.1+ (@ServiceConnection)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class PedidoIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @Container
    @ServiceConnection
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PedidoRepository pedidoRepository;

    @Test
    void deveSalvarPedidoNoBancoReal() {
        var request = new PedidoRequest(/* ... */);

        ResponseEntity<PedidoResponse> response = restTemplate.postForEntity(
            "/api/pedidos", request, PedidoResponse.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());

        // Verificação direta no PostgreSQL real
        List<Pedido> pedidos = pedidoRepository.findAll();
        assertEquals(1, pedidos.size());
    }
}
```

### 12.3 Classe base reutilizável com Testcontainers

```java
// Classe base que gerencia o ciclo de vida dos containers
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("integration-test")
public abstract class AbstractIntegrationTest {

    @Container
    @ServiceConnection
    protected static final PostgreSQLContainer<?> POSTGRES =
        new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("integration_test")
            .withUsername("test")
            .withPassword("test")
            .withReuse(true);  // Reutiliza container entre execuções (dev local)

    @Container
    @ServiceConnection
    protected static final GenericContainer<?> REDIS =
        new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379)
            .withReuse(true);

    @Autowired
    protected TestRestTemplate restTemplate;

    @Autowired
    protected JdbcTemplate jdbcTemplate;

    @BeforeEach
    void limparDados() {
        jdbcTemplate.execute("TRUNCATE TABLE pedidos, clientes, produtos RESTART IDENTITY CASCADE");
    }
}

// Testes herdam a infraestrutura
class ClienteIntegrationTest extends AbstractIntegrationTest {

    @Test
    void deveCriarCliente() {
        var request = new ClienteDTO("Maria", "maria@test.com", "01310-100");

        ResponseEntity<Cliente> response = restTemplate.postForEntity(
            "/api/clientes", request, Cliente.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
    }
}
```

### 12.4 Testcontainers com Kafka

```java
@SpringBootTest
@Testcontainers
class MensageriaIntegrationTest {

    @Container
    @ServiceConnection
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.6.0"));

    @Autowired
    private KafkaTemplate<String, PedidoEvent> kafkaTemplate;

    @Autowired
    private PedidoEventConsumer consumer;

    @Test
    void deveProcessarEventoDePedido() {
        var evento = new PedidoEvent(1L, StatusPedido.CONFIRMADO, Instant.now());

        kafkaTemplate.send("pedidos-topic", evento);

        await().atMost(Duration.ofSeconds(10))
               .untilAsserted(() ->
                   verify(consumer.getProcessados()).contains(1L)
               );
    }
}
```

### 12.5 Testcontainers com MongoDB, MySQL e outros

```java
// MongoDB
@Container
@ServiceConnection
static MongoDBContainer mongo = new MongoDBContainer("mongo:7.0");

// MySQL
@Container
@ServiceConnection
static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
    .withDatabaseName("testdb")
    .withUsername("test")
    .withPassword("test");

// RabbitMQ
@Container
@ServiceConnection
static RabbitMQContainer rabbit = new RabbitMQContainer("rabbitmq:3.13-management");

// LocalStack (AWS S3, SQS, etc.)
@Container
static LocalStackContainer localstack = new LocalStackContainer(
        DockerImageName.parse("localstack/localstack:3.0"))
    .withServices(LocalStackContainer.Service.S3, LocalStackContainer.Service.SQS);
```

### 12.6 Inicialização com scripts SQL

```java
@Container
@ServiceConnection
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
    .withDatabaseName("testdb")
    .withInitScript("sql/init-test-data.sql");  // src/test/resources/sql/init-test-data.sql
```

```sql
-- src/test/resources/sql/init-test-data.sql
CREATE TABLE IF NOT EXISTS categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

INSERT INTO categorias (nome) VALUES ('ELETRONICOS'), ('PERIFERICOS'), ('SOFTWARE');
```

---

## 13. Contract Testing — Pact e Spring Cloud Contract

### 13.1 Conceito

Contract Testing garante que a comunicação entre microsserviços esteja em conformidade — o **consumer** (quem consome a API) e o **producer** (quem fornece a API) concordam sobre o formato das requisições e respostas, sem precisar subir ambos ao mesmo tempo.

```
┌──────────────┐                    ┌──────────────┐
│   Consumer   │                    │   Producer   │
│  (Frontend/  │◄── Contrato ──►   │  (Backend/   │
│   outro MS)  │                    │   API)       │
└──────┬───────┘                    └──────┬───────┘
       │                                    │
       ▼                                    ▼
  Gera contrato                     Valida contrato
  a partir dos                      contra implementação
  testes do consumer                real
```

| Abordagem | Ferramenta | Direção |
|-----------|-----------|---------|
| **Consumer-Driven** | Pact | Consumer define o contrato, producer valida |
| **Producer-Driven** | Spring Cloud Contract | Producer define o contrato, gera stubs para consumers |

### 13.2 Spring Cloud Contract — Producer Side

```xml
<!-- Producer -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <scope>test</scope>
</dependency>
```

```groovy
// src/test/resources/contracts/produto/buscar_produto.groovy
Contract.make {
    description "Deve retornar produto por ID"
    request {
        method GET()
        url "/api/produtos/1"
        headers {
            accept applicationJson()
        }
    }
    response {
        status OK()
        headers {
            contentType applicationJson()
        }
        body([
            id: 1,
            nome: "Notebook",
            preco: "3500.00",
            categoria: "ELETRONICOS"
        ])
    }
}
```

```yaml
# Alternativa em YAML: src/test/resources/contracts/produto/buscar_produto.yml
description: Deve retornar produto por ID
request:
  method: GET
  url: /api/produtos/1
  headers:
    Accept: application/json
response:
  status: 200
  headers:
    Content-Type: application/json
  body:
    id: 1
    nome: Notebook
    preco: "3500.00"
    categoria: ELETRONICOS
```

```java
// Classe base para testes gerados automaticamente pelo plugin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public abstract class ContractBaseTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProdutoService produtoService;

    @BeforeEach
    void setup() {
        RestAssuredMockMvc.mockMvc(mockMvc);

        when(produtoService.buscarPorId(1L))
            .thenReturn(new ProdutoDTO(1L, "Notebook", "3500.00", "ELETRONICOS"));
    }
}
```

```xml
<!-- Plugin Maven para gerar testes e stubs -->
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
        <baseClassForTests>com.example.ContractBaseTest</baseClassForTests>
        <testFramework>JUNIT5</testFramework>
    </configuration>
</plugin>
```

### 13.3 Spring Cloud Contract — Consumer Side (Stub Runner)

```xml
<!-- Consumer -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@AutoConfigureStubRunner(
    ids = "com.example:produto-service:+:stubs:8090",
    stubsMode = StubRunnerProperties.StubsMode.LOCAL
)
class ProdutoClientContractTest {

    @Autowired
    private ProdutoClient produtoClient;  // Consome a API do producer

    @Test
    void deveBuscarProdutoUsandoStub() {
        // O Stub Runner sobe o WireMock na porta 8090 com os stubs gerados
        ProdutoDTO produto = produtoClient.buscarPorId(1L);

        assertNotNull(produto);
        assertEquals("Notebook", produto.nome());
        assertEquals("3500.00", produto.preco());
    }
}
```

### 13.4 Pact — Consumer-Driven Contract

```xml
<!-- Consumer -->
<dependency>
    <groupId>au.com.dius.pact.consumer</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.14</version>
    <scope>test</scope>
</dependency>
```

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "ProdutoService", port = "8090")
class ProdutoConsumerPactTest {

    @Pact(provider = "ProdutoService", consumer = "FrontendApp")
    V4Pact buscarProdutoPact(PactDslWithProvider builder) {
        return builder
            .given("produto com ID 1 existe")
            .uponReceiving("requisição para buscar produto por ID")
                .method("GET")
                .path("/api/produtos/1")
                .headers("Accept", "application/json")
            .willRespondWith()
                .status(200)
                .headers(Map.of("Content-Type", "application/json"))
                .body(newJsonBody(body -> {
                    body.integerType("id", 1);
                    body.stringType("nome", "Notebook");
                    body.stringMatcher("preco", "\\d+\\.\\d{2}", "3500.00");
                    body.stringType("categoria", "ELETRONICOS");
                }).build())
            .toPact(V4Pact.class);
    }

    @Test
    @PactTestFor(pactMethod = "buscarProdutoPact")
    void deveConsumirAPIDeProduto(MockServer mockServer) {
        RestClient client = RestClient.builder()
            .baseUrl(mockServer.getUrl())
            .build();

        ProdutoDTO produto = client.get()
            .uri("/api/produtos/1")
            .retrieve()
            .body(ProdutoDTO.class);

        assertNotNull(produto);
        assertEquals("Notebook", produto.nome());
    }
}
```

```java
// Producer — verificação do contrato Pact
@Provider("ProdutoService")
@PactBroker(url = "https://pact-broker.example.com")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProdutoProviderPactTest {

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verificarPact(PactVerificationContext context) {
        context.verifyInteraction();
    }

    @State("produto com ID 1 existe")
    void produtoExiste() {
        // Setup: garantir que o produto existe no banco para o teste
        produtoRepository.save(new Produto(1L, "Notebook",
            new BigDecimal("3500.00"), Categoria.ELETRONICOS, 10));
    }
}
```

### 13.5 Quando usar cada abordagem

| Cenário | Abordagem recomendada |
|---------|----------------------|
| Equipe controla producer e consumer | Spring Cloud Contract |
| Consumer é de outra equipe/empresa | Pact (consumer-driven) |
| Múltiplos consumers para mesma API | Pact (cada consumer define seu contrato) |
| Ecossistema Spring homogêneo | Spring Cloud Contract (integração nativa) |
| APIs públicas | OpenAPI + validação de schema |

---

## 14. Testes Reativos — WebFlux e WebTestClient

### 14.1 Configuração

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 14.2 StepVerifier — Testando Mono e Flux

```java
@ExtendWith(MockitoExtension.class)
class ProdutoReativoServiceTest {

    @Mock
    private ProdutoReativoRepository repository;

    @InjectMocks
    private ProdutoReativoService service;

    @Test
    void deveBuscarProdutoPorId() {
        var produto = new Produto(1L, "Notebook", new BigDecimal("3500.00"));
        when(repository.findById(1L)).thenReturn(Mono.just(produto));

        StepVerifier.create(service.buscarPorId(1L))
            .expectNextMatches(p -> p.getNome().equals("Notebook"))
            .verifyComplete();
    }

    @Test
    void deveRetornarErroParaProdutoInexistente() {
        when(repository.findById(999L)).thenReturn(Mono.empty());

        StepVerifier.create(service.buscarPorId(999L))
            .expectError(ResourceNotFoundException.class)
            .verify();
    }

    @Test
    void deveListarTodosProdutos() {
        when(repository.findAll()).thenReturn(Flux.just(
            new Produto(1L, "Notebook", new BigDecimal("3500")),
            new Produto(2L, "Mouse", new BigDecimal("60"))
        ));

        StepVerifier.create(service.listarTodos())
            .expectNextCount(2)
            .verifyComplete();
    }

    @Test
    void deveAplicarDescontoEmFluxo() {
        Flux<Produto> produtos = Flux.just(
            new Produto(1L, "A", new BigDecimal("100")),
            new Produto(2L, "B", new BigDecimal("200")),
            new Produto(3L, "C", new BigDecimal("300"))
        );
        when(repository.findAll()).thenReturn(produtos);

        StepVerifier.create(service.listarComDesconto(10))
            .assertNext(p -> assertEquals(new BigDecimal("90.00"), p.getPreco()))
            .assertNext(p -> assertEquals(new BigDecimal("180.00"), p.getPreco()))
            .assertNext(p -> assertEquals(new BigDecimal("270.00"), p.getPreco()))
            .verifyComplete();
    }

    @Test
    void deveTestarFluxComDelay() {
        when(repository.findAll()).thenReturn(
            Flux.just(new Produto(1L, "A", BigDecimal.TEN))
                .delayElements(Duration.ofSeconds(1))
        );

        StepVerifier.withVirtualTime(() -> service.listarTodos())
            .thenAwait(Duration.ofSeconds(1))
            .expectNextCount(1)
            .verifyComplete();
    }
}
```

### 14.3 @WebFluxTest — Slice test para controllers reativos

```java
@WebFluxTest(ProdutoReativoController.class)
class ProdutoReativoControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @MockBean
    private ProdutoReativoService service;

    @Test
    void deveListarProdutos() {
        when(service.listarTodos()).thenReturn(Flux.just(
            new ProdutoDTO(1L, "Notebook", "R$ 3.500,00"),
            new ProdutoDTO(2L, "Mouse", "R$ 60,00")
        ));

        webTestClient.get()
            .uri("/api/produtos")
            .accept(MediaType.APPLICATION_JSON)
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(ProdutoDTO.class)
            .hasSize(2)
            .value(produtos -> {
                assertEquals("Notebook", produtos.get(0).nome());
                assertEquals("Mouse", produtos.get(1).nome());
            });
    }

    @Test
    void deveCriarProduto() {
        var request = new CriarProdutoDTO("Teclado", "149.90", "PERIFERICOS");
        var response = new ProdutoDTO(3L, "Teclado", "R$ 149,90");

        when(service.criar(any())).thenReturn(Mono.just(response));

        webTestClient.post()
            .uri("/api/produtos")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(request)
            .exchange()
            .expectStatus().isCreated()
            .expectBody(ProdutoDTO.class)
            .value(produto -> {
                assertEquals(3L, produto.id());
                assertEquals("Teclado", produto.nome());
            });
    }

    @Test
    void deveStreamarEventos_SSE() {
        when(service.streamEstoque()).thenReturn(Flux.just(
            new EstoqueEvent("PROD-001", 10),
            new EstoqueEvent("PROD-002", 5)
        ));

        webTestClient.get()
            .uri("/api/produtos/estoque/stream")
            .accept(MediaType.TEXT_EVENT_STREAM)
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(EstoqueEvent.class)
            .hasSize(2);
    }
}
```

### 14.4 WebTestClient com servidor real

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProdutoReativoIntegrationTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void deveRealizarCrudCompleto() {
        // Create
        ProdutoDTO criado = webTestClient.post()
            .uri("/api/produtos")
            .bodyValue(new CriarProdutoDTO("Monitor", "1800.00", "ELETRONICOS"))
            .exchange()
            .expectStatus().isCreated()
            .expectBody(ProdutoDTO.class)
            .returnResult().getResponseBody();

        // Read
        webTestClient.get()
            .uri("/api/produtos/{id}", criado.id())
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.nome").isEqualTo("Monitor");

        // Update
        webTestClient.put()
            .uri("/api/produtos/{id}", criado.id())
            .bodyValue(new AtualizarProdutoDTO("Monitor 4K", "2200.00"))
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.nome").isEqualTo("Monitor 4K");

        // Delete
        webTestClient.delete()
            .uri("/api/produtos/{id}", criado.id())
            .exchange()
            .expectStatus().isNoContent();

        // Verify deleted
        webTestClient.get()
            .uri("/api/produtos/{id}", criado.id())
            .exchange()
            .expectStatus().isNotFound();
    }
}
```

---

## 15. Testes End-to-End com Selenium

### 15.1 Configuração

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

### 15.2 Teste E2E com Page Object Pattern

```java
// Page Object
public class LoginPage {

    private final WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(id = "password")
    private WebElement passwordInput;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    @FindBy(css = ".error-message")
    private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public DashboardPage loginComSucesso(String username, String password) {
        usernameInput.sendKeys(username);
        passwordInput.sendKeys(password);
        loginButton.click();
        return new DashboardPage(driver);
    }

    public LoginPage loginComFalha(String username, String password) {
        usernameInput.sendKeys(username);
        passwordInput.sendKeys(password);
        loginButton.click();
        return this;
    }

    public String getErrorMessage() {
        return errorMessage.getText();
    }
}

public class DashboardPage {

    private final WebDriver driver;

    @FindBy(css = ".welcome-message")
    private WebElement welcomeMessage;

    @FindBy(id = "logout-button")
    private WebElement logoutButton;

    public DashboardPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public String getWelcomeMessage() {
        return welcomeMessage.getText();
    }

    public boolean isDisplayed() {
        try {
            return welcomeMessage.isDisplayed();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
}
```

### 15.3 Teste E2E completo com Selenium

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class LoginE2ETest {

    @LocalServerPort
    private int port;

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    private WebDriver driver;

    @BeforeEach
    void setup() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless", "--no-sandbox", "--disable-dev-shm-usage");
        driver = new ChromeDriver(options);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    void deveRealizarLoginComSucesso() {
        driver.get("http://localhost:" + port + "/login");
        LoginPage loginPage = new LoginPage(driver);

        DashboardPage dashboard = loginPage.loginComSucesso("admin", "senha123");

        assertTrue(dashboard.isDisplayed());
        assertThat(dashboard.getWelcomeMessage()).contains("Bem-vindo, admin");
    }

    @Test
    void deveMostrarErroParaCredenciaisInvalidas() {
        driver.get("http://localhost:" + port + "/login");
        LoginPage loginPage = new LoginPage(driver);

        loginPage.loginComFalha("admin", "senha-errada");

        assertEquals("Credenciais inválidas", loginPage.getErrorMessage());
    }

    @Test
    void deveRealizarFluxoCompletoDeCadastroDeProduto() {
        // Login
        driver.get("http://localhost:" + port + "/login");
        DashboardPage dashboard = new LoginPage(driver).loginComSucesso("admin", "senha123");

        // Navegar para cadastro de produtos
        driver.findElement(By.linkText("Produtos")).click();
        driver.findElement(By.id("btn-novo-produto")).click();

        // Preencher formulário
        driver.findElement(By.id("nome")).sendKeys("Notebook Dell");
        driver.findElement(By.id("preco")).sendKeys("4500.00");
        new Select(driver.findElement(By.id("categoria"))).selectByVisibleText("Eletrônicos");
        driver.findElement(By.id("estoque")).sendKeys("10");
        driver.findElement(By.id("btn-salvar")).click();

        // Verificar redirecionamento e mensagem de sucesso
        WebElement mensagem = new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.visibilityOfElementLocated(By.css(".alert-success")));

        assertThat(mensagem.getText()).contains("Produto cadastrado com sucesso");
    }
}
```

### 15.4 Selenium com containers (Selenium Grid)

```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SeleniumGridE2ETest {

    @Container
    static BrowserWebDriverContainer<?> chrome = new BrowserWebDriverContainer<>()
        .withCapabilities(new ChromeOptions())
        .withNetwork(Network.SHARED)
        .withRecordingMode(BrowserWebDriverContainer.VncRecordingMode.RECORD_ALL,
            new File("./target/selenium-recordings"));

    @Test
    void deveTestarNoContainerChrome() {
        RemoteWebDriver driver = chrome.getWebDriver();
        driver.get("http://host.testcontainers.internal:" + port + "/login");
        // ... testes usando o browser containerizado
    }
}
```

---

## 16. Testes End-to-End com Playwright

### 16.1 Configuração para Java

```xml
<dependency>
    <groupId>com.microsoft.playwright</groupId>
    <artifactId>playwright</artifactId>
    <version>1.49.0</version>
    <scope>test</scope>
</dependency>
```

### 16.2 Teste E2E com Playwright

Playwright oferece auto-wait, seletores resilientes e suporte multi-browser de forma nativa:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class PlaywrightE2ETest {

    @LocalServerPort
    private int port;

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    private static Playwright playwright;
    private static Browser browser;
    private BrowserContext context;
    private Page page;

    @BeforeAll
    static void setupBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(
            new BrowserType.LaunchOptions().setHeadless(true));
    }

    @AfterAll
    static void closeBrowser() {
        browser.close();
        playwright.close();
    }

    @BeforeEach
    void setupPage() {
        context = browser.newContext();
        page = context.newPage();
    }

    @AfterEach
    void closePage() {
        context.close();
    }

    @Test
    void deveRealizarLoginComSucesso() {
        page.navigate("http://localhost:" + port + "/login");

        page.fill("#username", "admin");
        page.fill("#password", "senha123");
        page.click("#login-button");

        // Auto-wait: Playwright espera o elemento aparecer automaticamente
        assertThat(page.locator(".welcome-message")).containsText("Bem-vindo, admin");
    }

    @Test
    void deveRealizarCadastroDeProduto() {
        // Login
        page.navigate("http://localhost:" + port + "/login");
        page.fill("#username", "admin");
        page.fill("#password", "senha123");
        page.click("#login-button");

        // Cadastro
        page.click("text=Produtos");
        page.click("#btn-novo-produto");
        page.fill("#nome", "Monitor 4K");
        page.fill("#preco", "2800.00");
        page.selectOption("#categoria", "ELETRONICOS");
        page.fill("#estoque", "15");
        page.click("#btn-salvar");

        // Verificação
        assertThat(page.locator(".alert-success"))
            .containsText("Produto cadastrado com sucesso");

        // Verificar na listagem
        page.click("text=Produtos");
        assertThat(page.locator("table tbody tr")).hasCount(1);
        assertThat(page.locator("table")).containsText("Monitor 4K");
    }

    @Test
    void deveTirarScreenshotEmCasoDeErro() {
        page.navigate("http://localhost:" + port + "/pagina-inexistente");

        if (page.locator(".error-page").isVisible()) {
            page.screenshot(new Page.ScreenshotOptions()
                .setPath(Paths.get("target/screenshots/erro-404.png"))
                .setFullPage(true));
        }

        assertThat(page.locator(".error-page")).isVisible();
    }
}
```

### 16.3 Playwright — Interceptação de rede e API testing

```java
@Test
void deveInterceptarRequisicoesDeRede() {
    // Interceptar chamadas de API e mockar respostas
    page.route("**/api/produtos", route -> {
        String mockResponse = """
            [{"id": 1, "nome": "Produto Mock", "preco": "R$ 99,90"}]
            """;
        route.fulfill(new Route.FulfillOptions()
            .setContentType("application/json")
            .setBody(mockResponse));
    });

    page.navigate("http://localhost:" + port + "/produtos");
    assertThat(page.locator("table")).containsText("Produto Mock");
}

@Test
void deveTestarAPIComPlaywright() {
    // API Testing direto (sem navegador)
    APIRequestContext apiContext = playwright.request().newContext(
        new APIRequest.NewContextOptions()
            .setBaseURL("http://localhost:" + port)
    );

    APIResponse response = apiContext.get("/api/produtos");
    assertEquals(200, response.status());

    APIResponse createResponse = apiContext.post("/api/produtos",
        RequestOptions.create()
            .setData(Map.of("nome", "Teclado", "preco", "149.90",
                           "categoria", "PERIFERICOS", "estoque", 30)));
    assertEquals(201, createResponse.status());
}
```

### 16.4 Playwright vs Selenium — Comparação

| Característica | Selenium | Playwright |
|---------------|----------|------------|
| **Linguagens** | Java, Python, JS, C#, Ruby | Java, Python, JS/TS, C# |
| **Auto-wait** | Não (precisa de waits explícitos) | Sim (embutido) |
| **Seletores** | CSS, XPath | CSS, XPath, texto, role, test-id |
| **Multi-browser** | Via WebDriver separado | Chromium, Firefox, WebKit nativos |
| **Velocidade** | Moderada | Rápida (protocolo CDP direto) |
| **Interceptação de rede** | Não nativo | Nativo (`page.route()`) |
| **Traces / Debug** | Screenshot manual | Trace viewer, vídeo, screenshot |
| **API Testing** | Não | Sim (`playwright.request()`) |
| **Maturidade** | Alta (desde 2004) | Crescente (desde 2020) |
| **Ecossistema** | Muito amplo | Crescendo rapidamente |
| **Uso recomendado** | Projetos legados, ampla compatibilidade | Projetos novos, velocidade de execução |

---

## 17. BDD com Cucumber

### 17.1 Conceito

Behavior-Driven Development (BDD) usa linguagem natural (Gherkin) para descrever cenários de teste, permitindo que stakeholders não-técnicos participem da definição e revisão dos critérios de aceitação.

```
┌──────────────────────────────────────────────────┐
│                  Fluxo BDD                        │
│                                                   │
│  Product Owner ─→ .feature (Gherkin)              │
│                      │                            │
│  Dev/QA ─────────→ Step Definitions (Java)        │
│                      │                            │
│  CI/CD ──────────→ Execução + Relatório           │
└──────────────────────────────────────────────────┘
```

### 17.2 Configuração

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>7.20.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>7.20.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit-platform-engine</artifactId>
    <version>7.20.1</version>
    <scope>test</scope>
</dependency>
<!-- Configuração do JUnit Platform para Cucumber -->
<!-- src/test/resources/junit-platform.properties -->
<!-- cucumber.plugin=pretty,html:target/cucumber-report.html -->
<!-- cucumber.glue=com.example.bdd -->
<!-- cucumber.features=src/test/resources/features -->
```

### 17.3 Feature files (Gherkin)

```gherkin
# src/test/resources/features/pedido.feature
Feature: Gerenciamento de Pedidos
  Como um cliente da loja
  Eu quero criar e gerenciar pedidos
  Para comprar produtos online

  Background:
    Given o sistema possui os seguintes produtos:
      | nome      | preco   | estoque |
      | Notebook  | 3500.00 | 10      |
      | Mouse     | 60.00   | 50      |
      | Monitor   | 1800.00 | 5       |

  Scenario: Criar pedido com sucesso
    Given eu sou um cliente autenticado com email "maria@mail.com"
    When eu crio um pedido com os itens:
      | produto  | quantidade |
      | Notebook | 1          |
      | Mouse    | 2          |
    Then o pedido deve ser criado com status "PENDENTE"
    And o total do pedido deve ser "3620.00"
    And o estoque do "Notebook" deve ser 9
    And o estoque do "Mouse" deve ser 48

  Scenario: Rejeitar pedido sem estoque
    Given eu sou um cliente autenticado com email "joao@mail.com"
    When eu tento criar um pedido com os itens:
      | produto  | quantidade |
      | Monitor  | 10         |
    Then o sistema deve rejeitar com mensagem "Estoque insuficiente para Monitor"

  Scenario Outline: Validar desconto por faixa de valor
    Given eu sou um cliente autenticado com email "user@mail.com"
    When eu crio um pedido no valor de "<total>"
    Then o desconto aplicado deve ser "<desconto>"

    Examples:
      | total    | desconto |
      | 99.00    | 0.00     |
      | 200.00   | 10.00    |
      | 500.00   | 25.00    |
      | 1000.00  | 100.00   |

  Scenario: Cancelar pedido pendente
    Given eu sou um cliente autenticado com email "maria@mail.com"
    And eu possuo um pedido com status "PENDENTE"
    When eu cancelo o pedido
    Then o status do pedido deve ser "CANCELADO"
    And o estoque dos itens deve ser restaurado
```

### 17.4 Step Definitions

```java
@CucumberContextConfiguration
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class CucumberSpringConfig {
    // Classe de configuração — vincula Cucumber ao contexto Spring Boot
}

public class PedidoSteps {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private ProdutoRepository produtoRepository;

    @Autowired
    private PedidoRepository pedidoRepository;

    private ResponseEntity<?> lastResponse;
    private String authToken;
    private Long pedidoId;

    @Given("o sistema possui os seguintes produtos:")
    public void sistemaTemProdutos(DataTable dataTable) {
        produtoRepository.deleteAll();
        dataTable.asMaps().forEach(row -> {
            var produto = new Produto(
                row.get("nome"),
                new BigDecimal(row.get("preco")),
                Integer.parseInt(row.get("estoque"))
            );
            produtoRepository.save(produto);
        });
    }

    @Given("eu sou um cliente autenticado com email {string}")
    public void clienteAutenticado(String email) {
        // Autenticar e obter token
        var loginRequest = Map.of("email", email, "password", "senha123");
        ResponseEntity<Map> response = restTemplate.postForEntity(
            "/api/auth/login", loginRequest, Map.class);
        authToken = (String) response.getBody().get("token");
    }

    @When("eu crio um pedido com os itens:")
    public void criarPedido(DataTable dataTable) {
        List<Map<String, String>> itens = dataTable.asMaps();
        var itensPedido = itens.stream()
            .map(row -> Map.of(
                "produto", row.get("produto"),
                "quantidade", Integer.parseInt(row.get("quantidade"))
            )).toList();

        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(authToken);

        var request = Map.of("itens", itensPedido);
        lastResponse = restTemplate.exchange(
            "/api/pedidos", HttpMethod.POST,
            new HttpEntity<>(request, headers), Map.class);

        if (lastResponse.getStatusCode().is2xxSuccessful()) {
            pedidoId = ((Number) ((Map) lastResponse.getBody()).get("id")).longValue();
        }
    }

    @When("eu tento criar um pedido com os itens:")
    public void tentarCriarPedido(DataTable dataTable) {
        criarPedido(dataTable);  // Reutiliza, mas espera falha
    }

    @Then("o pedido deve ser criado com status {string}")
    public void pedidoCriadoComStatus(String status) {
        assertEquals(HttpStatus.CREATED, lastResponse.getStatusCode());
        assertEquals(status, ((Map) lastResponse.getBody()).get("status"));
    }

    @Then("o total do pedido deve ser {string}")
    public void totalDoPedido(String total) {
        BigDecimal esperado = new BigDecimal(total);
        BigDecimal atual = new BigDecimal(((Map) lastResponse.getBody()).get("total").toString());
        assertEquals(0, esperado.compareTo(atual));
    }

    @Then("o estoque do {string} deve ser {int}")
    public void verificarEstoque(String nomeProduto, int estoqueEsperado) {
        Produto produto = produtoRepository.findByNome(nomeProduto).orElseThrow();
        assertEquals(estoqueEsperado, produto.getEstoque());
    }

    @Then("o sistema deve rejeitar com mensagem {string}")
    public void sistemaDeveRejeitar(String mensagem) {
        assertEquals(HttpStatus.BAD_REQUEST, lastResponse.getStatusCode());
        assertThat(((Map) lastResponse.getBody()).get("message").toString())
            .contains(mensagem);
    }
}
```

### 17.5 Executando testes Cucumber

```java
// Classe runner do Cucumber com JUnit Platform
@Suite
@IncludeEngines("cucumber")
@SelectPackages("com.example.bdd")
@ConfigurationParameter(key = "cucumber.features", value = "src/test/resources/features")
@ConfigurationParameter(key = "cucumber.plugin", value = "pretty,html:target/cucumber-report.html")
class CucumberRunnerTest {
}
```

```bash
# Via Maven
mvn test -Dcucumber.filter.tags="@smoke"
mvn test -Dcucumber.filter.tags="not @slow"
```

---

## 18. Testes de Carga e Performance

### 18.1 Panorama de ferramentas

| Ferramenta | Linguagem de Script | Ideal para |
|-----------|-------------------|-----------|
| **Gatling** | Scala / Java DSL | Desenvolvedores Java, relatórios detalhados |
| **k6** | JavaScript | DevOps, CI/CD, scripts leves |
| **JMeter** | GUI / XML | QA tradicional, testes complexos |
| **Locust** | Python | Equipes Python, simplicidade |

### 18.2 Gatling — Teste de carga em Java

```xml
<dependency>
    <groupId>io.gatling.highcharts</groupId>
    <artifactId>gatling-charts-highcharts</artifactId>
    <version>3.11.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.gatling</groupId>
    <artifactId>gatling-app</artifactId>
    <version>3.11.5</version>
    <scope>test</scope>
</dependency>
```

```java
// Simulação Gatling com Java DSL
public class ProdutoLoadSimulation extends Simulation {

    HttpProtocolBuilder httpProtocol = http
        .baseUrl("http://localhost:8080")
        .acceptHeader("application/json")
        .contentTypeHeader("application/json");

    // Cenário: Listagem de produtos (leitura)
    ScenarioBuilder cenarioLeitura = scenario("Listagem de Produtos")
        .exec(
            http("Listar todos")
                .get("/api/produtos")
                .check(status().is(200))
                .check(jsonPath("$").count().gte(0))
        )
        .pause(1, 3)
        .exec(
            http("Buscar por ID")
                .get("/api/produtos/1")
                .check(status().is(200))
                .check(jsonPath("$.nome").exists())
        );

    // Cenário: Criação de pedidos (escrita)
    ScenarioBuilder cenarioCriacao = scenario("Criação de Pedidos")
        .exec(
            http("Criar Pedido")
                .post("/api/pedidos")
                .body(StringBody("""
                    {
                        "clienteId": 1,
                        "itens": [{"produtoId": 1, "quantidade": 2}]
                    }
                    """))
                .check(status().is(201))
        )
        .pause(2, 5);

    {
        setUp(
            // Ramp-up: de 0 a 100 usuários em 60 segundos
            cenarioLeitura.injectOpen(
                rampUsers(100).during(Duration.ofSeconds(60)),
                constantUsersPerSec(50).during(Duration.ofMinutes(5))
            ),
            cenarioCriacao.injectOpen(
                rampUsers(20).during(Duration.ofSeconds(30)),
                constantUsersPerSec(10).during(Duration.ofMinutes(5))
            )
        ).protocols(httpProtocol)
         .assertions(
             global().responseTime().percentile3().lt(500),   // p95 < 500ms
             global().responseTime().percentile4().lt(1000),  // p99 < 1s
             global().successfulRequests().percent().gt(99.0), // > 99% sucesso
             forAll().failedRequests().percent().lt(1.0)       // < 1% falha
         );
    }
}
```

### 18.3 k6 — Teste de carga com JavaScript

```javascript
// load-test.js — executar com: k6 run load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const responseTime = new Trend('response_time');

export const options = {
  stages: [
    { duration: '30s', target: 20 },   // Ramp-up para 20 VUs
    { duration: '2m',  target: 100 },  // Escalar para 100 VUs
    { duration: '5m',  target: 100 },  // Manter 100 VUs por 5 min
    { duration: '30s', target: 0 },    // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'], // Latência
    errors: ['rate<0.01'],                           // Taxa de erro
    http_req_failed: ['rate<0.01'],                  // Requisições falhadas
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:8080';

export default function () {
  // Listar produtos
  const listResponse = http.get(`${BASE_URL}/api/produtos`);
  check(listResponse, {
    'listagem retornou 200': (r) => r.status === 200,
    'listagem retornou array': (r) => JSON.parse(r.body).length >= 0,
  });
  responseTime.add(listResponse.timings.duration);
  errorRate.add(listResponse.status !== 200);

  sleep(Math.random() * 3 + 1);

  // Buscar produto específico
  const id = Math.floor(Math.random() * 100) + 1;
  const getResponse = http.get(`${BASE_URL}/api/produtos/${id}`);
  check(getResponse, {
    'busca retornou 200 ou 404': (r) => [200, 404].includes(r.status),
  });

  sleep(Math.random() * 2 + 1);

  // Criar pedido (10% das requisições)
  if (Math.random() < 0.1) {
    const payload = JSON.stringify({
      clienteId: Math.floor(Math.random() * 50) + 1,
      itens: [{ produtoId: id, quantidade: 1 }],
    });

    const createResponse = http.post(`${BASE_URL}/api/pedidos`, payload, {
      headers: { 'Content-Type': 'application/json' },
    });
    check(createResponse, {
      'criação retornou 201': (r) => r.status === 201,
    });
  }

  sleep(1);
}
```

### 18.4 Métricas essenciais de performance

```
┌─────────────────────────────────────────────────────────┐
│              Métricas de Performance                     │
├──────────────────┬──────────────────────────────────────┤
│ Throughput       │ Requisições/segundo (RPS)            │
│ Latência p50     │ 50% das requisições abaixo deste     │
│ Latência p95     │ 95% (baseline SLA)                   │
│ Latência p99     │ 99% (cauda — detecta outliers)       │
│ Error Rate       │ % de requisições com erro            │
│ Saturação        │ Ponto onde RPS para de crescer       │
│ Concurrent Users │ Usuários simultâneos no sistema      │
│ Apdex Score      │ Satisfação: (satisfeito + tolerável  │
│                  │ × 0.5) / total                       │
└──────────────────┴──────────────────────────────────────┘
```

### 18.5 Tipos de teste de carga

| Tipo | Objetivo | Padrão de carga |
|------|---------|----------------|
| **Smoke Test** | Verificar que funciona sob carga mínima | 1-5 VUs, 1 min |
| **Load Test** | Validar comportamento sob carga esperada | Carga normal, 5-10 min |
| **Stress Test** | Encontrar o ponto de ruptura | Carga crescente até falhar |
| **Spike Test** | Testar picos súbitos de tráfego | Pico abrupto de VUs |
| **Soak Test** | Detectar memory leaks e degradação | Carga constante, 1-8 horas |
| **Breakpoint Test** | Encontrar capacidade máxima | Escalar até o limite |

---

## 19. Testes de Segurança — Vulnerabilidades e Penetração

### 19.1 Pirâmide de testes de segurança

```
              ╱╲
             ╱  ╲         Pentest Manual
            ╱ PT ╲        (especialista, caro, profundo)
           ╱──────╲
          ╱        ╲      DAST — Dynamic Analysis
         ╱  DAST    ╲     (OWASP ZAP, em runtime)
        ╱────────────╲
       ╱              ╲   IAST — Interactive Analysis
      ╱     IAST       ╲  (agente em runtime, combina SAST+DAST)
     ╱──────────────────╲
    ╱                    ╲ SAST — Static Analysis
   ╱   SAST / SCA / Lint  ╲ (código-fonte, dependências, linters)
  ╱────────────────────────╲
```

### 19.2 Testes de segurança com Spring Security Test

```java
@WebMvcTest(ProdutoController.class)
@Import(SecurityConfig.class)
class ProdutoSecurityTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProdutoService produtoService;

    // --- Testes de autenticação ---

    @Test
    void deveExigirAutenticacaoParaEndpointsProtegidos() throws Exception {
        mockMvc.perform(get("/api/produtos"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    void devePermitirAcessoAEndpointsPublicos() throws Exception {
        mockMvc.perform(get("/api/public/health"))
            .andExpect(status().isOk());
    }

    // --- Testes de autorização ---

    @Test
    @WithMockUser(roles = "USER")
    void devePermitirLeituraParaUsuarioComum() throws Exception {
        when(produtoService.listarTodos()).thenReturn(List.of());
        mockMvc.perform(get("/api/produtos"))
            .andExpect(status().isOk());
    }

    @Test
    @WithMockUser(roles = "USER")
    void deveNegarCriacaoParaUsuarioSemRoleAdmin() throws Exception {
        mockMvc.perform(post("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"nome\":\"Test\"}"))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void devePermitirCriacaoParaAdmin() throws Exception {
        when(produtoService.criar(any())).thenReturn(new ProdutoDTO(1L, "Test", "R$ 10", "CAT"));
        mockMvc.perform(post("/api/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"nome\":\"Test\",\"preco\":\"10.00\",\"categoria\":\"CAT\",\"estoque\":1}")
                .with(csrf()))
            .andExpect(status().isCreated());
    }

    // --- Testes de CORS ---

    @Test
    void deveRejeitarOrigemNaoPermitida() throws Exception {
        mockMvc.perform(options("/api/produtos")
                .header("Origin", "https://malicious-site.com")
                .header("Access-Control-Request-Method", "GET"))
            .andExpect(status().isForbidden());
    }

    @Test
    void deveAceitarOrigemPermitida() throws Exception {
        mockMvc.perform(options("/api/produtos")
                .header("Origin", "https://meu-frontend.com")
                .header("Access-Control-Request-Method", "GET"))
            .andExpect(status().isOk())
            .andExpect(header().string("Access-Control-Allow-Origin", "https://meu-frontend.com"));
    }
}
```

### 19.3 Testes contra OWASP Top 10

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OwaspSecurityTest {

    @Autowired
    private TestRestTemplate restTemplate;

    // --- A01: Broken Access Control ---

    @Test
    void deveImpedirAcessoHorizontal_IDOR() {
        // Usuário 1 não pode acessar dados do Usuário 2
        HttpHeaders headers = criarHeadersComToken("usuario1");
        ResponseEntity<String> response = restTemplate.exchange(
            "/api/usuarios/2/dados-pessoais",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            String.class
        );
        assertEquals(HttpStatus.FORBIDDEN, response.getStatusCode());
    }

    @Test
    void deveImpedirEscalacaoDePrivilegio() {
        HttpHeaders headers = criarHeadersComToken("usuario-comum");
        ResponseEntity<String> response = restTemplate.exchange(
            "/api/admin/configuracoes",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            String.class
        );
        assertNotEquals(HttpStatus.OK, response.getStatusCode());
    }

    // --- A03: Injection ---

    @Test
    void deveProtegerContraSqlInjection() {
        String maliciousInput = "'; DROP TABLE usuarios; --";
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/api/produtos?nome=" + maliciousInput, String.class);

        assertNotEquals(HttpStatus.INTERNAL_SERVER_ERROR, response.getStatusCode());
        // A tabela ainda deve existir após a tentativa
        ResponseEntity<String> check = restTemplate.getForEntity("/api/produtos", String.class);
        assertEquals(HttpStatus.OK, check.getStatusCode());
    }

    @Test
    void deveProtegerContraXSS() {
        var request = Map.of(
            "nome", "<script>alert('xss')</script>",
            "descricao", "<img src=x onerror=alert('xss')>"
        );

        ResponseEntity<String> response = restTemplate.postForEntity(
            "/api/produtos", request, String.class);

        if (response.getStatusCode().is2xxSuccessful()) {
            String body = response.getBody();
            assertFalse(body.contains("<script>"));
            assertFalse(body.contains("onerror"));
        }
    }

    // --- A07: Identification and Authentication Failures ---

    @Test
    void deveImplementarRateLimiting() {
        int tentativas = 20;
        int bloqueados = 0;

        for (int i = 0; i < tentativas; i++) {
            ResponseEntity<String> response = restTemplate.postForEntity(
                "/api/auth/login",
                Map.of("username", "admin", "password", "senha-errada-" + i),
                String.class
            );
            if (response.getStatusCode() == HttpStatus.TOO_MANY_REQUESTS) {
                bloqueados++;
            }
        }

        assertTrue(bloqueados > 0,
            "Rate limiting deveria bloquear tentativas excessivas de login");
    }

    @Test
    void deveExigirSenhaForte() {
        var senhasFracas = List.of("123456", "password", "abc", "12345678");

        for (String senha : senhasFracas) {
            ResponseEntity<String> response = restTemplate.postForEntity(
                "/api/auth/registrar",
                Map.of("username", "novo-user", "password", senha, "email", "test@mail.com"),
                String.class
            );
            assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode(),
                "Senha fraca deveria ser rejeitada: " + senha);
        }
    }

    // --- A08: Software and Data Integrity Failures ---

    @Test
    void deveValidarJwtIntegridade() {
        // Token com assinatura manipulada
        String tokenAdulterado = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJBRE1JTiJ9.FAKE_SIGNATURE";

        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(tokenAdulterado);

        ResponseEntity<String> response = restTemplate.exchange(
            "/api/produtos",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            String.class
        );

        assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
    }

    // --- A09: Security Logging and Monitoring ---

    @Test
    void deveNaoExporDadosSensiveisNoLog() {
        // Verificar que stacktraces não vazam para o cliente
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/api/produtos/abc",  // ID inválido deve retornar erro tratado
            String.class
        );

        String body = response.getBody();
        assertFalse(body.contains("Exception"), "Stacktrace não deve ser exposto");
        assertFalse(body.contains("org.springframework"), "Detalhes internos não devem vazar");
        assertFalse(body.contains("jdbc:"), "String de conexão não deve vazar");
    }
}
```

### 19.4 OWASP ZAP — Teste automatizado DAST

```java
// Integração do OWASP ZAP com testes JUnit
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class ZapSecurityScanTest {

    @LocalServerPort
    private int port;

    @Container
    static GenericContainer<?> zap = new GenericContainer<>("ghcr.io/zaproxy/zaproxy:stable")
        .withExposedPorts(8080)
        .withCommand("zap.sh", "-daemon", "-host", "0.0.0.0",
                     "-port", "8080", "-config", "api.disablekey=true");

    @Test
    void devePassarScanBasicoDoZap() throws Exception {
        String zapUrl = "http://localhost:" + zap.getMappedPort(8080);
        String targetUrl = "http://host.testcontainers.internal:" + port;

        // Iniciar scan via API do ZAP
        HttpClient client = HttpClient.newHttpClient();

        // Spider (descoberta de URLs)
        HttpRequest spiderRequest = HttpRequest.newBuilder()
            .uri(URI.create(zapUrl + "/JSON/spider/action/scan/?url=" + targetUrl))
            .build();
        client.send(spiderRequest, HttpResponse.BodyHandlers.ofString());

        // Aguardar spider completar
        Thread.sleep(10000);

        // Active scan
        HttpRequest scanRequest = HttpRequest.newBuilder()
            .uri(URI.create(zapUrl + "/JSON/ascan/action/scan/?url=" + targetUrl))
            .build();
        client.send(scanRequest, HttpResponse.BodyHandlers.ofString());

        // Aguardar scan completar
        Thread.sleep(30000);

        // Verificar alertas
        HttpRequest alertsRequest = HttpRequest.newBuilder()
            .uri(URI.create(zapUrl + "/JSON/core/view/alerts/?baseurl=" + targetUrl))
            .build();
        HttpResponse<String> alertsResponse = client.send(alertsRequest,
            HttpResponse.BodyHandlers.ofString());

        ObjectMapper mapper = new ObjectMapper();
        JsonNode alerts = mapper.readTree(alertsResponse.body()).get("alerts");

        // Não deve ter vulnerabilidades HIGH ou CRITICAL
        long highRiskAlerts = StreamSupport.stream(alerts.spliterator(), false)
            .filter(a -> {
                String risk = a.get("risk").asText();
                return "High".equals(risk) || "Critical".equals(risk);
            })
            .count();

        assertEquals(0, highRiskAlerts,
            "Encontradas " + highRiskAlerts + " vulnerabilidades de alto risco");
    }
}
```

### 19.5 Testes de Headers de segurança

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SecurityHeadersTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void deveIncluirHeadersDeSeguranca() {
        ResponseEntity<String> response = restTemplate.getForEntity("/api/public/health", String.class);

        assertAll("Security Headers",
            () -> assertEquals("nosniff",
                response.getHeaders().getFirst("X-Content-Type-Options")),
            () -> assertEquals("DENY",
                response.getHeaders().getFirst("X-Frame-Options")),
            () -> assertNotNull(
                response.getHeaders().getFirst("Strict-Transport-Security")),
            () -> assertEquals("1; mode=block",
                response.getHeaders().getFirst("X-XSS-Protection")),
            () -> assertNotNull(
                response.getHeaders().getFirst("Content-Security-Policy")),
            () -> assertNull(
                response.getHeaders().getFirst("Server"),
                "Header 'Server' não deve ser exposto")
        );
    }

    @Test
    void deveIncluirCacheControlParaDadosSensiveis() {
        HttpHeaders headers = criarHeadersComToken("usuario1");
        ResponseEntity<String> response = restTemplate.exchange(
            "/api/usuarios/me",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            String.class
        );

        String cacheControl = response.getHeaders().getFirst("Cache-Control");
        assertNotNull(cacheControl);
        assertTrue(cacheControl.contains("no-store") || cacheControl.contains("no-cache"));
    }
}
```

### 19.6 Pipeline de segurança no CI/CD

```yaml
# .github/workflows/security.yml
name: Security Checks

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: SpotBugs + Find Security Bugs
        run: mvn spotbugs:check -Dspotbugs.plugins=com.h3xstream.findsecbugs

      - name: Semgrep SAST
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/java
            p/owasp-top-ten
            p/spring

  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: OWASP Dependency Check
        run: mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7

      - name: Trivy (vulnerabilidades em dependências)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'CRITICAL,HIGH'

  dast:
    runs-on: ubuntu-latest
    needs: [sast, sca]
    steps:
      - uses: actions/checkout@v4

      - name: Build e subir aplicação
        run: |
          mvn package -DskipTests
          java -jar target/*.jar &
          sleep 30

      - name: OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'http://localhost:8080'
          rules_file_name: '.zap/rules.tsv'

  secrets:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Gitleaks (secrets no código)
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 20. Property-Based Testing com jqwik

### 20.1 Conceito

Property-Based Testing (PBT) inverte a abordagem dos testes tradicionais: em vez de definir entradas e saídas específicas, você define **propriedades** que devem ser verdadeiras para **qualquer** entrada válida, e o framework gera centenas de inputs aleatórios para tentar falsificá-las.

```
Teste Tradicional                Property-Based Testing
─────────────────                ──────────────────────
entrada = "abc"                  ∀ entrada ∈ String:
esperado = "ABC"                   toUpper(toLower(entrada)) == toUpper(entrada)
assert toUpper("abc") == "ABC"     → Gera 1000 strings aleatórias e valida
```

### 20.2 Configuração (jqwik)

```xml
<dependency>
    <groupId>net.jqwik</groupId>
    <artifactId>jqwik</artifactId>
    <version>1.9.2</version>
    <scope>test</scope>
</dependency>
<!-- Integração com Spring Boot -->
<dependency>
    <groupId>net.jqwik</groupId>
    <artifactId>jqwik-spring</artifactId>
    <version>0.12.0</version>
    <scope>test</scope>
</dependency>
```

### 20.3 Exemplos de propriedades

```java
import net.jqwik.api.*;

class DescontoPropertyTest {

    @Property
    void descontoNuncaDeveResultarEmValorNegativo(
            @ForAll @DoubleRange(min = 0.01, max = 100000) double preco,
            @ForAll @IntRange(min = 0, max = 100) int percentual) {

        BigDecimal valor = BigDecimal.valueOf(preco).setScale(2, RoundingMode.HALF_UP);
        BigDecimal resultado = DescontoService.aplicar(valor, percentual);

        assertThat(resultado).isGreaterThanOrEqualTo(BigDecimal.ZERO);
    }

    @Property
    void descontoZeroDeveRetornarValorOriginal(
            @ForAll @DoubleRange(min = 0.01, max = 100000) double preco) {

        BigDecimal valor = BigDecimal.valueOf(preco).setScale(2, RoundingMode.HALF_UP);
        BigDecimal resultado = DescontoService.aplicar(valor, 0);

        assertThat(resultado).isEqualByComparingTo(valor);
    }

    @Property
    void desconto100DeveRetornarZero(
            @ForAll @DoubleRange(min = 0.01, max = 100000) double preco) {

        BigDecimal valor = BigDecimal.valueOf(preco).setScale(2, RoundingMode.HALF_UP);
        BigDecimal resultado = DescontoService.aplicar(valor, 100);

        assertThat(resultado).isEqualByComparingTo(BigDecimal.ZERO);
    }
}
```

### 20.4 Geradores customizados (Arbitraries)

```java
class PedidoPropertyTest {

    @Provide
    Arbitrary<Pedido> pedidosValidos() {
        Arbitrary<String> nomes = Arbitraries.strings().alpha().ofMinLength(3).ofMaxLength(50);
        Arbitrary<BigDecimal> precos = Arbitraries.bigDecimals()
            .between(BigDecimal.ONE, new BigDecimal("99999.99"))
            .ofScale(2);
        Arbitrary<Integer> quantidades = Arbitraries.integers().between(1, 100);

        return Combinators.combine(nomes, precos, quantidades)
            .as((nome, preco, qtd) -> new Pedido(nome, preco, qtd));
    }

    @Property
    void totalDeveSerPrecoVezesQuantidade(@ForAll("pedidosValidos") Pedido pedido) {
        BigDecimal esperado = pedido.getPreco()
            .multiply(BigDecimal.valueOf(pedido.getQuantidade()));

        assertThat(pedido.calcularTotal()).isEqualByComparingTo(esperado);
    }

    @Property
    void cpfValidoDevePassarNaValidacao(
            @ForAll("cpfsValidos") String cpf) {
        assertTrue(ValidadorCpf.isValido(cpf));
    }

    @Provide
    Arbitrary<String> cpfsValidos() {
        return Arbitraries.of(
            "529.982.247-25", "111.444.777-35", "123.456.789-09"
        );
    }

    // Testando invariantes de domínio
    @Property(tries = 500)
    void saldoNuncaDeveSerNegativoAposOperacoes(
            @ForAll @Size(min = 1, max = 20) List<@DoubleRange(min = -1000, max = 1000) Double> operacoes) {

        Conta conta = new Conta(new BigDecimal("1000.00"));

        for (Double op : operacoes) {
            try {
                if (op >= 0) conta.depositar(BigDecimal.valueOf(op));
                else conta.sacar(BigDecimal.valueOf(-op));
            } catch (SaldoInsuficienteException e) {
                // Esperado — saque maior que saldo
            }
        }

        assertThat(conta.getSaldo()).isGreaterThanOrEqualTo(BigDecimal.ZERO);
    }
}
```

### 20.5 Quando usar Property-Based Testing

| Cenário | Exemplo |
|---------|---------|
| **Funções matemáticas** | Desconto, juros, impostos, conversão de moeda |
| **Serialização/deserialização** | `deserialize(serialize(x)) == x` |
| **Invariantes de domínio** | Saldo nunca negativo, estoque >= 0, data fim > data início |
| **Validação de entrada** | Nenhuma string válida causa exception inesperada |
| **Idempotência** | `f(f(x)) == f(x)` para operações idempotentes |
| **Comutatividade** | `a + b == b + a` para cálculos combinatórios |

---

## 21. Snapshot e Approval Testing

### 21.1 Conceito

Snapshot Testing (ou Approval Testing) compara a saída atual de uma operação contra uma versão previamente "aprovada" (baseline). Ideal para testar saídas complexas como JSON, HTML, relatórios ou respostas de API sem escrever dezenas de assertions individuais.

```
Primeira execução:              Execuções seguintes:
─────────────────               ───────────────────
1. Gera output                  1. Gera output
2. Salva como .approved         2. Compara com .approved
3. Dev revisa e confirma        3. Igual? ✅ Passa
                                4. Diferente? ❌ Falha + diff
```

### 21.2 Configuração (ApprovalTests)

```xml
<dependency>
    <groupId>com.approvaltests</groupId>
    <artifactId>approvaltests</artifactId>
    <version>24.3.0</version>
    <scope>test</scope>
</dependency>
```

### 21.3 Snapshot de resposta JSON de API

```java
class ProdutoSnapshotTest {

    private final ObjectMapper objectMapper = new ObjectMapper()
        .enable(SerializationFeature.INDENT_OUTPUT)
        .registerModule(new JavaTimeModule());

    @Test
    void deveGerarJsonDeProdutoCorreto() {
        var produto = new ProdutoDTO(1L, "Notebook Dell XPS 15",
            "R$ 8.500,00", "ELETRONICOS",
            LocalDate.of(2025, 6, 1), true, List.of("tag1", "tag2"));

        String json = objectMapper.writeValueAsString(produto);

        Approvals.verify(json);
        // Na primeira execução: gera ProdutoSnapshotTest.deveGerarJsonDeProdutoCorreto.approved.txt
        // Nas próximas: compara automaticamente
    }

    @Test
    void deveGerarRelatorioDeVendas() {
        var relatorio = relatorioService.gerar(
            LocalDate.of(2025, 1, 1), LocalDate.of(2025, 1, 31));

        Approvals.verify(objectMapper.writeValueAsString(relatorio));
    }

    @Test
    void deveGerarListaDeProdutosPorCategoria() {
        var categorias = Map.of(
            "ELETRONICOS", List.of("Notebook", "Monitor", "Mouse"),
            "SOFTWARE", List.of("IntelliJ", "Windows")
        );

        Approvals.verifyAll("Categorias", categorias.entrySet(),
            entry -> entry.getKey() + ": " + String.join(", ", entry.getValue()));
    }
}
```

### 21.4 Snapshot com AssertJ (JSON)

```java
// Alternativa sem biblioteca extra — usando AssertJ JSON
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ApiSnapshotTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void deveRetornarEstruturaDeProdutoEsperada() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/api/produtos/1", String.class);

        // Comparação com arquivo JSON de referência
        assertThatJson(response.getBody())
            .isEqualTo(resource("snapshots/produto-1.json"));
    }

    @Test
    void deveRetornarErroComEstruturaCorreta() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/api/produtos/999", String.class);

        // Verifica estrutura sem valores exatos
        assertThatJson(response.getBody())
            .isObject()
            .containsKeys("timestamp", "status", "message", "path")
            .node("status").isEqualTo(404);
    }
}
```

### 21.5 Quando usar Snapshot Testing

| Usar quando | Evitar quando |
|------------|--------------|
| Saídas complexas com muitos campos | Valores mudam a cada execução (timestamps, UUIDs) |
| Relatórios, e-mails HTML renderizados | Lógica de negócio simples (use assertions diretas) |
| Respostas de API com estrutura estável | Dados sensíveis que não devem ir para o repositório |
| Migração — capturar comportamento existente | Campos voláteis (excluir ou normalizar antes) |

---

## 22. ArchUnit — Testes de Arquitetura

### 22.1 Conceito

ArchUnit permite escrever testes automatizados que verificam se o código segue as regras arquiteturais definidas pela equipe — dependências entre camadas, naming conventions, padrões de pacote, e proibições de uso.

### 22.2 Configuração

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.3.0</version>
    <scope>test</scope>
</dependency>
```

### 22.3 Regras de camadas (Layered Architecture)

```java
@AnalyzeClasses(packages = "com.example", importOptions = ImportOption.DoNotIncludeTests.class)
class ArquiteturaCamadasTest {

    @ArchTest
    static final ArchRule arquitetura_em_camadas = layeredArchitecture()
        .consideringAllDependencies()
        .layer("Controller").definedBy("..controller..")
        .layer("Service").definedBy("..service..")
        .layer("Repository").definedBy("..repository..")
        .layer("Model").definedBy("..model..")
        .layer("DTO").definedBy("..dto..")
        .layer("Config").definedBy("..config..")

        .whereLayer("Controller").mayNotBeAccessedByAnyLayer()
        .whereLayer("Service").mayOnlyBeAccessedByLayers("Controller", "Service")
        .whereLayer("Repository").mayOnlyBeAccessedByLayers("Service")
        .whereLayer("Model").mayOnlyBeAccessedByLayers("Service", "Repository", "DTO")
        .whereLayer("DTO").mayOnlyBeAccessedByLayers("Controller", "Service");

    @ArchTest
    static final ArchRule controllers_nao_devem_acessar_repositories =
        noClasses()
            .that().resideInAPackage("..controller..")
            .should().dependOnClassesThat().resideInAPackage("..repository..");
}
```

### 22.4 Regras de naming conventions

```java
@AnalyzeClasses(packages = "com.example")
class NamingConventionTest {

    @ArchTest
    static final ArchRule controllers_devem_ter_sufixo_Controller =
        classes()
            .that().resideInAPackage("..controller..")
            .and().areAnnotatedWith(RestController.class)
            .should().haveSimpleNameEndingWith("Controller");

    @ArchTest
    static final ArchRule services_devem_ter_sufixo_Service =
        classes()
            .that().resideInAPackage("..service..")
            .and().areAnnotatedWith(Service.class)
            .should().haveSimpleNameEndingWith("Service")
            .orShould().haveSimpleNameEndingWith("ServiceImpl");

    @ArchTest
    static final ArchRule repositories_devem_ser_interfaces =
        classes()
            .that().resideInAPackage("..repository..")
            .and().haveSimpleNameEndingWith("Repository")
            .should().beInterfaces();

    @ArchTest
    static final ArchRule dtos_devem_ser_records =
        classes()
            .that().resideInAPackage("..dto..")
            .and().haveSimpleNameEndingWith("DTO")
            .should().beRecords();
}
```

### 22.5 Regras de dependência e proibições

```java
@AnalyzeClasses(packages = "com.example")
class DependenciaTest {

    @ArchTest
    static final ArchRule nao_usar_java_util_logging =
        noClasses()
            .should().dependOnClassesThat()
            .resideInAPackage("java.util.logging");

    @ArchTest
    static final ArchRule usar_slf4j_para_logging =
        fields()
            .that().haveRawType(org.slf4j.Logger.class)
            .should().bePrivate()
            .andShould().beStatic()
            .andShould().beFinal();

    @ArchTest
    static final ArchRule nao_usar_field_injection =
        noFields()
            .should().beAnnotatedWith(Autowired.class)
            .because("Prefira injeção por construtor");

    @ArchTest
    static final ArchRule entities_nao_devem_expor_em_controllers =
        noMethods()
            .that().areDeclaredInClassesThat().resideInAPackage("..controller..")
            .should().haveRawReturnType(resideInAPackage("..model.."))
            .because("Controllers devem retornar DTOs, não entidades JPA");

    @ArchTest
    static final ArchRule services_devem_ser_transacionais =
        classes()
            .that().resideInAPackage("..service..")
            .and().areAnnotatedWith(Service.class)
            .should().beAnnotatedWith(Transactional.class)
            .orShould().haveMethodThat(method -> method.isAnnotatedWith(Transactional.class));
}
```

### 22.6 Regras de ciclos e dependências entre pacotes

```java
@AnalyzeClasses(packages = "com.example")
class CicloDependenciaTest {

    @ArchTest
    static final ArchRule sem_ciclos_entre_pacotes =
        slices().matching("com.example.(*)..")
            .should().beFreeOfCycles();

    @ArchTest
    static final ArchRule modulos_independentes =
        slices().matching("com.example.modulo.(*)..")
            .should().notDependOnEachOther();
}
```

---

## 23. Testes de Observabilidade

### 23.1 Panorama

Testes de observabilidade verificam que a aplicação emite corretamente métricas, logs e traces — fundamentais para monitoramento em produção (dashboards, alertas, troubleshooting).

```
┌────────────────────────────────────────────────┐
│           Pilares da Observabilidade            │
├─────────────┬──────────────┬───────────────────┤
│   Métricas  │     Logs     │     Traces        │
│ (Micrometer)│  (SLF4J/     │ (Micrometer       │
│             │   Logback)   │  Tracing/OTel)    │
├─────────────┼──────────────┼───────────────────┤
│ Contadores  │ Eventos      │ Spans             │
│ Gauges      │ Estruturados │ Propagação de     │
│ Timers      │ Níveis       │ contexto          │
│ Histogramas │ MDC          │ Parent-child      │
└─────────────┴──────────────┴───────────────────┘
```

### 23.2 Testando métricas (Micrometer)

```java
@SpringBootTest
class MetricasTest {

    @Autowired
    private MeterRegistry meterRegistry;

    @Autowired
    private PedidoService pedidoService;

    @BeforeEach
    void limparMetricas() {
        meterRegistry.clear();
    }

    @Test
    void deveIncrementarContadorDePedidosCriados() {
        pedidoService.criar(criarPedidoRequest());
        pedidoService.criar(criarPedidoRequest());

        Counter counter = meterRegistry.find("pedidos.criados").counter();

        assertNotNull(counter, "Métrica 'pedidos.criados' deveria existir");
        assertEquals(2.0, counter.count());
    }

    @Test
    void deveRegistrarTempoDeProcessamento() {
        pedidoService.processar(1L);

        Timer timer = meterRegistry.find("pedido.processamento.tempo").timer();

        assertNotNull(timer);
        assertTrue(timer.count() > 0);
        assertTrue(timer.mean(TimeUnit.MILLISECONDS) > 0);
    }

    @Test
    void deveRegistrarMetricasComTags() {
        pedidoService.criar(criarPedidoRequest("CARTAO"));
        pedidoService.criar(criarPedidoRequest("PIX"));
        pedidoService.criar(criarPedidoRequest("PIX"));

        double pedidosPix = meterRegistry.find("pedidos.criados")
            .tag("metodo_pagamento", "PIX")
            .counter().count();

        assertEquals(2.0, pedidosPix);
    }

    @Test
    void deveRegistrarGaugeDeEstoque() {
        Gauge gauge = meterRegistry.find("estoque.quantidade")
            .tag("produto", "PROD-001")
            .gauge();

        assertNotNull(gauge);
        assertTrue(gauge.value() >= 0);
    }
}
```

### 23.3 Testando logs (OutputCaptureExtension)

```java
// JUnit 5 — Spring Boot OutputCaptureExtension
@ExtendWith(OutputCaptureExtension.class)
class LoggingTest {

    private final PedidoService pedidoService = new PedidoService();

    @Test
    void deveLogarCriacaoDePedido(CapturedOutput output) {
        pedidoService.criar(criarPedidoRequest());

        assertThat(output.getOut()).contains("Pedido criado com sucesso");
        assertThat(output.getOut()).contains("pedidoId=");
    }

    @Test
    void deveLogarErroComoWarn(CapturedOutput output) {
        try {
            pedidoService.processar(999L);
        } catch (Exception e) {
            // esperado
        }

        assertThat(output.getOut()).contains("WARN");
        assertThat(output.getOut()).contains("Pedido não encontrado: 999");
    }

    @Test
    void naoDeveLogarDadosSensiveis(CapturedOutput output) {
        var request = criarPedidoRequestComCartao("4111111111111111");
        pedidoService.criar(request);

        assertThat(output.getOut()).doesNotContain("4111111111111111");
        assertThat(output.getOut()).doesNotContain("senha");
        assertThat(output.getOut()).doesNotContain("password");
    }
}
```

### 23.4 Testando logs estruturados com Logback

```java
// Teste com appender customizado para capturar eventos de log
class LogStructuredTest {

    private ListAppender<ILoggingEvent> listAppender;
    private Logger logger;

    @BeforeEach
    void setup() {
        logger = (Logger) LoggerFactory.getLogger(PedidoService.class);
        listAppender = new ListAppender<>();
        listAppender.start();
        logger.addAppender(listAppender);
    }

    @AfterEach
    void teardown() {
        logger.detachAppender(listAppender);
    }

    @Test
    void deveIncluirMDCNosLogs() {
        var service = new PedidoService();
        service.criar(criarPedidoRequest());

        List<ILoggingEvent> logs = listAppender.list;
        ILoggingEvent logEvento = logs.stream()
            .filter(e -> e.getMessage().contains("Pedido criado"))
            .findFirst().orElseThrow();

        // Verificar campos MDC (Mapped Diagnostic Context)
        assertEquals("pedido-service", logEvento.getMDCPropertyMap().get("service"));
        assertNotNull(logEvento.getMDCPropertyMap().get("requestId"));
        assertEquals(Level.INFO, logEvento.getLevel());
    }

    @Test
    void deveLogarApenasNiveisCorretos() {
        var service = new PedidoService();
        service.criar(criarPedidoRequest());

        // Não deve ter logs DEBUG em código de produção (quando profile != dev)
        boolean temDebug = listAppender.list.stream()
            .anyMatch(e -> e.getLevel() == Level.DEBUG);
        assertFalse(temDebug, "Logs DEBUG não devem estar ativos em produção");
    }
}
```

### 23.5 Testando health checks (Actuator)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class HealthCheckTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void deveRetornarHealthUp() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/actuator/health", String.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertThatJson(response.getBody())
            .node("status").isEqualTo("UP");
    }

    @Test
    void deveIncluirComponentesNoHealthDetail() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/actuator/health", String.class);

        assertThatJson(response.getBody())
            .node("components.db.status").isEqualTo("UP")
            .node("components.diskSpace.status").isEqualTo("UP");
    }

    @Test
    void deveExporMetricasNoEndpointPrometheus() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/actuator/prometheus", String.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        String body = response.getBody();
        assertThat(body).contains("jvm_memory_used_bytes");
        assertThat(body).contains("http_server_requests_seconds");
    }
}
```

---

## 24. Cobertura de Código e Mutation Testing

### 24.1 JaCoCo — Cobertura de código

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <phase>verify</phase>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.70</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 24.2 Mutation Testing com Pitest

Mutation Testing vai além da cobertura de linhas — verifica se os testes realmente detectam bugs ao introduzir mutações no código:

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.16.1</version>
    <dependencies>
        <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>
    <configuration>
        <targetClasses>
            <param>com.example.service.*</param>
        </targetClasses>
        <targetTests>
            <param>com.example.service.*Test</param>
        </targetTests>
        <mutationThreshold>75</mutationThreshold>
        <mutators>
            <mutator>DEFAULTS</mutator>
        </mutators>
        <outputFormats>
            <param>HTML</param>
        </outputFormats>
    </configuration>
</plugin>
```

```bash
# Executar mutation testing
mvn test-compile org.pitest:pitest-maven:mutationCoverage
```

```
Exemplo de relatório Pitest:

┌──────────────────────────────────────────────┐
│          Mutation Testing Report              │
├──────────────────────┬───────────────────────┤
│ Mutações geradas     │ 120                   │
│ Mutações mortas      │ 105 (87.5%)           │
│ Mutações sobreviventes│ 15 (12.5%)           │
│ Sem cobertura        │ 0                     │
├──────────────────────┴───────────────────────┤
│ DescontoService.java                          │
│   Linha 42: substituiu > por >= → SOBREVIVEU │
│   → O teste não detecta boundary condition!   │
│   Linha 58: removeu chamada validate()        │
│   → SOBREVIVEU — falta teste de validação     │
└──────────────────────────────────────────────┘
```

---

## 25. Organização, Boas Práticas e CI/CD

### 25.1 Estrutura de diretórios recomendada

```
src/
├── main/java/com/example/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   └── model/
└── test/
    ├── java/com/example/
    │   ├── unit/                  # Testes unitários (sem Spring)
    │   │   ├── service/
    │   │   └── mapper/
    │   ├── integration/           # Testes de integração
    │   │   ├── controller/        # @WebMvcTest
    │   │   ├── repository/        # @DataJpaTest
    │   │   └── api/               # @SpringBootTest
    │   ├── e2e/                   # Testes end-to-end
    │   │   ├── selenium/
    │   │   └── playwright/
    │   ├── security/              # Testes de segurança
    │   ├── load/                  # Testes de carga (Gatling)
    │   └── support/               # Classes utilitárias de teste
    │       ├── AbstractIntegrationTest.java
    │       ├── TestDataFactory.java
    │       └── CustomAssertions.java
    └── resources/
        ├── application-test.yml
        └── sql/
            └── init-test-data.sql
```

### 25.2 Test Data Factory (Object Mother)

```java
public class TestDataFactory {

    public static Cliente clientePadrao() {
        return new Cliente(null, "Maria Silva", "maria@test.com", "01310-100");
    }

    public static Cliente clienteComId(Long id) {
        return new Cliente(id, "Maria Silva", "maria@test.com", "01310-100");
    }

    public static Pedido pedidoPendente(Cliente cliente) {
        Pedido pedido = new Pedido();
        pedido.setCliente(cliente);
        pedido.setStatus(StatusPedido.PENDENTE);
        pedido.setDataCriacao(LocalDateTime.now());
        pedido.setItens(List.of(itemPadrao()));
        pedido.calcularTotal();
        return pedido;
    }

    public static ItemPedido itemPadrao() {
        return new ItemPedido("PROD-001", 2, new BigDecimal("50.00"));
    }

    public static PedidoRequest pedidoRequest() {
        return new PedidoRequest(1L, List.of(new ItemPedido("PROD-001", 2, new BigDecimal("50.00"))));
    }
}
```

### 25.3 Separação de testes por profile Maven

```xml
<profiles>
    <!-- Testes unitários (default, rápido) -->
    <profile>
        <id>unit</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/unit/**Test.java</include>
                        </includes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>

    <!-- Testes de integração -->
    <profile>
        <id>integration</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-failsafe-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/integration/**IT.java</include>
                        </includes>
                    </configuration>
                    <executions>
                        <execution>
                            <goals>
                                <goal>integration-test</goal>
                                <goal>verify</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>

    <!-- Testes E2E -->
    <profile>
        <id>e2e</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-failsafe-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/e2e/**IT.java</include>
                        </includes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

```bash
# Executar cada camada separadamente
mvn test                           # Unitários (rápido)
mvn verify -P integration          # Integração
mvn verify -P e2e                  # End-to-end
mvn verify -P integration,e2e     # Ambos
```

### 25.4 Tags do JUnit 5 para categorização

```java
@Tag("unit")
class ProdutoServiceTest { /* ... */ }

@Tag("integration")
@Tag("database")
class ProdutoRepositoryTest { /* ... */ }

@Tag("e2e")
@Tag("slow")
class LoginE2ETest { /* ... */ }

@Tag("security")
class OwaspSecurityTest { /* ... */ }
```

```xml
<!-- Surefire — executar apenas testes com tag "unit" -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <groups>unit</groups>
        <excludedGroups>slow, e2e</excludedGroups>
    </configuration>
</plugin>
```

### 25.5 Pipeline CI/CD com testes

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Testes unitários
        run: mvn test -P unit

      - name: Upload cobertura (JaCoCo)
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-report
          path: target/site/jacoco/

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Testes de integração
        run: mvn verify -P integration
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb

  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: Install Playwright browsers
        run: mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="install --with-deps chromium"

      - name: Testes E2E
        run: mvn verify -P e2e

      - name: Upload screenshots de falha
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-screenshots
          path: target/screenshots/

  security-scan:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'

      - name: OWASP Dependency Check
        run: mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=8

      - name: SpotBugs Security
        run: mvn spotbugs:check
```

### 25.6 Boas práticas — Resumo

| Prática | Descrição |
|---------|-----------|
| **Teste uma coisa por vez** | Cada `@Test` valida um único comportamento |
| **Nomes descritivos** | `deveRetornar404QuandoProdutoNaoExiste` > `testGet` |
| **AAA / Given-When-Then** | Organizar em Arrange-Act-Assert |
| **Independência** | Testes não devem depender de ordem de execução |
| **Dados próprios** | Cada teste cria seus dados; não depender de estado compartilhado |
| **Sem lógica no teste** | Sem `if`, `for`, `try-catch` nos testes |
| **Evitar mocks em excesso** | Se precisa mockar 5+ dependências, a classe faz demais |
| **Teste o comportamento** | Não teste implementação interna (getters, setters) |
| **Teste o contrato** | Verificar entradas/saídas, não sequência de chamadas |
| **Fast feedback** | Unitários < 1s, Integração < 10s, E2E < 60s |
| **Determinismo** | Sem `Random`, `new Date()` ou dependência de rede nos unitários |
| **CI obrigatório** | Testes devem rodar em todo push/PR |

### 25.7 Checklist de testes por camada

```
┌─────────────────────────────────────────────────────────────┐
│                 Checklist de Testes                           │
├─────────────────────────────────────────────────────────────┤
│ □ Unitários                                                  │
│   □ Services (lógica de negócio, edge cases)                │
│   □ Mappers/DTOs (conversão correta)                        │
│   □ Validações (Bean Validation, regras custom)             │
│   □ Utilitários (formatação, cálculos)                      │
│                                                              │
│ □ Slice Tests                                                │
│   □ @WebMvcTest (controllers, validação de request/response)│
│   □ @DataJpaTest (queries, projections, specifications)     │
│   □ @JsonTest (serialização/deserialização)                 │
│                                                              │
│ □ Integração                                                 │
│   □ @SpringBootTest (fluxos completos via API)              │
│   □ Testcontainers (banco real, cache, mensageria)          │
│   □ Eventos e mensageria assíncrona                         │
│                                                              │
│ □ E2E                                                        │
│   □ Fluxos críticos de negócio (login, compra, checkout)    │
│   □ Multi-browser (Chrome, Firefox, Safari via Playwright)  │
│                                                              │
│ □ Segurança                                                  │
│   □ Autenticação e autorização                              │
│   □ OWASP Top 10 (injection, XSS, IDOR, CSRF)             │
│   □ Headers de segurança                                    │
│   □ Rate limiting                                            │
│   □ DAST scan (OWASP ZAP)                                  │
│                                                              │
│ □ Performance                                                │
│   □ Smoke test de carga                                     │
│   □ Load test com SLAs definidos                            │
│   □ Stress test (ponto de ruptura)                          │
│                                                              │
│ □ Qualidade                                                  │
│   □ Cobertura de código ≥ 80% (JaCoCo)                     │
│   □ Mutation testing ≥ 75% (Pitest)                         │
│   □ Testes determinísticos e independentes                  │
│   □ CI/CD com gates de qualidade                            │
└─────────────────────────────────────────────────────────────┘
```

---

> **Referências:**
> - [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
> - [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html)
> - [Spring Boot Testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
> - [Testcontainers](https://testcontainers.com/)
> - [Playwright for Java](https://playwright.dev/java/)
> - [Gatling](https://gatling.io/docs/)
> - [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
