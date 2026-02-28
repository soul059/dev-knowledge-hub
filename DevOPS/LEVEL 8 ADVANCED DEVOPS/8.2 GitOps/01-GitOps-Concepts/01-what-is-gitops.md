# Chapter 1.1: What is GitOps?

[← Back to README](../README.md) | [Next → GitOps Principles](02-gitops-principles.md)

---

## Overview

GitOps is an operational framework that applies DevOps best practices — such as version control, collaboration, compliance, and CI/CD tooling — to infrastructure automation. Coined by **Weaveworks** in 2017, GitOps uses **Git as the single source of truth** for declarative infrastructure and applications.

At its core, GitOps means:
> **The desired state of your system is stored in Git, and automated processes ensure the actual system matches that desired state.**

---

## The Evolution: From Traditional Ops to GitOps

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    EVOLUTION OF OPERATIONS                              │
│                                                                          │
│  Manual Ops        DevOps            CI/CD              GitOps           │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐       ┌─────────────┐     │
│  │ SSH into │     │ Scripts │     │ Pipeline│       │ Git Commit  │     │
│  │ servers  │ ──> │ & Tools │ ──> │ Deploy  │ ──>   │ = Deploy    │     │
│  │ manually │     │ Ansible │     │ Jenkins │       │ Auto-Sync   │     │
│  └─────────┘     └─────────┘     └─────────┘       └─────────────┘     │
│                                                                          │
│  • Error-prone    • Repeatable    • Automated        • Declarative      │
│  • No audit       • Some audit    • Push-based       • Pull-based       │
│  • No versioning  • Versioned     • Partial truth    • Git = truth      │
│  • Snowflakes     • Consistent    • Pipeline-centric • Reconciliation   │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## GitOps in a Nutshell

### The Core Idea

```
   ┌──────────────┐          ┌────────────────┐          ┌──────────────┐
   │  Developer   │  commit  │   Git Repo     │  detect  │  GitOps      │
   │  makes a     │────────> │  (Desired      │────────> │  Agent       │
   │  change      │          │   State)       │          │  (ArgoCD/    │
   └──────────────┘          └────────────────┘          │   Flux)      │
                                                          └──────┬───────┘
                                                                 │
                                                          apply  │ reconcile
                                                                 │
                                                          ┌──────▼───────┐
                                                          │  Kubernetes  │
                                                          │  Cluster     │
                                                          │  (Actual     │
                                                          │   State)     │
                                                          └──────────────┘
```

### How It Works — Step by Step

1. **Declare** — Define your infrastructure and apps as code (YAML, Helm charts, Kustomize overlays)
2. **Store** — Commit everything to a Git repository
3. **Detect** — A GitOps agent watches the Git repo for changes
4. **Compare** — The agent compares the desired state (Git) with the actual state (cluster)
5. **Reconcile** — The agent applies changes to make actual = desired
6. **Repeat** — The reconciliation loop runs continuously

---

## GitOps vs Traditional CI/CD

```
  TRADITIONAL CI/CD (Push)                    GITOPS (Pull)
  ━━━━━━━━━━━━━━━━━━━━━━                    ━━━━━━━━━━━━━━
  
  Developer ──> CI Pipeline ──> kubectl ──> Cluster
                    │
                    │  Pipeline PUSHES to cluster
                    │  Pipeline needs cluster creds
                    ▼
  
  Developer ──> Git Repo <── Agent ──> Cluster
                                │
                                │  Agent PULLS from Git
                                │  Agent runs IN the cluster
                                ▼
```

