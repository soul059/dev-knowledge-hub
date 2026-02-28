# Chapter 4: Tree BFS/DFS Problems

## 📋 Chapter Overview
Trees are the most common BFS/DFS interview topic. This chapter covers essential tree problems using both approaches.

---

## 🔍 Problem 1: Maximum Depth of Binary Tree

### DFS (Recursive — Cleanest)

```
function maxDepth(root):
    if root is null: return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

### Trace

```
        3
       / \
      9   20
         / \
        15   7

  maxDepth(3)
  = 1 + max(maxDepth(9), maxDepth(20))
  = 1 + max(1, 1 + max(maxDepth(15), maxDepth(7)))
  = 1 + max(1, 1 + max(1, 1))
  = 1 + max(1, 2)
  = 3  ✓
```

### BFS (Count Levels)

```
function maxDepth(root):
    if root is null: return 0
    queue = [root], depth = 0
    while queue is not empty:
        depth += 1
        levelSize = queue.size()
        for i = 0 to levelSize-1:
            node = queue.dequeue()
            if node.left:  queue.enqueue(node.left)
            if node.right: queue.enqueue(node.right)
    return depth
```

---

## 🔍 Problem 2: Path Sum (DFS)

**Does any root-to-leaf path have sum = target?**

```
function hasPathSum(root, target):
    if root is null: return false
    
    target -= root.val
    
    if root.left is null AND root.right is null:
        return target == 0             // leaf check
    
    return hasPathSum(root.left, target) OR
           hasPathSum(root.right, target)
```

### Trace

```
        5             target = 22
       / \
      4   8           5→4→11→2 = 22 ✓
     /   / \
    11  13  4
   / \       \
  7   2       1

  hasPathSum(5, 22) → target=17
  hasPathSum(4, 17) → target=13
  hasPathSum(11, 13) → target=2
  hasPathSum(7, 2)  → target=-5, leaf, -5≠0 → false
  hasPathSum(2, 2)  → target=0, leaf, 0==0 → TRUE ✓
```

---

## 🔍 Problem 3: Lowest Common Ancestor (LCA)

```
function lowestCommonAncestor(root, p, q):
    if root is null OR root == p OR root == q:
        return root
    
    left  = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)
    
    if left AND right:
        return root            // p and q in different subtrees
    
    return left if left else right
```

### Visualization

```
        3
       / \
      5   1        LCA(5, 1) = 3
     / \ / \       LCA(5, 4) = 5
    6  2 0  8
      / \
     7   4
  
  At node 3:
    left  = LCA(5, ...) returns 5 (found p)
    right = LCA(1, ...) returns 1 (found q)
    Both non-null → return 3 ✓
```

---

## 🔍 Problem 4: Zigzag Level Order (BFS)

```
  Level 0 (L→R): [3]
  Level 1 (R→L): [20, 9]
  Level 2 (L→R): [15, 7]

        3
       / \
      9   20
         / \
        15   7
```

```
function zigzagLevelOrder(root):
    if root is null: return []
    result = [], queue = [root], leftToRight = true
    
    while queue is not empty:
        levelSize = queue.size()
        level = []
        for i = 0 to levelSize-1:
            node = queue.dequeue()
            if leftToRight:
                level.append(node.val)
            else:
                level.prepend(node.val)     // reverse direction
            if node.left:  queue.enqueue(node.left)
            if node.right: queue.enqueue(node.right)
        result.append(level)
        leftToRight = !leftToRight
    return result
```

---

## 🔍 Problem 5: Validate BST (DFS with Range)

```
function isValidBST(root):
    return validate(root, -INF, +INF)

function validate(node, minVal, maxVal):
    if node is null: return true
    if node.val <= minVal OR node.val >= maxVal:
        return false
    return validate(node.left, minVal, node.val) AND
           validate(node.right, node.val, maxVal)
```

### Trace

```
        5
       / \
      1   7        validate(5, -INF, INF) → ok
         / \       validate(1, -INF, 5)   → ok
        6   8      validate(7, 5, INF)    → ok
                   validate(6, 5, 7)      → ok
                   validate(8, 7, INF)    → ok ✓
```

---

## 🔍 Problem 6: Serialize / Deserialize Binary Tree

### BFS Serialization

```
function serialize(root):
    if root is null: return "null"
    result = []
    queue = [root]
    while queue is not empty:
        node = queue.dequeue()
        if node is null:
            result.append("null")
        else:
            result.append(str(node.val))
            queue.enqueue(node.left)
            queue.enqueue(node.right)
    return join(result, ",")

function deserialize(data):
    values = split(data, ",")
    if values[0] == "null": return null
    root = new Node(int(values[0]))
    queue = [root]
    i = 1
    while queue is not empty:
        node = queue.dequeue()
        if values[i] != "null":
            node.left = new Node(int(values[i]))
            queue.enqueue(node.left)
        i += 1
        if values[i] != "null":
            node.right = new Node(int(values[i]))
            queue.enqueue(node.right)
        i += 1
    return root
```

---

## 🔍 Problem 7: Right Side View (BFS)

**Return values visible from the right side = last node of each level.**

```
function rightSideView(root):
    if root is null: return []
    result = [], queue = [root]
    while queue is not empty:
        levelSize = queue.size()
        for i = 0 to levelSize-1:
            node = queue.dequeue()
            if i == levelSize - 1:
                result.append(node.val)    // last in level
            if node.left:  queue.enqueue(node.left)
            if node.right: queue.enqueue(node.right)
    return result
```

---

## 📊 BFS vs DFS for Tree Problems

| Problem | Better Approach | Why |
|---------|----------------|-----|
| Max depth | DFS | Simple recursion |
| Level order | BFS | Natural level grouping |
| Path sum | DFS | Follow root-to-leaf paths |
| LCA | DFS | Post-order bottom-up |
| Zigzag order | BFS | Level-by-level with flip |
| Validate BST | DFS | Pass range down |
| Serialize | BFS or DFS | Both work well |
| Right side view | BFS | Need last node per level |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| DFS on trees | Recursion is natural — pre/in/post order |
| BFS on trees | Level-order — use `levelSize` snapshot |
| Path problems | DFS — pass state down recursively |
| LCA | Post-order DFS — return found node upward |
| BST validation | DFS with min/max range constraint |
| Serialization | BFS for level-order, DFS for pre-order |

---

## ❓ Revision Questions

1. **Max depth: DFS or BFS?** → DFS is simpler (1 + max(left, right)), BFS also works (count levels).
2. **How does LCA work?** → Post-order DFS: if both subtrees return non-null, current node is the LCA.
3. **Zigzag: how to reverse alternate levels?** → Prepend instead of append on odd levels.
4. **Validate BST: why pass min/max range?** → Each node must be within a valid range; just checking left < root < right isn't enough (grandchild constraints).
5. **When is BFS better than DFS for trees?** → Level-order problems, right side view, minimum depth (stop at first leaf).

---

[← Previous: Graph Traversal Applications](03-graph-traversal-applications.md) | [Next: Classic Problems →](05-classic-problems.md)
