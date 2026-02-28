# Chapter 6.5: Tags and Releases

[← Previous: Reflog and Recovery](04-reflog-and-recovery.md) | [Back to README](../README.md) | [Next: Git LFS →](06-git-lfs.md)

---

## Overview

**Tags** are named references that point to specific commits — typically used to mark **release points** (v1.0, v2.3.1). Unlike branches, tags don't move — they permanently point to a single commit, making them ideal for marking versions.

---

## Types of Tags

```
┌──────────────────────────────────────────────────────────────┐
│       TWO TYPES OF TAGS                                     │
│                                                              │
│  ┌─────────────────┐       ┌─────────────────────┐          │
│  │  LIGHTWEIGHT     │       │  ANNOTATED           │          │
│  │                 │       │                     │          │
│  │  Just a pointer │       │  Full Git object     │          │
│  │  to a commit    │       │  with:               │          │
│  │                 │       │  • Tagger name       │          │
│  │  Like a branch  │       │  • Date              │          │
│  │  that doesn't   │       │  • Message           │          │
│  │  move            │       │  • Optional GPG sig  │          │
│  │                 │       │                     │          │
│  │  $ git tag v1.0 │       │  $ git tag -a v1.0   │          │
│  │                 │       │    -m "Release 1.0"  │          │
│  └─────────────────┘       └─────────────────────┘          │
│                                                              │
│  RECOMMENDATION: Always use ANNOTATED tags for releases     │
│  (they have metadata and can be verified)                   │
└──────────────────────────────────────────────────────────────┘
```

---

## Creating Tags

### Annotated Tags (Recommended)

```bash
# Create annotated tag on current commit
$ git tag -a v1.0.0 -m "Release version 1.0.0"

# Create annotated tag on a specific commit
$ git tag -a v0.9.0 -m "Beta release" abc1234

# Create with GPG signature
$ git tag -s v1.0.0 -m "Signed release 1.0.0"
```

### Lightweight Tags

```bash
# Create lightweight tag (just a pointer)
$ git tag v1.0.0-rc1

# Tag a specific commit
$ git tag v0.8.0 abc1234
```

---

## Viewing Tags

```bash
# List all tags
$ git tag

# List with pattern matching
$ git tag -l "v1.*"
v1.0.0
v1.0.1
v1.1.0

# Show tag details
$ git show v1.0.0
# (For annotated: shows tagger, date, message, and commit)
# (For lightweight: shows only the commit)

# List tags with commit messages
$ git tag -n
v1.0.0    Release version 1.0.0
v1.1.0    Added new features
```

---

## Pushing Tags to Remote

```bash
# Tags are NOT pushed by default!

# Push a specific tag
$ git push origin v1.0.0

# Push ALL tags
$ git push origin --tags

# Push only annotated tags (not lightweight)
$ git push origin --follow-tags
```

```
┌──────────────────────────────────────────────────────────────┐
│  IMPORTANT: Tags are NOT pushed with git push               │
│                                                              │
│  $ git push origin main     → pushes commits only           │
│  $ git push origin v1.0.0   → pushes one tag               │
│  $ git push origin --tags   → pushes ALL tags               │
│                                                              │
│  Set auto-push for annotated tags:                          │
│  $ git config --global push.followTags true                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Deleting Tags

```bash
# Delete local tag
$ git tag -d v1.0.0

# Delete remote tag
$ git push origin --delete v1.0.0
# Or:
$ git push origin :refs/tags/v1.0.0
```

---

## Checking Out Tags

```bash
# View code at a tagged point (detached HEAD)
$ git checkout v1.0.0
# ⚠️ You're in "detached HEAD" state — can view but shouldn't commit

# Create a branch from a tag (for making changes)
$ git switch -c hotfix/v1.0.1 v1.0.0
```

```
┌──────────────────────────────────────────────────────────────┐
│  CHECKING OUT A TAG                                         │
│                                                              │
│  ●──●──●──●──●──●──●                                       │
│        ▲        ▲   ▲                                      │
│      v1.0     v1.1 HEAD (main)                             │
│                                                              │
│  $ git checkout v1.0                                        │
│                                                              │
│  ●──●──●──●──●──●──●                                       │
│        ▲                                                   │
│       HEAD (detached!) — viewing v1.0                       │
│                                                              │
│  To make changes: create a branch first!                    │
│  $ git switch -c fix/from-v1.0                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Semantic Versioning with Tags

