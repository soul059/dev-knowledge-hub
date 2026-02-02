# Chapter 5.4: Virtual Memory

## Overview

Virtual memory is a technique that allows execution of processes that are not completely in main memory. It separates logical memory from physical memory, enabling programs larger than physical memory to run.

---

## Why Virtual Memory?

```
┌──────────────────────────────────────────────────────────────────┐
│                   WHY VIRTUAL MEMORY?                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problems with loading entire program:                          │
│                                                                  │
│  1. Program may be larger than physical memory                  │
│     • 8 GB program on 4 GB RAM machine                         │
│                                                                  │
│  2. Many program parts rarely used                              │
│     • Error handling code                                       │
│     • Unusual options                                           │
│     • Large data structures (only parts used)                   │
│                                                                  │
│  3. Limits multiprogramming                                      │
│     • If programs fully loaded, fewer can run                   │
│                                                                  │
│  Solution: VIRTUAL MEMORY                                        │
│  • Load only parts of program that are needed                   │
│  • Keep rest on disk                                            │
│  • Swap parts in/out as needed                                  │
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │  Virtual Address Space: 4 GB             │                   │
│  │  Physical Memory: 1 GB                    │                   │
│  │  Each process THINKS it has 4 GB         │                   │
│  │  OS manages the illusion                  │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Virtual Address Space

```
┌──────────────────────────────────────────────────────────────────┐
│               VIRTUAL ADDRESS SPACE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each process has its own virtual address space                 │
│                                                                  │
│  ┌──────────────────────────────┐  Max Address (e.g., 4GB)      │
│  │          Stack              │  ↓ Grows down                  │
│  │             ↓               │                                │
│  │                             │                                │
│  │         (unused)            │  ← Sparse: not all allocated  │
│  │                             │                                │
│  │             ↑               │                                │
│  │          Heap               │  ↑ Grows up                   │
│  ├──────────────────────────────┤                                │
│  │          Data               │  Global variables             │
│  ├──────────────────────────────┤                                │
│  │          Code               │  Program instructions         │
│  └──────────────────────────────┘  Address 0                    │
│                                                                  │
│  Only pages actually USED are in physical memory               │
│  Unused pages (holes) don't consume physical RAM               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Demand Paging

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    DEMAND PAGING                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Load pages only when demanded (accessed)                       │
│  "Lazy Loading"                                                  │
│                                                                  │
│  Initially:                                                      │
│  • No pages in memory                                           │
│  • All pages marked invalid in page table                       │
│                                                                  │
│  When page accessed:                                             │
│  • Page fault occurs                                            │
│  • OS loads page from disk                                      │
│  • Update page table                                            │
│  • Restart instruction                                          │
│                                                                  │
│  Page Table Entry:                                               │
│  ┌───────┬────────────────────────────────────────────┐         │
│  │Valid=1│ Frame number (page is in memory)          │         │
│  ├───────┼────────────────────────────────────────────┤         │
│  │Valid=0│ Disk address (page is on disk)            │         │
│  └───────┴────────────────────────────────────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Valid-Invalid Bit

```
Page Table with valid bits:

┌───────┬─────────┬────────────────────────────┐
│ Page  │  Valid  │       Frame / Disk         │
├───────┼─────────┼────────────────────────────┤
│   0   │    1    │       Frame 4              │  ← In memory
│   1   │    0    │       Disk block 127       │  ← On disk
│   2   │    1    │       Frame 6              │  ← In memory
│   3   │    1    │       Frame 2              │  ← In memory
│   4   │    0    │       Disk block 342       │  ← On disk
│   5   │    0    │       Never allocated      │  ← Invalid
└───────┴─────────┴────────────────────────────┘

Accessing page 1 → Valid bit = 0 → PAGE FAULT!
```

---

## Page Fault Handling

