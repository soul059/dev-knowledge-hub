# Chapter 1.2: Centralized vs Distributed VCS

[← Previous: What is Version Control?](01-what-is-version-control.md) | [Back to README](../README.md) | [Next: Git History and Architecture →](03-git-history-and-architecture.md)

---

## Overview

Understanding the architectural differences between Centralized and Distributed version control systems is essential for appreciating **why Git was designed the way it was** and why it dominates modern software development.

---

## Centralized Version Control Systems (CVCS)

### Architecture

```
┌──────────────────────────────────────────────────────────┐
│              CENTRALIZED VCS ARCHITECTURE                │
│                                                          │
│                  ┌──────────────┐                        │
│                  │   CENTRAL    │                        │
│                  │   SERVER     │                        │
│                  │              │                        │
│                  │ ┌──────────┐ │                        │
│                  │ │ Complete │ │                        │
│                  │ │ History  │ │                        │
│                  │ │ + Files  │ │                        │
│                  │ └──────────┘ │                        │
│                  └──────┬───────┘                        │
│                         │                                │
│            ┌────────────┼────────────┐                   │
│            │            │            │                   │
│       ┌────▼────┐  ┌────▼────┐  ┌────▼────┐             │
│       │ Dev A   │  │ Dev B   │  │ Dev C   │             │
│       │         │  │         │  │         │             │
│       │ Working │  │ Working │  │ Working │             │
│       │  Copy   │  │  Copy   │  │  Copy   │             │
│       │ (latest │  │ (latest │  │ (latest │             │
│       │  only)  │  │  only)  │  │  only)  │             │
│       └─────────┘  └─────────┘  └─────────┘             │
│                                                          │
│  Developers have ONLY the latest version of files        │
│  ALL operations require network access to the server     │
└──────────────────────────────────────────────────────────┘
```

### How CVCS Works

1. Developer **checks out** (downloads) the latest files from the central server
2. Makes changes locally in their **working copy**
3. **Commits** changes back to the central server
4. Other developers **update** to get the latest changes

### Examples of CVCS
- **SVN (Subversion)** — Most popular CVCS, still used in some enterprises
- **CVS (Concurrent Versions System)** — One of the earliest; largely obsolete
- **Perforce (Helix Core)** — Used in game development for large binary files

### CVCS Workflow

```
  Developer A                Server              Developer B
  ──────────                ────────             ──────────
       │                       │                      │
       │──── checkout ────────▶│                      │
       │◀─── files ───────────│                      │
       │                       │                      │
       │   (edit files)        │                      │
       │                       │                      │
       │──── commit ──────────▶│                      │
       │                       │                      │
       │                       │◀──── update ─────────│
       │                       │───── files ─────────▶│
       │                       │                      │
       │                       │       (edit files)   │
       │                       │                      │
       │                       │◀──── commit ─────────│
       │                       │                      │
```

---

## Distributed Version Control Systems (DVCS)

### Architecture

```
┌──────────────────────────────────────────────────────────┐
│              DISTRIBUTED VCS ARCHITECTURE                │
│                                                          │
│                  ┌──────────────┐                        │
│                  │   REMOTE     │                        │
│                  │   SERVER     │                        │
│                  │ (GitHub etc) │                        │
│                  │ ┌──────────┐ │                        │
│                  │ │ Complete │ │                        │
│                  │ │   Repo   │ │                        │
│                  │ └──────────┘ │                        │
│                  └──────┬───────┘                        │
│                         │                                │
│            ┌────────────┼────────────┐                   │
│            │            │            │                   │
│       ┌────▼────┐  ┌────▼────┐  ┌────▼────┐             │
│       │ Dev A   │  │ Dev B   │  │ Dev C   │             │
│       │         │  │         │  │         │             │
│       │Complete │  │Complete │  │Complete │             │
│       │  Repo   │  │  Repo   │  │  Repo   │             │
│       │ + Full  │  │ + Full  │  │ + Full  │             │
│       │ History │  │ History │  │ History │             │
│       └─────────┘  └─────────┘  └─────────┘             │
│                                                          │
│  Every developer has a COMPLETE copy of the repository   │
│  Most operations work OFFLINE (no network needed)        │
└──────────────────────────────────────────────────────────┘
```

### How DVCS Works

1. Developer **clones** (downloads the entire repository including full history)
2. Makes changes and **commits locally** — no network needed
3. **Pushes** changes to a shared remote when ready
4. **Pulls** changes from the remote to stay in sync

