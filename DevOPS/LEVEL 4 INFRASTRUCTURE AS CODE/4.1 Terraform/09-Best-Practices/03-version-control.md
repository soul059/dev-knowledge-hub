# Chapter 3: Version Control

[← Previous: Naming Conventions](02-naming-conventions.md) | [Back to README](../README.md) | [Next: CI/CD Pipelines →](04-ci-cd-pipelines.md)

---

## Overview

Terraform code must be **version-controlled** like application code. This chapter covers Git workflows, branching strategies, `.gitignore` configuration, and collaboration patterns for infrastructure teams.

---

## 3.1 Git Repository Setup

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM GIT REPO SETUP                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform-infrastructure/                                │
│  ├── .gitignore              ← Critical for security      │
│  ├── .pre-commit-config.yaml ← Automated checks           │
│  ├── .github/                                             │
│  │   └── workflows/                                       │
│  │       └── terraform.yml   ← CI/CD pipeline             │
│  ├── README.md               ← Project documentation      │
│  ├── CODEOWNERS              ← Review requirements        │
│  ├── environments/                                        │
│  │   ├── dev/                                             │
│  │   ├── staging/                                         │
│  │   └── prod/                                            │
│  └── modules/                                             │
│      └── ...                                              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 .gitignore for Terraform

```gitignore
# ─── Essential Terraform .gitignore ──────────────

# Local .terraform directories
**/.terraform/*

# .tfstate files (state should be remote)
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude override files (used for local overrides)
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Exclude CLI configuration files
.terraformrc
terraform.rc

# Exclude tfvars files with secrets
# (but keep non-sensitive example files)
*.auto.tfvars
secret.tfvars
# DO include dev/staging/prod.tfvars if they don't have secrets

# Lock file - INCLUDE THIS (do not ignore)
# .terraform.lock.hcl   ← DO NOT ADD THIS LINE

# Plan output files
*.tfplan
out.plan

# Sensitive variable files
*.env
.env
```

```
┌──────────────────────────────────────────────────────────┐
│        WHAT TO COMMIT vs WHAT TO IGNORE                   │
├─────────────────────────┬────────────────────────────────┤
│  ✓ COMMIT               │  ✗ IGNORE                     │
├─────────────────────────┼────────────────────────────────┤
│ .tf files               │ .terraform/ directory          │
│ .terraform.lock.hcl     │ *.tfstate files                │
│ Non-secret .tfvars      │ *.tfstate.backup               │
│ README.md               │ crash.log                      │
│ .gitignore              │ Secret/sensitive .tfvars       │
│ CI/CD configs           │ override.tf files              │
│ Module code             │ *.tfplan files                 │
│ CODEOWNERS              │ .terraformrc                   │
└─────────────────────────┴────────────────────────────────┘
```

---

## 3.3 Lock File Management

```
┌──────────────────────────────────────────────────────────┐
│       .terraform.lock.hcl BEST PRACTICES                  │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ALWAYS commit .terraform.lock.hcl                        │
│                                                           │
│  Why?                                                     │
│  • Ensures all team members use same provider versions    │
│  • Prevents unexpected upgrades                           │
│  • Contains hashes for integrity verification             │
│                                                           │
│  Updating:                                                │
│  ┌──────────────────────────────────────────────┐        │
│  │ terraform init -upgrade                       │        │
│  │   → Updates providers to latest matching       │        │
│  │   → Regenerates lock file                      │        │
│  │   → Commit the updated lock file               │        │
│  └──────────────────────────────────────────────┘        │
│                                                           │
│  Multi-platform teams:                                    │
│  ┌──────────────────────────────────────────────┐        │
│  │ terraform providers lock \                    │        │
│  │   -platform=linux_amd64 \                     │        │
│  │   -platform=darwin_amd64 \                    │        │
│  │   -platform=darwin_arm64 \                    │        │
│  │   -platform=windows_amd64                     │        │
│  └──────────────────────────────────────────────┘        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.4 Branching Strategy

```
┌──────────────────────────────────────────────────────────┐
│    GITFLOW FOR INFRASTRUCTURE                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  main ──────●──────●──────●──────●──────→  (production)   │
│              \    / \    /      /                          │
│  staging ─────●──●───●──●────●─→           (staging)      │
│                \   /   \   /                              │
│  feature/* ─────●─●─────●─●─→              (development)  │
│                                                           │
│  Branch Naming:                                           │
│  ├── feature/add-rds-cluster                              │
│  ├── fix/security-group-rules                             │
│  ├── refactor/module-restructure                          │
│  └── hotfix/prod-scaling-fix                              │
│                                                           │
│  Rules:                                                   │
│  • feature/* → staging (PR + review)                      │
│  • staging → main (PR + approval + plan review)           │
│  • hotfix/* → main (expedited review)                     │
│  • Never push directly to main                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```
┌──────────────────────────────────────────────────────────┐
│    TRUNK-BASED (simpler alternative)                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  main ──●──●──●──●──●──●──●──●──→                        │
│          \/ \/ \/   \/                                    │
│  short-lived branches (< 1 day)                           │
│                                                           │
│  • All changes via short PRs to main                      │
│  • Feature flags for incomplete work                      │
│  • CD deploys main to dev → staging → prod                │
│  • Best for small teams with strong CI/CD                 │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.5 Pre-commit Hooks

```yaml
# ─── .pre-commit-config.yaml ─────────────────────
repos:
  - repo: https://github.com/antonbabenko/pre-commit-tf
    rev: v1.83.6
    hooks:
      - id: terraform_fmt
        name: Terraform Format
      - id: terraform_validate
        name: Terraform Validate
      - id: terraform_tflint
        name: TFLint
        args:
          - --args=--config=__GIT_WORKING_DIR__/.tflint.hcl
      - id: terraform_docs
        name: Terraform Docs
        args:
          - --hook-config=--path-to-file=README.md
          - --hook-config=--add-to-existing-file=true
          - --args=--config=.terraform-docs.yml
      - id: terraform_trivy
        name: Trivy Security Scan

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-merge-conflict
      - id: detect-private-key
      - id: no-commit-to-branch
        args: ['--branch', 'main']
```

```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run manually on all files
pre-commit run --all-files
```

---

## 3.6 CODEOWNERS

```
# ─── .github/CODEOWNERS ─────────────────────────

# Default owners
*                           @infra-team

# Production requires senior review
environments/prod/          @infra-team-leads

# Security-related changes
modules/iam/                @security-team @infra-team
modules/security/           @security-team @infra-team
**/security*.tf             @security-team

