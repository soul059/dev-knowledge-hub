# Unit 6 – Topic 1: Build from Inorder + Preorder

[← Previous: BST Iterator](../05-BST-Problems/06-bst-iterator.md) | [Back to TOC](../README.md) | [Next: Build from Inorder + Postorder →](02-build-from-inorder-postorder.md)

---

## Chapter Overview

This unit covers how to **construct (reconstruct) binary trees** from various traversal combinations, how to serialize/deserialize trees, clone trees, and convert between different tree types. These problems test deep understanding of traversal properties and recursive thinking.

---

## 6.1 Build from Inorder + Preorder

### 📖 Problem

Given the inorder and preorder traversals of a binary tree, reconstruct the tree.

### 💡 Key Insight

```
    Preorder: [Root] [Left subtree] [Right subtree]
               ↑
               First element is always the ROOT

    Inorder:  [Left subtree] [Root] [Right subtree]
                               ↑
               Root splits left and right subtrees
```

> **Strategy**: Pick root from preorder (first element), find it in inorder to determine left and right subtree boundaries, then recurse.

### Algorithm

```
FUNCTION buildTree(preorder, inorder):
    IF preorder is empty:
        RETURN NULL
    
    // Root is first element of preorder
    rootVal ← preorder[0]
    root ← new TreeNode(rootVal)
    
    // Find root's position in inorder
    mid ← index of rootVal in inorder
    
    // Elements left of mid in inorder = left subtree
    // Elements right of mid in inorder = right subtree
    root.left  ← buildTree(preorder[1..mid], inorder[0..mid-1])
    root.right ← buildTree(preorder[mid+1..end], inorder[mid+1..end])
    
    RETURN root
```

### 🔍 Trace

```
    Preorder: [3, 9, 20, 15, 7]
    Inorder:  [9, 3, 15, 20, 7]
    
    Step 1: root = 3 (first in preorder)
            Find 3 in inorder → index 1
            Left inorder:  [9]        (1 element)
            Right inorder: [15, 20, 7] (3 elements)
            Left preorder:  [9]        (next 1 from preorder)
            Right preorder: [20, 15, 7] (next 3 from preorder)
    
    Step 2: Build left subtree
            Preorder: [9], Inorder: [9]
            root = 9 (leaf)
    
    Step 3: Build right subtree
            Preorder: [20, 15, 7], Inorder: [15, 20, 7]
            root = 20
            Find 20 in inorder → index 1
            Left: pre=[15], in=[15]  → leaf node 15
            Right: pre=[7], in=[7]   → leaf node 7
    
    Result:
            3
           / \
          9   20
             / \
            15   7
```

### Optimization: HashMap for O(1) Lookup

```
FUNCTION buildTreeOptimized(preorder, inorder):
    // Pre-build hashmap: value → index in inorder
    map ← {}
    FOR i ← 0 to inorder.length - 1:
        map[inorder[i]] ← i
    
    preIdx ← 0   // Global index tracking preorder position
    
    FUNCTION build(inLeft, inRight):
        IF inLeft > inRight:
            RETURN NULL
        
        rootVal ← preorder[preIdx]
        preIdx ← preIdx + 1
        root ← new TreeNode(rootVal)
        
        mid ← map[rootVal]
        
        root.left  ← build(inLeft, mid - 1)    // LEFT first!
        root.right ← build(mid + 1, inRight)
        
        RETURN root
    
    RETURN build(0, inorder.length - 1)
```

### ⏱️ Complexity

| Version | Time | Space |
|---------|------|-------|
| Basic (linear search) | O(n²) | O(n) |
| Optimized (hashmap) | O(n) | O(n) |

---

### ❓ Revision Question

**Why do we need inorder traversal to uniquely reconstruct a binary tree?**
<details><summary>Answer</summary>Inorder traversal tells us which nodes belong to the left subtree and which to the right subtree of any given root. Without it, we cannot determine the boundary between left and right subtrees. Preorder or postorder alone can give us the root but not the split.</details>

---

[← Previous: BST Iterator](../05-BST-Problems/06-bst-iterator.md) | [Back to TOC](../README.md) | [Next: Build from Inorder + Postorder →](02-build-from-inorder-postorder.md)
