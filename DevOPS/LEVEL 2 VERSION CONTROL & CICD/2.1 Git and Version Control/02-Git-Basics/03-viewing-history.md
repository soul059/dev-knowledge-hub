# Chapter 2.3: Viewing History (log, show, diff)

[← Previous: Staging and Committing](02-staging-and-committing.md) | [Back to README](../README.md) | [Next: .gitignore Configuration →](04-gitignore-configuration.md)

---

## Overview

Git's history tools let you explore the complete evolution of your project. Mastering `git log`, `git show`, and `git diff` is essential for understanding what changed, when, why, and by whom.

---

## git log — View Commit History

### Basic Usage

```bash
# Full log (most recent first)
$ git log

# Output:
commit a1b2c3d4e5f6789012345678901234567890abcd (HEAD -> main)
Author: Alice <alice@example.com>
Date:   Thu Feb 27 10:30:00 2026 +0000

    Add user authentication

commit fedcba9876543210fedcba9876543210fedcba98
Author: Bob <bob@example.com>
Date:   Wed Feb 26 15:00:00 2026 +0000

    Fix database connection timeout
```

### Compact Formats

```bash
# One-line format
$ git log --oneline
a1b2c3d Add user authentication
fedcba9 Fix database connection timeout
1234567 Initial commit

# Short format
$ git log --format=short

# Custom format
$ git log --format="%h %an %ar %s"
a1b2c3d Alice 2 hours ago Add user authentication
fedcba9 Bob 1 day ago Fix database connection timeout
```

### Format Placeholders

```
┌─────────────────────────────────────────────────────────┐
│           COMMON FORMAT PLACEHOLDERS                    │
│                                                         │
│  %H  = Full commit hash                                │
│  %h  = Abbreviated commit hash                         │
│  %T  = Tree hash                                       │
│  %an = Author name                                     │
│  %ae = Author email                                    │
│  %ar = Author date (relative)                          │
│  %ad = Author date (format respects --date=)           │
│  %cn = Committer name                                  │
│  %s  = Subject (first line of message)                 │
│  %b  = Body (rest of message)                          │
│  %d  = Ref names (branches, tags)                      │
│  %n  = Newline                                         │
│                                                         │
│  Example:                                               │
│  git log --format="%h | %an | %ar | %s"                │
│  a1b2c3d | Alice | 2 hours ago | Add authentication    │
└─────────────────────────────────────────────────────────┘
```

### Filtering Log Output

```bash
# Limit number of commits
$ git log -5                          # Last 5 commits
$ git log -n 10                       # Last 10 commits

# Filter by date
$ git log --since="2026-01-01"
$ git log --until="2026-02-01"
$ git log --since="2 weeks ago"
$ git log --after="yesterday"

# Filter by author
$ git log --author="Alice"
$ git log --author="alice@example.com"

# Filter by commit message
$ git log --grep="fix"                # Messages containing "fix"
$ git log --grep="bug" -i             # Case-insensitive

# Filter by file
$ git log -- path/to/file.js         # Commits affecting this file
$ git log -- "*.css"                  # Commits affecting CSS files

# Filter by content change (pickaxe)
$ git log -S "functionName"          # Commits that add/remove this string
$ git log -G "regex_pattern"         # Regex version
```

### Visual Graph

```bash
# Show branch graph
$ git log --graph --oneline --all

# Example output:
* a1b2c3d (HEAD -> main) Merge feature/auth
|\
| * fedcba9 (feature/auth) Add login page
| * 1234567 Add auth middleware
|/
* 9876543 Update dependencies
* abcdef0 Initial commit

# Decorated with branch and tag names
$ git log --graph --oneline --all --decorate
```

```
┌────────────────────────────────────────────────────────────┐
│           VISUAL LOG WITH --graph                          │
│                                                            │
│  * a1b2c3d (HEAD -> main) Merge feature/auth              │
│  |\                                                        │
│  | * fedcba9 Add login page                                │
│  | * 1234567 Add auth middleware                           │
│  |/                                                        │
│  * 9876543 Update dependencies                             │
│  |                                                         │
│  | * 3456789 (feature/payments) WIP: payment form          │
│  |/                                                        │
│  * abcdef0 Initial commit                                  │
│                                                            │
│  * = commit   | = branch line   / \ = merge/diverge       │
└────────────────────────────────────────────────────────────┘
```

---

## git show — Inspect a Specific Commit

