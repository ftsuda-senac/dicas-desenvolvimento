# Testes para Frontend — JavaScript, TypeScript e React — Guia Abrangente

> **Objetivo:** Cobrir de forma prática e didática todas as camadas de teste em aplicações frontend — desde testes unitários com Vitest e Jest até testes end-to-end com Cypress e Playwright — abrangendo projetos HTML/CSS/JavaScript puros e aplicações React, com exemplos de código, configurações e boas práticas para cada cenário.

---

## Sumário

1. [Fundamentos e Pirâmide de Testes no Frontend](#1-fundamentos-e-pirâmide-de-testes-no-frontend)
2. [Ambiente e Ferramentas — Visão Geral](#2-ambiente-e-ferramentas--visão-geral)
3. [Vitest — Configuração e Fundamentos](#3-vitest--configuração-e-fundamentos)
4. [Jest — Configuração e Fundamentos](#4-jest--configuração-e-fundamentos)
5. [Mocking e Spies](#5-mocking-e-spies)
6. [Testando DOM com JSDOM e Happy DOM](#6-testando-dom-com-jsdom-e-happy-dom)
7. [Testes de Funções Utilitárias e Lógica Pura](#7-testes-de-funções-utilitárias-e-lógica-pura)
8. [Testes de Acessibilidade](#8-testes-de-acessibilidade)
9. [React Testing Library — Fundamentos](#9-react-testing-library--fundamentos)
10. [React Testing Library — Interações e Eventos](#10-react-testing-library--interações-e-eventos)
11. [Testando Componentes React — Cenários Comuns](#11-testando-componentes-react--cenários-comuns)
12. [Testando Hooks Customizados](#12-testando-hooks-customizados)
13. [Snapshot Testing](#13-snapshot-testing)
14. [Cypress — Testes E2E](#14-cypress--testes-e2e)
15. [Cypress — Component Testing](#15-cypress--component-testing)
16. [Playwright — Testes E2E](#16-playwright--testes-e2e)
17. [Comparativo Prático: Cypress vs Playwright](#17-comparativo-prático-cypress-vs-playwright)
18. [Testes Visuais (Visual Regression)](#18-testes-visuais-visual-regression)
19. [Testes de Performance e Qualidade com Lighthouse](#19-testes-de-performance-e-qualidade-com-lighthouse)
20. [Cobertura de Código e Métricas de Testes](#20-cobertura-de-código-e-métricas-de-testes)
21. [Organização, Boas Práticas e CI/CD](#21-organização-boas-práticas-e-cicd)
22. [MSW (Mock Service Worker) — Mocking de API na Camada de Rede](#22-msw-mock-service-worker--mocking-de-api-na-camada-de-rede)
23. [Testes com Storybook — Documentação Viva e Interaction Tests](#23-testes-com-storybook--documentação-viva-e-interaction-tests)
24. [Testes com TanStack Query (React Query)](#24-testes-com-tanstack-query-react-query)
25. [Testes de Segurança no Frontend](#25-testes-de-segurança-no-frontend)
26. [Testes de Responsividade e Media Queries](#26-testes-de-responsividade-e-media-queries)

---

## 1. Fundamentos e Pirâmide de Testes no Frontend

### 1.1 Por que testar o frontend?

Aplicações frontend modernas concentram lógica de negócio, gerenciamento de estado, validações, roteamento e interações complexas com o usuário. Testar o frontend garante que:

- **A interface funciona como o usuário espera** — cliques, digitação e navegação produzem o resultado correto.
- **Regressões são detectadas cedo** — uma mudança em um componente não quebra outros silenciosamente.
- **Refatorações são seguras** — testes dão confiança para reestruturar código sem medo.
- **Acessibilidade é mantida** — testes automatizados verificam que a aplicação continua acessível.
- **Integrações com APIs são confiáveis** — chamadas HTTP, tratamento de erros e estados de loading funcionam corretamente.

### 1.2 A Pirâmide de Testes no Frontend

A pirâmide de testes aplicada ao frontend mantém o mesmo princípio do backend: muitos testes rápidos na base, poucos testes lentos no topo.

```
              ╱╲
             ╱  ╲           E2E (Cypress, Playwright)
            ╱ E2E╲          Fluxos completos no browser real
           ╱──────╲
          ╱        ╲        Integração / Componente
         ╱Component ╲       (React Testing Library, Vitest)
        ╱────────────╲
       ╱              ╲     Unitários
      ╱  Unit Tests     ╲   (Vitest, Jest — lógica pura, utilitários)
     ╱────────────────────╲
```

| Camada | Quantidade | Velocidade | O que testa | Ferramentas |
|--------|-----------|------------|-------------|-------------|
| **Unitários** | ~60-70% | Milissegundos | Funções puras, validações, formatadores, hooks | Vitest, Jest |
| **Componente/Integração** | ~20-30% | Centenas de ms | Componentes renderizados, interações do usuário, integração entre componentes | React Testing Library + Vitest |
| **E2E** | ~5-10% | Segundos/Minutos | Fluxos completos no browser real (login, checkout, CRUD) | Cypress, Playwright |

### 1.3 Tipos de teste no contexto frontend

| Tipo | O que testa | Exemplo |
|------|------------|---------|
| **Unitário** | Função isolada sem dependências de DOM | Validar CPF, formatar moeda, calcular desconto |
| **Componente** | Um componente renderizado isoladamente | Botão desabilita após clique, formulário exibe erro |
| **Integração** | Múltiplos componentes interagindo | Página de listagem com filtro, busca e paginação |
| **Snapshot** | Estrutura HTML renderizada não mudou | Componente Card mantém a mesma estrutura |
| **Visual (Regression)** | Aparência visual não mudou | Screenshot do componente comparado pixel a pixel |
| **Acessibilidade** | Conformidade com WCAG/ARIA | Formulário tem labels, botões têm roles corretos |
| **E2E** | Fluxo completo no browser real | Usuário faz login, adiciona produto e finaliza compra |

### 1.4 Frontend puro vs React — o que muda nos testes?

| Aspecto | HTML/CSS/JS puro | React |
|---------|-------------------|-------|
| **Renderização** | Manipulação direta do DOM (`document.querySelector`) | Virtual DOM, componentes declarativos |
| **Ambiente de teste** | JSDOM simula o browser | React Testing Library abstrai a renderização |
| **Eventos** | `addEventListener`, `dispatchEvent` | `fireEvent`, `userEvent` |
| **Estado** | Variáveis globais, atributos `data-*` | `useState`, `useReducer`, Context, Redux |
| **Roteamento** | `window.location`, History API | React Router com `MemoryRouter` para testes |
| **Assertions** | Verificar `textContent`, `classList`, `value` | Queries semânticas: `getByRole`, `getByText` |

### 1.5 Vocabulário essencial

| Termo | Significado |
|-------|------------|
| **Test Runner** | Ferramenta que descobre, executa e reporta testes (Vitest, Jest, Playwright Test) |
| **Assertion Library** | API para verificar resultados (`expect(x).toBe(y)`) |
| **Mock** | Substituição de uma dependência por uma implementação controlada |
| **Stub** | Mock que retorna um valor fixo sem lógica |
| **Spy** | Observador que registra chamadas sem alterar o comportamento original |
| **Fixture** | Dados de exemplo reutilizáveis nos testes |
| **Snapshot** | Registro serializado da saída de um componente para comparação futura |
| **SUT** | *System Under Test* — o código sendo testado |
| **AAA** | *Arrange, Act, Assert* — padrão para estruturar cada teste |
| **Flaky Test** | Teste que passa ou falha de forma intermitente sem mudança no código |
| **Test Double** | Termo genérico para mock, stub, spy, fake ou dummy |
| **JSDOM** | Implementação JavaScript do DOM para rodar em Node.js sem browser |
| **Happy DOM** | Alternativa ao JSDOM, mais rápida e com mais APIs Web implementadas |

---

## 2. Ambiente e Ferramentas — Visão Geral

### 2.1 Ecossistema de ferramentas de teste frontend

```
┌─────────────────────────────────────────────────────────┐
│                   Testes Frontend                        │
├──────────────┬──────────────┬──────────────┬────────────┤
│   Unitários  │  Componente  │     E2E      │  Visuais   │
├──────────────┼──────────────┼──────────────┼────────────┤
│ Vitest       │ RTL + Vitest │ Cypress      │ Playwright │
│ Jest         │ Cypress CT   │ Playwright   │ Percy      │
│              │              │              │ Chromatic  │
├──────────────┴──────────────┴──────────────┴────────────┤
│              Ambiente simulado: JSDOM / Happy DOM        │
├─────────────────────────────────────────────────────────┤
│              Ambiente real: Chromium / Firefox / WebKit   │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Comparativo: Vitest vs Jest vs Cypress vs Playwright

| Característica | Vitest | Jest | Cypress | Playwright |
|---------------|--------|------|---------|------------|
| **Tipo principal** | Unit / Integração | Unit / Integração | E2E / Component | E2E |
| **Ambiente** | Node (JSDOM/Happy DOM) | Node (JSDOM) | Browser real (Chromium) | Browser real (Chromium, Firefox, WebKit) |
| **Velocidade** | Muito rápido | Rápido | Médio | Rápido |
| **ESM nativo** | Sim | Parcial (experimental) | N/A | N/A |
| **Config Vite** | Reutiliza `vite.config` | Configuração própria | Configuração própria | Configuração própria |
| **TypeScript** | Nativo (via Vite) | Requer ts-jest ou @swc/jest | Nativo | Nativo |
| **Watch mode** | HMR-based (instantâneo) | Polling/watchman | Dashboard interativo | Modo UI |
| **Snapshot** | Sim | Sim | Screenshot | Screenshot |
| **Mocking** | `vi.fn()`, `vi.mock()` | `jest.fn()`, `jest.mock()` | `cy.stub()`, `cy.intercept()` | `page.route()` |
| **Paralelismo** | Por arquivo (threads/forks) | Por arquivo (workers) | Limitado | Por arquivo (workers) |
| **Browser real** | Experimental (browser mode) | Não | Sim | Sim |
| **Custo** | Gratuito | Gratuito | Gratuito (Cloud pago) | Gratuito |
| **Comunidade** | Crescente | Muito grande | Grande | Crescente rápido |

### 2.3 Quando usar cada ferramenta

| Cenário | Ferramenta recomendada |
|---------|----------------------|
| Testar funções puras (validação, formatação, cálculo) | **Vitest** ou Jest |
| Testar componente React isolado | **Vitest** + React Testing Library |
| Testar interação entre múltiplos componentes | **Vitest** + React Testing Library |
| Testar hooks customizados | **Vitest** + `renderHook` |
| Testar fluxo E2E (login → dashboard → logout) | **Playwright** ou Cypress |
| Testar em múltiplos browsers | **Playwright** |
| Testar visualmente com dashboard interativo | **Cypress** |
| Testar componente no browser real | **Cypress Component Testing** |
| Visual regression (comparação de screenshots) | **Playwright** (built-in) |
| Testar acessibilidade automaticamente | **Vitest** + axe-core ou Playwright + axe |
| Projeto com Vite | **Vitest** (integração nativa) |
| Projeto legado com Webpack/CRA | **Jest** (já incluído) |

### 2.4 JSDOM vs Happy DOM vs Browser real

Testes unitários e de componente geralmente rodam em um ambiente **simulado** do DOM em Node.js, sem abrir um browser real. As duas opções principais são JSDOM e Happy DOM.

| Característica | JSDOM | Happy DOM | Browser real |
|---------------|-------|-----------|--------------|
| **O que é** | Implementação JS do DOM W3C | Alternativa mais rápida ao JSDOM | Chromium, Firefox ou WebKit real |
| **Velocidade** | Médio | Rápido (2-3x mais rápido) | Lento (startup + renderização) |
| **Fidelidade** | Boa (maioria das APIs Web) | Boa (algumas APIs faltantes) | Total |
| **CSS** | Não processa (apenas parsing) | Não processa | Processa completamente |
| **Layout** | Não calcula (`getBoundingClientRect` = 0) | Não calcula | Calcula |
| **`fetch`** | Disponível (Node 18+) | Disponível | Disponível |
| **Canvas** | Requer `canvas` npm | Suporte básico | Completo |
| **Web Workers** | Não | Parcial | Sim |
| **Quando usar** | Padrão para maioria dos projetos | Quando velocidade é prioridade | Quando fidelidade é crítica (CSS, layout) |

**Configuração no Vitest:**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    // Opção 1: JSDOM (padrão mais comum)
    environment: 'jsdom',

    // Opção 2: Happy DOM (mais rápido)
    // environment: 'happy-dom',
  },
});
```

**Configuração no Jest:**

```javascript
// jest.config.js
module.exports = {
  // JSDOM é o padrão no Jest
  testEnvironment: 'jsdom',
};
```

### 2.5 Estrutura de projeto com testes (visão geral)

```
meu-projeto/
├── src/
│   ├── components/
│   │   ├── Button.tsx
│   │   └── Button.test.tsx          ← teste de componente (co-located)
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── useAuth.test.ts          ← teste de hook
│   ├── utils/
│   │   ├── validators.ts
│   │   └── validators.test.ts       ← teste unitário
│   └── pages/
│       └── Login.tsx
├── tests/                            ← ou __tests__/
│   └── e2e/
│       ├── login.spec.ts             ← teste E2E (Playwright/Cypress)
│       └── checkout.spec.ts
├── cypress/                          ← (se usar Cypress)
│   ├── e2e/
│   ├── fixtures/
│   └── support/
├── vitest.config.ts                  ← (ou jest.config.ts)
├── playwright.config.ts              ← (se usar Playwright)
└── package.json
```

---

## 3. Vitest — Configuração e Fundamentos

### 3.1 O que é o Vitest?

Vitest é um framework de testes ultrarrápido construído sobre o Vite. Ele reutiliza a configuração, plugins e pipeline de transformação do Vite, oferecendo execução de testes com HMR (Hot Module Replacement) para feedback instantâneo durante o desenvolvimento.

**Principais vantagens:**

- Compatível com a API do Jest (migração simples)
- ESM nativo — suporte completo a `import`/`export` sem transpilação extra
- TypeScript nativo — sem necessidade de `ts-jest` ou configurações adicionais
- Watch mode baseado em HMR — re-executa apenas testes afetados em milissegundos
- Reutiliza `vite.config.ts` — aliases, plugins e configurações compartilhadas

### 3.2 Instalação e configuração

```bash
# Instalar Vitest e ambiente de DOM
npm install -D vitest @vitest/coverage-v8 jsdom

# Para projetos React — adicionar testing-library
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

**Configuração básica (`vitest.config.ts`):**

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    // Ambiente simulado do DOM
    environment: 'jsdom',

    // Arquivos de setup executados antes de cada arquivo de teste
    setupFiles: ['./src/test/setup.ts'],

    // Padrão de arquivos de teste
    include: ['src/**/*.{test,spec}.{ts,tsx}'],

    // Habilitar globals (describe, it, expect sem import)
    globals: true,

    // Configuração de cobertura
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['src/**/*.test.{ts,tsx}', 'src/**/*.spec.{ts,tsx}', 'src/test/**'],
    },
  },
});
```

**Para projetos que já usam `vite.config.ts`:**

```typescript
// vite.config.ts — a configuração de test pode ficar junto
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
  },
});
```

**Arquivo de setup (`src/test/setup.ts`):**

```typescript
import '@testing-library/jest-dom/vitest';

// Extensões de matchers como toBeInTheDocument(), toHaveTextContent(), etc.
// O import acima registra automaticamente os matchers do jest-dom no Vitest.
```

**Scripts no `package.json`:**

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

### 3.3 Estrutura de testes: `describe`, `it`/`test`, `expect`

```typescript
// src/utils/math.ts
export function somar(a: number, b: number): number {
  return a + b;
}

export function dividir(a: number, b: number): number {
  if (b === 0) throw new Error('Divisão por zero');
  return a / b;
}
```

```typescript
// src/utils/math.test.ts
import { describe, it, expect } from 'vitest';
import { somar, dividir } from './math';

// describe agrupa testes relacionados
describe('somar', () => {
  // it (ou test) define um caso de teste individual
  it('deve somar dois números positivos', () => {
    expect(somar(2, 3)).toBe(5);
  });

  it('deve somar números negativos', () => {
    expect(somar(-1, -2)).toBe(-3);
  });

  it('deve retornar o mesmo número ao somar com zero', () => {
    expect(somar(5, 0)).toBe(5);
  });
});

describe('dividir', () => {
  it('deve dividir dois números', () => {
    expect(dividir(10, 2)).toBe(5);
  });

  it('deve retornar decimal quando necessário', () => {
    expect(dividir(7, 2)).toBe(3.5);
  });

  it('deve lançar erro ao dividir por zero', () => {
    expect(() => dividir(10, 0)).toThrow('Divisão por zero');
  });
});
```

> **Dica:** `it` e `test` são sinônimos — use o que preferir. A convenção mais comum é `it('deve...')` ou `test('should...')`.

### 3.4 Matchers principais

#### Igualdade

```typescript
// toBe — igualdade estrita (===), ideal para primitivos
expect(1 + 1).toBe(2);
expect('hello').toBe('hello');
expect(true).toBe(true);

// toEqual — igualdade profunda, ideal para objetos e arrays
expect({ nome: 'Ana', idade: 25 }).toEqual({ nome: 'Ana', idade: 25 });
expect([1, 2, 3]).toEqual([1, 2, 3]);

// toStrictEqual — como toEqual, mas verifica propriedades undefined e tipos de classe
expect({ a: 1 }).toStrictEqual({ a: 1 });
// Falha: { a: 1, b: undefined } !== { a: 1 }
```

#### Veracidade (truthiness)

```typescript
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect(valor).toBeDefined();
expect(1).toBeTruthy();
expect(0).toBeFalsy();
```

#### Números

```typescript
expect(10).toBeGreaterThan(5);
expect(10).toBeGreaterThanOrEqual(10);
expect(5).toBeLessThan(10);
expect(5).toBeLessThanOrEqual(5);

// Para ponto flutuante — evitar toBe por imprecisão
expect(0.1 + 0.2).toBeCloseTo(0.3, 5); // 5 casas decimais
```

#### Strings

```typescript
expect('Hello World').toContain('World');
expect('Hello World').toMatch(/hello/i);     // regex
expect('Hello World').toHaveLength(11);
```

#### Arrays e iteráveis

```typescript
expect([1, 2, 3]).toContain(2);
expect([{ id: 1 }, { id: 2 }]).toContainEqual({ id: 1 });
expect([1, 2, 3]).toHaveLength(3);

// Verificar que array contém pelo menos esses elementos (em qualquer ordem)
expect([1, 2, 3, 4]).toEqual(expect.arrayContaining([3, 1]));
```

#### Objetos

```typescript
expect({ a: 1, b: 2, c: 3 }).toMatchObject({ a: 1, b: 2 });
expect({ name: 'Ana' }).toHaveProperty('name');
expect({ name: 'Ana' }).toHaveProperty('name', 'Ana');

// Verificar que objeto contém pelo menos essas chaves
expect({ a: 1, b: 2 }).toEqual(expect.objectContaining({ a: 1 }));
```

#### Exceções

```typescript
// Verificar que lança erro (a função deve ser passada como callback)
expect(() => funcaoQueFalha()).toThrow();
expect(() => funcaoQueFalha()).toThrow('mensagem específica');
expect(() => funcaoQueFalha()).toThrow(TypeError);
expect(() => funcaoQueFalha()).toThrow(/regex/);

// Para funções async
await expect(funcaoAsyncQueFalha()).rejects.toThrow('erro');
```

#### Negação com `.not`

```typescript
expect(1).not.toBe(2);
expect([1, 2]).not.toContain(3);
expect({ a: 1 }).not.toEqual({ a: 2 });
```

### 3.5 Setup e teardown

```typescript
import { describe, it, expect, beforeEach, afterEach, beforeAll, afterAll } from 'vitest';

describe('CarrinhoDeCompras', () => {
  let carrinho: string[];

  // Executa UMA vez antes de TODOS os testes do bloco
  beforeAll(() => {
    console.log('Inicializando recursos compartilhados...');
  });

  // Executa antes de CADA teste
  beforeEach(() => {
    carrinho = []; // estado limpo para cada teste
  });

  // Executa depois de CADA teste
  afterEach(() => {
    carrinho = [];
  });

  // Executa UMA vez depois de TODOS os testes do bloco
  afterAll(() => {
    console.log('Limpando recursos...');
  });

  it('deve adicionar item ao carrinho', () => {
    carrinho.push('Notebook');
    expect(carrinho).toHaveLength(1);
  });

  it('deve iniciar vazio (graças ao beforeEach)', () => {
    expect(carrinho).toHaveLength(0);
  });
});
```

### 3.6 Testes parametrizados com `it.each`

```typescript
import { describe, it, expect } from 'vitest';
import { classificarNota } from './notas';

describe('classificarNota', () => {
  it.each([
    { nota: 10, esperado: 'A' },
    { nota: 8, esperado: 'B' },
    { nota: 6, esperado: 'C' },
    { nota: 4, esperado: 'D' },
    { nota: 2, esperado: 'F' },
  ])('deve retornar "$esperado" para nota $nota', ({ nota, esperado }) => {
    expect(classificarNota(nota)).toBe(esperado);
  });
});

// Formato alternativo com arrays
describe('somar', () => {
  it.each([
    [1, 2, 3],
    [0, 0, 0],
    [-1, 1, 0],
    [100, 200, 300],
  ])('somar(%i, %i) deve retornar %i', (a, b, resultado) => {
    expect(somar(a, b)).toBe(resultado);
  });
});
```

### 3.7 Testes assíncronos

```typescript
import { describe, it, expect } from 'vitest';

// Opção 1: async/await (recomendado)
it('deve buscar dados do usuário', async () => {
  const usuario = await buscarUsuario(1);
  expect(usuario.nome).toBe('Ana');
});

// Opção 2: retornar a Promise
it('deve buscar dados do usuário', () => {
  return buscarUsuario(1).then((usuario) => {
    expect(usuario.nome).toBe('Ana');
  });
});

// Testando rejeição de Promise
it('deve rejeitar para ID inválido', async () => {
  await expect(buscarUsuario(-1)).rejects.toThrow('ID inválido');
});
```

### 3.8 Modos de execução

```bash
# Watch mode (padrão) — re-executa testes afetados ao salvar arquivos
npx vitest

# Run mode — executa uma vez e finaliza (ideal para CI)
npx vitest run

# Executar apenas testes que correspondem a um padrão
npx vitest run math
npx vitest run --grep "deve somar"

# Executar um arquivo específico
npx vitest run src/utils/math.test.ts

# Com cobertura de código
npx vitest run --coverage

# Interface gráfica no browser
npx vitest --ui
```

### 3.9 Configuração de cobertura com `@vitest/coverage-v8`

```bash
npm install -D @vitest/coverage-v8
```

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov', 'json-summary'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: [
        'src/**/*.test.{ts,tsx}',
        'src/**/*.spec.{ts,tsx}',
        'src/test/**',
        'src/**/*.d.ts',
        'src/main.tsx',
      ],
      // Thresholds mínimos de cobertura
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
  },
});
```

Saída do relatório de cobertura no terminal:

```
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |   92.3  |    87.5  |   90.0  |   92.3  |
 math.ts  |  100.0  |   100.0  |  100.0  |  100.0  |
 utils.ts |   85.7  |    75.0  |   80.0  |   85.7  | 23-28
----------|---------|----------|---------|---------|-------------------
```

---

## 4. Jest — Configuração e Fundamentos

### 4.1 O que é o Jest?

Jest é o framework de testes mais popular do ecossistema JavaScript, criado pelo Facebook. Por muitos anos foi o padrão de fato para projetos React (incluído no Create React App). Embora o Vitest esteja ganhando adoção rapidamente, o Jest continua amplamente usado, especialmente em projetos existentes.

### 4.2 Instalação e configuração

```bash
# Para projetos TypeScript com Jest
npm install -D jest @types/jest ts-jest

# Ou usando SWC (mais rápido que ts-jest)
npm install -D jest @types/jest @swc/core @swc/jest

# Para testes de DOM/React
npm install -D jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

**Configuração com ts-jest (`jest.config.ts`):**

```typescript
import type { Config } from 'jest';

const config: Config = {
  // Ambiente de DOM simulado
  testEnvironment: 'jsdom',

  // Transformação TypeScript
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },

  // Padrão de arquivos de teste
  testMatch: ['<rootDir>/src/**/*.{test,spec}.{ts,tsx}'],

  // Setup files
  setupFilesAfterSetup: ['<rootDir>/src/test/setup.ts'],

  // Module aliases (correspondendo ao tsconfig.json paths)
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    // Mock de arquivos estáticos (CSS, imagens)
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/src/test/__mocks__/fileMock.ts',
  },
};

export default config;
```

**Configuração com SWC (mais rápido):**

```typescript
import type { Config } from 'jest';

const config: Config = {
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.tsx?$': ['@swc/jest', {
      jsc: {
        transform: {
          react: { runtime: 'automatic' },
        },
      },
    }],
  },
  testMatch: ['<rootDir>/src/**/*.{test,spec}.{ts,tsx}'],
  setupFilesAfterSetup: ['<rootDir>/src/test/setup.ts'],
};

export default config;
```

**Mock de arquivos estáticos (`src/test/__mocks__/fileMock.ts`):**

```typescript
export default 'file-stub';
```

**Scripts no `package.json`:**

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --maxWorkers=50%"
  }
}
```

### 4.3 Estrutura de testes (compatível com Vitest)

A API de testes do Jest é essencialmente idêntica à do Vitest:

```typescript
// src/utils/string.test.ts
import { capitalizar, slugify } from './string';

describe('capitalizar', () => {
  test('deve capitalizar a primeira letra', () => {
    expect(capitalizar('hello')).toBe('Hello');
  });

  test('deve retornar string vazia para entrada vazia', () => {
    expect(capitalizar('')).toBe('');
  });
});

describe('slugify', () => {
  test('deve converter espaços em hifens', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  test('deve remover caracteres especiais', () => {
    expect(slugify('Café & Pão!')).toBe('cafe-pao');
  });
});
```

> **Nota:** No Jest, `describe`, `it`, `test`, `expect` são globais por padrão — não precisam de import. No Vitest, eles são globais apenas se `globals: true` estiver configurado.

### 4.4 Diferenças práticas Vitest vs Jest

| Aspecto | Vitest | Jest |
|---------|--------|------|
| **Import de API** | `import { describe, it } from 'vitest'` (ou `globals: true`) | Globais por padrão |
| **ESM** | Nativo | Experimental (`--experimental-vm-modules`) |
| **Config** | `vitest.config.ts` (ou dentro de `vite.config.ts`) | `jest.config.ts` |
| **TypeScript** | Nativo via Vite | Requer `ts-jest` ou `@swc/jest` |
| **CSS/Assets** | Tratados pelo Vite automaticamente | Requer `moduleNameMapper` |
| **Aliases** | Herda do `vite.config` | Requer `moduleNameMapper` |
| **Mock de módulos** | `vi.mock()` | `jest.mock()` |
| **Mock de funções** | `vi.fn()` | `jest.fn()` |
| **Fake timers** | `vi.useFakeTimers()` | `jest.useFakeTimers()` |
| **Spy** | `vi.spyOn()` | `jest.spyOn()` |
| **Watch mode** | HMR-based (instantâneo) | Baseado em watchman/polling |
| **Velocidade típica** | Mais rápido (especialmente em projetos Vite) | Rápido |

**Exemplo de migração — mesma lógica, APIs diferentes:**

```typescript
// ========== Vitest ==========
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { buscarUsuarios } from './api';

vi.mock('./http-client');

describe('buscarUsuarios', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('deve retornar lista de usuários', async () => {
    const spy = vi.fn().mockResolvedValue([{ id: 1, nome: 'Ana' }]);
    // ...
  });
});
```

```typescript
// ========== Jest ==========
import { buscarUsuarios } from './api';

jest.mock('./http-client');

describe('buscarUsuarios', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('deve retornar lista de usuários', async () => {
    const spy = jest.fn().mockResolvedValue([{ id: 1, nome: 'Ana' }]);
    // ...
  });
});
```

### 4.5 Quando escolher Jest vs Vitest

| Cenário | Recomendação |
|---------|-------------|
| Projeto novo com Vite | **Vitest** — integração nativa, zero configuração extra |
| Projeto existente com CRA ou Webpack | **Jest** — já configurado, migração desnecessária |
| Projeto migrando de CRA para Vite | **Vitest** — aproveitar a migração para trocar |
| Equipe familiarizada apenas com Jest | **Jest** — a API é quase idêntica, não vale o custo de migração |
| Performance é prioridade | **Vitest** — HMR watch mode é significativamente mais rápido |
| Projeto com muitas configs Jest customizadas | **Jest** — migrar configurações complexas pode ser trabalhoso |

---

## 5. Mocking e Spies

Mocking é uma técnica essencial em testes que substitui dependências externas (APIs, módulos, timers) por implementações controladas, permitindo testar código isoladamente e de forma determinística.

> Os exemplos desta seção usam a API do **Vitest** (`vi.*`). Para Jest, substitua `vi` por `jest` — a API é idêntica.

### 5.1 Funções mock com `vi.fn()`

Uma função mock registra todas as chamadas (argumentos, retornos, quantidade) e permite configurar retornos controlados.

```typescript
import { describe, it, expect, vi } from 'vitest';

describe('vi.fn()', () => {
  it('deve registrar chamadas', () => {
    const callback = vi.fn();

    callback('arg1', 'arg2');
    callback('arg3');

    // Verificar que foi chamada
    expect(callback).toHaveBeenCalled();
    expect(callback).toHaveBeenCalledTimes(2);
    expect(callback).toHaveBeenCalledWith('arg1', 'arg2');
    expect(callback).toHaveBeenLastCalledWith('arg3');
  });

  it('deve configurar retorno', () => {
    const calcular = vi.fn()
      .mockReturnValue(42)               // retorno padrão
      .mockReturnValueOnce(10)            // retorno apenas na 1ª chamada
      .mockReturnValueOnce(20);           // retorno apenas na 2ª chamada

    expect(calcular()).toBe(10);  // 1ª chamada → mockReturnValueOnce
    expect(calcular()).toBe(20);  // 2ª chamada → mockReturnValueOnce
    expect(calcular()).toBe(42);  // 3ª chamada → mockReturnValue (padrão)
  });

  it('deve configurar implementação', () => {
    const somar = vi.fn((a: number, b: number) => a + b);

    expect(somar(2, 3)).toBe(5);
    expect(somar).toHaveBeenCalledWith(2, 3);
  });

  it('deve configurar retorno de Promise', async () => {
    const buscar = vi.fn()
      .mockResolvedValue({ id: 1, nome: 'Ana' })         // resolve
      .mockRejectedValueOnce(new Error('Falha na rede')); // rejeita na 1ª

    await expect(buscar()).rejects.toThrow('Falha na rede');
    await expect(buscar()).resolves.toEqual({ id: 1, nome: 'Ana' });
  });
});
```

#### Matchers para funções mock

```typescript
expect(fn).toHaveBeenCalled();                    // chamada ao menos 1 vez
expect(fn).toHaveBeenCalledTimes(3);              // chamada exatamente 3 vezes
expect(fn).toHaveBeenCalledWith('arg1', 'arg2');  // chamada com esses args
expect(fn).toHaveBeenLastCalledWith('ultimo');     // última chamada com esse arg
expect(fn).toHaveBeenNthCalledWith(2, 'segundo');  // 2ª chamada com esse arg
expect(fn).toHaveReturnedWith(42);                // retornou 42
expect(fn).not.toHaveBeenCalled();                // nunca foi chamada
```

### 5.2 Espionagem com `vi.spyOn()`

Um spy observa chamadas a um método existente sem substituir sua implementação (por padrão). Útil para verificar que um método foi chamado sem alterar seu comportamento.

```typescript
import { describe, it, expect, vi, afterEach } from 'vitest';

const calculadora = {
  somar(a: number, b: number): number {
    return a + b;
  },
  multiplicar(a: number, b: number): number {
    return a * b;
  },
};

describe('vi.spyOn()', () => {
  afterEach(() => {
    vi.restoreAllMocks(); // restaurar implementações originais
  });

  it('deve observar chamadas sem alterar o comportamento', () => {
    const spy = vi.spyOn(calculadora, 'somar');

    const resultado = calculadora.somar(2, 3);

    expect(resultado).toBe(5); // implementação original mantida
    expect(spy).toHaveBeenCalledWith(2, 3);
    expect(spy).toHaveBeenCalledTimes(1);
  });

  it('deve substituir a implementação quando desejado', () => {
    const spy = vi.spyOn(calculadora, 'multiplicar')
      .mockReturnValue(999);

    expect(calculadora.multiplicar(2, 3)).toBe(999); // implementação substituída
    expect(spy).toHaveBeenCalledWith(2, 3);
  });

  it('deve espionar console.log', () => {
    const spy = vi.spyOn(console, 'log').mockImplementation(() => {});

    console.log('teste');

    expect(spy).toHaveBeenCalledWith('teste');
  });
});
```

### 5.3 Mock de módulos com `vi.mock()`

Mock de módulos inteiros substitui todas as exportações de um arquivo ou pacote npm.

```typescript
// src/services/api.ts
import { httpClient } from './http-client';

export async function buscarProdutos(): Promise<Produto[]> {
  const response = await httpClient.get('/api/produtos');
  return response.data;
}
```

```typescript
// src/services/api.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { buscarProdutos } from './api';
import { httpClient } from './http-client';

// Mock do módulo inteiro — todas as exportações viram vi.fn()
vi.mock('./http-client');

describe('buscarProdutos', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('deve retornar lista de produtos', async () => {
    const produtosMock = [
      { id: 1, nome: 'Notebook', preco: 3500 },
      { id: 2, nome: 'Mouse', preco: 150 },
    ];

    // Configurar o retorno do mock
    vi.mocked(httpClient.get).mockResolvedValue({ data: produtosMock });

    const resultado = await buscarProdutos();

    expect(httpClient.get).toHaveBeenCalledWith('/api/produtos');
    expect(resultado).toEqual(produtosMock);
  });

  it('deve propagar erro da API', async () => {
    vi.mocked(httpClient.get).mockRejectedValue(new Error('Erro 500'));

    await expect(buscarProdutos()).rejects.toThrow('Erro 500');
  });
});
```

**Mock com implementação customizada (factory):**

```typescript
// Mock com factory — substitui o módulo por uma implementação customizada
vi.mock('./http-client', () => ({
  httpClient: {
    get: vi.fn(),
    post: vi.fn(),
    put: vi.fn(),
    delete: vi.fn(),
  },
}));
```

### 5.4 Mock de `fetch` e APIs externas

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

// Função que usa fetch diretamente
async function buscarUsuario(id: number) {
  const response = await fetch(`/api/usuarios/${id}`);
  if (!response.ok) throw new Error(`Erro: ${response.status}`);
  return response.json();
}

describe('buscarUsuario', () => {
  beforeEach(() => {
    // Substituir fetch global por um mock
    global.fetch = vi.fn();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('deve retornar dados do usuário', async () => {
    const mockResponse = { id: 1, nome: 'Ana', email: 'ana@email.com' };

    vi.mocked(fetch).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockResponse),
    } as Response);

    const usuario = await buscarUsuario(1);

    expect(fetch).toHaveBeenCalledWith('/api/usuarios/1');
    expect(usuario).toEqual(mockResponse);
  });

  it('deve lançar erro para resposta não-ok', async () => {
    vi.mocked(fetch).mockResolvedValue({
      ok: false,
      status: 404,
    } as Response);

    await expect(buscarUsuario(999)).rejects.toThrow('Erro: 404');
  });
});
```

### 5.5 Mock de timers

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

function debounce<T extends (...args: unknown[]) => void>(fn: T, ms: number): T {
  let timer: ReturnType<typeof setTimeout>;
  return ((...args: unknown[]) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  }) as T;
}

describe('debounce', () => {
  beforeEach(() => {
    vi.useFakeTimers(); // ativar timers falsos
  });

  afterEach(() => {
    vi.useRealTimers(); // restaurar timers reais
  });

  it('deve executar a função após o delay', () => {
    const fn = vi.fn();
    const debounced = debounce(fn, 300);

    debounced();
    expect(fn).not.toHaveBeenCalled(); // ainda não passou o delay

    vi.advanceTimersByTime(300); // avançar 300ms
    expect(fn).toHaveBeenCalledTimes(1);
  });

  it('deve resetar o timer a cada chamada', () => {
    const fn = vi.fn();
    const debounced = debounce(fn, 300);

    debounced();
    vi.advanceTimersByTime(200); // avançar 200ms
    debounced(); // reseta o timer
    vi.advanceTimersByTime(200); // mais 200ms (total 400ms desde o início, 200ms desde o reset)

    expect(fn).not.toHaveBeenCalled(); // ainda não completou 300ms desde o último reset

    vi.advanceTimersByTime(100); // agora sim: 300ms desde o último reset
    expect(fn).toHaveBeenCalledTimes(1);
  });

  it('deve funcionar com advanceTimersByTime e Date', () => {
    vi.setSystemTime(new Date('2025-01-01'));
    expect(new Date().getFullYear()).toBe(2025);

    vi.setSystemTime(new Date('2030-06-15'));
    expect(new Date().getFullYear()).toBe(2030);
  });
});
```

### 5.6 Mock de `localStorage` e `sessionStorage`

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// Serviço que usa localStorage
const tokenService = {
  salvar(token: string): void {
    localStorage.setItem('auth_token', token);
  },
  obter(): string | null {
    return localStorage.getItem('auth_token');
  },
  remover(): void {
    localStorage.removeItem('auth_token');
  },
};

describe('tokenService', () => {
  beforeEach(() => {
    // localStorage já está disponível no JSDOM, mas podemos espionar
    localStorage.clear();
  });

  it('deve salvar token no localStorage', () => {
    const spy = vi.spyOn(Storage.prototype, 'setItem');

    tokenService.salvar('meu-token-jwt');

    expect(spy).toHaveBeenCalledWith('auth_token', 'meu-token-jwt');
    expect(localStorage.getItem('auth_token')).toBe('meu-token-jwt');
  });

  it('deve retornar null quando não há token', () => {
    expect(tokenService.obter()).toBeNull();
  });

  it('deve remover token', () => {
    localStorage.setItem('auth_token', 'token-existente');

    tokenService.remover();

    expect(localStorage.getItem('auth_token')).toBeNull();
  });
});
```

### 5.7 Boas práticas de mocking

| Prática | Recomendação |
|---------|-------------|
| **Mocke apenas fronteiras externas** | APIs, módulos npm, localStorage, timers — não funções internas do seu código |
| **Prefira spy a mock** | Use `spyOn` quando possível para manter o comportamento real |
| **Limpe mocks entre testes** | `vi.clearAllMocks()` no `beforeEach` ou `vi.restoreAllMocks()` no `afterEach` |
| **Não mocke demais** | Se você precisa mockar muitas dependências, o código pode estar muito acoplado |
| **Mock inline vs factory** | Use `vi.mock()` sem factory para auto-mock; com factory para controle fino |
| **Evite mocks de implementação** | `mockReturnValue` é mais claro que `mockImplementation` quando possível |
| **Tipagem** | Use `vi.mocked(fn)` para TypeScript inferir tipos do mock corretamente |

```typescript
// Limpeza de mocks — 3 métodos com diferenças sutis

vi.clearAllMocks();    // Limpa registros (chamadas, retornos), mas mantém implementação mock
vi.resetAllMocks();    // clearAll + remove implementação mock (volta a retornar undefined)
vi.restoreAllMocks();  // resetAll + restaura implementação ORIGINAL (desfaz vi.spyOn)
```

---

## 6. Testando DOM com JSDOM e Happy DOM

Esta seção aborda testes de código que manipula o DOM diretamente — típico de projetos HTML/CSS/JavaScript puros sem frameworks como React. Para projetos React, veja as seções 9-12 que usam React Testing Library.

### 6.1 Configuração do ambiente simulado

O JSDOM (ou Happy DOM) simula um browser no Node.js, fornecendo `document`, `window`, `HTMLElement` e outras APIs Web para que testes possam manipular o DOM sem abrir um browser real.

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    environment: 'jsdom', // ou 'happy-dom'
  },
});
```

Com o ambiente configurado, `document`, `window` e as APIs do DOM estão disponíveis globalmente nos testes.

### 6.2 Criando e manipulando elementos no teste

```typescript
import { describe, it, expect, beforeEach } from 'vitest';

describe('Manipulação de DOM', () => {
  beforeEach(() => {
    // Limpar o body antes de cada teste
    document.body.innerHTML = '';
  });

  it('deve criar e adicionar elemento ao DOM', () => {
    const div = document.createElement('div');
    div.id = 'container';
    div.textContent = 'Hello World';
    document.body.appendChild(div);

    const element = document.getElementById('container');
    expect(element).not.toBeNull();
    expect(element?.textContent).toBe('Hello World');
  });

  it('deve definir HTML completo e consultar elementos', () => {
    document.body.innerHTML = `
      <form id="login-form">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" />
        <label for="senha">Senha:</label>
        <input type="password" id="senha" name="senha" />
        <button type="submit">Entrar</button>
      </form>
    `;

    const form = document.getElementById('login-form') as HTMLFormElement;
    const emailInput = document.getElementById('email') as HTMLInputElement;
    const button = document.querySelector('button') as HTMLButtonElement;

    expect(form).not.toBeNull();
    expect(form.tagName).toBe('FORM');
    expect(emailInput.type).toBe('email');
    expect(button.textContent).toBe('Entrar');
  });
});
```

### 6.3 Testando event listeners

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// src/counter.ts — módulo que adiciona comportamento a um botão
export function inicializarContador(container: HTMLElement): void {
  const display = container.querySelector('.display') as HTMLElement;
  const btnIncrement = container.querySelector('.btn-increment') as HTMLElement;
  const btnDecrement = container.querySelector('.btn-decrement') as HTMLElement;

  let count = 0;

  btnIncrement.addEventListener('click', () => {
    count++;
    display.textContent = String(count);
  });

  btnDecrement.addEventListener('click', () => {
    count = Math.max(0, count - 1);
    display.textContent = String(count);
  });
}

describe('inicializarContador', () => {
  let container: HTMLDivElement;

  beforeEach(() => {
    container = document.createElement('div');
    container.innerHTML = `
      <span class="display">0</span>
      <button class="btn-increment">+</button>
      <button class="btn-decrement">-</button>
    `;
    document.body.appendChild(container);
    inicializarContador(container);
  });

  it('deve incrementar o contador ao clicar no botão +', () => {
    const display = container.querySelector('.display') as HTMLElement;
    const btnIncrement = container.querySelector('.btn-increment') as HTMLElement;

    btnIncrement.click();
    expect(display.textContent).toBe('1');

    btnIncrement.click();
    btnIncrement.click();
    expect(display.textContent).toBe('3');
  });

  it('deve decrementar o contador sem ir abaixo de zero', () => {
    const display = container.querySelector('.display') as HTMLElement;
    const btnDecrement = container.querySelector('.btn-decrement') as HTMLElement;

    btnDecrement.click(); // já está em 0
    expect(display.textContent).toBe('0');

    // Incrementar e depois decrementar
    const btnIncrement = container.querySelector('.btn-increment') as HTMLElement;
    btnIncrement.click();
    btnIncrement.click(); // agora 2

    btnDecrement.click(); // agora 1
    expect(display.textContent).toBe('1');
  });
});
```

### 6.4 Testando formulários HTML puros

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// src/form-handler.ts
export function configurarFormulario(form: HTMLFormElement, onSubmit: (dados: Record<string, string>) => void): void {
  form.addEventListener('submit', (e) => {
    e.preventDefault();

    const formData = new FormData(form);
    const dados: Record<string, string> = {};

    formData.forEach((value, key) => {
      dados[key] = value as string;
    });

    // Validação simples
    const erroEl = form.querySelector('.erro') as HTMLElement;
    if (!dados.email || !dados.email.includes('@')) {
      erroEl.textContent = 'Email inválido';
      erroEl.style.display = 'block';
      return;
    }

    erroEl.style.display = 'none';
    onSubmit(dados);
  });
}

describe('configurarFormulario', () => {
  let form: HTMLFormElement;
  let onSubmit: ReturnType<typeof vi.fn>;

  beforeEach(() => {
    document.body.innerHTML = `
      <form id="meu-form">
        <input type="email" name="email" value="" />
        <input type="text" name="nome" value="" />
        <div class="erro" style="display: none;"></div>
        <button type="submit">Enviar</button>
      </form>
    `;
    form = document.getElementById('meu-form') as HTMLFormElement;
    onSubmit = vi.fn();
    configurarFormulario(form, onSubmit);
  });

  it('deve chamar onSubmit com os dados do formulário', () => {
    const emailInput = form.querySelector('input[name="email"]') as HTMLInputElement;
    const nomeInput = form.querySelector('input[name="nome"]') as HTMLInputElement;

    emailInput.value = 'ana@email.com';
    nomeInput.value = 'Ana Silva';

    form.dispatchEvent(new Event('submit', { bubbles: true, cancelable: true }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'ana@email.com',
      nome: 'Ana Silva',
    });
  });

  it('deve exibir erro para email inválido', () => {
    const emailInput = form.querySelector('input[name="email"]') as HTMLInputElement;
    emailInput.value = 'email-invalido';

    form.dispatchEvent(new Event('submit', { bubbles: true, cancelable: true }));

    const erro = form.querySelector('.erro') as HTMLElement;
    expect(erro.textContent).toBe('Email inválido');
    expect(erro.style.display).toBe('block');
    expect(onSubmit).not.toHaveBeenCalled();
  });

  it('deve esconder erro quando email é válido', () => {
    const emailInput = form.querySelector('input[name="email"]') as HTMLInputElement;
    emailInput.value = 'ana@email.com';

    form.dispatchEvent(new Event('submit', { bubbles: true, cancelable: true }));

    const erro = form.querySelector('.erro') as HTMLElement;
    expect(erro.style.display).toBe('none');
  });
});
```

### 6.5 Testando navegação e History API

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';

// src/router.ts — router simples baseado em hash
export function criarRouter(routes: Record<string, () => string>): {
  navegar: (path: string) => void;
  obterConteudo: () => string;
} {
  function obterConteudo(): string {
    const path = window.location.hash.slice(1) || '/';
    const handler = routes[path];
    return handler ? handler() : '<h1>404 — Página não encontrada</h1>';
  }

  function navegar(path: string): void {
    window.location.hash = path;
    window.dispatchEvent(new HashChangeEvent('hashchange'));
  }

  return { navegar, obterConteudo };
}

describe('criarRouter', () => {
  let router: ReturnType<typeof criarRouter>;

  beforeEach(() => {
    window.location.hash = '';
    router = criarRouter({
      '/': () => '<h1>Home</h1>',
      '/sobre': () => '<h1>Sobre</h1>',
      '/contato': () => '<h1>Contato</h1>',
    });
  });

  it('deve retornar conteúdo da rota raiz', () => {
    expect(router.obterConteudo()).toBe('<h1>Home</h1>');
  });

  it('deve retornar 404 para rota desconhecida', () => {
    window.location.hash = '#/inexistente';
    expect(router.obterConteudo()).toContain('404');
  });

  it('deve navegar para outra rota', () => {
    router.navegar('/sobre');
    expect(window.location.hash).toBe('#/sobre');
    expect(router.obterConteudo()).toBe('<h1>Sobre</h1>');
  });
});
```

### 6.6 Testando manipulação de classes CSS e estilos

```typescript
import { describe, it, expect, beforeEach } from 'vitest';

// src/theme.ts
export function alternarTema(element: HTMLElement): void {
  element.classList.toggle('dark-theme');

  if (element.classList.contains('dark-theme')) {
    element.style.backgroundColor = '#1a1a1a';
    element.style.color = '#ffffff';
  } else {
    element.style.backgroundColor = '#ffffff';
    element.style.color = '#000000';
  }
}

export function aplicarEstiloCondicional(element: HTMLElement, ativo: boolean): void {
  element.classList.toggle('ativo', ativo);
  element.classList.toggle('inativo', !ativo);
  element.setAttribute('aria-pressed', String(ativo));
}

describe('alternarTema', () => {
  let container: HTMLDivElement;

  beforeEach(() => {
    container = document.createElement('div');
    document.body.appendChild(container);
  });

  it('deve adicionar classe dark-theme e estilos', () => {
    alternarTema(container);

    expect(container.classList.contains('dark-theme')).toBe(true);
    expect(container.style.backgroundColor).toBe('#1a1a1a');
    expect(container.style.color).toBe('#ffffff');
  });

  it('deve remover classe dark-theme ao alternar novamente', () => {
    alternarTema(container); // ativa
    alternarTema(container); // desativa

    expect(container.classList.contains('dark-theme')).toBe(false);
    expect(container.style.backgroundColor).toBe('#ffffff');
  });
});

describe('aplicarEstiloCondicional', () => {
  it('deve aplicar classe "ativo" e aria-pressed', () => {
    const btn = document.createElement('button');

    aplicarEstiloCondicional(btn, true);

    expect(btn.classList.contains('ativo')).toBe(true);
    expect(btn.classList.contains('inativo')).toBe(false);
    expect(btn.getAttribute('aria-pressed')).toBe('true');
  });

  it('deve aplicar classe "inativo" quando false', () => {
    const btn = document.createElement('button');

    aplicarEstiloCondicional(btn, false);

    expect(btn.classList.contains('ativo')).toBe(false);
    expect(btn.classList.contains('inativo')).toBe(true);
    expect(btn.getAttribute('aria-pressed')).toBe('false');
  });
});
```

### 6.7 Disparando eventos customizados

```typescript
import { describe, it, expect, vi } from 'vitest';

describe('Eventos customizados', () => {
  it('deve disparar e capturar evento customizado', () => {
    const handler = vi.fn();
    const element = document.createElement('div');

    element.addEventListener('item-selecionado', ((e: CustomEvent) => {
      handler(e.detail);
    }) as EventListener);

    element.dispatchEvent(new CustomEvent('item-selecionado', {
      detail: { id: 42, nome: 'Produto A' },
    }));

    expect(handler).toHaveBeenCalledWith({ id: 42, nome: 'Produto A' });
  });

  it('deve simular eventos de teclado', () => {
    const handler = vi.fn();
    const input = document.createElement('input');

    input.addEventListener('keydown', (e: KeyboardEvent) => {
      if (e.key === 'Enter') handler();
    });

    input.dispatchEvent(new KeyboardEvent('keydown', { key: 'Enter' }));
    expect(handler).toHaveBeenCalledTimes(1);

    input.dispatchEvent(new KeyboardEvent('keydown', { key: 'Escape' }));
    expect(handler).toHaveBeenCalledTimes(1); // Escape não dispara
  });

  it('deve simular eventos de mouse', () => {
    const handler = vi.fn();
    const button = document.createElement('button');

    button.addEventListener('mouseenter', handler);
    button.addEventListener('mouseleave', handler);

    button.dispatchEvent(new MouseEvent('mouseenter'));
    button.dispatchEvent(new MouseEvent('mouseleave'));

    expect(handler).toHaveBeenCalledTimes(2);
  });
});
```

---

## 7. Testes de Funções Utilitárias e Lógica Pura

Funções puras — que não dependem de DOM, APIs externas ou estado global — são as mais simples de testar. São o alicerce da pirâmide de testes e devem representar a maior parte da suite.

### 7.1 Testando validações

```typescript
// src/utils/validators.ts
export function validarEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

export function validarCPF(cpf: string): boolean {
  const limpo = cpf.replace(/\D/g, '');
  if (limpo.length !== 11) return false;
  if (/^(\d)\1{10}$/.test(limpo)) return false; // todos iguais

  let soma = 0;
  for (let i = 0; i < 9; i++) soma += parseInt(limpo[i]) * (10 - i);
  let resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  if (resto !== parseInt(limpo[9])) return false;

  soma = 0;
  for (let i = 0; i < 10; i++) soma += parseInt(limpo[i]) * (11 - i);
  resto = (soma * 10) % 11;
  if (resto === 10) resto = 0;
  return resto === parseInt(limpo[10]);
}

export function validarSenha(senha: string): { valida: boolean; erros: string[] } {
  const erros: string[] = [];
  if (senha.length < 8) erros.push('Mínimo de 8 caracteres');
  if (!/[A-Z]/.test(senha)) erros.push('Pelo menos uma letra maiúscula');
  if (!/[a-z]/.test(senha)) erros.push('Pelo menos uma letra minúscula');
  if (!/\d/.test(senha)) erros.push('Pelo menos um número');
  if (!/[!@#$%^&*]/.test(senha)) erros.push('Pelo menos um caractere especial');
  return { valida: erros.length === 0, erros };
}
```

```typescript
// src/utils/validators.test.ts
import { describe, it, expect } from 'vitest';
import { validarEmail, validarCPF, validarSenha } from './validators';

describe('validarEmail', () => {
  it.each([
    'ana@email.com',
    'usuario.nome@dominio.com.br',
    'test+tag@example.org',
  ])('deve aceitar email válido: %s', (email) => {
    expect(validarEmail(email)).toBe(true);
  });

  it.each([
    '',
    'sem-arroba.com',
    '@sem-usuario.com',
    'sem-dominio@',
    'espaço @email.com',
    'duplo@@email.com',
  ])('deve rejeitar email inválido: "%s"', (email) => {
    expect(validarEmail(email)).toBe(false);
  });
});

describe('validarCPF', () => {
  it('deve aceitar CPF válido sem formatação', () => {
    expect(validarCPF('52998224725')).toBe(true);
  });

  it('deve aceitar CPF válido com formatação', () => {
    expect(validarCPF('529.982.247-25')).toBe(true);
  });

  it.each([
    '00000000000', '11111111111', '22222222222',
  ])('deve rejeitar CPF com todos os dígitos iguais: %s', (cpf) => {
    expect(validarCPF(cpf)).toBe(false);
  });

  it('deve rejeitar CPF com tamanho incorreto', () => {
    expect(validarCPF('1234567')).toBe(false);
    expect(validarCPF('123456789012')).toBe(false);
  });

  it('deve rejeitar CPF com dígito verificador inválido', () => {
    expect(validarCPF('52998224720')).toBe(false);
  });
});

describe('validarSenha', () => {
  it('deve aceitar senha forte', () => {
    const resultado = validarSenha('MinhaSenh@1');
    expect(resultado.valida).toBe(true);
    expect(resultado.erros).toHaveLength(0);
  });

  it('deve rejeitar senha curta', () => {
    const resultado = validarSenha('Ab1!');
    expect(resultado.valida).toBe(false);
    expect(resultado.erros).toContain('Mínimo de 8 caracteres');
  });

  it('deve retornar múltiplos erros', () => {
    const resultado = validarSenha('abc');
    expect(resultado.valida).toBe(false);
    expect(resultado.erros).toContain('Mínimo de 8 caracteres');
    expect(resultado.erros).toContain('Pelo menos uma letra maiúscula');
    expect(resultado.erros).toContain('Pelo menos um número');
    expect(resultado.erros).toContain('Pelo menos um caractere especial');
  });

  it('deve rejeitar senha sem caractere especial', () => {
    const resultado = validarSenha('MinhaSenha1');
    expect(resultado.valida).toBe(false);
    expect(resultado.erros).toContain('Pelo menos um caractere especial');
  });
});
```

### 7.2 Testando formatadores

```typescript
// src/utils/formatters.ts
export function formatarMoeda(valor: number, locale = 'pt-BR', moeda = 'BRL'): string {
  return new Intl.NumberFormat(locale, { style: 'currency', currency: moeda }).format(valor);
}

export function formatarData(data: Date, locale = 'pt-BR'): string {
  return new Intl.DateTimeFormat(locale, {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
  }).format(data);
}

export function formatarCPF(cpf: string): string {
  const limpo = cpf.replace(/\D/g, '');
  return limpo.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, '$1.$2.$3-$4');
}

export function formatarTelefone(telefone: string): string {
  const limpo = telefone.replace(/\D/g, '');
  if (limpo.length === 11) {
    return limpo.replace(/(\d{2})(\d{5})(\d{4})/, '($1) $2-$3');
  }
  return limpo.replace(/(\d{2})(\d{4})(\d{4})/, '($1) $2-$3');
}
```

```typescript
// src/utils/formatters.test.ts
import { describe, it, expect } from 'vitest';
import { formatarMoeda, formatarData, formatarCPF, formatarTelefone } from './formatters';

describe('formatarMoeda', () => {
  it('deve formatar valor em reais', () => {
    expect(formatarMoeda(1234.56)).toBe('R$ 1.234,56');
  });

  it('deve formatar valor zero', () => {
    expect(formatarMoeda(0)).toBe('R$ 0,00');
  });

  it('deve formatar valor negativo', () => {
    expect(formatarMoeda(-99.9)).toContain('99,90');
  });
});

describe('formatarData', () => {
  it('deve formatar data no padrão brasileiro', () => {
    const data = new Date(2025, 5, 15); // mês é 0-based
    expect(formatarData(data)).toBe('15/06/2025');
  });
});

describe('formatarCPF', () => {
  it('deve formatar CPF', () => {
    expect(formatarCPF('52998224725')).toBe('529.982.247-25');
  });

  it('deve tratar CPF já com pontuação', () => {
    expect(formatarCPF('529.982.247-25')).toBe('529.982.247-25');
  });
});

describe('formatarTelefone', () => {
  it('deve formatar celular (11 dígitos)', () => {
    expect(formatarTelefone('11999887766')).toBe('(11) 99988-7766');
  });

  it('deve formatar fixo (10 dígitos)', () => {
    expect(formatarTelefone('1133445566')).toBe('(11) 3344-5566');
  });
});
```

### 7.3 Testando funções assíncronas

```typescript
// src/utils/async-helpers.ts
export async function tentarComRetry<T>(
  fn: () => Promise<T>,
  tentativas: number = 3,
  delayMs: number = 1000,
): Promise<T> {
  for (let i = 0; i < tentativas; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === tentativas - 1) throw error;
      await new Promise((resolve) => setTimeout(resolve, delayMs));
    }
  }
  throw new Error('Não deveria chegar aqui');
}

export function comTimeout<T>(promise: Promise<T>, ms: number): Promise<T> {
  const timeout = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error(`Timeout de ${ms}ms excedido`)), ms),
  );
  return Promise.race([promise, timeout]);
}
```

```typescript
// src/utils/async-helpers.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { tentarComRetry, comTimeout } from './async-helpers';

describe('tentarComRetry', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('deve retornar resultado na primeira tentativa', async () => {
    const fn = vi.fn().mockResolvedValue('sucesso');

    const resultado = await tentarComRetry(fn);

    expect(resultado).toBe('sucesso');
    expect(fn).toHaveBeenCalledTimes(1);
  });

  it('deve tentar novamente após falha', async () => {
    const fn = vi.fn()
      .mockRejectedValueOnce(new Error('Falha 1'))
      .mockRejectedValueOnce(new Error('Falha 2'))
      .mockResolvedValue('sucesso');

    const promise = tentarComRetry(fn, 3, 100);
    await vi.advanceTimersByTimeAsync(200);
    const resultado = await promise;

    expect(resultado).toBe('sucesso');
    expect(fn).toHaveBeenCalledTimes(3);
  });

  it('deve lançar erro após esgotar tentativas', async () => {
    const fn = vi.fn().mockRejectedValue(new Error('Falha persistente'));

    const promise = tentarComRetry(fn, 3, 100);
    await vi.advanceTimersByTimeAsync(300);

    await expect(promise).rejects.toThrow('Falha persistente');
    expect(fn).toHaveBeenCalledTimes(3);
  });
});

describe('comTimeout', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('deve resolver se completar dentro do timeout', async () => {
    const promise = new Promise<string>((resolve) =>
      setTimeout(() => resolve('ok'), 500),
    );

    const resultado = comTimeout(promise, 1000);
    await vi.advanceTimersByTimeAsync(500);

    await expect(resultado).resolves.toBe('ok');
  });

  it('deve rejeitar se exceder o timeout', async () => {
    const promise = new Promise<string>((resolve) =>
      setTimeout(() => resolve('ok'), 2000),
    );

    const resultado = comTimeout(promise, 1000);
    await vi.advanceTimersByTimeAsync(1000);

    await expect(resultado).rejects.toThrow('Timeout de 1000ms excedido');
  });
});
```

### 7.4 Testando tratamento de erros

```typescript
// src/utils/parser.ts
export function parseJSON<T>(json: string): T {
  try {
    return JSON.parse(json) as T;
  } catch {
    throw new Error(`JSON inválido: ${json.substring(0, 50)}...`);
  }
}

export function parseIntSeguro(valor: string, fallback: number = 0): number {
  const numero = parseInt(valor, 10);
  return isNaN(numero) ? fallback : numero;
}
```

```typescript
// src/utils/parser.test.ts
import { describe, it, expect } from 'vitest';
import { parseJSON, parseIntSeguro } from './parser';

describe('parseJSON', () => {
  it('deve fazer parse de JSON válido', () => {
    const resultado = parseJSON<{ nome: string }>('{"nome":"Ana"}');
    expect(resultado).toEqual({ nome: 'Ana' });
  });

  it('deve fazer parse de array JSON', () => {
    const resultado = parseJSON<number[]>('[1,2,3]');
    expect(resultado).toEqual([1, 2, 3]);
  });

  it('deve lançar erro para JSON inválido', () => {
    expect(() => parseJSON('{invalido}')).toThrow('JSON inválido');
  });

  it('deve lançar erro para string vazia', () => {
    expect(() => parseJSON('')).toThrow('JSON inválido');
  });
});

describe('parseIntSeguro', () => {
  it('deve converter string numérica', () => {
    expect(parseIntSeguro('42')).toBe(42);
  });

  it('deve retornar fallback para string não numérica', () => {
    expect(parseIntSeguro('abc')).toBe(0);
    expect(parseIntSeguro('abc', -1)).toBe(-1);
  });

  it('deve converter string com parte numérica', () => {
    expect(parseIntSeguro('42px')).toBe(42);
  });
});
```

### 7.5 Property-based testing com fast-check

Property-based testing gera automaticamente centenas de entradas aleatórias para verificar que propriedades/invariantes do código sempre são verdadeiras — uma abordagem complementar aos testes baseados em exemplos.

```bash
npm install -D fast-check
```

```typescript
// src/utils/math.test.ts
import { describe, it, expect } from 'vitest';
import fc from 'fast-check';
import { somar, ordenar } from './math';

describe('Property-based testing', () => {
  it('somar deve ser comutativa: a + b === b + a', () => {
    fc.assert(
      fc.property(fc.integer(), fc.integer(), (a, b) => {
        expect(somar(a, b)).toBe(somar(b, a));
      }),
    );
  });

  it('somar com zero deve retornar o mesmo número', () => {
    fc.assert(
      fc.property(fc.integer(), (n) => {
        expect(somar(n, 0)).toBe(n);
      }),
    );
  });

  it('ordenar deve retornar array com mesmo tamanho', () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        expect(ordenar([...arr])).toHaveLength(arr.length);
      }),
    );
  });

  it('ordenar deve retornar elementos em ordem crescente', () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const resultado = ordenar([...arr]);
        for (let i = 1; i < resultado.length; i++) {
          expect(resultado[i]).toBeGreaterThanOrEqual(resultado[i - 1]);
        }
      }),
    );
  });

  it('formatarCPF deve manter 11 dígitos após formatação', () => {
    fc.assert(
      fc.property(
        fc.stringOf(fc.constantFrom('0','1','2','3','4','5','6','7','8','9'), { minLength: 11, maxLength: 11 }),
        (cpf) => {
          const formatado = formatarCPF(cpf);
          const digitos = formatado.replace(/\D/g, '');
          expect(digitos).toHaveLength(11);
        },
      ),
    );
  });
});
```

---

## 8. Testes de Acessibilidade

Testes de acessibilidade automatizados verificam que a aplicação segue as diretrizes WCAG e usa atributos ARIA corretamente, garantindo que pessoas com deficiência possam utilizá-la.

### 8.1 axe-core — o motor de acessibilidade

O **axe-core** é o motor de verificação de acessibilidade mais utilizado. Ele analisa o DOM e reporta violações de regras WCAG. Existem integrações para Vitest/Jest, Cypress e Playwright.

```bash
# Para Vitest/Jest
npm install -D axe-core vitest-axe

# Ou com jest-axe
npm install -D axe-core jest-axe @types/jest-axe
```

**Setup (`src/test/setup.ts`):**

```typescript
import '@testing-library/jest-dom/vitest';
import 'vitest-axe/extend-expect';
```

### 8.2 Verificando acessibilidade com vitest-axe

```typescript
import { describe, it, expect } from 'vitest';
import { axe } from 'vitest-axe';

describe('Acessibilidade — Formulário de login', () => {
  it('deve não ter violações de acessibilidade', async () => {
    document.body.innerHTML = `
      <main>
        <h1>Login</h1>
        <form aria-label="Formulário de login">
          <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required aria-required="true" />
          </div>
          <div>
            <label for="senha">Senha:</label>
            <input type="password" id="senha" name="senha" required aria-required="true" />
          </div>
          <button type="submit">Entrar</button>
        </form>
      </main>
    `;

    const results = await axe(document.body);
    expect(results).toHaveNoViolations();
  });

  it('deve detectar violações — input sem label', async () => {
    document.body.innerHTML = `
      <form>
        <input type="text" name="nome" />
        <button type="submit">Enviar</button>
      </form>
    `;

    const results = await axe(document.body);
    expect(results.violations.length).toBeGreaterThan(0);

    const labelViolation = results.violations.find(v => v.id === 'label');
    expect(labelViolation).toBeDefined();
  });
});
```

### 8.3 Acessibilidade em componentes React com Testing Library

```typescript
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/react';
import { axe } from 'vitest-axe';
import { LoginForm } from './LoginForm';

describe('LoginForm — Acessibilidade', () => {
  it('deve não ter violações de acessibilidade', async () => {
    const { container } = render(<LoginForm onSubmit={() => {}} />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### 8.4 Testando ARIA roles e labels

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';

describe('Atributos ARIA', () => {
  it('deve ter role e aria-label corretos no nav', () => {
    render(
      <nav aria-label="Menu principal">
        <ul role="menubar">
          <li role="menuitem"><a href="/">Home</a></li>
          <li role="menuitem"><a href="/sobre">Sobre</a></li>
        </ul>
      </nav>,
    );

    expect(screen.getByRole('navigation', { name: 'Menu principal' })).toBeInTheDocument();
    expect(screen.getAllByRole('menuitem')).toHaveLength(2);
  });

  it('deve marcar campo obrigatório com aria-required', () => {
    render(
      <form>
        <label htmlFor="nome">Nome:</label>
        <input id="nome" type="text" aria-required="true" required />
      </form>,
    );

    const input = screen.getByRole('textbox', { name: 'Nome:' });
    expect(input).toHaveAttribute('aria-required', 'true');
    expect(input).toBeRequired();
  });

  it('deve associar mensagem de erro com aria-describedby', () => {
    render(
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          aria-invalid="true"
          aria-describedby="email-error"
        />
        <span id="email-error" role="alert">Email inválido</span>
      </div>,
    );

    const input = screen.getByRole('textbox', { name: 'Email:' });
    expect(input).toHaveAttribute('aria-invalid', 'true');
    expect(input).toHaveAttribute('aria-describedby', 'email-error');
    expect(screen.getByRole('alert')).toHaveTextContent('Email inválido');
  });

  it('deve ter botão com aria-expanded para menu dropdown', () => {
    render(
      <div>
        <button aria-expanded="false" aria-controls="menu-dropdown">
          Menu ▼
        </button>
        <ul id="menu-dropdown" role="menu" hidden>
          <li role="menuitem">Item 1</li>
        </ul>
      </div>,
    );

    const botao = screen.getByRole('button', { name: /menu/i });
    expect(botao).toHaveAttribute('aria-expanded', 'false');
    expect(botao).toHaveAttribute('aria-controls', 'menu-dropdown');
  });
});
```

### 8.5 Testando navegação por teclado

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Navegação por teclado', () => {
  it('deve focar campos na ordem correta com Tab', async () => {
    const user = userEvent.setup();

    render(
      <form>
        <label htmlFor="nome">Nome:</label>
        <input id="nome" type="text" />
        <label htmlFor="email">Email:</label>
        <input id="email" type="email" />
        <button type="submit">Enviar</button>
      </form>,
    );

    await user.tab();
    expect(screen.getByRole('textbox', { name: 'Nome:' })).toHaveFocus();

    await user.tab();
    expect(screen.getByRole('textbox', { name: 'Email:' })).toHaveFocus();

    await user.tab();
    expect(screen.getByRole('button', { name: 'Enviar' })).toHaveFocus();
  });

  it('deve submeter formulário com Enter', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn((e: React.FormEvent) => e.preventDefault());

    render(
      <form onSubmit={onSubmit}>
        <label htmlFor="busca">Buscar:</label>
        <input id="busca" type="text" />
        <button type="submit">Buscar</button>
      </form>,
    );

    const input = screen.getByRole('textbox', { name: 'Buscar:' });
    await user.click(input);
    await user.keyboard('{Enter}');

    expect(onSubmit).toHaveBeenCalled();
  });

  it('deve fechar modal com Escape', async () => {
    const user = userEvent.setup();
    const onClose = vi.fn();

    render(
      <div role="dialog" aria-label="Confirmação" onKeyDown={(e) => {
        if (e.key === 'Escape') onClose();
      }}>
        <p>Tem certeza?</p>
        <button onClick={onClose}>Fechar</button>
      </div>,
    );

    const dialog = screen.getByRole('dialog');
    await user.click(dialog);
    await user.keyboard('{Escape}');

    expect(onClose).toHaveBeenCalled();
  });
});
```

### 8.6 Checklist de testes de acessibilidade

| Verificação | Como testar | Ferramenta |
|-------------|-------------|------------|
| Todas as imagens têm `alt` | `axe()` detecta automaticamente | vitest-axe |
| Inputs têm labels associados | `getByRole('textbox', { name: '...' })` | RTL |
| Contraste de cores é suficiente | `axe()` com regra `color-contrast` | vitest-axe |
| Foco visível em todos os interativos | `toHaveFocus()` após Tab | RTL + userEvent |
| Hierarquia de headings é correta | `axe()` com regra `heading-order` | vitest-axe |
| Formulários têm mensagens de erro acessíveis | `aria-describedby` + `role="alert"` | RTL |
| Modais capturam o foco (focus trap) | Tab + Shift+Tab dentro do modal | RTL + userEvent |
| Conteúdo dinâmico usa `aria-live` | `getByRole('status')` ou `getByRole('alert')` | RTL |
| Idioma da página está definido | `<html lang="pt-BR">` | axe |
| Links e botões têm texto descritivo | `getByRole('link', { name: '...' })` | RTL |

### 8.7 Níveis de conformidade WCAG

As **Web Content Accessibility Guidelines (WCAG)** definem três níveis de conformidade. Cada nível inclui todos os critérios do anterior:

| Nível | Descrição | Exemplos de critérios |
|-------|-----------|----------------------|
| **A** (mínimo) | Funcionalidade básica para pessoas com deficiência | Texto alternativo em imagens, conteúdo não depende apenas de cor, teclado acessível |
| **AA** (recomendado) | Elimina barreiras significativas — **meta para a maioria dos projetos** | Contraste mínimo 4.5:1, redimensionamento até 200%, foco visível, labels descritivos |
| **AAA** (ideal) | Nível mais alto — difícil de atingir para todo o conteúdo | Contraste 7:1, linguagem simplificada, sem limite de tempo, interpretação em Libras |

A maioria das legislações (incluindo a Lei Brasileira de Inclusão — LBI) recomenda o nível **AA** como meta mínima.

**Configurando axe-core para um nível específico:**

```typescript
import { describe, it, expect } from 'vitest';
import { axe } from 'vitest-axe';

describe('Conformidade WCAG', () => {
  it('deve atender nível WCAG 2.1 AA', async () => {
    document.body.innerHTML = `
      <main>
        <h1>Cadastro</h1>
        <form aria-label="Formulário de cadastro">
          <label for="nome">Nome completo:</label>
          <input id="nome" type="text" required aria-required="true" />
          <button type="submit">Cadastrar</button>
        </form>
      </main>
    `;

    const results = await axe(document.body, {
      runOnly: {
        type: 'tag',
        values: ['wcag2a', 'wcag2aa', 'wcag21aa'],
      },
    });

    expect(results).toHaveNoViolations();
  });

  it('deve atender nível WCAG 2.1 AAA (critérios específicos)', async () => {
    document.body.innerHTML = `
      <article lang="pt-BR">
        <h1>Artigo com contraste elevado</h1>
        <p style="color: #000; background: #fff;">
          Texto com contraste 21:1 (máximo).
        </p>
      </article>
    `;

    const results = await axe(document.body, {
      runOnly: {
        type: 'tag',
        values: ['wcag2aaa'],
      },
    });

    expect(results).toHaveNoViolations();
  });
});
```

**Tags disponíveis no axe-core para WCAG:**

| Tag | Descrição |
|-----|-----------|
| `wcag2a` | WCAG 2.0 Nível A |
| `wcag2aa` | WCAG 2.0 Nível AA |
| `wcag2aaa` | WCAG 2.0 Nível AAA |
| `wcag21a` | WCAG 2.1 Nível A |
| `wcag21aa` | WCAG 2.1 Nível AA |
| `wcag22aa` | WCAG 2.2 Nível AA |
| `best-practice` | Boas práticas além da WCAG |
| `section508` | Section 508 (EUA) |

### 8.8 Testando critérios WCAG específicos

#### 8.8.1 Contraste de cores (WCAG 1.4.3 / 1.4.6)

```typescript
import { describe, it, expect } from 'vitest';
import { axe } from 'vitest-axe';

describe('Contraste de cores', () => {
  it('deve ter contraste suficiente para texto normal (4.5:1 — AA)', async () => {
    document.body.innerHTML = `
      <p style="color: #767676; background-color: #ffffff;">
        Texto cinza — contraste exato de 4.54:1 (passa AA)
      </p>
    `;

    const results = await axe(document.body, {
      runOnly: ['color-contrast'],
    });
    expect(results).toHaveNoViolations();
  });

  it('deve detectar contraste insuficiente', async () => {
    document.body.innerHTML = `
      <p style="color: #aaaaaa; background-color: #ffffff;">
        Texto claro demais — contraste de 2.32:1 (falha)
      </p>
    `;

    const results = await axe(document.body, {
      runOnly: ['color-contrast'],
    });
    expect(results.violations.length).toBeGreaterThan(0);
    expect(results.violations[0].id).toBe('color-contrast');
  });
});
```

#### 8.8.2 Skip navigation (WCAG 2.4.1)

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <a href="#main-content" className="skip-link">
        Pular para o conteúdo principal
      </a>
      <nav aria-label="Menu principal">
        <a href="/">Home</a>
        <a href="/sobre">Sobre</a>
        <a href="/contato">Contato</a>
      </nav>
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </>
  );
}

describe('Skip navigation', () => {
  it('deve ter link para pular navegação', () => {
    render(<Layout><p>Conteúdo</p></Layout>);

    const skipLink = screen.getByRole('link', { name: /pular para o conteúdo/i });
    expect(skipLink).toBeInTheDocument();
    expect(skipLink).toHaveAttribute('href', '#main-content');
  });

  it('deve ser o primeiro elemento focável da página', async () => {
    const user = userEvent.setup();
    render(<Layout><p>Conteúdo</p></Layout>);

    await user.tab();
    expect(screen.getByRole('link', { name: /pular para o conteúdo/i })).toHaveFocus();
  });

  it('deve ter target com id correspondente', () => {
    render(<Layout><p>Conteúdo</p></Layout>);

    const main = screen.getByRole('main');
    expect(main).toHaveAttribute('id', 'main-content');
  });
});
```

#### 8.8.3 Foco visível (WCAG 2.4.7)

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Foco visível', () => {
  it('deve manter outline visível em elementos focáveis', async () => {
    const user = userEvent.setup();

    render(
      <div>
        <button>Primeiro</button>
        <a href="/pagina">Link</a>
        <input type="text" aria-label="Campo" />
      </div>,
    );

    const botao = screen.getByRole('button', { name: 'Primeiro' });
    await user.tab();
    expect(botao).toHaveFocus();
    // Verificar que o outline não foi removido via CSS
    const styles = window.getComputedStyle(botao);
    expect(styles.outlineStyle).not.toBe('none');
  });

  it('não deve ter outline: none em elementos interativos', () => {
    document.head.innerHTML = `
      <style>
        /* Anti-pattern: remover outline globalmente */
        /* *:focus { outline: none; } */

        /* Correto: customizar, não remover */
        *:focus-visible {
          outline: 2px solid #005fcc;
          outline-offset: 2px;
        }
      </style>
    `;

    render(
      <button>Teste</button>,
    );

    const botao = screen.getByRole('button');
    botao.focus();
    const styles = window.getComputedStyle(botao);
    // outline não deve ser 'none' — deve ser customizado ou padrão
    expect(styles.outlineStyle).not.toBe('none');
  });
});
```

#### 8.8.4 Landmarks e estrutura semântica (WCAG 1.3.1)

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { axe } from 'vitest-axe';

describe('Landmarks e semântica', () => {
  it('deve ter landmarks principais', () => {
    render(
      <>
        <header><nav aria-label="Principal"><a href="/">Home</a></nav></header>
        <main><h1>Título</h1><p>Conteúdo</p></main>
        <aside aria-label="Links relacionados"><p>Sidebar</p></aside>
        <footer><p>© 2025</p></footer>
      </>,
    );

    expect(screen.getByRole('banner')).toBeInTheDocument();       // <header>
    expect(screen.getByRole('navigation')).toBeInTheDocument();   // <nav>
    expect(screen.getByRole('main')).toBeInTheDocument();         // <main>
    expect(screen.getByRole('complementary')).toBeInTheDocument();// <aside>
    expect(screen.getByRole('contentinfo')).toBeInTheDocument();  // <footer>
  });

  it('deve ter hierarquia de headings correta', async () => {
    const { container } = render(
      <main>
        <h1>Título principal</h1>
        <section>
          <h2>Subtítulo</h2>
          <h3>Sub-subtítulo</h3>
        </section>
        <section>
          <h2>Outro subtítulo</h2>
        </section>
      </main>,
    );

    const results = await axe(container, {
      runOnly: ['heading-order'],
    });
    expect(results).toHaveNoViolations();
  });

  it('deve detectar headings fora de ordem', async () => {
    const { container } = render(
      <main>
        <h1>Título</h1>
        <h3>Pulou h2 — violação!</h3>
      </main>,
    );

    const results = await axe(container, {
      runOnly: ['heading-order'],
    });
    expect(results.violations.length).toBeGreaterThan(0);
  });
});
```

### 8.9 Padrões ARIA avançados — Widgets interativos

#### 8.9.1 Tabs (ARIA Tabs Pattern)

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Tabs() {
  const [activeTab, setActiveTab] = React.useState(0);
  const tabs = ['Detalhes', 'Avaliações', 'Perguntas'];
  const panels = [
    'Conteúdo de detalhes do produto.',
    'Conteúdo de avaliações dos clientes.',
    'Conteúdo de perguntas frequentes.',
  ];

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    if (e.key === 'ArrowRight') setActiveTab((index + 1) % tabs.length);
    if (e.key === 'ArrowLeft') setActiveTab((index - 1 + tabs.length) % tabs.length);
  };

  return (
    <div>
      <div role="tablist" aria-label="Informações do produto">
        {tabs.map((tab, i) => (
          <button
            key={tab}
            role="tab"
            id={`tab-${i}`}
            aria-selected={activeTab === i}
            aria-controls={`panel-${i}`}
            tabIndex={activeTab === i ? 0 : -1}
            onClick={() => setActiveTab(i)}
            onKeyDown={(e) => handleKeyDown(e, i)}
          >
            {tab}
          </button>
        ))}
      </div>
      {panels.map((content, i) => (
        <div
          key={i}
          role="tabpanel"
          id={`panel-${i}`}
          aria-labelledby={`tab-${i}`}
          hidden={activeTab !== i}
        >
          {content}
        </div>
      ))}
    </div>
  );
}

describe('ARIA Tabs Pattern', () => {
  it('deve ter estrutura ARIA correta', () => {
    render(<Tabs />);

    expect(screen.getByRole('tablist')).toBeInTheDocument();
    expect(screen.getAllByRole('tab')).toHaveLength(3);
    expect(screen.getByRole('tabpanel')).toBeInTheDocument();

    const firstTab = screen.getByRole('tab', { name: 'Detalhes' });
    expect(firstTab).toHaveAttribute('aria-selected', 'true');
  });

  it('deve alternar tabs com clique', async () => {
    const user = userEvent.setup();
    render(<Tabs />);

    await user.click(screen.getByRole('tab', { name: 'Avaliações' }));

    expect(screen.getByRole('tab', { name: 'Avaliações' })).toHaveAttribute('aria-selected', 'true');
    expect(screen.getByRole('tab', { name: 'Detalhes' })).toHaveAttribute('aria-selected', 'false');
    expect(screen.getByRole('tabpanel')).toHaveTextContent('avaliações dos clientes');
  });

  it('deve navegar entre tabs com setas do teclado', async () => {
    const user = userEvent.setup();
    render(<Tabs />);

    const firstTab = screen.getByRole('tab', { name: 'Detalhes' });
    await user.click(firstTab);
    await user.keyboard('{ArrowRight}');

    expect(screen.getByRole('tab', { name: 'Avaliações' })).toHaveAttribute('aria-selected', 'true');

    await user.keyboard('{ArrowRight}');
    expect(screen.getByRole('tab', { name: 'Perguntas' })).toHaveAttribute('aria-selected', 'true');

    // Circular: após o último, volta ao primeiro
    await user.keyboard('{ArrowRight}');
    expect(screen.getByRole('tab', { name: 'Detalhes' })).toHaveAttribute('aria-selected', 'true');
  });
});
```

#### 8.9.2 Accordion (ARIA Accordion Pattern)

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Accordion({ items }: { items: { title: string; content: string }[] }) {
  const [openIndex, setOpenIndex] = React.useState<number | null>(null);

  return (
    <div>
      {items.map((item, i) => (
        <div key={i}>
          <h3>
            <button
              aria-expanded={openIndex === i}
              aria-controls={`accordion-panel-${i}`}
              id={`accordion-header-${i}`}
              onClick={() => setOpenIndex(openIndex === i ? null : i)}
            >
              {item.title}
            </button>
          </h3>
          <div
            id={`accordion-panel-${i}`}
            role="region"
            aria-labelledby={`accordion-header-${i}`}
            hidden={openIndex !== i}
          >
            <p>{item.content}</p>
          </div>
        </div>
      ))}
    </div>
  );
}

describe('ARIA Accordion', () => {
  const items = [
    { title: 'O que é React?', content: 'Uma biblioteca JavaScript para UIs.' },
    { title: 'O que é Vitest?', content: 'Um framework de testes rápido.' },
  ];

  it('deve iniciar com todos os painéis fechados', () => {
    render(<Accordion items={items} />);

    const buttons = screen.getAllByRole('button');
    buttons.forEach(btn => {
      expect(btn).toHaveAttribute('aria-expanded', 'false');
    });
  });

  it('deve expandir painel ao clicar no header', async () => {
    const user = userEvent.setup();
    render(<Accordion items={items} />);

    await user.click(screen.getByRole('button', { name: 'O que é React?' }));

    expect(screen.getByRole('button', { name: 'O que é React?' }))
      .toHaveAttribute('aria-expanded', 'true');
    expect(screen.getByRole('region', { name: 'O que é React?' }))
      .toBeVisible();
  });

  it('deve fechar painel ao clicar novamente', async () => {
    const user = userEvent.setup();
    render(<Accordion items={items} />);

    const btn = screen.getByRole('button', { name: 'O que é Vitest?' });
    await user.click(btn);
    expect(btn).toHaveAttribute('aria-expanded', 'true');

    await user.click(btn);
    expect(btn).toHaveAttribute('aria-expanded', 'false');
  });
});
```

#### 8.9.3 Live regions (aria-live) — Conteúdo dinâmico

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function FormComFeedback() {
  const [message, setMessage] = React.useState('');
  const [status, setStatus] = React.useState<'idle' | 'success' | 'error'>('idle');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setStatus('success');
    setMessage('Dados salvos com sucesso!');
  };

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="nome">Nome:</label>
      <input id="nome" type="text" required />
      <button type="submit">Salvar</button>

      {/* Live region — leitores de tela anunciam mudanças automaticamente */}
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
      >
        {message}
      </div>

      {status === 'error' && (
        <div role="alert" aria-live="assertive">
          Erro ao salvar. Tente novamente.
        </div>
      )}
    </form>
  );
}

describe('Live regions (aria-live)', () => {
  it('deve anunciar mensagem de sucesso via role="status"', async () => {
    const user = userEvent.setup();
    render(<FormComFeedback />);

    await user.type(screen.getByLabelText('Nome:'), 'Maria');
    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    await waitFor(() => {
      expect(screen.getByRole('status')).toHaveTextContent('Dados salvos com sucesso!');
    });
  });

  it('deve ter aria-live="polite" para mensagens não urgentes', () => {
    render(<FormComFeedback />);

    const statusRegion = screen.getByRole('status');
    expect(statusRegion).toHaveAttribute('aria-live', 'polite');
    expect(statusRegion).toHaveAttribute('aria-atomic', 'true');
  });
});
```

### 8.10 Testes de acessibilidade E2E com Cypress e Playwright

Testes de acessibilidade em nível E2E verificam a página completa renderizada em um browser real, incluindo CSS aplicado, fontes carregadas e scripts executados.

#### 8.10.1 Cypress + cypress-axe

```bash
npm install -D cypress-axe axe-core
```

```typescript
// cypress/support/e2e.ts
import 'cypress-axe';
```

```typescript
// cypress/e2e/accessibility.cy.ts
describe('Acessibilidade — fluxo completo', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.injectAxe();
  });

  it('página home deve ser acessível (WCAG AA)', () => {
    cy.checkA11y(null, {
      runOnly: {
        type: 'tag',
        values: ['wcag2a', 'wcag2aa', 'wcag21aa'],
      },
    });
  });

  it('formulário de login deve ser acessível', () => {
    cy.visit('/login');
    cy.injectAxe();
    cy.checkA11y('#login-form');
  });

  it('deve reportar violações com contexto', () => {
    cy.checkA11y(null, undefined, (violations) => {
      violations.forEach((violation) => {
        const nodes = violation.nodes.map(n => n.target).join(', ');
        cy.log(`${violation.id} [${violation.impact}]: ${violation.description}`);
        cy.log(`  Elementos: ${nodes}`);
      });
    });
  });

  it('deve ser acessível após interação (estado dinâmico)', () => {
    cy.get('[data-testid="abrir-menu"]').click();
    cy.checkA11y('[data-testid="menu-dropdown"]');
  });
});
```

#### 8.10.2 Playwright + @axe-core/playwright

```bash
npm install -D @axe-core/playwright
```

```typescript
// tests/e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Acessibilidade E2E', () => {
  test('página home deve ser acessível (WCAG AA)', async ({ page }) => {
    await page.goto('/');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('deve verificar apenas uma seção da página', async ({ page }) => {
    await page.goto('/produtos');

    const results = await new AxeBuilder({ page })
      .include('#lista-produtos')
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('deve excluir componentes de terceiros da verificação', async ({ page }) => {
    await page.goto('/dashboard');

    const results = await new AxeBuilder({ page })
      .exclude('.third-party-widget')
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('deve verificar acessibilidade após interação', async ({ page }) => {
    await page.goto('/');
    await page.getByRole('button', { name: 'Abrir menu' }).click();
    await page.getByRole('menu').waitFor({ state: 'visible' });

    const results = await new AxeBuilder({ page })
      .include('[role="menu"]')
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('deve gerar relatório detalhado de violações', async ({ page }) => {
    await page.goto('/');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    // Log detalhado para CI
    for (const violation of results.violations) {
      console.log(`\n[${violation.impact}] ${violation.id}: ${violation.description}`);
      console.log(`  Help: ${violation.helpUrl}`);
      for (const node of violation.nodes) {
        console.log(`  → ${node.target.join(' > ')}`);
        console.log(`    ${node.failureSummary}`);
      }
    }

    expect(results.violations).toEqual([]);
  });
});
```

### 8.11 Tabela resumo — Ferramentas de acessibilidade por camada

| Camada | Ferramenta | O que verifica |
|--------|-----------|----------------|
| **Unitário / Componente** | vitest-axe / jest-axe | HTML renderizado contra regras WCAG (sem CSS real) |
| **Unitário / Componente** | React Testing Library (`getByRole`) | Semântica, ARIA, labels |
| **Unitário / Componente** | userEvent (Tab, Enter, Escape) | Navegação por teclado |
| **E2E** | cypress-axe | Página completa com CSS/JS no Electron/Chrome |
| **E2E** | @axe-core/playwright | Página completa em múltiplos browsers |
| **E2E / CI** | Lighthouse (accessibility audit) | Score de acessibilidade + sugestões práticas |
| **Manual / Extensão** | axe DevTools, WAVE | Inspeção visual durante desenvolvimento |

---

## 9. React Testing Library — Fundamentos

### 9.1 Filosofia

A React Testing Library (RTL) segue o princípio:

> *"Quanto mais seus testes se parecem com a forma como o software é usado, mais confiança eles dão."*

Em vez de testar detalhes de implementação (estado interno, nomes de métodos), a RTL encoraja testes que interagem com a interface **como o usuário faria**: buscando elementos por texto visível, roles e labels — não por classes CSS ou `data-testid`.

### 9.2 Instalação e configuração com Vitest

```bash
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event vitest jsdom
```

**Setup (`src/test/setup.ts`):**

```typescript
import '@testing-library/jest-dom/vitest';

// Registra matchers como:
// - toBeInTheDocument()
// - toHaveTextContent()
// - toBeVisible()
// - toBeDisabled()
// - toHaveAttribute()
// - toHaveClass()
// - toHaveValue()
// - toBeChecked()
// - toHaveFocus()
// - toBeRequired()
```

**Configuração (`vitest.config.ts`):**

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
  },
});
```

### 9.3 Primeiro teste com RTL

```typescript
// src/components/Greeting.tsx
interface GreetingProps {
  nome: string;
}

export function Greeting({ nome }: GreetingProps) {
  return (
    <div>
      <h1>Olá, {nome}!</h1>
      <p>Bem-vindo ao sistema.</p>
    </div>
  );
}
```

```typescript
// src/components/Greeting.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { Greeting } from './Greeting';

describe('Greeting', () => {
  it('deve exibir o nome do usuário', () => {
    render(<Greeting nome="Ana" />);

    expect(screen.getByRole('heading', { level: 1 })).toHaveTextContent('Olá, Ana!');
    expect(screen.getByText('Bem-vindo ao sistema.')).toBeInTheDocument();
  });

  it('deve renderizar com nome diferente', () => {
    render(<Greeting nome="Carlos" />);

    expect(screen.getByText(/olá, carlos/i)).toBeInTheDocument();
  });
});
```

### 9.4 Queries: `getBy`, `queryBy`, `findBy`

A RTL oferece três famílias de queries, cada uma com comportamento diferente:

| Família | Retorno se NÃO encontrar | Retorno se encontrar | Assíncrono? | Quando usar |
|---------|--------------------------|---------------------|-------------|-------------|
| **`getBy`** | Lança erro | Elemento | Não | Elemento DEVE existir |
| **`queryBy`** | `null` | Elemento | Não | Verificar que NÃO existe |
| **`findBy`** | Lança erro (após timeout) | Promise\<Elemento> | Sim | Elemento aparece de forma assíncrona |

**Variantes com `All`** — para múltiplos elementos:

| Singular | Plural | Retorno plural |
|----------|--------|---------------|
| `getBy` | `getAllBy` | `Element[]` |
| `queryBy` | `queryAllBy` | `Element[]` (vazio se não encontrar) |
| `findBy` | `findAllBy` | `Promise<Element[]>` |

```typescript
import { render, screen } from '@testing-library/react';

render(<ListaDeProdutos />);

// getBy — deve existir (lança erro se não encontrar)
const titulo = screen.getByText('Produtos');

// queryBy — pode não existir (retorna null)
const mensagemVazia = screen.queryByText('Nenhum produto encontrado');
expect(mensagemVazia).toBeNull(); // OK se não existir

// findBy — espera aparecer (assíncrono)
const produto = await screen.findByText('Notebook');

// getAllBy — múltiplos elementos
const items = screen.getAllByRole('listitem');
expect(items).toHaveLength(5);
```

### 9.5 Tipos de query por seletor

A RTL oferece queries por diferentes critérios. A **prioridade recomendada** (do mais acessível ao menos):

| Prioridade | Query | Busca por | Exemplo |
|-----------|-------|-----------|---------|
| 1 | **`ByRole`** | Role ARIA (button, textbox, heading...) | `getByRole('button', { name: 'Salvar' })` |
| 2 | **`ByLabelText`** | Texto do `<label>` associado | `getByLabelText('Email:')` |
| 3 | **`ByPlaceholderText`** | Atributo `placeholder` | `getByPlaceholderText('Digite seu nome')` |
| 4 | **`ByText`** | Conteúdo de texto visível | `getByText('Bem-vindo')` |
| 5 | **`ByDisplayValue`** | Valor atual de input/select/textarea | `getByDisplayValue('ana@email.com')` |
| 6 | **`ByAltText`** | Atributo `alt` de imagens | `getByAltText('Logo da empresa')` |
| 7 | **`ByTitle`** | Atributo `title` | `getByTitle('Fechar')` |
| 8 | **`ByTestId`** | Atributo `data-testid` | `getByTestId('submit-btn')` |

> **Regra:** Prefira `ByRole` e `ByLabelText` — eles refletem como usuários e leitores de tela acessam a interface. Use `ByTestId` como **último recurso**.

#### Exemplos de `ByRole`

```typescript
// Roles comuns e seus elementos HTML correspondentes
screen.getByRole('button');                         // <button>, <input type="submit">
screen.getByRole('textbox');                        // <input type="text">, <textarea>
screen.getByRole('checkbox');                       // <input type="checkbox">
screen.getByRole('radio');                          // <input type="radio">
screen.getByRole('combobox');                       // <select>
screen.getByRole('link');                           // <a href="...">
screen.getByRole('heading', { level: 1 });          // <h1>
screen.getByRole('heading', { level: 2 });          // <h2>
screen.getByRole('navigation');                     // <nav>
screen.getByRole('list');                           // <ul>, <ol>
screen.getByRole('listitem');                       // <li>
screen.getByRole('img');                            // <img>
screen.getByRole('dialog');                         // <dialog>, [role="dialog"]
screen.getByRole('alert');                          // [role="alert"]
screen.getByRole('tab');                            // [role="tab"]
screen.getByRole('tabpanel');                       // [role="tabpanel"]

// Filtrar por nome acessível (texto visível, aria-label, ou label associado)
screen.getByRole('button', { name: 'Salvar' });
screen.getByRole('button', { name: /salvar/i });    // regex case-insensitive
screen.getByRole('textbox', { name: 'Email:' });

// Filtrar por estado
screen.getByRole('checkbox', { checked: true });
screen.getByRole('button', { pressed: true });
```

### 9.6 Matchers do jest-dom

```typescript
const botao = screen.getByRole('button', { name: 'Enviar' });
const input = screen.getByRole('textbox', { name: 'Nome:' });

// Presença
expect(botao).toBeInTheDocument();

// Visibilidade
expect(botao).toBeVisible();

// Conteúdo
expect(botao).toHaveTextContent('Enviar');
expect(botao).toHaveTextContent(/enviar/i); // regex

// Estado
expect(botao).toBeEnabled();
expect(botao).toBeDisabled();
expect(input).toBeRequired();
expect(input).toHaveValue('Ana');
expect(input).toHaveDisplayValue('Ana');

// Classes e estilos
expect(botao).toHaveClass('btn-primary');
expect(botao).toHaveStyle({ backgroundColor: 'blue' });

// Atributos
expect(input).toHaveAttribute('type', 'text');
expect(input).toHaveAttribute('aria-required', 'true');

// Checkbox / Radio
expect(screen.getByRole('checkbox')).toBeChecked();
expect(screen.getByRole('checkbox')).not.toBeChecked();

// Foco
expect(input).toHaveFocus();

// Formulário
expect(input).toBeValid();
expect(input).toBeInvalid();
```

### 9.7 `render()`, `screen` e `cleanup`

```typescript
import { render, screen, cleanup } from '@testing-library/react';

// render — renderiza o componente no DOM virtual
const { container, unmount, rerender } = render(<MeuComponente prop="A" />);

// screen — acesso global às queries (recomendado)
screen.getByText('Hello');

// container — referência direta ao nó DOM (evitar quando possível)
container.querySelector('.minha-classe');

// rerender — re-renderizar com novas props (mesmo componente)
rerender(<MeuComponente prop="B" />);

// unmount — desmontar o componente
unmount();

// cleanup — limpar o DOM (automático com Vitest/Jest, mas pode ser explícito)
cleanup();
```

### 9.8 Debug e troubleshooting

```typescript
import { render, screen } from '@testing-library/react';

render(<MeuComponente />);

// Imprimir o DOM atual no console (útil para debug)
screen.debug();

// Imprimir um elemento específico
screen.debug(screen.getByRole('form'));

// Imprimir com limite de caracteres maior
screen.debug(undefined, 20000);

// logRoles — mostra todos os roles disponíveis no DOM
import { logRoles } from '@testing-library/react';
const { container } = render(<MeuComponente />);
logRoles(container);
// Saída:
// heading: <h1 />
// textbox: <input /> (2)
// button: <button />
```

---

## 10. React Testing Library — Interações e Eventos

### 10.1 `userEvent` vs `fireEvent`

| Aspecto | `userEvent` | `fireEvent` |
|---------|-----------|------------|
| **Pacote** | `@testing-library/user-event` | `@testing-library/react` (built-in) |
| **Fidelidade** | Alta — simula sequência real de eventos | Baixa — dispara evento isolado |
| **Digitação** | Dispara `keyDown`, `keyPress`, `keyUp`, `input`, `change` | Dispara apenas `change` |
| **Clique** | Simula `pointerDown`, `mouseDown`, `pointerUp`, `mouseUp`, `click` | Dispara apenas `click` |
| **Focus** | Move foco automaticamente | Não move foco |
| **Assíncrono** | Sim — todas as ações são `async` | Não |
| **Recomendação** | **Preferido** para todos os testes | Fallback para casos edge |

```typescript
import userEvent from '@testing-library/user-event';
import { fireEvent } from '@testing-library/react';

// RECOMENDADO: userEvent (async)
const user = userEvent.setup();
await user.click(botao);
await user.type(input, 'texto');

// ALTERNATIVA: fireEvent (sync)
fireEvent.click(botao);
fireEvent.change(input, { target: { value: 'texto' } });
```

### 10.2 Simulando cliques

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Componente de exemplo
function Contador() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Contagem: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Incrementar</button>
      <button onClick={() => setCount(0)}>Resetar</button>
    </div>
  );
}

describe('Contador', () => {
  it('deve incrementar ao clicar', async () => {
    const user = userEvent.setup();
    render(<Contador />);

    const botao = screen.getByRole('button', { name: 'Incrementar' });

    await user.click(botao);
    expect(screen.getByText('Contagem: 1')).toBeInTheDocument();

    await user.click(botao);
    await user.click(botao);
    expect(screen.getByText('Contagem: 3')).toBeInTheDocument();
  });

  it('deve resetar ao clicar em Resetar', async () => {
    const user = userEvent.setup();
    render(<Contador />);

    await user.click(screen.getByRole('button', { name: 'Incrementar' }));
    await user.click(screen.getByRole('button', { name: 'Incrementar' }));
    expect(screen.getByText('Contagem: 2')).toBeInTheDocument();

    await user.click(screen.getByRole('button', { name: 'Resetar' }));
    expect(screen.getByText('Contagem: 0')).toBeInTheDocument();
  });
});
```

### 10.3 Simulando digitação

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function CampoBusca({ onSearch }: { onSearch: (term: string) => void }) {
  const [termo, setTermo] = useState('');

  return (
    <div>
      <label htmlFor="busca">Buscar:</label>
      <input
        id="busca"
        type="text"
        value={termo}
        onChange={(e) => setTermo(e.target.value)}
      />
      <button onClick={() => onSearch(termo)}>Buscar</button>
      {termo && <p>Buscando por: {termo}</p>}
    </div>
  );
}

describe('CampoBusca', () => {
  it('deve atualizar o campo ao digitar', async () => {
    const user = userEvent.setup();
    render(<CampoBusca onSearch={() => {}} />);

    const input = screen.getByRole('textbox', { name: 'Buscar:' });
    await user.type(input, 'React');

    expect(input).toHaveValue('React');
    expect(screen.getByText('Buscando por: React')).toBeInTheDocument();
  });

  it('deve limpar e digitar novo valor', async () => {
    const user = userEvent.setup();
    render(<CampoBusca onSearch={() => {}} />);

    const input = screen.getByRole('textbox', { name: 'Buscar:' });

    await user.type(input, 'Vue');
    expect(input).toHaveValue('Vue');

    await user.clear(input);
    expect(input).toHaveValue('');

    await user.type(input, 'React');
    expect(input).toHaveValue('React');
  });

  it('deve chamar onSearch ao clicar em Buscar', async () => {
    const user = userEvent.setup();
    const onSearch = vi.fn();
    render(<CampoBusca onSearch={onSearch} />);

    await user.type(screen.getByRole('textbox', { name: 'Buscar:' }), 'TypeScript');
    await user.click(screen.getByRole('button', { name: 'Buscar' }));

    expect(onSearch).toHaveBeenCalledWith('TypeScript');
  });
});
```

### 10.4 Simulando seleção em select e checkbox

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function FormularioProduto() {
  const [categoria, setCategoria] = useState('');
  const [ativo, setAtivo] = useState(false);

  return (
    <form>
      <label htmlFor="categoria">Categoria:</label>
      <select
        id="categoria"
        value={categoria}
        onChange={(e) => setCategoria(e.target.value)}
      >
        <option value="">Selecione...</option>
        <option value="eletronicos">Eletrônicos</option>
        <option value="livros">Livros</option>
        <option value="roupas">Roupas</option>
      </select>

      <label>
        <input
          type="checkbox"
          checked={ativo}
          onChange={(e) => setAtivo(e.target.checked)}
        />
        Produto ativo
      </label>

      {categoria && <p>Categoria selecionada: {categoria}</p>}
      {ativo && <p>Status: Ativo</p>}
    </form>
  );
}

describe('FormularioProduto', () => {
  it('deve selecionar categoria no dropdown', async () => {
    const user = userEvent.setup();
    render(<FormularioProduto />);

    const select = screen.getByRole('combobox', { name: 'Categoria:' });
    await user.selectOptions(select, 'livros');

    expect(select).toHaveValue('livros');
    expect(screen.getByText('Categoria selecionada: livros')).toBeInTheDocument();
  });

  it('deve marcar e desmarcar checkbox', async () => {
    const user = userEvent.setup();
    render(<FormularioProduto />);

    const checkbox = screen.getByRole('checkbox', { name: 'Produto ativo' });

    expect(checkbox).not.toBeChecked();

    await user.click(checkbox);
    expect(checkbox).toBeChecked();
    expect(screen.getByText('Status: Ativo')).toBeInTheDocument();

    await user.click(checkbox);
    expect(checkbox).not.toBeChecked();
    expect(screen.queryByText('Status: Ativo')).not.toBeInTheDocument();
  });
});
```

### 10.5 Testando formulários com submit e validação

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

interface DadosContato {
  nome: string;
  email: string;
  mensagem: string;
}

function FormularioContato({ onSubmit }: { onSubmit: (dados: DadosContato) => void }) {
  const [erros, setErros] = useState<Record<string, string>>({});

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const dados = Object.fromEntries(formData) as unknown as DadosContato;

    const novosErros: Record<string, string> = {};
    if (!dados.nome.trim()) novosErros.nome = 'Nome é obrigatório';
    if (!dados.email.includes('@')) novosErros.email = 'Email inválido';
    if (dados.mensagem.length < 10) novosErros.mensagem = 'Mensagem deve ter pelo menos 10 caracteres';

    if (Object.keys(novosErros).length > 0) {
      setErros(novosErros);
      return;
    }

    setErros({});
    onSubmit(dados);
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="nome">Nome:</label>
        <input id="nome" name="nome" type="text" aria-describedby={erros.nome ? 'erro-nome' : undefined} />
        {erros.nome && <span id="erro-nome" role="alert">{erros.nome}</span>}
      </div>
      <div>
        <label htmlFor="email">Email:</label>
        <input id="email" name="email" type="email" aria-describedby={erros.email ? 'erro-email' : undefined} />
        {erros.email && <span id="erro-email" role="alert">{erros.email}</span>}
      </div>
      <div>
        <label htmlFor="mensagem">Mensagem:</label>
        <textarea id="mensagem" name="mensagem" aria-describedby={erros.mensagem ? 'erro-mensagem' : undefined} />
        {erros.mensagem && <span id="erro-mensagem" role="alert">{erros.mensagem}</span>}
      </div>
      <button type="submit">Enviar</button>
    </form>
  );
}

describe('FormularioContato', () => {
  it('deve submeter com dados válidos', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<FormularioContato onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Nome:'), 'Ana Silva');
    await user.type(screen.getByLabelText('Email:'), 'ana@email.com');
    await user.type(screen.getByLabelText('Mensagem:'), 'Esta é uma mensagem de teste.');
    await user.click(screen.getByRole('button', { name: 'Enviar' }));

    expect(onSubmit).toHaveBeenCalledWith({
      nome: 'Ana Silva',
      email: 'ana@email.com',
      mensagem: 'Esta é uma mensagem de teste.',
    });
    expect(screen.queryByRole('alert')).not.toBeInTheDocument();
  });

  it('deve exibir erros de validação', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<FormularioContato onSubmit={onSubmit} />);

    await user.click(screen.getByRole('button', { name: 'Enviar' }));

    const alertas = screen.getAllByRole('alert');
    expect(alertas.length).toBeGreaterThanOrEqual(2);
    expect(screen.getByText('Nome é obrigatório')).toBeInTheDocument();
    expect(screen.getByText('Email inválido')).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });

  it('deve limpar erros após correção e submit', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<FormularioContato onSubmit={onSubmit} />);

    // Submit vazio → erros
    await user.click(screen.getByRole('button', { name: 'Enviar' }));
    expect(screen.getAllByRole('alert').length).toBeGreaterThan(0);

    // Preencher e submeter novamente
    await user.type(screen.getByLabelText('Nome:'), 'Ana');
    await user.type(screen.getByLabelText('Email:'), 'ana@email.com');
    await user.type(screen.getByLabelText('Mensagem:'), 'Mensagem válida com mais de 10 caracteres');
    await user.click(screen.getByRole('button', { name: 'Enviar' }));

    expect(screen.queryByRole('alert')).not.toBeInTheDocument();
    expect(onSubmit).toHaveBeenCalled();
  });
});
```

### 10.6 Testando listas e tabelas dinâmicas

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

interface Produto {
  id: number;
  nome: string;
  preco: number;
}

function TabelaProdutos({ produtos, onRemover }: {
  produtos: Produto[];
  onRemover: (id: number) => void;
}) {
  if (produtos.length === 0) {
    return <p>Nenhum produto cadastrado.</p>;
  }

  return (
    <table>
      <thead>
        <tr>
          <th>Nome</th>
          <th>Preço</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        {produtos.map((p) => (
          <tr key={p.id}>
            <td>{p.nome}</td>
            <td>R$ {p.preco.toFixed(2)}</td>
            <td>
              <button onClick={() => onRemover(p.id)}>Remover</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}

describe('TabelaProdutos', () => {
  const produtos: Produto[] = [
    { id: 1, nome: 'Notebook', preco: 3500 },
    { id: 2, nome: 'Mouse', preco: 150 },
    { id: 3, nome: 'Teclado', preco: 250 },
  ];

  it('deve renderizar todos os produtos', () => {
    render(<TabelaProdutos produtos={produtos} onRemover={() => {}} />);

    expect(screen.getByText('Notebook')).toBeInTheDocument();
    expect(screen.getByText('Mouse')).toBeInTheDocument();
    expect(screen.getByText('Teclado')).toBeInTheDocument();
    expect(screen.getAllByRole('row')).toHaveLength(4); // 1 header + 3 data
  });

  it('deve exibir mensagem quando lista está vazia', () => {
    render(<TabelaProdutos produtos={[]} onRemover={() => {}} />);

    expect(screen.getByText('Nenhum produto cadastrado.')).toBeInTheDocument();
    expect(screen.queryByRole('table')).not.toBeInTheDocument();
  });

  it('deve chamar onRemover com o id correto', async () => {
    const user = userEvent.setup();
    const onRemover = vi.fn();
    render(<TabelaProdutos produtos={produtos} onRemover={onRemover} />);

    // Encontrar o botão Remover na linha do "Mouse"
    const linhaDoMouse = screen.getByText('Mouse').closest('tr')!;
    const botaoRemover = within(linhaDoMouse).getByRole('button', { name: 'Remover' });

    await user.click(botaoRemover);

    expect(onRemover).toHaveBeenCalledWith(2); // id do Mouse
  });
});
```

### 10.7 `waitFor` e `findBy` — testes assíncronos

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function ListaDeUsuarios() {
  const [usuarios, setUsuarios] = useState<string[]>([]);
  const [loading, setLoading] = useState(false);

  async function carregar() {
    setLoading(true);
    const response = await fetch('/api/usuarios');
    const data = await response.json();
    setUsuarios(data);
    setLoading(false);
  }

  return (
    <div>
      <button onClick={carregar}>Carregar Usuários</button>
      {loading && <p>Carregando...</p>}
      <ul>
        {usuarios.map((u, i) => <li key={i}>{u}</li>)}
      </ul>
    </div>
  );
}

describe('ListaDeUsuarios', () => {
  it('deve carregar e exibir usuários', async () => {
    const user = userEvent.setup();

    // Mock de fetch
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(['Ana', 'Carlos', 'Maria']),
    });

    render(<ListaDeUsuarios />);

    await user.click(screen.getByRole('button', { name: 'Carregar Usuários' }));

    // findByText espera até o elemento aparecer (timeout padrão: 1000ms)
    expect(await screen.findByText('Ana')).toBeInTheDocument();
    expect(screen.getByText('Carlos')).toBeInTheDocument();
    expect(screen.getByText('Maria')).toBeInTheDocument();

    // Verificar que loading sumiu
    expect(screen.queryByText('Carregando...')).not.toBeInTheDocument();
  });

  it('deve exibir loading enquanto carrega', async () => {
    const user = userEvent.setup();

    let resolvePromise: (value: string[]) => void;
    global.fetch = vi.fn().mockReturnValue({
      ok: true,
      json: () => new Promise<string[]>((resolve) => { resolvePromise = resolve; }),
    });

    render(<ListaDeUsuarios />);
    await user.click(screen.getByRole('button', { name: 'Carregar Usuários' }));

    // Enquanto a Promise não resolve, loading deve estar visível
    expect(screen.getByText('Carregando...')).toBeInTheDocument();

    // Resolver a Promise
    resolvePromise!(['Ana']);

    // waitFor tenta a assertion repetidamente até passar ou expirar
    await waitFor(() => {
      expect(screen.queryByText('Carregando...')).not.toBeInTheDocument();
    });

    expect(screen.getByText('Ana')).toBeInTheDocument();
  });
});
```

**Diferença entre `findBy` e `waitFor`:**

| | `findBy` | `waitFor` |
|---|---------|----------|
| **Retorno** | O elemento encontrado | Nada — executa uma assertion |
| **Uso** | Esperar que um elemento apareça | Esperar qualquer condição |
| **Equivalente** | `waitFor(() => getBy...)` | — |
| **Timeout** | 1000ms (padrão) | 1000ms (padrão) |

```typescript
// Estes dois são equivalentes:
const elemento = await screen.findByText('Olá');
await waitFor(() => expect(screen.getByText('Olá')).toBeInTheDocument());

// waitFor é mais flexível — pode verificar qualquer condição:
await waitFor(() => {
  expect(screen.getAllByRole('listitem')).toHaveLength(5);
});

// Configurar timeout e intervalo de polling:
await waitFor(() => { /* ... */ }, { timeout: 3000, interval: 100 });
```

---

## 11. Testando Componentes React — Cenários Comuns

### 11.1 Componentes com props e children

```typescript
// src/components/Card.tsx
interface CardProps {
  titulo: string;
  destaque?: boolean;
  children: React.ReactNode;
}

export function Card({ titulo, destaque = false, children }: CardProps) {
  return (
    <article className={`card ${destaque ? 'card--destaque' : ''}`}>
      <h2>{titulo}</h2>
      <div className="card__body">{children}</div>
    </article>
  );
}
```

```typescript
// src/components/Card.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { Card } from './Card';

describe('Card', () => {
  it('deve renderizar título e conteúdo', () => {
    render(
      <Card titulo="Meu Card">
        <p>Conteúdo do card</p>
      </Card>,
    );

    expect(screen.getByRole('heading', { level: 2 })).toHaveTextContent('Meu Card');
    expect(screen.getByText('Conteúdo do card')).toBeInTheDocument();
  });

  it('deve aplicar classe de destaque quando prop é true', () => {
    render(
      <Card titulo="Destaque" destaque>
        <p>Conteúdo</p>
      </Card>,
    );

    expect(screen.getByRole('article')).toHaveClass('card--destaque');
  });

  it('não deve ter classe de destaque por padrão', () => {
    render(
      <Card titulo="Normal">
        <p>Conteúdo</p>
      </Card>,
    );

    expect(screen.getByRole('article')).not.toHaveClass('card--destaque');
  });
});
```

### 11.2 Componentes com useState e useEffect

```typescript
// src/components/Timer.tsx
import { useState, useEffect } from 'react';

export function Timer() {
  const [segundos, setSegundos] = useState(0);
  const [ativo, setAtivo] = useState(false);

  useEffect(() => {
    if (!ativo) return;

    const interval = setInterval(() => {
      setSegundos((s) => s + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, [ativo]);

  return (
    <div>
      <p>Tempo: {segundos}s</p>
      <button onClick={() => setAtivo(!ativo)}>
        {ativo ? 'Pausar' : 'Iniciar'}
      </button>
      <button onClick={() => { setAtivo(false); setSegundos(0); }}>
        Resetar
      </button>
    </div>
  );
}
```

```typescript
// src/components/Timer.test.tsx
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { render, screen, act } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Timer } from './Timer';

describe('Timer', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('deve iniciar em 0 e parado', () => {
    render(<Timer />);
    expect(screen.getByText('Tempo: 0s')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Iniciar' })).toBeInTheDocument();
  });

  it('deve contar segundos ao iniciar', async () => {
    const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
    render(<Timer />);

    await user.click(screen.getByRole('button', { name: 'Iniciar' }));

    act(() => { vi.advanceTimersByTime(3000); });

    expect(screen.getByText('Tempo: 3s')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Pausar' })).toBeInTheDocument();
  });

  it('deve pausar e retomar', async () => {
    const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
    render(<Timer />);

    await user.click(screen.getByRole('button', { name: 'Iniciar' }));
    act(() => { vi.advanceTimersByTime(2000); });
    expect(screen.getByText('Tempo: 2s')).toBeInTheDocument();

    await user.click(screen.getByRole('button', { name: 'Pausar' }));
    act(() => { vi.advanceTimersByTime(3000); });
    expect(screen.getByText('Tempo: 2s')).toBeInTheDocument(); // não avançou

    await user.click(screen.getByRole('button', { name: 'Iniciar' }));
    act(() => { vi.advanceTimersByTime(1000); });
    expect(screen.getByText('Tempo: 3s')).toBeInTheDocument();
  });

  it('deve resetar para zero', async () => {
    const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
    render(<Timer />);

    await user.click(screen.getByRole('button', { name: 'Iniciar' }));
    act(() => { vi.advanceTimersByTime(5000); });

    await user.click(screen.getByRole('button', { name: 'Resetar' }));

    expect(screen.getByText('Tempo: 0s')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Iniciar' })).toBeInTheDocument();
  });
});
```

### 11.3 Componentes com useContext (Provider wrapper)

```typescript
// src/contexts/ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface ThemeContextType {
  tema: 'light' | 'dark';
  alternarTema: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

export function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme deve ser usado dentro de ThemeProvider');
  return context;
}

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [tema, setTema] = useState<'light' | 'dark'>('light');
  return (
    <ThemeContext.Provider value={{ tema, alternarTema: () => setTema(t => t === 'light' ? 'dark' : 'light') }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

```typescript
// src/components/ThemeToggle.tsx
import { useTheme } from '../contexts/ThemeContext';

export function ThemeToggle() {
  const { tema, alternarTema } = useTheme();
  return (
    <button onClick={alternarTema}>
      Tema atual: {tema === 'light' ? '☀️ Claro' : '🌙 Escuro'}
    </button>
  );
}
```

```typescript
// src/components/ThemeToggle.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ThemeToggle } from './ThemeToggle';
import { ThemeProvider } from '../contexts/ThemeContext';

// Helper: wrapper com Provider (reutilizável em múltiplos testes)
function renderWithTheme(ui: React.ReactElement) {
  return render(<ThemeProvider>{ui}</ThemeProvider>);
}

describe('ThemeToggle', () => {
  it('deve exibir tema claro por padrão', () => {
    renderWithTheme(<ThemeToggle />);
    expect(screen.getByRole('button')).toHaveTextContent('Claro');
  });

  it('deve alternar para tema escuro', async () => {
    const user = userEvent.setup();
    renderWithTheme(<ThemeToggle />);

    await user.click(screen.getByRole('button'));
    expect(screen.getByRole('button')).toHaveTextContent('Escuro');
  });

  it('deve alternar de volta para claro', async () => {
    const user = userEvent.setup();
    renderWithTheme(<ThemeToggle />);

    await user.click(screen.getByRole('button')); // dark
    await user.click(screen.getByRole('button')); // light

    expect(screen.getByRole('button')).toHaveTextContent('Claro');
  });

  it('deve lançar erro sem Provider', () => {
    // Suprimir erro do console durante o teste
    const spy = vi.spyOn(console, 'error').mockImplementation(() => {});

    expect(() => render(<ThemeToggle />)).toThrow('useTheme deve ser usado dentro de ThemeProvider');

    spy.mockRestore();
  });
});
```

### 11.4 Componentes com React Router (MemoryRouter)

```typescript
// src/components/Navigation.tsx
import { Link, useLocation } from 'react-router-dom';

export function Navigation() {
  const location = useLocation();

  return (
    <nav aria-label="Menu principal">
      <ul>
        <li><Link to="/" className={location.pathname === '/' ? 'active' : ''}>Home</Link></li>
        <li><Link to="/produtos" className={location.pathname === '/produtos' ? 'active' : ''}>Produtos</Link></li>
        <li><Link to="/contato" className={location.pathname === '/contato' ? 'active' : ''}>Contato</Link></li>
      </ul>
    </nav>
  );
}
```

```typescript
// src/components/Navigation.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import { Navigation } from './Navigation';

function renderWithRouter(ui: React.ReactElement, { initialEntries = ['/'] } = {}) {
  return render(
    <MemoryRouter initialEntries={initialEntries}>
      {ui}
    </MemoryRouter>,
  );
}

describe('Navigation', () => {
  it('deve renderizar todos os links', () => {
    renderWithRouter(<Navigation />);

    expect(screen.getByRole('link', { name: 'Home' })).toHaveAttribute('href', '/');
    expect(screen.getByRole('link', { name: 'Produtos' })).toHaveAttribute('href', '/produtos');
    expect(screen.getByRole('link', { name: 'Contato' })).toHaveAttribute('href', '/contato');
  });

  it('deve destacar link ativo na rota /', () => {
    renderWithRouter(<Navigation />, { initialEntries: ['/'] });

    expect(screen.getByRole('link', { name: 'Home' })).toHaveClass('active');
    expect(screen.getByRole('link', { name: 'Produtos' })).not.toHaveClass('active');
  });

  it('deve destacar link ativo na rota /produtos', () => {
    renderWithRouter(<Navigation />, { initialEntries: ['/produtos'] });

    expect(screen.getByRole('link', { name: 'Produtos' })).toHaveClass('active');
    expect(screen.getByRole('link', { name: 'Home' })).not.toHaveClass('active');
  });
});
```

### 11.5 Componentes com chamadas de API (mock de fetch)

```typescript
// src/components/UserProfile.tsx
import { useState, useEffect } from 'react';

interface User {
  id: number;
  nome: string;
  email: string;
}

export function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [erro, setErro] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    setErro(null);
    fetch(`/api/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error('Usuário não encontrado');
        return res.json();
      })
      .then(setUser)
      .catch((e) => setErro(e.message))
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <p role="status">Carregando perfil...</p>;
  if (erro) return <p role="alert">Erro: {erro}</p>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.nome}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

```typescript
// src/components/UserProfile.test.tsx
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import { UserProfile } from './UserProfile';

describe('UserProfile', () => {
  beforeEach(() => {
    global.fetch = vi.fn();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('deve exibir loading inicialmente', () => {
    vi.mocked(fetch).mockReturnValue(new Promise(() => {})); // nunca resolve
    render(<UserProfile userId={1} />);

    expect(screen.getByRole('status')).toHaveTextContent('Carregando perfil...');
  });

  it('deve exibir dados do usuário após carregar', async () => {
    vi.mocked(fetch).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ id: 1, nome: 'Ana Silva', email: 'ana@email.com' }),
    } as Response);

    render(<UserProfile userId={1} />);

    expect(await screen.findByText('Ana Silva')).toBeInTheDocument();
    expect(screen.getByText('ana@email.com')).toBeInTheDocument();
    expect(screen.queryByRole('status')).not.toBeInTheDocument();
  });

  it('deve exibir erro quando a API falha', async () => {
    vi.mocked(fetch).mockResolvedValue({
      ok: false,
      status: 404,
    } as Response);

    render(<UserProfile userId={999} />);

    expect(await screen.findByRole('alert')).toHaveTextContent('Erro: Usuário não encontrado');
  });

  it('deve recarregar ao mudar userId', async () => {
    vi.mocked(fetch)
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ id: 1, nome: 'Ana', email: 'ana@email.com' }),
      } as Response)
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ id: 2, nome: 'Carlos', email: 'carlos@email.com' }),
      } as Response);

    const { rerender } = render(<UserProfile userId={1} />);
    expect(await screen.findByText('Ana')).toBeInTheDocument();

    rerender(<UserProfile userId={2} />);
    expect(await screen.findByText('Carlos')).toBeInTheDocument();

    expect(fetch).toHaveBeenCalledTimes(2);
    expect(fetch).toHaveBeenCalledWith('/api/users/1');
    expect(fetch).toHaveBeenCalledWith('/api/users/2');
  });
});
```

### 11.6 Componentes com Redux/Zustand

**Testando com Redux Toolkit:**

```typescript
// src/store/cartSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CartItem {
  id: number;
  nome: string;
  preco: number;
  quantidade: number;
}

interface CartState {
  items: CartItem[];
}

const initialState: CartState = { items: [] };

export const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<Omit<CartItem, 'quantidade'>>) => {
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) {
        existing.quantidade += 1;
      } else {
        state.items.push({ ...action.payload, quantidade: 1 });
      }
    },
    removeItem: (state, action: PayloadAction<number>) => {
      state.items = state.items.filter(i => i.id !== action.payload);
    },
  },
});
```

```typescript
// src/test/renderWithProviders.tsx
import { render } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { cartSlice } from '../store/cartSlice';
import type { RootState } from '../store';

