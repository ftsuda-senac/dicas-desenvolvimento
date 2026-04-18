# Terraform — Guia Abrangente de Infraestrutura como Código

> **Objetivo:** Cobrir os principais conceitos, recursos e padrões do Terraform para provisionamento e gerenciamento de infraestrutura como código (IaC), com exemplos práticos usando AWS como provedor de referência e boas práticas aplicáveis a qualquer nuvem.

---

## Sumário

1. [Fundamentos de IaC e Terraform](#1-fundamentos-de-iac-e-terraform)
2. [Instalação e Configuração](#2-instalação-e-configuração)
3. [Sintaxe HCL](#3-sintaxe-hcl)
4. [Providers](#4-providers)
5. [Resources](#5-resources)
6. [Variables e Outputs](#6-variables-e-outputs)
7. [Data Sources](#7-data-sources)
8. [State — Estado da Infraestrutura](#8-state--estado-da-infraestrutura)
9. [Fluxo de Trabalho: init, plan, apply, destroy](#9-fluxo-de-trabalho-init-plan-apply-destroy)
10. [Módulos](#10-módulos)
11. [Expressões, Funções e Meta-argumentos](#11-expressões-funções-e-meta-argumentos)
12. [Workspaces](#12-workspaces)
13. [Remote State e Backends](#13-remote-state-e-backends)
14. [Provisionamento Avançado](#14-provisionamento-avançado)
15. [Testes e Validação](#15-testes-e-validação)
16. [Integração com CI/CD](#16-integração-com-cicd)
17. [Boas Práticas e Organização de Projetos](#17-boas-práticas-e-organização-de-projetos)
18. [Estudos de Caso](#18-estudos-de-caso)
    - [18.1 Infraestrutura de uma API REST na AWS](#181-infraestrutura-de-uma-api-rest-na-aws)
    - [18.2 Ambiente Multi-conta com Organizations](#182-ambiente-multi-conta-com-organizations)
    - [18.3 Pipeline Completo com GitHub Actions](#183-pipeline-completo-com-github-actions)

---

## 1. Fundamentos de IaC e Terraform

### 1.1 O que é Infraestrutura como Código

Infraestrutura como Código (IaC) é a prática de gerenciar e provisionar recursos de infraestrutura por meio de arquivos de configuração legíveis por humanos, versionados no mesmo repositório que o código da aplicação — em vez de configurações manuais via console ou scripts ad-hoc.

```
Abordagem Tradicional                    Abordagem IaC
─────────────────────────                ──────────────────────────
Console AWS / Portal Azure               Arquivos .tf no Git
Cliques manuais                          Código declarativo
Sem histórico de mudanças                git log / pull requests
Difícil de reproduzir                    terraform apply = resultado idêntico
Documentação desatualizada               Código é a documentação
```

**Benefícios principais:**

| Benefício | Descrição |
|-----------|-----------|
| **Reprodutibilidade** | O mesmo código gera a mesma infraestrutura em qualquer ambiente |
| **Versionamento** | Histórico completo de mudanças, rollback, code review |
| **Automação** | Integração com pipelines CI/CD sem intervenção manual |
| **Idempotência** | Aplicar o mesmo código múltiplas vezes não cria recursos duplicados |
| **Documentação viva** | O estado desejado da infraestrutura está sempre no código |

### 1.2 O que é o Terraform

O Terraform é uma ferramenta de IaC **declarativa** e **agnóstica de nuvem**, criada pela HashiCorp. Em vez de descrever *como* provisionar (scripts imperativos), você descreve *o que* quer ter (estado desejado) e o Terraform calcula o plano de execução necessário para chegar lá.

```
Código Terraform (.tf)
        │
        ▼
  terraform plan ──► Diferença entre estado atual e estado desejado
        │
        ▼
  terraform apply ──► Chamadas às APIs dos providers (AWS, GCP, Azure…)
        │
        ▼
  terraform.tfstate ──► Registro do estado atual da infraestrutura
```

### 1.3 Terraform vs. outras ferramentas de IaC

| Ferramenta | Abordagem | Escopo | Linguagem |
|------------|-----------|--------|-----------|
| **Terraform** | Declarativa | Multi-cloud, infraestrutura | HCL |
| **Pulumi** | Imperativa/Declarativa | Multi-cloud | Python, TypeScript, Go… |
| **CloudFormation** | Declarativa | Apenas AWS | YAML/JSON |
| **Ansible** | Imperativa | Configuração de SO/apps | YAML |
| **Chef/Puppet** | Declarativa | Configuração de SO/apps | Ruby DSL |
| **CDK** | Imperativa | AWS (gera CloudFormation) | Python, TypeScript… |

### 1.4 Arquitetura interna do Terraform

```
┌─────────────────────────────────────────────────────────┐
│                   Terraform CLI                         │
│                                                         │
│  ┌─────────────┐   ┌────────────┐   ┌───────────────┐  │
│  │  HCL Parser │   │   Planner  │   │    Executor   │  │
│  │             │──►│  (diff)    │──►│  (apply/      │  │
│  │  .tf files  │   │            │   │   destroy)    │  │
│  └─────────────┘   └────────────┘   └───────┬───────┘  │
│                                             │           │
└─────────────────────────────────────────────┼───────────┘
                                              │ RPC / gRPC
              ┌────────────────────────────────┼────────────┐
              │           Provider Plugins      │            │
              │                                ▼            │
              │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
              │  │  AWS     │  │  GCP     │  │  Azure   │  │
              │  │ Provider │  │ Provider │  │ Provider │  │
              │  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
              └───────┼─────────────┼──────────────┼────────┘
                      │             │              │
                   AWS API       GCP API       Azure API
```

---

## 2. Instalação e Configuração

### 2.1 Instalação do Terraform CLI

**Linux (via apt):**

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor \
  -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform
```

**macOS (via Homebrew):**

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

**Windows (via Chocolatey):**

```powershell
choco install terraform
```

**Verificar instalação:**

```bash
terraform version
# Terraform v1.9.x
```

### 2.2 Gerenciador de versões — tfenv

Para projetos com versões diferentes do Terraform, use o `tfenv`:

```bash
# Instalar tfenv
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
export PATH="$HOME/.tfenv/bin:$PATH"

# Listar versões disponíveis
tfenv list-remote

# Instalar versão específica
tfenv install 1.9.5

# Usar uma versão
tfenv use 1.9.5

# Arquivo .terraform-version no projeto (lido automaticamente pelo tfenv)
echo "1.9.5" > .terraform-version
```

### 2.3 Configuração das credenciais AWS

O provider AWS lê as credenciais das seguintes fontes (em ordem de prioridade):

```
1. Variáveis de ambiente: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
2. Arquivo ~/.aws/credentials
3. Perfil de instância EC2 / ECS task role
4. AWS SSO (Single Sign-On)
```

```bash
# Configurar via AWS CLI
aws configure --profile meu-projeto
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region name: us-east-1
# Default output format: json

# Usar o perfil no Terraform
export AWS_PROFILE=meu-projeto
```

---

## 3. Sintaxe HCL

### 3.1 HashiCorp Configuration Language

O HCL (HashiCorp Configuration Language) é a linguagem declarativa usada pelo Terraform. É projetada para ser legível por humanos e editável por máquinas.

**Estrutura básica de um bloco:**

```hcl
# Tipo de bloco  "Tipo do recurso"  "Nome local"
resource           "aws_instance"   "web_server" {
  # Argumentos (atributos de configuração)
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}
```

### 3.2 Tipos de dados

```hcl
# String
nome = "minha-aplicacao"

# Número
porta = 8080
preco = 3.14

# Booleano
habilitado = true

# Lista (todos os elementos do mesmo tipo)
regioes = ["us-east-1", "us-west-2", "eu-west-1"]

# Set (lista sem duplicatas e sem ordem garantida)
zonas_disponiveis = toset(["a", "b", "c"])

# Map (chave → valor do mesmo tipo)
tags = {
  Ambiente = "producao"
  Projeto  = "minha-app"
  Time     = "backend"
}

# Object (chave → valor de tipos diferentes)
configuracao = {
  nome     = "api"      # string
  replicas = 3          # number
  ativo    = true       # bool
}

# Tuple (lista com elementos de tipos diferentes)
mixed = ["texto", 42, true]
```

### 3.3 Expressões de referência

```hcl
# Referenciar atributo de outro recurso
subnet_id = aws_subnet.private.id

# Referenciar variável
instance_type = var.instance_type

# Referenciar output de módulo
vpc_id = module.network.vpc_id

# Referenciar data source
ami = data.aws_ami.ubuntu.id

# Interpolação de string
name = "app-${var.environment}-server"

# Condicional ternária
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
```

### 3.4 Comentários

```hcl
# Comentário de linha única (estilo Shell)
// Comentário de linha única (estilo C)

/*
  Comentário
  de múltiplas linhas
*/
```

---

## 4. Providers

### 4.1 O que são Providers

Providers são plugins que traduzem os recursos Terraform em chamadas de API para um serviço específico (AWS, GCP, Azure, Kubernetes, GitHub, Datadog, etc.). Cada provider é publicado no [Terraform Registry](https://registry.terraform.io/).

```
Terraform Core
      │
      │  via RPC
      ▼
Provider AWS ──► AWS API (EC2, S3, RDS, VPC…)
Provider GCP ──► GCP API (GCE, GCS, GKE…)
Provider K8s ──► Kubernetes API Server
```

### 4.2 Declaração e configuração

```hcl
# versions.tf — declare as versões esperadas
terraform {
  required_version = ">= 1.9.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"   # aceita 5.x, não aceita 6.0
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}

# provider.tf — configure o provider
provider "aws" {
  region  = var.aws_region
  profile = var.aws_profile

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Repository  = "github.com/minha-org/infra"
      Environment = var.environment
    }
  }
}
```

### 4.3 Múltiplos providers (alias)

Use `alias` para trabalhar com o mesmo provider em configurações diferentes (ex.: múltiplas regiões):

```hcl
# Provider principal em us-east-1
provider "aws" {
  region = "us-east-1"
}

# Provider secundário em eu-west-1
provider "aws" {
  alias  = "europa"
  region = "eu-west-1"
}

# Usando o provider com alias
resource "aws_s3_bucket" "backup_europa" {
  provider = aws.europa
  bucket   = "minha-app-backup-eu"
}
```

---

## 5. Resources

### 5.1 Declaração de recursos

Os recursos são o elemento central do Terraform. Cada bloco `resource` representa um objeto de infraestrutura gerenciado.

```hcl
resource "<TIPO_DO_PROVIDER>" "<NOME_LOCAL>" {
  # argumentos...
}
```

**Exemplos básicos:**

```hcl
# Instância EC2
resource "aws_instance" "app_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.small"
  subnet_id     = aws_subnet.private.id

  vpc_security_group_ids = [aws_security_group.app.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get install -y openjdk-21-jre
    java -jar /opt/app.jar
  EOF

  tags = {
    Name = "app-server"
  }
}

# Bucket S3
resource "aws_s3_bucket" "assets" {
  bucket = "minha-app-assets-${random_id.suffix.hex}"
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id

  versioning_configuration {
    status = "Enabled"
  }
}

# Banco de dados RDS
resource "aws_db_instance" "postgres" {
  identifier        = "minha-app-db"
  engine            = "postgres"
  engine_version    = "16"
  instance_class    = "db.t3.micro"
  allocated_storage = 20

  db_name  = "appdb"
  username = var.db_username
  password = var.db_password  # use secrets manager na prática

  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]

  skip_final_snapshot = false
  final_snapshot_identifier = "minha-app-db-final"

  tags = {
    Name = "app-database"
  }
}
```

### 5.2 Dependências entre recursos

O Terraform constrói automaticamente um grafo de dependências baseado nas referências entre recursos.

```
aws_vpc.main
    │
    ├── aws_subnet.public  ◄──── aws_route_table_association.public
    │         │                           │
    │         │                    aws_route_table.public ◄── aws_internet_gateway.main
    │         │
    │         └── aws_instance.web ◄── aws_security_group.web
    │
    └── aws_subnet.private ◄──── aws_route_table_association.private
              │
              └── aws_db_instance.postgres ◄── aws_db_subnet_group.main
```

**Dependência explícita com `depends_on`:**

Use `depends_on` somente quando a dependência não é óbvia pelas referências:

```hcl
resource "aws_iam_role_policy_attachment" "app_policy" {
  role       = aws_iam_role.app.name
  policy_arn = aws_iam_policy.app.arn
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.small"

  # A instância precisa do role policy antes de iniciar
  depends_on = [aws_iam_role_policy_attachment.app_policy]
}
```

### 5.3 Lifecycle

O bloco `lifecycle` controla o comportamento do Terraform durante atualizações e destruições:

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    # Cria o novo recurso antes de destruir o antigo (zero-downtime updates)
    create_before_destroy = true

    # Impede que o Terraform destrua este recurso
    prevent_destroy = true

    # Ignora mudanças nesses atributos (ex.: ami atualizada fora do Terraform)
    ignore_changes = [ami, user_data]

    # Validação customizada antes de criar/atualizar
    precondition {
      condition     = var.instance_type != "t2.micro"
      error_message = "t2.micro está obsoleto; use t3.micro ou superior."
    }
  }
}
```

---

## 6. Variables e Outputs

### 6.1 Input Variables

Variáveis tornam configurações reutilizáveis e parametrizáveis.

```hcl
# variables.tf

variable "environment" {
  type        = string
  description = "Ambiente de deployment (dev, staging, prod)"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "O ambiente deve ser dev, staging ou prod."
  }
}

variable "instance_type" {
  type        = string
  description = "Tipo da instância EC2"
  default     = "t3.micro"
}

variable "allowed_cidr_blocks" {
  type        = list(string)
  description = "CIDRs com acesso permitido ao load balancer"
  default     = ["0.0.0.0/0"]
}

variable "db_config" {
  type = object({
    instance_class    = string
    allocated_storage = number
    multi_az          = bool
  })
  description = "Configuração do banco de dados"
  default = {
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    multi_az          = false
  }
}

variable "db_password" {
  type        = string
  description = "Senha do banco de dados"
  sensitive   = true   # não exibido em logs e outputs
}
```

### 6.2 Formas de passar valores às variáveis

```
Prioridade (maior → menor):
────────────────────────────────────────────────────────
1. -var "nome=valor"                   (linha de comando)
2. -var-file="arquivo.tfvars"          (linha de comando)
3. terraform.tfvars ou *.auto.tfvars   (arquivo automático)
4. Variáveis de ambiente TF_VAR_nome   (ambiente)
5. Valor default declarado na variável
```

**Arquivo `terraform.tfvars`:**

```hcl
environment  = "staging"
instance_type = "t3.small"

allowed_cidr_blocks = [
  "10.0.0.0/8",
  "172.16.0.0/12"
]

db_config = {
  instance_class    = "db.t3.small"
  allocated_storage = 50
  multi_az          = true
}
```

**Arquivo `prod.tfvars`:**

```hcl
environment  = "prod"
instance_type = "t3.large"

db_config = {
  instance_class    = "db.r6g.large"
  allocated_storage = 200
  multi_az          = true
}
```

```bash
# Aplicar com arquivo de variáveis específico
terraform apply -var-file="prod.tfvars"
```

### 6.3 Output Values

Outputs exportam informações dos recursos criados, úteis para integração entre módulos e para exibir informações ao usuário.

```hcl
# outputs.tf

output "app_server_ip" {
  description = "IP público do servidor de aplicação"
  value       = aws_instance.app_server.public_ip
}

output "database_endpoint" {
  description = "Endpoint de conexão do banco de dados"
  value       = aws_db_instance.postgres.endpoint
  sensitive   = true   # não exibido no terminal, mas acessível via API
}

output "load_balancer_dns" {
  description = "DNS do Application Load Balancer"
  value       = aws_lb.main.dns_name
}

output "s3_bucket_arn" {
  description = "ARN do bucket S3 de assets"
  value       = aws_s3_bucket.assets.arn
}
```

```bash
# Ver outputs após o apply
terraform output

# Ver um output específico
terraform output app_server_ip

# Ver em formato JSON (útil em scripts)
terraform output -json
```

---

## 7. Data Sources

### 7.1 O que são Data Sources

Data sources permitem que o Terraform **consulte** informações de recursos existentes ou externos — sem criá-los ou gerenciá-los. São somente leitura.

```hcl
# Buscar a AMI Ubuntu mais recente
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Buscar VPC existente pelo nome
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["vpc-producao"]
  }
}

# Buscar zonas de disponibilidade da região
data "aws_availability_zones" "available" {
  state = "available"
}

# Buscar secret do AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "minha-app/prod/db-password"
}

# Buscar certificado ACM existente
data "aws_acm_certificate" "domain" {
  domain      = "*.minha-app.com.br"
  statuses    = ["ISSUED"]
  most_recent = true
}
```

**Usando os data sources:**

```hcl
resource "aws_instance" "app" {
  # Usar a AMI buscada pelo data source
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.small"
  subnet_id     = data.aws_vpc.existing.id
  availability_zone = data.aws_availability_zones.available.names[0]
}

resource "aws_db_instance" "postgres" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
  # ...
}
```

---

## 8. State — Estado da Infraestrutura

### 8.1 O que é o State

O state (`terraform.tfstate`) é um arquivo JSON que mapeia os recursos declarados no código para os recursos reais criados no provedor. Ele é o "cérebro" do Terraform.

```
Código .tf                    State                      Infraestrutura real
──────────────────            ─────────────────────      ──────────────────────
resource "aws_instance"       {                          EC2 Instance
  "app_server" {       ──►      "aws_instance.        ──► id: i-0abc123def456789
  ami = "ami-..."               app_server": {             ami: ami-0c55b...
  instance_type = "t3.micro"      "id": "i-0abc...",       instance_type: t3.micro
}                                 "ami": "ami-...",        state: running
                                  ...
                              }
                            }
```

**Por que o state é importante:**

| Função | Descrição |
|--------|-----------|
| **Mapeamento** | Liga recursos do código a IDs reais no provedor |
| **Performance** | Evita chamadas desnecessárias às APIs (cache local) |
| **Dependências** | Guarda metadados para calcular o grafo de dependências |
| **diff** | Permite calcular o plano de mudanças (`terraform plan`) |

### 8.2 Comandos de gerenciamento do state

```bash
# Listar todos os recursos no state
terraform state list

# Exibir detalhes de um recurso
terraform state show aws_instance.app_server

# Mover recurso para novo endereço (renomear sem recriar)
terraform state mv aws_instance.old_name aws_instance.new_name

# Remover recurso do state (sem destruí-lo na nuvem)
terraform state rm aws_instance.app_server

# Importar recurso existente para o state
terraform import aws_instance.app_server i-0abc123def456789

# Baixar e salvar o state atual (debugging)
terraform state pull > current-state.json
```

### 8.3 Cuidados com o state

```
NUNCA faça:
──────────────────────────────────────────────────────────────────
✗ Editar terraform.tfstate manualmente (exceto em emergências)
✗ Deletar terraform.tfstate sem fazer backup
✗ Commitar terraform.tfstate com credenciais sensíveis
✗ Usar state local em ambientes de equipe (use remote state)
✗ Rodar dois terraform apply simultâneos no mesmo state (use locking)

SEMPRE faça:
──────────────────────────────────────────────────────────────────
✓ Adicionar terraform.tfstate ao .gitignore
✓ Usar remote state em equipes (S3, GCS, Terraform Cloud)
✓ Habilitar state locking (DynamoDB para AWS)
✓ Habilitar versionamento no bucket de state (S3 versioning)
✓ Fazer backup antes de operações de state manual
```

---

## 9. Fluxo de Trabalho: init, plan, apply, destroy

### 9.1 Visão geral do fluxo

```
Escrever código .tf
        │
        ▼
terraform init ──► Baixa providers e módulos, inicializa backend
        │
        ▼
terraform validate ──► Valida sintaxe e semântica do HCL
        │
        ▼
terraform fmt ──► Formata o código (estilo canônico)
        │
        ▼
terraform plan ──► Mostra o que será criado/modificado/destruído
        │           (não aplica nenhuma mudança)
        ▼
terraform apply ──► Executa o plano e modifica a infraestrutura
        │
        ▼
  Infraestrutura criada/atualizada
        │
        ▼ (quando necessário)
terraform destroy ──► Destroi TODOS os recursos gerenciados
```

### 9.2 terraform init

```bash
# Inicializar o diretório de trabalho
terraform init

# Reforçar o download dos providers (após atualizar versões)
terraform init -upgrade

# Especificar backend diferente do declarado no código
terraform init -backend-config="bucket=meu-state-bucket"

# Saída típica:
# Initializing the backend...
# Initializing provider plugins...
# - Finding hashicorp/aws versions matching "~> 5.0"...
# - Installing hashicorp/aws v5.62.0...
# Terraform has been successfully initialized!
```

### 9.3 terraform plan

```bash
# Gerar plano de execução
terraform plan

# Salvar o plano em arquivo (recomendado para CI/CD)
terraform plan -out=tfplan

# Plano com variáveis
terraform plan -var-file="prod.tfvars"

# Plano de destruição
terraform plan -destroy

# Plano para um recurso específico
terraform plan -target=aws_instance.app_server
```

**Lendo a saída do plan:**

```
  # aws_instance.app_server will be created
+ resource "aws_instance" "app_server" {   # + = será CRIADO
    + ami           = "ami-0c55b159cbfafe1f0"
    + instance_type = "t3.small"
    + id            = (known after apply)
  }

  # aws_instance.web will be updated in-place
~ resource "aws_instance" "web" {           # ~ = será MODIFICADO
    ~ instance_type = "t3.micro" -> "t3.small"
      id            = "i-0abc123def456789"
  }

  # aws_instance.old_server will be destroyed
- resource "aws_instance" "old_server" {   # - = será DESTRUÍDO
    - ami           = "ami-0c55b159cbfafe1f0" -> null
    - id            = "i-0def456ghi789012" -> null
  }

Plan: 1 to add, 1 to change, 1 to destroy.
```

### 9.4 terraform apply

```bash
# Aplicar com confirmação interativa
terraform apply

# Aplicar plano salvo (sem pedir confirmação)
terraform apply tfplan

# Aplicar sem confirmação (para CI/CD)
terraform apply -auto-approve

# Aplicar apenas um recurso específico
terraform apply -target=aws_instance.app_server

# Aplicar com variáveis
terraform apply -var="environment=prod" -var-file="prod.tfvars"
```

### 9.5 terraform destroy

```bash
# Destruir todos os recursos (pede confirmação)
terraform destroy

# Destruir sem confirmação (cuidado!)
terraform destroy -auto-approve

# Destruir apenas um recurso
terraform destroy -target=aws_instance.app_server

# Planejar a destruição sem executar
terraform plan -destroy
```

---

## 10. Módulos

### 10.1 O que são módulos

Módulos são conjuntos de recursos Terraform agrupados em uma unidade reutilizável. Todo diretório com arquivos `.tf` é um módulo. O diretório raiz é chamado de **root module**.

```
Sem módulos                          Com módulos
──────────────────────────           ──────────────────────────────────
main.tf (500 linhas)                 main.tf (50 linhas)
                                     │
                                     ├── module "network" { }   ──► modules/network/
                                     ├── module "compute" { }   ──► modules/compute/
                                     └── module "database" { }  ──► modules/database/
```

### 10.2 Estrutura de um módulo

```
modules/
└── compute/
    ├── main.tf          # Recursos principais
    ├── variables.tf     # Inputs do módulo
    ├── outputs.tf       # Outputs do módulo
    ├── versions.tf      # Versões de providers
    └── README.md        # Documentação
```

**`modules/compute/variables.tf`:**

```hcl
variable "environment" {
  type        = string
  description = "Nome do ambiente"
}

variable "instance_count" {
  type        = number
  description = "Número de instâncias"
  default     = 1
}

variable "instance_type" {
  type        = string
  description = "Tipo da instância EC2"
}

variable "subnet_ids" {
  type        = list(string)
  description = "IDs das subnets"
}

variable "security_group_ids" {
  type        = list(string)
  description = "IDs dos security groups"
}
```

**`modules/compute/main.tf`:**

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-24.04-amd64-server-*"]
  }
}

resource "aws_instance" "this" {
  count = var.instance_count

  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_ids[count.index % length(var.subnet_ids)]

  vpc_security_group_ids = var.security_group_ids

  tags = {
    Name        = "${var.environment}-server-${count.index + 1}"
    Environment = var.environment
  }
}
```

**`modules/compute/outputs.tf`:**

```hcl
output "instance_ids" {
  description = "IDs das instâncias criadas"
  value       = aws_instance.this[*].id
}

output "private_ips" {
  description = "IPs privados das instâncias"
  value       = aws_instance.this[*].private_ip
}
```

### 10.3 Usando o módulo

```hcl
# main.tf (root module)

module "compute_prod" {
  source = "./modules/compute"  # caminho local

  environment        = "prod"
  instance_count     = 3
  instance_type      = "t3.large"
  subnet_ids         = module.network.private_subnet_ids
  security_group_ids = [aws_security_group.app.id]
}

# Usar output do módulo
output "app_server_ids" {
  value = module.compute_prod.instance_ids
}
```

### 10.4 Módulos do Terraform Registry

```hcl
# Módulo público do Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "minha-app-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = false  # HA: um NAT por AZ

  tags = {
    Terraform   = "true"
    Environment = var.environment
  }
}
```

---

## 11. Expressões, Funções e Meta-argumentos

### 11.1 count

Cria múltiplas instâncias de um recurso com base em um número:

```hcl
resource "aws_instance" "web" {
  count = 3

  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  # count.index: 0, 1, 2
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

# Referenciar instância específica
output "first_server_ip" {
  value = aws_instance.web[0].private_ip
}

# Referenciar todas as instâncias
output "all_server_ips" {
  value = aws_instance.web[*].private_ip
}
```

### 11.2 for_each

Cria recursos com base em um map ou set, usando chaves como identificadores:

```hcl
variable "servidores" {
  type = map(object({
    instance_type = string
    zone          = string
  }))
  default = {
    "api"     = { instance_type = "t3.small",  zone = "us-east-1a" }
    "worker"  = { instance_type = "t3.medium", zone = "us-east-1b" }
    "monitor" = { instance_type = "t3.micro",  zone = "us-east-1c" }
  }
}

resource "aws_instance" "servidores" {
  for_each = var.servidores

  ami               = data.aws_ami.ubuntu.id
  instance_type     = each.value.instance_type
  availability_zone = each.value.zone

  tags = {
    Name = "server-${each.key}"
    Role = each.key
  }
}

# Referenciar servidor específico pela chave
output "api_server_ip" {
  value = aws_instance.servidores["api"].private_ip
}
```

**`count` vs. `for_each`:**

```
count                              for_each
──────────────────────────         ──────────────────────────────
Indexado por número [0, 1, 2]      Indexado por chave ["api", "worker"]
Remover item do meio               Remover "worker" não afeta "api"
reindexará e recriará itens        outros recursos permanecem intactos
seguintes                          
Ideal para recursos idênticos      Ideal para recursos parametrizados
```

### 11.3 Expressões for

Transformam coleções:

```hcl
# Criar lista de nomes a partir de objetos
locals {
  server_names = [for s in aws_instance.web : s.tags["Name"]]
  # ["web-server-1", "web-server-2", "web-server-3"]

  # Filtrar apenas instâncias em execução
  running_ids = [for s in aws_instance.web : s.id if s.instance_state == "running"]

  # Transformar lista em map
  ip_by_name = { for s in aws_instance.web : s.tags["Name"] => s.private_ip }
  # { "web-server-1" = "10.0.1.10", "web-server-2" = "10.0.1.11", ... }
}
```

### 11.4 Funções built-in

```hcl
locals {
  # Funções de string
  nome_lower  = lower("MinhaApp")         # "minhaapp"
  nome_upper  = upper("MinhaApp")         # "MINHAAPP"
  nome_trim   = trimspace("  api  ")      # "api"
  prefixo     = substr("app-prod", 0, 3)  # "app"
  tem_prefixo = startswith("app-prod", "app")  # true
  joined      = join("-", ["app", "prod", "us"])  # "app-prod-us"
  parts       = split("-", "app-prod-us")         # ["app", "prod", "us"]
  replaced    = replace("app_prod", "_", "-")     # "app-prod"
  formatted   = format("server-%03d", 5)          # "server-005"

  # Funções de coleção
  comprimento  = length(["a", "b", "c"])         # 3
  primeiro     = element(["a", "b", "c"], 0)      # "a"
  ultimo       = element(["a", "b", "c"], -1)     # "c"
  aplanado     = flatten([["a", "b"], ["c"]])     # ["a", "b", "c"]
  chaves       = keys({ a = 1, b = 2 })           # ["a", "b"]
  valores      = values({ a = 1, b = 2 })         # [1, 2]
  compactado   = compact(["a", "", "b", null])    # ["a", "b"]
  distinto     = distinct(["a", "b", "a"])        # ["a", "b"]
  merged       = merge({ a = 1 }, { b = 2 })      # { a = 1, b = 2 }

  # Funções numéricas
  minimo = min(3, 1, 2)   # 1
  maximo = max(3, 1, 2)   # 3
  abs_val = abs(-5)        # 5
  ceil_val = ceil(2.1)     # 3
  floor_val = floor(2.9)   # 2

  # Funções de tipo
  como_string = tostring(42)              # "42"
  como_numero = tonumber("42")            # 42
  como_lista  = tolist(toset(["a", "b"])) # ["a", "b"]
  como_set    = toset(["a", "b", "a"])    # {"a", "b"}

  # Funções de codificação
  base64enc  = base64encode("hello")           # "aGVsbG8="
  base64dec  = base64decode("aGVsbG8=")        # "hello"
  json_enc   = jsonencode({ key = "value" })   # "{\"key\":\"value\"}"
  json_dec   = jsondecode("{\"key\":\"value\"}") # { key = "value" }

  # Funções de arquivo
  conteudo   = file("${path.module}/scripts/init.sh")
  template   = templatefile("${path.module}/templates/config.tpl", {
    ambiente = var.environment
    porta    = var.app_port
  })
}
```

### 11.5 locals

O bloco `locals` define valores calculados reutilizáveis no módulo:

```hcl
locals {
  # Prefixo padrão para nomear recursos
  nome_prefix = "${var.project}-${var.environment}"

  # Tags comuns a todos os recursos
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedAt   = timestamp()
  }

  # Configuração calculada
  is_production = var.environment == "prod"
  replica_count = local.is_production ? 3 : 1
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = local.is_production ? "t3.large" : "t3.micro"

  tags = merge(local.common_tags, {
    Name = "${local.nome_prefix}-app-server"
  })
}
```

---

## 12. Workspaces

### 12.1 O que são Workspaces

Workspaces permitem manter múltiplos states distintos para o mesmo código Terraform — útil para gerenciar ambientes com a mesma estrutura.

```
Workspace: default    →  terraform.tfstate
Workspace: staging    →  terraform.tfstate.d/staging/terraform.tfstate
Workspace: prod       →  terraform.tfstate.d/prod/terraform.tfstate
```

### 12.2 Comandos de workspace

```bash
# Listar workspaces
terraform workspace list
# * default
#   staging
#   prod

# Criar workspace
terraform workspace new staging

# Selecionar workspace
terraform workspace select prod

# Mostrar workspace atual
terraform workspace show
# prod

# Deletar workspace (deve estar vazio)
terraform workspace delete staging
```

### 12.3 Usando workspace no código

```hcl
locals {
  # Configuração por ambiente via workspace
  config = {
    default = {
      instance_type = "t3.micro"
      min_size      = 1
      max_size      = 2
    }
    staging = {
      instance_type = "t3.small"
      min_size      = 1
      max_size      = 3
    }
    prod = {
      instance_type = "t3.large"
      min_size      = 3
      max_size      = 10
    }
  }

  env_config = local.config[terraform.workspace]
}

resource "aws_autoscaling_group" "app" {
  min_size         = local.env_config.min_size
  max_size         = local.env_config.max_size
  desired_capacity = local.env_config.min_size

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
}
```

> **Nota:** Workspaces são adequados para variações leves do mesmo ambiente. Para ambientes com configurações muito distintas (contas AWS separadas, regiões diferentes), prefira **diretórios separados** com seus próprios states.

---

## 13. Remote State e Backends

### 13.1 Por que usar Remote State

```
State local (terraform.tfstate)          Remote State (S3 + DynamoDB)
────────────────────────────────         ─────────────────────────────────────
Apenas na máquina do desenvolvedor       Acessível por toda a equipe
Sem controle de concorrência             Locking via DynamoDB evita conflitos
Sem versionamento automático             S3 versioning permite rollback
Credenciais podem vazar via Git          Arquivo separado do repositório
```

### 13.2 Configurar backend S3 (AWS)

**Criar os recursos de state com Terraform (bootstrapping):**

```hcl
# bootstrap/main.tf — executado uma vez para criar a infraestrutura de state

resource "aws_s3_bucket" "terraform_state" {
  bucket = "minha-org-terraform-state"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "minha-org-terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

**Configurar o backend nos projetos:**

```hcl
# versions.tf
terraform {
  required_version = ">= 1.9.0"

  backend "s3" {
    bucket         = "minha-org-terraform-state"
    key            = "projetos/minha-app/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "minha-org-terraform-locks"
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 13.3 Remote State como Data Source

Permite compartilhar outputs entre projetos Terraform distintos:

```hcl
# Projeto "app" lendo outputs do projeto "network"
data "terraform_remote_state" "network" {
  backend = "s3"

  config = {
    bucket = "minha-org-terraform-state"
    key    = "projetos/network/prod/terraform.tfstate"
    region = "us-east-1"
  }
}

# Usando os outputs do state remoto
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.private_subnet_ids[0]
}
```

### 13.4 Diagrama de organização de states

```
minha-org-terraform-state (S3 Bucket)
├── projetos/
│   ├── network/
│   │   ├── dev/terraform.tfstate
│   │   ├── staging/terraform.tfstate
│   │   └── prod/terraform.tfstate
│   │
│   ├── minha-app/
│   │   ├── dev/terraform.tfstate
│   │   ├── staging/terraform.tfstate
│   │   └── prod/terraform.tfstate
│   │
│   └── data-platform/
│       ├── dev/terraform.tfstate
│       └── prod/terraform.tfstate
│
└── bootstrap/
    └── terraform.tfstate
```

---

## 14. Provisionamento Avançado

### 14.1 Provisioners

Provisioners executam scripts ou comandos durante a criação ou destruição de um recurso. Devem ser usados como **último recurso** — prefira sempre ferramentas de gerenciamento de configuração (Ansible, Chef, user_data).

```hcl
resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.small"
  key_name      = aws_key_pair.deployer.key_name

  # Copiar arquivo para a instância
  provisioner "file" {
    source      = "scripts/install.sh"
    destination = "/tmp/install.sh"

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Executar comando na instância
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/install.sh",
      "sudo /tmp/install.sh",
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Executar comando localmente ao destruir
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Recurso ${self.id} será destruído' >> destroy.log"
  }
}
```

### 14.2 templatefile

Gera conteúdo dinâmico a partir de templates:

**`templates/user_data.sh.tpl`:**

```bash
#!/bin/bash
set -euo pipefail

# Configurações do ambiente
export APP_ENV="${environment}"
export APP_PORT="${app_port}"
export DB_HOST="${db_host}"
export DB_NAME="${db_name}"

# Instalar Java
apt-get update -y
apt-get install -y openjdk-21-jre-headless

# Criar usuário da aplicação
useradd -r -s /bin/false app

# Configurar a aplicação
%{ for key, value in env_vars ~}
echo "${key}=${value}" >> /etc/app.env
%{ endfor ~}

# Iniciar serviço
systemctl enable app
systemctl start app
```

**Usando o template:**

```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "${local.nome_prefix}-lt-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  user_data = base64encode(templatefile("${path.module}/templates/user_data.sh.tpl", {
    environment = var.environment
    app_port    = var.app_port
    db_host     = aws_db_instance.postgres.address
    db_name     = var.db_name
    env_vars = {
      JAVA_OPTS    = "-Xms512m -Xmx1024m"
      LOG_LEVEL    = var.environment == "prod" ? "WARN" : "DEBUG"
    }
  }))
}
```

### 14.3 moved — Renomear recursos sem recriar

O bloco `moved` registra renomeações de recursos, evitando destruição e recriação:

```hcl
# Renomear recurso (sem recriar)
moved {
  from = aws_instance.servidor_legado
  to   = aws_instance.app_server
}

# Mover recurso para dentro de um módulo
moved {
  from = aws_security_group.app
  to   = module.compute.aws_security_group.app
}
```

---

## 15. Testes e Validação

### 15.1 terraform validate e fmt

```bash
# Verificar sintaxe e semântica
terraform validate
# Success! The configuration is valid.

# Formatar código (modifica arquivos no lugar)
terraform fmt -recursive

# Verificar formatação sem modificar (útil em CI)
terraform fmt -check -recursive
```

### 15.2 Testes nativos com terraform test

O Terraform 1.6+ inclui um framework de testes nativo. Os arquivos de teste usam a extensão `.tftest.hcl`.

**`tests/compute.tftest.hcl`:**

```hcl
# Variáveis para os testes
variables {
  environment   = "test"
  instance_type = "t3.micro"
  instance_count = 1
}

# Teste: verificar que o módulo cria a instância corretamente
run "cria_instancia_com_tipo_correto" {
  command = plan

  assert {
    condition     = aws_instance.app.instance_type == "t3.micro"
    error_message = "O tipo da instância deveria ser t3.micro"
  }

  assert {
    condition     = length(aws_instance.app[*].id) == 1
    error_message = "Deveria criar exatamente 1 instância"
  }
}

# Teste: verificar tags obrigatórias
run "tags_obrigatorias_presentes" {
  command = plan

  assert {
    condition     = aws_instance.app.tags["Environment"] == "test"
    error_message = "Tag Environment ausente ou incorreta"
  }

  assert {
    condition     = aws_instance.app.tags["ManagedBy"] == "Terraform"
    error_message = "Tag ManagedBy ausente"
  }
}
```

```bash
# Executar os testes
terraform test

# Executar testes com output detalhado
terraform test -verbose
```

### 15.3 Checov — Análise estática de segurança

```bash
# Instalar Checkov
pip install checkov

# Analisar diretório Terraform
checkov -d . --framework terraform

# Exemplo de saída:
# Check: CKV_AWS_8: "Ensure all data stored in the Launch configuration EBS is securely encrypted"
# PASSED for resource: aws_launch_configuration.app
#
# Check: CKV_AWS_79: "Ensure Instance Metadata Service Version 1 is not enabled"
# FAILED for resource: aws_instance.app
# File: /main.tf:10-25
```

### 15.4 tflint — Linter para Terraform

```bash
# Instalar tflint
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Inicializar (baixar plugins)
tflint --init

# Executar lint
tflint --recursive
```

---

## 16. Integração com CI/CD

### 16.1 Fluxo recomendado em pipelines

```
Pull Request aberto
        │
        ▼
terraform fmt -check    ──► Falha se o código não estiver formatado
        │
        ▼
terraform validate      ──► Falha se o código tiver erros de sintaxe
        │
        ▼
tflint                  ──► Falha se houver problemas de lint
        │
        ▼
checkov                 ──► Falha se houver vulnerabilidades de segurança
        │
        ▼
terraform plan          ──► Gera e publica o plano como comentário no PR
        │
        ▼
Revisão humana do plano
        │
        ▼ (merge para main)
terraform apply         ──► Aplica o plano aprovado
        │
        ▼
Infraestrutura atualizada
```

### 16.2 GitHub Actions — Pipeline completo

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  TF_VERSION: "1.9.5"
  AWS_REGION: "us-east-1"

permissions:
  id-token: write   # OIDC para autenticação sem credenciais estáticas
  contents: read
  pull-requests: write

jobs:
  validate:
    name: Validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-terraform
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate

      - name: TFLint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest

      - run: tflint --init && tflint --recursive

  plan:
    name: Plan
    runs-on: ubuntu-latest
    needs: validate
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-terraform
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        id: plan
        run: terraform plan -no-color -out=tfplan 2>&1
        continue-on-error: true

      - name: Comentar plano no PR
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Plan 📋
            \`\`\`
            ${{ steps.plan.outputs.stdout }}
            \`\`\`
            *Executado por: @${{ github.actor }}*`;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })

  apply:
    name: Apply
    runs-on: ubuntu-latest
    needs: validate
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production  # exige aprovação manual no GitHub
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-terraform
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Apply
        run: terraform apply -auto-approve
```

### 16.3 Autenticação OIDC — Sem credenciais estáticas

Em vez de armazenar `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY` como secrets no GitHub, use OIDC para que o GitHub Actions assuma um role IAM diretamente:

```hcl
# Configurar provider OIDC do GitHub na conta AWS
resource "aws_iam_openid_connect_provider" "github" {
  url = "https://token.actions.githubusercontent.com"

  client_id_list = ["sts.amazonaws.com"]

  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

# Role que o GitHub Actions pode assumir
resource "aws_iam_role" "github_actions_terraform" {
  name = "github-actions-terraform"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = aws_iam_openid_connect_provider.github.arn
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
        }
        StringLike = {
          # Restringir ao repositório específico
          "token.actions.githubusercontent.com:sub" = "repo:minha-org/infra:*"
        }
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "github_actions_terraform" {
  role       = aws_iam_role.github_actions_terraform.name
  policy_arn = "arn:aws:iam::aws:policy/PowerUserAccess"
}
```

---

## 17. Boas Práticas e Organização de Projetos

### 17.1 Estrutura de arquivos recomendada

```
minha-infra/
├── README.md
├── .gitignore
├── .terraform-version          # Versão do Terraform (para tfenv)
│
├── modules/                    # Módulos reutilizáveis internos
│   ├── compute/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── versions.tf
│   ├── network/
│   │   └── ...
│   └── database/
│       └── ...
│
├── environments/               # Configuração por ambiente
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── versions.tf         # inclui configuração do backend
│   │   └── terraform.tfvars
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
│
└── bootstrap/                  # Infraestrutura do state (executado uma vez)
    ├── main.tf
    └── outputs.tf
```

### 17.2 Arquivo .gitignore

```gitignore
# Diretório de trabalho do Terraform
.terraform/

# State local (NUNCA comitar)
*.tfstate
*.tfstate.*
*.tfstate.backup

# Planos salvos
tfplan
*.tfplan

# Arquivos de variáveis com dados sensíveis
*.tfvars
!example.tfvars   # manter o exemplo sem dados reais

# Logs
crash.log
crash.*.log

# Override files (uso local apenas)
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Arquivo de lock de providers (commitar este!)
# .terraform.lock.hcl  ← NÃO adicionar aqui
```

> **Importante:** o arquivo `.terraform.lock.hcl` **deve ser commitado**. Ele garante que todos usem as mesmas versões dos providers.

### 17.3 Convenções de nomenclatura

```hcl
# Recursos: snake_case, descritivo
resource "aws_instance" "app_server" { }        # bom
resource "aws_instance" "AppServer" { }         # ruim
resource "aws_instance" "server1" { }           # ruim (não descritivo)

# Variáveis: snake_case
variable "instance_type" { }                    # bom
variable "instanceType" { }                     # ruim

# Outputs: descrevem o que exportam
output "vpc_id" { }                             # bom
output "resultado" { }                          # ruim

# Módulos: descritivos, sem prefixo terraform_
module "network" { }                            # bom
module "terraform_network" { }                  # ruim

# Nomes de recursos na nuvem: usar locals com prefixo de ambiente
locals {
  name_prefix = "${var.project}-${var.environment}"
}
resource "aws_s3_bucket" "assets" {
  bucket = "${local.name_prefix}-assets"  # "minha-app-prod-assets"
}
```

### 17.4 Segurança

```hcl
# NUNCA hardcode credenciais
# Ruim:
resource "aws_db_instance" "postgres" {
  password = "MinhaSenh@123"   # ← NUNCA faça isso
}

# Bom: variável marcada como sensitive + AWS Secrets Manager
variable "db_password" {
  type      = string
  sensitive = true
}

data "aws_secretsmanager_secret_version" "db" {
  secret_id = "minha-app/prod/db"
}

resource "aws_db_instance" "postgres" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db.secret_string)["password"]
}
```

### 17.5 Checklist de código Terraform

```
Organização
□ Arquivos separados por responsabilidade (main.tf, variables.tf, outputs.tf)
□ Módulos para lógica reutilizada mais de uma vez
□ .gitignore configurado corretamente
□ .terraform.lock.hcl commitado

Qualidade
□ terraform fmt executado (ou verificado em CI)
□ terraform validate passando
□ tflint sem erros
□ Variáveis com description e validation quando necessário
□ Outputs com description

Segurança
□ Nenhuma credencial ou secret no código
□ Variáveis sensíveis marcadas com sensitive = true
□ State armazenado remotamente com criptografia
□ S3 bucket de state com bloqueio de acesso público
□ Locking de state habilitado (DynamoDB)
□ Checkov sem falhas críticas

State
□ Remote backend configurado
□ State locking habilitado
□ Versionamento do bucket S3 habilitado
□ Acesso ao state restrito por IAM
```

---

## 18. Estudos de Caso

### 18.1 Infraestrutura de uma API REST na AWS

**Diagrama da infraestrutura:**

```
Internet
    │
    ▼
┌──────────────────────────────────────────────────────────────────────┐
│ AWS                                                                  │
│                                                                      │
│   Route 53 (DNS)                                                     │
│       │                                                              │
│       ▼                                                              │
│   ACM Certificate                                                    │
│       │                                                              │
│   ┌───┴───────────────────────────────────────────┐                 │
│   │ VPC (10.0.0.0/16)                             │                 │
│   │                                               │                 │
│   │  ┌─────────────────────────────────────────┐ │                 │
│   │  │ Public Subnets (us-east-1a/1b/1c)       │ │                 │
│   │  │                                         │ │                 │
│   │  │  Internet Gateway ── NAT Gateway (x3)   │ │                 │
│   │  │  Application Load Balancer (HTTPS/443)  │ │                 │
│   │  └──────────────┬──────────────────────────┘ │                 │
│   │                 │                             │                 │
│   │  ┌──────────────▼──────────────────────────┐ │                 │
│   │  │ Private Subnets (us-east-1a/1b/1c)      │ │                 │
│   │  │                                         │ │                 │
│   │  │  Auto Scaling Group                     │ │                 │
│   │  │  ┌──────────┐ ┌──────────┐ ┌────────┐  │ │                 │
│   │  │  │ EC2 App  │ │ EC2 App  │ │EC2 App │  │ │                 │
│   │  │  │ (t3.sm.) │ │ (t3.sm.) │ │(t3.sm.)│  │ │                 │
│   │  │  └──────────┘ └──────────┘ └────────┘  │ │                 │
│   │  └──────────────┬──────────────────────────┘ │                 │
│   │                 │                             │                 │
│   │  ┌──────────────▼──────────────────────────┐ │                 │
│   │  │ Database Subnets                        │ │                 │
│   │  │  RDS PostgreSQL (Multi-AZ)              │ │                 │
│   │  │  ElastiCache Redis (cluster)            │ │                 │
│   │  └─────────────────────────────────────────┘ │                 │
│   └───────────────────────────────────────────────┘                 │
│                                                                      │
│   S3 (assets, logs)    CloudWatch (métricas, alarmes)               │
│   Secrets Manager      IAM Roles                                    │
└──────────────────────────────────────────────────────────────────────┘
```

**Estrutura do projeto:**

```
infra-api-rest/
├── environments/prod/
│   ├── versions.tf     # backend S3 + required_providers
│   ├── main.tf         # chamadas aos módulos
│   ├── variables.tf
│   ├── outputs.tf
│   └── terraform.tfvars
└── modules/
    ├── network/        # VPC, subnets, IGW, NAT, route tables
    ├── compute/        # ALB, ASG, launch template, security groups
    ├── database/       # RDS, ElastiCache, subnet groups
    └── dns/            # Route 53, ACM
```

**`environments/prod/main.tf`:**

```hcl
locals {
  project     = "minha-api"
  environment = "prod"
  region      = "us-east-1"
}

module "network" {
  source = "../../modules/network"

  project     = local.project
  environment = local.environment
  vpc_cidr    = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24", "10.0.13.0/24"]
  database_subnets = ["10.0.21.0/24", "10.0.22.0/24", "10.0.23.0/24"]
}

module "database" {
  source = "../../modules/database"

  project     = local.project
  environment = local.environment

  subnet_ids         = module.network.database_subnet_ids
  vpc_id             = module.network.vpc_id
  app_security_group = module.compute.app_security_group_id

  instance_class    = "db.r6g.large"
  allocated_storage = 100
  multi_az          = true
}

module "compute" {
  source = "../../modules/compute"

  project     = local.project
  environment = local.environment

  vpc_id             = module.network.vpc_id
  public_subnet_ids  = module.network.public_subnet_ids
  private_subnet_ids = module.network.private_subnet_ids

  instance_type = "t3.small"
  min_size      = 3
  max_size      = 10
  desired_size  = 3

  certificate_arn = module.dns.certificate_arn
  db_endpoint     = module.database.endpoint
}

module "dns" {
  source = "../../modules/dns"

  domain      = "api.minha-app.com.br"
  alb_dns_name = module.compute.alb_dns_name
  alb_zone_id  = module.compute.alb_zone_id
}
```

---

### 18.2 Ambiente Multi-conta com Organizations

Em organizações maiores, cada ambiente (dev, staging, prod) fica em uma conta AWS separada para isolamento completo.

```
AWS Organizations
├── Root (gerenciamento)
│   └── SCPs (Service Control Policies)
│
├── Unidade Organizacional: Workloads
│   ├── Conta: minha-app-dev   (123456789001)
│   ├── Conta: minha-app-staging (123456789002)
│   └── Conta: minha-app-prod  (123456789003)
│
├── Unidade Organizacional: Shared Services
│   └── Conta: shared-services  (123456789004)
│       ├── ECR (Docker images)
│       ├── Route 53 (DNS)
│       └── AWS SSO
│
└── Unidade Organizacional: Security
    └── Conta: security-audit   (123456789005)
        ├── CloudTrail (todos os logs)
        ├── AWS Config
        └── GuardDuty
```

**Configuração multi-conta com provider `alias`:**

```hcl
# providers.tf
provider "aws" {
  alias  = "dev"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::123456789001:role/OrganizationAccountAccessRole"
  }
}

provider "aws" {
  alias  = "prod"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::123456789003:role/OrganizationAccountAccessRole"
  }
}

# Criar recursos em contas separadas
module "rede_dev" {
  source    = "./modules/network"
  providers = { aws = aws.dev }
  # ...
}

module "rede_prod" {
  source    = "./modules/network"
  providers = { aws = aws.prod }
  # ...
}
```

---

### 18.3 Pipeline Completo com GitHub Actions

**Diagrama do fluxo:**

```
Developer
    │
    │ git push (feature branch)
    ▼
GitHub
    │
    ├── terraform fmt --check   (falha imediata se mal formatado)
    ├── terraform validate       (falha se há erros de sintaxe)
    ├── tflint                   (falha se há problemas de qualidade)
    ├── checkov                  (falha se há vulnerabilidades)
    └── terraform plan           (publica plano como comentário no PR)
                │
                ▼
        Revisão do PR
        (humano revisa o plano
         antes de aprovar o merge)
                │
                │ merge to main
                ▼
        terraform apply ──► Infraestrutura atualizada
                │
                ▼
        Notificação (Slack/Teams)
```

**Separação entre plan e apply:**

```yaml
# Estratégia recomendada:
# - plan roda em QUALQUER push (inclusive PRs)
# - apply roda APENAS após merge para main
# - apply exige environment "production" (aprovação manual no GitHub)

on:
  pull_request:
    branches: [main]     # plan
  push:
    branches: [main]     # apply

jobs:
  plan:
    if: github.event_name == 'pull_request'
    # ...

  apply:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment: production   # requer aprovador configurado no GitHub
    # ...
```

**Notificação no Slack após apply:**

```yaml
- name: Notificar Slack
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "${{ job.status == 'success' && '✅' || '❌' }} Terraform Apply - ${{ github.repository }}",
        "blocks": [{
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": "*${{ job.status == 'success' && 'Sucesso' || 'Falha' }}* no deploy de infraestrutura\n*Ambiente:* prod\n*Commit:* <${{ github.server_url }}/${{ github.repository }}/commit/${{ github.sha }}|${{ github.sha }}>\n*Executado por:* ${{ github.actor }}"
          }
        }]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

*Documentação produzida e revisada com apoio de agentes de IA (Claude Code).*
