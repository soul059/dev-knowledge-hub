# Chapter 7.5: Branch Protection Rules

[← Previous: Commit Message Conventions](04-commit-message-conventions.md) | [Back to README](../README.md) | [Next: Semantic Versioning →](06-semantic-versioning.md)

---

## Overview

**Branch protection rules** prevent direct pushes, enforce code review, and require status checks before code can be merged into critical branches like `main` or `develop`. They are a key safeguard in team workflows, supported by GitHub, GitLab, Bitbucket, and Azure DevOps.

---

## Why Protect Branches?

```
┌──────────────────────────────────────────────────────────────┐
│  WITHOUT PROTECTION              WITH PROTECTION            │
│                                                              │
│  • Anyone can push to main       • Changes go through PR    │
│  • No review required            • Review required           │
│  • Tests can be skipped          • CI must pass             │
│  • Force-push can lose history   • Force-push blocked       │
│  • Accidental deletion           • Deletion prevented       │
│                                                              │
│  Result: Broken builds,          Result: Stable main,       │
│  untested code, bugs in prod     reviewed, tested code      │
└──────────────────────────────────────────────────────────────┘
```

---

## GitHub Branch Protection Rules

```
┌──────────────────────────────────────────────────────────────┐
│  GITHUB: Settings → Branches → Branch Protection Rules      │
│                                                              │
│  Branch name pattern: main                                  │
│                                                              │
│  ☑ Require a pull request before merging                    │
│    ☑ Require approvals: 2                                   │
│    ☑ Dismiss stale reviews when new commits push            │
│    ☑ Require review from Code Owners                        │
│                                                              │
│  ☑ Require status checks to pass before merging             │
│    ☑ Require branches to be up to date                      │
│    Status checks: ci/build, ci/test, ci/lint                │
│                                                              │
│  ☑ Require conversation resolution before merging           │
│                                                              │
│  ☑ Require signed commits                                   │
│                                                              │
│  ☑ Require linear history                                   │
│                                                              │
│  ☑ Include administrators                                   │
│                                                              │
│  ☐ Allow force pushes                                       │
│  ☐ Allow deletions                                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Key Protection Settings Explained

### Required Pull Request Reviews

```
┌──────────────────────────────────────────────────────────────┐
│  REQUIRED REVIEWS                                           │
│                                                              │
│  ┌──────────────┐     ┌──────────────┐                      │
│  │  Developer    │────►│  PR Created  │                      │
│  └──────────────┘     └──────┬───────┘                      │
│                              │                              │
│                     ┌────────▼────────┐                     │
│                     │ Requires 2      │                     │
│                     │ approvals       │                     │
│                     └───┬────────┬────┘                     │
│                         │        │                          │
│                  ┌──────▼──┐  ┌──▼──────┐                   │
│                  │Reviewer1│  │Reviewer2│                   │
│                  │Approves✅│  │Approves✅│                   │
│                  └─────────┘  └─────────┘                   │
│                         │        │                          │
│                     ┌───▼────────▼────┐                     │
│                     │ Merge allowed ✅ │                     │
│                     └─────────────────┘                     │
│                                                              │
│  "Dismiss stale reviews" → If author pushes new commits,   │
│  previous approvals are reset and re-review is needed.      │
└──────────────────────────────────────────────────────────────┘
```

### Required Status Checks

```
┌──────────────────────────────────────────────────────────────┐
│  STATUS CHECK FLOW                                          │
│                                                              │
│  PR Opened / Updated                                        │
│       │                                                     │
│       ├──► CI Build    ─── ✅ Pass                           │
│       ├──► CI Tests    ─── ✅ Pass                           │
│       ├──► CI Lint     ─── ❌ Fail → Merge BLOCKED          │
│       └──► Security    ─── ✅ Pass                           │
│                                                              │
│  ALL required checks must pass before merge is allowed!     │
│                                                              │
│  "Require branches to be up to date" means the PR branch   │
│  must include the latest commits from the base branch.      │
└──────────────────────────────────────────────────────────────┘
```

---

## GitLab Protected Branches

```
┌──────────────────────────────────────────────────────────────┐
│  GITLAB: Settings → Repository → Protected Branches         │
│                                                              │
│  Branch: main                                               │
│                                                              │
│  Allowed to merge:   Maintainers                            │
│  Allowed to push:    No one                                 │
│  Allowed to force push: ☐                                   │
│                                                              │
│  Additionally (Merge Request settings):                     │
│  • Required approvals: 2                                    │
│  • Merge method: Fast-forward only                          │
│  • Pipelines must succeed                                   │
│  • All discussions must be resolved                         │
└──────────────────────────────────────────────────────────────┘
```

---

## CODEOWNERS for Automatic Review Assignment

```bash
# .github/CODEOWNERS (GitHub)
# or CODEOWNERS in root (GitLab)

# Global owners
*                       @team-lead @senior-dev