export function renderWithRedux(
  ui: React.ReactElement,
  { preloadedState }: { preloadedState?: Partial<RootState> } = {},
) {
  const store = configureStore({
    reducer: { cart: cartSlice.reducer },
    preloadedState,
  });

  return {
    ...render(<Provider store={store}>{ui}</Provider>),
    store,
  };
}
```

```typescript
// src/components/CartSummary.test.tsx
import { describe, it, expect } from 'vitest';
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithRedux } from '../test/renderWithProviders';
import { CartSummary } from './CartSummary';

describe('CartSummary', () => {
  it('deve exibir carrinho vazio', () => {
    renderWithRedux(<CartSummary />);
    expect(screen.getByText('Carrinho vazio')).toBeInTheDocument();
  });

  it('deve exibir itens do carrinho com estado pré-carregado', () => {
    renderWithRedux(<CartSummary />, {
      preloadedState: {
        cart: {
          items: [
            { id: 1, nome: 'Notebook', preco: 3500, quantidade: 1 },
            { id: 2, nome: 'Mouse', preco: 150, quantidade: 2 },
          ],
        },
      },
    });

    expect(screen.getByText('Notebook')).toBeInTheDocument();
    expect(screen.getByText('Mouse')).toBeInTheDocument();
    expect(screen.getByText(/R\$ 3\.800,00/)).toBeInTheDocument(); // total
  });
});
```

**Testando com Zustand:**

```typescript
// src/stores/useCounterStore.ts
import { create } from 'zustand';

interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

```typescript
// src/components/ZustandCounter.test.tsx
import { describe, it, expect, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { useCounterStore } from '../stores/useCounterStore';
import { ZustandCounter } from './ZustandCounter';

describe('ZustandCounter', () => {
  // Resetar estado da store entre testes
  beforeEach(() => {
    useCounterStore.setState({ count: 0 });
  });

  it('deve exibir contagem inicial', () => {
    render(<ZustandCounter />);
    expect(screen.getByText('Contagem: 0')).toBeInTheDocument();
  });

  it('deve incrementar', async () => {
    const user = userEvent.setup();
    render(<ZustandCounter />);

    await user.click(screen.getByRole('button', { name: '+' }));
    expect(screen.getByText('Contagem: 1')).toBeInTheDocument();
  });

  it('deve funcionar com estado pré-configurado', () => {
    useCounterStore.setState({ count: 42 });
    render(<ZustandCounter />);
    expect(screen.getByText('Contagem: 42')).toBeInTheDocument();
  });
});
```

### 11.7 Componentes com React Hook Form + Zod

```typescript
// src/components/RegisterForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  nome: z.string().min(3, 'Nome deve ter pelo menos 3 caracteres'),
  email: z.string().email('Email inválido'),
  senha: z.string().min(8, 'Senha deve ter pelo menos 8 caracteres'),
  confirmarSenha: z.string(),
}).refine((data) => data.senha === data.confirmarSenha, {
  message: 'Senhas não conferem',
  path: ['confirmarSenha'],
});

type RegisterData = z.infer<typeof schema>;

export function RegisterForm({ onSubmit }: { onSubmit: (data: RegisterData) => void }) {
  const { register, handleSubmit, formState: { errors } } = useForm<RegisterData>({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <div>
        <label htmlFor="nome">Nome:</label>
        <input id="nome" {...register('nome')} />
        {errors.nome && <span role="alert">{errors.nome.message}</span>}
      </div>
      <div>
        <label htmlFor="email">Email:</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <span role="alert">{errors.email.message}</span>}
      </div>
      <div>
        <label htmlFor="senha">Senha:</label>
        <input id="senha" type="password" {...register('senha')} />
        {errors.senha && <span role="alert">{errors.senha.message}</span>}
      </div>
      <div>
        <label htmlFor="confirmarSenha">Confirmar Senha:</label>
        <input id="confirmarSenha" type="password" {...register('confirmarSenha')} />
        {errors.confirmarSenha && <span role="alert">{errors.confirmarSenha.message}</span>}
      </div>
      <button type="submit">Cadastrar</button>
    </form>
  );
}
```

