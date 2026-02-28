# 4.5 BST Validation

[← Previous: BST Deletion](04-bst-deletion.md) | [Back to TOC](../README.md) | [Next: Inorder Successor & Predecessor →](06-inorder-successor-predecessor.md)

---

## 📖 Problem

Given a binary tree, determine if it is a valid BST.

---

## ⚠️ Common Mistake

```
    This approach is WRONG:
    
    FUNCTION isValidBST_WRONG(node):
        IF node == NULL: RETURN true
        IF node.left ≠ NULL AND node.left.data ≥ node.data: RETURN false
        IF node.right ≠ NULL AND node.right.data ≤ node.data: RETURN false
        RETURN isValidBST_WRONG(node.left) AND isValidBST_WRONG(node.right)
    
    Why wrong?
         5
        / \
       1   6        This check would say ✓
          / \       But 4 < 5 and is in RIGHT subtree!
         4   7      So it's actually INVALID!
```

---

## Correct Approach: Min-Max Range

```
FUNCTION isValidBST(node, min, max):
    IF node == NULL:
        RETURN true
    
    IF node.data ≤ min OR node.data ≥ max:
        RETURN false
    
    RETURN isValidBST(node.left, min, node.data) AND
           isValidBST(node.right, node.data, max)

// Initial call:
isValidBST(root, -INFINITY, +INFINITY)
```

---

## 🔍 Trace: Validate BST

```
         5
        / \
       3   7
      / \
     1   4

    isValidBST(5, -∞, +∞)
    ├── 5 is in (-∞, +∞)? YES
    ├── isValidBST(3, -∞, 5)
    │   ├── 3 is in (-∞, 5)? YES
    │   ├── isValidBST(1, -∞, 3)
    │   │   ├── 1 is in (-∞, 3)? YES
    │   │   ├── isValidBST(NULL) → true
    │   │   └── isValidBST(NULL) → true
    │   │   → true
    │   └── isValidBST(4, 3, 5)
    │       ├── 4 is in (3, 5)? YES
    │       ├── isValidBST(NULL) → true
    │       └── isValidBST(NULL) → true
    │       → true
    │   → true
    └── isValidBST(7, 5, +∞)
        ├── 7 is in (5, +∞)? YES
        ├── isValidBST(NULL) → true
        └── isValidBST(NULL) → true
        → true
    → true ✓ Valid BST!
```

---

## Alternative: Inorder Check

```
FUNCTION isValidBST_Inorder(root):
    prev ← -INFINITY
    
    FUNCTION inorder(node):
        IF node == NULL: RETURN true
        
        IF NOT inorder(node.left): RETURN false
        
        IF node.data ≤ prev: RETURN false
        prev ← node.data
        
        RETURN inorder(node.right)
    
    RETURN inorder(root)
```

> **Logic**: If inorder traversal is strictly increasing, it's a valid BST.

---

## ⏱️ Complexity

| Approach | Time | Space |
|----------|------|-------|
| Min-Max range | O(n) | O(h) |
| Inorder check | O(n) | O(h) |

---

[← Previous: BST Deletion](04-bst-deletion.md) | [Back to TOC](../README.md) | [Next: Inorder Successor & Predecessor →](06-inorder-successor-predecessor.md)
