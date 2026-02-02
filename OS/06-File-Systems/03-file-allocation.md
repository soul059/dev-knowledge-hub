# Chapter 6.3: File Allocation Methods

## Overview

When a file is created, disk space must be allocated to store its data. The allocation method determines how disk blocks are assigned to files and significantly impacts performance and storage efficiency.

---

## Disk Structure Background

```
┌──────────────────────────────────────────────────────────────────┐
│                    DISK STRUCTURE                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Disk is divided into fixed-size BLOCKS (or sectors)            │
│  Typical block size: 512 bytes to 4 KB                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │ Block │ Block │ Block │ Block │ Block │ ... │ Block   │     │
│  │   0   │   1   │   2   │   3   │   4   │     │   n     │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  File System's job:                                              │
│  • Track which blocks belong to which file                      │
│  • Track which blocks are free                                  │
│  • Allocate blocks when files grow                              │
│  • Free blocks when files deleted                               │
│                                                                  │
│  Key considerations:                                             │
│  • Storage efficiency (minimize wasted space)                   │
│  • Access speed (sequential and random)                         │
│  • Simplicity of implementation                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Contiguous Allocation

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                  CONTIGUOUS ALLOCATION                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each file occupies a set of CONSECUTIVE blocks on disk         │
│                                                                  │
│  Directory entry: (filename, start_block, length)               │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ File    │ Start │ Length │                              │    │
│  ├─────────┼───────┼────────┤                              │    │
│  │ file_A  │   0   │   3    │                              │    │
│  │ file_B  │   5   │   4    │                              │    │
│  │ file_C  │  12   │   2    │                              │    │
│  └─────────┴───────┴────────┘                              │    │
│                                                                  │
│  Disk layout:                                                    │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐     │
│  │ A │ A │ A │   │   │ B │ B │ B │ B │   │   │   │ C │ C │     │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘     │
│    0   1   2   3   4   5   6   7   8   9  10  11  12  13        │
│                                                                  │
│  To access block i of file_B:                                   │
│  Physical block = start + i = 5 + i                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
┌──────────────────────────────────────────────────────────────────┐
│           CONTIGUOUS ALLOCATION: PROS AND CONS                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADVANTAGES:                                                     │
│  ✓ Simple implementation                                        │
│  ✓ Excellent for sequential access (no seeking)                │
│  ✓ Easy random access: block_addr = start + offset            │
│  ✓ Minimal disk head movement                                   │
│                                                                  │
│  DISADVANTAGES:                                                  │
│  ✗ External fragmentation                                       │
│                                                                  │
│    After deletions:                                              │
│    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐                   │
│    │ A │   │   │ B │ B │   │ C │   │   │ D │                   │
│    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘                   │
│        ↑   ↑           ↑       ↑   ↑                            │
│        Fragmented free space (holes)                            │
│        Cannot allocate file needing 4 contiguous blocks!        │
│                                                                  │
│  ✗ File cannot grow easily                                      │
│    - Must know file size at creation                            │
│    - Growing requires moving entire file                        │
│                                                                  │
│  ✗ Need compaction (expensive)                                  │
│    - Move files to create larger contiguous space              │
│    - Like defragmentation                                       │
│                                                                  │
│  USED IN: CD-ROM, DVD (read-only, known sizes)                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Linked Allocation

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   LINKED ALLOCATION                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each file is a linked list of disk blocks                      │
│  Blocks may be scattered anywhere on disk                        │
│                                                                  │
│  Directory entry: (filename, start_block, end_block)            │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ File    │ Start │ End  │                                │    │
│  ├─────────┼───────┼──────┤                                │    │
│  │ file_A  │   2   │  10  │                                │    │
│  │ file_B  │   5   │  15  │                                │    │
│  └─────────┴───────┴──────┘                                │    │
│                                                                  │
│  file_A: Block 2 → Block 7 → Block 10 → NULL                   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │   │   │   │   │   │   │   │   │   │   │   │             │    │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘             │    │
│    0   1   2   3   4   5   6   7   8   9  10               │    │
│            │               │       │           │            │    │
│            │               │       │           │            │    │
│            ▼               ▼       ▼           ▼            │    │
│       ┌────────┐     ┌────────┐ ┌────────┐ ┌────────┐      │    │
│       │ Data   │     │ Data   │ │ Data   │ │ Data   │      │    │
│       │ ptr→7  │     │ ptr→15 │ │ ptr→10 │ │ ptr→-1 │      │    │
│       └────────┘     └────────┘ └────────┘ └────────┘      │    │
│       Block 2        Block 5    Block 7    Block 10        │    │
│       (file_A)       (file_B)   (file_A)   (file_A end)    │    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
┌──────────────────────────────────────────────────────────────────┐
│            LINKED ALLOCATION: PROS AND CONS                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADVANTAGES:                                                     │
│  ✓ No external fragmentation                                    │
│  ✓ Any free block can be used                                  │
│  ✓ Files can grow easily                                        │
│  ✓ No need to know file size in advance                        │
│  ✓ No compaction needed                                         │
│                                                                  │
│  DISADVANTAGES:                                                  │
│  ✗ Slow random access                                           │
│    - To access block i, must traverse i-1 blocks               │
│    - O(n) time complexity                                       │
│                                                                  │
│  ✗ Pointer overhead                                             │
│    - Each block wastes space for pointer                        │
│    - 4 bytes out of 512 = 0.78% overhead                       │
│                                                                  │
│  ✗ Reliability issues                                           │
│    - If one pointer corrupted, lose rest of file               │
│    - Pointers scattered across disk                            │
│                                                                  │
│  ✗ Poor for sequential access                                   │
│    - Blocks scattered, lots of seeking                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### FAT (File Allocation Table)

```
┌──────────────────────────────────────────────────────────────────┐
│              FILE ALLOCATION TABLE (FAT)                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Variation of linked allocation                                  │
│  Move all pointers to a separate table at beginning of disk     │
│                                                                  │
│  FAT Table:                    Disk Data Blocks:                │
│  ┌───────┬─────────┐          ┌───┬───┬───┬───┬───┬───┐        │
│  │ Block │ Next    │          │   │   │   │   │   │   │        │
│  ├───────┼─────────┤          └───┴───┴───┴───┴───┴───┘        │
│  │   0   │  FREE   │            0   1   2   3   4   5          │
│  │   1   │   4     │──┐                                        │
│  │   2   │  FREE   │  │        file_A: starts at block 1       │
│  │   3   │  EOF    │  │        chain: 1 → 4 → 3 (EOF)          │
│  │   4   │   3     │←─┘                                        │
│  │   5   │  FREE   │                                           │
│  └───────┴─────────┘                                           │
│                                                                  │
│  Advantages over simple linked:                                  │
│  ✓ FAT can be cached in memory                                  │
│  ✓ Random access just needs FAT lookup                         │
│  ✓ Data blocks fully available (no pointer space)              │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ FAT can be large (needs space)                              │
│  ✗ If FAT corrupted, entire disk affected                      │
│                                                                  │
│  USED IN: MS-DOS, USB drives, SD cards (FAT32)                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Indexed Allocation

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   INDEXED ALLOCATION                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each file has an INDEX BLOCK containing pointers to data blocks│
│                                                                  │
│  Directory entry: (filename, index_block)                       │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ File    │ Index Block │                                 │    │
│  ├─────────┼─────────────┤                                 │    │
│  │ file_A  │      5      │                                 │    │
│  │ file_B  │      8      │                                 │    │
│  └─────────┴─────────────┘                                 │    │
│                                                                  │
│  Block 5 (index block for file_A):                              │
│  ┌─────────────────┐                                            │
│  │ 0 → block 2     │───→ Data block 2                          │
│  │ 1 → block 7     │───→ Data block 7                          │
│  │ 2 → block 3     │───→ Data block 3                          │
│  │ 3 → block 12    │───→ Data block 12                         │
│  │ 4 → NULL        │                                           │
│  │ ...             │                                           │
│  └─────────────────┘                                            │
│                                                                  │
│  To access block i of file_A:                                   │
│  1. Go to index block 5                                        │
│  2. Read entry i to get physical block number                  │
│  3. Access that physical block                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
┌──────────────────────────────────────────────────────────────────┐
│            INDEXED ALLOCATION: PROS AND CONS                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADVANTAGES:                                                     │
│  ✓ Direct random access                                         │
│    - O(1) time to access any block                             │
│  ✓ No external fragmentation                                    │
│  ✓ Good for both sequential and random access                  │
│                                                                  │
│  DISADVANTAGES:                                                  │
│  ✗ Pointer overhead (index block space)                         │
│  ✗ Wasted space for small files                                │
│    - Every file needs at least one index block                 │
│    - Small file: 1 data block + 1 index block = 50% overhead   │
│  ✗ Index block size limits maximum file size                   │
│                                                                  │
│  Maximum file size problem:                                      │
│  Block = 4 KB, pointer = 4 bytes                                │
│  Pointers per index block = 4096/4 = 1024                       │
│  Max file = 1024 × 4 KB = 4 MB (too small!)                    │
│                                                                  │
│  Solutions: Multi-level indexing, combined scheme               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Multi-Level Index

