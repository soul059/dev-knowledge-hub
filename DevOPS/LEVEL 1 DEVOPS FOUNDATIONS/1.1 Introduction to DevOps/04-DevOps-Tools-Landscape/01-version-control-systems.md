# Chapter 4.1 — Version Control Systems

[← Previous: Monitoring & Observability](../03-DevOps-Practices/06-monitoring-and-observability.md) | [Next: CI/CD Tools →](02-cicd-tools.md)

---

## Overview

**Version Control Systems (VCS)** track changes to files over time, enabling multiple people to collaborate on the same codebase. Git is the industry standard — nearly every DevOps workflow begins and ends with Git.

---

## Types of Version Control

```
┌──────────────────────────────────────────────────────────┐
│  VERSION CONTROL EVOLUTION                               │
│                                                          │
│  1. LOCAL VCS              2. CENTRALIZED VCS            │
│     (RCS)                     (SVN, CVS, Perforce)       │
│                                                          │
│  ┌────────┐               ┌──────────┐                  │
│  │ Local  │               │ Central  │                  │
│  │  DB    │               │ Server   │                  │
│  └────────┘               └────┬─────┘                  │
│  Single machine only       ┌───┼────┐                   │
│  No collaboration          │   │    │                    │
│                            Dev1 Dev2 Dev3                │
│                            (checkout/commit)             │
│                                                          │
│  3. DISTRIBUTED VCS (Git, Mercurial) — MODERN STANDARD  │
│                                                          │
│            ┌───────────┐                                │
│            │  Remote   │ ← GitHub / GitLab / Bitbucket  │
│            │  (origin) │                                │
│            └─────┬─────┘                                │
│           ┌──────┼──────┐                               │
│     ┌─────▼──┐ ┌─▼────┐ ┌▼──────┐                     │
│     │ Full   │ │ Full │ │ Full  │                     │
│     │Clone 1 │ │Clone2│ │Clone3 │                     │
│     └────────┘ └──────┘ └───────┘                     │
│     Dev 1       Dev 2    Dev 3                          │
│     (complete   (works   (offline                       │
│      history)   offline)  commits)                      │
└──────────────────────────────────────────────────────────┘
```

---

## Git Fundamentals

### Git Areas

```
┌──────────────────────────────────────────────────────────┐
│  GIT'S THREE AREAS                                       │
│                                                          │
│  ┌───────────┐  git add  ┌──────────┐ git commit ┌─────┐│
│  │ Working   │──────────►│ Staging  │───────────►│Repo ││
│  │ Directory │           │  Area    │            │(.git)││
│  │           │◄──────────│ (Index)  │◄───────────│     ││
│  └───────────┘ git restore└──────────┘ git reset  └─────┘│
│                                                          │
│  Working Dir: Your actual files                          │
│  Staging:     Files marked for next commit               │
│  Repository:  Committed history (.git folder)            │
│                                                          │
│  Add remote:                                             │
│  ┌──────┐  git push  ┌─────────┐  git pull  ┌──────┐   │
│  │Local │────────────►│ Remote  │────────────►│Local │   │
│  │ Repo │            │(GitHub) │            │ Repo │   │
│  └──────┘◄────────────└─────────┘            └──────┘   │
│           git fetch/pull                                 │
└──────────────────────────────────────────────────────────┘
```

### Essential Git Commands

```bash
# ═══ SETUP ═══
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git init                          # Initialize new repo
git clone <url>                   # Clone existing repo

# ═══ DAILY WORKFLOW ═══
git status                        # Check what's changed
git add .                         # Stage all changes
git add file.txt                  # Stage specific file
git commit -m "feat: add login"   # Commit with message
git push origin main              # Push to remote
git pull origin main              # Pull latest changes

# ═══ BRANCHING ═══
git branch feature/login          # Create branch
git checkout feature/login        # Switch to branch
git checkout -b feature/login     # Create + switch (shortcut)
git branch -d feature/login       # Delete branch
git merge feature/login           # Merge branch into current

# ═══ HISTORY ═══
git log --oneline --graph         # Visual commit history
git diff                          # Show unstaged changes
git diff --staged                 # Show staged changes
git blame file.txt                # Who changed each line

# ═══ UNDO ═══
git restore file.txt              # Discard working changes
git restore --staged file.txt     # Unstage a file
git revert <commit>               # Create undo commit
git reset --hard HEAD~1           # ⚠️ Destroy last commit
git stash                         # Temporarily shelve changes
git stash pop                     # Restore shelved changes
```

