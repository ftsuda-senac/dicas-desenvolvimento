# TypeScript com Node.js — Backend com Express e NestJS

TypeScript combinado com Node.js tornou-se uma das stacks mais populares para desenvolvimento backend moderno. A tipagem estática do TypeScript traz segurança em tempo de compilação, autocompletar rico e refatoração confiável — benefícios críticos em aplicações server-side onde erros em runtime significam downtime. Este guia cobre o desenvolvimento de APIs REST com **Express.js** e **NestJS**, focando no tratamento de requisições HTTP, validação de dados e integração com banco de dados.

> Referência completa: [Node.js Docs](https://nodejs.org/docs/latest/api/) · [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/) · [Express.js](https://expressjs.com/) · [NestJS Docs](https://docs.nestjs.com/)

---

## Sumário

1. [Por que TypeScript no Backend?](#1-por-que-typescript-no-backend)
   - [Node.js/TypeScript vs. Spring Boot — Comparação Geral](#nodejs-typescript-vs-spring-boot--comparação-geral)
2. [Ambiente de Desenvolvimento](#2-ambiente-de-desenvolvimento)
3. [Estrutura de Projeto e Organização de Arquivos](#3-estrutura-de-projeto-e-organização-de-arquivos)
   - [Boas Práticas de Organização](#boas-práticas-de-organização)
   - [Convenções de Nomenclatura](#convenções-de-nomenclatura)
   - [Organização por Camada vs. por Feature](#organização-por-camada-vs-por-feature)
   - [Path Aliases](#path-aliases)
   - [Barrel Files (index.ts)](#barrel-files-indexts)
4. [Express.js com TypeScript](#4-expressjs-com-typescript)
   - [Configuração inicial](#configuração-inicial)
   - [Rotas e Controllers](#rotas-e-controllers)
   - [Middlewares](#middlewares)
   - [Tratamento de erros](#tratamento-de-erros)
5. [NestJS — Framework Opinado](#5-nestjs--framework-opinado)
   - [Configuração inicial](#configuração-inicial-nestjs)
   - [Controllers e Rotas](#controllers-e-rotas)
   - [Services e Injeção de Dependência](#services-e-injeção-de-dependência)
   - [Pipes e Exception Filters](#pipes-e-exception-filters)
6. [Tratamento de Requisições HTTP](#6-tratamento-de-requisições-http)
   - [Parâmetros de rota, query e body](#parâmetros-de-rota-query-e-body)
   - [Upload de arquivos](#upload-de-arquivos)
   - [Headers e Cookies](#headers-e-cookies)
7. [Validação de Dados](#7-validação-de-dados)
   - [Zod (Express)](#zod-express)
   - [class-validator (NestJS)](#class-validator-nestjs)
   - [Validação customizada](#validação-customizada)
8. [Integração com Banco de Dados](#8-integração-com-banco-de-dados)
   - [Prisma ORM](#prisma-orm)
   - [TypeORM](#typeorm)
   - [Drizzle ORM](#drizzle-orm)
   - [Queries e Relacionamentos](#queries-e-relacionamentos)
   - [Migrations](#migrations)
9. [Autenticação e Autorização](#9-autenticação-e-autorização)
10. [Testes](#10-testes)
11. [Deploy e Boas Práticas](#11-deploy-e-boas-práticas)
12. [Referências](#12-referências)

---

## 1. Por que TypeScript no Backend?

| Característica | Detalhe |
|---|---|
| **Tipagem estática** | Erros detectados em tempo de compilação, não em produção às 3h da manhã |
| **Autocompletar inteligente** | IDEs oferecem sugestões precisas para APIs, queries e objetos de domínio |
| **Refatoração segura** | Renomear um campo propaga a mudança para toda a codebase automaticamente |
| **Ecossistema compartilhado** | Mesmo tipo de dado usado no backend e frontend (monorepos com compartilhamento de tipos) |
| **Comunidade ativa** | DefinitelyTyped contém tipagens para milhares de pacotes npm |
| **Suporte nativo do Node.js** | A partir do Node.js 23.6+, TypeScript pode ser executado diretamente com `--experimental-strip-types`; a partir da versão 22.7+ com flag `--experimental-transform-types` |
| **Compatibilidade total** | Todo código JavaScript é código TypeScript válido — migração incremental |

### TypeScript vs. JavaScript no Backend — Quando Escolher

| Cenário | Recomendação |
|---|---|
| API REST com múltiplas entidades e relações | TypeScript |
| Script simples de automação / one-off | JavaScript |
| Projeto com equipe de 3+ desenvolvedores | TypeScript |
| Prototipagem rápida / PoC descartável | JavaScript |
| Microsserviço com contrato de API definido | TypeScript |
| Lambda/Cloud Function simples | JavaScript ou TypeScript |

### Node.js/TypeScript vs. Spring Boot — Comparação Geral

A escolha entre Node.js com TypeScript e Java com Spring Boot envolve trade-offs em desempenho, produtividade, uso de recursos e maturidade do ecossistema. A tabela abaixo resume os pontos principais:

| Aspecto | Node.js + TypeScript | Java + Spring Boot |
|---|---|---|
| **Modelo de execução** | Single-thread com event loop não-bloqueante | Multi-thread com pool de threads |
| **Concorrência** | Excelente para I/O-bound (milhares de conexões simultâneas com pouca memória) | Excelente para CPU-bound; Virtual Threads (Java 21+) eliminam a limitação de threads do SO para I/O |
| **Cold start** | ~100-300 ms — ideal para serverless/Lambda | ~2-8 s (JVM warmup); GraalVM Native Image reduz para ~100 ms, mas com trade-offs |
| **Consumo de memória** | ~30-80 MB para uma API simples | ~150-400 MB (JVM + classloading + metaspace) |
| **Throughput (CPU-bound)** | Limitado pelo single-thread; Worker Threads ajudam mas sem o mesmo nível | Superior — JIT (C2) otimiza hot paths ao longo do tempo, GC maduro (G1/ZGC) |
| **Throughput (I/O-bound)** | Altíssimo — event loop processa milhares de requisições sem criar threads | Comparável com Virtual Threads; WebFlux (reativo) se equipara ao modelo do Node |

#### Produtividade e Experiência de Desenvolvimento

| Aspecto | Node.js + TypeScript | Java + Spring Boot |
|---|---|---|
| **Curva de aprendizado** | Baixa para quem já conhece JavaScript; TypeScript adiciona gradualmente | Mais íngreme (JVM, build tools, anotações, IoC/DI, JPA) |
| **Feedback loop** | Instantâneo com `tsx watch` — salvar e ver o resultado | Spring Boot DevTools oferece hot-reload, mas reinício parcial leva ~2-5 s |
| **Verbosidade** | Conciso — funções, arrow functions, destructuring, template literals | Mais verboso — classes obrigatórias, getters/setters, anotações extensas |
| **Tipagem** | Estrutural (duck typing) — mais flexível, mas erros sutis com `any` | Nominal e rígida — mais segura, mas requer boilerplate (generics, casts) |
| **Ecossistema de pacotes** | npm com ~2.5M pacotes; qualidade variável, muitas opções para cada necessidade | Maven Central bem curado; menos opções, mas mais estáveis e mantidas |
| **Monorepo full-stack** | Natural — mesmo linguagem no frontend e backend, tipos compartilhados | Possível mas com atrito — frontend em JS/TS e backend em Java são mundos separados |
| **Tooling** | ESLint, Prettier, Vitest — leves, rápidos, configuração simples | Checkstyle, SpotBugs, JUnit, JaCoCo — mais pesados, mas mais maduros em análise estática |

#### Ecossistema e Maturidade

| Aspecto | Node.js + TypeScript | Java + Spring Boot |
|---|---|---|
| **ORM / Banco de dados** | Prisma, TypeORM, Drizzle — type-safe, migrations automáticas | JPA/Hibernate — padrão da indústria, extremamente maduro, documentação vasta |
| **Validação** | Zod (schema-first), class-validator (decorators) | Bean Validation (JSR 380) — padrão Java, integrado ao Spring |
| **Autenticação** | Passport.js, jose — funcional mas montagem manual | Spring Security — completo (OAuth2, SAML, LDAP, CORS, CSRF) mas complexo |
| **Observabilidade** | pino/winston + OpenTelemetry | SLF4J/Logback + Micrometer + Actuator — integração nativa e madura |
| **Documentação de API** | Swagger via `@nestjs/swagger` ou `swagger-jsdoc` | SpringDoc/Swagger — geração automática a partir das anotações |
| **Mensageria** | BullMQ (Redis), kafkajs — funcional | Spring Kafka, Spring AMQP, Spring Cloud Stream — abstrações maduras |
| **Testes** | Vitest/Jest + Supertest — rápidos, setup simples | JUnit 5 + Mockito + Testcontainers — robusto, mais verboso |

#### Operações e Deploy

| Aspecto | Node.js + TypeScript | Java + Spring Boot |
|---|---|---|
| **Imagem Docker** | ~50-150 MB (node:alpine + deps) | ~200-400 MB (JRE + app); ~30 MB com GraalVM Native Image |
| **Serverless** | Excelente — cold start rápido, baixo consumo | Viável com GraalVM ou SnapStart (AWS); JVM pura penaliza cold start |
| **Escalabilidade horizontal** | Fácil — processos leves, rápido para subir/descer | Fácil — mas cada instância consome mais memória |
| **Estabilidade em produção** | Boa, mas event loop bloqueado trava toda a aplicação | Excelente — uma thread travada não afeta as demais |
| **Debugging em produção** | `--inspect` + Chrome DevTools; diagnóstico de memória menos maduro | JFR, async-profiler, jmap, jstack — ferramentas de diagnóstico de classe enterprise |
| **Suporte enterprise** | Sem suporte comercial oficial do Node.js (comunidade-driven) | Suporte comercial via VMware/Broadcom (Spring), Oracle (JVM), Red Hat (Quarkus) |

#### Quando Escolher Cada Stack

| Cenário | Recomendação |
|---|---|
| API REST para SPA/mobile com equipe full-stack JS/TS | **Node.js + TypeScript** |
| BFF (Backend for Frontend) para agregar microsserviços | **Node.js + TypeScript** |
| Serverless / Functions com requisito de cold start rápido | **Node.js + TypeScript** |
| Aplicação real-time (WebSocket, SSE, chat) | **Node.js + TypeScript** |
| Prototipagem rápida e MVPs | **Node.js + TypeScript** |
| Sistema corporativo com regras de negócio complexas | **Java + Spring Boot** |
| Aplicação que exige processamento pesado de CPU | **Java + Spring Boot** |
| Integração com sistemas legados Java/JEE | **Java + Spring Boot** |
| Requisitos de conformidade e suporte enterprise | **Java + Spring Boot** |
| Processamento de grandes volumes de dados (batch) | **Java + Spring Boot** |
| Microsserviço I/O-bound leve em ambiente containerizado | **Ambos são boas opções** |
| API GraphQL | **Ambos são boas opções** |

> **Nota prática:** a escolha entre as stacks raramente é puramente técnica. Fatores como experiência da equipe, ecossistema existente na empresa, requisitos de contratação e integração com sistemas legados geralmente pesam mais que benchmarks de performance. Uma equipe produtiva em Node.js/TypeScript entrega mais valor do que uma equipe aprendendo Spring Boot — e vice-versa.

---

## 2. Ambiente de Desenvolvimento

### Instalação e Ferramentas

```bash
# Instalar Node.js 22+ (LTS) via nvm (recomendado)
nvm install 22
nvm use 22

# Verificar versões
node --version   # v22.x.x
npm --version    # 10.x.x
```

### Inicialização de Projeto TypeScript

```bash
# Criar projeto
mkdir minha-api && cd minha-api
npm init -y

# Instalar TypeScript e tipos do Node
npm install -D typescript @types/node

# Gerar tsconfig.json
npx tsc --init
```

### tsconfig.json recomendado para backend

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**Opções importantes para backend:**

| Opção | Propósito |
|---|---|
| `strict: true` | Ativa todas as verificações de tipo estritas — **sempre usar** |
| `noUncheckedIndexedAccess` | Acesso a arrays/objetos retorna `T \| undefined`, forçando verificação de null |
| `exactOptionalPropertyTypes` | Diferencia `prop?: string` (ausente) de `prop: string \| undefined` (presente mas undefined) |
| `sourceMap: true` | Mapeia erros em runtime de volta para o código TypeScript original |

### Ferramentas de Desenvolvimento

```bash
# tsx — executor TypeScript rápido com watch mode (substitui ts-node)
npm install -D tsx

# Scripts no package.json
# "dev": "tsx watch src/server.ts"
# "build": "tsc"
# "start": "node dist/server.js"
```

### ESLint e Prettier

```bash
# ESLint com suporte TypeScript
npm install -D eslint @eslint/js typescript-eslint

# Prettier
npm install -D prettier eslint-config-prettier
```

```js
// eslint.config.mjs
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  { ignores: ['dist/'] }
);
```

---

## 3. Estrutura de Projeto e Organização de Arquivos

### Express.js — Estrutura Recomendada

```
projeto-express/
├── src/
│   ├── controllers/        # Handlers de rotas
│   │   ├── user.controller.ts
│   │   └── product.controller.ts
│   ├── services/           # Lógica de negócio
│   │   ├── user.service.ts
│   │   └── product.service.ts
│   ├── repositories/       # Acesso a dados
│   │   └── user.repository.ts
│   ├── middlewares/         # Middlewares customizados
│   │   ├── auth.middleware.ts
│   │   ├── validation.middleware.ts
│   │   └── error-handler.middleware.ts
│   ├── routes/             # Definição de rotas
│   │   ├── user.routes.ts
│   │   └── index.ts
│   ├── schemas/            # Schemas de validação (Zod)
│   │   └── user.schema.ts
│   ├── types/              # Tipos e interfaces
│   │   └── index.ts
│   ├── config/             # Configurações
│   │   └── database.ts
│   ├── app.ts              # Configuração do Express
│   └── server.ts           # Entry point
├── prisma/
│   └── schema.prisma       # Schema do banco de dados
├── tests/
│   ├── unit/
│   └── integration/
├── tsconfig.json
├── package.json
└── .env
```

### NestJS — Estrutura Padrão (gerada pelo CLI)

```
projeto-nest/
├── src/
│   ├── users/              # Módulo de Usuários
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   └── update-user.dto.ts
│   │   └── entities/
│   │       └── user.entity.ts
│   ├── products/           # Módulo de Produtos
│   │   └── ...
│   ├── common/             # Pipes, Guards, Interceptors
│   │   ├── pipes/
│   │   ├── guards/
│   │   └── filters/
│   ├── config/             # Configuração
│   │   └── database.config.ts
│   ├── app.module.ts       # Módulo raiz
│   └── main.ts             # Entry point
├── test/
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.json
├── nest-cli.json
└── package.json
```

> **Diferença fundamental:** No Express você monta a arquitetura manualmente — é flexível mas exige disciplina. No NestJS a estrutura modular é imposta pelo framework via decorators e injeção de dependência, semelhante ao Spring Boot / Angular.

### Boas Práticas de Organização

#### 1. Um arquivo = uma responsabilidade

Cada arquivo deve exportar **uma única unidade lógica** — um controller, um service, um schema de validação. Isso facilita localização, testes e code review.

```
# Ruim — arquivo "faz-tudo"
src/users.ts             # controller + service + tipos + validação

# Bom — separação clara
src/users/
├── users.controller.ts  # apenas handlers de rota
├── users.service.ts     # apenas lógica de negócio
├── users.schema.ts      # apenas validação (Zod)
└── users.types.ts       # apenas interfaces/types
```

#### 2. Separar camadas com responsabilidades claras

Independentemente do framework, o backend deve seguir uma separação em camadas. A regra de dependência é de cima para baixo — uma camada só conhece a camada imediatamente abaixo:

```
┌─────────────────────────────────────────────┐
│  Routes / Controllers                       │  ← Recebe HTTP, delega, retorna resposta
│  (Express: Router + handler functions)      │
│  (NestJS: @Controller com decorators)       │
├─────────────────────────────────────────────┤
│  Services                                   │  ← Regras de negócio, orquestração
│  (Express: funções ou classes exportadas)    │
│  (NestJS: @Injectable)                      │
├─────────────────────────────────────────────┤
│  Repositories / Data Access                 │  ← Queries, ORM, acesso a dados
│  (Prisma Client, TypeORM Repository, etc.)  │
├─────────────────────────────────────────────┤
│  Database                                   │
└─────────────────────────────────────────────┘
```

**Regras de ouro:**

| Regra | Motivo |
|---|---|
| Controller **nunca** acessa o banco diretamente | Mantém a lógica de negócio testável sem HTTP |
| Service **nunca** acessa `req` ou `res` | Permite reutilizar o service em CLI, fila, WebSocket |
| Repository isola as queries do ORM | Trocar de Prisma para Drizzle afeta só esta camada |
| Validação ocorre **antes** do controller processar | Dados inválidos nunca chegam ao service |

```ts
// Ruim — controller acessa banco e lógica misturada
app.post('/api/users', async (req, res) => {
  const exists = await prisma.user.findUnique({ where: { email: req.body.email } });
  if (exists) return res.status(409).json({ error: 'E-mail já cadastrado' });
  const hashed = await bcrypt.hash(req.body.password, 12);
  const user = await prisma.user.create({ data: { ...req.body, password: hashed } });
  res.status(201).json(user);
});

// Bom — controller delega para service
app.post('/api/users', validate(createUserSchema), async (req, res, next) => {
  try {
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
});
```

#### 3. Agrupar arquivos transversais em `common/` ou `shared/`

Código que não pertence a nenhum módulo de domínio específico vai em um diretório transversal:

```
src/
├── common/                     # ou shared/
│   ├── middlewares/             # auth, rate-limit, logging
│   ├── guards/                 # (NestJS) guards globais
│   ├── pipes/                  # (NestJS) pipes globais
│   ├── filters/                # (NestJS) exception filters
│   ├── interceptors/           # (NestJS) interceptors
│   ├── decorators/             # decorators customizados
│   ├── utils/                  # funções utilitárias puras
│   │   ├── hash.ts
│   │   ├── pagination.ts
│   │   └── date.ts
│   └── types/                  # tipos globais, interfaces de resposta
│       ├── pagination.types.ts
│       └── api-response.types.ts
├── config/                     # variáveis de ambiente, conexão de banco
│   ├── env.ts
│   └── database.ts
└── modules/                    # ou cada módulo na raiz de src/
    ├── users/
    └── products/
```

### Convenções de Nomenclatura

#### Nomes de Arquivos

O ecossistema Node.js/TypeScript adota **kebab-case** para nomes de arquivos, com sufixo indicando o papel:

| Padrão | Exemplo | Papel |
|---|---|---|
| `*.controller.ts` | `users.controller.ts` | Handlers de rota / endpoints |
| `*.service.ts` | `users.service.ts` | Lógica de negócio |
| `*.repository.ts` | `users.repository.ts` | Acesso a dados |
| `*.module.ts` | `users.module.ts` | Módulo NestJS |
| `*.routes.ts` | `users.routes.ts` | Definição de rotas Express |
| `*.schema.ts` | `users.schema.ts` | Schema de validação (Zod) |
| `*.dto.ts` | `create-user.dto.ts` | Data Transfer Object (NestJS) |
| `*.entity.ts` | `user.entity.ts` | Entidade do ORM |
| `*.types.ts` | `users.types.ts` | Interfaces e types |
| `*.middleware.ts` | `auth.middleware.ts` | Middleware Express/NestJS |
| `*.guard.ts` | `roles.guard.ts` | Guard NestJS |
| `*.pipe.ts` | `parse-date.pipe.ts` | Pipe NestJS |
| `*.filter.ts` | `http-exception.filter.ts` | Exception Filter NestJS |
| `*.interceptor.ts` | `logging.interceptor.ts` | Interceptor NestJS |
| `*.spec.ts` / `*.test.ts` | `users.service.spec.ts` | Testes unitários |
| `*.e2e-spec.ts` | `users.e2e-spec.ts` | Testes de integração / E2E |
| `*.config.ts` | `database.config.ts` | Configuração |
| `*.constant.ts` | `roles.constant.ts` | Constantes |

> **Comparação com Spring Boot:** o sufixo nos nomes segue a mesma lógica — `UserController.java`, `UserService.java`, `UserRepository.java`. A diferença é que no Java cada classe fica em um arquivo com o nome da classe (PascalCase), enquanto no TypeScript/Node.js o padrão é kebab-case com sufixo separado por ponto.

#### Nomes no Código

| Elemento | Convenção | Exemplo |
|---|---|---|
| **Variáveis e funções** | camelCase | `findAllUsers`, `userService`, `isActive` |
| **Classes** | PascalCase | `UsersController`, `CreateUserDto` |
| **Interfaces e Types** | PascalCase | `UserResponse`, `PaginationParams` |
| **Enums** | PascalCase (membros em UPPER_SNAKE) | `enum Role { ADMIN, USER }` |
| **Constantes** | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE`, `JWT_EXPIRES_IN` |
| **Arquivos** | kebab-case com sufixo | `create-user.dto.ts` |
| **Diretórios** | kebab-case (plural para coleções) | `users/`, `common/`, `dto/` |

### Organização por Camada vs. por Feature

Existem duas abordagens principais para organizar os arquivos. A escolha impacta diretamente a escalabilidade do projeto.

#### Por Camada (Layer-first)

Todos os arquivos do mesmo tipo ficam juntos. Funciona bem para projetos pequenos (até ~10 entidades):

```
src/
├── controllers/
│   ├── users.controller.ts
│   ├── products.controller.ts
│   ├── orders.controller.ts
│   └── categories.controller.ts
├── services/
│   ├── users.service.ts
│   ├── products.service.ts
│   ├── orders.service.ts
│   └── categories.service.ts
├── repositories/
│   ├── users.repository.ts
│   ├── products.repository.ts
│   └── orders.repository.ts
├── schemas/
│   ├── users.schema.ts
│   ├── products.schema.ts
│   └── orders.schema.ts
├── types/
│   ├── users.types.ts
│   ├── products.types.ts
│   └── orders.types.ts
└── routes/
    ├── users.routes.ts
    ├── products.routes.ts
    └── index.ts
```

**Problema:** à medida que o projeto cresce, adicionar uma feature exige tocar 5-6 diretórios diferentes. Entender todo o código de "orders" exige navegar em pastas espalhadas.

#### Por Feature / Módulo (Feature-first) — Recomendado

Cada feature agrupa todos os seus arquivos. É o padrão do NestJS e a organização recomendada para projetos Express com mais de 5-6 entidades:

```
src/
├── users/
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.repository.ts
│   ├── users.routes.ts          # Express
│   ├── users.module.ts          # NestJS
│   ├── users.schema.ts
│   ├── users.types.ts
│   └── __tests__/
│       ├── users.service.spec.ts
│       └── users.controller.spec.ts
├── products/
│   ├── products.controller.ts
│   ├── products.service.ts
│   ├── products.schema.ts
│   ├── products.types.ts
│   └── __tests__/
│       └── products.service.spec.ts
├── orders/
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   ├── dto/                     # NestJS — DTOs específicos do módulo
│   │   ├── create-order.dto.ts
│   │   └── update-order.dto.ts
│   └── entities/                # Entidades ORM do módulo
│       └── order.entity.ts
├── common/                      # Código transversal
│   ├── middlewares/
│   ├── utils/
│   └── types/
└── config/
```

**Vantagens:**

| Benefício | Detalhe |
|---|---|
| **Coesão** | Tudo sobre "users" está em `src/users/` — fácil de encontrar e entender |
| **Independência** | Cada módulo pode ser movido, extraído para um microsserviço ou deletado sem mexer nos demais |
| **Code review** | PRs de uma feature tocam poucos diretórios, facilitando revisão |
| **Testes co-localizados** | Testes ficam perto do código que testam, sem duplicar a árvore em outro diretório |
| **Onboarding** | Novos desenvolvedores entendem um módulo sem conhecer o projeto inteiro |

> **Analogia com Spring Boot:** a organização por feature é equivalente a estruturar pacotes Java como `com.app.users`, `com.app.products`, `com.app.orders` em vez de `com.app.controllers`, `com.app.services`, `com.app.repositories`.

#### Organização Híbrida para Projetos Grandes

Em projetos maiores, é comum combinar agrupamento por domínio de negócio no nível externo e por camada dentro de cada módulo:

```
src/
├── modules/
│   ├── identity/               # Domínio: identidade e acesso
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── strategies/
│   │   │   │   ├── jwt.strategy.ts
│   │   │   │   └── local.strategy.ts
│   │   │   └── guards/
│   │   │       └── roles.guard.ts
│   │   └── users/
│   │       ├── users.controller.ts
│   │       ├── users.service.ts
│   │       └── users.repository.ts
│   ├── catalog/                # Domínio: catálogo de produtos
│   │   ├── products/
│   │   ├── categories/
│   │   └── reviews/
│   └── sales/                  # Domínio: vendas
│       ├── orders/
│       ├── cart/
│       └── payments/
├── common/
│   ├── database/
│   │   ├── prisma.service.ts
│   │   └── prisma.module.ts
│   ├── middlewares/
│   ├── interceptors/
│   └── types/
│       ├── pagination.types.ts
│       └── api-response.types.ts
├── config/
│   ├── env.ts
│   ├── env.schema.ts
│   └── cors.config.ts
├── app.module.ts               # NestJS
├── app.ts                      # Express
└── main.ts / server.ts
```

#### Quando Usar Cada Abordagem

| Critério | Por Camada | Por Feature | Híbrida |
|---|---|---|---|
| **Tamanho do projeto** | Pequeno (≤5 entidades) | Médio (5-20 entidades) | Grande (20+ entidades) |
| **Equipe** | 1-2 devs | 3-8 devs | 8+ devs, múltiplos times |
| **Framework** | Express simples | Express ou NestJS | NestJS (módulos obrigatórios) |
| **Migração para microsserviços** | Difícil (código misturado) | Natural (cada feature = potencial serviço) | Ideal (cada domínio = serviço) |

### Path Aliases

Em projetos com muitos níveis de diretório, os imports relativos ficam difíceis de ler. Path aliases resolvem isso com caminhos absolutos semânticos.

```ts
// Ruim — imports relativos profundos e frágeis
import { PrismaService } from '../../../common/database/prisma.service';
import { PaginationParams } from '../../../common/types/pagination.types';

// Bom — path aliases
import { PrismaService } from '@/common/database/prisma.service';
import { PaginationParams } from '@/common/types/pagination.types';
```

**Configuração no `tsconfig.json`:**

```jsonc
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@common/*": ["src/common/*"],
      "@config/*": ["src/config/*"],
      "@modules/*": ["src/modules/*"]
    }
  }
}
```

**Para funcionar em runtime** é necessário um dos seguintes:

| Ferramenta | Configuração |
|---|---|
| **tsx** | Resolve `paths` do tsconfig automaticamente |
| **tsc + tsc-alias** | `npm install -D tsc-alias` → build script: `tsc && tsc-alias` |
| **SWC (NestJS)** | Funciona com paths nativamente via `nest-cli.json` |

### Barrel Files (index.ts)

Barrel files (`index.ts`) reexportam os membros públicos de um módulo, simplificando os imports. Use com moderação — barrels grandes podem causar imports circulares e prejudicar o tree-shaking.

```ts
// src/users/index.ts
export { UsersController } from './users.controller';
export { UsersService } from './users.service';
export { UsersModule } from './users.module';
export type { CreateUserInput, UpdateUserInput } from './users.types';
```

```ts
// Antes (sem barrel)
import { UsersService } from '../users/users.service';
import { CreateUserInput } from '../users/users.types';

// Depois (com barrel)
import { UsersService, CreateUserInput } from '../users';
```

**Quando usar e quando evitar:**

| Situação | Recomendação |
|---|---|
| Módulo com API pública clara (exports que outros módulos consomem) | Usar barrel — organiza a interface pública |
| Módulo interno (ex: `utils/`, `config/`) | Usar barrel — reduz imports repetitivos |
| Módulo com muitas classes que nem sempre são usadas juntas | Evitar barrel — causa imports desnecessários |
| Projetos com problemas de imports circulares | Evitar barrels nos módulos envolvidos no ciclo |

> **Regra prática:** crie barrels apenas quando um diretório é consumido por **outros módulos**. Dentro do próprio módulo, use imports diretos para evitar ciclos.

### Resumo — Checklist de Organização

| Prática | Express | NestJS |
|---|---|---|
| Sufixo no nome do arquivo (`.controller.ts`, `.service.ts`) | Convenção da comunidade | Gerado pelo CLI (`nest g resource`) |
| Separação controller → service → repository | Manual, exige disciplina | Forçada pelo framework (DI) |
| Organização por feature | Recomendada manualmente | Padrão (cada `nest g resource` cria um módulo) |
| Path aliases (`@/`, `@common/`) | Configurar `tsconfig.json` + `tsc-alias` ou `tsx` | Funciona nativamente com SWC |
| Co-localização de testes (`__tests__/` dentro do módulo) | Boa prática, mas não é padrão | Gerado pelo CLI (`.spec.ts` ao lado do arquivo) |
| Barrel files (`index.ts`) | Útil para módulos com API pública | Útil, mas NestJS já resolve via DI |
| Diretório `common/` ou `shared/` para código transversal | Essencial — middlewares, utils, types | Essencial — guards, pipes, filters, interceptors |
| Um arquivo por responsabilidade | Disciplina do time | Padrão natural do framework |

---

## 4. Express.js com TypeScript

Express é o framework HTTP minimalista mais popular do Node.js. Com TypeScript, ganha tipagem para request, response, middlewares e handlers.

### Configuração Inicial

```bash
npm install express
npm install -D @types/express
```

```ts
// src/app.ts
import express from 'express';
import { userRoutes } from './routes/user.routes';
import { errorHandler } from './middlewares/error-handler.middleware';

const app = express();

// Middlewares globais
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Rotas
app.use('/api/users', userRoutes);

// Handler de erros — deve ser o último middleware
app.use(errorHandler);

export { app };
```

```ts
// src/server.ts
import { app } from './app';

const PORT = process.env.PORT ?? 3000;

app.listen(PORT, () => {
  console.log(`Servidor rodando em http://localhost:${PORT}`);
});
```

### Rotas e Controllers

```ts
// src/types/index.ts
export interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

export type CreateUserInput = Omit<User, 'id' | 'createdAt'>;
export type UpdateUserInput = Partial<CreateUserInput>;
```

```ts
// src/routes/user.routes.ts
import { Router } from 'express';
import * as userController from '../controllers/user.controller';
import { validate } from '../middlewares/validation.middleware';
import { createUserSchema, updateUserSchema } from '../schemas/user.schema';

const router = Router();

router.get('/',       userController.findAll);
router.get('/:id',    userController.findById);
router.post('/',      validate(createUserSchema), userController.create);
router.put('/:id',    validate(updateUserSchema), userController.update);
router.delete('/:id', userController.remove);

export { router as userRoutes };
```

```ts
// src/controllers/user.controller.ts
import type { Request, Response, NextFunction } from 'express';
import * as userService from '../services/user.service';

export async function findAll(req: Request, res: Response, next: NextFunction) {
  try {
    const { page = '1', limit = '10', role } = req.query;

    const users = await userService.findAll({
      page: Number(page),
      limit: Number(limit),
      role: role as string | undefined,
    });

    res.json(users);
  } catch (error) {
    next(error);
  }
}

export async function findById(req: Request, res: Response, next: NextFunction) {
  try {
    const user = await userService.findById(Number(req.params.id));

    if (!user) {
      res.status(404).json({ message: 'Usuário não encontrado' });
      return;
    }

    res.json(user);
  } catch (error) {
    next(error);
  }
}

export async function create(req: Request, res: Response, next: NextFunction) {
  try {
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
}

export async function update(req: Request, res: Response, next: NextFunction) {
  try {
    const user = await userService.update(Number(req.params.id), req.body);

    if (!user) {
      res.status(404).json({ message: 'Usuário não encontrado' });
      return;
    }

    res.json(user);
  } catch (error) {
    next(error);
  }
}

export async function remove(req: Request, res: Response, next: NextFunction) {
  try {
    await userService.remove(Number(req.params.id));
    res.status(204).send();
  } catch (error) {
    next(error);
  }
}
```

### Tipagem Customizada do Request

Quando middlewares adicionam dados ao `req` (ex: usuário autenticado), é preciso estender a interface:

```ts
// src/types/express.d.ts
declare namespace Express {
  interface Request {
    userId?: number;
    userRole?: 'admin' | 'user';
  }
}
```

Assim `req.userId` é acessível com tipagem em qualquer handler sem casts.

### Middlewares

```ts
// src/middlewares/auth.middleware.ts
import type { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

interface JwtPayload {
  userId: number;
  role: 'admin' | 'user';
}

export function authenticate(req: Request, res: Response, next: NextFunction) {
  const header = req.headers.authorization;

  if (!header?.startsWith('Bearer ')) {
    res.status(401).json({ message: 'Token não fornecido' });
    return;
  }

  try {
    const token = header.slice(7);
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;

    req.userId = payload.userId;
    req.userRole = payload.role;
    next();
  } catch {
    res.status(401).json({ message: 'Token inválido' });
  }
}

export function authorize(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.userRole || !roles.includes(req.userRole)) {
      res.status(403).json({ message: 'Acesso negado' });
      return;
    }
    next();
  };
}
```

**Uso nas rotas:**

```ts
router.delete('/:id', authenticate, authorize('admin'), userController.remove);
```

### Tratamento de Erros

```ts
// src/middlewares/error-handler.middleware.ts
import type { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  constructor(
    public statusCode: number,
    message: string,
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export function errorHandler(
  err: Error,
  _req: Request,
  res: Response,
  _next: NextFunction,
) {
  console.error(`[ERROR] ${err.message}`, err.stack);

  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: err.message,
    });
    return;
  }

  res.status(500).json({
    error: 'Erro interno do servidor',
  });
}
```

**Uso no service:**

```ts
import { AppError } from '../middlewares/error-handler.middleware';

export async function findById(id: number) {
  const user = await prisma.user.findUnique({ where: { id } });
  if (!user) throw new AppError(404, 'Usuário não encontrado');
  return user;
}
```

---

## 5. NestJS — Framework Opinado

NestJS é um framework para Node.js construído sobre Express (ou Fastify), inspirado fortemente no Angular e no Spring Boot. Usa decorators, injeção de dependência e módulos para organizar aplicações de forma escalável.

### Configuração Inicial (NestJS)

```bash
# Instalar CLI global
npm install -g @nestjs/cli

# Criar projeto (escolher npm/yarn/pnpm)
nest new minha-api-nest

# Gerar módulo, controller e service
nest generate resource users
# Seleciona REST API → gera module, controller, service, DTOs e testes
```

### Arquitetura NestJS — Fluxo da Requisição

```
Request HTTP
    │
    ▼
┌─────────┐    ┌──────────┐    ┌────────────┐    ┌──────────┐
│  Guard   │───▶│   Pipe   │───▶│ Controller │───▶│ Service  │
│ (auth)   │    │(validação│    │  (rota)    │    │ (lógica) │
└─────────┘    └──────────┘    └────────────┘    └──────────┘
                                                       │
                                                       ▼
                                                ┌──────────┐
                                                │Repository│
                                                │  (dados) │
                                                └──────────┘
    │
    ▼
┌──────────────┐    ┌──────────┐
│ Interceptor  │───▶│ Response │
│(transformação│    │          │
└──────────────┘    └──────────┘
```

### Controllers e Rotas

```ts
// src/users/users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  ParseIntPipe,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('api/users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(
    @Query('page') page = 1,
    @Query('limit') limit = 10,
    @Query('role') role?: string,
  ) {
    return this.usersService.findAll({ page: +page, limit: +limit, role });
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: UpdateUserDto,
  ) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.remove(id);
  }
}
```

> **Comparação com Spring Boot:** `@Controller` e `@Get/@Post` no NestJS correspondem a `@RestController` e `@GetMapping/@PostMapping` no Spring. `@Body()` é equivalente a `@RequestBody`, `@Param()` a `@PathVariable`, e `@Query()` a `@RequestParam`.

### Services e Injeção de Dependência

```ts
// src/users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(params: { page: number; limit: number; role?: string }) {
    const { page, limit, role } = params;
    const skip = (page - 1) * limit;

    const where = role ? { role } : {};

    const [data, total] = await Promise.all([
      this.prisma.user.findMany({ where, skip, take: limit, orderBy: { createdAt: 'desc' } }),
      this.prisma.user.count({ where }),
    ]);

    return {
      data,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async findOne(id: number) {
    const user = await this.prisma.user.findUnique({ where: { id } });
    if (!user) throw new NotFoundException(`Usuário #${id} não encontrado`);
    return user;
  }

  async create(dto: CreateUserDto) {
    return this.prisma.user.create({ data: dto });
  }

  async update(id: number, dto: UpdateUserDto) {
    await this.findOne(id);
    return this.prisma.user.update({ where: { id }, data: dto });
  }

  async remove(id: number) {
    await this.findOne(id);
    return this.prisma.user.delete({ where: { id } });
  }
}
```

### Módulos

```ts
// src/users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { PrismaModule } from '../prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';

@Module({
  imports: [UsersModule, ProductsModule],
})
export class AppModule {}
```

### Pipes e Exception Filters

**Pipes** transformam e validam dados de entrada. **Exception Filters** capturam exceções e formatam a resposta de erro.

```ts
// Pipe customizado — converte string para Date
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseDatePipe implements PipeTransform<string, Date> {
  transform(value: string): Date {
    const date = new Date(value);
    if (isNaN(date.getTime())) {
      throw new BadRequestException(`"${value}" não é uma data válida`);
    }
    return date;
  }
}

// Uso no controller
@Get('by-date')
findByDate(@Query('from', ParseDatePipe) from: Date) {
  return this.service.findByDate(from);
}
```

```ts
// Exception Filter customizado
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
} from '@nestjs/common';
import type { Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message:
        typeof exceptionResponse === 'string'
          ? exceptionResponse
          : (exceptionResponse as Record<string, unknown>).message,
    });
  }
}

// Registrar globalmente em main.ts
// app.useGlobalFilters(new HttpExceptionFilter());
```

---

## 6. Tratamento de Requisições HTTP

### Parâmetros de Rota, Query e Body

#### Express

```ts
// GET /api/products/42?fields=name,price
app.get('/api/products/:id', (req: Request, res: Response) => {
  const id: string = req.params.id;
  const fields: string | undefined = req.query.fields as string | undefined;
  // ...
});

// POST /api/products — body JSON
app.post('/api/products', (req: Request, res: Response) => {
  const { name, price, category }: CreateProductInput = req.body;
  // ...
});

// Múltiplos parâmetros de rota
// GET /api/stores/5/products/42
app.get('/api/stores/:storeId/products/:productId', (req, res) => {
  const { storeId, productId } = req.params;
  // ...
});
```

#### NestJS

```ts
@Controller('api/products')
export class ProductsController {
  // GET /api/products/42?fields=name,price
  @Get(':id')
  findOne(
    @Param('id', ParseIntPipe) id: number,
    @Query('fields') fields?: string,
  ) {
    const fieldList = fields?.split(',');
    return this.service.findOne(id, fieldList);
  }

  // POST /api/products
  @Post()
  create(@Body() dto: CreateProductDto) {
    return this.service.create(dto);
  }

  // Parâmetro de rota tipado
  @Get(':storeId/products/:productId')
  findStoreProduct(
    @Param('storeId', ParseIntPipe) storeId: number,
    @Param('productId', ParseIntPipe) productId: number,
  ) {
    return this.service.findStoreProduct(storeId, productId);
  }
}
```

### Upload de Arquivos

#### Express (com multer)

```bash
npm install multer
npm install -D @types/multer
```

```ts
import multer from 'multer';
import path from 'node:path';

const storage = multer.diskStorage({
  destination: './uploads',
  filename: (_req, file, cb) => {
    const uniqueName = `${Date.now()}-${Math.round(Math.random() * 1e9)}`;
    cb(null, `${uniqueName}${path.extname(file.originalname)}`);
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5 MB
  fileFilter: (_req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowed.includes(file.mimetype));
  },
});

router.post('/avatar', upload.single('avatar'), (req, res) => {
  if (!req.file) {
    res.status(400).json({ message: 'Arquivo não enviado' });
    return;
  }
  res.json({ path: req.file.path, size: req.file.size });
});
```

#### NestJS (com @nestjs/platform-express)

```ts
import {
  Controller,
  Post,
  UseInterceptors,
  UploadedFile,
  ParseFilePipe,
  MaxFileSizeValidator,
  FileTypeValidator,
} from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';

@Controller('api/files')
export class FilesController {
  @Post('upload')
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(
    @UploadedFile(
      new ParseFilePipe({
        validators: [
          new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }),
          new FileTypeValidator({ fileType: /image\/(jpeg|png|webp)/ }),
        ],
      }),
    )
    file: Express.Multer.File,
  ) {
    return {
      originalName: file.originalname,
      size: file.size,
      mimetype: file.mimetype,
    };
  }
}
```

### Headers e Cookies

#### Express

```ts
app.get('/api/info', (req: Request, res: Response) => {
  // Ler headers
  const userAgent = req.headers['user-agent'];
  const customHeader = req.headers['x-request-id'];

  // Ler cookies (requer cookie-parser)
  const sessionId = req.cookies.sessionId;

  // Definir headers na resposta
  res.setHeader('X-Request-Id', crypto.randomUUID());

  // Definir cookie
  res.cookie('sessionId', 'abc123', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000, // 1 dia
  });

  res.json({ userAgent });
});
```

#### NestJS

```ts
import { Controller, Get, Headers, Res } from '@nestjs/common';
import type { Response } from 'express';

@Controller('api/info')
export class InfoController {
  @Get()
  getInfo(
    @Headers('user-agent') userAgent: string,
    @Headers('x-request-id') requestId: string,
    @Res({ passthrough: true }) res: Response,
  ) {
    res.setHeader('X-Request-Id', crypto.randomUUID());
    res.cookie('sessionId', 'abc123', {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
    });
    return { userAgent };
  }
}
```

---

## 7. Validação de Dados

### Zod (Express)

Zod é a biblioteca de validação mais popular para TypeScript. Define schemas que inferem tipos automaticamente — **um único schema gera tanto o tipo quanto a validação em runtime**.

```bash
npm install zod
```

```ts
// src/schemas/user.schema.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  name: z
    .string()
    .min(2, 'Nome deve ter no mínimo 2 caracteres')
    .max(100, 'Nome deve ter no máximo 100 caracteres'),

  email: z
    .string()
    .email('E-mail inválido')
    .toLowerCase(),

  password: z
    .string()
    .min(8, 'Senha deve ter no mínimo 8 caracteres')
    .regex(/[A-Z]/, 'Senha deve conter ao menos uma letra maiúscula')
    .regex(/[0-9]/, 'Senha deve conter ao menos um número'),

  role: z.enum(['admin', 'user']).default('user'),

  birthDate: z
    .string()
    .date('Data inválida (esperado YYYY-MM-DD)')
    .optional(),
});

export const updateUserSchema = createUserSchema.partial();

// Inferência de tipos — evita duplicação
export type CreateUserInput = z.infer<typeof createUserSchema>;
// Resultado: { name: string; email: string; password: string; role: 'admin' | 'user'; birthDate?: string }

export type UpdateUserInput = z.infer<typeof updateUserSchema>;
// Resultado: Partial<CreateUserInput>
```

**Middleware de validação genérico:**

```ts
// src/middlewares/validation.middleware.ts
import type { Request, Response, NextFunction } from 'express';
import type { ZodSchema } from 'zod';

export function validate(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      const errors = result.error.issues.map((issue) => ({
        field: issue.path.join('.'),
        message: issue.message,
      }));
      res.status(400).json({ errors });
      return;
    }

    req.body = result.data;
    next();
  };
}
```

**Resposta de erro de validação:**

```json
{
  "errors": [
    { "field": "email", "message": "E-mail inválido" },
    { "field": "password", "message": "Senha deve ter no mínimo 8 caracteres" }
  ]
}
```

#### Schemas Complexos com Zod

```ts
// Schema de endereço (reutilizável)
const addressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  state: z.string().length(2),
  zipCode: z.string().regex(/^\d{5}-?\d{3}$/, 'CEP inválido'),
});

// Schema de produto com validações cruzadas
const createProductSchema = z
  .object({
    name: z.string().min(1).max(200),
    price: z.number().positive('Preço deve ser positivo'),
    discountPrice: z.number().positive().optional(),
    category: z.enum(['electronics', 'clothing', 'food']),
    tags: z.array(z.string()).max(10).default([]),
    metadata: z.record(z.string(), z.unknown()).optional(),
    address: addressSchema.optional(),
  })
  .refine(
    (data) => !data.discountPrice || data.discountPrice < data.price,
    { message: 'Preço com desconto deve ser menor que o preço original', path: ['discountPrice'] },
  );

// Schema de query params
const paginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
  sort: z.enum(['name', 'price', 'createdAt']).default('createdAt'),
  order: z.enum(['asc', 'desc']).default('desc'),
});
```

### class-validator (NestJS)

NestJS usa o `class-validator` com decorators, semelhante ao Bean Validation do Java/Spring.

```bash
npm install class-validator class-transformer
```

```ts
// src/users/dto/create-user.dto.ts
import {
  IsString,
  IsEmail,
  IsEnum,
  IsOptional,
  MinLength,
  MaxLength,
  Matches,
  IsDateString,
} from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2, { message: 'Nome deve ter no mínimo 2 caracteres' })
  @MaxLength(100)
  name: string;

  @IsEmail({}, { message: 'E-mail inválido' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Senha deve ter no mínimo 8 caracteres' })
  @Matches(/[A-Z]/, { message: 'Senha deve conter ao menos uma letra maiúscula' })
  @Matches(/[0-9]/, { message: 'Senha deve conter ao menos um número' })
  password: string;

  @IsEnum(['admin', 'user'])
  @IsOptional()
  role?: 'admin' | 'user' = 'user';

  @IsDateString()
  @IsOptional()
  birthDate?: string;
}
```

```ts
// src/users/dto/update-user.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

**Habilitar validação global em `main.ts`:**

```ts
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,       // remove propriedades não declaradas no DTO
      forbidNonWhitelisted: true, // rejeita propriedades extras
      transform: true,       // converte tipos automaticamente (string → number)
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

> **Comparação com Spring Boot:** `@IsEmail()` equivale a `@Email`, `@MinLength(2)` a `@Size(min = 2)`, `@IsNotEmpty()` a `@NotBlank`. O `ValidationPipe` do NestJS faz o papel do `@Valid` do Spring + Bean Validation.

### Validação Customizada

#### Express com Zod — Validação com lógica assíncrona

```ts
const createUserSchema = z.object({
  email: z.string().email(),
}).superRefine(async (data, ctx) => {
  const exists = await userRepository.existsByEmail(data.email);
  if (exists) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: 'E-mail já cadastrado',
      path: ['email'],
    });
  }
});

// Validação assíncrona no middleware
export function validateAsync(schema: ZodSchema) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const result = await schema.safeParseAsync(req.body);
    if (!result.success) {
      const errors = result.error.issues.map((i) => ({
        field: i.path.join('.'),
        message: i.message,
      }));
      res.status(400).json({ errors });
      return;
    }
    req.body = result.data;
    next();
  };
}
```

#### NestJS — Decorator Customizado

```ts
import {
  registerDecorator,
  ValidationOptions,
  ValidatorConstraint,
  ValidatorConstraintInterface,
} from 'class-validator';
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users.service';

@ValidatorConstraint({ async: true })
@Injectable()
export class IsUniqueEmailConstraint implements ValidatorConstraintInterface {
  constructor(private readonly usersService: UsersService) {}

  async validate(email: string): Promise<boolean> {
    const user = await this.usersService.findByEmail(email);
    return !user;
  }

  defaultMessage(): string {
    return 'E-mail já cadastrado';
  }
}

export function IsUniqueEmail(options?: ValidationOptions) {
  return function (object: object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName,
      options,
      constraints: [],
      validator: IsUniqueEmailConstraint,
    });
  };
}

// Uso no DTO
export class CreateUserDto {
  @IsEmail()
  @IsUniqueEmail()
  email: string;
}
```

---

## 8. Integração com Banco de Dados

### Prisma ORM

Prisma é o ORM mais popular para TypeScript — oferece schema declarativo, migrations automáticas, client type-safe gerado e excelente DX.

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

#### Schema do Prisma

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"     // ou mysql, sqlite, mongodb
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String   @db.VarChar(100)
  email     String   @unique
  password  String
  role      Role     @default(USER)
  posts     Post[]
  profile   Profile?
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")
}

model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  avatar String?
  userId Int     @unique @map("user_id")
  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("profiles")
}

model Post {
  id         Int        @id @default(autoincrement())
  title      String     @db.VarChar(200)
  content    String     @db.Text
  published  Boolean    @default(false)
  authorId   Int        @map("author_id")
  author     User       @relation(fields: [authorId], references: [id])
  categories Category[]
  createdAt  DateTime   @default(now()) @map("created_at")
  updatedAt  DateTime   @updatedAt @map("updated_at")

  @@index([authorId])
  @@map("posts")
}

model Category {
  id    Int    @id @default(autoincrement())
  name  String @unique @db.VarChar(50)
  posts Post[]

  @@map("categories")
}

enum Role {
  ADMIN
  USER
}
```

#### Configuração do Client

**Express:**

```ts
// src/config/database.ts
import { PrismaClient } from '@prisma/client';

export const prisma = new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'warn', 'error'] : ['error'],
});
```

**NestJS (como módulo injetável):**

```ts
// src/prisma/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  constructor() {
    super({
      log: process.env.NODE_ENV === 'development' ? ['query', 'warn', 'error'] : ['error'],
    });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

```ts
// src/prisma/prisma.module.ts
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

#### CRUD e Queries com Prisma

```ts
// Criar com relacionamento
const user = await prisma.user.create({
  data: {
    name: 'João',
    email: 'joao@email.com',
    password: hashedPassword,
    profile: {
      create: { bio: 'Desenvolvedor TypeScript' },
    },
  },
  include: { profile: true },
});

// Buscar com filtros, paginação e relacionamentos
const posts = await prisma.post.findMany({
  where: {
    published: true,
    author: { role: 'USER' },
    title: { contains: 'TypeScript', mode: 'insensitive' },
  },
  include: {
    author: { select: { id: true, name: true, email: true } },
    categories: true,
  },
  orderBy: { createdAt: 'desc' },
  skip: 0,
  take: 10,
});

// Atualizar
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'João Silva' },
});

// Upsert — cria ou atualiza
const category = await prisma.category.upsert({
  where: { name: 'TypeScript' },
  update: {},
  create: { name: 'TypeScript' },
});

// Deletar em cascata (configurado no schema)
await prisma.user.delete({ where: { id: 1 } });

// Transação
const [post, updatedUser] = await prisma.$transaction([
  prisma.post.create({ data: { title: 'Novo Post', content: '...', authorId: 1 } }),
  prisma.user.update({ where: { id: 1 }, data: { name: 'Atualizado' } }),
]);

// Transação interativa
await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUnique({ where: { id: 1 } });
  if (!user) throw new Error('Usuário não encontrado');

  await tx.post.create({
    data: { title: 'Post do usuário', content: '...', authorId: user.id },
  });
});
```

### TypeORM

TypeORM é um ORM maduro inspirado no Hibernate/JPA, usando decorators para definir entidades. É a escolha padrão do NestJS quando se prefere o padrão Active Record ou Data Mapper.

```bash
npm install typeorm reflect-metadata pg    # ou mysql2, better-sqlite3
npm install -D @types/node
```

#### Entidades

```ts
// src/users/entities/user.entity.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
  OneToOne,
} from 'typeorm';
import { Post } from '../../posts/entities/post.entity';
import { Profile } from './profile.entity';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'varchar', length: 100 })
  name: string;

  @Column({ type: 'varchar', unique: true })
  email: string;

  @Column({ type: 'varchar', select: false })
  password: string;

  @Column({ type: 'enum', enum: ['admin', 'user'], default: 'user' })
  role: 'admin' | 'user';

  @OneToOne(() => Profile, (profile) => profile.user, { cascade: true })
  profile: Profile;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

#### Repository Pattern com TypeORM no NestJS

```ts
// src/users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}

  async findAll(page: number, limit: number) {
    const [data, total] = await this.userRepo.findAndCount({
      relations: ['profile'],
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return { data, meta: { total, page, limit } };
  }

  async findOne(id: number) {
    const user = await this.userRepo.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
    if (!user) throw new NotFoundException(`Usuário #${id} não encontrado`);
    return user;
  }

  async create(dto: CreateUserDto) {
    const user = this.userRepo.create(dto);
    return this.userRepo.save(user);
  }
}
```

> **Comparação com Spring:** `@Entity`, `@Column`, `@OneToMany` do TypeORM são equivalentes diretos das anotações JPA. `@InjectRepository(User)` funciona como `@Autowired UserRepository` no Spring Data JPA.

### Drizzle ORM

Drizzle é um ORM leve, SQL-first e totalmente type-safe — os schemas são definidos em TypeScript puro, sem decorators nem geração de código.

```bash
npm install drizzle-orm pg
npm install -D drizzle-kit @types/pg
```

```ts
// src/db/schema.ts
import { pgTable, serial, varchar, text, boolean, integer, timestamp, pgEnum } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const roleEnum = pgEnum('role', ['admin', 'user']);

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: varchar('name', { length: 100 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  password: varchar('password', { length: 255 }).notNull(),
  role: roleEnum('role').default('user').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: varchar('title', { length: 200 }).notNull(),
  content: text('content').notNull(),
  published: boolean('published').default(false).notNull(),
  authorId: integer('author_id').references(() => users.id).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));
```

```ts
// Queries type-safe com Drizzle
import { db } from './db';
import { users, posts } from './db/schema';
import { eq, like, and, desc } from 'drizzle-orm';

// Select com filtros
const result = await db
  .select()
  .from(users)
  .where(and(
    eq(users.role, 'user'),
    like(users.name, '%João%'),
  ))
  .orderBy(desc(users.createdAt))
  .limit(10);

// Insert
const [newUser] = await db
  .insert(users)
  .values({ name: 'Maria', email: 'maria@email.com', password: hash })
  .returning();

// Update
await db
  .update(users)
  .set({ name: 'Maria Silva' })
  .where(eq(users.id, 1));

// Query com relacionamento (relational query API)
const usersWithPosts = await db.query.users.findMany({
  with: { posts: true },
  where: eq(users.role, 'admin'),
});
```

### Comparação entre ORMs

| Aspecto | Prisma | TypeORM | Drizzle |
|---|---|---|---|
| **Abordagem** | Schema DSL (`.prisma`) | Decorators em classes | TypeScript puro (tabelas como objetos) |
| **Type safety** | Client gerado, 100% tipado | Parcial (generics no Repository) | 100% tipado, inferido do schema |
| **Migrations** | `prisma migrate` automático | Geração automática ou manual | `drizzle-kit` gera SQL |
| **SQL bruto** | `$queryRaw` com template literal | `query()` ou QueryBuilder | `db.execute(sql\`...\`)` |
| **Curva de aprendizado** | Baixa | Média (semelhante ao JPA) | Baixa (próximo do SQL) |
| **Performance** | Boa (query engine em Rust) | Boa | Excelente (gera SQL direto) |
| **Uso recomendado** | Projetos novos, prototipagem rápida | Quem vem do Java/JPA, projetos NestJS | Projetos que valorizam controle do SQL |

### Migrations

#### Prisma

```bash
# Criar migration a partir de mudanças no schema.prisma
npx prisma migrate dev --name add_user_role

# Aplicar migrations em produção
npx prisma migrate deploy

# Resetar banco (desenvolvimento)
npx prisma migrate reset

# Visualizar dados no browser
npx prisma studio
```

#### TypeORM

```bash
# Gerar migration a partir de mudanças nas entidades
npx typeorm migration:generate -d src/data-source.ts src/migrations/AddUserRole

# Executar migrations
npx typeorm migration:run -d src/data-source.ts

# Reverter última migration
npx typeorm migration:revert -d src/data-source.ts
```

#### Drizzle

```bash
# Gerar migration
npx drizzle-kit generate

# Aplicar migrations
npx drizzle-kit migrate

# Interface visual
npx drizzle-kit studio
```

---

## 9. Autenticação e Autorização

### JWT com Express

```bash
npm install jsonwebtoken bcryptjs
npm install -D @types/jsonwebtoken @types/bcryptjs
```

```ts
// src/services/auth.service.ts
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { prisma } from '../config/database';
import { AppError } from '../middlewares/error-handler.middleware';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_EXPIRES_IN = '24h';

interface TokenPayload {
  userId: number;
  role: string;
}

export async function register(data: { name: string; email: string; password: string }) {
  const exists = await prisma.user.findUnique({ where: { email: data.email } });
  if (exists) throw new AppError(409, 'E-mail já cadastrado');

  const hashedPassword = await bcrypt.hash(data.password, 12);

  const user = await prisma.user.create({
    data: { ...data, password: hashedPassword },
    select: { id: true, name: true, email: true, role: true },
  });

  const token = generateToken({ userId: user.id, role: user.role });

  return { user, token };
}

export async function login(email: string, password: string) {
  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) throw new AppError(401, 'Credenciais inválidas');

  const passwordMatch = await bcrypt.compare(password, user.password);
  if (!passwordMatch) throw new AppError(401, 'Credenciais inválidas');

  const token = generateToken({ userId: user.id, role: user.role });

  return { user: { id: user.id, name: user.name, email: user.email, role: user.role }, token };
}

function generateToken(payload: TokenPayload): string {
  return jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
}
```

### JWT com NestJS (@nestjs/jwt)

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcryptjs
npm install -D @types/passport-jwt @types/bcryptjs
```

```ts
// src/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './jwt.strategy';

@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '24h' },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
})
export class AuthModule {}
```

```ts
// src/auth/jwt.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

interface JwtPayload {
  userId: number;
  role: string;
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  validate(payload: JwtPayload) {
    return { userId: payload.userId, role: payload.role };
  }
}
```

```ts
// Guard de autorização por role
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.includes(user.role);
  }
}

// Uso no controller
@Delete(':id')
@UseGuards(AuthGuard('jwt'), RolesGuard)
@Roles('admin')
remove(@Param('id', ParseIntPipe) id: number) {
  return this.usersService.remove(id);
}
```

---

## 10. Testes

### Vitest (recomendado para projetos TypeScript)

```bash
npm install -D vitest
```

#### Testes Unitários

```ts
// tests/unit/user.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UsersService } from '../../src/services/user.service';

const mockPrisma = {
  user: {
    findMany: vi.fn(),
    findUnique: vi.fn(),
    create: vi.fn(),
    update: vi.fn(),
    delete: vi.fn(),
    count: vi.fn(),
  },
};

vi.mock('../../src/config/database', () => ({
  prisma: mockPrisma,
}));

describe('UsersService', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('deve retornar usuário por ID', async () => {
    const mockUser = { id: 1, name: 'João', email: 'joao@email.com', role: 'user' };
    mockPrisma.user.findUnique.mockResolvedValue(mockUser);

    const result = await UsersService.findById(1);

    expect(result).toEqual(mockUser);
    expect(mockPrisma.user.findUnique).toHaveBeenCalledWith({ where: { id: 1 } });
  });

  it('deve lançar erro se usuário não existir', async () => {
    mockPrisma.user.findUnique.mockResolvedValue(null);

    await expect(UsersService.findById(999)).rejects.toThrow('Usuário não encontrado');
  });
});
```

#### Testes de Integração (API)

```bash
npm install -D supertest @types/supertest
```

```ts
// tests/integration/users.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { app } from '../../src/app';
import { prisma } from '../../src/config/database';

describe('Users API', () => {
  beforeAll(async () => {
    await prisma.$connect();
  });

  afterAll(async () => {
    await prisma.user.deleteMany();
    await prisma.$disconnect();
  });

  it('POST /api/users - deve criar usuário', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Maria', email: 'maria@test.com', password: 'Senha123!' })
      .expect(201);

    expect(response.body).toMatchObject({
      name: 'Maria',
      email: 'maria@test.com',
    });
    expect(response.body.password).toBeUndefined();
  });

  it('POST /api/users - deve rejeitar e-mail inválido', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Maria', email: 'invalido', password: 'Senha123!' })
      .expect(400);

    expect(response.body.errors).toContainEqual(
      expect.objectContaining({ field: 'email' }),
    );
  });

  it('GET /api/users/:id - deve retornar 404 para ID inexistente', async () => {
    await request(app)
      .get('/api/users/99999')
      .expect(404);
  });
});
```

### Testes no NestJS

```ts
// src/users/users.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  const mockService = {
    findAll: vi.fn(),
    findOne: vi.fn(),
    create: vi.fn(),
    update: vi.fn(),
    remove: vi.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [{ provide: UsersService, useValue: mockService }],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('deve retornar lista de usuários', async () => {
    const expected = { data: [{ id: 1, name: 'João' }], meta: { total: 1, page: 1, limit: 10 } };
    mockService.findAll.mockResolvedValue(expected);

    const result = await controller.findAll(1, 10);

    expect(result).toEqual(expected);
    expect(service.findAll).toHaveBeenCalledWith({ page: 1, limit: 10, role: undefined });
  });
});
```

---

## 11. Deploy e Boas Práticas

### Variáveis de Ambiente

```bash
npm install dotenv    # para Express (NestJS tem @nestjs/config)
```

```ts
// src/config/env.ts — validação com Zod
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  CORS_ORIGIN: z.string().default('http://localhost:5173'),
});

export const env = envSchema.parse(process.env);
```

### Dockerfile para Produção

```dockerfile
# Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate && npm run build

# Produção
FROM node:22-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY --from=builder /app/prisma ./prisma

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Boas Práticas de Segurança

```bash
npm install helmet cors express-rate-limit
```

```ts
import helmet from 'helmet';
import cors from 'cors';
import rateLimit from 'express-rate-limit';

// Segurança — headers HTTP
app.use(helmet());

// CORS
app.use(cors({
  origin: env.CORS_ORIGIN,
  credentials: true,
}));

// Rate limiting
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 100,                  // máximo 100 requisições por IP
  standardHeaders: true,
  legacyHeaders: false,
}));
```

### Checklist de Produção

| Item | Express | NestJS |
|---|---|---|
| **Validação de entrada** | Zod + middleware | class-validator + ValidationPipe |
| **Tratamento de erros** | Error handler middleware | Exception Filters |
| **Logs estruturados** | pino / winston | @nestjs/common Logger ou pino |
| **Variáveis de ambiente** | dotenv + Zod | @nestjs/config + Joi/Zod |
| **CORS** | cors middleware | `app.enableCors()` |
| **Rate limiting** | express-rate-limit | @nestjs/throttler |
| **Helmet** | helmet middleware | helmet middleware |
| **Health check** | Rota manual `/health` | @nestjs/terminus |
| **Graceful shutdown** | `process.on('SIGTERM')` | `app.enableShutdownHooks()` |
| **Compressão** | compression middleware | compression middleware |

### Express vs. NestJS — Quando Escolher

| Critério | Express | NestJS |
|---|---|---|
| **Projeto** | APIs simples, microsserviços leves, BFFs | APIs complexas, aplicações enterprise |
| **Equipe** | Desenvolvedores experientes que sabem montar arquitetura | Equipes que preferem convenções prontas |
| **Flexibilidade** | Total — você decide cada padrão | Limitada pelo framework (mas extensível) |
| **Curva de aprendizado** | Baixa (Express é minimal) | Média-alta (decorators, DI, módulos) |
| **Boilerplate** | Mínimo | Mais código inicial, mas mais organizado |
| **Ecossistema** | Maior (Express é o mais popular do Node.js) | Oficial e integrado (Swagger, GraphQL, WebSockets, microservices) |
| **Familiar para** | Desenvolvedores JavaScript/Node.js | Desenvolvedores Angular, Java/Spring Boot |

---

## 12. Referências

| Recurso | Link |
|---|---|
| **Node.js** | [https://nodejs.org/docs/latest/api/](https://nodejs.org/docs/latest/api/) |
| **TypeScript** | [https://www.typescriptlang.org/docs/](https://www.typescriptlang.org/docs/) |
| **Express.js** | [https://expressjs.com/](https://expressjs.com/) |
| **NestJS** | [https://docs.nestjs.com/](https://docs.nestjs.com/) |
| **Prisma** | [https://www.prisma.io/docs](https://www.prisma.io/docs) |
| **TypeORM** | [https://typeorm.io/](https://typeorm.io/) |
| **Drizzle ORM** | [https://orm.drizzle.team/](https://orm.drizzle.team/) |
| **Zod** | [https://zod.dev/](https://zod.dev/) |
| **class-validator** | [https://github.com/typestack/class-validator](https://github.com/typestack/class-validator) |
| **Vitest** | [https://vitest.dev/](https://vitest.dev/) |
| **tsx** | [https://tsx.is/](https://tsx.is/) |
