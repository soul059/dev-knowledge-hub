# 1.4 Array Representation

[← Previous: Complete Binary Tree](03-complete-binary-tree.md) | [Next: Parent-Child Indices →](05-parent-child-indices.md)

---

## Overview

This is the **key insight** that makes heaps so efficient. Instead of using pointers (left/right child), we store the heap in a **flat array**.

---

## Mapping: Tree → Array

```
  Tree:                 Array (0-indexed):
  
       50               Index: [0]  [1]  [2]  [3]  [4]  [5]  [6]
      /  \               Value: 50   30   40   10   20   35   25
    30    40
   / \   / \
  10 20 35  25
```

We fill the array **level by level, left to right**:

```
Level 0:  50                          → index 0
Level 1:  30, 40                      → index 1, 2
Level 2:  10, 20, 35, 25             → index 3, 4, 5, 6
```

---

## Why This Works

Because the tree is **complete**, there are **no gaps** in the array. Every index from 0 to n-1 is occupied.

If the tree had a gap (not complete), we'd need a "null" placeholder in the array, wasting space and complicating navigation.

---

## Advantages Over Pointer-Based Trees

| Aspect | Array Heap | Pointer-Based Tree |
|--------|-----------|-------------------|
| **Memory** | Contiguous, no pointer overhead | Extra left/right/parent pointers per node |
| **Cache** | Excellent locality | Poor — nodes scattered in memory |
| **Navigation** | Simple arithmetic | Pointer dereference |
| **Space** | O(n) for n elements | O(n) but with higher constant |

---

## Quick Check

1. **If a max-heap stored in an array has 10 elements, what is the index of the last element?**
   <details><summary>Answer</summary>
   Index 9 (0-indexed). The array uses indices 0 through n-1.
   </details>

2. **What happens if you try to store a non-complete binary tree in an array without sentinel values?**
   <details><summary>Answer</summary>
   The parent-child index formulas break down. A child index might land on an element that doesn't actually correspond to a child in the tree, leading to incorrect behavior.
   </details>

---

[← Previous: Complete Binary Tree](03-complete-binary-tree.md) | [Next: Parent-Child Indices →](05-parent-child-indices.md)