# Network changes
modules/networking/         @network-team @infra-team

# Database changes
modules/database/           @dba-team @infra-team
```

---

## 3.7 Commit Message Standards

```
┌──────────────────────────────────────────────────────────┐
│           COMMIT MESSAGE FORMAT                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  type(scope): short description                           │
│                                                           │
│  Types:                                                   │
│  ├── feat:     New resource or module                     │
│  ├── fix:      Bug fix in configuration                   │
│  ├── refactor: Code restructuring                         │
│  ├── docs:     Documentation changes                      │
│  ├── chore:    Maintenance (upgrades, formatting)         │
│  ├── security: Security-related changes                   │
│  └── ci:       CI/CD pipeline changes                     │
│                                                           │
│  Examples:                                                │
│  ├── feat(networking): add NAT gateway for private subnets│
│  ├── fix(rds): correct backup retention period            │
│  ├── refactor(vpc): extract subnet module                 │
│  ├── chore(providers): upgrade AWS provider to 5.30       │
│  └── security(iam): restrict S3 bucket policy             │
│                                                           │
│  Body (optional):                                         │
│  Include terraform plan summary for significant changes   │
│                                                           │
│  Plan: 3 to add, 1 to change, 0 to destroy              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.8 Pull Request Workflow

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM PR WORKFLOW                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Create Branch                                         │
│     └─ git checkout -b feature/add-monitoring             │
│                                                           │
│  2. Make Changes                                          │
│     └─ Edit .tf files, run fmt/validate locally           │
│                                                           │
│  3. Commit + Push                                         │
│     └─ Pre-commit hooks run automatically                 │
│                                                           │
│  4. Open PR                                               │
│     └─ CI runs: fmt → validate → tflint → plan           │
│                                                           │
│  5. Review                                                │
│     ├─ Reviewer checks plan output                        │
│     ├─ CODEOWNERS auto-assigned                           │
│     └─ Security scan results reviewed                     │
│                                                           │
│  6. Approve + Merge                                       │
│     └─ Merge to main triggers apply (or manual approval)  │
│                                                           │
│  7. Apply                                                 │
│     └─ CD pipeline runs terraform apply                   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Secrets committed to repo | Missing .gitignore entries | Add `*.tfvars` patterns, use `git-secrets` tool |
| State file in repo | .gitignore not configured | Add `*.tfstate*`, migrate to remote state |
| Lock file conflicts | Multiple people ran `init` | Run `terraform providers lock` for all platforms |
| Pre-commit hooks slow | Running on all files | Use `--from-ref` and `--to-ref` to check only changed files |
| PR too large to review | Bundled unrelated changes | One PR per logical change (one module, one resource group) |

---

## Summary Table

| Practice | Implementation | Priority |
|----------|---------------|----------|
| **`.gitignore`** | Exclude state, .terraform/, secrets | Critical |
| **Lock file** | Always commit `.terraform.lock.hcl` | Critical |
| **Branch protection** | Require PR reviews for main | High |
| **Pre-commit hooks** | fmt, validate, tflint, docs | High |
| **CODEOWNERS** | Auto-assign reviewers by path | Medium |
| **Commit messages** | Conventional commits format | Medium |
| **PR templates** | Include plan output checklist | Medium |

---

## Quick Revision Questions

1. **Should you commit `.terraform.lock.hcl` to Git?**
   > Yes, always. It ensures all team members and CI/CD use the same provider versions and includes integrity hashes.

2. **What files must NEVER be committed?**
   > `*.tfstate` files, `.terraform/` directory, sensitive `.tfvars` files containing secrets, `crash.log`, and plan output files.

3. **Why use pre-commit hooks for Terraform?**
   > To automatically run `terraform fmt`, `validate`, linting (TFLint), and security scanning before code is committed, catching errors early.

4. **What branching strategy is recommended for infrastructure?**
   > GitFlow with feature branches and staged promotion (dev → staging → prod), or trunk-based for small teams with strong CI/CD.

5. **What is the purpose of CODEOWNERS?**
   > To automatically assign reviewers based on file paths, ensuring security changes are reviewed by the security team, network changes by the network team, etc.

---

[← Previous: Naming Conventions](02-naming-conventions.md) | [Back to README](../README.md) | [Next: CI/CD Pipelines →](04-ci-cd-pipelines.md)
