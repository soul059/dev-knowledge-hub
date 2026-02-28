# Chapter 5.2: Gitflow Workflow

[← Previous: Feature Branch Workflow](01-feature-branch-workflow.md) | [Back to README](../README.md) | [Next: Trunk-Based Development →](03-trunk-based-development.md)

---

## Overview

**Gitflow** is a branching model designed by Vincent Driessen that provides a rigid framework for managing releases. It defines specific branch types with clear purposes and strict rules about how they interact. Gitflow is ideal for projects with **scheduled releases** and **multiple versions in production**.

---

## The Gitflow Branch Structure

```
┌──────────────────────────────────────────────────────────────┐
│           GITFLOW BRANCH MODEL                              │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                                                         ││
│  │  main      ●───────────────●─────────────────●          ││
│  │  (prod)    v1.0            v1.1              v2.0       ││
│  │             \             / \               /           ││
│  │  hotfix/    │            / │ ●──●──────────/            ││
│  │             │           /  │  hotfix                    ││
│  │             │          /   │                            ││
│  │  release/   │    ●──●──●   │                            ││
│  │             │   / bug fix  │                            ││
│  │             │  /   only    │                            ││
│  │  develop   ●──●──●──●──●──●──●──●──●──●                ││
│  │             \        /           \   /                   ││
│  │  feature/    ●──●──●/             ●──●                  ││
│  │              feature A           feature B              ││
│  │                                                         ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  TWO PERMANENT BRANCHES: main + develop                     │
│  THREE TEMPORARY BRANCHES: feature, release, hotfix         │
└──────────────────────────────────────────────────────────────┘
```

---

## Branch Types

### Permanent Branches

| Branch | Purpose | Rules |
|--------|---------|-------|
| **main** | Production-ready code | Only receives merges from release and hotfix branches; every commit is tagged with a version |
| **develop** | Integration branch | All features merge here; base for release branches |

### Temporary Branches

