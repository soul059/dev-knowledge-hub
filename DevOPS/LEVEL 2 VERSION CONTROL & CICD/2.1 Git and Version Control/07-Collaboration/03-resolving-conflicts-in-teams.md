# Chapter 7.3: Resolving Conflicts in Teams

[← Previous: Code Reviews](02-code-reviews.md) | [Back to README](../README.md) | [Next: Commit Message Conventions →](04-commit-message-conventions.md)

---

## Overview

In team environments, **merge conflicts** are inevitable when multiple developers edit the same files. This chapter covers strategies for **preventing, detecting, and resolving** conflicts efficiently in collaborative workflows.

---

## Why Conflicts Happen in Teams

```
┌──────────────────────────────────────────────────────────────┐
│  CONFLICT SCENARIO                                          │
│                                                              │
│  Alice and Bob both work on the same file:                  │
│                                                              │
│  main:     ●──────────────────●                             │
│             \                / (conflict!)                   │
│  alice:     ●───●──●        /                               │
│              (edits login.js line 42)                        │
│             \              /                                │
│  bob:        ●───●──●                                       │
│              (edits login.js line 42)                        │
│                                                              │
│  Both changed the same line → CONFLICT on merge!            │
└──────────────────────────────────────────────────────────────┘
```

### Common Causes

| Cause | Example |
|-------|---------|
| Same line edited | Two devs modify the same function |
| Adjacent edits | Changes near each other confuse merge |
| File renamed + edited | One renames, another edits the original |
| Long-lived branches | More time = more divergence = more conflicts |
| Large files | Bigger files = higher chance of overlap |

---

## Preventing Conflicts

```
┌──────────────────────────────────────────────────────────────┐
│  CONFLICT PREVENTION STRATEGIES                             │
│                                                              │
│  1. Keep branches SHORT-LIVED                               │
│     ○ Merge within 1-3 days                                 │
│     ○ Small, focused features                               │
│                                                              │
│  2. Pull/rebase FREQUENTLY                                  │
│     ○ $ git pull --rebase origin main   (daily!)            │
│     ○ Stay in sync with main                                │
│                                                              │
│  3. Communicate with your team                              │
│     ○ "I'm working on auth.js"                              │
│     ○ Use task boards to show WIP                           │
│                                                              │
│  4. Modular code architecture                               │
│     ○ Separate concerns into files                          │
│     ○ Avoid "god files" with everything                     │
│                                                              │
│  5. Use feature flags                                       │
│     ○ Merge to main early (disabled)                        │
│     ○ Enable when ready                                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Resolving Conflicts Step by Step

### Step 1: Identify the Conflict

```bash
$ git merge main
# Auto-merging src/auth.js
# CONFLICT (content): Merge conflict in src/auth.js
# Automatic merge failed; fix conflicts and then commit the result.

$ git status
# both modified:   src/auth.js
```

### Step 2: Examine the Conflict Markers

```javascript
function login(username, password) {
<<<<<<< HEAD (your branch)
    const token = jwt.sign({ user: username }, SECRET_KEY);
    return { token, expiresIn: '24h' };
=======
    const token = jwt.sign({ user: username }, process.env.JWT_SECRET);
    return { token, expiresIn: '1h' };
>>>>>>> main
}
```

```
┌──────────────────────────────────────────────────────────────┐
│  READING CONFLICT MARKERS                                   │
│                                                              │
│  <<<<<<< HEAD                                               │
│  Your branch's version of the code                          │
│  =======                                                    │
│  The other branch's version of the code                     │
│  >>>>>>> main                                               │
│                                                              │
│  YOUR JOB: Combine both versions into the correct result,   │
│  then remove ALL marker lines (<<<, ===, >>>)               │
└──────────────────────────────────────────────────────────────┘
```

### Step 3: Resolve the Conflict

```javascript
// Option A: Keep yours
function login(username, password) {
    const token = jwt.sign({ user: username }, SECRET_KEY);
    return { token, expiresIn: '24h' };
}

// Option B: Keep theirs
function login(username, password) {
    const token = jwt.sign({ user: username }, process.env.JWT_SECRET);
    return { token, expiresIn: '1h' };
}

// Option C: Combine both (usually the best approach!)
function login(username, password) {
    const token = jwt.sign({ user: username }, process.env.JWT_SECRET);
    return { token, expiresIn: '24h' };
}
```

### Step 4: Mark Resolved and Commit

```bash
$ git add src/auth.js
$ git commit -m "Merge main: resolve auth.js conflict, use env var for JWT secret"
```

---

## Merge Tools

```bash
# Configure a merge tool
$ git config --global merge.tool vscode
$ git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'

# Launch merge tool
$ git mergetool

# Other popular tools:
# • VS Code (built-in 3-way merge editor)
# • IntelliJ/WebStorm
# • Beyond Compare
# • KDiff3
# • Meld
```

```
┌──────────────────────────────────────────────────────────────┐
│  VS CODE 3-WAY MERGE EDITOR                                │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────┐                  │
│  │ CURRENT (yours)  │  │ INCOMING (theirs)│                  │
│  │                 │  │                  │                  │
│  │ Accept Current  │  │ Accept Incoming  │                  │
│  │ Accept Both     │  │                  │                  │
│  └────────┬────────┘  └────────┬─────────┘                  │
│           │                    │                             │
│           ▼                    ▼                             │
│  ┌──────────────────────────────────┐                       │
│  │ RESULT (editable)                │                       │
│  │                                  │                       │
│  │ Final merged version goes here   │                       │
│  └──────────────────────────────────┘                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Team Conflict Resolution Patterns

