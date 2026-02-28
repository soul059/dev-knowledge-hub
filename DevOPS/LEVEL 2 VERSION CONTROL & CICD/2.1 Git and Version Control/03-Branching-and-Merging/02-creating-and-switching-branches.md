# Chapter 3.2: Creating and Switching Branches

[← Previous: Branch Concepts](01-branch-concepts.md) | [Back to README](../README.md) | [Next: Merging Strategies →](03-merging-strategies.md)

---

## Overview

This chapter covers the practical commands for creating, listing, switching, renaming, and deleting branches — the daily operations you'll use constantly in any Git-based workflow.

---

## Creating Branches

```bash
# Create a new branch (stays on current branch)
$ git branch feature/login

# Create AND switch to the new branch
$ git checkout -b feature/login
# Modern syntax (Git 2.23+):
$ git switch -c feature/login

# Create a branch from a specific commit
$ git branch feature/login abc1234

# Create a branch from another branch
$ git branch feature/login develop

# Create a branch from a tag
$ git branch hotfix/v1-fix v1.0.0
```

```
┌──────────────────────────────────────────────────────────────┐
│           CREATING A BRANCH                                 │
│                                                              │
│  BEFORE: git branch feature/login                           │
│                                                              │
│  ●───────●───────●───────●                                  │
│  C1      C2      C3      C4                                 │
│                           ▲                                 │
│                           │                                 │
│                         main ← HEAD                         │
│                                                              │
│  AFTER: git branch feature/login                            │
│                                                              │
│  ●───────●───────●───────●                                  │
│  C1      C2      C3      C4                                 │
│                           ▲                                 │
│                           ├── main ← HEAD                   │
│                           └── feature/login                 │
│                                                              │
│  Both branches point to the same commit                     │
│  HEAD still points to main (we didn't switch)               │
└──────────────────────────────────────────────────────────────┘
```

---

## Switching Branches

```bash
# Switch to an existing branch
$ git checkout feature/login
# Modern syntax (Git 2.23+):
$ git switch feature/login

# Switch back to the previous branch
$ git checkout -
# or
$ git switch -

# Switch to a remote tracking branch
$ git checkout origin/feature/login
# This creates a detached HEAD — better to do:
$ git checkout -b feature/login origin/feature/login
# or:
$ git switch feature/login  # Auto-tracks if remote branch exists
```

```
┌──────────────────────────────────────────────────────────────┐
│           SWITCHING BRANCHES                                │
│                                                              │
│  BEFORE: git switch feature/login                           │
│                                                              │
│  ●───────●───────●───────●                                  │
│  C1      C2      C3      C4                                 │
│                           ▲                                 │
│                           ├── main ← HEAD                   │
│                           └── feature/login                 │
│                                                              │
│  AFTER: git switch feature/login                            │
│                                                              │
│  ●───────●───────●───────●                                  │
│  C1      C2      C3      C4                                 │
│                           ▲                                 │
│                           ├── main                          │
│                           └── feature/login ← HEAD          │
│                                                              │
│  HEAD now points to feature/login                           │
│  Working directory updated to match                         │
└──────────────────────────────────────────────────────────────┘
```

### What Happens When You Switch

1. HEAD is updated to point to the new branch
2. The staging area is updated to match the new branch's latest commit
3. Working directory files are changed to match

**Important:** Git won't let you switch if you have uncommitted changes that would conflict with the target branch. Options:
```bash
# Option 1: Commit your changes
$ git commit -am "WIP: save progress"

# Option 2: Stash your changes
$ git stash
$ git switch other-branch
# ... do work ...
$ git switch original-branch
$ git stash pop

# Option 3: Force switch (DISCARDS changes — dangerous!)
$ git checkout -f other-branch
```

---

## `checkout` vs `switch` vs `restore`

```
┌──────────────────────────────────────────────────────────────┐
│        THE EVOLUTION OF BRANCH COMMANDS                     │
│                                                              │
│  OLD (git checkout does EVERYTHING):                        │
│    git checkout <branch>       → switch branches            │
│    git checkout -b <branch>    → create + switch            │
│    git checkout -- <file>      → restore file               │
│    git checkout <hash>         → detached HEAD              │
│                                                              │
│  NEW (Git 2.23+, separated concerns):                       │
│    git switch <branch>         → switch branches            │
│    git switch -c <branch>      → create + switch            │
│    git restore <file>          → restore file               │
│    git restore --staged <file> → unstage file               │
│                                                              │
│  checkout still works, but switch/restore are clearer       │
│                                                              │
│  ┌──────────────┐           ┌─────────────┐                 │
│  │ git checkout │    →      │ git switch  │  (branches)     │
│  │   (does a    │    →      │ git restore │  (files)        │
│  │   lot of     │           └─────────────┘                 │
│  │   things)    │                                           │
│  └──────────────┘                                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Listing Branches

```bash
# List local branches (* = current)
$ git branch
  develop
* main
  feature/login
  feature/payments

# List remote branches
$ git branch -r
  origin/main
  origin/develop
  origin/feature/login

# List ALL branches (local + remote)
$ git branch -a

# List with last commit info
$ git branch -v
  develop       abc1234 Add navigation
* main          def5678 Merge feature/auth
  feature/login 9876543 Add login form

# List branches containing a specific commit
$ git branch --contains abc1234

# List merged branches (safe to delete)
$ git branch --merged
  feature/login
  bugfix/typo

