# 🚀 ECS CI/CD Pipeline - GitHub Actions + Amazon ECS Fargate

Pipeline CI/CD automatizado com GitHub Actions para deploy de uma ASP.NET Core Web API no Amazon ECS Fargate.

---

## 🎯 Objetivo

Automatizar completamente o processo de build, containerização e deploy de uma API .NET no ECS — a cada push no `main`, o pipeline executa tudo sem intervenção manual.

---

## 🏗️ Arquitetura do Pipeline

```
push main → GitHub Actions
    ├── job: build  → dotnet restore + build
    ├── job: publish → docker build + push ECR
    └── job: deploy  → atualiza ECS task definition + service
```

---

## 📁 Estrutura do Projeto

```
ecs-cicd-pipeline/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Pipeline CI/CD
├── BookManager/
│   ├── Program.cs              # API minimal (Hello World)
│   └── BookManager.csproj      # Container config
└── deployments/
    └── ecs-task-definition.json # Task definition do ECS
```

---

## ⚙️ Configuração

### 1. Secrets no GitHub

**Repositório → Settings → Secrets and variables → Actions**

| Secret | Descrição |
|--------|-----------|
| `AWS_ACCESS_KEY_ID` | Access Key do IAM user |
| `AWS_SECRET_ACCESS_KEY` | Secret Key do IAM user |
| `REPOSITORY` | URI do repositório ECR |

### 2. IAM User (github-actions)

Permissões necessárias:
- `AmazonEC2ContainerRegistryFullAccess`
- `AmazonECS_FullAccess`

### 3. Infraestrutura AWS

- **ECR:** repositório `bookmanager`
- **ECS Cluster:** `book-manager-cluster` (Fargate)
- **Task Definition:** `bookmanager`
- **Service:** `bookmanager-svc`
- **IAM Role:** `ecsTaskExecutionRole` com `CloudWatchLogsFullAccess`

---

## 🔄 Pipeline (deploy.yml)

### Job: build
```yaml
- dotnet restore
- dotnet build
```

### Job: publish
```yaml
- Configure AWS Credentials
- Login to ECR
- dotnet publish → gera imagem Docker
- docker push → envia para ECR
```

### Job: deploy
```yaml
- Configure AWS Credentials
- aws-actions/amazon-ecs-deploy-task-definition
  → atualiza task definition com nova imagem
  → força novo deploy no ECS service
```

---

## 🧪 Testando o Pipeline

Qualquer push no `main` dispara o pipeline automaticamente:

```bash
# Alterar a mensagem
# Program.cs: app.MapGet("/", () => "Hello World V2!");

git add .
git commit -m "feat: atualiza mensagem"
git push
```

Acompanhe em: **GitHub → Actions → Deploy 🚀**

---

## 🔑 Pontos-Chave

### GitHub Actions
- `workflow_dispatch` — permite disparar manualmente pelo console
- `needs: build` — garante ordem de execução dos jobs
- Secrets criptografados — credenciais AWS nunca expostas no código

### Containerização sem Dockerfile
O `.csproj` define a imagem diretamente:
```xml
<ContainerImageTags>1.0.0;latest</ContainerImageTags>
<ContainerRepository>bookmanager</ContainerRepository>
<PublishProfile>DefaultContainer</PublishProfile>
```

### ECS Task Definition como código
O arquivo `deployments/ecs-task-definition.json` versionado no repositório garante que a infraestrutura seja rastreável e reproduzível.

---

## 🛠️ Tecnologias

- **.NET 10** - Runtime
- **GitHub Actions** - CI/CD
- **Amazon ECR** - Registry de imagens Docker
- **Amazon ECS + Fargate** - Orquestração serverless
- **CloudWatch Logs** - Logs dos containers

---

## 📚 Aprendizados

✅ Criar pipeline CI/CD com GitHub Actions  
✅ Configurar secrets no GitHub  
✅ Build e push automático de imagem Docker para ECR  
✅ Deploy automático no ECS via GitHub Actions  
✅ Containerizar .NET sem Dockerfile usando `dotnet publish`  
✅ Versionar task definition como código  
✅ Configurar permissões IAM para GitHub Actions e ECS
