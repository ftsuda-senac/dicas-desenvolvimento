# Tópicos Complementares para Desenvolvimento Web

> **Objetivo:** Reunir tópicos que complementam os documentos existentes do repositório, cobrindo áreas essenciais para o desenvolvedor web moderno: containerização, testes, CI/CD, mensageria, documentação de APIs, frameworks fullstack, performance e práticas de qualidade de código — com exemplos práticos em Java/Spring Boot e ecossistema frontend.

---

## Sumário

1. [Docker e Kubernetes](#1-docker-e-kubernetes)
2. [Testes em Java/Spring (JUnit 5, Mockito, Testcontainers)](#2-testes-em-javaspring-junit-5-mockito-testcontainers)
3. [Testes de Frontend (Vitest, Cypress, Playwright)](#3-testes-de-frontend-vitest-cypress-playwright)
4. [CI/CD — GitHub Actions, Jenkins e GitLab CI](#4-cicd--github-actions-jenkins-e-gitlab-ci)
5. [Git Avançado e Estratégias de Branching](#5-git-avançado-e-estratégias-de-branching)
6. [GraphQL com Spring Boot](#6-graphql-com-spring-boot)
7. [Mensageria na Prática (Kafka e RabbitMQ)](#7-mensageria-na-prática-kafka-e-rabbitmq)
8. [Flyway e Liquibase — Migrations e Versionamento de Schema](#8-flyway-e-liquibase--migrations-e-versionamento-de-schema)
9. [Documentação de APIs (OpenAPI/Swagger)](#9-documentação-de-apis-openapiswagger)
10. [Microsserviços com Spring Cloud](#10-microsserviços-com-spring-cloud)
11. [Feature Flags e Estratégias de Deploy](#11-feature-flags-e-estratégias-de-deploy)
12. [Clean Code e Code Review](#12-clean-code-e-code-review)
13. [Next.js e Nuxt.js — Frameworks Fullstack](#13-nextjs-e-nuxtjs--frameworks-fullstack)
14. [Tailwind CSS e Design Systems](#14-tailwind-css-e-design-systems)
15. [Progressive Web Apps (PWA)](#15-progressive-web-apps-pwa)
16. [Performance e Profiling Java](#16-performance-e-profiling-java)
17. [OAuth2 e OpenID Connect](#17-oauth2-e-openid-connect)
18. [Acessibilidade Web (WCAG)](#18-acessibilidade-web-wcag)

---

## 1. Docker e Kubernetes

### 1.1 Conceitos fundamentais do Docker

Docker é uma plataforma de containerização que empacota a aplicação junto com todas as suas dependências em um container isolado, garantindo que o comportamento seja idêntico em qualquer ambiente (desenvolvimento, homologação, produção).

```
┌──────────────────────────────────────────────┐
│               Host (SO)                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │Container1│  │Container2│  │Container3│   │
│  │ App Java │  │ Postgres │  │  Redis   │   │
│  │ JRE 21   │  │  16.x    │  │  7.x     │   │
│  └──────────┘  └──────────┘  └──────────┘   │
│         Docker Engine                         │
└──────────────────────────────────────────────┘
```

| Conceito | Descrição |
|----------|-----------|
| **Imagem** | Template imutável com o SO, runtime e aplicação |
| **Container** | Instância em execução de uma imagem |
| **Dockerfile** | Script de construção da imagem |
| **Volume** | Armazenamento persistente externo ao container |
| **Network** | Rede virtual que conecta containers |
| **Registry** | Repositório de imagens (Docker Hub, ECR, GCR) |

### 1.2 Dockerfile para Spring Boot

```dockerfile
# === Build stage ===
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN ./mvnw dependency:resolve -B
COPY src src
RUN ./mvnw package -DskipTests -B

# === Runtime stage ===
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Multi-stage build** reduz o tamanho da imagem final — a etapa de build usa o JDK completo, enquanto o runtime usa apenas o JRE.

### 1.3 Docker Compose para ambiente de desenvolvimento

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
      SPRING_PROFILES_ACTIVE: dev
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

### 1.4 Boas práticas de Dockerfile

| Prática | Motivo |
|---------|--------|
| Usar imagens `alpine` | Menor superfície de ataque e tamanho |
| Multi-stage build | Imagem final não contém compilador/código-fonte |
| Usuário não-root | Princípio do menor privilégio |
| `.dockerignore` | Evita copiar `target/`, `.git/`, `node_modules/` |
| Camadas ordenadas por frequência de mudança | Cache eficiente (dependências antes de código) |
| Health checks | Orquestrador sabe quando o container está pronto |

### 1.5 Kubernetes — Conceitos essenciais

Kubernetes (K8s) é um orquestrador de containers que automatiza deploy, escalonamento e gerenciamento de aplicações containerizadas.

```
┌─────────────────────────────────────────────────────────────┐
│                       Cluster K8s                            │
│                                                              │
│  Control Plane                                               │
│  ┌──────────┬──────────┬───────────┬──────────┐             │
│  │API Server│Scheduler │Controller │  etcd    │             │
│  └──────────┴──────────┴───────────┴──────────┘             │
│                                                              │
│  Worker Nodes                                                │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ Node 1          │  │ Node 2          │                   │
│  │ ┌─────┐ ┌─────┐│  │ ┌─────┐ ┌─────┐│                   │
│  │ │Pod A│ │Pod B││  │ │Pod C│ │Pod D││                   │
│  │ └─────┘ └─────┘│  │ └─────┘ └─────┘│                   │
│  │    kubelet      │  │    kubelet      │                   │
│  └─────────────────┘  └─────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

| Recurso | Descrição |
|---------|-----------|
| **Pod** | Menor unidade deployável; contém 1+ containers |
| **Deployment** | Declara o estado desejado (réplicas, imagem, etc.) |
| **Service** | Abstração de rede para acessar Pods (ClusterIP, NodePort, LoadBalancer) |
| **Ingress** | Roteamento HTTP externo (similar a um reverse proxy) |
| **ConfigMap / Secret** | Configuração externalizada e segredos |
| **PersistentVolumeClaim** | Armazenamento persistente |
| **HPA** | Horizontal Pod Autoscaler — escalonamento automático |
| **Namespace** | Isolamento lógico de recursos dentro do cluster |

### 1.6 Deployment e Service para Spring Boot

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-app
  template:
    metadata:
      labels:
        app: api-app
    spec:
      containers:
        - name: api-app
          image: registry.example.com/api-app:1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: api-app-service
spec:
  selector:
    app: api-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

### 1.7 Spring Boot com Kubernetes — Configurações recomendadas

```yaml
# application-prod.yml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s

server:
  shutdown: graceful

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
        readiness:
          include: db,redis
        liveness:
          include: ping
```

---

## 2. Testes em Java/Spring (JUnit 5, Mockito, Testcontainers)

### 2.1 Pirâmide de testes

```
        ╱╲
       ╱  ╲         E2E / UI
      ╱────╲         (poucos, lentos, frágeis)
     ╱      ╲
    ╱────────╲       Integração
   ╱          ╲       (moderados, testam interações)
  ╱────────────╲
 ╱              ╲    Unitários
╱────────────────╲    (muitos, rápidos, isolados)
```

| Nível | Escopo | Velocidade | Ferramentas |
|-------|--------|------------|-------------|
| **Unitário** | Classe/método isolado | Milissegundos | JUnit 5, Mockito |
| **Integração** | Componentes conectados | Segundos | Spring Boot Test, Testcontainers |
| **E2E** | Sistema completo | Minutos | REST Assured, Selenium |

### 2.2 JUnit 5 — Fundamentos

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class PedidoServiceTest {

    private PedidoService service;

    @BeforeEach
    void setUp() {
        service = new PedidoService();
    }

    @Test
    @DisplayName("deve calcular total com desconto para pedidos acima de R$100")
    void deveAplicarDescontoAcimaDeCem() {
        var pedido = new Pedido(List.of(
            new Item("Produto A", new BigDecimal("80.00"), 2)
        ));

        BigDecimal total = service.calcularTotal(pedido);

        assertEquals(new BigDecimal("144.00"), total); // 10% de desconto
    }

    @Test
    @DisplayName("deve lançar exceção para pedido sem itens")
    void deveLancarExcecaoParaPedidoVazio() {
        var pedido = new Pedido(List.of());

        assertThrows(PedidoInvalidoException.class,
            () -> service.calcularTotal(pedido));
    }

    @ParameterizedTest
    @CsvSource({
        "50.00,  1, 50.00",
        "100.00, 1, 90.00",
        "200.00, 2, 360.00"
    })
    @DisplayName("deve calcular total corretamente para diferentes cenários")
    void deveCalcularTotalParametrizado(
            String preco, int quantidade, String esperado) {
        var pedido = new Pedido(List.of(
            new Item("Produto", new BigDecimal(preco), quantidade)
        ));

        assertEquals(new BigDecimal(esperado), service.calcularTotal(pedido));
    }
}
```

### 2.3 Mockito — Simulando dependências

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {

    @Mock
    private PedidoRepository repository;

    @Mock
    private NotificacaoService notificacaoService;

    @InjectMocks
    private PedidoService service;

    @Test
    @DisplayName("deve salvar pedido e notificar cliente")
    void deveSalvarENotificar() {
        var pedido = new Pedido("cliente@email.com", List.of(
            new Item("Produto A", BigDecimal.TEN, 1)
        ));
        when(repository.save(any(Pedido.class))).thenReturn(pedido);

        service.finalizarPedido(pedido);

        verify(repository).save(pedido);
        verify(notificacaoService).enviar(
            eq("cliente@email.com"),
            contains("confirmado")
        );
    }

    @Test
    @DisplayName("não deve notificar se o salvamento falhar")
    void naoDeveNotificarSeFalhar() {
        when(repository.save(any())).thenThrow(new RuntimeException("DB error"));

        assertThrows(RuntimeException.class,
            () -> service.finalizarPedido(new Pedido()));

        verify(notificacaoService, never()).enviar(anyString(), anyString());
    }
}
```

### 2.4 Spring Boot Test — Testes de integração

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class PedidoControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PedidoRepository repository;

    @BeforeEach
    void setUp() {
        repository.deleteAll();
    }

    @Test
    @DisplayName("POST /api/pedidos deve criar pedido e retornar 201")
    void deveCriarPedido() {
        var request = new CriarPedidoRequest(
            "cliente@email.com",
            List.of(new ItemRequest("Produto A", "25.00", 2))
        );

        ResponseEntity<PedidoResponse> response = restTemplate.postForEntity(
            "/api/pedidos", request, PedidoResponse.class
        );

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody().id());
        assertEquals(1, repository.count());
    }
}
```

### 2.5 `@WebMvcTest` — Teste isolado do Controller

```java
@WebMvcTest(PedidoController.class)
class PedidoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PedidoService pedidoService;

    @Test
    void deveCriarPedido() throws Exception {
        when(pedidoService.criar(any())).thenReturn(
            new PedidoResponse(1L, "CRIADO", BigDecimal.valueOf(50))
        );

        mockMvc.perform(post("/api/pedidos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "email": "teste@email.com",
                        "itens": [{"nome": "A", "preco": "25.00", "qtd": 2}]
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.status").value("CRIADO"));
    }

    @Test
    void deveRetornar400ParaEmailInvalido() throws Exception {
        mockMvc.perform(post("/api/pedidos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"email": "invalido", "itens": []}
                    """))
            .andExpect(status().isBadRequest());
    }
}
```

### 2.6 `@DataJpaTest` — Teste isolado do Repository

```java
@DataJpaTest
class PedidoRepositoryTest {

    @Autowired
    private PedidoRepository repository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void deveBuscarPedidosPorStatus() {
        var pedido1 = entityManager.persist(new Pedido("CRIADO"));
        var pedido2 = entityManager.persist(new Pedido("FINALIZADO"));
        entityManager.flush();

        List<Pedido> criados = repository.findByStatus("CRIADO");

        assertEquals(1, criados.size());
        assertEquals(pedido1.getId(), criados.get(0).getId());
    }
}
```

### 2.7 Testcontainers — Banco de dados real nos testes

Testcontainers sobe containers Docker descartáveis durante a execução dos testes, garantindo que os testes de integração rodem contra um banco de dados real.

```xml
<!-- pom.xml -->
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

```java
@SpringBootTest
@Testcontainers
class PedidoRepositoryIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private PedidoRepository repository;

    @Test
    void devePersistirERecuperarPedido() {
        var pedido = repository.save(new Pedido("cliente@email.com", "CRIADO"));

        var encontrado = repository.findById(pedido.getId());

        assertTrue(encontrado.isPresent());
        assertEquals("CRIADO", encontrado.get().getStatus());
    }
}
```

### 2.8 Organização dos testes

```
src/test/java/
├── com.exemplo.app/
│   ├── unit/                    # Testes unitários (sem Spring)
│   │   ├── PedidoServiceTest.java
│   │   └── DescontoCalculatorTest.java
│   ├── integration/             # Testes de integração
│   │   ├── PedidoControllerIntegrationTest.java
│   │   └── PedidoRepositoryIntegrationTest.java
│   └── config/
│       └── TestcontainersConfiguration.java
```

---

## 3. Testes de Frontend (Vitest, Cypress, Playwright)

### 3.1 Vitest — Testes unitários para componentes

Vitest é o framework de testes recomendado para projetos Vite (React, Vue, Svelte). É compatível com a API do Jest, mas significativamente mais rápido.

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

```typescript
// src/components/Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Counter } from './Counter';

describe('Counter', () => {
  it('deve renderizar com valor inicial zero', () => {
    render(<Counter />);
    expect(screen.getByText('Contagem: 0')).toBeInTheDocument();
  });

  it('deve incrementar ao clicar no botão', () => {
    render(<Counter />);
    fireEvent.click(screen.getByRole('button', { name: /incrementar/i }));
    expect(screen.getByText('Contagem: 1')).toBeInTheDocument();
  });

  it('deve chamar callback onChange quando o valor muda', () => {
    const onChange = vi.fn();
    render(<Counter onChange={onChange} />);

    fireEvent.click(screen.getByRole('button', { name: /incrementar/i }));

    expect(onChange).toHaveBeenCalledWith(1);
  });
});
```

### 3.2 Testing Library — Princípios

A Testing Library segue o princípio de testar como o **usuário** interage com a aplicação, não detalhes de implementação.

| Seletor | Quando usar |
|---------|-------------|
| `getByRole` | Botões, links, inputs (preferido) |
| `getByLabelText` | Campos de formulário com label |
| `getByText` | Texto visível na tela |
| `getByPlaceholderText` | Fallback para inputs sem label |
| `getByTestId` | Último recurso (data-testid) |

```typescript
// Evite: testa implementação interna
const { container } = render(<UserList />);
const items = container.querySelectorAll('.user-item');

// Prefira: testa o que o usuário vê
const items = screen.getAllByRole('listitem');
```

### 3.3 Testando hooks customizados

```typescript
// src/hooks/useDebounce.test.ts
import { renderHook, act } from '@testing-library/react';
import { useDebounce } from './useDebounce';

describe('useDebounce', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('deve retornar o valor atualizado após o delay', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 500),
      { initialProps: { value: 'inicial' } }
    );

    expect(result.current).toBe('inicial');

    rerender({ value: 'atualizado' });
    expect(result.current).toBe('inicial'); // ainda não atualizou

    act(() => vi.advanceTimersByTime(500));
    expect(result.current).toBe('atualizado');
  });
});
```

### 3.4 Cypress — Testes E2E

Cypress executa testes no navegador real, simulando a interação do usuário com a aplicação.

```bash
npm install -D cypress
```

```typescript
// cypress/e2e/login.cy.ts
describe('Login', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('deve fazer login com credenciais válidas', () => {
    cy.get('[data-testid="email"]').type('usuario@email.com');
    cy.get('[data-testid="senha"]').type('senha123');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Bem-vindo').should('be.visible');
  });

  it('deve mostrar erro com credenciais inválidas', () => {
    cy.get('[data-testid="email"]').type('invalido@email.com');
    cy.get('[data-testid="senha"]').type('errada');
    cy.get('button[type="submit"]').click();

    cy.contains('Credenciais inválidas').should('be.visible');
    cy.url().should('include', '/login');
  });

  it('deve validar campo de email obrigatório', () => {
    cy.get('button[type="submit"]').click();
    cy.contains('Email é obrigatório').should('be.visible');
  });
});
```

### 3.5 Playwright — Testes E2E multi-navegador

Playwright permite testar em Chromium, Firefox e WebKit com uma única API.

```bash
npm init playwright@latest
```

```typescript
// tests/pedido.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Fluxo de Pedido', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/produtos');
  });

  test('deve adicionar produto ao carrinho e finalizar compra', async ({ page }) => {
    await page.getByRole('button', { name: 'Adicionar ao Carrinho' }).first().click();

    await page.getByRole('link', { name: 'Carrinho (1)' }).click();
    await expect(page.getByText('1 item no carrinho')).toBeVisible();

    await page.getByRole('button', { name: 'Finalizar Compra' }).click();
    await page.getByLabel('CEP').fill('01001-000');
    await page.getByRole('button', { name: 'Confirmar' }).click();

    await expect(page.getByText('Pedido realizado com sucesso')).toBeVisible();
  });
});
```

### 3.6 Comparativo de ferramentas de teste frontend

| Característica | Vitest | Cypress | Playwright |
|---------------|--------|---------|------------|
| **Tipo** | Unitário/Componente | E2E | E2E |
| **Velocidade** | Muito rápido | Moderado | Rápido |
| **Navegadores** | jsdom (simulado) | Chromium, Firefox, Edge | Chromium, Firefox, WebKit |
| **Paralelismo** | Sim | Pago (Dashboard) | Sim (nativo) |
| **Debugging** | Console | Time Travel GUI | Trace Viewer |
| **API** | Compatível Jest | Chainable | async/await |
| **CI** | Simples | Requer navegador | Requer navegador |

---

## 4. CI/CD — GitHub Actions, Jenkins e GitLab CI

### 4.1 Conceitos de CI/CD

```
Código ──▶ Build ──▶ Testes ──▶ Análise ──▶ Artefato ──▶ Deploy ──▶ Monitoramento
           │         │          │           │            │
           CI ───────┘──────────┘           │            │
                                            CD ──────────┘
```

| Conceito | Descrição |
|----------|-----------|
| **CI** (Continuous Integration) | Build e testes automatizados a cada push |
| **CD** (Continuous Delivery) | Artefato pronto para deploy a qualquer momento |
| **CD** (Continuous Deployment) | Deploy automático em produção após aprovação |
| **Pipeline** | Sequência de etapas automatizadas |
| **Artefato** | Resultado do build (JAR, imagem Docker, bundle JS) |

### 4.2 GitHub Actions — Pipeline completo para Spring Boot

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven

      - name: Run tests
        run: ./mvnw verify -B
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_PASSWORD: test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: target/surefire-reports/

  docker:
    needs: build-and-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### 4.3 GitHub Actions — Pipeline para projeto React/Vite

```yaml
# .github/workflows/frontend-ci.yml
name: Frontend CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npx tsc --noEmit

      - name: Unit tests
        run: npm run test -- --coverage

      - name: Build
        run: npm run build

  e2e:
    needs: lint-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - name: Playwright tests
        run: npx playwright install --with-deps && npx playwright test
```

### 4.4 Jenkins — Jenkinsfile declarativo

```groovy
// Jenkinsfile
pipeline {
    agent any

    tools {
        jdk 'jdk-21'
        maven 'maven-3.9'
    }

    environment {
        REGISTRY = 'registry.exemplo.com'
        IMAGE = "${REGISTRY}/api-app"
    }

    stages {
        stage('Build') {
            steps {
                sh './mvnw compile -B'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw verify -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            when { branch 'main' }
            steps {
                sh "docker build -t ${IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/api-app api-app=${IMAGE}:${BUILD_NUMBER}"
            }
        }
    }

    post {
        failure {
            mail to: 'equipe@exemplo.com',
                 subject: "Build falhou: ${currentBuild.fullDisplayName}",
                 body: "Verifique: ${env.BUILD_URL}"
        }
    }
}
```

### 4.5 GitLab CI — `.gitlab-ci.yml`

```yaml
stages:
  - build
  - test
  - docker
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  paths:
    - .m2/repository/

build:
  stage: build
  image: eclipse-temurin:21-jdk
  script:
    - ./mvnw compile -B

test:
  stage: test
  image: eclipse-temurin:21-jdk
  services:
    - postgres:16-alpine
  variables:
    POSTGRES_DB: testdb
    POSTGRES_PASSWORD: test
    SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/testdb
  script:
    - ./mvnw verify -B
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml

docker:
  stage: docker
  image: docker:24
  services:
    - docker:24-dind
  only:
    - main
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

---

## 5. Git Avançado e Estratégias de Branching

### 5.1 Estratégias de branching

#### Git Flow

```
main      ●─────────────────●──────────────────●
           \               / \                /
release     \         ●───●   \          ●───●
             \       /         \        /
develop       ●─●─●─●───●──●───●──●─●─●
               \     /         \    /
feature         ●─●─●           ●──●
```

| Branch | Propósito | Merge para |
|--------|-----------|------------|
| `main` | Código em produção | — |
| `develop` | Integração de features | `release` |
| `feature/*` | Desenvolvimento de funcionalidades | `develop` |
| `release/*` | Preparação para produção | `main` + `develop` |
| `hotfix/*` | Correções urgentes em produção | `main` + `develop` |

#### Trunk-Based Development

```
main      ●──●──●──●──●──●──●──●──●
            \  /     \  /
feature      ●        ●     (branches curtas, < 1 dia)
```

Indicado para equipes com CI/CD maduro e feature flags.

#### GitHub Flow

```
main      ●─────●─────●─────●
           \   /   \   /
feature     ●─●     ●─●     (PR + code review + merge)
```

Modelo simplificado: branch de `main`, PR com review, merge para `main`.

### 5.2 Conventional Commits

Formato padronizado de mensagens de commit que facilita geração de changelogs e versionamento semântico automático.

```
<tipo>(<escopo>): <descrição>

[corpo opcional]

[rodapé opcional]
```

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| `feat` | Nova funcionalidade | `feat(auth): adicionar login com Google` |
| `fix` | Correção de bug | `fix(api): corrigir paginação de pedidos` |
| `docs` | Documentação | `docs: atualizar README com instruções de deploy` |
| `refactor` | Refatoração sem mudança de comportamento | `refactor(service): extrair cálculo de desconto` |
| `test` | Adição/correção de testes | `test(pedido): adicionar testes de integração` |
| `chore` | Tarefas de manutenção | `chore: atualizar dependências do Maven` |
| `ci` | Mudanças no CI/CD | `ci: adicionar job de deploy no GitHub Actions` |

### 5.3 Comandos Git avançados úteis

```bash
# Rebase interativo — reorganizar commits antes do push
git rebase -i HEAD~3

# Cherry-pick — trazer commit específico de outra branch
git cherry-pick abc1234

# Stash — salvar alterações temporariamente
git stash push -m "trabalho em progresso no checkout"
git stash pop

# Bisect — encontrar commit que introduziu um bug
git bisect start
git bisect bad              # commit atual tem o bug
git bisect good v1.0.0      # versão sem o bug
# Git testa commits intermediários automaticamente

# Log visual compacto
git log --oneline --graph --all

# Reflog — histórico de movimentações do HEAD (útil para recuperar commits perdidos)
git reflog

# Blame — quem alterou cada linha
git blame -L 10,20 src/main/java/App.java

# Diff entre branches
git diff main...feature/login

# Reset misto — desfaz commit mas mantém mudanças
git reset --mixed HEAD~1
```

### 5.4 `.gitignore` recomendado para projetos Java/Spring

```gitignore
# Build
target/
build/
*.jar
*.war

# IDE
.idea/
*.iml
.vscode/
.settings/
.classpath
.project
*.swp

# OS
.DS_Store
Thumbs.db

# Environment
.env
*.env.local
application-local.yml

# Logs
*.log
logs/
```

---

## 6. GraphQL com Spring Boot

### 6.1 GraphQL vs REST

| Aspecto | REST | GraphQL |
|---------|------|---------|
| **Endpoints** | Múltiplos (`/users`, `/orders`) | Único (`/graphql`) |
| **Dados retornados** | Fixos por endpoint | Cliente escolhe os campos |
| **Over-fetching** | Comum | Eliminado |
| **Under-fetching** | Requer múltiplas chamadas | Uma query resolve |
| **Versionamento** | URLs (`/v1/`, `/v2/`) | Schema evolui sem versão |
| **Caching** | HTTP nativo (GET + ETag) | Requer estratégia específica |
| **Curva de aprendizado** | Baixa | Moderada |

### 6.2 Configuração com Spring for GraphQL

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

```graphql
# src/main/resources/graphql/schema.graphqls
type Query {
    produtos(categoria: String, page: Int = 0, size: Int = 10): ProdutoPage!
    produto(id: ID!): Produto
}

type Mutation {
    criarProduto(input: ProdutoInput!): Produto!
    atualizarEstoque(id: ID!, quantidade: Int!): Produto!
}

type Produto {
    id: ID!
    nome: String!
    preco: BigDecimal!
    categoria: Categoria!
    avaliacoes: [Avaliacao!]!
}

type Categoria {
    id: ID!
    nome: String!
}

type Avaliacao {
    id: ID!
    nota: Int!
    comentario: String
    autor: String!
}

type ProdutoPage {
    content: [Produto!]!
    totalElements: Int!
    totalPages: Int!
}

input ProdutoInput {
    nome: String!
    preco: BigDecimal!
    categoriaId: ID!
}

scalar BigDecimal
```

### 6.3 Controllers GraphQL

```java
@Controller
public class ProdutoGraphQLController {

    private final ProdutoService produtoService;
    private final AvaliacaoService avaliacaoService;

    public ProdutoGraphQLController(ProdutoService produtoService,
                                     AvaliacaoService avaliacaoService) {
        this.produtoService = produtoService;
        this.avaliacaoService = avaliacaoService;
    }

    @QueryMapping
    public ProdutoPage produtos(@Argument String categoria,
                                 @Argument int page,
                                 @Argument int size) {
        return produtoService.buscar(categoria, PageRequest.of(page, size));
    }

    @QueryMapping
    public Produto produto(@Argument Long id) {
        return produtoService.buscarPorId(id);
    }

    @MutationMapping
    public Produto criarProduto(@Argument ProdutoInput input) {
        return produtoService.criar(input);
    }

    @BatchMapping
    public Map<Produto, List<Avaliacao>> avaliacoes(List<Produto> produtos) {
        return avaliacaoService.buscarPorProdutos(produtos);
    }
}
```

O `@BatchMapping` resolve o problema N+1: em vez de buscar avaliações para cada produto individualmente, busca todas de uma vez.

### 6.4 Query do cliente

```graphql
# Buscar apenas os campos necessários — sem over-fetching
query {
  produtos(categoria: "ELETRONICOS", page: 0, size: 5) {
    content {
      id
      nome
      preco
      avaliacoes {
        nota
        comentario
      }
    }
    totalElements
  }
}
```

---

## 7. Mensageria na Prática (Kafka e RabbitMQ)

### 7.1 Quando usar mensageria

| Cenário | Benefício |
|---------|-----------|
| Processamento assíncrono | Resposta imediata ao usuário, trabalho em background |
| Desacoplamento de serviços | Produtor e consumidor independentes |
| Picos de carga | Fila absorve a demanda e processa no ritmo do consumidor |
| Event-driven architecture | Serviços reagem a eventos de domínio |
| Garantia de entrega | Mensagens persistidas até serem consumidas |

### 7.2 RabbitMQ vs Kafka

| Aspecto | RabbitMQ | Kafka |
|---------|----------|-------|
| **Modelo** | Message broker (fila) | Event streaming (log) |
| **Mensagem após consumo** | Removida da fila | Retida por período configurado |
| **Ordering** | Por fila | Por partição |
| **Throughput** | Milhares/s | Milhões/s |
| **Replay** | Não | Sim (consumer offset) |
| **Caso de uso** | Tarefas assíncronas, RPC | Streaming, event sourcing, analytics |
| **Complexidade** | Baixa | Alta |

### 7.3 RabbitMQ com Spring Boot

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

```java
// Configuração
@Configuration
public class RabbitConfig {

    public static final String PEDIDO_QUEUE = "pedido.criado";
    public static final String PEDIDO_EXCHANGE = "pedido.exchange";

    @Bean
    public Queue pedidoQueue() {
        return QueueBuilder.durable(PEDIDO_QUEUE).build();
    }

    @Bean
    public TopicExchange pedidoExchange() {
        return new TopicExchange(PEDIDO_EXCHANGE);
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("pedido.#");
    }

    @Bean
    public Jackson2JsonMessageConverter messageConverter(ObjectMapper mapper) {
        return new Jackson2JsonMessageConverter(mapper);
    }
}
```

```java
// Produtor
@Service
public class PedidoEventPublisher {

    private final RabbitTemplate rabbitTemplate;

    public PedidoEventPublisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void publicarPedidoCriado(PedidoCriadoEvent evento) {
        rabbitTemplate.convertAndSend(
            RabbitConfig.PEDIDO_EXCHANGE,
            "pedido.criado",
            evento
        );
    }
}
```

```java
// Consumidor
@Component
public class PedidoEventConsumer {

    private static final Logger log = LoggerFactory.getLogger(PedidoEventConsumer.class);

    private final NotificacaoService notificacaoService;

    public PedidoEventConsumer(NotificacaoService notificacaoService) {
        this.notificacaoService = notificacaoService;
    }

    @RabbitListener(queues = RabbitConfig.PEDIDO_QUEUE)
    public void processarPedidoCriado(PedidoCriadoEvent evento) {
        log.info("Pedido recebido: {}", evento.pedidoId());
        notificacaoService.enviarConfirmacao(evento);
    }
}
```

### 7.4 Kafka com Spring Boot

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: pedido-service
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.exemplo.eventos"
```

```java
// Produtor
@Service
public class PedidoKafkaProducer {

    private final KafkaTemplate<String, PedidoCriadoEvent> kafkaTemplate;

    public PedidoKafkaProducer(KafkaTemplate<String, PedidoCriadoEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publicar(PedidoCriadoEvent evento) {
        kafkaTemplate.send("pedido.criado", evento.pedidoId(), evento);
    }
}
```

```java
// Consumidor
@Component
public class PedidoKafkaConsumer {

    private static final Logger log = LoggerFactory.getLogger(PedidoKafkaConsumer.class);

    @KafkaListener(topics = "pedido.criado", groupId = "notificacao-service")
    public void processar(PedidoCriadoEvent evento) {
        log.info("Evento recebido: pedido={}", evento.pedidoId());
        // processar evento
    }

    @KafkaListener(topics = "pedido.criado", groupId = "analytics-service")
    public void registrarAnalytics(PedidoCriadoEvent evento) {
        // outro consumer group processa o mesmo evento independentemente
    }
}
```

---

## 8. Flyway e Liquibase — Migrations e Versionamento de Schema

### 8.1 Por que usar migrations

Migrations são scripts versionados que evoluem o schema do banco de dados de forma controlada, rastreável e reproduzível — como um "Git para o banco de dados".

| Sem migrations | Com migrations |
|----------------|----------------|
| Scripts manuais em produção | Versionamento automático |
| "Funciona na minha máquina" | Mesmo schema em todos os ambientes |
| Sem histórico de alterações | Histórico completo e auditável |
| Rollback manual e arriscado | Rollback controlado por script |

### 8.2 Flyway com Spring Boot

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

### 8.3 Convenção de nomes do Flyway

```
src/main/resources/db/migration/
├── V1__criar_tabela_usuario.sql
├── V2__criar_tabela_pedido.sql
├── V3__adicionar_coluna_telefone_usuario.sql
├── V4__criar_indice_pedido_status.sql
└── V5__inserir_dados_iniciais.sql
```

| Prefixo | Tipo | Descrição |
|---------|------|-----------|
| `V` | Versionada | Executada uma única vez, na ordem |
| `R` | Repetível | Re-executada quando o conteúdo muda (views, procedures) |

Formato: `V<versão>__<descricao>.sql` (dois underscores após a versão).

### 8.4 Exemplo de migrations

```sql
-- V1__criar_tabela_usuario.sql
CREATE TABLE usuario (
    id         BIGSERIAL    PRIMARY KEY,
    nome       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL UNIQUE,
    senha_hash VARCHAR(255) NOT NULL,
    ativo      BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_usuario_email ON usuario(email);
```

```sql
-- V2__criar_tabela_pedido.sql
CREATE TABLE pedido (
    id          BIGSERIAL      PRIMARY KEY,
    usuario_id  BIGINT         NOT NULL REFERENCES usuario(id),
    status      VARCHAR(20)    NOT NULL DEFAULT 'CRIADO',
    total       DECIMAL(12, 2) NOT NULL,
    criado_em   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP
);

CREATE INDEX idx_pedido_usuario ON pedido(usuario_id);
CREATE INDEX idx_pedido_status ON pedido(status);
```

```sql
-- V3__adicionar_coluna_telefone_usuario.sql
ALTER TABLE usuario ADD COLUMN telefone VARCHAR(20);
```

### 8.5 Liquibase — Alternativa ao Flyway

Liquibase usa um formato declarativo (XML, YAML ou JSON) que é independente de banco de dados.

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

```yaml
# src/main/resources/db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: equipe
      changes:
        - createTable:
            tableName: usuario
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nome
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
              - column:
                  name: email
                  type: VARCHAR(150)
                  constraints:
                    nullable: false
                    unique: true

  - changeSet:
      id: 2
      author: equipe
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: telefone
                  type: VARCHAR(20)
```

### 8.6 Flyway vs Liquibase

| Aspecto | Flyway | Liquibase |
|---------|--------|-----------|
| **Formato** | SQL puro | XML, YAML, JSON ou SQL |
| **Curva de aprendizado** | Baixa | Moderada |
| **Portabilidade** | Depende do dialeto SQL | Abstração multi-banco |
| **Rollback** | Versão paga | Nativo |
| **Comunidade Spring** | Mais popular | Bem suportado |
| **Recomendação** | Projetos simples, banco único | Multi-banco, rollback automático |

---

## 9. Documentação de APIs (OpenAPI/Swagger)

### 9.1 SpringDoc OpenAPI

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

```yaml
# application.yml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: method
```

Após subir a aplicação:
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- JSON OpenAPI: `http://localhost:8080/api-docs`

### 9.2 Anotações para documentação

```java
@RestController
@RequestMapping("/api/produtos")
@Tag(name = "Produtos", description = "Gerenciamento de produtos do catálogo")
public class ProdutoController {

    @Operation(
        summary = "Buscar produtos",
        description = "Retorna lista paginada de produtos com filtro opcional por categoria"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista retornada com sucesso"),
        @ApiResponse(responseCode = "400", description = "Parâmetros inválidos")
    })
    @GetMapping
    public Page<ProdutoResponse> listar(
            @Parameter(description = "Nome da categoria para filtrar")
            @RequestParam(required = false) String categoria,
            @ParameterObject Pageable pageable) {
        return produtoService.buscar(categoria, pageable);
    }

    @Operation(summary = "Criar produto")
    @ApiResponse(responseCode = "201", description = "Produto criado com sucesso")
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProdutoResponse criar(
            @Valid @RequestBody CriarProdutoRequest request) {
        return produtoService.criar(request);
    }
}
```

```java
@Schema(description = "Dados para criação de um produto")
public record CriarProdutoRequest(
    @Schema(description = "Nome do produto", example = "Notebook Dell XPS 15")
    @NotBlank
    String nome,

    @Schema(description = "Preço unitário", example = "4999.90")
    @NotNull @Positive
    BigDecimal preco,

    @Schema(description = "ID da categoria", example = "1")
    @NotNull
    Long categoriaId
) {}
```

### 9.3 Agrupamento de APIs e segurança

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("API do E-Commerce")
                .version("1.0")
                .description("API REST para gerenciamento de produtos e pedidos")
                .contact(new Contact()
                    .name("Equipe Backend")
                    .email("backend@exemplo.com")))
            .addSecurityItem(new SecurityRequirement().addList("Bearer"))
            .components(new Components()
                .addSecuritySchemes("Bearer",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

---

## 10. Microsserviços com Spring Cloud

### 10.1 Componentes do Spring Cloud

```
                         ┌───────────────┐
  Clientes ──────────▶   │  API Gateway  │
                         │ (Spring Cloud │
                         │   Gateway)    │
                         └──────┬────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                  │
      ┌───────▼──────┐  ┌──────▼───────┐  ┌──────▼───────┐
      │ Serviço A    │  │ Serviço B    │  │ Serviço C    │
      │ (Produtos)   │  │ (Pedidos)    │  │ (Pagamentos) │
      └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
             │                 │                  │
      ┌──────▼─────────────────▼──────────────────▼──────┐
      │              Service Discovery                    │
      │              (Eureka / Consul)                    │
      └──────────────────────┬───────────────────────────┘
                             │
      ┌──────────────────────▼───────────────────────────┐
      │              Config Server                        │
      │         (Spring Cloud Config)                     │
      └──────────────────────────────────────────────────┘
```

| Componente | Função | Implementação |
|------------|--------|---------------|
| **Service Discovery** | Registro e localização de serviços | Eureka, Consul |
| **API Gateway** | Roteamento, autenticação, rate limiting | Spring Cloud Gateway |
| **Config Server** | Configuração centralizada | Spring Cloud Config |
| **Circuit Breaker** | Resiliência a falhas | Resilience4j |
| **Distributed Tracing** | Rastreamento de requisições | Micrometer + Zipkin |
| **Load Balancer** | Distribuição de carga client-side | Spring Cloud LoadBalancer |

### 10.2 Eureka Server — Service Discovery

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml do Eureka Server
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

```yaml
# application.yml do microsserviço cliente
spring:
  application:
    name: produto-service
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### 10.3 Spring Cloud Gateway

```yaml
# application.yml do Gateway
spring:
  cloud:
    gateway:
      routes:
        - id: produto-service
          uri: lb://produto-service
          predicates:
            - Path=/api/produtos/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: produtoCircuitBreaker
                fallbackUri: forward:/fallback/produtos

        - id: pedido-service
          uri: lb://pedido-service
          predicates:
            - Path=/api/pedidos/**
          filters:
            - StripPrefix=1
```

### 10.4 Comunicação entre serviços com OpenFeign

```java
@FeignClient(name = "produto-service")
public interface ProdutoClient {

    @GetMapping("/produtos/{id}")
    ProdutoResponse buscarPorId(@PathVariable Long id);

    @PutMapping("/produtos/{id}/estoque")
    void atualizarEstoque(@PathVariable Long id,
                          @RequestBody EstoqueRequest request);
}
```

```java
@Service
public class PedidoService {

    private final ProdutoClient produtoClient;

    public PedidoService(ProdutoClient produtoClient) {
        this.produtoClient = produtoClient;
    }

    public PedidoResponse criarPedido(CriarPedidoRequest request) {
        ProdutoResponse produto = produtoClient.buscarPorId(request.produtoId());
        // ... criar pedido com dados do produto
    }
}
```

### 10.5 Resilience4j — Circuit Breaker

```java
@Service
public class ProdutoServiceClient {

    private final ProdutoClient produtoClient;

    public ProdutoServiceClient(ProdutoClient produtoClient) {
        this.produtoClient = produtoClient;
    }

    @CircuitBreaker(name = "produto", fallbackMethod = "fallbackBuscar")
    @Retry(name = "produto")
    public ProdutoResponse buscar(Long id) {
        return produtoClient.buscarPorId(id);
    }

    private ProdutoResponse fallbackBuscar(Long id, Throwable ex) {
        return new ProdutoResponse(id, "Produto indisponível", BigDecimal.ZERO);
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      produto:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      produto:
        max-attempts: 3
        wait-duration: 500ms
```

---

## 11. Feature Flags e Estratégias de Deploy

### 11.1 Estratégias de deploy

```
Blue-Green Deploy:
┌──────────┐     ┌──────────┐
│  Blue    │ ──▶ │  Green   │     Load Balancer alterna 100% do tráfego
│  (v1.0)  │     │  (v2.0)  │     de um ambiente para outro
└──────────┘     └──────────┘

Canary Deploy:
┌──────────┐     ┌──────────┐
│  Stable  │ 90% │  Canary  │ 10%   Tráfego gradual: 10% → 25% → 50% → 100%
│  (v1.0)  │ ──▶ │  (v2.0)  │ ──▶   Rollback automático se erro > threshold
└──────────┘     └──────────┘

Rolling Update:
Pod1 [v1] → [v2]
Pod2 [v1] → [v2]     Atualiza um pod por vez sem downtime
Pod3 [v1] → [v2]
```

| Estratégia | Downtime | Rollback | Custo | Risco |
|------------|----------|----------|-------|-------|
| **Blue-Green** | Zero | Instantâneo | 2x recursos | Baixo |
| **Canary** | Zero | Rápido | +10-20% recursos | Muito baixo |
| **Rolling** | Zero | Moderado | Mesmo | Moderado |
| **Recreate** | Sim | Lento | Mesmo | Alto |

### 11.2 Feature Flags — Conceito

Feature flags permitem ativar/desativar funcionalidades sem novo deploy, separando **deploy** de **release**.

```java
@RestController
@RequestMapping("/api/checkout")
public class CheckoutController {

    private final FeatureFlagService featureFlags;

    public CheckoutController(FeatureFlagService featureFlags) {
        this.featureFlags = featureFlags;
    }

    @PostMapping
    public ResponseEntity<?> checkout(@RequestBody CheckoutRequest request) {
        if (featureFlags.isEnabled("novo-gateway-pagamento")) {
            return processarComNovoGateway(request);
        }
        return processarComGatewayAtual(request);
    }
}
```

### 11.3 Implementação simples com Spring

```java
@Component
@ConfigurationProperties(prefix = "features")
public class FeatureFlagService {

    private Map<String, Boolean> flags = new HashMap<>();

    public Map<String, Boolean> getFlags() {
        return flags;
    }

    public void setFlags(Map<String, Boolean> flags) {
        this.flags = flags;
    }

    public boolean isEnabled(String flag) {
        return flags.getOrDefault(flag, false);
    }
}
```

```yaml
features:
  flags:
    novo-gateway-pagamento: false
    redesign-checkout: true
    relatorio-v2: false
```

Para cenários mais complexos (segmentação por usuário, A/B testing, porcentagem de rollout), considere ferramentas como **Unleash**, **LaunchDarkly** ou **Flagsmith**.

---

## 12. Clean Code e Code Review

### 12.1 Princípios de Clean Code

| Princípio | Ruim | Bom |
|-----------|------|-----|
| **Nomes significativos** | `int d; // dias` | `int diasDesdeUltimaCompra;` |
| **Funções pequenas** | Método com 100+ linhas | Métodos com 5-20 linhas, responsabilidade única |
| **Sem efeitos colaterais** | `getUsuario()` que também loga | Separar `getUsuario()` e `logAcesso()` |
| **DRY** | Mesmo cálculo em 3 lugares | Extrair para método reutilizável |
| **Fail fast** | Validação no final do método | Validação logo no início |

### 12.2 Exemplos práticos

```java
// Ruim: nomes genéricos, lógica acoplada
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<>();
    for (int[] x : theList) {
        if (x[0] == 4) list1.add(x);
    }
    return list1;
}

// Bom: nomes revelam a intenção
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<>();
    for (Cell cell : board) {
        if (cell.isFlagged()) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}

// Melhor ainda: com Streams
public List<Cell> getFlaggedCells() {
    return board.stream()
        .filter(Cell::isFlagged)
        .toList();
}
```

```java
// Ruim: método faz muitas coisas
public void processarPedido(Pedido pedido) {
    // valida
    if (pedido.getItens().isEmpty()) throw new RuntimeException("sem itens");
    if (pedido.getCliente() == null) throw new RuntimeException("sem cliente");

    // calcula
    double total = 0;
    for (Item item : pedido.getItens()) {
        total += item.getPreco() * item.getQuantidade();
    }
    if (total > 100) total *= 0.9;
    pedido.setTotal(total);

    // salva
    repository.save(pedido);

    // notifica
    emailService.enviar(pedido.getCliente().getEmail(), "Pedido confirmado");
}

// Bom: cada método tem uma responsabilidade
public void processarPedido(Pedido pedido) {
    validar(pedido);
    BigDecimal total = calcularTotal(pedido.getItens());
    pedido.setTotal(total);
    repository.save(pedido);
    notificacaoService.confirmarPedido(pedido);
}
```

### 12.3 Code Review — Checklist

| Categoria | O que verificar |
|-----------|----------------|
| **Correção** | A lógica resolve o problema? Trata edge cases? |
| **Segurança** | SQL injection, XSS, dados sensíveis expostos? |
| **Performance** | Queries N+1, loops desnecessários, objetos grandes? |
| **Legibilidade** | Nomes claros, métodos curtos, sem comentários obvios? |
| **Testes** | Cenários de sucesso e erro cobertos? |
| **Design** | Segue os padrões do projeto? Acoplamento adequado? |
| **Erros** | Exceções tratadas adequadamente? Logs informativos? |

### 12.4 Boas práticas de Code Review

**Para quem revisa:**
- Foque no que importa: bugs, segurança e design — não estilo pessoal
- Faça perguntas em vez de afirmações: "O que acontece se X for null?" em vez de "Isso vai quebrar"
- Elogie boas soluções — review não é só apontar problemas
- Sugira, não ordene: "Considere usar Optional aqui" em vez de "Mude para Optional"

**Para quem submete:**
- PRs pequenos (< 400 linhas) são revisados mais rápido e com mais atenção
- Descreva o contexto e decisões no PR — o que não é óbvio pelo código
- Responda comentários, mesmo que seja para explicar por que manteve a escolha

---

## 13. Next.js e Nuxt.js — Frameworks Fullstack

### 13.1 Por que frameworks fullstack

| Problema com SPA puro | Solução do framework fullstack |
|------------------------|-------------------------------|
| SEO ruim (JavaScript renderizado no cliente) | Server-Side Rendering (SSR) |
| Tempo de carregamento lento | Static Site Generation (SSG) |
| Sem API routes nativos | API Routes integradas |
| Configuração complexa de routing | File-based routing |
| Bundle grande | Code splitting automático |

### 13.2 Next.js (React) — App Router

```
app/
├── layout.tsx              # Layout raiz
├── page.tsx                # Rota /
├── loading.tsx             # Loading UI
├── error.tsx               # Error boundary
├── produtos/
│   ├── page.tsx            # /produtos
│   └── [id]/
│       └── page.tsx        # /produtos/123
└── api/
    └── produtos/
        └── route.ts        # API: /api/produtos
```

```tsx
// app/produtos/page.tsx — Server Component (padrão)
async function getProdutos() {
  const res = await fetch('https://api.exemplo.com/produtos', {
    next: { revalidate: 60 } // ISR: revalida a cada 60 segundos
  });
  return res.json();
}

export default async function ProdutosPage() {
  const produtos = await getProdutos();

  return (
    <main>
      <h1>Produtos</h1>
      <ul>
        {produtos.map((p: Produto) => (
          <li key={p.id}>
            <Link href={`/produtos/${p.id}`}>{p.nome}</Link>
            <span>R$ {p.preco}</span>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

```tsx
// app/produtos/[id]/page.tsx
interface Props {
  params: Promise<{ id: string }>;
}

export default async function ProdutoPage({ params }: Props) {
  const { id } = await params;
  const produto = await fetch(`https://api.exemplo.com/produtos/${id}`)
    .then(res => res.json());

  return (
    <article>
      <h1>{produto.nome}</h1>
      <p>{produto.descricao}</p>
      <p>R$ {produto.preco}</p>
      <AddToCartButton produtoId={produto.id} />
    </article>
  );
}
```

```tsx
// Client Component — interatividade no navegador
'use client';

import { useState } from 'react';

export function AddToCartButton({ produtoId }: { produtoId: number }) {
  const [adicionado, setAdicionado] = useState(false);

  async function handleClick() {
    await fetch('/api/carrinho', {
      method: 'POST',
      body: JSON.stringify({ produtoId }),
    });
    setAdicionado(true);
  }

  return (
    <button onClick={handleClick} disabled={adicionado}>
      {adicionado ? 'Adicionado' : 'Adicionar ao Carrinho'}
    </button>
  );
}
```

### 13.3 Nuxt.js (Vue) — Equivalente

```
pages/
├── index.vue               # /
├── produtos/
│   ├── index.vue            # /produtos
│   └── [id].vue             # /produtos/123
server/
├── api/
│   └── produtos.ts          # API: /api/produtos
```

```vue
<!-- pages/produtos/index.vue -->
<script setup lang="ts">
const { data: produtos } = await useFetch('/api/produtos');
</script>

<template>
  <main>
    <h1>Produtos</h1>
    <ul>
      <li v-for="p in produtos" :key="p.id">
        <NuxtLink :to="`/produtos/${p.id}`">{{ p.nome }}</NuxtLink>
        <span>R$ {{ p.preco }}</span>
      </li>
    </ul>
  </main>
</template>
```

### 13.4 Estratégias de renderização

| Estratégia | Sigla | Quando o HTML é gerado | Quando usar |
|------------|-------|------------------------|-------------|
| **Server-Side Rendering** | SSR | A cada requisição | Dados dinâmicos, personalização |
| **Static Site Generation** | SSG | No build | Blog, docs, landing pages |
| **Incremental Static Regen.** | ISR | Build + revalidação periódica | Catálogos, listagens |
| **Client-Side Rendering** | CSR | No navegador | Dashboards autenticados |

---

## 14. Tailwind CSS e Design Systems

### 14.1 Tailwind CSS — Utility-first

Tailwind aplica estilos diretamente via classes utilitárias, eliminando a necessidade de escrever CSS customizado na maioria dos casos.

```bash
npm install -D tailwindcss @tailwindcss/vite
```

```css
/* src/styles.css */
@import "tailwindcss";
```

```tsx
// Componente com Tailwind
function ProductCard({ produto }: { produto: Produto }) {
  return (
    <div className="rounded-lg border border-gray-200 bg-white p-4
                    shadow-sm transition hover:shadow-md">
      <img
        src={produto.imagem}
        alt={produto.nome}
        className="mb-3 h-48 w-full rounded object-cover"
      />
      <h3 className="text-lg font-semibold text-gray-900">
        {produto.nome}
      </h3>
      <p className="mt-1 text-sm text-gray-600">
        {produto.descricao}
      </p>
      <div className="mt-4 flex items-center justify-between">
        <span className="text-xl font-bold text-green-600">
          R$ {produto.preco}
        </span>
        <button className="rounded-md bg-blue-600 px-4 py-2 text-sm
                           font-medium text-white hover:bg-blue-700
                           focus:outline-none focus:ring-2
                           focus:ring-blue-500 focus:ring-offset-2">
          Comprar
        </button>
      </div>
    </div>
  );
}
```

### 14.2 CSS tradicional vs Tailwind

```css
/* CSS tradicional */
.card { border-radius: 8px; padding: 16px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.card:hover { box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.card-title { font-size: 1.125rem; font-weight: 600; color: #111; }
```

```html
<!-- Tailwind -->
<div class="rounded-lg p-4 shadow-sm hover:shadow-md">
  <h3 class="text-lg font-semibold text-gray-900">Título</h3>
</div>
```

### 14.3 Design Systems — Componentização

Um Design System é um conjunto de componentes reutilizáveis com padrões visuais, comportamentais e de acessibilidade consistentes.

```tsx
// Componente Button do Design System
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
}

const variants = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
  danger: 'bg-red-600 text-white hover:bg-red-700',
};

const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
};

export function Button({
  variant = 'primary',
  size = 'md',
  className,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`rounded-md font-medium transition focus:outline-none
                  focus:ring-2 focus:ring-offset-2
                  ${variants[variant]} ${sizes[size]} ${className ?? ''}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

### 14.4 Ferramentas populares de Design System

| Ferramenta | Descrição |
|------------|-----------|
| **shadcn/ui** | Componentes copiáveis (não é lib), Tailwind + Radix UI |
| **Radix UI** | Primitivos acessíveis e sem estilo |
| **Headless UI** | Componentes sem estilo do time Tailwind |
| **Material UI** | Design System do Google para React |
| **PrimeVue / PrimeReact** | Componentes ricos com temas customizáveis |

---

## 15. Progressive Web Apps (PWA)

### 15.1 O que é uma PWA

PWA (Progressive Web App) é uma aplicação web que oferece experiência similar a um app nativo: instalável, funciona offline e recebe notificações push.

| Recurso | Web tradicional | PWA | App nativo |
|---------|----------------|-----|------------|
| Instalável | Não | Sim | Sim |
| Offline | Não | Sim | Sim |
| Push notifications | Não | Sim | Sim |
| Acesso à câmera | Limitado | Sim | Sim |
| Loja de apps | Não | Opcional | Sim |
| Atualização | Automática | Automática | Manual |
| Custo de desenvolvimento | Baixo | Baixo | Alto (iOS + Android) |

### 15.2 Service Worker — Cache e offline

```typescript
// sw.ts — Service Worker básico
const CACHE_NAME = 'app-v1';
const ASSETS = ['/', '/index.html', '/styles.css', '/app.js'];

self.addEventListener('install', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS))
  );
});

self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request)
      .then(cached => cached || fetch(event.request))
  );
});
```

### 15.3 Web App Manifest

```json
{
  "name": "Minha Aplicação",
  "short_name": "MeuApp",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "icons": [
    { "src": "/icons/192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### 15.4 PWA com Vite

```bash
npm install -D vite-plugin-pwa
```

```typescript
// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'Minha Aplicação',
        short_name: 'MeuApp',
        theme_color: '#2563eb',
      },
      workbox: {
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.exemplo\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-cache',
              expiration: { maxEntries: 50, maxAgeSeconds: 300 },
            },
          },
        ],
      },
    }),
  ],
});
```

### 15.5 Estratégias de cache

| Estratégia | Comportamento | Quando usar |
|------------|---------------|-------------|
| **Cache First** | Cache → rede (se não encontrar) | Assets estáticos (CSS, JS, imagens) |
| **Network First** | Rede → cache (se offline) | APIs, dados dinâmicos |
| **Stale While Revalidate** | Cache imediato + atualiza em background | Dados semi-estáticos |
| **Network Only** | Sempre rede | Transações, dados em tempo real |
| **Cache Only** | Sempre cache | Assets versionados no build |

---

## 16. Performance e Profiling Java

### 16.1 Ferramentas de profiling

| Ferramenta | Tipo | Gratuita | Quando usar |
|------------|------|----------|-------------|
| **VisualVM** | Profiler geral | Sim | Análise de memória e CPU |
| **JFR (Java Flight Recorder)** | Profiler de baixo overhead | Sim (JDK 11+) | Produção |
| **async-profiler** | Sampling profiler | Sim | Flame graphs, CPU profiling |
| **JMH** | Micro-benchmarking | Sim | Comparar implementações |
| **Micrometer + Prometheus** | Métricas de aplicação | Sim | Monitoramento contínuo |

### 16.2 Flags JVM essenciais

```bash
# Memória
java -Xms512m -Xmx1g -jar app.jar

# GC logging (JDK 17+)
java -Xlog:gc*:file=gc.log:time,uptime,level,tags -jar app.jar

# Heap dump automático em OutOfMemoryError
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/dumps/ \
     -jar app.jar

# Flight Recorder
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr \
     -jar app.jar
```

### 16.3 JMH — Micro-benchmarks

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
    <scope>test</scope>
</dependency>
```

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Thread)
@Warmup(iterations = 3)
@Measurement(iterations = 5)
public class StringConcatBenchmark {

    private List<String> palavras;

    @Setup
    public void setup() {
        palavras = List.of("Spring", "Boot", "é", "rápido", "e", "produtivo");
    }

    @Benchmark
    public String comConcat() {
        String resultado = "";
        for (String p : palavras) {
            resultado += p + " ";
        }
        return resultado;
    }

    @Benchmark
    public String comStringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (String p : palavras) {
            sb.append(p).append(" ");
        }
        return sb.toString();
    }

    @Benchmark
    public String comStringJoin() {
        return String.join(" ", palavras);
    }
}
```

### 16.4 Otimizações comuns em Spring Boot

| Problema | Diagnóstico | Solução |
|----------|-------------|---------|
| **Query N+1** | Log SQL com `spring.jpa.show-sql=true` | `@EntityGraph`, `JOIN FETCH`, `@BatchSize` |
| **Startup lento** | Spring Boot Startup Actuator | Lazy initialization, GraalVM Native |
| **Memória alta** | Heap dump + VisualVM | Ajustar pool sizes, evitar cache ilimitado |
| **Conexões esgotadas** | Hikari metrics | Ajustar `maximum-pool-size`, timeout |
| **Serialização lenta** | Profiler CPU | `@JsonView` para limitar campos, DTOs |
| **Thread starvation** | Thread dump | Virtual threads (JDK 21), async processing |

### 16.5 Virtual Threads (JDK 21+)

```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true
```

Virtual threads permitem que o Spring Boot use threads leves gerenciadas pela JVM, melhorando a escalabilidade de aplicações com muitas operações bloqueantes (I/O de banco, chamadas HTTP).

```java
// Antes (Platform Threads): 200 threads no pool = 200 requisições simultâneas
// Depois (Virtual Threads): milhares de requisições simultâneas com baixo overhead
```

---

## 17. OAuth2 e OpenID Connect

### 17.1 Fluxos OAuth2

```
Authorization Code Flow (recomendado para web apps):

Usuário ──▶ App ──▶ Authorization Server ──▶ Login ──▶ Consent
                                                          │
                          ┌──────────── code ◀─────────────┘
                          │
                    App + code ──▶ Authorization Server
                                         │
                                   access_token + id_token
                                         │
                              App ◀──────┘
                              │
                        API ◀─┘ (Bearer token)
```

| Fluxo | Quando usar |
|-------|-------------|
| **Authorization Code + PKCE** | SPAs, aplicações web (recomendado) |
| **Client Credentials** | Comunicação máquina-a-máquina (serviços) |
| **Device Code** | Dispositivos sem teclado (Smart TV, CLI) |

### 17.2 OAuth2 vs OpenID Connect

| Conceito | OAuth2 | OpenID Connect |
|----------|--------|----------------|
| **Propósito** | Autorização (acesso a recursos) | Autenticação (identidade do usuário) |
| **Token** | Access Token | Access Token + ID Token (JWT) |
| **Endpoint** | `/authorize`, `/token` | + `/userinfo`, `/.well-known/openid-configuration` |
| **Claim** | `scope` | `sub`, `name`, `email`, `roles` |

### 17.3 Spring Boot como Resource Server

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8180/realms/meu-realm
```

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtConverter()))
            )
            .build();
    }

    private JwtAuthenticationConverter jwtConverter() {
        var grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        var converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return converter;
    }
}
```

### 17.4 Spring Boot como OAuth2 Client

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid, profile, email
          keycloak:
            client-id: meu-app
            client-secret: ${KC_SECRET}
            scope: openid, profile
            authorization-grant-type: authorization_code
        provider:
          keycloak:
            issuer-uri: http://localhost:8180/realms/meu-realm
```

### 17.5 Client Credentials — Comunicação entre serviços

```java
@Configuration
public class OAuth2ClientConfig {

    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository registrations,
            OAuth2AuthorizedClientRepository authorizedClients) {

        var provider = OAuth2AuthorizedClientProviderBuilder.builder()
            .clientCredentials()
            .build();

        var manager = new DefaultOAuth2AuthorizedClientManager(
            registrations, authorizedClients
        );
        manager.setAuthorizedClientProvider(provider);
        return manager;
    }
}
```

---

## 18. Acessibilidade Web (WCAG)

### 18.1 Princípios WCAG (POUR)

| Princípio | Descrição | Exemplos |
|-----------|-----------|----------|
| **Perceptível** | Conteúdo apresentável de formas alternativas | Alt em imagens, legendas em vídeos, contraste |
| **Operável** | Interface navegável por diferentes meios | Teclado, tempo suficiente, sem flashes |
| **Compreensível** | Conteúdo e operação previsíveis | Linguagem clara, validação de formulários |
| **Robusto** | Compatível com tecnologias assistivas | HTML semântico, ARIA quando necessário |

### 18.2 Níveis de conformidade

| Nível | Descrição | Obrigatório por lei (Brasil - LBI 13.146/2015) |
|-------|-----------|------------------------------------------------|
| **A** | Requisitos mínimos | Sim |
| **AA** | Recomendado (padrão da maioria das legislações) | Sim |
| **AAA** | Nível mais alto (nem sempre possível) | Não |

### 18.3 HTML semântico — A base da acessibilidade

```html
<!-- Ruim: div para tudo -->
<div class="header">
  <div class="nav">
    <div class="link" onclick="goto('/')">Home</div>
  </div>
</div>
<div class="main">
  <div class="title">Produtos</div>
  <div class="card" onclick="select(1)">
    <div class="img"><img src="foto.jpg"></div>
    <div>Produto A</div>
  </div>
</div>

<!-- Bom: elementos semânticos -->
<header>
  <nav aria-label="Principal">
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <h1>Produtos</h1>
  <article>
    <img src="foto.jpg" alt="Notebook Dell XPS 15 prateado, aberto">
    <h2>Produto A</h2>
    <button type="button">Selecionar</button>
  </article>
</main>
```

### 18.4 Formulários acessíveis

```html
<!-- Labels explícitas e mensagens de erro associadas -->
<form>
  <div>
    <label for="email">E-mail</label>
    <input
      id="email"
      type="email"
      required
      aria-describedby="email-error"
      aria-invalid="true"
    >
    <span id="email-error" role="alert">
      Por favor, insira um e-mail válido.
    </span>
  </div>

  <fieldset>
    <legend>Forma de pagamento</legend>
    <label>
      <input type="radio" name="pagamento" value="cartao"> Cartão de crédito
    </label>
    <label>
      <input type="radio" name="pagamento" value="boleto"> Boleto
    </label>
    <label>
      <input type="radio" name="pagamento" value="pix"> PIX
    </label>
  </fieldset>

  <button type="submit">Finalizar pedido</button>
</form>
```

### 18.5 ARIA — Quando e como usar

**Regra de ouro:** Não use ARIA se um elemento HTML nativo já cumpre o papel.

| Situação | Sem ARIA (preferível) | Com ARIA (quando necessário) |
|----------|----------------------|------------------------------|
| Botão | `<button>Fechar</button>` | `<div role="button" tabindex="0">Fechar</div>` |
| Navegação | `<nav>` | `<div role="navigation">` |
| Link | `<a href="...">` | `<span role="link" tabindex="0">` |
| Modal | — | `role="dialog"` + `aria-modal="true"` + `aria-labelledby` |

```tsx
// Modal acessível em React
function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const titleId = useId();

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
    >
      <h2 id={titleId}>{title}</h2>
      <div>{children}</div>
      <button onClick={onClose} aria-label="Fechar modal">
        &times;
      </button>
    </div>
  );
}
```

### 18.6 Contraste e cores

| Nível | Texto normal | Texto grande (18px+ bold, 24px+) |
|-------|-------------|----------------------------------|
| **AA** | 4.5:1 | 3:1 |
| **AAA** | 7:1 | 4.5:1 |

```css
/* Exemplo de cores com contraste adequado (AA) */
:root {
  --text-primary: #1a1a1a;    /* sobre branco = 16.6:1 */
  --text-secondary: #525252;  /* sobre branco = 7.4:1  */
  --text-on-blue: #ffffff;    /* sobre #2563eb = 5.1:1  */
  --error: #dc2626;           /* sobre branco = 4.6:1   */
}

/* Nunca dependa só de cor para transmitir informação */
.campo-invalido {
  border-color: var(--error);
  border-width: 2px;          /* reforço visual além da cor */
}
.campo-invalido::after {
  content: "⚠";               /* ícone como indicador adicional */
}
```

### 18.7 Ferramentas de teste de acessibilidade

| Ferramenta | Tipo | Uso |
|------------|------|-----|
| **axe DevTools** | Extensão do navegador | Auditoria automática da página |
| **Lighthouse** | Chrome DevTools | Score de acessibilidade |
| **WAVE** | Extensão/online | Visualização de problemas |
| **eslint-plugin-jsx-a11y** | Linter | Erros de acessibilidade no JSX |
| **Pa11y** | CLI/CI | Testes automatizados em pipeline |
| **NVDA / VoiceOver** | Screen reader | Teste real com leitor de tela |

### 18.8 Checklist mínimo de acessibilidade

- [ ] Todas as imagens têm `alt` descritivo (ou `alt=""` se decorativas)
- [ ] Toda a aplicação é navegável apenas por teclado
- [ ] Contraste mínimo de 4.5:1 para texto
- [ ] Formulários têm labels visíveis associadas
- [ ] Mensagens de erro são anunciadas por screen readers (`role="alert"`)
- [ ] A ordem de foco (tab order) é lógica e previsível
- [ ] Não há conteúdo que pisca mais de 3 vezes por segundo
- [ ] Headings (`h1`-`h6`) seguem hierarquia lógica
- [ ] Links e botões têm texto descritivo (evitar "clique aqui")
- [ ] A aplicação funciona com zoom de 200%

---

## Referências e próximos passos

| Tópico | Referência principal |
|--------|---------------------|
| Docker | [docs.docker.com](https://docs.docker.com) |
| Kubernetes | [kubernetes.io/docs](https://kubernetes.io/docs) |
| JUnit 5 | [junit.org/junit5](https://junit.org/junit5/docs/current/user-guide/) |
| Testcontainers | [testcontainers.com](https://testcontainers.com) |
| GitHub Actions | [docs.github.com/actions](https://docs.github.com/en/actions) |
| Spring for GraphQL | [spring.io/projects/spring-graphql](https://spring.io/projects/spring-graphql) |
| Spring Cloud | [spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud) |
| Flyway | [flywaydb.org](https://flywaydb.org) |
| SpringDoc OpenAPI | [springdoc.org](https://springdoc.org) |
| Next.js | [nextjs.org/docs](https://nextjs.org/docs) |
| Nuxt.js | [nuxt.com/docs](https://nuxt.com/docs) |
| Tailwind CSS | [tailwindcss.com/docs](https://tailwindcss.com/docs) |
| WCAG | [w3.org/WAI/WCAG22](https://www.w3.org/WAI/WCAG22/quickref/) |