```typescript
// src/components/RegisterForm.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { RegisterForm } from './RegisterForm';

describe('RegisterForm', () => {
  it('deve submeter com dados válidos', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<RegisterForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Nome:'), 'Ana Silva');
    await user.type(screen.getByLabelText('Email:'), 'ana@email.com');
    await user.type(screen.getByLabelText('Senha:'), 'MinhaSenh@1');
    await user.type(screen.getByLabelText('Confirmar Senha:'), 'MinhaSenh@1');
    await user.click(screen.getByRole('button', { name: 'Cadastrar' }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith(
        expect.objectContaining({
          nome: 'Ana Silva',
          email: 'ana@email.com',
          senha: 'MinhaSenh@1',
        }),
        expect.anything(),
      );
    });
  });

  it('deve exibir erros de validação Zod', async () => {
    const user = userEvent.setup();
    render(<RegisterForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: 'Cadastrar' }));

    await waitFor(() => {
      expect(screen.getAllByRole('alert').length).toBeGreaterThanOrEqual(3);
    });

    expect(screen.getByText('Nome deve ter pelo menos 3 caracteres')).toBeInTheDocument();
    expect(screen.getByText('Email inválido')).toBeInTheDocument();
    expect(screen.getByText('Senha deve ter pelo menos 8 caracteres')).toBeInTheDocument();
  });

  it('deve validar que senhas conferem', async () => {
    const user = userEvent.setup();
    render(<RegisterForm onSubmit={vi.fn()} />);

    await user.type(screen.getByLabelText('Nome:'), 'Ana Silva');
    await user.type(screen.getByLabelText('Email:'), 'ana@email.com');
    await user.type(screen.getByLabelText('Senha:'), 'MinhaSenh@1');
    await user.type(screen.getByLabelText('Confirmar Senha:'), 'SenhaDiferente1');
    await user.click(screen.getByRole('button', { name: 'Cadastrar' }));

    await waitFor(() => {
      expect(screen.getByText('Senhas não conferem')).toBeInTheDocument();
    });
  });
});
```

---

## 12. Testando Hooks Customizados

### 12.1 `renderHook` do `@testing-library/react`

O `renderHook` permite testar hooks React isoladamente, sem precisar criar um componente wrapper.

```bash
# Já incluído no @testing-library/react desde a v14
npm install -D @testing-library/react
```

### 12.2 Hook simples com estado

```typescript
// src/hooks/useToggle.ts
import { useState, useCallback } from 'react';

export function useToggle(inicial = false) {
  const [valor, setValor] = useState(inicial);

  const toggle = useCallback(() => setValor((v) => !v), []);
  const ativar = useCallback(() => setValor(true), []);
  const desativar = useCallback(() => setValor(false), []);

  return { valor, toggle, ativar, desativar };
}
```

```typescript
// src/hooks/useToggle.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useToggle } from './useToggle';

describe('useToggle', () => {
  it('deve iniciar com false por padrão', () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current.valor).toBe(false);
  });

  it('deve iniciar com valor fornecido', () => {
    const { result } = renderHook(() => useToggle(true));
    expect(result.current.valor).toBe(true);
  });

  it('deve alternar valor com toggle', () => {
    const { result } = renderHook(() => useToggle());

    act(() => { result.current.toggle(); });
    expect(result.current.valor).toBe(true);

    act(() => { result.current.toggle(); });
    expect(result.current.valor).toBe(false);
  });

  it('deve ativar e desativar diretamente', () => {
    const { result } = renderHook(() => useToggle());

    act(() => { result.current.ativar(); });
    expect(result.current.valor).toBe(true);

    act(() => { result.current.ativar(); }); // idempotente
    expect(result.current.valor).toBe(true);

    act(() => { result.current.desativar(); });
    expect(result.current.valor).toBe(false);
  });
});
```

### 12.3 Hook com efeitos colaterais

```typescript
// src/hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(chave: string, valorInicial: T) {
  const [valor, setValor] = useState<T>(() => {
    const salvo = localStorage.getItem(chave);
    return salvo ? JSON.parse(salvo) : valorInicial;
  });

  useEffect(() => {
    localStorage.setItem(chave, JSON.stringify(valor));
  }, [chave, valor]);

  return [valor, setValor] as const;
}
```

```typescript
// src/hooks/useLocalStorage.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('deve usar valor inicial quando não há nada salvo', () => {
    const { result } = renderHook(() => useLocalStorage('tema', 'light'));
    expect(result.current[0]).toBe('light');
  });

  it('deve recuperar valor salvo no localStorage', () => {
    localStorage.setItem('tema', JSON.stringify('dark'));
    const { result } = renderHook(() => useLocalStorage('tema', 'light'));
    expect(result.current[0]).toBe('dark');
  });

  it('deve salvar no localStorage ao atualizar', () => {
    const { result } = renderHook(() => useLocalStorage('tema', 'light'));

    act(() => { result.current[1]('dark'); });

    expect(result.current[0]).toBe('dark');
    expect(localStorage.getItem('tema')).toBe('"dark"');
  });

  it('deve funcionar com objetos', () => {
    const { result } = renderHook(() =>
      useLocalStorage('config', { idioma: 'pt', notificacoes: true }),
    );

    act(() => {
      result.current[1]({ idioma: 'en', notificacoes: false });
    });

    expect(result.current[0]).toEqual({ idioma: 'en', notificacoes: false });
    expect(JSON.parse(localStorage.getItem('config')!)).toEqual({
      idioma: 'en',
      notificacoes: false,
    });
  });
});
```

### 12.4 Hook com contexto (wrapper)

```typescript
// src/hooks/useAuth.ts
import { useContext } from 'react';
import { AuthContext } from '../contexts/AuthContext';

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth deve ser usado dentro de AuthProvider');
  return context;
}
```

```typescript
// src/hooks/useAuth.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useAuth } from './useAuth';
import { AuthProvider } from '../contexts/AuthContext';

describe('useAuth', () => {
  it('deve retornar estado inicial não autenticado', () => {
    const { result } = renderHook(() => useAuth(), {
      wrapper: AuthProvider, // passa o Provider como wrapper
    });

    expect(result.current.usuario).toBeNull();
    expect(result.current.autenticado).toBe(false);
  });

  it('deve autenticar usuário', async () => {
    const { result } = renderHook(() => useAuth(), {
      wrapper: AuthProvider,
    });

    await act(async () => {
      await result.current.login('ana@email.com', 'senha123');
    });

    expect(result.current.autenticado).toBe(true);
    expect(result.current.usuario?.email).toBe('ana@email.com');
  });

  it('deve fazer logout', async () => {
    const { result } = renderHook(() => useAuth(), {
      wrapper: AuthProvider,
    });

    await act(async () => {
      await result.current.login('ana@email.com', 'senha123');
    });
    expect(result.current.autenticado).toBe(true);

    act(() => { result.current.logout(); });
    expect(result.current.autenticado).toBe(false);
    expect(result.current.usuario).toBeNull();
  });

  it('deve lançar erro sem Provider', () => {
    const spy = vi.spyOn(console, 'error').mockImplementation(() => {});

    expect(() => renderHook(() => useAuth())).toThrow(
      'useAuth deve ser usado dentro de AuthProvider',
    );

    spy.mockRestore();
  });
});
```

### 12.5 Hook assíncrono com fetch

```typescript
// src/hooks/useFetch.ts
import { useState, useEffect } from 'react';

interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [trigger, setTrigger] = useState(0);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    setError(null);

    fetch(url)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => { if (!cancelled) setData(json); })
      .catch((e) => { if (!cancelled) setError(e.message); })
      .finally(() => { if (!cancelled) setLoading(false); });

    return () => { cancelled = true; };
  }, [url, trigger]);

  const refetch = () => setTrigger((t) => t + 1);

  return { data, loading, error, refetch };
}
```

```typescript
// src/hooks/useFetch.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { useFetch } from './useFetch';

describe('useFetch', () => {
  beforeEach(() => {
    global.fetch = vi.fn();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it('deve retornar dados após fetch bem-sucedido', async () => {
    vi.mocked(fetch).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ nome: 'Ana' }),
    } as Response);

    const { result } = renderHook(() => useFetch('/api/user'));

    // Inicialmente loading
    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual({ nome: 'Ana' });
    expect(result.current.error).toBeNull();
  });

  it('deve retornar erro quando fetch falha', async () => {
    vi.mocked(fetch).mockResolvedValue({
      ok: false,
      status: 500,
    } as Response);

    const { result } = renderHook(() => useFetch('/api/user'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toBeNull();
    expect(result.current.error).toBe('HTTP 500');
  });

  it('deve refazer a requisição ao chamar refetch', async () => {
    vi.mocked(fetch)
      .mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({ v: 1 }) } as Response)
      .mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({ v: 2 }) } as Response);

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => expect(result.current.data).toEqual({ v: 1 }));

    result.current.refetch();

    await waitFor(() => expect(result.current.data).toEqual({ v: 2 }));
    expect(fetch).toHaveBeenCalledTimes(2);
  });
});
```

---

## 13. Snapshot Testing

Snapshot testing captura a saída renderizada de um componente (HTML serializado) e a compara com uma versão salva anteriormente. Se a saída mudar, o teste falha — forçando o desenvolvedor a revisar se a mudança foi intencional.

### 13.1 Snapshot básico com Vitest

```typescript
// src/components/Badge.tsx
interface BadgeProps {
  texto: string;
  variante: 'info' | 'success' | 'warning' | 'danger';
}

export function Badge({ texto, variante }: BadgeProps) {
  return <span className={`badge badge--${variante}`}>{texto}</span>;
}
```

```typescript
// src/components/Badge.test.tsx
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/react';
import { Badge } from './Badge';

describe('Badge', () => {
  it('deve corresponder ao snapshot — variante info', () => {
    const { container } = render(<Badge texto="Novo" variante="info" />);
    expect(container.firstChild).toMatchSnapshot();
  });

  it('deve corresponder ao snapshot — variante danger', () => {
    const { container } = render(<Badge texto="Erro" variante="danger" />);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

Na primeira execução, o Vitest cria um arquivo `__snapshots__/Badge.test.tsx.snap`:

```
// Vitest Snapshot v1

exports[`Badge > deve corresponder ao snapshot — variante info 1`] = `
<span
  class="badge badge--info"
>
  Novo
</span>
`;

