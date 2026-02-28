# 1.2 Tree Terminology

[← Previous: What is a Tree?](01-what-is-a-tree.md) | [Back to TOC](../README.md) | [Next: Tree Properties →](03-tree-properties.md)

---

## 📖 Essential Terms

```
            1  ← Root
           / \
          2    3
         / \    \
        4   5    6
       /   / \
      7   8   9
```

| Term | Definition | Example (from above) |
|------|-----------|---------------------|
| **Root** | The topmost node (no parent) | Node `1` |
| **Parent** | A node that has children | `2` is parent of `4, 5` |
| **Child** | A node that has a parent | `4, 5` are children of `2` |
| **Siblings** | Nodes sharing the same parent | `4` and `5` are siblings |
| **Leaf** | A node with no children (external node) | `7, 8, 9, 6` |
| **Internal node** | A node with at least one child | `1, 2, 3, 4, 5` |
| **Edge** | A link between a parent and child | `1-2`, `2-4`, etc. |
| **Path** | A sequence of nodes connected by edges | `1 → 2 → 5 → 9` |
| **Ancestor** | Any node on the path from root to a node | Ancestors of `7`: `4, 2, 1` |
| **Descendant** | Any node reachable going downward | Descendants of `2`: `4, 5, 7, 8, 9` |
| **Subtree** | A node and all its descendants | Subtree rooted at `5`: `{5, 8, 9}` |
| **Degree** | Number of children of a node | Degree of `5` = 2, degree of `6` = 0 |
| **Degree of tree** | Maximum degree among all nodes | 2 (nodes `2` and `5`) |

---

## Visual Guide to Relationships

```
                    1          Level 0  (Root)
                   / \
                  2    3       Level 1
                 / \    \
                4   5    6     Level 2
               /   / \
              7   8   9       Level 3  (Leaves: 7, 8, 9, 6)
                
    Depth of node 5 = 2  (edges from root to node 5: 1→2→5)
    Height of node 2 = 2  (edges from node 2 to deepest leaf: 2→5→8)
    Height of tree = 3    (edges from root to deepest leaf: 1→2→5→8)
```

---

## Key Observations

- **Every node except root** has exactly one parent
- **Leaf nodes** are also called **external nodes**
- **Degree of a node** = number of its children (not including parent)
- A tree with `n` nodes has `n - 1` edges (each non-root node contributes exactly one edge to its parent)

---

[← Previous: What is a Tree?](01-what-is-a-tree.md) | [Back to TOC](../README.md) | [Next: Tree Properties →](03-tree-properties.md)
