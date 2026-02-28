# 1.5 Types of Trees Overview

[← Previous: Tree vs Graph](04-tree-vs-graph.md) | [Back to TOC](../README.md) | [Next: Tree Applications →](06-tree-applications.md)

---

## 📖 Tree Family

```
                            Trees
                              │
            ┌────────┬────────┼────────┬──────────┐
            │        │        │        │          │
        General   Binary    N-ary   Trie    Specialized
        Tree      Tree      Tree            Trees
                    │
        ┌───────┬───┴───┬────────┐
        │       │       │        │
       BST   AVL    Red-Black  Heap
        │     Tree    Tree     (Tree form)
        │
    ┌───┴───┐
    │       │
  B-Tree  Splay
          Tree
```

---

## Quick Reference

| Tree Type | Key Property | Use Case |
|-----------|-------------|----------|
| **Binary Tree** | Each node has at most 2 children | Foundation for many tree types |
| **BST** | Left < Root < Right | Searching, sorting |
| **AVL Tree** | Self-balancing BST (height diff ≤ 1) | Databases, lookups |
| **Red-Black Tree** | Self-balancing with color properties | Language libraries (maps, sets) |
| **B-Tree** | Multi-way search tree, disk-optimized | File systems, databases |
| **Trie** | Character-by-character storage | Autocomplete, spell check |
| **Heap** | Parent ≥ children (max) or ≤ (min) | Priority queues |
| **Segment Tree** | Range query support | Competitive programming |
| **Fenwick Tree** | Prefix sums with updates | Range sum queries |
| **Suffix Tree** | All suffixes of a string | String matching |

---

## Binary Tree Subtypes (Preview)

```
  Full Binary Tree       Complete Binary Tree     Perfect Binary Tree
                          
       1                       1                        1
      / \                     / \                      / \
     2   3                   2   3                    2   3
    / \                     / \  /                   / \ / \
   4   5                   4  5 6                   4  5 6  7

  Every node has          All levels full           All internal nodes
  0 or 2 children         except possibly last,     have 2 children,
                          filled left to right       all leaves same level
```

---

[← Previous: Tree vs Graph](04-tree-vs-graph.md) | [Back to TOC](../README.md) | [Next: Tree Applications →](06-tree-applications.md)
