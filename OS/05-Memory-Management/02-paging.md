# Chapter 5.2: Paging

## Overview

Paging is a memory management scheme that permits the physical address space of a process to be non-contiguous. It eliminates external fragmentation and makes efficient use of memory.

---

## Basic Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    PAGING CONCEPT                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Key Idea: Divide memory into fixed-size blocks                 │
│                                                                  │
│  LOGICAL MEMORY                    PHYSICAL MEMORY               │
│  (Process View)                    (Actual RAM)                  │
│                                                                  │
│  ┌───────────────┐                 ┌───────────────┐            │
│  │   Page 0      │ ─────────────→  │   Frame 5     │            │
│  ├───────────────┤                 ├───────────────┤            │
│  │   Page 1      │ ───┐            │   Frame 1     │ ←──┐       │
│  ├───────────────┤    │            ├───────────────┤    │       │
│  │   Page 2      │ ───┼─────────→  │   Frame 2     │    │       │
│  ├───────────────┤    │            ├───────────────┤    │       │
│  │   Page 3      │    └─────────→  │   Frame 3     │    │       │
│  └───────────────┘                 ├───────────────┤    │       │
│                                    │   Frame 4     │    │       │
│  Pages don't need to be           ├───────────────┤    │       │
│  in contiguous frames!             │   Frame 5     │ ←──│       │
│                                    ├───────────────┤    │       │
│                                    │   Frame 6     │    │       │
│                                    └───────────────┘    │       │
│                                                         │       │
│  Page = Fixed-size block of logical memory             ─┘       │
│  Frame = Fixed-size block of physical memory                    │
│  Page Size = Frame Size (power of 2, e.g., 4KB)                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Address Translation

### Components of Logical Address

```
┌──────────────────────────────────────────────────────────────────┐
│               LOGICAL ADDRESS STRUCTURE                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Logical Address (m bits):                                       │
│  ┌─────────────────────────┬───────────────────────┐            │
│  │      Page Number (p)    │    Page Offset (d)    │            │
│  │       (m - n bits)      │       (n bits)        │            │
│  └─────────────────────────┴───────────────────────┘            │
│                                                                  │
│  Where:                                                          │
│  • Page Size = 2^n bytes                                        │
│  • Number of pages = 2^(m-n)                                    │
│                                                                  │
│  Example: 32-bit address, 4KB pages (2^12)                      │
│  ┌────────────────────────┬────────────────────────┐            │
│  │   Page Number (20 bits)│   Offset (12 bits)    │            │
│  └────────────────────────┴────────────────────────┘            │
│                                                                  │
│  • Can address 2^20 = 1M pages per process                      │
│  • Each page is 4KB                                             │
│  • Max process size = 1M × 4KB = 4GB                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Translation Process

```
┌──────────────────────────────────────────────────────────────────┐
│                ADDRESS TRANSLATION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│       CPU                                                        │
│        │                                                        │
│        ▼                                                        │
│  ┌───────────┬──────────┐                                       │
│  │  Page (p) │ Offset(d)│    Logical Address                    │
│  └─────┬─────┴──────────┘                                       │
│        │           │                                            │
│        │           │                                            │
│        ▼           │                                            │
│  ┌─────────────┐   │                                            │
│  │ Page Table  │   │                                            │
│  ├─────────────┤   │                                            │
│  │  0 │   5    │   │       Page 0 → Frame 5                    │
│  ├─────────────┤   │                                            │
│  │  1 │   3    │   │       Page 1 → Frame 3                    │
│  ├─────────────┤   │                                            │
│  │  2 │   2    │   │       Page 2 → Frame 2                    │
│  ├─────────────┤   │                                            │
│  │  3 │   7    │   │       Page 3 → Frame 7                    │
│  └──────┬──────┘   │                                            │
│         │          │                                            │
│         ▼          │                                            │
│  ┌───────────┬─────┴────┐                                       │
│  │ Frame (f) │ Offset(d)│    Physical Address                   │
│  └───────────┴──────────┘                                       │
│                                                                  │
│  Physical Address = (Frame Number × Page Size) + Offset         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example Calculation

