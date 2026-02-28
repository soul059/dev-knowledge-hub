# Chapter 1: Bubble Sort — Algorithm Description

[← Previous Unit: Sorting Lower Bound](../01-Sorting-Fundamentals/06-sorting-lower-bound.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Bubble Sort is the simplest comparison-based sorting algorithm. It works by repeatedly stepping through the list, comparing adjacent elements, and **swapping** them if they are in the wrong order. The largest unsorted element "bubbles up" to its correct position in each pass.

---

## Core Idea

```
  Like bubbles rising in water — the LARGEST element
  "bubbles up" to the end of the array in each pass.
  
       ○         ← largest element rises
      ○ ○
     ○ ○ ○
    ○ ○ ○ ○
   ─────────── water surface (end of array)
```

---

## How It Works — Step by Step

### Pass 1: Find the largest, bubble it to the end

```
  Array: [5, 3, 8, 1, 2]
  
  Compare adjacent pairs, swap if left > right:
  
  Step 1: [5, 3, 8, 1, 2]      Compare 5 and 3 → 5 > 3 → SWAP
           ↑  ↑
          [3, 5, 8, 1, 2]
  
  Step 2: [3, 5, 8, 1, 2]      Compare 5 and 8 → 5 < 8 → no swap
              ↑  ↑
          [3, 5, 8, 1, 2]
  
  Step 3: [3, 5, 8, 1, 2]      Compare 8 and 1 → 8 > 1 → SWAP
                 ↑  ↑
          [3, 5, 1, 8, 2]
  
  Step 4: [3, 5, 1, 8, 2]      Compare 8 and 2 → 8 > 2 → SWAP
                    ↑  ↑
          [3, 5, 1, 2, 8]      ← 8 is now in its CORRECT position!
                          ✓
```

### Pass 2: Bubble the next largest

```
  Array: [3, 5, 1, 2, | 8]     (8 is already sorted)
  
  Step 1: [3, 5, 1, 2]         Compare 3 and 5 → 3 < 5 → no swap
           ↑  ↑
  
  Step 2: [3, 5, 1, 2]         Compare 5 and 1 → 5 > 1 → SWAP
              ↑  ↑
          [3, 1, 5, 2]
  
  Step 3: [3, 1, 5, 2]         Compare 5 and 2 → 5 > 2 → SWAP
                 ↑  ↑
          [3, 1, 2, 5, | 8]    ← 5 is now in correct position!
                    ✓    ✓
```

### Pass 3:

```
  Array: [3, 1, 2, | 5, 8]
  
  Step 1: [3, 1, 2]            Compare 3 and 1 → 3 > 1 → SWAP
           ↑  ↑
          [1, 3, 2]
  
  Step 2: [1, 3, 2]            Compare 3 and 2 → 3 > 2 → SWAP
              ↑  ↑
          [1, 2, 3, | 5, 8]    ← 3 in correct position!
                 ✓    ✓   ✓
```

### Pass 4:

```
  Array: [1, 2, | 3, 5, 8]
  
  Step 1: [1, 2]               Compare 1 and 2 → 1 < 2 → no swap
           ↑  ↑
          [1, 2, | 3, 5, 8]    ← All sorted!
           ✓  ✓    ✓   ✓   ✓
```

---

## Complete Trace Visualization

```
  Original: [5, 3, 8, 1, 2]
  
  Pass 1:   [3, 5, 1, 2, 8]    ← 8 bubbles to end
                            ■
  Pass 2:   [3, 1, 2, 5, 8]    ← 5 bubbles to position
                        ■  ■
  Pass 3:   [1, 2, 3, 5, 8]    ← 3 bubbles to position
                    ■  ■  ■
  Pass 4:   [1, 2, 3, 5, 8]    ← No swaps needed, array sorted
                ■  ■  ■  ■
  
  Final:    [1, 2, 3, 5, 8]    ✓ SORTED
             ■  ■  ■  ■  ■
  
  ■ = element in its final correct position
```

---

## Pseudocode

```
BUBBLE-SORT(A, n):
    for i = 0 to n-2:                    // n-1 passes
        for j = 0 to n-2-i:              // each pass shrinks by 1
            if A[j] > A[j+1]:            // compare adjacent
                swap(A[j], A[j+1])       // swap if wrong order
```

---

## Key Observations

```
  ┌─────────────────────────────────────────────────────────┐
  │  1. After pass i, the i-th LARGEST element is in its   │
  │     correct final position.                            │
  │                                                         │
  │  2. The unsorted portion shrinks by 1 after each pass. │
  │                                                         │
  │  3. Requires at most n-1 passes.                       │
  │                                                         │
  │  4. Each pass makes at most n-1-i comparisons.         │
  │                                                         │
  │  5. Total comparisons:                                 │
  │     (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²)       │
  └─────────────────────────────────────────────────────────┘
```

---

## Properties at a Glance

| Property | Value |
|----------|-------|
| **Type** | Comparison-based |
| **In-place** | Yes — O(1) extra space |
| **Stable** | Yes — equal elements maintain order |
| **Adaptive** | Yes (with optimization) |
| **Online** | No |

---

## Why Is Bubble Sort Stable?

```
  When A[j] == A[j+1], we do NOT swap.
  
  Example: [3a, 3b, 1]    (3a and 3b are both value 3)
  
  Compare 3a and 3b: 3a == 3b → NO swap → 3a stays before 3b
  Compare 3b and 1:  3b > 1  → SWAP
  
  Result: [3a, 1, 3b]  then  [1, 3a, 3b]
  
  3a is still before 3b → STABLE! ✓
```

---

## Analogy

```
  Think of Bubble Sort like sorting playing cards 
  by repeatedly comparing neighboring cards:
  
  ┌───┐┌───┐┌───┐┌───┐┌───┐
  │ 5 ││ 3 ││ 8 ││ 1 ││ 2 │
  └───┘└───┘└───┘└───┘└───┘
    ↕    ←── compare and swap neighbors
  ┌───┐┌───┐┌───┐┌───┐┌───┐
  │ 3 ││ 5 ││ 8 ││ 1 ││ 2 │
  └───┘└───┘└───┘└───┘└───┘
              ↕    ←── 8 keeps moving right (bubbling up)
  ...
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core operation | Compare and swap adjacent elements |
| Bubbling | Largest unsorted element moves to end each pass |
| Passes needed | At most n-1 |
| Comparisons per pass | Decreases by 1 each pass |
| Total comparisons | n(n-1)/2 |
| Stability | Stable (equal elements not swapped) |

---

## Quick Revision Questions

1. **Why is it called "Bubble" Sort?**
2. **What happens after the first complete pass?**
3. **How many passes does Bubble Sort need for n elements?**
4. **Why is the inner loop limit `n-2-i` instead of `n-2`?**
5. **Is Bubble Sort stable? Explain with an example.**
6. **What is the total number of comparisons in the worst case?**

---

[← Previous Unit: Sorting Lower Bound](../01-Sorting-Fundamentals/06-sorting-lower-bound.md) | [Next: Implementation →](02-implementation.md)
