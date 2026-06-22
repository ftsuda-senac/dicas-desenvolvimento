# JavaScript e TypeScript

Referência consolidada sobre JavaScript moderno e TypeScript para desenvolvimento web.

> Referência completa: [MDN — JavaScript](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript) · [TypeScript Docs](https://www.typescriptlang.org/docs/)

---

## Sumário

1. [JavaScript — Fundamentos](#1-javascript--fundamentos)
2. [ES2015+ — Recursos Modernos](#2-es2015-recursos-modernos)
3. [Manipulação do DOM](#3-manipulação-do-dom)
4. [JSON](#4-json)
5. [Promise e async/await](#5-promise-e-asyncawait)
6. [Fetch API](#6-fetch-api)
7. [TypeScript](#7-typescript)
8. [Projeto TypeScript para Web (HTML, CSS e SCSS)](#8-projeto-typescript-para-web-html-css-e-scss)
9. [Exemplos Práticos](#9-exemplos-práticos)
10. [Recursos Avançados](#10-recursos-avançados)

---

## 1. JavaScript — Fundamentos

### Incluindo JavaScript na página — async e defer

> MDN: [script — async e defer](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/script#attr-async)

Por padrão, quando o browser encontra uma tag `<script>`, ele **pausa o parsing do HTML**, baixa e executa o script, e só então retoma. Isso pode atrasar a renderização da página.

```html
<!-- Comportamento padrão: bloqueia o parsing do HTML -->
<script src="app.js"></script>
```

Os atributos `defer` e `async` tornam o download do script não-bloqueante:

```html
<!-- defer: baixa em paralelo, executa após o HTML ser totalmente parseado -->
<script src="app.js" defer></script>

<!-- async: baixa em paralelo, executa assim que terminar (pode ser antes do DOM estar pronto) -->
<script src="analytics.js" async></script>
```

**Comparação:**

| | `<script>` | `defer` | `async` |
|---|---|---|---|
| **Download** | Bloqueia o parsing | Em paralelo | Em paralelo |
| **Execução** | Imediata (bloqueia) | Após o HTML ser parseado | Assim que termina o download |
| **Ordem entre scripts** | Mantida | Mantida | Não garantida |
| **Aguarda o DOM** | Não | Sim | Não |
| **Uso indicado** | — | Scripts que dependem do DOM | Scripts independentes (analytics, ads) |

**Boas práticas:**
- Usar `defer` como padrão para scripts que manipulam o DOM.
- Usar `async` apenas para scripts completamente independentes (sem depender do DOM ou de outros scripts).
- Scripts no `<head>` sem atributos bloqueiam a renderização — preferir `defer` ou mover para o final do `<body>`.
- Módulos ES (`type="module"`) já têm comportamento `defer` por padrão.

```html
<head>
  <!-- Módulo: defer automático -->
  <script type="module" src="app.js"></script>

  <!-- Script externo independente -->
  <script src="https://analytics.exemplo.com/script.js" async></script>
</head>
```

---

### Variáveis

```js
var   nome = 'Ana';   // escopo de função, içada (hoisting) — evitar
let   idade = 30;     // escopo de bloco, pode ser reatribuída
const PI = 3.14;      // escopo de bloco, não pode ser reatribuída
```

> **Regra:** preferir `const` por padrão; usar `let` quando precisar reatribuir; evitar `var`.

### Tipos de dados

```js
// Primitivos
let texto     = 'Olá';          // string
let numero    = 42;             // number (inteiros e decimais)
let decimal   = 3.14;          // number
let booleano  = true;          // boolean
let nulo      = null;          // null — ausência intencional de valor
let indefinido;                // undefined — variável declarada sem valor
let simbolo   = Symbol('id');  // symbol — identificador único

// Objeto (não primitivo)
let obj   = { nome: 'Ana', idade: 30 };
let arr   = [1, 2, 3];
let fn    = function() {};
```

**Verificar tipo:**
```js
typeof 'texto'     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object" — comportamento histórico do JS
typeof []          // "object"
typeof {}          // "object"
Array.isArray([])  // true — forma correta de verificar array
```

### Conversão de tipos

```js
// Para número
Number('42')      // 42
Number('3.14')    // 3.14
Number('')        // 0
Number('abc')     // NaN
parseInt('42px')  // 42   — para inteiro, ignora sufixo não numérico
parseFloat('3.5') // 3.5

// Para string
String(42)        // "42"
(42).toString()   // "42"
(255).toString(16) // "ff" — base hexadecimal

// Para boolean
Boolean(0)        // false
Boolean('')       // false
Boolean(null)     // false
Boolean(undefined)// false
Boolean(NaN)      // false
Boolean(1)        // true
Boolean('texto')  // true
Boolean([])       // true — array vazio é truthy!
Boolean({})       // true — objeto vazio é truthy!
```

### Operadores

```js
// Aritméticos
+  -  *  /       // soma, subtração, multiplicação, divisão
%               // resto da divisão (módulo)
**              // exponenciação (ES2016)
++  --          // incremento e decremento

// Atribuição
=   +=  -=  *=  /=  %=  **=

// Comparação
==   !=          // igualdade/diferença com coerção de tipo — evitar
===  !==         // igualdade/diferença estrita (tipo + valor) — preferir
>    <    >=  <=

// Lógicos
&&   // AND — retorna o primeiro valor falsy ou o último valor
||   // OR  — retorna o primeiro valor truthy ou o último valor
!    // NOT

// Nullish coalescing (ES2020)
??   // retorna o lado direito apenas se o esquerdo for null ou undefined
const nome = usuario.nome ?? 'Anônimo';

// Optional chaining (ES2020)
?.   // acessa propriedade sem lançar erro se o valor for null/undefined
const cidade = usuario?.endereco?.cidade;

// Ternário
const status = ativo ? 'Ativo' : 'Inativo';
```

### Estruturas de controle

```js
// if / else if / else
if (nota >= 7) {
  console.log('Aprovado');
} else if (nota >= 5) {
  console.log('Recuperação');
} else {
  console.log('Reprovado');
}

// switch
switch (diaSemana) {
  case 1:
    console.log('Segunda');
    break;
  case 2:
    console.log('Terça');
    break;
  default:
    console.log('Outro dia');
}

// for
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// for...of — itera sobre valores de iteráveis (array, string, Set, Map)
for (const item of lista) {
  console.log(item);
}

// for...in — itera sobre chaves de um objeto
for (const chave in objeto) {
  console.log(chave, objeto[chave]);
}

// while
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}

// do...while — executa ao menos uma vez
do {
  console.log(i);
  i++;
} while (i < 5);
```

### Funções

```js
// Declaração de função (hoisting — pode ser chamada antes da declaração)
function somar(a, b) {
  return a + b;
}

// Expressão de função (sem hoisting)
const subtrair = function(a, b) {
  return a - b;
};

// Parâmetros com valor padrão
function saudacao(nome = 'visitante') {
  return `Olá, ${nome}!`;
}

// Rest parameters — agrupa argumentos restantes em array
function somar(...numeros) {
  return numeros.reduce((acc, n) => acc + n, 0);
}
somar(1, 2, 3, 4); // 10

// Retorno implícito de objeto (parênteses obrigatórios)
const criar = (nome) => ({ nome, ativo: true });
```

### Closure

> MDN: [Closures](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Closures)

Uma **closure** (fechamento) é a capacidade de uma função de "lembrar" e acessar variáveis do escopo onde foi criada, mesmo após esse escopo ter encerrado a execução.

Em JavaScript, toda função cria uma closure — ela carrega consigo uma referência ao ambiente léxico onde foi definida.

```js
function criarContador() {
  let contagem = 0; // variável do escopo externo

  return function () {
    contagem++;          // acessa e modifica a variável do escopo pai
    return contagem;
  };
}

const contador = criarContador();
contador(); // 1
contador(); // 2
contador(); // 3

// Cada chamada de criarContador() cria um escopo independente
const outroContador = criarContador();
outroContador(); // 1 — não compartilha estado com `contador`
```

**Por que funciona?** Quando `criarContador` termina, `contagem` não é coletada pelo garbage collector porque a função interna ainda mantém uma referência a ela.

**Casos de uso práticos:**

```js
// 1. Encapsulamento — variável privada inacessível diretamente de fora
function criarConta(saldoInicial) {
  let saldo = saldoInicial; // "privado"

  return {
    depositar(valor) { saldo += valor; },
    sacar(valor) {
      if (valor > saldo) throw new Error('Saldo insuficiente');
      saldo -= valor;
    },
    verSaldo() { return saldo; },
  };
}

const conta = criarConta(100);
conta.depositar(50);
conta.verSaldo(); // 150
conta.saldo;      // undefined — inacessível diretamente

// 2. Fábrica de funções — gera funções especializadas
function multiplicarPor(fator) {
  return (numero) => numero * fator; // `fator` fica na closure
}

const dobrar = multiplicarPor(2);
const triplicar = multiplicarPor(3);

dobrar(5);    // 10
triplicar(5); // 15

// 3. Memoização — cache de resultados de funções puras
function memoizar(fn) {
  const cache = new Map();
  return function (...args) {
    const chave = JSON.stringify(args);
    if (cache.has(chave)) return cache.get(chave);
    const resultado = fn(...args);
    cache.set(chave, resultado);
    return resultado;
  };
}

const fatorial = memoizar(function f(n) {
  return n <= 1 ? 1 : n * f(n - 1);
});

fatorial(5); // 120 — calcula
fatorial(5); // 120 — retorna do cache

// 4. Debounce — adia a execução enquanto o evento continua disparando
function debounce(fn, delay) {
  let timerId;
  return function (...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn(...args), delay);
  };
}

const salvarRascunho = debounce((texto) => console.log('Salvando:', texto), 500);
// Só executa 500ms após o último keystroke
input.addEventListener('input', (e) => salvarRascunho(e.target.value));
```

**Armadilha clássica com `var` em loops:**

```js
// Problema: var tem escopo de função, não de bloco
// Todos os callbacks capturam a mesma variável `i`
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Imprime: 3, 3, 3

// Solução 1: usar let (escopo de bloco — cria uma nova variável por iteração)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Imprime: 0, 1, 2

// Solução 2: IIFE que captura o valor de i em cada iteração
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}
// Imprime: 0, 1, 2
```

---

### IIFE — Immediately Invoked Function Expression

> MDN: [IIFE](https://developer.mozilla.org/pt-BR/docs/Glossary/IIFE)

Uma **IIFE** é uma função definida e executada imediatamente, sem ser atribuída a uma variável. Cria um escopo isolado que não vaza variáveis para o escopo externo.

```js
// Sintaxe básica — parênteses ao redor da função tornam-na uma expressão
(function () {
  const mensagem = 'privada';
  console.log(mensagem);
})();

// Com arrow function
(() => {
  const mensagem = 'privada';
  console.log(mensagem);
})();

// Passando argumentos
((nome, versao) => {
  console.log(`App: ${nome} v${versao}`);
})('MeuApp', '1.0');

// Capturando o valor de retorno
const resultado = (() => {
  const x = 10;
  const y = 20;
  return x + y;
})();
console.log(resultado); // 30
```

**Por que usar?**

Antes do ES2015 (que introduziu `let`, `const` e módulos), a IIFE era a principal forma de evitar poluição do escopo global e conflitos entre scripts diferentes carregados na mesma página.

```js
// Sem IIFE: `contador` vaza para o escopo global
var contador = 0;

// Com IIFE: `contador` fica encapsulado
(function () {
  var contador = 0; // não conflita com nenhuma outra variável
  contador++;
  console.log(contador);
})();
```

**Uso moderno**

Com módulos ES (`import`/`export`) e `let`/`const`, a IIFE perdeu parte do seu papel de encapsulamento. Ainda é útil em alguns cenários:

```js
// Inicialização assíncrona em contextos sem top-level await
(async () => {
  const dados = await fetch('/api/config').then(r => r.json());
  inicializar(dados);
})();

// Escopo isolado para um bloco de lógica inline sem criar função nomeada
const config = (() => {
  const env = 'production';
  const base = env === 'production' ? 'https://api.exemplo.com' : 'http://localhost:3000';
  return { env, base, timeout: 5000 };
})();
```

---

### Strings — métodos essenciais

> MDN: [String](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/String)

```js
const texto = 'Olá, Mundo!';

// Tamanho
texto.length;                      // 11

// Busca
texto.includes('Mundo');           // true
texto.startsWith('Olá');           // true
texto.endsWith('!');               // true
texto.indexOf('o');                // 1  — primeira ocorrência (-1 se não encontrar)
texto.lastIndexOf('o');            // 8  — última ocorrência

// Extração
texto.slice(5, 10);                // 'Mundo'
texto.slice(-6);                   // 'Mundo!' — índice negativo conta do fim
texto.charAt(0);                   // 'O'
texto.at(-1);                      // '!' — índice negativo (ES2022)

// Transformação (retornam nova string — strings são imutáveis)
texto.toUpperCase();               // 'OLÁ, MUNDO!'
texto.toLowerCase();               // 'olá, mundo!'
texto.trim();                      // remove espaços do início e do fim
'  espaços  '.trimStart();         // 'espaços  '
'  espaços  '.trimEnd();           // '  espaços'
texto.replace('Mundo', 'JS');      // 'Olá, JS!' — substitui a primeira ocorrência
texto.replaceAll('o', '0');        // substitui todas as ocorrências
texto.repeat(3);                   // 'Olá, Mundo!Olá, Mundo!Olá, Mundo!'
texto.padStart(15, '*');           // '****Olá, Mundo!' — preenche à esquerda
texto.padEnd(15, '-');             // 'Olá, Mundo!----' — preenche à direita

// Divisão e junção
'a,b,c'.split(',');                // ['a', 'b', 'c']
'abc'.split('');                   // ['a', 'b', 'c'] — cada caractere
'abc'.split('', 2);                // ['a', 'b'] — limita resultados
['a', 'b', 'c'].join(' - ');       // 'a - b - c'

// Expressões regulares
'abc123'.match(/\d+/);             // ['123'] — primeira correspondência
'abc123def456'.match(/\d+/g);      // ['123', '456'] — todas (flag g)
'abc'.search(/b/);                 // 1 — índice da primeira correspondência
'olá mundo'.replace(/\s+/g, '-'); // 'olá-mundo'
/^\d+$/.test('123');              // true — testa se toda a string bate

// Acesso a caractere
texto[0];                          // 'O' — como array (somente leitura)
```

**Casos de uso comuns:**
```js
// Capitalizar primeira letra
const capitalizar = str => str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();

// Verificar string vazia ou só espaços
const vazia = str => str.trim().length === 0;

// Truncar com reticências
const truncar = (str, max) => str.length > max ? str.slice(0, max) + '...' : str;

// Converter kebab-case para camelCase
'meu-nome-composto'.split('-').map((p, i) => i === 0 ? p : capitalizar(p)).join('');
// 'meuNomeComposto'

// Extrair extensão de arquivo
const extensao = nome => nome.slice(nome.lastIndexOf('.') + 1);
extensao('imagem.png'); // 'png'
```

### Number — métodos e propriedades

> MDN: [Number](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Number)

```js
// Propriedades estáticas
Number.MAX_SAFE_INTEGER;   // 9007199254740991 (2^53 - 1)
Number.MIN_SAFE_INTEGER;   // -9007199254740991
Number.MAX_VALUE;          // maior número representável (~1.8e308)
Number.EPSILON;            // menor diferença entre dois floats (~2.2e-16)
Number.POSITIVE_INFINITY;  // Infinity
Number.NEGATIVE_INFINITY;  // -Infinity
Number.NaN;                // NaN

// Verificações estáticas (preferir aos equivalentes globais)
Number.isInteger(42);        // true
Number.isInteger(42.5);      // false
Number.isFinite(Infinity);   // false
Number.isFinite(42);         // true
Number.isNaN(NaN);           // true  — mais seguro que o global isNaN()
Number.isNaN('texto');       // false — global isNaN('texto') retornaria true
Number.isSafeInteger(9007199254740991);  // true
Number.isSafeInteger(9007199254740992);  // false — perde precisão

// Conversão
Number.parseInt('42px');     // 42
Number.parseFloat('3.14em'); // 3.14
```

**Métodos de instância:**

```js
const n = 1234.5678;

// Casas decimais (arredonda e retorna string)
n.toFixed(2);          // '1234.57'
n.toFixed(0);          // '1235'

// Dígitos significativos
n.toPrecision(6);      // '1234.57'
n.toPrecision(3);      // '1.23e+3'

// Base numérica
(255).toString(16);    // 'ff'  — hexadecimal
(255).toString(2);     // '11111111' — binário
(255).toString(8);     // '377' — octal

// Verificar se é um número válido antes de operar
const entrada = '42';
const num = Number(entrada);
if (!Number.isNaN(num) && Number.isFinite(num)) {
  console.log(num.toFixed(2)); // '42.00'
}
```

**Armadilhas comuns com ponto flutuante:**

```js
0.1 + 0.2 === 0.3;           // false — resultado é 0.30000000000000004

// Forma correta de comparar floats
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true

// Ou arredondar antes de comparar
parseFloat((0.1 + 0.2).toFixed(10)) === 0.3; // true
```

---

### Math — funções matemáticas

> MDN: [Math](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Math)

`Math` é um objeto estático (não é construtor) com constantes e funções matemáticas.

**Constantes:**

```js
Math.PI;      // 3.141592653589793
Math.E;       // 2.718281828459045 — base do logaritmo natural
Math.SQRT2;   // 1.4142135623730951
Math.LN2;     // 0.6931471805599453
Math.LOG2E;   // 1.4426950408889634
```

**Arredondamento:**

```js
Math.round(4.5);   // 5  — arredonda para o inteiro mais próximo
Math.round(4.4);   // 4
Math.ceil(4.1);    // 5  — sempre arredonda para cima
Math.floor(4.9);   // 4  — sempre arredonda para baixo
Math.trunc(4.9);   // 4  — remove a parte decimal (sem arredondar)
Math.trunc(-4.9);  // -4 — diferente de floor: floor(-4.9) === -5
```

**Valor absoluto, sinal e potências:**

```js
Math.abs(-42);         // 42
Math.sign(-5);         // -1
Math.sign(0);          // 0
Math.sign(5);          // 1

Math.pow(2, 10);       // 1024 — equivalente a 2 ** 10
Math.sqrt(16);         // 4
Math.cbrt(27);         // 3 — raiz cúbica
Math.hypot(3, 4);      // 5 — hipotenusa (raiz da soma dos quadrados)
```

**Mínimo, máximo e clamp:**

```js
Math.max(1, 5, 3, 9, 2);  // 9
Math.min(1, 5, 3, 9, 2);  // 1

// Com array — usar spread
Math.max(...[1, 5, 3, 9]); // 9

// Clamp — limitar um valor entre min e max
const clamp = (valor, min, max) => Math.min(Math.max(valor, min), max);
clamp(15, 0, 10);  // 10
clamp(-5, 0, 10);  // 0
clamp(7,  0, 10);  // 7
```

**Logaritmos e exponencial:**

```js
Math.log(Math.E);   // 1   — logaritmo natural (base e)
Math.log2(8);       // 3   — logaritmo base 2
Math.log10(1000);   // 3   — logaritmo base 10
Math.exp(1);        // 2.718... — e elevado ao expoente
```

**Trigonometria (ângulos em radianos):**

```js
const grausParaRad = (graus) => graus * (Math.PI / 180);

Math.sin(grausParaRad(30));  // 0.5
Math.cos(grausParaRad(60));  // 0.5
Math.tan(grausParaRad(45));  // 1
Math.atan2(1, 1);            // π/4 — ângulo entre dois pontos (y, x)
```

**Números aleatórios:**

```js
Math.random();                      // float em [0, 1)

// Inteiro aleatório entre min (inclusivo) e max (exclusivo)
const randInt = (min, max) => Math.floor(Math.random() * (max - min)) + min;
randInt(1, 7);   // simula dado de 6 faces: 1 a 6

// Inteiro aleatório entre min e max (ambos inclusivos)
const randIntInc = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
randIntInc(1, 6); // 1, 2, 3, 4, 5 ou 6

// Elemento aleatório de um array
const items = ['a', 'b', 'c', 'd'];
const aleatorio = items[randInt(0, items.length)];

// Cor hexadecimal aleatória
const corHex = () => '#' + Math.floor(Math.random() * 0xFFFFFF).toString(16).padStart(6, '0');
```

---

### Date — métodos essenciais

> MDN: [Date](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Date)

```js
// Criação
const agora   = new Date();                      // data e hora atual
const data1   = new Date('2025-03-20');          // a partir de string ISO
const data2   = new Date('2025-03-20T10:30:00'); // com hora
const data3   = new Date(2025, 2, 20, 10, 30);  // ano, mês (0-11), dia, hora, min
const data4   = new Date(0);                     // epoch: 1970-01-01T00:00:00Z

// Timestamp (milissegundos desde epoch)
Date.now();          // número — mais eficiente que new Date().getTime()
data1.getTime();     // número equivalente

// Leitura de componentes (baseado no horário local)
const d = new Date('2025-03-20T14:35:50');
d.getFullYear();     // 2025
d.getMonth();        // 2   — atenção: janeiro = 0, dezembro = 11
d.getDate();         // 20  — dia do mês
d.getDay();          // 4   — dia da semana (0=domingo, 6=sábado)
d.getHours();        // 14
d.getMinutes();      // 35
d.getSeconds();      // 50
d.getMilliseconds(); // 0

// Escrita de componentes (altera o objeto)
d.setFullYear(2026);
d.setMonth(0);       // janeiro
d.setDate(1);
d.setHours(9, 0, 0, 0); // hora, min, seg, ms
```

**Formatação:**
```js
const d = new Date('2025-03-20T14:35:00');

// Métodos nativos de conversão
d.toISOString();            // '2025-03-20T14:35:00.000Z' — UTC, formato ISO 8601
d.toLocaleDateString('pt-BR');               // '20/03/2025'
d.toLocaleDateString('pt-BR', {
  day: '2-digit', month: 'long', year: 'numeric'
});                                          // '20 de março de 2025'
d.toLocaleTimeString('pt-BR');               // '14:35:00'
d.toLocaleString('pt-BR');                   // '20/03/2025, 14:35:00'

// Intl.DateTimeFormat — mais controle e reutilizável
const fmt = new Intl.DateTimeFormat('pt-BR', {
  dateStyle: 'full',   // 'full' | 'long' | 'medium' | 'short'
  timeStyle: 'short',
});
fmt.format(d); // 'quinta-feira, 20 de março de 2025 às 14:35'
```

**Operações com datas:**
```js
const hoje = new Date();

// Diferença entre datas em dias
const outra = new Date('2025-12-31');
const diffMs   = outra - hoje;                      // subtração retorna ms
const diffDias = Math.ceil(diffMs / (1000 * 60 * 60 * 24));

// Adicionar dias
const amanha = new Date(hoje);
amanha.setDate(amanha.getDate() + 1);

// Adicionar meses (getTime preserva a hora)
const proximoMes = new Date(hoje);
proximoMes.setMonth(proximoMes.getMonth() + 1);

// Primeiro e último dia do mês
const primeiroDia = new Date(hoje.getFullYear(), hoje.getMonth(), 1);
const ultimoDia   = new Date(hoje.getFullYear(), hoje.getMonth() + 1, 0);

// Comparar datas
data1 > data2;           // compara timestamps
data1.getTime() === data2.getTime(); // igualdade exata
```

> **Atenção:** meses no `Date` são indexados de 0 (janeiro) a 11 (dezembro). Para projetos com manipulação intensiva de datas, considerar bibliotecas como **date-fns** ou **Day.js**.

### Arrays — métodos essenciais

> MDN: [Array](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array)

```js
const nums = [3, 1, 4, 1, 5, 9, 2, 6];

// Iteração
nums.forEach(n => console.log(n));

// Transformação — retorna novo array
nums.map(n => n * 2);             // [6, 2, 8, 2, 10, 18, 4, 12]

// Filtro — retorna novo array com elementos que passam no teste
nums.filter(n => n > 3);          // [4, 5, 9, 6]

// Redução — retorna um único valor acumulado
nums.reduce((acc, n) => acc + n, 0); // 31

// Busca
nums.find(n => n > 4);            // 5 — primeiro que satisfaz
nums.findIndex(n => n > 4);       // 4 — índice do primeiro que satisfaz
nums.includes(9);                 // true
nums.some(n => n > 8);            // true — ao menos um satisfaz
nums.every(n => n > 0);           // true — todos satisfazem

// Ordenação (altera o array original)
nums.sort((a, b) => a - b);       // crescente
nums.sort((a, b) => b - a);       // decrescente

// Adição e remoção
nums.push(7);                     // adiciona no final
nums.pop();                       // remove do final
nums.unshift(0);                  // adiciona no início
nums.shift();                     // remove do início
nums.splice(2, 1);                // remove 1 elemento no índice 2
nums.splice(2, 0, 99);            // insere 99 no índice 2

// Slicing — retorna novo array (não altera o original)
nums.slice(1, 4);                 // elementos do índice 1 ao 3

// Achatar array aninhado
[[1, 2], [3, 4]].flat();          // [1, 2, 3, 4]
[[1, [2, [3]]]].flat(Infinity);   // [1, 2, 3] — todos os níveis

// Combinar map + flat em uma operação
['Ana', 'Bob'].flatMap(n => [n, n.toUpperCase()]);
// ["Ana", "ANA", "Bob", "BOB"]

// Verificar e converter
Array.isArray([]);                // true
Array.from('abc');                // ["a", "b", "c"]
Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]
```

### Objetos — operações essenciais

> MDN: [Object](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object)

#### Criação e acesso

```js
// Literal (forma mais comum)
const pessoa = { nome: 'Ana', idade: 30, cidade: 'SP' };

// Acesso a propriedades
pessoa.nome;           // "Ana" — notação de ponto
pessoa['nome'];        // "Ana" — notação de colchetes (útil para chaves dinâmicas)

const campo = 'email';
pessoa[campo];         // equivale a pessoa.email

// Shorthand properties (ES2015) — quando variável e chave têm o mesmo nome
const nome = 'Ana';
const idade = 30;
const obj = { nome, idade }; // equivale a { nome: nome, idade: idade }

// Shorthand methods (ES2015)
const usuario = {
  nome: 'Ana',
  saudar() {             // equivale a saudar: function() { ... }
    return `Olá, ${this.nome}!`;
  },
};

// Propriedade computada (ES2015) — chave definida em tempo de execução
const prefixo = 'get';
const api = {
  [prefixo + 'Nome']() { return 'Ana'; },  // método getNome()
  [`${prefixo}Idade`]() { return 30; },    // método getIdade()
};
```

#### Métodos estáticos essenciais

```js
const pessoa = { nome: 'Ana', idade: 30, cidade: 'SP' };

// Listar chaves, valores e entradas
Object.keys(pessoa);    // ["nome", "idade", "cidade"]
Object.values(pessoa);  // ["Ana", 30, "SP"]
Object.entries(pessoa); // [["nome","Ana"], ["idade",30], ["cidade","SP"]]

// Criar objeto a partir de entradas (ES2019) — inverso de Object.entries
Object.fromEntries([['nome', 'Ana'], ['idade', 30]]); // { nome: "Ana", idade: 30 }

// Útil para transformar objetos via Map
const precos = { banana: 1.5, maca: 2.0, uva: 3.5 };
const comDesconto = Object.fromEntries(
  Object.entries(precos).map(([chave, val]) => [chave, val * 0.9])
);
// { banana: 1.35, maca: 1.8, uva: 3.15 }

// Copiar/mesclar objetos (shallow copy — cópia rasa)
const copia = { ...pessoa };
const mesclado = { ...pessoa, profissao: 'Dev' };
const mesclado2 = Object.assign({}, pessoa, { profissao: 'Dev' });

// Propriedades posteriores sobrescrevem as anteriores
const a = { x: 1, y: 2 };
const b = { y: 99, z: 3 };
const merged = { ...a, ...b }; // { x: 1, y: 99, z: 3 }

// Verificar propriedade
'nome' in pessoa;                    // true — inclui propriedades herdadas
pessoa.hasOwnProperty('nome');       // true — apenas propriedades próprias
Object.hasOwn(pessoa, 'nome');       // true — forma moderna (ES2022), preferir
Object.hasOwn(pessoa, 'toString');   // false — toString é herdado do protótipo

// Comparação de identidade (Object.is — mais rigoroso que ===)
Object.is(NaN, NaN);   // true  — diferente de NaN === NaN (false)
Object.is(0, -0);      // false — diferente de 0 === -0 (true)
Object.is(1, 1);       // true
```

#### Imutabilidade

```js
const config = { host: 'localhost', porta: 3000, debug: true };

// Object.freeze — impede adição, remoção e alteração de propriedades (raso)
const frozen = Object.freeze(config);
frozen.porta = 9000;    // silencioso em modo normal, TypeError em strict mode
frozen.porta;           // 3000 — não foi alterado
Object.isFrozen(frozen); // true

// Object.seal — impede adição e remoção, mas permite alterar valores existentes
const sealed = Object.seal({ x: 1, y: 2 });
sealed.x = 99;   // permitido — alteração de valor
sealed.z = 3;    // ignorado — adição bloqueada
delete sealed.x; // ignorado — remoção bloqueada
Object.isSealed(sealed); // true

// Atenção: freeze e seal são rasos — objetos aninhados não são protegidos
const app = Object.freeze({ config: { debug: true } });
app.config.debug = false; // FUNCIONA — config aponta para objeto não congelado
app.config = {};          // bloqueado — a propriedade config em si não muda

// Deep freeze (recursivo)
function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return Object.freeze(obj);
}
```

#### Descritores de propriedade

```js
// Object.defineProperty — controle fino sobre uma propriedade
const obj = {};
Object.defineProperty(obj, 'id', {
  value: 42,
  writable: false,    // não pode ser reatribuída
  enumerable: false,  // não aparece em for...in, Object.keys, spread
  configurable: false // não pode ser deletada ou redefinida
});

obj.id;           // 42
obj.id = 99;      // silencioso (ou TypeError em strict mode)
Object.keys(obj); // [] — id não é enumerável
'id' in obj;      // true — ainda está lá

// Getters e setters via defineProperty
const pessoa = { _nome: 'Ana' };
Object.defineProperty(pessoa, 'nome', {
  get() { return this._nome.toUpperCase(); },
  set(val) { this._nome = val.trim(); },
  enumerable: true,
  configurable: true,
});

pessoa.nome;         // "ANA"
pessoa.nome = '  Bob  ';
pessoa._nome;        // "Bob"

// Getter/setter na notação literal (mais comum)
const circulo = {
  raio: 5,
  get area() { return Math.PI * this.raio ** 2; },
  set diametro(d) { this.raio = d / 2; },
};
circulo.area;        // 78.53...
circulo.diametro = 20;
circulo.raio;        // 10

// Inspecionar descritor
Object.getOwnPropertyDescriptor(pessoa, '_nome');
// { value: "Bob", writable: true, enumerable: true, configurable: true }
```

#### Clonagem profunda

```js
// structuredClone (ES2022) — deep clone nativo, sem dependências externas
const original = {
  nome: 'Ana',
  endereco: { cidade: 'SP', bairro: 'Centro' },
  tags: ['dev', 'js'],
};

const clone = structuredClone(original);
clone.endereco.cidade = 'RJ';  // não afeta o original
original.endereco.cidade;       // "SP"

// structuredClone suporta: Date, Map, Set, RegExp, ArrayBuffer, etc.
// Não suporta: funções, nós do DOM, objetos com referências circulares complexas

// Alternativa legada (não recomendada — perde Date, Map, Set, undefined, etc.)
const cloneLegado = JSON.parse(JSON.stringify(original));
```

#### Padrões utilitários

```js
// Filtrar propriedades (selecionar apenas algumas chaves)
const usuario = { id: 1, nome: 'Ana', senha: 'secret', role: 'admin' };

// Pick — manter apenas as chaves desejadas
const publico = Object.fromEntries(
  Object.entries(usuario).filter(([k]) => ['id', 'nome', 'role'].includes(k))
);
// { id: 1, nome: "Ana", role: "admin" }

// Omit — excluir chaves específicas
const semSenha = Object.fromEntries(
  Object.entries(usuario).filter(([k]) => k !== 'senha')
);
// { id: 1, nome: "Ana", role: "admin" }

// Agrupar array de objetos por propriedade (Object.groupBy — ES2024)
const produtos = [
  { nome: 'Caneta', categoria: 'escritório' },
  { nome: 'Borracha', categoria: 'escritório' },
  { nome: 'Mouse', categoria: 'tecnologia' },
];
const porCategoria = Object.groupBy(produtos, p => p.categoria);
// { escritório: [...], tecnologia: [...] }

// Verificar se objeto está vazio
const vazio = obj => Object.keys(obj).length === 0;
vazio({});         // true
vazio({ a: 1 });   // false

// Mescla profunda simples (sem bibliotecas)
function mergeDeep(alvo, ...fontes) {
  for (const fonte of fontes) {
    for (const [chave, val] of Object.entries(fonte)) {
      alvo[chave] =
        val && typeof val === 'object' && !Array.isArray(val)
          ? mergeDeep(alvo[chave] ?? {}, val)
          : val;
    }
  }
  return alvo;
}
```

---

## 2. ES2015+ Recursos Modernos

> MDN: [JavaScript — Novidades em cada versão](https://developer.mozilla.org/en-US/docs/Web/JavaScript/JavaScript_technologies_overview)

### Arrow Functions (ES2015)

```js
// Função tradicional
function dobrar(n) { return n * 2; }

// Arrow function
const dobrar = (n) => n * 2;
const dobrar = n => n * 2;       // parênteses opcionais com 1 parâmetro

// Múltiplas linhas exigem chaves e return explícito
const processar = (a, b) => {
  const resultado = a + b;
  return resultado * 2;
};

// Diferença importante: arrow functions não têm próprio `this`
// Herdam o `this` do contexto em que foram criadas
class Timer {
  constructor() {
    this.segundos = 0;
  }
  iniciar() {
    // Arrow: `this` é o Timer — correto
    setInterval(() => { this.segundos++; }, 1000);
    // Function: `this` seria undefined (strict) ou window — incorreto
  }
}
```

### Template Literals (ES2015)

```js
const nome = 'Ana';
const idade = 30;

// Interpolação de variáveis e expressões
console.log(`Olá, ${nome}! Você tem ${idade} anos.`);
console.log(`Daqui a 5 anos terá ${idade + 5} anos.`);

// Strings multilinhas
const html = `
  <div class='card'>
    <h2>${nome}</h2>
    <p>Idade: ${idade}</p>
  </div>
`;
```

### Destructuring (ES2015)

```js
// Array destructuring
const [primeiro, segundo, ...restante] = [1, 2, 3, 4, 5];
// primeiro=1, segundo=2, restante=[3,4,5]

// Ignorar elementos
const [, , terceiro] = [10, 20, 30];  // terceiro=30

// Com valor padrão
const [a = 0, b = 0] = [5];          // a=5, b=0

// Object destructuring
const { nome, idade, cidade = 'SP' } = pessoa;

// Renomear ao desestruturar
const { nome: nomeCompleto, idade: anos } = pessoa;

// Aninhado
const { endereco: { rua, numero } } = usuario;

// Em parâmetros de função
function exibir({ nome, idade }) {
  console.log(`${nome}, ${idade} anos`);
}

// Troca de valores sem variável auxiliar
let x = 1, y = 2;
[x, y] = [y, x];  // x=2, y=1
```

### Spread e Rest (ES2015)

```js
// Spread — expande iterável em elementos individuais
const a = [1, 2, 3];
const b = [4, 5, 6];
const todos = [...a, ...b];          // [1, 2, 3, 4, 5, 6]

const original = { x: 1, y: 2 };
const extendido = { ...original, z: 3 }; // { x:1, y:2, z:3 }

Math.max(...a);                      // 3

// Cópia rasa de array e objeto
const copiaArr = [...a];
const copiaObj = { ...original };

// Rest — agrupa argumentos restantes
function log(primeiro, ...outros) {
  console.log(primeiro); // primeiro argumento
  console.log(outros);   // array com os demais
}
```

### Classes (ES2015)

```js
class Animal {
  // Campo de classe (ES2022)
  #nome;  // campo privado (não acessível fora da classe)

  constructor(nome, som) {
    this.#nome = nome;
    this.som = som;
  }

  // Getter
  get nome() {
    return this.#nome;
  }

  // Método de instância
  falar() {
    return `${this.#nome} faz ${this.som}`;
  }

  // Método estático — chamado na classe, não na instância
  static criar(nome, som) {
    return new Animal(nome, som);
  }
}

// Herança
class Cachorro extends Animal {
  constructor(nome) {
    super(nome, 'Au Au');  // chama o construtor do pai
    this.raca = null;
  }

  // Sobrescrita de método
  falar() {
    return super.falar() + '!';
  }
}

const dog = new Cachorro('Rex');
dog.falar();       // "Rex faz Au Au!"
dog.nome;          // "Rex" — via getter
dog.#nome;         // SyntaxError — campo privado
```

### Modules (ES2015)

```js
// math.js — exportações
export const PI = 3.14159;

export function somar(a, b) { return a + b; }

export default class Calculadora {  // exportação padrão (uma por arquivo)
  multiplicar(a, b) { return a * b; }
}

// app.js — importações
import Calculadora from './math.js';          // importação padrão
import { PI, somar } from './math.js';        // importações nomeadas
import { somar as add } from './math.js';     // com alias
import * as math from './math.js';            // tudo como namespace
```

### Map e Set (ES2015)

```js
// Map — dicionário com qualquer tipo de chave
const mapa = new Map();
mapa.set('nome', 'Ana');
mapa.set(42, 'número como chave');
mapa.set({ id: 1 }, 'objeto como chave');

mapa.get('nome');      // "Ana"
mapa.has('nome');      // true
mapa.size;             // 3
mapa.delete('nome');

for (const [chave, valor] of mapa) {
  console.log(chave, valor);
}

// Set — coleção de valores únicos
const conjunto = new Set([1, 2, 2, 3, 3, 3]);
// Set { 1, 2, 3 } — duplicatas removidas

conjunto.add(4);
conjunto.has(2);       // true
conjunto.size;         // 4
conjunto.delete(1);

// Remover duplicatas de array
const semDuplicatas = [...new Set([1, 2, 2, 3])]; // [1, 2, 3]
```

### Encadeamento Opcional e Nullish Coalescing (ES2020)

```js
// Optional chaining (?.) — não lança erro se valor for null/undefined
const usuario = { perfil: { cidade: 'SP' } };

usuario?.perfil?.cidade;         // "SP"
usuario?.endereco?.cep;          // undefined (sem erro)
usuario?.getSaldo?.();           // undefined se getSaldo não existir
lista?.[0];                      // undefined se lista for null/undefined

// Nullish coalescing (??) — fallback apenas para null e undefined
// Diferente de || que considera qualquer valor falsy (0, "", false)
const saldo = usuario.saldo ?? 0;  // 0 se saldo for null/undefined
const nome = '' ?? 'Anônimo';      // "" (string vazia não é null/undefined)
const nome2 = '' || 'Anônimo';     // "Anônimo" (|| trata "" como falsy)

// Atribuição com nullish (ES2021)
usuario.apelido ??= 'Sem apelido';  // atribui apenas se for null/undefined
usuario.score ||= 0;                 // atribui se for qualquer valor falsy
usuario.tentativas &&= usuario.tentativas + 1; // atribui se for truthy
```

### Iteração avançada

```js
// Object.entries + destructuring
for (const [chave, valor] of Object.entries(objeto)) {
  console.log(`${chave}: ${valor}`);
}

// Array.from com função de mapeamento
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]

// structuredClone (ES2022) — cópia profunda nativa
const original = { a: { b: 1 } };
const clone = structuredClone(original);
clone.a.b = 99;
original.a.b; // 1 — não foi afetado
```

---

## 3. Manipulação do DOM

> MDN: [DOM](https://developer.mozilla.org/pt-BR/docs/Web/API/Document_Object_Model) · [Document](https://developer.mozilla.org/pt-BR/docs/Web/API/Document)

O **DOM** (Document Object Model) é a representação em árvore da página HTML, acessível e manipulável via JavaScript.

### O objeto window

> MDN: [Window](https://developer.mozilla.org/pt-BR/docs/Web/API/Window)

`window` é o objeto global do navegador — o contexto raiz de todo código JavaScript executado em uma página. Variáveis e funções declaradas no escopo global tornam-se propriedades de `window`. Todos os outros objetos globais (`document`, `navigator`, `location`, `history`, `console`) são propriedades dele.

```js
// O prefixo window. é opcional na maioria dos casos
window.console.log('ok') === console.log('ok');
window.document      === document;
window.setTimeout    === setTimeout;
```

#### globalThis

> MDN: [globalThis](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/globalThis)

O objeto global tem nomes diferentes dependendo do ambiente de execução:

| Ambiente | Objeto global |
|---|---|
| Navegador | `window` |
| Node.js | `global` |
| Web Worker | `self` |
| Módulos ES (qualquer ambiente) | `undefined` (sem global implícito) |

**`globalThis`** é uma referência padronizada (ES2020) ao objeto global de qualquer ambiente, eliminando a necessidade de detectar o contexto manualmente.

```js
// Antes do globalThis — detecção manual frágil
const global =
  typeof window !== 'undefined' ? window :
  typeof global !== 'undefined' ? global :
  typeof self   !== 'undefined' ? self   : undefined;

// Com globalThis — funciona em qualquer ambiente
globalThis.setTimeout(() => console.log('ok'), 100);

// Verificar se um recurso existe globalmente, independente do ambiente
if (typeof globalThis.fetch === 'function') {
  // fetch disponível
}

// Adicionar propriedade ao escopo global (usar com cautela)
globalThis.minhaConfig = { versao: '1.0', debug: false };
```

**Casos de uso práticos:**

```js
// Biblioteca que precisa rodar no navegador e no Node.js
function obterStorage() {
  if (typeof globalThis.localStorage !== 'undefined') {
    return globalThis.localStorage;
  }
  // fallback para Node.js: usar Map em memória
  return new Map();
}

// Polyfill seguro: adiciona recurso apenas se não existir
if (!globalThis.crypto?.randomUUID) {
  globalThis.crypto = {
    ...globalThis.crypto,
    randomUUID: () => Math.random().toString(36).slice(2),
  };
}
```

> **Atenção:** no navegador, `globalThis === window` é `true`. Em módulos ES (`type="module"`), o escopo global ainda existe via `globalThis`, mas variáveis declaradas com `let`/`const`/`class` no topo do módulo **não** se tornam propriedades dele — apenas `var` e declarações de função clássica fazem isso.

**Dimensões e posição:**

```js
// Tamanho da área visível (viewport), sem barras do navegador
window.innerWidth;    // largura em px
window.innerHeight;   // altura em px

// Tamanho da janela inteira incluindo barras do navegador
window.outerWidth;
window.outerHeight;

// Posição do scroll atual da página
window.scrollX;       // deslocamento horizontal (alias: pageXOffset)
window.scrollY;       // deslocamento vertical  (alias: pageYOffset)

// Rolar a página
window.scrollTo(0, 500);                        // posição absoluta
window.scrollTo({ top: 500, behavior: 'smooth' });
window.scrollBy(0, 200);                        // deslocamento relativo
window.scrollBy({ top: 200, behavior: 'smooth' });
```

**Navegação e URL:**

```js
// Informações da URL atual
window.location.href;       // URL completa
window.location.origin;     // 'https://exemplo.com'
window.location.pathname;   // '/produtos/lista'
window.location.search;     // '?pagina=2&ordem=nome'
window.location.hash;       // '#secao-1'
window.location.host;       // 'exemplo.com:3000'

// Navegar para outra URL
window.location.href = 'https://outro.com';
window.location.assign('https://outro.com');   // equivalente, cria histórico
window.location.replace('https://outro.com');  // sem criar entrada no histórico

// Recarregar a página
window.location.reload();

// Histórico de navegação
window.history.back();                         // equivale ao botão Voltar
window.history.forward();
window.history.go(-2);                         // volta 2 páginas
window.history.pushState({}, '', '/nova-url'); // muda URL sem recarregar
window.history.replaceState({}, '', '/outra'); // substitui sem criar histórico

// Informações do navegador e dispositivo
window.navigator.userAgent;        // string de identificação do navegador
window.navigator.language;         // 'pt-BR'
window.navigator.onLine;           // true se há conexão de rede
window.navigator.clipboard;        // Clipboard API
window.navigator.serviceWorker;    // Service Worker API
```

**Diálogos nativos:**

```js
// Caixa de alerta (bloqueia execução — evitar em produção)
alert('Mensagem');

// Caixa de confirmação — retorna true ou false
const confirmado = confirm('Deseja continuar?');

// Caixa de entrada de texto — retorna string ou null (se cancelar)
const nome = prompt('Qual é o seu nome?', 'Visitante'); // segundo arg: valor padrão
```

**Timers:**

```js
// Executar uma vez após o delay (ms)
const id = setTimeout(() => console.log('executou'), 1000);
clearTimeout(id); // cancela antes de executar

// Executar repetidamente a cada intervalo
const id = setInterval(() => console.log('tick'), 500);
clearInterval(id); // cancela o intervalo

// Executar antes do próximo repaint (animações, leitura de layout)
requestAnimationFrame((timestamp) => {
  // timestamp: ms desde o carregamento da página
  console.log('frame:', timestamp);
});
```

**Outros recursos úteis:**

```js
// Abrir nova aba ou janela
const novaJanela = window.open('https://exemplo.com', '_blank');
novaJanela?.close();

// Detectar evento de redimensionamento
window.addEventListener('resize', () => {
  console.log(window.innerWidth, window.innerHeight);
});

// Detectar scroll da página
window.addEventListener('scroll', () => {
  console.log('scrollY:', window.scrollY);
});

// Detectar quando o usuário está prestes a sair da página
window.addEventListener('beforeunload', (e) => {
  e.preventDefault();
  // Alguns navegadores exibem um diálogo de confirmação
});

// Executar após todo o conteúdo (imagens, scripts) ser carregado
window.addEventListener('load', () => console.log('tudo carregado'));

// Executar assim que o HTML foi parseado (sem esperar imagens)
document.addEventListener('DOMContentLoaded', () => console.log('DOM pronto'));
```

### Seleção de elementos

```js
// Retorna o primeiro elemento correspondente
document.querySelector('.card');          // por classe
document.querySelector('#titulo');        // por ID
document.querySelector('input[required]');// por atributo
document.querySelector('nav > a.active'); // seletor composto

// Retorna NodeList com todos os correspondentes
document.querySelectorAll('.card');
document.querySelectorAll('li:nth-child(odd)');

// Métodos legados (ainda úteis para IDs e classes)
document.getElementById('titulo');
document.getElementsByClassName('card'); // HTMLCollection (live)
document.getElementsByTagName('li');     // HTMLCollection (live)

// Busca dentro de um elemento
const lista = document.querySelector('.lista');
lista.querySelectorAll('li');            // apenas <li> dentro de .lista
```

### Leitura e modificação de conteúdo

```js
const el = document.querySelector('.card');

// Conteúdo de texto
el.textContent;               // texto puro (sem HTML)
el.textContent = 'Novo texto';

// Conteúdo HTML (cuidado com XSS ao inserir dados do usuário)
el.innerHTML;                 // HTML interno como string
el.innerHTML = '<strong>Negrito</strong>';

// Conteúdo HTML fora do elemento
el.outerHTML;                 // inclui o próprio elemento na string
```

### Classes e atributos

```js
const btn = document.querySelector('.btn');

// classList — API recomendada para classes
btn.classList.add('ativo');
btn.classList.remove('desabilitado');
btn.classList.toggle('aberto');           // adiciona se não tem, remove se tem
btn.classList.toggle('aberto', condicao); // adiciona/remove conforme boolean
btn.classList.contains('ativo');          // true/false
btn.classList.replace('antigo', 'novo');

// Atributos
btn.getAttribute('type');               // "button"
btn.setAttribute('disabled', '');       // adiciona atributo
btn.removeAttribute('disabled');
btn.hasAttribute('disabled');           // true/false

// Propriedades diretas (para atributos comuns)
btn.disabled = true;
btn.value;
btn.checked;  // checkbox/radio
```

### Estilos inline

```js
const el = document.querySelector('.box');

// Leitura e escrita de propriedade inline
el.style.backgroundColor = '#2563eb';
el.style.display = 'none';
el.style.setProperty('--cor-custom', '#ff0');  // variável CSS

// Estilos computados (incluindo CSS externo)
const estilos = window.getComputedStyle(el);
estilos.getPropertyValue('font-size');   // "16px"
estilos.getPropertyValue('--cor-custom');// valor da variável CSS
```

### Criação e inserção de elementos

```js
// Criar
const li = document.createElement('li');
li.textContent = 'Novo item';
li.classList.add('item');

// Inserção
const lista = document.querySelector('ul');
lista.appendChild(li);                    // no final (legado)
lista.append(li);                         // no final (moderno, aceita string)
lista.prepend(li);                        // no início
lista.insertBefore(li, lista.firstChild); // antes de um elemento

// Inserção relativa a um elemento de referência
const ref = document.querySelector('.ref');
ref.before(li);         // imediatamente antes
ref.after(li);          // imediatamente após
ref.replaceWith(li);    // substitui o elemento

// insertAdjacentHTML — inserção de HTML sem recriar elementos existentes
el.insertAdjacentHTML('beforebegin', '<p>antes do elemento</p>');
el.insertAdjacentHTML('afterbegin',  '<p>primeiro filho</p>');
el.insertAdjacentHTML('beforeend',   '<p>último filho</p>');
el.insertAdjacentHTML('afterend',    '<p>após o elemento</p>');
```

### Remoção e travessia

```js
// Remover
el.remove();                    // remove o próprio elemento
el.parentElement.removeChild(el); // forma legada

// Travessia (navegação entre nós)
el.parentElement;               // elemento pai
el.children;                    // filhos (HTMLCollection, apenas elementos)
el.firstElementChild;           // primeiro filho elemento
el.lastElementChild;            // último filho elemento
el.nextElementSibling;          // próximo irmão
el.previousElementSibling;      // irmão anterior
el.closest('.card');            // ancestral mais próximo que bate no seletor
el.matches('.ativo');           // true se o elemento bate no seletor
```

### Eventos

> MDN: [Events](https://developer.mozilla.org/pt-BR/docs/Web/Events)

Existem duas formas de associar eventos a elementos. A forma mais antiga usa propriedades diretamente no elemento (`onclick`, `onsubmit`, `onchange`, etc.) — só permite um handler por evento e mistura comportamento com estrutura. A forma moderna e recomendada é `addEventListener`, que permite múltiplos listeners, controle de fase e remoção posterior.

```js
// Forma antiga — evitar (sobrescreve qualquer handler anterior)
btn.onclick = () => console.log('clicado');
form.onsubmit = (e) => e.preventDefault();

// Forma moderna — preferir
btn.addEventListener('click', handler);
```

```js
const btn = document.querySelector('#btn');

// Adicionar listener
btn.addEventListener('click', function(event) {
  console.log('clicado', event.target);
});

// Remover listener (a função deve ter referência nomeada)
function handleClick(event) { /* ... */ }
btn.addEventListener('click', handleClick);
btn.removeEventListener('click', handleClick);

// Opções do addEventListener
btn.addEventListener('click', handler, {
  once: true,     // executa apenas uma vez e remove o listener
  passive: true,  // indica que não chama preventDefault (melhora scroll)
  capture: true,  // captura na fase de descida (antes do elemento alvo)
});
```

**Eventos comuns:**

| Categoria | Eventos |
|---|---|
| Documento | `DOMContentLoaded`, `load`, `resize`, `scroll` |
| Mouse | `click`, `dblclick`, `mousedown`, `mouseup`, `mouseover`, `mouseout`, `mousemove`, `contextmenu` |
| Touch | `touchstart`, `touchmove`, `touchend` |
| Teclado | `keydown`, `keyup`, `keypress` (depreciado) |
| Formulário | `submit`, `change`, `input`, `focus`, `blur`, `reset` |
| Drag | `dragstart`, `dragover`, `drop`, `dragend` |


**O objeto Event — propriedades e métodos essenciais:**

> MDN: [Event](https://developer.mozilla.org/pt-BR/docs/Web/API/Event)

Todo handler de evento recebe um objeto `Event` como argumento. Ele contém informações sobre o que aconteceu, onde aconteceu e permite controlar a propagação.

**`event.target` vs `event.currentTarget`:**

`target` é o elemento que **originou** o evento (onde o clique/digitação de fato ocorreu). `currentTarget` é o elemento onde o listener **está registrado**. Quando não há delegação, ambos apontam para o mesmo elemento. Com delegação, são diferentes.

```js
// HTML: <ul id="lista"> <li><button>Excluir</button></li> </ul>
document.querySelector('#lista').addEventListener('click', (e) => {
  console.log(e.target);        // <button> — onde o clique ocorreu de fato
  console.log(e.currentTarget); // <ul#lista> — onde o listener está registrado
});
```

```js
// Sem delegação — target e currentTarget são o mesmo
const btn = document.querySelector('#meu-btn');
btn.addEventListener('click', (e) => {
  console.log(e.target === e.currentTarget); // true
});

// Com delegação — são diferentes
document.querySelector('ul').addEventListener('click', (e) => {
  console.log(e.target);        // o <li> ou filho que foi clicado
  console.log(e.currentTarget); // o <ul> pai
});
```

| Propriedade | Descrição |
|---|---|
| `e.target` | Elemento que originou o evento (o mais interno que foi clicado) |
| `e.currentTarget` | Elemento onde o `addEventListener` foi chamado |
| `e.type` | Tipo do evento (`"click"`, `"submit"`, `"keydown"`, etc.) |
| `e.timeStamp` | Milissegundos desde o carregamento da página até o disparo |

**`event.preventDefault()`** — cancela o comportamento padrão do navegador para aquele evento. O evento continua propagando normalmente — apenas a ação nativa é impedida.

```js
// Impedir envio do formulário (recarregamento da página)
document.querySelector('form').addEventListener('submit', (e) => {
  e.preventDefault();
  // processar dados com JS em vez de enviar ao servidor
  const dados = new FormData(e.currentTarget);
  console.log(Object.fromEntries(dados));
});

// Impedir navegação de um link
document.querySelector('a.interno').addEventListener('click', (e) => {
  e.preventDefault();
  // navegar via JS (SPA)
  console.log('Navegar para:', e.currentTarget.href);
});

// Impedir digitação de caracteres não numéricos
document.querySelector('input.somente-numeros').addEventListener('keydown', (e) => {
  if (!/[\d]/.test(e.key) && !['Backspace', 'Tab', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
    e.preventDefault();
  }
});
```

**`event.stopPropagation()`** — interrompe a propagação do evento pela árvore do DOM. Eventos no DOM propagam em duas fases: **captura** (da raiz até o alvo) e **bubbling** (do alvo de volta até a raiz). `stopPropagation()` impede que o evento continue para o próximo elemento na cadeia.

```js
// Problema: clicar no botão dentro do card dispara o handler do card também
document.querySelector('.card').addEventListener('click', () => {
  console.log('Card clicado — abrir detalhes');
});

document.querySelector('.card .btn-fechar').addEventListener('click', (e) => {
  e.stopPropagation(); // impede que o clique chegue ao .card
  console.log('Fechar card');
});
```

```js
// Exemplo prático: modal que fecha ao clicar no fundo (overlay),
// mas NÃO fecha ao clicar dentro do conteúdo
document.querySelector('.modal-overlay').addEventListener('click', () => {
  fecharModal();
});

document.querySelector('.modal-conteudo').addEventListener('click', (e) => {
  e.stopPropagation(); // cliques dentro do conteúdo não alcançam o overlay
});
```

**Propagação de eventos — captura e bubbling:**

```
          Fase de captura (↓)          Fase de bubbling (↑)
          ──────────────────          ────────────────────
          document                    document
            ↓                           ↑
          <html>                      <html>
            ↓                           ↑
          <body>                      <body>
            ↓                           ↑
          <div class="card">          <div class="card">
            ↓                           ↑
          <button> ← alvo (target) → <button>
```

```js
// Por padrão, listeners são executados na fase de bubbling (↑)
pai.addEventListener('click', () => console.log('pai'));
filho.addEventListener('click', () => console.log('filho'));
// Clique no filho → imprime: "filho", "pai"

// Com capture: true, o listener executa na fase de descida (↓)
pai.addEventListener('click', () => console.log('pai (captura)'), { capture: true });
filho.addEventListener('click', () => console.log('filho'));
// Clique no filho → imprime: "pai (captura)", "filho"
```

**Delegação de eventos — um listener no pai captura eventos dos filhos:**

```js
// Útil quando os filhos são criados dinamicamente
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.matches('li')) {
    console.log('Item clicado:', e.target.textContent);
  }
});
```

**Eventos de document, window e navigator:**

Eventos globais relacionados ao ciclo de vida da página, conectividade, navegação e visibilidade. Muitos deles são essenciais para criar aplicações que reagem ao estado do ambiente em que estão rodando.

**Ciclo de vida da página:**

| Evento | Disparado em | Quando dispara |
|---|---|---|
| `DOMContentLoaded` | `document` | HTML parseado e DOM pronto (sem esperar imagens, CSS, fontes) |
| `load` | `window` | Tudo carregado (imagens, CSS, scripts, iframes) |
| `beforeunload` | `window` | Usuário está prestes a sair da página (fechar aba, navegar) |
| `unload` | `window` | Página está sendo descarregada (evitar — usar `beforeunload` ou `visibilitychange`) |

```js
// DOMContentLoaded — momento ideal para inicializar scripts que manipulam o DOM
// Mais rápido que load: não espera imagens e recursos externos
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM pronto — pode manipular elementos');
  inicializarApp();
});

// load — útil quando precisa de dimensões de imagens ou recursos completos
window.addEventListener('load', () => {
  console.log('Tudo carregado — imagens, fontes, CSS');
  calcularLayoutComImagens();
});

// beforeunload — avisar sobre dados não salvos
// O navegador exibe um diálogo genérico de confirmação (mensagem personalizada é ignorada)
let dadosAlterados = false;

window.addEventListener('beforeunload', (e) => {
  if (dadosAlterados) {
    e.preventDefault(); // dispara o diálogo de confirmação do navegador
  }
});

// Marcar como alterado quando o formulário muda
document.querySelector('form')?.addEventListener('input', () => {
  dadosAlterados = true;
});
```

**Visibilidade da página:**

> MDN: [Page Visibility API](https://developer.mozilla.org/pt-BR/docs/Web/API/Page_Visibility_API)

`visibilitychange` dispara quando o usuário troca de aba, minimiza o navegador ou volta à página. Útil para pausar vídeos, economizar requisições ou atualizar dados ao retornar.

```js
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'hidden') {
    // Página não está visível — pausar atividades
    pausarVideo();
    pararPolling();
    console.log('Aba em segundo plano');
  } else {
    // Página visível novamente — retomar
    retomarVideo();
    atualizarDados();
    console.log('Aba em primeiro plano');
  }
});

// document.visibilityState: "visible" | "hidden"
// document.hidden: true | false (atalho booleano)
```

```js
// Exemplo: pausar polling de API quando a aba não está ativa
let intervaloId;

function iniciarPolling() {
  intervaloId = setInterval(() => {
    fetch('/api/notificacoes').then(r => r.json()).then(atualizarUI);
  }, 5000);
}

function pararPolling() {
  clearInterval(intervaloId);
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) pararPolling();
  else iniciarPolling();
});

iniciarPolling();
```

**Conectividade — online e offline:**

```js
// Detectar mudança de conexão de rede
window.addEventListener('online', () => {
  console.log('Conexão restaurada');
  sincronizarDadosPendentes();
  mostrarNotificacao('Você está online novamente');
});

window.addEventListener('offline', () => {
  console.log('Conexão perdida');
  mostrarNotificacao('Sem conexão — alterações serão salvas localmente');
});

// Verificar estado atual
console.log('Online:', navigator.onLine); // true ou false
```

```js
// Exemplo: fila de requisições offline
const filaPendente = [];

async function enviarDado(dado) {
  if (navigator.onLine) {
    await fetch('/api/dados', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(dado),
    });
  } else {
    filaPendente.push(dado);
    localStorage.setItem('filaPendente', JSON.stringify(filaPendente));
  }
}

window.addEventListener('online', async () => {
  const pendentes = JSON.parse(localStorage.getItem('filaPendente') || '[]');
  for (const dado of pendentes) {
    await fetch('/api/dados', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(dado),
    });
  }
  localStorage.removeItem('filaPendente');
});
```

**Navegação — popstate e hashchange:**

> MDN: [popstate](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/popstate_event) · [hashchange](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/hashchange_event)

```js
// popstate — dispara ao navegar pelo histórico (botões voltar/avançar)
// NÃO dispara quando pushState/replaceState são chamados
window.addEventListener('popstate', (e) => {
  console.log('Navegação no histórico');
  console.log('Estado:', e.state); // objeto passado em pushState/replaceState

  // Atualizar a interface conforme a URL atual
  renderizarPagina(window.location.pathname);
});

// Criar entrada no histórico (popstate dispara quando o usuário volta)
history.pushState({ pagina: 'produtos' }, '', '/produtos');
history.pushState({ pagina: 'detalhe', id: 42 }, '', '/produtos/42');
```

```js
// hashchange — dispara quando o fragmento (#) da URL muda
// Útil para SPAs baseadas em hash
window.addEventListener('hashchange', (e) => {
  console.log('Hash anterior:', new URL(e.oldURL).hash);
  console.log('Hash atual:', new URL(e.newURL).hash);
  console.log('Hash:', window.location.hash);

  // Roteamento simples baseado em hash
  switch (window.location.hash) {
    case '#/inicio':   renderizar('inicio'); break;
    case '#/sobre':    renderizar('sobre'); break;
    case '#/contato':  renderizar('contato'); break;
    default:           renderizar('404');
  }
});
```

**Redimensionamento e scroll:**

```js
// resize — dispara em alta frequência ao redimensionar a janela
// Sempre usar debounce ou throttle para evitar performance ruim
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    console.log(`Viewport: ${window.innerWidth} x ${window.innerHeight}`);
    ajustarLayout();
  }, 150);
});
```

```js
// Alternativa moderna: ResizeObserver — observa redimensionamento de elementos específicos
const observer = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    console.log(`Elemento redimensionado: ${width} x ${height}`);
  }
});

