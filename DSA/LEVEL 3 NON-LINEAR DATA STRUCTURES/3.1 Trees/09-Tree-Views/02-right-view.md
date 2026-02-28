# Unit 9 – Topic 2: Right View

[← Previous: Left View](01-left-view.md) | [Back to TOC](../README.md) | [Next: Top View →](03-top-view.md)

---

## 9.2 Right View

### 📖 Problem

Print the nodes visible from the **right side** — the last node at each level.

```
              1    ← visible (level 0)
            /   \
           2     3    →            (level 1: see 3)
          / \   / \
         4   5 6   7    →         (level 2: see 7)
            / \
           8   9          →       (level 3: see 9)

    Right View: [1, 3, 7, 9]
```

### BFS Approach

```
FUNCTION rightView(root):
    IF root == NULL: RETURN []
    
    result ← []
    queue ← [root]
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            
            IF i == levelSize - 1:         // LAST node of level
                result.append(node.data)
            
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
    
    RETURN result
```

### DFS Approach

Same as left view, but visit **right child first**.

```
FUNCTION rightViewDFS(root):
    result ← []
    maxLevel ← -1
    
    FUNCTION dfs(node, level):
        IF node == NULL: RETURN
        
        IF level > maxLevel:
            result.append(node.data)
            maxLevel ← level
        
        dfs(node.right, level + 1)     // Right FIRST
        dfs(node.left, level + 1)
    
    dfs(root, 0)
    RETURN result
```

### 💡 Key Insight: Left vs Right View

```
    Left View DFS:  visit left first  → first node at each level
    Right View DFS: visit right first → first node at each level (from right)
    
    Left View BFS:  first in level  (i == 0)
    Right View BFS: last in level   (i == levelSize - 1)
```

---

[← Previous: Left View](01-left-view.md) | [Back to TOC](../README.md) | [Next: Top View →](03-top-view.md)
