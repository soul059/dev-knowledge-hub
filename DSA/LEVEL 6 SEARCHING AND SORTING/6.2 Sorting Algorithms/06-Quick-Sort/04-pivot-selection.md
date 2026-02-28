# Unit 6: Quick Sort — Pivot Selection Strategies

[← Previous: Implementation](03-implementation.md) | [Next: Complexity Analysis →](05-complexity-analysis.md)

---

## Overview

The choice of pivot **makes or breaks** Quick Sort. A good pivot splits the array roughly in half (O(n log n)), while a bad pivot creates highly unbalanced splits (O(n²)). This chapter explores all major pivot selection strategies.

---

## Strategy 1: First/Last Element

```
  Simplest approach: always pick first or last element.
  
  PROBLEM: Catastrophic on sorted/nearly sorted data!
  
  Sorted: [1, 2, 3, 4, 5, 6, 7, 8]   pivot = 8 (last)
  
  Partition: [1, 2, 3, 4, 5, 6, 7] [8] []
                                     ↑
  One side has n-1 elements, other has 0!
  
  Recursion tree becomes:
  
  n
  └── n-1
      └── n-2
          └── n-3
              └── ...          Height = n → O(n²)
  
  This happens for: sorted, reverse sorted, nearly sorted input.
  These are VERY COMMON in practice!
```

---

## Strategy 2: Random Pivot

```
  Pick a random element as pivot.
  
  EXPECTED performance: O(n log n) for ANY input.
  
  ┌──────────────────────────────────────────────────────────┐
  │  Why randomization works:                                │
  │                                                          │
  │  For worst case to occur, the random choices would      │
  │  need to consistently pick the smallest/largest element │
  │  Probability: (2/n) × (2/(n-1)) × ... ≈ 2ⁿ/n!        │
  │  This is ASTRONOMICALLY unlikely for large n            │
  │                                                          │
  │  Expected # comparisons = 2n·ln(n) ≈ 1.39·n·log₂(n)   │
  └──────────────────────────────────────────────────────────┘
```

```java
int randomizedPartition(int[] arr, int low, int high) {
    int randIndex = low + random.nextInt(high - low + 1);
    swap(arr, randIndex, high);
    return partition(arr, low, high);
}
```

---

## Strategy 3: Median-of-Three

```
  Pick the MEDIAN of first, middle, and last elements.
  
  Array: [8, 2, 4, 7, 1, 3, 9, 6, 5]
  
  First:  arr[0] = 8
  Middle: arr[4] = 1
  Last:   arr[8] = 5
  
  Sort these three: [1, 5, 8] → Median = 5
  
  Use 5 as pivot!
  
  ┌──────────────────────────────────────────────────────────┐
  │  Benefits:                                               │
  │  1. Avoids worst case on sorted input (median of        │
  │     first/mid/last of sorted array IS the median)       │
  │  2. No random number generation overhead                │
  │  3. Better pivot than random on average                 │
  │  4. Used by most production implementations             │
  └──────────────────────────────────────────────────────────┘
```

```java
int medianOfThree(int[] arr, int low, int high) {
    int mid = low + (high - low) / 2;
    
    // Sort arr[low], arr[mid], arr[high]
    if (arr[low] > arr[mid])  swap(arr, low, mid);
    if (arr[low] > arr[high]) swap(arr, low, high);
    if (arr[mid] > arr[high]) swap(arr, mid, high);
    
    // arr[mid] is now the median
    // Move median to high-1 position (out of the way)
    swap(arr, mid, high - 1);
    return arr[high - 1];  // This is the pivot
}

// Or simpler: just find median index without rearranging
int medianOfThreeIndex(int[] arr, int low, int high) {
    int mid = low + (high - low) / 2;
    
    if ((arr[low] <= arr[mid] && arr[mid] <= arr[high]) ||
        (arr[high] <= arr[mid] && arr[mid] <= arr[low]))
        return mid;
    if ((arr[mid] <= arr[low] && arr[low] <= arr[high]) ||
        (arr[high] <= arr[low] && arr[low] <= arr[mid]))
        return low;
    return high;
}
```

---

## Strategy 4: Median-of-Medians (Guaranteed O(n log n))

