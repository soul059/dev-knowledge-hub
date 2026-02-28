# Chapter 6.4: Reflog and Recovery

[← Previous: Resetting and Reverting](03-resetting-and-reverting.md) | [Back to README](../README.md) | [Next: Tags and Releases →](05-tags-and-releases.md)

---

## Overview

**Git reflog** (reference log) records every time HEAD moves in your local repository. It's your safety net — even when commits seem "lost" after a reset, rebase, or branch deletion, reflog can help you find and recover them.

---

## What is Reflog?

```
┌──────────────────────────────────────────────────────────────┐
│           REFLOG — YOUR SAFETY NET                          │
│                                                              │
│  Every time HEAD changes, Git records an entry:             │
│                                                              │
│  $ git reflog                                               │
│  abc1234 HEAD@{0}: commit: Add user auth                    │
│  def5678 HEAD@{1}: checkout: moving from main to feature    │
│  ghi9012 HEAD@{2}: commit: Update README                    │
│  jkl3456 HEAD@{3}: reset: moving to HEAD~2                  │
│  mno7890 HEAD@{4}: commit: Important work                   │
│  pqr1234 HEAD@{5}: commit: More work                        │
│                                                              │
│  HEAD@{0} = current state                                   │
│  HEAD@{1} = previous state                                  │
│  HEAD@{n} = n states ago                                    │
│                                                              │
│  Reflog is LOCAL only — not shared with remotes             │
│  Entries expire after ~90 days by default                   │
└──────────────────────────────────────────────────────────────┘
```

---

## Reading the Reflog

```bash
# Show HEAD reflog
$ git reflog
# Or:
$ git reflog show HEAD

# Show reflog for a specific branch
$ git reflog show main
$ git reflog show feature/login

# Show with dates
$ git reflog --date=relative
abc1234 HEAD@{2 hours ago}: commit: Add user auth
def5678 HEAD@{3 hours ago}: checkout: moving from main to feature

# Show with ISO dates
$ git reflog --date=iso

# Limit output
$ git reflog -n 10    # Last 10 entries
```

---

## Recovery Scenarios

### Scenario 1: Recover from `git reset --hard`

```
┌──────────────────────────────────────────────────────────────┐
│  RECOVER AFTER git reset --hard                             │
│                                                              │
│  You had:    ●──●──●──●──●                                  │
│                          ▲ HEAD (abc1234)                    │
│                                                              │
│  $ git reset --hard HEAD~3                                  │
│                                                              │
│  Now:        ●──●                                           │
│                 ▲ HEAD                                       │
│                                                              │
│  The 3 commits seem GONE... but reflog has them!            │
│                                                              │
│  $ git reflog                                               │
│  def5678 HEAD@{0}: reset: moving to HEAD~3                  │
│  abc1234 HEAD@{1}: commit: Important commit 3  ← HERE!     │
│                                                              │
│  $ git reset --hard abc1234                                 │
│                                                              │
│  Restored:   ●──●──●──●──●                                  │
│                          ▲ HEAD (abc1234)                    │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Step by step:
$ git reflog                     # Find the commit
$ git reset --hard abc1234       # Restore to that point
```

### Scenario 2: Recover a Deleted Branch

```bash
# Oops — deleted a branch with unmerged work
$ git branch -D feature/important-work
# Deleted branch feature/important-work (was abc1234)

# Find it in reflog
$ git reflog | grep "important-work"
# Or just scan the reflog for the last commit on that branch

# Recreate the branch
$ git branch feature/important-work abc1234
# Or:
$ git switch -c feature/important-work abc1234
```

### Scenario 3: Recover from a Bad Rebase

```bash
# Interactive rebase went wrong
$ git reflog
abc1234 HEAD@{0}: rebase (finish): ...
def5678 HEAD@{1}: rebase (pick): ...
ghi9012 HEAD@{2}: rebase (start): ...
jkl3456 HEAD@{3}: checkout: moving from feature to feature
mno7890 HEAD@{4}: commit: Last good commit  ← BEFORE rebase

# Restore to pre-rebase state
$ git reset --hard mno7890
```

### Scenario 4: Recover from Accidental `git checkout`

```bash
# Accidentally overwrote local changes
$ git checkout -- important-file.js   # Oops!

# If the changes were previously committed or staged,
# reflog + reset can help.
# If changes were NEVER staged or committed — they're lost.
# (This is why frequent commits matter!)
```

