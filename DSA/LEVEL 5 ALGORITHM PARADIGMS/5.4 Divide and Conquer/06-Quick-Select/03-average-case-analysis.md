# Chapter 3: Average Case Analysis of Quick Select

## Overview

Quick Select achieves **O(n) expected time** for finding the k-th smallest element. This chapter provides a rigorous analysis of why the average case is linear, using the geometric series argument and the formal indicator variable approach.

---

## Intuitive Argument: Geometric Series

```
With a random pivot, on average we reduce the problem to ~n/2 elements.

Work at each level:
  Level 0: scan n elements to partition       → cn
  Level 1: scan ~n/2 elements to partition    → cn/2
  Level 2: scan ~n/4 elements to partition    → cn/4
  Level 3: scan ~n/8 elements to partition    → cn/8
  ...
  
Total = cn + cn/2 + cn/4 + cn/8 + ...
      = cn × (1 + 1/2 + 1/4 + 1/8 + ...)
      = cn × 2
      = 2cn
      = O(n)

┌─────────────────────────────────────────────────────────┐
│  KEY INSIGHT: The geometric series 1 + 1/2 + 1/4 + ... │
│  converges to 2, so total work = 2n, still O(n)!       │
│                                                         │
│  Compare with Quick Sort:                                │
│    Quick Sort: cn per level × log n levels = O(n log n) │
│    Quick Select: cn, cn/2, cn/4, ... = O(n)            │
│                                                         │
│  The difference: Quick Select recurses on ONE side,     │
│  so the work DECREASES geometrically!                   │
└─────────────────────────────────────────────────────────┘
```

---

## Visual: Why Sum Converges

```
Total work visualization (each bar = work at that level):

Level 0: ████████████████████████████████  (n)
Level 1: ████████████████                  (n/2)
Level 2: ████████                          (n/4)
Level 3: ████                              (n/8)
Level 4: ██                                (n/16)
Level 5: █                                 (n/32)
         ─────────────────────────────────
Total:   ≤ 2n

All the remaining levels combined never exceed the first level!
```

---

## Formal Analysis: Recurrence

```
T(n) = T(n/2) + O(n)     (ideal: pivot at median)

By Master Theorem:
  a = 1, b = 2, f(n) = cn
  log_b(a) = log_2(1) = 0
  f(n) = cn = Ω(n^(0+ε)) for ε = 1

  Case 3: T(n) = Θ(f(n)) = Θ(n)

But the pivot isn't always at the median. So let's be more precise.
```

---

## Expected Time with Random Pivot

```
After partitioning n elements, the pivot lands at position i
(0-indexed) with probability 1/n for each i.

If we want the k-th element:
  - If i == k: done, cost = cn
  - If i > k:  recurse on arr[0..i-1], size = i
  - If i < k:  recurse on arr[i+1..n-1], size = n-1-i

Expected cost:
  E[T(n)] = cn + (1/n) × Σ_{i=0}^{n-1} T(max(i, n-1-i))

Since the recursive call is on the side containing k, and 
in the worst case we recurse on the larger side:

  E[T(n)] ≤ cn + (1/n) × Σ_{i=⌊n/2⌋}^{n-1} T(i) × 2

The factor of 2: pivot could be on either side of k.
```

---

## Proving E[T(n)] ≤ 4cn

```
Claim: E[T(n)] ≤ 4cn for all n ≥ 1.

Proof by induction (substitution method):

E[T(n)] ≤ cn + (2/n) × Σ_{i=⌊n/2⌋}^{n-1} E[T(i)]

Assume E[T(i)] ≤ 4ci for all i < n.

E[T(n)] ≤ cn + (2/n) × Σ_{i=⌊n/2⌋}^{n-1} 4ci
        = cn + (8c/n) × Σ_{i=⌊n/2⌋}^{n-1} i
        = cn + (8c/n) × [Σ_{i=1}^{n-1} i - Σ_{i=1}^{⌊n/2⌋-1} i]
        = cn + (8c/n) × [n(n-1)/2 - (⌊n/2⌋-1)⌊n/2⌋/2]

Simplifying (using n/2 as approximation):
        ≤ cn + (8c/n) × [n²/2 - n²/8] / 2
        ... (after algebra) ...
        ≤ cn + 3cn
        = 4cn

Therefore E[T(n)] ≤ 4cn = O(n). ✓

Expected number of comparisons ≈ 3.39n for finding the median.
```

---

## Why Quick Select Is O(n) But Quick Sort Is O(n log n)

```
QUICK SORT:                        QUICK SELECT:
  Two recursive calls                One recursive call
  
  T(n) = T(left) + T(right) + cn   T(n) = T(one side) + cn
  
  Level 0:  cn                      Level 0:  cn
  Level 1:  cn  (both halves)       Level 1:  cn/2  (one half)
  Level 2:  cn  (all quarters)      Level 2:  cn/4  (one quarter)
  Level 3:  cn  (all eighths)       Level 3:  cn/8  (one eighth)
  ...                               ...
  
  Total: cn × log n                Total: cn × (1+1/2+1/4+...)
  = O(n log n)                     = cn × 2
                                    = O(n)

The crucial difference:
  Quick Sort processes ALL elements at every level
  Quick Select processes a SHRINKING fraction at each level
```

---

## What About the Worst Case?

```
Worst case: every pivot is the extreme element.

T(n) = T(n-1) + cn
     = cn + c(n-1) + c(n-2) + ... + c
     = c × n(n+1)/2
     = O(n²)

This happens when:
  - Array is sorted and we always pick first/last
  - Adversarial input chosen after seeing the algorithm
  
Probability with random pivot: exponentially small
  P(worst case) ≈ 2ⁿ/n! → negligibly small
```

---

## Summary of Time Bounds

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Best case:     O(n)     - pivot = k-th element         │
│  Average case:  O(n)     - ~4n comparisons              │
│  Worst case:    O(n²)    - always extreme pivot         │
│                                                         │
│  Expected comparisons to find median: ≈ 3.39n           │
│                                                         │
│  With median-of-medians: guaranteed O(n) worst case     │
│  (covered in next chapter)                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Expected time | O(n), specifically ≤ 4cn |
| Why O(n) not O(n log n) | Recurse on ONE side only |
| Geometric series | 1 + 1/2 + 1/4 + ... = 2 |
| Expected comparisons (median) | ~3.39n |
| Worst case | O(n²) with bad pivots |
| Probability of worst | ~2ⁿ/n! (negligible) |

---

## Quick Revision Questions

1. **Why does Quick Select's total work form a geometric series?**
2. **What is the sum of 1 + 1/2 + 1/4 + 1/8 + ... and why does this matter?**
3. **What constant factor does the rigorous proof give for E[T(n)]?**
4. **Why is Quick Sort O(n log n) while Quick Select is O(n)?**
5. **What is the probability of worst-case behavior with a random pivot?**

---

[← Previous: Quick Select Algorithm](02-quickselect-algorithm.md) | [Next: Worst Case & Median of Medians →](04-median-of-medians.md)
