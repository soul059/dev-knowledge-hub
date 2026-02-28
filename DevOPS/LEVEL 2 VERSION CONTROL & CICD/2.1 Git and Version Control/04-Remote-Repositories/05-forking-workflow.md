# Chapter 4.5: Forking Workflow

[← Previous: Remote Branches](04-remote-branches.md) | [Back to README](../README.md) | [Next: Multiple Remotes →](06-multiple-remotes.md)

---

## Overview

The **forking workflow** is the standard model for contributing to open-source projects. Instead of pushing directly to the original repository (which requires write access), contributors create a personal **fork** (copy), make changes there, and submit a **pull request** to propose merging their changes.

---

## How Forking Works

```
┌──────────────────────────────────────────────────────────────┐
│            FORKING WORKFLOW — BIG PICTURE                   │
│                                                              │
│  ┌─────────────────────────────────┐                        │
│  │  ORIGINAL REPO (upstream)       │                        │
│  │  github.com/org/project         │                        │
│  │  Maintainers have write access  │                        │
│  └──────────┬──────────────────────┘                        │
│             │                                               │
│             │ Fork (GitHub UI button)                        │
│             ▼                                               │
│  ┌─────────────────────────────────┐                        │
│  │  YOUR FORK (origin)             │                        │
│  │  github.com/YOU/project         │                        │
│  │  You have FULL write access     │                        │
│  └──────────┬──────────────────────┘                        │
│             │                                               │
│             │ git clone                                      │
│             ▼                                               │
│  ┌─────────────────────────────────┐                        │
│  │  LOCAL CLONE                    │                        │
│  │  (your machine)                 │                        │
│  │                                 │                        │
│  │  origin   → your fork          │                        │
│  │  upstream → original repo      │                        │
│  └─────────────────────────────────┘                        │
│                                                              │
│  You push to YOUR fork, then create a Pull Request          │
│  to propose merging into the ORIGINAL repo.                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Forking Workflow

### Step 1: Fork on GitHub/GitLab

```
┌──────────────────────────────────────────────┐
│  On GitHub:                                  │
│  1. Go to github.com/org/project             │
│  2. Click "Fork" button (top right)          │
│  3. Select your account                      │
│  4. Wait for fork to be created              │
│  Result: github.com/YOU/project              │
└──────────────────────────────────────────────┘
```

### Step 2: Clone Your Fork

```bash
$ git clone https://github.com/YOU/project.git
$ cd project
```

### Step 3: Add Upstream Remote

```bash
$ git remote add upstream https://github.com/ORG/project.git

# Verify
$ git remote -v
origin    https://github.com/YOU/project.git (fetch)
origin    https://github.com/YOU/project.git (push)
upstream  https://github.com/ORG/project.git (fetch)
upstream  https://github.com/ORG/project.git (push)
```

### Step 4: Create a Feature Branch

```bash
# Always branch off the latest upstream main
$ git fetch upstream
$ git switch -c fix/typo-in-docs upstream/main
```

### Step 5: Make Changes and Commit

```bash
$ code README.md    # Edit file
$ git add README.md
$ git commit -m "Fix typo in installation section"
```

### Step 6: Push to Your Fork

```bash
$ git push -u origin fix/typo-in-docs
```

### Step 7: Create a Pull Request

```
┌──────────────────────────────────────────────┐
│  On GitHub:                                  │
│  1. Go to YOUR fork or the ORIGINAL repo     │
│  2. Click "Compare & pull request"           │
│  3. Set:                                     │
│     Base: org/project : main                 │
│     Compare: YOU/project : fix/typo-in-docs  │
│  4. Write description of changes             │
│  5. Submit pull request                      │
└──────────────────────────────────────────────┘
```

---

## Complete Workflow Diagram

```
┌──────────────────────────────────────────────────────────────┐
│           FORKING WORKFLOW — COMPLETE FLOW                  │
│                                                              │
│  ┌──────────┐   Pull Request    ┌──────────────┐           │
│  │ YOUR     │ ────────────────► │  ORIGINAL    │           │
│  │ FORK     │                   │  REPO        │           │
│  │ (origin) │ ◄──── Sync ───── │  (upstream)  │           │
│  └────┬─────┘                   └──────────────┘           │
│       │                                                     │
│  push │ ▲ clone                                             │
│       ▼ │                                                   │
│  ┌──────────┐                                               │
│  │  LOCAL   │                                               │
│  │  REPO    │                                               │
│  │          │                                               │
│  │  1. fetch upstream                                       │
│  │  2. create feature branch                                │
│  │  3. commit changes                                       │
│  │  4. push to origin (fork)                                │
│  │  5. create PR on GitHub                                  │
│  └──────────┘                                               │
│                                                              │
│  SYNC YOUR FORK:                                            │
│  $ git fetch upstream                                       │
│  $ git switch main                                          │
│  $ git merge upstream/main                                  │
│  $ git push origin main                                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Keeping Your Fork in Sync

