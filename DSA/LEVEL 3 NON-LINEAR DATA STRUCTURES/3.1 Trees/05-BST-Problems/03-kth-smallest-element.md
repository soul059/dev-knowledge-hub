# Unit 5 – Topic 3: Kth Smallest Element

[← Previous: BST to Sorted Array](02-bst-to-sorted-array.md) | [Back to TOC](../README.md) | [Next: LCA in BST →](04-lca-in-bst.md)

---

## 5.3 Kth Smallest Element

### 📖 Problem

Find the Kth smallest element in a BST.

### 💡 Key Insight

> Inorder traversal visits nodes in sorted order. The Kth node visited in inorder is the Kth smallest element.

### Approach 1: Inorder with Counter

```
FUNCTION kthSmallest(node, k):
    count ← 0
    result ← -1
    
    FUNCTION inorder(node):
        IF node == NULL OR count ≥ k:
            RETURN
        
        inorder(node.left)
        
        count ← count + 1
        IF count == k:
            result ← node.data
            RETURN
        
        inorder(node.right)
    
    inorder(node)
    RETURN result
```

### 🔍 Trace: Find 3rd Smallest

```
            5
           / \
          3   7
         / \
        2   4
       /
      1

    Inorder: 1, 2, 3, 4, 5, 7
    
    Visit 1: count=1 (k=3, continue)
    Visit 2: count=2 (k=3, continue)
    Visit 3: count=3 (k=3, FOUND!) → Answer = 3
```

### Approach 2: Augmented BST

If Kth smallest is queried frequently, augment each node with a **size** field.

```
    Node:  (data, size)
    
            5(6)          size = total nodes in subtree
           / \
          3(4)  7(1)
         / \
        2(2) 4(1)
       /
      1(1)
    
    To find Kth smallest:
    leftSize = size of left subtree
    
    If k == leftSize + 1 → current node is answer
    If k ≤ leftSize      → go to left subtree
    If k > leftSize + 1  → go to right subtree with k = k - leftSize - 1
```

### ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Inorder traversal | O(h + k) | O(h) |
| Augmented BST | O(h) | O(n) for augmentation |

---

### ❓ Revision Question

**Can you find the Kth largest element efficiently in a BST?**
<details><summary>Answer</summary>Yes — use reverse inorder traversal (Right → Root → Left) and count. The Kth node visited is the Kth largest. This is O(h + k) time and O(h) space.</details>

---

[← Previous: BST to Sorted Array](02-bst-to-sorted-array.md) | [Back to TOC](../README.md) | [Next: LCA in BST →](04-lca-in-bst.md)