```
┌──────────────────────────────────────────────────────────────────┐
│                  PAGE FAULT HANDLING                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. Process references a page                                   │
│   2. Check page table: Valid bit = 0                            │
│   3. TRAP to OS: Page Fault                                     │
│                                                                  │
│   4. OS checks internal table:                                   │
│      - Invalid reference? → Terminate process                   │
│      - Valid but not in memory? → Continue                      │
│                                                                  │
│   5. Find a free frame                                           │
│      - If none free, use page replacement algorithm             │
│                                                                  │
│   6. Schedule disk read to load page into frame                 │
│                                                                  │
│   7. While waiting for I/O:                                      │
│      - Block this process                                       │
│      - CPU switches to another process                          │
│                                                                  │
│   8. When I/O complete (interrupt):                             │
│      - Update page table (valid=1, frame#)                      │
│      - Mark process ready                                       │
│                                                                  │
│   9. Restart the instruction that caused fault                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Page Fault Diagram

```
                                    ┌───────────────┐
        ┌──────────┐    reference   │               │
        │   CPU    │ ──────────────→│ Page Table    │
        └────┬─────┘       1        │               │
             │                      └───────┬───────┘
             │                              │ 2
             │                              ▼
             │                      ┌───────────────┐
             │                      │  Valid = 0?   │
             │                      │  PAGE FAULT   │
             │                      └───────┬───────┘
             │                              │ 3
             │                              ▼
             │                      ┌───────────────┐
             │    6 restart         │      OS       │
             │←───────────────────  │   Handler     │
             │                      └───────┬───────┘
                                            │ 4
                                            ▼
                                    ┌───────────────┐
                                    │ Find free     │
                                    │ frame         │
                                    └───────┬───────┘
                                            │ 5
                                            ▼
    ┌─────────────────┐             ┌───────────────┐
    │  Physical       │←────────────│  Read page    │
    │  Memory         │  load       │  from disk    │
    │ ┌─────────────┐ │             └───────────────┘
    │ │  New page   │ │                    ↑
    │ └─────────────┘ │                    │
    └─────────────────┘             ┌──────┴────────┐
                                    │    DISK       │
                                    │ (Swap Space)  │
                                    └───────────────┘
```

---

## Performance of Demand Paging

### Effective Access Time

```
Let:
  p = page fault rate (0 ≤ p ≤ 1)
  ma = memory access time
  pft = page fault time (disk I/O)

Effective Access Time (EAT):
  EAT = (1 - p) × ma + p × pft

Example:
  ma = 100 ns
  pft = 10 ms = 10,000,000 ns
  p = 1/1000 = 0.001

  EAT = (0.999 × 100) + (0.001 × 10,000,000)
      = 99.9 + 10,000
      = 10,099.9 ns

  That's 100x slower than no paging!

For less than 10% slowdown:
  EAT < 110 ns
  (1-p)(100) + p(10,000,000) < 110
  Solving: p < 0.000001 (one in a million!)

Conclusion: Page faults are VERY expensive, must be rare
```

---

## Copy-on-Write (COW)

```
┌──────────────────────────────────────────────────────────────────┐
│                    COPY-ON-WRITE                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When fork() creates child process:                             │
│  • Don't copy all pages immediately                             │
│  • Both parent and child share same pages (marked read-only)    │
│  • Only copy a page when it's written to                        │
│                                                                  │
│  Before write:                       After P2 writes:            │
│                                                                  │
│  P1's Page Table    Physical         P1's Page Table             │
│  ┌─────┬──────┐    Memory            ┌─────┬──────┐             │
│  │  0  │  F3  │───→┌────┐            │  0  │  F3  │───→┌────┐   │
│  │  1  │  F5  │──┐ │ F3 │            │  1  │  F5  │──┐ │ F3 │   │
│  └─────┴──────┘  │ ├────┤            └─────┴──────┘  │ ├────┤   │
│                  └→│ F5 │ (shared)                   │ │ F5 │   │
│  P2's Page Table   ├────┤            P2's Page Table │ ├────┤   │
│  ┌─────┬──────┐    │    │            ┌─────┬──────┐  │ │ F7 │ ← │
│  │  0  │  F3  │───→└────┘            │  0  │  F3  │──┘ └────┘   │
│  │  1  │  F5  │────────┘             │  1  │  F7  │───→ (copy)  │
│  └─────┴──────┘                      └─────┴──────┘             │
│                                                                  │
│  Benefits:                                                       │
│  • Fast process creation (fork)                                 │
│  • Memory efficient (share unchanged pages)                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Page Replacement

