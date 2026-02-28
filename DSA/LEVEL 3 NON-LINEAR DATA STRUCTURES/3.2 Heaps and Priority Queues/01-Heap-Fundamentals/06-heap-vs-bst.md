# 1.6 Heap vs Binary Search Tree (BST)

[← Previous: Parent-Child Indices](05-parent-child-indices.md) | [Next Unit: Heap Operations →](../02-Heap-Operations/01-insert-bubble-up.md)

---

## Overview

This is one of the most common confusions. A heap and a BST are both tree-based structures, but they serve very different purposes.

---

## Visual Comparison

```
     Max-Heap:                  BST:
     
        90                       50
       /  \                     /  \
     80    70                 30    70
    / \   /                  / \   / \
   50 60 65                20  40 60  90

 "Parent ≥ Children"      "Left < Parent < Right"
```

---

## Detailed Comparison

| Feature | Heap | BST |
|---------|------|-----|
| **Ordering** | Parent vs children only | Left < Parent < Right (global order) |
| **Structure** | Complete binary tree | Any shape (can be skewed) |
| **Storage** | Array | Linked nodes (typically) |
| **Find Min/Max** | O(1) — root | O(log n) or O(n) if unbalanced |
| **Search arbitrary** | O(n) — must scan | O(log n) if balanced |
| **Insert** | O(log n) | O(log n) if balanced |
| **Delete Min/Max** | O(log n) | O(log n) if balanced |
| **In-order traversal** | NOT sorted | Gives sorted order |
| **Use case** | Priority queue, sorting | Searching, ordered data |

---

## When to Use Which?

| Scenario | Use |
|----------|-----|
| Need the min or max quickly | **Heap** |
| Need to search for specific values | **BST** |
| Need sorted traversal | **BST** |
| Need efficient priority scheduling | **Heap** |
| Building a sorting algorithm | **Heap** (HeapSort) |
| Need both insert and find-min fast | **Heap** |

---

## Key Insight

> A heap gives you **partial ordering** (just enough to find min/max fast).
> A BST gives you **total ordering** (everything is sorted).

If you only need the extremum (min or max), a heap is simpler and faster. If you need to answer queries like "is value X present?", use a BST.

---

## Unit 1 Summary Table

| Concept | Key Point |
|---------|-----------|
| Heap | A complete binary tree satisfying heap property |
| Max-Heap | Parent ≥ Children; root = maximum |
| Min-Heap | Parent ≤ Children; root = minimum |
| Complete Binary Tree | All levels full except last (filled left to right) |
| Array Storage | Level-order mapping; no pointers needed |
| Parent Index | `(i - 1) / 2` (0-indexed) |
| Left Child | `2*i + 1` (0-indexed) |
| Right Child | `2*i + 2` (0-indexed) |
| Height | ⌊log₂ n⌋ for n elements |
| Heap vs BST | Heap = partial order (fast min/max); BST = total order (fast search) |

---

## Quick Check

1. **You need to find both the minimum AND maximum efficiently. Should you use a heap or BST?**
   <details><summary>Answer</summary>
   A single standard heap only gives fast access to one extremum. You could use two heaps (min + max), a BST, or a specialized double-ended priority queue / interval heap.
   </details>

2. **Can a heap give you a sorted traversal of all elements?**
   <details><summary>Answer</summary>
   Not directly. In-order traversal of a heap does NOT yield sorted order. You'd need to repeatedly extract the root (heap sort) to get sorted output — which takes O(n log n).
   </details>

---

[← Previous: Parent-Child Indices](05-parent-child-indices.md) | [Next Unit: Heap Operations →](../02-Heap-Operations/01-insert-bubble-up.md)