### Examples of DVCS
- **Git** — By far the most popular VCS in the world
- **Mercurial (Hg)** — Simpler alternative, used by Facebook (now Meta)
- **Bazaar** — Canonical's VCS; largely deprecated

### DVCS Workflow

```
  Developer A              Remote              Developer B
  ──────────              ────────             ──────────
       │                     │                      │
       │──── clone ─────────▶│                      │
       │◀─── full repo ──────│                      │
       │                     │◀──── clone ──────────│
       │                     │───── full repo ─────▶│
       │                     │                      │
       │  (commit locally)   │                      │
       │  (commit locally)   │                      │
       │  (commit locally)   │     (commit locally) │
       │                     │                      │
       │──── push ──────────▶│                      │
       │                     │                      │
       │                     │◀──── pull ───────────│
       │                     │───── changes ───────▶│
       │                     │                      │
```

---

## Head-to-Head Comparison

```
┌──────────────────────────────────────────────────────────────┐
│           CVCS vs DVCS — SIDE BY SIDE                       │
│                                                              │
│   CENTRALIZED (SVN)          │    DISTRIBUTED (Git)          │
│                              │                               │
│   Server ◄──► Client         │    Peer ◄──► Peer             │
│                              │                               │
│   ┌────────┐                 │    ┌────────┐   ┌────────┐   │
│   │ Server │                 │    │ Repo A │◄─▶│ Repo B │   │
│   │(master)│                 │    │ (full) │   │ (full) │   │
│   └───┬────┘                 │    └───┬────┘   └───┬────┘   │
│       │                      │        │            │        │
│   ┌───▼────┐                 │    ┌───▼────┐   ┌───▼────┐   │
│   │Partial │                 │    │ Repo C │◄─▶│ Remote │   │
│   │ Copies │                 │    │ (full) │   │ (full) │   │
│   └────────┘                 │    └────────┘   └────────┘   │
│                              │                               │
│   Single point of failure    │    No single point of failure │
│   Network required           │    Works offline              │
└──────────────────────────────────────────────────────────────┘
```

### Detailed Comparison Table

| Feature | Centralized (SVN) | Distributed (Git) |
|---------|-------------------|-------------------|
| **Repository Copy** | Only server has full history | Every developer has full history |
| **Offline Work** | ❌ Most operations need network | ✅ Commit, branch, log — all offline |
| **Speed** | Slower (network round-trips) | Faster (local operations) |
| **Branching** | Expensive, heavyweight | Cheap, lightweight |
| **Single Point of Failure** | ❌ Server goes down = no work | ✅ Any clone can restore the project |
| **Backup** | Must backup server separately | Every clone is a full backup |
| **Learning Curve** | Simpler mental model | Steeper initial learning curve |
| **Permission Control** | Easier (centralized) | More complex (needs platform tools) |
| **Large Binary Files** | Better (Perforce especially) | Needs Git LFS for large files |
| **Merging** | Merge tracking added late (SVN 1.5+) | First-class merge support built-in |
| **Atomic Commits** | Entire commit or nothing | Entire commit or nothing |
| **Revision Numbers** | Sequential (r1, r2, r3...) | SHA-1 hashes (a1b2c3d...) |

---

## Why Git Won

```
┌─────────────────────────────────────────────────────────┐
│              WHY GIT DOMINATES TODAY                     │
│                                                         │
│  1. SPEED                                               │
│     └─ Local operations = near-instant                  │
│                                                         │
│  2. BRANCHING                                           │
│     └─ Branches are cheap pointers, not file copies     │
│                                                         │
│  3. DISTRIBUTED                                         │
│     └─ Work anywhere, sync when ready                   │
│                                                         │
│  4. DATA INTEGRITY                                      │
│     └─ SHA-1 hash ensures nothing is corrupted          │
│                                                         │
│  5. ECOSYSTEM                                           │
│     └─ GitHub, GitLab, Bitbucket, CI/CD integration     │
│                                                         │
│  6. COMMUNITY                                           │
│     └─ Massive adoption → massive tooling support       │
│                                                         │
│  7. FLEXIBILITY                                         │
│     └─ Supports many different workflows                │
│                                                         │
│  Market Share (2025): Git > 95% of all VCS usage        │
└─────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Working on a Plane (No Internet)

```
CVCS (SVN):
  ❌ Can't commit — server unreachable
  ❌ Can't view history — stored on server
  ❌ Can't create branches — server operation
  → You can only edit files and hope for the best

