# Chapter 2.2: Branch Strategies

[← Previous: Repository Structure](01-repository-structure.md) | [Back to README](../README.md) | [Next → Change Management](03-change-management.md)

---

## Overview

In GitOps, branches determine **when and where** changes are deployed. Choosing the right branching strategy affects your release velocity, stability, and team workflow. Unlike application development, GitOps config repos require strategies that map branches to environments clearly.

---

## Branch-to-Environment Mapping Models

```
┌──────────────────────────────────────────────────────────────────┐
│                BRANCH STRATEGIES FOR GITOPS                     │
│                                                                  │
│  Model A: BRANCH-PER-ENVIRONMENT                                │
│  ┌──────────────────────────────────────────────────────┐       │
│  │  main ─────────────────────────────────► production   │       │
│  │  staging ──────────────────────────────► staging      │       │
│  │  develop ──────────────────────────────► dev          │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                  │
│  Model B: SINGLE-BRANCH + DIRECTORY-PER-ENVIRONMENT             │
│  ┌──────────────────────────────────────────────────────┐       │
│  │  main ──┬── overlays/dev/ ──────────────► dev         │       │
│  │         ├── overlays/staging/ ──────────► staging     │       │
│  │         └── overlays/production/ ──────► production   │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                  │
│  Model C: TAG/RELEASE-BASED PROMOTION                           │
│  ┌──────────────────────────────────────────────────────┐       │
│  │  main ──── v1.0 tag ───► staging ───► v1.0-prod ──► prod│   │
│  └──────────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```

---

## Model A: Branch-Per-Environment

Each environment has a dedicated long-lived branch.

```
                Feature Branch Workflow
                ═══════════════════════

  feature/new-api ──┐
                    ├──► PR to develop ─── merge ──► DEV cluster
                    │
  develop ──────────┤
                    ├──► PR to staging ─── merge ──► STAGING cluster
                    │
  staging ──────────┤
                    ├──► PR to main ────── merge ──► PRODUCTION cluster
                    │
  main ─────────────┘
```

### Configuration

```yaml
# ArgoCD Application for dev environment
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
spec:
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: develop          # Branch maps to env
    path: k8s/
  destination:
    server: https://dev-cluster.example.com
    namespace: myapp

---
# ArgoCD Application for production
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
spec:
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: main             # Production = main branch
    path: k8s/
  destination:
    server: https://prod-cluster.example.com
    namespace: myapp
```

| Pros | Cons |
|------|------|
| Clear separation per env | Cherry-pick / merge between branches = pain |
| Easy access control per branch | Configs drift between branches over time |
| Familiar Git workflow | Hard to compare environments |
| | Merge conflicts common |

---

## Model B: Single Branch + Directory-Per-Environment (Recommended)

One branch (`main`), environments differentiated by directory (Kustomize overlays).

```
  main branch
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │  myapp-config/                                     │
  │  ├── base/                  ← Shared config        │
  │  │   ├── deployment.yaml                           │
  │  │   ├── service.yaml                              │
  │  │   └── kustomization.yaml                        │
  │  └── overlays/                                     │
  │      ├── dev/               ← Dev overrides        │
  │      │   └── kustomization.yaml                    │
  │      ├── staging/           ← Staging overrides    │
  │      │   └── kustomization.yaml                    │
  │      └── production/        ← Prod overrides       │
  │          └── kustomization.yaml                    │
  │                                                    │
  │  Deploy flow:                                      │
  │  PR → review → merge to main → ALL envs updated   │
  │  (each ArgoCD App points to its overlay directory) │
  └────────────────────────────────────────────────────┘
```

### Configuration

```yaml
# ArgoCD — dev points to dev overlay
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
spec:
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: main
    path: overlays/dev             # Directory = environment
  destination:
    server: https://dev-cluster.example.com
    namespace: myapp

---
# ArgoCD — production points to production overlay
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
spec:
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: main
    path: overlays/production      # Directory = environment
  destination:
    server: https://prod-cluster.example.com
    namespace: myapp
```

| Pros | Cons |
|------|------|
| No merge between branches | All envs update on merge (need careful PRs) |
| Easy to compare envs (`diff`) | Requires discipline for env-specific changes |
| One PR for all env changes | Fine-grained rollback per env is harder |
| No config drift between branches | |
| Recommended by ArgoCD & Flux | |

---

## Model C: Tag/Release-Based Promotion

Tags or releases gate environment promotion.

```
  main ──── commit ──── commit ──── commit
                         │
                         └── tag: v1.2.0
                              │
                              ├── dev: auto-deploy v1.2.0
                              │
                              ├── staging: manual approve → deploy v1.2.0
                              │
                              └── prod: manual approve → deploy v1.2.0
```

