# Chapter 1: Selection Sort — Algorithm Description

[← Previous Unit: Bubble Sort Variations](../02-Bubble-Sort/06-variations.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Selection Sort works by repeatedly **finding the minimum element** from the unsorted portion and placing it at the beginning. Unlike Bubble Sort which moves elements one position at a time, Selection Sort directly selects the correct element for each position.

---

## Core Idea

```
  Divide array into two parts:
  
  ┌──────────────────────┬──────────────────────┐
  │   SORTED portion     │   UNSORTED portion    │
  │   (grows →)          │   (← shrinks)         │
  └──────────────────────┴──────────────────────┘
  
  In each pass:
  1. Find the MINIMUM element in the unsorted portion
  2. SWAP it with the first element of the unsorted portion
  3. The sorted portion grows by 1
```

---

## Step-by-Step Walkthrough

```
  Array: [29, 10, 14, 37, 13]
  
  ═══ Pass 0 ═══
  Find minimum in [29, 10, 14, 37, 13] → min = 10 at index 1
  Swap arr[0] and arr[1]:
  
  [29, 10, 14, 37, 13]
   ↑   ↑
   swap these
   
  [10, 29, 14, 37, 13]
   ✓ ──────────────────  10 is in its final position
  
  ═══ Pass 1 ═══
  Find minimum in [29, 14, 37, 13] → min = 13 at index 4
  Swap arr[1] and arr[4]:
  
  [10, 29, 14, 37, 13]
       ↑            ↑
       swap these
  
  [10, 13, 14, 37, 29]
   ✓   ✓ ─────────────  13 is in its final position
  
  ═══ Pass 2 ═══
  Find minimum in [14, 37, 29] → min = 14 at index 2
  Swap arr[2] and arr[2] (same position — no actual swap):
  
  [10, 13, 14, 37, 29]
   ✓   ✓   ✓ ─────────  14 is already correct
  
  ═══ Pass 3 ═══
  Find minimum in [37, 29] → min = 29 at index 4
  Swap arr[3] and arr[4]:
  
  [10, 13, 14, 37, 29]
                ↑   ↑
                swap
  
  [10, 13, 14, 29, 37]
   ✓   ✓   ✓   ✓   ✓   ALL SORTED!
```

---

## Visual Animation

```
  [29, 10, 14, 37, 13]     Initial
   │                │
   └─── scan all ───┘   Find min = 10
  
  [10 | 29, 14, 37, 13]    After pass 0
        │            │
        └── scan ────┘   Find min = 13
  
  [10, 13 | 14, 37, 29]    After pass 1
              │       │
              └ scan ─┘   Find min = 14
  
  [10, 13, 14 | 37, 29]    After pass 2
                  │   │
                  └─┘    Find min = 29
  
  [10, 13, 14, 29 | 37]    After pass 3
  
  [10, 13, 14, 29, 37]     SORTED ✓
  
  | = boundary between sorted and unsorted portions
```

---

## Pseudocode

```
SELECTION-SORT(A, n):
    for i = 0 to n-2:                     // for each position
        minIndex = i                       // assume current is minimum
        
        for j = i+1 to n-1:               // scan unsorted portion
            if A[j] < A[minIndex]:         // found new minimum?
                minIndex = j               // update minimum index
        
        if minIndex != i:                  // if min is not at position i
            swap(A[i], A[minIndex])        // place min at position i
```

---

## Key Observations

```
  ┌───────────────────────────────────────────────────────────┐
  │  1. Makes exactly n-1 passes (one less than n)           │
  │                                                           │
  │  2. Each pass makes exactly ONE swap (at most)           │
  │                                                           │
  │  3. Makes the same number of comparisons regardless      │
  │     of input — NOT adaptive!                             │
  │                                                           │
  │  4. After pass i, the first i+1 elements are sorted     │
  │     AND in their final positions                         │
  │                                                           │
  │  5. Total comparisons: n(n-1)/2 always                   │
  │     Total swaps: at most n-1                             │
  └───────────────────────────────────────────────────────────┘
```

---

## Properties

| Property | Value |
|----------|-------|
| **Type** | Comparison-based |
| **In-place** | Yes — O(1) extra space |
| **Stable** | No (default implementation) |
| **Adaptive** | No — always O(n²) |
| **Online** | No |
| **Swaps** | O(n) — minimum among O(n²) sorts |

---

## Analogy

```
  Selection Sort is like selecting the shortest person
  from a line and placing them at the front:
  
  Unsorted line:    😀 😊 😎 🙂 😄
                   (5) (3) (4) (1) (2)
  
  Scan all → Find shortest (1) → place at front:
  
  Position 0:  🙂 😊 😎 😀 😄
               (1)
  
  Scan remaining → Find next shortest (2):
  
  Position 1:  🙂 😄 😎 😀 😊
               (1)(2)
  
  Continue until everyone is in order...
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core operation | Find minimum, swap to front |
| Sorted portion | Grows from left to right |
| Passes | Exactly n-1 |
| Swaps per pass | Exactly 1 (or 0 if already in place) |
| Total swaps | At most n-1 (excellent!) |
| Total comparisons | n(n-1)/2 always |
| Adaptive | No — same work for any input |

---

## Quick Revision Questions

1. **Describe the core idea of Selection Sort in one sentence.**
2. **How many swaps does Selection Sort make in the worst case?**
3. **Why is Selection Sort NOT adaptive?**
4. **After pass i, what can you guarantee about the array?**
5. **Trace Selection Sort on [4, 2, 1, 5, 3].**
6. **What advantage does Selection Sort's low swap count provide?**

---

[← Previous Unit: Bubble Sort Variations](../02-Bubble-Sort/06-variations.md) | [Next: Implementation →](02-implementation.md)
