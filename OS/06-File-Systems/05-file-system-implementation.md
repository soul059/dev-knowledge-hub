# Chapter 6.5: File System Implementation

## Overview

This chapter explores how file systems are implemented, including on-disk structures, in-memory structures, and the Virtual File System (VFS) layer that provides a uniform interface across different file system types.

---

## File System Layers

```
┌──────────────────────────────────────────────────────────────────┐
│                 FILE SYSTEM LAYERS                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Application                              │ │
│  │                     open(), read(), write()                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                Logical File System                          │ │
│  │    - File metadata (inodes, directories)                   │ │
│  │    - Protection and security                                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              File Organization Module                       │ │
│  │    - Logical blocks → Physical blocks                      │ │
│  │    - Free space management                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                 Basic File System                           │ │
│  │    - Issue commands to device driver                       │ │
│  │    - Manage memory buffers and caches                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                 I/O Control                                 │ │
│  │    - Device drivers                                         │ │
│  │    - Interrupt handlers                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Devices                                  │ │
│  │              (Hard disk, SSD, etc.)                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## On-Disk Structures

```
┌──────────────────────────────────────────────────────────────────┐
│                   ON-DISK STRUCTURES                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Disk Layout (typical Unix-like file system):                   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │Boot│Super│Inode │Data  │Inode │Data  │ ... │Inode │Data   ││
│  │Blk │Block│Bitmap│Bitmap│Table │Blocks│     │Table │Blocks ││
│  └────┴─────┴──────┴──────┴──────┴──────┴─────┴──────┴───────┘│
│  │←── Fixed ──→│←────────── Block Group 0 ──────────────────→││
│                                                                  │
│  1. BOOT BLOCK                                                   │
│     • First block (or sector)                                   │
│     • Contains bootstrap code                                   │
│     • Loads OS kernel                                           │
│                                                                  │
│  2. SUPERBLOCK                                                   │
│     • File system metadata                                      │
│     • Size, number of blocks, number of inodes                 │
│     • Block size, magic number                                  │
│     • Mount count, state                                        │
│                                                                  │
│  3. BITMAPS                                                      │
│     • Inode bitmap: which inodes are allocated                 │
│     • Data bitmap: which data blocks are allocated             │
│                                                                  │
│  4. INODE TABLE                                                  │
│     • Array of inode structures                                 │
│     • Fixed size per inode (e.g., 128 bytes)                   │
│                                                                  │
│  5. DATA BLOCKS                                                  │
│     • Actual file and directory contents                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Superblock

```
┌──────────────────────────────────────────────────────────────────┐
│                      SUPERBLOCK                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Contains critical file system information:                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Field                    │ Example Value                   │ │
│  ├──────────────────────────┼─────────────────────────────────┤ │
│  │ Magic number             │ 0xEF53 (ext2/3/4 signature)    │ │
│  │ Block size               │ 4096 bytes                      │ │
│  │ Total blocks             │ 10,000,000                      │ │
│  │ Free blocks              │ 5,432,100                       │ │
│  │ Total inodes             │ 2,500,000                       │ │
│  │ Free inodes              │ 2,100,000                       │ │
│  │ First data block         │ 1                               │ │
│  │ Blocks per group         │ 32,768                          │ │
│  │ Inodes per group         │ 8,192                           │ │
│  │ Mount count              │ 15                              │ │
│  │ Max mount count          │ 30                              │ │
│  │ Last mount time          │ 2024-01-15 10:30:00            │ │
│  │ Last write time          │ 2024-01-15 10:45:00            │ │
│  │ State                    │ Clean / Not clean              │ │
│  └──────────────────────────┴─────────────────────────────────┘ │
│                                                                  │
│  Superblock is CRITICAL:                                         │
│  • Corrupted superblock = unusable file system                  │
│  • Solution: Store backup copies in block groups               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## In-Memory Structures

```
┌──────────────────────────────────────────────────────────────────┐
│                   IN-MEMORY STRUCTURES                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When file system is mounted, OS loads structures into RAM:     │
│                                                                  │
│  1. MOUNT TABLE                                                  │
│     ┌─────────────────────────────────────────────────────────┐ │
│     │ Device    │ Mount Point │ FS Type │ Superblock Ptr     │ │
│     ├───────────┼─────────────┼─────────┼────────────────────┤ │
│     │ /dev/sda1 │ /           │ ext4    │ 0x12340000         │ │
│     │ /dev/sda2 │ /home       │ ext4    │ 0x12350000         │ │
│     │ /dev/sdb1 │ /mnt/usb    │ vfat    │ 0x12360000         │ │
│     └───────────┴─────────────┴─────────┴────────────────────┘ │
│                                                                  │
│  2. IN-MEMORY SUPERBLOCK                                         │
│     • Copy of on-disk superblock                                │
│     • Modified in memory, periodically synced to disk          │
│                                                                  │
│  3. IN-MEMORY INODES (Inode Cache)                              │
│     • Recently accessed inodes                                  │
│     • Faster than reading from disk each time                  │
│                                                                  │
│  4. BUFFER CACHE                                                 │
│     • Recently accessed data blocks                             │
│     • Write-back caching (dirty blocks)                        │
│                                                                  │
│  5. OPEN FILE TABLE (System-wide)                               │
│     • One entry per open file instance                         │
│     • File position, access mode, inode pointer                │
│                                                                  │
│  6. FILE DESCRIPTOR TABLE (Per-process)                         │
│     • Maps fd numbers to open file table entries               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Virtual File System (VFS)

