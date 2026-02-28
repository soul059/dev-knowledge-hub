# Chapter 3.6: Cherry-Picking

[← Previous: Rebasing vs Merging](05-rebasing-vs-merging.md) | [Back to README](../README.md) | [Next: Remote Concepts →](../04-Remote-Repositories/01-remote-concepts.md)

---

## Overview

Cherry-picking allows you to **select specific commits** from one branch and apply them to another branch. Unlike merge or rebase (which bring all commits), cherry-pick gives you surgical precision — picking exactly the commits you want.

---

## How Cherry-Pick Works

```
┌──────────────────────────────────────────────────────────────┐
│              git cherry-pick                                │
│                                                              │
│  BEFORE cherry-pick:                                         │
│                                                              │
│  main:       ●───●───●                                      │
│              C1  C2  C3                                      │
│                                                              │
│  feature:    ●───●───●───●───●                              │
│              C1  C2  F1  F2  F3                              │
│                         ▲                                   │
│                    We want ONLY this commit                  │
│                                                              │
│  $ git switch main                                          │
│  $ git cherry-pick <F2-hash>                                │
│                                                              │
│  AFTER cherry-pick:                                          │
│                                                              │
│  main:       ●───●───●───●                                  │
│              C1  C2  C3  F2'    ← New commit (same changes) │
│                                                              │
│  feature:    ●───●───●───●───●  ← Unchanged                │
│              C1  C2  F1  F2  F3                              │
│                                                              │
│  F2' has a NEW hash but contains the SAME changes as F2     │
└──────────────────────────────────────────────────────────────┘
```

---

## Cherry-Pick Commands

### Basic Cherry-Pick

```bash
# Apply a single commit to the current branch
$ git cherry-pick <commit-hash>

# Example:
$ git switch main
$ git cherry-pick a1b2c3d
```

### Cherry-Pick Multiple Commits

```bash
# Pick specific commits (space-separated)
$ git cherry-pick <hash1> <hash2> <hash3>

# Pick a range of commits (exclusive start .. inclusive end)
$ git cherry-pick A..B
# This picks all commits AFTER A up to and INCLUDING B

# Pick a range (inclusive of both ends)
$ git cherry-pick A^..B
# This picks commit A, everything after A, up to and including B
```

### Cherry-Pick Options

```bash
# Apply changes but DON'T commit (stage only)
$ git cherry-pick --no-commit <hash>
$ git cherry-pick -n <hash>

# Edit the commit message before committing
$ git cherry-pick --edit <hash>
$ git cherry-pick -e <hash>

# Append "cherry picked from commit <hash>" to message
$ git cherry-pick -x <hash>

# Record who cherry-picked (adds Signed-off-by)
$ git cherry-pick --signoff <hash>
$ git cherry-pick -s <hash>
```

---

## Cherry-Pick Workflow Diagram

```
┌──────────────────────────────────────────────────────────────┐
│           CHERRY-PICK WORKFLOW                               │
│                                                              │
│  1. Identify the commit you need                            │
│     $ git log --oneline feature/payments                    │
│                                                              │
│     abc1234 Fix payment validation bug     ← want this!     │
│     def5678 Add payment gateway                             │
│     ghi9012 WIP payment form                                │
│                                                              │
│  2. Switch to the target branch                             │
│     $ git switch main                                       │
│                                                              │
│  3. Cherry-pick the commit                                  │
│     $ git cherry-pick abc1234                               │
│                                                              │
│  4. Verify                                                  │
│     $ git log --oneline -3                                  │
│                                                              │
│     xyz7890 Fix payment validation bug     ← new commit!    │
│     111aaaa Previous main commit                            │
│     222bbbb Older main commit                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Handling Cherry-Pick Conflicts

```bash
# If a conflict occurs during cherry-pick:
$ git cherry-pick abc1234
# CONFLICT (content): Merge conflict in file.txt

# Option 1: Resolve and continue
$ code file.txt          # Fix conflict markers
$ git add file.txt       # Stage resolution
$ git cherry-pick --continue

# Option 2: Abort the cherry-pick
$ git cherry-pick --abort

# Option 3: Skip this commit (in a multi-pick)
$ git cherry-pick --skip
```

```
┌──────────────────────────────────────────────────────────────┐
│        CONFLICT RESOLUTION FLOW                             │
│                                                              │
│  git cherry-pick <hash>                                     │
│         │                                                   │
│         ▼                                                   │
│  ┌─ Conflict? ──┐                                           │
│  │              │                                           │
│  No             Yes                                         │
│  │              │                                           │
│  ▼              ▼                                           │
│  Done!    Fix conflicts                                     │
│           │              │                                  │
│           ▼              ▼                                  │
│     git add <files>    git cherry-pick --abort              │
│           │            (cancel everything)                  │
│           ▼                                                 │
│     git cherry-pick --continue                              │
│           │                                                 │
│           ▼                                                 │
│         Done!                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Common Use Cases

### 1. Hotfix from Feature to Main

```bash
# A bug fix was committed on a feature branch
# but needs to go into production (main) immediately

$ git switch main
$ git cherry-pick <bugfix-commit-hash>
$ git push origin main
```

