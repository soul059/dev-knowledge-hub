# Chapter 3.4: Merge Conflicts Resolution

[← Previous: Merging Strategies](03-merging-strategies.md) | [Back to README](../README.md) | [Next: Rebasing vs Merging →](05-rebasing-vs-merging.md)

---

## Overview

Merge conflicts occur when Git can't automatically combine changes from different branches because **both branches modified the same part of the same file differently**. Resolving conflicts is a fundamental skill for every developer.

---

## When Do Conflicts Happen?

```
┌──────────────────────────────────────────────────────────────┐
│              WHEN CONFLICTS OCCUR                           │
│                                                              │
│  Ancestor (C3):        main (M1):         feature (F1):     │
│  ┌──────────────┐      ┌──────────────┐   ┌──────────────┐  │
│  │ function     │      │ function     │   │ function     │  │
│  │ greet() {    │      │ greet() {    │   │ greet() {    │  │
│  │  return "Hi" │  →   │  return "Hey"│   │  return "Yo" │  │
│  │ }            │      │ }            │   │ }            │  │
│  └──────────────┘      └──────────────┘   └──────────────┘  │
│                                                              │
│  Both changed the SAME line DIFFERENTLY                     │
│  Git doesn't know which version to keep                     │
│  → CONFLICT! Human must decide.                             │
│                                                              │
│  ────────────────────────────────────────────────────        │
│                                                              │
│  NO conflict when:                                           │
│  • Different files modified                                  │
│  • Same file, different sections modified                    │
│  • Only one branch changed a line                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Anatomy of a Conflict

When a conflict occurs, Git marks the file with conflict markers:

```
┌──────────────────────────────────────────────────────────────┐
│              CONFLICT MARKERS IN A FILE                     │
│                                                              │
│  function greet() {                                          │
│  <<<<<<< HEAD                                               │
│    return "Hey there!";         ← YOUR current branch       │
│  =======                        ← separator                 │
│    return "Yo, what's up!";     ← INCOMING branch           │
│  >>>>>>> feature/greeting                                    │
│  }                                                           │
│                                                              │
│  MARKERS:                                                    │
│  <<<<<<< HEAD         = Start of YOUR version               │
│  =======              = Divider between versions            │
│  >>>>>>> branch-name  = End of THEIR version                │
│                                                              │
│  YOUR JOB: Remove markers and keep the correct code         │
└──────────────────────────────────────────────────────────────┘
```

With `merge.conflictstyle = diff3`:
```
<<<<<<< HEAD
  return "Hey there!";
||||||| merged common ancestor
  return "Hi";                     ← Original version too!
=======
  return "Yo, what's up!";
>>>>>>> feature/greeting
```

---

## Step-by-Step Conflict Resolution

### Step 1: Start the Merge

```bash
$ git switch main
$ git merge feature/greeting

# Output:
Auto-merging greeting.js
CONFLICT (content): Merge conflict in greeting.js
Automatic merge failed; fix conflicts and then commit the result.
```

### Step 2: Identify Conflicted Files

```bash
$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   greeting.js
```

### Step 3: Open and Resolve the File

Edit the file — remove ALL conflict markers and keep the correct code:

```javascript
// BEFORE (conflicted):
function greet() {
<<<<<<< HEAD
  return "Hey there!";
=======
  return "Yo, what's up!";
>>>>>>> feature/greeting
}

// AFTER (resolved — you decide what to keep):

// Option A: Keep ours
function greet() {
  return "Hey there!";
}

// Option B: Keep theirs
function greet() {
  return "Yo, what's up!";
}

// Option C: Combine both (most common!)
function greet() {
  return "Hey there! What's up!";
}

// Option D: Write something completely new
function greet(formal = false) {
  return formal ? "Good day" : "Hey there!";
}
```

### Step 4: Mark as Resolved and Commit

```bash
# Stage the resolved file
$ git add greeting.js

# Complete the merge
$ git commit
# Git auto-generates: "Merge branch 'feature/greeting'"

# Or with a custom message
$ git commit -m "Merge feature/greeting: combine greeting styles"
```

---

## Using Visual Merge Tools

```bash
# Open the configured merge tool
$ git mergetool

# Use a specific tool
$ git mergetool --tool=vscode
$ git mergetool --tool=meld
$ git mergetool --tool=kdiff3
```

### VS Code Merge UI

VS Code provides an excellent built-in merge editor:

```
┌──────────────────────────────────────────────────────────────┐
│           VS CODE MERGE EDITOR                              │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Accept Current Change | Accept Incoming Change |     │  │
│  │  Accept Both Changes | Compare Changes                │  │
│  │                                                       │  │
│  │  <<<<<<< HEAD                                         │  │
│  │    return "Hey there!";                               │  │
│  │  =======                                              │  │
│  │    return "Yo, what's up!";                           │  │
│  │  >>>>>>> feature/greeting                             │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  Click:                                                     │
│  • "Accept Current" → keeps HEAD version                    │
│  • "Accept Incoming" → keeps feature version                │
│  • "Accept Both" → keeps both (one after the other)        │
│  • "Compare" → side-by-side view                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Conflict Resolution Strategies

```
┌────────────────────────────────────────────────────────────┐
│           RESOLUTION DECISION TREE                         │
│                                                            │
│  Is the conflict in code you wrote?                        │
│  ├── YES → You probably know what the final result should  │
│  │         be. Edit accordingly.                           │
│  │                                                         │
│  └── NO → Consult the other developer                     │
│           ├── Their change is correct → Accept Incoming    │
│           ├── Your change is correct → Accept Current      │
│           ├── Both needed → Combine carefully              │
│           └── Neither is right → Rewrite together         │
│                                                            │
│  ALWAYS: Test after resolving! The merge might compile     │
│  but behave incorrectly.                                   │
└────────────────────────────────────────────────────────────┘
```

