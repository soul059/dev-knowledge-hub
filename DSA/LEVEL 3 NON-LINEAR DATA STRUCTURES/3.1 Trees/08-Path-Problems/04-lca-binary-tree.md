# Unit 8 – Topic 4: Lowest Common Ancestor (LCA) in Binary Tree

[← Previous: Maximum Path Sum](03-maximum-path-sum.md) | [Back to TOC](../README.md) | [Next: Distance Between Nodes →](05-distance-between-nodes.md)

---

## 8.4 Lowest Common Ancestor (LCA) in Binary Tree

### 📖 Problem

Find the **lowest common ancestor** of two given nodes in a binary tree (not necessarily a BST).

### 💡 Key Insight

> Unlike BST where we use ordering, in a general binary tree we must **search both subtrees**. If both p and q are found in different subtrees, the current node is the LCA.

### Algorithm

```
FUNCTION lowestCommonAncestor(root, p, q):
    // Base cases
    IF root == NULL:
        RETURN NULL
    IF root == p OR root == q:
        RETURN root
    
    // Search in left and right subtrees
    left  ← lowestCommonAncestor(root.left, p, q)
    right ← lowestCommonAncestor(root.right, p, q)
    
    // If both found in different subtrees → root is LCA
    IF left ≠ NULL AND right ≠ NULL:
        RETURN root
    
    // Otherwise, return whichever is non-NULL
    RETURN left IF left ≠ NULL ELSE right
```

### 🔍 Trace: LCA(4, 5)

```
            3
           / \
          5    1
         / \  / \
        6   2 0  8
           / \
          7   4

    LCA(3, p=5, q=4):
    ├── LCA(5, 5, 4):
    │   root == p → return 5 (found p!)
    │
    └── LCA(1, 5, 4):
        ├── LCA(0, 5, 4) → NULL
        └── LCA(8, 5, 4) → NULL
        left=NULL, right=NULL → return NULL
    
    Back at 3: left=5, right=NULL
    → return 5
    
    LCA(5, 4) = 5 ✓  (5 is ancestor of 4, and also equals p)
```

### 🔍 Trace: LCA(6, 4)

```
    LCA(3, p=6, q=4):
    ├── LCA(5, 6, 4):
    │   ├── LCA(6, 6, 4): root == p → return 6
    │   └── LCA(2, 6, 4):
    │       ├── LCA(7, 6, 4) → NULL
    │       └── LCA(4, 6, 4): root == q → return 4
    │       left=NULL, right=4 → return 4
    │   left=6, right=4 → BOTH non-NULL! → return 5
    └── LCA(1, 6, 4) → NULL
    
    left=5, right=NULL → return 5
    
    LCA(6, 4) = 5 ✓
```

### ⏱️ Complexity: O(n) time, O(h) space

---

### ❓ Revision Question

**Why is LCA in a general binary tree O(n) while LCA in BST is O(h)?**
<details><summary>Answer</summary>In a BST, the ordering property lets us go directly left or right at each node (one path from root). In a general binary tree, we have no ordering, so we must potentially search both subtrees at every node, visiting all n nodes in the worst case.</details>

---

[← Previous: Maximum Path Sum](03-maximum-path-sum.md) | [Back to TOC](../README.md) | [Next: Distance Between Nodes →](05-distance-between-nodes.md)
