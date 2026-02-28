# 2.2 Extract Min/Max — Bubble Down (Sift Down)

[← Previous: Insert](01-insert-bubble-up.md) | [Next: Peek Operation →](03-peek-operation.md)

---

## Overview

Removing the root (min or max) is the second core heap operation. It uses the **sift down** procedure to restore the heap property.

---

## The Problem

Remove and return the root element (min or max) while maintaining the heap property.

### Why Not Just Remove Root?

If we simply remove the root, we're left with **two separate subtrees** — not a heap.

---

## Strategy

1. **Replace** the root with the **last element** in the array
2. **Remove** the last element (reduce size)
3. **Bubble down** (sift down): Compare the new root with its children, swap with the **larger child** (max-heap) or **smaller child** (min-heap), repeat

---

## Pseudocode (Max-Heap Extract Max)

```
EXTRACT_MAX(heap):
    IF heap is empty:
        ERROR "Heap underflow"
    
    max_value = heap[0]             // Save the root
    heap[0] = heap[heap.size - 1]   // Move last to root
    heap.remove_last()              // Shrink array
    
    SIFT_DOWN(heap, 0)              // Fix heap property
    RETURN max_value

SIFT_DOWN(heap, i):
    n = heap.size
    WHILE TRUE:
        largest = i
        left  = 2*i + 1
        right = 2*i + 2
        
        IF left < n AND heap[left] > heap[largest]:
            largest = left
        IF right < n AND heap[right] > heap[largest]:
            largest = right
        
        IF largest != i:
            SWAP(heap[i], heap[largest])
            i = largest
        ELSE:
            BREAK   // Heap property restored
```

---

## Step-by-Step Trace — Extract Max from Max-Heap

```
Initial Max-Heap:
Array: [90, 85, 70, 80, 60, 65, 30, 50]

              90
             /  \
           85    70
          / \   / \
        80  60 65  30
       /
     50

Step 1: Save max = 90, replace root with last element (50)
Array: [50, 85, 70, 80, 60, 65, 30]

              50  ← doesn't belong here
             /  \
           85    70
          / \   / \
        80  60 65  30

Step 2: Sift down — Compare 50 with children 85 and 70
        largest child = 85 (index 1)
        50 < 85 → SWAP

Array: [85, 50, 70, 80, 60, 65, 30]

              85
             /  \
           50    70
          / \   / \
        80  60 65  30

Step 3: Sift down — Compare 50 with children 80 and 60
        largest child = 80 (index 3)
        50 < 80 → SWAP

Array: [85, 80, 70, 50, 60, 65, 30]

              85
             /  \
           80    70
          / \   / \
        50  60 65  30

Step 4: Sift down — 50 at index 3, children would be at 7, 8
        No children (indices >= array size) → STOP ✅

Final Heap: [85, 80, 70, 50, 60, 65, 30]
Returned: 90
```

---

## Time Complexity

| Case | Complexity | When |
|------|-----------|------|
| Best | O(1) | Last element happens to be valid at root (rare) |
| Worst | O(log n) | Sifts all the way to a leaf |
| Average | O(log n) | |

---

## Quick Check

1. **During extract-max in a max-heap, why do we swap the root with the LAST element rather than simply shifting everything up?**
   <details><summary>Answer</summary>
   Shifting would be O(n) and would break the complete binary tree structure. Swapping with the last element is O(1) and preserves completeness — we only need to fix the heap property via sift-down.
   </details>

2. **When sifting down in a max-heap, why must we swap with the LARGER child?**
   <details><summary>Answer</summary>
   If we swap with the smaller child, the other (larger) child would be greater than the new parent, violating the max-heap property.
   </details>

---

[← Previous: Insert](01-insert-bubble-up.md) | [Next: Peek Operation →](03-peek-operation.md)
