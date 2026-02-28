# Chapter 3: Stable vs Unstable Sorting

[← Previous: In-place vs Out-of-place](02-inplace-vs-outofplace.md) | [Next: Comparison vs Non-comparison →](04-comparison-vs-noncomparison.md)

---

## Overview

**Stability** is a critical property that determines whether a sorting algorithm preserves the **relative order** of elements with equal keys. This concept is essential when sorting by multiple criteria or when the original ordering carries meaning.

---

## What is Stability?

A sorting algorithm is **stable** if two elements with **equal keys** appear in the **same relative order** in the sorted output as they appeared in the input.

```
  STABLE SORT EXAMPLE:
  
  Input (sort by Age):
  ┌──────────┬─────┐
  │  Name    │ Age │
  ├──────────┼─────┤
  │  Alice   │  25 │  ← appears first among age 25
  │  Bob     │  22 │
  │  Charlie │  25 │  ← appears second among age 25
  │  Diana   │  22 │
  └──────────┴─────┘
  
  STABLE sort result:              UNSTABLE sort result:
  ┌──────────┬─────┐               ┌──────────┬─────┐
  │  Bob     │  22 │               │  Diana   │  22 │  ← order of 22s changed!
  │  Diana   │  22 │  ✓ order      │  Bob     │  22 │
  │  Alice   │  25 │    preserved  │  Charlie │  25 │  ← order of 25s changed!
  │  Charlie │  25 │               │  Alice   │  25 │
  └──────────┴─────┘               └──────────┴─────┘
  
  In stable sort:   Bob before Diana (same as input)
                    Alice before Charlie (same as input)
  
  In unstable sort: Relative order NOT guaranteed for equal elements
```

---

## Why Does Stability Matter?

### 1. Multi-key Sorting

When you need to sort by **multiple criteria**, stability lets you sort by secondary key first, then by primary key:

```
  PROBLEM: Sort students by Grade (primary), then by Name (secondary)
  
  Step 1 — Sort by Name (secondary key):
  ┌──────────┬───────┐
  │  Name    │ Grade │
  ├──────────┼───────┤
  │  Alice   │   B   │
  │  Bob     │   A   │
  │  Charlie │   A   │
  │  Diana   │   B   │
  └──────────┴───────┘
  
  Step 2 — STABLE sort by Grade (primary key):
  ┌──────────┬───────┐
  │  Name    │ Grade │
  ├──────────┼───────┤
  │  Bob     │   A   │  ← Among A's: Bob before Charlie ✓ (preserved from Step 1)
  │  Charlie │   A   │
  │  Alice   │   B   │  ← Among B's: Alice before Diana ✓ (preserved from Step 1)
  │  Diana   │   B   │
  └──────────┴───────┘
  
  Result: Sorted by Grade, and within same Grade, sorted by Name!
  This ONLY works if the second sort is STABLE.
```

### 2. Preserving Meaningful Order

```
  E-commerce: Products sorted by "Date Added", then re-sorted by "Price"
  
  If stable: Products with same price stay in "Date Added" order
  If unstable: Products with same price appear in random order
```

### 3. Database Operations

```
  SQL equivalent:
  SELECT * FROM students ORDER BY grade, name;
  
  This requires stable sorting to produce consistent results.
```

---

## Visualizing Stability with Cards

```
  Sort these cards by VALUE only (ignore suit):
  
  Input:    5♠  3♥  5♥  3♠  5♦
  
  STABLE:   3♥  3♠  5♠  5♥  5♦    (original order of equal values preserved)
            ↑   ↑   ↑   ↑   ↑
            3♥ was before 3♠       5♠ was before 5♥ was before 5♦
  
  UNSTABLE: 3♠  3♥  5♦  5♠  5♥    (equal values may be reordered)
            ↑   ↑   ↑   ↑   ↑
            3♠ and 3♥ swapped      5s completely reordered
```

---

## Which Algorithms Are Stable?

```
  ┌─────────────────────────────────────────────────────────┐
  │              STABILITY CLASSIFICATION                    │
  ├───────────────────────┬─────────────────────────────────┤
  │     STABLE ✓          │     UNSTABLE ✗                  │
  ├───────────────────────┼─────────────────────────────────┤
  │  Bubble Sort          │  Selection Sort                 │
  │  Insertion Sort       │  Quick Sort                     │
  │  Merge Sort           │  Heap Sort                      │
  │  Counting Sort        │                                 │
  │  Radix Sort           │                                 │
  │  Bucket Sort          │                                 │
  │  Tim Sort             │                                 │
  └───────────────────────┴─────────────────────────────────┘
```