observer.observe(document.querySelector('.container'));
```

```js
// scroll — dispara em alta frequência ao rolar a página
// Exemplo: mostrar botão "voltar ao topo" após rolar 300px
window.addEventListener('scroll', () => {
  const btnTopo = document.querySelector('.btn-topo');
  btnTopo.style.display = window.scrollY > 300 ? 'block' : 'none';
});

// Detectar se o usuário chegou ao final da página (infinite scroll)
window.addEventListener('scroll', () => {
  const distanciaDoFim = document.documentElement.scrollHeight
    - window.scrollY
    - window.innerHeight;

  if (distanciaDoFim < 200) {
    carregarMaisItens();
  }
});
```

**Storage — comunicação entre abas:**

> MDN: [storage event](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/storage_event)

O evento `storage` dispara em **outras abas** do mesmo domínio quando `localStorage` é alterado. Não dispara na aba que fez a alteração.

```js
// Aba 1: salva dado
localStorage.setItem('tema', 'escuro');

// Aba 2: detecta a mudança automaticamente
window.addEventListener('storage', (e) => {
  console.log('Chave alterada:', e.key);      // 'tema'
  console.log('Valor anterior:', e.oldValue); // 'claro'
  console.log('Valor novo:', e.newValue);     // 'escuro'
  console.log('URL de origem:', e.url);

  if (e.key === 'tema') {
    aplicarTema(e.newValue);
  }
});
```

```js
// Exemplo: logout sincronizado entre abas
// Quando o usuário faz logout em uma aba, todas as outras abas também deslogam

