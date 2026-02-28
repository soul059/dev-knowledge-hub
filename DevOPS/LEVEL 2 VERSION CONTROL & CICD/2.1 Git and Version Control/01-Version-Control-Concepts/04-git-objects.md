# Chapter 1.4: Git Objects (Blob, Tree, Commit, Tag)

[← Previous: Git History and Architecture](03-git-history-and-architecture.md) | [Back to README](../README.md) | [Next: Repository Initialization →](../02-Git-Basics/01-repository-initialization.md)

---

## Overview

Git's elegance comes from its surprisingly simple internal data model. Everything in Git is built on just **four types of objects**: blobs, trees, commits, and tags. Understanding these objects gives you mastery over Git's internals.

---

## The Four Git Objects

```
┌──────────────────────────────────────────────────────────────┐
│                   GIT OBJECT MODEL                          │
│                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │   BLOB   │   │   TREE   │   │  COMMIT  │   │   TAG    │ │
│  │          │   │          │   │          │   │          │ │
│  │  File    │   │Directory │   │ Snapshot │   │  Named   │ │
│  │ content  │   │ listing  │   │ metadata │   │  label   │ │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘ │
│                                                              │
│  Every object is identified by its SHA-1 hash               │
│  Every object is immutable (never changes once created)     │
│  Every object is stored in .git/objects/                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 1. Blob (Binary Large Object)

A blob stores the **contents** of a file — nothing else. No filename, no permissions, no metadata. Just raw content.

```
┌────────────────────────────────────────────┐
│                BLOB OBJECT                 │
│                                            │
│  Type: blob                                │
│  Content: (raw file content)               │
│                                            │
│  Example:                                  │
│  ┌──────────────────────────────────────┐  │
│  │ blob 26\0                           │  │
│  │ console.log("Hello, World!");       │  │
│  │                                      │  │
│  └──────────────────────────────────────┘  │
│                    │                       │
│                    ▼                       │
│  SHA-1: 3b18e512dba79e4c8300dd08aeb37f8e7  │
│                                            │
│  • Stores ONLY content (no filename!)      │
│  • Two files with same content = ONE blob  │
│  • Compressed with zlib                    │
└────────────────────────────────────────────┘
```

### Exploring Blobs

```bash
# Create a blob manually
$ echo "Hello, World!" | git hash-object --stdin -w
8ab686eafeb1f44702738c8b0f24f2567c36da6d

# Read the blob content
$ git cat-file -p 8ab686ea
Hello, World!

# Check the object type
$ git cat-file -t 8ab686ea
blob

# Check the object size
$ git cat-file -s 8ab686ea
14
```

### Key Properties of Blobs
- **Content-only** — The same content always produces the same hash
- **Deduplication** — If 10 files have identical content, only 1 blob is stored
- **No metadata** — Filename and permissions are stored in the tree object

---

## 2. Tree Object

A tree object represents a **directory**. It maps filenames to blobs (files) and other trees (subdirectories).

```
┌────────────────────────────────────────────────────────────┐
│                   TREE OBJECT                              │
│                                                            │
│  Represents: A directory listing                           │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  tree <size>\0                                       │  │
│  │                                                      │  │
│  │  100644 blob a1b2c3d  README.md                      │  │
│  │  100644 blob d4e5f6a  index.html                     │  │
│  │  100755 blob b7c8d9e  deploy.sh                      │  │
│  │  040000 tree f0a1b2c  src/                           │  │
│  │  040000 tree e3d4c5b  docs/                          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  Format: <mode> <type> <hash> <name>                      │
│                                                            │
│  Modes:                                                    │
│    100644 = regular file                                   │
│    100755 = executable file                                │
│    040000 = subdirectory (tree)                            │
│    120000 = symbolic link                                  │
│    160000 = git submodule                                  │
└────────────────────────────────────────────────────────────┘
```

### Tree Structure Example

```
  PROJECT DIRECTORY              GIT TREE STRUCTURE
  
  my-project/                    Tree (root)
  ├── README.md                  ├── blob → README.md
  ├── app.js                     ├── blob → app.js
  └── src/                       └── tree → src/
      ├── utils.js                   ├── blob → utils.js
      └── components/                └── tree → components/
          └── Header.js                  └── blob → Header.js
