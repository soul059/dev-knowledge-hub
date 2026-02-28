# Chapter 5: Internal vs External Sorting

[← Previous: Comparison vs Non-comparison](04-comparison-vs-noncomparison.md) | [Next: Sorting Complexity Lower Bound →](06-sorting-lower-bound.md)

---

## Overview

When data is too large to fit in main memory (RAM), we can't use regular sorting algorithms. This leads to the distinction between **internal sorting** (data fits in RAM) and **external sorting** (data resides on disk/external storage).

---

## Internal Sorting

All data resides in **main memory (RAM)** during the sorting process.

```
  INTERNAL SORTING:
  
  ┌─────────────────────────────┐
  │          RAM                │
  │  ┌───┬───┬───┬───┬───────┐ │
  │  │ 5 │ 3 │ 1 │ 4 │ ...   │ │  ← Entire dataset fits in memory
  │  └───┴───┴───┴───┴───────┘ │
  │                             │
  │  Sorting happens HERE       │
  │  Random access: O(1)       │
  │  Fast swaps and comparisons│
  └─────────────────────────────┘
```

### Characteristics
- **Fast random access** — any element accessible in O(1)
- All standard sorting algorithms work here
- Data size limited by available RAM (typically GB range)
- No I/O overhead

### Algorithms
- Bubble, Selection, Insertion Sort
- Merge Sort, Quick Sort, Heap Sort
- Counting, Radix, Bucket Sort

---

## External Sorting

Data is too large for RAM and resides on **external storage** (hard disk, SSD, distributed storage).

```
  EXTERNAL SORTING:
  
  ┌──────────────┐         ┌──────────────────────────────────┐
  │     RAM      │         │        DISK / SSD                │
  │              │         │                                  │
  │  ┌─────────┐ │  read   │  ┌───┬───┬───┬───┬───┬───┬───┐  │
  │  │ buffer  │◄├─────────┤──│ 5 │ 3 │ 1 │ 4 │ 2 │ 8 │...│  │
  │  │ (small) │ │         │  └───┴───┴───┴───┴───┴───┴───┘  │
  │  └─────────┘ │  write  │                                  │
  │       │      ├─────────┤► Sorted chunks written back      │
  │              │         │                                  │
  │  Limited     │         │  Terabytes of data               │
  │  capacity    │         │  Sequential access preferred     │
  └──────────────┘         └──────────────────────────────────┘
  
  Key challenge: Disk I/O is 100,000× slower than RAM access!
```

### The External Merge Sort Algorithm

The most common external sorting technique is **External Merge Sort**:

```
  PHASE 1: CREATE SORTED RUNS
  ═══════════════════════════
  
  Disk Data: [5,3,1,4,2,8,7,6,9,0,...]    (too big for RAM)
  RAM size: can hold 4 elements
  
  Read chunk 1 → RAM: [5,3,1,4] → Sort → Write: [1,3,4,5] → Run 1
  Read chunk 2 → RAM: [2,8,7,6] → Sort → Write: [2,6,7,8] → Run 2
  Read chunk 3 → RAM: [9,0,...] → Sort → Write: [0,9,...] → Run 3
  
  Disk after Phase 1:
  ┌─────────────┬─────────────┬─────────────┐
  │ Run 1       │ Run 2       │ Run 3       │
  │ [1,3,4,5]   │ [2,6,7,8]   │ [0,9,...]   │
  └─────────────┴─────────────┴─────────────┘
  Each run is sorted internally
  
  
  PHASE 2: MERGE RUNS (K-way merge)
  ══════════════════════════════════
  
  ┌──────────────────────────────────────────┐
  │  RAM                                     │
  │                                          │
  │  Input Buffer 1: [1, ...]  ← from Run 1 │
  │  Input Buffer 2: [2, ...]  ← from Run 2 │
  │  Input Buffer 3: [0, ...]  ← from Run 3 │
  │                                          │
  │  Output Buffer:  [0, 1, 2, ...]          │
  │     (flush to disk when full)            │
  └──────────────────────────────────────────┘
  
  Compare heads of all buffers → pick smallest → write to output
  Repeat until all runs merged
  
  Final sorted output: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, ...]
```

### Step-by-Step K-Way Merge

