# Chapter 7.1: Pull Requests

[← Previous: Git LFS](../06-Advanced-Git/06-git-lfs.md) | [Back to README](../README.md) | [Next: Code Reviews →](02-code-reviews.md)

---

## Overview

A **Pull Request** (PR) — called a **Merge Request** (MR) on GitLab — is a proposal to merge changes from one branch into another. PRs are the primary mechanism for **code review, discussion, and quality assurance** in collaborative Git workflows.

---

## Pull Request Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│       PULL REQUEST LIFECYCLE                                │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌──────────┐    ┌─────────┐ │
│  │ Create  │───►│ Review  │───►│ Approve  │───►│  Merge  │ │
│  │ Branch  │    │ & Disc. │    │ & Tests  │    │ & Close │ │
│  └─────────┘    └─────────┘    └──────────┘    └─────────┘ │
│       │              │              │              │        │
│  • Create feature    │ • Reviewers  │ • All checks │ • Merge│
│    branch            │   comment    │   pass       │   into │
│  • Make commits      │ • Author     │ • Required   │   main │
│  • Push to remote    │   responds   │   approvals  │ • Del. │
│  • Open PR           │ • Iterate    │   met        │   branch│
│                      │   on changes │              │        │
└──────────────────────────────────────────────────────────────┘
```

---

## Creating a Pull Request

### Step 1: Create and Push a Branch

```bash
$ git switch -c feature/user-auth
# Make changes...
$ git add -A
$ git commit -m "feat: add user authentication"
$ git push -u origin feature/user-auth
```

### Step 2: Open the PR (on GitHub/GitLab)

```
┌──────────────────────────────────────────────────────────────┐
│  NEW PULL REQUEST                                           │
│                                                              │
│  base: main  ◄──────  compare: feature/user-auth            │
│                                                              │
│  Title: feat: add user authentication                       │
│                                                              │
│  Description:                                               │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ ## Summary                                             │  │
│  │ Adds JWT-based authentication for users.               │  │
│  │                                                        │  │
│  │ ## Changes                                             │  │
│  │ - Added login/signup endpoints                         │  │
│  │ - Integrated bcrypt for password hashing               │  │
│  │ - Added middleware for token verification              │  │
│  │                                                        │  │
│  │ ## Testing                                             │  │
│  │ - All unit tests passing                               │  │
│  │ - Manual testing on staging                            │  │
│  │                                                        │  │
│  │ Closes #42                                             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Reviewers:  @alice, @bob                                   │
│  Labels:     feature, auth                                  │
│  Milestone:  v2.0                                           │
│                                                              │
│  [Create Pull Request]                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## PR Description Template

```markdown
## Summary
<!-- Brief description of the change -->

## Motivation
<!-- Why is this change needed? Link to issues. -->

## Changes Made
<!-- Bullet list of key changes -->

## Screenshots (if applicable)
<!-- Add screenshots for UI changes -->

## Testing
<!-- How was this tested? -->
- [ ] Unit tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Code follows project style guide
- [ ] Self-reviewed the code
- [ ] Added comments for complex logic
- [ ] Updated documentation
- [ ] No new warnings introduced
```

---

## Merge Strategies in Pull Requests

```
┌──────────────────────────────────────────────────────────────┐
│  THREE MERGE OPTIONS IN GITHUB                              │
│                                                              │
│  1. MERGE COMMIT (default)                                  │
│     ●──●──●──●──M──●    (all commits preserved + merge)    │
│          \     /                                            │
│           ●──●           (feature branch history visible)   │
│                                                              │
│  2. SQUASH AND MERGE                                        │
│     ●──●──●──●──S──●    (all feature commits → 1 commit)   │
│                          (clean linear history)             │
│                                                              │
│  3. REBASE AND MERGE                                        │
│     ●──●──●──●──A'─B'─● (commits replayed linearly)       │
│                          (no merge commit, linear history)  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

| Strategy | Best For | Trade-off |
|----------|----------|-----------|
| Merge commit | Preserving full branch history | Noisy history with merge commits |
| Squash & merge | Clean history, small features | Loses individual commit detail |
| Rebase & merge | Linear history without squash | Can complicate shared branches |

---

## Draft Pull Requests

```
┌──────────────────────────────────────────────────────────────┐
│  DRAFT PR — Work in Progress                                │
│                                                              │
│  Use Draft PRs to:                                          │
│  • Signal work is not ready for review                      │
│  • Get early feedback on approach                           │
│  • Run CI checks before formal review                       │
│  • Share progress with the team                             │
│                                                              │
│  Draft → Ready for Review → Review → Approve → Merge       │
│                                                              │
│  GitHub: Click "Create Draft Pull Request"                  │
│  GitLab: Prefix title with "Draft:" or "WIP:"              │
└──────────────────────────────────────────────────────────────┘
```

---

## Linking Issues

```bash
# In PR description or commit message:
Closes #42          # Closes issue #42 when PR is merged
Fixes #42           # Same effect
Resolves #42        # Same effect

