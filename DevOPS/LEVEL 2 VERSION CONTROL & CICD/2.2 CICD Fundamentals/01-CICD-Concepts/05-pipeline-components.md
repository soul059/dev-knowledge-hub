# Chapter 1.5: CI/CD Pipeline Components

[← Previous: CI/CD Benefits](04-cicd-benefits.md) | [Back to README](../README.md) | [Next: Pipeline Stages →](../02-Pipeline-Design/01-pipeline-stages.md)

---

## Overview

A CI/CD pipeline is composed of several interconnected components that work together to move code from a developer's machine to production. Understanding each component is essential for designing effective pipelines.

---

## Pipeline Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE COMPONENTS                             │
│                                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  SOURCE  │  │  BUILD   │  │   TEST   │  │ ARTIFACT │  │  DEPLOY  │     │
│  │ CONTROL  │─▶│  SYSTEM  │─▶│  RUNNER  │─▶│  STORE   │─▶│  ENGINE  │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │              │             │              │              │           │
│  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐     │
│  │ TRIGGERS │  │  AGENTS  │  │ QUALITY  │  │ REGISTRY │  │  ENVIRO- │     │
│  │ & EVENTS │  │ /RUNNERS │  │  GATES   │  │          │  │  NMENTS  │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │              │             │              │              │           │
│  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐     │
│  │  NOTIFI- │  │  CACHE   │  │ SECURITY │  │  CONFIG  │  │ MONITOR- │     │
│  │  CATIONS │  │          │  │ SCANNING │  │  MGMT    │  │   ING    │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Source Control Management (SCM)

The **origin** of every pipeline — where code lives and changes are tracked.

```
┌──────────────────────────────────────────────┐
│           SOURCE CONTROL (Git)                │
│                                               │
│  Repository                                   │
│  ├── main (production-ready)                  │
│  ├── develop (integration branch)             │
│  ├── feature/* (feature branches)             │
│  └── release/* (release branches)             │
│                                               │
│  Events that trigger CI/CD:                   │
│  • Push to branch                             │
│  • Pull Request created/updated               │
│  • Tag created (v1.0.0)                       │
│  • Merge to main                              │
└──────────────────────────────────────────────┘
```

**Common SCM Platforms:**
| Platform | CI/CD Integration |
|---|---|
| GitHub | GitHub Actions (native) |
| GitLab | GitLab CI/CD (native) |
| Bitbucket | Bitbucket Pipelines (native) |
| Azure Repos | Azure Pipelines |

---

## 2. CI/CD Server / Orchestrator

The **brain** that coordinates pipeline execution.

```
┌───────────────────────────────────────────────────────────────┐
│                CI/CD SERVER                                    │
│                                                                │
│  Responsibilities:                                             │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐     │
│  │ Listen for     │ │ Parse pipeline │ │ Dispatch jobs  │     │
│  │ triggers       │ │ definition     │ │ to runners     │     │
│  └───────┬────────┘ └───────┬────────┘ └───────┬────────┘     │
│          │                  │                   │              │
│  ┌───────┴────────┐ ┌──────┴─────────┐ ┌──────┴─────────┐   │
│  │ Manage secrets │ │ Track status   │ │ Report results │   │
│  │ & credentials  │ │ & logs         │ │ & notify       │   │
│  └────────────────┘ └────────────────┘ └────────────────┘   │
└───────────────────────────────────────────────────────────────┘
```

**Popular CI/CD Servers:**

| Tool | Type | Key Feature |
|---|---|---|
| Jenkins | Self-hosted | Highly extensible via plugins |
| GitHub Actions | Cloud/Self-hosted | Deep GitHub integration |
| GitLab CI | Cloud/Self-hosted | Complete DevOps platform |
| CircleCI | Cloud/Self-hosted | Fast, container-first |
| Azure Pipelines | Cloud/Self-hosted | Microsoft ecosystem |
| ArgoCD | Self-hosted | GitOps for Kubernetes |

---

## 3. Build Agents / Runners

The **workers** that execute pipeline jobs.

