# Chapter 2.1: Repository Structure

[← Previous: GitOps Benefits](../01-GitOps-Concepts/05-gitops-benefits.md) | [Back to README](../README.md) | [Next → Branch Strategies](02-branch-strategies.md)

---

## Overview

How you structure your Git repositories is one of the most impactful architectural decisions in a GitOps implementation. The repository layout determines team boundaries, access control, blast radius, and operational workflows. This chapter covers the main repository patterns, their trade-offs, and recommended structures.

---

## The Three Repository Models

```
┌──────────────────────────────────────────────────────────────────┐
│                 REPOSITORY MODELS                                │
│                                                                  │
│  1. MONOREPO              2. POLYREPO           3. HYBRID        │
│  ┌──────────────────┐     ┌───────────────┐     ┌────────────┐  │
│  │ single-repo/     │     │ app-a-config/ │     │ platform/  │  │
│  │ ├── app-a/       │     │ └── k8s/      │     │ └── base/  │  │
│  │ ├── app-b/       │     │               │     │            │  │
│  │ ├── app-c/       │     │ app-b-config/ │     │ app-a/     │  │
│  │ └── platform/    │     │ └── k8s/      │     │ └── k8s/   │  │
│  └──────────────────┘     │               │     │            │  │
│                           │ app-c-config/ │     │ app-b/     │  │
│  All in one repo          │ └── k8s/      │     │ └── k8s/   │  │
│                           └───────────────┘     └────────────┘  │
│                           One repo per app      Platform + Apps  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Model 1: Monorepo

All application and infrastructure configs live in a single repository.

```
gitops-config/
├── README.md
├── apps/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       │   └── kustomization.yaml
│   │       ├── staging/
│   │       │   └── kustomization.yaml
│   │       └── production/
│   │           └── kustomization.yaml
│   ├── backend-api/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── staging/
│   │       └── production/
│   └── payment-service/
│       ├── base/
│       └── overlays/
│           ├── dev/
│           ├── staging/
│           └── production/
├── infrastructure/
│   ├── cert-manager/
│   ├── ingress-nginx/
│   ├── monitoring/
│   │   ├── prometheus/
│   │   └── grafana/
│   └── sealed-secrets/
└── clusters/
    ├── dev/
    │   └── kustomization.yaml       # References apps and infra for dev
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        └── kustomization.yaml
```

| Pros | Cons |
|------|------|
| Single source of truth | Large repo, slower clones |
| Easy cross-app changes | Broad access (hard to restrict per-app) |
| Atomic multi-app updates | One bad commit affects everything |
| Simple CI/CD setup | Merge conflicts across teams |

---

## Model 2: Polyrepo (Repo per App)

Each application or service has its own config repository.

```
# Repo: frontend-config
frontend-config/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    ├── staging/
    └── production/

# Repo: backend-api-config
backend-api-config/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    ├── staging/
    └── production/

# Repo: platform-config
platform-config/
├── cert-manager/
├── ingress-nginx/
├── monitoring/
└── sealed-secrets/
```

| Pros | Cons |
|------|------|
| Fine-grained access control | Many repos to manage |
| Independent team workflows | Cross-app changes require multi-repo PRs |
| Smaller blast radius | Harder to get holistic view |
| Fast clones | Governance overhead |

---

## Model 3: Hybrid (Recommended)

Separate **platform/infra** from **application** configs. Group related apps together.

```
# Repo 1: platform-config (managed by platform team)
platform-config/
├── base/
│   ├── namespaces/
│   ├── rbac/
│   ├── network-policies/
│   └── resource-quotas/
├── infrastructure/
│   ├── cert-manager/
│   ├── ingress-nginx/
│   ├── monitoring/
│   ├── logging/
│   └── sealed-secrets/
└── clusters/
    ├── dev-cluster/
    ├── staging-cluster/
    └── prod-cluster/

# Repo 2: team-alpha-apps (managed by Team Alpha)
team-alpha-apps/
├── frontend/
│   ├── base/
│   └── overlays/
├── bff-gateway/
│   ├── base/
│   └── overlays/
└── apps.yaml           # ArgoCD ApplicationSet or Flux Kustomization