---

## Aborting a Merge

```bash
# Abort merge and go back to before the merge started
$ git merge --abort

# This resets:
# - Working directory → pre-merge state
# - Staging area → pre-merge state
# - HEAD → stays where it was
```

---

## Preventing Conflicts

```
┌──────────────────────────────────────────────────────────────┐
│             HOW TO MINIMIZE CONFLICTS                       │
│                                                              │
│  1. PULL FREQUENTLY                                         │
│     $ git pull origin main                                  │
│     └─ Stay in sync with the team                           │
│                                                              │
│  2. MERGE MAIN INTO YOUR BRANCH REGULARLY                   │
│     $ git switch feature/x                                  │
│     $ git merge main                                        │
│     └─ Resolve small conflicts early                        │
│                                                              │
│  3. KEEP BRANCHES SHORT-LIVED                               │
│     └─ Less time diverged = fewer conflicts                 │
│                                                              │
│  4. COMMUNICATE WITH YOUR TEAM                              │
│     └─ "I'm refactoring utils.js this week"                │
│                                                              │
│  5. USE SMALL, FOCUSED COMMITS                              │
│     └─ Easier to understand and resolve                     │
│                                                              │
│  6. AVOID REFORMATTING + LOGIC CHANGES IN SAME COMMIT      │
│     └─ Formatting changes cause massive diffs               │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Both Developers Edit the Same Config

```bash
# Developer A changed port to 8080
# Developer B changed port to 9090
# Conflict!

<<<<<<< HEAD
const PORT = 8080;
=======
const PORT = 9090;
>>>>>>> feature/new-port

# Resolution: Use environment variable (best of both worlds)
const PORT = process.env.PORT || 3000;
```

### Scenario 2: Conflicting Package.json

```bash
# Very common — both devs added different dependencies
<<<<<<< HEAD
  "axios": "^1.5.0",
  "lodash": "^4.17.21",
=======
  "axios": "^1.5.0",
  "moment": "^2.30.0",
>>>>>>> feature/dates

# Resolution: Keep both
  "axios": "^1.5.0",
  "lodash": "^4.17.21",
  "moment": "^2.30.0",
```

### Scenario 3: Structural Conflict (Function Moved)

```bash
# Dev A moved a function to a new file
# Dev B modified the same function in the original file
# This requires understanding BOTH changes and applying them correctly
# Sometimes you need to talk to the other developer
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Conflict markers left in file | Search for `<<<<<<<`, `=======`, `>>>>>>>` |
| Merge shows "Already up to date" | The branch is already merged into current branch |
| Too many conflicts to handle | `git merge --abort`, update your branch with main first, try again |
| Resolved but `git commit` fails | Make sure you `git add` all resolved files first |
| `.orig` files appearing | Delete them, or set `mergetool.keepBackup false` |
| Same conflict keeps appearing | Consider using `git rerere` (Reuse Recorded Resolution) |

### Enabling `git rerere` (Reuse Recorded Resolution)

```bash
# Enable rerere — Git remembers how you resolved conflicts
$ git config --global rerere.enabled true

# Next time the same conflict occurs, Git resolves it automatically
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Conflict** | Both branches modified the same lines differently |
| `<<<<<<< HEAD` | Start of your (current) branch's version |
| `=======` | Separator between the two versions |
| `>>>>>>> branch` | End of incoming branch's version |
| `git mergetool` | Open visual merge tool |
| `git merge --abort` | Cancel the merge and restore pre-merge state |
| `git add <file>` | Mark a conflict as resolved |
| `git commit` | Complete the merge after resolving all conflicts |
| `merge.conflictstyle diff3` | Show ancestor version in conflict markers too |
| `rerere.enabled true` | Git remembers and reuses your conflict resolutions |

---

## Quick Revision Questions

1. **What causes a merge conflict?**
   <details><summary>Answer</summary>A conflict occurs when both branches modify the same lines of the same file in different ways. Git can't determine which change to keep, so it requires manual resolution.</details>

2. **What are the three sections in a conflict marker and what do they represent?**
   <details><summary>Answer</summary>`<<<<<<< HEAD` (your current branch's version), `=======` (separator), `>>>>>>> branch-name` (incoming branch's version). With diff3 style, there's also a `||||||| ancestor` section showing the original.</details>

3. **What are the steps to resolve a merge conflict?**
   <details><summary>Answer</summary>1) Open conflicted files, 2) Edit to remove conflict markers and choose/combine the correct code, 3) `git add` each resolved file, 4) `git commit` to complete the merge.</details>

4. **How do you abort a merge that has conflicts you can't resolve right now?**
   <details><summary>Answer</summary>Run `git merge --abort`. This returns your working directory and staging area to the state before the merge attempt.</details>

5. **What is `git rerere` and why is it useful?**
   <details><summary>Answer</summary>`rerere` (Reuse Recorded Resolution) makes Git remember how you resolved conflicts. If the same conflict appears again (e.g., during rebasing), Git applies the same resolution automatically.</details>

6. **Name three practices that help minimize merge conflicts.**
   <details><summary>Answer</summary>1) Pull from main frequently to stay in sync, 2) Keep branches short-lived, 3) Communicate with team about what files you're modifying. Also: small focused commits and avoiding formatting changes mixed with logic changes.</details>

---

[← Previous: Merging Strategies](03-merging-strategies.md) | [Back to README](../README.md) | [Next: Rebasing vs Merging →](05-rebasing-vs-merging.md)
