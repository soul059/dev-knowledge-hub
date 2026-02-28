# Chapter 2.4: .gitignore Configuration

[← Previous: Viewing History](03-viewing-history.md) | [Back to README](../README.md) | [Next: Git Configuration →](05-git-configuration.md)

---

## Overview

The `.gitignore` file tells Git which files and directories to **ignore** — meaning they won't be tracked, staged, or committed. Properly configuring `.gitignore` is essential for keeping your repository clean, secure, and efficient.

---

## Why .gitignore Matters

```
┌────────────────────────────────────────────────────────────┐
│           FILES YOU SHOULD NEVER COMMIT                    │
│                                                            │
│  🔒 SECRETS & CREDENTIALS                                  │
│     .env, .env.local, *.pem, credentials.json             │
│     API keys, database passwords, tokens                   │
│                                                            │
│  📦 DEPENDENCIES                                           │
│     node_modules/, venv/, vendor/                          │
│     (Restored via package manager)                         │
│                                                            │
│  🔨 BUILD OUTPUTS                                          │
│     dist/, build/, out/, *.o, *.class                     │
│     (Regenerated from source)                              │
│                                                            │
│  💻 IDE & OS FILES                                         │
│     .vscode/, .idea/, .DS_Store, Thumbs.db                │
│     (Personal configuration)                               │
│                                                            │
│  📝 LOGS & TEMP FILES                                      │
│     *.log, *.tmp, *.cache                                  │
│     (Transient, not part of source)                        │
└────────────────────────────────────────────────────────────┘
```

---

## .gitignore Syntax

### Basic Patterns

```gitignore
# This is a comment

# Ignore a specific file
secrets.json

# Ignore all files with an extension
*.log
*.tmp
*.cache

# Ignore a specific directory
node_modules/
dist/
build/

# Ignore files in any directory with this name
**/debug.log

# Ignore files only in the root directory
/TODO.md

# Ignore all files in a directory
doc/**

# Negate a pattern (DON'T ignore this)
*.log
!important.log

# Ignore everything inside a folder except specific files
build/*
!build/.gitkeep
```

### Pattern Rules

```
┌──────────────────────────────────────────────────────────────┐
│              .gitignore PATTERN REFERENCE                   │
│                                                              │
│  PATTERN        │ MATCHES                                   │
│  ───────────────┼───────────────────────────────────────     │
│  file.txt       │ file.txt in any directory                 │
│  /file.txt      │ file.txt ONLY in root directory           │
│  dir/           │ The directory "dir" and its contents      │
│  *.log          │ Any file ending in .log                   │
│  *.py[cod]      │ .pyc, .pyo, .pyd files                   │
│  debug[0-9].log │ debug0.log through debug9.log             │
│  debug?.log     │ debug0.log, debuga.log (single char)     │
│  **/logs        │ "logs" directory anywhere in tree         │
│  **/logs/*.log  │ .log files in any "logs" directory        │
│  logs/**        │ Everything inside "logs/" directory       │
│  !important.log │ DON'T ignore this (negation)             │
│  #comment       │ Comment line (ignored by Git)             │
│  \#file         │ File literally named "#file"              │
│                                                              │
│  SPECIAL CHARACTERS:                                        │
│  *  = matches any characters (except /)                    │
│  ** = matches any characters (including /)                  │
│  ?  = matches any single character                         │
│  [] = matches any character inside brackets                │
│  !  = negates (un-ignores) a pattern                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Common .gitignore Templates

### Node.js / JavaScript

```gitignore
# Dependencies
node_modules/
bower_components/

# Build output
dist/
build/
out/

# Environment variables
.env
.env.local
.env.*.local

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Coverage
coverage/
*.lcov

# Cache
.cache/
.parcel-cache/
.next/
```

### Python

```gitignore
# Bytecode
__pycache__/
*.py[cod]
*$py.class

# Virtual environments
venv/
.venv/
env/
ENV/

# Distribution
dist/
build/
*.egg-info/
*.egg

# Environment
.env
.env.local

# IDE
.vscode/
.idea/
*.swp

