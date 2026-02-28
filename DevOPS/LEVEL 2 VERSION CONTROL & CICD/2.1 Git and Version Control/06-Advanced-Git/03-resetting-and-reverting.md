# Chapter 6.3: Resetting and Reverting

[← Previous: Stashing](02-stashing.md) | [Back to README](../README.md) | [Next: Reflog and Recovery →](04-reflog-and-recovery.md)

---

## Overview

Git provides multiple ways to undo changes: **reset** rewrites history by moving the branch pointer, while **revert** creates a new commit that undoes a previous one. Understanding the difference is critical — using the wrong one can lose work or break shared history.

---

## The Three Types of Reset

```
┌──────────────────────────────────────────────────────────────┐
│        git reset — THREE MODES                              │
│                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                │
│  │ Working  │   │ Staging  │   │  Repo    │                │
│  │   Dir    │   │  Area    │   │ (commits)│                │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘                │
│       │              │              │                       │
│  --soft               │              │                       │
│  (keeps)         (keeps)        ◄── moves HEAD              │
│                                                              │
│  --mixed (default)    │                                      │
│  (keeps)         ◄── unstages   ◄── moves HEAD              │
│                                                              │
│  --hard                                                      │
│  ◄── DISCARDS    ◄── unstages   ◄── moves HEAD              │
│                                                              │
│  ┌─────────┬───────────┬───────────┬─────────────┐          │
│  │ Mode    │Working Dir│ Staging   │ Commits     │          │
│  ├─────────┼───────────┼───────────┼─────────────┤          │
│  │ --soft  │ Unchanged │ Unchanged │ Moved back  │          │
│  │ --mixed │ Unchanged │ Cleared   │ Moved back  │          │
│  │ --hard  │ CLEARED   │ Cleared   │ Moved back  │          │
│  └─────────┴───────────┴───────────┴─────────────┘          │
└──────────────────────────────────────────────────────────────┘
```

---

## git reset

### --soft: Undo commits, keep everything staged

```bash
$ git reset --soft HEAD~1

# Commit is undone, but changes remain in staging area
# Perfect for: re-doing a commit message or combining commits
```

```
┌──────────────────────────────────────────────────────────────┐
│  BEFORE: ●──●──●──●                                         │
│                    ▲ HEAD                                    │
│                                                              │
│  $ git reset --soft HEAD~1                                  │
│                                                              │
│  AFTER:  ●──●──●                                            │
│                 ▲ HEAD                                       │
│                                                              │
│  Changes from the undone commit are STAGED                  │
│  (ready to commit again with a new message)                 │
└──────────────────────────────────────────────────────────────┘
```

### --mixed (default): Undo commits, unstage changes

```bash
$ git reset HEAD~1
# Same as: git reset --mixed HEAD~1

# Commit is undone, changes are in working directory (unstaged)
# Perfect for: re-organizing which files go in which commit
```

### --hard: Undo commits, DISCARD everything

```bash
$ git reset --hard HEAD~1

# ⚠️ DESTRUCTIVE — commits AND changes are GONE
# Working directory matches the target commit exactly
# Only use when you're sure you want to discard everything
```

---

## Reset vs Revert

```
┌──────────────────────────────────────────────────────────────┐
│        RESET vs REVERT                                      │
│                                                              │
│  RESET (rewrites history):                                  │
│                                                              │
│  Before: ●──●──●──●                                         │
│                    ▲ HEAD                                    │
│                                                              │
│  After:  ●──●──●                                            │
│                 ▲ HEAD (commit removed from history)         │
│                                                              │
│  ✅ Clean history                                           │
│  ❌ UNSAFE for shared branches (rewrites history)           │
│                                                              │
│  ─────────────────────────────────────────────              │
│                                                              │
│  REVERT (preserves history):                                │
│                                                              │
│  Before: ●──●──●──●                                         │
│                    ▲ HEAD                                    │
│                                                              │
│  After:  ●──●──●──●──● (revert commit)                     │
│                       ▲ HEAD                                │
│                                                              │
│  ✅ SAFE for shared branches (adds new commit)              │
│  ✅ Full audit trail                                        │
│  ❌ Extra commit in history                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## git revert

```bash
# Revert a specific commit (creates a new "undo" commit)
$ git revert <commit-hash>

# Revert without auto-committing (stage changes only)
$ git revert --no-commit <hash>
$ git revert -n <hash>

# Revert multiple commits
$ git revert <hash1> <hash2> <hash3>

# Revert a range
$ git revert HEAD~3..HEAD

