# 1.5 Parent-Child Index Relationships

[← Previous: Array Representation](04-array-representation.md) | [Next: Heap vs BST →](06-heap-vs-bst.md)

---

## Overview

Navigating a heap stored in an array requires simple arithmetic formulas to compute parent, left-child, and right-child indices.

---

## 0-Indexed (Most Common)

For a node at index `i`:

| Relationship | Formula | Example (i=1) |
|-------------|---------|----------------|
| **Parent** | `(i - 1) / 2` (integer division) | (1-1)/2 = 0 |
| **Left Child** | `2*i + 1` | 2*1+1 = 3 |
| **Right Child** | `2*i + 2` | 2*1+2 = 4 |

---

## 1-Indexed (Some textbooks)

For a node at index `i`:

| Relationship | Formula | Example (i=2) |
|-------------|---------|----------------|
| **Parent** | `i / 2` (integer division) | 2/2 = 1 |
| **Left Child** | `2*i` | 2*2 = 4 |
| **Right Child** | `2*i + 1` | 2*2+1 = 5 |

---

## Traced Example (0-indexed)

```
Array: [90, 80, 70, 50, 60, 65, 30]
Index:  0   1   2   3   4   5   6

Tree representation:

              90 (i=0)
             /       \
         80 (i=1)    70 (i=2)
         /    \       /    \
     50(i=3) 60(i=4) 65(i=5) 30(i=6)


Verify relationships:
┌──────────┬────────┬──────────────┬──────────────┐
│  Node    │ Index  │ Left Child   │ Right Child  │
├──────────┼────────┼──────────────┼──────────────┤
│  90      │  0     │ 2*0+1=1 (80) │ 2*0+2=2 (70) │
│  80      │  1     │ 2*1+1=3 (50) │ 2*1+2=4 (60) │
│  70      │  2     │ 2*2+1=5 (65) │ 2*2+2=6 (30) │
│  50      │  3     │ 2*3+1=7 (N/A)│ 2*3+2=8 (N/A)│
└──────────┴────────┴──────────────┴──────────────┘

Parent verification:
  Parent of 80 (i=1): (1-1)/2 = 0 → 90 ✅
  Parent of 70 (i=2): (2-1)/2 = 0 → 90 ✅
  Parent of 50 (i=3): (3-1)/2 = 1 → 80 ✅
  Parent of 60 (i=4): (4-1)/2 = 1 → 80 ✅
  Parent of 65 (i=5): (5-1)/2 = 2 → 70 ✅
  Parent of 30 (i=6): (6-1)/2 = 2 → 70 ✅
```

---

## Checking if a Node is a Leaf

A node at index `i` is a **leaf** if it has no children:

```
is_leaf(i) = (2*i + 1) >= n     // left child index >= array size
```

For n=7: Leaves are at indices 3, 4, 5, 6 (where 2*i+1 >= 7)

---

## Last Non-Leaf Node

```
last_non_leaf = (n / 2) - 1     // 0-indexed
```

For n=7: last_non_leaf = 7/2 - 1 = 2

> This is critical for the **Build Heap** algorithm (Unit 3).

---

## Quick Check

1. **Given a node at index 5 (0-indexed), what are the indices of its parent, left child, and right child?**
   <details><summary>Answer</summary>
   Parent = (5-1)/2 = 2. Left child = 2*5+1 = 11. Right child = 2*5+2 = 12.
   </details>

2. **In a heap with 15 elements (0-indexed), which indices are leaves?**
   <details><summary>Answer</summary>
   Leaves are indices where 2*i+1 >= 15, i.e., i >= 7. So leaves are at indices 7, 8, 9, 10, 11, 12, 13, 14.
   </details>

---

[← Previous: Array Representation](04-array-representation.md) | [Next: Heap vs BST →](06-heap-vs-bst.md)
