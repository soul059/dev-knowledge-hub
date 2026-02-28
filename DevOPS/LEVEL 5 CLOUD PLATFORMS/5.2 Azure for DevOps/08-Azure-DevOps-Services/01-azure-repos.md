# Chapter 1: Azure Repos

## Overview
**Azure Repos** provides Git repositories (and legacy TFVC) for source control in Azure DevOps. It supports unlimited private Git repos, branch policies, pull request workflows, and integrates tightly with Azure Pipelines.

---

## 1.1 Azure Repos Architecture

```
┌──────────── AZURE REPOS ──────────────────────┐
│                                                 │
│   Azure DevOps Organization                     │
│   └── Project: MyApp                            │
│       └── Repos:                                │
│           ├── myapp-frontend (Git)              │
│           ├── myapp-backend (Git)               │
│           └── myapp-infra (Git)                 │
│                                                 │
│   ┌──────────────────────────────────────────┐  │
│   │  Git Repository: myapp-frontend          │  │
│   │                                          │  │
│   │  Branches:                               │  │
│   │  ┌──────────────────────────────┐        │  │
│   │  │ main ──────────────────►     │        │  │
│   │  │   ├─ develop ──────────►     │        │  │
│   │  │   │   ├─ feature/login ►     │        │  │
│   │  │   │   └─ feature/cart  ►     │        │  │
│   │  │   └─ release/v1.2 ────►     │        │  │
│   │  └──────────────────────────────┘        │  │
│   │                                          │  │
│   │  Pull Requests → Code Review → Merge     │  │
│   │                                          │  │
│   └──────────────────────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 1.2 Branch Policies

| Policy | Description |
|--------|-------------|
| **Minimum reviewers** | Require 1-10 approvals before merge |
| **Linked work items** | Require associated work item |
| **Comment resolution** | All PR comments must be resolved |
| **Build validation** | PR must pass a build pipeline |
| **Merge strategy** | Enforce squash merge, rebase, etc. |
| **Auto-reviewers** | Automatically add reviewers by path |

```
┌────── BRANCH POLICY WORKFLOW ──────────────┐
│                                             │
│  Developer                                  │
│     │                                       │
│     ▼                                       │
│  Push to feature/login                      │
│     │                                       │
│     ▼                                       │
│  Create Pull Request → main                 │
│     │                                       │
│     ├──► Build Validation ✅                │
│     ├──► Minimum 2 Reviewers ✅             │
│     ├──► Linked Work Item ✅                │
│     ├──► Comments Resolved ✅               │
│     │                                       │
│     ▼                                       │
│  PR Approved → Merge (squash) → main        │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 1.3 Common Git Commands with Azure Repos

```bash
# Clone repository
git clone https://dev.azure.com/myorg/MyApp/_git/myapp-frontend

# Create feature branch
git checkout -b feature/user-auth

# Stage and commit changes
git add .
git commit -m "feat: add user authentication module"

# Push branch
git push -u origin feature/user-auth

# Pull latest from main
git checkout main
git pull origin main

# Merge feature branch (squash)
git checkout main
git merge --squash feature/user-auth
git commit -m "feat: user authentication (#123)"

# Delete merged branch
git branch -d feature/user-auth
git push origin --delete feature/user-auth
```

---

## 1.4 Pull Request Best Practices

| Practice | Description |
|----------|-------------|
| Small PRs | Easier to review, fewer conflicts |
| Descriptive title | "feat: add payment processing" not "changes" |
| Link work items | Traceability from requirement to code |
| Draft PRs | Share work-in-progress for early feedback |
| Squash merge | Clean commit history on main |
| Auto-complete | Merge automatically when policies pass |

---

## 1.5 Git Branching Strategies

```
┌────── GITFLOW vs TRUNK-BASED ──────────────┐
│                                             │
│  GITFLOW:                                   │
│  main ─────────────────────────────────►    │
│     └─ develop ────────────────────────►    │
│          ├─ feature/a ──┐                   │
│          ├─ feature/b ──┤──► develop        │
│          └─ release/1.0 ──── main           │
│  Good for: longer release cycles            │
│                                             │
│  TRUNK-BASED:                               │
│  main ─────────────────────────────────►    │
│     ├─ short-lived/a ─┐                     │
│     ├─ short-lived/b ─┤──► main (daily)     │
│     └─ short-lived/c ─┘                     │
│  Good for: CI/CD, frequent deployments      │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 1.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot push to main | Branch policy blocking direct push | Create PR instead |
| Auth failure | PAT expired or wrong credentials | Generate new Personal Access Token |
| Merge conflicts | Diverged branches | Rebase feature branch on main |
| PR build failing | Code breaks existing tests | Fix tests before merge |
| Large repo slow | Too many large files | Use Git LFS for large binaries |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Repos** | Unlimited private Git repos in Azure DevOps |
| **Branch Policies** | Enforce reviews, builds, work items on PRs |
| **Pull Requests** | Code review workflow with approval gates |
| **Branching** | GitFlow (longer cycles) or Trunk-Based (CI/CD) |
| **Squash Merge** | Clean history — combines feature commits into one |
| **PAT** | Personal Access Tokens for authentication |

---

## Quick Revision Questions

1. **What are branch policies?**
   > Rules enforced on branches (like main) that require reviewers, passing builds, linked work items, or resolved comments before merging.

2. **What is the difference between GitFlow and Trunk-Based Development?**
   > GitFlow uses long-lived branches (develop, release); Trunk-Based uses short-lived feature branches merged to main daily.

3. **Why use squash merge?**
   > It combines all feature branch commits into a single commit on main, keeping the history clean and readable.

4. **What is a Personal Access Token (PAT)?**
   > A token used for Git authentication instead of a password, with configurable scopes and expiry.

5. **How do branch policies prevent bad code in main?**
   > They block direct pushes and require PRs that must pass build validation, code reviews, and other quality gates.

---

[⬅ Previous: Docker Fundamentals](../07-Container-Services/04-docker-fundamentals.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Pipelines ➡](02-azure-pipelines.md)