```
  Merging 3 sorted runs with 3 input buffers + 1 output buffer:
  
  Run 1: [1, 3, 5, 9]
  Run 2: [2, 4, 6, 8]
  Run 3: [0, 7, 10, 11]
  
  Step  │ Buf1 │ Buf2 │ Buf3 │ Output
  ──────┼──────┼──────┼──────┼────────────────
    1   │  1   │  2   │  0*  │ [0]              ← min is 0 from Buf3
    2   │  1   │  2   │  7   │ [0, 1]           ← min is 1 from Buf1
    3   │  3   │  2*  │  7   │ [0, 1, 2]        ← min is 2 from Buf2
    4   │  3*  │  4   │  7   │ [0, 1, 2, 3]     ← min is 3 from Buf1
    5   │  5   │  4*  │  7   │ [0, 1, 2, 3, 4]  ← min is 4 from Buf2
    ...
  
  * = element chosen (minimum)
  Uses a MIN-HEAP for efficient selection: O(log k) per element
```

---

## Comparison

| Property | Internal Sorting | External Sorting |
|----------|-----------------|------------------|
| **Data location** | RAM | Disk / External storage |
| **Data size** | Fits in memory | Larger than memory |
| **Access pattern** | Random access O(1) | Sequential preferred |
| **I/O cost** | Negligible | Dominant factor |
| **Algorithms** | All standard sorts | External merge sort |
| **Optimization goal** | Minimize comparisons | Minimize disk I/O |
| **Typical use** | In-memory applications | Big data, databases |

---

## I/O Complexity

```
  Internal sorting focuses on:
  ┌──────────────────────────────────┐
  │  # of comparisons / swaps       │
  │  Time: O(n log n)               │
  └──────────────────────────────────┘
  
  External sorting focuses on:
  ┌──────────────────────────────────┐
  │  # of disk reads / writes       │
  │  I/O: O((n/B) × logₖ(n/M))     │
  │                                 │
  │  B = block size (disk page)     │
  │  M = memory size                │
  │  k = merge factor (# of runs)  │
  │  n = total data size            │
  └──────────────────────────────────┘
  
  Why disk I/O dominates:
  ┌────────────┬────────────────┐
  │ Operation  │ Time           │
  ├────────────┼────────────────┤
  │ RAM access │ ~100 ns        │
  │ SSD access │ ~100,000 ns    │  ← 1,000× slower
  │ HDD access │ ~10,000,000 ns │  ← 100,000× slower
  └────────────┴────────────────┘
```

---

## Real-World Applications

```
  External sorting is used when:
  
  ┌─────────────────────────────────────────────────────┐
  │  • Sorting 1 TB of log files on a 16 GB RAM server │
  │  • Database index creation                         │
  │  • MapReduce shuffle phase (Hadoop, Spark)          │
  │  • Sorting genomic data (human genome ~3 GB)        │
  │  • Large-scale data analytics                      │
  └─────────────────────────────────────────────────────┘
```

---

## Optimization Techniques for External Sort

1. **Replacement Selection**: Create initial runs longer than RAM (up to 2× on average)
2. **Multi-way Merge**: Merge k runs simultaneously to reduce passes
3. **Double Buffering**: Read next block while processing current one
4. **Compression**: Compress data to reduce I/O volume

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Internal sorting | Data fits in RAM, random access |
| External sorting | Data on disk, sequential access preferred |
| External merge sort | Two phases: create runs, then merge |
| K-way merge | Uses min-heap to merge k sorted runs |
| Optimization goal | Minimize disk I/O operations |
| Disk vs RAM speed | Disk is 1,000–100,000× slower |
| Applications | Big data, databases, distributed systems |

---

## Quick Revision Questions

1. **When do we need external sorting instead of internal sorting?**
2. **Describe the two phases of External Merge Sort.**
3. **Why is disk I/O the dominant cost in external sorting?**
4. **What data structure helps efficiently merge k sorted runs?**
5. **How does the K-way merge factor affect the number of passes?**
6. **Name two optimization techniques for external sorting.**

---

[← Previous: Comparison vs Non-comparison](04-comparison-vs-noncomparison.md) | [Next: Sorting Complexity Lower Bound →](06-sorting-lower-bound.md)
