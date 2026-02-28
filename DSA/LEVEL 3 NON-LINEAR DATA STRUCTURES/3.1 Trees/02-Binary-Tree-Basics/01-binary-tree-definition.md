# 2.1 Binary Tree Definition

[← Previous Unit: Tree Applications](../01-Tree-Fundamentals/06-tree-applications.md) | [Back to TOC](../README.md) | [Next: Types of Binary Trees →](02-types-of-binary-trees.md)

---

## 📖 Definition

A **binary tree** is a tree in which every node has **at most two children**, referred to as the **left child** and the **right child**.

```
    Node Structure:
    
    ┌──────────────────┐
    │       data       │
    ├──────┬───────────┤
    │ left │   right   │
    └──┬───┴─────┬─────┘
       │         │
       ▼         ▼
    Left       Right
    Child      Child
```

---

## Pseudocode: Node Definition

```
CLASS TreeNode:
    data    ← value
    left    ← NULL    // pointer to left child
    right   ← NULL    // pointer to right child
    
    CONSTRUCTOR(value):
        data  = value
        left  = NULL
        right = NULL
```

---

## Example Binary Trees

```
  Valid Binary Trees:
  
  (a)   1       (b)  1       (c)   1       (d) 1      (e) Empty
       / \           /             \                       (NULL)
      2   3         2               3
     / \           /
    4   5         4
    
  Invalid (NOT binary):
  
  (f)     1          A node has 3 children → NOT binary
         /|\
        2 3 4
```

---

## Key Points

- Each node has **at most** 2 children (0, 1, or 2)
- The two children are explicitly distinguished as **left** and **right**
- An empty tree (NULL) is also a valid binary tree
- Binary trees are the foundation for BSTs, heaps, and many advanced structures

---

[← Previous Unit: Tree Applications](../01-Tree-Fundamentals/06-tree-applications.md) | [Back to TOC](../README.md) | [Next: Types of Binary Trees →](02-types-of-binary-trees.md)
