# Unit 5: Merge Sort — Complexity Analysis

[← Previous: Merge Process Deep Dive](03-merge-process.md) | [Next: Space Optimization →](05-space-optimization.md)

---

## Overview

Merge Sort has a remarkable property: it runs in **O(n log n) in ALL cases** — best, average, and worst. This chapter rigorously derives this complexity using the recursion tree method and the Master Theorem.

---

## Recurrence Relation

```
  T(n) = 2·T(n/2) + O(n)
         ─────────   ────
            │          │
            │          └── Merge step: O(n) to merge two halves
            │
            └── Two recursive calls, each on half the array
  
  Base case: T(1) = O(1)
```

---

## Method 1: Recursion Tree

```
  Level 0:            c·n                     Total: c·n
                     /   \
  Level 1:       c·n/2   c·n/2               Total: c·n
                 / \      / \
  Level 2:   c·n/4 c·n/4 c·n/4 c·n/4        Total: c·n
               ...       ...
  Level k:   n/2^k problems of size c        Total: c·n
               ...       ...
  Level log₂n: c·1  c·1  ... c·1  (n leaves) Total: c·n
  
  ┌──────────────────────────────────────────────────────┐
  │  Number of levels = log₂(n)                         │
  │  Work at EACH level = c·n                            │
  │                                                      │
  │  Total = c·n × log₂(n) = O(n log n)                │
  └──────────────────────────────────────────────────────┘
  
  Why is work at each level always c·n?
  
  Level k has 2^k subproblems, each of size n/2^k.
  Merge work for each = c·(n/2^k)
  Total work at level k = 2^k × c·(n/2^k) = c·n  ✓
```

---

## Method 2: Master Theorem

```
  T(n) = a·T(n/b) + O(n^d)
  
  For Merge Sort: a = 2, b = 2, d = 1
  
  Compare log_b(a) with d:
    log₂(2) = 1 = d
  
  Case 2 of Master Theorem: log_b(a) = d
  
  T(n) = O(n^d · log n) = O(n¹ · log n) = O(n log n)  ✓
  
  ┌──────────────────────────────────────────────────────┐
  │  Quick Master Theorem Reference:                     │
  │                                                      │
  │  Case 1: log_b(a) > d  →  T(n) = O(n^(log_b(a)))  │
  │  Case 2: log_b(a) = d  →  T(n) = O(n^d · log n)   │
  │  Case 3: log_b(a) < d  →  T(n) = O(n^d)            │
  └──────────────────────────────────────────────────────┘
```

---

## Method 3: Unrolling the Recurrence

```
  T(n) = 2·T(n/2) + cn
       = 2·[2·T(n/4) + c·n/2] + cn
       = 4·T(n/4) + cn + cn
       = 4·T(n/4) + 2cn
       = 4·[2·T(n/8) + c·n/4] + 2cn
       = 8·T(n/8) + cn + 2cn
       = 8·T(n/8) + 3cn
       ...
       = 2^k · T(n/2^k) + k·cn
  
  Recursion ends when n/2^k = 1, i.e., k = log₂(n)
  
  T(n) = 2^(log₂n) · T(1) + log₂(n) · cn
       = n · c + cn · log₂(n)
       = O(n log n)  ✓
```

---

## Exact Comparison Count

```
  Let C(n) = number of comparisons for merge sort on n elements.
  
  C(n) = 2·C(n/2) + merge comparisons
  
  BEST CASE (each merge takes n/2 comparisons):
    C(n) = 2·C(n/2) + n/2
    C(n) = (n/2)·log₂(n) = O(n log n)
  
  WORST CASE (each merge takes n-1 comparisons):
    C(n) = 2·C(n/2) + (n-1)
    C(n) ≈ n·log₂(n) - n + 1 ≈ O(n log n)
  
  AVERAGE CASE:
    C(n) ≈ n·log₂(n) - 1.26n ≈ O(n log n)
  
  ┌──────────────────────────────────────────────────────┐
  │  For n = 1,000,000:                                  │
  │  Best:  ~10,000,000  comparisons                    │
  │  Worst: ~19,931,569  comparisons                    │
  │  Avg:   ~18,671,569  comparisons                    │
  │                                                      │
  │  Compare with O(n²) = 10¹² comparisons!            │
  └──────────────────────────────────────────────────────┘
```

---

## Space Complexity

