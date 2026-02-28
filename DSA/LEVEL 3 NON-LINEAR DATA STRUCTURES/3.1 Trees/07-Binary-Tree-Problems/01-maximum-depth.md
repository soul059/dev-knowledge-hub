# Unit 7 – Topic 1: Maximum Depth (Height)

[← Previous: Convert Tree Types](../06-Tree-Construction/06-convert-tree-types.md) | [Back to TOC](../README.md) | [Next: Minimum Depth →](02-minimum-depth.md)

---

## Chapter Overview

This unit covers the **most common binary tree problems** — depth calculations, diameter, balance checking, tree comparison, and symmetry. These form the foundation of tree problem-solving and are frequently asked in interviews. Each problem illustrates key recursive thinking patterns.

---

## Reference Tree

```
            1
           / \
          2    3
         / \
        4   5
           /
          6
```

---

## 7.1 Maximum Depth (Height)

### 📖 Problem

Find the maximum depth (height) of a binary tree. The depth is the number of nodes along the longest path from the root to the deepest leaf.

### 💡 Key Insight

> Maximum depth = 1 + max(depth of left subtree, depth of right subtree). This is a classic **bottom-up recursion**.

### Algorithm

```
FUNCTION maxDepth(node):
    IF node == NULL:
        RETURN 0
    
    leftDepth  ← maxDepth(node.left)
    rightDepth ← maxDepth(node.right)
    
    RETURN 1 + MAX(leftDepth, rightDepth)
```

### 🔍 Trace

```
            1
           / \
          2    3
         / \
        4   5

    maxDepth(4) → 1 + max(0, 0) = 1
    maxDepth(5) → 1 + max(0, 0) = 1
    maxDepth(2) → 1 + max(1, 1) = 2
    maxDepth(3) → 1 + max(0, 0) = 1
    maxDepth(1) → 1 + max(2, 1) = 3
    
    Answer: 3
```

### Iterative (Level Order)

```
FUNCTION maxDepthIterative(root):
    IF root == NULL: RETURN 0
    
    queue ← [root]
    depth ← 0
    
    WHILE queue is not empty:
        depth ← depth + 1
        levelSize ← queue.size()
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
    
    RETURN depth
```

### ⏱️ Complexity: O(n) time, O(h) space recursive / O(w) space iterative

---

[← Previous: Convert Tree Types](../06-Tree-Construction/06-convert-tree-types.md) | [Back to TOC](../README.md) | [Next: Minimum Depth →](02-minimum-depth.md)
