# Chapter 1.3: Push vs Pull Model

[← Previous: GitOps Principles](02-gitops-principles.md) | [Back to README](../README.md) | [Next → Desired State vs Actual State](04-desired-vs-actual-state.md)

---

## Overview

One of the most important architectural decisions in continuous delivery is whether to use a **push-based** or **pull-based** deployment model. GitOps strongly favors the **pull model**, where an in-cluster agent pulls the desired state from Git, rather than having an external system push changes into the cluster.

---

## Push Model — Traditional CI/CD

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                      PUSH MODEL                                 │
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │Developer │───>│  Git     │───>│ CI/CD    │───>│ Target   │  │
│  │  Push    │    │  Repo    │    │ Pipeline │    │ Cluster  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│                                       │                         │
│                                       │  kubectl apply          │
│                                       │  helm install           │
│                                       │  (pushes OUT)           │
│                                       ▼                         │
│                              Pipeline needs:                    │
│                              • Cluster credentials              │
│                              • Network access to cluster        │
│                              • kubectl/helm installed           │
└─────────────────────────────────────────────────────────────────┘
```

### Push Model — Example Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml — Push Model
name: Deploy to Kubernetes
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        kubeconfig: ${{ secrets.KUBECONFIG }}  # Cluster creds in CI!

    - name: Deploy
      run: |
        kubectl apply -f k8s/
        kubectl rollout status deployment/myapp
```

### Push Model — Characteristics

| Aspect | Detail |
|--------|--------|
| Direction | External system pushes TO the cluster |
| Credentials | CI system stores cluster credentials |
| Trigger | Pipeline event (push, merge, tag) |
| Drift handling | None — fire and forget |
| Network | CI must reach cluster API (firewall rules needed) |
| Rollback | Re-run old pipeline or manually `kubectl rollout undo` |

---

## Pull Model — GitOps

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                       PULL MODEL                                │
│                                                                 │
│  ┌──────────┐    ┌──────────┐         ┌────────────────────┐   │
│  │Developer │───>│  Git     │◄────────│  GitOps Agent      │   │
│  │  Push    │    │  Repo    │  poll/  │  (ArgoCD / Flux)   │   │
│  └──────────┘    └──────────┘  webhook│                    │   │
│                                       │  ┌──────────────┐  │   │
│                                       │  │ Runs INSIDE  │  │   │
│                                       │  │ the cluster  │  │   │
│                                       │  └──────┬───────┘  │   │
│                                       │         │          │   │
│                                       │    apply│to cluster│   │
│                                       │         ▼          │   │
│                                       │  ┌──────────────┐  │   │
│                                       │  │  Kubernetes  │  │   │
│                                       │  │  API Server  │  │   │
│                                       │  └──────────────┘  │   │
│                                       └────────────────────┘   │
│                                                                 │
│  Agent PULLS from Git — no external access to cluster needed    │
└─────────────────────────────────────────────────────────────────┘
```

### Pull Model — Example (ArgoCD Application)

```yaml
# ArgoCD Application — Pull Model
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: main
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true      # Revert manual changes
    syncOptions:
    - CreateNamespace=true
```

### Pull Model — Characteristics

| Aspect | Detail |
|--------|--------|
| Direction | Agent inside cluster pulls FROM Git |
| Credentials | Agent uses in-cluster service account |
| Trigger | Agent detects Git changes (poll/webhook) |
| Drift handling | Continuous reconciliation and self-healing |
| Network | Only outbound from cluster to Git (no inbound needed) |
| Rollback | `git revert` — agent applies automatically |

---

## Side-by-Side Comparison

```
  PUSH MODEL                              PULL MODEL
  ══════════                              ══════════

  ┌────────┐       ┌────────┐            ┌────────┐       ┌────────┐
  │   CI   │──────>│Cluster │            │  Git   │<──────│ Agent  │
  │Pipeline│ push  │        │            │  Repo  │ pull  │(in     │
  └────────┘       └────────┘            └────────┘       │cluster)│
       │                                                   └───┬────┘
       │                                                       │
  Creds flow                                              Apply│locally
  outward ──>                                                  ▼
  (less secure)                                           ┌────────┐
                                                          │Cluster │
                                                          └────────┘
