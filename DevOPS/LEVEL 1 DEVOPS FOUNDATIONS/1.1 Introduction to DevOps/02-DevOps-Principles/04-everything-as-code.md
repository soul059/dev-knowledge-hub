# Chapter 2.4 — Everything as Code

[← Previous: Shift-Left Approach](03-shift-left-approach.md) | [Next: Fail Fast, Recover Fast →](05-fail-fast-recover-fast.md)

---

## Overview

**Everything as Code (EaC)** is the principle that all aspects of your system — infrastructure, configuration, policies, documentation, and pipelines — should be defined in version-controlled code rather than managed through manual processes or GUIs.

---

## The Concept

```
┌──────────────────────────────────────────────────────────┐
│              EVERYTHING AS CODE                          │
│                                                          │
│  Traditional (Click-Ops)           EaC (Code-Ops)        │
│  ═══════════════════════           ══════════════         │
│                                                          │
│  ┌─────────────────┐              ┌─────────────────┐   │
│  │  AWS Console    │              │  main.tf         │   │
│  │  [Click] [Click]│     ──►      │  resource "aws   │   │
│  │  [Configure]    │              │    _instance"... │   │
│  │  [Save]         │              │  }               │   │
│  └─────────────────┘              └─────────────────┘   │
│                                          │               │
│  - Not reproducible                      ▼               │
│  - No audit trail              ┌─────────────────┐      │
│  - Hard to review              │  Git Repository  │      │
│  - Cannot rollback             │  - Version ctrl  │      │
│  - Tribal knowledge            │  - PR reviews    │      │
│                                │  - Audit trail   │      │
│                                │  - Rollback      │      │
│                                └─────────────────┘      │
└──────────────────────────────────────────────────────────┘
```

---

## What Can Be "As Code"?

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Infrastructure as Code (IaC)                            │
│  ├── Servers, networks, databases                        │
│  ├── Tools: Terraform, CloudFormation, Pulumi            │
│  │                                                       │
│  Configuration as Code                                   │
│  ├── App settings, feature flags, env variables          │
│  ├── Tools: Ansible, Chef, Puppet                        │
│  │                                                       │
│  Pipeline as Code                                        │
│  ├── CI/CD pipeline definitions                          │
│  ├── Tools: Jenkinsfile, .github/workflows, .gitlab-ci   │
│  │                                                       │
│  Policy as Code                                          │
│  ├── Security rules, compliance checks                   │
│  ├── Tools: OPA/Rego, HashiCorp Sentinel                 │
│  │                                                       │
│  Security as Code                                        │
│  ├── RBAC definitions, network policies                  │
│  ├── Tools: Kubernetes RBAC YAML, AWS IAM JSON           │
│  │                                                       │
│  Documentation as Code                                   │
│  ├── API docs, architecture diagrams                     │
│  ├── Tools: Markdown, OpenAPI/Swagger, Mermaid           │
│  │                                                       │
│  Monitoring as Code                                      │
│  ├── Alerts, dashboards, SLOs                            │
│  ├── Tools: Grafana JSON, Prometheus rules               │
│  │                                                       │
│  Environment as Code                                     │
│  ├── Dev environments, dependencies                      │
│  └── Tools: Dockerfile, docker-compose, Devcontainers    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Examples of Everything as Code

### 1. Infrastructure as Code (Terraform)

```hcl
# infrastructure/main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name        = "production-vpc"
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

resource "aws_rds_instance" "database" {
  engine         = "postgres"
  engine_version = "16.1"
  instance_class = "db.t3.large"
  allocated_storage = 100
  
  backup_retention_period = 7
  multi_az                = true
}
```

### 2. Pipeline as Code (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Run tests
        run: docker run myapp:${{ github.sha }} npm test
      - name: Push to registry
        run: |
          docker tag myapp:${{ github.sha }} ecr.aws/myapp:${{ github.sha }}
          docker push ecr.aws/myapp:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=ecr.aws/myapp:${{ github.sha }}
          kubectl rollout status deployment/myapp
```

### 3. Policy as Code (OPA/Rego)

```rego
# policy/kubernetes.rego
package kubernetes.admission