---

## Reflog vs Log

```
┌──────────────────────────────────────────────────────────────┐
│        git log vs git reflog                                │
│                                                              │
│  git log:                                                   │
│  • Shows COMMIT history (the DAG)                           │
│  • Shared across clones                                     │
│  • Lost commits don't appear                                │
│  • Follows parent chain                                     │
│                                                              │
│  git reflog:                                                │
│  • Shows HEAD MOVEMENT history                              │
│  • LOCAL only (not shared)                                  │
│  • "Lost" commits still appear                              │
│  • Records checkouts, resets, rebases, commits              │
│  • Expires (~90 days for unreachable, ~30 days for heads)   │
│                                                              │
│  Think of it this way:                                      │
│  log    = "What commits are in this branch?"                │
│  reflog = "Where has HEAD been recently?"                   │
└──────────────────────────────────────────────────────────────┘
```

---

## Using Reflog References

```bash
# You can use reflog references anywhere you use a commit hash

# Reset to where HEAD was 5 moves ago
$ git reset --hard HEAD@{5}

# Diff against previous HEAD position
$ git diff HEAD@{0} HEAD@{3}

# Create branch from reflog entry
$ git branch recovered HEAD@{7}

# Show a reflog entry
$ git show HEAD@{4}

# Time-based references
$ git diff main@{yesterday}
$ git diff main@{2.hours.ago}
$ git log main@{1.week.ago}..main
```

---

## Reflog Expiration

```bash
# Check expiration settings
$ git config gc.reflogExpire          # Default: 90 days
$ git config gc.reflogExpireUnreachable  # Default: 30 days

# Change expiration
$ git config --global gc.reflogExpire "180 days"

# Manually expire old entries
$ git reflog expire --expire=now --all

# Garbage collection (removes unreachable objects)
$ git gc
# ⚠️ After gc, expired reflog entries' objects are truly gone
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Can't find lost commit in reflog | Use `git fsck --unreachable` to find dangling commits |
| Reflog is empty | You may be in a fresh clone; reflog only has YOUR local operations |
| "fatal: bad revision" with reflog ref | The reflog entry may have expired; try `git fsck` |
| Need to recover on a different machine | Reflog is local only — you can't use clone's reflog for another machine |
| Reflog entries expired | Increase `gc.reflogExpire` for future protection |
| Lost uncommitted changes (never staged) | Unfortunately, Git can't recover changes that were never added to the index or committed |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git reflog` | Show HEAD movement history |
| `git reflog show <branch>` | Show reflog for a specific branch |
| `git reflog --date=relative` | Show with human-readable dates |
| `git reset --hard HEAD@{n}` | Restore to a previous state |
| `git branch <name> HEAD@{n}` | Create branch from reflog entry |
| `git show HEAD@{n}` | Inspect a reflog entry |
| `git diff HEAD@{0} HEAD@{3}` | Compare reflog states |
| `git fsck --unreachable` | Find dangling/lost objects |

---

## Quick Revision Questions

1. **What is the reflog and what does it record?**
   <details><summary>Answer</summary>The reflog (reference log) records every time HEAD moves in your local repository — commits, checkouts, resets, rebases, merges, etc. It's a chronological log of your local HEAD movements.</details>

2. **How do you recover a commit after `git reset --hard`?**
   <details><summary>Answer</summary>Run `git reflog` to find the hash of the lost commit (it will still be listed). Then `git reset --hard <hash>` to restore your branch to that commit.</details>

3. **What is the difference between `git log` and `git reflog`?**
   <details><summary>Answer</summary>`git log` shows the commit history (DAG) of the current branch — lost commits don't appear. `git reflog` shows all HEAD movements locally, including "lost" commits from resets or rebases. Reflog is local-only and expires.</details>

4. **How long does the reflog keep entries?**
   <details><summary>Answer</summary>By default, 90 days for reachable entries and 30 days for unreachable entries. After expiration and garbage collection (`git gc`), the actual objects may be permanently removed.</details>

5. **How do you recover a deleted branch?**
   <details><summary>Answer</summary>Use `git reflog` to find the last commit hash from the deleted branch. Then recreate it: `git branch <branch-name> <hash>`. The deletion message in reflog often shows the commit hash directly.</details>

---

[← Previous: Resetting and Reverting](03-resetting-and-reverting.md) | [Back to README](../README.md) | [Next: Tags and Releases →](05-tags-and-releases.md)
