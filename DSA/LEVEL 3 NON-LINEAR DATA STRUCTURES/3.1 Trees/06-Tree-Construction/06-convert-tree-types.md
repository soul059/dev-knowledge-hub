# Unit 6 – Topic 6: Convert Tree Types

[← Previous: Clone Binary Tree](05-clone-binary-tree.md) | [Back to TOC](../README.md) | [Next: Binary Tree Problems →](../07-Binary-Tree-Problems/01-maximum-depth.md)

---

## 6.6 Convert Tree Types

### Binary Tree to its Mirror

```
FUNCTION mirror(node):
    IF node == NULL:
        RETURN NULL
    
    // Swap left and right children
    temp ← node.left
    node.left ← node.right
    node.right ← temp
    
    // Recurse
    mirror(node.left)
    mirror(node.right)
    
    RETURN node
```

```
    Original:        Mirror:
    
        1               1
       / \             / \
      2   3           3   2
     / \   \         /   / \
    4   5   6       6   5   4
```

### Convert BST to Greater Sum Tree

Each node's value becomes the sum of all values greater than or equal to it.

```
FUNCTION convertToGST(root):
    sum ← 0
    
    FUNCTION reverseInorder(node):
        IF node == NULL: RETURN
        
        reverseInorder(node.right)    // Visit right first (larger values)
        
        sum ← sum + node.data
        node.data ← sum
        
        reverseInorder(node.left)
    
    reverseInorder(root)
    RETURN root
```

### 🔍 Trace: BST to Greater Sum Tree

```
    BST:   4
          / \
         1   6
    
    Values: [1, 4, 6]
    Reverse inorder: 6, 4, 1
    sum=6  → node 6 becomes 6
    sum=10 → node 4 becomes 10
    sum=11 → node 1 becomes 11
    
    GST:   10
          / \
         11   6
```

### Why Can't We Build a Unique Tree from Preorder + Postorder?

```
    Preorder:  [1, 2]    Postorder: [2, 1]
    
    Possibility 1:     Possibility 2:
        1                   1
       /                     \
      2                       2
    
    Both give the same preorder and postorder!
    Without inorder, we can't determine if a single child 
    is left or right.
    
    Exception: We CAN build unique full binary trees from 
    preorder + postorder (every node has 0 or 2 children).
```

---

## 🧩 Construction Patterns

| Given Traversals | Can Build Unique Tree? | Notes |
|-----------------|----------------------|-------|
| Inorder + Preorder | ✅ Yes | Most common approach |
| Inorder + Postorder | ✅ Yes | Build right first when using postorder index |
| Preorder + Postorder | ❌ No (in general) | Ambiguous for nodes with single child |
| Preorder + Postorder (full BT) | ✅ Yes | Unique for full binary trees |
| Level Order (with NULLs) | ✅ Yes | Queue-based construction |
| Level Order (without NULLs) | ❌ No | Missing structural info |
| Inorder alone | ❌ No | Multiple valid trees |
| Preorder alone (with NULLs) | ✅ Yes | Serialization format |

---

## 📝 Unit 6 Summary Table

| Concept | Key Point |
|---------|-----------|
| Inorder + Preorder | Preorder gives root; inorder splits left/right subtrees |
| Inorder + Postorder | Postorder gives root (last element); build right first |
| Level order build | Queue-based; process children pair by pair |
| Serialize | Preorder with NULL markers; converts tree → string |
| Deserialize | Parse tokens, build recursively; string → tree |
| Clone tree | Recursive deep copy; O(n) time and space |
| Mirror tree | Swap left and right at every node |
| Greater Sum Tree | Reverse inorder traversal with cumulative sum |
| Unique reconstruction | Requires inorder + one of (preorder/postorder) |

---

### ❓ Revision Questions

1. **Given Preorder [1,2,3] and Postorder [3,2,1], how many valid binary trees exist?**
   <details><summary>Answer</summary>Multiple trees are valid (e.g., all-left-skewed, or left child=2 with right-child=3 of node 2, etc.). Preorder + Postorder cannot uniquely determine the tree when nodes have exactly one child, since we can't tell if it's the left or right child.</details>

2. **What is the time complexity of the optimized inorder + preorder construction using a hashmap?**
   <details><summary>Answer</summary>O(n) time and O(n) space. The hashmap provides O(1) lookup for root position in inorder (instead of O(n) linear search), and each node is processed exactly once.</details>

---

[← Previous: Clone Binary Tree](05-clone-binary-tree.md) | [Back to TOC](../README.md) | [Next: Binary Tree Problems →](../07-Binary-Tree-Problems/01-maximum-depth.md)