```
Given:
• Page size = 4 bytes (for simplicity)
• Logical address = 5
• Page table: [0→1, 1→4, 2→3, 3→7]

Step 1: Find page number and offset
  Page number = 5 / 4 = 1
  Offset = 5 % 4 = 1

Step 2: Look up page table
  Page 1 → Frame 4

Step 3: Calculate physical address
  Physical address = (4 × 4) + 1 = 17

┌─────┬─────┬─────┬─────┐    ┌─────┬─────┬─────┬─────┐
│  0  │  1  │  2  │  3  │    │  0  │  1  │  2  │  3  │
├─────┼─────┼─────┼─────┤    ├─────┼─────┼─────┼─────┤
│  4  │  5  │  6  │  7  │    │  4  │  5  │  6  │  7  │
├─────┼─────┼─────┼─────┤    ├─────┼─────┼─────┼─────┤
│  8  │  9  │ 10  │ 11  │    │  8  │  9  │ 10  │ 11  │
├─────┼─────┼─────┼─────┤    ├─────┼─────┼─────┼─────┤
│ 12  │ 13  │ 14  │ 15  │    │ 12  │ 13  │ 14  │ 15  │
└─────┴─────┴─────┴─────┘    ├─────┼─────┼─────┼─────┤
   Logical Memory            │ 16  │ 17  │ 18  │ 19  │
   (Page 1 = addresses 4-7)  └─────┴─────┴─────┴─────┘
                                Physical Memory
                              (Frame 4 = addresses 16-19)
                                        ↑
                              Logical 5 → Physical 17
```

---

## Page Table

### Page Table Entry (PTE)

```
┌──────────────────────────────────────────────────────────────────┐
│                  PAGE TABLE ENTRY                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────┬────┬────┬────┬────┬────────────────────────────────┐    │
│  │ V  │ R  │ M  │ Prot│ PCD│       Frame Number             │    │
│  │(1) │(1) │(1) │(2)  │(1) │         (20+ bits)             │    │
│  └────┴────┴────┴────┴────┴────────────────────────────────┘    │
│                                                                  │
│  V (Valid/Present bit):                                          │
│  • 1 = Page is in physical memory                               │
│  • 0 = Page not in memory (on disk)                             │
│                                                                  │
│  R (Referenced bit):                                             │
│  • 1 = Page has been accessed recently                          │
│  • Used by page replacement algorithms                          │
│                                                                  │
│  M (Modified/Dirty bit):                                         │
│  • 1 = Page has been written to                                 │
│  • If dirty, must write back to disk before replacing          │
│                                                                  │
│  Prot (Protection bits):                                         │
│  • Read/Write/Execute permissions                               │
│  • Example: 00=none, 01=read, 10=read-write, 11=all            │
│                                                                  │
│  PCD (Page Cache Disable):                                       │
│  • Controls caching behavior                                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Page Table Size Problem

```
Example: 32-bit address space, 4KB pages

Page table entries needed = 2^32 / 2^12 = 2^20 = 1 million entries
If each entry is 4 bytes: Page table size = 4 MB per process!

With 100 processes: 400 MB just for page tables!

This is the motivation for hierarchical page tables.
```

---

## Hierarchical Paging

### Two-Level Paging

```
┌──────────────────────────────────────────────────────────────────┐
│                  TWO-LEVEL PAGING                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Logical Address (32-bit, 4KB pages):                           │
│  ┌────────────┬────────────┬────────────────┐                   │
│  │   P1 (10)  │   P2 (10)  │  Offset (12)   │                   │
│  └─────┬──────┴─────┬──────┴────────────────┘                   │
│        │            │                                            │
│        ▼            │                                            │
│  ┌───────────┐      │      (Outer Page Table)                   │
│  │ Outer PT  │      │                                           │
│  ├───────────┤      │                                           │
│  │     ●─────┼──────┼─────────────────────┐                    │
│  │     ●     │      │                     │                     │
│  │     ●     │      │                     ▼                     │
│  └───────────┘      │            ┌───────────┐                  │
│                     │            │ Inner PT  │  (Page of        │
│                     └───────────→│           │   page table)    │
│                                  ├───────────┤                  │
│                                  │   Frame # │                  │
│                                  │     ●─────┼────→ Physical   │
│                                  │     ●     │       Frame      │
│                                  └───────────┘                  │
│                                                                  │
│  Advantage: Only allocate inner page tables as needed          │
│  Memory saved for sparse address spaces                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Address Translation Steps

