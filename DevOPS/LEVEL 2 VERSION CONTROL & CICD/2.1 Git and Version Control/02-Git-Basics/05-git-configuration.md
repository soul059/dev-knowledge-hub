# Chapter 2.5: Git Configuration

[← Previous: .gitignore Configuration](04-gitignore-configuration.md) | [Back to README](../README.md) | [Next: Branch Concepts →](../03-Branching-and-Merging/01-branch-concepts.md)

---

## Overview

Git's behavior is controlled by a layered configuration system. Understanding and customizing these settings makes Git work the way **you** want it to, increasing productivity and reducing friction.

---

## Configuration Levels

```
┌──────────────────────────────────────────────────────────────┐
│           GIT CONFIGURATION LEVELS (PRIORITY ORDER)         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  3. LOCAL (Highest Priority)                         │   │
│  │     File: .git/config                                │   │
│  │     Scope: This repository only                      │   │
│  │     Set with: git config --local                     │   │
│  │                                                      │   │
│  │  ┌──────────────────────────────────────────────┐    │   │
│  │  │  2. GLOBAL (User-level)                      │    │   │
│  │  │     File: ~/.gitconfig or ~/.config/git/config│   │   │
│  │  │     Scope: All repos for this user            │   │   │
│  │  │     Set with: git config --global             │   │   │
│  │  │                                               │    │   │
│  │  │  ┌──────────────────────────────────────┐     │    │   │
│  │  │  │  1. SYSTEM (Lowest Priority)        │     │    │   │
│  │  │  │     File: /etc/gitconfig            │     │    │   │
│  │  │  │     Scope: All users on the machine │     │    │   │
│  │  │  │     Set with: git config --system   │     │    │   │
│  │  │  └──────────────────────────────────────┘     │    │   │
│  │  └──────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Local overrides Global, Global overrides System            │
└──────────────────────────────────────────────────────────────┘
```

### Viewing Configuration

```bash
# List all settings with their source
$ git config --list --show-origin

# List only global settings
$ git config --global --list

# List only local settings
$ git config --local --list

# Get a specific setting
$ git config user.name
$ git config user.email

# See where a specific setting is defined
$ git config --show-origin user.name
```

---

## Essential Configuration

### Identity Setup (Required)

```bash
# Set your name and email (used in every commit)
$ git config --global user.name "Alice Johnson"
$ git config --global user.email "alice@example.com"

# Per-repo identity (for work vs personal)
$ cd ~/work/company-project
$ git config --local user.name "Alice Johnson"
$ git config --local user.email "alice@company.com"
```

### Default Branch Name

```bash
# Set default branch to "main" (instead of "master")
$ git config --global init.defaultBranch main
```

### Default Editor

```bash
# VS Code
$ git config --global core.editor "code --wait"

# Vim
$ git config --global core.editor "vim"

# Nano
$ git config --global core.editor "nano"

# Notepad++ (Windows)
$ git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession"
```

### Line Endings

```bash
# Windows: convert LF to CRLF on checkout
$ git config --global core.autocrlf true

# macOS/Linux: convert CRLF to LF on commit
$ git config --global core.autocrlf input

# Disable auto-conversion
$ git config --global core.autocrlf false
```

```
┌────────────────────────────────────────────────────────────┐
│           LINE ENDING SETTINGS                             │
│                                                            │
│  Setting           │ On Commit    │ On Checkout            │
│  ──────────────────┼──────────────┼───────────────         │
│  autocrlf = true   │ CRLF → LF   │ LF → CRLF             │
│  (Windows)         │              │                        │
│  ──────────────────┼──────────────┼───────────────         │
│  autocrlf = input  │ CRLF → LF   │ No conversion          │
│  (macOS/Linux)     │              │                        │
│  ──────────────────┼──────────────┼───────────────         │
│  autocrlf = false  │ No change    │ No change              │
│  (Manual control)  │              │                        │
│                                                            │
│  Best practice: Use .gitattributes for per-file rules     │
└────────────────────────────────────────────────────────────┘
```

---

## Useful Configuration Options

### Diff and Merge Tools

```bash
# Set diff tool
$ git config --global diff.tool vscode
$ git config --global difftool.vscode.cmd "code --wait --diff \$LOCAL \$REMOTE"

# Set merge tool
$ git config --global merge.tool vscode
$ git config --global mergetool.vscode.cmd "code --wait \$MERGED"

# Don't keep .orig backup files after merge
$ git config --global mergetool.keepBackup false
```