| Aspect | Traditional CI/CD | GitOps |
|--------|------------------|--------|
| Deploy trigger | Pipeline pushes | Agent pulls |
| Credentials | CI needs cluster creds | Agent has cluster access natively |
| Source of truth | Pipeline config | Git repository |
| Drift detection | None (fire and forget) | Continuous reconciliation |
| Rollback | Re-run old pipeline | `git revert` |
| Audit trail | CI logs | Git history |
| Security | Creds in CI system | No external access needed |

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **Desired State** | The state of the system as defined in Git (manifests, configs) |
| **Actual State** | The current live state of the system (what's running in the cluster) |
| **Reconciliation** | The process of making actual state match desired state |
| **Drift** | When actual state diverges from desired state |
| **Agent / Operator** | Software that watches Git and reconciles (ArgoCD, Flux) |
| **Declarative** | Describing WHAT you want, not HOW to achieve it |
| **Sync** | The act of applying desired state to the cluster |
| **Self-healing** | Automatically correcting drift without human intervention |

---

## Real-World Example

### Scenario: Deploying a Web Application

**Without GitOps:**
```bash
# Developer builds and pushes
docker build -t myapp:v2 .
docker push registry.example.com/myapp:v2

# Then SSHes or uses CI to deploy
kubectl set image deployment/myapp myapp=registry.example.com/myapp:v2

# No record of who changed what, when, or why
# If someone runs kubectl edit, the change is lost
```

**With GitOps:**
```yaml
# deployment.yaml in Git repo
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: registry.example.com/myapp:v2  # Changed from v1 to v2
```
```bash
# Developer makes a PR, gets review, merges
git add deployment.yaml
git commit -m "feat: upgrade myapp to v2 - fixes login bug #123"
git push

# ArgoCD/Flux detects the change and applies it automatically
# Full audit trail in Git — who, what, when, why
# If someone runs kubectl edit, the agent reverts the change
```

---

## When to Use GitOps

### Great Fit ✓
- Kubernetes-native workloads
- Microservices architectures
- Multi-environment deployments (dev/staging/prod)
- Teams that need strong audit trails
- Organizations with compliance requirements
- Infrastructure-as-Code workflows

### Less Ideal ✗
- Stateful, imperative workflows (database migrations)
- One-off scripts and ad-hoc tasks
- Non-declarative infrastructure
- Very small projects with a single developer

---

## The GitOps Ecosystem

```
┌─────────────────────────────────────────────────────────┐
│                   GITOPS ECOSYSTEM                      │
│                                                         │
│  ┌─── Git Providers ───┐  ┌─── GitOps Agents ───┐      │
│  │ • GitHub             │  │ • ArgoCD             │     │
│  │ • GitLab             │  │ • Flux CD            │     │
│  │ • Bitbucket          │  │ • Jenkins X          │     │
│  │ • Azure DevOps       │  │ • Rancher Fleet      │     │
│  └──────────────────────┘  └──────────────────────┘     │
│                                                         │
│  ┌─── Config Tools ────┐  ┌─── Secret Mgmt ─────┐      │
│  │ • Kustomize          │  │ • Sealed Secrets     │     │
│  │ • Helm               │  │ • SOPS               │     │
│  │ • Jsonnet            │  │ • Vault              │     │
│  │ • CUE                │  │ • External Secrets   │     │
│  └──────────────────────┘  └──────────────────────┘     │
│                                                         │
│  ┌─── Platforms ────────┐  ┌─── Observability ───┐      │
│  │ • Kubernetes          │  │ • Prometheus         │     │
│  │ • OpenShift           │  │ • Grafana            │     │
│  │ • EKS / AKS / GKE    │  │ • ArgoCD Dashboard   │     │
│  │ • k3s / kind          │  │ • Flux Notifications │     │
│  └──────────────────────┘  └──────────────────────┘     │
└─────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| GitOps Definition | Operational framework using Git as single source of truth |
| Origin | Coined by Weaveworks in 2017 |
| Core Mechanism | Reconciliation loop (desired state → actual state) |
| Deployment Model | Pull-based (agent pulls from Git) |
| Primary Tools | ArgoCD, Flux CD |
| Best For | Kubernetes, declarative infrastructure |
| Key Benefit | Auditability, reproducibility, self-healing |
| Rollback Method | `git revert` — no need to touch the cluster directly |

---

## Quick Revision Questions

1. **What is GitOps and who coined the term?**
   > GitOps is an operational framework that uses Git as the single source of truth for declarative infrastructure. It was coined by Weaveworks in 2017.

2. **How does GitOps differ from traditional CI/CD in terms of deployment flow?**
   > Traditional CI/CD uses a push model where pipelines push changes to the cluster. GitOps uses a pull model where an in-cluster agent pulls desired state from Git.

3. **What is the reconciliation loop in GitOps?**
   > It is the continuous process where a GitOps agent compares the desired state in Git with the actual state of the cluster and applies changes to eliminate drift.

4. **Why is `git revert` considered a rollback mechanism in GitOps?**
   > Because Git is the single source of truth, reverting a commit changes the desired state back, and the GitOps agent automatically reconciles the cluster to match.

5. **Name three scenarios where GitOps is a great fit and one where it is less ideal.**
   > Great: Kubernetes workloads, microservices, multi-env deployments. Less ideal: stateful imperative workflows like database migrations.

6. **What happens in a GitOps system if someone manually edits a resource with `kubectl edit`?**
   > The GitOps agent detects drift between the actual state and the desired state in Git, and reverts the manual change (self-healing).

---

[← Back to README](../README.md) | [Next → GitOps Principles](02-gitops-principles.md)
