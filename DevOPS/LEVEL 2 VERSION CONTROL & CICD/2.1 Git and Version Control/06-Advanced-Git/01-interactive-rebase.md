# Chapter 6.1: Interactive Rebase

[← Previous: Choosing the Right Workflow](../05-Git-Workflows/05-choosing-the-right-workflow.md) | [Back to README](../README.md) | [Next: Stashing →](02-stashing.md)

---

## Overview

**Interactive rebase** (`git rebase -i`) gives you full control over your commit history. You can reorder, edit, squash, split, or drop commits before sharing them. It's one of Git's most powerful features for crafting clean, meaningful commit histories.

---

## How Interactive Rebase Works

```
┌──────────────────────────────────────────────────────────────┐
│       INTERACTIVE REBASE FLOW                               │
│                                                              │
│  BEFORE: messy work-in-progress commits                     │
│                                                              │
│  ●──●──●──●──●──●                                           │
│     │  │  │  │  │                                           │
│     C1 C2 C3 C4 C5                                          │
│                                                              │
│  C2: "WIP"                                                  │
│  C3: "fix typo"                                             │
│  C4: "WIP again"                                            │
│  C5: "actually done now"                                    │
│                                                              │
│  $ git rebase -i HEAD~4                                     │
│                                                              │
│  AFTER: clean, meaningful commits                           │
│                                                              │
│  ●──●──●                                                    │
│     │  │                                                    │
│     C1 C2' (squashed C2-C5 into one clean commit)           │
│                                                              │
│  C2': "Add user authentication feature"                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Starting Interactive Rebase

```bash
# Rebase last N commits
$ git rebase -i HEAD~4

# Rebase onto a specific commit (all commits after it)
$ git rebase -i abc1234

# Rebase onto a branch
$ git rebase -i main
```

When you run this, Git opens your editor with a **todo list**:

```
pick abc1234 Add login form
pick def5678 WIP - fixing stuff
pick ghi9012 Fix typo in login
pick jkl3456 Final login implementation

# Rebase abc1234..jkl3456 onto xyz0000 (4 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup  = like "squash", but discard this commit's log message
# d, drop   = remove commit
# x, exec   = run command
# b, break  = stop here (continue later with 'git rebase --continue')
```

---

## The Commands

```
┌──────────────────────────────────────────────────────────────┐
│       INTERACTIVE REBASE COMMANDS                           │
│                                                              │
│  pick (p)    Keep the commit as-is                          │
│  reword (r)  Keep commit, change its message                │
│  edit (e)    Pause at commit for amending                   │
│  squash (s)  Merge into previous, combine messages          │
│  fixup (f)   Merge into previous, discard this message      │
│  drop (d)    Delete the commit entirely                     │
│  exec (x)    Run a shell command at this point              │
│  break (b)   Pause here — continue with --continue          │
│                                                              │
│  You can also REORDER lines to reorder commits!             │
└──────────────────────────────────────────────────────────────┘
```

---

## Common Operations

### 1. Squash Multiple Commits

```bash
$ git rebase -i HEAD~4
```

Editor shows:
```
pick abc1234 Add login form
squash def5678 WIP - fixing stuff
squash ghi9012 Fix typo in login
squash jkl3456 Final login implementation
```

Result: All four commits become ONE commit. Git opens editor to let you write a combined message.

```
┌──────────────────────────────────────────────────────────────┐
│  SQUASH — BEFORE AND AFTER                                  │
│                                                              │
│  BEFORE:   ●──●──●──●                                       │
│            C1 C2 C3 C4                                      │
│                                                              │
│  AFTER:    ●                                                │
│           C1' (all changes combined)                        │
│                                                              │
│  Use squash when you have WIP/fix commits that              │
│  should be one logical change                               │
└──────────────────────────────────────────────────────────────┘
```

### 2. Reword a Commit Message

```bash
$ git rebase -i HEAD~3
```

```
pick abc1234 Add login form
reword def5678 fixd teh bug      ← Change to "reword"
pick ghi9012 Add tests
```

Git pauses at that commit and opens editor for you to fix the message.

### 3. Reorder Commits

Simply change the line order:

```
pick ghi9012 Add tests           ← moved up
pick abc1234 Add login form      ← moved down
pick def5678 Fix login bug
```

### 4. Drop (Delete) a Commit

```
pick abc1234 Add login form
drop def5678 Debug logging        ← This commit is removed
pick ghi9012 Add tests
```

### 5. Edit a Commit (Amend Mid-History)

```
pick abc1234 Add login form
edit def5678 Add validation       ← Git pauses here
pick ghi9012 Add tests
```

When Git pauses:
```bash
# Make changes to files
$ code src/validation.js
$ git add src/validation.js
$ git commit --amend              # Amend THIS commit
$ git rebase --continue           # Continue with remaining commits
```

### 6. Fixup (Squash Without Message)

```
pick abc1234 Add login form
fixup def5678 Fix typo            ← Changes merged, message discarded
pick ghi9012 Add tests
```

---

## Autosquash with Fixup Commits

```bash
# Create a fixup commit that will automatically squash
$ git commit --fixup=abc1234
# Creates: "fixup! Add login form"