```
┌──────────────────────────────────────────────────────────────────┐
│                VIRTUAL FILE SYSTEM (VFS)                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem: Different file systems have different implementations  │
│  Solution: VFS provides unified interface                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Applications                          │   │
│  │            open(), read(), write(), close()              │   │
│  └────────────────────────────┬─────────────────────────────┘   │
│                               │                                  │
│                               ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    VFS Layer                             │   │
│  │     Unified API: vfs_open, vfs_read, vfs_write          │   │
│  │     Common structures: vfs_inode, vfs_dentry            │   │
│  └──────┬──────────────┬──────────────┬─────────────────────┘   │
│         │              │              │                          │
│         ▼              ▼              ▼                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐     │
│  │  ext4    │   │  NTFS    │   │   NFS    │   │   FAT    │     │
│  │ driver   │   │ driver   │   │ driver   │   │ driver   │     │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘     │
│       │              │              │              │             │
│       ▼              ▼              ▼              ▼             │
│   Local Disk    Local Disk    Network       USB Drive           │
│                                                                  │
│  VFS allows:                                                     │
│  • Same system calls work on any file system                   │
│  • Easy to add new file system types                           │
│  • Network file systems appear local                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### VFS Objects

```
┌──────────────────────────────────────────────────────────────────┐
│                     VFS OBJECTS                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. SUPERBLOCK OBJECT                                            │
│     • Represents a mounted file system                          │
│     • Operations: alloc_inode, destroy_inode, sync_fs          │
│                                                                  │
│  2. INODE OBJECT                                                 │
│     • Represents a specific file                                │
│     • Operations: create, lookup, link, unlink, mkdir          │
│                                                                  │
│  3. DENTRY OBJECT (Directory Entry)                              │
│     • Maps name to inode                                        │
│     • Cached for fast path lookups                              │
│     • Operations: compare, hash, delete                         │
│                                                                  │
│  4. FILE OBJECT                                                  │
│     • Represents an open file                                   │
│     • Operations: read, write, llseek, mmap, flush             │
│                                                                  │
│  Each object has:                                                │
│  • Data fields (file-system specific data)                     │
│  • Operations table (function pointers)                        │
│                                                                  │
│  Example: file_operations                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ struct file_operations {                                │    │
│  │     ssize_t (*read)(struct file *, char *, size_t);    │    │
│  │     ssize_t (*write)(struct file *, char *, size_t);   │    │
│  │     int (*open)(struct inode *, struct file *);        │    │
│  │     int (*release)(struct inode *, struct file *);     │    │
│  │     ...                                                 │    │
│  │ };                                                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Directory Implementation

```
┌──────────────────────────────────────────────────────────────────┐
│               DIRECTORY IMPLEMENTATION                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  A directory is a special file containing name→inode mappings   │
│                                                                  │
│  Simple approach (linear list):                                  │
│  ┌───────────────────────────────────────────────────────┐      │
│  │ Length │ Name      │ Inode │ Length │ Name    │ Inode │      │
│  ├────────┼───────────┼───────┼────────┼─────────┼───────┤      │
│  │   6    │ "file1"   │  42   │   4    │ "dir"  │  67   │      │
│  └────────┴───────────┴───────┴────────┴─────────┴───────┘      │
│                                                                  │
│  Problem: Linear search O(n) for each lookup                    │
│                                                                  │
│  Better: Hash table or B-tree                                    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              B-TREE DIRECTORY                            │   │
│  │                                                          │   │
│  │                    [d-m]                                 │   │
│  │                   /     \                                │   │
│  │            [a-c]         [n-z]                          │   │
│  │           / | \          / | \                          │   │
│  │         a  bin cat    opt sys usr                       │   │
│  │                                                          │   │
│  │  Lookup: O(log n)                                       │   │
│  │  Used in: NTFS, HFS+, modern ext4                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Path Name Resolution

```
┌──────────────────────────────────────────────────────────────────┐
│               PATH NAME RESOLUTION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  To open "/home/alice/docs/report.txt":                         │
│                                                                  │
│  Step 1: Start at root inode (inode 2 by convention)            │
│          Read root directory data                               │
│                                                                  │
│  Step 2: Look up "home" in root directory                       │
│          Found: inode 100                                       │
│          Read inode 100, get data blocks                        │
│                                                                  │
│  Step 3: Look up "alice" in /home directory                     │
│          Found: inode 500                                       │
│          Read inode 500, get data blocks                        │
│                                                                  │
│  Step 4: Look up "docs" in /home/alice directory                │
│          Found: inode 520                                       │
│          Read inode 520, get data blocks                        │
│                                                                  │
│  Step 5: Look up "report.txt" in /home/alice/docs               │
│          Found: inode 555                                       │
│          Return inode 555 to caller                             │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  / ──→ inode 2                                          │    │
│  │        ├── home → inode 100                             │    │
│  │        │         └── alice → inode 500                  │    │
│  │        │                    └── docs → inode 520        │    │
│  │        │                              └── report.txt    │    │
│  │        │                                    → inode 555 │    │
│  │        └── etc → inode 50                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Optimization: Directory Entry Cache (dcache)                   │
│  • Cache recent name→inode mappings                            │
│  • Avoid disk reads for common paths                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## File System Consistency