# Multiple issues:
Closes #42, closes #43

# Cross-repo reference:
Fixes owner/repo#42
```

---

## Keeping a PR Up to Date

```bash
# Method 1: Merge main into your feature branch
$ git switch feature/user-auth
$ git fetch origin
$ git merge origin/main
# Resolve conflicts if any
$ git push origin feature/user-auth

# Method 2: Rebase onto main (cleaner history)
$ git switch feature/user-auth
$ git fetch origin
$ git rebase origin/main
# Resolve conflicts if any
$ git push --force-with-lease origin feature/user-auth
```

```
┌──────────────────────────────────────────────────────────────┐
│  KEEPING PR UP TO DATE                                      │
│                                                              │
│  main:     ●──●──●──●──●──●                                │
│                    \                                        │
│  feature:           ●──●──●     (PR opened)                │
│                                                              │
│  New commits on main — PR is behind:                        │
│  main:     ●──●──●──●──●──●──N──N                          │
│                    \                                        │
│  feature:           ●──●──●     (outdated!)                │
│                                                              │
│  After merge main into feature:                             │
│  main:     ●──●──●──●──●──●──N──N                          │
│                    \               \                        │
│  feature:           ●──●──●────────M  (up to date)         │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Small Bug Fix PR

```bash
$ git switch -c fix/login-redirect
$ git add src/auth.js
$ git commit -m "fix: redirect to dashboard after login"
$ git push -u origin fix/login-redirect
# → Open PR with title "fix: redirect to dashboard after login"
# → Reference: "Fixes #87"
# → Reviewer: team lead
# → Usually merged with squash
```

### Scenario 2: Large Feature PR

```bash
# Consider breaking into smaller PRs:
# PR 1: Database schema changes
# PR 2: API endpoints
# PR 3: Frontend integration
# → Each PR is reviewable in isolation
# → Stacked PRs: PR2 depends on PR1, PR3 depends on PR2
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| PR has merge conflicts | Merge or rebase main into your branch, resolve conflicts |
| CI checks failing | Fix the code, push new commits — PR auto-updates |
| PR too large to review | Break into smaller, focused PRs |
| Reviewers not responding | Add a polite comment, reassign if needed |
| Accidentally merged wrong branch | Revert the merge commit on main |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Pull Request | Proposal to merge one branch into another |
| Merge Request | GitLab term for Pull Request |
| Draft PR | Work-in-progress PR, not ready for review |
| Reviewers | Team members assigned to review code |
| Checks/CI | Automated tests that run on PR |
| Squash & Merge | Combines all PR commits into one |
| Closes #N | Links PR to issue, auto-closes on merge |
| Force-with-lease | Safe force push after rebase |

---

## Quick Revision Questions

1. **What is a Pull Request and why is it important?**
   <details><summary>Answer</summary>A Pull Request is a proposal to merge changes from one branch into another. It's important because it enables code review, team discussion, automated testing (CI), and serves as documentation of why changes were made. It's the primary quality gate in collaborative development.</details>

2. **What are the three merge strategies available in GitHub PRs?**
   <details><summary>Answer</summary>1) Merge commit — preserves all commits and creates a merge commit. 2) Squash and merge — combines all PR commits into a single commit. 3) Rebase and merge — replays commits linearly on top of the base branch without a merge commit.</details>

3. **What is a Draft Pull Request?**
   <details><summary>Answer</summary>A Draft PR signals that work is in progress and not yet ready for formal review. It allows running CI checks and getting early feedback on approach. On GitHub, you select "Create Draft Pull Request"; on GitLab, prefix the title with "Draft:" or "WIP:".</details>

4. **How do you link a PR to an issue so the issue auto-closes?**
   <details><summary>Answer</summary>Use keywords like `Closes #42`, `Fixes #42`, or `Resolves #42` in the PR description or commit message. When the PR is merged, the linked issue is automatically closed. For cross-repository references, use `Fixes owner/repo#42`.</details>

5. **How do you update a PR when the base branch has new commits?**
   <details><summary>Answer</summary>Either merge the base branch into your feature branch (`git merge origin/main`) or rebase onto it (`git rebase origin/main`). Merging is simpler; rebasing gives cleaner history but requires `git push --force-with-lease`.</details>

---

[← Previous: Git LFS](../06-Advanced-Git/06-git-lfs.md) | [Back to README](../README.md) | [Next: Code Reviews →](02-code-reviews.md)
