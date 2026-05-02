# Kotlin — Guia Prático para Desenvolvimento Web com Spring Boot

Kotlin é uma linguagem estaticamente tipada, compilada para JVM (e também para JavaScript e binários nativos), criada pela JetBrains e tornada open-source em 2011. Desde 2017 é a linguagem oficial para desenvolvimento Android e, no ecossistema Spring, é **suportada como primeira classe** desde o Spring Boot 2.x. Seu design combina a robustez da JVM com **concisão, segurança contra nulos e um sistema de tipos expressivo**, reduzindo o código boilerplate sem abrir mão da interoperabilidade total com Java.

---

## Sumário

1. [Por que Kotlin?](#por-que-kotlin)
2. [Kotlin vs Java — Comparações Diretas](#kotlin-vs-java--comparações-diretas)
3. [Ambiente de Desenvolvimento](#ambiente-de-desenvolvimento)
4. [IDEs e Ferramentas](#ides-e-ferramentas)
5. [Fundamentos da Linguagem](#fundamentos-da-linguagem)
6. [Coleções de Dados](#coleções-de-dados)
7. [Funções Avançadas](#funções-avançadas)
8. [Orientação a Objetos em Kotlin](#orientação-a-objetos-em-kotlin)
9. [Coroutines — Programação Assíncrona](#coroutines--programação-assíncrona)
   - [Spring WebFlux com Coroutines — Stack Completo](#spring-webflux-com-coroutines--stack-completo)
10. [Spring Boot com Kotlin](#spring-boot-com-kotlin)
    - [Configuração Tipada com @ConfigurationProperties](#configuração-tipada-com-configurationproperties)
11. [Integração com Banco de Dados](#integração-com-banco-de-dados)
12. [Spring Security com Kotlin](#spring-security-com-kotlin)
13. [Testes](#testes)
    - [Teste de Unidade com MockK](#teste-de-unidade-com-mockk)
14. [Referências](#referências)

---

## Por que Kotlin?

| Característica | Detalhe |
|---|---|
| **Null Safety nativo** | Tipos não-nulos e nulos são distintos em tempo de compilação; elimina `NullPointerException` em grande parte do código |
| **Concisão** | Data classes, smart casts, extension functions e inferência de tipos reduzem o boilerplate em ~40% vs. Java equivalente |
| **Interoperabilidade total com Java** | Chama código Java diretamente e é chamado por Java — mesma JVM, mesmas bibliotecas |
| **Coroutines** | Programação assíncrona estruturada e leve, integrada à linguagem; alternativa ao `CompletableFuture` e Reactor |
| **Kotlin DSL** | Permite criar APIs fluentes e type-safe; usado pelo Gradle e pelo Spring Security |
| **Multiplataforma** | Kotlin/JVM, Kotlin/JS, Kotlin/Native e Kotlin Multiplatform para código compartilhado |
| **Suporte oficial Spring** | Spring Boot, Spring MVC, Spring WebFlux e Spring Security possuem extensões Kotlin de primeira classe |
| **Ferramentas da JetBrains** | IntelliJ IDEA (criador da linguagem) oferece refatoração, depuração e inspeções nativas |

---

## Kotlin vs Java — Comparações Diretas

Esta seção apresenta equivalências lado a lado para quem já conhece Java.

### Declaração de variáveis

```java
// Java
final String nome = "Maria";        // imutável (equivalente a val)
String sobrenome = "Silva";         // mutável (equivalente a var)
String prefixo;                     // declarado sem valor inicial
```

```kotlin
// Kotlin
val nome = "Maria"                  // imutável (inferência de tipo)
var sobrenome = "Silva"             // mutável
val prefixo: String                 // declarado, deve ser inicializado antes do uso
```

---

### Null Safety

```java
// Java — NullPointerException é possível em tempo de execução
String valor = null;
System.out.println(valor.length()); // lança NPE

// Java moderno — Optional
Optional<String> opcional = Optional.ofNullable(getValor());
String resultado = opcional.orElse("padrão");
```

```kotlin
// Kotlin — erros de nulo são verificados em tempo de compilação
val valor: String = "texto"         // não-nulo: não aceita null
val anulavel: String? = null        // tipo nulável declarado explicitamente

// Safe call — retorna null se anulavel for null, sem NPE
val tamanho: Int? = anulavel?.length

// Elvis operator — valor padrão quando null
val tamanho: Int = anulavel?.length ?: 0

// Not-null assertion — lança KotlinNullPointerException se null (use com cautela)
val tamanho: Int = anulavel!!.length
```

---

### Classes e Data Classes

```java
// Java — POJO verboso
public class Produto {
    private final Long id;
    private final String nome;
    private final Double preco;

    public Produto(Long id, String nome, Double preco) {
        this.id = id; this.nome = nome; this.preco = preco;
    }
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public Double getPreco() { return preco; }

    @Override
    public boolean equals(Object o) { /* boilerplate */ }
    @Override
    public int hashCode() { /* boilerplate */ }
    @Override
    public String toString() { /* boilerplate */ }
}

// Java 16+ Records — melhor, mas ainda limitado
public record Produto(Long id, String nome, Double preco) {}
```

```kotlin
// Kotlin — data class gera equals, hashCode, toString e copy automaticamente
data class Produto(
    val id: Long,
    val nome: String,
    val preco: Double
)

// copy() para criar cópia com campos modificados (imutabilidade facilitada)
val original = Produto(1L, "Café", 12.90)
val promovido = original.copy(preco = 9.90)
```

---

### Condicionais e when

```java
// Java switch (expression, Java 14+)
String descricao = switch (status) {
    case "A" -> "Ativo";
    case "I" -> "Inativo";
    default  -> "Desconhecido";
};
```

```kotlin
// Kotlin when — mais expressivo, sem fall-through, funciona com qualquer tipo
val descricao = when (status) {
    "A"  -> "Ativo"
    "I"  -> "Inativo"
    else -> "Desconhecido"
}

// when com ranges e múltiplos valores
val classificacao = when (nota) {
    in 90..100 -> "Excelente"
    in 70..89  -> "Bom"
    in 50..69  -> "Regular"
    else       -> "Insuficiente"
}

// when sem argumento (substitui if-else if encadeado)
val mensagem = when {
    usuario.isAdmin()  -> "Administrador"
    usuario.isAtivo()  -> "Usuário ativo"
    else               -> "Acesso negado"
}
```

---

### Funções e Lambdas

```java
// Java
public int somar(int a, int b) { return a + b; }

// Lambda Java
Function<Integer, Integer> dobrar = x -> x * 2;
List<Integer> pares = lista.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

```kotlin
// Kotlin — corpo de expressão (= em vez de { return })
fun somar(a: Int, b: Int): Int = a + b

// Função com corpo de bloco (múltiplas linhas) — tipo de retorno obrigatório
fun calcularDesconto(preco: Double, percentual: Int): Double {
    require(percentual in 0..100) { "Percentual inválido: $percentual" }
    val desconto = preco * percentual / 100.0
    val precoFinal = preco - desconto
    return precoFinal.coerceAtLeast(0.0)   // nunca retorna valor negativo
}

// Função com múltiplos tipos de retorno usando when
fun classificarIdade(idade: Int): String {
    require(idade >= 0) { "Idade não pode ser negativa" }
    return when {
        idade < 13  -> "Criança"
        idade < 18  -> "Adolescente"
        idade < 60  -> "Adulto"
        else        -> "Idoso"
    }
}

// Parâmetros com valores padrão e argumentos nomeados
fun criarUrl(host: String, porta: Int = 8080, tls: Boolean = false): String =
    "${if (tls) "https" else "http"}://$host:$porta"

criarUrl("localhost")                         // http://localhost:8080
criarUrl("api.example.com", tls = true)       // https://api.example.com:8080

// Lambda e coleções — mais conciso que Streams
val pares = lista.filter { it % 2 == 0 }

// Lambda com múltiplas linhas — chaves obrigatórias, última expressão é o retorno
val calcularFrete: (Double) -> Double = { peso ->
    val taxaBase = 8.90
    val taxaPorKg = 2.50
    val total = taxaBase + (peso * taxaPorKg)
    total.coerceAtMost(99.90)   // frete máximo de R$ 99,90
}

// it — nome implícito do parâmetro em lambdas de um argumento
val dobrados = lista.map { it * 2 }
```

---

### String Templates

```java
// Java
String msg = String.format("Olá, %s! Você tem %d mensagens.", nome, qtd);
// Java 15+ Text Blocks
String json = """
    { "nome": "%s" }
    """.formatted(nome);
```

```kotlin
// Kotlin — interpolação nativa com ${}
val msg = "Olá, $nome! Você tem $qtd mensagens."
val json = """
    { "nome": "$nome" }
"""

// Expressões dentro de ${}
val info = "Preço: ${produto.preco.toBigDecimal().setScale(2)} BRL"
```

---

### Herança e Interfaces

```java
// Java — classes são abertas por padrão
public class Animal {
    public void falar() { System.out.println("..."); }
}
public class Cachorro extends Animal {
    @Override
    public void falar() { System.out.println("Au!"); }
}
```

```kotlin
// Kotlin — classes são FECHADAS por padrão (equivalente a final em Java)
// Use open para permitir herança
open class Animal {
    open fun falar() = println("...")
}

class Cachorro : Animal() {
    override fun falar() = println("Au!")
}
```

---

## Ambiente de Desenvolvimento

### Pré-requisitos

Kotlin roda na JVM, portanto o JDK é obrigatório. Recomenda-se **JDK 21 (LTS)** para projetos Spring Boot 3.x.

**Windows:**

```powershell
# Via winget
winget install EclipseAdoptium.Temurin.21.JDK

# Via SDKMAN (requer WSL ou Git Bash)
sdk install java 21.0.3-tem
sdk install kotlin

# Verificar
java -version
kotlinc -version
```

**macOS:**

```bash
# Via Homebrew
brew install --cask temurin@21
brew install kotlin

java -version
kotlinc -version
```

**Linux (Debian/Ubuntu):**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk

# SDKMAN (recomendado para gerenciar versões)
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install kotlin

java -version
kotlinc -version
```

---

### SDKMAN — Gerenciador de Versões (recomendado)

SDKMAN é o equivalente ao `pyenv` para o ecossistema JVM.

```bash
# Listar versões disponíveis
sdk list java
sdk list kotlin

# Instalar e usar versão específica
sdk install java 21.0.3-tem
sdk install kotlin 2.1.0

# Definir versão padrão
sdk default java 21.0.3-tem

# Usar versão específica no projeto (cria .sdkmanrc)
sdk use java 21.0.3-tem
sdk env init
```

---

### Criando um Projeto Spring Boot com Kotlin

A forma mais rápida é via **Spring Initializr**:

```bash
# Via curl (cria projeto com Gradle Kotlin DSL)
curl https://start.spring.io/starter.tgz \
  -d type=gradle-project-kotlin \
  -d language=kotlin \
  -d bootVersion=3.5.0 \
  -d groupId=br.com.exemplo \
  -d artifactId=minha-api \
  -d name=minha-api \
  -d packageName=br.com.exemplo.minhaapi \
  -d javaVersion=21 \
  -d dependencies=web,data-jpa,postgresql,security,validation \
  | tar -xzf -
```

Ou acesse [start.spring.io](https://start.spring.io) e selecione **Language: Kotlin**, **Build: Gradle - Kotlin**.

---

### Estrutura do Projeto

```
minha-api/
├── build.gradle.kts          ← Script de build (Kotlin DSL, em vez de Groovy)
├── settings.gradle.kts
├── src/
│   ├── main/
│   │   ├── kotlin/
│   │   │   └── br/com/exemplo/minhaapi/
│   │   │       ├── MinhaApiApplication.kt
│   │   │       ├── config/
│   │   │       ├── controller/
│   │   │       ├── domain/
│   │   │       │   ├── model/
│   │   │       │   └── repository/
│   │   │       └── service/
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       └── kotlin/
```

---

### build.gradle.kts

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.spring") version "2.1.0"   // abre classes para Spring AOP
    kotlin("plugin.jpa") version "2.1.0"      // gera construtores sem args para entidades JPA
    id("org.springframework.boot") version "3.5.0"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "br.com.exemplo"
version = "0.0.1-SNAPSHOT"

java {
    toolchain { languageVersion = JavaLanguageVersion.of(21) }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")

    runtimeOnly("org.postgresql:postgresql")

    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
}

kotlin {
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")   // habilita verificações null JSR-305
    }
}
```

> **Nota:** O plugin `kotlin("plugin.spring")` é essencial — ele torna as classes Kotlin `open` automaticamente onde o Spring precisa criar proxies AOP (em `@Service`, `@Component`, `@Configuration`, etc.). O plugin `kotlin("plugin.jpa")` gera construtores sem argumentos para entidades JPA, necessários pelo Hibernate.

---

## IDEs e Ferramentas

### IntelliJ IDEA (recomendado)

Criada pela JetBrains (mesma empresa do Kotlin), é a IDE com melhor suporte nativo.

| Edição | Preço | Indicada para |
|---|---|---|
| **Community** | Gratuita | Kotlin/JVM, projetos sem Spring Enterprise features |
| **Ultimate** | Paga (free para estudantes) | Spring Boot, bancos de dados, HTTP client integrado |

**Recursos relevantes para Kotlin:**
- Conversão automática Java → Kotlin (`Ctrl+Alt+Shift+K`)
- Detecção de idioms Kotlin (sugere substituir `if-null` por `?:`)
- Depurador com suporte a coroutines (painel "Coroutines" dedicado)
- HTTP Client integrado (`.http` files) para testar APIs
- Inspeções específicas de null safety

**Plugins recomendados:**
- **Kotlin** (pré-instalado)
- **Spring Boot** (pré-instalado na Ultimate)
- **SonarLint** — análise de qualidade de código

---

### VS Code

Alternativa gratuita com suporte razoável via extensões:

- **Kotlin Language** (mathiasfrohlich) — syntax highlighting, formatação
- **Extension Pack for Java** (Microsoft) — suporte JVM, debug, Maven/Gradle
- **Spring Boot Extension Pack** (VMware) — Spring Boot support

> Para desenvolvimento Spring Boot profissional, IntelliJ IDEA Ultimate é significativamente mais produtiva. VS Code funciona bem para edições pontuais ou projetos simples.

---

### Android Studio

Baseada no IntelliJ IDEA, é a IDE oficial para desenvolvimento Android com Kotlin. Para projetos exclusivamente web/backend, prefira IntelliJ IDEA.

---

## Fundamentos da Linguagem

### Variáveis e Tipos

```kotlin
// Tipos primitivos — sem distinção explícita de boxed vs. unboxed (o compilador decide)
val inteiro: Int = 42
val longo: Long = 10_000_000L     // underscore para legibilidade
val decimal: Double = 3.14
val texto: String = "Kotlin"
val booleano: Boolean = true
val caractere: Char = 'K'

// Inferência de tipo
val nome = "Ana"             // String inferido
val preco = 29.99            // Double inferido
val quantidade = 100         // Int inferido

// Constantes em nível de arquivo (equivalente a static final)
const val VERSAO_API = "v1"
const val TIMEOUT_MS = 5000L
```

---

### Condicionais

```kotlin
// if como expressão (retorna valor)
val status = if (ativo) "Ativo" else "Inativo"

// if tradicional
if (saldo >= valor) {
    println("Transação aprovada")
} else if (saldo > 0) {
    println("Saldo insuficiente")
} else {
    println("Conta sem saldo")
}

// when — substitui switch e if-else if
val descricaoPerfil = when (perfil) {
    "ADMIN"   -> "Administrador com acesso total"
    "EDITOR"  -> "Editor de conteúdo"
    "VIEWER"  -> "Somente leitura"
    else      -> throw IllegalArgumentException("Perfil desconhecido: $perfil")
}

// when com smart cast — verifica tipo E faz cast automaticamente
fun processar(obj: Any): String = when (obj) {
    is Int    -> "Inteiro: ${obj + 1}"    // obj já é Int aqui
    is String -> "Texto: ${obj.uppercase()}"
    is List<*>-> "Lista com ${obj.size} itens"
    else      -> "Tipo desconhecido"
}
```

---

### Loops

```kotlin
// for com range
for (i in 1..10) print("$i ")        // 1 2 3 4 5 6 7 8 9 10
for (i in 1 until 10) print("$i ")   // 1 2 3 4 5 6 7 8 9 (exclui 10)
for (i in 10 downTo 1) print("$i ")  // 10 9 8 ... 1
for (i in 0..20 step 5) print("$i ") // 0 5 10 15 20

// for sobre coleção
val frutas = listOf("maçã", "banana", "laranja")
for (fruta in frutas) println(fruta)

// withIndex — índice + valor
for ((indice, fruta) in frutas.withIndex()) {
    println("[$indice] $fruta")
}

// forEach (funcional)
frutas.forEach { fruta -> println(fruta) }
frutas.forEachIndexed { i, fruta -> println("[$i] $fruta") }

// while e do-while (igual a Java)
var contador = 0
while (contador < 5) {
    println(contador++)
}

// repeat — loop simples N vezes
repeat(3) { println("Iteração $it") }
```

---

### Tratamento de Erros

```kotlin
// try-catch como expressão
val numero: Int = try {
    entrada.toInt()
} catch (e: NumberFormatException) {
    -1  // valor padrão em caso de erro
}

// Kotlin NÃO tem checked exceptions — não é preciso declarar throws
fun lerArquivo(caminho: String): String {
    return File(caminho).readText()  // IOException não precisa ser declarada
}

// Rethrow com contexto
fun buscarProduto(id: Long): Produto {
    return repositorio.findById(id)
        ?: throw NoSuchElementException("Produto não encontrado: id=$id")
}
```

> **Diferença importante do Java:** Kotlin não tem *checked exceptions*. Toda exception é unchecked, simplificando lambdas e código funcional.

---

## Coleções de Dados

Kotlin separa explicitamente coleções **imutáveis** (somente leitura) e **mutáveis**.

```kotlin
// List — imutável (leitura)
val frutas: List<String> = listOf("maçã", "banana", "laranja")
// frutas.add("uva")  // erro de compilação — List não tem add()

// MutableList
val lista: MutableList<String> = mutableListOf("maçã", "banana")
lista.add("uva")
lista.removeAt(0)

// Set
val conjunto: Set<Int> = setOf(1, 2, 3, 2, 1)  // {1, 2, 3} — sem duplicatas
val mutavel: MutableSet<Int> = mutableSetOf(1, 2, 3)

// Map
val paises: Map<String, String> = mapOf(
    "BR" to "Brasil",
    "US" to "Estados Unidos",
    "PT" to "Portugal"
)
println(paises["BR"])                  // Brasil
println(paises.getOrDefault("AR", "Desconhecido"))

val mutavelMap: MutableMap<String, Int> = mutableMapOf("a" to 1)
mutavelMap["b"] = 2

// Pair e Triple
val coordenada = Pair(10.5, -23.1)
val (lat, lon) = coordenada            // destructuring
val triple = Triple("João", 30, "SP")
val (nome, idade, estado) = triple
```

---

### Operações Funcionais em Coleções

```kotlin
val produtos = listOf(
    Produto(1L, "Notebook", 3500.0),
    Produto(2L, "Mouse", 120.0),
    Produto(3L, "Teclado", 250.0),
    Produto(4L, "Monitor", 1800.0)
)

// filter — retorna lista filtrada
val caros = produtos.filter { it.preco > 500.0 }

// map — transforma cada elemento
val nomes = produtos.map { it.nome }
val precosMajoriados = produtos.map { it.copy(preco = it.preco * 1.1) }

// find / firstOrNull — retorna primeiro elemento ou null
val notebook = produtos.find { it.nome == "Notebook" }

// any / all / none
val temBarato = produtos.any { it.preco < 200.0 }
val todosBaratos = produtos.all { it.preco < 5000.0 }

// groupBy — agrupa em Map
val porFaixaDePreco = produtos.groupBy {
    when {
        it.preco < 200  -> "Barato"
        it.preco < 1000 -> "Médio"
        else            -> "Caro"
    }
}

// sortedBy / sortedByDescending
val porPreco = produtos.sortedBy { it.preco }
val porPrecoDesc = produtos.sortedByDescending { it.preco }

// sumOf / maxByOrNull / minByOrNull
val totalEstoque = produtos.sumOf { it.preco }
val maisCaro = produtos.maxByOrNull { it.preco }

// flatMap — achata lista de listas
val pedidos = listOf(
    listOf("Item A", "Item B"),
    listOf("Item C")
)
val todosItens = pedidos.flatMap { it }  // ["Item A", "Item B", "Item C"]

// Encadeamento de operações (similar a Streams Java, mas sem .stream() e .collect())
val resultado = produtos
    .filter { it.preco > 100.0 }
    .sortedBy { it.preco }
    .map { "${it.nome}: R$ ${"%.2f".format(it.preco)}" }
    .joinToString(separator = "\n")
```

> **vs. Java Streams:** Em Kotlin, as operações de coleções são chamadas diretamente — sem `.stream()`, sem `.collect(Collectors.toList())`. Para coleções grandes com múltiplos estágios, use `asSequence()` (equivalente lazy ao Stream Java).

```kotlin
// Sequence — avaliação lazy (equivalente a Stream Java)
val resultado = produtos.asSequence()
    .filter { it.preco > 100.0 }
    .map { it.nome }
    .take(3)
    .toList()
```

---

## Funções Avançadas

### Extension Functions

Permitem adicionar métodos a classes existentes sem herança — uma das features mais usadas em Kotlin.

```kotlin
// Adiciona .formatarBRL() a qualquer Double
fun Double.formatarBRL(): String = "R$ ${"%.2f".format(this)}"

// Adiciona .capitalizar() a String (além do capitalize() padrão)
fun String.capitalizar(): String = this.lowercase().replaceFirstChar { it.uppercase() }

// Uso
val preco = 1234.56
println(preco.formatarBRL())  // R$ 1234,56

val nome = "MARIA"
println(nome.capitalizar())   // Maria

// Extension functions em classes do Spring
fun HttpServletRequest.bearerToken(): String? =
    getHeader("Authorization")?.removePrefix("Bearer ")?.trim()
```

---

### Scope Functions

Kotlin oferece funções de escopo que substituem padrões verbosos do Java:

| Função | Contexto (`this`/`it`) | Retorna | Uso típico |
|---|---|---|---|
| `let` | `it` | resultado do bloco | Operações em objetos não-nulos, transformação |
| `run` | `this` | resultado do bloco | Inicialização e cálculo de resultado |
| `apply` | `this` | o próprio objeto | Configuração de objeto (builder pattern) |
| `also` | `it` | o próprio objeto | Efeitos colaterais (logging, validação) |
| `with` | `this` | resultado do bloco | Operações agrupadas em um objeto |

```kotlin
// let — executar apenas se não-nulo
val usuario: Usuario? = repositorio.findByEmail(email)
usuario?.let {
    enviarEmail(it.email, "Bem-vindo, ${it.nome}!")
    registrarLogin(it.id)
}

// apply — configurar objeto (retorna o próprio objeto)
val request = HttpEntity<Map<String, String>>(
    mapOf("usuario" to "admin")
).apply {
    // encadeamento de configurações
}

// also — logging/auditoria sem alterar o fluxo
fun criarProduto(dto: ProdutoDto): Produto =
    Produto(nome = dto.nome, preco = dto.preco)
        .also { log.info("Criando produto: ${it.nome}") }
        .let { repositorio.save(it) }
        .also { log.info("Produto criado com id=${it.id}") }

// with — múltiplas operações no mesmo objeto
val relatorio = with(pedido) {
    """
    Pedido #$numero
    Cliente: ${cliente.nome}
    Total: ${itens.sumOf { it.preco }.formatarBRL()}
    """.trimIndent()
}
```

---

### Funções de Ordem Superior (Higher-Order Functions)

```kotlin
// Função que recebe outra função como parâmetro
fun <T> medirTempo(nome: String, bloco: () -> T): T {
    val inicio = System.currentTimeMillis()
    val resultado = bloco()
    val duracao = System.currentTimeMillis() - inicio
    println("[$nome] $duracao ms")
    return resultado
}

// Uso com trailing lambda
val produtos = medirTempo("buscarProdutos") {
    repositorio.findAll()
}

// Tipo de função como parâmetro
fun validarCampo(valor: String, validador: (String) -> Boolean, mensagem: String) {
    if (!validador(valor)) throw IllegalArgumentException(mensagem)
}

validarCampo("usuario@email.com", { it.contains("@") }, "E-mail inválido")
validarCampo("abc", String::isNotBlank, "Campo obrigatório")  // referência de método
```

---

## Orientação a Objetos em Kotlin

### Classes, Construtores e Propriedades

```kotlin
// Construtor primário inline na declaração da classe
class Pessoa(
    val nome: String,           // propriedade imutável
    var email: String,          // propriedade mutável
    val dataNascimento: LocalDate
) {
    // Construtor secundário (raro — prefira valores padrão no primário)
    constructor(nome: String, email: String) : this(nome, email, LocalDate.now())

    // Bloco de inicialização
    init {
        require(email.contains("@")) { "E-mail inválido: $email" }
    }

    // Propriedade calculada
    val idade: Int
        get() = Period.between(dataNascimento, LocalDate.now()).years

    // Método
    fun apresentar() = "Olá, sou $nome e tenho $idade anos."

    // toString automático não existe — implemente se precisar
    override fun toString() = "Pessoa(nome='$nome', email='$email')"
}
```

---

### Data Classes

Gera automaticamente `equals()`, `hashCode()`, `toString()` e `copy()`:

```kotlin
data class Endereco(
    val rua: String,
    val numero: String,
    val cidade: String,
    val estado: String,
    val cep: String
)

data class Cliente(
    val id: Long,
    val nome: String,
    val email: String,
    val endereco: Endereco
)

// Destructuring
val (id, nome, email) = cliente

// copy com campos modificados
val novoEndereco = cliente.endereco.copy(cidade = "Campinas")
val clienteAtualizado = cliente.copy(endereco = novoEndereco)
```

---

### Sealed Classes e Sealed Interfaces

Representam hierarquias fechadas — o `when` garante exaustividade em tempo de compilação:

```kotlin
// Resultado de operação — alternativa a exceções ou null
sealed class Resultado<out T> {
    data class Sucesso<T>(val dados: T) : Resultado<T>()
    data class Erro(val mensagem: String, val causa: Throwable? = null) : Resultado<Nothing>()
    object Carregando : Resultado<Nothing>()
}

// O compilador exige que todos os casos sejam cobertos
fun tratarResultado(resultado: Resultado<Produto>) = when (resultado) {
    is Resultado.Sucesso   -> println("Produto: ${resultado.dados.nome}")
    is Resultado.Erro      -> println("Erro: ${resultado.mensagem}")
    Resultado.Carregando   -> println("Aguarde...")
    // sem 'else' — compilador valida exaustividade
}

// Uso em serviço
fun buscarProduto(id: Long): Resultado<Produto> {
    return try {
        val produto = repositorio.findById(id)
            ?: return Resultado.Erro("Produto não encontrado: id=$id")
        Resultado.Sucesso(produto)
    } catch (e: Exception) {
        Resultado.Erro("Falha ao buscar produto", e)
    }
}
```

---

### Object e Companion Object

```kotlin
// object — singleton thread-safe (equivalente a classe com métodos static em Java)
object ConfiguracaoApp {
    val versao = "1.0.0"
    val ambiente = System.getenv("APP_ENV") ?: "desenvolvimento"

    fun estaEmProducao() = ambiente == "producao"
}

// Companion object — métodos e propriedades estáticas associadas a uma classe
class UsuarioMapper {
    companion object {
        fun deDto(dto: CriarUsuarioDto): Usuario =
            Usuario(nome = dto.nome, email = dto.email, senha = dto.senha)

        // factory method idiomático em Kotlin
        fun de(entidade: UsuarioEntity): UsuarioDto =
            UsuarioDto(id = entidade.id!!, nome = entidade.nome, email = entidade.email)
    }
}

// Uso — sem instanciar a classe
val usuario = UsuarioMapper.deDto(dto)
```

---

## Coroutines — Programação Assíncrona

Coroutines são a solução Kotlin para código assíncrono sem callbacks ou Futures explícitos. No Spring Boot, o suporte é via WebFlux com `kotlinx-coroutines-reactor`.

```kotlin
// build.gradle.kts — adicionar dependências
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
    // Para Spring WebFlux + Coroutines:
    implementation("org.springframework.boot:spring-boot-starter-webflux")
}
```

```kotlin
// suspend fun — função que pode ser suspensa sem bloquear thread
suspend fun buscarProduto(id: Long): Produto {
    delay(100)  // suspende, não bloqueia (equivalente a Thread.sleep assíncrono)
    return repositorio.findById(id)
}

// Lançar coroutines
import kotlinx.coroutines.*

fun main() = runBlocking {  // ponte entre código síncrono e coroutines
    // launch — inicia coroutine sem retorno
    val job = launch {
        println("Coroutine executando em: ${Thread.currentThread().name}")
    }

    // async/await — coroutine com retorno
    val resultado1 = async { buscarProduto(1L) }
    val resultado2 = async { buscarProduto(2L) }

    // Executa em paralelo, aguarda os dois
    val produto1 = resultado1.await()
    val produto2 = resultado2.await()

    job.join()
}

// Controller Spring WebFlux com Coroutines (mais legível que Reactor)
@RestController
@RequestMapping("/api/produtos")
class ProdutoController(private val servico: ProdutoService) {

    @GetMapping("/{id}")
    suspend fun buscarPorId(@PathVariable id: Long): ProdutoDto =
        servico.buscarPorId(id)

    @GetMapping
    suspend fun listar(): List<ProdutoDto> =
        servico.listar()
}
```

> **vs. Java CompletableFuture/Reactor:** Coroutines permitem escrever código assíncrono com a aparência de código síncrono, sem callbacks aninhados. O `suspend` modifier marca onde a suspensão pode ocorrer, e o compilador gera a máquina de estados internamente.

---

### Spring WebFlux com Coroutines — Stack Completo

O Spring WebFlux integrado a coroutines e R2DBC oferece I/O totalmente não-bloqueante sem o modelo de programação reativo do Reactor (sem `Mono`/`Flux` explícitos no código da aplicação).

```kotlin
// build.gradle.kts
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("org.springframework.boot:spring-boot-starter-data-r2dbc")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
    runtimeOnly("org.postgresql:r2dbc-postgresql")
}
```

```yaml
# application.yml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/minha_api
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}
```

```kotlin
// Entidade R2DBC — sem JPA, sem Hibernate
@Table("produtos")
data class Produto(
    @Id val id: Long? = null,
    val nome: String,
    val descricao: String,
    val preco: Double,
    val estoque: Int = 0
)

// Repository reativo — Spring Data R2DBC gera a implementação
interface ProdutoRepository : CoroutineCrudRepository<Produto, Long> {
    suspend fun findByNomeContainingIgnoreCase(nome: String): List<Produto>
    fun findByPrecoLessThan(preco: Double): Flow<Produto>  // Flow = stream assíncrono
}
```

```kotlin
@Service
class ProdutoService(private val repositorio: ProdutoRepository) {

    suspend fun buscarPorId(id: Long): Produto =
        repositorio.findById(id) ?: throw NoSuchElementException("Produto não encontrado: id=$id")

    suspend fun listar(): List<Produto> =
        repositorio.findAll().toList()

    // Flow — processamento de grandes volumes linha a linha sem carregar tudo em memória
    fun listarFlow(): Flow<ProdutoDto> =
        repositorio.findAll().map { ProdutoDto(it.id!!, it.nome, it.preco) }

    suspend fun criar(dto: CriarProdutoDto): Produto =
        repositorio.save(Produto(nome = dto.nome, descricao = dto.descricao, preco = dto.preco))

    // Buscas paralelas — async lança as duas coroutines simultaneamente
    suspend fun buscarParalelo(id1: Long, id2: Long): Pair<Produto, Produto> =
        coroutineScope {
            val p1 = async { buscarPorId(id1) }
            val p2 = async { buscarPorId(id2) }
            p1.await() to p2.await()
        }
}
```

```kotlin
@RestController
@RequestMapping("/api/v1/produtos")
class ProdutoController(private val servico: ProdutoService) {

    // suspend fun — o Spring WebFlux executa no contexto de coroutines automaticamente
    @GetMapping("/{id}")
    suspend fun buscarPorId(@PathVariable id: Long): Produto =
        servico.buscarPorId(id)

    @GetMapping
    suspend fun listar(): List<Produto> =
        servico.listar()

    // Flow no retorno — o Spring serializa elemento a elemento (streaming JSON)
    @GetMapping("/stream", produces = [MediaType.APPLICATION_NDJSON_VALUE])
    fun listarStream(): Flow<ProdutoDto> =
        servico.listarFlow()

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    suspend fun criar(@Valid @RequestBody dto: CriarProdutoDto): Produto =
        servico.criar(dto)
}
```

```kotlin
// Tratamento de erros em contexto reativo com coroutines
@RestControllerAdvice
class TratadorDeExcecoesReativo {

    @ExceptionHandler(NoSuchElementException::class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    fun handleNotFound(ex: NoSuchElementException): Map<String, String?> =
        mapOf("erro" to ex.message)

    // CancellationException NÃO deve ser capturada — sinaliza cancelamento normal de coroutine
    @ExceptionHandler(Exception::class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    fun handleGeneric(ex: Exception): Map<String, String> =
        mapOf("erro" to "Erro interno no servidor")
}
```

> **`Flow` vs `List`:** Use `Flow<T>` quando o volume de dados for grande ou quando quiser iniciar a resposta antes de processar tudo (streaming). Use `List<T>` para conjuntos pequenos onde a simplicidade importa mais.

---

## Spring Boot com Kotlin

### Ponto de Entrada

```kotlin
package br.com.exemplo.minhaapi

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MinhaApiApplication

fun main(args: Array<String>) {
    runApplication<MinhaApiApplication>(*args)
}
```

---

### DTOs com Bean Validation

```kotlin
import jakarta.validation.constraints.*

data class CriarProdutoDto(
    @field:NotBlank(message = "Nome é obrigatório")
    @field:Size(min = 2, max = 100)
    val nome: String,

    @field:NotBlank
    val descricao: String,

    @field:Positive(message = "Preço deve ser positivo")
    val preco: Double,

    @field:Min(0)
    val estoque: Int = 0
)

data class ProdutoDto(
    val id: Long,
    val nome: String,
    val descricao: String,
    val preco: Double,
    val estoque: Int
)
```

> **Atenção:** Use `@field:` ao anotar propriedades de data classes com Bean Validation. Em Kotlin, a anotação precisa especificar o alvo `field` para ser colocada no campo gerado, não no parâmetro do construtor.

---

### Configuração Tipada com @ConfigurationProperties

Em vez de múltiplos `@Value` espalhados, agrupe propriedades relacionadas em uma data class validada — o padrão idiomático Kotlin/Spring para configuração.

```yaml
# application.yml
app:
  email:
    host: smtp.example.com
    porta: 587
    remetente: noreply@example.com
    timeout-ms: 5000
  paginacao:
    tamanho-padrao: 20
    tamanho-maximo: 100
```

```kotlin
@ConfigurationProperties(prefix = "app.email")
@Validated
data class EmailProperties(
    @field:NotBlank val host: String,
    @field:Min(1) val porta: Int,
    @field:Email val remetente: String,
    val timeoutMs: Long = 5000
)

@ConfigurationProperties(prefix = "app.paginacao")
data class PaginacaoProperties(
    val tamanhoPadrao: Int = 20,
    val tamanhoMaximo: Int = 100
)
```

```kotlin
// build.gradle.kts — gera metadados para autocompletar no application.yml
dependencies {
    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")
}
```

```kotlin
// Habilitar as classes de configuração na aplicação
@SpringBootApplication
@EnableConfigurationProperties(EmailProperties::class, PaginacaoProperties::class)
class MinhaApiApplication

fun main(args: Array<String>) {
    runApplication<MinhaApiApplication>(*args)
}
```

```kotlin
// Injeção via construtor — mesmo padrão de qualquer outro bean Spring
@Service
class EmailService(private val config: EmailProperties) {

    fun enviar(destinatario: String, assunto: String, corpo: String) {
        println("Enviando de ${config.remetente} via ${config.host}:${config.porta}")
    }
}

// Controller usando PaginacaoProperties para limitar page size
@RestController
@RequestMapping("/api/v1/produtos")
class ProdutoController(
    private val servico: ProdutoService,
    private val paginacao: PaginacaoProperties
) {
    @GetMapping
    fun listar(
        @RequestParam(defaultValue = "0") pagina: Int,
        @RequestParam tamanho: Int?
    ): Page<ProdutoDto> {
        val tamanhoPagina = (tamanho ?: paginacao.tamanhoPadrao)
            .coerceAtMost(paginacao.tamanhoMaximo)
        return servico.listar(PageRequest.of(pagina, tamanhoPagina))
    }
}
```

> **vs. `@Value`:** `@ConfigurationProperties` é type-safe, documenta os campos com autocompletar no IDE (via `configuration-processor`), suporta validação com Bean Validation na própria classe e agrupa configurações relacionadas em um único lugar testável.

---

### Controller REST

```kotlin
@RestController
@RequestMapping("/api/v1/produtos")
@Validated
class ProdutoController(private val servico: ProdutoService) {

    @GetMapping
    fun listar(
        @RequestParam(defaultValue = "0") pagina: Int,
        @RequestParam(defaultValue = "20") tamanho: Int
    ): Page<ProdutoDto> = servico.listar(PageRequest.of(pagina, tamanho))

    @GetMapping("/{id}")
    fun buscarPorId(@PathVariable id: Long): ProdutoDto =
        servico.buscarPorId(id)

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    fun criar(@Valid @RequestBody dto: CriarProdutoDto): ProdutoDto =
        servico.criar(dto)

    @PutMapping("/{id}")
    fun atualizar(
        @PathVariable id: Long,
        @Valid @RequestBody dto: CriarProdutoDto
    ): ProdutoDto = servico.atualizar(id, dto)

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    fun remover(@PathVariable id: Long) = servico.remover(id)
}
```

---

### Service

```kotlin
@Service
@Transactional
class ProdutoService(
    private val repositorio: ProdutoRepository,
    private val mapper: ProdutoMapper
) {

    @Transactional(readOnly = true)
    fun listar(paginacao: Pageable): Page<ProdutoDto> =
        repositorio.findAll(paginacao).map { mapper.paraDto(it) }

    @Transactional(readOnly = true)
    fun buscarPorId(id: Long): ProdutoDto =
        repositorio.findById(id)
            .map { mapper.paraDto(it) }
            .orElseThrow { NoSuchElementException("Produto não encontrado: id=$id") }

    fun criar(dto: CriarProdutoDto): ProdutoDto {
        val produto = mapper.paraEntidade(dto)
        return mapper.paraDto(repositorio.save(produto))
    }

    fun atualizar(id: Long, dto: CriarProdutoDto): ProdutoDto {
        val produto = repositorio.findById(id)
            .orElseThrow { NoSuchElementException("Produto não encontrado: id=$id") }

        val atualizado = produto.copy(
            nome = dto.nome,
            descricao = dto.descricao,
            preco = dto.preco,
            estoque = dto.estoque
        )
        return mapper.paraDto(repositorio.save(atualizado))
    }

    fun remover(id: Long) {
        if (!repositorio.existsById(id)) {
            throw NoSuchElementException("Produto não encontrado: id=$id")
        }
        repositorio.deleteById(id)
    }
}
```

---

### Tratamento Global de Erros

```kotlin
@RestControllerAdvice
class TratadorDeExcecoes {

    @ExceptionHandler(NoSuchElementException::class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    fun handleNotFound(ex: NoSuchElementException): Map<String, String> =
        mapOf("erro" to (ex.message ?: "Recurso não encontrado"))

    @ExceptionHandler(MethodArgumentNotValidException::class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    fun handleValidation(ex: MethodArgumentNotValidException): Map<String, Any> {
        val erros = ex.bindingResult.fieldErrors
            .associate { it.field to (it.defaultMessage ?: "inválido") }
        return mapOf("erros" to erros)
    }

    @ExceptionHandler(Exception::class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    fun handleGeneric(ex: Exception): Map<String, String> {
        log.error("Erro inesperado", ex)
        return mapOf("erro" to "Erro interno no servidor")
    }

    companion object {
        private val log = LoggerFactory.getLogger(TratadorDeExcecoes::class.java)
    }
}
```

---

## Integração com Banco de Dados

### Entidade JPA

```kotlin
import jakarta.persistence.*

@Entity
@Table(name = "produtos")
class Produto(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false, length = 100)
    var nome: String,

    @Column(nullable = false, columnDefinition = "TEXT")
    var descricao: String,

    @Column(nullable = false, precision = 10, scale = 2)
    var preco: Double,

    @Column(nullable = false)
    var estoque: Int = 0,

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id")
    var categoria: Categoria? = null,

    @CreationTimestamp
    @Column(name = "criado_em", updatable = false)
    val criadoEm: LocalDateTime? = null,

    @UpdateTimestamp
    @Column(name = "atualizado_em")
    var atualizadoEm: LocalDateTime? = null
)
```

> **Importante:** Em Kotlin, entidades JPA **devem usar `class`**, não `data class`. O Hibernate precisa criar instâncias via construtor sem argumentos e modificar campos. O plugin `kotlin("plugin.jpa")` gera esses construtores automaticamente. Use `var` para campos que o Hibernate irá modificar.

---

### Repository com Spring Data

```kotlin
interface ProdutoRepository : JpaRepository<Produto, Long> {

    // Query derivada do nome do método (igual a Java)
    fun findByNomeContainingIgnoreCase(nome: String, pageable: Pageable): Page<Produto>

    fun findByCategoriaId(categoriaId: Long): List<Produto>

    fun existsByNomeAndCategoriaId(nome: String, categoriaId: Long): Boolean

    // JPQL com @Query
    @Query("SELECT p FROM Produto p WHERE p.preco BETWEEN :min AND :max ORDER BY p.preco")
    fun findByFaixaDePreco(
        @Param("min") min: Double,
        @Param("max") max: Double,
        pageable: Pageable
    ): Page<Produto>

    // Projeção — retorna apenas campos necessários
    @Query("SELECT p.id AS id, p.nome AS nome, p.preco AS preco FROM Produto p WHERE p.estoque > 0")
    fun findDisponiveis(): List<ProdutoResumo>
}

// Interface de projeção
interface ProdutoResumo {
    val id: Long
    val nome: String
    val preco: Double
}
```

---

### Configuração do Banco de Dados

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/minha_api
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}

  jpa:
    hibernate:
      ddl-auto: validate        # use 'create-drop' em testes, 'validate' em produção
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
        default_batch_fetch_size: 16   # otimiza N+1 em coleções lazy

  flyway:
    enabled: true
    locations: classpath:db/migration
```

---

### Migrações com Flyway

```sql
-- src/main/resources/db/migration/V1__criar_tabela_produtos.sql
CREATE TABLE categorias (
    id      BIGSERIAL PRIMARY KEY,
    nome    VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE produtos (
    id            BIGSERIAL PRIMARY KEY,
    nome          VARCHAR(100) NOT NULL,
    descricao     TEXT NOT NULL,
    preco         NUMERIC(10,2) NOT NULL,
    estoque       INTEGER NOT NULL DEFAULT 0,
    categoria_id  BIGINT REFERENCES categorias(id),
    criado_em     TIMESTAMP NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

### Mapper Kotlin

```kotlin
@Component
class ProdutoMapper {

    fun paraDto(produto: Produto): ProdutoDto = ProdutoDto(
        id = produto.id!!,
        nome = produto.nome,
        descricao = produto.descricao,
        preco = produto.preco,
        estoque = produto.estoque
    )

    fun paraEntidade(dto: CriarProdutoDto): Produto = Produto(
        nome = dto.nome,
        descricao = dto.descricao,
        preco = dto.preco,
        estoque = dto.estoque
    )
}
```

---

## Spring Security com Kotlin

O Spring Security 6.x oferece um **Kotlin DSL** para configuração fluente e type-safe, eliminando o boilerplate das lambdas Java.

### Configuração com JWT

```kotlin
// Dependências adicionais no build.gradle.kts
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-security")
    // spring-security-oauth2-jose inclui o Nimbus JOSE+JWT transitivamente
    implementation("org.springframework.security:spring-security-oauth2-jose")
}
```

```yaml
# application.yml
app:
  jwt:
    private-key: classpath:certs/private.pem
    public-key: classpath:certs/public.pem
    expiracao-ms: 86400000  # 24 horas
    issuer: https://meuapp.com.br
```

```bash
# Gerar par de chaves RSA (executar uma vez; armazene a chave privada fora do repositório em produção)
mkdir -p src/main/resources/certs
openssl genrsa -out src/main/resources/certs/private.pem 2048
openssl rsa -in src/main/resources/certs/private.pem -pubout -out src/main/resources/certs/public.pem
```

> **Atenção:** Em produção, nunca commite a chave privada no repositório. Injete-a via variável de ambiente ou secret manager (AWS Secrets Manager, HashiCorp Vault, etc.). O Spring Boot converte automaticamente arquivos PEM para `RSAPrivateKey`/`RSAPublicKey` quando `spring-security-oauth2-jose` está no classpath.

---

### Serviço JWT

```kotlin
import com.nimbusds.jose.*
import com.nimbusds.jose.crypto.*
import com.nimbusds.jose.jwk.RSAKey
import com.nimbusds.jwt.*
import java.security.interfaces.RSAPrivateKey
import java.security.interfaces.RSAPublicKey
import java.time.Instant
import java.util.*

@Service
class JwtService(
    @Value("\${app.jwt.private-key}") private val privateKey: RSAPrivateKey,
    @Value("\${app.jwt.public-key}")  private val publicKey: RSAPublicKey,
    @Value("\${app.jwt.expiracao-ms}") private val expiracaoMs: Long,
    @Value("\${app.jwt.issuer}")      private val issuer: String
) {
    // RSAKey combina chave pública e privada; keyID identifica o par em JWKs
    private val rsaKey: RSAKey by lazy {
        RSAKey.Builder(publicKey)
            .privateKey(privateKey)
            .keyID(UUID.randomUUID().toString())
            .build()
    }

    fun gerarToken(usuario: UserDetails): String {
        val agora = Instant.now()

        val claims = JWTClaimsSet.Builder()
            .subject(usuario.username)
            .issuer(issuer)
            .issueTime(Date.from(agora))
            .expirationTime(Date.from(agora.plusMillis(expiracaoMs)))
            .jwtID(UUID.randomUUID().toString())
            .claim("roles", usuario.authorities.map { it.authority })
            .build()

        val cabecalho = JWSHeader.Builder(JWSAlgorithm.RS256)
            .keyID(rsaKey.keyID)
            .type(JOSEObjectType.JWT)
            .build()

        return SignedJWT(cabecalho, claims)
            .also { it.sign(RSASSASigner(rsaKey)) }
            .serialize()
    }

    fun extrairUsername(token: String): String? = runCatching {
        validarEExtrairClaims(token).subject
    }.getOrNull()

    fun isTokenValido(token: String, usuario: UserDetails): Boolean = runCatching {
        validarEExtrairClaims(token).subject == usuario.username
    }.getOrDefault(false)

    private fun validarEExtrairClaims(token: String): JWTClaimsSet {
        val jwt = SignedJWT.parse(token)
        check(jwt.verify(RSASSAVerifier(rsaKey.toRSAPublicKey()))) { "Assinatura JWT inválida" }
        val claims = jwt.jwtClaimsSet
        check(claims.expirationTime.after(Date()))  { "Token JWT expirado" }
        check(claims.issuer == issuer)              { "Issuer inválido: ${claims.issuer}" }
        return claims
    }
}
```

---

### Filtro JWT

```kotlin
@Component
class JwtAuthenticationFilter(
    private val jwtService: JwtService,
    private val userDetailsService: UserDetailsService
) : OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        val token = request.getHeader("Authorization")
            ?.takeIf { it.startsWith("Bearer ") }
            ?.removePrefix("Bearer ")

        if (token != null && SecurityContextHolder.getContext().authentication == null) {
            val username = jwtService.extrairUsername(token)

            if (username != null) {
                val usuario = userDetailsService.loadUserByUsername(username)
                if (jwtService.isTokenValido(token, usuario)) {
                    val auth = UsernamePasswordAuthenticationToken(
                        usuario, null, usuario.authorities
                    ).also { it.details = WebAuthenticationDetailsSource().buildDetails(request) }
                    SecurityContextHolder.getContext().authentication = auth
                }
            }
        }

        filterChain.doFilter(request, response)
    }
}
```

---

### SecurityConfig com Kotlin DSL

```kotlin
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
class SecurityConfig(
    private val jwtFiltro: JwtAuthenticationFilter,
    private val userDetailsService: UserDetailsService
) {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            csrf { disable() }
            cors { }
            sessionManagement {
                sessionCreationPolicy = SessionCreationPolicy.STATELESS
            }
            authorizeHttpRequests {
                authorize("/api/v1/auth/**", permitAll)
                authorize("/api/v1/produtos", permitAll)
                authorize(HttpMethod.GET, "/api/v1/produtos/**", permitAll)
                authorize("/api/v1/admin/**", hasRole("ADMIN"))
                authorize(anyRequest, authenticated)
            }
            addFilterBefore<UsernamePasswordAuthenticationFilter>(jwtFiltro)
        }
        return http.build()
    }

    @Bean
    fun passwordEncoder(): PasswordEncoder = BCryptPasswordEncoder()

    @Bean
    fun authenticationManager(config: AuthenticationConfiguration): AuthenticationManager =
        config.authenticationManager

    @Bean
    fun corsConfigurationSource(): CorsConfigurationSource =
        UrlBasedCorsConfigurationSource().apply {
            registerCorsConfiguration("/**", CorsConfiguration().apply {
                allowedOrigins = listOf("http://localhost:3000", "https://meuapp.com.br")
                allowedMethods = listOf("GET", "POST", "PUT", "DELETE", "OPTIONS")
                allowedHeaders = listOf("*")
                allowCredentials = true
            })
        }
}
```

> **Kotlin DSL vs. Java lambda:** A configuração com Kotlin DSL (bloco `http { ... }`) é mais legível que as lambdas Java (`http.csrf(csrf -> csrf.disable())`). O DSL usa extension functions do Spring Security Kotlin.

---

### Entidade de Usuário e UserDetails

```kotlin
@Entity
@Table(name = "usuarios")
class Usuario(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false, length = 100)
    var nome: String,

    @Column(nullable = false, unique = true)
    var email: String,

    @Column(nullable = false)
    var senha: String,

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    var perfil: Perfil = Perfil.USER,

    var ativo: Boolean = true
)

enum class Perfil { USER, EDITOR, ADMIN }

@Service
class UsuarioDetailsService(
    private val repositorio: UsuarioRepository
) : UserDetailsService {

    override fun loadUserByUsername(email: String): UserDetails =
        repositorio.findByEmail(email)
            ?: throw UsernameNotFoundException("Usuário não encontrado: $email")
            .let { usuario ->
                User.builder()
                    .username(usuario.email)
                    .password(usuario.senha)
                    .roles(usuario.perfil.name)
                    .accountExpired(!usuario.ativo)
                    .build()
            }
}
```

---

### Controller de Autenticação

```kotlin
data class LoginDto(
    @field:NotBlank val email: String,
    @field:NotBlank val senha: String
)

data class TokenDto(val token: String, val tipo: String = "Bearer")

@RestController
@RequestMapping("/api/v1/auth")
class AuthController(
    private val authManager: AuthenticationManager,
    private val jwtService: JwtService,
    private val repositorio: UsuarioRepository
) {

    @PostMapping("/login")
    fun login(@Valid @RequestBody dto: LoginDto): TokenDto {
        val auth = authManager.authenticate(
            UsernamePasswordAuthenticationToken(dto.email, dto.senha)
        )
        val usuario = auth.principal as UserDetails
        return TokenDto(jwtService.gerarToken(usuario))
    }

    @PostMapping("/registrar")
    @ResponseStatus(HttpStatus.CREATED)
    fun registrar(@Valid @RequestBody dto: CriarUsuarioDto): Map<String, String> {
        if (repositorio.existsByEmail(dto.email)) {
            throw IllegalArgumentException("E-mail já cadastrado")
        }
        repositorio.save(
            Usuario(nome = dto.nome, email = dto.email, senha = passwordEncoder.encode(dto.senha))
        )
        return mapOf("mensagem" to "Usuário criado com sucesso")
    }
}
```

---

### Method Security com @PreAuthorize

```kotlin
@Service
class AdminService(private val usuarioRepositorio: UsuarioRepository) {

    // Acesso restrito a ADMIN
    @PreAuthorize("hasRole('ADMIN')")
    fun listarTodosUsuarios(): List<UsuarioDto> =
        usuarioRepositorio.findAll().map { UsuarioMapper.de(it) }

    // Dono do recurso OU admin pode acessar
    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    fun buscarPerfil(id: Long): UsuarioDto =
        usuarioRepositorio.findById(id)
            .map { UsuarioMapper.de(it) }
            .orElseThrow { NoSuchElementException("Usuário não encontrado") }

    // SpEL com anotação customizada
    @PreAuthorize("@permissaoService.podeEditar(#id, authentication)")
    fun editarUsuario(id: Long, dto: EditarUsuarioDto): UsuarioDto { TODO() }
}
```

---

## Testes

### Teste de Unidade com MockK

MockK é a biblioteca de mocks nativa para Kotlin, preferida ao Mockito em projetos Kotlin. Ela entende a semântica do Kotlin: suporta classes `final` (o padrão em Kotlin) sem configuração adicional, trata `suspend fun` nativamente e elimina o `@Suppress("unchecked_cast")` comum com Mockito.

```kotlin
// build.gradle.kts
dependencies {
    testImplementation("io.mockk:mockk:1.14.0")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test")
}
```

```kotlin
class ProdutoServiceTest {

    // mockk<T>() — cria mock de qualquer classe, incluindo final
    private val repositorio: ProdutoRepository = mockk()
    private val mapper: ProdutoMapper = mockk()
    private val servico = ProdutoService(repositorio, mapper)

    @Test
    fun `buscarPorId deve retornar dto quando produto encontrado`() {
        // Arrange
        val produto = Produto(id = 1L, nome = "Notebook", descricao = "Desc", preco = 3500.0)
        val dto = ProdutoDto(id = 1L, nome = "Notebook", descricao = "Desc", preco = 3500.0, estoque = 0)

        every { repositorio.findById(1L) } returns Optional.of(produto)
        every { mapper.paraDto(produto) } returns dto

        // Act
        val resultado = servico.buscarPorId(1L)

        // Assert
        assertThat(resultado.nome).isEqualTo("Notebook")
        verify(exactly = 1) { repositorio.findById(1L) }
    }

    @Test
    fun `buscarPorId deve lançar exceção quando produto não existe`() {
        every { repositorio.findById(99L) } returns Optional.empty()

        assertThrows<NoSuchElementException> { servico.buscarPorId(99L) }
            .also { assertThat(it.message).contains("99") }
    }

    @Test
    fun `criar deve salvar e retornar dto`() {
        val dto = CriarProdutoDto(nome = "Mouse", descricao = "USB", preco = 80.0)
        val entidade = Produto(nome = "Mouse", descricao = "USB", preco = 80.0)
        val salvo = entidade.copy(id = 5L)
        val dtoRetorno = ProdutoDto(id = 5L, nome = "Mouse", descricao = "USB", preco = 80.0, estoque = 0)

        every { mapper.paraEntidade(dto) } returns entidade
        every { repositorio.save(entidade) } returns salvo
        every { mapper.paraDto(salvo) } returns dtoRetorno

        val resultado = servico.criar(dto)

        assertThat(resultado.id).isEqualTo(5L)
        verify { repositorio.save(entidade) }
    }
}
```

```kotlin
// Teste de suspend fun com MockK + kotlinx-coroutines-test
class ProdutoServiceCoroutineTest {

    private val repositorio: ProdutoRepository = mockk()
    private val servico = ProdutoService(repositorio)

    @Test
    fun `buscarPorId suspend deve retornar produto`() = runTest {
        val produto = Produto(id = 1L, nome = "Teclado", descricao = "Mecânico", preco = 350.0)

        // coEvery — equivalente de every para suspend fun
        coEvery { repositorio.findById(1L) } returns produto

        val resultado = servico.buscarPorId(1L)

        assertThat(resultado.nome).isEqualTo("Teclado")
        // coVerify — equivalente de verify para suspend fun
        coVerify(exactly = 1) { repositorio.findById(1L) }
    }
}
```

| MockK | Mockito (equivalente) |
|---|---|
| `mockk<T>()` | `mock(T::class.java)` |
| `every { obj.metodo() } returns valor` | `whenever(obj.metodo()).thenReturn(valor)` |
| `every { obj.metodo() } throws Ex()` | `whenever(obj.metodo()).thenThrow(Ex())` |
| `verify { obj.metodo() }` | `verify(obj).metodo()` |
| `coEvery { ... }` | não tem equivalente direto para `suspend` |
| `coVerify { ... }` | não tem equivalente direto para `suspend` |
| `slot<T>()` + `capture(slot)` | `ArgumentCaptor.forClass(T::class.java)` |

> **Nomes de testes em backticks:** Em Kotlin, métodos de teste podem usar frases legíveis entre acentos graves — `` `buscarPorId deve lançar exceção quando produto não existe` `` — sem a necessidade de camelCase ou anotações extras.

---

### Teste de Integração com Spring Boot Test

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Transactional
class ProdutoControllerIT {

    @Autowired
    private lateinit var mockMvc: MockMvc

    @Autowired
    private lateinit var objectMapper: ObjectMapper

    @Autowired
    private lateinit var repositorio: ProdutoRepository

    @Test
    @WithMockUser(roles = ["ADMIN"])
    fun `POST deve criar produto e retornar 201`() {
        val dto = CriarProdutoDto(nome = "Fone", descricao = "Bluetooth", preco = 199.0)

        mockMvc.post("/api/v1/produtos") {
            contentType = MediaType.APPLICATION_JSON
            content = objectMapper.writeValueAsString(dto)
        }.andExpect {
            status { isCreated() }
            jsonPath("$.nome") { value("Fone") }
            jsonPath("$.preco") { value(199.0) }
        }
    }

    @Test
    fun `GET deve retornar 404 para produto inexistente`() {
        mockMvc.get("/api/v1/produtos/99999")
            .andExpect { status { isNotFound() } }
    }
}
```

---

### Teste de Segurança

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class SecurityIT {

    @Autowired
    private lateinit var mockMvc: MockMvc

    @Test
    fun `endpoint protegido deve retornar 401 sem token`() {
        mockMvc.get("/api/v1/admin/usuarios")
            .andExpect { status { isUnauthorized() } }
    }

    @Test
    @WithMockUser(roles = ["USER"])
    fun `endpoint admin deve retornar 403 para perfil USER`() {
        mockMvc.get("/api/v1/admin/usuarios")
            .andExpect { status { isForbidden() } }
    }

    @Test
    @WithMockUser(roles = ["ADMIN"])
    fun `endpoint admin deve retornar 200 para perfil ADMIN`() {
        mockMvc.get("/api/v1/admin/usuarios")
            .andExpect { status { isOk() } }
    }
}
```

---

## Referências

### Documentação Oficial

- [kotlinlang.org](https://kotlinlang.org/docs/home.html) — Documentação oficial do Kotlin
- [spring.io/guides/tutorials/spring-boot-kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin/) — Tutorial oficial Spring Boot + Kotlin
- [kotlinlang.org/docs/coroutines-overview.html](https://kotlinlang.org/docs/coroutines-overview.html) — Guia de Coroutines
- [start.spring.io](https://start.spring.io) — Spring Initializr

### Documentos Relacionados neste Repositório

- [Dicas-Spring-Inicial.md](Dicas-Spring-Inicial.md) — IoC/DI e camada de serviço
- [Dicas-Spring-MVC-REST.md](Dicas-Spring-MVC-REST.md) — APIs REST com Spring MVC
- [Dicas-JPA-Hibernate-Modelagem-Entities.md](Dicas-JPA-Hibernate-Modelagem-Entities.md) — Modelagem JPA
- [Dicas-JPA-Hibernate-Queries.md](Dicas-JPA-Hibernate-Queries.md) — Queries JPQL e Spring Data
- [Dicas-Spring-Security.md](Dicas-Spring-Security.md) — Spring Security completo (com exemplos Java)

### Livros e Cursos

- *Kotlin in Action* (Jemerov & Afonina) — referência definitiva da linguagem
- *Spring Boot: Up and Running* (Mark Heckler) — Spring Boot moderno com exemplos Kotlin
- [Kotlin Koans](https://kotlinlang.org/docs/koans.html) — exercícios interativos online para aprender Kotlin

### Ferramentas

- [SDKMAN](https://sdkman.io) — gerenciamento de versões JVM/Kotlin
- [IntelliJ IDEA Community](https://www.jetbrains.com/idea/download/) — IDE gratuita
- [Kotlin Playground](https://play.kotlinlang.org) — ambiente online para testar código Kotlin
