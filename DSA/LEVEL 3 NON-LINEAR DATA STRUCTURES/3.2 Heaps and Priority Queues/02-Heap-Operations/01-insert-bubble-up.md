# 2.1 Insert — Bubble Up (Sift Up)

[← Previous Unit: Heap Fundamentals](../01-Heap-Fundamentals/06-heap-vs-bst.md) | [Next: Extract Min/Max →](02-extract-min-max.md)

---

## Overview

Every heap operation revolves around one core idea: **fix the heap property after making a change**. Insert is the first and most fundamental modification.

---

## The Problem

We want to add a new element while maintaining the heap property.

---

## Strategy

1. **Add** the new element at the **end** of the array (next available position in the complete tree)
2. **Bubble up**: Compare with parent, swap if heap property is violated, repeat

### Why at the End?

Adding at the end preserves the **complete binary tree** structure. We only need to fix the **heap property** — which we do by bubbling up.

---

## Pseudocode (Max-Heap Insert)

```
INSERT(heap, value):
    heap.append(value)              // Add at end
    i = heap.size - 1              // Index of new element
    
    WHILE i > 0:
        parent = (i - 1) / 2
        IF heap[i] > heap[parent]:  // Violates max-heap property
            SWAP(heap[i], heap[parent])
            i = parent              // Move up
        ELSE:
            BREAK                   // Heap property satisfied
```

---

## Step-by-Step Trace — Insert 85 into Max-Heap

```
Initial Max-Heap:
Array: [90, 80, 70, 50, 60, 65, 30]

              90
             /  \
           80    70
          / \   / \
        50  60 65  30

Step 1: Append 85 at index 7
Array: [90, 80, 70, 50, 60, 65, 30, 85]

              90
             /  \
           80    70
          / \   / \
        50  60 65  30
       /
     85

Step 2: Compare 85 with parent at (7-1)/2 = 3 → parent = 50
        85 > 50 → SWAP

Array: [90, 80, 70, 85, 60, 65, 30, 50]

              90
             /  \
           80    70
          / \   / \
        85  60 65  30
       /
     50

Step 3: Compare 85 with parent at (3-1)/2 = 1 → parent = 80
        85 > 80 → SWAP

Array: [90, 85, 70, 80, 60, 65, 30, 50]

              90
             /  \
           85    70
          / \   / \
        80  60 65  30
       /
     50

Step 4: Compare 85 with parent at (1-1)/2 = 0 → parent = 90
        85 < 90 → STOP ✅

Final Heap:
              90
             /  \
           85    70
          / \   / \
        80  60 65  30
       /
     50
```

---

## Time Complexity

| Case | Complexity | When |
|------|-----------|------|
| Best | O(1) | New element belongs at the bottom |
| Worst | O(log n) | New element bubbles all the way to root |
| Average | O(log n) | Bubbles up roughly half the height |

> Height of heap = ⌊log₂ n⌋, so at most ⌊log₂ n⌋ swaps.

---

## Quick Check

1. **Why do we add the new element at the end of the array during insert, and not at the beginning?**
   <details><summary>Answer</summary>
   Adding at the end preserves the complete binary tree structure. Adding at the beginning would shift all elements and destroy the parent-child relationships.
   </details>

---

[← Previous Unit: Heap Fundamentals](../01-Heap-Fundamentals/06-heap-vs-bst.md) | [Next: Extract Min/Max →](02-extract-min-max.md)
