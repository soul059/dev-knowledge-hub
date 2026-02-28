# Unit 6: Quick Sort vs Merge Sort — Detailed Comparison

[← Previous: Complexity Analysis](05-complexity-analysis.md) | [Next: Unit 7 — Heap Sort →](../07-Heap-Sort/01-algorithm-description.md)

---

## Overview

Quick Sort and Merge Sort are both O(n log n) divide-and-conquer algorithms, but they make fundamentally different trade-offs. This chapter provides a deep, practical comparison to help you choose the right one.

---

## Head-to-Head Comparison

```
  ┌────────────────────┬──────────────────────┬──────────────────────┐
  │  Criterion         │  Quick Sort          │  Merge Sort          │
  ├────────────────────┼──────────────────────┼──────────────────────┤
  │  Worst-case time   │  O(n²)               │  O(n log n)  ✓      │
  │  Average time      │  O(1.39n log n)      │  O(n log n)          │
  │  Best-case time    │  O(n log n)          │  O(n log n)          │
  │  Space             │  O(log n)  ✓         │  O(n)               │
  │  In-place          │  Yes  ✓              │  No                  │
  │  Stable            │  No                  │  Yes  ✓              │
  │  Cache performance │  Excellent  ✓        │  Good                │
  │  Practical speed   │  Fastest  ✓          │  ~30% slower         │
  │  Worst-case safety │  No (without intro)  │  Yes  ✓              │
  │  For arrays        │  Excellent  ✓        │  Good                │
  │  For linked lists  │  Poor                │  Excellent  ✓        │
  │  Parallelizable    │  Moderate            │  Excellent  ✓        │
  │  External sorting  │  Poor                │  Excellent  ✓        │
  │  Adaptive          │  No                  │  No (basic)          │
  └────────────────────┴──────────────────────┴──────────────────────┘
```

---

## Why Quick Sort Is Faster in Practice

```
  Despite same O(n log n) average, Quick Sort is ~2-3x faster:
  
  1. CACHE LOCALITY
  ┌──────────────────────────────────────────────────────────┐
  │  Quick Sort: accesses elements SEQUENTIALLY within      │
  │  the partition. Everything stays in CPU cache.          │
  │                                                          │
  │  Merge Sort: copies data between main array and aux     │
  │  array. Cache misses when switching between them.       │
  │                                                          │
  │  Cache hit rate:                                         │
  │  Quick Sort: ████████████████████  ~95%                 │
  │  Merge Sort: █████████████         ~75%                 │
  └──────────────────────────────────────────────────────────┘
  
  2. FEWER DATA MOVEMENTS
  ┌──────────────────────────────────────────────────────────┐
  │  Quick Sort: swaps in-place (no copy to/from aux)      │
  │  Merge Sort: copies EVERY element to aux, then back    │
  │                                                          │
  │  Data movements per element:                            │
  │  Quick Sort: ~1 swap per comparison                     │
  │  Merge Sort: 2 copies per level × log(n) levels        │
  └──────────────────────────────────────────────────────────┘
  
  3. SIMPLER INNER LOOP
  ┌──────────────────────────────────────────────────────────┐
  │  Quick Sort partition: compare + maybe swap              │
  │  Merge: compare + copy + index management               │
  │                                                          │
  │  Quick Sort inner loop compiles to ~5 instructions      │
  │  Merge inner loop compiles to ~8 instructions           │
  └──────────────────────────────────────────────────────────┘
```

---

## When to Choose Merge Sort

```
  ┌──────────────────────────────────────────────────────────┐
  │  USE MERGE SORT when:                                   │
  │                                                          │
  │  1. STABILITY required                                  │
  │     Sorting objects by multiple keys                    │
  │     Example: Sort students by name, then by grade      │
  │                                                          │
  │  2. WORST-CASE GUARANTEE required                       │
  │     Real-time systems, safety-critical code             │
  │     Cannot afford O(n²) ever                            │
  │                                                          │
  │  3. LINKED LISTS                                        │
  │     Merge sort on linked lists is O(1) extra space!    │
  │     Quick Sort needs random access (poor for lists)     │
  │                                                          │
  │  4. EXTERNAL SORTING (data on disk)                     │
  │     Natural fit for merge-based external sorting        │
  │     Data doesn't fit in RAM                             │
  │                                                          │
  │  5. PARALLEL SORTING                                    │
  │     Merge sort parallelizes beautifully                 │
  │     Independent halves can be sorted concurrently       │
  └──────────────────────────────────────────────────────────┘
```

---

## When to Choose Quick Sort

