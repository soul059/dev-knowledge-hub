# Chapter 3.5: Rollouts and Rollbacks

[← Previous: Sync Policies](04-sync-policies.md) | [Back to README](../README.md) | [Next → Multi-Cluster Management](06-multi-cluster-management.md)

---

## Overview

Rollouts and rollbacks are critical operational capabilities. In GitOps, rollbacks are simply `git revert` operations. For advanced deployment strategies (canary, blue-green), ArgoCD integrates with **Argo Rollouts**, a Kubernetes controller that provides progressive delivery features.

---

## GitOps Rollback — The Simple Way

```
┌──────────────────────────────────────────────────────────────┐
│                  GITOPS ROLLBACK                             │
│                                                              │
│  Git History:                                                │
│  ┌──────────────────────────────────────────────────┐       │
│  │  HEAD   abc123  chore: update to v3.0 ← BAD!    │       │
│  │         def456  fix: adjust memory limits        │       │
│  │         ghi789  feat: add health checks          │       │
│  │         jkl012  chore: update to v2.0 ← GOOD    │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  Rollback:                                                   │
│  $ git revert abc123 --no-edit                               │
│  $ git push origin main                                      │
│                                                              │
│  Result:                                                     │
│  ┌──────────────────────────────────────────────────┐       │
│  │  HEAD   mno345  Revert "chore: update to v3.0"  │       │
│  │         abc123  chore: update to v3.0            │       │
│  │         ...                                      │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  ArgoCD detects the revert → syncs → cluster reverts to v2  │
│  Time: under 60 seconds                                     │
└──────────────────────────────────────────────────────────────┘
```

### ArgoCD Rollback Commands

```bash
# View application history
argocd app history myapp

# ID  DATE                  REVISION
# 1   2026-02-25 10:00:00  jkl012
# 2   2026-02-26 14:30:00  ghi789
# 3   2026-02-27 09:00:00  def456
# 4   2026-02-28 11:00:00  abc123

# Rollback to a specific history ID
argocd app rollback myapp 3

# NOTE: ArgoCD rollback is a TEMPORARY override
# The next Git change will re-sync to HEAD
# For permanent rollback, use git revert
```

---

## Argo Rollouts — Progressive Delivery

```
┌──────────────────────────────────────────────────────────────┐
│               ARGO ROLLOUTS                                  │
│                                                              │
│  Standard K8s Deployment:                                    │
│  ┌────────────────────────────────────────┐                  │
│  │  All pods update at once               │                  │
│  │  (rolling update — limited control)    │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
│  Argo Rollouts:                                              │
│  ┌────────────────────────────────────────┐                  │
│  │  ┌──────────┐  ┌──────────┐           │                  │
│  │  │  Canary  │  │Blue/Green│           │                  │
│  │  │  Deploy  │  │  Deploy  │           │                  │
│  │  │ 5% → 25% │  │ Switch   │           │                  │
│  │  │ → 50%    │  │ traffic  │           │                  │
│  │  │ → 100%   │  │ at once  │           │                  │
│  │  └──────────┘  └──────────┘           │                  │
│  │                                       │                  │
│  │  + Analysis (auto-promote/rollback)   │                  │
│  │  + Traffic management (Istio/Nginx)   │                  │
│  └────────────────────────────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

### Install Argo Rollouts

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts \
  -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install kubectl plugin
brew install argoproj/tap/kubectl-argo-rollouts
# OR
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

---

## Canary Deployment

```
┌──────────────────────────────────────────────────────────────┐
│                  CANARY DEPLOYMENT                           │
│                                                              │
│  Step 1: 5% traffic to canary                                │
│  ┌─────────────────────────────────────────┐                 │
│  │  Stable (v1) ████████████████████ 95%   │                 │
│  │  Canary (v2) █                     5%   │                 │
│  └─────────────────────────────────────────┘                 │
│               ▼ Analysis: check error rate                   │
│                                                              │
│  Step 2: 25% traffic to canary                               │
│  ┌─────────────────────────────────────────┐                 │
│  │  Stable (v1) ███████████████      75%   │                 │
│  │  Canary (v2) █████                25%   │                 │
│  └─────────────────────────────────────────┘                 │
│               ▼ Analysis: check latency                      │
│                                                              │
│  Step 3: 50% traffic (pause for manual check)                │
│  ┌─────────────────────────────────────────┐                 │
│  │  Stable (v1) ██████████           50%   │                 │
│  │  Canary (v2) ██████████           50%   │                 │
│  └─────────────────────────────────────────┘                 │
│               ▼ Manual promotion                             │
│                                                              │
│  Step 4: 100% — canary becomes stable                        │
│  ┌─────────────────────────────────────────┐                 │
│  │  Stable (v2) ████████████████████ 100%  │                 │
│  └─────────────────────────────────────────┘                 │
└──────────────────────────────────────────────────────────────┘
```

### Canary Rollout Spec

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:v2
        ports:
        - containerPort: 8080
  strategy:
    canary:
      steps:
      - setWeight: 5
      - pause: {duration: 5m}       # Wait 5 minutes
      - setWeight: 25
      - analysis:                    # Run automated analysis
          templates:
          - templateName: success-rate
      - setWeight: 50
      - pause: {}                    # Manual promotion required
      - setWeight: 100
      canaryService: myapp-canary    # Service for canary pods
      stableService: myapp-stable    # Service for stable pods
```