```
┌──────────────────────────────────────────────────────────────────┐
│              FILE SYSTEM CONSISTENCY                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem: Crash during multi-block update leaves inconsistency  │
│                                                                  │
│  Example: Creating a file requires:                              │
│  1. Allocate inode (update inode bitmap)                        │
│  2. Initialize inode                                            │
│  3. Add entry to directory                                      │
│  4. Allocate data blocks (update block bitmap)                  │
│                                                                  │
│  If crash between steps → Inconsistent state!                   │
│                                                                  │
│  SOLUTIONS:                                                      │
│                                                                  │
│  1. FSCK (File System Check)                                    │
│     • Run after crash to fix inconsistencies                   │
│     • Slow (must scan entire disk)                             │
│                                                                  │
│  2. JOURNALING                                                   │
│     • Write changes to log first, then to actual location      │
│     • On crash: replay log (fast recovery)                     │
│     • Used in: ext3, ext4, NTFS, HFS+                          │
│                                                                  │
│  3. COPY-ON-WRITE (COW)                                         │
│     • Never overwrite existing data                            │
│     • Write new data elsewhere, update pointers atomically     │
│     • Used in: ZFS, Btrfs                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Journaling

```
┌──────────────────────────────────────────────────────────────────┐
│                      JOURNALING                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Write-Ahead Logging:                                            │
│  1. Write operation to journal (log)                            │
│  2. Commit the transaction                                      │
│  3. Perform actual writes (checkpoint)                          │
│  4. Mark transaction complete                                   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                        JOURNAL                           │    │
│  │ ┌─────────┬─────────┬─────────┬─────────┬─────────┐    │    │
│  │ │  TxB    │  Data   │  Data   │  Data   │  TxE    │    │    │
│  │ │ (Begin) │ Block 1 │ Block 2 │ Block 3 │ (End)   │    │    │
│  │ └─────────┴─────────┴─────────┴─────────┴─────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Recovery:                                                       │
│  • If TxE missing: Transaction incomplete, ignore               │
│  • If TxE present: Replay writes if not already done           │
│                                                                  │
│  Journaling modes (ext4):                                        │
│  • data=journal: Journal all data (safest, slowest)            │
│  • data=ordered: Journal metadata, write data first (default)  │
│  • data=writeback: Journal metadata only (fastest, risky)      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Common File Systems

| File System | OS | Features |
|-------------|-----|----------|
| **ext4** | Linux | Journaling, extents, large files |
| **NTFS** | Windows | Journaling, ACLs, encryption |
| **HFS+/APFS** | macOS | Journaling, snapshots (APFS) |
| **FAT32** | Universal | Simple, portable, no journaling |
| **ZFS** | Unix/Linux | COW, checksums, pooled storage |
| **Btrfs** | Linux | COW, snapshots, checksums |

---

## Summary

| Concept | Description |
|---------|-------------|
| **Superblock** | File system metadata (size, counts) |
| **Inode** | File metadata and block pointers |
| **VFS** | Unified interface for different file systems |
| **Dentry** | Directory entry (name → inode cache) |
| **Buffer Cache** | Cache for disk blocks |
| **Journaling** | Log changes for crash recovery |
| **COW** | Copy-on-write for consistency |

---

## Quick Revision Questions

1. What information is stored in the superblock?
2. How does VFS allow multiple file system types to coexist?
3. Describe the steps in path name resolution for "/home/user/file.txt".
4. How does journaling help with crash recovery?
5. What are the advantages of copy-on-write file systems?

---

[← Previous: Free Space Management](./04-free-space-management.md) | [Next: I/O Hardware →](../07-IO-Systems/01-io-hardware.md)
