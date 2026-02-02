# Chapter 6.4: Free Space Management

## Overview

The file system must track which disk blocks are free (available for allocation) and which are in use. This chapter covers various methods for managing free space on disk.

---

## Why Free Space Management?

```
┌──────────────────────────────────────────────────────────────────┐
│               WHY FREE SPACE MANAGEMENT?                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When creating or extending a file, OS needs to:                │
│  1. Find free blocks                                            │
│  2. Allocate them to the file                                   │
│  3. Update free space tracking                                   │
│                                                                  │
│  When deleting a file:                                           │
│  1. Release blocks back to free pool                            │
│  2. Update free space tracking                                   │
│                                                                  │
│  Requirements for free space management:                         │
│  • Quick to find free blocks                                    │
│  • Efficient space usage for tracking itself                    │
│  • Support for contiguous allocation (if needed)               │
│  • Fast updates on allocation/deallocation                      │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Disk: 1 TB = 250 million blocks (4 KB each)            │    │
│  │ Need efficient structure to track all these blocks!     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Bit Vector (Bitmap)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    BIT VECTOR (BITMAP)                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  One bit per disk block:                                         │
│  • 0 = block is free                                            │
│  • 1 = block is allocated (or vice versa)                       │
│                                                                  │
│  Example:                                                        │
│  Block:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15         │
│  Bitmap: 1  1  0  1  1  1  0  0  1  1  0  0  0  1  1  1         │
│          ↑     ↑           ↑  ↑     ↑  ↑  ↑                     │
│        used  free        free free free free free               │
│                                                                  │
│  Free blocks: 2, 6, 7, 10, 11, 12                               │
│                                                                  │
│  Stored as words for efficient processing:                       │
│  Word 0: 1101110011000111                                       │
│          ↑                                                       │
│         bit 0                                                    │
│                                                                  │
│  Block number from position:                                     │
│  block_number = (word_number × bits_per_word) + bit_offset      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Finding Free Block

```
┌──────────────────────────────────────────────────────────────────┐
│              FINDING FREE BLOCK IN BITMAP                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Algorithm:                                                      │
│  1. Scan bitmap for word not all 1s (has at least one 0)       │
│  2. Within that word, find first 0 bit                          │
│  3. Calculate block number                                       │
│                                                                  │
│  Hardware support (common):                                      │
│  • Find-first-zero instruction                                  │
│  • Scan word quickly                                            │
│                                                                  │
│  Example: Find first free block                                  │
│  Word 0: 11111111  (all allocated, skip)                        │
│  Word 1: 11111111  (all allocated, skip)                        │
│  Word 2: 11110111  (bit 3 is 0!)                                │
│                    ↑                                            │
│                   bit 3                                         │
│                                                                  │
│  Free block = 2 × 8 + 3 = 19                                    │
│               ↑   ↑   ↑                                         │
│            word  bits  offset                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Space Analysis

```
┌──────────────────────────────────────────────────────────────────┐
│                 BITMAP SPACE ANALYSIS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Disk size: 1 TB = 2^40 bytes                                   │
│  Block size: 4 KB = 2^12 bytes                                  │
│  Number of blocks: 2^40 / 2^12 = 2^28 = 256 million blocks     │
│                                                                  │
│  Bitmap size: 2^28 bits = 2^28 / 8 bytes = 32 MB               │
│                                                                  │
│  For 1 TB disk with 4 KB blocks:                                │
│  Bitmap overhead = 32 MB / 1 TB = 0.003%                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Disk Size    Block Size    # Blocks      Bitmap Size   │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  100 GB       4 KB          25 million    3.125 MB      │   │
│  │  500 GB       4 KB          125 million   15.6 MB       │   │
│  │  1 TB         4 KB          250 million   31.25 MB      │   │
│  │  4 TB         4 KB          1 billion     125 MB        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Ideally: Keep entire bitmap in memory for fast access          │
│           May not be possible for very large disks              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
✓ Simple and efficient
✓ Easy to find contiguous free blocks (for contiguous allocation)
✓ Compact representation
✓ Fast with hardware support (bit manipulation instructions)

DISADVANTAGES:
✗ Must be in memory for efficiency
✗ Large bitmap for very large disks
✗ Scanning can be slow if disk nearly full
```

---

