# Chapter 6.2: Directory Structure

## Overview

Directories organize files into a hierarchical structure, making it easier to locate and manage files. This chapter explores different directory structures, from simple single-level to complex graph structures.

---

## What is a Directory?

```
┌──────────────────────────────────────────────────────────────────┐
│                   WHAT IS A DIRECTORY?                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  A directory is a special file that contains information        │
│  about other files (directory entries).                         │
│                                                                  │
│  Directory Entry contains:                                       │
│  ┌─────────────────┬────────────────────────────┐               │
│  │ File Name       │ Attributes or Pointer      │               │
│  ├─────────────────┼────────────────────────────┤               │
│  │ report.txt      │ → inode 42                 │               │
│  │ photo.jpg       │ → inode 67                 │               │
│  │ Documents       │ → inode 89 (directory)     │               │
│  └─────────────────┴────────────────────────────┘               │
│                                                                  │
│  Operations on directories:                                      │
│  • Search for a file                                            │
│  • Create a file                                                │
│  • Delete a file                                                │
│  • List directory contents                                      │
│  • Rename a file                                                │
│  • Traverse the file system                                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Single-Level Directory

```
┌──────────────────────────────────────────────────────────────────┐
│                 SINGLE-LEVEL DIRECTORY                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  All files in one directory                                      │
│                                                                  │
│                    ┌───────────────┐                             │
│                    │   Directory   │                             │
│                    └───────┬───────┘                             │
│        ┌──────┬──────┬─────┼─────┬──────┬──────┐                │
│        ▼      ▼      ▼     ▼     ▼      ▼      ▼                │
│      ┌───┐  ┌───┐  ┌───┐ ┌───┐ ┌───┐  ┌───┐  ┌───┐             │
│      │cat│  │bo │  │a  │ │tes│ │data│  │mail│  │hex│             │
│      └───┘  └───┘  └───┘ └───┘ └───┘  └───┘  └───┘             │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Simple to implement                                          │
│  ✓ Easy to understand                                           │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ Naming collisions (all names must be unique)                │
│  ✗ No grouping (files from different users mixed)              │
│  ✗ Difficult to manage many files                              │
│  ✗ Poor scalability                                            │
│                                                                  │
│  Example: Early batch systems, simple embedded systems          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Two-Level Directory

```
┌──────────────────────────────────────────────────────────────────┐
│                  TWO-LEVEL DIRECTORY                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Separate directory for each user                               │
│                                                                  │
│                  ┌──────────────────────┐                        │
│                  │  Master File         │                        │
│                  │  Directory (MFD)     │                        │
│                  └──────────┬───────────┘                        │
│           ┌─────────────────┼─────────────────┐                  │
│           ▼                 ▼                 ▼                  │
│     ┌──────────┐     ┌──────────┐     ┌──────────┐              │
│     │  User 1  │     │  User 2  │     │  User 3  │              │
│     │   UFD    │     │   UFD    │     │   UFD    │              │
│     └────┬─────┘     └────┬─────┘     └────┬─────┘              │
│     ┌────┴────┐      ┌────┴────┐      ┌────┴────┐               │
│     ▼    ▼    ▼      ▼    ▼    ▼      ▼    ▼    ▼               │
│    cat  bo   a     test data mail    a   hex  cat               │
│                                             ↑                    │
│                              Same name OK (different users)      │
│                                                                  │
│  Path: /user1/cat  vs  /user3/cat                               │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Users can have same file names                              │
│  ✓ Better isolation between users                              │
│  ✓ Efficient searching (search only user's directory)          │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ No grouping within user's files                             │
│  ✗ Sharing between users is complicated                        │
│  ✗ Still limited organization                                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Tree-Structured Directory

```
┌──────────────────────────────────────────────────────────────────┐
│               TREE-STRUCTURED DIRECTORY                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Generalization of two-level: directories within directories    │
│  Most common structure (Unix, Windows)                           │
│                                                                  │
│                          ┌─────┐                                 │
│                          │  /  │ (root)                          │
│                          └──┬──┘                                 │
│           ┌─────────────────┼─────────────────┐                  │
│           ▼                 ▼                 ▼                  │
│       ┌──────┐         ┌──────┐         ┌──────┐                │
│       │ bin  │         │ home │         │ etc  │                │
│       └──┬───┘         └──┬───┘         └──┬───┘                │
│          │         ┌──────┴──────┐         │                    │
│      ┌───┴───┐     ▼             ▼     ┌───┴───┐                │
│      │       │  ┌──────┐    ┌──────┐   │       │                │
│      ls     cat │alice │    │ bob  │  passwd  hosts             │
│                 └──┬───┘    └──┬───┘                            │
│               ┌────┴────┐      │                                │
│               ▼         ▼      ▼                                │
│          ┌──────┐  ┌──────┐ ┌──────┐                            │
│          │ docs │  │code  │ │notes │                            │
│          └──┬───┘  └──┬───┘ └──────┘                            │
│             │         │                                         │
│          ┌──┴──┐   ┌──┴──┐                                      │
│       report.txt  main.c                                        │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Efficient searching                                          │
│  ✓ Grouping capability                                          │
│  ✓ Scalable                                                     │
│  ✓ Same name allowed in different directories                  │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ No sharing (each file in exactly one place)                 │
│  ✗ Longer path names                                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Path Names

