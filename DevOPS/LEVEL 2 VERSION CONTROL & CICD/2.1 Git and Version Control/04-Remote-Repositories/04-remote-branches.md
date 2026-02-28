# Chapter 4.4: Remote Branches

[← Previous: Fetch, Pull, and Push](03-fetch-pull-push.md) | [Back to README](../README.md) | [Next: Forking Workflow →](05-forking-workflow.md)

---

## Overview

Remote branches are references to the state of branches on your remote repositories. Understanding how to create, track, publish, and delete remote branches is key to effective Git collaboration.

---

## Types of Branch References

```
┌──────────────────────────────────────────────────────────────┐
│       THREE TYPES OF BRANCH REFERENCES                      │
│                                                              │
│  1. LOCAL BRANCHES                                          │
│     Stored in: refs/heads/                                  │
│     Example:   main, feature/login                          │
│     $ git branch                                            │
│                                                              │
│  2. REMOTE-TRACKING BRANCHES                                │
│     Stored in: refs/remotes/<remote>/                       │
│     Example:   origin/main, origin/develop                  │
│     $ git branch -r                                         │
│     (Read-only — updated by git fetch)                      │
│                                                              │
│  3. ALL BRANCHES                                            │
│     $ git branch -a                                         │
│       * main                          ← local              │
│         feature/login                 ← local              │
│         remotes/origin/main           ← remote-tracking    │
│         remotes/origin/develop        ← remote-tracking    │
│         remotes/origin/feature/api    ← remote-tracking    │
└──────────────────────────────────────────────────────────────┘
```

---

## Publishing a Local Branch to Remote

```bash
# Push a local branch to remote (first time)
$ git push -u origin feature/login
# -u sets upstream tracking

# After -u is set, just use:
$ git push
```

```
┌──────────────────────────────────────────────────────────────┐
│        PUBLISHING A BRANCH                                  │
│                                                              │
│  BEFORE push:                                                │
│                                                              │
│  Local:    main ●     feature/login ●                       │
│  Remote:   main ●     (no feature/login)                    │
│                                                              │
│  $ git push -u origin feature/login                         │
│                                                              │
│  AFTER push:                                                 │
│                                                              │
│  Local:    main ●     feature/login ●                       │
│  Remote:   main ●     feature/login ●   ← Created!         │
│                                                              │
│  Local now also has:                                        │
│  origin/feature/login  (remote-tracking branch)             │
│  Upstream tracking:                                         │
│  feature/login → origin/feature/login                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Tracking a Remote Branch Locally

When a branch exists on the remote but not locally:

```bash
# List remote branches
$ git branch -r
  origin/main
  origin/develop
  origin/feature/api

# Create local branch tracking the remote branch
$ git switch --track origin/feature/api
# Shorthand (auto-detects remote):
$ git switch feature/api

# Or with checkout:
$ git checkout -b feature/api origin/feature/api
$ git checkout --track origin/feature/api
```

```
┌──────────────────────────────────────────────────────────────┐
│        TRACKING A REMOTE BRANCH                             │
│                                                              │
│  Remote has:   origin/feature/api                           │
│  Local:        (nothing)                                    │
│                                                              │
│  $ git switch feature/api                                   │
│  Branch 'feature/api' set up to track 'origin/feature/api'  │
│                                                              │
│  Now:                                                        │
│  Local:    feature/api → tracks → origin/feature/api        │
│  You can now git push and git pull on this branch           │
└──────────────────────────────────────────────────────────────┘
```

---

## Checking Branch Tracking Status

```bash
# Show tracking info for all branches
$ git branch -vv
  * main          abc1234 [origin/main] Latest commit message
    develop       def5678 [origin/develop: ahead 2] Dev work
    feature/login ghi9012 [origin/feature/login: behind 3] Login page
    local-only    jkl3456 No tracking info here

# Understanding the output:
#   [origin/main]           → up to date with remote
#   [origin/develop: ahead 2] → you have 2 commits not on remote
#   [origin/feature: behind 3] → remote has 3 commits you don't have
#   [origin/feature: ahead 1, behind 2] → diverged
#   (no brackets)           → no upstream tracking set
```

```
┌──────────────────────────────────────────────────────────────┐
│       BRANCH TRACKING STATES                                │
│                                                              │
│  UP TO DATE:           ●──●──●                              │
│                              ▲ both local and origin/       │
│                                                              │
│                                                              │
│  AHEAD by 2:           ●──●──●──●──●                        │
│                              ▲        ▲                     │
│                           origin/    local (2 ahead)        │
│                                                              │
│                                                              │
│  BEHIND by 3:          ●──●──●──●──●──●                     │
│                              ▲           ▲                  │
│                            local      origin/ (3 ahead)     │
│                                                              │
│                                                              │
│  DIVERGED:             ●──●──●──L1──L2                      │
│                              │                              │
│                              └──R1──R2──R3                  │
│                          ahead 2, behind 3                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Deleting Remote Branches

