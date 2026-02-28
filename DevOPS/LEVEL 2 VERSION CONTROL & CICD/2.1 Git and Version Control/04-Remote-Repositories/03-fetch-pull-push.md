# Chapter 4.3: Fetch, Pull, and Push

[← Previous: Adding and Managing Remotes](02-adding-and-managing-remotes.md) | [Back to README](../README.md) | [Next: Remote Branches →](04-remote-branches.md)

---

## Overview

**Fetch**, **pull**, and **push** are the three fundamental commands for synchronizing your local repository with a remote. Understanding the differences between them — especially between fetch and pull — is essential for safe collaboration.

---

## The Three Operations

```
┌──────────────────────────────────────────────────────────────┐
│        FETCH vs PULL vs PUSH                                │
│                                                              │
│  ┌──────────┐              ┌──────────────┐                 │
│  │  LOCAL   │              │   REMOTE     │                 │
│  │  REPO    │              │   REPO       │                 │
│  │          │              │              │                 │
│  │  main    │              │  main        │                 │
│  │  origin/ │              │              │                 │
│  │   main   │              │              │                 │
│  └────┬─────┘              └──────┬───────┘                 │
│       │                           │                         │
│       │◄──── git fetch ──────────│                         │
│       │     (downloads objects    │                         │
│       │      updates origin/main  │                         │
│       │      does NOT touch       │                         │
│       │      your local main)     │                         │
│       │                           │                         │
│       │◄──── git pull ───────────│                         │
│       │     (fetch + merge        │                         │
│       │      updates origin/main  │                         │
│       │      AND merges into      │                         │
│       │      your local main)     │                         │
│       │                           │                         │
│       │────── git push ─────────►│                         │
│       │     (uploads your local   │                         │
│       │      commits to remote)   │                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## git fetch

Fetch downloads commits, files, and refs from a remote — but **does not modify your working directory or local branches**.

```bash
# Fetch from default remote (origin)
$ git fetch

# Fetch from a specific remote
$ git fetch origin

# Fetch a specific branch
$ git fetch origin main

# Fetch all remotes
$ git fetch --all

# Fetch and prune stale branches
$ git fetch --prune
$ git fetch -p
```

```
┌──────────────────────────────────────────────────────────────┐
│          WHAT git fetch DOES                                │
│                                                              │
│  BEFORE fetch:                                               │
│                                                              │
│  Local:     ●──●──●              (main, origin/main)        │
│                    C3                                        │
│                                                              │
│  Remote:    ●──●──●──●──●        (main)                     │
│                    C3 C4 C5      ← 2 new commits            │
│                                                              │
│  AFTER git fetch:                                            │
│                                                              │
│  Local:     ●──●──●              (main) ← UNCHANGED         │
│                    │                                        │
│                    └──●──●       (origin/main) ← UPDATED    │
│                       C4 C5                                 │
│                                                              │
│  Your working files are NOT changed.                        │
│  You can inspect changes before integrating.                │
│                                                              │
│  # See what was fetched:                                    │
│  $ git log main..origin/main     # Commits you don't have  │
│  $ git diff main origin/main     # Differences             │
└──────────────────────────────────────────────────────────────┘
```

### Why Fetch is Safe

```
┌──────────────────────────────────────────────────────────────┐
│  fetch is ALWAYS safe because it:                           │
│                                                              │
│  ✅ Downloads data without modifying your branches          │
│  ✅ Lets you review changes before integrating              │
│  ✅ Never causes merge conflicts                            │
│  ✅ Can be run at any time                                  │
│                                                              │
│  After fetching, YOU decide how to integrate:               │
│                                                              │
│  $ git fetch origin                                         │
│  $ git log main..origin/main --oneline   # Review          │
│  $ git merge origin/main                 # Merge manually   │
│  # OR                                                       │
│  $ git rebase origin/main                # Rebase instead   │
└──────────────────────────────────────────────────────────────┘
```

---

## git pull

Pull is essentially **fetch + merge** (or fetch + rebase) in one command.

```bash
# Pull from tracked upstream
$ git pull

# Pull from specific remote/branch
$ git pull origin main

# Pull with rebase instead of merge
$ git pull --rebase
$ git pull --rebase origin main