```
┌──────────────────────────────────────────────────────────────────┐
│                      PATH NAMES                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ABSOLUTE PATH:                                                  │
│  • Starts from root directory                                   │
│  • Complete path specification                                  │
│  • Examples:                                                     │
│    Unix:    /home/alice/docs/report.txt                         │
│    Windows: C:\Users\Alice\Documents\report.txt                 │
│                                                                  │
│  RELATIVE PATH:                                                  │
│  • Starts from current (working) directory                      │
│  • Shorter, context-dependent                                   │
│                                                                  │
│  Current directory: /home/alice                                  │
│  ┌──────────────────────┬─────────────────────────────┐         │
│  │ Relative Path        │ Equivalent Absolute Path    │         │
│  ├──────────────────────┼─────────────────────────────┤         │
│  │ docs/report.txt      │ /home/alice/docs/report.txt │         │
│  │ ./code/main.c        │ /home/alice/code/main.c     │         │
│  │ ../bob/notes         │ /home/bob/notes             │         │
│  │ ../../etc/passwd     │ /etc/passwd                 │         │
│  └──────────────────────┴─────────────────────────────┘         │
│                                                                  │
│  Special entries:                                                │
│  • .   (dot)      = current directory                           │
│  • ..  (dot-dot)  = parent directory                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Acyclic-Graph Directory

```
┌──────────────────────────────────────────────────────────────────┐
│              ACYCLIC-GRAPH DIRECTORY                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Allows file/directory sharing without cycles                   │
│  Uses links (hard links or symbolic links)                      │
│                                                                  │
│              ┌─────┐                                             │
│              │  /  │                                             │
│              └──┬──┘                                             │
│        ┌────────┴────────┐                                      │
│        ▼                 ▼                                      │
│    ┌──────┐          ┌──────┐                                   │
│    │alice │          │ bob  │                                   │
│    └──┬───┘          └──┬───┘                                   │
│       │                 │                                        │
│   ┌───┴───┐         ┌───┴───┐                                   │
│   ▼       ▼         ▼       ▼                                   │
│ ┌────┐ ┌────┐    ┌────┐  ┌────┐                                │
│ │proj│ │docs│    │work│  │link│──┐                              │
│ └──┬─┘ └────┘    └────┘  └────┘  │                              │
│    │                              │                              │
│    ▼                              │                              │
│  ┌────────────────────────────────┘                             │
│  │ shared_file.txt │                                            │
│  └──────────────────┘                                           │
│                                                                  │
│  Two paths to same file:                                        │
│  /alice/proj/shared_file.txt                                    │
│  /bob/link/shared_file.txt                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Types of Links

```
┌──────────────────────────────────────────────────────────────────┐
│                     TYPES OF LINKS                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HARD LINK:                                                      │
│  • Multiple directory entries point to same inode              │
│  • File deleted only when all links removed                    │
│  • Cannot cross file systems                                    │
│  • Cannot link to directories (prevents cycles)                │
│                                                                  │
│  Directory A:         Directory B:                              │
│  ┌──────────────┐    ┌──────────────┐                          │
│  │file.txt → 42 │    │copy.txt → 42 │ (same inode!)            │
│  └──────────────┘    └──────────────┘                          │
│           │                 │                                   │
│           └────────┬────────┘                                   │
│                    ▼                                            │
│              ┌──────────┐                                       │
│              │ Inode 42 │                                       │
│              │ links: 2 │                                       │
│              └──────────┘                                       │
│                                                                  │
│  $ ln file.txt copy.txt                                         │
│                                                                  │
│  ─────────────────────────────────────────────────              │
│                                                                  │
│  SYMBOLIC (SOFT) LINK:                                          │
│  • Separate file containing path to target                     │
│  • Can cross file systems                                       │
│  • Can link to directories                                      │
│  • Becomes "dangling" if target deleted                        │
│                                                                  │
│  Directory:                                                      │
│  ┌─────────────────┐      ┌─────────────────────┐              │
│  │shortcut → inode │ ──→  │"/home/alice/file.txt"│              │
│  └─────────────────┘      └──────────┬──────────┘              │
│                                      │                          │
│                                      ▼                          │
│                              /home/alice/file.txt               │
│                                                                  │
│  $ ln -s /home/alice/file.txt shortcut                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. General Graph Directory

```
┌──────────────────────────────────────────────────────────────────┐
│               GENERAL GRAPH DIRECTORY                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Allows cycles (directories can link back to ancestors)         │
│                                                                  │
│              ┌─────┐                                             │
│              │  /  │                                             │
│              └──┬──┘                                             │
│        ┌────────┴────────┐                                      │
│        ▼                 ▼                                      │
│    ┌──────┐          ┌──────┐                                   │
│    │  A   │←─────────│  B   │                                   │
│    └──┬───┘          └──┬───┘                                   │
│       │                 │                                        │
│       ▼                 │                                        │
│    ┌──────┐             │                                        │
│    │  C   │─────────────┘ (creates cycle: /A/C → /B → /A)       │
│    └──────┘                                                      │
│                                                                  │
│  Problems with cycles:                                           │
│  • Infinite loops during traversal                              │
│  • Difficulty determining when to delete                        │
│  • Disk space calculation errors                                │
│                                                                  │
│  Solutions:                                                      │
│  1. Only allow links to files, not directories                 │
│  2. Use garbage collection                                      │
│  3. Limit search depth                                          │
│  4. Mark directories during traversal                           │
│                                                                  │
│  Most systems: AVOID cycles (use acyclic graphs)                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Directory Implementation

