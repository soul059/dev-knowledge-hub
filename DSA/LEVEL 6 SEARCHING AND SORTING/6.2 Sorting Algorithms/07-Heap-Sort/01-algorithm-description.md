# Unit 7: Heap Sort — Algorithm Description

[← Previous Unit: Quick vs Merge Sort](../06-Quick-Sort/06-quicksort-vs-mergesort.md) | [Next: Heapify Process →](02-heapify.md)

---

## Overview

Heap Sort leverages the **binary heap** data structure to sort in O(n log n) time with **O(1) extra space**. It's the only comparison-based sort that is simultaneously O(n log n) worst case AND in-place. It combines the best of Merge Sort (guaranteed O(n log n)) and Quick Sort (in-place).

---

## What Is a Binary Heap?

```
  A Binary Heap is a COMPLETE BINARY TREE that satisfies
  the HEAP PROPERTY.
  
  MAX-HEAP: Every parent ≥ its children
  
              90
            /    \
          80      70
         / \     / \
       50   60  30  40
      / \
    20  10
  
  90 ≥ 80 ✓   90 ≥ 70 ✓
  80 ≥ 50 ✓   80 ≥ 60 ✓
  70 ≥ 30 ✓   70 ≥ 40 ✓
  50 ≥ 20 ✓   50 ≥ 10 ✓
  
  MIN-HEAP: Every parent ≤ its children (used for ascending sort)
  
              10
            /    \
          20      30
         / \     / \
       50   40  70  60
  
  For HEAP SORT, we use a MAX-HEAP to sort in ASCENDING order.
```

---

## Array Representation of Heap

```
  A complete binary tree maps perfectly to an array!
  
  Index:    0    1    2    3    4    5    6    7    8
  Array:  [90,  80,  70,  50,  60,  30,  40,  20,  10]
  
  Tree:              90 (index 0)
                   /    \
               80 (1)    70 (2)
              / \        / \
          50 (3) 60(4) 30(5) 40(6)
          / \
      20(7)  10(8)
  
  ┌──────────────────────────────────────────────────────┐
  │  For node at index i (0-indexed):                    │
  │                                                      │
  │  Parent:       (i - 1) / 2                           │
  │  Left child:   2*i + 1                               │
  │  Right child:  2*i + 2                               │
  │                                                      │
  │  Example: node at index 3 (value 50)                │
  │  Parent: (3-1)/2 = 1 → 80  ✓ (80 ≥ 50)            │
  │  Left:  2*3+1 = 7 → 20                              │
  │  Right: 2*3+2 = 8 → 10                              │
  └──────────────────────────────────────────────────────┘
```

---

## How Heap Sort Works

```
  TWO PHASES:
  
  Phase 1: BUILD MAX-HEAP from unsorted array
  Phase 2: Repeatedly EXTRACT MAX and place at end
  
  ┌───────────────────────────────────────────────────────┐
  │  Phase 1: Build Max-Heap                             │
  │                                                       │
  │  Unsorted: [4, 10, 3, 5, 1]                         │
  │                                                       │
  │  After heapify: [10, 5, 3, 4, 1]                    │
  │                                                       │
  │        10                                             │
  │       /  \                                            │
  │      5    3                                           │
  │     / \                                               │
  │    4   1                                              │
  ├───────────────────────────────────────────────────────┤
  │  Phase 2: Extract-Sort                               │
  │                                                       │
  │  Step 1: Swap root(10) with last(1)                  │
  │  [1, 5, 3, 4, | 10]   Heap size reduced by 1        │
  │  Heapify root: [5, 4, 3, 1, | 10]                   │
  │                                                       │
  │  Step 2: Swap root(5) with last in heap(1)           │
  │  [1, 4, 3, | 5, 10]                                 │
  │  Heapify root: [4, 1, 3, | 5, 10]                   │
  │                                                       │
  │  Step 3: Swap root(4) with last(3)                   │
  │  [3, 1, | 4, 5, 10]                                 │
  │  Heapify root: [3, 1, | 4, 5, 10]                   │
  │                                                       │
  │  Step 4: Swap root(3) with last(1)                   │
  │  [1, | 3, 4, 5, 10]                                 │
  │  Already done!                                        │
  │                                                       │
  │  Result: [1, 3, 4, 5, 10]  ✓ SORTED!               │
  └───────────────────────────────────────────────────────┘
```

---

## Visual Trace

