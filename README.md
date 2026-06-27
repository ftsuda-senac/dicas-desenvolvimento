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

---

## Referências para desenvolvimento Frontend Web

Documentação consolidada com apoio de agentes de IA (Claude Code).

* [Dicas-Desenvolvimento-Web-Frontend.md](Dicas-Desenvolvimento-Web-Frontend.md) — Usabilidade, acessibilidade. mobile-first, Figma, HTML, CSS moderno e SCSS
* [Dicas-Javascript-Typescript.md](Dicas-Javascript-Typescript.md)
* [Frontend-React.md](Frontend-React.md) — Visão geral do React 19 com TypeScript
* [Frontend-Angular.md](Frontend-Angular.md) — Visão geral do Angular 21
* [Frontend-Vue.md](Frontend-Vue.md) — Visão geral do Vue 3.5 com TypeScript
* [Frontend-Flutter.md](Frontend-Flutter.md) — Flutter 3 com Dart: navegação, estado global, backend, i18n, JWT e distribuição Android/iOS

---

## Outros assuntos avançados

* [Spring-Boot-Avancado.md](Spring-Boot-Avancado.md) — Tópicos avançados do Spring Boot
* [Dicas-Logs-Observabilidade.md](Dicas-Logs-Observabilidade.md) — Logs estruturados, MDC, rastreamento distribuído e infraestrutura de observabilidade

---

## Segurança de Software

* [Analise-Seguranca.md](Analise-Seguranca.md) — DevSecOps, SAST, SCA, DAST, secrets, segurança no frontend/backend/banco de dados, pipeline CI/CD seguro, monitoramento e resposta a incidentes

---

## Arquitetura e System Design

* [Boas-Praticas-Arquitetura.md](Boas-Praticas-Arquitetura.md) — 12 Factor App, SOLID, Arquitetura Hexagonal/Onion, TDD e Requirement Driven Development com IA — com exemplos em Java e Spring Boot
* [System-design.md](outros/System-design.md) — Escalabilidade, Load Balancing, Caching, Banco de Dados, Replicação, Sharding, Mensageria, Design de APIs, Microsserviços, Circuit Breaker, CQRS, Event Sourcing, Service Discovery, API Gateway, Teorema CAP, Rate Limiting, Segurança e Observabilidade em sistemas distribuídos — com exemplos em Java e Spring Boot
* [Spring-Cloud-Modulith.md](outros/Spring-Cloud-Modulith.md) — Spring Cloud (Gateway, Config, Eureka, Feign, Stream, LoadBalancer) e Spring Modulith para arquitetura modular — com tutorial prático completo e guia de decisão

---

## Infraestrutura e DevOps

* [Docker-Compose-Kubernetes.md](outros/Docker-Compose-Kubernetes.md) — Docker (Dockerfile, otimização de imagens, registries, segurança e vulnerabilidades, Testcontainers, alternativas como Podman), Docker Compose (stacks, profiles, override, YAML anchors, recursos, réplicas), Docker Swarm, Kubernetes (arquitetura, Pods, init containers, sidecar, Services, Ingress, Service Mesh, RBAC, Secrets avançados, HPA, estratégias de deploy, Helm, Operators, GitOps, Kustomize, K8s em cloud, backup/DR) — com exemplos em Java/Spring Boot e pipelines CI/CD
* [Git-Estrategias-Branching.md](outros/Git-Estrategias-Branching.md) — Git avançado: configuração, comandos essenciais e avançados, estratégias de branching (Git Flow, GitHub Flow, Trunk-Based, GitLab Flow), Conventional Commits, tags e SemVer, hooks, Pull Requests, .gitignore, submódulos, monorepos e boas práticas
* [CI-CD-Github-Actions-Gitlab.md](outros/CI-CD-Github-Actions-Gitlab.md) — CI/CD com GitHub Actions e GitLab CI/CD: pipelines, cache, matrices, secrets, environments, deploy em cloud (AWS/Azure/GCP), estratégias de deploy (blue-green, canary, rollback), database migrations, feature flags, Terraform/IaC, GitOps (ArgoCD/Flux), segurança (SAST/SCA/SBOM), testes avançados (E2E, performance, SonarQube), Dependabot/Renovate, self-hosted runners, gerenciamento de projetos (Issues, Kanban, Quick Actions, CODEOWNERS), instalação on-premise do GitLab CE com dimensionamento de hardware — com exemplos para Java/Spring Boot, frontend (React/Angular/Vue) e mobile (React Native/Flutter)

---

# Dicas e boas práticas de desenvolvimento (legado)

Documentações, tutoriais e dicas gerais

* [Dicas Java](Dicas-Java.md)
* [Dicas e Tutoriais Spring](Dicas-e-Tutoriais-Spring.md)
* [Dicas usabilidade](Dicas-usabilidade.md)

---