```
┌──────────────────────────────────────────────────────────────────┐
│                  MULTI-LEVEL INDEX                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Use multiple levels of index blocks for large files            │
│                                                                  │
│  Two-Level Index:                                                │
│                                                                  │
│  Directory ─→ Outer Index ─→ Index Block ─→ Data Block          │
│                                                                  │
│  ┌────────┐     ┌──────────────┐     ┌──────────────┐           │
│  │  file  │     │ Outer Index  │     │ Index Block  │           │
│  │   │    │────→│   0 → ────────────→│  0 → data    │           │
│  └───┼────┘     │   1 → ●      │     │  1 → data    │           │
│                 │   2 → ●      │     │  ...         │           │
│                 └──────┼───────┘     └──────────────┘           │
│                        │                                        │
│                        ▼                                        │
│                  ┌──────────────┐                               │
│                  │ Index Block  │                               │
│                  │  0 → data    │                               │
│                  │  1 → data    │                               │
│                  └──────────────┘                               │
│                                                                  │
│  With 4 KB blocks, 4-byte pointers:                             │
│  Pointers per block = 1024                                      │
│  Two-level max = 1024 × 1024 × 4 KB = 4 GB                     │
│  Three-level max = 1024³ × 4 KB = 4 TB                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Combined Scheme (Unix Inode)

```
┌──────────────────────────────────────────────────────────────────┐
│                 COMBINED SCHEME (UNIX INODE)                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Use different methods for different file sizes                 │
│                                                                  │
│  ┌───────────────────────────────────────────────────────┐      │
│  │                     INODE                              │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ Mode (permissions, type)                              │      │
│  │ Owner, Group                                          │      │
│  │ Size, Timestamps                                      │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ Direct blocks (12 pointers)                           │      │
│  │   0  ─→ data block     (fastest access)              │      │
│  │   1  ─→ data block                                   │      │
│  │   ...                                                 │      │
│  │   11 ─→ data block     (up to 48 KB directly)       │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ Single indirect ─→ index ─→ data blocks             │      │
│  │                     (adds 4 MB)                       │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ Double indirect ─→ index ─→ index ─→ data           │      │
│  │                     (adds 4 GB)                       │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ Triple indirect ─→ index ─→ index ─→ index ─→ data  │      │
│  │                     (adds 4 TB)                       │      │
│  └───────────────────────────────────────────────────────┘      │
│                                                                  │
│  Benefits:                                                       │
│  • Small files: Only direct blocks used (efficient)            │
│  • Large files: Multi-level indices provide capacity           │
│  • Most files are small (< 48 KB) → fast access                │
│                                                                  │
│  USED IN: Unix, Linux (ext2/3/4), BSD                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Comparison of Allocation Methods

