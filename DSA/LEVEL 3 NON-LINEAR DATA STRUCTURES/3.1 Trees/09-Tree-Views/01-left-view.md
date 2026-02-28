# Unit 9 – Topic 1: Left View

[← Previous: Nodes at Distance K](../08-Path-Problems/06-nodes-at-distance-k.md) | [Back to TOC](../README.md) | [Next: Right View →](02-right-view.md)

---

## Chapter Overview

Tree "view" problems ask you to visualize the tree from a specific perspective — left, right, top, bottom, or boundary. These problems combine traversal techniques with clever tracking of positions. They are **extremely popular** in interviews.

---

## Reference Tree

```
              1
            /   \
           2     3
          / \   / \
         4   5 6   7
            / \
           8   9
```

---

## 9.1 Left View

### 📖 Problem

Print the nodes visible when looking at the tree from the **left side** — the first node at each level.

```
              1    ← visible (level 0)
            /   \
    →      2     3                  (level 1: see 2)
          / \   / \
    →    4   5 6   7               (level 2: see 4)
            / \
    →      8   9                   (level 3: see 8)

    Left View: [1, 2, 4, 8]
```

### Approach 1: Level Order (BFS)

The first node in each level is the left view.

```
FUNCTION leftView(root):
    IF root == NULL: RETURN []
    
    result ← []
    queue ← [root]
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            
            IF i == 0:                    // First node of level
                result.append(node.data)
            
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
    
    RETURN result
```

### Approach 2: DFS (Recursive)

Track the maximum level visited. First time we reach a new level = left view node.

```
FUNCTION leftViewDFS(root):
    result ← []
    maxLevel ← -1
    
    FUNCTION dfs(node, level):
        IF node == NULL: RETURN
        
        IF level > maxLevel:
            result.append(node.data)
            maxLevel ← level
        
        dfs(node.left, level + 1)      // Left FIRST
        dfs(node.right, level + 1)
    
    dfs(root, 0)
    RETURN result
```

### 🔍 Trace: Left View DFS

```
              1
            /   \
           2     3
          / \     \
         4   5     7

    dfs(1, 0): level 0 > maxLevel -1 → add 1, maxLevel=0
    dfs(2, 1): level 1 > 0 → add 2, maxLevel=1
    dfs(4, 2): level 2 > 1 → add 4, maxLevel=2
    dfs(5, 2): level 2 = 2, NOT > → skip
    dfs(3, 1): level 1 = 1, NOT > → skip
    dfs(7, 2): level 2 = 2, NOT > → skip
    
    Left View: [1, 2, 4]
```

---

### ❓ Revision Question

**In Left View using DFS, why do we visit the left child before the right child?**
<details><summary>Answer</summary>Because we want the first node at each level. By visiting left first, the leftmost node at each depth is encountered first and recorded. If we visited right first, we'd get the Right View instead.</details>

---

[← Previous: Nodes at Distance K](../08-Path-Problems/06-nodes-at-distance-k.md) | [Back to TOC](../README.md) | [Next: Right View →](02-right-view.md)
