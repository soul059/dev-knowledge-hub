# Chapter 6.2: Stashing

[← Previous: Interactive Rebase](01-interactive-rebase.md) | [Back to README](../README.md) | [Next: Resetting and Reverting →](03-resetting-and-reverting.md)

---

## Overview

**Git stash** temporarily shelves (saves) your uncommitted changes so you can work on something else, then come back and re-apply them later. Think of it as a clipboard for your working directory changes.

---

## How Stash Works

```
┌──────────────────────────────────────────────────────────────┐
│           GIT STASH FLOW                                    │
│                                                              │
│  Working Directory          Stash Stack                     │
│  ┌──────────────┐          ┌──────────────┐                 │
│  │ Modified     │ ──push──►│ stash@{0}    │ (latest)        │
│  │ files        │          │ stash@{1}    │                 │
│  │              │ ◄──pop───│ stash@{2}    │                 │
│  └──────────────┘          └──────────────┘                 │
│                                                              │
│  STASH = Stack (Last In, First Out)                         │
│  • git stash push  → saves changes, cleans working dir      │
│  • git stash pop   → restores changes, removes from stash   │
│  • git stash apply → restores changes, KEEPS in stash       │
└──────────────────────────────────────────────────────────────┘
```

---

## Basic Stash Commands

```bash
# Save all uncommitted changes (tracked files)
$ git stash
# Shorthand for: git stash push

# Save with a descriptive message
$ git stash push -m "WIP: halfway through login refactor"

# List all stashes
$ git stash list
stash@{0}: On feature/login: WIP: halfway through login refactor
stash@{1}: WIP on main: abc1234 Previous stash

# Apply most recent stash (and remove it from stash)
$ git stash pop

# Apply most recent stash (keep it in stash)
$ git stash apply

# Apply a specific stash
$ git stash apply stash@{2}
$ git stash pop stash@{1}

# Drop (delete) a specific stash
$ git stash drop stash@{0}

# Clear ALL stashes
$ git stash clear
```

---

## Stash Options

```bash
# Include untracked files
$ git stash push -u
$ git stash push --include-untracked

# Include untracked AND ignored files
$ git stash push -a
$ git stash push --all

# Stash specific files only
$ git stash push -m "stash only utils" src/utils.js src/helpers.js

# Stash interactively (choose which hunks to stash)
$ git stash push -p
$ git stash push --patch
```

```
┌──────────────────────────────────────────────────────────────┐
│  WHAT GETS STASHED BY DEFAULT                               │
│                                                              │
│  ✅ Modified tracked files (working directory)              │
│  ✅ Staged changes (index/staging area)                     │
│  ❌ Untracked files (need -u flag)                          │
│  ❌ Ignored files (need -a flag)                            │
│                                                              │
│  $ git stash              → tracked changes only            │
│  $ git stash push -u      → tracked + untracked            │
│  $ git stash push -a      → tracked + untracked + ignored  │
└──────────────────────────────────────────────────────────────┘
```

---

## Inspecting Stashes

```bash
# Show the diff of the most recent stash
$ git stash show
 src/auth.js | 15 +++++++++------
 src/api.js  |  8 +++++---
 2 files changed, 14 insertions(+), 9 deletions(-)

# Show full diff (patch format)
$ git stash show -p
$ git stash show -p stash@{2}
```

---

## Stash and Branch

```bash
# Create a new branch from a stash
$ git stash branch feature/from-stash stash@{0}
# This:
# 1. Creates the branch from the commit where stash was created
# 2. Applies the stash
# 3. Drops the stash (if apply succeeds)
```

```
┌──────────────────────────────────────────────────────────────┐
│  STASH BRANCH — USEFUL WHEN                                │
│                                                              │
│  You stashed changes on main but they should be on a        │
│  feature branch:                                            │
│                                                              │
│  $ git stash                         # Save work            │
│  $ git stash branch feature/new-work # Move to new branch   │
│                                                              │
│  Before:                                                    │
│  main:  ●──●──●  (stash saved here)                         │
│                                                              │
│  After:                                                      │
│  main:  ●──●──●                                              │
│                \                                            │
│  feature/:      ● (stash applied here)                      │
└──────────────────────────────────────────────────────────────┘
```

---

## Common Use Cases

### Use Case 1: Quick Branch Switch

```bash
# Working on feature, need to fix a bug on main
$ git stash push -m "WIP: user dashboard"
$ git switch main
$ git switch -c hotfix/critical-bug
# ... fix the bug ...
$ git commit -m "Fix critical bug"
$ git switch feature/dashboard
$ git stash pop
# Continue working where you left off
```

### Use Case 2: Pull with Dirty Working Directory

```bash
# Can't pull with uncommitted changes
$ git stash
$ git pull --rebase origin main
$ git stash pop
# Resolve any conflicts between your stash and pulled changes
```

### Use Case 3: Testing a Clean State

```bash
# Temporarily remove changes to test something
$ git stash
$ npm test    # Run tests on clean state
$ git stash pop
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Stash pop causes conflicts | Resolve conflicts, then `git add` the files. The stash is NOT dropped when conflicts occur |
| Accidentally dropped a stash | `git fsck --unreachable` may find the dangling stash commit |
| "No local changes to save" | Your working directory is clean; nothing to stash |
| Stash doesn't include new files | Use `git stash push -u` to include untracked files |
| Applied wrong stash | `git checkout -- .` to discard, then apply correct one |
| Too many stashes, can't find the right one | Use descriptive messages: `git stash push -m "description"` |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git stash` | Stash tracked changes |
| `git stash push -m "msg"` | Stash with description |
| `git stash push -u` | Include untracked files |
| `git stash list` | Show all stashes |
| `git stash show -p` | Show stash diff |
| `git stash pop` | Apply and remove latest stash |
| `git stash apply` | Apply but keep stash |
| `git stash drop stash@{n}` | Remove a specific stash |
| `git stash clear` | Remove all stashes |
| `git stash branch <name>` | Create branch from stash |
| `git stash push -p` | Interactive (partial) stash |

---

## Quick Revision Questions

1. **What does `git stash` do?**
   <details><summary>Answer</summary>It saves your uncommitted changes (both staged and unstaged modifications to tracked files) to a stack, and reverts your working directory to a clean state matching the latest commit.</details>

2. **What is the difference between `git stash pop` and `git stash apply`?**
   <details><summary>Answer</summary>`pop` applies the stash AND removes it from the stash stack. `apply` applies the stash but KEEPS it in the stash stack, so you can apply it again later or to a different branch.</details>

3. **How do you stash untracked (new) files?**
   <details><summary>Answer</summary>Use `git stash push -u` or `git stash push --include-untracked`. By default, `git stash` only saves changes to tracked files.</details>

4. **How do you create a branch from a stash?**
   <details><summary>Answer</summary>`git stash branch <branch-name> stash@{n}`. This creates a new branch from the commit where the stash was made, applies the stash, and drops it if successful.</details>

5. **What happens if `git stash pop` causes a conflict?**
   <details><summary>Answer</summary>Git applies the stash changes and marks conflicts. You need to resolve the conflicts manually and `git add` the resolved files. Importantly, the stash is NOT dropped when conflicts occur — you must drop it manually after resolving.</details>

---

[← Previous: Interactive Rebase](01-interactive-rebase.md) | [Back to README](../README.md) | [Next: Resetting and Reverting →](03-resetting-and-reverting.md)
