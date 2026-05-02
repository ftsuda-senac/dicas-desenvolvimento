# Rust — Guia Prático com Foco em WebAssembly (WASM)

Rust é uma linguagem de programação de sistemas compilada, estaticamente tipada, criada pela Mozilla e tornada open-source em 2010, com a versão 1.0 estável lançada em 2015. Seu design prioriza **segurança de memória sem garbage collector, desempenho de nível C/C++ e concorrência sem data races**, tornando-a a escolha dominante para WebAssembly, sistemas embarcados, ferramentas de linha de comando e infraestrutura de alta performance.

Rust é, consecutivamente desde 2016, a linguagem "mais amada" no Stack Overflow Developer Survey — reflexo direto da sensação de segurança que o compilador oferece.

---

## Sumário

1. [Por que Rust?](#por-que-rust)
2. [Casos de Uso e Ecossistema](#casos-de-uso-e-ecossistema)
3. [Ambiente de Desenvolvimento](#ambiente-de-desenvolvimento)
4. [IDEs e Ferramentas](#ides-e-ferramentas)
5. [Estrutura de um Projeto Rust](#estrutura-de-um-projeto-rust)
6. [Fundamentos da Linguagem](#fundamentos-da-linguagem)
7. [Coleções de Dados](#coleções-de-dados)
8. [Ownership, Borrowing e Lifetimes](#ownership-borrowing-e-lifetimes)
9. [Structs, Enums e Pattern Matching](#structs-enums-e-pattern-matching)
10. [Tratamento de Erros](#tratamento-de-erros)
11. [Traits e Generics](#traits-e-generics)
12. [Concorrência](#concorrência)
13. [WebAssembly com Rust](#webassembly-com-rust)
14. [Projeto WASM: Calculadora Interativa](#projeto-wasm-calculadora-interativa)
15. [Projeto WASM: Processamento de Imagens no Browser](#projeto-wasm-processamento-de-imagens-no-browser)
16. [Projeto WASM: Comunicação com JavaScript](#projeto-wasm-comunicação-com-javascript)
17. [API REST com Rust](#api-rest-com-rust)
18. [Testes](#testes)
19. [Referências](#referências)

---

## Por que Rust?

| Característica | Detalhe |
|---|---|
| **Segurança de memória** | Sem null pointer dereferences, buffer overflows ou use-after-free — garantidos em tempo de compilação |
| **Zero-cost abstractions** | Iterators, closures e generics compilam para código tão eficiente quanto C manual |
| **Sem GC** | Ideal para WASM, sistemas embarcados e aplicações de tempo real |
| **WASM de primeira classe** | Toolchain oficial `wasm-pack`, `wasm-bindgen` e `web-sys` com suporte nativo |
| **Binários pequenos** | Módulos WASM gerados por Rust são menores e mais rápidos que equivalentes em C++ |
| **Concorrência segura** | O sistema de tipos impede data races em tempo de compilação |
| **Tooling excelente** | `cargo` unifica build, deps, testes, benchmarks, docs e publicação |

---

## Casos de Uso e Ecossistema

### WebAssembly (foco principal deste documento)

Rust é a linguagem mais recomendada para WASM por combinar desempenho máximo com binários enxutos e ausência de GC (fundamental para módulos WASM previsíveis).

| Uso | Exemplos |
|---|---|
| Processamento pesado no browser | Codecs de vídeo/áudio, algoritmos de IA, criptografia |
| Jogos | Engines 2D/3D rodando no browser sem plugins |
| Ferramentas web | Compiladores, formatadores, linters em WASM |
| Edge Computing | Cloudflare Workers, Fastly Compute |
| Plugins seguros | Extender aplicações de forma isolada (WASI) |

### Outros Casos de Uso

- **Sistemas operacionais**: Redox OS (escrito integralmente em Rust)
- **Browsers**: Componentes do Firefox, Servo (engine experimental da Mozilla)
- **Infraestrutura**: Cloudflare, Discord (substituíram Go por Rust em serviços críticos)
- **Blockchain**: Solana, Polkadot, Near Protocol
- **CLIs de alta performance**: `ripgrep` (busca de texto), `bat` (clone do `cat`), `exa`/`eza`
- **Embarcados**: Microcontroladores sem OS (bare-metal)

---

## Ambiente de Desenvolvimento

### Instalação do Rust (todas as plataformas)

A forma oficial e recomendada é via **`rustup`** — gerenciador de toolchains que permite instalar múltiplas versões e targets.

```bash
# Linux / macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Windows — baixar e executar: https://rustup.rs
# Ou via winget:
winget install Rustlang.Rustup
```

Após a instalação, reinicie o terminal e verifique:

```bash
rustc --version    # ex: rustc 1.78.0 (9b00956e5 2024-04-29)
cargo --version    # ex: cargo 1.78.0 (54d8815d0 2024-03-26)
rustup --version   # ex: rustup 1.27.0 (bbb9276d2 2024-03-08)
```

### Componentes Essenciais

```bash
# Formatador de código (obrigatório)
rustup component add rustfmt

# Linter (altamente recomendado)
rustup component add clippy

# Servidor de linguagem para IDEs
rustup component add rust-analyzer

# Documentação offline
rustup component add rust-docs
```

### Ferramentas para WASM

```bash
# Instalar wasm-pack (build, test e publish de crates WASM)
cargo install wasm-pack

# Instalar wasm-bindgen CLI (geração de bindings JS)
cargo install wasm-bindgen-cli

# Adicionar target WASM ao Rust
rustup target add wasm32-unknown-unknown

# Target WASI (para ambientes server-side WASM)
rustup target add wasm32-wasip1

# Ferramenta para otimizar binários WASM
cargo install wasm-opt   # wrapper do binaryen

# Servidor de desenvolvimento web simples
cargo install miniserve
# ou
npm install -g serve
```

### Verificar instalação WASM

```bash
# Compilar "hello world" para WASM
echo 'fn main() { println!("Hello, WASM!"); }' > /tmp/test.rs
rustc --target wasm32-unknown-unknown /tmp/test.rs -o /tmp/test.wasm
file /tmp/test.wasm  # deve mostrar: WebAssembly (wasm) binary module
```

---

## IDEs e Ferramentas

### VS Code (recomendado para iniciantes)

**Extensão principal:**
- **rust-analyzer** (`rust-lang.rust-analyzer`) — LSP oficial: autocomplete, go-to-definition, inline errors, refactoring

**Extensões complementares:**
- **CodeLLDB** (`vadimcn.vscode-lldb`) — debugger nativo para Rust
- **Even Better TOML** (`tamasfe.even-better-toml`) — syntax highlighting para `Cargo.toml`
- **crates** (`serayuzgur.crates`) — mostra versões disponíveis no `Cargo.toml`

```json
// .vscode/settings.json recomendado para projetos Rust
{
  "rust-analyzer.checkOnSave.command": "clippy",
  "rust-analyzer.cargo.features": "all",
  "editor.formatOnSave": true,
  "[rust]": {
    "editor.defaultFormatter": "rust-lang.rust-analyzer"
  }
}
```

### RustRover (JetBrains)

IDE dedicada exclusivamente a Rust, lançada em 2024. Oferece a mesma qualidade de ferramentas dos outros produtos JetBrains (IntelliJ IDEA, CLion).

- Análise semântica completa, refactoring inteligente
- Debugger integrado (LLDB/GDB)
- Profiler de CPU e memória
- Suporte nativo a `cargo`, `clippy`, `rustfmt`
- Gratuito para uso não-comercial

### Outras Opções

| IDE/Editor | Plugin | Adequação |
|---|---|---|
| **Neovim** | `nvim-lspconfig` + rust-analyzer | Avançado, muito customizável |
| **Helix** | nativo (rust-analyzer integrado) | Terminal, moderno, zero config |
| **Zed** | suporte nativo a Rust | Rápido, colaborativo |
| **CLion** | Plugin Rust (JetBrains) | Para usuários C/C++ JetBrains |

---

## Estrutura de um Projeto Rust

```bash
# Criar novo projeto binário
cargo new meu-projeto

# Criar nova biblioteca (crate)
cargo new --lib minha-lib

# Estrutura gerada:
meu-projeto/
├── Cargo.toml      # Manifesto: dependências, metadata, features
├── Cargo.lock      # Lock file (commitar em binários, ignorar em libs)
└── src/
    └── main.rs     # Ponto de entrada (ou lib.rs para libraries)
```

### Cargo.toml

```toml
[package]
name = "meu-projeto"
version = "0.1.0"
edition = "2021"          # Edição do Rust (2015, 2018, 2021)
authors = ["Seu Nome <email@exemplo.com>"]
description = "Descrição do projeto"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

[dev-dependencies]         # Apenas para testes
pretty_assertions = "1.4"

[profile.release]          # Otimizações para produção
opt-level = 3
lto = true
codegen-units = 1
```

### Comandos Cargo Essenciais

```bash
cargo build           # Compilar em modo debug
cargo build --release # Compilar otimizado para produção
cargo run             # Compilar e executar
cargo run -- arg1     # Passar argumentos ao programa
cargo test            # Executar todos os testes
cargo test nome_teste # Executar teste específico
cargo check           # Verificar erros sem gerar binário (mais rápido)
cargo clippy          # Linter (verificação de qualidade)
cargo fmt             # Formatar todo o código
cargo doc --open      # Gerar e abrir documentação
cargo add serde       # Adicionar dependência (cargo-add)
cargo update          # Atualizar dependências
cargo clean           # Limpar artefatos de build
```

---

## Fundamentos da Linguagem

### Variáveis e Tipos

```rust
fn main() {
    // Imutável por padrão
    let x = 5;
    // x = 6; // ERRO: cannot assign twice to immutable variable

    // Mutável com mut
    let mut contador = 0;
    contador += 1;

    // Shadowing — redeclarar com mesmo nome
    let x = x * 2;          // x agora é 10
    let x = x.to_string();  // x agora é String "10"
    println!("x = {x}");

    // Tipos escalares
    let inteiro: i32 = -42;
    let inteiro_sem_sinal: u64 = 1_000_000; // _ como separador visual
    let ponto_flutuante: f64 = 3.14159;
    let booleano: bool = true;
    let caractere: char = 'ã'; // char é Unicode (4 bytes)

    // Inferência de tipos
    let inferido = 3.14; // f64 inferido
    let inferido_int = 42; // i32 inferido

    // Constantes — tipo obrigatório, sempre imutáveis, escopo global possível
    const MAX_PONTOS: u32 = 100_000;
    println!("Máximo: {MAX_PONTOS}");

    // Tuplas — tipos heterogêneos, tamanho fixo
    let tupla: (i32, f64, &str) = (42, 3.14, "olá");
    let (a, b, c) = tupla; // destructuring
    println!("a={a}, b={b}, c={c}");
    println!("primeiro: {}", tupla.0);

    // Arrays — tipo homogêneo, tamanho fixo em tempo de compilação
    let arr: [i32; 5] = [1, 2, 3, 4, 5];
    let arr_repetido = [0; 10]; // [0, 0, 0, ..., 0] com 10 elementos
    println!("arr[2] = {}", arr[2]);
}
```

### Condicionais

```rust
fn classificar_nota(nota: u32) -> &'static str {
    // if é uma expressão — pode retornar valor
    if nota >= 90 {
        "Excelente"
    } else if nota >= 70 {
        "Bom"
    } else if nota >= 50 {
        "Regular"
    } else {
        "Insuficiente"
    }
}

fn verificar_numero(n: i32) {
    // if-let: desempacotar Option/Result
    let opcional: Option<i32> = Some(42);
    if let Some(valor) = opcional {
        println!("Valor encontrado: {valor}");
    }

    // Atribuição com if
    let descricao = if n > 0 { "positivo" } else if n < 0 { "negativo" } else { "zero" };
    println!("{n} é {descricao}");
}

fn main() {
    println!("{}", classificar_nota(85)); // Bom
    verificar_numero(-5);
}
```

### Loops

```rust
fn exemplos_loops() {
    // loop — laço infinito com break para retornar valor
    let mut contador = 0;
    let resultado = loop {
        contador += 1;
        if contador == 10 {
            break contador * 2; // retorna 20
        }
    };
    println!("Resultado do loop: {resultado}");

    // while
    let mut n = 1;
    while n < 100 {
        n *= 2;
    }
    println!("Primeira potência de 2 >= 100: {n}");

    // for — iterador (idiomático em Rust)
    for i in 0..5 {        // range exclusivo: 0, 1, 2, 3, 4
        print!("{i} ");
    }
    println!();

    for i in 0..=5 {       // range inclusivo: 0, 1, 2, 3, 4, 5
        print!("{i} ");
    }
    println!();

    // for sobre coleção
    let frutas = ["maçã", "banana", "laranja"];
    for fruta in &frutas {
        println!("Fruta: {fruta}");
    }

    // enumerate — índice + valor
    for (i, fruta) in frutas.iter().enumerate() {
        println!("[{i}] {fruta}");
    }

    // Labels para loops aninhados
    'externo: for x in 0..5 {
        for y in 0..5 {
            if x == 2 && y == 2 {
                break 'externo; // sai do loop externo
            }
            print!("({x},{y}) ");
        }
    }
    println!("\nFim dos loops aninhados");
}

fn main() {
    exemplos_loops();
}
```

### Funções

```rust
// Tipo de retorno após ->
fn somar(a: i32, b: i32) -> i32 {
    a + b  // sem ; => expressão de retorno implícita
}

// Múltiplos retornos via tupla
fn dividir(dividendo: f64, divisor: f64) -> (f64, bool) {
    if divisor == 0.0 {
        return (0.0, false); // return explícito também funciona
    }
    (dividendo / divisor, true)
}

// Closures — funções anônimas que capturam o ambiente
fn aplicar<F: Fn(i32) -> i32>(f: F, valor: i32) -> i32 {
    f(valor)
}

fn main() {
    println!("{}", somar(3, 4)); // 7

    let (resultado, sucesso) = dividir(10.0, 3.0);
    if sucesso {
        println!("Resultado: {resultado:.2}"); // 3.33
    }

    // Closures capturam variáveis do escopo externo
    let fator = 3;
    let triplicar = |x| x * fator;
    println!("{}", aplicar(triplicar, 5)); // 15

    // Closures inline
    let dobrar = |x: i32| -> i32 { x * 2 };
    let numeros: Vec<i32> = (1..=5).map(dobrar).collect();
    println!("{numeros:?}"); // [2, 4, 6, 8, 10]
}
```

---

## Coleções de Dados

### Vec (Vetor Dinâmico)

```rust
fn exemplos_vec() {
    // Criação
    let mut v: Vec<i32> = Vec::new();
    let v2 = vec![1, 2, 3, 4, 5]; // macro vec!

    // Adicionar e remover
    v.push(10);
    v.push(20);
    v.push(30);
    let ultimo = v.pop(); // Some(30)
    println!("Removido: {ultimo:?}");

    // Acesso — dois modos
    let terceiro = &v2[2];       // panic se out of bounds
    let seguro = v2.get(2);      // retorna Option<&i32>
    match seguro {
        Some(val) => println!("Terceiro: {val}"),
        None      => println!("Índice fora do limite"),
    }

    // Iteração
    let soma: i32 = v2.iter().sum();
    let dobrados: Vec<i32> = v2.iter().map(|x| x * 2).collect();
    let pares: Vec<&i32> = v2.iter().filter(|&&x| x % 2 == 0).collect();

    println!("Soma: {soma}");
    println!("Dobrados: {dobrados:?}");
    println!("Pares: {pares:?}");

    // Ordenação
    let mut desordenado = vec![3, 1, 4, 1, 5, 9, 2, 6];
    desordenado.sort();
    desordenado.dedup(); // remove duplicatas adjacentes
    println!("Ordenado e sem duplicatas: {desordenado:?}");
}
```

### HashMap

```rust
use std::collections::HashMap;

fn exemplos_hashmap() {
    let mut estoque: HashMap<String, u32> = HashMap::new();

    // Inserir
    estoque.insert("maçã".to_string(), 50);
    estoque.insert("banana".to_string(), 30);
    estoque.insert("laranja".to_string(), 20);

    // Ler
    if let Some(qtd) = estoque.get("maçã") {
        println!("Maçãs: {qtd}");
    }

    // entry API — inserir apenas se ausente
    estoque.entry("uva".to_string()).or_insert(15);
    estoque.entry("maçã".to_string()).or_insert(999); // não altera — já existe

    // Atualizar baseado no valor anterior
    let count = estoque.entry("banana".to_string()).or_insert(0);
    *count += 10; // banana agora é 40

    // Iterar
    let mut frutas: Vec<(&String, &u32)> = estoque.iter().collect();
    frutas.sort_by_key(|&(k, _)| k);
    for (fruta, qtd) in &frutas {
        println!("{fruta}: {qtd}");
    }

    // Verificar existência
    println!("Tem manga? {}", estoque.contains_key("manga"));
    println!("Total de itens: {}", estoque.len());
}
```

### HashSet e BTreeMap

```rust
use std::collections::{HashSet, BTreeMap};

fn exemplos_set_btree() {
    // HashSet — sem duplicatas, sem ordem garantida
    let mut tags: HashSet<&str> = HashSet::new();
    tags.insert("rust");
    tags.insert("wasm");
    tags.insert("rust"); // ignorado — já existe
    println!("Tags: {tags:?}");

    let a: HashSet<i32> = [1, 2, 3, 4].iter().cloned().collect();
    let b: HashSet<i32> = [3, 4, 5, 6].iter().cloned().collect();
    let intersecao: HashSet<&i32> = a.intersection(&b).collect();
    let uniao: HashSet<&i32> = a.union(&b).collect();
    println!("Interseção: {intersecao:?}");
    println!("União: {uniao:?}");

    // BTreeMap — HashMap ordenado por chave (útil para iteração ordenada)
    let mut ranking: BTreeMap<u32, &str> = BTreeMap::new();
    ranking.insert(3, "bronze");
    ranking.insert(1, "ouro");
    ranking.insert(2, "prata");
    for (pos, medalha) in &ranking {
        println!("{pos}º lugar: {medalha}");
    }
}
```

### Strings

```rust
fn exemplos_strings() {
    // &str — string slice, imutável, referência
    let s1: &str = "Olá, mundo!";

    // String — string heap-allocated, mutável
    let mut s2 = String::from("Olá");
    s2.push_str(", mundo!"); // append string
    s2.push('!');            // append char

    // Concatenação
    let s3 = String::from("Hello");
    let s4 = String::from(" World");
    let s5 = s3 + &s4; // s3 é movido aqui, s4 ainda válido

    // format! — preferível para múltiplas strings (não consome)
    let nome = "Rust";
    let versao = "1.78";
    let info = format!("{nome} v{versao}");

    // Métodos úteis
    let texto = "  Rust é incrível!  ";
    println!("{}", texto.trim());
    println!("{}", texto.trim().to_uppercase());
    println!("{}", "olá mundo".replace("mundo", "Rust"));
    println!("{}", "a,b,c".split(',').collect::<Vec<&str>>().join(" | "));

    // Iterar sobre caracteres (correto para Unicode)
    for c in "Olá".chars() {
        print!("{c} ");
    }
    println!();

    // Verificações
    let url = "https://rust-lang.org";
    println!("É HTTPS? {}", url.starts_with("https"));
    println!("Contém 'rust'? {}", url.contains("rust"));
    println!("Tamanho em bytes: {}", url.len());
    println!("Número de chars: {}", url.chars().count());
}
```

---

## Ownership, Borrowing e Lifetimes

O sistema de ownership é o diferencial central de Rust — substitui GC e malloc/free por regras verificadas em tempo de compilação.

### Regras de Ownership

```rust
fn ownership_basico() {
    // Cada valor tem exatamente um owner
    let s1 = String::from("olá");
    let s2 = s1;    // s1 é MOVIDO para s2
    // println!("{s1}"); // ERRO: value borrowed here after move

    // Clone — cópia profunda explícita
    let s3 = String::from("olá");
    let s4 = s3.clone(); // cópia independente
    println!("{s3} e {s4}"); // ambos válidos

    // Tipos Copy (stack-allocated) são copiados automaticamente
    let x = 5;
    let y = x; // x é copiado, não movido
    println!("{x} e {y}"); // ambos válidos

    // Funções tomam ownership
    let s = String::from("teste");
    recebe_ownership(s);
    // println!("{s}"); // ERRO: s foi movido para a função

    // Funções podem retornar ownership
    let s = retorna_ownership();
    println!("{s}");
}

fn recebe_ownership(s: String) {
    println!("Recebi: {s}");
    // s é dropado aqui
}

fn retorna_ownership() -> String {
    String::from("novo valor")
}
```

### Borrowing (Referências)

```rust
fn borrowing() {
    let s = String::from("olá mundo");

    // Referência imutável — &T
    let len = calcular_tamanho(&s); // empresta s
    println!("{s} tem {len} chars");

    // Múltiplas referências imutáveis simultâneas: OK
    let r1 = &s;
    let r2 = &s;
    println!("{r1} e {r2}");

    // Referência mutável — &mut T
    let mut s2 = String::from("olá");
    adicionar_mundo(&mut s2);
    println!("{s2}");

    // REGRA: só uma referência mutável por vez no mesmo escopo
    let r3 = &mut s2;
    // let r4 = &mut s2; // ERRO: cannot borrow `s2` as mutable more than once
    println!("{r3}");
}

fn calcular_tamanho(s: &String) -> usize {
    s.len() // não toma ownership
}

fn adicionar_mundo(s: &mut String) {
    s.push_str(", mundo!");
}
```

### Lifetimes

```rust
// Lifetimes garantem que referências são sempre válidas
// O compilador infere na maioria dos casos

// Lifetime explícita necessária quando retornamos referência
fn maior<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// Struct com referência precisa de lifetime annotation
struct Trecho<'a> {
    parte: &'a str,
}

impl<'a> Trecho<'a> {
    fn nivel(&self) -> &str {
        self.parte
    }
}

fn main() {
    let s1 = String::from("longa string");
    let resultado;
    {
        let s2 = String::from("xyz");
        resultado = maior(s1.as_str(), s2.as_str());
        println!("Maior: {resultado}");
    }

    let texto = String::from("primeira linha. segunda linha.");
    let trecho = Trecho {
        parte: texto.split('.').next().expect("deve ter '.'"),
    };
    println!("Trecho: {}", trecho.nivel());
}
```

---

## Structs, Enums e Pattern Matching

### Structs

```rust
#[derive(Debug, Clone)]
struct Ponto {
    x: f64,
    y: f64,
}

impl Ponto {
    // Método associado (construtor por convenção)
    fn novo(x: f64, y: f64) -> Self {
        Ponto { x, y }
    }

    fn origem() -> Self {
        Ponto { x: 0.0, y: 0.0 }
    }

    // Método de instância
    fn distancia(&self, outro: &Ponto) -> f64 {
        ((self.x - outro.x).powi(2) + (self.y - outro.y).powi(2)).sqrt()
    }

    fn mover(&mut self, dx: f64, dy: f64) {
        self.x += dx;
        self.y += dy;
    }
}

#[derive(Debug)]
struct Retangulo {
    largura: f64,
    altura: f64,
}

impl Retangulo {
    fn area(&self) -> f64 {
        self.largura * self.altura
    }

    fn perimetro(&self) -> f64 {
        2.0 * (self.largura + self.altura)
    }

    fn e_quadrado(&self) -> bool {
        (self.largura - self.altura).abs() < f64::EPSILON
    }
}

fn main() {
    let mut p1 = Ponto::novo(3.0, 4.0);
    let p2 = Ponto::origem();
    println!("Distância: {:.2}", p1.distancia(&p2)); // 5.00
    p1.mover(1.0, 1.0);
    println!("Nova posição: {:?}", p1);

    let rect = Retangulo { largura: 10.0, altura: 5.0 };
    println!("Área: {}, Perímetro: {}", rect.area(), rect.perimetro());
}
```

### Enums e Pattern Matching

```rust
#[derive(Debug)]
enum Forma {
    Circulo(f64),                    // raio
    Retangulo { largura: f64, altura: f64 },
    Triangulo(f64, f64, f64),        // três lados
}

impl Forma {
    fn area(&self) -> f64 {
        match self {
            Forma::Circulo(r) => std::f64::consts::PI * r * r,
            Forma::Retangulo { largura, altura } => largura * altura,
            Forma::Triangulo(a, b, c) => {
                // Fórmula de Heron
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }

    fn nome(&self) -> &str {
        match self {
            Forma::Circulo(_)      => "Círculo",
            Forma::Retangulo {..}  => "Retângulo",
            Forma::Triangulo(..)   => "Triângulo",
        }
    }
}

// Option e Result são enums da stdlib
fn buscar_usuario(id: u32) -> Option<String> {
    match id {
        1 => Some("Alice".to_string()),
        2 => Some("Bob".to_string()),
        _ => None,
    }
}

fn main() {
    let formas = vec![
        Forma::Circulo(5.0),
        Forma::Retangulo { largura: 4.0, altura: 6.0 },
        Forma::Triangulo(3.0, 4.0, 5.0),
    ];

    for forma in &formas {
        println!("{}: área = {:.2}", forma.nome(), forma.area());
    }

    // Pattern matching com Option
    match buscar_usuario(1) {
        Some(nome) => println!("Usuário: {nome}"),
        None       => println!("Não encontrado"),
    }

    // Métodos de Option
    let nome = buscar_usuario(99).unwrap_or_else(|| "Desconhecido".to_string());
    println!("Nome: {nome}");

    // if let — pattern matching simples
    if let Some(usuario) = buscar_usuario(2) {
        println!("Encontrou: {usuario}");
    }

    // while let
    let mut pilha = vec![1, 2, 3];
    while let Some(topo) = pilha.pop() {
        println!("Removido: {topo}");
    }
}
```

---

## Tratamento de Erros

```rust
use std::fs;
use std::num::ParseIntError;

// Enum de erro customizado
#[derive(Debug)]
enum ErroApp {
    Io(std::io::Error),
    Parsing(ParseIntError),
    Negativo,
}

impl std::fmt::Display for ErroApp {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ErroApp::Io(e)      => write!(f, "Erro de I/O: {e}"),
            ErroApp::Parsing(e) => write!(f, "Erro de parsing: {e}"),
            ErroApp::Negativo   => write!(f, "Valor não pode ser negativo"),
        }
    }
}

impl From<std::io::Error> for ErroApp {
    fn from(e: std::io::Error) -> Self { ErroApp::Io(e) }
}

impl From<ParseIntError> for ErroApp {
    fn from(e: ParseIntError) -> Self { ErroApp::Parsing(e) }
}

// ? propaga erros automaticamente (requer impl From)
fn ler_numero_do_arquivo(caminho: &str) -> Result<i32, ErroApp> {
    let conteudo = fs::read_to_string(caminho)?; // propaga std::io::Error → ErroApp::Io
    let numero: i32 = conteudo.trim().parse()?;  // propaga ParseIntError → ErroApp::Parsing
    if numero < 0 {
        return Err(ErroApp::Negativo);
    }
    Ok(numero)
}

fn main() {
    match ler_numero_do_arquivo("numero.txt") {
        Ok(n)  => println!("Número lido: {n}"),
        Err(e) => eprintln!("Falha: {e}"),
    }

    // unwrap() — panic em Err (use só em prototipagem/testes)
    // let n = "42".parse::<i32>().unwrap();

    // expect() — panic com mensagem customizada
    // let n = "42".parse::<i32>().expect("Deveria ser número");

    // Encadeamento com map, and_then
    let resultado = "42"
        .parse::<i32>()
        .map(|n| n * 2)
        .map_err(|e| format!("Falha: {e}"));
    println!("{resultado:?}"); // Ok(84)

    // Coletar Results em um Vec
    let strings = vec!["1", "2", "abc", "4"];
    let numeros: Result<Vec<i32>, _> = strings.iter()
        .map(|s| s.parse::<i32>())
        .collect(); // Err se qualquer um falhar
    println!("{numeros:?}");
}
```

---

## Traits e Generics

```rust
// Trait — define comportamento compartilhado (similar a interface)
trait Resumivel {
    fn resumo(&self) -> String;

    // Método com implementação padrão
    fn autor(&self) -> String {
        String::from("(Autor desconhecido)")
    }
}

struct Artigo {
    titulo: String,
    autor: String,
    conteudo: String,
}

struct Tweet {
    usuario: String,
    texto: String,
}

impl Resumivel for Artigo {
    fn resumo(&self) -> String {
        format!("{} — por {} ({}...)", self.titulo, self.autor, &self.conteudo[..50.min(self.conteudo.len())])
    }

    fn autor(&self) -> String {
        self.autor.clone()
    }
}

impl Resumivel for Tweet {
    fn resumo(&self) -> String {
        format!("{}: {}", self.usuario, self.texto)
    }
}

// Generics com trait bounds
fn notificar(item: &impl Resumivel) {
    println!("Nova publicação: {}", item.resumo());
}

// Sintaxe alternativa com where
fn notificar_dois<T, U>(t: &T, u: &U)
where
    T: Resumivel + std::fmt::Debug,
    U: Resumivel,
{
    println!("{:?}: {}", t, t.resumo()); // requer Debug
    println!("{}", u.resumo());
}

// Função genérica — funciona com qualquer tipo que implemente PartialOrd
fn encontrar_maior<T: PartialOrd>(lista: &[T]) -> &T {
    let mut maior = &lista[0];
    for item in lista {
        if item > maior {
            maior = item;
        }
    }
    maior
}

fn main() {
    let tweet = Tweet {
        usuario: "rustacean".to_string(),
        texto: "Rust é incrível para WASM! 🦀".to_string(),
    };
    notificar(&tweet);

    let numeros = vec![34, 50, 25, 100, 65];
    println!("Maior: {}", encontrar_maior(&numeros));

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("Maior char: {}", encontrar_maior(&chars));
}
```

---

## Concorrência

```rust
use std::thread;
use std::sync::{Arc, Mutex};
use std::sync::mpsc;

fn concorrencia_basica() {
    // Spawn de thread
    let handle = thread::spawn(|| {
        for i in 1..=5 {
            println!("Thread filha: {i}");
        }
    });

    for i in 1..=3 {
        println!("Thread principal: {i}");
    }

    handle.join().unwrap(); // aguarda thread terminar
}

fn compartilhar_dados() {
    // Arc — contagem de referências atômica (thread-safe)
    // Mutex — exclusão mútua para acesso exclusivo
    let contador = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let contador = Arc::clone(&contador);
        let handle = thread::spawn(move || {
            let mut num = contador.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Contador final: {}", *contador.lock().unwrap()); // 10
}

fn canal_mensagens() {
    // mpsc — multiple producer, single consumer
    let (tx, rx) = mpsc::channel();
    let tx2 = tx.clone(); // múltiplos produtores

    thread::spawn(move || {
        tx.send("mensagem 1").unwrap();
        tx.send("mensagem 2").unwrap();
    });

    thread::spawn(move || {
        tx2.send("mensagem 3").unwrap();
    });

    for msg in rx {
        println!("Recebido: {msg}");
    }
}

fn main() {
    concorrencia_basica();
    compartilhar_dados();
    canal_mensagens();
}
```

---

## WebAssembly com Rust

### Conceitos Fundamentais

WebAssembly (WASM) é um formato binário portável que roda em browsers e ambientes server-side com desempenho próximo ao nativo. Rust é a linguagem mais adequada para WASM pelos seguintes motivos:

| Aspecto | Detalhe |
|---|---|
| **Sem GC** | Módulos WASM não precisam embutir um garbage collector — binários menores |
| **Desempenho previsível** | Sem pausas de GC — crítico para jogos, áudio e processamento em tempo real |
| **Segurança** | Garantias de memória de Rust se mantêm dentro do sandbox WASM |
| **Tooling maduro** | `wasm-pack`, `wasm-bindgen`, `web-sys`, `js-sys` oficialmente suportados |
| **Interoperabilidade** | `wasm-bindgen` gera bindings TypeScript/JavaScript automaticamente |

### Ferramentas do Ecossistema WASM

| Ferramenta | Papel |
|---|---|
| `wasm-pack` | Build, test e publish de crates para npm |
| `wasm-bindgen` | Geração de bindings JS/TS — expõe structs, funções, callbacks |
| `web-sys` | Bindings para todas as APIs do browser (DOM, fetch, canvas, etc.) |
| `js-sys` | Bindings para tipos JavaScript primitivos (Array, Promise, Date) |
| `gloo` | Abstração idiomática sobre `web-sys` |
| `wasm-opt` | Otimizador de binários WASM (reduz tamanho em 10-30%) |
| `Trunk` | Bundler para apps Rust+WASM (similar ao Vite) |

### Configuração Mínima de um Projeto WASM

```bash
# Criar projeto library
cargo new --lib minha-lib-wasm
cd minha-lib-wasm
```

```toml
# Cargo.toml
[package]
name = "minha-lib-wasm"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]
# cdylib — gera .wasm para consumo externo
# rlib  — permite uso como crate Rust normal (testes)

[dependencies]
wasm-bindgen = "0.2"

[profile.release]
opt-level = "s"    # otimiza para tamanho
lto = true
```

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;

// #[wasm_bindgen] expõe ao JavaScript
#[wasm_bindgen]
pub fn saudacao(nome: &str) -> String {
    format!("Olá, {nome}! Escrito em Rust 🦀")
}

#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

```bash
# Compilar para WASM com geração automática de bindings JS
wasm-pack build --target web

# Saída gerada em pkg/:
# minha-lib-wasm_bg.wasm   — módulo WASM compilado
# minha-lib-wasm.js        — glue code JavaScript
# minha-lib-wasm.d.ts      — tipos TypeScript gerados automaticamente
# package.json             — manifesto npm
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>Rust WASM Demo</title></head>
<body>
  <script type="module">
    import init, { saudacao, fibonacci } from './pkg/minha-lib-wasm.js';

    async function main() {
      await init(); // carrega e inicializa o módulo WASM
      console.log(saudacao("mundo"));       // Olá, mundo! Escrito em Rust 🦀
      console.log(fibonacci(10));           // 55
    }

    main();
  </script>
</body>
</html>
```

---

## Projeto WASM: Calculadora Interativa

```bash
cargo new --lib calculadora-wasm
cd calculadora-wasm
```

```toml
# Cargo.toml
[package]
name = "calculadora-wasm"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen = "0.2"
serde = { version = "1.0", features = ["derive"] }
serde-wasm-bindgen = "0.6"
```

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct ResultadoCalculo {
    pub valor: f64,
    pub operacao: String,
    pub sucesso: bool,
    pub erro: Option<String>,
}

#[wasm_bindgen]
pub struct Calculadora {
    historico: Vec<f64>,
    acumulador: f64,
}

#[wasm_bindgen]
impl Calculadora {
    #[wasm_bindgen(constructor)]
    pub fn new() -> Self {
        Calculadora {
            historico: Vec::new(),
            acumulador: 0.0,
        }
    }

    pub fn somar(&mut self, a: f64, b: f64) -> JsValue {
        let resultado = a + b;
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("{a} + {b}"), None)
    }

    pub fn subtrair(&mut self, a: f64, b: f64) -> JsValue {
        let resultado = a - b;
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("{a} - {b}"), None)
    }

    pub fn multiplicar(&mut self, a: f64, b: f64) -> JsValue {
        let resultado = a * b;
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("{a} × {b}"), None)
    }

    pub fn dividir(&mut self, a: f64, b: f64) -> JsValue {
        if b == 0.0 {
            return self.serializar_resultado(
                0.0,
                format!("{a} ÷ {b}"),
                Some("Divisão por zero".to_string()),
            );
        }
        let resultado = a / b;
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("{a} ÷ {b}"), None)
    }

    pub fn raiz_quadrada(&mut self, n: f64) -> JsValue {
        if n < 0.0 {
            return self.serializar_resultado(
                0.0,
                format!("√{n}"),
                Some("Raiz de número negativo".to_string()),
            );
        }
        let resultado = n.sqrt();
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("√{n}"), None)
    }

    pub fn potencia(&mut self, base: f64, exp: f64) -> JsValue {
        let resultado = base.powf(exp);
        self.registrar(resultado);
        self.serializar_resultado(resultado, format!("{base}^{exp}"), None)
    }

    pub fn historico_como_json(&self) -> JsValue {
        serde_wasm_bindgen::to_value(&self.historico).unwrap_or(JsValue::NULL)
    }

    pub fn media_historico(&self) -> f64 {
        if self.historico.is_empty() {
            return 0.0;
        }
        self.historico.iter().sum::<f64>() / self.historico.len() as f64
    }

    pub fn limpar_historico(&mut self) {
        self.historico.clear();
        self.acumulador = 0.0;
    }

    fn registrar(&mut self, valor: f64) {
        self.historico.push(valor);
        self.acumulador = valor;
    }

    fn serializar_resultado(&self, valor: f64, operacao: String, erro: Option<String>) -> JsValue {
        let resultado = ResultadoCalculo {
            valor,
            operacao,
            sucesso: erro.is_none(),
            erro,
        };
        serde_wasm_bindgen::to_value(&resultado).unwrap_or(JsValue::NULL)
    }
}
```

```bash
wasm-pack build --target web
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <title>Calculadora Rust WASM</title>
  <style>
    body { font-family: sans-serif; max-width: 400px; margin: 2rem auto; }
    input { width: 100px; padding: 4px; }
    button { margin: 4px; padding: 8px 16px; cursor: pointer; }
    #resultado { font-size: 1.5rem; font-weight: bold; margin: 1rem 0; }
    #historico { background: #f0f0f0; padding: 1rem; border-radius: 4px; }
  </style>
</head>
<body>
  <h1>Calculadora Rust 🦀</h1>
  <div>
    <input type="number" id="a" value="10" placeholder="A">
    <input type="number" id="b" value="3" placeholder="B">
  </div>
  <div>
    <button onclick="operar('somar')">+</button>
    <button onclick="operar('subtrair')">-</button>
    <button onclick="operar('multiplicar')">×</button>
    <button onclick="operar('dividir')">÷</button>
    <button onclick="operar('potencia')">^</button>
    <button onclick="operar('raiz_quadrada')">√A</button>
  </div>
  <div id="resultado">—</div>
  <div id="historico">Histórico: —</div>
  <button onclick="limpar()">Limpar Histórico</button>

  <script type="module">
    import init, { Calculadora } from './pkg/calculadora_wasm.js';

    let calc;

    await init();
    calc = new Calculadora();

    window.operar = function(operacao) {
      const a = parseFloat(document.getElementById('a').value);
      const b = parseFloat(document.getElementById('b').value);

      let res;
      if (operacao === 'raiz_quadrada') {
        res = calc.raiz_quadrada(a);
      } else {
        res = calc[operacao](a, b);
      }

      const el = document.getElementById('resultado');
      if (res.sucesso) {
        el.textContent = `${res.operacao} = ${res.valor.toFixed(4)}`;
        el.style.color = 'green';
      } else {
        el.textContent = `Erro: ${res.erro}`;
        el.style.color = 'red';
      }

      const hist = calc.historico_como_json();
      const media = calc.media_historico();
      document.getElementById('historico').innerHTML =
        `Histórico: [${hist.map(v => v.toFixed(2)).join(', ')}]<br>
         Média: ${media.toFixed(4)}`;
    };

    window.limpar = function() {
      calc.limpar_historico();
      document.getElementById('resultado').textContent = '—';
      document.getElementById('historico').textContent = 'Histórico: —';
    };
  </script>
</body>
</html>
```

---

## Projeto WASM: Processamento de Imagens no Browser

Este exemplo demonstra o poder do WASM: processar pixels de imagem com desempenho nativo, algo que seria muito lento em JavaScript puro.

```toml
# Cargo.toml
[dependencies]
wasm-bindgen = "0.2"
js-sys = "0.3"
web-sys = { version = "0.3", features = [
  "CanvasRenderingContext2d",
  "ImageData",
  "HtmlCanvasElement",
  "Window",
  "Document",
]}
```

```rust
// src/lib.rs
use wasm_bindgen::prelude::*;

// Processar array de pixels RGBA diretamente
// Cada pixel = 4 bytes: [R, G, B, A, R, G, B, A, ...]
#[wasm_bindgen]
pub fn aplicar_escala_cinza(dados: &mut [u8]) {
    for pixel in dados.chunks_mut(4) {
        // Luminância perceptual (pesos ITU-R BT.709)
        let cinza = (0.2126 * pixel[0] as f64
                   + 0.7152 * pixel[1] as f64
                   + 0.0722 * pixel[2] as f64) as u8;
        pixel[0] = cinza;
        pixel[1] = cinza;
        pixel[2] = cinza;
        // pixel[3] = alpha, não alterado
    }
}

#[wasm_bindgen]
pub fn aplicar_negativo(dados: &mut [u8]) {
    for pixel in dados.chunks_mut(4) {
        pixel[0] = 255 - pixel[0];
        pixel[1] = 255 - pixel[1];
        pixel[2] = 255 - pixel[2];
    }
}

#[wasm_bindgen]
pub fn aplicar_sepia(dados: &mut [u8]) {
    for pixel in dados.chunks_mut(4) {
        let r = pixel[0] as f64;
        let g = pixel[1] as f64;
        let b = pixel[2] as f64;

        pixel[0] = ((r * 0.393) + (g * 0.769) + (b * 0.189)).min(255.0) as u8;
        pixel[1] = ((r * 0.349) + (g * 0.686) + (b * 0.168)).min(255.0) as u8;
        pixel[2] = ((r * 0.272) + (g * 0.534) + (b * 0.131)).min(255.0) as u8;
    }
}

#[wasm_bindgen]
pub fn ajustar_brilho(dados: &mut [u8], fator: i16) {
    for pixel in dados.chunks_mut(4) {
        pixel[0] = (pixel[0] as i16 + fator).clamp(0, 255) as u8;
        pixel[1] = (pixel[1] as i16 + fator).clamp(0, 255) as u8;
        pixel[2] = (pixel[2] as i16 + fator).clamp(0, 255) as u8;
    }
}

#[wasm_bindgen]
pub fn blur_simples(dados: &mut [u8], largura: u32, altura: u32) {
    let w = largura as usize;
    let h = altura as usize;
    let original = dados.to_vec();

    for y in 1..(h - 1) {
        for x in 1..(w - 1) {
            for c in 0..3usize {
                let soma: u32 = [
                    original[((y-1)*w + x-1)*4 + c] as u32,
                    original[((y-1)*w + x  )*4 + c] as u32,
                    original[((y-1)*w + x+1)*4 + c] as u32,
                    original[(y*w    + x-1)*4 + c] as u32,
                    original[(y*w    + x  )*4 + c] as u32,
                    original[(y*w    + x+1)*4 + c] as u32,
                    original[((y+1)*w + x-1)*4 + c] as u32,
                    original[((y+1)*w + x  )*4 + c] as u32,
                    original[((y+1)*w + x+1)*4 + c] as u32,
                ].iter().sum();
                dados[(y*w + x)*4 + c] = (soma / 9) as u8;
            }
        }
    }
}
```

```html
<!-- index.html para processamento de imagem -->
<script type="module">
  import init, { aplicar_escala_cinza, aplicar_sepia, ajustar_brilho, blur_simples }
    from './pkg/processador_imagem.js';

  await init();

  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  const img = document.getElementById('imagem-original');

  function resetarImagem() {
    ctx.drawImage(img, 0, 0);
  }

  function obterPixels() {
    return ctx.getImageData(0, 0, canvas.width, canvas.height);
  }

  window.aplicarFiltro = function(filtro) {
    resetarImagem();
    const imageData = obterPixels();
    const dados = imageData.data; // Uint8ClampedArray

    // Passa os dados diretamente para o WASM (zero-copy via SharedArrayBuffer)
    if (filtro === 'cinza')    aplicar_escala_cinza(dados);
    if (filtro === 'sepia')    aplicar_sepia(dados);
    if (filtro === 'negativo') aplicar_negativo(dados);
    if (filtro === 'brilho+')  ajustar_brilho(dados, 50);
    if (filtro === 'brilho-')  ajustar_brilho(dados, -50);
    if (filtro === 'blur')     blur_simples(dados, canvas.width, canvas.height);

    ctx.putImageData(imageData, 0, 0);
  };
</script>
```

---

## Projeto WASM: Comunicação com JavaScript

### Chamar APIs do Browser via web-sys

```rust
use wasm_bindgen::prelude::*;
use wasm_bindgen_futures::JsFuture;
use web_sys::{Request, RequestInit, Response, console};
use js_sys::Promise;

// Logar no console do browser
#[wasm_bindgen]
pub fn logar_mensagem(msg: &str) {
    console::log_1(&JsValue::from_str(msg));
}

// Função assíncrona WASM → chama fetch do browser
#[wasm_bindgen]
pub async fn buscar_dados(url: &str) -> Result<JsValue, JsValue> {
    let opts = RequestInit::new();
    let request = Request::new_with_str_and_init(url, &opts)?;

    let window = web_sys::window().unwrap();
    let resp_value = JsFuture::from(window.fetch_with_request(&request)).await?;
    let resp: Response = resp_value.dyn_into()?;

    let json = JsFuture::from(resp.json()?).await?;
    Ok(json)
}

// Callback JavaScript → Rust
#[wasm_bindgen]
pub fn ao_clicar(callback: &js_sys::Function) {
    let window = web_sys::window().unwrap();
    let document = window.document().unwrap();
    let botao = document.get_element_by_id("meu-botao").unwrap();

    let closure = Closure::wrap(Box::new(move || {
        callback.call0(&JsValue::NULL).unwrap();
    }) as Box<dyn Fn()>);

    botao.add_event_listener_with_callback(
        "click",
        closure.as_ref().unchecked_ref(),
    ).unwrap();

    closure.forget(); // mantém closure viva
}
```

```toml
# Dependências adicionais para async e web APIs
[dependencies]
wasm-bindgen = "0.2"
wasm-bindgen-futures = "0.4"
web-sys = { version = "0.3", features = [
    "Window", "Document", "Element", "HtmlElement",
    "Request", "RequestInit", "Response",
    "console", "fetch",
] }
js-sys = "0.3"
```

### Estruturas Complexas com Serde

```rust
use wasm_bindgen::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct Produto {
    pub id: u32,
    pub nome: String,
    pub preco: f64,
    pub disponivel: bool,
    pub tags: Vec<String>,
}

#[wasm_bindgen]
pub fn processar_produto(dados: JsValue) -> JsValue {
    // Deserializar do JavaScript
    let produto: Produto = serde_wasm_bindgen::from_value(dados)
        .expect("Dados inválidos");

    // Processar em Rust
    let preco_com_desconto = if produto.disponivel {
        produto.preco * 0.9
    } else {
        produto.preco
    };

    let produto_processado = Produto {
        preco: preco_com_desconto,
        tags: produto.tags.iter()
            .map(|t| t.to_uppercase())
            .collect(),
        ..produto
    };

    // Serializar de volta para JavaScript
    serde_wasm_bindgen::to_value(&produto_processado)
        .expect("Falha na serialização")
}
```

### Aplicação Full SPA com Yew (React para Rust)

**Yew** é o framework frontend mais popular para Rust+WASM, com API similar ao React (componentes, hooks, estado).

```bash
# Instalar Trunk — bundler para apps Yew
cargo install trunk

# Criar projeto
cargo new --lib app-yew
cd app-yew
```

```toml
# Cargo.toml
[dependencies]
yew = { version = "0.21", features = ["csr"] }
```

```rust
// src/lib.rs — contador com Yew
use yew::prelude::*;

#[function_component(Contador)]
fn contador() -> Html {
    let count = use_state(|| 0i32);

    let incrementar = {
        let count = count.clone();
        Callback::from(move |_| count.set(*count + 1))
    };

    let decrementar = {
        let count = count.clone();
        Callback::from(move |_| count.set(*count - 1))
    };

    let resetar = {
        let count = count.clone();
        Callback::from(move |_| count.set(0))
    };

    let cor = if *count > 0 { "green" } else if *count < 0 { "red" } else { "black" };

    html! {
        <div style="text-align: center; font-family: sans-serif;">
            <h1>{"Contador Rust + Yew 🦀"}</h1>
            <p style={format!("font-size: 3rem; color: {cor};")}>
                { *count }
            </p>
            <button onclick={decrementar}>{ "-" }</button>
            <button onclick={resetar} style="margin: 0 8px;">{ "Reset" }</button>
            <button onclick={incrementar}>{ "+" }</button>
        </div>
    }
}

#[function_component(App)]
pub fn app() -> Html {
    html! {
        <Contador />
    }
}

fn main() {
    yew::Renderer::<App>::new().render();
}
```

```bash
# Executar com hot-reload
trunk serve

# Build para produção
trunk build --release
```

```html
<!-- index.html (na raiz do projeto — Trunk usa este arquivo) -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>App Yew</title>
</head>
<body>
  <!-- Trunk injeta o bundle WASM automaticamente -->
</body>
</html>
```

---

## API REST com Rust

Rust possui um ecossistema maduro para construção de APIs Web REST com acesso a bancos de dados relacionais, combinando alta performance, segurança de tipos e verificação em tempo de compilação.

### Frameworks e Bibliotecas

| Camada | Opção | Destaque |
|---|---|---|
| **Web framework** | **Axum** | Moderno, ergonômico, construído sobre Tower+Tokio — escolha padrão |
| **Web framework** | **Actix-web** | Performance máxima (top benchmarks), mais verboso |
| **Web framework** | **Rocket** | Muito ergonômico, macro-heavy, bom para prototipagem |
| **Banco de dados** | **SQLx** | SQL nativo verificado em **tempo de compilação**, async |
| **ORM async** | **SeaORM** | Construído sobre SQLx, estilo ActiveRecord |
| **ORM síncrono** | **Diesel** | ORM mais maduro, sem async nativo |
| **Async runtime** | **Tokio** | Runtime padrão do ecossistema |
| **Serialização** | **Serde** | Serialização/desserialização JSON universal |

### Stack Recomendada

**Axum + SQLx + Tokio + Serde** — amplamente adotada, com excelente suporte a PostgreSQL, MySQL e SQLite.

### Cargo.toml

```toml
[package]
name = "api-rust"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "uuid", "chrono"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
uuid = { version = "1", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
tower-http = { version = "0.5", features = ["cors", "trace"] }
tracing = "0.1"
tracing-subscriber = "0.3"
dotenvy = "0.15"
```

### Estrutura do Projeto

```
src/
├── main.rs
├── errors.rs       # tratamento centralizado de erros
├── models.rs       # structs de dados
└── routes/
    ├── mod.rs
    └── produtos.rs
migrations/
└── 0001_criar_produtos.sql
```

### src/models.rs

```rust
use serde::{Deserialize, Serialize};
use sqlx::FromRow;
use uuid::Uuid;
use chrono::NaiveDateTime;

#[derive(Debug, Serialize, Deserialize, FromRow)]
pub struct Produto {
    pub id: Uuid,
    pub nome: String,
    pub preco: f64,
    pub estoque: i32,
    pub criado_em: NaiveDateTime,
}

#[derive(Debug, Deserialize)]
pub struct CriarProduto {
    pub nome: String,
    pub preco: f64,
    pub estoque: i32,
}

#[derive(Debug, Deserialize)]
pub struct AtualizarProduto {
    pub nome: Option<String>,
    pub preco: Option<f64>,
    pub estoque: Option<i32>,
}
```

### src/errors.rs

```rust
use axum::{http::StatusCode, response::{IntoResponse, Response}, Json};
use serde_json::json;

pub enum AppError {
    NaoEncontrado,
    BancoDados(sqlx::Error),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, msg) = match self {
            AppError::NaoEncontrado => (StatusCode::NOT_FOUND, "Recurso não encontrado"),
            AppError::BancoDados(_) => (StatusCode::INTERNAL_SERVER_ERROR, "Erro no banco de dados"),
        };
        (status, Json(json!({ "erro": msg }))).into_response()
    }
}

impl From<sqlx::Error> for AppError {
    fn from(e: sqlx::Error) -> Self {
        match e {
            sqlx::Error::RowNotFound => AppError::NaoEncontrado,
            outros => AppError::BancoDados(outros),
        }
    }
}
```

### src/routes/produtos.rs

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    Json,
};
use sqlx::PgPool;
use uuid::Uuid;

use crate::{errors::AppError, models::{AtualizarProduto, CriarProduto, Produto}};

pub async fn listar(State(pool): State<PgPool>) -> Result<Json<Vec<Produto>>, AppError> {
    let produtos = sqlx::query_as!(Produto, "SELECT * FROM produtos ORDER BY criado_em DESC")
        .fetch_all(&pool)
        .await?;
    Ok(Json(produtos))
}

pub async fn buscar(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> Result<Json<Produto>, AppError> {
    let produto = sqlx::query_as!(Produto, "SELECT * FROM produtos WHERE id = $1", id)
        .fetch_one(&pool)
        .await?;
    Ok(Json(produto))
}

pub async fn criar(
    State(pool): State<PgPool>,
    Json(body): Json<CriarProduto>,
) -> Result<(StatusCode, Json<Produto>), AppError> {
    let produto = sqlx::query_as!(
        Produto,
        "INSERT INTO produtos (id, nome, preco, estoque, criado_em)
         VALUES ($1, $2, $3, $4, NOW())
         RETURNING *",
        Uuid::new_v4(),
        body.nome,
        body.preco,
        body.estoque,
    )
    .fetch_one(&pool)
    .await?;
    Ok((StatusCode::CREATED, Json(produto)))
}

pub async fn atualizar(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
    Json(body): Json<AtualizarProduto>,
) -> Result<Json<Produto>, AppError> {
    let produto = sqlx::query_as!(
        Produto,
        "UPDATE produtos
         SET nome    = COALESCE($2, nome),
             preco   = COALESCE($3, preco),
             estoque = COALESCE($4, estoque)
         WHERE id = $1
         RETURNING *",
        id,
        body.nome,
        body.preco,
        body.estoque,
    )
    .fetch_one(&pool)
    .await?;
    Ok(Json(produto))
}

pub async fn remover(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> Result<StatusCode, AppError> {
    let resultado = sqlx::query!("DELETE FROM produtos WHERE id = $1", id)
        .execute(&pool)
        .await?;

    if resultado.rows_affected() == 0 {
        return Err(AppError::NaoEncontrado);
    }
    Ok(StatusCode::NO_CONTENT)
}
```

### src/main.rs

```rust
use axum::{routing::{delete, get, post, put}, Router};
use sqlx::postgres::PgPoolOptions;
use tower_http::cors::CorsLayer;
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

mod errors;
mod models;
mod routes;

#[tokio::main]
async fn main() {
    dotenvy::dotenv().ok();

    tracing_subscriber::registry()
        .with(tracing_subscriber::EnvFilter::new("info"))
        .with(tracing_subscriber::fmt::layer())
        .init();

    let db_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL deve estar definida");

    let pool = PgPoolOptions::new()
        .max_connections(10)
        .connect(&db_url)
        .await
        .expect("Falha ao conectar ao banco");

    sqlx::migrate!("./migrations").run(&pool).await.unwrap();

    let app = Router::new()
        .route("/produtos",     get(routes::produtos::listar).post(routes::produtos::criar))
        .route("/produtos/:id", get(routes::produtos::buscar)
                                    .put(routes::produtos::atualizar)
                                    .delete(routes::produtos::remover))
        .layer(CorsLayer::permissive())
        .with_state(pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    tracing::info!("Servidor ouvindo em http://0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}
```

### Migration SQL

```sql
-- migrations/0001_criar_produtos.sql
CREATE TABLE IF NOT EXISTS produtos (
    id         UUID        PRIMARY KEY,
    nome       TEXT        NOT NULL,
    preco      FLOAT8      NOT NULL CHECK (preco >= 0),
    estoque    INTEGER     NOT NULL DEFAULT 0,
    criado_em  TIMESTAMP   NOT NULL DEFAULT NOW()
);
```

### Executar o Projeto

```bash
# Definir URL do banco
export DATABASE_URL="postgres://usuario:senha@localhost/minha_api"

# Criar banco e executar migrations
cargo install sqlx-cli
sqlx database create
sqlx migrate run

# Rodar em modo watch (hot-reload)
cargo install cargo-watch
cargo watch -x run
```

### Verificação de Queries em Tempo de Compilação

O SQLx verifica se as queries SQL batem com o schema do banco **durante o `cargo build`**. Para ambientes de CI sem banco disponível, use o modo offline:

```bash
# Gerar cache das queries para build offline
cargo sqlx prepare

# O arquivo .sqlx/ deve ser commitado no repositório
```

### Endpoints Gerados

| Método | Rota | Ação |
|---|---|---|
| `GET` | `/produtos` | Listar todos |
| `GET` | `/produtos/:id` | Buscar por ID |
| `POST` | `/produtos` | Criar novo |
| `PUT` | `/produtos/:id` | Atualizar parcialmente |
| `DELETE` | `/produtos/:id` | Remover |

---

## Testes

### Testes Unitários

```rust
// src/lib.rs — testes ficam no mesmo arquivo ou em src/tests/

pub fn fatorial(n: u64) -> u64 {
    (1..=n).product()
}

pub fn e_primo(n: u64) -> bool {
    if n < 2 { return false; }
    if n == 2 { return true; }
    if n % 2 == 0 { return false; }
    (3..).step_by(2)
         .take_while(|&i| i * i <= n)
         .all(|i| n % i != 0)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn fatorial_de_zero_e_um() {
        assert_eq!(fatorial(0), 1); // 0! = 1 por convenção? — veremos
        // Para Rust, (1..=0).product() = 1 ✓
    }

    #[test]
    fn fatorial_de_cinco() {
        assert_eq!(fatorial(5), 120);
    }

    #[test]
    fn primos_conhecidos() {
        assert!(e_primo(2));
        assert!(e_primo(7));
        assert!(e_primo(97));
    }

    #[test]
    fn nao_primos() {
        assert!(!e_primo(1));
        assert!(!e_primo(4));
        assert!(!e_primo(100));
    }

    #[test]
    #[should_panic(expected = "overflow")]
    fn fatorial_overflow() {
        fatorial(21); // u64 transborda a partir de 21!
    }
}
```

### Testes de Integração

```rust
// tests/integracao.rs — crate separada, testa API pública
use minha_lib::fatorial;

#[test]
fn sequencia_fatorial() {
    let esperado = [1, 1, 2, 6, 24, 120];
    for (n, &valor) in esperado.iter().enumerate() {
        assert_eq!(fatorial(n as u64), valor, "fatorial({n}) falhou");
    }
}
```

### Testes WASM com wasm-pack

```rust
// tests/wasm.rs — executa no browser (via headless Chrome/Firefox)
use wasm_bindgen_test::*;

wasm_bindgen_test_configure!(run_in_browser);

#[wasm_bindgen_test]
fn teste_saudacao() {
    use minha_lib_wasm::saudacao;
    assert_eq!(saudacao("Rust"), "Olá, Rust! Escrito em Rust 🦀");
}

#[wasm_bindgen_test]
async fn teste_assincrono() {
    // Pode testar funções async/await
    let resultado = minha_operacao_async().await;
    assert!(resultado.is_ok());
}
```

```bash
# Executar testes em headless Chrome
wasm-pack test --headless --chrome

# Executar testes em headless Firefox
wasm-pack test --headless --firefox

# Executar testes Node.js (sem browser)
wasm-pack test --node
```

### Benchmarks com Criterion

```toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "meu_benchmark"
harness = false
```

```rust
// benches/meu_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use minha_lib::fatorial;

fn benchmark_fatorial(c: &mut Criterion) {
    c.bench_function("fatorial 15", |b| {
        b.iter(|| fatorial(black_box(15)))
    });
}

criterion_group!(benches, benchmark_fatorial);
criterion_main!(benches);
```

```bash
cargo bench               # executa e gera relatório HTML
cargo bench -- fatorial   # filtra por nome
```

---

## Referências

### Documentação Oficial

- [The Rust Programming Language (The Book)](https://doc.rust-lang.org/book/) — livro oficial, gratuito online
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) — aprendizado por exemplos
- [Rustonomicon](https://doc.rust-lang.org/nomicon/) — Rust unsafe avançado
- [Rust Reference](https://doc.rust-lang.org/reference/) — especificação da linguagem
- [Standard Library Docs](https://doc.rust-lang.org/std/) — referência da stdlib

### WASM e Frontend

- [The wasm-bindgen Guide](https://rustwasm.github.io/wasm-bindgen/) — guia oficial wasm-bindgen
- [Rust and WebAssembly Book](https://rustwasm.github.io/docs/book/) — tutorial completo Rust+WASM
- [wasm-pack Docs](https://rustwasm.github.io/docs/wasm-pack/) — referência do wasm-pack
- [Yew Framework](https://yew.rs/) — SPA framework Rust+WASM estilo React
- [Leptos](https://leptos.dev/) — framework reativo com SSR, similar ao SolidJS
- [Dioxus](https://dioxuslabs.com/) — framework cross-platform (web, desktop, mobile)

### Ferramentas e Pacotes

- [crates.io](https://crates.io/) — repositório oficial de pacotes Rust
- [docs.rs](https://docs.rs/) — documentação gerada para todos os crates
- [lib.rs](https://lib.rs/) — alternativa ao crates.io com melhor busca
- [Awesome Rust](https://github.com/rust-unofficial/awesome-rust) — curadoria de crates por categoria

### Exercícios e Prática

- [Rustlings](https://github.com/rust-lang/rustlings) — exercícios interativos para iniciantes
- [Exercism — Rust Track](https://exercism.org/tracks/rust) — exercícios com mentoria
- [Advent of Code](https://adventofcode.com/) — desafios anuais populares com a comunidade Rust
