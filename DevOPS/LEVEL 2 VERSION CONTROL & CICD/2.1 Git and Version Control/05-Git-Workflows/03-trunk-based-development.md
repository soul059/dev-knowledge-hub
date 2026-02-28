# Chapter 5.3: Trunk-Based Development

[← Previous: Gitflow Workflow](02-gitflow-workflow.md) | [Back to README](../README.md) | [Next: GitHub Flow →](04-github-flow.md)

---

## Overview

**Trunk-Based Development (TBD)** is a branching strategy where all developers integrate their work into a single shared branch (the "trunk", typically `main`) **frequently** — at least once per day. It emphasizes small, incremental changes, continuous integration, and avoids long-lived feature branches.

---

## Core Principles

```
┌──────────────────────────────────────────────────────────────┐
│         TRUNK-BASED DEVELOPMENT — OVERVIEW                  │
│                                                              │
│  main (trunk):                                              │
│  ●──●──●──●──●──●──●──●──●──●──●──●──●──●──●──●            │
│     │     │     │        │     │        │                   │
│     A     B     A        C     B        A                   │
│                                                              │
│  Developers A, B, C all commit directly to main             │
│  (or via very short-lived branches — hours, not days)       │
│                                                              │
│  KEY RULES:                                                 │
│  ✅ Commit to main at least once per day                    │
│  ✅ Short-lived branches only (< 1 day)                     │
│  ✅ Small, incremental changes                              │
│  ✅ Feature flags for incomplete features                   │
│  ✅ Comprehensive automated testing (CI)                    │
│  ✅ Main is ALWAYS releasable                               │
│                                                              │
│  ❌ No long-lived feature branches                          │
│  ❌ No develop, release, or staging branches                │
└──────────────────────────────────────────────────────────────┘
```

---

## TBD vs Feature Branch vs Gitflow

```
┌──────────────────────────────────────────────────────────────┐
│         BRANCH LIFESPAN COMPARISON                          │
│                                                              │
│  TRUNK-BASED:                                               │
│  main:  ●──●──●──●──●──●──●──●                              │
│            \/ \/                                            │
│         (tiny branches, merged same day)                     │
│                                                              │
│  FEATURE BRANCH:                                            │
│  main:  ●──────●─────────●──────●                           │
│           \   /   \     /                                   │
│            ●-●     ●-●-●                                    │
│         (days to weeks)                                      │
│                                                              │
│  GITFLOW:                                                   │
│  main:    ●──────────────●──────────●                       │
│  develop: ●──●──●──●──●──●──●──●──●                        │
│              \    /    \     /                               │
│               ●-●      ●-●-●-●                              │
│         (weeks to months for full cycle)                     │
└──────────────────────────────────────────────────────────────┘
```

| Aspect | Trunk-Based | Feature Branch | Gitflow |
|--------|------------|----------------|---------|
| **Branch lifespan** | Hours | Days–weeks | Weeks–months |
| **Integration frequency** | Daily or more | When feature done | At release |
| **Merge conflicts** | Rare, small | Moderate | Large, painful |
| **CI/CD fit** | Excellent | Good | Challenging |
| **Complexity** | Low | Low–Medium | High |
| **Release method** | Continuous deployment | PR merge + deploy | Tagged releases |

---

## How It Works

### Direct Trunk Commits (Small Teams)

```bash
# Pull latest
$ git pull --rebase origin main

# Make small change
$ git add src/utils.js
$ git commit -m "Add input sanitization helper"

# Push immediately
$ git push origin main
```

### Short-Lived Branches (Larger Teams)

```bash
# Create branch (lives < 1 day)
$ git switch -c fix/input-validation
$ git add .
$ git commit -m "Add input validation for email field"

# Push and create PR
$ git push -u origin fix/input-validation
# PR is reviewed and merged within hours

# Branch is deleted after merge
```

