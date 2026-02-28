# Chapter 1.4: Desired State vs Actual State

[← Previous: Push vs Pull Model](03-push-vs-pull-model.md) | [Back to README](../README.md) | [Next → GitOps Benefits](05-gitops-benefits.md)

---

## Overview

The concepts of **desired state** and **actual state** are the pillars of GitOps. Understanding the relationship between these two states — and how a GitOps agent bridges the gap — is essential to operating a GitOps system effectively.

---

## Defining the Two States

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   DESIRED STATE                    ACTUAL STATE              │
│   (What you WANT)                  (What you HAVE)           │
│                                                              │
│   ┌────────────────┐              ┌────────────────┐         │
│   │  Git Repo      │              │  Kubernetes    │         │
│   │                │              │  Cluster       │         │
│   │  deployment:   │              │                │         │
│   │    replicas: 3 │              │  Pods: 2       │         │
│   │    image: v2   │              │  Image: v1     │         │
│   │    cpu: 500m   │              │  CPU: 250m     │         │
│   │                │              │                │         │
│   └────────────────┘              └────────────────┘         │
│          │                               │                   │
│          └──────────┐  ┌─────────────────┘                   │
│                     ▼  ▼                                     │
│               ┌─────────────┐                                │
│               │   COMPARE   │                                │
│               │             │                                │
│               │ Desired ≠   │                                │
│               │ Actual      │                                │
│               │ = DRIFT!    │                                │
│               └──────┬──────┘                                │
│                      │                                       │
│                      ▼                                       │
│               ┌─────────────┐                                │
│               │  RECONCILE  │                                │
│               │  Apply Δ    │                                │
│               └─────────────┘                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Desired State

The **desired state** is the configuration you define in your Git repository. It answers the question: *"What should the system look like?"*

```yaml
# Desired State — stored in Git
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
  namespace: production
spec:
  replicas: 5               # I want 5 replicas
  template:
    spec:
      containers:
      - name: payment
        image: payments:v3.2  # I want version 3.2
        resources:
          requests:
            cpu: "500m"       # I want 500m CPU allocated
            memory: "512Mi"   # I want 512Mi memory
```

### Actual State

The **actual state** is what is currently running in the cluster. It's what Kubernetes reports when you query the API.

```bash
# Check actual state
$ kubectl get deployment payment-service -n production -o yaml

# Output shows what's ACTUALLY running:
# spec.replicas: 3          (not 5!)
# image: payments:v3.1      (not v3.2!)
# cpu request: 250m         (not 500m!)
```

---

## State Comparison: The Diff

```
┌─────────────────────────────────────────────────────────────┐
│                     STATE COMPARISON                        │
│                                                             │
│  Field          Desired (Git)     Actual (Cluster)   Match? │
│  ─────────────  ───────────────   ────────────────   ────── │
│  replicas       5                 3                  ✗ DRIFT│
│  image          payments:v3.2     payments:v3.1      ✗ DRIFT│
│  cpu request    500m              250m               ✗ DRIFT│
│  memory request 512Mi             512Mi              ✓ OK   │
│  namespace      production        production         ✓ OK   │
│  service type   ClusterIP         ClusterIP          ✓ OK   │
│                                                             │
│  Status: OUT OF SYNC — 3 fields drifted                     │
│  Action: Agent will apply changes to fix drift              │
└─────────────────────────────────────────────────────────────┘
```

---

## How Drift Occurs

Drift is when the actual state diverges from the desired state. It can happen in many ways:

```
┌──────────────────────────────────────────────────────────────┐
│                    CAUSES OF DRIFT                           │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 1. Manual Changes   │  kubectl edit, kubectl scale,       │
│  │    (most common)    │  kubectl patch — someone bypasses   │
│  └─────────────────────┘  Git and edits the cluster directly │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 2. Failed Rollouts  │  Deployment starts but crashes,     │
│  │                     │  leaving a partial state            │
│  └─────────────────────┘                                     │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 3. Auto-scaling     │  HPA changes replica count, which   │
│  │                     │  differs from Git-defined replicas  │
│  └─────────────────────┘                                     │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 4. Admission        │  Webhooks inject sidecars, labels,  │
│  │    Controllers      │  or annotations not in Git          │
│  └─────────────────────┘                                     │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 5. External         │  CRD controllers or operators       │
│  │    Controllers      │  modify resources dynamically       │
│  └─────────────────────┘                                     │
│                                                              │
│  ┌─────────────────────┐                                     │
│  │ 6. Resource         │  Someone deletes a resource that    │
│  │    Deletion         │  should exist according to Git      │
│  └─────────────────────┘                                     │
└──────────────────────────────────────────────────────────────┘
```

