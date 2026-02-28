# 2.3 Binary Tree Representation

[← Previous: Types of Binary Trees](02-types-of-binary-trees.md) | [Back to TOC](../README.md) | [Next: Maximum and Minimum Nodes →](04-max-min-nodes.md)

---

## Method 1: Linked Representation (Node-based)

Each node stores data and two pointers.

```
    Tree:       1
               / \
              2   3
             /
            4

    Memory Representation:
    
    ┌───┬───┬───┐     ┌───┬───┬───┐
    │ / │ 1 │ ●─┼────►│ / │ 3 │ / │
    └─┬─┴───┴───┘     └───┴───┴───┘
      │
      ▼
    ┌───┬───┬───┐
    │ ●─┤ 2 │ / │
    └─┬─┴───┴───┘
      │
      ▼
    ┌───┬───┬───┐
    │ / │ 4 │ / │
    └───┴───┴───┘
    
    '/' means NULL pointer
    '●' means valid pointer
```

**Pseudocode**:
```
CLASS TreeNode:
    data
    left  ← NULL
    right ← NULL
```

**Advantages**: Dynamic size, easy insertion/deletion.
**Disadvantages**: Extra memory for pointers, not cache-friendly.

---

## Method 2: Array Representation (Sequential)

Store the tree in an array using **level-order indexing**.

```
    Tree:       A              Index:  0
               / \
              B   C                   1   2
             / \   \
            D   E   F                3  4    5

    Array (0-indexed):
    ┌───┬───┬───┬───┬───┬───┐
    │ A │ B │ C │ D │ E │ / │ F │
    └───┴───┴───┴───┴───┴───┘
      0   1   2   3   4   5   6
```

---

## Index Formulas

| Relationship | 0-indexed | 1-indexed |
|-------------|-----------|-----------|
| Left child of `i` | `2i + 1` | `2i` |
| Right child of `i` | `2i + 2` | `2i + 1` |
| Parent of `i` | `⌊(i-1)/2⌋` | `⌊i/2⌋` |

---

## 🔍 Trace: Array Representation

```
    Tree:       10 (index 0)
               /  \
              20   30 (index 1, 2)
             / \
            40  50 (index 3, 4)

    Array: [10, 20, 30, 40, 50]
    
    Left child of 10 (i=0):  arr[2*0+1] = arr[1] = 20  ✓
    Right child of 10 (i=0): arr[2*0+2] = arr[2] = 30  ✓
    Parent of 40 (i=3):      arr[(3-1)/2] = arr[1] = 20  ✓
    Left child of 30 (i=2):  arr[2*2+1] = arr[5] → doesn't exist ✓ (no child)
```

---

## When to Use What?

| Criteria | Linked | Array |
|----------|--------|-------|
| **Complete binary trees** | Works | **Best** (no wasted space) → Heaps! |
| **Sparse / skewed trees** | **Best** | Too much wasted space |
| **Dynamic insertions/deletions** | **Best** | Costly resizing |
| **Random access by index** | Not efficient | **Best** |
| **Cache performance** | Poor | **Best** (contiguous memory) |

---

[← Previous: Types of Binary Trees](02-types-of-binary-trees.md) | [Back to TOC](../README.md) | [Next: Maximum and Minimum Nodes →](04-max-min-nodes.md)