| Branch | Created From | Merges Into | Purpose |
|--------|-------------|-------------|---------|
| **feature/** | develop | develop | New feature development |
| **release/** | develop | main + develop | Release preparation, bug fixes only |
| **hotfix/** | main | main + develop | Urgent production fixes |

---

## Workflow Step by Step

### 1. Feature Development

```bash
# Create feature from develop
$ git switch develop
$ git pull origin develop
$ git switch -c feature/shopping-cart

# Develop...
$ git add .
$ git commit -m "Add shopping cart component"

# Merge back to develop
$ git switch develop
$ git merge --no-ff feature/shopping-cart
$ git push origin develop
$ git branch -d feature/shopping-cart
```

```
┌──────────────────────────────────────────────────────────────┐
│  FEATURE BRANCH FLOW                                        │
│                                                              │
│  develop:  ●──●──●──────────●──●                            │
│                  \          /                               │
│  feature/:        ●──●──●──                                │
│                                                              │
│  • Created from: develop                                    │
│  • Merges into: develop                                     │
│  • Naming: feature/description                              │
│  • Deleted after merge                                      │
└──────────────────────────────────────────────────────────────┘
```

### 2. Creating a Release

```bash
# Start release from develop
$ git switch develop
$ git switch -c release/1.2.0

# Only bug fixes and metadata (version numbers, docs)
$ git commit -m "Bump version to 1.2.0"
$ git commit -m "Fix edge case in cart calculation"

# Merge into MAIN
$ git switch main
$ git merge --no-ff release/1.2.0
$ git tag -a v1.2.0 -m "Release version 1.2.0"

# Merge back into DEVELOP (to get bug fixes)
$ git switch develop
$ git merge --no-ff release/1.2.0

# Delete release branch
$ git branch -d release/1.2.0
$ git push origin main develop --tags
```

```
┌──────────────────────────────────────────────────────────────┐
│  RELEASE BRANCH FLOW                                        │
│                                                              │
│  main:      ●──────────────────●  (v1.2.0 tag)             │
│                               / \                           │
│  release/:          ●──●──●──    \                          │
│                    / (fixes)      \                         │
│  develop:   ●──●──●────────────────●                        │
│                                                              │
│  • Created from: develop                                    │
│  • Merges into: main AND develop                            │
│  • Only bug fixes — NO new features                         │
│  • Tag main after merge                                     │
└──────────────────────────────────────────────────────────────┘
```

### 3. Hotfix (Urgent Production Fix)

```bash
# Create hotfix from main
$ git switch main
$ git switch -c hotfix/critical-security-fix

# Fix the issue
$ git commit -m "Patch XSS vulnerability in login form"

# Merge into MAIN
$ git switch main
$ git merge --no-ff hotfix/critical-security-fix
$ git tag -a v1.2.1 -m "Hotfix: security patch"

# Merge into DEVELOP
$ git switch develop
$ git merge --no-ff hotfix/critical-security-fix

# Clean up
$ git branch -d hotfix/critical-security-fix
$ git push origin main develop --tags
```

```
┌──────────────────────────────────────────────────────────────┐
│  HOTFIX BRANCH FLOW                                         │
│                                                              │
│  main:     ●────●────────●                                  │
│            v1.2 \       / v1.2.1                            │
│  hotfix/:        ●──●──                                    │
│                  (fix)  \                                   │
│  develop:  ●──●──●───────●                                  │
│                                                              │
│  • Created from: main                                       │
│  • Merges into: main AND develop                            │
│  • For urgent fixes that can't wait for next release        │
│  • Creates a patch version tag                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Gitflow with git-flow Extension

The `git-flow` tool automates Gitflow commands:

```bash
# Install git-flow
# macOS: brew install git-flow
# Ubuntu: apt install git-flow
# Windows: included with Git for Windows

# Initialize in a repo
$ git flow init
# Prompts for branch naming conventions

# Feature
$ git flow feature start shopping-cart
$ git flow feature finish shopping-cart

# Release
$ git flow release start 1.2.0
$ git flow release finish 1.2.0

# Hotfix
$ git flow hotfix start security-patch
$ git flow hotfix finish security-patch
```

---

## Advantages and Limitations

| Advantage | Limitation |
|-----------|-----------|
| Clear structure for releases | Complex — many branches to manage |
| Parallel development + releases | Overhead for small teams/projects |
| Hotfix pipeline for emergencies | Not suited for continuous deployment |
| Supports multiple production versions | Can slow down fast-moving teams |
| Well-documented and battle-tested | Feature branches can live too long |

---

## When to Use Gitflow

```
┌──────────────────────────────────────────────────────────────┐
│  USE GITFLOW WHEN:                                          │
│  ✅ You have scheduled releases (v1.0, v2.0, etc.)          │
│  ✅ Multiple versions need to be supported                  │
│  ✅ There's a QA/staging phase before release               │
│  ✅ Team is large with defined roles                        │
│                                                              │
│  DON'T USE GITFLOW WHEN:                                    │
│  ❌ You deploy continuously (use trunk-based dev)           │
│  ❌ Small team / solo developer (use feature branch)        │
│  ❌ Simple project without formal releases                  │
│  ❌ Web app with single production version                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Forgot to merge release into develop | Merge now: `git switch develop && git merge release/x.y.z` |
| Feature branch drifted far from develop | Rebase or merge develop into feature regularly |
| Hotfix conflicts with develop | Resolve during the hotfix → develop merge |
| Release branch has too many fixes | It should be bug-fix only — features belong in feature branches |
| Don't know if change is hotfix or feature | Hotfix = urgent prod fix. Feature = everything else. |

---

## Summary Table

| Branch Type | From | Into | Purpose |
|-------------|------|------|---------|
| `main` | — | — | Production code, tagged releases |
| `develop` | main (initial) | — | Integration, next release prep |
| `feature/*` | develop | develop | New features |
| `release/*` | develop | main + develop | Release stabilization |
| `hotfix/*` | main | main + develop | Urgent production fixes |

---

## Quick Revision Questions

1. **What are the two permanent branches in Gitflow?**
   <details><summary>Answer</summary>`main` (production-ready code, tagged with versions) and `develop` (integration branch where features are merged and the next release is prepared).</details>

2. **Where does a release branch get created from, and where does it merge into?**
   <details><summary>Answer</summary>A release branch is created from `develop` and merges into BOTH `main` (for production) and `develop` (to bring back bug fixes made during release preparation).</details>

3. **What is the difference between a hotfix and a regular bug fix in Gitflow?**
   <details><summary>Answer</summary>A hotfix is created from `main` for urgent production issues that can't wait for the next release. A regular bug fix is done on a feature branch or release branch from `develop`. Hotfixes are merged into both main and develop.</details>

4. **When should you NOT use Gitflow?**
   <details><summary>Answer</summary>When you deploy continuously, have a small team, or maintain a web app with only one production version. Gitflow adds overhead that doesn't benefit fast-moving, continuously-deployed projects.</details>

5. **What kind of work is allowed on a release branch?**
   <details><summary>Answer</summary>Only bug fixes, version number bumps, documentation updates, and other release-related metadata. No new features — those must go through feature branches into develop.</details>

---

[← Previous: Feature Branch Workflow](01-feature-branch-workflow.md) | [Back to README](../README.md) | [Next: Trunk-Based Development →](03-trunk-based-development.md)
