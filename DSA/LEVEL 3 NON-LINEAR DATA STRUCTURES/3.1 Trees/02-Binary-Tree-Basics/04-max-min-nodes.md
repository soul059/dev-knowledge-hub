# 2.4 Maximum and Minimum Nodes

[← Previous: Binary Tree Representation](03-binary-tree-representation.md) | [Back to TOC](../README.md) | [Next: Properties and Theorems →](05-properties-and-theorems.md)

---

## 📖 Node Count Formulas

| Condition | Minimum Nodes | Maximum Nodes |
|-----------|--------------|---------------|
| Height `h` (binary tree) | `h + 1` | `2^(h+1) - 1` |
| Height `h` (full binary tree) | `2h + 1` | `2^(h+1) - 1` |
| `n` nodes (height range) | `⌊log₂(n)⌋` (min height) | `n - 1` (max height) |

---

## Visual Examples

```
    Height h = 2:
    
    Minimum nodes (h+1 = 3):     Maximum nodes (2³-1 = 7):
    
        1                              1
       /                              / \
      2                              2   3
     /                              / \ / \
    3                              4  5 6  7
    
    (Skewed/degenerate)            (Perfect binary tree)
```

---

## Height Range for `n` Nodes

```
    n = 7 nodes:
    
    Minimum height = ⌊log₂(7)⌋ = 2       Maximum height = 7 - 1 = 6
    
        1                                      1
       / \                                    /
      2   3                                  2
     / \ / \                                /
    4  5 6  7                              3
                                          /
    h = 2 (perfect)                      4
                                        /
                                       5
                                      /
                                     6
                                    /
                                   7
                                   
                                   h = 6 (skewed)
```

---

## Key Takeaway

- **Balanced / complete trees** → height ≈ log₂(n) → efficient operations
- **Skewed trees** → height ≈ n → degrades to linked list performance

---

[← Previous: Binary Tree Representation](03-binary-tree-representation.md) | [Back to TOC](../README.md) | [Next: Properties and Theorems →](05-properties-and-theorems.md)
