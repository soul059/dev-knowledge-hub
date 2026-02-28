# 🌳 Trees — Complete Study Guide

> **LEVEL 3: Non-Linear Data Structures — Module 3.1**
>
> A comprehensive study guide covering tree data structures from fundamentals to advanced concepts, with ASCII diagrams, pseudocode, complexity analysis, and problem-solving strategies.

---

## Course Overview

Trees are one of the most important non-linear data structures in computer science. Unlike arrays, linked lists, stacks, and queues (which are linear), trees represent **hierarchical relationships** between data.

This guide takes you from the very basics of what a tree is, through binary trees and binary search trees, all the way to advanced concepts like threaded trees and Huffman coding. Each chapter emphasizes **concepts**, **logic building**, and **problem-solving approaches**.

```
                    TREES — Learning Roadmap
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │   Fundamentals ──► Binary Trees ──► Traversals  │
    │        │                                │       │
    │        ▼                                ▼       │
    │   BST Basics ──► BST Problems    Tree Views     │
    │        │              │               │         │
    │        ▼              ▼               ▼         │
    │   Construction ──► Path Problems ──► Advanced   │
    │                                                 │
    └─────────────────────────────────────────────────┘
```

---

## Complete Table of Contents

### [Unit 1: Tree Fundamentals](01-Tree-Fundamentals/01-tree-fundamentals.md)
| # | Topic |
|---|-------|
| 1.1 | What is a Tree? |
| 1.2 | Tree Terminology (root, parent, child, leaf) |
| 1.3 | Tree Properties (height, depth, level) |
| 1.4 | Tree vs Graph |
| 1.5 | Types of Trees Overview |
| 1.6 | Tree Applications |

### [Unit 2: Binary Tree Basics](02-Binary-Tree-Basics/01-binary-tree-basics.md)
| # | Topic |
|---|-------|
| 2.1 | Binary Tree Definition |
| 2.2 | Types (Full, Complete, Perfect, Balanced) |
| 2.3 | Binary Tree Representation |
| 2.4 | Maximum and Minimum Nodes |
| 2.5 | Properties and Theorems |
| 2.6 | Height Calculation |

### [Unit 3: Tree Traversals](03-Tree-Traversals/01-tree-traversals.md)
| # | Topic |
|---|-------|
| 3.1 | Inorder Traversal (Recursive, Iterative) |
| 3.2 | Preorder Traversal (Recursive, Iterative) |
| 3.3 | Postorder Traversal (Recursive, Iterative) |
| 3.4 | Level Order Traversal (BFS) |
| 3.5 | Morris Traversal |
| 3.6 | Vertical / Diagonal Traversal |

### [Unit 4: Binary Search Tree (BST)](04-Binary-Search-Tree/01-binary-search-tree.md)
| # | Topic |
|---|-------|
| 4.1 | BST Property |
| 4.2 | Search Operation |
| 4.3 | Insertion Operation |
| 4.4 | Deletion Operation |
| 4.5 | BST Validation |
| 4.6 | Inorder Successor / Predecessor |

### [Unit 5: BST Problems](05-BST-Problems/01-bst-problems.md)
| # | Topic |
|---|-------|
| 5.1 | Sorted Array to BST |
| 5.2 | BST to Sorted Array |
| 5.3 | Kth Smallest Element |
| 5.4 | LCA in BST |
| 5.5 | Floor and Ceiling |
| 5.6 | BST Iterator |

### [Unit 6: Tree Construction](06-Tree-Construction/01-tree-construction.md)
| # | Topic |
|---|-------|
| 6.1 | Build from Inorder + Preorder |
| 6.2 | Build from Inorder + Postorder |
| 6.3 | Build from Level Order |
| 6.4 | Serialize and Deserialize |
| 6.5 | Clone a Binary Tree |
| 6.6 | Convert Tree Types |

### [Unit 7: Binary Tree Problems](07-Binary-Tree-Problems/01-binary-tree-problems.md)
| # | Topic |
|---|-------|
| 7.1 | Maximum Depth |
| 7.2 | Minimum Depth |
| 7.3 | Diameter of Tree |
| 7.4 | Check Balanced Tree |
| 7.5 | Same Tree / Subtree |
| 7.6 | Symmetric Tree |

### [Unit 8: Path Problems](08-Path-Problems/01-path-problems.md)
| # | Topic |
|---|-------|
| 8.1 | Root to Leaf Paths |
| 8.2 | Path Sum Problems |
| 8.3 | Maximum Path Sum |
| 8.4 | LCA in Binary Tree |
| 8.5 | Distance Between Nodes |
| 8.6 | Nodes at Distance K |

### [Unit 9: Tree Views](09-Tree-Views/01-tree-views.md)
| # | Topic |
|---|-------|
| 9.1 | Left View |
| 9.2 | Right View |
| 9.3 | Top View |
| 9.4 | Bottom View |
| 9.5 | Boundary Traversal |
| 9.6 | Zigzag Traversal |

### [Unit 10: Advanced Tree Concepts](10-Advanced-Tree-Concepts/01-advanced-tree-concepts.md)
| # | Topic |
|---|-------|
| 10.1 | Threaded Binary Tree |
| 10.2 | Expression Tree |
| 10.3 | Huffman Tree (Introduction) |
| 10.4 | Binary Tree to DLL |
| 10.5 | Flatten Tree to Linked List |
| 10.6 | Vertical Sum |

---

## How to Use This Guide

| Symbol | Meaning |
|--------|---------|
| 📖 | Concept explanation |
| 🔍 | Step-by-step trace |
| ⏱️ | Complexity analysis |
| 💡 | Key insight or trick |
| ⚠️ | Common pitfall |
| 🧩 | Practice problem type |
| 📝 | Summary |

---

## Quick Reference: Complexity Cheat Sheet

| Operation | BST (avg) | BST (worst) | Balanced BST |
|-----------|-----------|-------------|--------------|
| Search    | O(log n)  | O(n)        | O(log n)     |
| Insert    | O(log n)  | O(n)        | O(log n)     |
| Delete    | O(log n)  | O(n)        | O(log n)     |
| Traversal | O(n)      | O(n)        | O(n)         |
| Min/Max   | O(log n)  | O(n)        | O(log n)     |

---

## Prerequisites

- Arrays and Linked Lists
- Recursion fundamentals
- Stack and Queue basics
- Big-O notation understanding

---

> **Start here →** [Unit 1: Tree Fundamentals](01-Tree-Fundamentals/01-tree-fundamentals.md)
