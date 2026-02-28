# Chapter 1.2: GitOps Principles

[← Previous: What is GitOps?](01-what-is-gitops.md) | [Back to README](../README.md) | [Next → Push vs Pull Model](03-push-vs-pull-model.md)

---

## Overview

GitOps is built on four foundational principles defined by the **OpenGitOps** project (a CNCF Sandbox project). These principles provide a clear, vendor-neutral definition of what makes a system "GitOps-compliant." Understanding these principles is essential to designing and evaluating GitOps implementations.

---

## The Four Principles of GitOps

```
┌─────────────────────────────────────────────────────────────────┐
│                  FOUR PRINCIPLES OF GITOPS                      │
│                                                                 │
│  ┌───────────────────┐       ┌───────────────────┐              │
│  │  1. DECLARATIVE   │       │  2. VERSIONED &   │              │
│  │                   │       │     IMMUTABLE      │              │
│  │  System described │       │  Desired state     │              │
│  │  declaratively    │       │  stored in Git     │              │
│  └───────────────────┘       └───────────────────┘              │
│                                                                 │
│  ┌───────────────────┐       ┌───────────────────┐              │
│  │  3. PULLED        │       │  4. CONTINUOUSLY   │              │
│  │     AUTOMATICALLY │       │     RECONCILED     │              │
│  │  Agents pull      │       │  Drift detected    │              │
│  │  desired state    │       │  and corrected     │              │
│  └───────────────────┘       └───────────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Principle 1: Declarative

> **The entire system must be described declaratively.**

### What Does Declarative Mean?

```
  IMPERATIVE (How)                    DECLARATIVE (What)
  ━━━━━━━━━━━━━━━━                    ━━━━━━━━━━━━━━━━━

  Step 1: Create namespace             "I want 3 replicas of
  Step 2: Create deployment              nginx running on
  Step 3: Set replicas to 3              port 80 in the
  Step 4: Expose on port 80              'production' namespace"
  Step 5: Scale up if needed

  ┌────────────────────┐              ┌────────────────────┐
  │  kubectl run       │              │  apiVersion: apps/v1│
  │  kubectl scale     │              │  kind: Deployment   │
  │  kubectl expose    │              │  spec:              │
  │  (sequence of      │              │    replicas: 3      │
  │   commands)        │              │    ...              │
  └────────────────────┘              └────────────────────┘

  Order matters!                       Order doesn't matter!
  Hard to reproduce                    Always reproducible
```

### Declarative Configuration Example

```yaml
# This YAML is declarative — it describes the DESIRED end state
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-frontend
  namespace: production
  labels:
    app: web-frontend
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-frontend
  template:
    metadata:
      labels:
        app: web-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: web-frontend
  namespace: production
spec:
  selector:
    app: web-frontend
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### Why Declarative Matters for GitOps

- **Idempotent** — Applying the same config multiple times produces the same result
- **Diffable** — You can see exactly what changed between two commits
- **Mergeable** — Teams can work on different parts and merge without conflicts
- **Auditable** — The entire desired state is visible at any point in history

---

## Principle 2: Versioned and Immutable

> **The desired state is stored in a way that enforces immutability, versioning, and retains a complete version history.**

### Git as the Version Store

```
┌─────────────────────────────────────────────────────────┐
│                   GIT VERSION HISTORY                   │
│                                                         │
│  commit  a1b2c3d  "feat: add web-frontend v2"          │
│    │                                                    │
│  commit  e4f5g6h  "fix: increase replicas to 5"        │
│    │                                                    │
│  commit  i7j8k9l  "refactor: move to namespace prod"   │
│    │                                                    │
│  commit  m0n1o2p  "feat: initial web-frontend deploy"  │
│    │                                                    │
│    ▼                                                    │
│  Every change is:                                       │
│  ✓ Timestamped                                          │
│  ✓ Attributed to an author                              │
│  ✓ Linked to a message/PR                               │
│  ✓ Immutable (can't silently change history)            │
│  ✓ Reversible (git revert)                              │
└─────────────────────────────────────────────────────────┘
```