### Analysis Template (Auto-Promote/Rollback)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  metrics:
  - name: success-rate
    interval: 60s
    count: 5
    successCondition: result[0] >= 0.95    # 95% success rate
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring:9090
        query: |
          sum(rate(http_requests_total{status=~"2.*",app="myapp-canary"}[5m])) /
          sum(rate(http_requests_total{app="myapp-canary"}[5m]))
```

---

## Blue-Green Deployment

```
┌──────────────────────────────────────────────────────────────┐
│                BLUE-GREEN DEPLOYMENT                         │
│                                                              │
│  Before cutover:                                             │
│  ┌─────────────────────────────────────────┐                 │
│  │  Active Service ──► Blue (v1) ████ LIVE │                 │
│  │  Preview Service ──► Green (v2) ████ TEST│                │
│  └─────────────────────────────────────────┘                 │
│                                                              │
│  After cutover (instant traffic switch):                     │
│  ┌─────────────────────────────────────────┐                 │
│  │  Active Service ──► Green (v2) ████ LIVE│                 │
│  │  Preview Service ──► Blue (v1) ████ OLD │                 │
│  └─────────────────────────────────────────┘                 │
│                                                              │
│  Instant rollback: switch back to Blue                       │
└──────────────────────────────────────────────────────────────┘
```

### Blue-Green Rollout Spec

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:v2
  strategy:
    blueGreen:
      activeService: myapp-active       # Production traffic
      previewService: myapp-preview     # Preview/test traffic
      autoPromotionEnabled: false       # Manual promotion
      previewReplicaCount: 2            # Scale preview to 2
      scaleDownDelaySeconds: 30         # Wait before scaling old
      prePromotionAnalysis:
        templates:
        - templateName: smoke-tests
        args:
        - name: service-name
          value: myapp-preview
```

---

## Rollout Commands

```bash
# Watch rollout progress
kubectl argo rollouts get rollout myapp --watch

# Promote canary to next step
kubectl argo rollouts promote myapp

# Full promotion (skip remaining steps)
kubectl argo rollouts promote myapp --full

# Abort rollout (rollback to stable)
kubectl argo rollouts abort myapp

# Retry aborted rollout
kubectl argo rollouts retry rollout myapp

# Undo rollout (revert to previous version)
kubectl argo rollouts undo myapp

# View rollout dashboard
kubectl argo rollouts dashboard
```

---

## Summary Table

| Strategy | Traffic Control | Rollback Speed | Complexity | Use Case |
|----------|----------------|----------------|------------|----------|
| Git revert | N/A | < 1 min | Low | Any GitOps deployment |
| Rolling update | Gradual (K8s native) | Minutes | Low | Simple apps |
| Canary | Percentage-based | Seconds | Medium | Risk-sensitive apps |
| Blue-Green | Instant switch | Instant | Medium | Zero-downtime required |
| Canary + Analysis | Auto-promote/rollback | Seconds | High | Production-critical |

---

## Quick Revision Questions

1. **How does a GitOps rollback differ from a traditional rollback?**
   > GitOps rollback is `git revert` — it changes the desired state in Git, and the agent auto-syncs. Traditional rollback requires re-running pipelines or `kubectl rollout undo`.

2. **What is the difference between ArgoCD rollback and git revert?**
   > ArgoCD rollback is a temporary override that reverts to a previous sync state. The next Git change overwrites it. `git revert` is permanent and changes the source of truth.

3. **How does canary deployment work with Argo Rollouts?**
   > Traffic is gradually shifted from stable to canary pods in configurable steps (e.g., 5% → 25% → 50% → 100%), with optional analysis at each step.

4. **What does an AnalysisTemplate do?**
   > It defines metrics (e.g., from Prometheus) and success/failure conditions. During a rollout, analysis runs automatically and can promote or abort the rollout.

5. **When would you choose blue-green over canary?**
   > When you need instant traffic switching (no gradual rollout) and want the ability to instantly roll back by switching services.

6. **What command aborts a canary rollout and reverts to the stable version?**
   > `kubectl argo rollouts abort myapp`

---

[← Previous: Sync Policies](04-sync-policies.md) | [Back to README](../README.md) | [Next → Multi-Cluster Management](06-multi-cluster-management.md)
