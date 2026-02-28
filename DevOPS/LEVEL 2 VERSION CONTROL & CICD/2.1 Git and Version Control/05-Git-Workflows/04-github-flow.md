# Chapter 5.4: GitHub Flow

[← Previous: Trunk-Based Development](03-trunk-based-development.md) | [Back to README](../README.md) | [Next: Choosing the Right Workflow →](05-choosing-the-right-workflow.md)

---

## Overview

**GitHub Flow** is a lightweight, branch-based workflow designed for teams that deploy regularly. It's simpler than Gitflow — there's only one permanent branch (`main`), and all work happens in feature branches that are merged via **Pull Requests**.

---

## The Six Steps of GitHub Flow

```
┌──────────────────────────────────────────────────────────────┐
│          GITHUB FLOW — SIX STEPS                            │
│                                                              │
│  1. CREATE A BRANCH                                         │
│     ┌────────────┐                                          │
│     │ Branch off │                                          │
│     │ from main  │                                          │
│     └─────┬──────┘                                          │
│           ▼                                                 │
│  2. ADD COMMITS                                             │
│     ┌────────────┐                                          │
│     │ Develop &  │                                          │
│     │ commit     │                                          │
│     └─────┬──────┘                                          │
│           ▼                                                 │
│  3. OPEN A PULL REQUEST                                     │
│     ┌────────────┐                                          │
│     │ Push & open│                                          │
│     │ PR for     │                                          │
│     │ discussion │                                          │
│     └─────┬──────┘                                          │
│           ▼                                                 │
│  4. DISCUSS AND REVIEW                                      │
│     ┌────────────┐                                          │
│     │ Team code  │                                          │
│     │ review     │                                          │
│     └─────┬──────┘                                          │
│           ▼                                                 │
│  5. DEPLOY (optional — before merge)                        │
│     ┌────────────┐                                          │
│     │ Deploy from│                                          │
│     │ branch to  │                                          │
│     │ staging    │                                          │
│     └─────┬──────┘                                          │
│           ▼                                                 │
│  6. MERGE                                                   │
│     ┌────────────┐                                          │
│     │ Merge PR   │                                          │
│     │ into main  │                                          │
│     └────────────┘                                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Visual Timeline

```
┌──────────────────────────────────────────────────────────────┐
│   GITHUB FLOW TIMELINE                                      │
│                                                              │
│   main:     ●───●───●────────────────────●───●              │
│                  \                       /                   │
│   feature/:      ●───●───●───●───●─────/                   │
│                  │   │   │   │   │   merge                  │
│                  │   │   │   │   │                          │
│              create  commits  PR  review                    │
│              branch        opened  +                        │
│                                  deploy                     │
│                                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐            │
│  │ Branch │→│ Commit │→│  PR    │→│ Merge  │            │
│  │        │  │        │  │+Review │  │+Deploy │            │
│  └────────┘  └────────┘  └────────┘  └────────┘            │
│                                                              │
│  Main is ALWAYS deployable.                                 │
│  Merging to main = deploying to production.                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Detailed Steps

### Step 1: Create a Branch

```bash
$ git switch main
$ git pull origin main
$ git switch -c feature/user-notifications
```

### Step 2: Add Commits

```bash
$ git add src/notifications.js
$ git commit -m "Add notification service"

$ git add src/notifications.test.js
$ git commit -m "Add notification tests"

# Push early and often
$ git push -u origin feature/user-notifications
```

### Step 3: Open a Pull Request

```
┌──────────────────────────────────────────────────────────────┐
│  On GitHub:                                                  │
│                                                              │
│  Pull Request #42: Add User Notifications                   │
│  feature/user-notifications → main                          │
│                                                              │
│  Description:                                               │
│  - Adds real-time notification service                      │
│  - Includes unit and integration tests                      │
│  - Closes #38                                               │
│                                                              │
│  PRs are for DISCUSSION — you can open one even before      │
│  the code is finished (use "Draft PR" for WIP)              │
└──────────────────────────────────────────────────────────────┘
```

### Step 4: Discuss and Review

```
Team members review code, leave comments, suggest changes.
CI runs automated tests on the PR.
Developer addresses feedback with additional commits.
```

### Step 5: Deploy (Branch Deploy)

```bash
# Some teams deploy the branch to staging/preview before merging
# This verifies the change works in a production-like environment
# If something breaks, main is unaffected
```

### Step 6: Merge

```bash
# On GitHub: Click "Merge pull request"
# Options: merge commit, squash merge, or rebase merge

# After merge, delete the branch
# GitHub has "Delete branch" button on merged PR
```

---

## GitHub Flow vs Gitflow vs Trunk-Based