exports[`Badge > deve corresponder ao snapshot — variante danger 1`] = `
<span
  class="badge badge--danger"
>
  Erro
</span>
`;
```

### 13.2 Inline snapshots

Em vez de salvar em arquivo separado, inline snapshots armazenam o snapshot diretamente no código do teste:

```typescript
it('deve corresponder ao inline snapshot', () => {
  const { container } = render(<Badge texto="Ok" variante="success" />);

  // Na primeira execução, o Vitest preenche automaticamente o conteúdo
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span
      class="badge badge--success"
    >
      Ok
    </span>
  `);
});
```

### 13.3 Atualizando snapshots

Quando a saída muda intencionalmente (ex: mudou uma classe CSS), o snapshot fica desatualizado e o teste falha. Para atualizar:

```bash
# Atualizar todos os snapshots desatualizados
npx vitest run --update     # ou -u
npx jest --updateSnapshot   # Jest

# Em watch mode, pressionar 'u' para atualizar
```

> **Cuidado:** `--update` atualiza **todos** os snapshots desatualizados de uma vez. Revise as mudanças no diff do Git antes de fazer commit.

### 13.4 Quando usar e quando evitar snapshots

| Usar snapshots para | Evitar snapshots para |
|---------------------|-----------------------|
| Componentes visuais estáveis (ícones, badges, cards) | Componentes com conteúdo dinâmico (timestamps, IDs aleatórios) |
| Garantir que a estrutura HTML não muda sem intenção | Lógica de negócio (use assertions explícitas) |
| Detectar mudanças acidentais em componentes compartilhados | Componentes grandes e complexos (snapshots ficam enormes) |
| Complementar testes de comportamento | Como substituto de testes de interação |

**Problemas comuns com snapshots:**

1. **Snapshots enormes** — difíceis de revisar, qualquer mudança gera diff grande.
2. **Atualizações cegamente** — desenvolvedores executam `--update` sem revisar.
3. **Falsos negativos** — mudança cosmética (espaçamento, atributo de ordem) falha o teste sem bug real.
4. **Dados dinâmicos** — datas, IDs ou textos que mudam a cada execução.

**Tratando dados dinâmicos:**

```typescript
it('deve corresponder ao snapshot ignorando data', () => {
  const { container } = render(<Pedido id={123} data={new Date()} />);

  // Usar serializer customizado ou expect.any()
  expect(container.firstChild).toMatchSnapshot({
    // propriedades com valores dinâmicos
  });
});

// Alternativa: mock de Date
vi.setSystemTime(new Date('2025-01-01'));
```

### 13.5 Boas práticas de snapshots

1. **Mantenha snapshots pequenos** — capture apenas o nó relevante, não o container inteiro.
2. **Revise diffs de snapshot** como revisaria código — snapshots fazem parte do commit.
3. **Prefira inline snapshots** para saídas curtas — mantém contexto junto do teste.
4. **Combine com testes de comportamento** — snapshot verifica estrutura, `userEvent` verifica interação.
5. **Delete snapshots sem teste** — se remover um teste, remova o snapshot correspondente.

```typescript
// BOM: snapshot pequeno e focado
expect(container.querySelector('.badge')).toMatchInlineSnapshot(`
  <span class="badge badge--info">Novo</span>
`);

// RUIM: snapshot do container inteiro com muitos elementos
expect(container).toMatchSnapshot(); // gera snapshot enorme
```

---

## 14. Cypress — Testes E2E

### 14.1 O que é o Cypress?

Cypress é um framework de testes end-to-end (E2E) que roda diretamente no browser, oferecendo uma experiência de desenvolvimento interativa com time-travel debugging, screenshots automáticos e uma API fluente baseada em encadeamento (chaining).

### 14.2 Instalação e configuração

```bash
npm install -D cypress
npx cypress open   # abre o Cypress pela primeira vez e cria a estrutura
```

**Estrutura de pastas criada:**

```
cypress/
├── e2e/                    ← testes E2E
│   ├── login.cy.ts
│   └── produtos.cy.ts
├── fixtures/               ← dados de teste (JSON)
│   └── usuarios.json
├── support/
│   ├── commands.ts         ← custom commands
│   └── e2e.ts              ← configuração global dos testes
└── tsconfig.json
cypress.config.ts           ← configuração principal
```

**Configuração (`cypress.config.ts`):**

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:5173',   // URL do dev server
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,                         // gravar vídeo dos testes
    screenshotOnRunFailure: true,        // screenshot ao falhar
    defaultCommandTimeout: 10000,        // timeout padrão para comandos
    setupNodeEvents(on, config) {
      // listeners para eventos do Node
    },
  },
});
```

**Scripts no `package.json`:**

```json
{
  "scripts": {
    "cy:open": "cypress open",
    "cy:run": "cypress run",
    "cy:run:chrome": "cypress run --browser chrome",
    "test:e2e": "start-server-and-test dev http://localhost:5173 cy:run"
  }
}
```

```bash
# Instalar utilitário para aguardar o servidor
npm install -D start-server-and-test
```

### 14.3 Estrutura de testes

```typescript
// cypress/e2e/home.cy.ts
describe('Página Home', () => {
  beforeEach(() => {
    cy.visit('/'); // visita baseUrl + '/'
  });

  it('deve exibir o título da aplicação', () => {
    cy.get('h1').should('contain.text', 'Bem-vindo');
  });

  it('deve ter link para página de produtos', () => {
    cy.get('a[href="/produtos"]').should('be.visible');
  });
});
```

### 14.4 Seletores e interações

```typescript
describe('Interações básicas', () => {
  beforeEach(() => {
    cy.visit('/formulario');
  });

  it('deve preencher e submeter formulário', () => {
    // Digitação
    cy.get('#nome').type('Ana Silva');
    cy.get('#email').type('ana@email.com');
    cy.get('#senha').type('MinhaSenh@1');

    // Seleção em dropdown
    cy.get('#estado').select('SP');

    // Checkbox
    cy.get('#termos').check();

    // Radio button
    cy.get('input[value="premium"]').check();

    // Clique em botão
    cy.get('button[type="submit"]').click();

    // Verificar resultado
    cy.get('.mensagem-sucesso').should('be.visible');
  });

  it('deve limpar campo e digitar novamente', () => {
    cy.get('#nome').type('texto errado');
    cy.get('#nome').clear().type('texto correto');
    cy.get('#nome').should('have.value', 'texto correto');
  });
});
```

**Seletores recomendados (do melhor ao pior):**

| Prioridade | Seletor | Exemplo | Por que |
|-----------|---------|---------|---------|
| 1 | `data-cy` | `cy.get('[data-cy="submit"]')` | Específico para testes, não muda com design |
| 2 | `data-testid` | `cy.get('[data-testid="submit"]')` | Convenção compartilhada com RTL |
| 3 | `aria-label` | `cy.get('[aria-label="Salvar"]')` | Acessível e semântico |
| 4 | Tag + conteúdo | `cy.contains('button', 'Salvar')` | Legível, mas frágil com i18n |
| 5 | ID | `cy.get('#submit-btn')` | OK se único e estável |
| 6 | Classe CSS | `cy.get('.btn-primary')` | Frágil — muda com refatoração visual |

### 14.5 Assertions (should)

```typescript
// Visibilidade
cy.get('.modal').should('be.visible');
cy.get('.modal').should('not.exist');
cy.get('.loading').should('not.be.visible');

// Texto
cy.get('h1').should('have.text', 'Título Exato');
cy.get('h1').should('contain.text', 'Título');
cy.get('p').should('include.text', 'parcial');

// Atributos
cy.get('input').should('have.attr', 'type', 'email');
cy.get('input').should('have.value', 'ana@email.com');
cy.get('button').should('be.disabled');
cy.get('button').should('be.enabled');

// Classes
cy.get('.card').should('have.class', 'active');
cy.get('.card').should('not.have.class', 'hidden');

// CSS
cy.get('.badge').should('have.css', 'background-color', 'rgb(0, 128, 0)');

// Contagem
cy.get('li').should('have.length', 5);
cy.get('li').should('have.length.greaterThan', 2);

// Checkbox/Radio
cy.get('#termos').should('be.checked');
cy.get('#newsletter').should('not.be.checked');

// Foco
cy.get('#email').should('have.focus');

// Encadeamento múltiplo
cy.get('button')
  .should('be.visible')
  .and('be.enabled')
  .and('contain.text', 'Salvar');
```

### 14.6 Interceptação de requisições (`cy.intercept`)

```typescript
describe('Lista de Produtos', () => {
  it('deve exibir produtos da API', () => {
    // Interceptar GET /api/produtos e retornar dados mock
    cy.intercept('GET', '/api/produtos', {
      statusCode: 200,
      body: [
        { id: 1, nome: 'Notebook', preco: 3500 },
        { id: 2, nome: 'Mouse', preco: 150 },
      ],
    }).as('getProdutos'); // alias para referência

    cy.visit('/produtos');

    // Esperar a requisição ser feita
    cy.wait('@getProdutos');

    // Verificar que os produtos foram renderizados
    cy.get('[data-cy="produto-item"]').should('have.length', 2);
    cy.contains('Notebook').should('be.visible');
    cy.contains('Mouse').should('be.visible');
  });

  it('deve exibir erro quando API falha', () => {
    cy.intercept('GET', '/api/produtos', {
      statusCode: 500,
      body: { error: 'Internal Server Error' },
    }).as('getProdutosErro');

    cy.visit('/produtos');
    cy.wait('@getProdutosErro');

    cy.get('[data-cy="error-message"]').should('contain.text', 'Erro ao carregar produtos');
  });

  it('deve enviar dados corretos ao criar produto', () => {
    cy.intercept('POST', '/api/produtos', (req) => {
      // Verificar o corpo da requisição
      expect(req.body).to.deep.equal({
        nome: 'Teclado',
        preco: 250,
        categoria: 'perifericos',
      });
      req.reply({ statusCode: 201, body: { id: 3, ...req.body } });
    }).as('criarProduto');

    cy.visit('/produtos/novo');
    cy.get('#nome').type('Teclado');
    cy.get('#preco').type('250');
    cy.get('#categoria').select('perifericos');
    cy.get('button[type="submit"]').click();

    cy.wait('@criarProduto');
    cy.get('.toast-sucesso').should('contain.text', 'Produto criado');
  });
});
```

### 14.7 Fixtures e dados de teste

```json
// cypress/fixtures/usuarios.json
[
  {
    "id": 1,
    "nome": "Ana Silva",
    "email": "ana@email.com",
    "role": "admin"
  },
  {
    "id": 2,
    "nome": "Carlos Santos",
    "email": "carlos@email.com",
    "role": "user"
  }
]
```

```typescript
// Usando fixture no teste
describe('Usuários', () => {
  it('deve listar usuários do fixture', () => {
    cy.intercept('GET', '/api/usuarios', { fixture: 'usuarios.json' }).as('getUsuarios');

    cy.visit('/usuarios');
    cy.wait('@getUsuarios');

    cy.get('[data-cy="usuario-row"]').should('have.length', 2);
    cy.contains('Ana Silva').should('be.visible');
  });
});
```

### 14.8 Custom commands

```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, senha: string): Chainable<void>;
      getByTestId(testId: string): Chainable<JQuery<HTMLElement>>;
    }
  }
}

Cypress.Commands.add('login', (email: string, senha: string) => {
  cy.visit('/login');
  cy.get('#email').type(email);
  cy.get('#senha').type(senha);
  cy.get('button[type="submit"]').click();
  cy.url().should('not.include', '/login'); // verifica redirecionamento
});

Cypress.Commands.add('getByTestId', (testId: string) => {
  return cy.get(`[data-testid="${testId}"]`);
});
```

```typescript
// Usando custom commands nos testes
describe('Dashboard', () => {
  beforeEach(() => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 200,
      body: { token: 'fake-jwt-token', user: { nome: 'Ana' } },
    });
    cy.login('ana@email.com', 'senha123');
  });

  it('deve exibir nome do usuário logado', () => {
    cy.contains('Olá, Ana').should('be.visible');
  });
});
```

### 14.9 Testando fluxo completo: Login → CRUD → Logout

```typescript
// cypress/e2e/fluxo-completo.cy.ts
describe('Fluxo completo: Produtos', () => {
  beforeEach(() => {
    // Interceptar autenticação
    cy.intercept('POST', '/api/auth/login', {
      body: { token: 'jwt-token', user: { nome: 'Admin', role: 'admin' } },
    });

    // Interceptar listagem
    cy.intercept('GET', '/api/produtos', {
      body: [{ id: 1, nome: 'Notebook', preco: 3500 }],
    }).as('listaProdutos');

    cy.login('admin@email.com', 'admin123');
  });

  it('deve criar, editar e excluir produto', () => {
    // CRIAR
    cy.intercept('POST', '/api/produtos', {
      statusCode: 201,
      body: { id: 2, nome: 'Mouse Gamer', preco: 250 },
    }).as('criarProduto');

    cy.contains('Novo Produto').click();
    cy.get('#nome').type('Mouse Gamer');
    cy.get('#preco').type('250');
    cy.get('button[type="submit"]').click();
    cy.wait('@criarProduto');
    cy.get('.toast').should('contain.text', 'Produto criado');

    // EDITAR
    cy.intercept('PUT', '/api/produtos/2', {
      body: { id: 2, nome: 'Mouse Gamer RGB', preco: 300 },
    }).as('editarProduto');

    cy.contains('Mouse Gamer').parent().find('[data-cy="btn-editar"]').click();
    cy.get('#nome').clear().type('Mouse Gamer RGB');
    cy.get('#preco').clear().type('300');
    cy.get('button[type="submit"]').click();
    cy.wait('@editarProduto');

    // EXCLUIR
    cy.intercept('DELETE', '/api/produtos/2', { statusCode: 204 }).as('excluirProduto');

    cy.contains('Mouse Gamer RGB').parent().find('[data-cy="btn-excluir"]').click();
    cy.get('[data-cy="confirmar-exclusao"]').click();
    cy.wait('@excluirProduto');
    cy.contains('Mouse Gamer RGB').should('not.exist');
  });
});
```

### 14.10 Boas práticas e anti-patterns

| Boa prática | Anti-pattern |
|-------------|-------------|
| Usar `data-cy` ou `data-testid` para seletores | Depender de classes CSS ou estrutura DOM |
| Interceptar APIs com `cy.intercept` | Depender de um backend real rodando |
| Cada teste deve ser independente | Testes que dependem da ordem de execução |
| `beforeEach` para setup repetitivo | Duplicar código de setup em cada `it` |
| `cy.wait('@alias')` para aguardar requisições | `cy.wait(3000)` — esperas fixas |
| Verificar estado final com assertions | Verificar apenas que clicou no botão |
| Testar apenas fluxos críticos como E2E | Replicar todos os testes unitários como E2E |
| Limpar estado entre testes | Assumir que estado do teste anterior persiste |

---

## 15. Cypress — Component Testing

### 15.1 O que é Component Testing no Cypress?

O Cypress Component Testing permite montar e testar componentes React isoladamente **no browser real**, combinando a velocidade de testes de componente com a fidelidade de um browser.

### 15.2 Configuração para React

```bash
npx cypress open --component
# O Cypress detecta o framework e configura automaticamente
```

**Configuração (`cypress.config.ts`):**

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite',
    },
    specPattern: 'src/**/*.cy.{ts,tsx}',
  },
  e2e: {
    baseUrl: 'http://localhost:5173',
  },
});
```

### 15.3 Teste de componente isolado

```typescript
// src/components/Button.cy.tsx
import { Button } from './Button';

describe('Button', () => {
  it('deve renderizar com texto', () => {
    cy.mount(<Button label="Clique aqui" />);
    cy.get('button').should('have.text', 'Clique aqui');
  });

  it('deve chamar onClick ao clicar', () => {
    const onClick = cy.stub().as('clickHandler');
    cy.mount(<Button label="Salvar" onClick={onClick} />);

    cy.get('button').click();
    cy.get('@clickHandler').should('have.been.calledOnce');
  });

  it('deve estar desabilitado quando disabled=true', () => {
    cy.mount(<Button label="Enviar" disabled />);
    cy.get('button').should('be.disabled');
  });

  it('deve aplicar variante de estilo', () => {
    cy.mount(<Button label="Danger" variant="danger" />);
    cy.get('button').should('have.class', 'btn--danger');
    cy.get('button').should('have.css', 'background-color', 'rgb(220, 53, 69)');
  });
});
```

### 15.4 Component Test vs E2E — quando usar cada um

| Aspecto | Component Test | E2E |
|---------|---------------|-----|
| **Ambiente** | Browser real (Chromium) | Browser real |
| **Escopo** | Componente isolado | Fluxo completo da aplicação |
| **Velocidade** | Rápido (sem servidor) | Lento (requer servidor + navegação) |
| **Fidelidade CSS** | Total (browser real) | Total |
| **Roteamento** | Não disponível | Disponível |
| **Servidor/API** | Stubs e mocks | Intercept ou backend real |
| **Quando usar** | Componentes visuais complexos, interações CSS reais | Fluxos de negócio críticos |

---

## 16. Playwright — Testes E2E

### 16.1 O que é o Playwright?

Playwright é um framework de testes E2E criado pela Microsoft que suporta **múltiplos browsers** (Chromium, Firefox e WebKit) com uma API unificada. Destaca-se pela velocidade, auto-wait inteligente e ferramentas de debug integradas.

### 16.2 Instalação e configuração

```bash
# Instalar Playwright
npm init playwright@latest
# Responder as perguntas do wizard ou usar:
npm install -D @playwright/test
npx playwright install   # baixar os browsers
```

**Configuração (`playwright.config.ts`):**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,       // rodar testes em paralelo
  forbidOnly: !!process.env.CI,  // falhar se alguém esqueceu .only
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],                // relatório HTML interativo
    ['list'],                // saída no terminal
  ],

  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry', // gravar trace apenas em retries
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  // Configurar múltiplos browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    // Testes em dispositivo mobile
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  // Iniciar servidor automaticamente antes dos testes
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

**Scripts no `package.json`:**

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug",
    "test:e2e:report": "playwright show-report"
  }
}
```

### 16.3 Estrutura de testes com `@playwright/test`

```typescript
// tests/e2e/home.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Página Home', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('deve exibir o título da aplicação', async ({ page }) => {
    await expect(page.getByRole('heading', { level: 1 })).toHaveText('Bem-vindo');
  });

  test('deve navegar para produtos', async ({ page }) => {
    await page.getByRole('link', { name: 'Produtos' }).click();
    await expect(page).toHaveURL('/produtos');
    await expect(page.getByRole('heading', { level: 1 })).toHaveText('Produtos');
  });
});
```

### 16.4 Locators — encontrando elementos

Playwright usa **locators** com auto-wait: toda ação espera automaticamente o elemento estar visível e interativo.

```typescript
import { test, expect } from '@playwright/test';

test('locators do Playwright', async ({ page }) => {
  await page.goto('/formulario');

  // Por role (recomendado — mesmo princípio da Testing Library)
  page.getByRole('button', { name: 'Salvar' });
  page.getByRole('textbox', { name: 'Email' });
  page.getByRole('heading', { level: 2 });
  page.getByRole('link', { name: /produtos/i });
  page.getByRole('checkbox', { name: 'Aceito os termos' });
  page.getByRole('combobox', { name: 'Estado' });

  // Por label
  page.getByLabel('Email:');
  page.getByLabel('Senha:');

  // Por placeholder
  page.getByPlaceholder('Digite seu nome');

  // Por texto visível
  page.getByText('Bem-vindo');
  page.getByText(/bem-vindo/i); // regex

  // Por alt text (imagens)
  page.getByAltText('Logo da empresa');

  // Por title
  page.getByTitle('Fechar modal');

  // Por test-id (último recurso)
  page.getByTestId('submit-btn'); // busca data-testid="submit-btn"

  // CSS selector (evitar quando possível)
  page.locator('.btn-primary');
  page.locator('#email');
  page.locator('div.card >> text=Notebook');

  // Encadeamento e filtragem
  page.getByRole('listitem').filter({ hasText: 'Notebook' });
  page.getByRole('listitem').filter({ has: page.getByRole('button', { name: 'Comprar' }) });
  page.getByRole('listitem').nth(2); // terceiro item (0-based)
  page.getByRole('listitem').first();
  page.getByRole('listitem').last();
});
```

### 16.5 Ações

```typescript
test('ações do Playwright', async ({ page }) => {
  await page.goto('/formulario');

  // Clique
  await page.getByRole('button', { name: 'Enviar' }).click();
  await page.getByRole('button', { name: 'Menu' }).dblclick();
  await page.getByRole('link', { name: 'Abrir' }).click({ button: 'right' });

  // Digitação
  await page.getByLabel('Nome:').fill('Ana Silva');     // substitui todo o valor
  await page.getByLabel('Busca:').pressSequentially('React', { delay: 100 }); // digita letra por letra
  await page.getByLabel('Nome:').clear();

  // Seleção em dropdown
  await page.getByLabel('Estado:').selectOption('SP');
  await page.getByLabel('Estado:').selectOption({ label: 'São Paulo' });

  // Checkbox e radio
  await page.getByRole('checkbox', { name: 'Termos' }).check();
  await page.getByRole('checkbox', { name: 'Termos' }).uncheck();
  await page.getByRole('radio', { name: 'Premium' }).check();

  // Teclado
  await page.getByLabel('Busca:').press('Enter');
  await page.keyboard.press('Escape');
  await page.keyboard.press('Control+a');

  // Upload de arquivo
  await page.getByLabel('Foto:').setInputFiles('test-data/foto.jpg');
  await page.getByLabel('Documentos:').setInputFiles(['doc1.pdf', 'doc2.pdf']);

  // Hover
  await page.getByRole('button', { name: 'Menu' }).hover();

  // Drag and drop
  await page.getByTestId('item-1').dragTo(page.getByTestId('drop-zone'));

  // Focus
  await page.getByLabel('Email:').focus();
});
```

### 16.6 Assertions

```typescript
test('assertions do Playwright', async ({ page }) => {
  await page.goto('/');

  // Página
  await expect(page).toHaveURL('/');
  await expect(page).toHaveURL(/\/produtos/);
  await expect(page).toHaveTitle('Minha App');
  await expect(page).toHaveTitle(/minha/i);

  // Elemento — visibilidade
  await expect(page.getByRole('heading')).toBeVisible();
  await expect(page.getByRole('dialog')).toBeHidden();
  await expect(page.getByTestId('removed')).not.toBeAttached();

  // Texto
  await expect(page.getByRole('heading')).toHaveText('Bem-vindo');
  await expect(page.getByRole('heading')).toContainText('Bem');

  // Atributos
  await expect(page.getByLabel('Email:')).toHaveAttribute('type', 'email');
  await expect(page.getByLabel('Email:')).toHaveValue('ana@email.com');
  await expect(page.getByLabel('Email:')).toBeEmpty();

  // Estado
  await expect(page.getByRole('button', { name: 'Salvar' })).toBeEnabled();
  await expect(page.getByRole('button', { name: 'Salvar' })).toBeDisabled();
  await expect(page.getByRole('checkbox')).toBeChecked();
  await expect(page.getByLabel('Nome:')).toBeFocused();
  await expect(page.getByLabel('Email:')).toBeEditable();

  // Classes e CSS
  await expect(page.locator('.card')).toHaveClass(/active/);
  await expect(page.locator('.badge')).toHaveCSS('background-color', 'rgb(0, 128, 0)');

  // Contagem
  await expect(page.getByRole('listitem')).toHaveCount(5);

  // Screenshot comparison (visual regression)
  await expect(page.getByTestId('card')).toHaveScreenshot('card.png');
});
```

### 16.7 Interceptação de rede (`page.route`)

```typescript
test.describe('Interceptação de rede', () => {
  test('deve mockar resposta da API', async ({ page }) => {
    // Interceptar e retornar dados mock
    await page.route('/api/produtos', async (route) => {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([
          { id: 1, nome: 'Notebook', preco: 3500 },
          { id: 2, nome: 'Mouse', preco: 150 },
        ]),
      });
    });

    await page.goto('/produtos');
    await expect(page.getByText('Notebook')).toBeVisible();
    await expect(page.getByText('Mouse')).toBeVisible();
  });

  test('deve simular erro de rede', async ({ page }) => {
    await page.route('/api/produtos', async (route) => {
      await route.abort('failed');
    });

    await page.goto('/produtos');
    await expect(page.getByText('Erro ao carregar')).toBeVisible();
  });

  test('deve modificar resposta do servidor', async ({ page }) => {
    await page.route('/api/usuario', async (route) => {
      const response = await route.fetch(); // busca real
      const json = await response.json();
      json.nome = 'Nome Modificado'; // altera
      await route.fulfill({ response, body: JSON.stringify(json) });
    });

    await page.goto('/perfil');
    await expect(page.getByText('Nome Modificado')).toBeVisible();
  });

  test('deve aguardar uma requisição específica', async ({ page }) => {
    await page.goto('/produtos');

    const responsePromise = page.waitForResponse('/api/produtos');
    await page.getByRole('button', { name: 'Recarregar' }).click();
    const response = await responsePromise;

    expect(response.status()).toBe(200);
  });
});
```

### 16.8 Testando fluxo: Login → Dashboard

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Fluxo de Login', () => {
  test('deve fazer login com credenciais válidas', async ({ page }) => {
    await page.route('/api/auth/login', async (route) => {
      const body = route.request().postDataJSON();
      if (body.email === 'ana@email.com' && body.senha === 'senha123') {
        await route.fulfill({
          status: 200,
          body: JSON.stringify({ token: 'jwt-token', user: { nome: 'Ana', role: 'admin' } }),
        });
      } else {
        await route.fulfill({
          status: 401,
          body: JSON.stringify({ error: 'Credenciais inválidas' }),
        });
      }
    });

    await page.goto('/login');

    await page.getByLabel('Email:').fill('ana@email.com');
    await page.getByLabel('Senha:').fill('senha123');
    await page.getByRole('button', { name: 'Entrar' }).click();

    // Verificar redirecionamento para dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Olá, Ana')).toBeVisible();
  });

  test('deve exibir erro com credenciais inválidas', async ({ page }) => {
    await page.route('/api/auth/login', async (route) => {
      await route.fulfill({
        status: 401,
        body: JSON.stringify({ error: 'Credenciais inválidas' }),
      });
    });

    await page.goto('/login');
    await page.getByLabel('Email:').fill('ana@email.com');
    await page.getByLabel('Senha:').fill('senhaerrada');
    await page.getByRole('button', { name: 'Entrar' }).click();

    await expect(page.getByRole('alert')).toContainText('Credenciais inválidas');
    await expect(page).toHaveURL('/login'); // não redirecionou
  });
});
```

### 16.9 Screenshots, vídeos e traces

```typescript
// Screenshots manuais
test('deve capturar screenshot', async ({ page }) => {
  await page.goto('/');
  await page.screenshot({ path: 'screenshots/home.png', fullPage: true });

  // Screenshot de elemento específico
  await page.getByTestId('hero-banner').screenshot({ path: 'screenshots/hero.png' });
});

// Trace — gravação detalhada para debug
test('trace para debug', async ({ page, context }) => {
  await context.tracing.start({ screenshots: true, snapshots: true, sources: true });

  await page.goto('/');
  await page.getByRole('link', { name: 'Produtos' }).click();

  await context.tracing.stop({ path: 'traces/produtos-trace.zip' });
  // Visualizar: npx playwright show-trace traces/produtos-trace.zip
});
```

**Configuração automática no `playwright.config.ts`:**

```typescript
use: {
  screenshot: 'only-on-failure',     // capturar screenshot só quando falha
  video: 'retain-on-failure',        // manter vídeo só quando falha
  trace: 'on-first-retry',           // gravar trace no primeiro retry
},
```

### 16.10 Paralelismo e sharding

```bash
# Rodar testes em paralelo (padrão)
npx playwright test

# Limitar número de workers
npx playwright test --workers=4

# Sharding para CI (dividir entre máquinas)
# Máquina 1:
npx playwright test --shard=1/3
# Máquina 2:
npx playwright test --shard=2/3
# Máquina 3:
npx playwright test --shard=3/3
```

### 16.11 Page Object Model (POM)

```typescript
// tests/pages/LoginPage.ts
import { type Page, type Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly senhaInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email:');
    this.senhaInput = page.getByLabel('Senha:');
    this.submitButton = page.getByRole('button', { name: 'Entrar' });
    this.errorMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, senha: string) {
    await this.emailInput.fill(email);
    await this.senhaInput.fill(senha);
    await this.submitButton.click();
  }
}
```

```typescript
// tests/pages/DashboardPage.ts
import { type Page, type Locator, expect } from '@playwright/test';

export class DashboardPage {
  readonly page: Page;
  readonly welcomeMessage: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.welcomeMessage = page.getByTestId('welcome-message');
    this.logoutButton = page.getByRole('button', { name: 'Sair' });
  }

  async expectLoggedInAs(nome: string) {
    await expect(this.welcomeMessage).toContainText(nome);
  }

  async logout() {
    await this.logoutButton.click();
  }
}
```

```typescript
// tests/e2e/login-pom.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('Login com POM', () => {
  test('deve fazer login com sucesso', async ({ page }) => {
    await page.route('/api/auth/login', async (route) => {
      await route.fulfill({
        status: 200,
        body: JSON.stringify({ token: 'jwt', user: { nome: 'Ana' } }),
      });
    });

    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.login('ana@email.com', 'senha123');

    await expect(page).toHaveURL('/dashboard');
    await dashboardPage.expectLoggedInAs('Ana');
  });
});
```

---

## 17. Comparativo Prático: Cypress vs Playwright

### 17.1 Mesmo cenário nos dois frameworks

**Cenário:** Testar formulário de contato — preencher campos, submeter, verificar mensagem de sucesso.

**Cypress:**

```typescript
// cypress/e2e/contato.cy.ts
describe('Formulário de Contato', () => {
  beforeEach(() => {
    cy.intercept('POST', '/api/contato', { statusCode: 200, body: { ok: true } }).as('enviar');
    cy.visit('/contato');
  });

  it('deve enviar mensagem com sucesso', () => {
    cy.get('[data-cy="nome"]').type('Ana Silva');
    cy.get('[data-cy="email"]').type('ana@email.com');
    cy.get('[data-cy="assunto"]').select('Suporte');
    cy.get('[data-cy="mensagem"]').type('Preciso de ajuda com meu pedido.');
    cy.get('[data-cy="termos"]').check();
    cy.get('button[type="submit"]').click();

    cy.wait('@enviar');
    cy.get('.toast-sucesso').should('be.visible').and('contain.text', 'Mensagem enviada');
  });
});
```

**Playwright:**

```typescript
// tests/e2e/contato.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Formulário de Contato', () => {
  test.beforeEach(async ({ page }) => {
    await page.route('/api/contato', async (route) => {
      await route.fulfill({ status: 200, body: JSON.stringify({ ok: true }) });
    });
    await page.goto('/contato');
  });

  test('deve enviar mensagem com sucesso', async ({ page }) => {
    await page.getByLabel('Nome:').fill('Ana Silva');
    await page.getByLabel('Email:').fill('ana@email.com');
    await page.getByLabel('Assunto:').selectOption('Suporte');
    await page.getByLabel('Mensagem:').fill('Preciso de ajuda com meu pedido.');
    await page.getByRole('checkbox', { name: /termos/i }).check();
    await page.getByRole('button', { name: 'Enviar' }).click();

    await expect(page.getByText('Mensagem enviada')).toBeVisible();
  });
});
```

### 17.2 Tabela comparativa

| Aspecto | Cypress | Playwright |
|---------|---------|------------|
| **Linguagem** | JavaScript/TypeScript | JavaScript/TypeScript, Python, Java, C# |
| **Browsers** | Chromium (padrão), Firefox, Edge, WebKit (experimental) | Chromium, Firefox, WebKit (completo) |
| **Execução** | Dentro do browser (mesmo processo) | Fora do browser (controle via protocolo) |
| **Auto-wait** | Sim (retry automático em commands) | Sim (auto-wait em locators e actions) |
| **API** | Encadeamento (chaining): `cy.get().click()` | Async/await: `await page.click()` |
| **Múltiplas abas** | Não suporta nativamente | Suporta nativamente |
| **Múltiplos browsers simultâneos** | Não | Sim (útil para chat em tempo real) |
| **iFrames** | Plugin necessário (`cypress-iframe`) | Suporte nativo (`page.frameLocator`) |
| **Interceptação de rede** | `cy.intercept()` | `page.route()` |
| **Screenshots** | Automáticas + manuais | Automáticas + manuais |
| **Vídeo** | Gravação automática | Gravação automática |
| **Trace/debug** | Time-travel no dashboard | Trace viewer com snapshots DOM |
| **Component Testing** | Sim (built-in) | Experimental |
| **Paralelismo** | Cypress Cloud (pago) ou plugins | Nativo e gratuito |
| **Sharding em CI** | Cypress Cloud (pago) | Nativo (`--shard`) |
| **Dashboard/UI** | Cypress Dashboard (interativo) | Playwright UI mode |
| **Velocidade** | Médio | Rápido |
| **CI headless** | Sim | Sim |
| **Comunidade** | Muito grande, muitos plugins | Crescente rápido, boa documentação |
| **Licença** | MIT (Cloud é pago) | Apache 2.0 (tudo gratuito) |

### 17.3 Quando escolher cada um

| Cenário | Recomendação |
|---------|-------------|
| Projeto novo, precisa de multi-browser | **Playwright** |
| Equipe já familiarizada com Cypress | **Cypress** |
| Precisa testar em Safari (WebKit) | **Playwright** |
| Quer dashboard interativo para debug | **Cypress** |
| Precisa de paralelismo gratuito em CI | **Playwright** |
| Precisa de Component Testing no browser | **Cypress** |
| Precisa testar múltiplas abas ou iFrames | **Playwright** |
| Quer test runner completo sem dependências extras | **Playwright** |
| Projeto com muitos plugins Cypress existentes | **Cypress** |

---

## 18. Testes Visuais (Visual Regression)

### 18.1 O que é visual regression testing?

Visual regression testing compara screenshots de componentes ou páginas com uma versão de referência (*baseline*). Se houver diferenças visuais — mesmo que o HTML e CSS estejam tecnicamente corretos — o teste falha, alertando sobre mudanças visuais inesperadas.

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Baseline   │    │   Atual     │    │    Diff     │
│  (salvo)    │ vs │  (novo)     │ →  │ (diferenças)│
│             │    │             │    │  destacadas │
└─────────────┘    └─────────────┘    └─────────────┘
```

**Quando usar:**

- Componentes visuais compartilhados (design system)
- Páginas com layout complexo
- Verificar que mudanças de CSS não quebraram outras partes
- Garantir consistência visual cross-browser

### 18.2 Visual comparisons com Playwright (built-in)

Playwright inclui suporte nativo a visual comparison sem dependências extras.

```typescript
// tests/e2e/visual.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Testes visuais', () => {
  test('página home deve corresponder ao screenshot de referência', async ({ page }) => {
    await page.goto('/');

    // Compara com screenshot salvo em tests/e2e/visual.spec.ts-snapshots/
    await expect(page).toHaveScreenshot('home.png');
  });

  test('componente card deve manter aparência', async ({ page }) => {
    await page.goto('/componentes');

    const card = page.getByTestId('product-card');
    await expect(card).toHaveScreenshot('product-card.png');
  });

  test('formulário em estado de erro', async ({ page }) => {
    await page.goto('/login');
    await page.getByRole('button', { name: 'Entrar' }).click();

    // Capturar screenshot após erros de validação aparecerem
    await expect(page.getByRole('form')).toHaveScreenshot('login-form-errors.png');
  });
});
```

**Atualizando screenshots de referência:**

```bash
# Gerar/atualizar todos os screenshots baseline
npx playwright test --update-snapshots