```
32-bit address with two-level paging:

Address: 0x00403004

Binary: 0000 0000 0100 | 0000 0011 | 0000 0000 0100
        P1 (10 bits)   | P2 (10 bits) | Offset (12 bits)
        = 1            | = 3          | = 4

Translation:
1. Use P1 (1) to index outer page table → Get pointer to inner PT
2. Use P2 (3) to index inner page table → Get frame number (say, 7)
3. Physical address = Frame 7 + Offset 4 = 7 × 4096 + 4 = 28676
```

---

## Inverted Page Table

```
┌──────────────────────────────────────────────────────────────────┐
│                 INVERTED PAGE TABLE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Instead of one PT per process, ONE global table for all frames │
│                                                                  │
│  Traditional:                  Inverted:                         │
│  ┌─────────────┐              ┌──────────────────────┐          │
│  │ Process 1   │              │  Frame 0: P1, Page 3 │          │
│  │ Page Table  │              │  Frame 1: P3, Page 7 │          │
│  ├─────────────┤              │  Frame 2: P1, Page 1 │          │
│  │ Process 2   │              │  Frame 3: P2, Page 0 │          │
│  │ Page Table  │              │  Frame 4: P3, Page 2 │          │
│  ├─────────────┤              │  ...                 │          │
│  │ Process 3   │              │  Frame N: Pid, Page# │          │
│  │ Page Table  │              └──────────────────────┘          │
│  └─────────────┘                                                │
│                                                                  │
│  Entry format: <process-id, page-number>                        │
│                                                                  │
│  To find frame for (pid, page):                                 │
│  Search table for matching entry → Index is frame number        │
│                                                                  │
│  Advantages:                                                     │
│  • Fixed size (one entry per physical frame)                    │
│  • Saves memory with many processes                             │
│                                                                  │
│  Disadvantages:                                                  │
│  • Search required for each address translation                 │
│  • Solved with hash table                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Translation Lookaside Buffer (TLB)

### The Problem

```
Without TLB:
Every memory access requires TWO memory accesses:
1. Access page table to get frame number
2. Access actual data in that frame

This DOUBLES memory access time!
```

### Solution: TLB

```
┌──────────────────────────────────────────────────────────────────┐
│          TRANSLATION LOOKASIDE BUFFER (TLB)                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TLB = Fast cache of recent page table entries                  │
│  (Stored in associative memory - parallel search)               │
│                                                                  │
│  ┌──────────────────┬───────────────────┐                       │
│  │    Page Number   │   Frame Number    │                       │
│  ├──────────────────┼───────────────────┤                       │
│  │        3         │        7          │                       │
│  │       10         │        4          │                       │
│  │        5         │        2          │                       │
│  │       21         │       15          │                       │
│  │       ...        │       ...         │                       │
│  └──────────────────┴───────────────────┘                       │
│        (typically 32-1024 entries)                              │
│                                                                  │
│  Address Translation with TLB:                                   │
│                                                                  │
│          ┌─────────────────┐                                    │
│          │ Logical Address │                                    │
│          │  (Page, Offset) │                                    │
│          └────────┬────────┘                                    │
│                   │                                             │
│                   ▼                                             │
│          ┌─────────────────┐                                    │
│          │   Check TLB     │                                    │
│          └────────┬────────┘                                    │
│             Hit? /   \ Miss?                                    │
│                /       \                                         │
│               ▼         ▼                                       │
│         Get frame   Access Page Table                           │
│         from TLB    Get frame                                   │
│                     Update TLB                                  │
│               \       /                                          │
│                \     /                                           │
│                 ▼   ▼                                           │
│          ┌─────────────────┐                                    │
│          │Physical Address │                                    │
│          └─────────────────┘                                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Effective Access Time (EAT)

