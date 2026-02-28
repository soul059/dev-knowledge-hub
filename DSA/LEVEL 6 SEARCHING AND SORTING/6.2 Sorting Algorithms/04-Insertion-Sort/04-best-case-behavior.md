# Chapter 4: Insertion Sort — Best Case Behavior

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: When Insertion Sort Shines →](05-when-insertion-sort-shines.md)

---

## Overview

Insertion Sort's O(n) best-case behavior is one of its most valuable properties. It's not just a theoretical curiosity — it makes Insertion Sort the **preferred algorithm for nearly sorted data** and a key component of hybrid sorting algorithms like TimSort.

---

## Why O(n) for Sorted Input

```
  Array: [1, 2, 3, 4, 5, 6, 7, 8]
  
  i=1: key=2, j=0: arr[0]=1 > 2? NO → insert at pos 1 (no shift)
  i=2: key=3, j=1: arr[1]=2 > 3? NO → insert at pos 2 (no shift)
  i=3: key=4, j=2: arr[2]=3 > 4? NO → insert at pos 3 (no shift)
  i=4: key=5, j=3: arr[3]=4 > 5? NO → insert at pos 4 (no shift)
  ...
  i=7: key=8, j=6: arr[6]=7 > 8? NO → insert at pos 7 (no shift)
  
  Each iteration: 1 comparison + 0 shifts = O(1) work
  Total: n-1 iterations × O(1) = O(n)
  
  ┌──────────────────────────────────────────┐
  │  The while loop body NEVER executes      │
  │  because arr[j] is always ≤ key.         │
  │  The inner loop exits immediately.       │
  └──────────────────────────────────────────┘
```

---

## Nearly Sorted Data (Few Inversions)

```
  Array: [1, 2, 4, 3, 5, 7, 6, 8]    (only 2 inversions: (4,3) and (7,6))
  
  i=1: key=2, compare 1<2 → 0 shifts
  i=2: key=4, compare 2<4 → 0 shifts
  i=3: key=3, compare 4>3 → shift 4, compare 2<3 → 1 shift
  i=4: key=5, compare 4<5 → 0 shifts
  i=5: key=7, compare 5<7 → 0 shifts
  i=6: key=6, compare 7>6 → shift 7, compare 5<6 → 1 shift
  i=7: key=8, compare 7<8 → 0 shifts
  
  Total: 9 comparisons + 2 shifts = O(n + k) where k = 2 inversions
  
  ┌──────────────────────────────────────────────────────────┐
  │  For data with k inversions:                             │
  │  Time = O(n + k)                                         │
  │                                                          │
  │  If k = O(n), then time = O(n)  → LINEAR!               │
  │  If k = O(n²), then time = O(n²) → quadratic            │
  └──────────────────────────────────────────────────────────┘
```

---

## When Is Data "Nearly Sorted"?

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Data is "nearly sorted" when each element is close to      │
  │  its final position. Formally, when inversions ≤ c×n        │
  │  for some constant c.                                       │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  Examples of nearly sorted data:                             │
  │                                                              │
  │  1. Adding a few new items to a sorted list                 │
  │     [1, 2, 3, 4, 5, 6, 7, 8, 3, 5]                        │
  │                                   ↑  ↑ new items            │
  │                                                              │
  │  2. Data that was sorted but slightly corrupted             │
  │     [1, 2, 4, 3, 5, 6, 8, 7, 9, 10]                       │
  │              ↕        ↕    two adjacent swaps               │
  │                                                              │
  │  3. Streaming data arriving mostly in order                  │
  │     Stock prices, sensor readings, log timestamps           │
  └──────────────────────────────────────────────────────────────┘
```

---

## Comparison: Best Case Across Algorithms

```
  Already sorted input [1, 2, 3, ..., n]:
  
  Algorithm        │ Comparisons │ Work Done │ Time
  ─────────────────┼─────────────┼───────────┼──────
  Insertion Sort   │    n-1      │  O(n)     │ O(n) ✓ FASTEST
  Bubble (optimized)│   n-1      │  O(n)     │ O(n) ✓
  Bubble (basic)   │  n(n-1)/2   │  O(n²)   │ O(n²)
  Selection Sort   │  n(n-1)/2   │  O(n²)   │ O(n²)
  Merge Sort       │  n/2·log n  │  O(n log n)│ O(n log n)
  Quick Sort       │  n·log n    │  O(n log n)│ O(n log n)
  Heap Sort        │  n·log n    │  O(n log n)│ O(n log n)
  
  For sorted data, Insertion Sort BEATS even O(n log n) algorithms!
```

---

## Practical Significance

Insertion Sort's best-case behavior is used inside real-world sorting libraries:

```
  TIMSORT (Python, Java's Arrays.sort for objects):
  ┌──────────────────────────────────────────────────────┐
  │  1. Detect "runs" (already sorted subsequences)     │
  │  2. Extend short runs using INSERTION SORT          │
  │  3. Merge runs using merge sort                     │
  │                                                      │
  │  Why Insertion Sort for small runs?                  │
  │  → O(n) for nearly sorted segments                  │
  │  → Small constant factors                            │
  │  → Cache-friendly                                    │
  └──────────────────────────────────────────────────────┘
  
  INTROSORT (C++ std::sort):
  ┌──────────────────────────────────────────────────────┐
  │  Uses Quick Sort for large arrays                    │
  │  Switches to INSERTION SORT for subarrays < 16      │
  │                                                      │
  │  Why? Insertion Sort has lower overhead              │
  │  and is faster for small n due to simplicity         │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Scenario | Inversions (k) | Time |
|----------|----------------|------|
| Sorted | 0 | O(n) |
| Nearly sorted | O(n) | O(n) |
| Random | O(n²) | O(n²) |
| Reverse sorted | n(n-1)/2 | O(n²) |
| Used in TimSort | Small runs | O(n) per run |
| Used in Introsort | n < 16 | O(n²) but tiny n |

---

## Quick Revision Questions

1. **Why does Insertion Sort achieve O(n) on sorted input?**
2. **What is the time complexity in terms of inversions?**
3. **Give three examples of "nearly sorted" data in practice.**
4. **Why does TimSort use Insertion Sort for small runs?**
5. **How many comparisons for a sorted array of size 1000?**
6. **Can Merge Sort or Quick Sort achieve O(n) on sorted input?**

---

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: When Insertion Sort Shines →](05-when-insertion-sort-shines.md)
