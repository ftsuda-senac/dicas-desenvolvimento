# Desenvolvimento de Telas Web — Frontend

Referência consolidada sobre usabilidade, acessibilidade, HTML, CSS e SCSS para desenvolvimento de interfaces web.

---

## Sumário

1. [Usabilidade](#1-usabilidade)
2. [Design de Interfaces](#2-design-de-interfaces)
   - [10 Heurísticas de Jakob Nielsen](#10-heurísticas-de-jakob-nielsen)
   - [Regras de Ouro de Theo Mandel](#regras-de-ouro-de-theo-mandel)
   - [ISO 9241 — Ergonomia da Interação Humano-Sistema](#iso-9241--ergonomia-da-interação-humano-sistema)
3. [Acessibilidade](#3-acessibilidade)
4. [Mobile-First e Design Responsivo](#4-mobile-first-e-design-responsivo)
5. [Visão Geral do Figma](#5-visão-geral-do-figma)
6. [HTML — Principais Tags](#6-html--principais-tags)
7. [CSS — Seletores e Propriedades](#7-css--seletores-e-propriedades)
8. [CSS Moderno](#8-css-moderno)
9. [SCSS](#9-scss)

---

## 1. Usabilidade

Usabilidade é a medida de quão fácil, eficiente e satisfatório é para o usuário realizar seus objetivos em um sistema.

### Princípios fundamentais

| Princípio | Descrição |
|---|---|
| **Aprendizagem** | O sistema deve ser fácil de aprender na primeira vez |
| **Eficiência** | Usuários experientes devem conseguir operar com rapidez |
| **Memorização** | Após um período sem uso, o usuário deve retomar sem dificuldade |
| **Erros** | Baixa taxa de erros; erros graves devem ser evitáveis |
| **Satisfação** | A experiência deve ser agradável |

### Boas práticas

- **Feedback imediato:** Exibir resposta visual após toda ação do usuário (carregando, sucesso, erro).
- **Confirmação para ações destrutivas:** Operações como excluir devem pedir confirmação antes de executar.
- **Prevenção de cliques duplos:** Desabilitar botões após o clique e exibir indicador de carregamento, especialmente em operações lentas (debounce/disable).
- **Mensagens de erro claras:** Indicar o que deu errado e como corrigir, não apenas um código de erro genérico.
- **Consistência visual:** Mesmos padrões de botões, cores e tipografia em todo o sistema.
- **Hierarquia visual:** Destacar visualmente as ações primárias; ações secundárias devem ser menos evidentes.
- **Formulários bem projetados:** Indicar campos obrigatórios, fornecer textos de ajuda (placeholder/hint), agrupar campos relacionados e mostrar erros próximos ao campo correspondente.
- **URLs amigáveis:** Usar kebab-case em caminhos compostos (ex: `/meu-produto/lista`), boa prática para SEO e compartilhamento de links.

---

## 2. Design de Interfaces

Projetar boas interfaces exige conhecer os princípios e modelos que guiam as decisões de design. Esta seção apresenta três referências fundamentais: as heurísticas de Nielsen, as regras de Mandel e a norma ISO 9241.

---

### 10 Heurísticas de Jakob Nielsen

As heurísticas de Nielsen são 10 princípios gerais de design de interface, amplamente usados para avaliar e projetar sistemas.

#### 1. Visibilidade do status do sistema
O sistema deve sempre manter o usuário informado sobre o que está acontecendo, por meio de feedback adequado e em tempo razoável.

> Exemplos: barra de progresso, spinner de carregamento, mensagem "Salvo com sucesso", badge de notificações.

#### 2. Correspondência entre o sistema e o mundo real
O sistema deve falar a linguagem do usuário — palavras, frases e conceitos familiares — em vez de jargões técnicos. Seguir convenções do mundo real.

> Exemplo: ícone de lixeira para excluir, ícone de disquete (ainda reconhecido) para salvar, carrinho de compras para e-commerce.

#### 3. Controle e liberdade do usuário
Usuários frequentemente escolhem funções por engano e precisam de uma "saída de emergência" claramente marcada para sair do estado indesejado.

> Exemplos: botão "Desfazer", confirmação antes de ações irreversíveis, botão "Cancelar" em modais.

#### 4. Consistência e padrões
Usuários não devem se perguntar se palavras, situações ou ações diferentes significam a mesma coisa. Seguir convenções da plataforma.

> Exemplos: botão primário sempre na mesma posição, mesma cor para links, mesma terminologia em toda a aplicação.

#### 5. Prevenção de erros
Melhor do que boas mensagens de erro é um design cuidadoso que evita o problema antes de ocorrer.

> Exemplos: desabilitar o botão "Enviar" até o formulário ser válido, confirmar antes de excluir, input de data com datepicker em vez de campo livre.

#### 6. Reconhecimento em vez de memorização
Minimizar a carga de memória do usuário tornando objetos, ações e opções visíveis. O usuário não deve precisar lembrar informações de uma parte para outra.

> Exemplos: breadcrumbs mostrando onde o usuário está, histórico de pesquisas, preencher automaticamente dados já conhecidos.

#### 7. Flexibilidade e eficiência de uso
Atalhos — invisíveis para usuários novatos — podem acelerar a interação para usuários experientes. Permitir personalização de ações frequentes.

> Exemplos: atalhos de teclado, filtros avançados, buscas salvas, acesso rápido a itens recentes.

#### 8. Estética e design minimalista
As interfaces não devem conter informações irrelevantes ou raramente necessárias. Cada unidade extra de informação compete com as relevantes e diminui sua visibilidade.

> Exemplos: remover campos de formulário não essenciais, evitar textos longos em telas de ação, usar progressive disclosure (mostrar detalhes conforme necessário).

#### 9. Ajudar usuários a reconhecer, diagnosticar e recuperar erros
Mensagens de erro devem ser expressas em linguagem simples (sem códigos), indicar precisamente o problema e sugerir uma solução.

> Exemplos: "O campo E-mail é obrigatório" em vez de "Erro 422", "Senha deve ter ao menos 8 caracteres" em vez de "Senha inválida".

#### 10. Ajuda e documentação
Mesmo que seja melhor que o sistema possa ser usado sem documentação, pode ser necessário fornecer ajuda. A ajuda deve ser fácil de encontrar, focada na tarefa do usuário e concisa.

> Exemplos: tooltips em ícones menos óbvios, página de FAQ, documentação contextual, onboarding para novos usuários.

---

### Regras de Ouro de Theo Mandel

Publicadas no livro *The Elements of User Interface Design* (1997), as três regras de Mandel organizam o design de interfaces em torno do controle do usuário, da carga cognitiva e da consistência. Complementam as heurísticas de Nielsen com uma visão mais estrutural.

#### 1. Coloque o usuário no controle

O usuário deve sentir que comanda a interface, não o contrário.

- **Modos de interação flexíveis** — oferecer diferentes formas de realizar a mesma ação (mouse, teclado, toque).
- **Interação interrompível e reversível** — o usuário pode cancelar ou desfazer qualquer operação a qualquer momento.
- **Progressão por nível de habilidade** — iniciantes e experientes conseguem operar o sistema; atalhos ficam disponíveis para usuários avançados.
- **Esconder a complexidade técnica** — o usuário não precisa conhecer o funcionamento interno do sistema.
- **Interação direta com objetos na tela** — elementos visuais devem reagir diretamente às ações (arrastar, clicar, redimensionar).

#### 2. Reduza a carga de memória do usuário

A interface deve lembrar pelo usuário, não exigir que ele lembre por conta própria.

- **Minimizar o uso da memória de curto prazo** — não exigir que o usuário memorize informações de uma tela para aplicar em outra.
- **Estabelecer padrões significativos** — valores padrão devem ser os mais prováveis ou seguros para a maioria dos casos.
- **Metáforas do mundo real** — o layout e os ícones devem remeter a objetos e conceitos familiares.
- **Divulgação progressiva** — mostrar apenas o necessário no momento; detalhes avançados aparecem conforme a necessidade.

#### 3. Torne a interface consistente

O comportamento deve ser previsível em toda a aplicação e alinhado com as expectativas do usuário.

- **Contexto claro da tarefa atual** — o usuário deve sempre saber onde está e o que está fazendo.
- **Consistência dentro da família de aplicações** — sistemas relacionados devem compartilhar padrões visuais e de interação.
- **Não quebrar expectativas já estabelecidas** — se um padrão foi aprendido pelo usuário (ex: `Ctrl+Z` para desfazer), não alterá-lo sem motivo forte.

#### Relação entre Mandel e Nielsen

| Regra de Mandel | Heurísticas de Nielsen equivalentes |
|---|---|
| Coloque o usuário no controle | Controle e liberdade do usuário · Flexibilidade e eficiência de uso · Prevenção de erros |
| Reduza a carga de memória | Reconhecimento em vez de memorização · Correspondência com o mundo real · Estética minimalista |
| Torne a interface consistente | Consistência e padrões · Visibilidade do status · Ajuda e documentação |

---

### ISO 9241 — Ergonomia da Interação Humano-Sistema

A **ISO 9241** é uma norma internacional que define requisitos e recomendações para o design ergonômico de sistemas interativos. Seu foco é garantir que produtos digitais sejam eficazes, eficientes e satisfatórios para os usuários.

A norma é dividida em várias partes. A mais referenciada na área de design de interfaces é a **ISO 9241-11**, que define formalmente o conceito de **usabilidade**, e a **ISO 9241-110**, que trata dos princípios de diálogo.

#### ISO 9241-11 — Definição de Usabilidade

Define usabilidade como a medida em que um produto pode ser usado por usuários específicos para alcançar objetivos específicos com **eficácia**, **eficiência** e **satisfação** em um contexto de uso específico.

| Dimensão | Definição |
|---|---|
| **Eficácia** | O usuário consegue completar a tarefa com precisão e completude |
| **Eficiência** | Os recursos necessários (tempo, esforço) para completar a tarefa |
| **Satisfação** | O grau de conforto e aceitação positiva durante o uso |

#### ISO 9241-110 — Princípios de Diálogo

Define sete princípios para o design da interação entre usuário e sistema:

| Princípio | Descrição |
|---|---|
| **Adequação à tarefa** | O sistema apoia o usuário a concluir suas tarefas sem esforço desnecessário |
| **Autodescritivo** | Cada etapa é compreensível por si só, com feedback suficiente |
| **Controlabilidade** | O usuário pode iniciar, controlar e encerrar a interação |
| **Conformidade com as expectativas** | O sistema se comporta de forma consistente e previsível |
| **Tolerância a erros** | Erros podem ser corrigidos com o mínimo de esforço; o sistema previne erros graves |
| **Adequação à individualização** | A interface pode ser adaptada às necessidades e preferências do usuário |
| **Adequação ao aprendizado** | O sistema facilita o aprendizado progressivo sem exigir treinamento extenso |

#### Relação com Nielsen e Mandel

A ISO 9241 atua em um nível normativo e formal, enquanto Nielsen e Mandel oferecem princípios práticos e heurísticos. As três abordagens se complementam:

- A **definição de usabilidade** da ISO 9241-11 (eficácia, eficiência, satisfação) é a base conceitual mais usada em avaliações formais e documentos de requisitos.
- Os **princípios de diálogo** da ISO 9241-110 têm correspondência direta com as heurísticas de Nielsen e as regras de Mandel.
- Na prática, as heurísticas de Nielsen são mais usadas em avaliações rápidas (heuristic evaluation), enquanto a ISO 9241 embasa contratos, licitações e requisitos de sistemas críticos.

---

## 3. Acessibilidade

> MDN: [Accessibility](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility) · [ARIA](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility/ARIA)

Acessibilidade (a11y) garante que pessoas com deficiências visuais, auditivas, motoras ou cognitivas possam usar a aplicação. O padrão de referência é o **WCAG** (Web Content Accessibility Guidelines), atualmente na versão 2.2, com três níveis: A, AA e AAA.

### Por que importa?

- Atinge um público maior, incluindo pessoas com deficiência permanente ou situacional (ex: uso com uma mão, ambiente com luz solar direta).
- Exigência legal em muitos países e no Brasil (Lei Brasileira de Inclusão — Lei 13.146/2015).
- Melhora SEO e a qualidade geral do código.

### Princípios WCAG (POUR)

| Princípio | O que significa |
|---|---|
| **Perceptível** | Informação deve ser apresentada de forma que o usuário possa perceber |
| **Operável** | A interface deve ser navegável e utilizável |
| **Compreensível** | Conteúdo e operações devem ser compreensíveis |
| **Robusto** | Conteúdo deve ser interpretado por tecnologias assistivas |

### Boas práticas de acessibilidade

**HTML semântico**
- Usar as tags corretas para cada conteúdo (`<nav>`, `<main>`, `<article>`, `<button>`, `<label>`, etc.).
- Tags semânticas comunicam significado para leitores de tela automaticamente.

**Textos alternativos**
- Imagens informativas devem ter `alt` descritivo.
- Imagens decorativas devem ter `alt=""` para serem ignoradas por leitores de tela.

**Contraste de cores**
- Texto normal: mínimo 4.5:1 (nível AA).
- Texto grande (≥18pt ou ≥14pt negrito): mínimo 3:1.
- Ferramentas: [Colour Contrast Checker](https://colourcontrast.cc), extensão Axe DevTools.

**Navegação por teclado**
- Todo elemento interativo deve ser acessível via `Tab`.
- O foco visível (`outline`) nunca deve ser removido sem substituto adequado.
- Modais devem prender o foco enquanto abertos (focus trap).

**ARIA (Accessible Rich Internet Applications)**
- Usar atributos ARIA quando o HTML semântico não for suficiente.
- `aria-label`: nomeia elementos sem texto visível.
- `aria-describedby`: associa uma descrição adicional.
- `aria-live`: anuncia atualizações dinâmicas ao leitor de tela.
- `role`: define a função de um elemento (ex: `role="dialog"`, `role="alert"`).
- Regra: **prefira HTML semântico nativo antes de usar ARIA**.

**Formulários acessíveis**
```html
<!-- Correto: label associado ao input -->
<label for="email">E-mail</label>
<input type="email" id="email" name="email" required aria-describedby="email-hint">
<span id="email-hint">Informe seu e-mail corporativo</span>
```

**Links e botões**
- Links navegam para destinos; botões executam ações — não trocar semanticamente.
- Evitar "clique aqui" como texto de link; descrever o destino.

---

## 4. Mobile-First e Design Responsivo

### O que é Mobile-First?

**Mobile-First** é uma estratégia de design e desenvolvimento onde a interface é criada primeiro para telas pequenas (smartphones) e progressivamente aprimorada para telas maiores.

**Por quê?**
- Mais de 60% do tráfego web global vem de dispositivos móveis.
- Forçar a disciplina: com espaço limitado, mantém-se apenas o essencial.
- Melhor performance: o CSS base (sem media queries) é o mais leve.
- Alinhado ao critério de indexação do Google (Mobile-First Indexing).

### Design Responsivo

**Responsive Web Design (RWD)** é a técnica de fazer layouts que se adaptam ao tamanho do viewport, usando:

- **Grades fluidas:** larguras em `%` ou `fr` em vez de pixels fixos.
- **Imagens flexíveis:** `max-width: 100%` para imagens não excederem o container.
- **Media queries:** aplicar estilos conforme o tamanho da tela.

### Breakpoints comuns

| Nome | Largura mínima | Dispositivos típicos |
|---|---|---|
| `xs` (base) | 0px | Smartphones pequenos |
| `sm` | 576px | Smartphones grandes |
| `md` | 768px | Tablets |
| `lg` | 992px | Laptops |
| `xl` | 1200px | Desktops |
| `xxl` | 1400px | Monitores grandes |

> Os breakpoints do Bootstrap 5 são amplamente adotados como referência.

### Abordagem Mobile-First em CSS

```css
/* Base: mobile (sem media query) */
.container {
  padding: 1rem;
  flex-direction: column;
}

/* Tablet em diante */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    flex-direction: row;
  }
}

/* Desktop em diante */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
    margin-inline: auto;
  }
}
```

> **Dica:** Usar `min-width` (mobile-first) é o padrão moderno. Evitar `max-width` (desktop-first), que vai contra a estratégia.

### Viewport e unidades relativas

```html
<!-- Essencial no <head> -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

| Unidade | Significa |
|---|---|
| `%` | Relativo ao elemento pai |
| `vw` / `vh` | % da largura/altura do viewport |
| `dvw` / `dvh` | Viewport dinâmico (considera barras do navegador móvel) |
| `rem` | Relativo ao `font-size` do `<html>` |
| `em` | Relativo ao `font-size` do elemento pai |

---

## 5. Visão Geral do Figma

**Figma** é a ferramenta de design de interfaces (UI/UX) mais utilizada atualmente. Funciona no navegador e possui versão desktop, com colaboração em tempo real.

### Para que serve no fluxo de desenvolvimento

| Fase | Uso |
|---|---|
| **Wireframe** | Rascunho da estrutura das telas (baixa fidelidade) |
| **Protótipo** | Simulação de navegação entre telas clicáveis |
| **Design final (mockup)** | Telas com visual final, cores, tipografia e espaçamentos |
| **Handoff para dev** | Especificações de CSS, exportação de assets |

### Conceitos principais

- **Frame:** equivalente a uma tela ou componente. Define a área de trabalho.
- **Component:** elemento reutilizável (ex: botão, card). Alterações no componente mestre propagam para todas as instâncias.
- **Variant:** variações de um componente (ex: botão primário / secundário / desabilitado).
- **Auto Layout:** equivalente ao Flexbox do CSS — organiza elementos com espaçamento e direção automáticos.
- **Styles:** define paleta de cores e estilos de texto reutilizáveis (equivalente a variáveis CSS).
- **Design Tokens:** exportação dos valores de design (cores, espaçamentos) para uso direto no código.

### Inspecionar propriedades para CSS

No painel "Inspect" (Dev Mode), o Figma exibe:
- `font-family`, `font-size`, `font-weight`, `line-height`
- `color`, `background`, `border-radius`, `box-shadow`
- Espaçamentos (`padding`, `gap`, `margin`)
- Dimensões e posicionamento

### Dica para desenvolvedores

- Sempre conferir o **Inspect** antes de codificar um componente.
- Pedir ao designer que use **Auto Layout** e **Components**, pois facilita muito a tradução para CSS/HTML.
- O **Figma Community** oferece kits de UI gratuitos para estudo.

---

## 6. HTML — Principais Tags

> Referência completa: [MDN — HTML elements reference](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element)

### Estrutura do documento

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="Descrição da página para SEO">
  <title>Título da Página</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <!-- conteúdo -->
</body>
</html>
```

### Tags semânticas de layout

| Tag | Uso |
|---|---|
| `<header>` | Cabeçalho da página ou de uma seção |
| `<nav>` | Menu de navegação principal ou secundário |
| `<main>` | Conteúdo principal da página (único por página) |
| `<section>` | Seção temática de conteúdo |
| `<article>` | Conteúdo independente e auto-suficiente (post, card) |
| `<aside>` | Conteúdo relacionado mas periférico (sidebar, widgets) |
| `<footer>` | Rodapé da página ou seção |

### Títulos e texto

| Tag | Uso |
|---|---|
| `<h1>` a `<h6>` | Hierarquia de títulos (apenas um `<h1>` por página) |
| `<p>` | Parágrafo de texto |
| `<strong>` | Ênfase forte (negrito semântico) |
| `<em>` | Ênfase (itálico semântico) |
| `<span>` | Container inline genérico (sem semântica) |
| `<blockquote>` | Citação em bloco |
| `<code>` | Trecho de código inline |
| `<pre>` | Texto pré-formatado (código em bloco) |
| `<abbr>` | Abreviação com tooltip de expansão |

### Listas

#### Lista não ordenada (`<ul>`)

Usada quando a ordem dos itens não importa.

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

**Atributos e variações úteis via CSS:**
```css
ul { list-style-type: disc; }    /* padrão: bolinha sólida */
ul { list-style-type: circle; }  /* bolinha vazia */
ul { list-style-type: square; }  /* quadrado */
ul { list-style: none; }         /* remove o marcador — comum em menus */
```

#### Lista ordenada (`<ol>`)

Usada quando a sequência é relevante (passos, rankings, instruções).

```html
<!-- Padrão: 1, 2, 3... -->
<ol>
  <li>Instalar o Node.js</li>
  <li>Criar o projeto</li>
  <li>Instalar dependências</li>
</ol>

<!-- Atributo type — altera o tipo de marcador -->
<ol type="A">          <!-- A, B, C... -->
<ol type="a">          <!-- a, b, c... -->
<ol type="I">          <!-- I, II, III... (romano maiúsculo) -->
<ol type="i">          <!-- i, ii, iii... (romano minúsculo) -->

<!-- Atributo start — inicia a contagem a partir de um número -->
<ol start="5">
  <li>Quinto item</li>   <!-- exibe "5." -->
  <li>Sexto item</li>    <!-- exibe "6." -->
</ol>

<!-- Atributo reversed — contagem decrescente -->
<ol reversed>
  <li>Terceiro</li>  <!-- exibe "3." -->
  <li>Segundo</li>   <!-- exibe "2." -->
  <li>Primeiro</li>  <!-- exibe "1." -->
</ol>
```

**Controle de numeração via CSS:**
```css
ol { list-style-type: decimal; }        /* 1, 2, 3 (padrão) */
ol { list-style-type: lower-alpha; }    /* a, b, c */
ol { list-style-type: upper-roman; }    /* I, II, III */

/* Marcador personalizado com ::marker */
ol li::marker {
  color: #2563eb;
  font-weight: bold;
}
```

#### Listas aninhadas

```html
<ul>
  <li>Frontend
    <ul>
      <li>HTML</li>
      <li>CSS
        <ul>
          <li>Flexbox</li>
          <li>Grid</li>
        </ul>
      </li>
    </ul>
  </li>
  <li>Backend</li>
</ul>
```

#### Lista de definição (`<dl>`)

Usada para pares termo/descrição: glossários, metadados, especificações.

```html
<dl>
  <dt>HTML</dt>
  <dd>Linguagem de marcação para estrutura do conteúdo web.</dd>

  <dt>CSS</dt>
  <dd>Linguagem de estilos para apresentação visual das páginas.</dd>

  <!-- Um termo pode ter múltiplas descrições -->
  <dt>HTTP</dt>
  <dd>Protocolo de transferência de hipertexto.</dd>
  <dd>Base da comunicação de dados na Web.</dd>
</dl>
```

### Links e mídias

| Tag | Atributos importantes |
|---|---|
| `<a>` | `href`, `target="_blank"`, `rel="noopener noreferrer"` |
| `<img>` | `src`, `alt`, `width`, `height`, `loading="lazy"` |
| `<video>` | `src`, `controls`, `autoplay`, `muted`, `loop` |
| `<audio>` | `src`, `controls` |
| `<figure>` + `<figcaption>` | Agrupa mídia com legenda |
| `<picture>` + `<source>` | Imagens responsivas com múltiplos formatos/tamanhos |

### Formulários

> MDN: [Web forms](https://developer.mozilla.org/pt-BR/docs/Learn_web_development/Extensions/Forms) · [input types](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types)

```html
<form action="/enviar" method="post">

  <fieldset>
    <legend>Dados pessoais</legend>

    <label for="nome">Nome completo</label>
    <input type="text" id="nome" name="nome" required placeholder="Seu nome">

    <label for="email">E-mail</label>
    <input type="email" id="email" name="email" required>

    <label for="telefone">Telefone</label>
    <input type="tel" id="telefone" name="telefone">

    <label for="nascimento">Data de nascimento</label>
    <input type="date" id="nascimento" name="nascimento">
  </fieldset>

  <label for="categoria">Categoria</label>
  <select id="categoria" name="categoria">
    <option value="">Selecione...</option>
    <option value="a">Opção A</option>
  </select>

  <label for="mensagem">Mensagem</label>
  <textarea id="mensagem" name="mensagem" rows="4"></textarea>

  <!-- Checkbox -->
  <input type="checkbox" id="aceito" name="aceito">
  <label for="aceito">Aceito os termos</label>

  <!-- Radio -->
  <input type="radio" id="op1" name="opcao" value="1">
  <label for="op1">Opção 1</label>

  <button type="submit">Enviar</button>
  <button type="reset">Limpar</button>
</form>
```

**Tipos de `<input>` úteis:**
`text`, `email`, `password`, `number`, `tel`, `url`, `date`, `time`, `datetime-local`, `color`, `range`, `file`, `checkbox`, `radio`, `hidden`, `search`

### Tabelas

```html
<table>
  <caption>Título da tabela (acessibilidade)</caption>
  <thead>
    <tr>
      <th scope="col">Nome</th>
      <th scope="col">Valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Item A</td>
      <td>R$ 10,00</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td>Total</td>
      <td>R$ 10,00</td>
    </tr>
  </tfoot>
</table>
```

### Outros elementos úteis

| Tag | Uso |
|---|---|
| `<div>` | Container de bloco genérico (sem semântica) |
| `<details>` + `<summary>` | Accordion nativo sem JavaScript |
| `<dialog>` | Modal nativo do HTML5 |
| `<progress>` | Barra de progresso |
| `<meter>` | Medidor de valor em uma faixa |
| `<time>` | Data/hora com atributo `datetime` para máquinas |
| `<mark>` | Texto destacado/marcado |
| `<hr>` | Separador temático horizontal |
| `<br>` | Quebra de linha (usar com moderação) |

### SEO — Boas práticas no HTML

**SEO** (Search Engine Optimization) é o conjunto de técnicas que melhora o posicionamento de uma página nos resultados de busca. Grande parte do SEO técnico é definida diretamente no HTML.

> MDN: [What is SEO?](https://developer.mozilla.org/en-US/docs/Glossary/SEO)

#### Meta tags essenciais

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <!-- Descrição exibida nos resultados de busca (150–160 caracteres) -->
  <meta name="description" content="Aprenda desenvolvimento web com exemplos práticos de HTML, CSS e JavaScript.">

  <!-- Impede indexação (usar em páginas internas, admin, etc.) -->
  <meta name="robots" content="noindex, nofollow">

  <!-- URL canônica — evita conteúdo duplicado -->
  <link rel="canonical" href="https://exemplo.com.br/pagina">

  <!-- Open Graph — compartilhamento em redes sociais -->
  <meta property="og:title" content="Título da Página">
  <meta property="og:description" content="Descrição para redes sociais.">
  <meta property="og:image" content="https://exemplo.com.br/imagem.jpg">
  <meta property="og:url" content="https://exemplo.com.br/pagina">
  <meta property="og:type" content="website">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Título da Página">
  <meta name="twitter:description" content="Descrição para o Twitter.">
  <meta name="twitter:image" content="https://exemplo.com.br/imagem.jpg">

  <title>Título da Página — Nome do Site</title>
</head>
```

#### Hierarquia de títulos

- Usar **apenas um `<h1>` por página**, contendo o tema principal.
- Manter a hierarquia lógica: `<h1>` → `<h2>` → `<h3>`, sem pular níveis.
- Os títulos comunicam a estrutura do conteúdo tanto para buscadores quanto para leitores de tela.

```html
<h1>Guia de CSS Moderno</h1>
  <h2>Flexbox</h2>
    <h3>Propriedades do contêiner</h3>
    <h3>Propriedades dos itens</h3>
  <h2>Grid</h2>
```

#### Tags semânticas e SEO

Buscadores interpretam a semântica do HTML para entender o conteúdo da página:

| Tag | Relevância para SEO |
|---|---|
| `<title>` | Fator de ranqueamento direto — exibido no resultado de busca |
| `<h1>`–`<h6>` | Indicam a hierarquia e os tópicos principais da página |
| `<main>` | Delimita o conteúdo principal, ignorando navegação e rodapé |
| `<article>` | Sinaliza conteúdo independente (posts, notícias) |
| `<nav>` | Identifica menus de navegação para o crawler |
| `<a href>` | Links internos distribuem autoridade entre páginas |
| `<img alt>` | Texto alternativo indexado e usado na busca de imagens |
| `<time datetime>` | Data legível por máquina (ex: publicação de artigos) |

#### URLs amigáveis

- Usar **kebab-case** para separar palavras: `/guia-css-moderno` e não `/guiaCSSModerno` nem `/guia_css_moderno`.
- URLs curtas, descritivas e sem parâmetros desnecessários.
- Evitar números ou IDs sem significado: `/produto/tenis-corrida` é melhor que `/produto/4872`.
- Manter URLs estáveis — redirecionar (301) caso precise renomear.

#### Performance e Core Web Vitals

O Google usa métricas de performance como fator de ranqueamento. As principais são:

| Métrica | O que mede | Meta |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Tempo para renderizar o maior elemento visível | < 2,5s |
| **CLS** (Cumulative Layout Shift) | Estabilidade visual (elementos que "pulam") | < 0,1 |
| **INP** (Interaction to Next Paint) | Resposta às interações do usuário | < 200ms |

**Boas práticas no HTML para performance:**

```html
<!-- Definir width e height em imagens evita CLS -->
<img src="banner.jpg" alt="Banner principal" width="1200" height="600">

<!-- Lazy loading para imagens fora da viewport inicial -->
<img src="produto.jpg" alt="Produto X" loading="lazy" width="400" height="300">

<!-- Preload para recursos críticos (hero image, fonte principal) -->
<link rel="preload" href="hero.jpg" as="image">
<link rel="preload" href="fonte.woff2" as="font" type="font/woff2" crossorigin>

<!-- Imagens modernas com fallback -->
<picture>
  <source srcset="imagem.avif" type="image/avif">
  <source srcset="imagem.webp" type="image/webp">
  <img src="imagem.jpg" alt="Descrição" width="800" height="400">
</picture>
```

#### Dados estruturados (Schema.org)

Permitem que buscadores entendam o tipo de conteúdo e exibam resultados enriquecidos (rich snippets).

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Guia de CSS Moderno",
  "author": {
    "@type": "Person",
    "name": "João Silva"
  },
  "datePublished": "2025-03-20",
  "image": "https://exemplo.com.br/imagem.jpg"
}
</script>
```

Tipos comuns: `Article`, `Product`, `FAQPage`, `BreadcrumbList`, `Organization`, `LocalBusiness`.

> Validador: [Google Rich Results Test](https://search.google.com/test/rich-results) · Referência: [Schema.org](https://schema.org)

#### Checklist SEO no HTML

- [ ] `<title>` único e descritivo por página (50–60 caracteres)
- [ ] `<meta name="description">` única por página (150–160 caracteres)
- [ ] Um único `<h1>` com o tema principal da página
- [ ] Hierarquia de headings lógica e sem saltos
- [ ] Todas as imagens com `alt` descritivo
- [ ] Todas as imagens com `width` e `height` definidos
- [ ] `<link rel="canonical">` para evitar conteúdo duplicado
- [ ] URLs em kebab-case e sem parâmetros desnecessários
- [ ] Links internos com texto descritivo (não "clique aqui")
- [ ] Open Graph tags para compartilhamento em redes sociais
- [ ] `loading="lazy"` em imagens abaixo da dobra

---

## 7. CSS — Seletores e Propriedades

> Referência completa: [MDN — CSS reference](https://developer.mozilla.org/pt-BR/docs/Web/CSS/Reference)

### Seletores básicos

| Seletor | Sintaxe | Seleciona |
|---|---|---|
| Universal | `*` | Todos os elementos |
| Tipo/Tag | `p` | Todos os `<p>` |
| Classe | `.card` | Elementos com `class="card"` |
| ID | `#menu` | Elemento com `id="menu"` |
| Atributo (presença) | `[disabled]` | Elementos que possuem o atributo, independente do valor |
| Atributo (valor exato) | `[type="email"]` | Elementos com atributo igual ao valor |
| Atributo (começa com) | `[href^="https"]` | Atributo cujo valor começa com a string |
| Atributo (termina com) | `[href$=".pdf"]` | Atributo cujo valor termina com a string |
| Atributo (contém) | `[class*="btn"]` | Atributo cujo valor contém a string em qualquer posição |
| Atributo (palavra exata) | `[class~="active"]` | Atributo contém a palavra exata (separada por espaços) |

```css
/* Presença do atributo — qualquer input desabilitado */
input[disabled] { opacity: 0.5; cursor: not-allowed; }

/* Valor exato */
input[type="checkbox"] { width: 1rem; height: 1rem; }

/* Começa com — links externos (https) */
a[href^="https"]::after { content: " ↗"; font-size: 0.75em; }

/* Termina com — links para PDF */
a[href$=".pdf"] { color: #dc2626; }
a[href$=".pdf"]::before { content: "📄 "; }

/* Contém — qualquer classe que contenha "col-" */
[class*="col-"] { padding-inline: 0.75rem; }
```

> MDN: [Attribute selectors](https://developer.mozilla.org/pt-BR/docs/Web/CSS/Attribute_selectors)

### Seletores combinadores

| Seletor | Sintaxe | Seleciona |
|---|---|---|
| Descendente | `nav a` | `<a>` em qualquer nível dentro de `<nav>` |
| Filho direto | `ul > li` | `<li>` filhos diretos de `<ul>` |
| Irmão adjacente | `h2 + p` | `<p>` imediatamente após `<h2>` |
| Irmãos gerais | `h2 ~ p` | Todos os `<p>` irmãos após `<h2>` |

### Pseudo-classes

| Pseudo-classe | Quando se aplica |
|---|---|
| `:hover` | Mouse sobre o elemento |
| `:focus` | Elemento com foco do teclado/clique |
| `:focus-visible` | Foco via teclado (evita outline no clique com mouse) |
| `:active` | Elemento sendo clicado |
| `:visited` | Link já visitado |
| `:disabled` | Elemento desabilitado |
| `:checked` | Checkbox/radio marcado |
| `:required` | Campo obrigatório |
| `:valid` / `:invalid` | Campo com valor válido/inválido |
| `:first-child` | Primeiro filho do pai |
| `:last-child` | Último filho do pai |
| `:nth-child(n)` | N-ésimo filho — aceita `even` (pares, equiv. `2n`), `odd` (ímpares, equiv. `2n+1`) ou fórmula `An+B` |
| `:not(seletor)` | Elementos que NÃO correspondem ao seletor |
| `:is(a, b)` | Corresponde a qualquer da lista (simplifica seletores) |
| `:where(a, b)` | Igual ao `:is()`, mas com especificidade zero |
| `:has(seletor)` | Pai que contém o filho correspondente |

```css
/* Linhas alternadas em tabela — palavras-chave even (par) e odd (ímpar) */
tr:nth-child(even) { background: #f5f5f5; }
tr:nth-child(odd)  { background: #ffffff; }

/* Equivalentes com fórmula An+B */
tr:nth-child(2n)   { background: #f5f5f5; } /* mesmo que even */
tr:nth-child(2n+1) { background: #ffffff; } /* mesmo que odd  */

/* Outros exemplos com fórmula */
li:nth-child(3)    { font-weight: bold; }    /* apenas o 3º item */
li:nth-child(3n)   { color: #2563eb; }       /* a cada 3 itens: 3, 6, 9... */
li:nth-child(n+4)  { display: none; }        /* oculta do 4º em diante */

/* Botão que não está desabilitado */
button:not(:disabled):hover { opacity: 0.85; }

/* Card que contém imagem */
.card:has(img) { padding: 0; }

/* Focus visível apenas para teclado */
button:focus-visible { outline: 2px solid blue; }
```

### Pseudo-elementos

| Pseudo-elemento | Uso |
|---|---|
| `::before` | Insere conteúdo antes do elemento |
| `::after` | Insere conteúdo após o elemento |
| `::placeholder` | Estiliza o placeholder de inputs |
| `::selection` | Texto selecionado pelo usuário |
| `::first-line` | Primeira linha de um parágrafo |
| `::marker` | Marcadores de listas (`<li>`) |

```css
.required::after {
  content: " *";
  color: red;
}

input::placeholder {
  color: #999;
  font-style: italic;
}
```

### Especificidade

A especificidade determina qual regra CSS vence quando há conflito. Calculada como `(a, b, c)`:

| Tipo de seletor | Peso |
|---|---|
| `!important` | Sobrescreve tudo (evitar) |
| Estilo inline (`style=""`) | 1,0,0,0 |
| ID (`#id`) | 0,1,0,0 |
| Classe, atributo, pseudo-classe | 0,0,1,0 |
| Tag, pseudo-elemento | 0,0,0,1 |
| Universal (`*`), combinadores | 0,0,0,0 |

> **Dica:** Preferir classes em vez de IDs para estilização. Manter especificidade baixa para facilitar manutenção.

### Principais propriedades CSS

#### Box Model

```css
.box {
  /* Tamanho */
  width: 300px;
  height: 200px;
  min-width: 100px;
  max-width: 100%;

  /* Espaçamento interno */
  padding: 1rem;                   /* todos os lados */
  padding: 1rem 2rem;              /* vertical | horizontal */
  padding: 1rem 2rem 1.5rem 2rem;  /* top right bottom left */
  padding-inline: 2rem;            /* esquerda e direita (lógico) */
  padding-block: 1rem;             /* cima e baixo (lógico) */

  /* Borda */
  border: 1px solid #ccc;
  border-radius: 8px;
  border-top: 2px solid blue;

  /* Espaçamento externo */
  margin: 1rem auto;       /* centraliza horizontalmente */
  margin-block: 2rem;

  /* Modelo de caixa recomendado */
  box-sizing: border-box;  /* padding e border incluídos no width/height */

  /* Contorno (não ocupa espaço, não afeta layout) */
  outline: 2px solid blue;  /* usado em foco — não remover sem substituto acessível */
}
```

> **Boa prática global:**
> ```css
> *, *::before, *::after { box-sizing: border-box; }
> ```

#### Tipografia

```css
.text {
  font-family: 'Inter', sans-serif;
  font-size: 1rem;          /* 16px por padrão */
  font-size: clamp(1rem, 2.5vw, 1.5rem); /* fluido entre min e max */
  font-weight: 400;         /* 100-900 */
  font-style: italic;
  line-height: 1.5;         /* sem unidade: relativo ao font-size */
  letter-spacing: 0.05em;
  text-align: left | center | right | justify;
  text-decoration: none | underline | line-through;
  text-transform: uppercase | lowercase | capitalize;
  color: #333;
  white-space: nowrap;      /* impede quebra de linha */
  overflow: hidden;
  text-overflow: ellipsis;  /* trunca com "..." */

  /* Shorthand: font-style font-weight font-size/line-height font-family */
  font: italic 700 1rem/1.5 'Inter', sans-serif;
}
```

#### Cores e fundos

```css
.element {
  /* Cores */
  color: #2563eb;
  color: rgb(37, 99, 235);
  color: hsl(221, 83%, 53%);
  color: oklch(55% 0.2 260); /* espaço de cor moderno */

  /* Fundo */
  background-color: #f8fafc;
  background-image: url('imagem.jpg');
  background-size: cover | contain | 50%;
  background-position: center;
  background-repeat: no-repeat;
  background: linear-gradient(135deg, #667eea, #764ba2);

  /* Shorthand: color image position/size repeat attachment origin clip */
  background: #f8fafc url('imagem.jpg') center/cover no-repeat;
}
```

#### Posicionamento

```css
.positioned {
  position: static;    /* padrão — sem posicionamento especial */
  position: relative;  /* referência para filhos absolute */
  position: absolute;  /* remove do fluxo, relativo ao pai com position */
  position: fixed;     /* fixo ao viewport */
  position: sticky;    /* relativo até atingir ponto de "cola" */

  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  inset: 0;       /* shorthand para top/right/bottom/left simultaneamente */
  z-index: 10;
}
```

#### Display e visibilidade

```css
.el {
  display: block | inline | inline-block | flex | grid | none;
  visibility: visible | hidden;  /* hidden: oculto mas ocupa espaço */
  opacity: 0;                    /* transparente mas ocupa espaço */
}
```

#### Overflow e dimensões

```css
.scroll-area {
  overflow: auto | scroll | hidden | visible;
  overflow-x: hidden;
  overflow-y: auto;
  max-height: 400px;
  aspect-ratio: 16 / 9;  /* mantém proporção */
  object-fit: cover | contain | fill;  /* para img/video */
}
```

#### Transições e animações

```css
/* Transição */
.btn {
  transition: background-color 0.2s ease, transform 0.1s ease;
}
.btn:hover {
  background-color: #1d4ed8;
  transform: translateY(-2px);
}

/* Animação */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.modal {
  animation: fadeIn 0.3s ease forwards;
}
```

#### Sombras e efeitos

```css
.card {
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  filter: blur(4px) | brightness(0.9) | grayscale(100%);
  backdrop-filter: blur(10px); /* blur no fundo — efeito vidro */
}

.text {
  text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
}
```

---

## 8. CSS Moderno

### Flexbox

> MDN: [Flexbox](https://developer.mozilla.org/pt-BR/docs/Learn_web_development/Core/CSS_layout/Flexbox) · [flex (propriedade)](https://developer.mozilla.org/pt-BR/docs/Web/CSS/flex)

Flexbox é o sistema de layout unidimensional do CSS (uma direção: linha ou coluna).

```css
.container {
  display: flex;
  flex-direction: row | column | row-reverse | column-reverse;
  flex-wrap: nowrap | wrap;
  justify-content: flex-start | center | flex-end | space-between | space-around | space-evenly;
  align-items: stretch | flex-start | center | flex-end | baseline;
  align-content: flex-start | center | space-between; /* múltiplas linhas */
  gap: 1rem;               /* espaço entre itens (row-gap e column-gap) */
  gap: 1rem 2rem;          /* row-gap | column-gap */
}

.item {
  flex: 1;                 /* cresce para preencher espaço disponível */
  flex: 0 0 200px;         /* grow shrink basis — largura fixa de 200px */
  align-self: center;      /* sobrescreve align-items para este item */
  order: 2;                /* altera ordem visual sem mudar o HTML */
}
```

**Casos de uso comuns:**
```css
/* Centralizar elemento */
.center { display: flex; justify-content: center; align-items: center; }

/* Navbar com logo à esquerda e menu à direita */
.navbar { display: flex; justify-content: space-between; align-items: center; }

/* Cards em linha com quebra automática */
.cards { display: flex; flex-wrap: wrap; gap: 1.5rem; }
.card { flex: 1 1 280px; } /* cresce, encolhe, mínimo 280px */
```

### CSS Grid

> MDN: [CSS Grid Layout](https://developer.mozilla.org/pt-BR/docs/Web/CSS/CSS_grid_layout) · [grid-template-areas](https://developer.mozilla.org/pt-BR/docs/Web/CSS/grid-template-areas)

Grid é o sistema de layout bidimensional (linhas e colunas simultâneas).

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 1fr; /* coluna fixa + 2 flexíveis */
  grid-template-columns: repeat(3, 1fr); /* 3 colunas iguais */
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); /* responsivo automático */
  grid-template-rows: auto;
  gap: 1.5rem;
}

/* Posicionamento de itens */
.item {
  grid-column: 1 / 3;     /* ocupa da coluna 1 até a 3 */
  grid-column: span 2;    /* ocupa 2 colunas a partir da posição atual */
  grid-row: 1 / 3;        /* ocupa 2 linhas */
}
```

**Layout com named areas:**
```css
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

header { grid-area: header; }
aside  { grid-area: sidebar; }
main   { grid-area: main; }
footer { grid-area: footer; }
```

**Grid vs Flexbox — quando usar cada um:**
| Flexbox | Grid |
|---|---|
| Layout em uma direção | Layout em duas direções |
| Componentes: navbar, botões, cards em linha | Layouts de página, dashboards, galerias |
| Quando o tamanho vem do conteúdo | Quando o tamanho vem do layout |

### Variáveis CSS (Custom Properties)

> MDN: [Using CSS custom properties (variables)](https://developer.mozilla.org/pt-BR/docs/Web/CSS/Using_CSS_custom_properties)

```css
/* Definição — geralmente no :root para escopo global */
:root {
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-text: #1e293b;
  --color-bg: #f8fafc;
  --font-size-base: 1rem;
  --border-radius: 8px;
  --shadow-sm: 0 1px 3px rgb(0 0 0 / 0.12);
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 2rem;
}

/* Uso */
.btn-primary {
  background-color: var(--color-primary);
  border-radius: var(--border-radius);
  padding: var(--spacing-sm) var(--spacing-md);
}

.btn-primary:hover {
  background-color: var(--color-primary-hover);
}

/* Sobrescrita com escopo (tema escuro, componente específico) */
[data-theme="dark"] {
  --color-text: #f1f5f9;
  --color-bg: #0f172a;
}
```

**Valor padrão (fallback):**
```css
color: var(--color-accent, #e11d48); /* usa #e11d48 se a variável não existir */
```

### Property Nesting

> MDN: [CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting/Using_CSS_nesting)

O CSS nativo (a partir de 2023/2024) suporta aninhamento de regras, sem precisar de SCSS.

```css
/* CSS nativo com nesting */
.card {
  background: white;
  border-radius: 8px;

  /* Elemento filho */
  .card-title {
    font-size: 1.25rem;
    font-weight: 600;
  }

  /* Pseudo-classe no próprio .card */
  &:hover {
    box-shadow: 0 4px 12px rgb(0 0 0 / 0.15);
  }

  /* Modificador (BEM) */
  &--featured {
    border: 2px solid var(--color-primary);
  }

  /* Media query aninhada */
  @media (min-width: 768px) {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}
```

> **Suporte:** Chrome 112+, Firefox 117+, Safari 16.5+ — ampla compatibilidade em 2025.

### Media Queries

> MDN: [Using media queries](https://developer.mozilla.org/pt-BR/docs/Web/CSS/CSS_media_queries/Using_media_queries)

```css
/* Largura da tela */
@media (min-width: 768px) { ... }   /* mobile-first (recomendado) */
@media (max-width: 767px) { ... }   /* desktop-first (evitar) */

/* Faixa de largura */
@media (768px <= width < 1200px) { ... }  /* sintaxe moderna */

/* Orientação */
@media (orientation: landscape) { ... }
@media (orientation: portrait) { ... }

/* Preferência do sistema — dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-text: #f1f5f9;
  }
}

/* Preferência por menos movimento (acessibilidade) */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* Impressão */
@media print {
  nav, footer { display: none; }
  body { font-size: 12pt; color: black; }
}
```

### Container Queries

> MDN: [CSS Container Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)

Container Queries permitem aplicar estilos baseados no **tamanho do contêiner pai**, não do viewport. Ideal para componentes reutilizáveis em diferentes contextos.

```css
/* 1. Definir o contêiner */
.card-wrapper {
  container-type: inline-size; /* monitora largura */
  container-name: card;        /* nome opcional */
}

/* 2. Estilos baseados no contêiner */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 120px 1fr;
  }
}

@container (min-width: 600px) {
  .card-title {
    font-size: 1.5rem;
  }
}
```

**Por que usar em vez de media queries?**
- Um mesmo componente `.card` se adapta automaticamente, independente de onde for usado (sidebar estreita ou conteúdo principal largo).
- Não é necessário alterar o CSS do componente ao movê-lo de lugar.

> **Suporte:** Chrome 105+, Firefox 110+, Safari 16+ — disponível para uso em produção desde 2023.

### `@layer` — Cascata em Camadas

> MDN: [@layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

Permite organizar e controlar a prioridade de estilos sem depender de especificidade.

```css
/* Define a ordem das camadas (menor prioridade primeiro) */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

@layer base {
  body { font-family: sans-serif; }
}

@layer components {
  .btn { padding: 0.5rem 1rem; border-radius: 4px; }
}

@layer utilities {
  .mt-auto { margin-top: auto; }
}
```

### Outras propriedades modernas úteis

```css
/* Scroll suave */
html { scroll-behavior: smooth; }

/* Scroll snap — carrossel sem JS */
.scroll-container {
  overflow-x: scroll;
  scroll-snap-type: x mandatory;
}
.slide {
  scroll-snap-align: start;
  flex: 0 0 100%;
}

/* Tamanho de fonte fluido */
font-size: clamp(1rem, 1rem + 1vw, 1.5rem);

/* Espaçamento lógico (respeita LTR/RTL) */
margin-inline: auto;      /* margin-left + margin-right */
padding-block: 1rem;      /* padding-top + padding-bottom */
border-inline-start: 3px solid blue; /* borda à esquerda (LTR) */

/* Cor com função moderna */
color: oklch(55% 0.2 250deg);
background: color-mix(in oklch, var(--color-primary) 80%, white);
```

---

## 9. SCSS

> Documentação oficial: [Sass Documentation](https://sass-lang.com/documentation/) · MDN: [CSS preprocessors](https://developer.mozilla.org/en-US/docs/Glossary/CSS_preprocessor)

**SCSS** (Sassy CSS) é um pré-processador CSS que adiciona funcionalidades ao CSS e compila para CSS padrão. Ainda é muito usado, mesmo com o CSS nativo evoluindo.

> **Nota:** Com o suporte nativo a variáveis, nesting e outras features no CSS moderno, parte do SCSS pode ser substituída. No entanto, o SCSS ainda oferece funcionalidades sem equivalente nativo, como `@mixin`, `@each`, `@function` e a organização em partials.

### Variáveis SCSS

```scss
// Variáveis SCSS (compiladas — não existem em tempo de execução)
$color-primary: #2563eb;
$font-size-base: 16px;
$border-radius: 8px;

.btn {
  background: $color-primary;
  font-size: $font-size-base;
  border-radius: $border-radius;
}
```

> **SCSS vs CSS:** Variáveis SCSS são resolvidas em tempo de compilação. Variáveis CSS (`--var`) existem no navegador e podem ser alteradas em tempo de execução (JavaScript, dark mode, etc.). Prefira **variáveis CSS** para valores que mudam dinamicamente e **variáveis SCSS** para valores de configuração/build.

### Nesting

```scss
.nav {
  display: flex;
  gap: 1rem;

  a {
    color: #333;
    text-decoration: none;

    &:hover {
      color: $color-primary;
    }

    &.active {
      font-weight: 700;
      color: $color-primary;
    }
  }

  &--vertical {
    flex-direction: column;
  }
}
```

### Partials e @use / @forward

Organiza o SCSS em múltiplos arquivos.

```
styles/
├── _variables.scss
├── _reset.scss
├── _typography.scss
├── _buttons.scss
├── _forms.scss
└── main.scss
```

```scss
// _variables.scss
$color-primary: #2563eb;
$spacing-unit: 0.5rem;

// main.scss
@use 'variables' as v;        // importa com namespace
@use 'reset';
@use 'typography';
@use 'buttons';

.element {
  color: v.$color-primary;
}
```

> **Atenção:** `@import` (antigo) está depreciado. Usar `@use` e `@forward`.

### Mixins

Blocos de código reutilizáveis, podendo receber parâmetros.

```scss
// Definição
@mixin flex-center($direction: row) {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: $direction;
}

@mixin responsive($breakpoint) {
  @if $breakpoint == md {
    @media (min-width: 768px) { @content; }
  } @else if $breakpoint == lg {
    @media (min-width: 1200px) { @content; }
  }
}

// Uso
.hero {
  @include flex-center(column);
  min-height: 100vh;

  @include responsive(md) {
    flex-direction: row;
  }
}

.container {
  padding: 1rem;

  @include responsive(lg) {
    max-width: 1140px;
    margin-inline: auto;
  }
}
```

### @extend

Compartilha um conjunto de propriedades entre seletores.

```scss
%button-base {
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1.25rem;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: opacity 0.2s;

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.btn-primary {
  @extend %button-base;
  background: $color-primary;
  color: white;
}

.btn-secondary {
  @extend %button-base;
  background: #e2e8f0;
  color: #334155;
}
```

> **Dica:** Preferir `@mixin` a `@extend` na maioria dos casos. O `@extend` gera seletores agrupados que podem criar problemas em media queries.

### Funções

```scss
// Função embutida úteis
color: darken($color-primary, 10%);
color: lighten($color-primary, 20%);
color: rgba($color-primary, 0.8);
$complementar: complement($color-primary);

// Função personalizada
@function rem($px) {
  @return $px / 16px * 1rem;
}

.titulo {
  font-size: rem(24px); // → 1.5rem
}
```

> **Nota:** `darken()` e `lighten()` do SCSS são compilados. Para manipulação dinâmica de cores, usar `color-mix()` do CSS moderno.

### @each e @for — Geração de classes

```scss
// @each com lista
$sizes: sm, md, lg, xl;
$values: 0.5rem, 1rem, 1.5rem, 2rem;

@each $size, $value in (sm: 0.5rem, md: 1rem, lg: 1.5rem, xl: 2rem) {
  .gap-#{$size} { gap: $value; }
  .p-#{$size}   { padding: $value; }
  .m-#{$size}   { margin: $value; }
}

// @for
@for $i from 1 through 12 {
  .col-#{$i} {
    width: percentage($i / 12);
  }
}

// @each com mapa de cores
$theme-colors: (
  'primary': #2563eb,
  'success': #16a34a,
  'danger':  #dc2626,
  'warning': #d97706,
);

@each $name, $color in $theme-colors {
  .text-#{$name}   { color: $color; }
  .bg-#{$name}     { background-color: $color; }
  .border-#{$name} { border-color: $color; }
}
```

### Interpolação

```scss
$propriedade: margin;
$lado: top;

.elemento {
  #{$propriedade}-#{$lado}: 1rem; // → margin-top: 1rem
}
```

### Estrutura de projeto SCSS recomendada (7-1 Pattern simplificado)

```
scss/
├── abstracts/
│   ├── _variables.scss   // variáveis
│   ├── _mixins.scss      // mixins e funções
│   └── _placeholders.scss // %extends
├── base/
│   ├── _reset.scss
│   └── _typography.scss
├── components/
│   ├── _buttons.scss
│   ├── _cards.scss
│   └── _forms.scss
├── layout/
│   ├── _header.scss
│   ├── _footer.scss
│   └── _grid.scss
├── pages/
│   └── _home.scss
└── main.scss             // apenas @use dos partials
```

---

## Recursos Avançados de CSS

### Transitions e Animações CSS

> MDN: [transition](https://developer.mozilla.org/pt-BR/docs/Web/CSS/transition) · [animation](https://developer.mozilla.org/pt-BR/docs/Web/CSS/animation) · [@keyframes](https://developer.mozilla.org/pt-BR/docs/Web/CSS/@keyframes)

#### transition

Define a transição suave entre dois estados de uma propriedade CSS. Disparada automaticamente quando o valor da propriedade muda (por `:hover`, troca de classe, etc.).

```css
.btn {
  background: #2563eb;
  transform: translateY(0);
  box-shadow: none;

  /* propriedade | duração | easing | delay */
  transition: background 0.2s ease,
              transform  0.15s ease,
              box-shadow 0.2s ease;
}

.btn:hover {
  background: #1d4ed8;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgb(37 99 235 / 0.4);
}

/* Transicionar tudo — usar com cuidado (pode afetar performance) */
.el { transition: all 0.3s ease; }

/* Delay: útil para sequenciar a entrada de múltiplos elementos */
.item:nth-child(1) { transition-delay: 0ms; }
.item:nth-child(2) { transition-delay: 100ms; }
.item:nth-child(3) { transition-delay: 200ms; }
```

**Funções de easing:**

| Valor | Comportamento |
|---|---|
| `ease` | Suave no início e no fim (padrão) |
| `linear` | Velocidade constante |
| `ease-in` | Começa devagar, termina rápido |
| `ease-out` | Começa rápido, termina devagar |
| `ease-in-out` | Suave no início e no fim, mais pronunciado |
| `cubic-bezier(x1,y1,x2,y2)` | Curva personalizada |
| `steps(n)` | Transição em degraus (efeito typewriter, sprites) |

#### @keyframes e animation

Permite definir animações com múltiplos estágios e rodá-las de forma contínua ou repetida, sem depender de mudança de estado.

```css
/* Definição dos keyframes */
@keyframes surgir {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulsar {
  0%   { transform: scale(1); }
  50%  { transform: scale(1.05); }
  100% { transform: scale(1); }
}

/* Aplicação */
.card {
  /* nome | duração | easing | delay | iterações | direção | fill-mode */
  animation: surgir 0.4s ease-out 0s 1 normal forwards;

  /* Forma abreviada legível */
  animation: surgir 0.4s ease-out forwards;
}

.badge {
  animation: pulsar 1.5s ease-in-out infinite;
}
```

**Propriedades individuais de `animation`:**

```css
.el {
  animation-name: surgir;
  animation-duration: 0.4s;
  animation-timing-function: ease-out;
  animation-delay: 0.1s;
  animation-iteration-count: 1;        /* ou infinite */
  animation-direction: normal;         /* reverse | alternate | alternate-reverse */
  animation-fill-mode: forwards;       /* none | backwards | both */
  animation-play-state: running;       /* paused — controla via JS */
}
```

**Controle via JavaScript:**

```js
const el = document.querySelector('.animado');

// Pausar e retomar
el.style.animationPlayState = 'paused';
el.style.animationPlayState = 'running';

// Reiniciar animação (truque: remove e readiciona a classe)
el.classList.remove('animado');
el.offsetWidth; // força reflow
el.classList.add('animado');

// Detectar fim da animação
el.addEventListener('animationend', () => {
  console.log('Animação concluída');
});

// Detectar fim da transição
el.addEventListener('transitionend', (e) => {
  console.log('Propriedade que transitou:', e.propertyName);
});
```

**Boas práticas de performance:**
- Preferir animar `transform` e `opacity` — rodam na GPU e não causam reflow.
- Evitar animar `width`, `height`, `top`, `left`, `margin` — causam reflow e são mais custosas.
- Respeitar `prefers-reduced-motion` para acessibilidade:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Referências

- [MDN Web Docs](https://developer.mozilla.org) — referência completa de HTML e CSS
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) — diretrizes de acessibilidade
- [Nielsen Norman Group](https://www.nngroup.com/articles/ten-usability-heuristics/) — heurísticas de Nielsen
- [Can I Use](https://caniuse.com) — suporte de recursos CSS/HTML por navegador
- [CSS Tricks — Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS Tricks — Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Sass Documentation](https://sass-lang.com/documentation/)
- [Figma](https://www.figma.com) — ferramenta de design
