# Chapter 5: Time Complexity of Array Operations

[← Previous: Static vs Dynamic Arrays](04-static-vs-dynamic-arrays.md) | [Back to README](../README.md) | [Next: Array vs Other Structures →](06-array-vs-other-structures.md)

---

## Overview

Time complexity tells us how the running time of an operation grows as the input size increases. This chapter provides a detailed analysis of every array operation.

---

## Big-O Notation Refresher

| Notation | Name | Example |
|----------|------|---------|
| O(1) | Constant | Array access by index |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search, traversal |
| O(n log n) | Linearithmic | Merge sort, quick sort |
| O(n²) | Quadratic | Nested loops, bubble sort |

```
  Time
  ▲
  │                                    O(n²)
  │                                  ╱
  │                                ╱
  │                             ╱
  │                          ╱
  │                    ╱───── O(n log n)
  │              ╱────
  │         ╱──── O(n)
  │     ╱───
  │  ╱── O(log n)
  │──────────────────────── O(1)
  └──────────────────────────────► n (input size)
```

---

## Detailed Complexity Analysis

### 1. Access — O(1)

```
arr[i]   →   Base + i × ElementSize   →   single arithmetic operation

No matter if the array has 10 or 10 million elements,
accessing arr[i] takes the SAME time.

  n = 10:       arr[5]  → 1 step
  n = 1,000:    arr[500] → 1 step
  n = 1,000,000: arr[999999] → 1 step
```

### 2. Search (Unsorted) — O(n)

```
FUNCTION linearSearch(arr, n, target):
    FOR i = 0 TO n-1:                    // up to n iterations
        IF arr[i] == target:
            RETURN i
    RETURN -1

Best case:  O(1)  — target at arr[0]
Worst case: O(n)  — target at arr[n-1] or not present
Average:    O(n/2) = O(n)
```

### 3. Search (Sorted) — O(log n)

```
FUNCTION binarySearch(arr, n, target):
    low = 0, high = n - 1
    WHILE low <= high:                   // log₂(n) iterations
        mid = (low + high) / 2
        IF arr[mid] == target: RETURN mid
        ELSE IF arr[mid] < target: low = mid + 1
        ELSE: high = mid - 1
    RETURN -1

n = 1,000,000 → only ~20 comparisons!
```

### 4. Insertion — O(1) to O(n)

```
  Position        Best    Worst    Average
  ─────────────────────────────────────────
  End              O(1)    O(1)     O(1)
  Beginning        O(n)    O(n)     O(n)
  Middle (idx k)   O(1)    O(n)     O(n/2) = O(n)
  
  Why? Must shift elements right:
  
  Insert at index k → shift (n - k) elements
  k = n (end)    → shift 0 elements    → O(1)
  k = 0 (begin)  → shift n elements    → O(n)
  k = n/2 (mid)  → shift n/2 elements  → O(n)
```

### 5. Deletion — O(1) to O(n)

```
  Position        Best    Worst    Average
  ─────────────────────────────────────────
  End              O(1)    O(1)     O(1)
  Beginning        O(n)    O(n)     O(n)
  Middle (idx k)   O(1)    O(n)     O(n/2) = O(n)
  
  Why? Must shift elements left:
  
  Delete at index k → shift (n - k - 1) elements
  k = n-1 (end)  → shift 0 elements    → O(1)
  k = 0 (begin)  → shift n-1 elements  → O(n)
```

### 6. Dynamic Array Append — Amortized O(1)

```
  Append sequence with doubling (starting cap = 1):

  Insert #   Cap Before   Resize?   Copy Cost   Insert Cost
  ────────   ──────────   ───────   ─────────   ───────────
     1          1          Yes(→2)     0 + 1        2
     2          2          Yes(→4)     2            3
     3          4          No          0            1
     4          4          Yes(→8)     4            5
     5          8          No          0            1
     6          8          No          0            1
     7          8          No          0            1
     8          8          Yes(→16)    8            9
     9         16          No          0            1
    ...
  
  Total cost for n inserts: n + (1 + 2 + 4 + ... + n) = n + 2n - 1 ≈ 3n
  Amortized per insert: 3n / n = 3 = O(1)
```