# List unmerged branches
$ git branch --no-merged
  feature/payments
```

---

## Renaming Branches

```bash
# Rename the CURRENT branch
$ git branch -m new-name

# Rename a specific branch
$ git branch -m old-name new-name

# Rename on remote (delete old, push new)
$ git push origin --delete old-name
$ git push -u origin new-name
```

---

## Deleting Branches

```bash
# Delete a merged branch (safe)
$ git branch -d feature/login
# Git will REFUSE if the branch isn't merged

# Force delete an unmerged branch (data loss risk!)
$ git branch -D feature/experiment

# Delete a remote branch
$ git push origin --delete feature/login
# or
$ git push origin :feature/login

# Clean up stale remote tracking branches
$ git fetch --prune
# or
$ git remote prune origin
```

```
┌──────────────────────────────────────────────────────────────┐
│           DELETING BRANCHES SAFELY                          │
│                                                              │
│  $ git branch -d feature/login                              │
│                                                              │
│  IF MERGED:                                                  │
│  ✅ Deleted branch feature/login (was abc1234).             │
│                                                              │
│  IF NOT MERGED:                                              │
│  ❌ error: The branch 'feature/login' is not fully merged.  │
│     If you are sure you want to delete it, run              │
│     'git branch -D feature/login'.                          │
│                                                              │
│  -d = safe delete (refuses if not merged)                   │
│  -D = force delete (deletes regardless)                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Starting Work on a New Feature

```bash
# Make sure you're on main and up to date
$ git switch main
$ git pull origin main

# Create feature branch
$ git switch -c feature/user-dashboard

# ... work on the feature ...
$ git add .
$ git commit -m "feat: add dashboard layout"
$ git push -u origin feature/user-dashboard
```

### Scenario 2: Quick Switch to Fix a Bug

```bash
# You're working on a feature, but need to fix a bug
$ git stash                          # Save current work
$ git switch main                    # Go to main
$ git switch -c hotfix/fix-crash     # Create hotfix branch
# ... fix the bug ...
$ git commit -am "fix: resolve crash on login"
$ git push -u origin hotfix/fix-crash
$ git switch feature/user-dashboard  # Back to feature
$ git stash pop                      # Restore saved work
```

### Scenario 3: Cleaning Up Old Branches

```bash
# Find all merged branches
$ git branch --merged main
  feature/login
  bugfix/old-fix
  feature/done-feature

# Delete them
$ git branch -d feature/login bugfix/old-fix feature/done-feature

# Clean up remote tracking refs
$ git fetch --prune
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Can't switch — uncommitted changes | Commit, stash, or discard changes first |
| "Detached HEAD" after checkout | You checked out a commit/tag. Create a branch: `git switch -c name` |
| Deleted branch by mistake | `git reflog` to find the commit, then `git branch <name> <hash>` |
| Branch not showing up | Check `git branch -a` — might be remote only |
| Too many branches | Use `git branch --merged` to find and delete merged ones |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git branch <name>` | Create a branch (don't switch) |
| `git switch -c <name>` | Create and switch to a branch |
| `git checkout -b <name>` | Create and switch (older syntax) |
| `git switch <name>` | Switch to existing branch |
| `git checkout <name>` | Switch to branch (older syntax) |
| `git switch -` | Switch to previous branch |
| `git branch` | List local branches |
| `git branch -a` | List all branches (local + remote) |
| `git branch -v` | List branches with last commit |
| `git branch --merged` | List branches merged into current |
| `git branch -m <new>` | Rename current branch |
| `git branch -d <name>` | Delete merged branch (safe) |
| `git branch -D <name>` | Force delete branch |
| `git push origin --delete <name>` | Delete remote branch |
| `git fetch --prune` | Remove stale remote tracking refs |

---

## Quick Revision Questions

1. **What is the difference between `git branch <name>` and `git switch -c <name>`?**
   <details><summary>Answer</summary>`git branch <name>` creates the branch but stays on the current branch. `git switch -c <name>` creates the branch AND switches to it immediately.</details>

2. **Why did Git introduce `git switch` and `git restore` in version 2.23?**
   <details><summary>Answer</summary>Because `git checkout` was overloaded — it handled both switching branches and restoring files. The new commands separate these concerns for clarity: `switch` for branches, `restore` for files.</details>

3. **What happens to your working directory when you switch branches?**
   <details><summary>Answer</summary>Git updates HEAD to point to the new branch, then updates the staging area and working directory files to match the new branch's latest commit. Files change to reflect the target branch.</details>

4. **How do you recover a branch that was accidentally deleted?**
   <details><summary>Answer</summary>Use `git reflog` to find the commit the branch was pointing to, then recreate it: `git branch <name> <commit-hash>`.</details>

5. **What's the difference between `git branch -d` and `git branch -D`?**
   <details><summary>Answer</summary>`-d` is safe delete — it refuses to delete unmerged branches. `-D` is force delete — it deletes regardless of merge status, with potential data loss.</details>

6. **How do you delete a branch from a remote repository?**
   <details><summary>Answer</summary>`git push origin --delete <branch-name>`. Then run `git fetch --prune` locally to clean up stale tracking references.</details>

---

[← Previous: Branch Concepts](01-branch-concepts.md) | [Back to README](../README.md) | [Next: Merging Strategies →](03-merging-strategies.md)
