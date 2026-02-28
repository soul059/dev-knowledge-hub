# Chapter 2.1: Repository Initialization

[← Previous: Git Objects](../01-Version-Control-Concepts/04-git-objects.md) | [Back to README](../README.md) | [Next: Staging and Committing →](02-staging-and-committing.md)

---

## Overview

Every Git project starts with initializing a repository. This chapter covers how to create new repositories, what happens behind the scenes when you run `git init`, and best practices for setting up projects.

---

## Creating a New Repository

### Method 1: `git init` — Start from Scratch

```bash
# Create a new directory and initialize Git
$ mkdir my-project
$ cd my-project
$ git init

# Output:
Initialized empty Git repository in /path/to/my-project/.git/

# Or do it in one step
$ git init my-project
```

### Method 2: `git clone` — Copy an Existing Repository

```bash
# Clone via HTTPS
$ git clone https://github.com/user/repo.git

# Clone via SSH
$ git clone git@github.com:user/repo.git

# Clone into a specific directory
$ git clone https://github.com/user/repo.git my-local-name
```

```
┌──────────────────────────────────────────────────────────┐
│           git init vs git clone                          │
│                                                          │
│  git init                    git clone                   │
│  ┌──────────────┐            ┌──────────────┐            │
│  │ Empty .git/  │            │  Full .git/  │            │
│  │ No commits   │            │  All commits │            │
│  │ No remote    │            │  Remote set  │            │
│  │ No files     │            │  All files   │            │
│  └──────────────┘            └──────────────┘            │
│                                                          │
│  Use for NEW projects        Use for EXISTING projects   │
└──────────────────────────────────────────────────────────┘
```

---

## What `git init` Creates

```
┌──────────────────────────────────────────────────────────┐
│           AFTER "git init"                               │
│                                                          │
│  my-project/                                             │
│  ├── (your project files go here)                        │
│  └── .git/                    ◄── Hidden directory       │
│      ├── HEAD                 ◄── ref: refs/heads/main   │
│      ├── config               ◄── Repo configuration     │
│      ├── description          ◄── For GitWeb             │
│      ├── hooks/               ◄── Script templates       │
│      │   ├── pre-commit.sample                           │
│      │   ├── pre-push.sample                             │
│      │   ├── commit-msg.sample                           │
│      │   └── ... (more samples)                          │
│      ├── info/                                           │
│      │   └── exclude          ◄── Local gitignore        │
│      ├── objects/             ◄── Object database        │
│      │   ├── info/                                       │
│      │   └── pack/                                       │
│      └── refs/                ◄── Branch/tag pointers    │
│          ├── heads/                                      │
│          └── tags/                                       │
│                                                          │
│  Total: ~30 files, only a few KB                         │
└──────────────────────────────────────────────────────────┘
```

---

## Initial Setup After `git init`

```bash
# Step 1: Set your identity (if not set globally)
$ git config user.name "Your Name"
$ git config user.email "your.email@example.com"

# Step 2: Set the default branch name
$ git branch -M main

# Step 3: Create initial files
$ echo "# My Project" > README.md
$ echo "node_modules/" > .gitignore

# Step 4: Make your first commit
$ git add .
$ git commit -m "Initial commit"

# Step 5: Add a remote (if needed)
$ git remote add origin https://github.com/user/repo.git

# Step 6: Push to remote
$ git push -u origin main
```

---

## Repository Types

### Regular Repository (Default)

```bash
$ git init
# Creates a working directory + .git/ folder
# This is what you use for development
```

### Bare Repository

```bash
$ git init --bare my-project.git
# Creates ONLY the .git contents (no working directory)
# Used for shared/remote repositories (servers)
```

```
┌──────────────────────────────────────────────────────────┐
│     REGULAR vs BARE REPOSITORY                           │
│                                                          │
│  Regular Repository:          Bare Repository:           │
│  ┌────────────────┐          ┌────────────────┐          │
│  │  Working Dir   │          │                │          │
│  │  ┌──────────┐  │          │   objects/     │          │
│  │  │  .git/   │  │          │   refs/        │          │
│  │  │ objects/ │  │          │   HEAD         │          │
│  │  │ refs/    │  │          │   config       │          │
│  │  └──────────┘  │          │   (no working  │          │
│  │  + your files  │          │    directory)  │          │
│  └────────────────┘          └────────────────┘          │
│                                                          │
│  For: Development            For: Shared server          │
│  You: Edit, commit, push     Others: Push/pull to it     │
└──────────────────────────────────────────────────────────┘
```

---

## Verifying Repository State

```bash
# Check if you're in a Git repo
$ git status
# If NOT in a repo: "fatal: not a git repository"

# See repository root
$ git rev-parse --show-toplevel
/home/user/my-project

# See .git directory location
$ git rev-parse --git-dir
.git

# Check if bare
$ git rev-parse --is-bare-repository
false
```

