# Chapter 6.1: File Concepts

## Overview

Files are the fundamental abstraction for storing information permanently. This chapter covers what files are, their attributes, operations, types, and how they provide a logical view of secondary storage.

---

## What is a File?

```
┌──────────────────────────────────────────────────────────────────┐
│                    WHAT IS A FILE?                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: A file is a named collection of related            │
│              information stored on secondary storage             │
│                                                                  │
│  Physical View (Disk):       Logical View (User):               │
│  ┌─────────────────────┐     ┌─────────────────────┐            │
│  │ ░░░▓▓▓░░░▓▓▓▓░░░░░ │     │                     │            │
│  │ ░▓▓▓░░░░▓▓░░░░▓▓░░ │     │     report.pdf     │            │
│  │ ░░░░▓▓▓░░░░░▓▓▓░░░ │     │     200 KB         │            │
│  │ ▓▓░░░░░▓▓▓░░░░░░▓▓ │     │     Created: 2024  │            │
│  │ ░░▓▓░░░░░░▓▓░░▓▓░░ │     │                     │            │
│  └─────────────────────┘     └─────────────────────┘            │
│  Scattered disk blocks       Single logical unit               │
│                                                                  │
│  The OS provides this abstraction!                              │
│                                                                  │
│  Smallest unit of storage that can be:                          │
│  • Named                                                        │
│  • Accessed                                                     │
│  • Protected                                                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File Attributes (Metadata)

```
┌──────────────────────────────────────────────────────────────────┐
│                    FILE ATTRIBUTES                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┬────────────────────────────────────────┐   │
│  │ Attribute       │ Description                             │   │
│  ├─────────────────┼────────────────────────────────────────┤   │
│  │ Name            │ Human-readable identifier               │   │
│  │ Identifier      │ Unique number (inode number)           │   │
│  │ Type            │ File type (text, binary, directory)    │   │
│  │ Location        │ Pointer to data blocks on disk         │   │
│  │ Size            │ Current size in bytes                  │   │
│  │ Protection      │ Access permissions (rwx)               │   │
│  │ Time/Date       │ Created, modified, accessed times      │   │
│  │ Owner           │ User who owns the file                 │   │
│  └─────────────────┴────────────────────────────────────────┘   │
│                                                                  │
│  Example (Unix):                                                 │
│  $ ls -l report.pdf                                             │
│  -rw-r--r-- 1 alice staff 204800 Nov 15 10:30 report.pdf       │
│  ▲▲▲▲▲▲▲▲▲▲ ▲ ▲▲▲▲▲ ▲▲▲▲▲ ▲▲▲▲▲▲ ▲▲▲▲▲▲▲▲▲▲▲▲ ▲▲▲▲▲▲▲▲▲▲       │
│  │          │ │     │     │      │            └─ Name           │
│  │          │ │     │     │      └─ Modification time          │
│  │          │ │     │     └─ Size (bytes)                      │
│  │          │ │     └─ Group                                   │
│  │          │ └─ Owner                                         │
│  │          └─ Link count                                      │
│  └─ Permissions (type + rwx for owner/group/others)            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File Operations

