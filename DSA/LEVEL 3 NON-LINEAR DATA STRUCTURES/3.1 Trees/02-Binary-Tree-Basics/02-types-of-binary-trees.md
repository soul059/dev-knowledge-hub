# 2.2 Types of Binary Trees

[← Previous: Binary Tree Definition](01-binary-tree-definition.md) | [Back to TOC](../README.md) | [Next: Binary Tree Representation →](03-binary-tree-representation.md)

---

## Full (Strict / Proper) Binary Tree

Every node has **0 or 2 children** — no node has just one child.

```
         1
        / \
       2   3          ✓ Full: every node has 0 or 2 children
      / \
     4   5
    
         1
        / \
       2   3          ✗ NOT Full: node 3 has only 1 child
      / \   \
     4   5   6
```

**Property**: If `L` = number of leaves and `I` = number of internal nodes, then **`L = I + 1`**.

---

## Complete Binary Tree

All levels are completely filled **except possibly the last level**, which is filled from **left to right**.

```
         1                    1
        / \                  / \
       2   3                2   3          
      / \ /                / \   \         
     4  5 6               4   5   6        
     
     ✓ Complete            ✗ NOT Complete
     (last level           (node 6 is right
      filled left→right)    child, but 3 has
                             no left child)
```

**Property**: A complete binary tree of height `h` has between `2^h` and `2^(h+1) - 1` nodes.

**Key Use**: Heaps are always complete binary trees!

---

## Perfect Binary Tree

All internal nodes have **exactly 2 children** AND all **leaves are at the same level**.

```
         1               Level 0: 1 node
        / \
       2   3             Level 1: 2 nodes
      / \ / \
     4  5 6  7           Level 2: 4 nodes
     
     ✓ Perfect Binary Tree
     
     Total nodes = 2^(h+1) - 1 = 2^3 - 1 = 7
```

**Properties**:
- Total nodes = `2^(h+1) - 1`
- Leaf nodes = `2^h`
- Internal nodes = `2^h - 1`
- Leaves = Internal nodes + 1

---

## Balanced Binary Tree

The **height difference** between left and right subtrees of **every node** is at most 1.

```
       1                        1
      / \                      / \
     2   3      ✓ Balanced    2   3       ✗ NOT Balanced
    / \                      /
   4   5                    4             |height(left) - height(right)|
                           /              = |2 - 0| = 2 > 1 at node 1
                          5
```

**Balance Factor** = `height(left subtree) - height(right subtree)`

A tree is balanced if the balance factor of every node is in `{-1, 0, 1}`.

---

## Degenerate (Pathological / Skewed) Tree

Every internal node has **only one child**. Behaves like a linked list.

```
  Left Skewed:     Right Skewed:
  
       1                1
      /                  \
     2                    2
    /                      \
   3                        3
  /                          \
 4                            4
 
 Height = n - 1 (worst case)
 All operations become O(n)
```

---

## Comparison Chart

```
  Full             Complete          Perfect           Balanced
  
     A               A                 A                 A
    / \             / \               / \               / \
   B   C           B   C             B   C             B   C
  / \             / \ /             / \ / \           / \
 D   E           D  E F            D  E F  G         D   E
 
 0 or 2          Fill left         All leaves        |hL - hR| ≤ 1
 children        to right          same level        at every node
```

| Type | Key Property | Nodes (height h) |
|------|-------------|-------------------|
| Full | 0 or 2 children only | 2h+1 to 2^(h+1)-1 |
| Complete | Last level filled L→R | 2^h to 2^(h+1)-1 |
| Perfect | All leaves same level | Exactly 2^(h+1)-1 |
| Balanced | Height diff ≤ 1 at every node | — |
| Degenerate | Each node has ≤ 1 child | Exactly h+1 |

---

[← Previous: Binary Tree Definition](01-binary-tree-definition.md) | [Back to TOC](../README.md) | [Next: Binary Tree Representation →](03-binary-tree-representation.md)
