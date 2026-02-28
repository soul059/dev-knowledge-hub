# Chapter 3.5: Rebasing vs Merging

[← Previous: Merge Conflicts Resolution](04-merge-conflicts-resolution.md) | [Back to README](../README.md) | [Next: Cherry-Picking →](06-cherry-picking.md)

---

## Overview

Rebasing and merging are two ways to integrate changes from one branch into another. Both achieve the same end result but create **very different histories**. Understanding when to use each is crucial for maintaining a clean and useful project history.

---

## What is Rebase?

Rebase **replays** your commits on top of another branch, creating **new commits** with new hashes but the same changes.

```
┌──────────────────────────────────────────────────────────────┐
│                    git rebase                                │
│                                                              │
│  BEFORE rebase:                                              │
│                   ●───────●        (feature)                │
│                  / F1      F2                                │
│  ●───────●───────●───────●         (main)                   │
│  C1      C2      C3      M1                                 │
│                                                              │
│  $ git switch feature                                       │
│  $ git rebase main                                          │
│                                                              │
│  AFTER rebase:                                               │
│                                                              │
│  ●───────●───────●───────●───────●───────●                  │
│  C1      C2      C3      M1     F1'     F2'                 │
│                           ▲               ▲                 │
│                         main           feature              │
│                                                              │
│  F1' and F2' are NEW commits (same changes, new hashes)     │
│  The history is now LINEAR — as if feature was started      │
│  from M1 instead of C3                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Merge vs Rebase — Visual Comparison

```
┌──────────────────────────────────────────────────────────────┐
│         MERGE vs REBASE — SAME RESULT, DIFFERENT HISTORY    │
│                                                              │
│  ═══════ MERGE ═══════                                      │
│                                                              │
│                   ●───────●                                 │
│                  / F1      F2\                               │
│  ●───────●───────●───────●───●───●                          │
│  C1      C2      C3      M1     MERGE                       │
│                                   ▲                         │
│                                 main                        │
│                                                              │
│  ✅ Non-destructive (original commits unchanged)             │
│  ✅ Full context of when/where branch was created            │
│  ❌ Cluttered history with merge commits                     │
│                                                              │
│  ═══════ REBASE ═══════                                     │
│                                                              │
│  ●───────●───────●───────●───────●───────●                  │
│  C1      C2      C3      M1     F1'     F2'                 │
│                                          ▲                  │
│                                        main                 │
│                                                              │
│  ✅ Clean, linear history                                    │
│  ✅ Easy to read and navigate                                │
│  ❌ Rewrites history (new commit hashes)                     │
│  ❌ DANGEROUS if branch is shared/pushed                     │
└──────────────────────────────────────────────────────────────┘
```

---

## How Rebase Works Internally

```
┌──────────────────────────────────────────────────────────────┐
│           REBASE STEP BY STEP                               │
│                                                              │
│  Step 1: Git finds the common ancestor (C3)                 │
│                                                              │
│  Step 2: Git saves your commits (F1, F2) as patches         │
│          (temporary area)                                    │
│                                                              │
│  Step 3: Git resets your branch to the target (main/M1)     │
│                                                              │
│  Step 4: Git replays each patch on top:                     │
│                                                              │
│          ●───────●───────●───────●                           │
│          C1      C2      C3      M1  ← start here           │
│                                  │                          │
│                                  ├── Apply F1 → F1'         │
│                                  │                          │
│                                  └── Apply F2 → F2'         │
│                                                              │
│  Step 5: Update feature branch pointer to F2'               │
│                                                              │
│  If a patch causes a conflict, Git pauses for resolution    │
└──────────────────────────────────────────────────────────────┘
```

---

## Rebase Commands

```bash
# Rebase current branch onto main
$ git switch feature/login
$ git rebase main

# Rebase onto a specific commit
$ git rebase abc1234

# If conflicts occur during rebase:
$ git rebase --continue    # After resolving, continue
$ git rebase --abort       # Cancel the rebase entirely
$ git rebase --skip        # Skip the conflicting commit

# Interactive rebase (covered in detail in Unit 6)
$ git rebase -i main
```

---

## The Golden Rule of Rebasing

```
┌──────────────────────────────────────────────────────────────┐
│         ⚠️  THE GOLDEN RULE OF REBASING  ⚠️                  │
│                                                              │
│     NEVER rebase commits that have been PUSHED               │
│     to a shared/public branch.                               │
│                                                              │
│  WHY?                                                        │
│  Rebase REWRITES HISTORY (creates new commit hashes).        │
│  If others have based work on the original commits,          │
│  their history diverges from yours → CHAOS.                  │
│                                                              │
│  SAFE to rebase:                                             │
│  ✅ Your local feature branch (not pushed yet)               │
│  ✅ Your personal branch that nobody else uses               │
│                                                              │
│  NEVER rebase:                                               │
│  ❌ main / develop (shared branches)                         │
│  ❌ Any branch others are working from                       │
│  ❌ Commits that are already pushed (unless you force push   │
│     and coordinate with the team)                            │
│                                                              │
│  Rule of thumb: "If in doubt, MERGE"                        │
└──────────────────────────────────────────────────────────────┘
```

### What Goes Wrong if You Rebase Shared History

```
┌──────────────────────────────────────────────────────────────┐
│  Developer A rebases shared commits:                        │
│                                                              │
│  Before (shared):   C1 ── C2 ── C3                          │
│  After rebase:      C1 ── C2'── C3'  (new hashes!)         │
│                                                              │
│  Developer B still has:  C1 ── C2 ── C3 ── B1              │
│  Remote now has:         C1 ── C2'── C3'                    │
│                                                              │
│  When B tries to push or pull:                              │
│  💥 Diverged histories! Duplicate commits! Confusion!       │
│                                                              │
│  B has C2 and C2' (same changes, different hashes)          │
│  → Messy merges, duplicated commits, frustration            │
└──────────────────────────────────────────────────────────────┘
```

---

## When to Use Merge vs Rebase

| Scenario | Recommended | Why |
|----------|-------------|-----|
| Integrating feature into main | **Merge (--no-ff)** | Preserves feature branch context |
| Updating feature with latest main | **Rebase** | Keeps feature branch linear, clean |
| Shared/public branch | **Merge** | Never rewrite shared history |
| Local branch, not yet pushed | **Rebase** | Clean up before sharing |
| Want to preserve full history | **Merge** | Merge commits show when integration happened |
| Want a clean, linear log | **Rebase** | No merge commits |
| Multiple contributors on same branch | **Merge** | Everyone's history stays intact |

### Common Workflow: Rebase + Merge

```bash
# The best of both worlds:

