# Unit 7: Heap Sort — The Heapify Process

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)

---

## Overview

**Heapify** (also called sift-down) is the core operation of Heap Sort. It takes a node that may violate the heap property and "sinks" it down to its correct position. Understanding heapify deeply is essential for understanding both heap construction and heap sort.

---

## What Heapify Does

```
  Given a node at index i whose SUBTREES are already valid heaps,
  but the node itself may violate the heap property:
  
  BEFORE heapify(i=0):           AFTER heapify(i=0):
  
        4  ← violates!                10
       / \                            / \
     10    8                         9    8
    / \   / \                       / \  / \
   9   5 7   6                     4   5 7  6
  
  4 is smaller than its children → sinks down
  4 swaps with 10 (largest child)
  4 swaps with 9 (largest child at new position)
  4 reaches a leaf → done
```

---

## Sift-Down Step by Step

```
  Array: [4, 10, 8, 9, 5, 7, 6]     heapSize=7, i=0
  
  Step 1: Compare node 4 with children
    Node: arr[0] = 4
    Left:  arr[1] = 10
    Right: arr[2] = 8
    Largest = 10 (index 1)
    4 ≠ 10 → SWAP arr[0] ↔ arr[1]
    Array: [10, 4, 8, 9, 5, 7, 6]
    
         10
        /  \
       4    8       ← 4 sank one level
      / \  / \
     9  5 7   6
  
  Step 2: Continue at new position (index 1)
    Node: arr[1] = 4
    Left:  arr[3] = 9
    Right: arr[4] = 5
    Largest = 9 (index 3)
    4 ≠ 9 → SWAP arr[1] ↔ arr[3]
    Array: [10, 9, 8, 4, 5, 7, 6]
    
         10
        /  \
       9    8
      / \  / \
     4  5 7   6    ← 4 sank to leaf level
  
  Step 3: Continue at index 3
    Node: arr[3] = 4
    Left:  arr[7] = doesn't exist (7 ≥ heapSize)
    No children → 4 is at a leaf → STOP
  
  Final: [10, 9, 8, 4, 5, 7, 6]  ← Valid max-heap ✓
```

---

## Heapify Code

### Java

```java
// Sift-down heapify
private static void heapify(int[] arr, int heapSize, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    // Check if left child is larger
    if (left < heapSize && arr[left] > arr[largest]) {
        largest = left;
    }
    
    // Check if right child is larger
    if (right < heapSize && arr[right] > arr[largest]) {
        largest = right;
    }
    
    // If largest is not the current node, swap and recurse
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        
        heapify(arr, heapSize, largest);  // Recurse down
    }
}
```

### Iterative Version (No Recursion)

```java
private static void heapify(int[] arr, int heapSize, int i) {
    while (true) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left < heapSize && arr[left] > arr[largest])
            largest = left;
        if (right < heapSize && arr[right] > arr[largest])
            largest = right;
        
        if (largest == i) break;  // Heap property satisfied
        
        // Swap
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        
        i = largest;  // Move down
    }
}
```

### Python

```python
def heapify(arr, heap_size, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < heap_size and arr[left] > arr[largest]:
        largest = left
    if right < heap_size and arr[right] > arr[largest]:
        largest = right
    
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, heap_size, largest)
```

---

## Building the Heap: Bottom-Up Heapify