## 2. Linked List (Free List)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                 LINKED LIST (FREE LIST)                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Link all free blocks together                                   │
│  Keep pointer to first free block (head)                        │
│                                                                  │
│  Free List Head: block 3                                        │
│                                                                  │
│  Disk blocks:                                                    │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐             │
│  │ D │ D │ D │ F │ D │ D │ F │ D │ F │ D │ F │ D │             │
│  └───┴───┴───┴─┼─┴───┴───┴─┼─┴───┴─┼─┴───┴─┼─┴───┘             │
│    0   1   2   3   4   5   6   7   8   9  10  11               │
│                │           │       │       │                    │
│                └───────────┴───────┴───────┴─→ NULL             │
│                 3 → 6 → 8 → 10 → NULL                           │
│                                                                  │
│  D = Data (allocated)                                            │
│  F = Free (contains pointer to next free block)                 │
│                                                                  │
│  Allocation: Remove from head (O(1))                            │
│  Deallocation: Add to head (O(1))                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Operations

```
┌──────────────────────────────────────────────────────────────────┐
│              FREE LIST OPERATIONS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ALLOCATE ONE BLOCK:                                             │
│  1. Take block from head of list                                │
│  2. Update head to point to next free block                     │
│                                                                  │
│  Before: Head → 3 → 6 → 8 → 10 → NULL                          │
│  After:  Head → 6 → 8 → 10 → NULL                              │
│  Allocated: Block 3                                              │
│                                                                  │
│  DEALLOCATE ONE BLOCK:                                           │
│  1. Add block to head of list                                   │
│                                                                  │
│  Before: Head → 6 → 8 → 10 → NULL                              │
│  Free block 5                                                    │
│  After:  Head → 5 → 6 → 8 → 10 → NULL                          │
│                                                                  │
│  Problem: Allocating n contiguous blocks                         │
│  Must traverse list to find consecutive blocks                  │
│  Very inefficient!                                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
✓ No wasted space (pointers stored in free blocks themselves)
✓ Simple allocation/deallocation (O(1) for single block)

DISADVANTAGES:
✗ Cannot easily find contiguous blocks
✗ Must read each block to traverse list (many disk I/Os)
✗ Poor performance if many free blocks
✗ Only first pointer in memory
```

---

## 3. Grouping

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                       GROUPING                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Store addresses of n free blocks in first free block           │
│  Last address points to block with next n free addresses        │
│                                                                  │
│  Free block (can hold 4 addresses):                             │
│  ┌─────────────────────────────────┐                            │
│  │  Block 10 (first free block)   │                            │
│  │ ┌───┬───┬───┬───┐              │                            │
│  │ │ 15│ 20│ 25│ 30│              │                            │
│  │ └───┴───┴───┴─┬─┘              │                            │
│  └───────────────┼─────────────────┘                            │
│                  │                                               │
│                  ▼                                               │
│  ┌─────────────────────────────────┐                            │
│  │  Block 30 (next group)         │                            │
│  │ ┌───┬───┬───┬───┐              │                            │
│  │ │ 35│ 40│ 45│ 50│              │                            │
│  │ └───┴───┴───┴─┬─┘              │                            │
│  └───────────────┼─────────────────┘                            │
│                  ▼                                               │
│                 ...                                              │
│                                                                  │
│  Free blocks available from block 10:                           │
│  10, 15, 20, 25, then follow to 30, 35, 40, 45...              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages

```
ADVANTAGES:
✓ Large number of free block addresses quickly accessible
✓ Fewer disk I/Os than simple linked list
✓ Can allocate multiple blocks efficiently

Example:
Block holds 100 addresses
First block gives 99 free blocks + link to next group
One I/O provides 99 allocatable blocks
```

---

## 4. Counting

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                       COUNTING                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Keep count of contiguous free blocks with first address        │
│  Store (start_address, count) pairs                             │
│                                                                  │
│  Disk state:                                                     │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐     │
│  │ D │ D │ F │ F │ F │ D │ D │ F │ F │ D │ F │ F │ F │ F │     │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘     │
│    0   1   2   3   4   5   6   7   8   9  10  11  12  13        │
│            └───┬───┘         └─┬─┘     └────┬────────┘          │
│           (2, 3)            (7, 2)       (10, 4)                │
│                                                                  │
│  Free space table:                                               │
│  ┌─────────┬───────┐                                            │
│  │ Start   │ Count │                                            │
│  ├─────────┼───────┤                                            │
│  │    2    │   3   │  (blocks 2, 3, 4)                         │
│  │    7    │   2   │  (blocks 7, 8)                            │
│  │   10    │   4   │  (blocks 10, 11, 12, 13)                  │
│  └─────────┴───────┘                                            │
│                                                                  │
│  Total free: 3 + 2 + 4 = 9 blocks                               │
│  Stored in just 3 entries instead of 9                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Operations

