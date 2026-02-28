# Chapter 2.2: Staging and Committing

[← Previous: Repository Initialization](01-repository-initialization.md) | [Back to README](../README.md) | [Next: Viewing History →](03-viewing-history.md)

---

## Overview

The staging area is one of Git's most powerful and unique features. It gives you fine-grained control over exactly what goes into each commit, enabling clean, logical, well-structured commit histories.

---

## The File Lifecycle in Git

```
┌──────────────────────────────────────────────────────────────┐
│                 FILE LIFECYCLE IN GIT                        │
│                                                              │
│   ┌───────────┐    ┌───────────┐    ┌──────────┐    ┌─────┐ │
│   │ Untracked │    │Unmodified │    │ Modified │    │Staged│ │
│   │           │    │ (Clean)   │    │          │    │      │ │
│   └─────┬─────┘    └─────┬─────┘    └────┬─────┘    └──┬───┘ │
│         │                │               │             │     │
│         │── git add ─────────────────────────────────▶│     │
│         │                │               │             │     │
│         │                │── edit file ──▶│             │     │
│         │                │               │             │     │
│         │                │               │── git add ─▶│     │
│         │                │               │             │     │
│         │                │◀──────────── git commit ────│     │
│         │                │               │             │     │
│         │◀── git rm ─────│               │             │     │
│         │                │               │             │     │
│   ┌─────▼─────┐    ┌─────▼─────┐    ┌────▼─────┐    ┌──▼───┐ │
│   │ Untracked │    │Unmodified │    │ Modified │    │Staged│ │
│   └───────────┘    └───────────┘    └──────────┘    └──────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## The Staging Area Explained

```
┌────────────────────────────────────────────────────────────┐
│              WHY A STAGING AREA?                           │
│                                                            │
│  Without staging (like SVN):                               │
│    edit → commit ALL changes at once                       │
│    ❌ Can't select which changes to commit                  │
│                                                            │
│  With staging (Git):                                       │
│    edit → selectively stage → commit chosen changes        │
│    ✅ Precise, logical commits                              │
│                                                            │
│  ANALOGY: Packing a suitcase                               │
│                                                            │
│   Closet          Suitcase        Checked Bag              │
│  (Working Dir)   (Staging Area)   (Repository)             │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │ Shirt    │    │ Shirt    │    │          │              │
│  │ Pants    │──▶ │ Pants    │──▶ │ Packed!  │              │
│  │ Jacket   │    │          │    │          │              │
│  │ Shoes    │    │ (Jacket  │    │          │              │
│  │ Hat      │    │  stays   │    │          │              │
│  └──────────┘    │  behind) │    └──────────┘              │
│                  └──────────┘                              │
│                                                            │
│  You CHOOSE what to pack (stage) before shipping (commit) │
└────────────────────────────────────────────────────────────┘
```

---

## Staging Commands

### `git add` — Stage Changes

```bash
# Stage a specific file
$ git add filename.txt

# Stage multiple specific files
$ git add file1.txt file2.txt file3.txt

# Stage all changes in current directory and subdirectories
$ git add .

# Stage all changes in the entire repo
$ git add -A
# or
$ git add --all

# Stage only modified and deleted (not new) files
$ git add -u