---

## Re-initializing a Repository

Running `git init` in an existing repo is **safe** — it won't overwrite anything:

```bash
# Safe to run again
$ git init
Reinitialized existing Git repository in /path/to/my-project/.git/

# Useful for:
# - Adding missing hook templates
# - Fixing a corrupted .git directory structure
```

---

## Real-World Scenarios

### Scenario 1: Starting a New Project

```bash
# Professional project setup
$ mkdir awesome-api
$ cd awesome-api
$ git init
$ git branch -M main

# Create essential files
$ echo "# Awesome API" > README.md
$ cat > .gitignore << 'EOF'
node_modules/
.env
dist/
*.log
.DS_Store
EOF

$ cat > .editorconfig << 'EOF'
root = true
[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
EOF

# First commit
$ git add .
$ git commit -m "Initial project setup"

# Connect to remote
$ git remote add origin git@github.com:user/awesome-api.git
$ git push -u origin main
```

### Scenario 2: Converting Existing Folder to Git Repo

```bash
# You already have files in a folder
$ cd existing-project/
$ ls
app.js  package.json  src/  docs/

# Initialize Git
$ git init

# Check what's there
$ git status
# Shows all files as "Untracked"

# Add everything and commit
$ git add .
$ git commit -m "Initial commit: import existing project"
```

### Scenario 3: Setting Up a Shared Bare Repo on a Server

```bash
# On the server
$ cd /srv/git
$ git init --bare team-project.git

# On developer machines
$ git clone user@server:/srv/git/team-project.git
$ cd team-project
# Start working...
```

---

## Common Mistakes and Fixes

| Mistake | Fix |
|---------|-----|
| `git init` in the wrong directory | Remove `.git/` folder: `rm -rf .git` |
| Nested git repositories | Move inner `.git/` or use submodules |
| Committed sensitive files in first commit | See `git filter-branch` or BFG Repo Cleaner |
| Forgot to create `.gitignore` before first commit | Create it now, unwanted files need `git rm --cached` |
| Wrong default branch name | `git branch -M main` to rename |

---

## Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "fatal: not a git repository" | You're not inside a repo. Run `git init` or `cd` to the repo |
| `.git` folder not visible | It's hidden. Use `ls -la` (Linux/Mac) or `dir /a` (Windows) |
| Want to undo `git init` | Simply delete the `.git` directory |
| Clone fails with "permission denied" | Check SSH keys (`ssh -T git@github.com`) or use HTTPS |
| Clone is very slow | Use `--depth 1` for a shallow clone |

---

## Summary Table

| Command / Concept | Description |
|-------------------|-------------|
| `git init` | Create a new empty repository in the current directory |
| `git init <name>` | Create a new directory with an empty repository |
| `git init --bare` | Create a bare repository (no working directory) |
| `git clone <url>` | Copy an existing repository including full history |
| `.git/` directory | Contains all Git data (objects, refs, config, etc.) |
| `HEAD` | Pointer to the current branch reference |
| Regular repo | Has working directory + `.git/` |
| Bare repo | Only `.git/` contents — for servers/sharing |
| `git rev-parse` | Utility to inspect repository properties |

---

## Quick Revision Questions

1. **What is the difference between `git init` and `git clone`?**
   <details><summary>Answer</summary>`git init` creates a new, empty repository from scratch. `git clone` copies an existing repository including all its history, branches, and files from a remote source.</details>

2. **What is a bare repository and when would you use one?**
   <details><summary>Answer</summary>A bare repository has no working directory — only the Git database. It's used on servers as a shared central repository that developers push to and pull from.</details>

3. **Is it safe to run `git init` in a directory that's already a Git repository?**
   <details><summary>Answer</summary>Yes, it's safe. It will "reinitialize" the repo without overwriting existing data. It can help restore missing hook templates or fix structural issues.</details>

4. **What are the first three things you should do after `git init`?**
   <details><summary>Answer</summary>1) Set user identity (`git config user.name/email`), 2) Create essential files (README.md, .gitignore), 3) Make the initial commit.</details>

5. **How would you convert an existing project folder into a Git repository?**
   <details><summary>Answer</summary>Navigate into the folder, run `git init`, then `git add .` to stage all files, and `git commit -m "Initial commit"` to create the first snapshot.</details>

6. **What happens if you accidentally run `git init` in the wrong directory?**
   <details><summary>Answer</summary>Simply delete the `.git` directory (`rm -rf .git` on Linux/Mac, or `Remove-Item -Recurse .git` on PowerShell). This removes all Git tracking from that folder.</details>

---

[← Previous: Git Objects](../01-Version-Control-Concepts/04-git-objects.md) | [Back to README](../README.md) | [Next: Staging and Committing →](02-staging-and-committing.md)