```
┌────────────────────────────────────────────────────────────────┐
│              CI/CD SERVER (Orchestrator)                        │
│                     │                                          │
│        ┌────────────┼────────────┐                             │
│        │            │            │                             │
│        ▼            ▼            ▼                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                      │
│  │ Runner 1 │ │ Runner 2 │ │ Runner 3 │                      │
│  │ (Linux)  │ │ (Windows)│ │ (macOS)  │                      │
│  │          │ │          │ │          │                      │
│  │ Job: A   │ │ Job: B   │ │ Job: C   │                      │
│  └──────────┘ └──────────┘ └──────────┘                      │
│                                                                │
│  Runner Types:                                                 │
│  • Cloud-hosted (managed by CI provider)                       │
│  • Self-hosted  (managed by your team)                         │
│  • Container    (ephemeral Docker containers)                  │
│  • VM-based     (full virtual machine per job)                 │
└────────────────────────────────────────────────────────────────┘
```

| Runner Type | Pros | Cons |
|---|---|---|
| Cloud-hosted | Zero maintenance, scalable | Limited customization, cost |
| Self-hosted | Full control, custom software | Maintenance burden |
| Container | Consistent environments | Build image overhead |

---

## 4. Pipeline Definition (Pipeline as Code)

The **blueprint** that defines what the pipeline does.

```yaml
# Generic pipeline structure (YAML-based)
pipeline:
  trigger:               # WHEN to run
    - push to main
    - pull request

  stages:                # WHAT to do
    - name: build
      steps:
        - checkout code
        - install dependencies
        - compile source
        - create artifact

    - name: test
      steps:
        - run unit tests
        - run integration tests
        - generate coverage report

    - name: deploy
      steps:
        - deploy to staging
        - run smoke tests
        - deploy to production
```

**Pipeline Definition Files by Tool:**

| Tool | File | Format |
|---|---|---|
| Jenkins | `Jenkinsfile` | Groovy |
| GitHub Actions | `.github/workflows/*.yml` | YAML |
| GitLab CI | `.gitlab-ci.yml` | YAML |
| CircleCI | `.circleci/config.yml` | YAML |
| Azure Pipelines | `azure-pipelines.yml` | YAML |
| Travis CI | `.travis.yml` | YAML |

---

## 5. Artifact Repository

Where **build outputs** (artifacts) are stored and versioned.

```
┌──────────────────────────────────────────────────────────┐
│                ARTIFACT REPOSITORY                        │
│                                                           │
│  Build Pipeline                                           │
│       │                                                   │
│       ▼                                                   │
│  ┌─────────┐     ┌─────────────────────────────────────┐ │
│  │ Artifact│────▶│  Artifact Store                      │ │
│  │ Created │     │                                      │ │
│  └─────────┘     │  myapp-1.0.0.jar                    │ │
│                  │  myapp-1.0.1.jar                    │ │
│                  │  myapp-1.1.0.jar  ← current staging │ │
│                  │  myapp-1.2.0.jar  ← current prod    │ │
│                  └─────────────┬───────────────────────┘ │
│                                │                          │
│                    ┌───────────┼───────────┐              │
│                    ▼           ▼           ▼              │
│                ┌──────┐  ┌────────┐  ┌────────┐          │
│                │ Dev  │  │Staging │  │  Prod  │          │
│                └──────┘  └────────┘  └────────┘          │
└──────────────────────────────────────────────────────────┘
```

**Common Artifact Repositories:**

| Repository | Artifact Type |
|---|---|
| Docker Hub / ECR / GCR | Container images |
| JFrog Artifactory | Universal (JAR, npm, Docker, etc.) |
| Nexus Repository | Maven, npm, Docker, PyPI |
| npm Registry | Node.js packages |
| PyPI | Python packages |
| GitHub Packages | Multi-format (npm, Maven, Docker) |
| AWS S3 | Generic binary storage |

---

## 6. Environment Management

The **target destinations** for deployments.

