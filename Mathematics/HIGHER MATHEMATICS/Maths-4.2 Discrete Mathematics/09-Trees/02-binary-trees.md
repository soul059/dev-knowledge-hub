# Chapter 9.2: Binary Trees

[← Previous: Tree Properties](01-tree-properties.md) | [Back to README](../README.md) | [Next: Tree Traversals →](03-tree-traversals.md)

---

## 📋 Chapter Overview

A **binary tree** is a rooted tree where each node has at most **two children** (left and right). Binary trees are the foundation of search structures, expression parsing, and decision-making algorithms.

---

## 1. Definition

A **binary tree** is either:
- **Empty** ($\emptyset$), or
- A **root** node with a **left subtree** and a **right subtree**, each of which is a binary tree

```
  Binary Tree:
  
         a
        / \
       b   c
      / \   \
     d   e   f
        / \
       g   h
  
  Root: a
  Left subtree of a: rooted at b
  Right subtree of a: rooted at c
```

---

## 2. Types of Binary Trees

| Type | Definition | Example |
|------|------------|---------|
| **Full** (proper) | Every node has 0 or 2 children | No single-child nodes |
| **Complete** | All levels full except possibly last; last level filled left to right | Heap structure |
| **Perfect** | All internal nodes have 2 children; all leaves at same level | Balanced, $2^{h+1}-1$ nodes |
| **Balanced** | Heights of left and right subtrees differ by at most 1 | AVL trees |
| **Degenerate** | Each internal node has exactly 1 child | Essentially a linked list |

```
  Full:           Complete:        Perfect:         Degenerate:
  
     a               a                a                a
    / \             / \              / \                \
   b   c           b   c            b   c               b
  / \             / \ /            / \ / \               \
 d   e           d  e f           d  e f  g               c
                                                           \
                                                            d
```

---

## 3. Binary Tree Properties

For a binary tree with height $h$:

| Property | Minimum Nodes | Maximum Nodes |
|----------|:------------:|:------------:|
| Nodes at level $k$ | 1 | $2^k$ |
| Total nodes | $h + 1$ | $2^{h+1} - 1$ |
| Leaves | 1 | $2^h$ |

### Key Formulas

| Measure | Formula |
|---------|---------|
| Min height for $n$ nodes | $h = \lfloor \log_2 n \rfloor$ |
| Max height for $n$ nodes | $h = n - 1$ (degenerate) |
| Nodes in perfect tree | $n = 2^{h+1} - 1$ |
| Leaves in full tree with $i$ internal nodes | $l = i + 1$ |
| Internal nodes | $i = l - 1$ |

---

## 4. Full Binary Trees: Extended Properties

For a **full** binary tree:

$$\boxed{l = i + 1}$$

where $l$ = leaves, $i$ = internal nodes.

**Proof:** Count edges. Each non-root node has exactly 1 incoming edge. Each internal node has exactly 2 outgoing edges. Total edges = $n - 1 = 2i$. Since $n = i + l$: $i + l - 1 = 2i$, so $l = i + 1$. ∎

---

## 5. Binary Search Trees (BST)

A **BST** stores keys such that:
- Left subtree contains keys **less than** the node's key
- Right subtree contains keys **greater than** the node's key

```
  BST with keys {2, 3, 5, 7, 8, 10, 15}:
  
           8
          / \
         3   10
        / \    \
       2   5   15
          / 
         7
  
  Wait — let me correct: 7 > 5 so 7 goes right of 5.
  
           8
          / \
         3   10
        / \    \
       2   5   15
            \
             7
  
  Searching for 7:
  8 → go left → 3 → go right → 5 → go right → 7 ✓
```

### BST Operations (average case, balanced)

| Operation | Time |
|-----------|:----:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |
| Worst case (degenerate) | $O(n)$ |

---

## 6. Expression Trees

Arithmetic expressions can be represented as binary trees:

```
  Expression: (3 + 4) × (5 - 2)
  
           ×
          / \
         +   -
        / \ / \
       3  4 5  2
  
  • Leaves = operands
  • Internal nodes = operators
  • Left child = left operand
  • Right child = right operand
```

| Traversal | Output | Name |
|-----------|--------|------|
| Inorder | $3 + 4 \times 5 - 2$ | Infix (needs parentheses) |
| Preorder | $\times + 3 \; 4 \; - 5 \; 2$ | Prefix (Polish notation) |
| Postorder | $3 \; 4 \; + \; 5 \; 2 \; - \; \times$ | Postfix (Reverse Polish) |

---

## 7. Counting Binary Trees: Catalan Numbers

The number of **structurally distinct** binary trees with $n$ nodes is the $n$-th **Catalan number**:

$$C_n = \frac{1}{n+1}\binom{2n}{n}$$

| $n$ | $C_n$ | Binary Trees |
|:---:|:-----:|:------------:|
| 0 | 1 | $\emptyset$ |
| 1 | 1 | Single node |
| 2 | 2 | Left child or right child |
| 3 | 5 | |
| 4 | 14 | |
| 5 | 42 | |

```
  The 5 binary trees with 3 nodes:
  
   *       *       *         *       *
  /       /         \       / \       \
 *       *           *     *   *       *
/         \         /                   \
*           *     *                      *
```

---

## 8. Balanced Binary Trees

### AVL Trees
- Height of left and right subtrees differ by at most 1
- Guarantees $O(\log n)$ operations
- Uses **rotations** to maintain balance after insert/delete

### Complete Binary Trees
- Used as the underlying structure for **binary heaps**
- Can be stored efficiently in an array:
  - Parent of node $i$: $\lfloor i/2 \rfloor$
  - Left child of node $i$: $2i$
  - Right child of node $i$: $2i + 1$

```
  Array representation of complete binary tree:
  
  Index:   1    2    3    4    5    6    7
  Value: [50] [30] [40] [10] [20] [35] [25]
  
  Tree:
            50
           /  \
         30    40
        / \   / \
      10  20 35 25
```

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │         Binary Trees in Practice                 │
  │                                                  │
  │  1. Binary Search Trees                          │
  │     Databases, dictionaries, symbol tables       │
  │                                                  │
  │  2. Expression Trees                             │
  │     Compilers, calculators                       │
  │                                                  │
  │  3. Huffman Coding                               │
  │     Data compression (full binary tree)          │
  │                                                  │
  │  4. Binary Heaps                                 │
  │     Priority queues, heap sort                   │
  │                                                  │
  │  5. Decision Trees                               │
  │     Machine learning classification              │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Binary tree | Each node has at most 2 children |
| Full | Every node has 0 or 2 children |
| Complete | All levels full; last fills left to right |
| Perfect | All leaves at same depth, $2^{h+1}-1$ nodes |
| $l = i + 1$ | Leaves = internal nodes + 1 (full tree) |
| Height bounds | $\lfloor\log_2 n\rfloor \leq h \leq n-1$ |
| Catalan number | $C_n$ = number of trees with $n$ nodes |
| BST | Left < root < right |

---

## ❓ Quick Revision Questions

1. **A full binary tree has 15 leaves. How many internal nodes does it have?**

2. **What is the minimum height of a binary tree with 100 nodes?**

3. **How many structurally distinct binary trees have 4 nodes?**

4. **Draw the BST that results from inserting 5, 3, 7, 1, 4, 6, 8 in that order.**

5. **Convert the expression $(a + b) \times (c - d / e)$ into an expression tree.**

6. **A complete binary tree is stored in an array. Node at index 5 — what are its parent and children indices?**

---

[← Previous: Tree Properties](01-tree-properties.md) | [Back to README](../README.md) | [Next: Tree Traversals →](03-tree-traversals.md)