```
  Array: [4, 10, 3, 5, 1]
  
  === Phase 1: Build Max-Heap ===
  
  Start:         4              After heapify:    10
                / \                              /  \
              10    3                            5    3
             / \                               / \
            5   1                              4   1
  
  Heapify from last non-leaf (index 1) up to root (index 0):
  
  heapify(1): node=10, children=5,1  → 10 ≥ both ✓ (no swap)
  heapify(0): node=4, children=10,3  → 10 > 4 → swap 4↔10
              [10, 4, 3, 5, 1]
              Then check 4 at index 1: children=5,1 → 5 > 4 → swap
              [10, 5, 3, 4, 1]  ← MAX-HEAP ✓
  
  === Phase 2: Extract-Sort ===
  
  Iteration 1:
  Swap 10↔1:  [1, 5, 3, 4, |10]     10 is sorted
  Heapify(0): [5, 4, 3, 1, |10]
  
         5                     sorted: [10]
        / \
       4   3
      /
     1
  
  Iteration 2:
  Swap 5↔1:   [1, 4, 3, |5, 10]     5 is sorted
  Heapify(0): [4, 1, 3, |5, 10]
  
       4                       sorted: [5, 10]
      / \
     1   3
  
  Iteration 3:
  Swap 4↔3:   [3, 1, |4, 5, 10]     4 is sorted
  Heapify(0): [3, 1, |4, 5, 10]
  
     3                         sorted: [4, 5, 10]
    /
   1
  
  Iteration 4:
  Swap 3↔1:   [1, |3, 4, 5, 10]     3 is sorted
  
  Result: [1, 3, 4, 5, 10]  ✓ SORTED!
```

---

## Properties of Heap Sort

```
  ┌────────────────────────────────────────────────────┐
  │  Property           │  Value                       │
  ├─────────────────────┼──────────────────────────────┤
  │  Time (Best)        │  O(n log n)                  │
  │  Time (Average)     │  O(n log n)                  │
  │  Time (Worst)       │  O(n log n)  ✓              │
  │  Space              │  O(1)  ✓ truly in-place     │
  │  Stable             │  NO ✗                        │
  │  In-place           │  YES ✓                       │
  │  Adaptive           │  NO                          │
  │  Comparison-based   │  YES                         │
  │  Cache performance  │  POOR (jumps around array)   │
  └─────────────────────┴──────────────────────────────┘
  
  Heap Sort = O(n log n) guaranteed + O(1) space
  The ONLY sort with both properties!
```

---

## Heap Sort vs Other O(n log n) Sorts

```
  ┌──────────────┬──────────────┬──────────┬────────┐
  │              │ Worst Case   │ Space    │ Stable │
  ├──────────────┼──────────────┼──────────┼────────┤
  │ Quick Sort   │ O(n²)  ✗    │ O(log n) │ No     │
  │ Merge Sort   │ O(n log n) ✓│ O(n) ✗  │ Yes ✓  │
  │ HEAP SORT    │ O(n log n) ✓│ O(1) ✓  │ No     │
  └──────────────┴──────────────┴──────────┴────────┘
  
  Heap Sort fills a unique niche:
  When you need O(n log n) GUARANTEED with NO extra memory.
```

---

## Pseudocode

```
  HEAP-SORT(A, n)
  ──────────────
  // Phase 1: Build max-heap
  for i ← ⌊n/2⌋ - 1 downto 0
      HEAPIFY(A, n, i)
  
  // Phase 2: Extract elements
  for i ← n-1 downto 1
      swap A[0] and A[i]      // Move max to sorted region
      HEAPIFY(A, i, 0)         // Fix heap of size i
  
  
  HEAPIFY(A, heapSize, i)
  ───────────────────────
  largest ← i
  left ← 2*i + 1
  right ← 2*i + 2
  
  if left < heapSize AND A[left] > A[largest]
      largest ← left
  if right < heapSize AND A[right] > A[largest]
      largest ← right
  
  if largest ≠ i
      swap A[i] and A[largest]
      HEAPIFY(A, heapSize, largest)    // Recurse down
```

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| Data structure | Binary max-heap in array |
| Phase 1 | Build max-heap: O(n) |
| Phase 2 | Extract-sort: n × O(log n) = O(n log n) |
| Total time | O(n log n) all cases |
| Space | O(1) — truly in-place |
| Key operation | Heapify (sift-down) |
| Unique advantage | Only O(n log n) + O(1) space sort |
| Main weakness | Poor cache performance |

---

## Quick Revision Questions

1. **What is the heap property for a max-heap?**
2. **Given index i, what are the formulas for parent, left, right?**
3. **Why does Heap Sort use a MAX-heap for ASCENDING order?**
4. **What are the two phases of Heap Sort?**
5. **Why is Heap Sort not stable? Give an example.**
6. **What unique combination of properties does Heap Sort offer?**

---

[← Previous Unit: Quick vs Merge Sort](../06-Quick-Sort/06-quicksort-vs-mergesort.md) | [Next: Heapify Process →](02-heapify.md)