```
┌──────────────────────────────────────────────────────────────┐
│       SEMANTIC VERSIONING (SemVer)                          │
│                                                              │
│       v MAJOR . MINOR . PATCH                               │
│       v 2     . 4     . 1                                   │
│         │       │       │                                   │
│         │       │       └── Bug fixes (backward compatible) │
│         │       └────────── New features (backward compat.) │
│         └────────────────── Breaking changes                │
│                                                              │
│  Examples:                                                  │
│  v1.0.0 → v1.0.1  (patch: bug fix)                         │
│  v1.0.1 → v1.1.0  (minor: new feature)                     │
│  v1.1.0 → v2.0.0  (major: breaking change)                 │
│                                                              │
│  Pre-release: v1.0.0-alpha, v1.0.0-beta.1, v1.0.0-rc.1    │
│  Build meta:  v1.0.0+build.123                              │
└──────────────────────────────────────────────────────────────┘
```

---

## GitHub Releases

```
┌──────────────────────────────────────────────────────────────┐
│  GitHub Releases build on Git tags:                         │
│                                                              │
│  1. Create a tag: $ git tag -a v1.0.0 -m "Release 1.0"     │
│  2. Push it:      $ git push origin v1.0.0                  │
│  3. On GitHub → Releases → "Create release from tag"        │
│  4. Add:                                                    │
│     • Release title                                         │
│     • Release notes / changelog                             │
│     • Binary attachments (e.g., compiled builds)            │
│                                                              │
│  GitHub can also AUTO-GENERATE release notes from           │
│  PR titles and commit messages.                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Tagging a Release

```bash
$ git switch main
$ git pull origin main
$ git tag -a v2.0.0 -m "Release 2.0.0 - Major redesign"
$ git push origin v2.0.0
```

### Scenario 2: Tagging a Past Commit

```bash
# Forgot to tag — tag an older commit
$ git log --oneline
abc1234 Current commit
def5678 This should have been v1.5.0
ghi9012 Older commit

$ git tag -a v1.5.0 -m "Release 1.5.0" def5678
$ git push origin v1.5.0
```

### Scenario 3: Finding What Version Has a Bug

```bash
# What tag contains a specific commit?
$ git tag --contains abc1234

# Which tags are between two commits?
$ git log --oneline v1.0.0..v2.0.0
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Tag not showing on remote | `git push origin <tag-name>` — tags aren't auto-pushed |
| "tag already exists" | Delete first: `git tag -d <name>`, then recreate |
| Need to move a tag | Delete and recreate: `git tag -d v1.0 && git tag -a v1.0 -m "msg"` |
| "detached HEAD" after checkout tag | Normal — create a branch if you need to make changes |
| Tags not appearing after clone | Tags are included in clone; use `git tag` to list them |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `git tag -a v1.0 -m "msg"` | Create annotated tag |
| `git tag v1.0` | Create lightweight tag |
| `git tag` | List all tags |
| `git tag -l "v1.*"` | List tags matching pattern |
| `git show v1.0` | Show tag details |
| `git push origin v1.0` | Push specific tag |
| `git push origin --tags` | Push all tags |
| `git tag -d v1.0` | Delete local tag |
| `git push origin --delete v1.0` | Delete remote tag |
| `git checkout v1.0` | View code at tag (detached HEAD) |

---

## Quick Revision Questions

1. **What is the difference between annotated and lightweight tags?**
   <details><summary>Answer</summary>Annotated tags are full Git objects with tagger name, date, message, and optional GPG signature. Lightweight tags are simple pointers to a commit with no additional metadata. Annotated tags are recommended for releases.</details>

2. **Are tags pushed to the remote with `git push`?**
   <details><summary>Answer</summary>No. Tags must be explicitly pushed with `git push origin <tag-name>` or `git push origin --tags`. You can enable automatic pushing of annotated tags with `git config --global push.followTags true`.</details>

3. **What is Semantic Versioning?**
   <details><summary>Answer</summary>A versioning scheme using MAJOR.MINOR.PATCH format. MAJOR for breaking changes, MINOR for new backward-compatible features, PATCH for backward-compatible bug fixes. Example: v2.4.1.</details>

4. **What happens when you `git checkout` a tag?**
   <details><summary>Answer</summary>You enter "detached HEAD" state — you can view the code at that tag but shouldn't make commits. To make changes, create a branch from the tag: `git switch -c branch-name v1.0.0`.</details>

5. **How do you tag a commit that was already pushed?**
   <details><summary>Answer</summary>Use `git tag -a v1.0.0 -m "message" <commit-hash>` to tag a specific commit, then `git push origin v1.0.0` to push the tag to the remote.</details>

---

[← Previous: Reflog and Recovery](04-reflog-and-recovery.md) | [Back to README](../README.md) | [Next: Git LFS →](06-git-lfs.md)
