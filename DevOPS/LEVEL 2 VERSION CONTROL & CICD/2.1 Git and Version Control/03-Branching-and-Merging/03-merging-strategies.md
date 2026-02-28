# Chapter 3.3: Merging Strategies (Fast-Forward, 3-Way)

[← Previous: Creating and Switching Branches](02-creating-and-switching-branches.md) | [Back to README](../README.md) | [Next: Merge Conflicts Resolution →](04-merge-conflicts-resolution.md)

---

## Overview

Merging combines the work from different branches. Git uses different merging strategies depending on the branch history. Understanding these strategies is key to maintaining a clean and meaningful project history.

---

## Fast-Forward Merge

A fast-forward merge happens when the target branch has **no new commits** since the branch was created. Git simply moves the pointer forward.

```
┌──────────────────────────────────────────────────────────────┐
│              FAST-FORWARD MERGE                             │
│                                                              │
│  BEFORE: git switch main && git merge feature               │
│                                                              │
│  ●───────●───────●───────●───────●                          │
│  C1      C2      C3      F1      F2                         │
│                  ▲                ▲                          │
│                  │                │                          │
│                main            feature ← HEAD               │
│                                                              │
│  main has NO new commits since feature branched              │
│                                                              │
│  AFTER: (fast-forward — just move the pointer)              │
│                                                              │
│  ●───────●───────●───────●───────●                          │
│  C1      C2      C3      F1      F2                         │
│                                   ▲                         │
│                                   │                         │
│                              main ← HEAD                    │
│                              feature                        │
│                                                              │
│  Result: main pointer "fast-forwarded" to F2                │
│  No merge commit created — linear history preserved         │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Fast-forward merge
$ git switch main
$ git merge feature/login
# Output: "Fast-forward"

# Force a merge commit even when fast-forward is possible
$ git merge --no-ff feature/login
```

---

## Three-Way Merge

A three-way merge happens when **both branches have new commits**. Git creates a new **merge commit** with two parents.

```
┌──────────────────────────────────────────────────────────────┐
│                THREE-WAY MERGE                              │
│                                                              │
│  BEFORE: git switch main && git merge feature               │
│                                                              │
│                   ●───────●           (feature)             │
│                  / F1      F2                                │
│  ●───────●───────●───────●───────●     (main)              │
│  C1      C2      C3      M1      M2                        │
│                                   ▲                         │
│                                 main ← HEAD                 │
│                                                              │
│  Both branches have new commits since they diverged          │
│                                                              │
│  AFTER: (three-way merge — creates merge commit)            │
│                                                              │
│                   ●───────●                                 │
│                  / F1      F2\                               │
│  ●───────●───────●───────●───────●───────●                  │
│  C1      C2      C3      M1      M2     MERGE               │
│                   ▲                       ▲                  │
│                   │                       │                  │
│                (common                  main ← HEAD         │
│                ancestor)                                    │
│                                                              │
│  MERGE commit has TWO parents: M2 and F2                    │
│  Git uses C3 as the "common ancestor" (merge base)          │
│                                                              │
│  The "3 ways": ancestor (C3), main (M2), feature (F2)      │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Three-way merge
$ git switch main
$ git merge feature/payments
# Git opens editor for merge commit message
# Default: "Merge branch 'feature/payments' into main"
```

### Why It's Called "Three-Way"

Git compares **three versions** of each file:
1. **Common ancestor** (merge base) — the last commit shared by both branches
2. **Ours** (current branch) — the file on main
3. **Theirs** (incoming branch) — the file on feature

```
┌─────────────────────────────────────────────────────┐
│         THREE-WAY MERGE LOGIC                       │
│                                                     │
│   Ancestor    Ours (main)    Theirs (feature)      │
│  ┌─────────┐  ┌─────────┐    ┌─────────┐           │
│  │ Line A  │  │ Line A  │    │ Line A  │   → A     │
│  │ Line B  │  │ Line B' │    │ Line B  │   → B'    │
│  │ Line C  │  │ Line C  │    │ Line C' │   → C'    │
│  │ Line D  │  │ Line D  │    │ Line D  │   → D     │
│  └─────────┘  └─────────┘    └─────────┘           │
│                                                     │
│  Rules:                                             │
│  • If only ONE side changed → take the change       │
│  • If BOTH sides changed the SAME way → take it    │
│  • If BOTH sides changed DIFFERENTLY → CONFLICT!   │
│  • If NEITHER changed → keep original               │
└─────────────────────────────────────────────────────┘
```

---

## --no-ff (No Fast-Forward)

Forces a merge commit even when fast-forward is possible. This preserves the branch history.

```
┌──────────────────────────────────────────────────────────────┐
│           FAST-FORWARD vs --no-ff                           │
│                                                              │
│  Fast-Forward (default):                                     │
│                                                              │
│  ●───────●───────●───────●───────●                          │
│  C1      C2      C3      F1      F2                         │
│                                   ▲                         │
│                                 main                        │
│                                                              │
│  Linear history — can't tell F1/F2 were on a branch        │
│                                                              │
│  ─────────────────────────────────────────────────          │
│                                                              │
│  --no-ff:                                                    │
│                                                              │
│                   ●───────●                                 │
│                  / F1      F2\                               │
│  ●───────●───────●────────────●───── MERGE                  │
│  C1      C2      C3            ▲                            │
│                               main                          │
│                                                              │
│  Merge commit preserves that F1/F2 were on a branch        │
│  ✅ Recommended for feature branches                         │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Always create a merge commit
$ git merge --no-ff feature/login -m "Merge feature/login into main"

# Configure to always use --no-ff for a specific branch
$ git config branch.main.mergeoptions "--no-ff"
```

---

## Merge Strategies in Detail

