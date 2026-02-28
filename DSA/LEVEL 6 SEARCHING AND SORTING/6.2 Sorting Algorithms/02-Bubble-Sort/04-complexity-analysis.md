# Chapter 4: Bubble Sort — Complexity Analysis

[← Previous: Optimization](03-optimization.md) | [Next: When to Use →](05-when-to-use.md)

---

## Overview

Understanding the time and space complexity of Bubble Sort in all cases: best, average, and worst. We analyze both the basic and optimized variants.

---

## Time Complexity

### Worst Case: O(n²) — Reverse Sorted Input

```
  Input: [5, 4, 3, 2, 1]    ← Every pair needs swapping!
  
  Pass 0: 4 comparisons, 4 swaps → [4,3,2,1,5]
  Pass 1: 3 comparisons, 3 swaps → [3,2,1,4,5]
  Pass 2: 2 comparisons, 2 swaps → [2,1,3,4,5]
  Pass 3: 1 comparison,  1 swap  → [1,2,3,4,5]
  
  Comparisons = (n-1) + (n-2) + ... + 1
              = n(n-1)/2
              = 5(4)/2 = 10
  
  Swaps = n(n-1)/2 = 10  (same as comparisons in worst case)
  
  ┌──────────────────────────────────────────────┐
  │  T(n) = n(n-1)/2 = (n² - n)/2 = O(n²)      │
  └──────────────────────────────────────────────┘
```

### Counting Comparisons

```
  Pass i  │  Comparisons  │  Index range (j)
  ────────┼───────────────┼──────────────────
    0     │     n - 1     │   0 to n-2
    1     │     n - 2     │   0 to n-3
    2     │     n - 3     │   0 to n-4
   ...    │     ...       │
   n-2    │       1       │   0 to 0
  ────────┼───────────────┤
  Total   │  Σ(n-1 to 1)  │
          │  = n(n-1)/2   │
  
  For n = 10:  10 × 9 / 2 = 45 comparisons
  For n = 100: 100 × 99 / 2 = 4,950 comparisons
  For n = 1000: 1000 × 999 / 2 = 499,500 comparisons
```

### Best Case

```
  ═══ BASIC Bubble Sort (no optimization) ═══
  
  Input: [1, 2, 3, 4, 5]    ← Already sorted
  
  Still does ALL passes:
  Pass 0: 4 comparisons, 0 swaps
  Pass 1: 3 comparisons, 0 swaps
  Pass 2: 2 comparisons, 0 swaps
  Pass 3: 1 comparison,  0 swaps
  Total: 10 comparisons → O(n²) even for sorted input!
  
  ═══ OPTIMIZED Bubble Sort (with swapped flag) ═══
  
  Input: [1, 2, 3, 4, 5]    ← Already sorted
  
  Pass 0: 4 comparisons, 0 swaps → swapped = false → BREAK!
  Total: 4 comparisons = n-1     → O(n) best case!
```

### Average Case: O(n²)

```
  On average, about half the pairs need swapping:
  
  Expected comparisons ≈ n(n-1)/2        (all comparisons still needed)
  Expected swaps       ≈ n(n-1)/4        (about half of comparisons)
  
  Average case = O(n²)
  
  ┌──────────────────────────────────────────────┐  
  │  Even with optimization, average case is     │
  │  still O(n²) — early termination rarely      │
  │  helps for random input.                     │
  └──────────────────────────────────────────────┘
```

---

## Space Complexity: O(1)

```
  Memory usage:
  
  ┌──────────────────────────────────────┐
  │  Input array: n elements             │  ← given, not counted
  ├──────────────────────────────────────┤
  │  Variables:                          │
  │    i      → 1 integer               │
  │    j      → 1 integer               │
  │    temp   → 1 integer (for swap)    │
  │    swapped → 1 boolean (optimized)  │
  ├──────────────────────────────────────┤
  │  Total extra space: O(1)            │
  │  = CONSTANT, regardless of n        │
  └──────────────────────────────────────┘
  
  Bubble Sort is an IN-PLACE algorithm.
```

