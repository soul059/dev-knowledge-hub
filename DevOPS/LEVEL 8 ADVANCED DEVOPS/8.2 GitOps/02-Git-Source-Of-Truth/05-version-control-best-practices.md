# Chapter 2.5: Version Control Best Practices

[← Previous: Audit and Compliance](04-audit-and-compliance.md) | [Back to README](../README.md) | [Next → ArgoCD Architecture](../03-ArgoCD/01-argocd-architecture.md)

---

## Overview

Effective version control practices are the foundation of a reliable GitOps workflow. This chapter covers commit conventions, branch hygiene, tagging strategies, and operational practices that keep your GitOps config repos healthy and maintainable.

---

## Commit Message Conventions

### Conventional Commits Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types for GitOps Config Repos

```
┌──────────────────────────────────────────────────────────────┐
│               COMMIT TYPE CONVENTIONS                        │
│                                                              │
│  Type        When to Use                  Example            │
│  ──────────  ───────────────────────────  ─────────────────  │
│  feat:       New resource or feature      feat(api): add HPA │
│  fix:        Bug fix in config            fix(db): memory    │
│  chore:      Image tag update, routine    chore: update v2.1 │
│  refactor:   Restructure without changes  refactor: move ns  │
│  security:   Security-related change      security: tighten  │
│  docs:       Documentation update         docs: add README   │
│  revert:     Reverting a change           revert: abc1234    │
└──────────────────────────────────────────────────────────────┘
```

### Good vs Bad Commit Messages

```
BAD:
  "update yaml"
  "fix stuff"
  "changes"
  "WIP"

GOOD:
  "feat(frontend): add horizontal pod autoscaler for production"
  "fix(payment-service): increase memory limit to prevent OOMKill"
  "chore(backend): update image tag to v4.2.1 (fixes CVE-2025-1234)"
  "security(rbac): restrict payment-service to payment namespace only"
  "revert: abc1234 — rollback frontend v3.1 due to login failures"
```

---

## Tagging Strategy

```
┌──────────────────────────────────────────────────────────────┐
│                   TAGGING STRATEGY                           │
│                                                              │
│  Semantic Versioning for Releases                            │
│                                                              │
│  Format: v<MAJOR>.<MINOR>.<PATCH>                            │
│                                                              │
│  v1.0.0  ── Initial production release                       │
│  v1.1.0  ── Added new service (minor)                        │
│  v1.1.1  ── Bug fix in config (patch)                        │
│  v2.0.0  ── Breaking change (major)                          │
│                                                              │
│  Tag Commands:                                               │
│  ─────────────                                               │
│  git tag -a v1.2.0 -m "Release 1.2.0: add rate limiting"    │
│  git push origin v1.2.0                                      │
│                                                              │
│  Environment Tags:                                           │
│  ─────────────────                                           │
│  git tag -a staging-2026-02-28 -m "Staging release"          │
│  git tag -a prod-2026-02-28 -m "Production release"          │
└──────────────────────────────────────────────────────────────┘
```

---

## .gitignore for GitOps Repos

```gitignore
# .gitignore for GitOps config repo

# Never commit secrets
*.key
*.pem
*.p12
secret*.yaml
!sealed-secret*.yaml    # Sealed secrets are OK

# Build artifacts
charts/*.tgz
*.tar.gz

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Temporary files
tmp/
*.tmp
*.bak

# Helm chart dependencies (download fresh)
charts/*/charts/
charts/*/tmpcharts/
```

---

## Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.5.0
  hooks:
  - id: trailing-whitespace
  - id: end-of-file-fixer
  - id: check-yaml
    args: ['--allow-multiple-documents']
  - id: check-merge-conflict
  - id: detect-private-key        # Prevent committing secrets!

- repo: https://github.com/adrienverge/yamllint
  rev: v1.33.0
  hooks:
  - id: yamllint
    args: ['-c', '.yamllint.yml']

- repo: local
  hooks:
  - id: kustomize-build
    name: Validate Kustomize Build
    entry: bash -c 'for d in overlays/*/; do kustomize build "$d" > /dev/null || exit 1; done'
    language: system
    pass_filenames: false