# Jupyter
.ipynb_checkpoints/

# Testing
.pytest_cache/
.coverage
htmlcov/

# OS
.DS_Store
Thumbs.db
```

### Java

```gitignore
# Compiled
*.class
*.jar
*.war
*.ear

# Build
target/
build/
out/

# IDE
.idea/
*.iml
.eclipse/
.settings/
.project
.classpath

# Maven
pom.xml.tag
pom.xml.releaseBackup

# Gradle
.gradle/
gradle-app.setting

# OS
.DS_Store
Thumbs.db
```

### General Purpose (Multi-language)

```gitignore
# === OS Generated ===
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
desktop.ini

# === IDE & Editors ===
.vscode/
.idea/
*.swp
*.swo
*~
.project
.settings/

# === Environment ===
.env
.env.*
!.env.example

# === Logs ===
*.log
logs/

# === Dependencies ===
node_modules/
vendor/
venv/

# === Build ===
dist/
build/
out/
*.o
*.so
*.dll
```

---

## .gitignore Placement and Scope

```
┌──────────────────────────────────────────────────────────────┐
│           .gitignore LEVELS                                 │
│                                                              │
│  1. REPOSITORY .gitignore (committed, shared)               │
│     Location: <repo-root>/.gitignore                        │
│     Scope: Everyone who clones the repo                     │
│     Use for: Language-specific ignores (node_modules, etc.) │
│                                                              │
│  2. SUBDIRECTORY .gitignore (committed, shared)             │
│     Location: <any-subdir>/.gitignore                       │
│     Scope: Only that directory and below                    │
│     Use for: Directory-specific overrides                   │
│                                                              │
│  3. .git/info/exclude (local, NOT shared)                   │
│     Location: .git/info/exclude                              │
│     Scope: Only your local clone                            │
│     Use for: Personal files you don't want tracked          │
│                                                              │
│  4. GLOBAL .gitignore (local, NOT shared)                   │
│     Location: ~/.gitignore_global                           │
│     Scope: All repos on your machine                        │
│     Use for: OS/IDE files (.DS_Store, .vscode/)             │
└──────────────────────────────────────────────────────────────┘
```

### Setting Up Global .gitignore

```bash
# Create a global gitignore
$ cat > ~/.gitignore_global << 'EOF'
# OS files
.DS_Store
Thumbs.db
desktop.ini

# IDE files
.vscode/
.idea/
*.swp

# Temp files
*.tmp
*~
EOF

# Tell Git to use it
$ git config --global core.excludesfile ~/.gitignore_global
```

---

## Handling Already-Tracked Files

`.gitignore` only affects **untracked files**. If a file is already being tracked, adding it to `.gitignore` won't stop tracking it.

```bash
# Problem: .env was committed before .gitignore was set up

# Step 1: Add to .gitignore
$ echo ".env" >> .gitignore

# Step 2: Remove from Git tracking (but keep the file)
$ git rm --cached .env

# Step 3: Commit the changes
$ git add .gitignore
$ git commit -m "Stop tracking .env file"

# The file stays on your disk but Git ignores it now
```

### Bulk Untrack

```bash
# Remove ALL files that should be ignored (nuclear option)
# WARNING: This re-adds everything — make sure .gitignore is correct first!

$ git rm -r --cached .
$ git add .
$ git commit -m "Apply .gitignore rules to tracked files"
```

---

## Debugging .gitignore

```bash
# Check if a file is ignored and by which rule
$ git check-ignore -v filename.txt
# Output: .gitignore:3:*.txt  filename.txt
#         ^file      ^line ^pattern  ^file checked

# List all ignored files
$ git status --ignored

# Check why a specific file is or isn't ignored
$ git check-ignore -v path/to/file

# Force add an ignored file (not recommended)
$ git add -f ignored-file.txt
```

---

## .gitkeep — Tracking Empty Directories

Git doesn't track empty directories. Use a `.gitkeep` convention:

```bash
# Create empty directory with placeholder
$ mkdir -p logs/
$ touch logs/.gitkeep