```

```
┌──────────────────────────────────────────────────────────┐
│         TREE OBJECT HIERARCHY (VISUAL)                   │
│                                                          │
│    Root Tree (abc123)                                    │
│    ├── blob (def456) ──▶ "README.md"                    │
│    ├── blob (789abc) ──▶ "app.js"                       │
│    └── tree (321fed) ──▶ "src/"                         │
│         ├── blob (654cba) ──▶ "utils.js"                │
│         └── tree (987654) ──▶ "components/"             │
│              └── blob (abcdef) ──▶ "Header.js"          │
│                                                          │
│    Trees contain blobs (files) and other trees (dirs)   │
└──────────────────────────────────────────────────────────┘
```

### Exploring Trees

```bash
# View the tree of the latest commit
$ git cat-file -p main^{tree}
100644 blob a1b2c3d4e5f6    README.md
100644 blob d4e5f6a7b8c9    app.js
040000 tree f0a1b2c3d4e5    src

# View a subtree
$ git cat-file -p f0a1b2c3d4e5
100644 blob abcdef123456    utils.js
040000 tree 987654abcdef    components

# List all files in a tree (recursively)
$ git ls-tree -r HEAD
```

---

## 3. Commit Object

A commit object ties everything together. It records a **snapshot** (tree) along with metadata: author, committer, date, message, and parent commit(s).

```
┌──────────────────────────────────────────────────────────┐
│                  COMMIT OBJECT                           │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  commit <size>\0                                   │  │
│  │                                                    │  │
│  │  tree      a1b2c3d4e5f6789012345678901234567890    │  │
│  │  parent    fedcba9876543210fedcba9876543210fedc    │  │
│  │  author    Alice <alice@example.com> 1740700800    │  │
│  │  committer Alice <alice@example.com> 1740700800    │  │
│  │                                                    │  │
│  │  Add user authentication feature                   │  │
│  │                                                    │  │
│  │  Implemented login/logout functionality with       │  │
│  │  session management and CSRF protection.           │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Components:                                             │
│  • tree    → points to the root tree (project snapshot)  │
│  • parent  → points to previous commit(s)               │
│  • author  → who wrote the changes                      │
│  • committer → who applied the commit                   │
│  • message → description of changes                     │
│                                                          │
│  NOTE: author ≠ committer when patches are applied       │
│  by someone other than the original author               │
└──────────────────────────────────────────────────────────┘
```

### Types of Commits by Parent Count

```
┌────────────────────────────────────────────────────────┐
│          COMMIT TYPES BY PARENT COUNT                  │
│                                                        │
│  ROOT COMMIT (0 parents):                             │
│  ┌────────┐                                           │
│  │Commit A│  ← First commit in the repo              │
│  │parent: │     (no parent)                           │
│  │ (none) │                                           │
│  └────────┘                                           │
│                                                        │
│  REGULAR COMMIT (1 parent):                           │
│  ┌────────┐    ┌────────┐                             │
│  │Commit A│◄───│Commit B│  ← Most common type        │
│  └────────┘    │parent:A│                             │
│                └────────┘                             │
│                                                        │
│  MERGE COMMIT (2+ parents):                           │
│  ┌────────┐    ┌────────┐                             │
│  │Commit E│◄───│Commit M│  ← Result of merge         │
│  └────────┘  ┌▶│parent: │                             │
│  ┌────────┐  │ │ E, H   │                             │
│  │Commit H│──┘ └────────┘                             │
│  └────────┘                                           │
└────────────────────────────────────────────────────────┘
```

### Exploring Commits

```bash
# View a commit object
$ git cat-file -p HEAD
tree 4b825dc642cb6eb9a060e54bf899d31e5baf3412
parent a1b2c3d4e5f6789012345678901234567890abcd
author Alice <alice@example.com> 1740700800 +0000
committer Alice <alice@example.com> 1740700800 +0000

Add user authentication feature

# See the commit log
$ git log --oneline -5