---

## Complexity Summary

```
  ┌─────────────────────────────────────────────────────────────┐
  │              BUBBLE SORT COMPLEXITY                          │
  ├──────────────┬──────────────┬──────────────┬────────────────┤
  │   Case       │ Comparisons  │    Swaps     │  Time          │
  ├──────────────┼──────────────┼──────────────┼────────────────┤
  │  Best        │    n - 1     │      0       │  O(n)*         │
  │  Average     │   n(n-1)/2   │   n(n-1)/4   │  O(n²)        │
  │  Worst       │   n(n-1)/2   │   n(n-1)/2   │  O(n²)        │
  ├──────────────┼──────────────┴──────────────┼────────────────┤
  │  Space       │        O(1) auxiliary        │  In-place      │
  └──────────────┴──────────────────────────────┴────────────────┘
  
  * O(n) best case only with the swapped flag optimization
```

---

## Growth Rate Visualization

```
  Operations
  │
  │                                          ╱ O(n²) 
  │                                        ╱   worst/avg
  │                                      ╱
  │                                   ╱
  │                                ╱
  │                            ╱
  │                        ╱
  │                    ╱          ╱ O(n log n) — what we'd prefer
  │               ╱          ╱
  │          ╱          ╱
  │     ╱          ╱
  │ ╱     ╱  ╱────────────── O(n) best case (optimized)
  │__╱_______________________________________
  0              n →
  
  For n = 1000:
  O(n)      =     1,000 operations
  O(n log n)=    10,000 operations
  O(n²)     = 1,000,000 operations  ← 1000× more than O(n)!
```

---

## Why O(n²) Is Bad for Large Inputs

```
  n          │  O(n²) operations  │  Time @ 10⁹ ops/sec
  ───────────┼────────────────────┼──────────────────────
  100        │       10,000       │  0.00001 sec
  1,000      │    1,000,000       │  0.001 sec
  10,000     │  100,000,000       │  0.1 sec
  100,000    │ 10,000,000,000     │  10 sec         ← slow!
  1,000,000  │ 10¹²              │  16.7 minutes    ← impractical!
  
  Compare with O(n log n):
  1,000,000  │ ~20,000,000        │  0.02 sec       ← fast!
```

---

## Number of Inversions

The number of swaps in Bubble Sort equals the number of **inversions** in the array:

```
  An INVERSION is a pair (i, j) where i < j but A[i] > A[j].
  
  Array: [3, 1, 4, 2]
  
  Inversions:
    (3,1) → i=0, j=1: A[0]=3 > A[1]=1 ✓ inversion
    (3,2) → i=0, j=3: A[0]=3 > A[3]=2 ✓ inversion
    (4,2) → i=2, j=3: A[2]=4 > A[3]=2 ✓ inversion
  
  Total inversions = 3
  Bubble Sort will do exactly 3 swaps.
  
  ┌──────────────────────────────────────────────────────┐
  │  # swaps in Bubble Sort = # inversions in array      │
  │                                                      │
  │  Sorted array: 0 inversions                          │
  │  Reverse sorted: n(n-1)/2 inversions (maximum)       │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Best case time | O(n) with optimization |
| Average case time | O(n²) |
| Worst case time | O(n²) |
| Space | O(1) |
| Best case input | Already sorted array |
| Worst case input | Reverse sorted array |
| # comparisons (worst) | n(n-1)/2 |
| # swaps (worst) | n(n-1)/2 |
| # swaps = | # inversions |

---

## Quick Revision Questions

1. **Derive the formula for total comparisons in Bubble Sort.**
2. **What input produces the worst case? Why?**
3. **How does the optimization change the best-case complexity?**
4. **What is the relationship between swaps and inversions?**
5. **For n = 1000, how many comparisons does Bubble Sort make in the worst case?**
6. **Why is the average case still O(n²) even with the optimization?**

---

[← Previous: Optimization](03-optimization.md) | [Next: When to Use →](05-when-to-use.md)
