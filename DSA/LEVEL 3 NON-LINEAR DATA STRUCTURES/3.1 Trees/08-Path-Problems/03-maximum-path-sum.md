# Unit 8 – Topic 3: Maximum Path Sum

[← Previous: Path Sum Problems](02-path-sum-problems.md) | [Back to TOC](../README.md) | [Next: LCA in Binary Tree →](04-lca-binary-tree.md)

---

## 8.3 Maximum Path Sum

### 📖 Problem

Find the maximum sum of any path in the tree. A path can start and end at **any node** (not necessarily root or leaf). The path must go through connected nodes going only downward from a node (no revisiting).

### 💡 Key Insight

```
    At each node, we consider:
    1. Node alone
    2. Node + best left path
    3. Node + best right path
    4. Node + best left + best right (path PASSES through this node)
    
    For the ANSWER: consider all 4 cases
    For RETURNING to parent: only cases 1-3 (can't split at two branches)
```

### Algorithm

```
FUNCTION maxPathSum(root):
    maxSum ← -INFINITY
    
    FUNCTION maxGain(node):
        IF node == NULL:
            RETURN 0
        
        // Max gain from left and right (ignore negative gains)
        leftGain  ← MAX(0, maxGain(node.left))
        rightGain ← MAX(0, maxGain(node.right))
        
        // Path through this node (potential answer)
        currentPathSum ← node.data + leftGain + rightGain
        maxSum ← MAX(maxSum, currentPathSum)
        
        // Return max gain going through ONE side only
        RETURN node.data + MAX(leftGain, rightGain)
    
    maxGain(root)
    RETURN maxSum
```

### 🔍 Trace

```
            -10
           /   \
          9     20
               / \
              15   7

    maxGain(9):  left=0, right=0
                 pathSum = 9+0+0 = 9 → maxSum=9
                 return 9
    
    maxGain(15): left=0, right=0
                 pathSum = 15 → maxSum=15
                 return 15
    
    maxGain(7):  left=0, right=0
                 pathSum = 7 → maxSum=15
                 return 7
    
    maxGain(20): left=15, right=7
                 pathSum = 20+15+7 = 42 → maxSum=42
                 return 20+max(15,7) = 35
    
    maxGain(-10): left=max(0,9)=9, right=max(0,35)=35
                  pathSum = -10+9+35 = 34 → maxSum stays 42
                  return -10+max(9,35) = 25
    
    Answer: 42  (path: 15 → 20 → 7)
```

### ⏱️ Complexity: O(n) time, O(h) space

---

### ❓ Revision Question

**In Maximum Path Sum, why do we return only one branch to the parent but consider both branches for the answer?**
<details><summary>Answer</summary>A valid path can't fork — it goes from one node to another. When a path passes through a node using both branches, it "turns" at that node and can't extend further upward. So we update the global answer with left+right+node, but return only max(left,right)+node to the parent for possible extension.</details>

---

[← Previous: Path Sum Problems](02-path-sum-problems.md) | [Back to TOC](../README.md) | [Next: LCA in Binary Tree →](04-lca-binary-tree.md)
