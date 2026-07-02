# Gráficos e Conteúdo Visual para Web — Canvas, SVG, WebGL, Three.js, A-Frame e Bibliotecas de Visualização

Referência consolidada sobre tecnologias gráficas para web, cobrindo desde gráficos 2D (Canvas API, SVG) e visualização de dados (D3.js, Chart.js) até renderização 3D (WebGL, Three.js, Babylon.js), Realidade Virtual e Aumentada (A-Frame, AR.js), engines 2D de alta performance (Pixi.js) e integração com React (React Three Fiber) — com pipeline completo de criação de assets 3D no Blender e importação otimizada para web.

> Para fundamentos de JavaScript e TypeScript, consulte [Dicas-Javascript-Typescript.md](../Dicas-Javascript-Typescript.md).
> Para CSS, animações e acessibilidade básica, consulte [Dicas-Desenvolvimento-Web-Frontend.md](../Dicas-Desenvolvimento-Web-Frontend.md).
> Para visualização de dados em dashboards React/Vue/Angular, consulte [Frontend-Design-UI-Avancado.md](../Frontend-Design-UI-Avancado.md).

---

## Sumário

**Parte 1 — Fundamentos de Gráficos 2D**

1. [Visão Geral — Tecnologias Gráficas para Web](#1-visão-geral--tecnologias-gráficas-para-web)
2. [Canvas API (2D)](#2-canvas-api-2d)
3. [SVG (Scalable Vector Graphics)](#3-svg-scalable-vector-graphics)

**Parte 2 — Visualização de Dados**

4. [D3.js — Data-Driven Documents](#4-d3js--data-driven-documents)
5. [Chart.js](#5-chartjs)
6. [Apache ECharts](#6-apache-echarts)

**Parte 3 — Gráficos 2D de Alta Performance**

7. [Pixi.js — Engine 2D com WebGL](#7-pixijs--engine-2d-com-webgl)

**Parte 4 — WebGL e Fundamentos 3D**

8. [WebGL — Fundamentos](#8-webgl--fundamentos)

**Parte 5 — Three.js**

9. [Three.js — Fundamentos](#9-threejs--fundamentos)
10. [Three.js — Tópicos Avançados](#10-threejs--tópicos-avançados)
11. [React Three Fiber (R3F)](#11-react-three-fiber-r3f)

**Parte 6 — Babylon.js**

12. [Babylon.js — Engine 3D Alternativa](#12-babylonjs--engine-3d-alternativa)

**Parte 7 — A-Frame e WebXR**

13. [A-Frame — Realidade Virtual e Aumentada na Web](#13-a-frame--realidade-virtual-e-aumentada-na-web)

**Parte 8 — Pipeline de Assets 3D (Blender e Outros)**

14. [Formatos de Arquivos 3D — Visão Geral](#14-formatos-de-arquivos-3d--visão-geral)
15. [Blender — Modelagem e Exportação para Web](#15-blender--modelagem-e-exportação-para-web)
16. [Importação de Assets no Three.js e A-Frame](#16-importação-de-assets-no-threejs-e-a-frame)

**Parte 9 — Boas Práticas e Referências**

17. [Performance e Otimização Geral](#17-performance-e-otimização-geral)
18. [Acessibilidade em Conteúdo Gráfico](#18-acessibilidade-em-conteúdo-gráfico)
19. [Referências e Leitura Complementar](#19-referências-e-leitura-complementar)

---

**Parte 1 — Fundamentos de Gráficos 2D**

## 1. Visao Geral — Tecnologias Graficas para Web

A web moderna oferece um conjunto robusto de tecnologias para renderizacao de graficos, desde graficos vetoriais escaláveis até renderizacao 3D acelerada por GPU. Cada tecnologia atende a cenarios distintos e entender suas diferencas e fundamental para escolher a ferramenta certa.

As tres tecnologias principais sao:

- **Canvas API (2D)** — renderizacao rasterizada via JavaScript em um bitmap
- **SVG (Scalable Vector Graphics)** — graficos vetoriais declarativos integrados ao DOM
- **WebGL / WebGL2** — renderizacao 2D/3D acelerada por GPU via shaders OpenGL ES

> MDN: [Graficos na Web](https://developer.mozilla.org/pt-BR/docs/Web/Guide/Graphics)

### Canvas vs SVG vs WebGL — Comparativo Detalhado

| Caracteristica | Canvas 2D | SVG | WebGL |
|---|---|---|---|
| **Modelo de renderizacao** | Rasterizado (bitmap). Desenha pixels diretamente; depois de desenhado, o canvas "esquece" o que foi desenhado | Vetorial e declarativo. Cada elemento e um no do DOM que pode ser inspecionado e manipulado | Rasterizado via GPU. Usa shaders (GLSL) para processar vertices e fragmentos |
| **Desempenho com poucos objetos** | Bom | Excelente — o DOM gerencia cada elemento | Overhead de setup alto; nao compensa para poucos objetos |
| **Desempenho com muitos objetos** | Excelente — custo nao cresce com quantidade | Degrada — cada elemento e um no DOM | Excelente — GPU processa milhares de vertices em paralelo |
| **Integracao com DOM** | Nenhuma — o canvas e um unico elemento `<canvas>` | Total — cada forma e um elemento DOM com eventos, CSS, etc. | Nenhuma — o WebGL renderiza em um `<canvas>` |
| **Acessibilidade** | Ruim — conteudo invisivel para leitores de tela sem fallback manual | Boa — elementos podem ter `<title>`, `<desc>`, ARIA | Ruim — mesma limitacao do Canvas |
| **Abordagem de animacao** | Imperativa via `requestAnimationFrame` | Declarativa (SMIL, CSS) ou imperativa (JS) | Imperativa via loop de renderizacao |
| **Resolucao** | Dependente de pixels; requer tratamento para HiDPI | Independente de resolucao — escalona sem perda | Dependente de pixels do canvas, mas GPU suaviza |
| **Casos de uso ideais** | Jogos 2D, editores de imagem, visualizacoes com muitos pontos, filtros de pixel | Icones, logos, infograficos, mapas interativos, graficos com poucos elementos | Jogos 3D, visualizacoes 3D, efeitos visuais complexos, simulacoes |
| **Curva de aprendizado** | Moderada — API imperativa mas direta | Baixa a moderada — marcacao declarativa familiar | Alta — requer conhecimento de shaders GLSL e pipeline grafico |
| **Exportacao** | `toDataURL()` / `toBlob()` para PNG/JPEG | Pode ser serializado como XML/string | Requer `preserveDrawingBuffer` + `toDataURL()` |

### Fluxograma de Decisao

```
Preciso de graficos na web
|
+-- Os graficos sao 3D ou exigem GPU?
|   +-- Sim --> WebGL (ou WebGPU se suporte ja for suficiente)
|   +-- Nao --> Continue
|
+-- Os elementos precisam ser interativos individualmente?
|   (ex: clicar em uma barra de um grafico, hover em um icone)
|   +-- Sim --> Continue
|   |   +-- Sao menos de ~1000 elementos?
|   |   |   +-- Sim --> SVG
|   |   |   +-- Nao --> Canvas (com hit detection manual)
|   +-- Nao --> Continue
|
+-- Preciso manipular pixels diretamente?
|   (ex: filtros de imagem, efeitos bitmap)
|   +-- Sim --> Canvas 2D
|   +-- Nao --> Continue
|
+-- O grafico precisa escalar sem perda de qualidade?
|   (ex: logos, icones, ilustracoes)
|   +-- Sim --> SVG
|   +-- Nao --> Continue
|
+-- Ha muitos objetos animados simultaneamente?
|   (ex: particulas, simulacoes)
|   +-- Sim --> Canvas 2D (ou WebGL para milhares)
|   +-- Nao --> SVG (mais simples de manter)
```

### WebGPU — O Futuro da Renderizacao na Web

O **WebGPU** e a API de proxima geracao que substitui o WebGL. Baseado nos backends modernos (Vulkan, Metal, Direct3D 12), oferece:

- **Compute Shaders** — processamento paralelo generico na GPU (machine learning, simulacoes fisicas)
- **Melhor desempenho** — menos overhead de driver, pipeline mais eficiente
- **API mais moderna** — inspirada em Vulkan/Metal, com gerenciamento explicito de recursos

```javascript
// Verificando suporte ao WebGPU
if (navigator.gpu) {
  const adapter = await navigator.gpu.requestAdapter();
  const device = await adapter.requestDevice();
  console.log('WebGPU disponivel!');
} else {
  console.log('WebGPU nao suportado — usando WebGL como fallback');
}
```

> O WebGPU esta disponivel no Chrome 113+ e Edge 113+. Firefox e Safari possuem implementacoes experimentais. Para producao, mantenha WebGL como fallback.

> MDN: [WebGPU API](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API)

---

## 2. Canvas API (2D)

A Canvas API fornece uma superficie de desenho rasterizado controlada inteiramente por JavaScript. Ao contrario do SVG, o canvas nao mantem um grafo de cena — cada operacao de desenho modifica pixels diretamente no bitmap, e uma vez desenhado, o canvas nao tem "memoria" do que foi desenhado.

> MDN: [Canvas API](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API) · [Tutorial Canvas](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API/Tutorial)

### Fundamentos

O elemento `<canvas>` cria uma area retangular de desenho. Por padrao, tem **300x150 pixels**. O tamanho do canvas deve ser definido pelos atributos `width` e `height` (nao por CSS, que apenas estica a imagem).

```html
<!-- Tamanho padrao: 300x150 -->
<canvas id="meuCanvas"></canvas>

<!-- Tamanho definido por atributos (correto) -->
<canvas id="meuCanvas" width="800" height="600"></canvas>
```

> **Importante:** definir tamanho via CSS (`style="width: 800px"`) estica o bitmap padrao de 300x150, causando distorcao. Sempre use os atributos `width` e `height`.

O contexto de renderizacao 2D e obtido com `getContext('2d')`:

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

if (!ctx) {
  console.error('Canvas 2D nao suportado neste navegador');
}
```

**Sistema de coordenadas:** a origem (0, 0) fica no canto superior esquerdo. O eixo X cresce para a direita e o eixo Y cresce para baixo.

```
(0,0) -------- X+ --------->
  |
  |
  Y+
  |
  |
  v
```

Exemplo completo de setup basico:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Canvas Basico</title>
  <style>
    canvas {
      border: 1px solid #333;
      display: block;
      margin: 20px auto;
    }
  </style>
</head>
<body>
  <canvas id="meuCanvas" width="600" height="400"></canvas>
  <script>
    const canvas = document.getElementById('meuCanvas');
    const ctx = canvas.getContext('2d');

    // Desenho simples para verificar que funciona
    ctx.fillStyle = '#3498db';
    ctx.fillRect(50, 50, 200, 100);

    ctx.strokeStyle = '#e74c3c';
    ctx.lineWidth = 3;
    ctx.strokeRect(300, 50, 200, 100);

    ctx.fillStyle = '#2c3e50';
    ctx.font = '24px Arial';
    ctx.fillText('Canvas funcionando!', 150, 250);
  </script>
</body>
</html>
```

### Desenho de Formas

#### Retangulos

O canvas oferece tres metodos diretos para retangulos — sao as unicas formas primitivas do canvas:

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

// fillRect(x, y, largura, altura) — retangulo preenchido
ctx.fillStyle = '#3498db';
ctx.fillRect(20, 20, 150, 100);

// strokeRect(x, y, largura, altura) — contorno do retangulo
ctx.strokeStyle = '#e74c3c';
ctx.lineWidth = 3;
ctx.strokeRect(200, 20, 150, 100);

// clearRect(x, y, largura, altura) — apaga uma area retangular
// Util para "recortar" partes do canvas
ctx.fillStyle = '#2ecc71';
ctx.fillRect(400, 20, 150, 100);
ctx.clearRect(430, 40, 90, 60); // "furo" no retangulo verde
```

#### Arcos e Circulos

Arcos sao desenhados com `arc()` dentro de um path:

```javascript
// arc(x, y, raio, anguloInicial, anguloFinal, antiHorario?)
// Angulos em radianos: 0 = direita, Math.PI/2 = baixo, Math.PI = esquerda

const ctx = canvas.getContext('2d');

// Circulo completo
ctx.beginPath();
ctx.arc(100, 100, 50, 0, Math.PI * 2);
ctx.fillStyle = '#9b59b6';
ctx.fill();

// Semicirculo
ctx.beginPath();
ctx.arc(250, 100, 50, 0, Math.PI); // 0 a PI = semicirculo inferior
ctx.fillStyle = '#e67e22';
ctx.fill();

// Arco (fatia — tipo grafico de pizza)
ctx.beginPath();
ctx.moveTo(400, 100); // Move para o centro
ctx.arc(400, 100, 50, 0, Math.PI * 0.75);
ctx.closePath(); // Fecha o path de volta ao centro
ctx.fillStyle = '#1abc9c';
ctx.fill();
ctx.strokeStyle = '#16a085';
ctx.stroke();

// arcTo(x1, y1, x2, y2, raio) — arco tangente a duas linhas
// Util para cantos arredondados
ctx.beginPath();
ctx.moveTo(50, 250);
ctx.arcTo(150, 250, 150, 350, 30); // canto arredondado
ctx.arcTo(150, 350, 50, 350, 30);
ctx.arcTo(50, 350, 50, 250, 30);
ctx.arcTo(50, 250, 150, 250, 30);
ctx.stroke();
```

#### Paths (Caminhos)

Todas as formas alem de retangulos sao desenhadas usando paths:

```javascript
const ctx = canvas.getContext('2d');

// Triangulo
ctx.beginPath();
ctx.moveTo(100, 20);   // Move a "caneta" sem desenhar
ctx.lineTo(20, 180);   // Linha ate o ponto
ctx.lineTo(180, 180);  // Outra linha
ctx.closePath();        // Fecha o path (volta ao inicio)
ctx.fillStyle = '#e74c3c';
ctx.fill();
ctx.strokeStyle = '#c0392b';
ctx.lineWidth = 2;
ctx.stroke();

// Estrela de 5 pontas
function desenharEstrela(ctx, cx, cy, pontas, raioExterno, raioInterno) {
  ctx.beginPath();
  for (let i = 0; i < pontas * 2; i++) {
    const raio = i % 2 === 0 ? raioExterno : raioInterno;
    const angulo = (Math.PI / pontas) * i - Math.PI / 2;
    const x = cx + Math.cos(angulo) * raio;
    const y = cy + Math.sin(angulo) * raio;
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.closePath();
}

desenharEstrela(ctx, 300, 100, 5, 60, 25);
ctx.fillStyle = '#f1c40f';
ctx.fill();
ctx.strokeStyle = '#f39c12';
ctx.lineWidth = 2;
ctx.stroke();

// Hexagono regular
function desenharPoligono(ctx, cx, cy, lados, raio) {
  ctx.beginPath();
  for (let i = 0; i < lados; i++) {
    const angulo = (Math.PI * 2 / lados) * i - Math.PI / 2;
    const x = cx + Math.cos(angulo) * raio;
    const y = cy + Math.sin(angulo) * raio;
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.closePath();
}

desenharPoligono(ctx, 500, 100, 6, 50);
ctx.fillStyle = '#3498db';
ctx.fill();
```

#### Curvas Bezier

```javascript
const ctx = canvas.getContext('2d');

// quadraticCurveTo(cpX, cpY, endX, endY)
// Curva quadratica — 1 ponto de controle
ctx.beginPath();
ctx.moveTo(50, 300);
ctx.quadraticCurveTo(200, 150, 350, 300); // ponto de controle no topo
ctx.strokeStyle = '#e74c3c';
ctx.lineWidth = 3;
ctx.stroke();

// Visualizando o ponto de controle
ctx.fillStyle = '#e74c3c';
ctx.beginPath();
ctx.arc(200, 150, 5, 0, Math.PI * 2);
ctx.fill();

// bezierCurveTo(cp1X, cp1Y, cp2X, cp2Y, endX, endY)
// Curva cubica — 2 pontos de controle (mais flexivel)
ctx.beginPath();
ctx.moveTo(400, 300);
ctx.bezierCurveTo(420, 100, 580, 500, 700, 300);
ctx.strokeStyle = '#3498db';
ctx.lineWidth = 3;
ctx.stroke();

// Visualizando os pontos de controle
ctx.fillStyle = '#3498db';
ctx.beginPath();
ctx.arc(420, 100, 5, 0, Math.PI * 2);
ctx.fill();
ctx.beginPath();
ctx.arc(580, 500, 5, 0, Math.PI * 2);
ctx.fill();

// Desenhando um coracao com curvas Bezier
ctx.beginPath();
ctx.moveTo(350, 250);
ctx.bezierCurveTo(350, 220, 300, 200, 300, 240);
ctx.bezierCurveTo(300, 280, 350, 310, 350, 340);
ctx.bezierCurveTo(350, 310, 400, 280, 400, 240);
ctx.bezierCurveTo(400, 200, 350, 220, 350, 250);
ctx.fillStyle = '#e74c3c';
ctx.fill();
```

### Estilos

#### Cores e Transparencia

```javascript
const ctx = canvas.getContext('2d');

// fillStyle — cor de preenchimento
ctx.fillStyle = '#3498db';             // Hexadecimal
ctx.fillStyle = 'rgb(52, 152, 219)';   // RGB
ctx.fillStyle = 'rgba(52, 152, 219, 0.5)'; // RGBA com transparencia
ctx.fillStyle = 'hsl(204, 70%, 53%)';  // HSL

// strokeStyle — cor do contorno
ctx.strokeStyle = '#e74c3c';

// globalAlpha — transparencia global (0.0 a 1.0)
ctx.globalAlpha = 0.5;
ctx.fillStyle = '#3498db';
ctx.fillRect(20, 20, 100, 100);  // Todos os desenhos ficam semi-transparentes

ctx.globalAlpha = 1.0; // Restaurar opacidade total
```

#### Gradientes

```javascript
const ctx = canvas.getContext('2d');

// Gradiente linear — createLinearGradient(x0, y0, x1, y1)
const gradLinear = ctx.createLinearGradient(0, 0, 300, 0);
gradLinear.addColorStop(0, '#3498db');
gradLinear.addColorStop(0.5, '#9b59b6');
gradLinear.addColorStop(1, '#e74c3c');
ctx.fillStyle = gradLinear;
ctx.fillRect(20, 20, 300, 80);

// Gradiente radial — createRadialGradient(x0, y0, r0, x1, y1, r1)
const gradRadial = ctx.createRadialGradient(200, 200, 10, 200, 200, 100);
gradRadial.addColorStop(0, '#f1c40f');
gradRadial.addColorStop(1, '#e67e22');
ctx.fillStyle = gradRadial;
ctx.beginPath();
ctx.arc(200, 200, 100, 0, Math.PI * 2);
ctx.fill();

// Gradiente conico — createConicGradient(anguloInicial, cx, cy)
const gradConico = ctx.createConicGradient(0, 450, 200);
gradConico.addColorStop(0, '#e74c3c');
gradConico.addColorStop(0.25, '#f1c40f');
gradConico.addColorStop(0.5, '#2ecc71');
gradConico.addColorStop(0.75, '#3498db');
gradConico.addColorStop(1, '#e74c3c');
ctx.fillStyle = gradConico;
ctx.beginPath();
ctx.arc(450, 200, 80, 0, Math.PI * 2);
ctx.fill();
```

#### Patterns (Padroes)

```javascript
const ctx = canvas.getContext('2d');

// Criando um pattern a partir de uma imagem
const img = new Image();
img.src = 'textura.png';
img.onload = () => {
  // repeat, repeat-x, repeat-y, no-repeat
  const pattern = ctx.createPattern(img, 'repeat');
  ctx.fillStyle = pattern;
  ctx.fillRect(0, 0, canvas.width, canvas.height);
};

// Pattern a partir de outro canvas (util para padroes procedurais)
const patternCanvas = document.createElement('canvas');
patternCanvas.width = 20;
patternCanvas.height = 20;
const pCtx = patternCanvas.getContext('2d');

// Desenha um padrao xadrez
pCtx.fillStyle = '#ecf0f1';
pCtx.fillRect(0, 0, 20, 20);
pCtx.fillStyle = '#bdc3c7';
pCtx.fillRect(0, 0, 10, 10);
pCtx.fillRect(10, 10, 10, 10);

const xadrez = ctx.createPattern(patternCanvas, 'repeat');
ctx.fillStyle = xadrez;
ctx.fillRect(0, 0, canvas.width, canvas.height);
```

#### Estilos de Linha

```javascript
const ctx = canvas.getContext('2d');

// lineWidth — espessura
ctx.lineWidth = 1;   // Padrao
ctx.lineWidth = 5;
ctx.lineWidth = 0.5; // Linhas finas

// lineCap — terminacao da linha: butt (padrao), round, square
['butt', 'round', 'square'].forEach((cap, i) => {
  ctx.beginPath();
  ctx.lineCap = cap;
  ctx.lineWidth = 15;
  ctx.moveTo(50, 50 + i * 50);
  ctx.lineTo(300, 50 + i * 50);
  ctx.stroke();
});

// lineJoin — juncao entre linhas: miter (padrao), round, bevel
['miter', 'round', 'bevel'].forEach((join, i) => {
  ctx.beginPath();
  ctx.lineJoin = join;
  ctx.lineWidth = 15;
  ctx.moveTo(350, 30 + i * 80);
  ctx.lineTo(420, 80 + i * 80);
  ctx.lineTo(490, 30 + i * 80);
  ctx.stroke();
});

// setLineDash — linhas tracejadas
ctx.beginPath();
ctx.setLineDash([10, 5]);        // 10px tracejado, 5px espaco
ctx.moveTo(50, 250);
ctx.lineTo(550, 250);
ctx.stroke();

ctx.beginPath();
ctx.setLineDash([20, 5, 5, 5]);  // Padrao mais complexo
ctx.moveTo(50, 280);
ctx.lineTo(550, 280);
ctx.stroke();

ctx.setLineDash([]); // Restaurar linha solida

// lineDashOffset — deslocamento do padrao (util para animacao)
ctx.lineDashOffset = 5;
```

#### Sombras

```javascript
const ctx = canvas.getContext('2d');

ctx.shadowColor = 'rgba(0, 0, 0, 0.5)'; // Cor da sombra
ctx.shadowBlur = 10;                      // Desfoque da sombra
ctx.shadowOffsetX = 5;                    // Deslocamento horizontal
ctx.shadowOffsetY = 5;                    // Deslocamento vertical

ctx.fillStyle = '#3498db';
ctx.fillRect(50, 50, 200, 100);

// Sombra colorida (efeito neon)
ctx.shadowColor = '#3498db';
ctx.shadowBlur = 20;
ctx.shadowOffsetX = 0;
ctx.shadowOffsetY = 0;
ctx.fillStyle = '#2980b9';
ctx.fillRect(50, 200, 200, 100);

// Remover sombra para desenhos seguintes
ctx.shadowColor = 'transparent';
ctx.shadowBlur = 0;
ctx.shadowOffsetX = 0;
ctx.shadowOffsetY = 0;
```

### Texto no Canvas

```javascript
const ctx = canvas.getContext('2d');

// Propriedades de fonte (mesma sintaxe do CSS font shorthand)
ctx.font = '32px Arial';
ctx.font = 'bold 24px "Segoe UI", sans-serif';
ctx.font = 'italic 18px Georgia';

// textAlign — alinhamento horizontal: start, end, left, right, center
ctx.textAlign = 'center';

// textBaseline — alinhamento vertical: top, hanging, middle,
//                alphabetic (padrao), ideographic, bottom
ctx.textBaseline = 'middle';

// fillText(texto, x, y, maxLargura?) — texto preenchido
ctx.fillStyle = '#2c3e50';
ctx.font = 'bold 36px Arial';
ctx.textAlign = 'center';
ctx.fillText('Texto Preenchido', canvas.width / 2, 60);

// strokeText(texto, x, y, maxLargura?) — contorno do texto
ctx.strokeStyle = '#e74c3c';
ctx.lineWidth = 2;
ctx.font = 'bold 48px Arial';
ctx.strokeText('Texto Contornado', canvas.width / 2, 140);

// Texto com sombra
ctx.shadowColor = 'rgba(0, 0, 0, 0.3)';
ctx.shadowBlur = 4;
ctx.shadowOffsetX = 2;
ctx.shadowOffsetY = 2;
ctx.fillStyle = '#2c3e50';
ctx.font = '28px Arial';
ctx.fillText('Texto com Sombra', canvas.width / 2, 220);
ctx.shadowColor = 'transparent';

// measureText — medir a largura do texto (util para posicionamento)
ctx.font = '20px Arial';
const medida = ctx.measureText('Texto de exemplo');
console.log('Largura:', medida.width);
console.log('Altura aprox:', medida.actualBoundingBoxAscent + medida.actualBoundingBoxDescent);

// Texto multilinha (o canvas nao quebra linhas automaticamente)
function desenharTextoMultilinha(ctx, texto, x, y, larguraMax, alturaLinha) {
  const palavras = texto.split(' ');
  let linha = '';

  for (const palavra of palavras) {
    const teste = linha + palavra + ' ';
    const medida = ctx.measureText(teste);
    if (medida.width > larguraMax && linha !== '') {
      ctx.fillText(linha.trim(), x, y);
      linha = palavra + ' ';
      y += alturaLinha;
    } else {
      linha = teste;
    }
  }
  ctx.fillText(linha.trim(), x, y);
}

ctx.font = '16px Arial';
ctx.textAlign = 'left';
ctx.fillStyle = '#333';
desenharTextoMultilinha(
  ctx,
  'Este e um texto longo que sera quebrado automaticamente em varias linhas conforme a largura maxima definida.',
  20, 300, 300, 24
);
```

### Transformacoes

O canvas usa uma **matriz de transformacao** que afeta todas as operacoes de desenho subsequentes. As transformacoes sao cumulativas.

```javascript
const ctx = canvas.getContext('2d');

// translate(x, y) — move a origem do sistema de coordenadas
ctx.translate(100, 100);
ctx.fillStyle = '#3498db';
ctx.fillRect(0, 0, 80, 80); // Desenhado em (100, 100)

// rotate(angulo) — rotaciona em radianos ao redor da origem ATUAL
// Para rotacionar ao redor do centro de um objeto:
// 1. translate ate o centro
// 2. rotate
// 3. desenhe centralizado
ctx.save();
ctx.translate(300, 100);           // Move para o centro desejado
ctx.rotate(Math.PI / 4);          // Rotaciona 45 graus
ctx.fillStyle = '#e74c3c';
ctx.fillRect(-40, -40, 80, 80);   // Desenha centralizado na origem
ctx.restore();

// scale(sx, sy) — escala (1 = normal, 0.5 = metade, 2 = dobro)
ctx.save();
ctx.translate(500, 100);
ctx.scale(1.5, 0.8);              // Estica horizontalmente, comprime verticalmente
ctx.fillStyle = '#2ecc71';
ctx.fillRect(-40, -40, 80, 80);
ctx.restore();
```

#### save() e restore()

O estado do canvas inclui: transformacoes, estilos (fillStyle, strokeStyle, etc.), clipping, globalAlpha, globalCompositeOperation, entre outros. `save()` empilha o estado atual e `restore()` desempilha.

```javascript
const ctx = canvas.getContext('2d');

ctx.fillStyle = '#3498db';
ctx.globalAlpha = 1.0;

ctx.save(); // Salva estado original
  ctx.fillStyle = '#e74c3c';
  ctx.globalAlpha = 0.5;
  ctx.translate(100, 100);
  ctx.fillRect(0, 0, 80, 80); // Vermelho, semi-transparente, deslocado

  ctx.save(); // Salva estado modificado
    ctx.fillStyle = '#2ecc71';
    ctx.rotate(Math.PI / 6);
    ctx.fillRect(0, 0, 80, 80); // Verde, rotacionado, ainda deslocado
  ctx.restore(); // Volta para vermelho semi-transparente

  ctx.fillRect(100, 0, 80, 80); // Vermelho, semi-transparente
ctx.restore(); // Volta para azul, opaco, sem translacao

ctx.fillRect(0, 0, 80, 80); // Azul, opaco, posicao original
```

#### Transformacoes de Matriz

```javascript
const ctx = canvas.getContext('2d');

// setTransform(a, b, c, d, e, f) — substitui a matriz atual
// | a  c  e |     a = escala horizontal
// | b  d  f |     b = inclinacao vertical
// | 0  0  1 |     c = inclinacao horizontal
//                 d = escala vertical
//                 e = translacao horizontal
//                 f = translacao vertical

// Equivalente a translate(100, 50) + scale(2, 1)
ctx.setTransform(2, 0, 0, 1, 100, 50);
ctx.fillStyle = '#3498db';
ctx.fillRect(0, 0, 80, 80);

// resetTransform() — volta a matriz identidade
ctx.resetTransform();

// transform(a, b, c, d, e, f) — multiplica pela matriz atual (cumulativo)
ctx.transform(1, 0.2, 0.2, 1, 0, 0); // Inclinacao (skew)
ctx.fillStyle = '#e74c3c';
ctx.fillRect(300, 50, 80, 80);

// Usando DOMMatrix para transformacoes mais complexas
const matrix = new DOMMatrix();
matrix.translateSelf(400, 200);
matrix.rotateSelf(30);
matrix.scaleSelf(1.5, 1.5);
ctx.setTransform(matrix);
ctx.fillStyle = '#2ecc71';
ctx.fillRect(-40, -40, 80, 80);
ctx.resetTransform();
```

### Composicao e Clipping

#### globalCompositeOperation

Define como novos desenhos interagem com os pixels ja existentes no canvas:

```javascript
const ctx = canvas.getContext('2d');

// Lista dos modos mais uteis:
// 'source-over'      — padrao: novo sobre existente
// 'source-in'        — novo apenas onde ja existe conteudo
// 'source-out'       — novo apenas onde NAO existe conteudo
// 'source-atop'      — novo sobre existente, apenas onde existe
// 'destination-over' — existente sobre novo
// 'destination-in'   — existente apenas onde novo seria desenhado
// 'destination-out'  — existente apenas onde novo NAO seria desenhado
// 'lighter'          — cores somadas (efeito brilho)
// 'multiply'         — cores multiplicadas (escurece)
// 'screen'           — inverso de multiply (clareia)
// 'xor'              — transparente onde ambos se sobrepoem

// Exemplo: efeito de mascara com 'source-in'
// 1. Desenha a forma que sera a mascara
ctx.fillStyle = '#3498db';
ctx.beginPath();
ctx.arc(200, 200, 100, 0, Math.PI * 2);
ctx.fill();

// 2. Aplica composicao
ctx.globalCompositeOperation = 'source-in';

// 3. Desenha o conteudo que sera mascarado — so aparece dentro do circulo
const img = new Image();
img.src = 'foto.jpg';
img.onload = () => {
  ctx.drawImage(img, 50, 50, 300, 300);
  ctx.globalCompositeOperation = 'source-over'; // Restaurar
};
```

Exemplo visual dos modos mais comuns:

```javascript
const modos = [
  'source-over', 'source-in', 'source-out', 'source-atop',
  'destination-over', 'destination-in', 'destination-out', 'destination-atop',
  'lighter', 'multiply', 'screen', 'xor'
];

modos.forEach((modo, i) => {
  const col = i % 4;
  const row = Math.floor(i / 4);
  const x = col * 160 + 20;
  const y = row * 160 + 40;

  // Cria canvas temporario para cada modo
  const temp = document.createElement('canvas');
  temp.width = 140;
  temp.height = 140;
  const tCtx = temp.getContext('2d');

  // Desenho existente (azul)
  tCtx.fillStyle = '#3498db';
  tCtx.fillRect(10, 10, 80, 80);

  // Aplica o modo
  tCtx.globalCompositeOperation = modo;

  // Novo desenho (vermelho)
  tCtx.fillStyle = '#e74c3c';
  tCtx.beginPath();
  tCtx.arc(90, 90, 50, 0, Math.PI * 2);
  tCtx.fill();

  // Copia para o canvas principal
  ctx.drawImage(temp, x, y);
  ctx.font = '12px Arial';
  ctx.fillStyle = '#333';
  ctx.textAlign = 'center';
  ctx.fillText(modo, x + 70, y + 155);
});
```

#### Clipping (Recorte)

```javascript
const ctx = canvas.getContext('2d');

// clip() transforma o path atual em regiao de recorte
// Tudo desenhado depois so aparece dentro da regiao

ctx.save();

// Define uma regiao circular de recorte
ctx.beginPath();
ctx.arc(200, 200, 100, 0, Math.PI * 2);
ctx.clip();

// Apenas o que esta dentro do circulo sera visivel
ctx.fillStyle = '#3498db';
ctx.fillRect(0, 0, 400, 400);  // Retangulo, mas so circulo aparece

// Grade sobre o circulo
ctx.strokeStyle = '#fff';
ctx.lineWidth = 1;
for (let i = 0; i < 400; i += 20) {
  ctx.beginPath();
  ctx.moveTo(i, 0);
  ctx.lineTo(i, 400);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(0, i);
  ctx.lineTo(400, i);
  ctx.stroke();
}

ctx.restore(); // Restaura a regiao de clip original (sem recorte)

// Clip com formato de estrela
ctx.save();
ctx.beginPath();
for (let i = 0; i < 5; i++) {
  const anguloExt = (Math.PI * 2 / 5) * i - Math.PI / 2;
  const anguloInt = anguloExt + Math.PI / 5;
  if (i === 0) ctx.moveTo(450 + Math.cos(anguloExt) * 80, 200 + Math.sin(anguloExt) * 80);
  else ctx.lineTo(450 + Math.cos(anguloExt) * 80, 200 + Math.sin(anguloExt) * 80);
  ctx.lineTo(450 + Math.cos(anguloInt) * 35, 200 + Math.sin(anguloInt) * 35);
}
ctx.closePath();
ctx.clip();

// Gradiente visivel apenas na forma de estrela
const grad = ctx.createRadialGradient(450, 200, 0, 450, 200, 80);
grad.addColorStop(0, '#f1c40f');
grad.addColorStop(1, '#e67e22');
ctx.fillStyle = grad;
ctx.fillRect(350, 100, 200, 200);

ctx.restore();
```

### Imagens no Canvas

O metodo `drawImage()` e versatil e suporta tres assinaturas:

```javascript
const ctx = canvas.getContext('2d');
const img = new Image();
img.src = 'paisagem.jpg';
img.onload = () => {

  // 1. Desenha na posicao (dx, dy) — tamanho original
  ctx.drawImage(img, 10, 10);

  // 2. Desenha na posicao com tamanho especificado
  // drawImage(img, dx, dy, dLargura, dAltura)
  ctx.drawImage(img, 10, 10, 300, 200);

  // 3. Recorte da imagem + posicao + tamanho no canvas
  // drawImage(img, sx, sy, sLargura, sAltura, dx, dy, dLargura, dAltura)
  // s = source (recorte na imagem), d = destination (posicao no canvas)
  ctx.drawImage(img, 100, 50, 200, 150, 10, 10, 400, 300);
};
```

A fonte pode ser uma `<img>`, `<video>`, outro `<canvas>` ou `ImageBitmap`:

```javascript
// Capturar frame de video
const video = document.getElementById('meuVideo');
canvas.addEventListener('click', () => {
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
});

// Copiar de outro canvas
const outroCanvas = document.getElementById('canvas2');
ctx.drawImage(outroCanvas, 0, 0);
```

#### Exportando o Canvas

```javascript
// Como Data URL (base64)
const dataUrl = canvas.toDataURL('image/png');            // PNG (padrao)
const dataUrlJpeg = canvas.toDataURL('image/jpeg', 0.9);  // JPEG com qualidade 90%
const dataUrlWebp = canvas.toDataURL('image/webp', 0.8);  // WebP

// Inserir em uma tag <img>
const imgElement = document.createElement('img');
imgElement.src = dataUrl;
document.body.appendChild(imgElement);

// Como Blob (mais eficiente para upload/download)
canvas.toBlob((blob) => {
  // Download
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = 'captura.png';
  link.click();
  URL.revokeObjectURL(url);
}, 'image/png');

// Upload para servidor
canvas.toBlob(async (blob) => {
  const formData = new FormData();
  formData.append('imagem', blob, 'captura.png');
  await fetch('/api/upload', { method: 'POST', body: formData });
}, 'image/png');
```

> **Seguranca:** se o canvas contiver imagens de outros dominios sem CORS habilitado, `toDataURL()` e `toBlob()` lançam uma excecao de seguranca ("tainted canvas"). Sempre use `crossOrigin = 'anonymous'` em imagens de CDN/outros dominios.

### Manipulacao de Pixels

A Canvas API permite acesso direto aos pixels do bitmap, possibilitando manipulacao pixel a pixel para filtros e efeitos:

```javascript
const ctx = canvas.getContext('2d');

// Carrega uma imagem no canvas
const img = new Image();
img.crossOrigin = 'anonymous'; // Necessario para imagens de outros dominios
img.src = 'foto.jpg';
img.onload = () => {
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

  // getImageData(x, y, largura, altura) — retorna os pixels
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const data = imageData.data; // Uint8ClampedArray: [R, G, B, A, R, G, B, A, ...]

  // Cada pixel ocupa 4 posicoes no array: Red, Green, Blue, Alpha (0-255)
  // Para acessar o pixel na posicao (x, y):
  // const index = (y * largura + x) * 4;
  // data[index]     = Red
  // data[index + 1] = Green
  // data[index + 2] = Blue
  // data[index + 3] = Alpha

  // putImageData(imageData, x, y) — coloca os pixels de volta
  ctx.putImageData(imageData, 0, 0);
};
```

#### Filtros de Imagem

```javascript
// Filtro: escala de cinza
function filtroCinza(imageData) {
  const data = imageData.data;
  for (let i = 0; i < data.length; i += 4) {
    const media = (data[i] + data[i + 1] + data[i + 2]) / 3;
    data[i] = media;     // R
    data[i + 1] = media; // G
    data[i + 2] = media; // B
    // Alpha permanece inalterado
  }
  return imageData;
}

// Filtro: brilho (fator > 1 = mais claro, < 1 = mais escuro)
function filtroBrilho(imageData, fator) {
  const data = imageData.data;
  for (let i = 0; i < data.length; i += 4) {
    data[i] = Math.min(255, data[i] * fator);
    data[i + 1] = Math.min(255, data[i + 1] * fator);
    data[i + 2] = Math.min(255, data[i + 2] * fator);
  }
  return imageData;
}

// Filtro: inverter cores
function filtroInverter(imageData) {
  const data = imageData.data;
  for (let i = 0; i < data.length; i += 4) {
    data[i] = 255 - data[i];
    data[i + 1] = 255 - data[i + 1];
    data[i + 2] = 255 - data[i + 2];
  }
  return imageData;
}

// Filtro: sepia
function filtroSepia(imageData) {
  const data = imageData.data;
  for (let i = 0; i < data.length; i += 4) {
    const r = data[i], g = data[i + 1], b = data[i + 2];
    data[i] = Math.min(255, r * 0.393 + g * 0.769 + b * 0.189);
    data[i + 1] = Math.min(255, r * 0.349 + g * 0.686 + b * 0.168);
    data[i + 2] = Math.min(255, r * 0.272 + g * 0.534 + b * 0.131);
  }
  return imageData;
}

// Filtro: contraste
function filtroContraste(imageData, fator) {
  const data = imageData.data;
  const intercept = 128 * (1 - fator);
  for (let i = 0; i < data.length; i += 4) {
    data[i] = Math.min(255, Math.max(0, data[i] * fator + intercept));
    data[i + 1] = Math.min(255, Math.max(0, data[i + 1] * fator + intercept));
    data[i + 2] = Math.min(255, Math.max(0, data[i + 2] * fator + intercept));
  }
  return imageData;
}

// Aplicando filtro
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const resultado = filtroCinza(imageData);
ctx.putImageData(resultado, 0, 0);
```

> **Dica:** para filtros simples, o canvas tambem oferece a propriedade `ctx.filter` que aceita filtros CSS: `ctx.filter = 'blur(5px) grayscale(100%)'`. Mais performatico que manipulacao pixel a pixel, mas com menos controle.

### Animacoes com requestAnimationFrame

`requestAnimationFrame` e a forma recomendada de criar animacoes — o navegador sincroniza o callback com a taxa de atualizacao do monitor (normalmente 60fps):

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

// Padrao basico de loop de animacao
function animar() {
  ctx.clearRect(0, 0, canvas.width, canvas.height); // Limpa o frame anterior
  // ... desenhar frame atual ...
  requestAnimationFrame(animar); // Agenda o proximo frame
}
requestAnimationFrame(animar); // Inicia o loop
```

#### Delta Time para Animacao Independente de Frame Rate

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

let ultimoTempo = 0;

function animar(tempoAtual) {
  const deltaTime = (tempoAtual - ultimoTempo) / 1000; // Em segundos
  ultimoTempo = tempoAtual;

  // Evita delta time enorme na primeira execucao ou apos aba inativa
  const dt = Math.min(deltaTime, 0.1);

  atualizar(dt);
  desenhar();

  requestAnimationFrame(animar);
}

requestAnimationFrame(animar);
```

#### Exemplo: Bola Quicando

```javascript
const canvas = document.getElementById('meuCanvas');
canvas.width = 600;
canvas.height = 400;
const ctx = canvas.getContext('2d');

const bola = {
  x: 100,
  y: 100,
  raio: 20,
  vx: 200,   // pixels por segundo
  vy: 150,
  cor: '#e74c3c',
  gravidade: 400, // pixels/s^2
  amortecimento: 0.8,
};

let ultimoTempo = 0;

function atualizar(dt) {
  // Aplicar gravidade
  bola.vy += bola.gravidade * dt;

  // Atualizar posicao
  bola.x += bola.vx * dt;
  bola.y += bola.vy * dt;

  // Colisao com paredes laterais
  if (bola.x - bola.raio < 0) {
    bola.x = bola.raio;
    bola.vx = Math.abs(bola.vx) * bola.amortecimento;
  } else if (bola.x + bola.raio > canvas.width) {
    bola.x = canvas.width - bola.raio;
    bola.vx = -Math.abs(bola.vx) * bola.amortecimento;
  }

  // Colisao com teto e chao
  if (bola.y - bola.raio < 0) {
    bola.y = bola.raio;
    bola.vy = Math.abs(bola.vy) * bola.amortecimento;
  } else if (bola.y + bola.raio > canvas.height) {
    bola.y = canvas.height - bola.raio;
    bola.vy = -Math.abs(bola.vy) * bola.amortecimento;
  }
}

function desenhar() {
  // Fundo com leve transparencia para efeito de "rastro"
  ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Sombra da bola
  ctx.beginPath();
  ctx.ellipse(bola.x, canvas.height - 5, bola.raio * 0.8, 5, 0, 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
  ctx.fill();

  // Bola com gradiente
  const grad = ctx.createRadialGradient(
    bola.x - bola.raio * 0.3, bola.y - bola.raio * 0.3, bola.raio * 0.1,
    bola.x, bola.y, bola.raio
  );
  grad.addColorStop(0, '#ff6b6b');
  grad.addColorStop(1, bola.cor);

  ctx.beginPath();
  ctx.arc(bola.x, bola.y, bola.raio, 0, Math.PI * 2);
  ctx.fillStyle = grad;
  ctx.fill();
}

function loop(tempoAtual) {
  const deltaTime = (tempoAtual - ultimoTempo) / 1000;
  ultimoTempo = tempoAtual;
  const dt = Math.min(deltaTime, 0.1);

  atualizar(dt);
  desenhar();
  requestAnimationFrame(loop);
}

requestAnimationFrame(loop);
```

### Interatividade

O elemento `<canvas>` recebe eventos de mouse e toque como qualquer outro elemento DOM. O desafio e converter as coordenadas do evento para coordenadas do canvas.

#### Coordenadas do Mouse no Canvas

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

function getMousePos(canvas, evento) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  return {
    x: (evento.clientX - rect.left) * scaleX,
    y: (evento.clientY - rect.top) * scaleY,
  };
}

canvas.addEventListener('mousemove', (e) => {
  const pos = getMousePos(canvas, e);
  console.log(`Mouse em: (${pos.x.toFixed(0)}, ${pos.y.toFixed(0)})`);
});
```

#### Hit Detection (Deteccao de Clique)

```javascript
// 1. Bounding Box — simples e rapido
const objetos = [
  { x: 50, y: 50, largura: 100, altura: 80, cor: '#3498db', nome: 'Azul' },
  { x: 200, y: 100, largura: 120, altura: 60, cor: '#e74c3c', nome: 'Vermelho' },
  { x: 100, y: 200, largura: 80, altura: 100, cor: '#2ecc71', nome: 'Verde' },
];

function pontoEmRetangulo(px, py, obj) {
  return px >= obj.x && px <= obj.x + obj.largura &&
         py >= obj.y && py <= obj.y + obj.altura;
}

canvas.addEventListener('click', (e) => {
  const pos = getMousePos(canvas, e);
  for (const obj of objetos) {
    if (pontoEmRetangulo(pos.x, pos.y, obj)) {
      console.log(`Clicou em: ${obj.nome}`);
      break;
    }
  }
});

// 2. Deteccao circular
function pontoEmCirculo(px, py, cx, cy, raio) {
  const dx = px - cx;
  const dy = py - cy;
  return dx * dx + dy * dy <= raio * raio;
}

// 3. isPointInPath — o canvas verifica se o ponto esta dentro do ultimo path
canvas.addEventListener('click', (e) => {
  const pos = getMousePos(canvas, e);

  ctx.beginPath();
  ctx.arc(300, 200, 50, 0, Math.PI * 2);
  if (ctx.isPointInPath(pos.x, pos.y)) {
    console.log('Clicou dentro do circulo!');
  }
});
```

#### Drag and Drop no Canvas

```javascript
const canvas = document.getElementById('meuCanvas');
canvas.width = 600;
canvas.height = 400;
const ctx = canvas.getContext('2d');

const caixas = [
  { x: 50, y: 50, w: 80, h: 80, cor: '#3498db' },
  { x: 200, y: 100, w: 100, h: 60, cor: '#e74c3c' },
  { x: 100, y: 250, w: 70, h: 90, cor: '#2ecc71' },
];

let arrastando = null;
let offsetX = 0, offsetY = 0;

function desenharCaixas() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Fundo
  ctx.fillStyle = '#ecf0f1';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  for (const caixa of caixas) {
    ctx.fillStyle = caixa.cor;
    ctx.fillRect(caixa.x, caixa.y, caixa.w, caixa.h);
    ctx.strokeStyle = '#333';
    ctx.lineWidth = 2;
    ctx.strokeRect(caixa.x, caixa.y, caixa.w, caixa.h);
  }
}

function getCaixaEm(px, py) {
  // Itera de tras para frente (ultimo desenhado = mais acima visualmente)
  for (let i = caixas.length - 1; i >= 0; i--) {
    const c = caixas[i];
    if (px >= c.x && px <= c.x + c.w && py >= c.y && py <= c.y + c.h) {
      return c;
    }
  }
  return null;
}

canvas.addEventListener('mousedown', (e) => {
  const pos = getMousePos(canvas, e);
  arrastando = getCaixaEm(pos.x, pos.y);
  if (arrastando) {
    offsetX = pos.x - arrastando.x;
    offsetY = pos.y - arrastando.y;
    canvas.style.cursor = 'grabbing';
  }
});

canvas.addEventListener('mousemove', (e) => {
  const pos = getMousePos(canvas, e);

  if (arrastando) {
    arrastando.x = pos.x - offsetX;
    arrastando.y = pos.y - offsetY;
    desenharCaixas();
  } else {
    // Cursor de mao quando sobre uma caixa
    canvas.style.cursor = getCaixaEm(pos.x, pos.y) ? 'grab' : 'default';
  }
});

canvas.addEventListener('mouseup', () => {
  arrastando = null;
  canvas.style.cursor = 'default';
});

canvas.addEventListener('mouseleave', () => {
  arrastando = null;
  canvas.style.cursor = 'default';
});

desenharCaixas();
```

#### Exemplo: Desenho Interativo com o Mouse

```javascript
const canvas = document.getElementById('meuCanvas');
canvas.width = 600;
canvas.height = 400;
const ctx = canvas.getContext('2d');

let desenhando = false;
let ultimoX = 0, ultimoY = 0;

ctx.strokeStyle = '#2c3e50';
ctx.lineWidth = 3;
ctx.lineCap = 'round';
ctx.lineJoin = 'round';

canvas.addEventListener('mousedown', (e) => {
  desenhando = true;
  const pos = getMousePos(canvas, e);
  ultimoX = pos.x;
  ultimoY = pos.y;
});

canvas.addEventListener('mousemove', (e) => {
  if (!desenhando) return;
  const pos = getMousePos(canvas, e);

  ctx.beginPath();
  ctx.moveTo(ultimoX, ultimoY);
  ctx.lineTo(pos.x, pos.y);
  ctx.stroke();

  ultimoX = pos.x;
  ultimoY = pos.y;
});

canvas.addEventListener('mouseup', () => { desenhando = false; });
canvas.addEventListener('mouseleave', () => { desenhando = false; });
```

### Eventos de Toque (Touch) no Canvas

Para suporte mobile, e necessario tratar eventos touch alem de mouse:

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

function getTouchPos(canvas, touch) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  return {
    x: (touch.clientX - rect.left) * scaleX,
    y: (touch.clientY - rect.top) * scaleY,
  };
}

let desenhando = false;
let ultimoX, ultimoY;

// Funcao unificada para iniciar traço
function iniciarTraco(x, y) {
  desenhando = true;
  ultimoX = x;
  ultimoY = y;
}

// Funcao unificada para continuar traço
function continuarTraco(x, y) {
  if (!desenhando) return;
  ctx.beginPath();
  ctx.moveTo(ultimoX, ultimoY);
  ctx.lineTo(x, y);
  ctx.strokeStyle = '#2c3e50';
  ctx.lineWidth = 3;
  ctx.lineCap = 'round';
  ctx.stroke();
  ultimoX = x;
  ultimoY = y;
}

function pararTraco() {
  desenhando = false;
}

// Eventos de mouse
canvas.addEventListener('mousedown', (e) => {
  const pos = getMousePos(canvas, e);
  iniciarTraco(pos.x, pos.y);
});
canvas.addEventListener('mousemove', (e) => {
  const pos = getMousePos(canvas, e);
  continuarTraco(pos.x, pos.y);
});
canvas.addEventListener('mouseup', pararTraco);
canvas.addEventListener('mouseleave', pararTraco);

// Eventos de toque
canvas.addEventListener('touchstart', (e) => {
  e.preventDefault(); // Previne scroll ao tocar no canvas
  const pos = getTouchPos(canvas, e.touches[0]);
  iniciarTraco(pos.x, pos.y);
}, { passive: false });

canvas.addEventListener('touchmove', (e) => {
  e.preventDefault();
  const pos = getTouchPos(canvas, e.touches[0]);
  continuarTraco(pos.x, pos.y);
}, { passive: false });

canvas.addEventListener('touchend', pararTraco);
canvas.addEventListener('touchcancel', pararTraco);
```

> **Dica:** use `{ passive: false }` no `touchstart` e `touchmove` apenas quando precisar de `e.preventDefault()`. Caso contrario, use `{ passive: true }` para melhor performance de scroll.

### Dicas de Performance do Canvas

O Canvas 2D pode renderizar dezenas de milhares de elementos se otimizado corretamente:

```javascript
// 1. Agrupe operacoes de path — um unico fill/stroke para multiplas formas
// RUIM: um path por forma
for (const p of particulas) {
  ctx.beginPath();
  ctx.arc(p.x, p.y, 3, 0, Math.PI * 2);
  ctx.fill(); // Chamada de fill para cada particula!
}

// BOM: um unico path para todas as formas de mesma cor
ctx.beginPath();
for (const p of particulas) {
  ctx.moveTo(p.x + 3, p.y);
  ctx.arc(p.x, p.y, 3, 0, Math.PI * 2);
}
ctx.fill(); // Uma unica chamada de fill!

// 2. Cache de formas complexas em canvas offscreen
const cacheSprite = document.createElement('canvas');
cacheSprite.width = 64;
cacheSprite.height = 64;
const cacheCtx = cacheSprite.getContext('2d');
// Desenha a forma complexa uma vez
cacheCtx.beginPath();
cacheCtx.arc(32, 32, 30, 0, Math.PI * 2);
cacheCtx.fillStyle = '#3498db';
cacheCtx.fill();
cacheCtx.strokeStyle = '#2980b9';
cacheCtx.lineWidth = 3;
cacheCtx.stroke();

// Na animacao, apenas "cola" o cache (muito mais rapido)
for (const p of particulas) {
  ctx.drawImage(cacheSprite, p.x - 32, p.y - 32);
}

// 3. Limpe apenas a area necessaria (nao o canvas inteiro)
// Em vez de ctx.clearRect(0, 0, canvas.width, canvas.height):
for (const p of particulas) {
  ctx.clearRect(p.x - 5, p.y - 5, 10, 10); // Limpa so onde o objeto estava
}

// 4. Use canvas em camadas (layer separation)
// Canvas de fundo (raramente muda)
const canvasFundo = document.getElementById('fundo');
// Canvas de objetos (atualiza a cada frame)
const canvasObjetos = document.getElementById('objetos');
// Canvas de UI (atualiza quando interacao muda)
const canvasUI = document.getElementById('ui');

// 5. Evite propriedades que causam recalculo frequente
// Evite mudar ctx.font repetidamente dentro do loop
// Evite shadowBlur em animacoes (muito caro)
// Evite globalCompositeOperation com modos complexos em cada frame

// 6. Use numeros inteiros para coordenadas quando possivel
// Coordenadas fracionarias forcam anti-aliasing sub-pixel
ctx.fillRect(Math.round(x), Math.round(y), w, h);

// 7. Desative suavizacao de imagem quando nao necessaria (pixel art)
ctx.imageSmoothingEnabled = false;
ctx.drawImage(spritePixelArt, 0, 0, 64, 64);
```

> **Medicao de performance:** use `performance.now()` para medir tempo por frame e `requestAnimationFrame` para monitorar FPS:

```javascript
let frames = 0;
let fpsAnterior = performance.now();

function monitorarFPS() {
  frames++;
  const agora = performance.now();
  if (agora - fpsAnterior >= 1000) {
    console.log(`FPS: ${frames}`);
    frames = 0;
    fpsAnterior = agora;
  }
}
// Chamar monitorarFPS() dentro do loop de animacao
```

### Canvas Responsivo e HiDPI

#### Lidando com devicePixelRatio

Em telas de alta resolucao (Retina, HiDPI), o canvas pode ficar borrado se nao for tratado corretamente. O `devicePixelRatio` indica quantos pixels fisicos correspondem a um pixel CSS:

```javascript
function criarCanvasHiDPI(canvas, largura, altura) {
  const dpr = window.devicePixelRatio || 1;

  // Define o tamanho CSS (visual)
  canvas.style.width = largura + 'px';
  canvas.style.height = altura + 'px';

  // Define o tamanho real do bitmap (maior para telas HiDPI)
  canvas.width = largura * dpr;
  canvas.height = altura * dpr;

  // Escala o contexto para compensar
  const ctx = canvas.getContext('2d');
  ctx.scale(dpr, dpr);

  return ctx;
}

// Uso
const canvas = document.getElementById('meuCanvas');
const ctx = criarCanvasHiDPI(canvas, 600, 400);

// Agora desenhe normalmente usando coordenadas CSS
ctx.fillStyle = '#3498db';
ctx.fillRect(10, 10, 200, 100); // Crispy em qualquer tela!
```

#### Canvas Responsivo (Redimensionavel)

```javascript
const canvas = document.getElementById('meuCanvas');
const ctx = canvas.getContext('2d');

function redimensionar() {
  const dpr = window.devicePixelRatio || 1;
  const container = canvas.parentElement;
  const largura = container.clientWidth;
  const altura = container.clientHeight;

  canvas.style.width = largura + 'px';
  canvas.style.height = altura + 'px';
  canvas.width = largura * dpr;
  canvas.height = altura * dpr;
  ctx.scale(dpr, dpr);

  // Re-desenhar conteudo apos redimensionar
  desenhar(largura, altura);
}

function desenhar(largura, altura) {
  ctx.clearRect(0, 0, largura, altura);
  // ... desenhar usando largura e altura como referencia ...
  ctx.fillStyle = '#3498db';
  ctx.fillRect(largura * 0.1, altura * 0.1, largura * 0.8, altura * 0.8);
}

// Ouvir evento de redimensionamento com debounce
let timeoutResize;
window.addEventListener('resize', () => {
  clearTimeout(timeoutResize);
  timeoutResize = setTimeout(redimensionar, 100);
});

// Redimensionar na inicializacao
redimensionar();
```

```html
<style>
  .canvas-container {
    width: 100%;
    height: 80vh;
  }
  .canvas-container canvas {
    display: block;
  }
</style>
<div class="canvas-container">
  <canvas id="meuCanvas"></canvas>
</div>
```

### OffscreenCanvas e Web Workers

`OffscreenCanvas` permite renderizar graficos em uma Web Worker, liberando a thread principal para interacao do usuario:

```javascript
// main.js — Thread principal
const canvas = document.getElementById('meuCanvas');

// Transfere o controle do canvas para um OffscreenCanvas
const offscreen = canvas.transferControlToOffscreen();

const worker = new Worker('canvas-worker.js');

// Envia o OffscreenCanvas para o worker (transferido, nao copiado)
worker.postMessage({ canvas: offscreen }, [offscreen]);

// Comunicacao com o worker
worker.postMessage({ tipo: 'mudarCor', cor: '#e74c3c' });
```

```javascript
// canvas-worker.js — Web Worker
let canvas;
let ctx;

self.onmessage = function(e) {
  if (e.data.canvas) {
    canvas = e.data.canvas;
    ctx = canvas.getContext('2d');
    iniciarRenderizacao();
    return;
  }

  if (e.data.tipo === 'mudarCor') {
    corAtual = e.data.cor;
  }
};

let corAtual = '#3498db';
const particulas = [];

// Cria 10.000 particulas
for (let i = 0; i < 10000; i++) {
  particulas.push({
    x: Math.random() * 600,
    y: Math.random() * 400,
    vx: (Math.random() - 0.5) * 2,
    vy: (Math.random() - 0.5) * 2,
    raio: Math.random() * 3 + 1,
  });
}

function iniciarRenderizacao() {
  function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = corAtual;
    for (const p of particulas) {
      p.x += p.vx;
      p.y += p.vy;
      if (p.x < 0 || p.x > canvas.width) p.vx *= -1;
      if (p.y < 0 || p.y > canvas.height) p.vy *= -1;

      ctx.beginPath();
      ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
      ctx.fill();
    }

    requestAnimationFrame(loop);
  }
  loop();
}
```

> **Quando usar OffscreenCanvas:** renderizacao pesada com milhares de elementos, processamento de imagem (filtros complexos), jogos com logica pesada de atualizacao. Para canvas simples, a thread principal e suficiente.

> **Suporte:** OffscreenCanvas e suportado em Chrome, Edge, Firefox e Safari 16.4+.

### Exemplo Pratico: Editor de Desenho Simples

Exemplo completo que combina multiplos conceitos — desenho com mouse, seletor de cor, tamanho do pincel e botao de limpar:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Editor de Desenho</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: Arial, sans-serif; background: #f5f5f5; padding: 20px; }
    .toolbar {
      display: flex; gap: 15px; align-items: center;
      padding: 10px 20px; background: #fff; border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 10px;
      max-width: 802px; margin-left: auto; margin-right: auto;
    }
    .toolbar label { font-size: 14px; color: #555; }
    .toolbar input[type="color"] { width: 40px; height: 30px; border: none; cursor: pointer; }
    .toolbar input[type="range"] { width: 120px; }
    .toolbar button {
      padding: 6px 16px; border: none; border-radius: 4px;
      background: #e74c3c; color: #fff; cursor: pointer; font-size: 14px;
    }
    .toolbar button:hover { background: #c0392b; }
    canvas {
      display: block; margin: 0 auto; background: #fff;
      border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      cursor: crosshair;
    }
  </style>
</head>
<body>
  <div class="toolbar">
    <label>Cor: <input type="color" id="corPincel" value="#2c3e50"></label>
    <label>Tamanho: <input type="range" id="tamanhoPincel" min="1" max="50" value="5">
      <span id="tamanhoLabel">5</span>px
    </label>
    <button id="btnLimpar">Limpar</button>
    <button id="btnSalvar">Salvar PNG</button>
  </div>
  <canvas id="editor" width="800" height="500"></canvas>
  <script>
    const canvas = document.getElementById('editor');
    const ctx = canvas.getContext('2d');
    const corInput = document.getElementById('corPincel');
    const tamanhoInput = document.getElementById('tamanhoPincel');
    const tamanhoLabel = document.getElementById('tamanhoLabel');

    let desenhando = false;
    let ultimoX, ultimoY;

    // HiDPI
    const dpr = window.devicePixelRatio || 1;
    const w = canvas.width, h = canvas.height;
    canvas.width = w * dpr;
    canvas.height = h * dpr;
    canvas.style.width = w + 'px';
    canvas.style.height = h + 'px';
    ctx.scale(dpr, dpr);

    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';

    function getPos(e) {
      const rect = canvas.getBoundingClientRect();
      return {
        x: (e.clientX - rect.left) * (w / rect.width),
        y: (e.clientY - rect.top) * (h / rect.height),
      };
    }

    canvas.addEventListener('mousedown', (e) => {
      desenhando = true;
      const pos = getPos(e);
      ultimoX = pos.x;
      ultimoY = pos.y;
    });

    canvas.addEventListener('mousemove', (e) => {
      if (!desenhando) return;
      const pos = getPos(e);
      ctx.strokeStyle = corInput.value;
      ctx.lineWidth = parseInt(tamanhoInput.value);
      ctx.beginPath();
      ctx.moveTo(ultimoX, ultimoY);
      ctx.lineTo(pos.x, pos.y);
      ctx.stroke();
      ultimoX = pos.x;
      ultimoY = pos.y;
    });

    canvas.addEventListener('mouseup', () => { desenhando = false; });
    canvas.addEventListener('mouseleave', () => { desenhando = false; });

    tamanhoInput.addEventListener('input', () => {
      tamanhoLabel.textContent = tamanhoInput.value;
    });

    document.getElementById('btnLimpar').addEventListener('click', () => {
      ctx.clearRect(0, 0, w, h);
    });

    document.getElementById('btnSalvar').addEventListener('click', () => {
      const link = document.createElement('a');
      link.download = 'desenho.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    });

    // Suporte a toque (mobile)
    canvas.addEventListener('touchstart', (e) => {
      e.preventDefault();
      const touch = e.touches[0];
      const mouseEvent = new MouseEvent('mousedown', { clientX: touch.clientX, clientY: touch.clientY });
      canvas.dispatchEvent(mouseEvent);
    });
    canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      const touch = e.touches[0];
      const mouseEvent = new MouseEvent('mousemove', { clientX: touch.clientX, clientY: touch.clientY });
      canvas.dispatchEvent(mouseEvent);
    });
    canvas.addEventListener('touchend', () => {
      canvas.dispatchEvent(new MouseEvent('mouseup'));
    });
  </script>
</body>
</html>
```

> Este editor pode ser estendido com: ferramenta de formas (retangulo, circulo), borracha, desfazer/refazer (armazenar `ImageData` a cada acao), camadas (multiplos canvas empilhados).

---

## 3. SVG (Scalable Vector Graphics)

SVG e um formato de graficos vetoriais baseado em XML, integrado nativamente ao DOM do navegador. Diferente do Canvas, cada elemento SVG e um no DOM que pode ser estilizado com CSS, manipulado com JavaScript e inspecionado por ferramentas de acessibilidade.

> MDN: [SVG](https://developer.mozilla.org/pt-BR/docs/Web/SVG) . [Tutorial SVG](https://developer.mozilla.org/pt-BR/docs/Web/SVG/Tutorial)

### Fundamentos

#### O Elemento `<svg>`

```html
<!-- SVG inline no HTML -->
<svg width="400" height="300" xmlns="http://www.w3.org/2000/svg">
  <!-- Elementos graficos aqui -->
  <rect x="10" y="10" width="100" height="80" fill="#3498db" />
</svg>
```

- `xmlns="http://www.w3.org/2000/svg"` — namespace obrigatorio para SVG standalone; em HTML5 inline pode ser omitido
- `width` e `height` — dimensoes do viewport (area visivel)

#### viewBox e Sistema de Coordenadas

O atributo `viewBox` define o **sistema de coordenadas interno** do SVG, independente do tamanho de exibicao:

```html
<!-- viewBox="minX minY largura altura" -->
<svg width="400" height="300" viewBox="0 0 100 100">
  <!-- Sistema interno: 100x100 unidades, exibido em 400x300px -->
  <!-- Um retangulo de 50x50 ocupa metade da largura do SVG -->
  <rect x="0" y="0" width="50" height="50" fill="#3498db" />
</svg>
```

> **Dica:** use `viewBox` sem `width`/`height` para SVGs que se adaptam ao container:

```html
<!-- SVG responsivo — preenche o container mantendo proporcao -->
<svg viewBox="0 0 100 100" style="width: 100%; height: auto;">
  <circle cx="50" cy="50" r="40" fill="#e74c3c" />
</svg>
```

#### preserveAspectRatio

Controla como o conteudo do `viewBox` se encaixa no viewport quando as proporcoes sao diferentes:

```html
<!-- Sintaxe: preserveAspectRatio="<alinhamento> <estrategia>" -->
<!-- Alinhamentos: xMinYMin, xMidYMin, xMaxYMin, xMinYMid, xMidYMid, etc. -->
<!-- Estrategia: meet (cabe todo, pode ter espaco) ou slice (preenche, pode cortar) -->

<!-- Centralizado, cabe inteiro (padrao) -->
<svg viewBox="0 0 100 50" preserveAspectRatio="xMidYMid meet" width="400" height="300">
  <rect width="100" height="50" fill="#3498db" />
</svg>

<!-- Estica sem manter proporcao (como background-size: 100% 100%) -->
<svg viewBox="0 0 100 50" preserveAspectRatio="none" width="400" height="300">
  <rect width="100" height="50" fill="#e74c3c" />
</svg>

<!-- Preenche cortando (como background-size: cover) -->
<svg viewBox="0 0 100 50" preserveAspectRatio="xMidYMid slice" width="400" height="300">
  <rect width="100" height="50" fill="#2ecc71" />
</svg>
```

### Formas Basicas

SVG oferece seis formas primitivas:

#### Retangulo (`<rect>`)

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Retangulo simples -->
  <rect x="10" y="10" width="120" height="80" fill="#3498db" />

  <!-- Retangulo com contorno -->
  <rect x="150" y="10" width="120" height="80"
        fill="#2ecc71" stroke="#27ae60" stroke-width="3" />

  <!-- Retangulo com cantos arredondados -->
  <rect x="290" y="10" width="100" height="80"
        rx="15" ry="15"
        fill="#e74c3c" />

  <!-- Retangulo so com contorno (sem preenchimento) -->
  <rect x="10" y="110" width="120" height="80"
        fill="none" stroke="#9b59b6" stroke-width="2" stroke-dasharray="8 4" />
</svg>
```

#### Circulo (`<circle>`)

```html
<svg viewBox="0 0 300 150" xmlns="http://www.w3.org/2000/svg">
  <!-- cx, cy = centro; r = raio -->
  <circle cx="75" cy="75" r="60" fill="#3498db" />

  <circle cx="200" cy="75" r="50"
          fill="rgba(231, 76, 60, 0.7)"
          stroke="#c0392b" stroke-width="3" />
</svg>
```

#### Elipse (`<ellipse>`)

```html
<svg viewBox="0 0 300 150" xmlns="http://www.w3.org/2000/svg">
  <!-- cx, cy = centro; rx = raio horizontal; ry = raio vertical -->
  <ellipse cx="150" cy="75" rx="120" ry="60"
           fill="#9b59b6" opacity="0.8" />
</svg>
```

#### Linha (`<line>`)

```html
<svg viewBox="0 0 300 150" xmlns="http://www.w3.org/2000/svg">
  <!-- x1, y1 = inicio; x2, y2 = fim -->
  <line x1="10" y1="10" x2="290" y2="140"
        stroke="#2c3e50" stroke-width="3" />

  <!-- Linha tracejada -->
  <line x1="10" y1="75" x2="290" y2="75"
        stroke="#e74c3c" stroke-width="2"
        stroke-dasharray="10 5" />
</svg>
```

#### Polilinha (`<polyline>`)

```html
<svg viewBox="0 0 300 150" xmlns="http://www.w3.org/2000/svg">
  <!-- Sequencia de pontos conectados (nao fecha automaticamente) -->
  <polyline points="10,140 50,30 100,120 150,20 200,100 250,10 290,80"
            fill="none" stroke="#3498db" stroke-width="2" />

  <!-- Com preenchimento (fecha visualmente) -->
  <polyline points="10,140 50,80 100,130 150,60 200,110 250,50 290,90 290,140"
            fill="rgba(46, 204, 113, 0.3)" stroke="#2ecc71" stroke-width="2" />
</svg>
```

#### Poligono (`<polygon>`)

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Como polyline, mas fecha automaticamente -->

  <!-- Triangulo -->
  <polygon points="60,10 10,110 110,110"
           fill="#e74c3c" stroke="#c0392b" stroke-width="2" />

  <!-- Pentagono -->
  <polygon points="200,20 240,70 225,120 175,120 160,70"
           fill="#3498db" stroke="#2980b9" stroke-width="2" />

  <!-- Estrela -->
  <polygon points="350,25 362,65 405,65 370,90 382,130 350,105 318,130 330,90 295,65 338,65"
           fill="#f1c40f" stroke="#f39c12" stroke-width="2" />
</svg>
```

### Paths

O elemento `<path>` e o mais poderoso do SVG — pode desenhar qualquer forma. O atributo `d` contem uma sequencia de comandos:

| Comando | Significado | Parametros |
|---|---|---|
| `M` / `m` | Move to (mover sem desenhar) | `x y` |
| `L` / `l` | Line to (linha reta) | `x y` |
| `H` / `h` | Horizontal line to | `x` |
| `V` / `v` | Vertical line to | `y` |
| `C` / `c` | Curva Bezier cubica | `x1 y1 x2 y2 x y` |
| `S` / `s` | Curva cubica suave (espelhada) | `x2 y2 x y` |
| `Q` / `q` | Curva Bezier quadratica | `x1 y1 x y` |
| `T` / `t` | Curva quadratica suave | `x y` |
| `A` / `a` | Arco eliptico | `rx ry rotacao large-arc sweep x y` |
| `Z` / `z` | Fechar path (volta ao M) | — |

> Maiuscula = coordenadas **absolutas**; minuscula = coordenadas **relativas** (ao ponto atual).

#### Exemplos de Path

```html
<svg viewBox="0 0 500 400" xmlns="http://www.w3.org/2000/svg">

  <!-- Triangulo com path -->
  <path d="M 50 10 L 10 90 L 90 90 Z"
        fill="#e74c3c" />

  <!-- Quadrado com linhas horizontais e verticais -->
  <path d="M 120 10 H 200 V 90 H 120 Z"
        fill="#3498db" />

  <!-- Curva Bezier quadratica -->
  <path d="M 10 200 Q 100 120 200 200"
        fill="none" stroke="#9b59b6" stroke-width="3" />

  <!-- Curva Bezier cubica -->
  <path d="M 220 200 C 250 120 350 280 400 200"
        fill="none" stroke="#e67e22" stroke-width="3" />

  <!-- Arco -->
  <path d="M 50 300 A 50 50 0 1 1 150 300"
        fill="none" stroke="#2ecc71" stroke-width="3" />

</svg>
```

#### Desenhando um Coracao com Path

```html
<svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <path d="M 100 180
           C 40 120, 0 80, 0 50
           A 50 50 0 0 1 100 30
           A 50 50 0 0 1 200 50
           C 200 80, 160 120, 100 180 Z"
        fill="#e74c3c" />
</svg>
```

#### Desenhando uma Estrela com Path

```html
<svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Estrela de 5 pontas calculada com coordenadas -->
  <path d="M 100 10
           L 120 75 L 190 75 L 135 115
           L 155 180 L 100 140 L 45 180
           L 65 115 L 10 75 L 80 75 Z"
        fill="#f1c40f" stroke="#f39c12" stroke-width="2" />
</svg>
```

#### Gerando Paths com JavaScript

```javascript
// Gerar path de poligono regular
function gerarPoligono(cx, cy, raio, lados) {
  let d = '';
  for (let i = 0; i < lados; i++) {
    const angulo = (Math.PI * 2 / lados) * i - Math.PI / 2;
    const x = cx + Math.cos(angulo) * raio;
    const y = cy + Math.sin(angulo) * raio;
    d += (i === 0 ? 'M' : 'L') + ` ${x.toFixed(2)} ${y.toFixed(2)} `;
  }
  return d + 'Z';
}

// Gerar path de estrela
function gerarEstrela(cx, cy, pontas, raioExt, raioInt) {
  let d = '';
  for (let i = 0; i < pontas * 2; i++) {
    const raio = i % 2 === 0 ? raioExt : raioInt;
    const angulo = (Math.PI / pontas) * i - Math.PI / 2;
    const x = cx + Math.cos(angulo) * raio;
    const y = cy + Math.sin(angulo) * raio;
    d += (i === 0 ? 'M' : 'L') + ` ${x.toFixed(2)} ${y.toFixed(2)} `;
  }
  return d + 'Z';
}

// Aplicar ao SVG
const path = document.querySelector('#meuPath');
path.setAttribute('d', gerarEstrela(100, 100, 5, 80, 35));
```

### Texto SVG

```html
<svg viewBox="0 0 500 300" xmlns="http://www.w3.org/2000/svg">

  <!-- Texto simples -->
  <text x="10" y="40" font-size="24" fill="#2c3e50">
    Texto SVG Simples
  </text>

  <!-- Texto com estilos -->
  <text x="10" y="80" font-size="20" font-family="Georgia"
        font-weight="bold" font-style="italic" fill="#e74c3c">
    Texto Estilizado
  </text>

  <!-- tspan — partes do texto com estilos diferentes -->
  <text x="10" y="130" font-size="18" fill="#333">
    Texto com
    <tspan fill="#3498db" font-weight="bold">partes</tspan>
    <tspan fill="#e74c3c" font-size="24">diferentes</tspan>
  </text>

  <!-- Texto em multiplas linhas com tspan -->
  <text x="10" y="170" font-size="16" fill="#2c3e50">
    <tspan x="10" dy="0">Primeira linha do texto</tspan>
    <tspan x="10" dy="24">Segunda linha do texto</tspan>
    <tspan x="10" dy="24">Terceira linha do texto</tspan>
  </text>

  <!-- dx e dy — deslocamento relativo -->
  <text x="10" y="270" font-size="20" fill="#9b59b6">
    <tspan>N</tspan>
    <tspan dy="-5">o</tspan>
    <tspan dy="5">n</tspan>
    <tspan dy="-8">d</tspan>
    <tspan dy="8">a</tspan>
  </text>

</svg>
```

#### Texto ao Longo de um Path (`<textPath>`)

```html
<svg viewBox="0 0 500 200" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <path id="curva" d="M 20 150 Q 250 20 480 150" fill="none" />
  </defs>

  <!-- Texto segue o caminho curvo -->
  <text font-size="18" fill="#2c3e50">
    <textPath href="#curva" startOffset="50%" text-anchor="middle">
      Texto seguindo uma curva Bezier no SVG
    </textPath>
  </text>

  <!-- Mostrar o caminho para referencia -->
  <use href="#curva" stroke="#ccc" stroke-width="1" stroke-dasharray="4 2" />
</svg>
```

### Gradientes e Patterns

#### Gradiente Linear

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Gradiente horizontal (padrao: x1=0 y1=0 x2=1 y2=0) -->
    <linearGradient id="gradH">
      <stop offset="0%" stop-color="#3498db" />
      <stop offset="50%" stop-color="#9b59b6" />
      <stop offset="100%" stop-color="#e74c3c" />
    </linearGradient>

    <!-- Gradiente vertical -->
    <linearGradient id="gradV" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="#f1c40f" />
      <stop offset="100%" stop-color="#e67e22" />
    </linearGradient>

    <!-- Gradiente diagonal -->
    <linearGradient id="gradD" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#2ecc71" />
      <stop offset="100%" stop-color="#3498db" />
    </linearGradient>
  </defs>

  <rect x="10" y="10" width="120" height="80" fill="url(#gradH)" rx="8" />
  <rect x="140" y="10" width="120" height="80" fill="url(#gradV)" rx="8" />
  <rect x="270" y="10" width="120" height="80" fill="url(#gradD)" rx="8" />
</svg>
```

#### Gradiente Radial

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Gradiente radial centrado -->
    <radialGradient id="gradR1">
      <stop offset="0%" stop-color="#f1c40f" />
      <stop offset="100%" stop-color="#e67e22" />
    </radialGradient>

    <!-- Gradiente radial deslocado (efeito esfera 3D) -->
    <radialGradient id="gradR2" cx="0.35" cy="0.35" r="0.6">
      <stop offset="0%" stop-color="#fff" />
      <stop offset="50%" stop-color="#3498db" />
      <stop offset="100%" stop-color="#1a5276" />
    </radialGradient>
  </defs>

  <circle cx="80" cy="100" r="70" fill="url(#gradR1)" />
  <circle cx="250" cy="100" r="70" fill="url(#gradR2)" />
</svg>
```

#### Patterns (Padroes)

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Padrao xadrez -->
    <pattern id="xadrez" x="0" y="0" width="20" height="20"
             patternUnits="userSpaceOnUse">
      <rect width="10" height="10" fill="#ecf0f1" />
      <rect x="10" y="10" width="10" height="10" fill="#ecf0f1" />
      <rect x="10" width="10" height="10" fill="#bdc3c7" />
      <rect y="10" width="10" height="10" fill="#bdc3c7" />
    </pattern>

    <!-- Padrao de linhas diagonais -->
    <pattern id="listras" width="10" height="10"
             patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
      <line x1="0" y1="0" x2="0" y2="10"
            stroke="#3498db" stroke-width="4" />
    </pattern>

    <!-- Padrao de pontos -->
    <pattern id="pontos" width="15" height="15"
             patternUnits="userSpaceOnUse">
      <circle cx="7.5" cy="7.5" r="3" fill="#e74c3c" />
    </pattern>
  </defs>

  <rect x="10" y="10" width="120" height="180" fill="url(#xadrez)" stroke="#999" />
  <rect x="140" y="10" width="120" height="180" fill="url(#listras)" stroke="#999" />
  <rect x="270" y="10" width="120" height="180" fill="url(#pontos)" stroke="#999" />
</svg>
```

### Transformacoes e Grupos

#### Grupos (`<g>`)

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <!-- <g> agrupa elementos — estilos e transformacoes se aplicam a todos -->
  <g fill="#3498db" stroke="#2980b9" stroke-width="2">
    <rect x="10" y="10" width="60" height="40" />
    <rect x="80" y="10" width="60" height="40" />
    <rect x="10" y="60" width="130" height="40" />
  </g>

  <!-- Grupos aninhados -->
  <g transform="translate(200, 10)" opacity="0.8">
    <g fill="#e74c3c">
      <circle cx="30" cy="30" r="25" />
      <circle cx="90" cy="30" r="25" />
    </g>
    <g fill="#2ecc71">
      <circle cx="60" cy="80" r="25" />
    </g>
  </g>
</svg>
```

#### Definicoes e Reutilizacao (`<defs>`, `<use>`, `<symbol>`)

```html
<svg viewBox="0 0 500 200" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Elementos dentro de <defs> nao sao renderizados diretamente -->

    <!-- Icone de seta reutilizavel -->
    <path id="seta" d="M 0 0 L 20 10 L 0 20 L 5 10 Z" fill="currentColor" />

    <!-- <symbol> e como <defs> + <g> com viewBox proprio -->
    <symbol id="iconeAlerta" viewBox="0 0 24 24">
      <path d="M 12 2 L 22 20 H 2 Z" fill="#f39c12" stroke="#e67e22" stroke-width="1" />
      <text x="12" y="17" text-anchor="middle" font-size="12" font-weight="bold"
            fill="#fff">!</text>
    </symbol>
  </defs>

  <!-- <use> referencia e "clona" um elemento definido -->
  <use href="#seta" x="10" y="50" fill="#3498db" />
  <use href="#seta" x="10" y="80" fill="#e74c3c" />
  <use href="#seta" x="10" y="110" fill="#2ecc71" />

  <!-- <use> com <symbol> — usa o viewBox do symbol -->
  <use href="#iconeAlerta" x="100" y="40" width="48" height="48" />
  <use href="#iconeAlerta" x="160" y="40" width="32" height="32" />
  <use href="#iconeAlerta" x="200" y="40" width="24" height="24" />
</svg>
```

#### Transformacoes

```html
<svg viewBox="0 0 500 300" xmlns="http://www.w3.org/2000/svg">
  <!-- translate(tx, ty) — deslocar -->
  <rect width="60" height="40" fill="#3498db" transform="translate(10, 10)" />

  <!-- rotate(angulo, cx, cy) — rotacionar ao redor de (cx, cy) -->
  <rect x="150" y="30" width="60" height="40" fill="#e74c3c"
        transform="rotate(30, 180, 50)" />

  <!-- scale(sx, sy) — escalar -->
  <rect x="10" y="10" width="30" height="20" fill="#2ecc71"
        transform="translate(300, 50) scale(2, 1.5)" />

  <!-- skewX(angulo) e skewY(angulo) — inclinacao -->
  <rect x="10" y="150" width="80" height="50" fill="#9b59b6"
        transform="skewX(20)" />

  <!-- Transformacoes combinadas (aplicadas da direita para esquerda) -->
  <rect width="60" height="40" fill="#e67e22"
        transform="translate(300, 200) rotate(45) scale(1.5)" />

  <!-- matrix(a, b, c, d, e, f) — transformacao completa via matriz -->
  <rect width="60" height="40" fill="#1abc9c"
        transform="matrix(1, 0.3, -0.3, 1, 200, 200)" />
</svg>
```

### Filtros SVG

Os filtros SVG aplicam efeitos visuais complexos como desfoque, sombras e manipulacao de cores:

```html
<svg viewBox="0 0 600 400" xmlns="http://www.w3.org/2000/svg">
  <defs>

    <!-- Desfoque Gaussiano -->
    <filter id="desfoque">
      <feGaussianBlur in="SourceGraphic" stdDeviation="3" />
    </filter>

    <!-- Sombra projetada -->
    <filter id="sombra" x="-20%" y="-20%" width="140%" height="140%">
      <feDropShadow dx="4" dy="4" stdDeviation="3" flood-color="rgba(0,0,0,0.4)" />
    </filter>

    <!-- Sombra projetada manual (mais controle) -->
    <filter id="sombraManual" x="-20%" y="-20%" width="140%" height="140%">
      <feOffset in="SourceAlpha" dx="4" dy="4" result="offsetAlpha" />
      <feGaussianBlur in="offsetAlpha" stdDeviation="3" result="blurAlpha" />
      <feMerge>
        <feMergeNode in="blurAlpha" />
        <feMergeNode in="SourceGraphic" />
      </feMerge>
    </filter>

    <!-- Manipulacao de cor — escala de cinza -->
    <filter id="cinza">
      <feColorMatrix type="saturate" values="0" />
    </filter>

    <!-- Manipulacao de cor — sepia -->
    <filter id="sepia">
      <feColorMatrix type="matrix"
        values="0.393 0.769 0.189 0 0
                0.349 0.686 0.168 0 0
                0.272 0.534 0.131 0 0
                0     0     0     1 0" />
    </filter>

    <!-- Efeito de brilho (glow) -->
    <filter id="brilho">
      <feGaussianBlur in="SourceGraphic" stdDeviation="4" result="blur" />
      <feMerge>
        <feMergeNode in="blur" />
        <feMergeNode in="SourceGraphic" />
      </feMerge>
    </filter>

    <!-- Efeito emboss -->
    <filter id="emboss">
      <feConvolveMatrix kernelMatrix="
        -2 -1 0
        -1  1 1
         0  1 2" />
    </filter>

  </defs>

  <!-- Aplicando filtros -->
  <circle cx="80" cy="80" r="50" fill="#3498db" filter="url(#desfoque)" />

  <rect x="160" y="30" width="100" height="80" fill="#e74c3c" rx="8"
        filter="url(#sombra)" />

  <rect x="300" y="30" width="100" height="80" fill="#2ecc71" rx="8"
        filter="url(#sombraManual)" />

  <circle cx="80" cy="220" r="50" fill="#e74c3c" filter="url(#cinza)" />
  <circle cx="200" cy="220" r="50" fill="#3498db" filter="url(#sepia)" />
  <circle cx="320" cy="220" r="50" fill="#f1c40f" filter="url(#brilho)" />

  <!-- Labels -->
  <text x="80" y="150" text-anchor="middle" font-size="11" fill="#666">Desfoque</text>
  <text x="210" y="150" text-anchor="middle" font-size="11" fill="#666">Sombra</text>
  <text x="350" y="150" text-anchor="middle" font-size="11" fill="#666">Sombra Manual</text>
  <text x="80" y="290" text-anchor="middle" font-size="11" fill="#666">Cinza</text>
  <text x="200" y="290" text-anchor="middle" font-size="11" fill="#666">Sepia</text>
  <text x="320" y="290" text-anchor="middle" font-size="11" fill="#666">Brilho</text>
</svg>
```

> **Desempenho:** filtros SVG podem ser pesados para renderizacao. Evite aplicar filtros complexos em elementos animados. Para sombras simples, prefira `filter: drop-shadow()` via CSS que e otimizado pelo navegador.

### Animacoes SVG

#### SMIL (Synchronized Multimedia Integration Language)

Animacoes declarativas diretamente no markup SVG:

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">

  <!-- <animate> — anima um atributo -->
  <circle cx="50" cy="100" r="30" fill="#3498db">
    <!-- Circulo move horizontalmente -->
    <animate attributeName="cx" from="50" to="350" dur="3s"
             repeatCount="indefinite" />
  </circle>

  <!-- Mudanca de cor -->
  <rect x="50" y="20" width="80" height="40" fill="#e74c3c">
    <animate attributeName="fill" values="#e74c3c;#3498db;#2ecc71;#e74c3c"
             dur="4s" repeatCount="indefinite" />
  </rect>

  <!-- <animateTransform> — anima transformacoes -->
  <rect x="-25" y="-25" width="50" height="50" fill="#f1c40f"
        transform="translate(300, 100)">
    <animateTransform attributeName="transform" type="rotate"
                      from="0 0 0" to="360 0 0" dur="3s"
                      repeatCount="indefinite" additive="sum" />
  </rect>

  <!-- <animateMotion> — move ao longo de um path -->
  <circle r="10" fill="#9b59b6">
    <animateMotion dur="5s" repeatCount="indefinite" rotate="auto"
                   path="M 20 100 C 100 20 200 180 350 100" />
  </circle>

</svg>
```

> **Nota:** SMIL foi "deprecado" pelo Chrome em 2015, mas a decisao foi revertida. SMIL continua suportado em todos os navegadores modernos e faz parte da especificacao SVG 2.

#### Animacoes CSS em SVG

```html
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <style>
    .pulsar {
      animation: pulso 1.5s ease-in-out infinite;
      transform-origin: center;
    }
    @keyframes pulso {
      0%, 100% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.3); opacity: 0.7; }
    }

    .girar {
      animation: rotacao 4s linear infinite;
      transform-origin: 300px 100px; /* centro de rotacao */
    }
    @keyframes rotacao {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }

    .loading-dash {
      stroke-dasharray: 150;
      stroke-dashoffset: 150;
      animation: tracar 2s ease-in-out infinite;
    }
    @keyframes tracar {
      0% { stroke-dashoffset: 150; }
      50% { stroke-dashoffset: 0; }
      100% { stroke-dashoffset: -150; }
    }
  </style>

  <!-- Circulo pulsando -->
  <circle cx="80" cy="100" r="30" fill="#e74c3c" class="pulsar" />

  <!-- Retangulo girando -->
  <rect x="270" y="70" width="60" height="60" fill="#3498db" class="girar" />

  <!-- Circulo de loading com traço animado -->
  <circle cx="180" cy="100" r="35" fill="none" stroke="#2ecc71"
          stroke-width="4" stroke-linecap="round" class="loading-dash" />
</svg>
```

#### Animacao SVG com JavaScript

```javascript
// Animacao simples com requestAnimationFrame
const circulo = document.querySelector('#meuCirculo');
let angulo = 0;

function animarCirculo() {
  angulo += 0.02;
  const x = 200 + Math.cos(angulo) * 100;
  const y = 150 + Math.sin(angulo) * 80;
  circulo.setAttribute('cx', x);
  circulo.setAttribute('cy', y);
  requestAnimationFrame(animarCirculo);
}
animarCirculo();
```

```javascript
// Usando a Web Animations API (nativa, sem biblioteca)
const elemento = document.querySelector('#meuRect');

elemento.animate([
  { transform: 'translateX(0) rotate(0deg)', fill: '#3498db' },
  { transform: 'translateX(200px) rotate(180deg)', fill: '#e74c3c' },
  { transform: 'translateX(0) rotate(360deg)', fill: '#3498db' },
], {
  duration: 3000,
  iterations: Infinity,
  easing: 'ease-in-out',
});
```

```javascript
// Exemplo com GSAP (biblioteca popular de animacao)
// npm install gsap

// Animacao basica
gsap.to('#meuCirculo', {
  attr: { cx: 350, cy: 50 },
  fill: '#e74c3c',
  duration: 2,
  ease: 'power2.inOut',
  yoyo: true,
  repeat: -1,
});

// Timeline — sequencia de animacoes
const tl = gsap.timeline({ repeat: -1, repeatDelay: 0.5 });
tl.to('#forma1', { attr: { cx: 300 }, duration: 1 })
  .to('#forma2', { attr: { r: 50 }, duration: 0.5 }, '-=0.3')
  .to('#forma3', { fill: '#2ecc71', duration: 0.8 });

// Morph de paths (necessita MorphSVGPlugin ou Flubber)
// gsap.to('#path1', { morphSVG: '#path2', duration: 2 });
```

### Manipulacao de SVG com JavaScript

Diferente do Canvas, elementos SVG fazem parte do DOM e podem ser criados, modificados e removidos dinamicamente:

#### Criando Elementos SVG via JavaScript

```javascript
const SVG_NS = 'http://www.w3.org/2000/svg';

// O namespace e obrigatorio ao criar elementos SVG via JS
const svg = document.createElementNS(SVG_NS, 'svg');
svg.setAttribute('viewBox', '0 0 400 300');
svg.setAttribute('width', '400');
svg.setAttribute('height', '300');

// Criar um circulo
const circulo = document.createElementNS(SVG_NS, 'circle');
circulo.setAttribute('cx', '200');
circulo.setAttribute('cy', '150');
circulo.setAttribute('r', '50');
circulo.setAttribute('fill', '#3498db');
svg.appendChild(circulo);

// Criar um retangulo
const rect = document.createElementNS(SVG_NS, 'rect');
rect.setAttribute('x', '50');
rect.setAttribute('y', '50');
rect.setAttribute('width', '100');
rect.setAttribute('height', '60');
rect.setAttribute('fill', '#e74c3c');
rect.setAttribute('rx', '8');
svg.appendChild(rect);

document.body.appendChild(svg);
```

#### Modificando Atributos Existentes

```javascript
const circulo = document.querySelector('#meuCirculo');

// Lendo atributos
const raioAtual = circulo.getAttribute('r');
const corAtual = circulo.getAttribute('fill');

// Modificando atributos
circulo.setAttribute('r', '80');
circulo.setAttribute('fill', '#e74c3c');

// Estilo via CSS (tambem funciona para SVG)
circulo.style.fill = '#2ecc71';
circulo.style.transition = 'fill 0.3s ease, r 0.3s ease';

// Classes CSS
circulo.classList.add('ativo');
circulo.classList.toggle('destacado');
```

#### Eventos em Elementos SVG

```javascript
// Cada elemento SVG individual pode receber eventos
const barras = document.querySelectorAll('.barra');

barras.forEach((barra) => {
  barra.addEventListener('mouseenter', (e) => {
    e.target.setAttribute('opacity', '0.8');

    // Mostrar tooltip
    const valor = e.target.dataset.valor;
    const tooltip = document.getElementById('tooltip');
    tooltip.textContent = `Valor: ${valor}`;
    tooltip.setAttribute('x', e.target.getAttribute('x'));
    tooltip.setAttribute('y', parseFloat(e.target.getAttribute('y')) - 10);
    tooltip.style.display = 'block';
  });

  barra.addEventListener('mouseleave', (e) => {
    e.target.setAttribute('opacity', '1');
    document.getElementById('tooltip').style.display = 'none';
  });

  barra.addEventListener('click', (e) => {
    console.log('Barra clicada:', e.target.dataset.label);
  });
});
```

#### Gerando Graficos Dinamicamente

```javascript
// Exemplo: grafico de barras gerado com dados dinamicos
function criarGraficoBarras(container, dados, opcoes = {}) {
  const {
    largura = 500,
    altura = 300,
    margem = { top: 20, right: 20, bottom: 40, left: 50 },
    cores = ['#3498db', '#2ecc71', '#e74c3c', '#f39c12', '#9b59b6'],
  } = opcoes;

  const SVG_NS = 'http://www.w3.org/2000/svg';
  const svg = document.createElementNS(SVG_NS, 'svg');
  svg.setAttribute('viewBox', `0 0 ${largura} ${altura}`);
  svg.style.width = '100%';
  svg.style.maxWidth = largura + 'px';

  const areaLargura = largura - margem.left - margem.right;
  const areaAltura = altura - margem.top - margem.bottom;
  const maxValor = Math.max(...dados.map(d => d.valor));
  const barWidth = areaLargura / dados.length * 0.7;
  const gap = areaLargura / dados.length * 0.3;

  // Grupo principal com margem
  const g = document.createElementNS(SVG_NS, 'g');
  g.setAttribute('transform', `translate(${margem.left}, ${margem.top})`);

  // Eixo Y (linhas de grade)
  for (let i = 0; i <= 4; i++) {
    const y = areaAltura * (1 - i / 4);
    const linha = document.createElementNS(SVG_NS, 'line');
    linha.setAttribute('x1', '0');
    linha.setAttribute('y1', y.toString());
    linha.setAttribute('x2', areaLargura.toString());
    linha.setAttribute('y2', y.toString());
    linha.setAttribute('stroke', '#eee');
    linha.setAttribute('stroke-width', '1');
    g.appendChild(linha);

    const label = document.createElementNS(SVG_NS, 'text');
    label.setAttribute('x', '-8');
    label.setAttribute('y', (y + 4).toString());
    label.setAttribute('text-anchor', 'end');
    label.setAttribute('font-size', '11');
    label.setAttribute('fill', '#999');
    label.textContent = Math.round(maxValor * i / 4).toString();
    g.appendChild(label);
  }

  // Barras
  dados.forEach((d, i) => {
    const barHeight = (d.valor / maxValor) * areaAltura;
    const x = i * (areaLargura / dados.length) + gap / 2;
    const y = areaAltura - barHeight;

    const rect = document.createElementNS(SVG_NS, 'rect');
    rect.setAttribute('x', x.toString());
    rect.setAttribute('y', y.toString());
    rect.setAttribute('width', barWidth.toString());
    rect.setAttribute('height', barHeight.toString());
    rect.setAttribute('fill', cores[i % cores.length]);
    rect.setAttribute('rx', '4');
    rect.dataset.valor = d.valor;
    rect.dataset.label = d.label;
    rect.style.cursor = 'pointer';
    rect.style.transition = 'opacity 0.2s';

    rect.addEventListener('mouseenter', () => { rect.setAttribute('opacity', '0.7'); });
    rect.addEventListener('mouseleave', () => { rect.setAttribute('opacity', '1'); });

    g.appendChild(rect);

    // Label do eixo X
    const text = document.createElementNS(SVG_NS, 'text');
    text.setAttribute('x', (x + barWidth / 2).toString());
    text.setAttribute('y', (areaAltura + 20).toString());
    text.setAttribute('text-anchor', 'middle');
    text.setAttribute('font-size', '12');
    text.setAttribute('fill', '#666');
    text.textContent = d.label;
    g.appendChild(text);

    // Valor no topo da barra
    const valorText = document.createElementNS(SVG_NS, 'text');
    valorText.setAttribute('x', (x + barWidth / 2).toString());
    valorText.setAttribute('y', (y - 5).toString());
    valorText.setAttribute('text-anchor', 'middle');
    valorText.setAttribute('font-size', '11');
    valorText.setAttribute('font-weight', 'bold');
    valorText.setAttribute('fill', '#333');
    valorText.textContent = d.valor.toString();
    g.appendChild(valorText);
  });

  svg.appendChild(g);
  container.appendChild(svg);
}

// Uso
criarGraficoBarras(document.getElementById('grafico'), [
  { label: 'Jan', valor: 65 },
  { label: 'Fev', valor: 42 },
  { label: 'Mar', valor: 88 },
  { label: 'Abr', valor: 55 },
  { label: 'Mai', valor: 73 },
  { label: 'Jun', valor: 91 },
]);
```

#### SVG com Frameworks (React/Vue/Angular)

Em frameworks modernos, SVG inline e tratado como JSX/template nativo:

```javascript
// React — SVG como componente
function IconeCoracao({ tamanho = 24, cor = 'currentColor', preenchido = false }) {
  return (
    <svg width={tamanho} height={tamanho} viewBox="0 0 24 24"
         fill={preenchido ? cor : 'none'} stroke={cor} strokeWidth="2"
         strokeLinecap="round" strokeLinejoin="round">
      <path d="M20.84 4.61a5.5 5.5 0 00-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 00-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 000-7.78z" />
    </svg>
  );
}

// Uso: <IconeCoracao tamanho={32} cor="#e74c3c" preenchido />
```

```html
<!-- Vue — SVG inline no template -->
<template>
  <svg :width="tamanho" :height="tamanho" viewBox="0 0 24 24"
       :fill="preenchido ? cor : 'none'" :stroke="cor">
    <path d="M20.84 4.61a5.5 5.5 0 00-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 00-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 000-7.78z" />
  </svg>
</template>

<script setup>
defineProps({
  tamanho: { type: Number, default: 24 },
  cor: { type: String, default: 'currentColor' },
  preenchido: { type: Boolean, default: false },
});
</script>
```

> **Bibliotecas de icones SVG para frameworks:**
> - React: `react-icons`, `@heroicons/react`, `lucide-react`
> - Vue: `vue-feather-icons`, `@heroicons/vue`, `lucide-vue`
> - Angular: `@angular/material` (Material Icons), `angular-feather`

### SVG Inline vs img vs object

| Metodo | Estilizacao CSS | Scripting JS | Cache | Performance | Acessibilidade |
|---|---|---|---|---|---|
| `<svg>` inline | Total — CSS externo e interno | Total — acesso ao DOM do SVG | Nao — faz parte do HTML | Renderiza com o HTML; aumenta tamanho do documento | Total — ARIA, title, desc |
| `<img src="x.svg">` | Nenhuma — CSS externo nao afeta | Nenhuma | Sim — cacheado pelo navegador | Boa — carregamento paralelo | Limitada — alt apenas |
| `<object data="x.svg">` | Parcial — CSS interno do SVG | Parcial — via contentDocument | Sim | Boa | Limitada |
| CSS `background-image` | Nenhuma | Nenhuma | Sim | Boa | Nenhuma — decorativo |
| `<iframe src="x.svg">` | Parcial — CSS interno | Parcial — via contentDocument | Sim | Boa | Ruim |

> **Recomendacao:** use SVG inline para icones interativos e componentes que precisam de estilizacao dinamica. Use `<img>` para ilustracoes estaticas. Use `<object>` quando precisa de cache mas quer manter scripting interno.

### SVG Responsivo e Acessibilidade

#### SVG Responsivo

```html
<!-- SVG que se adapta ao container -->
<svg viewBox="0 0 200 100" role="img" aria-labelledby="titulo desc"
     style="width: 100%; height: auto; max-width: 600px;">

  <title id="titulo">Grafico de Barras - Vendas 2024</title>
  <desc id="desc">
    Grafico de barras mostrando vendas trimestrais:
    Q1: 45 unidades, Q2: 72 unidades, Q3: 58 unidades, Q4: 85 unidades.
  </desc>

  <!-- Eixos -->
  <line x1="30" y1="10" x2="30" y2="85" stroke="#666" stroke-width="0.5" />
  <line x1="30" y1="85" x2="190" y2="85" stroke="#666" stroke-width="0.5" />

  <!-- Barras com ARIA -->
  <g role="list" aria-label="Dados de vendas">
    <g role="listitem">
      <rect x="40" y="51" width="25" height="34" fill="#3498db" rx="2">
        <title>Q1: 45 unidades</title>
      </rect>
      <text x="52" y="95" text-anchor="middle" font-size="6" fill="#666">Q1</text>
    </g>
    <g role="listitem">
      <rect x="75" y="26" width="25" height="59" fill="#2ecc71" rx="2">
        <title>Q2: 72 unidades</title>
      </rect>
      <text x="87" y="95" text-anchor="middle" font-size="6" fill="#666">Q2</text>
    </g>
    <g role="listitem">
      <rect x="110" y="37" width="25" height="48" fill="#e67e22" rx="2">
        <title>Q3: 58 unidades</title>
      </rect>
      <text x="122" y="95" text-anchor="middle" font-size="6" fill="#666">Q3</text>
    </g>
    <g role="listitem">
      <rect x="145" y="15" width="25" height="70" fill="#e74c3c" rx="2">
        <title>Q4: 85 unidades</title>
      </rect>
      <text x="157" y="95" text-anchor="middle" font-size="6" fill="#666">Q4</text>
    </g>
  </g>
</svg>
```

> **Acessibilidade SVG — Checklist:**
> - Use `role="img"` no `<svg>` para graficos informativos
> - Adicione `<title>` como primeiro filho do `<svg>` (titulo curto)
> - Adicione `<desc>` para descricao detalhada
> - Use `aria-labelledby` referenciando `<title>` e `<desc>`
> - Para SVGs decorativos, use `aria-hidden="true"` e `role="presentation"`
> - Cada forma interativa deve ter `<title>` ou `aria-label`

### Otimizacao de SVG

#### SVGO (SVG Optimizer)

```bash
# Instalacao
npm install -g svgo

# Otimizar um arquivo
svgo icone.svg -o icone.min.svg

# Otimizar todos os SVGs de uma pasta
svgo -f ./icons -o ./icons-otimizados

# Com configuracao personalizada
svgo icone.svg --config svgo.config.js
```

Arquivo de configuracao `svgo.config.js`:

```javascript
module.exports = {
  plugins: [
    'preset-default',           // Conjunto padrao de otimizacoes
    'removeDimensions',         // Remove width/height (usa viewBox)
    'sortAttrs',                // Ordena atributos (melhora diff)
    {
      name: 'removeAttrs',
      params: {
        attrs: ['data-name'],   // Remove atributos especificos
      },
    },
    {
      name: 'preset-default',
      params: {
        overrides: {
          removeViewBox: false,  // NUNCA remover viewBox
          cleanupIds: false,     // Manter IDs se usados por CSS/JS
        },
      },
    },
  ],
};
```

#### Tecnicas Manuais de Otimizacao

1. **Simplificar paths** — reduzir o numero de pontos de controle
2. **Remover metadados** — editores como Illustrator/Inkscape adicionam metadados desnecessarios
3. **Usar formas primitivas** — `<circle>` e menor que `<path>` equivalente
4. **Reutilizar com `<use>`** — evita duplicacao de formas repetidas
5. **Remover grupos desnecessarios** — `<g>` sem atributos pode ser eliminado
6. **Reduzir casas decimais** — `M 10.123456 20.789012` pode virar `M 10.12 20.79`

```html
<!-- Antes (exportado do Illustrator): ~800 bytes -->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
     version="1.1" id="Camada_1" x="0px" y="0px" width="24px" height="24px"
     viewBox="0 0 24 24" style="enable-background:new 0 0 24 24;" xml:space="preserve">
  <g>
    <path d="M 12.000000,2.000000 C 6.480000,2.000000 2.000000,6.480000 2.000000,12.000000 ..."/>
  </g>
</svg>

<!-- Depois (otimizado): ~200 bytes -->
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <path d="M12 2C6.48 2 2 6.48 2 12..."/>
</svg>
```

### Exemplo Pratico: Conjunto de Icones Interativos

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Icones SVG Interativos</title>
  <style>
    .icon-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
      gap: 20px; max-width: 600px; margin: 40px auto;
    }
    .icon-card {
      text-align: center; padding: 20px;
      border-radius: 12px; background: #fff;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
      cursor: pointer; transition: transform 0.2s, box-shadow 0.2s;
    }
    .icon-card:hover {
      transform: translateY(-4px);
      box-shadow: 0 8px 24px rgba(0,0,0,0.12);
    }
    .icon-card svg {
      width: 48px; height: 48px;
      transition: transform 0.3s ease;
    }
    .icon-card:hover svg { transform: scale(1.15); }
    .icon-card p {
      margin-top: 8px; font-size: 13px; color: #666;
      font-family: Arial, sans-serif;
    }

    /* Animacoes CSS nos icones SVG */
    .icon-card:hover .stroke-animate {
      stroke-dashoffset: 0;
    }
    .stroke-animate {
      stroke-dasharray: 100;
      stroke-dashoffset: 100;
      transition: stroke-dashoffset 0.6s ease;
    }
    .icon-card:hover .heart-icon { fill: #e74c3c; }
    .heart-icon { fill: #ccc; transition: fill 0.3s ease; }
  </style>
</head>
<body>

<div class="icon-grid">
  <!-- Icone Home -->
  <div class="icon-card" role="button" aria-label="Inicio">
    <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M3 12L12 3l9 9" stroke="#3498db" stroke-width="2"
            stroke-linecap="round" stroke-linejoin="round" class="stroke-animate"/>
      <path d="M5 10v9a1 1 0 001 1h3v-5h6v5h3a1 1 0 001-1v-9"
            stroke="#3498db" stroke-width="2" stroke-linecap="round"
            stroke-linejoin="round"/>
    </svg>
    <p>Inicio</p>
  </div>

  <!-- Icone Configuracao (engrenagem) -->
  <div class="icon-card" role="button" aria-label="Configuracoes">
    <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <style>
        .engrenagem:hover .gear { animation: girarGear 3s linear infinite; transform-origin: 12px 12px; }
        @keyframes girarGear { to { transform: rotate(360deg); } }
      </style>
      <g class="gear">
        <circle cx="12" cy="12" r="3" stroke="#9b59b6" stroke-width="2"/>
        <path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 11-2.83 2.83l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 11-4 0v-.09A1.65 1.65 0 009 19.4a1.65 1.65 0 00-1.82.33l-.06.06a2 2 0 11-2.83-2.83l.06-.06A1.65 1.65 0 004.68 15a1.65 1.65 0 00-1.51-1H3a2 2 0 110-4h.09A1.65 1.65 0 004.6 9a1.65 1.65 0 00-.33-1.82l-.06-.06a2 2 0 112.83-2.83l.06.06A1.65 1.65 0 009 4.68a1.65 1.65 0 001-1.51V3a2 2 0 114 0v.09a1.65 1.65 0 001 1.51 1.65 1.65 0 001.82-.33l.06-.06a2 2 0 112.83 2.83l-.06.06A1.65 1.65 0 0019.4 9a1.65 1.65 0 001.51 1H21a2 2 0 110 4h-.09a1.65 1.65 0 00-1.51 1z"
              stroke="#9b59b6" stroke-width="2"/>
      </g>
    </svg>
    <p>Config</p>
  </div>

  <!-- Icone Coracao (toggle) -->
  <div class="icon-card" role="button" aria-label="Favoritar">
    <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
      <path class="heart-icon"
            d="M20.84 4.61a5.5 5.5 0 00-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 00-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 000-7.78z"
            stroke="#e74c3c" stroke-width="1.5"/>
    </svg>
    <p>Favorito</p>
  </div>

  <!-- Icone Notificacao (sino com badge) -->
  <div class="icon-card" role="button" aria-label="Notificacoes">
    <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
      <style>
        .icon-card:hover .sino {
          animation: balanco 0.5s ease-in-out;
          transform-origin: 12px 2px;
        }
        @keyframes balanco {
          0%, 100% { transform: rotate(0); }
          25% { transform: rotate(15deg); }
          75% { transform: rotate(-15deg); }
        }
      </style>
      <path class="sino"
            d="M18 8A6 6 0 006 8c0 7-3 9-3 9h18s-3-2-3-9M13.73 21a2 2 0 01-3.46 0"
            stroke="#e67e22" stroke-width="2" stroke-linecap="round"
            stroke-linejoin="round"/>
      <circle cx="18" cy="5" r="4" fill="#e74c3c"/>
      <text x="18" y="7" text-anchor="middle" font-size="6"
            fill="#fff" font-weight="bold">3</text>
    </svg>
    <p>Alertas</p>
  </div>
</div>

<script>
  // Toggle do coracao
  document.querySelector('.heart-icon').addEventListener('click', function() {
    const isFavorito = this.style.fill === 'rgb(231, 76, 60)';
    this.style.fill = isFavorito ? '#ccc' : '#e74c3c';
  });
</script>

</body>
</html>
```

> **Dicas para icones SVG em producao:**
> - Use um sistema de sprites SVG ou componentes (React/Vue/Angular) para gerenciar icones
> - Bibliotecas como [Lucide](https://lucide.dev/), [Heroicons](https://heroicons.com/) e [Phosphor Icons](https://phosphoricons.com/) oferecem icones SVG otimizados com componentes prontos
> - Para icones monocromaticos, use `currentColor` como `fill`/`stroke` para herdar a cor do CSS pai

### Exemplo Pratico: Infografico SVG com Efeitos de Hover

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Infografico SVG - Tecnologias Web</title>
  <style>
    body {
      font-family: 'Segoe UI', Arial, sans-serif;
      display: flex; justify-content: center;
      padding: 40px; background: #f0f2f5;
    }
    .info-card {
      cursor: pointer;
      transition: transform 0.3s ease, filter 0.3s ease;
    }
    .info-card:hover { transform: scale(1.05); filter: brightness(1.1); }
    .info-card text { pointer-events: none; }
    .tooltip-box { opacity: 0; transition: opacity 0.3s; pointer-events: none; }
    .info-card:hover + .tooltip-box,
    .info-card:hover ~ .tooltip-box[data-for] { opacity: 1; }
  </style>
</head>
<body>

<svg viewBox="0 0 700 420" xmlns="http://www.w3.org/2000/svg"
     style="max-width: 700px; width: 100%;" role="img"
     aria-labelledby="infoTitulo infoDesc">

  <title id="infoTitulo">Ecossistema de Tecnologias Web</title>
  <desc id="infoDesc">Diagrama mostrando tres pilares do desenvolvimento web:
    Frontend, Backend e DevOps, com suas principais tecnologias.</desc>

  <defs>
    <linearGradient id="gradFront" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="#3498db"/>
      <stop offset="100%" stop-color="#2980b9"/>
    </linearGradient>
    <linearGradient id="gradBack" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="#2ecc71"/>
      <stop offset="100%" stop-color="#27ae60"/>
    </linearGradient>
    <linearGradient id="gradDevOps" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="#e67e22"/>
      <stop offset="100%" stop-color="#d35400"/>
    </linearGradient>
    <filter id="sombra3d">
      <feDropShadow dx="2" dy="4" stdDeviation="4" flood-color="rgba(0,0,0,0.15)"/>
    </filter>
  </defs>

  <!-- Titulo -->
  <text x="350" y="35" text-anchor="middle" font-size="22"
        font-weight="bold" fill="#2c3e50">Ecossistema Web</text>
  <line x1="200" y1="45" x2="500" y2="45" stroke="#bdc3c7" stroke-width="1"/>

  <!-- Card Frontend -->
  <g class="info-card" filter="url(#sombra3d)">
    <rect x="30" y="70" width="200" height="320" rx="12" fill="url(#gradFront)"/>
    <text x="130" y="105" text-anchor="middle" font-size="18"
          font-weight="bold" fill="#fff">Frontend</text>
    <line x1="60" y1="115" x2="200" y2="115" stroke="rgba(255,255,255,0.3)"/>
    <text x="50" y="145" font-size="13" fill="rgba(255,255,255,0.9)">HTML5 / CSS3</text>
    <text x="50" y="170" font-size="13" fill="rgba(255,255,255,0.9)">JavaScript / TS</text>
    <text x="50" y="195" font-size="13" fill="rgba(255,255,255,0.9)">React / Angular / Vue</text>
    <text x="50" y="220" font-size="13" fill="rgba(255,255,255,0.9)">Tailwind / Bootstrap</text>
    <text x="50" y="245" font-size="13" fill="rgba(255,255,255,0.9)">Canvas / SVG / WebGL</text>
    <text x="50" y="270" font-size="13" fill="rgba(255,255,255,0.9)">PWA / Service Workers</text>
    <text x="50" y="295" font-size="13" fill="rgba(255,255,255,0.9)">Webpack / Vite</text>
    <!-- Icone decorativo -->
    <circle cx="130" cy="355" r="18" fill="rgba(255,255,255,0.15)"/>
    <text x="130" y="361" text-anchor="middle" font-size="20" fill="#fff">&#60;/&#62;</text>
  </g>

  <!-- Card Backend -->
  <g class="info-card" filter="url(#sombra3d)">
    <rect x="250" y="70" width="200" height="320" rx="12" fill="url(#gradBack)"/>
    <text x="350" y="105" text-anchor="middle" font-size="18"
          font-weight="bold" fill="#fff">Backend</text>
    <line x1="280" y1="115" x2="420" y2="115" stroke="rgba(255,255,255,0.3)"/>
    <text x="270" y="145" font-size="13" fill="rgba(255,255,255,0.9)">Java / Spring Boot</text>
    <text x="270" y="170" font-size="13" fill="rgba(255,255,255,0.9)">Node.js / Express</text>
    <text x="270" y="195" font-size="13" fill="rgba(255,255,255,0.9)">Python / Django</text>
    <text x="270" y="220" font-size="13" fill="rgba(255,255,255,0.9)">REST / GraphQL</text>
    <text x="270" y="245" font-size="13" fill="rgba(255,255,255,0.9)">PostgreSQL / MongoDB</text>
    <text x="270" y="270" font-size="13" fill="rgba(255,255,255,0.9)">Redis / RabbitMQ</text>
    <text x="270" y="295" font-size="13" fill="rgba(255,255,255,0.9)">JPA / Hibernate</text>
    <circle cx="350" cy="355" r="18" fill="rgba(255,255,255,0.15)"/>
    <text x="350" y="361" text-anchor="middle" font-size="20" fill="#fff">{ }</text>
  </g>

  <!-- Card DevOps -->
  <g class="info-card" filter="url(#sombra3d)">
    <rect x="470" y="70" width="200" height="320" rx="12" fill="url(#gradDevOps)"/>
    <text x="570" y="105" text-anchor="middle" font-size="18"
          font-weight="bold" fill="#fff">DevOps</text>
    <line x1="500" y1="115" x2="640" y2="115" stroke="rgba(255,255,255,0.3)"/>
    <text x="490" y="145" font-size="13" fill="rgba(255,255,255,0.9)">Docker / Kubernetes</text>
    <text x="490" y="170" font-size="13" fill="rgba(255,255,255,0.9)">CI/CD (GitHub Actions)</text>
    <text x="490" y="195" font-size="13" fill="rgba(255,255,255,0.9)">AWS / Azure / GCP</text>
    <text x="490" y="220" font-size="13" fill="rgba(255,255,255,0.9)">Terraform / Ansible</text>
    <text x="490" y="245" font-size="13" fill="rgba(255,255,255,0.9)">Prometheus / Grafana</text>
    <text x="490" y="270" font-size="13" fill="rgba(255,255,255,0.9)">Git / Branching</text>
    <text x="490" y="295" font-size="13" fill="rgba(255,255,255,0.9)">Seguranca / SAST</text>
    <circle cx="570" cy="355" r="18" fill="rgba(255,255,255,0.15)"/>
    <text x="570" y="361" text-anchor="middle" font-size="20" fill="#fff">&#9881;</text>
  </g>

  <!-- Setas de conexao -->
  <defs>
    <marker id="setaPonta" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="#95a5a6"/>
    </marker>
  </defs>
  <line x1="230" y1="230" x2="250" y2="230" stroke="#95a5a6" stroke-width="2"
        marker-end="url(#setaPonta)" marker-start="url(#setaPonta)"/>
  <line x1="450" y1="230" x2="470" y2="230" stroke="#95a5a6" stroke-width="2"
        marker-end="url(#setaPonta)" marker-start="url(#setaPonta)"/>

  <!-- Rodape -->
  <text x="350" y="412" text-anchor="middle" font-size="11" fill="#95a5a6">
    Cada area possui tecnologias especializadas que se integram no ciclo de desenvolvimento
  </text>
</svg>

</body>
</html>
```

> Este infografico demonstra: gradientes, filtros de sombra, agrupamento com `<g>`, markers para setas, transformacoes via CSS hover, acessibilidade com `<title>` e `<desc>`, e layout responsivo com `viewBox`.
## 4. D3.js — Data-Driven Documents

D3.js (Data-Driven Documents) é uma biblioteca JavaScript para manipular documentos baseados em dados. Diferente de bibliotecas de gráficos prontos, D3 oferece controle total sobre cada pixel renderizado, utilizando SVG, Canvas e HTML para transformar dados em visualizações interativas.

> Referência: [D3.js Official](https://d3js.org/) · [D3 Gallery](https://observablehq.com/@d3/gallery) · [D3 API Reference](https://d3js.org/d3-selection)

---

### Conceitos Fundamentais

D3 não é uma biblioteca de gráficos — é um conjunto de módulos para vincular dados a elementos do DOM e aplicar transformações orientadas por dados. Sua filosofia central é o **data join**: associar arrays de dados a elementos visuais de forma declarativa.

| Conceito | Descrição |
|---|---|
| **Data Join** | Vinculação de dados a elementos DOM — o coração do D3 |
| **Selections** | Wrappers sobre conjuntos de elementos DOM |
| **Scales** | Funções que mapeiam domínios de dados para intervalos visuais |
| **Shapes** | Geradores de caminhos SVG (linhas, arcos, áreas) |
| **Layouts** | Algoritmos que transformam dados em coordenadas (tree, force, treemap) |
| **Transitions** | Animações interpoladas entre estados |

**Instalação:**

```bash
# Via npm (recomendado para projetos com bundler)
npm install d3

# Ou módulos individuais para tree-shaking
npm install d3-selection d3-scale d3-axis d3-shape d3-transition
```

```html
<!-- Via CDN -->
<script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
```

```javascript
// Importação modular (ES Modules)
import * as d3 from 'd3';

// Ou importações seletivas (melhor para bundle size)
import { select, selectAll } from 'd3-selection';
import { scaleLinear } from 'd3-scale';
```

**Selections — Selecionando elementos:**

Selections são o ponto de partida de qualquer operação em D3. Funcionam de forma similar ao `querySelector` e `querySelectorAll`, mas retornam objetos D3 encadeáveis.

```javascript
// Selecionar um único elemento (primeiro encontrado)
const svg = d3.select('#meu-grafico');

// Selecionar múltiplos elementos
const circulos = d3.selectAll('circle');

// Encadeamento — selecionar e modificar
d3.select('#meu-grafico')
  .attr('width', 600)
  .attr('height', 400)
  .style('background', '#f0f0f0');

// Seleção aninhada
d3.select('#container')
  .selectAll('div')
  .style('color', 'blue');
```

**Data Binding — Vinculando dados:**

O data binding é o mecanismo que conecta um array de dados a uma seleção de elementos DOM. Cada dado é associado a um elemento correspondente.

```javascript
const dados = [10, 20, 30, 40, 50];

// Vincular dados a parágrafos existentes
d3.selectAll('p')
  .data(dados)
  .text(d => `Valor: ${d}`);
```

**Enter/Update/Exit — Padrão Clássico:**

Quando a quantidade de dados não corresponde à quantidade de elementos, D3 divide a seleção em três grupos:

```
Dados:     [A] [B] [C] [D] [E]
Elementos: <p> <p> <p>

Enter:                 [D] [E]     → criar novos elementos
Update:    [A] [B] [C]            → atualizar existentes
Exit:      (nenhum)               → remover excedentes
```

```javascript
const dados = [10, 20, 30, 40, 50];

// Padrão clássico (D3 v3/v4 — ainda funcional)
const paragrafos = d3.select('#container')
  .selectAll('p')
  .data(dados);

// Enter — criar elementos para dados novos
paragrafos.enter()
  .append('p')
  .text(d => `Valor: ${d}`)
  .style('color', 'green');

// Update — atualizar elementos existentes
paragrafos
  .text(d => `Atualizado: ${d}`)
  .style('color', 'blue');

// Exit — remover elementos sem dados correspondentes
paragrafos.exit()
  .remove();
```

**Padrão Moderno com .join() (D3 v5+):**

O método `.join()` simplifica o padrão Enter/Update/Exit em uma única chamada:

```javascript
const dados = [10, 20, 30, 40, 50];

// Forma simplificada — cria elementos automaticamente
d3.select('#container')
  .selectAll('p')
  .data(dados)
  .join('p')
  .text(d => `Valor: ${d}`)
  .style('font-size', d => `${d}px`);

// Forma completa — controle individual de enter, update e exit
d3.select('#container')
  .selectAll('p')
  .data(dados, d => d.id) // key function para identificar dados
  .join(
    enter => enter.append('p')
      .style('opacity', 0)
      .call(enter => enter.transition()
        .style('opacity', 1)),
    update => update
      .style('color', 'blue'),
    exit => exit
      .call(exit => exit.transition()
        .style('opacity', 0)
        .remove())
  )
  .text(d => `Valor: ${d}`);
```

> **Dica:** Prefira `.join()` ao padrão clássico Enter/Update/Exit. Além de mais conciso, `.join()` gerencia automaticamente a mesclagem das seleções enter e update.

**Exemplo completo — Criando círculos a partir de dados:**

```html
<svg id="circulos" width="500" height="100"></svg>

<script>
const dados = [
  { x: 50, y: 50, r: 20, cor: '#e63946' },
  { x: 150, y: 50, r: 30, cor: '#457b9d' },
  { x: 280, y: 50, r: 25, cor: '#2a9d8f' },
  { x: 400, y: 50, r: 35, cor: '#e9c46a' }
];

d3.select('#circulos')
  .selectAll('circle')
  .data(dados)
  .join('circle')
  .attr('cx', d => d.x)
  .attr('cy', d => d.y)
  .attr('r', d => d.r)
  .attr('fill', d => d.cor);
</script>
```

---

### Escalas e Eixos

Escalas são funções que mapeiam um **domínio** (intervalo dos dados) para um **range** (intervalo visual em pixels). Eixos são componentes visuais que representam escalas com ticks e labels.

**Tipos de Escalas:**

| Escala | Entrada | Saída | Uso Típico |
|---|---|---|---|
| `scaleLinear` | Contínua (números) | Contínua | Posições, tamanhos |
| `scaleLog` | Contínua (logarítmica) | Contínua | Dados com grande amplitude |
| `scaleSqrt` | Contínua (raiz quadrada) | Contínua | Áreas de círculos |
| `scaleBand` | Discreta (categorias) | Contínua (faixas) | Barras de gráficos |
| `scaleOrdinal` | Discreta | Discreta | Mapeamento de cores por categoria |
| `scaleTime` | Temporal (Date) | Contínua | Eixos de tempo |
| `scalePoint` | Discreta | Contínua (pontos) | Posições sem largura de banda |

```javascript
import { scaleLinear, scaleBand, scaleOrdinal, scaleTime, scaleLog } from 'd3-scale';
import { schemeCategory10 } from 'd3-scale-chromatic';

// scaleLinear — mapeia valores numéricos para pixels
const escalaY = d3.scaleLinear()
  .domain([0, 100])     // dados: de 0 a 100
  .range([400, 0]);     // pixels: de 400 (baixo) a 0 (topo) — invertido no SVG

console.log(escalaY(50));  // 200 — metade do range
console.log(escalaY(0));   // 400 — valor mínimo = base do gráfico

// scaleLog — para dados com grande variação
const escalaLog = d3.scaleLog()
  .domain([1, 1000000])
  .range([0, 600]);

// scaleBand — para categorias em gráficos de barras
const escalaX = d3.scaleBand()
  .domain(['Jan', 'Fev', 'Mar', 'Abr', 'Mai'])
  .range([0, 500])
  .padding(0.2);        // espaço entre barras (0 a 1)

console.log(escalaX('Mar'));        // posição x da barra "Mar"
console.log(escalaX.bandwidth());   // largura de cada barra

// scaleOrdinal — mapear categorias para cores
const escalaCor = d3.scaleOrdinal()
  .domain(['Frontend', 'Backend', 'DevOps'])
  .range(['#e63946', '#457b9d', '#2a9d8f']);

// Ou usar esquemas de cores prontos do D3
const escalaCor2 = d3.scaleOrdinal(d3.schemeCategory10);

// scaleTime — para datas
const escalaTempoX = d3.scaleTime()
  .domain([new Date('2024-01-01'), new Date('2024-12-31')])
  .range([0, 800]);
```

**Eixos (Axes):**

```javascript
const largura = 600;
const altura = 400;
const margem = { top: 20, right: 30, bottom: 40, left: 50 };
const larguraInterna = largura - margem.left - margem.right;
const alturaInterna = altura - margem.top - margem.bottom;

// Criar SVG com grupo para margens
const svg = d3.select('#grafico')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

// Definir escalas
const x = d3.scaleBand()
  .domain(['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'])
  .range([0, larguraInterna])
  .padding(0.2);

const y = d3.scaleLinear()
  .domain([0, 100])
  .range([alturaInterna, 0])
  .nice(); // arredonda os limites do domínio

// Criar eixo X (bottom)
svg.append('g')
  .attr('transform', `translate(0,${alturaInterna})`)
  .call(d3.axisBottom(x))
  .selectAll('text')
  .style('font-size', '12px');

// Criar eixo Y (left)
svg.append('g')
  .call(d3.axisLeft(y).ticks(5).tickFormat(d => `${d}%`))
  .selectAll('text')
  .style('font-size', '12px');

// Label do eixo Y
svg.append('text')
  .attr('transform', 'rotate(-90)')
  .attr('y', -margem.left + 15)
  .attr('x', -alturaInterna / 2)
  .attr('text-anchor', 'middle')
  .text('Percentual (%)');
```

> **Dica:** Use `.nice()` nas escalas lineares para arredondar os limites do domínio para valores mais legíveis (ex.: 97 vira 100).

---

### Gráficos Básicos e Avançados

**Gráfico de Barras Vertical:**

```html
<div id="barras-vertical"></div>

<script>
const dadosBarras = [
  { mes: 'Jan', vendas: 45 },
  { mes: 'Fev', vendas: 72 },
  { mes: 'Mar', vendas: 58 },
  { mes: 'Abr', vendas: 90 },
  { mes: 'Mai', vendas: 63 },
  { mes: 'Jun', vendas: 85 }
];

const largura = 600, altura = 400;
const margem = { top: 20, right: 20, bottom: 40, left: 50 };
const w = largura - margem.left - margem.right;
const h = altura - margem.top - margem.bottom;

const svg = d3.select('#barras-vertical')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

const x = d3.scaleBand()
  .domain(dadosBarras.map(d => d.mes))
  .range([0, w])
  .padding(0.2);

const y = d3.scaleLinear()
  .domain([0, d3.max(dadosBarras, d => d.vendas)])
  .range([h, 0])
  .nice();

// Eixos
svg.append('g')
  .attr('transform', `translate(0,${h})`)
  .call(d3.axisBottom(x));

svg.append('g')
  .call(d3.axisLeft(y));

// Barras
svg.selectAll('rect')
  .data(dadosBarras)
  .join('rect')
  .attr('x', d => x(d.mes))
  .attr('y', d => y(d.vendas))
  .attr('width', x.bandwidth())
  .attr('height', d => h - y(d.vendas))
  .attr('fill', '#457b9d')
  .attr('rx', 4); // bordas arredondadas

// Labels nas barras
svg.selectAll('.label')
  .data(dadosBarras)
  .join('text')
  .attr('class', 'label')
  .attr('x', d => x(d.mes) + x.bandwidth() / 2)
  .attr('y', d => y(d.vendas) - 5)
  .attr('text-anchor', 'middle')
  .style('font-size', '12px')
  .text(d => d.vendas);
</script>
```

**Gráfico de Barras Horizontal:**

```javascript
const dadosHorizontal = [
  { linguagem: 'JavaScript', uso: 65 },
  { linguagem: 'Python', uso: 48 },
  { linguagem: 'Java', uso: 35 },
  { linguagem: 'TypeScript', uso: 30 },
  { linguagem: 'C#', uso: 27 }
];

const largura = 600, altura = 300;
const margem = { top: 20, right: 30, bottom: 30, left: 100 };
const w = largura - margem.left - margem.right;
const h = altura - margem.top - margem.bottom;

const svg = d3.select('#barras-horizontal')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

const x = d3.scaleLinear()
  .domain([0, d3.max(dadosHorizontal, d => d.uso)])
  .range([0, w])
  .nice();

const y = d3.scaleBand()
  .domain(dadosHorizontal.map(d => d.linguagem))
  .range([0, h])
  .padding(0.3);

svg.append('g')
  .attr('transform', `translate(0,${h})`)
  .call(d3.axisBottom(x).ticks(5).tickFormat(d => `${d}%`));

svg.append('g')
  .call(d3.axisLeft(y));

svg.selectAll('rect')
  .data(dadosHorizontal)
  .join('rect')
  .attr('x', 0)
  .attr('y', d => y(d.linguagem))
  .attr('width', d => x(d.uso))
  .attr('height', y.bandwidth())
  .attr('fill', '#2a9d8f')
  .attr('rx', 3);
```

**Gráfico de Linhas:**

```javascript
const dadosLinha = [
  { data: new Date('2024-01-01'), valor: 30 },
  { data: new Date('2024-02-01'), valor: 45 },
  { data: new Date('2024-03-01'), valor: 38 },
  { data: new Date('2024-04-01'), valor: 62 },
  { data: new Date('2024-05-01'), valor: 55 },
  { data: new Date('2024-06-01'), valor: 78 },
  { data: new Date('2024-07-01'), valor: 72 },
  { data: new Date('2024-08-01'), valor: 90 }
];

const largura = 700, altura = 400;
const margem = { top: 20, right: 30, bottom: 40, left: 50 };
const w = largura - margem.left - margem.right;
const h = altura - margem.top - margem.bottom;

const svg = d3.select('#grafico-linha')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

const x = d3.scaleTime()
  .domain(d3.extent(dadosLinha, d => d.data))
  .range([0, w]);

const y = d3.scaleLinear()
  .domain([0, d3.max(dadosLinha, d => d.valor)])
  .range([h, 0])
  .nice();

// Eixos
svg.append('g')
  .attr('transform', `translate(0,${h})`)
  .call(d3.axisBottom(x).ticks(d3.timeMonth.every(1)).tickFormat(d3.timeFormat('%b')));

svg.append('g')
  .call(d3.axisLeft(y));

// Gerador de linha
const linha = d3.line()
  .x(d => x(d.data))
  .y(d => y(d.valor))
  .curve(d3.curveMonotoneX); // suavização da curva

// Desenhar a linha
svg.append('path')
  .datum(dadosLinha)
  .attr('fill', 'none')
  .attr('stroke', '#e63946')
  .attr('stroke-width', 2.5)
  .attr('d', linha);

// Área sob a linha (opcional)
const area = d3.area()
  .x(d => x(d.data))
  .y0(h)
  .y1(d => y(d.valor))
  .curve(d3.curveMonotoneX);

svg.append('path')
  .datum(dadosLinha)
  .attr('fill', '#e6394620')
  .attr('d', area);

// Pontos nos dados
svg.selectAll('circle')
  .data(dadosLinha)
  .join('circle')
  .attr('cx', d => x(d.data))
  .attr('cy', d => y(d.valor))
  .attr('r', 4)
  .attr('fill', '#e63946');
```

**Gráfico de Pizza e Donut:**

```javascript
const dadosPizza = [
  { categoria: 'Frontend', valor: 35 },
  { categoria: 'Backend', valor: 30 },
  { categoria: 'DevOps', valor: 15 },
  { categoria: 'Mobile', valor: 12 },
  { categoria: 'Data', valor: 8 }
];

const largura = 500, altura = 400;
const raio = Math.min(largura, altura) / 2 - 40;

const svg = d3.select('#grafico-pizza')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${largura / 2},${altura / 2})`);

const cores = d3.scaleOrdinal()
  .domain(dadosPizza.map(d => d.categoria))
  .range(d3.schemeTableau10);

// Gerador de fatias
const pie = d3.pie()
  .value(d => d.valor)
  .sort(null); // manter ordem original

// Gerador de arcos — pizza completa
const arcoPizza = d3.arc()
  .innerRadius(0)      // 0 = pizza, > 0 = donut
  .outerRadius(raio);

// Gerador de arcos — donut
const arcoDonut = d3.arc()
  .innerRadius(raio * 0.5)  // raio interno cria o "buraco"
  .outerRadius(raio);

// Arco para posicionar labels
const arcoLabel = d3.arc()
  .innerRadius(raio * 0.7)
  .outerRadius(raio * 0.7);

// Desenhar fatias (usando donut neste exemplo)
const fatias = svg.selectAll('.fatia')
  .data(pie(dadosPizza))
  .join('g')
  .attr('class', 'fatia');

fatias.append('path')
  .attr('d', arcoDonut)
  .attr('fill', d => cores(d.data.categoria))
  .attr('stroke', '#fff')
  .attr('stroke-width', 2);

// Labels
fatias.append('text')
  .attr('transform', d => `translate(${arcoLabel.centroid(d)})`)
  .attr('text-anchor', 'middle')
  .style('font-size', '12px')
  .style('font-weight', 'bold')
  .text(d => `${d.data.categoria} (${d.data.valor}%)`);
```

**Scatter Plot (Gráfico de Dispersão):**

```javascript
const dadosScatter = [
  { experiencia: 1, salario: 3500, area: 'Frontend' },
  { experiencia: 2, salario: 5200, area: 'Backend' },
  { experiencia: 3, salario: 6800, area: 'Frontend' },
  { experiencia: 4, salario: 7500, area: 'DevOps' },
  { experiencia: 5, salario: 9200, area: 'Backend' },
  { experiencia: 3, salario: 5800, area: 'DevOps' },
  { experiencia: 6, salario: 11000, area: 'Frontend' },
  { experiencia: 2, salario: 4800, area: 'Frontend' },
  { experiencia: 7, salario: 12500, area: 'Backend' },
  { experiencia: 4, salario: 8200, area: 'DevOps' },
  { experiencia: 8, salario: 14000, area: 'Backend' },
  { experiencia: 1, salario: 3200, area: 'DevOps' }
];

const largura = 600, altura = 400;
const margem = { top: 20, right: 120, bottom: 40, left: 60 };
const w = largura - margem.left - margem.right;
const h = altura - margem.top - margem.bottom;

const svg = d3.select('#scatter')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

const x = d3.scaleLinear()
  .domain([0, d3.max(dadosScatter, d => d.experiencia) + 1])
  .range([0, w]);

const y = d3.scaleLinear()
  .domain([0, d3.max(dadosScatter, d => d.salario)])
  .range([h, 0])
  .nice();

const cor = d3.scaleOrdinal()
  .domain(['Frontend', 'Backend', 'DevOps'])
  .range(['#e63946', '#457b9d', '#2a9d8f']);

svg.append('g')
  .attr('transform', `translate(0,${h})`)
  .call(d3.axisBottom(x).ticks(8).tickFormat(d => `${d} anos`));

svg.append('g')
  .call(d3.axisLeft(y).tickFormat(d => `R$ ${d / 1000}k`));

// Pontos
svg.selectAll('circle')
  .data(dadosScatter)
  .join('circle')
  .attr('cx', d => x(d.experiencia))
  .attr('cy', d => y(d.salario))
  .attr('r', 6)
  .attr('fill', d => cor(d.area))
  .attr('opacity', 0.7)
  .attr('stroke', d => cor(d.area))
  .attr('stroke-width', 1.5);

// Legenda
const legenda = svg.append('g')
  .attr('transform', `translate(${w + 10}, 0)`);

['Frontend', 'Backend', 'DevOps'].forEach((area, i) => {
  const g = legenda.append('g')
    .attr('transform', `translate(0, ${i * 25})`);

  g.append('circle')
    .attr('r', 6)
    .attr('fill', cor(area));

  g.append('text')
    .attr('x', 12)
    .attr('y', 4)
    .style('font-size', '12px')
    .text(area);
});
```

---

### Mapas Geográficos com D3-Geo

D3 oferece módulos completos para visualização de dados geográficos, suportando os formatos GeoJSON e TopoJSON e dezenas de projeções cartográficas.

**GeoJSON vs TopoJSON:**

| Formato | Descrição | Tamanho | Uso |
|---|---|---|---|
| **GeoJSON** | Formato padrão para geometrias geográficas (RFC 7946) | Maior | Dados simples, compatibilidade ampla |
| **TopoJSON** | Extensão do GeoJSON com topologia compartilhada | 60-80% menor | Mapas complexos, fronteiras compartilhadas |

```javascript
// Instalar módulos de geo
// npm install d3-geo d3-geo-projection topojson-client

import { geoPath, geoMercator, geoNaturalEarth1, geoEquirectangular } from 'd3-geo';
import { feature } from 'topojson-client';
```

**Projeções Comuns:**

| Projeção | Método | Uso Típico |
|---|---|---|
| Mercator | `d3.geoMercator()` | Mapas web (similar ao Google Maps) |
| Natural Earth | `d3.geoNaturalEarth1()` | Mapas-múndi com proporções naturais |
| Equirretangular | `d3.geoEquirectangular()` | Simples, proporcional a lat/long |
| Albers USA | `d3.geoAlbersUsa()` | EUA com Alaska e Hawaii reposicionados |
| Ortográfico | `d3.geoOrthographic()` | Globo 3D |

**Carregando e renderizando um mapa:**

```javascript
const largura = 800, altura = 500;

const svg = d3.select('#mapa')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura);

// Definir projeção
const projecao = d3.geoNaturalEarth1()
  .scale(150)
  .translate([largura / 2, altura / 2]);

// Gerador de caminhos SVG a partir de geometrias geo
const caminho = d3.geoPath().projection(projecao);

// Carregar dados TopoJSON
d3.json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json')
  .then(mundo => {
    // Converter TopoJSON para GeoJSON
    const paises = topojson.feature(mundo, mundo.objects.countries);

    // Desenhar países
    svg.selectAll('path')
      .data(paises.features)
      .join('path')
      .attr('d', caminho)
      .attr('fill', '#cce5df')
      .attr('stroke', '#69b3a2')
      .attr('stroke-width', 0.5);

    // Adicionar contorno do graticule (linhas de grade)
    const graticule = d3.geoGraticule();
    svg.append('path')
      .datum(graticule())
      .attr('d', caminho)
      .attr('fill', 'none')
      .attr('stroke', '#ddd')
      .attr('stroke-width', 0.3);
  });
```

**Mapa Coroplético (Choropleth):**

```javascript
// Mapa coroplético — cores representam valores de dados por região
const largura = 800, altura = 500;

const svg = d3.select('#mapa-coropletico')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura);

const projecao = d3.geoNaturalEarth1()
  .scale(150)
  .translate([largura / 2, altura / 2]);

const caminho = d3.geoPath().projection(projecao);

// Escala de cores para o choropleth
const escalaCor = d3.scaleSequential()
  .domain([0, 100])
  .interpolator(d3.interpolateYlOrRd); // amarelo → laranja → vermelho

// Carregar mapa e dados simultaneamente
Promise.all([
  d3.json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json'),
  d3.csv('dados-paises.csv') // CSV com colunas: id, nome, indicador
]).then(([mundo, dados]) => {
  const paises = topojson.feature(mundo, mundo.objects.countries);

  // Criar mapa de lookup: id → valor
  const indicadores = new Map(dados.map(d => [d.id, +d.indicador]));

  svg.selectAll('path')
    .data(paises.features)
    .join('path')
    .attr('d', caminho)
    .attr('fill', d => {
      const valor = indicadores.get(d.id);
      return valor != null ? escalaCor(valor) : '#ccc';
    })
    .attr('stroke', '#fff')
    .attr('stroke-width', 0.5)
    .append('title') // tooltip nativo
    .text(d => {
      const valor = indicadores.get(d.id);
      return `${d.properties.name}: ${valor ?? 'N/A'}`;
    });

  // Legenda do mapa
  const legendaLargura = 300;
  const legendaAltura = 10;

  const legendaSvg = svg.append('g')
    .attr('transform', `translate(${largura - legendaLargura - 40}, ${altura - 40})`);

  const escalaLegenda = d3.scaleLinear()
    .domain([0, 100])
    .range([0, legendaLargura]);

  // Gradiente para a legenda
  const defs = svg.append('defs');
  const gradiente = defs.append('linearGradient')
    .attr('id', 'gradiente-legenda');

  gradiente.selectAll('stop')
    .data(d3.range(0, 1.01, 0.1))
    .join('stop')
    .attr('offset', d => `${d * 100}%`)
    .attr('stop-color', d => escalaCor(d * 100));

  legendaSvg.append('rect')
    .attr('width', legendaLargura)
    .attr('height', legendaAltura)
    .style('fill', 'url(#gradiente-legenda)');

  legendaSvg.append('g')
    .attr('transform', `translate(0,${legendaAltura})`)
    .call(d3.axisBottom(escalaLegenda).ticks(5));
});
```

---

### Transições e Animações com D3

D3 oferece um sistema de transições que interpola suavemente entre estados visuais, facilitando a criação de animações fluidas e informativas.

```javascript
// Transição básica
d3.select('#meu-circulo')
  .transition()
  .duration(1000)      // duração em milissegundos
  .delay(200)          // atraso antes de iniciar
  .ease(d3.easeCubicOut) // função de easing
  .attr('cx', 400)
  .attr('r', 30)
  .style('fill', '#e63946');
```

**Funções de Easing:**

| Função | Descrição |
|---|---|
| `d3.easeLinear` | Velocidade constante |
| `d3.easeCubicOut` | Desacelera no final (mais natural) |
| `d3.easeCubicInOut` | Acelera e desacelera |
| `d3.easeBounceOut` | Efeito de "quicar" |
| `d3.easeElasticOut` | Efeito elástico |
| `d3.easeBackOut` | Ultrapassa e retorna |

**Transições encadeadas:**

```javascript
d3.select('#meu-retangulo')
  .transition()
  .duration(500)
  .attr('width', 200)
  .style('fill', '#457b9d')
  .transition()         // encadeia automaticamente após a anterior
  .duration(500)
  .attr('height', 100)
  .style('fill', '#2a9d8f')
  .transition()
  .duration(300)
  .attr('rx', 20)
  .style('opacity', 0.8);
```

**Exemplo completo — Gráfico de barras animado:**

```html
<div id="barras-animadas"></div>
<button id="btn-atualizar">Atualizar Dados</button>

<script>
const largura = 600, altura = 400;
const margem = { top: 20, right: 20, bottom: 40, left: 50 };
const w = largura - margem.left - margem.right;
const h = altura - margem.top - margem.bottom;

const svg = d3.select('#barras-animadas')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

const categorias = ['A', 'B', 'C', 'D', 'E', 'F'];

const x = d3.scaleBand()
  .domain(categorias)
  .range([0, w])
  .padding(0.2);

const y = d3.scaleLinear()
  .range([h, 0]);

// Eixo X fixo
svg.append('g')
  .attr('class', 'eixo-x')
  .attr('transform', `translate(0,${h})`)
  .call(d3.axisBottom(x));

// Eixo Y dinâmico
const eixoY = svg.append('g').attr('class', 'eixo-y');

function gerarDados() {
  return categorias.map(c => ({
    categoria: c,
    valor: Math.floor(Math.random() * 100) + 10
  }));
}

function atualizar(dados) {
  // Atualizar escala Y
  y.domain([0, d3.max(dados, d => d.valor)]).nice();

  // Animar eixo Y
  eixoY.transition().duration(750)
    .call(d3.axisLeft(y));

  // Atualizar barras com transição
  svg.selectAll('.barra')
    .data(dados, d => d.categoria)
    .join(
      enter => enter.append('rect')
        .attr('class', 'barra')
        .attr('x', d => x(d.categoria))
        .attr('width', x.bandwidth())
        .attr('y', h)           // começa na base
        .attr('height', 0)       // altura zero
        .attr('fill', '#457b9d')
        .attr('rx', 4)
        .call(enter => enter.transition().duration(750)
          .attr('y', d => y(d.valor))
          .attr('height', d => h - y(d.valor))),
      update => update
        .call(update => update.transition().duration(750)
          .attr('y', d => y(d.valor))
          .attr('height', d => h - y(d.valor))
          .attr('fill', '#2a9d8f')),
      exit => exit
        .call(exit => exit.transition().duration(300)
          .attr('y', h)
          .attr('height', 0)
          .remove())
    );

  // Labels animados
  svg.selectAll('.valor-label')
    .data(dados, d => d.categoria)
    .join(
      enter => enter.append('text')
        .attr('class', 'valor-label')
        .attr('x', d => x(d.categoria) + x.bandwidth() / 2)
        .attr('y', h)
        .attr('text-anchor', 'middle')
        .style('font-size', '12px')
        .call(enter => enter.transition().duration(750)
          .attr('y', d => y(d.valor) - 5)
          .text(d => d.valor)),
      update => update
        .call(update => update.transition().duration(750)
          .attr('y', d => y(d.valor) - 5)
          .text(d => d.valor))
    );
}

// Dados iniciais
atualizar(gerarDados());

// Atualizar ao clicar
d3.select('#btn-atualizar').on('click', () => {
  atualizar(gerarDados());
});
</script>
```

> **Dica:** Use `.delay((d, i) => i * 50)` para criar efeitos de transição escalonada (stagger), onde cada elemento anima com um pequeno atraso em relação ao anterior.

---

### Layouts

Layouts são algoritmos que transformam dados hierárquicos ou relacionais em coordenadas posicionáveis no espaço visual.

**Force Simulation — Grafo de forças:**

```javascript
// Dados do grafo
const nos = [
  { id: 'React', grupo: 1 },
  { id: 'Vue', grupo: 1 },
  { id: 'Angular', grupo: 1 },
  { id: 'Node.js', grupo: 2 },
  { id: 'Express', grupo: 2 },
  { id: 'Spring', grupo: 3 },
  { id: 'PostgreSQL', grupo: 4 },
  { id: 'MongoDB', grupo: 4 }
];

const links = [
  { source: 'React', target: 'Node.js' },
  { source: 'Vue', target: 'Node.js' },
  { source: 'Angular', target: 'Node.js' },
  { source: 'Node.js', target: 'Express' },
  { source: 'Express', target: 'PostgreSQL' },
  { source: 'Express', target: 'MongoDB' },
  { source: 'Spring', target: 'PostgreSQL' },
  { source: 'React', target: 'Spring' }
];

const largura = 600, altura = 400;
const cor = d3.scaleOrdinal(d3.schemeCategory10);

const svg = d3.select('#grafo')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura);

// Criar simulação de forças
const simulacao = d3.forceSimulation(nos)
  .force('link', d3.forceLink(links).id(d => d.id).distance(100))
  .force('charge', d3.forceManyBody().strength(-300))
  .force('center', d3.forceCenter(largura / 2, altura / 2))
  .force('collision', d3.forceCollide().radius(30));

// Desenhar links
const linkElements = svg.append('g')
  .selectAll('line')
  .data(links)
  .join('line')
  .attr('stroke', '#999')
  .attr('stroke-opacity', 0.6)
  .attr('stroke-width', 2);

// Desenhar nós
const noElements = svg.append('g')
  .selectAll('circle')
  .data(nos)
  .join('circle')
  .attr('r', 15)
  .attr('fill', d => cor(d.grupo))
  .attr('stroke', '#fff')
  .attr('stroke-width', 2)
  .call(d3.drag()
    .on('start', dragIniciado)
    .on('drag', arrastando)
    .on('end', dragFinalizado));

// Labels dos nós
const labels = svg.append('g')
  .selectAll('text')
  .data(nos)
  .join('text')
  .text(d => d.id)
  .style('font-size', '11px')
  .attr('dx', 18)
  .attr('dy', 4);

// Atualizar posições a cada tick da simulação
simulacao.on('tick', () => {
  linkElements
    .attr('x1', d => d.source.x)
    .attr('y1', d => d.source.y)
    .attr('x2', d => d.target.x)
    .attr('y2', d => d.target.y);

  noElements
    .attr('cx', d => d.x)
    .attr('cy', d => d.y);

  labels
    .attr('x', d => d.x)
    .attr('y', d => d.y);
});

// Funções de drag
function dragIniciado(event, d) {
  if (!event.active) simulacao.alphaTarget(0.3).restart();
  d.fx = d.x;
  d.fy = d.y;
}

function arrastando(event, d) {
  d.fx = event.x;
  d.fy = event.y;
}

function dragFinalizado(event, d) {
  if (!event.active) simulacao.alphaTarget(0);
  d.fx = null;
  d.fy = null;
}
```

**Tree Layout — Árvore hierárquica:**

```javascript
const dadosHierarquia = {
  name: 'CEO',
  children: [
    {
      name: 'CTO',
      children: [
        { name: 'Dev Lead', children: [
          { name: 'Dev Senior' },
          { name: 'Dev Pleno' }
        ]},
        { name: 'QA Lead', children: [
          { name: 'QA Analyst' }
        ]}
      ]
    },
    {
      name: 'CFO',
      children: [
        { name: 'Controller' },
        { name: 'Tesoureiro' }
      ]
    },
    {
      name: 'COO',
      children: [
        { name: 'RH', children: [
          { name: 'Recrutamento' },
          { name: 'Treinamento' }
        ]},
        { name: 'Operações' }
      ]
    }
  ]
};

const largura = 800, altura = 500;
const margem = { top: 20, right: 100, bottom: 20, left: 100 };

const svg = d3.select('#arvore')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura)
  .append('g')
  .attr('transform', `translate(${margem.left},${margem.top})`);

// Criar hierarquia D3
const raiz = d3.hierarchy(dadosHierarquia);

// Layout de árvore
const treeLayout = d3.tree()
  .size([altura - margem.top - margem.bottom, largura - margem.left - margem.right]);

treeLayout(raiz);

// Desenhar links (curvas)
svg.selectAll('.link')
  .data(raiz.links())
  .join('path')
  .attr('class', 'link')
  .attr('fill', 'none')
  .attr('stroke', '#ccc')
  .attr('stroke-width', 1.5)
  .attr('d', d3.linkHorizontal()
    .x(d => d.y)
    .y(d => d.x));

// Desenhar nós
const no = svg.selectAll('.no')
  .data(raiz.descendants())
  .join('g')
  .attr('class', 'no')
  .attr('transform', d => `translate(${d.y},${d.x})`);

no.append('circle')
  .attr('r', 6)
  .attr('fill', d => d.children ? '#457b9d' : '#2a9d8f');

no.append('text')
  .attr('dx', d => d.children ? -10 : 10)
  .attr('dy', 4)
  .attr('text-anchor', d => d.children ? 'end' : 'start')
  .style('font-size', '12px')
  .text(d => d.data.name);
```

**Treemap:**

```javascript
const dadosTreemap = {
  name: 'Vendas',
  children: [
    { name: 'Eletrônicos', children: [
      { name: 'Smartphones', valor: 450 },
      { name: 'Laptops', valor: 320 },
      { name: 'Tablets', valor: 180 },
      { name: 'Acessórios', valor: 90 }
    ]},
    { name: 'Vestuário', children: [
      { name: 'Camisetas', valor: 200 },
      { name: 'Calças', valor: 150 },
      { name: 'Calçados', valor: 280 }
    ]},
    { name: 'Alimentos', children: [
      { name: 'Bebidas', valor: 120 },
      { name: 'Snacks', valor: 80 },
      { name: 'Orgânicos', valor: 60 }
    ]}
  ]
};

const largura = 700, altura = 450;

const svg = d3.select('#treemap')
  .append('svg')
  .attr('width', largura)
  .attr('height', altura);

const raiz = d3.hierarchy(dadosTreemap)
  .sum(d => d.valor)
  .sort((a, b) => b.value - a.value);

const treemapLayout = d3.treemap()
  .size([largura, altura])
  .padding(2)
  .round(true);

treemapLayout(raiz);

const cor = d3.scaleOrdinal()
  .domain(['Eletrônicos', 'Vestuário', 'Alimentos'])
  .range(['#457b9d', '#e63946', '#2a9d8f']);

// Desenhar retângulos
const celulas = svg.selectAll('g')
  .data(raiz.leaves())
  .join('g')
  .attr('transform', d => `translate(${d.x0},${d.y0})`);

celulas.append('rect')
  .attr('width', d => d.x1 - d.x0)
  .attr('height', d => d.y1 - d.y0)
  .attr('fill', d => cor(d.parent.data.name))
  .attr('opacity', 0.85)
  .attr('stroke', '#fff');

celulas.append('text')
  .attr('x', 5)
  .attr('y', 18)
  .style('font-size', '11px')
  .style('fill', '#fff')
  .style('font-weight', 'bold')
  .text(d => d.data.name);

celulas.append('text')
  .attr('x', 5)
  .attr('y', 34)
  .style('font-size', '10px')
  .style('fill', '#ffffffcc')
  .text(d => `R$ ${d.data.valor}`);
```

---

### D3 + React/Angular/Vue

Integrar D3 com frameworks modernos requer decidir quem controla o DOM. Existem duas estratégias principais:

| Estratégia | Quem controla o DOM | Vantagens | Desvantagens |
|---|---|---|---|
| **D3 para math, framework para DOM** | Framework (React/Vue/Angular) | Melhor integração, server-side rendering, acessibilidade | Mais código manual |
| **D3 controla um `ref`** | D3 | Acesso completo às features do D3 (transições, drag) | Conflito potencial com virtual DOM |

> **Recomendação:** Prefira a primeira estratégia (D3 para cálculos, framework para renderização) para a maioria dos casos. Use a segunda quando precisar de transições complexas ou layouts de força.

**React — D3 para cálculos, React para DOM (recomendado):**

```typescript
import { useMemo } from 'react';
import * as d3 from 'd3';

interface DadoBarra {
  label: string;
  valor: number;
}

interface BarChartProps {
  dados: DadoBarra[];
  largura?: number;
  altura?: number;
}

function BarChart({ dados, largura = 600, altura = 400 }: BarChartProps) {
  const margem = { top: 20, right: 20, bottom: 40, left: 50 };
  const w = largura - margem.left - margem.right;
  const h = altura - margem.top - margem.bottom;

  // D3 apenas para cálculos — sem manipulação de DOM
  const escalas = useMemo(() => {
    const x = d3.scaleBand()
      .domain(dados.map(d => d.label))
      .range([0, w])
      .padding(0.2);

    const y = d3.scaleLinear()
      .domain([0, d3.max(dados, d => d.valor) ?? 0])
      .range([h, 0])
      .nice();

    return { x, y };
  }, [dados, w, h]);

  // React renderiza o SVG
  return (
    <svg width={largura} height={altura}>
      <g transform={`translate(${margem.left},${margem.top})`}>
        {/* Barras */}
        {dados.map(d => (
          <rect
            key={d.label}
            x={escalas.x(d.label)}
            y={escalas.y(d.valor)}
            width={escalas.x.bandwidth()}
            height={h - escalas.y(d.valor)}
            fill="#457b9d"
            rx={4}
          />
        ))}

        {/* Eixo X */}
        <g transform={`translate(0,${h})`}>
          {escalas.x.domain().map(label => (
            <text
              key={label}
              x={(escalas.x(label) ?? 0) + escalas.x.bandwidth() / 2}
              y={25}
              textAnchor="middle"
              fontSize={12}
            >
              {label}
            </text>
          ))}
        </g>

        {/* Eixo Y */}
        {escalas.y.ticks(5).map(tick => (
          <g key={tick} transform={`translate(0,${escalas.y(tick)})`}>
            <line x2={w} stroke="#e0e0e0" />
            <text x={-10} dy="0.32em" textAnchor="end" fontSize={11}>
              {tick}
            </text>
          </g>
        ))}
      </g>
    </svg>
  );
}

// Uso
function App() {
  const dados = [
    { label: 'Jan', valor: 45 },
    { label: 'Fev', valor: 72 },
    { label: 'Mar', valor: 58 },
    { label: 'Abr', valor: 90 }
  ];

  return <BarChart dados={dados} largura={600} altura={400} />;
}
```

**React — D3 controlando um ref (para transições e interações complexas):**

```typescript
import { useRef, useEffect } from 'react';
import * as d3 from 'd3';

interface ForceGraphProps {
  nos: { id: string; grupo: number }[];
  links: { source: string; target: string }[];
  largura?: number;
  altura?: number;
}

function ForceGraph({ nos, links, largura = 600, altura = 400 }: ForceGraphProps) {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    if (!svgRef.current) return;

    const svg = d3.select(svgRef.current);
    svg.selectAll('*').remove(); // limpar renderização anterior

    const cor = d3.scaleOrdinal(d3.schemeCategory10);

    const simulacao = d3.forceSimulation(nos as any)
      .force('link', d3.forceLink(links).id((d: any) => d.id))
      .force('charge', d3.forceManyBody().strength(-200))
      .force('center', d3.forceCenter(largura / 2, altura / 2));

    const linkEl = svg.append('g')
      .selectAll('line')
      .data(links)
      .join('line')
      .attr('stroke', '#999')
      .attr('stroke-width', 2);

    const noEl = svg.append('g')
      .selectAll('circle')
      .data(nos)
      .join('circle')
      .attr('r', 12)
      .attr('fill', (d: any) => cor(d.grupo));

    simulacao.on('tick', () => {
      linkEl
        .attr('x1', (d: any) => d.source.x)
        .attr('y1', (d: any) => d.source.y)
        .attr('x2', (d: any) => d.target.x)
        .attr('y2', (d: any) => d.target.y);

      noEl
        .attr('cx', (d: any) => d.x)
        .attr('cy', (d: any) => d.y);
    });

    return () => { simulacao.stop(); };
  }, [nos, links, largura, altura]);

  return <svg ref={svgRef} width={largura} height={altura} />;
}
```

**Vue — Composable com D3:**

```html
<!-- BarChart.vue -->
<template>
  <svg :width="largura" :height="altura">
    <g :transform="`translate(${margem.left},${margem.top})`">
      <rect
        v-for="d in dados"
        :key="d.label"
        :x="escalaX(d.label)"
        :y="escalaY(d.valor)"
        :width="escalaX.bandwidth()"
        :height="alturaInterna - escalaY(d.valor)"
        fill="#457b9d"
        rx="4"
      />
      <!-- Eixo X -->
      <g :transform="`translate(0,${alturaInterna})`">
        <text
          v-for="label in escalaX.domain()"
          :key="label"
          :x="escalaX(label) + escalaX.bandwidth() / 2"
          y="25"
          text-anchor="middle"
          font-size="12"
        >
          {{ label }}
        </text>
      </g>
    </g>
  </svg>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import * as d3 from 'd3';

interface Props {
  dados: { label: string; valor: number }[];
  largura?: number;
  altura?: number;
}

const props = withDefaults(defineProps<Props>(), {
  largura: 600,
  altura: 400
});

const margem = { top: 20, right: 20, bottom: 40, left: 50 };
const larguraInterna = computed(() => props.largura - margem.left - margem.right);
const alturaInterna = computed(() => props.altura - margem.top - margem.bottom);

const escalaX = computed(() =>
  d3.scaleBand()
    .domain(props.dados.map(d => d.label))
    .range([0, larguraInterna.value])
    .padding(0.2)
);

const escalaY = computed(() =>
  d3.scaleLinear()
    .domain([0, d3.max(props.dados, d => d.valor) ?? 0])
    .range([alturaInterna.value, 0])
    .nice()
);
</script>
```

**Angular — Diretiva com D3:**

```typescript
// bar-chart.component.ts
import { Component, Input, ElementRef, OnChanges, ViewChild, AfterViewInit } from '@angular/core';
import * as d3 from 'd3';

interface DadoBarra {
  label: string;
  valor: number;
}

@Component({
  selector: 'app-bar-chart',
  standalone: true,
  template: '<svg #chart></svg>'
})
export class BarChartComponent implements OnChanges, AfterViewInit {
  @Input() dados: DadoBarra[] = [];
  @Input() largura = 600;
  @Input() altura = 400;
  @ViewChild('chart', { static: true }) chartRef!: ElementRef;

  private margem = { top: 20, right: 20, bottom: 40, left: 50 };

  ngAfterViewInit() {
    this.desenhar();
  }

  ngOnChanges() {
    this.desenhar();
  }

  private desenhar() {
    const svg = d3.select(this.chartRef.nativeElement)
      .attr('width', this.largura)
      .attr('height', this.altura);

    svg.selectAll('*').remove();

    const w = this.largura - this.margem.left - this.margem.right;
    const h = this.altura - this.margem.top - this.margem.bottom;

    const g = svg.append('g')
      .attr('transform', `translate(${this.margem.left},${this.margem.top})`);

    const x = d3.scaleBand()
      .domain(this.dados.map(d => d.label))
      .range([0, w])
      .padding(0.2);

    const y = d3.scaleLinear()
      .domain([0, d3.max(this.dados, d => d.valor) ?? 0])
      .range([h, 0])
      .nice();

    g.append('g')
      .attr('transform', `translate(0,${h})`)
      .call(d3.axisBottom(x));

    g.append('g')
      .call(d3.axisLeft(y));

    g.selectAll('rect')
      .data(this.dados)
      .join('rect')
      .attr('x', d => x(d.label)!)
      .attr('y', d => y(d.valor))
      .attr('width', x.bandwidth())
      .attr('height', d => h - y(d.valor))
      .attr('fill', '#457b9d')
      .attr('rx', 4);
  }
}
```

---

### Exemplo Prático: Dashboard Interativo

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard D3.js</title>
  <script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
  <style>
    .dashboard { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; max-width: 1200px; margin: 0 auto; }
    .card { background: #fff; border-radius: 8px; padding: 16px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
    .tooltip { position: absolute; background: rgba(0,0,0,0.8); color: #fff; padding: 8px 12px;
               border-radius: 4px; font-size: 12px; pointer-events: none; opacity: 0; transition: opacity 0.2s; }
    body { font-family: system-ui, sans-serif; background: #f5f5f5; padding: 20px; }
  </style>
</head>
<body>
  <div class="dashboard">
    <div class="card"><h3>Vendas Mensais</h3><div id="vendas"></div></div>
    <div class="card"><h3>Distribuicao por Categoria</h3><div id="categorias"></div></div>
  </div>
  <div class="tooltip" id="tooltip"></div>

  <script>
  const vendas = [
    { mes: 'Jan', valor: 42 }, { mes: 'Fev', valor: 58 }, { mes: 'Mar', valor: 65 },
    { mes: 'Abr', valor: 48 }, { mes: 'Mai', valor: 82 }, { mes: 'Jun', valor: 71 }
  ];
  const cats = [
    { nome: 'Eletronicos', valor: 35 }, { nome: 'Roupas', valor: 28 },
    { nome: 'Alimentos', valor: 22 }, { nome: 'Outros', valor: 15 }
  ];

  const tooltip = d3.select('#tooltip');
  function mostrarTooltip(event, texto) {
    tooltip.style('opacity', 1).html(texto)
      .style('left', (event.pageX + 10) + 'px').style('top', (event.pageY - 20) + 'px');
  }
  function esconderTooltip() { tooltip.style('opacity', 0); }

  // --- Grafico de Barras Responsivo ---
  const containerVendas = document.getElementById('vendas');
  const larg = containerVendas.clientWidth, alt = 250;
  const m = { top: 10, right: 10, bottom: 30, left: 40 };
  const svgV = d3.select('#vendas').append('svg').attr('viewBox', `0 0 ${larg} ${alt}`);
  const gV = svgV.append('g').attr('transform', `translate(${m.left},${m.top})`);
  const xV = d3.scaleBand().domain(vendas.map(d => d.mes)).range([0, larg-m.left-m.right]).padding(0.25);
  const yV = d3.scaleLinear().domain([0, d3.max(vendas, d => d.valor)]).range([alt-m.top-m.bottom, 0]).nice();
  gV.append('g').attr('transform', `translate(0,${alt-m.top-m.bottom})`).call(d3.axisBottom(xV));
  gV.append('g').call(d3.axisLeft(yV).ticks(5));
  gV.selectAll('rect').data(vendas).join('rect')
    .attr('x', d => xV(d.mes)).attr('width', xV.bandwidth()).attr('rx', 4)
    .attr('y', alt-m.top-m.bottom).attr('height', 0).attr('fill', '#457b9d')
    .on('mouseover', (e, d) => mostrarTooltip(e, `${d.mes}: R$ ${d.valor}k`))
    .on('mouseout', esconderTooltip)
    .transition().duration(800).delay((d, i) => i * 100)
    .attr('y', d => yV(d.valor)).attr('height', d => (alt-m.top-m.bottom) - yV(d.valor));

  // --- Grafico de Donut ---
  const raio = 100;
  const svgD = d3.select('#categorias').append('svg').attr('viewBox', `0 0 ${larg} ${alt}`)
    .append('g').attr('transform', `translate(${larg/2},${alt/2})`);
  const cor = d3.scaleOrdinal(d3.schemeTableau10);
  const arco = d3.arc().innerRadius(raio * 0.5).outerRadius(raio);
  svgD.selectAll('path').data(d3.pie().value(d => d.valor)(cats)).join('path')
    .attr('d', arco).attr('fill', d => cor(d.data.nome)).attr('stroke', '#fff').attr('stroke-width', 2)
    .on('mouseover', (e, d) => mostrarTooltip(e, `${d.data.nome}: ${d.data.valor}%`))
    .on('mouseout', esconderTooltip);
  svgD.selectAll('text').data(d3.pie().value(d => d.valor)(cats)).join('text')
    .attr('transform', d => `translate(${d3.arc().innerRadius(raio*0.7).outerRadius(raio*0.7).centroid(d)})`)
    .attr('text-anchor', 'middle').style('font-size', '11px').text(d => d.data.nome);
  </script>
</body>
</html>
```

---

## 5. Chart.js

Chart.js é uma biblioteca de gráficos JavaScript simples e flexível, ideal para criar visualizações de dados bonitas e responsivas com mínima configuração. Utiliza o elemento `<canvas>` do HTML5 para renderização.

> Referência: [Chart.js Docs](https://www.chartjs.org/docs/) · [Chart.js Samples](https://www.chartjs.org/docs/latest/samples/)

---

### Instalação e Configuração

```bash
# Instalação via npm
npm install chart.js

# Para plugins oficiais
npm install chartjs-plugin-zoom chartjs-plugin-annotation chartjs-plugin-datalabels
```

```html
<!-- Via CDN -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
```

**Auto-registration vs Tree-shaking (Chart.js 4):**

No Chart.js 4, os componentes podem ser registrados automaticamente ou seletivamente para reduzir o bundle size:

```javascript
// Auto-registration — registra todos os componentes (~60 KB gzipped)
import Chart from 'chart.js/auto';

// Tree-shaking — registrar apenas o necessário (~16-30 KB gzipped)
import {
  Chart,
  BarController,
  LineController,
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  Tooltip,
  Legend
} from 'chart.js';

Chart.register(
  BarController,
  LineController,
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  Tooltip,
  Legend
);
```

**Setup básico:**

```html
<canvas id="meu-grafico" width="600" height="400"></canvas>

<script>
import Chart from 'chart.js/auto';

const ctx = document.getElementById('meu-grafico').getContext('2d');

const meuGrafico = new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
    datasets: [{
      label: 'Vendas (R$ mil)',
      data: [45, 72, 58, 90, 63, 85],
      backgroundColor: '#457b9d',
      borderRadius: 6
    }]
  },
  options: {
    responsive: true,
    plugins: {
      legend: { position: 'top' },
      title: {
        display: true,
        text: 'Vendas Mensais 2024'
      }
    }
  }
});
</script>
```

---

### Tipos de Gráficos

Chart.js oferece 8 tipos de gráficos nativos, todos altamente configuráveis:

| Tipo | Descrição | Uso Típico |
|---|---|---|
| `bar` | Barras verticais ou horizontais | Comparações entre categorias |
| `line` | Linhas com pontos | Tendências ao longo do tempo |
| `pie` | Pizza | Proporções de um todo |
| `doughnut` | Donut (pizza com furo) | Proporções com visual moderno |
| `radar` | Radar/Aranha | Comparação multidimensional |
| `polarArea` | Área polar | Similar a pizza com raios variáveis |
| `bubble` | Bolhas | Três dimensões de dados |
| `scatter` | Dispersão | Correlação entre variáveis |

**Gráfico de Linhas:**

```javascript
const graficoLinha = new Chart(document.getElementById('linha'), {
  type: 'line',
  data: {
    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago'],
    datasets: [
      {
        label: 'Receita',
        data: [30, 45, 38, 62, 55, 78, 72, 90],
        borderColor: '#e63946',
        backgroundColor: '#e6394620',
        fill: true,               // preencher área sob a linha
        tension: 0.3,             // suavização da curva (0 = reta)
        pointRadius: 4,
        pointHoverRadius: 7
      },
      {
        label: 'Despesas',
        data: [25, 38, 42, 50, 48, 60, 65, 70],
        borderColor: '#457b9d',
        backgroundColor: '#457b9d20',
        fill: true,
        tension: 0.3,
        borderDash: [5, 5]       // linha tracejada
      }
    ]
  },
  options: {
    responsive: true,
    interaction: {
      mode: 'index',             // tooltip mostra todos os datasets no mesmo X
      intersect: false
    },
    plugins: {
      legend: { position: 'top' }
    },
    scales: {
      y: {
        beginAtZero: true,
        ticks: {
          callback: value => `R$ ${value}k`
        }
      }
    }
  }
});
```

**Gráfico Radar:**

```javascript
const graficoRadar = new Chart(document.getElementById('radar'), {
  type: 'radar',
  data: {
    labels: ['JavaScript', 'TypeScript', 'React', 'Node.js', 'SQL', 'Docker'],
    datasets: [
      {
        label: 'Dev Senior',
        data: [90, 85, 88, 80, 75, 70],
        borderColor: '#e63946',
        backgroundColor: '#e6394630',
        pointBackgroundColor: '#e63946'
      },
      {
        label: 'Dev Junior',
        data: [60, 40, 55, 35, 45, 20],
        borderColor: '#457b9d',
        backgroundColor: '#457b9d30',
        pointBackgroundColor: '#457b9d'
      }
    ]
  },
  options: {
    responsive: true,
    scales: {
      r: {
        beginAtZero: true,
        max: 100,
        ticks: { stepSize: 20 }
      }
    }
  }
});
```

**Gráfico de Pizza/Donut:**

```javascript
const graficoPizza = new Chart(document.getElementById('pizza'), {
  type: 'doughnut', // trocar para 'pie' para pizza completa
  data: {
    labels: ['Frontend', 'Backend', 'DevOps', 'Mobile', 'Data Science'],
    datasets: [{
      data: [35, 30, 15, 12, 8],
      backgroundColor: [
        '#e63946', '#457b9d', '#2a9d8f', '#e9c46a', '#264653'
      ],
      borderWidth: 2,
      borderColor: '#fff',
      hoverOffset: 8
    }]
  },
  options: {
    responsive: true,
    cutout: '60%', // tamanho do furo (apenas doughnut)
    plugins: {
      legend: {
        position: 'bottom',
        labels: {
          usePointStyle: true,  // ícone circular na legenda
          padding: 16
        }
      }
    }
  }
});
```

**Gráfico de Bolhas (Bubble):**

```javascript
const graficoBolhas = new Chart(document.getElementById('bolhas'), {
  type: 'bubble',
  data: {
    datasets: [
      {
        label: 'Frontend',
        data: [
          { x: 3, y: 7000, r: 12 },  // experiência, salário, tamanho
          { x: 5, y: 9500, r: 15 },
          { x: 2, y: 5000, r: 8 }
        ],
        backgroundColor: '#e6394680'
      },
      {
        label: 'Backend',
        data: [
          { x: 4, y: 8500, r: 14 },
          { x: 7, y: 13000, r: 18 },
          { x: 1, y: 4000, r: 6 }
        ],
        backgroundColor: '#457b9d80'
      }
    ]
  },
  options: {
    responsive: true,
    scales: {
      x: {
        title: { display: true, text: 'Experiência (anos)' }
      },
      y: {
        title: { display: true, text: 'Salário (R$)' },
        ticks: { callback: v => `R$ ${v / 1000}k` }
      }
    }
  }
});
```

**Gráfico Misto (Mixed Chart):**

```javascript
const graficoMisto = new Chart(document.getElementById('misto'), {
  type: 'bar', // tipo base
  data: {
    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
    datasets: [
      {
        type: 'bar',
        label: 'Vendas',
        data: [45, 72, 58, 90, 63, 85],
        backgroundColor: '#457b9d',
        borderRadius: 4,
        order: 2 // renderizado atrás
      },
      {
        type: 'line',
        label: 'Meta',
        data: [60, 60, 65, 70, 70, 75],
        borderColor: '#e63946',
        borderWidth: 2,
        borderDash: [6, 4],
        pointRadius: 0,
        fill: false,
        order: 1 // renderizado na frente
      }
    ]
  },
  options: {
    responsive: true,
    plugins: {
      legend: { position: 'top' }
    },
    scales: {
      y: { beginAtZero: true }
    }
  }
});
```

---

### Customização

**Cores, Fontes e Tooltips:**

```javascript
// Configuração global de fontes
Chart.defaults.font.family = "'Inter', 'Segoe UI', sans-serif";
Chart.defaults.font.size = 13;
Chart.defaults.color = '#333';

// Gráfico com tema customizado
const graficoCustom = new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Q1', 'Q2', 'Q3', 'Q4'],
    datasets: [{
      label: 'Receita',
      data: [120, 190, 170, 240],
      backgroundColor: (context) => {
        // Gradiente por barra
        const chart = context.chart;
        const { ctx: canvasCtx, chartArea } = chart;
        if (!chartArea) return '#457b9d';

        const gradient = canvasCtx.createLinearGradient(0, chartArea.bottom, 0, chartArea.top);
        gradient.addColorStop(0, '#457b9d');
        gradient.addColorStop(1, '#2a9d8f');
        return gradient;
      },
      borderRadius: 8,
      borderSkipped: false // arredondar todas as bordas
    }]
  },
  options: {
    responsive: true,
    plugins: {
      // Tooltip customizado
      tooltip: {
        backgroundColor: 'rgba(0, 0, 0, 0.85)',
        titleFont: { size: 14, weight: 'bold' },
        bodyFont: { size: 13 },
        padding: 12,
        cornerRadius: 8,
        displayColors: false,
        callbacks: {
          title: (items) => `Trimestre: ${items[0].label}`,
          label: (item) => `Receita: R$ ${item.parsed.y}k`,
          afterLabel: (item) => {
            const anterior = item.dataIndex > 0
              ? item.dataset.data[item.dataIndex - 1]
              : null;
            if (anterior) {
              const diff = ((item.parsed.y - anterior) / anterior * 100).toFixed(1);
              return `Variação: ${diff > 0 ? '+' : ''}${diff}%`;
            }
            return '';
          }
        }
      },
      // Legenda customizada
      legend: {
        display: true,
        position: 'top',
        labels: {
          usePointStyle: true,
          pointStyle: 'rectRounded',
          padding: 20,
          font: { size: 13 }
        }
      }
    },
    scales: {
      y: {
        beginAtZero: true,
        grid: {
          color: '#f0f0f0',
          drawBorder: false
        },
        ticks: {
          callback: value => `R$ ${value}k`,
          padding: 8
        }
      },
      x: {
        grid: { display: false },
        ticks: { padding: 8 }
      }
    }
  }
});
```

---

### Responsividade e Animações

```javascript
const graficoResponsivo = new Chart(ctx, {
  type: 'line',
  data: { /* ... */ },
  options: {
    // Responsividade
    responsive: true,          // redimensiona com o container
    maintainAspectRatio: true, // mantém proporção largura/altura
    aspectRatio: 2,            // proporção 2:1

    // Animações
    animation: {
      duration: 1500,          // duração em ms
      easing: 'easeOutQuart',  // função de easing
      delay: (context) => {
        // Animação escalonada por ponto
        return context.dataIndex * 100;
      }
    },

    // Animações por propriedade
    animations: {
      y: {
        from: (ctx) => {
          if (ctx.type === 'data') {
            // Barras crescem de baixo
            return ctx.chart.scales.y.getPixelForValue(0);
          }
        }
      },
      opacity: {
        from: 0,
        to: 1,
        duration: 800
      }
    },

    // Transições para atualizações
    transitions: {
      active: {
        animation: { duration: 300 }
      }
    }
  }
});

// Atualizar dados com animação
function atualizarDados(novosDados) {
  graficoResponsivo.data.datasets[0].data = novosDados;
  graficoResponsivo.update('active'); // 'active', 'resize', 'none'
}
```

> **Dica:** Encapsule o `<canvas>` em uma `<div>` com largura definida. Chart.js usa as dimensões do container pai quando `responsive: true`.

---

### Plugins

**Plugins Oficiais:**

```bash
npm install chartjs-plugin-zoom chartjs-plugin-annotation chartjs-plugin-datalabels
```

```javascript
// Plugin de Zoom (zoom e pan)
import zoomPlugin from 'chartjs-plugin-zoom';
Chart.register(zoomPlugin);

const graficoComZoom = new Chart(ctx, {
  type: 'line',
  data: { /* ... */ },
  options: {
    plugins: {
      zoom: {
        zoom: {
          wheel: { enabled: true },    // zoom com scroll do mouse
          pinch: { enabled: true },    // zoom com pinch em touch
          mode: 'x'                    // zoom apenas no eixo X
        },
        pan: {
          enabled: true,
          mode: 'x'
        },
        limits: {
          x: { min: 0, max: 100 }
        }
      }
    }
  }
});

// Plugin de Anotações (linhas, caixas, labels sobre o gráfico)
import annotationPlugin from 'chartjs-plugin-annotation';
Chart.register(annotationPlugin);

const graficoComAnotacoes = new Chart(ctx, {
  type: 'line',
  data: { /* ... */ },
  options: {
    plugins: {
      annotation: {
        annotations: {
          linhaMeta: {
            type: 'line',
            yMin: 75,
            yMax: 75,
            borderColor: '#e63946',
            borderWidth: 2,
            borderDash: [6, 4],
            label: {
              display: true,
              content: 'Meta: 75',
              position: 'end'
            }
          },
          faixaAlerta: {
            type: 'box',
            yMin: 20,
            yMax: 40,
            backgroundColor: '#e9c46a30',
            borderWidth: 0,
            label: {
              display: true,
              content: 'Zona de atenção'
            }
          }
        }
      }
    }
  }
});

// Plugin de Data Labels (valores sobre barras/pontos)
import ChartDataLabels from 'chartjs-plugin-datalabels';
Chart.register(ChartDataLabels);

const graficoComLabels = new Chart(ctx, {
  type: 'bar',
  data: { /* ... */ },
  options: {
    plugins: {
      datalabels: {
        anchor: 'end',
        align: 'top',
        font: { weight: 'bold', size: 12 },
        color: '#333',
        formatter: (value) => `R$ ${value}k`
      }
    }
  }
});
```

**Criando um Plugin Customizado:**

```javascript
// Plugin que desenha uma marca d'água no centro do gráfico
const pluginMarcaDagua = {
  id: 'marcaDagua',

  // Hook: após desenhar o gráfico
  afterDraw(chart, args, options) {
    const { ctx: canvasCtx, chartArea } = chart;
    if (!chartArea) return;

    canvasCtx.save();
    canvasCtx.globalAlpha = options.opacity || 0.1;
    canvasCtx.font = `${options.fontSize || 48}px ${options.fontFamily || 'Arial'}`;
    canvasCtx.fillStyle = options.color || '#000';
    canvasCtx.textAlign = 'center';
    canvasCtx.textBaseline = 'middle';

    const centerX = (chartArea.left + chartArea.right) / 2;
    const centerY = (chartArea.top + chartArea.bottom) / 2;
    canvasCtx.fillText(options.text || '', centerX, centerY);

    canvasCtx.restore();
  },

  // Valores padrão
  defaults: {
    text: 'RASCUNHO',
    opacity: 0.08,
    fontSize: 56,
    color: '#888'
  }
};

// Registrar e usar
Chart.register(pluginMarcaDagua);

const grafico = new Chart(ctx, {
  type: 'bar',
  data: { /* ... */ },
  options: {
    plugins: {
      marcaDagua: {
        text: 'CONFIDENCIAL',
        opacity: 0.05,
        fontSize: 40
      }
    }
  }
});
```

---

### Chart.js vs D3.js

| Critério | Chart.js | D3.js |
|---|---|---|
| **Tipo** | Biblioteca de gráficos prontos | Toolkit de visualização de dados |
| **Curva de aprendizado** | Baixa — funciona em minutos | Alta — requer entender SVG, scales, joins |
| **Flexibilidade** | Média — personalizável via options | Total — controle pixel a pixel |
| **Tipos de gráfico** | 8 tipos nativos + plugins | Ilimitado — você constrói qualquer visualização |
| **Renderização** | Canvas (rasterizado) | SVG ou Canvas (vetorial ou rasterizado) |
| **Bundle size** | ~60 KB (gzip, auto) / ~20 KB (tree-shaking) | ~30 KB (gzip, modular) |
| **Responsividade** | Nativa (`responsive: true`) | Manual (viewBox, resize listeners) |
| **Animações** | Configuráveis via options | Transições programáticas completas |
| **Interatividade** | Tooltips e hover nativos | Ilimitada (drag, zoom, brush, force) |
| **Acessibilidade** | Básica (alt text no canvas) | Completa (SVG permite ARIA e navegação) |
| **Server-side** | Possível com node-canvas | Possível com jsdom |
| **Uso ideal** | Dashboards corporativos, relatórios | Visualizações customizadas, mapas, infográficos |

> **Quando usar cada um:** Use **Chart.js** quando precisa de gráficos padrão com pouca configuração. Use **D3.js** quando precisa de visualizações únicas, mapas geográficos ou controle total sobre cada detalhe visual.

---

### Integração com Frameworks

**React — react-chartjs-2:**

```bash
npm install react-chartjs-2 chart.js
```

```typescript
import { Bar, Line, Doughnut } from 'react-chartjs-2';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  ArcElement,
  Title,
  Tooltip,
  Legend,
  Filler
} from 'chart.js';

// Registrar componentes necessários
ChartJS.register(
  CategoryScale, LinearScale, BarElement, LineElement,
  PointElement, ArcElement, Title, Tooltip, Legend, Filler
);

interface VendasChartProps {
  dados: number[];
  labels: string[];
}

function VendasChart({ dados, labels }: VendasChartProps) {
  const chartData = {
    labels,
    datasets: [{
      label: 'Vendas (R$ mil)',
      data: dados,
      backgroundColor: '#457b9d',
      borderRadius: 6
    }]
  };

  const options = {
    responsive: true,
    plugins: {
      legend: { position: 'top' as const },
      title: {
        display: true,
        text: 'Vendas Mensais'
      }
    },
    scales: {
      y: {
        beginAtZero: true,
        ticks: {
          callback: (value: number) => `R$ ${value}k`
        }
      }
    }
  };

  return <Bar data={chartData} options={options} />;
}

// Múltiplos gráficos em um dashboard
function Dashboard() {
  const labels = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'];
  const vendas = [45, 72, 58, 90, 63, 85];

  return (
    <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '20px' }}>
      <div>
        <VendasChart dados={vendas} labels={labels} />
      </div>
      <div>
        <Doughnut data={{
          labels: ['Frontend', 'Backend', 'DevOps'],
          datasets: [{
            data: [45, 35, 20],
            backgroundColor: ['#e63946', '#457b9d', '#2a9d8f']
          }]
        }} />
      </div>
    </div>
  );
}
```

**Vue — vue-chartjs:**

```bash
npm install vue-chartjs chart.js
```

```html
<!-- BarChart.vue -->
<template>
  <Bar :data="chartData" :options="chartOptions" />
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { Bar } from 'vue-chartjs';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js';

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

interface Props {
  dados: number[];
  labels: string[];
  titulo?: string;
}

const props = withDefaults(defineProps<Props>(), {
  titulo: 'Vendas Mensais'
});

const chartData = computed(() => ({
  labels: props.labels,
  datasets: [{
    label: props.titulo,
    data: props.dados,
    backgroundColor: '#457b9d',
    borderRadius: 6
  }]
}));

const chartOptions = {
  responsive: true,
  plugins: {
    legend: { position: 'top' as const },
    title: {
      display: true,
      text: props.titulo
    }
  },
  scales: {
    y: { beginAtZero: true }
  }
};
</script>
```

```html
<!-- Uso no componente pai -->
<template>
  <div class="dashboard">
    <BarChart
      :dados="[45, 72, 58, 90, 63, 85]"
      :labels="['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun']"
      titulo="Vendas 2024"
    />
  </div>
</template>
```

> **Angular:** Utilize a biblioteca [ng2-charts](https://valor-software.com/ng2-charts/) que encapsula Chart.js em componentes Angular com suporte a tipos de gráfico via diretiva `baseChart`. A instalação é feita com `npm install ng2-charts chart.js`.

---

## 6. Apache ECharts

Apache ECharts é uma biblioteca de visualização de dados open-source incubada pela Apache Software Foundation, originalmente criada pela Baidu. Destaca-se pela enorme variedade de tipos de gráficos, renderização de alta performance (Canvas e SVG), temas customizáveis, suporte nativo a datasets grandes e interatividade avançada — tudo em uma API declarativa baseada em opções JSON.

> Documentação oficial: [echarts.apache.org](https://echarts.apache.org/en/index.html) · [Exemplos](https://echarts.apache.org/examples/) · [API Reference](https://echarts.apache.org/en/api.html)

---

### Visão Geral e Diferenciais

| Característica | Apache ECharts | Chart.js | D3.js |
|----------------|---------------|----------|-------|
| **Abordagem** | Declarativa (JSON de opções) | Declarativa (objeto de config) | Imperativa (DOM manipulation) |
| **Tipos de gráficos** | 20+ nativos (incluindo mapas, grafos, Sankey, árvores, heatmap, treemap, sunburst, radar, candlestick, gauge, funnel, etc.) | ~8 tipos nativos | Ilimitado (você constrói) |
| **Renderizador** | Canvas (padrão) + SVG (alternável) | Canvas | SVG |
| **Datasets grandes** | Excelente (sampling, progressive rendering, dataZoom) | Limitado (~5K pontos) | Manual (precisa otimizar) |
| **Temas** | Built-in (dark/light) + editor de temas online | Manual | Manual |
| **Tamanho (min+gzip)** | ~400 KB (completo) / ~200 KB (tree-shaking) | ~65 KB | ~80 KB |
| **Tooltips e interações** | Ricos e configuráveis nativamente | Básicos (extensíveis via plugins) | Manual |
| **Mapas geográficos** | Nativos (GeoJSON, registrar mapas) | Não | Sim (d3-geo) |
| **Licença** | Apache 2.0 | MIT | ISC |

> **Quando escolher ECharts:** ideal para dashboards corporativos com muitos tipos de gráfico, visualizações de dados complexas (Sankey, treemap, grafos), projetos que precisam de temas consistentes e interatividade rica com mínima customização manual.

---

### Instalação e Configuração

```bash
# Instalação via npm
npm install echarts

# Wrapper para Vue 3
npm install echarts vue-echarts

# Wrapper para React
npm install echarts echarts-for-react

# Wrapper para Angular
npm install echarts ngx-echarts
```

**Uso básico com CDN:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>ECharts — Exemplo Básico</title>
  <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
  <style>
    #grafico { width: 600px; height: 400px; }
  </style>
</head>
<body>
  <div id="grafico"></div>

  <script>
    const chart = echarts.init(document.getElementById('grafico'));

    const option = {
      title: {
        text: 'Vendas Mensais',
        subtext: 'Dados fictícios',
        left: 'center'
      },
      tooltip: {
        trigger: 'axis'
      },
      xAxis: {
        type: 'category',
        data: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun']
      },
      yAxis: {
        type: 'value',
        axisLabel: { formatter: 'R$ {value}' }
      },
      series: [{
        name: 'Vendas',
        type: 'bar',
        data: [45000, 52000, 61000, 48000, 72000, 65000],
        itemStyle: {
          color: '#5470c6',
          borderRadius: [4, 4, 0, 0]
        }
      }]
    };

    chart.setOption(option);

    window.addEventListener('resize', () => chart.resize());
  </script>
</body>
</html>
```

**Uso com ES Modules e tree-shaking:**

```javascript
import * as echarts from 'echarts/core';
import { BarChart, LineChart, PieChart } from 'echarts/charts';
import {
  TitleComponent, TooltipComponent, GridComponent,
  LegendComponent, DataZoomComponent, ToolboxComponent
} from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';

echarts.use([
  BarChart, LineChart, PieChart,
  TitleComponent, TooltipComponent, GridComponent,
  LegendComponent, DataZoomComponent, ToolboxComponent,
  CanvasRenderer
]);

const chart = echarts.init(document.getElementById('grafico'));
chart.setOption({ /* ... */ });
```

> **Tree-shaking** reduz o bundle de ~400 KB para ~200 KB ou menos, importando apenas os componentes necessários.

---

### Tipos de Gráficos

ECharts oferece mais de 20 tipos de gráficos nativos. Os mais usados:

**Gráfico de linhas com área:**

```javascript
const option = {
  title: { text: 'Acessos ao Site' },
  tooltip: {
    trigger: 'axis',
    axisPointer: { type: 'cross' }
  },
  legend: { data: ['Desktop', 'Mobile'] },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: ['Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb', 'Dom']
  },
  yAxis: { type: 'value' },
  series: [
    {
      name: 'Desktop',
      type: 'line',
      smooth: true,
      areaStyle: { opacity: 0.3 },
      data: [820, 932, 901, 1234, 1290, 1330, 1520]
    },
    {
      name: 'Mobile',
      type: 'line',
      smooth: true,
      areaStyle: { opacity: 0.3 },
      data: [620, 732, 801, 934, 1090, 1230, 1420]
    }
  ]
};
```

**Gráfico de pizza/rosca (doughnut):**

```javascript
const option = {
  title: {
    text: 'Distribuição por Canal',
    left: 'center'
  },
  tooltip: {
    trigger: 'item',
    formatter: '{b}: {c} ({d}%)'
  },
  legend: {
    orient: 'vertical',
    left: 'left'
  },
  series: [{
    name: 'Canal',
    type: 'pie',
    radius: ['40%', '70%'], // rosca (doughnut)
    avoidLabelOverlap: true,
    itemStyle: {
      borderRadius: 6,
      borderColor: '#fff',
      borderWidth: 2
    },
    label: {
      show: true,
      formatter: '{b}\n{d}%'
    },
    emphasis: {
      label: { show: true, fontSize: 16, fontWeight: 'bold' }
    },
    data: [
      { value: 1048, name: 'Orgânico' },
      { value: 735, name: 'Direto' },
      { value: 580, name: 'Email' },
      { value: 484, name: 'Social Media' },
      { value: 300, name: 'Referência' }
    ]
  }]
};
```

**Gráfico de radar:**

```javascript
const option = {
  title: { text: 'Avaliação de Competências' },
  tooltip: {},
  legend: { data: ['Candidato A', 'Candidato B'] },
  radar: {
    indicator: [
      { name: 'JavaScript', max: 10 },
      { name: 'CSS', max: 10 },
      { name: 'React', max: 10 },
      { name: 'Node.js', max: 10 },
      { name: 'SQL', max: 10 },
      { name: 'DevOps', max: 10 }
    ],
    shape: 'circle'
  },
  series: [{
    type: 'radar',
    data: [
      {
        value: [9, 7, 8, 6, 5, 4],
        name: 'Candidato A',
        areaStyle: { opacity: 0.2 }
      },
      {
        value: [6, 8, 5, 9, 8, 7],
        name: 'Candidato B',
        areaStyle: { opacity: 0.2 }
      }
    ]
  }]
};
```

**Heatmap:**

```javascript
const horas = ['00h', '01h', '02h', '03h', '04h', '05h', '06h', '07h',
               '08h', '09h', '10h', '11h', '12h', '13h', '14h', '15h',
               '16h', '17h', '18h', '19h', '20h', '21h', '22h', '23h'];
const dias = ['Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb', 'Dom'];

// dados: [horaIndex, diaIndex, valor]
const dados = [];
for (let i = 0; i < 7; i++) {
  for (let j = 0; j < 24; j++) {
    dados.push([j, i, Math.round(Math.random() * 100)]);
  }
}

const option = {
  title: { text: 'Acessos por Hora e Dia da Semana' },
  tooltip: {
    position: 'top',
    formatter: (p) => `${dias[p.value[1]]} ${horas[p.value[0]]}: ${p.value[2]} acessos`
  },
  xAxis: { type: 'category', data: horas, splitArea: { show: true } },
  yAxis: { type: 'category', data: dias, splitArea: { show: true } },
  visualMap: {
    min: 0, max: 100,
    calculable: true,
    orient: 'horizontal',
    left: 'center', bottom: '5%',
    inRange: { color: ['#e0f3db', '#a8ddb5', '#43a2ca', '#0868ac'] }
  },
  series: [{
    type: 'heatmap',
    data: dados,
    label: { show: true, fontSize: 9 },
    emphasis: {
      itemStyle: { shadowBlur: 10, shadowColor: 'rgba(0,0,0,0.5)' }
    }
  }]
};
```

**Treemap:**

```javascript
const option = {
  title: { text: 'Uso de Disco por Pasta', left: 'center' },
  tooltip: {
    formatter: (info) => `${info.name}: ${info.value} MB`
  },
  series: [{
    type: 'treemap',
    data: [
      {
        name: 'src',
        value: 450,
        children: [
          { name: 'components', value: 180 },
          { name: 'pages', value: 120 },
          { name: 'utils', value: 80 },
          { name: 'styles', value: 70 }
        ]
      },
      {
        name: 'node_modules',
        value: 1200,
        children: [
          { name: 'react', value: 300 },
          { name: 'typescript', value: 250 },
          { name: 'webpack', value: 200 },
          { name: 'outros', value: 450 }
        ]
      },
      {
        name: 'public',
        value: 300,
        children: [
          { name: 'images', value: 200 },
          { name: 'fonts', value: 60 },
          { name: 'icons', value: 40 }
        ]
      }
    ],
    label: { show: true, formatter: '{b}\n{c} MB' },
    upperLabel: { show: true, height: 20 },
    levels: [
      { itemStyle: { borderColor: '#555', borderWidth: 2, gapWidth: 2 } },
      { colorSaturation: [0.3, 0.7], itemStyle: { borderColorSaturation: 0.6, gapWidth: 1 } }
    ]
  }]
};
```

**Diagrama Sankey:**

```javascript
const option = {
  title: { text: 'Fluxo de Conversão do Funil' },
  tooltip: { trigger: 'item', triggerOn: 'mousemove' },
  series: [{
    type: 'sankey',
    emphasis: { focus: 'adjacency' },
    nodeAlign: 'left',
    data: [
      { name: 'Visitantes' },
      { name: 'Cadastro' },
      { name: 'Ativação' },
      { name: 'Plano Free' },
      { name: 'Plano Pro' },
      { name: 'Churn' },
      { name: 'Retidos' }
    ],
    links: [
      { source: 'Visitantes', target: 'Cadastro', value: 5000 },
      { source: 'Visitantes', target: 'Churn', value: 15000 },
      { source: 'Cadastro', target: 'Ativação', value: 3500 },
      { source: 'Cadastro', target: 'Churn', value: 1500 },
      { source: 'Ativação', target: 'Plano Free', value: 2000 },
      { source: 'Ativação', target: 'Plano Pro', value: 1500 },
      { source: 'Plano Free', target: 'Retidos', value: 1200 },
      { source: 'Plano Free', target: 'Churn', value: 800 },
      { source: 'Plano Pro', target: 'Retidos', value: 1400 },
      { source: 'Plano Pro', target: 'Churn', value: 100 }
    ],
    lineStyle: { color: 'gradient', curveness: 0.5 }
  }]
};
```

**Gauge (medidor):**

```javascript
const option = {
  series: [{
    type: 'gauge',
    startAngle: 180,
    endAngle: 0,
    min: 0,
    max: 100,
    splitNumber: 5,
    progress: { show: true, width: 18 },
    axisLine: { lineStyle: { width: 18 } },
    axisTick: { show: false },
    splitLine: { length: 12, lineStyle: { width: 2, color: '#999' } },
    axisLabel: { distance: 25, fontSize: 12 },
    anchor: { show: true, size: 20, itemStyle: { borderWidth: 2 } },
    title: { show: true, offsetCenter: [0, '70%'], fontSize: 16 },
    detail: {
      valueAnimation: true,
      fontSize: 28,
      offsetCenter: [0, '40%'],
      formatter: '{value}%'
    },
    data: [{ value: 72, name: 'CPU' }]
  }]
};
```

**Candlestick (gráfico financeiro):**

```javascript
const option = {
  title: { text: 'PETR4 — Cotação Diária' },
  tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } },
  xAxis: {
    type: 'category',
    data: ['2024-01-02', '2024-01-03', '2024-01-04', '2024-01-05', '2024-01-08']
  },
  yAxis: { type: 'value', scale: true },
  series: [{
    type: 'candlestick',
    // dados: [abertura, fechamento, mínima, máxima]
    data: [
      [36.80, 37.20, 36.50, 37.50],
      [37.10, 36.60, 36.40, 37.30],
      [36.70, 37.40, 36.50, 37.60],
      [37.30, 37.80, 37.10, 38.00],
      [37.70, 37.50, 37.20, 38.10]
    ],
    itemStyle: {
      color: '#06b6d4',       // alta (bullish)
      color0: '#ef4444',      // baixa (bearish)
      borderColor: '#06b6d4',
      borderColor0: '#ef4444'
    }
  }]
};
```

> **Mais tipos nativos:** `scatter`, `effectScatter` (scatter com ondas), `graph` (grafos/redes), `tree`, `sunburst`, `parallel` (coordenadas paralelas), `funnel`, `themeRiver`, `boxplot`, `map` (geográfico), `pictorialBar`, `custom` (renderização customizada).

---

### Interatividade Avançada

**DataZoom (zoom em dados):**

```javascript
const option = {
  title: { text: 'Série Temporal com Zoom' },
  tooltip: { trigger: 'axis' },
  xAxis: {
    type: 'category',
    data: Array.from({ length: 365 }, (_, i) => {
      const d = new Date(2024, 0, i + 1);
      return d.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit' });
    })
  },
  yAxis: { type: 'value' },
  dataZoom: [
    { type: 'slider', start: 80, end: 100 },           // barra inferior
    { type: 'inside', start: 80, end: 100 }             // zoom com scroll
  ],
  series: [{
    type: 'line',
    smooth: true,
    data: Array.from({ length: 365 }, () => Math.round(Math.random() * 500 + 200)),
    areaStyle: { opacity: 0.2 }
  }]
};
```

**Toolbox (ferramentas integradas):**

```javascript
const option = {
  toolbox: {
    feature: {
      saveAsImage: { title: 'Salvar como imagem' },
      dataView: { title: 'Ver dados', lang: ['Dados', 'Fechar', 'Atualizar'] },
      magicType: {
        type: ['line', 'bar', 'stack'],
        title: { line: 'Linha', bar: 'Barra', stack: 'Empilhado' }
      },
      restore: { title: 'Restaurar' },
      dataZoom: { title: { zoom: 'Zoom', back: 'Voltar' } }
    }
  },
  // ... restante da configuração
};
```

**Eventos e interação via JavaScript:**

```javascript
const chart = echarts.init(document.getElementById('grafico'));

chart.on('click', (params) => {
  console.log('Clicado:', params.name, params.value);
  // Navegar para detalhe, abrir modal, etc.
});

chart.on('legendselectchanged', (params) => {
  console.log('Legenda alterada:', params.selected);
});

// Dispatch de ações programáticas
chart.dispatchAction({
  type: 'highlight',
  seriesIndex: 0,
  dataIndex: 2
});

// Conectar múltiplos gráficos (tooltips sincronizados)
echarts.connect([chart1, chart2]);
```

---

### Temas e Customização Visual

**Temas built-in:**

```javascript
// Tema dark
const chart = echarts.init(document.getElementById('grafico'), 'dark');

// Tema customizado via editor online
// https://echarts.apache.org/en/theme-builder.html
```

**Registrar tema customizado:**

```javascript
echarts.registerTheme('meuTema', {
  color: ['#5470c6', '#91cc75', '#fac858', '#ee6666', '#73c0de', '#3ba272'],
  backgroundColor: '#1a1a2e',
  textStyle: { color: '#e0e0e0' },
  title: {
    textStyle: { color: '#ffffff' },
    subtextStyle: { color: '#aaaaaa' }
  },
  legend: { textStyle: { color: '#cccccc' } },
  categoryAxis: {
    axisLine: { lineStyle: { color: '#444' } },
    axisLabel: { color: '#ccc' }
  },
  valueAxis: {
    axisLine: { lineStyle: { color: '#444' } },
    splitLine: { lineStyle: { color: '#333' } },
    axisLabel: { color: '#ccc' }
  }
});

const chart = echarts.init(document.getElementById('grafico'), 'meuTema');
```

> **Editor de temas online:** [echarts.apache.org/en/theme-builder.html](https://echarts.apache.org/en/theme-builder.html) — permite criar temas visualmente e exportar como JSON ou JavaScript.

---

### Mapas Geográficos

```javascript
// Registrar mapa do Brasil (GeoJSON)
fetch('https://raw.githubusercontent.com/codeforamerica/click_that_hood/master/public/data/brazil-states.geojson')
  .then(res => res.json())
  .then(geoJson => {
    echarts.registerMap('BR', geoJson);

    const option = {
      title: { text: 'População por Estado', left: 'center' },
      tooltip: {
        trigger: 'item',
        formatter: '{b}: {c} milhões'
      },
      visualMap: {
        min: 1, max: 46,
        text: ['Alta', 'Baixa'],
        calculable: true,
        inRange: { color: ['#e0f3db', '#a8ddb5', '#43a2ca', '#0868ac'] }
      },
      series: [{
        type: 'map',
        map: 'BR',
        roam: true,
        emphasis: {
          label: { show: true, fontSize: 14 },
          itemStyle: { areaColor: '#ffd700' }
        },
        data: [
          { name: 'São Paulo', value: 46 },
          { name: 'Minas Gerais', value: 21 },
          { name: 'Rio de Janeiro', value: 17 },
          { name: 'Bahia', value: 15 },
          { name: 'Paraná', value: 11 },
          { name: 'Rio Grande do Sul', value: 11 },
          // ... demais estados
        ]
      }]
    };

    chart.setOption(option);
  });
```

---

### Gráficos com Múltiplos Eixos e Séries Mistas

```javascript
const option = {
  title: { text: 'Vendas e Lucro Mensal' },
  tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } },
  legend: { data: ['Vendas', 'Custos', 'Margem %'] },
  xAxis: {
    type: 'category',
    data: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun']
  },
  yAxis: [
    {
      type: 'value',
      name: 'R$ (milhares)',
      axisLabel: { formatter: 'R$ {value}k' }
    },
    {
      type: 'value',
      name: 'Margem (%)',
      axisLabel: { formatter: '{value}%' },
      splitLine: { show: false }
    }
  ],
  series: [
    {
      name: 'Vendas',
      type: 'bar',
      data: [120, 135, 150, 140, 170, 160],
      itemStyle: { color: '#5470c6' }
    },
    {
      name: 'Custos',
      type: 'bar',
      data: [85, 92, 100, 98, 110, 105],
      itemStyle: { color: '#ee6666' }
    },
    {
      name: 'Margem %',
      type: 'line',
      yAxisIndex: 1,
      smooth: true,
      data: [29, 32, 33, 30, 35, 34],
      itemStyle: { color: '#91cc75' },
      lineStyle: { width: 3 }
    }
  ]
};
```

---

### Datasets Grandes e Performance

ECharts suporta datasets grandes nativamente com estratégias de otimização:

```javascript
// Gráfico de dispersão com 100.000 pontos
const dados = Array.from({ length: 100000 }, () => [
  Math.random() * 1000,
  Math.random() * 1000
]);

const option = {
  title: { text: '100K Pontos — Scatter Plot' },
  tooltip: {},
  xAxis: { type: 'value' },
  yAxis: { type: 'value' },
  series: [{
    type: 'scatter',
    data: dados,
    symbolSize: 2,
    large: true,           // ativa renderização otimizada para grandes datasets
    largeThreshold: 5000,  // limiar para ativar o modo large
    progressive: 5000,     // renderização progressiva (5000 pontos por frame)
    progressiveThreshold: 10000
  }]
};
```

**Renderizador SVG vs Canvas:**

```javascript
// Canvas (padrão) — melhor para muitos elementos, animações, datasets grandes
const chart = echarts.init(document.getElementById('grafico'), null, { renderer: 'canvas' });

// SVG — melhor para poucos elementos, exportação vetorial, acessibilidade
const chart = echarts.init(document.getElementById('grafico'), null, { renderer: 'svg' });
```

| Renderizador | Vantagem | Melhor para |
|-------------|----------|-------------|
| **Canvas** | Performance com muitos elementos, animações fluidas | Dashboards com dados dinâmicos, datasets grandes, gráficos animados |
| **SVG** | Escalável, acessível, exportável como vetor | Relatórios impressos, poucos elementos, integração com DOM |

---

### Responsividade

```javascript
const chart = echarts.init(document.getElementById('grafico'));
chart.setOption(option);

// Redimensionar com a janela
window.addEventListener('resize', () => chart.resize());

// Ou usar ResizeObserver para containers flexíveis
const observer = new ResizeObserver(() => chart.resize());
observer.observe(document.getElementById('grafico'));
```

**Media query com ECharts (opções responsivas):**

```javascript
const option = {
  // configuração base
  title: { text: 'Vendas', left: 'center' },
  series: [{ type: 'bar', data: [120, 200, 150, 80, 70] }],

  // opções responsivas por tamanho do container
  media: [
    {
      query: { maxWidth: 500 },
      option: {
        title: { textStyle: { fontSize: 12 } },
        legend: { orient: 'horizontal', bottom: 0 },
        grid: { left: '15%', right: '5%' },
        series: [{ label: { show: false } }]
      }
    },
    {
      query: { minWidth: 501 },
      option: {
        title: { textStyle: { fontSize: 18 } },
        legend: { orient: 'vertical', right: 10, top: 'middle' },
        grid: { left: '10%', right: '15%' }
      }
    }
  ]
};
```

---

### Integração com React

```bash
npm install echarts echarts-for-react
```

```tsx
// VendasChart.tsx
import ReactECharts from 'echarts-for-react';

interface Props {
  dados: number[];
  categorias: string[];
}

export function VendasChart({ dados, categorias }: Props) {
  const option = {
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'category', data: categorias },
    yAxis: { type: 'value' },
    series: [{
      type: 'bar',
      data: dados,
      itemStyle: { color: '#5470c6', borderRadius: [4, 4, 0, 0] }
    }]
  };

  return (
    <ReactECharts
      option={option}
      style={{ height: 400 }}
      notMerge={true}
      lazyUpdate={true}
    />
  );
}
```

```tsx
// Dashboard com múltiplos gráficos
import ReactECharts from 'echarts-for-react';
import { useRef, useCallback } from 'react';

export function Dashboard() {
  const chartRef = useRef<ReactECharts>(null);

  const handleClick = useCallback((params: any) => {
    console.log('Clicado:', params.name);
  }, []);

  const onEvents = { click: handleClick };

  return (
    <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 16 }}>
      <ReactECharts
        ref={chartRef}
        option={barOption}
        onEvents={onEvents}
        style={{ height: 350 }}
      />
      <ReactECharts option={pieOption} style={{ height: 350 }} />
      <ReactECharts option={lineOption} style={{ height: 350, gridColumn: 'span 2' }} />
    </div>
  );
}
```

### Integração com Vue 3

```bash
npm install echarts vue-echarts
```

```html
<!-- DashboardChart.vue -->
<template>
  <v-chart :option="chartOption" autoresize style="height: 400px" />
</template>

<script setup lang="ts">
import { computed } from 'vue';
import VChart from 'vue-echarts';
import { use } from 'echarts/core';
import { BarChart, PieChart } from 'echarts/charts';
import {
  TitleComponent, TooltipComponent,
  GridComponent, LegendComponent
} from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';

use([BarChart, PieChart, TitleComponent, TooltipComponent,
     GridComponent, LegendComponent, CanvasRenderer]);

interface Props {
  dados: { nome: string; valor: number }[];
  titulo?: string;
}

const props = withDefaults(defineProps<Props>(), { titulo: 'Gráfico' });

const chartOption = computed(() => ({
  title: { text: props.titulo, left: 'center' },
  tooltip: { trigger: 'item', formatter: '{b}: {c} ({d}%)' },
  series: [{
    type: 'pie',
    radius: ['35%', '65%'],
    data: props.dados.map(d => ({ name: d.nome, value: d.valor })),
    emphasis: {
      label: { show: true, fontSize: 16, fontWeight: 'bold' }
    }
  }]
}));
</script>
```

### Integração com Angular

```bash
npm install echarts ngx-echarts
```

```typescript
// app.config.ts
import { provideEcharts } from 'ngx-echarts';

export const appConfig = {
  providers: [
    provideEcharts()
  ]
};
```

```typescript
// vendas-chart.component.ts
import { Component, Input, OnChanges } from '@angular/core';
import { NgxEchartsDirective } from 'ngx-echarts';
import type { EChartsOption } from 'echarts';

@Component({
  selector: 'app-vendas-chart',
  standalone: true,
  imports: [NgxEchartsDirective],
  template: `
    <div echarts [options]="chartOption" [merge]="mergeOption"
         style="height: 400px;" (chartClick)="onChartClick($event)">
    </div>
  `
})
export class VendasChartComponent implements OnChanges {
  @Input() dados: number[] = [];
  @Input() categorias: string[] = [];

  chartOption: EChartsOption = {};
  mergeOption: EChartsOption = {};

  ngOnChanges() {
    this.chartOption = {
      tooltip: { trigger: 'axis' },
      xAxis: { type: 'category', data: this.categorias },
      yAxis: { type: 'value' },
      series: [{
        type: 'bar',
        data: this.dados,
        itemStyle: { color: '#5470c6' }
      }]
    };
  }

  onChartClick(event: any) {
    console.log('Clicado:', event.name, event.value);
  }
}
```

---

### ECharts vs Chart.js vs D3.js — Quando Usar Cada Um

| Critério | Apache ECharts | Chart.js | D3.js |
|----------|---------------|----------|-------|
| **Variedade de gráficos** | 20+ nativos (mapas, Sankey, gauge, etc.) | ~8 tipos | Ilimitado (manual) |
| **Facilidade de uso** | Fácil (JSON declarativo) | Muito fácil | Difícil (curva de aprendizado alta) |
| **Customização visual** | Alta (temas, editor visual) | Média | Total (você controla tudo) |
| **Datasets grandes** | Excelente (progressive rendering) | Limitado | Manual |
| **Bundle size** | ~200-400 KB | ~65 KB | ~80 KB |
| **Interatividade** | Rica nativamente | Básica (plugins) | Manual (total controle) |
| **Mapas geográficos** | Nativos | Não | Sim (d3-geo) |
| **Gráficos financeiros** | Candlestick nativo | Não | Manual |
| **Ecossistema React/Vue/Angular** | Bom (wrappers oficiais) | Bom | Precisa integrar manualmente |
| **Melhor para** | Dashboards corporativos, visualizações complexas | Gráficos simples e rápidos | Visualizações totalmente customizadas |

---

## 7. Pixi.js — Engine 2D com WebGL

Pixi.js é um engine de renderização 2D de alto desempenho que utiliza WebGL (com fallback automático para Canvas). É ideal para jogos 2D, visualizações interativas complexas, animações ricas e aplicações que exigem renderização em tempo real de milhares de elementos.

> Referência: [PixiJS Official](https://pixijs.com/) · [PixiJS API Docs](https://pixijs.download/release/docs/) · [PixiJS Examples](https://pixijs.io/examples/)

---

### Arquitetura e Conceitos

**Quando usar Pixi.js:**

| Cenário | Pixi.js é adequado? |
|---|---|
| Jogo 2D (sprites, animações, colisões) | Sim |
| Dashboard com milhares de pontos animados | Sim |
| Animações interativas complexas (ex.: editor visual) | Sim |
| Gráficos de dados simples (bar chart, line chart) | Nao — use Chart.js ou D3.js |
| Conteúdo textual com layout (blog, formulário) | Nao — use HTML/CSS |

**Hierarquia de objetos:**

```
Application
  └── Stage (Container raiz)
        ├── Container "cenário"
        │     ├── Sprite "fundo"
        │     ├── Sprite "personagem"
        │     └── Graphics "chão"
        ├── Container "UI"
        │     ├── Text "pontuação"
        │     └── Sprite "botão"
        └── Container "partículas"
              └── ... (muitos sprites)
```

| Conceito | Descrição |
|---|---|
| **Application** | Gerencia o renderer, stage, ticker e resize |
| **Stage** | Container raiz da cena — tudo visível é filho do stage |
| **Container** | Agrupa display objects — transformações propagam para filhos |
| **Sprite** | Imagem renderizável posicionável na cena |
| **Graphics** | Desenho vetorial (retângulos, círculos, linhas) |
| **Text** | Texto renderizado na cena |
| **Ticker** | Loop de atualização (~60 FPS) |

**Instalação e Setup:**

```bash
npm install pixi.js
```

```html
<!-- Via CDN -->
<script src="https://cdn.jsdelivr.net/npm/pixi.js@8"></script>
```

```javascript
import { Application, Sprite, Graphics, Text, TextStyle } from 'pixi.js';

// Pixi.js v8 — inicialização assíncrona
const app = new Application();

async function inicializar() {
  // Inicializar aplicação (renderer WebGL/WebGPU + canvas fallback)
  await app.init({
    width: 800,
    height: 600,
    background: '#1a1a2e',
    antialias: true,
    resolution: window.devicePixelRatio || 1,
    autoDensity: true
  });

  // Adicionar canvas ao DOM
  document.getElementById('container').appendChild(app.canvas);

  // Criar um retângulo com Graphics
  const retangulo = new Graphics()
    .rect(100, 100, 200, 150)
    .fill('#e63946');

  app.stage.addChild(retangulo);

  // Criar texto
  const estilo = new TextStyle({
    fontFamily: 'Arial',
    fontSize: 24,
    fill: '#ffffff',
    fontWeight: 'bold'
  });

  const texto = new Text({ text: 'Hello Pixi.js!', style: estilo });
  texto.x = 100;
  texto.y = 50;
  app.stage.addChild(texto);
}

inicializar();
```

---

### Carregamento de Assets

```javascript
import { Application, Sprite, Assets } from 'pixi.js';

const app = new Application();

async function carregarAssets() {
  await app.init({ width: 800, height: 600, background: '#1a1a2e' });
  document.body.appendChild(app.canvas);

  // Carregar uma única textura
  const textura = await Assets.load('imagens/personagem.png');
  const sprite = new Sprite(textura);
  sprite.x = 400;
  sprite.y = 300;
  sprite.anchor.set(0.5); // centro como ponto de origem
  app.stage.addChild(sprite);

  // Carregar múltiplas texturas
  const assets = await Assets.load([
    { alias: 'heroi', src: 'imagens/heroi.png' },
    { alias: 'inimigo', src: 'imagens/inimigo.png' },
    { alias: 'fundo', src: 'imagens/fundo.jpg' }
  ]);

  const fundo = new Sprite(assets.fundo);
  fundo.width = 800;
  fundo.height = 600;
  app.stage.addChild(fundo);

  const heroi = new Sprite(assets.heroi);
  heroi.anchor.set(0.5);
  heroi.x = 400;
  heroi.y = 300;
  app.stage.addChild(heroi);
}

carregarAssets();
```

**Spritesheets (Atlas de Texturas):**

```javascript
// spritesheet.json — formato padrão
// Contém metadados sobre as posições de cada frame na imagem atlas

async function carregarSpritesheet() {
  // Carregar spritesheet (json + imagem)
  const sheet = await Assets.load('sprites/personagem.json');

  // Acessar texturas individuais do spritesheet
  const frame1 = Sprite.from('personagem_andar_01.png');
  const frame2 = Sprite.from('personagem_andar_02.png');

  app.stage.addChild(frame1);
}
```

> **Dica:** Use ferramentas como [TexturePacker](https://www.codeandweb.com/texturepacker) ou [Shoebox](https://renderhjs.net/shoebox/) para gerar spritesheets otimizados a partir de imagens individuais.

---

### Sprites e Containers

```javascript
import { Application, Sprite, Container, Assets } from 'pixi.js';

const app = new Application();

async function criarCena() {
  await app.init({ width: 800, height: 600, background: '#16213e' });
  document.body.appendChild(app.canvas);

  const textura = await Assets.load('imagens/estrela.png');

  // Container para agrupar objetos
  const grupoEstrelas = new Container();
  grupoEstrelas.x = 400;
  grupoEstrelas.y = 300;
  app.stage.addChild(grupoEstrelas);

  // Criar múltiplos sprites
  for (let i = 0; i < 20; i++) {
    const estrela = new Sprite(textura);

    // Posição relativa ao container
    estrela.x = (Math.random() - 0.5) * 600;
    estrela.y = (Math.random() - 0.5) * 400;

    // Anchor — ponto de origem (0,0 = topo-esquerdo, 0.5 = centro)
    estrela.anchor.set(0.5);

    // Escala
    const escala = 0.3 + Math.random() * 0.7;
    estrela.scale.set(escala);

    // Rotação (em radianos)
    estrela.rotation = Math.random() * Math.PI * 2;

    // Alpha (transparência)
    estrela.alpha = 0.5 + Math.random() * 0.5;

    // Tint (colorir a textura)
    estrela.tint = Math.random() * 0xffffff;

    grupoEstrelas.addChild(estrela);
  }

  // Transformações no container afetam todos os filhos
  grupoEstrelas.rotation = 0.1;
  grupoEstrelas.scale.set(0.8);

  // Pivot — ponto de rotação do container
  grupoEstrelas.pivot.set(0, 0);
}

criarCena();
```

---

### Animações e Interatividade

```javascript
import { Application, Sprite, Graphics, Assets, FederatedPointerEvent } from 'pixi.js';

const app = new Application();

async function criarCenaInterativa() {
  await app.init({ width: 800, height: 600, background: '#0f3460' });
  document.body.appendChild(app.canvas);

  // --- Animação com Ticker ---
  const textura = await Assets.load('imagens/nave.png');
  const nave = new Sprite(textura);
  nave.anchor.set(0.5);
  nave.x = 400;
  nave.y = 300;
  nave.scale.set(0.5);
  app.stage.addChild(nave);

  // Game loop — executa a cada frame (~60 FPS)
  let tempo = 0;
  app.ticker.add((ticker) => {
    tempo += ticker.deltaTime;

    // Rotação contínua
    nave.rotation += 0.02 * ticker.deltaTime;

    // Movimento senoidal
    nave.y = 300 + Math.sin(tempo * 0.05) * 50;
  });

  // --- Interatividade ---
  const botao = new Graphics()
    .roundRect(300, 500, 200, 50, 12)
    .fill('#e63946');

  // Habilitar interatividade
  botao.eventMode = 'static'; // 'static' = interativo, 'none' = ignora eventos
  botao.cursor = 'pointer';

  // Eventos de mouse/touch
  botao.on('pointerdown', () => {
    botao.tint = 0xaaaaaa;
    nave.scale.set(nave.scale.x + 0.1);
  });

  botao.on('pointerup', () => {
    botao.tint = 0xffffff;
  });

  botao.on('pointerover', () => {
    botao.alpha = 0.8;
  });

  botao.on('pointerout', () => {
    botao.alpha = 1;
  });

  app.stage.addChild(botao);

  // --- Drag and Drop ---
  const caixa = new Graphics()
    .rect(0, 0, 80, 80)
    .fill('#2a9d8f');

  caixa.x = 100;
  caixa.y = 200;
  caixa.eventMode = 'static';
  caixa.cursor = 'grab';

  let arrastando = false;
  let offsetX = 0, offsetY = 0;

  caixa.on('pointerdown', (event: FederatedPointerEvent) => {
    arrastando = true;
    caixa.cursor = 'grabbing';
    caixa.alpha = 0.7;
    const pos = event.global;
    offsetX = pos.x - caixa.x;
    offsetY = pos.y - caixa.y;
  });

  app.stage.eventMode = 'static';
  app.stage.on('pointermove', (event: FederatedPointerEvent) => {
    if (arrastando) {
      caixa.x = event.global.x - offsetX;
      caixa.y = event.global.y - offsetY;
    }
  });

  app.stage.on('pointerup', () => {
    arrastando = false;
    caixa.cursor = 'grab';
    caixa.alpha = 1;
  });

  app.stage.addChild(caixa);
}

criarCenaInterativa();
```

---

### Filtros e Efeitos Visuais

Pixi.js oferece filtros GPU que aplicam efeitos visuais em tempo real sobre sprites, containers ou toda a cena.

```javascript
import {
  Application, Sprite, Container, Assets,
  BlurFilter, ColorMatrixFilter
} from 'pixi.js';

const app = new Application();

async function aplicarFiltros() {
  await app.init({ width: 800, height: 600, background: '#1a1a2e' });
  document.body.appendChild(app.canvas);

  const textura = await Assets.load('imagens/paisagem.jpg');
  const imagem = new Sprite(textura);
  imagem.width = 800;
  imagem.height = 600;
  app.stage.addChild(imagem);

  // Filtro de desfoque (blur)
  const blur = new BlurFilter();
  blur.blur = 4; // intensidade (0 = sem blur)

  // Filtro de matriz de cores
  const colorMatrix = new ColorMatrixFilter();

  // Aplicar efeitos predefinidos
  colorMatrix.saturate(0.5);    // reduzir saturação
  // colorMatrix.greyscale(0.8);  // escala de cinza
  // colorMatrix.sepia();         // tom sépia
  // colorMatrix.brightness(1.2); // brilho
  // colorMatrix.contrast(1.5);   // contraste
  // colorMatrix.hue(90);         // rotação de matiz

  // Aplicar filtros ao sprite (array de filtros)
  imagem.filters = [colorMatrix];

  // Alternar filtros com interação
  const texturaBotao = await Assets.load('imagens/icone.png');
  const botaoBlur = new Sprite(texturaBotao);
  botaoBlur.x = 20;
  botaoBlur.y = 20;
  botaoBlur.eventMode = 'static';
  botaoBlur.cursor = 'pointer';

  let blurAtivo = false;
  botaoBlur.on('pointerdown', () => {
    blurAtivo = !blurAtivo;
    imagem.filters = blurAtivo
      ? [blur, colorMatrix]
      : [colorMatrix];
  });

  app.stage.addChild(botaoBlur);

  // Animar propriedades de filtros
  app.ticker.add((ticker) => {
    // Pulsação de blur
    if (blurAtivo) {
      blur.blur = 2 + Math.sin(Date.now() * 0.003) * 3;
    }
  });
}

aplicarFiltros();
```

---

### Particle Systems

Sistemas de partículas permitem criar efeitos visuais como fogo, fumaça, neve, explosões e faíscas com alto desempenho.

```bash
npm install @barvian/particle-emitter
```

```javascript
import { Application, Assets } from 'pixi.js';
import { Emitter } from '@barvian/particle-emitter';

const app = new Application();

async function criarParticulas() {
  await app.init({ width: 800, height: 600, background: '#0a0a23' });
  document.body.appendChild(app.canvas);

  const texturaParticula = await Assets.load('imagens/particula.png');

  // Configuração do emissor de partículas
  const emissor = new Emitter(app.stage, {
    lifetime: { min: 0.5, max: 1.5 },
    frequency: 0.01,
    maxParticles: 200,
    pos: { x: 400, y: 500 },
    behaviors: [
      {
        type: 'textureSingle',
        config: { texture: texturaParticula }
      },
      {
        type: 'alpha',
        config: {
          alpha: { list: [
            { value: 1, time: 0 },
            { value: 0, time: 1 }
          ]}
        }
      },
      {
        type: 'scale',
        config: {
          scale: { list: [
            { value: 0.5, time: 0 },
            { value: 0.1, time: 1 }
          ]}
        }
      },
      {
        type: 'moveSpeed',
        config: {
          speed: { list: [
            { value: 300, time: 0 },
            { value: 50, time: 1 }
          ]}
        }
      },
      {
        type: 'spawnShape',
        config: {
          type: 'torus',
          data: { x: 0, y: 0, radius: 20, innerRadius: 0 }
        }
      }
    ]
  });

  // Atualizar partículas no game loop
  app.ticker.add((ticker) => {
    emissor.update(ticker.deltaMS * 0.001);
  });
}

criarParticulas();
```

> **Dica:** Para efeitos simples de partículas sem dependências extras, crie sprites individuais gerenciados por um pool de objetos e animados no ticker.

---

### Performance

Pixi.js pode renderizar milhares de sprites a 60 FPS quando otimizado corretamente. As principais estratégias de otimização:

| Técnica | Descrição | Impacto |
|---|---|---|
| **Texture Atlases** | Combinar imagens em uma spritesheet | Reduz draw calls drasticamente |
| **Object Pooling** | Reutilizar objetos em vez de criar/destruir | Reduz garbage collection |
| **Culling** | Não renderizar objetos fora da tela | Reduz carga de GPU |
| **Batching** | Pixi agrupa automaticamente sprites com mesma textura | Menos draw calls |
| **Resolution** | Usar `resolution: 1` em dispositivos não-retina | Menos pixels processados |
| **Container caching** | `container.cacheAsTexture(true)` para containers estáticos | Renderiza uma vez como textura |
| **Evitar filters** | Filtros GPU são custosos — usar com parcimônia | Cada filtro = render pass extra |
| **ParticleContainer** | Container otimizado para muitos sprites simples | 2-5x mais rápido que Container |

```javascript
import { Application, Sprite, Assets } from 'pixi.js';

// Object Pooling — reutilizar sprites
class SpritePool {
  private pool: Sprite[] = [];

  obter(textura: any): Sprite {
    const sprite = this.pool.pop() || new Sprite(textura);
    sprite.visible = true;
    sprite.alpha = 1;
    sprite.scale.set(1);
    sprite.rotation = 0;
    return sprite;
  }

  devolver(sprite: Sprite) {
    sprite.visible = false;
    this.pool.push(sprite);
  }
}

// Culling — desabilitar objetos fora da tela
function aplicarCulling(container: any, viewport: { x: number; y: number; w: number; h: number }) {
  for (const filho of container.children) {
    const dentroDoViewport =
      filho.x + filho.width > viewport.x &&
      filho.x < viewport.x + viewport.w &&
      filho.y + filho.height > viewport.y &&
      filho.y < viewport.y + viewport.h;

    filho.renderable = dentroDoViewport;
  }
}
```

---

### Pixi.js vs Canvas Puro

| Critério | Pixi.js | Canvas 2D API |
|---|---|---|
| **Renderização** | WebGL/WebGPU (GPU) com fallback Canvas | CPU (Canvas 2D context) |
| **Performance (sprites)** | Excelente — milhares de sprites a 60 FPS | Limitada — centenas de sprites |
| **API** | Alto nível (Sprite, Container, Filters) | Baixo nível (drawImage, fillRect) |
| **Curva de aprendizado** | Média | Baixa |
| **Bundle size** | ~150 KB (gzip) | 0 KB (nativo do browser) |
| **Filtros/Efeitos** | BlurFilter, ColorMatrix (GPU) | Filtros manuais (CPU, lento) |
| **Interatividade** | eventMode nativo com hit testing | Manual (calcular posições) |
| **Hierarquia de cena** | Sim (Container → filhos) | Não — redesenhar tudo manualmente |
| **Texto** | TextStyle com opções ricas | fillText/strokeText básico |
| **Uso ideal** | Jogos, animações complexas, editores visuais | Desenhos simples, gráficos pontuais |

> **Resumo:** Use Canvas 2D puro para desenhos simples e pontuais. Use Pixi.js quando precisar de performance com muitos objetos, hierarquia de cena, filtros GPU ou interatividade avançada.

---

### Exemplo Prático: Animação Interativa

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Pixi.js — Cena Interativa</title>
  <script src="https://cdn.jsdelivr.net/npm/pixi.js@8"></script>
  <style>
    body { margin: 0; overflow: hidden; background: #0a0a23; }
    canvas { display: block; }
  </style>
</head>
<body>
<script>
async function main() {
  const app = new PIXI.Application();
  await app.init({
    width: window.innerWidth,
    height: window.innerHeight,
    background: '#0a0a23',
    antialias: true,
    resolution: window.devicePixelRatio || 1,
    autoDensity: true
  });
  document.body.appendChild(app.canvas);

  const particulas = [];
  const TOTAL = 150;

  // Criar partículas como círculos
  for (let i = 0; i < TOTAL; i++) {
    const circulo = new PIXI.Graphics()
      .circle(0, 0, 3 + Math.random() * 5)
      .fill([ '#e63946', '#457b9d', '#2a9d8f', '#e9c46a', '#f4a261' ][Math.floor(Math.random() * 5)]);

    circulo.x = Math.random() * app.screen.width;
    circulo.y = Math.random() * app.screen.height;
    circulo.alpha = 0.4 + Math.random() * 0.6;

    // Dados customizados para animação
    circulo.vx = (Math.random() - 0.5) * 2;
    circulo.vy = (Math.random() - 0.5) * 2;

    // Habilitar interatividade
    circulo.eventMode = 'static';
    circulo.cursor = 'pointer';
    circulo.on('pointerover', () => { circulo.scale.set(3); circulo.alpha = 1; });
    circulo.on('pointerout', () => { circulo.scale.set(1); circulo.alpha = 0.6; });

    app.stage.addChild(circulo);
    particulas.push(circulo);
  }

  // Mouse tracking
  let mouseX = app.screen.width / 2;
  let mouseY = app.screen.height / 2;

  app.stage.eventMode = 'static';
  app.stage.hitArea = app.screen;
  app.stage.on('pointermove', (e) => {
    mouseX = e.global.x;
    mouseY = e.global.y;
  });

  // Game loop — animação contínua
  app.ticker.add((ticker) => {
    for (const p of particulas) {
      // Movimento base
      p.x += p.vx * ticker.deltaTime;
      p.y += p.vy * ticker.deltaTime;

      // Atração suave em direção ao mouse
      const dx = mouseX - p.x;
      const dy = mouseY - p.y;
      const dist = Math.sqrt(dx * dx + dy * dy);
      if (dist < 200) {
        p.vx += (dx / dist) * 0.05;
        p.vy += (dy / dist) * 0.05;
      }

      // Limite de velocidade
      const speed = Math.sqrt(p.vx * p.vx + p.vy * p.vy);
      if (speed > 3) { p.vx *= 0.98; p.vy *= 0.98; }

      // Wrap — reaparecer do outro lado
      if (p.x < -10) p.x = app.screen.width + 10;
      if (p.x > app.screen.width + 10) p.x = -10;
      if (p.y < -10) p.y = app.screen.height + 10;
      if (p.y > app.screen.height + 10) p.y = -10;
    }
  });

  // Redimensionar com a janela
  window.addEventListener('resize', () => {
    app.renderer.resize(window.innerWidth, window.innerHeight);
    app.stage.hitArea = app.screen;
  });
}

main();
</script>
</body>
</html>
```

---
## 8. WebGL — Fundamentos

WebGL (Web Graphics Library) é uma API JavaScript de baixo nível para renderização de gráficos 2D e 3D no navegador, baseada no OpenGL ES. Ela permite acesso direto à GPU sem plugins, possibilitando visualizações interativas, jogos e experiências 3D na web.

> WebGL opera sobre o elemento `<canvas>` do HTML5 e é suportado por todos os navegadores modernos.

---

### Pipeline de Renderização

O pipeline de renderização WebGL transforma dados geométricos (vértices) em pixels na tela. Compreender esse fluxo é essencial para trabalhar com WebGL.

**Etapas do Pipeline:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PIPELINE DE RENDERIZAÇÃO WebGL                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Dados da       Vertex        Montagem de    Rasterização           │
│  Aplicação  ──► Shader    ──► Primitivas ──► (Interpolação)         │
│  (Buffers)      (GLSL)        (Triângulos)                          │
│                                                                     │
│                                     │                               │
│                                     ▼                               │
│                                                                     │
│  Tela ◄──── Operações ◄──── Fragment     ◄── Fragmentos            │
│  (Pixels)   de Saída        Shader           (Pixels candidatos)    │
│             (Depth,         (GLSL)                                  │
│              Blend)                                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

| Etapa | Descrição |
|-------|-----------|
| **Dados da Aplicação** | Vértices, cores, coordenadas UV e índices armazenados em buffers |
| **Vertex Shader** | Programa GLSL que processa cada vértice (posição, transformações) |
| **Montagem de Primitivas** | Agrupa vértices em triângulos, linhas ou pontos |
| **Rasterização** | Converte primitivas em fragmentos (pixels candidatos) |
| **Fragment Shader** | Programa GLSL que calcula a cor final de cada fragmento |
| **Operações de Saída** | Testes de profundidade, stencil, blending e escrita no framebuffer |

**Obtendo o contexto WebGL:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>WebGL — Contexto Básico</title>
  <style>
    canvas { border: 1px solid #333; display: block; margin: 20px auto; }
  </style>
</head>
<body>
  <canvas id="glCanvas" width="800" height="600"></canvas>

  <script>
    const canvas = document.getElementById('glCanvas');

    // Tenta obter contexto WebGL 2, com fallback para WebGL 1
    let gl = canvas.getContext('webgl2');
    let isWebGL2 = true;

    if (!gl) {
      gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
      isWebGL2 = false;
      console.warn('WebGL 2 não disponível, usando WebGL 1');
    }

    if (!gl) {
      document.body.innerHTML = '<p>Seu navegador não suporta WebGL.</p>';
      throw new Error('WebGL não suportado');
    }

    console.log(`Usando WebGL ${isWebGL2 ? '2' : '1'}`);
    console.log('Renderer:', gl.getParameter(gl.RENDERER));
    console.log('Vendor:', gl.getParameter(gl.VENDOR));

    // Limpa o canvas com cor azul escuro
    gl.clearColor(0.1, 0.1, 0.2, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
  </script>
</body>
</html>
```

---

### Shaders em GLSL

Shaders são pequenos programas executados na GPU escritos em GLSL (OpenGL Shading Language). WebGL exige dois tipos obrigatórios:

- **Vertex Shader**: processado uma vez por vértice; define a posição final na tela
- **Fragment Shader**: processado uma vez por fragmento (pixel candidato); define a cor final

**Tipos de dados GLSL:**

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| `float` | Número de ponto flutuante | `float x = 1.0;` |
| `int` | Inteiro | `int i = 5;` |
| `bool` | Booleano | `bool active = true;` |
| `vec2` | Vetor 2D (x, y) | `vec2 pos = vec2(1.0, 2.0);` |
| `vec3` | Vetor 3D (x, y, z) ou (r, g, b) | `vec3 cor = vec3(1.0, 0.0, 0.0);` |
| `vec4` | Vetor 4D (x, y, z, w) ou (r, g, b, a) | `vec4 rgba = vec4(1.0, 0.0, 0.0, 1.0);` |
| `mat2` | Matriz 2×2 | `mat2 m;` |
| `mat3` | Matriz 3×3 | `mat3 m;` |
| `mat4` | Matriz 4×4 | `mat4 mvp;` |
| `sampler2D` | Referência a textura 2D | `uniform sampler2D uTexture;` |

**Qualificadores de variáveis:**

| Qualificador | WebGL 1 | WebGL 2 (GLSL ES 3.0) | Descrição |
|--------------|---------|------------------------|-----------|
| `attribute` | ✔ | Substituído por `in` | Dados por vértice (posição, cor) |
| `uniform` | ✔ | ✔ | Constante por draw call (matrizes, tempo) |
| `varying` | ✔ | Substituído por `out`/`in` | Interpolado entre vertex e fragment shader |

**Exemplo — Par de shaders simples (WebGL 1):**

```glsl
// ===== VERTEX SHADER =====
attribute vec2 aPosition;
attribute vec3 aColor;
varying vec3 vColor;

void main() {
    vColor = aColor;
    gl_Position = vec4(aPosition, 0.0, 1.0);
}
```

```glsl
// ===== FRAGMENT SHADER =====
precision mediump float;
varying vec3 vColor;

void main() {
    gl_FragColor = vec4(vColor, 1.0);
}
```

**Mesmo exemplo em WebGL 2 (GLSL ES 3.0):**

```glsl
#version 300 es
// ===== VERTEX SHADER =====
in vec2 aPosition;
in vec3 aColor;
out vec3 vColor;

void main() {
    vColor = aColor;
    gl_Position = vec4(aPosition, 0.0, 1.0);
}
```

```glsl
#version 300 es
// ===== FRAGMENT SHADER =====
precision mediump float;
in vec3 vColor;
out vec4 fragColor;

void main() {
    fragColor = vec4(vColor, 1.0);
}
```

> No WebGL 2, `gl_FragColor` é substituído por uma variável de saída declarada com `out`.

---

### Buffers, Atributos e Uniforms

Buffers são blocos de memória na GPU que armazenam dados dos vértices. Para renderizar, é preciso: criar buffers, enviar dados, e vincular esses dados aos atributos dos shaders.

**Exemplo completo — Triângulo colorido:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>WebGL — Triângulo Colorido</title>
  <style>
    canvas { display: block; margin: 20px auto; }
  </style>
</head>
<body>
  <canvas id="glCanvas" width="600" height="600"></canvas>

  <script>
    const canvas = document.getElementById('glCanvas');
    const gl = canvas.getContext('webgl');

    // ---------- Código-fonte dos shaders ----------
    const vsSource = `
      attribute vec2 aPosition;
      attribute vec3 aColor;
      varying vec3 vColor;
      void main() {
          vColor = aColor;
          gl_Position = vec4(aPosition, 0.0, 1.0);
      }
    `;

    const fsSource = `
      precision mediump float;
      varying vec3 vColor;
      void main() {
          gl_FragColor = vec4(vColor, 1.0);
      }
    `;

    // ---------- Compilação de shaders ----------
    function compileShader(gl, source, type) {
      const shader = gl.createShader(type);
      gl.shaderSource(shader, source);
      gl.compileShader(shader);

      if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.error('Erro ao compilar shader:', gl.getShaderInfoLog(shader));
        gl.deleteShader(shader);
        return null;
      }
      return shader;
    }

    const vertexShader = compileShader(gl, vsSource, gl.VERTEX_SHADER);
    const fragmentShader = compileShader(gl, fsSource, gl.FRAGMENT_SHADER);

    // ---------- Criação do programa ----------
    const program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
      console.error('Erro ao linkar programa:', gl.getProgramInfoLog(program));
    }

    gl.useProgram(program);

    // ---------- Dados dos vértices (posição + cor) ----------
    //        x,    y,    r,   g,   b
    const vertices = new Float32Array([
       0.0,  0.5,  1.0, 0.0, 0.0,  // Topo (vermelho)
      -0.5, -0.5,  0.0, 1.0, 0.0,  // Esquerda (verde)
       0.5, -0.5,  0.0, 0.0, 1.0,  // Direita (azul)
    ]);

    // ---------- Criação e preenchimento do buffer ----------
    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

    // ---------- Vinculação de atributos ----------
    const FLOAT_SIZE = Float32Array.BYTES_PER_ELEMENT; // 4 bytes
    const stride = 5 * FLOAT_SIZE; // 5 floats por vértice

    const aPosition = gl.getAttribLocation(program, 'aPosition');
    gl.enableVertexAttribArray(aPosition);
    gl.vertexAttribPointer(aPosition, 2, gl.FLOAT, false, stride, 0);

    const aColor = gl.getAttribLocation(program, 'aColor');
    gl.enableVertexAttribArray(aColor);
    gl.vertexAttribPointer(aColor, 3, gl.FLOAT, false, stride, 2 * FLOAT_SIZE);

    // ---------- Renderização ----------
    gl.clearColor(0.1, 0.1, 0.15, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLES, 0, 3);
  </script>
</body>
</html>
```

**Usando Uniforms:**

```javascript
// Uniforms são valores constantes durante um draw call
const uTime = gl.getUniformLocation(program, 'uTime');
gl.uniform1f(uTime, performance.now() / 1000);

const uResolution = gl.getUniformLocation(program, 'uResolution');
gl.uniform2f(uResolution, canvas.width, canvas.height);

// Para matrizes 4x4 (transformações 3D)
const uMatrix = gl.getUniformLocation(program, 'uModelViewProjection');
gl.uniformMatrix4fv(uMatrix, false, matrixArray); // Float32Array de 16 elementos
```

---

### Texturas

Texturas permitem mapear imagens sobre geometrias, adicionando detalhes visuais sem aumentar a complexidade geométrica.

```javascript
function loadTexture(gl, url) {
  const texture = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, texture);

  // Pixel temporário enquanto a imagem carrega
  const pixel = new Uint8Array([128, 128, 128, 255]);
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0,
                gl.RGBA, gl.UNSIGNED_BYTE, pixel);

  const image = new Image();
  image.crossOrigin = 'anonymous';
  image.onload = () => {
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,
                  gl.UNSIGNED_BYTE, image);

    // Verifica se as dimensões são potência de 2
    if (isPowerOf2(image.width) && isPowerOf2(image.height)) {
      gl.generateMipmap(gl.TEXTURE_2D);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER,
                       gl.LINEAR_MIPMAP_LINEAR);
    } else {
      // NPOT (Non-Power-Of-Two): restrições adicionais no WebGL 1
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
    }
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
  };
  image.src = url;

  return texture;
}

function isPowerOf2(value) {
  return (value & (value - 1)) === 0 && value !== 0;
}
```

**Parâmetros de textura:**

| Parâmetro | Valores | Descrição |
|-----------|---------|-----------|
| `TEXTURE_MIN_FILTER` | `LINEAR`, `NEAREST`, `LINEAR_MIPMAP_LINEAR` | Filtro ao reduzir textura |
| `TEXTURE_MAG_FILTER` | `LINEAR`, `NEAREST` | Filtro ao ampliar textura |
| `TEXTURE_WRAP_S` | `REPEAT`, `CLAMP_TO_EDGE`, `MIRRORED_REPEAT` | Repetição horizontal |
| `TEXTURE_WRAP_T` | `REPEAT`, `CLAMP_TO_EDGE`, `MIRRORED_REPEAT` | Repetição vertical |

**Exemplo — Quad texturizado:**

```javascript
// Vertex shader para textura
const vsTexture = `
  attribute vec2 aPosition;
  attribute vec2 aTexCoord;
  varying vec2 vTexCoord;
  void main() {
      vTexCoord = aTexCoord;
      gl_Position = vec4(aPosition, 0.0, 1.0);
  }
`;

// Fragment shader para textura
const fsTexture = `
  precision mediump float;
  varying vec2 vTexCoord;
  uniform sampler2D uSampler;
  void main() {
      gl_FragColor = texture2D(uSampler, vTexCoord);
  }
`;

// Dados do quad (2 triângulos) com coordenadas de textura
const quadVertices = new Float32Array([
  // posição (x,y)  texCoord (u,v)
  -0.5, -0.5,       0.0, 0.0,
   0.5, -0.5,       1.0, 0.0,
   0.5,  0.5,       1.0, 1.0,
  -0.5, -0.5,       0.0, 0.0,
   0.5,  0.5,       1.0, 1.0,
  -0.5,  0.5,       0.0, 1.0,
]);

// Ativar textura na unidade 0
gl.activeTexture(gl.TEXTURE0);
gl.bindTexture(gl.TEXTURE_2D, texture);
gl.uniform1i(gl.getUniformLocation(program, 'uSampler'), 0);

gl.drawArrays(gl.TRIANGLES, 0, 6);
```

---

### Transformações 3D

Gráficos 3D utilizam três matrizes fundamentais para projetar objetos tridimensionais na tela 2D:

| Matriz | Descrição |
|--------|-----------|
| **Model** | Posiciona e orienta o objeto no mundo (translação, rotação, escala) |
| **View** | Posiciona a câmera (inversa da transformação da câmera) |
| **Projection** | Define a projeção (perspectiva ou ortográfica) |

A posição final de cada vértice é: `gl_Position = Projection × View × Model × vertexPosition`

**Usando a biblioteca gl-matrix:**

```html
<!-- CDN do gl-matrix -->
<script src="https://cdn.jsdelivr.net/npm/gl-matrix@3.4.3/gl-matrix-min.js"></script>

<script>
  const { mat4, vec3 } = glMatrix;

  // Matriz de projeção perspectiva
  const projection = mat4.create();
  mat4.perspective(projection,
    45 * Math.PI / 180,   // campo de visão (FOV) em radianos
    canvas.width / canvas.height, // aspect ratio
    0.1,                  // plano near
    100.0                 // plano far
  );

  // Matriz de visualização (câmera)
  const view = mat4.create();
  mat4.lookAt(view,
    vec3.fromValues(0, 0, 5),  // posição da câmera
    vec3.fromValues(0, 0, 0),  // ponto alvo
    vec3.fromValues(0, 1, 0)   // vetor "para cima"
  );

  // Matriz do modelo (rotação animada)
  const model = mat4.create();
  let angle = 0;

  function render() {
    angle += 0.01;

    mat4.identity(model);
    mat4.rotateY(model, model, angle);
    mat4.rotateX(model, model, angle * 0.5);

    // Envia matrizes para os uniforms
    gl.uniformMatrix4fv(uProjection, false, projection);
    gl.uniformMatrix4fv(uView, false, view);
    gl.uniformMatrix4fv(uModel, false, model);

    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    gl.drawElements(gl.TRIANGLES, indexCount, gl.UNSIGNED_SHORT, 0);

    requestAnimationFrame(render);
  }

  render();
</script>
```

**Cubo 3D rotativo (dados de vértices):**

```javascript
// Vértices de um cubo (posição x, y, z)
const cubeVertices = new Float32Array([
  // Face frontal
  -1, -1,  1,    1, -1,  1,    1,  1,  1,   -1,  1,  1,
  // Face traseira
  -1, -1, -1,   -1,  1, -1,    1,  1, -1,    1, -1, -1,
  // Face superior
  -1,  1, -1,   -1,  1,  1,    1,  1,  1,    1,  1, -1,
  // Face inferior
  -1, -1, -1,    1, -1, -1,    1, -1,  1,   -1, -1,  1,
  // Face direita
   1, -1, -1,    1,  1, -1,    1,  1,  1,    1, -1,  1,
  // Face esquerda
  -1, -1, -1,   -1, -1,  1,   -1,  1,  1,   -1,  1, -1,
]);

// Índices (2 triângulos por face, 6 faces = 36 índices)
const cubeIndices = new Uint16Array([
   0,  1,  2,   0,  2,  3,  // frontal
   4,  5,  6,   4,  6,  7,  // traseira
   8,  9, 10,   8, 10, 11,  // superior
  12, 13, 14,  12, 14, 15,  // inferior
  16, 17, 18,  16, 18, 19,  // direita
  20, 21, 22,  20, 22, 23,  // esquerda
]);

// Habilitar teste de profundidade para 3D
gl.enable(gl.DEPTH_TEST);
```

---

### Iluminação Básica

A iluminação dá realismo aos objetos 3D calculando como a luz interage com as superfícies.

**Modelo Difuso (Lambert):**

A intensidade da luz é proporcional ao cosseno do ângulo entre a normal da superfície e a direção da luz.

```glsl
// No vertex shader ou fragment shader
vec3 lightDir = normalize(uLightPosition - vWorldPosition);
vec3 normal = normalize(vNormal);
float diffuse = max(dot(normal, lightDir), 0.0);
vec3 diffuseColor = uLightColor * uObjectColor * diffuse;
```

**Modelo Especular (Phong):**

Adiciona brilho reflexivo simulando superfícies polidas.

```glsl
// Componente especular (Phong)
vec3 viewDir = normalize(uCameraPosition - vWorldPosition);
vec3 reflectDir = reflect(-lightDir, normal);
float spec = pow(max(dot(viewDir, reflectDir), 0.0), uShininess);
vec3 specular = uLightColor * spec * uSpecularStrength;

// Cor final = ambiente + difuso + especular
vec3 ambient = uAmbientStrength * uLightColor;
vec3 finalColor = (ambient + diffuseColor + specular) * uObjectColor;
gl_FragColor = vec4(finalColor, 1.0);
```

**Fragment shader completo com iluminação Phong:**

```glsl
precision mediump float;

varying vec3 vNormal;
varying vec3 vWorldPosition;

uniform vec3 uLightPosition;
uniform vec3 uLightColor;
uniform vec3 uCameraPosition;
uniform vec3 uObjectColor;
uniform float uAmbientStrength;
uniform float uSpecularStrength;
uniform float uShininess;

void main() {
    // Ambiente
    vec3 ambient = uAmbientStrength * uLightColor;

    // Difuso (Lambert)
    vec3 norm = normalize(vNormal);
    vec3 lightDir = normalize(uLightPosition - vWorldPosition);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * uLightColor;

    // Especular (Phong)
    vec3 viewDir = normalize(uCameraPosition - vWorldPosition);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), uShininess);
    vec3 specular = uSpecularStrength * spec * uLightColor;

    vec3 result = (ambient + diffuse + specular) * uObjectColor;
    gl_FragColor = vec4(result, 1.0);
}
```

---

### WebGL 2

WebGL 2, baseado no OpenGL ES 3.0, traz melhorias significativas em relação ao WebGL 1:

| Recurso | WebGL 1 | WebGL 2 |
|---------|---------|---------|
| **GLSL** | ES 1.0 | ES 3.0 (`#version 300 es`) |
| **Texturas 3D** | Não | `TEXTURE_3D`, `texImage3D` |
| **Texturas de array** | Não | `TEXTURE_2D_ARRAY` |
| **Instancing** | Extensão | Nativo (`drawArraysInstanced`) |
| **VAOs** | Extensão | Nativo (`createVertexArray`) |
| **Transform Feedback** | Não | Sim (capturar saída do vertex shader) |
| **Uniform Buffer Objects** | Não | Sim (`createBuffer` + `UNIFORM_BUFFER`) |
| **Multiple Render Targets** | Extensão | Nativo |
| **Texturas NPOT** | Limitado | Suporte completo |
| **Inteiros no shader** | Parcial | Completo |
| **Query objects** | Não | Occlusion query, timer query |

**Exemplo — VAO no WebGL 2:**

```javascript
const gl = canvas.getContext('webgl2');

// Vertex Array Object encapsula o estado dos atributos
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);

// Configurar buffers e atributos (mesma API)
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.enableVertexAttribArray(posLocation);
gl.vertexAttribPointer(posLocation, 3, gl.FLOAT, false, 0, 0);

gl.bindVertexArray(null);

// Na hora de renderizar, basta vincular o VAO
gl.bindVertexArray(vao);
gl.drawArrays(gl.TRIANGLES, 0, vertexCount);
```

---

### WebGPU

WebGPU é a API de próxima geração para gráficos e computação na GPU no navegador, projetada para substituir o WebGL.

**Principais diferenças:**

| Aspecto | WebGL | WebGPU |
|---------|-------|--------|
| **Base** | OpenGL ES | Vulkan / Metal / Direct3D 12 |
| **Design** | Máquina de estados | Objetos imutáveis, command buffers |
| **Compute Shaders** | Não nativo | Suporte completo |
| **Linguagem de shader** | GLSL | WGSL (WebGPU Shading Language) |
| **Multi-thread** | Limitado | OffscreenCanvas + workers |
| **Validação** | Por chamada | Antecipada (pipeline validation) |
| **Overhead de CPU** | Alto | Baixo (menos mudanças de estado) |
| **Maturidade** | Estável, universal | Em adoção progressiva |

**Suporte atual de navegadores (2025):**

| Navegador | Suporte |
|-----------|---------|
| Chrome / Edge | Estável (Chrome 113+) |
| Firefox | Em desenvolvimento (Nightly) |
| Safari | Suporte parcial (Safari 17+) |

**Exemplo mínimo de inicialização WebGPU:**

```javascript
async function initWebGPU() {
  if (!navigator.gpu) {
    throw new Error('WebGPU não suportado neste navegador');
  }

  const adapter = await navigator.gpu.requestAdapter();
  const device = await adapter.requestDevice();
  const canvas = document.getElementById('gpuCanvas');
  const context = canvas.getContext('webgpu');

  const format = navigator.gpu.getPreferredCanvasFormat();
  context.configure({ device, format, alphaMode: 'premultiplied' });

  console.log('WebGPU inicializado com sucesso');
  return { device, context, format };
}
```

> WebGPU é significativamente mais complexo que WebGL para código direto, mas oferece desempenho muito superior. Na prática, frameworks como Three.js já abstraem ambas as APIs com o backend `WebGPURenderer`.

---

### Por que Usar Frameworks

Código WebGL puro (raw) é extremamente verboso. Tarefas simples como renderizar um cubo exigem centenas de linhas. Frameworks como **Three.js** e **Babylon.js** abstraem essa complexidade.

**Comparação — Cubo rotativo 3D:**

| Aspecto | WebGL Puro | Three.js |
|---------|-----------|----------|
| Linhas de código | ~200-300 | ~30-40 |
| Shaders | Escritos manualmente em GLSL | Gerados automaticamente |
| Matrizes | Cálculo manual (gl-matrix) | Gerenciadas pelo framework |
| Texturas | Carregamento e configuração manual | `TextureLoader` (1 linha) |
| Iluminação | Implementada no shader | `DirectionalLight`, `PointLight` |
| Interação | Raycasting manual | `Raycaster` integrado |

**Principais frameworks:**

| Framework | Foco | Destaques |
|-----------|------|-----------|
| **Three.js** | Gráficos 3D de propósito geral | Mais popular, grande comunidade, leve |
| **Babylon.js** | Jogos e experiências interativas | Editor visual, física integrada, PBR avançado |
| **PlayCanvas** | Jogos web | Editor web, motor de jogos completo |
| **A-Frame** | WebXR / Realidade Virtual | HTML declarativo, baseado em Three.js |

> Para a maioria dos projetos web, **Three.js** é a escolha recomendada por seu equilíbrio entre funcionalidades, performance e facilidade de aprendizado. As próximas seções cobrem Three.js em profundidade.

---

## 9. Three.js — Fundamentos

Three.js é a biblioteca JavaScript mais popular para criação de gráficos 3D na web. Ela abstrai a complexidade do WebGL (e WebGPU), oferecendo uma API orientada a objetos intuitiva para construir cenas 3D interativas com geometrias, materiais, luzes, câmeras e animações.

> Documentação oficial: [https://threejs.org/docs/](https://threejs.org/docs/)
> Exemplos interativos: [https://threejs.org/examples/](https://threejs.org/examples/)

---

### Arquitetura

Three.js organiza a cena 3D como um **grafo de cena** (scene graph) — uma estrutura hierárquica em árvore onde cada nó é um `Object3D`.

```
┌─────────────────────────────────────────────────────────────────┐
│                      ARQUITETURA THREE.JS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Renderer ──────► Renderiza a cena no canvas                    │
│       │                                                          │
│       ├── Scene (raiz do grafo de cena)                          │
│       │     ├── Mesh (Geometry + Material)                       │
│       │     │     ├── BoxGeometry                                │
│       │     │     └── MeshStandardMaterial                       │
│       │     ├── Mesh                                             │
│       │     │     ├── SphereGeometry                             │
│       │     │     └── MeshPhongMaterial                          │
│       │     ├── Group                                            │
│       │     │     ├── Mesh (filho)                               │
│       │     │     └── Mesh (filho)                               │
│       │     ├── AmbientLight                                     │
│       │     ├── DirectionalLight                                 │
│       │     └── PointLight                                       │
│       │                                                          │
│       └── Camera (PerspectiveCamera / OrthographicCamera)        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Conceitos fundamentais:**

| Conceito | Classe | Descrição |
|----------|--------|-----------|
| **Scene** | `THREE.Scene` | Container raiz de todos os objetos 3D |
| **Camera** | `THREE.PerspectiveCamera` | Define o ponto de vista do observador |
| **Renderer** | `THREE.WebGLRenderer` | Renderiza a cena no elemento `<canvas>` |
| **Mesh** | `THREE.Mesh` | Objeto visível = Geometry + Material |
| **Geometry** | `THREE.BufferGeometry` | Define a forma (vértices, faces, normais, UVs) |
| **Material** | `THREE.Material` | Define a aparência (cor, textura, brilho) |
| **Light** | `THREE.Light` | Fontes de luz que iluminam a cena |
| **Group** | `THREE.Group` | Agrupa objetos para transformações conjuntas |
| **Object3D** | `THREE.Object3D` | Classe base com posição, rotação e escala |

Todos os objetos herdam de `Object3D`, que possui:
- `position` (`Vector3`) — posição no espaço 3D
- `rotation` (`Euler`) — rotação em cada eixo
- `scale` (`Vector3`) — escala em cada eixo
- `children` — lista de objetos filhos
- `add()` / `remove()` — gerenciar filhos

---

### Configuração e Setup

**Instalação via npm (recomendado para projetos modernos):**

```bash
npm install three
npm install -D @types/three   # tipos TypeScript
```

**Instalação via CDN (prototipação rápida):**

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170/examples/jsm/"
  }
}
</script>
```

**Exemplo completo — Cubo rotativo:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Three.js — Cubo Rotativo</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.170/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';

    // ---------- Cena ----------
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x1a1a2e);

    // ---------- Câmera ----------
    const camera = new THREE.PerspectiveCamera(
      75,                          // FOV (graus)
      window.innerWidth / window.innerHeight, // aspect ratio
      0.1,                         // plano near
      1000                         // plano far
    );
    camera.position.z = 3;

    // ---------- Renderer ----------
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    document.body.appendChild(renderer.domElement);

    // ---------- Geometria + Material = Mesh ----------
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshStandardMaterial({
      color: 0x00aaff,
      metalness: 0.3,
      roughness: 0.5,
    });
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // ---------- Luzes ----------
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(5, 5, 5);
    scene.add(directionalLight);

    // ---------- Loop de Animação ----------
    function animate() {
      cube.rotation.x += 0.01;
      cube.rotation.y += 0.015;

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    animate();

    // ---------- Responsividade ----------
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
```

**Setup com TypeScript (projeto com bundler):**

```typescript
// src/main.ts
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const scene: THREE.Scene = new THREE.Scene();

const camera: THREE.PerspectiveCamera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 2, 5);

const renderer: THREE.WebGLRenderer = new THREE.WebGLRenderer({
  antialias: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const controls: OrbitControls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

function animate(): void {
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();
```

**Estrutura de projeto recomendada com Vite:**

```bash
# Criar projeto Three.js com Vite
npm create vite@latest meu-projeto-3d -- --template vanilla-ts
cd meu-projeto-3d
npm install three
npm install -D @types/three
```

```
meu-projeto-3d/
├── public/
│   ├── models/          # Arquivos .glb, .gltf
│   ├── textures/        # Imagens de textura
│   └── audio/           # Arquivos de áudio
├── src/
│   ├── main.ts          # Entry point
│   ├── scene.ts         # Configuração da cena
│   ├── objects/          # Objetos 3D organizados
│   │   ├── cube.ts
│   │   └── environment.ts
│   └── utils/
│       ├── loader.ts    # Loaders centralizados
│       └── resize.ts    # Handler de resize
├── index.html
├── package.json
└── tsconfig.json
```

**Opções do WebGLRenderer:**

```javascript
const renderer = new THREE.WebGLRenderer({
  antialias: true,         // suavização de bordas
  alpha: true,             // fundo transparente (integrar com HTML)
  powerPreference: 'high-performance', // preferir GPU dedicada
  stencil: false,          // desativar stencil buffer (performance)
  depth: true,             // buffer de profundidade
});

// Tone mapping — controle de gama dinâmico
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.0;

// Encoding de saída (Three.js r152+)
renderer.outputColorSpace = THREE.SRGBColorSpace;
```

---

### Geometrias Primitivas

Three.js inclui diversas geometrias prontas. Todas herdam de `BufferGeometry`.

| Geometria | Descrição | Parâmetros Principais |
|-----------|-----------|----------------------|
| `BoxGeometry` | Cubo / paralelepípedo | `width, height, depth, widthSegments...` |
| `SphereGeometry` | Esfera | `radius, widthSegments, heightSegments` |
| `CylinderGeometry` | Cilindro | `radiusTop, radiusBottom, height, radialSegments` |
| `ConeGeometry` | Cone | `radius, height, radialSegments` |
| `PlaneGeometry` | Plano retangular | `width, height, widthSegments, heightSegments` |
| `CircleGeometry` | Disco / círculo | `radius, segments` |
| `TorusGeometry` | Toro (rosquinha) | `radius, tube, radialSegments, tubularSegments` |
| `TorusKnotGeometry` | Nó toroidal | `radius, tube, tubularSegments, radialSegments, p, q` |
| `RingGeometry` | Anel | `innerRadius, outerRadius, thetaSegments` |
| `DodecahedronGeometry` | Dodecaedro | `radius, detail` |
| `IcosahedronGeometry` | Icosaedro | `radius, detail` |
| `OctahedronGeometry` | Octaedro | `radius, detail` |

**Exemplo — Múltiplas geometrias em cena:**

```javascript
import * as THREE from 'three';

// Geometrias primitivas lado a lado
const geometries = [
  { geo: new THREE.BoxGeometry(1, 1, 1), pos: [-4, 0, 0], color: 0xff6b6b },
  { geo: new THREE.SphereGeometry(0.6, 32, 32), pos: [-2, 0, 0], color: 0x4ecdc4 },
  { geo: new THREE.CylinderGeometry(0.4, 0.6, 1.2, 32), pos: [0, 0, 0], color: 0x45b7d1 },
  { geo: new THREE.ConeGeometry(0.6, 1.2, 32), pos: [2, 0, 0], color: 0xf9ca24 },
  { geo: new THREE.TorusGeometry(0.5, 0.2, 16, 48), pos: [4, 0, 0], color: 0xa55eea },
  { geo: new THREE.TorusKnotGeometry(0.4, 0.15, 100, 16), pos: [-3, 2, 0], color: 0xfd79a8 },
  { geo: new THREE.RingGeometry(0.3, 0.6, 32), pos: [-1, 2, 0], color: 0x6c5ce7 },
  { geo: new THREE.DodecahedronGeometry(0.6), pos: [1, 2, 0], color: 0x00cec9 },
  { geo: new THREE.OctahedronGeometry(0.6), pos: [3, 2, 0], color: 0xe17055 },
];

geometries.forEach(({ geo, pos, color }) => {
  const material = new THREE.MeshStandardMaterial({
    color,
    metalness: 0.3,
    roughness: 0.6,
  });
  const mesh = new THREE.Mesh(geo, material);
  mesh.position.set(...pos);
  scene.add(mesh);
});
```

**BufferGeometry personalizada:**

```javascript
// Criando geometria customizada com vértices manuais
const customGeo = new THREE.BufferGeometry();

const vertices = new Float32Array([
  // Triângulo
  -1.0, -1.0,  0.0,
   1.0, -1.0,  0.0,
   0.0,  1.0,  0.0,
]);

const colors = new Float32Array([
  1.0, 0.0, 0.0,  // vermelho
  0.0, 1.0, 0.0,  // verde
  0.0, 0.0, 1.0,  // azul
]);

customGeo.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
customGeo.setAttribute('color', new THREE.BufferAttribute(colors, 3));
customGeo.computeVertexNormals();

const customMat = new THREE.MeshBasicMaterial({ vertexColors: true });
const customMesh = new THREE.Mesh(customGeo, customMat);
scene.add(customMesh);
```

---

### Materiais

Materiais definem a aparência visual dos objetos. Three.js oferece materiais com diferentes modelos de iluminação:

| Material | Modelo de Iluminação | Performance | Realismo | Sombras |
|----------|---------------------|-------------|----------|---------|
| `MeshBasicMaterial` | Nenhum (sem luz) | Muito alta | Baixo | Não reage |
| `MeshLambertMaterial` | Lambert (difuso) | Alta | Médio | Sim |
| `MeshPhongMaterial` | Phong (especular) | Média | Médio-Alto | Sim |
| `MeshStandardMaterial` | PBR (Physically Based) | Média-Baixa | Alto | Sim |
| `MeshPhysicalMaterial` | PBR Avançado | Baixa | Muito Alto | Sim |
| `MeshToonMaterial` | Cel-shading (cartoon) | Média | Estilizado | Sim |
| `MeshNormalMaterial` | Normais como cor | Alta | Debug | Não |
| `MeshDepthMaterial` | Profundidade como cor | Alta | Debug | N/A |

**Propriedades comuns dos materiais:**

| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `color` | `Color` | Cor base do material |
| `map` | `Texture` | Textura de cor (albedo) |
| `normalMap` | `Texture` | Mapa de normais para detalhes de superfície |
| `roughness` | `float` | Rugosidade (0 = espelho, 1 = fosco) — PBR |
| `metalness` | `float` | Metalicidade (0 = dielétrico, 1 = metal) — PBR |
| `emissive` | `Color` | Cor emissiva (brilho próprio) |
| `emissiveIntensity` | `float` | Intensidade da emissão |
| `opacity` | `float` | Opacidade (0 = transparente, 1 = opaco) |
| `transparent` | `boolean` | Habilita transparência |
| `wireframe` | `boolean` | Renderiza apenas as arestas |
| `side` | `enum` | `FrontSide`, `BackSide`, `DoubleSide` |
| `flatShading` | `boolean` | Shading plano (facetado) |

**Exemplo — Vitrine de materiais:**

```javascript
import * as THREE from 'three';

const materials = [
  {
    name: 'Basic (sem luz)',
    mat: new THREE.MeshBasicMaterial({ color: 0xff6b6b }),
  },
  {
    name: 'Lambert (difuso)',
    mat: new THREE.MeshLambertMaterial({ color: 0x4ecdc4 }),
  },
  {
    name: 'Phong (especular)',
    mat: new THREE.MeshPhongMaterial({
      color: 0x45b7d1,
      shininess: 100,
      specular: 0xffffff,
    }),
  },
  {
    name: 'Standard (PBR)',
    mat: new THREE.MeshStandardMaterial({
      color: 0xf9ca24,
      roughness: 0.4,
      metalness: 0.6,
    }),
  },
  {
    name: 'Physical (PBR avançado)',
    mat: new THREE.MeshPhysicalMaterial({
      color: 0xa55eea,
      roughness: 0.2,
      metalness: 0.8,
      clearcoat: 1.0,
      clearcoatRoughness: 0.1,
    }),
  },
  {
    name: 'Toon (cartoon)',
    mat: new THREE.MeshToonMaterial({ color: 0x00cec9 }),
  },
];

const sphereGeo = new THREE.SphereGeometry(0.7, 32, 32);

materials.forEach(({ mat }, i) => {
  const mesh = new THREE.Mesh(sphereGeo, mat);
  mesh.position.x = (i - materials.length / 2) * 2;
  scene.add(mesh);
});
```

**MeshPhysicalMaterial — Propriedades avançadas:**

```javascript
// Vidro translúcido
const glassMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0,
  transmission: 1.0,   // transparência física (refração)
  thickness: 0.5,       // espessura aparente
  ior: 1.5,             // índice de refração (vidro)
  transparent: true,
});

// Verniz automotivo
const carPaintMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xcc0000,
  metalness: 0.9,
  roughness: 0.5,
  clearcoat: 1.0,           // camada de verniz
  clearcoatRoughness: 0.03, // rugosidade do verniz
});

// Tecido (sheen)
const fabricMaterial = new THREE.MeshPhysicalMaterial({
  color: 0x2255aa,
  roughness: 1.0,
  metalness: 0.0,
  sheen: 1.0,           // brilho de tecido
  sheenRoughness: 0.5,
  sheenColor: new THREE.Color(0x88ccff),
});
```

**Tabela detalhada — Comparação de materiais:**

| Característica | Basic | Lambert | Phong | Standard (PBR) | Physical |
|----------------|-------|---------|-------|----------------|----------|
| Reage a luzes | Nao | Sim | Sim | Sim | Sim |
| Reflexo especular | Nao | Nao | Sim | Sim | Sim |
| Modelo de iluminacao | Nenhum | Lambert | Blinn-Phong | Cook-Torrance | Cook-Torrance+ |
| `roughness` / `metalness` | Nao | Nao | Nao | Sim | Sim |
| `clearcoat` | Nao | Nao | Nao | Nao | Sim |
| `transmission` (vidro) | Nao | Nao | Nao | Nao | Sim |
| `sheen` (tecido) | Nao | Nao | Nao | Nao | Sim |
| Normal maps | Nao | Sim | Sim | Sim | Sim |
| Environment maps | Sim | Sim | Sim | Sim | Sim |
| Custo GPU | Muito baixo | Baixo | Medio | Medio-alto | Alto |
| Caso de uso | UI, debug, partículas | Objetos simples | Objetos com brilho | Uso geral (recomendado) | Materiais especiais |

> Para a maioria dos projetos, **MeshStandardMaterial** oferece o melhor equilíbrio entre realismo e performance. Use **MeshPhysicalMaterial** apenas para efeitos específicos como vidro, verniz ou tecido.

**Compartilhando materiais entre meshes:**

```javascript
// Criar material uma vez e reutilizar
const sharedMaterial = new THREE.MeshStandardMaterial({
  color: 0x00aaff,
  roughness: 0.5,
  metalness: 0.3,
});

// Vários meshes usando o mesmo material (economia de memória)
const mesh1 = new THREE.Mesh(new THREE.BoxGeometry(1, 1, 1), sharedMaterial);
const mesh2 = new THREE.Mesh(new THREE.SphereGeometry(0.5), sharedMaterial);
const mesh3 = new THREE.Mesh(new THREE.ConeGeometry(0.5, 1), sharedMaterial);

// Para alterar propriedade de apenas um mesh, clone o material
mesh2.material = sharedMaterial.clone();
mesh2.material.color.set(0xff0000);
```

---

### Câmeras e Controles

**PerspectiveCamera** — simula a visão humana com perspectiva (objetos distantes parecem menores):

```javascript
const camera = new THREE.PerspectiveCamera(
  75,    // fov: campo de visão vertical em graus
  window.innerWidth / window.innerHeight, // aspect: proporção largura/altura
  0.1,   // near: plano de recorte próximo
  1000   // far: plano de recorte distante
);
camera.position.set(0, 5, 10);
camera.lookAt(0, 0, 0);
```

**OrthographicCamera** — sem perspectiva (projeção paralela, útil para jogos 2D isométricos):

```javascript
const frustumSize = 10;
const aspect = window.innerWidth / window.innerHeight;

const orthoCamera = new THREE.OrthographicCamera(
  -frustumSize * aspect / 2,  // left
   frustumSize * aspect / 2,  // right
   frustumSize / 2,           // top
  -frustumSize / 2,           // bottom
  0.1,                        // near
  1000                        // far
);
orthoCamera.position.set(5, 5, 5);
orthoCamera.lookAt(0, 0, 0);
```

**OrbitControls — Controle orbital interativo:**

```javascript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const controls = new OrbitControls(camera, renderer.domElement);

// Configurações
controls.enableDamping = true;     // suavização de movimento
controls.dampingFactor = 0.05;
controls.enableZoom = true;
controls.enablePan = true;
controls.minDistance = 2;          // zoom mínimo
controls.maxDistance = 20;         // zoom máximo
controls.maxPolarAngle = Math.PI / 2; // limita rotação (não olhar por baixo)
controls.autoRotate = true;       // rotação automática
controls.autoRotateSpeed = 2.0;

// No loop de animação
function animate() {
  controls.update();  // OBRIGATÓRIO quando enableDamping = true
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

**Outros controles disponíveis:**

| Controle | Descrição | Uso Típico |
|----------|-----------|------------|
| `OrbitControls` | Orbitar, zoom e pan ao redor de um alvo | Visualizadores de produto |
| `FlyControls` | Voo livre tipo avião | Exploração de ambientes |
| `FirstPersonControls` | Primeira pessoa (WASD + mouse) | Navegação em cenários |
| `PointerLockControls` | FPS com captura de ponteiro | Jogos em primeira pessoa |
| `TrackballControls` | Trackball livre sem restrição de eixo | Modelagem 3D |
| `MapControls` | Similar ao OrbitControls, otimizado para mapas | Visualização de terrenos |

---

### Iluminação

A iluminação é essencial para dar realismo e profundidade às cenas 3D.

| Tipo de Luz | Direção | Sombras | Performance | Uso Típico |
|-------------|---------|---------|-------------|------------|
| `AmbientLight` | Uniforme | Não | Muito barata | Iluminação base |
| `DirectionalLight` | Paralela (sol) | Sim | Média | Cenas externas |
| `PointLight` | Omnidirecional | Sim | Alta | Lâmpadas, velas |
| `SpotLight` | Cone | Sim | Alta | Holofotes, lanternas |
| `HemisphereLight` | Céu + chão | Não | Barata | Iluminação ambiente natural |
| `RectAreaLight` | Área retangular | Não* | Alta | Janelas, monitores |

> * `RectAreaLight` não suporta sombras nativamente e funciona apenas com `MeshStandardMaterial` e `MeshPhysicalMaterial`.

**Exemplo — Setup de iluminação completo:**

```javascript
import * as THREE from 'three';
import { RectAreaLightHelper } from 'three/addons/helpers/RectAreaLightHelper.js';
import { RectAreaLightUniformsLib } from 'three/addons/lights/RectAreaLightUniformsLib.js';

// Inicializar uniforms para RectAreaLight
RectAreaLightUniformsLib.init();

// Luz ambiente — base uniforme
const ambient = new THREE.AmbientLight(0xffffff, 0.3);
scene.add(ambient);

// Luz hemisférica — céu azul + chão marrom
const hemisphere = new THREE.HemisphereLight(0x87ceeb, 0x8b4513, 0.4);
scene.add(hemisphere);

// Luz direcional — simula o sol
const sunLight = new THREE.DirectionalLight(0xfff4e6, 1.5);
sunLight.position.set(10, 15, 10);
sunLight.castShadow = true;
sunLight.shadow.mapSize.set(2048, 2048);
sunLight.shadow.camera.left = -10;
sunLight.shadow.camera.right = 10;
sunLight.shadow.camera.top = 10;
sunLight.shadow.camera.bottom = -10;
scene.add(sunLight);

// Helper para visualizar a direção da luz
const sunHelper = new THREE.DirectionalLightHelper(sunLight, 2);
scene.add(sunHelper);

// Luz pontual — lâmpada
const pointLight = new THREE.PointLight(0xff9500, 2, 15);
pointLight.position.set(-3, 3, 0);
pointLight.castShadow = true;
scene.add(pointLight);

const pointHelper = new THREE.PointLightHelper(pointLight, 0.5);
scene.add(pointHelper);

// Spot light — holofote
const spotLight = new THREE.SpotLight(0xffffff, 3);
spotLight.position.set(5, 8, 2);
spotLight.angle = Math.PI / 6;       // abertura do cone
spotLight.penumbra = 0.3;            // suavidade da borda
spotLight.decay = 2;                 // atenuação
spotLight.castShadow = true;
spotLight.shadow.mapSize.set(1024, 1024);
scene.add(spotLight);
scene.add(spotLight.target); // alvo do spot

const spotHelper = new THREE.SpotLightHelper(spotLight);
scene.add(spotHelper);
```

**Boas praticas de iluminacao:**

```javascript
// Esquema de iluminação de 3 pontos (padrão da fotografia/cinema)

// 1. Key Light — luz principal, mais forte
const keyLight = new THREE.DirectionalLight(0xfff4e6, 2.0);
keyLight.position.set(5, 5, 5);
keyLight.castShadow = true;
scene.add(keyLight);

// 2. Fill Light — preenche sombras do lado oposto, mais fraca
const fillLight = new THREE.DirectionalLight(0xc4d7ff, 0.8);
fillLight.position.set(-5, 3, 2);
scene.add(fillLight);

// 3. Back Light (Rim Light) — destaca contorno por trás
const backLight = new THREE.DirectionalLight(0xffffff, 1.2);
backLight.position.set(0, 5, -5);
scene.add(backLight);

// Ambiente para evitar sombras completamente pretas
const ambientLight = new THREE.AmbientLight(0x404040, 0.5);
scene.add(ambientLight);
```

> Helpers visuais (`DirectionalLightHelper`, `PointLightHelper`, `SpotLightHelper`) são essenciais durante o desenvolvimento para posicionar luzes corretamente. Remova-os em produção.

---

### Sombras

Sombras adicionam profundidade e ancoragem espacial aos objetos. O processo exige três ativações:

1. **Renderer** — habilitar o shadow map
2. **Luz** — marcar como geradora de sombras
3. **Objetos** — marcar como projetores e/ou receptores

```javascript
// 1. Habilitar no renderer
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap; // sombras suaves

// 2. Configurar a luz
const dirLight = new THREE.DirectionalLight(0xffffff, 1.5);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;

// Configuração da shadow camera (importante para qualidade)
dirLight.shadow.mapSize.width = 2048;
dirLight.shadow.mapSize.height = 2048;
dirLight.shadow.camera.near = 0.5;
dirLight.shadow.camera.far = 50;
dirLight.shadow.camera.left = -10;
dirLight.shadow.camera.right = 10;
dirLight.shadow.camera.top = 10;
dirLight.shadow.camera.bottom = -10;
dirLight.shadow.bias = -0.0001; // evita artefatos (shadow acne)
scene.add(dirLight);

// 3. Objetos
const cube = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshStandardMaterial({ color: 0x00aaff })
);
cube.castShadow = true;    // projeta sombra
cube.position.y = 1;
scene.add(cube);

const floor = new THREE.Mesh(
  new THREE.PlaneGeometry(20, 20),
  new THREE.MeshStandardMaterial({ color: 0x888888 })
);
floor.rotation.x = -Math.PI / 2;
floor.receiveShadow = true; // recebe sombra
scene.add(floor);
```

**Tipos de shadow map:**

| Tipo | Qualidade | Performance | Descrição |
|------|-----------|-------------|-----------|
| `BasicShadowMap` | Baixa | Alta | Sem filtro, sombras pixeladas |
| `PCFShadowMap` | Média | Média | Percentage-Closer Filtering |
| `PCFSoftShadowMap` | Alta | Média-Baixa | PCF com amostragem suave |
| `VSMShadowMap` | Alta | Baixa | Variance Shadow Map, bom blur |

> Para depurar a shadow camera, use `CameraHelper`:
> `scene.add(new THREE.CameraHelper(dirLight.shadow.camera));`

---

### Texturas e UV Mapping

Texturas são imagens aplicadas sobre geometrias para adicionar detalhes visuais (cor, rugosidade, normais, etc.).

**Carregando texturas:**

```javascript
const textureLoader = new THREE.TextureLoader();

// Carregamento básico
const colorMap = textureLoader.load('/textures/brick_color.jpg');
const normalMap = textureLoader.load('/textures/brick_normal.jpg');
const roughnessMap = textureLoader.load('/textures/brick_roughness.jpg');
const aoMap = textureLoader.load('/textures/brick_ao.jpg');
const displacementMap = textureLoader.load('/textures/brick_displacement.jpg');

// Configuração de repetição
colorMap.wrapS = THREE.RepeatWrapping;
colorMap.wrapT = THREE.RepeatWrapping;
colorMap.repeat.set(4, 4);

// Encoding correto para texturas de cor (sRGB)
colorMap.colorSpace = THREE.SRGBColorSpace;
```

**Tipos de mapas de textura:**

| Mapa | Propriedade | Descrição |
|------|-------------|-----------|
| Color / Albedo | `map` | Cor base da superfície |
| Normal | `normalMap` | Simula detalhes de superfície sem geometria extra |
| Roughness | `roughnessMap` | Variação de rugosidade (branco = rugoso) |
| Metalness | `metalnessMap` | Variação de metalicidade (branco = metal) |
| AO (Ambient Occlusion) | `aoMap` | Sombras suaves em cavidades |
| Displacement | `displacementMap` | Deforma a geometria real (precisa de segmentos) |
| Emissive | `emissiveMap` | Áreas que emitem luz própria |
| Environment | `envMap` | Reflexões do ambiente |
| Alpha | `alphaMap` | Transparência (branco = opaco) |

**Exemplo — Material com múltiplos mapas:**

```javascript
const wall = new THREE.Mesh(
  new THREE.PlaneGeometry(5, 5, 128, 128), // Segmentos para displacement
  new THREE.MeshStandardMaterial({
    map: colorMap,
    normalMap: normalMap,
    normalScale: new THREE.Vector2(1, 1),
    roughnessMap: roughnessMap,
    roughness: 1.0,
    aoMap: aoMap,
    aoMapIntensity: 1.0,
    displacementMap: displacementMap,
    displacementScale: 0.1,
  })
);

// AO map requer segundo conjunto de UVs
wall.geometry.setAttribute('uv2',
  new THREE.BufferAttribute(wall.geometry.attributes.uv.array, 2)
);
scene.add(wall);
```

**Environment Maps (reflexões):**

```javascript
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

// Carregando HDR environment map
const rgbeLoader = new RGBELoader();
rgbeLoader.load('/textures/environment.hdr', (envMap) => {
  envMap.mapping = THREE.EquirectangularReflectionMapping;
  scene.environment = envMap;  // iluminação baseada em imagem (IBL)
  scene.background = envMap;   // skybox

  // Material com reflexões do ambiente
  const chrome = new THREE.MeshStandardMaterial({
    color: 0xffffff,
    metalness: 1.0,
    roughness: 0.0,
    envMap: envMap,
    envMapIntensity: 1.0,
  });
});
```

**LoadingManager — Acompanhar progresso de carregamento:**

```javascript
const loadingManager = new THREE.LoadingManager();

loadingManager.onStart = (url, loaded, total) => {
  console.log(`Iniciando carregamento: ${url} (${loaded}/${total})`);
};

loadingManager.onProgress = (url, loaded, total) => {
  const progress = (loaded / total) * 100;
  console.log(`Progresso: ${progress.toFixed(0)}%`);
  // Atualizar barra de loading na UI
  document.getElementById('progress-bar').style.width = `${progress}%`;
};

loadingManager.onLoad = () => {
  console.log('Todos os recursos carregados!');
  document.getElementById('loading-screen').style.display = 'none';
};

loadingManager.onError = (url) => {
  console.error(`Erro ao carregar: ${url}`);
};

// Usar o manager em todos os loaders
const textureLoader = new THREE.TextureLoader(loadingManager);
const gltfLoader = new GLTFLoader(loadingManager);
```

**Texturas procedurais com Canvas:**

```javascript
// Criar textura a partir de um canvas 2D
function createCheckerTexture(size = 256, divisions = 8) {
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');

  const cellSize = size / divisions;
  for (let i = 0; i < divisions; i++) {
    for (let j = 0; j < divisions; j++) {
      ctx.fillStyle = (i + j) % 2 === 0 ? '#ffffff' : '#888888';
      ctx.fillRect(i * cellSize, j * cellSize, cellSize, cellSize);
    }
  }

  const texture = new THREE.CanvasTexture(canvas);
  texture.wrapS = THREE.RepeatWrapping;
  texture.wrapT = THREE.RepeatWrapping;
  return texture;
}

const checkerMaterial = new THREE.MeshStandardMaterial({
  map: createCheckerTexture(),
});
```

---

### Animações

**Loop de animação básico:**

```javascript
const clock = new THREE.Clock();

function animate() {
  const elapsed = clock.getElapsedTime();
  const delta = clock.getDelta();

  // Rotação contínua
  cube.rotation.y = elapsed * 0.5;

  // Movimento senoidal
  sphere.position.y = Math.sin(elapsed * 2) * 0.5 + 1;

  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();
```

**Animações de modelos GLTF:**

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const gltfLoader = new GLTFLoader();
let mixer; // AnimationMixer

gltfLoader.load('/models/character.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // Criar mixer para gerenciar animações
  mixer = new THREE.AnimationMixer(model);

  // Listar animações disponíveis
  console.log('Animações:', gltf.animations.map(a => a.name));

  // Reproduzir a primeira animação
  const idleAction = mixer.clipAction(gltf.animations[0]);
  idleAction.play();

  // Transição entre animações (crossfade)
  const walkAction = mixer.clipAction(gltf.animations[1]);
  walkAction.play();
  idleAction.crossFadeTo(walkAction, 0.5, true);

  // Controle de velocidade
  walkAction.timeScale = 1.5; // 1.5x mais rápido
});

// No loop de animação
const clock = new THREE.Clock();
function animate() {
  const delta = clock.getDelta();
  if (mixer) mixer.update(delta);  // Atualizar animações
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

**Integração com GSAP (GreenSock):**

```javascript
import gsap from 'gsap';

// Animar posição
gsap.to(cube.position, {
  x: 3,
  y: 2,
  duration: 2,
  ease: 'power2.inOut',
});

// Animar rotação
gsap.to(cube.rotation, {
  y: Math.PI * 2,
  duration: 3,
  repeat: -1,       // repetir infinitamente
  ease: 'none',     // velocidade constante
});

// Animar material
gsap.to(cube.material.color, {
  r: 1, g: 0, b: 0,
  duration: 1.5,
  yoyo: true,
  repeat: -1,
});

// Timeline para sequência de animações
const tl = gsap.timeline({ repeat: -1 });
tl.to(cube.position, { y: 3, duration: 1, ease: 'power2.out' })
  .to(cube.position, { y: 0, duration: 0.5, ease: 'bounce.out' })
  .to(cube.rotation, { z: Math.PI, duration: 0.8 }, '-=0.3');
```

**Carregamento de modelos 3D (GLTF/GLB):**

O formato **glTF** (GL Transmission Format) é o padrão recomendado para modelos 3D na web. O formato `.glb` é a versão binária compacta.

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

// Configurar compressão Draco (reduz tamanho de modelos em ~90%)
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/libs/draco/'); // decodificador WASM

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load(
  '/models/building.glb',
  (gltf) => {
    const model = gltf.scene;

    // Configurar sombras em todos os meshes do modelo
    model.traverse((child) => {
      if (child.isMesh) {
        child.castShadow = true;
        child.receiveShadow = true;
      }
    });

    // Ajustar escala e posição
    model.scale.setScalar(0.5);
    model.position.set(0, 0, 0);

    scene.add(model);
  },
  (progress) => {
    const percent = (progress.loaded / progress.total) * 100;
    console.log(`Carregando modelo: ${percent.toFixed(0)}%`);
  },
  (error) => {
    console.error('Erro ao carregar modelo:', error);
  }
);
```

**Formatos de modelos 3D suportados:**

| Formato | Extensão | Descrição | Recomendação |
|---------|----------|-----------|--------------|
| **glTF / GLB** | `.gltf` / `.glb` | Padrão web, compacto, eficiente | Recomendado |
| **FBX** | `.fbx` | Popular em jogos e modelagem | Bom para importação |
| **OBJ** | `.obj` | Formato legado, sem animações | Apenas geometria simples |
| **USDZ** | `.usdz` | Formato Apple (AR Quick Look) | iOS AR |
| **Draco** | `.drc` | Compressão de geometria | Usar com glTF |

> Ferramentas como **Blender** exportam diretamente para glTF/GLB. Para otimização, use o [gltf-transform](https://gltf-transform.dev/) ou o [glTF Pipeline](https://github.com/CesiumGS/gltf-pipeline) para comprimir texturas e geometria.

---

### Raycasting e Interação

Raycasting permite detectar quais objetos 3D estão sob o ponteiro do mouse, habilitando interatividade.

```javascript
import * as THREE from 'three';

const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

// Objetos interativos
const interactiveObjects = [];

function createInteractiveBox(x, y, z, color) {
  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshStandardMaterial({ color })
  );
  mesh.position.set(x, y, z);
  mesh.userData.originalColor = color;
  scene.add(mesh);
  interactiveObjects.push(mesh);
  return mesh;
}

createInteractiveBox(-2, 0.5, 0, 0xff6b6b);
createInteractiveBox(0, 0.5, 0, 0x4ecdc4);
createInteractiveBox(2, 0.5, 0, 0xf9ca24);

// Atualizar coordenadas normalizadas do mouse
window.addEventListener('pointermove', (event) => {
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
});

// Detectar clique
window.addEventListener('click', () => {
  raycaster.setFromCamera(pointer, camera);
  const intersects = raycaster.intersectObjects(interactiveObjects);

  if (intersects.length > 0) {
    const clicked = intersects[0].object;
    console.log('Objeto clicado:', clicked.uuid);

    // Animar escala ao clicar
    gsap.to(clicked.scale, {
      x: 1.3, y: 1.3, z: 1.3,
      duration: 0.15,
      yoyo: true,
      repeat: 1,
    });
  }
});

// Hover no loop de animação
let hoveredObject = null;

function checkHover() {
  raycaster.setFromCamera(pointer, camera);
  const intersects = raycaster.intersectObjects(interactiveObjects);

  // Reset do objeto anterior
  if (hoveredObject) {
    hoveredObject.material.emissive.setHex(0x000000);
    document.body.style.cursor = 'default';
    hoveredObject = null;
  }

  if (intersects.length > 0) {
    hoveredObject = intersects[0].object;
    hoveredObject.material.emissive.setHex(0x333333);
    document.body.style.cursor = 'pointer';
  }
}

// Chamar checkHover() dentro do loop animate()
```

**Informações do intersect:**

```javascript
raycaster.setFromCamera(pointer, camera);
const intersects = raycaster.intersectObjects(scene.children, true); // true = recursivo

if (intersects.length > 0) {
  const hit = intersects[0];

  console.log('Objeto:', hit.object.name);
  console.log('Distância:', hit.distance);
  console.log('Ponto de impacto:', hit.point);     // Vector3
  console.log('Normal da face:', hit.face.normal);  // Vector3
  console.log('Coordenada UV:', hit.uv);            // Vector2
  console.log('Índice da face:', hit.faceIndex);
}
```

> O segundo parâmetro `true` em `intersectObjects` faz a verificação recursiva em todos os filhos. Isso é essencial ao trabalhar com modelos GLTF, que possuem hierarquias profundas de objetos.

---

### Fog e Ambientes

Fog (névoa) simula profundidade atmosférica, fazendo objetos distantes se misturarem com a cor de fundo.

```javascript
// Fog linear — transição gradual entre near e far
scene.fog = new THREE.Fog(
  0xcccccc, // cor da névoa
  10,       // distância onde a névoa começa
  50        // distância onde a névoa é total
);
scene.background = new THREE.Color(0xcccccc); // combinar com a cor da névoa

// Fog exponencial — mais natural
scene.fog = new THREE.FogExp2(0xcccccc, 0.02); // cor, densidade
```

**Skybox com CubeTextureLoader:**

```javascript
const cubeTextureLoader = new THREE.CubeTextureLoader();
const skyboxTexture = cubeTextureLoader.load([
  '/textures/skybox/px.jpg', // positivo X (direita)
  '/textures/skybox/nx.jpg', // negativo X (esquerda)
  '/textures/skybox/py.jpg', // positivo Y (cima)
  '/textures/skybox/ny.jpg', // negativo Y (baixo)
  '/textures/skybox/pz.jpg', // positivo Z (frente)
  '/textures/skybox/nz.jpg', // negativo Z (trás)
]);

scene.background = skyboxTexture;
scene.environment = skyboxTexture; // reflexões nos materiais PBR
```

---

### Responsividade

Para que a cena 3D se adapte a diferentes tamanhos de tela e dispositivos:

```javascript
function handleResize() {
  // Atualizar dimensões
  const width = window.innerWidth;
  const height = window.innerHeight;

  // Atualizar câmera
  camera.aspect = width / height;
  camera.updateProjectionMatrix();

  // Atualizar renderer
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
}

window.addEventListener('resize', handleResize);

// Também tratar orientação em dispositivos móveis
window.addEventListener('orientationchange', () => {
  setTimeout(handleResize, 100); // delay para o navegador atualizar
});
```

> Limitar `devicePixelRatio` a 2 evita problemas de performance em telas de alta densidade (como Retina 3x) sem perda perceptível de qualidade.

---

## 10. Three.js — Tópicos Avançados

Esta seção cobre técnicas avançadas de Three.js para criar experiências 3D de alta qualidade: pós-processamento, partículas, shaders customizados, física, áudio espacial e otimização de performance.

---

### Post-processing

Efeitos de pós-processamento são aplicados após a renderização da cena, manipulando a imagem 2D resultante para adicionar efeitos visuais como bloom, blur, vinheta e correção de cor.

**Setup do EffectComposer:**

```javascript
import * as THREE from 'three';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';

// Criar o composer
const composer = new EffectComposer(renderer);

// Pass 1: renderizar a cena normalmente
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Pass 2: efeito de bloom (brilho)
const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  1.5,   // strength — intensidade do bloom
  0.4,   // radius — raio de dispersão
  0.85   // threshold — limiar de luminosidade
);
composer.addPass(bloomPass);

// Pass 3: correção de saída (tone mapping, color space)
const outputPass = new OutputPass();
composer.addPass(outputPass);

// Substituir renderer.render() por composer.render()
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  composer.render(); // em vez de renderer.render(scene, camera)
}

// Responsividade: atualizar também o composer
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  composer.setSize(window.innerWidth, window.innerHeight);
});
```

**Outros passes disponíveis:**

| Pass | Efeito | Descrição |
|------|--------|-----------|
| `RenderPass` | Renderização base | Renderiza a cena no framebuffer |
| `UnrealBloomPass` | Bloom | Brilho nas áreas mais luminosas |
| `SSAOPass` | Ambient Occlusion | Sombras suaves em cantos e cavidades |
| `OutlinePass` | Contorno | Destaca objetos com borda |
| `SMAAPass` | Anti-aliasing | Suavização de bordas em pós-processamento |
| `BokehPass` | Profundidade de campo | Desfoque baseado na distância (DoF) |
| `FilmPass` | Ruído e scanlines | Efeito de filme antigo |
| `GlitchPass` | Glitch digital | Distorção aleatória |
| `ShaderPass` | Customizado | Aplica qualquer shader personalizado |

**Exemplo — SSAO + Outline:**

```javascript
import { SSAOPass } from 'three/addons/postprocessing/SSAOPass.js';
import { OutlinePass } from 'three/addons/postprocessing/OutlinePass.js';

// SSAO — sombras de oclusão ambiente em tela
const ssaoPass = new SSAOPass(scene, camera, window.innerWidth, window.innerHeight);
ssaoPass.kernelRadius = 16;
ssaoPass.minDistance = 0.005;
ssaoPass.maxDistance = 0.1;
composer.addPass(ssaoPass);

// Outline — contorno em objetos selecionados
const outlinePass = new OutlinePass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  scene,
  camera
);
outlinePass.edgeStrength = 3;
outlinePass.edgeGlow = 0.5;
outlinePass.edgeThickness = 1;
outlinePass.visibleEdgeColor.set(0x00ff00);
outlinePass.hiddenEdgeColor.set(0x004400);
composer.addPass(outlinePass);

// Selecionar objetos para destaque
outlinePass.selectedObjects = [cube, sphere];
```

**ShaderPass — Efeito customizado (vinheta + correção de cor):**

```javascript
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';

const VignetteShader = {
  uniforms: {
    tDiffuse: { value: null },  // textura de entrada (obrigatório)
    uDarkness: { value: 1.5 },
    uOffset: { value: 1.0 },
    uSaturation: { value: 1.2 },
  },
  vertexShader: `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform sampler2D tDiffuse;
    uniform float uDarkness;
    uniform float uOffset;
    uniform float uSaturation;
    varying vec2 vUv;

    void main() {
      vec4 texel = texture2D(tDiffuse, vUv);

      // Vinheta
      vec2 uv = (vUv - vec2(0.5)) * vec2(uOffset);
      float vignette = clamp(1.0 - dot(uv, uv), 0.0, 1.0);
      texel.rgb *= mix(1.0, vignette, uDarkness);

      // Saturação
      float gray = dot(texel.rgb, vec3(0.299, 0.587, 0.114));
      texel.rgb = mix(vec3(gray), texel.rgb, uSaturation);

      gl_FragColor = texel;
    }
  `,
};

const vignettePass = new ShaderPass(VignetteShader);
composer.addPass(vignettePass);
```

---

### Particle Systems e InstancedMesh

**Sistema de partículas com Points:**

Partículas são renderizadas como pontos usando `Points` com `PointsMaterial`. Ideais para efeitos como estrelas, neve, fogo e poeira.

```javascript
import * as THREE from 'three';

// Criar geometria com posições aleatórias
const particleCount = 5000;
const positions = new Float32Array(particleCount * 3);
const colors = new Float32Array(particleCount * 3);
const sizes = new Float32Array(particleCount);

for (let i = 0; i < particleCount; i++) {
  const i3 = i * 3;

  // Posições aleatórias em uma esfera
  const radius = Math.random() * 20;
  const theta = Math.random() * Math.PI * 2;
  const phi = Math.acos(2 * Math.random() - 1);

  positions[i3]     = radius * Math.sin(phi) * Math.cos(theta);
  positions[i3 + 1] = radius * Math.sin(phi) * Math.sin(theta);
  positions[i3 + 2] = radius * Math.cos(phi);

  // Cores aleatórias (tons de azul/branco)
  colors[i3]     = 0.5 + Math.random() * 0.5;
  colors[i3 + 1] = 0.5 + Math.random() * 0.5;
  colors[i3 + 2] = 1.0;

  sizes[i] = Math.random() * 3;
}

const particleGeo = new THREE.BufferGeometry();
particleGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
particleGeo.setAttribute('color', new THREE.BufferAttribute(colors, 3));
particleGeo.setAttribute('size', new THREE.BufferAttribute(sizes, 1));

// Material para partículas
const particleMat = new THREE.PointsMaterial({
  size: 0.1,
  sizeAttenuation: true, // tamanho diminui com distância
  vertexColors: true,
  transparent: true,
  opacity: 0.8,
  depthWrite: false,     // evita artefatos de ordenação
  blending: THREE.AdditiveBlending, // brilho aditivo
});

const particles = new THREE.Points(particleGeo, particleMat);
scene.add(particles);

// Animar partículas
function animateParticles(elapsed) {
  particles.rotation.y = elapsed * 0.05;

  const pos = particleGeo.attributes.position.array;
  for (let i = 1; i < pos.length; i += 3) {
    pos[i] += Math.sin(elapsed + i) * 0.001; // oscilação vertical
  }
  particleGeo.attributes.position.needsUpdate = true;
}
```

**Exemplo — Neve caindo:**

```javascript
const snowCount = 3000;
const snowPositions = new Float32Array(snowCount * 3);
const snowVelocities = new Float32Array(snowCount); // velocidade de queda

for (let i = 0; i < snowCount; i++) {
  snowPositions[i * 3]     = (Math.random() - 0.5) * 30; // x
  snowPositions[i * 3 + 1] = Math.random() * 20;          // y
  snowPositions[i * 3 + 2] = (Math.random() - 0.5) * 30; // z
  snowVelocities[i] = 0.5 + Math.random() * 1.5;          // velocidade
}

const snowGeo = new THREE.BufferGeometry();
snowGeo.setAttribute('position', new THREE.BufferAttribute(snowPositions, 3));

const snowMat = new THREE.PointsMaterial({
  color: 0xffffff,
  size: 0.08,
  sizeAttenuation: true,
  transparent: true,
  opacity: 0.9,
  depthWrite: false,
});

const snow = new THREE.Points(snowGeo, snowMat);
scene.add(snow);

function animateSnow(delta) {
  const pos = snowGeo.attributes.position.array;

  for (let i = 0; i < snowCount; i++) {
    const i3 = i * 3;
    // Queda
    pos[i3 + 1] -= snowVelocities[i] * delta;
    // Movimento lateral suave (vento)
    pos[i3] += Math.sin(pos[i3 + 1] * 0.5) * 0.002;

    // Reposicionar no topo quando chegar ao chão
    if (pos[i3 + 1] < 0) {
      pos[i3 + 1] = 20;
      pos[i3] = (Math.random() - 0.5) * 30;
      pos[i3 + 2] = (Math.random() - 0.5) * 30;
    }
  }

  snowGeo.attributes.position.needsUpdate = true;
}
```

> Para partículas com textura customizada (sprites), use uma textura com transparência no `PointsMaterial.map` e defina `alphaMap` ou `alphaTest`.

**InstancedMesh — Renderizar muitos objetos iguais:**

`InstancedMesh` renderiza milhares de cópias da mesma geometria em uma única draw call, com transformações individuais.

```javascript
import * as THREE from 'three';

const treeCount = 2000;
const treeGeometry = new THREE.ConeGeometry(0.3, 1.5, 8);
const treeMaterial = new THREE.MeshStandardMaterial({ color: 0x228b22 });

const trees = new THREE.InstancedMesh(treeGeometry, treeMaterial, treeCount);
trees.castShadow = true;

const dummy = new THREE.Object3D();
const color = new THREE.Color();

for (let i = 0; i < treeCount; i++) {
  // Posição aleatória em um terreno
  dummy.position.set(
    (Math.random() - 0.5) * 100,
    0,
    (Math.random() - 0.5) * 100
  );

  // Escala aleatória
  const scale = 0.5 + Math.random() * 1.5;
  dummy.scale.set(scale, scale, scale);

  // Rotação aleatória no eixo Y
  dummy.rotation.y = Math.random() * Math.PI * 2;

  dummy.updateMatrix();
  trees.setMatrixAt(i, dummy.matrix);

  // Cor levemente variada
  color.setHSL(0.3, 0.6, 0.2 + Math.random() * 0.2);
  trees.setColorAt(i, color);
}

trees.instanceMatrix.needsUpdate = true;
trees.instanceColor.needsUpdate = true;
scene.add(trees);
```

> `InstancedMesh` é dramaticamente mais eficiente que criar milhares de `Mesh` individuais. 2000 objetos individuais gerariam 2000 draw calls; com instancing, é apenas 1.

---

### Shaders Customizados

Para efeitos visuais que não são possíveis com materiais padrão, Three.js permite escrever shaders GLSL diretamente com `ShaderMaterial`.

```javascript
import * as THREE from 'three';

// Shader customizado: onda animada com gradiente de cor
const customVertexShader = `
  uniform float uTime;
  uniform float uAmplitude;
  varying vec2 vUv;
  varying float vElevation;

  void main() {
    vUv = uv;

    // Deformação senoidal
    float elevation = sin(position.x * 3.0 + uTime) * uAmplitude;
    elevation += sin(position.z * 2.0 + uTime * 0.8) * uAmplitude * 0.5;

    vec3 newPosition = position;
    newPosition.y += elevation;
    vElevation = elevation;

    gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
  }
`;

const customFragmentShader = `
  uniform vec3 uColorA;
  uniform vec3 uColorB;
  uniform float uTime;
  varying vec2 vUv;
  varying float vElevation;

  void main() {
    // Gradiente baseado na elevação
    float mixFactor = (vElevation + 0.5) * 0.5 + 0.25;
    vec3 color = mix(uColorA, uColorB, mixFactor);

    // Efeito pulsante
    color += 0.05 * sin(uTime * 2.0);

    gl_FragColor = vec4(color, 1.0);
  }
`;

// Criar material com shader customizado
const shaderMaterial = new THREE.ShaderMaterial({
  vertexShader: customVertexShader,
  fragmentShader: customFragmentShader,
  uniforms: {
    uTime: { value: 0.0 },
    uAmplitude: { value: 0.3 },
    uColorA: { value: new THREE.Color(0x0066ff) },
    uColorB: { value: new THREE.Color(0xff6600) },
  },
  side: THREE.DoubleSide,
});

// Plano com muitos segmentos para a deformação ser suave
const wavePlane = new THREE.Mesh(
  new THREE.PlaneGeometry(10, 10, 128, 128),
  shaderMaterial
);
wavePlane.rotation.x = -Math.PI / 2;
scene.add(wavePlane);

// Atualizar uniforms no loop de animação
const clock = new THREE.Clock();
function animate() {
  shaderMaterial.uniforms.uTime.value = clock.getElapsedTime();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

**Passando posição do mouse como uniform:**

```javascript
const mouseUniform = { value: new THREE.Vector2(0, 0) };
shaderMaterial.uniforms.uMouse = mouseUniform;

window.addEventListener('mousemove', (event) => {
  mouseUniform.value.x = event.clientX / window.innerWidth;
  mouseUniform.value.y = 1.0 - event.clientY / window.innerHeight;
});
```

> `ShaderMaterial` inclui automaticamente as variáveis `projectionMatrix`, `modelViewMatrix`, `position`, `uv` e `normal`. Para ter controle total (sem variáveis automáticas), use `RawShaderMaterial`.

---

### Física

Integrar um motor de física permite que objetos 3D respondam a gravidade, colisões e forças, criando simulações realistas.

**Integração com cannon-es:**

```bash
npm install cannon-es
```

```javascript
import * as THREE from 'three';
import * as CANNON from 'cannon-es';

// ---------- Mundo físico ----------
const world = new CANNON.World({
  gravity: new CANNON.Vec3(0, -9.82, 0), // gravidade terrestre
});
world.broadphase = new CANNON.SAPBroadphase(world);

// ---------- Chão ----------
// Three.js
const floorMesh = new THREE.Mesh(
  new THREE.PlaneGeometry(20, 20),
  new THREE.MeshStandardMaterial({ color: 0x666666 })
);
floorMesh.rotation.x = -Math.PI / 2;
floorMesh.receiveShadow = true;
scene.add(floorMesh);

// Cannon.js
const floorBody = new CANNON.Body({
  type: CANNON.Body.STATIC,
  shape: new CANNON.Plane(),
});
floorBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0);
world.addBody(floorBody);

// ---------- Caixas que caem ----------
const boxes = [];

function createPhysicsBox(x, y, z) {
  const size = 0.5 + Math.random() * 0.5;

  // Mesh Three.js
  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(size, size, size),
    new THREE.MeshStandardMaterial({
      color: new THREE.Color().setHSL(Math.random(), 0.7, 0.5),
    })
  );
  mesh.castShadow = true;
  scene.add(mesh);

  // Body Cannon.js
  const body = new CANNON.Body({
    mass: 1,
    shape: new CANNON.Box(new CANNON.Vec3(size / 2, size / 2, size / 2)),
    position: new CANNON.Vec3(x, y, z),
  });
  body.angularVelocity.set(
    Math.random() * 5,
    Math.random() * 5,
    Math.random() * 5
  );
  world.addBody(body);

  boxes.push({ mesh, body });
}

// Criar várias caixas
for (let i = 0; i < 30; i++) {
  createPhysicsBox(
    (Math.random() - 0.5) * 4,
    5 + Math.random() * 10,
    (Math.random() - 0.5) * 4
  );
}

// ---------- Sincronização no loop ----------
const clock = new THREE.Clock();
const timeStep = 1 / 60;

function animate() {
  const delta = clock.getDelta();

  // Avançar simulação física
  world.step(timeStep, delta, 3);

  // Sincronizar posição/rotação: Cannon → Three.js
  boxes.forEach(({ mesh, body }) => {
    mesh.position.copy(body.position);
    mesh.quaternion.copy(body.quaternion);
  });

  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();
```

> **Rapier** (`@dimforge/rapier3d`) é uma alternativa moderna compilada em WebAssembly, com performance superior para simulações complexas.

**Integração com Rapier:**

```bash
npm install @dimforge/rapier3d
```

```javascript
import RAPIER from '@dimforge/rapier3d';
import * as THREE from 'three';

// Rapier requer inicialização assíncrona (WebAssembly)
async function initPhysics() {
  await RAPIER.init();

  const gravity = { x: 0.0, y: -9.81, z: 0.0 };
  const world = new RAPIER.World(gravity);

  // Criar chão estático
  const groundDesc = RAPIER.RigidBodyDesc.fixed()
    .setTranslation(0, -0.5, 0);
  const groundBody = world.createRigidBody(groundDesc);
  const groundCollider = RAPIER.ColliderDesc.cuboid(25, 0.5, 25);
  world.createCollider(groundCollider, groundBody);

  // Criar cubo dinâmico
  const cubeDesc = RAPIER.RigidBodyDesc.dynamic()
    .setTranslation(0, 5, 0);
  const cubeBody = world.createRigidBody(cubeDesc);
  const cubeCollider = RAPIER.ColliderDesc.cuboid(0.5, 0.5, 0.5)
    .setRestitution(0.5)   // quicar
    .setFriction(0.8);     // atrito
  world.createCollider(cubeCollider, cubeBody);

  // Mesh Three.js correspondente
  const cubeMesh = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshStandardMaterial({ color: 0xff6b6b })
  );
  scene.add(cubeMesh);

  // Loop de sincronização
  function physicsStep() {
    world.step();
    const pos = cubeBody.translation();
    const rot = cubeBody.rotation();
    cubeMesh.position.set(pos.x, pos.y, pos.z);
    cubeMesh.quaternion.set(rot.x, rot.y, rot.z, rot.w);
  }

  return { world, physicsStep };
}
```

**Comparação — Motores de física para web:**

| Característica | cannon-es | Rapier | Ammo.js |
|----------------|-----------|--------|---------|
| Linguagem | JavaScript | Rust (WASM) | C++ (WASM) |
| Performance | Boa | Excelente | Excelente |
| Tamanho | ~150 KB | ~300 KB | ~500 KB |
| API | Simples | Moderna | Complexa (port do Bullet) |
| Inicialização | Síncrona | Assíncrona (WASM) | Assíncrona (WASM) |
| Documentação | Boa | Boa | Limitada |
| Comunidade | Grande | Crescente | Média |

---

### Audio 3D

Three.js suporta áudio posicional usando a Web Audio API, permitindo que sons variem de intensidade e posição conforme a câmera se move pela cena.

```javascript
import * as THREE from 'three';

// Listener vinculado à câmera (os "ouvidos" do usuário)
const listener = new THREE.AudioListener();
camera.add(listener);

// ---------- Áudio global (música de fundo) ----------
const bgMusic = new THREE.Audio(listener);
const audioLoader = new THREE.AudioLoader();

audioLoader.load('/audio/ambient.mp3', (buffer) => {
  bgMusic.setBuffer(buffer);
  bgMusic.setLoop(true);
  bgMusic.setVolume(0.3);
  // bgMusic.play(); // precisa de interação do usuário primeiro
});

// Iniciar áudio após interação (exigência dos navegadores)
document.addEventListener('click', () => {
  if (!bgMusic.isPlaying) bgMusic.play();
}, { once: true });

// ---------- Áudio posicional (som 3D) ----------
const positionalSound = new THREE.PositionalAudio(listener);

audioLoader.load('/audio/engine.mp3', (buffer) => {
  positionalSound.setBuffer(buffer);
  positionalSound.setLoop(true);
  positionalSound.setRefDistance(5);   // distância de referência
  positionalSound.setRolloffFactor(2); // quão rápido atenua com distância
  positionalSound.setVolume(1.0);
});

// Vincular a um objeto 3D (o som "vem" desse objeto)
const soundSource = new THREE.Mesh(
  new THREE.SphereGeometry(0.3),
  new THREE.MeshStandardMaterial({ color: 0xff0000, emissive: 0xff0000 })
);
soundSource.position.set(5, 1, 0);
soundSource.add(positionalSound); // som acompanha o objeto
scene.add(soundSource);
```

> O `PositionalAudio` usa HRTF (Head-Related Transfer Function) do navegador para criar a sensação de som tridimensional. A distância e direção do som em relação à câmera afetam volume e panning.

---

### Performance

Otimizar a performance é crucial para manter a fluidez (60 FPS) em cenas 3D complexas.

**LOD (Level of Detail):**

```javascript
const lod = new THREE.LOD();

// Geometria detalhada (perto)
const highDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 64, 64),
  new THREE.MeshStandardMaterial({ color: 0x00aaff })
);
lod.addLevel(highDetail, 0); // distância 0 (perto)

// Geometria média
const mediumDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 16, 16),
  new THREE.MeshStandardMaterial({ color: 0x00aaff })
);
lod.addLevel(mediumDetail, 15); // a partir de 15 unidades

// Geometria simplificada (longe)
const lowDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 6, 6),
  new THREE.MeshStandardMaterial({ color: 0x00aaff })
);
lod.addLevel(lowDetail, 30); // a partir de 30 unidades

scene.add(lod);
```

**Descarte de recursos:**

```javascript
// SEMPRE descartar recursos que não são mais necessários
function disposeObject(obj) {
  if (obj.geometry) obj.geometry.dispose();

  if (obj.material) {
    if (Array.isArray(obj.material)) {
      obj.material.forEach(disposeMaterial);
    } else {
      disposeMaterial(obj.material);
    }
  }
}

function disposeMaterial(material) {
  // Descartar todas as texturas do material
  for (const key of Object.keys(material)) {
    const value = material[key];
    if (value && typeof value.dispose === 'function') {
      value.dispose(); // textura
    }
  }
  material.dispose();
}

// Ao remover da cena
scene.remove(mesh);
disposeObject(mesh);
```

**Tabela — Problemas comuns de performance e soluções:**

| Problema | Sintoma | Solução |
|----------|---------|---------|
| Muitas draw calls | FPS baixo com muitos objetos | `InstancedMesh`, merge geometries |
| Texturas grandes | Uso alto de VRAM, carregamento lento | Compressão KTX2/Basis, reduzir resolução |
| Geometria complexa | FPS baixo ao rotacionar | LOD, reduzir polígonos, decimation |
| Shadow maps | FPS cai ao habilitar sombras | Reduzir `shadow.mapSize`, limitar luzes com sombra |
| Post-processing | FPS baixo com efeitos | Reduzir resolução dos passes, limitar efeitos |
| Memory leaks | Uso de memória crescente | `dispose()` em geometrias, materiais e texturas |
| Transparência | Artefatos de ordenação | `depthWrite: false`, ordenação manual |
| Overdraw | Muitos objetos sobrepostos | Frustum culling, occlusion culling |
| GC pauses | Stuttering periódico | Reutilizar objetos (`Vector3`, `Matrix4`), object pooling |

**Texturas comprimidas com KTX2:**

```javascript
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

const ktx2Loader = new KTX2Loader()
  .setTranscoderPath('/libs/basis/')  // transcodificador Basis Universal
  .detectSupport(renderer);

ktx2Loader.load('/textures/brick.ktx2', (texture) => {
  material.map = texture;
  material.needsUpdate = true;
});
```

> Texturas KTX2 com compressão Basis Universal reduzem o tamanho em 4-6x comparado a PNG/JPG e são decodificadas diretamente na GPU (sem decompressão na CPU).

**Merge de geometrias para reduzir draw calls:**

```javascript
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

// Em vez de 100 meshes separados (100 draw calls)...
const geometries = [];
const tempMatrix = new THREE.Matrix4();

for (let i = 0; i < 100; i++) {
  const geo = new THREE.BoxGeometry(1, 1, 1);
  tempMatrix.makeTranslation(
    (Math.random() - 0.5) * 20,
    0,
    (Math.random() - 0.5) * 20
  );
  geo.applyMatrix4(tempMatrix);
  geometries.push(geo);
}

// ...criar uma única geometria mesclada (1 draw call)
const mergedGeo = mergeGeometries(geometries);
const mergedMesh = new THREE.Mesh(
  mergedGeo,
  new THREE.MeshStandardMaterial({ color: 0x00aaff })
);
scene.add(mergedMesh);

// Limpar geometrias temporárias
geometries.forEach(g => g.dispose());
```

**Frustum culling — entendendo o mecanismo:**

```javascript
// Frustum culling é automático no Three.js
// Objetos fora do campo de visão da câmera não são renderizados

// Desativar para objetos que devem sempre renderizar (raro)
mesh.frustumCulled = false;

// Para objetos com geometria dinâmica, atualizar bounding sphere
mesh.geometry.computeBoundingSphere();

// Verificar manualmente se um objeto está no frustum
const frustum = new THREE.Frustum();
const projScreenMatrix = new THREE.Matrix4();
projScreenMatrix.multiplyMatrices(
  camera.projectionMatrix,
  camera.matrixWorldInverse
);
frustum.setFromProjectionMatrix(projScreenMatrix);

const isVisible = frustum.intersectsObject(mesh);
```

---

### Debug e Ferramentas

**Stats.js — Monitor de FPS:**

```javascript
import Stats from 'three/addons/libs/stats.module.js';

const stats = new Stats();
stats.showPanel(0); // 0 = FPS, 1 = ms/frame, 2 = memória
document.body.appendChild(stats.dom);

function animate() {
  stats.begin();

  // ... lógica de animação e renderização ...
  renderer.render(scene, camera);

  stats.end();
  requestAnimationFrame(animate);
}
```

**lil-gui — Painel de controles interativos:**

```javascript
import GUI from 'three/addons/libs/lil-gui.module.min.js';

const gui = new GUI();

// Controles de material
const materialFolder = gui.addFolder('Material');
materialFolder.addColor({ color: '#00aaff' }, 'color').onChange((value) => {
  cube.material.color.set(value);
});
materialFolder.add(cube.material, 'metalness', 0, 1, 0.01);
materialFolder.add(cube.material, 'roughness', 0, 1, 0.01);
materialFolder.add(cube.material, 'wireframe');

// Controles de luz
const lightFolder = gui.addFolder('Luz');
lightFolder.add(directionalLight.position, 'x', -10, 10);
lightFolder.add(directionalLight.position, 'y', 0, 20);
lightFolder.add(directionalLight.position, 'z', -10, 10);
lightFolder.add(directionalLight, 'intensity', 0, 3, 0.1);
lightFolder.addColor({ color: '#ffffff' }, 'color').onChange((value) => {
  directionalLight.color.set(value);
});

// Controles de animação
const animConfig = { speed: 1.0, paused: false };
const animFolder = gui.addFolder('Animação');
animFolder.add(animConfig, 'speed', 0, 3, 0.1).name('Velocidade');
animFolder.add(animConfig, 'paused').name('Pausar');
```

**Ferramentas complementares:**

| Ferramenta | Descrição |
|------------|-----------|
| **Stats.js** | Monitora FPS, tempo de frame e uso de memória |
| **lil-gui** | Painel para ajustar parâmetros em tempo real |
| **Three.js Editor** | Editor 3D online ([editor.threejs.org](https://editor.threejs.org)) |
| **Spector.js** | Extensão do browser para inspecionar chamadas WebGL |
| **renderer.info** | Objeto do Three.js com estatísticas de render (`calls`, `triangles`, `textures`) |

```javascript
// Acessar estatísticas de renderização do Three.js
console.log('Draw calls:', renderer.info.render.calls);
console.log('Triângulos:', renderer.info.render.triangles);
console.log('Texturas:', renderer.info.memory.textures);
console.log('Geometrias:', renderer.info.memory.geometries);
```

---

## 11. React Three Fiber (R3F)

React Three Fiber (R3F) é um renderer React para Three.js que permite construir cenas 3D de forma declarativa usando componentes JSX. Em vez de código imperativo, cada objeto 3D é um componente React que pode usar hooks, estado e todo o ecossistema React.

> Documentação oficial: [https://r3f.docs.pmnd.rs/](https://r3f.docs.pmnd.rs/)

---

### Conceitos

R3F mapeia a API do Three.js diretamente para componentes JSX:

| Three.js (imperativo) | R3F (declarativo) |
|------------------------|-------------------|
| `new THREE.Mesh()` | `<mesh>` |
| `new THREE.BoxGeometry()` | `<boxGeometry>` |
| `new THREE.MeshStandardMaterial()` | `<meshStandardMaterial>` |
| `new THREE.DirectionalLight()` | `<directionalLight>` |
| `new THREE.PerspectiveCamera()` | `<perspectiveCamera>` |
| `scene.add(mesh)` | Basta incluir como JSX filho |
| `mesh.position.set(1, 2, 3)` | `<mesh position={[1, 2, 3]}>` |
| `requestAnimationFrame(loop)` | `useFrame()` hook |

**Princípios:**

- Todo componente Three.js com construtor pode ser usado como tag JSX em camelCase
- Props correspondem às propriedades do Three.js (`position`, `rotation`, `scale`, `color`...)
- Hierarquia pai-filho no JSX = hierarquia `add()`/`remove()` no Three.js
- O loop de renderização é gerenciado automaticamente
- `dispose()` é chamado automaticamente quando componentes são desmontados

---

### Setup

```bash
npm install three @react-three/fiber
npm install -D @types/three
```

**Exemplo — Cubo rotativo em React:**

```typescript
// src/App.tsx
import { useRef } from 'react';
import { Canvas, useFrame } from '@react-three/fiber';
import * as THREE from 'three';

function SpinningCube() {
  const meshRef = useRef<THREE.Mesh>(null!);

  // useFrame é chamado a cada frame (equivalente ao loop animate)
  useFrame((state, delta) => {
    meshRef.current.rotation.x += delta * 0.5;
    meshRef.current.rotation.y += delta * 0.7;
  });

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#4ecdc4" metalness={0.3} roughness={0.5} />
    </mesh>
  );
}

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Canvas camera={{ position: [0, 0, 3], fov: 75 }}>
        {/* Luzes */}
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} intensity={1} />

        {/* Objetos */}
        <SpinningCube />
      </Canvas>
    </div>
  );
}
```

> O componente `<Canvas>` cria automaticamente o `WebGLRenderer`, a `Scene` e gerencia o loop de animação. Basta adicionar objetos como filhos.

---

### Componentes

Cada classe do Three.js é acessível como um elemento JSX. Propriedades do construtor passam pelo prop `args`.

```typescript
import { Canvas } from '@react-three/fiber';

function Scene() {
  return (
    <Canvas>
      {/* Luzes */}
      <ambientLight intensity={0.4} />
      <pointLight position={[10, 10, 10]} intensity={1.5} />

      {/* Grupo — transforma todos os filhos juntos */}
      <group position={[0, 0, 0]} rotation={[0, Math.PI / 4, 0]}>
        {/* Cubo */}
        <mesh position={[-2, 0, 0]}>
          <boxGeometry args={[1, 1, 1]} />
          <meshStandardMaterial color="tomato" />
        </mesh>

        {/* Esfera */}
        <mesh position={[0, 0, 0]}>
          <sphereGeometry args={[0.7, 32, 32]} />
          <meshStandardMaterial color="royalblue" metalness={0.8} roughness={0.2} />
        </mesh>

        {/* Toro */}
        <mesh position={[2, 0, 0]} rotation={[Math.PI / 3, 0, 0]}>
          <torusGeometry args={[0.5, 0.2, 16, 48]} />
          <meshPhysicalMaterial
            color="gold"
            clearcoat={1}
            clearcoatRoughness={0.1}
          />
        </mesh>
      </group>

      {/* Chão */}
      <mesh rotation={[-Math.PI / 2, 0, 0]} position={[0, -1, 0]}>
        <planeGeometry args={[20, 20]} />
        <meshStandardMaterial color="#444" />
      </mesh>
    </Canvas>
  );
}
```

---

### @react-three/drei

**drei** ("three" em alemão) é uma coleção de helpers e abstrações prontas para R3F, cobrindo controles, loaders, efeitos e utilitários.

```bash
npm install @react-three/drei
```

```typescript
import { Canvas } from '@react-three/fiber';
import {
  OrbitControls,
  Environment,
  Sky,
  Stars,
  Text,
  Html,
  Float,
  MeshReflectorMaterial,
  useGLTF,
  useTexture,
  Bounds,
} from '@react-three/drei';

function Scene() {
  return (
    <Canvas shadows camera={{ position: [0, 3, 8], fov: 60 }}>
      {/* Controles orbitais */}
      <OrbitControls enableDamping autoRotate autoRotateSpeed={0.5} />

      {/* Ambiente HDR para iluminação e reflexões */}
      <Environment preset="sunset" background blur={0.5} />

      {/* Ou céu procedural */}
      {/* <Sky sunPosition={[100, 20, 100]} /> */}

      {/* Estrelas de fundo */}
      <Stars radius={100} depth={50} count={3000} factor={4} fade speed={1} />

      {/* Texto 3D (usa troika-three-text) */}
      <Text
        position={[0, 3, 0]}
        fontSize={0.8}
        color="white"
        anchorX="center"
        anchorY="middle"
        font="/fonts/Roboto-Bold.woff"
      >
        Three.js + React
      </Text>

      {/* HTML incorporado no espaço 3D */}
      <Html position={[2, 1, 0]} distanceFactor={10} transform>
        <div style={{
          background: 'rgba(0,0,0,0.7)',
          color: 'white',
          padding: '10px',
          borderRadius: '8px',
        }}>
          <h3>Painel HTML</h3>
          <p>Embarcado no 3D</p>
        </div>
      </Html>

      {/* Float — animação de flutuação */}
      <Float speed={2} rotationIntensity={0.5} floatIntensity={1}>
        <mesh>
          <dodecahedronGeometry args={[0.8]} />
          <meshStandardMaterial color="#a55eea" />
        </mesh>
      </Float>

      {/* Chão com reflexão */}
      <mesh rotation={[-Math.PI / 2, 0, 0]}>
        <planeGeometry args={[50, 50]} />
        <MeshReflectorMaterial
          blur={[300, 100]}
          resolution={1024}
          mixBlur={1}
          mixStrength={40}
          roughness={1}
          depthScale={1.2}
          color="#101010"
          metalness={0.5}
        />
      </mesh>
    </Canvas>
  );
}
```

**Carregando modelos GLTF com drei:**

```typescript
import { useGLTF, useAnimations } from '@react-three/drei';
import { useEffect, useRef } from 'react';
import * as THREE from 'three';

// Pre-load para evitar delay
useGLTF.preload('/models/robot.glb');

function Robot(props: JSX.IntrinsicElements['group']) {
  const group = useRef<THREE.Group>(null!);
  const { scene, animations } = useGLTF('/models/robot.glb');
  const { actions } = useAnimations(animations, group);

  useEffect(() => {
    // Reproduzir a animação 'idle'
    actions['idle']?.play();
    return () => {
      actions['idle']?.stop();
    };
  }, [actions]);

  return <primitive ref={group} object={scene} {...props} />;
}

// Uso
<Robot position={[0, 0, 0]} scale={0.5} />
```

---

### Hooks

**useFrame — Loop de animação:**

```typescript
import { useFrame, useThree } from '@react-three/fiber';
import { useRef } from 'react';
import * as THREE from 'three';

function AnimatedSphere() {
  const ref = useRef<THREE.Mesh>(null!);

  useFrame((state, delta) => {
    // state.clock.elapsedTime — tempo total
    // delta — tempo desde o último frame
    // state.pointer — posição normalizada do mouse

    ref.current.position.y = Math.sin(state.clock.elapsedTime) * 0.5;
    ref.current.rotation.z += delta;

    // Seguir o mouse suavemente
    ref.current.position.x = THREE.MathUtils.lerp(
      ref.current.position.x,
      state.pointer.x * 3,
      0.05
    );
  });

  return (
    <mesh ref={ref}>
      <sphereGeometry args={[0.5, 32, 32]} />
      <meshStandardMaterial color="#ff6b6b" />
    </mesh>
  );
}
```

**useThree — Acesso ao estado do renderer:**

```typescript
import { useThree } from '@react-three/fiber';

function CameraInfo() {
  const { camera, gl, scene, size, viewport } = useThree();

  console.log('Tamanho do canvas:', size.width, size.height);
  console.log('Viewport (unidades 3D):', viewport.width, viewport.height);
  console.log('Renderer:', gl);
  console.log('Câmera:', camera.position);

  return null; // componente sem renderização visual
}
```

**useLoader — Carregamento de recursos:**

```typescript
import { useLoader } from '@react-three/fiber';
import { TextureLoader } from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

function TexturedBox() {
  const texture = useLoader(TextureLoader, '/textures/brick.jpg');

  return (
    <mesh>
      <boxGeometry args={[2, 2, 2]} />
      <meshStandardMaterial map={texture} />
    </mesh>
  );
}

function Model() {
  const gltf = useLoader(GLTFLoader, '/models/scene.glb');
  return <primitive object={gltf.scene} />;
}
```

---

### Interatividade

R3F embute raycasting automaticamente nos elementos JSX. Basta adicionar event handlers como em HTML:

```typescript
import { useState, useRef } from 'react';
import { Canvas, useFrame, ThreeEvent } from '@react-three/fiber';
import * as THREE from 'three';

function InteractiveBox({ position }: { position: [number, number, number] }) {
  const ref = useRef<THREE.Mesh>(null!);
  const [hovered, setHovered] = useState(false);
  const [clicked, setClicked] = useState(false);

  useFrame((state, delta) => {
    // Animação suave de escala
    const targetScale = clicked ? 1.5 : 1;
    ref.current.scale.lerp(
      new THREE.Vector3(targetScale, targetScale, targetScale),
      0.1
    );
    ref.current.rotation.y += delta * (hovered ? 2 : 0.5);
  });

  return (
    <mesh
      ref={ref}
      position={position}
      onClick={(e: ThreeEvent<MouseEvent>) => {
        e.stopPropagation(); // evita propagar para objetos atrás
        setClicked(!clicked);
      }}
      onPointerOver={(e: ThreeEvent<PointerEvent>) => {
        e.stopPropagation();
        setHovered(true);
        document.body.style.cursor = 'pointer';
      }}
      onPointerOut={() => {
        setHovered(false);
        document.body.style.cursor = 'default';
      }}
    >
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial
        color={hovered ? '#ff6b6b' : '#4ecdc4'}
        emissive={hovered ? '#331111' : '#000000'}
      />
    </mesh>
  );
}

export default function App() {
  return (
    <Canvas camera={{ position: [0, 0, 6] }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[5, 5, 5]} />
      <InteractiveBox position={[-2, 0, 0]} />
      <InteractiveBox position={[0, 0, 0]} />
      <InteractiveBox position={[2, 0, 0]} />
    </Canvas>
  );
}
```

**Eventos disponíveis nos meshes R3F:**

| Evento | Descrição |
|--------|-----------|
| `onClick` | Clique no objeto |
| `onDoubleClick` | Duplo clique |
| `onPointerOver` | Mouse entra no objeto |
| `onPointerOut` | Mouse sai do objeto |
| `onPointerDown` | Botão pressionado sobre o objeto |
| `onPointerUp` | Botão solto sobre o objeto |
| `onPointerMove` | Mouse se move sobre o objeto |
| `onPointerMissed` | Clique que NÃO atingiu nenhum objeto |
| `onWheel` | Scroll do mouse sobre o objeto |

---

### Estado e Integração

**Usando Zustand para estado global:**

```bash
npm install zustand
```

```typescript
// src/store.ts
import { create } from 'zustand';

interface AppState {
  color: string;
  wireframe: boolean;
  rotationSpeed: number;
  setColor: (color: string) => void;
  toggleWireframe: () => void;
  setSpeed: (speed: number) => void;
}

export const useStore = create<AppState>((set) => ({
  color: '#4ecdc4',
  wireframe: false,
  rotationSpeed: 1,
  setColor: (color) => set({ color }),
  toggleWireframe: () => set((s) => ({ wireframe: !s.wireframe })),
  setSpeed: (rotationSpeed) => set({ rotationSpeed }),
}));
```

```typescript
// src/App.tsx — Misturando UI 2D com 3D
import { Canvas, useFrame } from '@react-three/fiber';
import { OrbitControls } from '@react-three/drei';
import { useStore } from './store';
import { useRef } from 'react';
import * as THREE from 'three';

// Componente 3D que lê o estado
function ControlledCube() {
  const ref = useRef<THREE.Mesh>(null!);
  const { color, wireframe, rotationSpeed } = useStore();

  useFrame((_, delta) => {
    ref.current.rotation.y += delta * rotationSpeed;
  });

  return (
    <mesh ref={ref}>
      <boxGeometry args={[1.5, 1.5, 1.5]} />
      <meshStandardMaterial color={color} wireframe={wireframe} />
    </mesh>
  );
}

// Painel de UI 2D (React normal)
function UIPanel() {
  const { color, setColor, wireframe, toggleWireframe, rotationSpeed, setSpeed } = useStore();

  return (
    <div style={{
      position: 'absolute', top: 20, left: 20, zIndex: 1,
      background: 'rgba(0,0,0,0.8)', color: 'white',
      padding: '16px', borderRadius: '8px', minWidth: '200px',
    }}>
      <h3 style={{ margin: '0 0 12px' }}>Controles</h3>

      <label>
        Cor:
        <input
          type="color"
          value={color}
          onChange={(e) => setColor(e.target.value)}
          style={{ marginLeft: 8 }}
        />
      </label>
      <br /><br />

      <label>
        <input
          type="checkbox"
          checked={wireframe}
          onChange={toggleWireframe}
        />
        {' '}Wireframe
      </label>
      <br /><br />

      <label>
        Velocidade: {rotationSpeed.toFixed(1)}
        <br />
        <input
          type="range"
          min="0" max="5" step="0.1"
          value={rotationSpeed}
          onChange={(e) => setSpeed(Number(e.target.value))}
          style={{ width: '100%' }}
        />
      </label>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh', position: 'relative' }}>
      {/* UI 2D sobreposta */}
      <UIPanel />

      {/* Canvas 3D */}
      <Canvas camera={{ position: [0, 0, 4] }}>
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} intensity={1.5} />
        <OrbitControls />
        <ControlledCube />
      </Canvas>
    </div>
  );
}
```

---

### Performance

**React.memo para meshes estáticos:**

```typescript
import { memo } from 'react';

// Componentes que não mudam não precisam re-renderizar
const StaticFloor = memo(function StaticFloor() {
  return (
    <mesh rotation={[-Math.PI / 2, 0, 0]} receiveShadow>
      <planeGeometry args={[50, 50]} />
      <meshStandardMaterial color="#333" />
    </mesh>
  );
});
```

**Instances do drei para muitos objetos iguais:**

```typescript
import { Instances, Instance } from '@react-three/drei';

function Forest() {
  const trees = Array.from({ length: 500 }, (_, i) => ({
    position: [
      (Math.random() - 0.5) * 50,
      0,
      (Math.random() - 0.5) * 50,
    ] as [number, number, number],
    scale: 0.5 + Math.random(),
  }));

  return (
    <Instances limit={500}>
      <coneGeometry args={[0.3, 1.5, 8]} />
      <meshStandardMaterial color="green" />
      {trees.map((tree, i) => (
        <Instance
          key={i}
          position={tree.position}
          scale={tree.scale}
        />
      ))}
    </Instances>
  );
}
```

**Suspense e lazy loading:**

```typescript
import { Suspense } from 'react';
import { Canvas } from '@react-three/fiber';
import { useGLTF } from '@react-three/drei';

function HeavyModel() {
  const { scene } = useGLTF('/models/detailed-scene.glb');
  return <primitive object={scene} />;
}

// Fallback enquanto o modelo carrega
function LoadingIndicator() {
  return (
    <mesh>
      <sphereGeometry args={[0.5, 16, 16]} />
      <meshBasicMaterial color="gray" wireframe />
    </mesh>
  );
}

export default function App() {
  return (
    <Canvas>
      <Suspense fallback={<LoadingIndicator />}>
        <HeavyModel />
      </Suspense>
    </Canvas>
  );
}
```

---

### Exemplo Pratico

Visualizador 3D de produto interativo completo com modelo GLTF, controles orbitais e painel de UI:

```typescript
// src/ProductViewer.tsx
import { Suspense, useState } from 'react';
import { Canvas } from '@react-three/fiber';
import {
  OrbitControls,
  Environment,
  ContactShadows,
  Html,
  useGLTF,
} from '@react-three/drei';

function Product({ color }: { color: string }) {
  const { scene } = useGLTF('/models/sneaker.glb');

  // Aplicar cor selecionada ao material principal
  scene.traverse((child) => {
    if ((child as any).isMesh) {
      const mesh = child as any;
      if (mesh.material.name === 'main') {
        mesh.material.color.set(color);
      }
    }
  });

  return <primitive object={scene} scale={2} />;
}

function Loader() {
  return (
    <Html center>
      <div style={{ color: 'white', fontSize: '1.2rem' }}>Carregando...</div>
    </Html>
  );
}

const COLORS = ['#e74c3c', '#3498db', '#2ecc71', '#f39c12', '#9b59b6', '#1abc9c'];

export default function ProductViewer() {
  const [selectedColor, setSelectedColor] = useState(COLORS[0]);

  return (
    <div style={{ width: '100vw', height: '100vh', position: 'relative' }}>
      {/* Painel de cores */}
      <div style={{
        position: 'absolute', bottom: 30, left: '50%',
        transform: 'translateX(-50%)', zIndex: 1,
        display: 'flex', gap: 10, padding: '12px 20px',
        background: 'rgba(0,0,0,0.6)', borderRadius: 30,
      }}>
        {COLORS.map((c) => (
          <button
            key={c}
            onClick={() => setSelectedColor(c)}
            style={{
              width: 36, height: 36, borderRadius: '50%',
              background: c, border: selectedColor === c
                ? '3px solid white' : '3px solid transparent',
              cursor: 'pointer', transition: 'border 0.2s',
            }}
          />
        ))}
      </div>

      <Canvas
        shadows
        camera={{ position: [0, 1.5, 4], fov: 45 }}
        style={{ background: '#1a1a2e' }}
      >
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} intensity={1} castShadow />

        <Suspense fallback={<Loader />}>
          <Product color={selectedColor} />
          <Environment preset="city" />
        </Suspense>

        <ContactShadows
          position={[0, -0.5, 0]}
          opacity={0.5}
          width={10}
          height={10}
          blur={2}
        />

        <OrbitControls
          enablePan={false}
          minDistance={2}
          maxDistance={8}
          minPolarAngle={Math.PI / 6}
          maxPolarAngle={Math.PI / 2}
        />
      </Canvas>
    </div>
  );
}
```

> Este padrão de visualizador de produto com seletor de cores, controles orbitais e sombras de contato é amplamente usado em e-commerce, configuradores de produtos e portfólios interativos.

---

## 12. Babylon.js — Engine 3D Alternativa

Babylon.js é uma engine 3D completa para a web, criada e mantida pela Microsoft. Diferentemente do Three.js, que segue uma abordagem modular e minimalista, o Babylon.js adota a filosofia **"batteries-included"** — oferecendo nativamente física, GUI, inspetor de cena, animações avançadas, suporte a VR/AR, sistema de partículas e muito mais, tudo integrado no mesmo ecossistema.

> Documentação oficial: [https://doc.babylonjs.com/](https://doc.babylonjs.com/)
> Playground interativo: [https://playground.babylonjs.com/](https://playground.babylonjs.com/)

---

### Arquitetura e Comparativo com Three.js

A arquitetura do Babylon.js é centrada em cinco objetos fundamentais que cooperam para produzir a renderização 3D:

```
┌──────────────────────────────────────────────────────────────────┐
│                    ARQUITETURA BABYLON.JS                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   Engine ──────► Gerencia o contexto WebGL/WebGPU e o canvas      │
│       │                                                           │
│       └── Scene (container principal)                             │
│             ├── Camera (ArcRotateCamera, FreeCamera, etc.)        │
│             ├── Light (HemisphericLight, PointLight, etc.)        │
│             ├── Mesh (criado via MeshBuilder)                     │
│             │     └── Material (StandardMaterial, PBRMaterial)    │
│             ├── Mesh                                              │
│             │     └── Material                                    │
│             ├── ParticleSystem                                    │
│             ├── GUI (AdvancedDynamicTexture)                      │
│             └── PhysicsAggregate (Havok/Cannon)                   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Objetos fundamentais:**

| Objeto | Descrição |
|--------|-----------|
| **Engine** | Abstrai o contexto WebGL/WebGPU, gerencia o loop de renderização e o redimensionamento do canvas |
| **Scene** | Container raiz que mantém todas as câmeras, luzes, meshes, materiais e sistemas |
| **Camera** | Define o ponto de vista — `ArcRotateCamera` (orbital), `FreeCamera` (FPS), `FollowCamera` |
| **Mesh** | Objeto visível criado via `MeshBuilder` com geometria e material associados |
| **Material** | Define aparência — `StandardMaterial` para uso geral, `PBRMaterial` para realismo físico |

O **scene graph** do Babylon.js funciona de forma hierárquica — cada `TransformNode` ou `Mesh` pode ter filhos, e as transformações (posição, rotação, escala) são herdadas. Isso permite criar grupos complexos de objetos que se movem de forma coordenada:

```javascript
// Hierarquia pai-filho no scene graph
const pai = new BABYLON.TransformNode("pai", scene);
pai.position.y = 2;

const filho = BABYLON.MeshBuilder.CreateBox("filho", { size: 1 }, scene);
filho.parent = pai; // herda transformações do pai

const neto = BABYLON.MeshBuilder.CreateSphere("neto", { diameter: 0.5 }, scene);
neto.parent = filho; // herda transformações do filho e do pai
```

**Comparativo Three.js vs Babylon.js:**

| Critério | Three.js | Babylon.js |
|----------|----------|------------|
| **Filosofia** | Modular — importe apenas o necessário | Batteries-included — tudo integrado |
| **Estilo de API** | Funcional/OOP leve | OOP robusto, estilo enterprise |
| **Física integrada** | Não (requer Cannon.js, Rapier, etc.) | Sim (Havok, Cannon, Oimo nativos) |
| **GUI nativa** | Não (requer libs externas) | Sim (GUI 2D e 3D integrada) |
| **Inspetor/Debug** | Não tem nativo | Inspector embutido completo |
| **Suporte TypeScript** | Tipagens via @types/three | Escrito em TypeScript nativamente |
| **Tamanho do bundle** | ~150KB (min+gzip, core) | ~300KB (min+gzip, core) |
| **Comunidade** | Maior, mais exemplos, mais tutoriais | Menor, mas documentação oficial excelente |
| **Curva de aprendizado** | Moderada | Moderada-Alta (mais conceitos nativos) |
| **WebGPU** | Suporte experimental via WebGPURenderer | Suporte nativo robusto |
| **VR/AR (WebXR)** | Via WebXRManager | WebXR integrado com helpers nativos |
| **Playground** | Não tem oficial | Playground interativo oficial |
| **Editor visual** | Não tem oficial | Babylon.js Editor disponível |
| **Documentação** | Boa, mas dispersa em exemplos | Excelente, estruturada e com playground |

> Babylon.js é especialmente forte em projetos que precisam de **física realista**, **interfaces gráficas 3D**, **VR/AR** ou **ferramentas de depuração visual** sem depender de bibliotecas externas.

---

### Setup e Cena Básica

**Instalação via npm (recomendado para projetos modernos):**

```bash
npm install @babylonjs/core @babylonjs/loaders
# Pacotes opcionais conforme necessidade:
npm install @babylonjs/gui           # Interface gráfica 2D/3D
npm install @babylonjs/materials     # Materiais adicionais
npm install @babylonjs/inspector     # Inspetor de cena
npm install @babylonjs/havok         # Motor de física Havok
```

**Via CDN (para prototipagem rápida):**

```html
<script src="https://cdn.babylonjs.com/babylon.js"></script>
<script src="https://cdn.babylonjs.com/loaders/babylonjs.loaders.min.js"></script>
<script src="https://cdn.babylonjs.com/gui/babylon.gui.min.js"></script>
```

**Cena básica completa com CDN:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Babylon.js — Cena Básica</title>
  <style>
    html, body { margin: 0; padding: 0; overflow: hidden; width: 100%; height: 100%; }
    #renderCanvas {
      width: 100%; height: 100%;
      display: block; outline: none;
      touch-action: none; /* necessário para controles de toque */
    }
  </style>
  <script src="https://cdn.babylonjs.com/babylon.js"></script>
</head>
<body>
  <canvas id="renderCanvas"></canvas>

  <script>
    // 1. Obter referência ao canvas
    const canvas = document.getElementById('renderCanvas');

    // 2. Criar a engine (gerencia o contexto WebGL)
    const engine = new BABYLON.Engine(canvas, true, {
      preserveDrawingBuffer: true,
      stencil: true
    });

    // 3. Criar a cena
    const scene = new BABYLON.Scene(engine);
    scene.clearColor = new BABYLON.Color4(0.05, 0.05, 0.15, 1);

    // 4. Criar câmera orbital
    const camera = new BABYLON.ArcRotateCamera(
      "camera",        // nome
      Math.PI / 4,     // alpha (rotação horizontal)
      Math.PI / 3,     // beta (rotação vertical)
      8,               // radius (distância do alvo)
      BABYLON.Vector3.Zero(), // alvo
      scene
    );
    camera.attachControl(canvas, true); // habilita controle pelo mouse
    camera.lowerRadiusLimit = 3;
    camera.upperRadiusLimit = 20;

    // 5. Criar luz hemisférica (simula luz ambiente + direcional)
    const light = new BABYLON.HemisphericLight(
      "light",
      new BABYLON.Vector3(0, 1, 0), // direção "para cima"
      scene
    );
    light.intensity = 0.8;
    light.diffuse = new BABYLON.Color3(1, 1, 1);
    light.groundColor = new BABYLON.Color3(0.2, 0.2, 0.3);

    // 6. Criar meshes usando MeshBuilder
    const box = BABYLON.MeshBuilder.CreateBox("box", {
      size: 2,
      faceColors: [
        new BABYLON.Color4(1, 0, 0, 1),   // frente
        new BABYLON.Color4(0, 1, 0, 1),   // trás
        new BABYLON.Color4(0, 0, 1, 1),   // direita
        new BABYLON.Color4(1, 1, 0, 1),   // esquerda
        new BABYLON.Color4(1, 0, 1, 1),   // topo
        new BABYLON.Color4(0, 1, 1, 1),   // base
      ]
    }, scene);
    box.position.y = 1.5;

    const ground = BABYLON.MeshBuilder.CreateGround("ground", {
      width: 10,
      height: 10
    }, scene);
    const groundMat = new BABYLON.StandardMaterial("groundMat", scene);
    groundMat.diffuseColor = new BABYLON.Color3(0.3, 0.3, 0.35);
    ground.material = groundMat;

    // 7. Animação de rotação
    scene.registerBeforeRender(() => {
      box.rotation.y += 0.01;
      box.rotation.x += 0.005;
    });

    // 8. Loop de renderização
    engine.runRenderLoop(() => {
      scene.render();
    });

    // 9. Redimensionamento responsivo
    window.addEventListener('resize', () => {
      engine.resize();
    });
  </script>
</body>
</html>
```

**Setup com ES Modules e Vite:**

```javascript
// src/main.js — Projeto Vite com ES Modules
import {
  Engine,
  Scene,
  ArcRotateCamera,
  HemisphericLight,
  MeshBuilder,
  StandardMaterial,
  Color3,
  Color4,
  Vector3,
} from '@babylonjs/core';

// Função para criar a cena
function createScene(engine, canvas) {
  const scene = new Scene(engine);
  scene.clearColor = new Color4(0.02, 0.02, 0.1, 1);

  // Câmera
  const camera = new ArcRotateCamera(
    'camera', -Math.PI / 2, Math.PI / 3, 10,
    Vector3.Zero(), scene
  );
  camera.attachControl(canvas, true);

  // Luz
  const light = new HemisphericLight('light', new Vector3(0, 1, 0), scene);
  light.intensity = 0.9;

  // Esfera
  const sphere = MeshBuilder.CreateSphere('sphere', {
    diameter: 2,
    segments: 32,
  }, scene);
  sphere.position.y = 1;

  const sphereMat = new StandardMaterial('sphereMat', scene);
  sphereMat.diffuseColor = new Color3(0.2, 0.5, 1.0);
  sphereMat.specularColor = new Color3(0.8, 0.8, 0.8);
  sphere.material = sphereMat;

  // Toroide
  const torus = MeshBuilder.CreateTorus('torus', {
    diameter: 3,
    thickness: 0.5,
    tessellation: 32,
  }, scene);
  torus.position.y = 1;

  const torusMat = new StandardMaterial('torusMat', scene);
  torusMat.diffuseColor = new Color3(1.0, 0.3, 0.3);
  torus.material = torusMat;

  // Chão
  const ground = MeshBuilder.CreateGround('ground', { width: 12, height: 12 }, scene);
  const groundMat = new StandardMaterial('groundMat', scene);
  groundMat.diffuseColor = new Color3(0.25, 0.25, 0.3);
  ground.material = groundMat;

  // Animação
  scene.registerBeforeRender(() => {
    torus.rotation.x += 0.01;
    torus.rotation.z += 0.015;
    sphere.rotation.y += 0.008;
  });

  return scene;
}

// Inicialização
const canvas = document.getElementById('renderCanvas');
const engine = new Engine(canvas, true);
const scene = createScene(engine, canvas);

engine.runRenderLoop(() => scene.render());
window.addEventListener('resize', () => engine.resize());
```

```html
<!-- index.html (Vite) -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Babylon.js + Vite</title>
  <style>
    html, body { margin: 0; overflow: hidden; width: 100%; height: 100%; }
    #renderCanvas { width: 100%; height: 100%; display: block; touch-action: none; }
  </style>
</head>
<body>
  <canvas id="renderCanvas"></canvas>
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```

> No setup com Vite e ES Modules, o tree-shaking do bundler remove código não utilizado, reduzindo significativamente o tamanho final do bundle comparado ao uso da versão CDN completa.

---

### Materiais PBR

O Babylon.js oferece três níveis de materiais com renderização fisicamente correta (PBR — Physically Based Rendering):

| Material | Complexidade | Uso Recomendado |
|----------|-------------|-----------------|
| **StandardMaterial** | Baixa | Protótipos, cenas simples, objetos sem necessidade de realismo |
| **PBRMaterial** | Alta | Controle total sobre todas as propriedades PBR |
| **PBRMetallicRoughnessMaterial** | Média | Interface simplificada seguindo o padrão glTF metallic-roughness |

**Propriedades principais do PBRMaterial:**

| Propriedade | Tipo | Descrição |
|------------|------|-----------|
| `albedoColor` | `Color3` | Cor base do material |
| `albedoTexture` | `Texture` | Textura de cor base |
| `metallic` | `number` | 0.0 (dielétrico) a 1.0 (metal) |
| `roughness` | `number` | 0.0 (liso/espelhado) a 1.0 (rugoso/fosco) |
| `normalTexture` | `Texture` | Mapa de normais para detalhes de superfície |
| `emissiveColor` | `Color3` | Cor de emissão (brilho próprio) |
| `emissiveTexture` | `Texture` | Textura de emissão |
| `ambientTexture` | `Texture` | Mapa de oclusão ambiente (AO) |
| `environmentTexture` | `CubeTexture` | Textura de ambiente para reflexos |

```javascript
import {
  Engine, Scene, ArcRotateCamera, HemisphericLight, PointLight,
  MeshBuilder, PBRMaterial, PBRMetallicRoughnessMaterial,
  CubeTexture, Texture, Color3, Color4, Vector3,
} from '@babylonjs/core';

function createPBRScene(engine, canvas) {
  const scene = new Scene(engine);
  scene.clearColor = new Color4(0.05, 0.05, 0.12, 1);

  const camera = new ArcRotateCamera(
    'camera', -Math.PI / 2, Math.PI / 3, 12,
    new Vector3(0, 0.5, 0), scene
  );
  camera.attachControl(canvas, true);

  // Luz hemisférica + ponto de luz para reflexos
  const hemiLight = new HemisphericLight('hemi', new Vector3(0, 1, 0), scene);
  hemiLight.intensity = 0.3;

  const pointLight = new PointLight('point', new Vector3(3, 5, 3), scene);
  pointLight.intensity = 1.5;

  // Textura de ambiente HDR para reflexos realistas
  // O Babylon.js suporta .env (formato próprio) e .hdr
  const envTexture = CubeTexture.CreateFromPrefilteredData(
    '/textures/environment.env', scene
  );
  scene.environmentTexture = envTexture;

  // --- Esfera 1: Ouro metálico ---
  const goldSphere = MeshBuilder.CreateSphere('gold', { diameter: 2, segments: 64 }, scene);
  goldSphere.position.x = -3;
  goldSphere.position.y = 1;

  const goldMat = new PBRMaterial('goldMat', scene);
  goldMat.albedoColor = new Color3(1.0, 0.84, 0.0);
  goldMat.metallic = 1.0;
  goldMat.roughness = 0.15;
  goldSphere.material = goldMat;

  // --- Esfera 2: Borracha fosca ---
  const rubberSphere = MeshBuilder.CreateSphere('rubber', { diameter: 2, segments: 64 }, scene);
  rubberSphere.position.y = 1;

  const rubberMat = new PBRMaterial('rubberMat', scene);
  rubberMat.albedoColor = new Color3(0.9, 0.1, 0.1);
  rubberMat.metallic = 0.0;
  rubberMat.roughness = 0.95;
  rubberSphere.material = rubberMat;

  // --- Esfera 3: Vidro/cristal ---
  const glassSphere = MeshBuilder.CreateSphere('glass', { diameter: 2, segments: 64 }, scene);
  glassSphere.position.x = 3;
  glassSphere.position.y = 1;

  const glassMat = new PBRMaterial('glassMat', scene);
  glassMat.albedoColor = new Color3(0.8, 0.9, 1.0);
  glassMat.metallic = 0.0;
  glassMat.roughness = 0.05;
  glassMat.alpha = 0.4;               // Transparência
  glassMat.subSurface.isRefractionEnabled = true;
  glassMat.subSurface.indexOfRefraction = 1.5;
  glassSphere.material = glassMat;

  // --- Esfera 4: PBRMetallicRoughnessMaterial (padrão glTF) ---
  const gltfSphere = MeshBuilder.CreateSphere('gltf', { diameter: 2, segments: 64 }, scene);
  gltfSphere.position.x = 6;
  gltfSphere.position.y = 1;

  const gltfMat = new PBRMetallicRoughnessMaterial('gltfMat', scene);
  gltfMat.baseColor = new Color3(0.4, 0.6, 1.0);
  gltfMat.metallic = 0.7;
  gltfMat.roughness = 0.3;
  // Texturas no formato glTF:
  // gltfMat.baseTexture = new Texture('/textures/albedo.png', scene);
  // gltfMat.normalTexture = new Texture('/textures/normal.png', scene);
  // gltfMat.metallicRoughnessTexture = new Texture('/textures/mr.png', scene);
  gltfSphere.material = gltfMat;

  // --- Esfera 5: Material emissivo (neon) ---
  const neonSphere = MeshBuilder.CreateSphere('neon', { diameter: 2, segments: 64 }, scene);
  neonSphere.position.x = -6;
  neonSphere.position.y = 1;

  const neonMat = new PBRMaterial('neonMat', scene);
  neonMat.albedoColor = new Color3(0.0, 0.0, 0.0);
  neonMat.metallic = 0.0;
  neonMat.roughness = 0.5;
  neonMat.emissiveColor = new Color3(0.0, 1.0, 0.6);
  neonMat.emissiveIntensity = 2.0;
  neonSphere.material = neonMat;

  // Chão reflexivo
  const ground = MeshBuilder.CreateGround('ground', { width: 20, height: 20 }, scene);
  const groundMat = new PBRMaterial('groundMat', scene);
  groundMat.albedoColor = new Color3(0.15, 0.15, 0.2);
  groundMat.metallic = 0.3;
  groundMat.roughness = 0.4;
  ground.material = groundMat;

  return scene;
}
```

> Para obter reflexos realistas em materiais PBR, é essencial definir uma `environmentTexture` na cena. O Babylon.js oferece a ferramenta **Sandbox** ([sandbox.babylonjs.com](https://sandbox.babylonjs.com/)) para converter arquivos `.hdr` para o formato `.env` otimizado.

---

### Playground e Inspector

O **Babylon.js Playground** ([playground.babylonjs.com](https://playground.babylonjs.com/)) é um ambiente de desenvolvimento online que permite prototipar cenas 3D instantaneamente, compartilhar exemplos via URL e explorar centenas de demos oficiais.

O **Inspector** (inspetor de cena) é uma ferramenta de depuração integrada que permite inspecionar e modificar em tempo real todos os elementos da cena — meshes, materiais, texturas, luzes, câmeras, animações e performance.

```javascript
// Habilitando o Inspector na cena
import { Engine, Scene, ArcRotateCamera, HemisphericLight, MeshBuilder, Vector3 } from '@babylonjs/core';

// O Inspector precisa ser importado separadamente (tree-shaking)
import '@babylonjs/inspector';

function createScene(engine, canvas) {
  const scene = new Scene(engine);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 3, 10, Vector3.Zero(), scene);
  camera.attachControl(canvas, true);

  new HemisphericLight('light', new Vector3(0, 1, 0), scene);

  const box = MeshBuilder.CreateBox('box', { size: 2 }, scene);
  box.position.y = 1;

  const sphere = MeshBuilder.CreateSphere('sphere', { diameter: 1.5 }, scene);
  sphere.position = new Vector3(3, 1, 0);

  // Abrir o Inspector automaticamente
  scene.debugLayer.show({
    embedMode: true,      // incorporado na página (não abre popup)
    overlay: false,       // painel lateral, não sobrepõe a cena
    handleResize: true,   // redimensiona o canvas automaticamente
  });

  // Controle via teclado (toggle com Shift+Ctrl+Alt+I)
  window.addEventListener('keydown', (e) => {
    if (e.shiftKey && e.ctrlKey && e.altKey && e.key === 'I') {
      if (scene.debugLayer.isVisible()) {
        scene.debugLayer.hide();
      } else {
        scene.debugLayer.show({ embedMode: true });
      }
    }
  });

  return scene;
}
```

**Funcionalidades do Inspector:**

| Aba | Funcionalidade |
|-----|----------------|
| **Scene Explorer** | Árvore hierárquica de todos os nós, meshes, luzes e câmeras |
| **Properties** | Edição em tempo real de posição, rotação, escala, cor, texturas |
| **Materials** | Visualização e edição de propriedades de materiais |
| **Textures** | Preview de todas as texturas carregadas |
| **Performance** | FPS, draw calls, triângulos ativos, uso de memória GPU |
| **Tools** | Screenshot, gravação de vídeo, exportação de cena |

> O Inspector do Babylon.js é equivalente ao DevTools do navegador para o mundo 3D. Use-o para diagnosticar problemas de performance, ajustar materiais visualmente e entender a hierarquia da cena durante o desenvolvimento.

---

### GUI 3D e 2D

O Babylon.js possui um sistema de GUI (Graphical User Interface) integrado com suporte a interfaces 2D (overlay sobre a cena) e 3D (elementos no espaço 3D).

**GUI 2D — Overlay sobre a cena:**

```javascript
import { Engine, Scene, ArcRotateCamera, HemisphericLight, MeshBuilder, Vector3, Color3 } from '@babylonjs/core';
import { AdvancedDynamicTexture, Button, TextBlock, Slider, StackPanel, Control, Rectangle } from '@babylonjs/gui';

function createGUIScene(engine, canvas) {
  const scene = new Scene(engine);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 3, 10, Vector3.Zero(), scene);
  camera.attachControl(canvas, true);
  new HemisphericLight('light', new Vector3(0, 1, 0), scene);

  const sphere = MeshBuilder.CreateSphere('sphere', { diameter: 2 }, scene);
  sphere.position.y = 1;

  // Criar textura de GUI 2D (fullscreen overlay)
  const gui = AdvancedDynamicTexture.CreateFullscreenUI('gui');

  // --- Painel lateral ---
  const panel = new StackPanel('panel');
  panel.width = '220px';
  panel.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_LEFT;
  panel.verticalAlignment = Control.VERTICAL_ALIGNMENT_CENTER;
  panel.paddingLeft = '20px';
  gui.addControl(panel);

  // Título
  const title = new TextBlock('title');
  title.text = 'Controles';
  title.color = 'white';
  title.fontSize = 22;
  title.height = '40px';
  title.fontWeight = 'bold';
  panel.addControl(title);

  // Slider de escala
  const scaleLabel = new TextBlock('scaleLabel');
  scaleLabel.text = 'Escala: 1.0';
  scaleLabel.color = '#aaa';
  scaleLabel.height = '30px';
  scaleLabel.fontSize = 14;
  panel.addControl(scaleLabel);

  const scaleSlider = new Slider('scaleSlider');
  scaleSlider.minimum = 0.5;
  scaleSlider.maximum = 3;
  scaleSlider.value = 1;
  scaleSlider.height = '20px';
  scaleSlider.width = '180px';
  scaleSlider.color = '#4fc3f7';
  scaleSlider.background = '#333';
  scaleSlider.onValueChangedObservable.add((value) => {
    sphere.scaling.setAll(value);
    scaleLabel.text = `Escala: ${value.toFixed(1)}`;
  });
  panel.addControl(scaleSlider);

  // Botão de reset
  const resetBtn = Button.CreateSimpleButton('reset', 'Resetar');
  resetBtn.width = '180px';
  resetBtn.height = '40px';
  resetBtn.color = 'white';
  resetBtn.background = '#e74c3c';
  resetBtn.cornerRadius = 8;
  resetBtn.paddingTop = '10px';
  resetBtn.onPointerClickObservable.add(() => {
    sphere.scaling.setAll(1);
    scaleSlider.value = 1;
  });
  panel.addControl(resetBtn);

  // --- HUD com informações (canto superior direito) ---
  const infoRect = new Rectangle('infoRect');
  infoRect.width = '200px';
  infoRect.height = '60px';
  infoRect.cornerRadius = 10;
  infoRect.color = '#4fc3f7';
  infoRect.thickness = 2;
  infoRect.background = 'rgba(0, 0, 0, 0.6)';
  infoRect.horizontalAlignment = Control.HORIZONTAL_ALIGNMENT_RIGHT;
  infoRect.verticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
  infoRect.top = '20px';
  infoRect.left = '-20px';
  gui.addControl(infoRect);

  const fpsText = new TextBlock('fps');
  fpsText.text = 'FPS: --';
  fpsText.color = '#4fc3f7';
  fpsText.fontSize = 16;
  infoRect.addControl(fpsText);

  // Atualizar FPS
  scene.registerBeforeRender(() => {
    fpsText.text = `FPS: ${engine.getFps().toFixed(0)}`;
  });

  return scene;
}
```

**GUI 3D — Elementos no espaço 3D:**

```javascript
import { Engine, Scene, ArcRotateCamera, HemisphericLight, MeshBuilder, Vector3, Color3 } from '@babylonjs/core';
import { GUI3DManager, HolographicButton, TextBlock } from '@babylonjs/gui';

function createGUI3DScene(engine, canvas) {
  const scene = new Scene(engine);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 3, 8, Vector3.Zero(), scene);
  camera.attachControl(canvas, true);
  new HemisphericLight('light', new Vector3(0, 1, 0), scene);

  // Gerenciador de GUI 3D
  const guiManager = new GUI3DManager(scene);

  // Criar botões holográficos posicionados no espaço 3D
  const actions = [
    { name: 'Cubo', color: '#e74c3c', pos: new Vector3(-2, 2, 0) },
    { name: 'Esfera', color: '#2ecc71', pos: new Vector3(0, 2, 0) },
    { name: 'Toroide', color: '#3498db', pos: new Vector3(2, 2, 0) },
  ];

  let currentMesh = null;

  actions.forEach(({ name, color, pos }) => {
    const button = new HolographicButton(`btn-${name}`);
    guiManager.addControl(button);

    button.position = pos;
    button.scaling = new Vector3(0.5, 0.5, 0.5);

    // Texto no botão
    const text = new TextBlock();
    text.text = name;
    text.color = 'white';
    text.fontSize = 48;
    button.content = text;

    // Evento de clique
    button.onPointerClickObservable.add(() => {
      // Remover mesh anterior
      if (currentMesh) currentMesh.dispose();

      // Criar novo mesh conforme botão clicado
      switch (name) {
        case 'Cubo':
          currentMesh = MeshBuilder.CreateBox('mesh', { size: 1.5 }, scene);
          break;
        case 'Esfera':
          currentMesh = MeshBuilder.CreateSphere('mesh', { diameter: 2 }, scene);
          break;
        case 'Toroide':
          currentMesh = MeshBuilder.CreateTorus('mesh', { diameter: 2, thickness: 0.5 }, scene);
          break;
      }

      if (currentMesh) {
        const mat = new BABYLON.StandardMaterial('mat', scene);
        mat.diffuseColor = Color3.FromHexString(color);
        currentMesh.material = mat;
      }
    });
  });

  return scene;
}
```

> A GUI 3D do Babylon.js é especialmente útil para experiências VR/AR, onde interfaces tradicionais 2D não funcionam bem. Os botões holográficos respondem a gaze, controladores VR e toque.

---

### Animações e Física

**Sistema de Animação:**

O Babylon.js oferece um sistema de animação nativo completo, baseado em keyframes, com controle de interpolação e easing.

```javascript
import {
  Engine, Scene, ArcRotateCamera, HemisphericLight, MeshBuilder,
  StandardMaterial, Color3, Vector3, Animation, CubicEase, EasingFunction,
} from '@babylonjs/core';

function createAnimationScene(engine, canvas) {
  const scene = new Scene(engine);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 3, 12, Vector3.Zero(), scene);
  camera.attachControl(canvas, true);
  new HemisphericLight('light', new Vector3(0, 1, 0), scene);

  // Criar cubo para animar
  const box = MeshBuilder.CreateBox('box', { size: 1.5 }, scene);
  const mat = new StandardMaterial('mat', scene);
  mat.diffuseColor = new Color3(0.2, 0.6, 1.0);
  box.material = mat;

  // --- Animação por keyframes ---

  // Animação de posição Y (pulo)
  const jumpAnim = new Animation(
    'jump',                              // nome
    'position.y',                        // propriedade alvo
    30,                                  // FPS da animação
    Animation.ANIMATIONTYPE_FLOAT,       // tipo do valor
    Animation.ANIMATIONLOOPMODE_CYCLE    // modo de loop
  );

  jumpAnim.setKeys([
    { frame: 0,  value: 1 },
    { frame: 15, value: 4 },   // ponto alto
    { frame: 30, value: 1 },   // volta ao chão
  ]);

  // Aplicar easing (suavização)
  const ease = new CubicEase();
  ease.setEasingMode(EasingFunction.EASINGMODE_EASEINOUT);
  jumpAnim.setEasingFunction(ease);

  box.animations.push(jumpAnim);

  // Animação de rotação Y
  const rotateAnim = new Animation(
    'rotate', 'rotation.y', 30,
    Animation.ANIMATIONTYPE_FLOAT,
    Animation.ANIMATIONLOOPMODE_CYCLE
  );
  rotateAnim.setKeys([
    { frame: 0,  value: 0 },
    { frame: 60, value: Math.PI * 2 },
  ]);
  box.animations.push(rotateAnim);

  // Iniciar ambas as animações
  scene.beginAnimation(box, 0, 60, true); // (alvo, frameInicio, frameFim, loop)

  // --- Animação rápida com CreateAndStartAnimation ---
  const sphere = MeshBuilder.CreateSphere('sphere', { diameter: 1 }, scene);
  sphere.position = new Vector3(3, 1, 0);
  const sphereMat = new StandardMaterial('sphereMat', scene);
  sphereMat.diffuseColor = new Color3(1.0, 0.3, 0.3);
  sphere.material = sphereMat;

  // Animar escala X de 1 a 2 em 60 frames
  Animation.CreateAndStartAnimation(
    'scaleX', sphere, 'scaling.x',
    30, 60, 1, 2,
    Animation.ANIMATIONLOOPMODE_YOYO  // vai e volta
  );

  // Chão
  MeshBuilder.CreateGround('ground', { width: 15, height: 15 }, scene);

  return scene;
}
```

**Sistema de Física com Havok:**

O Babylon.js integra o motor de física **Havok** (o mesmo usado em jogos AAA) para simulações realistas de corpos rígidos, colisões e constraints.

```javascript
import {
  Engine, Scene, ArcRotateCamera, HemisphericLight, DirectionalLight,
  MeshBuilder, StandardMaterial, Color3, Color4, Vector3,
  PhysicsAggregate, PhysicsShapeType,
} from '@babylonjs/core';
import HavokPhysics from '@babylonjs/havok';

async function createPhysicsScene(engine, canvas) {
  const scene = new Scene(engine);
  scene.clearColor = new Color4(0.05, 0.05, 0.12, 1);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 4, 20, Vector3.Zero(), scene);
  camera.attachControl(canvas, true);

  const hemiLight = new HemisphericLight('hemi', new Vector3(0, 1, 0), scene);
  hemiLight.intensity = 0.5;
  const dirLight = new DirectionalLight('dir', new Vector3(-1, -2, -1), scene);
  dirLight.intensity = 0.8;

  // Inicializar motor de física Havok
  const havokInstance = await HavokPhysics();
  const havokPlugin = new BABYLON.HavokPlugin(true, havokInstance);
  scene.enablePhysics(new Vector3(0, -9.81, 0), havokPlugin);

  // --- Chão estático ---
  const ground = MeshBuilder.CreateGround('ground', { width: 20, height: 20 }, scene);
  const groundMat = new StandardMaterial('groundMat', scene);
  groundMat.diffuseColor = new Color3(0.3, 0.3, 0.35);
  ground.material = groundMat;

  // Agregado de física para o chão (massa 0 = estático)
  new PhysicsAggregate(
    ground,
    PhysicsShapeType.BOX,
    { mass: 0, restitution: 0.5, friction: 0.8 },
    scene
  );

  // --- Função para criar caixas dinâmicas ---
  const colors = ['#e74c3c', '#3498db', '#2ecc71', '#f39c12', '#9b59b6', '#1abc9c'];

  function spawnBox(position) {
    const box = MeshBuilder.CreateBox('box', {
      width: 0.8 + Math.random() * 0.6,
      height: 0.8 + Math.random() * 0.6,
      depth: 0.8 + Math.random() * 0.6,
    }, scene);
    box.position = position;

    const mat = new StandardMaterial('mat', scene);
    mat.diffuseColor = Color3.FromHexString(colors[Math.floor(Math.random() * colors.length)]);
    box.material = mat;

    new PhysicsAggregate(
      box,
      PhysicsShapeType.BOX,
      { mass: 1, restitution: 0.3, friction: 0.6 },
      scene
    );

    return box;
  }

  // --- Função para criar esferas dinâmicas ---
  function spawnSphere(position) {
    const sphere = MeshBuilder.CreateSphere('sphere', {
      diameter: 0.6 + Math.random() * 0.8,
      segments: 16,
    }, scene);
    sphere.position = position;

    const mat = new StandardMaterial('mat', scene);
    mat.diffuseColor = Color3.FromHexString(colors[Math.floor(Math.random() * colors.length)]);
    sphere.material = mat;

    new PhysicsAggregate(
      sphere,
      PhysicsShapeType.SPHERE,
      { mass: 1, restitution: 0.6, friction: 0.4 },
      scene
    );

    return sphere;
  }

  // Criar objetos iniciais caindo de diferentes alturas
  for (let i = 0; i < 10; i++) {
    const x = (Math.random() - 0.5) * 6;
    const y = 5 + Math.random() * 10;
    const z = (Math.random() - 0.5) * 6;

    if (Math.random() > 0.5) {
      spawnBox(new Vector3(x, y, z));
    } else {
      spawnSphere(new Vector3(x, y, z));
    }
  }

  // Clique para adicionar novos objetos
  scene.onPointerDown = (evt) => {
    if (evt.button === 0) {
      const x = (Math.random() - 0.5) * 4;
      const z = (Math.random() - 0.5) * 4;
      if (Math.random() > 0.5) {
        spawnBox(new Vector3(x, 8, z));
      } else {
        spawnSphere(new Vector3(x, 8, z));
      }
    }
  };

  return scene;
}

// Inicialização assíncrona (necessária para carregar Havok)
const canvas = document.getElementById('renderCanvas');
const engine = new Engine(canvas, true);

createPhysicsScene(engine, canvas).then((scene) => {
  engine.runRenderLoop(() => scene.render());
});

window.addEventListener('resize', () => engine.resize());
```

> O Havok é o motor de física padrão do Babylon.js a partir da versão 6. Para projetos que não precisam da precisão do Havok, o Cannon.js continua disponível como alternativa mais leve. Instale com `npm install @babylonjs/havok`.

---

### Particle Systems e Efeitos

O sistema de partículas do Babylon.js permite criar efeitos visuais como fogo, fumaça, faíscas, neve e explosões sem bibliotecas externas.

```javascript
import {
  Engine, Scene, ArcRotateCamera, HemisphericLight, PointLight,
  MeshBuilder, StandardMaterial, ParticleSystem, Texture,
  Color3, Color4, Vector3,
} from '@babylonjs/core';

function createParticleScene(engine, canvas) {
  const scene = new Scene(engine);
  scene.clearColor = new Color4(0.02, 0.02, 0.05, 1);

  const camera = new ArcRotateCamera('cam', -Math.PI / 2, Math.PI / 3, 12, new Vector3(0, 2, 0), scene);
  camera.attachControl(canvas, true);

  const hemiLight = new HemisphericLight('hemi', new Vector3(0, 1, 0), scene);
  hemiLight.intensity = 0.2;

  // Luz pontual que acompanha o fogo
  const fireLight = new PointLight('fireLight', new Vector3(0, 2, 0), scene);
  fireLight.diffuse = new Color3(1, 0.6, 0.2);
  fireLight.intensity = 3;
  fireLight.range = 15;

  // --- Efeito de Fogo ---
  const fireEmitter = MeshBuilder.CreateBox('fireEmitter', { size: 0.1 }, scene);
  fireEmitter.position.y = 0.5;
  fireEmitter.isVisible = false;

  const fireSystem = new ParticleSystem('fire', 2000, scene);

  // Textura de partícula (usar uma textura de chama ou flare)
  fireSystem.particleTexture = new Texture('/textures/flare.png', scene);

  // Emissor
  fireSystem.emitter = fireEmitter;
  fireSystem.minEmitBox = new Vector3(-0.3, 0, -0.3);
  fireSystem.maxEmitBox = new Vector3(0.3, 0, 0.3);

  // Tamanho das partículas
  fireSystem.minSize = 0.3;
  fireSystem.maxSize = 0.8;

  // Tempo de vida
  fireSystem.minLifeTime = 0.3;
  fireSystem.maxLifeTime = 0.8;

  // Taxa de emissão
  fireSystem.emitRate = 500;

  // Direção
  fireSystem.direction1 = new Vector3(-0.2, 1, -0.2);
  fireSystem.direction2 = new Vector3(0.2, 2, 0.2);

  // Velocidade
  fireSystem.minEmitPower = 1;
  fireSystem.maxEmitPower = 3;

  // Gravidade (leve, puxando para cima para simular convecção)
  fireSystem.gravity = new Vector3(0, -0.5, 0);

  // Gradiente de cor (amarelo → laranja → vermelho → transparente)
  fireSystem.addColorGradient(0.0, new Color4(1.0, 1.0, 0.3, 1.0));   // amarelo brilhante
  fireSystem.addColorGradient(0.3, new Color4(1.0, 0.5, 0.0, 0.9));   // laranja
  fireSystem.addColorGradient(0.6, new Color4(0.8, 0.1, 0.0, 0.6));   // vermelho
  fireSystem.addColorGradient(1.0, new Color4(0.2, 0.0, 0.0, 0.0));   // desaparece

  // Gradiente de tamanho
  fireSystem.addSizeGradient(0.0, 0.3);
  fireSystem.addSizeGradient(0.5, 0.6);
  fireSystem.addSizeGradient(1.0, 0.1);

  // Blending aditivo para efeito de brilho
  fireSystem.blendMode = ParticleSystem.BLENDMODE_ADD;

  fireSystem.start();

  // --- Efeito de Fumaça ---
  const smokeSystem = new ParticleSystem('smoke', 500, scene);
  smokeSystem.particleTexture = new Texture('/textures/flare.png', scene);

  smokeSystem.emitter = fireEmitter;
  smokeSystem.minEmitBox = new Vector3(-0.1, 0, -0.1);
  smokeSystem.maxEmitBox = new Vector3(0.1, 0, 0.1);

  smokeSystem.minSize = 0.5;
  smokeSystem.maxSize = 1.5;
  smokeSystem.minLifeTime = 0.8;
  smokeSystem.maxLifeTime = 2.0;
  smokeSystem.emitRate = 80;

  smokeSystem.direction1 = new Vector3(-0.3, 2, -0.3);
  smokeSystem.direction2 = new Vector3(0.3, 4, 0.3);
  smokeSystem.minEmitPower = 0.5;
  smokeSystem.maxEmitPower = 1.5;
  smokeSystem.gravity = new Vector3(0, -0.2, 0);

  // Fumaça: cinza escuro, opacidade gradual
  smokeSystem.addColorGradient(0.0, new Color4(0.3, 0.3, 0.3, 0.0));
  smokeSystem.addColorGradient(0.2, new Color4(0.2, 0.2, 0.2, 0.3));
  smokeSystem.addColorGradient(0.7, new Color4(0.15, 0.15, 0.15, 0.2));
  smokeSystem.addColorGradient(1.0, new Color4(0.1, 0.1, 0.1, 0.0));

  smokeSystem.addSizeGradient(0.0, 0.4);
  smokeSystem.addSizeGradient(1.0, 2.0);

  smokeSystem.blendMode = ParticleSystem.BLENDMODE_STANDARD;

  smokeSystem.start();

  // Variação de luz para simular cintilação do fogo
  scene.registerBeforeRender(() => {
    fireLight.intensity = 2.5 + Math.random() * 1.5;
    fireLight.position.x = Math.sin(performance.now() * 0.003) * 0.1;
  });

  // Chão
  const ground = MeshBuilder.CreateGround('ground', { width: 12, height: 12 }, scene);
  const groundMat = new StandardMaterial('groundMat', scene);
  groundMat.diffuseColor = new Color3(0.15, 0.12, 0.1);
  ground.material = groundMat;

  return scene;
}
```

> Para texturas de partícula, o Babylon.js pode usar qualquer imagem PNG com transparência. A textura padrão `flare.png` está disponível nos exemplos oficiais. Gradientes de cor e tamanho são a chave para efeitos realistas.

---

### Babylon.js vs Three.js — Quando Escolher

| Cenário | Recomendação | Motivo |
|---------|-------------|--------|
| **Projeto com física complexa** | Babylon.js | Havok integrado, sem configuração extra |
| **Interface 3D / VR UI** | Babylon.js | GUI 3D nativa com botões holográficos |
| **Experiência VR/AR completa** | Babylon.js | WebXR helpers robustos e testados |
| **Depuração visual da cena** | Babylon.js | Inspector integrado e completo |
| **Projeto leve / landing page** | Three.js | Bundle menor, setup mais rápido |
| **Maior quantidade de exemplos** | Three.js | Comunidade maior, mais tutoriais e StackOverflow |
| **Integração com React** | Three.js | React Three Fiber (R3F) é excelente |
| **Prototipagem rápida** | Babylon.js | Playground oficial online |
| **Projeto enterprise / equipe grande** | Babylon.js | TypeScript nativo, tooling mais robusto |
| **Portfólio / arte generativa** | Three.js | Mais flexível para efeitos criativos |
| **Importar modelos glTF** | Ambos | Ambos têm excelente suporte a glTF |
| **WebGPU** | Babylon.js | Suporte mais maduro e estável |

**Quando preferir Babylon.js:**
- Projetos que exigem **simulação de física** realista (jogos, simuladores, treinamento)
- Aplicações **enterprise** que se beneficiam de ferramentas integradas (inspector, editor, GUI)
- Experiências **VR/AR (WebXR)** com interação complexa (controladores, hand tracking)
- Cenas com **muitos sistemas** (física + GUI + partículas + animações) onde a integração nativa evita problemas de compatibilidade entre bibliotecas

**Quando preferir Three.js:**
- Projetos que priorizam **bundle size** mínimo
- Aplicações **React** (React Three Fiber / R3F oferece uma experiência declarativa excelente)
- Projetos com grande necessidade de **exemplos da comunidade** e referências online
- Experimentos **criativos e artísticos** onde a flexibilidade modular é mais importante
- Equipes com **experiência prévia** em Three.js e ecossistema existente

> Ambas as engines são maduras, bem mantidas e capazes de produzir resultados profissionais. A escolha geralmente depende mais das necessidades específicas do projeto e da experiência da equipe do que de limitações técnicas.

---

## 13. A-Frame — Realidade Virtual na Web

A-Frame é um framework web para construção de experiências de realidade virtual (VR) e aumentada (AR), baseado em HTML. Criado originalmente pela equipe de VR do Mozilla, o A-Frame permite criar cenas 3D imersivas usando apenas tags HTML, sem necessidade de escrever código JavaScript complexo.

> Documentação oficial: [https://aframe.io/docs/](https://aframe.io/docs/)
> Registro de componentes: [https://aframe.io/aframe-registry/](https://aframe.io/aframe-registry/)

---

### Conceitos

O A-Frame é construído sobre o **Three.js**, abstraindo sua API JavaScript em uma interface declarativa baseada em HTML. Qualquer cena A-Frame é, internamente, uma cena Three.js — mas o desenvolvedor interage com ela usando a familiaridade do HTML.

**Características principais:**

| Característica | Descrição |
|---------------|-----------|
| **HTML-first** | Cenas 3D declaradas com tags HTML customizadas |
| **Baseado em Three.js** | Todo o poder do Three.js acessível via JavaScript quando necessário |
| **Entity-Component-System** | Arquitetura ECS para composição flexível de objetos |
| **WebXR nativo** | Suporte a headsets VR (Meta Quest, HTC Vive, etc.) sem configuração |
| **Cross-platform** | Desktop, mobile, e headsets VR com a mesma codebase |
| **Comunidade ativa** | Centenas de componentes reutilizáveis disponíveis |

**Suporte a navegadores e dispositivos:**

| Plataforma | Navegador | Suporte |
|-----------|-----------|---------|
| Desktop | Chrome, Firefox, Edge | 3D com mouse/teclado |
| Mobile | Chrome (Android), Safari (iOS) | 3D com toque e giroscópio |
| Meta Quest | Oculus Browser | VR imersivo completo |
| HTC Vive/Valve Index | SteamVR Browser | VR imersivo completo |
| Apple Vision Pro | Safari (visionOS) | VR/MR (suporte inicial) |

**Arquitetura ECS (Entity-Component-System):**

A-Frame segue o padrão arquitetural **Entity-Component-System**, diferente do modelo de herança tradicional da orientação a objetos:

```
┌──────────────────────────────────────────────────────────────────┐
│               ENTITY-COMPONENT-SYSTEM (ECS)                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Entity (Entidade)                                                │
│  └── Container vazio, ganha comportamento através de componentes  │
│                                                                   │
│  Component (Componente)                                           │
│  └── Dados + lógica reutilizável (posição, geometria, material)   │
│                                                                   │
│  System (Sistema)                                                 │
│  └── Lógica global que opera sobre entidades com certos           │
│      componentes (ex: sistema de física aplica gravidade a        │
│      todas as entidades com componente "dynamic-body")            │
│                                                                   │
│  Composição:                                                      │
│  <a-entity position="0 1 0"     ← componente posição             │
│            geometry="primitive: box"  ← componente geometria      │
│            material="color: red"      ← componente material       │
│            animation="property: rotation; to: 0 360 0">           │
│                                       ← componente animação       │
│  </a-entity>                                                      │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

> O ECS favorece **composição sobre herança**. Em vez de criar uma classe "CuboVermelho" que herda de "Cubo" que herda de "Mesh3D", você cria uma entidade vazia e adiciona componentes: geometria de cubo + material vermelho + animação de rotação.

---

### Entidades, Componentes e Sistemas (ECS)

**Entidade (`<a-entity>`)** — o bloco fundamental:

```html
<!-- Entidade vazia (não renderiza nada) -->
<a-entity></a-entity>

<!-- Entidade com componentes (renderiza um cubo vermelho na posição 0,2,0) -->
<a-entity
  position="0 2 -3"
  rotation="45 45 0"
  scale="1 1 1"
  geometry="primitive: box; width: 1; height: 1; depth: 1"
  material="color: #e74c3c; metalness: 0.3; roughness: 0.7"
></a-entity>
```

**Componentes multi-propriedade:**

Os componentes do A-Frame aceitam múltiplas propriedades em formato similar a CSS inline:

```html
<!-- Formato: componente="prop1: valor1; prop2: valor2" -->
<a-entity
  geometry="primitive: cylinder; radius: 0.5; height: 2; segmentsRadial: 32"
  material="color: #3498db; opacity: 0.8; transparent: true"
  position="3 1 -4"
  shadow="cast: true; receive: false"
></a-entity>
```

**Registrando componentes customizados:**

```javascript
// Componente de auto-rotação
AFRAME.registerComponent('auto-rotate', {
  // Schema define as propriedades configuráveis
  schema: {
    speed: { type: 'number', default: 1 },
    axis: { type: 'string', default: 'y' }, // 'x', 'y' ou 'z'
  },

  // Chamado uma vez quando o componente é inicializado
  init: function () {
    this.rotation = this.el.object3D.rotation;
    console.log('Auto-rotate inicializado:', this.data);
  },

  // Chamado a cada frame de renderização
  tick: function (time, deltaTime) {
    const speed = this.data.speed;
    const delta = (deltaTime / 1000) * speed;

    switch (this.data.axis) {
      case 'x': this.rotation.x += delta; break;
      case 'y': this.rotation.y += delta; break;
      case 'z': this.rotation.z += delta; break;
    }
  },

  // Chamado quando o componente é removido
  remove: function () {
    this.el.object3D.rotation.set(0, 0, 0);
  },
});

// Componente de mudança de cor ao passar o cursor
AFRAME.registerComponent('color-on-hover', {
  schema: {
    hoverColor: { type: 'color', default: '#ff0' },
    originalColor: { type: 'color', default: '#fff' },
  },

  init: function () {
    const el = this.el;
    const data = this.data;

    el.addEventListener('mouseenter', () => {
      el.setAttribute('material', 'color', data.hoverColor);
      el.object3D.scale.multiplyScalar(1.1);
    });

    el.addEventListener('mouseleave', () => {
      el.setAttribute('material', 'color', data.originalColor);
      el.object3D.scale.set(1, 1, 1);
    });
  },
});
```

```html
<!-- Uso dos componentes customizados -->
<a-scene>
  <a-entity
    geometry="primitive: box"
    material="color: #2ecc71"
    position="0 1.5 -4"
    auto-rotate="speed: 2; axis: y"
    color-on-hover="hoverColor: #e74c3c; originalColor: #2ecc71"
  ></a-entity>

  <a-entity
    geometry="primitive: torus; radius: 1; radiusTubular: 0.2"
    material="color: #9b59b6"
    position="3 1.5 -4"
    auto-rotate="speed: 0.5; axis: x"
  ></a-entity>

  <a-camera><a-cursor></a-cursor></a-camera>
</a-scene>
```

**Registrando Sistemas:**

```javascript
// Sistema que gerencia todos os objetos coletáveis na cena
AFRAME.registerSystem('collectible-system', {
  schema: {},

  init: function () {
    this.collectibles = [];
    this.score = 0;
  },

  registerCollectible: function (el) {
    this.collectibles.push(el);
  },

  collect: function (el) {
    const index = this.collectibles.indexOf(el);
    if (index > -1) {
      this.collectibles.splice(index, 1);
      this.score += 10;
      el.parentNode.removeChild(el);
      console.log('Score:', this.score);
    }
  },
});

// Componente que registra a entidade no sistema
AFRAME.registerComponent('collectible', {
  init: function () {
    // Acessar o sistema pelo nome
    const system = this.el.sceneEl.systems['collectible-system'];
    system.registerCollectible(this.el);

    this.el.addEventListener('click', () => {
      system.collect(this.el);
    });
  },
});
```

---

### Primitivas

As **primitivas** do A-Frame são atalhos semânticos para combinações comuns de entidade + componentes. Internamente, cada primitiva é um `<a-entity>` com valores padrão.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>A-Frame — Primitivas</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
</head>
<body>
  <a-scene background="color: #1a1a2e">

    <!-- Cubo -->
    <a-box
      position="-3 1 -5"
      width="1.5" height="1.5" depth="1.5"
      color="#e74c3c"
      rotation="0 45 0"
      shadow="cast: true"
    ></a-box>

    <!-- Esfera -->
    <a-sphere
      position="0 1.5 -5"
      radius="1"
      color="#3498db"
      segments-height="32"
      segments-width="32"
      metalness="0.4"
      roughness="0.3"
    ></a-sphere>

    <!-- Cilindro -->
    <a-cylinder
      position="3 1 -5"
      radius="0.6" height="2"
      color="#2ecc71"
      segments-radial="32"
      shadow="cast: true"
    ></a-cylinder>

    <!-- Plano (chão) -->
    <a-plane
      position="0 0 -5"
      rotation="-90 0 0"
      width="15" height="15"
      color="#2c3e50"
      shadow="receive: true"
    ></a-plane>

    <!-- Anel -->
    <a-ring
      position="-3 3 -5"
      radius-inner="0.5" radius-outer="0.8"
      color="#f39c12"
      segments-theta="64"
    ></a-ring>

    <!-- Toroide -->
    <a-torus
      position="0 3.5 -5"
      radius="0.7" radius-tubular="0.15"
      color="#e91e63"
      segments-radial="32"
      segments-tubular="48"
      animation="property: rotation; to: 360 360 0; loop: true; dur: 5000"
    ></a-torus>

    <!-- Céu (skybox esférico) -->
    <a-sky color="#0d1117"></a-sky>

    <!-- Texto 3D -->
    <a-text
      value="Bem-vindo ao A-Frame!"
      position="0 4.5 -5"
      align="center"
      color="#ecf0f1"
      width="8"
      font="mozillavr"
    ></a-text>

    <!-- Luzes -->
    <a-light type="ambient" color="#404040" intensity="0.5"></a-light>
    <a-light type="directional" color="#ffffff" intensity="0.8"
             position="-2 4 3" castShadow="true"></a-light>
    <a-light type="point" color="#ff6600" intensity="0.6"
             position="3 3 -3" distance="10"></a-light>

    <!-- Câmera com cursor (interação por gaze) -->
    <a-camera position="0 1.6 0">
      <a-cursor
        color="#ffffff"
        fuse="true"
        fuse-timeout="1500"
        raycaster="objects: .clickable"
      ></a-cursor>
    </a-camera>

  </a-scene>
</body>
</html>
```

> As primitivas simplificam a escrita, mas toda primitiva pode ser reescrita como `<a-entity>`. Por exemplo, `<a-box color="red">` equivale a `<a-entity geometry="primitive: box" material="color: red">`.

---

### Assets e Asset Management

O sistema de assets do A-Frame gerencia o carregamento prévio de recursos (imagens, modelos 3D, áudio, vídeo) para evitar flickering e garantir que tudo esteja pronto antes de renderizar a cena.

```html
<a-scene>
  <!-- Sistema de assets — carregamento prévio -->
  <a-assets timeout="30000">
    <!-- Imagens / Texturas -->
    <img id="textura-madeira" src="/textures/wood.jpg" crossorigin="anonymous">
    <img id="textura-metal" src="/textures/metal.jpg" crossorigin="anonymous">
    <img id="foto-panorama" src="/textures/panorama360.jpg" crossorigin="anonymous">

    <!-- Modelos 3D -->
    <a-asset-item id="modelo-cadeira" src="/models/chair.glb"></a-asset-item>
    <a-asset-item id="modelo-mesa" src="/models/table.glb"></a-asset-item>

    <!-- Áudio -->
    <audio id="som-ambiente" src="/audio/ambient.mp3" preload="auto"></audio>
    <audio id="som-click" src="/audio/click.ogg" preload="auto"></audio>

    <!-- Vídeo -->
    <video id="video-tutorial" src="/video/tutorial.mp4"
           autoplay loop="true" crossorigin="anonymous"></video>
  </a-assets>

  <!-- Usando os assets carregados -->

  <!-- Skybox panorâmico com imagem 360° -->
  <a-sky src="#foto-panorama" rotation="0 -90 0"></a-sky>

  <!-- Chão com textura de madeira -->
  <a-plane
    position="0 0 0"
    rotation="-90 0 0"
    width="10" height="10"
    src="#textura-madeira"
    repeat="4 4"
  ></a-plane>

  <!-- Cubo com textura de metal -->
  <a-box
    position="0 1 -3"
    src="#textura-metal"
    width="1.5" height="1.5" depth="1.5"
  ></a-box>

  <!-- Modelo 3D carregado -->
  <a-entity
    gltf-model="#modelo-cadeira"
    position="2 0 -4"
    scale="0.8 0.8 0.8"
  ></a-entity>

  <a-entity
    gltf-model="#modelo-mesa"
    position="-2 0 -4"
    scale="1 1 1"
  ></a-entity>

  <!-- Vídeo como textura em um plano -->
  <a-plane
    src="#video-tutorial"
    position="0 2 -6"
    width="4" height="2.25"
  ></a-plane>

  <!-- Som ambiente -->
  <a-entity
    sound="src: #som-ambiente; autoplay: true; loop: true; volume: 0.3"
  ></a-entity>

  <a-camera position="0 1.6 0">
    <a-cursor></a-cursor>
  </a-camera>
</a-scene>
```

**Eventos de carregamento:**

```javascript
// Monitorar progresso de carregamento
const assets = document.querySelector('a-assets');

assets.addEventListener('loaded', () => {
  console.log('Todos os assets foram carregados!');
  document.getElementById('loading-screen').style.display = 'none';
});

assets.addEventListener('timeout', () => {
  console.warn('Timeout no carregamento de assets');
});

// Monitorar asset individual
const modelo = document.querySelector('#modelo-cadeira');
modelo.addEventListener('loaded', () => {
  console.log('Modelo da cadeira carregado');
});
```

> Defina o atributo `timeout` em `<a-assets>` para controlar quanto tempo esperar pelo carregamento completo antes de renderizar a cena mesmo assim. O valor padrão é 3 segundos — aumente para cenas com muitos assets pesados.

---

### Animações

O componente `animation` do A-Frame permite animar qualquer propriedade de uma entidade de forma declarativa, diretamente no HTML.

```html
<a-scene background="color: #1a1a2e">

  <!-- Animação básica de rotação contínua -->
  <a-box
    position="-3 1.5 -5"
    color="#e74c3c"
    animation="property: rotation;
               to: 0 360 0;
               dur: 3000;
               loop: true;
               easing: linear"
  ></a-box>

  <!-- Múltiplas animações na mesma entidade -->
  <a-sphere
    position="0 1.5 -5"
    color="#3498db"
    radius="0.8"
    animation__rotation="property: rotation;
                         to: 360 360 0;
                         dur: 4000;
                         loop: true;
                         easing: linear"
    animation__bounce="property: position;
                       from: 0 1.5 -5;
                       to: 0 3 -5;
                       dur: 1500;
                       loop: true;
                       dir: alternate;
                       easing: easeInOutQuad"
    animation__color="property: material.color;
                      from: #3498db;
                      to: #e74c3c;
                      dur: 2000;
                      loop: true;
                      dir: alternate"
  ></a-sphere>

  <!-- Animação disparada por evento (hover) -->
  <a-cylinder
    position="3 1 -5"
    radius="0.5" height="2"
    color="#2ecc71"
    class="clickable"
    animation__grow="property: scale;
                     to: 1.5 1.5 1.5;
                     dur: 300;
                     startEvents: mouseenter;
                     easing: easeOutElastic"
    animation__shrink="property: scale;
                       to: 1 1 1;
                       dur: 300;
                       startEvents: mouseleave;
                       easing: easeOutQuad"
    animation__click="property: material.color;
                      from: #2ecc71;
                      to: #f39c12;
                      dur: 500;
                      startEvents: click;
                      dir: alternate;
                      loop: 2"
  ></a-cylinder>

  <!-- Animação com delay (timeline sequencial) -->
  <a-entity position="0 0 -8">
    <a-box position="-2 0.5 0" color="#e74c3c"
           animation="property: position; to: -2 3 0; dur: 1000;
                      delay: 0; loop: true; dir: alternate; easing: easeInOutQuad">
    </a-box>
    <a-box position="0 0.5 0" color="#f39c12"
           animation="property: position; to: 0 3 0; dur: 1000;
                      delay: 300; loop: true; dir: alternate; easing: easeInOutQuad">
    </a-box>
    <a-box position="2 0.5 0" color="#2ecc71"
           animation="property: position; to: 2 3 0; dur: 1000;
                      delay: 600; loop: true; dir: alternate; easing: easeInOutQuad">
    </a-box>
  </a-entity>

  <a-plane position="0 0 -5" rotation="-90 0 0" width="15" height="15" color="#2c3e50"></a-plane>

  <a-camera position="0 1.6 0">
    <a-cursor color="#fff" raycaster="objects: .clickable"></a-cursor>
  </a-camera>

</a-scene>
```

**Propriedades do componente `animation`:**

| Propriedade | Tipo | Descrição |
|------------|------|-----------|
| `property` | string | Propriedade a animar (ex: `position`, `rotation`, `material.color`) |
| `from` | any | Valor inicial (se omitido, usa o valor atual) |
| `to` | any | Valor final |
| `dur` | number | Duração em milissegundos |
| `delay` | number | Atraso antes de iniciar (ms) |
| `loop` | boolean/number | `true` para infinito ou número de repetições |
| `dir` | string | `normal`, `alternate` (vai e volta), `reverse` |
| `easing` | string | Função de easing (ex: `easeInOutQuad`, `linear`, `easeOutElastic`) |
| `startEvents` | string | Evento(s) que disparam a animação |
| `pauseEvents` | string | Evento(s) que pausam a animação |
| `resumeEvents` | string | Evento(s) que retomam a animação |
| `autoplay` | boolean | Iniciar automaticamente (padrão: `true`) |
| `enabled` | boolean | Habilitar/desabilitar o componente |

---

### Interação

A-Frame suporta múltiplos tipos de interação: gaze (olhar), controladores VR, mãos (hand tracking) e mouse/toque para desktop e mobile.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>A-Frame — Galeria Interativa VR</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script>
    // Componente: exibir informação ao selecionar
    AFRAME.registerComponent('info-panel', {
      schema: {
        title: { type: 'string', default: '' },
        description: { type: 'string', default: '' },
      },

      init: function () {
        const el = this.el;
        const data = this.data;
        const infoEl = document.getElementById('info-display');
        const infoTitle = document.getElementById('info-title');
        const infoDesc = document.getElementById('info-desc');

        el.addEventListener('mouseenter', () => {
          el.setAttribute('material', 'emissive', '#333');
          el.setAttribute('scale', '1.05 1.05 1.05');
        });

        el.addEventListener('mouseleave', () => {
          el.setAttribute('material', 'emissive', '#000');
          el.setAttribute('scale', '1 1 1');
        });

        el.addEventListener('click', () => {
          infoTitle.setAttribute('value', data.title);
          infoDesc.setAttribute('value', data.description);
          infoEl.setAttribute('visible', 'true');

          // Esconder após 4 segundos
          setTimeout(() => {
            infoEl.setAttribute('visible', 'false');
          }, 4000);
        });
      },
    });

    // Componente: reação a controladores VR
    AFRAME.registerComponent('vr-interactive', {
      init: function () {
        const el = this.el;

        // Eventos de controlador VR
        el.addEventListener('triggerdown', () => {
          el.setAttribute('material', 'color', '#ff0');
          el.emit('selected');
        });

        el.addEventListener('gripdown', () => {
          el.setAttribute('scale', '2 2 2');
        });

        el.addEventListener('gripup', () => {
          el.setAttribute('scale', '1 1 1');
        });
      },
    });
  </script>
</head>
<body>
  <a-scene>
    <a-assets>
      <img id="img1" src="/gallery/photo1.jpg" crossorigin="anonymous">
      <img id="img2" src="/gallery/photo2.jpg" crossorigin="anonymous">
      <img id="img3" src="/gallery/photo3.jpg" crossorigin="anonymous">
    </a-assets>

    <!-- Céu -->
    <a-sky color="#1a1a2e"></a-sky>
    <a-plane rotation="-90 0 0" width="20" height="20" color="#2c3e50"></a-plane>

    <!-- Quadros da galeria (parede esquerda) -->
    <a-entity position="-4 2 -3" rotation="0 90 0">
      <a-plane
        src="#img1" width="2.5" height="1.8"
        class="clickable"
        info-panel="title: Paisagem Urbana; description: Fotografia urbana capturada ao entardecer"
      ></a-plane>
      <!-- Moldura -->
      <a-box position="0 0 -0.05" width="2.7" height="2.0" depth="0.08"
             color="#8B4513" metalness="0.2" roughness="0.8"></a-box>
    </a-entity>

    <!-- Quadro frontal -->
    <a-entity position="0 2 -5">
      <a-plane
        src="#img2" width="3" height="2"
        class="clickable"
        info-panel="title: Natureza; description: Vista panoramica de montanha ao amanhecer"
      ></a-plane>
      <a-box position="0 0 -0.05" width="3.2" height="2.2" depth="0.08"
             color="#8B4513" metalness="0.2" roughness="0.8"></a-box>
    </a-entity>

    <!-- Quadro (parede direita) -->
    <a-entity position="4 2 -3" rotation="0 -90 0">
      <a-plane
        src="#img3" width="2.5" height="1.8"
        class="clickable"
        info-panel="title: Abstrato; description: Arte digital generativa com cores vibrantes"
      ></a-plane>
      <a-box position="0 0 -0.05" width="2.7" height="2.0" depth="0.08"
             color="#8B4513" metalness="0.2" roughness="0.8"></a-box>
    </a-entity>

    <!-- Painel de informações (inicialmente invisível) -->
    <a-entity id="info-display" position="0 1 -2" visible="false">
      <a-plane width="3" height="1" color="#000" opacity="0.8" side="double"></a-plane>
      <a-text id="info-title" value="" position="0 0.2 0.01"
              align="center" color="#fff" width="2.5" font="mozillavr"></a-text>
      <a-text id="info-desc" value="" position="0 -0.15 0.01"
              align="center" color="#aaa" width="2.5" font="mozillavr"
              wrap-count="40"></a-text>
    </a-entity>

    <!-- Luzes -->
    <a-light type="ambient" intensity="0.4"></a-light>
    <a-light type="point" position="0 4 -3" intensity="0.8" color="#fff"></a-light>
    <a-light type="spot" position="-4 3.5 -3" rotation="-45 90 0"
             intensity="0.5" angle="30"></a-light>
    <a-light type="spot" position="4 3.5 -3" rotation="-45 -90 0"
             intensity="0.5" angle="30"></a-light>

    <!-- Câmera com cursor de gaze e suporte a controladores VR -->
    <a-camera position="0 1.6 0">
      <a-cursor
        color="#fff"
        fuse="true"
        fuse-timeout="2000"
        raycaster="objects: .clickable"
        animation__fusing="property: scale; from: 1 1 1; to: 0.5 0.5 0.5;
                           dur: 2000; startEvents: fusing"
        animation__defuse="property: scale; to: 1 1 1;
                           dur: 200; startEvents: mouseleave"
      ></a-cursor>
    </a-camera>

    <!-- Controladores VR (aparecem automaticamente em modo VR) -->
    <a-entity
      laser-controls="hand: left"
      raycaster="objects: .clickable; far: 10"
    ></a-entity>
    <a-entity
      laser-controls="hand: right"
      raycaster="objects: .clickable; far: 10"
    ></a-entity>

  </a-scene>
</body>
</html>
```

> O cursor com `fuse="true"` simula um clique após o usuário olhar fixamente para um objeto por um tempo definido (`fuse-timeout`). Esse é o principal mecanismo de interação em headsets VR sem controladores.

---

### Integração com Three.js

Como o A-Frame é construído sobre o Three.js, é possível acessar os objetos Three.js subjacentes para operações avançadas que não são possíveis apenas com HTML.

```javascript
// Acessando objetos Three.js a partir de entidades A-Frame

AFRAME.registerComponent('three-js-shader', {
  init: function () {
    const el = this.el;

    // Acessar o Object3D da entidade (grupo Three.js)
    const object3D = el.object3D;

    // Acessar o mesh Three.js específico
    const mesh = el.getObject3D('mesh');

    // Acessar a cena Three.js global
    const threeScene = el.sceneEl.object3D;

    // Acessar o renderer Three.js
    const renderer = el.sceneEl.renderer;

    // Acessar a câmera Three.js
    const camera = el.sceneEl.camera;

    if (!mesh) {
      // Aguardar até que o mesh esteja disponível
      el.addEventListener('object3dset', () => {
        this.applyShader(el.getObject3D('mesh'));
      });
      return;
    }

    this.applyShader(mesh);
  },

  applyShader: function (mesh) {
    // Substituir material por ShaderMaterial customizado
    const shaderMaterial = new THREE.ShaderMaterial({
      uniforms: {
        time: { value: 0.0 },
        color1: { value: new THREE.Color('#e74c3c') },
        color2: { value: new THREE.Color('#3498db') },
      },
      vertexShader: `
        varying vec2 vUv;
        varying vec3 vPosition;
        uniform float time;

        void main() {
          vUv = uv;
          vPosition = position;
          vec3 pos = position;
          pos.z += sin(pos.x * 3.0 + time * 2.0) * 0.15;
          pos.z += cos(pos.y * 3.0 + time * 1.5) * 0.15;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
        }
      `,
      fragmentShader: `
        uniform float time;
        uniform vec3 color1;
        uniform vec3 color2;
        varying vec2 vUv;

        void main() {
          float pattern = sin(vUv.x * 10.0 + time) * cos(vUv.y * 10.0 + time * 0.5);
          vec3 color = mix(color1, color2, pattern * 0.5 + 0.5);
          gl_FragColor = vec4(color, 1.0);
        }
      `,
      side: THREE.DoubleSide,
    });

    mesh.material = shaderMaterial;
    this.shaderMaterial = shaderMaterial;
  },

  tick: function (time) {
    if (this.shaderMaterial) {
      this.shaderMaterial.uniforms.time.value = time / 1000;
    }
  },
});
```

```html
<!-- Uso do componente com shader Three.js -->
<a-scene>
  <a-plane
    position="0 2 -4"
    width="3" height="3"
    segments-width="32"
    segments-height="32"
    three-js-shader
  ></a-plane>

  <a-camera position="0 1.6 0"><a-cursor></a-cursor></a-camera>
</a-scene>
```

> A integração com Three.js é o "escape hatch" do A-Frame. Use-a quando precisar de shaders customizados, post-processing, ou qualquer funcionalidade avançada que o A-Frame não expõe via HTML.

---

### Componentes da Comunidade

O ecossistema A-Frame possui uma vasta coleção de componentes criados pela comunidade que estendem suas funcionalidades sem necessidade de código JavaScript.

| Componente | Pacote npm | Descrição | Caso de Uso |
|-----------|------------|-----------|-------------|
| **aframe-extras** | `aframe-extras` | Controles de movimento, loaders, animações | Navegação, modelos animados |
| **aframe-physics-system** | `aframe-physics-system` | Física baseada em CANNON.js | Gravidade, colisões, corpos rígidos |
| **aframe-particle-system** | `aframe-particle-system-component` | Sistema de partículas declarativo | Fogo, neve, chuva, magia |
| **aframe-environment** | `aframe-environment-component` | Ambientes procedurais completos | Cenários rápidos (floresta, deserto, etc.) |
| **super-hands** | `super-hands` | Interação natural com as mãos | Agarrar, esticar, arrastar objetos |
| **networked-aframe** | `networked-aframe` | Experiências VR multiplayer | Salas VR compartilhadas |
| **aframe-teleport-controls** | `aframe-teleport-controls` | Teletransporte com controladores | Navegação em espaços grandes |
| **aframe-state-component** | `aframe-state-component` | Gerenciamento de estado centralizado | Aplicações VR complexas |
| **aframe-event-set** | `aframe-event-set-component` | Setar atributos em resposta a eventos | Interações sem JavaScript |

```html
<!-- Exemplo: ambiente procedural + controles de movimento + partículas -->
<head>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/aframe-extras@7/dist/aframe-extras.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/aframe-environment-component@1/dist/aframe-environment-component.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/aframe-particle-system-component@1/dist/aframe-particle-system-component.min.js"></script>
</head>
<body>
  <a-scene>
    <!-- Ambiente procedural (floresta, com neblina) -->
    <a-entity environment="preset: forest; fog: 0.7; groundTexture: walkernoise;
              playArea: 1; dressingAmount: 200"></a-entity>

    <!-- Neve com partículas -->
    <a-entity
      position="0 10 0"
      particle-system="preset: dust;
                       color: #fff, #ddd;
                       particleCount: 2000;
                       maxAge: 8;
                       size: 0.3, 0.6;
                       velocityValue: 0 -2 0;
                       velocitySpread: 2 0 2;
                       opacity: 0.8, 0"
    ></a-entity>

    <!-- Câmera com controles de movimento (WASD + look) -->
    <a-entity
      movement-controls="speed: 0.08"
      position="0 0 0"
    >
      <a-entity
        camera
        look-controls
        position="0 1.6 0"
      >
        <a-cursor color="#fff"></a-cursor>
      </a-entity>
    </a-entity>
  </a-scene>
</body>
```

> O componente `aframe-environment-component` gera cenários completos (floresta, deserto, vulcão, Japão, etc.) com uma única linha de HTML — excelente para prototipagem rápida.

---

### Realidade Aumentada com AR.js

**AR.js** é uma biblioteca leve que adiciona capacidades de Realidade Aumentada ao A-Frame, suportando dois modos principais: detecção de marcadores (marker-based) e posicionamento por GPS (location-based).

**AR baseada em Marcadores:**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>A-Frame + AR.js — Marcador</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
</head>
<body style="margin: 0; overflow: hidden;">
  <!--
    embedded: não ocupa tela inteira
    arjs: configurações do AR.js
    vr-mode-ui: desabilita botão VR (estamos em AR)
  -->
  <a-scene
    embedded
    arjs="sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix;
          matrixCodeType: 3x3"
    renderer="logarithmicDepthBuffer: true; colorManagement: true"
    vr-mode-ui="enabled: false"
  >
    <!-- Marcador padrão "Hiro" -->
    <a-marker preset="hiro">
      <!-- Modelo 3D que aparece sobre o marcador -->
      <a-box
        position="0 0.5 0"
        color="#e74c3c"
        animation="property: rotation; to: 0 360 0; loop: true; dur: 3000; easing: linear"
        animation__bounce="property: position; from: 0 0.5 0; to: 0 1.2 0;
                           dur: 1500; loop: true; dir: alternate; easing: easeInOutQuad"
      ></a-box>

      <a-sphere
        position="0 2 0"
        radius="0.3"
        color="#f39c12"
        animation="property: rotation; to: 360 360 0; loop: true; dur: 2000"
      ></a-sphere>

      <a-text
        value="Babylon.js & A-Frame"
        position="0 0 0.5"
        rotation="-90 0 0"
        align="center"
        color="#fff"
        width="3"
      ></a-text>
    </a-marker>

    <!-- Marcador customizado (padrão .patt) -->
    <!--
    <a-marker type="pattern" url="/markers/custom-marker.patt">
      <a-entity gltf-model="/models/robot.glb" scale="0.5 0.5 0.5"></a-entity>
    </a-marker>
    -->

    <!-- Câmera (obrigatória para AR.js) -->
    <a-entity camera></a-entity>
  </a-scene>
</body>
</html>
```

> O marcador "Hiro" é o marcador padrão do AR.js. Para criar marcadores personalizados, use o gerador em [https://ar-js-org.github.io/AR.js/three.js/examples/marker-training/examples/generator.html](https://ar-js-org.github.io/AR.js/three.js/examples/marker-training/examples/generator.html).

**AR baseada em Localização (GPS):**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>A-Frame + AR.js — Localização GPS</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>
</head>
<body style="margin: 0; overflow: hidden;">
  <a-scene
    vr-mode-ui="enabled: false"
    arjs="sourceType: webcam; videoTexture: true; debugUIEnabled: false"
    renderer="antialias: true; alpha: true"
  >
    <!--
      Objetos posicionados em coordenadas GPS reais.
      Substitua latitude e longitude por coordenadas reais do local desejado.
    -->

    <!-- Ponto de interesse 1 — Ex: entrada do campus -->
    <a-entity
      gps-entity-place="latitude: -23.5505; longitude: -46.6333"
    >
      <a-box
        position="0 1 0"
        color="#e74c3c"
        scale="2 2 2"
        animation="property: rotation; to: 0 360 0; loop: true; dur: 5000"
      ></a-box>
      <a-text
        value="Ponto A — Entrada"
        position="0 3.5 0"
        align="center"
        color="#fff"
        width="6"
        look-at="[gps-camera]"
      ></a-text>
    </a-entity>

    <!-- Ponto de interesse 2 — Ex: biblioteca -->
    <a-entity
      gps-entity-place="latitude: -23.5510; longitude: -46.6340"
    >
      <a-sphere
        position="0 2 0"
        color="#3498db"
        radius="1.5"
      ></a-sphere>
      <a-text
        value="Ponto B — Biblioteca"
        position="0 4.5 0"
        align="center"
        color="#fff"
        width="6"
        look-at="[gps-camera]"
      ></a-text>
    </a-entity>

    <!-- Câmera GPS -->
    <a-camera gps-camera rotation-reader></a-camera>
  </a-scene>
</body>
</html>
```

> AR baseada em GPS funciona melhor em ambientes externos. A precisão depende do GPS do dispositivo (geralmente 3-10 metros). Para ambientes internos, prefira marcadores ou NFT (Natural Feature Tracking).

---

### Performance em VR

Para experiências VR confortáveis, é necessário manter uma taxa de quadros alta e consistente. Quedas de frame rate causam desconforto e náusea (motion sickness).

**Metas de performance por dispositivo:**

| Dispositivo | FPS Alvo | Observação |
|------------|---------|------------|
| Meta Quest 2/3 | 72-120 FPS | 72 FPS mínimo, 90+ recomendado |
| HTC Vive / Index | 90-144 FPS | Depende do refresh rate do headset |
| Desktop (sem VR) | 60 FPS | Monitor padrão |
| Mobile | 60 FPS | Dispositivos com menor GPU |

**Tabela de problemas e soluções comuns:**

| Problema | Causa | Solução |
|----------|-------|---------|
| **FPS baixo (< 60)** | Muitos draw calls | Mesclar geometrias com `geometry-merger`, reduzir número de entidades |
| **FPS instável (stuttering)** | Garbage collection | Evitar criar/destruir objetos no `tick()`, usar pool de objetos |
| **Carregamento lento** | Assets muito pesados | Comprimir texturas (KTX2/Basis), simplificar modelos (Draco), lazy loading |
| **Texturas borradas** | Resolução muito alta | Usar texture atlas, limitar resolução a 2048x2048 |
| **Muitos triângulos** | Modelos detalhados demais | Implementar LOD (Level of Detail), simplificar modelos |
| **Luzes custosas** | Muitas luzes dinâmicas | Limitar a 3-4 luzes dinâmicas, usar lightmaps pré-calculados |
| **Transparência lenta** | Ordenação de fragmentos | Minimizar objetos transparentes sobrepostos |
| **Sombras pesadas** | Shadow maps grandes | Reduzir resolução de shadow map, limitar raio de sombras |

**Estratégias de otimização:**

```html
<a-scene
  renderer="antialias: false;
            colorManagement: true;
            physicallyCorrectLights: false;
            maxCanvasWidth: 1920;
            maxCanvasHeight: 1080"
  stats
>
  <!-- stats: exibe painel de FPS, draw calls, triângulos -->

  <!-- Usar geometry-merger para reduzir draw calls -->
  <!-- Ao invés de 100 <a-box> individuais, agrupe-os -->
  <a-entity geometry-merger="preserveOriginal: false">
    <a-box position="0 0.5 -3" color="#e74c3c"></a-box>
    <a-box position="1 0.5 -3" color="#e74c3c"></a-box>
    <a-box position="2 0.5 -3" color="#e74c3c"></a-box>
    <!-- Várias boxes mescladas em um único draw call -->
  </a-entity>

  <!-- Lazy loading de modelos distantes -->
  <a-entity
    id="detalhe-distante"
    gltf-model=""
    visible="false"
  ></a-entity>

</a-scene>

<script>
  // Carregar modelo apenas quando o usuário se aproximar
  AFRAME.registerComponent('proximity-loader', {
    schema: {
      target: { type: 'selector' },
      distance: { type: 'number', default: 10 },
      model: { type: 'string' },
    },

    tick: function () {
      const camera = this.el.sceneEl.camera.el;
      const target = this.data.target;
      const dist = camera.object3D.position.distanceTo(target.object3D.position);

      if (dist < this.data.distance && !target.getAttribute('gltf-model')) {
        target.setAttribute('gltf-model', this.data.model);
        target.setAttribute('visible', 'true');
      }
    },
  });
</script>
```

> Use o componente `stats` do A-Frame (atributo `stats` na `<a-scene>`) para monitorar FPS, draw calls e contagem de triângulos durante o desenvolvimento. Remova-o em produção.

---

### Exemplo Prático: Galeria Virtual

Um exemplo completo de galeria VR com panorama de céu, quadros nas paredes, navegação e interação por gaze:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Galeria Virtual VR</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/aframe-extras@7/dist/aframe-extras.min.js"></script>
  <script>
    AFRAME.registerComponent('gallery-item', {
      schema: {
        title: { type: 'string' },
        artist: { type: 'string' },
      },
      init: function () {
        const el = this.el;
        const data = this.data;
        const label = document.getElementById('label');
        const labelTitle = document.getElementById('label-title');
        const labelArtist = document.getElementById('label-artist');

        el.addEventListener('mouseenter', () => {
          el.setAttribute('animation__glow', {
            property: 'material.emissiveIntensity',
            to: 0.15, dur: 300,
          });
        });
        el.addEventListener('mouseleave', () => {
          el.setAttribute('animation__glow', {
            property: 'material.emissiveIntensity',
            to: 0, dur: 300,
          });
          label.setAttribute('visible', false);
        });
        el.addEventListener('click', () => {
          labelTitle.setAttribute('value', data.title);
          labelArtist.setAttribute('value', data.artist);
          label.setAttribute('visible', true);
          label.object3D.position.copy(el.object3D.position);
          label.object3D.position.y -= 1.2;
        });
      },
    });
  </script>
</head>
<body>
  <a-scene background="color: #111">
    <a-assets>
      <img id="sky" src="/textures/gallery-sky.jpg" crossorigin="anonymous">
      <img id="floor-tex" src="/textures/marble.jpg" crossorigin="anonymous">
      <img id="art1" src="/gallery/art1.jpg" crossorigin="anonymous">
      <img id="art2" src="/gallery/art2.jpg" crossorigin="anonymous">
      <img id="art3" src="/gallery/art3.jpg" crossorigin="anonymous">
      <img id="art4" src="/gallery/art4.jpg" crossorigin="anonymous">
    </a-assets>

    <a-sky src="#sky" rotation="0 -90 0"></a-sky>
    <a-plane src="#floor-tex" rotation="-90 0 0" width="16" height="16"
             repeat="4 4" roughness="0.3" metalness="0.1"></a-plane>

    <!-- Paredes -->
    <a-box position="0 2.5 -7" width="16" height="5" depth="0.2" color="#f5f0e8"></a-box>
    <a-box position="-7 2.5 0" width="0.2" height="5" depth="14" color="#f5f0e8"></a-box>
    <a-box position="7 2.5 0" width="0.2" height="5" depth="14" color="#f5f0e8"></a-box>

    <!-- Quadros — parede frontal -->
    <a-entity position="-3 2.2 -6.8">
      <a-plane src="#art1" width="2" height="1.5" class="clickable"
               gallery-item="title: Aurora Boreal; artist: Maria Silva"></a-plane>
      <a-box position="0 0 -0.06" width="2.2" height="1.7" depth="0.08" color="#3e2723"></a-box>
    </a-entity>
    <a-entity position="3 2.2 -6.8">
      <a-plane src="#art2" width="2" height="1.5" class="clickable"
               gallery-item="title: Oceano Profundo; artist: Carlos Santos"></a-plane>
      <a-box position="0 0 -0.06" width="2.2" height="1.7" depth="0.08" color="#3e2723"></a-box>
    </a-entity>

    <!-- Quadros — paredes laterais -->
    <a-entity position="-6.8 2.2 -3" rotation="0 90 0">
      <a-plane src="#art3" width="2.5" height="1.8" class="clickable"
               gallery-item="title: Metropolis; artist: Ana Costa"></a-plane>
      <a-box position="0 0 -0.06" width="2.7" height="2.0" depth="0.08" color="#3e2723"></a-box>
    </a-entity>
    <a-entity position="6.8 2.2 -3" rotation="0 -90 0">
      <a-plane src="#art4" width="2.5" height="1.8" class="clickable"
               gallery-item="title: Fractais; artist: Pedro Lima"></a-plane>
      <a-box position="0 0 -0.06" width="2.7" height="2.0" depth="0.08" color="#3e2723"></a-box>
    </a-entity>

    <!-- Label de informações -->
    <a-entity id="label" visible="false">
      <a-plane width="2.2" height="0.5" color="#111" opacity="0.85"></a-plane>
      <a-text id="label-title" value="" position="0 0.08 0.01" align="center"
              color="#fff" width="2"></a-text>
      <a-text id="label-artist" value="" position="0 -0.12 0.01" align="center"
              color="#aaa" width="1.8"></a-text>
    </a-entity>

    <!-- Iluminação -->
    <a-light type="ambient" intensity="0.4"></a-light>
    <a-light type="point" position="0 4.5 -3" intensity="0.8" decay="2" distance="15"></a-light>
    <a-light type="spot" position="-3 4 -6" rotation="-60 0 0"
             intensity="0.6" angle="35" penumbra="0.4"></a-light>
    <a-light type="spot" position="3 4 -6" rotation="-60 0 0"
             intensity="0.6" angle="35" penumbra="0.4"></a-light>

    <!-- Navegação -->
    <a-entity movement-controls="speed: 0.06" position="0 0 2">
      <a-entity camera look-controls="pointerLockEnabled: false" position="0 1.6 0">
        <a-cursor color="#fff" fuse="true" fuse-timeout="1500"
                  raycaster="objects: .clickable; far: 12"></a-cursor>
      </a-entity>
    </a-entity>
  </a-scene>
</body>
</html>
```

> Esta galeria funciona tanto em desktop (WASD + mouse) quanto em VR (controladores ou gaze). O componente `movement-controls` do aframe-extras detecta automaticamente o dispositivo e adapta a navegação.

---

### Exemplo Prático: Cena AR

Um exemplo completo de AR com marcador, onde um modelo 3D aparece e rotaciona quando o marcador "Hiro" é detectado pela câmera:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Cena AR — Modelo sobre Marcador</title>
  <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
  <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
  <style>
    body { margin: 0; overflow: hidden; }
    /* Esconder botões desnecessários */
    .a-enter-vr-button, .a-enter-ar-button { display: none !important; }
  </style>
</head>
<body>
  <a-scene
    embedded
    arjs="sourceType: webcam; debugUIEnabled: false;"
    vr-mode-ui="enabled: false"
    renderer="logarithmicDepthBuffer: true; alpha: true"
    gesture-detector
  >
    <a-assets>
      <!-- Substitua pelo caminho do seu modelo -->
      <a-asset-item id="robot" src="/models/robot.glb"></a-asset-item>
    </a-assets>

    <a-marker preset="hiro" smooth="true" smoothCount="5" smoothThreshold="2">
      <!-- Modelo 3D com rotação contínua -->
      <a-entity
        gltf-model="#robot"
        scale="0.3 0.3 0.3"
        position="0 0 0"
        animation="property: rotation; to: 0 360 0; loop: true; dur: 4000; easing: linear"
      ></a-entity>

      <!-- Base decorativa -->
      <a-cylinder
        position="0 -0.05 0"
        radius="0.8" height="0.1"
        color="#2c3e50"
        opacity="0.7"
        metalness="0.5"
      ></a-cylinder>

      <!-- Texto identificador -->
      <a-text
        value="Modelo 3D"
        position="0 0 0.9"
        rotation="-90 0 0"
        align="center"
        color="#ecf0f1"
        width="2"
      ></a-text>
    </a-marker>

    <a-entity camera></a-entity>
  </a-scene>
</body>
</html>
```

> Para testar, imprima o marcador Hiro ([disponível aqui](https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png)) ou exiba-o em outro dispositivo. Aponte a câmera do navegador para o marcador e o modelo 3D aparecerá sobre ele.

---

## 14. Formatos de Arquivos 3D — Visão Geral

Escolher o formato correto de arquivo 3D é fundamental para a performance e qualidade da experiência web. Esta seção apresenta os principais formatos e suas características.

---

### glTF 2.0 / GLB — O "JPEG dos Modelos 3D"

**glTF (GL Transmission Format)** é o formato padrão da indústria para transmissão de assets 3D, mantido pelo Khronos Group (mesmo grupo do OpenGL e Vulkan).

**Duas variantes:**

| Variante | Estrutura | Uso recomendado |
|----------|-----------|-----------------|
| **glTF** (.gltf) | JSON + binário (.bin) + texturas separadas | Desenvolvimento, quando texturas são compartilhadas |
| **GLB** (.glb) | Arquivo único (JSON + binário + texturas embutidas) | Produção web — um único download |

**Recursos suportados:**
- Malhas (meshes) com geometria otimizada
- Materiais PBR (metallic-roughness)
- Texturas (albedo, normal, metallic, roughness, occlusion, emissive)
- Animações (keyframe, skinning, morph targets/blend shapes)
- Câmeras e luzes (via extensão `KHR_lights_punctual`)
- Múltiplas cenas e hierarquia de nós

**Extensões importantes:**

| Extensão | Descrição |
|----------|-----------|
| `KHR_draco_mesh_compression` | Compressão Draco para geometria |
| `KHR_texture_basisu` | Texturas GPU-compressed (KTX2/Basis Universal) |
| `KHR_materials_unlit` | Materiais sem iluminação (UI, sprites) |
| `KHR_materials_clearcoat` | Camada de verniz (carros, pisos) |
| `KHR_materials_transmission` | Transparência física (vidro, água) |
| `KHR_mesh_quantization` | Quantização de dados da malha |
| `EXT_meshopt_compression` | Compressão Meshopt (alternativa ao Draco) |

> **Por que glTF é recomendado para web:** tamanho compacto, carregamento rápido, suporte direto em Three.js/Babylon.js/A-Frame, materiais PBR compatíveis, extensões de compressão, padrão aberto e bem mantido.

---

### OBJ (Wavefront)

Formato legado composto por múltiplos arquivos:
- `.obj` — geometria (vértices, faces, normais, UVs)
- `.mtl` — materiais (cores, texturas)
- Arquivos de textura separados (PNG, JPG)

**Limitações:** sem animação, sem PBR, sem hierarquia de cena, sem compressão. Arquivos tendem a ser grandes.

**Uso atual:** intercâmbio simples entre ferramentas, modelos estáticos básicos.

---

### FBX (Autodesk)

Formato proprietário da Autodesk amplamente usado em pipelines de produção:
- Suporta animações, skinning, blend shapes e hierarquia de cena
- Formatos binário e ASCII
- Padrão de fato para troca entre Maya, 3ds Max e Blender

**Para web:** não é recomendado diretamente — converta para glTF antes de usar. Three.js possui `FBXLoader`, mas glTF é preferível.

---

### COLLADA (DAE)

Formato XML aberto do Khronos Group (predecessor do glTF):
- Descrição completa de cena (geometria, materiais, animações, câmeras, luzes)
- Legível por humanos (XML)
- Amplamente suportado, mas verboso e lento para carregar

**Status:** substituído pelo glTF 2.0 na maioria dos casos.

---

### USD (Universal Scene Description)

Formato criado pela Pixar para pipelines de produção cinematográfica:
- Cenas complexas com múltiplas camadas e referências
- USDZ: variante compactada usada pela Apple (AR Quick Look no iOS/macOS)
- Adoção crescente (Apple, NVIDIA Omniverse)
- Suporte web ainda em estágio inicial

---

### Tabela Comparativa de Formatos

| Característica | glTF/GLB | OBJ | FBX | COLLADA | USD/USDZ |
|----------------|----------|-----|-----|---------|----------|
| **Animação** | Sim | Não | Sim | Sim | Sim |
| **PBR** | Sim (nativo) | Não | Parcial | Parcial | Sim |
| **Compressão** | Draco, Meshopt, KTX2 | Não | Não | Não | Parcial |
| **Arquivo único** | Sim (GLB) | Não | Sim (binário) | Não | Sim (USDZ) |
| **Tamanho** | Pequeno | Grande | Médio | Grande | Médio |
| **Suporte web** | Excelente | Bom (legado) | Limitado | Limitado | Inicial |
| **Three.js** | GLTFLoader | OBJLoader | FBXLoader | ColladaLoader | USDZExporter |
| **A-Frame** | `<a-gltf-model>` | `<a-obj-model>` | Via loader externo | Via loader externo | Não |
| **Babylon.js** | Sim (nativo) | Sim | Sim | Não oficial | Não oficial |
| **Padrão aberto** | Sim | Sim | Não (Autodesk) | Sim | Sim |
| **Recomendado para web** | **Sim** | Não | Não | Não | Futuro |

---

## 15. Blender — Modelagem e Exportação para Web

Blender é uma ferramenta open-source de modelagem, animação e renderização 3D. Para desenvolvedores web, é a principal ferramenta gratuita para criar e preparar assets 3D para uso em Three.js, Babylon.js e A-Frame.

> Documentação oficial: [docs.blender.org](https://docs.blender.org) | Download: [blender.org](https://www.blender.org)

---

### Visão Geral para Desenvolvedores Web

Não é necessário dominar todo o Blender — para web, o foco está em:
1. Modelagem básica de objetos
2. Configuração de materiais PBR compatíveis com glTF
3. UV unwrapping para texturas
4. Animação simples por keyframes
5. Exportação otimizada para glTF/GLB

**Áreas essenciais da interface:**

| Área | Função |
|------|--------|
| **3D Viewport** | Visualização e edição do modelo |
| **Properties** | Propriedades do objeto, materiais, modificadores |
| **Outliner** | Hierarquia da cena (objetos, coleções) |
| **Timeline** | Controle de animação por keyframes |
| **UV Editor** | Edição de coordenadas UV |
| **Shader Editor** | Criação visual de materiais (nós) |

**Atalhos essenciais:**

| Atalho | Ação |
|--------|------|
| `G` | Mover (Grab) |
| `R` | Rotacionar |
| `S` | Escalar |
| `G/R/S + X/Y/Z` | Restringir ao eixo |
| `Tab` | Alternar Object/Edit Mode |
| `X` / `Delete` | Deletar |
| `E` | Extrudar (Edit Mode) |
| `I` | Inset Faces (Edit Mode) |
| `Ctrl+R` | Loop Cut |
| `Shift+A` | Adicionar objeto/primitiva |
| `Ctrl+J` | Unir objetos selecionados |
| `Alt+Click` (aresta) | Selecionar loop de arestas |
| `Numpad 1/3/7` | Vista frontal/lateral/topo |
| `Numpad 5` | Alternar perspectiva/ortográfica |
| `Z` | Menu de visualização (wireframe, solid, material, rendered) |

---

### Modelagem Básica

**Object Mode vs Edit Mode:**
- **Object Mode:** manipula objetos inteiros (mover, rotacionar, escalar)
- **Edit Mode:** edita a geometria (vértices, arestas, faces)

**Primitivas disponíveis:** Mesh (Cube, Sphere, Cylinder, Cone, Torus, Plane, Circle, Monkey), Curve, Surface, Text, Empty, Camera, Light.

**Modificadores essenciais para assets web:**

| Modificador | Descrição | Uso web |
|-------------|-----------|---------|
| **Subdivision Surface** | Suaviza a geometria subdividindo faces | Detalhamento — aplicar antes de exportar para controlar poly count |
| **Mirror** | Espelha o modelo em um eixo | Modelar metade, espelhar para simetria perfeita |
| **Boolean** | Combina ou subtrai formas | Criar furos, cortes, formas complexas |
| **Array** | Repete o objeto em padrão | Cercas, escadas, padrões repetitivos |
| **Decimate** | Reduz contagem de polígonos | **Essencial** para otimizar modelos para web |
| **Solidify** | Adiciona espessura a superfícies planas | Paredes, folhas, cascas |
| **Bevel** | Chanfra arestas | Suavizar arestas para iluminação realista |

**Contagem de polígonos recomendada:**

| Cenário | Triângulos (por modelo) | Observação |
|---------|------------------------|------------|
| VR mobile (Quest standalone) | 1K - 10K | Otimização extrema necessária |
| Desktop web | 10K - 100K | Equilíbrio qualidade/performance |
| Hero model (destaque) | 50K - 200K | Aceitável para objeto central da cena |
| Background/props | 500 - 5K | Mínimo necessário |
| Cena total (VR) | < 200K | Soma de todos os modelos |
| Cena total (desktop) | < 1M | Soma de todos os modelos |

> **Visualizar contagem:** No Blender, ative em Viewport Overlays > Statistics para ver vértices, arestas, faces e triângulos.

---

### Materiais PBR no Blender → Compatibilidade com glTF

O shader **Principled BSDF** do Blender mapeia diretamente para o modelo PBR do glTF:

| Principled BSDF (Blender) | glTF PBR | Three.js (MeshStandardMaterial) |
|----------------------------|----------|--------------------------------|
| Base Color | `baseColorFactor` / `baseColorTexture` | `color` / `map` |
| Metallic | `metallicFactor` | `metalness` |
| Roughness | `roughnessFactor` | `roughness` |
| Normal Map (nó Normal Map) | `normalTexture` | `normalMap` |
| Emission | `emissiveFactor` / `emissiveTexture` | `emissive` / `emissiveMap` |
| Alpha (Blend Mode) | `alphaMode` | `transparent` + `opacity` |
| Ambient Occlusion (nó separado) | `occlusionTexture` | `aoMap` |

**Setup de material compatível com glTF:**

```
Shader Editor (Blender):

[Image Texture: albedo.png] ──► Base Color
[Image Texture: normal.png] ──► [Normal Map] ──► Normal
[Image Texture: arm.png (R)] ──► Ambient Occlusion (separado)
[Image Texture: arm.png (G)] ──► Roughness
[Image Texture: arm.png (B)] ──► Metallic
[Image Texture: emissive.png] ──► Emission
```

> **Dica:** Combine Ambient Occlusion (R), Roughness (G) e Metallic (B) em uma única textura ORM/ARM para reduzir o número de texturas carregadas.

**O que NÃO é exportado para glTF:**
- Shaders customizados (Shader Editor complexo)
- Subsurface scattering (parcial com extensão)
- Nodes de procedural textures (Noise, Voronoi, etc.) — precisam ser baked
- Displacement sem bake

---

### UV Unwrapping e Texturização

UV unwrapping é o processo de "desdobrar" a superfície 3D em um mapa 2D para aplicar texturas.

**Métodos de unwrap no Blender:**

| Método | Quando usar |
|--------|-------------|
| **Smart UV Project** | Rápido, bom para objetos sem formas orgânicas |
| **Lightmap Pack** | Gerar UV sem sobreposição para bake de lightmaps |
| **Unwrap** (com seams) | Controle manual — melhor qualidade para modelos importantes |
| **Cube/Cylinder/Sphere Projection** | Objetos simples com forma correspondente |

**Workflow recomendado:**
1. Em Edit Mode, selecione todas as faces (`A`)
2. Marque seams nas arestas onde a textura deve "cortar" (`Ctrl+E > Mark Seam`)
3. Unwrap (`U > Unwrap`)
4. No UV Editor, ajuste as ilhas UV para minimizar distorção
5. Exporte o UV layout como referência para texturização

**Resoluções de textura recomendadas:**

| Resolução | Tamanho (PNG) | Tamanho (JPEG 80%) | Uso recomendado |
|-----------|---------------|---------------------|-----------------|
| 256×256 | ~50 KB | ~15 KB | Props pequenos, objetos distantes |
| 512×512 | ~200 KB | ~50 KB | Props médios, mobile VR |
| 1024×1024 | ~700 KB | ~150 KB | Objetos principais, desktop web |
| 2048×2048 | ~2.5 MB | ~500 KB | Hero models, close-up |
| 4096×4096 | ~10 MB | ~2 MB | Evitar para web (exceto backgrounds) |

---

### Rigging e Animação Básica

**Rigging** é o processo de criar um esqueleto (armature) para controlar a deformação do modelo.

**Workflow básico:**
1. Criar armature (`Shift+A > Armature`)
2. Entrar em Edit Mode e adicionar bones na hierarquia desejada
3. Parent do mesh ao armature (`Ctrl+P > Armature Deform > With Automatic Weights`)
4. Testar em Pose Mode movendo os bones

**Animação por keyframes:**
1. Selecionar o bone em Pose Mode
2. Na Timeline, posicionar no frame desejado
3. Transformar o bone (G/R/S) e inserir keyframe (`I > Location/Rotation/Scale`)
4. Repetir em outros frames para criar a animação

**Para exportação glTF:**
- Cada animação deve ser uma **Action** separada (NLA Editor)
- Marque as Actions com Fake User (escudo) para não perder
- Use o **NLA Editor** para organizar múltiplas ações (idle, walk, run, etc.)
- No export, ative "Group by NLA Track" para manter as animações separadas

> **Dica:** Plataformas como [Mixamo](https://www.mixamo.com) oferecem animações prontas gratuitamente. Importe o modelo, aplique animações e exporte como FBX. Depois converta para glTF no Blender ou com ferramentas CLI.

---

### Exportação glTF/GLB

**File > Export > glTF 2.0 (.glb/.gltf)**

**Configurações recomendadas para web:**

| Configuração | Valor recomendado | Motivo |
|--------------|-------------------|--------|
| **Format** | glTF Binary (.glb) | Arquivo único, menor overhead de rede |
| **Include > Limit to** | Selected Objects | Exportar apenas o necessário |
| **Transform > +Y Up** | Ativado | Padrão web (Three.js, A-Frame) |
| **Geometry > Apply Modifiers** | Ativado | Exportar geometria final |
| **Geometry > UVs** | Ativado | Necessário para texturas |
| **Geometry > Normals** | Ativado | Necessário para iluminação |
| **Geometry > Vertex Colors** | Se usado | Pode aumentar tamanho |
| **Geometry > Compression** | Draco (se desejado) | Reduz tamanho 5-10x |
| **Material** | Export | Exportar materiais PBR |
| **Images > Format** | WebP ou JPEG (qualidade 85) | Equilíbrio qualidade/tamanho |
| **Animation** | Ativado (se houver) | Exportar todas as actions |
| **Animation > Optimize Keyframes** | Ativado | Remove keyframes redundantes |

**Draco compression no export:**

Ative em Geometry > Compression e ajuste:
- **Quantization Position:** 14 (padrão, boa qualidade)
- **Quantization Normal:** 10
- **Quantization Tex Coord:** 12
- **Quantization Color:** 10
- **Compression Level:** 6 (0-10, maior = mais lento mas menor)

---

### Bake de Texturas

Bake converte materiais procedurais ou iluminação em texturas de imagem — essencial quando materiais usam nós procedurais (Noise, Voronoi, etc.) que o glTF não suporta.

**Workflow de bake (Cycles):**

1. **Preparar UV Map:** certifique-se que o modelo tem UVs sem sobreposição
2. **Criar imagem de destino:** no UV Editor, criar nova imagem (1024×1024 ou 2048×2048)
3. **No material:** adicionar nó Image Texture apontando para a imagem criada (mas **não** conectar a nenhuma saída)
4. **Selecionar o nó** Image Texture no Shader Editor
5. **Render Properties > Bake:**
   - Bake Type: Diffuse (apenas Color, sem Direct/Indirect)
   - Ou: Normal, Ambient Occlusion, Roughness, Emit
6. **Clicar Bake** — o Blender renderiza o material na textura
7. **Salvar a imagem** (Image > Save As)
8. **Repetir** para cada tipo de mapa necessário

**Tipos de bake e uso:**

| Tipo de bake | Resultado | Uso no glTF |
|--------------|-----------|-------------|
| **Diffuse** (Color only) | Mapa de albedo | `baseColorTexture` |
| **Normal** | Mapa de normais (tangent space) | `normalTexture` |
| **Ambient Occlusion** | Sombras suaves em cavidades | `occlusionTexture` |
| **Roughness** | Mapa de rugosidade | `metallicRoughnessTexture` (G) |
| **Emit** | Mapa de emissão | `emissiveTexture` |

> **Texture Atlas:** Para múltiplos objetos, combine todos os UVs em um único UV map e bake todos os objetos na mesma textura. Isso reduz draw calls ao usar um único material.

---

### Otimização de Modelos para Web

**Decimate Modifier:**

| Modo | Descrição | Quando usar |
|------|-----------|-------------|
| **Collapse** | Remove vértices mantendo a forma | Geral — ajuste Ratio (0.5 = metade dos polígonos) |
| **Un-Subdivide** | Reverte subdivisões uniformes | Modelos que foram subdivididos demais |
| **Planar** | Remove faces co-planares | Superfícies planas com faces desnecessárias |

**Checklist de otimização:**

1. **Remover vértices duplicados:** Edit Mode > `M > By Distance`
2. **Dissolver geometria desnecessária:** `X > Dissolve Edges/Faces`
3. **Aplicar Decimate** para atingir poly count alvo
4. **Unir objetos** com mesmo material: `Ctrl+J`
5. **Remover materiais não usados:** Properties > Material > limpar slots vazios
6. **Reduzir resolução de texturas** conforme distância do objeto
7. **Combinar texturas** em atlas quando possível
8. **Verificar normais:** Edit Mode > Mesh > Normals > Recalculate Outside

---

### Compressão com Draco e KTX2

**Draco (compressão de geometria):**

```bash
# Instalar gltf-pipeline
npm install -g gltf-pipeline

# Comprimir com Draco
gltf-pipeline -i modelo.glb -o modelo-draco.glb -d

# Ajustar nível de compressão
gltf-pipeline -i modelo.glb -o modelo-draco.glb -d \
  --draco.compressionLevel 7 \
  --draco.quantizePositionBits 14 \
  --draco.quantizeNormalBits 10 \
  --draco.quantizeTexcoordBits 12
```

**KTX2/Basis Universal (compressão de texturas):**

```bash
# Instalar gltf-transform
npm install -g @gltf-transform/cli

# Comprimir texturas para KTX2
gltf-transform ktx2 modelo.glb modelo-ktx2.glb --slots "baseColor,normal,emissive"

# Pipeline completo: otimizar + Draco + KTX2
gltf-transform optimize modelo.glb modelo-otimizado.glb
gltf-transform draco modelo-otimizado.glb modelo-draco.glb
gltf-transform ktx2 modelo-draco.glb modelo-final.glb
```

**Comparativo de compressão:**

| Técnica | Tipo | Redução típica | Decodificação | Qualidade |
|---------|------|----------------|---------------|-----------|
| **Draco** | Geometria (lossy) | 80-95% | WASM decoder (~35 KB) | Muito boa (configurável) |
| **Meshopt** | Geometria (lossless) | 60-80% | WASM decoder (~20 KB) | Sem perda |
| **KTX2/Basis** | Texturas (GPU) | 75-85% | GPU nativa | Boa (leve perda) |
| **WebP** | Texturas (CPU) | 30-50% vs JPEG | CPU (decode antes de upload) | Excelente |
| **gzip/brotli** | Transfer (server) | 20-50% adicional | Automático (HTTP) | Sem perda |

> **Pipeline recomendado para produção:**
> 1. Otimizar geometria no Blender (decimate, merge)
> 2. Exportar GLB sem compressão
> 3. Aplicar Draco ou Meshopt via gltf-transform
> 4. Comprimir texturas para KTX2 via gltf-transform
> 5. Validar com glTF Validator
> 6. Servir com Brotli/gzip no servidor

---

### Ferramentas Auxiliares

| Ferramenta | Instalação | Função |
|------------|------------|--------|
| **glTF Validator** | [Online](https://github.khronos.org/glTF-Validator/) ou `npm i -g gltf-validator` | Valida conformidade com a especificação glTF |
| **gltf-pipeline** | `npm i -g gltf-pipeline` | Conversão, Draco compression |
| **gltf-transform** | `npm i -g @gltf-transform/cli` | Otimização, Draco, Meshopt, KTX2, merge, prune |
| **glTF Viewer** | [gltf-viewer.donmccurdy.com](https://gltf-viewer.donmccurdy.com) | Visualizar modelos glTF no browser (Three.js) |
| **Babylon.js Sandbox** | [sandbox.babylonjs.com](https://sandbox.babylonjs.com) | Visualizar e inspecionar modelos |
| **SVGO** | `npm i -g svgo` | Otimizar arquivos SVG |

```bash
# Validar modelo
gltf-validator modelo.glb

# Inspecionar modelo (estatísticas)
gltf-transform inspect modelo.glb

# Otimizar tudo de uma vez
gltf-transform optimize input.glb output.glb \
  --compress draco \
  --texture-compress ktx2
```

---

## 16. Importação de Assets no Three.js e A-Frame

---

### GLTFLoader (Three.js)

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { LoadingManager } from 'three';

const manager = new LoadingManager();
manager.onProgress = (url, loaded, total) => {
  const progress = (loaded / total) * 100;
  document.getElementById('loading-bar').style.width = `${progress}%`;
  console.log(`Carregando: ${progress.toFixed(0)}%`);
};
manager.onLoad = () => {
  document.getElementById('loading-screen').style.display = 'none';
};

const loader = new GLTFLoader(manager);

loader.load(
  'models/cena.glb',
  (gltf) => {
    const model = gltf.scene;
    model.position.set(0, 0, 0);
    model.scale.setScalar(1);

    model.traverse((child) => {
      if (child.isMesh) {
        child.castShadow = true;
        child.receiveShadow = true;
      }
    });

    scene.add(model);

    if (gltf.animations.length > 0) {
      const mixer = new THREE.AnimationMixer(model);
      gltf.animations.forEach((clip) => {
        mixer.clipAction(clip).play();
      });

      function animate() {
        requestAnimationFrame(animate);
        mixer.update(clock.getDelta());
        renderer.render(scene, camera);
      }
      animate();
    }
  },
  (progress) => {
    console.log(`${(progress.loaded / progress.total * 100).toFixed(0)}% carregado`);
  },
  (error) => {
    console.error('Erro ao carregar modelo:', error);
  }
);
```

**Acessando partes do modelo carregado:**

```javascript
loader.load('models/cena.glb', (gltf) => {
  const model = gltf.scene;

  const porta = model.getObjectByName('Porta');
  if (porta) {
    porta.material = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
  }

  const animacoes = gltf.animations;
  console.log('Animações disponíveis:', animacoes.map(a => a.name));

  const cameras = [];
  model.traverse((child) => {
    if (child.isCamera) cameras.push(child);
  });

  scene.add(model);
});
```

---

### DRACOLoader — Modelos Comprimidos

```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');
dracoLoader.setDecoderConfig({ type: 'js' });
dracoLoader.preload();

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load('models/cena-draco.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

> **Dica:** Os decoders Draco podem ser servidos localmente copiando os arquivos de `node_modules/three/examples/jsm/libs/draco/` para a pasta `public/` do projeto.

---

### KTX2Loader — Texturas Comprimidas

```javascript
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js';

const ktx2Loader = new KTX2Loader();
ktx2Loader.setTranscoderPath('https://cdn.jsdelivr.net/npm/three/examples/jsm/libs/basis/');
ktx2Loader.detectSupport(renderer);

const gltfLoader = new GLTFLoader();
gltfLoader.setKTX2Loader(ktx2Loader);
gltfLoader.setMeshoptDecoder(MeshoptDecoder);

gltfLoader.load('models/cena-ktx2.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

---

### Carregamento de Assets no A-Frame

**glTF:**

```html
<a-scene>
  <a-assets>
    <a-asset-item id="robo" src="models/robo.glb"></a-asset-item>
  </a-assets>

  <a-gltf-model src="#robo" position="0 0 -3" scale="0.5 0.5 0.5"
                 animation-mixer="clip: idle; loop: repeat">
  </a-gltf-model>
</a-scene>
```

> O componente `animation-mixer` vem do pacote `aframe-extras`. Instale via `<script src="https://unpkg.com/aframe-extras/dist/aframe-extras.min.js"></script>`.

**Controle de animações via JavaScript:**

```javascript
const modelo = document.querySelector('a-gltf-model');

modelo.addEventListener('model-loaded', () => {
  modelo.setAttribute('animation-mixer', { clip: 'walk', loop: 'repeat' });
});

document.getElementById('btn-idle').addEventListener('click', () => {
  modelo.setAttribute('animation-mixer', { clip: 'idle' });
});
document.getElementById('btn-walk').addEventListener('click', () => {
  modelo.setAttribute('animation-mixer', { clip: 'walk' });
});
```

---

### Lazy Loading e Streaming

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const placeholder = entry.target;
      const modelUrl = placeholder.dataset.model;

      const loader = new GLTFLoader();
      loader.setDRACOLoader(dracoLoader);

      loader.load(modelUrl, (gltf) => {
        const model = gltf.scene;
        model.position.copy(placeholder.position);
        model.rotation.copy(placeholder.rotation);
        model.scale.copy(placeholder.scale);

        scene.remove(placeholder);
        scene.add(model);
      });

      observer.unobserve(placeholder);
    }
  });
}, { threshold: 0.1 });

const placeholder = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshBasicMaterial({ color: 0xcccccc, wireframe: true })
);
placeholder.position.set(0, 0, -10);
placeholder.dataset = { model: 'models/detalhe.glb' };
scene.add(placeholder);
```

**Progressive loading (low-res → high-res):**

```javascript
async function loadProgressive(lowUrl, highUrl, position) {
  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoLoader);

  const lowRes = await loader.loadAsync(lowUrl);
  lowRes.scene.position.copy(position);
  scene.add(lowRes.scene);

  const highRes = await loader.loadAsync(highUrl);
  highRes.scene.position.copy(position);
  scene.remove(lowRes.scene);
  scene.add(highRes.scene);

  lowRes.scene.traverse((child) => {
    if (child.isMesh) {
      child.geometry.dispose();
      child.material.dispose();
    }
  });
}

loadProgressive(
  'models/personagem-low.glb',
  'models/personagem-high.glb',
  new THREE.Vector3(0, 0, -5)
);
```

---

### Otimização de Assets Carregados

**Instancing de modelos repetidos:**

```javascript
const loader = new GLTFLoader();
loader.load('models/arvore.glb', (gltf) => {
  let templateGeometry, templateMaterial;

  gltf.scene.traverse((child) => {
    if (child.isMesh) {
      templateGeometry = child.geometry;
      templateMaterial = child.material;
    }
  });

  const count = 500;
  const instancedMesh = new THREE.InstancedMesh(templateGeometry, templateMaterial, count);
  const dummy = new THREE.Object3D();

  for (let i = 0; i < count; i++) {
    dummy.position.set(
      (Math.random() - 0.5) * 100,
      0,
      (Math.random() - 0.5) * 100
    );
    dummy.rotation.y = Math.random() * Math.PI * 2;
    const scale = 0.8 + Math.random() * 0.4;
    dummy.scale.setScalar(scale);
    dummy.updateMatrix();
    instancedMesh.setMatrixAt(i, dummy.matrix);
  }

  instancedMesh.castShadow = true;
  instancedMesh.receiveShadow = true;
  scene.add(instancedMesh);
});
```

---

### Assets de Outras Fontes

| Fonte | Licença | Formatos | Conteúdo |
|-------|---------|----------|----------|
| **[Poly Haven](https://polyhaven.com)** | CC0 (domínio público) | glTF, FBX, Blend | HDRIs, texturas PBR, modelos 3D |
| **[Sketchfab](https://sketchfab.com)** | Variável (CC, pago) | glTF, FBX, OBJ, USDZ | Milhões de modelos (download via UI ou API) |
| **[Mixamo](https://www.mixamo.com)** | Grátis (uso comercial) | FBX, COLLADA | Animações de personagens (auto-rigging) |
| **[Kenney](https://kenney.nl)** | CC0 | glTF, OBJ, FBX | Assets de jogos (low-poly) |
| **[Quaternius](https://quaternius.com)** | CC0 | FBX, OBJ | Assets low-poly para jogos |
| **[ambientCG](https://ambientcg.com)** | CC0 | PNG, EXR | Texturas PBR e HDRIs |
| **[OpenGameArt](https://opengameart.org)** | Variável (CC, GPL) | Diversos | Assets 2D e 3D para jogos |

> **Workflow com Mixamo:**
> 1. Faça upload do seu modelo (FBX ou OBJ) no Mixamo
> 2. O Mixamo faz auto-rigging (esqueleto automático)
> 3. Escolha animações (idle, walk, run, jump, etc.)
> 4. Exporte como FBX
> 5. No Blender: Import FBX → Export glTF/GLB

---

### Licenciamento e Atribuição

| Licença | Uso comercial | Modificação | Atribuição | Share-alike |
|---------|---------------|-------------|------------|-------------|
| **CC0** | Sim | Sim | Não | Não |
| **CC-BY** | Sim | Sim | **Sim** | Não |
| **CC-BY-SA** | Sim | Sim | **Sim** | **Sim** (mesma licença) |
| **CC-BY-NC** | **Não** | Sim | **Sim** | Não |
| **CC-BY-NC-SA** | **Não** | Sim | **Sim** | **Sim** |

**Como atribuir assets CC-BY em projetos web:**

```html
<!-- No HTML (footer ou página de créditos) -->
<footer>
  <h3>Créditos de Assets 3D</h3>
  <ul>
    <li>"Cadeira Moderna" por João Silva — <a href="https://sketchfab.com/...">Sketchfab</a> (CC-BY 4.0)</li>
    <li>HDRI "Studio Lighting" — <a href="https://polyhaven.com/...">Poly Haven</a> (CC0)</li>
    <li>Animações — <a href="https://www.mixamo.com">Mixamo</a></li>
  </ul>
</footer>
```

---

## 17. Performance e Otimização Geral

---

### GPU vs CPU — Entendendo o Gargalo

| Sintoma | Gargalo provável | Solução |
|---------|-------------------|---------|
| FPS cai com muitos objetos, mesmo simples | **CPU** — draw calls excessivos | Merge geometries, instancing, frustum culling |
| FPS cai com objetos grandes/complexos, poucos objetos | **GPU** — fill rate/shader complexity | Simplificar materiais, reduzir resolução, LOD |
| FPS cai ao mover câmera rapidamente | **GPU** — overdraw, transparência | Ordenar objetos, reduzir transparência |
| FPS cai com muitas animações | **CPU** — AnimationMixer processing | Simplificar animações, LOD de animação |
| FPS cai com física | **CPU** — simulação de física | Reduzir corpos físicos, simplificar colliders |
| Carregamento lento | **CPU/Rede** — parsing, decodificação | Compressão Draco/KTX2, lazy loading |
| Uso de memória alto | **GPU** — texturas não comprimidas | KTX2, reduzir resolução, dispose() |

---

### Compressão de Texturas para GPU

| Formato | Plataforma | Qualidade | Tamanho vs PNG | GPU decode |
|---------|-----------|-----------|----------------|------------|
| **KTX2/Basis Universal** | Cross-platform | Boa | 75-85% menor | Sim (transcoding) |
| **BC1-BC7** (DXT) | Desktop (Windows/Mac/Linux) | Excelente | 75-85% menor | Sim (nativo) |
| **ETC2** | Mobile (Android/iOS), WebGL2 | Boa | 75% menor | Sim (nativo) |
| **ASTC** | Mobile moderno | Excelente (variável) | 80-90% menor | Sim (nativo) |
| **WebP** | Browser (CPU decode) | Excelente | 30-50% vs JPEG | Não — CPU decode |

> **KTX2 com Basis Universal** é a melhor escolha para web: o Basis transcoder converte automaticamente para o formato nativo da GPU do dispositivo (BC para desktop, ETC2/ASTC para mobile).

---

### Compressão de Geometria

| Ferramenta | Tipo | Compressão | Decode speed | Streaming |
|------------|------|------------|--------------|-----------|
| **Draco** | Lossy | 80-95% | Moderado (WASM) | Não |
| **Meshopt** | Lossless | 60-80% | Rápido (WASM) | Sim |
| **Quantização** (glTF) | Lossy | 30-50% | Instantâneo | Sim |

> **Draco vs Meshopt:** Draco oferece maior compressão mas é lossy e não suporta streaming progressivo. Meshopt é lossless, mais rápido para decodificar e suporta streaming. Para modelos onde precisão geométrica importa, prefira Meshopt.

---

### Culling

```javascript
// LOD (Level of Detail) — Three.js
const lod = new THREE.LOD();

const highDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 64, 64),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);
const medDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 16, 16),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);
const lowDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 4, 4),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);

lod.addLevel(highDetail, 0);    // < 10 unidades de distância
lod.addLevel(medDetail, 10);    // 10-30 unidades
lod.addLevel(lowDetail, 30);    // > 30 unidades

scene.add(lod);
```

> **Frustum Culling** é automático no Three.js e Babylon.js — objetos fora do campo de visão da câmera não são renderizados. Certifique-se de que `mesh.frustumCulled = true` (padrão).

---

### Object Pooling

```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];

    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  get() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createFn();
  }

  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

const particlePool = new ObjectPool(
  () => new THREE.Mesh(
    new THREE.SphereGeometry(0.05, 8, 8),
    new THREE.MeshBasicMaterial({ color: 0xffff00 })
  ),
  (mesh) => {
    mesh.visible = false;
    mesh.position.set(0, 0, 0);
    mesh.scale.setScalar(1);
  },
  100
);
```

---

### Progressive Loading

```javascript
async function initScene() {
  const loadingBar = document.getElementById('loading-bar');

  loadingBar.textContent = 'Carregando ambiente...';
  const envTexture = await new THREE.RGBELoader().loadAsync('hdr/ambiente.hdr');
  scene.environment = envTexture;
  scene.background = envTexture;

  loadingBar.textContent = 'Carregando modelo principal...';
  const mainModel = await gltfLoader.loadAsync('models/principal-low.glb');
  scene.add(mainModel.scene);

  loadingBar.textContent = 'Pronto!';
  document.getElementById('loading-screen').classList.add('hidden');

  gltfLoader.loadAsync('models/principal-high.glb').then((highRes) => {
    scene.remove(mainModel.scene);
    scene.add(highRes.scene);
    disposeModel(mainModel.scene);
  });

  gltfLoader.loadAsync('models/detalhes.glb').then((details) => {
    scene.add(details.scene);
  });
}
```

---

### Adaptação para Mobile

```javascript
function getQualityLevel() {
  const gl = renderer.getContext();
  const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
  const gpu = debugInfo ? gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL) : '';

  const isMobile = /Android|iPhone|iPad/i.test(navigator.userAgent);
  const isLowEnd = /Mali|Adreno [23]|PowerVR/i.test(gpu);

  if (isLowEnd || (isMobile && window.devicePixelRatio <= 2)) return 'low';
  if (isMobile) return 'medium';
  return 'high';
}

const quality = getQualityLevel();

const qualitySettings = {
  low:    { pixelRatio: 1,   shadows: false, antialias: false, maxLights: 2 },
  medium: { pixelRatio: 1.5, shadows: true,  antialias: false, maxLights: 4 },
  high:   { pixelRatio: 2,   shadows: true,  antialias: true,  maxLights: 8 }
};

const settings = qualitySettings[quality];
renderer.setPixelRatio(settings.pixelRatio);
renderer.shadowMap.enabled = settings.shadows;
```

---

### Métricas e Metas

| Métrica | Desktop | Mobile | VR |
|---------|---------|--------|----|
| **FPS** | 60 | 30-60 | 72-90 |
| **Draw calls** | < 500 | < 100 | < 100 |
| **Triângulos (cena)** | < 1M | < 200K | < 200K |
| **Texturas (VRAM)** | < 256 MB | < 128 MB | < 128 MB |
| **Tempo de carregamento** | < 5s | < 8s | < 10s |
| **Tamanho total assets** | < 20 MB | < 10 MB | < 15 MB |

**Ferramentas de medição:**

```javascript
import Stats from 'three/addons/libs/stats.module.js';

const stats = new Stats();
stats.showPanel(0); // 0: FPS, 1: MS, 2: MB
document.body.appendChild(stats.dom);

function animate() {
  stats.begin();
  renderer.render(scene, camera);
  stats.end();
  requestAnimationFrame(animate);
}
```

> **Chrome DevTools > Performance:** grave uma sessão, analise o flame chart para identificar gargalos JavaScript. **Spector.js:** extensão que captura chamadas WebGL frame a frame para identificar draw calls redundantes.

---

## 18. Acessibilidade em Conteúdo Gráfico

Conteúdo gráfico interativo (Canvas, SVG, WebGL) apresenta desafios específicos de acessibilidade. Garantir que todos os usuários possam acessar o conteúdo é tanto uma boa prática quanto um requisito legal em muitos contextos.

---

### Canvas e Acessibilidade

Canvas é opaco para tecnologias assistivas — screen readers não conseguem ler seu conteúdo.

**Estratégias de acessibilidade:**

```html
<!-- Fallback textual dentro do canvas -->
<canvas id="grafico" width="600" height="400" role="img"
        aria-label="Gráfico de vendas mensais mostrando crescimento de 20% no último trimestre">
  <!-- Conteúdo de fallback para screen readers e navegadores sem Canvas -->
  <table>
    <caption>Vendas mensais</caption>
    <tr><th>Mês</th><th>Vendas</th></tr>
    <tr><td>Janeiro</td><td>R$ 45.000</td></tr>
    <tr><td>Fevereiro</td><td>R$ 52.000</td></tr>
    <tr><td>Março</td><td>R$ 61.000</td></tr>
  </table>
</canvas>
```

**Canvas interativo com suporte a teclado:**

```javascript
const canvas = document.getElementById('editor');
canvas.setAttribute('tabindex', '0');
canvas.setAttribute('role', 'application');
canvas.setAttribute('aria-label', 'Editor de desenho. Use setas para mover, Enter para desenhar.');

const liveRegion = document.createElement('div');
liveRegion.setAttribute('aria-live', 'polite');
liveRegion.setAttribute('class', 'sr-only');
document.body.appendChild(liveRegion);

canvas.addEventListener('keydown', (e) => {
  switch (e.key) {
    case 'ArrowUp':    cursor.y -= 10; break;
    case 'ArrowDown':  cursor.y += 10; break;
    case 'ArrowLeft':  cursor.x -= 10; break;
    case 'ArrowRight': cursor.x += 10; break;
    case 'Enter':
      draw(cursor.x, cursor.y);
      liveRegion.textContent = `Ponto desenhado em (${cursor.x}, ${cursor.y})`;
      break;
  }
  e.preventDefault();
});
```

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
```

---

### SVG e Screen Readers

SVG é intrinsecamente acessível — cada elemento é parte do DOM:

```html
<svg role="img" aria-labelledby="titulo-grafico desc-grafico"
     viewBox="0 0 400 300" xmlns="http://www.w3.org/2000/svg">

  <title id="titulo-grafico">Distribuição de Vendas por Região</title>
  <desc id="desc-grafico">
    Gráfico de pizza mostrando: Sudeste 45%, Sul 25%, Nordeste 20%, Norte 10%.
  </desc>

  <!-- Fatia Sudeste -->
  <path d="M200,150 L200,20 A130,130 0 0,1 340,100 Z"
        fill="#4FC3F7" role="graphics-symbol" aria-roledescription="fatia">
    <title>Sudeste: 45% (R$ 2.3 milhões)</title>
  </path>

  <!-- Fatia Sul -->
  <path d="M200,150 L340,100 A130,130 0 0,1 290,270 Z"
        fill="#81C784" role="graphics-symbol" aria-roledescription="fatia">
    <title>Sul: 25% (R$ 1.2 milhões)</title>
  </path>

  <!-- Elementos decorativos ocultos de screen readers -->
  <line x1="0" y1="295" x2="400" y2="295" stroke="#ccc" aria-hidden="true" />
</svg>
```

---

### WebGL/3D — Alternativas Textuais e Navegação

```html
<div class="cena-3d-container">
  <canvas id="cena-3d" aria-label="Visualização 3D do produto. Use controles abaixo para interagir."
          role="img"></canvas>

  <!-- Controles acessíveis ao lado do canvas -->
  <div class="controles" role="toolbar" aria-label="Controles do modelo 3D">
    <button onclick="rotateLeft()" aria-label="Rotacionar modelo para a esquerda">
      ← Rotacionar
    </button>
    <button onclick="rotateRight()" aria-label="Rotacionar modelo para a direita">
      Rotacionar →
    </button>
    <button onclick="zoomIn()" aria-label="Aproximar modelo">+ Zoom</button>
    <button onclick="zoomOut()" aria-label="Afastar modelo">- Zoom</button>
    <button onclick="resetView()" aria-label="Resetar visualização">Reset</button>
  </div>

  <!-- Descrição alternativa para screen readers -->
  <div class="sr-only" aria-live="polite" id="descricao-3d">
    Modelo 3D de uma cadeira moderna de escritório em couro preto.
    Possui base giratória com 5 rodízios, apoio de braço ajustável
    e encosto reclinável com suporte lombar.
  </div>
</div>
```

---

### Preferências do Usuário

```javascript
// prefers-reduced-motion — desativar ou simplificar animações
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

if (prefersReducedMotion) {
  renderer.setAnimationLoop(null);
  renderer.render(scene, camera);
} else {
  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  }
  animate();
}

// Monitorar mudanças em tempo real
window.matchMedia('(prefers-reduced-motion: reduce)').addEventListener('change', (e) => {
  if (e.matches) {
    particleSystem.visible = false;
    autoRotate = false;
  } else {
    particleSystem.visible = true;
    autoRotate = true;
  }
});
```

```javascript
// prefers-color-scheme — adaptar tema
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

scene.background = new THREE.Color(prefersDark ? 0x1a1a2e : 0xf0f0f0);
scene.fog = new THREE.Fog(prefersDark ? 0x1a1a2e : 0xf0f0f0, 10, 50);
```

```css
/* Canvas — respeitar preferências no CSS */
canvas {
  background: #f0f0f0;
}

@media (prefers-color-scheme: dark) {
  canvas { background: #1a1a2e; }
}

@media (prefers-reduced-motion: reduce) {
  canvas { animation: none !important; }
  svg * { animation: none !important; transition: none !important; }
}

@media (prefers-contrast: more) {
  svg text { font-weight: bold; }
  svg [stroke] { stroke-width: 2; }
}
```

> **WCAG 2.2:** Para conteúdo gráfico interativo, forneça sempre uma alternativa textual (`aria-label`, `<title>`, fallback content), suporte navegação por teclado e respeite as preferências do usuário (motion, color scheme, contrast).

---

## Exemplo Completo — Catálogo de Vídeos com Three.js

Este exemplo demonstra como usar Three.js para criar uma interface de catálogo de vídeos estilo plataforma de streaming. Cada card é um grupo 3D composto por um plano com textura de vídeo (preview automático ao hover), título, descrição, e indicadores de views e likes — tudo renderizado no espaço 3D com OrbitControls e raycasting para interatividade.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Catálogo de Vídeos — Three.js</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #0d0d1a; overflow: hidden; font-family: system-ui, sans-serif; }
    canvas { display: block; }

    #info-overlay {
      position: fixed; top: 16px; left: 50%;
      transform: translateX(-50%);
      color: #aaa; font-size: 14px;
      pointer-events: none; z-index: 10;
    }
  </style>
</head>
<body>
  <div id="info-overlay">Passe o mouse sobre um card para ver a prévia do vídeo</div>

  <!-- Importmap para Three.js via CDN -->
  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // =========================================================
    // Dados do catálogo
    // =========================================================
    const catalogo = [
      {
        titulo: 'Introdução ao Three.js',
        descricao: 'Aprenda os fundamentos de cenas 3D na web.',
        views: 124500, likes: 8920,
        cor: '#e74c3c',
        video: 'https://www.w3schools.com/html/mov_bbb.mp4'
      },
      {
        titulo: 'CSS Grid na Prática',
        descricao: 'Layouts responsivos com CSS Grid moderno.',
        views: 89200, likes: 5430,
        cor: '#3498db',
        video: 'https://www.w3schools.com/html/movie.mp4'
      },
      {
        titulo: 'React 19 — Novidades',
        descricao: 'Server Components, Actions e novos hooks.',
        views: 203800, likes: 15700,
        cor: '#2ecc71',
        video: 'https://www.w3schools.com/html/mov_bbb.mp4'
      },
      {
        titulo: 'WebGL Shaders',
        descricao: 'Vertex e fragment shaders do zero.',
        views: 67400, likes: 4100,
        cor: '#f39c12',
        video: 'https://www.w3schools.com/html/movie.mp4'
      },
      {
        titulo: 'Node.js + WebSockets',
        descricao: 'Comunicação em tempo real com Socket.io.',
        views: 145600, likes: 9800,
        cor: '#9b59b6',
        video: 'https://www.w3schools.com/html/mov_bbb.mp4'
      },
      {
        titulo: 'Docker para Devs',
        descricao: 'Containers, Compose e deploy simplificado.',
        views: 178300, likes: 12400,
        cor: '#1abc9c',
        video: 'https://www.w3schools.com/html/movie.mp4'
      },
    ];

    // =========================================================
    // Setup da cena
    // =========================================================
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0d0d1a);
    scene.fog = new THREE.FogExp2(0x0d0d1a, 0.06);

    const camera = new THREE.PerspectiveCamera(50, innerWidth / innerHeight, 0.1, 100);
    camera.position.set(0, 1, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(innerWidth, innerHeight);
    renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.08;
    controls.maxPolarAngle = Math.PI / 1.8;
    controls.minDistance = 4;
    controls.maxDistance = 18;

    // =========================================================
    // Iluminação
    // =========================================================
    scene.add(new THREE.AmbientLight(0x334455, 1.2));

    const dirLight = new THREE.DirectionalLight(0xffffff, 1.5);
    dirLight.position.set(5, 8, 5);
    dirLight.castShadow = true;
    dirLight.shadow.mapSize.set(1024, 1024);
    scene.add(dirLight);

    const fillLight = new THREE.PointLight(0x4488ff, 0.6, 30);
    fillLight.position.set(-5, 3, 2);
    scene.add(fillLight);

    // Chão sutil
    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(40, 40),
      new THREE.MeshStandardMaterial({ color: 0x0a0a14, roughness: 0.9 })
    );
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = -2.2;
    floor.receiveShadow = true;
    scene.add(floor);

    // =========================================================
    // Utilitários de texto com Canvas 2D → textura Three.js
    // =========================================================
    function createTextTexture(text, opts = {}) {
      const {
        width = 512, height = 64,
        fontSize = 28, fontWeight = 'normal',
        color = '#ffffff', align = 'left',
        paddingX = 12,
        bgColor = null,
        maxChars = 40,
      } = opts;

      const canvas = document.createElement('canvas');
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext('2d');

      if (bgColor) {
        ctx.fillStyle = bgColor;
        ctx.roundRect(0, 0, width, height, 8);
        ctx.fill();
      }

      ctx.fillStyle = color;
      ctx.font = `${fontWeight} ${fontSize}px system-ui, Arial, sans-serif`;
      ctx.textBaseline = 'middle';
      ctx.textAlign = align;

      let display = text;
      if (display.length > maxChars) display = display.substring(0, maxChars - 1) + '…';

      const x = align === 'center' ? width / 2 : paddingX;
      ctx.fillText(display, x, height / 2);

      const tex = new THREE.CanvasTexture(canvas);
      tex.colorSpace = THREE.SRGBColorSpace;
      return tex;
    }

    function formatNumber(n) {
      if (n >= 1_000_000) return (n / 1_000_000).toFixed(1) + 'M';
      if (n >= 1_000) return (n / 1_000).toFixed(1) + 'K';
      return String(n);
    }

    // =========================================================
    // Construção de um card de vídeo
    // =========================================================
    const CARD_W = 3.2;
    const CARD_H = 3.6;
    const VIDEO_H = 1.8;
    const cards = [];

    function createVideoCard(data, index) {
      const group = new THREE.Group();
      group.userData = { data, index, hovering: false, video: null };

      // --- Fundo do card (retângulo arredondado simulado) ---
      const bgGeo = new THREE.PlaneGeometry(CARD_W, CARD_H);
      const bgMat = new THREE.MeshStandardMaterial({
        color: 0x1a1a2e,
        roughness: 0.8,
        metalness: 0.1,
      });
      const bg = new THREE.Mesh(bgGeo, bgMat);
      bg.castShadow = true;
      bg.receiveShadow = true;
      group.add(bg);

      // --- Borda superior colorida (accent) ---
      const accentGeo = new THREE.PlaneGeometry(CARD_W, 0.06);
      const accentMat = new THREE.MeshBasicMaterial({ color: data.cor });
      const accent = new THREE.Mesh(accentGeo, accentMat);
      accent.position.set(0, CARD_H / 2 - 0.03, 0.01);
      group.add(accent);

      // --- Área de vídeo (thumbnail / preview) ---
      // Imagem placeholder (gradiente escuro)
      const thumbCanvas = document.createElement('canvas');
      thumbCanvas.width = 640;
      thumbCanvas.height = 360;
      const tctx = thumbCanvas.getContext('2d');

      // Gradiente de fundo
      const grad = tctx.createLinearGradient(0, 0, 640, 360);
      grad.addColorStop(0, '#1a1a2e');
      grad.addColorStop(1, data.cor + '44');
      tctx.fillStyle = grad;
      tctx.fillRect(0, 0, 640, 360);

      // Ícone de play
      tctx.fillStyle = 'rgba(255,255,255,0.15)';
      tctx.beginPath();
      tctx.moveTo(280, 130);
      tctx.lineTo(280, 230);
      tctx.lineTo(380, 180);
      tctx.closePath();
      tctx.fill();

      const thumbTex = new THREE.CanvasTexture(thumbCanvas);
      thumbTex.colorSpace = THREE.SRGBColorSpace;

      const videoGeo = new THREE.PlaneGeometry(CARD_W - 0.2, VIDEO_H);
      const videoMat = new THREE.MeshBasicMaterial({ map: thumbTex });
      const videoMesh = new THREE.Mesh(videoGeo, videoMat);
      videoMesh.position.set(0, (CARD_H / 2) - 0.1 - VIDEO_H / 2, 0.02);
      group.add(videoMesh);
      group.userData.videoMesh = videoMesh;
      group.userData.thumbTex = thumbTex;

      // --- Título ---
      const titleTex = createTextTexture(data.titulo, {
        width: 512, height: 48,
        fontSize: 30, fontWeight: 'bold',
        color: '#ffffff',
      });
      const titleMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.3, 0.35),
        new THREE.MeshBasicMaterial({ map: titleTex, transparent: true })
      );
      titleMesh.position.set(0, (CARD_H / 2) - VIDEO_H - 0.4, 0.02);
      group.add(titleMesh);

      // --- Descrição ---
      const descTex = createTextTexture(data.descricao, {
        width: 512, height: 40,
        fontSize: 20,
        color: '#999999',
      });
      const descMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.3, 0.28),
        new THREE.MeshBasicMaterial({ map: descTex, transparent: true })
      );
      descMesh.position.set(0, (CARD_H / 2) - VIDEO_H - 0.75, 0.02);
      group.add(descMesh);

      // --- Views e Likes (ícone textual + número) ---
      const statsText = `👁 ${formatNumber(data.views)}    ♥ ${formatNumber(data.likes)}`;
      const statsTex = createTextTexture(statsText, {
        width: 512, height: 36,
        fontSize: 22,
        color: '#6a6a8a',
      });
      const statsMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.3, 0.25),
        new THREE.MeshBasicMaterial({ map: statsTex, transparent: true })
      );
      statsMesh.position.set(0, (CARD_H / 2) - VIDEO_H - 1.10, 0.02);
      group.add(statsMesh);

      // --- Borda inferior sutil ---
      const bottomLine = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.4, 0.015),
        new THREE.MeshBasicMaterial({ color: 0x2a2a4a })
      );
      bottomLine.position.set(0, -CARD_H / 2 + 0.25, 0.02);
      group.add(bottomLine);

      return group;
    }

    // =========================================================
    // Posicionar cards em grid (3 colunas x 2 linhas)
    // =========================================================
    const COLS = 3;
    const GAP_X = 3.6;
    const GAP_Y = 4.0;

    catalogo.forEach((data, i) => {
      const col = i % COLS;
      const row = Math.floor(i / COLS);

      const x = (col - (COLS - 1) / 2) * GAP_X;
      const y = -row * GAP_Y + 1.5;

      const card = createVideoCard(data, i);
      card.position.set(x, y, 0);
      scene.add(card);
      cards.push(card);
    });

    // =========================================================
    // Raycasting — hover e vídeo preview
    // =========================================================
    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let hoveredCard = null;

    function onPointerMove(e) {
      pointer.x = (e.clientX / innerWidth) * 2 - 1;
      pointer.y = -(e.clientY / innerHeight) * 2 + 1;
    }
    window.addEventListener('pointermove', onPointerMove);

    function startVideoPreview(card) {
      const { data, videoMesh, thumbTex } = card.userData;
      if (card.userData.video) return;

      const video = document.createElement('video');
      video.src = data.video;
      video.crossOrigin = 'anonymous';
      video.loop = true;
      video.muted = true;
      video.playsInline = true;
      video.play().catch(() => {});

      const videoTex = new THREE.VideoTexture(video);
      videoTex.colorSpace = THREE.SRGBColorSpace;
      videoMesh.material.map = videoTex;
      videoMesh.material.needsUpdate = true;

      card.userData.video = video;
      card.userData.videoTex = videoTex;
    }

    function stopVideoPreview(card) {
      const { video, videoTex, videoMesh, thumbTex } = card.userData;
      if (!video) return;

      video.pause();
      video.src = '';
      video.load();
      videoTex.dispose();

      videoMesh.material.map = thumbTex;
      videoMesh.material.needsUpdate = true;

      card.userData.video = null;
      card.userData.videoTex = null;
    }

    // =========================================================
    // Animação
    // =========================================================
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);
      const dt = clock.getDelta();
      controls.update();

      // Raycasting
      raycaster.setFromCamera(pointer, camera);
      const allMeshes = [];
      cards.forEach(c => c.traverse(child => {
        if (child.isMesh) allMeshes.push(child);
      }));
      const intersects = raycaster.intersectObjects(allMeshes);

      // Determinar qual card está sob o cursor
      let hitCard = null;
      if (intersects.length > 0) {
        let obj = intersects[0].object;
        while (obj.parent && !cards.includes(obj)) obj = obj.parent;
        if (cards.includes(obj)) hitCard = obj;
      }

      // Gerenciar hover
      if (hitCard !== hoveredCard) {
        // Sair do card anterior
        if (hoveredCard) {
          hoveredCard.userData.hovering = false;
          stopVideoPreview(hoveredCard);
        }
        // Entrar no novo card
        if (hitCard) {
          hitCard.userData.hovering = true;
          startVideoPreview(hitCard);
        }
        hoveredCard = hitCard;
      }

      // Animar cards (escala suave no hover, leve flutuação)
      const time = clock.elapsedTime;
      cards.forEach((card, i) => {
        const target = card.userData.hovering ? 1.06 : 1.0;
        card.scale.lerp(
          new THREE.Vector3(target, target, target),
          dt * 8
        );

        const targetZ = card.userData.hovering ? 0.3 : 0;
        card.position.z = THREE.MathUtils.lerp(card.position.z, targetZ, dt * 8);

        // Flutuação sutil
        card.position.y += Math.sin(time * 0.8 + i * 1.2) * 0.0004;
      });

      // Cursor
      document.body.style.cursor = hitCard ? 'pointer' : 'default';

      renderer.render(scene, camera);
    }

    animate();

    // =========================================================
    // Responsividade
    // =========================================================
    window.addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth, innerHeight);
    });
  </script>
</body>
</html>
```

**O que este exemplo demonstra:**

| Conceito Three.js | Uso no exemplo |
|-------------------|----------------|
| **Scene, Camera, Renderer** | Setup completo com fog, sombras PCF e pixel ratio adaptativo |
| **Group** | Cada card é um `Group` com múltiplos meshes filhos |
| **PlaneGeometry + MeshStandardMaterial** | Fundo dos cards com sombras |
| **CanvasTexture** | Texto (título, descrição, stats) renderizado via Canvas 2D → textura |
| **VideoTexture** | Preview de vídeo como textura em tempo real |
| **Raycaster** | Detecção de hover para ativar/desativar preview |
| **OrbitControls** | Navegação orbital com damping e limites |
| **Iluminação** | Ambient + Directional (com sombra) + Point (fill) |
| **Animação** | Lerp de escala e posição Z no hover, flutuação sutil contínua |
| **Responsividade** | Atualização de aspect ratio e tamanho no resize |

> **Para usar com vídeos reais:** substitua as URLs em `catalogo[].video` pelos caminhos dos seus arquivos `.mp4`. Os vídeos precisam estar no mesmo domínio ou ter CORS habilitado (`crossOrigin: 'anonymous'`). Para thumbnails reais, substitua o canvas de placeholder por uma imagem carregada com `TextureLoader`.

---

## Exemplo Completo — Kanban Board 3D com Three.js

Um quadro Kanban tridimensional com três colunas (A Fazer, Em Progresso, Concluído), cards com título, responsável, prioridade e tags — com drag and drop entre colunas via raycasting, animações suaves e visual estilo glassmorphism.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Kanban Board 3D — Three.js</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #0b0b1e; overflow: hidden; }
    canvas { display: block; }

    #hud {
      position: fixed; top: 0; left: 0; right: 0;
      display: flex; justify-content: center; align-items: center;
      padding: 12px; gap: 24px;
      background: rgba(11, 11, 30, 0.85);
      backdrop-filter: blur(8px);
      border-bottom: 1px solid rgba(255,255,255,0.06);
      z-index: 10;
    }
    #hud h1 {
      font: bold 18px system-ui, sans-serif; color: #ccc;
      letter-spacing: 1px;
    }
    #hud button {
      padding: 6px 16px; border: 1px solid rgba(255,255,255,0.15);
      border-radius: 6px; background: rgba(255,255,255,0.06);
      color: #aaa; font: 13px system-ui; cursor: pointer;
      transition: background 0.2s;
    }
    #hud button:hover { background: rgba(255,255,255,0.12); color: #fff; }

    #tooltip {
      position: fixed; display: none;
      padding: 8px 14px; border-radius: 8px;
      background: rgba(20, 20, 50, 0.95);
      border: 1px solid rgba(255,255,255,0.1);
      color: #ccc; font: 13px system-ui;
      pointer-events: none; z-index: 20;
      max-width: 240px;
    }
  </style>
</head>
<body>
  <div id="hud">
    <h1>KANBAN BOARD</h1>
    <button onclick="addRandomCard()">+ Novo Card</button>
    <button onclick="resetCamera()">Reset Câmera</button>
  </div>
  <div id="tooltip"></div>

  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // =========================================================
    // Dados das colunas e cards
    // =========================================================
    const COLUMNS = [
      { id: 'todo',       titulo: 'A FAZER',       cor: '#6366f1', cards: [] },
      { id: 'inprogress', titulo: 'EM PROGRESSO',   cor: '#f59e0b', cards: [] },
      { id: 'done',       titulo: 'CONCLUÍDO',      cor: '#10b981', cards: [] },
    ];

    const PRIORIDADES = {
      alta:   { label: 'ALTA',   cor: '#ef4444' },
      media:  { label: 'MÉDIA',  cor: '#f59e0b' },
      baixa:  { label: 'BAIXA',  cor: '#22c55e' },
    };

    const TAGS_POOL = ['Frontend', 'Backend', 'Design', 'DevOps', 'Docs', 'Bug', 'Feature'];
    const NOMES = ['Ana', 'Bruno', 'Carla', 'Diego', 'Elena', 'Felipe', 'Gabi', 'Hugo'];
    const CORES_AVATAR = ['#e74c3c','#3498db','#2ecc71','#9b59b6','#e67e22','#1abc9c','#f39c12','#e84393'];

    const initialCards = [
      { titulo: 'Criar componente Header', responsavel: 'Ana', prioridade: 'alta',
        tags: ['Frontend', 'Feature'], coluna: 'todo' },
      { titulo: 'Configurar pipeline CI/CD', responsavel: 'Bruno', prioridade: 'alta',
        tags: ['DevOps'], coluna: 'todo' },
      { titulo: 'Escrever testes unitários', responsavel: 'Carla', prioridade: 'media',
        tags: ['Backend', 'Docs'], coluna: 'todo' },
      { titulo: 'Implementar autenticação JWT', responsavel: 'Diego', prioridade: 'alta',
        tags: ['Backend', 'Feature'], coluna: 'inprogress' },
      { titulo: 'Redesign da tela de login', responsavel: 'Elena', prioridade: 'media',
        tags: ['Design', 'Frontend'], coluna: 'inprogress' },
      { titulo: 'Corrigir bug no formulário', responsavel: 'Felipe', prioridade: 'baixa',
        tags: ['Frontend', 'Bug'], coluna: 'done' },
      { titulo: 'Deploy do ambiente staging', responsavel: 'Gabi', prioridade: 'baixa',
        tags: ['DevOps'], coluna: 'done' },
    ];

    // =========================================================
    // Setup da cena
    // =========================================================
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0b0b1e);

    const camera = new THREE.PerspectiveCamera(45, innerWidth / innerHeight, 0.1, 100);
    camera.position.set(0, 3, 14);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(innerWidth, innerHeight);
    renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.07;
    controls.maxPolarAngle = Math.PI / 2.1;
    controls.minDistance = 5;
    controls.maxDistance = 25;
    controls.target.set(0, 0, 0);

    // =========================================================
    // Iluminação
    // =========================================================
    scene.add(new THREE.AmbientLight(0x444466, 1.5));

    const mainLight = new THREE.DirectionalLight(0xffffff, 1.8);
    mainLight.position.set(6, 10, 8);
    mainLight.castShadow = true;
    mainLight.shadow.mapSize.set(2048, 2048);
    mainLight.shadow.camera.left = -12;
    mainLight.shadow.camera.right = 12;
    mainLight.shadow.camera.top = 8;
    mainLight.shadow.camera.bottom = -8;
    scene.add(mainLight);

    scene.add(new THREE.PointLight(0x6366f1, 0.4, 20).position.set(-6, 4, 4) && scene.children[scene.children.length-1] || new THREE.Object3D());
    const p1 = new THREE.PointLight(0x6366f1, 0.4, 20);
    p1.position.set(-6, 4, 4);
    scene.add(p1);
    const p2 = new THREE.PointLight(0x10b981, 0.3, 20);
    p2.position.set(6, 4, 4);
    scene.add(p2);

    // Chão
    const floorGeo = new THREE.PlaneGeometry(30, 20);
    const floorMat = new THREE.MeshStandardMaterial({
      color: 0x08081a, roughness: 0.95, metalness: 0.05
    });
    const floor = new THREE.Mesh(floorGeo, floorMat);
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = -3.5;
    floor.receiveShadow = true;
    scene.add(floor);

    // Grid sutil no chão
    const gridHelper = new THREE.GridHelper(28, 28, 0x1a1a3a, 0x12122a);
    gridHelper.position.y = -3.49;
    scene.add(gridHelper);

    // =========================================================
    // Utilitários de texto
    // =========================================================
    function textTexture(text, opts = {}) {
      const {
        w = 512, h = 48, size = 26, bold = false,
        color = '#ffffff', align = 'left', px = 10,
        bg = null, maxLen = 38,
      } = opts;
      const c = document.createElement('canvas');
      c.width = w; c.height = h;
      const ctx = c.getContext('2d');
      if (bg) { ctx.fillStyle = bg; ctx.fillRect(0, 0, w, h); }
      ctx.fillStyle = color;
      ctx.font = `${bold ? 'bold ' : ''}${size}px system-ui, sans-serif`;
      ctx.textBaseline = 'middle';
      ctx.textAlign = align;
      let s = text;
      if (s.length > maxLen) s = s.substring(0, maxLen - 1) + '…';
      ctx.fillText(s, align === 'center' ? w / 2 : px, h / 2);
      const t = new THREE.CanvasTexture(c);
      t.colorSpace = THREE.SRGBColorSpace;
      return t;
    }

    function pillTexture(items, colors, opts = {}) {
      const { w = 512, h = 40, size = 18 } = opts;
      const c = document.createElement('canvas');
      c.width = w; c.height = h;
      const ctx = c.getContext('2d');
      let x = 8;
      items.forEach((item, i) => {
        const col = colors[i % colors.length];
        ctx.font = `bold ${size}px system-ui`;
        const tw = ctx.measureText(item).width + 16;
        ctx.fillStyle = col + '33';
        ctx.beginPath();
        ctx.roundRect(x, 4, tw, h - 8, 12);
        ctx.fill();
        ctx.fillStyle = col;
        ctx.fillText(item, x + 8, h / 2 + 1);
        x += tw + 6;
      });
      const t = new THREE.CanvasTexture(c);
      t.colorSpace = THREE.SRGBColorSpace;
      return t;
    }

    function avatarTexture(name, bgColor) {
      const c = document.createElement('canvas');
      c.width = 128; c.height = 128;
      const ctx = c.getContext('2d');
      ctx.fillStyle = bgColor;
      ctx.beginPath();
      ctx.arc(64, 64, 60, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 52px system-ui';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(name.charAt(0).toUpperCase(), 64, 66);
      const t = new THREE.CanvasTexture(c);
      t.colorSpace = THREE.SRGBColorSpace;
      return t;
    }

    // =========================================================
    // Dimensões
    // =========================================================
    const COL_W = 4.2;
    const COL_GAP = 4.8;
    const CARD_W = 3.6;
    const CARD_H = 2.0;
    const CARD_GAP = 2.3;
    const CARD_START_Y = 1.5;

    // =========================================================
    // Criar colunas
    // =========================================================
    const columnGroups = [];

    COLUMNS.forEach((col, ci) => {
      const group = new THREE.Group();
      const x = (ci - 1) * COL_GAP;
      group.position.set(x, 0, 0);
      group.userData = { column: col };

      // Painel de fundo da coluna (sutil, transparente)
      const panelGeo = new THREE.PlaneGeometry(COL_W, 10);
      const panelMat = new THREE.MeshStandardMaterial({
        color: 0x14142e,
        transparent: true, opacity: 0.4,
        roughness: 0.9,
      });
      const panel = new THREE.Mesh(panelGeo, panelMat);
      panel.position.set(0, -1, -0.15);
      panel.receiveShadow = true;
      group.add(panel);

      // Borda superior colorida
      const accentGeo = new THREE.PlaneGeometry(COL_W, 0.08);
      const accentMat = new THREE.MeshBasicMaterial({ color: col.cor });
      const accent = new THREE.Mesh(accentGeo, accentMat);
      accent.position.set(0, 3.95, -0.14);
      group.add(accent);

      // Título da coluna
      const titleTex = textTexture(col.titulo, {
        w: 512, h: 56, size: 30, bold: true,
        color: col.cor, align: 'center',
      });
      const titleMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(COL_W - 0.4, 0.5),
        new THREE.MeshBasicMaterial({ map: titleTex, transparent: true })
      );
      titleMesh.position.set(0, 3.5, -0.1);
      group.add(titleMesh);

      // Contador
      const countTex = textTexture('0 cards', {
        w: 256, h: 40, size: 20, color: '#555577', align: 'center',
      });
      const countMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(1.5, 0.3),
        new THREE.MeshBasicMaterial({ map: countTex, transparent: true })
      );
      countMesh.position.set(0, 3.1, -0.1);
      group.add(countMesh);
      group.userData.countMesh = countMesh;

      scene.add(group);
      columnGroups.push(group);
    });

    function updateColumnCount(colIndex) {
      const col = COLUMNS[colIndex];
      const g = columnGroups[colIndex];
      const tex = textTexture(`${col.cards.length} card${col.cards.length !== 1 ? 's' : ''}`, {
        w: 256, h: 40, size: 20, color: '#555577', align: 'center',
      });
      g.userData.countMesh.material.map.dispose();
      g.userData.countMesh.material.map = tex;
      g.userData.countMesh.material.needsUpdate = true;
    }

    // =========================================================
    // Criar card 3D
    // =========================================================
    const allCardMeshes = [];

    function createCard(data, colIndex, slotIndex) {
      const group = new THREE.Group();
      group.userData = { data, colIndex, slotIndex };

      // Fundo do card
      const bgGeo = new THREE.BoxGeometry(CARD_W, CARD_H, 0.08);
      const bgMat = new THREE.MeshStandardMaterial({
        color: 0x1c1c3a,
        roughness: 0.7, metalness: 0.05,
      });
      const bg = new THREE.Mesh(bgGeo, bgMat);
      bg.castShadow = true;
      bg.receiveShadow = true;
      group.add(bg);

      // Faixa lateral de prioridade
      const prio = PRIORIDADES[data.prioridade];
      const prioGeo = new THREE.PlaneGeometry(0.08, CARD_H - 0.15);
      const prioMat = new THREE.MeshBasicMaterial({ color: prio.cor });
      const prioMesh = new THREE.Mesh(prioGeo, prioMat);
      prioMesh.position.set(-CARD_W / 2 + 0.12, 0, 0.05);
      group.add(prioMesh);

      // Badge de prioridade
      const prioBadgeTex = textTexture(prio.label, {
        w: 128, h: 32, size: 18, bold: true,
        color: prio.cor, align: 'center',
      });
      const prioBadge = new THREE.Mesh(
        new THREE.PlaneGeometry(0.7, 0.22),
        new THREE.MeshBasicMaterial({ map: prioBadgeTex, transparent: true })
      );
      prioBadge.position.set(CARD_W / 2 - 0.5, CARD_H / 2 - 0.2, 0.05);
      group.add(prioBadge);

      // Título do card
      const titleTex = textTexture(data.titulo, {
        w: 512, h: 44, size: 24, bold: true, color: '#e0e0f0', maxLen: 30,
      });
      const titleMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.6, 0.32),
        new THREE.MeshBasicMaterial({ map: titleTex, transparent: true })
      );
      titleMesh.position.set(0.1, CARD_H / 2 - 0.5, 0.05);
      group.add(titleMesh);

      // Tags
      const tagColors = ['#6366f1', '#ec4899', '#14b8a6', '#f59e0b', '#8b5cf6', '#ef4444', '#22c55e'];
      const tagTex = pillTexture(
        data.tags,
        data.tags.map((_, i) => tagColors[i % tagColors.length])
      );
      const tagMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(CARD_W - 0.4, 0.28),
        new THREE.MeshBasicMaterial({ map: tagTex, transparent: true })
      );
      tagMesh.position.set(0, -0.05, 0.05);
      group.add(tagMesh);

      // Avatar + nome do responsável
      const avatarIdx = NOMES.indexOf(data.responsavel);
      const avatarColor = CORES_AVATAR[avatarIdx >= 0 ? avatarIdx : 0];
      const avTex = avatarTexture(data.responsavel, avatarColor);
      const avMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(0.4, 0.4),
        new THREE.MeshBasicMaterial({ map: avTex, transparent: true })
      );
      avMesh.position.set(-CARD_W / 2 + 0.45, -CARD_H / 2 + 0.3, 0.05);
      group.add(avMesh);

      const nameTex = textTexture(data.responsavel, {
        w: 256, h: 32, size: 19, color: '#8888aa',
      });
      const nameMesh = new THREE.Mesh(
        new THREE.PlaneGeometry(1.4, 0.22),
        new THREE.MeshBasicMaterial({ map: nameTex, transparent: true })
      );
      nameMesh.position.set(-CARD_W / 2 + 1.5, -CARD_H / 2 + 0.3, 0.05);
      group.add(nameMesh);

      // Posicionar na coluna
      const colGroup = columnGroups[colIndex];
      const x = 0;
      const y = CARD_START_Y - slotIndex * CARD_GAP;
      group.position.set(x, y, 0);
      colGroup.add(group);

      COLUMNS[colIndex].cards.push(group);
      allCardMeshes.push(group);

      return group;
    }

    // Cards iniciais
    COLUMNS.forEach((col, ci) => {
      const cardsForCol = initialCards.filter(c => c.coluna === col.id);
      cardsForCol.forEach((data, si) => createCard(data, ci, si));
      updateColumnCount(ci);
    });

    // =========================================================
    // Drag & Drop via raycasting
    // =========================================================
    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    const dragPlane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);
    const intersection = new THREE.Vector3();
    const offset = new THREE.Vector3();

    let hoveredCard = null;
    let draggedCard = null;
    let dragStartCol = -1;

    function getCardFromIntersect(obj) {
      while (obj) {
        if (obj.userData && obj.userData.data && allCardMeshes.includes(obj)) return obj;
        obj = obj.parent;
      }
      return null;
    }

    function getColIndexFromWorldX(wx) {
      let best = 0, bestDist = Infinity;
      columnGroups.forEach((g, i) => {
        const d = Math.abs(g.position.x - wx);
        if (d < bestDist) { bestDist = d; best = i; }
      });
      return best;
    }

    function repositionCards(colIndex, skipCard = null) {
      const col = COLUMNS[colIndex];
      let slot = 0;
      col.cards.forEach(card => {
        if (card === skipCard) return;
        const targetY = CARD_START_Y - slot * CARD_GAP;
        card.userData.targetY = targetY;
        card.userData.targetX = 0;
        slot++;
      });
    }

    // Pointer events
    function onPointerMove(e) {
      pointer.x = (e.clientX / innerWidth) * 2 - 1;
      pointer.y = -(e.clientY / innerHeight) * 2 + 1;

      if (draggedCard) {
        raycaster.setFromCamera(pointer, camera);
        raycaster.ray.intersectPlane(dragPlane, intersection);

        const colGroup = columnGroups[dragStartCol];
        const worldPos = new THREE.Vector3();
        colGroup.getWorldPosition(worldPos);

        draggedCard.position.x = intersection.x - worldPos.x - offset.x;
        draggedCard.position.y = intersection.y - worldPos.y - offset.y;
        draggedCard.position.z = 0.8;

        // Destacar coluna alvo
        const targetCol = getColIndexFromWorldX(intersection.x);
        columnGroups.forEach((g, i) => {
          const panel = g.children[0];
          if (i === targetCol) {
            panel.material.opacity = 0.65;
            panel.material.color.set(COLUMNS[i].cor);
          } else {
            panel.material.opacity = 0.4;
            panel.material.color.set(0x14142e);
          }
        });
      }
    }

    function onPointerDown(e) {
      if (e.button !== 0) return;
      raycaster.setFromCamera(pointer, camera);

      const meshes = [];
      allCardMeshes.forEach(c => c.traverse(ch => { if (ch.isMesh) meshes.push(ch); }));
      const hits = raycaster.intersectObjects(meshes);
      if (hits.length === 0) return;

      const card = getCardFromIntersect(hits[0].object);
      if (!card) return;

      draggedCard = card;
      dragStartCol = card.userData.colIndex;
      controls.enabled = false;

      // Plano de arrasto alinhado com a câmera
      const cardWorld = new THREE.Vector3();
      card.getWorldPosition(cardWorld);
      dragPlane.setFromNormalAndCoplanarPoint(
        camera.getWorldDirection(new THREE.Vector3()).negate(),
        cardWorld
      );

      raycaster.ray.intersectPlane(dragPlane, intersection);
      const colWorld = new THREE.Vector3();
      columnGroups[dragStartCol].getWorldPosition(colWorld);
      offset.set(
        intersection.x - colWorld.x - card.position.x,
        intersection.y - colWorld.y - card.position.y,
        0
      );

      card.position.z = 0.8;
    }

    function onPointerUp() {
      if (!draggedCard) return;

      const cardWorld = new THREE.Vector3();
      draggedCard.getWorldPosition(cardWorld);
      const newColIndex = getColIndexFromWorldX(cardWorld.x);
      const oldColIndex = draggedCard.userData.colIndex;

      // Remover do array da coluna antiga
      const oldCards = COLUMNS[oldColIndex].cards;
      const idx = oldCards.indexOf(draggedCard);
      if (idx > -1) oldCards.splice(idx, 1);

      // Reparentar se mudou de coluna
      if (newColIndex !== oldColIndex) {
        columnGroups[oldColIndex].remove(draggedCard);

        const oldColWorld = new THREE.Vector3();
        const newColWorld = new THREE.Vector3();
        columnGroups[oldColIndex].getWorldPosition(oldColWorld);
        columnGroups[newColIndex].getWorldPosition(newColWorld);
        draggedCard.position.x += oldColWorld.x - newColWorld.x;

        columnGroups[newColIndex].add(draggedCard);
        draggedCard.userData.colIndex = newColIndex;
      }

      // Determinar posição de inserção por Y
      const cards = COLUMNS[newColIndex].cards;
      let insertIdx = cards.length;
      for (let i = 0; i < cards.length; i++) {
        if (draggedCard.position.y > cards[i].position.y) {
          insertIdx = i;
          break;
        }
      }
      cards.splice(insertIdx, 0, draggedCard);

      // Reposicionar todos os cards nas colunas afetadas
      repositionCards(newColIndex);
      if (newColIndex !== oldColIndex) repositionCards(oldColIndex);
      updateColumnCount(newColIndex);
      if (newColIndex !== oldColIndex) updateColumnCount(oldColIndex);

      // Resetar visual das colunas
      columnGroups.forEach((g, i) => {
        g.children[0].material.opacity = 0.4;
        g.children[0].material.color.set(0x14142e);
      });

      draggedCard.position.z = 0;
      draggedCard = null;
      controls.enabled = true;
    }

    window.addEventListener('pointermove', onPointerMove);
    window.addEventListener('pointerdown', onPointerDown);
    window.addEventListener('pointerup', onPointerUp);

    // =========================================================
    // Tooltip
    // =========================================================
    const tooltip = document.getElementById('tooltip');

    function updateTooltip(e) {
      if (draggedCard) { tooltip.style.display = 'none'; return; }

      raycaster.setFromCamera(pointer, camera);
      const meshes = [];
      allCardMeshes.forEach(c => c.traverse(ch => { if (ch.isMesh) meshes.push(ch); }));
      const hits = raycaster.intersectObjects(meshes);

      const card = hits.length > 0 ? getCardFromIntersect(hits[0].object) : null;

      if (card && card !== hoveredCard) {
        hoveredCard = card;
        const d = card.userData.data;
        const prio = PRIORIDADES[d.prioridade];
        tooltip.innerHTML = `
          <strong>${d.titulo}</strong><br>
          <span style="color:${prio.cor}">● ${prio.label}</span>
          &nbsp;·&nbsp; ${d.responsavel}<br>
          <span style="color:#666">${d.tags.join(', ')}</span>
        `;
        tooltip.style.display = 'block';
      } else if (!card) {
        hoveredCard = null;
        tooltip.style.display = 'none';
      }

      if (tooltip.style.display === 'block') {
        tooltip.style.left = (e.clientX + 16) + 'px';
        tooltip.style.top = (e.clientY + 16) + 'px';
      }

      document.body.style.cursor = card ? 'grab' : 'default';
      if (draggedCard) document.body.style.cursor = 'grabbing';
    }
    window.addEventListener('pointermove', updateTooltip);

    // =========================================================
    // Adicionar card aleatório
    // =========================================================
    const TAREFAS = [
      'Refatorar módulo de pagamentos', 'Adicionar testes E2E',
      'Otimizar queries do dashboard', 'Criar documentação da API',
      'Implementar dark mode', 'Migrar para TypeScript',
      'Configurar monitoramento', 'Revisar PR #142',
      'Atualizar dependências', 'Criar seed do banco de dados',
    ];

    window.addRandomCard = function () {
      const colIndex = 0; // sempre adiciona em "A Fazer"
      const slot = COLUMNS[colIndex].cards.length;
      const data = {
        titulo: TAREFAS[Math.floor(Math.random() * TAREFAS.length)],
        responsavel: NOMES[Math.floor(Math.random() * NOMES.length)],
        prioridade: ['alta', 'media', 'baixa'][Math.floor(Math.random() * 3)],
        tags: [TAGS_POOL[Math.floor(Math.random() * TAGS_POOL.length)]],
        coluna: 'todo',
      };
      const card = createCard(data, colIndex, slot);
      // Animação de entrada (surge de baixo)
      card.position.y = -6;
      card.userData.targetY = CARD_START_Y - slot * CARD_GAP;
      updateColumnCount(colIndex);
    };

    window.resetCamera = function () {
      controls.target.set(0, 0, 0);
      camera.position.set(0, 3, 14);
    };

    // =========================================================
    // Loop de animação
    // =========================================================
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);
      const dt = clock.getDelta();
      const time = clock.elapsedTime;
      controls.update();

      // Animação suave de posição dos cards
      allCardMeshes.forEach((card, i) => {
        if (card === draggedCard) return;

        if (card.userData.targetY !== undefined) {
          card.position.y = THREE.MathUtils.lerp(
            card.position.y, card.userData.targetY, dt * 6
          );
        }
        if (card.userData.targetX !== undefined) {
          card.position.x = THREE.MathUtils.lerp(
            card.position.x, card.userData.targetX, dt * 6
          );
        }
        // Z volta a 0
        card.position.z = THREE.MathUtils.lerp(card.position.z, 0, dt * 8);

        // Hover: leve elevação
        const isHovered = (card === hoveredCard && !draggedCard);
        const targetScale = isHovered ? 1.04 : 1.0;
        card.scale.lerp(new THREE.Vector3(targetScale, targetScale, targetScale), dt * 10);
      });

      renderer.render(scene, camera);
    }

    animate();

    // =========================================================
    // Responsividade
    // =========================================================
    window.addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth, innerHeight);
    });
  </script>
</body>
</html>
```

**O que este exemplo demonstra:**

| Conceito | Uso no Kanban 3D |
|----------|-----------------|
| **Group hierarchy** | Cada coluna é um Group contendo painel, título e cards; cada card é um Group com fundo, textos e avatar |
| **BoxGeometry + MeshStandardMaterial** | Corpo dos cards com profundidade (0.08), sombras e aparência sólida |
| **CanvasTexture** | Título, tags (pills coloridas), badge de prioridade e avatar — tudo renderizado via Canvas 2D |
| **Raycaster + drag & drop** | Arrastar cards entre colunas com detecção de coluna alvo e inserção por posição Y |
| **Plane de arrasto** | `THREE.Plane` alinhado à câmera para converter coordenadas de mouse em movimento 3D |
| **Reparenting** | Ao soltar em outra coluna, o card é removido do Group antigo e adicionado ao novo |
| **Lerp de posição** | Cards animam suavemente para a posição correta após reordenação |
| **Hover com escala** | Card amplia levemente ao passar o mouse, com tooltip HTML |
| **OrbitControls** | Navegação 3D desabilitada durante drag para não conflitar |
| **Spawn com animação** | Botão "Novo Card" cria card que sobe suavemente da parte inferior |

> **Extensões sugeridas:** adicionar som ao arrastar/soltar (PositionalAudio), swimlanes por responsável no eixo Z (profundidade), filtros por tag/prioridade com animação de ocultação, persistência com localStorage ou backend via fetch, e websocket para atualização colaborativa em tempo real.

---

## Exemplo Completo — Visualização 3D com Fetch API Dinâmico

Este exemplo demonstra a integração entre **Fetch API** e **Three.js**: busca dados de uma API REST pública, constrói um gráfico de barras 3D com os resultados e permite recarregar com novos dados a qualquer momento. O ciclo completo — loading → fetch → parse → renderização 3D → atualização — é tratado de forma assíncrona.

O exemplo usa a API pública [restcountries.com](https://restcountries.com) para buscar dados de países da América do Sul e exibir a população de cada um como barras 3D proporcionais.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Fetch API + Three.js — População da América do Sul</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #0a0a1a; overflow: hidden; font-family: system-ui, sans-serif; }
    canvas { display: block; }

    #hud {
      position: fixed; top: 0; left: 0; right: 0;
      display: flex; justify-content: center; align-items: center;
      gap: 20px; padding: 12px;
      background: rgba(10, 10, 26, 0.9);
      backdrop-filter: blur(8px);
      border-bottom: 1px solid rgba(255,255,255,0.06);
      z-index: 10;
    }
    #hud h1 { font-size: 16px; color: #ccc; letter-spacing: 0.5px; }
    #hud button {
      padding: 6px 18px; border: 1px solid rgba(255,255,255,0.12);
      border-radius: 6px; background: rgba(99,102,241,0.15);
      color: #a5b4fc; font: 13px system-ui; cursor: pointer;
      transition: all 0.2s;
    }
    #hud button:hover { background: rgba(99,102,241,0.3); color: #fff; }
    #hud button:disabled { opacity: 0.4; cursor: not-allowed; }
    #status {
      font-size: 13px; color: #666;
      min-width: 180px; text-align: center;
    }

    #loading-overlay {
      position: fixed; inset: 0;
      display: flex; flex-direction: column;
      justify-content: center; align-items: center;
      background: #0a0a1a; z-index: 100;
      transition: opacity 0.5s;
    }
    #loading-overlay.hidden { opacity: 0; pointer-events: none; }
    .spinner {
      width: 40px; height: 40px;
      border: 3px solid rgba(99,102,241,0.2);
      border-top-color: #6366f1;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }
    #loading-overlay p { color: #888; margin-top: 16px; font-size: 14px; }

    #error-msg {
      position: fixed; bottom: 20px; left: 50%;
      transform: translateX(-50%);
      padding: 10px 24px; border-radius: 8px;
      background: rgba(239,68,68,0.15);
      border: 1px solid rgba(239,68,68,0.3);
      color: #fca5a5; font-size: 13px;
      display: none; z-index: 20;
    }
  </style>
</head>
<body>
  <div id="loading-overlay">
    <div class="spinner"></div>
    <p>Carregando dados da API...</p>
  </div>

  <div id="hud">
    <h1>POPULAÇÃO — AMÉRICA DO SUL</h1>
    <button id="btn-reload" onclick="reloadData()">↻ Recarregar API</button>
    <button id="btn-sort" onclick="toggleSort()">Ordenar ▼</button>
    <span id="status">Aguardando...</span>
  </div>

  <div id="error-msg"></div>

  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // =========================================================
    // Setup da cena
    // =========================================================
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a0a1a);

    const camera = new THREE.PerspectiveCamera(45, innerWidth / innerHeight, 0.1, 100);
    camera.position.set(8, 8, 16);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(innerWidth, innerHeight);
    renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.08;
    controls.target.set(0, 2, 0);

    // Iluminação
    scene.add(new THREE.AmbientLight(0x334466, 1.2));
    const dirLight = new THREE.DirectionalLight(0xffffff, 1.5);
    dirLight.position.set(8, 12, 8);
    dirLight.castShadow = true;
    dirLight.shadow.mapSize.set(1024, 1024);
    scene.add(dirLight);
    const rimLight = new THREE.PointLight(0x6366f1, 0.5, 30);
    rimLight.position.set(-8, 6, -4);
    scene.add(rimLight);

    // Chão
    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(30, 20),
      new THREE.MeshStandardMaterial({ color: 0x08081a, roughness: 0.95 })
    );
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = -0.01;
    floor.receiveShadow = true;
    scene.add(floor);

    const grid = new THREE.GridHelper(28, 28, 0x1a1a3a, 0x101028);
    scene.add(grid);

    // =========================================================
    // Utilitários
    // =========================================================
    function labelTexture(text, opts = {}) {
      const { w = 256, h = 64, size = 22, color = '#ccc', bold = false,
              align = 'center', bg = null } = opts;
      const c = document.createElement('canvas');
      c.width = w; c.height = h;
      const ctx = c.getContext('2d');
      if (bg) { ctx.fillStyle = bg; ctx.fillRect(0, 0, w, h); }
      ctx.fillStyle = color;
      ctx.font = `${bold ? 'bold ' : ''}${size}px system-ui, sans-serif`;
      ctx.textAlign = align;
      ctx.textBaseline = 'middle';
      let display = text.length > 20 ? text.substring(0, 19) + '…' : text;
      ctx.fillText(display, align === 'center' ? w / 2 : 8, h / 2);
      const t = new THREE.CanvasTexture(c);
      t.colorSpace = THREE.SRGBColorSpace;
      return t;
    }

    function formatPop(n) {
      if (n >= 1_000_000) return (n / 1_000_000).toFixed(1) + 'M';
      if (n >= 1_000) return (n / 1_000).toFixed(0) + 'K';
      return String(n);
    }

    // Gradiente de cor baseado em valor normalizado (0 a 1)
    function valueColor(t) {
      // azul → ciano → verde → amarelo → laranja
      const colors = [
        new THREE.Color(0x3b82f6),
        new THREE.Color(0x06b6d4),
        new THREE.Color(0x10b981),
        new THREE.Color(0xf59e0b),
        new THREE.Color(0xef4444),
      ];
      const idx = t * (colors.length - 1);
      const lo = Math.floor(idx);
      const hi = Math.min(lo + 1, colors.length - 1);
      return colors[lo].clone().lerp(colors[hi], idx - lo);
    }

    // =========================================================
    // Estado
    // =========================================================
    let barsGroup = new THREE.Group();
    scene.add(barsGroup);
    let currentData = [];
    let sortAsc = false;
    let barTargets = []; // { mesh, targetScaleY, targetColor }

    // =========================================================
    // Fetch e construção das barras
    // =========================================================
    async function fetchCountries() {
      const statusEl = document.getElementById('status');
      const btnReload = document.getElementById('btn-reload');
      const loadingEl = document.getElementById('loading-overlay');
      const errorEl = document.getElementById('error-msg');

      statusEl.textContent = 'Buscando dados...';
      btnReload.disabled = true;
      errorEl.style.display = 'none';

      try {
        const url = 'https://restcountries.com/v3.1/subregion/South%20America?fields=name,population,flags,cca2';
        const response = await fetch(url);

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        const data = await response.json();

        currentData = data
          .map(c => ({
            nome: c.name.common,
            nomeLocal: c.name.nativeName
              ? Object.values(c.name.nativeName)[0]?.common || c.name.common
              : c.name.common,
            populacao: c.population,
            sigla: c.cca2,
            bandeira: c.flags?.png || '',
          }))
          .filter(c => c.populacao > 0)
          .sort((a, b) => b.populacao - a.populacao);

        statusEl.textContent = `${currentData.length} países · ${new Date().toLocaleTimeString('pt-BR')}`;

        buildBars(currentData);

        loadingEl.classList.add('hidden');
      } catch (err) {
        console.error('Erro ao buscar dados:', err);
        statusEl.textContent = 'Erro ao carregar';
        errorEl.textContent = `Falha ao buscar API: ${err.message}`;
        errorEl.style.display = 'block';

        // Fallback com dados estáticos
        currentData = [
          { nome: 'Brazil', nomeLocal: 'Brasil', populacao: 214000000, sigla: 'BR' },
          { nome: 'Colombia', nomeLocal: 'Colombia', populacao: 51870000, sigla: 'CO' },
          { nome: 'Argentina', nomeLocal: 'Argentina', populacao: 45810000, sigla: 'AR' },
          { nome: 'Peru', nomeLocal: 'Perú', populacao: 33720000, sigla: 'PE' },
          { nome: 'Venezuela', nomeLocal: 'Venezuela', populacao: 28440000, sigla: 'VE' },
          { nome: 'Chile', nomeLocal: 'Chile', populacao: 19490000, sigla: 'CL' },
          { nome: 'Ecuador', nomeLocal: 'Ecuador', populacao: 17640000, sigla: 'EC' },
          { nome: 'Bolivia', nomeLocal: 'Bolivia', populacao: 11830000, sigla: 'BO' },
          { nome: 'Paraguay', nomeLocal: 'Paraguay', populacao: 7220000, sigla: 'PY' },
          { nome: 'Uruguay', nomeLocal: 'Uruguay', populacao: 3470000, sigla: 'UY' },
          { nome: 'Guyana', nomeLocal: 'Guyana', populacao: 790000, sigla: 'GY' },
          { nome: 'Suriname', nomeLocal: 'Suriname', populacao: 590000, sigla: 'SR' },
        ].sort((a, b) => b.populacao - a.populacao);

        statusEl.textContent = `${currentData.length} países (fallback) · ${new Date().toLocaleTimeString('pt-BR')}`;
        buildBars(currentData);
        loadingEl.classList.add('hidden');
      } finally {
        btnReload.disabled = false;
      }
    }

    function buildBars(data) {
      // Limpar barras anteriores
      barTargets = [];
      barsGroup.traverse(child => {
        if (child.isMesh) {
          child.geometry.dispose();
          child.material.dispose();
          if (child.material.map) child.material.map.dispose();
        }
      });
      scene.remove(barsGroup);
      barsGroup = new THREE.Group();
      scene.add(barsGroup);

      const maxPop = Math.max(...data.map(d => d.populacao));
      const BAR_W = 0.8;
      const GAP = 1.3;
      const MAX_H = 10;
      const startX = -((data.length - 1) * GAP) / 2;

      data.forEach((country, i) => {
        const normalized = country.populacao / maxPop;
        const barH = normalized * MAX_H + 0.1;
        const color = valueColor(normalized);

        // Barra
        const geo = new THREE.BoxGeometry(BAR_W, 1, BAR_W);
        const mat = new THREE.MeshStandardMaterial({
          color: color,
          roughness: 0.4,
          metalness: 0.2,
        });
        const bar = new THREE.Mesh(geo, mat);
        bar.position.set(startX + i * GAP, 0, 0);
        bar.scale.y = 0.01; // começa "flat" para animar
        bar.castShadow = true;
        bar.receiveShadow = true;
        barsGroup.add(bar);

        barTargets.push({
          mesh: bar,
          targetScaleY: barH,
        });

        // Label do país (sigla) — no chão
        const siglaTexture = labelTexture(country.sigla, {
          w: 128, h: 48, size: 28, bold: true, color: color.getStyle(),
        });
        const siglaPlane = new THREE.Mesh(
          new THREE.PlaneGeometry(BAR_W, 0.35),
          new THREE.MeshBasicMaterial({ map: siglaTexture, transparent: true })
        );
        siglaPlane.position.set(startX + i * GAP, 0.02, BAR_W / 2 + 0.3);
        siglaPlane.rotation.x = -Math.PI / 2;
        barsGroup.add(siglaPlane);

        // Nome do país — inclinado atrás da barra
        const nomeTexture = labelTexture(country.nomeLocal, {
          w: 256, h: 48, size: 20, color: '#888',
        });
        const nomePlane = new THREE.Mesh(
          new THREE.PlaneGeometry(1.5, 0.35),
          new THREE.MeshBasicMaterial({ map: nomeTexture, transparent: true })
        );
        nomePlane.position.set(startX + i * GAP, 0.4, -BAR_W / 2 - 0.2);
        nomePlane.rotation.x = -0.6;
        barsGroup.add(nomePlane);

        // Valor da população — acima da barra (posicionado depois na animação)
        const popTexture = labelTexture(formatPop(country.populacao), {
          w: 192, h: 48, size: 24, bold: true,
          color: '#e0e0ff',
        });
        const popPlane = new THREE.Mesh(
          new THREE.PlaneGeometry(1.0, 0.3),
          new THREE.MeshBasicMaterial({ map: popTexture, transparent: true })
        );
        popPlane.position.set(startX + i * GAP, barH + 0.3, 0);
        popPlane.userData.finalY = barH + 0.3;
        barsGroup.add(popPlane);

        barTargets[barTargets.length - 1].popLabel = popPlane;
      });

      // Ajustar câmera para caber todos os dados
      const totalWidth = data.length * GAP;
      controls.target.set(0, MAX_H / 3, 0);
      camera.position.set(0, MAX_H / 2 + 2, totalWidth * 0.7);
    }

    // =========================================================
    // Ordenação
    // =========================================================
    window.toggleSort = function () {
      sortAsc = !sortAsc;
      document.getElementById('btn-sort').textContent = sortAsc ? 'Ordenar ▲' : 'Ordenar ▼';

      currentData.sort((a, b) => sortAsc
        ? a.populacao - b.populacao
        : b.populacao - a.populacao
      );

      buildBars(currentData);
    };

    // =========================================================
    // Recarregar dados
    // =========================================================
    window.reloadData = function () {
      fetchCountries();
    };

    // =========================================================
    // Raycasting — hover com destaque
    // =========================================================
    const raycaster = new THREE.Raycaster();
    const pointer = new THREE.Vector2();
    let hoveredBar = null;

    window.addEventListener('pointermove', (e) => {
      pointer.x = (e.clientX / innerWidth) * 2 - 1;
      pointer.y = -(e.clientY / innerHeight) * 2 + 1;
    });

    // =========================================================
    // Animação
    // =========================================================
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);
      const dt = clock.getDelta();
      controls.update();

      // Animar barras crescendo
      barTargets.forEach(({ mesh, targetScaleY, popLabel }) => {
        mesh.scale.y = THREE.MathUtils.lerp(mesh.scale.y, targetScaleY, dt * 3);
        mesh.position.y = mesh.scale.y / 2;

        if (popLabel) {
          popLabel.position.y = mesh.scale.y + 0.3;
        }
      });

      // Raycasting hover
      raycaster.setFromCamera(pointer, camera);
      const barMeshes = barTargets.map(b => b.mesh);
      const hits = raycaster.intersectObjects(barMeshes);

      if (hits.length > 0) {
        const hit = hits[0].object;
        if (hoveredBar && hoveredBar !== hit) {
          hoveredBar.material.emissiveIntensity = 0;
        }
        hoveredBar = hit;
        hoveredBar.material.emissive = hoveredBar.material.color;
        hoveredBar.material.emissiveIntensity = 0.3;
        document.body.style.cursor = 'pointer';
      } else {
        if (hoveredBar) {
          hoveredBar.material.emissiveIntensity = 0;
          hoveredBar = null;
        }
        document.body.style.cursor = 'default';
      }

      renderer.render(scene, camera);
    }

    animate();

    // =========================================================
    // Responsividade e inicialização
    // =========================================================
    window.addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth, innerHeight);
    });

    // Fetch inicial
    fetchCountries();
  </script>
</body>
</html>
```

**Fluxo de dados — Fetch API → Three.js:**

```
┌──────────────────────────────────────────────────────────────────┐
│                    CICLO FETCH → RENDERIZAÇÃO                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. fetch() ──► restcountries.com/v3.1/subregion/South%20America │
│                                                                  │
│  2. response.json() ──► Array de objetos                         │
│     [{ name, population, cca2, flags }, ...]                     │
│                                                                  │
│  3. Transformar ──► Normalizar população (0 a 1)                 │
│                  ──► Calcular altura da barra (0 a MAX_H)        │
│                  ──► Mapear cor via gradiente (azul → vermelho)  │
│                                                                  │
│  4. Construir ──► BoxGeometry (barra) por país                   │
│               ──► CanvasTexture (sigla, nome, valor)             │
│               ──► Posicionar em grid linear                      │
│                                                                  │
│  5. Animar ──► scale.y lerp de 0 até altura alvo                 │
│            ──► Labels acompanham o topo da barra                 │
│                                                                  │
│  6. Interagir ──► Hover com emissive highlight                   │
│               ──► Reordenar (toggle asc/desc, rebuild)           │
│               ──► Recarregar (novo fetch, rebuild com animação)  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**O que este exemplo demonstra:**

| Conceito | Uso no exemplo |
|----------|----------------|
| **Fetch API (async/await)** | `fetch()` para API REST pública com tratamento de erro e fallback |
| **Loading state** | Overlay com spinner CSS enquanto aguarda a resposta |
| **Tratamento de erro** | Try/catch com mensagem visual e dados estáticos como fallback |
| **Dados → geometria** | População normalizada determina `scale.y`, cor via gradiente |
| **Rebuild dinâmico** | `dispose()` de geometrias/materiais/texturas antes de reconstruir |
| **BoxGeometry + MeshStandardMaterial** | Barras 3D com sombras e cor proporcional ao valor |
| **CanvasTexture** | Labels de sigla, nome e valor renderizados via Canvas 2D |
| **Animação de entrada** | Barras crescem de `scale.y = 0.01` até o alvo via lerp |
| **Raycaster** | Hover com destaque emissivo na barra sob o cursor |
| **Recarregamento** | Botão dispara novo fetch e reconstrói toda a visualização |
| **Ordenação** | Toggle alterna entre ordem crescente e decrescente |

> **Para adaptar a outras APIs:** substitua a URL do `fetch()` e ajuste o mapeamento em `currentData`. Qualquer API que retorne um array de objetos com um campo numérico pode ser visualizada — exemplos: cotações de moedas (Open Exchange Rates), estatísticas de repositórios GitHub (`api.github.com/users/{user}/repos`), dados climáticos (Open-Meteo), ou endpoints do seu próprio backend Spring Boot.

---

## Outras Ideias de Interfaces do Dia a Dia com Three.js

Além do catálogo de vídeos e do Kanban, diversas interfaces que normalmente são construídas em 2D podem ganhar uma dimensão visual extra com Three.js. A seguir, algumas sugestões com descrição da abordagem técnica.

---

### Dashboard de Monitoramento de Servidores

Visualização de racks de servidores em 3D, onde cada servidor é um `BoxGeometry` com faces coloridas por utilização de CPU/memória (gradiente verde → amarelo → vermelho via `CanvasTexture` atualizada em tempo real). Use `InstancedMesh` para renderizar dezenas de servidores com baixo custo de draw calls. Tooltips HTML com `CSS2DRenderer` exibem métricas detalhadas ao hover. Indicadores de status (online/offline/alerta) podem ser esferas com `MeshBasicMaterial` emissivo pulsando via shader uniform `time`. Dados reais viriam de uma API REST ou WebSocket (Prometheus, Grafana JSON API) com polling periódico atualizando as texturas.

```
Conceitos-chave: InstancedMesh, CanvasTexture dinâmica, CSS2DRenderer,
                 emissive pulsante, WebSocket para dados em tempo real
```

---

### Planta Baixa 3D Interativa

Ambiente arquitetônico onde paredes são `ExtrudeGeometry` a partir de `Shape` (planta 2D convertida em contorno 3D), e móveis são modelos glTF carregados com `GLTFLoader`. O usuário navega com `OrbitControls` ou `PointerLockControls` (primeira pessoa). Raycasting permite clicar em um móvel para abrir painel de edição (trocar cor via `material.color.set()`, trocar textura via `TextureLoader`, reposicionar com drag no plano do chão). Medidas podem ser linhas (`Line` com `BufferGeometry`) atualizadas ao mover objetos. Use `RectAreaLight` para simular iluminação de janelas e `PointLight` para luminárias.

```
Conceitos-chave: ExtrudeGeometry + Shape, GLTFLoader, PointerLockControls,
                 RectAreaLight, raycasting para seleção e drag de móveis
```

---

### Configurador de Produto (E-commerce)

Modelo 3D de um produto (tênis, cadeira, smartphone) carregado via `GLTFLoader`, centralizado na cena com `OrbitControls` para rotação livre. Um painel HTML lateral (ou `CSS2DRenderer`) oferece opções de cor, material e variante. Ao selecionar uma cor, percorra o modelo com `model.traverse()` e altere `material.color` dos meshes relevantes. Para troca de textura (couro, tecido, madeira), use `TextureLoader` e atribua a `material.map`. Adicione uma anotação 3D (hotspot) usando `Sprite` ou `CSS2DObject` posicionado no modelo, exibindo detalhes ao clicar. Um `ContactShadow` (plano com shadow blur) sob o produto e iluminação com `EnvironmentMap` (HDR via `RGBELoader`) completam o realismo. Screenshot para compartilhamento via `renderer.domElement.toDataURL()`.

```
Conceitos-chave: GLTFLoader + traverse, troca de material/textura em runtime,
                 Environment Map (HDR), CSS2DObject para hotspots, toDataURL
```

---

### Galeria de Fotos / Portfólio 3D

Fotos dispostas em uma parede curva (`CylinderGeometry` invertida com segmentos, ou cards posicionados em arco via trigonometria). Cada foto é um `PlaneGeometry` com `TextureLoader` carregando a imagem. Ao clicar, a câmera faz `tween` (via gsap ou lerp manual) até ficar de frente para a foto selecionada, ampliando-a. Navegação por setas do teclado ou swipe (touch events mapeados para rotação da câmera ao redor do centro). Modo slideshow com transição automática. Adicionando `RGBShiftShader` ou `UnrealBloomPass` via `EffectComposer` no post-processing, as transições ganham efeito visual. Para portfólios, cada foto pode ter um painel descritivo (`CSS2DObject`) que aparece ao selecionar.

```
Conceitos-chave: TextureLoader + PlaneGeometry em arco, tween de câmera,
                 EffectComposer + UnrealBloomPass, CSS2DObject, touch/swipe
```

---

### Mapa de Assentos / Reserva de Lugares

Interface para reserva de assentos em cinemas, teatros, aviões ou eventos. Cada assento é um `BoxGeometry` ou modelo glTF simples posicionado em grid curvo (filas de cinema) ou layout custom (planta de avião). Status representado por cor: verde (disponível), cinza (ocupado), azul (selecionado pelo usuário). Raycasting no `click` alterna o estado do assento. Um painel lateral (`CSS2DRenderer` ou HTML overlay) exibe resumo da seleção, preço total e botão de confirmar. Para aviões, use câmera com transição suave entre visão geral e close-up da fileira. Dados de disponibilidade vêm de API REST com `fetch()`, e a reserva é enviada via `POST`.

```
Conceitos-chave: grid curvo com trigonometria, raycaster para seleção múltipla,
                 material.color dinâmico por estado, fetch GET/POST, CSS2DRenderer
```

---

### Visualizador de Organograma / Grafo de Equipe

Organograma corporativo renderizado como árvore 3D, onde cada nó é um card com foto (textura circular via `CanvasTexture`), nome e cargo. Conexões entre nós são `TubeGeometry` ou `Line2` seguindo curvas (`CatmullRomCurve3`). O layout pode ser calculado com algoritmo de árvore (d3-hierarchy) e mapeado para coordenadas 3D com profundidade no eixo Z por nível hierárquico. Ao clicar em um nó, a câmera faz tween até ele e expande informações detalhadas. Filtros por departamento ocultam/mostram sub-árvores com animação de escala. Em grafos maiores (redes de colaboração), use `ForceGraph3D` (biblioteca baseada em Three.js) ou implemente force simulation com d3-force mapeando posições para o espaço 3D.

```
Conceitos-chave: d3-hierarchy para layout, CatmullRomCurve3 + TubeGeometry
                 para conexões, tween de câmera, CanvasTexture circular
```

---

### Calendário / Agenda 3D

Calendário mensal onde cada dia é uma coluna vertical e os eventos são blocos empilhados (`BoxGeometry`) com altura proporcional à duração. Cores representam categorias (reunião, deadline, pessoal). O mês inteiro forma um grid 7×5 no espaço 3D, navegável com `OrbitControls`. Ao passar o mouse (raycaster), o bloco do evento eleva-se e exibe tooltip com detalhes. Clicar abre modal HTML para edição. Navegação entre meses com transição de câmera lateral (lerp de `camera.position.x`). Dados carregados via `fetch()` de uma API de calendário (Google Calendar API, backend próprio). Dias com mais eventos formam "torres" mais altas, dando uma visão volumétrica da carga de trabalho.

```
Conceitos-chave: grid 7×5, BoxGeometry empilhado, altura = duração,
                 cor = categoria, fetch para Google Calendar API, tooltip hover
```

---

### Player de Música / Visualizador de Áudio

Visualização de áudio em tempo real usando Web Audio API (`AnalyserNode`) integrada a Three.js. Barras de frequência como `InstancedMesh` de cilindros ou caixas cujo `scale.y` é atualizado a cada frame com dados de `getByteFrequencyData()`. Efeitos visuais com `ShaderMaterial` reagindo ao volume (pulsação, distorção de geometria). Uma esfera central com `MeshPhysicalMaterial` muda de cor e escala conforme a amplitude. Post-processing com `UnrealBloomPass` para brilho neon. Controles de play/pause/seek via HTML integrados, e informações da faixa (título, artista, capa) exibidas como texturas em planos 3D. Capa do álbum mapeada em um plano com reflexo no chão (`MeshReflectorMaterial` do drei ou plano com environment map).

```
Conceitos-chave: Web Audio API (AnalyserNode), getByteFrequencyData(),
                 InstancedMesh + scale dinâmico, ShaderMaterial, UnrealBloomPass
```

---

### E-commerce — Vitrine de Produtos 3D

Loja virtual onde os produtos são exibidos em prateleiras 3D (estante modelada com `BoxGeometry` ou glTF). Cada produto é um modelo 3D (`GLTFLoader`) posicionado na prateleira com etiqueta de preço (`CSS2DObject`). O usuário navega pelas prateleiras com `OrbitControls` limitado (apenas rotação horizontal e zoom). Ao clicar em um produto, a câmera faz tween até ele, o modelo amplia com animação, e um painel lateral exibe informações completas, botão "Adicionar ao carrinho" e galeria de texturas alternativas. Carrinho de compras gerenciado no estado JavaScript e exibido como overlay HTML. Filtros por categoria reorganizam os produtos nas prateleiras com animação de reposicionamento. Busca de produtos via `fetch()` a uma API REST com paginação.

```
Conceitos-chave: GLTFLoader em grid de prateleiras, CSS2DObject para preços,
                 tween de câmera, fetch com paginação, carrinho em estado JS
```

---

### Painel de Controle IoT / Smart Home

Representação 3D de uma casa ou cômodo com dispositivos IoT interativos. Cada dispositivo (lâmpada, termostato, câmera, sensor) é um modelo glTF ou primitiva com indicador visual de estado: lâmpadas com `PointLight` que liga/desliga ao clicar, termostato com `CanvasTexture` exibindo temperatura atualizada via WebSocket, sensor de presença com esfera pulsante. Dashboard lateral (`CSS2DRenderer`) com gráficos históricos (Chart.js ou canvas inline). Câmeras de segurança exibem feed de vídeo real como `VideoTexture` no plano 3D correspondente. Automações podem ser testadas visualmente — ligar todas as luzes, simular cenário "saindo de casa" com animação de desligamento sequencial. Dados via MQTT over WebSocket ou API REST.

```
Conceitos-chave: GLTFLoader para ambiente, PointLight dinâmico, VideoTexture
                 para câmeras, WebSocket/MQTT, CanvasTexture atualizada em tempo real
```

---

### Timeline / Linha do Tempo 3D

Eventos históricos, sprints de projeto ou marcos de produto dispostos ao longo de um tubo (`TubeGeometry` seguindo `CatmullRomCurve3`). Cada evento é um card 3D posicionado ao longo da curva com data, título e ícone. A câmera percorre o tubo como um trilho — o usuário controla a posição na timeline com scroll ou slider, e a câmera interpola sua posição e lookAt ao longo da curva. Bifurcações (branches em timeline de projeto) podem ser tubos secundários divergindo do principal. Escala de tempo representada por marcadores (`Sprite`) a intervalos regulares. Zoom no scroll amplia a seção visível. Dados de eventos carregados via `fetch()` de um endpoint `/api/timeline`.

```
Conceitos-chave: CatmullRomCurve3 + TubeGeometry, câmera seguindo path,
                 getPointAt(t) para posição, Sprite para marcadores, scroll → t
```

---

### Tela de Login / Onboarding com Background 3D

Background animado com partículas flutuantes (`Points` + `BufferGeometry`), rede de conexões (`LineSegments` entre partículas próximas) ou formas geométricas orbitando suavemente. O formulário de login é HTML puro posicionado sobre o canvas com `position: fixed` e `z-index`. O background reage ao mouse — partículas se afastam do cursor (repulsão via distância) ou se atraem. Efeitos de pós-processamento (`UnrealBloomPass`) adicionam brilho neon. Para onboarding, a câmera avança por um túnel de partículas a cada etapa do formulário, criando sensação de progressão. Tudo com custo mínimo de performance usando `PointsMaterial` com attenuation e `AdditiveBlending`.

```
Conceitos-chave: Points + BufferGeometry (partículas), LineSegments para
                 conexões, repulsão do mouse, UnrealBloomPass, HTML sobre canvas
```

---

### Explorador de Sistema de Arquivos

Visualização hierárquica de diretórios e arquivos como cidade 3D (treemap extrudado) — cada pasta é um bloco com altura proporcional ao tamanho total, e arquivos são blocos menores dentro. Cores representam tipos de arquivo (`.js` azul, `.css` verde, `.png` laranja, etc.). Inspirado em ferramentas como CodeCity e WindingTree. Navegação com `OrbitControls` + duplo-clique para entrar em uma pasta (zoom + rebuild do nível filho). Breadcrumb HTML mostra o caminho atual. Dados podem vir de uma API backend (`/api/fs?path=/src`) que retorna a estrutura com tamanhos, ou de um JSON estático gerado por script. `InstancedMesh` é essencial para performance com milhares de arquivos.

```
Conceitos-chave: treemap layout (d3-hierarchy.treemap), ExtrudeGeometry,
                 InstancedMesh, duplo-clique para drill-down, fetch por nível
```

---

### Quiz / Jogo Educativo 3D

Jogo de perguntas e respostas com cenário 3D: um "palco" com pódio, placar e opções de resposta como caixas 3D flutuantes ao redor do jogador. Cada caixa tem `CanvasTexture` com o texto da alternativa. Ao clicar (raycaster), a caixa selecionada anima — verde com partículas de confetti se correta, vermelho com shake se errada. Placar atualizado com `CanvasTexture` dinâmica. Timer visual com `RingGeometry` que encolhe ao longo do tempo. Perguntas carregadas via `fetch()` de uma API como Open Trivia DB. Progressão de fases com transição de câmera entre cenários. Em modo multiplayer (via WebSocket), avatares de outros jogadores aparecem como esferas com iniciais.

```
Conceitos-chave: raycaster para seleção de resposta, CanvasTexture dinâmica
                 para textos e placar, RingGeometry como timer, fetch para
                 Open Trivia API, partículas de confetti, WebSocket multiplayer
```

---

## 19. Referências e Leitura Complementar

---

### Documentação Oficial

**Canvas e SVG:**
- [MDN — Canvas API](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API)
- [MDN — Tutorial Canvas](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API/Tutorial)
- [MDN — SVG](https://developer.mozilla.org/pt-BR/docs/Web/SVG)
- [W3C — SVG Specification](https://www.w3.org/TR/SVG2/)

**WebGL e WebGPU:**
- [MDN — WebGL API](https://developer.mozilla.org/pt-BR/docs/Web/API/WebGL_API)
- [WebGL2 Fundamentals](https://webgl2fundamentals.org)
- [MDN — WebGPU API](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API)
- [WebGPU Specification](https://www.w3.org/TR/webgpu/)

**Frameworks 3D:**
- [Three.js — Documentação](https://threejs.org/docs/)
- [Three.js — Exemplos](https://threejs.org/examples/)
- [React Three Fiber — Documentação](https://docs.pmnd.rs/react-three-fiber)
- [Drei — Helpers para R3F](https://github.com/pmndrs/drei)
- [Babylon.js — Documentação](https://doc.babylonjs.com)
- [Babylon.js — Playground](https://playground.babylonjs.com)
- [A-Frame — Documentação](https://aframe.io/docs/)
- [AR.js — Documentação](https://ar-js-org.github.io/AR.js-Docs/)

**Visualização de Dados:**
- [D3.js — Documentação](https://d3js.org)
- [Observable — D3](https://observablehq.com/@d3)
- [Chart.js — Documentação](https://www.chartjs.org/docs/)
- [Pixi.js — Guias](https://pixijs.com/guides)

**Assets 3D:**
- [Blender — Documentação](https://docs.blender.org)
- [glTF — Especificação](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html)
- [Khronos — glTF Overview](https://www.khronos.org/gltf/)

---

### Cursos e Tutoriais Recomendados

| Recurso | Conteúdo | Tipo |
|---------|----------|------|
| [Three.js Journey](https://threejs-journey.com) | Curso completo de Three.js (fundamentos → avançado) | Pago |
| [Discover Three.js](https://discoverthreejs.com) | Livro online sobre Three.js | Gratuito |
| [WebGL2 Fundamentals](https://webgl2fundamentals.org) | Fundamentos de WebGL2 do zero | Gratuito |
| [The Book of Shaders](https://thebookofshaders.com) | Guia interativo de GLSL shaders | Gratuito |
| [A-Frame School](https://aframe.io/aframe-school/) | Tutorial interativo de A-Frame/VR | Gratuito |
| [Blender Guru](https://www.youtube.com/@blenderguru) | Tutoriais de Blender para iniciantes | Gratuito (YouTube) |
| [D3.js in Depth](https://www.d3indepth.com) | Guia completo de D3.js | Gratuito |
| [Babylon.js Getting Started](https://doc.babylonjs.com/journey) | Tutorial oficial de Babylon.js | Gratuito |

---

### Ferramentas e Recursos

**Ferramentas de desenvolvimento:**
- [Stats.js](https://github.com/mrdoob/stats.js/) — Monitor de FPS/MS/MB para WebGL
- [lil-gui](https://github.com/georgealways/lil-gui) — Painel de controle para tweaking de parâmetros
- [Spector.js](https://spector.babylonjs.com) — Debugger de WebGL (extensão do Chrome)
- [Shadertoy](https://www.shadertoy.com) — Editor e galeria de shaders GLSL
- [Three.js Editor](https://threejs.org/editor/) — Editor visual de cenas Three.js

**Pipeline de assets:**
- [glTF Validator](https://github.khronos.org/glTF-Validator/) — Validação de arquivos glTF/GLB
- [gltf-transform](https://gltf-transform.dev) — Otimização, compressão, inspeção de glTF
- [gltf-pipeline](https://github.com/CesiumGS/gltf-pipeline) — Conversão e compressão Draco
- [SVGO](https://github.com/svg/svgo) — Otimizador de SVG

**Fontes de assets:**
- [Poly Haven](https://polyhaven.com) — HDRIs, texturas PBR e modelos 3D (CC0)
- [Sketchfab](https://sketchfab.com) — Maior repositório de modelos 3D
- [Mixamo](https://www.mixamo.com) — Animações de personagens (gratuito)
- [Kenney](https://kenney.nl) — Assets de jogos (CC0)
- [ambientCG](https://ambientcg.com) — Texturas PBR e HDRIs (CC0)
