# Unit 6 – Topic 4: Serialize and Deserialize

[← Previous: Build from Level Order](03-build-from-level-order.md) | [Back to TOC](../README.md) | [Next: Clone Binary Tree →](05-clone-binary-tree.md)

---

## 6.4 Serialize and Deserialize

### 📖 Problem

Design an algorithm to **serialize** a binary tree into a string and **deserialize** it back into the tree.

### 💡 Key Insight

> Use **preorder traversal** with NULL markers. Each NULL is serialized as a special marker (e.g., `"#"` or `"null"`).

### Serialize (Preorder with NULLs)

```
FUNCTION serialize(root):
    IF root == NULL:
        RETURN "#"
    
    RETURN root.data + "," + serialize(root.left) + "," + serialize(root.right)
```

### Deserialize

```
FUNCTION deserialize(data):
    tokens ← split data by ","
    index ← 0
    
    FUNCTION build():
        IF index ≥ tokens.length OR tokens[index] == "#":
            index ← index + 1
            RETURN NULL
        
        node ← new TreeNode(tokens[index])
        index ← index + 1
        node.left  ← build()
        node.right ← build()
        RETURN node
    
    RETURN build()
```

### 🔍 Trace

```
    Tree:       1
               / \
              2   3
                 / \
                4   5

    Serialize:
    serialize(1)
    = "1," + serialize(2) + "," + serialize(3)
    = "1," + "2,#,#" + "," + "3," + serialize(4) + "," + serialize(5)
    = "1,2,#,#,3,4,#,#,5,#,#"
    
    Deserialize: "1,2,#,#,3,4,#,#,5,#,#"
    tokens = [1, 2, #, #, 3, 4, #, #, 5, #, #]
    
    build(): token=1, idx=1
      ├── build(): token=2, idx=2
      │   ├── build(): token=#, idx=3 → NULL
      │   └── build(): token=#, idx=4 → NULL
      │   → node(2)
      └── build(): token=3, idx=5
          ├── build(): token=4, idx=6
          │   ├── build(): token=#, idx=7 → NULL
          │   └── build(): token=#, idx=8 → NULL
          │   → node(4)
          └── build(): token=5, idx=9
              ├── build(): token=#, idx=10 → NULL
              └── build(): token=#, idx=11 → NULL
              → node(5)
          → node(3)
      → node(1)
    
    Reconstructed tree:
            1
           / \
          2   3
             / \
            4   5  ✓
```

### Level-Order Serialization (Alternative)

```
    Serialize:  "1,2,3,#,#,4,5,#,#,#,#"
    (Level order with NULL markers for missing children)
    
    This is the format commonly used by LeetCode.
```

### ⏱️ Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Serialize | O(n) | O(n) |
| Deserialize | O(n) | O(n) |

---

### ❓ Revision Question

**What makes preorder serialization with NULL markers sufficient to reconstruct a unique tree?**
<details><summary>Answer</summary>The NULL markers explicitly encode the tree structure — they tell us exactly where each subtree ends. This is equivalent to having both the preorder and the structure, removing any ambiguity.</details>

---

[← Previous: Build from Level Order](03-build-from-level-order.md) | [Back to TOC](../README.md) | [Next: Clone Binary Tree →](05-clone-binary-tree.md)