# Revert a merge commit (must specify parent)
$ git revert -m 1 <merge-commit-hash>
```

```
┌──────────────────────────────────────────────────────────────┐
│  REVERTING A MERGE COMMIT                                   │
│                                                              │
│  A merge commit has TWO parents:                            │
│                                                              │
│  main:    ●──●──●──MERGE──●                                 │
│                   /                                         │
│  feature: ●──●──●/                                          │
│           │     │                                           │
│         parent 1: main side                                 │
│         parent 2: feature side                              │
│                                                              │
│  $ git revert -m 1 <merge-hash>                             │
│  -m 1: Keep parent 1 (main), undo parent 2 (feature)       │
│  -m 2: Keep parent 2 (feature), undo parent 1 (main)       │
│                                                              │
│  Usually: -m 1 (you want to undo the merged feature)       │
└──────────────────────────────────────────────────────────────┘
```

---

## When to Use Which

| Scenario | Use | Command |
|----------|-----|---------|
| Undo last commit, redo with new message | `reset --soft` | `git reset --soft HEAD~1` |
| Undo last commit, restage differently | `reset --mixed` | `git reset HEAD~1` |
| Discard last commit entirely | `reset --hard` | `git reset --hard HEAD~1` |
| Undo a commit on a shared branch | `revert` | `git revert <hash>` |
| Undo a merge on main | `revert -m` | `git revert -m 1 <hash>` |
| Remove uncommitted staged changes | `reset file` | `git reset HEAD <file>` |
| Discard all uncommitted changes | `reset --hard` | `git reset --hard HEAD` |

---

## Unstaging and Discarding (File-level)

```bash
# Unstage a file (keep changes in working dir)
$ git reset HEAD <file>
# Modern alternative:
$ git restore --staged <file>

# Discard changes in working directory
$ git checkout -- <file>
# Modern alternative:
$ git restore <file>

# Discard all uncommitted changes
$ git reset --hard HEAD
# Or:
$ git restore .
```

---

## Real-World Scenarios

### Scenario 1: Oops — Wrong Commit Message

```bash
# Reset soft + recommit
$ git reset --soft HEAD~1
$ git commit -m "Correct commit message"
```

### Scenario 2: Committed to Wrong Branch

```bash
# On main (should be on feature branch)
$ git reset --soft HEAD~1            # Undo commit, keep changes
$ git stash                          # Stash the changes
$ git switch feature/correct-branch  # Switch to correct branch
$ git stash pop                      # Apply changes
$ git commit -m "Change description" # Commit on correct branch
```

### Scenario 3: Undo a Pushed Commit

```bash
# Can't reset shared history — use revert
$ git revert abc1234
$ git push origin main
# History preserved, bad commit is undone
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Lost commits after `--hard` reset | `git reflog` to find the commit, `git reset --hard <hash>` to restore |
| Revert causes conflicts | Resolve manually, `git add`, then `git revert --continue` |
| "Unmerged paths" after reset | Resolve conflicts first, then reset |
| Want to undo a revert | Revert the revert: `git revert <revert-commit-hash>` |
| Reset moved HEAD but not branch | You may be in detached HEAD — `git switch <branch>` |

---

## Summary Table

| Command | History | Working Dir | Staging | Safe for Shared? |
|---------|---------|-------------|---------|-------------------|
| `reset --soft HEAD~1` | Removes commit | Unchanged | Unchanged | ❌ No |
| `reset --mixed HEAD~1` | Removes commit | Unchanged | Cleared | ❌ No |
| `reset --hard HEAD~1` | Removes commit | Cleared | Cleared | ❌ No |
| `revert <hash>` | New commit | Updated | Updated | ✅ Yes |
| `restore <file>` | N/A | Discards changes | N/A | ✅ Yes |
| `restore --staged <file>` | N/A | Unchanged | Unstaged | ✅ Yes |

---

## Quick Revision Questions

1. **What are the three modes of `git reset` and how do they differ?**
   <details><summary>Answer</summary>`--soft`: moves HEAD, keeps staging and working dir. `--mixed` (default): moves HEAD, clears staging, keeps working dir. `--hard`: moves HEAD, clears staging AND working dir. Only `--hard` is destructive to your working files.</details>

2. **When should you use `git revert` instead of `git reset`?**
   <details><summary>Answer</summary>Use `git revert` when the commits have been pushed to a shared branch. Revert creates a new commit that undoes the changes, preserving history. Reset rewrites history, which breaks other developers' work on shared branches.</details>

3. **How do you undo a merge commit?**
   <details><summary>Answer</summary>`git revert -m 1 <merge-commit-hash>`. The `-m 1` tells Git to keep parent 1 (usually the main branch) and undo the changes from parent 2 (the merged branch).</details>

4. **How do you recover from an accidental `git reset --hard`?**
   <details><summary>Answer</summary>Use `git reflog` to find the commit hash from before the reset. Then `git reset --hard <hash>` to restore to that point. Reflog keeps a record of all HEAD movements for about 90 days.</details>

5. **What is the modern alternative to `git checkout -- <file>`?**
   <details><summary>Answer</summary>`git restore <file>` discards working directory changes. `git restore --staged <file>` unstages a file. These replace the overloaded `git checkout` command for file-level operations.</details>

---

[← Previous: Stashing](02-stashing.md) | [Back to README](../README.md) | [Next: Reflog and Recovery →](04-reflog-and-recovery.md)
