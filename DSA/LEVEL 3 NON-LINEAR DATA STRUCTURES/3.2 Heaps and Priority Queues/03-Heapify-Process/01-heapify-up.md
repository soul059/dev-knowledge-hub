# 3.1 Heapify Up (Sift Up)

[← Previous Unit: Heap Operations](../02-Heap-Operations/06-operations-summary.md) | [Next: Heapify Down →](02-heapify-down.md)

---

## Overview

The **sift up** procedure is the engine behind heap insert. It moves a misplaced element **upward** toward the root until the heap property is restored.

---

## When Is It Used?

- After **inserting** a new element at the end
- After **increasing** a key in a max-heap (or decreasing in a min-heap)

---

## The Idea

The new/modified element might be **too large** (max-heap) or **too small** (min-heap) relative to its parent. We move it **upward** until it finds its correct position.

```
Before Sift Up:            After Sift Up:
      50                        80
     / \                       / \
   40   30                   50   30
  / \                       / \
 20  80  ← too large!     20  40
```

---

## Pseudocode (Max-Heap)

```
SIFT_UP(heap, i):
    WHILE i > 0:
        parent = (i - 1) / 2
        IF heap[i] > heap[parent]:
            SWAP(heap[i], heap[parent])
            i = parent
        ELSE:
            BREAK
```

---

## Detailed Trace — Sift Up Index 6

```
Array: [50, 40, 30, 20, 10, 25, 60]

Tree:
          50
         / \
       40   30
      / \  / \
    20 10 25  60 ← newly inserted

── Iteration 1 ──
  i = 6, parent = (6-1)/2 = 2
  heap[6] = 60, heap[2] = 30
  60 > 30 → SWAP

  Array: [50, 40, 60, 20, 10, 25, 30]
          50
         / \
       40   60
      / \  / \
    20 10 25  30

── Iteration 2 ──
  i = 2, parent = (2-1)/2 = 0
  heap[2] = 60, heap[0] = 50
  60 > 50 → SWAP

  Array: [60, 40, 50, 20, 10, 25, 30]
          60
         / \
       40   50
      / \  / \
    20 10 25  30

── Iteration 3 ──
  i = 0 → i is root, STOP ✅
```

---

## Properties of Sift Up

| Property | Value |
|----------|-------|
| Direction | Leaf → Root |
| Max swaps | Height of tree = O(log n) |
| Path | Single path from node to root |
| Comparisons per level | 1 (only with parent) |
| Restores property | Upward from modified node |

---

## Quick Check

1. **Why does sift up compare with only one node per level, while sift down compares with two?**
   <details><summary>Answer</summary>
   Sift up only compares with the parent (one node above). Sift down must compare with both children to find the correct swap target (the larger child in a max-heap).
   </details>

---

[← Previous Unit: Heap Operations](../02-Heap-Operations/06-operations-summary.md) | [Next: Heapify Down →](02-heapify-down.md)
