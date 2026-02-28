# Chapter 5.1: Feature Branch Workflow

[← Previous: Multiple Remotes](../04-Remote-Repositories/06-multiple-remotes.md) | [Back to README](../README.md) | [Next: Gitflow Workflow →](02-gitflow-workflow.md)

---

## Overview

The **Feature Branch Workflow** is the foundation of modern Git collaboration. Every new feature, bug fix, or experiment gets its own dedicated branch. The `main` branch always remains stable and deployable, while all development happens in isolated feature branches.

---

## Core Principle

```
┌──────────────────────────────────────────────────────────────┐
│   FEATURE BRANCH WORKFLOW — CORE IDEA                       │
│                                                              │
│   "main should ALWAYS be deployable."                       │
│   "ALL work happens in feature branches."                   │
│                                                              │
│   main: ●────●────●─────────────●────●────●                 │
│               \                /      \  /                  │
│   feat/A:      ●──●──●───────/        \/                   │
│                              merge    merge                 │
│   feat/B:            ●──●──●──●──●───/                      │
│                                                              │
│   Each feature branch:                                      │
│   ✅ Created from main                                      │
│   ✅ Developed independently                                │
│   ✅ Tested before merging                                  │
│   ✅ Merged back via Pull Request                           │
│   ✅ Deleted after merge                                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Workflow

```
┌──────────────────────────────────────────────────────────────┐
│       FEATURE BRANCH LIFECYCLE                              │
│                                                              │
│   ┌─────────┐     ┌──────────┐     ┌──────────┐            │
│   │ Create  │────►│ Develop  │────►│ Push to  │            │
│   │ Branch  │     │ & Commit │     │ Remote   │            │
│   └─────────┘     └──────────┘     └─────┬────┘            │
│                                          │                  │
│                                          ▼                  │
│   ┌─────────┐     ┌──────────┐     ┌──────────┐            │
│   │ Delete  │◄────│  Merge   │◄────│ Open PR  │            │
│   │ Branch  │     │ to Main  │     │ + Review │            │
│   └─────────┘     └──────────┘     └──────────┘            │
└──────────────────────────────────────────────────────────────┘
```

### 1. Create a Feature Branch

```bash
# Ensure main is up to date
$ git switch main
$ git pull origin main

# Create and switch to feature branch
$ git switch -c feature/user-authentication
```

### 2. Develop on the Feature Branch

```bash
# Make changes
$ code src/auth.js
$ git add src/auth.js
$ git commit -m "Add login form component"

$ code src/auth.test.js
$ git add src/auth.test.js
$ git commit -m "Add login form tests"

# Multiple commits are fine — they tell the story
```

### 3. Keep Branch Updated

```bash
# Periodically sync with main
$ git fetch origin
$ git rebase origin/main
# Or: git merge origin/main
```

### 4. Push and Create Pull Request

```bash
# Push to remote
$ git push -u origin feature/user-authentication

# Create PR on GitHub/GitLab (web UI)
# Request code review from teammates
```

### 5. Address Review Feedback

```bash
# Make changes based on review
$ git add .
$ git commit -m "Address review: add input validation"
$ git push
# PR automatically updates
```

### 6. Merge and Clean Up

```bash
# After PR approval, merge on GitHub (or locally):
$ git switch main
$ git merge --no-ff feature/user-authentication
$ git push origin main