```
  ┌──────────────────────────────────────────────────────────┐
  │  Guarantee O(n log n) WORST CASE by choosing the       │
  │  TRUE median (or approximate median) as pivot.          │
  │                                                          │
  │  Algorithm (BFPRT / Median of Medians):                 │
  │  1. Divide array into groups of 5                       │
  │  2. Find median of each group (constant time)           │
  │  3. Recursively find median of those medians            │
  │  4. Use that as pivot                                    │
  │                                                          │
  │  Guarantees at least 30% of elements on each side!      │
  │  T(n) = T(n/5) + T(7n/10) + O(n) = O(n)               │
  │                                                          │
  │  BUT: Constant factor is ~5x larger than random pivot   │
  │  RARELY used in practice — randomization is simpler     │
  └──────────────────────────────────────────────────────────┘
```

---

## Comparison of Pivot Strategies

```
  Input: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] (sorted)
  
  ┌──────────────────┬────────────┬──────────────────┐
  │  Strategy        │  Pivot     │  Split            │
  ├──────────────────┼────────────┼──────────────────┤
  │  First element   │  1         │  [] [1] [2..10]  │
  │  Last element    │  10        │  [1..9] [10] []  │
  │  Random          │  varies    │  varies           │
  │  Median of 3     │  5 or 6    │  ~half and half  │
  │  True median     │  5 or 6    │  exact half      │
  └──────────────────┴────────────┴──────────────────┘
  
  First/Last → 0 : n-1 split → O(n²)
  Median of 3 → ~n/2 : n/2 split → O(n log n)  ✓
```

---

## Effect of Pivot Quality on Performance

```
  Pivot quality = rank of pivot in the subarray (0 = worst, n/2 = best)
  
  BEST pivot (median):
  ┌───────────────────┤═══════════╠───────────────────┐
  │      n/2          │  PIVOT    │      n/2          │
  └───────────────────┴───────────┴───────────────────┘
  Depth: log₂(n)   Total: O(n log n)
  
  GOOD pivot (25th-75th percentile):
  ┌────────────╠═════════════════════════════════════════┐
  │    n/4     │  PIVOT + remaining 3n/4                │
  └────────────┴────────────────────────────────────────┘
  Depth: log₄/₃(n) ≈ 3.4·log₂(n)   Still O(n log n)!
  
  BAD pivot (min or max):
  ║═══════════════════════════════════════════════════════┐
  │  PIVOT + remaining n-1                                │
  └───────────────────────────────────────────────────────┘
  Depth: n    Total: O(n²)
  
  KEY INSIGHT: Even mediocre pivots (not exactly median)
  give O(n log n). Only EXTREME pivots cause O(n²).
  
  Any constant fraction split (30:70, 20:80, even 1:99):
  Depth = log(n) / log(1/fraction) = O(log n)
  Time = O(n log n)
```

---

## What Production Libraries Use

```
  ┌──────────────────────────────────────────────────────────┐
  │  C++ std::sort (GCC):                                   │
  │    Median-of-three for pivot                            │
  │    Falls back to heap sort if recursion too deep        │
  │    → Introsort (guarantees O(n log n) worst case)       │
  │                                                          │
  │  Java Arrays.sort (primitives):                          │
  │    Dual-pivot Quick Sort (two pivots!)                   │
  │    Uses insertion sort for n < 47                       │
  │    Median-of-five for pivot selection                    │
  │                                                          │
  │  .NET Array.Sort:                                        │
  │    Introsort (Quick + Heap + Insertion)                  │
  │    Median-of-three for pivot                            │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Strategy | Worst Case | Average | Overhead | Practical Use |
|----------|----------|---------|----------|---------------|
| First/Last | O(n²) on sorted | O(n log n) | None | Teaching only |
| Random | O(n²) rare | O(n log n) | RNG call | Good general choice |
| Median-of-3 | O(n²) rare | O(n log n) | 3 comparisons | Most production sorts |
| Median-of-medians | O(n log n) guaranteed | O(n log n) | O(n) extra work | Theory, rarely practical |

---

## Quick Revision Questions

1. **Why is first/last element a bad pivot for sorted data?**
2. **What is the expected number of comparisons with random pivot?**
3. **Explain median-of-three: which three elements, and why?**
4. **Why does even a 25:75 split still give O(n log n)?**
5. **What pivot strategy does C++ std::sort use?**
6. **Why is median-of-medians rarely used despite guaranteed O(n log n)?**

---

[← Previous: Implementation](03-implementation.md) | [Next: Complexity Analysis →](05-complexity-analysis.md)
