# Unit 8 – Topic 5: Distance Between Nodes

[← Previous: LCA in Binary Tree](04-lca-binary-tree.md) | [Back to TOC](../README.md) | [Next: Nodes at Distance K →](06-nodes-at-distance-k.md)

---

## 8.5 Distance Between Nodes

### 📖 Problem

Find the number of edges between two given nodes in a binary tree.

### 💡 Key Insight

```
    distance(a, b) = depth(a) + depth(b) - 2 × depth(LCA(a, b))
    
    Or equivalently:
    distance(a, b) = dist(LCA, a) + dist(LCA, b)
```

### Algorithm

```
FUNCTION distanceBetween(root, p, q):
    lca ← lowestCommonAncestor(root, p, q)
    d1 ← depth(lca, p, 0)    // Distance from LCA to p
    d2 ← depth(lca, q, 0)    // Distance from LCA to q
    RETURN d1 + d2

FUNCTION depth(root, target, currentDepth):
    IF root == NULL:
        RETURN -1
    IF root == target:
        RETURN currentDepth
    
    left ← depth(root.left, target, currentDepth + 1)
    IF left ≠ -1: RETURN left
    
    RETURN depth(root.right, target, currentDepth + 1)
```

### 🔍 Trace: Distance(4, 6)

```
            1
           / \
          2    3
         / \    \
        4   5    6

    Step 1: LCA(4, 6) = 1  (4 in left, 6 in right)
    Step 2: depth(1, 4) = 2  (1 → 2 → 4)
    Step 3: depth(1, 6) = 2  (1 → 3 → 6)
    
    Distance = 2 + 2 = 4
    
    Path: 4 → 2 → 1 → 3 → 6 (4 edges)
```

### All-in-One Approach

```
FUNCTION findDistance(root, p, q):
    // Find LCA and distances in one pass
    FUNCTION solve(node, p, q):
        IF node == NULL: RETURN (NULL, -1, -1)
        
        (leftLCA, leftDp, leftDq) ← solve(node.left, p, q)
        (rightLCA, rightDp, rightDq) ← solve(node.right, p, q)
        
        dp ← distance to p from current subtree
        dq ← distance to q from current subtree
        
        // If p is current node, dp = 0
        IF node == p: dp ← 0
        ELSE IF leftDp ≠ -1: dp ← leftDp + 1
        ELSE IF rightDp ≠ -1: dp ← rightDp + 1
        
        // Same for q
        IF node == q: dq ← 0
        ELSE IF leftDq ≠ -1: dq ← leftDq + 1
        ELSE IF rightDq ≠ -1: dq ← rightDq + 1
        
        // If both found, current is LCA or below LCA
        IF dp ≠ -1 AND dq ≠ -1:
            RETURN (node, dp, dq)
        
        RETURN (NULL, dp, dq)
    
    (lca, dp, dq) ← solve(root, p, q)
    RETURN dp + dq
```

---

### ❓ Revision Question

**What is the relationship between distance, LCA, and depth?**
<details><summary>Answer</summary>distance(a, b) = depth(a) + depth(b) - 2 × depth(LCA(a,b)), or equivalently, dist(LCA→a) + dist(LCA→b). The LCA is the highest node that is an ancestor of both a and b, so the path from a to b must go through the LCA.</details>

---

[← Previous: LCA in Binary Tree](04-lca-binary-tree.md) | [Back to TOC](../README.md) | [Next: Nodes at Distance K →](06-nodes-at-distance-k.md)