```

### Detailed Comparison Table

| Feature | Push Model | Pull Model (GitOps) |
|---------|-----------|---------------------|
| **Deploy trigger** | CI event | Git change detected by agent |
| **Credential location** | CI system (external) | In-cluster service account |
| **Network exposure** | Cluster API exposed to CI | Only outbound to Git |
| **Drift detection** | ✗ None | ✓ Continuous |
| **Self-healing** | ✗ No | ✓ Yes |
| **Audit trail** | CI logs (ephemeral) | Git history (permanent) |
| **Rollback** | Re-run pipeline | `git revert` |
| **Multi-cluster** | Pipeline per cluster | Agent per cluster, same repo |
| **Security** | Higher attack surface | Smaller attack surface |
| **Complexity** | Pipeline logic | Agent configuration |
| **Speed** | Immediate on trigger | Poll interval or webhook |

---

## Security Implications

### Push Model Security Risks

```
┌─────────────────────────────────────────────────────┐
│          PUSH MODEL - SECURITY CONCERNS             │
│                                                     │
│  ┌──────────┐                                       │
│  │ CI/CD    │─── stores kubeconfig ───┐             │
│  │ System   │                         │             │
│  └──────────┘                         ▼             │
│       │                        ┌─────────────┐      │
│       │  If CI is compromised: │  Cluster    │      │
│       │  • Attacker gets       │  (exposed)  │      │
│       │    cluster access      └─────────────┘      │
│       │  • Can deploy anything                      │
│       │  • Can exfiltrate data                      │
│       │  • Hard to detect                           │
│                                                     │
│  Risk: Cluster credentials stored outside cluster   │
│  Risk: CI system is a high-value target             │
│  Risk: Network path from CI to cluster = attack     │
│        vector                                       │
└─────────────────────────────────────────────────────┘
```

### Pull Model Security Benefits

```
┌─────────────────────────────────────────────────────┐
│          PULL MODEL - SECURITY BENEFITS             │
│                                                     │
│  ┌──────────┐          ┌─────────────────────┐      │
│  │ Git Repo │◄─── pull │  Agent (in-cluster) │      │
│  │          │          │  • Only needs Git    │      │
│  └──────────┘          │    read access       │      │
│                        │  • No external creds │      │
│  ✓ No cluster creds    │    stored outside    │      │
│    outside cluster     │  • Uses K8s RBAC     │      │
│  ✓ No inbound network  │    natively          │      │
│    access needed       └─────────────────────┘      │
│  ✓ Agent follows least                              │
│    privilege principle                               │
│  ✓ Changes must go                                   │
│    through Git (PR reviews)                          │
│  ✓ Blast radius is limited                           │
└─────────────────────────────────────────────────────┘
```

---

## Hybrid Approach

In practice, many teams use a **hybrid** approach:

```
┌──────────────────────────────────────────────────────────────┐
│                    HYBRID CI + GitOps                        │
│                                                              │
│  ┌──────┐   ┌──────┐   ┌──────────┐   ┌──────┐   ┌──────┐ │
│  │Code  │──>│  CI  │──>│Update Git│──>│Agent │──>│Cluster│ │
│  │Push  │   │Build │   │  Config  │   │ Pull │   │Deploy │ │
│  └──────┘   │ Test │   │  Repo    │   └──────┘   └──────┘ │
│             └──────┘   └──────────┘                        │
│                                                              │
│  CI handles: Build, Test, Push Image, Update manifests      │
│  GitOps handles: Deploy to cluster, Reconcile, Self-heal    │
└──────────────────────────────────────────────────────────────┘
```

### Hybrid Example — CI Updates Config Repo

```yaml
# CI Pipeline — builds image, updates config repo
# .github/workflows/ci.yml

name: CI Build
on:
  push:
    branches: [main]
    paths: ['src/**']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Build & Push Docker Image
      run: |
        docker build -t myregistry/myapp:${{ github.sha }} .
        docker push myregistry/myapp:${{ github.sha }}

    - name: Update Config Repo (trigger GitOps)
      run: |
        git clone https://github.com/myorg/myapp-config.git
        cd myapp-config
        # Update the image tag in the deployment manifest
        sed -i "s|image: myregistry/myapp:.*|image: myregistry/myapp:${{ github.sha }}|" \
          k8s/production/deployment.yaml
        git add .
        git commit -m "chore: update myapp to ${{ github.sha }}"
        git push
        # ArgoCD/Flux picks up the change automatically
```

---

## When to Use Which Model

| Scenario | Recommended Model |
|----------|------------------|
| Kubernetes-native workloads | **Pull (GitOps)** |
| Strong security requirements | **Pull (GitOps)** |
| Multi-cluster deployments | **Pull (GitOps)** |
| Compliance / audit needs | **Pull (GitOps)** |
| Simple, single-server deploys | **Push (CI/CD)** |
| Non-Kubernetes targets (VMs, FaaS) | **Push or Hybrid** |
| Image builds + K8s deploy | **Hybrid** |
| Database migrations | **Push** (imperative) |

---

## Troubleshooting: Common Issues

| Issue | Push Model Fix | Pull Model Fix |
|-------|---------------|----------------|
| Deployment not applying | Check CI logs, kubectl access | Check agent logs, Git repo connectivity |
| Drift after deploy | No fix (redesign needed) | Agent auto-corrects |
| Credentials expired | Rotate CI secrets | Rotate Git deploy key |
| Network blocked | Open firewall CI→Cluster | Open firewall Cluster→Git (outbound) |
| Rollback needed | Re-run old pipeline | `git revert` and push |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Push Model | External CI pushes to cluster — traditional approach |
| Pull Model | In-cluster agent pulls from Git — GitOps approach |
| Security | Pull model avoids exposing cluster creds externally |
| Hybrid | CI builds/tests, updates Git; Agent deploys from Git |
| Drift | Only Pull model detects and corrects drift |
| Rollback | Push: re-run pipeline; Pull: `git revert` |
| Network | Push: inbound to cluster; Pull: outbound to Git |

---

## Quick Revision Questions

1. **What is the fundamental difference between push and pull deployment models?**
   > Push: an external system sends changes to the cluster. Pull: an in-cluster agent fetches desired state from Git and applies it locally.

2. **Why is the pull model more secure than the push model?**
   > No cluster credentials are stored outside the cluster, no inbound network access is needed, and changes must go through Git (PR reviews).

3. **In a hybrid CI/GitOps approach, what does CI handle vs what does the GitOps agent handle?**
   > CI handles building, testing, and pushing images, then updates the config repo. The GitOps agent handles deploying to the cluster and reconciling.

4. **How does the pull model handle drift that a push model cannot?**
   > The pull model agent continuously compares Git (desired) with the cluster (actual) and corrects any drift. Push model is fire-and-forget with no drift detection.

5. **What network access does a pull-based GitOps agent need?**
   > Only outbound access from the cluster to the Git repository. No inbound access to the cluster API is needed from external systems.

6. **When would you still choose a push model over GitOps?**
   > For non-Kubernetes targets (VMs, FaaS), database migrations, simple single-server deploys, or purely imperative operations.

---

[← Previous: GitOps Principles](02-gitops-principles.md) | [Back to README](../README.md) | [Next → Desired State vs Actual State](04-desired-vs-actual-state.md)
