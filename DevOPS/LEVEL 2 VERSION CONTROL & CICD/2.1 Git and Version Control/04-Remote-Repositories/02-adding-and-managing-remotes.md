# Chapter 4.2: Adding and Managing Remotes

[← Previous: Remote Concepts](01-remote-concepts.md) | [Back to README](../README.md) | [Next: Fetch, Pull, and Push →](03-fetch-pull-push.md)

---

## Overview

Git allows you to connect your local repository to multiple remote repositories. This chapter covers how to add, remove, rename, and manage remote connections, along with inspecting remote details.

---

## Adding a Remote

```bash
# Syntax
$ git remote add <name> <url>

# Examples
$ git remote add origin https://github.com/user/repo.git
$ git remote add upstream https://github.com/original/repo.git
$ git remote add gitlab git@gitlab.com:user/repo.git
```

```
┌──────────────────────────────────────────────────────────────┐
│          ADDING A REMOTE                                     │
│                                                              │
│  Local Repo                        Remote Servers            │
│  ┌──────────┐                                               │
│  │          │                                               │
│  │  .git/   │── origin ──────► github.com/user/repo         │
│  │  config  │                                               │
│  │          │── upstream ────► github.com/org/repo           │
│  │          │                                               │
│  │          │── gitlab ──────► gitlab.com/user/repo          │
│  │          │                                               │
│  └──────────┘                                               │
│                                                              │
│  Each remote is a NAME pointing to a URL.                   │
│  Stored in .git/config                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Viewing Remotes

```bash
# List remote names
$ git remote
origin
upstream

# List with URLs (verbose)
$ git remote -v
origin    https://github.com/user/repo.git (fetch)
origin    https://github.com/user/repo.git (push)
upstream  https://github.com/org/repo.git (fetch)
upstream  https://github.com/org/repo.git (push)

# Show detailed info about a remote
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/user/repo.git
  Push  URL: https://github.com/user/repo.git
  HEAD branch: main
  Remote branches:
    main    tracked
    develop tracked
  Local branches configured for 'git pull':
    main merges with remote main
  Local refs configured for 'git push':
    main pushes to main (up to date)
```

---

## Managing Remotes

### Renaming a Remote

```bash
# Rename remote
$ git remote rename <old-name> <new-name>

# Example:
$ git remote rename origin github
# Now use 'github' instead of 'origin':
$ git push github main
```

### Removing a Remote

```bash
# Remove a remote
$ git remote remove <name>
# or
$ git remote rm <name>

# Example:
$ git remote remove upstream
# All remote-tracking branches for this remote are deleted too
```

### Changing a Remote URL

```bash
# Change URL (e.g., HTTPS → SSH)
$ git remote set-url origin git@github.com:user/repo.git

# Verify
$ git remote -v
origin  git@github.com:user/repo.git (fetch)
origin  git@github.com:user/repo.git (push)

