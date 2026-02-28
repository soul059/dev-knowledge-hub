# Unit 6: Quick Sort — Complexity Analysis

[← Previous: Pivot Selection](04-pivot-selection.md) | [Next: Quick Sort vs Merge Sort →](06-quicksort-vs-mergesort.md)

---

## Overview

Quick Sort's complexity varies dramatically with pivot quality. This chapter derives the best, worst, and average cases rigorously, and explains why the average case makes Quick Sort the fastest practical sorting algorithm.

---

## Best Case: O(n log n)

```
  Occurs when every partition splits the array EXACTLY in half.
  
  T(n) = 2·T(n/2) + O(n)
  
  Same recurrence as Merge Sort!
  
  Solution (Master Theorem): a=2, b=2, d=1 → Case 2 → O(n log n)
  
  Recursion tree:
  Level 0:           n                 work = n
  Level 1:       n/2   n/2            work = n
  Level 2:    n/4 n/4 n/4 n/4         work = n
  ...
  Level log n: 1 1 1 ... 1 (n nodes)  work = n
  
  Total: n × log₂(n) = O(n log n)
```

---

## Worst Case: O(n²)

```
  Occurs when every partition puts ALL elements on one side.
  
  Example: sorted array [1, 2, 3, 4, 5] with last-element pivot
  
  Partition 1: pivot=5, split: [1,2,3,4] [5] []     work = 5
  Partition 2: pivot=4, split: [1,2,3] [4] []       work = 4
  Partition 3: pivot=3, split: [1,2] [3] []          work = 3
  Partition 4: pivot=2, split: [1] [2] []            work = 2
  
  T(n) = T(n-1) + O(n)
  
  Unrolling:
  T(n) = T(n-1) + cn
       = T(n-2) + c(n-1) + cn
       = T(n-3) + c(n-2) + c(n-1) + cn
       ...
       = T(1) + c·(2 + 3 + ... + n)
       = O(1) + c·n(n+1)/2 - c
       = O(n²)
  
  Recursion tree (degenerate):
  
  n ────────────────── work = n
  └── n-1 ──────────── work = n-1
      └── n-2 ──────── work = n-2
          └── n-3 ──── work = n-3
              └── ...
                  └── 1  work = 1
  
  Height = n (not log n!)
  Total work = n + (n-1) + ... + 1 = n(n+1)/2 = O(n²)
```

---

## Average Case: O(n log n)

```
  The average case analysis assumes each element is equally likely
  to be the pivot (random pivot selection).
  
  If pivot lands at position k (0-indexed), the split is k : (n-1-k).
  
  T(n) = (1/n) × Σ[k=0 to n-1] [T(k) + T(n-1-k)] + cn
  
  By symmetry: T(k) and T(n-1-k) terms pair up, so:
  
  T(n) = (2/n) × Σ[k=0 to n-1] T(k) + cn
  
  ┌──────────────────────────────────────────────────────┐
  │  Solving this recurrence (advanced):                 │
  │                                                      │
  │  T(n) = 2(n+1) × H(n) - 4n                         │
  │  where H(n) = 1 + 1/2 + 1/3 + ... + 1/n ≈ ln(n)   │
  │                                                      │
  │  T(n) ≈ 2n·ln(n) = 2n·(ln2)·log₂(n)               │
  │       ≈ 1.39·n·log₂(n)                              │
  │                                                      │
  │  Only 39% more comparisons than the best case!      │
  └──────────────────────────────────────────────────────┘
```

---

## Comparison Count Summary

```
  ┌──────────────────────────────────────────────────────┐
  │  Case      │  Comparisons              │  Time      │
  ├────────────┼───────────────────────────┼────────────┤
  │  Best      │  ~0.5·n·log₂(n)          │  O(n log n)│
  │  Average   │  ~1.39·n·log₂(n)         │  O(n log n)│
  │  Worst     │  n(n-1)/2 ≈ 0.5·n²       │  O(n²)    │
  └────────────┴───────────────────────────┴────────────┘
  
  For n = 1,000,000:
  Best:       ~10,000,000  comparisons
  Average:    ~27,800,000  comparisons
  Worst:    ~500,000,000,000  comparisons  ← 50,000x slower!
```

---

## Space Complexity