```

### YAML Lint Configuration

```yaml
# .yamllint.yml
extends: default
rules:
  line-length:
    max: 120
    level: warning
  indentation:
    spaces: 2
  document-start: disable
  truthy:
    check-keys: false
```

---

## Git Workflow Practices

### Branch Naming Convention

```
┌──────────────────────────────────────────────────────────────┐
│              BRANCH NAMING CONVENTION                        │
│                                                              │
│  Pattern: <type>/<short-description>                         │
│                                                              │
│  feat/add-redis-cache                                        │
│  fix/payment-service-memory                                  │
│  chore/update-nginx-v125                                     │
│  security/restrict-rbac-policies                             │
│  hotfix/revert-broken-deploy                                 │
│  auto/update-image-abc1234    ← CI-generated                 │
│                                                              │
│  Rules:                                                      │
│  • Lowercase only                                            │
│  • Hyphens, not underscores                                  │
│  • Short and descriptive                                     │
│  • Delete after merge                                        │
└──────────────────────────────────────────────────────────────┘
```

### Keeping History Clean

```bash
# Squash commits before merging (recommended for feature branches)
# In GitHub PR: "Squash and merge"

# Result: One clean commit per change in main
# main:
#   abc1234 feat(api): add rate limiting (#45)
#   def5678 chore(frontend): update to v3.2 (#44)
#   ghi9012 fix(db): increase connection pool (#43)

# Interactive rebase to clean up local commits
git rebase -i HEAD~3
# pick abc1234 WIP: start rate limiting
# squash def5678 add tests
# squash ghi9012 finalize config
# → becomes one clean commit
```

---

## Repository Hygiene Checklist

```
  ☐ README.md with repo purpose and structure
  ☐ CODEOWNERS file for review enforcement
  ☐ .gitignore prevents secrets and artifacts
  ☐ .pre-commit-config.yaml validates before commit
  ☐ .yamllint.yml standardizes YAML formatting
  ☐ PR template guides change documentation
  ☐ Branch protection rules on main
  ☐ Signed commits enforced
  ☐ CI validates kustomize build / helm template
  ☐ Stale branches cleaned up regularly
  ☐ Tags for meaningful releases
  ☐ No secrets in plain text (use Sealed Secrets / SOPS)
```

---

## Summary Table

| Practice | Purpose |
|----------|---------|
| Conventional commits | Clear, searchable change history |
| Semantic versioning tags | Explicit release management |
| .gitignore | Prevent secrets and artifacts from being committed |
| Pre-commit hooks | Catch errors before they reach the repo |
| Branch naming | Consistent, readable branch names |
| Squash merge | Clean, one-commit-per-change history on main |
| CODEOWNERS | Enforce review by the right team |
| Signed commits | Verify author identity for compliance |
| Repository hygiene | README, templates, CI validation |

---

## Quick Revision Questions

1. **What is the Conventional Commits format and why is it useful for GitOps?**
   > Format: `type(scope): description`. It creates searchable, standardized commit history that makes audit queries and changelogs easy to generate.

2. **What files should a GitOps config repo's .gitignore include?**
   > Secret files (*.key, *.pem), Helm chart archives, IDE files, OS files, and temp files. Sealed secrets should be excluded from the ignore list.

3. **Why should you squash merge feature branches into main?**
   > It creates a clean, one-commit-per-change history on main, making it easier to revert, audit, and understand the change log.

4. **What do pre-commit hooks validate in a GitOps config repo?**
   > YAML syntax, trailing whitespace, merge conflicts, private key detection, and Kustomize/Helm build validity.

5. **How should branches be named in a GitOps config repo?**
   > Follow `<type>/<short-description>` pattern — lowercase, hyphens, short and descriptive. Delete branches after merging.

6. **What is the purpose of semantic versioning tags in a GitOps context?**
   > They provide explicit version control for releases, enable tag-based environment promotion, and make it easy to pin environments to specific versions.

---

[← Previous: Audit and Compliance](04-audit-and-compliance.md) | [Back to README](../README.md) | [Next → ArgoCD Architecture](../03-ArgoCD/01-argocd-architecture.md)
