# Chapter 1.3: Git History and Architecture

[← Previous: Centralized vs Distributed VCS](02-centralized-vs-distributed.md) | [Back to README](../README.md) | [Next: Git Objects →](04-git-objects.md)

---

## Overview

Understanding Git's origin story and internal architecture gives you a deeper mental model of **how** and **why** Git works the way it does. This knowledge is invaluable for troubleshooting and mastering advanced features.

---

## A Brief History of Git

### The Linux Kernel Problem (2002–2005)

```
┌──────────────────────────────────────────────────────────────┐
│                  THE BIRTH OF GIT                            │
│                                                              │
│  1991-2002: Linux kernel managed with patches & tarballs    │
│       └─ Thousands of developers, no VCS → chaos            │
│                                                              │
│  2002: Linux adopts BitKeeper (proprietary DVCS)            │
│       └─ Free license for open-source use                    │
│                                                              │
│  2005: BitKeeper revokes free license                        │
│       └─ Community member reverse-engineered the protocol    │
│                                                              │
│  April 2005: Linus Torvalds creates Git                     │
│       └─ "I'm an egotistical bastard, so I name all my      │
│          projects after myself. First Linux, now Git."       │
│          — Linus Torvalds                                    │
│                                                              │
│  June 2005: Linux kernel development moves to Git           │
│       └─ Git manages its own source code from day one        │
│                                                              │
│  2008: GitHub launches → Git adoption explodes              │
│       └─ Social coding platform makes Git accessible         │
│                                                              │
│  2025: Git used by >95% of developers worldwide             │
└──────────────────────────────────────────────────────────────┘
```

### Linus Torvalds' Design Goals for Git

1. **Speed** — Operations must be fast (Linux kernel is a massive project)
2. **Simple design** — The internal model should be elegant
3. **Strong support for non-linear development** — Thousands of parallel branches
4. **Fully distributed** — No central point of failure
5. **Able to handle large projects** — Like the Linux kernel (millions of lines of code)

---

## Git's Architecture — The Big Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GIT ARCHITECTURE                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  WORKING DIRECTORY                      │    │
│  │                                                         │    │
│  │   The files you see and edit in your project folder     │    │
│  │   ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                  │    │
│  │   │app.js│ │style │ │index │ │README│                  │    │
│  │   │      │ │.css  │ │.html │ │.md   │                  │    │
│  │   └──────┘ └──────┘ └──────┘ └──────┘                  │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │ git add                                │
│                        ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  STAGING AREA (INDEX)                    │    │
│  │                                                         │    │
│  │   Prepares the next commit — a "loading dock"           │    │
│  │   You choose exactly which changes to include           │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │ git commit                             │
│                        ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  LOCAL REPOSITORY (.git/)                │    │
│  │                                                         │    │
│  │   Complete history of all commits                       │    │
│  │   Stored in the .git directory                          │    │
│  │   ┌────────┐ ┌────────┐ ┌────────┐                      │    │
│  │   │Commit 3│─│Commit 2│─│Commit 1│                      │    │
│  │   └────────┘ └────────┘ └────────┘                      │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │ git push / git pull                    │
│                        ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  REMOTE REPOSITORY                      │    │
│  │                  (GitHub / GitLab)                       │    │
│  │                                                         │    │
│  │   Shared repository for collaboration                   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Three Areas of Git

### 1. Working Directory (Working Tree)

The actual files and folders in your project directory. This is what you see in your file explorer and editor.

```bash
# See the status of your working directory
$ git status

# Example output:
On branch main
Changes not staged for commit:
  modified:   app.js
  modified:   style.css

Untracked files:
  new-feature.js
```

**File States in the Working Directory:**
- **Untracked** — New file, Git doesn't know about it
- **Modified** — File has changed since last commit
- **Unmodified** — File matches the last commit (clean)

### 2. Staging Area (Index)

A preparation area between your working directory and the repository. It lets you **selectively choose** which changes to include in the next commit.

```bash
# Stage specific files
$ git add app.js

# Stage all changes
$ git add .

# Stage parts of a file (interactive)
$ git add -p app.js
```

**Why a staging area?**
- Craft precise, logical commits (not just "save everything")
- Review what's going into a commit before making it
- Split unrelated changes into separate commits

### 3. Local Repository (.git directory)

The `.git` folder in your project root contains **everything** Git needs:

```
.git/
├── HEAD              # Points to current branch
├── config            # Repository-specific config
├── description       # Used by GitWeb (rarely used)
├── hooks/            # Client/server-side scripts
├── index             # The staging area (binary file)
├── info/             # Global exclude patterns
├── objects/          # ALL content (blobs, trees, commits)
│   ├── pack/         # Packed objects for efficiency
│   └── info/         # Pack metadata
├── refs/             # Pointers to commits
│   ├── heads/        # Branch pointers
│   ├── tags/         # Tag pointers
│   └── remotes/      # Remote tracking branches
└── logs/             # History of ref changes (reflog)
```

---

## The Git Data Flow