// Na aba que faz logout:
function logout() {
  localStorage.setItem('logout', Date.now().toString());
  window.location.href = '/login';
}

// Nas outras abas:
window.addEventListener('storage', (e) => {
  if (e.key === 'logout') {
    window.location.href = '/login';
  }
});
```

**Clipboard — copiar e colar:**

```js
// copy — usuário copiou conteúdo (Ctrl+C ou menu de contexto)
document.addEventListener('copy', (e) => {
  e.preventDefault();
  const selecao = document.getSelection().toString();
  e.clipboardData.setData('text/plain', selecao.toUpperCase());
  // Copia o texto selecionado em maiúsculas
});

// paste — usuário colou conteúdo (Ctrl+V)
document.addEventListener('paste', (e) => {
  e.preventDefault();
  const texto = e.clipboardData.getData('text/plain');
  console.log('Texto colado:', texto);
  // Inserir texto filtrado/sanitizado no campo ativo
  document.execCommand('insertText', false, texto.trim());
});

// cut — usuário recortou conteúdo (Ctrl+X)
document.addEventListener('cut', (e) => {
  console.log('Conteúdo recortado');
});
```

```js
// Clipboard API assíncrona — copiar programaticamente (requer HTTPS)
async function copiarTexto(texto) {
  try {
    await navigator.clipboard.writeText(texto);
    console.log('Copiado para a área de transferência');
  } catch (err) {
    console.error('Falha ao copiar:', err);
  }
}

async function lerClipboard() {
  const texto = await navigator.clipboard.readText();
  console.log('Conteúdo:', texto);
}

// Exemplo: botão "copiar código"
document.querySelector('.btn-copiar').addEventListener('click', async () => {
  const codigo = document.querySelector('pre code').textContent;
  await copiarTexto(codigo);
  // Feedback visual temporário
  const btn = document.querySelector('.btn-copiar');
  btn.textContent = 'Copiado!';
  setTimeout(() => btn.textContent = 'Copiar', 2000);
});
```

**Mídia e preferências do sistema:**

```js
// matchMedia — reagir a media queries via JavaScript
// Útil para lógica condicional que depende de breakpoints ou preferências do sistema
const mediaEscuro = window.matchMedia('(prefers-color-scheme: dark)');

// Verificar estado atual
if (mediaEscuro.matches) {
  aplicarTemaEscuro();
}

// Reagir a mudanças (ex.: usuário altera tema do sistema)
mediaEscuro.addEventListener('change', (e) => {
  if (e.matches) aplicarTemaEscuro();
  else aplicarTemaClaro();
});
```

```js
// Detectar mudança de orientação (dispositivos móveis)
const mediaRetrato = window.matchMedia('(orientation: portrait)');

mediaRetrato.addEventListener('change', (e) => {
  console.log('Orientação:', e.matches ? 'retrato' : 'paisagem');
});

// Detectar se o usuário prefere menos animações (acessibilidade)
const prefereReduzir = window.matchMedia('(prefers-reduced-motion: reduce)');

if (prefereReduzir.matches) {
  desativarAnimacoes();
}
```

**Eventos de mouse — MouseEvent:**

> MDN: [MouseEvent](https://developer.mozilla.org/pt-BR/docs/Web/API/MouseEvent)

| Evento | Quando dispara |
|---|---|
| `click` | Após `mousedown` + `mouseup` no mesmo elemento |
| `dblclick` | Dois cliques rápidos |
| `mousedown` / `mouseup` | Botão pressionado / solto |
| `mouseover` / `mouseout` | Cursor entra / sai do elemento (dispara ao passar por filhos) |
| `mouseenter` / `mouseleave` | Cursor entra / sai do elemento (não dispara ao passar por filhos) |
| `mousemove` | Cursor se move sobre o elemento (alta frequência) |
| `contextmenu` | Clique com botão direito |

```js
const area = document.querySelector('.area');

// Propriedades de posição do cursor
area.addEventListener('mousemove', (e) => {
  e.clientX; // posição X relativa à viewport (área visível)
  e.clientY; // posição Y relativa à viewport
  e.pageX;   // posição X relativa ao documento (inclui scroll)
  e.pageY;   // posição Y relativa ao documento
  e.offsetX; // posição X relativa ao elemento alvo
  e.offsetY; // posição Y relativa ao elemento alvo
  e.screenX; // posição X relativa à tela do monitor
  e.screenY; // posição Y relativa à tela do monitor
});
```

```js
// Identificar qual botão do mouse foi pressionado
area.addEventListener('mousedown', (e) => {
  switch (e.button) {
    case 0: console.log('Botão esquerdo'); break;
    case 1: console.log('Botão do meio (scroll)'); break;
    case 2: console.log('Botão direito'); break;
  }
});

// Detectar teclas modificadoras pressionadas junto com o clique
area.addEventListener('click', (e) => {
  if (e.ctrlKey)  console.log('Ctrl + clique');
  if (e.shiftKey) console.log('Shift + clique');
  if (e.altKey)   console.log('Alt + clique');
  if (e.metaKey)  console.log('Meta (⌘/Win) + clique');
});
```

```js
// Exemplo: arrastar um elemento com mousedown + mousemove + mouseup
const caixa = document.querySelector('.arrastavel');

caixa.addEventListener('mousedown', (e) => {
  const deltaX = e.clientX - caixa.offsetLeft;
  const deltaY = e.clientY - caixa.offsetTop;

  function mover(e) {
    caixa.style.left = (e.clientX - deltaX) + 'px';
    caixa.style.top = (e.clientY - deltaY) + 'px';
  }

  document.addEventListener('mousemove', mover);
  document.addEventListener('mouseup', () => {
    document.removeEventListener('mousemove', mover);
  }, { once: true });
});
```

```js
// mouseover/mouseout vs mouseenter/mouseleave
// mouseover/mouseout disparam ao entrar/sair de CADA filho interno
// mouseenter/mouseleave disparam apenas ao entrar/sair do elemento em si
const card = document.querySelector('.card');

card.addEventListener('mouseover', () => console.log('over'));
// Dispara múltiplas vezes ao mover o cursor entre filhos dentro do card

card.addEventListener('mouseenter', () => console.log('enter'));
// Dispara apenas uma vez ao entrar no card (não redispara ao passar pelos filhos)
```

```js
// Menu de contexto personalizado (clique direito)
document.querySelector('.area-custom').addEventListener('contextmenu', (e) => {
  e.preventDefault(); // impede o menu padrão do navegador
  const menu = document.querySelector('.menu-contexto');
  menu.style.left = e.clientX + 'px';
  menu.style.top = e.clientY + 'px';
  menu.style.display = 'block';
});

document.addEventListener('click', () => {
  document.querySelector('.menu-contexto').style.display = 'none';
});
```

**Eventos de toque — TouchEvent:**

> MDN: [TouchEvent](https://developer.mozilla.org/pt-BR/docs/Web/API/TouchEvent) · [Touch](https://developer.mozilla.org/pt-BR/docs/Web/API/Touch)

Eventos de toque são disparados em dispositivos com tela sensível ao toque (smartphones, tablets). Diferente do mouse, o toque suporta **múltiplos pontos simultâneos** (multitouch).

| Evento | Quando dispara |
|---|---|
| `touchstart` | Dedo toca a tela |
| `touchmove` | Dedo se move enquanto está na tela |
| `touchend` | Dedo é levantado da tela |
| `touchcancel` | Toque é interrompido pelo sistema (ex.: notificação, gesto do navegador) |

**Propriedades do TouchEvent:**

```js
const area = document.querySelector('.area-touch');

area.addEventListener('touchstart', (e) => {
  // Listas de toques (cada uma é uma TouchList)
  e.touches;        // todos os dedos atualmente na tela
  e.targetTouches;  // dedos que estão sobre o elemento alvo
  e.changedTouches; // dedos que mudaram neste evento (iniciaram, moveram ou saíram)

  // Acessar o primeiro toque
  const toque = e.touches[0];
  toque.clientX;    // posição X relativa à viewport
  toque.clientY;    // posição Y relativa à viewport
  toque.pageX;      // posição X relativa ao documento (inclui scroll)
  toque.pageY;      // posição Y relativa ao documento
  toque.screenX;    // posição X relativa à tela do dispositivo
  toque.screenY;    // posição Y relativa à tela do dispositivo
  toque.identifier; // ID único do dedo (útil para rastrear múltiplos toques)
  toque.target;     // elemento onde o toque iniciou
});
```

**Interação entre touch e mouse:**

Em dispositivos touch, o navegador dispara eventos na seguinte ordem: `touchstart` → `touchend` → `mousemove` → `mousedown` → `mouseup` → `click`. Para evitar que o evento seja processado duas vezes, use `preventDefault()` no handler de touch.

```js
area.addEventListener('touchstart', (e) => {
  e.preventDefault(); // impede que o navegador dispare mousedown/click depois
  // processar o toque
});
```

**Exemplo: detectar swipe (deslizar):**

```js
function onSwipe(el, callback) {
  let inicioX, inicioY, inicioTempo;
  const DISTANCIA_MIN = 50;  // px mínimos para considerar swipe
  const TEMPO_MAX = 300;     // ms máximos para o gesto

  el.addEventListener('touchstart', (e) => {
    const toque = e.changedTouches[0];
    inicioX = toque.clientX;
    inicioY = toque.clientY;
    inicioTempo = Date.now();
  });

  el.addEventListener('touchend', (e) => {
    const toque = e.changedTouches[0];
    const deltaX = toque.clientX - inicioX;
    const deltaY = toque.clientY - inicioY;
    const tempo = Date.now() - inicioTempo;

    if (tempo > TEMPO_MAX) return;

    // Verifica se o movimento horizontal é maior que o vertical
    if (Math.abs(deltaX) > Math.abs(deltaY) && Math.abs(deltaX) > DISTANCIA_MIN) {
      callback(deltaX > 0 ? 'direita' : 'esquerda');
    } else if (Math.abs(deltaY) > DISTANCIA_MIN) {
      callback(deltaY > 0 ? 'baixo' : 'cima');
    }
  });
}

