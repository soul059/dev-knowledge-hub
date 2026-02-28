# Unit 8 – Topic 2: Path Sum Problems

[← Previous: Root to Leaf Paths](01-root-to-leaf-paths.md) | [Back to TOC](../README.md) | [Next: Maximum Path Sum →](03-maximum-path-sum.md)

---

## 8.2 Path Sum Problems

### Path Sum I: Does a path with given sum exist?

```
FUNCTION hasPathSum(node, targetSum):
    IF node == NULL:
        RETURN false
    
    // Subtract current node's value
    targetSum ← targetSum - node.data
    
    // If leaf and remaining sum is 0
    IF node.left == NULL AND node.right == NULL:
        RETURN targetSum == 0
    
    RETURN hasPathSum(node.left, targetSum) OR hasPathSum(node.right, targetSum)
```

### 🔍 Trace: hasPathSum(root, 7)

```
            1
           / \
          2    3
         / \
        4   5

    hasPathSum(1, 7):  remaining = 7-1 = 6
    ├── hasPathSum(2, 6): remaining = 6-2 = 4
    │   ├── hasPathSum(4, 4): remaining = 4-4 = 0
    │   │   LEAF! sum == 0? YES → return true ✓
    │   
    Path: 1 → 2 → 4, sum = 7 ✓
```

### Path Sum II: Find ALL paths with given sum

```
FUNCTION pathSumII(node, targetSum, path, result):
    IF node == NULL:
        RETURN
    
    path.append(node.data)
    targetSum ← targetSum - node.data
    
    IF node.left == NULL AND node.right == NULL AND targetSum == 0:
        result.append(copy of path)
    ELSE:
        pathSumII(node.left, targetSum, path, result)
        pathSumII(node.right, targetSum, path, result)
    
    path.removeLast()     // Backtrack
```

### Path Sum III: Count paths with sum (any start/end)

Paths don't have to start at root or end at leaf. Any downward path counts.

```
FUNCTION pathSumIII(root, targetSum):
    // Use prefix sum technique
    prefixSums ← {0: 1}   // Map: prefix_sum → count
    count ← 0
    
    FUNCTION dfs(node, currentSum):
        IF node == NULL: RETURN
        
        currentSum ← currentSum + node.data
        
        // Check if there's a prefix sum that gives target
        IF (currentSum - targetSum) in prefixSums:
            count ← count + prefixSums[currentSum - targetSum]
        
        // Add current sum to map
        prefixSums[currentSum] ← prefixSums.getOrDefault(currentSum, 0) + 1
        
        dfs(node.left, currentSum)
        dfs(node.right, currentSum)
        
        // Backtrack: remove current sum
        prefixSums[currentSum] ← prefixSums[currentSum] - 1
    
    dfs(root, 0)
    RETURN count
```

### 🔍 Trace: Path Sum III (target = 8)

```
            10
           / \
          5  -3
         / \   \
        3   2   11
       / \   \
      3  -2   1

    Prefix sum approach:
    
    Path 10→5→3:     sum=18, need prefix=18-8=10 → exists! (root 10)
                     → Path: 5→3
    Path 10→5→2→1:   sum=18, need prefix=18-8=10 → exists!
                     → Path: 5→2→1
    Path 10→-3→11:   sum=18, need prefix=18-8=10 → exists!
                     → Path: -3→11
    
    Total count = 3
```

### ⏱️ Complexity

| Variant | Time | Space |
|---------|------|-------|
| Path Sum I | O(n) | O(h) |
| Path Sum II | O(n²) worst | O(h × paths) |
| Path Sum III (prefix sum) | O(n) | O(n) |

---

### ❓ Revision Question

**How does the prefix sum technique for Path Sum III avoid O(n²) complexity?**
<details><summary>Answer</summary>Instead of checking all possible starting points for each ending point, we maintain a running prefix sum from root. If currentSum - targetSum exists in our prefix map, it means there's a subpath ending at the current node with the target sum. This is O(1) per node instead of O(n).</details>

---

[← Previous: Root to Leaf Paths](01-root-to-leaf-paths.md) | [Back to TOC](../README.md) | [Next: Maximum Path Sum →](03-maximum-path-sum.md)