---

## Branching Strategies

### Git Flow

```
┌──────────────────────────────────────────────────────────┐
│  GIT FLOW                                                │
│                                                          │
│  main      ──●──────────────●──────────────●──► (releases)│
│              │              ↑              ↑              │
│  release   ──┼──────────●───┘              │              │
│              │          ↑                  │              │
│  develop   ──●──●──●──●─┴──●──●──●──●─────┘              │
│              │  ↑  ↑                ↑                     │
│  feature/a ──┴──┘  │                │                     │
│  feature/b ────────┘                │                     │
│  hotfix    ─────────────────────────┘ (emergency fix)    │
│                                                          │
│  ✅ Good for scheduled releases                          │
│  ⚠️ Complex, many long-lived branches                   │
│  ⚠️ Not ideal for continuous deployment                  │
└──────────────────────────────────────────────────────────┘
```

### Trunk-Based Development

```
┌──────────────────────────────────────────────────────────┐
│  TRUNK-BASED DEVELOPMENT (recommended for DevOps)        │
│                                                          │
│  main   ──●──●──●──●──●──●──●──●──●──●──●──► (always     │
│           ↑     ↑        ↑     ↑              deployable) │
│           │     │        │     │                          │
│  feat/a ──┘     │        │     │  Short-lived branches   │
│  feat/b ────────┘        │     │  (< 1 day)              │
│  feat/c ─────────────────┘     │                          │
│  feat/d ───────────────────────┘                          │
│                                                          │
│  Rules:                                                  │
│  ├── Branches live < 1 day                               │
│  ├── Small, frequent merges                              │
│  ├── Feature flags for incomplete work                   │
│  ├── Main is always deployable                           │
│  └── CI runs on every merge                              │
│                                                          │
│  ✅ Simple, fast feedback, enables CD                    │
│  ✅ Preferred by Google, Meta, Microsoft                 │
└──────────────────────────────────────────────────────────┘
```

| Strategy | Branch Lifespan | Complexity | Best For |
|----------|----------------|------------|----------|
| **Git Flow** | Weeks/months | High | Scheduled releases |
| **GitHub Flow** | Days | Low | Web apps, SaaS |
| **Trunk-Based** | Hours | Minimal | CI/CD, DevOps |
| **GitLab Flow** | Days | Medium | Environment branches |

---

## Conventional Commits

```bash
# Format: <type>(<scope>): <description>

feat(auth): add OAuth2 login support
fix(cart): resolve race condition in checkout
docs(api): update REST API documentation
style(ui): format CSS with prettier
refactor(db): extract query builder module
test(auth): add unit tests for JWT validation
chore(deps): bump lodash from 4.17.20 to 4.17.21
ci(pipeline): add SonarQube analysis step
perf(search): optimize full-text search query

# Breaking change
feat(api)!: change response format to JSON:API
```

---

## Git Platforms Comparison

| Feature | GitHub | GitLab | Bitbucket | Azure DevOps |
|---------|--------|--------|-----------|-------------|
| **CI/CD** | GitHub Actions | GitLab CI/CD | Bitbucket Pipelines | Azure Pipelines |
| **Free Private Repos** | ✅ | ✅ | ✅ | ✅ |
| **Self-Hosted** | Enterprise | Community Edition | Data Center | Server |
| **Code Review** | Pull Requests | Merge Requests | Pull Requests | Pull Requests |
| **Best For** | Open source, startups | Full DevOps platform | Atlassian users | Microsoft shops |
| **Marketplace** | Largest ecosystem | Built-in features | Limited | Moderate |

---

## Real-World Scenario: Code Review Workflow

