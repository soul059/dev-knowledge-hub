# Chapter 4: Worst Case Analysis

## Overview

Quick Sort's Achilles' heel is its **O(n²) worst case**. Understanding when and why this happens — and how to prevent it — is crucial for both interviews and practical programming.

---

## When Does O(n²) Occur?

The worst case happens when every partition produces **maximally unbalanced** splits.

```
WORST CASE: Pivot is always the smallest or largest element

T(n) = T(0) + T(n-1) + O(n)
     = T(n-1) + O(n)

Unrolling:
  T(n) = T(n-1) + cn
       = T(n-2) + c(n-1) + cn
       = T(n-3) + c(n-2) + c(n-1) + cn
       = ...
       = c(1 + 2 + 3 + ... + n)
       = cn(n+1)/2
       = O(n²)
```

---

## Visual: Worst Case Recursion Tree

```
quickSort([1,2,3,4,5]) with first/last element as pivot

pivot=1: [] [1] [2,3,4,5]          work: 5
pivot=2:       [] [2] [3,4,5]      work: 4
pivot=3:             [] [3] [4,5]  work: 3
pivot=4:                   [] [4] [5]  work: 2
                                       
Total: 5+4+3+2+1 = 15 = O(n²)

VERSUS: BEST CASE  (balanced splits)

pivot=3: [1,2] [3] [4,5]           work: 5
         pivot=1: [] [1] [2]       work: 2
         pivot=4: [] [4] [5]       work: 2
                                   
Total: 5+2+2 = 9 = O(n log n)
```

---

## Worst Case Input Patterns

```
With Lomuto (pivot = last element):
  ✗ Already sorted:    [1, 2, 3, 4, 5] → O(n²)
  ✗ Reverse sorted:    [5, 4, 3, 2, 1] → O(n²)
  ✗ All equal:         [3, 3, 3, 3, 3] → O(n²)
  
With first element as pivot:
  ✗ Already sorted:    [1, 2, 3, 4, 5] → O(n²)
  ✗ Reverse sorted:    [5, 4, 3, 2, 1] → O(n²)

With median-of-three:
  ✗ Can be defeated with specially crafted "median-of-three killer" sequences
  ✓ BUT this is extremely rare in practice
```

---

## Stack Depth in Worst Case

```
Worst case: O(n) recursive calls deep

quickSort(0, n-1)
  quickSort(0, -1)
  quickSort(1, n-1)
    quickSort(1, 0)
    quickSort(2, n-1)
      quickSort(2, 1)
      quickSort(3, n-1)
        ...          ← n levels deep!

This can cause STACK OVERFLOW for large n!

Solution: Tail Call Optimization
```

---

## Tail Call Optimization (TCO)

Always recurse on the **smaller** partition first, then loop on the larger one:

```
function quickSortTCO(arr, lo, hi):
    while lo < hi:
        p = partition(arr, lo, hi)
        
        // Recurse on smaller side, loop on larger
        if p - lo < hi - p:
            quickSort(arr, lo, p - 1)    // recurse on smaller left
            lo = p + 1                    // loop on larger right
        else:
            quickSort(arr, p + 1, hi)    // recurse on smaller right
            hi = p - 1                   // loop on larger left

// Guarantees O(log n) stack depth even in worst case!
// Time is still O(n²) in worst case, but NO stack overflow.
```

```
Why O(log n) stack depth?

If we always recurse on the half that's ≤ n/2 elements:
  Level 0: recurse on ≤ n/2
  Level 1: recurse on ≤ n/4
  ...
  At most log₂(n) levels

Even if partitions are unbalanced, the RECURSIVE side
is always the smaller half.
```

---

## Average Case Analysis

```
Average case: random permutation input, each element
equally likely to be the pivot.

T(n) = (1/n) Σ [T(k) + T(n-1-k)] + cn    for k = 0 to n-1

Solving this recurrence:
  T(n) = 2n ln(n) + O(n)
       ≈ 1.39 × n log₂(n)

This is only 39% more comparisons than the theoretical minimum!

┌──────────────────────────────────────────────────────────┐
│  AVERAGE CASE INTUITION                                   │
│                                                           │
│  Even "mediocre" pivots (25th-75th percentile) give       │
│  good enough splits.                                      │
│                                                           │
│  Probability of "good" pivot: 50%                         │
│  Expected "good" pivots between bad ones: 2               │
│                                                           │
│  Worst split ratio: 1:3 (25th percentile)                 │
│  T(n) = T(n/4) + T(3n/4) + cn                           │
│  Still O(n log n)! (log base 4/3)                        │
│                                                           │
│  Only consistently extreme pivots (< 1% percentile)      │
│  cause O(n²), and this is exponentially unlikely.         │
└──────────────────────────────────────────────────────────┘
```

---

## Probability of Worst Case

```
For randomized Quick Sort on n elements:

P(worst case) ≈ P(always picking worst pivot)
              = (2/n) × (2/(n-1)) × (2/(n-2)) × ...
              ≈ 2ⁿ / n!
              → astronomically small for large n

For n = 100:
  P(worst case) ≈ 10⁻¹⁵⁸
  
You're more likely to win the lottery 20 times in a row.
```

---

## Preventing Worst Case

```
┌────────────────────────────────────────────────────────────┐
│  METHOD               │  GUARANTEES      │  OVERHEAD       │
├────────────────────────┼──────────────────┼─────────────────┤
│  Random pivot          │  O(n lg n) exp   │  rand() calls  │
│  Median of 3           │  O(n lg n) pract │  3 comparisons │
│  Median of medians     │  O(n lg n) worst │  O(n) per level│
│  IntroSort             │  O(n lg n) worst │  Depth counter │
│  3-way partition       │  O(n) for dups   │  Extra logic   │
│  Shuffle before sort   │  O(n lg n) exp   │  O(n) shuffle  │
│  Tail call optimization│  O(lg n) stack   │  Loop overhead │
└────────────────────────┴──────────────────┴─────────────────┘
```

---

## Worst vs Best vs Average

```
             Time              Space (stack)
Best:    O(n log n)            O(log n)
Average: O(n log n) ≈ 1.39n lg n   O(log n)
Worst:   O(n²)                O(n) or O(log n) with TCO

Comparison counts:
Best:    ~n log₂(n)           (balanced splits)
Average: ~1.39 n log₂(n)     (random input)
Worst:   ~n²/2                (sorted input, bad pivot)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Worst case time | O(n²) — unbalanced partitions every time |
| Worst case inputs | Sorted, reverse-sorted, all-equal |
| Average case | O(n log n) ≈ 1.39n log₂n |
| Stack depth worst | O(n) without TCO |
| Stack depth with TCO | O(log n) guaranteed |
| Prevention | Random pivot, median-of-3, IntroSort |
| Probability of worst | ~2ⁿ/n! (negligible with random pivot) |

---

## Quick Revision Questions

1. **What input causes worst case for Lomuto partition with last element as pivot?**
2. **Derive the worst case recurrence T(n) = T(n-1) + O(n) and show it's O(n²).**
3. **What is tail call optimization and how does it limit stack depth?**
4. **Why is the average case only 39% worse than the theoretical optimum?**
5. **How does IntroSort guarantee O(n log n) worst case?**

---

[← Previous: Pivot Selection](03-pivot-selection.md) | [Next: Randomized Quick Sort →](05-randomized-quicksort.md)
