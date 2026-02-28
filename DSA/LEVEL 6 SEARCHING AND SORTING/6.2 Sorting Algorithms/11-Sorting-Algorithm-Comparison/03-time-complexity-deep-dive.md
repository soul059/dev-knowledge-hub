# Unit 11: Sorting Algorithm Comparison — Time Complexity Deep Dive

[← Previous: Stability Analysis](02-stability-analysis.md) | [Next: Space Complexity Analysis →](04-space-complexity-analysis.md)

---

## Overview

This chapter goes beyond the big-O summary to analyze the **exact number of operations**, the **constants hidden in big-O**, and the **crossover points** where one algorithm becomes better than another.

---

## Exact Operation Counts

### Quadratic Sorts

```
  BUBBLE SORT (optimized with early exit):
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:                                                 │
  │   Best:  n-1                    (sorted → one pass, no swap)  │
  │   Avg:   n(n-1)/4              ≈ n²/4                        │
  │   Worst: n(n-1)/2              ≈ n²/2                        │
  │                                                               │
  │  Swaps:                                                       │
  │   Best:  0                                                   │
  │   Avg:   n(n-1)/4              ≈ n²/4                        │
  │   Worst: n(n-1)/2              ≈ n²/2                        │
  └────────────────────────────────────────────────────────────────┘
  
  SELECTION SORT:
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:  n(n-1)/2  ALWAYS  ≈ n²/2                      │
  │  Swaps:        n-1       ALWAYS  (one per pass)              │
  └────────────────────────────────────────────────────────────────┘
  
  INSERTION SORT:
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:                                                 │
  │   Best:  n-1                    (sorted → 1 compare per elem)│
  │   Avg:   n(n-1)/4              ≈ n²/4                        │
  │   Worst: n(n-1)/2              ≈ n²/2                        │
  │                                                               │
  │  Shifts:                                                      │
  │   Best:  0                                                   │
  │   Avg:   n(n-1)/4              ≈ n²/4                        │
  │   Worst: n(n-1)/2              ≈ n²/2                        │
  │                                                               │
  │  Key insight: #shifts = #inversions in the array             │
  └────────────────────────────────────────────────────────────────┘
```

### Efficient Comparison Sorts

```
  MERGE SORT:
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:                                                 │
  │   Best:  n lg n / 2      ≈ 0.5 n lg n                       │
  │   Avg:   n lg n - n + 1  ≈ n lg n                           │
  │   Worst: n lg n - n + 1  ≈ n lg n                           │
  │                                                               │
  │  Copies (to auxiliary array):                                │
  │   Always: 2n per level × lg n levels = 2n lg n              │
  └────────────────────────────────────────────────────────────────┘
  
  QUICK SORT:
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:                                                 │
  │   Best:  n lg n          (balanced partitions)               │
  │   Avg:   2n ln n ≈ 1.39 n lg n                              │
  │   Worst: n(n-1)/2        (already sorted + first pivot)      │
  │                                                               │
  │  Swaps:                                                       │
  │   Best:  n lg n / 6                                          │
  │   Avg:   0.7 n ln n ≈ 0.3 n lg n                            │
  │   Worst: n(n-1)/2                                            │
  │                                                               │
  │  Avg comparisons 39% more than merge sort,                   │
  │  but fewer memory operations → faster in practice            │
  └────────────────────────────────────────────────────────────────┘
  
  HEAP SORT:
  ┌────────────────────────────────────────────────────────────────┐
  │  Comparisons:                                                 │
  │   Build heap:  < 2n                                          │
  │   Extract all: < 2n lg n                                     │
  │   Total:       < 2n lg n + 2n  ≈ 2n lg n                    │
  │                                                               │
  │  Roughly 2x more comparisons than merge sort.                │
  │  This matters on modern hardware (branch misprediction).     │
  └────────────────────────────────────────────────────────────────┘
```

---

## Crossover Points