# In .gitignore, ignore log files but keep the directory
# logs/*.log
# !logs/.gitkeep
```

---

## Real-World Scenarios

### Scenario 1: New Developer Clones Repo, Sees Unwanted Files

```bash
# Problem: Someone committed node_modules/ to the repo

# Fix: Add to .gitignore and untrack
$ echo "node_modules/" >> .gitignore
$ git rm -r --cached node_modules/
$ git commit -m "Remove node_modules from tracking"
$ git push
```

### Scenario 2: Different Ignores for Different Environments

```
project/
├── .gitignore              # Shared ignores
├── .git/
│   └── info/
│       └── exclude         # Personal ignores (not committed)
├── frontend/
│   └── .gitignore          # Frontend-specific ignores
└── backend/
    └── .gitignore          # Backend-specific ignores
```

### Scenario 3: Using .env.example Pattern

```bash
# .gitignore
.env
.env.*
!.env.example

# .env.example (committed — template for other developers)
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
API_KEY=your-api-key-here
SECRET=your-secret-here

# .env (NOT committed — real values)
DATABASE_URL=postgresql://admin:s3cur3@prod-server:5432/proddb
API_KEY=sk_live_abc123xyz789
SECRET=my-real-secret-key-here
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `.gitignore` not working | File is already tracked — use `git rm --cached` |
| Need to ignore a tracked file | `git rm --cached <file>` then add to `.gitignore` |
| Want to check which rule ignores a file | `git check-ignore -v <file>` |
| Pattern not matching | Check for trailing spaces, use `git check-ignore -v` |
| Need to commit a normally ignored file | `git add -f <file>` (not recommended) |
| Global ignores not working | Verify `git config --global core.excludesfile` is set |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| `.gitignore` | File listing patterns of files/dirs Git should ignore |
| `*` | Wildcard — matches anything except `/` |
| `**` | Matches directories at any level |
| `!` | Negation — un-ignores a previously ignored pattern |
| `/pattern` | Matches only in root directory |
| `dir/` | Ignores a directory and all contents |
| `.git/info/exclude` | Local-only ignore rules (not committed) |
| `~/.gitignore_global` | Global ignores across all repos on your machine |
| `git rm --cached` | Stop tracking a file without deleting it |
| `git check-ignore -v` | Debug which rule is ignoring a file |
| `.gitkeep` | Convention to track otherwise empty directories |

---

## Quick Revision Questions

1. **What is the difference between `.gitignore` and `.git/info/exclude`?**
   <details><summary>Answer</summary>`.gitignore` is committed and shared with all collaborators. `.git/info/exclude` is local to your machine and never committed — for personal ignores that don't apply to the team.</details>

2. **Why doesn't adding a file to `.gitignore` stop Git from tracking it if it's already been committed?**
   <details><summary>Answer</summary>`.gitignore` only prevents untracked files from being added. To stop tracking an already-committed file, you must run `git rm --cached <file>` to remove it from the index, then commit.</details>

3. **What does the `!` prefix do in a `.gitignore` pattern?**
   <details><summary>Answer</summary>It negates (un-ignores) a pattern. For example, `*.log` ignores all log files, but `!important.log` makes an exception for `important.log`.</details>

4. **How do you set up a global `.gitignore` for your machine?**
   <details><summary>Answer</summary>Create a global gitignore file (e.g., `~/.gitignore_global`), add your patterns, then set it with `git config --global core.excludesfile ~/.gitignore_global`.</details>

5. **How can you debug why a file is being ignored?**
   <details><summary>Answer</summary>Use `git check-ignore -v <filename>` — it shows which gitignore file and which line/pattern is causing the file to be ignored.</details>

6. **Name four categories of files that should always be in `.gitignore`.**
   <details><summary>Answer</summary>1) Secrets/credentials (.env, API keys), 2) Dependencies (node_modules/, venv/), 3) Build outputs (dist/, *.class), 4) OS/IDE files (.DS_Store, .idea/).</details>

---

[← Previous: Viewing History](03-viewing-history.md) | [Back to README](../README.md) | [Next: Git Configuration →](05-git-configuration.md)
