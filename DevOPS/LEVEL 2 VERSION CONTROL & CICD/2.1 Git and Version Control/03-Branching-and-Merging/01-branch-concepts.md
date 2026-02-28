# Chapter 3.1: Branch Concepts

[← Previous: Git Configuration](../02-Git-Basics/05-git-configuration.md) | [Back to README](../README.md) | [Next: Creating and Switching Branches →](02-creating-and-switching-branches.md)

---

## Overview

Branching is Git's **killer feature**. It allows you to diverge from the main line of development, work on features or fixes in isolation, and then merge your changes back. Git branches are lightweight, fast, and fundamental to every modern workflow.

---

## What is a Branch?

A branch is simply a **movable pointer** to a commit. That's it. No file copying, no directory duplication — just a 41-byte file containing a commit hash.

```
┌──────────────────────────────────────────────────────────────┐
│              WHAT A BRANCH REALLY IS                        │
│                                                              │
│  A branch is a POINTER to a commit:                         │
│                                                              │
│  .git/refs/heads/main = a1b2c3d  (just a hash)             │
│  .git/refs/heads/feature = fedcba9                          │
│                                                              │
│  Commits:                                                    │
│   ●───────●───────●───────●───────●                         │
│   C1      C2      C3      C4      C5                        │
│                                    ▲                        │
│                                    │                        │
│                                  main                       │
│                                 (pointer)                   │
│                                                              │
│  Creating a branch = creating a new pointer                 │
│  Cost: ~41 bytes     Time: instant                          │
└──────────────────────────────────────────────────────────────┘
```

---

## HEAD — The "You Are Here" Pointer

```
┌──────────────────────────────────────────────────────────────┐
│                    HEAD POINTER                              │
│                                                              │
│  HEAD tells Git "which branch am I currently on?"           │
│                                                              │
│  Normal State:                                               │
│  HEAD → main → commit C5                                    │
│                                                              │
│  ┌──────┐    ┌──────┐                                       │
│  │ HEAD │───▶│ main │───▶ ● C5                              │
│  └──────┘    └──────┘                                       │
│                                                              │
│  $ cat .git/HEAD                                            │
│  ref: refs/heads/main                                       │
│                                                              │
│  When you switch branches:                                   │
│  $ git checkout feature                                     │
│                                                              │
│  HEAD → feature → commit C3                                 │
│                                                              │
│  ┌──────┐    ┌─────────┐                                    │
│  │ HEAD │───▶│ feature │───▶ ● C3                           │
│  └──────┘    └─────────┘                                    │
│                                                              │
│  HEAD moves, pointing to the new branch                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Branch Divergence

When two branches have different commits, they've **diverged**:

```
┌──────────────────────────────────────────────────────────────┐
│                  BRANCH DIVERGENCE                          │
│                                                              │
│  BEFORE branching:                                           │
│                                                              │
│  ●───────●───────●                                          │
│  C1      C2      C3                                         │
│                   ▲                                         │
│                   │                                         │
│                 main                                        │
│                                                              │
│  AFTER creating "feature" and making commits:               │
│                                                              │
│                   ●───────●        (feature branch)         │
│                  / F1      F2                                │
│  ●───────●───────●                                          │
│  C1      C2      C3───────●───────●  (main branch)         │
│                           M1      M2                        │
│                                    ▲                        │
│                                    │                        │
│                                  main                       │
│                                                              │
│  The branches DIVERGED at C3                                │
│  Each has commits the other doesn't                         │
└──────────────────────────────────────────────────────────────┘
```

---

## Why Branches Are Cheap in Git

| VCS | How Branches Work | Cost |
|-----|-------------------|------|
| **SVN** | Copy entire directory tree | Expensive (time + space) |
| **Git** | Create a 41-byte pointer file | Almost free |

```
┌────────────────────────────────────────────────────────────┐
│           SVN vs GIT BRANCHING                             │
│                                                            │
│  SVN Branch:                                               │
│  ┌────────────────────────────┐                            │
│  │ Copy ALL files to new dir  │                            │
│  │ /trunk/   →  /branches/x/ │                            │
│  │ 1000 files → 1000 files   │                            │
│  │ Time: seconds to minutes   │                            │
│  │ Space: doubles              │                            │
│  └────────────────────────────┘                            │
│                                                            │
│  Git Branch:                                               │
│  ┌────────────────────────────┐                            │
│  │ echo "abc123" >            │                            │
│  │   .git/refs/heads/branch  │                            │
│  │ 1 file, 41 bytes          │                            │
│  │ Time: instant              │                            │
│  │ Space: ~41 bytes           │                            │
│  └────────────────────────────┘                            │
│                                                            │
│  → Git encourages HEAVY branch use                        │
└────────────────────────────────────────────────────────────┘
```

---

## Common Branch Types

```
┌─────────────────────────────────────────────────────────────┐
│              COMMON BRANCH NAMING CONVENTIONS               │
│                                                             │
│  main (or master)                                           │
│  │    Production-ready code                                 │
│  │                                                          │
│  ├── develop                                                │
│  │    Integration branch for features                       │
│  │                                                          │
│  ├── feature/user-auth                                      │
│  │    New feature development                               │
│  │                                                          │
│  ├── bugfix/login-crash                                     │
│  │    Bug fix branch                                        │
│  │                                                          │
│  ├── hotfix/security-patch                                  │
│  │    Urgent production fix                                 │
│  │                                                          │
│  ├── release/v2.0                                           │
│  │    Release preparation                                   │
│  │                                                          │
│  └── experiment/new-algorithm                               │
│       Experimental work                                     │
│                                                             │
│  Convention: type/description (kebab-case)                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Real-World Branching Model