```
  At what n does algorithm A become faster than algorithm B?
  
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  Insertion Sort vs Merge Sort:                              │
  │    Insertion: ~n²/4 ops    Merge: ~n lg n ops              │
  │    n²/4 = n lg n  →  n = 4 lg n  →  n ≈ 20-50            │
  │                                                              │
  │    For n < ~30: Insertion Sort is faster                    │
  │    For n > ~30: Merge Sort is faster                       │
  │    (This is why Timsort uses insertion sort for runs < 32) │
  │                                                              │
  │  Insertion Sort vs Quick Sort:                              │
  │    Similar crossover around n ≈ 10-20                      │
  │    (Quick Sort uses insertion sort for partitions < 16)    │
  │                                                              │
  │  Merge Sort vs Quick Sort:                                  │
  │    Quick Sort ~1.39 n lg n comparisons                     │
  │    Merge Sort ~n lg n comparisons                          │
  │    But Quick Sort does less memory work → faster for n > 0 │
  │    Quick Sort wins in practice at almost all sizes         │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

---

## The Theoretical Lower Bound

```
  ┌──────────────────────────────────────────────────────────────┐
  │      COMPARISON-BASED SORTING LOWER BOUND: Ω(n log n)      │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  Decision tree argument:                                    │
  │                                                              │
  │  • n! possible permutations of n elements                  │
  │  • Each comparison gives 1 bit of information              │
  │  • Need ≥ log₂(n!) bits to distinguish all permutations   │
  │  • log₂(n!) = n log₂ n - n log₂ e + O(log n)             │
  │             ≈ n log₂ n  (Stirling's approximation)        │
  │                                                              │
  │  Therefore: ANY comparison-based sort needs                │
  │  at least ⌈log₂(n!)⌉ comparisons in the worst case       │
  │                                                              │
  │  For n = 10: ⌈log₂(10!)⌉ = ⌈log₂(3628800)⌉ = 22        │
  │  Merge sort uses about 34 comparisons → ~1.5x optimal     │
  │                                                              │
  │  Non-comparison sorts (counting, radix, bucket) bypass     │
  │  this bound by using VALUE information, not just comparisons│
  └──────────────────────────────────────────────────────────────┘
```

---

## Amortized and Expected Analysis

### Quick Sort: Average Case Derivation

```
  Let T(n) = expected comparisons for n elements.
  
  Each element equally likely to be pivot:
  T(n) = n - 1 + (1/n) Σᵢ₌₀ⁿ⁻¹ [T(i) + T(n-1-i)]
  
  By symmetry:
  T(n) = n - 1 + (2/n) Σᵢ₌₀ⁿ⁻¹ T(i)
  
  Solving this recurrence:
  T(n) = 2(n+1)Hₙ - 4n
       ≈ 2n ln n
       ≈ 1.39 n log₂ n
  
  Where Hₙ = 1 + 1/2 + 1/3 + ... + 1/n ≈ ln n + 0.577
```

### Merge Sort: Exact Counting

```
  T(n) = 2T(n/2) + merge_cost(n)
  
  Merge of two n/2-length arrays:
    Best:  n/2  comparisons (one runs out immediately)
    Worst: n-1  comparisons (interleaved perfectly)
    Avg:   n - n/(n+1) ≈ n    comparisons
  
  Total (using Master Theorem):
  T(n) = n log₂ n - n + 1  (worst case)
  T(n) ≈ n log₂ n / 2      (best case, already sorted halves)
```

---

## Practical Timing Perspective

```
  Approximate operations per second (modern CPU, 2024):
  ~10⁹ simple operations/second
  
  ┌──────────┬─────────────┬─────────────┬─────────────┬──────────┐
  │    n     │ O(n²)       │ O(n lg n)   │ O(n)        │ O(n+k)   │
  │          │ time        │ time        │ time        │ k=1000   │
  ├──────────┼─────────────┼─────────────┼─────────────┼──────────┤
  │   100    │ 10 μs       │ 0.7 μs      │ 0.1 μs      │ 1.1 μs   │
  │  1,000   │ 1 ms        │ 10 μs       │ 1 μs        │ 2 μs     │
  │  10,000  │ 100 ms      │ 130 μs      │ 10 μs       │ 11 μs    │
  │ 100,000  │ 10 sec      │ 1.7 ms      │ 100 μs      │ 0.1 ms   │
  │   10⁶    │ 17 min      │ 20 ms       │ 1 ms        │ 1 ms     │
  │   10⁸    │ 116 days    │ 2.7 sec     │ 100 ms      │ 0.1 sec  │
  │   10⁹    │ 31.7 years  │ 30 sec      │ 1 sec       │ 1 sec    │
  └──────────┴─────────────┴─────────────┴─────────────┴──────────┘
  
  At n = 10⁶: O(n²) takes 17 MINUTES, O(n lg n) takes 20 MILLISECONDS
  That's a 50,000x difference!
```

---

## Recurrence Relations Summary

```
  ┌──────────────────┬──────────────────────────────────────────┐
  │  Algorithm       │  Recurrence                             │
  ├──────────────────┼──────────────────────────────────────────┤
  │  Merge Sort      │  T(n) = 2T(n/2) + O(n)                │
  │  Quick Sort best │  T(n) = 2T(n/2) + O(n)                │
  │  Quick Sort worst│  T(n) = T(n-1) + O(n)                 │
  │  Quick Sort avg  │  T(n) = (1/n)Σ[T(i)+T(n-1-i)] + O(n) │
  │  Heap Sort build │  T(n) = O(n) (bottom-up)              │
  │  Insertion Sort  │  T(n) = T(n-1) + O(n) worst           │
  │                  │  T(n) = T(n-1) + O(1) best            │
  └──────────────────┴──────────────────────────────────────────┘
  
  Master Theorem: T(n) = aT(n/b) + f(n)
  Case 1: f(n) = O(n^(log_b(a) - ε))  → T(n) = Θ(n^log_b(a))
  Case 2: f(n) = Θ(n^log_b(a))        → T(n) = Θ(n^log_b(a) lg n)
  Case 3: f(n) = Ω(n^(log_b(a) + ε))  → T(n) = Θ(f(n))
  
  Merge Sort: a=2, b=2, f(n)=Θ(n), log₂2=1 → Case 2 → Θ(n lg n)
```

---

## Summary Table

| Algorithm | Comparisons (Avg) | Swaps/Copies (Avg) | Hidden Constant |
|-----------|-------------------|---------------------|-----------------|
| Bubble | n²/4 | n²/4 | ~1.0 |
| Selection | n²/2 | n | ~0.5 (comps only) |
| Insertion | n²/4 | n²/4 | ~0.5 |
| Merge | n lg n | 2n lg n (copies) | ~1.0 |
| Quick | 1.39 n lg n | 0.3 n lg n | ~0.7 |
| Heap | 2n lg n | n lg n | ~2.0 |

---

## Quick Revision Questions

1. **How many comparisons does Selection Sort always make for n = 100?**
2. **Derive Quick Sort's average-case comparison count using the recurrence relation.**
3. **Why does Quick Sort beat Merge Sort in practice despite 39% more comparisons?**
4. **What is the decision-tree lower bound for sorting 8 elements?**
5. **At what input size n does Insertion Sort become slower than Merge Sort?**
6. **How long would Bubble Sort take on 10⁸ elements at 10⁹ ops/sec?**

---

[← Previous: Stability Analysis](02-stability-analysis.md) | [Next: Space Complexity Analysis →](04-space-complexity-analysis.md)
