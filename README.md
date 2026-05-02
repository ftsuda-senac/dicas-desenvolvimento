# Dicas e referências para Desenvolvimento Web

Repositório de documentação técnica de referência para disciplinas de desenvolvimento web, cobrindo backend com Java e Spring Boot, frontend moderno (Angular, React, Vue, Flutter) e tópicos transversais como segurança, observabilidade e boas práticas. Os documentos foram produzidos e revisados com apoio de agentes de IA e servem como guia de consulta rápida durante o desenvolvimento, reunindo padrões, exemplos de código e orientações práticas organizados por tema.

## Referências para desenvolvimento com Spring Boot

Documentação consolidada com apoio de agentes de IA (Claude Code e OpenAI Codex).

Ordem recomendada para os documentos de referência do ecossistema Spring:

1. [Dicas-Spring-Inicial.md](Dicas-Spring-Inicial.md) — Java, Spring Framework (IoC/DI) e boas práticas da camada de serviço
2. [Dicas-Spring-MVC-REST.md](Dicas-Spring-MVC-REST.md) **ou** [Dicas-Spring-MVC-SSR.md](Dicas-Spring-MVC-SSR.md) — Desenvolvimento web de Backend no modelo API REST ou MVC clássico (SSR com Thymeleaf)
3. [Dicas-JPA-Hibernate-Modelagem-Entities.md](Dicas-JPA-Hibernate-Modelagem-Entities.md) — Modelagem de entidades JPA/Hibernate
4. [Dicas-JPA-Hibernate-Queries.md](Dicas-JPA-Hibernate-Queries.md) — Consultas JPQL, Criteria API e Spring Data
5. [Dicas-Spring-Security.md](Dicas-Spring-Security.md) — Autenticação e autorização com Spring Security
6. [Spring-Data-JDBC-R2DBC-Mongo-Redis-Elasticsearch.md](outros/Spring-Data-JDBC-R2DBC-Mongo-Redis-Elasticsearch.md) — Spring Data JDBC, R2DBC, MongoDB, Redis e Elasticsearch

---

## Referências para desenvolvimento Frontend Web

Documentação consolidada com apoio de agentes de IA (Claude Code).

* [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md) — Usabilidade, acessibilidade. mobile-first, Figma, HTML, CSS moderno e SCSS
* [Dicas-Javascript-Typescript.md](Dicas-Javascript-Typescript.md)
* [Frontend-Angular.md](Frontend-Angular.md) — Visão geral do Angular 21
* [Frontend-React.md](Frontend-React.md) — Visão geral do React 19 com TypeScript
* [Frontend-Vue.md](Frontend-Vue.md) — Visão geral do Vue 3.5 com TypeScript
* [Frontend-Flutter.md](Frontend-Flutter.md) — Flutter 3 com Dart: navegação, estado global, backend, i18n, JWT e distribuição Android/iOS

---

## Outros assuntos avançados

* [Spring-Boot-Avancado.md](Spring-Boot-Avancado.md) — Tópicos avançados do Spring Boot
* [Dicas-Logs-Observabilidade.md](Dicas-Logs-Observabilidade.md) — Logs estruturados, MDC, rastreamento distribuído e infraestrutura de observabilidade
* [GraalVM-Native-Image-Spring-Boot.md](outros/GraalVM-Native-Image-Spring-Boot.md) — Compilação nativa com GraalVM: AOT, reflexão, proxies, JPA/Hibernate, Jackson, AOP e bibliotecas de terceiros

---

## Segurança de Software

* [Analise-Seguranca.md](Analise-Seguranca.md) — DevSecOps, SAST, SCA, DAST, secrets, segurança no frontend/backend/banco de dados, pipeline CI/CD seguro, monitoramento e resposta a incidentes

---

## Arquitetura e System Design