---

## The Reconciliation Process — In Detail

### Step-by-Step

```
 ┌────────────────────────────────────────────────────────────────┐
 │              RECONCILIATION FLOW (Detailed)                   │
 │                                                               │
 │  Step 1: FETCH desired state from Git                         │
 │  ┌─────────┐     ┌──────────┐                                 │
 │  │  Agent  │────>│ Git Repo │  Clone/pull latest manifests    │
 │  └────┬────┘     └──────────┘                                 │
 │       │                                                       │
 │  Step 2: RENDER templates (if Helm/Kustomize)                 │
 │  ┌────▼────┐                                                  │
 │  │ Render  │  helm template / kustomize build                 │
 │  │ Engine  │  → produces raw Kubernetes YAML                  │
 │  └────┬────┘                                                  │
 │       │                                                       │
 │  Step 3: FETCH actual state from cluster                      │
 │  ┌────▼────┐     ┌──────────┐                                 │
 │  │  Agent  │────>│ K8s API  │  kubectl get (equivalent)       │
 │  └────┬────┘     └──────────┘                                 │
 │       │                                                       │
 │  Step 4: DIFF desired vs actual                               │
 │  ┌────▼────┐                                                  │
 │  │  Diff   │  Three-way merge diff:                           │
 │  │ Engine  │  last-applied vs desired vs live                 │
 │  └────┬────┘                                                  │
 │       │                                                       │
 │  Step 5: APPLY changes (if any)                               │
 │  ┌────▼────┐     ┌──────────┐                                 │
 │  │ Server  │────>│ K8s API  │  Apply patches/creates/deletes  │
 │  │  Side   │     └──────────┘                                 │
 │  │ Apply   │                                                  │
 │  └────┬────┘                                                  │
 │       │                                                       │
 │  Step 6: REPORT status                                        │
 │  ┌────▼────┐                                                  │
 │  │ Status: │  Synced ✓  |  OutOfSync ✗  |  Degraded ⚠        │
 │  └─────────┘                                                  │
 │                                                               │
 │  Step 7: WAIT and repeat (loop back to Step 1)               │
 └────────────────────────────────────────────────────────────────┘
```

---

## Sync Status States

GitOps agents report the relationship between desired and actual state using status indicators:

```
┌────────────────────────────────────────────────────────────┐
│                    SYNC STATUSES                           │
│                                                            │
│  ┌──────────┐                                              │
│  │  SYNCED  │  Desired = Actual                            │
│  │    ✓     │  Everything matches Git                      │
│  └──────────┘                                              │
│                                                            │
│  ┌──────────┐                                              │
│  │OUT OF    │  Desired ≠ Actual                            │
│  │ SYNC ✗   │  Drift detected, sync needed                 │
│  └──────────┘                                              │
│                                                            │
│  ┌──────────┐                                              │
│  │ UNKNOWN  │  Agent can't determine state                 │
│  │    ?     │  (connectivity issue, parse error)           │
│  └──────────┘                                              │
│                                                            │
│  ┌──────────┐                                              │
│  │PROGRESSING│  Sync in progress                           │
│  │   ⟳     │  Changes being applied                       │
│  └──────────┘                                              │
│                                                            │
│  ┌──────────┐                                              │
│  │DEGRADED  │  Resources exist but aren't healthy          │
│  │    ⚠     │  (CrashLoopBackOff, pending)                 │
│  └──────────┘                                              │
└────────────────────────────────────────────────────────────┘
```

---

## Handling Special Cases

### HPA (Horizontal Pod Autoscaler) Conflict

HPA changes replicas dynamically, which conflicts with a static replica count in Git.

