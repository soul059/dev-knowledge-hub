# Chapter 5.5: Choosing the Right Workflow

[← Previous: GitHub Flow](04-github-flow.md) | [Back to README](../README.md) | [Next: Interactive Rebase →](../06-Advanced-Git/01-interactive-rebase.md)

---

## Overview

There's no single "best" Git workflow — the right choice depends on your team size, release cadence, deployment strategy, and project complexity. This chapter provides a decision framework to help you choose and adapt the right workflow.

---

## Workflow Comparison Matrix

```
┌──────────────────────────────────────────────────────────────┐
│        WORKFLOW COMPARISON AT A GLANCE                      │
│                                                              │
│           Feature    Gitflow    GitHub    Trunk-             │
│           Branch               Flow      Based              │
│  ─────────────────────────────────────────────────           │
│  Complexity   Low      High      Low      Low               │
│  Branches     Few      Many      Few      Very few          │
│  CI required  Nice     Nice      Yes      Critical          │
│  Deploy freq  Weekly   Monthly   Daily    Continuous         │
│  Team size    1-5      5-20+     2-15     5-50+             │
│  Long branches No      Yes       No       No                │
│  Release tags  No      Yes       No       No                │
│  Feature flags No      No        No       Yes               │
└──────────────────────────────────────────────────────────────┘
```

| Criterion | Feature Branch | Gitflow | GitHub Flow | Trunk-Based |
|-----------|---------------|---------|-------------|-------------|
| **Complexity** | Low | High | Low | Low |
| **Permanent branches** | main | main + develop | main | main |
| **Release process** | Ad-hoc | Formal (release branch) | Merge = deploy | Continuous |
| **CI/CD dependency** | Recommended | Recommended | Required | Critical |
| **Deploy frequency** | Weekly | Monthly/quarterly | Daily | Multiple/day |
| **Team size** | 1–5 | 5–20+ | 2–15 | 5–50+ |
| **Long-lived branches** | No | Yes (develop) | No | No |
| **Feature flags needed** | No | No | No | Yes |
| **Version support** | Single | Multiple | Single | Single |

---

## Decision Flow Chart

```
┌──────────────────────────────────────────────────────────────┐
│        CHOOSING YOUR WORKFLOW — DECISION TREE               │
│                                                              │
│  Do you deploy continuously (multiple times/day)?           │
│  │                                                          │
│  ├── YES ─── Do you have strong CI + feature flags?         │
│  │           │                                              │
│  │           ├── YES ──► TRUNK-BASED DEVELOPMENT            │
│  │           │                                              │
│  │           └── NO ───► GITHUB FLOW                        │
│  │                       (simple, PR-based)                 │
│  │                                                          │
│  └── NO ──── Do you need to support multiple               │
│              production versions (v1, v2, etc.)?            │
│              │                                              │
│              ├── YES ──► GITFLOW                            │
│              │           (release branches + tags)           │
│              │                                              │
│              └── NO ──── Is your team small (≤5)?           │
│                          │                                  │
│                          ├── YES ──► FEATURE BRANCH         │
│                          │           (simple)               │
│                          │                                  │
│                          └── NO ───► GITHUB FLOW            │
│                                      (PR-based review)      │
└──────────────────────────────────────────────────────────────┘
```

---

## Workflow by Project Type

| Project Type | Recommended Workflow | Why |
|-------------|---------------------|-----|
| **Personal project** | Feature Branch | Simple, minimal overhead |
| **Open source** | Fork + GitHub Flow | Contributors don't need write access |
| **SaaS / Web app** | GitHub Flow or Trunk-Based | Continuous deployment |
| **Mobile app** | Gitflow or GitHub Flow | App store releases need staging |
| **Enterprise / Desktop** | Gitflow | Formal release cycles, multiple versions |
| **Microservices** | Trunk-Based | Independent deploy, small services |
| **Startup (small team)** | GitHub Flow | Simple, fast, collaborative |
| **Large team (50+)** | Trunk-Based | Scales with feature flags + CI |

---

## Hybrid Approaches

Many teams combine elements from different workflows:

```
┌──────────────────────────────────────────────────────────────┐
│        COMMON HYBRID APPROACHES                             │
│                                                              │
│  1. GITHUB FLOW + RELEASE TAGS                              │
│     Like GitHub Flow but tag releases on main               │
│     main: ●──●──●──●  (v1.0)  ●──●  (v1.1)                │
│                                                              │
│  2. SIMPLIFIED GITFLOW                                      │
│     main + develop only (no release/hotfix branches)        │
│     develop: ●──●──●──●                                     │
│     main:    ●─────────●  (merge when ready)                │
│                                                              │
│  3. TRUNK-BASED + SHORT PRs                                 │
│     Commit to main via tiny PRs (not direct push)           │
│     Adds code review without long branches                  │
│                                                              │
│  4. ENVIRONMENT BRANCHES                                    │
│     main → staging → production                             │
│     Merge flows forward through environments               │
└──────────────────────────────────────────────────────────────┘
```