---

## Why Are Certain Algorithms Unstable?

### Selection Sort — Unstable

```
  Array: [5a, 5b, 3]      (5a and 5b are both value 5)
  
  Pass 1: Find minimum = 3 (index 2)
          Swap with index 0:
          
  Before swap: [5a, 5b, 3]
                ↑          ↑
                swap these two
                
  After swap:  [3, 5b, 5a]    ← 5a and 5b swapped relative order!
                                  Originally 5a before 5b
                                  Now 5b before 5a → UNSTABLE
```

### Quick Sort — Unstable

```
  Array: [4a, 4b, 3, 4c]    Pivot = 4c
  
  Partition around 4c:
  Elements < 4c go left, elements ≥ 4c go right
  
  Result might be: [3, 4c, 4b, 4a]
                         ↑    ↑   ↑
                    Original order was 4a, 4b, 4c
                    Now: 4c, 4b, 4a → UNSTABLE
```

### Heap Sort — Unstable

```
  Array: [2a, 2b, 1]    Build max-heap → [2a, 2b, 1]
  
  Extract max (2a), swap with last:
  [1, 2b, 2a*]    (* = sorted position)
  
  Heapify: [2b, 1, 2a*]
  Extract max (2b), swap with last:
  [1, 2b*, 2a*]
  
  Result: [1, 2b, 2a]   ← 2b before 2a → UNSTABLE
                           Originally 2a was before 2b
```

---

## Making Unstable Algorithms Stable

You can make any unstable algorithm stable by **augmenting the key** with the original index:

```
  Original:  [5, 3, 5, 2]
  Indices:    0  1  2  3
  
  Augmented keys: [(5,0), (3,1), (5,2), (2,3)]
  
  Sort by (value, original_index):
  Result: [(2,3), (3,1), (5,0), (5,2)]
  
  Since (5,0) < (5,2), the first 5 stays before the second 5
  → Guaranteed stable!
  
  Cost: O(n) extra space for indices
```

### Pseudocode

```
function makeStable(arr):
    augmented = []
    for i = 0 to n-1:
        augmented.append( (arr[i], i) )   // attach original index
    
    sort augmented by (value, index)      // use any sort
    
    for i = 0 to n-1:
        arr[i] = augmented[i].value       // extract sorted values
```

---

## Practical Impact

```
  Scenario: Sort transactions by amount, preserving chronological order
  
  Transactions (in chronological order):
  ┌────┬────────┬─────────┐
  │ ID │ Amount │  Time   │
  ├────┼────────┼─────────┤
  │  1 │  $100  │  9:00   │
  │  2 │  $200  │  9:15   │
  │  3 │  $100  │  9:30   │
  │  4 │  $200  │  9:45   │
  └────┴────────┴─────────┘
  
  STABLE sort by Amount:             UNSTABLE sort by Amount:
  ┌────┬────────┬─────────┐          ┌────┬────────┬─────────┐
  │  1 │  $100  │  9:00   │          │  3 │  $100  │  9:30   │  ← wrong order!
  │  3 │  $100  │  9:30   │          │  1 │  $100  │  9:00   │
  │  2 │  $200  │  9:15   │          │  4 │  $200  │  9:45   │  ← wrong order!
  │  4 │  $200  │  9:45   │          │  2 │  $200  │  9:15   │
  └────┴────────┴─────────┘          └────┴────────┴─────────┘
  
  Stable: ✓ Chronological within same amount
  Unstable: ✗ Chronological order lost
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Stable sort | Preserves relative order of equal elements |
| Unstable sort | May change relative order of equal elements |
| Stable algorithms | Bubble, Insertion, Merge, Counting, Radix |
| Unstable algorithms | Selection, Quick, Heap |
| Multi-key sorting | Requires stability to sort by secondary then primary key |
| Making unstable stable | Augment keys with original indices |
| Cost of stabilization | O(n) extra space for index storage |

---

## Quick Revision Questions

1. **Define stability in the context of sorting algorithms.**
2. **Why is Selection Sort unstable? Give an example.**
3. **How does stability help in multi-key sorting?**
4. **Can an unstable algorithm be made stable? How?**
5. **Name all stable sorting algorithms from this course.**
6. **Why does Quick Sort break stability during partitioning?**

---

[← Previous: In-place vs Out-of-place](02-inplace-vs-outofplace.md) | [Next: Comparison vs Non-comparison →](04-comparison-vs-noncomparison.md)
