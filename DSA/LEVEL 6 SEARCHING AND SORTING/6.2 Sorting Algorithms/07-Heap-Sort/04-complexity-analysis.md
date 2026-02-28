# Unit 7: Heap Sort — Complexity Analysis

[← Previous: Implementation](03-implementation.md) | [Next: Applications →](05-heap-sort-applications.md)

---

## Overview

Heap Sort guarantees **O(n log n)** in ALL cases — best, average, and worst. This makes it unique among comparison-based sorts. This chapter provides rigorous analysis of both phases.

---

## Phase 1: Build Max-Heap — O(n)

```
  BUILD-HEAP runs heapify on all non-leaf nodes.
  
  Key insight: NOT all heapify calls cost O(log n)!
  
  For a complete binary tree with n nodes:
  
  Height h    Nodes at height    Max sift-down swaps
  ─────────   ─────────────────  ───────────────────
    0 (leaf)     ⌈n/2⌉           0 (skipped)
    1            ⌈n/4⌉           1
    2            ⌈n/8⌉           2
    3            ⌈n/16⌉          3
    ...          ...              ...
    k            ⌈n/2^(k+1)⌉    k
    ...          ...              ...
    log n        1               log n
  
  Total work = Σ(h=0 to log n) ⌈n/2^(h+1)⌉ × h
             ≤ n × Σ(h=0 to ∞) h/2^h
             = n × 2          (by the identity Σ h·x^h = x/(1-x)² at x=1/2)
             = O(n)
```

### Mathematical Proof

```
  Σ(h=1 to ∞) h/2^h  =  Σ(h=1 to ∞) h · (1/2)^h
  
  Using the identity: Σ(h=1 to ∞) h · x^h = x/(1-x)²
  
  At x = 1/2:
  = (1/2) / (1 - 1/2)²
  = (1/2) / (1/4)
  = 2
  
  Therefore: Total work = n/2 × 2 = n = O(n)
  
  BUILD-HEAP is LINEAR! ✓
```

### Why This Is Counter-Intuitive

```
  ┌──────────────────────────────────────────────────────────┐
  │  "But we call heapify n/2 times and each is O(log n)!" │
  │                                                          │
  │  True, but this is a LOOSE upper bound.                 │
  │                                                          │
  │  The TIGHT analysis reveals:                            │
  │  - n/2 nodes (leaves) do 0 work                        │
  │  - n/4 nodes do at most 1 swap                         │
  │  - n/8 nodes do at most 2 swaps                        │
  │  - Only 1 node (root) does log n swaps                 │
  │                                                          │
  │  Most nodes do very little work!                        │
  │                                                          │
  │  Analogy: If a company has 1000 employees:              │
  │  - 500 interns do 0 hours overtime                      │
  │  - 250 juniors do 1 hour overtime                       │
  │  - 125 mid-level do 2 hours                             │
  │  - 1 CEO does 10 hours                                  │
  │  Total ≈ 1000 hours, not 1000 × 10 = 10000             │
  └──────────────────────────────────────────────────────────┘
```

---

## Phase 2: Sorting (Extract-Max) — O(n log n)

```
  In Phase 2, we extract max n-1 times.
  Each extraction involves:
    1. Swap root with last element: O(1)
    2. Heapify root of reduced heap: O(log n)
  
  Total: (n-1) × O(log n) = O(n log n)
  
  Unlike build-heap, we CANNOT tighten this bound:
  - Every extraction heapifies from the ROOT
  - Root is at height log n (maximum depth)
  - So each heapify genuinely costs O(log n)
  
  Extraction trace for n=8:
  ┌─────────────┬──────────┬────────────────────┐
  │ Extraction  │ HeapSize │ Heapify cost       │
  ├─────────────┼──────────┼────────────────────┤
  │ 1st         │ 7        │ ≤ log 7 ≈ 2.8     │
  │ 2nd         │ 6        │ ≤ log 6 ≈ 2.6     │
  │ 3rd         │ 5        │ ≤ log 5 ≈ 2.3     │
  │ 4th         │ 4        │ ≤ log 4 = 2       │
  │ 5th         │ 3        │ ≤ log 3 ≈ 1.6     │
  │ 6th         │ 2        │ ≤ log 2 = 1       │
  │ 7th         │ 1        │ 0                  │
  └─────────────┴──────────┴────────────────────┘
  
  Total = Σ(i=1 to n-1) ⌊log i⌋ = Θ(n log n)
```

---

## Overall Time Complexity