* [Boas-Praticas-Arquitetura.md](Boas-Praticas-Arquitetura.md) — 12 Factor App, SOLID, Arquitetura Hexagonal/Onion, TDD e Requirement Driven Development com IA — com exemplos em Java e Spring Boot
* [System-design.md](System-design.md) — Escalabilidade, Load Balancing, Caching, Banco de Dados, Replicação, Sharding, Mensageria, Design de APIs, Microsserviços, Circuit Breaker, CQRS, Event Sourcing, Service Discovery, API Gateway, Teorema CAP, Rate Limiting, Segurança e Observabilidade em sistemas distribuídos — com exemplos em Java e Spring Boot

---

## Infraestrutura como Código

* [Terraform.md](outros/Terraform.md) — IaC com Terraform: providers, resources, variables, outputs, data sources, state, módulos, workspaces, remote backend, testes, integração com CI/CD e boas práticas com exemplos na AWS

---

## Kotlin

* [Kotlin.md](outros/Kotlin.md) — Guia prático focado em Spring Boot: comparações com Java (null safety, data classes, when, extension functions, coroutines), ambiente de desenvolvimento (SDKMAN, JDK 21, Gradle Kotlin DSL), fundamentos da linguagem (variáveis, condicionais, loops, coleções, funções de ordem superior, scope functions), orientação a objetos (data classes, sealed classes, companion object), integração com JPA/Hibernate, Spring Data, Spring Security com Kotlin DSL e JWT, e testes com JUnit 5 e MockMvc

---

## Python 3

* [Python.md](outros/Python.md) — Guia abrangente: ambiente de desenvolvimento (pyenv, venv, uv), IDEs, fundamentos da linguagem (variáveis, condicionais, loops, coleções, OOP, decoradores), desenvolvimento web com FastAPI e Django/DRF, integração com banco de dados (SQLAlchemy 2.x, Alembic, MongoDB, Redis), machine learning (scikit-learn, PyTorch, Transformers), visão computacional (OpenCV, YOLO, Pillow), automação e web scraping (Playwright, BeautifulSoup, Scrapy), análise de dados (pandas, NumPy, Polars, matplotlib, seaborn), scripting DevOps (click, Typer, boto3, Fabric) e testes com pytest

---

## Go (Golang)

* [Golang.md](outros/Golang.md) — Casos de uso e ecossistema (CLIs, infraestrutura, gRPC, workers, segurança), ambiente de desenvolvimento, fundamentos da linguagem (variáveis, condicionais, loops, coleções, structs, interfaces, concorrência), desenvolvimento web com net/http e Gin, integração com banco de dados via database/sql, GORM e sqlx, testes e Docker

---

## Rust e WebAssembly

* [Rust.md](outros/Rust.md) — Linguagem Rust com foco em WebAssembly: ownership/borrowing/lifetimes, structs/enums/pattern matching, tratamento de erros, traits/generics, concorrência; ambiente de desenvolvimento, IDEs (VS Code, RustRover); fundamentos da linguagem (variáveis, condicionais, loops, coleções, strings); projetos WASM com wasm-pack/wasm-bindgen, processamento de imagens no browser, comunicação bidirecional Rust↔JavaScript, framework Yew (SPA estilo React) e testes WASM com headless browser

---

## Desenvolvimento com Ferramentas de IA

* [Desenvolvimento-com-IA.md](outros/Desenvolvimento-com-IA.md) — Boas práticas, configurações e prompts para Claude Code, Gemini CLI e OpenAI Codex/ChatGPT; conceitos de MCP, vibe coding, spec-driven development e harness engineering
* [Java-Spring-AI-Integration.md](outros/Java-Spring-AI-Integration.md) — Integração de Java/Spring Boot com IA: Spring AI, LangChain4J, Semantic Kernel, SDKs nativos (OpenAI, Anthropic, Gemini), Ollama, bancos de dados vetoriais e padrões RAG/Agentes

---

# Dicas e boas práticas de desenvolvimento (legado)

Documentações, tutoriais e dicas gerais

* [Dicas Java](Dicas-Java.md)
* [Dicas e Tutoriais Spring](Dicas-e-Tutoriais-Spring.md)
* [Dicas usabilidade](Dicas-usabilidade.md)

---