### Color Output

```bash
# Enable colors (usually default)
$ git config --global color.ui auto

# Custom colors
$ git config --global color.status.changed "yellow"
$ git config --global color.status.untracked "red"
$ git config --global color.diff.meta "blue"
```

### Push Behavior

```bash
# Push only the current branch (safest)
$ git config --global push.default current

# Push and set upstream automatically
$ git config --global push.autoSetupRemote true
```

### Pull Behavior

```bash
# Rebase on pull instead of merge (cleaner history)
$ git config --global pull.rebase true

# Or always use --ff-only (fail if can't fast-forward)
$ git config --global pull.ff only
```

### Credential Storage

```bash
# Cache credentials in memory (15 min default)
$ git config --global credential.helper cache

# Cache for 1 hour
$ git config --global credential.helper 'cache --timeout=3600'

# Store permanently on disk (less secure)
$ git config --global credential.helper store

# Use OS keychain (recommended)
# macOS:
$ git config --global credential.helper osxkeychain
# Windows:
$ git config --global credential.helper manager
# Linux (GNOME):
$ git config --global credential.helper libsecret
```

---

## Git Aliases — Custom Shortcuts

```bash
# Create aliases for common commands
$ git config --global alias.st "status"
$ git config --global alias.co "checkout"
$ git config --global alias.br "branch"
$ git config --global alias.ci "commit"
$ git config --global alias.unstage "restore --staged"
$ git config --global alias.last "log -1 HEAD"
$ git config --global alias.visual "log --graph --oneline --all --decorate"

# Complex aliases
$ git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

$ git config --global alias.amend "commit --amend --no-edit"
$ git config --global alias.undo "reset HEAD~1 --mixed"
$ git config --global alias.wip "!git add -A && git commit -m 'WIP'"

# Usage:
$ git st                    # Instead of git status
$ git co feature/auth       # Instead of git checkout
$ git lg                    # Beautiful log
$ git undo                  # Undo last commit (keep changes)
$ git amend                 # Add staged changes to last commit
```

---

## The .gitconfig File

All global settings are stored in `~/.gitconfig`. You can edit it directly:

```ini
# ~/.gitconfig
[user]
    name = Alice Johnson
    email = alice@example.com

[init]
    defaultBranch = main

[core]
    editor = code --wait
    autocrlf = input
    whitespace = trailing-space,space-before-tab

[color]
    ui = auto

[push]
    default = current
    autoSetupRemote = true

[pull]
    rebase = true

[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    unstage = restore --staged
    last = log -1 HEAD
    amend = commit --amend --no-edit
    undo = reset HEAD~1 --mixed

[diff]
    tool = vscode

[difftool "vscode"]
    cmd = code --wait --diff $LOCAL $REMOTE

[merge]
    tool = vscode
    conflictstyle = diff3

[credential]
    helper = manager
```

---

## .gitattributes — Per-File Configuration

The `.gitattributes` file provides fine-grained control over how Git handles specific files:

```gitattributes
# Set default behavior
* text=auto

# Explicitly declare text files
*.js    text eol=lf
*.ts    text eol=lf
*.css   text eol=lf
*.html  text eol=lf
*.md    text eol=lf
*.json  text eol=lf
*.yml   text eol=lf
*.yaml  text eol=lf

# Declare binary files (don't attempt to diff/merge)
*.png   binary
*.jpg   binary
*.gif   binary
*.ico   binary
*.pdf   binary
*.zip   binary
*.woff  binary
*.woff2 binary

# Use Git LFS for large files
*.psd   filter=lfs diff=lfs merge=lfs -text
*.ai    filter=lfs diff=lfs merge=lfs -text

# Custom diff drivers
*.lock  linguist-generated=true
*.min.js linguist-generated=true
```

---

## Conditional Configuration

Use different configurations based on directory:

```ini
# ~/.gitconfig

[user]
    name = Alice Johnson
    email = alice@personal.com

# Use work email for repos under ~/work/
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work

# Use open-source email for repos under ~/oss/
[includeIf "gitdir:~/oss/"]
    path = ~/.gitconfig-oss
```