# Repo 3: team-beta-apps (managed by Team Beta)
team-beta-apps/
├── payment-service/
│   ├── base/
│   └── overlays/
├── invoice-service/
│   ├── base/
│   └── overlays/
└── apps.yaml
```

| Pros | Cons |
|------|------|
| Clear ownership boundaries | Slightly more complex initial setup |
| Platform team controls infra | Need conventions across repos |
| Teams own their app configs | Cross-team changes need coordination |
| Balanced blast radius | |

---

## App Code vs Config Separation

```
┌──────────────────────────────────────────────────────────────────┐
│        APP CODE vs CONFIG — SEPARATE REPOSITORIES               │
│                                                                  │
│  App Source Repo                    Config/Deploy Repo            │
│  ┌──────────────────┐              ┌──────────────────┐          │
│  │ myapp/            │              │ myapp-config/     │         │
│  │ ├── src/          │              │ ├── base/         │         │
│  │ │   ├── main.go   │              │ │   ├── deploy.yaml│        │
│  │ │   └── handler.go│              │ │   ├── svc.yaml  │         │
│  │ ├── tests/        │              │ │   └── kustom... │         │
│  │ ├── Dockerfile    │              │ └── overlays/     │         │
│  │ ├── go.mod        │  CI builds   │     ├── dev/      │         │
│  │ └── .github/      │──────────────│     ├── staging/  │         │
│  │     workflows/    │  updates     │     └── prod/     │         │
│  │     ci.yaml       │  image tag   │                   │         │
│  └──────────────────┘              └──────────────────┘          │
│                                                                  │
│  WHY SEPARATE?                                                   │
│  • Different change cadence (code changes daily, config weekly)  │
│  • Different access controls (devs vs ops)                       │
│  • App CI shouldn't trigger config CI                            │
│  • Clean separation of concerns                                  │
│  • Prevents accidental deploys from code commits                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## Kustomize Directory Pattern (Recommended)

```
app-name/
├── base/                          # Shared across all environments
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml
│   └── kustomization.yaml
└── overlays/                      # Environment-specific overrides
    ├── dev/
    │   ├── kustomization.yaml     # patches, replicas, image tag
    │   ├── resource-patch.yaml    # lower resource limits
    │   └── config-patch.yaml      # dev-specific config
    ├── staging/
    │   ├── kustomization.yaml
    │   └── resource-patch.yaml
    └── production/
        ├── kustomization.yaml
        ├── resource-patch.yaml    # higher resource limits
        ├── hpa-patch.yaml         # production scaling
        └── ingress.yaml           # production ingress rules
```

### Example: Base Kustomization

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- configmap.yaml
commonLabels:
  app.kubernetes.io/managed-by: gitops
```

### Example: Production Overlay

```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
namespace: production
patches:
- path: resource-patch.yaml
- path: hpa-patch.yaml
images:
- name: myapp
  newName: registry.example.com/myapp
  newTag: v2.3.1
replicas:
- name: myapp
  count: 5
```

---

## Helm-Based Repository Structure

```
helm-apps/
├── charts/                        # Custom Helm charts
│   ├── myapp/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── ingress.yaml
│   └── shared-lib/
│       ├── Chart.yaml
│       └── templates/
├── releases/                      # Helm release values per environment
│   ├── dev/
│   │   ├── myapp-values.yaml
│   │   └── monitoring-values.yaml
│   ├── staging/
│   │   ├── myapp-values.yaml
│   │   └── monitoring-values.yaml
│   └── production/
│       ├── myapp-values.yaml
│       └── monitoring-values.yaml
└── helmfile.yaml                  # (Optional) Helmfile for orchestration
```

---

## Summary Table

| Repository Model | Best For | Team Size | Blast Radius |
|-----------------|---------|-----------|--------------|
| **Monorepo** | Small teams, few apps | 1-10 | Entire org |
| **Polyrepo** | Large orgs, strict access control | 50+ | Single app |
| **Hybrid** | Medium-large teams, balanced control | 10-100 | Team scope |
| **Separate code/config** | All sizes | Any | Config only |

---

## Quick Revision Questions

1. **What are the three main repository models for GitOps?**
   > Monorepo (all in one), Polyrepo (one repo per app), and Hybrid (platform + team repos).

2. **Why should application source code and deployment configs be in separate repositories?**
   > Different change cadence, different access controls, prevents code commits from triggering config deploys, and clean separation of concerns.

3. **What is the recommended directory pattern when using Kustomize?**
   > A `base/` directory with shared configs and an `overlays/` directory with environment-specific patches (dev, staging, production).

4. **In a Hybrid model, who typically owns the platform-config repo?**
   > The platform/infra team manages the platform-config repo (namespaces, RBAC, monitoring, ingress), while application teams manage their own app config repos.

5. **What is the main disadvantage of a Monorepo for large teams?**
   > Broad access (hard to restrict per-app), merge conflicts across teams, and one bad commit can affect all applications.

6. **How does a Kustomize overlay modify the base configuration without duplicating it?**
   > Overlays reference the base via `resources: [../../base]` and use patches, image overrides, and namespace settings to customize without copying base files.

---

[← Previous: GitOps Benefits](../01-GitOps-Concepts/05-gitops-benefits.md) | [Back to README](../README.md) | [Next → Branch Strategies](02-branch-strategies.md)