### Pattern 1: Rebase Before PR

```bash
# Before submitting or updating a PR, rebase onto main
$ git fetch origin
$ git rebase origin/main
# Fix any conflicts during rebase
$ git push --force-with-lease origin feature/my-feature
# PR will now merge cleanly
```

### Pattern 2: Designated Integrator

```
┌──────────────────────────────────────────────────────────────┐
│  INTEGRATOR MODEL                                           │
│                                                              │
│  Dev A ──PR──►  ┌──────────┐                                │
│                 │ Lead /    │ ───► main                      │
│  Dev B ──PR──► │ Integrator│                                │
│                 └──────────┘                                │
│  Dev C ──PR──►  Resolves conflicts                          │
│                 during merge                                │
└──────────────────────────────────────────────────────────────┘
```

### Pattern 3: Using `git rerere`

```bash
# Enable rerere (Reuse Recorded Resolution)
$ git config --global rerere.enabled true

# Git remembers how you resolved a conflict and
# automatically applies the same resolution next time!

# See recorded resolutions
$ git rerere status
$ git rerere diff
```

---

## Handling Specific Conflict Types

### Binary Files

```bash
# Git can't merge binary files — you choose one version
$ git checkout --ours image.png     # Keep your version
$ git checkout --theirs image.png   # Keep their version
$ git add image.png
```

### Deleted vs Modified

```bash
# They deleted a file you modified
# Decide: keep the file or accept deletion?
$ git add file.txt      # Keep the file (your changes)
$ git rm file.txt       # Accept deletion (their change)
```

### Package Lock Files

```bash
# For package-lock.json conflicts:
$ git checkout --theirs package-lock.json
$ npm install    # Regenerate from merged package.json
$ git add package-lock.json
```

---

## Real-World Scenarios

### Scenario: Team of 5, All Editing Config

```bash
# Problem: 5 devs all add their settings to config.yml

# Prevention: Structure config into separate files
config/
  ├── auth.yml         # Auth team owns this
  ├── database.yml     # Backend team
  ├── frontend.yml     # Frontend team
  └── logging.yml      # DevOps team

# Each team only edits their own file — no conflicts!
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Conflicts on every merge | Rebase frequently; keep branches short-lived |
| Can't understand conflict markers | Use a visual merge tool (VS Code, IntelliJ) |
| Resolved conflict incorrectly | `git merge --abort` to start over, or revert the merge commit |
| Conflicts in auto-generated files | Regenerate after resolving source conflicts (e.g., `npm install`) |
| Same conflict keeps recurring | Enable `git rerere` to record and replay resolutions |

---

## Summary Table

| Strategy | Description |
|----------|-------------|
| Short-lived branches | Less divergence = fewer conflicts |
| Frequent rebasing | Stay in sync with main daily |
| Communication | Let team know what you're working on |
| Modular architecture | Separate files by concern/owner |
| `git rerere` | Auto-apply previously recorded conflict resolutions |
| Visual merge tools | 3-way merge editors for complex conflicts |
| Integrator pattern | One person responsible for merging |

---

## Quick Revision Questions

1. **What are the three sections in a conflict marker?**
   <details><summary>Answer</summary>The section between `<<<<<<< HEAD` and `=======` contains your branch's version. The section between `=======` and `>>>>>>> branch-name` contains the other branch's version. You must combine both into the correct result and remove all marker lines.</details>

2. **Name three strategies to prevent merge conflicts.**
   <details><summary>Answer</summary>1) Keep branches short-lived (merge within 1-3 days). 2) Pull/rebase from main frequently (daily). 3) Communicate with team about who is editing which files. Also: modular code architecture and using feature flags.</details>

3. **What is `git rerere` and how does it help?**
   <details><summary>Answer</summary>`git rerere` (Reuse Recorded Resolution) records how you resolve conflicts and automatically applies the same resolution if the same conflict occurs again. Enable with `git config --global rerere.enabled true`. It's especially useful during repeated rebases.</details>

4. **How do you resolve a conflict in a binary file?**
   <details><summary>Answer</summary>Git can't merge binary files, so you must choose one version: `git checkout --ours file.png` to keep yours, or `git checkout --theirs file.png` to keep theirs. Then `git add file.png` to mark it resolved.</details>

5. **What should you do if you resolve a conflict incorrectly?**
   <details><summary>Answer</summary>If you haven't committed yet, use `git merge --abort` to cancel and start over. If you already committed, you can `git revert` the merge commit. You can also `git reset --hard` to the commit before the merge (if not pushed).</details>

---

[← Previous: Code Reviews](02-code-reviews.md) | [Back to README](../README.md) | [Next: Commit Message Conventions →](04-commit-message-conventions.md)