```
ALLOCATION:
1. Find entry with count ≥ requested blocks
2. If exact match: Remove entry
3. If larger: Update start and count, allocate from beginning

Example: Allocate 2 blocks from entry (2, 3)
- Allocate blocks 2, 3
- Update entry to (4, 1)

DEALLOCATION:
1. Check if freed blocks are adjacent to existing entry
2. If yes: Merge (update count)
3. If no: Create new entry

Example: Free block 9
- Adjacent to (10, 4)
- Merge to (9, 5)
```

### Advantages

```
ADVANTAGES:
✓ Excellent for contiguous allocation
✓ Very compact when blocks tend to be contiguous
✓ Easy to find runs of free blocks
✓ Merging keeps table small

DISADVANTAGES:
✗ Table can grow large if free space is fragmented
✗ Each allocation/deallocation may require table update
```

---

## Comparison of Methods

```
┌──────────────────────────────────────────────────────────────────┐
│              FREE SPACE MANAGEMENT COMPARISON                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────┬──────────────┬─────────────┬───────────────┐    │
│  │ Method     │ Space        │ Contiguous  │ Speed         │    │
│  │            │ Overhead     │ Allocation  │               │    │
│  ├────────────┼──────────────┼─────────────┼───────────────┤    │
│  │ Bitmap     │ Fixed        │ Easy        │ Fast (in RAM) │    │
│  │            │ 1 bit/block  │ (scan bits) │               │    │
│  ├────────────┼──────────────┼─────────────┼───────────────┤    │
│  │ Linked     │ 0 (uses free │ Hard        │ Slow          │    │
│  │ List       │ blocks)      │ (traverse)  │ (disk I/O)    │    │
│  ├────────────┼──────────────┼─────────────┼───────────────┤    │
│  │ Grouping   │ 0 (uses free │ Moderate    │ Moderate      │    │
│  │            │ blocks)      │             │               │    │
│  ├────────────┼──────────────┼─────────────┼───────────────┤    │
│  │ Counting   │ Variable     │ Easy        │ Fast          │    │
│  │            │ (small if    │ (check      │               │    │
│  │            │ contiguous)  │ count)      │               │    │
│  └────────────┴──────────────┴─────────────┴───────────────┘    │
│                                                                  │
│  Most common: BITMAP (used in ext2/3/4, NTFS)                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Block Groups (Modern Approach)

```
┌──────────────────────────────────────────────────────────────────┐
│                    BLOCK GROUPS                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Divide disk into block groups (used in ext2/3/4)               │
│  Each group has its own bitmap and inode table                   │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                         Disk                               │  │
│  │ ┌─────────┬─────────┬─────────┬─────────┬─────────┐      │  │
│  │ │ Group 0 │ Group 1 │ Group 2 │ Group 3 │   ...   │      │  │
│  │ └────┬────┴────┬────┴────┬────┴─────────┴─────────┘      │  │
│  └──────┼─────────┼─────────┼────────────────────────────────┘  │
│         │         │         │                                    │
│         ▼         │         │                                    │
│  ┌──────────────┐ │         │                                    │
│  │ Super block  │ │         │                                    │
│  │ Group desc  │ │         │                                    │
│  │ Block bitmap │ │         │                                    │
│  │ Inode bitmap │ │         │                                    │
│  │ Inode table  │ │         │                                    │
│  │ Data blocks  │ │         │                                    │
│  └──────────────┘ │         │                                    │
│                   ▼         │                                    │
│            (same structure) │                                    │
│                             ▼                                    │
│                      (same structure)                            │
│                                                                  │
│  Benefits:                                                       │
│  • Keep file data near its inode (less seeking)                │
│  • Parallel allocation from multiple groups                     │
│  • Redundancy (superblock copies in each group)                │
│  • Smaller bitmaps to scan                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Method | How It Works | Best For |
|--------|--------------|----------|
| **Bitmap** | 1 bit per block | Most file systems |
| **Linked List** | Free blocks form a chain | Simple systems |
| **Grouping** | Groups of free addresses | Batch allocations |
| **Counting** | (start, count) pairs | Contiguous allocation |
| **Block Groups** | Divide disk into regions | Large modern disks |

---

## Quick Revision Questions

1. Calculate bitmap size for a 500 GB disk with 4 KB blocks.
2. Why is linked list inefficient for contiguous allocation?
3. How does counting improve over simple linked list?
4. What are the advantages of block groups?
5. Why is bitmap preferred in most modern file systems?

---

[← Previous: File Allocation Methods](./03-file-allocation.md) | [Next: File System Implementation →](./05-file-system-implementation.md)
