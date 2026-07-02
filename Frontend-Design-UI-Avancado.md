# Design de UI Avançado — Design Systems, Frameworks CSS, PWA e Acessibilidade

Referência consolidada sobre Design Systems, Design Tokens, Tailwind CSS v4, Bootstrap 5, Material UI (MUI), diretrizes de design mobile (Material Design 3 e Human Interface Guidelines), Progressive Web Apps (PWA) e acessibilidade avançada (WCAG 2.2), com exemplos práticos em React, Angular, Vue, Flutter e React Native.

> Para fundamentos de CSS, Flexbox, Grid, SCSS e acessibilidade básica, consulte [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md).
> Para Service Workers e Cache API em JavaScript puro, consulte [Dicas-Javascript-Typescript.md](Dicas-Javascript-Typescript.md).

---

## Sumário

1. [Design Systems e Design Tokens](#1-design-systems-e-design-tokens)
   - [O que é um Design System](#o-que-é-um-design-system)
   - [Anatomia de um Design System](#anatomia-de-um-design-system)
   - [Design Tokens — Conceito e Taxonomia](#design-tokens--conceito-e-taxonomia)
   - [Tokens de Cor](#tokens-de-cor)
   - [Tokens de Tipografia](#tokens-de-tipografia)
   - [Tokens de Espaçamento](#tokens-de-espaçamento)
   - [Tokens de Elevação (Sombras)](#tokens-de-elevação-sombras)
   - [Tokens de Borda e Raio](#tokens-de-borda-e-raio)
   - [Tokens de Animação e Transição](#tokens-de-animação-e-transição)
   - [Implementação com CSS Custom Properties](#implementação-com-css-custom-properties)
   - [Tokens Semânticos vs Primitivos](#tokens-semânticos-vs-primitivos)
   - [Multi-Tema com Design Tokens](#multi-tema-com-design-tokens)
   - [Arquitetura de Componentes em um Design System](#arquitetura-de-componentes-em-um-design-system)
   - [Consistência Web + Mobile — Tokens Multiplataforma](#consistência-web--mobile--tokens-multiplataforma)
   - [Ferramentas para Gerenciamento de Tokens](#ferramentas-para-gerenciamento-de-tokens)
   - [Storybook — Documentação Interativa de Componentes](#storybook--documentação-interativa-de-componentes)
   - [Padrões de UI — Estados Vazios, Erro e Loading](#padrões-de-ui--estados-vazios-erro-e-loading)
2. [Tailwind CSS v4](#2-tailwind-css-v4)
   - [Filosofia Utility-First](#filosofia-utility-first)
   - [Instalação e Configuração v4](#instalação-e-configuração-v4)
   - [Migração do v3 para v4](#migração-do-v3-para-v4)
   - [Classes Utilitárias Essenciais](#classes-utilitárias-essenciais)
   - [Design Responsivo com Tailwind](#design-responsivo-com-tailwind)
   - [Dark Mode](#dark-mode)
   - [Customização do Tema](#customização-do-tema)
   - [Reutilização com @apply e Componentes](#reutilização-com-apply-e-componentes)
   - [Plugins Oficiais e da Comunidade](#plugins-oficiais-e-da-comunidade)
   - [Tailwind + React](#tailwind--react)
   - [Tailwind + Vue](#tailwind--vue)
   - [Tailwind + Angular](#tailwind--angular)
   - [Tailwind vs CSS Tradicional vs CSS-in-JS](#tailwind-vs-css-tradicional-vs-css-in-js)
3. [Bootstrap 5](#3-bootstrap-5)
   - [Visão Geral e Filosofia](#visão-geral-e-filosofia)
   - [Instalação e Configuração](#instalação-e-configuração)
   - [Sistema de Grid](#sistema-de-grid)
   - [Componentes Essenciais](#componentes-essenciais)
   - [Classes Utilitárias](#classes-utilitárias)
   - [Customização com Variáveis SCSS](#customização-com-variáveis-scss)
   - [Bootstrap + React](#bootstrap--react)
   - [Bootstrap + Vue](#bootstrap--vue)
   - [Bootstrap + Angular](#bootstrap--angular)
   - [Tailwind vs Bootstrap — Comparativo Detalhado](#tailwind-vs-bootstrap--comparativo-detalhado)
4. [Material UI (MUI) e Bibliotecas de Componentes](#4-material-ui-mui-e-bibliotecas-de-componentes)
   - [Visão Geral e Material Design 3](#visão-geral-e-material-design-3)
   - [Instalação e Configuração do MUI](#instalação-e-configuração-do-mui)
   - [Sistema de Theming](#sistema-de-theming)
   - [Design Tokens no MUI](#design-tokens-no-mui)
   - [Componentes Essenciais do MUI](#componentes-essenciais-do-mui)
   - [Layout e Grid (MUI System)](#layout-e-grid-mui-system)
   - [Customização Avançada](#customização-avançada)
   - [MUI + React Hook Form](#mui--react-hook-form)
   - [Angular Material](#angular-material)
   - [shadcn/ui — Alternativa com Radix + Tailwind](#shadcnui--alternativa-com-radix--tailwind)
   - [Outras Bibliotecas de Componentes](#outras-bibliotecas-de-componentes)
   - [Comparativo: MUI vs shadcn/ui vs Radix vs Headless UI](#comparativo-mui-vs-shadcnui-vs-radix-vs-headless-ui)
   - [Theming Dinâmico por Tenant (White-Label)](#theming-dinâmico-por-tenant-white-label)
   - [Tabelas Avançadas e Virtualização](#tabelas-avançadas-e-virtualização)
   - [Dashboards e Visualização de Dados](#dashboards-e-visualização-de-dados)
   - [Formulários Complexos — Padrões de UX](#formulários-complexos--padrões-de-ux)
5. [Diretrizes de Design Mobile](#5-diretrizes-de-design-mobile)
   - [Material Design 3 (Android)](#material-design-3-android)
   - [Human Interface Guidelines (iOS)](#human-interface-guidelines-ios)
   - [Comparativo Material Design vs HIG](#comparativo-material-design-vs-hig)
   - [Convenções de Plataforma](#convenções-de-plataforma)
   - [Navegação Nativa](#navegação-nativa)
   - [Aplicando Material Design 3 no Flutter](#aplicando-material-design-3-no-flutter)
   - [Aplicando HIG no Flutter (Cupertino)](#aplicando-hig-no-flutter-cupertino)
   - [Design Adaptativo no React Native](#design-adaptativo-no-react-native)
   - [Design Responsivo para Tablets e Foldables](#design-responsivo-para-tablets-e-foldables)
   - [Acessibilidade em Apps Nativos](#acessibilidade-em-apps-nativos)
   - [Micro-interações e Animações](#micro-interações-e-animações)
   - [RTL e Internacionalização Visual](#rtl-e-internacionalização-visual)
6. [Progressive Web Apps (PWA)](#6-progressive-web-apps-pwa)
   - [PWA com Workbox](#pwa-com-workbox)
   - [Push Notifications](#push-notifications)
   - [Background Sync](#background-sync)
   - [APIs Web Modernas para PWA](#apis-web-modernas-para-pwa)
   - [Install Prompt Customizado](#install-prompt-customizado)
   - [PWA com React (Vite PWA)](#pwa-com-react-vite-pwa)
   - [PWA com Angular](#pwa-com-angular)
   - [PWA com Vue (Vite PWA)](#pwa-com-vue-vite-pwa)
   - [Estratégias de Atualização do Service Worker](#estratégias-de-atualização-do-service-worker)
   - [Core Web Vitals](#core-web-vitals)
   - [Performance — Lazy Loading e Otimização de Imagens](#performance--lazy-loading-e-otimização-de-imagens)
   - [Lighthouse e Auditoria de PWA](#lighthouse-e-auditoria-de-pwa)
   - [PWA vs App Nativo — Quando Usar Cada](#pwa-vs-app-nativo--quando-usar-cada)
7. [Acessibilidade Avançada (WCAG 2.2)](#7-acessibilidade-avançada-wcag-22)
   - [WCAG 2.2 — Novos Critérios de Sucesso](#wcag-22--novos-critérios-de-sucesso)
   - [Níveis de Conformidade na Prática](#níveis-de-conformidade-na-prática)
   - [Contraste de Cores — Ferramentas e APCA](#contraste-de-cores--ferramentas-e-apca)
   - [Padrões ARIA Avançados](#padrões-aria-avançados)
   - [Popover API, dialog Nativo e CSS Anchor Positioning](#popover-api-dialog-nativo-e-css-anchor-positioning)
   - [Scroll-driven Animations e View Transitions API](#scroll-driven-animations-e-view-transitions-api)
   - [Componentes Acessíveis — Modal/Dialog](#componentes-acessíveis--modaldialog)
   - [Componentes Acessíveis — Tabs](#componentes-acessíveis--tabs)
   - [Componentes Acessíveis — Accordion](#componentes-acessíveis--accordion)
   - [Componentes Acessíveis — Carrossel](#componentes-acessíveis--carrossel)
   - [Componentes Acessíveis — Combobox/Autocomplete](#componentes-acessíveis--comboboxautocomplete)
   - [Componentes Acessíveis — Data Table com Ordenação](#componentes-acessíveis--data-table-com-ordenação)
   - [Formulários Acessíveis Avançados](#formulários-acessíveis-avançados)
   - [Navegação por Teclado](#navegação-por-teclado)
   - [Teste com Leitores de Tela (NVDA/VoiceOver)](#teste-com-leitores-de-tela-nvdavoiceover)
   - [Teste Automatizado de Acessibilidade em CI/CD](#teste-automatizado-de-acessibilidade-em-cicd)
   - [Acessibilidade em SPAs](#acessibilidade-em-spas)
   - [Acessibilidade em E-mails HTML](#acessibilidade-em-e-mails-html)
   - [Checklist de Acessibilidade por Nível](#checklist-de-acessibilidade-por-nível)
8. [Referências](#8-referências)

---

## 1. Design Systems e Design Tokens

Um Design System é um conjunto unificado de padrões, componentes e diretrizes que garante consistência visual e funcional em todos os produtos de uma organização — web, mobile e desktop.

> Referência: [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/) · [Material Design](https://m3.material.io/) · [Apple HIG](https://developer.apple.com/design/human-interface-guidelines/)

---

### O que é um Design System

Um Design System vai além de um guia de estilo ou biblioteca de componentes. Ele é a **fonte única de verdade** que conecta design e desenvolvimento.

| Conceito | Descrição |
|---|---|
| **Guia de estilo** | Documento estático com cores, fontes e regras visuais |
| **Biblioteca de componentes** | Coleção de componentes reutilizáveis em código |
| **Design System** | Guia de estilo + biblioteca de componentes + tokens + documentação + governança |

**Exemplos de Design Systems conhecidos:**

| Design System | Empresa | Tecnologia |
|---|---|---|
| Material Design 3 | Componentes para Android, Flutter, Web | |
| Human Interface Guidelines | Apple | SwiftUI, UIKit |
| Carbon | IBM | React, Angular, Vue, Svelte |
| Ant Design | Alibaba | React |
| Fluent UI | Microsoft | React |
| Chakra UI | Comunidade | React |
| Polaris | Shopify | React |

**Por que adotar um Design System:**

- **Consistência** — mesma aparência e comportamento em todos os produtos
- **Velocidade** — desenvolvedores reutilizam componentes em vez de recriá-los
- **Qualidade** — componentes testados e acessíveis por padrão
- **Escalabilidade** — múltiplas equipes trabalham com os mesmos padrões
- **Comunicação** — vocabulário comum entre designers e desenvolvedores

---

### Anatomia de um Design System

Um Design System é organizado em camadas progressivas, da mais abstrata à mais concreta:

```
┌─────────────────────────────────────────────┐
│              Documentação                    │
│  (princípios, guidelines, governança)       │
├─────────────────────────────────────────────┤
│              Templates / Páginas            │
│  (layouts de página completos)              │
├─────────────────────────────────────────────┤
│          Componentes Compostos              │
│  (Header, Sidebar, Form, Card complexo)     │
├─────────────────────────────────────────────┤
│          Componentes Core                   │
│  (Button, Input, Select, Modal, Badge)      │
├─────────────────────────────────────────────┤
│            Design Tokens                    │
│  (cor, tipografia, espaçamento, elevação)   │
└─────────────────────────────────────────────┘
```

| Camada | Exemplo | Responsabilidade |
|---|---|---|
| **Tokens** | `--color-primary: #2563eb` | Valores primitivos de design |
| **Componentes Core** | `<Button variant="primary">` | Elementos atômicos reutilizáveis |
| **Componentes Compostos** | `<SearchBar>` (Input + Button + Dropdown) | Combinação de componentes core |
| **Templates** | Layout de Dashboard | Estruturas de página |
| **Documentação** | Storybook, site de docs | Como e quando usar cada peça |

---

### Design Tokens — Conceito e Taxonomia

Design Tokens são os **valores atômicos de design** — cores, tamanhos, espaçamentos, fontes — representados de forma agnóstica a plataforma. São o contrato entre design e código.

> Referência: [W3C Design Tokens Format](https://tr.designtokens.org/format/)

**Taxonomia em três níveis:**

```
┌──────────────────────────────────────────────┐
│        Component Tokens (específicos)        │
│   --button-bg, --card-radius, --input-border │
├──────────────────────────────────────────────┤
│        Semantic Tokens (aliases)             │
│   --color-primary, --color-surface,          │
│   --spacing-md, --font-body                  │
├──────────────────────────────────────────────┤
│        Primitive Tokens (globais)            │
│   --blue-500, --gray-100, --space-4,         │
│   --font-size-16                             │
└──────────────────────────────────────────────┘
```

**Formato W3C Design Tokens (JSON):**

```json
{
  "$name": "my-design-system",
  "color": {
    "blue": {
      "500": {
        "$value": "#2563eb",
        "$type": "color",
        "$description": "Azul primário"
      }
    },
    "primary": {
      "$value": "{color.blue.500}",
      "$type": "color",
      "$description": "Cor primária — alias para blue-500"
    }
  },
  "spacing": {
    "md": {
      "$value": "16px",
      "$type": "dimension"
    }
  }
}
```

---

### Tokens de Cor

O sistema de cores é organizado em **primitivos** (paleta completa) e **semânticos** (significado contextual).

**Tokens primitivos — paleta de cores:**

```css
:root {
  /* Azul */
  --blue-50: #eff6ff;
  --blue-100: #dbeafe;
  --blue-200: #bfdbfe;
  --blue-300: #93c5fd;
  --blue-400: #60a5fa;
  --blue-500: #3b82f6;
  --blue-600: #2563eb;
  --blue-700: #1d4ed8;
  --blue-800: #1e40af;
  --blue-900: #1e3a8a;
  --blue-950: #172554;

  /* Cinza (Neutral) */
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-400: #9ca3af;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;
  --gray-950: #030712;

  /* Verde (Sucesso) */
  --green-500: #22c55e;
  --green-600: #16a34a;
  --green-700: #15803d;

  /* Vermelho (Erro) */
  --red-500: #ef4444;
  --red-600: #dc2626;
  --red-700: #b91c1c;

  /* Amarelo (Aviso) */
  --yellow-500: #eab308;
  --yellow-600: #ca8a04;
}
```

**Tokens semânticos — light mode:**

```css
:root {
  /* Superfícies */
  --color-background: var(--gray-50);
  --color-surface: #ffffff;
  --color-surface-elevated: #ffffff;

  /* Texto */
  --color-on-background: var(--gray-900);
  --color-on-surface: var(--gray-800);
  --color-on-surface-muted: var(--gray-500);

  /* Interação */
  --color-primary: var(--blue-600);
  --color-primary-hover: var(--blue-700);
  --color-on-primary: #ffffff;

  /* Feedback */
  --color-success: var(--green-600);
  --color-error: var(--red-600);
  --color-warning: var(--yellow-600);

  /* Bordas */
  --color-border: var(--gray-200);
  --color-border-strong: var(--gray-300);

  /* Foco */
  --color-focus-ring: var(--blue-500);
}
```

---

### Tokens de Tipografia

Escala tipográfica consistente usando `rem` para respeitar as preferências do usuário.

```css
:root {
  /* Família */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;

  /* Tamanho — escala modular (ratio ~1.25) */
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 1.875rem;  /* 30px */
  --font-size-4xl: 2.25rem;   /* 36px */
  --font-size-5xl: 3rem;      /* 48px */

  /* Peso */
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  /* Altura de linha */
  --line-height-tight: 1.25;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.75;

  /* Espaçamento entre letras */
  --letter-spacing-tight: -0.025em;
  --letter-spacing-normal: 0;
  --letter-spacing-wide: 0.025em;
}
```

**Tokens compostos de tipografia (classes utilitárias):**

```css
.text-heading-1 {
  font-family: var(--font-sans);
  font-size: var(--font-size-4xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-tight);
  letter-spacing: var(--letter-spacing-tight);
}

.text-heading-2 {
  font-family: var(--font-sans);
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-tight);
}

.text-body {
  font-family: var(--font-sans);
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-regular);
  line-height: var(--line-height-normal);
}

.text-caption {
  font-family: var(--font-sans);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-regular);
  line-height: var(--line-height-normal);
  color: var(--color-on-surface-muted);
}

.text-code {
  font-family: var(--font-mono);
  font-size: var(--font-size-sm);
  line-height: var(--line-height-relaxed);
}
```

---

### Tokens de Espaçamento

Sistema baseado no **grid de 4px** (base unit = 4px). Cada nível é múltiplo de 4.

```css
:root {
  --space-0: 0;
  --space-0-5: 0.125rem;  /* 2px */
  --space-1: 0.25rem;     /* 4px */
  --space-1-5: 0.375rem;  /* 6px */
  --space-2: 0.5rem;      /* 8px */
  --space-3: 0.75rem;     /* 12px */
  --space-4: 1rem;        /* 16px */
  --space-5: 1.25rem;     /* 20px */
  --space-6: 1.5rem;      /* 24px */
  --space-8: 2rem;        /* 32px */
  --space-10: 2.5rem;     /* 40px */
  --space-12: 3rem;       /* 48px */
  --space-16: 4rem;       /* 64px */
  --space-20: 5rem;       /* 80px */
  --space-24: 6rem;       /* 96px */
}
```

> Para a implementação equivalente em Flutter (`AppSpacing`), consulte [Frontend-Flutter.md](Frontend-Flutter.md) — seção Design Token System.

**Uso semântico do espaçamento:**

| Token Semântico | Valor | Uso |
|---|---|---|
| `--spacing-xs` | `var(--space-1)` (4px) | Espaço entre ícone e texto |
| `--spacing-sm` | `var(--space-2)` (8px) | Padding interno de badges, chips |
| `--spacing-md` | `var(--space-4)` (16px) | Padding de botões, gap de formulários |
| `--spacing-lg` | `var(--space-6)` (24px) | Padding de cards, seções internas |
| `--spacing-xl` | `var(--space-8)` (32px) | Gap entre seções, margens de containers |
| `--spacing-2xl` | `var(--space-12)` (48px) | Separação de blocos de conteúdo |
| `--spacing-3xl` | `var(--space-16)` (64px) | Margens de página, hero sections |

---

### Tokens de Elevação (Sombras)

Escala de elevação para indicar hierarquia visual e interatividade.

```css
:root {
  --shadow-xs: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1),
               0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1),
               0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1),
               0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1),
               0 8px 10px -6px rgb(0 0 0 / 0.1);
  --shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
  --shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);
  --shadow-none: 0 0 #0000;
}
```

| Nível | Token | Uso típico |
|---|---|---|
| Repouso | `--shadow-xs` | Inputs, bordas sutis |
| Leve | `--shadow-sm` | Cards, botões em repouso |
| Médio | `--shadow-md` | Cards hover, dropdowns |
| Alto | `--shadow-lg` | Modais, popovers |
| Muito alto | `--shadow-xl` | Diálogos flutuantes |
| Máximo | `--shadow-2xl` | Elementos de destaque máximo |

---

### Tokens de Borda e Raio

```css
:root {
  /* Largura de borda */
  --border-width-thin: 1px;
  --border-width-medium: 2px;
  --border-width-thick: 4px;

  /* Raio de borda */
  --radius-none: 0;
  --radius-sm: 0.25rem;   /* 4px */
  --radius-md: 0.375rem;  /* 6px */
  --radius-lg: 0.5rem;    /* 8px */
  --radius-xl: 0.75rem;   /* 12px */
  --radius-2xl: 1rem;     /* 16px */
  --radius-3xl: 1.5rem;   /* 24px */
  --radius-full: 9999px;  /* Circular */
}
```

| Componente | Raio recomendado |
|---|---|
| Botão pequeno | `--radius-md` |
| Botão grande | `--radius-lg` |
| Card | `--radius-xl` |
| Input | `--radius-md` |
| Avatar | `--radius-full` |
| Modal | `--radius-2xl` |
| Badge/Chip | `--radius-full` |

---

### Tokens de Animação e Transição

```css
:root {
  /* Duração */
  --duration-fast: 100ms;
  --duration-normal: 200ms;
  --duration-slow: 300ms;
  --duration-slower: 500ms;

  /* Easing */
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-spring: cubic-bezier(0.22, 1, 0.36, 1);

  /* Transições compostas */
  --transition-colors: color, background-color, border-color, text-decoration-color, fill, stroke;
  --transition-transform: transform;
  --transition-all: all;
}
```

**Uso em componentes:**

```css
.button {
  transition-property: var(--transition-colors), var(--transition-transform);
  transition-duration: var(--duration-normal);
  transition-timing-function: var(--ease-default);
}

.button:hover {
  transform: translateY(-1px);
}

.button:active {
  transform: translateY(0);
  transition-duration: var(--duration-fast);
}

.modal-overlay {
  transition: opacity var(--duration-slow) var(--ease-out);
}

.modal-content {
  transition: transform var(--duration-slow) var(--ease-spring);
}
```

---

### Implementação com CSS Custom Properties

> Para introdução a CSS Custom Properties, consulte [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md) — seção CSS Moderno.

**Organização de arquivos de tokens:**

```
styles/
├── tokens/
│   ├── _primitives.css      /* Cores brutas, escalas numéricas */
│   ├── _semantic-light.css  /* Tokens semânticos - tema claro */
│   ├── _semantic-dark.css   /* Tokens semânticos - tema escuro */
│   ├── _typography.css      /* Fontes, tamanhos, pesos */
│   ├── _spacing.css         /* Espaçamento */
│   ├── _elevation.css       /* Sombras */
│   ├── _borders.css         /* Bordas e raios */
│   └── _animation.css       /* Durações e easings */
├── components/
│   ├── _button.css
│   ├── _card.css
│   ├── _input.css
│   └── _modal.css
├── layouts/
│   ├── _header.css
│   ├── _sidebar.css
│   └── _grid.css
└── main.css                 /* Importa tudo */
```

**Arquivo principal (`main.css`):**

```css
/* Tokens */
@import './tokens/_primitives.css';
@import './tokens/_semantic-light.css';
@import './tokens/_typography.css';
@import './tokens/_spacing.css';
@import './tokens/_elevation.css';
@import './tokens/_borders.css';
@import './tokens/_animation.css';

/* Componentes */
@import './components/_button.css';
@import './components/_card.css';
@import './components/_input.css';
@import './components/_modal.css';

/* Layouts */
@import './layouts/_header.css';
@import './layouts/_sidebar.css';
@import './layouts/_grid.css';
```

**Exemplo completo de componente usando tokens:**

```css
/* components/_card.css */
.card {
  background-color: var(--color-surface);
  border: var(--border-width-thin) solid var(--color-border);
  border-radius: var(--radius-xl);
  padding: var(--spacing-lg);
  box-shadow: var(--shadow-sm);
  transition: box-shadow var(--duration-normal) var(--ease-default),
              transform var(--duration-normal) var(--ease-default);
}

.card:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}

.card__title {
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-semibold);
  color: var(--color-on-surface);
  margin-bottom: var(--spacing-sm);
}

.card__description {
  font-size: var(--font-size-base);
  color: var(--color-on-surface-muted);
  line-height: var(--line-height-normal);
}

.card__actions {
  display: flex;
  gap: var(--spacing-sm);
  margin-top: var(--spacing-md);
}
```

---

### Tokens Semânticos vs Primitivos

| Aspecto | Primitivo | Semântico | Componente |
|---|---|---|---|
| **Exemplo** | `--blue-600` | `--color-primary` | `--button-bg` |
| **Propósito** | Valor bruto da paleta | Significado no design | Uso específico |
| **Muda com tema?** | Não | Sim | Sim (herda do semântico) |
| **Quem define?** | Designer de marca | Designer de sistema | Desenvolvedor |
| **Quantidade** | Muitos (~100+) | Moderada (~30-50) | Muitos (~100+) |

**Fluxo de resolução:**

```
--button-bg: var(--color-primary)
     ↓
--color-primary: var(--blue-600)    [light mode]
--color-primary: var(--blue-400)    [dark mode]
     ↓
--blue-600: #2563eb
--blue-400: #60a5fa
```

**Por que não usar primitivos diretamente nos componentes:**

```css
/* ❌ Ruim — acoplado à cor específica */
.button { background-color: var(--blue-600); }

/* ✅ Bom — acoplado ao significado */
.button { background-color: var(--color-primary); }
```

Quando o tema muda (dark mode, marca diferente), basta redefinir os tokens semânticos. Os componentes não precisam mudar.

---

### Multi-Tema com Design Tokens

**Abordagem com `data-theme` (recomendada para controle programático):**

```css
/* _semantic-light.css */
:root,
[data-theme="light"] {
  --color-background: var(--gray-50);
  --color-surface: #ffffff;
  --color-on-background: var(--gray-900);
  --color-on-surface: var(--gray-800);
  --color-on-surface-muted: var(--gray-500);
  --color-primary: var(--blue-600);
  --color-primary-hover: var(--blue-700);
  --color-on-primary: #ffffff;
  --color-border: var(--gray-200);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1);
}

/* _semantic-dark.css */
[data-theme="dark"] {
  --color-background: var(--gray-950);
  --color-surface: var(--gray-900);
  --color-on-background: var(--gray-50);
  --color-on-surface: var(--gray-200);
  --color-on-surface-muted: var(--gray-400);
  --color-primary: var(--blue-400);
  --color-primary-hover: var(--blue-300);
  --color-on-primary: var(--gray-900);
  --color-border: var(--gray-700);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.4);
}

/* Tema de alto contraste */
[data-theme="high-contrast"] {
  --color-background: #000000;
  --color-surface: #000000;
  --color-on-background: #ffffff;
  --color-on-surface: #ffffff;
  --color-on-surface-muted: #e0e0e0;
  --color-primary: #ffff00;
  --color-primary-hover: #ffff66;
  --color-on-primary: #000000;
  --color-border: #ffffff;
}
```

**Respeitar preferência do sistema com fallback:**

```css
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]):not([data-theme="high-contrast"]) {
    --color-background: var(--gray-950);
    --color-surface: var(--gray-900);
    /* ... demais tokens dark */
  }
}
```

**Alternância de tema com JavaScript:**

```typescript
type Theme = 'light' | 'dark' | 'high-contrast' | 'system';

function setTheme(theme: Theme): void {
  if (theme === 'system') {
    document.documentElement.removeAttribute('data-theme');
  } else {
    document.documentElement.setAttribute('data-theme', theme);
  }
  localStorage.setItem('theme', theme);
}

function getInitialTheme(): Theme {
  const stored = localStorage.getItem('theme') as Theme | null;
  return stored ?? 'system';
}

// Aplicar ao carregar a página
setTheme(getInitialTheme());
```

**Componente React de seletor de tema:**

```tsx
import { useState, useEffect } from 'react';

type Theme = 'light' | 'dark' | 'high-contrast' | 'system';

export function ThemeSelector() {
  const [theme, setThemeState] = useState<Theme>('system');

  useEffect(() => {
    const stored = localStorage.getItem('theme') as Theme | null;
    if (stored) setThemeState(stored);
  }, []);

  function handleChange(newTheme: Theme) {
    setThemeState(newTheme);
    if (newTheme === 'system') {
      document.documentElement.removeAttribute('data-theme');
    } else {
      document.documentElement.setAttribute('data-theme', newTheme);
    }
    localStorage.setItem('theme', newTheme);
  }

  return (
    <select
      value={theme}
      onChange={e => handleChange(e.target.value as Theme)}
      aria-label="Selecionar tema"
    >
      <option value="system">Sistema</option>
      <option value="light">Claro</option>
      <option value="dark">Escuro</option>
      <option value="high-contrast">Alto Contraste</option>
    </select>
  );
}
```

---

### Arquitetura de Componentes em um Design System

A arquitetura segue o conceito de **Atomic Design** (Brad Frost): átomos → moléculas → organismos → templates → páginas.

**Exemplo completo — Button com variantes (React + TypeScript):**

```tsx
import { type ButtonHTMLAttributes, type ReactNode } from 'react';

type ButtonVariant = 'primary' | 'secondary' | 'danger' | 'ghost';
type ButtonSize = 'sm' | 'md' | 'lg';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
  loading?: boolean;
  leftIcon?: ReactNode;
  children: ReactNode;
}

export function Button({
  variant = 'primary',
  size = 'md',
  loading = false,
  leftIcon,
  disabled,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant} btn--${size}`}
      disabled={disabled || loading}
      aria-busy={loading}
      {...props}
    >
      {loading ? (
        <span className="btn__spinner" aria-hidden="true" />
      ) : leftIcon ? (
        <span className="btn__icon" aria-hidden="true">{leftIcon}</span>
      ) : null}
      <span className="btn__label">{children}</span>
    </button>
  );
}
```

**CSS do Button usando tokens:**

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-xs);
  font-family: var(--font-sans);
  font-weight: var(--font-weight-medium);
  border: var(--border-width-medium) solid transparent;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition-property: background-color, color, border-color, box-shadow, transform;
  transition-duration: var(--duration-normal);
  transition-timing-function: var(--ease-default);
}

.btn:focus-visible {
  outline: var(--border-width-medium) solid var(--color-focus-ring);
  outline-offset: 2px;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variantes */
.btn--primary {
  background-color: var(--color-primary);
  color: var(--color-on-primary);
}
.btn--primary:hover:not(:disabled) {
  background-color: var(--color-primary-hover);
}

.btn--secondary {
  background-color: transparent;
  color: var(--color-primary);
  border-color: var(--color-primary);
}
.btn--secondary:hover:not(:disabled) {
  background-color: var(--color-primary);
  color: var(--color-on-primary);
}

.btn--danger {
  background-color: var(--color-error);
  color: #ffffff;
}
.btn--danger:hover:not(:disabled) {
  background-color: var(--red-700);
}

.btn--ghost {
  background-color: transparent;
  color: var(--color-on-surface);
}
.btn--ghost:hover:not(:disabled) {
  background-color: var(--gray-100);
}

/* Tamanhos */
.btn--sm {
  font-size: var(--font-size-sm);
  padding: var(--space-1-5) var(--space-3);
  border-radius: var(--radius-md);
}
.btn--md {
  font-size: var(--font-size-base);
  padding: var(--space-2) var(--space-4);
}
.btn--lg {
  font-size: var(--font-size-lg);
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-lg);
}

/* Spinner */
.btn__spinner {
  width: 1em;
  height: 1em;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: var(--radius-full);
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

### Consistência Web + Mobile — Tokens Multiplataforma

Para manter consistência entre web, Flutter e React Native, os tokens devem ter uma **fonte única de verdade** — tipicamente um arquivo JSON — que gera artefatos para cada plataforma.

**Fluxo de geração:**

```
tokens.json (fonte única)
     │
     ├──→ CSS Custom Properties  (Web)
     ├──→ Dart constants         (Flutter)
     ├──→ JS/TS constants        (React Native)
     └──→ SCSS variables         (Web legado)
```

**Style Dictionary (Amazon) — configuração:**

```json
{
  "source": ["tokens/**/*.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "build/css/",
      "files": [{
        "destination": "tokens.css",
        "format": "css/variables"
      }]
    },
    "flutter": {
      "transformGroup": "flutter",
      "buildPath": "build/flutter/",
      "files": [{
        "destination": "tokens.dart",
        "format": "flutter/class.dart"
      }]
    },
    "rn": {
      "transformGroup": "react-native",
      "buildPath": "build/rn/",
      "files": [{
        "destination": "tokens.ts",
        "format": "javascript/es6"
      }]
    }
  }
}
```

> Para a implementação de tokens em Flutter (`AppSpacing`, `AppSizes`, `ThemeExtension`), consulte [Frontend-Flutter.md](Frontend-Flutter.md).
> Para NativeWind e temas em React Native, consulte [Frontend-React-Native.md](Frontend-React-Native.md).

**Exemplo — mesmos tokens em três plataformas:**

```css
/* Web — CSS Custom Properties */
:root {
  --space-4: 1rem;
  --color-primary: #2563eb;
  --radius-lg: 0.5rem;
}
```

```dart
// Flutter — Dart constants
abstract class AppTokens {
  static const double space4 = 16.0;
  static const Color colorPrimary = Color(0xFF2563EB);
  static const double radiusLg = 8.0;
}
```

```typescript
// React Native — TypeScript constants
export const tokens = {
  space4: 16,
  colorPrimary: '#2563eb',
  radiusLg: 8,
} as const;
```

---

### Ferramentas para Gerenciamento de Tokens

| Ferramenta | Tipo | Descrição |
|---|---|---|
| **Style Dictionary** | Build tool | Transforma tokens JSON em CSS, SCSS, Dart, JS, iOS, Android |
| **Tokens Studio for Figma** | Plugin Figma | Gerencia tokens no Figma e sincroniza com repositório Git |
| **Specify** | Plataforma | Coleta tokens do Figma e distribui para código automaticamente |
| **Design Tokens W3C** | Especificação | Formato padrão para intercâmbio de tokens entre ferramentas |
| **Cobalt UI** | Build tool | Alternativa ao Style Dictionary com suporte a W3C DTCG format |
| **Figma Variables** | Nativo Figma | Sistema nativo para gerenciar variáveis de design no Figma |

> Para detalhes sobre Figma Variables e componentes, consulte [Figma.md](outros/Figma.md).

---

### Storybook — Documentação Interativa de Componentes

Storybook é a ferramenta padrão de mercado para desenvolver, documentar e testar componentes de UI de forma isolada. Cada componente recebe "stories" que exibem seus estados, variantes e interações.

> Referência: [Storybook Docs](https://storybook.js.org/) · [Chromatic](https://www.chromatic.com/)

**Instalação:**

```bash
# Em projeto React/Vue/Angular existente
npx storybook@latest init
```

**Estrutura de uma story (CSF 3 — Component Story Format):**

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger', 'ghost'],
      description: 'Estilo visual do botão',
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
    },
    disabled: { control: 'boolean' },
    loading: { control: 'boolean' },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Salvar',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Cancelar',
  },
};

export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="danger">Danger</Button>
      <Button variant="ghost">Ghost</Button>
    </div>
  ),
};

export const Loading: Story = {
  args: {
    variant: 'primary',
    loading: true,
    children: 'Salvando...',
  },
};
```

**Documentação com MDX (API docs do componente):**

```mdx
{/* Button.mdx */}
import { Meta, Canvas, Controls, Story } from '@storybook/blocks';
import * as ButtonStories from './Button.stories';

<Meta of={ButtonStories} />

# Button

Botão reutilizável com suporte a variantes, tamanhos e estado de loading.

## Quando usar
- **Primary:** Ação principal da tela (1 por seção)
- **Secondary:** Ações alternativas
- **Danger:** Ações destrutivas (excluir, cancelar)
- **Ghost:** Ações terciárias ou navegação

## Playground

<Canvas of={ButtonStories.Primary} />
<Controls />

## Todas as variantes

<Canvas of={ButtonStories.AllVariants} />
```

**Visual Regression Testing com Chromatic:**

```bash
npm install chromatic --save-dev
npx chromatic --project-token=<TOKEN>
```

Chromatic captura screenshots de cada story e compara com a versão anterior, detectando mudanças visuais indesejadas automaticamente no CI.

**Storybook com Tailwind CSS:**

```typescript
// .storybook/preview.ts
import '../src/index.css'; // Importa o CSS com @import "tailwindcss"
import type { Preview } from '@storybook/react';

const preview: Preview = {
  parameters: {
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#f8fafc' },
        { name: 'dark', value: '#0f172a' },
      ],
    },
  },
  decorators: [
    (Story, context) => (
      <div data-theme={context.globals.theme || 'light'}>
        <Story />
      </div>
    ),
  ],
};

export default preview;
```

**Addons recomendados:**

| Addon | Função |
|---|---|
| `@storybook/addon-essentials` | Controls, actions, viewport, backgrounds, docs |
| `@storybook/addon-a11y` | Auditoria de acessibilidade (axe-core) em cada story |
| `@storybook/addon-interactions` | Testes de interação (play functions) |
| `storybook-dark-mode` | Toggle dark mode no Storybook |

---

### Padrões de UI — Estados Vazios, Erro e Loading

Todo componente e toda tela tem estados além do "caminho feliz". Um Design System maduro define padrões visuais para cada estado.

**Os 5 estados de uma tela:**

| Estado | Descrição | Exemplo |
|---|---|---|
| **Loading** | Dados estão sendo carregados | Skeleton, spinner |
| **Empty** | Nenhum dado para exibir | Ilustração + mensagem + CTA |
| **Partial** | Dados parciais ou incompletos | Placeholders, valores padrão |
| **Error** | Falha ao carregar ou processar | Mensagem + botão "Tentar novamente" |
| **Ideal** | Dados completos | O estado "normal" do design |

**Skeleton Screens (shimmer/placeholder):**

```css
.skeleton {
  background: linear-gradient(
    90deg,
    var(--gray-200) 25%,
    var(--gray-100) 50%,
    var(--gray-200) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: var(--radius-md);
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton-text {
  height: 1em;
  margin-bottom: 0.5em;
}

.skeleton-text--short { width: 60%; }
.skeleton-text--full { width: 100%; }
.skeleton-avatar { width: 48px; height: 48px; border-radius: var(--radius-full); }
.skeleton-image { width: 100%; aspect-ratio: 16/9; }
```

```tsx
function ProductCardSkeleton() {
  return (
    <div className="card" aria-busy="true" aria-label="Carregando produto">
      <div className="skeleton skeleton-image" />
      <div style={{ padding: 'var(--spacing-md)' }}>
        <div className="skeleton skeleton-text skeleton-text--full" />
        <div className="skeleton skeleton-text skeleton-text--short" />
      </div>
    </div>
  );
}

function ProductList() {
  const { data, isLoading, error } = useProducts();

  if (isLoading) {
    return (
      <div className="grid grid-cols-3 gap-4">
        {Array.from({ length: 6 }, (_, i) => (
          <ProductCardSkeleton key={i} />
        ))}
      </div>
    );
  }

  if (error) return <ErrorState onRetry={() => refetch()} />;
  if (!data?.length) return <EmptyState />;

  return (
    <div className="grid grid-cols-3 gap-4">
      {data.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

**Empty State:**

```tsx
interface EmptyStateProps {
  icon?: React.ReactNode;
  title: string;
  description: string;
  action?: { label: string; onClick: () => void };
}

function EmptyState({ icon, title, description, action }: EmptyStateProps) {
  return (
    <div className="empty-state" role="status">
      {icon && <div className="empty-state__icon" aria-hidden="true">{icon}</div>}
      <h3 className="empty-state__title">{title}</h3>
      <p className="empty-state__description">{description}</p>
      {action && (
        <button className="btn btn--primary" onClick={action.onClick}>
          {action.label}
        </button>
      )}
    </div>
  );
}

// Uso
<EmptyState
  title="Nenhum produto encontrado"
  description="Tente ajustar os filtros ou adicione um novo produto."
  action={{ label: 'Adicionar produto', onClick: () => navigate('/novo') }}
/>
```

**Error State e Error Boundary (React):**

```tsx
import { Component, type ReactNode } from 'react';

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; }

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="error-state" role="alert">
          <h3>Algo deu errado</h3>
          <p>Ocorreu um erro inesperado. Tente recarregar a página.</p>
          <button onClick={() => window.location.reload()}>
            Recarregar página
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Error state para falhas de API
function ErrorState({ message, onRetry }: { message?: string; onRetry?: () => void }) {
  return (
    <div className="error-state" role="alert">
      <h3>Falha ao carregar</h3>
      <p>{message ?? 'Verifique sua conexão e tente novamente.'}</p>
      {onRetry && (
        <button className="btn btn--primary" onClick={onRetry}>
          Tentar novamente
        </button>
      )}
    </div>
  );
}
```

**Página 404 / 500:**

```tsx
function NotFoundPage() {
  return (
    <main className="error-page">
      <h1>404</h1>
      <p>A página que você procura não existe ou foi movida.</p>
      <a href="/" className="btn btn--primary">Voltar ao início</a>
    </main>
  );
}

function ServerErrorPage() {
  return (
    <main className="error-page" role="alert">
      <h1>500</h1>
      <p>Erro interno no servidor. Nossa equipe já foi notificada.</p>
      <button className="btn btn--primary" onClick={() => window.location.reload()}>
        Tentar novamente
      </button>
    </main>
  );
}
```

---

## 2. Tailwind CSS v4

Tailwind CSS é um framework CSS utility-first que fornece classes utilitárias de baixo nível para construir interfaces diretamente no HTML, sem escrever CSS customizado.

> Referência: [Tailwind CSS Docs](https://tailwindcss.com/docs) · [Tailwind UI](https://tailwindui.com/) · [Headless UI](https://headlessui.com/)

> Para uma visão geral introdutória do Tailwind, consulte [Topicos-Complementares-Desenvolvimento.md](outros/Topicos-Complementares-Desenvolvimento.md).

---

### Filosofia Utility-First

No modelo utility-first, em vez de escrever classes CSS semânticas (`.card`, `.navbar`), aplica-se classes utilitárias diretamente nos elementos HTML.

**Comparação de abordagens:**

```html
<!-- ❌ CSS tradicional (BEM) -->
<div class="card card--featured">
  <h2 class="card__title">Título</h2>
  <p class="card__description">Descrição do item</p>
</div>

<!-- ✅ Tailwind utility-first -->
<div class="rounded-xl bg-white p-6 shadow-md hover:shadow-lg transition-shadow">
  <h2 class="text-xl font-semibold text-gray-900">Título</h2>
  <p class="text-base text-gray-500 mt-2">Descrição do item</p>
</div>
```

| Aspecto | Utility-First | CSS Tradicional (BEM/SMACSS) |
|---|---|---|
| **HTML** | Mais verboso | Mais limpo |
| **CSS** | Mínimo ou zero customizado | Muitos arquivos CSS |
| **CSS morto** | Eliminado automaticamente (PurgeCSS) | Acumula ao longo do projeto |
| **Consistência** | Forçada pelas classes do tema | Depende de disciplina |
| **Curva de aprendizado** | Decorar classes | Aprender convenções de nomes |
| **Refatoração** | Mover HTML move o estilo junto | Trocar classe pode quebrar layout |
| **Prototipagem** | Muito rápida | Mais lenta (criar CSS antes) |

---

### Instalação e Configuração v4

O Tailwind v4 introduziu configuração **CSS-first** — o tema é definido diretamente no CSS via `@theme`, eliminando o `tailwind.config.js` na maioria dos casos.

**Instalação com Vite:**

```bash
npm install tailwindcss @tailwindcss/vite
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
});
```

**Arquivo CSS principal:**

```css
/* src/styles/main.css */
@import "tailwindcss";
```

**Instalação com PostCSS (projetos sem Vite):**

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```javascript
// postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

**Instalação com CLI standalone:**

```bash
# Download do binário (sem Node.js)
npx @tailwindcss/cli -i src/input.css -o dist/output.css --watch
```

---

### Migração do v3 para v4

Principais mudanças do Tailwind v3 para v4:

| v3 | v4 | Observação |
|---|---|---|
| `tailwind.config.js` | `@theme` no CSS | Configuração CSS-first |
| `@tailwind base/components/utilities` | `@import "tailwindcss"` | Import único |
| `theme.extend.colors` | `@theme { --color-*: ... }` | Cores no CSS |
| `darkMode: 'class'` | Automático via `prefers-color-scheme` | Usa `dark:` sem config |
| `content: ['./src/**/*.tsx']` | Detecção automática | Sem config de content |
| PostCSS + autoprefixer | `@tailwindcss/postcss` ou `@tailwindcss/vite` | Plugin unificado |

**Ferramenta de migração automática:**

```bash
npx @tailwindcss/upgrade
```

---

### Classes Utilitárias Essenciais

**Layout:**

| Classe | CSS gerado | Uso |
|---|---|---|
| `flex` | `display: flex` | Container flexível |
| `grid` | `display: grid` | Container grid |
| `hidden` | `display: none` | Ocultar elemento |
| `block` | `display: block` | Elemento de bloco |
| `inline-flex` | `display: inline-flex` | Flex inline |
| `flex-col` | `flex-direction: column` | Coluna |
| `flex-row` | `flex-direction: row` | Linha (padrão) |
| `flex-wrap` | `flex-wrap: wrap` | Quebrar linha |
| `items-center` | `align-items: center` | Centralizar vertical |
| `justify-between` | `justify-content: space-between` | Espaçar horizontal |
| `justify-center` | `justify-content: center` | Centralizar horizontal |
| `gap-4` | `gap: 1rem` | Espaçamento entre filhos |
| `grid-cols-3` | `grid-template-columns: repeat(3, 1fr)` | 3 colunas iguais |

**Espaçamento (padding/margin):**

| Classe | CSS gerado |
|---|---|
| `p-4` | `padding: 1rem` |
| `px-6` | `padding-left: 1.5rem; padding-right: 1.5rem` |
| `py-2` | `padding-top: 0.5rem; padding-bottom: 0.5rem` |
| `pt-8` | `padding-top: 2rem` |
| `m-4` | `margin: 1rem` |
| `mx-auto` | `margin-left: auto; margin-right: auto` |
| `mt-6` | `margin-top: 1.5rem` |
| `space-y-4` | Espaço vertical de 1rem entre filhos |

**Tipografia:**

| Classe | Efeito |
|---|---|
| `text-sm` / `text-base` / `text-lg` / `text-xl` / `text-2xl` | Tamanho da fonte |
| `font-normal` / `font-medium` / `font-semibold` / `font-bold` | Peso |
| `text-gray-500` / `text-gray-900` | Cor do texto |
| `leading-tight` / `leading-normal` / `leading-relaxed` | Altura de linha |
| `tracking-tight` / `tracking-wide` | Espaçamento entre letras |
| `text-left` / `text-center` / `text-right` | Alinhamento |
| `truncate` | Truncar com `...` |
| `line-clamp-3` | Limitar a 3 linhas |

**Dimensionamento:**

| Classe | CSS gerado |
|---|---|
| `w-full` | `width: 100%` |
| `w-1/2` | `width: 50%` |
| `w-64` | `width: 16rem` |
| `h-screen` | `height: 100vh` |
| `min-h-screen` | `min-height: 100vh` |
| `max-w-md` | `max-width: 28rem` |
| `max-w-7xl` | `max-width: 80rem` |
| `size-10` | `width: 2.5rem; height: 2.5rem` |

**Backgrounds e Bordas:**

| Classe | Efeito |
|---|---|
| `bg-white` / `bg-gray-100` / `bg-blue-600` | Cor de fundo |
| `bg-gradient-to-r from-blue-500 to-purple-600` | Gradiente |
| `border` | Borda 1px solid |
| `border-2` | Borda 2px |
| `border-gray-200` | Cor da borda |
| `rounded-md` / `rounded-lg` / `rounded-xl` / `rounded-full` | Raio da borda |
| `ring-2 ring-blue-500` | Anel de foco |
| `shadow-sm` / `shadow-md` / `shadow-lg` | Sombra |
| `divide-y` | Bordas entre filhos (vertical) |

**Efeitos e Transições:**

| Classe | Efeito |
|---|---|
| `opacity-50` | Opacidade 50% |
| `transition` | Transição padrão (colors) |
| `transition-all` | Transição de todas as propriedades |
| `duration-200` | 200ms |
| `ease-in-out` | Easing |
| `hover:bg-blue-700` | Background no hover |
| `focus:ring-2` | Anel no foco |
| `active:scale-95` | Escala no clique |

---

### Design Responsivo com Tailwind

Tailwind adota **mobile-first** — estilos sem prefixo aplicam-se ao menor viewport. Prefixos de breakpoint aplicam estilos a partir daquele tamanho.

**Breakpoints padrão:**

| Prefixo | Min-width | Dispositivo típico |
|---|---|---|
| *(sem prefixo)* | 0px | Mobile |
| `sm:` | 640px | Mobile paisagem |
| `md:` | 768px | Tablet |
| `lg:` | 1024px | Desktop |
| `xl:` | 1280px | Desktop grande |
| `2xl:` | 1536px | Monitor ultrawide |

**Exemplo — grid responsivo:**

```html
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  <div class="bg-white rounded-xl p-6 shadow-sm">Card 1</div>
  <div class="bg-white rounded-xl p-6 shadow-sm">Card 2</div>
  <div class="bg-white rounded-xl p-6 shadow-sm">Card 3</div>
  <div class="bg-white rounded-xl p-6 shadow-sm">Card 4</div>
</div>
```

**Container Queries (v4):**

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row gap-4">
    <img class="w-full @md:w-48 rounded-lg" src="..." alt="Produto" />
    <div>
      <h3 class="text-lg font-semibold">Nome do Produto</h3>
      <p class="text-gray-500 mt-1">Descrição do produto com mais detalhes.</p>
    </div>
  </div>
</div>
```

---

### Dark Mode

No Tailwind v4, o dark mode funciona automaticamente via `prefers-color-scheme`. O prefixo `dark:` aplica estilos quando o sistema ou a página estiver em modo escuro.

```html
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-2xl font-bold">Título</h1>
  <p class="text-gray-500 dark:text-gray-400">Subtítulo com cor adaptativa.</p>
  <button class="bg-blue-600 dark:bg-blue-500 text-white rounded-lg px-4 py-2
                 hover:bg-blue-700 dark:hover:bg-blue-400">
    Ação
  </button>
</div>
```

**Controle manual com classe (override da media query):**

```css
/* main.css */
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));
```

```typescript
// Toggle no JavaScript
function toggleDarkMode() {
  document.documentElement.classList.toggle('dark');
}
```

---

### Customização do Tema

No v4, o tema é configurado diretamente no CSS com `@theme`:

```css
@import "tailwindcss";

@theme {
  /* Cores da marca */
  --color-brand-50: #f0f9ff;
  --color-brand-100: #e0f2fe;
  --color-brand-500: #0ea5e9;
  --color-brand-600: #0284c7;
  --color-brand-700: #0369a1;
  --color-brand-900: #0c4a6e;

  /* Fontes */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  /* Breakpoints customizados */
  --breakpoint-xs: 475px;

  /* Espaçamento extra */
  --spacing-18: 4.5rem;
  --spacing-128: 32rem;

  /* Raio customizado */
  --radius-button: 0.625rem;

  /* Animações */
  --animate-fade-in: fade-in 0.5s ease-out;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}
```

**Integração com Design Tokens da seção 1:**

```css
@import "tailwindcss";

/* Tokens como base do tema Tailwind */
@theme {
  --color-primary: var(--color-primary);
  --color-surface: var(--color-surface);
  --color-error: var(--color-error);
  --color-success: var(--color-success);
}
```

---

### Reutilização com @apply e Componentes

**`@apply` — extrair classes repetidas (usar com moderação):**

```css
@utility btn-primary {
  @apply inline-flex items-center justify-center gap-2
         rounded-lg bg-blue-600 px-4 py-2
         text-sm font-medium text-white
         hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
         transition-colors duration-200
         disabled:opacity-50 disabled:cursor-not-allowed;
}
```

**Abordagem recomendada — componente extraído (React):**

```tsx
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

type Variant = 'primary' | 'secondary' | 'danger' | 'ghost';
type Size = 'sm' | 'md' | 'lg';

const variantStyles: Record<Variant, string> = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'border border-gray-300 text-gray-700 hover:bg-gray-50',
  danger: 'bg-red-600 text-white hover:bg-red-700',
  ghost: 'text-gray-600 hover:bg-gray-100',
};

const sizeStyles: Record<Size, string> = {
  sm: 'text-sm px-3 py-1.5',
  md: 'text-sm px-4 py-2',
  lg: 'text-base px-6 py-3',
};

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: Variant;
  size?: Size;
}

export function Button({
  variant = 'primary',
  size = 'md',
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      className={twMerge(clsx(
        'inline-flex items-center justify-center gap-2 rounded-lg font-medium',
        'transition-colors duration-200 focus:ring-2 focus:ring-offset-2',
        'disabled:opacity-50 disabled:cursor-not-allowed',
        variantStyles[variant],
        sizeStyles[size],
        className,
      ))}
      {...props}
    />
  );
}
```

**Class Variance Authority (CVA) — variantes tipadas:**

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { twMerge } from 'tailwind-merge';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors focus:ring-2 focus:ring-offset-2 disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
        secondary: 'border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-400',
        danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
        ghost: 'text-gray-600 hover:bg-gray-100 focus:ring-gray-400',
      },
      size: {
        sm: 'text-sm px-3 py-1.5',
        md: 'text-sm px-4 py-2',
        lg: 'text-base px-6 py-3',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement>
  & VariantProps<typeof buttonVariants>;

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button
      className={twMerge(buttonVariants({ variant, size }), className)}
      {...props}
    />
  );
}
```

**Padrão `cn()` helper — utilitário universal em projetos Tailwind + shadcn/ui:**

O padrão `cn()` combina `clsx` (classes condicionais) com `tailwind-merge` (resolve conflitos de classes Tailwind). É adotado como padrão em todos os projetos shadcn/ui e amplamente usado na comunidade.

```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```tsx
// Uso em componentes
import { cn } from '@/lib/utils';

function Card({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={cn(
        'rounded-xl border bg-white p-6 shadow-sm',
        className // Permite override externo sem conflito
      )}
      {...props}
    />
  );
}

// Quem usa o Card pode sobrescrever estilos sem conflito:
<Card className="p-8 shadow-lg" />
// Resultado: "rounded-xl border bg-white p-8 shadow-lg"
// (p-6 foi substituído por p-8, shadow-sm por shadow-lg)
```

---

### Plugins Oficiais e da Comunidade

**@tailwindcss/typography — prosa estilizada:**

```bash
npm install @tailwindcss/typography
```

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

```html
<article class="prose lg:prose-xl dark:prose-invert max-w-none">
  <h1>Título do Artigo</h1>
  <p>Conteúdo renderizado a partir de Markdown ou CMS, com tipografia otimizada.</p>
  <pre><code>const x = 42;</code></pre>
</article>
```

**@tailwindcss/forms — reset de formulários:**

```bash
npm install @tailwindcss/forms
```

```css
@plugin "@tailwindcss/forms";
```

```html
<input type="text" class="rounded-lg border-gray-300 focus:border-blue-500 focus:ring-blue-500" />
<select class="rounded-lg border-gray-300 focus:border-blue-500 focus:ring-blue-500">
  <option>Opção 1</option>
</select>
```

**@tailwindcss/container-queries:**

```bash
npm install @tailwindcss/container-queries
```

```css
@plugin "@tailwindcss/container-queries";
```

---

### Tailwind + React

**Setup completo (Vite + React + TypeScript):**

```bash
npm create vite@latest meu-app -- --template react-ts
cd meu-app
npm install tailwindcss @tailwindcss/vite
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

**Exemplo de componente — Card de produto:**

```tsx
interface ProductCardProps {
  name: string;
  price: number;
  image: string;
  discount?: number;
}

export function ProductCard({ name, price, image, discount }: ProductCardProps) {
  const finalPrice = discount ? price * (1 - discount / 100) : price;

  return (
    <div className="group relative overflow-hidden rounded-xl bg-white shadow-sm
                    hover:shadow-md transition-shadow duration-300">
      <div className="aspect-square overflow-hidden">
        <img
          src={image}
          alt={name}
          className="h-full w-full object-cover transition-transform duration-300
                     group-hover:scale-105"
        />
      </div>

      {discount && (
        <span className="absolute top-3 right-3 rounded-full bg-red-500 px-2.5 py-0.5
                         text-xs font-semibold text-white">
          -{discount}%
        </span>
      )}

      <div className="p-4">
        <h3 className="text-sm font-medium text-gray-900 line-clamp-2">{name}</h3>
        <div className="mt-2 flex items-baseline gap-2">
          <span className="text-lg font-bold text-gray-900">
            R$ {finalPrice.toFixed(2)}
          </span>
          {discount && (
            <span className="text-sm text-gray-400 line-through">
              R$ {price.toFixed(2)}
            </span>
          )}
        </div>
      </div>
    </div>
  );
}
```

---

### Tailwind + Vue

```html
<!-- ProductCard.vue -->
<script setup lang="ts">
interface Props {
  name: string;
  price: number;
  image: string;
  discount?: number;
}

const props = defineProps<Props>();
const finalPrice = computed(() =>
  props.discount ? props.price * (1 - props.discount / 100) : props.price
);
</script>

<template>
  <div class="group relative overflow-hidden rounded-xl bg-white shadow-sm
              hover:shadow-md transition-shadow duration-300">
    <div class="aspect-square overflow-hidden">
      <img
        :src="image"
        :alt="name"
        class="h-full w-full object-cover transition-transform duration-300
               group-hover:scale-105"
      />
    </div>

    <span
      v-if="discount"
      class="absolute top-3 right-3 rounded-full bg-red-500 px-2.5 py-0.5
             text-xs font-semibold text-white"
    >
      -{{ discount }}%
    </span>

    <div class="p-4">
      <h3 class="text-sm font-medium text-gray-900 line-clamp-2">{{ name }}</h3>
      <div class="mt-2 flex items-baseline gap-2">
        <span class="text-lg font-bold text-gray-900">
          R$ {{ finalPrice.toFixed(2) }}
        </span>
        <span v-if="discount" class="text-sm text-gray-400 line-through">
          R$ {{ price.toFixed(2) }}
        </span>
      </div>
    </div>
  </div>
</template>
```

---

### Tailwind + Angular

**Instalação:**

```bash
ng new meu-app
cd meu-app
npm install tailwindcss @tailwindcss/postcss postcss
```

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

```css
/* src/styles.css */
@import "tailwindcss";
```

**Componente Angular:**

```typescript
// product-card.component.ts
import { Component, input, computed } from '@angular/core';

@Component({
  selector: 'app-product-card',
  standalone: true,
  template: `
    <div class="group relative overflow-hidden rounded-xl bg-white shadow-sm
                hover:shadow-md transition-shadow duration-300">
      <div class="aspect-square overflow-hidden">
        <img
          [src]="image()"
          [alt]="name()"
          class="h-full w-full object-cover transition-transform duration-300
                 group-hover:scale-105"
        />
      </div>

      @if (discount()) {
        <span class="absolute top-3 right-3 rounded-full bg-red-500 px-2.5 py-0.5
                     text-xs font-semibold text-white">
          -{{ discount() }}%
        </span>
      }

      <div class="p-4">
        <h3 class="text-sm font-medium text-gray-900 line-clamp-2">{{ name() }}</h3>
        <div class="mt-2 flex items-baseline gap-2">
          <span class="text-lg font-bold text-gray-900">
            R$ {{ finalPrice().toFixed(2) }}
          </span>
          @if (discount()) {
            <span class="text-sm text-gray-400 line-through">
              R$ {{ price().toFixed(2) }}
            </span>
          }
        </div>
      </div>
    </div>
  `,
})
export class ProductCardComponent {
  name = input.required<string>();
  price = input.required<number>();
  image = input.required<string>();
  discount = input<number>();

  finalPrice = computed(() => {
    const d = this.discount();
    return d ? this.price() * (1 - d / 100) : this.price();
  });
}
```

---

### Tailwind vs CSS Tradicional vs CSS-in-JS

| Aspecto | Tailwind CSS | CSS Modules | styled-components | CSS/SCSS Tradicional |
|---|---|---|---|---|
| **Abordagem** | Utility-first | Scoped classes | CSS-in-JS | Global ou BEM |
| **Bundle size** | Muito pequeno (purge) | Pequeno | Maior (runtime JS) | Variável |
| **Custo runtime** | Zero | Zero | Sim (gera CSS no JS) | Zero |
| **Tipagem** | Via plugins IDE | Possível (typed-css-modules) | Nativa (TypeScript) | Não |
| **Theming** | `@theme` + `dark:` | CSS Custom Properties | `ThemeProvider` | SCSS variables |
| **DX (IntelliSense)** | Excelente (plugin VS Code) | Básica | Boa | Básica |
| **SSR** | Sem problemas | Sem problemas | Requer config extra | Sem problemas |
| **Curva de aprendizado** | Média (decorar classes) | Baixa | Média | Baixa |
| **Design System** | Tema centralizado | Manual | Provider | SCSS variables |
| **Quando usar** | SPAs, projetos novos, prototipação rápida | Projetos que valorizam CSS puro | Projetos React com theming dinâmico | Projetos legados, times que dominam CSS |

---

## 3. Bootstrap 5

Bootstrap é um framework CSS component-based que fornece componentes prontos (Navbar, Modal, Card, etc.) e um sistema de grid responsivo, acelerando o desenvolvimento de interfaces consistentes.

> Referência: [Bootstrap 5 Docs](https://getbootstrap.com/docs/) · [Bootstrap Icons](https://icons.getbootstrap.com/)

---

### Visão Geral e Filosofia

Bootstrap adota uma abordagem **component-first** — componentes pré-estilizados que funcionam imediatamente, com customização via SCSS variables.

**Quando usar Bootstrap vs Tailwind:**

| Cenário | Recomendação |
|---|---|
| Protótipo rápido com componentes prontos | Bootstrap |
| Design totalmente customizado | Tailwind |
| Time com pouca experiência CSS | Bootstrap |
| Projeto com design system próprio | Tailwind |
| Admin panel / dashboard | Bootstrap |
| Landing page com design único | Tailwind |
| Projeto que precisa funcionar sem build | Bootstrap (CDN) |

**Mudanças do Bootstrap 4 para 5:**

- Remoção do jQuery como dependência
- Novo sistema de utilitários expandido
- RTL (right-to-left) nativo
- Componente Offcanvas adicionado
- CSS Custom Properties em todos os componentes
- Novo componente Accordion
- Grid atualizado com suporte a `xxl`

---

### Instalação e Configuração

**Via npm (recomendado):**

```bash
npm install bootstrap @popperjs/core
```

```typescript
// main.ts — import seletivo
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap.bundle.min.js';
```

**Import seletivo de SCSS (reduz bundle):**

```scss
// styles.scss — importar apenas o necessário
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/mixins";
@import "bootstrap/scss/root";
@import "bootstrap/scss/reboot";
@import "bootstrap/scss/type";
@import "bootstrap/scss/containers";
@import "bootstrap/scss/grid";
@import "bootstrap/scss/buttons";
@import "bootstrap/scss/card";
@import "bootstrap/scss/modal";
@import "bootstrap/scss/navbar";
@import "bootstrap/scss/forms";
@import "bootstrap/scss/utilities";
@import "bootstrap/scss/utilities/api";
```

**Via CDN (sem build):**

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
      rel="stylesheet" />
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

---

### Sistema de Grid

O grid do Bootstrap usa um sistema de **12 colunas** com containers, linhas e colunas responsivas.

```html
<!-- Container centralizado com largura máxima -->
<div class="container">
  <div class="row g-4">
    <!-- 12 colunas no mobile, 6 no tablet, 4 no desktop -->
    <div class="col-12 col-md-6 col-lg-4">
      <div class="card">Conteúdo 1</div>
    </div>
    <div class="col-12 col-md-6 col-lg-4">
      <div class="card">Conteúdo 2</div>
    </div>
    <div class="col-12 col-md-12 col-lg-4">
      <div class="card">Conteúdo 3</div>
    </div>
  </div>
</div>
```

**Breakpoints do Bootstrap 5:**

| Classe | Breakpoint | Exemplo de uso |
|---|---|---|
| *(nenhum)* | <576px | `col-12` |
| `sm` | ≥576px | `col-sm-6` |
| `md` | ≥768px | `col-md-4` |
| `lg` | ≥992px | `col-lg-3` |
| `xl` | ≥1200px | `col-xl-2` |
| `xxl` | ≥1400px | `col-xxl-1` |

**Layout com sidebar e conteúdo principal:**

```html
<div class="container-fluid">
  <div class="row">
    <!-- Sidebar — oculta no mobile, 3 colunas no desktop -->
    <nav class="col-lg-3 d-none d-lg-block bg-light vh-100 p-3">
      <h5>Menu</h5>
      <ul class="nav flex-column">
        <li class="nav-item"><a class="nav-link" href="#">Dashboard</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Produtos</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Pedidos</a></li>
      </ul>
    </nav>

    <!-- Conteúdo principal — 12 colunas no mobile, 9 no desktop -->
    <main class="col-12 col-lg-9 p-4">
      <h1>Dashboard</h1>
      <div class="row g-3">
        <div class="col-sm-6 col-xl-3">
          <div class="card text-bg-primary p-3">
            <div class="card-body">
              <h6 class="card-title">Vendas</h6>
              <p class="card-text fs-3 fw-bold">R$ 45.230</p>
            </div>
          </div>
        </div>
        <div class="col-sm-6 col-xl-3">
          <div class="card text-bg-success p-3">
            <div class="card-body">
              <h6 class="card-title">Pedidos</h6>
              <p class="card-text fs-3 fw-bold">1.243</p>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</div>
```

---

### Componentes Essenciais

**Navbar responsiva:**

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">
    <a class="navbar-brand" href="#">MeuApp</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
            data-bs-target="#navbarNav" aria-controls="navbarNav"
            aria-expanded="false" aria-label="Alternar navegação">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav me-auto">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#">Home</a>
        </li>
        <li class="nav-item"><a class="nav-link" href="#">Produtos</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Contato</a></li>
      </ul>
      <form class="d-flex" role="search">
        <input class="form-control me-2" type="search" placeholder="Buscar"
               aria-label="Buscar" />
        <button class="btn btn-outline-light" type="submit">Buscar</button>
      </form>
    </div>
  </div>
</nav>
```

**Card com imagem:**

```html
<div class="card h-100 shadow-sm">
  <img src="produto.jpg" class="card-img-top" alt="Nome do produto" />
  <div class="card-body d-flex flex-column">
    <h5 class="card-title">Nome do Produto</h5>
    <p class="card-text text-muted flex-grow-1">Descrição breve do produto.</p>
    <div class="d-flex justify-content-between align-items-center mt-3">
      <span class="fs-5 fw-bold text-primary">R$ 99,90</span>
      <button class="btn btn-primary btn-sm">Comprar</button>
    </div>
  </div>
</div>
```

**Modal:**

```html
<button type="button" class="btn btn-danger" data-bs-toggle="modal"
        data-bs-target="#confirmModal">
  Excluir
</button>

<div class="modal fade" id="confirmModal" tabindex="-1"
     aria-labelledby="confirmModalLabel" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="confirmModalLabel">Confirmar exclusão</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"
                aria-label="Fechar"></button>
      </div>
      <div class="modal-body">
        Tem certeza que deseja excluir este item? Esta ação não pode ser desfeita.
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary"
                data-bs-dismiss="modal">Cancelar</button>
        <button type="button" class="btn btn-danger">Confirmar Exclusão</button>
      </div>
    </div>
  </div>
</div>
```

**Formulário com validação:**

```html
<form class="needs-validation" novalidate>
  <div class="mb-3">
    <label for="nome" class="form-label">Nome completo</label>
    <input type="text" class="form-control" id="nome" required
           minlength="3" />
    <div class="invalid-feedback">Informe seu nome (mínimo 3 caracteres).</div>
  </div>

  <div class="mb-3">
    <label for="email" class="form-label">E-mail</label>
    <input type="email" class="form-control" id="email" required />
    <div class="invalid-feedback">Informe um e-mail válido.</div>
  </div>

  <div class="mb-3">
    <label for="categoria" class="form-label">Categoria</label>
    <select class="form-select" id="categoria" required>
      <option value="">Selecione...</option>
      <option value="tech">Tecnologia</option>
      <option value="saude">Saúde</option>
    </select>
    <div class="invalid-feedback">Selecione uma categoria.</div>
  </div>

  <div class="form-check mb-3">
    <input class="form-check-input" type="checkbox" id="termos" required />
    <label class="form-check-label" for="termos">
      Aceito os termos de uso
    </label>
    <div class="invalid-feedback">Você deve aceitar os termos.</div>
  </div>

  <button class="btn btn-primary" type="submit">Cadastrar</button>
</form>

<script>
document.querySelector('.needs-validation').addEventListener('submit', (e) => {
  if (!e.target.checkValidity()) {
    e.preventDefault();
    e.stopPropagation();
  }
  e.target.classList.add('was-validated');
});
</script>
```

**Tabela de todos os componentes do Bootstrap 5:**

| Componente | Descrição | Uso típico |
|---|---|---|
| **Accordion** | Painéis colapsáveis | FAQs, seções expansíveis |
| **Alert** | Mensagem de feedback | Sucesso, erro, aviso |
| **Badge** | Rótulo/contador | Notificações, status |
| **Breadcrumb** | Trilha de navegação | Hierarquia de páginas |
| **Button** | Botão estilizado | Ações primárias/secundárias |
| **Card** | Container de conteúdo | Listagem de itens |
| **Carousel** | Slideshow | Banners, galerias |
| **Dropdown** | Menu suspenso | Menus, filtros |
| **List Group** | Lista estilizada | Menus laterais, listas |
| **Modal** | Diálogo sobreposto | Confirmações, formulários |
| **Navbar** | Barra de navegação | Header principal |
| **Offcanvas** | Painel lateral deslizante | Menus mobile, filtros |
| **Pagination** | Navegação de páginas | Listas paginadas |
| **Popover** | Tooltip expandido | Informações contextuais |
| **Progress** | Barra de progresso | Upload, processos |
| **Spinner** | Indicador de carregamento | Ações assíncronas |
| **Tab** | Abas de conteúdo | Organização em abas |
| **Toast** | Notificação temporária | Feedback de ações |
| **Tooltip** | Dica de texto | Explicações rápidas |

---

### Classes Utilitárias

O Bootstrap 5 inclui um sistema de utilitários que cobre os cenários mais comuns:

**Espaçamento (margin e padding):**

Formato: `{propriedade}{lados}-{breakpoint}-{tamanho}`

| Propriedade | Valores | Exemplo |
|---|---|---|
| `m` (margin) | 0, 1, 2, 3, 4, 5, auto | `mt-3` (margin-top: 1rem) |
| `p` (padding) | 0, 1, 2, 3, 4, 5 | `px-4` (padding horizontal: 1.5rem) |
| Lados: `t` `b` `s` `e` `x` `y` | | `mb-lg-5` (margin-bottom 3rem em lg+) |

**Display e Flex:**

```html
<!-- Ocultar/mostrar por breakpoint -->
<div class="d-none d-md-block">Visível apenas em md+</div>
<div class="d-block d-lg-none">Visível até lg</div>

<!-- Flexbox -->
<div class="d-flex justify-content-between align-items-center gap-3">
  <span>Item esquerda</span>
  <span>Item direita</span>
</div>
```

**Texto e cores:**

```html
<p class="text-primary">Texto primário</p>
<p class="text-muted">Texto secundário</p>
<p class="fw-bold fs-4 text-center">Texto grande, negrito, centralizado</p>

<div class="bg-primary text-white p-3 rounded">Fundo primário</div>
<div class="bg-light text-dark p-3 border rounded">Fundo claro com borda</div>
```

---

### Customização com Variáveis SCSS

> Para fundamentos de SCSS, consulte [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md) — seção SCSS.

As variáveis devem ser definidas **antes** do import do Bootstrap:

```scss
// custom.scss

// 1. Sobrescrever variáveis padrão
$primary: #2563eb;
$secondary: #64748b;
$success: #16a34a;
$danger: #dc2626;
$warning: #ca8a04;
$info: #0891b2;

$font-family-base: 'Inter', system-ui, sans-serif;
$font-size-base: 1rem;
$border-radius: 0.5rem;
$border-radius-lg: 0.75rem;
$border-radius-sm: 0.375rem;

$spacer: 1rem;
$enable-negative-margins: true;

$body-bg: #f8fafc;
$body-color: #1e293b;

// 2. Importar Bootstrap
@import "bootstrap/scss/bootstrap";

// 3. Estilos customizados adicionais
.card {
  border: none;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

  &:hover {
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
}
```

---

### Bootstrap + React

```bash
npm install react-bootstrap bootstrap
```

```tsx
import { Container, Row, Col, Card, Button, Modal } from 'react-bootstrap';
import { useState } from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

export function ProductList({ products }: { products: Product[] }) {
  const [selectedProduct, setSelectedProduct] = useState<Product | null>(null);

  return (
    <Container className="py-4">
      <Row xs={1} md={2} lg={3} className="g-4">
        {products.map(product => (
          <Col key={product.id}>
            <Card className="h-100 shadow-sm">
              <Card.Img variant="top" src={product.image} alt={product.name} />
              <Card.Body className="d-flex flex-column">
                <Card.Title>{product.name}</Card.Title>
                <Card.Text className="text-muted flex-grow-1">
                  R$ {product.price.toFixed(2)}
                </Card.Text>
                <Button
                  variant="primary"
                  onClick={() => setSelectedProduct(product)}
                >
                  Ver detalhes
                </Button>
              </Card.Body>
            </Card>
          </Col>
        ))}
      </Row>

      <Modal
        show={!!selectedProduct}
        onHide={() => setSelectedProduct(null)}
        centered
      >
        <Modal.Header closeButton>
          <Modal.Title>{selectedProduct?.name}</Modal.Title>
        </Modal.Header>
        <Modal.Body>
          <p className="fs-4 fw-bold text-primary">
            R$ {selectedProduct?.price.toFixed(2)}
          </p>
        </Modal.Body>
        <Modal.Footer>
          <Button variant="secondary" onClick={() => setSelectedProduct(null)}>
            Fechar
          </Button>
          <Button variant="primary">Adicionar ao carrinho</Button>
        </Modal.Footer>
      </Modal>
    </Container>
  );
}
```

---

### Bootstrap + Vue

```bash
npm install bootstrap-vue-next bootstrap
```

```html
<!-- ProductList.vue -->
<script setup lang="ts">
import { ref } from 'vue';
import { BContainer, BRow, BCol, BCard, BButton, BModal } from 'bootstrap-vue-next';

interface Product {
  id: number;
  name: string;
  price: number;
}

defineProps<{ products: Product[] }>();

const selectedProduct = ref<Product | null>(null);
const showModal = ref(false);

function openModal(product: Product) {
  selectedProduct.value = product;
  showModal.value = true;
}
</script>

<template>
  <BContainer class="py-4">
    <BRow cols="1" cols-md="2" cols-lg="3" class="g-4">
      <BCol v-for="product in products" :key="product.id">
        <BCard class="h-100 shadow-sm">
          <template #header>
            <h5 class="mb-0">{{ product.name }}</h5>
          </template>
          <p class="text-muted">R$ {{ product.price.toFixed(2) }}</p>
          <BButton variant="primary" @click="openModal(product)">
            Ver detalhes
          </BButton>
        </BCard>
      </BCol>
    </BRow>

    <BModal v-model="showModal" :title="selectedProduct?.name ?? ''" centered>
      <p class="fs-4 fw-bold text-primary">
        R$ {{ selectedProduct?.price.toFixed(2) }}
      </p>
    </BModal>
  </BContainer>
</template>
```

---

### Bootstrap + Angular

```bash
ng add @ng-bootstrap/ng-bootstrap
```

```typescript
// product-list.component.ts
import { Component, input } from '@angular/core';
import { NgbModal } from '@ng-bootstrap/ng-bootstrap';

interface Product {
  id: number;
  name: string;
  price: number;
}

@Component({
  selector: 'app-product-list',
  standalone: true,
  template: `
    <div class="container py-4">
      <div class="row g-4">
        @for (product of products(); track product.id) {
          <div class="col-12 col-md-6 col-lg-4">
            <div class="card h-100 shadow-sm">
              <div class="card-body d-flex flex-column">
                <h5 class="card-title">{{ product.name }}</h5>
                <p class="card-text text-muted flex-grow-1">
                  R$ {{ product.price.toFixed(2) }}
                </p>
                <button class="btn btn-primary" (click)="openModal(product)">
                  Ver detalhes
                </button>
              </div>
            </div>
          </div>
        }
      </div>
    </div>

    <ng-template #detailModal let-modal>
      <div class="modal-header">
        <h5 class="modal-title">{{ selectedProduct?.name }}</h5>
        <button type="button" class="btn-close" aria-label="Fechar"
                (click)="modal.dismiss()"></button>
      </div>
      <div class="modal-body">
        <p class="fs-4 fw-bold text-primary">
          R$ {{ selectedProduct?.price?.toFixed(2) }}
        </p>
      </div>
      <div class="modal-footer">
        <button class="btn btn-secondary" (click)="modal.close()">Fechar</button>
        <button class="btn btn-primary">Adicionar ao carrinho</button>
      </div>
    </ng-template>
  `,
})
export class ProductListComponent {
  products = input.required<Product[]>();
  selectedProduct: Product | null = null;

  constructor(private modalService: NgbModal) {}

  openModal(product: Product) {
    this.selectedProduct = product;
    this.modalService.open(/* template ref */);
  }
}
```

---

### Tailwind vs Bootstrap — Comparativo Detalhado

| Aspecto | Tailwind CSS | Bootstrap 5 |
|---|---|---|
| **Filosofia** | Utility-first (construa tudo) | Component-first (componentes prontos) |
| **CSS gerado** | ~10-30KB (purge agressivo) | ~25KB (minificado) + JS ~80KB |
| **Componentes prontos** | Nenhum (apenas utilitários) | 25+ componentes |
| **JavaScript incluso** | Não | Sim (Modal, Dropdown, etc.) |
| **Customização** | Total via `@theme` | Via SCSS variables |
| **Curva de aprendizado** | Média (memorizar classes) | Baixa (copiar exemplos) |
| **Design único** | Facilitado | Precisa sobrescrever muito |
| **Prototipação** | Muito rápida | Rápida |
| **Responsividade** | Mobile-first com prefixos | Mobile-first com grid 12 colunas |
| **Dark mode** | `dark:` variant nativo | `data-bs-theme="dark"` |
| **Acessibilidade** | Manual (ARIA no HTML) | Parcial (ARIA em componentes) |
| **Comunidade** | Crescendo rapidamente | Madura e estável |
| **Ideal para** | Projetos com design próprio, SPAs | Admin panels, MVPs, projetos sem designer |

---

## 4. Material UI (MUI) e Bibliotecas de Componentes

Material UI (MUI) é a biblioteca de componentes React mais popular, baseada no Material Design 3 do Google. Fornece componentes prontos, sistema de theming e integração com design tokens.

> Referência: [MUI Docs](https://mui.com/) · [Material Design 3](https://m3.material.io/) · [shadcn/ui](https://ui.shadcn.com/)

---

### Visão Geral e Material Design 3

| Pacote | Descrição |
|---|---|
| **@mui/material** | Componentes core (Button, TextField, Dialog, etc.) |
| **@mui/system** | Layout primitives (Box, Stack, Grid, Container) |
| **@mui/icons-material** | 2.100+ ícones do Material Design |
| **@mui/x-data-grid** | DataGrid avançado (ordenação, filtro, paginação) |
| **@mui/x-date-pickers** | Date/Time pickers |
| **@mui/lab** | Componentes experimentais |

O MUI implementa os princípios do **Material Design 3**: Dynamic Color, tipografia expressiva, formas arredondadas e elevação tonal.

---

### Instalação e Configuração do MUI

```bash
npm install @mui/material @emotion/react @emotion/styled @mui/icons-material
```

**Setup no App (React):**

```tsx
// App.tsx
import { ThemeProvider, CssBaseline } from '@mui/material';
import { theme } from './theme';

export function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {/* Conteúdo da aplicação */}
    </ThemeProvider>
  );
}
```

---

### Sistema de Theming

O tema do MUI centraliza todas as decisões de design: cores, tipografia, espaçamento, formas e breakpoints.

```tsx
// theme.ts
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  cssVariables: true,
  palette: {
    primary: {
      main: '#2563eb',
      light: '#60a5fa',
      dark: '#1d4ed8',
      contrastText: '#ffffff',
    },
    secondary: {
      main: '#7c3aed',
    },
    error: {
      main: '#dc2626',
    },
    success: {
      main: '#16a34a',
    },
    background: {
      default: '#f8fafc',
      paper: '#ffffff',
    },
    text: {
      primary: '#1e293b',
      secondary: '#64748b',
    },
  },
  typography: {
    fontFamily: '"Inter", system-ui, sans-serif',
    h1: { fontSize: '2.25rem', fontWeight: 700, lineHeight: 1.25 },
    h2: { fontSize: '1.875rem', fontWeight: 600, lineHeight: 1.3 },
    h3: { fontSize: '1.5rem', fontWeight: 600, lineHeight: 1.35 },
    h4: { fontSize: '1.25rem', fontWeight: 600, lineHeight: 1.4 },
    body1: { fontSize: '1rem', lineHeight: 1.5 },
    body2: { fontSize: '0.875rem', lineHeight: 1.5 },
    button: { textTransform: 'none', fontWeight: 600 },
  },
  shape: {
    borderRadius: 8,
  },
  spacing: 8,
});
```

**Tema dark com toggle:**

```tsx
import { createTheme, ThemeProvider, CssBaseline } from '@mui/material';
import { useState, useMemo } from 'react';

export function AppWithThemeToggle() {
  const [mode, setMode] = useState<'light' | 'dark'>('light');

  const theme = useMemo(() => createTheme({
    palette: {
      mode,
      primary: { main: '#2563eb' },
    },
  }), [mode]);

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <button onClick={() => setMode(prev => prev === 'light' ? 'dark' : 'light')}>
        Alternar tema
      </button>
    </ThemeProvider>
  );
}
```

---

### Design Tokens no MUI

Com `cssVariables: true`, o MUI gera CSS Custom Properties que podem ser usadas fora de componentes React:

```tsx
const theme = createTheme({ cssVariables: true });
```

```css
/* CSS Custom Properties geradas automaticamente */
.custom-element {
  color: var(--mui-palette-primary-main);
  background: var(--mui-palette-background-paper);
  padding: var(--mui-spacing-2);
  border-radius: var(--mui-shape-borderRadius);
}
```

**Acessando tokens em componentes via `sx` prop:**

```tsx
import { Box, Typography } from '@mui/material';

<Box sx={{
  bgcolor: 'background.paper',
  p: 3,
  borderRadius: 2,
  boxShadow: 1,
}}>
  <Typography variant="h4" color="primary">
    Título com tokens do tema
  </Typography>
  <Typography variant="body2" color="text.secondary" sx={{ mt: 1 }}>
    Subtítulo com cor semântica
  </Typography>
</Box>
```

---

### Componentes Essenciais do MUI

**Button com variantes:**

```tsx
import { Button, Stack, IconButton } from '@mui/material';
import { Delete, Add, Send } from '@mui/icons-material';

<Stack direction="row" spacing={2}>
  <Button variant="contained">Primário</Button>
  <Button variant="outlined">Secundário</Button>
  <Button variant="text">Texto</Button>
  <Button variant="contained" color="error" startIcon={<Delete />}>
    Excluir
  </Button>
  <Button variant="contained" endIcon={<Send />}>
    Enviar
  </Button>
  <IconButton color="primary" aria-label="Adicionar">
    <Add />
  </IconButton>
</Stack>
```

**TextField com validação:**

```tsx
import { TextField } from '@mui/material';
import { useState } from 'react';

function EmailField() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function validate(value: string) {
    if (!value) return 'E-mail é obrigatório';
    if (!/\S+@\S+\.\S+/.test(value)) return 'E-mail inválido';
    return '';
  }

  return (
    <TextField
      label="E-mail"
      type="email"
      value={email}
      onChange={e => {
        setEmail(e.target.value);
        setError(validate(e.target.value));
      }}
      error={!!error}
      helperText={error || 'Informe seu e-mail'}
      fullWidth
      required
    />
  );
}
```

**Dialog (Modal):**

```tsx
import { Dialog, DialogTitle, DialogContent, DialogContentText,
         DialogActions, Button } from '@mui/material';
import { useState } from 'react';

function ConfirmDialog() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <Button variant="contained" color="error" onClick={() => setOpen(true)}>
        Excluir
      </Button>
      <Dialog
        open={open}
        onClose={() => setOpen(false)}
        aria-labelledby="confirm-title"
        aria-describedby="confirm-description"
      >
        <DialogTitle id="confirm-title">Confirmar exclusão</DialogTitle>
        <DialogContent>
          <DialogContentText id="confirm-description">
            Tem certeza que deseja excluir este item? Esta ação não pode ser desfeita.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setOpen(false)}>Cancelar</Button>
          <Button color="error" variant="contained" onClick={() => setOpen(false)}>
            Excluir
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
}
```

**DataGrid (MUI X):**

```bash
npm install @mui/x-data-grid
```

```tsx
import { DataGrid, type GridColDef } from '@mui/x-data-grid';

const columns: GridColDef[] = [
  { field: 'id', headerName: 'ID', width: 70 },
  { field: 'name', headerName: 'Nome', flex: 1 },
  { field: 'email', headerName: 'E-mail', flex: 1 },
  {
    field: 'role',
    headerName: 'Perfil',
    width: 120,
    renderCell: (params) => (
      <Chip label={params.value} color={params.value === 'admin' ? 'primary' : 'default'} size="small" />
    ),
  },
];

const rows = [
  { id: 1, name: 'Ana Silva', email: 'ana@email.com', role: 'admin' },
  { id: 2, name: 'Carlos Lima', email: 'carlos@email.com', role: 'user' },
];

<DataGrid
  rows={rows}
  columns={columns}
  pageSizeOptions={[5, 10, 25]}
  initialState={{ pagination: { paginationModel: { pageSize: 5 } } }}
  disableRowSelectionOnClick
  autoHeight
/>
```

---

### Layout e Grid (MUI System)

```tsx
import { Box, Stack, Grid, Container } from '@mui/material';

// Box — div estilizada com acesso ao tema
<Box sx={{ p: 2, bgcolor: 'background.paper', borderRadius: 1 }}>
  Conteúdo
</Box>

// Stack — layout vertical ou horizontal com espaçamento
<Stack direction="row" spacing={2} alignItems="center">
  <Avatar />
  <Typography>Nome</Typography>
</Stack>

// Grid — layout responsivo
<Grid container spacing={3}>
  <Grid size={{ xs: 12, md: 6, lg: 4 }}>
    <Card>Item 1</Card>
  </Grid>
  <Grid size={{ xs: 12, md: 6, lg: 4 }}>
    <Card>Item 2</Card>
  </Grid>
  <Grid size={{ xs: 12, md: 12, lg: 4 }}>
    <Card>Item 3</Card>
  </Grid>
</Grid>

// Container — centraliza conteúdo com largura máxima
<Container maxWidth="lg">
  Conteúdo centralizado
</Container>
```

---

### Customização Avançada

**`sx` prop — estilos inline com acesso ao tema:**

```tsx
<Box sx={{
  display: 'flex',
  gap: 2,
  p: { xs: 2, md: 4 },
  bgcolor: 'background.paper',
  borderRadius: 2,
  '&:hover': {
    boxShadow: 3,
  },
}}>
  Conteúdo responsivo
</Box>
```

**`styled()` — componente estilizado reutilizável:**

```tsx
import { styled } from '@mui/material/styles';
import { Card } from '@mui/material';

const StyledCard = styled(Card)(({ theme }) => ({
  padding: theme.spacing(3),
  borderRadius: theme.shape.borderRadius * 2,
  transition: 'box-shadow 0.2s ease',
  '&:hover': {
    boxShadow: theme.shadows[4],
  },
}));

<StyledCard>Conteúdo com estilo customizado</StyledCard>
```

**Slots e slotProps — customização de partes internas:**

```tsx
<TextField
  label="Buscar"
  slotProps={{
    input: {
      startAdornment: <SearchIcon sx={{ mr: 1, color: 'text.secondary' }} />,
    },
    inputLabel: {
      shrink: true,
    },
  }}
/>
```

**Style overrides globais no tema:**

```tsx
const theme = createTheme({
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          borderRadius: '0.625rem',
          padding: '0.5rem 1.25rem',
        },
        containedPrimary: {
          '&:hover': {
            boxShadow: '0 4px 12px rgba(37, 99, 235, 0.4)',
          },
        },
      },
      defaultProps: {
        disableElevation: true,
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          borderRadius: '0.75rem',
          boxShadow: '0 1px 3px rgba(0,0,0,0.1)',
        },
      },
    },
  },
});
```

---

### MUI + React Hook Form

> Para detalhes sobre React Hook Form e Zod, consulte [Frontend-React.md](Frontend-React.md).

```tsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { TextField, Button, Stack, MenuItem, Alert } from '@mui/material';

const schema = z.object({
  name: z.string().min(3, 'Nome deve ter no mínimo 3 caracteres'),
  email: z.string().email('E-mail inválido'),
  role: z.enum(['admin', 'editor', 'viewer'], {
    required_error: 'Selecione um perfil',
  }),
});

type FormData = z.infer<typeof schema>;

export function UserForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const { control, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: { name: '', email: '', role: undefined },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <Stack spacing={3}>
        <Controller
          name="name"
          control={control}
          render={({ field }) => (
            <TextField
              {...field}
              label="Nome"
              error={!!errors.name}
              helperText={errors.name?.message}
              fullWidth
              required
            />
          )}
        />

        <Controller
          name="email"
          control={control}
          render={({ field }) => (
            <TextField
              {...field}
              label="E-mail"
              type="email"
              error={!!errors.email}
              helperText={errors.email?.message}
              fullWidth
              required
            />
          )}
        />

        <Controller
          name="role"
          control={control}
          render={({ field }) => (
            <TextField
              {...field}
              select
              label="Perfil"
              error={!!errors.role}
              helperText={errors.role?.message}
              fullWidth
              required
            >
              <MenuItem value="admin">Administrador</MenuItem>
              <MenuItem value="editor">Editor</MenuItem>
              <MenuItem value="viewer">Visualizador</MenuItem>
            </TextField>
          )}
        />

        <Button
          type="submit"
          variant="contained"
          disabled={isSubmitting}
          size="large"
        >
          {isSubmitting ? 'Salvando...' : 'Salvar'}
        </Button>
      </Stack>
    </form>
  );
}
```

---

### Angular Material

Angular Material é a implementação oficial do Material Design para Angular, equivalente ao MUI para React.

```bash
ng add @angular/material
```

**Setup do tema:**

```scss
// styles.scss
@use '@angular/material' as mat;

$my-theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: mat.$blue-palette,
    tertiary: mat.$violet-palette,
  ),
  typography: (
    brand-family: 'Inter',
    plain-family: 'Inter',
  ),
  density: (
    scale: 0,
  ),
));

html {
  @include mat.all-component-themes($my-theme);
}
```

**Componente com Angular Material:**

```typescript
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatInputModule } from '@angular/material/input';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatDialogModule, MatDialog } from '@angular/material/dialog';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [
    MatButtonModule, MatCardModule, MatInputModule,
    MatFormFieldModule, FormsModule,
  ],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>Cadastro de Usuário</mat-card-title>
      </mat-card-header>
      <mat-card-content>
        <mat-form-field appearance="outline" class="w-100">
          <mat-label>Nome</mat-label>
          <input matInput [(ngModel)]="name" required />
          <mat-error>Nome é obrigatório</mat-error>
        </mat-form-field>

        <mat-form-field appearance="outline" class="w-100">
          <mat-label>E-mail</mat-label>
          <input matInput type="email" [(ngModel)]="email" required />
          <mat-error>E-mail inválido</mat-error>
        </mat-form-field>
      </mat-card-content>
      <mat-card-actions align="end">
        <button mat-button>Cancelar</button>
        <button mat-flat-button color="primary">Salvar</button>
      </mat-card-actions>
    </mat-card>
  `,
})
export class UserFormComponent {
  name = '';
  email = '';
}
```

---

### shadcn/ui — Alternativa com Radix + Tailwind

shadcn/ui não é uma biblioteca instalável — é uma coleção de componentes que você **copia para o seu projeto**, mantendo controle total sobre o código. Usa Radix UI (primitivos acessíveis) + Tailwind CSS.

**Instalação:**

```bash
npx shadcn@latest init
npx shadcn@latest add button dialog card table form
```

**Exemplo — Button:**

```tsx
import { Button } from '@/components/ui/button';

<Button variant="default">Primário</Button>
<Button variant="outline">Secundário</Button>
<Button variant="destructive">Excluir</Button>
<Button variant="ghost">Ghost</Button>
```

**Exemplo — Dialog:**

```tsx
import {
  Dialog, DialogContent, DialogDescription, DialogFooter,
  DialogHeader, DialogTitle, DialogTrigger,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';

<Dialog>
  <DialogTrigger asChild>
    <Button variant="destructive">Excluir</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Confirmar exclusão</DialogTitle>
      <DialogDescription>
        Esta ação não pode ser desfeita.
      </DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <Button variant="outline">Cancelar</Button>
      <Button variant="destructive">Confirmar</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

**Quando escolher shadcn/ui vs MUI:**

| Aspecto | shadcn/ui | MUI |
|---|---|---|
| **Estilização** | Tailwind (utility-first) | Emotion (CSS-in-JS) |
| **Controle** | Total (código é seu) | Via API do componente |
| **Bundle** | Inclui só o que usa | Maior (runtime Emotion) |
| **Acessibilidade** | Radix (excelente) | Boa, mas menos granular |
| **Theming** | Tailwind `@theme` + CSS vars | `createTheme()` |
| **Componentes prontos** | ~50 | ~100+ (incluindo MUI X) |
| **DataGrid avançado** | Não (usar TanStack Table) | Sim (MUI X DataGrid) |
| **Ideal para** | Projetos com design próprio | Projetos que seguem Material Design |

---

### Outras Bibliotecas de Componentes

| Biblioteca | Framework | Filosofia | Componentes | Theming |
|---|---|---|---|---|
| **Ant Design** | React | Enterprise UI | 70+ | ConfigProvider + Design Tokens |
| **Chakra UI** | React | Acessibilidade + DX | 60+ | CSS-in-JS + theme tokens |
| **PrimeReact** | React | Temas prontos | 90+ | Theme Designer + SCSS |
| **PrimeVue** | Vue | Temas prontos | 90+ | Theme Designer + SCSS |
| **Vuetify** | Vue | Material Design | 80+ | SCSS + createVuetify |
| **Quasar** | Vue | Cross-platform (web, mobile, desktop) | 70+ | SCSS variables |
| **PrimeNG** | Angular | Temas prontos | 90+ | Theme Designer + SCSS |
| **Radix UI** | React | Headless (sem estilo) | 30+ | Estilo manual (CSS/Tailwind) |
| **Headless UI** | React, Vue | Headless (sem estilo) | 10 | Estilo manual |
| **Mantine** | React | DX + hooks | 100+ | CSS modules + theme |

---

### Comparativo: MUI vs shadcn/ui vs Radix vs Headless UI

| Critério | MUI | shadcn/ui | Radix UI | Headless UI |
|---|---|---|---|---|
| **Estilo incluído** | Sim (Material Design) | Sim (Tailwind) | Não (headless) | Não (headless) |
| **Framework** | React | React | React | React, Vue |
| **Bundle size** | ~80KB+ | ~0 (código local) | ~5-15KB por componente | ~3-8KB por componente |
| **Customização visual** | Média (override de tema) | Total (código é seu) | Total (CSS manual) | Total (CSS manual) |
| **Acessibilidade** | Boa | Excelente (Radix) | Excelente | Excelente |
| **Componentes complexos** | DataGrid, DatePicker, etc. | Não | Não | Não |
| **Manutenção** | Time MUI | Você mantém | Time Radix | Time Tailwind Labs |
| **Curva de aprendizado** | Média | Baixa | Baixa | Baixa |

---

### Theming Dinâmico por Tenant (White-Label)

Em aplicações SaaS, cada cliente (tenant) pode ter cores, logo e tipografia próprios. O tema deve ser carregado dinamicamente com base no tenant.

**MUI — tema dinâmico por tenant:**

```tsx
import { createTheme, ThemeProvider, CssBaseline } from '@mui/material';
import { useMemo } from 'react';

interface TenantBranding {
  primaryColor: string;
  secondaryColor: string;
  logoUrl: string;
  fontFamily?: string;
}

function useTenantTheme(branding: TenantBranding) {
  return useMemo(() => createTheme({
    cssVariables: true,
    palette: {
      primary: { main: branding.primaryColor },
      secondary: { main: branding.secondaryColor },
    },
    typography: {
      fontFamily: branding.fontFamily ?? '"Inter", system-ui, sans-serif',
    },
  }), [branding]);
}

function TenantApp({ branding }: { branding: TenantBranding }) {
  const theme = useTenantTheme(branding);

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <AppRoutes />
    </ThemeProvider>
  );
}
```

**CSS Custom Properties — tema via API:**

```typescript
async function applyTenantTheme(tenantId: string) {
  const res = await fetch(`/api/tenants/${tenantId}/branding`);
  const branding = await res.json();

  const root = document.documentElement;
  root.style.setProperty('--color-primary', branding.primaryColor);
  root.style.setProperty('--color-primary-hover', branding.primaryHover);
  root.style.setProperty('--color-secondary', branding.secondaryColor);
  root.style.setProperty('--font-sans', branding.fontFamily);
}
```

**Tailwind v4 — tema dinâmico com CSS variables:**

```css
@import "tailwindcss";

@theme {
  --color-brand: var(--tenant-primary, #2563eb);
  --color-brand-hover: var(--tenant-primary-hover, #1d4ed8);
}
```

```tsx
// Aplicar no carregamento do tenant
useEffect(() => {
  document.documentElement.style.setProperty('--tenant-primary', tenant.color);
  document.documentElement.style.setProperty('--tenant-primary-hover', tenant.hoverColor);
}, [tenant]);
```

---

### Tabelas Avançadas e Virtualização

Para tabelas com milhares de linhas, virtualização renderiza apenas as linhas visíveis no viewport, mantendo performance estável.

**TanStack Table (headless) + virtualização:**

```bash
npm install @tanstack/react-table @tanstack/react-virtual
```

```tsx
import {
  useReactTable, getCoreRowModel, getSortedRowModel,
  getFilteredRowModel, flexRender, type ColumnDef,
} from '@tanstack/react-table';
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef, useState } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

const columns: ColumnDef<User>[] = [
  { accessorKey: 'id', header: 'ID', size: 60 },
  { accessorKey: 'name', header: 'Nome' },
  { accessorKey: 'email', header: 'E-mail' },
  { accessorKey: 'role', header: 'Perfil', size: 100 },
];

function VirtualizedTable({ data }: { data: User[] }) {
  const [globalFilter, setGlobalFilter] = useState('');
  const parentRef = useRef<HTMLDivElement>(null);

  const table = useReactTable({
    data,
    columns,
    state: { globalFilter },
    onGlobalFilterChange: setGlobalFilter,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
  });

  const { rows } = table.getRowModel();

  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 48,
    overscan: 10,
  });

  return (
    <>
      <input
        placeholder="Buscar..."
        value={globalFilter}
        onChange={e => setGlobalFilter(e.target.value)}
        aria-label="Filtrar tabela"
      />
      <div ref={parentRef} style={{ height: '500px', overflow: 'auto' }}>
        <table>
          <thead>
            {table.getHeaderGroups().map(hg => (
              <tr key={hg.id}>
                {hg.headers.map(header => (
                  <th key={header.id}
                      onClick={header.column.getToggleSortingHandler()}
                      aria-sort={
                        header.column.getIsSorted() === 'asc' ? 'ascending' :
                        header.column.getIsSorted() === 'desc' ? 'descending' : undefined
                      }>
                    {flexRender(header.column.columnDef.header, header.getContext())}
                  </th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
            {virtualizer.getVirtualItems().map(virtualRow => {
              const row = rows[virtualRow.index];
              return (
                <tr key={row.id} style={{
                  position: 'absolute',
                  top: 0,
                  transform: `translateY(${virtualRow.start}px)`,
                  height: `${virtualRow.size}px`,
                }}>
                  {row.getVisibleCells().map(cell => (
                    <td key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </td>
                  ))}
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>
      <p aria-live="polite">{rows.length} resultados</p>
    </>
  );
}
```

**Quando usar virtualização vs paginação:**

| Cenário | Recomendação |
|---|---|
| < 100 linhas | Renderizar tudo (sem virtualização) |
| 100-1.000 linhas | Paginação server-side ou client-side |
| 1.000-100.000 linhas | Virtualização (TanStack Virtual, react-window) |
| > 100.000 linhas | Paginação server-side obrigatória |

---

### Dashboards e Visualização de Dados

**Bibliotecas de gráficos para React:**

| Biblioteca | Abordagem | Bundle | Ideal para |
|---|---|---|---|
| **Recharts** | Declarativa (componentes React) | ~60KB | Dashboards corporativos |
| **Chart.js + react-chartjs-2** | Imperativa (wrapper React) | ~45KB | Gráficos simples/moderados |
| **Nivo** | Declarativa (componentes React) | Modular | Visualizações ricas e interativas |
| **Visx** | Baixo nível (D3 primitives + React) | Modular | Visualizações customizadas |
| **Tremor** | Tailwind + Recharts | ~80KB | Dashboards rápidos com Tailwind |

**Exemplo com Recharts — dashboard de vendas:**

```bash
npm install recharts
```

```tsx
import {
  AreaChart, Area, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, BarChart, Bar,
} from 'recharts';

const salesData = [
  { month: 'Jan', revenue: 4000, orders: 240 },
  { month: 'Fev', revenue: 3000, orders: 198 },
  { month: 'Mar', revenue: 5000, orders: 305 },
  { month: 'Abr', revenue: 4500, orders: 278 },
  { month: 'Mai', revenue: 6000, orders: 389 },
  { month: 'Jun', revenue: 5500, orders: 345 },
];

function SalesDashboard() {
  return (
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      {/* KPI Cards */}
      <div className="lg:col-span-2 grid grid-cols-2 md:grid-cols-4 gap-4">
        <KpiCard title="Receita" value="R$ 28.000" change="+12%" />
        <KpiCard title="Pedidos" value="1.755" change="+8%" />
        <KpiCard title="Ticket Médio" value="R$ 15,95" change="-2%" />
        <KpiCard title="Conversão" value="3.2%" change="+0.5%" />
      </div>

      {/* Gráfico de área */}
      <div role="img" aria-label="Gráfico de receita mensal">
        <h3>Receita Mensal</h3>
        <ResponsiveContainer width="100%" height={300}>
          <AreaChart data={salesData}>
            <CartesianGrid strokeDasharray="3 3" />
            <XAxis dataKey="month" />
            <YAxis />
            <Tooltip formatter={(v: number) => `R$ ${v.toLocaleString()}`} />
            <Area type="monotone" dataKey="revenue" stroke="#2563eb"
                  fill="#2563eb" fillOpacity={0.1} />
          </AreaChart>
        </ResponsiveContainer>
      </div>

      {/* Gráfico de barras */}
      <div role="img" aria-label="Gráfico de pedidos mensais">
        <h3>Pedidos por Mês</h3>
        <ResponsiveContainer width="100%" height={300}>
          <BarChart data={salesData}>
            <CartesianGrid strokeDasharray="3 3" />
            <XAxis dataKey="month" />
            <YAxis />
            <Tooltip />
            <Bar dataKey="orders" fill="#7c3aed" radius={[4, 4, 0, 0]} />
          </BarChart>
        </ResponsiveContainer>
      </div>
    </div>
  );
}

function KpiCard({ title, value, change }: { title: string; value: string; change: string }) {
  const isPositive = change.startsWith('+');
  return (
    <div className="rounded-xl bg-white p-4 shadow-sm">
      <p className="text-sm text-gray-500">{title}</p>
      <p className="mt-1 text-2xl font-bold">{value}</p>
      <p className={`text-sm ${isPositive ? 'text-green-600' : 'text-red-600'}`}>
        {change} vs mês anterior
      </p>
    </div>
  );
}
```

**Acessibilidade em gráficos:**

- Usar `role="img"` + `aria-label` descritivo no container
- Fornecer tabela de dados como alternativa textual (visualmente oculta ou em aba separada)
- Não depender apenas de cor para diferenciar séries (usar padrões/hachuras)
- Tooltips devem ser acessíveis por teclado

---

### Formulários Complexos — Padrões de UX

Padrões de design para formulários que vão além de campos simples.

**Wizard / Stepper — formulário em etapas:**

```tsx
import { useState, type ReactNode } from 'react';

interface Step {
  title: string;
  content: ReactNode;
  validate?: () => boolean;
}

function FormWizard({ steps, onComplete }: { steps: Step[]; onComplete: () => void }) {
  const [current, setCurrent] = useState(0);

  function next() {
    const step = steps[current];
    if (step.validate && !step.validate()) return;
    if (current < steps.length - 1) setCurrent(c => c + 1);
    else onComplete();
  }

  return (
    <div>
      {/* Stepper visual */}
      <nav aria-label="Progresso do formulário">
        <ol className="flex gap-2">
          {steps.map((step, i) => (
            <li key={i} className="flex items-center gap-2"
                aria-current={i === current ? 'step' : undefined}>
              <span className={`rounded-full w-8 h-8 flex items-center justify-center text-sm
                ${i < current ? 'bg-green-500 text-white' :
                  i === current ? 'bg-blue-600 text-white' : 'bg-gray-200 text-gray-500'}`}>
                {i < current ? '✓' : i + 1}
              </span>
              <span className="text-sm">{step.title}</span>
            </li>
          ))}
        </ol>
      </nav>

      {/* Conteúdo da etapa */}
      <div role="group" aria-label={`Etapa ${current + 1}: ${steps[current].title}`}>
        {steps[current].content}
      </div>

      {/* Navegação */}
      <div className="flex gap-2 mt-4">
        {current > 0 && (
          <button onClick={() => setCurrent(c => c - 1)}>Voltar</button>
        )}
        <button onClick={next}>
          {current === steps.length - 1 ? 'Finalizar' : 'Próximo'}
        </button>
      </div>
    </div>
  );
}
```

**Inline editing — edição no local:**

```tsx
import { useState, useRef, useEffect } from 'react';

function InlineEdit({ value, onSave }: { value: string; onSave: (v: string) => void }) {
  const [editing, setEditing] = useState(false);
  const [draft, setDraft] = useState(value);
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (editing) inputRef.current?.focus();
  }, [editing]);

  function commit() {
    onSave(draft);
    setEditing(false);
  }

  if (!editing) {
    return (
      <button onClick={() => setEditing(true)}
              aria-label={`Editar: ${value}`}
              className="hover:bg-gray-100 rounded px-2 py-1">
        {value}
      </button>
    );
  }

  return (
    <input
      ref={inputRef}
      value={draft}
      onChange={e => setDraft(e.target.value)}
      onBlur={commit}
      onKeyDown={e => {
        if (e.key === 'Enter') commit();
        if (e.key === 'Escape') { setDraft(value); setEditing(false); }
      }}
      aria-label="Editando valor"
    />
  );
}
```

**Campos condicionais (show/hide por dependência):**

```tsx
function ConditionalForm() {
  const [deliveryType, setDeliveryType] = useState<'pickup' | 'delivery'>('pickup');

  return (
    <form>
      <fieldset>
        <legend>Tipo de entrega</legend>
        <label>
          <input type="radio" name="delivery" value="pickup"
                 checked={deliveryType === 'pickup'}
                 onChange={() => setDeliveryType('pickup')} />
          Retirar na loja
        </label>
        <label>
          <input type="radio" name="delivery" value="delivery"
                 checked={deliveryType === 'delivery'}
                 onChange={() => setDeliveryType('delivery')} />
          Entrega em domicílio
        </label>
      </fieldset>

      {deliveryType === 'delivery' && (
        <fieldset>
          <legend>Endereço de entrega</legend>
          <label htmlFor="cep">CEP</label>
          <input id="cep" required autoComplete="postal-code" />
          <label htmlFor="street">Rua</label>
          <input id="street" required autoComplete="street-address" />
        </fieldset>
      )}
    </form>
  );
}
```

**Auto-save (salvar rascunho):**

```tsx
import { useEffect, useCallback } from 'react';
import { useForm } from 'react-hook-form';
import { useDebouncedCallback } from 'use-debounce';

function AutoSaveForm() {
  const { register, watch, formState: { isDirty } } = useForm({
    defaultValues: loadDraft(),
  });

  const values = watch();

  const saveDraft = useDebouncedCallback((data: typeof values) => {
    localStorage.setItem('form-draft', JSON.stringify(data));
  }, 1000);

  useEffect(() => {
    if (isDirty) saveDraft(values);
  }, [values, isDirty, saveDraft]);

  return (
    <form>
      <input {...register('title')} placeholder="Título" />
      <textarea {...register('content')} placeholder="Conteúdo" />
      {isDirty && <span className="text-sm text-gray-400">Rascunho salvo</span>}
    </form>
  );
}

function loadDraft() {
  const stored = localStorage.getItem('form-draft');
  return stored ? JSON.parse(stored) : { title: '', content: '' };
}
```

---

## 5. Diretrizes de Design Mobile

As duas principais plataformas mobile — Android e iOS — possuem linguagens de design distintas que definem como os aplicativos devem parecer e se comportar. Compreender essas diretrizes é essencial para criar apps que se integrem naturalmente à experiência de cada plataforma.

> Referência: [Material Design 3](https://m3.material.io/) · [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)

> Para implementação de widgets e temas em Flutter, consulte [Frontend-Flutter.md](Frontend-Flutter.md).
> Para React Native com Expo, consulte [Frontend-React-Native.md](Frontend-React-Native.md).

---

### Material Design 3 (Android)

Material Design 3 (MD3), também chamado **Material You**, é a linguagem de design do Google para Android, web e multiplataforma. Introduzida com Android 12, enfatiza personalização, acessibilidade e adaptabilidade.

**Princípios fundamentais:**

| Princípio | Descrição |
|---|---|
| **Personalização** | Dynamic Color extrai cores do wallpaper do usuário |
| **Acessibilidade** | Contraste, touch targets 48dp, suporte a leitores de tela |
| **Adaptabilidade** | Layouts adaptáveis para celular, tablet, foldable e desktop |

**Sistema de Cores MD3:**

O MD3 usa um sistema de cores baseado em **paletas tonais** geradas a partir de uma cor semente.

| Role | Uso | Exemplo |
|---|---|---|
| **Primary** | Ações principais, FAB, botões | Botão "Salvar" |
| **On Primary** | Texto/ícones sobre primary | Texto do botão |
| **Primary Container** | Fundo de destaque suave | Card selecionado |
| **Secondary** | Elementos menos proeminentes | Filtros, chips |
| **Tertiary** | Acentos complementares | Badges, destaques |
| **Error** | Estados de erro | Validação de formulários |
| **Surface** | Fundo de containers | Cards, sheets |
| **On Surface** | Texto principal | Body text |
| **Outline** | Bordas e divisores | Inputs, dividers |

**Tipografia MD3:**

| Escala | Tamanho (sp) | Peso | Uso |
|---|---|---|---|
| Display Large | 57 | Regular | Headlines de marketing |
| Display Medium | 45 | Regular | Títulos hero |
| Display Small | 36 | Regular | Títulos de seção |
| Headline Large | 32 | Regular | Títulos de tela |
| Headline Medium | 28 | Regular | Subtítulos |
| Title Large | 22 | Regular | Títulos de card |
| Title Medium | 16 | Medium | Seções internas |
| Title Small | 14 | Medium | Rótulos enfatizados |
| Body Large | 16 | Regular | Texto principal |
| Body Medium | 14 | Regular | Texto secundário |
| Body Small | 12 | Regular | Captions |
| Label Large | 14 | Medium | Botões |
| Label Medium | 12 | Medium | Tabs, chips |
| Label Small | 11 | Medium | Badges |

**Sistema de Formas (Shape):**

| Forma | Raio | Uso |
|---|---|---|
| None | 0dp | Full-width containers |
| Extra Small | 4dp | Text fields |
| Small | 8dp | Chips, snackbars |
| Medium | 12dp | Cards, dialogs |
| Large | 16dp | FAB, navigation drawers |
| Extra Large | 28dp | Bottom sheets |
| Full | 50% | Avatares, badges |

**Elevação no MD3:**

O MD3 substitui sombras tradicionais por **elevação tonal** — quanto mais elevado o elemento, mais sua cor se aproxima da cor primária.

| Nível | Elevação | Uso |
|---|---|---|
| 0 | 0dp | Surface base |
| 1 | 1dp | Cards em repouso |
| 2 | 3dp | Bottom bars, top bars |
| 3 | 6dp | FAB, drawers |
| 4 | 8dp | Menus, dialogs em segundo plano |
| 5 | 12dp | Dialogs, modais |

**Componentes de navegação MD3:**

| Componente | Uso | Itens |
|---|---|---|
| **Navigation Bar** | Navegação principal (mobile) | 3-5 destinos |
| **Navigation Rail** | Navegação principal (tablet) | 3-7 destinos |
| **Navigation Drawer** | Navegação extensa | Sem limite |
| **Top App Bar** | Contexto da tela + ações | Título + até 3 ações |
| **Bottom App Bar** | Ações contextuais + FAB | FAB + até 4 ações |

---

### Human Interface Guidelines (iOS)

As Human Interface Guidelines (HIG) da Apple definem os padrões de design para iOS, iPadOS, macOS, watchOS e visionOS. O design iOS prioriza clareza, profundidade e manipulação direta.

**Princípios fundamentais:**

| Princípio | Descrição |
|---|---|
| **Clareza** | Texto legível, ícones precisos, adornos sutis |
| **Deferência** | O conteúdo é o protagonista; a UI é discreta |
| **Profundidade** | Camadas visuais, transições e feedback háptico criam contexto |

**Sistema de Cores iOS:**

iOS usa **cores semânticas do sistema** que se adaptam automaticamente a light/dark mode e alto contraste:

| Cor Semântica | Light Mode | Dark Mode | Uso |
|---|---|---|---|
| `label` | Preto | Branco | Texto principal |
| `secondaryLabel` | Cinza 60% | Cinza 60% | Texto secundário |
| `systemBackground` | Branco | Preto | Fundo principal |
| `secondarySystemBackground` | Cinza claro | Cinza escuro | Fundo de grupos |
| `systemBlue` | #007AFF | #0A84FF | Links, ações primárias |
| `systemRed` | #FF3B30 | #FF453A | Erros, exclusões |
| `systemGreen` | #34C759 | #30D158 | Sucesso |
| `separator` | Cinza 20% | Cinza 30% | Divisores |
| `tintColor` | Azul (padrão) | Azul (padrão) | Cor de destaque do app |

**Tipografia iOS (SF Pro):**

| Estilo | Tamanho (pt) | Peso | Uso |
|---|---|---|---|
| Large Title | 34 | Bold | Título principal (Navigation Bar) |
| Title 1 | 28 | Bold | Títulos de seção |
| Title 2 | 22 | Bold | Subtítulos |
| Title 3 | 20 | Semibold | Títulos de card |
| Headline | 17 | Semibold | Rótulos enfatizados |
| Body | 17 | Regular | Texto principal |
| Callout | 16 | Regular | Texto de destaque |
| Subheadline | 15 | Regular | Texto secundário |
| Footnote | 13 | Regular | Notas de rodapé |
| Caption 1 | 12 | Regular | Captions |
| Caption 2 | 11 | Regular | Micro labels |

**Dynamic Type** permite que o usuário ajuste o tamanho de texto globalmente. Apps devem suportar essa configuração.

**SF Symbols:**

SF Symbols é a biblioteca de ícones integrada do iOS com 5.000+ ícones vetoriais que se adaptam ao peso do texto, tamanho e acessibilidade automaticamente.

**Componentes de navegação iOS:**

| Componente | Uso | Posição |
|---|---|---|
| **Tab Bar** | Navegação principal | Inferior (sempre visível) |
| **Navigation Bar** | Contexto + back button | Superior |
| **Sidebar** | Navegação em iPad (split view) | Lateral esquerda |
| **Toolbar** | Ações contextuais | Inferior (em contexto) |
| **Search Bar** | Busca | Abaixo do navigation bar |

---

### Comparativo Material Design vs HIG

| Aspecto | Material Design 3 (Android) | HIG (iOS) |
|---|---|---|
| **Filosofia** | Personalização + expressão | Clareza + conteúdo primeiro |
| **Cores** | Dynamic Color (wallpaper) | Cores semânticas + tintColor |
| **Tipografia** | Roboto / scale de 15 estilos | SF Pro / Dynamic Type |
| **Ícones** | Material Symbols (outlined/filled/rounded) | SF Symbols (5.000+, adaptáveis) |
| **Formas** | Arredondadas (4dp a 28dp) | Squared rounding (iOS 7+ style) |
| **Elevação** | Tonal elevation (cor) | Layers + blur (vibrancy) |
| **Navegação principal** | Navigation Bar (inferior) | Tab Bar (inferior) |
| **Navegação de contexto** | Top App Bar + back | Navigation Bar + back arrow |
| **Back navigation** | Botão back do sistema (hardware/gesture) | Swipe-to-go-back (edge gesture) |
| **Menus contextuais** | Bottom Sheet | Action Sheet |
| **Feedback** | Snackbar | Banner / Alert |
| **Haptics** | Vibração simples | Taptic Engine (feedback rico) |
| **Pull-to-refresh** | CircularProgressIndicator | UIRefreshControl nativo |
| **FAB** | Sim (padrão) | Não (evitar) |
| **Gestos** | Swipe dismiss, long press | Swipe actions, peek & pop, force touch |

---

### Convenções de Plataforma

**Touch targets (área de toque):**

| Plataforma | Tamanho mínimo | Recomendado |
|---|---|---|
| Android (MD3) | 48x48dp | 48x48dp |
| iOS (HIG) | 44x44pt | 44x44pt |
| WCAG 2.2 | 24x24px | 44x44px |

**Safe areas:**

Regiões da tela que não são obstruídas por elementos do sistema (notch, barra de status, home indicator).

```dart
// Flutter — SafeArea
Scaffold(
  body: SafeArea(
    child: Column(
      children: [
        Text('Conteúdo seguro'),
      ],
    ),
  ),
);
```

```tsx
// React Native — SafeAreaView
import { SafeAreaView, SafeAreaProvider } from 'react-native-safe-area-context';

function Screen() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={{ flex: 1 }}>
        <Text>Conteúdo seguro</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

**Status bar:**

| Plataforma | Light background | Dark background |
|---|---|---|
| Android | Status bar escura (ícones escuros) | Status bar clara (ícones claros) |
| iOS | `.default` (ícones escuros) | `.lightContent` (ícones claros) |

```dart
// Flutter — controle da status bar
import 'package:flutter/services.dart';

SystemChrome.setSystemUIOverlayStyle(
  const SystemUiOverlayStyle(
    statusBarColor: Colors.transparent,
    statusBarIconBrightness: Brightness.dark, // Android
    statusBarBrightness: Brightness.light,    // iOS
  ),
);
```

**Haptic feedback:**

```dart
// Flutter
import 'package:flutter/services.dart';

HapticFeedback.lightImpact();   // Toque leve
HapticFeedback.mediumImpact();  // Toque médio
HapticFeedback.heavyImpact();   // Toque forte
HapticFeedback.selectionClick(); // Clique de seleção
```

```tsx
// React Native
import * as Haptics from 'expo-haptics';

Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
Haptics.selectionAsync();
```

---

### Navegação Nativa

**Padrões de navegação por plataforma:**

| Padrão | Android | iOS |
|---|---|---|
| **Voltar** | Botão back do sistema (gesture/hardware) | Swipe from left edge |
| **Menu principal** | Navigation Bar (bottom) ou Drawer | Tab Bar (bottom) |
| **Contexto da tela** | Top App Bar com título | Navigation Bar com Large Title |
| **Ações da tela** | Menu no Top App Bar (3 dots) | Toolbar buttons ou Edit |
| **Busca** | SearchView no Top App Bar | Search Bar abaixo do Navigation Bar |
| **Opções contextuais** | Bottom Sheet | Action Sheet |
| **Confirmação destrutiva** | AlertDialog | UIAlertController (Action Sheet) |

**Exemplo de tab navigation adaptativa em Flutter:**

```dart
import 'dart:io' show Platform;

class AdaptiveNavigation extends StatelessWidget {
  final int currentIndex;
  final ValueChanged<int> onIndexChanged;
  final Widget child;

  const AdaptiveNavigation({
    super.key,
    required this.currentIndex,
    required this.onIndexChanged,
    required this.child,
  });

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoTabScaffold(
        tabBar: CupertinoTabBar(
          currentIndex: currentIndex,
          onTap: onIndexChanged,
          items: const [
            BottomNavigationBarItem(
              icon: Icon(CupertinoIcons.home),
              label: 'Início',
            ),
            BottomNavigationBarItem(
              icon: Icon(CupertinoIcons.search),
              label: 'Buscar',
            ),
            BottomNavigationBarItem(
              icon: Icon(CupertinoIcons.person),
              label: 'Perfil',
            ),
          ],
        ),
        tabBuilder: (context, index) => child,
      );
    }

    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: currentIndex,
        onDestinationSelected: onIndexChanged,
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'Início',
          ),
          NavigationDestination(
            icon: Icon(Icons.search),
            label: 'Buscar',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'Perfil',
          ),
        ],
      ),
    );
  }
}
```

---

### Aplicando Material Design 3 no Flutter

> Para widgets e componentes Flutter detalhados, consulte [Frontend-Flutter.md](Frontend-Flutter.md).

**Configuração do tema MD3:**

```dart
import 'package:flutter/material.dart';

final theme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    seedColor: const Color(0xFF2563EB),
    brightness: Brightness.light,
  ),
  textTheme: const TextTheme(
    displayLarge: TextStyle(fontSize: 57, fontWeight: FontWeight.w400),
    headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.w400),
    titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.w400),
    bodyLarge: TextStyle(fontSize: 16, fontWeight: FontWeight.w400),
    labelLarge: TextStyle(fontSize: 14, fontWeight: FontWeight.w500),
  ),
);

final darkTheme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    seedColor: const Color(0xFF2563EB),
    brightness: Brightness.dark,
  ),
);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: theme,
      darkTheme: darkTheme,
      themeMode: ThemeMode.system,
      home: const HomeScreen(),
    );
  }
}
```

**Dynamic Color (Android 12+):**

```bash
flutter pub add dynamic_color
```

```dart
import 'package:dynamic_color/dynamic_color.dart';

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  static const _defaultColor = Color(0xFF2563EB);

  @override
  Widget build(BuildContext context) {
    return DynamicColorBuilder(
      builder: (ColorScheme? lightDynamic, ColorScheme? darkDynamic) {
        return MaterialApp(
          theme: ThemeData(
            useMaterial3: true,
            colorScheme: lightDynamic ?? ColorScheme.fromSeed(
              seedColor: _defaultColor,
            ),
          ),
          darkTheme: ThemeData(
            useMaterial3: true,
            colorScheme: darkDynamic ?? ColorScheme.fromSeed(
              seedColor: _defaultColor,
              brightness: Brightness.dark,
            ),
          ),
          themeMode: ThemeMode.system,
          home: const HomeScreen(),
        );
      },
    );
  }
}
```

**Componentes MD3 em Flutter:**

```dart
// Card MD3
Card(
  elevation: 1,
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
  child: Padding(
    padding: const EdgeInsets.all(16),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Título', style: Theme.of(context).textTheme.titleLarge),
        const SizedBox(height: 8),
        Text('Descrição do conteúdo',
             style: Theme.of(context).textTheme.bodyMedium),
        const SizedBox(height: 16),
        Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            TextButton(onPressed: () {}, child: const Text('Cancelar')),
            const SizedBox(width: 8),
            FilledButton(onPressed: () {}, child: const Text('Salvar')),
          ],
        ),
      ],
    ),
  ),
);

// FAB MD3
FloatingActionButton(
  onPressed: () {},
  child: const Icon(Icons.add),
);

// Navigation Bar MD3 (já mostrado na seção anterior)

// SearchBar MD3
SearchAnchor(
  builder: (BuildContext context, SearchController controller) {
    return SearchBar(
      controller: controller,
      hintText: 'Buscar produtos...',
      leading: const Icon(Icons.search),
      onTap: () => controller.openView(),
    );
  },
  suggestionsBuilder: (BuildContext context, SearchController controller) {
    return List.generate(5, (index) => ListTile(
      title: Text('Sugestão ${index + 1}'),
      onTap: () => controller.closeView('Sugestão ${index + 1}'),
    ));
  },
);
```

---

### Aplicando HIG no Flutter (Cupertino)

Flutter oferece widgets Cupertino que replicam o visual do iOS:

```dart
import 'package:flutter/cupertino.dart';

class CupertinoSettingsScreen extends StatefulWidget {
  const CupertinoSettingsScreen({super.key});

  @override
  State<CupertinoSettingsScreen> createState() => _CupertinoSettingsScreenState();
}

class _CupertinoSettingsScreenState extends State<CupertinoSettingsScreen> {
  bool notifications = true;
  bool darkMode = false;

  @override
  Widget build(BuildContext context) {
    return CupertinoPageScaffold(
      navigationBar: const CupertinoNavigationBar(
        middle: Text('Configurações'),
        previousPageTitle: 'Voltar',
      ),
      child: SafeArea(
        child: CupertinoListSection.insetGrouped(
          header: const Text('PREFERÊNCIAS'),
          children: [
            CupertinoListTile(
              title: const Text('Notificações'),
              trailing: CupertinoSwitch(
                value: notifications,
                onChanged: (value) => setState(() => notifications = value),
              ),
            ),
            CupertinoListTile(
              title: const Text('Modo escuro'),
              trailing: CupertinoSwitch(
                value: darkMode,
                onChanged: (value) => setState(() => darkMode = value),
              ),
            ),
            CupertinoListTile(
              title: const Text('Idioma'),
              additionalInfo: const Text('Português'),
              trailing: const CupertinoListTileChevron(),
              onTap: () {},
            ),
          ],
        ),
      ),
    );
  }
}
```

**Widgets adaptativos (`.adaptive`):**

Flutter oferece construtores `.adaptive` que renderizam automaticamente o widget correto para a plataforma:

| Widget Material | Widget Cupertino | Construtor adaptativo |
|---|---|---|
| `Switch` | `CupertinoSwitch` | `Switch.adaptive()` |
| `Slider` | `CupertinoSlider` | `Slider.adaptive()` |
| `CircularProgressIndicator` | `CupertinoActivityIndicator` | `CircularProgressIndicator.adaptive()` |
| `AlertDialog` | `CupertinoAlertDialog` | Decisão manual |
| `TextField` | `CupertinoTextField` | Decisão manual |

```dart
// Alerta adaptativo por plataforma
import 'dart:io' show Platform;

Future<bool?> showAdaptiveConfirmDialog(BuildContext context, String message) {
  if (Platform.isIOS) {
    return showCupertinoDialog<bool>(
      context: context,
      builder: (context) => CupertinoAlertDialog(
        title: const Text('Confirmar'),
        content: Text(message),
        actions: [
          CupertinoDialogAction(
            isDestructiveAction: true,
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Excluir'),
          ),
          CupertinoDialogAction(
            isDefaultAction: true,
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancelar'),
          ),
        ],
      ),
    );
  }

  return showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      title: const Text('Confirmar'),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: const Text('Cancelar'),
        ),
        FilledButton(
          onPressed: () => Navigator.pop(context, true),
          child: const Text('Excluir'),
        ),
      ],
    ),
  );
}
```

---

### Design Adaptativo no React Native

> Para fundamentos de React Native com Expo, consulte [Frontend-React-Native.md](Frontend-React-Native.md).

**Código específico por plataforma:**

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    padding: 16,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
      },
      android: {
        elevation: 4,
      },
    }),
  },
  header: {
    fontSize: Platform.OS === 'ios' ? 34 : 28,
    fontWeight: Platform.OS === 'ios' ? '700' : '400',
  },
});
```

**React Native Paper (Material Design):**

```bash
npx expo install react-native-paper
```

```tsx
import { Appbar, Card, Button, FAB, Searchbar } from 'react-native-paper';
import { useState } from 'react';

export function ProductScreen() {
  const [searchQuery, setSearchQuery] = useState('');

  return (
    <>
      <Appbar.Header>
        <Appbar.BackAction onPress={() => {}} />
        <Appbar.Content title="Produtos" />
        <Appbar.Action icon="filter-variant" onPress={() => {}} />
      </Appbar.Header>

      <Searchbar
        placeholder="Buscar produtos..."
        value={searchQuery}
        onChangeText={setSearchQuery}
        style={{ margin: 16 }}
      />

      <Card style={{ margin: 16 }}>
        <Card.Title title="Produto A" subtitle="Categoria X" />
        <Card.Content>
          <Text variant="bodyMedium">Descrição do produto.</Text>
        </Card.Content>
        <Card.Actions>
          <Button>Cancelar</Button>
          <Button mode="contained">Comprar</Button>
        </Card.Actions>
      </Card>

      <FAB
        icon="plus"
        style={{ position: 'absolute', right: 16, bottom: 16 }}
        onPress={() => {}}
      />
    </>
  );
}
```

---

### Design Responsivo para Tablets e Foldables

**Window Size Classes (Material Design 3):**

| Classe | Largura | Dispositivo | Layout |
|---|---|---|---|
| **Compact** | < 600dp | Celular | Coluna única |
| **Medium** | 600-840dp | Tablet retrato, foldable | Duas colunas ou rail |
| **Expanded** | > 840dp | Tablet paisagem, desktop | Três colunas ou sidebar |

**Master-Detail pattern em Flutter:**

```dart
class MasterDetailLayout extends StatelessWidget {
  final List<Item> items;
  final Item? selectedItem;
  final ValueChanged<Item> onItemSelected;

  const MasterDetailLayout({
    super.key,
    required this.items,
    this.selectedItem,
    required this.onItemSelected,
  });

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;
    final isExpanded = width >= 840;

    if (isExpanded) {
      return Row(
        children: [
          SizedBox(
            width: 360,
            child: MasterList(
              items: items,
              selectedItem: selectedItem,
              onItemSelected: onItemSelected,
            ),
          ),
          const VerticalDivider(width: 1),
          Expanded(
            child: selectedItem != null
                ? DetailView(item: selectedItem!)
                : const Center(child: Text('Selecione um item')),
          ),
        ],
      );
    }

    return MasterList(
      items: items,
      selectedItem: selectedItem,
      onItemSelected: (item) {
        onItemSelected(item);
        Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => DetailView(item: item)),
        );
      },
    );
  }
}
```

**Layout adaptativo em React Native:**

```tsx
import { useWindowDimensions } from 'react-native';

type WindowClass = 'compact' | 'medium' | 'expanded';

function useWindowClass(): WindowClass {
  const { width } = useWindowDimensions();
  if (width >= 840) return 'expanded';
  if (width >= 600) return 'medium';
  return 'compact';
}

function AdaptiveLayout({ master, detail }: { master: ReactNode; detail: ReactNode }) {
  const windowClass = useWindowClass();

  if (windowClass === 'expanded') {
    return (
      <View style={{ flex: 1, flexDirection: 'row' }}>
        <View style={{ width: 360 }}>{master}</View>
        <View style={{ flex: 1 }}>{detail}</View>
      </View>
    );
  }

  return <View style={{ flex: 1 }}>{master}</View>;
}
```

---

### Acessibilidade em Apps Nativos

**Flutter — Semantics:**

```dart
Semantics(
  label: 'Botão favoritar produto',
  hint: 'Toque duas vezes para adicionar aos favoritos',
  button: true,
  child: IconButton(
    icon: Icon(isFavorite ? Icons.favorite : Icons.favorite_border),
    onPressed: toggleFavorite,
    tooltip: isFavorite ? 'Remover dos favoritos' : 'Adicionar aos favoritos',
  ),
);

// Excluir elemento decorativo da árvore de acessibilidade
Semantics(
  excludeSemantics: true,
  child: Image.asset('assets/decorative-wave.png'),
);

// Agrupar elementos para leitura coerente
MergeSemantics(
  child: Row(
    children: [
      Icon(Icons.star, color: Colors.amber),
      const SizedBox(width: 4),
      const Text('4.8'),
      const SizedBox(width: 4),
      const Text('(127 avaliações)'),
    ],
  ),
);
```

**React Native — accessibilityLabel:**

```tsx
import { TouchableOpacity, Text, Image } from 'react-native';

// Botão acessível
<TouchableOpacity
  accessibilityRole="button"
  accessibilityLabel="Adicionar produto ao carrinho"
  accessibilityHint="Toque duas vezes para adicionar"
  onPress={addToCart}
>
  <Text>Comprar</Text>
</TouchableOpacity>

// Imagem decorativa (ignorada por leitores de tela)
<Image
  source={require('./decorative.png')}
  accessibilityElementsHidden={true}
  importantForAccessibility="no-hide-descendants"
/>

// Live region para atualizações dinâmicas
<Text
  accessibilityRole="alert"
  accessibilityLiveRegion="polite"
>
  {itemCount} itens no carrinho
</Text>
```

**Testando acessibilidade:**

| Ferramenta | Plataforma | Como testar |
|---|---|---|
| **TalkBack** | Android | Configurações → Acessibilidade → TalkBack |
| **VoiceOver** | iOS | Configurações → Acessibilidade → VoiceOver |
| **Accessibility Scanner** | Android | App do Google Play |
| **Accessibility Inspector** | iOS/macOS | Xcode → Open Developer Tool |
| **Flutter Semantics Debugger** | Flutter | `showSemanticsDebugger: true` no `MaterialApp` |

---

### Micro-interações e Animações

Micro-interações — pequenas animações em resposta a ações do usuário — elevam significativamente a qualidade percebida de um app.

**Framer Motion (React):**

```bash
npm install framer-motion
```

```tsx
import { motion, AnimatePresence } from 'framer-motion';

// Animação de entrada
function FadeInCard({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3, ease: 'easeOut' }}
      className="card"
    >
      {children}
    </motion.div>
  );
}

// Lista animada (itens entram/saem com animação)
function AnimatedList({ items }: { items: { id: string; text: string }[] }) {
  return (
    <ul>
      <AnimatePresence>
        {items.map(item => (
          <motion.li
            key={item.id}
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: 'auto' }}
            exit={{ opacity: 0, height: 0 }}
            transition={{ duration: 0.2 }}
          >
            {item.text}
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  );
}

// Layout animation (reordenação suave)
function ReorderableList({ items }: { items: string[] }) {
  return (
    <ul>
      {items.map(item => (
        <motion.li key={item} layout transition={{ type: 'spring', stiffness: 300, damping: 30 }}>
          {item}
        </motion.li>
      ))}
    </ul>
  );
}

// Botão com feedback de escala
function AnimatedButton({ children, ...props }: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  return (
    <motion.button
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
      transition={{ type: 'spring', stiffness: 400, damping: 17 }}
      {...props}
    >
      {children}
    </motion.button>
  );
}
```

**Animações em Flutter — implícitas e explícitas:**

```dart
// Implícita — AnimatedContainer (troca de valores com animação automática)
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: isExpanded ? 300 : 150,
  height: isExpanded ? 200 : 100,
  decoration: BoxDecoration(
    color: isActive ? Colors.blue : Colors.grey,
    borderRadius: BorderRadius.circular(isExpanded ? 16 : 8),
  ),
  child: const Center(child: Text('Conteúdo')),
);

// Implícita — AnimatedSwitcher (troca de widgets com crossfade)
AnimatedSwitcher(
  duration: const Duration(milliseconds: 200),
  child: isLoading
      ? const CircularProgressIndicator(key: ValueKey('loading'))
      : Text(data, key: ValueKey('data')),
);

// Explícita — AnimationController para controle total
class PulseAnimation extends StatefulWidget {
  const PulseAnimation({super.key});

  @override
  State<PulseAnimation> createState() => _PulseAnimationState();
}

class _PulseAnimationState extends State<PulseAnimation>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scale;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1000),
    )..repeat(reverse: true);

    _scale = Tween<double>(begin: 1.0, end: 1.1).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ScaleTransition(
      scale: _scale,
      child: const Icon(Icons.favorite, color: Colors.red, size: 48),
    );
  }
}
```

**Lottie — animações vetoriais leves (alternativa a GIFs):**

Lottie renderiza animações exportadas do After Effects (formato JSON) em qualquer plataforma, com qualidade vetorial e tamanho reduzido.

```bash
# React
npm install lottie-react

# Flutter
flutter pub add lottie

# React Native
npx expo install lottie-react-native
```

```tsx
// React
import Lottie from 'lottie-react';
import successAnimation from './animations/success.json';

function SuccessIndicator() {
  return (
    <Lottie
      animationData={successAnimation}
      loop={false}
      style={{ width: 120, height: 120 }}
      aria-label="Operação concluída com sucesso"
      role="img"
    />
  );
}
```

```dart
// Flutter
import 'package:lottie/lottie.dart';

Lottie.asset(
  'assets/animations/success.json',
  width: 120,
  height: 120,
  repeat: false,
  semanticsLabel: 'Operação concluída com sucesso',
);
```

**Recursos para animações Lottie gratuitas:** [LottieFiles](https://lottiefiles.com/) — biblioteca com milhares de animações prontas.

**Quando usar cada abordagem:**

| Tipo | Ferramenta | Quando |
|---|---|---|
| Transições CSS simples | CSS `transition` | Hover, foco, mudança de cor |
| Animações CSS | `@keyframes` | Spinners, shimmer, pulso |
| Animações declarativas | Framer Motion (React) | Entrada/saída, layout, gestos |
| Animações implícitas | Flutter `Animated*` | Mudanças de propriedade |
| Animações complexas | AnimationController / Rive | Sequências, interações ricas |
| Ilustrações animadas | Lottie | Onboarding, sucesso, loading customizado |

---

### RTL e Internacionalização Visual

Suporte a idiomas RTL (right-to-left) como árabe e hebraico exige espelhamento de todo o layout. CSS Logical Properties simplificam esse suporte.

**CSS Logical Properties (recomendado):**

Em vez de propriedades físicas (`left`, `right`, `margin-left`), usar propriedades lógicas que se adaptam automaticamente à direção do texto:

| Propriedade física | Propriedade lógica | Comportamento em RTL |
|---|---|---|
| `margin-left` | `margin-inline-start` | `margin-right` |
| `margin-right` | `margin-inline-end` | `margin-left` |
| `padding-left` | `padding-inline-start` | `padding-right` |
| `padding-right` | `padding-inline-end` | `padding-left` |
| `text-align: left` | `text-align: start` | `text-align: right` |
| `float: left` | `float: inline-start` | `float: right` |
| `border-left` | `border-inline-start` | `border-right` |
| `left: 0` | `inset-inline-start: 0` | `right: 0` |
| `width` | `inline-size` | Sem mudança |
| `height` | `block-size` | Sem mudança |

```css
/* ❌ Não funciona em RTL sem override */
.sidebar {
  margin-left: 1rem;
  padding-right: 2rem;
  border-left: 2px solid var(--color-border);
}

/* ✅ Funciona em qualquer direção */
.sidebar {
  margin-inline-start: 1rem;
  padding-inline-end: 2rem;
  border-inline-start: 2px solid var(--color-border);
}
```

**Ativando RTL no HTML:**

```html
<html lang="ar" dir="rtl">
```

**Tailwind CSS — suporte RTL:**

```html
<!-- Tailwind usa ltr:/rtl: modifiers -->
<div class="ltr:ml-4 rtl:mr-4">Conteúdo</div>

<!-- Ou com logical properties (v4) -->
<div class="ms-4 ps-6">Conteúdo com propriedades lógicas</div>
```

| Classe Tailwind | CSS gerado |
|---|---|
| `ms-4` | `margin-inline-start: 1rem` |
| `me-4` | `margin-inline-end: 1rem` |
| `ps-4` | `padding-inline-start: 1rem` |
| `pe-4` | `padding-inline-end: 1rem` |
| `text-start` | `text-align: start` |
| `text-end` | `text-align: end` |

**Flutter — Directionality:**

```dart
// Forçar RTL para teste
Directionality(
  textDirection: TextDirection.rtl,
  child: MyApp(),
);

// Usar EdgeInsetsDirectional em vez de EdgeInsets
Padding(
  padding: const EdgeInsetsDirectional.only(start: 16, end: 8),
  child: Text('Conteúdo'),
);

// AlignmentDirectional em vez de Alignment
Align(
  alignment: AlignmentDirectional.centerStart,
  child: Text('Alinhado ao início'),
);
```

**Pluralização e formatação por locale:**

```typescript
// Formatação de números
const formatter = new Intl.NumberFormat('pt-BR', {
  style: 'currency',
  currency: 'BRL',
});
formatter.format(1234.56); // "R$ 1.234,56"

// Formatação de datas
const dateFormatter = new Intl.DateTimeFormat('pt-BR', {
  dateStyle: 'long',
  timeStyle: 'short',
});
dateFormatter.format(new Date()); // "27 de junho de 2026 14:30"

// Pluralização
const pluralRules = new Intl.PluralRules('pt-BR');
function pluralize(count: number, forms: { one: string; other: string }) {
  const rule = pluralRules.select(count);
  return `${count} ${forms[rule as keyof typeof forms] ?? forms.other}`;
}
pluralize(1, { one: 'item', other: 'itens' }); // "1 item"
pluralize(5, { one: 'item', other: 'itens' }); // "5 itens"

// Listas
const listFormatter = new Intl.ListFormat('pt-BR', { style: 'long', type: 'conjunction' });
listFormatter.format(['React', 'Vue', 'Angular']); // "React, Vue e Angular"
```

**Impacto no layout — strings variam por idioma:**

| Idioma | "Configurações" | Expansão |
|---|---|---|
| Português | Configurações | Base |
| Inglês | Settings | -40% |
| Alemão | Einstellungen | +10% |
| Árabe | إعدادات | -30% (RTL) |

Botões e containers devem usar `min-width` e permitir crescimento para acomodar traduções mais longas.

---

## 6. Progressive Web Apps (PWA)

Progressive Web Apps combinam o alcance da web com recursos nativos — instalação, funcionamento offline, push notifications e acesso a APIs do dispositivo. Esta seção foca em padrões avançados e integração com frameworks.

> Para fundamentos de Service Workers, Cache API e PWA básico em JavaScript puro, consulte [Dicas-Javascript-Typescript.md](Dicas-Javascript-Typescript.md).
> Para uma introdução breve, consulte [Topicos-Complementares-Desenvolvimento.md](outros/Topicos-Complementares-Desenvolvimento.md).

---

### PWA com Workbox

Workbox é a biblioteca do Google que simplifica o gerenciamento de Service Workers e estratégias de cache.

```bash
npm install workbox-precaching workbox-routing workbox-strategies workbox-expiration
```

**Estratégias de cache:**

| Estratégia | Descrição | Uso |
|---|---|---|
| **CacheFirst** | Cache primeiro, rede como fallback | Assets estáticos (imagens, fontes, CSS) |
| **NetworkFirst** | Rede primeiro, cache como fallback | APIs com dados dinâmicos |
| **StaleWhileRevalidate** | Cache imediato + atualiza em background | Assets que mudam ocasionalmente |
| **NetworkOnly** | Sempre rede, sem cache | Login, pagamentos |
| **CacheOnly** | Sempre cache, sem rede | Assets pré-cacheados |

**Service Worker com Workbox (manual — `injectManifest`):**

```typescript
// sw.ts
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { CacheFirst, NetworkFirst, StaleWhileRevalidate } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';

// Pré-cache de assets gerados no build
precacheAndRoute(self.__WB_MANIFEST);

// Imagens — CacheFirst com expiração de 30 dias
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 30 * 24 * 60 * 60,
      }),
    ],
  })
);

// API — NetworkFirst com fallback de cache
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 5 * 60,
      }),
    ],
  })
);

// Google Fonts — StaleWhileRevalidate
registerRoute(
  ({ url }) => url.origin === 'https://fonts.googleapis.com' ||
               url.origin === 'https://fonts.gstatic.com',
  new StaleWhileRevalidate({
    cacheName: 'google-fonts',
  })
);
```

**`generateSW` vs `injectManifest`:**

| Aspecto | generateSW | injectManifest |
|---|---|---|
| **Controle** | Baixo (configuração declarativa) | Total (código JS) |
| **Push Notifications** | Não | Sim |
| **Background Sync** | Não | Sim |
| **Custom routes** | Limitado | Total |
| **Ideal para** | PWAs simples | PWAs com lógica avançada |

---

### Push Notifications

**Fluxo de Push Notifications:**

```
1. App solicita permissão ao usuário
2. Browser gera PushSubscription com endpoint + keys
3. App envia subscription para o backend
4. Backend envia mensagem via Web Push Protocol (VAPID)
5. Service Worker recebe o evento 'push'
6. Service Worker exibe notificação
```

**Frontend — solicitar permissão e obter subscription:**

```typescript
async function subscribeToPush(): Promise<PushSubscription | null> {
  const permission = await Notification.requestPermission();
  if (permission !== 'granted') return null;

  const registration = await navigator.serviceWorker.ready;

  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
  });

  // Enviar subscription para o backend
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription),
  });

  return subscription;
}

function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding).replace(/-/g, '+').replace(/_/g, '/');
  const rawData = window.atob(base64);
  return Uint8Array.from(rawData, (char) => char.charCodeAt(0));
}
```

**Service Worker — receber e exibir notificação:**

```typescript
// sw.ts
self.addEventListener('push', (event: PushEvent) => {
  const data = event.data?.json() ?? {};

  const options: NotificationOptions = {
    body: data.body ?? 'Nova notificação',
    icon: '/icons/icon-192.png',
    badge: '/icons/badge-72.png',
    vibrate: [100, 50, 100],
    data: { url: data.url ?? '/' },
    actions: [
      { action: 'open', title: 'Abrir' },
      { action: 'dismiss', title: 'Dispensar' },
    ],
  };

  event.waitUntil(
    self.registration.showNotification(data.title ?? 'Notificação', options)
  );
});

self.addEventListener('notificationclick', (event: NotificationEvent) => {
  event.notification.close();

  if (event.action === 'dismiss') return;

  const url = event.notification.data?.url ?? '/';
  event.waitUntil(
    self.clients.matchAll({ type: 'window' }).then(clients => {
      const existing = clients.find(c => c.url === url);
      if (existing) return existing.focus();
      return self.clients.openWindow(url);
    })
  );
});
```

**Boas práticas para permissão de notificações:**

- Não pedir permissão no primeiro acesso — esperar uma ação do usuário
- Mostrar um banner customizado antes do prompt nativo, explicando o valor
- Oferecer opção de recusar sem bloquear o prompt nativo
- Permitir desativar nas configurações do app

---

### Background Sync

Background Sync permite enfileirar requisições que falharam (offline) para reenvio automático quando a conexão retornar.

```bash
npm install workbox-background-sync
```

```typescript
// sw.ts
import { BackgroundSyncPlugin } from 'workbox-background-sync';
import { NetworkOnly } from 'workbox-strategies';
import { registerRoute } from 'workbox-routing';

const bgSyncPlugin = new BackgroundSyncPlugin('form-submissions', {
  maxRetentionTime: 24 * 60, // 24 horas em minutos
  onSync: async ({ queue }) => {
    let entry;
    while ((entry = await queue.shiftRequest())) {
      try {
        await fetch(entry.request);
      } catch (error) {
        await queue.unshiftRequest(entry);
        throw error;
      }
    }
  },
});

registerRoute(
  ({ url }) => url.pathname.startsWith('/api/forms'),
  new NetworkOnly({ plugins: [bgSyncPlugin] }),
  'POST'
);
```

---

### APIs Web Modernas para PWA

**Periodic Background Sync — sincronização periódica:**

Permite que o Service Worker execute tarefas periodicamente em background, mesmo quando o app não está aberto.

```typescript
// Registrar sync periódico (requer permissão do browser)
async function registerPeriodicSync() {
  const registration = await navigator.serviceWorker.ready;

  if ('periodicSync' in registration) {
    const status = await navigator.permissions.query({ name: 'periodic-background-sync' as any });
    if (status.state === 'granted') {
      await registration.periodicSync.register('update-content', {
        minInterval: 12 * 60 * 60 * 1000, // 12 horas
      });
    }
  }
}

// sw.ts — tratar no Service Worker
self.addEventListener('periodicsync', (event: any) => {
  if (event.tag === 'update-content') {
    event.waitUntil(fetchAndCacheLatestContent());
  }
});

async function fetchAndCacheLatestContent() {
  const cache = await caches.open('content-cache');
  await cache.add('/api/articles/latest');
}
```

**Web Share API — compartilhar como app nativo:**

```typescript
async function shareContent(title: string, text: string, url: string) {
  if (navigator.share) {
    await navigator.share({ title, text, url });
  } else {
    await navigator.clipboard.writeText(url);
    alert('Link copiado para a área de transferência');
  }
}
```

**Web Share Target — receber compartilhamentos:**

```json
// manifest.json
{
  "share_target": {
    "action": "/share",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [{ "name": "media", "accept": ["image/*", "video/*"] }]
    }
  }
}
```

**Badging API — badge numérico no ícone do app:**

```typescript
// Definir badge (ex: contagem de notificações)
if ('setAppBadge' in navigator) {
  navigator.setAppBadge(5); // Mostra "5" no ícone
}

// Limpar badge
navigator.clearAppBadge();
```

---

### Install Prompt Customizado

```tsx
import { useState, useEffect } from 'react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

export function InstallBanner() {
  const [deferredPrompt, setDeferredPrompt] =
    useState<BeforeInstallPromptEvent | null>(null);
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e as BeforeInstallPromptEvent);
      setShowBanner(true);
    };
    window.addEventListener('beforeinstallprompt', handler);
    return () => window.removeEventListener('beforeinstallprompt', handler);
  }, []);

  async function handleInstall() {
    if (!deferredPrompt) return;
    await deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    if (outcome === 'accepted') {
      setShowBanner(false);
    }
    setDeferredPrompt(null);
  }

  if (!showBanner) return null;

  return (
    <div role="banner" className="install-banner">
      <p>Instale nosso app para acesso rápido e offline!</p>
      <button onClick={handleInstall}>Instalar</button>
      <button onClick={() => setShowBanner(false)} aria-label="Fechar">
        &times;
      </button>
    </div>
  );
}
```

---

### PWA com React (Vite PWA)

```bash
npm install vite-plugin-pwa -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'prompt',
      includeAssets: ['favicon.ico', 'robots.txt', 'icons/*.png'],
      manifest: {
        name: 'Meu App PWA',
        short_name: 'MeuApp',
        description: 'Aplicação PWA com React e Vite',
        theme_color: '#2563eb',
        background_color: '#ffffff',
        display: 'standalone',
        start_url: '/',
        icons: [
          { src: '/icons/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icons/icon-512.png', sizes: '512x512', type: 'image/png' },
          { src: '/icons/icon-512.png', sizes: '512x512', type: 'image/png', purpose: 'maskable' },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\/.*/i,
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

**Componente de atualização (prompt):**

```tsx
import { useRegisterSW } from 'virtual:pwa-register/react';

export function PWAUpdatePrompt() {
  const {
    needRefresh: [needRefresh],
    updateServiceWorker,
  } = useRegisterSW();

  if (!needRefresh) return null;

  return (
    <div role="alert" className="pwa-update-toast">
      <p>Nova versão disponível!</p>
      <button onClick={() => updateServiceWorker(true)}>
        Atualizar agora
      </button>
    </div>
  );
}
```

---

### PWA com Angular

```bash
ng add @angular/pwa
```

Esse comando gera automaticamente: `ngsw-config.json`, `manifest.webmanifest`, ícones e registra o Service Worker.

**Configuração do ngsw-config.json:**

```json
{
  "$schema": "./node_modules/@angular/service-worker/config/schema.json",
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": {
        "files": ["/assets/**", "/*.(png|jpg|svg|webp)"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api",
      "urls": ["/api/**"],
      "cacheConfig": {
        "strategy": "freshness",
        "maxSize": 50,
        "maxAge": "5m",
        "timeout": "3s"
      }
    }
  ]
}
```

**Detectar atualizações do Service Worker:**

```typescript
import { Component, inject } from '@angular/core';
import { SwUpdate, VersionReadyEvent } from '@angular/service-worker';
import { filter } from 'rxjs';

@Component({
  selector: 'app-root',
  template: `
    @if (showUpdateBanner) {
      <div class="update-banner" role="alert">
        <p>Nova versão disponível!</p>
        <button (click)="updateApp()">Atualizar</button>
      </div>
    }
    <router-outlet />
  `,
})
export class AppComponent {
  private swUpdate = inject(SwUpdate);
  showUpdateBanner = false;

  constructor() {
    if (this.swUpdate.isEnabled) {
      this.swUpdate.versionUpdates
        .pipe(filter((e): e is VersionReadyEvent => e.type === 'VERSION_READY'))
        .subscribe(() => {
          this.showUpdateBanner = true;
        });
    }
  }

  updateApp() {
    this.swUpdate.activateUpdate().then(() => document.location.reload());
  }
}
```

**Push Notifications com Angular:**

```typescript
import { SwPush } from '@angular/service-worker';

@Component({ /* ... */ })
export class NotificationComponent {
  private swPush = inject(SwPush);

  async subscribe() {
    const sub = await this.swPush.requestSubscription({
      serverPublicKey: VAPID_PUBLIC_KEY,
    });

    await fetch('/api/push/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(sub),
    });
  }

  constructor() {
    this.swPush.messages.subscribe((msg) => {
      console.log('Push message received:', msg);
    });
  }
}
```

---

### PWA com Vue (Vite PWA)

A configuração é idêntica ao React (mesmo plugin `vite-plugin-pwa`), com integração Vue-specific:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    vue(),
    VitePWA({
      registerType: 'prompt',
      manifest: {
        name: 'Meu App Vue PWA',
        short_name: 'VueApp',
        theme_color: '#42b883',
        display: 'standalone',
        icons: [
          { src: '/icons/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icons/icon-512.png', sizes: '512x512', type: 'image/png' },
        ],
      },
    }),
  ],
});
```

**Composable para atualizações:**

```html
<!-- UpdatePrompt.vue -->
<script setup lang="ts">
import { useRegisterSW } from 'virtual:pwa-register/vue';

const {
  needRefresh,
  updateServiceWorker,
} = useRegisterSW();
</script>

<template>
  <div v-if="needRefresh" role="alert" class="update-toast">
    <p>Nova versão disponível!</p>
    <button @click="updateServiceWorker(true)">Atualizar</button>
  </div>
</template>
```

---

### Estratégias de Atualização do Service Worker

| Estratégia | Comportamento | Quando usar |
|---|---|---|
| **Auto update** | Atualiza silenciosamente em background | Apps sem dados em trânsito |
| **Prompt** | Mostra banner pedindo atualização | Apps com formulários/dados não salvos |
| **Skip waiting** | Ativa imediatamente (pode causar inconsistência) | Hotfixes críticos |
| **Page reload** | Recarrega a página após ativação | Garantia de consistência |

**Evitar "preso na versão antiga":**

```typescript
// Verificar atualizações periodicamente
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.ready.then(registration => {
    setInterval(() => {
      registration.update();
    }, 60 * 60 * 1000); // A cada hora
  });
}
```

---

### Core Web Vitals

Core Web Vitals são as métricas do Google para medir a experiência real do usuário. Impactam diretamente o ranking de busca (SEO) e a percepção de qualidade.

| Métrica | Nome | Bom | Precisa melhorar | Ruim | O que mede |
|---|---|---|---|---|---|
| **LCP** | Largest Contentful Paint | ≤ 2.5s | 2.5s – 4.0s | > 4.0s | Tempo até o maior elemento visível carregar |
| **INP** | Interaction to Next Paint | ≤ 200ms | 200ms – 500ms | > 500ms | Latência de resposta a interações do usuário |
| **CLS** | Cumulative Layout Shift | ≤ 0.1 | 0.1 – 0.25 | > 0.25 | Instabilidade visual (elementos "pulando") |

**Otimizando LCP:**

```html
<!-- Preload do recurso LCP (hero image, web font) -->
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high" />
<link rel="preload" as="font" href="/fonts/inter.woff2" type="font/woff2" crossorigin />

<!-- Priorizar imagem hero -->
<img src="/hero.webp" alt="Banner principal" fetchpriority="high"
     width="1200" height="600" />
```

**Otimizando INP:**

```typescript
// Evitar long tasks — dividir trabalho pesado
function processLargeList(items: Item[]) {
  const CHUNK_SIZE = 50;
  let index = 0;

  function processChunk() {
    const end = Math.min(index + CHUNK_SIZE, items.length);
    for (let i = index; i < end; i++) {
      processItem(items[i]);
    }
    index = end;
    if (index < items.length) {
      requestIdleCallback(processChunk);
    }
  }

  requestIdleCallback(processChunk);
}

// Usar startTransition para não bloquear interações (React 18+)
import { startTransition, useState } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<Item[]>([]);

  function handleChange(value: string) {
    setQuery(value); // Atualização urgente (input)
    startTransition(() => {
      setResults(filterItems(value)); // Atualização não-urgente
    });
  }

  return (
    <>
      <input value={query} onChange={e => handleChange(e.target.value)} />
      <ResultList items={results} />
    </>
  );
}
```

**Otimizando CLS:**

```css
/* Sempre definir dimensões em imagens e vídeos */
img, video {
  max-width: 100%;
  height: auto;
}

/* Reservar espaço com aspect-ratio */
.hero-image {
  aspect-ratio: 16 / 9;
  width: 100%;
  object-fit: cover;
}

/* Evitar injeção de conteúdo acima do viewport */
.banner-topo {
  min-height: 48px; /* Reservar espaço mesmo antes do conteúdo carregar */
}
```

**Medir Core Web Vitals:**

```typescript
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(metric => console.log('LCP:', metric.value));
onINP(metric => console.log('INP:', metric.value));
onCLS(metric => console.log('CLS:', metric.value));
```

---

### Performance — Lazy Loading e Otimização de Imagens

**Lazy loading de imagens (nativo):**

```html
<!-- Imagens abaixo do fold — carregam quando visíveis -->
<img src="produto.webp" alt="Produto" loading="lazy" width="400" height="300" />

<!-- Imagem hero — NÃO usar lazy (prejudica LCP) -->
<img src="hero.webp" alt="Banner" fetchpriority="high" width="1200" height="600" />
```

**Lazy loading de componentes (React):**

```tsx
import { lazy, Suspense } from 'react';

const AdminDashboard = lazy(() => import('./pages/AdminDashboard'));
const Charts = lazy(() => import('./components/Charts'));

function App() {
  return (
    <Suspense fallback={<div aria-busy="true">Carregando...</div>}>
      <Routes>
        <Route path="/admin" element={<AdminDashboard />} />
      </Routes>
    </Suspense>
  );
}
```

**Lazy loading em Angular:**

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent),
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes').then(m => m.REPORT_ROUTES),
  },
];
```

**Lazy loading em Vue:**

```typescript
const routes = [
  {
    path: '/admin',
    component: () => import('./views/AdminDashboard.vue'),
  },
];
```

**Formatos modernos de imagem:**

| Formato | Compressão | Transparência | Animação | Suporte |
|---|---|---|---|---|
| **JPEG** | Lossy | Não | Não | Universal |
| **PNG** | Lossless | Sim | Não | Universal |
| **WebP** | Lossy/Lossless | Sim | Sim | 97%+ browsers |
| **AVIF** | Lossy/Lossless | Sim | Sim | 92%+ browsers |
| **SVG** | Vetorial | Sim | Sim | Universal |

**Imagens responsivas com `<picture>` e `srcset`:**

```html
<!-- Formato moderno com fallback -->
<picture>
  <source srcset="produto.avif" type="image/avif" />
  <source srcset="produto.webp" type="image/webp" />
  <img src="produto.jpg" alt="Produto" width="800" height="600" loading="lazy" />
</picture>

<!-- Tamanhos diferentes por viewport -->
<img
  srcset="produto-400.webp 400w,
          produto-800.webp 800w,
          produto-1200.webp 1200w"
  sizes="(max-width: 640px) 100vw,
         (max-width: 1024px) 50vw,
         33vw"
  src="produto-800.webp"
  alt="Produto"
  loading="lazy"
  width="800"
  height="600"
/>
```

**Componente de imagem otimizada (React):**

```tsx
interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  priority?: boolean;
  className?: string;
}

function OptimizedImage({ src, alt, width, height, priority, className }: OptimizedImageProps) {
  const baseName = src.replace(/\.[^.]+$/, '');

  return (
    <picture>
      <source srcSet={`${baseName}.avif`} type="image/avif" />
      <source srcSet={`${baseName}.webp`} type="image/webp" />
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        loading={priority ? 'eager' : 'lazy'}
        fetchPriority={priority ? 'high' : undefined}
        decoding={priority ? 'sync' : 'async'}
        className={className}
      />
    </picture>
  );
}
```

---

### Lighthouse e Auditoria de PWA

**Critérios para PWA no Lighthouse:**

| Critério | Requisito |
|---|---|
| **Instalável** | Manifest com name, icons, start_url, display |
| **Service Worker** | Registrado e funcional |
| **HTTPS** | Obrigatório (exceto localhost) |
| **Offline** | Responde com status 200 quando offline |
| **Redirect HTTP→HTTPS** | Redireciona todo tráfego para HTTPS |
| **Viewport** | Meta tag viewport configurada |
| **Apple touch icon** | Ícone para iOS |
| **Maskable icon** | Ícone adaptável |

**Lighthouse CI no GitHub Actions:**

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: [push, pull_request]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci && npm run build

      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          uploadArtifacts: true
          configPath: .lighthouserc.json
```

```json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:pwa": ["warn", { "minScore": 1 }]
      }
    },
    "collect": {
      "staticDistDir": "./dist"
    }
  }
}
```

---

### PWA vs App Nativo — Quando Usar Cada

| Critério | PWA | App Nativo (Flutter/RN) |
|---|---|---|
| **Distribuição** | URL (sem loja) | App Store / Play Store |
| **Instalação** | Opcional, leve | Obrigatória, pesada |
| **Atualizações** | Instantâneas | Review da loja (1-7 dias) |
| **Acesso offline** | Service Worker (limitado) | Total |
| **Push Notifications** | Web Push (exceto iOS Safari limitado) | Nativo (completo) |
| **Câmera/GPS** | Sim (APIs web) | Sim (APIs nativas) |
| **Bluetooth/NFC** | Limitado (Web Bluetooth) | Completo |
| **Biometria** | Web Authentication API | Nativo |
| **Performance** | Boa (JS engine) | Excelente (nativo/compilado) |
| **Custo de dev** | Baixo (1 codebase web) | Médio-alto |
| **SEO** | Indexável | Não indexável |
| **Quando usar** | MVP, conteúdo, e-commerce, ferramentas internas | Jogos, apps com hardware, UX premium |

---

## 7. Acessibilidade Avançada (WCAG 2.2)

Esta seção aprofunda os conceitos de acessibilidade web além do básico, cobrindo os novos critérios do WCAG 2.2, padrões ARIA avançados e implementação de componentes acessíveis.

> Para fundamentos de acessibilidade (princípios POUR, ARIA básico, HTML semântico), consulte [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md).
> Para ferramentas de teste automatizado (vitest-axe, cypress-axe, playwright), consulte [Testes-Frontend-JS-React.md](outros/Testes-Frontend-JS-React.md).

> Referência: [WCAG 2.2](https://www.w3.org/TR/WCAG22/) · [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) · [WebAIM](https://webaim.org/)

---

### WCAG 2.2 — Novos Critérios de Sucesso

O WCAG 2.2 adicionou 9 novos critérios de sucesso, focados em usabilidade mobile, cognitiva e motora:

| Critério | Nível | Descrição |
|---|---|---|
| **2.4.11 Focus Not Obscured (Minimum)** | AA | O elemento em foco não pode estar totalmente oculto por outros elementos (sticky headers, modais) |
| **2.4.12 Focus Not Obscured (Enhanced)** | AAA | O elemento em foco não pode estar parcialmente oculto |
| **2.4.13 Focus Appearance** | AAA | O indicador de foco deve ter contraste e área suficientes |
| **2.5.7 Dragging Movements** | AA | Toda funcionalidade de arrastar deve ter alternativa sem arraste (click/tap) |
| **2.5.8 Target Size (Minimum)** | AA | Áreas de toque devem ter no mínimo 24x24px (com exceções) |
| **3.2.6 Consistent Help** | A | Mecanismos de ajuda devem estar na mesma posição relativa em todas as páginas |
| **3.3.7 Redundant Entry** | A | Não exigir que o usuário redigite informações já fornecidas na mesma sessão |
| **3.3.8 Accessible Authentication (Minimum)** | AA | Login não pode depender de testes cognitivos (CAPTCHA) sem alternativa |
| **3.3.9 Accessible Authentication (Enhanced)** | AAA | Login não pode depender de reconhecimento de objetos ou conteúdo pessoal |

**Exemplos de implementação:**

```css
/* 2.4.11 — Focus Not Obscured: garantir scroll-margin para sticky headers */
:target,
:focus {
  scroll-margin-top: 80px; /* Altura do header sticky */
}

/* 2.5.8 — Target Size Minimum: botões com área mínima */
button, a, [role="button"] {
  min-height: 44px;
  min-width: 44px;
}

/* Ícones pequenos com área de toque expandida */
.icon-button {
  position: relative;
  width: 24px;
  height: 24px;
}
.icon-button::before {
  content: '';
  position: absolute;
  top: -10px;
  left: -10px;
  right: -10px;
  bottom: -10px;
}
```

```html
<!-- 2.5.7 — Dragging Movements: alternativa para reordenar lista -->
<ul role="listbox" aria-label="Itens reordenáveis">
  <li role="option" draggable="true">
    <span>Item 1</span>
    <!-- Alternativa sem arraste -->
    <button aria-label="Mover Item 1 para cima">↑</button>
    <button aria-label="Mover Item 1 para baixo">↓</button>
  </li>
</ul>

<!-- 3.3.7 — Redundant Entry: auto-preenchimento -->
<label for="shipping-city">Cidade de entrega</label>
<input id="shipping-city" autocomplete="shipping locality" />

<!-- 3.3.8 — Accessible Authentication: suporte a password managers -->
<input type="email" autocomplete="username" />
<input type="password" autocomplete="current-password" />
<!-- Permitir colar senha (nunca bloquear paste em campos de senha) -->
```

---

### Níveis de Conformidade na Prática

| Nível | Requisitos | Esforço | Quando exigir |
|---|---|---|---|
| **A** | 32 critérios básicos | Baixo-médio | Mínimo aceitável para qualquer site |
| **AA** | A + 24 critérios adicionais | Médio | Padrão recomendado (obrigatório para governo/educação) |
| **AAA** | AA + 27 critérios extras | Alto | Aspiracional; raramente exigido integralmente |

**Requisitos legais no Brasil:**

- **Lei Brasileira de Inclusão (LBI — Lei 13.146/2015)**: Exige acessibilidade em sites e sistemas de empresas com sede no Brasil.
- **e-MAG (Modelo de Acessibilidade em Governo Eletrônico)**: Baseado no WCAG 2.0 AA, obrigatório para sites do governo federal.
- **Na prática**: O padrão **WCAG 2.2 AA** é o alvo recomendado para projetos comerciais e educacionais.

---

### Contraste de Cores — Ferramentas e APCA

O WCAG 2.2 exige razão de contraste mínima entre texto e fundo. A fórmula atual (WCAG 2.x) tem limitações, e o **APCA** (Accessible Perceptual Contrast Algorithm) é a evolução prevista para o WCAG 3.0.

**Requisitos de contraste WCAG 2.2:**

| Critério | Texto normal | Texto grande (≥24px ou ≥19px bold) |
|---|---|---|
| **AA** | ≥ 4.5:1 | ≥ 3:1 |
| **AAA** | ≥ 7:1 | ≥ 4.5:1 |
| **Componentes de UI e gráficos** | ≥ 3:1 | — |

**Ferramentas para verificar contraste:**

| Ferramenta | Tipo | Descrição |
|---|---|---|
| **WebAIM Contrast Checker** | Web | Verifica ratio WCAG 2.x entre duas cores |
| **Colour Contrast Analyser (CCA)** | Desktop (Windows/Mac) | Eyedropper para pegar cores da tela |
| **axe DevTools** | Extensão browser | Detecta violações de contraste automaticamente |
| **Polypane** | Browser dev | Simulação de daltonismo e contraste em tempo real |
| **Figma — A11y plugins** | Plugin Figma | Verifica contraste direto no design |
| **Chrome DevTools** | Browser | Inspect → mostra ratio ao selecionar texto |

**Verificar contraste via CSS (futuro — `color-contrast()`):**

```css
/* Proposta CSS — seleciona automaticamente a cor de melhor contraste */
.adaptive-text {
  color: color-contrast(var(--color-background) vs white, black);
}
```

**APCA — a evolução do cálculo de contraste:**

O APCA corrige limitações da fórmula WCAG 2.x (que pode aprovar combinações de baixa legibilidade e reprovar combinações legíveis). O APCA considera:

- Polaridade (texto claro em fundo escuro vs texto escuro em fundo claro)
- Tamanho e peso da fonte
- Percepção humana real de luminosidade

| Valor APCA (Lc) | Uso recomendado |
|---|---|
| ≥ 90 | Texto corpo (body text) |
| ≥ 75 | Texto de conteúdo geral |
| ≥ 60 | Texto grande, títulos |
| ≥ 45 | Texto muito grande, ícones |
| ≥ 30 | Elementos decorativos, bordas |
| < 30 | Não usar para texto |

> Referência: [APCA Contrast Calculator](https://www.myndex.com/APCA/)

**Função utilitária para calcular contraste WCAG 2.x:**

```typescript
function getContrastRatio(hex1: string, hex2: string): number {
  const lum1 = getRelativeLuminance(hex1);
  const lum2 = getRelativeLuminance(hex2);
  const lighter = Math.max(lum1, lum2);
  const darker = Math.min(lum1, lum2);
  return (lighter + 0.05) / (darker + 0.05);
}

function getRelativeLuminance(hex: string): number {
  const rgb = hexToRgb(hex).map(c => {
    const sRGB = c / 255;
    return sRGB <= 0.04045
      ? sRGB / 12.92
      : Math.pow((sRGB + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * rgb[0] + 0.7152 * rgb[1] + 0.0722 * rgb[2];
}

function hexToRgb(hex: string): number[] {
  const h = hex.replace('#', '');
  return [
    parseInt(h.substring(0, 2), 16),
    parseInt(h.substring(2, 4), 16),
    parseInt(h.substring(4, 6), 16),
  ];
}

// Uso
const ratio = getContrastRatio('#2563eb', '#ffffff'); // ~4.57:1
const passesAA = ratio >= 4.5;
const passesAAA = ratio >= 7;
```

---

### Padrões ARIA Avançados

**Live Regions — anunciar mudanças dinâmicas:**

```html
<!-- Mensagem que será lida automaticamente pelo leitor de tela -->
<div aria-live="polite" aria-atomic="true" class="sr-only" id="status-message">
  <!-- Atualizado via JavaScript -->
</div>

<!-- Erros urgentes (interrompe leitura atual) -->
<div role="alert" aria-live="assertive">
  Sessão expirada. Faça login novamente.
</div>
```

```typescript
// Anunciar mudanças de forma programática
function announce(message: string, priority: 'polite' | 'assertive' = 'polite') {
  const el = document.getElementById('status-message');
  if (!el) return;
  el.setAttribute('aria-live', priority);
  el.textContent = '';
  requestAnimationFrame(() => {
    el.textContent = message;
  });
}

// Uso
announce('3 resultados encontrados');
announce('Erro ao salvar. Verifique os campos.', 'assertive');
```

**`aria-activedescendant` — navegação sem mover o foco DOM:**

```html
<div role="listbox" tabindex="0" aria-activedescendant="option-2"
     aria-label="Selecionar país">
  <div role="option" id="option-1">Brasil</div>
  <div role="option" id="option-2" aria-selected="true">Portugal</div>
  <div role="option" id="option-3">Angola</div>
</div>
```

**Classe utilitária `sr-only` (visually hidden):**

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

---

### Popover API, dialog Nativo e CSS Anchor Positioning

APIs nativas do browser que substituem implementações JavaScript complexas, com acessibilidade embutida.

**`<dialog>` nativo — modal sem JavaScript de focus trap:**

O elemento `<dialog>` resolve automaticamente: focus trap, retorno de foco, Escape para fechar, `aria-modal`, e inert no conteúdo atrás. Elimina a necessidade de bibliotecas de modal.

```html
<button onclick="document.getElementById('myDialog').showModal()">
  Abrir modal
</button>

<dialog id="myDialog">
  <h2>Confirmar ação</h2>
  <p>Tem certeza que deseja prosseguir?</p>
  <form method="dialog">
    <button value="cancel">Cancelar</button>
    <button value="confirm">Confirmar</button>
  </form>
</dialog>
```

```css
dialog::backdrop {
  background: rgb(0 0 0 / 0.5);
  backdrop-filter: blur(4px);
}

dialog {
  border: none;
  border-radius: var(--radius-xl);
  padding: var(--spacing-lg);
  box-shadow: var(--shadow-xl);
  max-width: 480px;
  width: 90vw;
}

dialog[open] {
  animation: fade-in 0.2s ease-out;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(-8px); }
  to { opacity: 1; transform: translateY(0); }
}
```

```tsx
// React wrapper para <dialog>
import { useRef, useEffect } from 'react';

function NativeDialog({ open, onClose, children }: {
  open: boolean;
  onClose: () => void;
  children: React.ReactNode;
}) {
  const ref = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    const dialog = ref.current;
    if (!dialog) return;
    if (open) dialog.showModal();
    else dialog.close();
  }, [open]);

  return (
    <dialog ref={ref} onClose={onClose}>
      {children}
    </dialog>
  );
}
```

**Popover API — tooltips e dropdowns sem JavaScript:**

```html
<!-- Botão que controla o popover -->
<button popovertarget="menu">Menu ▾</button>

<div id="menu" popover>
  <ul>
    <li><a href="/perfil">Perfil</a></li>
    <li><a href="/config">Configurações</a></li>
    <li><a href="/sair">Sair</a></li>
  </ul>
</div>
```

```css
[popover] {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  padding: var(--spacing-sm);
  box-shadow: var(--shadow-lg);
  background: var(--color-surface);
}

/* Animação de entrada e saída */
[popover]:popover-open {
  animation: popover-show 0.15s ease-out;
}

@keyframes popover-show {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}
```

O Popover API gerencia automaticamente: fechar ao clicar fora, fechar com Escape, camada superior (`top layer`), e empilhamento correto.

**CSS Anchor Positioning — posicionar relativo a outro elemento:**

```css
/* Âncora (elemento de referência) */
.trigger {
  anchor-name: --my-trigger;
}

/* Popover posicionado em relação à âncora */
.tooltip {
  position-anchor: --my-trigger;
  position: fixed;
  top: anchor(bottom);
  left: anchor(center);
  translate: -50% 8px;
}
```

Quando usar `<dialog>` vs Popover vs biblioteca:

| Padrão | Solução recomendada |
|---|---|
| Modal de confirmação | `<dialog>` nativo |
| Dropdown menu | Popover API |
| Tooltip simples | Popover API + CSS Anchor |
| Modal complexo (formulário, multi-step) | `<dialog>` nativo ou Radix Dialog |
| Combobox / Autocomplete | Biblioteca (Radix, Headless UI) |

---

### Scroll-driven Animations e View Transitions API

**Scroll-driven Animations (CSS puro):**

Animações controladas pela posição do scroll, sem JavaScript. O browser otimiza na thread do compositor.

```css
/* Barra de progresso de leitura */
.reading-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: var(--color-primary);
  transform-origin: left;
  animation: grow-progress auto linear;
  animation-timeline: scroll();
}

@keyframes grow-progress {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

/* Fade-in ao entrar no viewport */
.fade-on-scroll {
  animation: fade-in auto linear;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Parallax com scroll-driven */
.parallax-bg {
  animation: parallax auto linear;
  animation-timeline: scroll();
}

@keyframes parallax {
  from { transform: translateY(0); }
  to { transform: translateY(-100px); }
}
```

**View Transitions API — animações entre estados/páginas:**

> Para a introdução à View Transitions API, consulte [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md).

A View Transitions API facilita transições animadas entre estados de uma SPA:

```typescript
// Transição ao mudar conteúdo
async function navigateTo(url: string) {
  if (!document.startViewTransition) {
    updateDOM(url);
    return;
  }

  const transition = document.startViewTransition(() => {
    updateDOM(url);
  });

  await transition.finished;
}
```

```css
/* Animação padrão (crossfade) */
::view-transition-old(root) {
  animation: fade-out 0.2s ease-out;
}
::view-transition-new(root) {
  animation: fade-in 0.2s ease-in;
}

/* Transição de elemento específico (ex: imagem de produto) */
.product-image {
  view-transition-name: product-hero;
}

::view-transition-group(product-hero) {
  animation-duration: 0.3s;
}
```

**View Transitions em frameworks:**

```tsx
// React Router — com startViewTransition
import { useNavigate } from 'react-router-dom';

function ProductCard({ id, image }: { id: string; image: string }) {
  const navigate = useNavigate();

  function handleClick() {
    if (document.startViewTransition) {
      document.startViewTransition(() => {
        navigate(`/produtos/${id}`);
      });
    } else {
      navigate(`/produtos/${id}`);
    }
  }

  return (
    <div onClick={handleClick}>
      <img src={image} alt="" style={{ viewTransitionName: `product-${id}` }} />
    </div>
  );
}
```

```typescript
// Angular — ativação nativa em router
// app.config.ts
import { provideRouter, withViewTransitions } from '@angular/router';

export const appConfig = {
  providers: [
    provideRouter(routes, withViewTransitions()),
  ],
};
```

---

### Componentes Acessíveis — Modal/Dialog

> Referência: [WAI-ARIA APG — Dialog Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

Requisitos:
- Focus trap — Tab/Shift+Tab ciclam dentro do modal
- Escape fecha o modal
- Ao fechar, o foco retorna ao elemento que abriu
- `aria-modal="true"` + `role="dialog"`
- `aria-labelledby` aponta para o título

```tsx
import { useEffect, useRef, type ReactNode } from 'react';

interface ModalProps {
  open: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

export function AccessibleModal({ open, onClose, title, children }: ModalProps) {
  const dialogRef = useRef<HTMLDivElement>(null);
  const triggerRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (open) {
      triggerRef.current = document.activeElement as HTMLElement;
      // Focar primeiro elemento focável dentro do modal
      const firstFocusable = dialogRef.current?.querySelector<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      firstFocusable?.focus();
    } else {
      triggerRef.current?.focus();
    }
  }, [open]);

  useEffect(() => {
    if (!open) return;

    function handleKeyDown(e: KeyboardEvent) {
      if (e.key === 'Escape') {
        onClose();
        return;
      }
      if (e.key !== 'Tab') return;

      const dialog = dialogRef.current;
      if (!dialog) return;

      const focusables = dialog.querySelectorAll<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const first = focusables[0];
      const last = focusables[focusables.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [open, onClose]);

  if (!open) return null;

  return (
    <>
      <div className="modal-overlay" onClick={onClose} aria-hidden="true" />
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        className="modal-content"
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose} aria-label="Fechar modal">×</button>
      </div>
    </>
  );
}
```

---

### Componentes Acessíveis — Tabs

> Referência: [WAI-ARIA APG — Tabs Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)

Navegação: Arrow Left/Right entre tabs, Home/End para primeira/última.

```tsx
import { useState, useRef, type ReactNode } from 'react';

interface Tab {
  id: string;
  label: string;
  content: ReactNode;
}

export function AccessibleTabs({ tabs }: { tabs: Tab[] }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const tabRefs = useRef<(HTMLButtonElement | null)[]>([]);

  function handleKeyDown(e: React.KeyboardEvent) {
    let newIndex = activeIndex;

    switch (e.key) {
      case 'ArrowRight':
        newIndex = (activeIndex + 1) % tabs.length;
        break;
      case 'ArrowLeft':
        newIndex = (activeIndex - 1 + tabs.length) % tabs.length;
        break;
      case 'Home':
        newIndex = 0;
        break;
      case 'End':
        newIndex = tabs.length - 1;
        break;
      default:
        return;
    }

    e.preventDefault();
    setActiveIndex(newIndex);
    tabRefs.current[newIndex]?.focus();
  }

  return (
    <div>
      <div role="tablist" aria-label="Abas de conteúdo" onKeyDown={handleKeyDown}>
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            ref={el => { tabRefs.current[index] = el; }}
            role="tab"
            id={`tab-${tab.id}`}
            aria-selected={activeIndex === index}
            aria-controls={`panel-${tab.id}`}
            tabIndex={activeIndex === index ? 0 : -1}
            onClick={() => setActiveIndex(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeIndex !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

---

### Componentes Acessíveis — Accordion

> Referência: [WAI-ARIA APG — Accordion Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/)

```tsx
import { useState } from 'react';

interface AccordionItem {
  id: string;
  title: string;
  content: React.ReactNode;
}

export function AccessibleAccordion({ items }: { items: AccordionItem[] }) {
  const [expandedIds, setExpandedIds] = useState<Set<string>>(new Set());

  function toggle(id: string) {
    setExpandedIds(prev => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id);
      else next.add(id);
      return next;
    });
  }

  return (
    <div>
      {items.map(item => {
        const isExpanded = expandedIds.has(item.id);
        return (
          <div key={item.id}>
            <h3>
              <button
                aria-expanded={isExpanded}
                aria-controls={`accordion-panel-${item.id}`}
                id={`accordion-header-${item.id}`}
                onClick={() => toggle(item.id)}
              >
                {item.title}
                <span aria-hidden="true">{isExpanded ? '−' : '+'}</span>
              </button>
            </h3>
            <div
              id={`accordion-panel-${item.id}`}
              role="region"
              aria-labelledby={`accordion-header-${item.id}`}
              hidden={!isExpanded}
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

---

### Componentes Acessíveis — Carrossel

> Referência: [WAI-ARIA APG — Carousel Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/)

```tsx
import { useState, useEffect, useRef } from 'react';

interface Slide {
  id: string;
  title: string;
  content: React.ReactNode;
}

export function AccessibleCarousel({ slides }: { slides: Slide[] }) {
  const [current, setCurrent] = useState(0);
  const [paused, setPaused] = useState(false);
  const liveRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (paused) return;
    const timer = setInterval(() => {
      setCurrent(prev => (prev + 1) % slides.length);
    }, 5000);
    return () => clearInterval(timer);
  }, [paused, slides.length]);

  return (
    <section
      aria-roledescription="carrossel"
      aria-label="Destaques"
      onMouseEnter={() => setPaused(true)}
      onMouseLeave={() => setPaused(false)}
      onFocus={() => setPaused(true)}
      onBlur={() => setPaused(false)}
    >
      <div className="carousel-controls">
        <button
          aria-label="Slide anterior"
          onClick={() => setCurrent((current - 1 + slides.length) % slides.length)}
        >
          ←
        </button>
        <button
          aria-label={paused ? 'Reproduzir carrossel' : 'Pausar carrossel'}
          onClick={() => setPaused(!paused)}
        >
          {paused ? '▶' : '⏸'}
        </button>
        <button
          aria-label="Próximo slide"
          onClick={() => setCurrent((current + 1) % slides.length)}
        >
          →
        </button>
      </div>

      <div ref={liveRef} aria-live="polite" aria-atomic="true">
        {slides.map((slide, index) => (
          <div
            key={slide.id}
            role="group"
            aria-roledescription="slide"
            aria-label={`Slide ${index + 1} de ${slides.length}: ${slide.title}`}
            hidden={index !== current}
          >
            {slide.content}
          </div>
        ))}
      </div>

      <div role="tablist" aria-label="Selecionar slide">
        {slides.map((slide, index) => (
          <button
            key={slide.id}
            role="tab"
            aria-selected={index === current}
            aria-label={`Slide ${index + 1}: ${slide.title}`}
            onClick={() => setCurrent(index)}
            tabIndex={index === current ? 0 : -1}
          />
        ))}
      </div>
    </section>
  );
}
```

---

### Componentes Acessíveis — Combobox/Autocomplete

> Referência: [WAI-ARIA APG — Combobox Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)

```tsx
import { useState, useRef } from 'react';

interface Option {
  value: string;
  label: string;
}

export function AccessibleCombobox({ options, label }: { options: Option[]; label: string }) {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const inputRef = useRef<HTMLInputElement>(null);
  const listRef = useRef<HTMLUListElement>(null);

  const filtered = options.filter(o =>
    o.label.toLowerCase().includes(query.toLowerCase())
  );

  function handleKeyDown(e: React.KeyboardEvent) {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setIsOpen(true);
        setActiveIndex(prev => Math.min(prev + 1, filtered.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(prev => Math.max(prev - 1, 0));
        break;
      case 'Enter':
        e.preventDefault();
        if (activeIndex >= 0 && filtered[activeIndex]) {
          selectOption(filtered[activeIndex]);
        }
        break;
      case 'Escape':
        setIsOpen(false);
        setActiveIndex(-1);
        break;
    }
  }

  function selectOption(option: Option) {
    setQuery(option.label);
    setIsOpen(false);
    setActiveIndex(-1);
    inputRef.current?.focus();
  }

  const activeId = activeIndex >= 0 ? `option-${filtered[activeIndex]?.value}` : undefined;

  return (
    <div className="combobox-container">
      <label htmlFor="combobox-input">{label}</label>
      <input
        ref={inputRef}
        id="combobox-input"
        role="combobox"
        aria-expanded={isOpen}
        aria-autocomplete="list"
        aria-controls="combobox-listbox"
        aria-activedescendant={activeId}
        value={query}
        onChange={e => {
          setQuery(e.target.value);
          setIsOpen(true);
          setActiveIndex(-1);
        }}
        onFocus={() => setIsOpen(true)}
        onBlur={() => setTimeout(() => setIsOpen(false), 200)}
        onKeyDown={handleKeyDown}
      />
      {isOpen && filtered.length > 0 && (
        <ul ref={listRef} id="combobox-listbox" role="listbox">
          {filtered.map((option, index) => (
            <li
              key={option.value}
              id={`option-${option.value}`}
              role="option"
              aria-selected={index === activeIndex}
              onClick={() => selectOption(option)}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
      {isOpen && filtered.length === 0 && (
        <div role="status">Nenhum resultado encontrado</div>
      )}
    </div>
  );
}
```

---

### Componentes Acessíveis — Data Table com Ordenação

```tsx
import { useState } from 'react';

type SortDirection = 'ascending' | 'descending' | 'none';

interface Column<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
}

interface DataTableProps<T> {
  columns: Column<T>[];
  data: T[];
  caption: string;
}

export function AccessibleDataTable<T extends Record<string, unknown>>({
  columns, data, caption,
}: DataTableProps<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null);
  const [sortDir, setSortDir] = useState<SortDirection>('none');
  const [announcement, setAnnouncement] = useState('');

  function handleSort(key: keyof T) {
    let newDir: SortDirection;
    if (sortKey !== key) {
      newDir = 'ascending';
    } else if (sortDir === 'ascending') {
      newDir = 'descending';
    } else {
      newDir = 'ascending';
    }
    setSortKey(key);
    setSortDir(newDir);

    const col = columns.find(c => c.key === key);
    setAnnouncement(
      `Tabela ordenada por ${col?.label}, ordem ${newDir === 'ascending' ? 'crescente' : 'decrescente'}`
    );
  }

  const sortedData = [...data].sort((a, b) => {
    if (!sortKey || sortDir === 'none') return 0;
    const aVal = a[sortKey];
    const bVal = b[sortKey];
    const cmp = String(aVal).localeCompare(String(bVal));
    return sortDir === 'ascending' ? cmp : -cmp;
  });

  return (
    <>
      <div aria-live="polite" className="sr-only">{announcement}</div>
      <table>
        <caption>{caption}</caption>
        <thead>
          <tr>
            {columns.map(col => (
              <th
                key={String(col.key)}
                aria-sort={sortKey === col.key ? sortDir : undefined}
              >
                {col.sortable ? (
                  <button onClick={() => handleSort(col.key)}>
                    {col.label}
                    <span aria-hidden="true">
                      {sortKey === col.key
                        ? sortDir === 'ascending' ? ' ↑' : ' ↓'
                        : ' ↕'}
                    </span>
                  </button>
                ) : (
                  col.label
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {sortedData.map((row, i) => (
            <tr key={i}>
              {columns.map(col => (
                <td key={String(col.key)}>{String(row[col.key])}</td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </>
  );
}
```

---

### Formulários Acessíveis Avançados

**Validação inline com `aria-invalid` e `aria-errormessage`:**

```html
<div class="form-field">
  <label for="email">E-mail <span aria-hidden="true">*</span></label>
  <input
    id="email"
    type="email"
    required
    aria-required="true"
    aria-invalid="true"
    aria-errormessage="email-error"
    aria-describedby="email-hint"
  />
  <span id="email-hint" class="hint">Será usado para login</span>
  <span id="email-error" class="error" role="alert">
    Informe um e-mail válido (ex: nome@dominio.com)
  </span>
</div>
```

**Formulário multi-step com indicação de progresso:**

```tsx
function MultiStepForm() {
  const [step, setStep] = useState(1);
  const totalSteps = 3;

  return (
    <form aria-label="Cadastro de conta">
      {/* Indicador de progresso */}
      <div role="progressbar"
           aria-valuenow={step}
           aria-valuemin={1}
           aria-valuemax={totalSteps}
           aria-label={`Passo ${step} de ${totalSteps}`}>
        <span className="sr-only">Passo {step} de {totalSteps}</span>
      </div>

      {/* Anúncio de mudança de etapa */}
      <div aria-live="polite" className="sr-only">
        {`Etapa ${step}: ${['Dados pessoais', 'Endereço', 'Confirmação'][step - 1]}`}
      </div>

      <fieldset>
        <legend>
          Passo {step} de {totalSteps} — {['Dados pessoais', 'Endereço', 'Confirmação'][step - 1]}
        </legend>

        {step === 1 && (
          <>
            <label htmlFor="name">Nome completo</label>
            <input id="name" required aria-required="true" autoComplete="name" />
          </>
        )}

        {step === 2 && (
          <>
            <label htmlFor="cep">CEP</label>
            <input id="cep" required aria-required="true" autoComplete="postal-code" />
          </>
        )}

        {step === 3 && (
          <p>Revise seus dados antes de confirmar.</p>
        )}
      </fieldset>

      <div>
        {step > 1 && <button type="button" onClick={() => setStep(s => s - 1)}>Voltar</button>}
        {step < totalSteps
          ? <button type="button" onClick={() => setStep(s => s + 1)}>Próximo</button>
          : <button type="submit">Confirmar</button>}
      </div>
    </form>
  );
}
```

---

### Navegação por Teclado

**Skip link — pular para o conteúdo principal:**

```html
<body>
  <a href="#main-content" class="skip-link">Pular para o conteúdo principal</a>
  <nav><!-- navegação --></nav>
  <main id="main-content" tabindex="-1">
    <!-- conteúdo -->
  </main>
</body>
```

```css
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: 9999;
  padding: 1rem;
  background: var(--color-primary);
  color: var(--color-on-primary);
}
.skip-link:focus {
  top: 0;
}
```

**Focus visible — indicador de foco acessível:**

```css
/* Remove outline apenas para mouse, mantém para teclado */
:focus:not(:focus-visible) {
  outline: none;
}

:focus-visible {
  outline: 3px solid var(--color-focus-ring);
  outline-offset: 2px;
}

/* Em dark mode, o anel de foco deve ter contraste suficiente */
[data-theme="dark"] :focus-visible {
  outline-color: var(--blue-400);
}
```

**Utilitário de focus trap (reutilizável):**

```typescript
export function trapFocus(container: HTMLElement): () => void {
  const focusables = container.querySelectorAll<HTMLElement>(
    'a[href], button:not(:disabled), input:not(:disabled), select:not(:disabled), textarea:not(:disabled), [tabindex]:not([tabindex="-1"])'
  );
  const first = focusables[0];
  const last = focusables[focusables.length - 1];

  function handler(e: KeyboardEvent) {
    if (e.key !== 'Tab') return;
    if (e.shiftKey && document.activeElement === first) {
      e.preventDefault();
      last.focus();
    } else if (!e.shiftKey && document.activeElement === last) {
      e.preventDefault();
      first.focus();
    }
  }

  container.addEventListener('keydown', handler);
  first?.focus();

  return () => container.removeEventListener('keydown', handler);
}
```

---

### Teste com Leitores de Tela (NVDA/VoiceOver)

**NVDA (Windows — gratuito):**

| Ação | Atalho |
|---|---|
| Ligar/desligar | `Ctrl + Alt + N` |
| Ler próximo elemento | `↓` |
| Ler elemento anterior | `↑` |
| Ativar elemento | `Enter` |
| Navegar por headings | `H` (próximo), `Shift+H` (anterior) |
| Navegar por landmarks | `D` |
| Navegar por links | `K` |
| Navegar por formulários | `F` |
| Listar elementos | `NVDA + F7` |
| Modo de navegação/foco | `NVDA + Espaço` |

**VoiceOver (macOS):**

| Ação | Atalho |
|---|---|
| Ligar/desligar | `Cmd + F5` |
| Ler próximo | `VO + →` (VO = Ctrl + Option) |
| Ler anterior | `VO + ←` |
| Ativar elemento | `VO + Espaço` |
| Navegar por headings | `VO + Cmd + H` |
| Abrir Rotor | `VO + U` |
| Interagir com grupo | `VO + Shift + ↓` |
| Sair de grupo | `VO + Shift + ↑` |

**O que testar:**

- [ ] Todos os elementos interativos são focáveis e ativáveis
- [ ] Imagens têm `alt` descritivo (ou estão ocultas se decorativas)
- [ ] Formulários anunciam labels, hints e erros
- [ ] Modais prendem o foco e são fecháveis com Escape
- [ ] Mudanças dinâmicas (toasts, filtros) são anunciadas via `aria-live`
- [ ] A ordem de leitura faz sentido semântico
- [ ] Headings formam hierarquia lógica (h1 → h2 → h3)
- [ ] Skip link funciona e leva ao conteúdo principal

---

### Teste Automatizado de Acessibilidade em CI/CD

> Para configuração detalhada de vitest-axe, cypress-axe e playwright, consulte [Testes-Frontend-JS-React.md](outros/Testes-Frontend-JS-React.md).

**Pipeline GitHub Actions com múltiplas verificações:**

```yaml
# .github/workflows/a11y.yml
name: Accessibility Tests
on: [push, pull_request]
jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      # 1. Testes unitários de acessibilidade (vitest + axe)
      - name: Unit a11y tests
        run: npm test -- --reporter=verbose

      # 2. Build para testes e2e
      - run: npm run build

      # 3. pa11y-ci — testa múltiplas páginas
      - name: pa11y-ci
        run: |
          npx serve -s dist -l 3000 &
          npx pa11y-ci --config .pa11yci.json

      # 4. Lighthouse CI
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: .lighthouserc.json
```

```json
// .pa11yci.json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 10000,
    "wait": 1000
  },
  "urls": [
    "http://localhost:3000/",
    "http://localhost:3000/login",
    "http://localhost:3000/produtos",
    "http://localhost:3000/cadastro"
  ]
}
```

---

### Acessibilidade em SPAs

SPAs não recarregam a página na navegação, então o leitor de tela não anuncia mudanças de rota automaticamente.

**Anunciar mudanças de rota:**

```tsx
// React Router
import { useLocation } from 'react-router-dom';
import { useEffect, useRef } from 'react';

export function RouteAnnouncer() {
  const location = useLocation();
  const announcerRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const pageTitle = document.title;
    if (announcerRef.current) {
      announcerRef.current.textContent = `Navegou para ${pageTitle}`;
    }

    // Mover foco para o heading principal
    const h1 = document.querySelector('h1');
    if (h1) {
      h1.setAttribute('tabindex', '-1');
      h1.focus();
    }
  }, [location.pathname]);

  return (
    <div
      ref={announcerRef}
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    />
  );
}
```

**Atualizar `<title>` em cada rota:**

```tsx
// React — useEffect em cada página
useEffect(() => {
  document.title = 'Produtos — MeuApp';
}, []);

// Angular — TitleStrategy
import { TitleStrategy, RouterStateSnapshot } from '@angular/router';
import { Title } from '@angular/platform-browser';

export class AppTitleStrategy extends TitleStrategy {
  constructor(private title: Title) { super(); }

  updateTitle(snapshot: RouterStateSnapshot) {
    const title = this.buildTitle(snapshot);
    this.title.setTitle(title ? `${title} — MeuApp` : 'MeuApp');
  }
}

// Vue Router — afterEach
router.afterEach((to) => {
  const title = to.meta.title as string;
  document.title = title ? `${title} — MeuApp` : 'MeuApp';
});
```

---

### Acessibilidade em E-mails HTML

E-mails transacionais (confirmação de pedido, reset de senha, notificações) são frequentemente esquecidos na estratégia de acessibilidade, mas são usados por todos os usuários.

**Limitações dos clientes de e-mail:**

| Recurso | Suporte |
|---|---|
| CSS externo / `<link>` | Não (maioria remove) |
| `<style>` no `<head>` | Parcial (Gmail remove em mobile) |
| CSS inline | Sim (recomendado) |
| `role`, `aria-*` | Parcial (Apple Mail sim, Gmail não) |
| `<div>` para layout | Inconsistente |
| `<table>` para layout | Sim (padrão seguro) |
| Imagens | Bloqueadas por padrão em muitos clientes |
| Dark mode | Suporte variável (pode inverter cores) |

**Boas práticas de acessibilidade em e-mails:**

```html
<!DOCTYPE html>
<html lang="pt-BR" xmlns="http://www.w3.org/1999/xhtml">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta name="color-scheme" content="light dark" />
  <title>Confirmação de Pedido #1234</title>
  <!--[if mso]>
  <style>table { border-collapse: collapse; }</style>
  <![endif]-->
</head>
<body style="margin: 0; padding: 0; background-color: #f8fafc;">

  <!-- Role presentation remove semântica de tabela para leitores de tela -->
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0"
         style="max-width: 600px; margin: 0 auto; background: #ffffff;">
    <tr>
      <td style="padding: 24px;">
        <!-- Logo com alt descritivo -->
        <img src="https://example.com/logo.png" alt="MeuApp" width="120"
             style="display: block;" />
      </td>
    </tr>
    <tr>
      <td style="padding: 0 24px;">
        <h1 style="font-size: 24px; color: #1e293b; font-family: Arial, sans-serif;">
          Pedido confirmado!
        </h1>
        <p style="font-size: 16px; color: #475569; line-height: 1.5;
                  font-family: Arial, sans-serif;">
          Seu pedido <strong>#1234</strong> foi confirmado com sucesso.
        </p>

        <!-- Botão CTA acessível -->
        <table role="presentation" cellpadding="0" cellspacing="0">
          <tr>
            <td style="background: #2563eb; border-radius: 8px; padding: 12px 24px;">
              <a href="https://example.com/pedido/1234"
                 style="color: #ffffff; text-decoration: none; font-size: 16px;
                        font-weight: 600; font-family: Arial, sans-serif;
                        display: inline-block;">
                Ver meu pedido
              </a>
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>

</body>
</html>
```

**Checklist para e-mails acessíveis:**

- [ ] `lang` definido no `<html>`
- [ ] `<title>` descritivo (leitores de tela anunciam)
- [ ] Tabelas de layout com `role="presentation"`
- [ ] Todas as imagens têm `alt` (ou `alt=""` se decorativas)
- [ ] O e-mail é legível sem imagens (texto alternativo suficiente)
- [ ] Contraste ≥ 4.5:1 para texto
- [ ] Fonte mínima de 14px para corpo
- [ ] Links descritivos (não "clique aqui")
- [ ] Hierarquia de headings (`<h1>`, `<h2>`)
- [ ] Funciona em plain-text (versão alternativa)
- [ ] Largura máxima de ~600px para legibilidade
- [ ] Testado em dark mode (cores adaptáveis)

---

### Checklist de Acessibilidade por Nível

**Nível A (mínimo obrigatório):**

- [ ] Conteúdo não-textual tem alternativa textual (`alt`, `aria-label`)
- [ ] Vídeos têm legendas
- [ ] Informação não depende apenas de cor
- [ ] Conteúdo pode ser apresentado sem perda em diferentes orientações
- [ ] Elementos interativos são acessíveis por teclado
- [ ] Não há armadilhas de teclado (keyboard traps)
- [ ] Mecanismos de ajuda estão na mesma posição relativa (3.2.6)
- [ ] Informações já fornecidas não são requisitadas novamente (3.3.7)
- [ ] Títulos de página são descritivos
- [ ] A ordem de foco é lógica
- [ ] Links têm texto descritivo

**Nível AA (recomendado):**

- [ ] Contraste de texto ≥ 4.5:1 (normal) ou ≥ 3:1 (grande)
- [ ] Texto pode ser ampliado até 200% sem perda de funcionalidade
- [ ] Imagens de texto não são usadas (exceto logotipos)
- [ ] Navegação consistente entre páginas
- [ ] Elementos de foco não estão totalmente ocultos (2.4.11)
- [ ] Arrastar tem alternativa sem arraste (2.5.7)
- [ ] Áreas de toque ≥ 24x24px (2.5.8)
- [ ] Login não depende de teste cognitivo sem alternativa (3.3.8)
- [ ] Mensagens de erro identificam o campo e sugerem correção
- [ ] Formulários têm labels visíveis
- [ ] Estados de hover têm equivalente em foco

**Nível AAA (aspiracional):**

- [ ] Contraste ≥ 7:1 (normal) ou ≥ 4.5:1 (grande)
- [ ] Linguagem simples ou glossário disponível
- [ ] Abreviações são expandidas
- [ ] Elementos de foco não estão parcialmente ocultos (2.4.12)
- [ ] Indicador de foco com contraste e área suficientes (2.4.13)
- [ ] Login não depende de reconhecimento de objetos (3.3.9)
- [ ] Timeouts podem ser desativados

---

## 8. Referências

### Design Systems e Design Tokens
- [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/)
- [W3C Design Tokens Format](https://tr.designtokens.org/format/)
- [Style Dictionary (Amazon)](https://amzn.github.io/style-dictionary/)
- [Tokens Studio for Figma](https://tokens.studio/)
- [Storybook](https://storybook.js.org/)
- [Chromatic — Visual Testing](https://www.chromatic.com/)

### Tailwind CSS
- [Tailwind CSS v4 Docs](https://tailwindcss.com/docs)
- [Tailwind UI](https://tailwindui.com/)
- [Headless UI](https://headlessui.com/)
- [Class Variance Authority (CVA)](https://cva.style/)
- [tailwind-merge](https://github.com/dcastil/tailwind-merge)

### Bootstrap
- [Bootstrap 5 Docs](https://getbootstrap.com/docs/)
- [Bootstrap Icons](https://icons.getbootstrap.com/)
- [react-bootstrap](https://react-bootstrap.github.io/)
- [ng-bootstrap](https://ng-bootstrap.github.io/)
- [bootstrap-vue-next](https://bootstrap-vue-next.github.io/)

### Material UI e Bibliotecas de Componentes
- [MUI (Material UI)](https://mui.com/)
- [Angular Material](https://material.angular.io/)
- [shadcn/ui](https://ui.shadcn.com/)
- [Radix UI](https://www.radix-ui.com/)
- [TanStack Table](https://tanstack.com/table)
- [TanStack Virtual](https://tanstack.com/virtual)
- [Recharts](https://recharts.org/)

### Design Mobile
- [Material Design 3](https://m3.material.io/)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [SF Symbols](https://developer.apple.com/sf-symbols/)
- [Material Symbols](https://fonts.google.com/icons)
- [Framer Motion](https://www.framer.com/motion/)
- [LottieFiles](https://lottiefiles.com/)

### Progressive Web Apps
- [web.dev — Progressive Web Apps](https://web.dev/progressive-web-apps/)
- [Workbox](https://developer.chrome.com/docs/workbox/)
- [MDN — Progressive Web Apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [Vite PWA Plugin](https://vite-pwa-org.netlify.app/)
- [web.dev — Core Web Vitals](https://web.dev/vitals/)
- [web-vitals (npm)](https://github.com/GoogleChrome/web-vitals)

### Acessibilidade
- [WCAG 2.2 (W3C)](https://www.w3.org/TR/WCAG22/)
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [WebAIM](https://webaim.org/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [APCA Contrast Calculator](https://www.myndex.com/APCA/)
- [Deque axe-core](https://www.deque.com/axe/)
- [pa11y](https://pa11y.org/)
- [e-MAG — Governo Federal](https://emag.governoeletronico.gov.br/)

### APIs Web Modernas
- [MDN — Popover API](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API)
- [MDN — dialog element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog)
- [MDN — CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)
- [MDN — Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)
- [MDN — View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)

---
