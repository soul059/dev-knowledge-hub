# Chapter 5.3: Environment Promotion

[← Previous: Infrastructure Management](02-infrastructure-management.md) | [Back to README](../README.md) | [Next → Secrets Management](04-secrets-management.md)

---

## Overview

**Environment promotion** is the process of moving a release through stages — dev → staging → production. In GitOps, promotion is done through **Git operations** (commits, PRs, tags) rather than pipeline scripts. This chapter covers promotion strategies and their trade-offs.

---

## Promotion Strategies

```
┌──────────────────────────────────────────────────────────────┐
│              PROMOTION STRATEGIES                            │
│                                                              │
│  Strategy 1: Directory-Based (Recommended)                   │
│  ┌──────────────────────────────────────────┐                │
│  │ config-repo/                             │                │
│  │ └── overlays/                            │                │
│  │     ├── dev/        ← image: v1.4.0      │                │
│  │     ├── staging/    ← image: v1.3.0      │                │
│  │     └── production/ ← image: v1.2.0      │                │
│  │                                          │                │
│  │ Promote: copy image tag from dev →       │                │
│  │          staging → production via PR      │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
│  Strategy 2: Branch-Based                                    │
│  ┌──────────────────────────────────────────┐                │
│  │ branch: dev        ← image: v1.4.0      │                │
│  │ branch: staging    ← image: v1.3.0      │                │
│  │ branch: main       ← image: v1.2.0      │                │
│  │                                          │                │
│  │ Promote: merge dev → staging → main      │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
│  Strategy 3: Tag-Based                                       │
│  ┌──────────────────────────────────────────┐                │
│  │ tag: v1.2.0-dev    → deploys to dev      │                │
│  │ tag: v1.2.0-staging → deploys to staging │                │
│  │ tag: v1.2.0        → deploys to prod     │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Strategy 1: Directory-Based Promotion (Recommended)

### Repository Layout

```
config-repo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml      # image: v1.4.0, replicas: 1
    ├── staging/
    │   └── kustomization.yaml      # image: v1.3.0, replicas: 2
    └── production/
        └── kustomization.yaml      # image: v1.2.0, replicas: 5
```

### Overlay Example

```yaml
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
images:
- name: myorg/my-app
  newTag: v1.4.0
patches:
- patch: |
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 1
```

### Promotion Flow

```
┌──────────────────────────────────────────────────────────┐
│         DIRECTORY-BASED PROMOTION                        │
│                                                          │
│  1. CI updates dev overlay:                              │
│     overlays/dev/kustomization.yaml                      │
│     images.newTag: v1.3.0 → v1.4.0                      │
│     (auto-commit by CI or image automation)              │
│     │                                                    │
│     ▼                                                    │
│  2. Dev cluster syncs automatically                      │
│     Tests pass in dev ✓                                  │
│     │                                                    │
│     ▼                                                    │
│  3. Developer opens PR:                                  │
│     "Promote v1.4.0 to staging"                          │
│     Updates overlays/staging/kustomization.yaml          │
│     │                                                    │
│     ▼                                                    │
│  4. PR approved + merged                                 │
│     Staging cluster syncs ✓                              │
│     │                                                    │
│     ▼                                                    │
│  5. Release manager opens PR:                            │
│     "Promote v1.4.0 to production"                       │
│     Updates overlays/production/kustomization.yaml       │
│     │                                                    │
│     ▼                                                    │
│  6. PR approved + merged                                 │
│     Production cluster syncs ✓                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Strategy 2: Branch-Based Promotion

```yaml
# ArgoCD: one Application per branch
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
spec:
  source:
    repoURL: https://github.com/myorg/config-repo.git
    targetRevision: dev          # Track dev branch
    path: deploy
  destination:
    server: https://dev-cluster:6443
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
spec:
  source:
    repoURL: https://github.com/myorg/config-repo.git
    targetRevision: main         # Track main branch
    path: deploy
  destination:
    server: https://prod-cluster:6443
```

Promotion: merge `dev` → `staging` → `main`

---

## Automated Promotion Pipeline

```yaml
# GitHub Actions: auto-promote after tests pass
name: Promote to Staging
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to promote'
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Update staging overlay
      run: |
        cd overlays/staging
        kustomize edit set image myorg/my-app:${{ inputs.version }}

    - name: Create PR
      uses: peter-evans/create-pull-request@v6
      with:
        title: "Promote ${{ inputs.version }} to staging"
        branch: promote/${{ inputs.version }}-staging
        body: |
          Automated promotion of version ${{ inputs.version }}
          
          - [ ] Smoke tests passed in dev
          - [ ] Ready for staging validation
```

### Fully Automated Promotion with Flux

```yaml
# Flux ImagePolicy per environment with different semver ranges
# Dev: latest pre-release
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: my-app-dev
spec:
  imageRepositoryRef:
    name: my-app
  policy:
    semver:
      range: ">=0.0.0-0"           # Include pre-releases
---
# Staging: latest stable minor
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: my-app-staging
spec:
  imageRepositoryRef:
    name: my-app
  policy:
    semver:
      range: ">=1.0.0 <2.0.0"      # Stable 1.x only
---
# Production: manual (no image automation)
# Promoted via PR only
```

---

## Comparison of Promotion Strategies

| Aspect | Directory-Based | Branch-Based | Tag-Based |
|--------|----------------|--------------|-----------|
| Git complexity | Low (single branch) | Medium (multiple branches) | Low |
| PR review | Per-environment PR | Merge PR between branches | Tag creation |
| Diff visibility | Clear per-env diff | Merge conflicts possible | Limited |
| Rollback | Revert per-env commit | Revert merge | Re-tag |
| Automation | Easy to automate | Merge automation needed | Tag automation |
| Recommended | ✓ Yes | For small teams | For release-based |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Directory-based | One branch, overlays per environment (recommended) |
| Branch-based | One branch per environment, promote via merge |
| Tag-based | Tags trigger environment deployments |
| PR-based promotion | Change environment config → PR → review → merge |
| Automated promotion | CI/Flux updates environment configs automatically |
| Image automation | Different ImagePolicy per environment |
| Rollback | `git revert` the promotion commit |

---

## Quick Revision Questions

1. **What is environment promotion in GitOps?**
   > The process of moving a release through environments (dev → staging → production) by updating the desired state in Git for each environment.

2. **Why is directory-based promotion recommended?**
   > It uses a single branch with clear per-environment overlays, avoids merge conflicts, provides clean diffs, and is easy to automate.

3. **How do you promote a version from staging to production?**
   > Open a PR that updates the production overlay's image tag (or Kustomize image reference) to the version validated in staging.

4. **Can promotion be fully automated?**
   > Yes — CI can auto-update dev overlays, and Flux image automation with per-environment ImagePolicies can handle different promotion criteria per stage.

5. **How do you roll back a production promotion?**
   > `git revert` the promotion commit, which restores the previous image tag. The GitOps operator deploys the previous version.

6. **What is the difference between branch-based and directory-based promotion?**
   > Branch-based uses separate branches per environment and promotes via merges. Directory-based uses a single branch with per-environment directories and promotes by updating files in those directories.

---

[← Previous: Infrastructure Management](02-infrastructure-management.md) | [Back to README](../README.md) | [Next → Secrets Management](04-secrets-management.md)