# See a commit with its diff
$ git show HEAD
```

---

## 4. Tag Object (Annotated Tag)

An annotated tag is a permanent label for a specific commit, often used for releases. It includes metadata about who created the tag and why.

```
┌────────────────────────────────────────────────────────┐
│                  TAG OBJECT                            │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  tag <size>\0                                    │  │
│  │                                                  │  │
│  │  object  a1b2c3d4e5f6789012345678901234567890   │  │
│  │  type    commit                                  │  │
│  │  tag     v2.0.0                                  │  │
│  │  tagger  Alice <alice@example.com> 1740700800   │  │
│  │                                                  │  │
│  │  Release version 2.0.0                           │  │
│  │                                                  │  │
│  │  Major features:                                 │  │
│  │  - User authentication                           │  │
│  │  - Payment processing                            │  │
│  │  - Dashboard redesign                            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  Lightweight tag = just a reference (NOT an object)    │
│  Annotated tag  = full object with metadata            │
└────────────────────────────────────────────────────────┘
```

### Lightweight vs Annotated Tags

```bash
# Lightweight tag — just a pointer (no object created)
$ git tag v1.0.0

# Annotated tag — creates a tag object
$ git tag -a v2.0.0 -m "Release version 2.0.0"

# View tag details
$ git cat-file -p v2.0.0