When no free frames available, must replace an existing page.

### Page Replacement Algorithm Goals

```
Goal: Minimize page faults

Metrics:
• Page fault rate
• Number of page faults for a given reference string

Reference String: Sequence of page numbers accessed
Example: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
```

### Basic Page Replacement

```
1. Find the desired page on disk
2. Find a free frame:
   a. If free frame exists, use it
   b. If no free frame, select victim using replacement algorithm
   c. If victim is dirty (modified), write it to disk
3. Load desired page into the newly freed frame
4. Update page table
5. Restart the instruction
```

---

## Page Replacement Algorithms

### 1. FIFO (First-In-First-Out)

```
Replace the oldest page in memory

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Frames: 3

Step   Ref   Frame1  Frame2  Frame3   Fault?
 1      7      7       -       -        F
 2      0      7       0       -        F
 3      1      7       0       1        F
 4      2      2       0       1        F (replace 7)
 5      0      2       0       1        
 6      3      2       3       1        F (replace 0)
 7      0      2       3       0        F (replace 1)
 8      4      4       3       0        F (replace 2)
 9      2      4       2       0        F (replace 3)
10      3      4       2       3        F (replace 0)
11      0      0       2       3        F (replace 4)
12      3      0       2       3        
13      2      0       2       3        
14      1      1       2       3        F (replace 0)
15      2      1       2       3        
...

Total Page Faults: 15

Simple but not optimal - may replace frequently used pages
```

**Belady's Anomaly:**
```
FIFO can have MORE faults with MORE frames!

Example reference string with 3 frames: 9 faults
Same string with 4 frames: 10 faults

This counterintuitive behavior is Belady's anomaly.
```

### 2. Optimal Algorithm (OPT)

```
Replace the page that will not be used for the longest time

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Frames: 3

At each replacement, look FORWARD in reference string
Choose page that appears furthest away (or never)

Step   Ref   Frame1  Frame2  Frame3   Fault?   Victim
 1      7      7       -       -        F
 2      0      7       0       -        F
 3      1      7       0       1        F
 4      2      2       0       1        F       7 (furthest)
 5      0      2       0       1        
 6      3      2       0       3        F       1 (not used until later)
 7      0      2       0       3        
 8      4      2       4       3        F       0 (choose wisely)
 9      2      2       4       3        
10      3      2       4       3        
...

Total Page Faults: 9 (minimum possible!)

Problem: Requires future knowledge - impossible to implement!
Used as benchmark to compare other algorithms.
```

### 3. LRU (Least Recently Used)

```
Replace page that hasn't been used for longest time
Approximation of optimal using past instead of future

Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Frames: 3

Step   Ref   Frame1  Frame2  Frame3   Fault?   LRU Order
 1      7      7       -       -        F       7
 2      0      7       0       -        F       7,0
 3      1      7       0       1        F       7,0,1
 4      2      2       0       1        F       0,1,2 (7 was LRU)
 5      0      2       0       1              1,2,0
 6      3      2       0       3        F       2,0,3 (1 was LRU)
 7      0      2       0       3              2,3,0
 8      4      4       0       3        F       3,0,4 (2 was LRU)
 9      2      4       0       2        F       0,4,2 (3 was LRU)
10      3      4       3       2        F       4,2,3 (0 was LRU)
11      0      0       3       2        F       2,3,0 (4 was LRU)
12      3      0       3       2              2,0,3
...

Total Page Faults: 12

Good performance but needs hardware support for timing.
```

### 4. LRU Approximations

**Reference Bit Algorithm:**
```
Each page has a reference bit (set by hardware on access)
Periodically:
  - Find page with reference bit = 0 (not recently used)
  - Clear all reference bits

Simple but only distinguishes "used" vs "not used"
```