# Frontend
src/components/**       @frontend-team
src/styles/**           @frontend-team

# Backend API
src/api/**              @backend-team
src/models/**           @backend-team @dba

# Infrastructure
Dockerfile              @devops-team
.github/workflows/**    @devops-team
terraform/**            @devops-team

# Security-sensitive files
src/auth/**             @security-team @team-lead
```

```
┌──────────────────────────────────────────────────────────────┐
│  HOW CODEOWNERS WORKS                                       │
│                                                              │
│  PR modifies: src/api/users.js                              │
│                                                              │
│  CODEOWNERS match: src/api/** → @backend-team               │
│                                                              │
│  Result: @backend-team automatically requested for review   │
│                                                              │
│  Combined with "Require review from Code Owners":           │
│  A member of @backend-team MUST approve before merge!       │
└──────────────────────────────────────────────────────────────┘
```

---

## Common Protection Configurations

### Small Team (2-5 developers)

| Setting | Value |
|---------|-------|
| Required approvals | 1 |
| Status checks | Build + test |
| Force push | Disabled |
| Delete branch | Disabled |
| Include admins | Yes |

### Medium Team (5-20 developers)

| Setting | Value |
|---------|-------|
| Required approvals | 2 |
| Status checks | Build + test + lint |
| Dismiss stale reviews | Yes |
| CODEOWNERS required | Yes |
| Require linear history | Optional |
| Signed commits | Optional |

### Large Team / Enterprise

| Setting | Value |
|---------|-------|
| Required approvals | 2-3 |
| Status checks | Build + test + lint + security + coverage |
| CODEOWNERS required | Yes |
| Signed commits | Yes |
| Require linear history | Yes |
| Include admins | Yes |
| Force push | Disabled |
| Branch deletion | Disabled |

---

## Rulesets (GitHub's Newer Feature)

```
┌──────────────────────────────────────────────────────────────┐
│  GITHUB RULESETS (Successor to branch protection)           │
│                                                              │
│  Advantages over classic branch protection:                 │
│  • Apply rules across multiple branches/tags at once        │
│  • Organization-level rules (not just repo)                 │
│  • More granular bypass permissions                         │
│  • Supports tag protection                                  │
│  • Can target branches by name, pattern, or default         │
│                                                              │
│  Settings → Rules → Rulesets → New ruleset                  │
│                                                              │
│  Target: main, release/*                                    │
│  Bypass list: @release-managers                             │
│  Rules: require PR, 2 approvals, status checks             │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Setting Up Protection for a New Project

```
1. Create main branch protection:
   - Require PR with 1 approval
   - Require CI checks to pass
   - Prevent force pushes

2. Set up CODEOWNERS:
   - Assign owners for each module

3. Configure CI pipeline:
   - Build, test, lint checks

4. Communicate rules to team:
   - Document in CONTRIBUTING.md
```

### Scenario 2: Handling an Emergency Hotfix

```
Problem: Branch protection blocks quick fixes

Solutions:
1. Use a "bypass" role for emergencies
2. Create a fast-track review process
3. Allow admin bypass with audit logging
4. Post-merge review for hotfixes
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "Merge blocked" but everything looks approved | Check if a required status check is pending/failed |
| Can't push directly to main | Expected behavior — create a PR instead |
| Admin also blocked | "Include administrators" is enabled — use bypass or disable for emergencies |
| Old approvals dismissed after push | "Dismiss stale reviews" is on — request re-review |
| CODEOWNERS not triggering | Ensure file is at `.github/CODEOWNERS` (GitHub) and syntax is correct |

---

## Summary Table

| Feature | Purpose |
|---------|---------|
| Required PR reviews | Ensure peer review before merge |
| Required status checks | Ensure CI passes before merge |
| Dismiss stale reviews | Re-require review after new pushes |
| CODEOWNERS | Auto-assign reviewers by file path |
| No force push | Prevent history rewriting on protected branches |
| No deletion | Prevent accidental branch removal |
| Signed commits | Verify commit author identity |
| Linear history | Enforce clean, linear commit history |
| Rulesets | Advanced, multi-branch rule management |

---

## Quick Revision Questions

1. **Why should you protect the `main` branch?**
   <details><summary>Answer</summary>To prevent direct pushes of untested/unreviewed code, force-pushes that could lose history, and accidental deletion. Branch protection enforces code review, CI checks, and approval requirements — ensuring main always contains stable, reviewed code.</details>

2. **What does "Dismiss stale pull request approvals" mean?**
   <details><summary>Answer</summary>When this setting is enabled, if the PR author pushes new commits after receiving approvals, those approvals are automatically dismissed. Reviewers must re-review the new changes. This prevents merging unreviewed code that was added after approval.</details>

3. **What is a CODEOWNERS file and where should it be placed?**
   <details><summary>Answer</summary>A CODEOWNERS file maps file paths/patterns to team members or teams who are automatically requested as reviewers when those files are modified. On GitHub, place it at `.github/CODEOWNERS`. Combined with "Require review from Code Owners," it ensures domain experts review their areas.</details>

4. **What are GitHub Rulesets and how do they differ from classic branch protection?**
   <details><summary>Answer</summary>Rulesets are GitHub's newer rule management system. They can apply rules across multiple branches/tags at once, work at the organization level (not just repository), offer more granular bypass permissions, and support tag protection. Classic branch protection works per-branch per-repo.</details>

5. **How should a team handle emergency hotfixes when branch protection is enabled?**
   <details><summary>Answer</summary>Options include: 1) Designate a "bypass" role for emergencies. 2) Use admin bypass with audit logging. 3) Create a fast-track review process where one reviewer can approve urgently. 4) Apply post-merge review for hotfixes. Avoid disabling protection entirely.</details>

---

[← Previous: Commit Message Conventions](04-commit-message-conventions.md) | [Back to README](../README.md) | [Next: Semantic Versioning →](06-semantic-versioning.md)