```
  To build a max-heap from an unsorted array,
  heapify all non-leaf nodes from BOTTOM to TOP.
  
  Why bottom-up? Because heapify requires subtrees to
  already be valid heaps. By starting from the bottom,
  we ensure this precondition holds.
  
  Array: [4, 1, 3, 2, 16, 9, 10, 14, 8, 7]
  
  Tree:          4
              /     \
            1         3
          /   \     /   \
        2      16  9     10
       / \    /
     14   8  7
  
  Last non-leaf = parent of last element = (10-1)/2 = 4
  
  Heapify order: index 4, 3, 2, 1, 0
  
  ┌────────────────────────────────────────────────────────┐
  │  Heapify(4): node=16, children=7                      │
  │  16 ≥ 7 ✓ → no change                                │
  │                                                        │
  │  Heapify(3): node=2, children=14,8                    │
  │  14 > 2 → swap    [4, 1, 3, 14, 16, 9, 10, 2, 8, 7] │
  │                                                        │
  │  Heapify(2): node=3, children=9,10                    │
  │  10 > 3 → swap    [4, 1, 10, 14, 16, 9, 3, 2, 8, 7] │
  │                                                        │
  │  Heapify(1): node=1, children=14,16                   │
  │  16 > 1 → swap    [4, 16, 10, 14, 1, 9, 3, 2, 8, 7] │
  │  Continue: 1, children=7                              │
  │  7 > 1 → swap     [4, 16, 10, 14, 7, 9, 3, 2, 8, 1] │
  │                                                        │
  │  Heapify(0): node=4, children=16,10                   │
  │  16 > 4 → swap    [16, 4, 10, 14, 7, 9, 3, 2, 8, 1] │
  │  Continue: 4, children=14,7                           │
  │  14 > 4 → swap    [16, 14, 10, 4, 7, 9, 3, 2, 8, 1] │
  │  Continue: 4, children=2,8                            │
  │  8 > 4 → swap     [16, 14, 10, 8, 7, 9, 3, 2, 4, 1] │
  └────────────────────────────────────────────────────────┘
  
  Final heap:     16
               /      \
             14        10
            /   \     /   \
           8     7   9     3
          / \   /
         2   4 1
  
  Valid max-heap ✓
```

---

## Build-Heap Time Complexity: O(n), Not O(n log n)!

```
  Intuitive (wrong) analysis:
    n/2 nodes × O(log n) heapify each = O(n log n)?
  
  WRONG! Most nodes are near the bottom where heapify is cheap.
  
  ┌──────────────────────────────────────────────────────────┐
  │  Level  │ Nodes  │ Max sift-down │ Work at level       │
  ├─────────┼────────┼───────────────┼─────────────────────┤
  │  0 (root)│  1    │  log n        │  1 × log n          │
  │  1      │  2     │  log n - 1    │  2 × (log n - 1)    │
  │  2      │  4     │  log n - 2    │  4 × (log n - 2)    │
  │  ...    │  ...   │  ...          │  ...                 │
  │  log n-1│  n/2   │  1            │  n/2 × 1            │
  │  log n  │  n/2   │  0 (leaves)   │  0 (skipped)        │
  └─────────┴────────┴───────────────┴─────────────────────┘
  
  Total = Σ(k=0 to log n) ⌈n/2^(k+1)⌉ × k
        = n × Σ(k=0 to ∞) k/2^(k+1)
        = n × 1                    (the series converges to 1)
        = O(n)  ✓
  
  BUILD-HEAP is O(n), not O(n log n)!
  
  Most of the work is at the bottom where:
  - Many nodes but small sift-down distance
  - n/2 leaves need 0 work
  - n/4 nodes need at most 1 swap
  - n/8 nodes need at most 2 swaps
  - etc.
```

---

## Sift-Up vs Sift-Down

```
  SIFT-DOWN (used in Heap Sort):
  - Node moves DOWN by swapping with largest child
  - Used in: build-heap, extract-max
  - O(log n) per call
  
  SIFT-UP (used in Heap Insert):
  - Node moves UP by swapping with parent
  - Used in: inserting new element into heap
  - O(log n) per call
  
  ┌──────────────────────────────────────────────────────┐
  │  For BUILDING a heap:                                │
  │                                                      │
  │  Top-down (sift-up insertion): O(n log n)           │
  │    Insert elements one by one, sift each up         │
  │                                                      │
  │  Bottom-up (sift-down):        O(n) ✓               │
  │    Heapify non-leaf nodes from bottom to top        │
  │                                                      │
  │  Bottom-up is BETTER for building heaps!            │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Operation | Direction | Time | Used In |
|-----------|-----------|------|---------|
| Sift-down (heapify) | Root toward leaves | O(log n) | Build-heap, extract-max |
| Sift-up | Leaf toward root | O(log n) | Insert into heap |
| Build heap (sift-down) | All non-leaves | O(n) | Heap Sort Phase 1 |
| Build heap (sift-up) | Insert one by one | O(n log n) | Priority queue construction |

---

## Quick Revision Questions

1. **What precondition must hold for heapify to work correctly?**
2. **Why do we heapify from the last non-leaf upward?**
3. **What is the index of the last non-leaf node for array of size n?**
4. **Prove that build-heap is O(n), not O(n log n).**
5. **What's the difference between sift-up and sift-down?**
6. **Trace heapify on arr=[1, 14, 10, 8, 7] at index 0.**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)
