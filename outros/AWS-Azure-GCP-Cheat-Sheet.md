# Cloud Services Cheat Sheet — AWS · Azure · GCP

Referência rápida dos principais serviços das três grandes plataformas de nuvem e seus comandos CLI.

---

## Sumário

- [Configuração das CLIs](#configuração-das-clis)
- [Computação](#computação)
- [Containers e Orquestração](#containers-e-orquestração)
  - [kubectl — Comandos Comuns](#kubectl--comandos-comuns)
- [Serverless](#serverless)
- [Armazenamento de Objetos](#armazenamento-de-objetos)
- [Banco de Dados Relacional](#banco-de-dados-relacional)
- [Banco de Dados NoSQL](#banco-de-dados-nosql)
- [Cache](#cache)
- [Mensageria e Filas](#mensageria-e-filas)
- [Rede e DNS](#rede-e-dns)
- [Load Balancer e CDN](#load-balancer-e-cdn)
- [Segurança e Gerenciamento de Segredos](#segurança-e-gerenciamento-de-segredos)
- [Autenticação e Identidade](#autenticação-e-identidade)
- [Monitoramento e Observabilidade](#monitoramento-e-observabilidade)
- [CI/CD e DevOps](#cicd-e-devops)
- [Infraestrutura como Código](#infraestrutura-como-código)
- [IA e Machine Learning](#ia-e-machine-learning)
- [Tabela Comparativa Geral](#tabela-comparativa-geral)

---

## Configuração das CLIs

### AWS CLI

```bash
# Instalação
pip install awscli
# ou via script oficial: https://aws.amazon.com/cli/

# Configuração inicial
aws configure
# AWS Access Key ID: <key>
# AWS Secret Access Key: <secret>
# Default region: us-east-1
# Default output format: json

# Usar perfil específico
aws configure --profile meu-perfil
aws s3 ls --profile meu-perfil

# Verificar identidade configurada
aws sts get-caller-identity

# Variáveis de ambiente alternativas
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=us-east-1
```

### Azure CLI

```bash
# Instalação (Linux/macOS)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
# Windows: winget install Microsoft.AzureCLI

# Login interativo
az login

# Login com service principal
az login --service-principal -u <appId> -p <password> --tenant <tenantId>

# Definir subscription padrão
az account set --subscription "<nome-ou-id>"

# Listar subscriptions disponíveis
az account list --output table

# Verificar identidade atual
az account show
```

### GCP CLI (gcloud)

```bash
# Instalação: https://cloud.google.com/sdk/docs/install

# Login interativo
gcloud auth login

# Login com service account
gcloud auth activate-service-account --key-file=chave.json

# Definir projeto padrão
gcloud config set project meu-projeto

# Definir região e zona padrão
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Listar configurações ativas
gcloud config list

# Verificar identidade atual
gcloud auth list
```

---

## Computação

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| VMs | EC2 | Virtual Machines | Compute Engine |
| Imagens de VM | AMI | Managed Images / Azure Marketplace | Custom Images |
| Auto Scaling | Auto Scaling Groups | Virtual Machine Scale Sets (VMSS) | Managed Instance Groups |
| Spot/Preemptível | Spot Instances | Spot VMs | Preemptible/Spot VMs |

### AWS EC2

```bash
# Listar instâncias
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,PublicIpAddress]' --output table

# Iniciar instância
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Parar instância
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Criar instância
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name minha-chave \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --count 1

# Terminar (deletar) instância
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Listar tipos de instância disponíveis
aws ec2 describe-instance-types --filters "Name=instance-type,Values=t3.*" --output table

# Criar AMI (snapshot da instância)
aws ec2 create-image --instance-id i-1234567890abcdef0 --name "minha-ami" --no-reboot

# Alocar IP elástico
aws ec2 allocate-address --domain vpc

# Associar IP elástico à instância
aws ec2 associate-address --instance-id i-1234567890abcdef0 --allocation-id eipalloc-0123456789abcdef0
```

### Azure Virtual Machines

```bash
# Listar VMs
az vm list --output table

# Criar VM
az vm create \
  --resource-group meu-rg \
  --name minha-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

# Iniciar VM
az vm start --resource-group meu-rg --name minha-vm

# Parar VM (desalocar)
az vm deallocate --resource-group meu-rg --name minha-vm

# Deletar VM
az vm delete --resource-group meu-rg --name minha-vm --yes

# Listar tamanhos disponíveis
az vm list-sizes --location eastus --output table

# Ver status da VM
az vm get-instance-view --resource-group meu-rg --name minha-vm --query instanceView.statuses

# Redimensionar VM
az vm resize --resource-group meu-rg --name minha-vm --size Standard_B2s
```

### GCP Compute Engine

```bash
# Listar instâncias
gcloud compute instances list

# Criar instância
gcloud compute instances create minha-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB

# Iniciar instância
gcloud compute instances start minha-vm --zone=us-central1-a

# Parar instância
gcloud compute instances stop minha-vm --zone=us-central1-a

# Deletar instância
gcloud compute instances delete minha-vm --zone=us-central1-a

# SSH na instância
gcloud compute ssh minha-vm --zone=us-central1-a

# Listar machine types
gcloud compute machine-types list --filter="zone:us-central1-a" --sort-by=name

# Criar snapshot do disco
gcloud compute disks snapshot meu-disco --zone=us-central1-a --snapshot-names=meu-snapshot
```

---

## Containers e Orquestração

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Registro de imagens | ECR | Azure Container Registry (ACR) | Artifact Registry / GCR |
| Kubernetes gerenciado | EKS | AKS | GKE |
| Kubernetes self-managed | EC2 + kubeadm | VMs + kubeadm | GCE + kubeadm |
| Nodes spot/preemptíveis | EKS Spot Managed NG | AKS Spot Node Pool | GKE Spot Node Pool |
| Ingress controller | AWS Load Balancer Controller | AGIC (App Gateway) | GKE Ingress / NGINX |
| Service Mesh | App Mesh | Open Service Mesh / Istio | Cloud Service Mesh (Istio) |
| Monitoramento k8s | CloudWatch Container Insights | Azure Monitor for Containers | GKE Observability / Cloud Ops |
| Containers sem k8s | ECS (Fargate) | Azure Container Apps | Cloud Run |
| Containers avulsos | ECS (EC2) | Azure Container Instances (ACI) | Cloud Run Jobs |

### AWS ECR + ECS

```bash
# --- ECR ---
# Autenticar Docker no ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Criar repositório
aws ecr create-repository --repository-name minha-app

# Listar repositórios
aws ecr describe-repositories

# Fazer push de imagem
docker build -t minha-app .
docker tag minha-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/minha-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/minha-app:latest

# --- EKS ---
# Criar cluster
aws eks create-cluster \
  --name meu-cluster \
  --kubernetes-version 1.29 \
  --role-arn arn:aws:iam::123456789012:role/eks-role \
  --resources-vpc-config subnetIds=subnet-xxx,subnet-yyy,securityGroupIds=sg-xxx

# Configurar kubectl para o cluster
aws eks update-kubeconfig --region us-east-1 --name meu-cluster

# Listar clusters
aws eks list-clusters

# Adicionar node group
aws eks create-nodegroup \
  --cluster-name meu-cluster \
  --nodegroup-name meu-ng \
  --node-role arn:aws:iam::123456789012:role/eks-node-role \
  --subnets subnet-xxx subnet-yyy \
  --instance-types t3.medium \
  --scaling-config minSize=1,maxSize=3,desiredSize=2

# --- ECS ---
# Listar clusters ECS
aws ecs list-clusters

# Listar serviços em um cluster
aws ecs list-services --cluster meu-cluster

# Descrever serviço
aws ecs describe-services --cluster meu-cluster --services meu-servico

# Atualizar imagem de um serviço (forçar novo deploy)
aws ecs update-service --cluster meu-cluster --service meu-servico --force-new-deployment
```

### Azure ACR + AKS

```bash
# --- ACR ---
# Criar registro
az acr create --resource-group meu-rg --name meuregistry --sku Basic

# Login no registro
az acr login --name meuregistry

# Listar imagens
az acr repository list --name meuregistry --output table

# Build e push direto no ACR
az acr build --registry meuregistry --image minha-app:v1 .

# --- AKS ---
# Criar cluster
az aks create \
  --resource-group meu-rg \
  --name meu-cluster \
  --node-count 2 \
  --node-vm-size Standard_B2s \
  --generate-ssh-keys \
  --attach-acr meuregistry

# Configurar kubectl
az aks get-credentials --resource-group meu-rg --name meu-cluster

# Listar clusters
az aks list --output table

# Escalar node pool
az aks scale --resource-group meu-rg --name meu-cluster --node-count 3

# Atualizar versão do Kubernetes
az aks upgrade --resource-group meu-rg --name meu-cluster --kubernetes-version 1.29

# Deletar cluster
az aks delete --resource-group meu-rg --name meu-cluster --yes
```

### GCP Artifact Registry + GKE

```bash
# --- Artifact Registry ---
# Criar repositório
gcloud artifacts repositories create meu-repo \
  --repository-format=docker \
  --location=us-central1

# Configurar Docker para Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev

# Push de imagem
docker tag minha-app us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:v1
docker push us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:v1

# Listar imagens
gcloud artifacts docker images list us-central1-docker.pkg.dev/meu-projeto/meu-repo

# --- GKE ---
# Criar cluster (Standard)
gcloud container clusters create meu-cluster \
  --zone us-central1-a \
  --num-nodes 2 \
  --machine-type e2-standard-2

# Criar cluster (Autopilot — gerenciado)
gcloud container clusters create-auto meu-cluster-auto --region us-central1

# Configurar kubectl
gcloud container clusters get-credentials meu-cluster --zone us-central1-a

# Listar clusters
gcloud container clusters list

# Redimensionar cluster
gcloud container clusters resize meu-cluster --zone us-central1-a --num-nodes 3

# Atualizar versão do cluster
gcloud container clusters upgrade meu-cluster --zone us-central1-a --master

# Deletar cluster
gcloud container clusters delete meu-cluster --zone us-central1-a
```

### kubectl — Comandos Comuns

> Após configurar o contexto com `aws eks update-kubeconfig`, `az aks get-credentials` ou `gcloud container clusters get-credentials`, os comandos `kubectl` abaixo funcionam igual em qualquer cluster.

#### Contextos e Clusters

```bash
# Listar contextos disponíveis
kubectl config get-contexts

# Trocar de contexto (trocar de cluster)
kubectl config use-context meu-cluster-producao

# Ver contexto atual
kubectl config current-context

# Ver configuração completa do kubeconfig
kubectl config view --minify

# Renomear contexto
kubectl config rename-context nome-antigo nome-novo

# Deletar contexto
kubectl config delete-context meu-cluster-antigo
```

#### Namespaces

```bash
# Listar namespaces
kubectl get namespaces

# Criar namespace
kubectl create namespace meu-namespace

# Definir namespace padrão no contexto atual
kubectl config set-context --current --namespace=meu-namespace

# Deletar namespace (remove TODOS os recursos dentro dele)
kubectl delete namespace meu-namespace
```

#### Pods

```bash
# Listar pods (namespace atual)
kubectl get pods

# Listar pods de todos os namespaces
kubectl get pods -A

# Listar pods com mais detalhes (IP, node, status)
kubectl get pods -o wide

# Descrever pod (eventos, condições, volumes)
kubectl describe pod meu-pod

# Ver logs de um pod
kubectl logs meu-pod

# Logs em tempo real (follow)
kubectl logs -f meu-pod

# Logs de container específico em pod multi-container
kubectl logs meu-pod -c nome-do-container

# Logs das últimas 100 linhas
kubectl logs meu-pod --tail=100

# Executar comando dentro do pod
kubectl exec -it meu-pod -- /bin/bash
kubectl exec -it meu-pod -- sh

# Copiar arquivo para/do pod
kubectl cp meu-pod:/caminho/arquivo.txt ./arquivo.txt
kubectl cp ./arquivo.txt meu-pod:/caminho/arquivo.txt

# Deletar pod (será recriado se gerenciado por Deployment)
kubectl delete pod meu-pod

# Ver eventos do cluster (debug)
kubectl get events --sort-by='.lastTimestamp'
```

#### Deployments

```bash
# Criar deployment
kubectl create deployment minha-app \
  --image=nginx:1.25 \
  --replicas=3

# Listar deployments
kubectl get deployments

# Descrever deployment
kubectl describe deployment minha-app

# Escalar deployment
kubectl scale deployment minha-app --replicas=5

# Atualizar imagem (rolling update)
kubectl set image deployment/minha-app minha-app=nginx:1.26

# Verificar status do rollout
kubectl rollout status deployment/minha-app

# Ver histórico de rollouts
kubectl rollout history deployment/minha-app

# Fazer rollback para versão anterior
kubectl rollout undo deployment/minha-app

# Rollback para revisão específica
kubectl rollout undo deployment/minha-app --to-revision=2

# Pausar/retomar rollout
kubectl rollout pause deployment/minha-app
kubectl rollout resume deployment/minha-app

# Deletar deployment
kubectl delete deployment minha-app

# Aplicar manifesto YAML
kubectl apply -f deployment.yaml

# Deletar por manifesto
kubectl delete -f deployment.yaml
```

#### Services

```bash
# Listar services
kubectl get services

# Expor deployment como service
kubectl expose deployment minha-app --port=80 --target-port=8080 --type=ClusterIP

# Tipos de service:
#   ClusterIP  — acesso apenas interno ao cluster (padrão)
#   NodePort   — expõe em porta estática de cada node (30000–32767)
#   LoadBalancer — provisiona LB externo na cloud (ELB, Azure LB, Cloud LB)

# Criar service LoadBalancer
kubectl expose deployment minha-app \
  --port=80 \
  --target-port=8080 \
  --type=LoadBalancer \
  --name=minha-app-lb

# Ver IP externo do LoadBalancer
kubectl get service minha-app-lb --watch

# Port forward local para pod/service (teste sem expor externamente)
kubectl port-forward service/minha-app 8080:80
kubectl port-forward pod/meu-pod 8080:8080

# Descrever service
kubectl describe service minha-app

# Deletar service
kubectl delete service minha-app
```

#### ConfigMaps e Secrets

```bash
# Criar ConfigMap a partir de valores literais
kubectl create configmap meu-config \
  --from-literal=DB_HOST=localhost \
  --from-literal=DB_PORT=5432

# Criar ConfigMap a partir de arquivo
kubectl create configmap meu-config --from-file=config.properties

# Listar ConfigMaps
kubectl get configmaps

# Ver conteúdo de ConfigMap
kubectl get configmap meu-config -o yaml

# Criar Secret (dados em base64 automaticamente)
kubectl create secret generic meu-secret \
  --from-literal=DB_PASSWORD=senha-secreta \
  --from-literal=API_KEY=minha-api-key

# Criar Secret a partir de arquivo
kubectl create secret generic meu-secret --from-file=.env

# Criar Secret TLS (certificado)
kubectl create secret tls meu-tls \
  --cert=certificado.crt \
  --key=chave-privada.key

# Listar Secrets
kubectl get secrets

# Ver Secret (valores em base64)
kubectl get secret meu-secret -o yaml

# Decodificar valor de Secret
kubectl get secret meu-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode

# Editar ConfigMap/Secret
kubectl edit configmap meu-config
```

#### Ingress

```bash
# Listar Ingress
kubectl get ingress -A

# Descrever Ingress
kubectl describe ingress meu-ingress

# Exemplo de manifesto Ingress (NGINX):
# kubectl apply -f - <<EOF
# apiVersion: networking.k8s.io/v1
# kind: Ingress
# metadata:
#   name: meu-ingress
#   annotations:
#     nginx.ingress.kubernetes.io/rewrite-target: /
# spec:
#   ingressClassName: nginx
#   rules:
#   - host: app.meudominio.com
#     http:
#       paths:
#       - path: /
#         pathType: Prefix
#         backend:
#           service:
#             name: minha-app
#             port:
#               number: 80
# EOF

# Instalar NGINX Ingress Controller via Helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# Instalar cert-manager (TLS automático com Let's Encrypt)
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true
```

#### Autoscaling (HPA / VPA)

```bash
# Criar HPA (Horizontal Pod Autoscaler) por CPU
kubectl autoscale deployment minha-app \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# Listar HPAs
kubectl get hpa

# Descrever HPA (ver métricas atuais)
kubectl describe hpa minha-app

# HPA por métrica customizada (manifesto)
# kubectl apply -f hpa-custom.yaml

# Deletar HPA
kubectl delete hpa minha-app
```

#### Recursos e Quotas

```bash
# Ver consumo de recursos por pod
kubectl top pods
kubectl top pods -A

# Ver consumo de recursos por node
kubectl top nodes

# Ver resource quotas de namespace
kubectl get resourcequota -n meu-namespace
kubectl describe resourcequota -n meu-namespace

# Ver LimitRange de namespace
kubectl get limitrange -n meu-namespace

# Ver capacidade e alocação dos nodes
kubectl describe nodes | grep -A 5 "Allocated resources"
```

#### Troubleshooting

```bash
# Pod travado em Pending — ver por quê
kubectl describe pod meu-pod | grep -A 10 Events

# Pod em CrashLoopBackOff — ver logs do crash anterior
kubectl logs meu-pod --previous

# Ver todos os eventos de erro no cluster
kubectl get events -A --field-selector type=Warning --sort-by='.lastTimestamp'

# Debug com pod temporário (imagem com ferramentas de rede)
kubectl run debug-pod --image=nicolaka/netshoot -it --rm -- /bin/bash

# Checar conectividade entre pods (dentro do cluster)
kubectl run curl-test --image=curlimages/curl -it --rm -- curl http://minha-app:80

# Checar DNS interno do cluster
kubectl run dns-test --image=busybox -it --rm -- nslookup minha-app.default.svc.cluster.local

# Forçar recriação de pods de um deployment
kubectl rollout restart deployment/minha-app

# Cordon node (impede novo agendamento, mantém pods existentes)
kubectl cordon meu-node

# Drain node (remove pods para manutenção)
kubectl drain meu-node --ignore-daemonsets --delete-emptydir-data

# Uncordon node (reativa agendamento)
kubectl uncordon meu-node
```

#### Helm — Gerenciador de Pacotes k8s

```bash
# Adicionar repositório
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Buscar charts disponíveis
helm search repo bitnami/postgresql

# Instalar chart
helm install meu-postgres bitnami/postgresql \
  --namespace banco \
  --create-namespace \
  --set auth.password=senha-segura \
  --set primary.persistence.size=10Gi

# Listar releases instalados
helm list -A

# Ver status de release
helm status meu-postgres -n banco

# Atualizar release (upgrade)
helm upgrade meu-postgres bitnami/postgresql \
  --namespace banco \
  --set auth.password=nova-senha

# Ver histórico de versões
helm history meu-postgres -n banco

# Rollback de release
helm rollback meu-postgres 1 -n banco

# Desinstalar release
helm uninstall meu-postgres -n banco

# Gerar manifesto sem instalar (dry-run)
helm install meu-postgres bitnami/postgresql --dry-run --debug
```

---

## Serverless

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Functions (FaaS) | Lambda | Azure Functions | Cloud Functions |
| API Gateway | API Gateway | API Management (APIM) | API Gateway / Cloud Endpoints |
| Orquestração | Step Functions | Logic Apps / Durable Functions | Workflows |
| Eventos | EventBridge | Event Grid | Eventarc |

### AWS Lambda

```bash
# Criar função Lambda (zip)
zip function.zip index.js
aws lambda create-function \
  --function-name minha-funcao \
  --zip-file fileb://function.zip \
  --handler index.handler \
  --runtime nodejs20.x \
  --role arn:aws:iam::123456789012:role/lambda-role

# Listar funções
aws lambda list-functions --output table

# Invocar função
aws lambda invoke \
  --function-name minha-funcao \
  --payload '{"key":"value"}' \
  output.json

# Atualizar código da função
zip function.zip index.js
aws lambda update-function-code \
  --function-name minha-funcao \
  --zip-file fileb://function.zip

# Atualizar variáveis de ambiente
aws lambda update-function-configuration \
  --function-name minha-funcao \
  --environment 'Variables={DB_URL=postgres://...,APP_ENV=production}'

# Ver logs (CloudWatch)
aws logs tail /aws/lambda/minha-funcao --follow

# Deletar função
aws lambda delete-function --function-name minha-funcao
```

### Azure Functions

```bash
# Criar Function App
az functionapp create \
  --resource-group meu-rg \
  --name minha-funcao-app \
  --storage-account minhastorageaccount \
  --runtime node \
  --runtime-version 20 \
  --functions-version 4 \
  --consumption-plan-location eastus

# Listar Function Apps
az functionapp list --output table

# Publicar função (requer Azure Functions Core Tools)
func azure functionapp publish minha-funcao-app

# Ver configurações (env vars)
az functionapp config appsettings list --name minha-funcao-app --resource-group meu-rg

# Definir variável de ambiente
az functionapp config appsettings set \
  --name minha-funcao-app \
  --resource-group meu-rg \
  --settings "DB_URL=postgres://..."

# Iniciar/Parar Function App
az functionapp start --name minha-funcao-app --resource-group meu-rg
az functionapp stop --name minha-funcao-app --resource-group meu-rg

# Ver logs em tempo real
az webapp log tail --name minha-funcao-app --resource-group meu-rg
```

### GCP Cloud Functions / Cloud Run

```bash
# Criar Cloud Function (Gen 2)
gcloud functions deploy minha-funcao \
  --gen2 \
  --region=us-central1 \
  --runtime=nodejs20 \
  --trigger-http \
  --allow-unauthenticated \
  --source=.

# Listar funções
gcloud functions list

# Invocar função
gcloud functions call minha-funcao --data '{"key":"value"}' --region=us-central1

# Ver logs
gcloud functions logs read minha-funcao --region=us-central1 --limit=50

# Deletar função
gcloud functions delete minha-funcao --region=us-central1

# --- Cloud Run (container serverless) ---
# Deploy de imagem
gcloud run deploy minha-app \
  --image us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:v1 \
  --region us-central1 \
  --allow-unauthenticated \
  --port 8080 \
  --memory 512Mi \
  --cpu 1

# Listar serviços Cloud Run
gcloud run services list --region us-central1

# Ver URL do serviço
gcloud run services describe minha-app --region us-central1 --format='value(status.url)'

# Definir variáveis de ambiente
gcloud run services update minha-app \
  --region us-central1 \
  --set-env-vars DB_URL=postgres://...,APP_ENV=production
```

---

## Armazenamento de Objetos

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Object Storage | S3 | Azure Blob Storage | Cloud Storage (GCS) |
| CDN integrada | CloudFront | Azure CDN | Cloud CDN |
| Arquivo frio | S3 Glacier | Azure Archive Storage | Cloud Storage Archive |

### AWS S3

```bash
# Criar bucket
aws s3 mb s3://meu-bucket --region us-east-1

# Listar buckets
aws s3 ls

# Listar objetos no bucket
aws s3 ls s3://meu-bucket/ --recursive

# Upload de arquivo
aws s3 cp arquivo.txt s3://meu-bucket/pasta/arquivo.txt

# Upload de diretório
aws s3 cp ./minha-pasta s3://meu-bucket/destino/ --recursive

# Download de arquivo
aws s3 cp s3://meu-bucket/pasta/arquivo.txt ./arquivo.txt

# Sincronizar diretório local com bucket
aws s3 sync ./dist s3://meu-bucket --delete

# Gerar URL pré-assinada (acesso temporário)
aws s3 presign s3://meu-bucket/arquivo.txt --expires-in 3600

# Habilitar versionamento
aws s3api put-bucket-versioning \
  --bucket meu-bucket \
  --versioning-configuration Status=Enabled

# Deletar objeto
aws s3 rm s3://meu-bucket/pasta/arquivo.txt

# Deletar bucket (deve estar vazio)
aws s3 rb s3://meu-bucket

# Deletar bucket com conteúdo
aws s3 rb s3://meu-bucket --force

# Configurar bucket como site estático
aws s3 website s3://meu-bucket \
  --index-document index.html \
  --error-document error.html

# Tornar objeto público
aws s3api put-object-acl --bucket meu-bucket --key arquivo.txt --acl public-read
```

### Azure Blob Storage

```bash
# Criar storage account
az storage account create \
  --name minhastorageaccount \
  --resource-group meu-rg \
  --location eastus \
  --sku Standard_LRS

# Criar container (equivalente ao bucket/pasta raiz)
az storage container create \
  --name meu-container \
  --account-name minhastorageaccount \
  --public-access blob

# Listar containers
az storage container list --account-name minhastorageaccount --output table

# Upload de arquivo
az storage blob upload \
  --account-name minhastorageaccount \
  --container-name meu-container \
  --name arquivo.txt \
  --file ./arquivo.txt

# Upload de diretório
az storage blob upload-batch \
  --account-name minhastorageaccount \
  --destination meu-container \
  --source ./minha-pasta

# Listar blobs
az storage blob list \
  --account-name minhastorageaccount \
  --container-name meu-container \
  --output table

# Download de blob
az storage blob download \
  --account-name minhastorageaccount \
  --container-name meu-container \
  --name arquivo.txt \
  --file ./arquivo-baixado.txt

# Gerar SAS token (URL temporária)
az storage blob generate-sas \
  --account-name minhastorageaccount \
  --container-name meu-container \
  --name arquivo.txt \
  --permissions r \
  --expiry 2024-12-31

# Deletar blob
az storage blob delete \
  --account-name minhastorageaccount \
  --container-name meu-container \
  --name arquivo.txt
```

### GCP Cloud Storage

```bash
# Criar bucket
gcloud storage buckets create gs://meu-bucket --location=US-CENTRAL1

# Listar buckets
gcloud storage buckets list

# Listar objetos
gcloud storage ls gs://meu-bucket/

# Upload de arquivo
gcloud storage cp arquivo.txt gs://meu-bucket/pasta/arquivo.txt

# Upload de diretório
gcloud storage cp -r ./minha-pasta gs://meu-bucket/destino/

# Download de arquivo
gcloud storage cp gs://meu-bucket/pasta/arquivo.txt ./arquivo.txt

# Sincronizar
gcloud storage rsync -r ./dist gs://meu-bucket --delete-unmatched-destination-objects

# Tornar objeto público
gcloud storage objects update gs://meu-bucket/arquivo.txt --add-acl-grant=entity=allUsers,role=READER

# Gerar URL assinada
gcloud storage sign-url gs://meu-bucket/arquivo.txt \
  --duration=1h \
  --private-key-file=chave-service-account.json

# Deletar objeto
gcloud storage rm gs://meu-bucket/pasta/arquivo.txt

# Deletar bucket com conteúdo
gcloud storage rm -r gs://meu-bucket
```

---

## Banco de Dados Relacional

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| PostgreSQL gerenciado | RDS (PostgreSQL) | Azure Database for PostgreSQL | Cloud SQL (PostgreSQL) |
| MySQL gerenciado | RDS (MySQL) | Azure Database for MySQL | Cloud SQL (MySQL) |
| SQL Server gerenciado | RDS (SQL Server) | Azure SQL Database | Cloud SQL (SQL Server) |
| Alta disponibilidade | Aurora (MySQL/PostgreSQL) | Azure SQL Hyperscale | AlloyDB (PostgreSQL) |

### AWS RDS

```bash
# Criar instância RDS PostgreSQL
aws rds create-db-instance \
  --db-instance-identifier meu-banco \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password senha-segura \
  --allocated-storage 20 \
  --no-publicly-accessible

# Listar instâncias RDS
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Endpoint.Address]' --output table

# Criar snapshot manual
aws rds create-db-snapshot \
  --db-instance-identifier meu-banco \
  --db-snapshot-identifier meu-banco-snapshot-$(date +%Y%m%d)

# Listar snapshots
aws rds describe-db-snapshots --db-instance-identifier meu-banco --output table

# Restaurar a partir de snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier meu-banco-restaurado \
  --db-snapshot-identifier meu-banco-snapshot-20240101

# Modificar instância (ex: aumentar storage)
aws rds modify-db-instance \
  --db-instance-identifier meu-banco \
  --allocated-storage 50 \
  --apply-immediately

# Deletar instância (sem snapshot final)
aws rds delete-db-instance \
  --db-instance-identifier meu-banco \
  --skip-final-snapshot
```

### Azure Database for PostgreSQL

```bash
# Criar servidor PostgreSQL (Flexible Server)
az postgres flexible-server create \
  --resource-group meu-rg \
  --name meu-servidor-pg \
  --location eastus \
  --admin-user adminuser \
  --admin-password SenhaSegura123! \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --version 15

# Listar servidores
az postgres flexible-server list --output table

# Criar banco de dados
az postgres flexible-server db create \
  --resource-group meu-rg \
  --server-name meu-servidor-pg \
  --database-name meu-banco

# Adicionar regra de firewall (ex: IP atual)
az postgres flexible-server firewall-rule create \
  --resource-group meu-rg \
  --name meu-servidor-pg \
  --rule-name allow-my-ip \
  --start-ip-address <meu-ip> \
  --end-ip-address <meu-ip>

# Criar backup manual / restore
az postgres flexible-server backup create \
  --resource-group meu-rg \
  --name meu-servidor-pg \
  --backup-name meu-backup

# Conectar via psql
az postgres flexible-server connect \
  --name meu-servidor-pg \
  --resource-group meu-rg \
  --admin-user adminuser \
  --database-name meu-banco
```

### GCP Cloud SQL

```bash
# Criar instância PostgreSQL
gcloud sql instances create meu-banco \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1 \
  --no-assign-ip \
  --network=default

# Listar instâncias
gcloud sql instances list

# Definir senha do usuário root/postgres
gcloud sql users set-password postgres \
  --instance=meu-banco \
  --password=senha-segura

# Criar banco de dados
gcloud sql databases create meu-banco-db --instance=meu-banco

# Criar backup manual
gcloud sql backups create --instance=meu-banco

# Listar backups
gcloud sql backups list --instance=meu-banco

# Restaurar backup
gcloud sql backups restore <backup-id> --restore-instance=meu-banco

# Conectar via Cloud SQL Proxy (recomendado)
# Baixar proxy: https://cloud.google.com/sql/docs/postgres/connect-auth-proxy
./cloud-sql-proxy meu-projeto:us-central1:meu-banco &
psql -h 127.0.0.1 -U postgres -d meu-banco-db

# Habilitar IP público e adicionar IP autorizado
gcloud sql instances patch meu-banco \
  --authorized-networks=<meu-ip>/32
```

---

## Banco de Dados NoSQL

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Chave-valor / Document | DynamoDB | Cosmos DB | Firestore / Datastore |
| Wide-column | Keyspaces (Cassandra) | Cosmos DB (Cassandra API) | Bigtable |
| In-memory / Redis | ElastiCache (Redis) | Azure Cache for Redis | Memorystore |
| Busca | OpenSearch Service | Cognitive Search | Cloud Search |

### AWS DynamoDB

```bash
# Criar tabela
aws dynamodb create-table \
  --table-name Usuarios \
  --attribute-definitions \
    AttributeName=id,AttributeType=S \
  --key-schema \
    AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Listar tabelas
aws dynamodb list-tables

# Inserir item
aws dynamodb put-item \
  --table-name Usuarios \
  --item '{"id": {"S": "usr-001"}, "nome": {"S": "João"}, "email": {"S": "joao@email.com"}}'

# Buscar item por chave
aws dynamodb get-item \
  --table-name Usuarios \
  --key '{"id": {"S": "usr-001"}}'

# Query (por partition key)
aws dynamodb query \
  --table-name Usuarios \
  --key-condition-expression "id = :id" \
  --expression-attribute-values '{":id": {"S": "usr-001"}}'

# Scan (leitura completa — evitar em produção)
aws dynamodb scan --table-name Usuarios

# Deletar item
aws dynamodb delete-item \
  --table-name Usuarios \
  --key '{"id": {"S": "usr-001"}}'

# Deletar tabela
aws dynamodb delete-table --table-name Usuarios
```

### GCP Firestore

```bash
# Criar banco Firestore (via Console ou Terraform; CLI é limitada)
gcloud firestore databases create --location=us-central1

# Exportar dados
gcloud firestore export gs://meu-bucket/backup-firestore

# Importar dados
gcloud firestore import gs://meu-bucket/backup-firestore

# Listar databases
gcloud firestore databases list

# Deletar banco
gcloud firestore databases delete --database=meu-banco
```

---

## Cache

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Redis gerenciado | ElastiCache (Redis) | Azure Cache for Redis | Memorystore for Redis |
| Memcached | ElastiCache (Memcached) | — | Memorystore for Memcached |

### AWS ElastiCache (Redis)

```bash
# Criar cluster Redis
aws elasticache create-replication-group \
  --replication-group-id meu-redis \
  --replication-group-description "Cache Redis" \
  --num-cache-clusters 1 \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --engine-version 7.0

# Listar clusters
aws elasticache describe-replication-groups --output table

# Criar snapshot
aws elasticache create-snapshot \
  --replication-group-id meu-redis \
  --snapshot-name meu-redis-snapshot

# Deletar cluster
aws elasticache delete-replication-group --replication-group-id meu-redis
```

### GCP Memorystore

```bash
# Criar instância Redis
gcloud redis instances create meu-redis \
  --size=1 \
  --region=us-central1 \
  --redis-version=redis_7_0 \
  --tier=BASIC

# Listar instâncias
gcloud redis instances list --region=us-central1

# Descrever instância (pegar IP)
gcloud redis instances describe meu-redis --region=us-central1

# Deletar instância
gcloud redis instances delete meu-redis --region=us-central1
```

---

## Mensageria e Filas

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Filas simples | SQS | Service Bus (Queues) | Cloud Tasks |
| Pub/Sub | SNS | Service Bus (Topics) | Pub/Sub |
| Streaming | Kinesis | Event Hubs | Dataflow / Pub/Sub |
| Message Broker | Amazon MQ | Service Bus | — |

### AWS SQS / SNS

```bash
# --- SQS ---
# Criar fila
aws sqs create-queue --queue-name minha-fila

# Criar fila FIFO
aws sqs create-queue \
  --queue-name minha-fila.fifo \
  --attributes FifoQueue=true,ContentBasedDeduplication=true

# Listar filas
aws sqs list-queues

# Enviar mensagem
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/minha-fila \
  --message-body "Olá, Mundo!"

# Receber mensagens (long polling)
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/minha-fila \
  --wait-time-seconds 10

# Deletar mensagem
aws sqs delete-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/minha-fila \
  --receipt-handle <receipt-handle>

# --- SNS ---
# Criar tópico
aws sns create-topic --name meu-topico

# Listar tópicos
aws sns list-topics

# Publicar mensagem
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:meu-topico \
  --message "Evento disparado!"

# Criar inscrição (SQS como subscriber)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:meu-topico \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:minha-fila
```

### GCP Pub/Sub

```bash
# Criar tópico
gcloud pubsub topics create meu-topico

# Listar tópicos
gcloud pubsub topics list

# Criar subscription
gcloud pubsub subscriptions create minha-sub --topic=meu-topico

# Publicar mensagem
gcloud pubsub topics publish meu-topico --message="Olá, Mundo!"

# Consumir mensagens
gcloud pubsub subscriptions pull minha-sub --auto-ack --limit=10

# Listar subscriptions
gcloud pubsub subscriptions list

# Deletar subscription e tópico
gcloud pubsub subscriptions delete minha-sub
gcloud pubsub topics delete meu-topico
```

---

## Rede e DNS

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Rede virtual privada | VPC | Virtual Network (VNet) | VPC |
| Sub-redes | Subnets | Subnets | Subnets |
| DNS gerenciado | Route 53 | Azure DNS | Cloud DNS |
| VPN Site-to-Site | AWS Site-to-Site VPN | VPN Gateway | Cloud VPN |
| Conexão dedicada | Direct Connect | ExpressRoute | Cloud Interconnect |
| Peering de VPCs | VPC Peering | VNet Peering | VPC Peering |

### AWS VPC e Route 53

```bash
# --- VPC ---
# Criar VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text

# Criar subnet pública
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Criar Internet Gateway e associar
aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxxxx --vpc-id vpc-xxxxx

# Listar VPCs
aws ec2 describe-vpcs --output table

# --- Route 53 ---
# Listar hosted zones
aws route53 list-hosted-zones --output table

# Criar hosted zone
aws route53 create-hosted-zone \
  --name meudominio.com \
  --caller-reference $(date +%s)

# Criar registro DNS (A record)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.meudominio.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }]
  }'
```

### GCP Cloud DNS

```bash
# Criar zona DNS
gcloud dns managed-zones create minha-zona \
  --dns-name=meudominio.com. \
  --description="Zona principal"

# Listar zonas
gcloud dns managed-zones list

# Adicionar registro A
gcloud dns record-sets create app.meudominio.com. \
  --zone=minha-zona \
  --type=A \
  --ttl=300 \
  --rrdatas=1.2.3.4

# Listar registros
gcloud dns record-sets list --zone=minha-zona

# Deletar zona
gcloud dns managed-zones delete minha-zona
```

---

## Load Balancer e CDN

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Load Balancer HTTP/S | Application Load Balancer (ALB) | Application Gateway | Cloud Load Balancing (HTTP/S) |
| Load Balancer TCP/UDP | Network Load Balancer (NLB) | Azure Load Balancer | Cloud Load Balancing (TCP/UDP) |
| CDN | CloudFront | Azure CDN / Front Door | Cloud CDN |
| DNS global + CDN | Route 53 + CloudFront | Azure Front Door | Cloud Armor + Cloud CDN |

### AWS CloudFront

```bash
# Criar distribuição CloudFront (origin = S3)
aws cloudfront create-distribution \
  --distribution-config '{
    "Origins": {
      "Quantity": 1,
      "Items": [{
        "Id": "meu-s3",
        "DomainName": "meu-bucket.s3.amazonaws.com",
        "S3OriginConfig": {"OriginAccessIdentity": ""}
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "meu-s3",
      "ViewerProtocolPolicy": "redirect-to-https",
      "ForwardedValues": {"QueryString": false, "Cookies": {"Forward": "none"}},
      "MinTTL": 0
    },
    "Comment": "Minha distribuição",
    "Enabled": true
  }'

# Listar distribuições
aws cloudfront list-distributions --query 'DistributionList.Items[*].[Id,DomainName,Status]' --output table

# Invalidar cache (após deploy)
aws cloudfront create-invalidation \
  --distribution-id E1234567890 \
  --paths "/*"
```

---

## Segurança e Gerenciamento de Segredos

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Gerenciamento de segredos | Secrets Manager | Key Vault | Secret Manager |
| Gerenciamento de chaves | KMS | Key Vault (Keys) | Cloud KMS |
| Firewall de aplicação | WAF | Azure WAF | Cloud Armor |
| DDoS protection | AWS Shield | Azure DDoS Protection | Cloud Armor |
| Varredura de vulnerabilidades | Amazon Inspector | Microsoft Defender | Security Command Center |
| Monitoramento de ameaças | Amazon GuardDuty | Microsoft Sentinel | Security Command Center |

### AWS Secrets Manager / KMS

```bash
# --- Secrets Manager ---
# Criar segredo
aws secretsmanager create-secret \
  --name /producao/banco/senha \
  --description "Senha do banco de produção" \
  --secret-string '{"username":"admin","password":"senha-super-secreta"}'

# Ler segredo
aws secretsmanager get-secret-value --secret-id /producao/banco/senha

# Atualizar segredo
aws secretsmanager put-secret-value \
  --secret-id /producao/banco/senha \
  --secret-string '{"username":"admin","password":"nova-senha"}'

# Listar segredos
aws secretsmanager list-secrets --output table

# Rotacionar segredo
aws secretsmanager rotate-secret --secret-id /producao/banco/senha

# Deletar segredo (com período de recuperação)
aws secretsmanager delete-secret \
  --secret-id /producao/banco/senha \
  --recovery-window-in-days 7

# --- KMS ---
# Criar chave KMS
aws kms create-key \
  --description "Chave de criptografia da aplicação" \
  --key-usage ENCRYPT_DECRYPT

# Criar alias para a chave
aws kms create-alias \
  --alias-name alias/minha-chave \
  --target-key-id <key-id>

# Criptografar dado
aws kms encrypt \
  --key-id alias/minha-chave \
  --plaintext "dados-sensíveis" \
  --output text \
  --query CiphertextBlob | base64 --decode > dado-cifrado.bin

# Descriptografar dado
aws kms decrypt \
  --ciphertext-blob fileb://dado-cifrado.bin \
  --output text \
  --query Plaintext | base64 --decode

# Listar chaves
aws kms list-keys --output table
```

### Azure Key Vault

```bash
# Criar Key Vault
az keyvault create \
  --name meu-keyvault \
  --resource-group meu-rg \
  --location eastus

# Criar segredo
az keyvault secret set \
  --vault-name meu-keyvault \
  --name "DbPassword" \
  --value "senha-super-secreta"

# Ler segredo
az keyvault secret show \
  --vault-name meu-keyvault \
  --name "DbPassword" \
  --query value --output tsv

# Listar segredos
az keyvault secret list --vault-name meu-keyvault --output table

# Criar chave criptográfica
az keyvault key create \
  --vault-name meu-keyvault \
  --name minha-chave \
  --kty RSA \
  --size 2048

# Definir política de acesso
az keyvault set-policy \
  --name meu-keyvault \
  --object-id <principal-id> \
  --secret-permissions get list set delete

# Deletar segredo (soft delete)
az keyvault secret delete --vault-name meu-keyvault --name "DbPassword"
```

### GCP Secret Manager

```bash
# Criar segredo
gcloud secrets create db-password \
  --replication-policy=automatic

# Adicionar versão ao segredo
echo -n "senha-super-secreta" | gcloud secrets versions add db-password --data-file=-

# Ler versão mais recente
gcloud secrets versions access latest --secret=db-password

# Listar segredos
gcloud secrets list

# Listar versões de um segredo
gcloud secrets versions list db-password

# Deletar segredo
gcloud secrets delete db-password

# Conceder acesso a service account
gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:minha-sa@meu-projeto.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

---

## Autenticação e Identidade

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Gerenciamento de identidade | IAM | Azure Active Directory (Entra ID) | Cloud IAM |
| Roles e políticas | IAM Roles / Policies | RBAC / Azure Roles | IAM Roles |
| Service accounts | IAM Roles for Services | Managed Identities | Service Accounts |
| SSO | AWS SSO (IAM Identity Center) | Azure AD SSO | Cloud Identity |
| OIDC / OAuth federado | IAM Identity Center | Azure AD B2C | Identity Platform |
| Cognito (usuário final) | Amazon Cognito | Azure AD B2C | Identity Platform / Firebase Auth |

### AWS IAM

```bash
# Criar usuário
aws iam create-user --user-name joao-developer

# Criar grupo
aws iam create-group --group-name Desenvolvedores

# Adicionar usuário ao grupo
aws iam add-user-to-group --user-name joao-developer --group-name Desenvolvedores

# Anexar política ao grupo
aws iam attach-group-policy \
  --group-name Desenvolvedores \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Criar política customizada
aws iam create-policy \
  --policy-name MinhaPolíticaCustom \
  --policy-document file://politica.json

# Criar role (para EC2 assumir)
aws iam create-role \
  --role-name MinhaRoleEC2 \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Listar usuários
aws iam list-users --output table

# Criar access key para usuário
aws iam create-access-key --user-name joao-developer

# Gerar relatório de credenciais
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 --decode
```

### Azure IAM (RBAC)

```bash
# Listar role assignments de um resource group
az role assignment list --resource-group meu-rg --output table

# Atribuir role a usuário
az role assignment create \
  --assignee joao@empresa.com \
  --role "Storage Blob Data Reader" \
  --scope /subscriptions/<sub-id>/resourceGroups/meu-rg

# Atribuir role a service principal
az role assignment create \
  --assignee <service-principal-id> \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/meu-rg

# Criar service principal
az ad sp create-for-rbac \
  --name meu-sp \
  --role Contributor \
  --scopes /subscriptions/<sub-id>/resourceGroups/meu-rg

# Listar service principals
az ad sp list --output table

# Criar managed identity
az identity create --resource-group meu-rg --name minha-identidade

# Listar custom roles
az role definition list --custom-role-only true --output table
```

### GCP IAM + Service Accounts

```bash
# Listar policies do projeto
gcloud projects get-iam-policy meu-projeto

# Adicionar role a usuário
gcloud projects add-iam-policy-binding meu-projeto \
  --member="user:joao@empresa.com" \
  --role="roles/storage.objectViewer"

# Criar service account
gcloud iam service-accounts create minha-sa \
  --display-name="Minha Service Account"

# Conceder role à service account
gcloud projects add-iam-policy-binding meu-projeto \
  --member="serviceAccount:minha-sa@meu-projeto.iam.gserviceaccount.com" \
  --role="roles/cloudsql.client"

# Criar chave para service account
gcloud iam service-accounts keys create chave.json \
  --iam-account=minha-sa@meu-projeto.iam.gserviceaccount.com

# Listar service accounts
gcloud iam service-accounts list

# Listar roles disponíveis
gcloud iam roles list --filter="name:roles/storage"

# Criar custom role
gcloud iam roles create MinhaRoleCustom \
  --project=meu-projeto \
  --title="Minha Role Customizada" \
  --permissions=storage.objects.get,storage.objects.list

# Impersonar service account (para testes)
gcloud auth impersonate-service-account minha-sa@meu-projeto.iam.gserviceaccount.com \
  -- gcloud storage ls
```

---

## Monitoramento e Observabilidade

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Métricas e dashboards | CloudWatch | Azure Monitor | Cloud Monitoring |
| Logs centralizados | CloudWatch Logs | Log Analytics (Monitor) | Cloud Logging |
| Traces distribuídos | X-Ray | Application Insights | Cloud Trace |
| Alertas | CloudWatch Alarms | Azure Alerts | Cloud Monitoring Alerts |
| Dashboards personalizados | CloudWatch Dashboards | Azure Workbooks | Cloud Monitoring Dashboards |

### AWS CloudWatch

```bash
# --- Logs ---
# Criar log group
aws logs create-log-group --log-group-name /minha-app/producao

# Listar log groups
aws logs describe-log-groups --output table

# Ver logs em tempo real (tail)
aws logs tail /minha-app/producao --follow --format short

# Filtrar logs
aws logs filter-log-events \
  --log-group-name /minha-app/producao \
  --start-time $(date -d '1 hour ago' +%s000) \
  --filter-pattern "ERROR"

# --- Métricas e Alarmes ---
# Publicar métrica customizada
aws cloudwatch put-metric-data \
  --namespace MinhaApp \
  --metric-name Erros \
  --value 1 \
  --unit Count

# Criar alarme
aws cloudwatch put-metric-alarm \
  --alarm-name "CPU-Alta" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alertas

# Listar alarmes
aws cloudwatch describe-alarms --output table

# Obter métricas
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average
```

### GCP Cloud Logging / Monitoring

```bash
# --- Logging ---
# Escrever log entry
gcloud logging write minha-app "Mensagem de teste" --severity=INFO

# Ler logs recentes
gcloud logging read "resource.type=gce_instance" --limit=50 --format=json

# Filtrar logs com expressão
gcloud logging read 'severity>=ERROR AND resource.type="cloud_run_revision"' --limit=20

# Criar sink (exportar logs para GCS)
gcloud logging sinks create meu-sink \
  storage.googleapis.com/meu-bucket-logs \
  --log-filter='severity>=WARNING'

# Listar sinks
gcloud logging sinks list

# --- Monitoring ---
# Listar políticas de alerta
gcloud alpha monitoring policies list

# Listar grupos de notificação
gcloud beta monitoring channels list
```

---

## CI/CD e DevOps

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| CI/CD nativo | CodePipeline + CodeBuild | Azure Pipelines | Cloud Build |
| Repositório Git | CodeCommit | Azure Repos | Cloud Source Repositories |
| Artefatos | CodeArtifact | Azure Artifacts | Artifact Registry |
| Deploy | CodeDeploy | Azure Deploy | Cloud Deploy |

### AWS CodeBuild / CodePipeline

```bash
# Iniciar build manualmente
aws codebuild start-build --project-name meu-projeto-build

# Listar projetos de build
aws codebuild list-projects --output table

# Ver histórico de builds
aws codebuild list-builds-for-project --project-name meu-projeto-build

# Obter detalhes de um build
aws codebuild batch-get-builds --ids <build-id>

# Listar pipelines
aws codepipeline list-pipelines --output table

# Executar pipeline manualmente
aws codepipeline start-pipeline-execution --name meu-pipeline

# Ver status do pipeline
aws codepipeline get-pipeline-state --name meu-pipeline
```

### GCP Cloud Build

```bash
# Iniciar build a partir de diretório local
gcloud builds submit \
  --tag us-central1-docker.pkg.dev/meu-projeto/meu-repo/minha-app:v1 \
  .

# Iniciar build com cloudbuild.yaml
gcloud builds submit --config cloudbuild.yaml .

# Listar builds
gcloud builds list --limit=20

# Ver logs de um build
gcloud builds log <build-id>

# Criar trigger de build (push para branch)
gcloud builds triggers create github \
  --repo-name=meu-repo \
  --repo-owner=meu-usuario \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml

# Listar triggers
gcloud builds triggers list
```

---

## Infraestrutura como Código

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| IaC nativa | CloudFormation | ARM Templates / Bicep | Deployment Manager |
| IaC multi-cloud (popular) | Terraform / CDK | Terraform / Pulumi | Terraform / Pulumi |

### CloudFormation

```bash
# Criar stack
aws cloudformation create-stack \
  --stack-name minha-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,ParameterValue=producao \
  --capabilities CAPABILITY_IAM

# Atualizar stack
aws cloudformation update-stack \
  --stack-name minha-stack \
  --template-body file://template.yaml

# Listar stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --output table

# Descrever stack (ver outputs)
aws cloudformation describe-stacks --stack-name minha-stack

# Validar template
aws cloudformation validate-template --template-body file://template.yaml

# Deletar stack
aws cloudformation delete-stack --stack-name minha-stack

# Ver eventos (debug de falhas)
aws cloudformation describe-stack-events --stack-name minha-stack --output table
```

### Azure Bicep / ARM

```bash
# Validar template Bicep
az deployment group validate \
  --resource-group meu-rg \
  --template-file main.bicep \
  --parameters env=producao

# Deploy de template Bicep
az deployment group create \
  --resource-group meu-rg \
  --template-file main.bicep \
  --parameters env=producao

# Listar deployments
az deployment group list --resource-group meu-rg --output table

# Ver resultado de deployment
az deployment group show \
  --resource-group meu-rg \
  --name main

# Rollback para versão anterior
az deployment group create \
  --resource-group meu-rg \
  --template-file main-anterior.bicep
```

---

## IA e Machine Learning

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| Plataforma ML | SageMaker | Azure Machine Learning | Vertex AI |
| LLMs / IA generativa | Amazon Bedrock | Azure OpenAI Service | Vertex AI Gemini |
| Visão computacional | Rekognition | Computer Vision / Azure AI Vision | Cloud Vision API |
| NLP / Texto | Comprehend | Azure Language Service | Natural Language API |
| Fala (STT/TTS) | Transcribe / Polly | Azure Speech Services | Speech-to-Text / Text-to-Speech |
| Tradução | Amazon Translate | Azure Translator | Cloud Translation API |
| Busca inteligente | Kendra | Azure Cognitive Search | Vertex AI Search |

### AWS Bedrock (LLMs)

```bash
# Listar modelos disponíveis
aws bedrock list-foundation-models --output table

# Invocar modelo (Claude)
aws bedrock-runtime invoke-model \
  --model-id anthropic.claude-3-5-sonnet-20241022-v2:0 \
  --body '{
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Explique cloud computing em 3 linhas"}]
  }' \
  output.json && cat output.json
```

### GCP Vertex AI

```bash
# Listar endpoints
gcloud ai endpoints list --region=us-central1

# Fazer predição em endpoint deployado
gcloud ai endpoints predict <endpoint-id> \
  --region=us-central1 \
  --json-request=request.json

# Listar datasets
gcloud ai datasets list --region=us-central1

# Criar dataset
gcloud ai datasets create \
  --display-name=meu-dataset \
  --metadata-schema-uri=gs://google-cloud-aiplatform/schema/dataset/metadata/text_1.0.0.yaml \
  --region=us-central1
```

---

## Tabela Comparativa Geral

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| **Computação** | EC2 | Virtual Machines | Compute Engine |
| **Kubernetes** | EKS | AKS | GKE |
| **Serverless containers** | ECS Fargate | Container Apps | Cloud Run |
| **Functions (FaaS)** | Lambda | Azure Functions | Cloud Functions |
| **Object Storage** | S3 | Blob Storage | Cloud Storage |
| **Banco SQL** | RDS | Azure DB for PostgreSQL/MySQL | Cloud SQL |
| **Banco NoSQL document** | DynamoDB | Cosmos DB | Firestore |
| **Cache** | ElastiCache | Azure Cache for Redis | Memorystore |
| **Filas** | SQS | Service Bus | Cloud Tasks |
| **Pub/Sub** | SNS | Service Bus Topics | Pub/Sub |
| **CDN** | CloudFront | Azure CDN / Front Door | Cloud CDN |
| **DNS** | Route 53 | Azure DNS | Cloud DNS |
| **Segredos** | Secrets Manager | Key Vault | Secret Manager |
| **Chaves criptográficas** | KMS | Key Vault | Cloud KMS |
| **Identidade** | IAM | Entra ID / RBAC | Cloud IAM |
| **Autenticação usuário final** | Cognito | Azure AD B2C | Identity Platform |
| **Monitoramento** | CloudWatch | Azure Monitor | Cloud Monitoring |
| **Logs** | CloudWatch Logs | Log Analytics | Cloud Logging |
| **IaC nativa** | CloudFormation | ARM / Bicep | Deployment Manager |
| **CI/CD** | CodePipeline + CodeBuild | Azure Pipelines | Cloud Build |
| **LLMs / IA generativa** | Bedrock | Azure OpenAI | Vertex AI Gemini |
| **Rede privada** | VPC | Virtual Network (VNet) | VPC |
| **Conexão dedicada** | Direct Connect | ExpressRoute | Cloud Interconnect |
| **WAF** | AWS WAF | Azure WAF | Cloud Armor |
| **Registro de container** | ECR | ACR | Artifact Registry |
| **Streaming de dados** | Kinesis | Event Hubs | Pub/Sub + Dataflow |

---

> Documentação oficial: [AWS](https://docs.aws.amazon.com) · [Azure](https://learn.microsoft.com/azure) · [GCP](https://cloud.google.com/docs)