---

## Space Complexity of Arrays

| Structure | Space |
|-----------|-------|
| Static array of size n | O(n) |
| Dynamic array (size n, cap c) | O(c), where n ≤ c ≤ 2n |
| Auxiliary space for operations | Usually O(1) |
| Copy during resize | O(n) temporary |

---

## Comparing Operations Visually

```
  O P E R A T I O N   C O S T   C H A R T

  Operation          O(1)              O(n)
                     │                 │
  Access         ████│                 │
                     │                 │
  Search(sorted) ────│──████           │   O(log n)
                     │                 │
  Search(unsort) ────│─────────────────│████
                     │                 │
  Insert(end)    ████│                 │
                     │                 │
  Insert(begin)  ────│─────────────────│████
                     │                 │
  Delete(end)    ████│                 │
                     │                 │
  Delete(begin)  ────│─────────────────│████
                     │                 │
  Append(dyn)    ████│*                │   *amortized
                     │                 │
```

---

## Practical Impact of Complexity

| n | O(1) | O(log n) | O(n) | O(n²) |
|---|------|----------|------|--------|
| 10 | 1 op | 3 ops | 10 ops | 100 ops |
| 100 | 1 op | 7 ops | 100 ops | 10,000 ops |
| 1,000 | 1 op | 10 ops | 1,000 ops | 1,000,000 ops |
| 1,000,000 | 1 op | 20 ops | 1,000,000 ops | 10¹² ops |

At 10⁸ operations/second:
- O(n) with n = 10⁶ → 0.01 seconds
- O(n²) with n = 10⁶ → ~11.5 days!

---

## When Complexity Matters — Rules of Thumb

```
  ┌──────────────────────────────────────────────┐
  │  COMPETITIVE PROGRAMMING TIME LIMITS (~1-2s)  │
  │                                                │
  │  n ≤ 10       →  O(n!) is fine                │
  │  n ≤ 20       →  O(2ⁿ) is fine               │
  │  n ≤ 500      →  O(n³) is fine                │
  │  n ≤ 5,000    →  O(n²) is fine                │
  │  n ≤ 10⁶      →  O(n log n) is fine           │
  │  n ≤ 10⁸      →  O(n) is fine                 │
  │  n ≤ 10¹⁸     →  O(log n) or O(1) needed      │
  └──────────────────────────────────────────────┘
```

---

## Summary Table

| Operation | Best | Average | Worst | Space |
|-----------|------|---------|-------|-------|
| Access by index | O(1) | O(1) | O(1) | O(1) |
| Linear search | O(1) | O(n) | O(n) | O(1) |
| Binary search | O(1) | O(log n) | O(log n) | O(1) |
| Insert at end | O(1) | O(1) | O(1)* | O(1) |
| Insert at beginning | O(n) | O(n) | O(n) | O(1) |
| Insert at index k | O(1) | O(n) | O(n) | O(1) |
| Delete at end | O(1) | O(1) | O(1) | O(1) |
| Delete at beginning | O(n) | O(n) | O(n) | O(1) |
| Append (dynamic) | O(1) | O(1)† | O(n) | O(n) |

*\* assuming space is available*
*† amortized*

---

## Quick Revision Questions

1. **Why is binary search O(log n)?** How many comparisons for an array of 1 million elements?

2. **Explain amortized analysis** for dynamic array appends. Total work for n inserts?

3. **Best/worst case for linear search:** When does each occur?

4. **If an interview says "optimize from O(n²) to O(n)"**, what techniques can achieve this for array problems?

5. **Why is O(n) deletion at the beginning unavoidable** (while maintaining order)?

6. **Given constraints n ≤ 10⁵**, what time complexities are acceptable within a 1-second time limit?

---

[← Previous: Static vs Dynamic Arrays](04-static-vs-dynamic-arrays.md) | [Back to README](../README.md) | [Next: Array vs Other Structures →](06-array-vs-other-structures.md)
