# Chapter 3: Pivot Selection Strategies

## Overview

The **pivot** determines how balanced the partition is, which directly impacts Quick Sort's performance. A good pivot splits the array roughly in half (O(n log n)); a bad one creates extreme imbalance (O(n²)). This chapter covers all major pivot selection strategies.

---

## Why Pivot Choice Matters

```
n = 8 elements

GOOD pivot (median):          BAD pivot (min or max):
    [4 elements][p][4 elems]      [][p][7 elements]
    
    Depth: log₂(8) = 3           Depth: 7
    Work: 8 × 3 = 24             Work: 8+7+6+5+4+3+2+1 = 36
    
    O(n log n)                    O(n²)
```

---

## Strategy 1: Fixed Position

### Last Element (Lomuto default)

```
pivot = arr[hi]

✓ Simplest implementation
✗ O(n²) on sorted/nearly-sorted arrays
✗ O(n²) on reverse-sorted arrays
✗ Predictable → vulnerable to adversarial input
```

### First Element

```
pivot = arr[lo]

✓ Simple
✗ Same problems as last element
✗ O(n²) on sorted arrays
```

---

## Strategy 2: Random Pivot

```
function randomPartition(arr, lo, hi):
    randomIndex = random(lo, hi)
    swap(arr[randomIndex], arr[hi])   // move to end
    return lomutoPartition(arr, lo, hi)
```

```
┌──────────────────────────────────────────────────────────┐
│  RANDOMIZED QUICK SORT                                    │
│                                                           │
│  Expected time: O(n log n) for ANY input                  │
│  Worst case: O(n²) but probability is 1/n!                │
│                                                           │
│  ✓ No adversarial input can force worst case              │
│  ✓ Expected O(n log n) regardless of input distribution   │
│  ✓ Simple to implement                                    │
│  ✗ Non-deterministic (different runs, different times)     │
│  ✗ Random number generation has small overhead             │
└──────────────────────────────────────────────────────────┘
```

---

## Strategy 3: Median of Three

Choose the **median** of the first, middle, and last elements.

```
function medianOfThree(arr, lo, hi):
    mid = lo + (hi - lo) / 2
    
    // Sort arr[lo], arr[mid], arr[hi]
    if arr[lo] > arr[mid]: swap(arr[lo], arr[mid])
    if arr[lo] > arr[hi]:  swap(arr[lo], arr[hi])
    if arr[mid] > arr[hi]: swap(arr[mid], arr[hi])
    
    // Now arr[lo] ≤ arr[mid] ≤ arr[hi]
    // Use arr[mid] as pivot
    swap(arr[mid], arr[hi - 1])    // move pivot near end
    return arr[hi - 1]              // return pivot value
```

### Example

```
Array: [8, 3, 1, 9, 7, 5, 2, 6]

lo=0: arr[0]=8
mid=3: arr[3]=9
hi=7: arr[7]=6

Sort these three: 6, 8, 9
Median = 8

Much better than picking 8 (first) or 6 (last)!
```

```
✓ Avoids worst case on sorted/reverse-sorted input
✓ Better average performance than fixed position
✓ Deterministic
✓ Used in many standard library implementations
✗ Can still be adversarially defeated (rare)
✗ Doesn't guarantee balanced split
```

---

## Strategy 4: Median of Medians (Guaranteed O(n log n))

Guarantees a "good enough" pivot — covered in detail in Unit 6 (Quick Select). The idea:

```
1. Divide array into groups of 5
2. Find median of each group
3. Recursively find median of those medians
4. Use that as pivot

Guarantees at least 30% on each side
→ T(n) ≤ T(7n/10) + T(n/5) + O(n) = O(n)
→ Quick Sort with this: guaranteed O(n log n)

But the constant factor is too large for practical use!
```

---

## Strategy 5: Ninther (Median of Medians of Three)

Used in some industrial implementations (e.g., Bentley-McIlroy):

```
For large arrays (n > 40):
  1. Pick three groups of three elements (9 total)
  2. Find median of each group of three
  3. Use median of those three medians

    Group 1: arr[lo], arr[lo+step], arr[lo+2*step]    → m1
    Group 2: arr[mid-step], arr[mid], arr[mid+step]   → m2
    Group 3: arr[hi-2*step], arr[hi-step], arr[hi]    → m3
    
    Pivot = median(m1, m2, m3)
```

---

## Comparison of Strategies

| Strategy | Sorted Input | Random Input | Adversarial | Deterministic |
|----------|-------------|--------------|-------------|---------------|
| First/Last | O(n²) | O(n log n) avg | O(n²) | Yes |
| Random | O(n log n) exp | O(n log n) exp | O(n log n) exp | No |
| Median of 3 | O(n log n) | O(n log n) | O(n²)* | Yes |
| Median of Medians | O(n log n) | O(n log n) | O(n log n) | Yes |
| Ninther | O(n log n) | O(n log n) | Hard to attack | Yes |

*Adversary can construct input to defeat median-of-three, but it's very rare.

---

## Practical Recommendation

```
┌─────────────────────────────────────────────────────────┐
│  For most applications:                                  │
│    → Median of Three for deterministic performance       │
│    → Random pivot for simplicity + adversarial safety    │
│                                                          │
│  For competitive programming:                            │
│    → Random pivot (simple, safe)                         │
│                                                          │
│  For production/library code:                            │
│    → Median of Three + switch to insertion sort          │
│      for small subarrays (n < 10-20)                     │
│    → Three-way partition for handling duplicates          │
│                                                          │
│  Theory/interview:                                       │
│    → Know median-of-medians guarantees O(n log n)        │
└─────────────────────────────────────────────────────────┘
```

---

## IntroSort: The Best of All Worlds

Modern C++ `std::sort` uses **IntroSort** (Introspective Sort):

```
function introSort(arr, lo, hi, depthLimit):
    if hi - lo < 16:
        insertionSort(arr, lo, hi)     // small arrays
        return
    
    if depthLimit == 0:
        heapSort(arr, lo, hi)          // worst case protection
        return
    
    pivot = medianOfThree(arr, lo, hi)
    p = partition(arr, lo, hi, pivot)
    introSort(arr, lo, p - 1, depthLimit - 1)
    introSort(arr, p + 1, hi, depthLimit - 1)

// depth limit = 2 × ⌊log₂(n)⌋
```

```
✓ O(n log n) guaranteed (falls back to heap sort)
✓ Fast in practice (quick sort for most of the work)
✓ Small subarray optimization (insertion sort)
✓ Used in C++ STL, .NET, and many production systems
```

---

## Summary Table

| Strategy | Time (worst) | Time (avg) | Overhead | Best For |
|----------|-------------|------------|----------|----------|
| Fixed position | O(n²) | O(n log n) | None | Teaching only |
| Random | O(n²) rare | O(n log n) | rand() call | General use |
| Median of 3 | O(n²) rare | O(n log n) | 3 comparisons | Production |
| Median of Medians | O(n log n) | O(n log n) | O(n) | Theoretical |
| IntroSort | O(n log n) | O(n log n) | Depth check | C++ STL |

---

## Quick Revision Questions

1. **Why does choosing the first element as pivot cause O(n²) on sorted arrays?**
2. **How does randomized pivot selection prevent adversarial worst cases?**
3. **What are the three elements used in "median of three"?**
4. **What is IntroSort and what three algorithms does it combine?**
5. **Why isn't median-of-medians used as the default pivot strategy?**

---

[← Previous: Partition Schemes](02-partition-schemes.md) | [Next: Worst Case Analysis →](04-worst-case-analysis.md)
