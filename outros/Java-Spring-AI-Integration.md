# Integração de Java e Spring Boot com IA

> **Objetivo:** Apresentar as principais ferramentas, frameworks e padrões para integrar aplicações Java e Spring Boot com modelos de inteligência artificial, cobrindo Spring AI, LangChain4J, Semantic Kernel for Java, SDKs nativos dos provedores e integração com bancos de dados vetoriais.

---

## Sumário

1. [Introdução](#1-introdução)
2. [Spring AI](#2-spring-ai)
   - [2.1 Visão Geral](#21-visão-geral)
   - [2.2 Dependências e Configuração](#22-dependências-e-configuração)
   - [2.3 Chat Client API](#23-chat-client-api)
   - [2.4 Prompt Templates](#24-prompt-templates)
   - [2.5 Output Parsers (Saída Estruturada)](#25-output-parsers-saída-estruturada)
   - [2.6 Embeddings](#26-embeddings)
   - [2.7 Vector Stores](#27-vector-stores)
   - [2.8 RAG — Retrieval-Augmented Generation](#28-rag--retrieval-augmented-generation)
   - [2.9 Tool Calling (Function Calling)](#29-tool-calling-function-calling)
   - [2.10 Chat Memory e Conversas Multi-turno](#210-chat-memory-e-conversas-multi-turno)
   - [2.11 Advisors](#211-advisors)
   - [2.12 Modelos de Imagem e Áudio](#212-modelos-de-imagem-e-áudio)
   - [2.13 Observabilidade](#213-observabilidade)
3. [LangChain4J](#3-langchain4j)
   - [3.1 Visão Geral](#31-visão-geral)
   - [3.2 Dependências e Configuração](#32-dependências-e-configuração)
   - [3.3 AI Services (Interface Declarativa)](#33-ai-services-interface-declarativa)
   - [3.4 Chat Memory](#34-chat-memory)
   - [3.5 Tools (Ferramentas / Function Calling)](#35-tools-ferramentas--function-calling)
   - [3.6 Document Loaders e RAG](#36-document-loaders-e-rag)
   - [3.7 Embedding Stores](#37-embedding-stores)
   - [3.8 Agentes e Chains](#38-agentes-e-chains)
4. [Semantic Kernel for Java](#4-semantic-kernel-for-java)
5. [SDKs Nativos dos Provedores](#5-sdks-nativos-dos-provedores)
   - [5.1 OpenAI Java SDK](#51-openai-java-sdk)
   - [5.2 Anthropic Java SDK](#52-anthropic-java-sdk)
   - [5.3 Google Gemini (Vertex AI / Generative AI)](#53-google-gemini-vertex-ai--generative-ai)
6. [Ollama — Modelos Locais](#6-ollama--modelos-locais)
7. [Bancos de Dados Vetoriais](#7-bancos-de-dados-vetoriais)
   - [7.1 pgvector (PostgreSQL)](#71-pgvector-postgresql)
   - [7.2 Redis Vector Search](#72-redis-vector-search)
   - [7.3 Chroma](#73-chroma)
   - [7.4 Weaviate](#74-weaviate)
   - [7.5 Pinecone](#75-pinecone)
8. [Padrões de Arquitetura com IA](#8-padrões-de-arquitetura-com-ia)
   - [8.1 RAG Avançado](#81-rag-avançado)
   - [8.2 Agentic Workflows](#82-agentic-workflows)
   - [8.3 Structured Output e Domain Mapping](#83-structured-output-e-domain-mapping)
9. [Spring AI vs LangChain4J — Comparativo](#9-spring-ai-vs-langchain4j--comparativo)
10. [Boas Práticas](#10-boas-práticas)

---

## 1. Introdução

A integração de aplicações Java/Spring Boot com modelos de linguagem de grande porte (LLMs) tornou-se uma demanda crescente no desenvolvimento moderno. As principais abordagens são:

| Abordagem | Descrição |
|---|---|
| **Spring AI** | Framework oficial do ecossistema Spring, integração nativa com Spring Boot, abstrações para múltiplos provedores |
| **LangChain4J** | Port Java do popular LangChain (Python), foco em RAG, agentes e composição de pipelines |
| **Semantic Kernel** | Framework da Microsoft, abordagem baseada em plugins e planos orquestrados |
| **SDKs nativos** | Acesso direto à API do provedor (OpenAI, Anthropic, Google), máximo controle |
| **Ollama** | Execução local de modelos open-source, sem custo de API e com privacidade dos dados |

A escolha depende do caso de uso, da complexidade da integração e dos requisitos de privacidade dos dados.

---

## 2. Spring AI

### 2.1 Visão Geral

Spring AI é o projeto oficial do ecossistema Spring para integração com IA. Ele fornece:

- Abstração sobre múltiplos provedores (OpenAI, Anthropic, Google Gemini, Azure OpenAI, Ollama, Mistral, Groq, Amazon Bedrock e outros)
- APIs para chat, embeddings, geração de imagens, transcrição de áudio e text-to-speech
- Suporte a RAG com `VectorStore` e `DocumentReader`
- `ChatClient` com API fluente para composição de prompts
- `Advisor` para interceptação e enriquecimento do pipeline de mensagens
- Integração com Spring Boot Actuator e Micrometer para observabilidade

**Versão de referência:** Spring AI 1.0.x (compatível com Spring Boot 3.3+, Java 17+)

**Site oficial:** [spring.io/projects/spring-ai](https://spring.io/projects/spring-ai)

---

### 2.2 Dependências e Configuração

#### BOM e dependência do starter

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Provedor OpenAI (substitua pelo provedor desejado) -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>

    <!-- Alternativa: Anthropic Claude -->
    <!-- <artifactId>spring-ai-anthropic-spring-boot-starter</artifactId> -->

    <!-- Alternativa: Google Gemini (Vertex AI) -->
    <!-- <artifactId>spring-ai-vertex-ai-gemini-spring-boot-starter</artifactId> -->

    <!-- Alternativa: Ollama (modelos locais) -->
    <!-- <artifactId>spring-ai-ollama-spring-boot-starter</artifactId> -->

    <!-- VectorStore com PgVector (PostgreSQL) -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

#### Configuração no `application.yml`

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o
          temperature: 0.7
          max-tokens: 2048
      embedding:
        options:
          model: text-embedding-3-small

    # Exemplo: Anthropic
    anthropic:
      api-key: ${ANTHROPIC_API_KEY}
      chat:
        options:
          model: claude-sonnet-4-6
          max-tokens: 4096

    # Exemplo: Ollama (local)
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: llama3.2
```

---

### 2.3 Chat Client API

O `ChatClient` é a API principal de alto nível do Spring AI, com estilo fluente (builder pattern).

#### Configuração do bean

```java
package br.com.exemplo.ai.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AiConfig {

    @Bean
    ChatClient chatClient(ChatClient.Builder builder) {
        return builder
            .defaultSystem("Você é um assistente especializado em Spring Boot. Responda sempre em português.")
            .build();
    }
}
```

#### Chamadas básicas

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
public class ChatService {

    private final ChatClient chatClient;

    public ChatService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    // Resposta como String simples
    public String responder(String pergunta) {
        return chatClient.prompt()
            .user(pergunta)
            .call()
            .content();
    }

    // Resposta com system prompt personalizado
    public String responderComContexto(String systemPrompt, String pergunta) {
        return chatClient.prompt()
            .system(systemPrompt)
            .user(pergunta)
            .call()
            .content();
    }

    // Streaming da resposta (Flux<String>)
    public Flux<String> responderStream(String pergunta) {
        return chatClient.prompt()
            .user(pergunta)
            .stream()
            .content();
    }
}
```

#### Controller com streaming SSE

```java
package br.com.exemplo.ai.controller;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/chat")
public class ChatController {

    private final ChatService chatService;

    public ChatController(ChatService chatService) {
        this.chatService = chatService;
    }

    @GetMapping
    public String chat(@RequestParam String mensagem) {
        return chatService.responder(mensagem);
    }

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestParam String mensagem) {
        return chatService.responderStream(mensagem);
    }
}
```

---

### 2.4 Prompt Templates

O Spring AI oferece `PromptTemplate` para construção dinâmica de prompts com variáveis.

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.prompt.PromptTemplate;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Service;

import java.util.Map;

@Service
public class TemplateService {

    private final ChatClient chatClient;

    public TemplateService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    // Template inline
    public String gerarResumo(String texto, String idioma) {
        String template = """
            Resuma o seguinte texto em {idioma}, em no máximo 3 frases:
            
            {texto}
            """;

        PromptTemplate promptTemplate = new PromptTemplate(template);
        String prompt = promptTemplate.render(Map.of("texto", texto, "idioma", idioma));

        return chatClient.prompt()
            .user(prompt)
            .call()
            .content();
    }

    // Template a partir de arquivo (src/main/resources/prompts/analise.st)
    public String analisarComArquivo(String produto, String publico) {
        PromptTemplate promptTemplate = new PromptTemplate(
            new ClassPathResource("prompts/analise.st")
        );

        String prompt = promptTemplate.render(Map.of(
            "produto", produto,
            "publicoAlvo", publico
        ));

        return chatClient.prompt()
            .user(prompt)
            .call()
            .content();
    }
}
```

Arquivo `src/main/resources/prompts/analise.st`:

```
Você é um especialista em marketing digital.

Analise o produto "{produto}" para o público-alvo "{publicoAlvo}".

Forneça:
1. Principais diferenciais do produto
2. Pontos de dor do público-alvo
3. 3 ideias de campanha
```

---

### 2.5 Output Parsers (Saída Estruturada)

O Spring AI permite mapear a resposta do modelo diretamente para objetos Java.

#### Mapeamento para record/POJO

```java
package br.com.exemplo.ai.model;

import java.util.List;

public record AnaliseFilme(
    String titulo,
    int anoLancamento,
    String genero,
    List<String> pontosFortex,
    List<String> pontosFracos,
    double nota
) {}
```

```java
package br.com.exemplo.ai.service;

import br.com.exemplo.ai.model.AnaliseFilme;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.converter.BeanOutputConverter;
import org.springframework.stereotype.Service;

@Service
public class FilmeService {

    private final ChatClient chatClient;

    public FilmeService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public AnaliseFilme analisarFilme(String tituloFilme) {
        BeanOutputConverter<AnaliseFilme> converter = new BeanOutputConverter<>(AnaliseFilme.class);

        String resposta = chatClient.prompt()
            .user(u -> u
                .text("""
                    Analise o filme '{titulo}' e retorne no formato JSON.
                    {format}
                    """)
                .param("titulo", tituloFilme)
                .param("format", converter.getFormat())
            )
            .call()
            .content();

        return converter.convert(resposta);
    }
}
```

#### Lista de objetos

```java
public List<Produto> listarProdutos(String categoria) {
    ListOutputConverter<Produto> converter = new ListOutputConverter<>(
        new BeanOutputConverter<>(Produto.class)
    );
    // ... mesma lógica
}
```

#### Usando `entity()` diretamente (Spring AI 1.0+)

```java
public AnaliseFilme analisarFilmeSimplificado(String titulo) {
    return chatClient.prompt()
        .user("Analise o filme '" + titulo + "' com detalhes técnicos e opinião crítica.")
        .call()
        .entity(AnaliseFilme.class);  // conversão automática
}
```

---

### 2.6 Embeddings

Embeddings são representações vetoriais de textos, usadas para busca semântica e RAG.

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.embedding.EmbeddingResponse;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class EmbeddingService {

    private final EmbeddingModel embeddingModel;

    public EmbeddingService(EmbeddingModel embeddingModel) {
        this.embeddingModel = embeddingModel;
    }

    public float[] gerarEmbedding(String texto) {
        return embeddingModel.embed(texto);
    }

    public List<float[]> gerarEmbeddings(List<String> textos) {
        EmbeddingResponse response = embeddingModel.embedForResponse(textos);
        return response.getResults().stream()
            .map(r -> r.getOutput())
            .toList();
    }

    // Calcular similaridade cosseno entre dois vetores
    public double similaridadeCosseno(float[] v1, float[] v2) {
        double dot = 0, norm1 = 0, norm2 = 0;
        for (int i = 0; i < v1.length; i++) {
            dot += v1[i] * v2[i];
            norm1 += v1[i] * v1[i];
            norm2 += v2[i] * v2[i];
        }
        return dot / (Math.sqrt(norm1) * Math.sqrt(norm2));
    }
}
```

---

### 2.7 Vector Stores

O Spring AI abstrai diferentes bancos de dados vetoriais com a interface `VectorStore`.

#### PgVector (PostgreSQL)

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meu_banco
    username: postgres
    password: ${DB_PASSWORD}
  ai:
    vectorstore:
      pgvector:
        initialize-schema: true
        dimensions: 1536          # dimensões do modelo de embedding
        distance-type: COSINE_DISTANCE
```

```sql
-- Extensão necessária no PostgreSQL
CREATE EXTENSION IF NOT EXISTS vector;
```

#### Redis (RedisVectorStore)

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-redis-store-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
  ai:
    vectorstore:
      redis:
        index: documentos-idx
        prefix: doc:
```

#### Uso genérico do VectorStore

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Map;

@Service
public class VectorStoreService {

    private final VectorStore vectorStore;

    public VectorStoreService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void indexarDocumentos(List<String> textos, String fonte) {
        List<Document> documentos = textos.stream()
            .map(texto -> new Document(texto, Map.of("fonte", fonte)))
            .toList();

        vectorStore.add(documentos);
    }

    public List<Document> buscarSimilares(String consulta, int topK) {
        return vectorStore.similaritySearch(
            SearchRequest.query(consulta)
                .withTopK(topK)
                .withSimilarityThreshold(0.7)
        );
    }

    public List<Document> buscarComFiltro(String consulta, String fonte) {
        return vectorStore.similaritySearch(
            SearchRequest.query(consulta)
                .withTopK(5)
                .withFilterExpression("fonte == '" + fonte + "'")
        );
    }
}
```

---

### 2.8 RAG — Retrieval-Augmented Generation

RAG combina busca em base de conhecimento com geração de linguagem natural para respostas fundamentadas em documentos reais.

#### Carregamento de documentos

```xml
<!-- Leitura de PDF -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pdf-document-reader</artifactId>
</dependency>

<!-- Leitura de páginas web (Jsoup) -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-jsoup-document-reader</artifactId>
</dependency>
```

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.document.Document;
import org.springframework.ai.reader.pdf.PagePdfDocumentReader;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@Service
public class DocumentoService {

    private final VectorStore vectorStore;

    public DocumentoService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void indexarPdf(MultipartFile arquivo) throws Exception {
        var reader = new PagePdfDocumentReader(arquivo.getResource());
        List<Document> paginas = reader.get();

        // Divide o texto em chunks menores para melhor precisão
        var splitter = new TokenTextSplitter(800, 100, 5, 10000, true);
        List<Document> chunks = splitter.apply(paginas);

        vectorStore.add(chunks);
    }

    public void indexarPdfClasspath(String caminho) {
        var reader = new PagePdfDocumentReader(new ClassPathResource(caminho));
        var splitter = new TokenTextSplitter();
        vectorStore.add(splitter.apply(reader.get()));
    }
}
```

#### Pipeline RAG completo com QuestionAnswerAdvisor

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.QuestionAnswerAdvisor;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;

@Service
public class RagService {

    private final ChatClient chatClient;
    private final VectorStore vectorStore;

    public RagService(ChatClient.Builder builder, VectorStore vectorStore) {
        this.vectorStore = vectorStore;
        this.chatClient = builder
            .defaultSystem("""
                Você é um assistente especializado. Responda apenas com base nos documentos fornecidos.
                Se a informação não estiver nos documentos, diga que não encontrou a informação.
                """)
            .defaultAdvisors(
                new QuestionAnswerAdvisor(
                    vectorStore,
                    SearchRequest.defaults().withTopK(5).withSimilarityThreshold(0.65)
                )
            )
            .build();
    }

    public String perguntarComContexto(String pergunta) {
        return chatClient.prompt()
            .user(pergunta)
            .call()
            .content();
    }
}
```

#### RAG manual (controle total)

```java
public String ragManual(String pergunta) {
    // 1. Recuperar documentos relevantes
    List<Document> documentos = vectorStore.similaritySearch(
        SearchRequest.query(pergunta).withTopK(4)
    );

    // 2. Montar o contexto
    String contexto = documentos.stream()
        .map(Document::getContent)
        .collect(Collectors.joining("\n\n---\n\n"));

    // 3. Gerar resposta com o contexto
    return chatClient.prompt()
        .user(u -> u
            .text("""
                Contexto dos documentos:
                {contexto}
                
                Pergunta: {pergunta}
                
                Responda com base no contexto acima.
                """)
            .param("contexto", contexto)
            .param("pergunta", pergunta)
        )
        .call()
        .content();
}
```

---

### 2.9 Tool Calling (Function Calling)

Tool Calling permite que o modelo invoque funções Java durante a geração da resposta.

#### Definindo ferramentas com `@Tool`

```java
package br.com.exemplo.ai.tools;

import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Component
public class UtilitarioTools {

    @Tool(description = "Retorna a data e hora atual do sistema")
    public String obterDataHoraAtual() {
        return LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss"));
    }

    @Tool(description = "Converte temperatura de Celsius para Fahrenheit")
    public double celsiusParaFahrenheit(
        @ToolParam(description = "Temperatura em graus Celsius") double celsius
    ) {
        return celsius * 9.0 / 5.0 + 32;
    }
}
```

#### Ferramentas de acesso a banco de dados

```java
package br.com.exemplo.ai.tools;

import br.com.exemplo.produto.ProdutoRepository;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class ProdutoTools {

    private final ProdutoRepository produtoRepository;

    public ProdutoTools(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    @Tool(description = "Busca produtos por categoria no catálogo da loja")
    public String buscarPorCategoria(
        @ToolParam(description = "Nome da categoria do produto") String categoria
    ) {
        return produtoRepository.findByCategoria(categoria).stream()
            .map(p -> p.getNome() + " - R$ " + p.getPreco())
            .collect(Collectors.joining("\n"));
    }

    @Tool(description = "Verifica o estoque disponível de um produto pelo seu nome")
    public String verificarEstoque(
        @ToolParam(description = "Nome do produto a verificar") String nomeProduto
    ) {
        return produtoRepository.findByNomeContainingIgnoreCase(nomeProduto).stream()
            .findFirst()
            .map(p -> "Produto: " + p.getNome() + " | Estoque: " + p.getEstoque() + " unidades")
            .orElse("Produto não encontrado");
    }
}
```

#### Registrando as ferramentas no ChatClient

```java
@Service
public class AssistenteLojaService {

    private final ChatClient chatClient;

    public AssistenteLojaService(ChatClient.Builder builder,
                                  ProdutoTools produtoTools,
                                  UtilitarioTools utilitarioTools) {
        this.chatClient = builder
            .defaultSystem("Você é um assistente de loja virtual. Use as ferramentas disponíveis para responder.")
            .defaultTools(produtoTools, utilitarioTools)
            .build();
    }

    public String atender(String mensagem) {
        return chatClient.prompt()
            .user(mensagem)
            .call()
            .content();
    }
}
```

---

### 2.10 Chat Memory e Conversas Multi-turno

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.MessageChatMemoryAdvisor;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.chat.memory.InMemoryChatMemory;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class ConversaService {

    private final ChatClient chatClient;
    private final Map<String, ChatMemory> sessoes = new ConcurrentHashMap<>();

    public ConversaService(ChatClient.Builder builder) {
        this.chatClient = builder
            .defaultSystem("Você é um assistente prestativo. Mantenha o contexto da conversa.")
            .build();
    }

    public String conversar(String sessaoId, String mensagem) {
        ChatMemory memoria = sessoes.computeIfAbsent(sessaoId, k -> new InMemoryChatMemory());

        return chatClient.prompt()
            .user(mensagem)
            .advisors(new MessageChatMemoryAdvisor(memoria))
            .call()
            .content();
    }

    public void encerrarSessao(String sessaoId) {
        sessoes.remove(sessaoId);
    }
}
```

#### Memória com Redis (sessões distribuídas)

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-redis-chat-memory-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    chat:
      memory:
        redis:
          ttl: 3600  # TTL em segundos
```

```java
// O Spring Boot auto-configura RedisChatMemory
@Service
public class ConversaRedisService {

    private final ChatClient chatClient;
    private final ChatMemory chatMemory;  // injetado automaticamente como RedisChatMemory

    public ConversaRedisService(ChatClient.Builder builder, ChatMemory chatMemory) {
        this.chatMemory = chatMemory;
        this.chatClient = builder.build();
    }

    public String conversar(String sessaoId, String mensagem) {
        return chatClient.prompt()
            .user(mensagem)
            .advisors(new MessageChatMemoryAdvisor(chatMemory, sessaoId, 20))
            .call()
            .content();
    }
}
```

---

### 2.11 Advisors

Advisors são interceptadores do pipeline de mensagens do `ChatClient`, similares a filtros/interceptors do Spring MVC.

#### SafeGuardAdvisor — proteção de conteúdo

```java
package br.com.exemplo.ai.advisor;

import org.springframework.ai.chat.client.AdvisedRequest;
import org.springframework.ai.chat.client.advisor.api.Advisor;
import org.springframework.ai.chat.client.advisor.api.AdvisorChain;
import org.springframework.ai.chat.model.ChatResponse;

import java.util.List;

public class ConteudoSafeGuardAdvisor implements Advisor {

    private static final List<String> PALAVRAS_PROIBIDAS =
        List.of("hack", "exploit", "invadir", "ddos");

    @Override
    public String getName() {
        return "ConteudoSafeGuardAdvisor";
    }

    @Override
    public int getOrder() {
        return 0;  // executa antes dos outros advisors
    }

    @Override
    public AdvisedRequest adviseRequest(AdvisedRequest request, AdvisorChain chain) {
        String userText = request.userText().toLowerCase();
        boolean contemPalavraProibida = PALAVRAS_PROIBIDAS.stream()
            .anyMatch(userText::contains);

        if (contemPalavraProibida) {
            throw new IllegalArgumentException("Solicitação não permitida pela política de uso.");
        }

        return chain.nextAroundRequest(request);
    }
}
```

#### LoggingAdvisor — auditoria

```java
package br.com.exemplo.ai.advisor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ai.chat.client.AdvisedRequest;
import org.springframework.ai.chat.client.advisor.api.Advisor;
import org.springframework.ai.chat.client.advisor.api.AdvisorChain;
import org.springframework.ai.chat.model.ChatResponse;

public class AuditoriaAdvisor implements Advisor {

    private static final Logger log = LoggerFactory.getLogger(AuditoriaAdvisor.class);

    @Override
    public String getName() {
        return "AuditoriaAdvisor";
    }

    @Override
    public int getOrder() {
        return 100;
    }

    @Override
    public AdvisedRequest adviseRequest(AdvisedRequest request, AdvisorChain chain) {
        log.info("[AI-AUDITORIA] Pergunta: {}", request.userText());
        AdvisedRequest resultado = chain.nextAroundRequest(request);
        return resultado;
    }
}
```

#### Registrando advisors customizados

```java
this.chatClient = builder
    .defaultAdvisors(
        new ConteudoSafeGuardAdvisor(),
        new AuditoriaAdvisor(),
        new MessageChatMemoryAdvisor(chatMemory),
        new QuestionAnswerAdvisor(vectorStore, SearchRequest.defaults())
    )
    .build();
```

---

### 2.12 Modelos de Imagem e Áudio

#### Geração de imagens (DALL-E)

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
```

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.image.ImageModel;
import org.springframework.ai.image.ImageOptions;
import org.springframework.ai.image.ImagePrompt;
import org.springframework.ai.image.ImageResponse;
import org.springframework.ai.openai.OpenAiImageOptions;
import org.springframework.stereotype.Service;

@Service
public class ImagemService {

    private final ImageModel imageModel;

    public ImagemService(ImageModel imageModel) {
        this.imageModel = imageModel;
    }

    public String gerarImagem(String descricao) {
        ImageOptions opcoes = OpenAiImageOptions.builder()
            .withModel("dall-e-3")
            .withQuality("hd")
            .withHeight(1024)
            .withWidth(1024)
            .withN(1)
            .build();

        ImageResponse response = imageModel.call(new ImagePrompt(descricao, opcoes));
        return response.getResult().getOutput().getUrl();
    }
}
```

#### Transcrição de áudio (Whisper)

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.audio.transcription.AudioTranscriptionModel;
import org.springframework.ai.audio.transcription.AudioTranscriptionPrompt;
import org.springframework.ai.openai.OpenAiAudioTranscriptionOptions;
import org.springframework.core.io.FileSystemResource;
import org.springframework.stereotype.Service;

@Service
public class TranscricaoService {

    private final AudioTranscriptionModel transcriptionModel;

    public TranscricaoService(AudioTranscriptionModel transcriptionModel) {
        this.transcriptionModel = transcriptionModel;
    }

    public String transcrever(String caminhoAudio) {
        var opcoes = OpenAiAudioTranscriptionOptions.builder()
            .withLanguage("pt")
            .withTemperature(0.0f)
            .build();

        var prompt = new AudioTranscriptionPrompt(
            new FileSystemResource(caminhoAudio),
            opcoes
        );

        return transcriptionModel.call(prompt).getResult().getOutput();
    }
}
```

---

### 2.13 Observabilidade

O Spring AI integra-se com Micrometer para métricas e rastreamento distribuído.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, metrics, prometheus
  metrics:
    tags:
      application: minha-app-ai

spring:
  ai:
    chat:
      observations:
        include-prompt: true      # inclui prompt nas métricas (cuidado com dados sensíveis)
        include-completion: false  # não inclui resposta completa nas métricas
```

As métricas disponíveis incluem:
- `gen_ai.client.token.usage` — tokens consumidos (prompt, completion, total)
- `gen_ai.client.operation.duration` — latência das chamadas ao modelo
- `spring.ai.vectorstore.add` — documentos indexados no VectorStore
- `spring.ai.vectorstore.similarity.search` — buscas realizadas

---

## 3. LangChain4J

### 3.1 Visão Geral

LangChain4J é o port Java do framework LangChain (Python), com foco em:

- **AI Services**: interfaces declarativas que o framework implementa automaticamente
- **Document Loading e Splitting**: carregamento de PDF, TXT, HTML, código-fonte
- **Embedding Stores**: integração com Chroma, Pinecone, Weaviate, pgvector e outros
- **Memory**: memória de curto e longo prazo para agentes
- **Tools**: chamada de funções Java pelo modelo
- **Chains**: composição de pipelines de processamento

**Versão de referência:** LangChain4J 0.36+  
**Site oficial:** [langchain4j.github.io](https://langchain4j.github.io/langchain4j/)

---

### 3.2 Dependências e Configuração

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j-bom</artifactId>
            <version>0.36.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Starter do Spring Boot -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-spring-boot-starter</artifactId>
    </dependency>

    <!-- Provedor OpenAI -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
    </dependency>

    <!-- Alternativa: Anthropic Claude -->
    <!-- <artifactId>langchain4j-anthropic-spring-boot-starter</artifactId> -->

    <!-- Alternativa: Ollama (local) -->
    <!-- <artifactId>langchain4j-ollama-spring-boot-starter</artifactId> -->

    <!-- Alternativa: Google Gemini -->
    <!-- <artifactId>langchain4j-google-ai-gemini-spring-boot-starter</artifactId> -->

    <!-- RAG com Chroma -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-chroma</artifactId>
    </dependency>

    <!-- Leitura de PDF -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-document-parser-apache-pdfbox</artifactId>
    </dependency>
</dependencies>
```

```yaml
langchain4j:
  open-ai:
    chat-model:
      api-key: ${OPENAI_API_KEY}
      model-name: gpt-4o-mini
      temperature: 0.7
      max-tokens: 2048
      log-requests: true
      log-responses: true
    embedding-model:
      api-key: ${OPENAI_API_KEY}
      model-name: text-embedding-3-small
```

---

### 3.3 AI Services (Interface Declarativa)

O maior diferencial do LangChain4J é a criação de serviços de IA como interfaces Java, implementadas automaticamente pelo framework via proxy dinâmico.

#### Interface simples

```java
package br.com.exemplo.ai.service;

import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import dev.langchain4j.service.spring.AiService;

@AiService
public interface AssistenteService {

    @SystemMessage("Você é um assistente de atendimento ao cliente da empresa XYZ.")
    String responder(String pergunta);

    @SystemMessage("Você é um especialista em resumo de textos. Seja conciso.")
    @UserMessage("Resuma o seguinte texto em português, em no máximo 3 frases: {{it}}")
    String resumir(String texto);
}
```

#### Interface com anotações avançadas

```java
package br.com.exemplo.ai.service;

import dev.langchain4j.model.output.structured.Description;
import dev.langchain4j.service.*;
import dev.langchain4j.service.spring.AiService;

import java.util.List;

@AiService
public interface AnalisadorService {

    record SentimentoAnalise(
        @Description("Sentimento predominante: POSITIVO, NEGATIVO ou NEUTRO")
        String sentimento,
        @Description("Pontuação de confiança de 0.0 a 1.0")
        double confianca,
        @Description("Lista de palavras-chave detectadas")
        List<String> palavrasChave
    ) {}

    @SystemMessage("Você é um especialista em análise de sentimento de textos em português.")
    SentimentoAnalise analisarSentimento(@UserMessage String texto);

    @SystemMessage("Você é um tradutor profissional.")
    @UserMessage("Traduza o seguinte texto para {{idioma}}: {{texto}}")
    String traduzir(@V("texto") String texto, @V("idioma") String idioma);
}
```

#### Streaming

```java
@AiService
public interface ChatStreamService {

    @SystemMessage("Você é um assistente que responde de forma detalhada.")
    TokenStream responderStream(String pergunta);
}
```

```java
// Uso no controller
@GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> stream(@RequestParam String pergunta) {
    return Flux.create(sink ->
        chatStreamService.responderStream(pergunta)
            .onNext(sink::next)
            .onComplete(c -> sink.complete())
            .onError(sink::error)
            .start()
    );
}
```

---

### 3.4 Chat Memory

```java
package br.com.exemplo.ai.config;

import dev.langchain4j.memory.chat.MessageWindowChatMemory;
import dev.langchain4j.service.spring.AiService;
import dev.langchain4j.service.MemoryId;
import dev.langchain4j.service.UserMessage;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ChatMemoryConfig {

    @Bean
    ChatMemoryProvider chatMemoryProvider() {
        return memoryId -> MessageWindowChatMemory.builder()
            .id(memoryId)
            .maxMessages(20)
            .build();
    }
}
```

```java
@AiService
public interface AssistenteComMemoria {

    @SystemMessage("Você é um assistente com memória da conversa.")
    String conversar(@MemoryId String sessaoId, @UserMessage String mensagem);
}
```

#### Memória de longo prazo com banco de dados

```java
package br.com.exemplo.ai.memory;

import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.store.memory.chat.ChatMemoryStore;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

// Implementação customizada (pode persistir em banco JPA)
@Component
public class JpaChatMemoryStore implements ChatMemoryStore {

    private final ChatMemoryRepository repository;  // repositório JPA customizado

    public JpaChatMemoryStore(ChatMemoryRepository repository) {
        this.repository = repository;
    }

    @Override
    public List<ChatMessage> getMessages(Object memoryId) {
        return repository.findBySessionId(memoryId.toString()).stream()
            .map(entity -> entity.toChatMessage())
            .toList();
    }

    @Override
    public void updateMessages(Object memoryId, List<ChatMessage> messages) {
        repository.deleteBySessionId(memoryId.toString());
        messages.forEach(msg ->
            repository.save(ChatMemoryEntity.fromChatMessage(memoryId.toString(), msg))
        );
    }

    @Override
    public void deleteMessages(Object memoryId) {
        repository.deleteBySessionId(memoryId.toString());
    }
}
```

---

### 3.5 Tools (Ferramentas / Function Calling)

```java
package br.com.exemplo.ai.tools;

import dev.langchain4j.agent.tool.Tool;
import dev.langchain4j.agent.tool.P;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

@Component
public class CalculadoraTools {

    @Tool("Calcula a diferença em dias entre duas datas no formato dd/MM/yyyy")
    public long calcularDiferencaDias(
        @P("Data inicial no formato dd/MM/yyyy") String dataInicial,
        @P("Data final no formato dd/MM/yyyy") String dataFinal
    ) {
        DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
        LocalDate inicio = LocalDate.parse(dataInicial, fmt);
        LocalDate fim = LocalDate.parse(dataFinal, fmt);
        return java.time.temporal.ChronoUnit.DAYS.between(inicio, fim);
    }

    @Tool("Realiza operações matemáticas básicas (soma, subtração, multiplicação, divisão)")
    public double calcular(
        @P("Primeiro número") double a,
        @P("Operação: soma, subtracao, multiplicacao, divisao") String operacao,
        @P("Segundo número") double b
    ) {
        return switch (operacao.toLowerCase()) {
            case "soma" -> a + b;
            case "subtracao" -> a - b;
            case "multiplicacao" -> a * b;
            case "divisao" -> b != 0 ? a / b : Double.NaN;
            default -> throw new IllegalArgumentException("Operação desconhecida: " + operacao);
        };
    }
}
```

#### Registrando tools no AI Service

```java
package br.com.exemplo.ai.config;

import dev.langchain4j.memory.chat.MessageWindowChatMemory;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AssistenteConfig {

    @Bean
    AssistenteMatematico assistenteMatematico(
        ChatLanguageModel chatModel,
        CalculadoraTools calculadoraTools
    ) {
        return AiServices.builder(AssistenteMatematico.class)
            .chatLanguageModel(chatModel)
            .tools(calculadoraTools)
            .chatMemory(MessageWindowChatMemory.withMaxMessages(10))
            .build();
    }
}
```

---

### 3.6 Document Loaders e RAG

```java
package br.com.exemplo.ai.service;

import dev.langchain4j.data.document.Document;
import dev.langchain4j.data.document.DocumentParser;
import dev.langchain4j.data.document.DocumentSplitter;
import dev.langchain4j.data.document.loader.FileSystemDocumentLoader;
import dev.langchain4j.data.document.parser.apache.pdfbox.ApachePdfBoxDocumentParser;
import dev.langchain4j.data.document.splitter.DocumentSplitters;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.store.embedding.EmbeddingStore;
import dev.langchain4j.store.embedding.EmbeddingStoreIngestor;
import org.springframework.stereotype.Service;

import java.nio.file.Path;
import java.util.List;

@Service
public class IndexacaoService {

    private final EmbeddingModel embeddingModel;
    private final EmbeddingStore<TextSegment> embeddingStore;

    public IndexacaoService(EmbeddingModel embeddingModel,
                             EmbeddingStore<TextSegment> embeddingStore) {
        this.embeddingModel = embeddingModel;
        this.embeddingStore = embeddingStore;
    }

    public void indexarPdf(Path arquivo) {
        DocumentParser parser = new ApachePdfBoxDocumentParser();
        Document documento = FileSystemDocumentLoader.loadDocument(arquivo, parser);

        DocumentSplitter splitter = DocumentSplitters.recursive(500, 50);

        EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
            .documentSplitter(splitter)
            .embeddingModel(embeddingModel)
            .embeddingStore(embeddingStore)
            .build();

        ingestor.ingest(documento);
    }

    public void indexarDiretorio(Path diretorio) {
        List<Document> documentos = FileSystemDocumentLoader.loadDocumentsRecursively(
            diretorio,
            new ApachePdfBoxDocumentParser()
        );

        EmbeddingStoreIngestor.builder()
            .documentSplitter(DocumentSplitters.recursive(800, 100))
            .embeddingModel(embeddingModel)
            .embeddingStore(embeddingStore)
            .build()
            .ingest(documentos);
    }
}
```

#### RAG com ContentRetriever

```java
package br.com.exemplo.ai.config;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.rag.content.retriever.ContentRetriever;
import dev.langchain4j.rag.content.retriever.EmbeddingStoreContentRetriever;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.store.embedding.EmbeddingStore;
import dev.langchain4j.data.segment.TextSegment;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RagConfig {

    @Bean
    ContentRetriever contentRetriever(EmbeddingStore<TextSegment> embeddingStore,
                                       EmbeddingModel embeddingModel) {
        return EmbeddingStoreContentRetriever.builder()
            .embeddingStore(embeddingStore)
            .embeddingModel(embeddingModel)
            .maxResults(5)
            .minScore(0.6)
            .build();
    }

    @Bean
    AssistenteDocumentos assistenteDocumentos(ChatLanguageModel chatModel,
                                               ContentRetriever contentRetriever) {
        return AiServices.builder(AssistenteDocumentos.class)
            .chatLanguageModel(chatModel)
            .contentRetriever(contentRetriever)
            .build();
    }
}
```

```java
@AiService
public interface AssistenteDocumentos {

    @SystemMessage("""
        Você é um assistente especializado. Responda apenas com base nos documentos fornecidos.
        Se não encontrar a informação, diga que não sabe.
        """)
    String responder(String pergunta);
}
```

---

### 3.7 Embedding Stores

#### In-Memory (testes e protótipos)

```java
@Bean
EmbeddingStore<TextSegment> embeddingStore() {
    return new InMemoryEmbeddingStore<>();
}
```

#### Chroma

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-chroma</artifactId>
</dependency>
```

```java
@Bean
EmbeddingStore<TextSegment> chromaEmbeddingStore() {
    return ChromaEmbeddingStore.builder()
        .baseUrl("http://localhost:8000")
        .collectionName("documentos")
        .build();
}
```

#### PgVector

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-pgvector</artifactId>
</dependency>
```

```java
@Bean
EmbeddingStore<TextSegment> pgVectorEmbeddingStore(DataSource dataSource) {
    return PgVectorEmbeddingStore.builder()
        .datasource(dataSource)
        .table("embeddings")
        .dimension(1536)
        .createTable(true)
        .build();
}
```

#### Pinecone

```java
@Bean
EmbeddingStore<TextSegment> pineconeStore() {
    return PineconeEmbeddingStore.builder()
        .apiKey(System.getenv("PINECONE_API_KEY"))
        .index("meu-indice")
        .nameSpace("producao")
        .dimension(1536)
        .build();
}
```

---

### 3.8 Agentes e Chains

```java
package br.com.exemplo.ai.service;

import dev.langchain4j.chain.ConversationalRetrievalChain;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.rag.content.retriever.ContentRetriever;
import org.springframework.stereotype.Service;

@Service
public class AgentService {

    private final ConversationalRetrievalChain chain;

    public AgentService(ChatLanguageModel chatModel, ContentRetriever contentRetriever) {
        this.chain = ConversationalRetrievalChain.builder()
            .chatLanguageModel(chatModel)
            .contentRetriever(contentRetriever)
            .build();
    }

    public String perguntar(String pergunta) {
        return chain.execute(pergunta);
    }
}
```

---

## 4. Semantic Kernel for Java

Semantic Kernel é o framework de IA da Microsoft, com suporte oficial para Java desde 2024. Destaca-se pela abordagem orientada a **plugins** e **planos orquestrados**.

### 4.1 Dependências

```xml
<dependency>
    <groupId>com.microsoft.semantic-kernel</groupId>
    <artifactId>semantickernel-api</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>com.microsoft.semantic-kernel</groupId>
    <artifactId>semantickernel-connectors-ai-openai</artifactId>
    <version>1.3.0</version>
</dependency>
```

### 4.2 Configuração e uso básico

```java
package br.com.exemplo.ai.semantickernel;

import com.microsoft.semantickernel.Kernel;
import com.microsoft.semantickernel.aiservices.openai.chatcompletion.OpenAIChatCompletion;
import com.microsoft.semantickernel.orchestration.InvocationContext;
import com.microsoft.semantickernel.semanticfunctions.KernelFunctionArguments;
import com.microsoft.semantickernel.services.chatcompletion.ChatCompletionService;
import com.openai.client.OpenAIClient;
import com.openai.client.okhttp.OpenAIOkHttpClient;

public class SemanticKernelExample {

    public static void main(String[] args) {
        OpenAIClient openAIClient = OpenAIOkHttpClient.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .build();

        ChatCompletionService chatService = OpenAIChatCompletion.builder()
            .withOpenAIAsyncClient(openAIClient)
            .withModelId("gpt-4o")
            .build();

        Kernel kernel = Kernel.builder()
            .withAIService(ChatCompletionService.class, chatService)
            .build();

        var function = kernel.createFunctionFromPrompt(
            "Explique {{$topico}} de forma simples para um iniciante."
        );

        var result = kernel.invokeAsync(function)
            .withArguments(
                KernelFunctionArguments.builder()
                    .withVariable("topico", "programação orientada a objetos")
                    .build()
            )
            .block();

        System.out.println(result.getResult());
    }
}
```

### 4.3 Plugins nativos (Java functions)

```java
package br.com.exemplo.ai.semantickernel;

import com.microsoft.semantickernel.semanticfunctions.annotations.DefineKernelFunction;
import com.microsoft.semantickernel.semanticfunctions.annotations.KernelFunctionParameter;

public class MathPlugin {

    @DefineKernelFunction(name = "soma", description = "Soma dois números inteiros")
    public int soma(
        @KernelFunctionParameter(name = "a", description = "Primeiro número") int a,
        @KernelFunctionParameter(name = "b", description = "Segundo número") int b
    ) {
        return a + b;
    }
}
```

```java
Kernel kernel = Kernel.builder()
    .withAIService(ChatCompletionService.class, chatService)
    .withPlugin(KernelPluginFactory.createFromObject(new MathPlugin(), "Math"))
    .build();
```

---

## 5. SDKs Nativos dos Provedores

### 5.1 OpenAI Java SDK

O SDK oficial da OpenAI para Java, sem dependência de frameworks adicionais.

```xml
<dependency>
    <groupId>com.openai</groupId>
    <artifactId>openai-java</artifactId>
    <version>2.5.0</version>
</dependency>
```

```java
package br.com.exemplo.ai.openai;

import com.openai.client.OpenAIClient;
import com.openai.client.okhttp.OpenAIOkHttpClient;
import com.openai.models.ChatCompletion;
import com.openai.models.ChatCompletionCreateParams;
import com.openai.models.ChatCompletionMessageParam;
import com.openai.models.ChatModel;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class OpenAINativeService {

    private final OpenAIClient client;

    public OpenAINativeService() {
        this.client = OpenAIOkHttpClient.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .build();
    }

    public String chat(String systemPrompt, String userMessage) {
        ChatCompletion completion = client.chat().completions().create(
            ChatCompletionCreateParams.builder()
                .model(ChatModel.GPT_4O)
                .maxCompletionTokens(2048)
                .messages(List.of(
                    ChatCompletionMessageParam.ofSystem(
                        ChatCompletionSystemMessageParam.builder()
                            .content(systemPrompt)
                            .build()
                    ),
                    ChatCompletionMessageParam.ofUser(
                        ChatCompletionUserMessageParam.builder()
                            .content(userMessage)
                            .build()
                    )
                ))
                .build()
        );

        return completion.choices().get(0).message().content().orElse("");
    }

    // Geração de embeddings
    public List<Float> embedding(String texto) {
        var response = client.embeddings().create(
            EmbeddingCreateParams.builder()
                .model(EmbeddingModel.TEXT_EMBEDDING_3_SMALL)
                .input(texto)
                .build()
        );
        return response.data().get(0).embedding();
    }
}
```

---

### 5.2 Anthropic Java SDK

```xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>sdk</artifactId>
    <version>1.3.0</version>
</dependency>
```

```java
package br.com.exemplo.ai.anthropic;

import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import com.anthropic.models.Message;
import com.anthropic.models.MessageCreateParams;
import com.anthropic.models.Model;
import org.springframework.stereotype.Service;

@Service
public class AnthropicNativeService {

    private final AnthropicClient client;

    public AnthropicNativeService() {
        this.client = AnthropicOkHttpClient.builder()
            .apiKey(System.getenv("ANTHROPIC_API_KEY"))
            .build();
    }

    public String chat(String systemPrompt, String userMessage) {
        Message message = client.messages().create(
            MessageCreateParams.builder()
                .model(Model.CLAUDE_SONNET_4_6)
                .maxTokens(4096)
                .system(systemPrompt)
                .addUserMessage(userMessage)
                .build()
        );

        return message.content().get(0).text().text();
    }

    // Streaming
    public void chatStream(String pergunta) {
        try (var stream = client.messages().stream(
            MessageCreateParams.builder()
                .model(Model.CLAUDE_SONNET_4_6)
                .maxTokens(4096)
                .addUserMessage(pergunta)
                .build()
        )) {
            stream.on("text", text -> System.out.print(text));
        }
    }
}
```

---

### 5.3 Google Gemini (Vertex AI / Generative AI)

```xml
<!-- Google AI (para acesso direto via API key) -->
<dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-vertexai</artifactId>
    <version>1.12.0</version>
</dependency>
```

```java
package br.com.exemplo.ai.google;

import com.google.cloud.vertexai.VertexAI;
import com.google.cloud.vertexai.api.GenerateContentResponse;
import com.google.cloud.vertexai.generativeai.GenerativeModel;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Service
public class GeminiService {

    private final GenerativeModel model;

    public GeminiService() throws IOException {
        VertexAI vertexAI = new VertexAI("meu-projeto-gcp", "us-central1");
        this.model = new GenerativeModel("gemini-2.0-flash", vertexAI);
    }

    public String gerarConteudo(String prompt) throws IOException {
        GenerateContentResponse response = model.generateContent(prompt);
        return response.getCandidates(0)
            .getContent()
            .getParts(0)
            .getText();
    }
}
```

---

## 6. Ollama — Modelos Locais

Ollama permite executar modelos open-source localmente (Llama, Mistral, Phi, Gemma, etc.) sem custo de API e com total privacidade dos dados.

### 6.1 Instalação e modelos

```bash
# Instalar Ollama (Linux/Mac)
curl -fsSL https://ollama.ai/install.sh | sh

# Baixar e executar modelos
ollama pull llama3.2          # Meta Llama 3.2 (3B)
ollama pull llama3.2:11b      # Llama 3.2 (11B)
ollama pull mistral            # Mistral 7B
ollama pull phi4               # Microsoft Phi-4 (14B)
ollama pull gemma3:12b         # Google Gemma 3 (12B)
ollama pull nomic-embed-text   # Embedding model
ollama pull qwen2.5-coder      # Modelo especializado em código

# Executar via Docker
docker run -d -p 11434:11434 ollama/ollama
```

### 6.2 Integração com Spring AI

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: llama3.2
          temperature: 0.7
          num-ctx: 4096         # tamanho do contexto
      embedding:
        options:
          model: nomic-embed-text
```

```java
package br.com.exemplo.ai.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.ollama.OllamaChatModel;
import org.springframework.ai.ollama.api.OllamaOptions;
import org.springframework.stereotype.Service;

@Service
public class OllamaService {

    private final ChatClient chatClient;

    public OllamaService(OllamaChatModel ollamaChatModel) {
        this.chatClient = ChatClient.builder(ollamaChatModel).build();
    }

    public String gerarTexto(String prompt) {
        return chatClient.prompt()
            .user(prompt)
            .call()
            .content();
    }

    // Trocar modelo em tempo de execução
    public String gerarComModelo(String prompt, String modelo) {
        return chatClient.prompt()
            .user(prompt)
            .options(OllamaOptions.create().withModel(modelo))
            .call()
            .content();
    }
}
```

### 6.3 Integração com LangChain4J

```yaml
langchain4j:
  ollama:
    chat-model:
      base-url: http://localhost:11434
      model-name: llama3.2
      temperature: 0.7
    embedding-model:
      base-url: http://localhost:11434
      model-name: nomic-embed-text
```

---

## 7. Bancos de Dados Vetoriais

Bancos de dados vetoriais armazenam e indexam embeddings para busca semântica eficiente (similaridade por cosseno, produto interno, distância euclidiana).

### 7.1 pgvector (PostgreSQL)

Extensão do PostgreSQL para armazenamento e busca de vetores — ideal para quem já usa Postgres.

```sql
-- Habilitar extensão
CREATE EXTENSION IF NOT EXISTS vector;

-- Criar tabela de documentos
CREATE TABLE documentos (
    id          BIGSERIAL PRIMARY KEY,
    conteudo    TEXT NOT NULL,
    metadata    JSONB,
    embedding   vector(1536),  -- dimensão do modelo text-embedding-3-small
    criado_em   TIMESTAMPTZ DEFAULT NOW()
);

-- Índice para busca por cosseno (hnsw é o mais rápido para grandes volumes)
CREATE INDEX ON documentos USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

-- Busca dos 5 mais similares
SELECT id, conteudo, 1 - (embedding <=> '[...]'::vector) AS similaridade
FROM documentos
ORDER BY embedding <=> '[...]'::vector
LIMIT 5;
```

```java
// Configuração via Spring AI (auto-configurado)
// Configuração via LangChain4J
@Bean
EmbeddingStore<TextSegment> pgvectorStore(DataSource dataSource) {
    return PgVectorEmbeddingStore.builder()
        .datasource(dataSource)
        .table("documentos_lc4j")
        .dimension(1536)
        .distanceType(PgVectorEmbeddingStore.DistanceType.COSINE_DISTANCE)
        .indexType(PgVectorEmbeddingStore.IndexType.HNSW)
        .createTable(true)
        .build();
}
```

---

### 7.2 Redis Vector Search

Redis com o módulo `redis-search` (disponível no Redis Stack e Redis Cloud).

```bash
# Docker com Redis Stack (inclui RedisSearch e RedisJSON)
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
  ai:
    vectorstore:
      redis:
        index: minha-busca
        prefix: emb:
        initialize-schema: true
```

---

### 7.3 Chroma

Banco de dados vetorial open-source, fácil de usar localmente ou em nuvem.

```bash
# Executar com Docker
docker run -d -p 8000:8000 chromadb/chroma

# Com persistência
docker run -d -p 8000:8000 -v ./chroma-data:/chroma/chroma chromadb/chroma
```

```java
// Spring AI
@Bean
VectorStore chromaVectorStore(EmbeddingModel embeddingModel) {
    return new ChromaVectorStore(
        embeddingModel,
        "http://localhost:8000",
        "minha-colecao",
        true  // initialize
    );
}

// LangChain4J
@Bean
EmbeddingStore<TextSegment> chromaStore() {
    return ChromaEmbeddingStore.builder()
        .baseUrl("http://localhost:8000")
        .collectionName("documentos")
        .build();
}
```

---

### 7.4 Weaviate

Banco vetorial nativo com suporte a filtros complexos, hybrid search (BM25 + vetores) e importação de módulos de embedding.

```bash
docker run -d -p 8080:8080 -p 50051:50051 \
  -e QUERY_DEFAULTS_LIMIT=25 \
  -e AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED=true \
  -e PERSISTENCE_DATA_PATH=/var/lib/weaviate \
  cr.weaviate.io/semitechnologies/weaviate:1.27.0
```

```java
// LangChain4J
@Bean
EmbeddingStore<TextSegment> weaviateStore() {
    return WeaviateEmbeddingStore.builder()
        .scheme("http")
        .host("localhost:8080")
        .objectClass("Documento")
        .avoidDups(true)
        .build();
}
```

---

### 7.5 Pinecone

Banco vetorial gerenciado na nuvem, sem necessidade de infraestrutura própria.

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pinecone-store-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  ai:
    vectorstore:
      pinecone:
        api-key: ${PINECONE_API_KEY}
        index-name: meu-indice
        namespace: producao
```

---

## 8. Padrões de Arquitetura com IA

### 8.1 RAG Avançado

O RAG básico pode ser aprimorado com técnicas mais sofisticadas:

#### Hybrid Search (BM25 + Vetorial)

Combina busca por palavras-chave (BM25) com busca semântica para maior precisão.

```java
package br.com.exemplo.ai.rag;

import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Stream;

@Component
public class HybridSearchRetriever {

    private final VectorStore vectorStore;
    private final BM25Service bm25Service;  // implementação customizada

    public HybridSearchRetriever(VectorStore vectorStore, BM25Service bm25Service) {
        this.vectorStore = vectorStore;
        this.bm25Service = bm25Service;
    }

    public List<Document> buscar(String query, int topK) {
        // Busca vetorial
        List<Document> vetorial = vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(topK)
        );

        // Busca por palavras-chave
        List<Document> keywords = bm25Service.search(query, topK);

        // Combinar e deduplicar com Reciprocal Rank Fusion (RRF)
        return aplicarRRF(vetorial, keywords, topK);
    }

    private List<Document> aplicarRRF(List<Document> lista1, List<Document> lista2, int topK) {
        // Implementação simplificada de RRF
        // Em produção, usar biblioteca dedicada ou implementação completa
        return Stream.concat(lista1.stream(), lista2.stream())
            .distinct()
            .limit(topK)
            .toList();
    }
}
```

#### Reranking

```java
package br.com.exemplo.ai.rag;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.document.Document;
import org.springframework.stereotype.Component;

import java.util.Comparator;
import java.util.List;

@Component
public class RerankerService {

    private final ChatClient chatClient;

    public RerankerService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public List<Document> rerankar(String query, List<Document> documentos) {
        record DocComScore(Document doc, double score) {}

        return documentos.stream()
            .map(doc -> {
                String prompt = """
                    Query: %s
                    
                    Documento: %s
                    
                    Em uma escala de 0 a 10, qual a relevância do documento para a query?
                    Responda APENAS com o número.
                    """.formatted(query, doc.getContent());

                String resposta = chatClient.prompt().user(prompt).call().content();
                double score = Double.parseDouble(resposta.trim());
                return new DocComScore(doc, score);
            })
            .sorted(Comparator.comparingDouble(DocComScore::score).reversed())
            .map(DocComScore::doc)
            .toList();
    }
}
```

---

### 8.2 Agentic Workflows

Agentes tomam decisões autonomamente sobre quais ferramentas usar e em que ordem.

```java
package br.com.exemplo.ai.agent;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
public class PesquisaAgente {

    private final ChatClient chatClient;

    public PesquisaAgente(ChatClient.Builder builder,
                           WebSearchTool webSearchTool,
                           DocumentSearchTool documentSearchTool,
                           CalculadoraTool calculadoraTool) {
        this.chatClient = builder
            .defaultSystem("""
                Você é um agente de pesquisa. Você tem acesso a:
                1. Busca na web para informações atuais
                2. Busca em documentos internos da empresa
                3. Calculadora para operações matemáticas
                
                Use as ferramentas necessárias para responder com precisão.
                Quando não tiver certeza, pesquise primeiro antes de responder.
                """)
            .defaultTools(webSearchTool, documentSearchTool, calculadoraTool)
            .build();
    }

    public String pesquisar(String pergunta) {
        return chatClient.prompt()
            .user(pergunta)
            .call()
            .content();
    }
}
```

#### Workflow Multi-Agente com Spring Integration

```java
package br.com.exemplo.ai.workflow;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
public class WorkflowMultiAgente {

    private final ChatClient extrator;
    private final ChatClient analisador;
    private final ChatClient redator;

    public WorkflowMultiAgente(ChatClient.Builder builder) {
        this.extrator = builder
            .defaultSystem("Você é especialista em extração de informações estruturadas de textos.")
            .build();

        this.analisador = builder
            .defaultSystem("Você é especialista em análise crítica e identificação de padrões.")
            .build();

        this.redator = builder
            .defaultSystem("Você é um redator técnico que transforma análises em relatórios claros.")
            .build();
    }

    public String processarDocumento(String textoOriginal) {
        // Etapa 1: Extrair informações estruturadas
        String dadosExtraidos = extrator.prompt()
            .user("Extraia os dados principais do seguinte texto: " + textoOriginal)
            .call()
            .content();

        // Etapa 2: Analisar os dados extraídos
        String analise = analisador.prompt()
            .user("Analise criticamente os seguintes dados: " + dadosExtraidos)
            .call()
            .content();

        // Etapa 3: Gerar relatório final
        return redator.prompt()
            .user("Crie um relatório executivo com base na análise: " + analise)
            .call()
            .content();
    }
}
```

---

### 8.3 Structured Output e Domain Mapping

```java
package br.com.exemplo.ai.service;

import br.com.exemplo.pedido.Pedido;
import br.com.exemplo.pedido.PedidoRepository;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class PedidoAIService {

    private final ChatClient chatClient;
    private final PedidoRepository pedidoRepository;

    public PedidoAIService(ChatClient chatClient, PedidoRepository pedidoRepository) {
        this.chatClient = chatClient;
        this.pedidoRepository = pedidoRepository;
    }

    record PedidoExtraido(
        String nomeCliente,
        String emailCliente,
        List<ItemPedido> itens,
        String enderecoEntrega
    ) {}

    record ItemPedido(
        String produto,
        int quantidade,
        String observacoes
    ) {}

    // Extrai pedido de texto livre (e.g. e-mail de cliente) e persiste no banco
    public Pedido processarPedidoTextoLivre(String textoEmail) {
        PedidoExtraido extraido = chatClient.prompt()
            .system("""
                Extraia as informações de pedido do texto fornecido.
                Se algum campo não estiver presente, use null.
                """)
            .user(textoEmail)
            .call()
            .entity(PedidoExtraido.class);

        Pedido pedido = new Pedido();
        pedido.setNomeCliente(extraido.nomeCliente());
        pedido.setEmailCliente(extraido.emailCliente());
        pedido.setEnderecoEntrega(extraido.enderecoEntrega());
        // ... mapear itens

        return pedidoRepository.save(pedido);
    }
}
```

---

## 9. Spring AI vs LangChain4J — Comparativo

| Critério | Spring AI | LangChain4J |
|---|---|---|
| **Integração Spring Boot** | Nativa, auto-configuração completa | Boa, via starters dedicados |
| **Curva de aprendizado** | Baixa para dev Spring | Moderada |
| **AI Services declarativos** | Não tem equivalente direto | `@AiService` + interfaces Java |
| **RAG** | `QuestionAnswerAdvisor` + `VectorStore` | `ContentRetriever` + `EmbeddingStore` |
| **Tool Calling** | `@Tool` em Spring beans | `@Tool` em qualquer objeto |
| **Memória** | `ChatMemory` com advisor | `ChatMemory` no AI Service |
| **Provedores suportados** | OpenAI, Anthropic, Gemini, Azure, Ollama, Mistral, Amazon Bedrock e outros | OpenAI, Anthropic, Gemini, Ollama, Mistral, Cohere e outros |
| **Streaming** | `Flux<String>` reativo | `TokenStream` |
| **Observabilidade** | Micrometer nativo | Logs básicos |
| **Vector Stores** | PgVector, Redis, Chroma, Pinecone, Weaviate, MongoDB Atlas e outros | PgVector, Chroma, Pinecone, Weaviate, Redis e outros |
| **Comunidade** | Ecossistema Spring | Comunidade própria crescente |
| **Maturidade** | 1.0 GA (2025) | 0.36+ (maduro para produção) |

**Recomendação:**
- Use **Spring AI** quando já usa Spring Boot e prefere integração nativa com o ecossistema
- Use **LangChain4J** quando precisa de `@AiService` declarativo, pipelines mais complexos ou quer um framework agnóstico ao Spring
- Os dois podem coexistir no mesmo projeto

---

## 10. Boas Práticas

### 10.1 Gerenciamento de custos

```yaml
# Limitar tokens para controlar custos
spring:
  ai:
    openai:
      chat:
        options:
          max-tokens: 1024      # limite conservador
          temperature: 0.3      # menos criatividade = menos tokens gastos em respostas longas
```

```java
// Usar modelos mais baratos para tarefas simples
// gpt-4o-mini para classificação, gpt-4o para raciocínio complexo
public String classificar(String texto) {
    return ChatClient.builder(chatModel)
        .build()
        .prompt()
        .options(OpenAiChatOptions.builder()
            .withModel("gpt-4o-mini")  // ~60x mais barato que gpt-4o
            .withMaxTokens(50)
            .build())
        .user("Classifique em POSITIVO/NEGATIVO/NEUTRO: " + texto)
        .call()
        .content();
}
```

### 10.2 Segurança e validação

```java
@Service
public class AIGatewayService {

    private static final int MAX_INPUT_LENGTH = 10_000;

    public String processar(String input, String userId) {
        // 1. Validar tamanho da entrada
        if (input == null || input.length() > MAX_INPUT_LENGTH) {
            throw new IllegalArgumentException("Entrada inválida ou muito longa");
        }

        // 2. Sanitizar entrada (remover caracteres de controle)
        String sanitized = input.replaceAll("[\\p{Cntrl}&&[^\n\r\t]]", "");

        // 3. Registrar para auditoria
        log.info("AI request - userId={} inputLength={}", userId, sanitized.length());

        // 4. Chamar o modelo
        return chatClient.prompt()
            .user(sanitized)
            .call()
            .content();
    }
}
```

### 10.3 Cache de respostas

```java
package br.com.exemplo.ai.service;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class CachedAIService {

    private final ChatClient chatClient;

    public CachedAIService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    // Respostas determinísticas (temperature=0) podem ser cacheadas
    @Cacheable(value = "ai-respostas", key = "#pergunta")
    public String responderComCache(String pergunta) {
        return chatClient.prompt()
            .user(pergunta)
            .options(OpenAiChatOptions.builder().withTemperature(0.0f).build())
            .call()
            .content();
    }
}
```

```yaml
spring:
  cache:
    type: redis
  data:
    redis:
      host: localhost
      port: 6379
```

### 10.4 Testes

```java
package br.com.exemplo.ai.service;

import org.junit.jupiter.api.Test;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

@SpringBootTest
class ChatServiceTest {

    @MockBean
    ChatClient.Builder chatClientBuilder;

    @Test
    void deveResponderPergunta() {
        // Arrange
        ChatClient mockClient = mock(ChatClient.class);
        when(chatClientBuilder.build()).thenReturn(mockClient);

        // ... setup do mock com resposta específica

        // Assert que a resposta foi processada corretamente
        // ...
    }
}
```

#### Testes de integração com Testcontainers + Ollama

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>ollama</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@Testcontainers
class RagServiceIntegrationTest {

    @Container
    static OllamaContainer ollama = new OllamaContainer("ollama/ollama:latest")
        .withModel("llama3.2");

    @DynamicPropertySource
    static void ollamaProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.ai.ollama.base-url", ollama::getEndpoint);
        registry.add("spring.ai.ollama.chat.options.model", () -> "llama3.2");
    }

    @Autowired
    RagService ragService;

    @Test
    void deveResponderComContexto() {
        ragService.indexarTexto("A capital do Brasil é Brasília.");
        String resposta = ragService.perguntarComContexto("Qual é a capital do Brasil?");
        assertThat(resposta).containsIgnoringCase("Brasília");
    }
}
```

### 10.5 Chunking estratégico para RAG

```java
// Configurações recomendadas de chunking por tipo de conteúdo
DocumentSplitter splitterDocumentosLongos = DocumentSplitters.recursive(
    1000,  // tamanho do chunk em tokens
    200    // overlap (sobreposição entre chunks)
);

DocumentSplitter splitterFAQ = DocumentSplitters.recursive(
    300,   // chunks menores para FAQ (cada Q&A é autossuficiente)
    50
);

DocumentSplitter splitterCodigo = new CodeSplitter(
    Language.JAVA,
    30     // linhas por chunk
);
```

### 10.6 Monitoramento em produção

```java
package br.com.exemplo.ai.advisor;

import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.ai.chat.client.AdvisedRequest;
import org.springframework.ai.chat.client.advisor.api.Advisor;
import org.springframework.ai.chat.client.advisor.api.AdvisorChain;

public class MetricasAdvisor implements Advisor {

    private final MeterRegistry meterRegistry;

    public MetricasAdvisor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Override
    public String getName() { return "MetricasAdvisor"; }

    @Override
    public int getOrder() { return Integer.MAX_VALUE; }

    @Override
    public AdvisedRequest adviseRequest(AdvisedRequest request, AdvisorChain chain) {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            AdvisedRequest resultado = chain.nextAroundRequest(request);
            sample.stop(meterRegistry.timer("ai.chamadas", "status", "sucesso"));
            return resultado;
        } catch (Exception e) {
            sample.stop(meterRegistry.timer("ai.chamadas", "status", "erro"));
            meterRegistry.counter("ai.erros", "tipo", e.getClass().getSimpleName()).increment();
            throw e;
        }
    }
}
```

---

## Referências

- [Spring AI — Documentação Oficial](https://docs.spring.io/spring-ai/reference/)
- [LangChain4J — Documentação Oficial](https://docs.langchain4j.dev/)
- [Semantic Kernel for Java — GitHub](https://github.com/microsoft/semantic-kernel/tree/main/java)
- [OpenAI Java SDK — GitHub](https://github.com/openai/openai-java)
- [Anthropic Java SDK — GitHub](https://github.com/anthropics/anthropic-sdk-java)
- [Ollama — Site Oficial](https://ollama.ai/)
- [pgvector — GitHub](https://github.com/pgvector/pgvector)
- [Chroma — Site Oficial](https://www.trychroma.com/)
- [Weaviate — Site Oficial](https://weaviate.io/)
- [Pinecone — Site Oficial](https://www.pinecone.io/)