Git has several merge algorithms. `recursive` (or `ort` in newer Git) is the default:

| Strategy | Command | Use Case |
|----------|---------|----------|
| **Fast-forward** | `git merge` (auto) | Simple linear history |
| **Recursive (ort)** | Default for 3-way | Most common, handles renames |
| **Ours** | `git merge -s ours` | Keep our version, discard theirs entirely |
| **Octopus** | Auto for 3+ branches | Merge multiple branches at once |
| **Subtree** | `git merge -s subtree` | Merging a subproject |

```bash
# Merge using "ours" strategy (discards all changes from the other branch)
$ git merge -s ours feature/experiment
# Useful for: "marking" a branch as merged without taking its changes

# Merge multiple branches at once (octopus)
$ git merge feature/a feature/b feature/c
```

---

## Squash Merge

Combines all commits from a branch into a **single commit** on the target branch — no merge commit, no branch reference.

```
┌──────────────────────────────────────────────────────────────┐
│              SQUASH MERGE                                   │
│                                                              │
│  BEFORE:                                                     │
│                   ●───────●───────●                         │
│                  / F1      F2      F3                        │
│  ●───────●───────●                    (main)                │
│  C1      C2      C3                                         │
│                                                              │
│  $ git merge --squash feature                               │
│  $ git commit -m "Add complete feature"                     │
│                                                              │
│  AFTER:                                                      │
│  ●───────●───────●───────●                                  │
│  C1      C2      C3      S                                  │
│                           ▲                                 │
│                         main                                │
│                                                              │
│  S contains ALL changes from F1+F2+F3, but as ONE commit   │
│  No merge commit, no reference to the feature branch        │
│  History is clean but branch history is lost                │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Squash merge
$ git switch main
$ git merge --squash feature/login
$ git commit -m "feat: add complete login system"
```

---

## Comparison of Merge Types

| Aspect | Fast-Forward | No-FF Merge | Squash Merge |
|--------|-------------|-------------|--------------|
| **Merge commit** | No | Yes | No |
| **Preserves branch history** | No | Yes | No |
| **Linear history** | Yes | No | Yes |
| **Commit count** | Same as branch | Branch + 1 merge | 1 single commit |
| **Best for** | Small changes | Feature branches | Noisy branch history |
| **Traceability** | Low | High | Medium |

---

## Real-World Scenarios

### Scenario 1: Simple Feature Merge

```bash
$ git switch main
$ git pull origin main
$ git merge --no-ff feature/add-footer -m "Merge feature/add-footer"
$ git push origin main
$ git branch -d feature/add-footer
$ git push origin --delete feature/add-footer
```

### Scenario 2: Squash-Merging a Messy Branch

```bash
# Feature branch has many "WIP" commits
$ git log feature/experiment --oneline
abc1234 WIP
def5678 WIP: more stuff
9876543 Fix thing
1111111 WIP: try something

# Squash into a clean single commit
$ git switch main
$ git merge --squash feature/experiment
$ git commit -m "feat: add experimental algorithm"
```

### Scenario 3: Merge Preview (Dry Run)

```bash
# See what WOULD be merged (without actually merging)
$ git log main..feature/login --oneline   # Commits to be merged
$ git diff main...feature/login --stat    # Files that would change
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Merge created unexpected conflicts | Check if you need to update your branch first: `git pull origin main` |
| Accidentally merged the wrong branch | `git reset --hard HEAD~1` (if not pushed), or `git revert -m 1 <merge-hash>` |
| Want to abort a merge in progress | `git merge --abort` |
| Fast-forward when you wanted a merge commit | Use `--no-ff` flag |
| Too many merge commits cluttering history | Consider squash merge or rebasing |

---

## Summary Table

| Merge Type | Command | Creates Merge Commit | Preserves Branch History |
|-----------|---------|---------------------|------------------------|
| **Fast-Forward** | `git merge <branch>` | No | No |
| **No-FF** | `git merge --no-ff <branch>` | Yes | Yes |
| **Three-Way** | `git merge <branch>` (auto) | Yes | Yes |
| **Squash** | `git merge --squash <branch>` | No (manual commit) | No |
| **Ours** | `git merge -s ours <branch>` | Yes | Marks as merged only |

---

## Quick Revision Questions

1. **When does a fast-forward merge occur?**
   <details><summary>Answer</summary>When the current branch has no new commits since the feature branch was created. Git simply moves the branch pointer forward to the latest commit on the feature branch.</details>

2. **What are the "three ways" in a three-way merge?**
   <details><summary>Answer</summary>The common ancestor (merge base), the current branch's version ("ours"), and the incoming branch's version ("theirs"). Git compares all three to determine the correct merged result.</details>

3. **Why would you use `--no-ff` instead of a regular merge?**
   <details><summary>Answer</summary>To create a merge commit even when fast-forward is possible. This preserves the fact that work was done on a separate branch, making the project history more informative and easier to revert if needed.</details>

4. **What is a squash merge and when would you use it?**
   <details><summary>Answer</summary>A squash merge combines all commits from a branch into a single commit on the target branch. Use it when the branch has many small/WIP commits and you want a clean, single-commit history.</details>

5. **How do you preview what a merge would do without actually merging?**
   <details><summary>Answer</summary>Use `git log main..feature --oneline` to see commits that would be merged, and `git diff main...feature --stat` to see which files would change.</details>

6. **How do you abort a merge that's in progress?**
   <details><summary>Answer</summary>Run `git merge --abort`. This restores your working directory and staging area to the state before the merge began.</details>

---

[← Previous: Creating and Switching Branches](02-creating-and-switching-branches.md) | [Back to README](../README.md) | [Next: Merge Conflicts Resolution →](04-merge-conflicts-resolution.md)
