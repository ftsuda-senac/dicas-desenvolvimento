# Tópicos Complementares para Desenvolvimento Web (2)

> **Objetivo:** Listar temas que ainda não possuem documento dedicado no repositório e seriam complementos naturais ao material existente.

---

## Backend / APIs

| Tema | Justificativa |
|------|--------------|
| **GraphQL com Spring Boot** | Mencionado no Tópicos-Complementares mas sem documento próprio. Alternativa moderna ao REST |
| **Mensageria (Kafka, RabbitMQ, SQS)** | Também mencionado nos tópicos complementares; essencial para sistemas distribuídos, complementa o System-design.md |
| **Documentação de APIs (OpenAPI/Swagger)** | Prática obrigatória em projetos reais; não coberto em nenhum documento |
| **gRPC e comunicação inter-serviços** | Complementa REST e WebSocket para comunicação entre microsserviços |
| **Versionamento de APIs** | Estratégias (path, header, media type) — tema recorrente em projetos profissionais |
| **Padrões de Projeto (Design Patterns)** | GoF aplicados a Java/Spring Boot — complementa Boas-Praticas-Arquitetura.md |

---

## Banco de Dados

| Tema | Justificativa |
|------|--------------|
| **SQL Avançado e Modelagem de Dados** | Fora do contexto JPA — SQL puro, indexação, views, CTEs, funções de janela, otimização de queries |
| **NoSQL na prática (MongoDB, Redis, Elasticsearch, Solr)** | O Spring-Data-JDBC-R2DBC-Mongo-Redis-Elasticsearch.md foca na integração Spring; falta um documento sobre quando/como usar cada tipo de banco, incluindo Apache Solr como engine de busca full-text |

---

## Frontend / Fullstack

| Tema | Justificativa |
|------|--------------|
| **Next.js / Nuxt.js (Frameworks Fullstack)** | Mencionado nos tópicos complementares; SSR/SSG com React e Vue são padrão de mercado |
| **Svelte / SvelteKit ou Astro** | Frameworks modernos em crescimento; alternativas leves aos "big three" |
| **Micro Frontends** | Arquitetura de frontend distribuído para projetos grandes |
| **Internacionalização (i18n/l10n)** | Tema transversal importante, pouco coberto nos docs de frontend |

---

## Segurança / Autenticação

| Tema | Justificativa |
|------|--------------|
| **OAuth2 / OpenID Connect em profundidade** | Vai além do que o Dicas-Spring-Security.md e Keycloak.md cobrem; fluxos, tokens, escopos |
| **Passkeys / WebAuthn / Autenticação Passwordless** | Tendência forte de mercado; complementa autenticação tradicional |

---

## DevOps / Infra

| Tema | Justificativa |
|------|--------------|
| **Serverless (AWS Lambda, Azure Functions, GCP Cloud Run)** | Paradigma complementar a containers; o AWS-Azure-GCP-Cheat-Sheet.md é genérico |
| **Linux para Desenvolvedores** | Comandos essenciais, shell scripting, administração básica — muitos alunos vêm do Windows |
| **Monitoramento e Alertas (Prometheus, Grafana, ELK)** | Aprofundamento do que o Dicas-Logs-Observabilidade.md introduz |

---

## Qualidade e Práticas

| Tema | Justificativa |
|------|--------------|
| **Clean Code e Code Review** | Mencionado nos tópicos complementares; práticas de legibilidade, refatoração e revisão de código |
| **Testes E2E e de Contrato (Pact, Playwright)** | Os docs de teste existentes focam em unitário/integração; falta E2E e contract testing |
| **Feature Flags e Entrega Progressiva** | Mencionado no CI/CD mas merece documento próprio com exemplos práticos |

---

## Temas Emergentes

| Tema | Justificativa |
|------|--------------|
| **WebAssembly (Wasm)** | Tecnologia em ascensão; complementa o doc de Web-Graphics-Canvas-SVG-3D.md |
| **Desenvolvimento com IA para Devs (além do Spring AI)** | O Desenvolvimento-com-IA.md e Java-Spring-AI-Integration.md existem, mas falta um guia prático de prompt engineering e uso de LLMs no dia-a-dia do desenvolvedor |
| **Electron / Tauri** | Apps desktop com tecnologias web — ponte entre frontend e desktop |