// Uso: navegação de carrossel por swipe
const carrossel = document.querySelector('.carrossel');
onSwipe(carrossel, (direcao) => {
  if (direcao === 'esquerda') avancarSlide();
  if (direcao === 'direita') voltarSlide();
});
```

**Exemplo: arrastar elemento com touch (equivalente ao drag com mouse):**

```js
const caixa = document.querySelector('.arrastavel');

caixa.addEventListener('touchstart', (e) => {
  e.preventDefault();
  const toque = e.targetTouches[0];
  const deltaX = toque.clientX - caixa.offsetLeft;
  const deltaY = toque.clientY - caixa.offsetTop;

  function mover(e) {
    const toque = e.targetTouches[0];
    caixa.style.left = (toque.clientX - deltaX) + 'px';
    caixa.style.top = (toque.clientY - deltaY) + 'px';
  }

  function soltar() {
    caixa.removeEventListener('touchmove', mover);
    caixa.removeEventListener('touchend', soltar);
  }

  caixa.addEventListener('touchmove', mover);
  caixa.addEventListener('touchend', soltar);
});
```

**Exemplo: pinch to zoom (dois dedos):**

```js
function onPinch(el, callback) {
  let distanciaInicial = 0;

  function distanciaEntreToques(t1, t2) {
    const dx = t1.clientX - t2.clientX;
    const dy = t1.clientY - t2.clientY;
    return Math.hypot(dx, dy);
  }

  el.addEventListener('touchstart', (e) => {
    if (e.touches.length === 2) {
      distanciaInicial = distanciaEntreToques(e.touches[0], e.touches[1]);
    }
  });

  el.addEventListener('touchmove', (e) => {
    if (e.touches.length === 2) {
      e.preventDefault(); // impede zoom nativo da página
      const distanciaAtual = distanciaEntreToques(e.touches[0], e.touches[1]);
      const escala = distanciaAtual / distanciaInicial;
      callback(escala); // escala > 1 = afastando (zoom in), < 1 = aproximando (zoom out)
    }
  });

  el.addEventListener('touchend', () => {
    distanciaInicial = 0;
  });
}

// Uso
const imagem = document.querySelector('.imagem-zoom');
onPinch(imagem, (escala) => {
  imagem.style.transform = `scale(${escala})`;
});
```

**Pointer Events — alternativa unificada:**

> MDN: [Pointer Events](https://developer.mozilla.org/pt-BR/docs/Web/API/Pointer_events)

`PointerEvent` unifica mouse, touch e caneta em uma única API — um handler funciona para todos os dispositivos sem duplicar código.

| Pointer Event | Equivalente mouse | Equivalente touch |
|---|---|---|
| `pointerdown` | `mousedown` | `touchstart` |
| `pointermove` | `mousemove` | `touchmove` |
| `pointerup` | `mouseup` | `touchend` |
| `pointerenter` | `mouseenter` | — |
| `pointerleave` | `mouseleave` | — |
| `pointercancel` | — | `touchcancel` |

```js
const caixa = document.querySelector('.arrastavel');

caixa.addEventListener('pointerdown', (e) => {
  caixa.setPointerCapture(e.pointerId); // garante que events continuem no elemento
  const deltaX = e.clientX - caixa.offsetLeft;
  const deltaY = e.clientY - caixa.offsetTop;

  function mover(e) {
    caixa.style.left = (e.clientX - deltaX) + 'px';
    caixa.style.top = (e.clientY - deltaY) + 'px';
  }

  caixa.addEventListener('pointermove', mover);
  caixa.addEventListener('pointerup', () => {
    caixa.removeEventListener('pointermove', mover);
  }, { once: true });
});
```

```js
// Propriedades exclusivas do PointerEvent
area.addEventListener('pointerdown', (e) => {
  e.pointerId;    // ID único do ponteiro (útil para multitouch)
  e.pointerType;  // "mouse", "touch" ou "pen"
  e.pressure;     // pressão do toque/caneta (0.0 a 1.0; mouse sempre 0.5)
  e.width;        // largura da área de contato (touch)
  e.height;       // altura da área de contato (touch)
  e.isPrimary;    // true se é o ponteiro principal (primeiro dedo, mouse)
});
```

> **Recomendação:** para novos projetos, preferir **Pointer Events** em vez de usar mouse + touch separadamente. A API é suportada em todos os navegadores modernos e elimina a necessidade de escrever handlers duplicados.

**Eventos de teclado — KeyboardEvent:**

> MDN: [KeyboardEvent](https://developer.mozilla.org/pt-BR/docs/Web/API/KeyboardEvent)

| Evento | Quando dispara |
|---|---|
| `keydown` | Tecla pressionada (dispara repetidamente se mantida pressionada) |
| `keyup` | Tecla solta |
| ~~`keypress`~~ | **Depreciado** — não usar; substituído por `keydown` |

```js
document.addEventListener('keydown', (e) => {
  e.key;    // valor legível da tecla: "a", "Enter", "ArrowUp", " " (espaço), "Shift"
  e.code;   // tecla física no teclado: "KeyA", "Enter", "ArrowUp", "Space", "ShiftLeft"
  e.repeat; // true se a tecla está sendo mantida pressionada (repetição automática)

  // Teclas modificadoras (mesmo comportamento que no MouseEvent)
  e.ctrlKey;  // true se Ctrl está pressionado
  e.shiftKey; // true se Shift está pressionado
  e.altKey;   // true se Alt está pressionado
  e.metaKey;  // true se Meta (⌘ no Mac, Win no Windows) está pressionado
});
```

**`key` vs `code`:**

`key` reflete o caractere produzido (varia conforme layout do teclado e idioma). `code` identifica a tecla física (sempre o mesmo, independente do layout).

```js
// Em um teclado ABNT2, pressionar a tecla ";" (ponto-e-vírgula):
// e.key  → ";" (o caractere produzido)
// e.code → "Semicolon" (a tecla física)

// A tecla "A" com e sem Shift:
// Sem Shift: e.key → "a",  e.code → "KeyA"
// Com Shift: e.key → "A",  e.code → "KeyA" (code não muda)

// Quando usar qual:
// - key: atalhos baseados no caractere (Ctrl+S, Ctrl+Z)
// - code: jogos e controles direcionais (WASD sempre nas mesmas teclas físicas)
```

```js
// Exemplo: atalhos de teclado
document.addEventListener('keydown', (e) => {
  // Ctrl+S — salvar (impede o comportamento padrão do navegador)
  if (e.ctrlKey && e.key === 's') {
    e.preventDefault();
    salvarDocumento();
  }

  // Escape — fechar modal
  if (e.key === 'Escape') {
    fecharModal();
  }

  // Ctrl+Enter — enviar formulário
  if (e.ctrlKey && e.key === 'Enter') {
    enviarFormulario();
  }
});
```

```js
// Exemplo: navegação com setas (ex.: carrossel, galeria)
document.addEventListener('keydown', (e) => {
  switch (e.key) {
    case 'ArrowLeft':  voltarSlide(); break;
    case 'ArrowRight': avancarSlide(); break;
    case 'ArrowUp':    e.preventDefault(); scrollParaCima(); break;
    case 'ArrowDown':  e.preventDefault(); scrollParaBaixo(); break;
  }
});
```

```js
// Exemplo: campo de busca com debounce no keydown
const inputBusca = document.querySelector('#busca');
let timerId;

inputBusca.addEventListener('keydown', (e) => {
  // Ignorar teclas que não produzem caractere
  if (e.key === 'Shift' || e.key === 'Control' || e.key === 'Alt') return;

  clearTimeout(timerId);
  timerId = setTimeout(() => {
    buscar(inputBusca.value);
  }, 300);
});
```

```js
// Valores comuns de e.key
// Letras e números: "a", "A", "1", "0"
// Setas:           "ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"
// Controle:        "Enter", "Escape", "Tab", "Backspace", "Delete"
// Modificadores:   "Shift", "Control", "Alt", "Meta"
// Espaço:          " " (string com um espaço)
// Função:          "F1", "F2", ..., "F12"
```

**Eventos de formulário:**

> MDN: [HTMLFormElement — Events](https://developer.mozilla.org/pt-BR/docs/Web/API/HTMLFormElement#events) · [HTMLInputElement — Events](https://developer.mozilla.org/pt-BR/docs/Web/API/HTMLInputElement#events)

| Evento | Disparado em | Quando dispara |
|---|---|---|
| `submit` | `<form>` | Formulário é enviado (botão submit ou Enter) |
| `reset` | `<form>` | Formulário é resetado |
| `input` | `<input>`, `<textarea>`, `<select>` | Valor muda a cada digitação/interação (tempo real) |
| `change` | `<input>`, `<textarea>`, `<select>` | Valor muda e o campo perde o foco (ou ao selecionar em `<select>` e checkbox/radio) |
| `focus` | qualquer elemento focável | Elemento recebe foco |
| `blur` | qualquer elemento focável | Elemento perde foco |
| `focusin` | qualquer elemento focável | Como `focus`, mas propaga (bubbling) — útil para delegação |
| `focusout` | qualquer elemento focável | Como `blur`, mas propaga (bubbling) |
| `invalid` | `<input>`, `<textarea>`, `<select>` | Validação nativa do campo falha ao tentar enviar o formulário |

**`input` vs `change`:**

```js
const campo = document.querySelector('#nome');

// input — dispara a cada tecla digitada (tempo real)
campo.addEventListener('input', (e) => {
  console.log('digitando:', e.target.value);
  // Útil para: busca ao vivo, contagem de caracteres, preview em tempo real
});

// change — dispara quando o valor muda E o campo perde o foco
campo.addEventListener('change', (e) => {
  console.log('valor confirmado:', e.target.value);
  // Útil para: salvar dado, validar após preenchimento, atualizar estado
});

// Em <select>, checkbox e radio, change dispara imediatamente ao selecionar
document.querySelector('select').addEventListener('change', (e) => {
  console.log('selecionou:', e.target.value);
});
```

**`focus` / `blur` e `focusin` / `focusout`:**

```js
const campo = document.querySelector('#email');

// focus — quando o campo recebe foco (não propaga)
campo.addEventListener('focus', () => {
  campo.parentElement.classList.add('campo-ativo');
});

// blur — quando o campo perde foco (não propaga)
campo.addEventListener('blur', () => {
  campo.parentElement.classList.remove('campo-ativo');
});

// focusin/focusout — mesma função, mas propagam (bubbling)
// Permitem usar delegação no formulário inteiro
document.querySelector('form').addEventListener('focusin', (e) => {
  e.target.closest('.campo-grupo')?.classList.add('campo-ativo');
});

document.querySelector('form').addEventListener('focusout', (e) => {
  e.target.closest('.campo-grupo')?.classList.remove('campo-ativo');
});
```

**`submit` e `reset`:**

```js
const form = document.querySelector('#meu-form');

// submit — interceptar envio do formulário
form.addEventListener('submit', (e) => {
  e.preventDefault(); // impede envio nativo (recarregamento da página)

  const dados = new FormData(form);
  console.log('Dados:', Object.fromEntries(dados));

  // enviar via fetch, validar, etc.
});

// reset — interceptar limpeza do formulário
form.addEventListener('reset', (e) => {
  // executa ANTES dos campos serem limpos
  const confirma = confirm('Deseja limpar todos os campos?');
  if (!confirma) e.preventDefault(); // cancela o reset
});
```

**Validação de formulários com JavaScript:**

> MDN: [Constraint Validation API](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Constraint_validation)

O HTML5 oferece validação nativa via atributos (`required`, `type`, `pattern`, `min`, `max`, `minlength`, `maxlength`). O JavaScript pode complementá-la com a **Constraint Validation API** para mensagens customizadas e validações mais complexas.

**Atributos de validação HTML:**

```html
<form id="form-cadastro" novalidate>
  <!-- novalidate desabilita os tooltips nativos do navegador -->
  <!-- a validação será feita via JavaScript para controle total -->

  <label>Nome *
    <input type="text" name="nome" required minlength="3" maxlength="100" />
  </label>

  <label>E-mail *
    <input type="email" name="email" required />
  </label>

  <label>Idade
    <input type="number" name="idade" min="1" max="150" />
  </label>

  <label>CPF
    <input type="text" name="cpf" pattern="\d{3}\.\d{3}\.\d{3}-\d{2}"
           title="Formato: 000.000.000-00" />
  </label>

  <label>Senha *
    <input type="password" name="senha" required minlength="8" />
  </label>

  <label>Site
    <input type="url" name="site" placeholder="https://exemplo.com" />
  </label>

  <button type="submit">Cadastrar</button>
</form>
```

**Constraint Validation API — propriedades e métodos:**

```js
const campo = document.querySelector('input[name="email"]');

// Verificar se o campo é válido
campo.validity;            // objeto ValidityState com detalhes do estado
campo.validity.valid;      // true se todas as restrições passam
campo.validity.valueMissing;   // true se required e vazio
campo.validity.typeMismatch;   // true se type="email" e valor não é e-mail válido
campo.validity.patternMismatch;// true se não bate com o atributo pattern
campo.validity.tooShort;       // true se menor que minlength
campo.validity.tooLong;        // true se maior que maxlength
campo.validity.rangeUnderflow; // true se menor que min (number/date)
campo.validity.rangeOverflow;  // true se maior que max (number/date)
campo.validity.stepMismatch;   // true se não respeita o step
campo.validity.customError;    // true se setCustomValidity foi chamado com mensagem

// Métodos
campo.checkValidity();     // retorna boolean e dispara evento 'invalid' se inválido
campo.reportValidity();    // como checkValidity, mas exibe tooltip nativo do navegador
campo.setCustomValidity('mensagem'); // define erro personalizado
campo.setCustomValidity('');         // limpa o erro personalizado (campo volta a ser válido)
```

**Validação ao perder o foco (blur) — feedback imediato:**

```js
const form = document.querySelector('#form-cadastro');

function validarCampo(campo) {
  const grupo = campo.closest('label') || campo.parentElement;
  let erroEl = grupo.querySelector('.erro');

  // Criar elemento de erro se não existir
  if (!erroEl) {
    erroEl = document.createElement('span');
    erroEl.className = 'erro';
    erroEl.setAttribute('role', 'alert');
    grupo.appendChild(erroEl);
  }

  // Limpar erro personalizado anterior antes de validar
  campo.setCustomValidity('');

  if (campo.validity.valid) {
    grupo.classList.remove('invalido');
    grupo.classList.add('valido');
    erroEl.textContent = '';
    return true;
  }

  // Mensagens personalizadas conforme o tipo de erro
  let mensagem = '';

  if (campo.validity.valueMissing) {
    mensagem = 'Este campo é obrigatório.';
  } else if (campo.validity.typeMismatch) {
    if (campo.type === 'email') mensagem = 'Informe um e-mail válido.';
    else if (campo.type === 'url') mensagem = 'Informe uma URL válida (ex: https://...).';
    else mensagem = 'Formato inválido.';
  } else if (campo.validity.tooShort) {
    mensagem = `Mínimo de ${campo.minLength} caracteres (atual: ${campo.value.length}).`;
  } else if (campo.validity.tooLong) {
    mensagem = `Máximo de ${campo.maxLength} caracteres.`;
  } else if (campo.validity.patternMismatch) {
    mensagem = campo.title || 'Formato inválido.';
  } else if (campo.validity.rangeUnderflow) {
    mensagem = `O valor mínimo é ${campo.min}.`;
  } else if (campo.validity.rangeOverflow) {
    mensagem = `O valor máximo é ${campo.max}.`;
  }

  grupo.classList.add('invalido');
  grupo.classList.remove('valido');
  erroEl.textContent = mensagem;
  return false;
}

// Validar cada campo ao perder o foco (delegação com focusout)
form.addEventListener('focusout', (e) => {
  if (e.target.matches('input, textarea, select')) {
    validarCampo(e.target);
  }
});

// Revalidar em tempo real após o primeiro erro (limpa o erro assim que corrige)
form.addEventListener('input', (e) => {
  const campo = e.target;
  const grupo = campo.closest('label') || campo.parentElement;
  if (grupo.classList.contains('invalido')) {
    validarCampo(campo);
  }
});
```

**Validação ao enviar o formulário:**

```js
form.addEventListener('submit', (e) => {
  e.preventDefault();

  const campos = form.querySelectorAll('input, textarea, select');
  let formValido = true;

  campos.forEach((campo) => {
    if (!validarCampo(campo)) {
      formValido = false;
    }
  });

  if (!formValido) {
    // Foco no primeiro campo com erro
    form.querySelector('.invalido input, .invalido textarea, .invalido select')?.focus();
    return;
  }

  // Formulário válido — enviar dados
  const dados = new FormData(form);
  console.log('Enviando:', Object.fromEntries(dados));
});
```

**Validação personalizada com `setCustomValidity`:**

```js
// Confirmar senha — validação que não existe nativamente
const senha = form.querySelector('[name="senha"]');
const confirmar = form.querySelector('[name="confirmar-senha"]');

confirmar?.addEventListener('input', () => {
  if (confirmar.value !== senha.value) {
    confirmar.setCustomValidity('As senhas não coincidem.');
  } else {
    confirmar.setCustomValidity(''); // limpa o erro
  }
});

// Validação assíncrona — verificar e-mail já cadastrado
const emailCampo = form.querySelector('[name="email"]');

emailCampo.addEventListener('blur', async () => {
  if (!emailCampo.validity.valid) return; // já tem erro nativo

  const resp = await fetch(`/api/verificar-email?email=${encodeURIComponent(emailCampo.value)}`);
  const { existe } = await resp.json();

  if (existe) {
    emailCampo.setCustomValidity('Este e-mail já está cadastrado.');
    validarCampo(emailCampo);
  } else {
    emailCampo.setCustomValidity('');
    validarCampo(emailCampo);
  }
});
```

**CSS para feedback visual:**

```css
/* Estilização dos estados de validação */
.campo-grupo.invalido input,
label.invalido input {
  border-color: #dc2626;
  outline-color: #dc2626;
}

.campo-grupo.valido input,
label.valido input {
  border-color: #16a34a;
}

.erro {
  color: #dc2626;
  font-size: 0.85em;
  display: block;
  margin-top: 4px;
  min-height: 1.2em; /* evita layout shift */
}

/* Alternativa: usar pseudo-classes nativas (sem novalidate) */
input:invalid:not(:placeholder-shown) {
  border-color: #dc2626;
}

input:valid:not(:placeholder-shown) {
  border-color: #16a34a;
}
```

> **Dica:** a pseudo-classe `:not(:placeholder-shown)` evita que campos vazios apareçam como inválidos antes do usuário interagir. Campos que usam `placeholder` só mostram o estilo de erro após o usuário começar a digitar.

### FormData

> MDN: [FormData](https://developer.mozilla.org/pt-BR/docs/Web/API/FormData)

`FormData` representa um conjunto de pares chave/valor que reproduz o comportamento de um formulário HTML — incluindo arquivos. É a forma mais simples de coletar dados de um `<form>` e enviá-los via `fetch`.

```js
// Criar a partir de um elemento <form> existente
const form = document.querySelector('form');
const dados = new FormData(form); // captura todos os campos com atributo name

// Criar manualmente
const dados = new FormData();
dados.append('nome', 'Ana');
dados.append('idade', '30');
dados.append('arquivo', inputFile.files[0]); // adiciona um arquivo

// Leitura
dados.get('nome');              // 'Ana' — primeiro valor da chave
dados.getAll('interesses');     // array — para campos com múltiplos valores (checkbox)
dados.has('nome');              // true
dados.keys();                   // iterável com as chaves
dados.values();                 // iterável com os valores
dados.entries();                // iterável com pares [chave, valor]

// Escrita
dados.set('nome', 'Bia');       // substitui todos os valores da chave
dados.append('tag', 'js');      // adiciona sem remover valores anteriores
dados.delete('idade');

// Iterar
for (const [chave, valor] of dados) {
  console.log(chave, valor);
}
```

**Converter FormData em objeto JSON:**

```js
// Caso simples — uma chave com um único valor
const obj = Object.fromEntries(dados.entries());
// { nome: 'Ana', email: 'ana@email.com' }

// Caso com múltiplos valores na mesma chave (ex: checkboxes com mesmo name)
// Object.fromEntries() mantém apenas o último valor — usar a função abaixo:
function formDataParaObj(formData) {
  const obj = {};
  for (const [chave, valor] of formData) {
    if (chave in obj) {
      // converte para array ou adiciona ao array existente
      obj[chave] = [].concat(obj[chave], valor);
    } else {
      obj[chave] = valor;
    }
  }
  return obj;
}

// Converter para JSON string (para enviar no body de uma requisição)
const json = JSON.stringify(Object.fromEntries(dados.entries()));
```

**Envio via fetch:**

```js
form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const dados = new FormData(form);

  // Enviar como multipart/form-data (obrigatório quando há arquivos)
  // Não definir Content-Type — o browser define com o boundary correto
  await fetch('/api/enviar', { method: 'POST', body: dados });

  // Enviar como JSON (somente quando não há arquivos)
  await fetch('/api/enviar', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(Object.fromEntries(dados.entries())),
  });
});
```

---

### URLSearchParams

> MDN: [URLSearchParams](https://developer.mozilla.org/pt-BR/docs/Web/API/URLSearchParams)

`URLSearchParams` lida com a query string de uma URL — o trecho após `?` — de forma segura, sem concatenação manual de strings e com encoding automático.

```js
// Criar a partir de uma string de query
const params = new URLSearchParams('pagina=2&ordem=nome&ativo=true');

// Criar a partir de um objeto
const params = new URLSearchParams({ pagina: '2', ordem: 'nome' });

// Criar a partir de pares
const params = new URLSearchParams([['pagina', '2'], ['tag', 'js']]);

// Leitura
params.get('pagina');           // '2' — sempre retorna string, ou null
params.getAll('tag');           // ['js', 'css'] — chaves repetidas
params.has('ordem');            // true

// Escrita
params.set('pagina', '3');      // substitui
params.append('tag', 'css');    // adiciona sem remover outros valores
params.delete('ativo');

// Serializar de volta para string
params.toString();              // 'pagina=3&ordem=nome&tag=js&tag=css'

// Iterar
for (const [chave, valor] of params) {
  console.log(chave, valor);
}
```

**Ler a query string da URL atual:**

```js
// URL: https://exemplo.com/busca?q=notebook&pagina=2&ordem=preco
const params = new URLSearchParams(window.location.search);

params.get('q');      // 'notebook'
params.get('pagina'); // '2'
params.get('ordem');  // 'preco'
```

**Construir URLs com parâmetros:**

```js
const base = 'https://api.exemplo.com/produtos';
const params = new URLSearchParams({
  categoria: 'eletronicos',
  pagina: '1',
  limite: '20',
  q: 'notebook gamer', // espaços e caracteres especiais são codificados automaticamente
});

const url = `${base}?${params}`;
// 'https://api.exemplo.com/produtos?categoria=eletronicos&pagina=1&limite=20&q=notebook+gamer'

await fetch(url);
```

**Atualizar a URL sem recarregar a página:**

```js
const params = new URLSearchParams(window.location.search);
params.set('pagina', '3');

// Atualiza a URL na barra de endereço sem navegar
history.pushState({}, '', `?${params}`);
// ou para substituir sem criar entrada no histórico:
history.replaceState({}, '', `?${params}`);
```

**FormData → URLSearchParams (quando não há arquivos):**

```js
const form = document.querySelector('form');
const formData = new FormData(form);
const params = new URLSearchParams(formData);

// Útil para requisições GET com dados do formulário
window.location.href = `/busca?${params}`;
```

---

### Helpers utilitários para DOM (vanilla)

Pequenas funções que encapsulam APIs nativas verbosas, reduzindo boilerplate em projetos sem framework. Nenhuma depende de bibliotecas externas.

**Seleção**

```js
// $ / $$ — querySelector com retorno tipado
const $  = (sel, ctx = document) => ctx.querySelector(sel);
const $$ = (sel, ctx = document) => [...ctx.querySelectorAll(sel)];

$('.card');             // primeiro elemento ou null
$$('.card');            // array de elementos (não NodeList)

// closest já existe nativamente — acessa o ancestral mais próximo pelo seletor
btn.closest('.modal');  // sobe a árvore até encontrar .modal
```

**Criação e manipulação**

```js
// createElement — cria elemento com atributos e filhos em uma única chamada
function el(tag, attrs = {}, ...filhos) {
  const node = document.createElement(tag);
  for (const [chave, valor] of Object.entries(attrs)) {
    if (chave === 'class') node.className = valor;
    else if (chave.startsWith('on') && typeof valor === 'function')
      node.addEventListener(chave.slice(2).toLowerCase(), valor);
    else node.setAttribute(chave, valor);
  }
  node.append(...filhos.flat());
  return node;
}

const card = el('div', { class: 'card' },
  el('h2', {}, 'Título'),
  el('button', { onClick: () => alert('clicou') }, 'OK'),
);
document.body.append(card);

