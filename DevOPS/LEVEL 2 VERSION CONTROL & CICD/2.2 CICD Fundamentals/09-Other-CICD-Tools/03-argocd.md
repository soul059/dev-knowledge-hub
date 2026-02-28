# Chapter 9.3: ArgoCD & GitOps

[← Previous: CircleCI](02-circleci.md) | [Back to README](../README.md) | [Next: Tekton →](04-tekton.md)

---

## Overview

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It continuously monitors Git repositories and ensures the live cluster state matches the desired state defined in Git. ArgoCD handles the **CD** part — it deploys but does not build or test.

---

## GitOps Principles

```
  GITOPS MODEL
  ═════════════

  Traditional CI/CD:
  Code Commit → Build → Test → Deploy (push to cluster)
                                  │
                            CI tool pushes
                            changes to cluster

  GitOps (ArgoCD):
  Code Commit → Build → Test → Update Manifests in Git
                                        │
                                  ArgoCD watches Git
                                        │
                                        ▼
                                  ArgoCD pulls changes
                                  and syncs to cluster

  KEY PRINCIPLES:
  1. Git is the single source of truth
  2. Desired state is declarative (YAML)
  3. Changes are made via Git commits (audit trail)
  4. Software agents ensure convergence (ArgoCD)
```

---

## Architecture

```
  ARGOCD ARCHITECTURE
  ════════════════════

  ┌──────────────────────────────────────────────┐
  │ Kubernetes Cluster                           │
  │                                              │
  │  ┌─────────────────────────────────────────┐ │
  │  │ ArgoCD Namespace                        │ │
  │  │                                         │ │
  │  │  ┌────────────┐   ┌──────────────────┐  │ │
  │  │  │ API Server │   │ Repo Server      │  │ │
  │  │  │ (UI + API) │   │ (clones Git repos│  │ │
  │  │  └────────────┘   │  generates       │  │ │
  │  │                    │  manifests)      │  │ │
  │  │  ┌────────────┐   └──────────────────┘  │ │
  │  │  │Application │                         │ │
  │  │  │Controller  │──── Watches Git repos   │ │
  │  │  │            │──── Compares desired vs  │ │
  │  │  │            │     live state           │ │
  │  │  │            │──── Syncs if needed      │ │
  │  │  └────────────┘                         │ │
  │  └─────────────────────────────────────────┘ │
  │                                              │
  │  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
  │  │ App A    │  │ App B    │  │ App C     │  │
  │  │ namespace│  │ namespace│  │ namespace │  │
  │  └──────────┘  └──────────┘  └───────────┘  │
  └──────────────────────────────────────────────┘
         ▲
         │ Watches & Syncs
         ▼
  ┌──────────────────┐
  │  Git Repository  │
  │  (manifests)     │
  │  ├── base/       │
  │  ├── staging/    │
  │  └── production/ │
  └──────────────────┘
```

---

## Application CRD

```yaml
# ArgoCD Application — defines what to deploy and where
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default

  # Source: where to get manifests
  source:
    repoURL: https://github.com/org/k8s-manifests.git
    targetRevision: main
    path: apps/my-app/overlays/production

  # Destination: where to deploy
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app

  # Sync policy
  syncPolicy:
    automated:
      prune: true                  # Delete resources not in Git
      selfHeal: true               # Auto-fix manual cluster changes
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

## Sync Policies

```
  SYNC MODES
  ══════════

  Manual Sync:
  ┌──────┐  git push   ┌──────┐  click sync  ┌─────────┐
  │ Git  │────────────▶│ArgoCD│──────────────▶│ Cluster │
  └──────┘             │ UI   │              └─────────┘
                       └──────┘
  ArgoCD shows: "OutOfSync" → User clicks "Sync"

  Auto Sync:
  ┌──────┐  git push   ┌──────┐  auto sync   ┌─────────┐
  │ Git  │────────────▶│ArgoCD│──────────────▶│ Cluster │
  └──────┘             └──────┘              └─────────┘
  ArgoCD detects change → Syncs automatically

  Self-Heal:
  Someone kubectl edits cluster → ArgoCD reverts to Git state
```

```yaml
syncPolicy:
  automated:
    prune: true          # Remove resources deleted from Git
    selfHeal: true       # Revert manual cluster changes

# Without automated: manual sync only (safer for production)
syncPolicy: {}
```

---

## Repository Layouts

### App of Apps Pattern

```
  APP OF APPS
  ═══════════

  Root Application (manages other applications):
  
  argocd-apps/
  ├── Chart.yaml
  └── templates/
      ├── app-frontend.yaml        # Application CRD
      ├── app-backend.yaml         # Application CRD
      ├── app-database.yaml        # Application CRD
      └── app-monitoring.yaml      # Application CRD

  Each template creates an ArgoCD Application
  that points to its own repo/path
```

```yaml
# argocd-apps/templates/app-frontend.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/org/k8s-manifests.git
    path: apps/frontend/overlays/{{ .Values.environment }}
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: frontend
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Kustomize Overlays

