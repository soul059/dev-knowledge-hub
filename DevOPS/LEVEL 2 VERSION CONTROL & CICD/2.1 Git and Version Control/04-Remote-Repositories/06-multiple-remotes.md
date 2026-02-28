# Chapter 4.6: Multiple Remotes

[← Previous: Forking Workflow](05-forking-workflow.md) | [Back to README](../README.md) | [Next: Feature Branch Workflow →](../05-Git-Workflows/01-feature-branch-workflow.md)

---

## Overview

Git allows you to configure **multiple remotes** — connections to different remote repositories. This is essential for fork-based development, mirroring, deploying to multiple servers, and collaborating across platforms.

---

## Why Multiple Remotes?

```
┌──────────────────────────────────────────────────────────────┐
│        COMMON MULTI-REMOTE SETUPS                           │
│                                                              │
│  1. FORK WORKFLOW (origin + upstream)                       │
│  ┌──────────┐                                               │
│  │  LOCAL   │── origin ──────► your fork (GitHub)           │
│  │  REPO    │── upstream ────► original repo (GitHub)       │
│  └──────────┘                                               │
│                                                              │
│  2. MULTI-PLATFORM (GitHub + GitLab)                        │
│  ┌──────────┐                                               │
│  │  LOCAL   │── github ──────► github.com/user/repo         │
│  │  REPO    │── gitlab ──────► gitlab.com/user/repo         │
│  └──────────┘                                               │
│                                                              │
│  3. DEPLOY (code + deploy targets)                          │
│  ┌──────────┐                                               │
│  │  LOCAL   │── origin ──────► github.com/user/repo         │
│  │  REPO    │── staging ─────► staging.server.com/repo      │
│  │          │── production ──► prod.server.com/repo         │
│  └──────────┘                                               │
│                                                              │
│  4. TEAM MEMBERS (peer-to-peer)                             │
│  ┌──────────┐                                               │
│  │  LOCAL   │── origin ──────► main server                  │
│  │  REPO    │── alice ───────► alice's repo                 │
│  │          │── bob ─────────► bob's repo                   │
│  └──────────┘                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Setting Up Multiple Remotes

```bash
# You already have origin (from clone)
$ git remote -v
origin  https://github.com/you/project.git (fetch)
origin  https://github.com/you/project.git (push)

# Add more remotes
$ git remote add upstream https://github.com/org/project.git
$ git remote add gitlab git@gitlab.com:you/project.git
$ git remote add staging ssh://deploy@staging.example.com/repo.git

# Verify all remotes
$ git remote -v
gitlab    git@gitlab.com:you/project.git (fetch)
gitlab    git@gitlab.com:you/project.git (push)
origin    https://github.com/you/project.git (fetch)
origin    https://github.com/you/project.git (push)
staging   ssh://deploy@staging.example.com/repo.git (fetch)
staging   ssh://deploy@staging.example.com/repo.git (push)
upstream  https://github.com/org/project.git (fetch)
upstream  https://github.com/org/project.git (push)
```

---

## Working with Multiple Remotes

### Fetching from Different Remotes

```bash
# Fetch from a specific remote
$ git fetch upstream
$ git fetch gitlab

# Fetch from ALL remotes at once
$ git fetch --all
```

### Pushing to Different Remotes

```bash
# Push to your fork
$ git push origin main

# Push to GitLab mirror
$ git push gitlab main

# Push to all remotes at once (custom alias)
$ git config alias.pushall '!git push origin && git push gitlab'
$ git pushall
```

### Pulling from Specific Remotes

```bash
# Pull from upstream
$ git pull upstream main

# Pull from a colleague's repo
$ git pull alice feature/experiment
```

---

## Mirroring to Multiple Platforms

```bash
# Push the same code to GitHub and GitLab simultaneously
# Option 1: Push separately
$ git push origin main
$ git push gitlab main

# Option 2: Add multiple push URLs to one remote
$ git remote set-url --add --push origin https://github.com/you/repo.git
$ git remote set-url --add --push origin git@gitlab.com:you/repo.git

# Now a single push goes to BOTH
$ git push origin main   # Pushes to GitHub AND GitLab

