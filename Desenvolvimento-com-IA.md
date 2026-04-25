# Desenvolvimento de Software com Ferramentas de IA

> **Objetivo:** Apresentar boas práticas, técnicas, configurações e exemplos de prompts para o desenvolvimento de software assistido por IA, cobrindo Claude Code, Google Gemini e OpenAI Codex/ChatGPT, além de explicar conceitos como MCP, vibe coding, spec-driven development e harness engineering.

---

## Sumário

1. [Introdução ao Desenvolvimento Assistido por IA](#1-introdução-ao-desenvolvimento-assistido-por-ia)
2. [Terminologia e Conceitos-Chave](#2-terminologia-e-conceitos-chave)
   - [2.1 Vibe Coding](#21-vibe-coding)
   - [2.2 Spec-Driven Development (SDD)](#22-spec-driven-development-sdd)
   - [2.3 Harness Engineering](#23-harness-engineering)
   - [2.4 Prompt Engineering](#24-prompt-engineering)
   - [2.5 MCP — Model Context Protocol](#25-mcp--model-context-protocol)
   - [2.6 Agentes de IA (Agentic Coding)](#26-agentes-de-ia-agentic-coding)
   - [2.7 Context Window e Gerenciamento de Contexto](#27-context-window-e-gerenciamento-de-contexto)
   - [2.8 RAG — Retrieval-Augmented Generation](#28-rag--retrieval-augmented-generation)
3. [Claude Code](#3-claude-code)
   - [3.1 Visão Geral](#31-visão-geral)
   - [3.2 Instalação e Configuração](#32-instalação-e-configuração)
   - [3.3 O Arquivo CLAUDE.md](#33-o-arquivo-claudemd)
   - [3.4 Configurações (settings.json)](#34-configurações-settingsjson)
   - [3.5 Slash Commands](#35-slash-commands)
   - [3.6 Hooks](#36-hooks)
   - [3.7 MCP no Claude Code](#37-mcp-no-claude-code)
   - [3.8 Boas Práticas com Claude Code](#38-boas-práticas-com-claude-code)
   - [3.9 Exemplos de Prompts para Claude Code](#39-exemplos-de-prompts-para-claude-code)
4. [Google Gemini](#4-google-gemini)
   - [4.1 Visão Geral das Ferramentas Gemini](#41-visão-geral-das-ferramentas-gemini)
   - [4.2 Gemini CLI](#42-gemini-cli)
   - [4.3 Gemini Code Assist (IDE)](#43-gemini-code-assist-ide)
   - [4.4 O Arquivo GEMINI.md](#44-o-arquivo-geminimd)
   - [4.5 MCP no Gemini CLI](#45-mcp-no-gemini-cli)
   - [4.6 Boas Práticas com Gemini](#46-boas-práticas-com-gemini)
   - [4.7 Exemplos de Prompts para Gemini](#47-exemplos-de-prompts-para-gemini)
5. [OpenAI Codex e ChatGPT](#5-openai-codex-e-chatgpt)
   - [5.1 Visão Geral](#51-visão-geral)
   - [5.2 OpenAI Codex CLI](#52-openai-codex-cli)
   - [5.3 GitHub Copilot](#53-github-copilot)
   - [5.4 ChatGPT para Desenvolvimento](#54-chatgpt-para-desenvolvimento)
   - [5.5 Boas Práticas com OpenAI](#55-boas-práticas-com-openai)
   - [5.6 Exemplos de Prompts para OpenAI](#56-exemplos-de-prompts-para-openai)
6. [MCP — Model Context Protocol em Profundidade](#6-mcp--model-context-protocol-em-profundidade)
   - [6.1 Arquitetura MCP](#61-arquitetura-mcp)
   - [6.2 Servidores MCP Populares](#62-servidores-mcp-populares)
   - [6.3 Criando um Servidor MCP Customizado](#63-criando-um-servidor-mcp-customizado)
7. [Spec-Driven Development com IA](#7-spec-driven-development-com-ia)
   - [7.1 Motivação e Conceito](#71-motivação-e-conceito)
   - [7.2 Estrutura de uma Especificação](#72-estrutura-de-uma-especificação)
   - [7.3 Fluxo de Trabalho SDD Completo](#73-fluxo-de-trabalho-sdd-completo)
   - [7.4 Exemplos Práticos (Spring Boot / Java)](#74-exemplos-práticos-spring-boot--java)
8. [Vibe Coding — Uso e Riscos](#8-vibe-coding--uso-e-riscos)
   - [8.1 O que é Vibe Coding](#81-o-que-é-vibe-coding)
   - [8.2 Quando Usar e Quando Evitar](#82-quando-usar-e-quando-evitar)
   - [8.3 Riscos e Mitigação](#83-riscos-e-mitigação)
9. [Harness Engineering](#9-harness-engineering)
   - [9.1 Conceito](#91-conceito)
   - [9.2 Componentes do Harness](#92-componentes-do-harness)
   - [9.3 CLAUDE.md como Harness Central](#93-claudemd-como-harness-central)
   - [9.4 Configuração Avançada de Hooks](#94-configuração-avançada-de-hooks)
   - [9.5 Harness para Projetos Java/Spring Boot](#95-harness-para-projetos-javaspring-boot)
10. [Técnicas Avançadas de Prompting](#10-técnicas-avançadas-de-prompting)
    - [10.1 Princípios Gerais](#101-princípios-gerais)
    - [10.2 Chain of Thought (CoT)](#102-chain-of-thought-cot)
    - [10.3 Few-Shot Prompting](#103-few-shot-prompting)
    - [10.4 Prompts para Geração de Código](#104-prompts-para-geração-de-código)
    - [10.5 Prompts para Revisão e Refatoração](#105-prompts-para-revisão-e-refatoração)
    - [10.6 Prompts para Testes](#106-prompts-para-testes)
    - [10.7 Prompts para Documentação](#107-prompts-para-documentação)
11. [Fluxos de Trabalho para Projetos Reais](#11-fluxos-de-trabalho-para-projetos-reais)
    - [11.1 Iniciando um Projeto do Zero](#111-iniciando-um-projeto-do-zero)
    - [11.2 Code Review Assistido por IA](#112-code-review-assistido-por-ia)
    - [11.3 Debugging com IA](#113-debugging-com-ia)
    - [11.4 Refatoração Guiada por IA](#114-refatoração-guiada-por-ia)
12. [Boas Práticas de Segurança com IA](#12-boas-práticas-de-segurança-com-ia)
13. [Comparativo das Ferramentas](#13-comparativo-das-ferramentas)
14. [Checklist para Projetos com IA](#14-checklist-para-projetos-com-ia)
15. [IDEs com IA de Primeira Classe](#15-ides-com-ia-de-primeira-classe)
    - [15.1 Cursor](#151-cursor)
    - [15.2 Windsurf](#152-windsurf)
    - [15.3 GitHub Copilot Workspace](#153-github-copilot-workspace)
    - [15.4 Comparativo IDEs com IA](#154-comparativo-ides-com-ia)
16. [Gerenciamento de Custo e Uso de Tokens](#16-gerenciamento-de-custo-e-uso-de-tokens)
    - [16.1 Entendendo Tokens e Custo](#161-entendendo-tokens-e-custo)
    - [16.2 Prompt Caching](#162-prompt-caching)
    - [16.3 Escolha de Modelo por Tipo de Tarefa](#163-escolha-de-modelo-por-tipo-de-tarefa)
    - [16.4 Otimizando o CLAUDE.md para Caching](#164-otimizando-o-claudemd-para-caching)
17. [Workflows Multi-Agente](#17-workflows-multi-agente)
    - [17.1 Conceito e Motivação](#171-conceito-e-motivação)
    - [17.2 Padrão ReAct](#172-padrão-react)
    - [17.3 Padrão Plan-and-Execute](#173-padrão-plan-and-execute)
    - [17.4 Subagentes no Claude Code](#174-subagentes-no-claude-code)
    - [17.5 Orquestração com a API Anthropic](#175-orquestração-com-a-api-anthropic)
18. [IA para Modernização de Código Legado](#18-ia-para-modernização-de-código-legado)
    - [18.1 Estratégia de Abordagem](#181-estratégia-de-abordagem)
    - [18.2 Migrações Java (8 → 21)](#182-migrações-java-8--21)
    - [18.3 Migrações de Frameworks](#183-migrações-de-frameworks)
    - [18.4 Explicação de Código Sem Documentação](#184-explicação-de-código-sem-documentação)
19. [IA no Ciclo de Git e CI/CD](#19-ia-no-ciclo-de-git-e-cicd)
    - [19.1 Mensagens de Commit e PRs](#191-mensagens-de-commit-e-prs)
    - [19.2 Changelogs e Release Notes Automáticos](#192-changelogs-e-release-notes-automáticos)
    - [19.3 Revisão de IA em Pipelines CI/CD](#193-revisão-de-ia-em-pipelines-cicd)
    - [19.4 Gates de Qualidade com IA](#194-gates-de-qualidade-com-ia)
20. [Estratégias Avançadas de Contexto](#20-estratégias-avançadas-de-contexto)
    - [20.1 Estrutura do Repositório para IA](#201-estrutura-do-repositório-para-ia)
    - [20.2 .claudeignore e .geminiignore](#202-claudeignore-e-geminiignore)
    - [20.3 Índices e Mapas de Código](#203-índices-e-mapas-de-código)
    - [20.4 Quando Usar RAG vs Contexto Direto](#204-quando-usar-rag-vs-contexto-direto)
21. [IA para Design de API e Documentação OpenAPI](#21-ia-para-design-de-api-e-documentação-openapi)
    - [21.1 Contract-First Development com IA](#211-contract-first-development-com-ia)
    - [21.2 Geração de Spec OpenAPI a partir de Requisitos](#212-geração-de-spec-openapi-a-partir-de-requisitos)
    - [21.3 Geração de Código a partir de Spec](#213-geração-de-código-a-partir-de-spec)
    - [21.4 Validação de Implementação contra Contrato](#214-validação-de-implementação-contra-contrato)
22. [Avaliação da Qualidade do Código Gerado](#22-avaliação-da-qualidade-do-código-gerado)
    - [22.1 Reconhecendo Alucinações de Código](#221-reconhecendo-alucinações-de-código)
    - [22.2 Métricas Objetivas de Qualidade](#222-métricas-objetivas-de-qualidade)
    - [22.3 LLM-as-Judge](#223-llm-as-judge)
23. [Adoção de IA em Equipes](#23-adoção-de-ia-em-equipes)
    - [23.1 Introduzindo IA Gradualmente](#231-introduzindo-ia-gradualmente)
    - [23.2 Políticas de Uso](#232-políticas-de-uso)
    - [23.3 Compartilhamento de Prompts e Harness](#233-compartilhamento-de-prompts-e-harness)
    - [23.4 Autoria, Rastreabilidade e Ética](#234-autoria-rastreabilidade-e-ética)
24. [IA para Tarefas Não-Código no Desenvolvimento](#24-ia-para-tarefas-não-código-no-desenvolvimento)
    - [24.1 Planejamento de Sprint e Estimativas](#241-planejamento-de-sprint-e-estimativas)
    - [24.2 Diagramas com Mermaid e PlantUML](#242-diagramas-com-mermaid-e-plantuml)
    - [24.3 Architecture Decision Records (ADRs)](#243-architecture-decision-records-adrs)
25. [Fine-tuning e RAG sobre Codebase Próprio](#25-fine-tuning-e-rag-sobre-codebase-próprio)
    - [25.1 Quando Vale a Pena Ir Além do Contexto Nativo](#251-quando-vale-a-pena-ir-além-do-contexto-nativo)
    - [25.2 Ferramentas de RAG sobre Código](#252-ferramentas-de-rag-sobre-código)
    - [25.3 Privacidade e Opções On-Premise](#253-privacidade-e-opções-on-premise)
26. [Limitações e Modos de Falha](#26-limitações-e-modos-de-falha)
    - [26.1 Alucinações em APIs e Bibliotecas](#261-alucinações-em-apis-e-bibliotecas)
    - [26.2 Falhas Sistemáticas por Categoria](#262-falhas-sistemáticas-por-categoria)
    - [26.3 Reconhecendo Padrões de Falha Antes da Produção](#263-reconhecendo-padrões-de-falha-antes-da-produção)

---

## 1. Introdução ao Desenvolvimento Assistido por IA

O desenvolvimento de software assistido por IA deixou de ser uma curiosidade experimental para se tornar uma prática consolidada no dia a dia das equipes de engenharia. Ferramentas como **Claude Code**, **Gemini CLI** e **OpenAI Codex** atuam como programadores colaborativos que entendem contexto, leem repositórios inteiros, executam comandos e propõem soluções completas — não apenas completam linhas de código.

```
Evolução das ferramentas de IA para desenvolvimento

2021  GitHub Copilot        — sugestões de linha/bloco no editor
2022  ChatGPT               — conversação livre, geração de código
2023  GitHub Copilot Chat   — chat contextual no IDE
2024  Claude Code / Codex   — agentes autônomos com acesso ao sistema
2025  Gemini CLI / MCP      — protocolo padrão para contexto e ferramentas
```

### Principais paradigmas modernos

| Paradigma               | Descrição resumida                                              |
|-------------------------|-----------------------------------------------------------------|
| **Vibe Coding**         | Descrever intenção em linguagem natural e aceitar o resultado   |
| **Spec-Driven Dev.**    | Escrever especificação estruturada antes de pedir código        |
| **Harness Engineering** | Configurar o ambiente de IA para maximizar consistência         |
| **Agentic Coding**      | IA executa tarefas completas de forma autônoma (lê, edita, testa) |
| **MCP**                 | Protocolo aberto para fornecer ferramentas e contexto a LLMs    |

---

## 2. Terminologia e Conceitos-Chave

### 2.1 Vibe Coding

**Vibe coding** (cunhado por Andrej Karpathy em 2025) descreve um modo de programar onde o desenvolvedor expressa a *intenção* em linguagem natural e aceita o código gerado pela IA sem revisão linha a linha — operando no "vibe" do que a IA produz.

```
Fluxo típico de vibe coding

Desenvolvedor: "Crie uma API REST de CRUD de produtos com Spring Boot,
                JPA e banco H2. Inclua validação e tratamento de erros."

IA: [gera toda a estrutura do projeto, controllers, service, repository,
     entidades, testes unitários e application.yml]

Desenvolvedor: aceita, executa, ajusta apenas o que quebra
```

**Quando é adequado:**
- Protótipos e MVPs rápidos
- Scripts descartáveis de automação
- Exploração de APIs desconhecidas
- Scaffolding inicial de projetos

**Quando NÃO é adequado:**
- Código de produção crítico sem revisão
- Sistemas financeiros ou de saúde
- Lógica de negócio complexa sem especificação clara

---

### 2.2 Spec-Driven Development (SDD)

**Spec-Driven Development** é a prática de escrever uma especificação estruturada e detalhada *antes* de solicitar código à IA. A especificação serve como contrato formal que guia a geração e permite validação posterior.

```
Ciclo SDD

  Requisito         Especificação        Código IA          Validação
  (informal)   →   (estruturada)   →   (gerado)      →   (automática)
  "Preciso de       "POST /api/          class Product      testes passam
   cadastro de       produtos retorna     Entity, Service,   contrato
   produtos"         201 com Location"    Controller"        respeitado
```

A diferença fundamental em relação ao vibe coding: a IA recebe *o que* deve produzir com precisão, não apenas *para quê*.

---

### 2.3 Harness Engineering

**Harness engineering** é a disciplina de configurar e manter o *harness* — o conjunto de arquivos de configuração, instruções do sistema, ferramentas MCP, hooks e convenções — que moldará o comportamento da IA dentro de um projeto.

O harness responde à pergunta: *"Como faço a IA se comportar de forma consistente em todas as interações com este projeto?"*

Componentes típicos de um harness:
- `CLAUDE.md` / `GEMINI.md` — instruções do projeto para a IA
- `settings.json` — permissões, tools habilitadas, hooks
- Servidores MCP configurados (banco de dados, docs internas, issues)
- Templates de especificação e convenções de código
- Scripts de validação pós-geração

---

### 2.4 Prompt Engineering

**Prompt engineering** é a arte de formular instruções que maximizem a qualidade e relevância da resposta do modelo. No contexto de desenvolvimento, envolve:

- **Clareza de tarefa**: especificar exatamente o que deve ser produzido
- **Contexto suficiente**: fornecer linguagem, framework, versão e restrições
- **Exemplos (few-shot)**: mostrar o padrão esperado
- **Restrições explícitas**: o que NÃO fazer
- **Formato de saída**: código, JSON, lista, explicação

```
Prompt fraco:
"Crie um endpoint de login"

Prompt forte:
"Crie um endpoint POST /api/auth/login no estilo Spring Boot 3.x que:
- recebe {email, password} no body (validados com @Valid)
- retorna 200 com {token, expiresAt} ou 401 com mensagem de erro
- usa o AuthService já existente (authenticate(email, password): UserDetails)
- gera JWT usando a classe JwtTokenProvider
- não armazena estado na sessão (stateless)
Não crie classes novas além do método no AuthController."
```

---

### 2.5 MCP — Model Context Protocol

**MCP (Model Context Protocol)** é um protocolo aberto, lançado pela Anthropic em 2024 e adotado pela comunidade, que padroniza como LLMs se conectam a ferramentas externas, fontes de dados e serviços.

```
Arquitetura MCP (simplificada)

┌─────────────────────────────────┐
│           LLM Host              │
│  (Claude Code / Gemini / etc.)  │
└────────────┬────────────────────┘
             │  protocolo MCP (JSON-RPC / stdio / HTTP)
    ┌────────┴────────────────────────────┐
    │                                     │
┌───┴────────┐  ┌───────────┐  ┌─────────┴──┐
│ MCP Server │  │ MCP Server│  │ MCP Server │
│ (Filesystem│  │ (Postgres)│  │  (GitHub)  │
│  local)    │  │           │  │            │
└────────────┘  └───────────┘  └────────────┘
```

Cada MCP Server expõe:
- **Tools**: ações que a IA pode executar (ex: `query_database`, `create_issue`)
- **Resources**: dados que a IA pode consultar (ex: schema do banco, docs)
- **Prompts**: templates reutilizáveis configurados por serviço

---

### 2.6 Agentes de IA (Agentic Coding)

Um **agente de IA** para desenvolvimento é um LLM que opera com um loop autônomo: recebe uma tarefa, usa ferramentas (lê arquivos, executa comandos, chama APIs), analisa resultados e itera até concluir — sem interação humana a cada passo.

```
Loop de um agente de coding

  Tarefa → [Ler código] → [Analisar] → [Editar arquivo] →
         → [Executar teste] → [Analisar falha] → [Corrigir] →
         → [Executar teste novamente] → [Concluído]
```

Claude Code, Gemini CLI e Codex CLI operam neste modo. A diferença para um chatbot comum: o agente *faz* coisas no sistema de arquivos, não apenas *sugere*.

---

### 2.7 Context Window e Gerenciamento de Contexto

**Context window** é a quantidade máxima de texto (tokens) que o modelo processa de uma vez. Modelos modernos têm janelas grandes (200k–1M tokens), mas ainda há limites práticos:

| Ferramenta     | Context window aprox. |
|----------------|-----------------------|
| Claude Sonnet 4.6 | 200.000 tokens     |
| Gemini 2.5 Pro | 1.000.000 tokens      |
| GPT-4o         | 128.000 tokens        |
| Codex (o1)     | 200.000 tokens        |

**Estratégias para gerenciar contexto:**
- Manter o `CLAUDE.md` / `GEMINI.md` conciso e focado
- Usar `/compact` ou `/clear` para limpar histórico quando mudar de tarefa
- Dividir tarefas grandes em sessões menores e focadas
- Usar MCP Resources para recuperar contexto sob demanda em vez de incluir tudo no prompt

---

### 2.8 RAG — Retrieval-Augmented Generation

**RAG** combina recuperação de informação com geração. Em vez de incluir toda a documentação no contexto, o sistema busca apenas os trechos relevantes para cada pergunta.

Em desenvolvimento de software, RAG é usado para:
- Consultar documentação de APIs internas
- Buscar exemplos no codebase
- Recuperar histórico de decisões arquiteturais
- Acessar tickets e PRs relevantes

Servidores MCP funcionam como camada RAG para LLMs: fornecem contexto recuperado sob demanda.

---

## 3. Claude Code

### 3.1 Visão Geral

**Claude Code** é a CLI agentic da Anthropic que executa diretamente no terminal. Lê o repositório, edita arquivos, executa comandos shell, usa ferramentas MCP e itera de forma autônoma. É especialmente eficaz em projetos com contexto bem definido via `CLAUDE.md`.

```
Modelos disponíveis (2025)

claude-opus-4-7        — máxima capacidade, tarefas complexas
claude-sonnet-4-6      — equilíbrio custo/desempenho (padrão)
claude-haiku-4-5       — mais rápido, tarefas simples
```

### 3.2 Instalação e Configuração

```bash
# Instalar via npm (Node.js 18+ necessário)
npm install -g @anthropic-ai/claude-code

# Autenticar (abre browser para OAuth)
claude

# Verificar versão
claude --version

# Iniciar em um projeto
cd meu-projeto
claude
```

**Variáveis de ambiente úteis:**

```bash
# .env ou perfil do shell
ANTHROPIC_API_KEY=sk-ant-...         # chave API direta (alternativa ao OAuth)
CLAUDE_MODEL=claude-sonnet-4-6       # modelo padrão
CLAUDE_MAX_TOKENS=8192               # limite de tokens por resposta
```

---

### 3.3 O Arquivo CLAUDE.md

O `CLAUDE.md` é o principal mecanismo de **harness engineering** no Claude Code. É lido automaticamente no início de cada sessão e define o comportamento esperado da IA para o projeto.

```markdown
<!-- CLAUDE.md — exemplo para projeto Spring Boot -->
# Projeto: Sistema de Gestão de Pedidos

## Stack e Versões
- Java 21 (LTS), Spring Boot 3.4, Maven
- PostgreSQL 16 (produção), H2 (testes)
- Flyway para migrations de banco
- JUnit 5 + Mockito para testes

## Estrutura de Pacotes
```
com.empresa.pedidos
├── controller     # @RestController — apenas HTTP, sem lógica
├── service        # @Service — lógica de negócio
├── repository     # @Repository — JPA, sem queries em XML
├── entity         # @Entity — sem lógica, apenas mapeamento
├── dto            # Records Java (imutáveis), validação com @Valid
└── exception      # exceções do domínio, GlobalExceptionHandler
```

## Convenções Obrigatórias
- DTOs sempre como Java Records (não classes)
- Injeção de dependência sempre por construtor (não @Autowired em campo)
- Nunca retornar entidades JPA diretamente em controllers
- Migrations Flyway: padrão `V{número}__{descricao}.sql`
- Testes: nomenclatura `deveria_{comportamento}_quando_{condição}`

## O que NÃO fazer
- Não criar arquivos XML de configuração (use annotations)
- Não usar Optional.get() sem isPresent()
- Não fazer lógica de negócio em controllers
- Não commitar sem executar `mvn verify`
```

**Hierarquia de CLAUDE.md:**

```
~/.claude/CLAUDE.md          ← instruções globais (todos os projetos)
~/projetos/CLAUDE.md         ← instruções da pasta pai
~/projetos/meu-app/CLAUDE.md ← instruções específicas do projeto (prioridade)
```

---

### 3.4 Configurações (settings.json)

O arquivo `~/.claude/settings.json` (global) ou `.claude/settings.json` (projeto) controla permissões, ferramentas e comportamentos automatizados.

```json
{
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": [
      "Bash(mvn:*)",
      "Bash(git:*)",
      "Bash(npm:*)",
      "Read(*)",
      "Edit(*)",
      "Write(*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(curl:*)",
      "Bash(wget:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "mvn checkstyle:check -q 2>&1 | head -20"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Sessão Claude Code encerrada' >> .claude/session.log"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/meubanco"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projetos"]
    }
  }
}
```

---

### 3.5 Slash Commands

Comandos especiais digitados diretamente no prompt do Claude Code:

| Comando           | Ação                                                        |
|-------------------|-------------------------------------------------------------|
| `/help`           | Lista todos os comandos disponíveis                         |
| `/clear`          | Limpa o histórico da conversa (nova sessão)                 |
| `/compact`        | Comprime o histórico mantendo contexto essencial            |
| `/config`         | Abre editor de configurações                                |
| `/model`          | Troca o modelo em uso                                       |
| `/fast`           | Ativa modo rápido (Opus com output acelerado)               |
| `/review`         | Executa skill de code review do PR atual                    |
| `/init`           | Cria CLAUDE.md com documentação do codebase                 |
| `/security-review`| Análise de segurança do branch atual                        |

**Criando slash commands customizados:**

```bash
# Criar skill customizado em ~/.claude/commands/test-report.md
# O arquivo descreve o comportamento do comando /test-report
```

```markdown
<!-- ~/.claude/commands/test-report.md -->
Execute os testes com `mvn test` e produza um relatório resumido com:
- Número de testes passando/falhando
- Lista de testes com falha com a causa raiz
- Sugestão de correção para cada falha
```

---

### 3.6 Hooks

Hooks executam scripts shell automaticamente em resposta a eventos do Claude Code — sem intervenção manual. São o coração do harness engineering.

```
Eventos disponíveis para hooks

PreToolUse    — antes de qualquer tool call
PostToolUse   — após qualquer tool call
Notification  — quando Claude envia uma notificação
Stop          — ao final da sessão ou resposta
SubagentStop  — ao final de um sub-agente
```

**Exemplo: validar checkstyle após cada edição de Java:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "find . -name '*.java' -newer .last-check -exec checkstyle {} \\; 2>&1 | head -30",
            "timeout": 15000
          }
        ]
      }
    ]
  }
}
```

**Exemplo: executar testes automaticamente após alterações críticas:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if git diff --name-only | grep -q 'Service\\|Repository'; then mvn test -q 2>&1 | tail -20; fi"
          }
        ]
      }
    ]
  }
}
```

---

### 3.7 MCP no Claude Code

Adicionar servidores MCP ao `settings.json` fornece ferramentas e recursos adicionais ao agente:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres",
               "postgresql://user:pass@localhost:5432/meubanco"],
      "env": {
        "PGPASSWORD": "${PGPASSWORD}"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

Com `postgres` configurado, é possível:
```
"Consulte o schema da tabela 'pedidos' e crie uma entidade JPA correspondente"
"Gere uma migration Flyway para adicionar o campo 'status' à tabela 'clientes'"
```

---

### 3.8 Boas Práticas com Claude Code

**1. Sempre ter um CLAUDE.md atualizado**
É a forma mais eficaz de garantir consistência entre sessões.

**2. Sessões focadas por tarefa**
Use `/clear` entre tarefas não relacionadas. Contexto irrelevante degrada a qualidade.

**3. Revisar antes de aceitar**
Claude Code pode editar muitos arquivos de uma vez. Use `git diff` para revisar antes de commitar.

**4. Permissões mínimas necessárias**
Configure `deny` para comandos destrutivos ou que gerem side effects indesejados.

**5. Usar `/compact` em sessões longas**
Mantém o contexto essencial sem desperdiçar tokens com histórico de debugging.

**6. Commitar em checkpoints**
Antes de pedir mudanças grandes, faça um commit. Isso torna fácil reverter se necessário.

**7. Especificar o escopo**
"Altere apenas o método `calcularDesconto` na classe `PedidoService`. Não modifique outros arquivos."

---

### 3.9 Exemplos de Prompts para Claude Code

**Scaffolding de nova feature:**
```
Crie o módulo de gestão de fornecedores seguindo a estrutura do módulo de clientes
já existente em src/main/java/com/empresa/clientes/. O fornecedor tem: razaoSocial,
cnpj (único), email, telefone e ativo (boolean). Inclua:
- Entidade JPA com validações
- DTO de entrada e saída (Records)
- Repository com query findByAtivoTrue()
- Service com CRUD completo
- Controller REST em /api/fornecedores
- Migration Flyway V5__cria_tabela_fornecedor.sql
- Testes unitários do Service com Mockito
```

**Refatoração com restrição:**
```
O método `processarPedido` em PedidoService.java:45 tem 120 linhas e faz muitas
coisas. Extraia métodos privados para: validarEstoque(), calcularTotais(),
aplicarDescontos() e notificarCliente(). Não altere a assinatura pública nem o
comportamento observável — os testes existentes devem continuar passando.
```

**Debugging com contexto de erro:**
```
Estou recebendo este erro em produção:
```
org.hibernate.LazyInitializationException: could not initialize proxy
[com.empresa.Pedido#42] - no Session
  at PedidoService.buscarComItens(PedidoService.java:78)
```
Analise o código em PedidoService.java e PedidoController.java e corrija o
problema. Prefira usar @EntityGraph a @Transactional no controller.
```

**Geração de testes:**
```
Analise a classe DescontoService.java e escreva testes JUnit 5 com Mockito para
todos os métodos públicos. Para cada método, cubra: caminho feliz, entradas
inválidas/nulas e casos de borda listados nos comentários da classe. Use o padrão
de nomenclatura `deveria_{resultado}_quando_{condição}`.
```

---

## 4. Google Gemini

### 4.1 Visão Geral das Ferramentas Gemini

O ecossistema Gemini para desenvolvedores inclui:

| Ferramenta            | Contexto de uso                                |
|-----------------------|------------------------------------------------|
| **Gemini CLI**        | Agente agentic no terminal (similar ao Claude Code) |
| **Gemini Code Assist**| Extensão IDE (VS Code, JetBrains)              |
| **AI Studio**         | Interface web para experimentação de prompts   |
| **Vertex AI**         | API enterprise para integração em produtos     |
| **NotebookLM**        | RAG sobre documentos próprios                  |

Modelos disponíveis (2025):

```
gemini-2.5-pro      — máxima capacidade, 1M tokens de contexto
gemini-2.5-flash    — rápido e econômico, bom custo-benefício
gemini-2.0-flash    — versão anterior, estável e amplamente disponível
```

---

### 4.2 Gemini CLI

O **Gemini CLI** é a interface agentic de linha de comando do Google para o Gemini, lançada em 2025, com suporte nativo a MCP e contexto de 1 milhão de tokens.

```bash
# Instalar via npm
npm install -g @google/gemini-cli

# Autenticar (Google Account ou API Key)
gemini auth login

# Iniciar em um projeto
cd meu-projeto
gemini

# Modo não-interativo
gemini -p "Liste todos os endpoints REST do projeto"
```

**Atalhos e flags úteis:**

```bash
gemini --model gemini-2.5-pro    # escolher modelo
gemini --sandbox                  # modo isolado (sem escrita em disco)
gemini --yolo                     # aprovar todas as ações automaticamente
gemini --debug                    # logs detalhados de tool calls
```

---

### 4.3 Gemini Code Assist (IDE)

**Gemini Code Assist** é a extensão para VS Code e JetBrains, com:
- Completions inline (como GitHub Copilot)
- Chat contextual sobre o código aberto
- Geração de testes
- Revisão de código em PRs (integração GitHub/GitLab)
- Contexto de todo o repositório via indexação

**Configuração no VS Code (settings.json do editor):**

```json
{
  "geminicodeassist.project": "meu-projeto-gcp",
  "geminicodeassist.enableInlineCompletion": true,
  "geminicodeassist.enableCodeActions": true,
  "geminicodeassist.maxCompletionTokens": 2048
}
```

---

### 4.4 O Arquivo GEMINI.md

Equivalente ao `CLAUDE.md`, o `GEMINI.md` é carregado automaticamente pelo Gemini CLI no início de cada sessão.

```markdown
<!-- GEMINI.md — exemplo para projeto Angular + Spring Boot -->
# Projeto: Portal do Aluno

## Stack
- Backend: Java 21, Spring Boot 3.4, PostgreSQL 16
- Frontend: Angular 19, TypeScript 5.7, Standalone Components
- Build: Maven (backend), Angular CLI (frontend)
- Testes: JUnit 5 + Mockito (backend), Jest + Testing Library (frontend)

## Arquitetura
- Backend API REST em /api/v1/**
- Frontend SPA em /
- JWT stateless (header Authorization: Bearer)
- CORS configurado para localhost:4200 em dev

## Regras de código
### Backend
- Controllers finos: apenas validação HTTP, delegam para Service
- Services: lógica de negócio, transações com @Transactional
- DTOs: Java Records para imutabilidade
### Frontend
- Componentes standalone (não usar NgModules)
- Signals para estado reativo (@signal, @computed, @effect)
- HttpClient com Observables + async pipe nos templates
- Serviços injetáveis com providedIn: 'root'

## Comandos úteis
- Backend: `mvn spring-boot:run`
- Frontend: `ng serve`
- Testes backend: `mvn test`
- Testes frontend: `ng test --watch=false`
- Build produção: `mvn package -P prod`
```

---

### 4.5 MCP no Gemini CLI

O Gemini CLI suporta MCP nativamente via arquivo `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projetos"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres",
               "postgresql://localhost/meubanco"]
    }
  }
}
```

---

### 4.6 Boas Práticas com Gemini

**1. Aproveitar o contexto enorme**
Com 1M de tokens, o Gemini 2.5 Pro pode processar repositórios maiores inteiros. Mas contexto grande = custo maior e resposta mais lenta. Use com propósito.

**2. Grounding com Google Search**
O Gemini CLI pode buscar informações atualizadas via Google Search. Útil para:
- Verificar se uma biblioteca tem vulnerabilidade conhecida
- Consultar documentação de versões recentes de frameworks
```
"Verifique a documentação mais recente do Spring Boot 3.4 para configurar
o novo suporte a Virtual Threads e aplique no nosso projeto"
```

**3. Usar o AI Studio para iterar prompts**
Antes de usar um prompt complexo na CLI, teste-o no Google AI Studio para ajustar o comportamento sem custo de execução.

**4. Gemini para tarefas multimodais**
Diferente de Claude Code, o Gemini aceita imagens e PDFs diretamente:
```
[anexa screenshot do erro]
"Este é o erro que aparece na tela. Analise e corrija o código responsável."
```

---

### 4.7 Exemplos de Prompts para Gemini

**Análise de repositório completo:**
```
Leia todo o projeto e produza um relatório com:
1. Diagrama de componentes (texto ASCII)
2. Principais fluxos de dados identificados
3. Problemas de arquitetura detectados
4. Dependências desatualizadas críticas
5. Estimativa de dívida técnica por módulo
```

**Migração de versão:**
```
O projeto usa Spring Boot 2.7. Analise o pom.xml e todos os arquivos Java e
produza um plano de migração para Spring Boot 3.4, listando:
- Mudanças obrigatórias (breaking changes)
- Dependências a atualizar com versões específicas
- Código a refatorar (javax → jakarta, etc.)
- Ordem recomendada de execução das mudanças
Em seguida, execute as mudanças no código com minha aprovação a cada etapa.
```

**Revisão de segurança:**
```
Revise todo o código do módulo de autenticação (src/main/java/**/auth/**) e
identifique vulnerabilidades de segurança, especialmente:
- Injeção SQL
- Problemas de JWT (alg:none, segredo fraco, sem validação de expiração)
- Senhas hardcoded
- Logging de dados sensíveis
Para cada problema encontrado, forneça o código corrigido e a justificativa.
```

---

## 5. OpenAI Codex e ChatGPT

### 5.1 Visão Geral

O ecossistema OpenAI para desenvolvimento inclui:

| Ferramenta          | Contexto de uso                                        |
|---------------------|--------------------------------------------------------|
| **Codex CLI**       | Agente agentic no terminal, similar ao Claude Code     |
| **ChatGPT**         | Interface conversacional web/app                       |
| **GitHub Copilot**  | Completions e chat no IDE (baseado em modelos OpenAI)  |
| **API (o1/o3/GPT-4o)** | Integração direta em produtos via API              |

---

### 5.2 OpenAI Codex CLI

```bash
# Instalar
npm install -g @openai/codex

# Configurar API key
export OPENAI_API_KEY=sk-...

# Iniciar em projeto
cd meu-projeto
codex

# Modo não-interativo com aprovação automática
codex --approval-mode full-auto -p "Execute os testes e corrija falhas"
```

**Modos de aprovação:**

```
suggest       — sugere, nunca executa automaticamente (padrão)
auto-edit     — edita arquivos automaticamente, pede para comandos shell
full-auto     — executa tudo automaticamente (use com cuidado)
```

**Arquivo de instrução `AGENTS.md`:**
```markdown
<!-- AGENTS.md — equivalente ao CLAUDE.md para Codex -->
# Instruções para o agente Codex

## Projeto
API REST Spring Boot 3.x, Java 21, Maven.

## Convenções
- Use sempre Lombok (@Data, @Builder, @RequiredArgsConstructor)
- Testes com sufixo Test, nunca Spec
- Profiles: dev (H2), test (H2), prod (PostgreSQL)

## Permissões
- Pode editar qualquer arquivo Java ou XML
- Não modifique arquivos de configuração de CI/CD (.github/)
- Não execute comandos de deploy ou push
```

---

### 5.3 GitHub Copilot

O **GitHub Copilot** é integrado aos IDEs (VS Code, JetBrains, Vim) e ao GitHub.com. Funciona em dois modos:

**Inline completions**: sugere código durante a digitação, ativado por `Tab`.

**Copilot Chat**: conversa sobre o código aberto. Comandos especiais:

| Comando     | Ação                                                |
|-------------|-----------------------------------------------------|
| `/explain`  | Explica o código selecionado                        |
| `/fix`      | Corrige bugs no código selecionado                  |
| `/tests`    | Gera testes para o código selecionado               |
| `/doc`      | Gera documentação (Javadoc/JSDoc)                   |
| `/optimize` | Sugere otimizações de performance                   |

**Copilot Instructions (`.github/copilot-instructions.md`):**

```markdown
<!-- .github/copilot-instructions.md -->
Este repositório usa Java 21 e Spring Boot 3.4.

Padrões obrigatórios:
- Nunca use @Autowired em campos; sempre injeção por construtor
- DTOs são Records Java imutáveis
- Exceções do domínio herdam de BusinessException
- Todas as queries JPQL no Repository, nunca no Service

Não gere comentários óbvios — apenas Javadoc em métodos públicos de API.
```

---

### 5.4 ChatGPT para Desenvolvimento

O **ChatGPT** (especialmente com GPT-4o ou o1/o3) é eficaz para:

- Explicar conceitos e algoritmos
- Code review com contexto limitado (cole o arquivo relevante)
- Geração de boilerplate com ajustes iterativos
- Debugging com stack traces
- Geração de queries SQL complexas

**Dica: usar o modo Canvas/Artifacts**
O modo Canvas do ChatGPT permite editar código gerado diretamente na interface, com realce de sintaxe e execução (para Python/JS).

**Limitação principal**: sem acesso ao sistema de arquivos — todo contexto deve ser colado manualmente.

---

### 5.5 Boas Práticas com OpenAI

**1. Escolher o modelo certo para a tarefa:**

```
GPT-4o      → tarefas de código gerais, boa velocidade
o1 / o3     → problemas complexos que exigem raciocínio (refatorações difíceis,
              algoritmos, debugging de concorrência)
GPT-4o-mini → tarefas simples e rápidas (renomear variáveis, formatar JSON)
```

**2. System prompt consistente na API:**

```python
import openai

SYSTEM_PROMPT = """Você é um engenheiro Java sênior especialista em Spring Boot.
Sempre gere código seguindo as convenções do projeto:
- Java Records para DTOs
- Injeção por construtor
- Testes com JUnit 5 e Mockito
Responda em português brasileiro."""

response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": "Crie o endpoint de busca de produtos por categoria"}
    ]
)
```

**3. Structured Outputs para geração confiável:**

```python
from pydantic import BaseModel
from typing import List

class CodeChange(BaseModel):
    file_path: str
    description: str
    code: str
    reason: str

class RefactoringPlan(BaseModel):
    changes: List[CodeChange]
    estimated_risk: str  # "low" | "medium" | "high"
    test_impact: str

response = openai.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[...],
    response_format=RefactoringPlan
)
plan = response.choices[0].message.parsed
```

---

### 5.6 Exemplos de Prompts para OpenAI

**Para ChatGPT (cole o código junto):**
```
Analisando o seguinte service Spring Boot [cole o código], identifique:
1. Violações dos princípios SOLID
2. Problemas de performance em loops ou queries N+1
3. Tratamento inadequado de exceções
4. Código duplicado extraível em métodos auxiliares

Para cada problema, forneça o código refatorado e a justificativa.
```

**Para Codex CLI:**
```
Analise o diretório src/main/java e identifique todos os lugares onde estou
usando @Autowired em campos (field injection). Substitua cada ocorrência por
injeção de construtor. Use @RequiredArgsConstructor do Lombok quando o campo
for final. Execute os testes após as mudanças para confirmar que nada quebrou.
```

**Para GitHub Copilot Chat (no IDE):**
```
/explain Explique a lógica do método processarPedido considerando o contexto
do domínio de e-commerce — por que cada validação existe?
```

---

## 6. MCP — Model Context Protocol em Profundidade

### 6.1 Arquitetura MCP

O MCP define três papéis:

```
┌──────────────────────────────────────────────────────┐
│                    MCP HOST                          │
│  (Claude Code, Gemini CLI, aplicação customizada)   │
│                                                      │
│  ┌─────────────┐    ┌──────────────────────────┐    │
│  │  LLM Engine │◄──►│     MCP Client           │    │
│  └─────────────┘    └───────────┬──────────────┘    │
└──────────────────────────────────┼───────────────────┘
                                   │ JSON-RPC 2.0
            ┌──────────────────────┼──────────────────┐
            │                      │                  │
   ┌────────┴──────┐    ┌──────────┴────┐    ┌────────┴──────┐
   │  MCP Server   │    │  MCP Server   │    │  MCP Server   │
   │  Filesystem   │    │  Postgres     │    │  GitHub       │
   │               │    │               │    │               │
   │ Tools:        │    │ Tools:        │    │ Tools:        │
   │ - read_file   │    │ - query       │    │ - create_pr   │
   │ - write_file  │    │ - describe    │    │ - list_issues │
   │ - list_dir    │    │ Resources:    │    │ - get_commit  │
   │               │    │ - schema      │    │               │
   └───────────────┘    └───────────────┘    └───────────────┘
```

**Tipos de capacidades MCP:**

- **Tools** — ações executáveis (efeitos colaterais: banco, API, filesystem)
- **Resources** — dados consultáveis sem efeito colateral (schema, docs)
- **Prompts** — templates reutilizáveis parametrizados
- **Sampling** — o servidor pode solicitar inferência ao LLM host

---

### 6.2 Servidores MCP Populares

| Servidor                                | Capacidades                                        | Instalação                              |
|-----------------------------------------|----------------------------------------------------|-----------------------------------------|
| `@modelcontextprotocol/server-filesystem` | Ler/escrever/listar arquivos locais              | `npx -y @modelcontextprotocol/...`      |
| `@modelcontextprotocol/server-postgres`  | Query, describe, list tables                      | `npx -y @modelcontextprotocol/...`      |
| `@modelcontextprotocol/server-sqlite`    | SQLite local                                       | `npx -y @modelcontextprotocol/...`      |
| `@modelcontextprotocol/server-github`    | Issues, PRs, commits, repos                       | Requer `GITHUB_TOKEN`                   |
| `@modelcontextprotocol/server-gitlab`    | Equivalente ao GitHub para GitLab                 | Requer `GITLAB_TOKEN`                   |
| `@modelcontextprotocol/server-brave-search` | Busca web via Brave API                        | Requer `BRAVE_API_KEY`                  |
| `@modelcontextprotocol/server-memory`    | Armazenamento persistente de fatos entre sessões  | Sem configuração adicional              |
| `@modelcontextprotocol/server-puppeteer` | Controlar browser Chromium                        | Requer Puppeteer                        |
| `@modelcontextprotocol/server-redis`     | Get/set/del no Redis                              | Requer Redis rodando                    |

---

### 6.3 Criando um Servidor MCP Customizado

Exemplo de servidor MCP simples em Node.js para acessar a API interna de tickets:

```typescript
// ticket-mcp-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "ticket-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_ticket",
      description: "Busca detalhes de um ticket pelo ID",
      inputSchema: {
        type: "object",
        properties: {
          ticket_id: { type: "string", description: "ID do ticket (ex: PROJ-123)" }
        },
        required: ["ticket_id"]
      }
    },
    {
      name: "list_open_tickets",
      description: "Lista tickets abertos de um projeto",
      inputSchema: {
        type: "object",
        properties: {
          project: { type: "string" },
          assignee: { type: "string", description: "Filtrar por responsável (opcional)" }
        },
        required: ["project"]
      }
    }
  ]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "get_ticket") {
    const response = await fetch(
      `https://api.empresa.com/tickets/${args.ticket_id}`,
      { headers: { Authorization: `Bearer ${process.env.TICKET_API_KEY}` } }
    );
    const ticket = await response.json();
    return { content: [{ type: "text", text: JSON.stringify(ticket, null, 2) }] };
  }

  // ... demais tools
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

```bash
# Compilar e registrar no settings.json
npx tsc ticket-mcp-server.ts --outDir dist/
```

```json
{
  "mcpServers": {
    "tickets": {
      "command": "node",
      "args": ["dist/ticket-mcp-server.js"],
      "env": {
        "TICKET_API_KEY": "${TICKET_API_KEY}"
      }
    }
  }
}
```

Agora é possível usar no Claude Code:
```
"Consulte o ticket PROJ-247 e implemente as alterações especificadas nele"
```

---

## 7. Spec-Driven Development com IA

### 7.1 Motivação e Conceito

O vibe coding funciona bem para protótipos, mas produz código imprevisível em sistemas complexos: a IA preenche lacunas com suposições que podem não refletir os requisitos reais.

O **Spec-Driven Development** (SDD) inverte a ordem: a especificação vem primeiro e é detalhada o suficiente para que o código gerado possa ser *validado automaticamente* contra ela.

```
Vibe Coding:   tarefa vaga → código → "parece certo?" → aceitar
Spec-Driven:   spec formal → código → testes automatizados → verificar
```

A especificação serve como:
1. **Contrato** entre o desenvolvedor e a IA
2. **Critério de aceitação** verificável com testes
3. **Documentação** do comportamento esperado
4. **Base para revisão** de código gerado

---

### 7.2 Estrutura de uma Especificação

Uma boa especificação para IA contém:

```markdown
## Especificação: [Nome da Feature]

### Contexto
[O que já existe, qual problema resolve]

### Endpoint / Interface
[Assinatura exata: URL, método HTTP, parâmetros, retornos]

### Regras de Negócio
1. [Regra clara e testável]
2. [...]

### Casos de Erro
- [Condição] → [Comportamento esperado, código HTTP, mensagem]

### Entradas / Saídas
[Schema JSON ou assinatura de método]

### Restrições de Implementação
- [O que usar/não usar]
- [Performance, segurança, dependências]

### Critérios de Aceitação (Gherkin opcional)
Given [pré-condição]
When [ação]
Then [resultado esperado]
```

---

### 7.3 Fluxo de Trabalho SDD Completo

```
1. ESCREVER SPEC
   Desenvolvedor escreve spec detalhada em Markdown
   (pode usar IA para ajudar a redigir a spec a partir de requisitos vagos)

2. REVISAR SPEC
   Equipe valida a spec antes de gerar qualquer código
   Dúvidas são resolvidas na spec, não no código

3. GERAR CÓDIGO
   "Implemente a feature descrita em SPEC.md seguindo as convenções do CLAUDE.md"

4. GERAR TESTES
   "Com base na spec em SPEC.md, escreva testes que cobram todos os critérios
   de aceitação e casos de erro descritos"

5. VALIDAR
   Executar os testes automaticamente
   Se falhar → ajustar código (não a spec)

6. REVISAR
   Code review humano focado em: segurança, performance, legibilidade
   A correção lógica já foi verificada pelos testes
```

---

### 7.4 Exemplos Práticos (Spring Boot / Java)

**Spec de endpoint de cadastro de pedido:**

```markdown
## Spec: POST /api/pedidos — Criar Pedido

### Contexto
O cliente autenticado cria um pedido selecionando produtos do catálogo.

### Endpoint
POST /api/pedidos
Authorization: Bearer {jwt}
Content-Type: application/json

### Request Body
{
  "itens": [
    { "produtoId": "uuid", "quantidade": integer (>0) }
  ],
  "enderecoEntregaId": "uuid"
}

### Response — Sucesso (201 Created)
Location: /api/pedidos/{id}
{
  "id": "uuid",
  "numero": "PED-2025-00042",
  "status": "AGUARDANDO_PAGAMENTO",
  "total": 249.90,
  "itens": [...],
  "criadoEm": "ISO-8601"
}

### Regras de Negócio
1. Cada produto deve estar ativo (ativo = true) e com estoque >= quantidade solicitada
2. O enderecoEntregaId deve pertencer ao cliente autenticado
3. O número do pedido é gerado sequencialmente no formato PED-{ano}-{sequencial 5 dígitos}
4. O preço unitário é capturado no momento da criação (não muda se o produto mudar de preço)
5. Estoque é reservado (não decrementado) até confirmação do pagamento

### Casos de Erro
- itens vazio ou null           → 400 "Pedido deve conter ao menos um item"
- quantidade <= 0               → 400 "Quantidade deve ser positiva"
- produto não encontrado        → 404 "Produto {id} não encontrado"
- produto sem estoque           → 409 "Estoque insuficiente para produto {nome}"
- endereço não pertence ao user → 403 "Endereço não autorizado"
- usuário não autenticado       → 401

### Restrições de Implementação
- Usar @Transactional no Service para garantir atomicidade
- Não retornar entidades JPA — usar PedidoResponseDTO (Record)
- A reserva de estoque deve ser feita em EstoqueService (já existente)
```

**Prompt para geração de código a partir da spec:**
```
Implemente a feature descrita em docs/specs/criar-pedido.md.

Contexto existente:
- ProdutoRepository, EstoqueService e EnderecoRepository já existem
- Autenticação JWT já está configurada — use @AuthenticationPrincipal UserDetails
- Estrutura de pacotes e convenções estão em CLAUDE.md

Crie apenas os arquivos necessários para esta feature específica.
Após criar o código, gere os testes unitários do PedidoService cobrindo
todos os casos de erro da spec.
```

---

## 8. Vibe Coding — Uso e Riscos

### 8.1 O que é Vibe Coding

Vibe coding é uma abordagem onde o desenvolvedor descreve **o que quer** em linguagem natural e aceita o código gerado pela IA com revisão mínima. O foco está na **velocidade de prototipagem**, não na perfeição do código.

A palavra "vibe" captura a ideia de que o desenvolvedor e a IA entram em sintonia — o desenvolvedor não precisa saber como implementar, apenas descrever a intenção com clareza suficiente.

```
Sessão típica de vibe coding

Dev: "Cria um app web simples de lista de tarefas com React"
IA:  [gera todo o projeto com CRUD, localStorage, UI completa]
Dev: "Adiciona filtro por status (todas/pendentes/concluídas)"
IA:  [adiciona filtros]
Dev: "Torna responsivo para mobile"
IA:  [ajusta CSS]
Dev: [abre no browser, testa, pede ajustes menores]
```

**Resultado típico:** funcionando em 20 minutos sem escrever uma linha.

---

### 8.2 Quando Usar e Quando Evitar

**Usar vibe coding para:**
- Scripts de automação internos descartáveis
- Protótipos de validação de ideia (antes de spec)
- Ferramentas internas de baixo risco
- Aprendizado de novas tecnologias
- Demos e proof-of-concept
- Frontends simples sem lógica de negócio crítica

**Evitar vibe coding em:**
- Código de produção de sistemas críticos
- Lógica financeira (cálculos de impostos, preços, descontos)
- Autenticação e autorização
- Sistemas com compliance regulatório (LGPD, PCI-DSS, HIPAA)
- Código que outros vão manter sem documentação
- Queries em bancos de produção com dados reais

---

### 8.3 Riscos e Mitigação

| Risco                          | Mitigação                                             |
|--------------------------------|-------------------------------------------------------|
| Código funciona mas é inseguro | Code review de segurança antes de subir para produção |
| Lógica incorreta não testada   | Sempre gerar testes junto com o código                |
| Dívida técnica elevada         | Refatorar após validar a ideia                        |
| Dependências desnecessárias    | Revisar `package.json`/`pom.xml` gerado               |
| Alucinação (API inexistente)   | Testar localmente, não confiar apenas na leitura      |
| Inconsistência entre sessões   | Manter `CLAUDE.md` atualizado com convenções          |

**Regra prática:** vibe coding para descobrir *o que* construir; spec-driven development para construir *corretamente*.

---

## 9. Harness Engineering

### 9.1 Conceito

**Harness engineering** é a prática de configurar o *ambiente* da IA — não apenas os prompts individuais — para que a IA se comporte de forma consistente e previsível em todo o projeto. O harness é o "contrato operacional" entre o desenvolvedor e a IA.

```
Sem harness                    Com harness

Dev: "Crie um DTO"             Dev: "Crie o DTO de resposta do pedido"
IA: [gera classe com getters]  IA: [gera Record imutável, pois CLAUDE.md
                                    especifica Records para DTOs]

Dev: "Crie um controller"      Dev: "Crie o controller de pedidos"
IA: [usa @Autowired no campo]  IA: [usa injeção por construtor + @RequiredArgsConstructor,
                                    pois CLAUDE.md proíbe field injection]
```

---

### 9.2 Componentes do Harness

```
harness/
├── CLAUDE.md              ← instruções do projeto (lidas automaticamente)
├── GEMINI.md              ← equivalente para Gemini CLI
├── .github/
│   └── copilot-instructions.md  ← para GitHub Copilot
├── AGENTS.md              ← para Codex CLI
├── .claude/
│   ├── settings.json      ← permissões, hooks, MCP servers
│   └── commands/          ← slash commands customizados
├── docs/
│   ├── specs/             ← especificações SDD
│   └── architecture/      ← decisões de arquitetura (ADRs)
└── scripts/
    ├── validate-code.sh   ← executado por hooks
    └── run-tests.sh       ← executado por hooks
```

---

### 9.3 CLAUDE.md como Harness Central

Um `CLAUDE.md` bem estruturado é o componente mais impactante do harness. Seções recomendadas:

```markdown
# [Nome do Projeto]

## Stack e Versões
[Linguagens, frameworks, versões exatas — a IA não deve adivinhar]

## Estrutura de Pacotes / Módulos
[Mapa da arquitetura com responsabilidades de cada camada]

## Convenções de Código
[Padrões que DEVEM ser seguidos sempre]

## Anti-padrões (O que NÃO fazer)
[O que a IA tende a gerar errado neste projeto]

## Comandos de Build e Teste
[Como rodar, testar e verificar o projeto]

## Contexto de Domínio
[Glossário dos termos do negócio usados no código]

## Integrações Externas
[APIs, serviços e sistemas integrados com notas sobre autenticação]

## Guia de Fluxo de Trabalho
[Como o time trabalha: branches, commits, PRs, revisões]
```

---

### 9.4 Configuração Avançada de Hooks

Hooks transformam o harness em um sistema ativo de validação:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[HOOK] Comando bash prestes a executar: $CLAUDE_TOOL_INPUT' >> .claude/audit.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash scripts/validate-code.sh 2>&1 | head -50",
            "timeout": 30000
          }
        ]
      }
    ]
  }
}
```

```bash
# scripts/validate-code.sh
#!/bin/bash
set -e

echo "=== Verificando arquivos Java alterados ==="

# Checkstyle nos arquivos modificados
MODIFIED=$(git diff --name-only HEAD | grep '\.java$')
if [ -n "$MODIFIED" ]; then
  mvn checkstyle:check -Dcheckstyle.includes="$MODIFIED" -q
fi

# Verificar que não há import * (wildcard)
if grep -r "^import .*\*;" src/main/java/ --include="*.java" -l 2>/dev/null; then
  echo "ERRO: Imports wildcard detectados. Use imports específicos."
  exit 1
fi

echo "=== Validação OK ==="
```

---

### 9.5 Harness para Projetos Java/Spring Boot

Exemplo completo e pronto para uso:

```markdown
<!-- CLAUDE.md para projeto Spring Boot 3.x -->
# [Nome do Projeto] — Guia para o Agente IA

## Stack
- **Linguagem:** Java 21 (use var, records, sealed classes quando adequado)
- **Framework:** Spring Boot 3.4.x
- **Build:** Maven 3.9 (não Gradle)
- **Banco:** PostgreSQL 16 (prod), H2 em memória (testes)
- **Migrations:** Flyway (pasta `src/main/resources/db/migration/`)
- **Testes:** JUnit 5, Mockito, Spring Boot Test, Testcontainers

## Estrutura de Pacotes
```
com.empresa.app
├── config/         @Configuration, @Bean, WebMvcConfigurer, SecurityConfig
├── controller/     @RestController — HTTP puro, zero lógica
├── service/        @Service — lógica de negócio, @Transactional aqui
├── repository/     @Repository, JpaRepository, @Query JPQL
├── entity/         @Entity, sem lógica, sem DTOs
├── dto/            Records Java (request/response separados)
├── mapper/         MapStruct @Mapper para entity↔dto
├── exception/      BusinessException, GlobalExceptionHandler (@ControllerAdvice)
└── security/       JWT filter, UserDetailsService
```

## Convenções OBRIGATÓRIAS
1. DTOs: sempre `record NomeDTO(...)` — nunca classes mutáveis
2. Injeção: sempre construtor + `@RequiredArgsConstructor` (Lombok) — nunca `@Autowired`
3. Entidades: nunca expor diretamente no controller — sempre mapear para DTO
4. Exceções: lançar `BusinessException` (domínio) ou Spring exceptions (infra)
5. Migrations: nomear `V{numero}__{descricao_snake_case}.sql`, sequencial
6. Testes: `deveria_{resultado}_quando_{condicao}` (ex: `deveRetornar404_quandoProdutoNaoEncontrado`)

## Anti-padrões (NUNCA fazer)
- `@Autowired` em campo
- Lógica de negócio em `@RestController`
- Retornar `Entity` diretamente no controller
- `Optional.get()` sem verificação prévia
- `System.out.println` — use `log.info()`
- `catch (Exception e) { e.printStackTrace(); }` — relançar ou logar com logger

## Comandos Úteis
```bash
mvn spring-boot:run                 # rodar local
mvn test                            # testes unitários
mvn verify                          # build + todos os testes
mvn checkstyle:check                # verificar estilo
mvn dependency:tree                 # árvore de dependências
```

## Variáveis de Ambiente (dev local)
```
DATABASE_URL=jdbc:postgresql://localhost:5432/meubanco
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
JWT_SECRET=dev-secret-nao-usar-em-producao
```
```

---

## 10. Técnicas Avançadas de Prompting

### 10.1 Princípios Gerais

**Seja específico sobre o escopo:**
```
Ruim:  "Melhore o código"
Bom:   "Refatore apenas o método calcularFrete em FreteService.java para
        eliminar o if/else encadeado usando o padrão Strategy"
```

**Forneça contexto de restrição:**
```
Ruim:  "Adicione cache"
Bom:   "Adicione cache em memória (Caffeine, já no pom.xml) no método
        buscarProdutoPorId do CatalogoService. TTL de 5 minutos,
        tamanho máximo de 1000 entradas. Não use Redis — indisponível em dev."
```

**Especifique o formato de saída:**
```
"Responda apenas com o código Java sem explicações.
Não inclua imports — já vou organizar com o IDE."
```

**Use delimitadores para isolar código:**
```
Analise o seguinte código e diga o que ele faz:
```java
[cole o código aqui]
```
Responda em português, máximo 3 parágrafos.
```

---

### 10.2 Chain of Thought (CoT)

Solicitar que a IA *raciocine passo a passo* antes de responder melhora a qualidade em tarefas complexas:

```
"Antes de gerar o código, descreva:
1. Quais classes e interfaces precisarão ser criadas ou modificadas
2. Qual a sequência de execução do fluxo principal
3. Quais casos de erro precisam ser tratados
4. Que testes de unidade serão necessários

Depois, implemente o código."
```

Isso força a IA a planejar antes de executar, reduzindo erros de lógica.

---

### 10.3 Few-Shot Prompting

Fornecer exemplos do padrão esperado antes do pedido:

```
Aqui está como criamos DTOs neste projeto:

// Exemplo existente:
public record ClienteResponseDTO(
    UUID id,
    String nome,
    String email,
    boolean ativo,
    LocalDateTime criadoEm
) {}

Seguindo exatamente este padrão (mesmo estilo de Record, mesma organização
de campos), crie o FornecedorResponseDTO com os campos:
id (UUID), razaoSocial, cnpj, email, telefone, ativo, criadoEm.
```

---

### 10.4 Prompts para Geração de Código

**Template de prompt para nova feature:**
```
## Contexto
[Descrever o que já existe e o que será adicionado]

## Tarefa
[O que precisa ser criado — nomes, campos, comportamento]

## Restrições
- [Linguagem/framework/versão]
- [O que não pode ser usado]
- [Arquivos que não devem ser modificados]

## Saída esperada
- [Lista de arquivos que serão criados/modificados]
- [Testes incluídos? Sim/Não]
- [Documentação incluída? Javadoc? Sim/Não]
```

**Exemplo concreto:**
```
## Contexto
Projeto Spring Boot 3.4 com módulo de clientes já implementado.
A estrutura segue o CLAUDE.md deste repositório.

## Tarefa
Implementar módulo de Produtos com:
- Entidade: id (UUID), nome, descricao, preco (BigDecimal), estoque (int),
  categoria (enum: ELETRONICO, VESTUARIO, ALIMENTO), ativo (boolean)
- CRUD completo via REST em /api/produtos
- Busca por categoria: GET /api/produtos?categoria=ELETRONICO
- Apenas produtos ativos visíveis para clientes (role CUSTOMER)
- Admin pode ver todos: GET /api/admin/produtos

## Restrições
- Seguir todas as convenções do CLAUDE.md
- Não modificar arquivos existentes do módulo de clientes
- Migration Flyway numerada como V3 (V1 e V2 já existem)

## Saída esperada
- Produto.java (entity), ProdutoRequestDTO.java, ProdutoResponseDTO.java
- ProdutoRepository.java, ProdutoService.java, ProdutoController.java
- ProdutoAdminController.java (endpoints de admin)
- V3__cria_tabela_produto.sql
- ProdutoServiceTest.java (testes unitários do Service)
```

---

### 10.5 Prompts para Revisão e Refatoração

**Code review focado:**
```
Revise o arquivo PedidoService.java com foco em:
1. Violações dos princípios SOLID (especialmente SRP e OCP)
2. Métodos com mais de 20 linhas que poderiam ser extraídos
3. Tratamento de exceções adequado (sem swallowing)
4. Possíveis NPEs não tratados
5. Queries N+1 em loops com chamadas ao repository

Para cada problema encontrado: cite a linha, explique o problema e
forneça o código refatorado. Se não encontrar problemas em alguma categoria,
diga explicitamente.
```

**Extração de complexidade:**
```
O método 'processarCheckout' em CheckoutService.java tem 150 linhas.
Extraia responsabilidades em métodos privados, seguindo os princípios:
- Cada método faz exatamente uma coisa
- Nomes de métodos descrevem o que fazem (verbos)
- Não crie classes auxiliares adicionais — apenas refatore dentro da classe
O comportamento externo deve ser idêntico — os testes existentes devem passar.
```

---

### 10.6 Prompts para Testes

**Geração de testes abrangentes:**
```
Para a classe DescontoService.java, escreva testes JUnit 5 com Mockito que:

1. Cubram 100% dos caminhos do método `calcularDesconto`
2. Usem @ExtendWith(MockitoExtension.class)
3. Nomeiem métodos como `deveria_{resultado}_quando_{condicao}`
4. Incluam casos:
   - Cliente VIP com compra acima do mínimo → desconto de 15%
   - Cliente comum → desconto de 5%
   - Compra abaixo do valor mínimo → sem desconto
   - Valor null → IllegalArgumentException
   - Cliente null → IllegalArgumentException
5. Não usem reflection nem PowerMock — se precisar, refatore o código
```

**Testes de integração:**
```
Crie testes de integração para ProdutoController usando:
- @SpringBootTest(webEnvironment = RANDOM_PORT)
- TestRestTemplate
- Banco H2 em memória (já configurado em application-test.yml)
- @Sql para popular dados de teste

Cubra os endpoints: GET /api/produtos, GET /api/produtos/{id},
POST /api/produtos (admin), DELETE /api/produtos/{id} (admin).
Inclua teste de autenticação: endpoint sem token → 401.
```

---

### 10.7 Prompts para Documentação

**Javadoc cirúrgico:**
```
Adicione Javadoc apenas nos métodos públicos das interfaces de serviço
(src/main/java/**/service/*Service.java). Para cada método, documente:
- O que o método faz (uma linha)
- @param com descrição de cada parâmetro
- @return descrevendo o que é retornado
- @throws listando as exceções esperadas e quando são lançadas
Não adicione comentários óbvios (@param nome O nome). Seja informativo.
```

**Documentação de arquitetura:**
```
Analise o projeto e gere um documento ADR (Architecture Decision Record)
para cada uma das seguintes decisões que você identificar no código:
- Escolha do banco de dados
- Estratégia de autenticação (JWT vs sessão)
- Padrão de tratamento de exceções
- Estratégia de migrations

Formato de cada ADR:
## ADR-{n}: {Título}
**Status:** Accepted
**Contexto:** [problema que motivou a decisão]
**Decisão:** [o que foi decidido]
**Consequências:** [trade-offs positivos e negativos]
```

---

## 11. Fluxos de Trabalho para Projetos Reais

### 11.1 Iniciando um Projeto do Zero

```
Passo 1 — Definir o harness antes de escrever qualquer código

"Antes de começar, vamos definir o projeto. Preciso de uma API de gestão
de reservas de sala. Ajude-me a criar:
1. O CLAUDE.md com stack, convenções e estrutura de pacotes
2. O settings.json com permissões adequadas para Maven
3. Uma especificação inicial para os primeiros endpoints

Stack escolhida: Java 21, Spring Boot 3.4, PostgreSQL, JWT auth."
```

```
Passo 2 — Scaffolding inicial com Spring Initializr

"Use o Spring Initializr (via curl) para criar o projeto base com as
dependências: spring-boot-starter-web, spring-boot-starter-data-jpa,
spring-boot-starter-security, spring-boot-starter-validation,
postgresql, flyway-core, lombok, mapstruct, spring-boot-starter-test."
```

```
Passo 3 — Implementação iterativa por feature

"Implemente a feature de autenticação primeiro, pois as demais dependem dela.
Spec: POST /api/auth/login e POST /api/auth/refresh. Siga o CLAUDE.md."
```

---

### 11.2 Code Review Assistido por IA

```
Fluxo recomendado (Claude Code):

1. "Faça um diff do que mudou nesta branch: git diff main...HEAD"
2. "Para cada arquivo alterado, identifique:
   - Violações de convenção do CLAUDE.md
   - Problemas de segurança (OWASP Top 10)
   - Problemas de performance óbvios
   - Testes ausentes para código novo"
3. Revisar os problemas apontados manualmente
4. "Corrija apenas os problemas de alta prioridade que você identificou"
```

**Prompt para revisão completa de PR:**
```
Revise o PR atual (git diff main...HEAD) como um engenheiro sênior.
Estruture a revisão em:

## Problemas Bloqueadores (impedem merge)
## Problemas de Alta Prioridade (devem ser corrigidos)
## Sugestões (melhorias não obrigatórias)
## Pontos Positivos (boas práticas identificadas)

Para cada problema, cite arquivo:linha e forneça solução concreta.
```

---

### 11.3 Debugging com IA

**Estrutura de prompt para debugging:**
```
## Sintoma
[O que está acontecendo de errado]

## Stack trace / Log
[Cole o erro completo aqui]

## Contexto
[O que foi alterado recentemente / quando começou a acontecer]

## Código relevante
[Arquivo e método onde o erro ocorre]

## O que já tentei
[Para não repetir abordagens já descartadas]
```

**Exemplo:**
```
## Sintoma
TransactionSystemException ao salvar Pedido com itens

## Stack trace
org.springframework.transaction.TransactionSystemException: Could not commit JPA transaction
Caused by: javax.persistence.RollbackException: Error while committing the transaction
Caused by: org.hibernate.exception.ConstraintViolationException
  at PedidoService.criarPedido(PedidoService.java:67)

## Contexto
Erro começou após adicionar a constraint NOT NULL em pedido_item.preco_unitario

## Código relevante
[cole PedidoService.criarPedido e PedidoItem entity]

## O que já tentei
- Verificar que PedidoItemRequestDTO.precoUnitario tem @NotNull — tem
- Verificar a migration — está correta
```

---

### 11.4 Refatoração Guiada por IA

```
Estratégia para refatorações grandes:

1. "Analise o módulo de pagamentos e identifique os principais problemas
   de design. Não faça nada ainda — apenas liste."

2. [Revise e priorize a lista]

3. "Execute a refatoração #1 (extrair PaymentGatewayAdapter) sem alterar
   o comportamento. Garanta que mvn test continue passando após a mudança."

4. [Commit checkpoint: "refactor: extract PaymentGatewayAdapter"]

5. Repetir para cada item da lista
```

**Regra de ouro:** nunca peça para a IA fazer refatoração e adicionar feature na mesma sessão. Separe os commits.

---

## 12. Boas Práticas de Segurança com IA

### Riscos do Código Gerado por IA

```
Categoria de risco          Exemplo                        Frequência
─────────────────────────────────────────────────────────────────────
SQL Injection               Query construída por concatenação  Alta
Segredos hardcoded          JWT_SECRET = "abc123"              Alta
Falta de autorização        Admin endpoint sem @PreAuthorize   Média
Dependências vulneráveis    Library obsoleta inserida pelo LLM Alta
Logging de dados sensíveis  log.info("Password: {}", pwd)      Média
CORS permissivo             allowedOrigins("*") em produção    Alta
```

### Checklist de Segurança Pré-Merge

```markdown
## Revisão de segurança — código gerado por IA

### Injeções
- [ ] Queries usam parâmetros nomeados (`:param`) ou `@Param`, não concatenação
- [ ] Inputs de usuário não são interpolados em strings de query/command
- [ ] Deserialização de JSON não usa tipos polimórficos sem restrição

### Autenticação e Autorização
- [ ] Todos os endpoints têm anotação de segurança explícita
- [ ] JWT: secret não está hardcoded, expiração validada, algoritmo seguro
- [ ] Senhas são armazenadas com BCrypt (fator >= 10)

### Dados Sensíveis
- [ ] Nenhum dado sensível em logs (senhas, CPF, tokens)
- [ ] DTOs de resposta não incluem campos sensíveis desnecessários

### Dependências
- [ ] Verificar novas dependências adicionadas pela IA no pom.xml
- [ ] Executar `mvn dependency-check:check` para CVEs conhecidos

### Segredos
- [ ] Nenhuma chave, senha ou segredo hardcoded no código
- [ ] Variáveis de ambiente são usadas para configurações sensíveis
```

### Boas Práticas

**Nunca inclua segredos reais nos prompts.** Use placeholders:
```
"Configure o datasource para: host=localhost, db=pedidos,
user=admin, password=SUBSTITUA_POR_ENV_VAR"
```

**Sempre revise dependências inseridas automaticamente.**
A IA pode adicionar bibliotecas desatualizadas ou com vulnerabilidades conhecidas.

**Adicione revisão de segurança como step do harness:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "grep -rn 'password\\|secret\\|api_key\\|token' --include='*.java' -i $(git diff --name-only HEAD) | grep -v 'test\\|spec' | grep -v '\\.properties' || true"
        }]
      }
    ]
  }
}
```

---

## 13. Comparativo das Ferramentas

| Critério                  | Claude Code         | Gemini CLI          | Codex CLI (OpenAI)  | GitHub Copilot      |
|---------------------------|---------------------|---------------------|---------------------|---------------------|
| **Contexto máximo**       | 200K tokens         | 1M tokens           | 200K tokens         | ~100K (workspace)   |
| **Acesso ao filesystem**  | Sim (nativo)        | Sim (nativo)        | Sim (nativo)        | Leitura via IDE     |
| **Execução de comandos**  | Sim                 | Sim                 | Sim                 | Não                 |
| **Suporte a MCP**         | Sim (nativo)        | Sim (nativo)        | Não (ainda)         | Não                 |
| **Hooks/Automação**       | Sim (avançado)      | Limitado            | Não                 | Não                 |
| **Grounding web**         | Via MCP             | Nativo (Google)     | Via tools           | Não                 |
| **Multimodal (imagens)**  | Sim                 | Sim                 | Sim                 | Limitado            |
| **Integração IDE**        | Via extensão        | Via extensão        | Não                 | Sim (principal)     |
| **Arquivo de instrução**  | CLAUDE.md           | GEMINI.md           | AGENTS.md           | copilot-instructions|
| **Raciocínio avançado**   | Alto (Opus 4)       | Alto (2.5 Pro)      | Alto (o1/o3)        | Médio               |
| **Custo por token**       | Médio               | Baixo/Médio         | Alto (o1/o3)        | Assinatura fixa     |
| **Melhor para**           | Projetos complexos, harness engineering | Repositórios grandes, análise holística | Problemas de raciocínio difícil | Completions no IDE, produtividade diária |

### Recomendações de uso combinado

```
Fluxo diário de desenvolvimento

IDE         → GitHub Copilot: completions e refactoring inline
Terminal    → Claude Code: features completas, debugging, análise
Exploração  → Gemini CLI: análise de repositório grande, contexto amplo
Problemas   → ChatGPT o1: algoritmos complexos, debugging de concorrência
             difíceis
```

---

## 14. Checklist para Projetos com IA

### Configuração Inicial
- [ ] `CLAUDE.md` criado com stack, convenções e anti-padrões
- [ ] `GEMINI.md` criado (se usando Gemini CLI)
- [ ] `.github/copilot-instructions.md` criado (se usando Copilot)
- [ ] `settings.json` com permissões e hooks configurados
- [ ] Servidores MCP necessários identificados e configurados
- [ ] Scripts de validação para hooks implementados

### Por Sessão de Trabalho
- [ ] Descrever a tarefa com escopo explícito
- [ ] Usar `/clear` ao mudar de contexto não relacionado
- [ ] Revisar `git diff` antes de aceitar mudanças grandes
- [ ] Commitar em checkpoints (antes de refatorações grandes)
- [ ] Verificar dependências novas adicionadas

### Por Feature Implementada
- [ ] Especificação escrita (ou ao menos revisada) antes do código
- [ ] Testes gerados e executados com sucesso
- [ ] Code review de segurança realizado
- [ ] Nenhum segredo hardcoded introduzido
- [ ] Convenções do `CLAUDE.md` respeitadas
- [ ] Documentação pública (Javadoc) atualizada se API pública

### Periodicidade
- [ ] Atualizar `CLAUDE.md` quando convenções mudarem
- [ ] Revisar e purgar `mcpServers` desnecessários
- [ ] Executar análise de vulnerabilidades em dependências
- [ ] Revisar logs de sessão para identificar padrões de correção frequentes
  → Incorporar esses padrões no `CLAUDE.md` como anti-padrões

---

## 15. IDEs com IA de Primeira Classe

### 15.1 Cursor

**Cursor** é um editor de código derivado do VS Code que foi reconstruído com IA como elemento central, não como extensão. A diferença fundamental: enquanto o VS Code hospeda plugins de IA, o Cursor integra o modelo diretamente na lógica de edição, permitindo que a IA entenda o projeto de forma holística.

**Recursos principais:**

```
Composer (Cmd/Ctrl+I)
  └─ Edição multi-arquivo guiada por linguagem natural
  └─ Mostra diff de todas as mudanças antes de aplicar
  └─ Suporta "apply" parcial ou total

Agent Mode
  └─ Executa tarefas autônomas: lê, edita, roda comandos
  └─ Semelhante ao Claude Code, mas dentro do editor

Codebase Indexing
  └─ Indexa o repositório localmente com embeddings
  └─ Permite @codebase para busca semântica
  └─ Atualização incremental (apenas mudanças recentes)

@-mentions no chat
  └─ @arquivo.java    → inclui o arquivo no contexto
  └─ @pasta/          → inclui toda a pasta
  └─ @web             → busca na internet
  └─ @docs            → consulta documentação externa configurada
  └─ @git             → inclui contexto de commits recentes
```

**Arquivo `.cursorrules` (equivalente ao CLAUDE.md):**

```
# .cursorrules
Você é um assistente de programação para este projeto Spring Boot 3.4.

## Stack
Java 21, Spring Boot 3.4, PostgreSQL 16, Flyway, JUnit 5, Mockito.

## Convenções
- DTOs: Java Records (nunca classes mutáveis)
- Injeção: sempre por construtor, nunca @Autowired em campo
- Exceptions: BusinessException para domínio, nunca swallowing
- Logs: SLF4J, nunca System.out
- Testes: nomenclatura deveria_{resultado}_quando_{condicao}

## Nunca fazer
- Lógica de negócio em @RestController
- Retornar @Entity diretamente em controllers
- Optional.get() sem verificação prévia
```

**Modelos suportados no Cursor:**
- Claude Sonnet / Opus (Anthropic)
- GPT-4o / o1 / o3 (OpenAI)
- Gemini 2.5 Pro (Google)
- Modelos locais via Ollama

---

### 15.2 Windsurf

**Windsurf** (anteriormente Codeium) é outro IDE AI-first com arquitetura centrada em agentes. O diferencial é o **Cascade** — um sistema agentic que mantém contexto do fluxo de trabalho em andamento, não apenas da conversa atual.

**Recursos principais:**

```
Cascade
  └─ Agente que acompanha o contexto de trabalho completo
  └─ Entende "o que você está tentando fazer" não só "o que pediu"
  └─ Deep Contextual Awareness (DCA): observa edições em tempo real
  └─ Pode propor próximos passos sem ser perguntado

Flow State
  └─ Histórico de ações do agente com explicações
  └─ Permite inspecionar cada decisão tomada
  └─ Reverter ações individualmente

Supercomplete
  └─ Completions contextuais mais longas (parágrafo inteiro)
  └─ Sugere trechos de múltiplas linhas baseado no contexto
```

**Diferença conceitual Cursor vs Windsurf:**

```
Cursor:   você guia → IA executa (mais controle explícito)
Windsurf: IA observa → propõe → você aprova (mais proativo)
```

---

### 15.3 GitHub Copilot Workspace

**GitHub Copilot Workspace** é um ambiente web da GitHub onde o ciclo completo de uma feature — de issue a código — acontece dentro do navegador, sem necessitar do IDE local.

**Fluxo de trabalho:**

```
1. Abrir uma issue no GitHub
   └─ "Adicionar campo 'desconto' ao modelo de Pedido"

2. Copilot Workspace gera um PLANO
   └─ Lista de arquivos a criar/modificar
   └─ Descrição das mudanças em cada arquivo

3. Revisar e ajustar o plano (sem código ainda)
   └─ Aceitar, rejeitar ou editar itens do plano

4. Copilot implementa o plano aprovado
   └─ Diff completo antes de commitar

5. Criar PR diretamente da interface
   └─ Descrição gerada automaticamente a partir do plano
```

**Quando é útil:**
- Issues bem definidas que podem ser resolvidas sem contexto local
- Times distribuídos sem acesso ao ambiente de dev
- Contribuidores externos em projetos open source
- Revisão de PRs com contexto enriquecido

---

### 15.4 Comparativo IDEs com IA

| Critério                   | Cursor        | Windsurf      | VS Code + Copilot | VS Code + Claude |
|----------------------------|---------------|---------------|-------------------|------------------|
| **Modelo base do editor**  | VS Code fork  | VS Code fork  | VS Code           | VS Code          |
| **Agente autônomo**        | Sim (Agent)   | Sim (Cascade) | Limitado          | Via extensão     |
| **Indexação do codebase**  | Sim (local)   | Sim (local)   | Workspace parcial | Não (via MCP)    |
| **Multi-arquivo (diff)**   | Sim           | Sim           | Limitado          | Via extensão     |
| **Escolha de modelos**     | Múltiplos     | Múltiplos     | Copilot only      | Claude only      |
| **@-mentions**             | Avançado      | Básico        | Básico            | Via chat         |
| **Plano antes de executar**| Composer      | Cascade       | Não               | Sim (Claude)     |
| **Preço (2025)**           | $20/mês       | $15/mês       | $10/mês (Copilot) | Por token        |
| **Melhor para**            | Equipes que querem controle | Desenvolvedores que querem proatividade da IA | Usuários VS Code com Copilot | Projetos com harness detalhado |

---

## 16. Gerenciamento de Custo e Uso de Tokens

### 16.1 Entendendo Tokens e Custo

**Token** é a unidade básica de texto processada pelos LLMs. Aproximadamente:

```
1 token ≈ 4 caracteres em inglês
1 token ≈ 3 caracteres em português
1 token ≈ 0,75 palavras

Exemplos de contagem:
"Hello, world!"            →  4 tokens
"Olá, mundo!"              →  5 tokens
Arquivo Java com 200 linhas → ~2.000–4.000 tokens
CLAUDE.md com 100 linhas   → ~800–1.500 tokens
```

**Preços de referência (Anthropic, 2025):**

```
Modelo               Input (por MTok)   Output (por MTok)
────────────────────────────────────────────────────────
Claude Opus 4.7      $15,00             $75,00
Claude Sonnet 4.6    $3,00              $15,00
Claude Haiku 4.5     $0,25              $1,25

MTok = 1 milhão de tokens
```

**Custo típico por tipo de sessão:**

```
Pergunta simples (1 turno)
  Input:  ~2.000 tokens (CLAUDE.md + pergunta)
  Output: ~500 tokens
  Custo Sonnet: ~$0,014  (menos de 2 centavos)

Feature completa (10 turnos)
  Input:  ~50.000 tokens acumulados
  Output: ~10.000 tokens
  Custo Sonnet: ~$0,30  (30 centavos)

Análise de repositório grande (1 turno)
  Input:  ~200.000 tokens (repositório inteiro)
  Output: ~5.000 tokens
  Custo Sonnet: ~$0,675  (67 centavos)
```

---

### 16.2 Prompt Caching

**Prompt caching** é uma funcionalidade da Anthropic (e OpenAI) que armazena prefixos longos repetidos — como o conteúdo do `CLAUDE.md` e o código-base — com desconto de 90% no custo de input.

```
Sem cache:
Turno 1: CLAUDE.md (1.000 tokens) + pergunta (100 tokens) = 1.100 tokens cobrados
Turno 2: CLAUDE.md (1.000 tokens) + história (500 tokens) + pergunta = 1.600 tokens

Com cache (após 1º turno):
Turno 1: 1.100 tokens (sem cache ainda)
Turno 2: 1.000 tokens com 90% desconto + 600 tokens normais = ~700 tokens equivalentes
```

**Regras do cache Anthropic:**
- Cache dura **5 minutos** sem uso; **1 hora** com uso contínuo
- Mínimo de tokens para cachear: 1.024 (Haiku) ou 2.048 (Sonnet/Opus)
- O prefixo cacheado deve ser idêntico byte a byte

**Como o Claude Code usa caching automaticamente:**
O Claude Code marca automaticamente para cache: conteúdo do `CLAUDE.md`, ferramentas do sistema, e a conversa até um ponto estável. Sessões longas com o mesmo arquivo base se beneficiam muito.

**Dica prática:** mantenha o `CLAUDE.md` no início da sessão e não o modifique durante uma sessão longa — isso preserva o cache.

---

### 16.3 Escolha de Modelo por Tipo de Tarefa

```
Tarefa                              Modelo recomendado    Justificativa
──────────────────────────────────────────────────────────────────────
Geração de scaffolding/boilerplate  Sonnet / Flash        Rápido, barato, suficiente
Refatoração simples                 Sonnet / Flash        Adequado
Debugging de stack trace claro      Sonnet                Contexto moderado
Feature completa com spec           Sonnet / Opus         Qualidade de raciocínio
Algoritmo complexo / concorrência   Opus / o1/o3          Raciocínio profundo necessário
Code review de segurança            Sonnet / Opus         Atenção a detalhes sutis
Explicação de código legado         Sonnet                Custo justificado
Renomear variável / formatar JSON   Haiku / Flash         Tarefa trivial
Geração de testes unitários         Sonnet                Cobertura de casos de borda
Análise de repositório inteiro      Gemini 2.5 Pro        Maior context window (1M)
```

**Estratégia de escalonamento:**

```python
# Padrão: começar com modelo barato e escalar se necessário

def escolher_modelo(complexidade: str) -> str:
    match complexidade:
        case "trivial":   return "claude-haiku-4-5"
        case "normal":    return "claude-sonnet-4-6"
        case "complexo":  return "claude-opus-4-7"
        case "raciocínio": return "o1"  # OpenAI para problemas NP-difíceis
```

---

### 16.4 Otimizando o CLAUDE.md para Caching

O `CLAUDE.md` é lido em toda sessão — o conteúdo estável no topo maximiza cache hits:

```markdown
<!-- Estrutura otimizada para caching -->

# Projeto XYZ                          ← estável, sempre no topo

## Stack (não muda)                    ← cacheável
Java 21, Spring Boot 3.4 ...

## Estrutura de Pacotes (raramente muda) ← cacheável
com.empresa.app
├── controller/ ...

## Convenções (raramente muda)         ← cacheável
...

## Contexto de Domínio (raramente muda) ← cacheável
...

## Status Atual do Projeto             ← NÃO colocar aqui — muda frequentemente
                                       ← Informações voláteis destroem o cache
```

**Regra:** informações que mudam frequentemente (sprint atual, bugs em aberto, tarefas pendentes) devem ir no **prompt**, não no `CLAUDE.md`.

---

## 17. Workflows Multi-Agente

### 17.1 Conceito e Motivação

Um único agente em sequência tem limitações:
- Context window se esgota em projetos grandes
- Um agente não consegue revisar seu próprio output de forma imparcial
- Tarefas paralelas e independentes esperariam em fila

**Multi-agente** divide o trabalho entre instâncias especializadas que cooperam:

```
Orquestrador
├── Agente Pesquisador   → analisa o código, levanta requisitos
├── Agente Codificador   → implementa a feature
├── Agente Testador      → gera e executa testes
└── Agente Revisor       → revisa segurança e convenções
```

---

### 17.2 Padrão ReAct

**ReAct** (Reason + Act) é o padrão fundamental dos agentes de coding: o modelo alterna entre **raciocínio** e **ação** até completar a tarefa.

```
Tarefa: "Adicione paginação ao endpoint GET /api/produtos"

[THOUGHT] Preciso entender como o endpoint atual funciona.
[ACTION]  read_file("ProdutoController.java")
[RESULT]  [conteúdo do arquivo]

[THOUGHT] O endpoint retorna List<ProdutoResponseDTO>. Para paginação,
          preciso mudar para Page<ProdutoResponseDTO> e aceitar
          parâmetros page e size. O Service também precisa mudar.
[ACTION]  read_file("ProdutoService.java")
[RESULT]  [conteúdo do arquivo]

[THOUGHT] O Service usa findAll(). Vou usar findAll(Pageable).
[ACTION]  edit_file("ProdutoController.java", ...)
[ACTION]  edit_file("ProdutoService.java", ...)

[THOUGHT] Devo rodar os testes para confirmar que nada quebrou.
[ACTION]  bash("mvn test -pl . -Dtest=ProdutoControllerTest")
[RESULT]  BUILD SUCCESS

[THOUGHT] Pronto. Tarefa concluída.
```

Claude Code, Gemini CLI e Codex CLI implementam ReAct nativamente.

---

### 17.3 Padrão Plan-and-Execute

**Plan-and-Execute** separa o planejamento da execução em duas fases distintas, com revisão humana no meio:

```
FASE 1 — PLANEJAMENTO (sem escrever código)
"Antes de implementar qualquer coisa, analise o projeto e liste:
1. Todos os arquivos que precisarão ser criados ou modificados
2. Para cada arquivo, o que exatamente mudará
3. A ordem de implementação com justificativa
4. Riscos identificados e como mitigá-los
Aguarde minha aprovação antes de prosseguir."

── Desenvolvedor revisa e aprova/ajusta o plano ──

FASE 2 — EXECUÇÃO (implementação do plano aprovado)
"O plano foi aprovado. Execute o item 1, depois 2, etc.
Após cada item, mostre o diff e aguarde confirmação."
```

**Quando usar Plan-and-Execute:**
- Features que tocam mais de 10 arquivos
- Refatorações estruturais (mudar arquitetura de pacotes)
- Migrações de versão
- Qualquer mudança com risco de regressão em produção

---

### 17.4 Subagentes no Claude Code

O Claude Code suporta internamente o uso de **subagentes** para tarefas paralelas. Na prática, isso aparece quando o Claude lança `Task()` calls em paralelo:

```
Tarefa: "Analise todos os controllers e gere relatório de segurança"

Claude (orquestrador) detecta 8 controllers e lança 8 subagentes em paralelo:
├── SubAgent 1: analisa ProdutoController.java
├── SubAgent 2: analisa ClienteController.java
├── SubAgent 3: analisa PedidoController.java
...
└── SubAgent 8: analisa AuthController.java

Orquestrador: consolida resultados → gera relatório final
```

Para **forçar** esse comportamento via prompt:
```
"Analise os seguintes 5 endpoints de forma paralela e independente,
depois consolide os resultados:
1. GET /api/produtos
2. POST /api/pedidos
3. DELETE /api/clientes/{id}
4. PUT /api/estoque/{id}
5. GET /api/relatorios/vendas"
```

---

### 17.5 Orquestração com a API Anthropic

Para automações customizadas, é possível orquestrar agentes programaticamente usando a API:

```python
import anthropic

client = anthropic.Anthropic()

def agente_pesquisador(codigo: str) -> str:
    """Analisa o código e levanta problemas."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system="Você é um analista de código. Identifique problemas de design, "
               "segurança e performance. Seja objetivo e cite linha:coluna.",
        messages=[{"role": "user", "content": f"Analise:\n```java\n{codigo}\n```"}]
    )
    return response.content[0].text

def agente_codificador(codigo: str, problemas: str) -> str:
    """Corrige os problemas identificados."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=8192,
        system="Você é um engenheiro Java sênior. Corrija os problemas fornecidos "
               "sem alterar a lógica de negócio. Responda apenas com o código corrigido.",
        messages=[{
            "role": "user",
            "content": f"Código original:\n```java\n{codigo}\n```\n\nProblemas:\n{problemas}"
        }]
    )
    return response.content[0].text

def agente_revisor(codigo_original: str, codigo_corrigido: str) -> str:
    """Verifica se as correções são adequadas."""
    response = client.messages.create(
        model="claude-opus-4-7",  # modelo mais capaz para revisão final
        max_tokens=2048,
        system="Você é um revisor de código sênior. Verifique se as mudanças "
               "corrigem os problemas sem introduzir novos. Responda: APROVADO ou REJEITADO + justificativa.",
        messages=[{
            "role": "user",
            "content": f"Original:\n```java\n{codigo_original}\n```\n\n"
                       f"Corrigido:\n```java\n{codigo_corrigido}\n```"
        }]
    )
    return response.content[0].text

# Pipeline multi-agente
with open("src/main/java/PedidoService.java") as f:
    codigo = f.read()

problemas   = agente_pesquisador(codigo)
corrigido   = agente_codificador(codigo, problemas)
aprovacao   = agente_revisor(codigo, corrigido)

if "APROVADO" in aprovacao:
    with open("src/main/java/PedidoService.java", "w") as f:
        f.write(corrigido)
    print("Código atualizado com sucesso.")
else:
    print(f"Revisão rejeitada: {aprovacao}")
```

---

## 18. IA para Modernização de Código Legado

### 18.1 Estratégia de Abordagem

Código legado apresenta desafios específicos para IA: lógica implícita não documentada, dependências ocultas, comportamentos que "não podem mudar mas ninguém sabe por quê". A estratégia correta é incremental e conservadora:

```
Abordagem SEGURA para legado com IA

1. ENTENDER antes de mudar
   "Explique o que este código faz, quais regras de negócio implementa
    e por que cada decisão de design pode ter sido tomada."

2. MAPEAR dependências
   "Liste todos os callers deste método e o que eles esperam receber."

3. ESCREVER TESTES antes de refatorar (Characterization Tests)
   "Escreva testes que documentam o comportamento ATUAL, mesmo que seja
    'errado'. Não mude a lógica — apenas capture o que acontece hoje."

4. REFATORAR com testes como rede de segurança
   "Refatore este método para usar streams Java 8+. Os testes que
    acabamos de criar devem continuar passando — eles são o contrato."

5. MODERNIZAR gradualmente
   "Agora sim, sugira modernizações para Java 21."
```

---

### 18.2 Migrações Java (8 → 21)

**Prompt de migração Java 8 → 21:**

```
Analise os arquivos Java em src/main/java/ e identifique oportunidades de
modernização de Java 8 para Java 21. Para cada categoria abaixo, liste os
arquivos e linhas afetados:

1. var (inferência de tipo local) — onde simplifica sem perder clareza
2. Records — classes com apenas getters, equals/hashCode, toString
3. Sealed classes — hierarquias de tipo fechadas
4. Pattern matching (instanceof) — substituição de casts explícitos
5. switch expressions — switch que retorna valor
6. Text blocks — strings multi-linha com """ 
7. Stream API e Optional — substituição de loops imperativos simples
8. Structured Concurrency / Virtual Threads — código com threads explícitas

Para cada caso: mostre o código atual e o código modernizado.
Priorize mudanças de MAIOR impacto em clareza e brevidade.
```

**Exemplo de transformação:**

```java
// Java 8 (legado)
public String processar(Object obj) {
    if (obj instanceof PedidoEntity) {
        PedidoEntity pedido = (PedidoEntity) obj;
        return "Pedido: " + pedido.getNumero();
    } else if (obj instanceof ClienteEntity) {
        ClienteEntity cliente = (ClienteEntity) obj;
        return "Cliente: " + cliente.getNome();
    }
    return "Desconhecido";
}

// Java 21 com pattern matching + switch expression
public String processar(Object obj) {
    return switch (obj) {
        case PedidoEntity  p -> "Pedido: "  + p.getNumero();
        case ClienteEntity c -> "Cliente: " + c.getNome();
        default              -> "Desconhecido";
    };
}
```

---

### 18.3 Migrações de Frameworks

**Spring Boot 2.x → 3.x:**

```
Analise o projeto Spring Boot 2.7 e produza um plano de migração para 3.4.

Para cada item abaixo, identifique os arquivos afetados e forneça o código migrado:

1. javax.* → jakarta.* (todos os imports)
2. WebSecurityConfigurerAdapter (removido) → SecurityFilterChain @Bean
3. spring.datasource.initialization-mode → spring.sql.init.mode
4. @EnableWebMvc (comportamento mudou) → verificar necessidade
5. HttpMethod.GET etc (mudanças de API menor)
6. spring-security (mudanças de configuração)
7. Dependências: versões compatíveis com Spring Boot 3.4

Proceda arquivo por arquivo com minha aprovação.
```

**Angular 14 → 19 (migração de módulos para standalone):**

```
Analise o projeto Angular 14 em src/ e planeje a migração para Angular 19.
Priorize:

1. NgModules → Standalone Components (execute ng generate @angular/core:standalone)
2. HttpClientModule → provideHttpClient() em app.config.ts
3. RouterModule → provideRouter() em app.config.ts
4. FormsModule → importar diretamente nos componentes
5. Signals: identificar onde @Input/@Output podem virar signal inputs/outputs
6. Atualizar angular.json e package.json para versões compatíveis

Comece pela migração automática com o ng migrate e depois ajuste manualmente
o que a ferramenta não conseguir.
```

---

### 18.4 Explicação de Código Sem Documentação

Prompts para decifrar código legado sem documentação:

**Entendimento inicial:**
```
Este código foi escrito em 2015 e não tem comentários. Analise o método
'calcularICMSST' em TributacaoService.java e responda:
1. O que este método calcula (em linguagem de negócio, não técnica)?
2. Quais são as entradas e o que cada parâmetro representa no domínio?
3. Quais regras tributárias brasileiras este código implementa?
4. O que acontece nos casos especiais (valores null, zero, negativos)?
5. Existem possíveis bugs ou comportamentos surpreendentes?
```

**Geração de documentação:**
```
Com base no código de TributacaoService.java que acabamos de analisar,
escreva:
1. Javadoc completo para a classe (propósito, contexto de uso)
2. Javadoc para cada método público (param, return, throws, regras aplicadas)
3. Um arquivo docs/tributacao-regras.md explicando as regras de negócio
   implementadas em linguagem acessível para a equipe de produto
```

---

## 19. IA no Ciclo de Git e CI/CD

### 19.1 Mensagens de Commit e PRs

**Mensagens de commit com IA (integrado ao harness):**

```bash
# scripts/commit-message.sh
# Uso: git diff --staged | bash scripts/commit-message.sh

DIFF=$(cat)
PROMPT="Analise o seguinte git diff e escreva uma mensagem de commit no padrão
Conventional Commits (feat/fix/refactor/docs/test/chore).

Regras:
- Primeira linha: tipo(escopo): descrição imperativa em português, máx 72 chars
- Pular linha
- Corpo (opcional): explicar O QUE mudou e POR QUÊ, não como
- Máx 3 bullet points se necessário

Responda APENAS com a mensagem de commit, nada mais.

Diff:
$DIFF"

echo "$PROMPT" | claude -p --output-format text
```

**Configurar como git hook:**

```bash
# .git/hooks/prepare-commit-msg
#!/bin/bash
# Só age se a mensagem estiver vazia (novo commit, não amend)
if [ -z "$(cat $1)" ]; then
  git diff --staged | bash scripts/commit-message.sh > $1
fi
```

**Descrições de PR automáticas:**

```bash
# Gerar descrição de PR a partir do diff com a branch main
git diff main...HEAD | claude -p "
Você é um engenheiro técnico. Analise este diff e gere uma descrição de PR em Markdown:

## Resumo
[2-3 bullets do que foi implementado]

## Motivação
[Por que esta mudança foi necessária]

## Mudanças Principais
[Arquivos e classes relevantes modificados]

## Como Testar
[Passos para verificar o funcionamento]

## Checklist
- [ ] Testes adicionados/atualizados
- [ ] Sem segredos hardcoded
- [ ] Documentação atualizada (se necessário)
"
```

---

### 19.2 Changelogs e Release Notes Automáticos

```bash
# Gerar CHANGELOG para uma release a partir dos commits
git log v1.2.0..HEAD --pretty=format:"%h %s" | claude -p "
Analise estes commits e gere um CHANGELOG no formato Keep a Changelog.
Agrupe por: Added, Changed, Fixed, Deprecated, Removed, Security.
Use linguagem voltada ao usuário final (não técnica), em português.
Versão: 1.3.0 — Data: $(date +%Y-%m-%d)
"
```

**Release notes voltadas ao produto (para stakeholders):**

```
Com base no CHANGELOG técnico abaixo, escreva release notes executivas:
- Voltadas para product managers e stakeholders não-técnicos
- Foco em VALOR entregue, não em implementação
- Tom positivo e claro
- Máximo 1 parágrafo por feature principal
- Inclua métricas de impacto se mencionadas nos commits

CHANGELOG:
[cole o changelog técnico]
```

---

### 19.3 Revisão de IA em Pipelines CI/CD

**GitHub Actions com revisão de segurança via IA:**

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get diff
        id: diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD > diff.txt
          echo "lines=$(wc -l < diff.txt)" >> $GITHUB_OUTPUT

      - name: AI Security Review
        if: steps.diff.outputs.lines < 2000  # limitar custo
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          REVIEW=$(cat diff.txt | npx claude-code-sdk review-security)
          echo "REVIEW<<EOF" >> $GITHUB_ENV
          echo "$REVIEW" >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Revisão de Segurança (IA)\n\n${process.env.REVIEW}`
            })
```

---

### 19.4 Gates de Qualidade com IA

Um **quality gate** de IA bloqueia o merge se a revisão identificar problemas críticos:

```yaml
# .github/workflows/ai-gate.yml
- name: AI Quality Gate
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: |
    RESULT=$(cat diff.txt | claude -p "
    Revise este diff. Responda APENAS com JSON no formato:
    {\"aprovado\": boolean, \"bloqueadores\": [\"...\"], \"avisos\": [\"...\"]}
    
    Bloquear se encontrar:
    - Segredos hardcoded (chaves, senhas, tokens)
    - SQL Injection (concatenação de input em query)
    - Endpoint sem autenticação onde deveria ter
    - Rollback de migration (DROP TABLE sem backup)
    ")
    
    APROVADO=$(echo $RESULT | jq '.aprovado')
    if [ "$APROVADO" != "true" ]; then
      echo "Gate BLOQUEADO:"
      echo $RESULT | jq '.bloqueadores[]'
      exit 1
    fi
    echo "Gate APROVADO"
```

---

## 20. Estratégias Avançadas de Contexto

### 20.1 Estrutura do Repositório para IA

A organização do repositório impacta diretamente a capacidade da IA de navegar e entender o projeto. Princípios para um repo "AI-friendly":

```
Convenções que ajudam a IA a navegar

1. Nomes descritivos de pacotes/pastas
   ✓ com.empresa.pedidos.service
   ✗ com.empresa.core.impl.svc

2. Uma responsabilidade por arquivo
   ✓ PedidoService.java (apenas lógica de pedidos)
   ✗ UtilsGeral.java (mistura de tudo)

3. Imports explícitos (sem wildcards)
   ✓ import java.util.List;
   ✗ import java.util.*;

4. README.md na raiz de cada módulo
   └─ módulo-pagamentos/
       ├── README.md  ← "Este módulo integra com gateway XYZ via..."
       └── src/...

5. CLAUDE.md com mapa de arquivos críticos
   "Arquivo mais importante: OrderProcessingService.java — orquestra o fluxo principal"
```

---

### 20.2 .claudeignore e .geminiignore

Assim como `.gitignore`, esses arquivos informam à IA o que **não** processar — reduzindo ruído e custo:

```gitignore
# .claudeignore
# Build e compilados
target/
build/
dist/
*.class
*.jar

# Dependências
node_modules/
.mvn/wrapper/

# Gerado automaticamente (não editar manualmente)
src/generated/
openapi-generated/
*.generated.java

# Dados e fixtures grandes
src/test/resources/fixtures/large/
**/*-dump.sql
**/*.csv

# Configurações sensíveis locais
.env.local
application-local.yml
secrets/

# Logs e temporários
logs/
*.log
*.tmp
```

---

### 20.3 Índices e Mapas de Código

Para projetos grandes, criar um índice manual ajuda a IA a navegar sem precisar ler tudo:

```markdown
<!-- docs/code-map.md — incluído no CLAUDE.md via referência -->
# Mapa do Código — Pontos de Entrada Principais

## Fluxo de Pedido (caminho crítico)
1. PedidoController.criarPedido()        ← ponto de entrada HTTP
2. PedidoService.processar()             ← orquestrador principal
3. EstoqueService.reservar()             ← chamado por #2
4. PagamentoGateway.cobrar()             ← chamado por #2
5. NotificacaoService.notificarCliente() ← chamado por #2 após sucesso

## Configurações de Segurança
SecurityConfig.java — toda configuração de autenticação/autorização
JwtFilter.java      — intercepta e valida tokens

## Integrações Externas
GatewayPagamentoClient.java    → API do Stripe (src/main/java/infra/pagamento/)
CorreiosClient.java            → CEP e frete (src/main/java/infra/logistica/)
EmailService.java              → SendGrid (src/main/java/infra/notificacao/)
```

---

### 20.4 Quando Usar RAG vs Contexto Direto

```
Decisão: incluir no contexto (CLAUDE.md/prompt) ou usar RAG (MCP/busca)?

Incluir diretamente quando:
✓ Arquivo pequeno (< 500 linhas) e sempre relevante
✓ Convenções e regras que se aplicam a tudo
✓ Schema do banco de dados principal
✓ Estrutura de pacotes e arquitetura geral

Usar RAG/MCP quando:
✓ Documentação extensa (> 10.000 tokens)
✓ Apenas partes são relevantes por tarefa
✓ Conteúdo muda frequentemente (issues, PRs abertos)
✓ Dados do banco de produção (nunca no prompt!)
✓ Histórico de commits (use git MCP server)

Exemplo com MCP Postgres em vez de dump no contexto:
```
"Consulte o schema atual da tabela 'pedidos' via MCP e crie a entidade JPA correspondente"
-- versus --
[colocar 500 linhas de schema SQL no prompt] ← desperdício de tokens
```
```

---

## 21. IA para Design de API e Documentação OpenAPI

### 21.1 Contract-First Development com IA

**Contract-first** significa escrever a especificação OpenAPI *antes* do código, usando a spec como contrato que a implementação deve satisfazer. A IA acelera cada fase desse ciclo:

```
Ciclo Contract-First com IA

Requisito (informal)
    ↓
[IA] Rascunho de spec OpenAPI (YAML)
    ↓
Revisão humana da spec
    ↓
[IA] Geração de código a partir da spec (stubs de controller)
    ↓
[IA] Geração de testes de contrato (Spring Cloud Contract / Pact)
    ↓
Implementação dos services
    ↓
[IA] Validação: implementação respeita a spec?
```

---

### 21.2 Geração de Spec OpenAPI a partir de Requisitos

```
Crie uma especificação OpenAPI 3.1 completa em YAML para uma API de
gestão de reservas de sala com os seguintes requisitos:

Regras de negócio:
- Salas têm capacidade máxima, localização (prédio/andar) e recursos (projetor, videoconferência)
- Reservas têm início, fim, organizador e lista de participantes (emails)
- Não pode haver sobreposição de horários na mesma sala
- Reservas só podem ser canceladas pelo organizador ou admin
- Duração máxima: 4 horas

Endpoints necessários:
- CRUD de salas (apenas admin)
- Buscar salas disponíveis em um intervalo de data/hora
- CRUD de reservas
- Listar reservas de uma sala (por período)
- Listar minhas reservas

Inclua na spec:
- Schemas de todos os DTOs de entrada e saída
- Exemplos realistas em cada endpoint
- Códigos de erro documentados (400, 401, 403, 404, 409)
- Tags para organização
- SecurityScheme com Bearer JWT
```

---

### 21.3 Geração de Código a partir de Spec

Com a spec pronta, gerar stubs de implementação:

```
Dado o arquivo openapi.yaml em docs/api/openapi.yaml, gere os stubs
de implementação Spring Boot 3.4:

1. Use o plugin openapi-generator-maven-plugin para gerar as interfaces
   dos controllers a partir da spec (adicione ao pom.xml)

2. Para cada interface gerada, crie uma classe concreta em
   src/main/java/com/empresa/controller/ implementando-a

3. Nos métodos dos controllers, adicione apenas:
   - @Valid nos parâmetros de body
   - Chamada ao Service correspondente (crie os Services como stubs)
   - Tratamento de erro com ResponseStatusException

Não implemente a lógica de negócio nos services — deixe como TODO.
```

---

### 21.4 Validação de Implementação contra Contrato

```
Verifique se a implementação atual está em conformidade com a spec OpenAPI
em docs/api/openapi.yaml.

Para cada endpoint da spec:
1. Localizar o método correspondente nos controllers
2. Verificar se os parâmetros de entrada correspondem à spec
3. Verificar se os tipos de retorno correspondem à spec
4. Verificar se os códigos de status HTTP estão corretos
5. Verificar se a validação (@Valid, @NotNull etc) está presente

Liste as divergências encontradas entre spec e implementação,
com arquivo:linha para cada uma.
```

---

## 22. Avaliação da Qualidade do Código Gerado

### 22.1 Reconhecendo Alucinações de Código

**Alucinação de código** é quando a IA gera código que parece correto mas referencia APIs, métodos ou classes que não existem (ou existiram em versões antigas).

**Sinais de alerta:**

```java
// Sinal 1: método que não existe na API
// A IA gerou chamada a método inexistente no Spring Security 6
authentication.getAuthorities().stream()
    .anyMatch(a -> a.hasRole("ADMIN"));  // ← hasRole() não existe em GrantedAuthority

// Correto:
authentication.getAuthorities().stream()
    .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));

// Sinal 2: import de classe inexistente
import org.springframework.security.core.AuthorityUtils.createAuthorityList; // ← método estático errado

// Sinal 3: anotação em versão errada
@RestController
@RequestMapping
public class Exemplo {
    @GetMapping(produces = MediaType.APPLICATION_JSON_UTF8_VALUE) // ← depreciado e removido no Spring 5.2
```

**Como verificar:**
```bash
# Após geração, compilar imediatamente
mvn compile 2>&1 | grep "error:" | head -20

# Verificar imports problemáticos
mvn dependency:resolve 2>&1 | grep "not found"
```

---

### 22.2 Métricas Objetivas de Qualidade

Usar ferramentas automáticas para medir a qualidade do código gerado objetivamente:

```bash
# Cobertura de testes (JaCoCo)
mvn test jacoco:report
open target/site/jacoco/index.html
# Meta: >= 80% de line coverage no código novo

# Complexidade ciclomática (PMD)
mvn pmd:check
# Alerta se qualquer método tiver complexidade > 10

# Code smells (SpotBugs)
mvn spotbugs:check

# Duplicação de código (CPD)
mvn pmd:cpd-check
# Alerta se blocos > 50 tokens forem duplicados

# Verificar que não há TODO/FIXME esquecidos
grep -r "TODO\|FIXME\|HACK\|XXX" src/main/java/ --include="*.java" | wc -l
```

**Integrando ao CLAUDE.md:**
```markdown
## Critérios de Aceitação de Código Gerado
- mvn compile: sem erros
- mvn test: todos os testes passam
- JaCoCo: cobertura >= 80% no código novo
- PMD: sem violations de prioridade HIGH
- SpotBugs: sem violations de prioridade HIGH
```

---

### 22.3 LLM-as-Judge

**LLM-as-Judge** usa uma instância separada do modelo para avaliar a qualidade do output de outra instância. É especialmente útil para aspectos qualitativos difíceis de medir com ferramentas estáticas.

```python
import anthropic

def avaliar_codigo(codigo_gerado: str, spec: str) -> dict:
    """Usa um LLM para avaliar a qualidade do código gerado por outro LLM."""
    client = anthropic.Anthropic()

    prompt = f"""Você é um engenheiro Java sênior fazendo code review.
    
Avalie o código abaixo em relação à especificação fornecida.
Responda em JSON com a estrutura:
{{
  "pontuacao": 1-10,
  "spec_atendida": true/false,
  "problemas_criticos": ["..."],
  "problemas_menores": ["..."],
  "sugestoes_melhoria": ["..."],
  "aprovado": true/false
}}

ESPECIFICAÇÃO:
{spec}

CÓDIGO GERADO:
{codigo_gerado}"""

    response = client.messages.create(
        model="claude-opus-4-7",   # modelo diferente/mais capaz para revisão
        max_tokens=2048,
        messages=[{"role": "user", "content": prompt}]
    )

    import json
    return json.loads(response.content[0].text)

# Uso no pipeline
avaliacao = avaliar_codigo(codigo_novo, spec_do_endpoint)
if not avaliacao["aprovado"]:
    print("Código rejeitado pelo LLM-as-Judge:")
    for problema in avaliacao["problemas_criticos"]:
        print(f"  - {problema}")
```

---

## 23. Adoção de IA em Equipes

### 23.1 Introduzindo IA Gradualmente

A adoção forçada gera resistência. Um modelo gradual funciona melhor:

```
Fase 1 — Exploração Individual (semanas 1-4)
└─ Cada dev usa a ferramenta no próprio ritmo
└─ Tarefas de baixo risco: testes, documentação, scripts
└─ Nenhuma pressão para usar em código crítico

Fase 2 — Compartilhamento (semanas 5-8)
└─ Sessões de 30min: "o que funcionou pra mim"
└─ Coletar prompts eficazes em prompts-eficazes.md
└─ Identificar casos de uso com maior ROI para o time

Fase 3 — Padronização (semanas 9-12)
└─ Criar CLAUDE.md compartilhado no repositório
└─ Definir política de revisão de código gerado por IA
└─ Integrar ao fluxo de CI/CD (quality gates)

Fase 4 — Otimização contínua
└─ Revisar CLAUDE.md mensalmente
└─ Medir métricas: tempo de feature, cobertura de testes, bugs
└─ Atualizar harness conforme o time aprende
```

---

### 23.2 Políticas de Uso

Um documento de política de uso de IA no time deve responder:

```markdown
## Política de Uso de IA — [Nome da Equipe]

### O que pode ser enviado para LLMs externos
✓ Código sem dados pessoais ou segredos
✓ Código de sistemas não-críticos
✓ Pseudocódigo e diagramas
✗ Chaves de API, senhas, tokens
✗ Dados pessoais de usuários (CPF, email, endereço)
✗ Segredos de negócio (algoritmos proprietários, dados financeiros)
✗ Código sob NDA específico

### Revisão obrigatória antes de merge
- Todo código com IA deve ter aprovação de um humano no PR
- Commits gerados por IA devem ter "co-authored-by" ou tag [AI]
- Code review de segurança obrigatório para: auth, payments, dados pessoais

### Ferramentas aprovadas
- Claude Code (versão X.X) — tarefas completas de desenvolvimento
- GitHub Copilot — completions no IDE
- ChatGPT / Gemini — consultas e prototipagem
- (Proibido: ferramentas não auditadas pela equipe de segurança)
```

---

### 23.3 Compartilhamento de Prompts e Harness

**Repositório de prompts compartilhados:**

```
docs/ai/
├── prompts/
│   ├── geracao-crud.md          ← template para gerar CRUD completo
│   ├── geracao-testes.md        ← template para testes unitários
│   ├── debug-lazyinit.md        ← prompt específico para LazyInitializationException
│   ├── refatoracao-service.md   ← prompt para extrair métodos de service
│   └── revisao-seguranca.md     ← prompt de security review
├── CLAUDE.md                    ← harness do projeto (versionado no git)
└── config/
    └── settings.json            ← configuração MCP e hooks (sem segredos)
```

**Formato de um prompt compartilhado:**

```markdown
<!-- docs/ai/prompts/geracao-crud.md -->
# Prompt: Geração de CRUD Completo

## Quando usar
Para criar um novo módulo de CRUD completo seguindo a arquitetura do projeto.

## Variáveis
- {ENTIDADE}: nome da entidade (ex: Fornecedor)
- {CAMPOS}: lista de campos com tipos (ex: razaoSocial: String, cnpj: String único)
- {NUMERO_MIGRATION}: próximo número disponível da migration Flyway

## Prompt
```
Crie o módulo completo de {ENTIDADE} com os campos: {CAMPOS}.
Siga rigorosamente as convenções do CLAUDE.md.
Inclua:
- Entidade JPA com validações Jakarta
- RequestDTO e ResponseDTO (Records)
- Repository com método findByAtivoTrue()
- Service com CRUD completo e tratamento de exceções
- Controller REST em /api/{entidade-plural-minusculo}
- Migration Flyway V{NUMERO_MIGRATION}__cria_tabela_{entidade_snake_case}.sql
- Testes unitários do Service cobrindo CRUD e casos de erro
```

## Resultado esperado
8-10 arquivos novos. Testes devem passar com mvn test.

## Última atualização
2025-04 por @ftsuda
```

---

### 23.4 Autoria, Rastreabilidade e Ética

**Rastreabilidade no git:**

```bash
# Commits com código gerado por IA — usar co-authored-by
git commit -m "feat(pedidos): adicionar paginação ao endpoint de listagem

Co-authored-by: Claude Sonnet 4.6 <noreply@anthropic.com>"

# Ou marcar com tag no commit message
git commit -m "[AI-assisted] refactor(service): extrair métodos de validação"
```

**Questões éticas e práticas:**

```
Responsabilidade: o desenvolvedor que aprova o commit é responsável pelo
código — independente de quem (ou o quê) o gerou. "A IA fez" não é defesa
válida em produção.

Propriedade intelectual: verifique os termos de serviço da ferramenta
usada. A maioria das ferramentas corporativas (Claude for Enterprise,
Copilot Business) garante que o código não é usado para treinar o modelo.

Viés e qualidade: código gerado por IA pode perpetuar anti-padrões
presentes nos dados de treinamento. Code review humano não é opcional.

Dependência excessiva: times que perdem a capacidade de escrever código
sem IA ficam vulneráveis a mudanças de pricing, disponibilidade e qualidade.
Manter o "músculo" de programação manual.
```

---

## 24. IA para Tarefas Não-Código no Desenvolvimento

### 24.1 Planejamento de Sprint e Estimativas

**Geração de user stories a partir de requisitos vagos:**

```
O product manager forneceu esta descrição:
"Precisamos que os clientes consigam ver o histórico de pedidos deles."

Transforme em:
1. Épico com descrição de valor de negócio
2. User stories no formato "Como [persona], quero [ação], para [benefício]"
   - Cubra: listagem, filtros úteis, detalhe do pedido, download de nota fiscal
3. Para cada story: critérios de aceitação (Gherkin) e pontos de atenção técnica
4. Identificar dependências entre stories
5. Estimativa relativa em story points (Fibonacci) justificada
```

**Análise de complexidade para estimativa:**

```
Analise a seguinte user story sob a perspectiva do nosso codebase
(arquitetura e stack no CLAUDE.md) e estime a complexidade:

Story: "Como cliente, quero receber notificação por email quando meu
pedido for enviado para entrega."

Responda:
- Componentes que precisam ser criados ou modificados
- Integrações externas envolvidas
- Riscos técnicos
- Estimativa em story points (1, 2, 3, 5, 8, 13) com justificativa
- Critérios de done técnico
```

---

### 24.2 Diagramas com Mermaid e PlantUML

A IA pode gerar diagramas a partir da análise do código — mais confiáveis do que diagramas escritos à mão que ficam desatualizados.

**Diagrama de sequência a partir de código:**

```
Analise o fluxo de criarPedido em PedidoService.java e todas as classes
chamadas por ele. Gere um diagrama de sequência Mermaid mostrando:
- Participantes (controller, services, repositories, externas)
- Chamadas e retornos principais
- Ponto de decisão de falha de estoque
- Transação de banco (início e fim)
```

```mermaid
sequenceDiagram
    Client->>PedidoController: POST /api/pedidos
    PedidoController->>PedidoService: criarPedido(dto)
    PedidoService->>EstoqueService: reservar(itens)
    EstoqueService-->>PedidoService: ReservaEstoque
    PedidoService->>PedidoRepository: save(pedido)
    PedidoRepository-->>PedidoService: Pedido (com ID)
    PedidoService->>NotificacaoService: notificarCliente(pedido)
    PedidoService-->>PedidoController: PedidoResponseDTO
    PedidoController-->>Client: 201 Created
```

**Diagrama de classes:**

```
Analise as entidades JPA em src/main/java/**/entity/ e gere um diagrama
de classes PlantUML mostrando:
- Todas as entidades e seus campos principais (omitir id, createdAt, etc.)
- Relacionamentos JPA (@OneToMany, @ManyToOne, @ManyToMany)
- Cardinalidade de cada relacionamento
- Enums usados pelas entidades
```

---

### 24.3 Architecture Decision Records (ADRs)

**Gerando ADRs a partir de decisões já tomadas no código:**

```
Analise as seguintes decisões de implementação no codebase e gere um ADR
para cada uma no formato MADR (Markdown Architectural Decision Records):

1. Uso de JWT stateless em vez de sessão no servidor
   (ver: SecurityConfig.java, JwtFilter.java)

2. Uso de Flyway para migrations em vez de schema automático do Hibernate
   (ver: pom.xml, src/main/resources/db/migration/)

3. Uso de DTOs separados para request e response em vez de um único DTO
   (ver: padrão NomeRequestDTO / NomeResponseDTO nos pacotes dto/)

Para cada ADR, inclua:
- Contexto e problema
- Opções consideradas (infira a partir do código e boas práticas comuns)
- Decisão tomada e justificativa
- Consequências positivas e negativas
```

**Template MADR:**

```markdown
# ADR-001: JWT Stateless para Autenticação

## Status
Accepted

## Contexto
A aplicação precisa de autenticação para uma API REST consumida por
clientes web e mobile. A equipe considerou sessões no servidor e JWT.

## Decisão
Usar JWT (JSON Web Tokens) stateless com validade de 1 hora e refresh token.

## Opções consideradas
| Opção            | Vantagem                    | Desvantagem               |
|------------------|-----------------------------|---------------------------|
| Sessão HTTP      | Revogação imediata          | Estado no servidor, escala horizontal complexa |
| JWT stateless    | Sem estado, escala horizontal | Revogação requer blacklist |
| OAuth2 externo   | Delegação de identidade     | Complexidade, dependência externa |

## Consequências
- Positivo: escala horizontal sem sincronização de sessão
- Positivo: mobile e SPA se autenticam com o mesmo mecanismo
- Negativo: tokens não podem ser revogados antes da expiração sem blacklist
- Negativo: payload do JWT é visível (não criptografado, apenas assinado)
```

---

## 25. Fine-tuning e RAG sobre Codebase Próprio

### 25.1 Quando Vale a Pena Ir Além do Contexto Nativo

A maioria dos projetos **não precisa** de fine-tuning ou RAG customizado — o contexto nativo (CLAUDE.md + código no contexto) é suficiente. Considerar RAG dedicado apenas quando:

```
Indicadores de que RAG dedicado pode valer a pena

✓ Repositório com > 500.000 linhas de código (não cabe no contexto)
✓ Documentação interna extensa (wikis, ADRs, runbooks) que mudam frequentemente
✓ Múltiplos repositórios interdependentes
✓ Histórico de decisões crítico (por que foi feito assim)
✓ API interna complexa que a IA frequentemente erra
✓ Time de > 20 devs com uso intensivo (custo de contexto fica alto)

Não vale a pena quando:
✗ Repositório cabe no contexto do modelo
✗ O problema é falta de instruções claras (resolver com CLAUDE.md melhor)
✗ Orçamento não suporta infraestrutura adicional
✗ Time pequeno (< 5 devs)
```

---

### 25.2 Ferramentas de RAG sobre Código

| Ferramenta        | Tipo          | Descrição                                        | Preço       |
|-------------------|---------------|--------------------------------------------------|-------------|
| **Sourcegraph Cody** | SaaS/On-prem | RAG sobre codebase com contexto de repositório inteiro | $19/dev/mês |
| **Greptile**      | SaaS          | API de Q&A sobre repositórios GitHub/GitLab      | Por uso     |
| **CodeRabbit**    | SaaS          | Revisão automática de PR com contexto do repo    | $12/dev/mês |
| **Continue.dev**  | Open source   | Extensão IDE com RAG local via Ollama ou APIs    | Gratuito    |
| **LlamaIndex**    | Biblioteca    | Framework Python para RAG customizado            | Gratuito    |
| **Weaviate/Chroma** | Banco vetorial | Infraestrutura para embeddings de código       | Gratuito    |

**RAG simples com LlamaIndex sobre o próprio codebase:**

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.node_parser import CodeSplitter

# Indexar o codebase Java
documents = SimpleDirectoryReader(
    input_dir="src/main/java",
    recursive=True,
    required_exts=[".java"]
).load_data()

# Splitter específico para código (mantém métodos inteiros)
splitter = CodeSplitter(language="java", chunk_lines=40, chunk_lines_overlap=5)
nodes = splitter.get_nodes_from_documents(documents)

# Criar índice com embeddings
index = VectorStoreIndex(nodes)
query_engine = index.as_query_engine()

# Consultar
response = query_engine.query(
    "Como o sistema trata erros de pagamento? Onde está essa lógica?"
)
print(response)
```

---

### 25.3 Privacidade e Opções On-Premise

Para projetos com restrições de privacidade (bancos, saúde, governo), enviar código para APIs externas pode não ser permitido:

```
Opções on-premise (sem enviar código para servidores externos)

1. Modelos locais via Ollama
   └─ ollama pull codellama:34b
   └─ ollama pull deepseek-coder-v2:16b
   └─ Continue.dev conecta ao Ollama como backend
   └─ Qualidade inferior a Claude/GPT-4, mas sem dados saírem da rede

2. Azure OpenAI / AWS Bedrock (Claude)
   └─ APIs OpenAI/Anthropic hospedadas na cloud contratada
   └─ Dados processados na região/conta da empresa
   └─ Sem uso para treinamento de modelos (por contrato)
   └─ Conformidade com SOC2, ISO 27001

3. Anthropic Claude for Enterprise
   └─ Garantia contratual de não uso dos dados para treino
   └─ Logs de auditoria
   └─ SSO integrado

4. Google Vertex AI (Gemini)
   └─ Processamento nos servidores Google Cloud da empresa
   └─ Sem envio para infraestrutura de treinamento

Comparativo de restrição de privacidade
Alta restrição: Ollama local > Azure/AWS hospedado > SaaS Enterprise
Baixa restrição: Claude.ai / ChatGPT gratuito / APIs padrão
```

---

## 26. Limitações e Modos de Falha

### 26.1 Alucinações em APIs e Bibliotecas

LLMs podem gerar código sintaticamente correto que referencia APIs que não existem ou foram removidas. Esse é o modo de falha mais comum e mais perigoso porque compila mas falha em tempo de execução.

**Categorias de alucinação:**

```java
// Tipo 1: Método inexistente na versão atual
// Spring Security 6 removeu hasAuthority() da interface Authentication
if (auth.hasAuthority("ADMIN")) { ... }   // ← não existe
// Correto:
auth.getAuthorities().stream().anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));

// Tipo 2: Parâmetros trocados
// IA inverteu a ordem de parâmetros
repository.findByNomeAndAtivo("João", true);  // ← pode ser findByAtivoAndNome
// Verificar sempre a assinatura real no Repository

// Tipo 3: Versão errada de API
// IA usou API de Spring Boot 2.x em projeto 3.x
WebSecurityConfigurerAdapter  // ← removido no Spring Security 6
// Correto: SecurityFilterChain @Bean

// Tipo 4: Biblioteca inventada
import com.example.utils.SmartValidator;  // ← não existe
// IA "criou" uma biblioteca que não existe no Maven Central
```

**Protocolo de verificação:**

```bash
# 1. Compilar imediatamente após geração
mvn compile 2>&1 | grep "error:"

# 2. Verificar imports de classes novas
mvn dependency:resolve 2>&1 | grep "missing\|not found"

# 3. Para cada método desconhecido: verificar no Javadoc oficial
# Não confiar apenas no que a IA diz que existe
```

---

### 26.2 Falhas Sistemáticas por Categoria

Padrões onde LLMs falham com consistência — exigem atenção extra na revisão:

**Concorrência e threading:**
```java
// IA frequentemente ignora race conditions
public class Contador {
    private int valor = 0;  // ← não thread-safe
    public void incrementar() { valor++; }  // ← operação não atômica
}
// Correto: AtomicInteger ou synchronized

// IA pode sugerir sincronização excessiva (bottleneck)
public synchronized List<Produto> buscarTodos() { ... }  // ← lock desnecessário em leitura
// Melhor: ReadWriteLock ou dados imutáveis
```

**Lógica de negócio com regras implícitas:**
```
A IA não sabe que:
- O desconto de fidelidade não se acumula com desconto de volume (regra oral)
- Pedidos de clientes bloqueados ainda aparecem no relatório (comportamento legado)
- O campo 'valor' usa centavos internamente, não reais (convenção não documentada)

→ Essas regras devem estar no CLAUDE.md ou na spec, senão a IA vai adivinhar errado
```

**Aritmética de ponto flutuante em valores monetários:**
```java
// IA frequentemente usa double para dinheiro (ERRADO)
double preco = 19.99;
double desconto = preco * 0.1;  // 1.999... (não 2.00)

// Correto para valores monetários:
BigDecimal preco = new BigDecimal("19.99");
BigDecimal desconto = preco.multiply(new BigDecimal("0.10"))
                           .setScale(2, RoundingMode.HALF_UP);
```

**Timezone e datas:**
```java
// IA ignora timezone com frequência
LocalDateTime agora = LocalDateTime.now();  // ← timezone do servidor, não do usuário
// Em sistemas distribuídos: sempre Instant ou ZonedDateTime

// Comparação de datas:
pedido.getDataCriacao().isAfter(LocalDateTime.now().minusDays(7));  // ← pode ser errado se timezone diferente
```

**Transações e consistência:**
```java
// IA frequentemente esquece @Transactional ou o coloca no lugar errado
@RestController  // ← @Transactional aqui não funciona (proxy não intercepta)
public class PedidoController {
    @Transactional  // ← ERRADO: deve estar no Service
    public ResponseEntity<?> criar(...) { ... }
}

// Outro erro: @Transactional em método privado (AOP não intercepta)
@Service
public class PedidoService {
    @Transactional  // ← SEM EFEITO em método privado
    private void reservarEstoque(...) { ... }
}
```

---

### 26.3 Reconhecendo Padrões de Falha Antes da Produção

**Checklist de revisão focado nos pontos cegos da IA:**

```markdown
## Revisão de Código IA — Pontos Cegos Conhecidos

### Dinheiro e Cálculos
- [ ] Tipos monetários usam BigDecimal (nunca float/double)
- [ ] Arredondamentos explícitos com RoundingMode definido
- [ ] Divisões inteiras intencionais (não acidentais)

### Datas e Tempo
- [ ] Fusos horários tratados explicitamente onde relevante
- [ ] Uso de Instant para timestamps de sistema
- [ ] Comparações de data não assumem timezone implícito

### Concorrência
- [ ] Campos compartilhados entre threads são thread-safe
- [ ] Lazy loading do Hibernate não ocorre fora de transação
- [ ] Caches locais não causam stale data visível ao usuário

### Transações
- [ ] @Transactional está no Service (não no Controller)
- [ ] Métodos @Transactional são públicos (AOP requirement)
- [ ] Rollback em exceções checked configurado se necessário

### Negócio
- [ ] Regras implícitas do domínio foram verificadas com PO/BA
- [ ] Casos especiais históricos ("nunca mude isso") foram preservados
- [ ] Valores de enum mapeados correspondem ao banco de dados real

### APIs e Dependências
- [ ] Todos os imports compilam (mvn compile sem erros)
- [ ] Versão das APIs verificada na documentação oficial
- [ ] Nenhuma nova dependência adicionada sem revisão
```

**Prompt para forçar a IA a detectar seus próprios pontos cegos:**

```
Revise o código que você acabou de gerar com o olhar crítico de um engenheiro
sênior que conhece os seguintes padrões de erro comuns em código gerado por IA:

1. Tipos monetários incorretos (float/double em vez de BigDecimal)
2. Timezone ignorado em datas
3. @Transactional em lugar errado ou em método privado
4. APIs que podem não existir na versão especificada no CLAUDE.md
5. Campos não thread-safe em beans Spring (singleton por padrão)
6. Optional.get() sem verificação prévia
7. Lógica de negócio que contradiz as regras do CLAUDE.md

Para cada item: confirme que está correto ou aponte o problema e corrija.
```

---

> **Lembre-se:** ferramentas de IA amplificam a capacidade do desenvolvedor, mas não substituem o julgamento de engenharia. O desenvolvedor continua responsável pelo código que vai para produção — independente de quem (ou o quê) o escreveu.
