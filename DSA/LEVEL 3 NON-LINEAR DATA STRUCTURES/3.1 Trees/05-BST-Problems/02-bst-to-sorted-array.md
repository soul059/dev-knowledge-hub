# Unit 5 – Topic 2: BST to Sorted Array

[← Previous: Sorted Array to BST](01-sorted-array-to-bst.md) | [Back to TOC](../README.md) | [Next: Kth Smallest Element →](03-kth-smallest-element.md)

---

## 5.2 BST to Sorted Array

### 📖 Problem

Convert a BST to a sorted array.

### 💡 Key Insight

> Simply do an **inorder traversal** — inorder on BST gives sorted order!

### Algorithm

```
FUNCTION bstToSortedArray(node, result):
    IF node == NULL:
        RETURN
    
    bstToSortedArray(node.left, result)
    result.append(node.data)
    bstToSortedArray(node.right, result)
```

### 🔍 Trace

```
            4
           / \
          2   6
         / \ / \
        1  3 5  7
    
    Inorder traversal: 1 → 2 → 3 → 4 → 5 → 6 → 7
    
    result = [1, 2, 3, 4, 5, 6, 7]  ✓ Sorted!
```

### ⏱️ Complexity

| Aspect | Value |
|--------|-------|
| Time | O(n) |
| Space | O(n) for result + O(h) for recursion stack |

---

[← Previous: Sorted Array to BST](01-sorted-array-to-bst.md) | [Back to TOC](../README.md) | [Next: Kth Smallest Element →](03-kth-smallest-element.md)
