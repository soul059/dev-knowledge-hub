# Chapter 3: Selection Sort — Complexity Analysis

[← Previous: Implementation](02-implementation.md) | [Next: Stability Consideration →](04-stability-consideration.md)

---

## Overview

Selection Sort has a unique complexity profile: **O(n²) in ALL cases** for comparisons, but only **O(n) swaps**. This makes it interesting for scenarios where swaps (writes) are expensive.

---

## Time Complexity

### Comparisons — Always O(n²)

```
  Pass i  │  Comparisons  │  Range
  ────────┼───────────────┼──────────
    0     │     n - 1     │  j: 1 to n-1
    1     │     n - 2     │  j: 2 to n-1
    2     │     n - 3     │  j: 3 to n-1
   ...    │     ...       │
   n-2    │       1       │  j: n-1 to n-1
  ────────┼───────────────┤
  Total   │  (n-1)+(n-2)+...+1 = n(n-1)/2
  
  ┌──────────────────────────────────────────────────────┐
  │  Selection Sort ALWAYS makes n(n-1)/2 comparisons   │
  │  regardless of input order.                         │
  │                                                      │
  │  Best = Average = Worst = O(n²) comparisons         │
  └──────────────────────────────────────────────────────┘
```

### Swaps — Always O(n)

```
  Each pass makes AT MOST 1 swap:
  
  Pass 0: 1 swap (or 0 if min is already at index 0)
  Pass 1: 1 swap (or 0)
  ...
  Pass n-2: 1 swap (or 0)
  
  Total swaps: at most n-1 = O(n)
  
  Compare with Bubble Sort:
  ┌──────────────────┬────────────────┬──────────────┐
  │                  │ Comparisons    │ Swaps        │
  ├──────────────────┼────────────────┼──────────────┤
  │ Selection Sort   │ n(n-1)/2       │ ≤ n-1        │
  │ Bubble Sort      │ n(n-1)/2       │ up to n(n-1)/2│
  └──────────────────┴────────────────┴──────────────┘
  
  Selection Sort: O(n) swaps  ← HUGE advantage when swaps are costly
  Bubble Sort: O(n²) swaps
```

---

## Case Analysis

### Best Case: Already Sorted

```
  Input: [1, 2, 3, 4, 5]
  
  Pass 0: Scan [1,2,3,4,5] → min=1 at index 0 → no swap
  Pass 1: Scan [2,3,4,5]   → min=2 at index 1 → no swap
  Pass 2: Scan [3,4,5]     → min=3 at index 2 → no swap
  Pass 3: Scan [4,5]       → min=4 at index 3 → no swap
  
  Comparisons: 4+3+2+1 = 10 = n(n-1)/2       ← STILL O(n²)
  Swaps: 0
  
  NOTE: Unlike optimized Bubble Sort which achieves O(n),
  Selection Sort CANNOT detect sorted input early!
```

### Worst Case: Reverse Sorted (for swaps)

```
  Input: [5, 4, 3, 2, 1]
  
  Pass 0: min=1 at index 4 → swap(5,1) → [1,4,3,2,5]  ← 1 swap
  Pass 1: min=2 at index 3 → swap(4,2) → [1,2,3,4,5]  ← 1 swap
  Pass 2: min=3 at index 2 → no swap                   ← 0 swaps
  Pass 3: min=4 at index 3 → no swap                   ← 0 swaps
  
  Comparisons: 10 → O(n²)
  Swaps: 2 ← Even worst case has few swaps!
  
  Maximum possible swaps for any input: n-1
```

---

## Space Complexity: O(1)

```
  Variables used:
  ┌──────────────────────────────┐
  │  i        → loop counter    │
  │  j        → loop counter    │
  │  minIndex → index tracker   │
  │  temp     → for swap        │
  ├──────────────────────────────┤
  │  Total: 4 variables = O(1)  │
  └──────────────────────────────┘
```

---

## Complete Complexity Summary

```
  ┌───────────────────────────────────────────────────────┐
  │            SELECTION SORT COMPLEXITY                   │
  ├──────────────┬──────────────┬──────────────────────────┤
  │   Case       │ Comparisons  │  Swaps                  │
  ├──────────────┼──────────────┼──────────────────────────┤
  │  Best        │   O(n²)      │  0                      │
  │  Average     │   O(n²)      │  O(n)                   │
  │  Worst       │   O(n²)      │  O(n)  [at most n-1]    │
  ├──────────────┼──────────────┴──────────────────────────┤
  │  Space       │           O(1)                         │
  ├──────────────┼─────────────────────────────────────────┤
  │  Stable      │           No                           │
  │  Adaptive    │           No                           │
  └──────────────┴─────────────────────────────────────────┘
```

---

## When O(n) Swaps Matter

```
  Scenario: Sorting records on a FLASH DRIVE
  
  Flash memory has limited write cycles:
  - Each cell can only be written ~10,000 times
  - Reads are unlimited
  
  Bubble Sort: O(n²) writes  → wears out flash faster
  Selection Sort: O(n) writes → extends flash lifetime!
  
  ┌──────────────────────────────────────────────────────┐
  │  When WRITES are expensive, Selection Sort's         │
  │  O(n) swap count is a significant advantage.        │
  └──────────────────────────────────────────────────────┘
  
  Other expensive-write scenarios:
  • EEPROM memory in embedded systems
  • Network transfers (each swap = 2 network ops)
  • Database row updates (each swap = disk I/O)
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Best case time | O(n²) |
| Average case time | O(n²) |
| Worst case time | O(n²) |
| Space | O(1) |
| Comparisons (all cases) | n(n-1)/2 |
| Swaps (max) | n-1 |
| Adaptive | No |
| Key advantage | Minimum number of swaps |

---

## Quick Revision Questions

1. **Why is Selection Sort O(n²) even for sorted input?**
2. **What is the maximum number of swaps for an array of size n?**
3. **Why does Selection Sort have fewer swaps than Bubble Sort?**
4. **In what scenario is Selection Sort's low swap count beneficial?**
5. **Is Selection Sort adaptive? What does that mean?**
6. **Calculate exact comparisons and swaps for [3, 1, 4, 2].**

---

[← Previous: Implementation](02-implementation.md) | [Next: Stability Consideration →](04-stability-consideration.md)