```
  Quick Sort is IN-PLACE (no auxiliary array), but uses STACK space.
  
  BEST/AVERAGE CASE: O(log n) stack depth
  ┌──────────────────────────────┐
  │  Balanced partitions →      │
  │  recursion tree height = log n│
  │  Each frame uses O(1) space │
  │  Total: O(log n)            │
  └──────────────────────────────┘
  
  WORST CASE: O(n) stack depth
  ┌──────────────────────────────┐
  │  Degenerate partitions →    │
  │  recursion depth = n        │
  │  Total: O(n)                │
  │  Can cause STACK OVERFLOW!  │
  └──────────────────────────────┘
  
  TAIL-CALL OPTIMIZED: O(log n) always
  ┌──────────────────────────────┐
  │  Always recurse on smaller  │
  │  half, loop on larger half  │
  │  Maximum depth: O(log n)    │
  └──────────────────────────────┘
```

---

## Why Average Case Matters More Than Worst Case

```
  The probability of worst case with random pivot:
  
  For n elements, worst case requires picking the min or max
  every single time:
  
  P(worst) = (2/n) × (2/(n-1)) × (2/(n-2)) × ...
           = 2^n / n!
  
  For n = 20: P ≈ 2^20 / 20! ≈ 4.3 × 10^-13
  For n = 100: essentially 0
  
  ┌──────────────────────────────────────────────────────────┐
  │  With randomized pivot selection:                       │
  │                                                          │
  │  - Worst case is theoretically possible but              │
  │    astronomically improbable                             │
  │  - No adversary can construct worst-case input           │
  │    (since pivot is random)                               │
  │  - Expected performance is O(n log n) for ALL inputs    │
  └──────────────────────────────────────────────────────────┘
```

---

## Quick Sort vs Other Algorithms — Full Complexity Table

```
  ┌──────────────────┬──────────┬──────────┬──────────┬────────┐
  │ Algorithm        │ Best     │ Average  │ Worst    │ Space  │
  ├──────────────────┼──────────┼──────────┼──────────┼────────┤
  │ Bubble Sort      │ O(n)     │ O(n²)    │ O(n²)    │ O(1)   │
  │ Selection Sort   │ O(n²)    │ O(n²)    │ O(n²)    │ O(1)   │
  │ Insertion Sort   │ O(n)     │ O(n²)    │ O(n²)    │ O(1)   │
  │ Merge Sort       │ O(nlogn) │ O(nlogn) │ O(nlogn) │ O(n)   │
  │ QUICK SORT       │ O(nlogn) │ O(nlogn) │ O(n²)    │ O(logn)│
  │ Heap Sort        │ O(nlogn) │ O(nlogn) │ O(nlogn) │ O(1)   │
  └──────────────────┴──────────┴──────────┴──────────┴────────┘
  
  Quick Sort: Best space of O(n log n) sorts (in-place!),
  but worst-case time is O(n²).
  
  The trade-off: Quick Sort = best average performance + smallest space
  but no worst-case guarantee (without Introsort modification).
```

---

## Introsort: Best of All Worlds

```
  ┌──────────────────────────────────────────────────────────┐
  │  INTROSORT = Quick Sort + Heap Sort + Insertion Sort    │
  │                                                          │
  │  Algorithm:                                              │
  │  1. Start with Quick Sort (fast average case)           │
  │  2. Track recursion depth                               │
  │  3. If depth exceeds 2·log₂(n):                        │
  │     → Switch to Heap Sort (O(n log n) guaranteed)       │
  │  4. If subarray < 16 elements:                          │
  │     → Switch to Insertion Sort (fast for small n)       │
  │                                                          │
  │  Result:                                                 │
  │  Time:  O(n log n) WORST CASE guaranteed               │
  │  Space: O(log n) in practice                            │
  │  Speed: Same as Quick Sort on average                   │
  │                                                          │
  │  Used by: C++ std::sort, .NET, Rust std sort           │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Case | When | Recurrence | Time | Space |
|------|------|-----------|------|-------|
| Best | Perfect splits | T(n)=2T(n/2)+cn | O(n log n) | O(log n) |
| Average | Random pivot | T(n)=(2/n)ΣT(k)+cn | O(1.39n log n) | O(log n) |
| Worst | Min/max pivot always | T(n)=T(n-1)+cn | O(n²) | O(n) |
| Tail-optimized worst | Degenerate but optimized | Same time | O(n²) | O(log n) |
| Introsort worst | Falls back to heap sort | — | O(n log n) | O(log n) |

---

## Quick Revision Questions

1. **Derive the worst-case recurrence and solve it.**
2. **What is the average-case comparison count? (~1.39n log n)**
3. **What input triggers worst case for last-element pivot?**
4. **Why is stack space O(n) in the worst case?**
5. **How does Introsort guarantee O(n log n) worst case?**
6. **Calculate: probability of worst case with random pivot for n=10.**

---

[← Previous: Pivot Selection](04-pivot-selection.md) | [Next: Quick Sort vs Merge Sort →](06-quicksort-vs-mergesort.md)
