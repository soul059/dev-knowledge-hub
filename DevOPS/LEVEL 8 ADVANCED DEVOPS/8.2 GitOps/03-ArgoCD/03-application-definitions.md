# Chapter 3.3: Application Definitions

[← Previous: ArgoCD Installation](02-argocd-installation.md) | [Back to README](../README.md) | [Next → Sync Policies](04-sync-policies.md)

---

## Overview

An ArgoCD **Application** is the core resource that defines *what* to deploy, *where* to deploy it, and *how* to manage it. This chapter covers Application CRDs, AppProjects, ApplicationSets, and the App of Apps pattern.

---

## Application CRD — Full Spec

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
  labels:
    team: backend
    env: production
  finalizers:
  - resources-finalizer.argocd.argoproj.io  # Clean up on delete
spec:
  project: default

  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: main           # Branch, tag, or commit SHA
    path: overlays/production      # Directory within repo

    # For Kustomize sources:
    kustomize:
      namePrefix: prod-
      commonLabels:
        env: production

  destination:
    server: https://kubernetes.default.svc   # Cluster URL
    namespace: production

  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true     # Revert manual changes
      allowEmpty: false  # Don't sync if manifests are empty
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas       # Ignore HPA-managed replicas
```

---

## Source Types

```
┌──────────────────────────────────────────────────────────────┐
│                   SOURCE TYPES                               │
│                                                              │
│  ┌─ Git Directory ──────────────────────────────────────┐    │
│  │  source:                                             │    │
│  │    repoURL: https://github.com/myorg/config.git      │    │
│  │    path: k8s/production                              │    │
│  │    targetRevision: main                              │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─ Helm Chart (from Git) ──────────────────────────────┐    │
│  │  source:                                             │    │
│  │    repoURL: https://github.com/myorg/charts.git      │    │
│  │    path: charts/myapp                                │    │
│  │    helm:                                             │    │
│  │      valueFiles:                                     │    │
│  │      - values-production.yaml                        │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─ Helm Chart (from Registry) ─────────────────────────┐    │
│  │  source:                                             │    │
│  │    repoURL: https://charts.bitnami.com/bitnami       │    │
│  │    chart: nginx                                      │    │
│  │    targetRevision: 15.3.0                            │    │
│  │    helm:                                             │    │
│  │      values: |                                       │    │
│  │        replicaCount: 3                               │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─ Kustomize ──────────────────────────────────────────┐    │
│  │  source:                                             │    │
│  │    repoURL: https://github.com/myorg/config.git      │    │
│  │    path: overlays/production                         │    │
│  │    kustomize:                                        │    │
│  │      images:                                         │    │
│  │      - name=myapp newTag=v2.0                        │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

---

## AppProject — Governance Boundaries

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-alpha
  namespace: argocd
