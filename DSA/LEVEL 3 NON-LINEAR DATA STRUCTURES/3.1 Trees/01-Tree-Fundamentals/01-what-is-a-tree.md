# 1.1 What is a Tree?

[Back to TOC](../README.md) | [Next: Tree Terminology →](02-tree-terminology.md)

---

## 📖 Definition

A **tree** is a hierarchical, non-linear data structure consisting of **nodes** connected by **edges**. It is a collection of entities called **nodes** linked together to simulate a hierarchy.

```
            A             ← Root node
          / | \
        B   C   D         ← Children of A
       / \     / \
      E   F   G   H       ← Grandchildren of A
         /
        I                  ← Great-grandchild
```

---

## Formal Definition

A tree `T` is a finite set of one or more nodes such that:
1. There is a specially designated node called the **root**.
2. The remaining nodes are partitioned into `n ≥ 0` disjoint sets `T₁, T₂, ..., Tₙ`, where each of these sets is itself a tree. `T₁, T₂, ..., Tₙ` are called **subtrees** of the root.

---

## Key Constraints

| Property | Description |
|----------|-------------|
| **One root** | Exactly one node has no parent |
| **Unique path** | Exactly one path between any two nodes |
| **Acyclic** | No cycles exist |
| **Connected** | All nodes are reachable from root |
| **Edge count** | A tree with `n` nodes has exactly `n - 1` edges |

---

## 💡 Intuition

Think of a tree like a **family tree** or a **file system**:

```
    Computer (root)
    ├── Documents/
    │   ├── Resume.pdf
    │   └── Notes.txt
    ├── Pictures/
    │   ├── Vacation/
    │   │   └── beach.jpg
    │   └── selfie.png
    └── Music/
        └── song.mp3
```

Every folder/file is a node. Every containment relationship is an edge. There's exactly one root (`Computer`), no cycles, and exactly one path from root to any file.

---

## ⏱️ Complexity Overview

| Operation on General Tree | Time Complexity |
|--------------------------|----------------|
| Access any node | O(n) |
| Search | O(n) |
| Insert (at specific position) | O(n) |
| Delete | O(n) |
| Traversal | O(n) |
| Height calculation | O(n) |

**Space Complexity**: O(n) for storing the tree, O(h) for recursive operations (where h = height).

---

[Back to TOC](../README.md) | [Next: Tree Terminology →](02-tree-terminology.md)
