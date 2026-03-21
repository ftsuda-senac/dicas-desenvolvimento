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

```js
const pessoa = { nome: 'Ana', idade: 30, cidade: 'SP' };

// Acesso
pessoa.nome;           // "Ana"
pessoa['nome'];        // "Ana" — útil para chaves dinâmicas

// Métodos estáticos
Object.keys(pessoa);   // ["nome", "idade", "cidade"]
Object.values(pessoa); // ["Ana", 30, "SP"]
Object.entries(pessoa);// [["nome","Ana"], ["idade",30], ["cidade","SP"]]

// Copiar/mesclar objetos (shallow copy)
const copia = { ...pessoa };
const mesclado = { ...pessoa, profissao: 'Dev' };
const mesclado2 = Object.assign({}, pessoa, { profissao: 'Dev' });

// Verificar propriedade
'nome' in pessoa;               // true
pessoa.hasOwnProperty('nome'); // true

// Propriedade computada
const campo = 'email';
const contato = { [campo]: 'ana@email.com' }; // { email: "ana@email.com" }
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
| Mouse | `click`, `dblclick`, `mousedown`, `mouseup`, `mouseover`, `mouseout`, `mousemove`, `contextmenu` |
| Teclado | `keydown`, `keyup`, `keypress` (depreciado) |
| Formulário | `submit`, `change`, `input`, `focus`, `blur`, `reset` |
| Documento | `DOMContentLoaded`, `load`, `resize`, `scroll` |
| Drag | `dragstart`, `dragover`, `drop`, `dragend` |
| Touch | `touchstart`, `touchmove`, `touchend` |

```js
// event.preventDefault() — cancela o comportamento padrão
document.querySelector('form').addEventListener('submit', (e) => {
  e.preventDefault(); // impede o envio do formulário
  // processar dados com JS
});

// event.stopPropagation() — interrompe a propagação (bubbling)
btn.addEventListener('click', (e) => {
  e.stopPropagation();
});

// Delegação de eventos — um listener no pai captura eventos dos filhos
// Útil quando os filhos são criados dinamicamente
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.matches('li')) {
    console.log('Item clicado:', e.target.textContent);
  }
});
```

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