```
┌───────────────────────────────────────────────────────────────┐
│              ENVIRONMENT PROMOTION PATH                        │
│                                                                │
│  ┌─────┐    ┌─────────┐    ┌─────────┐    ┌────────────┐     │
│  │ DEV │───▶│   QA    │───▶│ STAGING │───▶│ PRODUCTION │     │
│  └─────┘    └─────────┘    └─────────┘    └────────────┘     │
│                                                                │
│  Purpose:    Feature      Full         Pre-prod      Live     │
│              development  testing      validation    users    │
│                                                                │
│  Data:       Mock/seed    Test data    Prod-like     Real     │
│  Scale:      Minimal      Small        Prod-like     Full     │
│  Access:     Developers   Dev + QA     Team + PM     All      │
│  Deploy:     Auto on PR   Auto on      Auto on       Manual / │
│              merge        merge        merge         Auto      │
└───────────────────────────────────────────────────────────────┘
```

---

## 7. Secrets & Configuration Management

Securely managing credentials and environment-specific settings.

```
┌──────────────────────────────────────────────────────────┐
│            SECRETS MANAGEMENT                             │
│                                                           │
│  ┌──────────────┐                                        │
│  │ Secret Store │                                        │
│  │              │                                        │
│  │ DB_PASSWORD  │──┐                                     │
│  │ API_KEY      │  │  Injected at runtime                │
│  │ TLS_CERT     │  │  (never in source code)             │
│  └──────────────┘  │                                     │
│                    ▼                                     │
│  Pipeline: env.DB_PASSWORD → Application                 │
│                                                           │
│  Tools:                                                   │
│  • HashiCorp Vault    • Azure Key Vault                  │
│  • AWS Secrets Manager • GitHub Secrets                   │
│  • GitLab CI Variables • K8s Secrets                     │
└──────────────────────────────────────────────────────────┘
```

---

## 8. Notifications & Feedback

Keeping the team informed of pipeline results.

```
┌─────────────────────────────────────────────────────────┐
│              NOTIFICATION CHANNELS                       │
│                                                          │
│  Pipeline Result ──┬── ✅ Success ──▶ Slack #deploys    │
│                    │                                     │
│                    ├── ❌ Failure ──▶ Email to committer │
│                    │               ──▶ Slack #alerts     │
│                    │               ──▶ PagerDuty         │
│                    │                                     │
│                    └── ⏳ Pending ──▶ PR status check    │
│                                                          │
│  Integration Options:                                    │
│  • Slack / Microsoft Teams                               │
│  • Email                                                 │
│  • PR status checks / commit status                      │
│  • Dashboard (Grafana, DataDog)                          │
│  • PagerDuty / OpsGenie (for prod failures)              │
└─────────────────────────────────────────────────────────┘
```

---

## 9. Monitoring & Observability

Post-deployment validation and ongoing health monitoring.

```
┌──────────────────────────────────────────────────────────────┐
│           POST-DEPLOYMENT MONITORING                          │
│                                                               │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  │
│  │    METRICS     │  │     LOGS       │  │    TRACES      │  │
│  │                │  │                │  │                │  │
│  │ • Error rate   │  │ • App logs     │  │ • Request flow │  │
│  │ • Latency      │  │ • Access logs  │  │ • Service map  │  │
│  │ • Throughput   │  │ • Audit logs   │  │ • Bottlenecks  │  │
│  │ • CPU/Memory   │  │                │  │                │  │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘  │
│          │                   │                    │           │
│          └───────────────────┼────────────────────┘           │
│                              │                                │
│                    ┌─────────▼─────────┐                     │
│                    │   ALERTING        │                     │
│                    │   Auto-rollback?  │                     │
│                    └──────────────────┘                     │
│                                                               │
│  Tools: Prometheus, Grafana, Datadog, ELK, Jaeger            │
└──────────────────────────────────────────────────────────────┘
```

---

## Complete Pipeline — Component Interaction