```
┌──────────────────────────────────────────────────────────────────┐
│              COMPARISON OF ALLOCATION METHODS                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┬────────────┬────────────┬────────────┐         │
│  │ Aspect      │ Contiguous │ Linked     │ Indexed    │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ Sequential  │ Excellent  │ Good       │ Good       │         │
│  │ access      │            │            │            │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ Random      │ Easy       │ Slow       │ Easy       │         │
│  │ access      │ O(1)       │ O(n)       │ O(1)       │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ External    │ Yes        │ No         │ No         │         │
│  │ fragment    │            │            │            │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ File growth │ Difficult  │ Easy       │ Easy       │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ Space       │ Low        │ Pointer    │ Index      │         │
│  │ overhead    │            │ per block  │ block      │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ Reliability │ Good       │ Poor       │ Good       │         │
│  │             │            │ (chain)    │            │         │
│  ├─────────────┼────────────┼────────────┼────────────┤         │
│  │ Used in     │ CD-ROM     │ FAT        │ Unix/ext   │         │
│  └─────────────┴────────────┴────────────┴────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Method | Description | Best For |
|--------|-------------|----------|
| **Contiguous** | Consecutive blocks | Read-only media, known sizes |
| **Linked** | Chain of blocks with pointers | Sequential files |
| **FAT** | Linked with table | USB drives, simple systems |
| **Indexed** | Index block with pointers | Random access needed |
| **Multi-Level** | Hierarchy of index blocks | Large files |
| **Combined (Inode)** | Direct + indirect blocks | General purpose (Unix) |

---

## Quick Revision Questions

1. Why is contiguous allocation not suitable for files that grow?
2. What problem does FAT solve over simple linked allocation?
3. How does the Unix inode handle both small and large files efficiently?
4. Calculate max file size with single-level index (4 KB blocks, 4-byte pointers).
5. Which allocation method is best for random access? Why?

---

[← Previous: Directory Structure](./02-directory-structure.md) | [Next: Free Space Management →](./04-free-space-management.md)
