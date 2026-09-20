# DevOps Learning Journey

> A step-by-step plan to go from zero to understanding and applying core DevOps practices.
> This file is our source of truth — updated as we learn.

---

## The Big Picture

DevOps is about one thing: getting code from your laptop to users — fast, safely, and automatically.

Without DevOps: you write code → manually test → zip & send to ops team → ops deploys (maybe days later) → something breaks → nobody knows why → slow fix cycle.

With DevOps: you push code → tests run automatically → app is packaged automatically → deployed automatically → monitored automatically → alerts if something breaks → fix fast, repeat.

```
Without DevOps                  With DevOps
--------------                  -----------
Write code                      Write code → push to Git
  ↓ manually test                 ↓ tests run automatically
  ↓ zip & send to ops             ↓ app is packaged automatically
  ↓ ops deploys (days later)      ↓ deployed automatically
  ↓ something breaks              ↓ monitored automatically
  ↓ nobody knows why              ↓ alerts if something breaks
  ↓ slow fix cycle                ↓ fix fast, repeat
```

---

## Step 1 — DevOps Fundamentals


### Concepts

**What is DevOps?**

DevOps is a set of practices that combines software development (Dev) and IT operations (Ops). The goal is to shorten the time between writing code and getting it running in production — reliably, repeatedly, automatically.

It's not a tool. It's a culture + process + toolset.

---

**The problem it solves**

Traditional model:
- Devs write code → hand it to Ops
- Ops deploy manually → something breaks
- Blame game starts
- Slow feedback loop

DevOps model:
- Same team owns code from writing to running
- Everything automated
- Fast feedback when things break
- Deploy many times a day safely

---

**CI — Continuous Integration**

Every time code is pushed, it is automatically built and tested. If something breaks, the team knows immediately — not days later.

The goal: always have a codebase that is in a working, testable state.

---

**CD — Continuous Delivery / Deployment**

Continuous Delivery: every passing build is automatically packaged and ready to deploy with one click.

Continuous Deployment: every passing build is automatically deployed to production — no human click needed.

Most teams use Continuous Delivery and add a manual approval gate before production.

---

**Pipeline**

A pipeline is the automated sequence of steps your code goes through after a push.

```
push to git
  ↓ install dependencies
  ↓ run tests
  ↓ build artifact / Docker image
  ↓ push to registry
  ↓ deploy to staging
  ↓ (approval gate)
  ↓ deploy to production
```

---

**Infrastructure as Code (IaC)**

Instead of clicking through a cloud console to set up servers, you define your infrastructure in code files. Those files are version-controlled, reviewed, and applied automatically.

Tools: Terraform, AWS CDK, Pulumi.

---

**Shift Left**

Moving testing, security checks, and quality gates earlier in the development process. Instead of testing at the end, you test at every commit. Instead of scanning for security issues before release, you scan on every PR.

---

## Step 2 — Linux & The Terminal


### Concepts

**Why Linux**

Most servers, containers, and cloud instances run Linux. Being comfortable at the terminal is a prerequisite for everything else in DevOps.

---

**File system navigation**

```bash
pwd               # where am I
ls -la            # list files with permissions
cd /path/to/dir   # change directory
mkdir my-folder   # create folder
rm file.txt       # delete file
rm -rf folder/    # delete folder recursively
cp src dst        # copy
mv src dst        # move or rename
```

---

**Reading and editing files**

```bash
cat file.txt          # print file contents
less file.txt         # paginated view
echo "text" > file    # write to file
echo "text" >> file   # append to file
nano file.txt         # simple editor
```

---

**Permissions**

Every file has an owner and a permission set: read (r), write (w), execute (x) — for owner, group, and others.

```bash
ls -la          # shows permissions like -rwxr-xr-x
chmod 755 file  # owner=rwx, group=r-x, others=r-x
chmod +x file   # add execute permission
chown user file # change file owner
```

---