### Key Properties

| Property | How Git Provides It |
|----------|-------------------|
| **Versioned** | Every commit is a snapshot of the entire state |
| **Immutable** | Commit hashes are cryptographic; tampering is detectable |
| **Auditable** | `git log` shows who changed what, when, and why |
| **Reversible** | `git revert <commit>` undoes any change cleanly |
| **Branching** | Parallel development with feature branches and PRs |

### Real-World: Tracing a Production Issue

```bash
# Who changed the replica count last week?
git log --since="1 week ago" -- k8s/production/deployment.yaml

# What exactly changed?
git diff HEAD~3 HEAD -- k8s/production/deployment.yaml

# Roll back to a known good state
git revert abc1234
git push origin main
# ArgoCD/Flux automatically applies the reverted state
```

---

## Principle 3: Pulled Automatically

> **Software agents automatically pull the desired state declarations from the source.**

### Pull vs Push — Agent Behavior

```
┌────────────────────────────────────────────────────────────────┐
│                  PULL-BASED RECONCILIATION                     │
│                                                                │
│   ┌──────────┐                                                 │
│   │ Git Repo │◄──── Poll / Webhook ────┐                       │
│   │ (Source)  │                         │                       │
│   └──────────┘                         │                       │
│                                   ┌────┴─────┐                 │
│                                   │  GitOps  │                 │
│                                   │  Agent   │                 │
│                                   │  (runs   │                 │
│                                   │  inside  │                 │
│                                   │  cluster)│                 │
│                                   └────┬─────┘                 │
│                                        │                       │
│                                   Apply│changes                │
│                                        ▼                       │
│                                   ┌──────────┐                 │
│                                   │  K8s     │                 │
│                                   │  Cluster │                 │
│                                   └──────────┘                 │
│                                                                │
│   Key: Agent PULLS from Git — no external push needed          │
│   The cluster is never directly accessed from outside           │
└────────────────────────────────────────────────────────────────┘
```

### Detection Methods

| Method | How It Works | Latency |
|--------|-------------|---------|
| **Polling** | Agent checks Git repo every N seconds/minutes | Configurable (e.g., 3 min) |
| **Webhook** | Git provider notifies agent on push | Near-instant |
| **Hybrid** | Webhook for speed + polling as fallback | Near-instant + resilient |

### ArgoCD Polling Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Poll every 60 seconds (default is 180s)
  timeout.reconciliation: "60"
```

### Flux Webhook Setup

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1
kind: Receiver
metadata:
  name: github-receiver
  namespace: flux-system
spec:
  type: github
  secretRef:
    name: webhook-token
  resources:
  - apiVersion: source.toolkit.fluxcd.io/v1
    kind: GitRepository
    name: my-app
```

---

## Principle 4: Continuously Reconciled

> **Software agents continuously observe the actual system state and attempt to apply the desired state.**

### The Reconciliation Loop

```
          ┌──────────────────────────────────────┐
          │       CONTINUOUS RECONCILIATION       │
          │                                      │
          │    ┌─────────┐    ┌──────────┐       │
          │    │ Desired  │    │ Actual   │       │
          │    │ State    │    │ State    │       │
          │    │ (Git)    │    │ (Cluster)│       │
          │    └────┬─────┘    └─────┬────┘       │
          │         │                │            │
          │         └───────┬────────┘            │
          │                 │                     │
          │           ┌─────▼─────┐               │
          │           │  COMPARE  │               │
          │           └─────┬─────┘               │
          │                 │                     │
          │        ┌────────┼────────┐            │
          │        │                 │            │
          │   ┌────▼────┐      ┌────▼────┐       │
          │   │  MATCH  │      │  DRIFT  │       │
          │   │  (Synced│      │(Out of  │       │
          │   │   ✓)    │      │  Sync ✗)│       │
          │   └─────────┘      └────┬────┘       │
          │                         │            │
          │                    ┌────▼────┐       │
          │                    │  APPLY  │       │
          │                    │ Changes │       │
          │                    └────┬────┘       │
          │                         │            │
          │              ┌──────────▼──────┐     │
          │              │ Wait & Repeat   │     │
          │              │ (loop forever)  │     │
          │              └─────────────────┘     │
          └──────────────────────────────────────┘
```

