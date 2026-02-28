# Unit 9 – Topic 3: Top View

[← Previous: Right View](02-right-view.md) | [Back to TOC](../README.md) | [Next: Bottom View →](04-bottom-view.md)

---

## 9.3 Top View

### 📖 Problem

Print nodes visible when looking at the tree from **above** — for each horizontal distance, the topmost (first encountered in BFS) node.

```
              1(HD=0)
            /       \
       (HD=-1)2     3(HD=1)
          / \       / \
    (HD=-2)4 5(0) 6(0) 7(HD=2)
            / \
      (HD=-1)8 9(HD=1)

    HD:  -2  -1   0   1   2
         4    2   1   3   7
    
    Top View: [4, 2, 1, 3, 7]
    
    Note: nodes 5, 6, 8, 9 are hidden behind other nodes
```

### Algorithm (BFS + HashMap)

```
FUNCTION topView(root):
    IF root == NULL: RETURN []
    
    map ← TreeMap()    // HD → node value (sorted by HD)
    queue ← [(root, 0)]   // (node, horizontal_distance)
    
    WHILE queue is not empty:
        (node, hd) ← queue.dequeue()
        
        // Only record FIRST node at each HD
        IF hd NOT in map:
            map[hd] ← node.data
        
        IF node.left: queue.enqueue((node.left, hd - 1))
        IF node.right: queue.enqueue((node.right, hd + 1))
    
    RETURN map values sorted by HD
```

### 🔍 Trace

```
            1
           / \
          2   3
         / \   \
        4   5   7

    BFS with HD:
    Process (1, HD=0) → map = {0:1}
    Process (2, HD=-1) → map = {-1:2, 0:1}
    Process (3, HD=1) → map = {-1:2, 0:1, 1:3}
    Process (4, HD=-2) → map = {-2:4, -1:2, 0:1, 1:3}
    Process (5, HD=0) → HD=0 already in map → SKIP
    Process (7, HD=2) → map = {-2:4, -1:2, 0:1, 1:3, 2:7}
    
    Top View: [4, 2, 1, 3, 7]
```

---

### ❓ Revision Question

**Can Top View be solved using DFS instead of BFS?**
<details><summary>Answer</summary>Yes, but BFS is preferred because it naturally processes nodes level by level, ensuring the "topmost" node at each HD is correctly identified. With DFS, you'd need to track depth and only keep the node with the smallest depth at each HD, which is more complex.</details>

---

[← Previous: Right View](02-right-view.md) | [Back to TOC](../README.md) | [Next: Bottom View →](04-bottom-view.md)