```ini
# ~/.gitconfig-work
[user]
    email = alice@company.com
    signingkey = ABC123DEF456
[commit]
    gpgsign = true
```

---

## Real-World Scenarios

### Scenario 1: Setting Up a New Development Machine

```bash
# Essential first-time setup
$ git config --global user.name "Alice Johnson"
$ git config --global user.email "alice@example.com"
$ git config --global init.defaultBranch main
$ git config --global core.editor "code --wait"
$ git config --global push.default current
$ git config --global push.autoSetupRemote true
$ git config --global pull.rebase true
$ git config --global credential.helper manager  # Windows
$ git config --global core.autocrlf true           # Windows

# Set up aliases
$ git config --global alias.st "status -s"
$ git config --global alias.lg "log --graph --oneline --all --decorate"

# Set up global gitignore
$ echo ".DS_Store\nThumbs.db\n.vscode/\n.idea/" > ~/.gitignore_global
$ git config --global core.excludesfile ~/.gitignore_global
```

### Scenario 2: Different Identities for Work and Personal

```bash
# Global (personal)
$ git config --global user.email "alice@personal.com"

# Work projects (per-repo or conditional include)
$ cd ~/work/project
$ git config --local user.email "alice@company.com"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Wrong email on commits | `git config user.email` to check, set the correct one |
| Config not taking effect | Check precedence: local > global > system |
| Editor doesn't wait for input | Add `--wait` flag (e.g., `code --wait`) |
| Line ending issues across team | Use `.gitattributes` with `* text=auto` |
| Need to remove a config option | `git config --global --unset option.name` |
| Want to reset all config | Delete `~/.gitconfig` and start fresh |

---

## Summary Table

| Setting | Command | Purpose |
|---------|---------|---------|
| User name | `git config --global user.name "Name"` | Identity for commits |
| User email | `git config --global user.email "email"` | Identity for commits |
| Default branch | `git config --global init.defaultBranch main` | Branch name for new repos |
| Editor | `git config --global core.editor "code --wait"` | Editor for commit messages |
| Line endings | `git config --global core.autocrlf true/input` | Cross-platform compatibility |
| Push default | `git config --global push.default current` | Push behavior |
| Pull rebase | `git config --global pull.rebase true` | Rebase instead of merge on pull |
| Credentials | `git config --global credential.helper manager` | Credential storage |
| Aliases | `git config --global alias.st "status"` | Custom command shortcuts |
| Conditional | `[includeIf "gitdir:~/work/"]` | Per-directory config |

---

## Quick Revision Questions

1. **What are the three levels of Git configuration and their priority order?**
   <details><summary>Answer</summary>System (lowest), Global/user (middle), and Local/repo (highest). Local overrides Global, which overrides System.</details>

2. **How do you set different email addresses for work and personal projects?**
   <details><summary>Answer</summary>Use `git config --local user.email` in each repo, or use conditional includes in `~/.gitconfig` with `[includeIf "gitdir:~/work/"]` pointing to a separate config file.</details>

3. **What does `core.autocrlf = true` do and when should you use it?**
   <details><summary>Answer</summary>On commit, it converts CRLF to LF. On checkout, it converts LF to CRLF. Use it on Windows to ensure consistent line endings in the repository (LF) while working natively (CRLF).</details>

4. **How do you create a Git alias and name two useful ones?**
   <details><summary>Answer</summary>Use `git config --global alias.<name> "<command>"`. Useful aliases: `git config --global alias.lg "log --graph --oneline --all"` and `git config --global alias.unstage "restore --staged"`.</details>

5. **What is `.gitattributes` and how does it differ from `.gitconfig`?**
   <details><summary>Answer</summary>`.gitattributes` is committed to the repo and controls per-file handling (line endings, binary detection, LFS). `.gitconfig` is user-level configuration (identity, editor, aliases) that's not part of the repo.</details>

6. **How do you verify where a specific Git configuration value is coming from?**
   <details><summary>Answer</summary>Use `git config --show-origin <setting>`, e.g., `git config --show-origin user.email`. It shows the file path and the value.</details>

---

[← Previous: .gitignore Configuration](04-gitignore-configuration.md) | [Back to README](../README.md) | [Next: Branch Concepts →](../03-Branching-and-Merging/01-branch-concepts.md)
