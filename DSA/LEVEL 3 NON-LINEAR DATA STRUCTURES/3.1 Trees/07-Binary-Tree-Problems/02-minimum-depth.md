# Unit 7 – Topic 2: Minimum Depth

[← Previous: Maximum Depth](01-maximum-depth.md) | [Back to TOC](../README.md) | [Next: Diameter of Tree →](03-diameter-of-tree.md)

---

## 7.2 Minimum Depth

### 📖 Problem

Find the minimum depth — the shortest path from root to the **nearest leaf node**.

### ⚠️ Common Mistake

```
    WRONG approach:
    minDepth(node) = 1 + min(minDepth(left), minDepth(right))
    
    This fails for:
        1               The above would give minDepth = 1
       /                (from NULL right child)
      2                 But the actual minimum depth is 2
     /                  (path: 1 → 2 → 3)
    3
    
    A NULL child is NOT a leaf! We need to reach an actual leaf.
```

### Correct Algorithm

```
FUNCTION minDepth(node):
    IF node == NULL:
        RETURN 0
    
    // If only right child exists
    IF node.left == NULL:
        RETURN 1 + minDepth(node.right)
    
    // If only left child exists
    IF node.right == NULL:
        RETURN 1 + minDepth(node.left)
    
    // Both children exist
    RETURN 1 + MIN(minDepth(node.left), minDepth(node.right))
```

### 🔍 Trace

```
        1
       / \
      2    3
     /      \
    4        5

    minDepth(4) → leaf → 1
    minDepth(2) → right is NULL → 1 + minDepth(4) = 2
    minDepth(5) → leaf → 1
    minDepth(3) → left is NULL → 1 + minDepth(5) = 2
    minDepth(1) → both exist → 1 + min(2, 2) = 3
    
    Answer: 3 (paths: 1→2→4 or 1→3→5, both length 3)
```

### BFS Approach (Optimal for Min Depth)

```
FUNCTION minDepthBFS(root):
    IF root == NULL: RETURN 0
    
    queue ← [root]
    depth ← 0
    
    WHILE queue is not empty:
        depth ← depth + 1
        levelSize ← queue.size()
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            // First leaf we encounter = minimum depth
            IF node.left == NULL AND node.right == NULL:
                RETURN depth
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
    
    RETURN depth
```

> **Why BFS is optimal**: BFS explores level by level, so the **first leaf** found is guaranteed to be at the minimum depth.

---

### ❓ Revision Question

**Why does the naive approach for minimum depth fail when a node has only one child?**
<details><summary>Answer</summary>The naive min(left, right) returns 0 for the NULL child, treating it as a leaf. But NULL is not a leaf — we need the shortest path to an actual leaf node. We must only consider the non-NULL child's depth in this case.</details>

---

[← Previous: Maximum Depth](01-maximum-depth.md) | [Back to TOC](../README.md) | [Next: Diameter of Tree →](03-diameter-of-tree.md)