// html — template literal que retorna um DocumentFragment pronto para inserção
function html(strings, ...valores) {
  const template = document.createElement('template');
  template.innerHTML = strings.reduce((acc, str, i) => acc + valores[i - 1] + str);
  return template.content;
}

const frag = html`<li class="item">${nome}</li>`;
lista.append(frag);
```

> **Atenção:** `html` interpreta a string como HTML — assim como `innerHTML`, é seguro para conteúdo estático, mas exige sanitização se algum valor interpolado vier de input do usuário (risco de XSS). Quando o conteúdo é dinâmico/não confiável, preferir `el()`, que monta nós programaticamente e nunca interpreta HTML.

**Classes e estilos**

```js
// cls — manipulação declarativa de classes (semelhante a clsx/classnames)
function cls(el, mapa) {
  for (const [classe, condicao] of Object.entries(mapa)) {
    el.classList.toggle(classe, !!condicao);
  }
}
cls(btn, { ativo: estaAtivo, desabilitado: !podeClicar });

// setStyle — aplica múltiplas propriedades de estilo de uma vez
function setStyle(el, props) {
  Object.assign(el.style, props);
}
setStyle(box, { display: 'flex', gap: '8px', backgroundColor: '#2563eb' });
```

**Eventos**

```js
// on / off — wrapper com retorno de função de cleanup
function on(el, evento, handler, opts) {
  el.addEventListener(evento, handler, opts);
  return () => el.removeEventListener(evento, handler, opts);
}

const cleanup = on(window, 'resize', () => console.log('resize'));
cleanup(); // remove o listener quando não for mais necessário

// once — disparado uma única vez (atalho para addEventListener com { once: true })
function once(el, evento, handler) {
  return on(el, evento, handler, { once: true });
}

// delegate — event delegation: um único listener no pai captura eventos dos filhos
function delegate(parent, evento, seletor, handler) {
  return on(parent, evento, (e) => {
    const target = e.target.closest(seletor);
    if (target && parent.contains(target)) handler(e, target);
  });
}

// Funciona mesmo para itens adicionados dinamicamente depois do listener ser criado
delegate(document.querySelector('ul'), 'click', 'li', (e, item) => {
  console.log('Item clicado:', item.textContent);
});
```

> `on` com retorno de cleanup é o padrão mais reutilizável da lista: encaixa-se diretamente em `connectedCallback`/`disconnectedCallback` de Web Components, em `mount`/`unmount` de rotas de uma SPA vanilla, e é equivalente à função de cleanup retornada por um `useEffect` no React.
>
> `delegate` evita reatachar listeners a cada item de uma lista dinâmica — é o mesmo princípio usado internamente pelo React, que registra um único listener no root e despacha eventos via delegation.

**Ciclo de vida e timing**

```js
// ready — executa o callback quando o DOM estiver pronto (já parseado)
function ready(fn) {
  if (document.readyState !== 'loading') fn();
  else document.addEventListener('DOMContentLoaded', fn);
}
ready(() => console.log('DOM pronto'));

// onVisible — lazy execution via IntersectionObserver (ex.: lazy load, animações)
function onVisible(el, callback, opts = {}) {
  const observer = new IntersectionObserver((entradas) => {
    for (const entrada of entradas) {
      if (entrada.isIntersecting) {
        callback(entrada);
        observer.unobserve(el); // dispara uma vez e desconecta
      }
    }
  }, opts);
  observer.observe(el);
  return () => observer.disconnect();
}

onVisible($('.imagem-lazy'), (entrada) => {
  entrada.target.src = entrada.target.dataset.src;
});

// onMutate — observa mudanças no DOM via MutationObserver
function onMutate(el, callback, opts = { childList: true, subtree: true }) {
  const observer = new MutationObserver(callback);
  observer.observe(el, opts);
  return () => observer.disconnect();
}

const pararObservacao = onMutate(document.querySelector('#lista'), (mutacoes) => {
  console.log(`${mutacoes.length} mudança(s) detectada(s)`);
});
```

> `onVisible` e `onMutate` são os helpers que mais se justificam: as APIs nativas (`IntersectionObserver`, `MutationObserver`) têm interface verbosa (criar, `observe`, lembrar de `disconnect`/`unobserve`) e é fácil esquecer a limpeza, causando memory leaks.

**Scroll e geometria**

```js
// scrollTo / intoView — scroll suave declarativo
function scrollTo(el, opts = {}) {
  el.scrollIntoView({ behavior: 'smooth', block: 'start', ...opts });
}
const intoView = scrollTo; // alias semântico

scrollTo($('#secao-3'));

// rect — getBoundingClientRect simplificado (atalho para leituras comuns)
function rect(el) {
  const r = el.getBoundingClientRect();
  return { top: r.top, left: r.left, width: r.width, height: r.height, bottom: r.bottom, right: r.right };
}

rect($('.card')).width; // largura renderizada em px
```

### DocumentFragment — buffer de elementos antes de inserir no DOM

> MDN: [DocumentFragment](https://developer.mozilla.org/pt-BR/docs/Web/API/DocumentFragment)

Um `DocumentFragment` é um contêiner leve que existe apenas em memória — não faz parte da árvore do DOM. Ele permite montar vários elementos de uma vez e inseri-los com uma única operação, evitando múltiplos reflows e repaints.

**Por que usar?** Cada inserção direta no DOM (`append`, `appendChild`) pode disparar recálculo de layout. Quando se criam muitos elementos em um loop, o custo acumula. Com `DocumentFragment`, todas as inserções acontecem em memória e o DOM real é atualizado uma única vez.

```js
// ❌ Sem fragment — cada appendChild dispara reflow
const lista = document.querySelector('ul');
for (const item of dados) {
  const li = document.createElement('li');
  li.textContent = item.nome;
  lista.appendChild(li); // reflow a cada iteração
}

// ✅ Com fragment — um único reflow ao final
const lista = document.querySelector('ul');
const fragment = document.createDocumentFragment();

for (const item of dados) {
  const li = document.createElement('li');
  li.textContent = item.nome;
  fragment.appendChild(li); // adiciona ao fragment (em memória)
}