```
┌──────────────────────────────────────────────────────────────────┐
│              DIRECTORY IMPLEMENTATION                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. LINEAR LIST                                                  │
│     • Array of (filename, pointer) pairs                       │
│     • Simple but slow for large directories                    │
│                                                                  │
│     ┌─────────────┬─────────────┬─────────────┬─────┐          │
│     │ file1 → 42  │ file2 → 67  │ file3 → 89  │ ... │          │
│     └─────────────┴─────────────┴─────────────┴─────┘          │
│                                                                  │
│     Search: O(n) - linear scan                                  │
│     Insert: O(1) - add at end (or O(n) if sorted)              │
│     Delete: O(n) - find and remove                             │
│                                                                  │
│  2. HASH TABLE                                                   │
│     • Hash filename to get bucket                               │
│     • Fast lookup                                               │
│                                                                  │
│     hash("file.txt") = 3                                        │
│     ┌───┬───┬───┬───┬───┬───┬───┐                              │
│     │   │   │   │ ● │   │   │   │                              │
│     └───┴───┴───┴─┼─┴───┴───┴───┘                              │
│                   ▼                                             │
│            ┌──────────────┐                                     │
│            │file.txt → 42 │                                     │
│            └──────────────┘                                     │
│                                                                  │
│     Search: O(1) average                                        │
│     Collision handling needed                                   │
│                                                                  │
│  3. B-TREE                                                       │
│     • Balanced tree structure                                   │
│     • Good for very large directories                          │
│     • Used in modern file systems (NTFS, HFS+)                 │
│                                                                  │
│     Search: O(log n)                                            │
│     Insert: O(log n)                                            │
│     Delete: O(log n)                                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File System Mounting

```
┌──────────────────────────────────────────────────────────────────┐
│                  FILE SYSTEM MOUNTING                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Mount: Attach a file system to the directory tree              │
│                                                                  │
│  Before mounting:                                                │
│                                                                  │
│  Root FS:              USB Drive:                               │
│    /                     /                                       │
│   ├── home              ├── photos                              │
│   ├── etc               └── music                               │
│   └── mnt                                                        │
│       └── (empty)                                                │
│                                                                  │
│  $ mount /dev/sdb1 /mnt/usb                                     │
│                                                                  │
│  After mounting:                                                 │
│                                                                  │
│  Unified tree:                                                   │
│    /                                                             │
│   ├── home                                                       │
│   ├── etc                                                        │
│   └── mnt                                                        │
│       └── usb ←─────── Mount Point                              │
│           ├── photos                                             │
│           └── music                                              │
│                                                                  │
│  Mount table tracks:                                             │
│  ┌──────────────┬────────────────┬──────────────┐               │
│  │ Device       │ Mount Point    │ FS Type      │               │
│  ├──────────────┼────────────────┼──────────────┤               │
│  │ /dev/sda1    │ /              │ ext4         │               │
│  │ /dev/sdb1    │ /mnt/usb       │ vfat         │               │
│  │ /dev/sdc1    │ /home          │ ext4         │               │
│  └──────────────┴────────────────┴──────────────┘               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Comparison of Directory Structures

| Structure | Naming | Grouping | Sharing | Cycles | Complexity |
|-----------|--------|----------|---------|--------|------------|
| Single-Level | Unique | No | No | No | Simple |
| Two-Level | Per user | No | Hard | No | Simple |
| Tree | Per dir | Yes | No | No | Moderate |
| Acyclic Graph | Per dir | Yes | Links | No | Complex |
| General Graph | Per dir | Yes | Links | Yes | Very Complex |

---

## Summary

| Concept | Description |
|---------|-------------|
| **Directory** | File containing list of file names and pointers |
| **Single-Level** | All files in one directory |
| **Two-Level** | Separate directory per user |
| **Tree** | Hierarchical directories (most common) |
| **Acyclic Graph** | Tree with links (sharing) |
| **Hard Link** | Multiple names for same inode |
| **Symbolic Link** | File containing path to target |
| **Mount** | Attach file system to directory tree |
| **Absolute Path** | Path from root |
| **Relative Path** | Path from current directory |

---

## Quick Revision Questions

1. What are the disadvantages of a single-level directory?
2. Explain the difference between hard links and symbolic links.
3. Why are cycles problematic in directory structures?
4. What is a mount point?
5. Compare linear list vs hash table for directory implementation.

---

[← Previous: File Concepts](./01-file-concepts.md) | [Next: File Allocation Methods →](./03-file-allocation.md)