```
  Heap Sort = Build-Heap + Sort
            = O(n)       + O(n log n)
            = O(n log n)
  
  ┌────────────────────────────────────────────────┐
  │  Case     │  Build-Heap  │  Sort       │ Total │
  ├───────────┼──────────────┼─────────────┼───────┤
  │  Best     │  O(n)        │ O(n log n)  │ O(n log n) │
  │  Average  │  O(n)        │ O(n log n)  │ O(n log n) │
  │  Worst    │  O(n)        │ O(n log n)  │ O(n log n) │
  └───────────┴──────────────┴─────────────┴───────┘
  
  ALL CASES ARE O(n log n)! No degradation like QuickSort.
```

---

## Space Complexity

```
  ┌───────────────────────────────────────────────────┐
  │  Iterative heapify: O(1) auxiliary space          │
  │  Recursive heapify: O(log n) call stack           │
  │                                                    │
  │  The sorting is done IN-PLACE:                    │
  │  - No extra array needed                          │
  │  - Sorted elements stored at end of same array    │
  │                                                    │
  │  Space: O(1) with iterative heapify ✓             │
  └───────────────────────────────────────────────────┘
```

---

## Stability Analysis

```
  Heap Sort is NOT STABLE.
  
  Example: [3a, 3b, 1]
  
  Build max-heap: [3a, 3b, 1] (already a max-heap)
  
         3a
        /  \
      3b    1
  
  Extract 3a: swap 3a ↔ 1 → [1, 3b, | 3a]
  Heapify: [3b, 1, | 3a]
  
  Extract 3b: swap 3b ↔ 1 → [1, | 3b, 3a]
  
  Result: [1, 3b, 3a]
  
  Original order: 3a before 3b
  Sorted order:   3b before 3a  ← ORDER REVERSED!
  
  Heap Sort is UNSTABLE because:
  1. Swap with last element disrupts relative order
  2. Heap structure doesn't preserve insertion order
```

---

## Comparison with Other O(n log n) Sorts

```
  ┌──────────────┬──────────┬──────────┬──────────┬─────────┐
  │              │ Best     │ Average  │ Worst    │ Space   │
  ├──────────────┼──────────┼──────────┼──────────┼─────────┤
  │  Merge Sort  │ n log n  │ n log n  │ n log n  │ O(n)    │
  │  Quick Sort  │ n log n  │ n log n  │ n²       │ O(log n)│
  │  Heap Sort   │ n log n  │ n log n  │ n log n  │ O(1)    │
  └──────────────┴──────────┴──────────┴──────────┴─────────┘
  
  Heap Sort wins on:
  ✓ Worst-case guarantee (vs Quick Sort's O(n²))
  ✓ Space efficiency (vs Merge Sort's O(n))
  
  Heap Sort loses on:
  ✗ Cache performance (poor locality → jumps around array)
  ✗ Average-case constant factor (2× slower than Quick Sort)
  ✗ Not stable (vs Merge Sort)
  ✗ Not adaptive (doesn't benefit from sorted input)
```

---

## Why Heap Sort Has Poor Cache Performance

```
  ┌──────────────────────────────────────────────────────────┐
  │  Array index access pattern during heapify:             │
  │                                                          │
  │  Parent → Child: i → 2i+1 or 2i+2                      │
  │                                                          │
  │  For i = 0:  access 0, 1, 2                             │
  │  For i = 1:  access 1, 3, 4                             │
  │  For large i: jumps are HUGE                            │
  │    i = 100 → children at 201, 202                       │
  │    i = 1000 → children at 2001, 2002                    │
  │                                                          │
  │  These jumps cause CACHE MISSES:                        │
  │  - Parent and child are far apart in memory             │
  │  - CPU cache line doesn't help                          │
  │  - This is why Quick Sort (sequential access) wins      │
  │    in practice despite same O(n log n) complexity       │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Best Case | O(n log n) |
| Average Case | O(n log n) |
| Worst Case | O(n log n) |
| Build-Heap | O(n) |
| Space (iterative) | O(1) |
| Space (recursive) | O(log n) stack |
| Stable | No |
| Adaptive | No |
| Cache-friendly | No (poor locality) |

---

## Quick Revision Questions

1. **Why is build-heap O(n) and not O(n log n)?**
2. **Why can't the sort phase (Phase 2) be tightened below O(n log n)?**
3. **How does Heap Sort achieve O(1) space?**
4. **Give an example proving Heap Sort is unstable.**
5. **Why is Heap Sort slower than Quick Sort in practice despite same Big-O?**
6. **What is the total number of comparisons in worst-case Heap Sort?**

---

[← Previous: Implementation](03-implementation.md) | [Next: Applications →](05-heap-sort-applications.md)