```
  ┌──────────────────────────────────────────────────────────┐
  │  USE QUICK SORT when:                                   │
  │                                                          │
  │  1. SPEED is top priority                               │
  │     ~2-3x faster than merge sort in practice            │
  │                                                          │
  │  2. MEMORY is constrained                               │
  │     O(log n) vs O(n) — huge difference for large n      │
  │     Sorting 1GB of data: Quick Sort needs KB extra,     │
  │     Merge Sort needs another 1GB!                       │
  │                                                          │
  │  3. IN-PLACE required                                    │
  │     Embedded systems, tight memory environments         │
  │                                                          │
  │  4. ARRAYS (random access data)                          │
  │     Partition relies on random access → great for arrays│
  │                                                          │
  │  5. AVERAGE-CASE matters (not worst case)               │
  │     Most applications — worst case rarely occurs        │
  │     with randomized pivot                               │
  └──────────────────────────────────────────────────────────┘
```

---

## What Real Languages Use

```
  ┌──────────────────────────────────────────────────────────┐
  │  JAVA:                                                   │
  │    primitives → Dual-Pivot Quick Sort                   │
  │    objects    → TimSort (merge-based)  ← stability!    │
  │                                                          │
  │  PYTHON:                                                 │
  │    All types → TimSort (merge-based)  ← stability!     │
  │                                                          │
  │  C++:                                                    │
  │    std::sort → Introsort (Quick + Heap + Insertion)     │
  │    std::stable_sort → Merge Sort                        │
  │                                                          │
  │  JAVASCRIPT:                                             │
  │    V8 → TimSort                                          │
  │                                                          │
  │  RUST:                                                   │
  │    unstable sort → Pattern-defeating Quick Sort         │
  │    stable sort → Merge Sort variant                     │
  │                                                          │
  │  Pattern: Languages offer BOTH options!                 │
  │  Quick Sort for speed, Merge Sort for stability.        │
  └──────────────────────────────────────────────────────────┘
```

---

## Decision Flowchart

```
              Need to sort data?
                    │
                    ▼
          ┌─────────────────┐
          │  Stability      │── YES ──→ MERGE SORT (or TimSort)
          │  required?      │
          └────────┬────────┘
                   │ NO
                   ▼
          ┌─────────────────┐
          │  Linked list?   │── YES ──→ MERGE SORT
          └────────┬────────┘
                   │ NO (array)
                   ▼
          ┌─────────────────┐
          │  External data? │── YES ──→ MERGE SORT
          │  (too big for   │
          │   RAM)          │
          └────────┬────────┘
                   │ NO
                   ▼
          ┌─────────────────────┐
          │  Worst-case O(nlogn)│── YES ──→ INTROSORT
          │  guarantee needed?  │          (Quick+Heap)
          └────────┬────────────┘
                   │ NO
                   ▼
              QUICK SORT
              (fastest on average)
```

---

## Benchmark-Style Comparison

```
  Sorting 10,000,000 random integers:
  
  Quick Sort:     ██████████████████         1.0x  (baseline)
  Merge Sort:     ████████████████████████   1.3x
  Heap Sort:      ██████████████████████████ 1.7x
  Introsort:      ██████████████████         1.0x
  TimSort:        ███████████████████████    1.2x
  
  Sorting 10,000,000 nearly sorted integers:
  
  TimSort:        ██████                     0.3x  ← FASTEST
  Insertion Sort: ████████                   0.4x  (for small n)
  Quick Sort:     ██████████████████         1.0x
  Merge Sort:     ████████████████████████   1.3x
  Heap Sort:      ██████████████████████████ 1.7x
  
  Sorting 10,000,000 integers with many duplicates:
  
  3-Way Quick:    ████████████               0.6x  ← FASTEST
  Quick Sort:     ██████████████████████████ 1.5x  (unbalanced!)
  Merge Sort:     ████████████████████████   1.3x
```

---

## Summary Table

| Factor | Winner | Why |
|--------|--------|-----|
| Raw speed (arrays) | Quick Sort | Cache locality, fewer moves |
| Memory usage | Quick Sort | O(log n) vs O(n) |
| Worst-case guarantee | Merge Sort | Always O(n log n) |
| Stability | Merge Sort | Natural property |
| Linked lists | Merge Sort | O(1) space, no random access needed |
| External sorting | Merge Sort | Sequential I/O pattern |
| Duplicates | 3-Way Quick Sort | Linear for all-same |
| Small arrays | Insertion Sort | Both switch to it |
| Parallelism | Merge Sort | Independent subproblems |

---

## Quick Revision Questions

1. **Why is Quick Sort ~2-3x faster than Merge Sort in practice?**
2. **When would you choose Merge Sort over Quick Sort?**
3. **Why does Java use Quick Sort for primitives but Merge Sort for objects?**
4. **What is Introsort and why was it created?**
5. **For sorting 1 million records on a system with 2MB RAM, which would you choose?**
6. **Why is Quick Sort poor for linked lists but Merge Sort excels?**

---

[← Previous: Complexity Analysis](05-complexity-analysis.md) | [Next: Unit 7 — Heap Sort →](../07-Heap-Sort/01-algorithm-description.md)