DVCS (Git):
  ✅ Commit freely — everything is local
  ✅ View full history — it's on your machine
  ✅ Create branches — instant local operation
  → Full productivity, sync when you land
```

### Scenario 2: Server Catastrophic Failure

```
CVCS (SVN):
  ❌ If server backup is old or missing, history is LOST
  ❌ All developers are blocked until server is restored
  → Potential data loss, complete work stoppage

DVCS (Git):
  ✅ Every developer's clone IS a full backup
  ✅ Set up a new remote, push from any clone
  ✅ Zero data loss, minimal disruption
  → Pick any clone, designate it as the new remote
```

### Scenario 3: Experimenting with a Risky Feature

```
CVCS (SVN):
  Creating a branch = copying entire directory tree on server
  → Slow, consumes server space, discourages branching
  → Developers often work directly on trunk (risky!)

DVCS (Git):
  Creating a branch = creating a 41-byte pointer file
  → Instant, costs nothing, encourages experimentation
  $ git checkout -b experiment/risky-feature
  → Try it, keep it or delete it
```

---

## When to Use CVCS vs DVCS

| Use Case | Recommended | Why |
|----------|-------------|-----|
| Software development (any language) | **Git (DVCS)** | Speed, branching, ecosystem |
| Game development (large binaries) | **Perforce (CVCS)** or Git + LFS | Binary file handling |
| Legacy enterprise systems | **SVN (CVCS)** | Already established, simpler permissions |
| Open-source projects | **Git (DVCS)** | GitHub/GitLab ecosystem, forking model |
| Solo projects | **Git (DVCS)** | Even alone, history + branching = invaluable |
| Regulated industries (strict audit) | Either, often **Git** | Git provides strong audit trail via hashes |

---

## Troubleshooting Tips

| Issue | Tip |
|-------|-----|
| Migrating from SVN to Git | Use `git svn clone` to convert SVN repos to Git |
| Team resists moving to Git | Start with a GUI (GitKraken, Sourcetree) to ease the transition |
| Large repo is slow to clone | Use `git clone --depth 1` for shallow clone |
| Need CVCS-like permissions in Git | Use GitLab/GitHub branch protection + CODEOWNERS |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **CVCS** | Centralized VCS — single server, clients have working copies only |
| **DVCS** | Distributed VCS — every user has a full repository clone |
| **SVN** | Most common CVCS, uses sequential revision numbers |
| **Git** | Most common DVCS, uses SHA-1 hashes, designed for speed |
| **Clone** | Create a full local copy of a remote repository (DVCS) |
| **Checkout** | Get a working copy from the server (CVCS terminology) |
| **Push/Pull** | Sync changes between local and remote repos (DVCS) |
| **Commit/Update** | Send/receive changes to/from central server (CVCS) |
| **Offline Work** | Possible in DVCS, not in CVCS |
| **Branching** | Cheap in DVCS (pointer), expensive in CVCS (copy) |

---

## Quick Revision Questions

1. **What is the fundamental architectural difference between CVCS and DVCS?**
   <details><summary>Answer</summary>In CVCS, only the central server has the complete history; clients have only working copies. In DVCS, every developer has a complete copy of the entire repository including full history.</details>

2. **Why can you work offline with Git but not with SVN?**
   <details><summary>Answer</summary>Git stores the complete repository locally, so operations like commit, log, diff, and branch are all local. SVN requires network access to the central server for most operations.</details>

3. **What happens if the central server fails in a CVCS vs a DVCS?**
   <details><summary>Answer</summary>CVCS: All work stops and history may be lost if backups are inadequate. DVCS: Any developer's clone can be used to restore the full repository since every clone contains the complete history.</details>

4. **Why is branching "cheap" in Git but "expensive" in SVN?**
   <details><summary>Answer</summary>In Git, a branch is just a lightweight pointer (41 bytes) to a commit. In SVN, creating a branch typically involves copying the entire directory tree on the server.</details>

5. **Name two scenarios where a CVCS might still be preferred over Git.**
   <details><summary>Answer</summary>1) Game development with very large binary assets (Perforce handles large files better). 2) Legacy enterprise environments where SVN is already established and migration cost is high.</details>

6. **What does the command `git svn clone` do and when would you use it?**
   <details><summary>Answer</summary>It converts an SVN repository into a Git repository, preserving commit history. You'd use it when migrating a project from SVN to Git.</details>

---

[← Previous: What is Version Control?](01-what-is-version-control.md) | [Back to README](../README.md) | [Next: Git History and Architecture →](03-git-history-and-architecture.md)