# Deny containers running as root
deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  container.securityContext.runAsRoot == true
  msg := sprintf("Container '%v' must not run as root", [container.name])
}

# Require resource limits on all containers
deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not container.resources.limits
  msg := sprintf("Container '%v' must have resource limits", [container.name])
}
```

### 4. Monitoring as Code (Prometheus)

```yaml
# monitoring/alerts.yml
groups:
  - name: application
    rules:
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "P95 latency above 500ms"

      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 1%"
```

### 5. Environment as Code (Docker Compose)

```yaml
# docker-compose.yml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache

  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

  cache:
    image: redis:7-alpine

volumes:
  pgdata:
```

---

## Benefits of Everything as Code

```
┌──────────────────────────────────────────────────────────┐
│  WHY EVERYTHING AS CODE?                                 │
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐               │
│  │  REPRODUCIBLE   │  │  VERSIONABLE    │               │
│  │                 │  │                 │               │
│  │  Same code =    │  │  git log shows  │               │
│  │  same result    │  │  what changed,  │               │
│  │  every time     │  │  when, and why  │               │
│  └─────────────────┘  └─────────────────┘               │
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐               │
│  │  REVIEWABLE     │  │  TESTABLE       │               │
│  │                 │  │                 │               │
│  │  Pull requests  │  │  Validate infra │               │
│  │  for infra      │  │  before apply   │               │
│  │  changes        │  │  (terraform plan)│               │
│  └─────────────────┘  └─────────────────┘               │
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐               │
│  │  RECOVERABLE    │  │  SHAREABLE      │               │
│  │                 │  │                 │               │
│  │  git revert to  │  │  Reuse modules  │               │
│  │  rollback any   │  │  across teams   │               │
│  │  change         │  │  and projects   │               │
│  └─────────────────┘  └─────────────────┘               │
└──────────────────────────────────────────────────────────┘
```

---

## The GitOps Workflow

GitOps takes "Everything as Code" to its logical conclusion: **Git is the single source of truth for everything**.

```
┌──────────────────────────────────────────────────────────┐
│                    GITOPS WORKFLOW                        │
│                                                          │
│  Developer ──► Git Repo ──► GitOps Agent ──► Cluster     │
│                                                          │
│  1. Developer pushes         3. Agent detects            │
│     change to Git               Git change               │
│        │                           │                     │
│        ▼                           ▼                     │
│  2. PR reviewed &            4. Agent applies            │
│     merged                      change to                │
│                                 cluster                  │
│        │                           │                     │
│        ▼                           ▼                     │
│  ┌───────────┐              ┌───────────┐               │
│  │ Git Repo  │ ──────────── │ Kubernetes│               │
│  │ (desired  │   sync       │ (actual   │               │
│  │  state)   │ ◄──────────  │  state)   │               │
│  └───────────┘   reconcile  └───────────┘               │
│                                                          │
│  Tools: ArgoCD, Flux                                     │
│                                                          │
│  Key Principle: No manual kubectl commands               │
│  All changes go through Git                              │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: From Click-Ops to Code-Ops

### 🏢 Scenario: E-Commerce Platform Migration

```
BEFORE (Click-Ops):
├── 15 servers configured manually via SSH
├── App config stored in /etc/ on each server (different versions!)
├── Firewall rules managed via AWS Console
├── Monitoring alerts set up manually in CloudWatch
├── No one knows exact state of production
├── "Don't touch prod-server-7, only Dave knows how it works"
└── Disaster recovery: "Pray and rebuild from memory"

AFTER (Everything as Code):
├── infrastructure/     → Terraform (VPC, EC2, RDS, S3)
├── kubernetes/         → Helm charts (app deployments)
├── config/             → ConfigMaps in Git
├── .github/workflows/  → CI/CD pipeline definitions
├── monitoring/         → Prometheus rules + Grafana dashboards
├── policies/           → OPA policies for compliance
├── docs/               → Markdown + ADRs
└── Disaster recovery: "terraform apply" (15 minutes)
```

