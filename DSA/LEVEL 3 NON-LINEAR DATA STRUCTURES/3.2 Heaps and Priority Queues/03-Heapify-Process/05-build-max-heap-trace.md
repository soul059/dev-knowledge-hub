# 3.5 Complete Build Max-Heap Trace

[← Previous: Build Heap Proof](04-build-heap-on-proof.md) | [Next: Edge Cases →](06-edge-cases.md)

---

## Overview

A full step-by-step walkthrough of Floyd's build-heap algorithm on a 9-element array.

---

## Complete Example

```
Input: [3, 9, 2, 1, 4, 5, 7, 6, 8]

Tree (not a heap):
              3
           /     \
         9         2
        / \       / \
       1   4     5   7
      / \
     6   8

n = 9, last non-leaf = 9/2 - 1 = 3

═══ Step 1: i = 3, value = 1 ═══
Children: 6 (i=7), 8 (i=8)
Largest child: 8
1 < 8 → SWAP(1, 8)

              3
           /     \
         9         2
        / \       / \
       8   4     5   7
      / \
     6   1

═══ Step 2: i = 2, value = 2 ═══
Children: 5 (i=5), 7 (i=6)
Largest child: 7
2 < 7 → SWAP(2, 7)

              3
           /     \
         9         7
        / \       / \
       8   4     5   2
      / \
     6   1

═══ Step 3: i = 1, value = 9 ═══
Children: 8 (i=3), 4 (i=4)
Largest child: 8
9 > 8 → No swap needed ✅

              3
           /     \
         9         7
        / \       / \
       8   4     5   2
      / \
     6   1

═══ Step 4: i = 0, value = 3 ═══
Children: 9 (i=1), 7 (i=2)
Largest child: 9
3 < 9 → SWAP(3, 9)

              9
           /     \
         3         7
        / \       / \
       8   4     5   2
      / \
     6   1

Continue sifting 3 at i=1:
Children: 8 (i=3), 4 (i=4)
Largest child: 8
3 < 8 → SWAP(3, 8)

              9
           /     \
         8         7
        / \       / \
       3   4     5   2
      / \
     6   1

Continue sifting 3 at i=3:
Children: 6 (i=7), 1 (i=8)
Largest child: 6
3 < 6 → SWAP(3, 6)

              9
           /     \
         8         7
        / \       / \
       6   4     5   2
      / \
     3   1

3 at i=7 has no children → DONE ✅

═══ FINAL MAX-HEAP ═══

Array: [9, 8, 7, 6, 4, 5, 2, 3, 1]

              9
           /     \
         8         7
        / \       / \
       6   4     5   2
      / \
     3   1

Verify: Every parent ≥ its children ✅
```

---

## Building Min-Heap vs Max-Heap

The only difference is in the comparison:

```
SIFT_DOWN_MIN(heap, i):          SIFT_DOWN_MAX(heap, i):
    ...                              ...
    IF heap[left] < heap[smallest]:  IF heap[left] > heap[largest]:
        smallest = left                  largest = left
    IF heap[right] < heap[smallest]: IF heap[right] > heap[largest]:
        smallest = right                 largest = right
    ...                              ...
```

Same structure, opposite comparisons. Same O(n) complexity.

---

## Quick Check

1. **Draw the max-heap resulting from BUILD_HEAP([1, 5, 3, 7, 2]).**
   <details><summary>Answer</summary>
   Process: Start at i=1 (value 5, children 7,2). 7>5→swap. Then i=0 (value 1, children 7,3). 7>1→swap. Continue sifting 1: children 5,2. 5>1→swap. Result: [7, 5, 3, 1, 2].
   Tree: 7 at root, left=5 right=3, 5's children: 1, 2.
   </details>

---

[← Previous: Build Heap Proof](04-build-heap-on-proof.md) | [Next: Edge Cases →](06-edge-cases.md)