# List all tags
$ git tag -l
```

| Feature | Lightweight Tag | Annotated Tag |
|---------|----------------|---------------|
| Creates an object? | No — just a ref | Yes — full tag object |
| Has a message? | No | Yes |
| Has tagger info? | No | Yes (who, when) |
| Can be signed (GPG)? | No | Yes |
| Recommended for? | Temporary/private tags | Releases, public tags |

---

## How Objects Relate to Each Other

```
┌──────────────────────────────────────────────────────────────────┐
│              COMPLETE OBJECT RELATIONSHIP                       │
│                                                                  │
│  Tag ──▶ Commit ──▶ Tree ──▶ Blob                              │
│                       │                                          │
│                       └──▶ Tree ──▶ Blob                        │
│                             │                                    │
│                             └──▶ Blob                           │
│                                                                  │
│                                                                  │
│  Example: Tag "v1.0" pointing to a project                      │
│                                                                  │
│  ┌─────────┐    ┌──────────┐    ┌──────────────┐                │
│  │Tag v1.0 │───▶│ Commit   │───▶│ Tree (root)  │                │
│  │         │    │ abc123   │    │              │                │
│  └─────────┘    │parent:   │    │ README.md ──────▶ Blob (content│
│                 │ def456   │    │ app.js    ──────▶ Blob (content│
│                 └──────────┘    │ src/      ──────▶ Tree        │
│                      │         └──────────────┘    │           │
│                      ▼                             │           │
│                 ┌──────────┐              ┌────────▼──────┐    │
│                 │ Commit   │              │ Tree (src/)   │    │
│                 │ def456   │              │               │    │
│                 │(parent   │              │ index.js ────────▶Blob│
│                 │ commit)  │              │ style.css ───────▶Blob│
│                 └──────────┘              └───────────────┘    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Hands-On: Inspecting Git Objects

```bash
# Step 1: Initialize a test repo
$ mkdir git-objects-demo && cd git-objects-demo
$ git init

# Step 2: Create a file and commit
$ echo "Hello Git" > hello.txt
$ git add hello.txt
$ git commit -m "Initial commit"

# Step 3: Find the commit hash
$ git log --oneline
a1b2c3d (HEAD -> main) Initial commit

# Step 4: Inspect the commit object
$ git cat-file -p a1b2c3d
tree 68aba62e560c0ebc3396e8ae9335232cd93a3f60
author You <you@example.com> 1740700800 +0000
committer You <you@example.com> 1740700800 +0000

Initial commit

# Step 5: Inspect the tree object
$ git cat-file -p 68aba62e
100644 blob ce013625030ba8dba906f756967f9e9ca394464a    hello.txt

# Step 6: Inspect the blob object
$ git cat-file -p ce013625
Hello Git

# Step 7: See all objects
$ find .git/objects -type f
.git/objects/a1/b2c3d...   (commit)
.git/objects/68/aba62e...   (tree)
.git/objects/ce/013625...   (blob)
```

---

## Object Storage Format

```
┌────────────────────────────────────────────────────────────┐
│             HOW GIT STORES OBJECTS                         │
│                                                            │
│  1. Construct header:  "<type> <size>\0"                   │
│  2. Prepend header to content                              │
│  3. Calculate SHA-1 hash of (header + content)             │
│  4. Compress with zlib                                     │
│  5. Store in .git/objects/<first-2-chars>/<remaining>       │
│                                                            │
│  Example:                                                  │
│    Content: "Hello Git\n"                                  │
│    Header:  "blob 10\0"                                    │
│    Full:    "blob 10\0Hello Git\n"                         │
│    SHA-1:   ce013625030ba8dba906f756967f9e9ca394464a       │
│    Path:    .git/objects/ce/013625030ba8dba906f75696...    │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │ Content  │───▶│ SHA-1    │───▶│ .git/    │             │
│  │ + Header │    │  Hash    │    │ objects/ │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│                                                            │
│  Objects are IMMUTABLE — once written, never changed       │
└────────────────────────────────────────────────────────────┘
```

---

## Pack Files — Efficiency at Scale

When a repository grows large, Git **packs** objects for efficiency:

```
┌──────────────────────────────────────────────────────────┐
│                PACK FILES                                │
│                                                          │
│  Loose Objects (default):                                │
│  .git/objects/                                           │
│  ├── a1/b2c3d...    (each object = one file)            │
│  ├── ce/013625...                                        │
│  └── 68/aba62e...                                        │
│                                                          │
│  Packed Objects (git gc):                                │
│  .git/objects/pack/                                      │
│  ├── pack-abc123.pack   (all objects in one file)       │
│  └── pack-abc123.idx    (index for fast lookup)         │
│                                                          │
│  Pack files use delta compression:                       │
│  • Similar objects stored as deltas of each other        │
│  • Dramatically reduces disk space                       │
│  • Automatic on push/fetch, or manual with "git gc"     │
└──────────────────────────────────────────────────────────┘
```

```bash
# Manually trigger packing
$ git gc

# See pack statistics
$ git count-objects -v

# Verify pack integrity
$ git verify-pack -v .git/objects/pack/pack-*.idx
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Need to find a specific object | `git cat-file -t <hash>` (type), `git cat-file -p <hash>` (content) |
| Repo seems bloated | Run `git gc --aggressive` to repack objects |
| Want to find dangling objects | `git fsck --unreachable` |
| Object appears corrupted | `git fsck` will report corrupt objects |
| Need to verify a commit chain | `git log --graph --oneline` to visualize the DAG |

---

## Summary Table

| Object | Purpose | Contains | Created By |
|--------|---------|----------|------------|
| **Blob** | Store file content | Raw file content (no name) | `git add` |
| **Tree** | Represent a directory | List of blobs & trees with names/modes | `git commit` |
| **Commit** | Record a snapshot | Tree pointer, parent(s), author, message | `git commit` |
| **Tag** | Label a commit | Commit pointer, tagger, message | `git tag -a` |

| Property | Description |
|----------|-------------|
| **SHA-1 Hash** | 40-character hex string uniquely identifying each object |
| **Immutability** | Objects never change once written |
| **Content-Addressable** | Hash is derived from content |
| **Compression** | All objects are zlib-compressed |
| **Pack Files** | Multiple objects bundled for efficiency |
| **DAG Structure** | Commits form a directed acyclic graph via parent pointers |

---

## Quick Revision Questions

1. **What are the four types of Git objects?**
   <details><summary>Answer</summary>Blob (file content), Tree (directory listing), Commit (snapshot with metadata), and Tag (annotated label for a commit).</details>

2. **What information does a blob object store? What does it NOT store?**
   <details><summary>Answer</summary>A blob stores only the raw content of a file. It does NOT store the filename, path, or permissions — those are stored in the tree object that references the blob.</details>

3. **How does Git achieve file deduplication?**
   <details><summary>Answer</summary>Since blobs are identified by the SHA-1 hash of their content, two files with identical content produce the same hash and are stored as a single blob object — regardless of their filenames or locations.</details>

4. **What is the difference between a lightweight tag and an annotated tag?**
   <details><summary>Answer</summary>A lightweight tag is just a pointer/reference (no Git object created). An annotated tag creates a full tag object containing the tagger's name, date, message, and optionally a GPG signature.</details>

5. **How can you inspect the type and content of any Git object?**
   <details><summary>Answer</summary>Use `git cat-file -t <hash>` to see the type (blob/tree/commit/tag) and `git cat-file -p <hash>` to pretty-print the content of the object.</details>

6. **What is a merge commit and how does it differ structurally from a regular commit?**
   <details><summary>Answer</summary>A merge commit has two or more parent pointers (one for each branch being merged), while a regular commit has exactly one parent. The root commit has zero parents.</details>

---

[← Previous: Git History and Architecture](03-git-history-and-architecture.md) | [Back to README](../README.md) | [Next: Repository Initialization →](../02-Git-Basics/01-repository-initialization.md)