---

## Migration Between Workflows

### From Gitflow to GitHub Flow

```bash
# 1. Merge develop into main
$ git switch main
$ git merge develop

# 2. Delete develop
$ git push origin --delete develop
$ git branch -d develop

# 3. Set up branch protection rules on main
# 4. Require PRs for all merges to main
# 5. Set up CI to run on PRs
```

### From Feature Branch to GitHub Flow

```
Add these practices:
1. Require Pull Requests (don't merge locally)
2. Set up CI to run on PRs
3. Require at least one review before merge
4. Deploy from main after merge
```

---

## Factors to Consider

```
┌──────────────────────────────────────────────────────────────┐
│  ASK THESE QUESTIONS TO CHOOSE YOUR WORKFLOW                │
│                                                              │
│  1. How often do you deploy?                                │
│     • Hourly → Trunk-Based                                  │
│     • Daily → GitHub Flow                                   │
│     • Weekly/Monthly → Gitflow                              │
│                                                              │
│  2. How many people are on the team?                        │
│     • Solo/small → Feature Branch                           │
│     • Medium → GitHub Flow                                  │
│     • Large → Trunk-Based or Gitflow                        │
│                                                              │
│  3. How mature is your CI/CD pipeline?                      │
│     • None → Feature Branch                                 │
│     • Basic → GitHub Flow                                   │
│     • Strong → Trunk-Based                                  │
│                                                              │
│  4. Do you need multiple versions in production?            │
│     • Yes → Gitflow                                         │
│     • No → GitHub Flow or Trunk-Based                       │
│                                                              │
│  5. How experienced is your team with Git?                  │
│     • Beginners → Feature Branch                            │
│     • Intermediate → GitHub Flow                            │
│     • Advanced → Trunk-Based                                │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Workflow feels too complex | Simplify — remove unnecessary branches/steps |
| Too many merge conflicts | Integrate more frequently (shorter branch lifespans) |
| Main keeps breaking | Add CI gate, require PR reviews, add tests |
| Releases are chaotic | Add a release branch or tagging process |
| Team isn't following the workflow | Document it, automate enforcement with branch protection |

---

## Summary Table

| Workflow | Best For | Avoid When |
|----------|----------|------------|
| **Feature Branch** | Small teams, simple projects | Large teams needing review gates |
| **Gitflow** | Formal releases, multiple versions | Continuous deployment, small teams |
| **GitHub Flow** | SaaS, web apps, daily deploys | Multiple production versions needed |
| **Trunk-Based** | Large teams, continuous deployment | No CI, no feature flag infrastructure |
| **Fork + PR** | Open source | Internal team projects |

---

## Quick Revision Questions

1. **What factors determine which Git workflow to use?**
   <details><summary>Answer</summary>Key factors include: deployment frequency (continuous vs scheduled), team size, CI/CD maturity, whether multiple production versions are needed, and the team's Git experience level.</details>

2. **When is Gitflow a better choice than GitHub Flow?**
   <details><summary>Answer</summary>When you have scheduled releases, need to support multiple production versions (v1.x, v2.x), have a QA/stabilization period before releases, or are building desktop/mobile apps that go through formal release cycles.</details>

3. **Why is Trunk-Based Development not suitable for all teams?**
   <details><summary>Answer</summary>It requires strong CI/CD pipeline, comprehensive fast test suites, feature flag infrastructure, and disciplined developers who make small incremental commits. Without these prerequisites, main would frequently break.</details>

4. **Can you combine elements from different workflows?**
   <details><summary>Answer</summary>Yes — hybrid approaches are common. Examples: GitHub Flow with release tags, simplified Gitflow without separate release/hotfix branches, or Trunk-Based with short-lived PR branches for code review.</details>

5. **What's the simplest workflow for a solo developer?**
   <details><summary>Answer</summary>Feature Branch Workflow — just main + feature branches. No PR reviews needed (you're alone), no complex branching model. Commit to main directly for small changes, use feature branches for larger changes.</details>

---

[← Previous: GitHub Flow](04-github-flow.md) | [Back to README](../README.md) | [Next: Interactive Rebase →](../06-Advanced-Git/01-interactive-rebase.md)