```
1. Developer creates feature branch:
   git checkout -b feat/user-profile

2. Makes changes, commits with conventional commits:
   git add .
   git commit -m "feat(profile): add user avatar upload"

3. Pushes to remote:
   git push origin feat/user-profile

4. Opens Pull Request (PR) on GitHub:
   ├── Title: "feat: add user avatar upload"
   ├── Description: what, why, how, screenshots
   ├── Reviewers: 2 team members assigned
   └── CI pipeline runs automatically

5. CI checks pass:
   ├── ✅ Unit tests pass
   ├── ✅ Linting clean
   ├── ✅ Security scan clear
   └── ✅ Build successful

6. Code review:
   ├── Reviewer 1: Approved ✅
   └── Reviewer 2: "Add error handling" → changes requested

7. Developer addresses feedback, pushes update

8. Reviewer 2: Approved ✅

9. PR merged to main → auto-deploys to staging
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Merge conflicts | Same lines changed by different people | Pull frequently, use small PRs, communicate with team |
| Accidentally committed secrets | API keys in code | Use `.gitignore`, pre-commit hooks, `git-secrets`, rotate keys ASAP |
| Detached HEAD | Checked out a commit (not branch) | `git checkout main` to reattach |
| Lost commits | `git reset --hard` or deleted branch | `git reflog` to find lost commits (within 30 days) |
| Large repo / slow clone | Binary files or large assets | Use `.gitignore`, Git LFS for large files |
| Wrong commit message | Typo or wrong format | `git commit --amend` (before push) |

---

## Summary Table

| VCS Concept | Description |
|------------|-------------|
| **Git** | Distributed version control system — industry standard |
| **Repository** | Project folder tracked by Git |
| **Commit** | Snapshot of changes with a message |
| **Branch** | Independent line of development |
| **Merge** | Combine changes from two branches |
| **Pull Request** | Request to review and merge code |
| **Trunk-Based Dev** | Short-lived branches, always-deployable main |
| **Conventional Commits** | Standardized commit message format |

---

## Quick Revision Questions

1. **What is the difference between centralized and distributed version control?**
   <details><summary>Answer</summary>Centralized VCS (SVN) has a single central server — developers check out files and must be online to commit. Distributed VCS (Git) gives every developer a full copy of the repository including its history — they can work offline, commit locally, and sync later. Git is faster, supports branching, and has no single point of failure.</details>

2. **What are the three areas in Git, and how do files flow between them?**
   <details><summary>Answer</summary>1) Working Directory — your actual files. 2) Staging Area (Index) — files marked for the next commit via `git add`. 3) Repository (.git) — committed history. Flow: Edit files → `git add` (stage) → `git commit` (save to repo). To undo: `git restore` (unstage) or `git reset`.</details>

3. **Why is trunk-based development preferred for DevOps teams?**
   <details><summary>Answer</summary>Trunk-based development keeps branches short-lived (hours, not weeks), ensuring frequent integration into main. This reduces merge conflicts, enables fast feedback from CI, keeps main always deployable, and supports continuous deployment. It avoids the complexity of long-lived branches in Git Flow.</details>

4. **What are conventional commits, and why use them?**
   <details><summary>Answer</summary>Conventional commits follow a structured format: `type(scope): description`. Types include feat, fix, docs, chore, etc. Benefits: 1) Automated changelog generation. 2) Semantic versioning automation. 3) Consistent, readable history. 4) Easy to filter commits by type.</details>

5. **How would you recover a commit that was accidentally deleted?**
   <details><summary>Answer</summary>Use `git reflog` to see all recent HEAD movements, including deleted commits. Find the commit hash in the reflog, then `git checkout <hash>` or `git cherry-pick <hash>` to recover it. Git keeps unreachable commits for 30 days before garbage collection.</details>

6. **What security practices should be followed with Git repositories?**
   <details><summary>Answer</summary>1) Never commit secrets (API keys, passwords) — use `.gitignore` and tools like `git-secrets`. 2) Use branch protection rules (require reviews, status checks). 3) Sign commits with GPG keys. 4) Enable 2FA on Git platforms. 5) Use pre-commit hooks for security scanning. 6) If secrets are leaked, rotate them immediately.</details>

---

[← Previous: Monitoring & Observability](../03-DevOps-Practices/06-monitoring-and-observability.md) | [Next: CI/CD Tools →](02-cicd-tools.md)
