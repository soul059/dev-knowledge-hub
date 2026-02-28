# 2.5 Properties and Theorems

[← Previous: Maximum and Minimum Nodes](04-max-min-nodes.md) | [Back to TOC](../README.md) | [Next: Height Calculation →](06-height-calculation.md)

---

## 📖 Fundamental Properties

**Property 1**: Max nodes at level `l` = `2^l`
```
    Level 0: 2⁰ = 1 node       ●
    Level 1: 2¹ = 2 nodes      ● ●
    Level 2: 2² = 4 nodes      ● ● ● ●
    Level 3: 2³ = 8 nodes      ● ● ● ● ● ● ● ●
```

**Property 2**: Max nodes in tree of height `h` = `2^(h+1) - 1`
```
    h=0: 2¹-1 = 1
    h=1: 2²-1 = 3
    h=2: 2³-1 = 7
    h=3: 2⁴-1 = 15
```

**Property 3**: Min height of tree with `n` nodes = `⌊log₂(n)⌋`

**Property 4**: In a full binary tree, `Leaves = Internal nodes + 1`
```
          1          Internal nodes: {1, 2} → count = 2
         / \         Leaf nodes: {4, 5, 3}  → count = 3
        2   3        3 = 2 + 1 ✓
       / \
      4   5
```

**Property 5**: Number of NULL pointers in a binary tree with `n` nodes = `n + 1`
```
    General rule: n nodes → 2n pointers total → n-1 are used for edges 
    → (2n - (n-1)) = n+1 are NULL.
```

---

## Proof: Leaves = Internal + 1 (Full Binary Tree)

```
    Let n₀ = leaf nodes, n₂ = nodes with 2 children
    Total nodes n = n₀ + n₂
    Total edges = n - 1  (tree property)
    
    Edges contributed: Each n₂ node contributes 2 edges
    So: 2 * n₂ = n - 1 = n₀ + n₂ - 1
    Therefore: n₂ = n₀ - 1
    Or: n₀ = n₂ + 1  ✓
```

---

## ⏱️ Complexity Implications

| Tree Shape | Height | Search/Insert/Delete |
|-----------|--------|---------------------|
| **Perfect / Complete** | O(log n) | O(log n) |
| **Balanced** | O(log n) | O(log n) |
| **Skewed** | O(n) | O(n) |
| **Random** | O(log n) avg | O(log n) avg |

---

[← Previous: Maximum and Minimum Nodes](04-max-min-nodes.md) | [Back to TOC](../README.md) | [Next: Height Calculation →](06-height-calculation.md)
