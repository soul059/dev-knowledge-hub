# Chapter 6: External Sorting

## Overview

When data is too large to fit in memory, we need **external sorting** — sorting that efficiently uses disk I/O. **External Merge Sort** is the standard approach, directly applying the merge sort paradigm to out-of-core data.

---

## The Problem

```
┌──────────────────────────────────────────────────────┐
│  Scenario:                                            │
│    • 100 GB of data to sort                           │
│    • Only 1 GB of RAM available                       │
│    • Data resides on disk (slow random access)        │
│                                                       │
│  Challenge:                                           │
│    • Can't load all data into memory                  │
│    • Disk I/O is ~100,000× slower than RAM access     │
│    • Must minimize number of disk reads/writes        │
└──────────────────────────────────────────────────────┘
```

---

## External Merge Sort: Two Phases

```
PHASE 1: CREATE SORTED RUNS
┌──────────────────────────────────────────────────────┐
│  1. Read a chunk that fits in memory (1 GB)          │
│  2. Sort it in-memory (using any O(n log n) sort)    │
│  3. Write sorted chunk ("run") to disk               │
│  4. Repeat for all chunks                            │
│                                                      │
│  100 GB ÷ 1 GB = 100 sorted runs                   │
└──────────────────────────────────────────────────────┘

         Unsorted File (100 GB)
    ┌────┬────┬────┬────┬─────┬────┐
    │ C1 │ C2 │ C3 │ C4 │ ... │C100│
    └────┴────┴────┴────┴─────┴────┘
              ↓  Sort each in RAM
    ┌────┬────┬────┬────┬─────┬────┐
    │ R1 │ R2 │ R3 │ R4 │ ... │R100│  (sorted runs)
    └────┴────┴────┴────┴─────┴────┘


PHASE 2: MERGE RUNS (K-WAY MERGE)
┌──────────────────────────────────────────────────────┐
│  Merge all runs using K-way merge:                   │
│  1. Open all K run files simultaneously              │
│  2. Read one block from each run into memory         │
│  3. Use min-heap to find smallest element across K   │
│  4. Write output to result file                      │
│  5. When a block is exhausted, read next block       │
└──────────────────────────────────────────────────────┘
```

---

## K-Way Merge Visualization

```
Memory Layout during K-Way Merge:

┌─────────────────────────────────────────────────┐
│                    RAM (1 GB)                     │
│                                                  │
│  ┌──────┐ ┌──────┐ ┌──────┐      ┌──────┐      │
│  │Input │ │Input │ │Input │  ... │Input │      │
│  │Buf 1 │ │Buf 2 │ │Buf 3 │      │Buf K │      │
│  └──┬───┘ └──┬───┘ └──┬───┘      └──┬───┘      │
│     │        │        │              │           │
│     └────────┴────┬───┴──────────────┘           │
│                   │                              │
│              ┌────▼────┐                         │
│              │Min-Heap │                         │
│              │ (K els) │                         │
│              └────┬────┘                         │
│                   │                              │
│              ┌────▼────┐                         │
│              │ Output  │                         │
│              │ Buffer  │ → write to disk when full│
│              └─────────┘                         │
└─────────────────────────────────────────────────┘
```

---

## Memory Budget Calculation

```
Total RAM: M bytes
K sorted runs to merge

Buffer size per run: M / (K + 1)
  → K input buffers + 1 output buffer

Example:
  M = 1 GB, K = 100 runs
  Buffer per run = 1 GB / 101 ≈ 10 MB each

If K is too large (buffers too small):
  → Use multi-pass merging
```

---

## Multi-Pass Merge

When K is too large for single-pass merge:

```
100 runs, but can only merge 10 at a time:

Pass 1: Merge runs in groups of 10
  [R1..R10]   → M1
  [R11..R20]  → M2
  ...
  [R91..R100] → M10
  
  Result: 10 merged runs

Pass 2: Merge all 10
  [M1..M10] → Final sorted output

Total passes: ⌈log_K(R)⌉ where R = number of initial runs
```

---

## I/O Complexity Analysis

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1 (Create runs):                                     │
│    Read: N/B disk reads   (B = block size)                  │
│    Write: N/B disk writes                                   │
│    Total: 2N/B I/O operations                               │
│                                                             │
│  Phase 2 (K-way merge, single pass):                        │
│    Read: N/B disk reads                                     │
│    Write: N/B disk writes                                   │
│    Total: 2N/B I/O operations                               │
│                                                             │
│  Multi-pass (P passes):                                     │
│    Total I/O: 2(P+1) × N/B                                │
│    P = ⌈log_K(R)⌉ passes                                  │
│    R = N/M runs, K = M/B - 1                               │
│                                                             │
│  Total I/O = O((N/B) × log_{M/B}(N/M))                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Optimizations

```
1. REPLACEMENT SELECTION (create longer runs)
   → Use a priority queue during Phase 1
   → Average run length ≈ 2M instead of M
   → Fewer runs → fewer merge passes

2. DOUBLE BUFFERING
   → While processing one buffer, prefetch the next
   → Overlaps I/O with computation
   
3. FORECASTING
   → Predict which run will be needed next
   → Pre-read its next block

4. POLYPHASE MERGE
   → Optimize for limited number of disk drives
   → Uses Fibonacci-like distribution of runs
```

---

## Real-World Usage

```
┌──────────────────────────────────────────────────────────┐
│  External merge sort is used in:                         │
│                                                          │
│  • Database systems (ORDER BY on large tables)           │
│  • Unix 'sort' command (for files larger than RAM)       │
│  • MapReduce (shuffle/sort phase)                        │
│  • Data warehouse ETL processes                          │
│  • Generating sorted indexes                             │
│  • Merging log files from multiple sources               │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Phase 1 | Create sorted runs that fit in memory |
| Phase 2 | K-way merge using min-heap |
| Buffer management | M / (K+1) per buffer |
| Multi-pass | When K > available buffers |
| I/O complexity | O((N/B) × log_{M/B}(N/M)) |
| Replacement selection | Creates runs ~2× memory size |
| Key metric | Minimize disk I/O, not comparisons |

---

## Quick Revision Questions

1. **Why can't we use standard in-memory sort for 100 GB of data on a 1 GB machine?**
2. **What are the two phases of external merge sort?**
3. **How does the K-way merge use a min-heap?**
4. **When is multi-pass merging necessary?**
5. **What is replacement selection and how does it help?**

---

[← Previous: Merge Sort Variations](05-variations.md) | [Back to Unit 4](../README.md)