# Stage with wildcard patterns
$ git add *.js
$ git add src/*.py
$ git add docs/

# Interactive staging — choose hunks within a file
$ git add -p filename.txt
# or for all files:
$ git add -p
```

### Interactive Staging with `git add -p`

```bash
$ git add -p app.js

# Git shows each change hunk and asks:
# Stage this hunk [y,n,q,a,d,s,e,?]?
#
#  y = yes, stage this hunk
#  n = no, skip this hunk
#  q = quit (don't stage remaining hunks)
#  a = stage this and all remaining hunks
#  d = don't stage this or any remaining hunks
#  s = split into smaller hunks
#  e = manually edit the hunk
```

This is incredibly powerful for splitting unrelated changes into separate commits.

---

## Committing

### `git commit` — Create a Snapshot

```bash
# Commit staged changes with a message
$ git commit -m "Add user login functionality"

# Commit with a multi-line message
$ git commit -m "Add user login

- Implement session management
- Add CSRF protection
- Create login/logout endpoints"

# Open editor for commit message
$ git commit

# Stage all tracked file changes AND commit (skip git add)
$ git commit -a -m "Fix typo in header"
# or
$ git commit -am "Fix typo in header"

# Amend the last commit (change message or add files)
$ git add forgotten-file.txt
$ git commit --amend -m "Updated commit message"

# Amend without changing the message
$ git commit --amend --no-edit
```

### Anatomy of a Good Commit

```
┌──────────────────────────────────────────────────────┐
│              ANATOMY OF A COMMIT                     │
│                                                      │
│  $ git commit -m "Add user authentication"           │
│                                                      │
│  Creates:                                            │
│  ┌────────────────────────────────────────────────┐  │
│  │  Commit Object                                 │  │
│  │                                                │  │
│  │  Hash:      a1b2c3d4e5f6...                   │  │
│  │  Tree:      → snapshot of all files           │  │
│  │  Parent:    → previous commit hash            │  │
│  │  Author:    Alice <alice@ex.com> timestamp    │  │
│  │  Committer: Alice <alice@ex.com> timestamp    │  │
│  │  Message:   Add user authentication           │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  The commit is PERMANENT and IMMUTABLE               │
│  (unless you use rebase/amend)                       │
└──────────────────────────────────────────────────────┘
```

---

## Checking Status

### `git status` — See What's Going On

```bash
$ git status

# Example output:
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   api/routes.js
        modified:   app.js

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   config.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-feature.js
```

```bash
# Short status (compact view)
$ git status -s
A  api/routes.js      # A = Added (staged)
M  app.js             # M = Modified (staged)
 M config.js          # M = Modified (not staged) — note the space
?? new-feature.js     # ?? = Untracked
```

### Status Flag Reference

```
┌──────────────────────────────────────────────────────────┐
│           git status -s FLAGS                            │
│                                                          │
│  Column 1: Staging area status                           │
│  Column 2: Working directory status                      │
│                                                          │
│  FLAG │ MEANING                                          │
│  ─────┼──────────────────────────────────                │
│   M   │ Modified                                         │
│   A   │ Added (new file)                                 │
│   D   │ Deleted                                          │
│   R   │ Renamed                                          │
│   C   │ Copied                                           │
│   ??  │ Untracked                                        │
│   !!  │ Ignored                                          │
│                                                          │
│  Examples:                                               │
│   M  file.txt  → Staged modification                    │
│    M file.txt  → Unstaged modification                  │
│   MM file.txt  → Staged + additional unstaged changes   │
│   A  file.txt  → New file, staged                       │
│   D  file.txt  → Deleted, staged                        │
│   ?? file.txt  → Untracked (new, never added)           │
└──────────────────────────────────────────────────────────┘
```

---

## Unstaging and Undoing

```bash
# Unstage a file (keep changes in working directory)
$ git restore --staged filename.txt
# or (older syntax):
$ git reset HEAD filename.txt

# Discard changes in working directory (DANGEROUS — permanent!)
$ git restore filename.txt
# or (older syntax):
$ git checkout -- filename.txt

# Unstage everything
$ git restore --staged .
# or
$ git reset HEAD

# Discard ALL working directory changes (DANGEROUS!)
$ git restore .
```

```
┌────────────────────────────────────────────────────────────┐
│            UNDO OPERATIONS FLOW                            │
│                                                            │
│  Working Dir         Staging Area        Repository        │
│       │                   │                   │            │
│       │◄── git restore ───│                   │            │
│       │    (discard edits) │                   │            │
│       │                   │                   │            │
│       │    git restore ──▶│                   │            │
│       │    --staged        │                   │            │
│       │   (unstage file)   │                   │            │
│       │                   │                   │            │
│       │◄──────── git reset HEAD~1 ────────────│            │
│       │          (undo last commit,            │            │
│       │           keep changes)                │            │
│       │                   │                   │            │
│       │   git reset ─────────────────────────▶│            │
│       │   --hard HEAD~1                       │            │
│       │   (undo commit + discard changes)     │            │
│       │   ⚠️ DANGEROUS!                        │            │
└────────────────────────────────────────────────────────────┘
```

---

## Removing and Renaming Files

```bash
# Remove a file from Git AND working directory
$ git rm filename.txt

# Remove from Git but keep in working directory
$ git rm --cached filename.txt
# Useful for: accidentally committed files (like .env)

# Rename a file
$ git mv old-name.txt new-name.txt
# Equivalent to:
# $ mv old-name.txt new-name.txt
# $ git rm old-name.txt
# $ git add new-name.txt
```

---

## Real-World Scenarios

### Scenario 1: Splitting Changes into Logical Commits

```bash
# You've been working and have changes in 3 files:
# - auth.js (new login feature)
# - header.css (styling fix)
# - README.md (documentation update)

# DON'T do this:
$ git add .
$ git commit -m "Various changes"  # ❌ Vague, mixes concerns

# DO this:
$ git add auth.js
$ git commit -m "feat: add user authentication"

$ git add header.css
$ git commit -m "fix: correct header alignment on mobile"

$ git add README.md
$ git commit -m "docs: update API documentation"
```

### Scenario 2: Partially Staging a File

```bash
# You fixed a bug AND added a feature in the same file
# Stage only the bug fix using interactive mode

$ git add -p app.js

# Git shows hunk 1 (bug fix):
# Stage this hunk? y

# Git shows hunk 2 (new feature):
# Stage this hunk? n

$ git commit -m "fix: resolve null pointer in user lookup"

# Now stage the feature
$ git add -p app.js
# Stage remaining hunk? y
$ git commit -m "feat: add user search functionality"
```

### Scenario 3: Accidentally Staged a File

```bash
# Oops, staged the .env file
$ git add .
$ git status
# Changes to be committed:
#   new file: .env         ← Sensitive! Don't commit!
#   new file: app.js

# Unstage just .env
$ git restore --staged .env

# Also add it to .gitignore
$ echo ".env" >> .gitignore
$ git add .gitignore
$ git commit -m "Add app.js and .gitignore"
```

---

## Best Practices

```
┌────────────────────────────────────────────────────────────┐
│            STAGING & COMMITTING BEST PRACTICES             │
│                                                            │
│  1. COMMIT OFTEN                                           │
│     └─ Small, frequent commits > rare, massive commits     │
│                                                            │
│  2. COMMIT LOGICALLY                                       │
│     └─ Each commit = one logical change                    │
│     └─ Don't mix bug fixes with features                   │
│                                                            │
│  3. WRITE GOOD MESSAGES                                    │
│     └─ Imperative mood: "Add feature" not "Added feature"  │
│     └─ Be specific: "Fix login timeout" > "Fix bug"       │
│                                                            │
│  4. REVIEW BEFORE COMMITTING                               │
│     └─ Always check `git status` and `git diff --staged`  │
│                                                            │
│  5. DON'T COMMIT GENERATED FILES                           │
│     └─ node_modules, build/, dist/ → .gitignore           │
│                                                            │
│  6. DON'T COMMIT SECRETS                                   │
│     └─ .env, API keys, passwords → .gitignore             │
│                                                            │
│  7. USE -p FOR PRECISION                                   │
│     └─ `git add -p` to stage specific parts of files      │
└────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Committed to wrong branch | `git reset HEAD~1` (undo commit), switch branch, recommit |
| Typo in commit message | `git commit --amend -m "corrected message"` |
| Forgot to add a file to last commit | `git add file && git commit --amend --no-edit` |
| Accidentally staged sensitive file | `git restore --staged <file>` then add to `.gitignore` |
| `git add .` added too much | `git restore --staged .` then selectively `git add` |
| Want to see what's staged | `git diff --staged` (or `git diff --cached`) |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git add <file>` | Stage a specific file |
| `git add .` | Stage all changes in current directory |
| `git add -A` | Stage all changes in entire repo |
| `git add -p` | Interactive staging (by hunk) |
| `git add -u` | Stage modified/deleted tracked files only |
| `git commit -m "msg"` | Commit staged changes with a message |
| `git commit -am "msg"` | Stage tracked files + commit in one step |
| `git commit --amend` | Modify the last commit |
| `git status` | Show working directory and staging area status |
| `git status -s` | Compact status output |
| `git restore --staged <file>` | Unstage a file |
| `git restore <file>` | Discard working directory changes |
| `git rm <file>` | Remove file from Git and disk |
| `git rm --cached <file>` | Remove from Git, keep on disk |
| `git mv <old> <new>` | Rename/move a file |

---

## Quick Revision Questions

1. **What is the purpose of the staging area in Git?**
   <details><summary>Answer</summary>The staging area lets you selectively choose which changes to include in the next commit, enabling precise, logical commits rather than committing all changes at once.</details>

2. **What is the difference between `git add .` and `git add -A`?**
   <details><summary>Answer</summary>`git add .` stages changes in the current directory and subdirectories. `git add -A` stages all changes in the entire repository regardless of current directory.</details>

3. **How do you stage only part of a file's changes?**
   <details><summary>Answer</summary>Use `git add -p <file>` (patch mode). Git will show you each hunk of changes and let you choose which to stage (y/n/s/e options).</details>

4. **What does `git commit -am "message"` do and what's its limitation?**
   <details><summary>Answer</summary>It stages all modifications to tracked files and commits them. The limitation is it does NOT stage new (untracked) files — only files Git already knows about.</details>

5. **How do you undo staging a file without losing your changes?**
   <details><summary>Answer</summary>Use `git restore --staged <file>`. This removes the file from the staging area but keeps your changes in the working directory.</details>

6. **What's the difference between `git rm <file>` and `git rm --cached <file>`?**
   <details><summary>Answer</summary>`git rm` removes the file from both Git tracking AND the filesystem. `git rm --cached` removes it only from Git tracking, keeping the file on your disk — useful for accidentally tracked files.</details>

---

[← Previous: Repository Initialization](01-repository-initialization.md) | [Back to README](../README.md) | [Next: Viewing History →](03-viewing-history.md)
