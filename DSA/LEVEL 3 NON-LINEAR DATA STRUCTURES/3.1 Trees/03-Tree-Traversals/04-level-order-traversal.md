# 3.4 Level Order Traversal (BFS)

[← Previous: Postorder Traversal](03-postorder-traversal.md) | [Back to TOC](../README.md) | [Next: Morris Traversal →](05-morris-traversal.md)

---

## 📖 Concept

Visit nodes **level by level**, from top to bottom, left to right. Uses a **queue** (FIFO).

```
            1          Level 0: [1]
           / \
          2    3       Level 1: [2, 3]
         / \    \
        4   5    6     Level 2: [4, 5, 6]
           / \
          7   8        Level 3: [7, 8]

    Level Order: 1, 2, 3, 4, 5, 6, 7, 8
```

---

## Algorithm

```
FUNCTION levelOrder(root):
    IF root == NULL:
        RETURN
    
    queue ← empty queue
    queue.enqueue(root)
    
    WHILE queue is not empty:
        node ← queue.dequeue()
        PRINT node.data
        
        IF node.left ≠ NULL:
            queue.enqueue(node.left)
        IF node.right ≠ NULL:
            queue.enqueue(node.right)
```

---

## Level-by-Level Variant (Returns 2D List)

```
FUNCTION levelOrderByLevel(root):
    IF root == NULL:
        RETURN []
    
    result ← []
    queue ← empty queue
    queue.enqueue(root)
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        currentLevel ← []
        
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            currentLevel.append(node.data)
            
            IF node.left ≠ NULL:
                queue.enqueue(node.left)
            IF node.right ≠ NULL:
                queue.enqueue(node.right)
        
        result.append(currentLevel)
    
    RETURN result
```

---

## 🔍 Trace: Level Order

```
            1
           / \
          2    3
         / \    \
        4   5    6

    Step | Queue (front→)  | Dequeue | Enqueue      | Output
    -----+-----------------+---------+--------------+---------
      1  | [1]             |         |              |
      2  | [2, 3]          | 1       | L=2, R=3     | 1
      3  | [3, 4, 5]       | 2       | L=4, R=5     | 1,2
      4  | [4, 5, 6]       | 3       | R=6          | 1,2,3
      5  | [5, 6]          | 4       | (no children)| 1,2,3,4
      6  | [6]             | 5       | (no children)| 1,2,3,4,5
      7  | []              | 6       | (no children)| 1,2,3,4,5,6
    
    Result: 1, 2, 3, 4, 5, 6
```

---

## ⏱️ Complexity

| Aspect | Value |
|--------|-------|
| Time | O(n) — visit every node once |
| Space | O(w) — where w is maximum width of tree |
| Max width | Up to n/2 for a complete binary tree's last level |

---

[← Previous: Postorder Traversal](03-postorder-traversal.md) | [Back to TOC](../README.md) | [Next: Morris Traversal →](05-morris-traversal.md)
