# Chapter 1: Insertion Sort — Algorithm Description

[← Previous Unit: Selection vs Bubble](../03-Selection-Sort/05-comparison-with-bubble.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Insertion Sort builds the sorted array one element at a time by **inserting** each new element into its correct position within the already-sorted portion. It works exactly like how most people sort a hand of playing cards.

---

## Core Idea

```
  Like sorting playing cards in your hand:
  
  Hand:  [3]                    Pick up 3 → only card → sorted
  
  Pick 1: [3, 1]               1 < 3 → insert before 3
          [1, 3]
  
  Pick 5: [1, 3, 5]            5 > 3 → insert at end
          [1, 3, 5]
  
  Pick 2: [1, 3, 5, 2]         2 < 5, 2 < 3, 2 > 1 → insert after 1
          [1, 2, 3, 5]
  
  Pick 4: [1, 2, 3, 5, 4]      4 < 5, 4 > 3 → insert after 3
          [1, 2, 3, 4, 5]
```

---

## Step-by-Step Walkthrough

```
  Array: [7, 3, 5, 1, 9, 2]
  
  ═══ Step 1: key = 3 (index 1) ═══
  Sorted: [7]  |  Insert 3
  
  Compare 3 with 7: 3 < 7 → shift 7 right
  [_, 7, 5, 1, 9, 2]
  
  Insert 3 at position 0:
  [3, 7, 5, 1, 9, 2]
   ✓  ✓
  
  ═══ Step 2: key = 5 (index 2) ═══
  Sorted: [3, 7]  |  Insert 5
  
  Compare 5 with 7: 5 < 7 → shift 7 right
  [3, _, 7, 1, 9, 2]
  
  Compare 5 with 3: 5 > 3 → STOP
  
  Insert 5 at position 1:
  [3, 5, 7, 1, 9, 2]
   ✓  ✓  ✓
  
  ═══ Step 3: key = 1 (index 3) ═══
  Sorted: [3, 5, 7]  |  Insert 1
  
  Compare 1 with 7: 1 < 7 → shift 7 right
  Compare 1 with 5: 1 < 5 → shift 5 right
  Compare 1 with 3: 1 < 3 → shift 3 right
  [_, 3, 5, 7, 9, 2]
  
  Insert 1 at position 0:
  [1, 3, 5, 7, 9, 2]
   ✓  ✓  ✓  ✓
  
  ═══ Step 4: key = 9 (index 4) ═══
  Sorted: [1, 3, 5, 7]  |  Insert 9
  
  Compare 9 with 7: 9 > 7 → STOP (already in place!)
  
  [1, 3, 5, 7, 9, 2]
   ✓  ✓  ✓  ✓  ✓
  
  ═══ Step 5: key = 2 (index 5) ═══
  Sorted: [1, 3, 5, 7, 9]  |  Insert 2
  
  Compare 2 with 9: 2 < 9 → shift 9 right
  Compare 2 with 7: 2 < 7 → shift 7 right
  Compare 2 with 5: 2 < 5 → shift 5 right
  Compare 2 with 3: 2 < 3 → shift 3 right
  Compare 2 with 1: 2 > 1 → STOP
  
  Insert 2 at position 1:
  [1, 2, 3, 5, 7, 9]
   ✓  ✓  ✓  ✓  ✓  ✓   SORTED!
```

---

## Visual Animation

```
  Sorted  |  Unsorted
  ────────┼──────────────────
  [7]     | [3, 5, 1, 9, 2]     key=3: insert into sorted
  [3, 7]  | [5, 1, 9, 2]        key=5: insert into sorted
  [3,5,7] | [1, 9, 2]           key=1: insert into sorted
  [1,3,5,7]| [9, 2]             key=9: already in place!
  [1,3,5,7,9]| [2]              key=2: insert into sorted
  [1,2,3,5,7,9]                  DONE ✓
```

---

## Pseudocode

```
INSERTION-SORT(A, n):
    for i = 1 to n-1:                    // start from second element
        key = A[i]                        // element to insert
        j = i - 1                         // start of sorted portion (right end)
        
        while j >= 0 AND A[j] > key:     // shift larger elements right
            A[j+1] = A[j]                // shift right
            j = j - 1                     // move left
        
        A[j+1] = key                     // insert key at correct position
```

---

## Key Difference: Shifts vs Swaps

```
  BUBBLE SORT uses SWAPS:           INSERTION SORT uses SHIFTS:
  
  [5, 3] → swap → [3, 5]           key = 3
                                    [5, _] → shift 5 right
  Each swap = 3 assignments          [_, 5] → insert 3
  (temp=a, a=b, b=temp)             [3, 5] → 1 shift + 1 insert
  
  For moving element k positions:
  Bubble Sort:  3k assignments (k swaps × 3)
  Insertion Sort: k+2 assignments (k shifts + save + insert)
  
  ┌──────────────────────────────────────────────────────┐
  │  Insertion Sort does ~3× fewer assignments than      │
  │  Bubble Sort for the same data movements!            │
  └──────────────────────────────────────────────────────┘
```

---

## Properties

| Property | Value |
|----------|-------|
| **Type** | Comparison-based |
| **In-place** | Yes — O(1) extra space |
| **Stable** | Yes — equal elements maintain order |
| **Adaptive** | Yes — O(n) for nearly sorted |
| **Online** | Yes — can sort as data arrives |

---

## Why Is Insertion Sort Stable?

```
  When key == A[j], the while loop condition "A[j] > key" is FALSE.
  So equal elements are NOT shifted — key is inserted AFTER equals.
  
  Example: [3a, 3b, 1]   key = 3b (processing index 1)
  Compare 3b with 3a: 3a > 3b? NO (equal, not greater) → STOP
  3b stays after 3a → relative order preserved → STABLE ✓
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core operation | Insert element into correct position in sorted portion |
| Direction | Builds sorted portion from left to right |
| Mechanism | Shift larger elements right, insert key |
| Starting point | Second element (index 1) |
| Shifts vs swaps | Uses shifts — more efficient than swaps |
| Stability | Stable (stops shifting on equal elements) |
| Online | Can process elements as they arrive |

---

## Quick Revision Questions

1. **How does Insertion Sort relate to sorting playing cards?**
2. **Why does the outer loop start at index 1, not 0?**
3. **What is the "key" in Insertion Sort?**
4. **Why are shifts more efficient than swaps?**
5. **Is Insertion Sort stable? Explain why.**
6. **Trace Insertion Sort on [4, 2, 5, 1, 3].**

---

[← Previous Unit: Selection vs Bubble](../03-Selection-Sort/05-comparison-with-bubble.md) | [Next: Implementation →](02-implementation.md)
