# Unit 8 – Topic 6: Nodes at Distance K

[← Previous: Distance Between Nodes](05-distance-between-nodes.md) | [Back to TOC](../README.md) | [Next: Tree Views →](../09-Tree-Views/01-left-view.md)

---

## 8.6 Nodes at Distance K

### 📖 Problem

Given a **target node** and a distance `K`, find all nodes that are exactly K edges away from the target. Nodes can be in any direction — descendants, ancestors, or in other subtrees.

```
            3
           / \
          5    1
         / \  / \
        6   2 0  8
           / \
          7   4

    Target = 5, K = 2
    Answer: [7, 4, 1]  (all at distance 2 from node 5)
    
    7 is grandchild of 5 (distance 2 down-left)
    4 is grandchild of 5 (distance 2 down-right)
    1 is sibling of 5 via parent 3 (distance 2 up)
```

### 💡 Key Insight

> The tricky part is going **upward** from the target. We need parent pointers or a parent map. Build a parent map using BFS/DFS first, then do BFS from the target node.

### Algorithm

```
FUNCTION distanceK(root, target, K):
    // Step 1: Build parent map
    parentMap ← {}
    
    FUNCTION buildParentMap(node, parent):
        IF node == NULL: RETURN
        parentMap[node] ← parent
        buildParentMap(node.left, node)
        buildParentMap(node.right, node)
    
    buildParentMap(root, NULL)
    
    // Step 2: BFS from target node
    queue ← [target]
    visited ← {target}
    distance ← 0
    
    WHILE queue is not empty:
        IF distance == K:
            RETURN [node.data for node in queue]
        
        levelSize ← queue.size()
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            
            // Go to left child
            IF node.left ≠ NULL AND node.left NOT in visited:
                visited.add(node.left)
                queue.enqueue(node.left)
            
            // Go to right child
            IF node.right ≠ NULL AND node.right NOT in visited:
                visited.add(node.right)
                queue.enqueue(node.right)
            
            // Go to parent
            IF parentMap[node] ≠ NULL AND parentMap[node] NOT in visited:
                visited.add(parentMap[node])
                queue.enqueue(parentMap[node])
        
        distance ← distance + 1
    
    RETURN []
```

### 🔍 Trace: distanceK(target=5, K=2)

```
            3
           / \
          5    1
         / \  / \
        6   2 0  8
           / \
          7   4

    Parent map: {5→3, 1→3, 6→5, 2→5, 0→1, 8→1, 7→2, 4→2, 3→NULL}
    
    BFS from node 5:
    
    dist=0: queue=[5], visited={5}
    
    dist=1: Process 5
            children: 6 (add), 2 (add)
            parent: 3 (add)
            queue=[6, 2, 3], visited={5, 6, 2, 3}
    
    dist=2: Process 6 → children NULL, parent 5 visited → nothing new
            Process 2 → children 7 (add), 4 (add), parent 5 visited
            Process 3 → child 5 visited, child 1 (add), parent NULL
            queue=[7, 4, 1]
    
    dist == K == 2 → return [7, 4, 1] ✓
```

### ⏱️ Complexity: O(n) time, O(n) space

---

## 🧩 Problem-Solving Patterns

### Pattern: Path Tracking with Backtracking

```
    path.add(node)
    IF leaf: record path
    ELSE: recurse left/right
    path.removeLast()     ← backtrack!
```

### Pattern: Prefix Sum on Trees

For "sum of any subpath" problems, use prefix sums like in arrays:
```
    currentSum at node = sum from root to node
    If (currentSum - target) exists in prefix map, a valid subpath exists
    Remember to BACKTRACK the prefix map after recursion
```

### Pattern: LCA-Based Distance

```
    dist(a, b) = dist(LCA, a) + dist(LCA, b)
    
    Find LCA first, then compute individual depths
```

### Pattern: Convert to Graph (Parent Map)

When you need to traverse **upward** in a tree:
```
    Build parent map: node → parent
    Use BFS/DFS treating tree as undirected graph
    (left child, right child, parent)
```

---

## 📝 Unit 8 Summary Table

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| Root to leaf paths | DFS + backtracking | O(n) | O(h) |
| Path Sum I | Subtract and check at leaf | O(n) | O(h) |
| Path Sum II | DFS + backtracking, collect paths | O(n²) | O(n) |
| Path Sum III | Prefix sum HashMap | O(n) | O(n) |
| Max Path Sum | Bottom-up gain + global max | O(n) | O(h) |
| LCA (binary tree) | Post-order: check both subtrees | O(n) | O(h) |
| Distance between nodes | LCA + depth | O(n) | O(h) |
| Nodes at distance K | Parent map + BFS | O(n) | O(n) |

---

### ❓ Revision Question

**In the "Nodes at Distance K" problem, why can't we just do DFS downward from the target?**
<details><summary>Answer</summary>DFS downward only finds descendants at distance K. Nodes at distance K could also be ancestors, or in other subtrees (siblings, cousins, etc.). We need to traverse upward via parent pointers, which is why we build a parent map and treat the tree as an undirected graph.</details>

---

[← Previous: Distance Between Nodes](05-distance-between-nodes.md) | [Back to TOC](../README.md) | [Next: Tree Views →](../09-Tree-Views/01-left-view.md)
