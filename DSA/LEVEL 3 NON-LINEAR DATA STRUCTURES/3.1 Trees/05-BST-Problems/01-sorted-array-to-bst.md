# Unit 5 – Topic 1: Sorted Array to BST

[← Previous: Inorder Successor & Predecessor](../04-Binary-Search-Tree/06-inorder-successor-predecessor.md) | [Back to TOC](../README.md) | [Next: BST to Sorted Array →](02-bst-to-sorted-array.md)

---

## Chapter Overview

This unit covers **classic BST problems** that are frequently asked in interviews and competitive programming. Each problem exploits the **BST property** (left < root < right) for efficient solutions. You will learn patterns for converting between data structures, finding specific elements, and building iterators.

---

## 5.1 Sorted Array to BST

### 📖 Problem

Given a sorted array, convert it into a **height-balanced BST** (a BST where the depth of the two subtrees of every node never differs by more than 1).

### 💡 Key Insight

> The **middle element** of the sorted array should be the root (to ensure balance). The left half becomes the left subtree, and the right half becomes the right subtree. Apply this **recursively**.

### Algorithm

```
FUNCTION sortedArrayToBST(arr, left, right):
    IF left > right:
        RETURN NULL
    
    mid ← (left + right) / 2
    
    node ← new TreeNode(arr[mid])
    node.left  ← sortedArrayToBST(arr, left, mid - 1)
    node.right ← sortedArrayToBST(arr, mid + 1, right)
    
    RETURN node

// Initial call:
root = sortedArrayToBST(arr, 0, arr.length - 1)
```

### 🔍 Trace

```
    arr = [1, 2, 3, 4, 5, 6, 7]
    
    sortedArrayToBST(arr, 0, 6)
      mid = 3, root = 4
      ├── sortedArrayToBST(arr, 0, 2)
      │     mid = 1, root = 2
      │     ├── sortedArrayToBST(arr, 0, 0)
      │     │     mid = 0, root = 1 (leaf)
      │     └── sortedArrayToBST(arr, 2, 2)
      │           mid = 2, root = 3 (leaf)
      └── sortedArrayToBST(arr, 4, 6)
            mid = 5, root = 6
            ├── sortedArrayToBST(arr, 4, 4)
            │     mid = 4, root = 5 (leaf)
            └── sortedArrayToBST(arr, 6, 6)
                  mid = 6, root = 7 (leaf)
    
    Result:
            4
           / \
          2   6
         / \ / \
        1  3 5  7
    
    Height-balanced BST ✓
```

### ⏱️ Complexity

| Aspect | Value |
|--------|-------|
| Time | O(n) — visit each element once |
| Space | O(log n) — recursion depth for balanced tree |

---

### ❓ Revision Question

**How do you ensure a sorted array converts to a balanced BST?**
<details><summary>Answer</summary>Always pick the middle element of the current range as the root. This ensures the left and right subtrees have approximately equal numbers of nodes, giving a balanced tree with height O(log n).</details>

---

[← Previous: Inorder Successor & Predecessor](../04-Binary-Search-Tree/06-inorder-successor-predecessor.md) | [Back to TOC](../README.md) | [Next: BST to Sorted Array →](02-bst-to-sorted-array.md)