```
┌────────────────────────────────────────────────────────┐
│              GIT DATA FLOW DIAGRAM                     │
│                                                        │
│   Working          Staging         Local       Remote  │
│  Directory          Area           Repo        Repo    │
│     │                │               │           │     │
│     │── git add ────▶│               │           │     │
│     │                │── git commit─▶│           │     │
│     │                │               │── push ──▶│     │
│     │                │               │           │     │
│     │                │               │◀─ fetch ──│     │
│     │◀────────── git checkout ───────│           │     │
│     │                │               │           │     │
│     │◀──────────── git pull ─────────────────────│     │
│     │                │               │           │     │
│     │── git diff ───▶│               │           │     │
│     │                │── git diff ──▶│           │     │
│     │                │    --staged   │           │     │
│                                                        │
│  COMMANDS AND THEIR EFFECT ON DATA FLOW:               │
│                                                        │
│  git add       : Working Dir → Staging Area            │
│  git commit    : Staging Area → Local Repo             │
│  git push      : Local Repo → Remote Repo              │
│  git fetch     : Remote Repo → Local Repo              │
│  git pull      : Remote Repo → Working Dir (fetch+merge│
│  git checkout  : Local Repo → Working Dir               │
│  git diff      : Compare Working Dir vs Staging         │
│  git diff --staged : Compare Staging vs Last Commit    │
└────────────────────────────────────────────────────────┘
```

---

## How Git Stores Data — Snapshots, Not Diffs

### Other VCS: Delta-Based Storage

```
  OTHER VCS (SVN, CVS) — Stores DIFFERENCES (deltas)
  
  Version 1:   [File A v1] [File B v1] [File C v1]
                    │            │            │
  Version 2:   [  Δ A2   ] [          ] [  Δ C2   ]
                    │            │            │
  Version 3:   [          ] [  Δ B3   ] [  Δ C3   ]
                    │            │            │
  Version 4:   [  Δ A4   ] [          ] [  Δ C4   ]
  
  To reconstruct v4 of File A: Apply v1 + ΔA2 + ΔA4
  → SLOW for large histories
```

### Git: Snapshot-Based Storage

```
  GIT — Stores SNAPSHOTS (complete file states)
  
  Version 1:   [File A v1] [File B v1] [File C v1]
                    │            │            │
  Version 2:   [File A v2] [  ──────  ] [File C v2]
                    │       (unchanged,      │
  Version 3:   [  ──────  ]  stored as  [File C v3]
               (unchanged,   pointer)        │
  Version 4:   [File A v4]  to v1)     [File C v4]
  
  Unchanged files = pointer to previous version (not a copy)
  To get v4 of File A: Read snapshot directly
  → FAST — no reconstruction needed
```

**Key insight:** Git doesn't store diffs. It stores a snapshot of every file at every commit. Unchanged files are stored as references (pointers) to the previous identical version — extremely space-efficient.

---

## Content-Addressable Storage

Git uses **SHA-1 hashing** (transitioning to SHA-256) to identify every object:

```
Content  →  SHA-1 Hash  →  Storage Location

"Hello"  →  aaf4c61d...  →  .git/objects/aa/f4c61d...

┌──────────────────────────────────────────────────┐
│         CONTENT-ADDRESSABLE FILESYSTEM           │
│                                                  │
│    Content: "Hello, World!\n"                    │
│         │                                        │
│         ▼                                        │
│    SHA-1("blob 14\0Hello, World!\n")             │
│         │                                        │
│         ▼                                        │
│    Hash: 8ab686eafeb1f44702738c8b0f24f2567c36da6d│
│         │                                        │
│         ▼                                        │
│    Stored at: .git/objects/8a/b686ea...          │
│                                                  │
│    The CONTENT determines the ADDRESS            │
│    Same content = Same hash = Same object        │
│    → Automatic deduplication!                    │
└──────────────────────────────────────────────────┘
```

**Benefits:**
- **Data integrity** — If even one bit changes, the hash changes completely
- **Deduplication** — Identical files are stored only once
- **Verification** — Detect corruption instantly by recalculating hashes

---

## Git's Internal Architecture — DAG (Directed Acyclic Graph)