**Processes**

```bash
ps aux              # list all running processes
top                 # live process monitor
kill <pid>          # terminate a process
kill -9 <pid>       # force kill
```

---

**Networking basics**

```bash
ping google.com           # check connectivity
curl http://localhost:3000 # make an HTTP request
wget https://example.com/file.zip  # download a file
ss -tuln                  # list open ports
```

---

**Package managers**

```bash
# Debian / Ubuntu
sudo apt update
sudo apt install <package>

# RedHat / CentOS
sudo yum install <package>
```

---

**Environment variables**

```bash
export MY_VAR="hello"   # set a variable
echo $MY_VAR            # read it
env                     # list all env vars
```

---

## Step 3 — Git & Branching Workflows


### Concepts

**Why branching matters in DevOps**

A pipeline runs on code in a branch. How you branch directly affects how often you can deploy and how safely.

---

**Trunk-Based Development**

Everyone commits to one main branch (trunk/main) frequently — at least once a day. Feature flags hide incomplete features from users. This enables CI/CD because main is always deployable.

---

**GitFlow**

A heavier branching strategy with dedicated branches: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`. More process overhead, longer-lived branches. Common in teams with scheduled releases.

---

**Pull Requests / Merge Requests**

Before code merges into main, it goes through a PR. The pipeline runs automatically — tests, linting, security scans. Team members review the code. Only then does it merge.

This is the quality gate.

---

**Tagging releases**

```bash
git tag v1.0.0                     # create a tag
git tag v1.0.0 -m "First release"  # annotated tag
git push origin v1.0.0             # push tag to remote
git tag                            # list all tags
```

Tags mark a specific commit as a release. Your pipeline can use tags to trigger production deployments.

---

**Git hooks**

Scripts that run automatically at certain points in the git workflow.

- `pre-commit` — run linters/formatters before a commit is saved
- `pre-push` — run tests before code is pushed
- `commit-msg` — validate commit message format

Tools like Husky make these easy to configure.

---

**Conventional Commits**

A standardized commit message format that makes changelogs and versioning automatable.

```
feat: add user authentication
fix: correct null pointer in payment flow
docs: update API usage guide
chore: upgrade dependencies
```

---

## Step 4 — Docker & Containers


### Concepts

**What is a container?**

A container packages an application and all its dependencies — runtime, libraries, config — into one isolated unit. It runs the same way on any machine.

No more "works on my machine."

---

**Containers vs Virtual Machines**

| | Virtual Machine | Container |
|---|---|---|
| Includes | Full OS | Just the app + deps |
| Startup | Minutes | Seconds |
| Size | GBs | MBs |
| Isolation | Strong | Process-level |

Containers share the host OS kernel. VMs each have their own OS. Containers are lighter and faster.

---

**Dockerfile vs Image vs Container**

The easiest way to understand all three together:

- **Dockerfile** = the recipe (a text file with instructions: "use this base, install these packages, copy this code")
- **Image** = the baked cake (the actual result after running those instructions — a single packaged file ready to ship)
- **Container** = eating the cake (the image actually running as a live process)

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
(instructions)                 (built artifact)          (running process)
```

You write the Dockerfile once. Run `docker build` and it executes those instructions top to bottom and produces an image. That image is what you push to Docker Hub, pull on a server, and run.

