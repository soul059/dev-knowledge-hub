# Chapter 3: Insertion Sort — Complexity Analysis

[← Previous: Implementation](02-implementation.md) | [Next: Best Case Behavior →](04-best-case-behavior.md)

---

## Time Complexity

### Best Case: O(n) — Already Sorted

```
  Input: [1, 2, 3, 4, 5]
  
  i=1: key=2, compare with 1 → 1 < 2 → STOP    (1 comparison, 0 shifts)
  i=2: key=3, compare with 2 → 2 < 3 → STOP    (1 comparison, 0 shifts)
  i=3: key=4, compare with 3 → 3 < 4 → STOP    (1 comparison, 0 shifts)
  i=4: key=5, compare with 4 → 4 < 5 → STOP    (1 comparison, 0 shifts)
  
  Total: 4 comparisons, 0 shifts = n-1 = O(n)
  
  ┌──────────────────────────────────────────────────┐
  │  Each element only needs ONE comparison to       │
  │  confirm it's already in the right place.        │
  │  This is the BEST possible — you must at least   │
  │  check every element once!                       │
  └──────────────────────────────────────────────────┘
```

### Worst Case: O(n²) — Reverse Sorted

```
  Input: [5, 4, 3, 2, 1]
  
  i=1: key=4, shift 5           → [4,5,3,2,1]   (1 comparison, 1 shift)
  i=2: key=3, shift 5,4         → [3,4,5,2,1]   (2 comparisons, 2 shifts)
  i=3: key=2, shift 5,4,3       → [2,3,4,5,1]   (3 comparisons, 3 shifts)
  i=4: key=1, shift 5,4,3,2     → [1,2,3,4,5]   (4 comparisons, 4 shifts)
  
  Total comparisons: 1+2+3+4 = 10 = n(n-1)/2 = O(n²)
  Total shifts:      1+2+3+4 = 10 = n(n-1)/2
  Total assignments: shifts + n-1 (key saves) + n-1 (key inserts) 
                   = n(n-1)/2 + 2(n-1) = O(n²)
```

### Average Case: O(n²)

```
  On average, each element is inserted in the MIDDLE
  of the sorted portion:
  
  Element at index i → shifts ≈ i/2 on average
  
  Total shifts = Σ(i/2) for i=1 to n-1
               = (1/2) × n(n-1)/2
               = n(n-1)/4
               = O(n²)
  
  Average case is still O(n²), but with half the 
  constant factor of worst case.
```

---

## Space Complexity: O(1)

```
  Variables:
  ┌──────────────────────────────┐
  │  i    → loop counter         │
  │  j    → inner loop counter   │
  │  key  → element to insert    │
  ├──────────────────────────────┤
  │  Total: 3 variables = O(1)  │
  └──────────────────────────────┘
```

---

## Complexity Summary

```
  ┌──────────────────────────────────────────────────────────────┐
  │             INSERTION SORT COMPLEXITY                        │
  ├──────────────┬──────────────┬──────────────┬────────────────┤
  │   Case       │ Comparisons  │    Shifts    │  Time          │
  ├──────────────┼──────────────┼──────────────┼────────────────┤
  │  Best        │    n - 1     │      0       │  O(n)          │
  │  Average     │   n(n-1)/4   │   n(n-1)/4   │  O(n²)        │
  │  Worst       │   n(n-1)/2   │   n(n-1)/2   │  O(n²)        │
  ├──────────────┼──────────────┴──────────────┼────────────────┤
  │  Space       │        O(1) auxiliary        │  In-place      │
  └──────────────┴──────────────────────────────┴────────────────┘
```

---

## Inversions and Insertion Sort

```
  The number of shifts = the number of INVERSIONS in the array.
  
  Array: [3, 1, 4, 2]
  Inversions: (3,1), (3,2), (4,2) = 3 inversions
  
  Insertion Sort shifts:
  i=1: key=1, shift 3 → 1 shift
  i=2: key=4, no shift → 0 shifts
  i=3: key=2, shift 4, shift 3 → 2 shifts
  Total shifts = 3 = number of inversions ✓
  
  ┌──────────────────────────────────────────────────────┐
  │  T(n) = O(n + inversions)                            │
  │                                                      │
  │  Sorted array: 0 inversions → O(n)                  │
  │  Random array: ~n²/4 inversions → O(n²)              │
  │  Reverse sorted: n(n-1)/2 inversions → O(n²)        │
  └──────────────────────────────────────────────────────┘
```

---

## Comparison with Other O(n²) Sorts

| Metric | Bubble | Selection | Insertion |
|--------|--------|-----------|-----------|
| Best time | O(n)* | O(n²) | **O(n)** |
| Average time | O(n²) | O(n²) | O(n²) |
| Worst time | O(n²) | O(n²) | O(n²) |
| Best swaps/shifts | 0 | 0 | 0 |
| Worst swaps/shifts | n(n-1)/2 | n-1 | n(n-1)/2 |
| Assignments per move | 3 (swap) | 3 (swap) | **1 (shift)** |

*With optimization

---

## Summary Table

| Metric | Value |
|--------|-------|
| Best case | O(n) — already sorted |
| Average case | O(n²) |
| Worst case | O(n²) — reverse sorted |
| Space | O(1) |
| # shifts = | # inversions |
| Assignment efficiency | Shifts use 1/3 assignments of swaps |

---

## Quick Revision Questions

1. **Why is Insertion Sort O(n) for sorted input?**
2. **What input produces the worst case?**
3. **What is the relationship between inversions and shifts?**
4. **How does the average case compare to worst case?**
5. **Why are shifts more efficient than swaps in terms of assignments?**
6. **For an array with k inversions, what is the time complexity?**

---

[← Previous: Implementation](02-implementation.md) | [Next: Best Case Behavior →](04-best-case-behavior.md)