# Pull with fast-forward only (fail if not possible)
$ git pull --ff-only
```

```
┌──────────────────────────────────────────────────────────────┐
│         git pull = git fetch + git merge                    │
│                                                              │
│  BEFORE pull:                                                │
│                                                              │
│  Local main:  ●──●──●                                       │
│                     C3                                       │
│                                                              │
│  Remote main: ●──●──●──●──●                                 │
│                     C3 C4 C5                                 │
│                                                              │
│                                                              │
│  AFTER git pull (merge):                                     │
│                                                              │
│  Local main:  ●──●──●──────────●     (merge commit)         │
│                     │          │                            │
│                     └──●──●───/                              │
│                        C4 C5                                │
│                                                              │
│  AFTER git pull --rebase:                                    │
│                                                              │
│  Local main:  ●──●──●──●──●                                 │
│                     C3 C4 C5  ← clean, linear               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### git pull with Local Changes

```
┌──────────────────────────────────────────────────────────────┐
│  WHEN YOU HAVE LOCAL COMMITS + REMOTE COMMITS               │
│                                                              │
│  Local:    ●──●──●──●  (L1 = your commit)                   │
│                  C3 L1                                       │
│                                                              │
│  Remote:   ●──●──●──●──●                                    │
│                  C3 R1 R2  (remote commits)                  │
│                                                              │
│  git pull (merge):                                           │
│            ●──●──●──●──────────●                            │
│                  │  L1          \                            │
│                  └──●──●────────MERGE                        │
│                     R1 R2                                   │
│                                                              │
│  git pull --rebase:                                          │
│            ●──●──●──●──●──●                                 │
│                  C3 R1 R2 L1' (your commit replayed)         │
│                                                              │
│  Rebase is usually cleaner for this scenario                │
└──────────────────────────────────────────────────────────────┘
```

---

## git push

Push uploads your local commits to the remote repository.

```bash
# Push to tracked upstream
$ git push

# Push specific branch to remote
$ git push origin main

# Push and set upstream tracking
$ git push -u origin feature/login
$ git push --set-upstream origin feature/login

# Push all branches
$ git push --all

# Push tags
$ git push --tags
$ git push origin v1.0.0

# Force push (DANGEROUS — rewrites remote history)
$ git push --force
$ git push -f

# Safer force push (fails if remote has new commits)
$ git push --force-with-lease
```

```
┌──────────────────────────────────────────────────────────────┐
│           git push FLOW                                      │
│                                                              │
│  $ git push origin main                                     │
│                                                              │
│  1. Git checks: is remote main a direct ancestor?           │
│                                                              │
│     Local:   ●──●──●──●──●                                  │
│                        ▲  ▲                                 │
│                   remote  local (ahead by 1)                │
│                                                              │
│  2a. YES (fast-forward possible) → Push succeeds            │
│      Remote updated to match local                          │
│                                                              │
│  2b. NO (diverged histories) → Push REJECTED                │
│                                                              │
│      Local:   ●──●──●──L1                                   │
│                     │                                       │
│      Remote:  ●──●──●──R1  ← Diverged!                     │
│                                                              │
│      You must: git pull first, resolve, then push           │
│      Or:       git push --force-with-lease (CAREFUL!)       │
└──────────────────────────────────────────────────────────────┘
```

### Force Push Safety

```
┌──────────────────────────────────────────────────────────────┐
│         --force vs --force-with-lease                       │
│                                                              │
│  --force:                                                    │
│  ❌ Overwrites remote blindly                                │
│  ❌ Destroys other people's work if they pushed              │
│  ❌ No safety check                                          │
│                                                              │
│  --force-with-lease:                                         │
│  ✅ Checks that remote hasn't changed since your last fetch  │
│  ✅ Fails if someone else pushed (protects their work)       │
│  ✅ ALWAYS prefer this over --force                          │
│                                                              │
│  $ git push --force-with-lease origin feature/login         │
│                                                              │
│  Use case: After rebasing your own feature branch            │
│  NEVER force push to shared branches (main, develop)        │
└──────────────────────────────────────────────────────────────┘
```

---

## Comparison: fetch vs pull vs push