```
  ┌──────────────────────────────────────────────────────────┐
  │  AUXILIARY SPACE: O(n)                                  │
  │                                                          │
  │  The merge step needs O(n) extra space for temp arrays  │
  │  Even optimized version needs one aux array of size n   │
  │                                                          │
  │  RECURSION STACK: O(log n)                              │
  │                                                          │
  │  Maximum recursion depth = log₂(n)                      │
  │  Each frame uses O(1) space (just indices)             │
  │                                                          │
  │  TOTAL SPACE: O(n) + O(log n) = O(n)                   │
  └──────────────────────────────────────────────────────────┘
  
  Recursion stack at maximum depth:
  
  mergeSort(0, 7)         ← Frame 1
    mergeSort(0, 3)       ← Frame 2
      mergeSort(0, 1)     ← Frame 3
        mergeSort(0, 0)   ← Frame 4 (base case, returns)
  
  Maximum 4 frames for n=8 → log₂(8) + 1 = 4 frames
```

---

## Why Merge Sort Is NOT Adaptive

```
  Sorted input:           [1, 2, 3, 4, 5, 6, 7, 8]
  Random input:           [5, 2, 8, 1, 4, 7, 3, 6]
  Reverse sorted input:   [8, 7, 6, 5, 4, 3, 2, 1]
  
  ALL require:
  - Same number of recursive calls
  - Same number of levels (log₂n)
  - Same amount of merging work (each merge still processes all elements)
  
  ┌─────────────────────────────────────────────────────────┐
  │  Merge Sort does NOT adapt to input patterns.          │
  │  It always takes O(n log n) regardless.                │
  │                                                         │
  │  Contrast with Insertion Sort:                          │
  │  - Sorted: O(n)                                        │
  │  - Random: O(n²)                                       │
  │  - Reversed: O(n²)                                     │
  │                                                         │
  │  TimSort fixes this by detecting existing sorted runs  │
  └─────────────────────────────────────────────────────────┘
```

---

## Growth Rate Comparison

```
  n           │ n²          │ n log₂ n     │ Speedup
  ────────────┼─────────────┼──────────────┼──────────
  10          │ 100         │ 33           │ 3x
  100         │ 10,000      │ 664          │ 15x
  1,000       │ 1,000,000   │ 9,966        │ 100x
  10,000      │ 100,000,000 │ 132,877      │ 753x
  100,000     │ 10¹⁰        │ 1,660,964    │ 6,023x
  1,000,000   │ 10¹²        │ 19,931,569   │ 50,172x
  
  n log n vs n²:
  
  Operations
  ^
  │                                          n²
  │                                     ····
  │                                ····
  │                           ····
  │                      ····
  │                 ····
  │            ····
  │       ····
  │   ···
  │ ··____________________________________________ n log n
  │
  └──────────────────────────────────────────────> n
```

---

## Merge Sort vs O(n²) Sorts — Complexity Summary

```
  ┌──────────────────┬─────────┬─────────┬─────────┬───────┐
  │ Algorithm        │  Best   │ Average │  Worst  │ Space │
  ├──────────────────┼─────────┼─────────┼─────────┼───────┤
  │ Bubble Sort      │  O(n)   │  O(n²)  │  O(n²)  │ O(1)  │
  │ Selection Sort   │  O(n²)  │  O(n²)  │  O(n²)  │ O(1)  │
  │ Insertion Sort   │  O(n)   │  O(n²)  │  O(n²)  │ O(1)  │
  │ MERGE SORT       │O(n logn)│O(n logn)│O(n logn)│ O(n)  │
  └──────────────────┴─────────┴─────────┴─────────┴───────┘
  
  Key trade-off: Merge Sort trades O(n) space for O(n log n) time
```

---

## Summary Table

| Analysis Method | Result |
|----------------|--------|
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Recursion Tree | log n levels × O(n) per level = O(n log n) |
| Master Theorem | a=2, b=2, d=1, Case 2 → O(n log n) |
| Best case comparisons | ~(n/2)·log n |
| Worst case comparisons | ~n·log n - n + 1 |
| Auxiliary space | O(n) |
| Stack space | O(log n) |
| Adaptive? | No |

---

## Quick Revision Questions

1. **Write the recurrence relation for Merge Sort.**
2. **Use the Master Theorem to solve T(n) = 2T(n/2) + O(n).**
3. **Why does each level of the recursion tree do O(n) total work?**
4. **What is the space complexity? Break it into auxiliary + stack.**
5. **Why is Merge Sort NOT adaptive to sorted input?**
6. **For n = 1,000,000, approximately how many comparisons does Merge Sort make?**

---

[← Previous: Merge Process Deep Dive](03-merge-process.md) | [Next: Space Optimization →](05-space-optimization.md)