**Second Chance (Clock) Algorithm:**
```
┌────────────────────────────────────────────────────────────────┐
│                   CLOCK ALGORITHM                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Pages arranged in circular queue (clock)                      │
│  Hand points to oldest page                                    │
│                                                                │
│          ┌───┐                                                 │
│    ┌────→│ A │ R=1                                            │
│    │     │   │←──┐                                            │
│    │     └───┘   │                                            │
│  ┌─┴─┐         ┌─┴─┐                                          │
│  │ E │ R=0     │ B │ R=0                                       │
│  │   │         │   │                                          │
│  └───┘         └───┘                                          │
│    ↑             ↓                                             │
│  ┌───┐         ┌───┐                                          │
│  │ D │ R=1     │ C │ R=1                                       │
│  │   │←───────→│   │                                          │
│  └───┘   Hand  └───┘                                          │
│                                                                │
│  When replacement needed:                                      │
│  1. Check page at hand                                        │
│  2. If R=0: Replace this page                                 │
│  3. If R=1: Set R=0, advance hand, repeat                    │
│                                                                │
│  "Second chance" - pages with R=1 get another chance         │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Enhanced Second Chance:**
```
Use both reference (R) and modify (M) bits

Priority for replacement (prefer lower number):
1. (0,0): Not used, not modified - best victim
2. (0,1): Not used, modified - must write back
3. (1,0): Used, not modified - probably needed soon
4. (1,1): Used, modified - worst victim

Scan multiple times to find best category.
```

---

## Thrashing

```
┌──────────────────────────────────────────────────────────────────┐
│                      THRASHING                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: Process spends more time paging than executing    │
│                                                                  │
│  Cause: Too many processes, not enough frames each             │
│                                                                  │
│  Symptoms:                                                       │
│  • High page fault rate                                        │
│  • Low CPU utilization                                         │
│  • Disk always busy (paging I/O)                               │
│                                                                  │
│  CPU                                                            │
│  Util  ^                                                        │
│  100% │      ●───●                                             │
│       │    ●       ●                                           │
│       │   ●         ●                                          │
│       │  ●           ● Thrashing                               │
│       │ ●             ● starts                                  │
│       │●               ●●●●●                                   │
│       └─────────────────────────→                              │
│            Degree of Multiprogramming                           │
│                                                                  │
│  Solution:                                                       │
│  • Give each process minimum needed frames                     │
│  • Use working set model                                       │
│  • Reduce degree of multiprogramming                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Working Set Model

```
┌──────────────────────────────────────────────────────────────────┐
│                    WORKING SET MODEL                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Working Set = Set of pages used in the last Δ time units      │
│  (Δ = working set window)                                       │
│                                                                  │
│  Reference string:  ...2, 6, 1, 5, 7, 7, 7, 7, 5, 1...         │
│                              ├─────── Δ=10 ───────┤             │
│                              Working Set = {1,2,5,6,7}          │
│                                                                  │
│  WS(t, Δ) = set of pages in most recent Δ references           │
│                                                                  │
│  Policy:                                                         │
│  • Allocate frames = |WS| (working set size)                   │
│  • If Σ|WSi| > total frames: Suspend some processes            │
│  • Prevents thrashing                                           │
│                                                                  │
│  Choosing Δ:                                                     │
│  • Too small: Won't capture full working set                   │
│  • Too large: May include old pages                            │
│  • Typical: 10,000 to 1,000,000 references                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Virtual Memory** | Logical memory larger than physical |
| **Demand Paging** | Load pages only when needed |
| **Page Fault** | Access to page not in memory |
| **Dirty Bit** | Page modified, needs write-back |
| **FIFO** | Replace oldest page |
| **OPT** | Replace page used furthest in future |
| **LRU** | Replace least recently used page |
| **Clock** | Second-chance approximation of LRU |
| **Thrashing** | Excessive paging, low CPU utilization |
| **Working Set** | Pages needed at any time |

---

## Quick Revision Questions

1. What triggers a page fault?
2. Calculate EAT given page fault rate and times.
3. Explain Belady's anomaly in FIFO.
4. How does the clock algorithm work?
5. What causes thrashing and how can it be prevented?

---

[← Previous: Segmentation](./03-segmentation.md) | [Next: Page Replacement Algorithms →](./05-page-replacement.md)
