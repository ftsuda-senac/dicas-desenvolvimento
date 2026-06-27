# CI/CD — GitHub Actions e GitLab CI/CD — Guia Prático

> **Objetivo:** Cobrir de forma abrangente os conceitos de Integração Contínua (CI) e Entrega/Deploy Contínuo (CD), com implementações práticas usando GitHub Actions e GitLab CI/CD — desde pipelines básicos até cenários avançados com cache, matrices, ambientes, secrets, deploy em cloud e boas práticas — com exemplos voltados para projetos Java/Spring Boot e aplicações web modernas.

> **Pré-requisitos:** familiaridade com Git ([Git-Estrategias-Branching.md](Git-Estrategias-Branching.md)), Docker ([Docker-Compose-Kubernetes.md](Docker-Compose-Kubernetes.md)) e conceitos básicos de YAML.

---

## Sumário

1. [Fundamentos de CI/CD](#1-fundamentos-de-cicd)
2. [Conceitos Comuns](#2-conceitos-comuns)
3. [GitHub Actions — Arquitetura e Conceitos](#3-github-actions--arquitetura-e-conceitos)
4. [GitHub Actions — Sintaxe do Workflow](#4-github-actions--sintaxe-do-workflow)
5. [GitHub Actions — Exemplos Práticos](#5-github-actions--exemplos-práticos)
6. [GitHub Actions — Cache e Artefatos](#6-github-actions--cache-e-artefatos)
7. [GitHub Actions — Matrices e Estratégias](#7-github-actions--matrices-e-estratégias)
8. [GitHub Actions — Secrets e Variáveis de Ambiente](#8-github-actions--secrets-e-variáveis-de-ambiente)
9. [GitHub Actions — Environments e Deploy](#9-github-actions--environments-e-deploy)
10. [GitHub Actions — Actions Reutilizáveis e Composite Actions](#10-github-actions--actions-reutilizáveis-e-composite-actions)
11. [GitHub Actions — Workflows Reutilizáveis](#11-github-actions--workflows-reutilizáveis)
12. [GitLab CI/CD — Arquitetura e Conceitos](#12-gitlab-cicd--arquitetura-e-conceitos)
13. [GitLab CI/CD — Sintaxe do .gitlab-ci.yml](#13-gitlab-cicd--sintaxe-do-gitlab-ciyml)
14. [GitLab CI/CD — Exemplos Práticos](#14-gitlab-cicd--exemplos-práticos)
15. [GitLab CI/CD — Cache e Artefatos](#15-gitlab-cicd--cache-e-artefatos)
16. [GitLab CI/CD — Variáveis e Secrets](#16-gitlab-cicd--variáveis-e-secrets)
17. [GitLab CI/CD — Environments e Deploy](#17-gitlab-cicd--environments-e-deploy)
18. [GitLab CI/CD — Templates e Includes](#18-gitlab-cicd--templates-e-includes)
19. [GitLab CI/CD — Pipelines Pai-Filho e Multi-Projeto](#19-gitlab-cicd--pipelines-pai-filho-e-multi-projeto)
20. [Comparação GitHub Actions vs GitLab CI/CD](#20-comparação-github-actions-vs-gitlab-cicd)
21. [Pipeline Completo — Spring Boot (GitHub Actions)](#21-pipeline-completo--spring-boot-github-actions)
22. [Pipeline Completo — Spring Boot (GitLab CI/CD)](#22-pipeline-completo--spring-boot-gitlab-cicd)
23. [Pipeline para Frontend (React/Angular/Vue)](#23-pipeline-para-frontend-reactangularvue)
24. [Pipeline para Mobile — React Native e Flutter](#24-pipeline-para-mobile--react-native-e-flutter)
25. [Testes Avançados no Pipeline — E2E, Performance e Quality Gates](#25-testes-avançados-no-pipeline--e2e-performance-e-quality-gates)
26. [Deploy em Cloud — AWS, Azure e GCP](#26-deploy-em-cloud--aws-azure-e-gcp)
27. [Estratégias de Deploy — Blue-Green, Canary e Rollback](#27-estratégias-de-deploy--blue-green-canary-e-rollback)
28. [Database Migrations no Pipeline — Flyway e Liquibase](#28-database-migrations-no-pipeline--flyway-e-liquibase)
29. [Feature Flags e Progressive Delivery](#29-feature-flags-e-progressive-delivery)
30. [Infrastructure as Code no Pipeline — Terraform](#30-infrastructure-as-code-no-pipeline--terraform)
31. [GitOps com ArgoCD e Flux](#31-gitops-com-argocd-e-flux)
32. [Segurança no Pipeline — SAST, SCA e Secrets](#32-segurança-no-pipeline--sast-sca-e-secrets)
33. [Compliance, Auditoria e SBOM](#33-compliance-auditoria-e-sbom)
34. [Gerenciamento de Artefatos e Documentação Automática](#34-gerenciamento-de-artefatos-e-documentação-automática)
35. [Dependências Automáticas — Dependabot e Renovate](#35-dependências-automáticas--dependabot-e-renovate)
36. [Monorepo — Estratégias de Pipeline](#36-monorepo--estratégias-de-pipeline)
37. [Self-hosted Runners e Otimização de Custos](#37-self-hosted-runners-e-otimização-de-custos)
38. [Observabilidade e Notificações](#38-observabilidade-e-notificações)
39. [Boas Práticas e Checklist](#39-boas-práticas-e-checklist)
40. [GitHub — Gerenciamento de Projetos](#40-github--gerenciamento-de-projetos)
41. [GitLab — Gerenciamento de Projetos](#41-gitlab--gerenciamento-de-projetos)
42. [Automações de Gerenciamento via CI/CD](#42-automações-de-gerenciamento-via-cicd)
43. [Integração Completa — Do Commit ao Board](#43-integração-completa--do-commit-ao-board)
44. [GitLab CE On-Premise — Dimensionamento de Hardware](#44-gitlab-ce-on-premise--dimensionamento-de-hardware)
45. [GitLab CE On-Premise — Roteiro de Instalação](#45-gitlab-ce-on-premise--roteiro-de-instalação)

---

## 1. Fundamentos de CI/CD

### 1.1 O que é CI/CD

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        Pipeline CI/CD                                        │
│                                                                              │
│   ┌─────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│   │  Code    │──▶│  Build   │──▶│  Test    │──▶│ Package  │──▶│  Deploy  │  │
│   │  Commit  │   │  Compile │   │  Unit    │   │ Docker   │   │  Staging │  │
│   │         │   │  Lint    │   │  Integr  │   │ Artifact │   │  Prod    │  │
│   └─────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
│                                                                              │
│   ◀─────── Integração Contínua (CI) ───────▶  ◀── Entrega/Deploy (CD) ──▶  │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Conceito | Descrição |
|---|---|
| **Integração Contínua (CI)** | Prática de integrar código frequentemente com build e testes automatizados |
| **Entrega Contínua (CD)** | Extensão do CI — artefato pronto para deploy a qualquer momento (aprovação manual) |
| **Deploy Contínuo** | Extensão da entrega contínua — deploy automático em produção após aprovação nos testes |

### 1.2 Benefícios

- **Detecção precoce de bugs** — falhas são encontradas minutos após o commit
- **Feedback rápido** — desenvolvedor sabe imediatamente se o código quebra algo
- **Releases frequentes** — menor risco por deploy, rollback mais simples
- **Padronização** — todo código passa pelo mesmo fluxo de qualidade
- **Documentação viva** — pipeline como código descreve o processo de entrega

### 1.3 Fluxo Típico

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  Developer ──▶ Push/PR ──▶ CI Pipeline ──▶ CD Pipeline ──▶ Produção         │
│                                │                │                            │
│                           ┌────┴────┐      ┌────┴────┐                      │
│                           │ ✗ Falha │      │ ✗ Falha │                      │
│                           │ Notifica│      │ Rollback│                      │
│                           └─────────┘      └─────────┘                      │
│                                                                              │
│  CI:  Lint → Build → Test Unitário → Test Integração → SAST/SCA            │
│  CD:  Build Image → Push Registry → Deploy Staging → Smoke Test → Prod     │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Conceitos Comuns

### 2.1 Terminologia

| Termo | GitHub Actions | GitLab CI/CD |
|---|---|---|
| Arquivo de configuração | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| Unidade de execução | **Job** | **Job** |
| Agrupamento de jobs | **Workflow** | **Pipeline** |
| Executor | **Runner** | **Runner** |
| Etapa dentro de um job | **Step** | **Script line** |
| Agrupamento lógico | — | **Stage** |
| Trigger | **Event** (push, PR, schedule) | **Trigger** (push, MR, schedule, API) |
| Variáveis seguras | **Secrets** | **CI/CD Variables** (masked/protected) |
| Reutilização | **Reusable Workflows / Composite Actions** | **includes / extends** |

### 2.2 Pipeline como Código

Ambas as plataformas usam YAML para definir pipelines de forma declarativa, versionada junto ao código-fonte:

```yaml
# Estrutura conceitual comum
trigger: "quando executar"
stages: "em que ordem"
jobs:
  job_name:
    stage: "fase"
    environment: "onde"
    steps/script: "o que fazer"
    artifacts: "o que preservar"
```

---

## 3. GitHub Actions — Arquitetura e Conceitos

### 3.1 Componentes

```
┌──────────────────────────────────────────────────────────────────────┐
│                     GitHub Actions                                    │
│                                                                      │
│  ┌───────────┐    ┌───────────────────────────────────────────────┐  │
│  │  Event     │───▶│  Workflow (.github/workflows/*.yml)          │  │
│  │  (push,PR) │    │                                               │  │
│  └───────────┘    │  ┌─────────┐  ┌─────────┐  ┌─────────┐      │  │
│                    │  │  Job 1  │  │  Job 2  │  │  Job 3  │      │  │
│                    │  │         │  │         │  │         │      │  │
│                    │  │ Step 1  │  │ Step 1  │  │ Step 1  │      │  │
│                    │  │ Step 2  │  │ Step 2  │  │ Step 2  │      │  │
│                    │  │ Step 3  │  │         │  │         │      │  │
│                    │  └────┬────┘  └────┬────┘  └─────────┘      │  │
│                    │       │            │                          │  │
│                    │       ▼            ▼                          │  │
│                    │  ┌─────────┐  ┌─────────┐                    │  │
│                    │  │Runner 1 │  │Runner 2 │  (paralelo)        │  │
│                    │  │(ubuntu) │  │(windows)│                    │  │
│                    │  └─────────┘  └─────────┘                    │  │
│                    └───────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

| Componente | Descrição |
|---|---|
| **Workflow** | Processo automatizado definido em YAML. Um repositório pode ter múltiplos workflows |
| **Event** | Evento que dispara o workflow (push, pull_request, schedule, workflow_dispatch, etc.) |
| **Job** | Conjunto de steps que executam no mesmo runner. Jobs rodam em paralelo por padrão |
| **Step** | Tarefa individual — executa um Action ou um comando shell |
| **Action** | Unidade reutilizável de código (marketplace ou custom) |
| **Runner** | Máquina (VM) que executa os jobs. GitHub-hosted ou self-hosted |

### 3.2 Runners

| Tipo | Descrição | Quando usar |
|---|---|---|
| **GitHub-hosted** | VMs efêmeras gerenciadas pelo GitHub (Ubuntu, Windows, macOS) | Maioria dos projetos — zero manutenção |
| **Self-hosted** | Máquinas próprias registradas no GitHub | Necessidade de hardware especial, rede privada, GPU |
| **Larger runners** | GitHub-hosted com mais recursos (planos pagos) | Builds pesados, testes de carga |

### 3.3 Limites (plano Free / público)

| Recurso | Limite |
|---|---|
| Minutos/mês (repos privados) | 2.000 |
| Jobs concorrentes | 20 |
| Tempo máximo por job | 6 horas |
| Tempo máximo por workflow | 35 dias |
| Armazenamento de artefatos | 500 MB |

---

## 4. GitHub Actions — Sintaxe do Workflow

### 4.1 Estrutura Básica

```yaml
# .github/workflows/ci.yml
name: CI Pipeline                    # Nome exibido na UI

on:                                  # Eventos que disparam o workflow
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

permissions:                         # Permissões do GITHUB_TOKEN
  contents: read
  packages: write

env:                                 # Variáveis globais
  JAVA_VERSION: '21'
  REGISTRY: ghcr.io

jobs:                                # Definição dos jobs
  build:
    name: Build and Test
    runs-on: ubuntu-latest           # Runner
    steps:
      - uses: actions/checkout@v4    # Action do marketplace
      - name: Run tests
        run: echo "Hello CI"         # Comando shell
```

### 4.2 Triggers (Eventos)

```yaml
on:
  # Push em branches específicas
  push:
    branches: [main, 'release/**']
    tags: ['v*']
    paths:                             # Só dispara se esses arquivos mudarem
      - 'src/**'
      - 'pom.xml'
    paths-ignore:                      # Ignora mudanças nesses arquivos
      - '**.md'
      - 'docs/**'

  # Pull Request
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]

  # Agendamento (cron)
  schedule:
    - cron: '0 6 * * 1-5'             # Seg-Sex às 6h UTC

  # Dispatch manual (botão na UI)
  workflow_dispatch:
    inputs:
      environment:
        description: 'Ambiente de deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      dry_run:
        description: 'Execução simulada'
        type: boolean
        default: false

  # Ao completar outro workflow
  workflow_run:
    workflows: ['CI Pipeline']
    types: [completed]
    branches: [main]

  # Ao criar/atualizar release
  release:
    types: [published]
```

### 4.3 Jobs e Dependências

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./gradlew checkstyleMain

  test:
    runs-on: ubuntu-latest
    needs: lint                        # Executa após lint
    steps:
      - uses: actions/checkout@v4
      - run: ./gradlew test

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]                # Executa após ambos
    steps:
      - uses: actions/checkout@v4
      - run: ./gradlew build

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'  # Condicional
    steps:
      - run: echo "Deploying..."
```

```
                    ┌──────┐
                    │ lint │
                    └──┬───┘
                       │
                    ┌──▼───┐
                    │ test │
                    └──┬───┘
                       │
                    ┌──▼────┐
                    │ build │
                    └──┬────┘
                       │
               ┌───────▼───────┐
               │ deploy        │
               │ (only main)   │
               └───────────────┘
```

### 4.4 Steps — Sintaxe Detalhada

```yaml
steps:
  # Usar uma action
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0                   # Histórico completo

  # Comando simples
  - name: Show Java version
    run: java --version

  # Multiline
  - name: Build project
    run: |
      echo "Building..."
      ./gradlew clean build
      echo "Build complete"

  # Com variáveis de ambiente
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      ENV: production
    run: ./deploy.sh

  # Condicional
  - name: Publish snapshot
    if: github.event_name == 'push' && github.ref == 'refs/heads/develop'
    run: ./gradlew publish

  # Com timeout
  - name: Integration tests
    timeout-minutes: 15
    run: ./gradlew integrationTest

  # Continue on error
  - name: Optional lint
    continue-on-error: true
    run: ./gradlew spotlessCheck

  # Definir output
  - name: Get version
    id: version
    run: echo "version=$(./gradlew -q printVersion)" >> $GITHUB_OUTPUT

  # Usar output de step anterior
  - name: Use version
    run: echo "Version is ${{ steps.version.outputs.version }}"
```

### 4.5 Expressões e Contextos

```yaml
# Contextos disponíveis
${{ github.repository }}          # owner/repo
${{ github.sha }}                 # SHA do commit
${{ github.ref_name }}            # Nome do branch/tag
${{ github.actor }}               # Quem disparou
${{ github.event_name }}          # Tipo de evento
${{ github.run_id }}              # ID da execução
${{ github.workspace }}           # Diretório de trabalho

${{ secrets.MY_SECRET }}          # Secret
${{ vars.MY_VAR }}                # Variable (não-secreta)
${{ env.MY_ENV }}                 # Variável de ambiente
${{ steps.step_id.outputs.key }}  # Output de um step
${{ needs.job_id.outputs.key }}   # Output de um job
${{ runner.os }}                  # linux, windows, macos

# Funções
${{ contains(github.event.head_commit.message, '[skip ci]') }}
${{ startsWith(github.ref, 'refs/tags/v') }}
${{ format('Hello {0}!', github.actor) }}
${{ toJSON(github.event) }}
${{ hashFiles('**/pom.xml') }}
${{ fromJSON(needs.setup.outputs.matrix) }}

# Condicionais
if: success()                     # Steps anteriores passaram
if: failure()                     # Algum step falhou
if: always()                      # Sempre executa
if: cancelled()                   # Workflow cancelado
```

---

## 5. GitHub Actions — Exemplos Práticos

### 5.1 CI para Projeto Maven (Spring Boot)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready -U test"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5

    steps:
      - uses: actions/checkout@v4

      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'
          cache: 'maven'

      - name: Build and Test
        run: ./mvnw -B verify
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: test
          SPRING_DATASOURCE_PASSWORD: test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: target/surefire-reports/
          retention-days: 7
```

### 5.2 CI para Projeto Gradle (Spring Boot)

```yaml
name: CI Gradle

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'
          cache: 'gradle'

      - name: Grant execute permission
        run: chmod +x gradlew

      - name: Build
        run: ./gradlew build

      - name: Test Report
        if: always()
        uses: dorny/test-reporter@v1
        with:
          name: JUnit Tests
          path: build/test-results/test/*.xml
          reporter: java-junit
```

### 5.3 CI para Projeto Node.js (React/Angular/Vue)

```yaml
name: Frontend CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm run test -- --coverage
      - run: npm run build

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

### 5.4 Build e Push de Imagem Docker

```yaml
name: Docker Build

on:
  push:
    tags: ['v*']

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 6. GitHub Actions — Cache e Artefatos

### 6.1 Cache de Dependências

```yaml
# Cache automático via setup-java / setup-node (recomendado)
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '21'
    cache: 'maven'           # ou 'gradle'

# Cache manual (controle total)
- name: Cache Maven packages
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: maven-${{ runner.os }}-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      maven-${{ runner.os }}-

# Cache do Gradle (manual)
- name: Cache Gradle
  uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: gradle-${{ runner.os }}-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
    restore-keys: |
      gradle-${{ runner.os }}-
```

### 6.2 Artefatos

```yaml
# Upload de artefatos
- uses: actions/upload-artifact@v4
  with:
    name: app-jar
    path: target/*.jar
    retention-days: 5
    if-no-files-found: error        # warn, ignore, error

# Download em outro job
deploy:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - uses: actions/download-artifact@v4
      with:
        name: app-jar
        path: ./artifacts

    - run: ls -la ./artifacts/
```

### 6.3 Cache vs Artefatos

| Aspecto | Cache | Artefatos |
|---|---|---|
| **Propósito** | Acelerar builds reutilizando dependências | Compartilhar resultados entre jobs/workflows |
| **Chave** | Hash de arquivos (pom.xml, package-lock.json) | Nome definido pelo usuário |
| **Retenção** | 7 dias (sem acesso) | Configurável (padrão 90 dias) |
| **Entre workflows** | Sim (mesma chave/branch) | Sim (via workflow_run ou download) |
| **Limite** | 10 GB por repositório | 500 MB (Free) a ilimitado (Enterprise) |

---

## 7. GitHub Actions — Matrices e Estratégias

### 7.1 Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java: ['17', '21']
        db: ['postgres', 'mysql']
      fail-fast: false                 # Não cancela os outros se um falhar
      max-parallel: 2                  # Limitar paralelismo

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ matrix.java }}

      - run: ./mvnw test -Ddb.type=${{ matrix.db }}
```

### 7.2 Matrix com Include/Exclude

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    java: ['17', '21']
    exclude:
      - os: windows-latest
        java: '17'
    include:
      - os: macos-latest
        java: '21'
        extra-flag: '--enable-preview'
```

### 7.3 Matrix Dinâmica

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo 'matrix={"java":["17","21"],"os":["ubuntu-latest","windows-latest"]}' >> $GITHUB_OUTPUT
          else
            echo 'matrix={"java":["21"],"os":["ubuntu-latest"]}' >> $GITHUB_OUTPUT
          fi

  test:
    needs: setup
    runs-on: ${{ matrix.os }}
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ matrix.java }}
      - run: ./mvnw test
```

---

## 8. GitHub Actions — Secrets e Variáveis de Ambiente

### 8.1 Níveis de Secrets

```
┌─────────────────────────────────────────────────┐
│              Hierarquia de Secrets                │
│                                                   │
│  ┌─────────────────────────────────────────────┐ │
│  │  Organization Secrets                        │ │
│  │  (compartilhados entre repos selecionados)   │ │
│  │  ┌───────────────────────────────────────┐   │ │
│  │  │  Repository Secrets                    │   │ │
│  │  │  (escopo do repositório)               │   │ │
│  │  │  ┌─────────────────────────────────┐   │   │ │
│  │  │  │  Environment Secrets             │   │   │ │
│  │  │  │  (escopo do environment)          │   │   │ │
│  │  │  └─────────────────────────────────┘   │   │ │
│  │  └───────────────────────────────────────┘   │ │
│  └─────────────────────────────────────────────┘ │
│                                                   │
│  Prioridade: Environment > Repository > Org      │
└─────────────────────────────────────────────────┘
```

### 8.2 Uso de Secrets

```yaml
steps:
  - name: Deploy
    env:
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
      API_KEY: ${{ secrets.API_KEY }}
    run: ./deploy.sh

  # GITHUB_TOKEN — token automático
  - name: Create Release
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    run: gh release create v1.0.0
```

### 8.3 Variables (não-secretas)

```yaml
# Configuradas em Settings > Secrets and Variables > Actions > Variables
steps:
  - run: echo "Deploying to ${{ vars.DEPLOY_URL }}"
  - run: echo "Region is ${{ vars.AWS_REGION }}"
```

### 8.4 Boas Práticas de Secrets

- Nunca logar secrets — GitHub mascara automaticamente, mas evite `echo ${{ secrets.X }}`
- Usar OIDC para cloud em vez de credenciais estáticas (AWS, GCP, Azure)
- Rotacionar secrets periodicamente
- Limitar escopo — usar Environment Secrets quando possível
- Nunca colocar secrets em outputs de steps ou artefatos

---

## 9. GitHub Actions — Environments e Deploy

### 9.1 Configuração de Environments

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - run: echo "Deploy to staging"

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://example.com
    steps:
      - run: echo "Deploy to production"
```

### 9.2 Proteções de Environment

Configuradas na UI do GitHub em **Settings > Environments**:

| Proteção | Descrição |
|---|---|
| **Required reviewers** | Aprovação manual antes do deploy |
| **Wait timer** | Atraso antes de executar (ex: 30 min) |
| **Deployment branches** | Apenas branches específicas podem fazer deploy |
| **Custom rules** | Regras personalizadas via GitHub Apps |

### 9.3 Concurrency — Evitar Deploys Simultâneos

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: deploy-${{ github.ref }}
      cancel-in-progress: false        # true para PRs, false para deploys
    environment: production
    steps:
      - run: ./deploy.sh
```

---

## 10. GitHub Actions — Actions Reutilizáveis e Composite Actions

### 10.1 Composite Action

```yaml
# .github/actions/setup-java-build/action.yml
name: 'Setup Java and Build'
description: 'Configura JDK e executa build Maven'
inputs:
  java-version:
    description: 'Versão do Java'
    required: false
    default: '21'
outputs:
  artifact-version:
    description: 'Versão do artefato gerado'
    value: ${{ steps.version.outputs.version }}

runs:
  using: 'composite'
  steps:
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: ${{ inputs.java-version }}
        cache: 'maven'

    - name: Build
      shell: bash
      run: ./mvnw -B package -DskipTests

    - name: Get version
      id: version
      shell: bash
      run: |
        VERSION=$(./mvnw help:evaluate -Dexpression=project.version -q -DforceStdout)
        echo "version=$VERSION" >> $GITHUB_OUTPUT
```

### 10.2 Usando a Composite Action

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        id: build
        uses: ./.github/actions/setup-java-build
        with:
          java-version: '21'

      - run: echo "Built version ${{ steps.build.outputs.artifact-version }}"
```

---

## 11. GitHub Actions — Workflows Reutilizáveis

### 11.1 Workflow Chamável (Reusable)

```yaml
# .github/workflows/reusable-deploy.yml
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      DEPLOY_KEY:
        required: true
    outputs:
      deploy-url:
        description: 'URL do deploy'
        value: ${{ jobs.deploy.outputs.url }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    outputs:
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - name: Deploy
        id: deploy
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          echo "Deploying ${{ inputs.image-tag }} to ${{ inputs.environment }}"
          echo "url=https://${{ inputs.environment }}.example.com" >> $GITHUB_OUTPUT
```

### 11.2 Chamando o Workflow Reutilizável

```yaml
# .github/workflows/main.yml
name: Main Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.tag.outputs.tag }}
    steps:
      - uses: actions/checkout@v4
      - id: tag
        run: echo "tag=sha-$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: build
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
      image-tag: ${{ needs.build.outputs.image-tag }}
    secrets:
      DEPLOY_KEY: ${{ secrets.STAGING_DEPLOY_KEY }}

  deploy-production:
    needs: [build, deploy-staging]
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
      image-tag: ${{ needs.build.outputs.image-tag }}
    secrets:
      DEPLOY_KEY: ${{ secrets.PROD_DEPLOY_KEY }}
```

---

## 12. GitLab CI/CD — Arquitetura e Conceitos

### 12.1 Componentes

```
┌──────────────────────────────────────────────────────────────────────┐
│                       GitLab CI/CD                                    │
│                                                                      │
│  ┌───────────┐    ┌───────────────────────────────────────────────┐  │
│  │  Trigger   │───▶│  Pipeline (.gitlab-ci.yml)                   │  │
│  │  (push,MR) │    │                                               │  │
│  └───────────┘    │  Stages (sequenciais):                        │  │
│                    │  ┌────────┐  ┌────────┐  ┌─────────┐         │  │
│                    │  │ build  │─▶│  test  │─▶│ deploy  │         │  │
│                    │  │        │  │        │  │         │         │  │
│                    │  │ Job A  │  │ Job B  │  │ Job D   │         │  │
│                    │  │        │  │ Job C  │  │ Job E   │         │  │
│                    │  └───┬────┘  └───┬────┘  └────┬────┘         │  │
│                    │      │           │            │               │  │
│                    │      ▼           ▼            ▼               │  │
│                    │  ┌────────────────────────────────┐           │  │
│                    │  │  GitLab Runners                 │           │  │
│                    │  │  (Shell / Docker / Kubernetes)  │           │  │
│                    │  └────────────────────────────────┘           │  │
│                    └───────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

| Componente | Descrição |
|---|---|
| **Pipeline** | Conjunto completo de jobs para um commit/MR. Definido em `.gitlab-ci.yml` |
| **Stage** | Fase lógica da pipeline (build, test, deploy). Jobs do mesmo stage rodam em paralelo |
| **Job** | Unidade de execução. Define o que executar, em qual stage e com quais condições |
| **Runner** | Agente que executa os jobs. Pode ser shared (GitLab.com) ou self-managed |
| **Executor** | Método de execução do runner (Shell, Docker, Kubernetes, VirtualBox) |

### 12.2 Tipos de Runner

| Executor | Descrição | Quando usar |
|---|---|---|
| **Docker** | Cada job roda em um container isolado | Padrão recomendado — isolamento e reprodutibilidade |
| **Shell** | Executa diretamente no host do runner | Scripts simples, acesso a hardware |
| **Kubernetes** | Cria pods efêmeros para cada job | Ambientes Kubernetes, escalabilidade automática |
| **Docker Machine** | Auto-scaling com VMs na cloud | Grandes equipes com demanda variável |

### 12.3 Pipelines Especiais

| Tipo | Descrição |
|---|---|
| **Branch pipeline** | Disparada por push em qualquer branch |
| **Merge Request pipeline** | Disparada por criação/atualização de MR |
| **Merged results pipeline** | Simula o merge antes de executar |
| **Scheduled pipeline** | Execução agendada (cron) |
| **Parent-Child** | Pipeline pai dispara pipelines filhas |
| **Multi-project** | Pipeline dispara pipelines em outros projetos |

---

## 13. GitLab CI/CD — Sintaxe do .gitlab-ci.yml

### 13.1 Estrutura Básica

```yaml
# .gitlab-ci.yml
image: eclipse-temurin:21-jdk            # Imagem Docker padrão

stages:                                    # Ordem dos stages
  - build
  - test
  - package
  - deploy

variables:                                 # Variáveis globais
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"

# Job de build
build:
  stage: build
  script:
    - ./mvnw -B compile
  artifacts:
    paths:
      - target/
    expire_in: 1 hour

# Job de teste
test:
  stage: test
  script:
    - ./mvnw -B verify
  artifacts:
    reports:
      junit: target/surefire-reports/*.xml   # Integração com GitLab Test Report
```

### 13.2 Keywords Principais

```yaml
job_name:
  stage: test                              # Stage do job
  image: node:22-alpine                    # Imagem Docker específica
  services:                                # Containers auxiliares (como banco)
    - name: postgres:17
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test

  variables:                               # Variáveis do job
    DATABASE_URL: "postgresql://test:test@db:5432/testdb"

  before_script:                           # Executado antes do script principal
    - echo "Preparing..."

  script:                                  # Script principal (obrigatório)
    - npm ci
    - npm test

  after_script:                            # Executado após (mesmo em falha)
    - echo "Cleanup..."

  artifacts:                               # Artefatos gerados
    paths:
      - dist/
    expire_in: 1 week

  cache:                                   # Cache entre execuções
    key: $CI_COMMIT_REF_SLUG
    paths:
      - node_modules/

  tags:                                    # Selecionar runner por tag
    - docker
    - linux

  only:                                    # (legado) Quando executar
    - main
  except:                                  # (legado) Quando não executar
    - tags

  rules:                                   # (moderno) Regras condicionais
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: always
    - when: never
```

### 13.3 Rules — Controle de Execução

```yaml
# rules substitui only/except (recomendado)
deploy_staging:
  stage: deploy
  script:
    - ./deploy.sh staging
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - if: $CI_COMMIT_BRANCH == "develop"
      when: manual
      allow_failure: true                  # Manual não bloqueia pipeline
    - when: never

test:
  stage: test
  script:
    - ./mvnw test
  rules:
    - changes:                             # Executa se arquivos mudaram
        - src/**/*
        - pom.xml
      when: always
    - exists:                              # Executa se arquivo existe
        - Dockerfile
      when: always
    - if: $CI_PIPELINE_SOURCE == "schedule"
      when: always
```

### 13.4 Variáveis Predefinidas do GitLab

```yaml
# Variáveis mais usadas
$CI_PROJECT_NAME          # Nome do projeto
$CI_PROJECT_DIR           # Diretório raiz do projeto
$CI_COMMIT_SHA            # SHA completo do commit
$CI_COMMIT_SHORT_SHA      # SHA curto (8 chars)
$CI_COMMIT_REF_NAME       # Nome do branch ou tag
$CI_COMMIT_REF_SLUG       # Nome do branch URL-safe
$CI_COMMIT_BRANCH         # Nome do branch (só em branch pipelines)
$CI_COMMIT_TAG            # Nome da tag (só em tag pipelines)
$CI_PIPELINE_ID           # ID da pipeline
$CI_PIPELINE_SOURCE       # push, merge_request_event, schedule, api, web
$CI_JOB_ID                # ID do job
$CI_JOB_NAME              # Nome do job
$CI_MERGE_REQUEST_IID     # IID do MR (só em MR pipelines)
$CI_REGISTRY              # URL do container registry
$CI_REGISTRY_IMAGE        # Caminho da imagem no registry
$CI_DEFAULT_BRANCH        # Branch padrão (geralmente main)
$GITLAB_USER_LOGIN        # Usuário que disparou
```

### 13.5 Needs — DAG (Directed Acyclic Graph)

```yaml
# Sem needs: jobs seguem a ordem dos stages
# Com needs: jobs podem pular stages e definir dependências explícitas

stages:
  - build
  - test
  - deploy

build_backend:
  stage: build
  script: ./mvnw package

build_frontend:
  stage: build
  script: npm run build

test_backend:
  stage: test
  needs: [build_backend]              # Executa assim que build_backend terminar
  script: ./mvnw verify

test_frontend:
  stage: test
  needs: [build_frontend]             # Não espera build_backend
  script: npm test

deploy:
  stage: deploy
  needs: [test_backend, test_frontend]
  script: ./deploy.sh
```

```
  ┌────────────────┐    ┌─────────────────┐
  │ build_backend  │    │ build_frontend  │     (paralelo)
  └───────┬────────┘    └───────┬─────────┘
          │                     │
          ▼                     ▼
  ┌────────────────┐    ┌─────────────────┐
  │ test_backend   │    │ test_frontend   │     (paralelo, via needs)
  └───────┬────────┘    └───────┬─────────┘
          │                     │
          └──────────┬──────────┘
                     ▼
             ┌──────────────┐
             │    deploy    │
             └──────────────┘
```

---

## 14. GitLab CI/CD — Exemplos Práticos

### 14.1 Pipeline Maven (Spring Boot)

```yaml
image: eclipse-temurin:21-jdk

stages:
  - build
  - test
  - package
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"

cache:
  key: maven-$CI_COMMIT_REF_SLUG
  paths:
    - .m2/repository/

build:
  stage: build
  script:
    - ./mvnw -B compile
  artifacts:
    paths:
      - target/classes/
    expire_in: 1 hour

test:
  stage: test
  services:
    - name: postgres:17
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
  variables:
    SPRING_DATASOURCE_URL: "jdbc:postgresql://db:5432/testdb"
    SPRING_DATASOURCE_USERNAME: test
    SPRING_DATASOURCE_PASSWORD: test
  script:
    - ./mvnw -B verify
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
    paths:
      - target/site/jacoco/
    expire_in: 1 week
  coverage: '/Total.*?(\d+%)/'

package:
  stage: package
  script:
    - ./mvnw -B package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 14.2 Pipeline Gradle (Spring Boot)

```yaml
image: eclipse-temurin:21-jdk

stages:
  - build
  - test
  - package

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false -Dorg.gradle.caching=true"

cache:
  key: gradle-$CI_COMMIT_REF_SLUG
  paths:
    - .gradle/wrapper
    - .gradle/caches
  policy: pull-push

build:
  stage: build
  script:
    - chmod +x gradlew
    - ./gradlew assemble
  artifacts:
    paths:
      - build/classes/
      - build/libs/
    expire_in: 1 hour

test:
  stage: test
  script:
    - chmod +x gradlew
    - ./gradlew check
  artifacts:
    reports:
      junit: build/test-results/test/*.xml
  coverage: '/Total.*?(\d+%)/'

package:
  stage: package
  script:
    - chmod +x gradlew
    - ./gradlew bootJar
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 14.3 Build e Push de Imagem Docker

```yaml
docker_build:
  stage: package
  image: docker:27
  services:
    - docker:27-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --pull -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
    - |
      if [ "$CI_COMMIT_BRANCH" = "$CI_DEFAULT_BRANCH" ]; then
        docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
        docker push $CI_REGISTRY_IMAGE:latest
      fi
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_COMMIT_TAG
```

### 14.4 Pipeline para Node.js

```yaml
image: node:22-alpine

stages:
  - install
  - lint
  - test
  - build

cache:
  key: npm-$CI_COMMIT_REF_SLUG
  paths:
    - node_modules/

install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

lint:
  stage: lint
  needs: [install]
  script:
    - npm run lint

test:
  stage: test
  needs: [install]
  script:
    - npm run test -- --coverage
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  coverage: '/All files.*?\s+(\d+\.?\d*)/'

build:
  stage: build
  needs: [lint, test]
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
```

---

## 15. GitLab CI/CD — Cache e Artefatos

### 15.1 Cache — Estratégias

```yaml
# Cache global
cache:
  key: $CI_COMMIT_REF_SLUG              # Chave por branch
  paths:
    - .m2/repository/
  policy: pull-push                      # pull, push, pull-push

# Cache com fallback
cache:
  key:
    files:
      - pom.xml                          # Chave baseada no hash do arquivo
    prefix: maven
  paths:
    - .m2/repository/
  fallback_keys:                         # Fallback se a chave principal não existir
    - maven-$CI_DEFAULT_BRANCH

# Cache por job (sobrescreve global)
test:
  cache:
    key: test-$CI_COMMIT_REF_SLUG
    paths:
      - node_modules/
    policy: pull                         # Só lê, não grava
  script:
    - npm test
```

### 15.2 Artefatos — Tipos

```yaml
artifacts:
  paths:                                 # Arquivos genéricos
    - target/*.jar
    - build/reports/

  reports:                               # Relatórios integrados à UI
    junit: target/surefire-reports/*.xml
    coverage_report:
      coverage_format: cobertura
      path: target/site/jacoco/jacoco.xml
    sast: gl-sast-report.json
    dependency_scanning: gl-dependency-scanning-report.json
    container_scanning: gl-container-scanning-report.json

  expire_in: 1 week                     # Retenção
  when: always                          # always, on_success, on_failure
  expose_as: 'Build JAR'                # Link visível no MR
```

---

## 16. GitLab CI/CD — Variáveis e Secrets

### 16.1 Níveis de Variáveis

```
┌──────────────────────────────────────────────┐
│           Hierarquia de Variáveis             │
│                                               │
│  ┌─────────────────────────────────────────┐ │
│  │  Instance Variables (admin)              │ │
│  │  ┌───────────────────────────────────┐   │ │
│  │  │  Group Variables                   │   │ │
│  │  │  ┌─────────────────────────────┐   │   │ │
│  │  │  │  Project Variables           │   │   │ │
│  │  │  │  ┌───────────────────────┐   │   │   │ │
│  │  │  │  │  .gitlab-ci.yml vars  │   │   │   │ │
│  │  │  │  │  ┌─────────────────┐  │   │   │   │ │
│  │  │  │  │  │  Job variables  │  │   │   │   │ │
│  │  │  │  │  └─────────────────┘  │   │   │   │ │
│  │  │  │  └───────────────────────┘   │   │   │ │
│  │  │  └─────────────────────────────┘   │   │ │
│  │  └───────────────────────────────────┘   │ │
│  └─────────────────────────────────────────┘ │
│                                               │
│  Prioridade: Job > YAML > Project > Group    │
└──────────────────────────────────────────────┘
```

### 16.2 Opções de Variáveis

| Opção | Descrição |
|---|---|
| **Protected** | Disponível apenas em branches/tags protegidos |
| **Masked** | Valor oculto nos logs (requer formato específico) |
| **Expanded** | Permite referência a outras variáveis (`$OTHER_VAR`) |
| **Environment scope** | Limita a variável a um environment específico |

### 16.3 Uso em Pipeline

```yaml
variables:
  GLOBAL_VAR: "valor-global"

job:
  variables:
    JOB_VAR: "valor-job"
  script:
    - echo $GLOBAL_VAR
    - echo $JOB_VAR
    - echo $CI_PROJECT_VARIABLES_SECRET     # Definida na UI/API
```

---

## 17. GitLab CI/CD — Environments e Deploy

### 17.1 Definição de Environment

```yaml
deploy_staging:
  stage: deploy
  script:
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging                  # Job para parar o environment
    auto_stop_in: 1 week                   # Auto-stop após inatividade
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

stop_staging:
  stage: deploy
  script:
    - ./teardown.sh staging
  environment:
    name: staging
    action: stop
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"
      when: manual
```

### 17.2 Environments Dinâmicos (Review Apps)

```yaml
deploy_review:
  stage: deploy
  script:
    - ./deploy.sh review-$CI_COMMIT_REF_SLUG
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop_review
    auto_stop_in: 2 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

stop_review:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - ./teardown.sh review-$CI_COMMIT_REF_SLUG
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
```

### 17.3 Deploy com Aprovação Manual

```yaml
deploy_production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  when: manual                             # Botão manual na UI
  allow_failure: false                     # Bloqueia pipeline até aprovação
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

---

## 18. GitLab CI/CD — Templates e Includes

### 18.1 extends — Herança de Jobs

```yaml
.base_test:                                # Job oculto (prefixo com ponto)
  stage: test
  before_script:
    - echo "Preparing test environment"
  after_script:
    - echo "Cleanup"

test_unit:
  extends: .base_test
  script:
    - ./mvnw test

test_integration:
  extends: .base_test
  services:
    - postgres:17
  script:
    - ./mvnw verify -P integration
```

### 18.2 YAML Anchors

```yaml
.maven_cache: &maven_cache
  cache:
    key: maven-$CI_COMMIT_REF_SLUG
    paths:
      - .m2/repository/

build:
  <<: *maven_cache
  stage: build
  script:
    - ./mvnw compile

test:
  <<: *maven_cache
  stage: test
  script:
    - ./mvnw verify
```

### 18.3 include — Importar Configurações

```yaml
include:
  # Arquivo local
  - local: '/.gitlab/ci/build.yml'

  # Template do GitLab (built-in)
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml

  # Arquivo de outro projeto
  - project: 'devops/ci-templates'
    ref: main
    file: '/templates/spring-boot.yml'

  # URL remota
  - remote: 'https://example.com/ci/template.yml'

  # Múltiplos arquivos do mesmo projeto
  - project: 'devops/ci-templates'
    ref: v2.0
    file:
      - '/templates/docker.yml'
      - '/templates/deploy.yml'
```

### 18.4 Template Reutilizável Completo

```yaml
# devops/ci-templates/templates/spring-boot.yml

.spring_boot_build:
  image: eclipse-temurin:21-jdk
  variables:
    MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  cache:
    key:
      files:
        - pom.xml
      prefix: maven
    paths:
      - .m2/repository/

.spring_boot_test:
  extends: .spring_boot_build
  stage: test
  script:
    - ./mvnw -B verify
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml

.spring_boot_package:
  extends: .spring_boot_build
  stage: package
  script:
    - ./mvnw -B package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week

# Uso no projeto:
# include:
#   - project: 'devops/ci-templates'
#     file: '/templates/spring-boot.yml'
#
# test:
#   extends: .spring_boot_test
#
# package:
#   extends: .spring_boot_package
```

---

## 19. GitLab CI/CD — Pipelines Pai-Filho e Multi-Projeto

### 19.1 Pipeline Pai-Filho (Parent-Child)

```yaml
# .gitlab-ci.yml (pai)
stages:
  - triggers

trigger_backend:
  stage: triggers
  trigger:
    include: backend/.gitlab-ci.yml
    strategy: depend                       # Pai espera o filho completar
  rules:
    - changes:
        - backend/**/*

trigger_frontend:
  stage: triggers
  trigger:
    include: frontend/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - frontend/**/*
```

```yaml
# backend/.gitlab-ci.yml (filho)
stages:
  - build
  - test

build:
  stage: build
  image: eclipse-temurin:21-jdk
  script:
    - cd backend && ./mvnw compile

test:
  stage: test
  image: eclipse-temurin:21-jdk
  script:
    - cd backend && ./mvnw verify
```

### 19.2 Pipeline Multi-Projeto

```yaml
deploy_infrastructure:
  stage: deploy
  trigger:
    project: devops/infrastructure          # Outro projeto
    branch: main
    strategy: depend
  variables:
    APP_VERSION: $CI_COMMIT_SHORT_SHA
```

---

## 20. Comparação GitHub Actions vs GitLab CI/CD

### 20.1 Tabela Comparativa

| Aspecto | GitHub Actions | GitLab CI/CD |
|---|---|---|
| **Configuração** | `.github/workflows/*.yml` (múltiplos) | `.gitlab-ci.yml` (único + includes) |
| **Estrutura** | Workflow > Jobs > Steps | Pipeline > Stages > Jobs |
| **Paralelismo** | Jobs paralelos por padrão | Jobs do mesmo stage paralelos |
| **Dependência** | `needs` entre jobs | `needs` (DAG) ou ordem de stages |
| **Marketplace** | 20.000+ actions no Marketplace | Templates built-in + includes |
| **Runners** | GitHub-hosted (gratuitos p/ público) | Shared runners no GitLab.com |
| **Self-hosted** | Self-hosted runners | Self-managed runners |
| **Container Registry** | GitHub Packages (GHCR) | Built-in Container Registry |
| **Environments** | Sim, com proteções | Sim, com Review Apps |
| **Secrets** | Secrets (org/repo/environment) | CI/CD Variables (masked/protected) |
| **SAST/DAST** | Via actions de terceiros | Built-in (Ultimate) |
| **Monorepo** | `paths` filter ou action | `changes` + parent-child pipelines |
| **Auto DevOps** | Não possui | Sim (pipeline automática) |
| **Cache** | `actions/cache` | Built-in (`cache:` keyword) |
| **Preço (privado)** | 2.000 min/mês (Free) | 400 min/mês (Free) |

### 20.2 Quando Usar Qual

| Cenário | Recomendação |
|---|---|
| Projeto open-source no GitHub | **GitHub Actions** — minutos ilimitados para repos públicos |
| Equipe já usa GitLab | **GitLab CI/CD** — integração nativa com todo o GitLab |
| Necessidade de SAST/DAST integrado | **GitLab** (Ultimate) ou GitHub com actions de terceiros |
| Review Apps automáticas | **GitLab** — suporte nativo a environments dinâmicos |
| Marketplace de integrações | **GitHub Actions** — maior ecossistema de actions |
| Monorepo complexo | **GitLab** — parent-child pipelines são superiores |
| DevSecOps completo | **GitLab** — pipeline de segurança integrada |
| Projetos simples / starter | Ambos funcionam bem — use onde o código está |

---

## 21. Pipeline Completo — Spring Boot (GitHub Actions)

```yaml
# .github/workflows/ci-cd.yml
name: Spring Boot CI/CD

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write
  security-events: write

env:
  JAVA_VERSION: '21'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─── Lint e Análise Estática ───
  lint:
    name: Checkstyle & SpotBugs
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}
          cache: 'maven'

      - name: Checkstyle
        run: ./mvnw -B checkstyle:check

      - name: SpotBugs
        run: ./mvnw -B spotbugs:check

  # ─── Testes Unitários ───
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}
          cache: 'maven'

      - name: Run unit tests
        run: ./mvnw -B test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: unit-test-results
          path: target/surefire-reports/

  # ─── Testes de Integração ───
  test-integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready -U test"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}
          cache: 'maven'

      - name: Run integration tests
        run: ./mvnw -B verify -P integration-test
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: test
          SPRING_DATASOURCE_PASSWORD: test

  # ─── Build e Push Docker ───
  docker:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test-unit, test-integration]
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}
          cache: 'maven'

      - name: Build JAR
        run: ./mvnw -B package -DskipTests

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─── Deploy Staging ───
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com
    concurrency:
      group: deploy-staging
      cancel-in-progress: false
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging
        env:
          IMAGE_TAG: ${{ needs.docker.outputs.image-tag }}
          KUBE_CONFIG: ${{ secrets.STAGING_KUBECONFIG }}
        run: |
          echo "$KUBE_CONFIG" > kubeconfig.yml
          export KUBECONFIG=kubeconfig.yml
          kubectl set image deployment/app app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${IMAGE_TAG}
          kubectl rollout status deployment/app --timeout=300s

      - name: Smoke test
        run: |
          sleep 10
          curl -f https://staging.example.com/actuator/health || exit 1

  # ─── Deploy Production ───
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [docker, deploy-staging]
    if: startsWith(github.ref, 'refs/tags/v')
    environment:
      name: production
      url: https://example.com
    concurrency:
      group: deploy-production
      cancel-in-progress: false
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        env:
          IMAGE_TAG: ${{ needs.docker.outputs.image-tag }}
          KUBE_CONFIG: ${{ secrets.PROD_KUBECONFIG }}
        run: |
          echo "$KUBE_CONFIG" > kubeconfig.yml
          export KUBECONFIG=kubeconfig.yml
          kubectl set image deployment/app app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${IMAGE_TAG}
          kubectl rollout status deployment/app --timeout=300s
```

---

## 22. Pipeline Completo — Spring Boot (GitLab CI/CD)

```yaml
# .gitlab-ci.yml
image: eclipse-temurin:21-jdk

stages:
  - lint
  - test
  - package
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

cache: &maven_cache
  key:
    files:
      - pom.xml
    prefix: maven
  paths:
    - .m2/repository/
  policy: pull-push

# ─── Lint e Análise Estática ───
checkstyle:
  stage: lint
  script:
    - ./mvnw -B checkstyle:check
  cache:
    <<: *maven_cache
    policy: pull

spotbugs:
  stage: lint
  script:
    - ./mvnw -B spotbugs:check
  cache:
    <<: *maven_cache
    policy: pull

# ─── Testes Unitários ───
test_unit:
  stage: test
  needs: [checkstyle, spotbugs]
  script:
    - ./mvnw -B test
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
    paths:
      - target/site/jacoco/
    expire_in: 1 week
  coverage: '/Total.*?(\d+%)/'

# ─── Testes de Integração ───
test_integration:
  stage: test
  needs: [checkstyle, spotbugs]
  services:
    - name: postgres:17
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
  variables:
    SPRING_DATASOURCE_URL: "jdbc:postgresql://db:5432/testdb"
    SPRING_DATASOURCE_USERNAME: test
    SPRING_DATASOURCE_PASSWORD: test
  script:
    - ./mvnw -B verify -P integration-test
  artifacts:
    reports:
      junit: target/failsafe-reports/TEST-*.xml
    expire_in: 1 week

# ─── Build Docker ───
docker_build:
  stage: package
  image: docker:27
  services:
    - docker:27-dind
  needs: [test_unit, test_integration]
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --pull -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
    - |
      if [ "$CI_COMMIT_BRANCH" = "$CI_DEFAULT_BRANCH" ]; then
        docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
        docker push $CI_REGISTRY_IMAGE:latest
      fi
    - |
      if [ -n "$CI_COMMIT_TAG" ]; then
        docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
        docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
      fi
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_COMMIT_TAG

# ─── Deploy Staging ───
deploy_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  needs: [docker_build]
  script:
    - kubectl config use-context $KUBE_CONTEXT_STAGING
    - kubectl set image deployment/app app=$IMAGE_TAG
    - kubectl rollout status deployment/app --timeout=300s
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

stop_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - kubectl config use-context $KUBE_CONTEXT_STAGING
    - kubectl scale deployment/app --replicas=0
  environment:
    name: staging
    action: stop
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual

# ─── Deploy Production ───
deploy_production:
  stage: deploy
  image: bitnami/kubectl:latest
  needs: [deploy_staging]
  script:
    - kubectl config use-context $KUBE_CONTEXT_PROD
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
    - kubectl rollout status deployment/app --timeout=300s
  environment:
    name: production
    url: https://example.com
  when: manual
  allow_failure: false
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
```

---

## 23. Pipeline para Frontend (React/Angular/Vue)

### 23.1 GitHub Actions — React

```yaml
name: React CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm run test -- --coverage --watchAll=false
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: build/

  deploy:
    needs: ci
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: build/

      - uses: actions/configure-pages@v5

      - uses: actions/upload-pages-artifact@v3
        with:
          path: build/

      - id: deployment
        uses: actions/deploy-pages@v4
```

### 23.2 GitLab CI/CD — Angular

```yaml
image: node:22-alpine

stages:
  - install
  - quality
  - build
  - deploy

cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/

install:
  stage: install
  script:
    - npm ci

lint:
  stage: quality
  needs: [install]
  script:
    - npx ng lint

test:
  stage: quality
  needs: [install]
  image: node:22                             # Precisa de navegador para testes
  before_script:
    - apt-get update && apt-get install -y chromium
    - export CHROME_BIN=/usr/bin/chromium
  script:
    - npx ng test --watch=false --browsers=ChromeHeadless --code-coverage
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  needs: [lint, test]
  script:
    - npx ng build --configuration=production
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

pages:                                       # Deploy para GitLab Pages
  stage: deploy
  needs: [build]
  script:
    - mv dist/*/browser public               # Angular 17+ usa browser/
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 23.3 Deploy em CDN / S3

```yaml
# GitHub Actions — Deploy para AWS S3 + CloudFront
deploy-cdn:
  runs-on: ubuntu-latest
  needs: ci
  if: github.ref == 'refs/heads/main'
  permissions:
    id-token: write
    contents: read
  environment: production
  steps:
    - uses: actions/download-artifact@v4
      with:
        name: build

    - uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1

    - name: Sync to S3
      run: aws s3 sync build/ s3://${{ vars.S3_BUCKET }} --delete

    - name: Invalidate CloudFront
      run: |
        aws cloudfront create-invalidation \
          --distribution-id ${{ vars.CF_DISTRIBUTION_ID }} \
          --paths "/*"
```

---

## 24. Pipeline para Mobile — React Native e Flutter

### 24.1 React Native — GitHub Actions

```yaml
# .github/workflows/mobile.yml
name: React Native CI/CD

on:
  push:
    branches: [main]
    paths: ['mobile/**']
  pull_request:
    paths: ['mobile/**']

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: mobile
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: mobile/package-lock.json
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage

  build-android:
    needs: test
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: mobile
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: mobile/package-lock.json

      - run: npm ci

      - name: Cache Gradle
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: gradle-${{ hashFiles('mobile/android/gradle/wrapper/gradle-wrapper.properties') }}

      - name: Build APK
        working-directory: mobile/android
        run: ./gradlew assembleRelease

      - uses: actions/upload-artifact@v4
        with:
          name: android-apk
          path: mobile/android/app/build/outputs/apk/release/

  build-ios:
    needs: test
    runs-on: macos-latest
    defaults:
      run:
        working-directory: mobile
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: mobile/package-lock.json

      - run: npm ci

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'
          bundler-cache: true
          working-directory: mobile/ios

      - name: Install CocoaPods
        working-directory: mobile/ios
        run: bundle exec pod install

      - name: Build iOS
        working-directory: mobile/ios
        run: |
          xcodebuild -workspace App.xcworkspace \
            -scheme App -configuration Release \
            -sdk iphoneos \
            -archivePath build/App.xcarchive archive \
            CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
```

### 24.2 Flutter — GitLab CI/CD

```yaml
image: ghcr.io/cirruslabs/flutter:latest

stages:
  - test
  - build
  - distribute

variables:
  PUB_CACHE: "$CI_PROJECT_DIR/.pub-cache"

cache:
  key: flutter-$CI_COMMIT_REF_SLUG
  paths:
    - .pub-cache/

test:
  stage: test
  script:
    - flutter pub get
    - flutter analyze
    - flutter test --coverage

build_android:
  stage: build
  needs: [test]
  script:
    - flutter build apk --release
  artifacts:
    paths:
      - build/app/outputs/flutter-apk/app-release.apk
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

build_ios:
  stage: build
  needs: [test]
  tags:
    - macos
  script:
    - flutter build ios --release --no-codesign
  artifacts:
    paths:
      - build/ios/iphoneos/Runner.app
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 24.3 Distribuição — Fastlane, TestFlight e Play Store

```yaml
# GitHub Actions — Distribuição com Fastlane
distribute:
  needs: [build-android, build-ios]
  runs-on: macos-latest
  if: startsWith(github.ref, 'refs/tags/v')
  steps:
    - uses: actions/checkout@v4

    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.3'
        bundler-cache: true

    - uses: actions/download-artifact@v4
      with:
        name: android-apk
        path: android-build/

    - name: Upload to Play Store (internal track)
      env:
        SUPPLY_JSON_KEY_DATA: ${{ secrets.GOOGLE_PLAY_JSON_KEY }}
      run: bundle exec fastlane android deploy

    - name: Upload to TestFlight
      env:
        APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.ASC_KEY_ID }}
        APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
        APP_STORE_CONNECT_API_KEY: ${{ secrets.ASC_PRIVATE_KEY }}
      run: bundle exec fastlane ios beta
```

```ruby
# fastlane/Fastfile
platform :android do
  lane :deploy do
    upload_to_play_store(
      track: 'internal',
      aab: 'android-build/app-release.aab'
    )
  end
end

platform :ios do
  lane :beta do
    build_app(scheme: "App")
    upload_to_testflight(skip_waiting_for_build_processing: true)
  end
end
```

```yaml
# GitLab — Firebase App Distribution (alternativa)
distribute_firebase:
  stage: distribute
  image: node:22-alpine
  needs: [build_android]
  script:
    - npm install -g firebase-tools
    - |
      firebase appdistribution:distribute \
        build/app/outputs/flutter-apk/app-release.apk \
        --app $FIREBASE_APP_ID \
        --groups "qa-team" \
        --release-notes "Build $CI_COMMIT_SHORT_SHA"
  rules:
    - if: $CI_COMMIT_TAG
```

### 24.4 Code Signing no CI/CD

| Plataforma | Abordagem | Armazenamento |
|---|---|---|
| **Android** | Keystore (.jks) em Base64 como secret | GitHub Secrets / GitLab CI Variables |
| **iOS** | Provisioning profiles via Fastlane Match | Repositório Git privado criptografado |
| **Play Store** | Google Play JSON Key | Secret/CI Variable (masked) |
| **App Store** | App Store Connect API Key | Secret/CI Variable (masked) |

```yaml
# Decodificar keystore Android no pipeline
- name: Decode keystore
  run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > android/app/release.keystore
  env:
    KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
```

---

## 25. Testes Avançados no Pipeline — E2E, Performance e Quality Gates

### 25.1 Testes E2E com Playwright

```yaml
# GitHub Actions
e2e-playwright:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '22'
        cache: 'npm'
    - run: npm ci
    - run: npx playwright install --with-deps chromium firefox
    - name: Run E2E Tests
      run: npx playwright test
      env:
        BASE_URL: https://staging.example.com
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 14
```

```yaml
# GitLab CI/CD
e2e_playwright:
  stage: test
  image: mcr.microsoft.com/playwright:v1.49.0-noble
  needs: [deploy_staging]
  script:
    - npm ci
    - npx playwright test
  variables:
    BASE_URL: https://staging.example.com
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 2 weeks
```

### 25.2 Testes E2E com Cypress

```yaml
# GitHub Actions
e2e-cypress:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4
    - uses: cypress-io/github-action@v6
      with:
        browser: chrome
        config: baseUrl=https://staging.example.com
      env:
        CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
    - uses: actions/upload-artifact@v4
      if: failure()
      with:
        name: cypress-screenshots
        path: cypress/screenshots/
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: cypress-videos
        path: cypress/videos/
```

```yaml
# GitLab CI/CD
e2e_cypress:
  stage: test
  image: cypress/browsers:latest
  needs: [deploy_staging]
  script:
    - npm ci
    - npx cypress run --browser chrome
  variables:
    CYPRESS_BASE_URL: https://staging.example.com
  artifacts:
    when: always
    paths:
      - cypress/screenshots/
      - cypress/videos/
    expire_in: 1 week
```

### 25.3 Testes de Performance com k6

```yaml
# GitHub Actions
performance-test:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4

    - name: Run k6 load test
      uses: grafana/k6-action@v0.4.0
      with:
        filename: tests/load/smoke.js
      env:
        K6_BASE_URL: https://staging.example.com

    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: k6-results
        path: results.json
```

```javascript
// tests/load/smoke.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },     // Ramp-up
    { duration: '1m',  target: 20 },      // Sustain
    { duration: '10s', target: 0 },       // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],     // 95% das requests < 500ms
    http_req_failed:   ['rate<0.01'],     // < 1% de falha
  },
};

export default function () {
  const res = http.get(`${__ENV.K6_BASE_URL}/api/health`);
  check(res, {
    'status 200':       (r) => r.status === 200,
    'duration < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

```yaml
# GitLab CI/CD — k6 com Gatling como alternativa
performance_k6:
  stage: test
  image:
    name: grafana/k6:latest
    entrypoint: [""]
  needs: [deploy_staging]
  script:
    - k6 run --out json=results.json tests/load/smoke.js
  variables:
    K6_BASE_URL: https://staging.example.com
  artifacts:
    paths:
      - results.json
    expire_in: 1 week
  allow_failure: true

# Gatling (alternativa para projetos Java)
performance_gatling:
  stage: test
  image: eclipse-temurin:21-jdk
  script:
    - ./mvnw gatling:test
  artifacts:
    paths:
      - target/gatling/*/
    expire_in: 1 week
  allow_failure: true
```

### 25.4 Testcontainers no CI

```yaml
# GitHub Actions — Docker já disponível nos GitHub-hosted runners
test-with-testcontainers:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'
    - name: Run tests with Testcontainers
      run: ./mvnw -B verify -P integration-test
```

```yaml
# GitLab CI/CD — requer Docker-in-Docker
test_with_testcontainers:
  stage: test
  image: eclipse-temurin:21-jdk
  services:
    - docker:27-dind
  variables:
    DOCKER_HOST: "tcp://docker:2375"
    DOCKER_TLS_CERTDIR: ""
    TESTCONTAINERS_HOST_OVERRIDE: "docker"
    TESTCONTAINERS_RYUK_DISABLED: "true"
  script:
    - ./mvnw -B verify -P integration-test
  artifacts:
    reports:
      junit: target/failsafe-reports/TEST-*.xml
```

```java
// Spring Boot com @ServiceConnection (Spring Boot 3.1+)
@SpringBootTest
@Testcontainers
class PedidoRepositoryIT {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:17")
            .withDatabaseName("testdb");

    @Autowired
    private PedidoRepository repository;

    @Test
    void deveSalvarPedido() {
        var pedido = new Pedido("cliente-1", BigDecimal.valueOf(99.90));
        var salvo = repository.save(pedido);
        assertThat(salvo.getId()).isNotNull();
    }
}
```

### 25.5 Quality Gates com SonarQube / SonarCloud

```yaml
# GitHub Actions — SonarCloud
sonar:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'
    - name: Build and analyze
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: |
        ./mvnw -B verify sonar:sonar \
          -Dsonar.projectKey=org_project \
          -Dsonar.organization=org \
          -Dsonar.host.url=https://sonarcloud.io \
          -Dsonar.qualitygate.wait=true
```

```yaml
# GitLab CI/CD — SonarQube self-hosted
sonarqube:
  stage: test
  image: eclipse-temurin:21-jdk
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
    GIT_DEPTH: "0"
  cache:
    key: sonar-$CI_COMMIT_REF_SLUG
    paths:
      - .sonar/cache
  script:
    - |
      ./mvnw -B verify sonar:sonar \
        -Dsonar.host.url=$SONAR_HOST_URL \
        -Dsonar.token=$SONAR_TOKEN \
        -Dsonar.qualitygate.wait=true
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

| Métrica (Quality Gate padrão) | Condição |
|---|---|
| **Coverage** | Cobertura em código novo ≥ 80% |
| **Duplications** | Duplicação em código novo ≤ 3% |
| **Maintainability** | Rating A (debt ratio ≤ 5%) |
| **Reliability** | Rating A (zero bugs novos) |
| **Security** | Rating A (zero vulnerabilidades novas) |
| **Security Hotspots** | 100% revisados |

---

## 26. Deploy em Cloud — AWS, Azure e GCP

### 26.1 AWS — OIDC + ECS (GitHub Actions)

```yaml
deploy-aws:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
    contents: read
  environment: production
  steps:
    - uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789:role/github-actions
        aws-region: us-east-1

    - uses: aws-actions/amazon-ecr-login@v2

    - name: Push to ECR
      run: |
        docker tag app:latest $ECR_REGISTRY/app:${{ github.sha }}
        docker push $ECR_REGISTRY/app:${{ github.sha }}

    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster production \
          --service app \
          --force-new-deployment
```

### 26.2 Azure — Container Apps (GitHub Actions)

```yaml
deploy-azure:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
  environment: production
  steps:
    - uses: azure/login@v2
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - uses: azure/container-apps-deploy-action@v2
      with:
        resourceGroup: my-rg
        containerAppName: my-app
        imageToDeploy: myregistry.azurecr.io/app:${{ github.sha }}
```

### 26.3 GCP — Cloud Run (GitHub Actions)

```yaml
deploy-gcp:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
    contents: read
  environment: production
  steps:
    - uses: google-github-actions/auth@v2
      with:
        workload_identity_provider: projects/123/locations/global/workloadIdentityPools/github/providers/github
        service_account: deploy@project.iam.gserviceaccount.com

    - uses: google-github-actions/deploy-cloudrun@v2
      with:
        service: app
        region: us-central1
        image: gcr.io/project/app:${{ github.sha }}
```

### 26.4 OIDC vs Credenciais Estáticas

```
┌───────────────────────────────────────────────────────────────────────┐
│                                                                       │
│  Credenciais Estáticas (não recomendado):                             │
│  ┌──────────┐    Secret (AWS_ACCESS_KEY)    ┌──────────┐             │
│  │ Pipeline │ ────────────────────────────▶  │  Cloud   │             │
│  └──────────┘    Risco: vazamento           └──────────┘             │
│                                                                       │
│  OIDC (recomendado):                                                  │
│  ┌──────────┐  JWT Token   ┌──────────┐  STS Token  ┌──────────┐    │
│  │ Pipeline │ ───────────▶ │ Identity │ ──────────▶  │  Cloud   │    │
│  └──────────┘   curta      │ Provider │  temporário  └──────────┘    │
│                  duração   └──────────┘                               │
│                                                                       │
│  Benefícios OIDC:                                                     │
│  ✓ Sem secrets para rotacionar                                        │
│  ✓ Tokens de curta duração                                            │
│  ✓ Auditável (quem, quando, de onde)                                  │
│  ✓ Suportado: AWS, GCP, Azure, HashiCorp Vault                       │
└───────────────────────────────────────────────────────────────────────┘
```

---

## 27. Estratégias de Deploy — Blue-Green, Canary e Rollback

### 27.1 Visão Geral

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    Estratégias de Deploy                                      │
│                                                                              │
│  Blue-Green              Canary                  Rolling Update              │
│  ┌──────┐ ┌──────┐      ┌──────────────┐        ┌──────────────┐           │
│  │ Blue │ │Green │      │  90%    10%  │        │ ██░░░░░░░░░░ │ 10%       │
│  │ v1.0 │ │ v2.0 │      │  v1.0  v2.0  │        │ ████░░░░░░░░ │ 30%       │
│  │ ████ │ │ ████ │      │  ████  ██    │        │ ████████░░░░ │ 60%       │
│  └──┬───┘ └──┬───┘      └──────────────┘        │ ████████████ │ 100%      │
│     │ switch │           gradual shift           └──────────────┘           │
│     └────────┘                                    one-by-one               │
│  100% → 0%   0% → 100%                                                     │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Estratégia | Risco | Rollback | Custo Infra | Quando Usar |
|---|---|---|---|---|
| **Blue-Green** | Baixo | Instantâneo (switch) | 2x (dois ambientes) | Mudanças críticas, zero downtime |
| **Canary** | Muito baixo | Rápido (reverter %) | 1x + canary | Validar com tráfego real antes de 100% |
| **Rolling Update** | Médio | Lento (pod a pod) | 1x | Padrão K8s, atualizações incrementais |
| **Recreate** | Alto | Lento (novo deploy) | 1x | Aceita downtime, breaking changes |

### 27.2 Blue-Green com Kubernetes

```yaml
# GitHub Actions — Blue-Green
deploy-blue-green:
  runs-on: ubuntu-latest
  environment: production
  steps:
    - uses: actions/checkout@v4

    - name: Determine target slot
      id: slot
      run: |
        CURRENT=$(kubectl get svc app -o jsonpath='{.spec.selector.slot}')
        if [ "$CURRENT" = "blue" ]; then
          echo "target=green" >> $GITHUB_OUTPUT
        else
          echo "target=blue" >> $GITHUB_OUTPUT
        fi

    - name: Deploy to inactive slot
      run: |
        kubectl set image deployment/app-${{ steps.slot.outputs.target }} \
          app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        kubectl rollout status deployment/app-${{ steps.slot.outputs.target }} --timeout=300s

    - name: Smoke test inactive slot
      run: |
        SLOT_URL=$(kubectl get svc app-${{ steps.slot.outputs.target }} -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        curl -f "http://$SLOT_URL/actuator/health" || exit 1

    - name: Switch traffic
      run: |
        kubectl patch svc app -p '{"spec":{"selector":{"slot":"${{ steps.slot.outputs.target }}"}}}'

    - name: Verify production
      run: curl -f https://example.com/actuator/health || exit 1
```

### 27.3 Canary com Kubernetes (Nginx Ingress)

```yaml
# GitLab CI/CD — Canary progressivo
deploy_canary_10:
  stage: deploy
  script:
    - kubectl apply -f k8s/canary-deployment.yml
    - kubectl set image deployment/app-canary app=$IMAGE_TAG
    - kubectl rollout status deployment/app-canary --timeout=120s
    - |
      # Configurar Ingress canary com 10% do tráfego
      kubectl annotate ingress app-canary \
        nginx.ingress.kubernetes.io/canary="true" \
        nginx.ingress.kubernetes.io/canary-weight="10" --overwrite
  environment:
    name: production-canary
  rules:
    - if: $CI_COMMIT_TAG

promote_canary:
  stage: deploy
  needs: [deploy_canary_10]
  script:
    - kubectl set image deployment/app app=$IMAGE_TAG
    - kubectl rollout status deployment/app --timeout=300s
    - kubectl delete deployment app-canary
    - kubectl delete ingress app-canary
  environment:
    name: production
  when: manual
  rules:
    - if: $CI_COMMIT_TAG

rollback_canary:
  stage: deploy
  needs: [deploy_canary_10]
  script:
    - kubectl delete deployment app-canary
    - kubectl delete ingress app-canary
  environment:
    name: production-canary
    action: stop
  when: manual
  rules:
    - if: $CI_COMMIT_TAG
```

### 27.4 Rollback Automatizado

```yaml
# GitHub Actions — Rollback automático em falha do health check
deploy-with-rollback:
  runs-on: ubuntu-latest
  environment: production
  steps:
    - name: Save current version
      id: current
      run: |
        CURRENT_IMAGE=$(kubectl get deployment app -o jsonpath='{.spec.template.spec.containers[0].image}')
        echo "image=$CURRENT_IMAGE" >> $GITHUB_OUTPUT

    - name: Deploy new version
      run: |
        kubectl set image deployment/app app=${{ env.IMAGE_TAG }}
        kubectl rollout status deployment/app --timeout=300s

    - name: Health check
      id: health
      continue-on-error: true
      run: |
        for i in $(seq 1 5); do
          if curl -sf https://example.com/actuator/health; then
            echo "healthy=true" >> $GITHUB_OUTPUT
            exit 0
          fi
          sleep 10
        done
        echo "healthy=false" >> $GITHUB_OUTPUT
        exit 1

    - name: Rollback on failure
      if: steps.health.outputs.healthy != 'true'
      run: |
        echo "Health check failed — rolling back to ${{ steps.current.outputs.image }}"
        kubectl set image deployment/app app=${{ steps.current.outputs.image }}
        kubectl rollout status deployment/app --timeout=300s
```

```yaml
# GitLab CI/CD — Rollback via kubectl rollout undo
deploy_production:
  stage: deploy
  script:
    - kubectl set image deployment/app app=$IMAGE_TAG
    - kubectl rollout status deployment/app --timeout=300s
    - |
      # Health check
      for i in $(seq 1 5); do
        curl -sf https://example.com/actuator/health && exit 0
        sleep 10
      done
      echo "Health check failed — executing rollback"
      kubectl rollout undo deployment/app
      kubectl rollout status deployment/app --timeout=300s
      exit 1
  environment:
    name: production
```

---

## 28. Database Migrations no Pipeline — Flyway e Liquibase

### 28.1 Flyway — Validação no CI

```yaml
# GitHub Actions — Validar migrations antes do merge
validate-migrations:
  runs-on: ubuntu-latest
  services:
    postgres:
      image: postgres:17
      env:
        POSTGRES_DB: migration_test
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
      ports:
        - 5432:5432
      options: >-
        --health-cmd="pg_isready -U test"
        --health-interval=10s
        --health-timeout=5s
        --health-retries=5
  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'

    - name: Validate Flyway migrations
      run: ./mvnw -B flyway:validate flyway:info
      env:
        SPRING_FLYWAY_URL: jdbc:postgresql://localhost:5432/migration_test
        SPRING_FLYWAY_USER: test
        SPRING_FLYWAY_PASSWORD: test

    - name: Run migrations (dry-run)
      run: ./mvnw -B flyway:migrate
      env:
        SPRING_FLYWAY_URL: jdbc:postgresql://localhost:5432/migration_test
        SPRING_FLYWAY_USER: test
        SPRING_FLYWAY_PASSWORD: test
```

### 28.2 Flyway — Aplicação no CD

```yaml
# Job separado de migration antes do deploy da aplicação
migrate-database:
  runs-on: ubuntu-latest
  needs: build
  environment: production
  steps:
    - uses: actions/checkout@v4

    - name: Run Flyway migrate
      uses: docker://flyway/flyway:10
      with:
        args: >
          -url=${{ secrets.PROD_DB_URL }}
          -user=${{ secrets.PROD_DB_USER }}
          -password=${{ secrets.PROD_DB_PASSWORD }}
          -locations=filesystem:./src/main/resources/db/migration
          migrate

deploy-app:
  needs: migrate-database
  # ... deploy da aplicação após migrations
```

### 28.3 Liquibase no Pipeline

```yaml
# GitLab CI/CD — Liquibase
validate_changelog:
  stage: test
  image: liquibase/liquibase:latest
  services:
    - name: postgres:17
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
  script:
    - |
      liquibase \
        --url=jdbc:postgresql://db:5432/testdb \
        --username=test \
        --password=test \
        --changelog-file=src/main/resources/db/changelog/db.changelog-master.xml \
        validate
    - |
      liquibase \
        --url=jdbc:postgresql://db:5432/testdb \
        --username=test \
        --password=test \
        --changelog-file=src/main/resources/db/changelog/db.changelog-master.xml \
        update-testing-rollback
  rules:
    - changes:
        - src/main/resources/db/**

migrate_production:
  stage: deploy
  image: liquibase/liquibase:latest
  needs: [validate_changelog]
  script:
    - |
      liquibase \
        --url=$PROD_DB_URL \
        --username=$PROD_DB_USER \
        --password=$PROD_DB_PASSWORD \
        --changelog-file=src/main/resources/db/changelog/db.changelog-master.xml \
        update
  environment:
    name: production
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 28.4 Boas Práticas de Migrations no CI/CD

| Prática | Descrição |
|---|---|
| **Validar no CI** | Executar `validate` e `migrate` em banco limpo em cada PR |
| **Separar migration do deploy** | Job de migration roda antes e independente do deploy da app |
| **Migrations reversíveis** | Sempre escrever `undo`/`rollback` (Flyway Teams/Liquibase) |
| **Nunca alterar migrations** | Migrations aplicadas são imutáveis — criar nova para corrigir |
| **Testar com dados** | Usar banco com volume representativo de dados no staging |
| **Backward-compatible** | Migrations devem funcionar com a versão antiga E nova da app |
| **Lock de concorrência** | Flyway/Liquibase usam lock table — seguro para múltiplas instâncias |

---

## 29. Feature Flags e Progressive Delivery

### 29.1 Conceitos

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    Progressive Delivery                                       │
│                                                                              │
│  Deploy ≠ Release                                                            │
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │ Deploy Code  │───▶│ Feature Flag │───▶│ Release to   │                   │
│  │ (dark launch)│    │ Controls Who │    │ All Users    │                   │
│  └──────────────┘    │ Sees Feature │    └──────────────┘                   │
│                      └──────────────┘                                       │
│                            │                                                 │
│                     ┌──────┴──────┐                                          │
│                     │ 1% → 5%    │  gradual rollout                         │
│                     │ → 25% → 100%│                                          │
│                     └─────────────┘                                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Conceito | Descrição |
|---|---|
| **Feature Flag** | Toggle que liga/desliga uma feature sem redeploy |
| **Dark Launch** | Código em produção mas invisível aos usuários |
| **Progressive Rollout** | Liberação gradual (1% → 10% → 50% → 100%) |
| **A/B Testing** | Variantes diferentes para grupos distintos |
| **Kill Switch** | Desabilitar feature instantaneamente em caso de problema |

### 29.2 Unleash (Open Source)

```yaml
# docker-compose.yml — Unleash server
services:
  unleash:
    image: unleashorg/unleash-server:latest
    ports:
      - "4242:4242"
    environment:
      DATABASE_URL: postgres://unleash:password@db:5432/unleash
    depends_on:
      - db
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: unleash
      POSTGRES_USER: unleash
      POSTGRES_PASSWORD: password
```

```java
// Spring Boot — integração com Unleash
@Configuration
public class UnleashConfig {
    @Bean
    public Unleash unleash() {
        return new DefaultUnleash(UnleashConfig.builder()
            .appName("minha-app")
            .instanceId("instance-1")
            .unleashAPI("http://unleash:4242/api")
            .apiKey("default:development.unleash-insecure-api-token")
            .build());
    }
}

@RestController
public class PagamentoController {

    private final Unleash unleash;

    @PostMapping("/api/pagamentos")
    public ResponseEntity<?> processar(@RequestBody PagamentoDTO dto) {
        if (unleash.isEnabled("novo-gateway-pagamento")) {
            return novoGateway.processar(dto);     // Feature nova
        }
        return gatewayLegado.processar(dto);       // Comportamento atual
    }
}
```

### 29.3 Integração com CI/CD

```yaml
# GitHub Actions — Habilitar feature flag após deploy bem-sucedido
enable-feature:
  runs-on: ubuntu-latest
  needs: [deploy-production, smoke-test]
  steps:
    - name: Enable feature for 10% of users
      run: |
        curl -X POST "$UNLEASH_URL/api/admin/projects/default/features/novo-gateway-pagamento/environments/production/strategies" \
          -H "Authorization: ${{ secrets.UNLEASH_API_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d '{
            "name": "gradualRollout",
            "parameters": { "percentage": "10", "stickiness": "userId" }
          }'

# Rollback instantâneo — desabilitar flag sem redeploy
rollback-feature:
  runs-on: ubuntu-latest
  if: failure()
  steps:
    - name: Disable feature flag
      run: |
        curl -X POST "$UNLEASH_URL/api/admin/projects/default/features/novo-gateway-pagamento/environments/production/off" \
          -H "Authorization: ${{ secrets.UNLEASH_API_TOKEN }}"
```

### 29.4 Alternativas

| Ferramenta | Tipo | Destaque |
|---|---|---|
| **Unleash** | Open Source / SaaS | Simples, SDK em múltiplas linguagens |
| **LaunchDarkly** | SaaS | Enterprise, targeting avançado, analytics |
| **Flipt** | Open Source | Leve, sem dependência de banco externo |
| **Split** | SaaS | Foco em experimentação e métricas |
| **ConfigCat** | SaaS | Pricing por feature flag, não por MAU |
| **Spring Cloud Config** | Self-hosted | Se já usa Spring Cloud (sem UI de gestão) |

---

## 30. Infrastructure as Code no Pipeline — Terraform

### 30.1 Pipeline Terraform — GitHub Actions

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
    paths: ['infra/**']
  pull_request:
    paths: ['infra/**']

permissions:
  contents: read
  pull-requests: write
  id-token: write

defaults:
  run:
    working-directory: infra

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_TERRAFORM_ROLE }}
          aws-region: us-east-1

      - run: terraform init

      - run: terraform validate

      - name: Terraform Plan
        id: plan
        run: terraform plan -no-color -out=tfplan
        continue-on-error: true

      - name: Comment PR with plan
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const plan = `${{ steps.plan.outputs.stdout }}`;
            const truncated = plan.length > 60000 ? plan.substring(0, 60000) + '\n...(truncated)' : plan;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `#### Terraform Plan 📖\n<details><summary>Show Plan</summary>\n\n\`\`\`terraform\n${truncated}\n\`\`\`\n</details>`
            });

      - uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: infra/tfplan

  apply:
    needs: plan
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: infrastructure
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_TERRAFORM_ROLE }}
          aws-region: us-east-1

      - run: terraform init

      - uses: actions/download-artifact@v4
        with:
          name: tfplan
          path: infra/

      - run: terraform apply -auto-approve tfplan
```

### 30.2 Pipeline Terraform — GitLab CI/CD

```yaml
# .gitlab-ci.yml
image:
  name: hashicorp/terraform:1.9
  entrypoint: [""]

stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: infra

cache:
  key: terraform-$CI_COMMIT_REF_SLUG
  paths:
    - ${TF_ROOT}/.terraform/

validate:
  stage: validate
  script:
    - cd $TF_ROOT
    - terraform init -backend=false
    - terraform validate
    - terraform fmt -check -recursive

plan:
  stage: plan
  script:
    - cd $TF_ROOT
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
    expire_in: 1 day
  rules:
    - changes:
        - infra/**

apply:
  stage: apply
  needs: [plan]
  script:
    - cd $TF_ROOT
    - terraform init
    - terraform apply -auto-approve tfplan
  environment:
    name: infrastructure
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      changes:
        - infra/**
```

### 30.3 Drift Detection

```yaml
# GitHub Actions — detectar drift em schedule
name: Terraform Drift Detection

on:
  schedule:
    - cron: '0 8 * * 1-5'                 # Seg-Sex 8h

jobs:
  drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
        working-directory: infra
      - name: Check for drift
        id: drift
        working-directory: infra
        run: |
          terraform plan -detailed-exitcode -no-color > plan.txt 2>&1 || EXIT=$?
          if [ "$EXIT" = "2" ]; then
            echo "drift=true" >> $GITHUB_OUTPUT
          fi
      - name: Alert on drift
        if: steps.drift.outputs.drift == 'true'
        run: |
          curl -X POST "${{ secrets.SLACK_WEBHOOK }}" \
            -d '{"text":"⚠️ Terraform drift detectado! Verifique o pipeline."}'
```

---

## 31. GitOps com ArgoCD e Flux

### 31.1 Separação CI vs CD

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    GitOps — CI/CD Separation                                  │
│                                                                              │
│  ┌─────────────────────────────┐    ┌──────────────────────────────────┐    │
│  │  CI Pipeline (GitHub/GitLab) │    │  CD — GitOps (ArgoCD/Flux)       │    │
│  │                             │    │                                  │    │
│  │  Code Repo                  │    │  Config Repo (GitOps)            │    │
│  │  ┌───────┐                  │    │  ┌────────────────────┐          │    │
│  │  │ Build │ → Test → Image   │    │  │ k8s/deployment.yml │          │    │
│  │  └───────┘         Push ────┼───▶│  │ image: app:v2.1.0  │          │    │
│  │                             │    │  └─────────┬──────────┘          │    │
│  └─────────────────────────────┘    │            │ watch               │    │
│                                     │  ┌─────────▼──────────┐          │    │
│                                     │  │ ArgoCD / Flux       │          │    │
│                                     │  │ Sync → Kubernetes   │          │    │
│                                     │  └────────────────────┘          │    │
│                                     └──────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Aspecto | CI Tradicional (push) | GitOps (pull) |
|---|---|---|
| **Quem faz deploy** | Pipeline push para o cluster | Agent no cluster faz pull |
| **Source of truth** | Pipeline | Repositório Git |
| **Auditoria** | Logs do pipeline | Git history |
| **Rollback** | Re-run pipeline ou kubectl | `git revert` |
| **Drift detection** | Manual | Automático e contínuo |
| **Acesso ao cluster** | Pipeline precisa de kubeconfig | Apenas o agent no cluster |

### 31.2 ArgoCD — Workflow

```yaml
# CI Pipeline — atualiza imagem no repo de configuração
update-gitops-repo:
  runs-on: ubuntu-latest
  needs: docker-build
  steps:
    - uses: actions/checkout@v4
      with:
        repository: org/gitops-config
        token: ${{ secrets.GITOPS_TOKEN }}

    - name: Update image tag
      run: |
        cd apps/minha-app/overlays/production
        kustomize edit set image app=${{ env.REGISTRY }}/app:${{ github.sha }}

    - name: Commit and push
      run: |
        git config user.name "github-actions"
        git config user.email "actions@github.com"
        git add .
        git commit -m "chore: update app image to ${{ github.sha }}"
        git push
```

```yaml
# ArgoCD Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minha-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-config
    targetRevision: main
    path: apps/minha-app/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true                       # Corrige drift automaticamente
    syncOptions:
      - CreateNamespace=true
```

### 31.3 Flux — Workflow

```yaml
# Flux — Image Update Automation
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: app
  namespace: flux-system
spec:
  image: ghcr.io/org/app
  interval: 5m

---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: app
  policy:
    semver:
      range: ">=1.0.0"

---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: app
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: gitops-config
  git:
    commit:
      author:
        name: fluxcdbot
        email: flux@example.com
      messageTemplate: "chore: update {{ .AutomationObject.Name }} to {{ .NewTag }}"
    push:
      branch: main
  update:
    path: ./apps/minha-app
    strategy: Setters
```

### 31.4 Comparação ArgoCD vs Flux

| Aspecto | ArgoCD | Flux |
|---|---|---|
| **UI** | Dashboard web rico | Sem UI nativa (usar Weave GitOps) |
| **Multi-cluster** | Suporte nativo | Via Flux multi-tenancy |
| **SSO** | OIDC, LDAP, SAML | N/A (não tem UI) |
| **Helm** | Suporte nativo | Helm Controller |
| **Image Update** | Argo Image Updater | Image Automation Controller |
| **Notificações** | Argo Notifications | Flux Notification Controller |
| **Maturidade** | CNCF Graduated | CNCF Graduated |
| **Quando usar** | Equipes que querem UI e visibilidade | Equipes que preferem CLI/GitOps puro |

---

## 32. Segurança no Pipeline — SAST, SCA e Secrets

### 32.1 GitHub Actions — Segurança

```yaml
# SAST com CodeQL
security-sast:
  runs-on: ubuntu-latest
  permissions:
    security-events: write
  steps:
    - uses: actions/checkout@v4

    - uses: github/codeql-action/init@v3
      with:
        languages: java

    - uses: github/codeql-action/autobuild@v3

    - uses: github/codeql-action/analyze@v3

# SCA — Dependency Review (PRs)
dependency-review:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - uses: actions/dependency-review-action@v4
      with:
        fail-on-severity: high

# Secret Scanning — automático no GitHub (não requer workflow)

# OWASP Dependency Check
dependency-check:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'
    - run: ./mvnw -B org.owasp:dependency-check-maven:check
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: dependency-check-report
        path: target/dependency-check-report.html
```

### 32.2 GitLab CI/CD — Segurança (Templates Built-in)

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/License-Scanning.gitlab-ci.yml

# Os templates acima criam jobs automaticamente
# Resultados aparecem no MR como widget de segurança (Ultimate)

# Customizar variáveis dos templates
variables:
  SAST_EXCLUDED_PATHS: "spec,test,tests"
  DS_EXCLUDED_ANALYZERS: "bundler-audit"
  SECRET_DETECTION_EXCLUDED_PATHS: "test/"
```

### 32.3 Trivy — Scanner de Vulnerabilidades (Ambas Plataformas)

```yaml
# GitHub Actions
trivy-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'ghcr.io/org/app:latest'
        format: 'sarif'
        output: 'trivy-results.sarif'
        severity: 'CRITICAL,HIGH'

    - uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'

# GitLab CI/CD
trivy_scan:
  stage: test
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --exit-code 1 --severity CRITICAL,HIGH $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  allow_failure: true
```

---

## 33. Compliance, Auditoria e SBOM

### 33.1 GitLab Compliance Pipelines (Ultimate)

Compliance Pipelines garantem que jobs obrigatórios rodem em toda pipeline, sem que desenvolvedores possam remover ou modificar.

```yaml
# Configurado em Group > Settings > General > Compliance Pipeline
# Arquivo no repo de compliance: .compliance-gitlab-ci.yml

include:
  - project: '$CI_CONFIG_PATH'            # Pipeline do projeto
    file: '.gitlab-ci.yml'

# Jobs de compliance adicionados automaticamente
sast_compliance:
  stage: test
  script:
    - echo "Running mandatory SAST scan"
  allow_failure: false

license_check:
  stage: test
  script:
    - echo "Running license compliance check"
  allow_failure: false

audit_trail:
  stage: .post
  script:
    - |
      curl -X POST "$AUDIT_API_URL/deploys" \
        -H "Authorization: Bearer $AUDIT_TOKEN" \
        -d "{
          \"project\": \"$CI_PROJECT_PATH\",
          \"commit\": \"$CI_COMMIT_SHA\",
          \"pipeline\": \"$CI_PIPELINE_ID\",
          \"user\": \"$GITLAB_USER_LOGIN\",
          \"environment\": \"$CI_ENVIRONMENT_NAME\",
          \"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"
        }"
  rules:
    - if: $CI_ENVIRONMENT_NAME
```

### 33.2 Audit Trail de Deploys

```yaml
# GitHub Actions — Registrar deploy no log de auditoria
audit-deploy:
  runs-on: ubuntu-latest
  needs: deploy-production
  if: always()
  steps:
    - name: Record deploy event
      env:
        DEPLOY_STATUS: ${{ needs.deploy-production.result }}
      run: |
        curl -X POST "${{ vars.AUDIT_API_URL }}/events" \
          -H "Authorization: Bearer ${{ secrets.AUDIT_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d "{
            \"type\": \"deploy\",
            \"status\": \"$DEPLOY_STATUS\",
            \"repository\": \"${{ github.repository }}\",
            \"commit\": \"${{ github.sha }}\",
            \"actor\": \"${{ github.actor }}\",
            \"environment\": \"production\",
            \"image_tag\": \"${{ needs.docker.outputs.image-tag }}\",
            \"approvers\": \"${{ github.event.review.user.login }}\",
            \"workflow_run\": \"${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}\",
            \"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"
          }"
```

### 33.3 Assinatura de Artefatos — Sigstore / Cosign

```yaml
# GitHub Actions — Assinar imagem Docker com Cosign
sign-image:
  runs-on: ubuntu-latest
  needs: docker-build
  permissions:
    id-token: write
    packages: write
  steps:
    - uses: sigstore/cosign-installer@v3

    - uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Sign image with Cosign (keyless)
      run: |
        cosign sign --yes \
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ needs.docker-build.outputs.digest }}
      env:
        COSIGN_EXPERIMENTAL: 1

    - name: Verify signature
      run: |
        cosign verify \
          --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
          --certificate-identity-regexp="github.com/${{ github.repository }}" \
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ needs.docker-build.outputs.digest }}
```

```yaml
# GitLab CI/CD — Assinar imagem com Cosign
sign_image:
  stage: .post
  image: bitnami/cosign:latest
  needs: [docker_build]
  variables:
    COSIGN_YES: "true"
  id_tokens:
    SIGSTORE_ID_TOKEN:
      aud: sigstore
  script:
    - cosign sign $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
```

### 33.4 SBOM — Software Bill of Materials

```yaml
# GitHub Actions — Gerar SBOM com Syft
generate-sbom:
  runs-on: ubuntu-latest
  needs: docker-build
  permissions:
    contents: write
  steps:
    - uses: anchore/sbom-action@v0
      with:
        image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        format: spdx-json
        output-file: sbom.spdx.json

    - uses: actions/upload-artifact@v4
      with:
        name: sbom
        path: sbom.spdx.json

    - name: Attach SBOM to release
      if: startsWith(github.ref, 'refs/tags/v')
      run: gh release upload ${{ github.ref_name }} sbom.spdx.json
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

```yaml
# GitLab CI/CD — SBOM com CycloneDX
generate_sbom:
  stage: package
  image: eclipse-temurin:21-jdk
  script:
    - ./mvnw org.cyclonedx:cyclonedx-maven-plugin:makeBom
  artifacts:
    paths:
      - target/bom.json
    reports:
      cyclonedx: target/bom.json           # Integração nativa GitLab 16+
    expire_in: 90 days
```

| Formato SBOM | Descrição | Ferramentas |
|---|---|---|
| **SPDX** | Padrão ISO, foco em licenças | Syft, Tern, SPDX Tools |
| **CycloneDX** | OWASP, foco em segurança | CycloneDX Maven/Gradle, Syft |
| **SWID** | ISO/IEC 19770-2, foco enterprise | Pouco usado em CI/CD |

---

## 34. Gerenciamento de Artefatos e Documentação Automática

### 34.1 Package Registries

| Registry | Suporte | Integração |
|---|---|---|
| **GitHub Packages** | Maven, npm, NuGet, Docker, RubyGems | Nativo ao GitHub Actions |
| **GitLab Package Registry** | Maven, npm, NuGet, PyPI, Go, Helm, Docker | Nativo ao GitLab CI/CD |
| **Nexus Repository** | Todos os formatos + proxy de registries | Self-hosted, integração via API |
| **JFrog Artifactory** | Todos os formatos + builds info | SaaS/Self-hosted, CLI `jf` |

```yaml
# GitHub Actions — Publicar JAR no GitHub Packages (Maven)
publish:
  runs-on: ubuntu-latest
  permissions:
    contents: read
    packages: write
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'
        server-id: github
        server-username: GITHUB_ACTOR
        server-password: GITHUB_TOKEN
    - run: ./mvnw -B deploy -DskipTests
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

```xml
<!-- pom.xml — configuração do GitHub Packages -->
<distributionManagement>
    <repository>
        <id>github</id>
        <url>https://maven.pkg.github.com/OWNER/REPO</url>
    </repository>
</distributionManagement>
```

```yaml
# GitLab CI/CD — Publicar no GitLab Package Registry
publish:
  stage: deploy
  image: eclipse-temurin:21-jdk
  script:
    - |
      ./mvnw -B deploy -DskipTests \
        -DaltDeploymentRepository=gitlab::default::${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/maven
  rules:
    - if: $CI_COMMIT_TAG

# npm — publicar no GitLab
publish_npm:
  stage: deploy
  image: node:22-alpine
  script:
    - echo "@scope:registry=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/npm/" > .npmrc
    - echo "//${CI_SERVER_HOST}/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}" >> .npmrc
    - npm publish
  rules:
    - if: $CI_COMMIT_TAG
```

### 34.2 Geração de Javadoc no CI

```yaml
# GitHub Actions — Javadoc → GitHub Pages
javadoc:
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/main'
  permissions:
    pages: write
    id-token: write
  environment:
    name: github-pages
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'
    - run: ./mvnw -B javadoc:javadoc
    - uses: actions/upload-pages-artifact@v3
      with:
        path: target/reports/apidocs/
    - uses: actions/deploy-pages@v4
```

```yaml
# GitLab — Javadoc → GitLab Pages
pages:
  stage: deploy
  image: eclipse-temurin:21-jdk
  script:
    - ./mvnw -B javadoc:javadoc
    - mv target/reports/apidocs public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

### 34.3 Documentação OpenAPI / Swagger

```yaml
# GitHub Actions — Gerar e publicar OpenAPI spec
openapi:
  runs-on: ubuntu-latest
  needs: build
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'
        cache: 'maven'

    - name: Start app and extract OpenAPI
      run: |
        ./mvnw -B spring-boot:run &
        sleep 15
        curl -f http://localhost:8080/v3/api-docs > openapi.json
        curl -f http://localhost:8080/v3/api-docs.yaml > openapi.yaml
        kill %1

    - uses: actions/upload-artifact@v4
      with:
        name: openapi-spec
        path: |
          openapi.json
          openapi.yaml

    - name: Validate OpenAPI spec
      run: npx @openapitools/openapi-generator-cli validate -i openapi.json
```

### 34.4 Storybook para Componentes Frontend

```yaml
# GitHub Actions — Build e deploy Storybook
storybook:
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/main'
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '22'
        cache: 'npm'
    - run: npm ci
    - run: npm run build-storybook

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v4
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: storybook-static
        destination_dir: storybook
```

```yaml
# GitLab — Storybook → GitLab Pages
pages:
  stage: deploy
  image: node:22-alpine
  script:
    - npm ci
    - npm run build-storybook -- -o public
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

---

## 35. Dependências Automáticas — Dependabot e Renovate

### 35.1 GitHub Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Maven / Gradle
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "06:00"
      timezone: "America/Sao_Paulo"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "java"
    reviewers:
      - "backend-team"
    groups:
      spring:
        patterns:
          - "org.springframework*"
        update-types:
          - "minor"
          - "patch"
      testing:
        patterns:
          - "org.junit*"
          - "org.mockito*"
          - "org.assertj*"

  # npm
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
    groups:
      react:
        patterns:
          - "react*"
          - "@types/react*"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "ci"

  # Docker
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "monthly"
```

### 35.2 Auto-merge de Dependabot PRs

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot Auto-Merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: metadata

      - name: Auto-merge patch and minor updates
        if: steps.metadata.outputs.update-type != 'version-update:semver-major'
        run: gh pr merge "$PR_URL" --auto --squash
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 35.3 Renovate (Alternativa Multi-Plataforma)

```json5
// renovate.json (GitHub ou GitLab)
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":automergeMinor",
    ":automergePatch",
    "group:allNonMajor"
  ],
  "schedule": ["before 7am on Monday"],
  "timezone": "America/Sao_Paulo",
  "labels": ["dependencies"],
  "packageRules": [
    {
      "matchPackagePatterns": ["org.springframework*"],
      "groupName": "Spring Framework",
      "automerge": false
    },
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true,
      "automergeType": "pr",
      "platformAutomerge": true
    },
    {
      "matchPackagePatterns": ["*"],
      "matchUpdateTypes": ["major"],
      "labels": ["dependencies", "breaking-change"],
      "automerge": false
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "automerge": true
  }
}
```

### 35.4 Comparação

| Aspecto | Dependabot | Renovate |
|---|---|---|
| **Plataformas** | GitHub apenas | GitHub, GitLab, Bitbucket, Azure DevOps |
| **Hosting** | Gerenciado pelo GitHub | Self-hosted ou Mend.io (SaaS) |
| **Agrupamento** | Groups (nativo) | Package Rules (mais flexível) |
| **Auto-merge** | Via workflow separado | Built-in (`automerge: true`) |
| **Dashboard** | Não possui | Issue de dashboard com overview |
| **Ecosystems** | Maven, npm, Docker, Actions, etc. | Todos do Dependabot + muitos outros |
| **Preço** | Gratuito | Open source / SaaS gratuito |
| **Configuração** | YAML simples | JSON5 poderoso (mais curva) |

---

## 36. Monorepo — Estratégias de Pipeline

### 36.1 GitHub Actions — Path Filters

```yaml
# Pipeline por diretório
name: Backend CI
on:
  push:
    paths:
      - 'backend/**'
      - '.github/workflows/backend-ci.yml'
  pull_request:
    paths:
      - 'backend/**'

jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend
    steps:
      - uses: actions/checkout@v4
      - run: ./mvnw build

# Detecção dinâmica de mudanças
name: Monorepo CI
on: [push, pull_request]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'

  backend:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cd backend && ./mvnw test

  frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cd frontend && npm test
```

### 36.2 GitLab CI/CD — Parent-Child para Monorepo

```yaml
# .gitlab-ci.yml (raiz)
stages:
  - triggers

backend:
  stage: triggers
  trigger:
    include: backend/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - backend/**/*

frontend:
  stage: triggers
  trigger:
    include: frontend/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - frontend/**/*

shared-lib:
  stage: triggers
  trigger:
    include: shared/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - shared/**/*
```

---

## 37. Self-hosted Runners e Otimização de Custos

### 37.1 GitHub — Self-hosted Runners

```bash
# Registrar runner (Linux)
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-x64-2.321.0.tar.gz
tar xzf actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/org/repo \
  --token AXXXXXXXXXXXXXXXXX \
  --labels self-hosted,linux,x64,gpu
./run.sh                                   # Foreground
sudo ./svc.sh install && sudo ./svc.sh start   # Como serviço
```

```yaml
# Uso no workflow
jobs:
  build:
    runs-on: [self-hosted, linux, gpu]     # Selecionar por labels
    steps:
      - uses: actions/checkout@v4
      - run: nvidia-smi                    # Acesso ao hardware GPU
```

### 37.2 GitLab — Self-managed Runners

```bash
# Registrar runner com executor Docker
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "GR134_XXXXX" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --tag-list "docker,linux" \
  --description "Docker Runner" \
  --run-untagged="true"
```

```toml
# /etc/gitlab-runner/config.toml
concurrent = 10
check_interval = 3

[[runners]]
  name = "Docker Runner"
  url = "https://gitlab.com/"
  token = "XXXX"
  executor = "docker"
  [runners.docker]
    image = "alpine:latest"
    privileged = true                      # Necessário para DinD
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 268435456                   # 256MB para testes com browser
  [runners.cache]
    Type = "s3"
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      BucketName = "gitlab-runner-cache"
      BucketLocation = "us-east-1"
```

### 37.3 Auto-scaling com Kubernetes

```yaml
# GitHub Actions — Actions Runner Controller (ARC)
# Helm chart que cria runners efêmeros como pods K8s
# helm install arc oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

apiVersion: actions.github.com/v1alpha1
kind: AutoscalingRunnerSet
metadata:
  name: arc-runners
spec:
  githubConfigUrl: "https://github.com/org"
  githubConfigSecret: github-secret
  minRunners: 1
  maxRunners: 20
  template:
    spec:
      containers:
        - name: runner
          image: ghcr.io/actions/actions-runner:latest
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2"
              memory: "4Gi"
```

```yaml
# GitLab — Runner com Kubernetes Executor
# Os runners criam pods efêmeros para cada job
[[runners]]
  name = "K8s Runner"
  executor = "kubernetes"
  [runners.kubernetes]
    namespace = "gitlab-runners"
    image = "alpine:latest"
    cpu_request = "500m"
    cpu_limit = "2"
    memory_request = "1Gi"
    memory_limit = "4Gi"
    service_cpu_request = "200m"
    service_memory_request = "256Mi"
    [runners.kubernetes.node_selector]
      "node.kubernetes.io/instance-type" = "m5.large"
```

### 37.4 Imagens Docker Customizadas para Runners

```dockerfile
# Dockerfile para runner com ferramentas pré-instaladas
FROM eclipse-temurin:21-jdk

RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl unzip docker.io kubectl helm \
    && rm -rf /var/lib/apt/lists/*

# Maven wrapper cache
COPY .mvn/ /opt/maven-cache/.mvn/
COPY mvnw pom.xml /opt/maven-cache/
RUN cd /opt/maven-cache && ./mvnw dependency:go-offline -q

# Node.js para builds fullstack
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs
```

```yaml
# Uso no GitLab — imagem customizada acelera builds
build:
  image: registry.example.com/ci-images/java-fullstack:21
  stage: build
  script:
    - ./mvnw -B package -DskipTests       # Dependências já em cache na imagem
```

### 37.5 Otimização de Custos

| Estratégia | Impacto | Implementação |
|---|---|---|
| **Path filters** | Alto | Só executar pipeline se arquivos relevantes mudam |
| **Cache agressivo** | Alto | Maven/Gradle/npm cache entre builds |
| **Skip CI** | Médio | `[skip ci]` em commits de docs ou scripts |
| **Jobs condicionais** | Médio | Matrix reduzida em PRs, completa em main |
| **Imagens leves** | Médio | Alpine, slim, imagens customizadas |
| **Timeout curto** | Baixo | Evitar jobs travados consumindo minutos |
| **Spot/Preemptible** | Alto | Runners em instâncias spot (até 90% mais barato) |
| **Fail fast** | Médio | `fail-fast: true` em matrices |
| **Merge queue** | Médio | Evitar runs duplicadas em branch atualizada |

```yaml
# GitHub Actions — Skip CI em commits de documentação
on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'

# GitHub Actions — Matrix reduzida em PR, completa em main
strategy:
  matrix:
    java: ${{ github.event_name == 'push' && fromJSON('["17","21"]') || fromJSON('["21"]') }}
    os: ${{ github.event_name == 'push' && fromJSON('["ubuntu-latest","windows-latest"]') || fromJSON('["ubuntu-latest"]') }}
```

```yaml
# GitLab — Interruptible para cancelar pipelines obsoletas
build:
  interruptible: true                      # Cancela se novo push na mesma branch
  stage: build
  script:
    - ./mvnw package

# GitLab — Resource Group para evitar deploys simultâneos
deploy:
  resource_group: production               # Serializa deploys
  script:
    - ./deploy.sh
```

### 37.6 Segurança de Self-hosted Runners

| Risco | Mitigação |
|---|---|
| **Código malicioso em PRs** | Nunca usar self-hosted em repos públicos; usar `pull_request_target` com cuidado |
| **Persistência entre jobs** | Usar runners efêmeros (containers, K8s pods) |
| **Escalação de privilégios** | Executar runner como usuário sem root; restringir Docker socket |
| **Secrets expostos** | Limitar secrets a environments protegidos |
| **Supply chain** | Pinnar actions por SHA; auditar dependências de terceiros |

---

## 38. Observabilidade e Notificações

### 38.1 Notificação Slack (GitHub Actions)

```yaml
notify:
  runs-on: ubuntu-latest
  needs: [deploy-production]
  if: always()
  steps:
    - uses: slackapi/slack-github-action@v2
      with:
        webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
        webhook-type: incoming-webhook
        payload: |
          {
            "text": "${{ needs.deploy-production.result == 'success' && '✅' || '❌' }} Deploy ${{ github.ref_name }} — ${{ needs.deploy-production.result }}",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*Deploy ${{ github.ref_name }}*\nStatus: ${{ needs.deploy-production.result }}\nCommit: <${{ github.server_url }}/${{ github.repository }}/commit/${{ github.sha }}|${{ github.sha }}>\nActor: ${{ github.actor }}"
                }
              }
            ]
          }
```

### 38.2 Notificação Slack (GitLab CI/CD)

```yaml
notify_slack:
  stage: .post
  image: curlimages/curl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - |
      STATUS=$([ "$CI_JOB_STATUS" = "success" ] && echo "✅" || echo "❌")
      curl -X POST "$SLACK_WEBHOOK_URL" \
        -H 'Content-type: application/json' \
        -d "{
          \"text\": \"$STATUS Pipeline $CI_PIPELINE_ID — $CI_COMMIT_REF_NAME\",
          \"blocks\": [{
            \"type\": \"section\",
            \"text\": {
              \"type\": \"mrkdwn\",
              \"text\": \"*Pipeline $CI_PIPELINE_ID*\nBranch: $CI_COMMIT_REF_NAME\nCommit: $CI_COMMIT_SHORT_SHA\nUser: $GITLAB_USER_LOGIN\nURL: $CI_PIPELINE_URL\"
            }
          }]
        }"
  when: always
```

### 38.3 Status Badges

```markdown
<!-- GitHub Actions -->
![CI](https://github.com/owner/repo/actions/workflows/ci.yml/badge.svg)
![CI](https://github.com/owner/repo/actions/workflows/ci.yml/badge.svg?branch=develop)

<!-- GitLab CI/CD -->
![pipeline](https://gitlab.com/group/project/badges/main/pipeline.svg)
![coverage](https://gitlab.com/group/project/badges/main/coverage.svg)
```

---

## 39. Boas Práticas e Checklist

### 39.1 Design de Pipeline

- **Fail fast** — executar linting e testes unitários antes de testes demorados
- **Paralelizar** — jobs independentes devem rodar em paralelo
- **Cache agressivo** — cachear dependências (Maven, npm, Gradle) para acelerar builds
- **Artefatos com expiração** — definir `retention-days` / `expire_in` para não consumir storage
- **Idempotência** — pipelines devem poder ser re-executadas sem efeitos colaterais

### 39.2 Segurança

- **OIDC** — usar federação de identidade em vez de credenciais estáticas para cloud
- **Least privilege** — `permissions` mínimas no GitHub Actions; variáveis `protected` no GitLab
- **Pin de versões** — usar SHA em actions (`uses: actions/checkout@<sha>`) em produção
- **Scan de secrets** — habilitar secret scanning (GitHub) ou Secret Detection (GitLab)
- **SAST/SCA** — incluir análise estática e de dependências no pipeline
- **Imagens confiáveis** — usar imagens oficiais e verificadas nos runners

### 39.3 Manutenibilidade

- **DRY** — usar Composite Actions / Reusable Workflows (GitHub) ou includes/extends (GitLab)
- **Documentar** — comentar decisões não óbvias no YAML
- **Versionamento** — versionar templates compartilhados com tags
- **Monitorar tempos** — acompanhar duração dos pipelines e otimizar gargalos
- **Testar pipelines** — usar `act` (GitHub Actions local) ou `gitlab-ci-local` para testar antes de push

### 39.4 Checklist de Pipeline

```
┌──────────────────────────────────────────────────────────────────────┐
│                   Checklist de Pipeline CI/CD                         │
│                                                                      │
│  CI:                                                                 │
│  □ Lint / formatação automática                                      │
│  □ Compilação / build                                                │
│  □ Testes unitários                                                  │
│  □ Testes de integração                                              │
│  □ Análise estática (SAST)                                           │
│  □ Verificação de dependências (SCA)                                 │
│  □ Cobertura de código                                               │
│  □ Cache de dependências configurado                                 │
│                                                                      │
│  CD:                                                                 │
│  □ Build de imagem Docker                                            │
│  □ Push para registry                                                │
│  □ Deploy automático em staging                                      │
│  □ Smoke tests pós-deploy                                            │
│  □ Aprovação manual para produção                                    │
│  □ Deploy em produção                                                │
│  □ Health check pós-deploy                                           │
│  □ Rollback automatizado em caso de falha                            │
│                                                                      │
│  Segurança:                                                          │
│  □ Secrets não expostos em logs                                      │
│  □ OIDC para credenciais de cloud                                    │
│  □ Permissões mínimas (least privilege)                              │
│  □ Scan de vulnerabilidades em imagens                               │
│  □ Secret scanning habilitado                                        │
│                                                                      │
│  Observabilidade:                                                    │
│  □ Notificações de falha (Slack, email, etc.)                        │
│  □ Badges de status no README                                        │
│  □ Métricas de duração de pipeline                                   │
│  □ Relatórios de teste integrados                                    │
└──────────────────────────────────────────────────────────────────────┘
```

### 39.5 Ferramentas para Teste Local de Pipelines

| Ferramenta | Plataforma | Descrição |
|---|---|---|
| **act** | GitHub Actions | Executa workflows localmente usando Docker |
| **gitlab-ci-local** | GitLab CI/CD | Executa `.gitlab-ci.yml` localmente |
| **Dagger** | Ambas | Pipeline como código (Go/Python/Node) portável entre CI/CDs |

```bash
# act — testar GitHub Actions localmente
brew install act
act -l                                     # Listar workflows
act push                                   # Simular push event
act pull_request -j test                   # Executar job específico

# gitlab-ci-local — testar GitLab CI/CD localmente
npm install -g gitlab-ci-local
gitlab-ci-local                            # Executar pipeline
gitlab-ci-local --job test                 # Executar job específico
```

---

## 40. GitHub — Gerenciamento de Projetos

### 40.1 Visão Geral das Ferramentas

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    GitHub — Gerenciamento de Projetos                         │
│                                                                              │
│  ┌────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐    │
│  │  Issues     │  │  Projects   │  │ Milestones  │  │  Pull Requests   │    │
│  │  (tarefas)  │  │  (boards)   │  │ (releases)  │  │  (code review)   │    │
│  └──────┬─────┘  └──────┬──────┘  └──────┬──────┘  └────────┬─────────┘    │
│         │               │               │                   │              │
│         └───────────┬───┴───────────────┴───────────────────┘              │
│                     │                                                       │
│              ┌──────▼──────┐                                                │
│              │  Automation │                                                │
│              │  (Actions,  │                                                │
│              │   closing   │                                                │
│              │   keywords) │                                                │
│              └─────────────┘                                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 40.2 Issues

Issues são a unidade básica de trabalho no GitHub — representam tarefas, bugs, features ou qualquer item rastreável.

#### Criação via Template (Markdown)

```markdown
<!-- .github/ISSUE_TEMPLATE/bug_report.md -->
---
name: Bug Report
about: Reporte um bug encontrado na aplicação
title: '[BUG] '
labels: bug, triage
assignees: ''
---

## Descrição do Bug
Descreva o que aconteceu de forma clara e concisa.

## Passos para Reproduzir
1. Vá para '...'
2. Clique em '...'
3. Veja o erro

## Comportamento Esperado
O que deveria acontecer.

## Screenshots
Se aplicável, adicione capturas de tela.

## Ambiente
- OS: [ex: Windows 11]
- Browser: [ex: Chrome 130]
- Versão: [ex: 2.1.0]
```

#### Criação via Template (YAML Form — mais estruturado)

```yaml
# .github/ISSUE_TEMPLATE/feature_request.yml
name: Feature Request
description: Sugira uma nova funcionalidade
title: "[FEATURE] "
labels: ["enhancement", "triage"]
assignees:
  - octocat
body:
  - type: markdown
    attributes:
      value: "Obrigado por sugerir uma melhoria!"

  - type: input
    id: summary
    attributes:
      label: Resumo
      description: Descreva a feature em uma frase
      placeholder: "Ex: Adicionar filtro por data no relatório"
    validations:
      required: true

  - type: textarea
    id: motivation
    attributes:
      label: Motivação
      description: Por que essa feature é importante?
    validations:
      required: true

  - type: dropdown
    id: priority
    attributes:
      label: Prioridade
      options:
        - Alta
        - Média
        - Baixa
    validations:
      required: true

  - type: checkboxes
    id: checklist
    attributes:
      label: Checklist
      options:
        - label: Já verifiquei que não existe issue duplicada
          required: true
        - label: Estou disponível para implementar
```

#### Configuração do Repositório

```yaml
# .github/ISSUE_TEMPLATE/config.yml
blank_issues_enabled: false               # Desabilita issues sem template
contact_links:
  - name: Dúvidas Gerais
    url: https://github.com/org/repo/discussions
    about: Use Discussions para perguntas
  - name: Documentação
    url: https://docs.example.com
    about: Consulte a documentação antes de abrir uma issue
```

### 40.3 Labels — Organização

Labels categorizam issues e PRs. Podem ser criadas manualmente ou via API/CLI.

| Label | Cor | Uso |
|---|---|---|
| `bug` | `#d73a4a` | Defeito no código |
| `enhancement` | `#a2eeef` | Nova funcionalidade |
| `documentation` | `#0075ca` | Melhoria na documentação |
| `good first issue` | `#7057ff` | Boa para iniciantes |
| `priority: high` | `#b60205` | Prioridade alta |
| `priority: medium` | `#fbca04` | Prioridade média |
| `priority: low` | `#0e8a16` | Prioridade baixa |
| `status: in progress` | `#1d76db` | Em andamento |
| `status: review` | `#5319e7` | Aguardando revisão |
| `backend` | `#f9d0c4` | Componente backend |
| `frontend` | `#bfdadc` | Componente frontend |

```bash
# Criar labels via GitHub CLI
gh label create "priority: high" --color "b60205" --description "Prioridade alta"
gh label create "priority: medium" --color "fbca04" --description "Prioridade média"
gh label create "status: in progress" --color "1d76db" --description "Em andamento"
```

### 40.4 Milestones

Milestones agrupam issues e PRs em torno de uma entrega (sprint, release, versão).

```bash
# Criar milestone via CLI
gh api repos/{owner}/{repo}/milestones -f title="v2.0.0" \
  -f description="Release com novo módulo de pagamentos" \
  -f due_on="2026-08-01T00:00:00Z"

# Associar issue a milestone
gh issue edit 42 --milestone "v2.0.0"

# Listar progresso
gh api repos/{owner}/{repo}/milestones --jq '.[] | "\(.title): \(.open_issues) abertas, \(.closed_issues) fechadas"'
```

### 40.5 GitHub Projects (Quadro Kanban)

GitHub Projects (v2) é o sistema de gerenciamento visual integrado ao GitHub. Suporta views de Board (Kanban), Table (planilha) e Roadmap (timeline).

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    GitHub Projects — Board View (Kanban)                      │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Backlog     │  │  In Progress │  │  In Review   │  │    Done      │    │
│  │              │  │              │  │              │  │              │    │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │    │
│  │ │ Issue #5 │ │  │ │ Issue #3 │ │  │ │ PR #12   │ │  │ │ Issue #1 │ │    │
│  │ │ API auth │ │  │ │ Login UI │ │  │ │ Fix cart │ │  │ │ Setup DB │ │    │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │    │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │              │  │ ┌──────────┐ │    │
│  │ │ Issue #7 │ │  │ │ Issue #4 │ │  │              │  │ │ Issue #2 │ │    │
│  │ │ Reports  │ │  │ │ Payments │ │  │              │  │ │ CI Setup │ │    │
│  │ └──────────┘ │  │ └──────────┘ │  │              │  │ └──────────┘ │    │
│  │ ┌──────────┐ │  │              │  │              │  │              │    │
│  │ │ Issue #9 │ │  │              │  │              │  │              │    │
│  │ │ i18n     │ │  │              │  │              │  │              │    │
│  │ └──────────┘ │  │              │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                              │
│  Views: [Board] [Table] [Roadmap]         Filters: label:backend status:open│
└──────────────────────────────────────────────────────────────────────────────┘
```

#### Tipos de View

| View | Descrição | Quando Usar |
|---|---|---|
| **Board** | Kanban com colunas arrastáveis | Acompanhamento diário do time |
| **Table** | Planilha com campos customizáveis | Triagem, filtros complexos, bulk edit |
| **Roadmap** | Timeline com datas de início/fim | Planejamento de releases, visão executiva |

#### Campos Customizados

Projects v2 suporta campos customizados tipados:

| Tipo | Exemplo | Uso |
|---|---|---|
| **Text** | "Notas técnicas" | Informações livres |
| **Number** | "Story Points", "Estimativa (h)" | Estimativas, métricas |
| **Date** | "Data Limite" | Deadlines, datas de entrega |
| **Single Select** | "Prioridade", "Sprint" | Categorização com opções fixas |
| **Iteration** | "Sprint 1", "Sprint 2" | Ciclos de desenvolvimento (2-4 semanas) |

#### Automações Nativas do GitHub Projects

O GitHub Projects possui **workflows built-in** que automatizam a movimentação de itens:

| Automação | Trigger | Ação |
|---|---|---|
| **Item added to project** | Issue/PR adicionada ao project | Define status como "Backlog" |
| **Item reopened** | Issue/PR reaberta | Move para "In Progress" |
| **Item closed** | Issue/PR fechada | Move para "Done" |
| **PR merged** | Pull Request mergeado | Move para "Done" |
| **Code review approved** | PR recebe aprovação | Move para "Approved" |
| **Auto-add to project** | Issue/PR criada com filtro | Adiciona automaticamente ao project |

Para habilitar: **Project > Settings > Workflows** (toggle on/off para cada automação).

#### Auto-add — Adicionar Issues Automaticamente

```
Project Settings > Workflows > Auto-add to project

Filtro: is:issue is:open label:backend
→ Toda issue com label "backend" é adicionada automaticamente ao board
```

### 40.6 Fechamento Automático de Issues via Commit e PR

O GitHub fecha issues automaticamente quando palavras-chave aparecem em mensagens de commit ou descrições de PR.

#### Palavras-chave (Closing Keywords)

```
Closes #42          Fecha a issue #42
closes #42          (case-insensitive)
close #42
Fixes #42           Fecha a issue #42
fixes #42
fix #42
Resolves #42        Fecha a issue #42
resolves #42
resolve #42
```

#### Múltiplas Issues

```
Fixes #42, closes #43, resolves #44
```

#### Em Commits

```bash
git commit -m "feat: adicionar autenticação JWT

Implements login flow with access and refresh tokens.

Closes #42"
```

Quando esse commit é mergeado na branch padrão (main), a issue #42 é fechada automaticamente.

#### Em Descrição de Pull Request

```markdown
## Descrição
Implementa o fluxo de autenticação JWT com access e refresh tokens.

## Issues Relacionadas
Closes #42
Fixes #38
Resolves #45
```

Quando o PR é mergeado, todas as issues referenciadas são fechadas.

#### Referência Cross-Repository

```
Fixes owner/other-repo#42
```

#### Comportamento

| Contexto | Quando Fecha |
|---|---|
| Commit na branch padrão (main) | Imediatamente |
| Commit em outra branch | Quando o branch é mergeado em main |
| Descrição do PR | Quando o PR é mergeado |
| Comentário do PR | **Não fecha** (apenas referencia) |

### 40.7 Pull Request Templates

```markdown
<!-- .github/pull_request_template.md -->
## Descrição
<!-- Descreva o que foi alterado e por quê -->

## Tipo de Mudança
- [ ] Bug fix
- [ ] Nova feature
- [ ] Breaking change
- [ ] Refactoring
- [ ] Documentação

## Issues Relacionadas
<!-- Use closing keywords: Closes #123, Fixes #456 -->
Closes #

## Checklist
- [ ] Testes adicionados/atualizados
- [ ] Documentação atualizada
- [ ] Sem warnings no build
- [ ] Code review solicitado

## Screenshots (se aplicável)
```

### 40.8 GitHub CLI — Gerenciamento Rápido

```bash
# ─── Issues ───
gh issue create --title "Bug no login" --label "bug,backend" --assignee "@me"
gh issue list --label "bug" --state open
gh issue close 42 --comment "Corrigido no PR #55"
gh issue edit 42 --add-label "priority: high" --milestone "v2.0"
gh issue view 42

# ─── Pull Requests ───
gh pr create --title "feat: autenticação JWT" --body "Closes #42"
gh pr list --state open --assignee "@me"
gh pr review 55 --approve
gh pr merge 55 --squash --delete-branch
gh pr checks 55                            # Ver status dos checks CI

# ─── Projects ───
gh project list
gh project item-list 1 --format json
gh project item-add 1 --owner "@me" --url "https://github.com/org/repo/issues/42"

# ─── Releases ───
gh release create v2.0.0 --title "v2.0.0" --notes "Release notes..." --target main
gh release create v2.0.0 --generate-notes  # Gera notas automaticamente dos PRs
```

### 40.9 Discussions

GitHub Discussions funciona como fórum integrado ao repositório — ideal para perguntas, ideias e RFCs.

| Categoria | Uso |
|---|---|
| **Announcements** | Comunicados do time (apenas maintainers criam) |
| **General** | Conversas gerais |
| **Ideas** | Sugestões de features (podem ser convertidas em Issues) |
| **Q&A** | Perguntas com respostas marcáveis como "aceita" |
| **Show and Tell** | Compartilhar projetos, demos |

---

## 41. GitLab — Gerenciamento de Projetos

### 41.1 Visão Geral

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    GitLab — Gerenciamento de Projetos                         │
│                                                                              │
│  ┌────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐    │
│  │  Issues     │  │   Boards    │  │ Milestones  │  │ Merge Requests   │    │
│  │  (tarefas)  │  │  (kanban)   │  │ (releases)  │  │  (code review)   │    │
│  └──────┬─────┘  └──────┬──────┘  └──────┬──────┘  └────────┬─────────┘    │
│         │               │               │                   │              │
│  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐           │              │
│  │  Epics      │ │  Iterations │ │  Labels     │           │              │
│  │  (Premium)  │ │  (Premium)  │ │  (scoped)   │           │              │
│  └─────────────┘ └─────────────┘ └─────────────┘           │              │
│                                                              │              │
│              ┌───────────────────────────────────────────────▼─┐            │
│              │  Automation                                      │            │
│              │  (closing patterns, Quick Actions, CI/CD hooks)  │            │
│              └──────────────────────────────────────────────────┘            │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 41.2 Issues

#### Template de Issue (Markdown)

```markdown
<!-- .gitlab/issue_templates/Bug.md -->
## Descrição do Bug

Descreva o comportamento inesperado.

## Passos para Reproduzir

1. Vá para '...'
2. Clique em '...'
3. Observe o erro

## Comportamento Esperado

O que deveria acontecer.

## Ambiente
- Versão da aplicação:
- Browser:
- OS:

## Logs / Screenshots

/label ~bug ~"priority::high"
/assign @developer
/milestone %"v2.0"
```

#### Template de Issue (YAML — GitLab 17+)

```yaml
# .gitlab/issue_templates/feature_request.yml
name: Feature Request
description: Sugira uma nova funcionalidade
title: "[FEATURE] "
quick_actions:
  - /label ~enhancement ~triage
body:
  - type: input
    id: summary
    attributes:
      label: Resumo
      placeholder: Descreva a feature
    validations:
      required: true
  - type: textarea
    id: details
    attributes:
      label: Detalhes
      description: Descreva a feature em detalhes
  - type: dropdown
    id: component
    attributes:
      label: Componente
      options:
        - Backend
        - Frontend
        - Infra
```

### 41.3 Labels — Scoped Labels

O GitLab suporta **scoped labels** (mutuamente exclusivos dentro do mesmo escopo):

```
priority::high      ← só uma priority:: pode estar ativa
priority::medium
priority::low

status::backlog     ← só um status:: pode estar ativo
status::doing
status::review
status::done

workflow::design
workflow::development
workflow::testing

component::backend
component::frontend
component::infra
```

Quando você aplica `priority::medium` a uma issue que já tem `priority::high`, o GitLab **remove automaticamente** o label anterior.

### 41.4 Issue Boards (Kanban)

Issue Boards no GitLab mapeiam listas do Kanban diretamente para **labels**. Mover um card entre colunas aplica/remove labels automaticamente.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                     GitLab Issue Board                                        │
│                                                                              │
│  Board: "Development"    Scope: milestone = v2.0                             │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Open        │  │  Doing       │  │  Review      │  │   Closed     │    │
│  │  (sem label)  │  │ ~status::    │  │ ~status::    │  │  (fechadas)  │    │
│  │              │  │   doing      │  │   review     │  │              │    │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │    │
│  │ │ #15      │ │  │ │ #12      │ │  │ │ #8       │ │  │ │ #3       │ │    │
│  │ │ API docs │ │  │ │ Login    │ │  │ │ Fix cart │ │  │ │ DB setup │ │    │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │    │
│  │ ┌──────────┐ │  │              │  │              │  │ ┌──────────┐ │    │
│  │ │ #18      │ │  │              │  │              │  │ │ #5       │ │    │
│  │ │ Cache    │ │  │              │  │              │  │ │ CI pipe  │ │    │
│  │ └──────────┘ │  │              │  │              │  │ └──────────┘ │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                              │
│  Arrastar #15 de "Open" para "Doing" → aplica ~status::doing               │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### Múltiplos Boards

Um projeto pode ter vários boards com escopos diferentes:

| Board | Escopo | Listas |
|---|---|---|
| **Development** | `milestone = v2.0` | Open → Doing → Review → Closed |
| **Backend** | `label = component::backend` | Backlog → In Progress → Done |
| **Bug Triage** | `label = bug` | New → Confirmed → Fix in Progress → Verified |

#### Configuração de Board

```
Settings do Board:
├── Scope
│   ├── Milestone: v2.0
│   ├── Labels: component::backend
│   └── Assignee: (todos)
├── Lists (colunas)
│   ├── Open (padrão, não editável)
│   ├── ~status::backlog
│   ├── ~status::doing
│   ├── ~status::review
│   └── Closed (padrão, não editável)
└── WIP Limits: máximo de cards por coluna
```

### 41.5 Milestones e Iterations

#### Milestones

Agrupam issues por release/entrega com data de vencimento:

```
Milestone: v2.0.0
├── Due date: 2026-08-01
├── Issues: 12 abertas, 28 fechadas (70% concluído)
├── Merge Requests: 5 abertos, 18 mergeados
└── Burndown Chart: ████████░░░░ (andamento visual)
```

#### Iterations (Premium)

Iterations são ciclos de tempo fixo (sprints) associados a um **Iteration Cadence**:

```
Iteration Cadence: "Sprints de 2 semanas"
├── Sprint 1: Jun 16 – Jun 27 (concluída)
├── Sprint 2: Jun 30 – Jul 11 (atual)    ← issues atribuídas automaticamente
├── Sprint 3: Jul 14 – Jul 25 (futura)
└── Sprint 4: Jul 28 – Aug 08 (futura)
```

### 41.6 Fechamento Automático de Issues via Commit e MR

#### Palavras-chave (Closing Patterns)

O GitLab usa os mesmos padrões do GitHub, mas é **configurável** pelo admin:

```
Closes #42              Fecha a issue #42
closes #42
close #42
Fixes #42
fixes #42
fix #42
Resolves #42
resolves #42
resolve #42
Implement #42
Implements #42
```

#### Em Commits

```bash
git commit -m "feat: adicionar endpoint de pagamentos

Closes #42, Fixes #38"
```

#### Em Descrição de Merge Request

```markdown
## Descrição
Implementa o endpoint REST para processamento de pagamentos.

Closes #42
Fixes #38

/milestone %"v2.0"
/label ~backend ~"priority::high"
/assign @dev1
```

#### Comportamento no GitLab

| Contexto | Quando Fecha |
|---|---|
| Commit na branch padrão | Imediatamente |
| Commit em outra branch | Quando o MR é mergeado em main |
| Descrição do MR | Quando o MR é mergeado |
| Comentário do MR | **Não fecha** (apenas referencia) |

#### Configuração Personalizada (Admin)

```
Admin > Settings > Repository > Default closing pattern

Padrão:
\b((?:[Cc]los(?:e[sd]?|ing)|\b[Ff]ix(?:e[sd]|ing)?|\b[Rr]esolv(?:e[sd]?|ing)|\b[Ii]mplement(?:s|ed|ing)?)(:?) +(?:(?:issues? +)?%{ID_PATTERN}(?:(?: *,? +and +| *## )(?:issues? +)?)*)+)
```

### 41.7 Quick Actions

Quick Actions são comandos de texto que executam ações em issues e MRs, usáveis em descrições e comentários:

#### Quick Actions para Issues

```
/assign @user1 @user2           Atribuir a usuários
/unassign @user1                Remover atribuição
/label ~bug ~"priority::high"   Adicionar labels
/unlabel ~bug                   Remover label
/milestone %"v2.0"              Definir milestone
/iteration *iteration:"Sprint 3"  Definir iteration
/due 2026-08-01                 Definir data limite
/weight 3                       Definir peso (estimativa)
/estimate 4h                    Definir estimativa de tempo
/spend 2h                       Registrar tempo gasto
/close                          Fechar issue
/reopen                         Reabrir issue
/relate #43                     Relacionar com outra issue
/blocks #44                     Marcar como bloqueadora
/move project/path              Mover para outro projeto
/confidential                   Marcar como confidencial
/todo                           Adicionar ao seu To-Do list
```

#### Quick Actions para Merge Requests

```
/approve                        Aprovar o MR
/merge                          Iniciar merge (se pipeline passar)
/draft                          Marcar como Draft
/ready                          Remover Draft
/target_branch develop          Mudar branch alvo
/reviewer @user1                Solicitar review
/copy_metadata #42              Copiar labels/milestone de outra issue
```

#### Exemplo Prático — Triagem Rápida

```markdown
<!-- Comentário em uma issue recém-aberta -->
Confirmado o bug. Vou priorizar para a próxima sprint.

/label ~bug ~"priority::high" ~"component::backend"
/milestone %"v2.0"
/assign @senior-dev
/iteration *iteration:"Sprint 3"
/weight 5
/due 2026-07-25
```

Um único comentário executa 6 ações de gerenciamento.

### 41.8 Merge Request Templates

```markdown
<!-- .gitlab/merge_request_templates/Default.md -->
## Descrição
<!-- O que foi alterado e por quê -->

## Issues Relacionadas
Closes #

## Tipo de Mudança
- [ ] Bug fix
- [ ] Nova feature
- [ ] Breaking change
- [ ] Refactoring

## Checklist
- [ ] Testes adicionados
- [ ] Pipeline verde
- [ ] Documentação atualizada
- [ ] Sem TODOs pendentes

## Screenshots / Evidências

/assign @me
/label ~"status::review"
```

### 41.9 Epics (Premium)

Epics agrupam issues relacionadas em um nível superior, permitindo hierarquia:

```
Epic: "Sistema de Pagamentos"
├── Issue #42: Endpoint de pagamento (backend)
├── Issue #43: Tela de checkout (frontend)
├── Issue #44: Integração gateway
├── Child Epic: "Relatórios Financeiros"
│   ├── Issue #50: Dashboard de receitas
│   └── Issue #51: Exportação PDF
└── Progress: ████████░░ 80%
```

### 41.10 GitLab CLI (glab)

```bash
# ─── Issues ───
glab issue create --title "Bug no login" --label "bug" --assignee "@me"
glab issue list --label "bug" --milestone "v2.0"
glab issue close 42 --comment "Corrigido no MR !55"
glab issue view 42

# ─── Merge Requests ───
glab mr create --title "feat: autenticação JWT" --description "Closes #42"
glab mr list --state opened --assignee "@me"
glab mr approve 55
glab mr merge 55 --squash --remove-source-branch
glab mr diff 55

# ─── CI/CD ───
glab ci status                             # Status da pipeline atual
glab ci view                               # Ver pipeline no terminal
glab ci run                                # Disparar pipeline
glab ci list                               # Listar pipelines recentes
```

---

## 42. Automações de Gerenciamento via CI/CD

### 42.1 GitHub Actions — Automação de Issues e Projects

#### Auto-label em PRs por Arquivos Alterados

```yaml
# .github/workflows/auto-label.yml
name: Auto Label PR

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```

```yaml
# .github/labeler.yml
backend:
  - changed-files:
      - any-glob-to-any-file: ['src/main/java/**', 'pom.xml']

frontend:
  - changed-files:
      - any-glob-to-any-file: ['src/main/resources/static/**', 'frontend/**']

documentation:
  - changed-files:
      - any-glob-to-any-file: ['**/*.md', 'docs/**']

ci:
  - changed-files:
      - any-glob-to-any-file: ['.github/**']

database:
  - changed-files:
      - any-glob-to-any-file: ['src/main/resources/db/**', '**/migration/**']
```

#### Mover Issue no Project ao Abrir PR

```yaml
# .github/workflows/pr-to-project.yml
name: Move Issue to In Review

on:
  pull_request:
    types: [opened, ready_for_review]

permissions:
  contents: read
  pull-requests: read
  repository-projects: write

jobs:
  update-project:
    runs-on: ubuntu-latest
    steps:
      - name: Move linked issues to "In Review"
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.PROJECT_TOKEN }}
          script: |
            const pr = context.payload.pull_request;
            const body = pr.body || '';

            // Extrair números de issues do body do PR
            const issueNumbers = [...body.matchAll(/(closes|fixes|resolves)\s+#(\d+)/gi)]
              .map(m => parseInt(m[2]));

            for (const issueNumber of issueNumbers) {
              console.log(`Linked issue: #${issueNumber}`);
              // Adicionar label para mover no board
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issueNumber,
                labels: ['status: in review']
              });
            }
```

#### Fechar Issues Stale Automaticamente

```yaml
# .github/workflows/stale.yml
name: Close Stale Issues

on:
  schedule:
    - cron: '0 6 * * 1'                   # Toda segunda às 6h

permissions:
  issues: write
  pull-requests: write

jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v9
        with:
          stale-issue-message: >
            Esta issue está inativa há 60 dias. Será fechada em 14 dias
            se não houver atividade. Adicione o label "keep-open" para manter.
          stale-pr-message: >
            Este PR está inativo há 30 dias. Será fechado em 7 dias
            se não houver atividade.
          stale-issue-label: 'stale'
          stale-pr-label: 'stale'
          days-before-issue-stale: 60
          days-before-pr-stale: 30
          days-before-issue-close: 14
          days-before-pr-close: 7
          exempt-issue-labels: 'keep-open,security,bug'
          exempt-pr-labels: 'keep-open,wip'
```

#### Criar Issue Automaticamente em Falha de Deploy

```yaml
# Dentro do workflow de deploy
  create-issue-on-failure:
    runs-on: ubuntu-latest
    needs: deploy-production
    if: failure()
    permissions:
      issues: write
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const title = `Deploy falhou: ${context.sha.substring(0, 7)}`;
            const body = `## Deploy Falhou

            - **Commit:** ${context.sha}
            - **Branch:** ${context.ref}
            - **Actor:** ${context.actor}
            - **Workflow Run:** ${context.serverUrl}/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}

            Investigar e corrigir urgentemente.

            /cc @${context.actor}`;

            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: title,
              body: body,
              labels: ['bug', 'priority: critical', 'deploy-failure'],
              assignees: [context.actor]
            });
```

#### Atribuir Reviewer Automaticamente

```yaml
# .github/workflows/auto-assign.yml
name: Auto Assign Reviewers

on:
  pull_request:
    types: [opened, ready_for_review]

permissions:
  pull-requests: write

jobs:
  assign:
    if: "!github.event.pull_request.draft"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const author = context.payload.pull_request.user.login;

            // Round-robin ou baseado em CODEOWNERS
            const reviewers = ['dev1', 'dev2', 'dev3']
              .filter(r => r !== author);

            // Atribuir 2 reviewers aleatórios
            const selected = reviewers
              .sort(() => Math.random() - 0.5)
              .slice(0, 2);

            await github.rest.pulls.requestReviewers({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.payload.pull_request.number,
              reviewers: selected
            });
```

#### CODEOWNERS — Review Automático por Área do Código

```
# .github/CODEOWNERS

# Owners globais (review em qualquer PR)
*                       @tech-lead

# Backend
src/main/java/          @backend-team
pom.xml                 @backend-team @devops

# Frontend
frontend/               @frontend-team
*.css                   @frontend-team @designer

# Infraestrutura
.github/                @devops
Dockerfile              @devops
docker-compose*.yml     @devops

# Documentação
*.md                    @tech-writer @tech-lead

# Segurança (sempre requer review)
**/security/**          @security-team
**/auth/**              @security-team @backend-team
```

### 42.2 GitLab CI/CD — Automação de Issues e Boards

#### Atualizar Issue via API no Pipeline

```yaml
# Mover issue para "Doing" quando pipeline inicia
update_issue_status:
  stage: .pre
  image: curlimages/curl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - |
      # Extrair issue number do branch (ex: feature/42-login → 42)
      ISSUE_NUM=$(echo "$CI_COMMIT_REF_NAME" | grep -oP '\d+' | head -1)
      if [ -n "$ISSUE_NUM" ]; then
        # Remover label antigo e adicionar novo
        curl --request PUT \
          --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
          "$CI_API_V4_URL/projects/$CI_PROJECT_ID/issues/$ISSUE_NUM" \
          --data-urlencode "remove_labels=status::backlog" \
          --data-urlencode "add_labels=status::doing"
        echo "Issue #$ISSUE_NUM movida para Doing"
      fi
  rules:
    - if: $CI_COMMIT_REF_NAME =~ /^feature\//
```

#### Mover para "Review" ao Criar MR

```yaml
# Mover issue para "Review" quando pipeline de MR roda
move_to_review:
  stage: .pre
  image: curlimages/curl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - |
      if [ -n "$CI_MERGE_REQUEST_IID" ]; then
        # Buscar issues relacionadas ao MR
        MR_DESC=$(curl -s --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
          "$CI_API_V4_URL/projects/$CI_PROJECT_ID/merge_requests/$CI_MERGE_REQUEST_IID" \
          | python3 -c "import sys,json; print(json.load(sys.stdin).get('description',''))")

        # Extrair issues do padrão "Closes #N"
        echo "$MR_DESC" | grep -oP '(?:Closes|Fixes|Resolves)\s+#\K\d+' | while read ISSUE_NUM; do
          curl --request PUT \
            --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
            "$CI_API_V4_URL/projects/$CI_PROJECT_ID/issues/$ISSUE_NUM" \
            --data-urlencode "remove_labels=status::doing" \
            --data-urlencode "add_labels=status::review"
          echo "Issue #$ISSUE_NUM movida para Review"
        done
      fi
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

#### Criar Issue em Falha de Deploy

```yaml
create_issue_on_failure:
  stage: .post
  image: curlimages/curl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - |
      curl --request POST \
        --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
        --header "Content-Type: application/json" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/issues" \
        --data "{
          \"title\": \"[DEPLOY FAILURE] Pipeline $CI_PIPELINE_ID falhou\",
          \"description\": \"## Deploy Falhou\n\n- **Pipeline:** $CI_PIPELINE_URL\n- **Commit:** $CI_COMMIT_SHORT_SHA\n- **Branch:** $CI_COMMIT_REF_NAME\n- **User:** $GITLAB_USER_LOGIN\n\nInvestigar urgentemente.\",
          \"labels\": \"bug,priority::critical,deploy-failure\",
          \"assignee_ids\": [$GITLAB_USER_ID]
        }"
  when: on_failure
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

#### Comentar na Issue com Resultado do Pipeline

```yaml
comment_pipeline_result:
  stage: .post
  image: curlimages/curl:latest
  variables:
    GIT_STRATEGY: none
  script:
    - |
      ISSUE_NUM=$(echo "$CI_COMMIT_REF_NAME" | grep -oP '\d+' | head -1)
      if [ -n "$ISSUE_NUM" ]; then
        STATUS_EMOJI="✅"
        STATUS_TEXT="passou"
        if [ "$CI_JOB_STATUS" != "success" ]; then
          STATUS_EMOJI="❌"
          STATUS_TEXT="falhou"
        fi

        curl --request POST \
          --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
          "$CI_API_V4_URL/projects/$CI_PROJECT_ID/issues/$ISSUE_NUM/notes" \
          --data-urlencode "body=$STATUS_EMOJI Pipeline $STATUS_TEXT: [$CI_PIPELINE_ID]($CI_PIPELINE_URL) (commit: $CI_COMMIT_SHORT_SHA)"
      fi
  when: always
```

### 42.3 Automações Comuns — Comparação

| Automação | GitHub | GitLab |
|---|---|---|
| Fechar issue no merge | Closing keywords (nativo) | Closing patterns (nativo) |
| Mover card no board | Projects Workflows (built-in) | Labels + Board (nativo ao arrastar) |
| Auto-mover via CI | GitHub Actions + API | CI/CD job + API |
| Auto-label por arquivos | `actions/labeler` | CI job com script |
| Fechar issues inativas | `actions/stale` | GitLab Triage Bot |
| Auto-assign reviewers | `actions/github-script` + CODEOWNERS | CODEOWNERS (built-in) |
| Issue em falha de deploy | `actions/github-script` | CI job `when: on_failure` |
| Notificar na issue | `actions/github-script` | CI job + API `/notes` |
| Bloquear merge sem review | Branch protection rules | Merge request approvals |
| Bloquear merge sem CI verde | Branch protection + status checks | Pipeline must succeed (settings) |

---

## 43. Integração Completa — Do Commit ao Board

### 43.1 Fluxo End-to-End (GitHub)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│              Fluxo Integrado: Issue → Board → Commit → Deploy                │
│                                                                              │
│  1. CRIAR ISSUE                                                              │
│     └─▶ Issue #42 criada com template                                       │
│         └─▶ Auto-add workflow adiciona ao Project Board (coluna "Backlog")   │
│                                                                              │
│  2. INICIAR TRABALHO                                                         │
│     └─▶ Developer cria branch: feature/42-jwt-auth                          │
│         └─▶ Developer move card para "In Progress" (ou automation)          │
│                                                                              │
│  3. COMMIT                                                                   │
│     └─▶ git commit -m "feat: add JWT auth - Closes #42"                     │
│         └─▶ Commit referencia issue #42                                     │
│                                                                              │
│  4. PULL REQUEST                                                             │
│     └─▶ PR criado com "Closes #42" na descrição                             │
│         ├─▶ auto-label aplica labels (backend, auth)                        │
│         ├─▶ CODEOWNERS solicita review de @security-team                    │
│         ├─▶ CI pipeline executa (lint, test, build)                         │
│         └─▶ Projects workflow move card para "In Review"                    │
│                                                                              │
│  5. REVIEW E MERGE                                                           │
│     └─▶ Reviewer aprova o PR                                                │
│         └─▶ PR é mergeado (squash)                                          │
│             ├─▶ Issue #42 é fechada automaticamente                         │
│             ├─▶ Projects workflow move card para "Done"                     │
│             └─▶ CD pipeline executa deploy                                  │
│                                                                              │
│  6. RELEASE                                                                  │
│     └─▶ Tag v2.1.0 criada                                                   │
│         └─▶ Release notes geradas automaticamente dos PRs mergeados         │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 43.2 Configuração Passo a Passo (GitHub)

**1. Criar o Project Board:**
- Ir em **Projects** > **New Project** > Escolher template "Board"
- Adicionar colunas: Backlog, In Progress, In Review, Done
- Em **Settings > Workflows**: habilitar "Item closed → Done" e "Pull request merged → Done"

**2. Configurar Auto-add:**
- **Settings > Workflows > Auto-add to project**
- Filtro: `is:issue is:open` (adiciona toda issue automaticamente)

**3. Configurar Branch Protection:**
- **Settings > Branches > Branch protection rules** para `main`
- Habilitar: Require pull request reviews, Require status checks, Require branches to be up to date

**4. Adicionar CODEOWNERS:**
- Criar `.github/CODEOWNERS` com mapeamento de áreas → reviewers

**5. Adicionar workflows de automação:**
- `.github/workflows/auto-label.yml` — labels automáticos por arquivos
- `.github/workflows/stale.yml` — fechar issues inativas
- `.github/workflows/ci.yml` — pipeline de CI/CD

**6. Convenção de branches:**
- `feature/42-descricao` — o número da issue no nome do branch

**7. Convenção de commits:**
- Usar Conventional Commits + closing keywords: `feat: descricao - Closes #42`

### 43.3 Fluxo End-to-End (GitLab)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│             Fluxo Integrado: Issue → Board → Commit → Deploy                 │
│                                                                              │
│  1. CRIAR ISSUE                                                              │
│     └─▶ Issue #42 criada com template                                       │
│         ├─▶ Quick Actions aplicam labels e milestone                        │
│         └─▶ Aparece no Board na coluna "Open"                               │
│                                                                              │
│  2. INICIAR TRABALHO                                                         │
│     └─▶ Clicar "Create merge request" na issue                              │
│         ├─▶ Branch 42-jwt-auth criado automaticamente                       │
│         ├─▶ MR draft criado com "Closes #42"                                │
│         └─▶ Arrastar card para "Doing" (aplica ~status::doing)              │
│              OU pipeline .pre job aplica label automaticamente               │
│                                                                              │
│  3. COMMIT E PUSH                                                            │
│     └─▶ git commit -m "feat: add JWT auth"                                  │
│         └─▶ Pipeline de MR executa (lint, test, build)                      │
│             └─▶ CI job comenta resultado na issue                           │
│                                                                              │
│  4. MERGE REQUEST                                                            │
│     └─▶ Remover draft (marcar como Ready)                                   │
│         ├─▶ CODEOWNERS solicita review                                      │
│         ├─▶ CI job move card para "Review" (~status::review)                │
│         └─▶ Pipeline completa roda                                          │
│                                                                              │
│  5. REVIEW E MERGE                                                           │
│     └─▶ Reviewer aprova o MR                                                │
│         └─▶ MR é mergeado (squash)                                          │
│             ├─▶ Issue #42 é fechada automaticamente                         │
│             ├─▶ Card move para "Closed" no board                            │
│             ├─▶ Branch deletado automaticamente                             │
│             └─▶ Pipeline de deploy dispara                                  │
│                                                                              │
│  6. RELEASE                                                                  │
│     └─▶ Tag v2.1.0 + Release criada                                        │
│         └─▶ Changelog gerado automaticamente                               │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 43.4 Configuração Passo a Passo (GitLab)

**1. Criar Labels de Status (scoped):**
```
status::backlog
status::doing
status::review
status::done
```

**2. Configurar Issue Board:**
- **Plan > Boards > New Board**
- Adicionar listas: `status::backlog`, `status::doing`, `status::review`
- Listas "Open" e "Closed" são automáticas

**3. Criar Templates:**
- `.gitlab/issue_templates/Bug.md` com Quick Actions embutidos
- `.gitlab/issue_templates/Feature.md`
- `.gitlab/merge_request_templates/Default.md`

**4. Configurar Merge Request Settings:**
- **Settings > Merge Requests**
- Merge method: Squash commits (recomendado)
- Merge checks: Pipelines must succeed, All threads must be resolved
- Merge options: Delete source branch when MR is accepted

**5. Configurar Protected Branches:**
- **Settings > Repository > Protected Branches**
- `main`: Allowed to merge = Maintainers, Allowed to push = No one

**6. Adicionar CI/CD para automação:**
```yaml
# Jobs que movem issues no board automaticamente
include:
  - local: '/.gitlab/ci/project-automation.yml'
```

**7. Convenção de branches:**
- Usar "Create merge request" direto da issue (GitLab cria `42-titulo-da-issue`)
- O GitLab já vincula MR ↔ Issue automaticamente

### 43.5 GitHub — Automatically Generated Release Notes

```yaml
# .github/release.yml
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - dependabot
  categories:
    - title: "🚀 Features"
      labels:
        - enhancement
        - feature
    - title: "🐛 Bug Fixes"
      labels:
        - bug
        - fix
    - title: "📖 Documentation"
      labels:
        - documentation
    - title: "🔧 Maintenance"
      labels:
        - chore
        - refactoring
        - ci
```

```bash
# Criar release com notas automáticas baseadas nos PRs mergeados
gh release create v2.1.0 --generate-notes --target main
```

O GitHub gera automaticamente as release notes agrupando PRs por label desde a última release.

### 43.6 GitLab — Changelog Automático

```yaml
# .gitlab-ci.yml
generate_changelog:
  stage: .post
  image: alpine:latest
  script:
    - |
      # Gerar changelog via API
      curl --request POST \
        --header "PRIVATE-TOKEN: $GITLAB_API_TOKEN" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/repository/changelog" \
        --data "version=$CI_COMMIT_TAG"
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
```

O GitLab gera changelog automaticamente a partir dos commits que seguem Conventional Commits, usando trailers `Changelog: added|fixed|changed|removed` nos commits.

### 43.7 Comparação de Gerenciamento de Projetos

| Funcionalidade | GitHub | GitLab |
|---|---|---|
| **Issues** | Templates MD + YAML Forms | Templates MD + YAML (17+) |
| **Quadro Kanban** | Projects (Board view) | Issue Boards |
| **Visão de Tabela** | Projects (Table view) | Issue list com filtros |
| **Roadmap/Timeline** | Projects (Roadmap view) | Roadmaps (Premium) |
| **Sprints** | Projects (Iteration field) | Iterations (Premium) |
| **Épicos** | Não nativo (usar labels/milestones) | Epics (Premium) |
| **Scoped Labels** | Não nativo | Sim (`scope::value`) |
| **WIP Limits** | Não nativo | Sim (no Board) |
| **Quick Actions** | Slash commands limitados | Extenso (/assign, /label, /due...) |
| **Time Tracking** | Não nativo | Built-in (/estimate, /spend) |
| **Peso/Story Points** | Custom fields no Project | /weight nativo |
| **Fechamento automático** | Closing keywords | Closing patterns |
| **Auto-move no board** | Project Workflows | Arrastar = aplicar label |
| **Review Apps** | Via Actions (manual) | Nativo (environments dinâmicos) |
| **Release Notes** | Auto-generated (PRs + labels) | Changelog API (Conventional Commits) |
| **CODEOWNERS** | `.github/CODEOWNERS` | `CODEOWNERS` (raiz ou docs/) |
| **Discussions/Fórum** | GitHub Discussions | Não nativo (usar issues) |
| **Preço p/ boards** | Gratuito | Gratuito (Epics/Iterations: Premium) |

---

## 44. GitLab CE On-Premise — Dimensionamento de Hardware

### 44.1 Visão Geral da Arquitetura

```
┌──────────────────────────────────────────────────────────────────────────────┐
│              Arquitetura GitLab CE On-Premise — 20 devs / 10 projetos        │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │  Servidor GitLab (All-in-One)                                        │    │
│  │                                                                      │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │    │
│  │  │  Nginx   │ │  Puma    │ │ Sidekiq  │ │PostgreSQL│ │  Redis   │  │    │
│  │  │ (proxy)  │ │ (Rails)  │ │ (workers)│ │  (DB)    │ │ (cache)  │  │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │    │
│  │  │  Gitaly  │ │  Minio   │ │ Registry │ │Prometheus│              │    │
│  │  │ (Git ops)│ │(obj stor)│ │ (Docker) │ │(metrics) │              │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘              │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │  Servidor(es) de Runners                                             │    │
│  │                                                                      │    │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │    │
│  │  │ Runner 1 (Docker)│  │ Runner 2 (Docker)│  │ Runner 3 (Shell) │   │    │
│  │  │ Jobs: build/test │  │ Jobs: build/test │  │ Jobs: deploy     │   │    │
│  │  │ Concurrent: 4    │  │ Concurrent: 4    │  │ Concurrent: 2    │   │    │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘   │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │  Storage                                                             │    │
│  │  NAS/SAN ou discos locais para repositórios, artefatos e backups    │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 44.2 Cenário de Referência

| Parâmetro | Valor |
|---|---|
| Desenvolvedores ativos | 20 |
| Projetos/repositórios | 10 |
| Pipelines simultâneas (pico) | 5–8 |
| Jobs concorrentes (pico) | 8–12 |
| Tamanho médio dos repositórios | 100 MB – 500 MB |
| Artefatos retidos | 30 dias |
| Container Registry | Sim (imagens Docker) |
| Frequência de builds | ~50–80 pipelines/dia |
| Horário de pico | 9h–12h e 14h–17h |

### 44.3 Estimativa de Hardware — Servidor GitLab

O GitLab CE roda todos os componentes (Rails, PostgreSQL, Redis, Gitaly, Nginx, Sidekiq, Prometheus) em um único servidor no modelo **all-in-one** (Omnibus).

#### Configuração Recomendada (20 usuários / 10 projetos)

| Recurso | Mínimo | Recomendado | Observação |
|---|---|---|---|
| **CPU** | 4 cores | **8 cores** | Puma + Sidekiq + PostgreSQL competem por CPU |
| **RAM** | 8 GB | **16 GB** | GitLab consome ~6-8 GB em idle; picos até 12 GB |
| **Disco OS + App** | 50 GB SSD | **100 GB SSD/NVMe** | Omnibus + PostgreSQL + Redis + logs |
| **Disco Repositórios** | 100 GB | **250 GB SSD** | `/var/opt/gitlab/git-data` — repositórios Git |
| **Disco Artefatos** | 100 GB | **500 GB HDD/SSD** | CI artifacts + Container Registry + uploads |
| **Disco Backups** | 200 GB | **500 GB HDD** | Backups diários com retenção de 7 dias |
| **Rede** | 1 Gbps | **1 Gbps** | Tráfego Git + Docker Registry + UI web |
| **SO** | Ubuntu 22.04/24.04 LTS | **Ubuntu 24.04 LTS** | Ou Debian 12 / RHEL 9 / Rocky 9 |

#### Fórmula de Referência GitLab

```
RAM recomendada = número_de_usuários × 0.5 GB + 4 GB (base)
                = 20 × 0.5 + 4 = 14 GB → arredondar para 16 GB

CPU recomendada = número_de_usuários / 5 + 2
                = 20 / 5 + 2 = 6 → arredondar para 8 cores
```

#### Dimensionamento de Disco Detalhado

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Estimativa de Disco                                                         │
│                                                                              │
│  Repositórios Git:                                                           │
│  10 repos × 300 MB médio = 3 GB (com crescimento em 2 anos: ~20 GB)        │
│                                                                              │
│  PostgreSQL:                                                                 │
│  Metadata de 10 projetos, 20 users, issues, MRs, CI logs = ~5-15 GB        │
│                                                                              │
│  CI/CD Artifacts (30 dias retenção):                                         │
│  70 pipelines/dia × 50 MB/pipeline × 30 dias = ~100 GB                     │
│                                                                              │
│  Container Registry:                                                         │
│  10 projetos × 5 tags × 200 MB/imagem = ~10 GB (com cleanup policy)        │
│                                                                              │
│  Backups (7 dias):                                                           │
│  ~30 GB/backup × 7 = ~210 GB                                                │
│                                                                              │
│  TOTAL estimado: ~350-500 GB (sem contar SO)                                │
│  Recomendação: 1 TB total (com margem de crescimento para 2 anos)           │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 44.4 Estimativa de Hardware — Servidores de Runners

Os runners devem ficar em servidores **separados** do GitLab para não competir por recursos.

#### Opção A — Dois Servidores de Runner (recomendado)

| Recurso | Runner 1 (Build/Test) | Runner 2 (Build/Test) |
|---|---|---|
| **CPU** | 8 cores | 8 cores |
| **RAM** | 16 GB | 16 GB |
| **Disco** | 200 GB SSD | 200 GB SSD |
| **Docker** | Sim | Sim |
| **Concurrent jobs** | 4 | 4 |
| **Tipo de jobs** | Build, test, SAST | Build, test, SAST |

**Total de jobs concorrentes: 8** (suficiente para 20 devs com pico de 5-8 pipelines simultâneas)

#### Opção B — Três Servidores de Runner (confortável)

| Recurso | Runner 1 | Runner 2 | Runner 3 |
|---|---|---|---|
| **CPU** | 8 cores | 8 cores | 4 cores |
| **RAM** | 16 GB | 16 GB | 8 GB |
| **Disco** | 200 GB SSD | 200 GB SSD | 100 GB SSD |
| **Concurrent** | 4 | 4 | 2 |
| **Uso** | Build/Test Java | Build/Test Node | Deploy/Docker Build |

**Total de jobs concorrentes: 10**

#### Cálculo de Recursos por Job

| Tipo de Job | CPU | RAM | Disco Temp | Duração Típica |
|---|---|---|---|---|
| Build Maven/Gradle | 2 cores | 2-4 GB | 1-2 GB | 2-5 min |
| Testes unitários | 1-2 cores | 2-3 GB | 500 MB | 2-8 min |
| Testes integração (Testcontainers) | 2 cores | 3-4 GB | 2-3 GB | 5-15 min |
| Build npm/Angular/React | 2 cores | 2-4 GB | 1-2 GB | 2-5 min |
| Docker build + push | 2 cores | 2-3 GB | 2-5 GB | 3-8 min |
| SAST/SCA scan | 1-2 cores | 2-3 GB | 500 MB | 2-5 min |
| Deploy (kubectl) | 0.5 core | 512 MB | 100 MB | 30s-2 min |

```
Recurso por runner = concurrent_jobs × recursos_por_job
Runner com 4 jobs concorrentes:
  CPU = 4 × 2 cores = 8 cores
  RAM = 4 × 3.5 GB  = 14 GB → 16 GB
  Disco = Docker images cache + temp artifacts = 150-200 GB SSD
```

### 44.5 Resumo do Hardware Total

#### Configuração Recomendada (3 servidores)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Hardware Total — 20 desenvolvedores / 10 projetos                           │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────┐          │
│  │  Servidor 1: GitLab CE (All-in-One)                            │          │
│  │  CPU: 8 cores    RAM: 16 GB    Disco: 1 TB (SSD+HDD)         │          │
│  └────────────────────────────────────────────────────────────────┘          │
│  ┌────────────────────────────────────────────────────────────────┐          │
│  │  Servidor 2: Runner Docker (Build/Test)                        │          │
│  │  CPU: 8 cores    RAM: 16 GB    Disco: 200 GB SSD              │          │
│  └────────────────────────────────────────────────────────────────┘          │
│  ┌────────────────────────────────────────────────────────────────┐          │
│  │  Servidor 3: Runner Docker (Build/Test)                        │          │
│  │  CPU: 8 cores    RAM: 16 GB    Disco: 200 GB SSD              │          │
│  └────────────────────────────────────────────────────────────────┘          │
│                                                                              │
│  TOTAL: 24 cores | 48 GB RAM | ~1.4 TB disco                               │
│  Jobs concorrentes: 8 (escalável adicionando runners)                       │
│                                                                              │
│  Alternativa com VMs:                                                        │
│  Host físico: 32 cores / 64 GB RAM / 2 TB SSD                              │
│  3 VMs (GitLab + 2 Runners) com alocação conforme acima                     │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### Configuração Mínima Viável (2 servidores)

| Servidor | CPU | RAM | Disco | Função |
|---|---|---|---|---|
| GitLab CE | 8 cores | 16 GB | 500 GB SSD + 500 GB HDD | App + DB + Registry + Backups |
| Runner | 8 cores | 16 GB | 200 GB SSD | 4-6 jobs concorrentes |

**Total mínimo: 16 cores | 32 GB RAM | 1.2 TB**

#### Configuração com Margem de Crescimento (até 40 devs)

| Servidor | CPU | RAM | Disco | Função |
|---|---|---|---|---|
| GitLab CE | 16 cores | 32 GB | 500 GB NVMe + 1 TB HDD | App + DB + Registry |
| Runner 1 | 16 cores | 32 GB | 500 GB SSD | 6-8 jobs concorrentes |
| Runner 2 | 8 cores | 16 GB | 200 GB SSD | 4 jobs concorrentes |
| Backup/NAS | — | — | 2 TB HDD | Backups offsite |

### 44.6 Requisitos de Rede

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Rede                                                                        │
│                                                                              │
│  ┌──────────┐   1 Gbps    ┌────────────┐   1 Gbps    ┌──────────────┐      │
│  │  Devs    │────────────▶│  GitLab    │────────────▶│   Runners    │      │
│  │  (LAN)   │◀────────────│  Server   │◀────────────│   Server(s)  │      │
│  └──────────┘              └─────┬──────┘              └──────────────┘      │
│                                  │                                           │
│                            ┌─────▼──────┐                                    │
│                            │  Internet   │  (HTTPS, SMTP, package downloads) │
│                            └────────────┘                                    │
│                                                                              │
│  Portas necessárias:                                                         │
│  ├── 80/443    HTTP/HTTPS (UI + Git HTTPS + Registry)                       │
│  ├── 22        SSH (Git SSH)                                                │
│  ├── 5050      Container Registry (se porta separada)                       │
│  ├── 9090      Prometheus (interno, opcional)                               │
│  └── Runner → GitLab: HTTPS (443) outbound only (runner inicia conexão)    │
│                                                                              │
│  DNS:                                                                        │
│  ├── gitlab.empresa.com       → IP do servidor GitLab                       │
│  ├── registry.empresa.com    → IP do servidor GitLab (ou mesmo domínio)    │
│  └── pages.empresa.com       → IP do servidor GitLab (se usar Pages)       │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 44.7 Considerações Adicionais

| Aspecto | Recomendação |
|---|---|
| **Virtualização** | VMware, Proxmox, KVM, Hyper-V — todos suportados |
| **SSD vs HDD** | SSD/NVMe para PostgreSQL, Git e OS. HDD aceitável para backups e artefatos antigos |
| **RAID** | RAID 1 (mirror) para OS/DB. RAID 5/10 para storage de dados |
| **UPS** | Recomendado para o servidor GitLab (contém o banco de dados) |
| **Monitoramento** | Prometheus + Grafana (incluídos no Omnibus) ou Zabbix/Nagios externo |
| **Backup offsite** | Rsync ou S3 compatible (MinIO) para backup em local separado |
| **Certificado TLS** | Let's Encrypt (automático pelo Omnibus) ou certificado corporativo |
| **LDAP/AD** | GitLab CE suporta integração LDAP para autenticação centralizada |
| **Email/SMTP** | Configurar SMTP para notificações (Gmail, SendGrid, servidor corporativo) |

---

## 45. GitLab CE On-Premise — Roteiro de Instalação

### 45.1 Pré-requisitos

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Checklist Pré-Instalação                                                    │
│                                                                              │
│  Servidor GitLab:                                                            │
│  □ Ubuntu 24.04 LTS instalado (minimal server)                              │
│  □ 8 cores / 16 GB RAM / discos conforme dimensionamento                    │
│  □ IP fixo configurado                                                       │
│  □ Hostname definido (ex: gitlab.empresa.com)                               │
│  □ DNS apontando para o servidor (A record)                                 │
│  □ Portas 22, 80, 443 liberadas no firewall                                │
│  □ Acesso SSH como root ou usuário com sudo                                 │
│  □ Acesso à internet (para downloads de pacotes)                            │
│                                                                              │
│  Servidor(es) Runner:                                                        │
│  □ Ubuntu 24.04 LTS instalado                                               │
│  □ 8 cores / 16 GB RAM / 200 GB SSD                                        │
│  □ Docker Engine instalado                                                   │
│  □ Conectividade HTTPS com o servidor GitLab                                │
│                                                                              │
│  Dados de configuração:                                                      │
│  □ Domínio/hostname do GitLab                                               │
│  □ Configuração SMTP (servidor, porta, credenciais)                         │
│  □ Certificado TLS (ou usar Let's Encrypt)                                  │
│  □ Credenciais LDAP/AD (se aplicável)                                       │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 45.2 Etapa 1 — Preparação do Servidor GitLab

```bash
# ─── Atualizar o sistema ───
sudo apt update && sudo apt upgrade -y

# ─── Instalar dependências ───
sudo apt install -y curl openssh-server ca-certificates tzdata perl postfix

# Na tela do Postfix, selecionar "Internet Site" e informar o domínio (ex: empresa.com)

# ─── Configurar firewall (UFW) ───
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw enable

# ─── Configurar hostname ───
sudo hostnamectl set-hostname gitlab.empresa.com
echo "192.168.1.100 gitlab.empresa.com" | sudo tee -a /etc/hosts

# ─── Configurar partições / mount points (se usar discos separados) ───
# Exemplo com discos separados:
# /dev/sdb → /var/opt/gitlab/git-data         (repositórios Git)
# /dev/sdc → /var/opt/gitlab/backups           (backups)
# /dev/sdd → /var/opt/gitlab/shared/artifacts  (CI artifacts)

sudo mkfs.ext4 /dev/sdb
sudo mkdir -p /var/opt/gitlab/git-data
sudo mount /dev/sdb /var/opt/gitlab/git-data

# Adicionar ao /etc/fstab para mount automático
echo "/dev/sdb /var/opt/gitlab/git-data ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

### 45.3 Etapa 2 — Instalar GitLab CE (Omnibus)

```bash
# ─── Adicionar repositório oficial do GitLab CE ───
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

# ─── Instalar o GitLab CE ───
# Substituir pela URL real do servidor (HTTPS se tiver certificado)
sudo EXTERNAL_URL="https://gitlab.empresa.com" apt install gitlab-ce

# A instalação configura automaticamente:
# - Nginx (reverse proxy)
# - PostgreSQL 14+
# - Redis
# - Puma (application server)
# - Sidekiq (background jobs)
# - Gitaly (Git operations)
# - Prometheus + Grafana (monitoramento)
# - Let's Encrypt (se EXTERNAL_URL usar HTTPS e DNS estiver configurado)

# ─── Verificar status da instalação ───
sudo gitlab-ctl status

# ─── Obter a senha inicial do root ───
sudo cat /etc/gitlab/initial_root_password

# A senha expira em 24h — alterá-la imediatamente no primeiro login
# Acessar: https://gitlab.empresa.com
# Usuário: root
# Senha: (conteúdo do arquivo acima)
```

### 45.4 Etapa 3 — Configuração do GitLab (`gitlab.rb`)

```bash
sudo nano /etc/gitlab/gitlab.rb
```

```ruby
# /etc/gitlab/gitlab.rb — Configurações principais

# ─── URL externa ───
external_url 'https://gitlab.empresa.com'

# ─── Let's Encrypt (automático) ───
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['admin@empresa.com']
letsencrypt['auto_renew'] = true
letsencrypt['auto_renew_hour'] = 3
letsencrypt['auto_renew_day_of_month'] = "*/7"

# ─── Ou certificado próprio (desabilitar Let's Encrypt) ───
# letsencrypt['enable'] = false
# nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.empresa.com.crt"
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.empresa.com.key"

# ─── Configuração de Email (SMTP) ───
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "gitlab@empresa.com"
gitlab_rails['smtp_password'] = "app-password-aqui"
gitlab_rails['smtp_domain'] = "empresa.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['gitlab_email_from'] = 'gitlab@empresa.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@empresa.com'

# ─── Configuração de Email (SMTP corporativo / Outlook 365) ───
# gitlab_rails['smtp_address'] = "smtp.office365.com"
# gitlab_rails['smtp_port'] = 587
# gitlab_rails['smtp_user_name'] = "gitlab@empresa.com"
# gitlab_rails['smtp_password'] = "senha"
# gitlab_rails['smtp_authentication'] = "login"
# gitlab_rails['smtp_enable_starttls_auto'] = true

# ─── Fuso horário ───
gitlab_rails['time_zone'] = 'America/Sao_Paulo'

# ─── Repositórios Git — diretório customizado ───
git_data_dirs({
  "default" => {
    "path" => "/var/opt/gitlab/git-data"
  }
})

# ─── Container Registry ───
registry_external_url 'https://registry.empresa.com'
# Se usar o mesmo domínio (porta diferente):
# registry_external_url 'https://gitlab.empresa.com:5050'

# ─── Backups ───
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_keep_time'] = 604800  # 7 dias em segundos

# ─── LDAP / Active Directory ───
# gitlab_rails['ldap_enabled'] = true
# gitlab_rails['ldap_servers'] = {
#   'main' => {
#     'label' => 'Empresa AD',
#     'host' => 'ad.empresa.com',
#     'port' => 636,
#     'uid' => 'sAMAccountName',
#     'encryption' => 'simple_tls',
#     'bind_dn' => 'CN=GitLab,OU=Services,DC=empresa,DC=com',
#     'password' => 'senha-do-bind',
#     'active_directory' => true,
#     'base' => 'OU=Users,DC=empresa,DC=com',
#     'user_filter' => '(memberOf=CN=GitLab-Users,OU=Groups,DC=empresa,DC=com)'
#   }
# }

# ─── Tuning para 20 usuários ───
# Puma (application server)
puma['worker_processes'] = 4               # Regra: CPU cores - 1 (max 12)
puma['min_threads'] = 4
puma['max_threads'] = 4

# Sidekiq (background jobs)
sidekiq['max_concurrency'] = 20

# PostgreSQL
postgresql['shared_buffers'] = "4GB"       # 25% da RAM
postgresql['work_mem'] = "64MB"
postgresql['effective_cache_size'] = "8GB"  # 50% da RAM

# Gitaly
gitaly['configuration'] = {
  concurrency: [
    { rpc: "/gitaly.SmartHTTPService/PostReceivePack", max_per_repo: 20 },
  ],
}

# Prometheus e Grafana (monitoramento built-in)
prometheus_monitoring['enable'] = true
grafana['enable'] = true
grafana['admin_password'] = 'senha-grafana-segura'

# ─── Limitar registro de novos usuários ───
gitlab_rails['gitlab_signup_enabled'] = false
```

```bash
# ─── Aplicar configurações ───
sudo gitlab-ctl reconfigure

# ─── Verificar se tudo está rodando ───
sudo gitlab-ctl status

# ─── Testar envio de email ───
sudo gitlab-rails console
# Dentro do console:
# Notify.test_email('admin@empresa.com', 'Test', 'GitLab SMTP Test').deliver_now
```

### 45.5 Etapa 4 — Configuração Pós-Instalação (UI)

Acessar `https://gitlab.empresa.com` como **root** e configurar:

```
1. Alterar senha do root
   └─▶ Perfil > Password > trocar imediatamente

2. Admin Area > Settings > General
   ├─▶ Visibility: Default project visibility = "Private"
   ├─▶ Sign-up: Desabilitar "Sign-up enabled"
   ├─▶ Account and limit: Default projects limit = 20
   └─▶ Account and limit: Max attachment size = 50 MB

3. Admin Area > Settings > CI/CD
   ├─▶ Default artifacts expiration = "30 days"
   ├─▶ Max artifacts size = 500 MB
   └─▶ Default timeout = 60 minutes

4. Admin Area > Settings > Repository
   ├─▶ Default branch name = "main"
   └─▶ Protected branches: main → Allowed to push: No one, Allowed to merge: Maintainers

5. Admin Area > Settings > Network
   ├─▶ Outbound requests: Allow local requests from webhooks/services
   └─▶ Import sources: desabilitar os não utilizados

6. Criar grupos e projetos
   ├─▶ Grupo: "empresa" (ou departamento)
   │   ├─▶ Subgrupo: "backend"
   │   ├─▶ Subgrupo: "frontend"
   │   └─▶ Subgrupo: "infra"
   └─▶ Criar projetos dentro dos subgrupos

7. Criar usuários
   └─▶ Admin Area > Users > New User (ou configurar LDAP)
       ├─▶ Role: Developer (maioria) ou Maintainer (tech leads)
       └─▶ Adicionar aos grupos correspondentes
```

### 45.6 Etapa 5 — Instalar e Registrar Runners

#### No servidor de Runner:

```bash
# ─── Instalar Docker Engine ───
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verificar instalação
sudo docker run hello-world

# ─── Instalar GitLab Runner ───
curl -fsSL https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt install -y gitlab-runner

# Adicionar gitlab-runner ao grupo docker
sudo usermod -aG docker gitlab-runner
```

#### Registrar Runner com Authentication Token (GitLab 16+):

```bash
# Obter o token em:
# GitLab > Admin Area > CI/CD > Runners > New instance runner
# Ou: Grupo > Settings > CI/CD > Runners > New group runner
# Ou: Projeto > Settings > CI/CD > Runners > New project runner

# ─── Registrar Runner Docker (para builds) ───
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.empresa.com" \
  --token "glrt-XXXXXXXXXXXXX" \
  --executor "docker" \
  --docker-image "eclipse-temurin:21-jdk" \
  --description "Docker Runner - Build/Test" \
  --tag-list "docker,build,test" \
  --run-untagged="true" \
  --docker-privileged="true" \
  --docker-volumes "/cache" \
  --docker-shm-size="268435456"

# ─── Registrar Runner Shell (para deploy com kubectl) ───
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.empresa.com" \
  --token "glrt-YYYYYYYYYYYYY" \
  --executor "shell" \
  --description "Shell Runner - Deploy" \
  --tag-list "shell,deploy"
```

#### Configurar Runner (`config.toml`):

```bash
sudo nano /etc/gitlab-runner/config.toml
```

```toml
# /etc/gitlab-runner/config.toml
concurrent = 4                             # Jobs simultâneos neste runner
check_interval = 3                         # Segundos entre verificações de jobs

[session_server]
  session_timeout = 1800

[[runners]]
  name = "Docker Runner - Build/Test"
  url = "https://gitlab.empresa.com"
  token = "glrt-XXXXXXXXXXXXX"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "eclipse-temurin:21-jdk"
    privileged = true                      # Necessário para Docker-in-Docker
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 268435456                   # 256 MB (necessário para testes com Chrome)
    pull_policy = ["if-not-present"]       # Evitar pull desnecessário
    # Limitar recursos por container
    cpus = "2"
    memory = "4g"
  [runners.cache]
    Type = "local"                         # Ou "s3" para cache distribuído
    Path = "/cache"
    Shared = true
    [runners.cache.local]
      path = "/cache"

[[runners]]
  name = "Shell Runner - Deploy"
  url = "https://gitlab.empresa.com"
  token = "glrt-YYYYYYYYYYYYY"
  executor = "shell"
  [runners.custom_build_dir]
    enabled = true
```

```bash
# ─── Reiniciar o runner ───
sudo gitlab-runner restart

# ─── Verificar status ───
sudo gitlab-runner status
sudo gitlab-runner verify

# ─── Ver runners registrados ───
sudo gitlab-runner list
```

### 45.7 Etapa 6 — Configurar Backups Automatizados

```bash
# ─── Criar backup manualmente (teste) ───
sudo gitlab-backup create

# O backup é salvo em /var/opt/gitlab/backups/
# Formato: <timestamp>_<version>_gitlab_backup.tar

# ─── Backup dos arquivos de configuração (SEPARADO!) ───
sudo cp /etc/gitlab/gitlab.rb /var/opt/gitlab/backups/
sudo cp /etc/gitlab/gitlab-secrets.json /var/opt/gitlab/backups/
```

```bash
# ─── Agendar backup diário via cron ───
sudo crontab -e

# Adicionar (backup às 2h da manhã):
0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1 SKIP=artifacts 2>&1 | tee -a /var/log/gitlab-backup.log

# SKIP=artifacts: exclui artefatos CI (grandes, com expiração própria)
# Para backup completo, remover SKIP=artifacts
```

```bash
# ─── Script de backup completo (configs + dados + limpeza) ───
# /usr/local/bin/gitlab-full-backup.sh
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/var/opt/gitlab/backups"
CONFIG_BACKUP_DIR="/var/opt/gitlab/backups/config"
RETENTION_DAYS=7
LOG="/var/log/gitlab-backup.log"

echo "$(date): Iniciando backup GitLab" >> "$LOG"

# Backup dos dados
/opt/gitlab/bin/gitlab-backup create CRON=1 >> "$LOG" 2>&1

# Backup das configurações
mkdir -p "$CONFIG_BACKUP_DIR"
cp /etc/gitlab/gitlab.rb "$CONFIG_BACKUP_DIR/gitlab.rb.$(date +%Y%m%d)"
cp /etc/gitlab/gitlab-secrets.json "$CONFIG_BACKUP_DIR/gitlab-secrets.json.$(date +%Y%m%d)"

# Limpar backups antigos
find "$BACKUP_DIR" -name "*.tar" -mtime +$RETENTION_DAYS -delete
find "$CONFIG_BACKUP_DIR" -mtime +$RETENTION_DAYS -delete

# Copiar para storage remoto (opcional)
# rsync -avz "$BACKUP_DIR/" backup-server:/backups/gitlab/

echo "$(date): Backup concluído" >> "$LOG"
```

```bash
sudo chmod +x /usr/local/bin/gitlab-full-backup.sh

# Agendar no cron
sudo crontab -e
# 0 2 * * * /usr/local/bin/gitlab-full-backup.sh
```

### 45.8 Etapa 7 — Restauração de Backup (Disaster Recovery)

```bash
# ─── Procedimento de restauração ───

# 1. Instalar GitLab CE na mesma versão do backup
sudo EXTERNAL_URL="https://gitlab.empresa.com" apt install gitlab-ce=<VERSAO>

# 2. Restaurar configurações
sudo cp gitlab.rb.backup /etc/gitlab/gitlab.rb
sudo cp gitlab-secrets.json.backup /etc/gitlab/gitlab-secrets.json
sudo gitlab-ctl reconfigure

# 3. Parar processos que escrevem no banco
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-ctl status                    # Verificar que PostgreSQL está rodando

# 4. Restaurar backup (substituir TIMESTAMP pelo nome do arquivo)
sudo gitlab-backup restore BACKUP=<TIMESTAMP_VERSION>
# Ex: sudo gitlab-backup restore BACKUP=1719523200_2024_06_28_17.0.0

# 5. Reiniciar GitLab
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart

# 6. Verificar integridade
sudo gitlab-rake gitlab:check SANITIZE=true
sudo gitlab-rake gitlab:doctor:secrets
```

### 45.9 Etapa 8 — Monitoramento

#### Prometheus + Grafana (built-in)

O Omnibus inclui Prometheus e Grafana pré-configurados:

```
Prometheus: https://gitlab.empresa.com/-/metrics
Grafana:    https://gitlab.empresa.com/-/grafana   (admin / senha do gitlab.rb)

Dashboards pré-configurados:
├── GitLab Omnibus Overview
├── GitLab Sidekiq
├── GitLab PostgreSQL
├── GitLab Redis
├── GitLab Puma
├── GitLab Gitaly
└── GitLab Runner
```

#### Alertas e Health Check

```bash
# ─── Health check endpoint (sem autenticação) ───
curl -s https://gitlab.empresa.com/-/health       # "GitLab OK"
curl -s https://gitlab.empresa.com/-/readiness     # JSON detalhado
curl -s https://gitlab.empresa.com/-/liveness      # Liveness probe

# ─── Script de monitoramento externo ───
# /usr/local/bin/gitlab-health-check.sh
#!/bin/bash
STATUS=$(curl -sf -o /dev/null -w '%{http_code}' https://gitlab.empresa.com/-/health)
if [ "$STATUS" != "200" ]; then
  echo "GitLab DOWN! HTTP $STATUS" | mail -s "ALERTA: GitLab" admin@empresa.com
fi
```

```bash
# Agendar health check a cada 5 minutos
echo "*/5 * * * * /usr/local/bin/gitlab-health-check.sh" | sudo crontab -
```

### 45.10 Etapa 9 — Manutenção Periódica

```bash
# ─── Atualizar GitLab CE ───
# Sempre ler release notes antes de atualizar!
# Seguir upgrade path: https://gitlab-com.gitlab.io/cs-tools/gitlab-cs-tools/what-is-new/upgrade-path/

# 1. Fazer backup antes
sudo gitlab-backup create

# 2. Atualizar pacote
sudo apt update
sudo apt install gitlab-ce

# 3. O reconfigure é executado automaticamente
# Verificar status
sudo gitlab-ctl status
sudo gitlab-rake gitlab:check SANITIZE=true
```

```bash
# ─── Limpeza periódica (agendar mensalmente) ───

# Limpar artefatos expirados
sudo gitlab-rake gitlab:cleanup:orphan_job_artifact_files

# Limpar imagens do Container Registry
# No gitlab.rb, habilitar:
# gitlab_rails['registry_gc_enabled'] = true
# E executar:
sudo gitlab-ctl registry-garbage-collect

# Limpar LFS objects não referenciados
sudo gitlab-rake gitlab:cleanup:orphan_lfs_files

# Vacuum do PostgreSQL (automático, mas pode forçar)
sudo gitlab-psql -c "VACUUM ANALYZE;"

# Verificar uso de disco
sudo du -sh /var/opt/gitlab/git-data/
sudo du -sh /var/opt/gitlab/backups/
sudo du -sh /var/opt/gitlab/gitlab-rails/shared/artifacts/
sudo du -sh /var/opt/gitlab/gitlab-rails/shared/registry/
```

### 45.11 Etapa 10 — Checklist de Validação Final

```
┌──────────────────────────────────────────────────────────────────────────────┐
│              Checklist — Validação da Instalação GitLab CE                    │
│                                                                              │
│  Infraestrutura:                                                             │
│  □ Servidor GitLab acessível via HTTPS                                      │
│  □ Certificado TLS válido (Let's Encrypt ou corporativo)                    │
│  □ SSH habilitado (porta 22)                                                │
│  □ Firewall configurado (80, 443, 22)                                       │
│  □ DNS resolvendo corretamente                                              │
│                                                                              │
│  GitLab:                                                                     │
│  □ Login como root funciona                                                  │
│  □ Senha do root foi alterada                                               │
│  □ Sign-up desabilitado                                                      │
│  □ Usuários criados (ou LDAP configurado)                                   │
│  □ Grupos e projetos criados                                                │
│  □ Envio de email funciona (testar via rails console)                       │
│  □ Container Registry acessível                                             │
│  □ Visibility padrão: Private                                               │
│                                                                              │
│  CI/CD:                                                                      │
│  □ Runner(s) registrados e online (Admin > CI/CD > Runners)                 │
│  □ Pipeline de teste executa corretamente                                   │
│  □ Docker build funciona no runner                                          │
│  □ Artefatos são gerados e armazenados                                      │
│  □ Cache entre builds funciona                                              │
│                                                                              │
│  Backups:                                                                    │
│  □ Backup manual executado e arquivo gerado                                 │
│  □ Backup das configs (/etc/gitlab/) realizado                              │
│  □ Cron de backup diário configurado                                        │
│  □ Restauração testada (idealmente em ambiente separado)                    │
│                                                                              │
│  Monitoramento:                                                              │
│  □ Prometheus coletando métricas                                            │
│  □ Grafana dashboards acessíveis                                            │
│  □ Health check endpoint respondendo                                        │
│  □ Alerta de indisponibilidade configurado                                  │
│                                                                              │
│  Segurança:                                                                  │
│  □ Acesso SSH restrito (chave pública, sem senha)                           │
│  □ Atualizações automáticas de segurança do SO habilitadas                  │
│  □ Secrets do GitLab protegidos (/etc/gitlab/gitlab-secrets.json)           │
│  □ Política de senhas configurada (Admin > Settings > General > Sign-in)    │
│  □ 2FA habilitado (ou enforced) para admins                                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 45.12 Pipeline de Teste — Validar a Instalação

Criar um projeto de teste e executar esta pipeline para validar toda a infraestrutura:

```yaml
# .gitlab-ci.yml — Pipeline de validação da infraestrutura
image: eclipse-temurin:21-jdk

stages:
  - validate
  - build
  - test
  - docker
  - cleanup

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"

# Validar runner Docker
validate_runner:
  stage: validate
  script:
    - echo "Runner OK"
    - java --version
    - uname -a
    - df -h
    - free -m

# Validar cache
validate_cache:
  stage: validate
  cache:
    key: test-cache
    paths:
      - .test-cache/
  script:
    - mkdir -p .test-cache
    - echo "$(date)" >> .test-cache/log.txt
    - cat .test-cache/log.txt
    - echo "Cache entries:"
    - wc -l .test-cache/log.txt

# Validar build Java
build_java:
  stage: build
  script:
    - |
      mkdir -p src/main/java/com/test
      cat > src/main/java/com/test/App.java << 'JAVA'
      package com.test;
      public class App {
          public static String greet() { return "Hello GitLab CI!"; }
          public static void main(String[] args) { System.out.println(greet()); }
      }
      JAVA
    - javac -d target src/main/java/com/test/App.java
    - java -cp target com.test.App
  artifacts:
    paths:
      - target/
    expire_in: 1 hour

# Validar Docker-in-Docker
docker_build:
  stage: docker
  image: docker:27
  services:
    - docker:27-dind
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_TLS_CERTDIR: ""
  script:
    - docker version
    - docker info
    - |
      cat > Dockerfile << 'DOCKERFILE'
      FROM alpine:latest
      RUN echo "GitLab CI Docker test OK"
      DOCKERFILE
    - docker build -t test:latest .
    - docker run test:latest

# Validar Container Registry
registry_push:
  stage: docker
  image: docker:27
  services:
    - docker:27-dind
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_TLS_CERTDIR: ""
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - |
      cat > Dockerfile << 'DOCKERFILE'
      FROM alpine:latest
      RUN echo "Registry test"
      DOCKERFILE
    - docker build -t $CI_REGISTRY_IMAGE:test .
    - docker push $CI_REGISTRY_IMAGE:test
    - docker rmi $CI_REGISTRY_IMAGE:test
    - docker pull $CI_REGISTRY_IMAGE:test
    - echo "Registry push/pull OK!"
  after_script:
    - docker rmi $CI_REGISTRY_IMAGE:test || true

# Validar artefatos
validate_artifacts:
  stage: test
  needs: [build_java]
  script:
    - ls -la target/
    - java -cp target com.test.App
    - echo "Artifacts download OK!"

# Limpeza
cleanup:
  stage: cleanup
  script:
    - echo "All validations passed!"
    - echo "GitLab CE installation is ready for production use."
  when: on_success
```

Executar com `git push` e verificar que todos os jobs passam no pipeline.