```
┌──────────────────────────────────────────────────────────────────────┐
│                COMPLETE CI/CD COMPONENT MAP                          │
│                                                                       │
│    Developer                                                          │
│       │                                                               │
│       ▼                                                               │
│  ┌─────────┐  webhook   ┌──────────┐  dispatch  ┌──────────┐        │
│  │   Git   │──────────▶│ CI/CD    │──────────▶│  Runner  │        │
│  │  (SCM)  │           │ Server   │           │  (Agent) │        │
│  └─────────┘           └────┬─────┘           └────┬─────┘        │
│                              │                      │              │
│                         ┌────┴────┐            ┌────┴────┐        │
│                         │Secrets  │            │  Cache  │        │
│                         │Manager  │            │         │        │
│                         └─────────┘            └─────────┘        │
│                                                     │              │
│                                                     ▼              │
│                                               ┌──────────┐        │
│                                               │ Artifact │        │
│                                               │  Store   │        │
│                                               └────┬─────┘        │
│                                                    │              │
│                          ┌─────────────────────────┼──────┐       │
│                          ▼              ▼          ▼       │       │
│                     ┌────────┐   ┌──────────┐ ┌────────┐  │       │
│                     │  Dev   │   │ Staging  │ │  Prod  │  │       │
│                     └────────┘   └──────────┘ └───┬────┘  │       │
│                                                    │       │       │
│                                               ┌────┴────┐ │       │
│  ┌──────────┐                                 │Monitor- │ │       │
│  │ Notify   │◀────────────────────────────────│  ing    │ │       │
│  │ (Slack)  │                                 └─────────┘ │       │
│  └──────────┘                                             │       │
└───────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Pipeline Components

| Component | Common Issue | Fix |
|---|---|---|
| SCM webhook | Pipeline doesn't trigger | Verify webhook URL, check firewall rules |
| Runner/Agent | Job stuck in queue | Scale runners, check runner health |
| Artifact store | Out of space | Retention policies, clean old artifacts |
| Secrets | "Secret not found" | Check scope (repo/org), variable names |
| Environment | Deploy fails in staging | Check IAM roles, network access, env vars |
| Notifications | No alerts on failure | Verify integration config, check channel |

---

## Summary Table

| Component | Purpose | Examples |
|---|---|---|
| **Source Control** | Code hosting, change tracking | GitHub, GitLab, Bitbucket |
| **CI/CD Server** | Pipeline orchestration | Jenkins, GitHub Actions, GitLab CI |
| **Runners/Agents** | Execute pipeline jobs | Cloud-hosted, self-hosted, containers |
| **Pipeline Definition** | Pipeline as Code | Jenkinsfile, `.github/workflows/`, `.gitlab-ci.yml` |
| **Artifact Store** | Store build outputs | Artifactory, Nexus, Docker registries |
| **Environments** | Deployment targets | Dev, QA, Staging, Production |
| **Secrets Management** | Credential security | Vault, AWS Secrets Manager |
| **Notifications** | Feedback to team | Slack, email, PR checks |
| **Monitoring** | Post-deploy validation | Prometheus, Grafana, Datadog |

---

## Quick Revision Questions

1. **What is the role of the CI/CD server (orchestrator)?**  
   *It listens for triggers, parses pipeline definitions, dispatches jobs to runners, manages secrets, tracks status, and reports results.*

2. **What is the difference between cloud-hosted and self-hosted runners?**  
   *Cloud-hosted runners are managed by the CI provider (zero maintenance, limited customization); self-hosted runners are managed by your team (full control, maintenance burden).*

3. **Why should you "build artifacts once" instead of per environment?**  
   *To ensure the exact same binary is tested and deployed everywhere. Environment-specific config is injected at deploy time, not build time.*

4. **Name three common secret management tools used in CI/CD.**  
   *HashiCorp Vault, AWS Secrets Manager, Azure Key Vault (also: GitHub Secrets, GitLab CI Variables).*

5. **What are the three pillars of observability for monitoring deployments?**  
   *Metrics (error rate, latency), Logs (application/access logs), and Traces (distributed request tracing).*

6. **What does a typical environment promotion path look like?**  
   *Dev → QA → Staging → Production, with increasing data realism and testing rigor at each stage.*

---

[← Previous: CI/CD Benefits](04-cicd-benefits.md) | [Back to README](../README.md) | [Next: Pipeline Stages →](../02-Pipeline-Design/01-pipeline-stages.md)
