# Chapter 6.1: Repository Organization

[← Previous: Self-Healing](../05-GitOps-Workflows/06-self-healing.md) | [Back to README](../README.md) | [Next → Branch Protection](02-branch-protection.md)

---

## Overview

How you structure your Git repositories directly impacts your GitOps workflow's scalability, security, and maintainability. This chapter covers **monorepo vs polyrepo**, directory structures, naming conventions, and patterns for organizing applications and infrastructure.

---

## Repository Models

```
┌──────────────────────────────────────────────────────────────┐
│              REPOSITORY MODELS                               │
│                                                              │
│  MONOREPO:                                                   │
│  ┌──────────────────────────────────────┐                    │
│  │ fleet-infra/                         │                    │
│  │ ├── clusters/                        │                    │
│  │ ├── infrastructure/                  │                    │
│  │ ├── apps/                            │                    │
│  │ └── tenants/                         │                    │
│  │                                      │                    │
│  │ Everything in one repo               │                    │
│  └──────────────────────────────────────┘                    │
│                                                              │
│  POLYREPO:                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ cluster- │  │ infra-   │  │ app-a-   │                   │
│  │ config/  │  │ config/  │  │ config/  │                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                                                              │
│  Separate repo per concern/team                              │
│                                                              │
│  HYBRID (Recommended):                                       │
│  ┌──────────────────┐  ┌──────────┐  ┌──────────┐           │
│  │ fleet-infra/     │  │ team-a-  │  │ team-b-  │           │
│  │ ├── clusters/    │  │ config/  │  │ config/  │           │
│  │ ├── infra/       │  └──────────┘  └──────────┘           │
│  │ └── tenants/     │                                        │
│  └──────────────────┘                                        │
│  Platform in one, apps per team                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Comparison of Models

| Aspect | Monorepo | Polyrepo | Hybrid |
|--------|----------|----------|--------|
| Simplicity | High (one repo) | Low (many repos) | Medium |
| Access control | Coarse (repo-level) | Fine (per repo) | Best of both |
| Team autonomy | Low | High | High |
| Cross-cutting changes | Easy (one PR) | Hard (multi-repo PR) | Medium |
| CI/CD pipeline | Single (complex) | Per-repo (simple) | Mixed |
| Audit trail | Centralized | Distributed | Mixed |
| Best for | Small orgs (< 5 teams) | Large orgs (> 20 teams) | Medium orgs |

---

## Recommended Hybrid Structure

```
# Platform repo (managed by platform team)
fleet-infra/
├── clusters/
│   ├── dev/
│   │   ├── flux-system/             # GitOps operator config
│   │   ├── infrastructure.yaml      # Kustomization → infra
│   │   └── tenants.yaml             # Kustomization → tenants
│   ├── staging/
│   │   ├── flux-system/
│   │   ├── infrastructure.yaml
│   │   └── tenants.yaml
│   └── production/
│       ├── flux-system/
│       ├── infrastructure.yaml
│       └── tenants.yaml
├── infrastructure/
│   ├── base/
│   │   ├── cert-manager/
│   │   ├── ingress-nginx/
│   │   ├── monitoring/
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── production/
└── tenants/
    ├── team-a/
    │   ├── namespace.yaml
    │   ├── rbac.yaml
    │   ├── source.yaml             # Points to team-a-config repo
    │   └── sync.yaml               # Kustomization
    └── team-b/
        ├── namespace.yaml
        ├── rbac.yaml
        ├── source.yaml
        └── sync.yaml
```

```
# Team repo (managed by application team)
team-a-config/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        └── kustomization.yaml
```

---

## Naming Conventions

### Repository Naming

```
Pattern: <scope>-<type>[-<modifier>]

Examples:
├── fleet-infra            # Platform infrastructure
├── team-frontend-config   # Frontend team's config
├── team-backend-config    # Backend team's config
├── shared-charts          # Shared Helm charts
└── policy-library         # OPA/Kyverno policies
```

### Directory Naming

```
Pattern: lowercase-with-hyphens/

Good:
├── cert-manager/
├── ingress-nginx/
├── kube-prometheus-stack/
└── external-secrets/

Bad:
├── CertManager/          # No PascalCase
├── cert_manager/         # No underscores
└── certmgr/              # No abbreviations
```

### File Naming

```
Pattern: <resource-type>.yaml or <descriptive-name>.yaml

Good:
├── deployment.yaml
├── service.yaml
├── configmap.yaml
├── helmrelease.yaml
└── kustomization.yaml

Alternative (for multiple resources of same type):
├── deployment-web.yaml
├── deployment-worker.yaml
├── service-web.yaml
└── service-worker.yaml
```

---

## App-of-Apps / App-of-Kustomizations Pattern

### ArgoCD App of Apps

```yaml
# Root application that manages other applications
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/fleet-infra.git
    targetRevision: main
    path: clusters/production/apps  # Contains Application YAMLs
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

### Flux Kustomization of Kustomizations

```yaml
# Root Kustomization that manages other Kustomizations
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./clusters/production/apps  # Contains Kustomization YAMLs
  prune: true
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|-----------------|
| Manifests in app source repo | Coupling code and config | Separate config repo |
| Flat directory (all files in root) | Unmanageable at scale | Hierarchical structure |
| Environment in filename | `deploy-prod.yaml` is confusing | Use overlay directories |
| Hardcoded values | Can't reuse across environments | Use Kustomize/Helm |
| Git submodules for config | Complex, error-prone | Separate GitRepository sources |
| One giant kustomization.yaml | Blast radius too large | Split by concern |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Monorepo | Simple but limited access control |
| Polyrepo | Fine-grained access but complex cross-cutting changes |
| Hybrid | Platform repo + per-team app repos (recommended) |
| Directory structure | clusters/ → infrastructure/ → apps/ → tenants/ |
| Naming | Lowercase with hyphens, descriptive names |
| App of Apps | Root application manages child applications |
| Base + overlays | Kustomize pattern for environment separation |
| Config ≠ code | Always separate deployment config from source code |

---

## Quick Revision Questions

1. **What are the three repository models for GitOps?**
   > Monorepo (everything in one), Polyrepo (one repo per concern), Hybrid (platform repo + per-team repos).

2. **Why is the hybrid model recommended?**
   > Platform team controls infrastructure and tenant onboarding centrally, while application teams have autonomy over their own config repos.

3. **Why should deployment configuration be in a separate repo from application code?**
   > Different review processes, access controls, and change frequencies. Application code changes shouldn't trigger deployment operations.

4. **What is the App of Apps pattern?**
   > A root Application (ArgoCD) or Kustomization (Flux) that manages other Applications/Kustomizations, creating a hierarchy.

5. **What naming convention should you follow for directories?**
   > Lowercase with hyphens (e.g., `cert-manager/`, `ingress-nginx/`). Avoid PascalCase, underscores, or abbreviations.

---

[← Previous: Self-Healing](../05-GitOps-Workflows/06-self-healing.md) | [Back to README](../README.md) | [Next → Branch Protection](02-branch-protection.md)
