# Figma — Layouts Responsivos para Web e Mobile

Guia prático para criação de wireframes (baixa fidelidade) e layouts de alta fidelidade no Figma, com foco em design responsivo para web e aplicativos mobile.

---

## Sumário

1. [Configuração Inicial do Projeto](#1-configuração-inicial-do-projeto)
2. [Atalhos de Teclado Essenciais](#2-atalhos-de-teclado-essenciais)
3. [Frames e Tamanhos de Tela](#3-frames-e-tamanhos-de-tela)
4. [Auto Layout — Base do Design Responsivo](#4-auto-layout--base-do-design-responsivo)
5. [Constraints (Restrições)](#5-constraints-restrições)
6. [Grids e Colunas](#6-grids-e-colunas)
7. [Wireframes — Baixa Fidelidade](#7-wireframes--baixa-fidelidade)
8. [Layout de Alta Fidelidade](#8-layout-de-alta-fidelidade)
9. [Componentes e Variantes](#9-componentes-e-variantes)
10. [Dark Mode e Temas com Variables](#10-dark-mode-e-temas-com-variables)
11. [Acessibilidade no Design](#11-acessibilidade-no-design)
12. [Design Responsivo — Estratégias](#12-design-responsivo--estratégias)
13. [Prototipagem e Interações](#13-prototipagem-e-interações)
14. [Colaboração e Organização de Projeto](#14-colaboração-e-organização-de-projeto)
15. [Handoff para Desenvolvedores](#15-handoff-para-desenvolvedores)
16. [Do Figma para Código — Web (HTML/CSS)](#16-do-figma-para-código--web-htmlcss)
17. [Do Figma para Código — Flutter](#17-do-figma-para-código--flutter)
18. [Do Figma para Código — React Native](#18-do-figma-para-código--react-native)
19. [Onde Obter Assets e Recursos](#19-onde-obter-assets-e-recursos)
20. [Plugins Recomendados](#20-plugins-recomendados)
21. [Referências](#21-referências)

---

## 1. Configuração Inicial do Projeto

### Criar um novo arquivo

1. Acesse [figma.com](https://www.figma.com) ou abra o app desktop
2. Crie um novo **Design File**
3. Organize o arquivo em **Pages** (páginas):
   - `Wireframes` — rascunhos de baixa fidelidade
   - `Design` — telas de alta fidelidade
   - `Components` — biblioteca de componentes reutilizáveis
   - `Protótipo` — fluxos interativos

### Configurar estilos base

Antes de iniciar os layouts, defina os **estilos globais** do projeto:

| Tipo | O que definir | Exemplo |
|---|---|---|
| **Cores** | Paleta primária, secundária, neutros, feedback | `Primary/500: #1976D2` |
| **Tipografia** | Font family, tamanhos, pesos | `Heading/H1: Inter Bold 32px` |
| **Espaçamentos** | Escala de espaçamento consistente | `4, 8, 12, 16, 24, 32, 48, 64` |
| **Efeitos** | Sombras, blur | `Shadow/MD: 0 4 6 rgba(0,0,0,0.1)` |
| **Border radius** | Arredondamentos padrão | `SM: 4px, MD: 8px, LG: 16px` |

### Variables (Variáveis) — recurso mais recente

O Figma suporta **Variables** que funcionam de forma similar a variáveis CSS ou Design Tokens:

- **Color variables:** definem paletas que podem alternar entre temas (light/dark)
- **Number variables:** definem valores de espaçamento e border-radius reutilizáveis
- **Boolean variables:** controlam visibilidade de elementos
- **String variables:** textos dinâmicos

Para criar: menu principal → clique no ícone de variáveis no painel direito ou `Variables` na barra lateral.

---

## 2. Atalhos de Teclado Essenciais

Conhecer os atalhos do Figma acelera significativamente o fluxo de trabalho. Os atalhos abaixo usam a notação Windows — no macOS substitua **Ctrl** por **Cmd** e **Alt** por **Option**.

### Navegação e visualização

| Atalho | Ação |
|---|---|
| **Ctrl + Scroll** | Zoom in/out |
| **Ctrl + 0** | Zoom para 100% |
| **Ctrl + 1** | Zoom to fit (ajustar tudo na tela) |
| **Ctrl + 2** | Zoom to selection (ajustar no elemento selecionado) |
| **Espaço + Arrastar** | Mover a área de trabalho (pan) |
| **Ctrl + \\** | Mostrar/ocultar a interface do Figma |
| **Ctrl + Shift + \\** | Mostrar/ocultar painel esquerdo (layers) |

### Ferramentas

| Atalho | Ferramenta |
|---|---|
| **V** | Move (seleção — ferramenta padrão) |
| **F** | Frame |
| **R** | Retângulo |
| **O** | Elipse (círculo) |
| **L** | Linha |
| **T** | Texto |
| **P** | Caneta (Pen — vetores) |
| **I** | Eyedropper (conta-gotas de cor) |
| **K** | Scale (redimensionar proporcionalmente) |
| **S** | Slice (recortar para exportação) |
| **C** | Comentário |
| **H** | Hand tool (mover canvas) |

### Edição e organização

| Atalho | Ação |
|---|---|
| **Ctrl + D** | Duplicar elemento |
| **Ctrl + G** | Agrupar seleção (Group) |
| **Ctrl + Shift + G** | Desagrupar |
| **Ctrl + Alt + K** | Criar componente |
| **Ctrl + Shift + H** | Inverter horizontalmente (flip) |
| **Ctrl + Shift + V** | Inverter verticalmente (flip) |
| **Ctrl + ]** | Mover para frente (bring forward) |
| **Ctrl + [** | Mover para trás (send backward) |
| **Ctrl + Shift + ]** | Trazer para o topo |
| **Ctrl + Shift + [** | Enviar para o fundo |
| **Ctrl + R** | Renomear layer selecionado |

### Auto Layout e alinhamento

| Atalho | Ação |
|---|---|
| **Shift + A** | Adicionar Auto Layout ao frame selecionado |
| **Alt + A** | Alinhar à esquerda |
| **Alt + D** | Alinhar à direita |
| **Alt + W** | Alinhar ao topo |
| **Alt + S** | Alinhar à base |
| **Alt + H** | Centralizar horizontalmente |
| **Alt + V** | Centralizar verticalmente |
| **Ctrl + Shift + T** | Distribuir espaçamento horizontal uniforme (Tidy Up) |

### Prototipagem e inspeção

| Atalho | Ação |
|---|---|
| **Ctrl + Enter** | Abrir preview do protótipo |
| **Shift + E** | Alternar para Dev Mode |

### Dicas de produtividade

- **Alt + Arrastar** — duplica o elemento enquanto move (mais rápido que Ctrl+D)
- **Shift + Arrastar** — restringe o movimento ao eixo horizontal ou vertical
- **Alt + Hover** — mostra as distâncias entre o elemento selecionado e o elemento sob o cursor
- **Ctrl + Shift + O** — converte texto em contorno vetorial (Outline Stroke)
- **Shift + Click no layer** — seleciona múltiplos layers
- **Enter** — entra no grupo/frame selecionado (desce na hierarquia)
- **Esc** — sobe um nível na hierarquia de layers

> **Dica:** Pressione **Ctrl + Shift + ?** dentro do Figma para abrir o painel completo de atalhos de teclado.

---

## 3. Frames e Tamanhos de Tela

### Frames pré-definidos

Pressione **F** (ou **A** no modo antigo) para criar um Frame. No painel direito, escolha entre os presets:

| Categoria | Dispositivo | Largura × Altura |
|---|---|---|
| **Mobile** | iPhone 14 / 15 | 393 × 852 |
| **Mobile** | iPhone SE | 375 × 667 |
| **Mobile** | Android Small | 360 × 800 |
| **Mobile** | Android Large | 412 × 915 |
| **Tablet** | iPad Mini | 744 × 1133 |
| **Tablet** | iPad Pro 11" | 834 × 1194 |
| **Tablet** | iPad Pro 12.9" | 1024 × 1366 |
| **Desktop** | Desktop (padrão) | 1440 × 1024 |
| **Desktop** | MacBook Pro 14" | 1512 × 982 |
| **Desktop** | Full HD (1080p) | 1920 × 1080 |

### Breakpoints recomendados para web responsiva

| Breakpoint | Largura | Uso típico |
|---|---|---|
| **xs** | < 576px | Smartphones em retrato |
| **sm** | 576px – 767px | Smartphones em paisagem |
| **md** | 768px – 991px | Tablets |
| **lg** | 992px – 1199px | Laptops |
| **xl** | 1200px – 1399px | Desktops |
| **xxl** | ≥ 1400px | Telas grandes |

> **Dica:** Crie pelo menos 3 frames para cada tela: **Mobile (375px)**, **Tablet (768px)** e **Desktop (1440px)**.

---

## 4. Auto Layout — Base do Design Responsivo

**Auto Layout** é o recurso mais importante do Figma para criar layouts responsivos. Funciona de forma análoga ao **Flexbox** do CSS.

### Como ativar

- Selecione um frame ou grupo de elementos
- Pressione **Shift + A**
- Ou clique no ícone **+** ao lado de "Auto Layout" no painel direito

### Propriedades principais

| Propriedade Figma | Equivalente CSS | Descrição |
|---|---|---|
| **Direction** | `flex-direction` | Horizontal (row) ou Vertical (column) |
| **Gap** | `gap` | Espaço entre os itens filhos |
| **Padding** | `padding` | Espaço interno do container |
| **Alignment** | `align-items` / `justify-content` | Alinhamento dos itens |
| **Wrap** | `flex-wrap` | Permite quebra de linha quando não cabe |

### Redimensionamento dos filhos

| Opção | Equivalente CSS | Comportamento |
|---|---|---|
| **Fixed** | `width: Npx` | Tamanho fixo em pixels |
| **Hug** | `width: fit-content` | Ajusta ao conteúdo |
| **Fill** | `flex: 1` ou `width: 100%` | Preenche o espaço disponível |

### Exemplo: Card responsivo

```
Frame (Auto Layout - Vertical)
├── Padding: 16px
├── Gap: 12px
├── Width: Fill
│
├── Image (Fixed height: 200px, Width: Fill)
├── Title (Hug height, Width: Fill)
├── Description (Hug height, Width: Fill)
└── Button (Hug, Align: right)
```

### Auto Layout com Wrap

Para criar grids responsivos (ex: galeria de cards):

1. Crie um frame com Auto Layout **Horizontal**
2. Ative **Wrap** nas configurações do Auto Layout
3. Defina o **gap** horizontal e vertical
4. Defina largura **mínima** e **máxima** nos itens filhos

Isso é equivalente ao CSS:
```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}
.item {
  min-width: 280px;
  flex: 1;
}
```

---

## 5. Constraints (Restrições)

Constraints definem como um elemento se comporta quando o frame pai é redimensionado. Útil para posicionamento absoluto (equivalente a `position: absolute` + propriedades de ancoragem).

| Constraint | Comportamento | Equivalente CSS |
|---|---|---|
| **Left** | Mantém distância da borda esquerda | `left: Npx` |
| **Right** | Mantém distância da borda direita | `right: Npx` |
| **Left and Right** | Estica horizontalmente | `left: Npx; right: Npx` |
| **Center** | Centraliza no eixo | `margin: 0 auto` |
| **Scale** | Escala proporcionalmente | `width: N%` |
| **Top** | Mantém distância do topo | `top: Npx` |
| **Bottom** | Mantém distância da base | `bottom: Npx` |
| **Top and Bottom** | Estica verticalmente | `top: Npx; bottom: Npx` |

> **Quando usar Constraints vs Auto Layout:**
> - **Auto Layout:** para fluxos de conteúdo (listas, cards, formulários)
> - **Constraints:** para elementos fixos (navbar fixa, FAB, overlays)

---

## 6. Grids e Colunas

### Configurar Layout Grid

1. Selecione o frame da tela
2. No painel direito, clique em **+** ao lado de **Layout Grid**
3. Escolha o tipo de grid

### Tipos de grid

| Tipo | Uso |
|---|---|
| **Grid** | Grade quadriculada uniforme (útil para alinhamento geral) |
| **Columns** | Colunas verticais — o mais usado para web responsiva |
| **Rows** | Linhas horizontais (menos comum) |

### Configuração de colunas por breakpoint

| Breakpoint | Colunas | Margem | Gutter |
|---|---|---|---|
| **Mobile** (375px) | 4 | 16px | 16px |
| **Tablet** (768px) | 8 | 32px | 24px |
| **Desktop** (1440px) | 12 | 80-120px | 24-32px |

### Tipo de colunas

| Tipo | Descrição |
|---|---|
| **Stretch** | Colunas se ajustam à largura do frame (mais usado para responsivo) |
| **Center** | Colunas centralizadas com largura fixa |
| **Left** | Colunas alinhadas à esquerda com largura fixa |

> **Dica:** Use **Stretch** com margens fixas para simular o comportamento de containers CSS como o do Bootstrap ou Tailwind.

---

## 7. Wireframes — Baixa Fidelidade

Wireframes são rascunhos estruturais que focam na **disposição do conteúdo** e **hierarquia de informação**, sem se preocupar com cores, imagens ou detalhes visuais.

### Princípios

- Usar **tons de cinza** (sem cores)
- Tipografia simples (uma única font family)
- Retângulos simples para representar imagens
- Foco na **estrutura** e **fluxo de navegação**
- Rapidez na iteração

### Elementos básicos de wireframe

| Elemento | Como representar no Figma |
|---|---|
| **Imagem** | Retângulo cinza com ícone de paisagem ou "X" |
| **Texto título** | Retângulo ou texto em cinza escuro, fonte maior |
| **Texto corpo** | Linhas horizontais cinza claro ou texto Lorem Ipsum |
| **Botão** | Retângulo com bordas arredondadas + texto |
| **Input** | Retângulo com borda + placeholder text |
| **Ícone** | Círculo ou quadrado pequeno cinza |
| **Avatar** | Círculo cinza |
| **Card** | Retângulo com borda ou sombra sutil contendo elementos |

### Paleta de cores para wireframe

| Uso | Cor | Hex |
|---|---|---|
| **Fundo da tela** | Branco | `#FFFFFF` |
| **Fundo de seção** | Cinza muito claro | `#F5F5F5` |
| **Placeholder de imagem** | Cinza claro | `#E0E0E0` |
| **Texto primário** | Cinza escuro | `#333333` |
| **Texto secundário** | Cinza médio | `#888888` |
| **Bordas e linhas** | Cinza | `#CCCCCC` |
| **Destaque / CTA** | Cinza escuro (ou preto) | `#555555` |

### Roteiro: wireframe de uma tela web

1. Crie um frame **Desktop (1440×1024)**
2. Adicione um **Layout Grid** de 12 colunas (margin: 80px, gutter: 24px)
3. Monte a estrutura com Auto Layout vertical:

```
Frame Desktop (1440 × 1024)
│
├── Header (Auto Layout - Horizontal, Fill width)
│   ├── Logo (placeholder retangular)
│   ├── Nav links (Auto Layout - Horizontal, gap: 24px)
│   └── Botão CTA
│
├── Hero Section (Auto Layout - Horizontal, Fill width, padding: 64px 80px)
│   ├── Coluna texto (Fill, 50%)
│   │   ├── Título H1
│   │   ├── Subtítulo
│   │   └── Botão primário
│   └── Coluna imagem (Fill, 50%)
│       └── Placeholder imagem
│
├── Seção Cards (Auto Layout - Vertical, Fill width, padding: 48px 80px)
│   ├── Título da seção
│   └── Grid de Cards (Auto Layout - Horizontal, Wrap, gap: 24px)
│       ├── Card 1 (min-width: 300px, Fill)
│       ├── Card 2
│       └── Card 3
│
└── Footer (Auto Layout - Horizontal, Fill width, padding: 32px 80px)
    ├── Copyright
    └── Links
```

4. Duplique o frame e redimensione para **768px** (tablet) e **375px** (mobile)
5. Ajuste o layout:
   - **Tablet:** Hero muda para vertical (empilhado), grid de 2 colunas
   - **Mobile:** Tudo empilhado, menu vira hambúrguer, cards em coluna única

### Roteiro: wireframe de app mobile

1. Crie um frame **iPhone 14 (393×852)**
2. Adicione um Layout Grid de 4 colunas (margin: 16px, gutter: 16px)
3. Monte a estrutura:

```
Frame Mobile (393 × 852)
│
├── Status Bar (Fixed, 44px height)
│
├── App Bar (Auto Layout - Horizontal, height: 56px)
│   ├── Ícone menu (24×24)
│   ├── Título da tela (Fill)
│   └── Ícones de ação
│
├── Conteúdo scrollável (Auto Layout - Vertical, Fill, padding: 16px)
│   ├── Search Bar (Fill width, height: 48px)
│   ├── Seção (Auto Layout - Vertical, gap: 12px)
│   │   ├── Subtítulo da seção
│   │   └── Lista de itens
│   │       ├── Item 1 (Auto Layout - Horizontal)
│   │       │   ├── Thumbnail (48×48)
│   │       │   ├── Textos (Fill)
│   │       │   └── Ícone chevron
│   │       ├── Item 2
│   │       └── Item 3
│   └── Seção 2 ...
│
└── Bottom Navigation (Auto Layout - Horizontal, Fixed bottom)
    ├── Tab 1 (ícone + label)
    ├── Tab 2
    ├── Tab 3
    └── Tab 4
```

4. Use **constraints** no Status Bar e Bottom Navigation para fixá-los no topo e na base

---

## 8. Layout de Alta Fidelidade

Transição do wireframe para o design final, adicionando identidade visual completa.

### Checklist de transição wireframe → alta fidelidade

| Aspecto | O que fazer |
|---|---|
| **Cores** | Aplicar paleta de cores definida (Color Styles ou Variables) |
| **Tipografia** | Aplicar a font family, tamanhos e pesos definidos (Text Styles) |
| **Imagens** | Substituir placeholders por imagens reais ou ilustrações |
| **Ícones** | Substituir formas genéricas por ícones do icon set escolhido |
| **Espaçamentos** | Refinar usando a escala de espaçamento definida |
| **Sombras e efeitos** | Adicionar elevação (box-shadow) onde necessário |
| **Bordas** | Aplicar border-radius consistente |
| **Estados** | Criar variantes para hover, active, disabled, focus, error |
| **Conteúdo real** | Substituir Lorem Ipsum por textos reais ou realistas |

### Tipografia — escala recomendada

| Nível | Tamanho | Peso | Line Height | Uso |
|---|---|---|---|---|
| **Display** | 48-64px | Bold (700) | 1.1 | Landing pages, heroes |
| **H1** | 32-40px | Bold (700) | 1.2 | Títulos de página |
| **H2** | 24-28px | Semi-bold (600) | 1.3 | Títulos de seção |
| **H3** | 20-22px | Semi-bold (600) | 1.3 | Subtítulos |
| **Body** | 16px | Regular (400) | 1.5 | Texto principal |
| **Body Small** | 14px | Regular (400) | 1.5 | Texto secundário |
| **Caption** | 12px | Regular (400) | 1.4 | Labels, legendas |

Para mobile, reduza os tamanhos maiores em ~20%:

| Nível | Desktop | Mobile |
|---|---|---|
| **Display** | 56px | 36px |
| **H1** | 36px | 28px |
| **H2** | 28px | 22px |
| **Body** | 16px | 16px (manter) |

### Contraste e acessibilidade

| Nível WCAG | Ratio mínimo | Uso |
|---|---|---|
| **AA (normal text)** | 4.5:1 | Textos ≤ 18px regular |
| **AA (large text)** | 3:1 | Textos ≥ 18px bold ou ≥ 24px regular |
| **AAA** | 7:1 | Nível mais alto de acessibilidade |

> Use o plugin **Stark** ou **A11y - Color Contrast Checker** para verificar o contraste diretamente no Figma.

### Roteiro: layout de alta fidelidade — Landing Page Web

1. **Preparação:**
   - Defina Color Styles: Primary, Secondary, Neutral (50-900), Success, Warning, Error
   - Defina Text Styles: Display, H1-H3, Body, Body Small, Caption
   - Importe um icon set (ex: Material Icons, Phosphor, Lucide)

2. **Header/Navbar:**
   - Auto Layout Horizontal, padding: `12px 80px`, alinhamento centralizado
   - Logo à esquerda (SVG importado ou texto estilizado)
   - Links de navegação centralizados (gap: 32px, cor Neutral/700)
   - CTA button à direita (fundo Primary/500, texto branco, border-radius: 8px)
   - Largura: **Fill** | Constraints: **Left and Right, Top**

3. **Hero Section:**
   - Auto Layout Horizontal, padding: `80px`, gap: `48px`
   - Coluna de texto (Fill, 50%):
     - Título: Text Style Display, cor Neutral/900
     - Subtítulo: Text Style Body, cor Neutral/600, max-width ~500px
     - Botões: Auto Layout Horizontal, gap: 16px (primário + secundário)
   - Coluna visual (Fill, 50%):
     - Imagem ou ilustração com border-radius: 16px

4. **Seção de Features:**
   - Auto Layout Vertical, padding: `64px 80px`, gap: 48px, fundo Neutral/50
   - Título de seção centralizado (H2)
   - Grid de cards (Auto Layout Horizontal, Wrap, gap: 24px):
     - Cada card: Auto Layout Vertical, padding: 24px, border-radius: 12px, sombra Shadow/SM
     - Ícone (48×48, cor Primary/500), Título (H3), Descrição (Body Small, Neutral/600)

5. **Footer:**
   - Auto Layout Horizontal, padding: `48px 80px`, fundo Neutral/900
   - Colunas de links (texto Neutral/300)
   - Copyright (Body Small, Neutral/500)

### Roteiro: layout de alta fidelidade — App Mobile

1. **Status Bar:**
   - Use o componente de status bar do kit de UI do iOS ou Android

2. **App Bar (Material Design 3):**
   - Height: 64px, fundo Primary ou Surface
   - Ícone de navegação (24×24) + Título (H3) + Action icons
   - Elevação: Shadow/SM

3. **Conteúdo:**
   - Auto Layout Vertical, padding: 16px, gap: 16px
   - Cards com border-radius: 12px, padding: 16px
   - Listas com separadores (1px, Neutral/200)
   - Imagens com aspect ratio consistente (16:9, 4:3 ou 1:1)

4. **Bottom Navigation (Material Design 3):**
   - Height: 80px, 3–5 itens
   - Ícone (24×24) + Label (Caption)
   - Item ativo: cor Primary, ícone filled
   - Item inativo: cor Neutral/500, ícone outlined

5. **FAB (Floating Action Button):**
   - 56×56px, border-radius: 16px (MD3)
   - Constraint: Bottom + Right
   - Sombra: Shadow/LG
   - Cor: Primary ou Secondary container

---

## 9. Componentes e Variantes

### Criar componentes reutilizáveis

1. Selecione o elemento
2. Pressione **Ctrl + Alt + K** (Windows) ou **Cmd + Option + K** (Mac)
3. Nomeie com a convenção: `Categoria/NomeDoComponente` (ex: `Button/Primary`)

### Convenção de nomenclatura

```
Button/Primary
Button/Secondary
Button/Outline
Card/Default
Card/Horizontal
Input/Text
Input/Password
Input/Error
Nav/Header
Nav/BottomBar
Nav/Sidebar
```

### Variantes

Variantes permitem criar múltiplos estados de um componente em um único component set:

| Propriedade | Valores típicos |
|---|---|
| **Type** | Primary, Secondary, Outline, Ghost, Link |
| **Size** | Small, Medium, Large |
| **State** | Default, Hover, Active, Disabled, Focus |
| **Icon** | Leading, Trailing, Only, None |

Para criar:
1. Selecione o componente
2. No painel direito, clique em **+** ao lado de "Variants"
3. Configure as propriedades e seus valores

### Componentes essenciais para um Design System mínimo

| Componente | Variantes mínimas |
|---|---|
| **Button** | Primary, Secondary, Outline × Small, Medium, Large × Default, Hover, Disabled |
| **Input** | Text, Password, TextArea × Default, Focus, Error, Disabled |
| **Checkbox / Radio** | Checked, Unchecked × Default, Disabled |
| **Select / Dropdown** | Closed, Open |
| **Card** | Default, Horizontal |
| **Badge / Tag** | Info, Success, Warning, Error |
| **Alert / Toast** | Info, Success, Warning, Error |
| **Avatar** | Image, Initials × Small, Medium, Large |
| **Modal / Dialog** | Default |
| **Navbar** | Desktop, Mobile |
| **Tab Bar** | Default |
| **Skeleton** | Text, Image, Card (para loading states) |

---

## 10. Dark Mode e Temas com Variables

O recurso de **Variables** do Figma permite criar sistemas de temas (Light/Dark Mode) que alternam cores, espaçamentos e outros valores de design de forma centralizada — equivalente a variáveis CSS com media query `prefers-color-scheme` ou `ThemeData` no Flutter.

### Conceitos fundamentais

| Conceito | Descrição |
|---|---|
| **Variable** | Um valor nomeado reutilizável (cor, número, string, boolean) |
| **Collection** | Grupo de variáveis relacionadas (ex: "Colors", "Spacing") |
| **Mode** | Variação dentro de uma collection (ex: "Light" e "Dark") |

### Passo a passo: criar tema Light/Dark

#### 1. Criar a Collection de cores

1. Abra o painel **Variables** (ícone na barra lateral ou menu **Local Variables**)
2. Crie uma nova collection chamada `Colors`
3. Adicione dois **Modes**: `Light` e `Dark`

#### 2. Definir as variáveis de cor

| Nome da variável | Light | Dark |
|---|---|---|
| `surface/background` | `#FFFFFF` | `#121212` |
| `surface/card` | `#F5F5F5` | `#1E1E1E` |
| `surface/elevated` | `#FFFFFF` | `#2C2C2C` |
| `text/primary` | `#1A1A1A` | `#E0E0E0` |
| `text/secondary` | `#666666` | `#9E9E9E` |
| `text/disabled` | `#BDBDBD` | `#616161` |
| `border/default` | `#E0E0E0` | `#333333` |
| `primary/main` | `#1976D2` | `#90CAF9` |
| `primary/on-primary` | `#FFFFFF` | `#1A1A1A` |
| `error/main` | `#D32F2F` | `#EF9A9A` |
| `success/main` | `#388E3C` | `#81C784` |

> **Convenção de nomenclatura:** use `/` para criar grupos hierárquicos (surface, text, border, primary, etc.).

#### 3. Aplicar variáveis aos componentes

1. Selecione o elemento (ex: fundo de um card)
2. No campo de cor, clique no ícone de variável (quadrado com losango)
3. Escolha a variável correspondente (ex: `surface/card`)
4. Repita para textos, bordas, ícones, etc.

#### 4. Alternar entre temas

1. Selecione o frame da tela (frame raiz)
2. No painel direito, ao lado da collection `Colors`, clique no seletor de mode
3. Alterne entre `Light` e `Dark`
4. Todos os elementos que usam variáveis mudam automaticamente

### Variáveis de espaçamento e border-radius

Além de cores, crie collections para valores numéricos:

**Collection: Spacing**

| Variável | Mode: Compact | Mode: Comfortable |
|---|---|---|
| `spacing/xs` | 2 | 4 |
| `spacing/sm` | 4 | 8 |
| `spacing/md` | 8 | 16 |
| `spacing/lg` | 16 | 24 |
| `spacing/xl` | 24 | 48 |

**Collection: Radius**

| Variável | Mode: Sharp | Mode: Rounded |
|---|---|---|
| `radius/sm` | 2 | 4 |
| `radius/md` | 4 | 8 |
| `radius/lg` | 8 | 16 |
| `radius/full` | 4 | 9999 |

### Como o tema se traduz para código

#### CSS — Custom Properties com `prefers-color-scheme`

```css
:root {
  --surface-background: #FFFFFF;
  --surface-card: #F5F5F5;
  --text-primary: #1A1A1A;
  --text-secondary: #666666;
  --primary-main: #1976D2;
}

@media (prefers-color-scheme: dark) {
  :root {
    --surface-background: #121212;
    --surface-card: #1E1E1E;
    --text-primary: #E0E0E0;
    --text-secondary: #9E9E9E;
    --primary-main: #90CAF9;
  }
}

/* Ou com classes para toggle manual */
[data-theme="dark"] {
  --surface-background: #121212;
  --surface-card: #1E1E1E;
  --text-primary: #E0E0E0;
  /* ... */
}
```

#### Flutter — ThemeData com light/dark

```dart
final lightTheme = ThemeData(
  brightness: Brightness.light,
  colorScheme: const ColorScheme.light(
    surface: Color(0xFFFFFFFF),
    primary: Color(0xFF1976D2),
    onPrimary: Color(0xFFFFFFFF),
    error: Color(0xFFD32F2F),
  ),
  scaffoldBackgroundColor: const Color(0xFFFFFFFF),
);

final darkTheme = ThemeData(
  brightness: Brightness.dark,
  colorScheme: const ColorScheme.dark(
    surface: Color(0xFF121212),
    primary: Color(0xFF90CAF9),
    onPrimary: Color(0xFF1A1A1A),
    error: Color(0xFFEF9A9A),
  ),
  scaffoldBackgroundColor: const Color(0xFF121212),
);

// No MaterialApp:
MaterialApp(
  theme: lightTheme,
  darkTheme: darkTheme,
  themeMode: ThemeMode.system,  // segue preferência do sistema
);
```

#### React Native — temas com `useColorScheme`

```jsx
import { useColorScheme } from 'react-native';

const themes = {
  light: {
    background: '#FFFFFF',
    card: '#F5F5F5',
    textPrimary: '#1A1A1A',
    textSecondary: '#666666',
    primary: '#1976D2',
  },
  dark: {
    background: '#121212',
    card: '#1E1E1E',
    textPrimary: '#E0E0E0',
    textSecondary: '#9E9E9E',
    primary: '#90CAF9',
  },
};

function App() {
  const colorScheme = useColorScheme(); // 'light' ou 'dark'
  const theme = themes[colorScheme ?? 'light'];

  return (
    <View style={{ backgroundColor: theme.background, flex: 1 }}>
      <Text style={{ color: theme.textPrimary }}>Olá</Text>
    </View>
  );
}
```

### Dicas para temas

- **Nunca use cores absolutas** diretamente nos elementos — sempre referencie variáveis
- **Teste a legibilidade** em ambos os temas (contraste WCAG AA mínimo)
- No dark mode, **não inverta simplesmente** as cores — tons escuros precisam de ajustes específicos de saturação e luminosidade
- **Elevação no dark mode** é representada por superfícies mais claras (ao contrário do light mode que usa sombras)
- Mantenha a **mesma hierarquia visual** em ambos os temas

---

## 11. Acessibilidade no Design

Projetar para acessibilidade desde o Figma garante que o produto final seja utilizável pelo maior número de pessoas, incluindo aquelas com deficiências visuais, motoras, auditivas ou cognitivas.

### Tamanhos mínimos de toque (touch targets)

Elementos interativos devem ter área de toque suficiente para serem acionados sem dificuldade:

| Plataforma | Tamanho mínimo recomendado | Diretriz |
|---|---|---|
| **Android** | 48 × 48 dp | Material Design 3 |
| **iOS** | 44 × 44 pt | Apple HIG |
| **Web** | 44 × 44 px (WCAG 2.5.8) ou 24 × 24 px (WCAG 2.5.5 AA) | WCAG 2.2 |

> **No Figma:** mesmo que o ícone visual tenha 24×24px, o frame do botão deve ter pelo menos 44×44px (ou 48×48px para mobile) com padding adequado.

### Espaçamento entre elementos interativos

| Situação | Espaçamento mínimo |
|---|---|
| Entre botões adjacentes | 8px |
| Entre itens de lista clicáveis | 4-8px (separador ou gap) |
| Padding interno de botões | Mínimo 8px vertical, 16px horizontal |

### Contraste de cores

| Nível WCAG | Ratio mínimo | Aplicação |
|---|---|---|
| **AA (texto normal)** | 4.5:1 | Texto ≤ 18px regular ou ≤ 14px bold |
| **AA (texto grande)** | 3:1 | Texto ≥ 18px regular ou ≥ 14px bold |
| **AA (elementos UI)** | 3:1 | Bordas de inputs, ícones informativos, indicadores de foco |
| **AAA** | 7:1 | Nível mais alto (recomendado para textos longos) |

#### Exemplos de combinações acessíveis

| Texto | Fundo | Ratio | WCAG AA |
|---|---|---|---|
| `#1A1A1A` | `#FFFFFF` | 16.6:1 | Aprovado |
| `#666666` | `#FFFFFF` | 5.7:1 | Aprovado |
| `#999999` | `#FFFFFF` | 2.8:1 | Reprovado |
| `#E0E0E0` | `#121212` | 12.1:1 | Aprovado (dark mode) |
| `#FFFFFF` | `#1976D2` | 4.6:1 | Aprovado |

> **No Figma:** use os plugins **Stark** ou **A11y - Color Contrast Checker** para verificar o contraste em tempo real.

### Hierarquia visual e legibilidade

| Diretriz | Recomendação |
|---|---|
| **Tamanho mínimo de texto** | 16px para body text (web/mobile), nunca menor que 12px |
| **Line height** | 1.4 a 1.6 para body text (facilita a leitura) |
| **Largura de linha** | 50-75 caracteres por linha (ideal ~66) |
| **Peso da fonte** | Mínimo Regular (400) para body; evitar Light (300) em tamanhos pequenos |
| **Espaçamento entre letras** | Não comprimir — manter o padrão ou aumentar levemente |
| **Alinhamento** | Esquerda para textos longos (evitar justificado na web) |

### Estados visuais obrigatórios

Todo elemento interativo deve ter representação visual clara para cada estado:

| Estado | O que comunicar | Como representar no Figma |
|---|---|---|
| **Default** | Estado neutro | Aparência padrão do componente |
| **Hover** | O cursor está sobre o elemento (web) | Mudança de cor de fundo ou borda |
| **Focus** | Elemento selecionado via teclado | Outline visível (2px+, cor contrastante) — nunca remover |
| **Active / Pressed** | Sendo clicado/pressionado | Escurecimento ou redução de tamanho sutil |
| **Disabled** | Indisponível | Opacidade reduzida (40-50%) + cursor not-allowed |
| **Error** | Erro de validação | Borda vermelha + mensagem de erro + ícone |
| **Loading** | Processando | Spinner ou skeleton + desabilitar interação |

> **Importante:** o estado **Focus** é essencial para navegação por teclado. No Figma, crie variantes de foco com um outline visível (ex: `2px solid #1976D2` com offset de 2px).

### Formulários acessíveis

| Diretriz | Como aplicar no Figma |
|---|---|
| **Labels visíveis** | Cada campo deve ter um label acima ou ao lado (nunca depender apenas do placeholder) |
| **Placeholder ≠ Label** | Placeholder desaparece ao digitar — não é substituto do label |
| **Mensagens de erro** | Posicionar abaixo do campo, com ícone + texto vermelho (não depender só da cor) |
| **Campos obrigatórios** | Indicar com asterisco (*) no label + texto explicativo "* campos obrigatórios" |
| **Agrupamento** | Usar `fieldset` visual (borda ou fundo) para agrupar campos relacionados |
| **Ordem lógica** | Organizar campos na ordem natural de preenchimento (cima para baixo, esquerda para direita) |

### Não depender apenas de cor

Informações transmitidas por cor devem ter um indicador adicional:

| Situação | Apenas cor (inacessível) | Cor + indicador (acessível) |
|---|---|---|
| **Status** | Círculo verde / vermelho | Círculo com cor + texto "Ativo" / "Inativo" |
| **Erro em campo** | Borda vermelha | Borda vermelha + ícone de alerta + mensagem textual |
| **Link no texto** | Texto azul | Texto azul + sublinhado |
| **Gráficos** | Linhas de cores diferentes | Cores + padrões (tracejado, pontilhado) + legenda |

### Ordem de leitura e navegação

No Figma, a ordem dos layers no painel esquerdo define a ordem de leitura para tecnologias assistivas e navegação por teclado:

1. **Organize os layers de cima para baixo** na ordem em que devem ser lidos
2. **Nomeie os layers** de forma descritiva (ex: `Botão Enviar`, não `Rectangle 47`)
3. **Marque elementos decorativos** que devem ser ignorados por leitores de tela (no código: `aria-hidden="true"` ou `decorative` no Flutter)

### Checklist de acessibilidade no Figma

- [ ] Touch targets ≥ 44×44px (web) ou 48×48dp (mobile)
- [ ] Contraste de texto ≥ 4.5:1 (AA) para texto normal
- [ ] Contraste de elementos UI ≥ 3:1 (bordas, ícones)
- [ ] Todos os inputs têm labels visíveis (não apenas placeholder)
- [ ] Estados de focus visíveis em todos os elementos interativos
- [ ] Informações não dependem exclusivamente de cor
- [ ] Tamanho de texto body ≥ 16px
- [ ] Mensagens de erro têm ícone + texto (não apenas cor vermelha)
- [ ] Ordem dos layers reflete a ordem de leitura lógica
- [ ] Alternativa textual planejada para imagens informativas

### Plugins de acessibilidade para o Figma

| Plugin | Função |
|---|---|
| **Stark** | Verificação completa: contraste, simulação de daltonismo, ordem de foco, touch targets |
| **A11y - Color Contrast Checker** | Verificador de contraste WCAG rápido |
| **Color Blind** | Simula como daltônicos veem o design (protanopia, deuteranopia, tritanopia) |
| **Axe for Designers** | Identifica problemas de acessibilidade em componentes |
| **Include — Accessibility Annotations** | Adiciona anotações de acessibilidade para handoff |

---

## 12. Design Responsivo — Estratégias

### Abordagem Mobile-First no Figma

1. **Comece pelo frame Mobile** (375px ou 393px)
2. Defina toda a hierarquia de conteúdo e navegação para mobile
3. **Expanda para tablet** (768px): reorganize elementos, crie grids de 2 colunas
4. **Expanda para desktop** (1440px): aproveite o espaço com grids de 3-4 colunas, sidebars

### Padrões de adaptação responsiva

| Padrão | Mobile | Tablet | Desktop |
|---|---|---|---|
| **Menu** | Hambúrguer + drawer | Hambúrguer ou tabs | Links horizontais completos |
| **Grid de cards** | 1 coluna | 2 colunas | 3-4 colunas |
| **Hero** | Empilhado (imagem acima/abaixo do texto) | Empilhado ou lado a lado | Lado a lado (50/50) |
| **Sidebar** | Oculta (drawer) | Colapsada (ícones) | Expandida |
| **Tabela** | Cards empilhados ou scroll horizontal | Tabela compacta | Tabela completa |
| **Formulário** | Full-width, campos empilhados | 2 colunas em seções | 2-3 colunas |
| **Footer** | Links empilhados | 2-3 colunas | 4+ colunas |
| **Imagens** | Full-width | Contidas no grid | Contidas no grid |

### Sections (Seções) — recurso nativo do Figma

O Figma possui o conceito de **Sections** para organizar visualmente frames relacionados:

1. Pressione **Shift + S** ou arraste na toolbar para criar uma Section
2. Agrupe frames de um mesmo fluxo ou página
3. Nomeie: `Home — Mobile`, `Home — Tablet`, `Home — Desktop`

### Dicas práticas

- Use **Components** com variantes responsivas (ex: `Card/Mobile`, `Card/Desktop`)
- Mantenha os **mesmos nomes de layers** entre as versões mobile/tablet/desktop para facilitar o handoff
- Use **Min/Max width** nos Auto Layouts para controlar como os elementos se comportam em redimensionamento
- Teste redimensionando os frames manualmente para verificar se o Auto Layout se comporta corretamente
- Crie **breakpoints side-by-side** para visualizar todas as versões simultaneamente

---

## 13. Prototipagem e Interações

### Criar fluxo de navegação

1. Mude para a aba **Prototype** no painel direito
2. Selecione um elemento interativo (botão, card, link)
3. Arraste a seta azul para o frame de destino
4. Configure a interação:

| Propriedade | Opções comuns |
|---|---|
| **Trigger** | On Click, While Hovering, Mouse Enter/Leave, After Delay |
| **Action** | Navigate To, Open Overlay, Swap Overlay, Back, Scroll To, Open Link |
| **Animation** | Instant, Dissolve, Move In/Out, Push, Slide In/Out, Smart Animate |
| **Duration** | 150-300ms (recomendado) |
| **Easing** | Ease In Out (padrão), Spring |

### Smart Animate

**Smart Animate** interpola automaticamente entre dois frames quando os layers têm o mesmo nome:

1. Duplique o frame
2. Modifique posição, tamanho, cor ou opacidade dos elementos (mantenha os mesmos nomes)
3. Conecte os frames com **Smart Animate**
4. O Figma faz a transição suave automaticamente

### Overlays (modais, bottom sheets, menus)

1. Crie o conteúdo do overlay em um frame separado
2. Na conexão de protótipo, escolha **Open Overlay**
3. Configure:
   - **Position:** Centered, Top, Bottom (para bottom sheets)
   - **Background:** adicione overlay escuro (`#000000` 50% opacidade)
   - **Close on click outside:** ative para fechar ao clicar fora

### Testar o protótipo

- Pressione **Play** (▶) no canto superior direito ou **Ctrl/Cmd + Enter** na barra de ferramentas
- No mobile: use o app **Figma Mirror** (iOS/Android) para testar no dispositivo real
- Compartilhe o link do protótipo para testes com usuários

---

## 14. Colaboração e Organização de Projeto

Manter o arquivo Figma organizado facilita a colaboração entre designers e desenvolvedores, evita retrabalho e acelera o handoff.

### Estrutura de páginas (Pages)

Organize o arquivo em páginas separadas por função:

| Página | Conteúdo |
|---|---|
| **Cover** | Capa do projeto com título, status, data, responsáveis |
| **Wireframes** | Rascunhos de baixa fidelidade |
| **Design** | Telas de alta fidelidade (versão atual) |
| **Components** | Biblioteca de componentes e design system local |
| **Protótipo** | Frames conectados com fluxos interativos |
| **Archive** | Versões antigas ou descartadas (não deletar — mover para cá) |
| **Specs / Handoff** | Anotações e especificações para desenvolvimento |

### Convenções de nomenclatura

#### Layers e frames

| Tipo | Convenção | Exemplo |
|---|---|---|
| **Frames (telas)** | `NomeDaTela / Breakpoint` | `Home / Desktop`, `Home / Mobile` |
| **Sections** | `NomeDaTela — Descrição` | `Home — Hero`, `Home — Features` |
| **Componentes** | `Categoria/Nome/Variante` | `Button/Primary/Default` |
| **Layers internos** | PascalCase ou camelCase descritivo | `CardTitle`, `NavLinks`, `SearchInput` |
| **Ícones** | `icon/nome` | `icon/search`, `icon/menu` |

#### Regras gerais

- **Nunca** deixar layers com nomes padrão (`Frame 47`, `Rectangle 123`, `Group 5`)
- Renomear com **Ctrl + R** assim que criar o elemento
- Usar **nomes em inglês** para layers (facilita o handoff internacional e mapeamento para código)
- Agrupar layers relacionados em **frames nomeados**, não em groups genéricos

### Organização visual do canvas

```
┌─ Page: Design ──────────────────────────────────────────┐
│                                                          │
│  [Section: Home]                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Mobile   │  │  Tablet  │  │  Desktop │              │
│  │  375×812  │  │  768×1024│  │ 1440×1024│              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
│  [Section: Login]                                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Mobile   │  │  Tablet  │  │  Desktop │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

- Use **Sections** (**Shift + S**) para agrupar frames de uma mesma tela/fluxo
- Mantenha os breakpoints **lado a lado** na ordem Mobile → Tablet → Desktop
- Deixe espaçamento consistente entre sections (200-400px)
- Fluxos de navegação seguem da esquerda para a direita

### Branches (ramificações)

O Figma suporta **Branches** para trabalho paralelo sem afetar o arquivo principal (similar ao Git):

1. No menu do arquivo, clique em **Create branch**
2. Nomeie a branch de forma descritiva (ex: `feature/novo-checkout`, `fix/cores-dark-mode`)
3. Trabalhe na branch sem afetar o arquivo principal (Main)
4. Quando finalizar, clique em **Review and merge** para integrar as mudanças

| Situação | Usar branch? |
|---|---|
| Explorar alternativa de layout | Sim |
| Redesign de uma seção inteira | Sim |
| Ajuste pequeno de cor ou texto | Não (editar direto no Main) |
| Trabalho simultâneo de múltiplos designers | Sim (uma branch por designer/feature) |

> **Observação:** Branches estão disponíveis nos planos **Professional** e superiores do Figma.

### Comentários e revisões

| Recurso | Como usar | Atalho |
|---|---|---|
| **Comentários** | Clique em **C** e clique no ponto do design para adicionar um comentário | **C** |
| **Mencionar pessoas** | Use `@nome` dentro do comentário para notificar | — |
| **Resolver comentário** | Marque como resolvido após implementar o feedback | — |
| **Threads** | Responda a comentários para manter a discussão organizada | — |

#### Boas práticas para revisões

- Numere os rounds de revisão: `[R1]`, `[R2]`, `[R3]` no comentário
- Marque comentários como **resolvidos** após aplicar o feedback (não deletar)
- Use comentários no frame específico, não genéricos no canvas
- Para feedback de design, anexe referências visuais quando possível

### Team Libraries (Bibliotecas compartilhadas)

Componentes e estilos podem ser publicados como **Team Library** para uso em múltiplos arquivos:

1. Na página de componentes, selecione os componentes finalizados
2. Menu **Assets** → **Publish Library** (ou ícone de livro no painel Assets)
3. Adicione descrição das mudanças (changelog)
4. Outros arquivos do time podem ativar a biblioteca via **Assets → Libraries**

| Vantagem | Descrição |
|---|---|
| **Consistência** | Todos os arquivos usam os mesmos componentes |
| **Atualização centralizada** | Alterar o componente na biblioteca atualiza em todos os arquivos |
| **Changelog** | Histórico de mudanças publicadas |

### Versionamento e histórico

O Figma mantém **histórico automático** de versões:

- **Autosave:** salva automaticamente a cada mudança
- **Histórico de versões:** acessível em **File → Show Version History** (ou **Ctrl + Alt + H**)
- **Salvar versão nomeada:** para marcos importantes, clique em **Save to Version History** e adicione título + descrição (ex: `v1.0 — Design aprovado pelo cliente`)
- **Restaurar versão:** é possível voltar a qualquer ponto do histórico

### Permissões e compartilhamento

| Permissão | O que pode fazer |
|---|---|
| **Can view** | Visualizar, comentar, inspecionar (Dev Mode) |
| **Can edit** | Tudo acima + editar o design |
| **Owner** | Tudo acima + gerenciar permissões e excluir |

#### Recomendações de permissão

| Papel | Permissão sugerida |
|---|---|
| **Designer principal** | Can edit (Owner) |
| **Designers do time** | Can edit |
| **Desenvolvedores** | Can view (Dev Mode é suficiente para o handoff) |
| **Stakeholders / PO** | Can view (para revisão e comentários) |

---

## 15. Handoff para Desenvolvedores

### Dev Mode

O **Dev Mode** do Figma exibe informações prontas para desenvolvimento:

- Código CSS gerado automaticamente para cada elemento
- Espaçamentos entre elementos (distâncias em px)
- Cores em hex, RGB, HSL
- Tipografia com todas as propriedades
- Assets exportáveis (ícones, imagens)

### Como ativar

1. Alterne para **Dev Mode** clicando no botão `</>` na barra superior
2. Clique em qualquer elemento para ver as propriedades
3. Copie o código CSS diretamente

### Exportar assets

1. Selecione o elemento (ícone, imagem, ilustração)
2. No painel direito (Design), clique em **+** ao lado de **Export**
3. Configure:

| Formato | Uso |
|---|---|
| **SVG** | Ícones e ilustrações vetoriais (preferível) |
| **PNG** | Imagens com transparência, screenshots |
| **JPG** | Fotografias, imagens sem transparência |
| **PDF** | Impressão ou documentação |

4. Para telas de alta resolução, exporte em múltiplas escalas:
   - **1x** — telas padrão
   - **2x** — telas Retina / high-DPI
   - **3x** — iPhones Plus/Max

### Dicas para o handoff

- Mantenha os layers **nomeados e organizados** (o dev lê esses nomes)
- Use **Auto Layout** que traduz diretamente para Flexbox/CSS Grid
- Defina **Design Tokens** ou **Variables** que podem ser exportados como variáveis CSS/SCSS
- Adicione notas em frames usando **Comments** (atalho: **C**) para instruções especiais
- Exporte o **style guide** como referência visual (página separada com todos os componentes)

---

## 16. Do Figma para Código — Web (HTML/CSS)

### Mapeamento visual: Figma → HTML/CSS

A tradução de um layout Figma para código web segue um mapeamento direto entre os conceitos do Figma e as tecnologias CSS modernas.

#### Estrutura (Frames → Tags HTML)

| Elemento no Figma | Tag HTML equivalente |
|---|---|
| Frame principal (tela) | `<body>` ou `<main>` |
| Frame de seção | `<section>`, `<header>`, `<footer>`, `<nav>`, `<aside>` |
| Frame genérico | `<div>` |
| Text | `<h1>`–`<h6>`, `<p>`, `<span>` |
| Imagem (Fill Image) | `<img>` |
| Componente botão | `<button>` ou `<a>` |
| Componente input | `<input>`, `<textarea>`, `<select>` |
| Componente link | `<a>` |
| Lista de itens | `<ul>` + `<li>` ou `<ol>` + `<li>` |

#### Auto Layout → Flexbox / CSS Grid

| Propriedade Figma | CSS Flexbox | Exemplo |
|---|---|---|
| Direction: Horizontal | `flex-direction: row` | Layout lado a lado |
| Direction: Vertical | `flex-direction: column` | Itens empilhados |
| Gap: 16 | `gap: 16px` | Espaço entre itens |
| Padding: 24 | `padding: 24px` | Espaço interno |
| Alignment: Center | `align-items: center; justify-content: center` | Centralizar |
| Wrap | `flex-wrap: wrap` | Quebra de linha |
| Child: Fill | `flex: 1` ou `width: 100%` | Preencher espaço |
| Child: Hug | `width: fit-content` | Ajustar ao conteúdo |
| Child: Fixed (200px) | `width: 200px` | Tamanho fixo |

#### Exemplo prático: Card do Figma para HTML/CSS

**No Figma:** Frame com Auto Layout Vertical, padding 16px, gap 12px, border-radius 12px, sombra.

```html
<div class="card">
  <img src="produto.jpg" alt="Produto" class="card__image">
  <h3 class="card__title">Nome do Produto</h3>
  <p class="card__description">Descrição breve do produto.</p>
  <button class="card__button">Comprar</button>
</div>
```

```css
.card {
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 12px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  background: #ffffff;
}

.card__image {
  width: 100%;          /* Fill no Figma */
  height: 200px;        /* Fixed height no Figma */
  object-fit: cover;
  border-radius: 8px;
}

.card__title {
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

.card__description {
  font-size: 14px;
  color: #666666;
  line-height: 1.5;
}

.card__button {
  align-self: flex-end;  /* Alignment: Right no Figma */
  padding: 10px 20px;
  background: #1976d2;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
```

#### Grid de cards responsivo (Auto Layout com Wrap)

**No Figma:** Frame com Auto Layout Horizontal, Wrap ativado, gap 24px.

```css
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
  padding: 48px 80px;
}

.card {
  flex: 1 1 300px;    /* min-width do Figma */
  max-width: 400px;
}

/* Alternativa com CSS Grid */
.card-grid-alt {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
  padding: 48px 80px;
}
```

### Layout responsivo com Media Queries

No Figma, criamos frames separados para cada breakpoint. No CSS, usamos **media queries** para adaptar o layout:

```css
/* Mobile First — estilos base para mobile */
.hero {
  display: flex;
  flex-direction: column;
  padding: 24px 16px;
  gap: 24px;
}

.hero__image {
  width: 100%;
  order: -1;           /* Imagem acima do texto no mobile */
}

/* Tablet */
@media (min-width: 768px) {
  .hero {
    padding: 48px 32px;
    gap: 32px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .hero {
    flex-direction: row;   /* Lado a lado no desktop */
    padding: 80px;
    gap: 48px;
  }

  .hero__text,
  .hero__image {
    flex: 1;               /* 50/50 no desktop */
  }

  .hero__image {
    order: 0;
  }
}
```

### Constraints → CSS Positioning

| Constraint no Figma | CSS equivalente |
|---|---|
| Left + Top (padrão) | `position: absolute; left: Npx; top: Npx;` |
| Right + Bottom | `position: absolute; right: Npx; bottom: Npx;` |
| Left and Right | `position: absolute; left: Npx; right: Npx;` (estica) |
| Center | `position: absolute; left: 50%; transform: translateX(-50%);` |
| Fixed (navbar/footer) | `position: fixed; top: 0; left: 0; right: 0;` |

### Design Tokens / Variables → Variáveis CSS

Os estilos e variáveis do Figma traduzem diretamente para **CSS Custom Properties**:

```css
:root {
  /* Cores — correspondem aos Color Styles do Figma */
  --color-primary-500: #1976D2;
  --color-primary-700: #1565C0;
  --color-neutral-50: #FAFAFA;
  --color-neutral-900: #1A1A1A;
  --color-error-500: #D32F2F;

  /* Tipografia — correspondem aos Text Styles do Figma */
  --font-family: 'Inter', sans-serif;
  --font-size-h1: 36px;
  --font-size-body: 16px;
  --font-size-caption: 12px;

  /* Espaçamentos — escala do Figma */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 48px;

  /* Border radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;

  /* Sombras — correspondem aos Effect Styles do Figma */
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.15);
}

/* Uso */
.card {
  padding: var(--spacing-md);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  font-family: var(--font-family);
}
```

### Roteiro completo: Figma → HTML/CSS

1. **Analisar a estrutura do layout no Figma**
   - Identificar a hierarquia de frames (seções, containers, componentes)
   - Anotar quais frames usam Auto Layout e quais usam Constraints
   - Mapear as tags HTML semânticas para cada frame

2. **Extrair os Design Tokens**
   - No Dev Mode, copiar os valores de cores, tipografia, espaçamentos
   - Criar as variáveis CSS (`:root { ... }`)
   - Se usar SCSS/Tailwind, mapear para as variáveis do preprocessador/framework

3. **Montar o HTML semântico**
   - Criar a estrutura de tags seguindo a hierarquia do Figma
   - Usar tags semânticas (`header`, `main`, `section`, `footer`, `nav`, `article`)
   - Adicionar classes BEM ou a convenção do projeto

4. **Estilizar com CSS (Mobile First)**
   - Começar pelo layout mobile (estilos base)
   - Usar `display: flex` para traduzir Auto Layouts
   - Aplicar `gap`, `padding`, `border-radius`, `box-shadow` conforme o Figma
   - Adicionar media queries para tablet e desktop

5. **Exportar e integrar assets**
   - Exportar ícones como **SVG** (preferível) ou usar icon fonts
   - Exportar imagens em **2x** para telas de alta resolução
   - Otimizar imagens com ferramentas como TinyPNG ou Squoosh

6. **Comparar com o Figma**
   - Usar a extensão **PerfectPixel** no navegador para sobrepor o design
   - Verificar espaçamentos, cores e tipografia no Dev Mode
   - Testar responsividade redimensionando o navegador

### Ferramentas e plugins para exportação Web

| Ferramenta | Descrição |
|---|---|
| **Dev Mode (nativo)** | Gera CSS para cada elemento selecionado |
| **Figma to HTML (plugin)** | Exporta frames como HTML/CSS |
| **Locofy** | Converte designs Figma para código React, HTML, Next.js |
| **Anima** | Exporta Figma para HTML/CSS/React com interações |
| **Inspect (Dev Mode)** | Mostra propriedades CSS, espaçamentos, assets |

---

## 17. Do Figma para Código — Flutter

### Mapeamento visual: Figma → Flutter Widgets

A arquitetura do Flutter é baseada em **widgets compostos em árvore**, o que se alinha naturalmente com a hierarquia de frames do Figma.

#### Estrutura (Frames → Widgets)

| Elemento no Figma | Widget Flutter |
|---|---|
| Frame (Auto Layout Vertical) | `Column` |
| Frame (Auto Layout Horizontal) | `Row` |
| Frame (Auto Layout + Wrap) | `Wrap` |
| Frame (sem Auto Layout) | `Stack` (posicionamento livre) |
| Frame com scroll | `ListView` ou `SingleChildScrollView` |
| Text | `Text` |
| Imagem | `Image.asset` / `Image.network` |
| Retângulo com borda arredondada | `Container` com `BoxDecoration` |
| Componente | `StatelessWidget` / `StatefulWidget` customizado |
| Ícone | `Icon` |
| Botão | `ElevatedButton`, `TextButton`, `OutlinedButton`, `IconButton` |
| Input | `TextField` com `InputDecoration` |

#### Auto Layout → Row / Column / Wrap

| Propriedade Figma | Flutter equivalente |
|---|---|
| Direction: Vertical | `Column(children: [...])` |
| Direction: Horizontal | `Row(children: [...])` |
| Gap: 16 | Não nativo — usar `SizedBox(height: 16)` entre itens ou pacote `gap` |
| Padding: 24 | `Padding(padding: EdgeInsets.all(24), child: ...)` |
| Alignment: Center, Center | `mainAxisAlignment: MainAxisAlignment.center, crossAxisAlignment: CrossAxisAlignment.center` |
| Wrap | `Wrap(spacing: 16, runSpacing: 16, children: [...])` |
| Child: Fill | `Expanded(child: ...)` |
| Child: Hug | Widget sem Expanded (tamanho natural) |
| Child: Fixed (200px) | `SizedBox(width: 200, child: ...)` |

#### Alinhamentos — Figma vs Flutter

| Figma Alignment | `mainAxisAlignment` (direção principal) | `crossAxisAlignment` (perpendicular) |
|---|---|---|
| Top-Left | `MainAxisAlignment.start` | `CrossAxisAlignment.start` |
| Top-Center | `MainAxisAlignment.start` | `CrossAxisAlignment.center` |
| Center-Center | `MainAxisAlignment.center` | `CrossAxisAlignment.center` |
| Space Between | `MainAxisAlignment.spaceBetween` | — |
| Bottom-Right | `MainAxisAlignment.end` | `CrossAxisAlignment.end` |

### Exemplo prático: Card do Figma para Flutter

**No Figma:** Frame com Auto Layout Vertical, padding 16px, gap 12px, border-radius 12px, sombra.

```dart
class ProductCard extends StatelessWidget {
  final String imageUrl;
  final String title;
  final String description;

  const ProductCard({
    super.key,
    required this.imageUrl,
    required this.title,
    required this.description,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),  // border-radius do Figma
        boxShadow: [
          BoxShadow(
            color: Colors.black.withValues(alpha: 0.1),  // sombra do Figma
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Padding(
        padding: const EdgeInsets.all(16),  // padding do Figma
        child: Column(                       // Auto Layout Vertical
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                imageUrl,
                width: double.infinity,        // Fill no Figma
                height: 200,                   // Fixed height no Figma
                fit: BoxFit.cover,
              ),
            ),
            const SizedBox(height: 12),        // gap do Figma
            Text(
              title,
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.w600,
                color: Color(0xFF1A1A1A),
              ),
            ),
            const SizedBox(height: 12),        // gap do Figma
            Text(
              description,
              style: const TextStyle(
                fontSize: 14,
                color: Color(0xFF666666),
                height: 1.5,                   // line-height do Figma
              ),
            ),
            const SizedBox(height: 12),        // gap do Figma
            Align(
              alignment: Alignment.centerRight, // Alignment right no Figma
              child: ElevatedButton(
                onPressed: () {},
                style: ElevatedButton.styleFrom(
                  backgroundColor: const Color(0xFF1976D2),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                  padding: const EdgeInsets.symmetric(
                    horizontal: 20,
                    vertical: 10,
                  ),
                ),
                child: const Text('Comprar'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Layout responsivo no Flutter

No Flutter, a responsividade é tratada de forma diferente do CSS — usa-se `MediaQuery` e `LayoutBuilder`:

```dart
class ResponsiveCardGrid extends StatelessWidget {
  const ResponsiveCardGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Define colunas baseado na largura disponível (como breakpoints do Figma)
        int crossAxisCount;
        if (constraints.maxWidth < 600) {
          crossAxisCount = 1;       // Mobile: 1 coluna
        } else if (constraints.maxWidth < 900) {
          crossAxisCount = 2;       // Tablet: 2 colunas
        } else {
          crossAxisCount = 3;       // Desktop: 3 colunas
        }

        return GridView.builder(
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: 24,    // gap horizontal do Figma
            mainAxisSpacing: 24,     // gap vertical do Figma
            childAspectRatio: 0.85,
          ),
          itemCount: 6,
          itemBuilder: (context, index) {
            return ProductCard(
              imageUrl: 'https://exemplo.com/img$index.jpg',
              title: 'Produto $index',
              description: 'Descrição do produto.',
            );
          },
        );
      },
    );
  }
}
```

### Constraints (Figma) → Posicionamento no Flutter

| Constraint no Figma | Flutter equivalente |
|---|---|
| Posição absoluta (sem Auto Layout) | `Stack` + `Positioned` |
| Left: 16, Top: 16 | `Positioned(left: 16, top: 16, child: ...)` |
| Right: 16, Bottom: 16 | `Positioned(right: 16, bottom: 16, child: ...)` |
| Left and Right (estica) | `Positioned(left: 16, right: 16, child: ...)` |
| Center | `Positioned.fill(child: Center(child: ...))` |
| Fixed bottom (Bottom Nav) | `Scaffold(bottomNavigationBar: ...)` |
| FAB (Right + Bottom) | `Scaffold(floatingActionButton: ...)` |

### Design Tokens → ThemeData

As variáveis e estilos do Figma mapeiam para o **ThemeData** do Flutter:

```dart
final appTheme = ThemeData(
  // Cores — Color Styles do Figma
  colorScheme: ColorScheme.fromSeed(
    seedColor: const Color(0xFF1976D2),
    primary: const Color(0xFF1976D2),
    secondary: const Color(0xFF26A69A),
    error: const Color(0xFFD32F2F),
    surface: const Color(0xFFFFFFFF),
  ),

  // Tipografia — Text Styles do Figma
  textTheme: const TextTheme(
    displayLarge: TextStyle(fontSize: 56, fontWeight: FontWeight.w700),
    headlineLarge: TextStyle(fontSize: 36, fontWeight: FontWeight.w700),
    headlineMedium: TextStyle(fontSize: 28, fontWeight: FontWeight.w600),
    headlineSmall: TextStyle(fontSize: 22, fontWeight: FontWeight.w600),
    bodyLarge: TextStyle(fontSize: 16, fontWeight: FontWeight.w400, height: 1.5),
    bodyMedium: TextStyle(fontSize: 14, fontWeight: FontWeight.w400, height: 1.5),
    labelSmall: TextStyle(fontSize: 12, fontWeight: FontWeight.w400),
  ),

  // Espaçamentos podem ser definidos como extensão do tema
  // ou como constantes separadas
);

// Constantes de espaçamento (escala do Figma)
class AppSpacing {
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 16;
  static const double lg = 24;
  static const double xl = 48;
}

// Constantes de border radius
class AppRadius {
  static const double sm = 4;
  static const double md = 8;
  static const double lg = 16;
}
```

### Exemplo: tela completa — App Mobile (Figma → Flutter)

**No Figma:** tela com App Bar, lista de itens scrollável e Bottom Navigation.

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // App Bar — corresponde ao frame "App Bar" no Figma
      appBar: AppBar(
        leading: const Icon(Icons.menu),        // ícone menu do Figma
        title: const Text('Meu App'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {},
          ),
          IconButton(
            icon: const Icon(Icons.notifications_outlined),
            onPressed: () {},
          ),
        ],
      ),

      // Conteúdo scrollável — Auto Layout Vertical com scroll
      body: ListView(
        padding: const EdgeInsets.all(16),        // padding do frame
        children: [
          // Search Bar
          TextField(
            decoration: InputDecoration(
              hintText: 'Buscar...',
              prefixIcon: const Icon(Icons.search),
              border: OutlineInputBorder(
                borderRadius: BorderRadius.circular(12),
              ),
              filled: true,
              fillColor: Colors.grey[100],
            ),
          ),
          const SizedBox(height: 24),

          // Título de seção
          Text(
            'Destaques',
            style: Theme.of(context).textTheme.headlineSmall,
          ),
          const SizedBox(height: 12),

          // Lista de itens — cada "Item" é um frame no Figma
          ...List.generate(5, (index) => Padding(
            padding: const EdgeInsets.only(bottom: 12), // gap do Figma
            child: ListTile(
              leading: Container(                        // Thumbnail 48×48
                width: 48, height: 48,
                decoration: BoxDecoration(
                  color: Colors.grey[300],
                  borderRadius: BorderRadius.circular(8),
                ),
              ),
              title: Text('Item ${index + 1}'),
              subtitle: const Text('Descrição do item'),
              trailing: const Icon(Icons.chevron_right),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(12),
              ),
              tileColor: Colors.white,
            ),
          )),
        ],
      ),

      // Bottom Navigation — frame fixo na base do Figma
      bottomNavigationBar: NavigationBar(
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Home'),
          NavigationDestination(icon: Icon(Icons.explore_outlined), selectedIcon: Icon(Icons.explore), label: 'Explorar'),
          NavigationDestination(icon: Icon(Icons.favorite_outline), selectedIcon: Icon(Icons.favorite), label: 'Favoritos'),
          NavigationDestination(icon: Icon(Icons.person_outline), selectedIcon: Icon(Icons.person), label: 'Perfil'),
        ],
      ),

      // FAB — constraint Right + Bottom no Figma
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Ferramentas para conversão Figma → Flutter

| Ferramenta | Descrição |
|---|---|
| **Figma Dev Mode** | Inspecionar propriedades — traduzir manualmente para widgets |
| **FlutterFlow** | Ferramenta visual que importa do Figma e gera código Flutter |
| **Widgetbook** | Catálogo de widgets Flutter (como Storybook para Flutter) |
| **Figma to Flutter (plugin)** | Gera código Dart básico a partir de elementos selecionados |
| **Parabeac** | Converte designs Figma para código Flutter |

### Dicas para a conversão

- **Sempre use Auto Layout no Figma** — traduz diretamente para `Row`, `Column` e `Wrap`
- **Nomeie os layers** com nomes que façam sentido como nomes de widgets (ex: `ProductCard`, `HeaderSection`)
- **Componentes do Figma = Widgets reutilizáveis** — cada componente Figma deve virar um `StatelessWidget` ou `StatefulWidget`
- **Variantes do Figma ≈ parâmetros do widget** — propriedades como `Type: Primary/Secondary` viram parâmetros (`bool isPrimary`)
- Prefira usar **Material 3 widgets** (`NavigationBar`, `SearchBar`, `FilledButton`) que já seguem o design system do Figma Material Kit
- Use o **pacote `gap`** para simular o `gap` do Auto Layout sem precisar de `SizedBox` em todo lugar

---

## 18. Do Figma para Código — React Native

### Mapeamento visual: Figma → Componentes React Native

O React Native utiliza **Flexbox** por padrão em todos os componentes, com `flexDirection: 'column'` como valor inicial — o que se alinha naturalmente ao Auto Layout vertical do Figma.

#### Estrutura (Frames → Componentes)

| Elemento no Figma | Componente React Native |
|---|---|
| Frame (Auto Layout Vertical) | `<View style={{ flexDirection: 'column' }}>` |
| Frame (Auto Layout Horizontal) | `<View style={{ flexDirection: 'row' }}>` |
| Frame (Auto Layout + Wrap) | `<View style={{ flexDirection: 'row', flexWrap: 'wrap' }}>` |
| Frame (sem Auto Layout) | `<View>` com posicionamento absoluto |
| Frame com scroll | `<ScrollView>` ou `<FlatList>` |
| Text | `<Text>` |
| Imagem | `<Image source={...}>` |
| Retângulo / Container | `<View>` com estilos |
| Componente | Componente funcional customizado |
| Ícone | Componente de biblioteca de ícones (ex: `react-native-vector-icons`) |
| Botão | `<Pressable>`, `<TouchableOpacity>`, ou `<Button>` |
| Input | `<TextInput>` |

#### Auto Layout → Flexbox (React Native)

| Propriedade Figma | React Native StyleSheet |
|---|---|
| Direction: Vertical | `flexDirection: 'column'` (padrão — pode omitir) |
| Direction: Horizontal | `flexDirection: 'row'` |
| Gap: 16 | `gap: 16` |
| Padding: 24 | `padding: 24` |
| Alignment: Center (main axis) | `justifyContent: 'center'` |
| Alignment: Center (cross axis) | `alignItems: 'center'` |
| Wrap | `flexWrap: 'wrap'` |
| Child: Fill | `flex: 1` |
| Child: Hug | sem `flex` (tamanho natural) |
| Child: Fixed (200px) | `width: 200` ou `height: 200` |

> **Diferença importante:** No React Native, a unidade padrão é **dp** (density-independent pixels), não `px`. Os valores numéricos do Figma podem ser usados diretamente sem sufixo.

### Exemplo prático: Card do Figma para React Native

**No Figma:** Frame com Auto Layout Vertical, padding 16px, gap 12px, border-radius 12px, sombra.

```jsx
import { View, Text, Image, Pressable, StyleSheet } from 'react-native';

function ProductCard({ imageUrl, title, description, onPress }) {
  return (
    <View style={styles.card}>
      <Image
        source={{ uri: imageUrl }}
        style={styles.cardImage}
        resizeMode="cover"
      />
      <Text style={styles.cardTitle}>{title}</Text>
      <Text style={styles.cardDescription}>{description}</Text>
      <Pressable style={styles.cardButton} onPress={onPress}>
        <Text style={styles.cardButtonText}>Comprar</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#FFFFFF',
    borderRadius: 12,                  // border-radius do Figma
    padding: 16,                       // padding do Figma
    gap: 12,                           // gap do Auto Layout
    // Sombra (iOS)
    shadowColor: '#000000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    // Sombra (Android)
    elevation: 3,
  },
  cardImage: {
    width: '100%',                     // Fill no Figma
    height: 200,                       // Fixed height no Figma
    borderRadius: 8,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#1A1A1A',
  },
  cardDescription: {
    fontSize: 14,
    color: '#666666',
    lineHeight: 21,                    // line-height: 1.5 × 14px
  },
  cardButton: {
    alignSelf: 'flex-end',             // Alignment right no Figma
    backgroundColor: '#1976D2',
    borderRadius: 8,
    paddingHorizontal: 20,
    paddingVertical: 10,
  },
  cardButtonText: {
    color: '#FFFFFF',
    fontSize: 14,
    fontWeight: '600',
  },
});
```

### Layout responsivo no React Native

React Native usa `Dimensions` e `useWindowDimensions` para adaptar layouts:

```jsx
import { View, FlatList, useWindowDimensions, StyleSheet } from 'react-native';

function ResponsiveCardGrid({ products }) {
  const { width } = useWindowDimensions();

  // Define colunas baseado na largura (como breakpoints do Figma)
  const numColumns = width < 600 ? 1 : width < 900 ? 2 : 3;

  return (
    <FlatList
      data={products}
      key={numColumns}   // força re-render ao mudar colunas
      numColumns={numColumns}
      contentContainerStyle={styles.grid}
      columnWrapperStyle={numColumns > 1 ? styles.row : undefined}
      renderItem={({ item }) => (
        <View style={[styles.cardWrapper, { flex: 1 / numColumns }]}>
          <ProductCard
            imageUrl={item.imageUrl}
            title={item.title}
            description={item.description}
            onPress={() => {}}
          />
        </View>
      )}
      keyExtractor={(item) => item.id}
    />
  );
}

const styles = StyleSheet.create({
  grid: {
    padding: 16,
    gap: 16,
  },
  row: {
    gap: 16,
  },
  cardWrapper: {
    minWidth: 280,
  },
});
```

### Constraints (Figma) → Posicionamento absoluto no React Native

| Constraint no Figma | React Native equivalente |
|---|---|
| Frame sem Auto Layout | `<View>` container (já é `position: 'relative'` por padrão) |
| Left: 16, Top: 16 | `position: 'absolute', left: 16, top: 16` |
| Right: 16, Bottom: 16 | `position: 'absolute', right: 16, bottom: 16` |
| Left and Right | `position: 'absolute', left: 16, right: 16` |
| Center | `alignSelf: 'center'` ou `position: 'absolute'` com cálculo |
| FAB (Right + Bottom) | `position: 'absolute', right: 16, bottom: 16` |

### Design Tokens → Constantes / Theme

```jsx
// theme.js — corresponde aos Styles e Variables do Figma
export const colors = {
  primary: {
    500: '#1976D2',
    700: '#1565C0',
  },
  neutral: {
    50: '#FAFAFA',
    100: '#F5F5F5',
    200: '#EEEEEE',
    500: '#9E9E9E',
    700: '#616161',
    900: '#1A1A1A',
  },
  error: {
    500: '#D32F2F',
  },
  success: {
    500: '#388E3C',
  },
};

export const typography = {
  h1: { fontSize: 36, fontWeight: '700', lineHeight: 43 },
  h2: { fontSize: 28, fontWeight: '600', lineHeight: 36 },
  h3: { fontSize: 22, fontWeight: '600', lineHeight: 28 },
  body: { fontSize: 16, fontWeight: '400', lineHeight: 24 },
  bodySmall: { fontSize: 14, fontWeight: '400', lineHeight: 21 },
  caption: { fontSize: 12, fontWeight: '400', lineHeight: 16 },
};

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 48,
};

export const radius = {
  sm: 4,
  md: 8,
  lg: 16,
  full: 9999,
};

export const shadows = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.08,
    shadowRadius: 3,
    elevation: 2,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 6,
    elevation: 4,
  },
};
```

```jsx
// Uso nos componentes
import { colors, typography, spacing, radius, shadows } from './theme';

const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.neutral[50],
    borderRadius: radius.lg,
    padding: spacing.md,
    gap: spacing.sm,
    ...shadows.md,
  },
  title: {
    ...typography.h3,
    color: colors.neutral[900],
  },
});
```

### Exemplo: tela completa — App Mobile (Figma → React Native)

**No Figma:** tela com Header, lista de itens scrollável e Bottom Tab Navigation.

```jsx
import { View, Text, TextInput, FlatList, Pressable, StyleSheet } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import Icon from 'react-native-vector-icons/MaterialCommunityIcons';

function HomeScreen() {
  const items = Array.from({ length: 10 }, (_, i) => ({
    id: String(i),
    title: `Item ${i + 1}`,
    subtitle: 'Descrição do item',
  }));

  return (
    <SafeAreaView style={styles.container}>
      {/* Header — App Bar do Figma */}
      <View style={styles.header}>
        <Pressable>
          <Icon name="menu" size={24} color="#1A1A1A" />
        </Pressable>
        <Text style={styles.headerTitle}>Meu App</Text>
        <View style={styles.headerActions}>
          <Pressable>
            <Icon name="magnify" size={24} color="#1A1A1A" />
          </Pressable>
          <Pressable>
            <Icon name="bell-outline" size={24} color="#1A1A1A" />
          </Pressable>
        </View>
      </View>

      {/* Conteúdo scrollável */}
      <FlatList
        data={items}
        contentContainerStyle={styles.content}
        ListHeaderComponent={
          <View style={styles.listHeader}>
            {/* Search Bar */}
            <TextInput
              placeholder="Buscar..."
              style={styles.searchInput}
              placeholderTextColor="#9E9E9E"
            />
            <Text style={styles.sectionTitle}>Destaques</Text>
          </View>
        }
        renderItem={({ item }) => (
          <Pressable style={styles.listItem}>
            {/* Thumbnail 48×48 */}
            <View style={styles.thumbnail} />
            <View style={styles.listItemText}>
              <Text style={styles.itemTitle}>{item.title}</Text>
              <Text style={styles.itemSubtitle}>{item.subtitle}</Text>
            </View>
            <Icon name="chevron-right" size={24} color="#9E9E9E" />
          </Pressable>
        )}
        keyExtractor={(item) => item.id}
      />

      {/* FAB — constraint Right + Bottom no Figma */}
      <Pressable style={styles.fab}>
        <Icon name="plus" size={24} color="#FFFFFF" />
      </Pressable>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  // Header — Auto Layout Horizontal, height 56, padding horizontal 16
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    height: 56,
    paddingHorizontal: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#EEEEEE',
  },
  headerTitle: {
    flex: 1,                        // Fill no Figma
    fontSize: 20,
    fontWeight: '600',
    marginLeft: 16,
    color: '#1A1A1A',
  },
  headerActions: {
    flexDirection: 'row',
    gap: 12,
  },
  // Conteúdo
  content: {
    padding: 16,
  },
  listHeader: {
    gap: 16,
    marginBottom: 12,
  },
  searchInput: {
    backgroundColor: '#F5F5F5',
    borderRadius: 12,
    padding: 12,
    fontSize: 16,
  },
  sectionTitle: {
    fontSize: 22,
    fontWeight: '600',
    color: '#1A1A1A',
  },
  // List Item — Auto Layout Horizontal, gap 12
  listItem: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 12,
    padding: 12,
    marginBottom: 8,
    backgroundColor: '#FAFAFA',
    borderRadius: 12,
  },
  thumbnail: {
    width: 48,
    height: 48,
    backgroundColor: '#E0E0E0',
    borderRadius: 8,
  },
  listItemText: {
    flex: 1,                        // Fill no Figma
    gap: 4,
  },
  itemTitle: {
    fontSize: 16,
    fontWeight: '500',
    color: '#1A1A1A',
  },
  itemSubtitle: {
    fontSize: 14,
    color: '#9E9E9E',
  },
  // FAB — position absolute, right 16, bottom 16
  fab: {
    position: 'absolute',
    right: 16,
    bottom: 16,
    width: 56,
    height: 56,
    borderRadius: 16,
    backgroundColor: '#1976D2',
    alignItems: 'center',
    justifyContent: 'center',
    elevation: 6,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 8,
  },
});
```

### Navegação: Bottom Tabs (Figma → React Navigation)

A **Bottom Navigation** do Figma é implementada com **React Navigation**:

```jsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Icon from 'react-native-vector-icons/MaterialCommunityIcons';

const Tab = createBottomTabNavigator();

function AppNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#1976D2',      // cor ativa do Figma
        tabBarInactiveTintColor: '#9E9E9E',    // cor inativa do Figma
        tabBarStyle: {
          height: 80,                           // height do Bottom Nav no Figma
          paddingBottom: 16,
          paddingTop: 8,
        },
        tabBarLabelStyle: {
          fontSize: 12,                         // Caption do Figma
        },
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          headerShown: false,
          tabBarIcon: ({ color, size }) => (
            <Icon name="home" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Explorar"
        component={ExploreScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="compass" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Favoritos"
        component={FavoritesScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="heart" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Perfil"
        component={ProfileScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="account" size={size} color={color} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}
```

### Diferenças importantes: sombras iOS vs Android

No Figma, a sombra é definida com um único valor. No React Native, é preciso definir separadamente:

| Figma Shadow | iOS (React Native) | Android (React Native) |
|---|---|---|
| `x: 0, y: 2, blur: 8, color: rgba(0,0,0,0.1)` | `shadowColor: '#000', shadowOffset: {width: 0, height: 2}, shadowOpacity: 0.1, shadowRadius: 8` | `elevation: 3` |

> **Dica:** O `elevation` no Android não permite controle fino como no Figma. Para sombras precisas no Android, use bibliotecas como `react-native-shadow-2`.

### Ferramentas para conversão Figma → React Native

| Ferramenta | Descrição |
|---|---|
| **Figma Dev Mode** | Inspecionar propriedades — traduzir manualmente |
| **Locofy** | Converte designs Figma para código React Native |
| **Anima** | Gera código React Native a partir de designs Figma |
| **React Native Paper** | Biblioteca de componentes Material Design para RN |
| **NativeBase** | Biblioteca de componentes UI com suporte a temas |
| **Tamagui** | UI kit com temas e responsividade para RN e Web |
| **Gluestack UI** | Componentes acessíveis com suporte a temas |

### Comparativo: propriedades CSS vs React Native StyleSheet

| Propriedade CSS | React Native StyleSheet |
|---|---|
| `display: flex` | Sempre flex por padrão |
| `flex-direction: row` | `flexDirection: 'row'` |
| `justify-content: center` | `justifyContent: 'center'` |
| `align-items: center` | `alignItems: 'center'` |
| `gap: 16px` | `gap: 16` |
| `padding: 16px` | `padding: 16` |
| `margin: 8px 16px` | `marginVertical: 8, marginHorizontal: 16` |
| `border-radius: 12px` | `borderRadius: 12` |
| `background-color: #fff` | `backgroundColor: '#fff'` |
| `font-size: 16px` | `fontSize: 16` |
| `font-weight: bold` | `fontWeight: 'bold'` |
| `line-height: 24px` | `lineHeight: 24` |
| `color: #333` | `color: '#333'` |
| `box-shadow: ...` | `shadowColor, shadowOffset, shadowOpacity, shadowRadius` (iOS) + `elevation` (Android) |
| `overflow: hidden` | `overflow: 'hidden'` |
| `position: absolute` | `position: 'absolute'` |
| `width: 100%` | `width: '100%'` ou `flex: 1` (context-dependent) |
| `max-width: 400px` | `maxWidth: 400` |

---

## 19. Onde Obter Assets e Recursos

### Kits de UI e Design Systems

| Recurso | Descrição | Link |
|---|---|---|
| **Figma Community** | Comunidade oficial com milhares de arquivos gratuitos | [community.figma.com](https://www.figma.com/community) |
| **Material Design 3 Kit** | Kit oficial do Google para Material Design | Buscar "Material 3 Design Kit" no Figma Community |
| **iOS/iPadOS Design Kit** | Kit oficial da Apple para iOS | Buscar "Apple Design Resources" no Figma Community |
| **Ant Design** | Design system popular para aplicações web/enterprise | Buscar "Ant Design" no Figma Community |
| **Shadcn UI Kit** | Kit baseado no Shadcn/UI (React) | Buscar "shadcn/ui" no Figma Community |
| **Bootstrap 5 UI Kit** | Componentes do Bootstrap para Figma | Buscar "Bootstrap 5" no Figma Community |
| **Tailwind UI Kit** | Componentes inspirados no Tailwind CSS | Buscar "Tailwind" no Figma Community |

### Ícones

| Recurso | Estilo | Licença | Link |
|---|---|---|---|
| **Material Symbols** | Outlined, Rounded, Sharp | Apache 2.0 (gratuito) | [fonts.google.com/icons](https://fonts.google.com/icons) |
| **Phosphor Icons** | Variados (6 pesos) | MIT (gratuito) | [phosphoricons.com](https://phosphoricons.com/) |
| **Lucide** | Outlined (fork do Feather) | ISC (gratuito) | [lucide.dev](https://lucide.dev/) |
| **Heroicons** | Outline, Solid, Mini | MIT (gratuito) | [heroicons.com](https://heroicons.com/) |
| **Tabler Icons** | Outlined (5100+) | MIT (gratuito) | [tabler.io/icons](https://tabler.io/icons) |
| **Font Awesome** | Solid, Regular, Brands | CC BY 4.0 / comercial | [fontawesome.com](https://fontawesome.com/) |
| **Iconify** | Agregador de múltiplas bibliotecas | Varia por biblioteca | [iconify.design](https://iconify.design/) |

### Imagens e Fotografias

| Recurso | Tipo | Licença | Link |
|---|---|---|---|
| **Unsplash** | Fotografias | Gratuita para uso comercial | [unsplash.com](https://unsplash.com/) |
| **Pexels** | Fotografias e vídeos | Gratuita para uso comercial | [pexels.com](https://www.pexels.com/) |
| **Pixabay** | Fotografias, vetores, vídeos | Gratuita para uso comercial | [pixabay.com](https://pixabay.com/) |
| **Kaboompics** | Fotografias (estilo lifestyle) | Gratuita para uso comercial | [kaboompics.com](https://kaboompics.com/) |

### Ilustrações

| Recurso | Estilo | Licença | Link |
|---|---|---|---|
| **unDraw** | Flat, customizável por cor | MIT (gratuito) | [undraw.co](https://undraw.co/) |
| **Storyset** | Animadas e estáticas | Gratuita com atribuição | [storyset.com](https://storyset.com/) |
| **Humaaans** | Pessoas, mix-and-match | Gratuita para uso pessoal | [humaaans.com](https://www.humaaans.com/) |
| **Open Doodles** | Doodles / sketchy | CC0 (domínio público) | [opendoodles.com](https://www.opendoodles.com/) |
| **DrawKit** | Flat e 3D | Gratuita e paga | [drawkit.com](https://www.drawkit.com/) |
| **Blush** | Combináveis (plugin Figma) | Gratuita com limitações | [blush.design](https://blush.design/) |

### Fontes (Tipografia)

| Recurso | Descrição | Link |
|---|---|---|
| **Google Fonts** | Maior biblioteca de fontes gratuitas | [fonts.google.com](https://fonts.google.com/) |
| **Fontsource** | Fontes auto-hospedadas para projetos web (npm) | [fontsource.org](https://fontsource.org/) |
| **Font Pair** | Sugestões de combinação de fontes | [fontpair.co](https://www.fontpair.co/) |

**Fontes recomendadas para UI:**

| Fonte | Tipo | Uso recomendado |
|---|---|---|
| **Inter** | Sans-serif | UI geral, altamente legível em telas |
| **Roboto** | Sans-serif | Material Design, Android |
| **SF Pro** | Sans-serif | iOS, macOS (Apple) |
| **Nunito** | Sans-serif | UI amigável, arredondada |
| **Poppins** | Sans-serif | UI moderna, geométrica |
| **DM Sans** | Sans-serif | UI limpa, boa para headings |
| **JetBrains Mono** | Monospace | Código, terminais |

### Mockups e Apresentações

| Recurso | Descrição | Link |
|---|---|---|
| **Mockup World** | Mockups de dispositivos gratuitos | [mockupworld.co](https://www.mockupworld.co/) |
| **Previewed** | Mockups 3D animados (freemium) | [previewed.app](https://previewed.app/) |
| **Shots.so** | Mockups para screenshots de apps | [shots.so](https://shots.so/) |

### Paletas de Cores

| Recurso | Descrição | Link |
|---|---|---|
| **Coolors** | Gerador de paletas de cores | [coolors.co](https://coolors.co/) |
| **Color Hunt** | Paletas curadas por designers | [colorhunt.co](https://colorhunt.co/) |
| **Realtime Colors** | Visualizar paleta aplicada em layout real | [realtimecolors.com](https://www.realtimecolors.com/) |
| **Contrast Checker (WebAIM)** | Verificador de contraste WCAG | [webaim.org/resources/contrastchecker](https://webaim.org/resources/contrastchecker/) |

---

## 20. Plugins Recomendados

Para instalar: menu principal → **Plugins** → **Find Plugins** ou acesse via [Figma Community](https://www.figma.com/community/plugins).

| Plugin | Função |
|---|---|
| **Unsplash** | Inserir fotos diretamente do Unsplash |
| **Iconify** | Pesquisar e inserir ícones de múltiplas bibliotecas |
| **Stark** | Verificar contraste e acessibilidade |
| **Content Reel** | Preencher textos, nomes, avatares realistas |
| **Lorem Ipsum** | Gerar texto placeholder |
| **Blush** | Inserir e customizar ilustrações |
| **Figmotion** | Criar animações diretamente no Figma |
| **Autoflow** | Criar linhas de fluxo entre telas |
| **Color Contrast Checker** | Verificar conformidade WCAG |
| **Remove BG** | Remover fundo de imagens |
| **HTML to Figma** | Importar páginas web para o Figma |

---

## 21. Referências

### Documentação oficial

- [Figma Help Center](https://help.figma.com/) — documentação completa
- [Figma Best Practices](https://www.figma.com/best-practices/) — guias oficiais
- [Figma Community](https://www.figma.com/community) — templates, plugins e widgets

### Guias de Design Systems

- [Material Design 3](https://m3.material.io/) — Design System do Google
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines) — Diretrizes da Apple
- [Carbon Design System (IBM)](https://carbondesignsystem.com/) — Design System da IBM

### Cursos e tutoriais

- [Figma — canal oficial no YouTube](https://www.youtube.com/@Figma) — tutoriais e webinars
- [Figma for Beginners (playlist oficial)](https://www.youtube.com/playlist?list=PLXDU_eVOJTx7QHLShNqIXL1Cgbxj7HlN4) — série para iniciantes

### Acessibilidade

- [WCAG 2.1 (W3C)](https://www.w3.org/TR/WCAG21/) — diretrizes de acessibilidade
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) — verificador de contraste online