```bash
# Delete a remote branch
$ git push origin --delete feature/login
# Or shorthand:
$ git push origin :feature/login

# Delete the local branch too
$ git branch -d feature/login      # Safe delete (must be merged)
$ git branch -D feature/login      # Force delete

# Clean up stale remote-tracking branches
$ git fetch --prune
```

```
┌──────────────────────────────────────────────────────────────┐
│       DELETING REMOTE BRANCHES — COMPLETE CLEANUP           │
│                                                              │
│  Step 1: Delete on remote                                   │
│  $ git push origin --delete feature/login                   │
│  → Remote no longer has feature/login                       │
│                                                              │
│  Step 2: Delete local branch                                │
│  $ git branch -d feature/login                              │
│  → Local branch removed                                    │
│                                                              │
│  Step 3: Prune remote-tracking ref                          │
│  $ git fetch --prune                                        │
│  → origin/feature/login removed                            │
│                                                              │
│  Or do step 1 + step 3 together:                            │
│  $ git push origin --delete feature/login                   │
│  $ git remote prune origin                                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Listing and Inspecting Remote Branches

```bash
# List remote branches only
$ git branch -r

# List all (local + remote)
$ git branch -a

# Show which remote branches are merged into current branch
$ git branch -r --merged

# Show unmerged remote branches
$ git branch -r --no-merged

# See latest commit on each remote branch
$ git branch -r -v

# Inspect a specific remote branch
$ git log origin/feature/api --oneline -5
```

---

## Real-World Scenarios

### Scenario 1: Setting Up a New Feature

```bash
# Create and push new feature branch
$ git switch -c feature/dashboard
# ... make commits ...
$ git push -u origin feature/dashboard
# Team members can now see and track your branch
```

### Scenario 2: Cleaning Up After Merge

```bash
# Feature merged via PR on GitHub → branch deleted on remote
# Update your local repo:
$ git fetch --prune
$ git branch -d feature/dashboard    # Delete local copy
```

### Scenario 3: Checking Out a Teammate's Branch

```bash
$ git fetch origin
$ git switch feature/api   # Automatically tracks origin/feature/api
# Now you can work on it too
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "fatal: Cannot update paths and switch to branch" | Branch exists locally and remotely with different history — delete local first |
| Remote branch not showing up | `git fetch origin` to update remote-tracking branches |
| Can't delete remote branch | Check if it's the default branch on GitHub (protected) |
| Stale `origin/feature` after remote deletion | `git fetch --prune` |
| "branch already exists" when tracking | Use `git switch feature` (auto-tracks) or delete local first |

---

## Summary Table

| Task | Command |
|------|---------|
| List local branches | `git branch` |
| List remote branches | `git branch -r` |
| List all branches | `git branch -a` |
| Publish branch to remote | `git push -u origin <branch>` |
| Track a remote branch | `git switch <branch>` (auto-track) |
| Check tracking status | `git branch -vv` |
| Delete remote branch | `git push origin --delete <branch>` |
| Delete local branch | `git branch -d <branch>` |
| Clean stale refs | `git fetch --prune` |
| Show merged remote branches | `git branch -r --merged` |

---

## Quick Revision Questions

1. **What is a remote-tracking branch?**
   <details><summary>Answer</summary>A remote-tracking branch (like `origin/main`) is a read-only reference that mirrors the state of a branch on the remote. It's updated when you run `git fetch` or `git pull`, and stored under `refs/remotes/`.</details>

2. **How do you create a local branch that tracks a remote branch?**
   <details><summary>Answer</summary>Use `git switch <branch-name>` — Git automatically sets up tracking if a matching remote branch exists. Or explicitly: `git switch --track origin/<branch-name>`.</details>

3. **How do you delete a branch on the remote?**
   <details><summary>Answer</summary>`git push origin --delete <branch-name>`. This removes the branch from the remote server. You should also delete the local branch with `git branch -d <branch>` and prune with `git fetch --prune`.</details>

4. **What does `git branch -vv` show?**
   <details><summary>Answer</summary>It shows all local branches with their latest commit, their tracked upstream branch, and whether they are ahead, behind, or diverged from the upstream.</details>

5. **How do you publish a new local branch to the remote?**
   <details><summary>Answer</summary>`git push -u origin <branch-name>`. The `-u` flag sets up upstream tracking so future `git push` and `git pull` commands work without arguments.</details>

---

[← Previous: Fetch, Pull, and Push](03-fetch-pull-push.md) | [Back to README](../README.md) | [Next: Forking Workflow →](05-forking-workflow.md)