```
┌──────────────────────────────────────────────────────────────────┐
│                    FILE OPERATIONS                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Basic operations provided by OS:                                │
│                                                                  │
│  1. CREATE                                                       │
│     • Allocate space in file system                            │
│     • Create directory entry                                    │
│     int fd = creat("newfile.txt", 0644);                       │
│                                                                  │
│  2. OPEN                                                         │
│     • Locate file in directory                                  │
│     • Load metadata into memory                                 │
│     • Return file descriptor                                    │
│     int fd = open("file.txt", O_RDONLY);                       │
│                                                                  │
│  3. READ                                                         │
│     • Copy data from file to buffer                            │
│     int n = read(fd, buffer, 100);                             │
│                                                                  │
│  4. WRITE                                                        │
│     • Copy data from buffer to file                            │
│     int n = write(fd, buffer, 100);                            │
│                                                                  │
│  5. SEEK                                                         │
│     • Change current position in file                          │
│     lseek(fd, 0, SEEK_SET);  // Go to beginning               │
│                                                                  │
│  6. CLOSE                                                        │
│     • Free memory structures                                    │
│     • Flush buffers to disk                                    │
│     close(fd);                                                  │
│                                                                  │
│  7. DELETE                                                       │
│     • Remove directory entry                                    │
│     • Free disk space                                          │
│     unlink("file.txt");                                        │
│                                                                  │
│  8. TRUNCATE                                                     │
│     • Delete file contents but keep attributes                 │
│     truncate("file.txt", 0);                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Open File Table

```
┌──────────────────────────────────────────────────────────────────┐
│                   OPEN FILE TABLE                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When a file is opened, OS maintains information in tables:     │
│                                                                  │
│  Per-Process Table (File Descriptor Table):                     │
│  ┌────────────────────────────────────────┐                     │
│  │  FD   │ Pointer to System Table Entry  │                     │
│  ├───────┼────────────────────────────────┤                     │
│  │   0   │ → stdin                        │                     │
│  │   1   │ → stdout                       │                     │
│  │   2   │ → stderr                       │                     │
│  │   3   │ ──────┐                        │                     │
│  │   4   │ ─────┐│                        │                     │
│  └───────┴──────┼┼────────────────────────┘                     │
│                 ││                                               │
│  System-wide    ││                                               │
│  Open File      ↓↓                                               │
│  Table:   ┌──────────────────────────────────────┐              │
│           │ Entry │ Position │ Count │ → Inode   │              │
│           ├───────┼──────────┼───────┼───────────┤              │
│           │   0   │    0     │   1   │ → file A  │              │
│           │   1   │   500    │   2   │ → file B  │ ← shared    │
│           │   2   │   200    │   1   │ → file C  │              │
│           └───────┴──────────┴───────┴───────────┘              │
│                                   ↑                              │
│                              Open count                          │
│                              (# of processes)                    │
│                                                                  │
│  Information tracked:                                            │
│  • File pointer (current position)                              │
│  • File open count                                              │
│  • Disk location of file                                        │
│  • Access rights                                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File Types

```
┌──────────────────────────────────────────────────────────────────┐
│                      FILE TYPES                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Method 1: Magic Number (Unix)                                   │
│  • First few bytes identify file type                           │
│  • Examples:                                                     │
│    - PDF: %PDF (0x25 50 44 46)                                  │
│    - ELF: 0x7F 45 4C 46                                         │
│    - JPEG: 0xFF D8 FF                                           │
│                                                                  │
│  Method 2: Extension (Windows, common convention)                │
│  ┌────────────────┬──────────────────────────────┐              │
│  │ Extension      │ Type                          │              │
│  ├────────────────┼──────────────────────────────┤              │
│  │ .txt           │ Plain text                    │              │
│  │ .pdf           │ PDF document                  │              │
│  │ .jpg, .png     │ Image                         │              │
│  │ .exe           │ Windows executable            │              │
│  │ .c, .java      │ Source code                   │              │
│  │ .mp3, .wav     │ Audio                         │              │
│  │ .zip, .tar     │ Archive                       │              │
│  └────────────────┴──────────────────────────────┘              │
│                                                                  │
│  Common categories:                                              │
│  • Regular files: Data (text or binary)                         │
│  • Directory files: List of file names                          │
│  • Special files: Devices (character, block)                    │
│  • Symbolic links: Pointers to other files                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                    FILE STRUCTURE                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. NO STRUCTURE (Sequence of bytes):                           │
│     • Unix/Linux approach                                       │
│     • OS sees file as stream of bytes                          │
│     • Application interprets structure                          │
│                                                                  │
│     ┌─────────────────────────────────────────────┐             │
│     │ B1 B2 B3 B4 B5 B6 B7 B8 B9 B10 ... Bn      │             │
│     └─────────────────────────────────────────────┘             │
│                                                                  │
│  2. RECORD STRUCTURE (Sequence of records):                     │
│     • Fixed or variable length records                          │
│     • Used in older systems, databases                          │
│                                                                  │
│     ┌──────────┬──────────┬──────────┬──────────┐               │
│     │ Record 1 │ Record 2 │ Record 3 │ Record 4 │               │
│     └──────────┴──────────┴──────────┴──────────┘               │
│                                                                  │
│  3. COMPLEX STRUCTURE (Tree, indexed):                          │
│     • OS understands internal structure                         │
│     • Example: Indexed files, B-trees                           │
│                                                                  │
│              ┌─────┐                                            │
│              │Root │                                            │
│              └──┬──┘                                            │
│           ┌────┴────┐                                           │
│        ┌──┴──┐   ┌──┴──┐                                       │
│        │Node │   │Node │                                        │
│        └─────┘   └─────┘                                        │
│                                                                  │
│  Modern systems: OS provides byte stream, applications          │
│                  can layer any structure on top                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Access Methods

```
┌──────────────────────────────────────────────────────────────────┐
│                    ACCESS METHODS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. SEQUENTIAL ACCESS                                            │
│     • Read/write in order                                       │
│     • Most common method                                        │
│     • Like a tape                                               │
│                                                                  │
│     ┌───┬───┬───┬───┬───┬───┬───┬───┐                          │
│     │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │                          │
│     └───┴───┴───┴───┴─▲─┴───┴───┴───┘                          │
│                       │                                         │
│                   Current                                       │
│                   Position                                      │
│                                                                  │
│     Operations:                                                  │
│     read next      → read and advance                          │
│     write next     → write and advance                         │
│     reset          → go to beginning                           │
│                                                                  │
│  2. DIRECT (RANDOM) ACCESS                                       │
│     • Jump to any position                                      │
│     • File viewed as numbered blocks                            │
│     • Like a disk                                               │
│                                                                  │
│     ┌───┬───┬───┬───┬───┬───┬───┬───┐                          │
│     │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │                          │
│     └───┴───┴───┴───┴───┴───┴───┴───┘                          │
│       ↑               ↑           ↑                             │
│     Can jump to any block directly                              │
│                                                                  │
│     Operations:                                                  │
│     read n         → read block n                              │
│     write n        → write block n                             │
│     seek(n)        → position at block n                       │
│                                                                  │
│  3. INDEXED ACCESS                                               │
│     • Use index to find records                                 │
│     • Built on top of direct access                            │
│     • Used in databases                                         │
│                                                                  │
│     Index:                    Data File:                        │
│     ┌─────┬─────┐            ┌────────────────┐                │
│     │Key  │Block│            │                │                │
│     ├─────┼─────┤            │                │                │
│     │"Bob"│  3  │───────────→│ Bob's record   │                │
│     │"Sue"│  7  │            │                │                │
│     │"Tom"│  1  │            │                │                │
│     └─────┴─────┘            └────────────────┘                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File Locking

```
┌──────────────────────────────────────────────────────────────────┐
│                      FILE LOCKING                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Purpose: Prevent simultaneous access conflicts                 │
│                                                                  │
│  Types:                                                          │
│  1. SHARED LOCK (Read lock)                                     │
│     • Multiple processes can hold simultaneously               │
│     • Used for reading                                          │
│                                                                  │
│  2. EXCLUSIVE LOCK (Write lock)                                 │
│     • Only one process can hold                                 │
│     • Used for writing                                          │
│                                                                  │
│  Lock Compatibility:                                             │
│  ┌──────────────────┬─────────┬───────────┐                     │
│  │ Existing \ New   │ Shared  │ Exclusive │                     │
│  ├──────────────────┼─────────┼───────────┤                     │
│  │ None             │   OK    │    OK     │                     │
│  │ Shared           │   OK    │   WAIT    │                     │
│  │ Exclusive        │  WAIT   │   WAIT    │                     │
│  └──────────────────┴─────────┴───────────┘                     │
│                                                                  │
│  Approaches:                                                     │
│  • MANDATORY locking: OS enforces, all access blocked          │
│  • ADVISORY locking: Process must check, can bypass            │
│                                                                  │
│  Example (Linux):                                                │
│  flock(fd, LOCK_SH);  // Shared lock                           │
│  flock(fd, LOCK_EX);  // Exclusive lock                        │
│  flock(fd, LOCK_UN);  // Unlock                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Internal vs External Fragmentation

```
┌──────────────────────────────────────────────────────────────────┐
│            FILE SYSTEM FRAGMENTATION                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INTERNAL FRAGMENTATION:                                         │
│  • Wasted space inside allocated blocks                         │
│  • File uses less than allocated block size                    │
│                                                                  │
│  Block = 4KB, File = 5KB                                        │
│  ┌────────────────┬────────────────┐                            │
│  │ Block 1: 4KB   │ Block 2: 4KB   │                            │
│  │ ████████████   │ ████░░░░░░░░░  │                            │
│  │ (used)         │ used  wasted   │                            │
│  └────────────────┴────────────────┘                            │
│  Wasted: 3KB (internal fragmentation)                           │
│                                                                  │
│  EXTERNAL FRAGMENTATION:                                         │
│  • Scattered free blocks across disk                            │
│  • Files become non-contiguous                                  │
│                                                                  │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐                     │
│  │ A │ B │   │ C │ A │   │ B │   │ C │ A │                     │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘                     │
│        ↑           ↑       ↑                                    │
│       Free blocks scattered, can't allocate large file         │
│                                                                  │
│  Solution: Disk defragmentation                                  │
│  • Rearrange blocks to make files contiguous                   │
│  • Slow process, done periodically                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **File** | Named collection of related data |
| **Attributes** | Metadata: name, size, type, permissions |
| **File Descriptor** | Handle returned by open() |
| **Open File Table** | System tracks all open files |
| **Sequential Access** | Read/write in order |
| **Direct Access** | Jump to any position |
| **Indexed Access** | Use index to locate records |
| **Shared Lock** | Multiple readers allowed |
| **Exclusive Lock** | Single writer only |

---

## Quick Revision Questions

1. What information is stored in file attributes?
2. Explain the difference between per-process and system-wide open file tables.
3. What is the difference between sequential and direct file access?
4. How do shared and exclusive locks differ?
5. What causes internal vs external fragmentation?

---

[← Previous: Thrashing](../05-Memory-Management/06-thrashing.md) | [Next: Directory Structure →](./02-directory-structure.md)