```bash
# Fetch latest from the original repo
$ git fetch upstream

# Update your local main
$ git switch main
$ git merge upstream/main

# Push updated main to your fork
$ git push origin main

# Rebase your feature branch on the updated main
$ git switch fix/typo-in-docs
$ git rebase main
```

```
┌──────────────────────────────────────────────────────────────┐
│          SYNCING YOUR FORK                                  │
│                                                              │
│  upstream/main:  ●──●──●──●──●  (new commits)              │
│                                                              │
│  your main:      ●──●──●        (behind!)                  │
│                                                              │
│  After sync:     ●──●──●──●──●  (up to date)               │
│                                                              │
│  $ git fetch upstream                                       │
│  $ git switch main                                          │
│  $ git merge upstream/main                                  │
│  $ git push origin main                                     │
│                                                              │
│  Now both your local main and origin/main match             │
│  upstream/main.                                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Fork vs Clone vs Branch

| Concept | What It Is | Where It Exists | Use Case |
|---------|-----------|-----------------|----------|
| **Fork** | Full copy of a repo under your account | GitHub/GitLab server | Open-source contributions |
| **Clone** | Local copy of a repo | Your machine | Working on code locally |
| **Branch** | Pointer within a repo | Inside any repo | Feature isolation within a project |

```
┌──────────────────────────────────────────────────────────────┐
│  Fork  = Server-side COPY (different owner, same platform)  │
│  Clone = Client-side COPY (local machine)                   │
│  Branch = Same repo, different line of development           │
│                                                              │
│  You FORK a repo (on GitHub)                                │
│  then CLONE the fork (to your laptop)                       │
│  then create a BRANCH (for your feature)                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: First Open-Source Contribution

```bash
# 1. Fork the repo on GitHub (click Fork)
# 2. Clone your fork
$ git clone https://github.com/YOU/awesome-project.git
$ cd awesome-project

# 3. Set up upstream
$ git remote add upstream https://github.com/org/awesome-project.git

# 4. Create feature branch
$ git switch -c docs/improve-readme

# 5. Make changes, commit, push
$ git add .
$ git commit -m "Improve README with better examples"
$ git push -u origin docs/improve-readme

# 6. Open a Pull Request on GitHub
```

### Scenario 2: PR Needs Updates

```bash
# Reviewer requested changes on your PR
$ git switch docs/improve-readme
# ... make requested changes ...
$ git add .
$ git commit -m "Address review feedback"
$ git push
# PR automatically updates with the new commit
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Fork is behind the original repo | Sync with upstream: `git fetch upstream && git merge upstream/main` |
| PR has merge conflicts | Rebase on latest upstream/main, force-push to your fork |
| "Permission denied" on push | Make sure you're pushing to your fork (origin), not upstream |
| Can't create PR | Ensure you pushed your branch to your fork first |
| Want to update PR without new commit | Amend + force-push: `git commit --amend && git push --force-with-lease` |

---

## Summary Table

| Step | Command/Action |
|------|---------------|
| Fork repo | GitHub UI → Fork button |
| Clone fork | `git clone https://github.com/YOU/repo.git` |
| Add upstream | `git remote add upstream https://github.com/ORG/repo.git` |
| Create feature branch | `git switch -c feature/name` |
| Push to your fork | `git push -u origin feature/name` |
| Create Pull Request | GitHub UI |
| Sync fork with upstream | `git fetch upstream && git merge upstream/main && git push origin main` |

---

## Quick Revision Questions

1. **What is a fork in Git/GitHub?**
   <details><summary>Answer</summary>A fork is a server-side copy of a repository under your own account. It gives you full write access to your copy while the original remains unchanged. Forks are primarily used for contributing to projects where you don't have direct write access.</details>

2. **What is the difference between "origin" and "upstream" in a fork workflow?**
   <details><summary>Answer</summary>"origin" refers to your personal fork (you/project). "upstream" refers to the original repository (org/project). You push changes to origin and sync from upstream.</details>

3. **How do you keep your fork in sync with the original repository?**
   <details><summary>Answer</summary>`git fetch upstream`, then `git switch main`, then `git merge upstream/main`, then `git push origin main`. This updates both your local main and your fork's main with the latest changes from the original repo.</details>

4. **Why should you create a feature branch instead of working directly on main?**
   <details><summary>Answer</summary>Working on main makes it harder to keep your fork synced and creates problems when submitting PRs. Feature branches isolate your changes, allow multiple PRs simultaneously, and keep main clean for syncing with upstream.</details>

5. **What is the difference between a fork and a clone?**
   <details><summary>Answer</summary>A fork is a server-side copy on GitHub/GitLab under your account. A clone is a local copy on your machine. You fork on the server first, then clone the fork to your machine to work on it locally.</details>

---

[← Previous: Remote Branches](04-remote-branches.md) | [Back to README](../README.md) | [Next: Multiple Remotes →](06-multiple-remotes.md)