# Verify
$ git remote -v
origin  https://github.com/you/repo.git (fetch)
origin  https://github.com/you/repo.git (push)
origin  git@gitlab.com:you/repo.git (push)
```

```
┌──────────────────────────────────────────────────────────────┐
│        MIRROR PUSH SETUP                                    │
│                                                              │
│                      ┌──────── GitHub ◄─── push URL 1       │
│  $ git push origin ──┤                                      │
│                      └──────── GitLab ◄─── push URL 2       │
│                                                              │
│  One command pushes to both platforms!                       │
│  Fetch still comes from the first URL only.                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Comparing Branches Across Remotes

```bash
# Compare your branch with upstream's main
$ git log origin/main..upstream/main --oneline

# Compare your fork with original
$ git diff origin/main upstream/main

# See all remote branches
$ git branch -r
  origin/main
  origin/develop
  upstream/main
  upstream/develop
  gitlab/main
```

---

## Real-World Scenarios

### Scenario 1: Open-Source Fork Workflow

```bash
# Setup
$ git clone https://github.com/you/framework.git
$ git remote add upstream https://github.com/org/framework.git

# Daily sync
$ git fetch upstream
$ git switch main
$ git merge upstream/main
$ git push origin main

# Contribute
$ git switch -c fix/bug-123
# ... make changes ...
$ git push -u origin fix/bug-123
# Create PR from your fork to upstream
```

### Scenario 2: Multi-Platform Hosting

```bash
# Push code to both GitHub and GitLab
$ git remote add github https://github.com/you/project.git
$ git remote add gitlab git@gitlab.com:you/project.git

# Push to both
$ git push github main
$ git push gitlab main
```

### Scenario 3: Deploy via Git Push

```bash
# Add deployment remotes
$ git remote add staging ssh://deploy@staging.api.com/repo.git
$ git remote add production ssh://deploy@api.com/repo.git

# Deploy to staging
$ git push staging main

# Deploy to production
$ git push production main
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "remote already exists" | `git remote set-url <name> <url>` to update URL |
| Can't fetch from upstream | Check URL: `git remote -v`; verify network access |
| Push goes to wrong remote | Specify explicitly: `git push <remote> <branch>` |
| Remote-tracking branches mixed up | Use `git branch -r` and prefix with remote name |
| Want to remove a remote | `git remote remove <name>` |
| Multiple push URLs not working | Use `git remote set-url --add --push` (note: `--push` flag) |

---

## Summary Table

| Task | Command |
|------|---------|
| Add a remote | `git remote add <name> <url>` |
| List all remotes | `git remote -v` |
| Fetch from specific remote | `git fetch <remote>` |
| Fetch from all remotes | `git fetch --all` |
| Push to specific remote | `git push <remote> <branch>` |
| Add multiple push URLs | `git remote set-url --add --push <remote> <url>` |
| Compare across remotes | `git log origin/main..upstream/main` |
| Remove a remote | `git remote remove <name>` |

---

## Quick Revision Questions

1. **Why would you need multiple remotes?**
   <details><summary>Answer</summary>Common reasons include: fork workflow (origin + upstream), mirroring to multiple platforms (GitHub + GitLab), deployment targets (staging + production), and peer-to-peer collaboration (fetching from teammates' repos).</details>

2. **How do you push to multiple platforms with a single command?**
   <details><summary>Answer</summary>Add multiple push URLs to one remote: `git remote set-url --add --push origin <url2>`. Then `git push origin` sends to all configured push URLs simultaneously.</details>

3. **How do you fetch from all configured remotes at once?**
   <details><summary>Answer</summary>`git fetch --all` downloads new data from every configured remote repository.</details>

4. **In a fork workflow, what's the typical remote naming convention?**
   <details><summary>Answer</summary>"origin" points to your personal fork (where you push). "upstream" points to the original repository (where you pull updates from). This is a widely-adopted convention in open-source development.</details>

5. **How do you compare branches across different remotes?**
   <details><summary>Answer</summary>Use `git log origin/main..upstream/main` to see commits in upstream that aren't in origin, or `git diff origin/main upstream/main` to see the code differences.</details>

---

[← Previous: Forking Workflow](05-forking-workflow.md) | [Back to README](../README.md) | [Next: Feature Branch Workflow →](../05-Git-Workflows/01-feature-branch-workflow.md)