```yaml
# Problem: Git says replicas: 3, but HPA scales to 7
# Solution: Don't set replicas in the Deployment — let HPA control it

# deployment.yaml (Git)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  # replicas: OMITTED — HPA controls this
  template:
    spec:
      containers:
      - name: api
        image: api-server:v2

---
# hpa.yaml (Git)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-server
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
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

### Ignoring Fields in ArgoCD

```yaml
# ArgoCD: Ignore specific fields that change dynamically
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas           # Ignore replica drift (HPA manages)
  - group: ""
    kind: Service
    jsonPointers:
    - /metadata/annotations    # Ignore cloud-provider annotations
```

---

## Observing State in Practice

### ArgoCD Commands

```bash
# Check sync status
argocd app get myapp

# See the diff between desired and actual
argocd app diff myapp

# Force sync to desired state
argocd app sync myapp

# View detailed resource status
argocd app resources myapp
```

### Flux Commands

```bash
# Check Kustomization status
flux get kustomizations

# Force reconciliation
flux reconcile kustomization my-app --with-source

# View events
kubectl get events -n flux-system --sort-by=.lastTimestamp
```

### kubectl Commands

```bash
# View actual state
kubectl get deployment myapp -n production -o yaml

# Compare with desired state file
diff <(kubectl get deployment myapp -n production -o yaml) deployment.yaml

# View last-applied configuration
kubectl get deployment myapp -o jsonpath='{.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration}' | jq .
```

---

## Convergence: From Actual to Desired

```
┌────────────────────────────────────────────────────────────────┐
│                   CONVERGENCE OVER TIME                       │
│                                                               │
│  Desired: replicas=5, image=v3.2, cpu=500m                   │
│                                                               │
│  Time ─────────────────────────────────────────────►          │
│                                                               │
│  T0: Commit pushed to Git                                     │
│      Desired: replicas=5  |  Actual: replicas=3               │
│      Status: OUT OF SYNC                                      │
│                                                               │
│  T1: Agent detects change (poll/webhook)                      │
│      Agent starts reconciliation                              │
│      Status: PROGRESSING                                      │
│                                                               │
│  T2: Agent applies changes                                    │
│      replicas: 3 → 5                                          │
│      image: v3.1 → v3.2                                       │
│      cpu: 250m → 500m                                         │
│      Status: PROGRESSING                                      │
│                                                               │
│  T3: All pods running and healthy                             │
│      Desired = Actual                                         │
│      Status: SYNCED ✓                                          │
│                                                               │
│  T4: Someone runs kubectl scale --replicas=2                  │
│      Desired ≠ Actual (drift!)                                │
│      Status: OUT OF SYNC                                      │
│                                                               │
│  T5: Agent detects drift, reverts to replicas=5               │
│      Status: SYNCED ✓ (self-healed)                            │
└────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Desired State | Configuration stored in Git — the intended system state |
| Actual State | Live state of the cluster — what's currently running |
| Drift | When actual ≠ desired — caused by manual changes, HPA, etc. |
| Reconciliation | Process of making actual match desired |
| Sync Status | Synced, OutOfSync, Progressing, Degraded, Unknown |
| HPA Conflict | Omit replicas from Git, let HPA manage it |
| `ignoreDifferences` | ArgoCD feature to skip drift on specific fields |
| Convergence | System eventually reaches desired state through reconciliation |

---

## Quick Revision Questions

1. **Define desired state and actual state in the context of GitOps.**
   > Desired state: the infrastructure/app configuration stored declaratively in Git. Actual state: the live state of the system as reported by the cluster API.

2. **List four common causes of drift between desired and actual state.**
   > Manual `kubectl` edits, failed rollouts, HPA autoscaling, admission controller mutations.

3. **What are the five sync status states a GitOps agent can report?**
   > Synced, OutOfSync, Unknown, Progressing, Degraded.

4. **How do you resolve the conflict between HPA and a static replica count in Git?**
   > Omit the `replicas` field from the Deployment manifest in Git and let HPA control it. In ArgoCD, use `ignoreDifferences` on `/spec/replicas`.

5. **Describe the reconciliation flow from start to finish.**
   > Fetch desired state from Git → Render templates → Fetch actual state from cluster → Diff desired vs actual → Apply changes → Report status → Wait and repeat.

6. **What command shows the diff between desired and actual state in ArgoCD?**
   > `argocd app diff myapp`

---

[← Previous: Push vs Pull Model](03-push-vs-pull-model.md) | [Back to README](../README.md) | [Next → GitOps Benefits](05-gitops-benefits.md)