| Aspect | GitHub Flow | Gitflow | Trunk-Based |
|--------|------------|---------|-------------|
| **Permanent branches** | main only | main + develop | main only |
| **Branch lifespan** | Days | Weeks–months | Hours |
| **Release process** | Merge = deploy | Tagged releases | Continuous deploy |
| **Complexity** | Simple | Complex | Simple |
| **Best for** | Web apps, SaaS | Desktop apps, versioned software | High-velocity teams |
| **PR required?** | Yes, always | Optional | Optional |

---

## Rules of GitHub Flow

```
┌──────────────────────────────────────────────────────────────┐
│        RULES OF GITHUB FLOW                                 │
│                                                              │
│  1. main is ALWAYS deployable                               │
│     • Never commit broken code to main                      │
│     • CI must pass before merge                             │
│                                                              │
│  2. Branch from main for ALL work                           │
│     • Feature, fix, experiment — everything                 │
│     • Use descriptive branch names                          │
│                                                              │
│  3. Push regularly and open PRs early                       │
│     • Don't wait until code is "perfect"                    │
│     • Use Draft PRs for work-in-progress                    │
│                                                              │
│  4. Use Pull Requests for ALL merges                        │
│     • Every change gets reviewed                            │
│     • Discussion happens on the PR                          │
│                                                              │
│  5. Merge only after review + CI                            │
│     • At least one approval                                 │
│     • All CI checks pass                                    │
│                                                              │
│  6. Deploy immediately after merge                          │
│     • (or even before merge, from the branch)               │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Standard Feature Development

```bash
# Day 1
$ git switch -c feature/dark-mode
$ git commit -m "Add dark mode toggle component"
$ git push -u origin feature/dark-mode
# Open Draft PR on GitHub

# Day 2
$ git commit -m "Add dark mode CSS variables"
$ git commit -m "Add dark mode tests"
$ git push
# Mark PR as "Ready for Review"

# Day 3
$ git commit -m "Address review: fix contrast ratio"
$ git push
# Reviewer approves → Merge PR → Delete branch
```

### Scenario 2: Urgent Bug Fix

```bash
$ git switch -c fix/checkout-crash
$ git commit -m "Fix null reference in checkout flow"
$ git push -u origin fix/checkout-crash
# Open PR → Fast review → Merge → Auto-deploy
# Total time: ~1 hour
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| PR has merge conflicts | Rebase on main: `git rebase origin/main && git push --force-with-lease` |
| CI fails on PR | Fix locally, push new commits — PR updates automatically |
| Main has a broken commit | Revert immediately: `git revert <hash>` on main |
| PR review takes too long | Break into smaller PRs; discuss blockers in PR comments |
| Too many branches cluttering remote | Delete branches after merge; use `git fetch --prune` |

---

## Summary Table

| Step | Action | Where |
|------|--------|-------|
| 1. Create branch | `git switch -c feature/x` | Local |
| 2. Commit changes | `git commit` (multiple times) | Local |
| 3. Push + open PR | `git push -u origin feature/x` + GitHub UI | Remote |
| 4. Code review | Team reviews, CI runs | GitHub |
| 5. Deploy (optional) | Deploy branch to staging | Staging server |
| 6. Merge + delete | Merge PR on GitHub, delete branch | Remote |

---

## Quick Revision Questions

1. **How many permanent branches does GitHub Flow have?**
   <details><summary>Answer</summary>Just one — `main`. All other branches are temporary (feature/fix/etc.) and are deleted after merging.</details>

2. **What is the relationship between merging and deploying in GitHub Flow?**
   <details><summary>Answer</summary>In GitHub Flow, merging to main triggers deployment. Main is always deployable, so merging a PR effectively means deploying the change to production. Some teams even deploy the branch before merging to verify it works.</details>

3. **How does GitHub Flow differ from Gitflow?**
   <details><summary>Answer</summary>GitHub Flow has only `main` (no develop, release, or hotfix branches), uses PRs for all merges, and treats merging as deploying. Gitflow has multiple permanent branches and a formal release process. GitHub Flow is simpler and better suited for continuous deployment.</details>

4. **When should you open a Pull Request?**
   <details><summary>Answer</summary>Early — even before the code is finished. Use Draft PRs for work-in-progress. Opening PRs early enables discussion, feedback, and visibility into ongoing work.</details>

5. **What must be true before a PR can be merged in GitHub Flow?**
   <details><summary>Answer</summary>CI checks must pass (automated tests, linting, etc.) and at least one team member must approve the code review. This ensures main always remains stable and deployable.</details>

---

[← Previous: Trunk-Based Development](03-trunk-based-development.md) | [Back to README](../README.md) | [Next: Choosing the Right Workflow →](05-choosing-the-right-workflow.md)