```
┌──────────────────────────────────────────────────────────────┐
│  SHORT-LIVED BRANCH IN TRUNK-BASED DEV                      │
│                                                              │
│  main:  ●──●──●──●──●                                       │
│               \  /                                          │
│  branch:       ●    (exists for hours, not days)            │
│                                                              │
│  The branch is:                                             │
│  • Created in the morning                                   │
│  • Pushed and PR created by lunch                           │
│  • Reviewed and merged by end of day                        │
│  • Deleted immediately after merge                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Feature Flags

Since incomplete features can't live in long branches, TBD uses **feature flags** (toggles) to hide work-in-progress from users.

```
┌──────────────────────────────────────────────────────────────┐
│         FEATURE FLAGS                                       │
│                                                              │
│  Code is in main, but hidden behind a flag:                 │
│                                                              │
│  if (featureFlags.newDashboard) {                           │
│    showNewDashboard();     // WIP — only visible to devs    │
│  } else {                                                   │
│    showOldDashboard();     // Live for all users             │
│  }                                                          │
│                                                              │
│  Benefits:                                                  │
│  ✅ Code is integrated daily (no merge pain)                │
│  ✅ Feature can be toggled on for testing                   │
│  ✅ Gradual rollout (10% → 50% → 100% of users)            │
│  ✅ Instant rollback (flip flag off)                        │
│                                                              │
│  Tools: LaunchDarkly, Unleash, custom config                │
└──────────────────────────────────────────────────────────────┘
```

---

## Requirements for Trunk-Based Development

```
┌──────────────────────────────────────────────────────────────┐
│  PREREQUISITES FOR SUCCESSFUL TBD                           │
│                                                              │
│  1. STRONG CI PIPELINE                                      │
│     • Every commit triggers automated tests                 │
│     • Build must pass before merge                          │
│     • Fast feedback (< 10 minutes)                          │
│                                                              │
│  2. COMPREHENSIVE TEST SUITE                                │
│     • Unit, integration, and e2e tests                      │
│     • High code coverage                                    │
│     • Tests must be fast and reliable                       │
│                                                              │
│  3. SMALL, INCREMENTAL CHANGES                              │
│     • Each commit is small and focused                      │
│     • Changes are backward-compatible                       │
│     • Features are decomposed into tiny pieces              │
│                                                              │
│  4. FEATURE FLAGS                                           │
│     • Hide incomplete features from users                   │
│     • Enable gradual rollout                                │
│                                                              │
│  5. CODE REVIEW (for short-lived branches)                  │
│     • Quick reviews (within hours)                          │
│     • Pair programming as alternative                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Advantages and Limitations

| Advantage | Limitation |
|-----------|-----------|
| Minimal merge conflicts | Requires strong CI/CD |
| Fast integration and feedback | Needs feature flags for large features |
| Clean, linear history | Not suitable for all team cultures |
| Best fit for continuous deployment | Requires discipline and small commits |
| Encourages small, reviewable changes | Less isolation for experimental work |

---

## Real-World Scenarios

### Scenario 1: Daily Development Flow

```bash
# Start of day
$ git pull --rebase origin main

# Work on a small task
$ code src/api/users.js
$ git add src/api/users.js
$ git commit -m "Add email format validation to user API"

# Run tests locally
$ npm test

# Push to main
$ git push origin main
# CI pipeline runs automatically
```

### Scenario 2: Using Feature Flags for Large Feature

```bash
# Commit 1: Add data structure (behind flag)
$ git commit -m "Add new dashboard data service (flagged off)"

# Commit 2: Add UI component (behind flag)
$ git commit -m "Add dashboard chart component (flagged off)"

# Commit 3: Enable for beta users
$ git commit -m "Enable new dashboard for beta users"

# All commits go directly to main — feature is hidden until ready
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Change is too big for one commit | Break into smaller, backward-compatible pieces |
| Feature takes weeks | Use feature flags; commit partial work daily |
| Tests take too long | Optimize test suite; parallelize CI |
| Teammate broke main | Fix forward (fix the issue) or revert the commit |
| Hard to review small changes | Context in commit messages; link to overall plan |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Main idea** | Everyone integrates to trunk daily |
| **Branch lifespan** | Hours (or no branches at all) |
| **Permanent branches** | Only `main` |
| **Feature isolation** | Feature flags, not branches |
| **Merge conflicts** | Minimal (small, frequent integrations) |
| **Requires** | Strong CI, tests, discipline |
| **Best for** | Continuous deployment, web apps |
| **Used by** | Google, Meta, Netflix, Amazon |

---

## Quick Revision Questions

1. **What is the core idea behind Trunk-Based Development?**
   <details><summary>Answer</summary>All developers integrate their work into a single shared branch (main/trunk) at least once per day. Long-lived feature branches are avoided, and changes are kept small and incremental.</details>

2. **How do you handle incomplete features in Trunk-Based Development?**
   <details><summary>Answer</summary>Use feature flags (toggles) to hide work-in-progress from end users. Code is committed to main but only activated when the feature is complete. This allows daily integration without exposing unfinished work.</details>

3. **What are the prerequisites for successful Trunk-Based Development?**
   <details><summary>Answer</summary>A strong CI pipeline with fast automated tests, comprehensive test coverage, feature flag infrastructure, discipline to make small incremental changes, and quick code review turnaround.</details>

4. **How does TBD differ from Gitflow?**
   <details><summary>Answer</summary>TBD has one main branch with very short-lived branches (hours). Gitflow has multiple long-lived branches (main, develop, feature, release, hotfix) with branches lasting days to months. TBD favors continuous deployment while Gitflow favors scheduled releases.</details>

5. **When should you NOT use Trunk-Based Development?**
   <details><summary>Answer</summary>When your team lacks CI/CD infrastructure, when tests are slow or unreliable, when your release process requires long stabilization periods, or when supporting multiple production versions simultaneously.</details>

---

[← Previous: Gitflow Workflow](02-gitflow-workflow.md) | [Back to README](../README.md) | [Next: GitHub Flow →](04-github-flow.md)