# Gerar apenas para testes específicos
npx playwright test visual.spec.ts --update-snapshots
```

### 18.3 Configuração de thresholds

```typescript
// Tolerância global no playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,             // máximo de pixels diferentes
      maxDiffPixelRatio: 0.01,        // máximo de 1% de pixels diferentes
      threshold: 0.2,                  // tolerância por pixel (0-1)
      animations: 'disabled',         // desabilitar animações CSS
    },
  },
});
```

```typescript
// Tolerância por teste
test('visual com tolerância customizada', async ({ page }) => {
  await page.goto('/');

  await expect(page).toHaveScreenshot('home.png', {
    maxDiffPixels: 50,
    threshold: 0.3,
    // Mascarar elementos dinâmicos (datas, avatares)
    mask: [
      page.getByTestId('data-atual'),
      page.getByTestId('avatar-usuario'),
    ],
  });
});
```

### 18.4 Boas práticas para testes visuais

| Prática | Motivo |
|---------|--------|
| Desabilitar animações CSS | Animações em andamento geram diffs falsos |
| Mascarar conteúdo dinâmico | Datas, horários, avatares mudam a cada execução |
| Usar viewport fixo | Tamanhos diferentes geram screenshots diferentes |
| Testar em um único browser (CI) | Screenshots diferem entre browsers e OSs |
| Rodar em ambiente consistente (Docker) | Renderização de fontes varia entre sistemas |
| Separar testes visuais dos funcionais | Visuais são mais lentos e frágeis |
| Revisar diffs antes de atualizar baseline | Não executar `--update-snapshots` cegamente |

```typescript
// Exemplo: desabilitar animações e fixar viewport
test.use({
  viewport: { width: 1280, height: 720 },
});

test('visual consistente', async ({ page }) => {
  // Desabilitar animações via CSS
  await page.addStyleTag({
    content: `
      *, *::before, *::after {
        animation-duration: 0s !important;
        transition-duration: 0s !important;
      }
    `,
  });

  await page.goto('/dashboard');

  await expect(page).toHaveScreenshot('dashboard.png', {
    mask: [page.getByTestId('timestamp')],
  });
});
```

### 18.5 Ferramentas alternativas (menção)

| Ferramenta | Tipo | Descrição |
|------------|------|-----------|
| **Percy** (BrowserStack) | SaaS | Renderiza em múltiplos browsers na nuvem, UI de aprovação |
| **Chromatic** (Storybook) | SaaS | Integração nativa com Storybook, captura stories automaticamente |
| **BackstopJS** | Open-source | Comparação de screenshots configurável via JSON |
| **Loki** | Open-source | Visual testing para Storybook |

---

## 19. Testes de Performance e Qualidade com Lighthouse

### 19.1 O que é o Lighthouse?

**Lighthouse** é uma ferramenta open-source do Google que audita páginas web em cinco categorias:

| Categoria | O que avalia | Score ideal |
|-----------|-------------|-------------|
| **Performance** | Velocidade de carregamento, interatividade, estabilidade visual | ≥ 90 |
| **Accessibility** | Conformidade com WCAG, ARIA, semântica HTML | ≥ 90 |
| **Best Practices** | HTTPS, APIs modernas, sem erros no console, imagens corretas | ≥ 90 |
| **SEO** | Meta tags, robots.txt, links rastreáveis, mobile-friendly | ≥ 90 |
| **PWA** | Service worker, manifest, offline capability | Varia |

Lighthouse pode ser executado de várias formas:

- **Chrome DevTools** (aba Lighthouse) — manual, durante desenvolvimento
- **CLI** (`npx lighthouse`) — automação local
- **CI/CD** (`@lhci/cli`) — automação em pipeline
- **PageSpeed Insights** — análise com dados reais (CrUX) + lab data
- **Programaticamente** — via `lighthouse` npm package em testes

### 19.2 Core Web Vitals

Os **Core Web Vitals** são as métricas de experiência do usuário que o Google utiliza como fator de ranking:

| Métrica | Sigla | O que mede | Bom | Precisa melhorar | Ruim |
|---------|-------|-----------|-----|-------------------|------|
| **Largest Contentful Paint** | LCP | Tempo até o maior elemento visível ser renderizado | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **Interaction to Next Paint** | INP | Latência de interações (clique, toque, teclado) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **Cumulative Layout Shift** | CLS | Estabilidade visual — quanto o layout "pula" | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

Outras métricas importantes do Lighthouse:

| Métrica | Sigla | O que mede |
|---------|-------|-----------|
| **First Contentful Paint** | FCP | Tempo até o primeiro conteúdo ser pintado |
| **Time to Interactive** | TTI | Tempo até a página ser totalmente interativa |
| **Total Blocking Time** | TBT | Soma do tempo que a thread principal ficou bloqueada |
| **Speed Index** | SI | Quão rápido o conteúdo é visualmente populado |

### 19.3 Lighthouse via CLI

```bash
# Instalar
npm install -g lighthouse

# Executar auditoria completa
npx lighthouse https://localhost:3000 --output html --output-path ./lighthouse-report.html

# Apenas categorias específicas
npx lighthouse https://localhost:3000 \
  --only-categories=performance,accessibility \
  --output json \
  --output-path ./lighthouse.json

# Modo mobile (padrão) vs desktop
npx lighthouse https://localhost:3000 --preset=desktop

# Com throttling customizado
npx lighthouse https://localhost:3000 \
  --throttling.cpuSlowdownMultiplier=2 \
  --throttling.downloadThroughputKbps=5000
```

### 19.4 Lighthouse CI (LHCI) — Automação em pipelines

O **Lighthouse CI** permite executar Lighthouse em pipelines de CI/CD, definir thresholds e comparar resultados entre builds.

```bash
npm install -D @lhci/cli
```

**Configuração (`lighthouserc.js`):**

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      // URL da aplicação (pode ser startServerCommand para subir localmente)
      startServerCommand: 'npm run preview',
      startServerReadyPattern: 'Local',
      startServerReadyTimeout: 30000,

      url: [
        'http://localhost:4173/',
        'http://localhost:4173/login',
        'http://localhost:4173/produtos',
      ],

      numberOfRuns: 3,  // múltiplas execuções para reduzir variância

      settings: {
        preset: 'desktop',
        // Ou configuração mobile customizada
        // formFactor: 'mobile',
        // screenEmulation: { mobile: true, width: 375, height: 667 },
      },
    },

    assert: {
      assertions: {
        // Performance
        'categories:performance': ['warn', { minScore: 0.8 }],
        'first-contentful-paint': ['warn', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 4000 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['warn', { maxNumericValue: 300 }],

        // Acessibilidade
        'categories:accessibility': ['error', { minScore: 0.9 }],

        // Boas práticas
        'categories:best-practices': ['warn', { minScore: 0.9 }],

        // SEO
        'categories:seo': ['warn', { minScore: 0.9 }],

        // Auditorias específicas
        'is-on-https': 'off',            // desligar para localhost
        'uses-responsive-images': 'warn',
        'dom-size': ['warn', { maxNumericValue: 1500 }],
      },
    },

    upload: {
      // Salvar relatórios localmente
      target: 'filesystem',
      outputDir: './lighthouse-reports',
      // Ou usar Lighthouse CI Server para histórico
      // target: 'lhci',
      // serverBaseUrl: 'https://lhci.exemplo.com',
    },
  },
};
```

**Executando:**

```bash
# Coletar dados do Lighthouse
npx lhci collect

# Verificar thresholds
npx lhci assert

# Upload relatórios
npx lhci upload

# Tudo junto
npx lhci autorun
```

### 19.5 Lighthouse programático em testes

É possível executar Lighthouse como parte de testes Playwright para verificar métricas de performance automaticamente:

```typescript
// tests/e2e/performance.spec.ts
import { test, expect, chromium } from '@playwright/test';
import lighthouse from 'lighthouse';
import { playAudit } from 'playwright-lighthouse';

test.describe('Lighthouse — Performance audit', () => {
  test('página home deve ter scores mínimos', async () => {
    const browser = await chromium.launch({
      args: ['--remote-debugging-port=9222'],
    });
    const page = await browser.newPage();
    await page.goto('http://localhost:3000');

    await playAudit({
      page,
      port: 9222,
      thresholds: {
        performance: 80,
        accessibility: 90,
        'best-practices': 90,
        seo: 90,
      },
    });

    await browser.close();
  });
});
```

**Alternativa usando lighthouse diretamente:**

```typescript
// tests/lighthouse/audit.test.ts
import { describe, it, expect } from 'vitest';
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

describe('Lighthouse audit', () => {
  it('deve atingir scores mínimos na home', async () => {
    const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });

    const result = await lighthouse('http://localhost:3000', {
      port: chrome.port,
      output: 'json',
      onlyCategories: ['performance', 'accessibility', 'best-practices', 'seo'],
      formFactor: 'desktop',
      screenEmulation: { disabled: true },
    });

    const { categories } = result!.lhr;

    expect(categories.performance.score! * 100).toBeGreaterThanOrEqual(80);
    expect(categories.accessibility.score! * 100).toBeGreaterThanOrEqual(90);
    expect(categories['best-practices'].score! * 100).toBeGreaterThanOrEqual(90);
    expect(categories.seo.score! * 100).toBeGreaterThanOrEqual(90);

    await chrome.kill();
  }, 60000);  // timeout longo — Lighthouse leva ~15-30s

  it('deve ter LCP menor que 2.5s', async () => {
    const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });

    const result = await lighthouse('http://localhost:3000', {
      port: chrome.port,
      output: 'json',
      onlyCategories: ['performance'],
    });

    const lcp = result!.lhr.audits['largest-contentful-paint'];
    expect(lcp.numericValue).toBeLessThan(2500);

    const cls = result!.lhr.audits['cumulative-layout-shift'];
    expect(cls.numericValue).toBeLessThan(0.1);

    const tbt = result!.lhr.audits['total-blocking-time'];
    expect(tbt.numericValue).toBeLessThan(300);

    await chrome.kill();
  }, 60000);
});
```

### 19.6 Medindo Web Vitals no código da aplicação

O pacote `web-vitals` permite capturar métricas reais dos usuários (*Real User Monitoring*) e também pode ser usado em testes E2E:

```typescript
// src/reportWebVitals.ts
import { onCLS, onINP, onLCP, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics(metric: { name: string; value: number; rating: string }) {
  console.log(`[Web Vital] ${metric.name}: ${metric.value} (${metric.rating})`);
  // Enviar para serviço de analytics
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);
```

**Capturando Web Vitals em teste Playwright:**

```typescript
// tests/e2e/web-vitals.spec.ts
import { test, expect } from '@playwright/test';

test('deve ter Core Web Vitals aceitáveis', async ({ page }) => {
  // Injetar coleta de métricas
  await page.addInitScript(() => {
    (window as any).__webVitals = {};

    new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'largest-contentful-paint') {
          (window as any).__webVitals.lcp = entry.startTime;
        }
      }
    }).observe({ type: 'largest-contentful-paint', buffered: true });

    new PerformanceObserver((list) => {
      let cls = 0;
      for (const entry of list.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          cls += (entry as any).value;
        }
      }
      (window as any).__webVitals.cls = cls;
    }).observe({ type: 'layout-shift', buffered: true });
  });

  await page.goto('http://localhost:3000');
  await page.waitForLoadState('networkidle');

  // Aguardar métricas serem coletadas
  await page.waitForTimeout(3000);

  const vitals = await page.evaluate(() => (window as any).__webVitals);

  // Verificar LCP
  if (vitals.lcp !== undefined) {
    expect(vitals.lcp).toBeLessThan(2500);
  }

  // Verificar CLS
  if (vitals.cls !== undefined) {
    expect(vitals.cls).toBeLessThan(0.1);
  }
});
```

### 19.7 GitHub Actions com Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.js'
          uploadArtifacts: true     # salvar relatórios como artifacts
          temporaryPublicStorage: true  # link público temporário

      # Alternativa: rodar manualmente com lhci
      # - run: |
      #     npm install -g @lhci/cli
      #     lhci autorun
```

**Workflow combinado (testes + Lighthouse):**

```yaml
# .github/workflows/quality.yml
name: Quality Gate

on:
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npx vitest run --coverage

  lighthouse:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm run build
      - uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.js'
          uploadArtifacts: true
```

### 19.8 Ferramentas complementares ao Lighthouse

| Ferramenta | Foco | Uso |
|------------|------|-----|
| **PageSpeed Insights** | Performance com dados reais (CrUX) | Análise manual, dados de campo |
| **WebPageTest** | Performance detalhada (waterfall, filmstrip) | Análise profunda, comparação entre deploys |
| **Unlighthouse** | Lighthouse em todo o site (crawling) | Auditar múltiplas páginas automaticamente |
| **web-vitals** | Métricas RUM no código | Monitoramento em produção |
| **SpeedCurve** | Monitoramento contínuo de performance | Alertas e dashboards de longo prazo |
| **Calibre** | Monitoramento de performance | Testes agendados com alertas |

**Unlighthouse — auditar um site inteiro:**

```bash
# Instalar e executar
npx unlighthouse --site https://meusite.com

# Apenas categorias específicas
npx unlighthouse --site https://localhost:3000 \
  --only-categories performance,accessibility

# Gerar relatório estático
npx unlighthouse --site https://meusite.com --build-static
```

### 19.9 Boas práticas para testes de performance

| Prática | Motivo |
|---------|--------|
| Executar Lighthouse com `numberOfRuns: 3-5` | Reduz variância entre execuções |
| Usar modo `desktop` para CI | Resultados mais estáveis que mobile emulado |
| Definir budgets progressivos | Comece com thresholds alcançáveis e aumente gradualmente |
| Separar testes de performance dos funcionais | Performance testa a build final, não código-fonte |
| Executar após `npm run build` | Lighthouse avalia o artefato de produção |
| Monitorar tendências, não valores absolutos | Um score de 85 que era 90 é mais importante que um 85 estável |
| Desativar extensões e plugins no CI | Extensões de browser interferem nos resultados |
| Tratar `accessibility: error`, `performance: warn` | Acessibilidade é requisito; performance é meta |

---

## 20. Cobertura de Código e Métricas de Testes

### 21.1 O que é cobertura de código?

Cobertura de código (*code coverage*) mede quanto do código-fonte é executado durante os testes. É uma métrica quantitativa — alta cobertura não garante qualidade dos testes, mas baixa cobertura indica áreas não testadas.

### 21.2 Tipos de cobertura

| Métrica | O que mede | Exemplo |
|---------|-----------|---------|
| **Statements** | % de instruções executadas | `const x = 1;` foi executada? |
| **Branches** | % de ramificações (if/else) executadas | O `else` foi testado? |
| **Functions** | % de funções chamadas | `calcularDesconto()` foi chamada? |
| **Lines** | % de linhas executadas | Similar a statements, mas por linha |

```
Exemplo de coverage parcial:

function calcularPreco(valor: number, desconto: boolean): number {
  if (desconto) {           // ← branch 1 (testado ✓)
    return valor * 0.9;     // ← line testada ✓
  } else {                  // ← branch 2 (NÃO testado ✗)
    return valor;           // ← line NÃO testada ✗
  }
}

// Teste existente:
test('com desconto', () => {
  expect(calcularPreco(100, true)).toBe(90);
});
// Coverage: Statements 75%, Branches 50%, Functions 100%, Lines 75%
```

### 21.3 Configuração de coverage no Vitest

```bash
npm install -D @vitest/coverage-v8
```

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov', 'json-summary'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: [
        'src/**/*.test.{ts,tsx}',
        'src/**/*.spec.{ts,tsx}',
        'src/**/*.d.ts',
        'src/test/**',
        'src/main.tsx',
        'src/vite-env.d.ts',
      ],
      thresholds: {
        statements: 80,
        branches: 75,
        functions: 80,
        lines: 80,
      },
    },
  },
});
```

```bash
# Gerar relatório de cobertura
npx vitest run --coverage

# Saída no terminal:
# ----------|---------|----------|---------|---------|-------------------
# File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
# ----------|---------|----------|---------|---------|-------------------
# All files |   87.5  |    82.1  |   90.0  |   87.5  |
#  Button   |  100.0  |   100.0  |  100.0  |  100.0  |
#  utils    |   75.0  |    66.7  |   80.0  |   75.0  | 23-28, 45
# ----------|---------|----------|---------|---------|-------------------

# Relatório HTML (mais detalhado)
# Abrir coverage/index.html no browser
```

### 21.4 Configuração de coverage no Jest

```typescript
// jest.config.ts
const config: Config = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'html', 'lcov'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.test.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/main.tsx',
  ],
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 75,
      functions: 80,
      lines: 80,
    },
  },
};
```

```bash
npx jest --coverage
```

### 21.5 Interpretando relatórios

O relatório HTML (`coverage/index.html`) permite navegar arquivo por arquivo, vendo:

- **Linhas verdes:** executadas pelos testes
- **Linhas vermelhas:** não executadas
- **Linhas amarelas:** parcialmente executadas (ex: ternário com apenas um lado testado)
- **Números na margem:** quantas vezes cada linha foi executada

**O que a cobertura NÃO diz:**

| Cobertura alta NÃO significa | Cobertura baixa significa |
|------------------------------|--------------------------|
| Que os testes são bons | Que há código não testado |
| Que não há bugs | Que riscos não foram mitigados |
| Que os edge cases foram cobertos | Que pode haver regressões ocultas |
| Que a lógica está correta | Que a confiança para refatorar é menor |

### 21.6 Metas de cobertura recomendadas

| Tipo de código | Meta | Justificativa |
|---------------|------|---------------|
| **Utilitários/helpers** | 90-100% | Funções puras, fáceis de testar |
| **Lógica de negócio** | 80-90% | Crítico, vale o investimento |
| **Componentes UI** | 70-80% | Testar comportamento, não estilo |
| **Código de configuração** | 50-60% | Setup, bootstrapping — menos valor |
| **Código gerado** | Excluir | Auto-gerado não precisa de teste |

### 21.7 Mutation testing com Stryker (menção)

Mutation testing vai além da cobertura: modifica o código-fonte (*mutantes*) e verifica se os testes detectam a mudança. Se um mutante sobrevive (testes continuam passando), significa que os testes não são eficazes o suficiente.

```bash
npm install -D @stryker-mutator/core @stryker-mutator/vitest-runner
npx stryker init    # wizard de configuração
npx stryker run     # executa mutation testing
```

```
Exemplo de mutação:
Original:  return a + b;
Mutante 1: return a - b;    ← se testes passam, estão fracos
Mutante 2: return a * b;    ← se testes passam, estão fracos
Mutante 3: return 0;        ← se testes passam, estão fracos
```

| Métrica | Significado |
|---------|------------|
| **Mutation Score** | % de mutantes que os testes mataram |
| **Killed** | Mutante detectado (teste falhou — bom!) |
| **Survived** | Mutante NÃO detectado (teste não pegou — ruim) |
| **Timeout** | Mutante causou loop infinito |
| **No Coverage** | Código sem nenhum teste |

---

## 21. Organização, Boas Práticas e CI/CD

### 21.1 Estrutura de pastas de teste

Existem duas abordagens principais para organizar arquivos de teste:

**Abordagem 1 — Co-located (recomendada para componentes React):**

Os testes ficam ao lado do código que testam.

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx       ← teste ao lado do componente
│   │   └── Button.module.css
│   └── Card/
│       ├── Card.tsx
│       └── Card.test.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useAuth.test.ts
├── utils/
│   ├── validators.ts
│   └── validators.test.ts
└── test/
    ├── setup.ts                   ← configuração global
    └── helpers/
        └── renderWithProviders.tsx ← helpers de teste compartilhados
```

**Abordagem 2 — Diretório separado (comum para E2E):**

```
src/                               ← código-fonte
tests/
├── unit/                          ← testes unitários
│   └── validators.test.ts
├── integration/                   ← testes de componente/integração
│   └── LoginForm.test.tsx
└── e2e/                           ← testes E2E
    ├── login.spec.ts
    └── pages/                     ← Page Objects
        └── LoginPage.ts
```

**Abordagem mista (recomendada):**

```
src/
├── components/
│   ├── Button.tsx
│   └── Button.test.tsx            ← unitários/componente co-located
├── utils/
│   ├── math.ts
│   └── math.test.ts               ← unitários co-located
└── test/
    └── setup.ts
tests/
└── e2e/                           ← E2E separados
    ├── login.spec.ts
    └── checkout.spec.ts
cypress/                            ← (se usar Cypress)
    └── e2e/
```

### 21.2 Nomenclatura de arquivos de teste

| Convenção | Exemplo | Uso comum |
|-----------|---------|-----------|
| `.test.ts` / `.test.tsx` | `Button.test.tsx` | Vitest, Jest (padrão) |
| `.spec.ts` / `.spec.tsx` | `Button.spec.tsx` | Vitest, Jest (alternativa) |
| `.cy.ts` / `.cy.tsx` | `Button.cy.tsx` | Cypress Component Testing |
| `.spec.ts` (em `tests/e2e/`) | `login.spec.ts` | Playwright E2E |
| `.e2e.ts` | `login.e2e.ts` | Convenção alternativa para E2E |

> **Dica:** Use `.test.ts` para unitários/componente e `.spec.ts` para E2E — facilita separar nos scripts.

### 21.3 Padrão AAA (Arrange, Act, Assert)

O padrão AAA organiza cada teste em três fases claras:

```typescript
it('deve adicionar produto ao carrinho', async () => {
  // ARRANGE — preparar o cenário
  const user = userEvent.setup();
  const onAdd = vi.fn();
  render(<ProductCard produto={notebook} onAdd={onAdd} />);

  // ACT — executar a ação
  await user.click(screen.getByRole('button', { name: 'Adicionar' }));

  // ASSERT — verificar o resultado
  expect(onAdd).toHaveBeenCalledWith(notebook.id);
});
```

**Padrão alternativo: Given-When-Then (BDD):**

```typescript
describe('Carrinho de compras', () => {
  describe('dado que o carrinho está vazio', () => {
    it('quando adiciona um produto, então deve exibir 1 item', async () => {
      // Given
      const user = userEvent.setup();
      render(<Cart />);
      expect(screen.getByText('Carrinho vazio')).toBeInTheDocument();

      // When
      await user.click(screen.getByRole('button', { name: 'Adicionar Notebook' }));

      // Then
      expect(screen.getByText('1 item no carrinho')).toBeInTheDocument();
    });
  });
});
```

### 21.4 `data-testid` vs queries semânticas

| Abordagem | Quando usar | Exemplo |
|-----------|-------------|---------|
| **`getByRole`** | Sempre que possível (primeira escolha) | `getByRole('button', { name: 'Salvar' })` |
| **`getByLabelText`** | Campos de formulário com label | `getByLabelText('Email:')` |
| **`getByText`** | Texto visível estático | `getByText('Bem-vindo')` |
| **`data-testid`** | Elemento sem role ou texto identificável | `getByTestId('sidebar-toggle')` |

```typescript
// PREFIRA queries semânticas:
screen.getByRole('button', { name: 'Salvar' });     // acessível
screen.getByLabelText('Email:');                      // acessível

// USE data-testid apenas como último recurso:
screen.getByTestId('loading-spinner');                // quando não há role/texto
```

### 21.5 Testes determinísticos — evitar flaky tests

Flaky tests são testes que ora passam, ora falham sem mudança no código. São extremamente prejudiciais à confiança na suite de testes.

| Causa | Solução |
|-------|---------|
| **Dependência de tempo** | Usar `vi.useFakeTimers()` e `vi.setSystemTime()` |
| **Dependência de ordem de execução** | Cada teste deve ser independente (setup no `beforeEach`) |
| **Chamadas de rede reais** | Sempre mockar com `vi.mock`, `cy.intercept` ou `page.route` |
| **Animações CSS** | Desabilitar animações em testes E2E |
| **Dados aleatórios** | Usar seeds fixas ou dados determinísticos |
| **Race conditions** | Usar `waitFor`, `findBy`, ou `await` correto |
| **Estado global compartilhado** | Limpar estado entre testes (`localStorage.clear()`, `store.reset()`) |
| **Dependência de DOM não limpo** | RTL faz `cleanup` automático; verificar em testes DOM puros |

```typescript
// RUIM: depende do horário real
it('deve exibir saudação da manhã', () => {
  render(<Greeting />);
  expect(screen.getByText('Bom dia')).toBeInTheDocument(); // falha à noite
});

// BOM: controla o horário
it('deve exibir saudação da manhã', () => {
  vi.setSystemTime(new Date('2025-06-15T09:00:00'));
  render(<Greeting />);
  expect(screen.getByText('Bom dia')).toBeInTheDocument();
});
```

```typescript
// RUIM: depende de timeout arbitrário
it('deve carregar dados', async () => {
  render(<UserList />);
  await new Promise(r => setTimeout(r, 2000)); // espera fixa
  expect(screen.getByText('Ana')).toBeInTheDocument();
});

// BOM: espera pela condição
it('deve carregar dados', async () => {
  render(<UserList />);
  expect(await screen.findByText('Ana')).toBeInTheDocument();
});
```

### 21.6 GitHub Actions para testes frontend

**Workflow para testes unitários e de componente:**

```yaml
# .github/workflows/tests.yml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit & Component Tests
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Run tests with coverage
        run: npx vitest run --coverage

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  e2e-tests:
    name: E2E Tests (Playwright)
    runs-on: ubuntu-latest
    needs: unit-tests

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Run E2E tests
        run: npx playwright test --project=chromium

      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/
```

**Workflow para Cypress:**

```yaml
  e2e-cypress:
    name: E2E Tests (Cypress)
    runs-on: ubuntu-latest
    needs: unit-tests

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Cypress run
        uses: cypress-io/github-action@v6
        with:
          build: npm run build
          start: npm run preview
          wait-on: 'http://localhost:4173'
          browser: chrome

      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-screenshots
          path: cypress/screenshots/

      - name: Upload videos
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-videos
          path: cypress/videos/
```

### 21.7 Dicas de performance em CI

| Dica | Impacto |
|------|---------|
| **Cache de `node_modules`** | Reduz instalação de 60s para 5s |
| **`npm ci` em vez de `npm install`** | Instalação limpa e mais rápida |
| **Rodar unitários antes de E2E** | Falhas rápidas economizam tempo |
| **Paralelismo Playwright (`--shard`)** | Divide testes entre runners |
| **Instalar apenas browsers necessários** | `--with-deps chromium` em vez de todos |
| **Não gravar vídeo por padrão** | `video: 'retain-on-failure'` |
| **Limitar workers em CI** | `workers: 1` evita OOM em runners pequenos |
| **Cache de browsers do Playwright** | Evita redownload a cada build |

```yaml
# Cache de browsers do Playwright
- name: Cache Playwright browsers
  uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ hashFiles('package-lock.json') }}
```

### 21.8 Resumo — stack recomendada por tipo de projeto

**Projeto HTML/CSS/JS puro:**

| Camada | Ferramenta |
|--------|-----------|
| Unit tests | Vitest + JSDOM |
| DOM tests | Vitest + JSDOM |
| E2E | Playwright |
| Acessibilidade | vitest-axe + axe-core |
| Performance/Qualidade | Lighthouse CI |
| Coverage | @vitest/coverage-v8 |

**Projeto React (Vite):**

| Camada | Ferramenta |
|--------|-----------|
| Unit tests | Vitest |
| Component tests | Vitest + React Testing Library |
| Hook tests | Vitest + renderHook |
| API mocking | MSW (Mock Service Worker) |
| Data fetching | TanStack Query + MSW |
| Snapshot | Vitest (inline snapshots) |
| E2E | Playwright (ou Cypress) |
| Visual regression | Playwright screenshots |
| Responsividade | Playwright (múltiplos devices) |
| Acessibilidade | vitest-axe + @axe-core/playwright |
| Segurança | DOMPurify + testes XSS/CSP |
| Performance/Qualidade | Lighthouse CI + web-vitals |
| Documentação/Stories | Storybook + Test Runner |
| Coverage | @vitest/coverage-v8 |
| CI/CD | GitHub Actions |

**Projeto React (CRA/Webpack existente):**

| Camada | Ferramenta |
|--------|-----------|
| Unit/Component | Jest + React Testing Library |
| API mocking | MSW (Mock Service Worker) |
| E2E | Cypress ou Playwright |
| Acessibilidade | jest-axe + cypress-axe |
| Performance/Qualidade | Lighthouse CI |
| Coverage | Jest built-in |

---

## 22. MSW (Mock Service Worker) — Mocking de API na Camada de Rede

### 22.1 O que é o MSW?

O **Mock Service Worker (MSW)** intercepta requisições HTTP na camada de rede usando Service Workers (no browser) ou interceptadores Node.js (em testes). Diferente de `vi.mock('fetch')` ou `jest.fn()`, o MSW permite que o código real de fetch/axios execute normalmente — a requisição é interceptada antes de chegar ao servidor.

```
┌───────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Código App   │ ──→ │  fetch()     │ ──→ │  MSW Handler    │
│  (inalterado) │     │  (real)      │     │  (intercepta)   │
└───────────────┘     └──────────────┘     └─────────────────┘
                                                    │
                                            ┌───────▼───────┐
                                            │  Response mock │
                                            │  (controlada)  │
                                            └───────────────┘
```

**Vantagens sobre mock direto de fetch:**

| Aspecto | `vi.mock('fetch')` / `jest.fn()` | MSW |
|---------|----------------------------------|-----|
| **Nível de interceptação** | Substitui a função global | Intercepta na camada de rede |
| **Código da aplicação** | Precisa que fetch seja importável/mockável | Nenhuma alteração necessária |
| **Compatibilidade** | Apenas fetch ou apenas axios | Qualquer cliente HTTP (fetch, axios, ky, got) |
| **Reutilização** | Definido por teste/arquivo | Handlers compartilhados entre testes, Storybook e dev |
| **Realismo** | Baixo — pula headers, status, serialização | Alto — resposta completa com status, headers, delay |
| **Browser + Node** | Apenas Node (JSDOM) | Ambos (Service Worker no browser, interceptador no Node) |

### 22.2 Instalação e configuração

```bash
npm install -D msw
```

**Definindo handlers (`src/mocks/handlers.ts`):**

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse, delay } from 'msw';

// Tipagem dos dados
interface Produto {
  id: number;
  nome: string;
  preco: number;
  categoria: string;
}

interface Usuario {
  id: number;
  nome: string;
  email: string;
  role: 'admin' | 'user';
}

// Dados mock reutilizáveis
export const produtosMock: Produto[] = [
  { id: 1, nome: 'Notebook Dell', preco: 4500, categoria: 'Eletrônicos' },
  { id: 2, nome: 'Mouse Logitech', preco: 150, categoria: 'Periféricos' },
  { id: 3, nome: 'Monitor LG 27"', preco: 2200, categoria: 'Eletrônicos' },
];

export const handlers = [
  // GET — listar produtos
  http.get('/api/produtos', async () => {
    await delay(100); // simular latência realista
    return HttpResponse.json(produtosMock);
  }),

  // GET — buscar produto por ID
  http.get('/api/produtos/:id', ({ params }) => {
    const produto = produtosMock.find(p => p.id === Number(params.id));
    if (!produto) {
      return HttpResponse.json(
        { message: 'Produto não encontrado' },
        { status: 404 },
      );
    }
    return HttpResponse.json(produto);
  }),

  // POST — criar produto
  http.post('/api/produtos', async ({ request }) => {
    const body = (await request.json()) as Omit<Produto, 'id'>;
    const novoProduto: Produto = { ...body, id: produtosMock.length + 1 };
    return HttpResponse.json(novoProduto, { status: 201 });
  }),

  // PUT — atualizar produto
  http.put('/api/produtos/:id', async ({ params, request }) => {
    const body = (await request.json()) as Partial<Produto>;
    const produto = produtosMock.find(p => p.id === Number(params.id));
    if (!produto) {
      return HttpResponse.json({ message: 'Não encontrado' }, { status: 404 });
    }
    const atualizado = { ...produto, ...body };
    return HttpResponse.json(atualizado);
  }),

  // DELETE — remover produto
  http.delete('/api/produtos/:id', ({ params }) => {
    const existe = produtosMock.some(p => p.id === Number(params.id));
    if (!existe) {
      return HttpResponse.json({ message: 'Não encontrado' }, { status: 404 });
    }
    return new HttpResponse(null, { status: 204 });
  }),

  // POST — login
  http.post('/api/auth/login', async ({ request }) => {
    const { email, senha } = (await request.json()) as { email: string; senha: string };

    if (email === 'admin@teste.com' && senha === '123456') {
      return HttpResponse.json({
        token: 'fake-jwt-token-xyz',
        usuario: { id: 1, nome: 'Admin', email, role: 'admin' },
      });
    }

    return HttpResponse.json(
      { message: 'Credenciais inválidas' },
      { status: 401 },
    );
  }),

  // GET — rota autenticada
  http.get('/api/perfil', ({ request }) => {
    const auth = request.headers.get('Authorization');
    if (!auth || !auth.startsWith('Bearer ')) {
      return HttpResponse.json({ message: 'Não autorizado' }, { status: 401 });
    }

    return HttpResponse.json({
      id: 1,
      nome: 'Admin',
      email: 'admin@teste.com',
      role: 'admin',
    });
  }),
];
```

### 22.3 Configurando o servidor MSW para testes

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

**Setup global (`src/test/setup.ts`):**

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { server } from '../mocks/server';
import { beforeAll, afterEach, afterAll } from 'vitest';

// Iniciar servidor antes de todos os testes
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Resetar handlers após cada teste (remove overrides)
afterEach(() => server.resetHandlers());

// Fechar servidor após todos os testes
afterAll(() => server.close());
```

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    setupFiles: ['./src/test/setup.ts'],
    environment: 'jsdom',
  },
});
```

A opção `onUnhandledRequest: 'error'` faz o teste falhar se o código fizer uma requisição que não tem handler — útil para detectar chamadas de API inesperadas.

### 22.4 Usando MSW em testes de componentes React

```typescript
// src/components/ProdutoList.tsx
import { useEffect, useState } from 'react';

interface Produto {
  id: number;
  nome: string;
  preco: number;
  categoria: string;
}

export function ProdutoList() {
  const [produtos, setProdutos] = useState<Produto[]>([]);
  const [loading, setLoading] = useState(true);
  const [erro, setErro] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/produtos')
      .then(res => {
        if (!res.ok) throw new Error('Erro ao carregar');
        return res.json();
      })
      .then(data => setProdutos(data))
      .catch(err => setErro(err.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>Carregando...</p>;
  if (erro) return <p role="alert">Erro: {erro}</p>;

  return (
    <ul>
      {produtos.map(p => (
        <li key={p.id}>
          {p.nome} — R$ {p.preco.toFixed(2)}
        </li>
      ))}
    </ul>
  );
}
```