The Dockerfile lives in your codebase (it's just a text file). The image is the output — like how source code compiles into a binary. You ship the image, not the Dockerfile.

---

**Dockerfile**

A text file with step-by-step instructions to build an image. Each instruction creates a layer.

```dockerfile
FROM node:20-alpine          # base image
WORKDIR /app                 # set working directory
COPY package*.json ./        # copy dependency files
RUN npm install              # install dependencies
COPY . .                     # copy app code
RUN npm run build            # build
EXPOSE 3000                  # document the port
CMD ["node", "server.js"]    # default command to run
```

---

**Multi-stage builds**

Build in one stage, copy only the output to a smaller final image. Keeps images lean.

```dockerfile
# Stage 1: build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: serve
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

**Docker registry**

A storage and distribution system for images.

- Docker Hub — public registry, free tier available
- AWS ECR — private, integrates with AWS
- GitHub Container Registry (GHCR) — integrates with GitHub

```bash
docker login
docker push myrepo/my-app:1.0
docker pull myrepo/my-app:1.0
```

---

**Volumes**

Containers are ephemeral — when they stop, their filesystem is gone. Volumes persist data outside the container lifecycle.

```bash
docker run -v /host/path:/container/path my-app
docker run -v my-named-volume:/data my-app
```

---

**Port mapping**

Containers have their own internal network. You expose container ports to the host using `-p host:container`.

```bash
docker run -p 8080:80 nginx
# host port 8080 → container port 80
```

---

**Essential Docker commands**

```bash
# Images
docker build -t my-app:1.0 .   # build image from Dockerfile
docker images                   # list local images
docker rmi my-app:1.0           # delete image

# Containers
docker run -d -p 3000:3000 my-app:1.0   # run detached
docker ps                                # list running containers
docker ps -a                             # include stopped
docker stop <id>                         # stop container
docker rm <id>                           # remove container
docker logs <id>                         # view logs
docker logs -f <id>                      # follow logs
docker exec -it <id> bash               # shell into container
```

---

**Docker Compose**

A tool to define and run multi-container applications with a single YAML file.

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
docker compose up -d      # start all services
docker compose down       # stop all services
docker compose logs -f    # follow all logs
docker compose ps         # list services
```

---

**.dockerignore**

Like `.gitignore` — tells Docker which files to exclude from the build context. Always include this.

```
node_modules
.git
.env
dist
*.log
```

---

## Step 5 — CI/CD Pipelines


### Concepts

**What a pipeline does**

A pipeline is an automated workflow triggered by a git event (push, PR, tag). It runs a sequence of steps — install, test, build, deploy — and stops if any step fails.

---

**Triggers**

Common trigger events across all CI/CD tools:
- Push to a branch
- Pull request / merge request opened or updated
- Push of a version tag (e.g. `v1.0.0`)
- Manual trigger
- Scheduled (cron)

---

**CI/CD tools — the landscape**

| Tool | Where it lives | Common in |
|---|---|---|
| GitHub Actions | Inside GitHub | Open source, startups |
| Azure Pipelines | Azure DevOps | Enterprises, Microsoft stack |
| AWS CodePipeline | AWS | AWS-native teams |
| GitLab CI | Inside GitLab | Self-hosted enterprises |
| Jenkins | Self-hosted | Legacy, large orgs |
| CircleCI / Buildkite | SaaS | Various |

The concepts (stages, jobs, steps, artifacts, secrets) are the same across all of them. The syntax and integration points differ.

---

**GitHub Actions**

GitHub's built-in CI/CD. Workflows are YAML files in `.github/workflows/`.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm test
      - run: npm run build
```

Secrets are stored in GitHub repository settings and referenced as `${{ secrets.MY_SECRET }}`.

---

**Azure DevOps Pipelines**

Microsoft's CI/CD platform — part of the Azure DevOps suite. Pipelines are defined in `azure-pipelines.yml` at the root of the repo. Common in enterprise and Microsoft-stack teams.

Azure DevOps is a full platform: it includes Repos (git hosting), Pipelines (CI/CD), Boards (work tracking), Artifacts (package registry), and Test Plans — all integrated.

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node'

  - script: npm ci
    displayName: 'Install dependencies'

  - script: npm test
    displayName: 'Run tests'

  - script: npm run build
    displayName: 'Build'
```

---

**Azure Pipelines — key concepts**

**Stage**: A logical grouping of jobs. Common pattern: `Build → Test → Deploy-Staging → Deploy-Prod`.

**Job**: A set of steps that run on the same agent (machine). Jobs within a stage can run in parallel.

**Step / Task**: A single unit of work — either a script or a pre-built task from the marketplace.

**Agent**: The machine that runs your pipeline. Microsoft-hosted agents (ubuntu, windows, mac) are provided. You can also run self-hosted agents on your own infrastructure.

**Service Connection**: A secure way to connect Azure Pipelines to external services — Docker Hub, AWS, Kubernetes, Azure subscriptions, etc.

---

**Multi-stage Azure pipeline**

```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        pool:
          vmImage: ubuntu-latest
        steps:
          - script: npm ci
          - script: npm test
          - script: npm run build

          - task: Docker@2
            inputs:
              command: buildAndPush
              repository: myrepo/my-app
              dockerfile: Dockerfile
              tags: $(Build.BuildId)

  - stage: Deploy_Staging
    dependsOn: Build
    jobs:
      - deployment: DeployToStaging
        environment: staging
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploy to staging"

  - stage: Deploy_Prod
    dependsOn: Deploy_Staging
    jobs:
      - deployment: DeployToProd
        environment: production   # can require manual approval
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploy to production"
```

---

**Environments and approvals (Azure DevOps)**

In Azure DevOps, Environments are named targets (staging, production). You can add approval gates — a person must approve before the pipeline proceeds to that stage.

This is how most enterprise teams gate production deployments.

---

**Variable groups (Azure DevOps)**

Instead of storing secrets per-pipeline, Azure DevOps has Variable Groups — a shared set of variables and secrets you link to pipelines. Can also be backed by Azure Key Vault.

```yaml
variables:
  - group: my-app-secrets   # links a variable group
```

---

**Build and push Docker image in CI**

GitHub Actions:
```yaml
- uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- uses: docker/build-push-action@v5
  with:
    push: true
    tags: myrepo/my-app:${{ github.sha }}
```

Azure Pipelines:
```yaml
- task: Docker@2
  inputs:
    containerRegistry: my-docker-service-connection
    repository: myrepo/my-app
    command: buildAndPush
    tags: $(Build.BuildId)
```

---

**Artifact vs Image**

- Artifact: a build output (zip, JAR, binary) stored temporarily for later pipeline stages
- Image: a Docker image pushed to a registry, used by deployment systems

---

**Deployment strategies**

| Strategy | What it means |
|---|---|
| Recreate | Stop old, start new — brief downtime |
| Rolling update | Replace instances one by one — no downtime |
| Blue/Green | Two identical envs, switch traffic — instant rollback |
| Canary | Route small % of traffic to new version first |

---

**Rollback**

If a deploy goes wrong, rollback means switching back to the previous known-good version. With containers and image tags, this is straightforward — point back to the previous image tag and re-run the deploy stage.

---

**AWS CodePipeline**

AWS's native CI/CD orchestration service. Unlike GitHub Actions or Azure Pipelines where everything is one YAML file, AWS splits the pipeline across three separate services:

| Service | What it does |
|---|---|
| CodePipeline | Orchestrates the pipeline — connects stages together |
| CodeBuild | Runs the build/test steps (like a GitHub Actions job) |
| CodeDeploy | Handles deployment to EC2, ECS, Lambda, etc. |

You can also plug in GitHub or CodeCommit as the source, and ECR as the image registry.

---

**CodeBuild — buildspec.yml**

CodeBuild uses a `buildspec.yml` file to define what to run.

```yaml
# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci

  pre_build:
    commands:
      - echo Logging in to ECR
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY

  build:
    commands:
      - npm test
      - npm run build
      - docker build -t $ECR_REGISTRY/$IMAGE_NAME:$CODEBUILD_BUILD_NUMBER .
      - docker push $ECR_REGISTRY/$IMAGE_NAME:$CODEBUILD_BUILD_NUMBER

  post_build:
    commands:
      - echo Build complete

artifacts:
  files:
    - imagedefinitions.json   # used by CodeDeploy for ECS deployments
```

---

**CodePipeline — how it flows**

```
Source (GitHub / CodeCommit)
  ↓
Build (CodeBuild — runs buildspec.yml)
  ↓
Deploy Staging (CodeDeploy / ECS / Lambda)
  ↓
Manual Approval (optional gate)
  ↓
Deploy Production
```

Each stage passes artifacts (files, image tags) to the next stage through an S3 bucket managed by CodePipeline.

---

**ECR — Elastic Container Registry**

AWS's private Docker registry. Like Docker Hub but fully integrated with IAM and other AWS services. CodeBuild pushes images here; ECS/EKS pulls from here.

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Tag and push
docker tag my-app:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0
```

---

**AWS vs GitHub Actions vs Azure Pipelines — quick comparison**

| | GitHub Actions | Azure Pipelines | AWS CodePipeline |
|---|---|---|---|
| Config file | `.github/workflows/*.yml` | `azure-pipelines.yml` | Pipeline defined in console or CDK/Terraform |
| Build runner | GitHub-hosted runners | Microsoft-hosted agents | CodeBuild (buildspec.yml) |
| Secrets | GitHub Secrets | Variable Groups / Key Vault | AWS Secrets Manager / Parameter Store |
| Docker registry | GHCR / Docker Hub | ACR / Docker Hub | ECR |
| Best when | GitHub-hosted code | Microsoft/Azure stack | All-in on AWS |

---

## Step 6 — Cloud Fundamentals


### Concepts

**Regions and Availability Zones**

Cloud providers run data centers worldwide. A region is a geographic area (e.g., us-east-1). Within each region are multiple Availability Zones (AZs) — isolated data centers. Running across AZs makes apps resilient to single data center failures.

---

**Core service categories**

| Category | What it is | AWS example |
|---|---|---|
| Compute | Virtual servers | EC2, Lambda |
| Storage | File/object/block storage | S3, EBS |
| Database | Managed DB services | RDS, DynamoDB |
| Networking | VPCs, DNS, load balancers | VPC, Route53, ALB |
| Container | Run containers | ECS, EKS |
| IAM | Access control | IAM |

---

**Virtual Private Cloud (VPC)**

An isolated virtual network in the cloud. You control the IP ranges, subnets, routing, and security rules. Everything you run in AWS lives inside a VPC.

---

**Subnets**

A subdivision of a VPC. Public subnets have routes to the internet. Private subnets do not — they're for databases and internal services that shouldn't be directly exposed.

---

**Security Groups**

Virtual firewalls that control inbound and outbound traffic for your resources. Stateful — if you allow inbound traffic, the response is automatically allowed out.

```
Allow port 443 from anywhere → web traffic
Allow port 5432 from app-security-group only → database
```

---

**IAM — Identity and Access Management**

Controls who (user, service, application) can do what (action) on which resources. Always follow least privilege — grant only what is needed.

```
IAM User → developer with console access
IAM Role → assumed by an EC2 instance or Lambda to call other AWS services
IAM Policy → JSON document defining allowed/denied actions
```

---

**S3 — Object Storage**

Store and retrieve any file — images, build artifacts, static websites, logs. Files are objects in buckets. Globally accessible, highly durable.

```bash
aws s3 cp index.html s3://my-bucket/
aws s3 sync ./dist s3://my-bucket/
```

---

**Load Balancer**

Distributes incoming traffic across multiple instances. If one instance goes down, traffic routes to the healthy ones. Essential for high availability.

Types in AWS: ALB (Application, HTTP/S), NLB (Network, TCP), CLB (Classic, legacy).

---

**Auto Scaling**

Automatically adds or removes instances based on load. Define a minimum, maximum, and desired count. The cloud scales within those bounds based on CPU, memory, or custom metrics.

---

## Step 7 — Infrastructure as Code


### Concepts

**Why IaC**

Without IaC: you click through a console to create resources. It's not reproducible, not version-controlled, and a different person gets a different result.

With IaC: your entire infrastructure is a set of files. You review, version, test, and apply them like application code.

---

**Terraform**

The most widely used IaC tool. Declarative — you describe the desired end state, Terraform figures out how to get there.

Works across AWS, GCP, Azure, and hundreds of other providers.

---

**Core Terraform concepts**

**Provider**: The cloud platform you're managing (AWS, GCP, etc.)
**Resource**: A piece of infrastructure you declare (an S3 bucket, an EC2 instance)
**State**: Terraform keeps a state file tracking what currently exists so it knows what to create/update/destroy
**Plan**: Preview what changes will be applied before applying them

---

**Basic Terraform workflow**

```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "app_bucket" {
  bucket = "my-app-assets"
}

resource "aws_s3_bucket_public_access_block" "app_bucket" {
  bucket = aws_s3_bucket.app_bucket.id
  block_public_acls   = true
  block_public_policy = true
}
```

```bash
terraform init      # download providers
terraform plan      # preview changes
terraform apply     # apply changes
terraform destroy   # tear down all resources
```

---

**Variables**

```hcl
variable "environment" {
  type    = string
  default = "staging"
}

resource "aws_s3_bucket" "bucket" {
  bucket = "my-app-${var.environment}"
}
```

---

**Outputs**

Expose values after apply — useful for passing info to other tools or scripts.

```hcl
output "bucket_name" {
  value = aws_s3_bucket.bucket.id
}
```

---

**Remote state**

By default Terraform stores state locally. In teams, store it remotely (S3 + DynamoDB for locking) so everyone uses the same state.

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## Step 8 — Kubernetes


### Concepts

**What Kubernetes is**

Kubernetes (K8s) is a container orchestration platform. It manages running containers across a cluster of machines — handling scheduling, scaling, self-healing, networking, and rolling deployments.

Think of it as: you tell Kubernetes what you want running, and it makes it happen and keeps it that way.

---

**Cluster**

A Kubernetes cluster is a set of machines (nodes). There is a control plane (the brain) and worker nodes (where containers actually run).

---

**Pod**

The smallest deployable unit in Kubernetes. A pod wraps one or more containers that share network and storage. In practice, most pods contain one container.

Pods are ephemeral — they can die and be replaced. You never directly manage pods; higher-level objects manage them for you.

---

**Deployment**

A Deployment manages a set of identical pods. You tell it how many replicas you want, which image to use, and it handles creating, updating, and replacing pods.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: myrepo/my-app:1.0
          ports:
            - containerPort: 3000
```

---

**Service**

Pods get new IP addresses every time they restart. A Service gives your pods a stable network identity and load balances traffic across them.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer
```

Types: `ClusterIP` (internal only), `NodePort` (exposed on node IP), `LoadBalancer` (cloud load balancer).

---

**ConfigMap and Secret**

ConfigMap: store non-sensitive configuration (env vars, config files) outside the container image.
Secret: store sensitive values (passwords, tokens, API keys) — base64 encoded and access-controlled.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  API_URL: "https://api.example.com"
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0  # base64 encoded
```

---

**Ingress**

Routes external HTTP/S traffic to services inside the cluster based on host/path rules. Like a smart reverse proxy.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

---

**Namespace**

A way to logically isolate resources within a cluster. Common pattern: one namespace per environment (`dev`, `staging`, `prod`) or one per team.

---

**Essential kubectl commands**

```bash
# Cluster
kubectl cluster-info
kubectl get nodes

# Deployments and pods
kubectl apply -f deployment.yml     # create or update
kubectl get deployments
kubectl get pods
kubectl describe pod <name>         # detailed info
kubectl logs <pod-name>             # view logs
kubectl logs -f <pod-name>          # follow logs
kubectl exec -it <pod-name> -- bash # shell into pod

# Scaling
kubectl scale deployment my-app --replicas=5

# Rolling update
kubectl set image deployment/my-app my-app=myrepo/my-app:2.0

# Rollback
kubectl rollout undo deployment/my-app
kubectl rollout history deployment/my-app

# Delete
kubectl delete -f deployment.yml
```

---

**Namespace-scoped commands**

```bash
kubectl get pods -n production
kubectl apply -f app.yml -n staging
```

---

## Step 9 — Monitoring & Observability


### Concepts

**The 3 pillars of observability**

| Pillar | What it is | Example tools |
|---|---|---|
| Logs | Text records of events | ELK Stack, Loki |
| Metrics | Numbers over time | Prometheus, CloudWatch |
| Traces | Request flow across services | Jaeger, Zipkin |

Logs tell you what happened. Metrics tell you how the system is behaving. Traces tell you where time was spent.

---

**Metrics**

A metric is a numeric measurement sampled over time.

- CPU usage: 72%
- Request rate: 1,200 req/sec
- Error rate: 0.4%
- P99 latency: 340ms

You set alerts on metrics: "alert me if error rate > 1% for 5 minutes".

---

**Prometheus**

An open-source metrics collection system. It scrapes metrics from your apps and infrastructure on a schedule and stores them as time-series data.

Apps expose a `/metrics` endpoint. Prometheus pulls from it.

---

**Grafana**

A visualization layer. Connects to Prometheus (and many other sources) and lets you build dashboards with graphs, gauges, and alert panels.

Prometheus handles the data. Grafana handles the display.

---

**Alerting**

Rules that fire when a metric crosses a threshold. Alerts are sent to Slack, PagerDuty, email, etc.

```yaml
# Prometheus alert rule
- alert: HighErrorRate
  expr: rate(http_requests_total{status="500"}[5m]) > 0.01
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Error rate above 1%"
```

---

**Logs vs Metrics — when to use which**

Use metrics to detect that something is wrong (high error rate, slow responses).
Use logs to diagnose what went wrong (which specific requests failed, what the error message was).

---

**Structured logging**

Instead of plain text logs, log as JSON. Structured logs are searchable, filterable, and parseable by log aggregation tools.

```json
{
  "timestamp": "2024-01-15T10:23:00Z",
  "level": "error",
  "message": "Payment failed",
  "userId": "u_123",
  "requestId": "req_abc",
  "errorCode": "CARD_DECLINED"
}
```

---

**Health checks**

Endpoints your orchestrator (Kubernetes) or load balancer pings to know if your app is alive.

- Liveness probe: is the app running? (if not, restart it)
- Readiness probe: is the app ready to serve traffic? (if not, remove from load balancer)

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 30
```

---

## Step 10 — Security (DevSecOps)


### Concepts

**Shift left on security**

Security checks happen at every step of the pipeline — not as an afterthought before release. Developers are responsible for secure code, not just a separate security team.

---

**Secrets management**

Never store secrets in code or Docker images. Use:

- Environment variables injected at runtime
- Cloud secret stores: AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager
- Kubernetes Secrets (with RBAC + encryption at rest)

```bash
# Bad — secret in code
DB_PASSWORD="my-secret-password"

# Good — inject at runtime
export DB_PASSWORD=$(aws secretsmanager get-secret-value --secret-id db-password)
```

---

**Dependency scanning**

Your app's dependencies can have known vulnerabilities. Scan them automatically.

```bash
npm audit              # check for vulnerable packages
npm audit fix          # auto-fix where possible
```

Tools: Snyk, Dependabot (GitHub), OWASP Dependency-Check.

---

**Container image scanning**

Scan Docker images for known CVEs in base images and installed packages.

```bash
# Trivy — fast open-source scanner
trivy image my-app:1.0
```

Run this in your CI pipeline on every build. Fail the pipeline if critical vulnerabilities are found.

---

**SAST — Static Application Security Testing**

Analyze source code for security issues without running it.

Tools: SonarQube, Semgrep, CodeQL (built into GitHub).

---

**Least privilege**

Every service, user, or role should have only the permissions it needs — nothing more. If your app only reads from S3, its IAM role should only allow `s3:GetObject`, not `s3:*`.

---

**Network security**

- Expose only what needs to be public
- Put databases in private subnets
- Use security groups to allow only necessary ports
- Enable VPC flow logs to audit traffic

---

**HTTPS everywhere**

Never serve traffic over HTTP in production. Use TLS certificates.

- Let's Encrypt: free, automated cert issuance
- cert-manager: automates cert management in Kubernetes
- AWS ACM: manages certs for AWS services

---

## Glossary

| Term | Definition |
|------|------------|
| DevOps | Practice combining development and operations to automate and accelerate software delivery |
| CI | Continuous Integration — automatically build and test code on every push |
| CD | Continuous Delivery/Deployment — automatically prepare or deploy every passing build |
| Pipeline | Automated sequence of steps code goes through from commit to deployment |
| IaC | Infrastructure as Code — defining infrastructure in version-controlled files |
| Container | A lightweight, isolated package containing an app and all its dependencies |
| Image | Read-only blueprint for a container — built from a Dockerfile, stored in a registry |
| Dockerfile | Instructions for building a Docker image |
| Docker Compose | Tool for defining and running multi-container apps with a YAML file |
| Registry | Storage for Docker images (Docker Hub, ECR, GHCR) |
| Volume | Persistent storage that survives container restarts |
| Kubernetes | Container orchestration platform — manages running containers at scale |
| Pod | Smallest unit in Kubernetes — wraps one or more containers |
| Deployment | Kubernetes object that manages a set of replicated pods |
| Service | Gives pods a stable network identity and load balances traffic |
| Ingress | Routes external HTTP traffic to internal services |
| Namespace | Logical grouping of Kubernetes resources |
| Terraform | IaC tool for defining cloud infrastructure declaratively |
| VPC | Virtual Private Cloud — isolated network in the cloud |
| IAM | Identity and Access Management — controls who can do what on which resources |
| Least Privilege | Only granting the minimum permissions required |
| Prometheus | Open-source metrics collection and alerting system |
| Grafana | Visualization tool for metrics dashboards |
| Observability | The ability to understand what a system is doing from its outputs (logs, metrics, traces) |
| SAST | Static Application Security Testing — analyzing code for security issues without running it |
| CVE | Common Vulnerabilities and Exposures — publicly known security vulnerability |
| Shift Left | Moving testing and security checks earlier in the development process |
| Azure DevOps | Microsoft's full DevOps platform — includes Repos, Pipelines, Boards, Artifacts, and Test Plans |
| Azure Pipelines | The CI/CD component of Azure DevOps — pipelines defined in azure-pipelines.yml |
| Stage | A logical grouping of jobs in a pipeline — e.g. Build, Test, Deploy |
| Agent | The machine that runs pipeline jobs — either Microsoft-hosted or self-hosted |
| Service Connection | A secure link in Azure DevOps connecting a pipeline to an external service (Docker, AWS, K8s, etc.) |
| Variable Group | A shared set of variables/secrets in Azure DevOps that can be linked to multiple pipelines |
| Environment (Azure) | A named deployment target in Azure DevOps — can require manual approval before proceeding |
| AWS CodePipeline | AWS's pipeline orchestration service — connects source, build, and deploy stages |
| AWS CodeBuild | AWS's managed build service — runs steps defined in buildspec.yml |
| AWS CodeDeploy | AWS's deployment service — handles rollouts to EC2, ECS, Lambda |
| ECR | Elastic Container Registry — AWS's private Docker image registry |
| buildspec.yml | Configuration file for CodeBuild defining install, build, and post-build steps |
