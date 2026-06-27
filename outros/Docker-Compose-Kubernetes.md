# Docker, Docker Compose e Kubernetes — Guia Prático

> **Objetivo:** Cobrir de forma abrangente os conceitos, comandos, configurações e boas práticas de Docker, Docker Compose e Kubernetes, com exemplos práticos voltados para aplicações Java/Spring Boot e ecossistema web moderno.

---

## Sumário

1. [Fundamentos de Containers](#1-fundamentos-de-containers)
2. [Docker — Instalação e Arquitetura](#2-docker--instalação-e-arquitetura)
3. [Imagens Docker](#3-imagens-docker)
4. [Container Registries](#4-container-registries)
5. [Dockerfile — Construção de Imagens](#5-dockerfile--construção-de-imagens)
6. [Otimização de Imagens Docker](#6-otimização-de-imagens-docker)
7. [Comandos Essenciais do Docker CLI](#7-comandos-essenciais-do-docker-cli)
8. [Volumes e Persistência de Dados](#8-volumes-e-persistência-de-dados)
9. [Redes Docker](#9-redes-docker)
10. [Docker Compose](#10-docker-compose)
11. [Docker Compose — Exemplos Práticos](#11-docker-compose--exemplos-práticos)
12. [Docker Compose — YAML Anchors, Limitação de Recursos e Réplicas](#12-docker-compose--yaml-anchors-limitação-de-recursos-e-réplicas)
13. [Docker Swarm](#13-docker-swarm)
14. [Boas Práticas de Docker](#14-boas-práticas-de-docker)
15. [Segurança e Análise de Vulnerabilidades em Containers](#15-segurança-e-análise-de-vulnerabilidades-em-containers)
16. [Testcontainers — Testes de Integração com Docker](#16-testcontainers--testes-de-integração-com-docker)
17. [Alternativas ao Docker](#17-alternativas-ao-docker)
18. [Kubernetes — Arquitetura e Conceitos](#18-kubernetes--arquitetura-e-conceitos)
19. [Kubectl — Comandos Essenciais](#19-kubectl--comandos-essenciais)
20. [Pods e Workloads](#20-pods-e-workloads)
21. [Init Containers e Padrões Multi-container](#21-init-containers-e-padrões-multi-container)
22. [Services e Networking no Kubernetes](#22-services-e-networking-no-kubernetes)
23. [Ingress e Roteamento HTTP](#23-ingress-e-roteamento-http)
24. [Service Mesh](#24-service-mesh)
25. [ConfigMaps e Secrets](#25-configmaps-e-secrets)
26. [RBAC — Controle de Acesso](#26-rbac--controle-de-acesso)
27. [Gerenciamento Avançado de Secrets](#27-gerenciamento-avançado-de-secrets)
28. [Storage no Kubernetes](#28-storage-no-kubernetes)
29. [Escalabilidade e Autoscaling](#29-escalabilidade-e-autoscaling)
30. [Estratégias de Deploy no Kubernetes](#30-estratégias-de-deploy-no-kubernetes)
31. [Helm — Gerenciador de Pacotes](#31-helm--gerenciador-de-pacotes)
32. [Kubernetes Operators e CRDs](#32-kubernetes-operators-e-crds)
33. [Deploy de Spring Boot no Kubernetes](#33-deploy-de-spring-boot-no-kubernetes)
34. [GitOps com ArgoCD e Flux](#34-gitops-com-argocd-e-flux)
35. [Ambientes Locais de Kubernetes](#35-ambientes-locais-de-kubernetes)
36. [Kubernetes em Cloud (Managed)](#36-kubernetes-em-cloud-managed)
37. [Observabilidade no Kubernetes](#37-observabilidade-no-kubernetes)
38. [Backup e Disaster Recovery](#38-backup-e-disaster-recovery)
39. [Boas Práticas de Kubernetes](#39-boas-práticas-de-kubernetes)
40. [Troubleshooting](#40-troubleshooting)
41. [Exemplos Completos — Do Dockerfile ao Deploy em K8s](#41-exemplos-completos--do-dockerfile-ao-deploy-em-k8s)

---

## 1. Fundamentos de Containers

### 1.1 Máquinas Virtuais vs. Containers

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  VM vs. Container                                        │
│                                                                          │
│   Máquina Virtual                    Container                           │
│  ┌──────────┐ ┌──────────┐    ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │  App A   │ │  App B   │    │  App A   │ │  App B   │ │  App C   │   │
│  │  Libs    │ │  Libs    │    │  Libs    │ │  Libs    │ │  Libs    │   │
│  │  SO Guest│ │  SO Guest│    └──────────┘ └──────────┘ └──────────┘   │
│  └──────────┘ └──────────┘         Container Runtime (Docker)           │
│       Hypervisor (VMware,                                                │
│       VirtualBox, KVM)                  SO Host (Kernel compartilhado)   │
│     SO Host + Hardware                       Hardware                    │
└──────────────────────────────────────────────────────────────────────────┘
```

| Aspecto | Máquina Virtual | Container |
|---------|----------------|-----------|
| **Isolamento** | SO completo por VM | Processos isolados no mesmo kernel |
| **Tamanho** | GBs (SO inteiro) | MBs (apenas app + dependências) |
| **Startup** | Minutos | Segundos |
| **Overhead** | Alto (hypervisor + SO) | Mínimo (namespaces + cgroups) |
| **Portabilidade** | Limitada ao hypervisor | Qualquer host com Docker |
| **Densidade** | ~10 VMs/servidor | ~100+ containers/servidor |

### 1.2 Tecnologias base dos containers Linux

Containers não são uma tecnologia única — são um conjunto de funcionalidades do kernel Linux:

| Tecnologia | Função |
|------------|--------|
| **Namespaces** | Isolam recursos: PID, rede, filesystem, usuários |
| **cgroups** | Limitam CPU, memória, I/O de cada container |
| **Union FS** | Filesystem em camadas (OverlayFS) — imagens imutáveis com camadas de escrita |
| **seccomp** | Restringe chamadas de sistema disponíveis |
| **AppArmor / SELinux** | Políticas de segurança adicionais |

---

## 2. Docker — Instalação e Arquitetura

### 2.1 Arquitetura cliente-servidor

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Architecture                    │
│                                                          │
│  ┌──────────┐       ┌──────────────────────────────┐    │
│  │  Docker   │       │       Docker Daemon           │    │
│  │   CLI     │──────▶│       (dockerd)               │    │
│  │           │ REST  │                                │    │
│  │  docker   │  API  │  ┌──────────┐  ┌──────────┐  │    │
│  │  build    │       │  │ Images   │  │Containers│  │    │
│  │  run      │       │  └──────────┘  └──────────┘  │    │
│  │  push     │       │  ┌──────────┐  ┌──────────┐  │    │
│  └──────────┘       │  │ Volumes  │  │ Networks │  │    │
│                      │  └──────────┘  └──────────┘  │    │
│  ┌──────────┐       └──────────────────────────────┘    │
│  │ Registry │                                            │
│  │(Hub, ECR)│◀── pull/push ──────────────────────────    │
│  └──────────┘                                            │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Instalação

| Plataforma | Método |
|------------|--------|
| **Windows** | Docker Desktop (WSL2 backend) |
| **macOS** | Docker Desktop ou Colima (alternativa leve) |
| **Ubuntu/Debian** | `apt install docker-ce docker-ce-cli containerd.io` via repositório oficial |
| **Fedora/RHEL** | `dnf install docker-ce` ou Podman (compatível, daemonless) |

```bash
# Verificar instalação
docker --version
docker compose version

# Executar sem sudo (Linux) — adicionar usuário ao grupo docker
sudo usermod -aG docker $USER
# logout/login necessário para aplicar
```

---

## 3. Imagens Docker

### 3.1 Conceito de camadas

Cada instrução no Dockerfile cria uma camada (layer). Camadas são imutáveis e compartilhadas entre imagens.

```
┌───────────────────────────┐
│ ENTRYPOINT ["java","-jar"]│  ← Camada 5 (metadado)
├───────────────────────────┤
│ COPY app.jar              │  ← Camada 4 (app — muda frequentemente)
├───────────────────────────┤
│ RUN adduser appuser       │  ← Camada 3 (config)
├───────────────────────────┤
│ RUN apk add curl          │  ← Camada 2 (dependências SO)
├───────────────────────────┤
│ eclipse-temurin:21-jre    │  ← Camada base (imagem pai)
└───────────────────────────┘
```

### 3.2 Imagens base recomendadas

| Uso | Imagem | Tamanho aprox. |
|-----|--------|---------------|
| **Java 21 (build)** | `eclipse-temurin:21-jdk-alpine` | ~300 MB |
| **Java 21 (runtime)** | `eclipse-temurin:21-jre-alpine` | ~130 MB |
| **Node.js 22** | `node:22-alpine` | ~130 MB |
| **Python 3.12** | `python:3.12-slim` | ~150 MB |
| **Nginx** | `nginx:alpine` | ~45 MB |
| **PostgreSQL** | `postgres:16-alpine` | ~100 MB |
| **MySQL** | `mysql:8.4` | ~580 MB |
| **Redis** | `redis:7-alpine` | ~30 MB |
| **MongoDB** | `mongo:7` | ~700 MB |

> **Regra prática:** prefira variantes `-alpine` (baseadas em Alpine Linux, ~5 MB) ou `-slim` para produção. Use a variante completa apenas se precisar de ferramentas de diagnóstico.

### 3.3 Gerenciamento de imagens

```bash
# Listar imagens locais
docker images

# Baixar imagem do registry
docker pull postgres:16-alpine

# Remover imagem
docker rmi postgres:16-alpine

# Remover imagens não utilizadas (dangling)
docker image prune

# Remover TODAS as imagens não utilizadas
docker image prune -a

# Inspecionar camadas de uma imagem
docker history eclipse-temurin:21-jre-alpine

# Tag e push para registry
docker tag minha-app:latest registry.example.com/minha-app:1.0.0
docker push registry.example.com/minha-app:1.0.0
```

---

## 4. Container Registries

### 4.1 O que é um Registry

Um Container Registry é um repositório para armazenar, versionar e distribuir imagens Docker. Funciona como um "GitHub para imagens de container".

```
┌──────────────────────────────────────────────────────────────┐
│                  Fluxo com Registry                           │
│                                                               │
│  Desenvolvedor             Registry              Produção     │
│  ┌──────────┐            ┌──────────┐          ┌──────────┐ │
│  │ docker   │── push ──▶ │ registry │◀── pull ─│ K8s /    │ │
│  │ build    │            │ .com/    │          │ Swarm    │ │
│  │          │            │ app:1.0  │          │          │ │
│  └──────────┘            └──────────┘          └──────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Principais registries

| Registry | Provedor | Gratuito | Imagens privadas |
|----------|----------|----------|-----------------|
| **Docker Hub** | Docker | Sim (1 repo privado) | Pago |
| **GitHub Container Registry (GHCR)** | GitHub | Sim (repos públicos) | Sim (com limite de storage) |
| **AWS ECR** | Amazon | Não | Sim |
| **Google Artifact Registry** | Google Cloud | Não | Sim |
| **Azure Container Registry (ACR)** | Microsoft | Não | Sim |
| **Harbor** | CNCF (self-hosted) | Sim (open source) | Sim |
| **GitLab Container Registry** | GitLab | Sim | Sim |
| **Quay.io** | Red Hat | Sim | Sim |

### 4.3 Autenticação e push/pull

```bash
# === Docker Hub ===
docker login
docker tag minha-app:1.0 meuusuario/minha-app:1.0
docker push meuusuario/minha-app:1.0

# === GitHub Container Registry (GHCR) ===
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
docker tag minha-app:1.0 ghcr.io/minha-org/minha-app:1.0
docker push ghcr.io/minha-org/minha-app:1.0

# === AWS ECR ===
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456.dkr.ecr.us-east-1.amazonaws.com
docker tag minha-app:1.0 123456.dkr.ecr.us-east-1.amazonaws.com/minha-app:1.0
docker push 123456.dkr.ecr.us-east-1.amazonaws.com/minha-app:1.0

# === Google Artifact Registry ===
gcloud auth configure-docker us-central1-docker.pkg.dev
docker tag minha-app:1.0 us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:1.0
docker push us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:1.0

# === Logout ===
docker logout ghcr.io
```

### 4.4 ImagePullSecret no Kubernetes

Para que o cluster K8s faça pull de registries privados, é necessário criar um Secret de autenticação.

```bash
# Criar secret de autenticação
kubectl create secret docker-registry regcred \
  --docker-server=ghcr.io \
  --docker-username=meu-usuario \
  --docker-password=$GITHUB_TOKEN \
  --docker-email=meu@email.com
```

```yaml
# Referenciar no Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  template:
    spec:
      imagePullSecrets:
        - name: regcred
      containers:
        - name: api-app
          image: ghcr.io/minha-org/minha-app:1.0
```

### 4.5 Boas práticas de registry

| Prática | Motivo |
|---------|--------|
| Usar tags imutáveis (semver + SHA) | Evitar que `:latest` mude sem controle |
| Habilitar scan de vulnerabilidades no registry | Bloquear imagens com CVEs críticas antes do deploy |
| Configurar políticas de retenção | Limpar imagens antigas automaticamente |
| Usar registry próximo ao cluster | Reduzir latência de pull (mesma região cloud) |
| Assinar imagens (Cosign) | Garantir integridade e procedência |

---

## 5. Dockerfile — Construção de Imagens

### 5.1 Instruções do Dockerfile

| Instrução | Descrição | Exemplo |
|-----------|-----------|---------|
| `FROM` | Imagem base | `FROM eclipse-temurin:21-jre-alpine` |
| `WORKDIR` | Diretório de trabalho | `WORKDIR /app` |
| `COPY` | Copia arquivos do host | `COPY target/*.jar app.jar` |
| `ADD` | Como COPY, mas descompacta .tar e aceita URLs | `ADD config.tar.gz /etc/` |
| `RUN` | Executa comando durante build | `RUN apt-get update && apt-get install -y curl` |
| `ENV` | Define variável de ambiente | `ENV JAVA_OPTS="-Xmx512m"` |
| `ARG` | Argumento de build (não persiste no container) | `ARG JAR_FILE=app.jar` |
| `EXPOSE` | Documenta a porta (não publica) | `EXPOSE 8080` |
| `ENTRYPOINT` | Comando principal (não sobrescrito facilmente) | `ENTRYPOINT ["java", "-jar", "app.jar"]` |
| `CMD` | Argumentos default (sobrescrito via CLI) | `CMD ["--spring.profiles.active=prod"]` |
| `HEALTHCHECK` | Verificação de saúde | `HEALTHCHECK CMD curl -f http://localhost:8080/actuator/health` |
| `USER` | Usuário para execução | `USER appuser` |
| `VOLUME` | Ponto de montagem | `VOLUME /data` |
| `LABEL` | Metadados da imagem | `LABEL version="1.0"` |

### 5.2 Multi-stage build — Spring Boot (Maven)

```dockerfile
# === Etapa 1: Build ===
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app

COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN chmod +x mvnw && ./mvnw dependency:resolve -B

COPY src src
RUN ./mvnw package -DskipTests -B

# === Etapa 2: Runtime ===
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=build /app/target/*.jar app.jar

USER appuser
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-jar", "app.jar"]
```

### 5.3 Multi-stage build — Spring Boot (Gradle)

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app

COPY build.gradle.kts settings.gradle.kts gradlew ./
COPY gradle gradle
RUN chmod +x gradlew && ./gradlew dependencies --no-daemon

COPY src src
RUN ./gradlew bootJar --no-daemon -x test

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=build /app/build/libs/*.jar app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

### 5.4 Multi-stage build — React / Vite

```dockerfile
# === Build ===
FROM node:22-alpine AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# === Produção (Nginx) ===
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf — SPA com roteamento client-side
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend:8080/;
    }

    # Cache de assets estáticos
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2?)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 5.5 Multi-stage build — Angular

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npx ng build --configuration=production

FROM nginx:alpine
COPY --from=build /app/dist/meu-app/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 5.6 .dockerignore

```
# .dockerignore
target/
build/
node_modules/
dist/
.git/
.gitignore
*.md
.env
.env.*
.idea/
.vscode/
*.iml
Dockerfile
docker-compose*.yml
```

### 5.7 ENTRYPOINT vs CMD

```dockerfile
# ENTRYPOINT = comando fixo, CMD = argumentos default
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD ["--spring.profiles.active=prod"]

# Ao executar: docker run minha-app --spring.profiles.active=dev
# Resultado:   java -jar app.jar --spring.profiles.active=dev
#              CMD é sobrescrito, ENTRYPOINT mantido
```

| Cenário | ENTRYPOINT | CMD |
|---------|-----------|-----|
| Container sempre executa o mesmo binário | `["java", "-jar", "app.jar"]` | Argumentos variáveis |
| Container genérico/utilitário | Não usar | `["bash"]` |
| Sobrescrita total | `docker run --entrypoint sh ...` | `docker run img cmd` |

---

## 6. Otimização de Imagens Docker

### 6.1 Spring Boot Layered JARs

Spring Boot pode extrair o JAR em camadas, permitindo que dependências (que mudam raramente) fiquem em camadas Docker separadas do código da aplicação.

```dockerfile
# Dockerfile otimizado com layered JAR
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN chmod +x mvnw && ./mvnw dependency:resolve -B
COPY src src
RUN ./mvnw package -DskipTests -B

# Extrair camadas do JAR
FROM eclipse-temurin:21-jre-alpine AS layers
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# Imagem final com camadas otimizadas
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app

COPY --from=layers /app/dependencies/ ./
COPY --from=layers /app/spring-boot-loader/ ./
COPY --from=layers /app/snapshot-dependencies/ ./
COPY --from=layers /app/application/ ./

USER app
EXPOSE 8080
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "org.springframework.boot.loader.launch.JarLauncher"]
```

> **Vantagem:** quando apenas o código da aplicação muda, Docker reutiliza o cache das camadas de dependências — builds muito mais rápidos e push/pull menores.

### 6.2 Imagens Distroless (Google)

Imagens distroless contêm **apenas o runtime** — sem shell, sem gerenciador de pacotes, sem utilitários do SO. Superfície de ataque mínima.

```dockerfile
# Usar distroless como imagem final
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests -B

FROM gcr.io/distroless/java21-debian12
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

| Imagem | Tamanho | Shell? | Package manager? |
|--------|---------|--------|-----------------|
| `eclipse-temurin:21-jdk` | ~450 MB | Sim | Sim |
| `eclipse-temurin:21-jre-alpine` | ~130 MB | Sim | apk |
| `gcr.io/distroless/java21` | ~220 MB | Não | Não |
| `gcr.io/distroless/java21:debug` | ~240 MB | busybox | Não |

> **Trade-off:** sem shell, não é possível fazer `docker exec -it ... sh` para debug. Use a variante `:debug` em staging ou um pod efêmero de debug.

### 6.3 JLink — JRE customizado

JLink cria um JRE contendo apenas os módulos necessários para a aplicação, reduzindo drasticamente o tamanho.

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests -B

# Gerar JRE mínimo
RUN jlink \
  --add-modules java.base,java.logging,java.sql,java.naming,java.management,java.instrument,java.desktop,java.security.jgss \
  --strip-debug \
  --no-man-pages \
  --no-header-files \
  --compress=zip-6 \
  --output /custom-jre

FROM alpine:3.20
WORKDIR /app
COPY --from=build /custom-jre /opt/java
COPY --from=build /app/target/*.jar app.jar
ENV PATH="/opt/java/bin:$PATH"
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 6.4 BuildKit — Recursos avançados de build

```dockerfile
# syntax=docker/dockerfile:1

# Cache de dependências Maven persistente entre builds
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN --mount=type=cache,target=/root/.m2 \
    chmod +x mvnw && ./mvnw dependency:resolve -B
COPY src src
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw package -DskipTests -B

# Montar secret durante build (não persiste na imagem)
RUN --mount=type=secret,id=maven_settings,target=/root/.m2/settings.xml \
    ./mvnw deploy -B
```

```bash
# Habilitar BuildKit
export DOCKER_BUILDKIT=1
docker build -t minha-app:1.0 .

# Build com secret
docker build --secret id=maven_settings,src=settings.xml -t minha-app:1.0 .
```

### 6.5 Dive — Analisar camadas da imagem

```bash
# Instalar Dive
brew install dive          # macOS
# https://github.com/wagoodman/dive

# Analisar camadas
dive minha-app:1.0

# Modo CI — falha se eficiência for baixa
CI=true dive minha-app:1.0
```

### 6.6 Comparação de tamanhos

| Abordagem | Tamanho aprox. | Build speed |
|-----------|---------------|-------------|
| JDK completo (`temurin:21-jdk`) | ~450 MB | — |
| JRE Alpine (`temurin:21-jre-alpine`) | ~130 MB | Rápido |
| Layered JAR + JRE Alpine | ~130 MB | Rebuild rápido (cache) |
| Distroless (`distroless/java21`) | ~220 MB | Rápido |
| JLink customizado + Alpine | ~80-100 MB | Lento (jlink step) |

> **Recomendação:** para a maioria dos projetos, **JRE Alpine + multi-stage** é o melhor custo-benefício. Use layered JARs se rebuild speed for importante. Reserve JLink e distroless para produção com requisitos rigorosos de tamanho ou segurança.

---

## 7. Comandos Essenciais do Docker CLI

### 7.1 Ciclo de vida de containers

```bash
# Criar e executar container
docker run -d --name meu-app -p 8080:8080 minha-imagem:1.0

# Listar containers em execução
docker ps

# Listar todos (incluindo parados)
docker ps -a

# Parar container
docker stop meu-app

# Iniciar container parado
docker start meu-app

# Reiniciar
docker restart meu-app

# Remover container
docker rm meu-app

# Remover container em execução (forçar)
docker rm -f meu-app

# Remover todos os containers parados
docker container prune
```

### 7.2 Flags importantes do `docker run`

```bash
docker run \
  -d                           # Executar em background (detached)
  --name meu-app               # Nome do container
  -p 8080:8080                 # Mapear porta host:container
  -p 5432:5432/tcp             # Especificar protocolo
  -e SPRING_PROFILES_ACTIVE=prod  # Variável de ambiente
  --env-file .env              # Arquivo com variáveis de ambiente
  -v pgdata:/var/lib/data      # Volume nomeado
  -v $(pwd)/config:/app/config # Bind mount
  --network minha-rede         # Rede docker
  --restart unless-stopped     # Política de restart
  --memory 512m                # Limite de memória
  --cpus 1.5                   # Limite de CPU
  --health-cmd "curl -f http://localhost:8080/health" \
  minha-imagem:1.0
```

### 7.3 Logs e diagnóstico

```bash
# Ver logs do container
docker logs meu-app

# Logs em tempo real (follow)
docker logs -f meu-app

# Últimas 100 linhas
docker logs --tail 100 meu-app

# Executar comando dentro de container em execução
docker exec -it meu-app sh

# Estatísticas de uso de recursos
docker stats

# Inspecionar detalhes do container (JSON)
docker inspect meu-app

# Ver processos dentro do container
docker top meu-app

# Copiar arquivos de/para container
docker cp meu-app:/app/logs/app.log ./app.log
docker cp ./config.yml meu-app:/app/config.yml
```

### 7.4 Build

```bash
# Build com tag
docker build -t minha-app:1.0 .

# Build com Dockerfile em outro caminho
docker build -f docker/Dockerfile.prod -t minha-app:prod .

# Build com argumento
docker build --build-arg JAR_FILE=target/app.jar -t minha-app .

# Build sem cache
docker build --no-cache -t minha-app .

# Build multi-plataforma (requer buildx)
docker buildx build --platform linux/amd64,linux/arm64 -t minha-app:1.0 --push .
```

---

## 8. Volumes e Persistência de Dados

### 8.1 Tipos de armazenamento

```
┌─────────────────────────────────────────────────────┐
│                 Tipos de Storage                     │
│                                                      │
│  Named Volume          Bind Mount         tmpfs      │
│  ┌──────────┐    ┌─────────────────┐   ┌─────────┐ │
│  │ Docker   │    │ Diretório do    │   │ Memória │ │
│  │ gerencia │    │ host mapeado    │   │  (RAM)  │ │
│  │ /var/lib/│    │ diretamente     │   │         │ │
│  │docker/   │    │                 │   │         │ │
│  │volumes/  │    │ Ex: ./src:/app  │   │         │ │
│  └──────────┘    └─────────────────┘   └─────────┘ │
│  Produção ✓       Desenvolvimento ✓    Dados temp ✓│
└─────────────────────────────────────────────────────┘
```

| Tipo | Uso | Exemplo |
|------|-----|---------|
| **Named Volume** | Dados persistentes gerenciados pelo Docker | `pgdata:/var/lib/postgresql/data` |
| **Bind Mount** | Desenvolvimento — monta diretório do host | `./src:/app/src` |
| **tmpfs** | Dados temporários em memória | `--tmpfs /tmp` |

### 8.2 Comandos de volume

```bash
# Criar volume
docker volume create pgdata

# Listar volumes
docker volume ls

# Inspecionar
docker volume inspect pgdata

# Remover
docker volume rm pgdata

# Remover volumes não utilizados
docker volume prune
```

---

## 9. Redes Docker

### 9.1 Tipos de rede

| Driver | Descrição | Uso |
|--------|-----------|-----|
| **bridge** | Rede isolada (default) | Containers no mesmo host |
| **host** | Compartilha rede do host | Performance máxima, sem isolamento |
| **overlay** | Rede entre múltiplos hosts | Docker Swarm / clusters |
| **none** | Sem rede | Containers totalmente isolados |

### 9.2 Rede bridge customizada

```bash
# Criar rede
docker network create minha-rede

# Executar containers na mesma rede
docker run -d --name db --network minha-rede postgres:16-alpine
docker run -d --name app --network minha-rede -e DB_HOST=db minha-app

# Listar redes
docker network ls

# Inspecionar rede
docker network inspect minha-rede

# Conectar container a rede existente
docker network connect minha-rede outro-container
```

> **DNS automático:** Em redes bridge customizadas, containers podem se comunicar pelo nome. O container `app` acessa o banco via `jdbc:postgresql://db:5432/appdb`.

---

## 10. Docker Compose

### 10.1 O que é Docker Compose

Docker Compose é uma ferramenta para definir e gerenciar aplicações multi-container usando um arquivo YAML declarativo (`docker-compose.yml` ou `compose.yml`).

```
┌──────────────────────────────────────────────────┐
│              docker-compose.yml                    │
│                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  app     │  │   db     │  │  redis   │        │
│  │ build: . │  │ postgres │  │  redis:7 │        │
│  │ port:8080│──│ port:5432│  │ port:6379│        │
│  └──────────┘  └──────────┘  └──────────┘        │
│         │              │                           │
│         └──── rede compartilhada ────┘            │
│                        │                           │
│                   volumes                          │
└──────────────────────────────────────────────────┘
```

### 10.2 Estrutura do compose.yml

```yaml
# compose.yml (nome preferido desde Compose V2)
name: meu-projeto  # opcional — define prefixo dos recursos

services:
  # Cada serviço = 1 container
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: minha-app:latest
    container_name: minha-app
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
    driver: bridge
```

### 10.3 Comandos do Docker Compose

```bash
# Subir todos os serviços (build + start)
docker compose up -d

# Subir reconstruindo imagens
docker compose up -d --build

# Parar e remover containers, redes
docker compose down

# Parar e remover incluindo volumes
docker compose down -v

# Ver logs de todos os serviços
docker compose logs -f

# Logs de um serviço específico
docker compose logs -f app

# Ver status dos serviços
docker compose ps

# Executar comando em serviço
docker compose exec app sh

# Escalar serviço
docker compose up -d --scale app=3

# Rebuild de um serviço específico
docker compose build app

# Pull de imagens atualizadas
docker compose pull

# Restart de serviço específico
docker compose restart app
```

### 10.4 Variáveis de ambiente e `.env`

```bash
# .env (carregado automaticamente pelo Compose)
DB_PASSWORD=minha_senha_segura
SPRING_PROFILES_ACTIVE=dev
APP_VERSION=1.2.0
```

```yaml
# Referência no compose.yml
services:
  app:
    image: minha-app:${APP_VERSION}
    environment:
      # Referência direta
      SPRING_PROFILES_ACTIVE: ${SPRING_PROFILES_ACTIVE}
      # Com valor default
      DB_PASSWORD: ${DB_PASSWORD:-fallback_password}
```

### 10.5 Profiles — Serviços opcionais

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"

  db:
    image: postgres:16-alpine
    # sempre sobe (sem profile = ativo por default)

  adminer:
    image: adminer
    ports:
      - "9090:8080"
    profiles:
      - debug

  prometheus:
    image: prom/prometheus
    profiles:
      - monitoring

  grafana:
    image: grafana/grafana
    profiles:
      - monitoring
```

```bash
# Sobe apenas app + db
docker compose up -d

# Sobe app + db + adminer
docker compose --profile debug up -d

# Sobe app + db + prometheus + grafana
docker compose --profile monitoring up -d

# Sobe tudo
docker compose --profile debug --profile monitoring up -d
```

### 10.6 Depends_on com condições

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy      # espera healthcheck passar
      redis:
        condition: service_started      # espera container iniciar
      migrations:
        condition: service_completed_successfully  # espera terminar com sucesso
```

### 10.7 Múltiplos arquivos Compose (override)

```bash
# compose.yml        → configuração base
# compose.override.yml → sobrescrita para desenvolvimento (carregado automaticamente)
# compose.prod.yml   → sobrescrita para produção

# Desenvolvimento (merge automático de compose.yml + compose.override.yml)
docker compose up -d

# Produção (especificar arquivos explicitamente)
docker compose -f compose.yml -f compose.prod.yml up -d
```

```yaml
# compose.override.yml — desenvolvimento
services:
  app:
    build: .
    volumes:
      - ./src:/app/src  # hot reload
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DEVTOOLS_RESTART_ENABLED: "true"
    ports:
      - "8080:8080"
      - "5005:5005"  # debug remoto
    command: ["java", "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005", "-jar", "app.jar"]
```

```yaml
# compose.prod.yml — produção
services:
  app:
    image: registry.example.com/minha-app:${TAG}
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: prod
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 1G
          cpus: "2.0"
```

---

## 11. Docker Compose — Exemplos Práticos

### 11.1 Stack completa: Spring Boot + PostgreSQL + Redis + RabbitMQ

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      SPRING_DATA_REDIS_HOST: redis
      SPRING_RABBITMQ_HOST: rabbitmq
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
      rabbitmq:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --maxmemory 128mb --maxmemory-policy allkeys-lru
    volumes:
      - redisdata:/data

  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "15672:15672"  # painel de gerenciamento
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: ${RABBIT_PASSWORD}
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
  redisdata:
```

### 11.2 Stack frontend + backend + banco

```yaml
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

### 11.3 Stack de observabilidade

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - promdata:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafdata:/var/lib/grafana
    depends_on:
      - prometheus

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"

volumes:
  promdata:
  grafdata:
```

### 11.4 Banco de dados para desenvolvimento rápido

```yaml
# Subir apenas infraestrutura local para desenvolvimento
services:
  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: devdb
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/01-init.sql

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: dev
      MONGO_INITDB_ROOT_PASSWORD: dev
    volumes:
      - mongodata:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

volumes:
  pgdata:
  mongodata:
```

---

## 12. Docker Compose — YAML Anchors, Limitação de Recursos e Réplicas

### 12.1 YAML Anchors (`&`) e Aliases (`*`) — Reaproveitamento de configuração

YAML Anchors permitem definir um bloco de configuração uma vez (`&nome`) e reutilizá-lo em outros lugares (`*nome`), evitando repetição. O operador `<<:` (merge key) combina o bloco referenciado com propriedades adicionais.

```yaml
# compose.yml — sem anchors (muita repetição)
services:
  api:
    image: minha-app:1.0
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
      LOG_LEVEL: INFO
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
    restart: unless-stopped
    networks:
      - backend

  worker:
    image: minha-app:1.0
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
      LOG_LEVEL: INFO
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
    restart: unless-stopped
    networks:
      - backend
```

```yaml
# compose.yml — com anchors (DRY — Don't Repeat Yourself)

# Blocos reutilizáveis definidos no nível raiz (x- = extensão, ignorada pelo Compose)
x-app-common: &app-common
  image: minha-app:1.0
  restart: unless-stopped
  networks:
    - backend

x-app-env: &app-env
  SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
  SPRING_DATASOURCE_USERNAME: postgres
  SPRING_DATASOURCE_PASSWORD: secret
  LOG_LEVEL: INFO

x-resource-limits: &resource-limits
  deploy:
    resources:
      limits:
        memory: 512M
        cpus: "1.0"
      reservations:
        memory: 256M
        cpus: "0.5"

services:
  api:
    <<: *app-common           # merge: image, restart, networks
    <<: *resource-limits      # merge: deploy.resources
    ports:
      - "8080:8080"
    environment:
      <<: *app-env            # merge: todas as variáveis
      SPRING_PROFILES_ACTIVE: api   # adiciona/sobrescreve

  worker:
    <<: *app-common
    <<: *resource-limits
    environment:
      <<: *app-env
      SPRING_PROFILES_ACTIVE: worker
    command: ["java", "-jar", "app.jar", "--worker"]

  scheduler:
    <<: *app-common
    environment:
      <<: *app-env
      SPRING_PROFILES_ACTIVE: scheduler
    deploy:
      resources:
        limits:
          memory: 256M        # scheduler precisa de menos memória
          cpus: "0.5"

networks:
  backend:
```

> **Nota:** como o YAML padrão não permite dois `<<:` no mesmo nível, a forma mais portável é aninhar as anchors nas extensões `x-` ou usar um único bloco de merge.

### 12.2 Extensões `x-` — Blocos reutilizáveis

Chaves prefixadas com `x-` no nível raiz do `compose.yml` são ignoradas pelo Docker Compose, servindo como local para definir anchors.

```yaml
x-logging: &default-logging
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

x-healthcheck-spring: &spring-healthcheck
  healthcheck:
    test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
    interval: 30s
    timeout: 5s
    retries: 3
    start_period: 40s

services:
  api:
    image: minha-app:1.0
    <<: *default-logging
    <<: *spring-healthcheck
    ports:
      - "8080:8080"

  worker:
    image: minha-app:1.0
    <<: *default-logging
    <<: *spring-healthcheck
    command: ["java", "-jar", "app.jar", "--worker"]
```

### 12.3 Limitação de recursos (CPU, Memória)

```yaml
services:
  app:
    image: minha-app:1.0
    deploy:
      resources:
        # Limites máximos — container é morto (OOMKill) se exceder
        limits:
          memory: 512M
          cpus: "1.0"        # equivale a 1 core
        # Reservas — recursos garantidos pelo scheduler
        reservations:
          memory: 256M
          cpus: "0.25"

  db:
    image: postgres:16-alpine
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: "2.0"
        reservations:
          memory: 512M
          cpus: "0.5"

  redis:
    image: redis:7-alpine
    deploy:
      resources:
        limits:
          memory: 128M
          cpus: "0.5"
```

| Propriedade | Descrição |
|-------------|-----------|
| `limits.memory` | Máximo de memória — container é terminado (OOMKill) se ultrapassar |
| `limits.cpus` | Máximo de CPU — `"1.5"` = 1,5 cores; `"0.5"` = metade de um core |
| `reservations.memory` | Memória garantida — scheduler reserva esse mínimo |
| `reservations.cpus` | CPU garantida — não compartilhada com outros containers |

> **`limits` vs `reservations`:** limits define o teto; reservations define o piso garantido. Se o host não tem recursos para as reservations, o container não inicia.

#### Unidades de memória

| Unidade | Exemplo | Valor |
|---------|---------|-------|
| `b` | `536870912b` | bytes |
| `k` / `kb` | `524288k` | kilobytes |
| `m` / `mb` | `512m` | megabytes |
| `g` / `gb` | `1g` | gigabytes |

### 12.4 Réplicas

```yaml
services:
  app:
    image: minha-app:1.0
    deploy:
      replicas: 3             # 3 instâncias do container
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
    ports:
      - "8080"                # porta do host atribuída automaticamente
                              # (sem fixar porta do host quando há réplicas)
```

```bash
# Escalar via CLI
docker compose up -d --scale app=5

# Verificar réplicas
docker compose ps
```

> **Atenção com portas:** ao usar réplicas, não fixe a porta do host (`"8080:8080"`), pois causará conflito. Use apenas a porta do container (`"8080"`) e acesse via load balancer ou Docker Swarm.

### 12.5 Restart policies

```yaml
services:
  app:
    image: minha-app:1.0
    restart: unless-stopped    # recomendado para produção

  worker:
    image: minha-app:1.0
    restart: on-failure        # reinicia apenas em caso de erro
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

| Política | Comportamento |
|----------|--------------|
| `no` | Nunca reinicia (default) |
| `always` | Sempre reinicia, inclusive após reboot do host |
| `unless-stopped` | Como `always`, mas não reinicia se parado manualmente |
| `on-failure` | Reinicia apenas se sair com código de erro (exit ≠ 0) |

### 12.6 Exemplo completo com anchors, recursos e réplicas

```yaml
x-app-defaults: &app-defaults
  image: registry.example.com/minha-app:${TAG:-latest}
  restart: unless-stopped
  networks:
    - backend
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

x-spring-env: &spring-env
  SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
  SPRING_DATASOURCE_USERNAME: ${DB_USER:-postgres}
  SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
  SPRING_DATA_REDIS_HOST: redis

x-standard-resources: &standard-resources
  resources:
    limits:
      memory: 512M
      cpus: "1.0"
    reservations:
      memory: 256M
      cpus: "0.25"

x-small-resources: &small-resources
  resources:
    limits:
      memory: 256M
      cpus: "0.5"
    reservations:
      memory: 128M
      cpus: "0.1"

services:
  api:
    <<: *app-defaults
    ports:
      - "8080:8080"
    environment:
      <<: *spring-env
      SPRING_PROFILES_ACTIVE: api
    deploy:
      replicas: 3
      <<: *standard-resources
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/actuator/health"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 30s

  worker:
    <<: *app-defaults
    environment:
      <<: *spring-env
      SPRING_PROFILES_ACTIVE: worker
    deploy:
      replicas: 2
      <<: *standard-resources
    command: ["java", "-jar", "app.jar", "--spring.profiles.active=worker"]

  scheduler:
    <<: *app-defaults
    environment:
      <<: *spring-env
      SPRING_PROFILES_ACTIVE: scheduler
    deploy:
      replicas: 1
      <<: *small-resources

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    deploy:
      <<: *standard-resources
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    command: redis-server --maxmemory 64mb --maxmemory-policy allkeys-lru
    deploy:
      <<: *small-resources
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
    driver: bridge
```

---

## 13. Docker Swarm

### 13.1 O que é Docker Swarm

Docker Swarm é o orquestrador de containers nativo do Docker. Transforma um grupo de hosts Docker em um cluster, com deploy declarativo, load balancing, service discovery e auto-healing integrados.

```
┌──────────────────────────────────────────────────────────────┐
│                     Docker Swarm Cluster                       │
│                                                               │
│  ┌──────────────────── Manager Nodes ────────────────────┐   │
│  │                                                        │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐      │   │
│  │  │  Manager 1 │  │  Manager 2 │  │  Manager 3 │      │   │
│  │  │  (Leader)  │  │ (Reachable)│  │ (Reachable)│      │   │
│  │  │  Raft      │  │  Raft      │  │  Raft      │      │   │
│  │  └────────────┘  └────────────┘  └────────────┘      │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────── Worker Nodes ─────────────────────┐   │
│  │                                                        │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐      │   │
│  │  │  Worker 1  │  │  Worker 2  │  │  Worker 3  │      │   │
│  │  │ ┌────┐┌───┐│  │ ┌────┐    │  │ ┌────┐┌───┐│      │   │
│  │  │ │ T1 ││T2 ││  │ │ T3 │    │  │ │ T4 ││T5 ││      │   │
│  │  │ └────┘└───┘│  │ └────┘    │  │ └────┘└───┘│      │   │
│  │  └────────────┘  └────────────┘  └────────────┘      │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                               │
│  T = Task (instância de um Service = container em execução)   │
└──────────────────────────────────────────────────────────────┘
```

### 13.2 Swarm vs Kubernetes — Quando usar cada um

| Aspecto | Docker Swarm | Kubernetes |
|---------|-------------|------------|
| **Complexidade** | Baixa — embutido no Docker | Alta — instalação e config separada |
| **Curva de aprendizado** | Rápida — usa Docker Compose | Longa — novos conceitos (Pods, etc.) |
| **Escala** | Pequena/média (até ~100 nodes) | Grande (milhares de nodes) |
| **Ecossistema** | Limitado | Extenso (Helm, Operators, CRDs) |
| **Auto-scaling** | Manual | HPA, VPA, Cluster Autoscaler |
| **Service mesh** | Básico (routing mesh) | Istio, Linkerd, Cilium |
| **Adoção** | Projetos pequenos, times enxutos | Empresas, cloud-native, produção em larga escala |
| **Deploy** | `docker stack deploy` | `kubectl apply` / `helm install` |

> **Regra prática:** Swarm é suficiente para projetos pequenos/médios que já usam Docker Compose. Para produção em larga escala, multi-cloud ou ecossistema avançado, Kubernetes é a escolha padrão.

### 13.3 Inicializar e gerenciar o cluster

```bash
# === Inicializar Swarm (no manager principal) ===
docker swarm init --advertise-addr 192.168.1.10

# O comando acima retorna um token para adicionar workers:
# docker swarm join --token SWMTKN-1-xxx 192.168.1.10:2377

# Gerar token para adicionar workers
docker swarm join-token worker

# Gerar token para adicionar managers
docker swarm join-token manager

# === No worker — ingressar no cluster ===
docker swarm join --token SWMTKN-1-xxx 192.168.1.10:2377

# === Gerenciar nodes ===
docker node ls                          # listar nodes do cluster
docker node inspect worker-1            # detalhes de um node
docker node promote worker-1            # promover worker a manager
docker node demote manager-2            # rebaixar manager a worker
docker node update --availability drain worker-1  # drenar node (manutenção)

# === Sair do Swarm ===
docker swarm leave          # no worker
docker swarm leave --force  # no último manager
```

### 13.4 Services no Swarm

```bash
# Criar service
docker service create \
  --name api-app \
  --replicas 3 \
  --publish 8080:8080 \
  --limit-cpu 1.0 \
  --limit-memory 512M \
  --reserve-cpu 0.25 \
  --reserve-memory 256M \
  --restart-condition on-failure \
  --restart-max-attempts 3 \
  --update-parallelism 1 \
  --update-delay 10s \
  --env SPRING_PROFILES_ACTIVE=prod \
  registry.example.com/minha-app:1.0

# Listar services
docker service ls

# Ver tasks (containers) de um service
docker service ps api-app

# Logs de um service
docker service logs -f api-app

# Escalar service
docker service scale api-app=5

# Atualizar imagem (rolling update)
docker service update --image minha-app:1.1 api-app

# Rollback
docker service rollback api-app

# Remover service
docker service rm api-app
```

### 13.5 Docker Stack — Deploy via Compose no Swarm

Docker Stack permite fazer deploy de uma aplicação multi-container no Swarm usando o mesmo `compose.yml`, com a seção `deploy` controlando comportamento de produção.

```yaml
# stack.yml
services:
  api:
    image: registry.example.com/minha-app:1.0
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
    deploy:
      replicas: 3
      update_config:
        parallelism: 1        # atualiza 1 container por vez
        delay: 10s            # espera 10s entre cada atualização
        failure_action: rollback
        order: start-first    # inicia novo antes de parar o antigo
      rollback_config:
        parallelism: 1
        delay: 5s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
        reservations:
          memory: 256M
          cpus: "0.25"
      placement:
        constraints:
          - node.role == worker
        preferences:
          - spread: node.id   # distribui entre nodes
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/actuator/health"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 30s

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - pgdata:/var/lib/postgresql/data
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          memory: 1G
          cpus: "2.0"
    secrets:
      - db_password
    networks:
      - backend

  redis:
    image: redis:7-alpine
    deploy:
      replicas: 1
      resources:
        limits:
          memory: 128M
          cpus: "0.5"
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
    driver: overlay          # rede overlay para comunicação entre nodes

secrets:
  db_password:
    external: true           # secret criado previamente via docker secret create
```

```bash
# Criar secret
echo "minha_senha_segura" | docker secret create db_password -

# Deploy da stack
docker stack deploy -c stack.yml minha-stack

# Listar stacks
docker stack ls

# Ver services da stack
docker stack services minha-stack

# Ver tasks (containers) da stack
docker stack ps minha-stack

# Remover stack
docker stack rm minha-stack

# Listar secrets
docker secret ls
```

### 13.6 Seção `deploy` — Referência completa

| Propriedade | Descrição |
|-------------|-----------|
| `replicas` | Número de instâncias do service |
| `resources.limits.memory` | Limite máximo de memória |
| `resources.limits.cpus` | Limite máximo de CPU |
| `resources.reservations.memory` | Memória reservada (garantida) |
| `resources.reservations.cpus` | CPU reservada (garantida) |
| `update_config.parallelism` | Quantos containers atualizar simultaneamente |
| `update_config.delay` | Intervalo entre atualizações |
| `update_config.failure_action` | `pause`, `continue` ou `rollback` |
| `update_config.order` | `stop-first` (default) ou `start-first` (zero-downtime) |
| `rollback_config` | Configuração de rollback (mesmo formato do update_config) |
| `restart_policy.condition` | `none`, `on-failure` ou `any` |
| `restart_policy.max_attempts` | Máximo de tentativas de restart |
| `restart_policy.window` | Janela de tempo para contar tentativas |
| `placement.constraints` | Restrições de node (`node.role == worker`, `node.labels.zone == us-east`) |
| `placement.preferences` | Preferências de distribuição entre nodes |

### 13.7 Redes Overlay no Swarm

```bash
# Criar rede overlay (disponível em todos os nodes)
docker network create --driver overlay --attachable minha-rede

# Service discovery automático — containers acessam pelo nome do service
# Ex: api acessa banco via "db:5432"
```

```
┌───────────────────────────────────────────────────────────┐
│                  Routing Mesh do Swarm                     │
│                                                            │
│  Requisição externa                                        │
│       │                                                    │
│       ▼  (qualquer node, qualquer IP)                      │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐             │
│  │ Node 1  │     │ Node 2  │     │ Node 3  │             │
│  │ :8080 ──┼─────┼─────────┼─────┼─▶ api-1 │             │
│  │         │     │ api-2   │     │         │              │
│  └─────────┘     └─────────┘     └─────────┘             │
│                                                            │
│  O Swarm roteia automaticamente para um container          │
│  saudável, independente de em qual node a requisição       │
│  chegou (routing mesh / ingress network)                   │
└───────────────────────────────────────────────────────────┘
```

### 13.8 Monitoramento do cluster

```bash
# Status dos nodes
docker node ls

# Recursos usados por service
docker service ps api-app --format "table {{.Name}}\t{{.Node}}\t{{.CurrentState}}"

# Inspecionar service
docker service inspect --pretty api-app

# Logs centralizados
docker service logs -f --tail 100 api-app

# Monitoramento com Prometheus (config para scrape do Swarm)
# docker_sd_configs no prometheus.yml
```

---

## 14. Boas Práticas de Docker

### 14.1 Construção de imagens

| Prática | Por quê |
|---------|---------|
| Usar **multi-stage build** | Imagem final não contém JDK, node_modules ou código-fonte |
| Imagens **alpine/slim** | Menor tamanho e superfície de ataque |
| **Usuário não-root** | Princípio do menor privilégio — `RUN adduser` + `USER` |
| **`.dockerignore`** abrangente | Evita copiar `.git/`, `target/`, `node_modules/` para o build context |
| Ordenar instruções por **frequência de mudança** | Dependências antes de código → cache eficiente |
| **Um processo por container** | Facilita logging, scaling e lifecycle |
| **`HEALTHCHECK`** no Dockerfile | Orquestrador sabe quando container está pronto |
| Não armazenar **secrets** na imagem | Usar variáveis de ambiente, Docker Secrets ou vault externo |
| **Fixar versões** de imagens base | `postgres:16.3-alpine`, nunca `postgres:latest` |

### 14.2 Cache de build — Ordem importa

```dockerfile
# ❌ Ruim — qualquer mudança em src/ invalida cache de dependências
COPY . .
RUN ./mvnw package -DskipTests

# ✅ Bom — dependências em cache separado
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN ./mvnw dependency:resolve -B    # cache preservado se pom.xml não mudar
COPY src src
RUN ./mvnw package -DskipTests -B   # só recompila o código
```

### 14.3 Segurança

```dockerfile
# Nunca hardcode secrets
ENV DB_PASSWORD=minha_senha    # ❌

# Usar variáveis em runtime
# docker run -e DB_PASSWORD=xxx  # ✅

# Scan de vulnerabilidades
# docker scout cves minha-app:1.0
# trivy image minha-app:1.0
```

### 14.4 Tagging de imagens

```bash
# Usar tags semânticas — nunca depender apenas de :latest
docker build -t minha-app:1.2.0 -t minha-app:latest .

# Convenção recomendada
# minha-app:1.2.0        → versão exata (imutável)
# minha-app:1.2          → patch mais recente da minor
# minha-app:latest       → build mais recente
# minha-app:main-abc1234 → branch + SHA do commit
```

### 14.5 JVM em containers — Flags recomendadas

```dockerfile
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:InitialRAMPercentage=50.0", \
  "-XX:+UseG1GC", \
  "-XX:+ExitOnOutOfMemoryError", \
  "-jar", "app.jar"]
```

| Flag | Função |
|------|--------|
| `+UseContainerSupport` | JVM respeita limites de memória/CPU do container (default em Java 17+) |
| `MaxRAMPercentage=75` | Usa até 75% da memória do container para heap |
| `+UseG1GC` | Garbage collector otimizado para latência |
| `+ExitOnOutOfMemoryError` | Container morre e orquestrador reinicia em vez de ficar em estado inconsistente |

---

## 15. Segurança e Análise de Vulnerabilidades em Containers

### 15.1 Superfície de ataque em containers

```
┌──────────────────────────────────────────────────────────────────────┐
│              Superfície de Ataque em Containers                       │
│                                                                       │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────────┐  │
│  │  Imagem Base     │  │   Dependências   │  │  Configuração      │  │
│  │                  │  │                  │  │                    │  │
│  │ • CVEs no SO     │  │ • Libs com vuln. │  │ • Root no runtime  │  │
│  │ • Pacotes        │  │ • JARs/npm/pip   │  │ • Secrets na image │  │
│  │   desnecessários │  │   desatualizados │  │ • Portas expostas  │  │
│  │ • Versão antiga  │  │ • Transitividade │  │ • Capabilities     │  │
│  └─────────────────┘  └──────────────────┘  └────────────────────┘  │
│                                                                       │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────────┐  │
│  │  Registry        │  │   Runtime        │  │  Orquestração      │  │
│  │                  │  │                  │  │                    │  │
│  │ • Supply chain   │  │ • Container      │  │ • RBAC fraco       │  │
│  │   (imagem        │  │   escape         │  │ • Secrets sem      │  │
│  │   comprometida)  │  │ • Privilege      │  │   criptografia     │  │
│  │ • Sem assinatura │  │   escalation     │  │ • Network policies │  │
│  └─────────────────┘  └──────────────────┘  └────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

### 15.2 Ferramentas de análise de vulnerabilidades

| Ferramenta | Tipo | Descrição | Licença |
|------------|------|-----------|---------|
| **Trivy** | Scanner completo | Vulnerabilidades em imagens, IaC, Dockerfile, secrets e SBOM | Open Source (Aqua Security) |
| **Docker Scout** | Scanner integrado | Análise de CVEs nativa do Docker Desktop e CLI | Gratuito (limites) / Pago |
| **Grype** | Scanner de imagens | Scan de vulnerabilidades rápido e preciso — complemento ao Syft | Open Source (Anchore) |
| **Syft** | Gerador de SBOM | Gera Software Bill of Materials de imagens e filesystems | Open Source (Anchore) |
| **Snyk Container** | Scanner + monitoramento | Vulnerabilidades com recomendações de fix e monitoramento contínuo | Gratuito (limites) / Pago |
| **Clair** | Scanner server-side | Análise estática de vulnerabilidades em registries (Quay, Harbor) | Open Source (Red Hat) |
| **Dockle** | Linter de imagem | Verifica boas práticas e CIS Benchmark no Dockerfile e imagem | Open Source |
| **Hadolint** | Linter de Dockerfile | Análise estática do Dockerfile com regras de boas práticas | Open Source |
| **Cosign** | Assinatura de imagem | Assinatura e verificação de imagens com Sigstore | Open Source (Sigstore) |
| **Falco** | Runtime security | Detecção de comportamento anômalo em containers em execução | Open Source (Sysdig/CNCF) |
| **KubeAudit** | Auditor K8s | Audita manifestos Kubernetes contra boas práticas de segurança | Open Source |
| **Kubescape** | Scanner K8s | Verifica conformidade com NSA, MITRE, CIS Benchmarks em clusters K8s | Open Source (ARMO) |

### 15.3 Trivy — Scanner mais popular

Trivy é o scanner mais adotado pela comunidade: analisa imagens Docker, repositórios Git, filesystems, configurações IaC (Terraform, Kubernetes) e Dockerfiles em uma única ferramenta.

```bash
# Instalar (macOS)
brew install trivy

# Instalar (Linux)
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# === Scan de imagem Docker ===
trivy image minha-app:1.0

# Scan apenas para vulnerabilidades críticas e altas
trivy image --severity CRITICAL,HIGH minha-app:1.0

# Scan com saída em tabela (default) ou JSON
trivy image -f json -o resultado.json minha-app:1.0

# Ignorar vulnerabilidades sem fix disponível
trivy image --ignore-unfixed minha-app:1.0

# === Scan de Dockerfile ===
trivy config Dockerfile

# === Scan de repositório local (código + dependências) ===
trivy fs .

# === Scan de manifestos Kubernetes ===
trivy config k8s/

# === Gerar SBOM ===
trivy image --format spdx-json -o sbom.json minha-app:1.0
```

Exemplo de saída do Trivy:

```
minha-app:1.0 (alpine 3.19.1)
==============================
Total: 5 (CRITICAL: 1, HIGH: 2, MEDIUM: 2, LOW: 0)

┌───────────────┬──────────────────┬──────────┬────────────────┬───────────────┐
│   Library     │  Vulnerability   │ Severity │ Installed Ver  │  Fixed Ver    │
├───────────────┼──────────────────┼──────────┼────────────────┼───────────────┤
│ libcrypto3    │ CVE-2024-XXXXX   │ CRITICAL │ 3.1.4-r1       │ 3.1.4-r5      │
│ libssl3       │ CVE-2024-YYYYY   │ HIGH     │ 3.1.4-r1       │ 3.1.4-r5      │
│ curl          │ CVE-2024-ZZZZZ   │ HIGH     │ 8.5.0-r0       │ 8.6.0-r0      │
└───────────────┴──────────────────┴──────────┴────────────────┴───────────────┘

Java (jar)
==========
Total: 3 (HIGH: 1, MEDIUM: 2)

┌──────────────────────────┬──────────────────┬──────────┬─────────┬──────────┐
│        Library           │  Vulnerability   │ Severity │ Version │ Fixed    │
├──────────────────────────┼──────────────────┼──────────┼─────────┼──────────┤
│ org.springframework:     │ CVE-2024-AAAAA   │ HIGH     │ 6.1.4   │ 6.1.6   │
│   spring-web             │                  │          │         │          │
└──────────────────────────┴──────────────────┴──────────┴─────────┴──────────┘
```

### 15.4 Docker Scout

```bash
# Scan de imagem (requer login no Docker Hub)
docker scout cves minha-app:1.0

# Recomendações de atualização de imagem base
docker scout recommendations minha-app:1.0

# Comparar vulnerabilidades entre versões
docker scout compare minha-app:1.0 --to minha-app:1.1

# Quickview — resumo rápido
docker scout quickview minha-app:1.0
```

### 15.5 Grype e Syft

```bash
# Grype — scan de vulnerabilidades
grype minha-app:1.0

# Scan apenas para severidade alta e crítica
grype minha-app:1.0 --only-fixed --fail-on high

# Syft — gerar SBOM (Software Bill of Materials)
syft minha-app:1.0

# SBOM em formato CycloneDX (padrão para compliance)
syft minha-app:1.0 -o cyclonedx-json > sbom.json

# Pipeline combinado: Syft gera SBOM → Grype analisa
syft minha-app:1.0 -o json | grype
```

### 15.6 Snyk Container

Snyk é uma plataforma de segurança que combina scan de vulnerabilidades com monitoramento contínuo, recomendações de fix e integração nativa com repositórios Git e pipelines CI/CD. Diferente de scanners pontuais, Snyk monitora as imagens após o deploy e alerta quando novas CVEs são publicadas.

```bash
# Instalar CLI
npm install -g snyk
# ou
brew install snyk

# Autenticar (cria conta gratuita se necessário)
snyk auth

# === Scan de imagem Docker ===
snyk container test minha-app:1.0

# Scan com severidade mínima
snyk container test minha-app:1.0 --severity-threshold=high

# Scan com recomendação de imagem base alternativa
snyk container test minha-app:1.0 --file=Dockerfile

# Saída em JSON (útil para automação)
snyk container test minha-app:1.0 --json > snyk-report.json

# === Monitoramento contínuo ===
# Registra a imagem no Snyk — recebe alertas quando novas CVEs surgem
snyk container monitor minha-app:1.0

# Monitorar vinculando ao Dockerfile (recomendações de upgrade da base)
snyk container monitor minha-app:1.0 --file=Dockerfile

# === Scan de código e dependências ===
# Vulnerabilidades em dependências do projeto (pom.xml, package.json, etc.)
snyk test

# Monitorar dependências continuamente
snyk monitor

# === Scan de IaC (Dockerfile, K8s, Terraform) ===
snyk iac test Dockerfile
snyk iac test k8s/
```

Exemplo de saída do Snyk:

```
Testing minha-app:1.0...

Organization:      minha-org
Package manager:   apk
Target file:       Dockerfile
Project name:      docker-image|minha-app
Docker image:      minha-app:1.0
Platform:          linux/amd64
Base image:        eclipse-temurin:21-jre-alpine

✗ High severity vulnerability found in openssl/libcrypto3
  Description: Buffer Overflow
  Introduced through: openssl/libcrypto3@3.1.4-r1
  Fixed in: 3.1.4-r5

✗ Medium severity vulnerability found in spring-web
  Introduced through: org.springframework:spring-web@6.1.4
  Fixed in: 6.1.6

Tested 42 dependencies for known vulnerabilities, found 5 vulnerabilities.

Recommendations for base image upgrade:
  Minor upgrades: eclipse-temurin:21.0.4_7-jre-alpine
    ✓ Reduces vulnerabilities from 5 to 0
```

#### Snyk no CI/CD (GitHub Actions)

```yaml
# .github/workflows/snyk.yml
name: Snyk Security

on:
  push:
    branches: [main]
  pull_request:

jobs:
  snyk-container:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t minha-app:${{ github.sha }} .

      - name: Snyk Container scan
        uses: snyk/actions/docker@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          image: minha-app:${{ github.sha }}
          args: --severity-threshold=high --file=Dockerfile

  snyk-code:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Snyk Open Source (dependências)
        uses: snyk/actions/maven@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Snyk IaC (Dockerfile + K8s)
        uses: snyk/actions/iac@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=medium
```

#### Integração com IDE e repositório

| Integração | Descrição |
|------------|-----------|
| **VS Code / IntelliJ** | Plugin Snyk exibe vulnerabilidades diretamente no editor |
| **GitHub / GitLab / Bitbucket** | Scan automático em PRs; cria PRs de fix automaticamente |
| **Dashboard web** | Visão consolidada de todas as imagens e projetos monitorados |
| **Slack / Jira** | Notificações de novas vulnerabilidades |

#### Plano gratuito vs pago

| Recurso | Gratuito | Team / Enterprise |
|---------|----------|-------------------|
| Scans de container | 100/mês | Ilimitados |
| Scans de código | 200/mês | Ilimitados |
| Monitoramento contínuo | Sim (projetos limitados) | Sim (ilimitado) |
| Fix PRs automáticos | Sim | Sim |
| Políticas de segurança | Não | Sim |
| SSO / RBAC | Não | Sim |
| Relatórios de compliance | Não | Sim |

> **Snyk vs Trivy:** Trivy é gratuito, rápido e ideal para scan pontual em CI/CD. Snyk se diferencia pelo **monitoramento contínuo**, **PRs de fix automáticos** e **dashboard centralizado** — mais adequado para equipes que precisam de visibilidade e gestão contínua de vulnerabilidades.

### 15.7 Hadolint e Dockle — Linters

```bash
# === Hadolint — lint do Dockerfile ===
hadolint Dockerfile

# Com nível de severidade mínimo
hadolint --failure-threshold warning Dockerfile

# Ignorar regras específicas
hadolint --ignore DL3008 --ignore DL3018 Dockerfile
```

Exemplos de regras do Hadolint:

| Regra | Descrição |
|-------|-----------|
| `DL3006` | Sempre fixar tag na instrução `FROM` |
| `DL3008` | Fixar versão de pacotes em `apt-get install` |
| `DL3018` | Fixar versão de pacotes em `apk add` |
| `DL3025` | Usar forma JSON no `CMD`/`ENTRYPOINT` |
| `DL4006` | Definir `SHELL` com `pipefail` para detectar falhas em pipes |
| `SC2086` | Usar aspas duplas para evitar word splitting |

```bash
# === Dockle — auditoria de imagem construída ===
dockle minha-app:1.0

# Com saída JSON
dockle -f json -o dockle-report.json minha-app:1.0
```

Exemplos de verificações do Dockle:

| Nível | Verificação |
|-------|------------|
| **FATAL** | Executa como root; credenciais na imagem |
| **WARN** | Sem `HEALTHCHECK`; usando `ADD` em vez de `COPY` |
| **INFO** | Sem label `maintainer`; imagem base sem tag fixa |

### 15.8 Assinatura e verificação de imagens (Cosign)

```bash
# Gerar par de chaves
cosign generate-key-pair

# Assinar imagem
cosign sign --key cosign.key registry.example.com/minha-app:1.0

# Verificar assinatura
cosign verify --key cosign.pub registry.example.com/minha-app:1.0

# Assinar sem chave (keyless — via OIDC/Sigstore)
cosign sign registry.example.com/minha-app:1.0

# Anexar SBOM à imagem
cosign attach sbom --sbom sbom.json registry.example.com/minha-app:1.0
```

### 15.9 Segurança em runtime — Falco

Falco detecta comportamentos anômalos em containers em execução, como execução de shell, leitura de arquivos sensíveis ou conexões de rede inesperadas.

```yaml
# Exemplo de regra Falco — detectar shell em container de produção
- rule: Shell em container de produção
  desc: Detecta abertura de shell em container que não deveria ter acesso interativo
  condition: >
    spawned_process and
    container and
    proc.name in (bash, sh, zsh) and
    not container.image.repository in (debug-tools)
  output: "Shell aberto em container (user=%user.name container=%container.name image=%container.image.repository)"
  priority: WARNING
```

### 15.10 Boas práticas de segurança em Dockerfile

```dockerfile
# ✅ 1. Usar imagem base mínima e com tag fixa
FROM eclipse-temurin:21.0.4_7-jre-alpine

# ✅ 2. Não instalar pacotes desnecessários
RUN apk add --no-cache curl

# ✅ 3. Criar e usar usuário não-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# ✅ 4. Nunca usar ADD para URLs (usar COPY + download explícito)
COPY app.jar /app/app.jar

# ✅ 5. Usar COPY --chmod em vez de RUN chmod (menos camadas)
COPY --chmod=555 entrypoint.sh /app/entrypoint.sh

# ✅ 6. Definir HEALTHCHECK
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# ✅ 7. Usar forma exec (JSON) no ENTRYPOINT
ENTRYPOINT ["java", "-jar", "/app/app.jar"]

# ✅ 8. Definir labels de metadados
LABEL org.opencontainers.image.source="https://github.com/org/repo" \
      org.opencontainers.image.version="1.0.0"
```

```dockerfile
# ❌ Práticas inseguras a evitar
FROM ubuntu:latest                            # tag instável, imagem grande
RUN apt-get install -y python3 vim nano curl  # pacotes desnecessários
COPY .env /app/.env                           # secrets na imagem
ENV DB_PASSWORD=super_secret                  # secrets em ENV
RUN chmod 777 /app                            # permissões excessivas
USER root                                     # execução como root
ADD https://example.com/file.tar.gz /app/     # download sem verificação
```

### 15.11 Execução segura de containers

```bash
# ✅ Rodar com capabilities reduzidas
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE minha-app:1.0

# ✅ Filesystem somente leitura
docker run --read-only --tmpfs /tmp minha-app:1.0

# ✅ Sem escalação de privilégios
docker run --security-opt=no-new-privileges minha-app:1.0

# ✅ Limitar recursos
docker run --memory=512m --cpus=1.0 --pids-limit=100 minha-app:1.0

# ✅ Combinação recomendada para produção
docker run -d \
  --name minha-app \
  --user 1000:1000 \
  --cap-drop=ALL \
  --read-only \
  --tmpfs /tmp \
  --security-opt=no-new-privileges \
  --memory=512m \
  --cpus=1.0 \
  --pids-limit=100 \
  -p 8080:8080 \
  minha-app:1.0
```

| Flag | Proteção |
|------|----------|
| `--cap-drop=ALL` | Remove todas as Linux capabilities (ex: não pode montar FS, mudar dono de arquivos) |
| `--cap-add=NET_BIND_SERVICE` | Adiciona apenas o necessário (bind em portas < 1024) |
| `--read-only` | Filesystem do container em modo somente leitura |
| `--tmpfs /tmp` | Diretório temporário em memória (necessário para apps que escrevem em /tmp) |
| `--security-opt=no-new-privileges` | Previne escalação de privilégios via `setuid`/`setgid` |
| `--pids-limit` | Limita processos no container (proteção contra fork bomb) |

### 15.12 Integração em pipeline CI/CD

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint-dockerfile:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hadolint/hadolint-action@v3
        with:
          dockerfile: Dockerfile
          failure-threshold: warning

  scan-image:
    runs-on: ubuntu-latest
    needs: [lint-dockerfile]
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t minha-app:${{ github.sha }} .

      - name: Dockle — Auditoria de imagem
        uses: erzz/dockle-action@v1
        with:
          image: minha-app:${{ github.sha }}
          failure-threshold: warn

      - name: Trivy — Vulnerabilidades
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: minha-app:${{ github.sha }}
          format: table
          severity: CRITICAL,HIGH
          exit-code: 1   # falha o build se encontrar CRITICAL/HIGH

      - name: Trivy — Scan de IaC (Dockerfile + K8s)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: config
          scan-ref: .
          severity: CRITICAL,HIGH
```

### 15.13 Comparação de ferramentas

| Aspecto | Trivy | Docker Scout | Grype | Snyk | Clair |
|---------|-------|-------------|-------|------|-------|
| **Scan de imagem** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Scan de Dockerfile** | ✅ | ❌ | ❌ | ✅ | ❌ |
| **Scan de IaC/K8s** | ✅ | ❌ | ❌ | ✅ | ❌ |
| **Scan de código/deps** | ✅ | ❌ | ❌ | ✅ | ❌ |
| **Geração de SBOM** | ✅ | ✅ | Via Syft | ✅ | ❌ |
| **Monitoramento contínuo** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Fix PRs automáticos** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Plugin IDE** | Limitado | ❌ | ❌ | ✅ (VS Code, IntelliJ) | ❌ |
| **Integração CI/CD** | GitHub Actions, GitLab | Docker Hub, CLI | CLI, CI | SaaS, CLI, CI | Registry |
| **Velocidade** | Rápido | Rápido | Muito rápido | Médio | Lento |
| **Custo** | Gratuito | Freemium | Gratuito | Freemium | Gratuito |
| **Melhor para** | Uso geral, CI/CD | Quem já usa Docker Hub | Scan leve e rápido | Monitoramento contínuo, gestão centralizada | Registries privados |

> **Recomendação para começar:** use **Hadolint** para lint do Dockerfile + **Trivy** para scan de vulnerabilidades em CI/CD (gratuito e rápido). Se precisar de **monitoramento contínuo**, dashboard centralizado e fix PRs automáticos, adicione **Snyk** ao workflow.

---

## 16. Testcontainers — Testes de Integração com Docker

### 16.1 O que é Testcontainers

Testcontainers é uma biblioteca que cria containers Docker descartáveis durante testes de integração, garantindo que os testes usem serviços reais (PostgreSQL, Redis, Kafka) em vez de mocks.

```xml
<!-- Dependências Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

### 16.2 Teste de integração com PostgreSQL

```java
// Spring Boot 3.1+ com @ServiceConnection (configuração automática)
@SpringBootTest
@Testcontainers
class UsuarioRepositoryTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private UsuarioRepository repository;

    @Test
    void deveSalvarEBuscarUsuario() {
        var usuario = new Usuario("João", "joao@email.com");
        repository.save(usuario);

        var encontrado = repository.findByEmail("joao@email.com");
        assertThat(encontrado).isPresent();
        assertThat(encontrado.get().getNome()).isEqualTo("João");
    }
}
```

### 16.3 Abordagem com @DynamicPropertySource (Spring Boot < 3.1)

```java
@SpringBootTest
@Testcontainers
class PedidoServiceTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    static GenericContainer<?> redis =
        new GenericContainer<>("redis:7-alpine").withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
    }
}
```

### 16.4 Containers disponíveis

| Módulo | Container | Dependência Maven |
|--------|-----------|-------------------|
| `postgresql` | PostgreSQL | `org.testcontainers:postgresql` |
| `mysql` | MySQL | `org.testcontainers:mysql` |
| `mongodb` | MongoDB | `org.testcontainers:mongodb` |
| `kafka` | Apache Kafka | `org.testcontainers:kafka` |
| `rabbitmq` | RabbitMQ | `org.testcontainers:rabbitmq` |
| `elasticsearch` | Elasticsearch | `org.testcontainers:elasticsearch` |
| `localstack` | AWS (S3, SQS, etc.) | `org.testcontainers:localstack` |
| `GenericContainer` | Qualquer imagem Docker | `org.testcontainers:testcontainers` |

### 16.5 Classe base reutilizável

```java
// Classe base que todos os testes de integração estendem
@SpringBootTest
@Testcontainers
public abstract class IntegrationTestBase {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withInitScript("init-test.sql");

    @Container
    @ServiceConnection
    static GenericContainer<?> redis =
        new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);
}

// Testes individuais herdam a base
class PedidoServiceTest extends IntegrationTestBase {
    @Test
    void deveProcessarPedido() { /* ... */ }
}
```

> **Docker obrigatório:** Testcontainers requer Docker rodando na máquina. Em CI/CD (GitHub Actions, GitLab CI), containers Docker estão disponíveis por padrão.

---

## 17. Alternativas ao Docker

### 17.1 Ecossistema de containers além do Docker

| Ferramenta | Tipo | Descrição |
|------------|------|-----------|
| **Podman** | Container engine | Compatível com Docker, sem daemon, rootless por padrão |
| **Buildah** | Build de imagens | Constrói imagens OCI sem daemon, sem Dockerfile |
| **containerd** | Container runtime | Runtime que Docker usa internamente; padrão no K8s |
| **CRI-O** | Container runtime | Runtime leve criado especificamente para Kubernetes |
| **nerdctl** | CLI | CLI compatível com Docker para containerd |
| **Kaniko** | Build em CI/CD | Build de imagens dentro de containers (sem Docker-in-Docker) |

### 17.2 Podman — A principal alternativa

Podman é compatível com a CLI do Docker (mesmos comandos), mas sem daemon central e com suporte nativo a execução rootless (sem privilégios de root).

```bash
# Comandos Docker funcionam com Podman
podman build -t minha-app:1.0 .
podman run -d -p 8080:8080 minha-app:1.0
podman ps
podman logs meu-container
podman images

# Compose com Podman
podman compose up -d

# Rootless por padrão — não precisa de sudo
podman run --rm alpine whoami    # roda como usuário comum

# Gerar manifesto K8s a partir de container rodando
podman generate kube meu-container > pod.yaml
```

| Aspecto | Docker | Podman |
|---------|--------|--------|
| **Daemon** | Sim (dockerd) | Não (daemonless) |
| **Rootless** | Opt-in (config extra) | Default |
| **Compatibilidade CLI** | — | Quase 100% (`alias docker=podman`) |
| **Compose** | Docker Compose | `podman compose` ou `podman-compose` |
| **Desktop GUI** | Docker Desktop | Podman Desktop |
| **Licença** | Gratuito (Desktop pago para empresas 250+) | Totalmente gratuito (Apache 2.0) |
| **Padrão em** | Maioria dos tutoriais e docs | RHEL, Fedora, CentOS |

### 17.3 Kaniko — Build em CI/CD sem Docker

```yaml
# GitHub Actions — build com Kaniko (sem Docker-in-Docker)
- name: Build with Kaniko
  uses: aevea/action-kaniko@master
  with:
    image: minha-app
    tag: ${{ github.sha }}
    registry: ghcr.io/${{ github.repository_owner }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

### 17.4 Quando usar cada ferramenta

| Cenário | Recomendação |
|---------|-------------|
| Desenvolvimento local (qualquer SO) | Docker Desktop ou Podman Desktop |
| Ambiente corporativo (custo de licença) | Podman (gratuito) |
| Runtime em Kubernetes | containerd (default) ou CRI-O |
| Build em pipeline CI/CD (sem Docker) | Kaniko |
| RHEL/Fedora como SO do servidor | Podman (pré-instalado) |
| Aprendizado e tutoriais | Docker (mais documentação disponível) |

---

## 18. Kubernetes — Arquitetura e Conceitos

### 18.1 Visão geral da arquitetura

```
┌──────────────────────────────────────────────────────────────────────┐
│                          Cluster Kubernetes                           │
│                                                                       │
│  ┌─────────────────────────── Control Plane ───────────────────────┐ │
│  │                                                                  │ │
│  │  ┌────────────┐  ┌───────────┐  ┌─────────────────┐  ┌──────┐ │ │
│  │  │ API Server │  │ Scheduler │  │ Controller      │  │ etcd │ │ │
│  │  │ (kube-api) │  │           │  │ Manager         │  │      │ │ │
│  │  │            │  │ Decide em │  │ Reconcilia      │  │ Key- │ │ │
│  │  │ Ponto de   │  │ qual node │  │ estado atual    │  │Value │ │ │
│  │  │ entrada    │  │ o Pod roda│  │ vs desejado     │  │Store │ │ │
│  │  └────────────┘  └───────────┘  └─────────────────┘  └──────┘ │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌─────────────────────────── Worker Nodes ────────────────────────┐ │
│  │                                                                  │ │
│  │  ┌──────────── Node 1 ──────────┐  ┌──────── Node 2 ─────────┐│ │
│  │  │  kubelet    kube-proxy        │  │  kubelet    kube-proxy   ││ │
│  │  │  ┌──────┐  ┌──────┐  ┌─────┐│  │  ┌──────┐  ┌──────┐     ││ │
│  │  │  │Pod 1 │  │Pod 2 │  │Pod 3││  │  │Pod 4 │  │Pod 5 │     ││ │
│  │  │  │┌────┐│  │┌────┐│  │┌───┐││  │  │┌────┐│  │┌────┐│     ││ │
│  │  │  ││ C1 ││  ││ C1 ││  ││C1 │││  │  ││ C1 ││  ││ C1 ││     ││ │
│  │  │  │└────┘│  │└────┘│  │└───┘││  │  │└────┘│  │└────┘│     ││ │
│  │  │  └──────┘  └──────┘  └─────┘│  │  └──────┘  └──────┘     ││ │
│  │  └──────────────────────────────┘  └──────────────────────────┘│ │
│  └──────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### 18.2 Componentes do Control Plane

| Componente | Responsabilidade |
|------------|-----------------|
| **API Server** | Ponto de entrada para todas as operações (kubectl, dashboard, CI/CD) |
| **etcd** | Armazena todo o estado do cluster (key-value distribuído) |
| **Scheduler** | Decide em qual node um Pod será executado, baseado em recursos e afinidade |
| **Controller Manager** | Executa loops de controle — garante que estado atual = estado desejado |
| **Cloud Controller** | Integra com provedores cloud (AWS, GCP, Azure) para LB, volumes, etc. |

### 18.3 Componentes dos Worker Nodes

| Componente | Responsabilidade |
|------------|-----------------|
| **kubelet** | Agente que garante que os containers dos Pods estejam rodando |
| **kube-proxy** | Gerencia regras de rede para comunicação com os Pods |
| **Container Runtime** | Executa containers (containerd, CRI-O) |

### 18.4 Recursos principais do Kubernetes

| Recurso | Descrição | Analogia |
|---------|-----------|----------|
| **Pod** | Menor unidade deployável — 1+ containers compartilhando rede e storage | "Instância" |
| **ReplicaSet** | Garante N réplicas de um Pod | "Auto-healing" |
| **Deployment** | Gerencia ReplicaSets com rolling updates | "Deployment CI/CD" |
| **StatefulSet** | Pods com identidade persistente e storage dedicado | "Bancos de dados" |
| **DaemonSet** | Um Pod em cada node do cluster | "Agentes (logs, monitoring)" |
| **Job** | Executa tarefa até conclusão | "Batch / Cron" |
| **CronJob** | Job agendado por expressão cron | "Crontab" |
| **Service** | Abstração de rede estável para acessar Pods | "Load Balancer interno" |
| **Ingress** | Roteamento HTTP/HTTPS externo | "Reverse proxy" |
| **ConfigMap** | Configuração como key-value | "application.properties" |
| **Secret** | Dados sensíveis (base64) | "Variáveis de ambiente seguras" |
| **PVC** | Solicitação de armazenamento persistente | "Volume de disco" |
| **Namespace** | Isolamento lógico de recursos | "Ambiente/tenant" |
| **HPA** | Horizontal Pod Autoscaler | "Auto-scaling" |
| **NetworkPolicy** | Regras de firewall entre Pods | "Security groups" |
| **ServiceAccount** | Identidade para Pods acessarem a API | "IAM role" |

---

## 19. Kubectl — Comandos Essenciais

### 19.1 Configuração

```bash
# Verificar cluster e contexto
kubectl cluster-info
kubectl config current-context
kubectl config get-contexts

# Mudar de contexto
kubectl config use-context meu-cluster

# Mudar namespace default
kubectl config set-context --current --namespace=meu-namespace
```

### 19.2 Operações CRUD

```bash
# Aplicar manifesto (criar ou atualizar)
kubectl apply -f deployment.yaml

# Aplicar todos os manifestos de um diretório
kubectl apply -f k8s/

# Listar recursos
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get all                    # pods, services, deployments, replicasets

# Com mais detalhes
kubectl get pods -o wide           # IP, node
kubectl get pods -o yaml           # YAML completo
kubectl get pods -o json           # JSON completo

# Descrever recurso (eventos, condições, detalhes)
kubectl describe pod meu-pod
kubectl describe service meu-svc

# Deletar recurso
kubectl delete -f deployment.yaml
kubectl delete pod meu-pod
kubectl delete deployment meu-deploy

# Editar recurso em tempo real
kubectl edit deployment meu-deploy
```

### 19.3 Diagnóstico e debug

```bash
# Logs de um pod
kubectl logs meu-pod
kubectl logs meu-pod -f                    # follow
kubectl logs meu-pod -c meu-container      # container específico (multi-container pod)
kubectl logs meu-pod --previous            # logs do container anterior (crashloop)

# Executar shell em pod
kubectl exec -it meu-pod -- sh
kubectl exec -it meu-pod -c meu-container -- bash

# Port-forward para acesso local
kubectl port-forward pod/meu-pod 8080:8080
kubectl port-forward svc/meu-service 8080:80

# Ver eventos do cluster
kubectl get events --sort-by=.metadata.creationTimestamp

# Uso de recursos por node
kubectl top nodes

# Uso de recursos por pod
kubectl top pods

# Debug com pod temporário
kubectl run debug --image=busybox --rm -it -- sh
kubectl run debug --image=nicolaka/netshoot --rm -it -- bash
```

### 19.4 Atalhos úteis

```bash
# Aliases comuns (adicionar ao .bashrc/.zshrc)
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete -f'
alias klogs='kubectl logs -f'

# Autocompletion
source <(kubectl completion bash)   # bash
source <(kubectl completion zsh)    # zsh
```

---

## 20. Pods e Workloads

### 20.1 Pod básico (raramente usado diretamente)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  labels:
    app: minha-app
spec:
  containers:
    - name: app
      image: minha-app:1.0
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
```

### 20.2 Deployment (workload mais comum)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
  labels:
    app: api-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1           # cria 1 Pod a mais durante update
      maxUnavailable: 0     # nunca ter menos do que replicas desejadas
  template:
    metadata:
      labels:
        app: api-app
    spec:
      containers:
        - name: api-app
          image: registry.example.com/api-app:1.0.0
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: app-config
            - secretRef:
                name: app-secrets
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            failureThreshold: 30
            periodSeconds: 2
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
      terminationGracePeriodSeconds: 30
```

### 20.3 Probes — Verificações de saúde

```
┌────────────────────────────────────────────────────────────────┐
│                    Lifecycle de um Pod                          │
│                                                                 │
│  Container    startupProbe      readinessProbe   livenessProbe │
│  inicia   ──▶ (app iniciou?) ──▶ (pode receber ──▶ (ainda     │
│               Se falhar N vezes:  tráfego?)        funciona?)  │
│               restart container   Se falhar:       Se falhar:  │
│                                   remove do        restart     │
│                                   Service          container   │
└────────────────────────────────────────────────────────────────┘
```

| Probe | Propósito | Efeito ao falhar |
|-------|-----------|-----------------|
| **startupProbe** | Aplicação terminou de iniciar? | Reinicia container (apps lentas para iniciar) |
| **readinessProbe** | Pode receber tráfego? | Remove dos endpoints do Service (sem tráfego) |
| **livenessProbe** | Ainda está funcionando? | Reinicia container |

### 20.4 StatefulSet — Para aplicações com estado

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: pgdata
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 10Gi
```

> **StatefulSet vs Deployment:** StatefulSet garante nomes estáveis (`postgres-0`, `postgres-1`), ordem de criação/destruição e volumes individuais por Pod — essencial para bancos de dados e sistemas distribuídos.

### 20.5 Job e CronJob

```yaml
# Job — executa uma vez até conclusão
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
        - name: migrate
          image: minha-app:1.0
          command: ["java", "-jar", "app.jar", "--spring.batch.job.name=migration"]
      restartPolicy: Never
---
# CronJob — execução agendada
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-job
spec:
  schedule: "0 2 * * *"  # todo dia às 2h
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cleanup
              image: minha-app:1.0
              command: ["java", "-jar", "app.jar", "--job=cleanup"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
```

---

## 21. Init Containers e Padrões Multi-container

### 21.1 Init Containers

Init containers executam **antes** dos containers da aplicação e devem terminar com sucesso. São usados para setup, migrations ou espera por dependências.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  template:
    spec:
      initContainers:
        # 1. Esperar banco ficar disponível
        - name: wait-for-db
          image: busybox
          command: ['sh', '-c',
            'until nc -z postgres 5432; do echo "Aguardando DB..."; sleep 2; done']

        # 2. Executar migrations
        - name: run-migrations
          image: minha-app:1.0
          command: ['java', '-jar', 'app.jar', '--spring.flyway.enabled=true']
          env:
            - name: SPRING_DATASOURCE_URL
              value: jdbc:postgresql://postgres:5432/appdb

      containers:
        - name: api-app
          image: minha-app:1.0
```

| Característica | Init Container | Container principal |
|---------------|---------------|-------------------|
| **Execução** | Antes da app, sequencial | Após init, em paralelo |
| **Deve terminar?** | Sim (exit 0) | Não (fica rodando) |
| **Reinicia se falhar?** | Sim (Pod reinicia todos os inits) | Conforme restartPolicy |
| **Uso comum** | Migrations, wait-for-deps, setup | Aplicação principal |

### 21.2 Padrão Sidecar

Container auxiliar que roda ao lado do container principal, adicionando funcionalidade sem alterar a aplicação.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  template:
    spec:
      containers:
        - name: api-app
          image: minha-app:1.0
          ports:
            - containerPort: 8080
          volumeMounts:
            - name: logs
              mountPath: /app/logs

        # Sidecar: coleta e encaminha logs
        - name: log-forwarder
          image: fluent/fluent-bit
          volumeMounts:
            - name: logs
              mountPath: /app/logs
              readOnly: true

      volumes:
        - name: logs
          emptyDir: {}
```

**Exemplos comuns de sidecar:**

| Sidecar | Função |
|---------|--------|
| **Envoy/Istio proxy** | Service mesh — mTLS, routing, métricas |
| **Fluent Bit** | Coleta e encaminhamento de logs |
| **CloudSQL Proxy** | Proxy seguro para bancos em cloud |
| **Vault Agent** | Injeta secrets do HashiCorp Vault |
| **OAuth2 Proxy** | Autenticação na frente da aplicação |

### 21.3 Outros padrões

```
┌──────────────────────────────────────────────────────────┐
│               Padrões Multi-container                     │
│                                                           │
│  Ambassador            Adapter              Init          │
│  ┌─────┐  ┌──────┐   ┌─────┐  ┌──────┐   ┌──────┐     │
│  │ App │─▶│Proxy │   │ App │─▶│Adapt.│   │ Init │     │
│  └─────┘  └──┬───┘   └─────┘  └──┬───┘   └──┬───┘     │
│              │                    │           │ (exit 0) │
│         mundo externo      formato padrão    ▼           │
│  (API Gateway local)    (logs, métricas)   ┌─────┐      │
│                                             │ App │      │
│                                             └─────┘      │
└──────────────────────────────────────────────────────────┘
```

| Padrão | Descrição | Exemplo |
|--------|-----------|---------|
| **Sidecar** | Adiciona funcionalidade ao container principal | Log forwarder, proxy, monitoring agent |
| **Ambassador** | Proxy para comunicação com o mundo externo | API gateway local, connection pooling |
| **Adapter** | Padroniza saída do container principal | Converte formato de logs/métricas para padrão |
| **Init** | Executa setup antes do container principal | Migrations, download de config, wait-for-deps |

---

## 22. Services e Networking no Kubernetes

### 22.1 Tipos de Service

```
┌─────────────────────────────────────────────────────────────┐
│                  Tipos de Service                            │
│                                                              │
│  Internet                                                    │
│      │                                                       │
│      ▼                                                       │
│  ┌───────────────┐                                          │
│  │ LoadBalancer   │  ← IP público (cloud provider)          │
│  │ (externo)      │                                          │
│  └───────┬───────┘                                          │
│          ▼                                                   │
│  ┌───────────────┐                                          │
│  │   NodePort     │  ← Porta fixa em todos os nodes         │
│  │ (30000-32767)  │     (menos usado em prod)               │
│  └───────┬───────┘                                          │
│          ▼                                                   │
│  ┌───────────────┐                                          │
│  │  ClusterIP     │  ← Acessível apenas dentro do cluster   │
│  │ (default)      │     (comunicação entre serviços)        │
│  └───────┬───────┘                                          │
│          ▼                                                   │
│   ┌─────┐ ┌─────┐ ┌─────┐                                  │
│   │Pod 1│ │Pod 2│ │Pod 3│                                  │
│   └─────┘ └─────┘ └─────┘                                  │
└─────────────────────────────────────────────────────────────┘
```

### 22.2 ClusterIP (default — comunicação interna)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP   # default, pode omitir
  selector:
    app: api-app
  ports:
    - port: 80          # porta do Service
      targetPort: 8080   # porta do container
      protocol: TCP
```

Outros Pods acessam via `http://api-service` ou `http://api-service.meu-namespace.svc.cluster.local`.

### 22.3 NodePort (acesso externo por porta do node)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
spec:
  type: NodePort
  selector:
    app: api-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080   # porta acessível em qualquer IP de node
```

### 22.4 LoadBalancer (cloud provider)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: api-app
  ports:
    - port: 80
      targetPort: 8080
```

### 22.5 Headless Service (para StatefulSets)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None   # headless — DNS retorna IPs individuais dos Pods
  selector:
    app: postgres
  ports:
    - port: 5432
```

Cada Pod recebe DNS próprio: `postgres-0.postgres-headless.namespace.svc.cluster.local`.

### 22.6 DNS interno do Kubernetes

| Recurso | DNS |
|---------|-----|
| Service (mesmo namespace) | `meu-service` |
| Service (outro namespace) | `meu-service.outro-ns.svc.cluster.local` |
| Pod (StatefulSet headless) | `pod-name.service-name.namespace.svc.cluster.local` |

---

## 23. Ingress e Roteamento HTTP

### 23.1 Conceito

Ingress é um recurso que gerencia acesso HTTP/HTTPS externo aos Services do cluster, com roteamento baseado em host e path.

```
┌──────────────────────────────────────────────────────────────┐
│  Internet                                                     │
│      │                                                        │
│      ▼                                                        │
│  ┌──────────────────────────────┐                            │
│  │    Ingress Controller         │  (Nginx, Traefik, etc.)   │
│  │    (reverse proxy + TLS)      │                            │
│  └──────────┬───────────────────┘                            │
│             │                                                 │
│     ┌───────┴────────────┐                                   │
│     │                    │                                    │
│     ▼                    ▼                                    │
│  api.example.com    app.example.com                          │
│     │                    │                                    │
│     ▼                    ▼                                    │
│  ┌────────┐         ┌────────┐                               │
│  │API Svc │         │App Svc │                               │
│  │ :8080  │         │  :80   │                               │
│  └────────┘         └────────┘                               │
└──────────────────────────────────────────────────────────────┘
```

### 23.2 Ingress por path

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: meusite.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### 23.3 Ingress com TLS (HTTPS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress-tls
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - meusite.example.com
      secretName: meusite-tls
  rules:
    - host: meusite.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### 23.4 Ingress Controllers populares

| Controller | Características |
|------------|----------------|
| **NGINX Ingress** | Mais usado; maduro e bem documentado |
| **Traefik** | Auto-discovery; configuração dinâmica; bom para K8s |
| **HAProxy** | Alta performance; enterprise |
| **AWS ALB Ingress** | Integrado com AWS Application Load Balancer |
| **Istio Gateway** | Service mesh com observabilidade avançada |

---

## 24. Service Mesh

### 24.1 O que é Service Mesh

Service Mesh é uma camada de infraestrutura dedicada à comunicação entre serviços, fornecendo observabilidade, segurança (mTLS) e controle de tráfego de forma transparente — sem alterar código da aplicação.

```
┌──────────────────────────────────────────────────────────────┐
│                     Service Mesh                              │
│                                                               │
│  Sem mesh:      Service A ──────────────▶ Service B          │
│                 (HTTP direto, sem criptografia)               │
│                                                               │
│  Com mesh:      Service A    Sidecar A    Sidecar B   Svc B  │
│                 ┌──────┐    ┌──────┐     ┌──────┐   ┌─────┐ │
│                 │ App  │───▶│Proxy │═mTLS═│Proxy │──▶│ App │ │
│                 └──────┘    └──────┘     └──────┘   └─────┘ │
│                              Envoy        Envoy              │
│                                                               │
│                 ┌────────────────────────────────┐            │
│                 │       Control Plane             │            │
│                 │  (Istiod, Linkerd, Cilium)      │            │
│                 │  Configuração, certificados,    │            │
│                 │  políticas de tráfego           │            │
│                 └────────────────────────────────┘            │
└──────────────────────────────────────────────────────────────┘
```

### 24.2 Funcionalidades

| Funcionalidade | Descrição |
|---------------|-----------|
| **mTLS automático** | Criptografia e autenticação mútua entre todos os serviços |
| **Traffic splitting** | Direcionar % do tráfego para diferentes versões (canary) |
| **Circuit breaker** | Interromper chamadas a serviços com falha |
| **Retry e timeout** | Políticas automáticas de retry com backoff |
| **Observabilidade** | Métricas, traces e logs de toda comunicação entre serviços |
| **Rate limiting** | Limitar taxa de requisições por serviço |
| **Fault injection** | Injetar falhas para testes de resiliência |

### 24.3 Principais implementações

| Service Mesh | Proxy | Destaques |
|-------------|-------|-----------|
| **Istio** | Envoy (sidecar) | Mais funcionalidades; complexo; grande comunidade |
| **Linkerd** | linkerd2-proxy (Rust, sidecar) | Leve, rápido, fácil de operar; CNCF graduated |
| **Cilium** | eBPF (sem sidecar) | Performance superior; observabilidade com Hubble; sidecar-free |
| **Consul Connect** | Envoy (sidecar) | Integrado com ecossistema HashiCorp |

### 24.4 Exemplo básico com Istio

```bash
# Instalar Istio
istioctl install --set profile=demo
kubectl label namespace production istio-injection=enabled

# Pods no namespace "production" recebem sidecar Envoy automaticamente
```

```yaml
# VirtualService — routing de tráfego (canary com Istio)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-app
spec:
  hosts:
    - api-app
  http:
    - route:
        - destination:
            host: api-app
            subset: stable
          weight: 90
        - destination:
            host: api-app
            subset: canary
          weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-app
spec:
  host: api-app
  subsets:
    - name: stable
      labels:
        version: v1
    - name: canary
      labels:
        version: v2
```

### 24.5 Quando usar Service Mesh

| Cenário | Recomendação |
|---------|-------------|
| < 5 microsserviços | **Não usar** — complexidade não compensa |
| Necessidade de mTLS entre serviços | **Sim** — Linkerd (mais simples) ou Istio |
| Canary deploy avançado | **Sim** — controle fino do tráfego |
| Observabilidade entre serviços | **Considerar** — Cilium com Hubble é leve |
| Monolito ou poucos serviços | **Não usar** |

> **Recomendação para começar:** Linkerd tem a menor curva de aprendizado. Cilium é a escolha moderna para clusters que já usam eBPF. Istio é o mais completo, mas também o mais complexo.

---

## 25. ConfigMaps e Secrets

### 25.1 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  SPRING_PROFILES_ACTIVE: "prod"
  SERVER_PORT: "8080"
  LOG_LEVEL: "INFO"
  # Arquivo de configuração completo
  application.yml: |
    spring:
      jpa:
        hibernate:
          ddl-auto: validate
        show-sql: false
    logging:
      level:
        root: INFO
```

### 25.2 Secret

```bash
# Criar Secret via kubectl
kubectl create secret generic db-credentials \
  --from-literal=username=postgres \
  --from-literal=password=minha_senha_segura

# Criar a partir de arquivo
kubectl create secret tls meu-tls \
  --cert=cert.pem \
  --key=key.pem

# Criar a partir de .env
kubectl create secret generic app-secrets \
  --from-env-file=.env.prod
```

```yaml
# Secret via manifesto (valores em base64)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: cG9zdGdyZXM=       # echo -n 'postgres' | base64
  password: bWluaGFfc2VuaGE=   # echo -n 'minha_senha' | base64
```

### 25.3 Usando ConfigMap e Secret no Deployment

```yaml
spec:
  containers:
    - name: app
      image: minha-app:1.0

      # Opção 1: todas as chaves como variáveis de ambiente
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets

      # Opção 2: chaves específicas
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL

      # Opção 3: montar como arquivo
      volumeMounts:
        - name: config-volume
          mountPath: /app/config
          readOnly: true

  volumes:
    - name: config-volume
      configMap:
        name: app-config
        items:
          - key: application.yml
            path: application.yml
```

> **Secrets não são criptografados por padrão no etcd.** Para produção, habilite encryption at rest ou use soluções como Sealed Secrets, HashiCorp Vault ou External Secrets Operator.

---

## 26. RBAC — Controle de Acesso

### 26.1 Modelo de autorização

RBAC (Role-Based Access Control) controla quem pode fazer o quê no cluster. É composto por quatro recursos:

```
┌──────────────────────────────────────────────────────────────┐
│                    RBAC no Kubernetes                          │
│                                                               │
│  Quem?                O quê?               Onde?              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │ User         │    │ Role         │    │ Namespace    │   │
│  │ Group        │───▶│ (permissões) │───▶│ (escopo)     │   │
│  │ ServiceAccount│    │              │    │              │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│         │                    │                                │
│    RoleBinding         ClusterRole                           │
│    (namespace)     ClusterRoleBinding                        │
│                       (cluster inteiro)                       │
└──────────────────────────────────────────────────────────────┘
```

| Recurso | Escopo | Descrição |
|---------|--------|-----------|
| **Role** | Namespace | Conjunto de permissões dentro de um namespace |
| **ClusterRole** | Cluster | Permissões válidas em todo o cluster |
| **RoleBinding** | Namespace | Liga um Role a um usuário/grupo/ServiceAccount |
| **ClusterRoleBinding** | Cluster | Liga um ClusterRole em escopo global |
| **ServiceAccount** | Namespace | Identidade para Pods acessarem a API |

### 26.2 Role e RoleBinding

```yaml
# Role — permite ler e listar pods no namespace "dev"
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]
---
# RoleBinding — liga o Role ao desenvolvedor
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-pod-reader
  namespace: dev
subjects:
  - kind: User
    name: joao@empresa.com
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 26.3 ClusterRole e ClusterRoleBinding

```yaml
# ClusterRole — acesso de leitura em todos os namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devs-cluster-reader
subjects:
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

### 26.4 ServiceAccount para Pods

```yaml
# ServiceAccount dedicada para a aplicação
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-app-sa
  namespace: production
---
# Role específica para o que o Pod precisa acessar
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: api-app-role
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["db-credentials"]    # acesso apenas a um Secret específico
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-app-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: api-app-sa
    namespace: production
roleRef:
  kind: Role
  name: api-app-role
  apiGroup: rbac.authorization.k8s.io
---
# Deployment usando a ServiceAccount
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  template:
    spec:
      serviceAccountName: api-app-sa
      automountServiceAccountToken: false  # desabilitar se não precisa acessar a API
      containers:
        - name: api-app
          image: minha-app:1.0
```

### 26.5 Verbos e boas práticas

| Verbo | Descrição |
|-------|-----------|
| `get` | Ler um recurso específico |
| `list` | Listar recursos |
| `watch` | Observar mudanças em tempo real |
| `create` | Criar recursos |
| `update` | Atualizar recursos existentes |
| `patch` | Atualizar parcialmente |
| `delete` | Deletar recursos |
| `*` | Todos os verbos (evitar em produção) |

```bash
# Verificar permissões
kubectl auth can-i create deployments --namespace dev
kubectl auth can-i delete pods --namespace production --as joao@empresa.com

# Listar todos os roles/bindings de um namespace
kubectl get roles,rolebindings -n production
kubectl get clusterroles,clusterrolebindings
```

> **Princípio do menor privilégio:** cada ServiceAccount deve ter apenas as permissões necessárias. Nunca use a ServiceAccount `default` com permissões elevadas.

---

## 27. Gerenciamento Avançado de Secrets

### 27.1 Por que Kubernetes Secrets não bastam

Secrets nativos do K8s são codificados em base64 (não criptografados) e armazenados em texto plano no etcd por padrão. Para produção, é necessário usar soluções complementares.

| Problema | Solução |
|----------|---------|
| Secrets em base64 no Git (visíveis para todos) | Sealed Secrets, SOPS |
| Sem criptografia no etcd | Encryption at rest (EncryptionConfiguration) |
| Rotação manual de credenciais | External Secrets Operator, Vault |
| Acesso amplo a Secrets no cluster | RBAC granular por Secret |
| Auditoria de acesso | Vault Audit Log, K8s Audit Policy |

### 27.2 Sealed Secrets (Bitnami)

Sealed Secrets permite criptografar Secrets com uma chave pública e commitá-los no Git com segurança. Apenas o controller no cluster consegue descriptografar.

```bash
# Instalar controller no cluster
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets -n kube-system

# Instalar CLI (kubeseal)
brew install kubeseal     # macOS

# Criar Secret normal e converter para SealedSecret
kubectl create secret generic db-credentials \
  --from-literal=password=minha_senha \
  --dry-run=client -o yaml | \
  kubeseal --format yaml > sealed-secret.yaml

# O arquivo sealed-secret.yaml pode ser commitado no Git com segurança
kubectl apply -f sealed-secret.yaml
```

### 27.3 External Secrets Operator (ESO)

ESO sincroniza secrets de provedores externos (AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, HashiCorp Vault) para Kubernetes Secrets automaticamente.

```yaml
# SecretStore — conexão com o provedor
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
---
# ExternalSecret — sincroniza um secret específico
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h                    # re-sincroniza a cada 1h
  secretStoreRef:
    name: aws-secrets
    kind: SecretStore
  target:
    name: db-credentials                 # nome do K8s Secret gerado
  data:
    - secretKey: password                # chave no K8s Secret
      remoteRef:
        key: production/database         # chave no AWS Secrets Manager
        property: password               # campo dentro do secret
```

### 27.4 HashiCorp Vault

```bash
# Instalar Vault no K8s via Helm
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --set server.dev.enabled=true

# Habilitar Kubernetes auth
vault auth enable kubernetes
vault write auth/kubernetes/config \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

# Criar policy e role
vault policy write app-policy - <<EOF
path "secret/data/production/*" {
  capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/api-app \
  bound_service_account_names=api-app-sa \
  bound_service_account_namespaces=production \
  policies=app-policy \
  ttl=1h
```

### 27.5 SOPS (Mozilla)

SOPS criptografa valores dentro de arquivos YAML/JSON, mantendo as chaves visíveis. Integra com KMS (AWS, GCP, Azure) e age.

```bash
# Criptografar arquivo
sops --encrypt --age age1... secrets.yaml > secrets.enc.yaml

# Editar arquivo criptografado
sops secrets.enc.yaml

# Usar com Kustomize (kustomize-sops plugin) ou Flux (built-in)
```

### 27.6 Comparação

| Aspecto | Sealed Secrets | External Secrets | Vault | SOPS |
|---------|---------------|-----------------|-------|------|
| **Complexidade** | Baixa | Média | Alta | Baixa |
| **Secrets no Git** | Sim (criptografados) | Não (referências) | Não (referências) | Sim (criptografados) |
| **Rotação automática** | Não | Sim | Sim | Não |
| **Multi-cloud** | Não (apenas K8s) | Sim | Sim | Sim |
| **Auditoria** | Não | Via provedor | Sim (audit log) | Via Git |
| **Melhor para** | Projetos simples, GitOps | Multi-cloud, secrets centralizados | Segurança enterprise | Kustomize/Flux com Git |

---

## 28. Storage no Kubernetes

### 28.1 Modelo de storage

```
┌──────────────────────────────────────────────────────────┐
│              Storage no Kubernetes                        │
│                                                           │
│  Pod                                                      │
│  ┌─────────────────────┐                                 │
│  │ Container            │                                 │
│  │  volumeMounts:       │                                 │
│  │    /data ──────────┐ │                                 │
│  └────────────────────┼─┘                                 │
│                       │                                    │
│  PersistentVolumeClaim (PVC)                              │
│  ┌────────────────────┼────────────┐                     │
│  │ "Quero 10Gi, RWO"  │            │                     │
│  └────────────────────┼────────────┘                     │
│                       │  bind                              │
│  PersistentVolume (PV)│                                   │
│  ┌────────────────────┼────────────┐                     │
│  │ 10Gi, AWS EBS vol-abc123       │                      │
│  └─────────────────────────────────┘                     │
│                                                           │
│  StorageClass                                             │
│  ┌─────────────────────────────────┐                     │
│  │ Provisioner: aws-ebs            │                     │
│  │ Cria PVs automaticamente       │                      │
│  └─────────────────────────────────┘                     │
└──────────────────────────────────────────────────────────┘
```

### 28.2 PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce          # RWO: leitura/escrita por um node
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi
```

| Access Mode | Sigla | Descrição |
|-------------|-------|-----------|
| ReadWriteOnce | RWO | Montado para leitura/escrita por um único node |
| ReadOnlyMany | ROX | Montado para leitura por múltiplos nodes |
| ReadWriteMany | RWX | Montado para leitura/escrita por múltiplos nodes |

### 28.3 Usando PVC no Deployment

```yaml
spec:
  containers:
    - name: postgres
      image: postgres:16-alpine
      volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumes:
    - name: pgdata
      persistentVolumeClaim:
        claimName: postgres-pvc
```

---

## 29. Escalabilidade e Autoscaling

### 29.1 Scaling manual

```bash
# Escalar para 5 réplicas
kubectl scale deployment api-app --replicas=5

# Verificar
kubectl get pods -l app=api-app
```

### 29.2 Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120
```

```bash
# Criar HPA rápido via CLI
kubectl autoscale deployment api-app --min=2 --max=10 --cpu-percent=70

# Monitorar HPA
kubectl get hpa
kubectl describe hpa api-app-hpa
```

> **Pré-requisito:** Metrics Server deve estar instalado no cluster para que HPA funcione.

### 29.3 Vertical Pod Autoscaler (VPA)

VPA ajusta automaticamente os `requests` e `limits` de CPU/memória dos Pods, sendo útil para workloads difíceis de dimensionar manualmente.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-app
  updatePolicy:
    updateMode: Auto   # Off, Initial ou Auto
  resourcePolicy:
    containerPolicies:
      - containerName: api-app
        minAllowed:
          cpu: "100m"
          memory: "128Mi"
        maxAllowed:
          cpu: "2"
          memory: "2Gi"
```

> **VPA e HPA não devem gerenciar a mesma métrica.** Use HPA para CPU e VPA para memória, ou desabilite CPU no VPA se usar HPA baseado em CPU.

---

## 30. Estratégias de Deploy no Kubernetes

### 30.1 Visão geral

```
┌──────────────────────────────────────────────────────────────┐
│               Estratégias de Deploy                           │
│                                                               │
│  Rolling Update         Blue/Green            Canary          │
│  ┌───┐ ┌───┐           ┌───────┐             ┌───┐ ┌───┐   │
│  │v1 │→│v2 │           │  v1   │ (ativo)     │v1 │ │v1 │   │
│  │v1 │→│v2 │           │       │             │v1 │ │v2 │   │
│  │v1 │ │v1 │           ├───────┤             └───┘ └───┘   │
│  └───┘ └───┘           │  v2   │ (standby)    90%   10%    │
│  gradual                │       │             do tráfego     │
│  (1 por vez)           └───────┘                             │
│                         switch instantâneo                    │
└──────────────────────────────────────────────────────────────┘
```

### 30.2 Rolling Update (default do Kubernetes)

Substitui Pods gradualmente — sem downtime, mas versões v1 e v2 coexistem temporariamente.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1           # máximo de Pods extras durante update
      maxUnavailable: 0     # nunca ter menos que 4 Pods disponíveis
  template:
    spec:
      containers:
        - name: api-app
          image: minha-app:2.0    # atualizar tag para deploy
```

```bash
# Atualizar imagem (trigger rolling update)
kubectl set image deployment/api-app api-app=minha-app:2.0

# Acompanhar progresso
kubectl rollout status deployment/api-app

# Histórico de rollouts
kubectl rollout history deployment/api-app

# Rollback para versão anterior
kubectl rollout undo deployment/api-app

# Rollback para revisão específica
kubectl rollout undo deployment/api-app --to-revision=3

# Pausar/retomar rollout (para validação manual)
kubectl rollout pause deployment/api-app
kubectl rollout resume deployment/api-app
```

### 30.3 Blue/Green

Mantém dois ambientes idênticos (blue e green). O Service alterna o tráfego entre eles instantaneamente.

```yaml
# Deployment Blue (versão atual)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-app
      version: blue
  template:
    metadata:
      labels:
        app: api-app
        version: blue
    spec:
      containers:
        - name: api-app
          image: minha-app:1.0
---
# Deployment Green (versão nova)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-app
      version: green
  template:
    metadata:
      labels:
        app: api-app
        version: green
    spec:
      containers:
        - name: api-app
          image: minha-app:2.0
---
# Service — aponta para blue (trocar selector para green no switch)
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api-app
    version: blue      # ← alterar para "green" para fazer o switch
  ports:
    - port: 80
      targetPort: 8080
```

```bash
# Switch: alterar o Service para apontar para green
kubectl patch service api-service \
  -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback: voltar para blue
kubectl patch service api-service \
  -p '{"spec":{"selector":{"version":"blue"}}}'
```

### 30.4 Canary

Direciona uma fração do tráfego para a nova versão. Permite validar em produção antes de promover.

```yaml
# Deployment stable (90% do tráfego)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: api-app
      track: stable
  template:
    metadata:
      labels:
        app: api-app
        track: stable
    spec:
      containers:
        - name: api-app
          image: minha-app:1.0
---
# Deployment canary (10% do tráfego)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app-canary
spec:
  replicas: 1           # 1 de 10 Pods = ~10% do tráfego
  selector:
    matchLabels:
      app: api-app
      track: canary
  template:
    metadata:
      labels:
        app: api-app
        track: canary
    spec:
      containers:
        - name: api-app
          image: minha-app:2.0
---
# Service — seleciona ambos os tracks pelo label "app"
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api-app        # seleciona stable + canary
  ports:
    - port: 80
      targetPort: 8080
```

> **Canary avançado:** para controle fino do tráfego (ex: 5% exato, headers específicos), use Istio, Linkerd ou Argo Rollouts em vez de proporção de réplicas.

### 30.5 Comparação de estratégias

| Aspecto | Rolling Update | Blue/Green | Canary |
|---------|---------------|------------|--------|
| **Downtime** | Zero | Zero | Zero |
| **Coexistência de versões** | Sim (temporária) | Não (switch instantâneo) | Sim (controlada) |
| **Rollback** | `kubectl rollout undo` | Trocar selector do Service | Remover canary Deployment |
| **Uso de recursos** | Normal | 2× (dois ambientes completos) | Normal + poucos Pods extras |
| **Complexidade** | Baixa (default do K8s) | Média | Média a Alta |
| **Validação em produção** | Não | Não (tudo ou nada) | Sim (fração do tráfego) |
| **Melhor para** | A maioria dos deploys | Mudanças críticas, precisam de switch instantâneo | Mudanças arriscadas, validação gradual |

---

## 31. Helm — Gerenciador de Pacotes

### 31.1 O que é Helm

Helm é o gerenciador de pacotes do Kubernetes. Um **chart** é um pacote com templates de manifestos K8s parametrizáveis.

```
┌────────────────────────────────────────────────┐
│                 Helm Chart                      │
│                                                 │
│  meu-chart/                                     │
│  ├── Chart.yaml          # metadados            │
│  ├── values.yaml         # valores default      │
│  ├── templates/                                 │
│  │   ├── deployment.yaml # template com {{ }}   │
│  │   ├── service.yaml                           │
│  │   ├── ingress.yaml                           │
│  │   ├── configmap.yaml                         │
│  │   ├── hpa.yaml                               │
│  │   └── _helpers.tpl    # funções auxiliares   │
│  └── charts/             # dependências         │
└────────────────────────────────────────────────┘
```

### 31.2 Comandos essenciais

```bash
# Adicionar repositório de charts
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Buscar charts
helm search repo postgresql

# Instalar chart
helm install meu-postgres bitnami/postgresql \
  --namespace banco \
  --create-namespace \
  --set auth.postgresPassword=minha_senha \
  --set primary.persistence.size=10Gi

# Instalar com arquivo de valores
helm install meu-app ./meu-chart -f values-prod.yaml

# Listar releases
helm list
helm list --all-namespaces

# Atualizar release
helm upgrade meu-app ./meu-chart -f values-prod.yaml

# Install ou upgrade (idempotente — ideal para CI/CD)
helm upgrade --install meu-app ./meu-chart -f values-prod.yaml

# Rollback
helm rollback meu-app 1   # volta para revisão 1
helm history meu-app       # ver histórico de revisões

# Desinstalar
helm uninstall meu-app

# Gerar manifestos sem aplicar (debug)
helm template meu-app ./meu-chart -f values-prod.yaml > output.yaml

# Validar chart
helm lint ./meu-chart
```

### 31.3 Exemplo de chart para Spring Boot

```yaml
# Chart.yaml
apiVersion: v2
name: spring-boot-app
description: Chart genérico para aplicações Spring Boot
version: 1.0.0
appVersion: "1.0.0"
```

```yaml
# values.yaml
replicaCount: 2
image:
  repository: registry.example.com/minha-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  host: api.example.com
  tls: true

resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 8
  targetCPUUtilization: 70

spring:
  profiles: prod
  datasource:
    url: jdbc:postgresql://postgres:5432/appdb
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "spring-boot-app.fullname" . }}
  labels:
    {{- include "spring-boot-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "spring-boot-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "spring-boot-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: {{ .Values.spring.profiles }}
            - name: SPRING_DATASOURCE_URL
              value: {{ .Values.spring.datasource.url }}
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: 30
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

---

## 32. Kubernetes Operators e CRDs

### 32.1 O que é um Operator

Um Operator é um controlador customizado que estende o Kubernetes para gerenciar aplicações complexas de forma declarativa. Ele encapsula conhecimento operacional (instalação, configuração, backup, scaling) em código.

```
┌──────────────────────────────────────────────────────────────┐
│                    Operator Pattern                           │
│                                                               │
│  ┌─────────────────┐         ┌────────────────────┐         │
│  │ Custom Resource │         │    Operator        │          │
│  │ (CRD)           │────────▶│    (Controller)    │          │
│  │                 │ watches │                    │          │
│  │ kind: Postgres  │         │ Cria/gerencia:     │          │
│  │ spec:           │         │ • StatefulSet      │          │
│  │   version: 16   │         │ • Service          │          │
│  │   replicas: 3   │         │ • PVC              │          │
│  │   backup: daily │         │ • ConfigMap        │          │
│  └─────────────────┘         │ • CronJob (backup) │          │
│                               └────────────────────┘         │
│                                                               │
│  O usuário declara O QUE quer. O Operator sabe COMO fazer.   │
└──────────────────────────────────────────────────────────────┘
```

### 32.2 Custom Resource Definition (CRD)

```yaml
# Definir um novo tipo de recurso no Kubernetes
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum: [postgres, mysql]
                version:
                  type: string
                replicas:
                  type: integer
                storage:
                  type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
      - db
```

```yaml
# Usar o recurso customizado
apiVersion: example.com/v1
kind: Database
metadata:
  name: meu-banco
spec:
  engine: postgres
  version: "16"
  replicas: 3
  storage: 50Gi
```

### 32.3 Operators populares

| Operator | Gerencia | Instalação |
|----------|----------|------------|
| **CloudNativePG** | PostgreSQL (HA, backup, failover) | `kubectl apply -f` manifest |
| **Strimzi** | Apache Kafka (brokers, topics, users) | Helm / OLM |
| **Prometheus Operator** | Prometheus, Alertmanager, Grafana | Helm (`kube-prometheus-stack`) |
| **cert-manager** | Certificados TLS (Let's Encrypt) | Helm |
| **Redis Operator (Spotahome)** | Redis Cluster e Sentinel | Helm |
| **Elastic Cloud on K8s (ECK)** | Elasticsearch, Kibana, APM | `kubectl apply -f` |
| **MongoDB Community Operator** | MongoDB ReplicaSet | Helm / OLM |
| **ArgoCD** | GitOps e deploy contínuo | `kubectl apply -f` |

```bash
# Exemplo: instalar CloudNativePG para PostgreSQL gerenciado
kubectl apply --server-side \
  -f https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.24/releases/cnpg-1.24.1.yaml

# Criar cluster PostgreSQL via CRD
kubectl apply -f - <<EOF
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: meu-postgres
spec:
  instances: 3
  storage:
    size: 10Gi
  postgresql:
    parameters:
      max_connections: "200"
  backup:
    barmanObjectStore:
      destinationPath: s3://backups/postgres
EOF

# Verificar
kubectl get clusters
kubectl get pods -l cnpg.io/cluster=meu-postgres
```

### 32.4 Quando usar Operators vs manifestos tradicionais

| Cenário | Abordagem |
|---------|-----------|
| Stateless apps (APIs, workers) | Deployment + Service (manifestos simples) |
| Banco de dados com HA e backup | Operator (CloudNativePG, Strimzi) |
| Certificados TLS automatizados | Operator (cert-manager) |
| Stack de monitoramento completa | Operator (kube-prometheus-stack) |
| Aplicação interna simples | Manifestos + Helm/Kustomize |

> **Regra prática:** use um Operator existente para infraestrutura stateful complexa. Para aplicações próprias, Deployment + Service + Helm/Kustomize geralmente é suficiente. Criar um Operator próprio só vale se você distribui software para clientes rodarem em seus clusters.

---

## 33. Deploy de Spring Boot no Kubernetes

### 33.1 Configuração do Spring Boot para K8s

```yaml
# application-prod.yml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s

server:
  shutdown: graceful

management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true
      group:
        readiness:
          include: db,redis
          additional-path: server:/readyz
        liveness:
          include: ping
          additional-path: server:/livez
  metrics:
    tags:
      application: ${spring.application.name}
```

### 33.2 Dependências Maven para health probes e métricas

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 33.3 Graceful shutdown

O Kubernetes envia SIGTERM ao Pod antes de remover. O Spring precisa estar configurado para tratar conexões existentes antes de encerrar.

```
┌──────────────────────────────────────────────────────────────┐
│              Lifecycle de Shutdown no K8s                      │
│                                                               │
│  1. Pod marcado para termination                              │
│  2. Pod removido dos endpoints do Service (sem tráfego novo)  │
│  3. preStop hook executa (se configurado)                     │
│  4. SIGTERM enviado ao container                              │
│  5. Spring inicia graceful shutdown                           │
│     - Para de aceitar novas requisições                       │
│     - Aguarda requisições em andamento (timeout-per-shutdown) │
│  6. Após terminationGracePeriodSeconds → SIGKILL              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# No Deployment — garantir tempo para graceful shutdown
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 45
      containers:
        - name: app
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 5"]
                # aguarda remoção do endpoint antes de iniciar shutdown
```

### 33.4 Manifestos completos para deploy

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: minha-app
---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: minha-app
data:
  SPRING_PROFILES_ACTIVE: "prod"
  SERVER_PORT: "8080"
  SPRING_DATASOURCE_URL: "jdbc:postgresql://postgres:5432/appdb"
---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: minha-app
type: Opaque
data:
  SPRING_DATASOURCE_PASSWORD: c2VjcmV0   # base64
---
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
  namespace: minha-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: api-app
    spec:
      terminationGracePeriodSeconds: 45
      containers:
        - name: api-app
          image: registry.example.com/api-app:1.0.0
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: app-config
            - secretRef:
                name: app-secrets
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            failureThreshold: 30
            periodSeconds: 2
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 5"]
---
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: minha-app
spec:
  selector:
    app: api-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: minha-app
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
---
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-app-hpa
  namespace: minha-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

```bash
# Deploy completo
kubectl apply -f k8s/
```

---

## 34. GitOps com ArgoCD e Flux

### 34.1 O que é GitOps

GitOps é um paradigma operacional onde o **Git é a única fonte de verdade** para o estado desejado da infraestrutura e das aplicações. Um agente no cluster monitora o repositório e reconcilia automaticamente o estado do cluster com o que está no Git.

```
┌──────────────────────────────────────────────────────────────┐
│                     Fluxo GitOps                              │
│                                                               │
│  Desenvolvedor                                                │
│      │ git push                                               │
│      ▼                                                        │
│  ┌──────────────┐     ┌───────────────────┐                  │
│  │  Repositório │────▶│  Agente GitOps    │                  │
│  │  Git (GitHub)│     │  (ArgoCD / Flux)  │                  │
│  │              │     │                   │                  │
│  │  k8s/        │     │  Compara Git vs   │                  │
│  │  ├── base/   │     │  estado do cluster│                  │
│  │  └── prod/   │     │  e reconcilia     │                  │
│  └──────────────┘     └───────┬───────────┘                  │
│                               │ kubectl apply                 │
│                               ▼                               │
│                       ┌───────────────┐                       │
│                       │   Cluster K8s │                       │
│                       └───────────────┘                       │
│                                                               │
│  Push model (CI/CD tradicional): CI faz deploy no cluster     │
│  Pull model (GitOps): cluster puxa estado do Git              │
└──────────────────────────────────────────────────────────────┘
```

| Princípio | Descrição |
|-----------|-----------|
| **Declarativo** | Todo o estado desejado é descrito em manifestos no Git |
| **Versionado** | Git fornece histórico, auditoria e rollback nativos |
| **Automatizado** | Agente aplica mudanças automaticamente (pull-based) |
| **Auto-healing** | Se alguém alterar o cluster manualmente, o agente reverte |

### 34.2 ArgoCD

ArgoCD é a ferramenta GitOps mais popular, com interface web, CLI e API para gerenciar deploys.

```bash
# Instalar ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Acessar UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login via CLI
argocd login localhost:8080
# Senha inicial: kubectl -n argocd get secret argocd-initial-admin-secret \
#   -o jsonpath="{.data.password}" | base64 -d
```

```yaml
# argocd-app.yaml — Application que sincroniza do Git
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minha-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/minha-app.git
    targetRevision: main
    path: k8s/overlays/prod          # diretório Kustomize
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true                     # remove recursos que não estão no Git
      selfHeal: true                  # reverte mudanças manuais no cluster
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 3
      backoff:
        duration: 5s
        maxDuration: 3m
```

```bash
# Comandos úteis
argocd app list
argocd app get minha-app
argocd app sync minha-app           # forçar sincronização
argocd app history minha-app        # histórico de deploys
argocd app rollback minha-app 2     # rollback para revisão 2
```

### 34.3 Flux

```bash
# Instalar Flux
flux bootstrap github \
  --owner=minha-org \
  --repository=infra-k8s \
  --branch=main \
  --path=clusters/production \
  --personal

# Criar source (repositório Git)
flux create source git minha-app \
  --url=https://github.com/org/minha-app \
  --branch=main \
  --interval=1m

# Criar Kustomization (reconciliação)
flux create kustomization minha-app \
  --source=minha-app \
  --path=k8s/overlays/prod \
  --prune=true \
  --interval=5m

# Monitorar
flux get kustomizations
flux get sources git
flux logs
```

### 34.4 ArgoCD vs Flux

| Aspecto | ArgoCD | Flux |
|---------|--------|------|
| **Interface** | Web UI rica + CLI + API | CLI + API (sem UI nativa) |
| **Multi-cluster** | Nativo (gerencia vários clusters) | Requer Flux em cada cluster |
| **RBAC** | Integrado (projetos, roles) | Delega ao RBAC do K8s |
| **Suporte a Helm** | Sim (renderiza e aplica) | Sim (HelmRelease CRD) |
| **Suporte a Kustomize** | Sim | Sim (nativo) |
| **Notificações** | Integrado (Slack, email, webhook) | Via Notification Controller |
| **Complexidade** | Média | Baixa |
| **Recomendado para** | Equipes que precisam de UI e multi-cluster | Equipes que preferem CLI e simplicidade |

---

## 35. Ambientes Locais de Kubernetes

### 35.1 Opções para desenvolvimento local

| Ferramenta | Descrição | Melhor para |
|------------|-----------|-------------|
| **Docker Desktop** | K8s embutido (1 click) | Início rápido, Windows/Mac |
| **Minikube** | VM ou container com cluster single-node | Aprendizado, testes |
| **Kind** | Kubernetes in Docker — clusters multi-node | CI/CD, testes automatizados |
| **K3s** | Kubernetes leve (Rancher) | Edge, IoT, ambientes com poucos recursos |
| **K3d** | K3s dentro de Docker | Desenvolvimento rápido |

### 35.2 Minikube

```bash
# Instalar e iniciar
minikube start --driver=docker --memory=4096 --cpus=2

# Dashboard
minikube dashboard

# Addon de métricas (necessário para HPA)
minikube addons enable metrics-server

# Addon Ingress
minikube addons enable ingress

# Acessar serviço NodePort
minikube service meu-service

# Usar Docker do Minikube (evita push para registry)
eval $(minikube docker-env)
docker build -t minha-app:1.0 .

# Parar e deletar
minikube stop
minikube delete
```

### 35.3 Kind (Kubernetes in Docker)

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
      - containerPort: 443
        hostPort: 443
  - role: worker
  - role: worker
```

```bash
# Criar cluster
kind create cluster --config kind-config.yaml --name meu-cluster

# Carregar imagem local (sem precisar de registry)
kind load docker-image minha-app:1.0 --name meu-cluster

# Deletar cluster
kind delete cluster --name meu-cluster
```

---

## 36. Kubernetes em Cloud (Managed)

### 36.1 Principais provedores

| Provedor | Serviço | Control Plane | Preço Control Plane |
|----------|---------|---------------|-------------------|
| **AWS** | EKS (Elastic Kubernetes Service) | Gerenciado | ~$0.10/hora (~$73/mês) |
| **Google Cloud** | GKE (Google Kubernetes Engine) | Gerenciado | Gratuito (Autopilot: paga por Pod) |
| **Azure** | AKS (Azure Kubernetes Service) | Gerenciado | Gratuito (paga apenas pelos nodes) |
| **DigitalOcean** | DOKS | Gerenciado | Gratuito (paga apenas pelos nodes) |
| **Oracle Cloud** | OKE | Gerenciado | Gratuito (paga apenas pelos nodes) |
| **Linode/Akamai** | LKE | Gerenciado | Gratuito (paga apenas pelos nodes) |

### 36.2 Comparação de funcionalidades

| Aspecto | EKS (AWS) | GKE (Google) | AKS (Azure) |
|---------|-----------|-------------|-------------|
| **Facilidade de setup** | Média (eksctl simplifica) | Alta (mais automatizado) | Alta |
| **Autopilot/Serverless** | Fargate (por Pod) | GKE Autopilot | AKS Virtual Nodes |
| **Networking** | VPC-native, Calico | VPC-native, Cilium | Azure CNI, Calico |
| **Ingress nativo** | ALB Ingress Controller | GKE Ingress (GCLB) | Application Gateway |
| **Service Mesh** | App Mesh | Anthos Service Mesh | Open Service Mesh |
| **Observabilidade** | CloudWatch, X-Ray | Cloud Operations | Azure Monitor |
| **Registry integrado** | ECR | Artifact Registry | ACR |
| **IAM integrado** | IAM Roles for Pods | Workload Identity | Managed Identity |
| **GPU/ML** | GPU instances, SageMaker | GPU nodes, Vertex AI | GPU nodes, Azure ML |
| **Multi-cluster** | EKS Anywhere | Anthos (multi-cloud) | Azure Arc |
| **Certificação CIS** | Sim | Sim | Sim |

### 36.3 Setup rápido por provedor

```bash
# === AWS EKS ===
# Instalar eksctl
# https://eksctl.io

eksctl create cluster \
  --name meu-cluster \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5 \
  --managed

# Configurar kubectl
aws eks update-kubeconfig --name meu-cluster --region us-east-1

# === Google GKE ===
gcloud container clusters create meu-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type e2-medium \
  --enable-autoscaling --min-nodes 2 --max-nodes 5

gcloud container clusters get-credentials meu-cluster --zone us-central1-a

# === Azure AKS ===
az aks create \
  --resource-group meu-rg \
  --name meu-cluster \
  --node-count 3 \
  --node-vm-size Standard_B2s \
  --enable-cluster-autoscaler --min-count 2 --max-count 5

az aks get-credentials --resource-group meu-rg --name meu-cluster
```

### 36.4 Quando usar managed vs self-hosted

| Cenário | Recomendação |
|---------|-------------|
| Equipe pequena, foco no produto | **Managed** — não gaste tempo operando o cluster |
| Requisitos de compliance/regulação restritivos | **Self-hosted** ou managed com configuração avançada |
| Multi-cloud obrigatório | **Self-hosted** (K3s, Kubespray) ou Anthos/Arc |
| Ambiente de aprendizado | **Minikube/Kind** localmente → managed para staging/prod |
| Custos são prioridade | **DigitalOcean/Linode** (control plane gratuito, nodes baratos) |
| Workloads com GPU/ML | **GKE ou EKS** (melhor suporte a GPUs) |

---

## 37. Observabilidade no Kubernetes

### 37.1 Stack de observabilidade

```
┌──────────────────────────────────────────────────────────────┐
│                 Observabilidade no K8s                        │
│                                                               │
│  ┌─────────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │    Métricas      │  │    Logs      │  │   Traces     │    │
│  │                  │  │              │  │              │    │
│  │  Prometheus      │  │  Loki        │  │  Tempo       │    │
│  │  + Grafana       │  │  Fluentd     │  │  Jaeger      │    │
│  │                  │  │  Fluent Bit  │  │  Zipkin      │    │
│  │  Metrics Server  │  │              │  │              │    │
│  └─────────────────┘  └──────────────┘  └──────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Grafana — Dashboard unificado                       │     │
│  │  (métricas + logs + traces em um só lugar)           │     │
│  └─────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────┘
```

### 37.2 Prometheus ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-app-monitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: api-app
  endpoints:
    - port: http
      path: /actuator/prometheus
      interval: 15s
```

### 37.3 Coletando logs com Fluent Bit (DaemonSet)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:latest
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: containers
              mountPath: /var/lib/docker/containers
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: containers
          hostPath:
            path: /var/lib/docker/containers
```

---

## 38. Backup e Disaster Recovery

### 38.1 Estratégias de proteção

| Estratégia | O que protege | Ferramenta |
|------------|--------------|------------|
| **Backup do etcd** | Estado do cluster (todos os recursos K8s) | `etcdctl snapshot` |
| **Backup de recursos** | Manifestos, PVs, namespaces selecionados | Velero |
| **Snapshots de PV** | Dados persistentes (bancos, arquivos) | CSI Snapshots, cloud provider |
| **GitOps como backup** | Manifestos da aplicação | Git (ArgoCD/Flux) |
| **Backup do banco** | Dados da aplicação | pg_dump, mysqldump, mongodump |

### 38.2 Velero — Backup e restore de recursos K8s

Velero é a ferramenta padrão para backup, restore e migração de recursos Kubernetes e volumes persistentes.

```bash
# Instalar Velero (com plugin AWS como exemplo)
velero install \
  --provider aws \
  --bucket meu-bucket-backups \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1

# === Backups ===

# Backup completo do cluster
velero backup create backup-completo

# Backup de namespace específico
velero backup create backup-prod --include-namespaces production

# Backup com seletor de labels
velero backup create backup-api --selector app=api-app

# Backup excluindo recursos
velero backup create backup-parcial \
  --exclude-resources events,events.events.k8s.io

# Backup agendado (diário, retenção de 7 dias)
velero schedule create daily-backup \
  --schedule="0 2 * * *" \
  --ttl 168h \
  --include-namespaces production

# === Restore ===

# Restore completo
velero restore create --from-backup backup-completo

# Restore apenas de namespace
velero restore create --from-backup backup-completo \
  --include-namespaces production

# Restore para namespace diferente (migração)
velero restore create --from-backup backup-completo \
  --namespace-mappings staging:production

# === Gerenciamento ===

# Listar backups
velero backup get

# Listar schedules
velero schedule get

# Descrever backup (status, erros)
velero backup describe backup-completo --details

# Deletar backup
velero backup delete backup-completo
```

### 38.3 Backup do etcd

```bash
# Snapshot do etcd (executar no control plane)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verificar snapshot
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-table

# Restaurar snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restored
```

### 38.4 VolumeSnapshot (CSI)

```yaml
# Criar snapshot de um PVC
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: postgres-snapshot
spec:
  volumeSnapshotClassName: csi-snapclass
  source:
    persistentVolumeClaimName: postgres-pvc
---
# Restaurar PVC a partir do snapshot
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc-restored
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi
  dataSource:
    name: postgres-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

### 38.5 Checklist de Disaster Recovery

| Item | Frequência | Verificação |
|------|-----------|-------------|
| Backup do etcd | Diário | Snapshot + teste de restore trimestral |
| Backup Velero | Diário | Verificar logs e status dos backups |
| Snapshots de PV | Diário (bancos críticos) | Testar restore em ambiente de staging |
| Manifestos em Git | Contínuo (GitOps) | ArgoCD/Flux sync status |
| Runbook de DR | Trimestral | Simulação completa de disaster recovery |
| RPO/RTO documentados | Anual | Validar com stakeholders |

> **RPO** (Recovery Point Objective) = quanto tempo de dados posso perder. **RTO** (Recovery Time Objective) = quanto tempo para restaurar o serviço.

---

## 39. Boas Práticas de Kubernetes

### 39.1 Organização e segurança

| Prática | Descrição |
|---------|-----------|
| **Namespaces** | Separar por ambiente (`dev`, `staging`, `prod`) ou por equipe |
| **Labels e Annotations** | Padronizar: `app`, `version`, `team`, `environment` |
| **RBAC** | Princípio do menor privilégio — ServiceAccounts com roles mínimas |
| **Network Policies** | Restringir comunicação entre Pods (deny-all por default) |
| **Pod Security Standards** | Aplicar `restricted` para Pods em produção |
| **Resource Quotas** | Limitar recursos por namespace para evitar abuso |
| **Limit Ranges** | Definir defaults de requests/limits por namespace |

### 39.2 Resiliência

| Prática | Descrição |
|---------|-----------|
| **Sempre definir requests e limits** | Scheduler precisa saber quanto alocar; sem limits = risco de OOMKill |
| **PodDisruptionBudget** | Garante mínimo de Pods disponíveis durante rolling updates |
| **Anti-affinity** | Distribui réplicas entre nodes diferentes |
| **Probes configuradas** | startup + readiness + liveness adequados ao perfil da app |
| **Graceful shutdown** | preStop hook + terminationGracePeriodSeconds |
| **Backups do etcd** | Plano de disaster recovery para o cluster |

### 39.3 PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-app-pdb
spec:
  minAvailable: 2   # sempre manter pelo menos 2 Pods rodando
  selector:
    matchLabels:
      app: api-app
```

### 39.4 Anti-affinity — Distribuir Pods entre nodes

```yaml
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - api-app
                topologyKey: kubernetes.io/hostname
```

### 39.5 Network Policy — Restringir tráfego

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-app-netpol
spec:
  podSelector:
    matchLabels:
      app: api-app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres
      ports:
        - port: 5432
    - to:   # permitir DNS
        - namespaceSelector: {}
      ports:
        - port: 53
          protocol: UDP
```

### 39.6 Resource Quota por namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
    services: "10"
    persistentvolumeclaims: "5"
```

---

## 40. Troubleshooting

### 40.1 Container não inicia

```bash
# 1. Ver status do Pod
kubectl get pods
# STATUS: CrashLoopBackOff, ImagePullBackOff, Pending, etc.

# 2. Descrever o Pod (eventos detalhados)
kubectl describe pod meu-pod

# 3. Ver logs do container (incluindo crash anterior)
kubectl logs meu-pod --previous

# 4. Verificar se a imagem existe
docker pull minha-imagem:1.0
```

### 40.2 Diagnóstico por status

| Status | Causa provável | Ação |
|--------|---------------|------|
| **Pending** | Sem resources disponíveis ou PVC não bound | `kubectl describe pod` → ver eventos |
| **ImagePullBackOff** | Imagem não encontrada ou sem credenciais | Verificar nome/tag; criar `imagePullSecret` |
| **CrashLoopBackOff** | App crashando ao iniciar | `kubectl logs --previous`; verificar variáveis/config |
| **OOMKilled** | Container excedeu limite de memória | Aumentar `limits.memory` ou otimizar app |
| **Evicted** | Node sem recursos (disk pressure, memory pressure) | Verificar `kubectl describe node` |
| **Running mas sem tráfego** | readinessProbe falhando | `kubectl describe pod` → verificar probe |

### 40.3 Ferramentas de debug

```bash
# Pod temporário para debug de rede
kubectl run debug --image=nicolaka/netshoot --rm -it -- bash
# dentro do pod:
nslookup meu-service
curl http://meu-service:80/health

# Debug de DNS
kubectl run dns-test --image=busybox --rm -it -- nslookup kubernetes.default

# Ver todos os recursos de um namespace
kubectl api-resources --verbs=list --namespaced -o name | \
  xargs -n 1 kubectl get --show-kind -n meu-namespace

# Dry-run para validar manifesto sem aplicar
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server

# Diff entre manifesto e estado atual
kubectl diff -f deployment.yaml
```

---

## 41. Exemplos Completos — Do Dockerfile ao Deploy em K8s

### 41.1 Fluxo completo de uma aplicação Spring Boot

```
┌──────────────────────────────────────────────────────────────────┐
│           Pipeline: Código → Produção em K8s                      │
│                                                                   │
│  1. Desenvolvedor faz push                                        │
│        │                                                          │
│  2. CI/CD (GitHub Actions)                                        │
│        ├── Testes unitários + integração                          │
│        ├── Build da imagem Docker (multi-stage)                   │
│        ├── Scan de vulnerabilidades (Trivy)                       │
│        ├── Push para Container Registry                           │
│        └── Deploy no K8s (kubectl apply ou helm upgrade)          │
│                    │                                              │
│  3. Kubernetes                                                    │
│        ├── Rolling update (zero downtime)                         │
│        ├── Health checks (startup → readiness → liveness)         │
│        ├── HPA escala baseado em carga                            │
│        └── Ingress roteia tráfego externo                         │
│                    │                                              │
│  4. Observabilidade                                               │
│        ├── Prometheus coleta métricas                             │
│        ├── Loki agrega logs                                       │
│        └── Grafana visualiza tudo                                 │
└──────────────────────────────────────────────────────────────────┘
```

### 41.2 Estrutura de projeto recomendada

```
meu-projeto/
├── src/                          # código-fonte
├── pom.xml
├── Dockerfile                    # build da imagem
├── .dockerignore
├── compose.yml                   # ambiente local
├── compose.override.yml          # override para dev
├── k8s/                          # manifestos Kubernetes
│   ├── base/                     # recursos base (Kustomize)
│   │   ├── kustomization.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── overlays/
│   │   ├── dev/
│   │   │   ├── kustomization.yaml
│   │   │   └── patch-replicas.yaml
│   │   └── prod/
│   │       ├── kustomization.yaml
│   │       ├── patch-replicas.yaml
│   │       └── patch-resources.yaml
│   └── helm/                     # alternativa com Helm
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-prod.yaml
│       └── templates/
├── .github/
│   └── workflows/
│       └── deploy.yml            # CI/CD
└── README.md
```

### 41.3 GitHub Actions — Build e Deploy

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Run tests
        run: ./mvnw verify -B

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v5
        with:
          namespace: production
          manifests: k8s/
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

### 41.4 Kustomize — Customização sem templates

#### O que é Kustomize

Kustomize é uma ferramenta nativa do `kubectl` que permite customizar manifestos Kubernetes **sem usar templates**. Em vez de substituir variáveis em arquivos `.tpl` (como Helm faz), Kustomize trabalha com **patches e overlays** sobre YAML válido — os manifestos base são arquivos Kubernetes reais que podem ser aplicados diretamente.

```
┌─────────────────────────────────────────────────────────────┐
│                  Modelo do Kustomize                         │
│                                                              │
│   base/                    overlays/dev/    overlays/prod/   │
│  ┌──────────────┐        ┌─────────────┐  ┌─────────────┐  │
│  │ deployment   │        │ replicas: 1 │  │ replicas: 5 │  │
│  │ service      │──merge─│ memory: 256M│  │ memory: 1G  │  │
│  │ configmap    │        │ image: :dev │  │ image: :1.0 │  │
│  │ ingress      │        │ ns: dev     │  │ ns: prod    │  │
│  └──────────────┘        └─────────────┘  └─────────────┘  │
│   YAML real e válido       Patches que só mudam o que       │
│   (pode usar direto)       difere por ambiente              │
└─────────────────────────────────────────────────────────────┘
```

**Conceito central:** os manifestos em `base/` são YAML Kubernetes completo e funcional. Cada overlay aplica apenas as diferenças necessárias para um ambiente específico, sem duplicar o restante.

#### Estrutura de diretórios

```
k8s/
├── base/                        # manifestos comuns a todos os ambientes
│   ├── kustomization.yaml       # índice dos recursos e transformações
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── ingress.yaml
├── overlays/
│   ├── dev/                     # customizações para desenvolvimento
│   │   ├── kustomization.yaml
│   │   ├── patch-replicas.yaml
│   │   └── patch-env.yaml
│   ├── staging/                 # customizações para staging
│   │   ├── kustomization.yaml
│   │   └── patch-replicas.yaml
│   └── prod/                    # customizações para produção
│       ├── kustomization.yaml
│       ├── patch-replicas.yaml
│       ├── patch-resources.yaml
│       └── patch-hpa.yaml
└── components/                  # (opcional) blocos reutilizáveis
    ├── monitoring/
    │   ├── kustomization.yaml
    │   └── service-monitor.yaml
    └── external-secrets/
        ├── kustomization.yaml
        └── external-secret.yaml
```

#### Base — Manifestos compartilhados

```yaml
# k8s/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml
  - ingress.yaml

commonLabels:
  app: api-app
  managed-by: kustomize

commonAnnotations:
  team: backend
```

```yaml
# k8s/base/deployment.yaml — manifesto real, sem placeholders
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-app
  template:
    metadata:
      labels:
        app: api-app
    spec:
      containers:
        - name: api-app
          image: minha-app:latest
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: app-config
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

```yaml
# k8s/base/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  SPRING_PROFILES_ACTIVE: "default"
  SERVER_PORT: "8080"
  LOG_LEVEL: "INFO"
```

#### Overlay de desenvolvimento

```yaml
# k8s/overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base              # herda tudo da base

namespace: dev              # todos os recursos ficam no namespace "dev"

namePrefix: dev-            # prefixo nos nomes: api-app → dev-api-app

images:
  - name: minha-app         # sobrescreve a tag da imagem
    newTag: dev-latest

patches:
  - path: patch-env.yaml    # patch de arquivo (Strategic Merge Patch)

configMapGenerator:          # gera ConfigMap com hash no nome (force rolling update)
  - name: app-config
    behavior: merge          # merge com o ConfigMap da base
    literals:
      - SPRING_PROFILES_ACTIVE=dev
      - LOG_LEVEL=DEBUG
```

```yaml
# k8s/overlays/dev/patch-env.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: api-app
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "250m"
```

#### Overlay de produção

```yaml
# k8s/overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base
  - hpa.yaml                # recurso adicional apenas em prod

namespace: production

images:
  - name: minha-app
    newName: registry.example.com/api-app
    newTag: "1.2.0"

patches:
  - path: patch-replicas.yaml
  - path: patch-resources.yaml
  # Patch inline (sem arquivo separado)
  - patch: |-
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: app-config
      data:
        SPRING_PROFILES_ACTIVE: "prod"
        LOG_LEVEL: "WARN"

configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://prod-db:5432/appdb
```

```yaml
# k8s/overlays/prod/patch-replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  replicas: 5
```

```yaml
# k8s/overlays/prod/patch-resources.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
spec:
  template:
    spec:
      containers:
        - name: api-app
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
```

```yaml
# k8s/overlays/prod/hpa.yaml — recurso adicional (só existe em prod)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

#### Tipos de patch

| Tipo | Quando usar | Exemplo |
|------|------------|---------|
| **Strategic Merge Patch** | Alterar campos específicos de um recurso (default) | Arquivo YAML com `apiVersion`, `kind`, `metadata.name` + campos a mudar |
| **JSON Patch (RFC 6902)** | Operações precisas: add, remove, replace, move | `[{"op": "replace", "path": "/spec/replicas", "value": 5}]` |
| **Patch inline** | Patches pequenos sem criar arquivo separado | Bloco `patch: \|-` no kustomization.yaml |

```yaml
# Exemplo de JSON Patch no kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: api-app
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value:
          name: EXTRA_VAR
          value: "valor"
```

#### Generators — ConfigMap e Secret

Kustomize pode gerar ConfigMaps e Secrets com hash no nome, forçando rolling update automático quando o conteúdo muda.

```yaml
# kustomization.yaml
configMapGenerator:
  - name: app-config
    literals:
      - SPRING_PROFILES_ACTIVE=prod
      - LOG_LEVEL=INFO
    files:
      - application.yml                    # monta arquivo como chave

secretGenerator:
  - name: db-credentials
    literals:
      - username=postgres
      - password=minha_senha
    type: Opaque

generatorOptions:
  disableNameSuffixHash: false   # true desabilita o hash (não recomendado)
```

O nome gerado inclui um hash do conteúdo: `app-config-k5m8h2g`. Quando o conteúdo muda, o hash muda, e o Deployment referencia o novo ConfigMap automaticamente — garantindo que os Pods sejam recriados com a nova configuração.

#### Components — Blocos reutilizáveis (opcional)

Components permitem agrupar funcionalidades opcionais que podem ser incluídas em qualquer overlay.

```yaml
# k8s/components/monitoring/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - service-monitor.yaml

patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-app
      spec:
        template:
          metadata:
            annotations:
              prometheus.io/scrape: "true"
              prometheus.io/port: "8080"
              prometheus.io/path: "/actuator/prometheus"
```

```yaml
# k8s/overlays/prod/kustomization.yaml — incluindo o component
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
components:
  - ../../components/monitoring    # inclui monitoring em prod
```

#### Comandos do Kustomize

```bash
# Gerar YAML final sem aplicar (preview)
kubectl kustomize k8s/overlays/prod/

# Aplicar diretamente
kubectl apply -k k8s/overlays/prod/

# Deletar recursos de um overlay
kubectl delete -k k8s/overlays/prod/

# Diff entre o que está no cluster e o que seria aplicado
kubectl diff -k k8s/overlays/prod/

# Gerar e salvar em arquivo (útil para debug/auditoria)
kubectl kustomize k8s/overlays/prod/ > manifesto-final.yaml

# Usar Kustomize standalone (versão mais recente que a embutida no kubectl)
kustomize build k8s/overlays/prod/ | kubectl apply -f -
```

#### Integração com CI/CD

```yaml
# .github/workflows/deploy.yml — usando Kustomize
- name: Set image tag
  run: |
    cd k8s/overlays/prod
    kustomize edit set image minha-app=registry.example.com/api-app:${{ github.sha }}

- name: Deploy
  run: kubectl apply -k k8s/overlays/prod/
```

#### Resumo visual do fluxo

```
┌──────────────────────────────────────────────────────────────────┐
│                    Fluxo do Kustomize                             │
│                                                                   │
│  base/                                                            │
│  ┌────────────────────┐                                          │
│  │ deployment.yaml    │                                          │
│  │ service.yaml       │─────┐                                    │
│  │ configmap.yaml     │     │                                    │
│  │ kustomization.yaml │     │                                    │
│  └────────────────────┘     │                                    │
│                              │  merge                             │
│  overlays/prod/              ▼                                    │
│  ┌────────────────────┐   ┌─────────────────┐                    │
│  │ patch-replicas     │   │                 │    kubectl apply   │
│  │ patch-resources    │──▶│  YAML final     │──────────────────▶ │
│  │ hpa.yaml (extra)   │   │  (tudo junto)   │    cluster K8s    │
│  │ kustomization.yaml │   │                 │                    │
│  └────────────────────┘   └─────────────────┘                    │
│                                                                   │
│  O base é YAML real. O overlay só descreve o que muda.           │
│  Nenhum template, nenhuma variável — apenas patches.             │
└──────────────────────────────────────────────────────────────────┘
```

### 41.5 Comparação: Kustomize vs Helm

| Aspecto | Kustomize | Helm |
|---------|-----------|------|
| **Abordagem** | Patches sobre YAML base (sem templates) | Templates Go com variáveis (`{{ .Values.x }}`) |
| **Complexidade** | Baixa — apenas YAML válido | Média — precisa entender Go templates e funções |
| **Embutido no kubectl** | Sim (`kubectl apply -k`) | Não (ferramenta separada: `helm`) |
| **Manifestos base** | YAML real, aplicável diretamente | Templates — não são YAML válido sem renderização |
| **Customização por ambiente** | Overlays com patches (arquivos separados por ambiente) | Arquivos `values.yaml` por ambiente |
| **Geração de ConfigMap/Secret** | `configMapGenerator` com hash automático | Via templates no chart |
| **Reutilização** | Overlays e Components | Charts publicáveis em repositório |
| **Rollback** | Via Git (`git revert` + `kubectl apply -k`) | `helm rollback` nativo com histórico de revisões |
| **Ecossistema** | Apenas manifestos próprios | Milhares de charts públicos (Bitnami, Artifact Hub) |
| **Validação** | `kubectl kustomize` + `kubectl diff -k` | `helm lint` + `helm template` + `helm diff` |
| **CI/CD** | `kustomize edit set image` + `kubectl apply -k` | `helm upgrade --install` |
| **Recomendado para** | Projetos simples/médios; quem quer YAML puro | Projetos complexos; distribuição de software; charts de terceiros |

> **Na prática:** muitos projetos usam **ambos** — Helm para instalar dependências de terceiros (PostgreSQL, Redis, Prometheus) e Kustomize para gerenciar os manifestos da própria aplicação por ambiente.

---

> **Documento produzido e revisado com apoio de agentes de IA. Última atualização: Junho/2026.**