```bash
# Show the latest commit with its diff
$ git show

# Show a specific commit
$ git show a1b2c3d

# Show only the commit message (no diff)
$ git show --stat a1b2c3d

# Show a file at a specific commit
$ git show a1b2c3d:path/to/file.js

# Show a tag
$ git show v2.0.0

# Show just file names that changed
$ git show --name-only a1b2c3d

# Show file names with status (added/modified/deleted)
$ git show --name-status a1b2c3d
```

### Example Output of `git show`

```bash
$ git show a1b2c3d

commit a1b2c3d4e5f6789012345678901234567890abcd
Author: Alice <alice@example.com>
Date:   Thu Feb 27 10:30:00 2026 +0000

    Add user authentication

diff --git a/auth.js b/auth.js
new file mode 100644
index 0000000..abc1234
--- /dev/null
+++ b/auth.js
@@ -0,0 +1,15 @@
+const express = require('express');
+const router = express.Router();
+
+router.post('/login', (req, res) => {
+  // login logic
+});
```

---

## git diff — Compare Changes

### The Diff Flow

```
┌──────────────────────────────────────────────────────────┐
│                 git diff VARIANTS                        │
│                                                          │
│  Working Dir         Staging Area        Last Commit     │
│       │                   │                   │          │
│       │◄── git diff ─────▶│                   │          │
│       │  (unstaged changes)│                   │          │
│       │                   │                   │          │
│       │                   │◄─ git diff ──────▶│          │
│       │                   │   --staged        │          │
│       │                   │  (staged changes) │          │
│       │                   │                   │          │
│       │◄───────── git diff HEAD ─────────────▶│          │
│       │          (all changes vs last commit) │          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Common Diff Commands

```bash
# Compare working directory vs staging area
$ git diff

# Compare staging area vs last commit
$ git diff --staged
# (or equivalently)
$ git diff --cached

# Compare working directory vs last commit
$ git diff HEAD

# Compare two commits
$ git diff abc1234 def5678

# Compare two branches
$ git diff main feature/auth

# Diff a specific file
$ git diff -- path/to/file.js

# Diff with stats only (no actual diff content)
$ git diff --stat

# Show word-level diff (not line-level)
$ git diff --word-diff

# Show only file names that differ
$ git diff --name-only main feature/auth