```
  k8s-manifests/
  └── apps/
      └── my-app/
          ├── base/
          │   ├── kustomization.yaml
          │   ├── deployment.yaml
          │   ├── service.yaml
          │   └── ingress.yaml
          └── overlays/
              ├── staging/
              │   ├── kustomization.yaml
              │   └── patch-replicas.yaml
              └── production/
                  ├── kustomization.yaml
                  ├── patch-replicas.yaml
                  └── patch-resources.yaml
```

---

## ArgoCD with CI Pipeline

```
  FULL GITOPS WORKFLOW
  ═════════════════════

  App Repo                    Config Repo
  ┌─────────┐                ┌─────────────┐
  │ src/    │   CI Pipeline  │ manifests/  │
  │ tests/  │───────────────▶│ values.yaml │
  │ Dockerfile│  (updates    │ (image tag  │
  └─────────┘    image tag)  │  updated)   │
                             └──────┬──────┘
                                    │
                              ArgoCD watches
                                    │
                                    ▼
                             ┌─────────────┐
                             │ Kubernetes  │
                             │ Cluster     │
                             └─────────────┘
```

```yaml
# GitHub Actions CI — builds and updates config repo
name: CI
on:
  push:
    branches: [main]

jobs:
  build-and-update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build and push Docker image
        run: |
          docker build -t ghcr.io/org/myapp:${{ github.sha }} .
          docker push ghcr.io/org/myapp:${{ github.sha }}

      - name: Update manifests repo
        run: |
          git clone https://x-access-token:${{ secrets.CONFIG_REPO_TOKEN }}@github.com/org/k8s-manifests.git
          cd k8s-manifests
          # Update image tag in values.yaml
          yq e ".image.tag = \"${{ github.sha }}\"" -i apps/myapp/values.yaml
          git add .
          git commit -m "Update myapp to ${{ github.sha }}"
          git push
      # ArgoCD detects the change and deploys automatically
```

---

## ApplicationSet

```yaml
# Deploy same app to multiple clusters/environments
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-app
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - cluster: staging
            url: https://staging-cluster.example.com
            namespace: my-app
          - cluster: production
            url: https://prod-cluster.example.com
            namespace: my-app
  template:
    metadata:
      name: 'my-app-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/k8s-manifests.git
        path: 'apps/my-app/overlays/{{cluster}}'
        targetRevision: main
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
```

---

## Real-World Scenario

> **Scenario**: A platform team manages 30 microservices across 3 Kubernetes clusters (dev, staging, prod). They need consistent deployments with full audit trails.
>
> **Solution**: Adopt GitOps with ArgoCD. Create a config repo with Kustomize overlays per environment. Use the "App of Apps" pattern — one root application that manages all 30 service applications. CI pipelines build images and update the config repo image tags. ArgoCD auto-syncs dev and staging. Production uses manual sync with the ArgoCD UI requiring approval. All changes flow through Git PRs — providing review, audit trail, and easy rollback (just revert the commit).

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| App stuck "OutOfSync" | Diff between Git and cluster | Check diff in UI; sync or fix manifests |
| "ComparisonError" | Invalid manifests or path | Validate YAML; check repo path exists |
| Self-heal reverting changes | `selfHeal: true` enabled | Use Git for all changes — that's the point |
| Sync fails with hooks | Pre/post-sync hook errors | Check hook job logs in ArgoCD UI |
| Repo not accessible | Auth issues | Update repo credentials in ArgoCD settings |
| Too many resources | Large cluster state | Use label selectors, namespace isolation |

---

## Summary Table

| Feature | Description |
|---|---|
| GitOps model | Git is the source of truth for deployments |
| Application CRD | Kubernetes CR defining source repo and destination cluster |
| Auto sync | Automatically deploys when Git changes detected |
| Self-heal | Reverts manual cluster changes to match Git |
| Prune | Deletes resources removed from Git |
| App of Apps | Meta-application managing other applications |
| ApplicationSet | Template-based multi-cluster/multi-env deployments |
| Manifest tools | Supports Kustomize, Helm, Jsonnet, plain YAML |

---

## Quick Revision Questions

1. **What is GitOps?** *An operational model where Git is the single source of truth for declarative infrastructure. Software agents (like ArgoCD) ensure the actual system state matches the desired state defined in Git.*
2. **How does ArgoCD differ from traditional CI/CD?** *Traditional CI/CD pushes changes to clusters. ArgoCD operates as a pull-based model — it watches Git and pulls changes to the cluster, ensuring convergence.*
3. **What is the Application CRD?** *A Kubernetes custom resource that defines: source (Git repo + path), destination (cluster + namespace), and sync policy (auto/manual, prune, self-heal).*
4. **What does self-heal mean?** *When someone manually changes the cluster (e.g., via kubectl), ArgoCD detects the drift and automatically reverts the change to match the Git-defined state.*
5. **What is the App of Apps pattern?** *A root ArgoCD Application that manages other Application resources — enabling centralized control over many applications from a single entry point.*
6. **How do CI and ArgoCD work together?** *CI pipelines build and test code, then update image tags in a config repo. ArgoCD watches the config repo and deploys the new version — separating build (CI) from deploy (CD).*

---

[← Previous: CircleCI](02-circleci.md) | [Back to README](../README.md) | [Next: Tekton →](04-tekton.md)
