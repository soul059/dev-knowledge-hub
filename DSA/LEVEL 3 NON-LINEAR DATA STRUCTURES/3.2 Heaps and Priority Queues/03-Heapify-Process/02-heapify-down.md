# 3.2 Heapify Down (Sift Down)

[← Previous: Heapify Up](01-heapify-up.md) | [Next: Sift Up vs Sift Down →](03-sift-up-vs-sift-down.md)

---

## Overview

The **sift down** procedure moves a misplaced element **downward** toward the leaves. It is the workhorse behind extract, build heap, and heap sort.

---

## When Is It Used?

- After **extracting** the root (replacement needs to sink)
- After **decreasing** a key in a max-heap (or increasing in a min-heap)
- During **build heap** (Floyd's algorithm)

---

## The Idea

The element at position `i` might be **too small** (max-heap) or **too large** (min-heap) relative to its children. We move it **downward** by swapping with its largest (max-heap) or smallest (min-heap) child.

```
Before Sift Down:          After Sift Down:
      10  ← too small!         80
     / \                       / \
   80   70                   10   70
  / \                       / \
 50  60                    50  60
                              ↓
                           80
                          / \
                        60   70
                       / \
                      50  10
```

---

## Pseudocode (Max-Heap)

```
SIFT_DOWN(heap, i):
    n = heap.size
    
    WHILE TRUE:
        largest = i
        left  = 2 * i + 1
        right = 2 * i + 2
        
        // Check left child
        IF left < n AND heap[left] > heap[largest]:
            largest = left
        
        // Check right child
        IF right < n AND heap[right] > heap[largest]:
            largest = right
        
        // If largest is not current node, swap and continue
        IF largest != i:
            SWAP(heap[i], heap[largest])
            i = largest
        ELSE:
            BREAK    // Heap property restored
```

---

## Detailed Trace — Sift Down Index 0

```
Array: [15, 80, 70, 50, 60, 65, 30]

Tree:
          15  ← needs to sink
         / \
       80   70
      / \  / \
    50  60 65 30

── Iteration 1 ──
  i = 0
  left = 1 (value 80), right = 2 (value 70)
  
  Compare: 80 > 15 → largest = 1
  Compare: 70 > 15 but 70 < 80 → largest stays 1
  
  largest(1) ≠ i(0) → SWAP(15, 80)

  Array: [80, 15, 70, 50, 60, 65, 30]
          80
         / \
       15   70
      / \  / \
    50  60 65 30

── Iteration 2 ──
  i = 1
  left = 3 (value 50), right = 4 (value 60)
  
  Compare: 50 > 15 → largest = 3
  Compare: 60 > 50 → largest = 4
  
  largest(4) ≠ i(1) → SWAP(15, 60)

  Array: [80, 60, 70, 50, 15, 65, 30]
          80
         / \
       60   70
      / \  / \
    50  15 65 30

── Iteration 3 ──
  i = 4
  left = 9 (out of bounds), right = 10 (out of bounds)
  No children → largest = i → BREAK ✅

Final: [80, 60, 70, 50, 15, 65, 30]
```

---

## Properties of Sift Down

| Property | Value |
|----------|-------|
| Direction | Root → Leaf |
| Max swaps | Height of tree = O(log n) |
| Path | Single path from node to leaf |
| Comparisons per level | 2 (with both children) |
| Restores property | Downward from modified node |

---

## Quick Check

1. **In a max-heap sift down, why must we swap with the LARGER child?**
   <details><summary>Answer</summary>
   If we swap with the smaller child, the other (larger) child would be greater than the new parent, violating the max-heap property.
   </details>

2. **What is the maximum number of comparisons in one call to sift-down for a heap of n elements?**
   <details><summary>Answer</summary>
   At each level, we make 2 comparisons (left child vs current, right child vs current/largest). Maximum levels traversed = ⌊log₂ n⌋. So max comparisons ≈ 2⌊log₂ n⌋.
   </details>

---

[← Previous: Heapify Up](01-heapify-up.md) | [Next: Sift Up vs Sift Down →](03-sift-up-vs-sift-down.md)