```typescript
// src/components/ProdutoList.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { ProdutoList } from './ProdutoList';

describe('ProdutoList com MSW', () => {
  it('deve exibir lista de produtos da API', async () => {
    render(<ProdutoList />);

    // Estado de loading
    expect(screen.getByText('Carregando...')).toBeInTheDocument();

    // Aguardar dados carregarem (handlers padrão respondem)
    await waitFor(() => {
      expect(screen.getByText(/Notebook Dell/)).toBeInTheDocument();
    });

    expect(screen.getByText(/Mouse Logitech/)).toBeInTheDocument();
    expect(screen.getByText(/Monitor LG/)).toBeInTheDocument();
  });

  it('deve exibir erro quando API falha', async () => {
    // Override do handler apenas para este teste
    server.use(
      http.get('/api/produtos', () => {
        return HttpResponse.json(
          { message: 'Erro interno' },
          { status: 500 },
        );
      }),
    );

    render(<ProdutoList />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Erro: Erro ao carregar');
    });
  });

  it('deve exibir lista vazia quando não há produtos', async () => {
    server.use(
      http.get('/api/produtos', () => {
        return HttpResponse.json([]);
      }),
    );

    render(<ProdutoList />);

    await waitFor(() => {
      expect(screen.queryByText('Carregando...')).not.toBeInTheDocument();
    });

    expect(screen.queryByRole('listitem')).not.toBeInTheDocument();
  });
});
```

### 22.5 Testando formulários com POST/PUT

```typescript
// src/components/ProdutoForm.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { ProdutoForm } from './ProdutoForm';

describe('ProdutoForm — criação via MSW', () => {
  it('deve enviar dados e exibir sucesso', async () => {
    const user = userEvent.setup();
    const onSuccess = vi.fn();

    render(<ProdutoForm onSuccess={onSuccess} />);

    await user.type(screen.getByLabelText('Nome:'), 'Teclado Mecânico');
    await user.type(screen.getByLabelText('Preço:'), '350');
    await user.selectOptions(screen.getByLabelText('Categoria:'), 'Periféricos');

    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    await waitFor(() => {
      expect(onSuccess).toHaveBeenCalledWith(
        expect.objectContaining({
          nome: 'Teclado Mecânico',
          preco: 350,
          categoria: 'Periféricos',
        }),
      );
    });
  });

  it('deve exibir erro de validação do servidor', async () => {
    server.use(
      http.post('/api/produtos', () => {
        return HttpResponse.json(
          { errors: { nome: 'Nome já existe' } },
          { status: 422 },
        );
      }),
    );

    const user = userEvent.setup();
    render(<ProdutoForm onSuccess={() => {}} />);

    await user.type(screen.getByLabelText('Nome:'), 'Duplicado');
    await user.type(screen.getByLabelText('Preço:'), '100');
    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    await waitFor(() => {
      expect(screen.getByText('Nome já existe')).toBeInTheDocument();
    });
  });
});
```

### 22.6 Cenários avançados — delay, rede e autenticação

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import { http, HttpResponse, delay } from 'msw';
import { server } from '../mocks/server';

describe('Cenários avançados com MSW', () => {
  it('deve exibir loading durante requisição lenta', async () => {
    server.use(
      http.get('/api/produtos', async () => {
        await delay(2000);
        return HttpResponse.json([]);
      }),
    );

    render(<ProdutoList />);
    expect(screen.getByText('Carregando...')).toBeInTheDocument();

    // Após delay, loading desaparece
    await waitFor(() => {
      expect(screen.queryByText('Carregando...')).not.toBeInTheDocument();
    }, { timeout: 3000 });
  });

  it('deve simular erro de rede (conexão falhou)', async () => {
    server.use(
      http.get('/api/produtos', () => {
        return HttpResponse.error(); // simula network error
      }),
    );

    render(<ProdutoList />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toBeInTheDocument();
    });
  });

  it('deve enviar token de autenticação', async () => {
    let receivedAuth = '';

    server.use(
      http.get('/api/perfil', ({ request }) => {
        receivedAuth = request.headers.get('Authorization') || '';
        return HttpResponse.json({ id: 1, nome: 'Admin' });
      }),
    );

    render(<PerfilPage token="meu-token-jwt" />);

    await waitFor(() => {
      expect(receivedAuth).toBe('Bearer meu-token-jwt');
    });
  });

  it('deve interceptar e inspecionar body enviado', async () => {
    let bodyRecebido: Record<string, unknown> = {};

    server.use(
      http.post('/api/produtos', async ({ request }) => {
        bodyRecebido = (await request.json()) as Record<string, unknown>;
        return HttpResponse.json({ id: 99, ...bodyRecebido }, { status: 201 });
      }),
    );

    render(<ProdutoForm onSuccess={() => {}} />);

    const user = userEvent.setup();
    await user.type(screen.getByLabelText('Nome:'), 'Webcam HD');
    await user.type(screen.getByLabelText('Preço:'), '200');
    await user.click(screen.getByRole('button', { name: 'Salvar' }));

    await waitFor(() => {
      expect(bodyRecebido).toEqual(
        expect.objectContaining({ nome: 'Webcam HD' }),
      );
    });
  });
});
```

### 22.7 MSW com query parameters e paginação

```typescript
// src/mocks/handlers.ts (adicionando handler de busca com filtros)
import { http, HttpResponse } from 'msw';
import { produtosMock } from './data';

export const searchHandlers = [
  http.get('/api/produtos/busca', ({ request }) => {
    const url = new URL(request.url);
    const q = url.searchParams.get('q')?.toLowerCase() || '';
    const categoria = url.searchParams.get('categoria');
    const page = Number(url.searchParams.get('page') || '1');
    const limit = Number(url.searchParams.get('limit') || '10');

    let resultados = produtosMock;

    if (q) {
      resultados = resultados.filter(p => p.nome.toLowerCase().includes(q));
    }
    if (categoria) {
      resultados = resultados.filter(p => p.categoria === categoria);
    }

    const total = resultados.length;
    const inicio = (page - 1) * limit;
    const paginados = resultados.slice(inicio, inicio + limit);

    return HttpResponse.json({
      data: paginados,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    });
  }),
];
```

```typescript
// src/components/ProdutoBusca.test.tsx
describe('Busca de produtos com filtros', () => {
  it('deve filtrar por termo de busca', async () => {
    const user = userEvent.setup();
    render(<ProdutoBusca />);

    await user.type(screen.getByLabelText('Buscar:'), 'notebook');
    await user.click(screen.getByRole('button', { name: 'Buscar' }));

    await waitFor(() => {
      expect(screen.getByText(/Notebook Dell/)).toBeInTheDocument();
      expect(screen.queryByText(/Mouse Logitech/)).not.toBeInTheDocument();
    });
  });

  it('deve paginar resultados', async () => {
    const user = userEvent.setup();
    render(<ProdutoBusca />);

    // Página 1
    await waitFor(() => {
      expect(screen.getByText('Página 1 de 2')).toBeInTheDocument();
    });

    await user.click(screen.getByRole('button', { name: 'Próxima' }));

    await waitFor(() => {
      expect(screen.getByText('Página 2 de 2')).toBeInTheDocument();
    });
  });
});
```

### 22.8 Comparativo — quando usar MSW vs mock direto

| Cenário | Mock direto (`vi.fn()`) | MSW |
|---------|------------------------|-----|
| **Função utilitária com fetch interno** | ✓ Simples e rápido | Desnecessário |
| **Componente React com fetch** | Funciona, mas frágil | ✓ Mais realista |
| **Testar headers, status, CORS** | Difícil | ✓ Suporte nativo |
| **Múltiplos endpoints num fluxo** | Complexo de orquestrar | ✓ Handlers declarativos |
| **Reutilizar mocks no Storybook** | Não reutilizável | ✓ Mesmos handlers |
| **Testar retry/timeout** | Complicado | ✓ `delay()` e `HttpResponse.error()` |
| **Setup rápido em teste simples** | ✓ Menos config | Overhead de setup |

---

## 23. Testes com Storybook — Documentação Viva e Interaction Tests

### 23.1 O que é o Storybook?

**Storybook** é uma ferramenta para desenvolver, documentar e testar componentes de UI de forma isolada. Cada "story" representa um estado específico de um componente, criando uma galeria visual e interativa.

```
┌─────────────────────────────────────────────┐
│  Storybook                                  │
│  ┌──────────┐  ┌──────────────────────────┐ │
│  │ Sidebar  │  │ Canvas                   │ │
│  │          │  │ ┌──────────────────────┐  │ │
│  │ Button   │  │ │  Componente isolado  │  │ │
│  │  ├ Primary│  │ │  (renderizado)       │  │ │
│  │  ├ Disabled│ │ │                      │  │ │
│  │  └ Loading│  │ └──────────────────────┘  │ │
│  │ Card     │  │                            │ │
│  │ Modal    │  │ Controls | Actions | A11y  │ │
│  └──────────┘  └──────────────────────────┘ │
│                                             │
└─────────────────────────────────────────────┘
```

**Por que testar com Storybook?**

| Benefício | Descrição |
|-----------|-----------|
| **Documentação viva** | Stories são exemplos reais que nunca ficam desatualizados |
| **Testes visuais** | Chromatic/Percy capturam screenshots de cada story automaticamente |
| **Interaction tests** | `play` functions permitem testar interações diretamente na story |
| **Teste de acessibilidade** | Addon `@storybook/addon-a11y` roda axe-core em cada story |
| **Reutilização com MSW** | Handlers MSW funcionam no Storybook para simular APIs |

### 23.2 Instalação e configuração

```bash
# Instalar Storybook em projeto existente (React + Vite)
npx storybook@latest init

# Instalar addons úteis para testes
npm install -D @storybook/addon-a11y @storybook/test @storybook/addon-coverage
```

**Estrutura gerada:**

```
.storybook/
├── main.ts        ← configuração principal
├── preview.ts     ← decorators e parâmetros globais
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx    ← stories do componente
│   │   └── Button.test.tsx
```

### 23.3 Escrevendo stories — Component Story Format (CSF3)

```typescript
// src/components/Button/Button.tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
}

export function Button({
  label,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  onClick,
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
    >
      {loading ? 'Carregando...' : label}
    </button>
  );
}
```

```typescript
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],  // gera documentação automática
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
      description: 'Estilo visual do botão',
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
    },
    onClick: { action: 'clicked' },  // loga no painel Actions
  },
  args: {
    label: 'Botão',
    variant: 'primary',
    size: 'md',
    disabled: false,
    loading: false,
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Story padrão — usa args do meta
export const Primary: Story = {};

// Variantes
export const Secondary: Story = {
  args: {
    variant: 'secondary',
    label: 'Secundário',
  },
};

export const Danger: Story = {
  args: {
    variant: 'danger',
    label: 'Excluir',
  },
};

// Estados
export const Disabled: Story = {
  args: {
    disabled: true,
    label: 'Indisponível',
  },
};

export const Loading: Story = {
  args: {
    loading: true,
    label: 'Salvar',
  },
};

// Tamanhos lado a lado
export const AllSizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
      <Button label="Pequeno" size="sm" />
      <Button label="Médio" size="md" />
      <Button label="Grande" size="lg" />
    </div>
  ),
};
```

### 23.4 Interaction tests com play functions

Play functions permitem escrever testes de interação diretamente nas stories. Esses testes rodam no browser real do Storybook e também podem ser executados em CI com o Storybook Test Runner.

```typescript
// src/components/LoginForm/LoginForm.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, userEvent, expect, fn, waitFor } from '@storybook/test';
import { LoginForm } from './LoginForm';

const meta = {
  title: 'Components/LoginForm',
  component: LoginForm,
  args: {
    onSubmit: fn(),  // função mock rastreável
  },
} satisfies Meta<typeof LoginForm>;

export default meta;
type Story = StoryObj<typeof meta>;

// Story com estado vazio (padrão)
export const Empty: Story = {};

// Story com interaction test: preenchimento e submit
export const FilledAndSubmitted: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);

    // Preencher campos
    await userEvent.type(
      canvas.getByLabelText('Email:'),
      'usuario@teste.com',
    );
    await userEvent.type(
      canvas.getByLabelText('Senha:'),
      'senha123',
    );

    // Submeter
    await userEvent.click(canvas.getByRole('button', { name: 'Entrar' }));

    // Verificar que onSubmit foi chamado
    await waitFor(() => {
      expect(args.onSubmit).toHaveBeenCalledWith({
        email: 'usuario@teste.com',
        senha: 'senha123',
      });
    });
  },
};

// Story com interaction test: validação de campos vazios
export const ValidationErrors: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    // Submeter sem preencher
    await userEvent.click(canvas.getByRole('button', { name: 'Entrar' }));

    // Verificar mensagens de erro
    await waitFor(() => {
      expect(canvas.getByText('Email é obrigatório')).toBeInTheDocument();
      expect(canvas.getByText('Senha é obrigatória')).toBeInTheDocument();
    });
  },
};

// Story com interaction test: email inválido
export const InvalidEmail: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    await userEvent.type(canvas.getByLabelText('Email:'), 'email-invalido');
    await userEvent.type(canvas.getByLabelText('Senha:'), '123456');
    await userEvent.click(canvas.getByRole('button', { name: 'Entrar' }));

    await waitFor(() => {
      expect(canvas.getByText('Email inválido')).toBeInTheDocument();
    });
  },
};
```

### 23.5 Storybook Test Runner — executando stories como testes em CI

O **Test Runner** executa todas as play functions como testes automatizados, usando Playwright por baixo:

```bash
npm install -D @storybook/test-runner

# Executar (requer Storybook rodando)
npx storybook dev --port 6006 &
npx test-storybook --url http://localhost:6006

# Com coverage
npx test-storybook --coverage
```

**Configuração (`test-runner.ts`):**

```typescript
// .storybook/test-runner.ts
import type { TestRunnerConfig } from '@storybook/test-runner';
import { checkA11y, injectAxe } from 'axe-playwright';

const config: TestRunnerConfig = {
  // Executar acessibilidade em TODAS as stories automaticamente
  async preVisit(page) {
    await injectAxe(page);
  },
  async postVisit(page) {
    await checkA11y(page, '#storybook-root', {
      detailedReport: true,
      detailedReportOptions: {
        html: true,
      },
    });
  },
};

export default config;
```

```bash
# Com verificação de acessibilidade em todas as stories
npm install -D axe-playwright
npx test-storybook
```

### 23.6 Stories com MSW — simulando APIs no Storybook

```typescript
// .storybook/preview.ts — configurar MSW globalmente
import { initialize, mswLoader } from 'msw-storybook-addon';

// Inicializar MSW no Storybook
initialize();

export default {
  loaders: [mswLoader],
};
```

```bash
# Instalar addon MSW para Storybook
npm install -D msw-storybook-addon

# Gerar service worker público
npx msw init public/ --save
```

```typescript
// src/components/ProdutoList/ProdutoList.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, expect, waitFor } from '@storybook/test';
import { http, HttpResponse, delay } from 'msw';
import { ProdutoList } from './ProdutoList';
import { produtosMock } from '../../mocks/handlers';

const meta = {
  title: 'Components/ProdutoList',
  component: ProdutoList,
} satisfies Meta<typeof ProdutoList>;

export default meta;
type Story = StoryObj<typeof meta>;

// Story com dados normais
export const ComDados: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('/api/produtos', () => {
          return HttpResponse.json(produtosMock);
        }),
      ],
    },
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    await waitFor(() => {
      expect(canvas.getByText(/Notebook Dell/)).toBeInTheDocument();
    });
    expect(canvas.getAllByRole('listitem')).toHaveLength(3);
  },
};

// Story com loading lento
export const Loading: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('/api/produtos', async () => {
          await delay('infinite'); // nunca resolve — mostra loading
          return HttpResponse.json([]);
        }),
      ],
    },
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    expect(canvas.getByText('Carregando...')).toBeInTheDocument();
  },
};

// Story com erro de API
export const Erro: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('/api/produtos', () => {
          return HttpResponse.json(
            { message: 'Serviço indisponível' },
            { status: 503 },
          );
        }),
      ],
    },
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await waitFor(() => {
      expect(canvas.getByRole('alert')).toBeInTheDocument();
    });
  },
};

// Story com lista vazia
export const Vazio: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('/api/produtos', () => {
          return HttpResponse.json([]);
        }),
      ],
    },
  },
};
```

### 23.7 Addon de acessibilidade (@storybook/addon-a11y)

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(ts|tsx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',    // painel de acessibilidade
  ],
  framework: '@storybook/react-vite',
};

export default config;
```

```typescript
// Configurar regras de acessibilidade por story
export const FormularioCadastro: Story = {
  parameters: {
    a11y: {
      config: {
        rules: [
          { id: 'color-contrast', enabled: true },
          { id: 'label', enabled: true },
          { id: 'heading-order', enabled: true },
        ],
      },
    },
  },
};

// Desabilitar verificação para story específica (ex: componente em desenvolvimento)
export const EmDesenvolvimento: Story = {
  parameters: {
    a11y: { disable: true },
  },
};
```

### 23.8 Boas práticas para stories como testes

| Prática | Motivo |
|---------|--------|
| Uma story por estado relevante do componente | Facilita debug e documentação |
| Usar `play` para interações, não apenas estados estáticos | Testa comportamento real |
| Reutilizar handlers MSW entre stories e testes unitários | Consistência nos mocks |
| Usar `tags: ['autodocs']` no meta | Documentação gerada automaticamente |
| Configurar addon-a11y globalmente | Toda story verifica acessibilidade |
| Rodar Test Runner no CI | Stories não testadas em CI são apenas demos |
| Usar `fn()` para callbacks, não arrow functions vazias | `fn()` é rastreável nos testes |
| Nomear stories como cenários de uso | `FilledAndSubmitted` > `Story1` |

---

## 24. Testes com TanStack Query (React Query)

### 24.1 O que é o TanStack Query?

**TanStack Query** (anteriormente React Query) é a biblioteca mais popular para gerenciamento de estado assíncrono (data fetching, caching, sincronização) em aplicações React. Testar componentes que usam TanStack Query requer atenção especial ao `QueryClient`, cache, estados de loading/error e revalidação.

```
┌───────────────┐     ┌──────────────────┐     ┌────────────────┐
│  Componente   │ ──→ │  useQuery /       │ ──→ │  API / Backend │
│  React        │     │  useMutation      │     │  (ou MSW mock) │
└───────────────┘     └──────────────────┘     └────────────────┘
                            │
                      ┌─────▼──────┐
                      │ QueryClient│
                      │  (cache)   │
                      └────────────┘
```

### 24.2 Setup para testes

Cada teste deve usar um `QueryClient` isolado para evitar vazamento de cache entre testes:

```typescript
// src/test/query-utils.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { render, type RenderOptions } from '@testing-library/react';
import type { ReactElement } from 'react';

export function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,       // não tentar de novo em testes
        gcTime: Infinity,   // não limpar cache durante o teste
      },
      mutations: {
        retry: false,
      },
    },
    logger: {
      log: console.log,
      warn: console.warn,
      error: () => {},      // silenciar erros esperados em testes
    },
  });
}

export function renderWithQuery(
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>,
) {
  const queryClient = createTestQueryClient();

  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...options }),
    queryClient,
  };
}
```

### 24.3 Testando useQuery — carregamento de dados

```typescript
// src/hooks/useProdutos.ts
import { useQuery } from '@tanstack/react-query';

interface Produto {
  id: number;
  nome: string;
  preco: number;
}

async function fetchProdutos(): Promise<Produto[]> {
  const res = await fetch('/api/produtos');
  if (!res.ok) throw new Error('Erro ao carregar produtos');
  return res.json();
}

export function useProdutos() {
  return useQuery({
    queryKey: ['produtos'],
    queryFn: fetchProdutos,
  });
}
```

```typescript
// src/hooks/useProdutos.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { useProdutos } from './useProdutos';
import { createTestQueryClient } from '../test/query-utils';

function createWrapper() {
  const queryClient = createTestQueryClient();
  return function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  };
}

describe('useProdutos', () => {
  it('deve retornar lista de produtos', async () => {
    const { result } = renderHook(() => useProdutos(), {
      wrapper: createWrapper(),
    });

    // Estado inicial: loading
    expect(result.current.isLoading).toBe(true);
    expect(result.current.data).toBeUndefined();

    // Aguardar dados
    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toHaveLength(3);
    expect(result.current.data![0].nome).toBe('Notebook Dell');
  });

  it('deve retornar erro quando API falha', async () => {
    server.use(
      http.get('/api/produtos', () => {
        return HttpResponse.json(null, { status: 500 });
      }),
    );

    const { result } = renderHook(() => useProdutos(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error).toBeDefined();
    expect(result.current.error!.message).toBe('Erro ao carregar produtos');
  });
});
```

### 24.4 Testando componentes com useQuery

```typescript
// src/components/ProdutoList.tsx
import { useProdutos } from '../hooks/useProdutos';

export function ProdutoList() {
  const { data: produtos, isLoading, isError, error, refetch } = useProdutos();

  if (isLoading) return <p>Carregando produtos...</p>;

  if (isError) {
    return (
      <div role="alert">
        <p>Erro: {error.message}</p>
        <button onClick={() => refetch()}>Tentar novamente</button>
      </div>
    );
  }

  if (!produtos || produtos.length === 0) {
    return <p>Nenhum produto encontrado.</p>;
  }

  return (
    <ul aria-label="Lista de produtos">
      {produtos.map(p => (
        <li key={p.id}>
          <strong>{p.nome}</strong> — R$ {p.preco.toFixed(2)}
        </li>
      ))}
    </ul>
  );
}
```

```typescript
// src/components/ProdutoList.test.tsx
import { describe, it, expect } from 'vitest';
import { screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { renderWithQuery } from '../test/query-utils';
import { ProdutoList } from './ProdutoList';

describe('ProdutoList com TanStack Query', () => {
  it('deve exibir loading e depois os dados', async () => {
    renderWithQuery(<ProdutoList />);

    expect(screen.getByText('Carregando produtos...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText(/Notebook Dell/)).toBeInTheDocument();
    });

    const items = screen.getAllByRole('listitem');
    expect(items).toHaveLength(3);
  });

  it('deve exibir erro com botão de retry', async () => {
    server.use(
      http.get('/api/produtos', () => {
        return HttpResponse.json(null, { status: 500 });
      }),
    );

    renderWithQuery(<ProdutoList />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toBeInTheDocument();
    });

    expect(screen.getByText(/Erro/)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Tentar novamente' })).toBeInTheDocument();
  });

  it('deve recarregar dados ao clicar em "Tentar novamente"', async () => {
    const user = userEvent.setup();
    let chamada = 0;

    server.use(
      http.get('/api/produtos', () => {
        chamada++;
        if (chamada === 1) {
          return HttpResponse.json(null, { status: 500 });
        }
        return HttpResponse.json([
          { id: 1, nome: 'Produto Recuperado', preco: 100 },
        ]);
      }),
    );

    renderWithQuery(<ProdutoList />);

    // Primeiro: erro
    await waitFor(() => {
      expect(screen.getByRole('alert')).toBeInTheDocument();
    });

    // Clicar em retry
    await user.click(screen.getByRole('button', { name: 'Tentar novamente' }));

    // Segundo: sucesso
    await waitFor(() => {
      expect(screen.getByText(/Produto Recuperado/)).toBeInTheDocument();
    });
  });

  it('deve exibir mensagem quando lista está vazia', async () => {
    server.use(
      http.get('/api/produtos', () => HttpResponse.json([])),
    );

    renderWithQuery(<ProdutoList />);

    await waitFor(() => {
      expect(screen.getByText('Nenhum produto encontrado.')).toBeInTheDocument();
    });
  });
});
```

### 24.5 Testando useMutation — criação e atualização

```typescript
// src/hooks/useCriarProduto.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

interface NovoProduto {
  nome: string;
  preco: number;
  categoria: string;
}

interface Produto extends NovoProduto {
  id: number;
}

async function criarProduto(dados: NovoProduto): Promise<Produto> {
  const res = await fetch('/api/produtos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(dados),
  });
  if (!res.ok) {
    const error = await res.json();
    throw new Error(error.message || 'Erro ao criar produto');
  }
  return res.json();
}

export function useCriarProduto() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: criarProduto,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['produtos'] });
    },
  });
}
```

```typescript
// src/hooks/useCriarProduto.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, waitFor, act } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { useCriarProduto } from './useCriarProduto';
import { createTestQueryClient } from '../test/query-utils';
import { QueryClientProvider } from '@tanstack/react-query';

describe('useCriarProduto', () => {
  function createWrapper() {
    const queryClient = createTestQueryClient();
    return {
      queryClient,
      Wrapper: ({ children }: { children: React.ReactNode }) => (
        <QueryClientProvider client={queryClient}>
          {children}
        </QueryClientProvider>
      ),
    };
  }

  it('deve criar produto com sucesso', async () => {
    const { Wrapper } = createWrapper();
    const { result } = renderHook(() => useCriarProduto(), {
      wrapper: Wrapper,
    });

    act(() => {
      result.current.mutate({
        nome: 'Teclado RGB',
        preco: 450,
        categoria: 'Periféricos',
      });
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(
      expect.objectContaining({
        nome: 'Teclado RGB',
        preco: 450,
      }),
    );
  });

  it('deve invalidar cache de produtos após criação', async () => {
    const { queryClient, Wrapper } = createWrapper();

    // Pré-popular cache
    queryClient.setQueryData(['produtos'], [
      { id: 1, nome: 'Existente', preco: 100 },
    ]);

    const { result } = renderHook(() => useCriarProduto(), {
      wrapper: Wrapper,
    });

    act(() => {
      result.current.mutate({
        nome: 'Novo Produto',
        preco: 200,
        categoria: 'Geral',
      });
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    // Cache de produtos deve ter sido invalidado
    const state = queryClient.getQueryState(['produtos']);
    expect(state?.isInvalidated).toBe(true);
  });

  it('deve tratar erro de criação', async () => {
    server.use(
      http.post('/api/produtos', () => {
        return HttpResponse.json(
          { message: 'Nome do produto já existe' },
          { status: 409 },
        );
      }),
    );

    const { Wrapper } = createWrapper();
    const { result } = renderHook(() => useCriarProduto(), {
      wrapper: Wrapper,
    });

    act(() => {
      result.current.mutate({
        nome: 'Duplicado',
        preco: 100,
        categoria: 'Geral',
      });
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error!.message).toBe('Nome do produto já existe');
  });
});
```

### 24.6 Testando useQuery com parâmetros dinâmicos

```typescript
// src/hooks/useProduto.ts
import { useQuery } from '@tanstack/react-query';

export function useProduto(id: number) {
  return useQuery({
    queryKey: ['produto', id],
    queryFn: async () => {
      const res = await fetch(`/api/produtos/${id}`);
      if (!res.ok) throw new Error('Produto não encontrado');
      return res.json();
    },
    enabled: id > 0,  // não executar com id inválido
  });
}
```

```typescript
// src/hooks/useProduto.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { useProduto } from './useProduto';

describe('useProduto — query com parâmetros', () => {
  it('deve buscar produto por ID', async () => {
    const { result } = renderHook(() => useProduto(1), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data.nome).toBe('Notebook Dell');
  });

  it('deve retornar 404 para produto inexistente', async () => {
    const { result } = renderHook(() => useProduto(999), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error!.message).toBe('Produto não encontrado');
  });

  it('não deve executar query quando id é inválido (enabled: false)', () => {
    const { result } = renderHook(() => useProduto(0), {
      wrapper: createWrapper(),
    });

    // Query não foi executada
    expect(result.current.fetchStatus).toBe('idle');
    expect(result.current.isLoading).toBe(false);
    expect(result.current.data).toBeUndefined();
  });

  it('deve usar queryKey diferente para IDs diferentes', async () => {
    const wrapper = createWrapper();

    const { result: result1 } = renderHook(() => useProduto(1), { wrapper });
    const { result: result2 } = renderHook(() => useProduto(2), { wrapper });

    await waitFor(() => {
      expect(result1.current.isSuccess).toBe(true);
      expect(result2.current.isSuccess).toBe(true);
    });

    expect(result1.current.data.nome).toBe('Notebook Dell');
    expect(result2.current.data.nome).toBe('Mouse Logitech');
  });
});
```

### 24.7 Testando useInfiniteQuery — scroll infinito

```typescript
// src/hooks/useProdutosInfinitos.ts
import { useInfiniteQuery } from '@tanstack/react-query';

interface PaginatedResponse {
  data: { id: number; nome: string; preco: number }[];
  meta: { page: number; totalPages: number };
}

export function useProdutosInfinitos() {
  return useInfiniteQuery({
    queryKey: ['produtos', 'infinite'],
    queryFn: async ({ pageParam }): Promise<PaginatedResponse> => {
      const res = await fetch(`/api/produtos?page=${pageParam}&limit=10`);
      if (!res.ok) throw new Error('Erro ao carregar');
      return res.json();
    },
    initialPageParam: 1,
    getNextPageParam: (lastPage) => {
      return lastPage.meta.page < lastPage.meta.totalPages
        ? lastPage.meta.page + 1
        : undefined;
    },
  });
}
```

```typescript
// src/hooks/useProdutosInfinitos.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, waitFor, act } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { useProdutosInfinitos } from './useProdutosInfinitos';

describe('useProdutosInfinitos', () => {
  beforeEach(() => {
    server.use(
      http.get('/api/produtos', ({ request }) => {
        const url = new URL(request.url);
        const page = Number(url.searchParams.get('page') || '1');

        const allItems = Array.from({ length: 25 }, (_, i) => ({
          id: i + 1,
          nome: `Produto ${i + 1}`,
          preco: (i + 1) * 10,
        }));

        const limit = 10;
        const start = (page - 1) * limit;
        const data = allItems.slice(start, start + limit);

        return HttpResponse.json({
          data,
          meta: { page, totalPages: 3 },
        });
      }),
    );
  });

  it('deve carregar primeira página', async () => {
    const { result } = renderHook(() => useProdutosInfinitos(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data!.pages).toHaveLength(1);
    expect(result.current.data!.pages[0].data).toHaveLength(10);
    expect(result.current.hasNextPage).toBe(true);
  });

  it('deve carregar próxima página com fetchNextPage', async () => {
    const { result } = renderHook(() => useProdutosInfinitos(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    // Carregar página 2
    act(() => {
      result.current.fetchNextPage();
    });

    await waitFor(() => {
      expect(result.current.data!.pages).toHaveLength(2);
    });

    expect(result.current.data!.pages[1].data[0].nome).toBe('Produto 11');
  });

  it('deve indicar quando não há mais páginas', async () => {
    const { result } = renderHook(() => useProdutosInfinitos(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    // Carregar todas as páginas
    act(() => result.current.fetchNextPage());
    await waitFor(() => expect(result.current.data!.pages).toHaveLength(2));

    act(() => result.current.fetchNextPage());
    await waitFor(() => expect(result.current.data!.pages).toHaveLength(3));

    expect(result.current.hasNextPage).toBe(false);
  });
});
```

### 24.8 Testando Optimistic Updates

```typescript
// src/hooks/useToggleFavorito.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

interface Produto {
  id: number;
  nome: string;
  favorito: boolean;
}

export function useToggleFavorito() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (id: number) => {
      const res = await fetch(`/api/produtos/${id}/favorito`, { method: 'POST' });
      if (!res.ok) throw new Error('Erro ao favoritar');
      return res.json();
    },

    // Optimistic update
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ['produtos'] });

      const previous = queryClient.getQueryData<Produto[]>(['produtos']);

      queryClient.setQueryData<Produto[]>(['produtos'], (old) =>
        old?.map(p => p.id === id ? { ...p, favorito: !p.favorito } : p),
      );

      return { previous };
    },

    // Rollback em caso de erro
    onError: (_err, _id, context) => {
      if (context?.previous) {
        queryClient.setQueryData(['produtos'], context.previous);
      }
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['produtos'] });
    },
  });
}
```

```typescript
// src/hooks/useToggleFavorito.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, waitFor, act } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/server';
import { useToggleFavorito } from './useToggleFavorito';

describe('useToggleFavorito — Optimistic Update', () => {
  it('deve atualizar cache imediatamente (optimistic)', async () => {
    const { queryClient, Wrapper } = createWrapper();

    // Pré-popular cache
    queryClient.setQueryData(['produtos'], [
      { id: 1, nome: 'Notebook', favorito: false },
      { id: 2, nome: 'Mouse', favorito: true },
    ]);

    server.use(
      http.post('/api/produtos/:id/favorito', async () => {
        return HttpResponse.json({ success: true });
      }),
    );

    const { result } = renderHook(() => useToggleFavorito(), {
      wrapper: Wrapper,
    });

    act(() => {
      result.current.mutate(1);
    });

    // Verificar que o cache foi atualizado IMEDIATAMENTE (antes da resposta)
    const cached = queryClient.getQueryData<any[]>(['produtos']);
    expect(cached![0].favorito).toBe(true);  // era false, agora true
    expect(cached![1].favorito).toBe(true);  // não mudou

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });
  });

  it('deve fazer rollback quando API falha', async () => {
    const { queryClient, Wrapper } = createWrapper();

    queryClient.setQueryData(['produtos'], [
      { id: 1, nome: 'Notebook', favorito: false },
    ]);

    server.use(
      http.post('/api/produtos/:id/favorito', () => {
        return HttpResponse.json({ message: 'Erro' }, { status: 500 });
      }),
    );

    const { result } = renderHook(() => useToggleFavorito(), {
      wrapper: Wrapper,
    });

    act(() => {
      result.current.mutate(1);
    });

    // Optimistic: favorito = true
    expect(queryClient.getQueryData<any[]>(['produtos'])![0].favorito).toBe(true);

    // Após erro: rollback para favorito = false
    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    const cached = queryClient.getQueryData<any[]>(['produtos']);
    expect(cached![0].favorito).toBe(false);  // rollback
  });
});
```

### 24.9 Checklist de testes para TanStack Query

| Cenário | O que testar |
|---------|-------------|
| **useQuery — sucesso** | Dados retornados, loading → success |
| **useQuery — erro** | isError, mensagem de erro |
| **useQuery — enabled** | Query não executa quando condição é falsa |
| **useQuery — parâmetros** | QueryKey muda com parâmetros, dados corretos por ID |
| **useMutation — sucesso** | isSuccess, dados retornados, invalidação de cache |
| **useMutation — erro** | isError, mensagem, estado não muda |
| **Optimistic update** | Cache atualizado antes da resposta |
| **Rollback** | Cache restaurado quando mutation falha |
| **Infinite query** | Primeira página, próxima página, hasNextPage |
| **Retry** | Desabilitar em testes (`retry: false`) |
| **Componente** | Loading, dados, erro, lista vazia, retry button |

---

## 25. Testes de Segurança no Frontend

### 25.1 Por que testar segurança no frontend?

Embora a validação principal de segurança deva ocorrer no backend, o frontend é a primeira linha de defesa contra diversas vulnerabilidades. Testes automatizados ajudam a garantir que proteções não sejam removidas acidentalmente durante refatorações.

