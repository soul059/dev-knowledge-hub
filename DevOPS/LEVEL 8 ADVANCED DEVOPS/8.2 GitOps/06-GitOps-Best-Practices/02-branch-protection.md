# Chapter 6.2: Branch Protection

[← Previous: Repository Organization](01-repository-organization.md) | [Back to README](../README.md) | [Next → PR Workflows](03-pr-workflows.md)

---

## Overview

In GitOps, the main branch **is your production state**. Protecting it is as critical as protecting your production infrastructure. Branch protection rules enforce code review, automated checks, and prevent unauthorized changes from reaching your clusters.

---

## Why Branch Protection Matters

```
┌──────────────────────────────────────────────────────────┐
│         UNPROTECTED BRANCH = UNPROTECTED CLUSTER         │
│                                                          │
│  Without protection:                                     │
│  ┌────────────────────────────────────────┐              │
│  │ Developer pushes directly to main      │              │
│  │     │                                  │              │
│  │     ▼                                  │              │
│  │ Bad config goes to production          │              │
│  │     │                                  │              │
│  │     ▼                                  │              │
│  │ Outage! No review, no tests, no audit  │              │
│  └────────────────────────────────────────┘              │
│                                                          │
│  With protection:                                        │
│  ┌────────────────────────────────────────┐              │
│  │ Developer opens PR to main             │              │
│  │     │                                  │              │
│  │     ▼                                  │              │
│  │ CI validates YAML, policies, tests     │              │
│  │     │                                  │              │
│  │     ▼                                  │              │
│  │ Required reviewers approve             │              │
│  │     │                                  │              │
│  │     ▼                                  │              │
│  │ Merge → GitOps deploys safely          │              │
│  └────────────────────────────────────────┘              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## GitHub Branch Protection Rules

```
Repository Settings → Branches → Branch protection rules → Add rule

Branch name pattern: main

┌──────────────────────────────────────────────────────────┐
│              RECOMMENDED SETTINGS                        │
│                                                          │
│  ☑ Require a pull request before merging                 │
│    ☑ Require approvals: 2                                │
│    ☑ Dismiss stale PR approvals on new pushes            │
│    ☑ Require review from Code Owners                     │
│                                                          │
│  ☑ Require status checks to pass before merging          │
│    ☑ Require branches to be up to date                   │
│    Status checks:                                        │
│      ✓ yaml-lint                                         │
│      ✓ kustomize-build                                   │
│      ✓ policy-check                                      │
│      ✓ dry-run                                           │
│                                                          │
│  ☑ Require signed commits                                │
│                                                          │
│  ☑ Require linear history                                │
│    (enforces squash merge or rebase)                     │
│                                                          │
│  ☑ Restrict who can push to matching branches            │
│    Only: release-managers team                           │
│                                                          │
│  ☑ Do not allow bypassing the above settings             │
│    (even admins must follow the rules)                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## GitLab Protected Branches

```
Settings → Repository → Protected branches

┌──────────────────────────────────────────────────────────┐
│  Branch: main                                            │
│  Allowed to merge: Maintainers                           │
│  Allowed to push: No one                                 │
│  Allowed to force push: No                               │
│  Code owner approval required: Yes                       │
│                                                          │
│  Merge request approvals:                                │
│  - Required approvals: 2                                 │
│  - Approval rules:                                       │
│    - Platform team (1 required)                          │
│    - Security team (1 required for prod paths)           │
└──────────────────────────────────────────────────────────┘
```

---

## CODEOWNERS File

Control who must review changes to specific paths:

```
# .github/CODEOWNERS (GitHub) or CODEOWNERS (GitLab)

# Platform team owns infrastructure
/infrastructure/                 @myorg/platform-team
/clusters/                       @myorg/platform-team

# Security team must review production changes
/clusters/production/            @myorg/security-team @myorg/platform-team

# Team-specific ownership
/apps/frontend/                  @myorg/frontend-team
/apps/backend/                   @myorg/backend-team
/tenants/                        @myorg/platform-team

# Everyone must get platform review for cluster configs
/clusters/*/flux-system/         @myorg/platform-team
```