lista.appendChild(fragment); // insere tudo de uma vez no DOM
```

> Quando o `DocumentFragment` é inserido no DOM, ele "esvazia" — seus filhos são movidos para o elemento de destino e o fragment fica vazio. Ele não cria um nó wrapper extra na árvore.

**Exemplo prático — tabela gerada dinamicamente:**

```js
function renderizarTabela(produtos) {
  const tbody = document.querySelector('table tbody');
  const fragment = document.createDocumentFragment();

  for (const prod of produtos) {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${prod.nome}</td>
      <td>R$ ${prod.preco.toFixed(2)}</td>
      <td>${prod.estoque}</td>
    `;
    fragment.appendChild(tr);
  }

  tbody.innerHTML = ''; // limpa conteúdo anterior
  tbody.appendChild(fragment);
}

renderizarTabela([
  { nome: 'Notebook', preco: 3500, estoque: 12 },
  { nome: 'Mouse', preco: 89.90, estoque: 45 },
  { nome: 'Teclado', preco: 199, estoque: 30 },
]);
```

**Alternativa com `<template>`:**

O elemento HTML `<template>` também cria um `DocumentFragment` (via `.content`) e é útil quando o modelo do elemento já está definido no HTML.

```html
<template id="card-template">
  <div class="card">
    <h3 class="card-titulo"></h3>
    <p class="card-descricao"></p>
  </div>
</template>
```

```js
function criarCards(itens) {
  const template = document.querySelector('#card-template');
  const container = document.querySelector('.cards');
  const fragment = document.createDocumentFragment();

  for (const item of itens) {
    const clone = template.content.cloneNode(true); // clona o fragment do template
    clone.querySelector('.card-titulo').textContent = item.titulo;
    clone.querySelector('.card-descricao').textContent = item.descricao;
    fragment.appendChild(clone);
  }

  container.appendChild(fragment);
}
```

> `template.content` é um `DocumentFragment` — seu conteúdo não é renderizado até ser clonado e inserido no DOM. `cloneNode(true)` cria uma cópia profunda (incluindo filhos).

### Event Delegation com closest() e AbortController

A **delegação de eventos** consiste em registrar um único listener no elemento pai para capturar eventos de todos os filhos — inclusive os criados dinamicamente depois. Combinada com `closest()` para identificar o alvo e `AbortController` para gerenciar a remoção, é o padrão mais robusto para lidar com eventos em listas e conteúdo dinâmico.

#### Por que delegar?

```js
// ❌ Listener individual em cada botão — frágil com conteúdo dinâmico
document.querySelectorAll('.btn-excluir').forEach(btn => {
  btn.addEventListener('click', () => excluirItem(btn));
});
// Problema: botões adicionados depois NÃO recebem o listener

// ✅ Delegação — funciona para elementos atuais e futuros
document.querySelector('.lista').addEventListener('click', (e) => {
  const btn = e.target.closest('.btn-excluir');
  if (btn) excluirItem(btn);
});
```

#### closest() — encontrar o ancestral correto

> MDN: [Element.closest()](https://developer.mozilla.org/pt-BR/docs/Web/API/Element/closest)

`closest(seletor)` sobe a árvore do DOM a partir do elemento (incluindo ele mesmo) e retorna o primeiro ancestral que bate com o seletor, ou `null`.

```js
// Estrutura: <ul class="lista"> → <li> → <span> → <button class="btn-excluir">🗑</button>
// Se o clique for no ícone dentro do <button>, e.target é o ícone, não o <button>
// closest() resolve isso subindo até encontrar o seletor correto

document.querySelector('.lista').addEventListener('click', (e) => {
  // Subir até encontrar o .btn-excluir mais próximo
  const btn = e.target.closest('.btn-excluir');
  if (!btn) return; // clique fora de qualquer botão

  // Subir mais para encontrar o <li> ao qual o botão pertence
  const item = btn.closest('li');
  console.log('Excluir item:', item.dataset.id);
});
```

#### AbortController — gerenciar remoção de listeners

> MDN: [AbortController](https://developer.mozilla.org/pt-BR/docs/Web/API/AbortController)

`AbortController` permite remover múltiplos listeners de uma vez usando um único `signal`, sem precisar guardar referência de cada função de callback.

```js
const controller = new AbortController();

// Todos os listeners abaixo compartilham o mesmo signal
document.addEventListener('click', handleClick, { signal: controller.signal });
document.addEventListener('keydown', handleKey, { signal: controller.signal });
window.addEventListener('resize', handleResize, { signal: controller.signal });

// Uma única chamada remove todos os listeners associados ao signal
controller.abort();
```

**Comparação com a abordagem tradicional:**

```js
// ❌ Sem AbortController — precisa guardar referência de cada handler
const handleClick = (e) => { /* ... */ };
const handleKey = (e) => { /* ... */ };
document.addEventListener('click', handleClick);
document.addEventListener('keydown', handleKey);
// Para remover: precisa de cada referência
document.removeEventListener('click', handleClick);
document.removeEventListener('keydown', handleKey);

// ✅ Com AbortController — remoção centralizada
const controller = new AbortController();
document.addEventListener('click', (e) => { /* ... */ }, { signal: controller.signal });
document.addEventListener('keydown', (e) => { /* ... */ }, { signal: controller.signal });
// Uma chamada remove tudo (funciona mesmo com arrow functions anônimas)
controller.abort();
```

#### Exemplo completo — lista dinâmica com delegação e AbortController

```html
<div id="app">
  <input type="text" id="novo-item" placeholder="Novo item..." />
  <button id="btn-adicionar">Adicionar</button>
  <ul id="lista"></ul>
</div>
```

```js
function iniciarListaDinamica() {
  const controller = new AbortController();
  const { signal } = controller;

  const input = document.querySelector('#novo-item');
  const lista = document.querySelector('#lista');
  let contador = 0;

  // Adicionar item (cria elemento dinamicamente)
  document.querySelector('#btn-adicionar').addEventListener('click', () => {
    const texto = input.value.trim();
    if (!texto) return;

    const li = document.createElement('li');
    li.dataset.id = ++contador;
    li.innerHTML = `
      <span class="texto">${texto}</span>
      <button class="btn-editar">Editar</button>
      <button class="btn-excluir">Excluir</button>
    `;
    lista.appendChild(li);
    input.value = '';
  }, { signal });

  // Delegação — um único listener captura cliques em todos os botões
  lista.addEventListener('click', (e) => {
    const btnExcluir = e.target.closest('.btn-excluir');
    if (btnExcluir) {
      const li = btnExcluir.closest('li');
      li.remove();
      return;
    }

    const btnEditar = e.target.closest('.btn-editar');
    if (btnEditar) {
      const li = btnEditar.closest('li');
      const span = li.querySelector('.texto');
      const novoTexto = prompt('Editar item:', span.textContent);
      if (novoTexto !== null) span.textContent = novoTexto;
    }
  }, { signal });

  // Enter no input adiciona o item
  input.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') {
      document.querySelector('#btn-adicionar').click();
    }
  }, { signal });

  // Retorna função de cleanup — remove todos os listeners de uma vez
  return () => controller.abort();
}

// Iniciar e guardar a função de cleanup
const destruir = iniciarListaDinamica();

// Quando não precisar mais (ex: navegar para outra "página" em uma SPA)
// destruir();
```

**Por que o AbortController é importante aqui:**
- Em uma SPA (Single Page Application), ao navegar para outra tela, os listeners precisam ser removidos para evitar **memory leaks** e comportamentos duplicados.
- Sem `AbortController`, seria necessário guardar referência de cada handler e removê-los individualmente.
- Com `AbortController`, basta chamar `controller.abort()` e todos os listeners associados ao `signal` são removidos automaticamente.

---

## 4. JSON

> MDN: [JSON](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/JSON)

**JSON** (JavaScript Object Notation) é o formato padrão para troca de dados entre cliente e servidor.

### JSON.stringify — objeto para string

```js
const produto = {
  nome: 'Notebook',
  preco: 3500.00,
  disponivel: true,
  tags: ['eletronico', 'informatica'],
  fabricante: null,
  calcularDesconto() { return 0; }  // funções são ignoradas
};

// Serialização básica
JSON.stringify(produto);
// '{"nome":"Notebook","preco":3500,"disponivel":true,"tags":["eletronico","informatica"],"fabricante":null}'

// Com indentação (legível)
JSON.stringify(produto, null, 2);

// Com replacer — filtra ou transforma propriedades
JSON.stringify(produto, ['nome', 'preco']); // apenas nome e preco
JSON.stringify(produto, (chave, valor) => {
  if (typeof valor === 'number') return valor * 100; // centavos
  return valor;
});
```

### JSON.parse — string para objeto

```js
const json = '{"nome":"Notebook","preco":3500}';

const obj = JSON.parse(json);
obj.nome;  // "Notebook"

// Com reviver — transforma valores ao parsear
const dados = '{"createdAt":"2025-03-20T10:00:00.000Z"}';
const parsed = JSON.parse(dados, (chave, valor) => {
  if (chave === 'createdAt') return new Date(valor);
  return valor;
});
parsed.createdAt instanceof Date; // true

// Tratamento de erro (JSON inválido lança SyntaxError)
try {
  JSON.parse('texto inválido');
} catch (e) {
  console.error('JSON inválido:', e.message);
}
```

### Boas práticas

```js
// Cópia profunda simples via JSON (limitações: perde Date, undefined, funções)
const clone = JSON.parse(JSON.stringify(objeto));

// Para cópia profunda completa, preferir structuredClone (ES2022)
const clone = structuredClone(objeto);
```

---

## 5. Promise e async/await

### Operações assíncronas

O JavaScript é **single-threaded** — executa uma instrução por vez, em uma única fila. Operações que dependem de tempo (requisições de rede, leitura de arquivos, timers) não podem bloquear essa fila, pois a página inteira travaria enquanto aguardava a resposta.

A solução é o modelo **assíncrono**: ao iniciar uma operação demorada, o JS registra o que deve acontecer quando ela terminar (um callback) e segue executando o restante do código. Quando a operação conclui, o resultado volta para a fila de execução e o callback é chamado.

```js
console.log('1 — antes');

setTimeout(() => {
  console.log('3 — dentro do timeout (assíncrono)');
}, 0); // mesmo com 0ms, executa depois do código síncrono

console.log('2 — depois');

// Saída:
// 1 — antes
// 2 — depois
// 3 — dentro do timeout (assíncrono)
```

Esse mecanismo é coordenado pelo **Event Loop**: enquanto a call stack (pilha de execução) está vazia, ele pega a próxima tarefa pendente da fila e a executa.

Operações comuns que são assíncronas em JS:
- Requisições HTTP (`fetch`, `XMLHttpRequest`)
- Leitura e escrita de arquivos (Node.js)
- Timers (`setTimeout`, `setInterval`)
- Eventos do usuário (`click`, `input`, etc.)
- Acesso a banco de dados

### Promise

> MDN: [Promise](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise)

Uma **Promise** representa o resultado eventual (ou a falha) de uma operação assíncrona.

```js
// Criando uma Promise
const promise = new Promise((resolve, reject) => {
  const sucesso = true;
  if (sucesso) {
    resolve({ dados: 'resultado' });  // operação bem-sucedida
  } else {
    reject(new Error('Algo deu errado')); // operação falhou
  }
});

// Consumindo com .then/.catch/.finally
promise
  .then(resultado => console.log(resultado))
  .catch(erro => console.error(erro))
  .finally(() => console.log('Sempre executa'));
```

**Estados de uma Promise:**
- `pending` — aguardando resultado
- `fulfilled` — resolvida com sucesso
- `rejected` — rejeitada com erro

### async / await

Sintaxe mais legível para trabalhar com Promises.

```js
// async transforma a função em uma que retorna Promise
async function buscarUsuario(id) {
  try {
    const resposta = await fetch(`/api/usuarios/${id}`);

    if (!resposta.ok) {
      throw new Error(`Erro HTTP: ${resposta.status}`);
    }

    const usuario = await resposta.json();
    return usuario;

  } catch (erro) {
    console.error('Falha ao buscar usuário:', erro);
    throw erro;  // re-lança para quem chamou tratar
  }
}

// Top-level await (em módulos ES)
const usuario = await buscarUsuario(1);
```

### Métodos estáticos de Promise

```js
const p1 = fetch('/api/usuarios');
const p2 = fetch('/api/produtos');
const p3 = fetch('/api/pedidos');

// Aguarda todas resolverem — falha se qualquer uma rejeitar
const [usuarios, produtos, pedidos] = await Promise.all([p1, p2, p3]);

// Aguarda todas terminarem, independente de sucesso ou falha
const resultados = await Promise.allSettled([p1, p2, p3]);
resultados.forEach(({ status, value, reason }) => {
  if (status === 'fulfilled') console.log(value);
  else console.error(reason);
});

// Retorna a primeira que resolver ou rejeitar
const primeira = await Promise.race([p1, p2, p3]);

// Retorna a primeira que resolver com sucesso
const primeiraOk = await Promise.any([p1, p2, p3]);

// Promise já resolvida ou rejeitada
Promise.resolve(42).then(v => console.log(v)); // 42
Promise.reject(new Error('falha')).catch(e => console.error(e));
```

## 6. Fetch API

> MDN: [Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API)

```js
// GET básico
const resposta = await fetch('/api/produtos');
const produtos = await resposta.json();

// Verificar status
resposta.ok;          // true se status 200-299
resposta.status;      // 200, 404, 500...
resposta.statusText;  // "OK", "Not Found"...
resposta.headers.get('Content-Type');
```

**POST com JSON:**
```js
async function criarProduto(produto) {
  const resposta = await fetch('/api/produtos', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
    body: JSON.stringify(produto),
  });

  if (!resposta.ok) {
    const erro = await resposta.json();
    throw new Error(erro.mensagem ?? 'Erro ao criar produto');
  }

  return resposta.json();
}
```

**PUT, PATCH, DELETE:**
```js
// PUT — substituição completa
await fetch(`/api/produtos/${id}`, {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(dadosCompletos),
});

// PATCH — atualização parcial
await fetch(`/api/produtos/${id}`, {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ preco: 2999 }),
});

// DELETE
await fetch(`/api/produtos/${id}`, { method: 'DELETE' });
```

**Upload de arquivo:**
```js
const form = document.querySelector('form');
const dados = new FormData(form);  // inclui arquivos automaticamente

await fetch('/api/upload', {
  method: 'POST',
  body: dados,  // não definir Content-Type — o browser define com boundary
});
```

**Abort (cancelar requisição):**
```js
const controller = new AbortController();

// Cancela após 5 segundos
setTimeout(() => controller.abort(), 5000);

try {
  const resposta = await fetch('/api/dados', {
    signal: controller.signal,
  });
} catch (e) {
  if (e.name === 'AbortError') console.log('Requisição cancelada');
}
```

**Wrapper reutilizável:**
```js
async function api(url, opcoes = {}) {
  const config = {
    headers: {
      'Content-Type': 'application/json',
      ...opcoes.headers,
    },
    ...opcoes,
  };

  if (opcoes.body) {
    config.body = JSON.stringify(opcoes.body);
  }

  const resposta = await fetch(url, config);

  if (!resposta.ok) {
    const erro = await resposta.json().catch(() => ({}));
    throw new Error(erro.mensagem ?? `Erro ${resposta.status}`);
  }

  // 204 No Content — sem corpo
  if (resposta.status === 204) return null;

  return resposta.json();
}

// Uso
const usuario = await api('/api/usuarios/1');
const novo = await api('/api/usuarios', { method: 'POST', body: { nome: 'Ana' } });
```

---

## 7. TypeScript

> Documentação: [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

**TypeScript** é um superset do JavaScript que adiciona tipagem estática opcional. É compilado para JavaScript e detecta erros antes da execução.

### Tipos básicos

```ts
// Primitivos
let nome: string = 'Ana';
let idade: number = 30;
let ativo: boolean = true;

// Arrays
let numeros: number[] = [1, 2, 3];
let nomes: Array<string> = ['Ana', 'Bob'];

// Tupla — array com tipos e tamanho fixos
let coordenada: [number, number] = [10, 20];
let entrada: [string, number] = ['Ana', 30];

// any — desativa a verificação de tipo (evitar)
let qualquer: any = 'texto';
qualquer = 42;  // sem erro

// unknown — tipo seguro para valores desconhecidos
let desconhecido: unknown = obterDados();
if (typeof desconhecido === 'string') {
  desconhecido.toUpperCase();  // só funciona após verificação
}

// never — função que nunca retorna
function lancarErro(msg: string): never {
  throw new Error(msg);
}

// void — função sem retorno significativo
function logar(msg: string): void {
  console.log(msg);
}

// null e undefined
let vazio: null = null;
let indefinido: undefined = undefined;
```

### Tipos de união e interseção

```ts
// União (|) — pode ser um tipo OU outro
let id: number | string;
id = 1;
id = 'abc';

function formatar(valor: number | string): string {
  if (typeof valor === 'number') return valor.toFixed(2);
  return valor.trim();
}

// Interseção (&) — combina múltiplos tipos
type Admin = { nome: string } & { permissao: string };
const admin: Admin = { nome: 'Ana', permissao: 'total' };
```

### Tipos literais

```ts
// Apenas os valores especificados são aceitos
type Direcao = 'norte' | 'sul' | 'leste' | 'oeste';
type StatusHttp = 200 | 201 | 400 | 401 | 404 | 500;

let dir: Direcao = 'norte';  // ok
dir = 'cima';                 // erro!

function mover(dir: Direcao) { /* ... */ }
```

### Type Aliases

```ts
// Apelido para um tipo
type ID = number | string;
type Coordenada = [number, number];
type Callback = (erro: Error | null, resultado?: string) => void;

// Objeto com type
type Produto = {
  id: number;
  nome: string;
  preco: number;
  descricao?: string;  // propriedade opcional
  readonly codigo: string;  // não pode ser alterado após criação
};
```

### Interfaces

```ts
// Interface — define a forma de um objeto
interface Usuario {
  id: number;
  nome: string;
  email: string;
  telefone?: string;        // opcional
  readonly createdAt: Date; // somente leitura
}

// Interfaces podem ser estendidas
interface Admin extends Usuario {
  permissoes: string[];
  nivel: number;
}

// Interfaces podem ser declaradas múltiplas vezes (declaration merging)
interface Usuario {
  apelido?: string;  // adicionado à interface original
}

// Interface para função
interface Comparador {
  (a: number, b: number): number;
}

// Interface com índice dinâmico
interface Mapa {
  [chave: string]: number;
}
const contagem: Mapa = { maçã: 3, banana: 5 };
```

### Type vs Interface

| Aspecto | `type` | `interface` |
|---|---|---|
| Objeto | Sim | Sim |
| União e interseção | Sim | Apenas interseção (extends) |
| Declaration merging | Não | Sim |
| Tipos primitivos/tuplas | Sim | Não |
| Recomendado para | Tipos utilitários, uniões | Contratos de objetos e classes |

> **Convenção:** usar `interface` para definir contratos de objetos e classes; usar `type` para uniões, interseções e aliases de tipos complexos.

### Enums

```ts
// Enum numérico (começa em 0 por padrão)
enum Direcao {
  Norte,    // 0
  Sul,      // 1
  Leste,    // 2
  Oeste,    // 3
}

// Enum numérico com valor inicial
enum StatusPedido {
  Pendente = 1,
  Processando,  // 2
  Enviado,      // 3
  Entregue,     // 4
}

// Enum de string (mais legível em debug e serialização)
enum Status {
  Ativo = 'ATIVO',
  Inativo = 'INATIVO',
  Suspenso = 'SUSPENSO',
}

const s: Status = Status.Ativo;
console.log(s); // "ATIVO"

// Const enum — substituído em tempo de compilação (sem código gerado)
const enum Tamanho {
  P = 'P',
  M = 'M',
  G = 'G',
}
```

### Generics

Permitem criar funções, classes e interfaces reutilizáveis para múltiplos tipos.

```ts
// Função genérica
function primeiro<T>(array: T[]): T | undefined {
  return array[0];
}

primeiro([1, 2, 3]);          // retorna number
primeiro(['a', 'b']);         // retorna string
primeiro<boolean>([true]);    // tipo explícito

// Com constraint — T deve ter a propriedade length
function tamanho<T extends { length: number }>(valor: T): number {
  return valor.length;
}
tamanho('texto');   // 5
tamanho([1, 2, 3]); // 3

// Interface genérica
interface Resposta<T> {
  dados: T;
  total?: number;
  pagina?: number;
}

async function buscar<T>(url: string): Promise<Resposta<T>> {
  const resp = await fetch(url);
  return resp.json();
}

const resultado = await buscar<Usuario[]>('/api/usuarios');
resultado.dados; // Usuario[]
```

### Modificadores de acesso em classes

```ts
class ContaBancaria {
  public titular: string;        // acessível em qualquer lugar (padrão)
  protected saldo: number;       // acessível na classe e nas subclasses
  private senha: string;         // acessível apenas na classe
  readonly agencia: string;      // somente leitura

  // Atalho: declarar e inicializar propriedades no constructor
  constructor(
    public titular: string,
    protected saldo: number,
    private senha: string,
    readonly agencia: string,
  ) {}

  depositar(valor: number): void {
    this.saldo += valor;
  }

  get saldoAtual(): number {
    return this.saldo;
  }
}

class ContaEspecial extends ContaBancaria {
  sacar(valor: number): void {
    this.saldo -= valor;  // ok — protected é acessível em subclasse
    // this.senha          // erro — private não é acessível em subclasse
  }
}
```

### Tipos utilitários (Utility Types)

```ts
interface Produto {
  id: number;
  nome: string;
  preco: number;
  descricao: string;
}

// Partial — todas as propriedades ficam opcionais
type ProdutoAtualizar = Partial<Produto>;
// { id?: number; nome?: string; preco?: number; descricao?: string }

// Required — todas as propriedades ficam obrigatórias
type ProdutoCompleto = Required<Partial<Produto>>;

// Readonly — todas as propriedades ficam somente leitura
type ProdutoImutavel = Readonly<Produto>;

// Pick — seleciona propriedades específicas
type ProdutoResumo = Pick<Produto, 'id' | 'nome'>;
// { id: number; nome: string }

// Omit — exclui propriedades específicas
type ProdutoSemId = Omit<Produto, 'id'>;
// { nome: string; preco: number; descricao: string }

// Record — cria tipo de mapa
type EstoquePorProduto = Record<string, number>;
// { [chave: string]: number }

// ReturnType — obtém o tipo de retorno de uma função
function getUsuario() { return { id: 1, nome: 'Ana' }; }
type TipoUsuario = ReturnType<typeof getUsuario>;
// { id: number; nome: string }
```

### Type Assertions e Type Guards

```ts
// Type assertion — força um tipo (usar com cuidado)
const input = document.querySelector('#campo') as HTMLInputElement;
input.value;

// Type guard com typeof
function processar(valor: string | number) {
  if (typeof valor === 'string') {
    return valor.toUpperCase();  // TypeScript sabe que é string aqui
  }
  return valor.toFixed(2);       // TypeScript sabe que é number aqui
}

// Type guard com instanceof
function tratar(erro: unknown) {
  if (erro instanceof Error) {
    console.error(erro.message); // acesso seguro a .message
  }
}

// Type guard personalizado (type predicate)
function isUsuario(obj: unknown): obj is Usuario {
  return typeof obj === 'object' && obj !== null && 'email' in obj;
}
```

---

## 8. Projeto TypeScript para Web (HTML, CSS e SCSS)

Para projetos web com TypeScript, SCSS e HTML, o **Vite** é a ferramenta de build mais utilizada atualmente. Ele oferece servidor de desenvolvimento com hot reload instantâneo, compilação de TypeScript e SCSS sem configuração extra e geração de bundle otimizado para produção.

> Documentação: [Vite](https://vitejs.dev) · [Sass](https://sass-lang.com)

### Criando o projeto com Vite

```bash
npm create vite@latest meu-projeto -- --template vanilla-ts
cd meu-projeto
npm install
```

O template `vanilla-ts` cria um projeto sem framework, com TypeScript configurado.

### Instalação do SCSS

```bash
# Apenas o compilador sass — o Vite detecta automaticamente arquivos .scss
npm install --save-dev sass
```

Não é necessária nenhuma configuração adicional: basta renomear os arquivos `.css` para `.scss` e importá-los normalmente.

### Estrutura de projeto

```
meu-projeto/
├── public/                  # arquivos estáticos copiados sem processamento
│   └── favicon.ico
├── src/
│   ├── main.ts              # ponto de entrada TypeScript
│   ├── styles/
│   │   ├── main.scss        # estilos globais
│   │   ├── _variables.scss  # partial: variáveis
│   │   └── _components.scss # partial: componentes
│   ├── components/
│   │   ├── card.ts
│   │   └── card.scss
│   ├── services/
│   │   └── produto.service.ts
│   └── types/
│       └── index.ts
├── dist/                    # gerado pelo build (não versionar)
├── index.html               # único HTML do projeto — ponto de entrada do Vite
├── tsconfig.json
├── vite.config.ts
└── package.json
```

> O Vite usa o `index.html` da raiz como único ponto de entrada. Não existe `index.html` dentro de `src/` — essa pasta contém apenas arquivos TypeScript, SCSS e assets processados pelo bundler. O HTML referencia o `main.ts` diretamente com `<script type="module">`.

### index.html

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Meu App</title>
</head>
<body>
  <div id="app"></div>

  <!-- O Vite processa o TypeScript diretamente no HTML -->
  <script type="module" src="/src/main.ts"></script>
</body>
</html>
```

### vite.config.ts

```ts
import { defineConfig } from 'vite';

export default defineConfig({
  // Pasta raiz dos fontes
  root: '.',

  // Pasta de saída do build
  build: {
    outDir: 'dist',
    sourcemap: true,       // gera source maps para debug em produção
    minify: 'esbuild',     // minificação rápida (padrão)
  },

  // Servidor de desenvolvimento
  server: {
    port: 3000,
    open: true,            // abre o navegador automaticamente
  },

  // Variáveis CSS disponíveis globalmente em todos os arquivos SCSS
  // (sem precisar importar em cada arquivo)
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@use '/src/styles/_variables.scss' as *;`,
      },
    },
  },
});
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "resolveJsonModule": true,
    "noEmit": true,
    "lib": ["ES2022", "DOM", "DOM.Iterable"]
  },
  "include": ["src/**/*", "vite.config.ts"],
  "exclude": ["node_modules", "dist"]
}
```

**Diferenças em relação ao tsconfig para Node.js:**

| Opção | Web (Vite) | Node.js |
|---|---|---|
| `module` | `ESNext` | `NodeNext` |
| `moduleResolution` | `bundler` | `NodeNext` |
| `noEmit` | `true` (Vite compila) | `false` |
| `lib` | inclui `DOM` e `DOM.Iterable` | omite ou usa `ES2022` |

### Scripts no package.json

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview",
    "typecheck": "tsc --noEmit"
  }
}
```

| Script | O que faz |
|---|---|
| `dev` | Inicia o servidor de desenvolvimento com hot reload |
| `build` | Verifica os tipos e gera o bundle de produção em `dist/` |
| `preview` | Serve o bundle de `dist/` localmente para validação |
| `typecheck` | Verifica tipos sem gerar arquivos |

### Exemplo de código

```ts
// src/types/index.ts
export interface Produto {
  id: number;
  nome: string;
  preco: number;
  categoria: string;
}
```

```scss
// src/styles/_variables.scss
$color-primary: #2563eb;
$color-text: #1e293b;
$border-radius: 8px;
$spacing-md: 1rem;

:root {
  --color-primary: #{$color-primary};
  --color-text: #{$color-text};
}
```

```scss
// src/styles/main.scss
@use './variables' as *;

*, *::before, *::after { box-sizing: border-box; }

body {
  font-family: system-ui, sans-serif;
  color: $color-text;
  margin: 0;
  padding: $spacing-md;
}

#app {
  max-width: 960px;
  margin-inline: auto;
}
```

```ts
// src/services/produto.service.ts
import type { Produto } from '../types/index';

export async function listarProdutos(): Promise<Produto[]> {
  const resposta = await fetch('/api/produtos');
  if (!resposta.ok) throw new Error(`Erro ${resposta.status}`);
  return resposta.json();
}
```

```scss
// src/components/card.scss
.card {
  background: white;
  border-radius: $border-radius;
  padding: $spacing-md;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.12);

  &__titulo {
    font-size: 1.1rem;
    font-weight: 600;
    color: $color-primary;
  }

  &__preco {
    font-size: 1.25rem;
    margin-top: 0.5rem;
  }
}
```

```ts
// src/components/card.ts
import './card.scss'; // Vite processa o SCSS ao importar
import type { Produto } from '../types/index';

export function criarCard(produto: Produto): HTMLElement {
  const article = document.createElement('article');
  article.className = 'card';
  article.innerHTML = `
    <h2 class="card__titulo">${produto.nome}</h2>
    <p class="card__preco">R$ ${produto.preco.toFixed(2)}</p>
    <span>${produto.categoria}</span>
  `;
  return article;
}
```

```ts
// src/main.ts
import './styles/main.scss';
import { listarProdutos } from './services/produto.service';
import { criarCard } from './components/card';

const app = document.getElementById('app')!;

const produtos = await listarProdutos();
produtos.forEach(produto => app.appendChild(criarCard(produto)));
```

### Fluxo de desenvolvimento e build

```bash
# Iniciar em modo desenvolvimento (hot reload)
npm run dev
# → http://localhost:3000

# Verificar tipos isoladamente
npm run typecheck

# Gerar build de produção
npm run build
# → dist/index.html, dist/assets/*.js, dist/assets/*.css

# Visualizar o build localmente antes de publicar
npm run preview
# → http://localhost:4173
```

O Vite processa automaticamente durante o `build`:
- TypeScript → JavaScript minificado
- SCSS → CSS minificado com prefixos de vendor
- Imagens e assets → nomes com hash para cache busting
- `index.html` → atualizado com os paths dos assets gerados

---

## 9. Exemplos Práticos

Exemplos autocontidos usando apenas HTML, CSS e JavaScript nativo.

---

### Confirmação com `<dialog>`

O elemento `<dialog>` permite criar modais nativos sem bibliotecas. O método `showModal()` abre o diálogo bloqueando interação com o restante da página; `close()` o fecha.

```html
<button id="btn-excluir">Excluir registro</button>

<dialog id="dialog-confirmacao">
  <h3>Confirmar exclusão</h3>
  <p>Tem certeza que deseja excluir este registro? Esta ação não pode ser desfeita.</p>
  <div class="dialog-acoes">
    <button id="btn-cancelar">Cancelar</button>
    <button id="btn-confirmar">Excluir</button>
  </div>
</dialog>
```

```css
dialog {
  border: none;
  border-radius: 8px;
  padding: 2rem;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 8px 30px rgb(0 0 0 / 0.2);
}

/* Backdrop nativo do dialog */
dialog::backdrop {
  background: rgb(0 0 0 / 0.5);
}

.dialog-acoes {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1.5rem;
}
```

```js
const dialog   = document.getElementById('dialog-confirmacao');
const btnAbrir = document.getElementById('btn-excluir');
const btnCancelar  = document.getElementById('btn-cancelar');
const btnConfirmar = document.getElementById('btn-confirmar');

btnAbrir.addEventListener('click', () => {
  dialog.showModal();
});

btnCancelar.addEventListener('click', () => {
  dialog.close();
});

btnConfirmar.addEventListener('click', () => {
  // executa a ação confirmada
  console.log('Registro excluído.');
  dialog.close();
});

// Fecha ao clicar no backdrop (fora do dialog)
dialog.addEventListener('click', (e) => {
  if (e.target === dialog) dialog.close();
});
```

---

### Botão com debounce e estado de carregamento

Ao clicar, o botão é desabilitado, muda de visual para indicar processamento e volta ao estado normal após a operação concluir. Evita múltiplos cliques acidentais ou intencionais.

```html
<button id="btn-salvar" class="btn">
  <span class="btn-texto">Salvar</span>
  <span class="btn-spinner" aria-hidden="true"></span>
</button>
```

```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1.25rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s, opacity 0.2s;
}

.btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

/* Spinner: elemento vazio animado com border */
.btn-spinner {
  display: none;
  width: 1rem;
  height: 1rem;
  border: 2px solid rgb(255 255 255 / 0.4);
  border-top-color: white;
  border-radius: 50%;
  animation: girar 0.7s linear infinite;
}

.btn--carregando .btn-spinner {
  display: inline-block;
}

.btn--carregando .btn-texto::after {
  content: 'Salvando...';
}

.btn--carregando .btn-texto {
  font-size: 0; /* oculta o texto original sem remover do DOM */
}

.btn--carregando .btn-texto::after {
  font-size: 1rem;
}

@keyframes girar {
  to { transform: rotate(360deg); }
}
```

```js
const btn = document.getElementById('btn-salvar');

async function salvar() {
  // Simula uma operação demorada (ex: requisição ao servidor)
  await new Promise(resolve => setTimeout(resolve, 2500));
  console.log('Salvo com sucesso!');
}

btn.addEventListener('click', async () => {
  btn.disabled = true;
  btn.classList.add('btn--carregando');

  try {
    await salvar();
  } finally {
    // Restaura o botão mesmo se a operação falhar
    btn.disabled = false;
    btn.classList.remove('btn--carregando');
  }
});
```

---

### Limitar `datetime-local` até a data e hora atual

Impede que o usuário selecione datas futuras definindo o atributo `max` com a data e hora atuais no formato exigido pelo input (`YYYY-MM-DDTHH:MM`).

```html
<label for="data-evento">Data do evento</label>
<input type="datetime-local" id="data-evento" name="data-evento">
```

```js
const input = document.getElementById('data-evento');

function setMaxAgora() {
  const agora = new Date();

  // toISOString() retorna "2025-03-20T17:45:00.000Z" (UTC)
  // Ajusta para o fuso local antes de formatar
  const offset = agora.getTimezoneOffset() * 60 * 1000;
  const local  = new Date(agora - offset);

  // O atributo max exige o formato "YYYY-MM-DDTHH:MM"
  input.max = local.toISOString().slice(0, 16);
}

setMaxAgora();
```

---

### Campo de formulário acessível com validação visual

Estrutura completa de um campo: `<label>` associado, `placeholder`, texto de ajuda, e áreas para indicar sucesso ou erro, com classes CSS aplicadas via JavaScript após validação.

```html
<div class="campo" id="campo-email">
  <label class="campo-label" for="email">
    E-mail <span class="campo-obrigatorio" aria-hidden="true">*</span>
  </label>

  <input
    class="campo-input"
    type="email"
    id="email"
    name="email"
    placeholder="seu@email.com"
    required
    aria-describedby="email-ajuda email-erro"
  >

  <span class="campo-ajuda" id="email-ajuda">
    Informe o e-mail cadastrado na sua conta.
  </span>

  <span class="campo-erro" id="email-erro" role="alert" aria-live="polite"></span>
</div>
```

```css
.campo {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  max-width: 360px;
}

.campo-label {
  font-weight: 500;
  color: #1e293b;
}

.campo-obrigatorio {
  color: #dc2626;
  margin-left: 2px;
}

.campo-input {
  padding: 0.5rem 0.75rem;
  border: 1.5px solid #cbd5e1;
  border-radius: 6px;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.campo-input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgb(37 99 235 / 0.15);
}

.campo-ajuda {
  font-size: 0.875rem;
  color: #64748b;
}

.campo-erro {
  font-size: 0.875rem;
  color: #dc2626;
  min-height: 1.25rem; /* reserva espaço para evitar CLS */
}

/* Estado: válido */
.campo--valido .campo-input {
  border-color: #16a34a;
}

.campo--valido .campo-input:focus {
  box-shadow: 0 0 0 3px rgb(22 163 74 / 0.15);
}

/* Estado: inválido */
.campo--invalido .campo-input {
  border-color: #dc2626;
}

.campo--invalido .campo-input:focus {
  box-shadow: 0 0 0 3px rgb(220 38 38 / 0.15);
}
```

```js
const campoEl = document.getElementById('campo-email');
const inputEl = document.getElementById('email');
const erroEl  = document.getElementById('email-erro');

function validarEmail(valor) {
  if (!valor) return 'O e-mail é obrigatório.';
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(valor)) return 'Informe um e-mail válido.';
  return null; // sem erro
}

function aplicarEstado(erro) {
  campoEl.classList.remove('campo--valido', 'campo--invalido');

  if (erro) {
    campoEl.classList.add('campo--invalido');
    erroEl.textContent = erro;
  } else {
    campoEl.classList.add('campo--valido');
    erroEl.textContent = '';
  }
}

// Valida ao sair do campo (blur)
inputEl.addEventListener('blur', () => {
  const erro = validarEmail(inputEl.value.trim());
  aplicarEstado(erro);
});

// Revalida enquanto o usuário corrige (input)
inputEl.addEventListener('input', () => {
  if (campoEl.classList.contains('campo--invalido')) {
    const erro = validarEmail(inputEl.value.trim());
    aplicarEstado(erro);
  }
});
```

---

## 10. Recursos Avançados

---

### Intl — Internacionalização

> MDN: [Intl](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Intl)

O objeto global **`Intl`** fornece formatação sensível ao idioma e à localidade para números, datas, moedas, listas e textos relativos — sem bibliotecas externas. Todos os construtores aceitam uma _locale_ (ex: `'pt-BR'`, `'en-US'`) e um objeto de opções.

#### Intl.NumberFormat — números e moedas

```js
// Número com separadores da localidade
new Intl.NumberFormat('pt-BR').format(1234567.89);
// '1.234.567,89'

new Intl.NumberFormat('en-US').format(1234567.89);
// '1,234,567.89'

// Moeda
new Intl.NumberFormat('pt-BR', {
  style: 'currency',
  currency: 'BRL',
}).format(1990.5);
// 'R$ 1.990,50'

new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
}).format(1990.5);
// '$1,990.50'

// Porcentagem
new Intl.NumberFormat('pt-BR', {
  style: 'percent',
  minimumFractionDigits: 1,
}).format(0.1234);
// '12,3%'

// Unidades de medida
new Intl.NumberFormat('pt-BR', {
  style: 'unit',
  unit: 'kilometer-per-hour',
}).format(120);
// '120 km/h'

// Notação compacta (abreviada)
new Intl.NumberFormat('pt-BR', { notation: 'compact' }).format(1_500_000);
// '1,5 mi'

new Intl.NumberFormat('en-US', { notation: 'compact' }).format(1_500_000);
// '1.5M'

// Reutilizável — criar uma instância e chamar .format() múltiplas vezes
const moeda = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' });
[10, 200, 3000].map(v => moeda.format(v));
// ['R$ 10,00', 'R$ 200,00', 'R$ 3.000,00']
```

#### Intl.DateTimeFormat — datas e horas

```js
const data = new Date('2025-03-20T14:35:00');

// Data completa
new Intl.DateTimeFormat('pt-BR').format(data);
// '20/03/2025'

// Com opções de estilo
new Intl.DateTimeFormat('pt-BR', { dateStyle: 'full' }).format(data);
// 'quinta-feira, 20 de março de 2025'

new Intl.DateTimeFormat('pt-BR', { dateStyle: 'long' }).format(data);
// '20 de março de 2025'

// Data e hora juntas
new Intl.DateTimeFormat('pt-BR', {
  dateStyle: 'short',
  timeStyle: 'short',
}).format(data);
// '20/03/2025, 14:35'

// Controle granular dos campos
new Intl.DateTimeFormat('pt-BR', {
  weekday: 'long',
  day: 'numeric',
  month: 'long',
  year: 'numeric',
  hour: '2-digit',
  minute: '2-digit',
}).format(data);
// 'quinta-feira, 20 de março de 2025, 14:35'

// Apenas o nome do mês
new Intl.DateTimeFormat('pt-BR', { month: 'long' }).format(data);
// 'março'

// Fuso horário específico
new Intl.DateTimeFormat('pt-BR', {
  dateStyle: 'short',
  timeStyle: 'short',
  timeZone: 'America/Sao_Paulo',
}).format(data);
```

#### Intl.RelativeTimeFormat — tempo relativo

```js
const rtf = new Intl.RelativeTimeFormat('pt-BR', { numeric: 'auto' });

rtf.format(-1, 'day');     // 'ontem'
rtf.format(1, 'day');      // 'amanhã'
rtf.format(-3, 'day');     // 'há 3 dias'
rtf.format(2, 'week');     // 'em 2 semanas'
rtf.format(-1, 'month');   // 'mês passado'
rtf.format(1, 'year');     // 'próximo ano'
rtf.format(-30, 'minute'); // 'há 30 minutos'
rtf.format(-5, 'second');  // 'há 5 segundos'

// numeric: 'always' — não usa palavras como "ontem", sempre numérico
const rtfNumerico = new Intl.RelativeTimeFormat('pt-BR', { numeric: 'always' });
rtfNumerico.format(-1, 'day'); // 'há 1 dia'
```

**Utilitário para calcular e formatar tempo relativo automaticamente:**

```js
function tempoRelativo(data) {
  const rtf = new Intl.RelativeTimeFormat('pt-BR', { numeric: 'auto' });
  const diffMs = data - Date.now();
  const abs = Math.abs(diffMs);

  if (abs < 60_000)       return rtf.format(Math.round(diffMs / 1_000), 'second');
  if (abs < 3_600_000)    return rtf.format(Math.round(diffMs / 60_000), 'minute');
  if (abs < 86_400_000)   return rtf.format(Math.round(diffMs / 3_600_000), 'hour');
  if (abs < 2_592_000_000) return rtf.format(Math.round(diffMs / 86_400_000), 'day');
  return rtf.format(Math.round(diffMs / 2_592_000_000), 'month');
}

tempoRelativo(new Date(Date.now() - 5 * 60_000));    // 'há 5 minutos'
tempoRelativo(new Date(Date.now() + 2 * 86_400_000)); // 'em 2 dias'
```

#### Intl.ListFormat — formatação de listas

```js
const lf = new Intl.ListFormat('pt-BR', { style: 'long', type: 'conjunction' });
lf.format(['HTML', 'CSS', 'JavaScript']);
// 'HTML, CSS e JavaScript'

const lfOu = new Intl.ListFormat('pt-BR', { type: 'disjunction' });
lfOu.format(['segunda', 'quarta', 'sexta']);
// 'segunda, quarta ou sexta'

// Estilo narrow (sem palavras de conexão longas)
const lfNarrow = new Intl.ListFormat('pt-BR', { style: 'narrow', type: 'conjunction' });
lfNarrow.format(['A', 'B', 'C']);
// 'A, B e C'

// Em inglês
new Intl.ListFormat('en-US', { type: 'conjunction' }).format(['HTML', 'CSS', 'JS']);
// 'HTML, CSS, and JS'
```

#### Intl.Collator — ordenação de strings

Comparação e ordenação de strings respeitando as regras do idioma (acentuação, capitalização, etc.).

```js
// Ordenar array de strings com regras do português
const nomes = ['Álvaro', 'ana', 'Bruno', 'ágatha', 'Carla'];

// Ordenação nativa (incorreta para pt-BR — trata acentos e maiúsculas de forma errada)
nomes.sort();
// ['Bruno', 'Carla', 'ana', 'ágatha', 'Álvaro']

