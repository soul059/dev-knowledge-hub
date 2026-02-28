# Chapter 5.1: Application Deployment

[← Previous: Multi-Tenancy](../04-FluxCD/06-multi-tenancy.md) | [Back to README](../README.md) | [Next → Infrastructure Management](02-infrastructure-management.md)

---

## Overview

GitOps transforms application deployment from an imperative, script-driven process into a **declarative, Git-driven workflow**. This chapter covers end-to-end deployment patterns — from developer commit to production rollout — using both ArgoCD and Flux.

---

## GitOps Deployment Flow

```
┌──────────────────────────────────────────────────────────────┐
│           END-TO-END GITOPS DEPLOYMENT                       │
│                                                              │
│  ┌────────────┐     ┌────────────┐     ┌────────────┐       │
│  │ Developer  │────→│ App Repo   │────→│ CI Pipeline│       │
│  │ (code)     │     │ (source)   │     │ (build,    │       │
│  └────────────┘     └────────────┘     │  test,     │       │
│                                        │  image)    │       │
│                                        └─────┬──────┘       │
│                                              │               │
│                                        ┌─────▼──────┐       │
│                                        │ Container  │       │
│                                        │ Registry   │       │
│                                        └─────┬──────┘       │
│                                              │               │
│  ┌────────────┐     ┌────────────┐     ┌─────▼──────┐       │
│  │ Kubernetes │←────│ GitOps     │←────│ Config     │       │
│  │ Cluster    │     │ Operator   │     │ Repo       │       │
│  │ (runtime)  │     │ (ArgoCD/   │     │ (manifests)│       │
│  └────────────┘     │  Flux)     │     └────────────┘       │
│                     └────────────┘                           │
│                                                              │
│  Separation: App Repo (code) ≠ Config Repo (manifests)      │
└──────────────────────────────────────────────────────────────┘
```

---

## Deployment with ArgoCD

### Step 1: Application Manifest in Config Repo

```yaml
# config-repo/apps/my-app/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
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
        image: myorg/my-app:v1.2.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

### Step 2: ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/config-repo.git
    targetRevision: main
    path: apps/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Step 3: Deploy via Git

```bash
# Developer updates image tag in config repo
cd config-repo
sed -i 's|myorg/my-app:v1.2.0|myorg/my-app:v1.3.0|' apps/my-app/deployment.yaml
git add -A
git commit -m "chore: bump my-app to v1.3.0"
git push origin main

# ArgoCD detects the change and syncs automatically
# (if auto-sync is enabled)
```

---

## Deployment with Flux

### Step 1: Source and Kustomization

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/config-repo.git
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./apps/my-app
  prune: true
  wait: true
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: my-app
    namespace: my-app
```

---

## CI Pipeline Integration

```
┌──────────────────────────────────────────────────────────┐
│           CI/CD SPLIT IN GITOPS                          │
│                                                          │
│  CI (Continuous Integration):                            │
│  ┌──────────────────────────────────────┐                │
│  │ 1. Run tests              (app repo)│                │
│  │ 2. Build container image            │                │
│  │ 3. Push to registry                 │                │
│  │ 4. Update config repo              │                │
│  │    (image tag in deployment YAML)   │                │
│  └──────────────────────────────────────┘                │
│                                                          │
│  CD (Continuous Deployment):                             │
│  ┌──────────────────────────────────────┐                │
│  │ Handled entirely by GitOps operator │                │
│  │ ArgoCD / Flux detects config change │                │
│  │ Applies to cluster automatically    │                │
│  └──────────────────────────────────────┘                │
│                                                          │
│  Clean separation: CI never touches the cluster          │
└──────────────────────────────────────────────────────────┘
```

### GitHub Actions Example (CI updates Config Repo)

```yaml
# .github/workflows/ci.yaml (in app repo)
name: CI
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Build and push image
      run: |
        docker build -t myorg/my-app:${{ github.sha }} .
        docker push myorg/my-app:${{ github.sha }}

    - name: Update config repo
      run: |
        git clone https://x-access-token:${{ secrets.CONFIG_REPO_TOKEN }}@github.com/myorg/config-repo.git
        cd config-repo
        sed -i "s|myorg/my-app:.*|myorg/my-app:${{ github.sha }}|" apps/my-app/deployment.yaml
        git config user.name "CI Bot"
        git config user.email "ci@example.com"
        git add -A
        git commit -m "chore: update my-app to ${{ github.sha }}"
        git push
```

---

## Kustomize-Based Deployment

### Repository Structure

```
config-repo/
└── apps/
    └── my-app/
        ├── base/
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   └── kustomization.yaml
        └── overlays/
            ├── dev/
            │   ├── kustomization.yaml
            │   └── replica-patch.yaml
            ├── staging/
            │   ├── kustomization.yaml
            │   └── replica-patch.yaml
            └── production/
                ├── kustomization.yaml
                └── replica-patch.yaml
```

### Base kustomization.yaml

```yaml
# apps/my-app/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
```

### Production Overlay

```yaml
# apps/my-app/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patches:
- path: replica-patch.yaml
images:
- name: myorg/my-app
  newTag: v1.3.0
```

---

## Deployment Strategies Comparison

| Strategy | How It Works | Rollback | Risk |
|----------|-------------|----------|------|
| Recreate | Kill old → start new | Git revert | Downtime |
| Rolling update | Gradual pod replacement | Git revert | Low |
| Blue-Green | Switch traffic at once | Switch back | Medium (2x resources) |
| Canary | Route % of traffic | Argo Rollouts | Lowest |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| App repo vs config repo | Separate source code from deployment manifests |
| CI responsibility | Build, test, push image, update config repo |
| CD responsibility | GitOps operator syncs cluster from config repo |
| ArgoCD deployment | Application CRD with auto-sync |
| Flux deployment | GitRepository + Kustomization CRDs |
| Kustomize overlays | Base + per-environment patches |
| Image update | CI updates tag in config repo (or image automation) |

---

## Quick Revision Questions

1. **Why separate the app repo from the config repo?**
   > To decouple application source code from deployment configuration. Different teams, review processes, and access controls apply to each.

2. **What is the CI pipeline's role in GitOps?**
   > CI builds and tests the application, pushes the container image, and updates the image tag in the config repo. It never directly deploys to the cluster.

3. **How does a deployment happen without anyone running kubectl?**
   > The GitOps operator (ArgoCD/Flux) continuously watches the config repo. When a commit updates the image tag, the operator detects the change and syncs the cluster.

4. **What are the benefits of using Kustomize overlays for different environments?**
   > A single base config is shared across environments, with per-environment patches for replicas, resources, and configuration — reducing duplication and drift.

5. **How do you roll back a deployment in GitOps?**
   > `git revert` the commit that changed the image tag. The GitOps operator syncs the cluster back to the previous version.

---

[← Previous: Multi-Tenancy](../04-FluxCD/06-multi-tenancy.md) | [Back to README](../README.md) | [Next → Infrastructure Management](02-infrastructure-management.md)