### CODEOWNERS Flow

```
┌──────────────────────────────────────────────────────────┐
│           CODEOWNERS APPROVAL FLOW                       │
│                                                          │
│  PR touches:                                             │
│  └── apps/frontend/deployment.yaml                       │
│  └── clusters/production/apps.yaml                       │
│                                                          │
│  Required reviewers (from CODEOWNERS):                   │
│  ├── @myorg/frontend-team      (owns apps/frontend/)    │
│  ├── @myorg/security-team      (owns clusters/prod/)    │
│  └── @myorg/platform-team      (owns clusters/prod/)    │
│                                                          │
│  All three teams must approve before merge               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Environment-Specific Protection

```
┌──────────────────────────────────────────────────────────┐
│         PER-ENVIRONMENT PROTECTION LEVELS                │
│                                                          │
│  Dev environment:                                        │
│  ├── Branch: main (overlays/dev path)                    │
│  ├── Auto-merge enabled                                  │
│  ├── 1 reviewer required                                 │
│  └── CI checks: yaml-lint only                           │
│                                                          │
│  Staging environment:                                    │
│  ├── Branch: main (overlays/staging path)                │
│  ├── 1 reviewer required                                 │
│  ├── CI checks: lint + kustomize build + dry-run         │
│  └── CODEOWNERS: qa-team                                 │
│                                                          │
│  Production environment:                                 │
│  ├── Branch: main (overlays/production path)             │
│  ├── 2 reviewers required                                │
│  ├── CI checks: lint + build + policy + dry-run          │
│  ├── CODEOWNERS: platform-team + security-team           │
│  └── Signed commits required                             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Required Status Checks

```yaml
# GitHub Actions status checks for GitOps config repo
name: Validate Config
on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: YAML Lint
      run: yamllint -c .yamllint.yaml .

    - name: Kustomize Build (dev)
      run: kustomize build overlays/dev > /dev/null

    - name: Kustomize Build (production)
      run: kustomize build overlays/production > /dev/null

    - name: Kubeval Validation
      run: |
        kustomize build overlays/production | \
          kubeval --kubernetes-version 1.28.0

    - name: Policy Check (OPA/Conftest)
      run: |
        kustomize build overlays/production | \
          conftest test -p policies/ -
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Branch protection | Enforces PR, reviews, and checks before merge |
| Required reviews | Minimum approvals before merging (2 for production) |
| CODEOWNERS | Auto-assigns reviewers based on file paths changed |
| Status checks | CI validates YAML, builds, policies before merge |
| Signed commits | GPG/SSH signatures for commit authenticity |
| Linear history | Squash merge prevents confusing merge commits |
| No bypass | Even admins must follow protection rules |

---

## Quick Revision Questions

1. **Why is branch protection critical in GitOps?**
   > The main branch is the desired state of production. An unreviewed push can directly cause a production outage.

2. **What should required status checks include for a GitOps config repo?**
   > YAML linting, Kustomize/Helm build validation, Kubernetes manifest validation (kubeval), and policy checks (conftest/OPA).

3. **How does CODEOWNERS help in GitOps?**
   > It auto-assigns relevant team reviewers based on which files/paths are changed, ensuring the right people review production, infrastructure, or team-specific changes.

4. **Why require signed commits?**
   > To verify commit authenticity — preventing impersonation and ensuring only verified contributors can change the cluster's desired state.

5. **Should admins be able to bypass branch protection?**
   > No. Enable "Do not allow bypassing" so even admins follow the review and CI process.

---

[← Previous: Repository Organization](01-repository-organization.md) | [Back to README](../README.md) | [Next → PR Workflows](03-pr-workflows.md)