### Configuration

```yaml
# ArgoCD — staging tracks release tags
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-staging
spec:
  source:
    repoURL: https://github.com/myorg/myapp-config.git
    targetRevision: v1.2.0          # Pinned to specific tag
    path: overlays/staging
  destination:
    server: https://staging-cluster.example.com
    namespace: myapp
```

---

## Feature Branch Workflow for GitOps

```
┌──────────────────────────────────────────────────────────────┐
│             FEATURE BRANCH WORKFLOW                          │
│                                                              │
│  1. Create feature branch                                    │
│     git checkout -b feat/update-replicas                     │
│                                                              │
│  2. Make config changes                                      │
│     Edit overlays/production/kustomization.yaml              │
│                                                              │
│  3. Open Pull Request                                        │
│     PR: "Increase production replicas to 5"                  │
│     ┌─────────────────────────────────────────┐              │
│     │  PR #42: Increase prod replicas to 5    │              │
│     │                                         │              │
│     │  Reviewers: @ops-team                   │              │
│     │  CI Checks: ✓ YAML lint                 │              │
│     │             ✓ Kustomize build            │              │
│     │             ✓ Policy check (OPA)        │              │
│     │             ✓ Diff preview              │              │
│     │                                         │              │
│     │  [Approve] [Request Changes]            │              │
│     └─────────────────────────────────────────┘              │
│                                                              │
│  4. PR approved and merged to main                           │
│                                                              │
│  5. ArgoCD detects change on main, syncs to cluster          │
│                                                              │
│  6. Feature branch deleted                                   │
└──────────────────────────────────────────────────────────────┘
```

---

## PR Validation for Config Repos

```yaml
# .github/workflows/validate.yml
name: Validate GitOps Config
on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Lint YAML
      run: yamllint -c .yamllint.yml .

    - name: Validate Kustomize
      run: |
        for overlay in overlays/*/; do
          echo "Validating $overlay..."
          kustomize build "$overlay" > /dev/null
        done

    - name: Dry-run against cluster schema
      run: |
        kustomize build overlays/production | \
          kubeval --strict --kubernetes-version 1.28.0

    - name: ArgoCD Diff Preview
      run: |
        argocd app diff myapp-prod \
          --revision ${{ github.head_ref }} \
          --local overlays/production
```

---

## Choosing the Right Strategy

| Factor | Branch-Per-Env | Single-Branch + Dir | Tag-Based |
|--------|---------------|-------------------|-----------|
| Team size | Small (< 5) | Medium-Large | Any |
| Env count | 2-3 | 3+ | 3+ |
| Release cadence | Weekly | Continuous | Milestone |
| Promotion model | Merge between branches | Update overlay dirs | Bump tag reference |
| Config drift risk | High | Low | Low |
| Complexity | Low | Medium | Medium |
| Recommended | No | **Yes** | For releases |

---

## Summary Table

| Strategy | Mechanism | Key Benefit | Key Risk |
|----------|-----------|-------------|----------|
| Branch-per-env | Separate branches for dev/staging/prod | Clear separation | Config drift between branches |
| Single-branch + dir | Overlays in directories, main branch only | Easy comparison, no drift | All envs update together |
| Tag-based | Tags gate promotion | Explicit version control | Manual tag management |
| Feature branches | Short-lived PRs merged to main | Reviewed changes | None if short-lived |

---

## Quick Revision Questions

1. **What are the three main branching strategies for GitOps config repos?**
   > Branch-per-environment, single-branch with directory-per-environment, and tag/release-based promotion.

2. **Why is the single-branch + directory model generally recommended?**
   > It avoids config drift between branches, makes it easy to compare environments, and eliminates merge conflicts between environment branches.

3. **How does ArgoCD differentiate environments in the single-branch model?**
   > Each ArgoCD Application points to a different directory path (overlay) within the same branch of the same repo.

4. **What PR validation checks should run on a GitOps config repo?**
   > YAML linting, Kustomize/Helm build validation, schema validation (kubeval), policy checks (OPA/Kyverno), and diff preview.

5. **What is the main risk of the branch-per-environment strategy?**
   > Configuration drift — branches diverge over time, making it hard to know if staging truly matches production.

6. **How does tag-based promotion work in GitOps?**
   > Changes are tagged with version numbers. Each environment's ArgoCD Application is updated to point to the desired tag, providing explicit version control.

---

[← Previous: Repository Structure](01-repository-structure.md) | [Back to README](../README.md) | [Next → Change Management](03-change-management.md)