```
Given:
• Memory access time = 100 ns
• TLB access time = 10 ns
• TLB hit ratio = 98%

TLB Hit:   TLB access + Memory access = 10 + 100 = 110 ns
TLB Miss:  TLB access + PT access + Memory access = 10 + 100 + 100 = 210 ns

EAT = (Hit ratio × Hit time) + (Miss ratio × Miss time)
EAT = (0.98 × 110) + (0.02 × 210)
EAT = 107.8 + 4.2
EAT = 112 ns

Without TLB: 200 ns (two memory accesses)
With TLB:    112 ns (44% faster!)
```

---

## Advantages and Disadvantages of Paging

### Advantages

| Advantage | Description |
|-----------|-------------|
| **No external fragmentation** | Any free frame can be used |
| **Simple allocation** | Just find any free frame |
| **Easy sharing** | Multiple processes can map same frame |
| **Protection** | Each page can have protection bits |

### Disadvantages

| Disadvantage | Description |
|--------------|-------------|
| **Internal fragmentation** | Last page may not be fully used |
| **Page table overhead** | Large page tables need memory |
| **Memory access overhead** | Extra access for page table |
| **Not logical** | Process doesn't see logical divisions |

---

## Shared Pages

```
┌──────────────────────────────────────────────────────────────────┐
│                     SHARED PAGES                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Multiple processes can share the same frames                   │
│  Common use: Shared libraries, code segments                    │
│                                                                  │
│  Process P1                      Process P2                      │
│  ┌─────────────┐                 ┌─────────────┐                │
│  │ Page Table  │                 │ Page Table  │                │
│  ├─────────────┤                 ├─────────────┤                │
│  │ Code: Fr 3  │ ───────────┬───│ Code: Fr 3  │                │
│  │ Code: Fr 4  │ ─────────┬─┼───│ Code: Fr 4  │                │
│  │ Data: Fr 7  │          │ │   │ Data: Fr 9  │                │
│  │ Stack: Fr 2 │          │ │   │ Stack: Fr 5 │                │
│  └─────────────┘          │ │   └─────────────┘                │
│                           │ │                                    │
│                           ▼ ▼                                    │
│                    Physical Memory                               │
│                    ┌───────────┐                                │
│                    │  Frame 3  │  Shared code                   │
│                    ├───────────┤                                │
│                    │  Frame 4  │  Shared code                   │
│                    ├───────────┤                                │
│                    │  Frame 7  │  P1's data                     │
│                    ├───────────┤                                │
│                    │  Frame 9  │  P2's data                     │
│                    └───────────┘                                │
│                                                                  │
│  Requirements for sharing:                                       │
│  • Code must be reentrant (read-only, no self-modification)    │
│  • Same logical address preferred (but not required)           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Page** | Fixed-size block of logical memory |
| **Frame** | Fixed-size block of physical memory |
| **Page Table** | Maps pages to frames |
| **TLB** | Cache for page table entries |
| **Two-Level Paging** | Hierarchical page table structure |
| **Inverted PT** | One entry per frame, not per page |
| **Shared Pages** | Multiple processes share frames |

---

## Quick Revision Questions

1. How is a logical address divided in paging?
2. What is the purpose of the valid bit in a page table entry?
3. Why is a TLB necessary?
4. Calculate EAT given TLB hit ratio and access times.
5. How does two-level paging save memory?

---

[← Previous: Memory Introduction](./01-memory-introduction.md) | [Next: Segmentation →](./03-segmentation.md)