# Later, run rebase with autosquash
$ git rebase -i --autosquash main
# The fixup commit is automatically placed after its target
# and marked as "fixup"
```

```
┌──────────────────────────────────────────────────────────────┐
│  AUTOSQUASH WORKFLOW                                        │
│                                                              │
│  1. $ git commit --fixup=abc1234                            │
│     Creates: "fixup! Add login form"                        │
│                                                              │
│  2. $ git rebase -i --autosquash main                       │
│     Editor shows:                                           │
│     pick abc1234 Add login form                             │
│     fixup xyz9999 fixup! Add login form  ← auto-placed!    │
│     pick def5678 Add tests                                  │
│                                                              │
│  3. Save and close → fixup is automatically squashed        │
│                                                              │
│  Tip: Set as default:                                       │
│  $ git config --global rebase.autoSquash true               │
└──────────────────────────────────────────────────────────────┘
```

---

## Safety and Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│  INTERACTIVE REBASE SAFETY RULES                            │
│                                                              │
│  ⚠️  NEVER rebase commits that are already pushed to a      │
│     shared branch (same rule as regular rebase)             │
│                                                              │
│  ✅ SAFE: Rebase your local unpushed commits                │
│  ✅ SAFE: Rebase your own feature branch (force-push-with-  │
│           lease if already pushed)                           │
│                                                              │
│  ❌ UNSAFE: Rebasing main or develop                        │
│  ❌ UNSAFE: Rebasing commits others have pulled             │
│                                                              │
│  RECOVERY:                                                  │
│  If something goes wrong:                                   │
│  $ git rebase --abort     # Cancel in-progress rebase       │
│  $ git reflog             # Find pre-rebase state           │
│  $ git reset --hard HEAD@{n}  # Restore                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Clean Up Before PR

```bash
# You have 7 messy commits; clean up to 2 meaningful ones
$ git rebase -i main

# In editor:
pick abc1234 Add user registration form
squash def5678 WIP registration
squash ghi9012 Fix form validation
squash jkl3456 More WIP
pick mno7890 Add registration tests
squash pqr1234 Fix test setup
squash stu5678 Add missing assertions

# Result: 2 clean commits
# 1. "Add user registration form"
# 2. "Add registration tests"

$ git push --force-with-lease origin feature/registration
```

### Scenario 2: Fix a Typo in Old Commit Message

```bash
$ git rebase -i HEAD~5
# Change "pick" to "reword" on the commit with the typo
# Save → edit the message → done
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Made a mistake during rebase | `git rebase --abort` to cancel |
| Rebase completed but result is wrong | `git reflog` + `git reset --hard <hash>` |
| Conflicts on every commit during rebase | Consider merging instead, or resolve one at a time |
| Editor closes without saving | Set your preferred editor: `git config --global core.editor "code --wait"` |
| "Cannot rebase: unstaged changes" | Commit or stash changes first |

---

## Summary Table

| Command | What It Does |
|---------|-------------|
| `pick` | Keep commit as-is |
| `reword` | Keep commit, edit message |
| `edit` | Pause for amending |
| `squash` | Merge into previous (combine messages) |
| `fixup` | Merge into previous (discard message) |
| `drop` | Remove commit entirely |
| `git rebase -i HEAD~N` | Rebase last N commits |
| `git rebase --abort` | Cancel rebase |
| `git rebase --continue` | Continue after resolving |
| `git commit --fixup=<hash>` | Create auto-squash commit |

---

## Quick Revision Questions

1. **What does `git rebase -i` allow you to do?**
   <details><summary>Answer</summary>Interactive rebase lets you reorder, edit, squash, split, drop, or reword commits in your history. It opens an editor showing a list of commits that you can manipulate with commands like pick, squash, reword, edit, fixup, and drop.</details>

2. **What is the difference between squash and fixup?**
   <details><summary>Answer</summary>Both merge a commit into the previous one. `squash` combines the commit messages (opens editor to edit the combined message). `fixup` discards the commit's message and keeps only the previous commit's message.</details>

3. **How do you clean up 5 WIP commits into one meaningful commit?**
   <details><summary>Answer</summary>Run `git rebase -i HEAD~5`, keep the first commit as `pick`, change all others to `squash` or `fixup`. Save and close the editor. Git combines all changes into one commit. Edit the final message if using squash.</details>

4. **What is the `--autosquash` flag?**
   <details><summary>Answer</summary>When used with `git rebase -i --autosquash`, it automatically positions `fixup!` and `squash!` commits (created with `git commit --fixup=<hash>`) next to their target commits and marks them appropriately.</details>

5. **How do you recover from a bad interactive rebase?**
   <details><summary>Answer</summary>If rebase is still in progress: `git rebase --abort`. If it's already completed: use `git reflog` to find the commit hash from before the rebase, then `git reset --hard <hash>` to restore the original state.</details>

---

[← Previous: Choosing the Right Workflow](../05-Git-Workflows/05-choosing-the-right-workflow.md) | [Back to README](../README.md) | [Next: Stashing →](02-stashing.md)