```
┌──────────────────────────────────────────────────────────────┐
│             TYPICAL PROJECT BRANCHES                        │
│                                                              │
│  main ────●────●────●────●─────────●────●──── (stable)      │
│           │         ▲              ▲    ▲                    │
│           │         │              │    │                    │
│  feature  │    ●────●              │    │                    │
│  /auth    └───/  (merged)          │    │                    │
│                                    │    │                    │
│  feature       ●────●────●────●────┘    │                    │
│  /payments    (merged after review)     │                    │
│                                         │                    │
│  hotfix                           ●─────┘                   │
│  /fix-crash                    (emergency fix)              │
│                                                              │
│  Branches are created, worked on, merged, deleted            │
│  Main branch always stays stable                            │
└──────────────────────────────────────────────────────────────┘
```

---

## Branch Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│              BRANCH BEST PRACTICES                          │
│                                                              │
│  1. KEEP BRANCHES SHORT-LIVED                               │
│     └─ Days, not weeks. Long branches = merge pain          │
│                                                              │
│  2. ONE PURPOSE PER BRANCH                                  │
│     └─ feature/login, not feature/login-and-also-payments   │
│                                                              │
│  3. USE DESCRIPTIVE NAMES                                   │
│     └─ feature/user-authentication > feature/stuff          │
│                                                              │
│  4. DELETE AFTER MERGING                                    │
│     └─ Keep the branch list clean                           │
│                                                              │
│  5. PULL FROM MAIN REGULARLY                                │
│     └─ Stay in sync, reduce merge conflicts                 │
│                                                              │
│  6. PROTECT MAIN BRANCH                                     │
│     └─ No direct commits — use PRs                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Too many stale branches | `git branch --merged` to find merged branches, then delete them |
| "Detached HEAD" state | You checked out a commit, not a branch. `git checkout -b <name>` to fix |
| Wrong branch name | `git branch -m old-name new-name` to rename |
| Branch exists locally but not remote | `git push -u origin branch-name` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Branch** | A movable pointer to a commit (~41 bytes) |
| **HEAD** | Pointer to the currently checked-out branch |
| **main/master** | Default primary branch |
| **Divergence** | When two branches have different commits |
| **Branching in Git** | Instant, cheap, creates only a pointer file |
| **Branch naming** | Convention: `type/description` (e.g., `feature/login`) |
| **Short-lived branches** | Best practice: merge quickly, delete after |

---

## Quick Revision Questions

1. **What is a Git branch at the implementation level?**
   <details><summary>Answer</summary>A branch is a lightweight movable pointer — a small file (~41 bytes) in `.git/refs/heads/` that contains the SHA-1 hash of the commit it points to.</details>

2. **What is HEAD and what does it tell Git?**
   <details><summary>Answer</summary>HEAD is a pointer that tells Git which branch you're currently on. It's stored in `.git/HEAD` and typically contains a reference like `ref: refs/heads/main`.</details>

3. **Why is Git branching considered "cheap" compared to SVN?**
   <details><summary>Answer</summary>Git creates a branch by writing a 41-byte file (a commit hash). SVN creates a branch by copying the entire directory tree, which is much slower and uses more space.</details>

4. **What happens to the commit history when two branches diverge?**
   <details><summary>Answer</summary>Each branch accumulates its own commits independently. They share a common ancestor (the commit where they split), and each has commits the other doesn't.</details>

5. **Name four common branch type prefixes and their purposes.**
   <details><summary>Answer</summary>`feature/` for new features, `bugfix/` for bug fixes, `hotfix/` for urgent production fixes, and `release/` for release preparation.</details>

6. **What is "detached HEAD" state and how do you fix it?**
   <details><summary>Answer</summary>Detached HEAD occurs when HEAD points directly to a commit instead of a branch reference. Fix it by creating a branch: `git checkout -b new-branch-name`.</details>

---

[← Previous: Git Configuration](../02-Git-Basics/05-git-configuration.md) | [Back to README](../README.md) | [Next: Creating and Switching Branches →](02-creating-and-switching-branches.md)