Git's commit history forms a **DAG** — a graph where:
- Each node is a commit
- Edges point from child to parent (backwards in time)
- No cycles are possible (you can't be your own ancestor)

```
┌──────────────────────────────────────────────────────────────┐
│                  GIT COMMIT DAG                              │
│                                                              │
│    A ◄── B ◄── C ◄── D ◄── E       (main)                  │
│                │                                             │
│                └──── F ◄── G ◄── H  (feature branch)        │
│                                                              │
│    Each commit points BACK to its parent(s)                  │
│    Merge commits have TWO parents:                           │
│                                                              │
│    A ◄── B ◄── C ◄── D ◄── E ◄── M  (main, after merge)   │
│                │                   ▲                         │
│                └──── F ◄── G ◄── H─┘  (merged feature)     │
│                                                              │
│    M has two parents: E and H                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Git's Performance Architecture

```
┌──────────────────────────────────────────────────────────┐
│            WHY GIT IS FAST                               │
│                                                          │
│  ┌─────────────────────────────────────────────┐         │
│  │ OPERATION        │ WHERE    │ NETWORK?      │         │
│  ├─────────────────────────────────────────────┤         │
│  │ git add          │ Local    │ No            │         │
│  │ git commit       │ Local    │ No            │         │
│  │ git log          │ Local    │ No            │         │
│  │ git diff         │ Local    │ No            │         │
│  │ git branch       │ Local    │ No            │         │
│  │ git checkout     │ Local    │ No            │         │
│  │ git merge        │ Local    │ No            │         │
│  │ git stash        │ Local    │ No            │         │
│  │ git rebase       │ Local    │ No            │         │
│  ├─────────────────────────────────────────────┤         │
│  │ git push         │ Remote   │ YES           │         │
│  │ git pull         │ Remote   │ YES           │         │
│  │ git fetch        │ Remote   │ YES           │         │
│  │ git clone        │ Remote   │ YES           │         │
│  └─────────────────────────────────────────────┘         │
│                                                          │
│  ~95% of daily operations are LOCAL = INSTANT            │
└──────────────────────────────────────────────────────────┘
```

---

## The .git Directory In Detail

```bash
# Explore Git's internals
$ ls .git/

# See what HEAD points to
$ cat .git/HEAD
ref: refs/heads/main

# See the current branch's latest commit
$ cat .git/refs/heads/main
a1b2c3d4e5f6789012345678901234567890abcd

# Count objects in your repo
$ git count-objects -v

# Verify repository integrity
$ git fsck
```

---

## Real-World Application: Understanding Architecture Helps Debugging

### Why "detached HEAD" happens

```
Normal State:
  HEAD → refs/heads/main → commit abc123

  HEAD ──▶ main ──▶ [abc123]

Detached HEAD State:
  HEAD → commit abc123 directly (no branch reference)

  HEAD ──▶ [abc123]    main ──▶ [abc123]
  
  (HEAD is "detached" from any branch)
```

```bash
# This creates a detached HEAD:
$ git checkout abc1234    # Checking out a specific commit

# Fix: create a branch at the current position
$ git checkout -b my-new-branch
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `.git` directory accidentally deleted | If you have a remote, `git clone` again. If local only, the repo is lost. |
| Repo seems corrupted | Run `git fsck` to check integrity |
| Repository is huge | Run `git gc` to garbage collect and compress |
| Want to see what Git is doing internally | Use `GIT_TRACE=1 git <command>` |
| Need to understand an object | Use `git cat-file -t <hash>` (type) and `git cat-file -p <hash>` (content) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Working Directory** | The actual files you see and edit |
| **Staging Area (Index)** | Preparation area for the next commit |
| **Local Repository** | `.git/` directory with complete history |
| **Remote Repository** | Shared repo on a server (GitHub, GitLab) |
| **Snapshot Storage** | Git stores full snapshots, not deltas |
| **SHA-1 Hash** | Unique identifier for every Git object |
| **Content-Addressable** | Content determines storage address |
| **DAG** | Directed Acyclic Graph — commit history structure |
| **HEAD** | Pointer to the current branch/commit |
| **refs** | Named pointers to commits (branches, tags) |

---

## Quick Revision Questions

1. **Why did Linus Torvalds create Git?**
   <details><summary>Answer</summary>BitKeeper, the proprietary DVCS used by the Linux kernel project, revoked its free license in 2005. Torvalds created Git to meet the Linux kernel's needs: speed, distributed architecture, and support for non-linear development.</details>

2. **What are the three main areas in Git's architecture?**
   <details><summary>Answer</summary>Working Directory (where you edit files), Staging Area/Index (where you prepare the next commit), and the Local Repository (.git directory, where committed history is stored).</details>

3. **How does Git's storage model differ from SVN's?**
   <details><summary>Answer</summary>SVN stores deltas (differences between versions). Git stores snapshots (complete file states), with unchanged files stored as pointers to previous versions. This makes Git faster at reconstructing any version.</details>

4. **What is content-addressable storage and why is it important?**
   <details><summary>Answer</summary>The content itself determines its storage address (via SHA-1 hash). This ensures data integrity (corruption is detectable), enables deduplication (identical content stored once), and provides unique identification for every object.</details>

5. **What is a DAG and how does Git use it?**
   <details><summary>Answer</summary>A Directed Acyclic Graph. Git uses it to represent commit history — each commit points to its parent(s), edges go backward in time, and cycles are impossible. Merge commits have two parents.</details>

6. **Which Git operations require network access and which are local?**
   <details><summary>Answer</summary>Network: push, pull, fetch, clone. Local: add, commit, log, diff, branch, checkout, merge, stash, rebase — roughly 95% of daily operations are local.</details>

---

[← Previous: Centralized vs Distributed VCS](02-centralized-vs-distributed.md) | [Back to README](../README.md) | [Next: Git Objects →](04-git-objects.md)