# Set different URLs for fetch and push
$ git remote set-url --push origin https://other-server.com/repo.git
```

---

## Remote Configuration in .git/config

```
┌──────────────────────────────────────────────────────────────┐
│  .git/config (after adding remotes)                         │
│                                                              │
│  [remote "origin"]                                          │
│      url = https://github.com/user/repo.git                 │
│      fetch = +refs/heads/*:refs/remotes/origin/*            │
│                                                              │
│  [remote "upstream"]                                        │
│      url = https://github.com/org/repo.git                  │
│      fetch = +refs/heads/*:refs/remotes/upstream/*          │
│                                                              │
│  [branch "main"]                                            │
│      remote = origin                                        │
│      merge = refs/heads/main                                │
│                           │                                 │
│                           └── This sets the "upstream"      │
│                               tracking for the local        │
│                               main branch                   │
│                                                              │
│  The refspec: +refs/heads/*:refs/remotes/origin/*           │
│  Maps remote branches → local remote-tracking branches      │
│  + means force update even if not fast-forward              │
└──────────────────────────────────────────────────────────────┘
```

---

## Prune Stale Remote-Tracking Branches

When branches are deleted on the remote, your local remote-tracking branches still exist. Use `prune` to clean them up.

```bash
# Show stale branches (dry run)
$ git remote prune origin --dry-run

# Remove stale remote-tracking branches
$ git remote prune origin

# Or fetch with prune (combines fetch + prune)
$ git fetch --prune
$ git fetch -p

# Auto-prune on every fetch
$ git config --global fetch.prune true
```

```
┌──────────────────────────────────────────────────────────────┐
│           PRUNING STALE BRANCHES                            │
│                                                              │
│  Remote (GitHub):          Local remote-tracking:           │
│  ┌──────────────┐          ┌────────────────────┐           │
│  │ main         │          │ origin/main        │           │
│  │ develop      │          │ origin/develop     │           │
│  │ (deleted     │          │ origin/old-feature │ ← STALE   │
│  │  old-feature)│          │                    │           │
│  └──────────────┘          └────────────────────┘           │
│                                                              │
│  $ git fetch --prune                                        │
│  → Removes origin/old-feature (it no longer exists)         │
└──────────────────────────────────────────────────────────────┘
```

---

## Common Remote Workflows

### Fork and Upstream Pattern

```bash
# 1. Fork a repo on GitHub (creates your copy)
# 2. Clone YOUR fork
$ git clone https://github.com/YOU/repo.git

# 3. Add the original repo as "upstream"
$ git remote add upstream https://github.com/ORIGINAL/repo.git

# 4. Keep your fork updated
$ git fetch upstream
$ git switch main
$ git merge upstream/main
$ git push origin main
```

```
┌──────────────────────────────────────────────────────────────┐
│        FORK + UPSTREAM WORKFLOW                              │
│                                                              │
│  ┌───────────────────┐                                      │
│  │  Original Repo    │ (upstream)                           │
│  │  org/project      │                                      │
│  └─────────┬─────────┘                                      │
│            │ Fork (GitHub UI)                                │
│            ▼                                                │
│  ┌───────────────────┐                                      │
│  │  Your Fork        │ (origin)                             │
│  │  you/project      │                                      │
│  └─────────┬─────────┘                                      │
│            │ git clone                                       │
│            ▼                                                │
│  ┌───────────────────┐                                      │
│  │  Local Repo       │                                      │
│  │  (your machine)   │                                      │
│  │                   │                                      │
│  │  origin → you/project (push here)                        │
│  │  upstream → org/project (pull from here)                 │
│  └───────────────────┘                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Switching from HTTPS to SSH

```bash
# Check current URL
$ git remote -v
origin  https://github.com/user/repo.git (fetch)

# Switch to SSH
$ git remote set-url origin git@github.com:user/repo.git

# Verify
$ git remote -v
origin  git@github.com:user/repo.git (fetch)
origin  git@github.com:user/repo.git (push)
```

### Scenario 2: Repo Moved to New Location

```bash
# Old URL no longer works
$ git remote set-url origin https://github.com/new-org/new-repo.git
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "fatal: remote origin already exists" | Use `git remote set-url origin <url>` instead of `add` |
| "fatal: No such remote 'x'" | Check spelling with `git remote -v` |
| Stale remote-tracking branches | `git fetch --prune` to clean up |
| Push/fetch to wrong URL | `git remote set-url <name> <correct-url>` |
| Need different push/fetch URLs | `git remote set-url --push <name> <url>` |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git remote add <name> <url>` | Add a new remote |
| `git remote -v` | List all remotes with URLs |
| `git remote show <name>` | Detailed info about a remote |
| `git remote rename <old> <new>` | Rename a remote |
| `git remote remove <name>` | Delete a remote |
| `git remote set-url <name> <url>` | Change remote URL |
| `git remote prune <name>` | Delete stale remote-tracking branches |
| `git fetch --prune` | Fetch and prune in one step |

---

## Quick Revision Questions

1. **How do you add a new remote to your repository?**
   <details><summary>Answer</summary>`git remote add <name> <url>` — for example: `git remote add upstream https://github.com/org/repo.git`</details>

2. **How do you change a remote's URL (e.g., from HTTPS to SSH)?**
   <details><summary>Answer</summary>`git remote set-url origin git@github.com:user/repo.git` — this updates the URL without removing and re-adding the remote.</details>

3. **What are stale remote-tracking branches and how do you remove them?**
   <details><summary>Answer</summary>Stale remote-tracking branches are references to branches that no longer exist on the remote. Remove them with `git remote prune origin` or `git fetch --prune`.</details>

4. **In a fork workflow, what are "origin" and "upstream"?**
   <details><summary>Answer</summary>"origin" points to your personal fork on GitHub. "upstream" points to the original repository you forked from. You push to origin and pull updates from upstream.</details>

5. **Where is remote configuration stored?**
   <details><summary>Answer</summary>In the `.git/config` file within your repository. Each remote has a `[remote "name"]` section with its URL and fetch refspec.</details>

---

[← Previous: Remote Concepts](01-remote-concepts.md) | [Back to README](../README.md) | [Next: Fetch, Pull, and Push →](03-fetch-pull-push.md)