| Aspect | fetch | pull | push |
|--------|-------|------|------|
| **Direction** | Remote → Local | Remote → Local | Local → Remote |
| **Modifies working dir?** | No | Yes | No (local) |
| **Modifies local branches?** | No | Yes (merges) | No (local) |
| **Updates remote-tracking?** | Yes | Yes | Yes |
| **Can cause conflicts?** | No | Yes | Rejection only |
| **Safety** | Always safe | May conflict | May be rejected |
| **When to use** | Review first | Quick sync | Share your work |

---

## Real-World Scenarios

### Scenario 1: Safe Daily Workflow

```bash
# Start of day — check what's new
$ git fetch origin
$ git log --oneline main..origin/main   # Review new commits

# Integrate if everything looks good
$ git merge origin/main
# or
$ git pull --rebase
```

### Scenario 2: Push Rejected

```bash
$ git push origin main
# ! [rejected] main -> main (non-fast-forward)

# Solution: Pull first, then push
$ git pull --rebase origin main
# Resolve any conflicts if needed
$ git push origin main
```

### Scenario 3: Push After Rebase

```bash
# Rebased your feature branch — need to update remote
$ git push --force-with-lease origin feature/login
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Push rejected (non-fast-forward) | `git pull` first, resolve conflicts, then push |
| Pull causes unwanted merge commits | Use `git pull --rebase` instead |
| "There is no tracking information" | `git push -u origin <branch>` to set upstream |
| Fetch doesn't show new remote branches | `git fetch --all` or `git fetch origin` |
| Want to undo a pull | `git reflog` → find pre-pull commit → `git reset --hard <hash>` |
| Accidentally force pushed | Hope someone has the old commits; check `git reflog` on server |

---

## Summary Table

| Command | What it Does |
|---------|-------------|
| `git fetch` | Download remote data, update remote-tracking branches only |
| `git fetch --all` | Fetch from all remotes |
| `git fetch -p` | Fetch and prune stale remote-tracking branches |
| `git pull` | Fetch + merge into current branch |
| `git pull --rebase` | Fetch + rebase current branch |
| `git pull --ff-only` | Fetch + merge only if fast-forward possible |
| `git push` | Upload local commits to remote |
| `git push -u origin <branch>` | Push and set upstream tracking |
| `git push --force-with-lease` | Force push with safety check |
| `git push --tags` | Push all tags to remote |

---

## Quick Revision Questions

1. **What is the difference between `git fetch` and `git pull`?**
   <details><summary>Answer</summary>`git fetch` downloads remote changes and updates remote-tracking branches but does not modify your working directory or local branches. `git pull` does a fetch AND automatically merges (or rebases) the changes into your current branch.</details>

2. **Why is `git fetch` considered safer than `git pull`?**
   <details><summary>Answer</summary>Because fetch only downloads data and updates remote-tracking branches — it never modifies your working directory, local branches, or causes merge conflicts. You can review changes before deciding how to integrate them.</details>

3. **What does `git push -u origin feature/login` do?**
   <details><summary>Answer</summary>It pushes the `feature/login` branch to the remote named `origin` AND sets up upstream tracking. After this, you can use `git push` and `git pull` without specifying the remote and branch name.</details>

4. **Why should you use `--force-with-lease` instead of `--force`?**
   <details><summary>Answer</summary>`--force-with-lease` checks that the remote hasn't changed since your last fetch before overwriting. If someone else pushed new commits, it fails and protects their work. `--force` blindly overwrites, potentially destroying other people's commits.</details>

5. **What happens if `git push` is rejected?**
   <details><summary>Answer</summary>Push is rejected when the remote has commits that your local branch doesn't (histories diverged). Solution: `git pull` (or `git pull --rebase`) to integrate remote changes first, resolve any conflicts, then push again.</details>

6. **How do you review remote changes before integrating them?**
   <details><summary>Answer</summary>Use `git fetch` first, then inspect with `git log main..origin/main` (commits), `git diff main origin/main` (changes), or `git log --oneline --graph --all` (visual). Once satisfied, merge or rebase to integrate.</details>

---

[← Previous: Adding and Managing Remotes](02-adding-and-managing-remotes.md) | [Back to README](../README.md) | [Next: Remote Branches →](04-remote-branches.md)