// Ordenação com Intl.Collator (correta)
nomes.sort(new Intl.Collator('pt-BR', { sensitivity: 'base' }).compare);
// ['ágatha', 'Álvaro', 'ana', 'Bruno', 'Carla']

// Comparação direta (retorna -1, 0 ou 1)
const col = new Intl.Collator('pt-BR');
col.compare('a', 'á'); // -1 ('a' vem antes de 'á')
col.compare('A', 'a'); // -1 ou 0 dependendo de sensitivity

// sensitivity: 'base' ignora capitalização e diacríticos na comparação
const colBase = new Intl.Collator('pt-BR', { sensitivity: 'base' });
colBase.compare('a', 'Á'); // 0 — tratados como iguais
```

#### Intl.PluralRules — regras de plural

```js
const pr = new Intl.PluralRules('pt-BR');

pr.select(0); // 'other' → '0 itens'
pr.select(1); // 'one'   → '1 item'
pr.select(2); // 'other' → '2 itens'

// Utilitário para exibir a forma correta
function plural(quantidade, singular, pluralStr) {
  const pr = new Intl.PluralRules('pt-BR');
  return `${quantidade} ${pr.select(quantidade) === 'one' ? singular : pluralStr}`;
}

plural(1, 'resultado', 'resultados'); // '1 resultado'
plural(5, 'resultado', 'resultados'); // '5 resultados'
```

---

### localStorage e sessionStorage

> MDN: [localStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/localStorage) · [sessionStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/sessionStorage)

Ambos permitem armazenar pares chave/valor no navegador, sem expiração de cookies e sem enviar dados ao servidor.

| | `localStorage` | `sessionStorage` |
|---|---|---|
| **Persistência** | Permanente (até ser limpo) | Apenas enquanto a aba está aberta |
| **Escopo** | Compartilhado entre abas do mesmo domínio | Isolado por aba |
| **Capacidade** | ~5 MB por origem | ~5 MB por origem |
| **Uso típico** | Preferências do usuário, tema, token | Dados de formulário temporários, estado de sessão |

**API (idêntica nos dois):**

```js
// Salvar — somente strings; usar JSON para objetos
localStorage.setItem('tema', 'escuro');
localStorage.setItem('usuario', JSON.stringify({ id: 1, nome: 'Ana' }));

// Ler
const tema = localStorage.getItem('tema');          // 'escuro' ou null
const usuario = JSON.parse(localStorage.getItem('usuario'));

// Remover uma chave
localStorage.removeItem('tema');

// Limpar tudo
localStorage.clear();

// Iterar sobre todas as chaves
for (let i = 0; i < localStorage.length; i++) {
  const chave = localStorage.key(i);
  console.log(chave, localStorage.getItem(chave));
}
```

**Utilitário com tratamento de erro:**
```js
const storage = {
  get(chave, padrao = null) {
    try {
      const valor = localStorage.getItem(chave);
      return valor !== null ? JSON.parse(valor) : padrao;
    } catch {
      return padrao;
    }
  },
  set(chave, valor) {
    try {
      localStorage.setItem(chave, JSON.stringify(valor));
    } catch (e) {
      // QuotaExceededError: armazenamento cheio
      console.warn('localStorage cheio:', e);
    }
  },
  remove(chave) {
    localStorage.removeItem(chave);
  },
};

// Uso
storage.set('config', { idioma: 'pt-BR', notificacoes: true });
const config = storage.get('config', {});
```

**Detectar mudanças entre abas:**
```js
// O evento 'storage' dispara em outras abas quando localStorage muda
window.addEventListener('storage', (e) => {
  console.log('Chave alterada:', e.key);
  console.log('Valor anterior:', e.oldValue);
  console.log('Novo valor:', e.newValue);
});
```

> **Atenção:** nunca armazenar senhas, tokens de autenticação sensíveis ou dados pessoais no localStorage — é acessível por qualquer JavaScript da página e vulnerável a XSS.

---

### IndexedDB

> MDN: [IndexedDB API](https://developer.mozilla.org/pt-BR/docs/Web/API/IndexedDB_API)

**IndexedDB** é um banco de dados NoSQL orientado a objetos, embutido no navegador. Armazena grandes volumes de dados estruturados (objetos JS, Blobs, arquivos) com suporte a índices, transações e consultas — tudo de forma assíncrona.

**Comparação com localStorage:**

| | `localStorage` | `IndexedDB` |
|---|---|---|
| **Tipo de dado** | Somente strings | Qualquer valor JS, Blobs, arquivos |
| **Capacidade** | ~5 MB | Centenas de MB (limitado pelo disco) |
| **Acesso** | Síncrono (bloqueia a thread) | Assíncrono (não bloqueia) |
| **Consultas** | Somente por chave exata | Por chave, índice ou faixa de valores |
| **Transações** | Não | Sim |
| **Uso típico** | Preferências simples | Dados offline, cache de API, arquivos |

**Conceitos principais:**

| Conceito | Descrição |
|---|---|
| **Database** | O banco de dados, identificado por nome e versão |
| **Object Store** | Equivalente a uma tabela; armazena objetos JS |
| **Index** | Campo secundário para buscas rápidas |
| **Transaction** | Agrupa operações de leitura/escrita com rollback automático em caso de erro |
| **Cursor** | Itera sobre múltiplos registros de um store ou índice |

#### Abrir o banco e criar stores

```js
const DB_NOME = 'meu-banco';
const DB_VERSAO = 1;

function abrirBanco() {
  return new Promise((resolve, reject) => {
    const req = indexedDB.open(DB_NOME, DB_VERSAO);

    // onupgradeneeded: executado na criação ou quando a versão aumenta
    req.onupgradeneeded = (e) => {
      const db = e.target.result;

      // Criar object store com chave automática
      if (!db.objectStoreNames.contains('produtos')) {
        const store = db.createObjectStore('produtos', {
          keyPath: 'id',           // campo que serve como chave primária
          autoIncrement: true,     // gera id automaticamente se não informado
        });

        // Criar índices para busca por outros campos
        store.createIndex('por_nome',     'nome',     { unique: false });
        store.createIndex('por_categoria','categoria',{ unique: false });
      }
    };

    req.onsuccess = (e) => resolve(e.target.result);
    req.onerror   = (e) => reject(e.target.error);
  });
}
```

#### Operações CRUD

A API nativa do IndexedDB é baseada em eventos, o que resulta em muito código verboso. O padrão é encapsulá-la em Promises:

```js
// Inserir / atualizar (put sobrescreve; add lança erro se a chave já existir)
async function salvar(db, storeName, objeto) {
  return new Promise((resolve, reject) => {
    const tx    = db.transaction(storeName, 'readwrite');
    const store = tx.objectStore(storeName);
    const req   = store.put(objeto);

    req.onsuccess = () => resolve(req.result); // retorna a chave gerada
    req.onerror   = () => reject(req.error);
  });
}

// Buscar por chave primária
async function buscarPorId(db, storeName, id) {
  return new Promise((resolve, reject) => {
    const tx    = db.transaction(storeName, 'readonly');
    const store = tx.objectStore(storeName);
    const req   = store.get(id);

    req.onsuccess = () => resolve(req.result); // undefined se não encontrado
    req.onerror   = () => reject(req.error);
  });
}

// Buscar todos os registros
async function buscarTodos(db, storeName) {
  return new Promise((resolve, reject) => {
    const tx    = db.transaction(storeName, 'readonly');
    const store = tx.objectStore(storeName);
    const req   = store.getAll();

    req.onsuccess = () => resolve(req.result);
    req.onerror   = () => reject(req.error);
  });
}

// Buscar por índice
async function buscarPorCategoria(db, categoria) {
  return new Promise((resolve, reject) => {
    const tx    = db.transaction('produtos', 'readonly');
    const index = tx.objectStore('produtos').index('por_categoria');
    const req   = index.getAll(categoria); // todos com aquela categoria

    req.onsuccess = () => resolve(req.result);
    req.onerror   = () => reject(req.error);
  });
}

// Remover por chave
async function remover(db, storeName, id) {
  return new Promise((resolve, reject) => {
    const tx    = db.transaction(storeName, 'readwrite');
    const store = tx.objectStore(storeName);
    const req   = store.delete(id);

    req.onsuccess = () => resolve();
    req.onerror   = () => reject(req.error);
  });
}
```

#### Exemplo de uso

```js
const db = await abrirBanco();

// Inserir
const id = await salvar(db, 'produtos', {
  nome: 'Notebook',
  preco: 3500,
  categoria: 'eletronicos',
});

// Buscar
const produto = await buscarPorId(db, 'produtos', id);
console.log(produto.nome); // 'Notebook'

// Listar por índice
const eletronicos = await buscarPorCategoria(db, 'eletronicos');

// Remover
await remover(db, 'produtos', id);
```

#### Iteração com cursor

Útil para processar registros um a um ou aplicar filtros mais complexos:

```js
async function listarComCursor(db, storeName) {
  return new Promise((resolve, reject) => {
    const resultados = [];
    const tx    = db.transaction(storeName, 'readonly');
    const store = tx.objectStore(storeName);
    const req   = store.openCursor();

    req.onsuccess = (e) => {
      const cursor = e.target.result;
      if (cursor) {
        resultados.push(cursor.value);
        cursor.continue(); // avança para o próximo registro
      } else {
        resolve(resultados); // cursor esgotado
      }
    };
    req.onerror = () => reject(req.error);
  });
}
```

> **Dica:** para projetos reais, usar a biblioteca **[idb](https://github.com/jakearchibald/idb)** (wrapper com Promises sobre a API nativa, sem dependências, ~1 KB). Ela elimina o código de eventos e torna o IndexedDB tão simples quanto `await db.get('produtos', id)`.

---

### Geolocation API

> MDN: [Geolocation API](https://developer.mozilla.org/pt-BR/docs/Web/API/Geolocation_API)

Permite obter a posição geográfica do dispositivo (com permissão do usuário). Funciona via GPS, Wi-Fi ou IP.

```js
// Verificar suporte
if (!navigator.geolocation) {
  console.error('Geolocation não suportada neste navegador.');
}

// Obter posição uma única vez
navigator.geolocation.getCurrentPosition(
  (posicao) => {
    const { latitude, longitude, accuracy, altitude, speed } = posicao.coords;
    const timestamp = posicao.timestamp; // ms desde epoch

    console.log(`Lat: ${latitude}, Lon: ${longitude}`);
    console.log(`Precisão: ${accuracy} metros`);
  },
  (erro) => {
    switch (erro.code) {
      case erro.PERMISSION_DENIED:
        console.error('Usuário negou a permissão.');
        break;
      case erro.POSITION_UNAVAILABLE:
        console.error('Localização indisponível.');
        break;
      case erro.TIMEOUT:
        console.error('Tempo limite atingido.');
        break;
    }
  },
  {
    enableHighAccuracy: true, // tenta usar GPS (mais lento e consome bateria)
    timeout: 10000,           // ms antes de chamar o callback de erro
    maximumAge: 60000,        // aceita posição em cache de até 60s
  }
);

// Monitorar posição continuamente
const watchId = navigator.geolocation.watchPosition(
  (posicao) => console.log(posicao.coords),
  (erro) => console.error(erro),
);

// Parar o monitoramento
navigator.geolocation.clearWatch(watchId);
```

**Exemplo: link para o Google Maps com a posição atual:**
```js
navigator.geolocation.getCurrentPosition(({ coords }) => {
  const { latitude, longitude } = coords;
  const url = `https://www.google.com/maps?q=${latitude},${longitude}`;
  document.getElementById('link-mapa').href = url;
});
```

> A Geolocation API só funciona em **HTTPS** (ou `localhost`).

---

### Web Workers

> MDN: [Web Workers](https://developer.mozilla.org/pt-BR/docs/Web/API/Web_Workers_API)

Web Workers executam scripts em uma **thread separada**, sem bloquear a thread principal (UI). Úteis para processamento pesado como cálculos, parsing de arquivos grandes ou ordenação de grandes volumes de dados.

**Limitações do worker:** sem acesso ao DOM, `window`, `document` ou `localStorage`. Pode usar `fetch`, `IndexedDB` e a maioria das APIs de dados.

```js
// worker.js — arquivo separado executado na thread worker
self.addEventListener('message', (e) => {
  const { dados } = e.data;

  // Processamento pesado (ex: ordenar um array enorme)
  const resultado = dados.sort((a, b) => a - b);

  // Envia resultado de volta para a thread principal
  self.postMessage({ resultado });
});
```

```js
// main.js — thread principal
const worker = new Worker('worker.js');

// Enviar dados para o worker
worker.postMessage({ dados: [5, 3, 8, 1, 9, 2] });

// Receber resultado do worker
worker.addEventListener('message', (e) => {
  console.log('Resultado:', e.data.resultado); // [1, 2, 3, 5, 8, 9]
});

// Tratar erros do worker
worker.addEventListener('error', (e) => {
  console.error('Erro no worker:', e.message);
});

// Encerrar o worker quando não for mais necessário
worker.terminate();
```

**Transferência de dados volumosos (Transferable Objects):**

Por padrão, dados são copiados entre threads (serialização). Para arrays binários grandes, é possível **transferir** a propriedade (sem cópia), tornando o objeto inacessível na thread de origem:

```js
const buffer = new ArrayBuffer(1024 * 1024 * 32); // 32 MB
worker.postMessage({ buffer }, [buffer]); // buffer transferido, não copiado
// buffer agora tem byteLength === 0 nesta thread
```

---

### File API

> MDN: [File API](https://developer.mozilla.org/pt-BR/docs/Web/API/File_API) · [FileReader](https://developer.mozilla.org/pt-BR/docs/Web/API/FileReader) · [File](https://developer.mozilla.org/pt-BR/docs/Web/API/File)

Permite ao JavaScript ler arquivos selecionados pelo usuário via `<input type="file">` ou arrastados para a página (drag and drop), sem enviar ao servidor.

```html
<input type="file" id="arquivo" accept="image/*,.pdf" multiple>
```

```js
const input = document.getElementById('arquivo');

input.addEventListener('change', () => {
  const arquivos = Array.from(input.files); // FileList → Array

  arquivos.forEach((arquivo) => {
    console.log('Nome:', arquivo.name);
    console.log('Tipo:', arquivo.type);      // MIME: 'image/png', 'application/pdf'
    console.log('Tamanho:', arquivo.size);   // bytes
    console.log('Modificado:', new Date(arquivo.lastModified));
  });
});
```

**Ler conteúdo com FileReader:**

```js
function lerArquivo(arquivo) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.addEventListener('load', () => resolve(reader.result));
    reader.addEventListener('error', () => reject(reader.error));

    // Escolha o método conforme o tipo de conteúdo:
    reader.readAsText(arquivo);              // string (txt, csv, json)
    // reader.readAsDataURL(arquivo);        // base64 — para exibir imagens
    // reader.readAsArrayBuffer(arquivo);    // binário — para processamento
  });
}

// Exemplo: exibir preview de imagem
input.addEventListener('change', async () => {
  const arquivo = input.files[0];
  if (!arquivo?.type.startsWith('image/')) return;

  const reader = new FileReader();
  reader.addEventListener('load', () => {
    document.getElementById('preview').src = reader.result;
  });
  reader.readAsDataURL(arquivo);
});
```

**Alternativa moderna com `text()` e `arrayBuffer()` (sem FileReader):**

```js
input.addEventListener('change', async () => {
  const arquivo = input.files[0];

  const texto = await arquivo.text();          // lê como string
  const buffer = await arquivo.arrayBuffer();  // lê como ArrayBuffer
  const url = URL.createObjectURL(arquivo);    // URL temporária para o arquivo

  document.getElementById('preview').src = url;

  // Liberar memória quando não precisar mais
  URL.revokeObjectURL(url);
});
```

**Drag and drop de arquivos:**
```js
const zona = document.getElementById('zona-drop');

zona.addEventListener('dragover', (e) => {
  e.preventDefault(); // necessário para permitir o drop
  zona.classList.add('arrastando');
});

zona.addEventListener('dragleave', () => {
  zona.classList.remove('arrastando');
});

zona.addEventListener('drop', (e) => {
  e.preventDefault();
  zona.classList.remove('arrastando');
  const arquivos = Array.from(e.dataTransfer.files);
  console.log('Arquivos soltos:', arquivos);
});
```

---

### Service Worker

> MDN: [Service Worker API](https://developer.mozilla.org/pt-BR/docs/Web/API/Service_Worker_API)

Um **Service Worker** é um script JavaScript que o navegador executa em background, em uma thread separada da página, sem acesso ao DOM. Atua como um proxy programável entre a aplicação e a rede: intercepta requisições, gerencia cache e permite funcionamento offline.

**Ciclo de vida:**

```
register() → download → install → activate → idle → fetch/message → terminated
```

| Fase | Evento | O que fazer |
|---|---|---|
| **install** | `install` | Cachear os arquivos essenciais do app |
| **activate** | `activate` | Remover caches de versões antigas |
| **intercepção** | `fetch` | Decidir como responder cada requisição |
| **comunicação** | `message` | Trocar mensagens com a página |

**Registro (na página principal):**

```js
// main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js', { scope: '/' })
    .then(reg => {
      console.log('SW registrado, escopo:', reg.scope);

      // Detectar quando uma nova versão está disponível
      reg.addEventListener('updatefound', () => {
        const novoSW = reg.installing;
        novoSW.addEventListener('statechange', () => {
          if (novoSW.state === 'installed' && navigator.serviceWorker.controller) {
            console.log('Nova versão disponível — recarregue a página.');
          }
        });
      });
    })
    .catch(err => console.error('Falha ao registrar SW:', err));
}

// Comunicação página → service worker
navigator.serviceWorker.ready.then(reg => {
  reg.active.postMessage({ tipo: 'LIMPAR_CACHE' });
});

// Comunicação service worker → página
navigator.serviceWorker.addEventListener('message', (e) => {
  console.log('Mensagem do SW:', e.data);
});
```

**Arquivo do Service Worker (sw.js):**

```js
const CACHE_NAME = 'meu-app-v1';
const ASSETS_ESTATICOS = ['/', '/index.html', '/styles.css', '/app.js', '/icons/icon-192.png'];

// Instalação: pré-cacheia assets essenciais
self.addEventListener('install', (e) => {
  e.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS_ESTATICOS))
      .then(() => self.skipWaiting()) // ativa imediatamente sem esperar abas fecharem
  );
});

// Ativação: limpa caches de versões anteriores
self.addEventListener('activate', (e) => {
  e.waitUntil(
    caches.keys()
      .then(chaves => Promise.all(
        chaves
          .filter(chave => chave !== CACHE_NAME)
          .map(chave => caches.delete(chave))
      ))
      .then(() => self.clients.claim()) // assume controle das abas abertas imediatamente
  );
});

// Fetch: intercepta requisições e aplica estratégia de cache
self.addEventListener('fetch', (e) => {
  // Ignora requisições não-GET e de outras origens
  if (e.request.method !== 'GET' || !e.request.url.startsWith(self.location.origin)) return;

  e.respondWith(
    caches.match(e.request).then(cached => cached ?? fetch(e.request))
  );
});

// Comunicação: recebe mensagens da página
self.addEventListener('message', (e) => {
  if (e.data?.tipo === 'LIMPAR_CACHE') {
    caches.delete(CACHE_NAME);
  }
});
```

**Pontos importantes:**
- O arquivo `sw.js` deve estar na raiz (ou no `scope` desejado) — um SW em `/app/sw.js` não controla requisições de `/`.
- Service Workers só funcionam em **HTTPS** (ou `localhost`).
- Não têm acesso ao `DOM`, `window` ou `localStorage` — podem usar `IndexedDB`, `fetch` e `Cache API`.
- `skipWaiting()` + `clients.claim()` ativam o novo SW imediatamente, sem aguardar o fechamento das abas.

---

### Cache API

> MDN: [Cache API](https://developer.mozilla.org/pt-BR/docs/Web/API/Cache)

A **Cache API** é um armazenamento de pares `Request → Response`, acessível tanto no Service Worker quanto na página principal. É a base para o funcionamento offline de PWAs.

**Operações básicas:**

```js
// Abrir (ou criar) um cache pelo nome
const cache = await caches.open('meu-cache-v1');

// Adicionar uma URL ao cache (faz o fetch e armazena)
await cache.add('/index.html');

// Adicionar múltiplas URLs de uma vez
await cache.addAll(['/index.html', '/styles.css', '/app.js']);

// Armazenar um par Request/Response manualmente
const req = new Request('/api/config');
const res = await fetch(req);
await cache.put(req, res.clone()); // .clone() porque Response só pode ser lido uma vez

// Buscar no cache
const resposta = await caches.match('/index.html');
if (resposta) {
  const texto = await resposta.text();
}

// Buscar em todos os caches (sem especificar nome)
const respostaGlobal = await caches.match('/styles.css');

// Listar todos os caches da origem
const nomes = await caches.keys(); // ['meu-cache-v1', 'api-cache-v2']

// Remover um cache inteiro
await caches.delete('cache-antigo');

// Listar entradas de um cache específico
const entradas = await cache.keys(); // Array de Request
entradas.forEach(req => console.log(req.url));

// Remover uma entrada específica
await cache.delete('/arquivo-antigo.js');
```

**Estratégias de cache implementadas no `fetch` do SW:**

```js
// Cache First — assets estáticos (CSS, JS, imagens)
// Serve do cache; busca na rede apenas se não estiver cacheado
async function cacheFirst(req) {
  const cached = await caches.match(req);
  if (cached) return cached;

  const resposta = await fetch(req);
  const cache = await caches.open(CACHE_NAME);
  cache.put(req, resposta.clone());
  return resposta;
}

// Network First — dados dinâmicos (APIs, HTML atualizado)
// Tenta a rede; cai para o cache se offline ou com erro
async function networkFirst(req) {
  const cache = await caches.open(CACHE_NAME);
  try {
    const resposta = await fetch(req);
    cache.put(req, resposta.clone());
    return resposta;
  } catch {
    return await caches.match(req);
  }
}

// Stale While Revalidate — conteúdo que muda pouco (fontes, ícones)
// Retorna do cache imediatamente e atualiza em background
async function staleWhileRevalidate(req) {
  const cache = await caches.open(CACHE_NAME);
  const cached = await caches.match(req);

  const revalidar = fetch(req).then(resposta => {
    cache.put(req, resposta.clone());
    return resposta;
  });

  return cached ?? await revalidar;
}

