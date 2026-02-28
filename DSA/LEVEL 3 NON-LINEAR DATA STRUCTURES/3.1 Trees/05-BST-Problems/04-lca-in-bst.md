# Unit 5 – Topic 4: Lowest Common Ancestor (LCA) in BST

[← Previous: Kth Smallest Element](03-kth-smallest-element.md) | [Back to TOC](../README.md) | [Next: Floor and Ceiling →](05-floor-and-ceiling.md)

---

## 5.4 Lowest Common Ancestor (LCA) in BST

### 📖 Problem

Find the lowest (deepest) node that is an ancestor of both nodes `p` and `q`.

### 💡 Key Insight

> In a BST, if both `p` and `q` are smaller than a node, their LCA is in the left subtree. If both are larger, it's in the right subtree. If they **split** (one goes left, one goes right), the current node **is the LCA**.

```
            6
           / \
          2   8
         / \ / \
        0  4 7  9
          / \
         3   5

    LCA(2, 8) = 6    (split at root)
    LCA(2, 4) = 2    (both in left, but 2 is ancestor of 4)
    LCA(3, 5) = 4    (split at 4)
```

### Algorithm

```
FUNCTION lcaBST(root, p, q):
    current ← root
    
    WHILE current ≠ NULL:
        IF p.data < current.data AND q.data < current.data:
            current ← current.left       // Both in left subtree
        ELSE IF p.data > current.data AND q.data > current.data:
            current ← current.right      // Both in right subtree
        ELSE:
            RETURN current                // Split point = LCA
```

### 🔍 Trace: LCA(3, 5)

```
            6
           / \
          2   8
         / \ / \
        0  4 7  9
          / \
         3   5

    Step 1: current = 6,  3 < 6 AND 5 < 6  → go LEFT
    Step 2: current = 2,  3 > 2 AND 5 > 2  → go RIGHT
    Step 3: current = 4,  3 < 4 but 5 > 4  → SPLIT! → LCA = 4 ✓
```

### ⏱️ Complexity

| Aspect | Value |
|--------|-------|
| Time | O(h) — follow a single path |
| Space | O(1) iterative, O(h) recursive |

---

### ❓ Revision Question

**Why is the LCA algorithm in BST O(h) while in a general binary tree it's O(n)?**
<details><summary>Answer</summary>In a BST, the ordering property tells us exactly which direction to go — left if both values are smaller, right if both are larger. In a general binary tree, we have no ordering to exploit and must search both subtrees.</details>

---

[← Previous: Kth Smallest Element](03-kth-smallest-element.md) | [Back to TOC](../README.md) | [Next: Floor and Ceiling →](05-floor-and-ceiling.md)