### 2. Backporting to a Release Branch

```bash
# A fix on main needs to be applied to a release branch
$ git switch release/2.0
$ git cherry-pick <fix-hash>
```

### 3. Extracting a Commit from the Wrong Branch

```bash
# Oops — committed to main instead of feature
$ git switch feature/login
$ git cherry-pick <accidental-hash>   # Copy commit here

$ git switch main
$ git reset --hard HEAD~1             # Remove from main
```

### 4. Combining Specific Commits Without Merging

```bash
# Pick only the commits you want from a branch
$ git switch main
$ git cherry-pick -n <hash1>    # Stage but don't commit
$ git cherry-pick -n <hash2>    # Stage more changes
$ git commit -m "Combined selected changes"
```

---

## Cherry-Pick vs Other Integration Methods

```
┌──────────────────────────────────────────────────────────────┐
│     CHOOSING THE RIGHT INTEGRATION METHOD                   │
│                                                              │
│  Need ALL commits from a branch?                            │
│    ├─ Want linear history? → git rebase                     │
│    └─ Want merge context?  → git merge                      │
│                                                              │
│  Need SPECIFIC commits only?                                │
│    └─ git cherry-pick                                       │
│                                                              │
│  Need to apply a patch/diff?                                │
│    └─ git apply / git am                                    │
│                                                              │
│  ┌────────────┬───────────────────────────┐                 │
│  │ Method     │ What it brings            │                 │
│  ├────────────┼───────────────────────────┤                 │
│  │ merge      │ All commits + merge node  │                 │
│  │ rebase     │ All commits (rewritten)   │                 │
│  │ cherry-pick│ Selected commits only     │                 │
│  │ patch/am   │ Diff from email/file      │                 │
│  └────────────┴───────────────────────────┘                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Use `-x` flag | Records source commit hash in message for traceability |
| Cherry-pick sparingly | Duplicates commits — can cause confusion if branch is later merged |
| Prefer merge/rebase for full branches | Cherry-pick is for surgical, targeted use |
| Check for dependencies | A commit may depend on previous commits that aren't picked |
| Don't cherry-pick merge commits | Merge commits are complex — use `git merge` instead |
| Use `--no-commit` for grouping | Combine multiple picks into one meaningful commit |

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "bad object" error | Double-check the commit hash; ensure it exists in your repo |
| Duplicate commits after merging | Expected — cherry-pick creates new commits with new hashes |
| Cherry-pick introduces unexpected changes | The commit may depend on prior commits from the source branch |
| "could not apply" error | Conflict — resolve files, `git add`, then `git cherry-pick --continue` |
| Want to undo a cherry-pick | `git reset --hard HEAD~1` (if not pushed) or `git revert <hash>` |

---

## Summary Table

| Feature | Details |
|---------|---------|
| **Purpose** | Apply specific commits to another branch |
| **Command** | `git cherry-pick <hash>` |
| **New commits?** | Yes — creates new commits with new hashes |
| **Modifies source?** | No — source branch is unchanged |
| **Conflicts possible?** | Yes — resolved like merge conflicts |
| **Multiple commits** | `git cherry-pick <h1> <h2>` or `A^..B` range |
| **Without committing** | `--no-commit` / `-n` |
| **Record source** | `-x` flag appends source hash to message |
| **Abort** | `git cherry-pick --abort` |
| **Continue after fix** | `git cherry-pick --continue` |

---

## Quick Revision Questions

1. **What does `git cherry-pick` do?**
   <details><summary>Answer</summary>It takes a specific commit from one branch and applies its changes as a new commit on the current branch. The new commit has the same changes but a different hash.</details>

2. **Does cherry-pick affect the source branch?**
   <details><summary>Answer</summary>No. The source branch remains completely unchanged. Cherry-pick creates a copy of the commit on the target branch.</details>

3. **What does the `-x` flag do in `git cherry-pick -x <hash>`?**
   <details><summary>Answer</summary>It appends "(cherry picked from commit <original-hash>)" to the commit message, making it easy to trace where the commit originated from.</details>

4. **How do you cherry-pick a range of commits?**
   <details><summary>Answer</summary>Use `git cherry-pick A^..B` to pick all commits from A to B inclusive, or `git cherry-pick A..B` to pick commits after A up to and including B.</details>

5. **When should you use cherry-pick vs merge?**
   <details><summary>Answer</summary>Use cherry-pick when you need only specific commits from a branch (e.g., a hotfix). Use merge when you want to integrate all commits from a branch. Cherry-pick should be used sparingly since it duplicates commits.</details>

6. **What happens if you cherry-pick a commit and later merge that branch?**
   <details><summary>Answer</summary>You may end up with duplicate commits (same changes, different hashes) in the history. Git usually handles this gracefully during merge, but it can cause confusion. This is why cherry-pick should be used sparingly.</details>

---

[← Previous: Rebasing vs Merging](05-rebasing-vs-merging.md) | [Back to README](../README.md) | [Next: Remote Concepts →](../04-Remote-Repositories/01-remote-concepts.md)