spec:
  description: "Team Alpha applications"

  # Allowed source repositories
  sourceRepos:
  - https://github.com/myorg/team-alpha-*
  - https://github.com/myorg/shared-charts.git

  # Allowed destination clusters and namespaces
  destinations:
  - server: https://kubernetes.default.svc
    namespace: team-alpha-*
  - server: https://kubernetes.default.svc
    namespace: shared

  # Allowed cluster-scoped resources
  clusterResourceWhitelist:
  - group: ""
    kind: Namespace

  # Denied namespace-scoped resources
  namespaceResourceBlacklist:
  - group: ""
    kind: ResourceQuota
  - group: ""
    kind: LimitRange

  # RBAC roles within the project
  roles:
  - name: developer
    description: "Can sync apps"
    policies:
    - p, proj:team-alpha:developer, applications, sync, team-alpha/*, allow
    - p, proj:team-alpha:developer, applications, get, team-alpha/*, allow
    groups:
    - team-alpha-devs    # Maps to OIDC/LDAP group
```

### Project Isolation Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                  APPPROJECT ISOLATION                        │
│                                                              │
│  Project: team-alpha                                         │
│  ┌───────────────────────────────────────────────┐           │
│  │  Allowed Sources:                             │           │
│  │  ├── github.com/myorg/team-alpha-*            │           │
│  │  └── github.com/myorg/shared-charts           │           │
│  │                                               │           │
│  │  Allowed Destinations:                        │           │
│  │  ├── namespace: team-alpha-*                  │           │
│  │  └── namespace: shared                        │           │
│  │                                               │           │
│  │  Apps:                                        │           │
│  │  ├── frontend (team-alpha-frontend ns)        │           │
│  │  └── bff-api (team-alpha-api ns)              │           │
│  └───────────────────────────────────────────────┘           │
│                                                              │
│  Project: team-beta                                          │
│  ┌───────────────────────────────────────────────┐           │
│  │  Allowed Sources:                             │           │
│  │  └── github.com/myorg/team-beta-*             │           │
│  │                                               │           │
│  │  Allowed Destinations:                        │           │
│  │  └── namespace: team-beta-*                   │           │
│  │                                               │           │
│  │  Apps:                                        │           │
│  │  ├── payment-service                          │           │
│  │  └── invoice-service                          │           │
│  └───────────────────────────────────────────────┘           │
│                                                              │
│  Team Alpha CANNOT deploy to team-beta namespaces            │
│  Team Beta CANNOT use team-alpha's repos                     │
└──────────────────────────────────────────────────────────────┘
```

---

## ApplicationSet — Generating Multiple Apps

ApplicationSets generate ArgoCD Applications dynamically from templates.

### List Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-envs
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - env: dev
        cluster: https://dev-cluster.example.com
        namespace: myapp-dev
      - env: staging
        cluster: https://staging-cluster.example.com
        namespace: myapp-staging
      - env: production
        cluster: https://prod-cluster.example.com
        namespace: myapp-prod
  template:
    metadata:
      name: "myapp-{{env}}"
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/myapp-config.git
        targetRevision: main
        path: "overlays/{{env}}"
      destination:
        server: "{{cluster}}"
        namespace: "{{namespace}}"
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Git Directory Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: all-apps
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/myorg/config.git
      revision: main
      directories:
      - path: apps/*           # Each subdirectory = one Application
  template:
    metadata:
      name: "{{path.basename}}"
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/config.git
        targetRevision: main
        path: "{{path}}"
      destination:
        server: https://kubernetes.default.svc
        namespace: "{{path.basename}}"
```

---

## App of Apps Pattern

```
┌──────────────────────────────────────────────────────────────┐
│                   APP OF APPS PATTERN                        │
│                                                              │
│  Root Application (bootstraps everything)                    │
│  ┌────────────────────────────────────┐                      │
│  │  root-app                         │                      │
│  │  source: apps/ directory          │                      │
│  │                                   │                      │
│  │  Contains Application manifests:  │                      │
│  │  ├── frontend-app.yaml            │                      │
│  │  ├── backend-app.yaml             │                      │
│  │  ├── monitoring-app.yaml          │                      │
│  │  └── ingress-app.yaml             │                      │
│  └────────────────┬───────────────────┘                      │
│                   │                                          │
│         ┌─────────┼─────────┬─────────┐                      │
│         ▼         ▼         ▼         ▼                      │
│  ┌──────────┐ ┌──────┐ ┌────────┐ ┌──────┐                  │
│  │ frontend │ │ back │ │monitor │ │ingress│                  │
│  │ app      │ │ end  │ │ ing    │ │      │                  │
│  └──────────┘ └──────┘ └────────┘ └──────┘                  │
│                                                              │
│  Deploy ONE app → ALL apps are managed                       │
└──────────────────────────────────────────────────────────────┘
```

### Root App Definition

```yaml
# root-app.yaml — Bootstrap everything
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/platform-config.git
    targetRevision: main
    path: apps/            # Contains Application YAML files
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd      # Apps deploy to argocd namespace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Child App Example

```yaml
# apps/frontend-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend
  namespace: argocd
spec:
  project: team-alpha
  source:
    repoURL: https://github.com/myorg/frontend-config.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: frontend
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## CLI Application Management

```bash
# Create an application
argocd app create myapp \
  --repo https://github.com/myorg/config.git \
  --path overlays/production \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# List applications
argocd app list

# Get application details
argocd app get myapp

# Delete an application
argocd app delete myapp
# With cascade (delete resources too):
argocd app delete myapp --cascade
```

---

## Summary Table

| Resource | Purpose | Key Fields |
|----------|---------|------------|
| **Application** | Single deployable unit | source, destination, syncPolicy |
| **AppProject** | Governance boundary | sourceRepos, destinations, roles |
| **ApplicationSet** | Generate multiple apps | generators, template |
| **App of Apps** | Bootstrap pattern | Root app manages child apps |

---

## Quick Revision Questions

1. **What are the key fields in an ArgoCD Application spec?**
   > `source` (Git repo, path, revision), `destination` (cluster, namespace), `syncPolicy` (automated, prune, selfHeal), and `project`.

2. **What is the purpose of an AppProject?**
   > It defines governance boundaries — restricting which repos, clusters, and namespaces an Application can use, and assigning RBAC roles.

3. **How does an ApplicationSet differ from manually creating Applications?**
   > ApplicationSets use generators (list, git, cluster) to automatically create and manage multiple Applications from a single template.

4. **Explain the App of Apps pattern.**
   > A root Application points to a directory containing child Application manifests. Deploying the root app bootstraps all child applications automatically.

5. **How do you prevent a team from deploying to another team's namespace?**
   > Create separate AppProjects with restricted `destinations` — only allow each team's project to target their own namespaces.

6. **What does the `resources-finalizer.argocd.argoproj.io` finalizer do?**
   > When the Application is deleted, this finalizer ensures all managed Kubernetes resources are also deleted (cascade delete).

---

[← Previous: ArgoCD Installation](02-argocd-installation.md) | [Back to README](../README.md) | [Next → Sync Policies](04-sync-policies.md)
