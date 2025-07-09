# FIAP Hackathon - Infraestrutura para Processamento de Vídeos

Este repositório contém os arquivos de configuração de infraestrutura para o sistema de processamento de vídeos da FIAP Hackathon, utilizando Kubernetes e Terraform.

## Visão Geral da Arquitetura

O projeto é composto por três serviços principais executando em um cluster Kubernetes:

1. **Video Processing API Core** - API principal para processamento de vídeos
2. **Image Upload Service** - Serviço para upload e gerenciamento de frames de vídeo
3. **Notification Service** - Serviço para envio de notificações aos usuários

![Diagrama da Arquitetura](./docs/architecture.png)

## Estrutura do Repositório

```
fiap-hackathon-k8s/
├── kubernetes/               # Arquivos de configuração Kubernetes
│   ├── 01-namespace.yaml     # Definição do namespace
│   ├── ...                   # Outros recursos K8s
├── terraform/                # Configuração de infraestrutura Terraform
│   ├── main.tf               # Definição principal do Terraform
│   ├── variables.tf          # Variáveis da infraestrutura
│   └── outputs.tf            # Outputs do Terraform
└── README.md                 # Este arquivo
```

## Requisitos

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) v1.22+
- [Terraform](https://www.terraform.io/downloads.html) v1.0+
- [AWS CLI](https://aws.amazon.com/cli/) configurado com credenciais apropriadas
- [Helm](https://helm.sh/docs/intro/install/) v3.8+

## Terraform

O Terraform é utilizado para provisionar a infraestrutura na AWS, incluindo o cluster EKS (Elastic Kubernetes Service).

### Principais Recursos Provisionados

- Cluster EKS
- Grupos de nós (node groups)
- VPC e sub-redes
- Roles e políticas IAM
- Volumes EBS para persistência de dados

### Configuração e Uso do Terraform

1. Navegue até o diretório `terraform`:

```bash
cd terraform
```

2. Inicialize o Terraform:

```bash
terraform init
```

3. Planeje as alterações:

```bash
terraform plan -out=tfplan
```

4. Aplique as alterações:

```bash
terraform apply tfplan
```

5. Para destruir a infraestrutura (quando não for mais necessária):

```bash
terraform destroy
```

### Variáveis do Terraform

| Nome | Descrição | Padrão |
|------|-----------|--------|
| `region` | Região da AWS | `us-east-2` |
| `cluster_name` | Nome do cluster EKS | `fiap-hackathon-cluster` |
| `node_group_instance_type` | Tipo de instância para os nós | `t3.medium` |
| `min_nodes` | Número mínimo de nós | `2` |
| `max_nodes` | Número máximo de nós | `4` |

## Kubernetes

### Namespace

Todos os recursos são instalados no namespace `fiap-hackathon`.

```bash
kubectl apply -f kubernetes/01-namespace.yaml
```

### Deployments

Os serviços são implantados como deployments Kubernetes, cada um com seu próprio conjunto de recursos.

#### Video Processing API Core

```bash
kubectl apply -f kubernetes/03-video-api-configmap.yaml
kubectl apply -f kubernetes/04-db-configmap-video.yaml
kubectl apply -f kubernetes/05-db-video.yaml
kubectl apply -f kubernetes/06-svc-video-service-api.yaml
kubectl apply -f kubernetes/07-svc-db-video.yaml
kubectl apply -f kubernetes/08-video-api-deployment.yaml
```

#### Image Upload Service

```bash
kubectl apply -f kubernetes/03-image-upload-api-configmap.yaml
kubectl apply -f kubernetes/04-db-configmap-image-upload.yaml
kubectl apply -f kubernetes/05-db-image-upload.yaml
kubectl apply -f kubernetes/06-1-svc-image-upload-service-api.yaml
kubectl apply -f kubernetes/07-svc-db-image-upload.yaml
kubectl apply -f kubernetes/08-image-upload-api-deployment.yaml
```

#### Notification Service

```bash
kubectl apply -f kubernetes/03-notification-api-configmap.yaml
kubectl apply -f kubernetes/04-db-configmap-notification.yaml
kubectl apply -f kubernetes/05-db-notification.yaml
kubectl apply -f kubernetes/06-2-svc-notification-api-service-api.yaml
kubectl apply -f kubernetes/07-svc-db-notification.yaml
kubectl apply -f kubernetes/08-notification-api-deployment.yaml
```

### Volumes Persistentes

Os bancos de dados PostgreSQL utilizam PersistentVolume e PersistentVolumeClaim para garantir a persistência dos dados.

```bash
kubectl apply -f kubernetes/2-persistentVolume.db.yaml
kubectl apply -f kubernetes/3-persistentVolumeClaim.db.yaml
```

### ConfigMaps e Secrets

Os ConfigMaps são utilizados para armazenar configurações não sensíveis:

- Variáveis de ambiente para conexão com banco de dados
- Configurações das APIs
- URLs de serviços

Secrets são utilizados para armazenar informações sensíveis como:

- Senhas de banco de dados
- Chaves de API
- Tokens de autenticação

## Monitoramento

O cluster utiliza Prometheus e Grafana para monitoramento, instalados via Helm:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

## Pipeline CI/CD

O repositório inclui um workflow do GitHub Actions para automação do deploy:

1. Validação dos arquivos Terraform
2. Testes de configuração do Kubernetes
3. Aplicação automática das alterações

## Acesso aos Serviços

Após a implantação, os serviços estarão disponíveis nos seguintes endpoints:

- **Video Processing API**: `http://<LOAD_BALANCER_IP>:8080`
- **Image Upload Service**: `http://<LOAD_BALANCER_IP>:8081`
- **Notification Service**: `http://<LOAD_BALANCER_IP>:8082`

Para obter o IP do Load Balancer:

```bash
kubectl get svc -n fiap-hackathon
```

## Resolução de Problemas

### Verificando logs dos pods

```bash
kubectl logs -f deployment/video-processing-api -n fiap-hackathon
kubectl logs -f deployment/image-upload-service -n fiap-hackathon
kubectl logs -f deployment/notification-service -n fiap-hackathon
```

### Verificando status dos pods

```bash
kubectl get pods -n fiap-hackathon
```

### Reiniciando um deployment

```bash
kubectl rollout restart deployment/video-processing-api -n fiap-hackathon
```