# 1. Rebase your feature onto latest main (clean up)
$ git switch feature/login
$ git rebase main

# 2. Then merge into main with --no-ff (preserve context)
$ git switch main
$ git merge --no-ff feature/login

# Result: Clean feature history + merge commit showing integration
```

---

## git pull --rebase

```bash
# Instead of git pull (which does fetch + merge):
$ git pull --rebase origin main
# This does fetch + rebase

# Set as default:
$ git config --global pull.rebase true

# Then just:
$ git pull   # Automatically rebases instead of merging
```

```
┌──────────────────────────────────────────────────────────────┐
│        git pull (merge) vs git pull --rebase                │
│                                                              │
│  git pull (merge):                                           │
│     ●───●───●───●───●                                       │
│              \       \                                       │
│               ●───●───MERGE  ← extra merge commit           │
│                                                              │
│  git pull --rebase:                                          │
│     ●───●───●───●───●───●                                   │
│                                                              │
│  Rebasing on pull keeps a cleaner local history              │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Keeping Feature Branch Up to Date

```bash
# Your feature branch is behind main
$ git switch feature/dashboard
$ git rebase main
# Resolve any conflicts
$ git rebase --continue
# Your branch is now based on the latest main
```

### Scenario 2: Clean History Before PR

```bash
# Rebase onto main and squash WIP commits
$ git rebase -i main
# Mark WIP commits as "squash"
# Save and edit the combined commit message
$ git push --force-with-lease  # Update remote (you own this branch)
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Rebase causes many conflicts | Try merging instead, or rebase commit by commit |
| "Cannot rebase: you have unstaged changes" | Commit or stash changes first |
| Accidentally rebased a shared branch | `git reflog` to find pre-rebase state, `git reset --hard <hash>` |
| Need to undo a rebase | `git reflog` → find the original HEAD → `git reset --hard <hash>` |
| Force push after rebase | Use `--force-with-lease` (safer than `--force`) |

---

## Summary Table

| Aspect | Merge | Rebase |
|--------|-------|--------|
| **History** | Non-linear (branches visible) | Linear (branches flattened) |
| **Commits** | Preserves original + adds merge commit | Creates new commits (new hashes) |
| **Safety** | Safe for shared branches | Dangerous for shared branches |
| **Conflicts** | Resolve once | May resolve same conflict multiple times |
| **Readability** | Can be cluttered | Clean and easy to follow |
| **Traceability** | Shows when branches merged | Loses branch/merge context |
| **Command** | `git merge <branch>` | `git rebase <branch>` |
| **Undo** | `git reset --hard HEAD~1` | `git reflog` + `git reset` |

---

## Quick Revision Questions

1. **What does `git rebase` do at a high level?**
   <details><summary>Answer</summary>Rebase takes the commits from your current branch, temporarily removes them, updates the branch to match the target branch, then replays your commits on top — creating new commits with new hashes but the same changes.</details>

2. **What is the "Golden Rule" of rebasing?**
   <details><summary>Answer</summary>Never rebase commits that have been pushed to a shared/public branch. Rebasing rewrites history (new hashes), which causes problems for anyone who has based work on the original commits.</details>

3. **When would you choose rebase over merge?**
   <details><summary>Answer</summary>When updating a local/personal feature branch with the latest changes from main, or when cleaning up commit history before creating a pull request. Use rebase for clean, linear history on branches only you work with.</details>

4. **What does `git pull --rebase` do differently from `git pull`?**
   <details><summary>Answer</summary>`git pull` does fetch + merge (creates merge commits). `git pull --rebase` does fetch + rebase (replays your local commits on top of remote changes), keeping a cleaner linear history.</details>

5. **How do you undo a rebase that went wrong?**
   <details><summary>Answer</summary>Use `git reflog` to find the commit hash from before the rebase, then `git reset --hard <hash>` to restore the branch to its pre-rebase state.</details>

6. **Describe the "rebase then merge" workflow and why it's popular.**
   <details><summary>Answer</summary>First, rebase your feature branch onto the latest main (clean linear commits). Then merge the feature into main with `--no-ff` (creates a merge commit). This gives you clean feature history plus a merge commit that clearly shows when the feature was integrated.</details>

---

[← Previous: Merge Conflicts Resolution](04-merge-conflicts-resolution.md) | [Back to README](../README.md) | [Next: Cherry-Picking →](06-cherry-picking.md)