**Repository Structure:**
```
my-platform/
├── infrastructure/
│   ├── modules/
│   │   ├── vpc/
│   │   ├── rds/
│   │   └── eks/
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── main.tf
├── kubernetes/
│   ├── base/
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── kustomization.yaml
├── monitoring/
│   ├── alerts.yml
│   └── dashboards/
├── policies/
│   └── kubernetes.rego
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       └── infra.yml
└── README.md
```

---

## Troubleshooting Tips

| Issue | Problem | Solution |
|-------|---------|----------|
| **State drift** | Manual changes made outside of code | Use drift detection tools (Terraform plan, ArgoCD sync status) |
| **Secret management** | Secrets in code repos | Use Vault, AWS Secrets Manager, or sealed-secrets; NEVER commit secrets |
| **Complexity** | Too many YAML files | Use templating (Helm, Kustomize), modules (Terraform), and keep DRY |
| **Learning curve** | Team unfamiliar with IaC | Start with small projects, pair programming, internal training |
| **Blast radius** | One change breaks everything | Use environments (dev/staging/prod), plan before apply |

---

## Summary Table

| Category | What's Codified | Primary Tool |
|----------|----------------|-------------|
| **Infrastructure** | Servers, networks, databases | Terraform, Pulumi, CloudFormation |
| **Configuration** | App settings, env variables | Ansible, ConfigMaps, dotenv |
| **Pipelines** | Build, test, deploy workflows | GitHub Actions, GitLab CI, Jenkins |
| **Policies** | Security rules, compliance | OPA/Rego, Sentinel |
| **Monitoring** | Alerts, dashboards, SLOs | Prometheus rules, Grafana JSON |
| **Environments** | Dev setups, dependencies | Docker, docker-compose, Devcontainers |
| **Documentation** | API docs, architecture | OpenAPI, Markdown, Mermaid |

---

## Quick Revision Questions

1. **What does "Everything as Code" mean, and why is it important?**
   <details><summary>Answer</summary>EaC means defining all aspects of your system (infrastructure, configuration, pipelines, policies, monitoring) as version-controlled code files. It's important because it makes systems reproducible, versionable, reviewable, testable, recoverable, and shareable — eliminating manual processes and tribal knowledge.</details>

2. **What is the difference between Infrastructure as Code and Configuration Management?**
   <details><summary>Answer</summary>Infrastructure as Code (IaC) provisions and manages infrastructure resources (servers, networks, databases) — e.g., Terraform creates an EC2 instance. Configuration Management configures what runs ON those resources (software, settings, users) — e.g., Ansible installs Nginx and configures it.</details>

3. **What is GitOps, and how does it extend the "Everything as Code" principle?**
   <details><summary>Answer</summary>GitOps makes Git the single source of truth for system state. A GitOps agent (like ArgoCD) watches a Git repo and automatically reconciles the actual cluster state with the desired state in Git. All changes go through Git PRs — no manual kubectl or console changes allowed.</details>

4. **Why should secrets NEVER be stored in code repositories?**
   <details><summary>Answer</summary>Secrets in repos are exposed to anyone with repo access, persist in Git history even after deletion, and can be leaked if the repo is compromised or accidentally made public. Instead, use secrets management tools like HashiCorp Vault, AWS Secrets Manager, or Kubernetes sealed-secrets.</details>

5. **What is "state drift" and how do you prevent it?**
   <details><summary>Answer</summary>State drift occurs when the actual state of infrastructure diverges from what's defined in code — usually because someone made manual changes. Prevent it by: 1) Running regular drift detection (terraform plan). 2) Enforcing GitOps (no manual changes). 3) Setting up alerts for unauthorized changes. 4) Using read-only console access for production.</details>

6. **Give an example of Policy as Code and explain its benefit.**
   <details><summary>Answer</summary>Example: An OPA/Rego policy that denies Kubernetes pods running as root. Benefit: Compliance rules are codified, versioned, reviewed via PR, and automatically enforced in the pipeline — instead of relying on manual checklists or post-deployment audits.</details>

---

[← Previous: Shift-Left Approach](03-shift-left-approach.md) | [Next: Fail Fast, Recover Fast →](05-fail-fast-recover-fast.md)