# Show names with change status
$ git diff --name-status main feature/auth
```

### Reading a Diff Output

```
┌──────────────────────────────────────────────────────────────┐
│                    READING A DIFF                            │
│                                                              │
│  diff --git a/app.js b/app.js                               │
│  index abc1234..def5678 100644        ← file mode            │
│  --- a/app.js                         ← old version (a/)     │
│  +++ b/app.js                         ← new version (b/)     │
│  @@ -10,7 +10,9 @@ function init() {  ← hunk header          │
│                                        line 10, 7 lines old  │
│                                        line 10, 9 lines new  │
│                                                              │
│   // unchanged context line                                  │
│   const app = express();               ← context (no prefix) │
│  -app.listen(3000);                    ← removed (OLD)       │
│  +const PORT = process.env.PORT || 3000;  ← added (NEW)     │
│  +app.listen(PORT, () => {             ← added               │
│  +  console.log(`Server on ${PORT}`);  ← added               │
│  +});                                  ← added               │
│   // more context                                            │
│                                                              │
│  COLOR CODING:                                               │
│    Red   (-)  = Removed lines                                │
│    Green (+)  = Added lines                                  │
│    White      = Context (unchanged)                          │
└──────────────────────────────────────────────────────────────┘
```

---

## git blame — Who Changed What

```bash
# See line-by-line attribution
$ git blame path/to/file.js

# Output:
a1b2c3d4 (Alice  2026-02-27 10:30  1) const express = require('express');
a1b2c3d4 (Alice  2026-02-27 10:30  2) const router = express.Router();
fedcba98 (Bob    2026-02-26 15:00  3) 
fedcba98 (Bob    2026-02-26 15:00  4) router.post('/login', async (req, res) => {
12345678 (Alice  2026-02-25 09:00  5)   const { email, password } = req.body;

# Blame a specific line range
$ git blame -L 10,20 file.js

# Show email instead of name
$ git blame -e file.js

# Ignore whitespace changes
$ git blame -w file.js
```

---

## git shortlog — Summarize by Author

```bash
# Summary grouped by author
$ git shortlog
Alice (5):
      Add user authentication
      Fix login timeout
      Update README
      Add password reset
      Refactor auth module

Bob (3):
      Fix database connection
      Add unit tests
      Update dependencies

# Count only
$ git shortlog -s -n
     5  Alice
     3  Bob
```

---

## Useful Aliases for History Exploration

```bash
# Set up useful aliases
$ git config --global alias.lg "log --graph --oneline --all --decorate"
$ git config --global alias.last "log -1 HEAD --stat"
$ git config --global alias.changes "diff --stat"
$ git config --global alias.today "log --since='midnight' --oneline"

# Usage
$ git lg          # Beautiful graph view
$ git last        # Last commit with file stats
$ git changes     # Quick overview of current changes
$ git today       # What you committed today
```

---

## Real-World Scenarios

### Scenario 1: Finding When a Bug Was Introduced

```bash
# Search for when a specific function was added/removed
$ git log -S "calculateTotal" --oneline
fedcba9 Refactor billing module
1234567 Add shopping cart

# See the actual change
$ git show fedcba9

# See who last modified specific lines
$ git blame src/billing.js -L 45,55
```

### Scenario 2: What Changed Since Last Release

```bash
# See all commits since v1.0.0
$ git log v1.0.0..HEAD --oneline

# See the actual diff since v1.0.0
$ git diff v1.0.0 HEAD --stat

# See commits by author since the release
$ git shortlog v1.0.0..HEAD
```

### Scenario 3: Reviewing a Teammate's Work

```bash
# See what changed on their branch vs main
$ git log main..feature/payments --oneline
$ git diff main..feature/payments --stat
$ git diff main..feature/payments -- src/payments/
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Log is too long | Use `-n 10` or `--since` to limit output |
| Can't find a specific commit | Use `git log --all --grep="keyword"` |
| Diff output is confusing | Use `--word-diff` or `--stat` for clarity |
| Blame shows a bulk reformatting commit | Use `git blame -w` to ignore whitespace |
| Need to find a deleted file | `git log --all --diff-filter=D -- "**filename**"` |
| Want to see a file's entire history | `git log --follow -- path/to/file.js` (handles renames) |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git log` | View commit history |
| `git log --oneline` | Compact one-line-per-commit view |
| `git log --graph` | Show branch/merge graph |
| `git log --author="name"` | Filter by author |
| `git log --grep="text"` | Filter by commit message |
| `git log -S "code"` | Find commits that add/remove specific code |
| `git log -- file` | History of a specific file |
| `git show <hash>` | Show commit details + diff |
| `git diff` | Compare working dir vs staging area |
| `git diff --staged` | Compare staging area vs last commit |
| `git diff HEAD` | Compare working dir vs last commit |
| `git diff A..B` | Compare two branches or commits |
| `git blame <file>` | Line-by-line authorship |
| `git shortlog -s -n` | Commit count by author |

---

## Quick Revision Questions

1. **What is the difference between `git diff`, `git diff --staged`, and `git diff HEAD`?**
   <details><summary>Answer</summary>`git diff` compares working directory vs staging area. `git diff --staged` compares staging area vs last commit. `git diff HEAD` compares working directory vs last commit (all changes).</details>

2. **How do you find all commits that modified a specific file?**
   <details><summary>Answer</summary>`git log -- path/to/file.js`. Add `--follow` to track the file even through renames.</details>

3. **What does `git log -S "functionName"` do?**
   <details><summary>Answer</summary>It's the "pickaxe" search — finds commits where the number of occurrences of "functionName" changed (i.e., it was added or removed), helping trace when specific code was introduced.</details>

4. **How do you see a visual graph of branches and merges in the terminal?**
   <details><summary>Answer</summary>`git log --graph --oneline --all --decorate` shows a text-based graph of the commit history across all branches.</details>

5. **What information does `git blame` provide and when is it useful?**
   <details><summary>Answer</summary>`git blame` shows who last modified each line of a file, with the commit hash and date. It's useful for understanding code authorship and finding when/why a specific line was changed.</details>

6. **How would you see what changes were made between two releases (e.g., v1.0 and v2.0)?**
   <details><summary>Answer</summary>`git log v1.0..v2.0 --oneline` for a list of commits, `git diff v1.0 v2.0 --stat` for a summary of file changes, or `git diff v1.0 v2.0` for the full diff.</details>

---

[← Previous: Staging and Committing](02-staging-and-committing.md) | [Back to README](../README.md) | [Next: .gitignore Configuration →](04-gitignore-configuration.md)