### What Reconciliation Catches

| Drift Type | Example | How Agent Fixes It |
|-----------|---------|-------------------|
| Manual edit | `kubectl edit deployment` changes replicas | Reverts replicas to Git value |
| Accidental delete | Someone deletes a ConfigMap | Recreates the ConfigMap |
| Partial failure | Deployment rolls out but Service fails | Re-applies the Service |
| External change | Admission controller modifies a resource | Compares and corrects |

---

## The OpenGitOps Project

The **OpenGitOps** project (under CNCF) formalizes these principles:

```
┌──────────────────────────────────────────────────────┐
│               OPENGITOPS (CNCF)                      │
│                                                      │
│  Mission: Vendor-neutral, principle-based            │
│           definition of GitOps                       │
│                                                      │
│  Deliverables:                                       │
│  ├── GitOps Principles v1.0                          │
│  ├── Glossary of Terms                               │
│  └── Compliance criteria                             │
│                                                      │
│  Website: opengitops.dev                             │
│  GitHub:  github.com/open-gitops                     │
└──────────────────────────────────────────────────────┘
```

---

## Applying the Principles — Checklist

Use this checklist to evaluate whether a workflow is truly GitOps:

```
  ☐ Is the system state described declaratively?
  ☐ Is the desired state stored in Git?
  ☐ Is Git the ONLY way to make changes? (no manual kubectl)
  ☐ Is there an agent pulling from Git automatically?
  ☐ Does the agent run continuously (not one-shot)?
  ☐ Does the agent detect and correct drift?
  ☐ Is the full history of changes available in Git?
  ☐ Can you roll back by reverting a Git commit?
```

---

## Summary Table

| Principle | Description | Key Requirement |
|-----------|-------------|-----------------|
| **Declarative** | System described as desired state | YAML/Helm/Kustomize — no imperative scripts |
| **Versioned & Immutable** | State stored with full history | Git repo with commit history |
| **Pulled Automatically** | Agent pulls from source | In-cluster agent (ArgoCD/Flux) |
| **Continuously Reconciled** | Drift detected and corrected | Reconciliation loop runs forever |

---

## Quick Revision Questions

1. **What are the four principles of GitOps as defined by OpenGitOps?**
   > Declarative, Versioned & Immutable, Pulled Automatically, Continuously Reconciled.

2. **Why is declarative configuration essential for GitOps?**
   > It's idempotent, diffable, mergeable, and auditable. It describes WHAT you want, not HOW, making it safe to apply repeatedly and compare across versions.

3. **How does Git enforce immutability in GitOps?**
   > Git commit hashes are cryptographic (SHA-1/SHA-256), making any tampering with history detectable. Each commit is an immutable snapshot.

4. **What is the difference between polling and webhook detection?**
   > Polling: agent checks Git every N seconds (higher latency, simpler). Webhook: Git provider notifies agent on push (near-instant, requires setup).

5. **What happens during continuous reconciliation when drift is detected?**
   > The agent compares desired state (Git) with actual state (cluster), detects differences, and applies changes to bring the cluster back in line with Git.

6. **Give an example of a workflow that violates GitOps principles.**
   > A CI pipeline that runs `kubectl apply` directly (push model), has no agent watching Git, and doesn't detect or correct manual changes.

---

[← Previous: What is GitOps?](01-what-is-gitops.md) | [Back to README](../README.md) | [Next → Push vs Pull Model](03-push-vs-pull-model.md)