// Aplicando as estratégias no evento fetch
self.addEventListener('fetch', (e) => {
  const { url } = e.request;

  if (url.includes('/api/')) {
    e.respondWith(networkFirst(e.request));
  } else if (url.match(/\.(png|jpg|webp|woff2)$/)) {
    e.respondWith(staleWhileRevalidate(e.request));
  } else {
    e.respondWith(cacheFirst(e.request));
  }
});
```

| Estratégia | Velocidade | Frescor dos dados | Uso típico |
|---|---|---|---|
| **Cache First** | Muito rápido | Dados podem estar desatualizados | CSS, JS, imagens |
| **Network First** | Depende da rede | Sempre atualizado (se online) | APIs, dados dinâmicos |
| **Stale While Revalidate** | Muito rápido | Atualiza em background | Fontes, ícones, conteúdo semi-estático |

---

### PWA — Progressive Web App

> MDN: [Progressive Web Apps](https://developer.mozilla.org/pt-BR/docs/Web/Progressive_web_apps)

Uma **PWA** é uma aplicação web que usa recursos modernos do navegador para se comportar como um aplicativo nativo: pode ser instalada na tela inicial, funcionar offline e receber notificações push.

**Os três pilares de uma PWA:**

| Pilar | Tecnologia |
|---|---|
| Instalável | Web App Manifest |
| Funciona offline | Service Worker + Cache API |
| Segura | HTTPS obrigatório |

#### Web App Manifest

Arquivo JSON que descreve o aplicativo para o navegador.

```json
{
  "name": "Meu App",
  "short_name": "App",
  "description": "Descrição do aplicativo.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

```html
<!-- Referenciado no <head> -->
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#2563eb">
```

**Valores de `display`:**

| Valor | Comportamento |
|---|---|
| `standalone` | Abre sem barra de endereço (parece app nativo) |
| `fullscreen` | Tela inteira (jogos) |
| `minimal-ui` | Barra mínima com voltar/recarregar |
| `browser` | Abre como aba normal do navegador |

> Para projetos reais, usar a biblioteca **Workbox** (do Google) para gerenciar o Service Worker e as estratégias de cache com abstrações de alto nível.

**Checklist PWA básica:**
- [ ] Servido via HTTPS
- [ ] `manifest.json` com nome, ícones e `start_url`
- [ ] Service Worker registrado com estratégia de cache definida
- [ ] Funciona com conexão lenta ou offline
- [ ] Responsivo e mobile-friendly
- [ ] `<meta name="theme-color">` definido

---

### Web Crypto API

> MDN: [Web Crypto API](https://developer.mozilla.org/pt-BR/docs/Web/API/Web_Crypto_API)

A **Web Crypto API** é uma interface nativa do navegador (e do Node.js a partir da v19) que oferece operações criptográficas de baixo nível: hashing, geração de números aleatórios seguros, assinatura digital, cifragem e geração de chaves — tudo sem bibliotecas externas.

O ponto de entrada é `crypto.subtle` (`SubtleCrypto`), que expõe métodos assíncronos (retornam `Promise`). Os dados são trafegados como `ArrayBuffer` ou `TypedArray`.

> Funciona apenas em **HTTPS** (ou `localhost`).

#### Geração de números aleatórios seguros

```js
// Preenche um TypedArray com bytes aleatórios criptograficamente seguros
const bytes = new Uint8Array(16);
crypto.getRandomValues(bytes);
// Uint8Array(16) [203, 14, 87, ...]

// Gerar um UUID v4 (nativo, sem biblioteca)
const uuid = crypto.randomUUID();
// 'f47ac10b-58cc-4372-a567-0e02b2c3d479'
```

#### Hashing (digest)

Algoritmos disponíveis: `SHA-1` (não recomendado), `SHA-256`, `SHA-384`, `SHA-512`.

```js
async function hash(texto, algoritmo = 'SHA-256') {
  const encoder = new TextEncoder();
  const dados = encoder.encode(texto);                        // string → Uint8Array
  const hashBuffer = await crypto.subtle.digest(algoritmo, dados);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join(''); // → hex
}

await hash('senha123');
// 'ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f'

await hash('hello', 'SHA-512');
// '9b71d224bd62f3785d96d46ad3ea3d...' (128 caracteres hex)
```

#### Geração de chaves simétricas (AES)

```js
// Gerar chave AES-GCM de 256 bits
const chave = await crypto.subtle.generateKey(
  { name: 'AES-GCM', length: 256 },
  true,          // extractable: permite exportar a chave
  ['encrypt', 'decrypt']
);

// Exportar a chave (para armazenar ou transmitir)
const chaveExportada = await crypto.subtle.exportKey('raw', chave);
// ArrayBuffer com 32 bytes (256 bits)

// Importar a chave a partir de bytes
const chaveImportada = await crypto.subtle.importKey(
  'raw',
  chaveExportada,
  { name: 'AES-GCM' },
  true,
  ['encrypt', 'decrypt']
);
```

#### Cifragem e decifragem — AES-GCM

AES-GCM é o modo recomendado: fornece **confidencialidade + autenticidade** (AEAD).

```js
// Cifrar
async function cifrar(chave, texto) {
  const encoder = new TextEncoder();
  const iv = crypto.getRandomValues(new Uint8Array(12)); // 96 bits — obrigatório e único por operação

  const cifrado = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    chave,
    encoder.encode(texto)
  );

  return { iv, cifrado }; // iv deve ser armazenado junto com o cifrado
}

// Decifrar
async function decifrar(chave, iv, cifrado) {
  const decifrado = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv },
    chave,
    cifrado
  );
  return new TextDecoder().decode(decifrado);
}

// Uso
const chave = await crypto.subtle.generateKey(
  { name: 'AES-GCM', length: 256 }, true, ['encrypt', 'decrypt']
);

const { iv, cifrado } = await cifrar(chave, 'Mensagem secreta');
const original = await decifrar(chave, iv, cifrado);
console.log(original); // 'Mensagem secreta'
```

**Converter ArrayBuffer ↔ Base64** (para armazenar ou transmitir):

```js
function bufferParaBase64(buffer) {
  return btoa(String.fromCharCode(...new Uint8Array(buffer)));
}

function base64ParaBuffer(base64) {
  return Uint8Array.from(atob(base64), c => c.charCodeAt(0)).buffer;
}

// Exemplo: serializar o resultado cifrado
const cifradoBase64 = bufferParaBase64(cifrado);
const ivBase64      = bufferParaBase64(iv);
```

#### Par de chaves assimétricas — ECDSA (assinatura digital)

```js
// Gerar par de chaves ECDSA (curva P-256)
const { privateKey, publicKey } = await crypto.subtle.generateKey(
  { name: 'ECDSA', namedCurve: 'P-256' },
  true,
  ['sign', 'verify']
);

// Assinar dados
async function assinar(privateKey, dados) {
  const encoder = new TextEncoder();
  return crypto.subtle.sign(
    { name: 'ECDSA', hash: 'SHA-256' },
    privateKey,
    encoder.encode(dados)
  );
}

// Verificar assinatura
async function verificar(publicKey, assinatura, dados) {
  const encoder = new TextEncoder();
  return crypto.subtle.verify(
    { name: 'ECDSA', hash: 'SHA-256' },
    publicKey,
    assinatura,
    encoder.encode(dados)
  );
}

// Uso
const mensagem = 'Documento importante';
const assinatura = await assinar(privateKey, mensagem);
const valida = await verificar(publicKey, assinatura, mensagem);
console.log(valida); // true
```

#### Derivação de chave a partir de senha — PBKDF2

Nunca use uma senha diretamente como chave criptográfica. Use PBKDF2 (ou HKDF) para derivar uma chave forte a partir da senha.

```js
async function derivarChave(senha, salt) {
  const encoder = new TextEncoder();

  // Importar a senha como material base
  const material = await crypto.subtle.importKey(
    'raw',
    encoder.encode(senha),
    'PBKDF2',
    false,
    ['deriveKey']
  );

  // Derivar chave AES-GCM
  return crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt: encoder.encode(salt),
      iterations: 310_000,   // OWASP recomenda ≥ 310.000 para SHA-256
      hash: 'SHA-256',
    },
    material,
    { name: 'AES-GCM', length: 256 },
    false,           // não exportável
    ['encrypt', 'decrypt']
  );
}

// Uso: cifrar dados protegidos por senha
const salt = crypto.randomUUID(); // gerar e armazenar junto com o cifrado
const chave = await derivarChave('minha-senha-forte', salt);
const { iv, cifrado } = await cifrar(chave, 'Dados sensíveis');
```

#### Exportar e importar chaves no formato JWK

JWK (JSON Web Key) é o formato padrão para interoperabilidade de chaves entre sistemas.

```js
// Exportar chave pública ECDSA como JWK
const jwkPublica = await crypto.subtle.exportKey('jwk', publicKey);
// { kty: 'EC', crv: 'P-256', x: '...', y: '...', key_ops: ['verify'] }

// Exportar chave privada ECDSA como JWK
const jwkPrivada = await crypto.subtle.exportKey('jwk', privateKey);

// Importar de volta a partir do JWK
const chavePublicaImportada = await crypto.subtle.importKey(
  'jwk',
  jwkPublica,
  { name: 'ECDSA', namedCurve: 'P-256' },
  true,
  ['verify']
);
```

#### Resumo dos algoritmos suportados

| Algoritmo | Uso | Operações |
|---|---|---|
| `SHA-256` / `SHA-512` | Hash de dados | `digest` |
| `AES-GCM` | Cifragem simétrica autenticada | `encrypt`, `decrypt` |
| `AES-CBC` | Cifragem simétrica (sem autenticação) | `encrypt`, `decrypt` |
| `RSA-OAEP` | Cifragem assimétrica | `encrypt`, `decrypt` |
| `ECDSA` | Assinatura digital | `sign`, `verify` |
| `RSASSA-PKCS1-v1_5` | Assinatura digital (RSA) | `sign`, `verify` |
| `ECDH` | Acordo de chaves (Diffie-Hellman) | `deriveKey`, `deriveBits` |
| `PBKDF2` | Derivação de chave a partir de senha | `deriveKey`, `deriveBits` |
| `HKDF` | Derivação de chave a partir de material | `deriveKey`, `deriveBits` |

> **Boas práticas:**
> - Prefira **AES-GCM** a AES-CBC — GCM autentica os dados além de cifrá-los.
> - Use um **IV único** a cada operação de cifragem (nunca reutilize o mesmo IV com a mesma chave).
> - Use **PBKDF2** ou **Argon2** (via biblioteca) para derivar chaves de senhas.
> - Armazene chaves exportadas de forma segura — nunca em `localStorage` sem cifragem adicional.

---

### Intersection Observer API

> MDN: [Intersection Observer API](https://developer.mozilla.org/pt-BR/docs/Web/API/Intersection_Observer_API)

A **Intersection Observer API** observa de forma assíncrona se um elemento entra ou sai da área visível do viewport (ou de outro elemento ancestral). É a alternativa moderna e eficiente ao uso de `scroll` + `getBoundingClientRect()`, pois não bloqueia a thread principal.

**Casos de uso comuns:**
- Lazy loading de imagens
- Infinite scroll (carregar mais itens ao chegar ao fim da lista)
- Animações disparadas quando o elemento aparece na tela
- Contabilização de impressões (analytics)

#### Uso básico

```js
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        console.log('Elemento visível:', entry.target);
      } else {
        console.log('Elemento saiu da tela:', entry.target);
      }
    });
  },
  {
    root: null,           // null = viewport do navegador
    rootMargin: '0px',    // margem ao redor do root (CSS shorthand)
    threshold: 0.5,       // 0 = qualquer pixel visível; 1 = 100% visível
  }
);

// Observar um elemento
const elemento = document.getElementById('meu-elemento');
observer.observe(elemento);

// Parar de observar
observer.unobserve(elemento);

// Desconectar todos os alvos
observer.disconnect();
```

**Propriedades de `IntersectionObserverEntry`:**

| Propriedade | Tipo | Descrição |
|---|---|---|
| `isIntersecting` | `boolean` | `true` se o elemento está (parcialmente) visível |
| `intersectionRatio` | `number` (0–1) | Fração do elemento que está visível |
| `target` | `Element` | O elemento observado |
| `boundingClientRect` | `DOMRect` | Dimensões/posição do elemento |
| `intersectionRect` | `DOMRect` | Área de interseção visível |
| `time` | `number` | Timestamp da mudança (ms desde a navegação) |

#### Lazy loading de imagens

```html
<img data-src="foto.jpg" alt="Foto" class="lazy" src="placeholder.svg">
```

```js
const observer = new IntersectionObserver(
  (entries, obs) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;

      const img = entry.target;
      img.src = img.dataset.src;       // substitui pelo src real
      img.classList.remove('lazy');
      obs.unobserve(img);              // para de observar após carregar
    });
  },
  { rootMargin: '200px' }             // carrega 200px antes de entrar na tela
);

document.querySelectorAll('img.lazy').forEach(img => observer.observe(img));
```

#### Animação ao entrar na tela

```css
.revelar {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.revelar.visivel {
  opacity: 1;
  transform: none;
}
```

```js
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      entry.target.classList.toggle('visivel', entry.isIntersecting);
    });
  },
  { threshold: 0.15 }  // dispara quando 15% do elemento fica visível
);

document.querySelectorAll('.revelar').forEach(el => observer.observe(el));
```

#### Infinite scroll

```js
// Observa um elemento sentinela no final da lista
const sentinela = document.getElementById('sentinela');

const observer = new IntersectionObserver(async (entries) => {
  const [entry] = entries;
  if (!entry.isIntersecting || carregando) return;

  carregando = true;
  const novosItens = await buscarMaisItens();
  renderizarItens(novosItens);
  carregando = false;
});

observer.observe(sentinela);
```

#### Múltiplos thresholds

```js
// Dispara callbacks em 0%, 25%, 50%, 75% e 100% de visibilidade
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(({ target, intersectionRatio }) => {
      target.style.opacity = intersectionRatio;  // fade proporcional à visibilidade
    });
  },
  { threshold: [0, 0.25, 0.5, 0.75, 1] }
);
```

> **Vantagem sobre `scroll`:** o Intersection Observer é executado fora da thread principal, não causa reflow e não precisa de throttle/debounce manual.

---

### Drag and Drop API

> MDN: [Drag and Drop API](https://developer.mozilla.org/pt-BR/docs/Web/API/HTML_Drag_and_Drop_API)

A **Drag and Drop API** permite que elementos HTML sejam arrastados e soltos em alvos designados. O fluxo envolve dois participantes: o **elemento arrastável** (draggable) e a **zona de drop** (drop target).

#### Eventos principais

| Evento | Dispara em | Momento |
|---|---|---|
| `dragstart` | Elemento arrastável | Início do arraste |
| `drag` | Elemento arrastável | Durante o arraste (repetido) |
| `dragend` | Elemento arrastável | Fim do arraste (solto ou cancelado) |
| `dragenter` | Zona de drop | Elemento entra na zona |
| `dragover` | Zona de drop | Elemento está sobre a zona (repetido) |
| `dragleave` | Zona de drop | Elemento sai da zona |
| `drop` | Zona de drop | Elemento é solto na zona |

#### Configuração básica

```html
<!-- Elemento arrastável: atributo draggable="true" -->
<div draggable="true" id="item-1" class="item">Item 1</div>

<!-- Zona de drop -->
<div id="zona-drop" class="zona">Solte aqui</div>
```

```js
const item  = document.getElementById('item-1');
const zona  = document.getElementById('zona-drop');

// 1. dragstart — armazenar dados no DataTransfer
item.addEventListener('dragstart', (e) => {
  e.dataTransfer.setData('text/plain', e.target.id);  // passa o id do elemento
  e.dataTransfer.effectAllowed = 'move';              // cursor indica "mover"
  e.target.classList.add('arrastando');
});

item.addEventListener('dragend', (e) => {
  e.target.classList.remove('arrastando');
});

// 2. dragover — deve chamar preventDefault() para permitir o drop
zona.addEventListener('dragover', (e) => {
  e.preventDefault();
  e.dataTransfer.dropEffect = 'move';
  zona.classList.add('sobre');
});

zona.addEventListener('dragleave', () => {
  zona.classList.remove('sobre');
});

// 3. drop — recuperar os dados e executar a ação
zona.addEventListener('drop', (e) => {
  e.preventDefault();
  zona.classList.remove('sobre');

  const id = e.dataTransfer.getData('text/plain');
  const elementoArrastado = document.getElementById(id);
  zona.appendChild(elementoArrastado);  // move o elemento para a zona
});
```

#### DataTransfer — transferindo dados

```js
// Armazenar múltiplos tipos de dado
e.dataTransfer.setData('text/plain', 'texto simples');
e.dataTransfer.setData('text/html',  '<b>texto HTML</b>');
e.dataTransfer.setData('application/json', JSON.stringify({ id: 42, nome: 'Item' }));

// Recuperar no drop
const texto = e.dataTransfer.getData('text/plain');
const obj   = JSON.parse(e.dataTransfer.getData('application/json'));

// Imagem personalizada de arraste
const img = new Image();
img.src = '/icons/drag-handle.png';
e.dataTransfer.setDragImage(img, 10, 10); // (imagem, offsetX, offsetY)
```

#### Exemplo: lista reordenável com drag and drop

```js
const lista = document.getElementById('lista');
let itemArrastado = null;

lista.addEventListener('dragstart', (e) => {
  itemArrastado = e.target;
  e.dataTransfer.effectAllowed = 'move';
  setTimeout(() => itemArrastado.classList.add('oculto'), 0); // oculta durante o arraste
});

lista.addEventListener('dragend', () => {
  itemArrastado?.classList.remove('oculto');
  itemArrastado = null;
  lista.querySelectorAll('.sobre').forEach(el => el.classList.remove('sobre'));
});

lista.addEventListener('dragover', (e) => {
  e.preventDefault();
  const alvo = e.target.closest('[draggable]');
  if (alvo && alvo !== itemArrastado) {
    alvo.classList.add('sobre');
  }
});

lista.addEventListener('dragleave', (e) => {
  e.target.closest?.('[draggable]')?.classList.remove('sobre');
});

lista.addEventListener('drop', (e) => {
  e.preventDefault();
  const alvo = e.target.closest('[draggable]');
  if (alvo && alvo !== itemArrastado) {
    alvo.classList.remove('sobre');
    // Insere antes ou depois do alvo conforme a posição do mouse
    const rect = alvo.getBoundingClientRect();
    const depoisDoMeio = e.clientY > rect.top + rect.height / 2;
    lista.insertBefore(itemArrastado, depoisDoMeio ? alvo.nextSibling : alvo);
  }
});
```

#### Arrastar arquivos do sistema de arquivos para a página

```js
const zona = document.getElementById('zona-drop');

zona.addEventListener('dragover', (e) => {
  e.preventDefault();
  e.dataTransfer.dropEffect = 'copy';
});

zona.addEventListener('drop', (e) => {
  e.preventDefault();
  const arquivos = Array.from(e.dataTransfer.files);

  arquivos.forEach((arquivo) => {
    console.log('Arquivo:', arquivo.name, arquivo.type, arquivo.size);
  });

  // Ler o conteúdo (ex: imagem como URL temporária)
  arquivos
    .filter(f => f.type.startsWith('image/'))
    .forEach((arquivo) => {
      const url = URL.createObjectURL(arquivo);
      const img = document.createElement('img');
      img.src = url;
      zona.appendChild(img);
    });
});
```

#### effectAllowed vs dropEffect

| `effectAllowed` (dragstart) | `dropEffect` (dragover/drop) | Resultado |
|---|---|---|
| `'copy'` | `'copy'` | Cursor de cópia |
| `'move'` | `'move'` | Cursor de mover |
| `'link'` | `'link'` | Cursor de link |
| `'copyMove'` | `'copy'` ou `'move'` | Permite ambos |
| `'all'` | Qualquer | Sem restrição |
| `'none'` | — | Drop bloqueado |

> **Dicas:**
> - `dragover` **deve** chamar `e.preventDefault()` para que o `drop` seja disparado.
> - Evite `drag` (dispara muitas vezes por segundo) para operações custosas — prefira `dragenter`/`dragleave`.
> - Para interfaces de arrastar mais complexas (Kanban, editores), considere bibliotecas como **SortableJS** ou **dnd kit** (React).

---

### Page Visibility API

> MDN: [Page Visibility API](https://developer.mozilla.org/pt-BR/docs/Web/API/Page_Visibility_API)

A **Page Visibility API** permite detectar quando uma página está visível ou oculta para o usuário — por exemplo, ao trocar de aba, minimizar a janela ou bloquear a tela. Evita processamento desnecessário quando o usuário não está interagindo com a página.

**Casos de uso comuns:**
- Pausar vídeos, animações ou timers quando a aba está inativa
- Suspender polling de dados e reduzir requisições à rede
- Economizar bateria em dispositivos móveis
- Contabilizar tempo real de visualização para analytics

#### Propriedades e evento

```js
// document.visibilityState — estado atual da página
document.visibilityState; // 'visible' | 'hidden'

// document.hidden — atalho booleano (true quando visibilityState === 'hidden')
document.hidden; // false | true

// Evento disparado sempre que o estado muda
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    console.log('Página oculta');
  } else {
    console.log('Página visível');
  }
});
```

**Valores de `visibilityState`:**

| Valor | Quando ocorre |
|---|---|
| `'visible'` | A aba está ativa e visível para o usuário |
| `'hidden'` | A aba está em segundo plano, a janela está minimizada, ou a tela do dispositivo está bloqueada |

#### Pausar e retomar um timer

```js
let intervaloId = null;
let contagem = 0;

function iniciarTimer() {
  intervaloId = setInterval(() => {
    contagem++;
    console.log('Tick:', contagem);
  }, 1000);
}

function pararTimer() {
  clearInterval(intervaloId);
  intervaloId = null;
}

// Inicia ao carregar
iniciarTimer();

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    pararTimer();
  } else {
    iniciarTimer();
  }
});
```

#### Pausar e retomar um vídeo automaticamente

```js
const video = document.querySelector('video');

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    video.pause();
  } else {
    video.play();
  }
});
```

#### Pausar animações com requestAnimationFrame

```js
let rafId = null;

function animar(timestamp) {
  // lógica da animação
  rafId = requestAnimationFrame(animar);
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    cancelAnimationFrame(rafId);
    rafId = null;
  } else {
    rafId = requestAnimationFrame(animar);
  }
});

// Inicia a animação
rafId = requestAnimationFrame(animar);
```

#### Suspender polling de dados

```js
let pollingId = null;

async function buscarDados() {
  const dados = await fetch('/api/status').then(r => r.json());
  atualizarUI(dados);
}

function iniciarPolling() {
  buscarDados(); // busca imediatamente ao iniciar
  pollingId = setInterval(buscarDados, 30_000);
}

function pararPolling() {
  clearInterval(pollingId);
  pollingId = null;
}

iniciarPolling();

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    pararPolling();
  } else {
    iniciarPolling(); // retoma e busca dados atualizados imediatamente
  }
});
```

#### Medir tempo ativo na página (analytics)

```js
let inicioVisivel = Date.now();
let tempoTotalMs = 0;

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // acumula o tempo desde a última vez que ficou visível
    tempoTotalMs += Date.now() - inicioVisivel;
    console.log('Tempo ativo acumulado:', (tempoTotalMs / 1000).toFixed(1), 's');
  } else {
    inicioVisivel = Date.now();
  }
});

// Registrar ao sair da página
window.addEventListener('beforeunload', () => {
  if (!document.hidden) {
    tempoTotalMs += Date.now() - inicioVisivel;
  }
  // navigator.sendBeacon('/api/analytics', JSON.stringify({ tempoMs: tempoTotalMs }));
});
```

#### Combinando com Page Lifecycle (freeze/resume)

Navegadores modernos podem "congelar" abas inativas para economizar recursos. Os eventos `freeze` e `resume` do **Page Lifecycle API** complementam a Page Visibility API:

```js
// Disparado quando a aba é congelada (mais agressivo que "hidden")
document.addEventListener('freeze', () => {
  // salvar estado crítico — o processo pode ser terminado a qualquer momento
  localStorage.setItem('rascunho', obterConteudoAtual());
});

// Disparado quando a aba é retomada do estado congelado
document.addEventListener('resume', () => {
  console.log('Aba retomada do estado congelado');
  sincronizarDados();
});
```

> **Suporte:** `visibilitychange` e `document.hidden` têm suporte amplo em todos os navegadores modernos. Os eventos `freeze`/`resume` são mais recentes — verificar suporte com `'onfreeze' in document` antes de usar.

---

## Comparativo de Frameworks Frontend — Angular, React e Vue

Referências detalhadas de cada framework:

- [Frontend-Angular.md](Frontend-Angular.md) — Angular 21: Signals, NgRx Store/SignalStore, Resource API, formulários reativos, @ngx-translate, date-fns e autenticação JWT com roles
- [Frontend-React.md](Frontend-React.md) — React 19 com TypeScript: React Router, Redux Toolkit, Zustand, TanStack Query, RTK Query, React Hook Form + Zod, react-i18next, date-fns e autenticação JWT com roles
- [Frontend-Vue.md](Frontend-Vue.md) — Vue 3.5 com TypeScript: Composition API, Pinia, Vue Router, VeeValidate + Zod, Vue I18n, TanStack Query, date-fns e autenticação JWT com roles

### Padrões equivalentes entre os frameworks

| Conceito | Angular 21 | React 19 | Vue 3.5 |
|---|---|---|---|
| Reatividade | Signals | `useState` / `useMemo` | `ref` / `reactive` / `computed` |
| Template | Template + diretivas | JSX | SFC `<template>` + diretivas |
| Props | `input()` | Desestruturação tipada | `defineProps` |
| Eventos | `output()` | `onClick` / callbacks | `defineEmits` |
| Two-way binding | `[(ngModel)]` | Estado + onChange | `defineModel` |
| Lógica reutilizável | `inject()` + serviços | Hooks (`use*`) | Composables (`use*`) |
| Estado global | NgRx / SignalStore | Redux Toolkit / Zustand | Pinia |
| Dados remotos | Resource API / NgRx Effects | TanStack Query / RTK Query | TanStack Query / useFetch |
| Formulários | ReactiveFormsModule | React Hook Form + Zod | VeeValidate + Zod |
| Roteamento | `@angular/router` | React Router 7 | Vue Router 4 |
| Lazy loading | `loadComponent` | `React.lazy` + Suspense | `defineAsyncComponent` / `loadComponent` |
| i18n | @ngx-translate | react-i18next | Vue I18n 9 |
| Guards de rota | `CanActivateFn` | Componente `<RotaProtegida>` | `beforeEach` / `onBeforeRouteLeave` |
| Interceptor HTTP | `HttpInterceptorFn` | Wrapper sobre fetch | Wrapper sobre fetch |

---

## Referências

- [MDN — JavaScript](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript)
- [MDN — DOM](https://developer.mozilla.org/pt-BR/docs/Web/API/Document_Object_Model)
- [MDN — Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API)
- [MDN — Promise](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript — tsconfig reference](https://www.typescriptlang.org/tsconfig)
- [Can I Use](https://caniuse.com) — suporte de recursos por navegador