# Delete the feature branch
$ git branch -d feature/user-authentication
$ git push origin --delete feature/user-authentication
```

---

## Branch Naming Conventions

| Prefix | Use Case | Example |
|--------|----------|---------|
| `feature/` | New features | `feature/user-dashboard` |
| `fix/` or `bugfix/` | Bug fixes | `fix/login-crash` |
| `hotfix/` | Urgent production fixes | `hotfix/security-patch` |
| `docs/` | Documentation only | `docs/api-reference` |
| `refactor/` | Code refactoring | `refactor/auth-module` |
| `test/` | Adding tests | `test/payment-validation` |
| `chore/` | Maintenance tasks | `chore/update-dependencies` |

```
┌──────────────────────────────────────────────────────────────┐
│  NAMING BEST PRACTICES                                      │
│                                                              │
│  ✅ feature/JIRA-123-user-login    (ticket + description)   │
│  ✅ fix/null-pointer-on-save       (descriptive)            │
│  ✅ docs/update-api-endpoints      (clear intent)           │
│                                                              │
│  ❌ my-branch                       (vague)                 │
│  ❌ test123                         (meaningless)            │
│  ❌ feature/user_login_page_with_oauth_2.0_and_sso          │
│     (too long)                                              │
│                                                              │
│  Guidelines:                                                │
│  • Use lowercase with hyphens                               │
│  • Include ticket number if using issue tracker             │
│  • Keep under 50 characters                                 │
│  • Use category prefix (feature/, fix/, etc.)               │
└──────────────────────────────────────────────────────────────┘
```

---

## Advantages and Limitations

| Advantage | Limitation |
|-----------|-----------|
| Main is always stable | Simple for small teams only |
| Easy to understand | No release management built-in |
| Isolated development | No staging/develop branch |
| Enables code review via PRs | May need more structure for large projects |
| Easy parallel feature development | No hotfix pipeline |

---

## Real-World Scenarios

### Scenario 1: Two Developers Working on Different Features

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  main:     ●────●────●──────────●──────────●                │
│                  \              /           /                │
│  Alice:          ●──●──●──────/           /                 │
│  (feature/        user auth    merge      /                 │
│   user-auth)                             /                  │
│                      \                  /                   │
│  Bob:                 ●──●──●──●───────/                    │
│  (feature/             payment system   merge               │
│   payments)                                                 │
│                                                              │
│  Alice and Bob work independently.                          │
│  Each merges when their feature is ready.                   │
└──────────────────────────────────────────────────────────────┘
```

### Scenario 2: Handling a Bug Found During Review

```bash
# Reviewer found a bug in your PR
$ git switch feature/user-auth
$ code src/auth.js                  # Fix the bug
$ git add src/auth.js
$ git commit -m "Fix null check in auth validation"
$ git push
# PR is updated automatically — reviewer re-reviews
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Feature branch is behind main | `git fetch origin && git rebase origin/main` |
| Merge conflicts when merging to main | Update feature branch first, resolve conflicts there |
| Accidentally committed to main | Cherry-pick to feature branch, reset main |
| Branch name typo | `git branch -m old-name new-name` to rename |
| PR shows unrelated commits | Rebase on latest main to clean up |

---

## Summary Table

| Step | Action | Command |
|------|--------|---------|
| 1 | Update main | `git switch main && git pull` |
| 2 | Create branch | `git switch -c feature/name` |
| 3 | Develop | `git add . && git commit -m "..."` |
| 4 | Stay updated | `git fetch origin && git rebase origin/main` |
| 5 | Push | `git push -u origin feature/name` |
| 6 | Create PR | GitHub/GitLab UI |
| 7 | Merge | Merge via PR or `git merge --no-ff` |
| 8 | Clean up | `git branch -d feature/name` |

---

## Quick Revision Questions

1. **What is the key rule of the Feature Branch Workflow?**
   <details><summary>Answer</summary>The main branch should always be stable and deployable. All development work happens in dedicated feature branches, which are merged back into main only after review and testing.</details>

2. **Why use `--no-ff` when merging feature branches?**
   <details><summary>Answer</summary>`--no-ff` (no fast-forward) creates a merge commit even when a fast-forward is possible. This preserves the context that a feature branch existed and groups all its commits together, making history easier to understand.</details>

3. **What should you do before creating a new feature branch?**
   <details><summary>Answer</summary>Switch to main and pull the latest changes (`git switch main && git pull origin main`). This ensures your feature branch starts from the most current code.</details>

4. **How do you keep a feature branch in sync with main?**
   <details><summary>Answer</summary>Periodically fetch and rebase: `git fetch origin && git rebase origin/main`. This replays your feature commits on top of the latest main, keeping your branch up to date.</details>

5. **What are good branch naming conventions?**
   <details><summary>Answer</summary>Use a category prefix (feature/, fix/, docs/) followed by a short description with hyphens: `feature/user-login`, `fix/null-pointer`. Include ticket numbers when using issue trackers: `feature/JIRA-123-user-auth`.</details>

---

[← Previous: Multiple Remotes](../04-Remote-Repositories/06-multiple-remotes.md) | [Back to README](../README.md) | [Next: Gitflow Workflow →](02-gitflow-workflow.md)