```
┌─────────────────────────────────────────────────────────────┐
│                  Superfície de Ataque                       │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │     XSS     │  │   CSRF      │  │  Injeção em URL     │ │
│  │(Cross-Site  │  │(Cross-Site  │  │  (Open Redirect,    │ │
│  │ Scripting)  │  │ Request     │  │   javascript:)      │ │
│  │             │  │ Forgery)    │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Exposição   │  │  Clickjack  │  │  Dados sensíveis    │ │
│  │ de dados    │  │             │  │  em localStorage    │ │
│  │ no cliente  │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

| Vulnerabilidade | Risco | Prevenção no frontend |
|----------------|-------|----------------------|
| **XSS (Reflected/Stored)** | Execução de código malicioso | Sanitizar HTML, usar `textContent`, React escapa por padrão |
| **XSS via `dangerouslySetInnerHTML`** | Bypass da proteção do React | Sanitizar com DOMPurify |
| **Open Redirect** | Redirecionar para sites maliciosos | Validar URLs antes de redirecionar |
| **Dados sensíveis no cliente** | Exposição de tokens/senhas | Não armazenar em localStorage, usar httpOnly cookies |
| **Prototype Pollution** | Manipulação do Object.prototype | Validar objetos de input |
| **Clickjacking** | UI spoofing em iframe | Headers `X-Frame-Options`, CSP |

### 25.2 Testando proteção contra XSS

#### 25.2.1 React escapa conteúdo por padrão

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';

function MensagemUsuario({ texto }: { texto: string }) {
  return <p data-testid="mensagem">{texto}</p>;
}

describe('Proteção XSS — React escape padrão', () => {
  it('deve exibir script como texto, não executar', () => {
    const xssPayload = '<script>alert("XSS")</script>';

    render(<MensagemUsuario texto={xssPayload} />);

    const el = screen.getByTestId('mensagem');
    // React escapa automaticamente — script aparece como texto
    expect(el.textContent).toBe(xssPayload);
    expect(el.innerHTML).not.toContain('<script>');
    expect(el.innerHTML).toContain('&lt;script&gt;');
  });

  it('deve escapar atributos com conteúdo malicioso', () => {
    const xssPayload = '" onmouseover="alert(1)" data-x="';

    render(<img alt={xssPayload} src="foto.jpg" />);

    const img = screen.getByRole('img');
    expect(img.getAttribute('alt')).toBe(xssPayload);
    // Não deve criar novos atributos
    expect(img.hasAttribute('onmouseover')).toBe(false);
  });
});
```

#### 25.2.2 Testando dangerouslySetInnerHTML com DOMPurify

```bash
npm install dompurify
npm install -D @types/dompurify
```

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import DOMPurify from 'dompurify';

function HtmlSeguro({ html }: { html: string }) {
  const limpo = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
  });

  return <div data-testid="conteudo" dangerouslySetInnerHTML={{ __html: limpo }} />;
}

describe('dangerouslySetInnerHTML com DOMPurify', () => {
  it('deve permitir tags seguras', () => {
    render(<HtmlSeguro html="<p>Texto <strong>importante</strong></p>" />);

    const el = screen.getByTestId('conteudo');
    expect(el.innerHTML).toBe('<p>Texto <strong>importante</strong></p>');
  });

  it('deve remover tags script', () => {
    render(<HtmlSeguro html='<p>Olá</p><script>alert("xss")</script>' />);

    const el = screen.getByTestId('conteudo');
    expect(el.innerHTML).toBe('<p>Olá</p>');
    expect(el.innerHTML).not.toContain('script');
  });

  it('deve remover event handlers inline', () => {
    render(<HtmlSeguro html='<p onclick="alert(1)">Clique</p>' />);

    const el = screen.getByTestId('conteudo');
    expect(el.innerHTML).toBe('<p>Clique</p>');
    expect(el.innerHTML).not.toContain('onclick');
  });

  it('deve remover tag img com onerror', () => {
    render(<HtmlSeguro html='<img src="x" onerror="alert(1)">' />);

    const el = screen.getByTestId('conteudo');
    expect(el.innerHTML).not.toContain('onerror');
    expect(el.innerHTML).not.toContain('<img');
  });

  it('deve remover javascript: em href', () => {
    render(<HtmlSeguro html='<a href="javascript:alert(1)">Link</a>' />);

    const el = screen.getByTestId('conteudo');
    // DOMPurify remove javascript: URIs
    expect(el.querySelector('a')?.getAttribute('href')).not.toContain('javascript:');
  });

  it('deve remover tags perigosas (iframe, object, embed)', () => {
    const payloads = [
      '<iframe src="https://evil.com"></iframe>',
      '<object data="malware.swf"></object>',
      '<embed src="exploit.swf">',
      '<svg onload="alert(1)">',
    ];

    payloads.forEach(payload => {
      const { container } = render(<HtmlSeguro html={payload} />);
      const el = container.querySelector('[data-testid="conteudo"]')!;
      expect(el.innerHTML).not.toContain('<iframe');
      expect(el.innerHTML).not.toContain('<object');
      expect(el.innerHTML).not.toContain('<embed');
      expect(el.innerHTML).not.toContain('onload');
    });
  });
});
```

### 25.3 Testando validação e sanitização de URLs

```typescript
// src/utils/urlSecurity.ts
export function isUrlSegura(url: string): boolean {
  try {
    const parsed = new URL(url, window.location.origin);
    // Apenas protocolos seguros
    const protocolosPermitidos = ['http:', 'https:', 'mailto:'];
    if (!protocolosPermitidos.includes(parsed.protocol)) return false;

    // Bloquear domínios conhecidos como maliciosos (exemplo)
    const dominiosBloqueados = ['evil.com', 'phishing.net'];
    if (dominiosBloqueados.some(d => parsed.hostname.endsWith(d))) return false;

    return true;
  } catch {
    return false;
  }
}

export function sanitizarRedirectUrl(url: string): string {
  // Apenas permitir redirecionamento para mesma origem ou caminhos relativos
  try {
    const parsed = new URL(url, window.location.origin);
    if (parsed.origin === window.location.origin) {
      return parsed.pathname + parsed.search + parsed.hash;
    }
  } catch {
    // URL inválida
  }
  return '/';  // fallback seguro
}
```

```typescript
// src/utils/urlSecurity.test.ts
import { describe, it, expect } from 'vitest';
import { isUrlSegura, sanitizarRedirectUrl } from './urlSecurity';

describe('isUrlSegura', () => {
  it.each([
    ['https://exemplo.com', true],
    ['http://localhost:3000', true],
    ['mailto:user@exemplo.com', true],
    ['/pagina/interna', true],
    ['./relativo', true],
  ])('deve aceitar URL segura: %s', (url, esperado) => {
    expect(isUrlSegura(url)).toBe(esperado);
  });

  it.each([
    ['javascript:alert(1)', false],
    ['data:text/html,<script>alert(1)</script>', false],
    ['vbscript:msgbox(1)', false],
    ['https://evil.com/phishing', false],
    ['https://sub.phishing.net', false],
  ])('deve rejeitar URL perigosa: %s', (url, esperado) => {
    expect(isUrlSegura(url)).toBe(esperado);
  });

  it('deve rejeitar URLs com encoding para bypass', () => {
    expect(isUrlSegura('java\tscript:alert(1)')).toBe(false);
    expect(isUrlSegura('&#106;avascript:alert(1)')).toBe(false);
  });
});

describe('sanitizarRedirectUrl', () => {
  it('deve manter caminhos da mesma origem', () => {
    expect(sanitizarRedirectUrl('/dashboard')).toBe('/dashboard');
    expect(sanitizarRedirectUrl('/perfil?tab=config')).toBe('/perfil?tab=config');
  });

  it('deve bloquear redirect para outra origem (Open Redirect)', () => {
    expect(sanitizarRedirectUrl('https://evil.com/roubar-dados')).toBe('/');
    expect(sanitizarRedirectUrl('//evil.com')).toBe('/');
  });

  it('deve retornar / para URLs inválidas', () => {
    expect(sanitizarRedirectUrl('javascript:alert(1)')).toBe('/');
    expect(sanitizarRedirectUrl('')).toBe('/');
  });
});
```

### 25.4 Testando armazenamento seguro de dados sensíveis

```typescript
// src/utils/storage.ts
// NUNCA armazene tokens de autenticação no localStorage em produção!
// Use httpOnly cookies gerenciadas pelo servidor.

// Esta classe demonstra armazenamento com cuidados mínimos
// para dados não críticos (preferências, rascunhos).
export class SecureStorage {
  static setItem(key: string, value: unknown): void {
    const dados = JSON.stringify({
      value,
      timestamp: Date.now(),
    });
    sessionStorage.setItem(key, dados);
  }

  static getItem<T>(key: string, maxAgeMs?: number): T | null {
    const raw = sessionStorage.getItem(key);
    if (!raw) return null;

    try {
      const parsed = JSON.parse(raw);
      if (maxAgeMs && Date.now() - parsed.timestamp > maxAgeMs) {
        sessionStorage.removeItem(key);
        return null;
      }
      return parsed.value as T;
    } catch {
      sessionStorage.removeItem(key);
      return null;
    }
  }

  static clear(): void {
    sessionStorage.clear();
  }
}
```

```typescript
// src/utils/storage.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { SecureStorage } from './storage';

describe('SecureStorage', () => {
  beforeEach(() => {
    sessionStorage.clear();
  });

  it('deve armazenar e recuperar dados', () => {
    SecureStorage.setItem('preferencias', { tema: 'escuro' });
    const result = SecureStorage.getItem<{ tema: string }>('preferencias');
    expect(result).toEqual({ tema: 'escuro' });
  });

  it('deve retornar null para chave inexistente', () => {
    expect(SecureStorage.getItem('nao-existe')).toBeNull();
  });

  it('deve expirar dados após maxAge', () => {
    vi.useFakeTimers();

    SecureStorage.setItem('temporario', 'dados');

    // Avançar 2 horas
    vi.advanceTimersByTime(2 * 60 * 60 * 1000);

    // Com maxAge de 1 hora, deve expirar
    const result = SecureStorage.getItem('temporario', 1 * 60 * 60 * 1000);
    expect(result).toBeNull();

    vi.useRealTimers();
  });

  it('deve limpar dados corrompidos sem lançar erro', () => {
    sessionStorage.setItem('corrompido', 'não-é-json{{{');
    expect(SecureStorage.getItem('corrompido')).toBeNull();
    expect(sessionStorage.getItem('corrompido')).toBeNull(); // removido
  });
});

describe('Segurança — dados sensíveis NÃO devem estar no storage', () => {
  it('deve verificar que tokens NÃO estão no localStorage', () => {
    // Simular fluxo de login
    // (renderizar componente de login, autenticar, etc.)

    // Verificação: nenhuma chave sensível no localStorage
    const chavesPerigosas = ['token', 'jwt', 'access_token', 'refresh_token', 'password', 'senha'];
    const todasChaves = Object.keys(localStorage);

    chavesPerigosas.forEach(chave => {
      const encontrada = todasChaves.find(k =>
        k.toLowerCase().includes(chave),
      );
      expect(encontrada).toBeUndefined();
    });
  });

  it('deve verificar que senhas não ficam em variáveis de estado após login', async () => {
    // Após submit do formulário de login,
    // o campo de senha deve ser limpo
    const user = userEvent.setup();
    render(<LoginForm onSubmit={() => {}} />);

    const senhaInput = screen.getByLabelText('Senha:') as HTMLInputElement;
    await user.type(senhaInput, 'minha-senha-secreta');
    await user.click(screen.getByRole('button', { name: 'Entrar' }));

    // Senha deve ser limpa do input após submit
    await waitFor(() => {
      expect(senhaInput.value).toBe('');
    });
  });
});
```

### 25.5 Testando Content Security Policy (CSP) e headers de segurança

```typescript
// tests/e2e/security-headers.spec.ts (Playwright)
import { test, expect } from '@playwright/test';

test.describe('Headers de segurança', () => {
  test('deve retornar headers de segurança corretos', async ({ page }) => {
    const response = await page.goto('/');
    const headers = response!.headers();

    // X-Content-Type-Options
    expect(headers['x-content-type-options']).toBe('nosniff');

    // X-Frame-Options (proteção contra clickjacking)
    expect(headers['x-frame-options']).toMatch(/DENY|SAMEORIGIN/);

    // Strict-Transport-Security (HTTPS obrigatório)
    expect(headers['strict-transport-security']).toBeDefined();

    // Content-Security-Policy
    const csp = headers['content-security-policy'];
    if (csp) {
      // Deve restringir scripts inline (proteção XSS)
      expect(csp).toContain("script-src");
      // Não deve usar 'unsafe-inline' sem nonce
      // expect(csp).not.toContain("'unsafe-inline'");
    }

    // Referrer-Policy
    expect(headers['referrer-policy']).toBeDefined();
  });

  test('não deve carregar recursos de origens não autorizadas', async ({ page }) => {
    const blockedRequests: string[] = [];

    page.on('console', msg => {
      // CSP violations aparecem como erros no console
      if (msg.type() === 'error' && msg.text().includes('Content Security Policy')) {
        blockedRequests.push(msg.text());
      }
    });

    await page.goto('/');

    // Tentar injetar script externo (deve ser bloqueado pelo CSP)
    await page.evaluate(() => {
      const script = document.createElement('script');
      script.src = 'https://evil.com/malware.js';
      document.head.appendChild(script);
    });

    // Verificar que o script externo foi bloqueado
    // (a verificação exata depende da configuração do CSP)
  });
});
```

### 25.6 Testando proteção contra Prototype Pollution

```typescript
// src/utils/safeObject.ts
export function safeMerge<T extends Record<string, unknown>>(
  target: T,
  source: Record<string, unknown>,
): T {
  const result = { ...target };

  for (const key of Object.keys(source)) {
    // Bloquear propriedades perigosas
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      continue;
    }
    (result as Record<string, unknown>)[key] = source[key];
  }

  return result;
}

export function safeJsonParse<T>(json: string): T | null {
  try {
    return JSON.parse(json, (key, value) => {
      if (key === '__proto__' || key === 'constructor') {
        return undefined; // remover chaves perigosas
      }
      return value;
    });
  } catch {
    return null;
  }
}
```

```typescript
// src/utils/safeObject.test.ts
import { describe, it, expect } from 'vitest';
import { safeMerge, safeJsonParse } from './safeObject';

describe('Proteção contra Prototype Pollution', () => {
  it('safeMerge deve ignorar __proto__', () => {
    const target = { nome: 'seguro' };
    const source = { __proto__: { admin: true }, email: 'teste@teste.com' };

    const result = safeMerge(target, source as any);

    expect(result.email).toBe('teste@teste.com');
    expect((result as any).admin).toBeUndefined();
    expect(({} as any).admin).toBeUndefined(); // Object.prototype não foi poluído
  });

  it('safeMerge deve ignorar constructor', () => {
    const target = { nome: 'seguro' };
    const source = { constructor: { prototype: { admin: true } } };

    const result = safeMerge(target, source as any);

    expect(result.constructor).toBe(Object); // constructor original mantido
  });

  it('safeJsonParse deve remover __proto__ do JSON', () => {
    const json = '{"nome": "ok", "__proto__": {"admin": true}}';
    const result = safeJsonParse(json);

    expect(result).toEqual({ nome: 'ok' });
    expect((result as any).__proto__).toBe(Object.prototype);
    expect(({} as any).admin).toBeUndefined();
  });

  it('safeJsonParse deve retornar null para JSON inválido', () => {
    expect(safeJsonParse('não é json')).toBeNull();
    expect(safeJsonParse('')).toBeNull();
  });
});
```

### 25.7 Checklist de testes de segurança frontend

| Verificação | Teste | Ferramenta |
|-------------|-------|-----------|
| **XSS via input de usuário** | Renderizar payloads XSS, verificar que não executa | Vitest + RTL |
| **XSS via dangerouslySetInnerHTML** | Verificar sanitização com DOMPurify | Vitest |
| **Open Redirect** | Validar URLs antes de `window.location` | Vitest |
| **Dados sensíveis no storage** | Verificar ausência de tokens no localStorage | Vitest |
| **Headers de segurança** | Verificar CSP, X-Frame-Options, HSTS | Playwright |
| **Prototype Pollution** | Testar merge/parse com payloads maliciosos | Vitest |
| **CSRF** | Verificar presença de token CSRF em formulários | RTL / Playwright |
| **Clickjacking** | Verificar X-Frame-Options no response | Playwright |
| **Senhas em texto** | Verificar `type="password"` e limpeza após submit | RTL |
| **Console.log com dados sensíveis** | Verificar que logs não expõem tokens/senhas | Vitest (spy em console) |

---

## 26. Testes de Responsividade e Media Queries

### 26.1 Por que testar responsividade?

Aplicações web modernas devem funcionar em telas de celular (~375px) até monitores ultrawide (~2560px). Testes de responsividade garantem que layouts não quebram, elementos permanecem acessíveis e interações funcionam em todos os tamanhos de tela.

```
375px           768px          1024px          1440px         2560px
│  Mobile  │    │  Tablet  │    │  Laptop  │    │ Desktop │    │ Ultra │
│◄────────►│    │◄────────►│    │◄────────►│    │◄───────►│    │◄─────►│
│ 1 coluna │    │ 2 colunas│    │ sidebar  │    │ sidebar │    │ wide  │
│ menu ≡   │    │ menu ≡   │    │ + content│    │ + content│   │ layout│
└──────────┘    └──────────┘    └──────────┘    └─────────┘    └───────┘
```

### 26.2 Testando media queries com JSDOM (limitações)

O JSDOM **não renderiza CSS**, portanto `window.matchMedia` retorna `false` para todas as queries por padrão. É necessário mockar `matchMedia`:

```typescript
// src/test/matchMediaMock.ts
export function mockMatchMedia(query: string, matches: boolean) {
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: vi.fn().mockImplementation((q: string) => ({
      matches: q === query ? matches : false,
      media: q,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    })),
  });
}

// Mock que suporta múltiplas media queries
export function mockMatchMediaMultiple(queries: Record<string, boolean>) {
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: vi.fn().mockImplementation((q: string) => ({
      matches: queries[q] ?? false,
      media: q,
      onchange: null,
      addListener: vi.fn(),
      removeListener: vi.fn(),
      addEventListener: vi.fn(),
      removeEventListener: vi.fn(),
      dispatchEvent: vi.fn(),
    })),
  });
}
```

### 26.3 Testando hooks de responsividade

```typescript
// src/hooks/useMediaQuery.ts
import { useState, useEffect } from 'react';

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() =>
    window.matchMedia(query).matches,
  );

  useEffect(() => {
    const mql = window.matchMedia(query);
    const handler = (e: MediaQueryListEvent) => setMatches(e.matches);

    mql.addEventListener('change', handler);
    setMatches(mql.matches);

    return () => mql.removeEventListener('change', handler);
  }, [query]);

  return matches;
}

// Hook conveniente com breakpoints pré-definidos
export function useBreakpoint() {
  const isMobile = useMediaQuery('(max-width: 767px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1023px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');

  return { isMobile, isTablet, isDesktop };
}
```

```typescript
// src/hooks/useMediaQuery.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useMediaQuery, useBreakpoint } from './useMediaQuery';

describe('useMediaQuery', () => {
  let listeners: Map<string, (e: MediaQueryListEvent) => void>;

  beforeEach(() => {
    listeners = new Map();

    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation((query: string) => {
        const mql = {
          matches: false,
          media: query,
          onchange: null,
          addEventListener: vi.fn((_, handler) => {
            listeners.set(query, handler);
          }),
          removeEventListener: vi.fn((_, handler) => {
            if (listeners.get(query) === handler) {
              listeners.delete(query);
            }
          }),
          dispatchEvent: vi.fn(),
        };
        return mql;
      }),
    });
  });

  it('deve retornar false quando media query não corresponde', () => {
    const { result } = renderHook(() => useMediaQuery('(max-width: 767px)'));
    expect(result.current).toBe(false);
  });

  it('deve atualizar quando media query muda', () => {
    const { result } = renderHook(() => useMediaQuery('(max-width: 767px)'));

    expect(result.current).toBe(false);

    // Simular mudança na media query
    act(() => {
      const handler = listeners.get('(max-width: 767px)');
      handler?.({ matches: true } as MediaQueryListEvent);
    });

    expect(result.current).toBe(true);
  });

  it('deve limpar listener ao desmontar', () => {
    const { unmount } = renderHook(() => useMediaQuery('(max-width: 767px)'));

    expect(listeners.has('(max-width: 767px)')).toBe(true);

    unmount();

    expect(listeners.has('(max-width: 767px)')).toBe(false);
  });
});

describe('useBreakpoint', () => {
  function setupMatchMedia(breakpoint: 'mobile' | 'tablet' | 'desktop') {
    const queries: Record<string, boolean> = {
      '(max-width: 767px)': breakpoint === 'mobile',
      '(min-width: 768px) and (max-width: 1023px)': breakpoint === 'tablet',
      '(min-width: 1024px)': breakpoint === 'desktop',
    };

    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation((q: string) => ({
        matches: queries[q] ?? false,
        media: q,
        onchange: null,
        addEventListener: vi.fn(),
        removeEventListener: vi.fn(),
        dispatchEvent: vi.fn(),
      })),
    });
  }

  it('deve detectar mobile', () => {
    setupMatchMedia('mobile');
    const { result } = renderHook(() => useBreakpoint());

    expect(result.current.isMobile).toBe(true);
    expect(result.current.isTablet).toBe(false);
    expect(result.current.isDesktop).toBe(false);
  });

  it('deve detectar tablet', () => {
    setupMatchMedia('tablet');
    const { result } = renderHook(() => useBreakpoint());

    expect(result.current.isMobile).toBe(false);
    expect(result.current.isTablet).toBe(true);
    expect(result.current.isDesktop).toBe(false);
  });

  it('deve detectar desktop', () => {
    setupMatchMedia('desktop');
    const { result } = renderHook(() => useBreakpoint());

    expect(result.current.isMobile).toBe(false);
    expect(result.current.isTablet).toBe(false);
    expect(result.current.isDesktop).toBe(true);
  });
});
```

### 26.4 Testando componentes responsivos

```typescript
// src/components/Navigation.tsx
import { useBreakpoint } from '../hooks/useMediaQuery';

export function Navigation() {
  const { isMobile } = useBreakpoint();
  const [menuAberto, setMenuAberto] = React.useState(false);

  if (isMobile) {
    return (
      <nav aria-label="Menu principal">
        <button
          aria-expanded={menuAberto}
          aria-controls="mobile-menu"
          onClick={() => setMenuAberto(!menuAberto)}
        >
          ☰ Menu
        </button>
        {menuAberto && (
          <ul id="mobile-menu" role="menu">
            <li role="menuitem"><a href="/">Home</a></li>
            <li role="menuitem"><a href="/produtos">Produtos</a></li>
            <li role="menuitem"><a href="/contato">Contato</a></li>
          </ul>
        )}
      </nav>
    );
  }

  return (
    <nav aria-label="Menu principal">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/produtos">Produtos</a></li>
        <li><a href="/contato">Contato</a></li>
      </ul>
    </nav>
  );
}
```

```typescript
// src/components/Navigation.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Navigation } from './Navigation';

describe('Navigation — responsividade', () => {
  function mockBreakpoint(breakpoint: 'mobile' | 'desktop') {
    const queries: Record<string, boolean> = {
      '(max-width: 767px)': breakpoint === 'mobile',
      '(min-width: 768px) and (max-width: 1023px)': false,
      '(min-width: 1024px)': breakpoint === 'desktop',
    };

    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation((q: string) => ({
        matches: queries[q] ?? false,
        media: q,
        onchange: null,
        addEventListener: vi.fn(),
        removeEventListener: vi.fn(),
        dispatchEvent: vi.fn(),
      })),
    });
  }

  describe('Desktop', () => {
    beforeEach(() => mockBreakpoint('desktop'));

    it('deve exibir todos os links diretamente', () => {
      render(<Navigation />);

      expect(screen.getByRole('link', { name: 'Home' })).toBeVisible();
      expect(screen.getByRole('link', { name: 'Produtos' })).toBeVisible();
      expect(screen.getByRole('link', { name: 'Contato' })).toBeVisible();
    });

    it('não deve exibir botão de hamburger', () => {
      render(<Navigation />);
      expect(screen.queryByRole('button', { name: /menu/i })).not.toBeInTheDocument();
    });
  });

  describe('Mobile', () => {
    beforeEach(() => mockBreakpoint('mobile'));

    it('deve exibir botão de hamburger', () => {
      render(<Navigation />);
      expect(screen.getByRole('button', { name: /menu/i })).toBeInTheDocument();
    });

    it('menu deve estar fechado por padrão', () => {
      render(<Navigation />);
      expect(screen.queryByRole('menu')).not.toBeInTheDocument();
    });

    it('deve abrir menu ao clicar no hamburger', async () => {
      const user = userEvent.setup();
      render(<Navigation />);

      await user.click(screen.getByRole('button', { name: /menu/i }));

      expect(screen.getByRole('menu')).toBeInTheDocument();
      expect(screen.getAllByRole('menuitem')).toHaveLength(3);
    });

    it('deve ter aria-expanded correto no botão', async () => {
      const user = userEvent.setup();
      render(<Navigation />);

      const botao = screen.getByRole('button', { name: /menu/i });
      expect(botao).toHaveAttribute('aria-expanded', 'false');

      await user.click(botao);
      expect(botao).toHaveAttribute('aria-expanded', 'true');
    });

    it('deve fechar menu ao clicar novamente', async () => {
      const user = userEvent.setup();
      render(<Navigation />);

      const botao = screen.getByRole('button', { name: /menu/i });
      await user.click(botao);
      expect(screen.getByRole('menu')).toBeInTheDocument();

      await user.click(botao);
      expect(screen.queryByRole('menu')).not.toBeInTheDocument();
    });
  });
});
```

### 26.5 Testes E2E de responsividade com Playwright

Playwright permite testar em diferentes viewports com CSS real renderizado:

```typescript
// tests/e2e/responsive.spec.ts
import { test, expect } from '@playwright/test';

// Definir breakpoints de teste
const viewports = {
  mobile: { width: 375, height: 812 },      // iPhone 13
  mobileL: { width: 428, height: 926 },     // iPhone 14 Pro Max
  tablet: { width: 768, height: 1024 },     // iPad
  laptop: { width: 1366, height: 768 },     // Laptop comum
  desktop: { width: 1920, height: 1080 },   // Full HD
};

test.describe('Responsividade — Layout', () => {
  test('mobile: deve exibir menu hamburger', async ({ page }) => {
    await page.setViewportSize(viewports.mobile);
    await page.goto('/');

    const hamburger = page.getByRole('button', { name: /menu/i });
    await expect(hamburger).toBeVisible();

    // Links do menu não devem estar visíveis
    const navLinks = page.getByRole('navigation').getByRole('link');
    await expect(navLinks.first()).not.toBeVisible();

    // Abrir menu
    await hamburger.click();
    await expect(navLinks.first()).toBeVisible();
  });

  test('desktop: deve exibir menu horizontal', async ({ page }) => {
    await page.setViewportSize(viewports.desktop);
    await page.goto('/');

    // Não deve ter botão hamburger
    await expect(page.getByRole('button', { name: /menu/i })).not.toBeVisible();

    // Links visíveis diretamente
    await expect(page.getByRole('link', { name: 'Home' })).toBeVisible();
    await expect(page.getByRole('link', { name: 'Produtos' })).toBeVisible();
  });

  test('tablet: grid deve ter 2 colunas', async ({ page }) => {
    await page.setViewportSize(viewports.tablet);
    await page.goto('/produtos');

    // Verificar que o grid tem layout de 2 colunas via CSS computado
    const grid = page.getByTestId('produtos-grid');
    const gridStyle = await grid.evaluate(el =>
      window.getComputedStyle(el).getPropertyValue('grid-template-columns'),
    );

    // Deve ter 2 colunas (ex: "300px 300px" ou "1fr 1fr")
    const colunas = gridStyle.split(' ').filter(v => v.trim().length > 0);
    expect(colunas.length).toBe(2);
  });
});
```

### 26.6 Testando em múltiplos viewports com Playwright projects

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    // Desktop
    {
      name: 'Desktop Chrome',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Desktop Firefox',
      use: { ...devices['Desktop Firefox'] },
    },

    // Tablet
    {
      name: 'iPad',
      use: { ...devices['iPad Pro 11'] },
    },

    // Mobile
    {
      name: 'iPhone 13',
      use: { ...devices['iPhone 13'] },
    },
    {
      name: 'Pixel 7',
      use: { ...devices['Pixel 7'] },
    },

    // Landscape mobile
    {
      name: 'iPhone 13 Landscape',
      use: { ...devices['iPhone 13 landscape'] },
    },
  ],
});
```

```bash
# Executar testes em todos os dispositivos
npx playwright test

# Executar apenas mobile
npx playwright test --project="iPhone 13"

# Executar apenas desktop
npx playwright test --project="Desktop Chrome"
```

### 26.7 Testando orientação e resize dinâmico

```typescript
// tests/e2e/orientation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Orientação e resize', () => {
  test('deve adaptar layout ao rotacionar de portrait para landscape', async ({ page }) => {
    // Portrait (mobile)
    await page.setViewportSize({ width: 375, height: 812 });
    await page.goto('/');

    await expect(page.getByTestId('sidebar')).not.toBeVisible();

    // Rotacionar para landscape
    await page.setViewportSize({ width: 812, height: 375 });

    // Em landscape, layout pode mostrar sidebar ou reorganizar conteúdo
    await expect(page.getByTestId('content')).toBeVisible();
  });

  test('deve adaptar layout ao redimensionar a janela', async ({ page }) => {
    await page.setViewportSize({ width: 1920, height: 1080 });
    await page.goto('/');

    // Desktop: menu horizontal
    await expect(page.getByRole('button', { name: /menu/i })).not.toBeVisible();

    // Redimensionar para mobile
    await page.setViewportSize({ width: 375, height: 812 });

    // Agora deve ter hamburger
    await expect(page.getByRole('button', { name: /menu/i })).toBeVisible();
  });

  test('conteúdo não deve transbordar em tela estreita', async ({ page }) => {
    await page.setViewportSize({ width: 320, height: 568 }); // iPhone SE (menor tela comum)
    await page.goto('/');

    // Verificar que não há scroll horizontal indesejado
    const hasHorizontalScroll = await page.evaluate(() => {
      return document.documentElement.scrollWidth > document.documentElement.clientWidth;
    });

    expect(hasHorizontalScroll).toBe(false);
  });
});
```

### 26.8 Visual regression por viewport

```typescript
// tests/e2e/responsive-visual.spec.ts
import { test, expect } from '@playwright/test';

const breakpoints = [
  { name: 'mobile', width: 375, height: 812 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'desktop', width: 1440, height: 900 },
];

for (const bp of breakpoints) {
  test.describe(`Visual — ${bp.name} (${bp.width}x${bp.height})`, () => {
    test.use({ viewport: { width: bp.width, height: bp.height } });

    test(`home page — ${bp.name}`, async ({ page }) => {
      await page.goto('/');
      await page.waitForLoadState('networkidle');

      await expect(page).toHaveScreenshot(`home-${bp.name}.png`, {
        fullPage: true,
        animations: 'disabled',
      });
    });

    test(`página de produtos — ${bp.name}`, async ({ page }) => {
      await page.goto('/produtos');
      await page.waitForLoadState('networkidle');

      await expect(page).toHaveScreenshot(`produtos-${bp.name}.png`, {
        fullPage: true,
        animations: 'disabled',
      });
    });
  });
}
```

### 26.9 Testando touch events e gestos mobile

```typescript
// tests/e2e/touch.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Touch interactions', () => {
  test.use({
    ...{ hasTouch: true },
    viewport: { width: 375, height: 812 },
  });

  test('deve suportar swipe para abrir sidebar', async ({ page }) => {
    await page.goto('/');

    // Simular swipe da esquerda para direita
    await page.touchscreen.tap(20, 400);  // posição inicial
    await page.mouse.move(20, 400);
    await page.mouse.down();
    await page.mouse.move(300, 400, { steps: 10 });  // arrastar
    await page.mouse.up();

    // Sidebar deve estar visível após swipe
    // (depende da implementação)
  });

  test('deve suportar tap em botões sem hover', async ({ page }) => {
    await page.goto('/produtos');

    // Em dispositivos touch, tap = click
    const botao = page.getByRole('button', { name: 'Adicionar ao carrinho' }).first();
    await botao.tap();

    await expect(page.getByText('Produto adicionado!')).toBeVisible();
  });

  test('deve fechar modal com tap fora', async ({ page }) => {
    await page.goto('/');
    await page.getByRole('button', { name: 'Abrir modal' }).tap();

    const modal = page.getByRole('dialog');
    await expect(modal).toBeVisible();

    // Tap fora do modal (no overlay)
    const overlay = page.getByTestId('modal-overlay');
    await overlay.tap();

    await expect(modal).not.toBeVisible();
  });
});
```

### 26.10 Testando imagens responsivas e lazy loading

```typescript
// tests/e2e/images.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Imagens responsivas', () => {
  test('deve usar srcset para diferentes resoluções', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 812 });
    await page.goto('/produtos');

    const img = page.getByAltText('Foto do produto').first();

    // Verificar que a imagem tem srcset
    const srcset = await img.getAttribute('srcset');
    expect(srcset).toBeTruthy();
    expect(srcset).toContain('375w');
    expect(srcset).toContain('768w');
  });

  test('deve ter lazy loading em imagens abaixo da dobra', async ({ page }) => {
    await page.setViewportSize({ width: 1440, height: 900 });
    await page.goto('/produtos');

    // Imagens abaixo da dobra devem ter loading="lazy"
    const images = page.locator('img[loading="lazy"]');
    const count = await images.count();
    expect(count).toBeGreaterThan(0);

    // Primeira imagem (hero) NÃO deve ser lazy
    const heroImg = page.getByTestId('hero-image');
    await expect(heroImg).not.toHaveAttribute('loading', 'lazy');
  });

  test('deve carregar imagem lazy ao scrollar', async ({ page }) => {
    await page.setViewportSize({ width: 1440, height: 900 });
    await page.goto('/produtos');

    // Última imagem (abaixo da dobra) — ainda não carregada
    const lazyImg = page.locator('img[loading="lazy"]').last();

    // Scrollar até a imagem
    await lazyImg.scrollIntoViewIfNeeded();

    // Aguardar carregamento
    await expect(lazyImg).toHaveJSProperty('complete', true);
    const naturalWidth = await lazyImg.evaluate(el => (el as HTMLImageElement).naturalWidth);
    expect(naturalWidth).toBeGreaterThan(0);
  });
});
```

### 26.11 Boas práticas para testes de responsividade

| Prática | Motivo |
|---------|--------|
| Testar nos 3 breakpoints principais (mobile, tablet, desktop) | Cobre a maioria dos usuários |
| Incluir menor viewport (320px) como edge case | Alguns dispositivos ainda usam 320px |
| Usar `devices` do Playwright em vez de viewports manuais | Inclui user-agent, pixel ratio e touch |
| Verificar ausência de scroll horizontal | Overflow horizontal é o bug responsivo mais comum |
| Testar orientação portrait e landscape | Tablets usam ambas frequentemente |
| Combinar testes visuais com breakpoints | Screenshot por viewport detecta regressões visuais |
| Não depender de `window.innerWidth` no código | Usar CSS media queries e `matchMedia` |
| Testar conteúdo, não implementação CSS | Verificar "menu está visível?" em vez de "display é flex?" |

---
