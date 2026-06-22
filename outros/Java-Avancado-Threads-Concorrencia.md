# Java Avançado — Concorrência, Pattern Matching e Features Modernas

> **Baseline:** Java 25 (LTS, setembro 2025) · Funcionalidades finalizadas a partir do Java 21+
>
> **Pré-requisitos:** familiaridade com Java 21+, conceitos básicos de threads e `java.util.concurrent`.

Java 25 consolida as grandes adições de concorrência iniciadas no Project Loom (Java 21) e as features de linguagem amadurecidas ao longo das versões 22–24. Este guia está dividido em duas partes: **Parte I** cobre concorrência moderna (Virtual Threads, Structured Concurrency, Scoped Values, Channels, Locks, Atomics) e **Parte II** cobre features avançadas de linguagem (Pattern Matching, Sealed Classes, Panama FFM, Stream Gatherers e mais).

---

## Sumário

### Parte I — Concorrência Moderna

1. [Visão Geral — Evolução da Concorrência em Java](#1-visão-geral--evolução-da-concorrência-em-java)
2. [Virtual Threads](#2-virtual-threads)
   - [2.1 Conceito e Motivação](#21-conceito-e-motivação)
   - [2.2 Criando Virtual Threads](#22-criando-virtual-threads)
   - [2.3 Executors com Virtual Threads](#23-executors-com-virtual-threads)
   - [2.4 Virtual Threads em Servidores Web](#24-virtual-threads-em-servidores-web)
   - [2.5 Pinning — Quando a Virtual Thread Bloqueia a Carrier](#25-pinning--quando-a-virtual-thread-bloqueia-a-carrier)
   - [2.6 Boas Práticas com Virtual Threads](#26-boas-práticas-com-virtual-threads)
3. [Structured Concurrency](#3-structured-concurrency)
   - [3.1 O Problema da Concorrência Não-Estruturada](#31-o-problema-da-concorrência-não-estruturada)
   - [3.2 StructuredTaskScope — Conceito](#32-structuredtaskscope--conceito)
   - [3.3 ShutdownOnFailure — Falha Rápida](#33-shutdownonfailure--falha-rápida)
   - [3.4 ShutdownOnSuccess — Primeira Resposta Vence](#34-shutdownonsuccess--primeira-resposta-vence)
   - [3.5 StructuredTaskScope Customizado](#35-structuredtaskscope-customizado)
   - [3.6 Composição e Aninhamento de Scopes](#36-composição-e-aninhamento-de-scopes)
4. [Scoped Values — Compartilhamento de Dados entre Threads](#4-scoped-values--compartilhamento-de-dados-entre-threads)
   - [4.1 O Problema do ThreadLocal](#41-o-problema-do-threadlocal)
   - [4.2 ScopedValue — Conceito](#42-scopedvalue--conceito)
   - [4.3 Herança Automática em StructuredTaskScope](#43-herança-automática-em-structuredtaskscope)
   - [4.4 Rebinding — Sobrescrever Valor em Escopo Filho](#44-rebinding--sobrescrever-valor-em-escopo-filho)
   - [4.5 ScopedValue vs ThreadLocal — Comparação](#45-scopedvalue-vs-threadlocal--comparação)
5. [Channels — Comunicação entre Threads](#5-channels--comunicação-entre-threads)
   - [5.1 Conceito de Channel](#51-conceito-de-channel)
   - [5.2 BlockingQueue como Channel](#52-blockingqueue-como-channel)
   - [5.3 Channel Unbuffered (Rendez-vous)](#53-channel-unbuffered-rendez-vous)
   - [5.4 Channel com Capacidade (Buffered)](#54-channel-com-capacidade-buffered)
   - [5.5 Fan-Out — Múltiplos Consumidores](#55-fan-out--múltiplos-consumidores)
   - [5.6 Fan-In — Múltiplos Produtores](#56-fan-in--múltiplos-produtores)
   - [5.7 Pipeline de Channels](#57-pipeline-de-channels)
   - [5.8 Select — Multiplexação de Channels](#58-select--multiplexação-de-channels)
6. [Padrões Avançados de Concorrência](#6-padrões-avançados-de-concorrência)
   - [6.1 Rate Limiting com Semaphore](#61-rate-limiting-com-semaphore)
   - [6.2 Timeout e Cancelamento](#62-timeout-e-cancelamento)
   - [6.3 Producer-Consumer com Backpressure](#63-producer-consumer-com-backpressure)
   - [6.4 Fork-Join com Structured Concurrency](#64-fork-join-com-structured-concurrency)
7. [Locks Avançados — ReentrantLock, ReadWriteLock e StampedLock](#7-locks-avançados--reentrantlock-readwritelock-e-stampedlock)
   - [7.2 ReentrantLock — Substituto do synchronized](#72-reentrantlock--substituto-do-synchronized)
   - [7.3 ReadWriteLock — Leituras Concorrentes](#73-readwritelock--leituras-concorrentes)
   - [7.4 StampedLock — Locking Otimista](#74-stampedlock--locking-otimista)
   - [7.5 Quando Usar Cada Lock](#75-quando-usar-cada-lock)
8. [Atomic Variables e VarHandle](#8-atomic-variables-e-varhandle)
   - [8.1 Classes Atomic — Operações Lock-Free](#81-classes-atomic--operações-lock-free)
   - [8.2 LongAdder e LongAccumulator — Alta Contenção](#82-longadder-e-longaccumulator--alta-contenção)
   - [8.3 AtomicStampedReference — Resolvendo o Problema ABA](#83-atomicstampedreference--resolvendo-o-problema-aba)
   - [8.4 VarHandle — Acesso de Baixo Nível a Variáveis](#84-varhandle--acesso-de-baixo-nível-a-variáveis)
9. [CompletableFuture vs Structured Concurrency](#9-completablefuture-vs-structured-concurrency)
   - [9.1 Quando Ainda Usar CompletableFuture](#91-quando-ainda-usar-completablefuture)
   - [9.2 Comparação Lado a Lado](#92-comparação-lado-a-lado)
10. [Integração com Spring Boot](#10-integração-com-spring-boot)
    - [10.1 Configuração de Virtual Threads no Spring Boot](#101-configuração-de-virtual-threads-no-spring-boot)
    - [10.2 Structured Concurrency em Services](#102-structured-concurrency-em-services)
    - [10.3 Propagação de Contexto com Scoped Values](#103-propagação-de-contexto-com-scoped-values)
11. [Testes de Código Concorrente](#11-testes-de-código-concorrente)
12. [Comparação com Go e Kotlin](#12-comparação-com-go-e-kotlin)

### Parte II — Features Avançadas de Linguagem

13. [Pattern Matching Avançado](#13-pattern-matching-avançado)
    - [13.2 instanceof com Pattern Variable](#132-instanceof-com-pattern-variable)
    - [13.3 Switch com Pattern Matching](#133-switch-com-pattern-matching)
    - [13.4 Guard Clauses (when)](#134-guard-clauses-when)
    - [13.5 Record Patterns — Desestruturação](#135-record-patterns--desestruturação)
    - [13.6 Unnamed Variables e Patterns (_)](#136-unnamed-variables-e-patterns-_)
14. [Sealed Classes e Records — Modelagem de Domínio](#14-sealed-classes-e-records--modelagem-de-domínio)
    - [14.1 Conceito de Sealed Classes](#141-conceito-de-sealed-classes)
    - [14.2 Exhaustiveness no Switch](#142-exhaustiveness-no-switch)
    - [14.3 Hierarquias com Múltiplos Níveis](#143-hierarquias-com-múltiplos-níveis)
    - [14.4 Sealed Classes para Resultado de Operações](#144-sealed-classes-para-resultado-de-operações)
    - [14.5 Records Avançados](#145-records-avançados)
15. [Foreign Function & Memory API (Project Panama)](#15-foreign-function--memory-api-project-panama)
    - [15.2 Alocação de Memória Off-Heap](#152-alocação-de-memória-off-heap)
    - [15.3 Chamando Funções Nativas (FFM)](#153-chamando-funções-nativas-ffm)
    - [15.4 Trabalhando com Structs Nativos](#154-trabalhando-com-structs-nativos)
    - [15.5 Exemplo Prático — Integração com Biblioteca C](#155-exemplo-prático--integração-com-biblioteca-c)
16. [Stream Gatherers](#16-stream-gatherers)
    - [16.2 Gatherers Built-in](#162-gatherers-built-in)
    - [16.3 Criando Gatherers Customizados](#163-criando-gatherers-customizados)
    - [16.4 Composição de Gatherers](#164-composição-de-gatherers)
17. [Flexible Constructor Bodies](#17-flexible-constructor-bodies)
    - [17.2 Validação Antes do super()](#172-validação-antes-do-super)
    - [17.3 Preparação de Argumentos](#173-preparação-de-argumentos)
    - [17.4 Regras e Restrições](#174-regras-e-restrições)
18. [Module Import Declarations e Outras Conveniências](#18-module-import-declarations-e-outras-conveniências)
    - [18.1 Module Import Declarations](#181-module-import-declarations)
    - [18.2 Implicitly Declared Classes (Simple Main)](#182-implicitly-declared-classes-simple-main)
    - [18.3 Markdown em JavaDoc](#183-markdown-em-javadoc)
19. [Java NIO Channels — I/O de Alto Desempenho](#19-java-nio-channels--io-de-alto-desempenho)
    - [19.1 Hierarquia de Channels](#191-hierarquia-de-channels)
    - [19.2 FileChannel — I/O de Arquivo de Alto Desempenho](#192-filechannel--io-de-arquivo-de-alto-desempenho)
    - [19.3 SocketChannel e ServerSocketChannel — I/O de Rede](#193-socketchannel-e-serversocketchannel--io-de-rede)
    - [19.4 Pipe — Channel entre Threads](#194-pipe--channel-entre-threads)
    - [19.5 AsynchronousFileChannel e AsynchronousSocketChannel](#195-asynchronousfilechannel-e-asynchronoussocketchannel)

---

# Parte I — Concorrência Moderna

---

## 1. Visão Geral — Evolução da Concorrência em Java

| Versão | Recurso | Status |
|--------|---------|--------|
| Java 1.0 | `Thread`, `synchronized`, `wait/notify` | Legado |
| Java 5 | `java.util.concurrent` — `ExecutorService`, `Future`, `Lock`, `BlockingQueue` | Estável |
| Java 8 | `CompletableFuture`, parallel streams | Estável |
| Java 21 | **Virtual Threads** (JEP 444), Pattern Matching for switch (JEP 441), Record Patterns (JEP 440) | **Final** |
| Java 22 | Foreign Function & Memory API (JEP 454), Unnamed Variables (JEP 456) | **Final** |
| Java 24 | Stream Gatherers (JEP 485) | **Final** |
| Java 25 | Structured Concurrency (JEP 505), Scoped Values (JEP 502), Flexible Constructors (JEP 492), Module Imports (JEP 511) | **LTS** |

### Modelo Mental — Thread por Requisição Reimaginado

```
┌────────────────────────────────────────────────────────┐
│                    JVM (Java 25)                       │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Carrier      │  │ Carrier      │  │ Carrier      │  │
│  │ Thread (OS)  │  │ Thread (OS)  │  │ Thread (OS)  │  │
│  │              │  │              │  │              │  │
│  │  vt1  vt2    │  │  vt3  vt4    │  │  vt5  vt6    │  │
│  │  vt7  vt8    │  │  vt9  vt10   │  │  vt11 vt12   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                        │
│  Carrier threads ≈ núcleos de CPU                      │
│  Virtual threads ≈ milhões (uma por tarefa/requisição) │
└────────────────────────────────────────────────────────┘
```

---

## 2. Virtual Threads

### 2.1 Conceito e Motivação

Threads tradicionais do SO (platform threads) são caras: cada uma consome ~1 MB de stack e o escalonamento é feito pelo kernel. Com milhares de requisições simultâneas, o modelo thread-per-request rapidamente atinge o limite do SO.

**Virtual Threads** são threads gerenciadas pela JVM, extremamente leves (~poucos KB), que são multiplexadas sobre um pool pequeno de threads do SO (carrier threads). Quando uma virtual thread faz I/O bloqueante, a JVM a **desmonta** da carrier thread, liberando-a para outra virtual thread — sem callbacks, sem programação reativa.

```
Platform Thread:     1 thread OS = 1 thread Java (~1 MB stack)
Virtual Thread:      1 thread Java = poucos KB, montada/desmontada da carrier conforme necessário
```

### 2.2 Criando Virtual Threads

```java
// Forma mais simples — Thread.startVirtualThread
Thread vt = Thread.startVirtualThread(() -> {
    System.out.println("Executando na virtual thread: " + Thread.currentThread());
});
vt.join();

// Com Thread.ofVirtual() — builder pattern
Thread vt2 = Thread.ofVirtual()
    .name("minha-vt")
    .start(() -> {
        System.out.println("Virtual thread nomeada: " + Thread.currentThread().getName());
    });

// Factory para criar múltiplas virtual threads
ThreadFactory factory = Thread.ofVirtual()
    .name("worker-", 0) // worker-0, worker-1, worker-2...
    .factory();

for (int i = 0; i < 100_000; i++) {
    factory.newThread(() -> {
        try { Thread.sleep(Duration.ofSeconds(1)); }
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }).start();
}
```

### 2.3 Executors com Virtual Threads

```java
// ExecutorService que cria uma nova virtual thread para cada tarefa
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<String>> futures = new ArrayList<>();

    for (int i = 0; i < 10_000; i++) {
        final int id = i;
        futures.add(executor.submit(() -> {
            // I/O bloqueante é OK — a virtual thread será desmontada da carrier
            return fetchFromApi("/users/" + id);
        }));
    }

    for (var future : futures) {
        System.out.println(future.get());
    }
} // AutoCloseable — aguarda todas as tarefas e fecha o executor
```

**Diferença do pool tradicional:**

```java
// ❌ Pool fixo — limita a concorrência artificialmente com virtual threads
var pool = Executors.newFixedThreadPool(200);

// ✅ Virtual thread per task — a JVM gerencia a multiplexação
var executor = Executors.newVirtualThreadPerTaskExecutor();
```

### 2.4 Virtual Threads em Servidores Web

Com virtual threads, servidores como Tomcat e Jetty podem usar o modelo thread-per-request sem preocupação com o limite de threads:

```java
// Servidor HTTP simples com virtual threads (sem framework)
var server = HttpServer.create(new InetSocketAddress(8080), 0);
server.setExecutor(Executors.newVirtualThreadPerTaskExecutor());

server.createContext("/api/users", exchange -> {
    // Cada requisição roda em sua própria virtual thread
    String json = queryDatabase("SELECT * FROM users"); // bloqueante, OK
    exchange.getResponseHeaders().set("Content-Type", "application/json");
    byte[] response = json.getBytes(StandardCharsets.UTF_8);
    exchange.sendResponseHeaders(200, response.length);
    exchange.getResponseBody().write(response);
    exchange.close();
});

server.start();
```

### 2.5 Pinning — Quando a Virtual Thread Bloqueia a Carrier

Pinning ocorre quando a virtual thread não consegue ser desmontada da carrier thread. Isso acontece em dois cenários:

```java
// ❌ Cenário 1: bloco synchronized com I/O dentro
synchronized (lock) {
    // A virtual thread fica "pinned" à carrier thread durante todo o bloco
    var result = httpClient.send(request, BodyHandlers.ofString()); // I/O bloqueante
}

// ✅ Solução: usar ReentrantLock
private final ReentrantLock lock = new ReentrantLock();

lock.lock();
try {
    var result = httpClient.send(request, BodyHandlers.ofString());
} finally {
    lock.unlock();
}

// ❌ Cenário 2: native methods/JNI com I/O
// Solução: aguardar atualização da biblioteca ou isolar em platform thread
```

**Detectando pinning em desenvolvimento:**

```bash
# Flag da JVM para alertar sobre pinning
java -Djdk.tracePinnedThreads=short MinhaApp

# Ou mais detalhado (com stack trace completo)
java -Djdk.tracePinnedThreads=full MinhaApp
```

### 2.6 Boas Práticas com Virtual Threads

| Prática | Motivo |
|---------|--------|
| **Não use pool fixo** com virtual threads | Pools fixos limitam artificialmente a concorrência; virtual threads são baratas |
| **Substitua `synchronized` por `ReentrantLock`** quando houver I/O dentro | Evita pinning |
| **Não reutilize virtual threads** | Crie uma nova para cada tarefa — são descartáveis |
| **Use `Semaphore`** para limitar acesso a recursos | Em vez de limitar threads, limite o recurso (ex: conexões de banco) |
| **Não use `ThreadLocal`** como cache per-thread | Com milhões de virtual threads, `ThreadLocal` desperdiça memória — use `ScopedValue` |
| **Remova `-XX:ActiveProcessorCount` hacks** | A JVM gerencia carriers automaticamente com base nos cores disponíveis |

---

## 3. Structured Concurrency

### 3.1 O Problema da Concorrência Não-Estruturada

```java
// ❌ Concorrência não-estruturada — problemas sutis
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

Future<User> userFuture = executor.submit(() -> fetchUser(userId));
Future<List<Order>> ordersFuture = executor.submit(() -> fetchOrders(userId));

try {
    User user = userFuture.get();           // Se fetchOrders() falha enquanto esperamos aqui,
    List<Order> orders = ordersFuture.get(); // a exceção é perdida temporariamente
    return new UserProfile(user, orders);
} catch (ExecutionException e) {
    // Se fetchUser() falha, fetchOrders() continua executando desnecessariamente!
    // Precisamos cancelar manualmente — fácil de esquecer
    userFuture.cancel(true);
    ordersFuture.cancel(true);
    throw e;
}
```

Problemas da abordagem acima:
- Se uma subtarefa falha, as outras continuam executando (desperdício de recursos)
- O cancelamento precisa ser manual e é fácil de esquecer
- O stack trace não conecta a tarefa pai com as subtarefas
- O tempo de vida das subtarefas não está vinculado ao escopo do bloco

### 3.2 StructuredTaskScope — Conceito

`StructuredTaskScope` (JEP 505, final no Java 25) vincula o ciclo de vida das subtarefas ao bloco que as criou — semelhante ao escopo de um bloco `try-with-resources`:

```java
// ✅ Concorrência estruturada — subtarefas vinculadas ao escopo
UserProfile fetchUserProfile(long userId) throws Exception {
    try (var scope = StructuredTaskScope.open()) {
        // fork() cria uma virtual thread filha dentro do escopo
        Subtask<User> userTask = scope.fork(() -> fetchUser(userId));
        Subtask<List<Order>> ordersTask = scope.fork(() -> fetchOrders(userId));

        // join() aguarda TODAS as subtarefas completarem
        scope.join();

        // Após join(), os resultados estão disponíveis
        return new UserProfile(userTask.get(), ordersTask.get());
    }
    // Se o escopo sair (por exceção ou close), todas as subtarefas são canceladas automaticamente
}
```

**Garantias do `StructuredTaskScope`:**

1. **Todas as subtarefas terminam antes do escopo fechar** — sem tarefas órfãs
2. **Cancelamento automático** — se o escopo é cancelado, todas as subtarefas são interrompidas
3. **Observabilidade** — thread dumps mostram a hierarquia pai-filho
4. **Propagação de exceções** — exceções das subtarefas são agrupadas

### 3.3 ShutdownOnFailure — Falha Rápida

Quando qualquer subtarefa falha, todas as outras são canceladas imediatamente:

```java
record UserProfile(User user, List<Order> orders, double creditScore) {}

UserProfile fetchProfile(long userId) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {

        Subtask<User> userTask = scope.fork(() -> fetchUser(userId));
        Subtask<List<Order>> ordersTask = scope.fork(() -> fetchOrders(userId));
        Subtask<Double> creditTask = scope.fork(() -> fetchCreditScore(userId));

        scope.join(); // Lança se qualquer subtarefa falhou

        return new UserProfile(
            userTask.get(),
            ordersTask.get(),
            creditTask.get()
        );
    }
    // Se fetchUser() lança exceção → fetchOrders() e fetchCreditScore() são canceladas imediatamente
    // A exceção original é propagada; exceções das outras subtarefas são suppressed
}
```

### 3.4 ShutdownOnSuccess — Primeira Resposta Vence

Quando a primeira subtarefa retorna com sucesso, as demais são canceladas:

```java
// Busca em múltiplas fontes — usa o primeiro resultado disponível
String fetchPrice(String product) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.anySuccessfulResultOrThrow())) {

        scope.fork(() -> fetchPriceFromSupplierA(product));
        scope.fork(() -> fetchPriceFromSupplierB(product));
        scope.fork(() -> fetchPriceFromCache(product));

        // Retorna o resultado da primeira subtarefa que completou com sucesso
        return scope.join();
    }
    // As outras subtarefas são canceladas automaticamente
}
```

### 3.5 StructuredTaskScope Customizado

Para lógicas de decisão mais complexas, use `Joiner` customizado:

```java
// Joiner customizado: coleta todos os resultados, ignora falhas
<T> List<T> collectAllSuccessful(List<Callable<T>> tasks) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.allSuccessfulOrThrow())) {

        List<Subtask<T>> subtasks = tasks.stream()
            .map(scope::fork)
            .toList();

        scope.join();

        return subtasks.stream()
            .map(Subtask::get)
            .toList();
    }
}
```

**Exemplo prático — agregação de preços:**

```java
record PriceQuote(String supplier, BigDecimal price, Duration latency) {}

PriceQuote fetchBestPrice(String product) throws Exception {
    try (var scope = StructuredTaskScope.open()) {
        var quotes = List.of(
            scope.fork(() -> querySupplier("A", product)),
            scope.fork(() -> querySupplier("B", product)),
            scope.fork(() -> querySupplier("C", product))
        );

        scope.join();

        return quotes.stream()
            .filter(st -> st.state() == Subtask.State.SUCCESS)
            .map(Subtask::get)
            .min(Comparator.comparing(PriceQuote::price))
            .orElseThrow(() -> new RuntimeException("Nenhum fornecedor respondeu"));
    }
}
```

### 3.6 Composição e Aninhamento de Scopes

Scopes podem ser aninhados para criar hierarquias de tarefas:

```java
// Processamento de pedidos com subtarefas aninhadas
OrderResult processOrder(Order order) throws Exception {
    try (var scope = StructuredTaskScope.open()) {
        // Nível 1: validação e preparação em paralelo
        var validation = scope.fork(() -> validateOrder(order));
        var inventory = scope.fork(() -> {
            // Nível 2: checagem de estoque em múltiplos depósitos
            try (var innerScope = StructuredTaskScope.open(
                    StructuredTaskScope.Joiner.anySuccessfulResultOrThrow())) {
                innerScope.fork(() -> checkWarehouse("SP", order));
                innerScope.fork(() -> checkWarehouse("RJ", order));
                innerScope.fork(() -> checkWarehouse("MG", order));
                return innerScope.join(); // Primeiro depósito com estoque
            }
        });

        scope.join();
        return new OrderResult(validation.get(), inventory.get());
    }
}
```

---

## 4. Scoped Values — Compartilhamento de Dados entre Threads

### 4.1 O Problema do ThreadLocal

```java
// ❌ ThreadLocal — problemas com virtual threads
private static final ThreadLocal<UserContext> CONTEXT = new ThreadLocal<>();

void handleRequest(HttpRequest request) {
    CONTEXT.set(extractUserContext(request));
    try {
        processRequest(); // Acessa CONTEXT.get() em qualquer ponto da call stack
    } finally {
        CONTEXT.remove(); // Fácil de esquecer → memory leak
    }
}
```

Problemas do `ThreadLocal` com Virtual Threads:
- **Memória**: com milhões de virtual threads, cada `ThreadLocal` consome memória proporcional
- **Mutabilidade**: qualquer código pode chamar `set()` e alterar o valor, dificultando o rastreamento
- **Herança**: `InheritableThreadLocal` copia o valor para threads filhas, mas o custo cresce com o número de virtual threads
- **Lifecycle**: o `remove()` é fácil de esquecer, causando vazamento de memória

### 4.2 ScopedValue — Conceito

`ScopedValue` (JEP 502, final no Java 25) é a alternativa imutável e eficiente ao `ThreadLocal`:

```java
// Declaração: constante estática, como um "slot" de contexto
private static final ScopedValue<UserContext> CURRENT_USER = ScopedValue.newInstance();

void handleRequest(HttpRequest request) {
    UserContext user = extractUserContext(request);

    // where() define o valor; run() ou call() executa dentro do escopo
    ScopedValue.where(CURRENT_USER, user)
        .run(() -> {
            processRequest(); // Dentro deste bloco, CURRENT_USER.get() retorna 'user'
        });
    // Fora do bloco, CURRENT_USER.get() lança NoSuchElementException
}

void processRequest() {
    // Qualquer ponto da call stack dentro do escopo pode ler o valor
    UserContext user = CURRENT_USER.get();
    auditLog("Processando para: " + user.name());
}
```

**Com retorno de valor:**

```java
String result = ScopedValue.where(CURRENT_USER, user)
    .call(() -> {
        return computeResult(); // Pode retornar valor
    });
```

### 4.3 Herança Automática em StructuredTaskScope

O maior benefício do `ScopedValue` é a integração com `StructuredTaskScope` — o valor é automaticamente visível nas subtarefas sem cópia explícita:

```java
private static final ScopedValue<RequestContext> REQUEST_CTX = ScopedValue.newInstance();

void handleRequest(HttpRequest request) throws Exception {
    RequestContext ctx = new RequestContext(
        request.correlationId(),
        request.userId(),
        Instant.now()
    );

    ScopedValue.where(REQUEST_CTX, ctx).run(() -> {
        try (var scope = StructuredTaskScope.open(
                StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {

            // Ambas as subtarefas enxergam REQUEST_CTX automaticamente!
            scope.fork(() -> {
                log(REQUEST_CTX.get().correlationId(), "Buscando usuário");
                return fetchUser(REQUEST_CTX.get().userId());
            });

            scope.fork(() -> {
                log(REQUEST_CTX.get().correlationId(), "Buscando pedidos");
                return fetchOrders(REQUEST_CTX.get().userId());
            });

            scope.join();
        }
    });
}
```

### 4.4 Rebinding — Sobrescrever Valor em Escopo Filho

```java
private static final ScopedValue<String> TENANT = ScopedValue.newInstance();

void processMultiTenant() {
    ScopedValue.where(TENANT, "tenant-A").run(() -> {
        System.out.println(TENANT.get()); // "tenant-A"

        // Rebinding: cria um novo escopo com valor diferente
        ScopedValue.where(TENANT, "tenant-B").run(() -> {
            System.out.println(TENANT.get()); // "tenant-B"
        });

        System.out.println(TENANT.get()); // "tenant-A" — restaurado automaticamente
    });
}
```

### 4.5 ScopedValue vs ThreadLocal — Comparação

| Aspecto | `ThreadLocal` | `ScopedValue` |
|---------|---------------|---------------|
| **Mutabilidade** | Mutável (`set()` a qualquer momento) | Imutável dentro do escopo |
| **Escopo** | Vinculado à thread, sem escopo definido | Vinculado a um bloco de código (`run`/`call`) |
| **Herança** | `InheritableThreadLocal` (cópia) | Automática com `StructuredTaskScope` (sem cópia) |
| **Custo com Virtual Threads** | Alto (1 slot por thread × milhões) | Baixo (compartilhado eficientemente) |
| **Cleanup** | Manual (`remove()`) | Automático ao sair do escopo |
| **Concorrência** | Pode causar bugs se threads compartilham | Thread-safe por design |

---

## 5. Channels — Comunicação entre Threads

### 5.1 Conceito de Channel

Em linguagens como Go, **channels** são a primitiva principal de comunicação entre goroutines. Java não possui um tipo `Channel` nativo na linguagem, mas com Virtual Threads o padrão pode ser implementado de forma idiomática usando `BlockingQueue` — com a vantagem de que o bloqueio não desperdiça threads do SO.

```
Conceito:
┌──────────┐     Channel      ┌──────────┐
│ Producer │ ──── put ────▶   │          │
│ (VT)     │                  │  Queue   │ ──── take ────▶ Consumer (VT)
│          │ ◀── bloqueado ── │          │ ◀── bloqueado se vazio
└──────────┘   se cheio       └──────────┘
```

### 5.2 BlockingQueue como Channel

```java
// Channel genérico usando BlockingQueue
// Com virtual threads, put/take bloqueantes não desperdiçam threads do SO
public final class Channel<T> {
    private final BlockingQueue<T> queue;

    // Channel buffered (com capacidade)
    public Channel(int capacity) {
        this.queue = new ArrayBlockingQueue<>(capacity);
    }

    // Channel unbuffered (rendez-vous)
    public Channel() {
        this.queue = new SynchronousQueue<>();
    }

    public void send(T value) throws InterruptedException {
        queue.put(value); // Bloqueia se cheio
    }

    public T receive() throws InterruptedException {
        return queue.take(); // Bloqueia se vazio
    }

    public boolean trySend(T value, Duration timeout) throws InterruptedException {
        return queue.offer(value, timeout.toMillis(), TimeUnit.MILLISECONDS);
    }

    public T tryReceive(Duration timeout) throws InterruptedException {
        return queue.poll(timeout.toMillis(), TimeUnit.MILLISECONDS);
    }
}
```

### 5.3 Channel Unbuffered (Rendez-vous)

O channel sem buffer (`SynchronousQueue`) força produtor e consumidor a se sincronizar — o `send()` bloqueia até que alguém chame `receive()`, e vice-versa:

```java
void exemploRendezvous() throws Exception {
    var channel = new Channel<String>(); // SynchronousQueue internamente

    try (var scope = StructuredTaskScope.open()) {
        // Produtor
        scope.fork(() -> {
            channel.send("Olá"); // Bloqueia até o consumidor estar pronto
            System.out.println("Mensagem enviada");
            return null;
        });

        // Consumidor
        scope.fork(() -> {
            Thread.sleep(Duration.ofSeconds(1)); // Simula trabalho
            String msg = channel.receive(); // Desbloqueia o produtor
            System.out.println("Recebido: " + msg);
            return null;
        });

        scope.join();
    }
}
```

### 5.4 Channel com Capacidade (Buffered)

```java
void exemploBuffered() throws Exception {
    var channel = new Channel<Integer>(10); // Buffer de 10 elementos

    try (var scope = StructuredTaskScope.open()) {
        // Produtor — pode enviar até 10 itens sem bloquear
        scope.fork(() -> {
            for (int i = 1; i <= 100; i++) {
                channel.send(i);
                System.out.println("Enviado: " + i);
            }
            channel.send(-1); // Sentinel para sinalizar fim
            return null;
        });

        // Consumidor — processa no seu próprio ritmo
        scope.fork(() -> {
            while (true) {
                int value = channel.receive();
                if (value == -1) break; // Sentinel recebido
                System.out.println("Processado: " + value);
                Thread.sleep(Duration.ofMillis(50)); // Simula trabalho
            }
            return null;
        });

        scope.join();
    }
}
```

### 5.5 Fan-Out — Múltiplos Consumidores

Distribui trabalho entre múltiplos consumidores para processamento paralelo:

```java
void fanOut() throws Exception {
    var tasks = new Channel<String>(100);
    int numWorkers = 4;
    var poison = "DONE";

    try (var scope = StructuredTaskScope.open()) {
        // Produtor: envia tarefas para o channel
        scope.fork(() -> {
            for (String task : loadTasks()) {
                tasks.send(task);
            }
            // Envia um sentinel para cada worker
            for (int i = 0; i < numWorkers; i++) {
                tasks.send(poison);
            }
            return null;
        });

        // Workers: cada um consome do mesmo channel
        for (int w = 0; w < numWorkers; w++) {
            final int workerId = w;
            scope.fork(() -> {
                while (true) {
                    String task = tasks.receive();
                    if (task.equals(poison)) break;
                    System.out.printf("Worker-%d processando: %s%n", workerId, task);
                    processTask(task);
                }
                return null;
            });
        }

        scope.join();
    }
}
```

### 5.6 Fan-In — Múltiplos Produtores

Múltiplos produtores alimentam um único channel para consolidação:

```java
record LogEvent(String source, String message, Instant timestamp) {}

void fanIn() throws Exception {
    var logs = new Channel<LogEvent>(50);
    var sources = List.of("web-server", "api-gateway", "database", "cache");

    try (var scope = StructuredTaskScope.open()) {
        // Múltiplos produtores — cada fonte de log envia para o mesmo channel
        for (String source : sources) {
            scope.fork(() -> {
                for (int i = 0; i < 100; i++) {
                    logs.send(new LogEvent(source, "Event " + i, Instant.now()));
                    Thread.sleep(Duration.ofMillis(ThreadLocalRandom.current().nextInt(10, 100)));
                }
                return null;
            });
        }

        // Consumidor único — agrega todos os logs
        scope.fork(() -> {
            int count = 0;
            int total = sources.size() * 100;
            while (count < total) {
                LogEvent event = logs.receive();
                System.out.printf("[%s] %s: %s%n",
                    event.timestamp(), event.source(), event.message());
                count++;
            }
            return null;
        });

        scope.join();
    }
}
```

### 5.7 Pipeline de Channels

Encadeia estágios de processamento, onde a saída de um estágio é a entrada do próximo:

```java
// Pipeline: gerar números → filtrar pares → calcular quadrado → imprimir
void pipeline() throws Exception {
    var numbers = new Channel<Integer>(10);
    var evens = new Channel<Integer>(10);
    var squares = new Channel<Integer>(10);

    try (var scope = StructuredTaskScope.open()) {
        // Estágio 1: gerar números
        scope.fork(() -> {
            for (int i = 1; i <= 50; i++) {
                numbers.send(i);
            }
            numbers.send(-1); // Sentinel
            return null;
        });

        // Estágio 2: filtrar pares
        scope.fork(() -> {
            while (true) {
                int n = numbers.receive();
                if (n == -1) { evens.send(-1); break; }
                if (n % 2 == 0) evens.send(n);
            }
            return null;
        });

        // Estágio 3: calcular quadrado
        scope.fork(() -> {
            while (true) {
                int n = evens.receive();
                if (n == -1) { squares.send(-1); break; }
                squares.send(n * n);
            }
            return null;
        });

        // Estágio 4: consumir resultados
        scope.fork(() -> {
            while (true) {
                int result = squares.receive();
                if (result == -1) break;
                System.out.println("Resultado: " + result);
            }
            return null;
        });

        scope.join();
    }
}
```

### 5.8 Select — Multiplexação de Channels

Em Go, `select` permite aguardar múltiplos channels simultaneamente. Em Java, podemos emular esse comportamento:

```java
// Implementação de select para múltiplos channels
@SafeVarargs
static <T> ChannelResult<T> select(Channel<T>... channels) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.anySuccessfulResultOrThrow())) {
        for (int i = 0; i < channels.length; i++) {
            final int index = i;
            final Channel<T> ch = channels[i];
            scope.fork(() -> new ChannelResult<>(index, ch.receive()));
        }
        return scope.join();
    }
}

record ChannelResult<T>(int channelIndex, T value) {}

// Uso:
void exemploSelect() throws Exception {
    var ch1 = new Channel<String>(1);
    var ch2 = new Channel<String>(1);

    try (var scope = StructuredTaskScope.open()) {
        scope.fork(() -> {
            Thread.sleep(Duration.ofMillis(200));
            ch1.send("resposta do serviço A");
            return null;
        });
        scope.fork(() -> {
            Thread.sleep(Duration.ofMillis(100));
            ch2.send("resposta do serviço B");
            return null;
        });

        // Pega o primeiro resultado disponível
        scope.fork(() -> {
            var result = select(ch1, ch2);
            System.out.printf("Channel %d respondeu primeiro: %s%n",
                result.channelIndex(), result.value());
            return null;
        });

        scope.join();
    }
}
```

---

## 6. Padrões Avançados de Concorrência

### 6.1 Rate Limiting com Semaphore

Com virtual threads, o gargalo não é mais o número de threads, mas os recursos externos (banco, APIs). Use `Semaphore` para controlar o acesso:

```java
// Limitar a 50 chamadas simultâneas à API externa
private static final Semaphore API_LIMITER = new Semaphore(50);

String callExternalApi(String endpoint) throws Exception {
    API_LIMITER.acquire(); // Bloqueia se já houver 50 chamadas em andamento
    try {
        return httpClient.send(
            HttpRequest.newBuilder(URI.create(endpoint)).build(),
            BodyHandlers.ofString()
        ).body();
    } finally {
        API_LIMITER.release();
    }
}

// Uso: 10.000 virtual threads, mas no máximo 50 chamadas simultâneas à API
void processAll(List<String> endpoints) throws Exception {
    try (var scope = StructuredTaskScope.open()) {
        var results = endpoints.stream()
            .map(ep -> scope.fork(() -> callExternalApi(ep)))
            .toList();
        scope.join();
        results.forEach(r -> System.out.println(r.get()));
    }
}
```

### 6.2 Timeout e Cancelamento

```java
// Timeout no escopo inteiro
UserProfile fetchWithTimeout(long userId) throws Exception {
    try (var scope = StructuredTaskScope.open()) {
        var user = scope.fork(() -> fetchUser(userId));
        var orders = scope.fork(() -> fetchOrders(userId));

        // joinUntil: se não completar até o deadline, todas as subtarefas são canceladas
        scope.joinUntil(Instant.now().plusSeconds(5));

        return new UserProfile(user.get(), orders.get());
    }
}

// Timeout individual por subtarefa usando Channel
<T> T withTimeout(Callable<T> task, Duration timeout) throws Exception {
    var result = new Channel<T>(1);
    var error = new Channel<Exception>(1);

    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.anySuccessfulResultOrThrow())) {
        // Tarefa principal
        scope.fork(() -> {
            try {
                return task.call();
            } catch (Exception e) {
                throw e;
            }
        });
        // Timer
        scope.fork(() -> {
            Thread.sleep(timeout);
            throw new TimeoutException("Timeout após " + timeout);
        });

        return scope.join();
    }
}
```

### 6.3 Producer-Consumer com Backpressure

```java
// Channel com backpressure natural — quando o buffer enche, o produtor bloqueia
void processWithBackpressure() throws Exception {
    // Buffer pequeno = backpressure forte (produtor desacelera rápido)
    var channel = new Channel<Record>(5);
    var results = new Channel<ProcessedRecord>(5);

    try (var scope = StructuredTaskScope.open()) {
        // Produtor: lê do banco e envia para o channel
        scope.fork(() -> {
            try (var stream = database.streamAllRecords()) {
                stream.forEach(record -> {
                    try {
                        channel.send(record); // Bloqueia quando buffer cheio (backpressure)
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
            channel.send(null); // Sinaliza fim
            return null;
        });

        // Worker pool: processa registros
        int workers = Runtime.getRuntime().availableProcessors();
        var latch = new CountDownLatch(workers);

        for (int i = 0; i < workers; i++) {
            scope.fork(() -> {
                while (true) {
                    Record record = channel.receive();
                    if (record == null) {
                        channel.send(null); // Propaga sentinel para próximo worker
                        break;
                    }
                    results.send(process(record));
                }
                latch.countDown();
                return null;
            });
        }

        // Escritor: consome resultados e persiste
        scope.fork(() -> {
            while (true) {
                ProcessedRecord pr = results.tryReceive(Duration.ofSeconds(1));
                if (pr == null && latch.getCount() == 0) break;
                if (pr != null) database.save(pr);
            }
            return null;
        });

        scope.join();
    }
}
```

### 6.4 Fork-Join com Structured Concurrency

```java
// Merge sort paralelo usando structured concurrency
List<Integer> parallelMergeSort(List<Integer> list) throws Exception {
    if (list.size() <= 1000) {
        // Base case: ordenação sequencial para listas pequenas
        var result = new ArrayList<>(list);
        Collections.sort(result);
        return result;
    }

    int mid = list.size() / 2;

    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {
        var left = scope.fork(() -> parallelMergeSort(list.subList(0, mid)));
        var right = scope.fork(() -> parallelMergeSort(list.subList(mid, list.size())));
        scope.join();
        return merge(left.get(), right.get());
    }
}

private List<Integer> merge(List<Integer> left, List<Integer> right) {
    var result = new ArrayList<Integer>(left.size() + right.size());
    int i = 0, j = 0;
    while (i < left.size() && j < right.size()) {
        if (left.get(i) <= right.get(j)) {
            result.add(left.get(i++));
        } else {
            result.add(right.get(j++));
        }
    }
    while (i < left.size()) result.add(left.get(i++));
    while (j < right.size()) result.add(right.get(j++));
    return result;
}
```

---

## 7. Locks Avançados — ReentrantLock, ReadWriteLock e StampedLock

### 7.1 Evolução dos Mecanismos de Locking

| Mecanismo | Desde | Características |
|-----------|-------|-----------------|
| `synchronized` | Java 1.0 | Simples, mas causa **pinning** em virtual threads com I/O |
| `ReentrantLock` | Java 5 | Reentrante, interruptível, com `tryLock` e `Condition` — **compatível com virtual threads** |
| `ReadWriteLock` | Java 5 | Separa leitura (compartilhada) de escrita (exclusiva) |
| `StampedLock` | Java 8 | Locking otimista para leituras frequentes, melhor throughput |

### 7.2 ReentrantLock — Substituto do synchronized

```java
// ❌ synchronized — causa pinning com virtual threads
public class ContadorSynchronized {
    private int valor = 0;

    public synchronized void incrementar() {
        valor++; // OK para CPU-bound, mas se houvesse I/O aqui, causaria pinning
    }

    public synchronized int getValor() {
        return valor;
    }
}

// ✅ ReentrantLock — compatível com virtual threads
public class ContadorReentrant {
    private final ReentrantLock lock = new ReentrantLock();
    private int valor = 0;

    public void incrementar() {
        lock.lock();
        try {
            valor++;
        } finally {
            lock.unlock(); // SEMPRE no finally para garantir liberação
        }
    }

    public int getValor() {
        lock.lock();
        try {
            return valor;
        } finally {
            lock.unlock();
        }
    }
}
```

**Funcionalidades extras do `ReentrantLock`:**

```java
private final ReentrantLock lock = new ReentrantLock(true); // fair = true → FIFO

// tryLock — tentativa não-bloqueante
void atualizarSeDisponivel(String novoValor) {
    if (lock.tryLock()) {
        try {
            this.valor = novoValor;
        } finally {
            lock.unlock();
        }
    } else {
        System.out.println("Recurso ocupado, tentando depois...");
    }
}

// tryLock com timeout
void atualizarComTimeout(String novoValor) throws InterruptedException {
    if (lock.tryLock(500, TimeUnit.MILLISECONDS)) {
        try {
            this.valor = novoValor;
        } finally {
            lock.unlock();
        }
    } else {
        throw new TimeoutException("Não conseguiu obter o lock em 500ms");
    }
}

// lockInterruptibly — permite cancelamento via interrupt
void atualizarInterruptivel(String novoValor) throws InterruptedException {
    lock.lockInterruptibly(); // Lança InterruptedException se a thread for interrompida
    try {
        this.valor = novoValor;
    } finally {
        lock.unlock();
    }
}
```

**Condition — substituto do wait/notify:**

```java
public class FilaLimitada<T> {
    private final Queue<T> fila = new LinkedList<>();
    private final int capacidade;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition naoCheia = lock.newCondition();
    private final Condition naoVazia = lock.newCondition();

    public FilaLimitada(int capacidade) {
        this.capacidade = capacidade;
    }

    public void adicionar(T item) throws InterruptedException {
        lock.lock();
        try {
            while (fila.size() == capacidade) {
                naoCheia.await(); // Aguarda sinal de que há espaço
            }
            fila.add(item);
            naoVazia.signal(); // Avisa consumidores que há itens
        } finally {
            lock.unlock();
        }
    }

    public T remover() throws InterruptedException {
        lock.lock();
        try {
            while (fila.isEmpty()) {
                naoVazia.await(); // Aguarda sinal de que há itens
            }
            T item = fila.poll();
            naoCheia.signal(); // Avisa produtores que há espaço
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### 7.3 ReadWriteLock — Leituras Concorrentes

Permite **múltiplas threads lendo simultaneamente**, mas **apenas uma escrevendo** (exclusivo):

```java
public class CacheComReadWriteLock<K, V> {
    private final Map<K, V> cache = new HashMap<>();
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();

    // Múltiplas threads podem ler ao mesmo tempo
    public V get(K key) {
        readLock.lock();
        try {
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }

    // Apenas uma thread pode escrever por vez (bloqueia leituras também)
    public void put(K key, V value) {
        writeLock.lock();
        try {
            cache.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }

    public int size() {
        readLock.lock();
        try {
            return cache.size();
        } finally {
            readLock.unlock();
        }
    }
}
```

### 7.4 StampedLock — Locking Otimista

`StampedLock` introduz o conceito de **leitura otimista**: a thread lê os dados sem adquirir um lock real, e depois valida se houve escrita concorrente. Se não houve, a leitura é extremamente rápida (sem contenção).

```java
public class PontoGeografico {
    private double latitude;
    private double longitude;
    private final StampedLock lock = new StampedLock();

    public void mover(double novaLat, double novaLon) {
        long stamp = lock.writeLock();
        try {
            this.latitude = novaLat;
            this.longitude = novaLon;
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    // Leitura otimista — sem bloqueio, melhor performance
    public double calcularDistanciaOrigem() {
        // 1) Tenta leitura otimista (sem lock real)
        long stamp = lock.tryOptimisticRead();
        double lat = this.latitude;
        double lon = this.longitude;

        // 2) Valida se houve escrita enquanto lia
        if (!lock.validate(stamp)) {
            // 3) Se houve escrita, faz fallback para leitura pessimista
            stamp = lock.readLock();
            try {
                lat = this.latitude;
                lon = this.longitude;
            } finally {
                lock.unlockRead(stamp);
            }
        }

        return Math.sqrt(lat * lat + lon * lon);
    }

    // Upgrade: leitura → escrita sem soltar o lock
    public void moverSeMaisProximo(double novaLat, double novaLon) {
        long stamp = lock.readLock();
        try {
            double distanciaAtual = Math.sqrt(latitude * latitude + longitude * longitude);
            double distanciaNova = Math.sqrt(novaLat * novaLat + novaLon * novaLon);

            if (distanciaNova < distanciaAtual) {
                // Tenta upgrade para write lock
                long writeStamp = lock.tryConvertToWriteLock(stamp);
                if (writeStamp != 0L) {
                    stamp = writeStamp;
                    this.latitude = novaLat;
                    this.longitude = novaLon;
                } else {
                    // Upgrade falhou — solta read, adquire write
                    lock.unlockRead(stamp);
                    stamp = lock.writeLock();
                    this.latitude = novaLat;
                    this.longitude = novaLon;
                }
            }
        } finally {
            lock.unlock(stamp);
        }
    }
}
```

### 7.5 Quando Usar Cada Lock

| Cenário | Recomendação |
|---------|-------------|
| Seções críticas simples, sem I/O | `synchronized` (mais simples, sem overhead de virtual thread) |
| Seções com I/O bloqueante (banco, HTTP) | `ReentrantLock` (evita pinning) |
| Cache com muitas leituras e poucas escritas | `ReadWriteLock` |
| Dados lidos com altíssima frequência, raramente alterados | `StampedLock` com leitura otimista |
| Coordenação com `await/signal` (producer-consumer) | `ReentrantLock` + `Condition` |
| Limitar concorrência sem exclusão mútua | `Semaphore` |

---

## 8. Atomic Variables e VarHandle

### 8.1 Classes Atomic — Operações Lock-Free

As classes `java.util.concurrent.atomic` usam instruções **CAS (Compare-And-Swap)** do processador para garantir atomicidade sem locks:

```java
// AtomicInteger — contador thread-safe sem lock
AtomicInteger contador = new AtomicInteger(0);

// Operações atômicas básicas
contador.incrementAndGet();     // ++x (retorna novo valor)
contador.getAndIncrement();     // x++ (retorna valor antigo)
contador.addAndGet(5);          // x += 5
contador.compareAndSet(5, 10);  // Se x == 5, então x = 10 (retorna true/false)

// Operação atômica com função customizada
contador.updateAndGet(x -> x * 2); // Dobra o valor atomicamente
contador.accumulateAndGet(3, Integer::max); // x = max(x, 3)
```

```java
// AtomicReference — referência atômica a objetos
record Config(String url, int timeout) {}

AtomicReference<Config> configAtual = new AtomicReference<>(
    new Config("http://api.example.com", 5000)
);

// Atualização atômica da configuração
configAtual.updateAndGet(old ->
    new Config(old.url(), 10000) // Altera timeout, mantém URL
);

// CompareAndSet para atualizações condicionais
Config expected = configAtual.get();
Config updated = new Config("http://api-v2.example.com", 3000);
boolean success = configAtual.compareAndSet(expected, updated);
```

### 8.2 LongAdder e LongAccumulator — Alta Contenção

Para contadores com alta contenção (muitas threads incrementando), `LongAdder` tem throughput **muito superior** ao `AtomicLong`:

```java
// ❌ AtomicLong — todas as threads competem pelo mesmo valor
AtomicLong contadorAtomic = new AtomicLong();
contadorAtomic.incrementAndGet(); // CAS retry loop sob alta contenção

// ✅ LongAdder — distribui o contador entre cells, reduz contenção
LongAdder contadorAdder = new LongAdder();
contadorAdder.increment();      // Quase sem contenção
contadorAdder.add(5);
long total = contadorAdder.sum(); // Soma todas as cells (eventual consistency)

// LongAccumulator — versão generalizada com operação customizada
LongAccumulator maxValue = new LongAccumulator(Long::max, Long.MIN_VALUE);
maxValue.accumulate(42);
maxValue.accumulate(99);
long max = maxValue.get(); // 99
```

**Exemplo prático — métricas de requisições:**

```java
public class RequestMetrics {
    private final LongAdder totalRequests = new LongAdder();
    private final LongAdder errorCount = new LongAdder();
    private final LongAccumulator maxLatency = new LongAccumulator(Long::max, 0);
    private final LongAdder totalLatency = new LongAdder();

    public void recordRequest(long latencyMs, boolean success) {
        totalRequests.increment();
        totalLatency.add(latencyMs);
        maxLatency.accumulate(latencyMs);
        if (!success) errorCount.increment();
    }

    public String getStats() {
        long total = totalRequests.sum();
        return "Requests: %d | Errors: %d | Avg Latency: %dms | Max Latency: %dms"
            .formatted(
                total,
                errorCount.sum(),
                total > 0 ? totalLatency.sum() / total : 0,
                maxLatency.get()
            );
    }
}
```

### 8.3 AtomicStampedReference — Resolvendo o Problema ABA

O problema ABA ocorre quando um valor muda de A → B → A, e um `compareAndSet` não detecta que houve alteração intermediária:

```java
// AtomicStampedReference resolve o ABA mantendo um "stamp" (versão)
AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

// Leitura: captura valor E stamp
int[] stampHolder = new int[1];
String valor = ref.get(stampHolder);
int stamp = stampHolder[0];

// Escrita: só funciona se valor E stamp não mudaram
boolean ok = ref.compareAndSet("A", "B", stamp, stamp + 1);
// Se outra thread mudou A→B→A, o stamp é diferente → CAS falha
```

### 8.4 VarHandle — Acesso de Baixo Nível a Variáveis

`VarHandle` (Java 9+) oferece controle granular sobre a **semântica de memória** (memory ordering) ao acessar campos:

```java
public class ContadorVarHandle {
    private volatile int valor;
    private static final VarHandle VALOR_HANDLE;

    static {
        try {
            VALOR_HANDLE = MethodHandles.lookup()
                .findVarHandle(ContadorVarHandle.class, "valor", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    // Diferentes modos de acesso com semânticas de memória distintas:

    // Opaque: garante atomicidade, mas sem ordering guarantees
    public int getOpaque() {
        return (int) VALOR_HANDLE.getOpaque(this);
    }

    // Acquire/Release: ordering para pairs (release store → acquire load)
    public int getAcquire() {
        return (int) VALOR_HANDLE.getAcquire(this);
    }

    public void setRelease(int newValue) {
        VALOR_HANDLE.setRelease(this, newValue);
    }

    // CAS com VarHandle
    public boolean incrementarCAS() {
        int expected = (int) VALOR_HANDLE.get(this);
        return VALOR_HANDLE.compareAndSet(this, expected, expected + 1);
    }

    // getAndAdd atômico
    public int getAndAdd(int delta) {
        return (int) VALOR_HANDLE.getAndAdd(this, delta);
    }
}
```

**VarHandle para arrays:**

```java
// Acesso atômico a elementos de array
int[] dados = new int[10];
VarHandle arrayHandle = MethodHandles.arrayElementVarHandle(int[].class);

arrayHandle.set(dados, 0, 42);                     // dados[0] = 42
arrayHandle.compareAndSet(dados, 0, 42, 100);      // CAS: dados[0]: 42 → 100
int old = (int) arrayHandle.getAndAdd(dados, 0, 5); // dados[0] += 5 atomicamente
```

---

## 9. CompletableFuture vs Structured Concurrency

### 9.1 Quando Ainda Usar CompletableFuture

| Cenário | Recomendação |
|---------|-------------|
| Paralelismo simples com dependência entre resultados | **StructuredTaskScope** |
| Composição de operações assíncronas (pipeline) | **CompletableFuture** |
| Integração com APIs que retornam `CompletableFuture` | **CompletableFuture** |
| Falha rápida / primeira resposta vence | **StructuredTaskScope** |
| Callback-driven (registro de handlers) | **CompletableFuture** |
| Timeout global com cancelamento automático | **StructuredTaskScope** |

### 9.2 Comparação Lado a Lado

**Cenário: buscar dados em paralelo e combinar**

```java
// CompletableFuture — composição funcional
CompletableFuture<UserProfile> fetchProfileCF(long userId) {
    var userFuture = CompletableFuture.supplyAsync(() -> fetchUser(userId));
    var ordersFuture = CompletableFuture.supplyAsync(() -> fetchOrders(userId));

    return userFuture.thenCombine(ordersFuture, UserProfile::new)
        .orTimeout(5, TimeUnit.SECONDS)
        .exceptionally(ex -> {
            log.error("Falha ao buscar perfil", ex);
            return UserProfile.EMPTY;
        });
}

// Structured Concurrency — imperativo, com cancelamento automático
UserProfile fetchProfileSC(long userId) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {
        var user = scope.fork(() -> fetchUser(userId));
        var orders = scope.fork(() -> fetchOrders(userId));
        scope.joinUntil(Instant.now().plusSeconds(5));
        return new UserProfile(user.get(), orders.get());
    }
}
```

**Cenário: pipeline de transformações**

```java
// CompletableFuture é superior para pipelines encadeados
CompletableFuture<Report> generateReport(long userId) {
    return CompletableFuture.supplyAsync(() -> fetchUser(userId))
        .thenApplyAsync(user -> fetchTransactions(user))
        .thenApplyAsync(transactions -> calculateMetrics(transactions))
        .thenApplyAsync(metrics -> buildReport(metrics))
        .thenApplyAsync(report -> enrichWithCharts(report))
        .orTimeout(30, TimeUnit.SECONDS);
}

// Com StructuredTaskScope, o mesmo pipeline seria sequencial
// (a menos que haja oportunidades de paralelismo entre etapas)
Report generateReportSC(long userId) throws Exception {
    User user = fetchUser(userId);
    List<Transaction> txns = fetchTransactions(user);
    Metrics metrics = calculateMetrics(txns);
    Report report = buildReport(metrics);
    return enrichWithCharts(report);
}
```

**Cenário: combinando ambos**

```java
// Uso híbrido: StructuredTaskScope para o nível externo, CompletableFuture para pipelines internos
DashboardData buildDashboard(long userId) throws Exception {
    try (var scope = StructuredTaskScope.open(
            StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {

        // Cada fork executa seu próprio pipeline
        var reportTask = scope.fork(() ->
            CompletableFuture.supplyAsync(() -> fetchTransactions(userId))
                .thenApply(this::calculateMetrics)
                .thenApply(this::buildReport)
                .join()
        );

        var notificationsTask = scope.fork(() -> fetchNotifications(userId));
        var recommendationsTask = scope.fork(() -> generateRecommendations(userId));

        scope.joinUntil(Instant.now().plusSeconds(10));

        return new DashboardData(
            reportTask.get(),
            notificationsTask.get(),
            recommendationsTask.get()
        );
    }
}
```

---

## 10. Integração com Spring Boot

### 10.1 Configuração de Virtual Threads no Spring Boot

```yaml
# application.yml — Spring Boot 3.2+
spring:
  threads:
    virtual:
      enabled: true  # Tomcat/Jetty usam virtual threads para cada requisição
```

Com essa única propriedade, o Spring Boot configura:
- **Tomcat/Jetty**: cada requisição HTTP roda em uma virtual thread
- **@Async**: usa virtual threads automaticamente
- **@Scheduled**: usa virtual threads automaticamente

```java
// Configuração programática (alternativa ao YAML)
@Configuration
public class VirtualThreadConfig {

    @Bean
    TomcatProtocolHandlerCustomizer<?> virtualThreadCustomizer() {
        return handler -> handler.setExecutor(
            Executors.newVirtualThreadPerTaskExecutor()
        );
    }

    @Bean
    AsyncTaskExecutor applicationTaskExecutor() {
        return new TaskExecutorAdapter(
            Executors.newVirtualThreadPerTaskExecutor()
        );
    }
}
```

### 10.2 Structured Concurrency em Services

```java
@Service
public class ProductService {

    private final ProductRepository productRepo;
    private final ReviewClient reviewClient;
    private final PricingClient pricingClient;
    private final InventoryClient inventoryClient;

    // Construtor com injeção...

    public ProductDetail getProductDetail(Long productId) {
        try (var scope = StructuredTaskScope.open(
                StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {

            var product = scope.fork(() -> productRepo.findById(productId)
                .orElseThrow(() -> new NotFoundException("Produto não encontrado")));
            var reviews = scope.fork(() -> reviewClient.getReviews(productId));
            var pricing = scope.fork(() -> pricingClient.getPrice(productId));
            var stock = scope.fork(() -> inventoryClient.checkStock(productId));

            scope.join();

            return new ProductDetail(
                product.get(),
                reviews.get(),
                pricing.get(),
                stock.get()
            );
        } catch (Exception e) {
            throw new ServiceException("Falha ao buscar detalhes do produto", e);
        }
    }
}
```

### 10.3 Propagação de Contexto com Scoped Values

```java
// Definição centralizada dos scoped values
public final class RequestScopedValues {
    public static final ScopedValue<String> CORRELATION_ID = ScopedValue.newInstance();
    public static final ScopedValue<String> TENANT_ID = ScopedValue.newInstance();
    public static final ScopedValue<UserPrincipal> CURRENT_USER = ScopedValue.newInstance();

    private RequestScopedValues() {}
}

// Filter que estabelece o contexto
@Component
public class ScopedContextFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {
        String correlationId = Optional.ofNullable(request.getHeader("X-Correlation-ID"))
            .orElse(UUID.randomUUID().toString());
        String tenantId = request.getHeader("X-Tenant-ID");

        ScopedValue.where(RequestScopedValues.CORRELATION_ID, correlationId)
            .where(RequestScopedValues.TENANT_ID, tenantId)
            .run(() -> {
                try {
                    chain.doFilter(request, response);
                } catch (IOException | ServletException e) {
                    throw new RuntimeException(e);
                }
            });
    }
}

// Service que acessa o contexto — funciona inclusive em subtarefas fork()
@Service
public class AuditService {

    public void logAction(String action) {
        String corrId = RequestScopedValues.CORRELATION_ID.get();
        String tenant = RequestScopedValues.TENANT_ID.get();
        log.info("[{}] [tenant={}] {}", corrId, tenant, action);
    }
}
```

---

## 11. Testes de Código Concorrente

```java
@Test
void channelDeveTransmitirMensagens() throws Exception {
    var channel = new Channel<String>(5);

    try (var scope = StructuredTaskScope.open()) {
        scope.fork(() -> {
            channel.send("msg1");
            channel.send("msg2");
            channel.send("msg3");
            return null;
        });

        scope.fork(() -> {
            assertEquals("msg1", channel.receive());
            assertEquals("msg2", channel.receive());
            assertEquals("msg3", channel.receive());
            return null;
        });

        scope.join();
    }
}

@Test
void structuredConcurrencyDeveCancelarSubtarefasEmFalha() {
    assertThrows(Exception.class, () -> {
        try (var scope = StructuredTaskScope.open(
                StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {
            scope.fork(() -> { throw new RuntimeException("Falha intencional"); });
            scope.fork(() -> {
                Thread.sleep(Duration.ofSeconds(10)); // Será cancelada
                return "Nunca retorna";
            });
            scope.join();
        }
    });
}

@Test
void scopedValueDeveSerHerdadoPorSubtarefas() throws Exception {
    var CONTEXT = ScopedValue.newInstance();

    ScopedValue.where(CONTEXT, "valor-pai").run(() -> {
        try (var scope = StructuredTaskScope.open()) {
            var result = scope.fork(() -> CONTEXT.get());
            scope.join();
            assertEquals("valor-pai", result.get());
        } catch (Exception e) {
            fail(e);
        }
    });
}

@Test
void virtualThreadsDevemSerEscalonadasEficientemente() throws Exception {
    int numThreads = 100_000;
    var latch = new CountDownLatch(numThreads);
    var counter = new AtomicInteger();

    long start = System.nanoTime();

    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        for (int i = 0; i < numThreads; i++) {
            executor.submit(() -> {
                counter.incrementAndGet();
                latch.countDown();
            });
        }
        latch.await(10, TimeUnit.SECONDS);
    }

    long elapsed = Duration.ofNanos(System.nanoTime() - start).toMillis();
    assertEquals(numThreads, counter.get());
    System.out.printf("%d virtual threads em %d ms%n", numThreads, elapsed);
}
```

---

## 12. Comparação com Go e Kotlin

| Aspecto | Java 25 | Go | Kotlin |
|---------|---------|-----|--------|
| **Threads leves** | Virtual Threads | Goroutines | Coroutines |
| **Comunicação** | `BlockingQueue` (channel pattern) | `chan` (nativo) | `Channel` (kotlinx.coroutines) |
| **Concorrência estruturada** | `StructuredTaskScope` | `errgroup` (biblioteca) | `coroutineScope` (nativo) |
| **Contexto compartilhado** | `ScopedValue` | `context.Context` | `CoroutineContext` |
| **Cancelamento** | Automático no scope | `context.WithCancel` | Automático no scope |
| **Select/multiplex** | Emulado com scope | `select` (nativo) | `select` (nativo) |
| **Overhead por thread** | ~poucos KB | ~poucos KB | ~poucos centenas de bytes |
| **Modelo de I/O** | Bloqueante (transparente) | Bloqueante (transparente) | Suspend functions (explícito) |
| **Integração com ecossistema** | Todas as bibliotecas Java existentes | Stdlib Go | Interop com Java |

### Equivalência de Padrões Comuns

**Go → Java 25:**

```go
// Go
ch := make(chan string, 10)
go func() { ch <- "hello" }()
msg := <-ch
```

```java
// Java 25
var ch = new Channel<String>(10);
Thread.startVirtualThread(() -> {
    try { ch.send("hello"); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
String msg = ch.receive();
```

**Kotlin → Java 25:**

```kotlin
// Kotlin
coroutineScope {
    val user = async { fetchUser(id) }
    val orders = async { fetchOrders(id) }
    UserProfile(user.await(), orders.await())
}
```

```java
// Java 25
try (var scope = StructuredTaskScope.open(
        StructuredTaskScope.Joiner.awaitAllSuccessfulOrThrow())) {
    var user = scope.fork(() -> fetchUser(id));
    var orders = scope.fork(() -> fetchOrders(id));
    scope.join();
    new UserProfile(user.get(), orders.get());
}
```

---

---

# Parte II — Features Avançadas de Linguagem (Java 21–25)

As seções a seguir cobrem funcionalidades de linguagem que não são específicas de concorrência, mas que foram finalizadas ou amadureceram entre Java 21 e Java 25.

---

## 13. Pattern Matching Avançado

### 13.1 Evolução do Pattern Matching

| Versão | Recurso | JEP |
|--------|---------|-----|
| Java 16 | `instanceof` com pattern | JEP 394 |
| Java 21 | Pattern matching no `switch` (final) | JEP 441 |
| Java 21 | Record patterns (final) | JEP 440 |
| Java 22 | Unnamed variables e patterns | JEP 456 |
| Java 23+ | Primitive types in patterns (preview) | JEP 455 |

### 13.2 instanceof com Pattern Variable

```java
// ❌ Antes do Java 16 — cast manual
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.toUpperCase());
}

// ✅ Java 16+ — variável de pattern
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}

// Composição com operadores lógicos
if (obj instanceof String s && s.length() > 5) {
    System.out.println("String longa: " + s);
}

// Negação — variável NÃO está disponível no bloco if, mas está disponível APÓS
if (!(obj instanceof String s)) {
    System.out.println("Não é String");
    return;
}
// Aqui 's' está disponível (porque se chegou aqui, obj É String)
System.out.println(s.toUpperCase());
```

### 13.3 Switch com Pattern Matching

```java
// Switch com patterns de tipo — substitui cadeias de if-instanceof
String descricao(Object obj) {
    return switch (obj) {
        case Integer i    -> "Inteiro: " + i;
        case Long l       -> "Long: " + l;
        case Double d     -> "Double: " + d;
        case String s     -> "String: " + s;
        case int[] arr    -> "Array de int com " + arr.length + " elementos";
        case null         -> "Valor nulo";
        default           -> "Tipo desconhecido: " + obj.getClass().getSimpleName();
    };
}
```

### 13.4 Guard Clauses (when)

```java
// when adiciona condições extras ao pattern
String classificar(Object obj) {
    return switch (obj) {
        case Integer i when i < 0    -> "Negativo: " + i;
        case Integer i when i == 0   -> "Zero";
        case Integer i when i <= 100 -> "Pequeno: " + i;
        case Integer i               -> "Grande: " + i;
        case String s when s.isBlank() -> "String vazia";
        case String s                -> "String: " + s;
        case null                    -> "Nulo";
        default                      -> "Outro tipo";
    };
}

// Exemplo prático — processamento de eventos
String processarEvento(Event event) {
    return switch (event) {
        case LoginEvent e when e.isAdmin()    -> handleAdminLogin(e);
        case LoginEvent e                     -> handleUserLogin(e);
        case PaymentEvent e when e.amount() > 10000 -> handleLargePayment(e);
        case PaymentEvent e                   -> handlePayment(e);
        case ErrorEvent e when e.isCritical() -> handleCriticalError(e);
        case ErrorEvent e                     -> handleError(e);
    };
}
```

### 13.5 Record Patterns — Desestruturação

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}
record Circle(Point center, double radius) {}

// Desestruturação de records no instanceof
if (obj instanceof Point(int x, int y)) {
    System.out.println("Ponto em (" + x + ", " + y + ")");
}

// Desestruturação aninhada
if (obj instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
    double length = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
    System.out.println("Comprimento: " + length);
}

// Record patterns no switch
String descreverForma(Object forma) {
    return switch (forma) {
        case Point(int x, int y) -> "Ponto(%d, %d)".formatted(x, y);
        case Circle(Point(int x, int y), double r) ->
            "Círculo em (%d, %d) com raio %.1f".formatted(x, y, r);
        case Line(Point(int x1, int y1), Point(int x2, int y2)) ->
            "Linha de (%d,%d) até (%d,%d)".formatted(x1, y1, x2, y2);
        default -> "Forma desconhecida";
    };
}
```

### 13.6 Unnamed Variables e Patterns (_)

Quando um valor capturado pelo pattern **não será usado**, use `_` para indicar intenção:

```java
// _ indica que o valor não é necessário
if (obj instanceof Point(int x, _)) {
    System.out.println("Coordenada X: " + x); // y não é usado
}

// No switch
String tipo(Object obj) {
    return switch (obj) {
        case Point _   -> "Ponto";
        case Circle _  -> "Círculo";
        case Line _    -> "Linha";
        default        -> "Desconhecido";
    };
}

// Em loops — variável de iteração não usada
int count = 0;
for (var _ : lista) {
    count++;
}

// Em try-catch — exceção não usada
try {
    Integer.parseInt(valor);
} catch (NumberFormatException _) {
    return valorPadrao;
}

// Em lambdas — parâmetro não usado
map.computeIfAbsent(key, _ -> new ArrayList<>());
```

---

## 14. Sealed Classes e Records — Modelagem de Domínio

### 14.1 Conceito de Sealed Classes

`sealed` restringe **quais classes podem estender** uma classe ou implementar uma interface. Isso permite que o compilador garanta **exhaustiveness** no `switch`:

```java
// Hierarquia fechada — somente as subclasses listadas em 'permits' são permitidas
public sealed interface Shape
    permits Circle, Rectangle, Triangle {
}

public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double a, double b, double c) implements Shape {}

// Tentar criar outra implementação gera erro de compilação:
// public record Pentagon(...) implements Shape {} // ❌ Não listado em permits
```

### 14.2 Exhaustiveness no Switch

Com sealed classes, o compilador sabe que todas as subclasses estão cobertas — `default` é desnecessário:

```java
// O compilador garante que todos os casos estão cobertos
double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t  -> {
            double s = (t.a() + t.b() + t.c()) / 2;
            yield Math.sqrt(s * (s - t.a()) * (s - t.b()) * (s - t.c()));
        }
        // Sem default! O compilador sabe que Shape só pode ser Circle, Rectangle ou Triangle
    };
}
```

Se no futuro uma nova subclasse for adicionada ao `permits`, todos os `switch` que a usam **falharão na compilação**, forçando a atualização.

### 14.3 Hierarquias com Múltiplos Níveis

```java
public sealed interface Expr permits Literal, BinaryOp, UnaryOp {
}

public record Literal(double value) implements Expr {}

public sealed interface BinaryOp extends Expr permits Add, Sub, Mul, Div {}
public record Add(Expr left, Expr right) implements BinaryOp {}
public record Sub(Expr left, Expr right) implements BinaryOp {}
public record Mul(Expr left, Expr right) implements BinaryOp {}
public record Div(Expr left, Expr right) implements BinaryOp {}

public sealed interface UnaryOp extends Expr permits Neg {}
public record Neg(Expr operand) implements UnaryOp {}

// Avaliador com pattern matching recursivo
double evaluate(Expr expr) {
    return switch (expr) {
        case Literal(double v)      -> v;
        case Add(Expr l, Expr r)    -> evaluate(l) + evaluate(r);
        case Sub(Expr l, Expr r)    -> evaluate(l) - evaluate(r);
        case Mul(Expr l, Expr r)    -> evaluate(l) * evaluate(r);
        case Div(Expr l, Expr r)    -> evaluate(l) / evaluate(r);
        case Neg(Expr operand)      -> -evaluate(operand);
    };
}

// Uso: (3 + 5) * 2
Expr expr = new Mul(new Add(new Literal(3), new Literal(5)), new Literal(2));
double result = evaluate(expr); // 16.0
```

### 14.4 Sealed Classes para Resultado de Operações

```java
// Result type — alternativa a exceptions para erros esperados
public sealed interface Result<T> permits Result.Success, Result.Failure {
    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(String error, Exception cause) implements Result<T> {}

    default T orElse(T fallback) {
        return switch (this) {
            case Success<T>(T value) -> value;
            case Failure<T> _ -> fallback;
        };
    }

    default T orElseThrow() {
        return switch (this) {
            case Success<T>(T value) -> value;
            case Failure<T>(String msg, Exception cause) ->
                throw new RuntimeException(msg, cause);
        };
    }
}

// Uso em service
Result<User> findUser(long id) {
    try {
        User user = userRepository.findById(id).orElse(null);
        if (user == null) {
            return new Result.Failure<>("Usuário não encontrado", null);
        }
        return new Result.Success<>(user);
    } catch (Exception e) {
        return new Result.Failure<>("Erro ao buscar usuário", e);
    }
}

// Consumo
String nome = findUser(42)
    .orElse(User.ANONYMOUS)
    .name();
```

### 14.5 Records Avançados

```java
// Record com validação no construtor canônico
public record Email(String value) {
    public Email {
        if (value == null || !value.matches("^[\\w.-]+@[\\w.-]+\\.[a-zA-Z]{2,}$")) {
            throw new IllegalArgumentException("Email inválido: " + value);
        }
        value = value.toLowerCase().trim(); // Normalização — reatribuição permitida no compact constructor
    }
}

// Record com métodos derivados
public record Money(BigDecimal amount, Currency currency) {
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Moedas diferentes");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    public Money multiply(int quantity) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(quantity)), this.currency);
    }

    public boolean isPositive() {
        return amount.compareTo(BigDecimal.ZERO) > 0;
    }
}

// Record genérico
public record Pair<A, B>(A first, B second) {
    public <C> Pair<C, B> mapFirst(Function<A, C> fn) {
        return new Pair<>(fn.apply(first), second);
    }

    public <C> Pair<A, C> mapSecond(Function<B, C> fn) {
        return new Pair<>(first, fn.apply(second));
    }
}

// Record implementando interface
public sealed interface ApiResponse<T> permits ApiResponse.Ok, ApiResponse.Error {
    record Ok<T>(T data, Map<String, String> headers) implements ApiResponse<T> {}
    record Error<T>(int statusCode, String message) implements ApiResponse<T> {}
}

// Local records — declarados dentro de métodos
List<String> topProducts(List<Order> orders) {
    record ProductRevenue(String name, BigDecimal total) {}

    return orders.stream()
        .flatMap(o -> o.items().stream())
        .collect(Collectors.groupingBy(
            Item::productName,
            Collectors.reducing(BigDecimal.ZERO, Item::price, BigDecimal::add)
        ))
        .entrySet().stream()
        .map(e -> new ProductRevenue(e.getKey(), e.getValue()))
        .sorted(Comparator.comparing(ProductRevenue::total).reversed())
        .limit(10)
        .map(ProductRevenue::name)
        .toList();
}
```

---

## 15. Foreign Function & Memory API (Project Panama)

### 15.1 Conceito e Motivação

A **Foreign Function & Memory API** (JEP 454, final no Java 22) substitui JNI para chamadas a código nativo (C, C++, Rust). É mais segura, performática e não requer código C glue:

```
┌─────────────────────┐          ┌─────────────────────────┐
│     Java (JVM)      │          │    Biblioteca Nativa     │
│                     │          │     (C / C++ / Rust)     │
│  ┌───────────────┐  │  FFM API │  ┌───────────────────┐  │
│  │ MemorySegment │◄─┼──────────┼─►│ struct / buffer   │  │
│  │ Arena         │  │          │  │                   │  │
│  │ Linker        │◄─┼──────────┼─►│ função nativa     │  │
│  └───────────────┘  │          │  └───────────────────┘  │
└─────────────────────┘          └─────────────────────────┘
```

### 15.2 Alocação de Memória Off-Heap

```java
// Arena gerencia o ciclo de vida da memória — desaloca automaticamente no close()
try (Arena arena = Arena.ofConfined()) {
    // Aloca 100 ints off-heap (fora do GC do Java)
    MemorySegment segment = arena.allocate(ValueLayout.JAVA_INT, 100);

    // Escrita
    for (int i = 0; i < 100; i++) {
        segment.setAtIndex(ValueLayout.JAVA_INT, i, i * 10);
    }

    // Leitura
    int valor = segment.getAtIndex(ValueLayout.JAVA_INT, 42); // 420

    // Alocar string nativa (null-terminated)
    MemorySegment nativeStr = arena.allocateFrom("Olá, mundo!");

    System.out.println("Valor na posição 42: " + valor);
    System.out.println("String nativa: " + nativeStr.getString(0));
} // Toda a memória alocada é liberada automaticamente aqui
```

**Tipos de Arena:**

```java
// Confined — acessível apenas pela thread que criou (melhor performance)
try (Arena arena = Arena.ofConfined()) { /* ... */ }

// Shared — acessível por múltiplas threads
try (Arena arena = Arena.ofShared()) { /* ... */ }

// Auto — memória é liberada pelo GC (sem escopo definido)
Arena auto = Arena.ofAuto();

// Global — vive durante toda a execução da JVM
Arena global = Arena.global();
```

### 15.3 Chamando Funções Nativas (FFM)

```java
// Chamando strlen() da libc
long chamarStrlen(String texto) throws Throwable {
    // 1. Obtém o linker nativo (ponte Java ↔ C)
    Linker linker = Linker.nativeLinker();

    // 2. Busca o símbolo 'strlen' na biblioteca padrão do SO
    SymbolLookup stdlib = linker.defaultLookup();
    MemorySegment strlenAddr = stdlib.find("strlen")
        .orElseThrow(() -> new RuntimeException("strlen não encontrada"));

    // 3. Define a assinatura da função: long strlen(char*)
    FunctionDescriptor descriptor = FunctionDescriptor.of(
        ValueLayout.JAVA_LONG,     // retorno: size_t (long)
        ValueLayout.ADDRESS         // parâmetro: const char*
    );

    // 4. Cria o method handle (ponte de chamada)
    MethodHandle strlen = linker.downcallHandle(strlenAddr, descriptor);

    // 5. Prepara o argumento e chama
    try (Arena arena = Arena.ofConfined()) {
        MemorySegment nativeStr = arena.allocateFrom(texto);
        return (long) strlen.invoke(nativeStr);
    }
}

// Uso:
long len = chamarStrlen("Hello, World!"); // 13
```

### 15.4 Trabalhando com Structs Nativos

```java
// Struct C equivalente:
// struct Point { double x; double y; };

// Definição do layout do struct em Java
StructLayout POINT_LAYOUT = MemoryLayout.structLayout(
    ValueLayout.JAVA_DOUBLE.withName("x"),
    ValueLayout.JAVA_DOUBLE.withName("y")
);

// VarHandles para acessar os campos
VarHandle xHandle = POINT_LAYOUT.varHandle(
    MemoryLayout.PathElement.groupElement("x"));
VarHandle yHandle = POINT_LAYOUT.varHandle(
    MemoryLayout.PathElement.groupElement("y"));

void exemploStruct() {
    try (Arena arena = Arena.ofConfined()) {
        // Aloca um struct Point
        MemorySegment point = arena.allocate(POINT_LAYOUT);

        // Define valores dos campos
        xHandle.set(point, 0L, 3.14);
        yHandle.set(point, 0L, 2.71);

        // Lê valores
        double x = (double) xHandle.get(point, 0L);
        double y = (double) yHandle.get(point, 0L);
        System.out.printf("Point(%.2f, %.2f)%n", x, y);

        // Array de structs
        MemorySegment points = arena.allocate(POINT_LAYOUT, 10); // 10 Points
        for (int i = 0; i < 10; i++) {
            MemorySegment element = points.asSlice(
                POINT_LAYOUT.byteSize() * i, POINT_LAYOUT.byteSize());
            xHandle.set(element, 0L, (double) i);
            yHandle.set(element, 0L, (double) (i * 2));
        }
    }
}
```

### 15.5 Exemplo Prático — Integração com Biblioteca C

```java
// Chamando qsort() da libc com callback Java
void exemploQsort() throws Throwable {
    Linker linker = Linker.nativeLinker();

    // Busca qsort: void qsort(void *base, size_t nmemb, size_t size, int (*compar)(const void*, const void*))
    MethodHandle qsort = linker.downcallHandle(
        linker.defaultLookup().find("qsort").orElseThrow(),
        FunctionDescriptor.ofVoid(
            ValueLayout.ADDRESS,    // base
            ValueLayout.JAVA_LONG,  // nmemb
            ValueLayout.JAVA_LONG,  // size
            ValueLayout.ADDRESS     // compar (function pointer)
        )
    );

    // Cria callback Java que será chamada pelo C
    MethodHandle comparador = MethodHandles.lookup().findStatic(
        getClass(), "compararInts",
        MethodType.methodType(int.class, MemorySegment.class, MemorySegment.class)
    );

    try (Arena arena = Arena.ofConfined()) {
        // Cria o function pointer para o callback
        MemorySegment comparadorNativo = linker.upcallStub(
            comparador,
            FunctionDescriptor.of(
                ValueLayout.JAVA_INT,
                ValueLayout.ADDRESS,
                ValueLayout.ADDRESS
            ),
            arena
        );

        // Prepara array nativo
        int[] dados = {5, 3, 8, 1, 9, 2, 7, 4, 6};
        MemorySegment array = arena.allocateFrom(ValueLayout.JAVA_INT, dados);

        // Chama qsort
        qsort.invoke(array, (long) dados.length, (long) ValueLayout.JAVA_INT.byteSize(), comparadorNativo);

        // Lê resultado
        for (int i = 0; i < dados.length; i++) {
            System.out.print(array.getAtIndex(ValueLayout.JAVA_INT, i) + " ");
        }
        // Output: 1 2 3 4 5 6 7 8 9
    }
}

static int compararInts(MemorySegment a, MemorySegment b) {
    int va = a.reinterpret(ValueLayout.JAVA_INT.byteSize()).get(ValueLayout.JAVA_INT, 0);
    int vb = b.reinterpret(ValueLayout.JAVA_INT.byteSize()).get(ValueLayout.JAVA_INT, 0);
    return Integer.compare(va, vb);
}
```

---

## 16. Stream Gatherers

### 16.1 Conceito e Motivação

**Stream Gatherers** (JEP 485, final no Java 24) permitem criar **operações intermediárias customizadas** para streams — preenchendo a lacuna que existia entre `map/filter` (simples demais) e `Collector` (só opera no terminal):

```
Stream pipeline:
  source → [map] → [filter] → [gather] → [collect]
                                  ↑
                          Operação intermediária
                          customizada (novo!)
```

### 16.2 Gatherers Built-in

```java
import java.util.stream.Gatherers;

List<Integer> numeros = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// windowFixed — janelas de tamanho fixo
List<List<Integer>> janelas = numeros.stream()
    .gather(Gatherers.windowFixed(3))
    .toList();
// [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]

// windowSliding — janelas deslizantes
List<List<Integer>> deslizantes = numeros.stream()
    .gather(Gatherers.windowSliding(3))
    .toList();
// [[1, 2, 3], [2, 3, 4], [3, 4, 5], ..., [8, 9, 10]]

// fold — acumula em um único valor (como reduce, mas com tipo diferente)
String concatenado = numeros.stream()
    .gather(Gatherers.fold(
        () -> new StringBuilder(),          // valor inicial
        (sb, n) -> sb.append(n).append(",") // acumulador
    ))
    .map(StringBuilder::toString)
    .findFirst()
    .orElse("");
// "1,2,3,4,5,6,7,8,9,10,"

// scan — como fold, mas emite cada estado intermediário
List<Integer> somaAcumulada = numeros.stream()
    .gather(Gatherers.scan(() -> 0, Integer::sum))
    .toList();
// [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

// mapConcurrent — processamento paralelo mantendo a ordem
List<String> resultados = urls.stream()
    .gather(Gatherers.mapConcurrent(10, url -> fetchContent(url)))
    .toList();
// Até 10 chamadas em paralelo, resultado na ordem original
```

### 16.3 Criando Gatherers Customizados

```java
// Gatherer que remove duplicatas consecutivas
static <T> Gatherer<T, ?, T> distinctConsecutive() {
    return Gatherer.ofSequential(
        // Inicializador: estado interno (último elemento visto)
        () -> new Object() { T last = null; boolean first = true; },
        // Integrador: recebe cada elemento e decide se emite
        (state, element, downstream) -> {
            if (state.first || !Objects.equals(element, state.last)) {
                state.last = element;
                state.first = false;
                return downstream.push(element); // Emite o elemento
            }
            return true; // Continua processando sem emitir
        }
    );
}

// Uso:
List<Integer> resultado = List.of(1, 1, 2, 2, 2, 3, 1, 1, 4)
    .stream()
    .gather(distinctConsecutive())
    .toList();
// [1, 2, 3, 1, 4]
```

```java
// Gatherer que agrupa em batches e processa cada batch
static <T, R> Gatherer<T, ?, R> batchProcess(int batchSize, Function<List<T>, R> processor) {
    return Gatherer.ofSequential(
        ArrayList<T>::new,
        (batch, element, downstream) -> {
            batch.add(element);
            if (batch.size() >= batchSize) {
                R result = processor.apply(new ArrayList<>(batch));
                batch.clear();
                return downstream.push(result);
            }
            return true;
        },
        // Finalizador: processa o batch incompleto restante
        (batch, downstream) -> {
            if (!batch.isEmpty()) {
                downstream.push(processor.apply(batch));
            }
        }
    );
}

// Uso: insere registros em batches de 100
long totalInseridos = registros.stream()
    .gather(batchProcess(100, batch -> {
        jdbcTemplate.batchUpdate(sql, batch);
        return batch.size();
    }))
    .mapToLong(Integer::longValue)
    .sum();
```

```java
// Gatherer com rate limiting — emite no máximo N elementos por segundo
static <T> Gatherer<T, ?, T> rateLimited(int maxPerSecond) {
    return Gatherer.ofSequential(
        () -> new Object() {
            long windowStart = System.nanoTime();
            int count = 0;
        },
        (state, element, downstream) -> {
            long now = System.nanoTime();
            if (now - state.windowStart > 1_000_000_000L) {
                state.windowStart = now;
                state.count = 0;
            }
            if (state.count >= maxPerSecond) {
                try {
                    long sleepMs = (1_000_000_000L - (now - state.windowStart)) / 1_000_000;
                    if (sleepMs > 0) Thread.sleep(sleepMs);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return false;
                }
                state.windowStart = System.nanoTime();
                state.count = 0;
            }
            state.count++;
            return downstream.push(element);
        }
    );
}

// Uso: no máximo 100 chamadas/segundo à API
resultados = items.stream()
    .gather(rateLimited(100))
    .map(item -> callApi(item))
    .toList();
```

### 16.4 Composição de Gatherers

```java
// Gatherers podem ser compostos com andThen
Gatherer<String, ?, List<String>> pipeline =
    Gatherers.<String>windowFixed(5)
        .andThen(distinctConsecutive());

List<List<String>> result = dados.stream()
    .gather(pipeline)
    .toList();
```

---

## 17. Flexible Constructor Bodies

### 17.1 Conceito

**Flexible Constructor Bodies** (JEP 492, final no Java 25) permitem executar **statements antes da chamada `super()`** ou `this()`. Antes do Java 25, a primeira instrução de um construtor obrigatoriamente tinha que ser `super()` ou `this()`.

### 17.2 Validação Antes do super()

```java
// ❌ Antes do Java 25 — validação impossível antes do super()
public class PositiveCounter extends Counter {
    public PositiveCounter(int initialValue) {
        super(initialValue); // Obrigatoriamente primeiro
        if (initialValue < 0) {
            throw new IllegalArgumentException("Valor deve ser positivo");
            // Problema: o objeto já foi parcialmente construído pelo super()!
        }
    }
}

// ✅ Java 25 — validação antes do super()
public class PositiveCounter extends Counter {
    public PositiveCounter(int initialValue) {
        if (initialValue < 0) {
            throw new IllegalArgumentException("Valor deve ser positivo");
        }
        super(initialValue); // Só chama super() depois de validar
    }
}
```

### 17.3 Preparação de Argumentos

```java
// ❌ Antes — factory method estático para contornar a limitação
public class DatabaseConnection extends Connection {
    private static Properties buildProps(String url) {
        Properties props = new Properties();
        props.setProperty("url", url);
        props.setProperty("timeout", "5000");
        return props;
    }

    public DatabaseConnection(String url) {
        super(buildProps(url)); // Workaround: método estático auxiliar
    }
}

// ✅ Java 25 — lógica direta antes do super()
public class DatabaseConnection extends Connection {
    public DatabaseConnection(String url) {
        Properties props = new Properties();
        props.setProperty("url", url);
        props.setProperty("timeout", "5000");
        // Pode usar variáveis locais, mas NÃO pode acessar 'this' antes do super()
        super(props);
    }
}
```

### 17.4 Regras e Restrições

```java
public class Exemplo extends Base {
    private final String nome;

    public Exemplo(String nome) {
        // ✅ Pode: declarar variáveis locais
        String nomeNormalizado = nome.trim().toLowerCase();

        // ✅ Pode: chamadas a métodos estáticos
        Objects.requireNonNull(nomeNormalizado, "Nome não pode ser nulo");

        // ✅ Pode: try-catch
        try {
            validate(nomeNormalizado);
        } catch (Exception e) {
            throw new IllegalArgumentException("Nome inválido", e);
        }

        // ❌ NÃO pode: acessar this.campo antes do super()
        // this.nome = nomeNormalizado; // ERRO DE COMPILAÇÃO

        // ❌ NÃO pode: chamar métodos de instância antes do super()
        // this.getNome(); // ERRO DE COMPILAÇÃO

        super(nomeNormalizado); // Chamada ao super com argumento preparado

        // ✅ Agora pode acessar this
        this.nome = nomeNormalizado;
    }

    private static void validate(String nome) {
        if (nome.length() < 2) throw new IllegalArgumentException("Nome muito curto");
    }
}
```

---

## 18. Module Import Declarations e Outras Conveniências

### 18.1 Module Import Declarations

**Module Import Declarations** (JEP 511, final no Java 25) simplificam imports ao importar todos os pacotes exportados por um módulo:

```java
// ❌ Antes — múltiplos imports individuais
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import java.io.IOException;
import java.io.BufferedReader;
import java.nio.file.Files;
import java.nio.file.Path;

// ✅ Java 25 — import do módulo inteiro
import module java.base; // Importa todos os pacotes públicos do módulo java.base

public class Exemplo {
    // List, Map, Set, Stream, Collectors, Path, Files — tudo disponível
    public List<String> lerArquivo(Path caminho) throws IOException {
        return Files.readAllLines(caminho).stream()
            .filter(line -> !line.isBlank())
            .map(String::trim)
            .collect(Collectors.toList());
    }
}
```

```java
// Importando módulos específicos
import module java.base;       // Coleções, I/O, streams, concurrency, etc.
import module java.sql;        // JDBC (java.sql e javax.sql)
import module java.net.http;   // HttpClient

public class ApiService {
    // Todas as classes de java.base, java.sql e java.net.http disponíveis
    HttpClient client = HttpClient.newHttpClient();

    Connection getConnection(String url) throws SQLException {
        return DriverManager.getConnection(url);
    }
}
```

**Resolução de conflitos:**

```java
import module java.base;
import module java.sql;

// Ambos os módulos exportam 'java.sql.Date' e 'java.util.Date'
// Se houver ambiguidade, use import explícito para desambiguar:
import java.util.Date; // Este Date vence sobre java.sql.Date

public class Exemplo {
    Date utilDate = new Date();                          // java.util.Date (import explícito)
    java.sql.Date sqlDate = new java.sql.Date(0L);      // Usa nome completo para o outro
}
```

### 18.2 Implicitly Declared Classes (Simple Main)

Desde Java 21 (preview, refinado até Java 25), é possível escrever programas simples sem declarar classe, método `main` com `String[] args` ou `static`:

```java
// ❌ Antes — boilerplate para Hello World
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// ✅ Java 25 — programa simples (arquivo HelloWorld.java)
void main() {
    println("Hello, World!");
}
```

```java
// Programa com interação — println() e readln() disponíveis automaticamente
void main() {
    String nome = readln("Qual é o seu nome? ");
    println("Olá, " + nome + "!");

    int idade = Integer.parseInt(readln("Qual a sua idade? "));
    if (idade >= 18) {
        println("Você é maior de idade.");
    } else {
        println("Você é menor de idade.");
    }
}
```

```java
// Programa com métodos — sem necessidade de static
String saudar(String nome) {
    return "Olá, " + nome + "!";
}

void main() {
    println(saudar("Maria"));
    println(saudar("João"));
}
```

### 18.3 Markdown em JavaDoc

Java 23+ permite escrever JavaDoc usando **Markdown** em vez de HTML. Para indicar que um comentário usa Markdown, basta usar `///` (três barras) em vez do bloco `/** ... */` tradicional:

```java
/// Calcula o preço final com desconto.
///
/// O desconto é aplicado **proporcionalmente** ao valor total:
/// - Compras acima de R$ 100: 10% de desconto
/// - Compras acima de R$ 500: 15% de desconto
/// - Compras acima de R$ 1000: 20% de desconto
///
/// Exemplo:
/// ```java
/// BigDecimal preco = calcularPrecoFinal(new BigDecimal("250.00"));
/// // preco = 225.00 (10% de desconto)
/// ```
///
/// @param valorOriginal o valor antes do desconto (deve ser positivo)
/// @return o valor com desconto aplicado
/// @throws IllegalArgumentException se o valor for negativo
public BigDecimal calcularPrecoFinal(BigDecimal valorOriginal) {
    // ...
}
```

**Incluindo trechos de código-fonte no JavaDoc com `@snippet`:**

A tag `{@snippet}` (JEP 413, Java 18+) permite incluir exemplos de código no JavaDoc com **syntax highlighting**, **marcação de regiões** e **referência a código externo** — substituindo `<pre>{@code ...}</pre>` e `{@link}`.

```java
/// Serviço de autenticação.
///
/// Exemplo de uso básico:
/// {@snippet :
/// AuthService auth = new AuthService(tokenProvider);
/// Token token = auth.authenticate("user@example.com", "senha123");
/// System.out.println("Token: " + token.value());
/// }
public class AuthService {
    // ...
}
```

**Marcação de regiões com `@highlight`, `@replace` e `@link`:**

```java
/// Como configurar o pool de conexões:
/// {@snippet :
/// var config = new PoolConfig();
/// config.setMaxSize(20);           // @highlight substring="setMaxSize"
/// config.setMinIdle(5);            // @highlight substring="setMinIdle"
/// config.setTimeout(Duration.ofSeconds(30));
/// DataSource ds = PoolFactory.create(config);  // @highlight region="result" type="highlighted"
/// Connection conn = ds.getConnection();        // @end
/// }
public class PoolConfig {
    // ...
}
```

Os tipos de marcação disponíveis:

| Tag | Efeito no JavaDoc gerado |
|-----|--------------------------|
| `@highlight substring="X"` | Destaca o texto `X` na linha (negrito/cor) |
| `@highlight region="nome"` ... `@end` | Destaca um bloco de linhas |
| `@highlight type="highlighted"` | Destaque visual (padrão) |
| `@highlight type="italic"` | Itálico |
| `@highlight type="bold"` | Negrito |
| `@replace substring="X" replacement="Y"` | Substitui `X` por `Y` na saída (útil para simplificar exemplos) |
| `@link substring="X" target="Classe#método"` | Torna `X` um link clicável para o JavaDoc de outro elemento |

**Referenciando código-fonte externo (arquivo real):**

Em vez de duplicar código no JavaDoc, é possível referenciar um trecho de um arquivo `.java` existente:

```java
/// Exemplo de uso completo:
/// {@snippet file="ExemploAuthService.java" region="autenticacao"}
public class AuthService {
    // ...
}
```

O arquivo referenciado usa comentários `@start` e `@end` para delimitar a região:

```java
// Arquivo: src/snippet-files/ExemploAuthService.java
public class ExemploAuthService {

    void exemplo() {
        // @start region="autenticacao"
        AuthService auth = new AuthService(tokenProvider);
        Token token = auth.authenticate("user@example.com", "senha123");

        if (token.isExpired()) {
            token = auth.refresh(token);
        }

        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token.value());
        // @end
    }
}
```

**Comparação: `@snippet` vs abordagem tradicional:**

```java
// ❌ Abordagem tradicional — HTML manual, sem syntax highlighting
/**
 * Exemplo:
 * <pre>{@code
 * AuthService auth = new AuthService(provider);
 * Token token = auth.authenticate("user", "pass");
 * }</pre>
 */

// ✅ Markdown JavaDoc com @snippet — syntax highlighting, regiões, validação
/// Exemplo:
/// {@snippet :
/// AuthService auth = new AuthService(provider);
/// Token token = auth.authenticate("user", "pass");  // @highlight substring="authenticate"
/// }
```

---

## 19. Java NIO Channels — I/O de Alto Desempenho

O pacote `java.nio.channels` fornece **channels nativos de I/O** para operações de leitura/escrita em arquivos, sockets de rede e pipes. Com Virtual Threads (Java 21+), esses channels ganham nova relevância: o modelo bloqueante se torna tão eficiente quanto o non-blocking, sem a complexidade de `Selector`.

### 19.1 Hierarquia de Channels

```
                          Channel (interface)
                              │
              ┌───────────────┼───────────────┐
              │               │               │
        ReadableByteChannel  WritableByteChannel  NetworkChannel
              │               │               │
              └───────┬───────┘               │
                      │                       │
                 ByteChannel                  │
                      │                       │
         ┌────────────┼────────────┐          │
         │            │            │          │
    FileChannel  SocketChannel  ServerSocketChannel
                      │
              DatagramChannel

Channels assíncronos (separados):
    AsynchronousFileChannel
    AsynchronousSocketChannel
    AsynchronousServerSocketChannel
```

| Channel | Uso |
|---------|-----|
| `FileChannel` | Leitura/escrita de arquivos com suporte a memory-mapped I/O e transferência direta |
| `SocketChannel` | Comunicação TCP (cliente) — bloqueante ou non-blocking |
| `ServerSocketChannel` | Aceita conexões TCP (servidor) |
| `DatagramChannel` | Comunicação UDP |
| `Pipe.SourceChannel` / `Pipe.SinkChannel` | Comunicação unidirecional entre threads |
| `AsynchronousFileChannel` | I/O de arquivo assíncrono com callbacks |
| `AsynchronousSocketChannel` | I/O de rede assíncrono com callbacks |

### 19.2 FileChannel — I/O de Arquivo de Alto Desempenho

`FileChannel` oferece operações avançadas que `InputStream`/`OutputStream` não possuem: **memory-mapped files**, **transferência direta** (zero-copy) e **locks de arquivo**.

```java
// Leitura com FileChannel + ByteBuffer
void lerArquivoComChannel(Path caminho) throws IOException {
    try (FileChannel channel = FileChannel.open(caminho, StandardOpenOption.READ)) {
        ByteBuffer buffer = ByteBuffer.allocate(4096);

        while (channel.read(buffer) != -1) {
            buffer.flip(); // Prepara para leitura (limit = position, position = 0)
            while (buffer.hasRemaining()) {
                System.out.print((char) buffer.get());
            }
            buffer.clear(); // Prepara para próxima escrita no buffer
        }
    }
}

// Escrita com FileChannel
void escreverComChannel(Path caminho, String conteudo) throws IOException {
    try (FileChannel channel = FileChannel.open(caminho,
            StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING)) {
        ByteBuffer buffer = ByteBuffer.wrap(conteudo.getBytes(StandardCharsets.UTF_8));
        channel.write(buffer);
    }
}

// Leitura em posição específica (random access)
void lerPosicaoEspecifica(Path caminho, long posicao, int tamanho) throws IOException {
    try (FileChannel channel = FileChannel.open(caminho, StandardOpenOption.READ)) {
        ByteBuffer buffer = ByteBuffer.allocate(tamanho);
        channel.read(buffer, posicao); // Lê a partir da posição, sem alterar o position do channel
        buffer.flip();
        System.out.println(StandardCharsets.UTF_8.decode(buffer));
    }
}
```

**Memory-Mapped Files — mapeamento de arquivo na memória:**

```java
// Mapeia o arquivo inteiro na memória — ideal para arquivos grandes ou acesso randômico frequente
void exemploMemoryMapped(Path caminho) throws IOException {
    try (FileChannel channel = FileChannel.open(caminho, StandardOpenOption.READ, StandardOpenOption.WRITE)) {
        long tamanho = channel.size();

        // MapMode.READ_WRITE: alterações são refletidas no arquivo
        // MapMode.READ_ONLY: apenas leitura
        // MapMode.PRIVATE: copy-on-write (alterações não afetam o arquivo)
        MappedByteBuffer mapped = channel.map(FileChannel.MapMode.READ_WRITE, 0, tamanho);

        // Acesso direto à memória mapeada — sem syscalls de leitura
        byte primeiroByte = mapped.get(0);
        mapped.put(0, (byte) 'X'); // Escrita direta no arquivo via memória

        // Busca por padrão em arquivo grande (eficiente, sem carregar tudo na JVM heap)
        for (int i = 0; i < mapped.limit() - 4; i++) {
            if (mapped.get(i) == 'J' && mapped.get(i+1) == 'a'
                && mapped.get(i+2) == 'v' && mapped.get(i+3) == 'a') {
                System.out.println("Encontrou 'Java' na posição: " + i);
            }
        }

        mapped.force(); // Força flush para o disco
    }
}
```

**Transferência direta (zero-copy) entre channels:**

```java
// Copia arquivo usando transferTo — evita cópia entre kernel space e user space
void copiarArquivoZeroCopy(Path origem, Path destino) throws IOException {
    try (FileChannel source = FileChannel.open(origem, StandardOpenOption.READ);
         FileChannel target = FileChannel.open(destino,
             StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {

        long transferido = 0;
        long tamanho = source.size();
        while (transferido < tamanho) {
            transferido += source.transferTo(transferido, tamanho - transferido, target);
        }
    }
}

// Transferência direta de FileChannel para SocketChannel (envio de arquivo pela rede)
void enviarArquivoPelaRede(Path arquivo, SocketChannel socket) throws IOException {
    try (FileChannel fileChannel = FileChannel.open(arquivo, StandardOpenOption.READ)) {
        long tamanho = fileChannel.size();
        long enviado = 0;
        while (enviado < tamanho) {
            enviado += fileChannel.transferTo(enviado, tamanho - enviado, socket);
        }
    }
}
```

**File Locking — travas de arquivo entre processos:**

```java
void exemploFileLock(Path caminho) throws IOException {
    try (FileChannel channel = FileChannel.open(caminho,
            StandardOpenOption.READ, StandardOpenOption.WRITE)) {

        // Lock exclusivo — bloqueia até conseguir (outros processos não podem ler/escrever)
        try (FileLock lock = channel.lock()) {
            ByteBuffer buffer = ByteBuffer.wrap("dados críticos".getBytes(StandardCharsets.UTF_8));
            channel.write(buffer);
        } // Lock liberado automaticamente

        // tryLock — tentativa não-bloqueante
        FileLock lock = channel.tryLock();
        if (lock != null) {
            try {
                // Arquivo travado com sucesso
            } finally {
                lock.release();
            }
        } else {
            System.out.println("Arquivo já está travado por outro processo");
        }

        // Lock parcial — trava apenas uma região do arquivo
        try (FileLock regionLock = channel.lock(100, 200, false)) {
            // Trava bytes 100-299 (exclusivo)
        }

        // Lock compartilhado (shared) — múltiplos leitores, bloqueia escritores
        try (FileLock sharedLock = channel.lock(0, Long.MAX_VALUE, true)) {
            // Leitura compartilhada
        }
    }
}
```

### 19.3 SocketChannel e ServerSocketChannel — I/O de Rede

```java
// Servidor TCP com Virtual Threads (bloqueante — simples e eficiente)
void servidorTcpVirtualThreads(int porta) throws Exception {
    try (ServerSocketChannel server = ServerSocketChannel.open()) {
        server.bind(new InetSocketAddress(porta));
        System.out.println("Servidor ouvindo na porta " + porta);

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            while (true) {
                SocketChannel client = server.accept(); // Bloqueante — OK com virtual threads
                executor.submit(() -> {
                    handleClient(client);
                    return null;
                });
            }
        }
    }
}

void handleClient(SocketChannel client) {
    try (client) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while (client.read(buffer) != -1) {
            buffer.flip();
            // Echo: envia de volta o que recebeu
            client.write(buffer);
            buffer.clear();
        }
    } catch (IOException e) {
        System.err.println("Erro no cliente: " + e.getMessage());
    }
}
```

```java
// Cliente TCP com SocketChannel
String enviarMensagem(String host, int porta, String mensagem) throws IOException {
    try (SocketChannel channel = SocketChannel.open()) {
        channel.connect(new InetSocketAddress(host, porta));

        // Enviar
        ByteBuffer sendBuffer = ByteBuffer.wrap(mensagem.getBytes(StandardCharsets.UTF_8));
        channel.write(sendBuffer);
        channel.shutdownOutput(); // Sinaliza fim do envio

        // Receber resposta
        ByteBuffer receiveBuffer = ByteBuffer.allocate(4096);
        StringBuilder response = new StringBuilder();
        while (channel.read(receiveBuffer) != -1) {
            receiveBuffer.flip();
            response.append(StandardCharsets.UTF_8.decode(receiveBuffer));
            receiveBuffer.clear();
        }
        return response.toString();
    }
}
```

**Servidor non-blocking com Selector (modelo pré-virtual-threads):**

```java
// Selector — multiplexação de I/O sem uma thread por conexão
// Nota: com virtual threads, o modelo bloqueante acima é preferível na maioria dos casos.
// Selector ainda é útil quando se precisa de controle fino sobre o event loop.
void servidorNonBlocking(int porta) throws IOException {
    try (Selector selector = Selector.open();
         ServerSocketChannel server = ServerSocketChannel.open()) {

        server.bind(new InetSocketAddress(porta));
        server.configureBlocking(false); // Non-blocking
        server.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            selector.select(); // Bloqueia até haver eventos

            Iterator<SelectionKey> keys = selector.selectedKeys().iterator();
            while (keys.hasNext()) {
                SelectionKey key = keys.next();
                keys.remove();

                if (key.isAcceptable()) {
                    SocketChannel client = server.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                }

                if (key.isReadable()) {
                    SocketChannel client = (SocketChannel) key.channel();
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    int bytesRead = client.read(buffer);

                    if (bytesRead == -1) {
                        client.close();
                    } else {
                        buffer.flip();
                        client.write(buffer); // Echo
                    }
                }
            }
        }
    }
}
```

### 19.4 Pipe — Channel entre Threads

`java.nio.channels.Pipe` cria um par de channels conectados para comunicação unidirecional entre threads — um `SinkChannel` (escrita) e um `SourceChannel` (leitura):

```java
void exemploPipe() throws Exception {
    Pipe pipe = Pipe.open();
    Pipe.SinkChannel sink = pipe.sink();       // Lado da escrita
    Pipe.SourceChannel source = pipe.source(); // Lado da leitura

    try (var scope = StructuredTaskScope.open()) {
        // Produtor: escreve no SinkChannel
        scope.fork(() -> {
            try (sink) {
                for (int i = 0; i < 10; i++) {
                    String msg = "Mensagem " + i + "\n";
                    ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes(StandardCharsets.UTF_8));
                    sink.write(buffer);
                    Thread.sleep(Duration.ofMillis(100));
                }
            }
            return null;
        });

        // Consumidor: lê do SourceChannel
        scope.fork(() -> {
            try (source) {
                ByteBuffer buffer = ByteBuffer.allocate(256);
                while (source.read(buffer) != -1) {
                    buffer.flip();
                    System.out.print(StandardCharsets.UTF_8.decode(buffer));
                    buffer.clear();
                }
            }
            return null;
        });

        scope.join();
    }
}
```

**Pipe vs BlockingQueue — quando usar cada um:**

| Aspecto | `Pipe` (NIO) | `BlockingQueue` (Channel pattern) |
|---------|-------------|-----------------------------------|
| **Dados** | Bytes (`ByteBuffer`) | Objetos Java tipados |
| **Direção** | Unidirecional | Bidirecional (com dois channels) |
| **Protocolo** | Stream de bytes (sem delimitação) | Mensagens discretas |
| **Integração** | Compatível com `Selector` para multiplexação | Compatível com `StructuredTaskScope` |
| **Caso de uso** | Streams de dados binários, bridge com I/O nativo | Comunicação estruturada entre threads |

### 19.5 AsynchronousFileChannel e AsynchronousSocketChannel

Channels assíncronos permitem I/O com **callbacks** ou **Future**, sem bloquear threads:

```java
// Leitura assíncrona de arquivo com CompletionHandler (callback)
void lerAssincrono(Path caminho) throws Exception {
    try (AsynchronousFileChannel channel = AsynchronousFileChannel.open(caminho, StandardOpenOption.READ)) {
        ByteBuffer buffer = ByteBuffer.allocate(4096);
        CountDownLatch latch = new CountDownLatch(1);

        channel.read(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {
            @Override
            public void completed(Integer bytesRead, ByteBuffer buf) {
                buf.flip();
                System.out.println("Leu " + bytesRead + " bytes: "
                    + StandardCharsets.UTF_8.decode(buf));
                latch.countDown();
            }

            @Override
            public void failed(Throwable exc, ByteBuffer buf) {
                System.err.println("Falha na leitura: " + exc.getMessage());
                latch.countDown();
            }
        });

        latch.await();
    }
}

// Leitura assíncrona com Future (mais simples)
void lerAssincronoComFuture(Path caminho) throws Exception {
    try (AsynchronousFileChannel channel = AsynchronousFileChannel.open(caminho, StandardOpenOption.READ)) {
        ByteBuffer buffer = ByteBuffer.allocate(4096);
        Future<Integer> future = channel.read(buffer, 0);

        // Pode fazer outro trabalho aqui...

        int bytesRead = future.get(); // Bloqueia até completar
        buffer.flip();
        System.out.println("Leu " + bytesRead + " bytes");
    }
}
```

```java
// Servidor TCP assíncrono com AsynchronousServerSocketChannel
void servidorAssincrono(int porta) throws Exception {
    AsynchronousServerSocketChannel server = AsynchronousServerSocketChannel.open()
        .bind(new InetSocketAddress(porta));

    System.out.println("Servidor assíncrono na porta " + porta);

    server.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
        @Override
        public void completed(AsynchronousSocketChannel client, Void attachment) {
            // Aceita próxima conexão imediatamente (recursivo)
            server.accept(null, this);

            // Processa o cliente atual
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            client.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer bytesRead, ByteBuffer buf) {
                    if (bytesRead > 0) {
                        buf.flip();
                        client.write(buf, buf, new CompletionHandler<Integer, ByteBuffer>() {
                            @Override
                            public void completed(Integer result, ByteBuffer b) {
                                try { client.close(); } catch (IOException e) {}
                            }
                            @Override
                            public void failed(Throwable exc, ByteBuffer b) {
                                try { client.close(); } catch (IOException e) {}
                            }
                        });
                    }
                }

                @Override
                public void failed(Throwable exc, ByteBuffer buf) {
                    try { client.close(); } catch (IOException e) {}
                }
            });
        }

        @Override
        public void failed(Throwable exc, Void attachment) {
            System.err.println("Falha ao aceitar: " + exc.getMessage());
        }
    });

    Thread.currentThread().join(); // Mantém o servidor rodando
}
```

**Channels assíncronos vs Virtual Threads — recomendação:**

> Com Virtual Threads (Java 21+), o modelo **bloqueante com `SocketChannel`** é preferível ao modelo assíncrono com callbacks na maioria dos cenários. Virtual threads tornam o I/O bloqueante eficiente sem a complexidade dos callbacks aninhados (`callback hell`). Reserve `AsynchronousSocketChannel` e `Selector` para cenários onde se precisa de controle fino sobre o event loop (ex: proxies de alta performance, protocolos binários customizados).

```java
// ✅ Preferível com Virtual Threads — simples e legível
void clienteSimples(String host, int porta) throws IOException {
    try (SocketChannel ch = SocketChannel.open(new InetSocketAddress(host, porta))) {
        ch.write(ByteBuffer.wrap("ping".getBytes()));
        ByteBuffer resp = ByteBuffer.allocate(1024);
        ch.read(resp);
    }
}

// ⚠️ AsynchronousSocketChannel — mais complexo, útil apenas em cenários específicos
// (proxies de alta performance, protocolos customizados)
```

---

## Referências

### Concorrência (Project Loom)
- [JEP 444 — Virtual Threads](https://openjdk.org/jeps/444) — Final no Java 21
- [JEP 505 — Structured Concurrency](https://openjdk.org/jeps/505) — Final no Java 25
- [JEP 502 — Scoped Values](https://openjdk.org/jeps/502) — Final no Java 25
- [Inside Java — Project Loom](https://inside.java/tag/loom)
- [Java Concurrency in Practice (atualizado)](https://jcip.net/)

### Features de Linguagem
- [JEP 441 — Pattern Matching for switch](https://openjdk.org/jeps/441) — Final no Java 21
- [JEP 440 — Record Patterns](https://openjdk.org/jeps/440) — Final no Java 21
- [JEP 409 — Sealed Classes](https://openjdk.org/jeps/409) — Final no Java 17
- [JEP 456 — Unnamed Variables & Patterns](https://openjdk.org/jeps/456) — Final no Java 22
- [JEP 454 — Foreign Function & Memory API](https://openjdk.org/jeps/454) — Final no Java 22
- [JEP 485 — Stream Gatherers](https://openjdk.org/jeps/485) — Final no Java 24
- [JEP 492 — Flexible Constructor Bodies](https://openjdk.org/jeps/492) — Final no Java 25
- [JEP 511 — Module Import Declarations](https://openjdk.org/jeps/511) — Final no Java 25
- [JEP 512 — Compact Source Files](https://openjdk.org/jeps/512)

### Geral
- [JDK 25 Release Notes](https://openjdk.org/projects/jdk/25/)
- [Spring Boot — Virtual Threads Support](https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html)
- [Java Almanac — Novas Features por Versão](https://javaalmanac.io/)
