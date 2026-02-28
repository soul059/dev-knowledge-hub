# Unit 9 – Topic 6: Zigzag (Spiral) Traversal

[← Previous: Boundary Traversal](05-boundary-traversal.md) | [Back to TOC](../README.md) | [Next: Advanced Tree Concepts →](../10-Advanced-Tree-Concepts/01-threaded-binary-tree.md)

---

## 9.6 Zigzag (Spiral) Traversal

### 📖 Problem

Level-order traversal but alternating direction: left-to-right on even levels, right-to-left on odd levels (or vice versa).

```
              1             → left to right
            /   \
           2     3          ← right to left
          / \   / \
         4   5 6   7        → left to right
            / \
           8   9             ← right to left

    Zigzag: [[1], [3, 2], [4, 5, 6, 7], [9, 8]]
```

### Approach 1: BFS with Direction Flag

```
FUNCTION zigzagTraversal(root):
    IF root == NULL: RETURN []
    
    result ← []
    queue ← [root]
    leftToRight ← true
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        currentLevel ← []
        
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            currentLevel.append(node.data)
            
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
        
        IF NOT leftToRight:
            REVERSE(currentLevel)
        
        result.append(currentLevel)
        leftToRight ← NOT leftToRight
    
    RETURN result
```

### Approach 2: Using Deque (No Reversal)

```
FUNCTION zigzagDeque(root):
    IF root == NULL: RETURN []
    
    result ← []
    queue ← [root]
    leftToRight ← true
    
    WHILE queue is not empty:
        levelSize ← queue.size()
        currentLevel ← deque of size levelSize
        
        FOR i ← 0 to levelSize - 1:
            node ← queue.dequeue()
            
            IF leftToRight:
                currentLevel.addLast(node.data)     // Add at end
            ELSE:
                currentLevel.addFirst(node.data)    // Add at front
            
            IF node.left: queue.enqueue(node.left)
            IF node.right: queue.enqueue(node.right)
        
        result.append(currentLevel)
        leftToRight ← NOT leftToRight
    
    RETURN result
```

### 🔍 Trace

```
            1
           / \
          2   3
         / \   \
        4   5   7

    Level 0 (L→R): queue=[1], process → [1]
    Level 1 (R→L): queue=[2,3], process → [3, 2]
    Level 2 (L→R): queue=[4,5,7], process → [4, 5, 7]
    
    Result: [[1], [3, 2], [4, 5, 7]]
```

---

## 🧩 View Problems Pattern Summary

### Horizontal Distance (HD) Pattern

Used for: Top View, Bottom View, Vertical Order

```
    Rule:
    - Root has HD = 0
    - Left child: HD = parent_HD - 1
    - Right child: HD = parent_HD + 1
    
    Always use BFS (level order) for correct ordering
```

### Level-Based Pattern

Used for: Left View, Right View, Zigzag

```
    Left View:  first node at each level (DFS: visit left first)
    Right View: last node at each level  (DFS: visit right first)
    Zigzag:     alternate direction per level
```

### Boundary Pattern

Three separate traversals combined:
```
    1. Left boundary (top → down, follow left edge)
    2. Leaf nodes (left → right, standard DFS)
    3. Right boundary (bottom → up, follow right edge)
```

---

## 📝 Unit 9 Summary Table

| Concept | Key Point |
|---------|-----------|
| Left View | First node at each level (BFS: i==0, DFS: left first) |
| Right View | Last node at each level (BFS: i==last, DFS: right first) |
| Top View | First node at each HD in BFS order |
| Bottom View | Last node at each HD in BFS order |
| Boundary | Left edge + leaves + right edge (anti-clockwise) |
| Zigzag | Level order with alternating direction |
| HD concept | root=0, left: HD-1, right: HD+1 |
| Top vs Bottom | Top: first wins per HD. Bottom: last wins per HD |
| Left vs Right DFS trick | Change which child is visited first |

---

## ⏱️ Complexity Summary

| Problem | Time | Space | Data Structure |
|---------|------|-------|----------------|
| Left View | O(n) | O(h) DFS / O(w) BFS | DFS or Queue |
| Right View | O(n) | O(h) DFS / O(w) BFS | DFS or Queue |
| Top View | O(n) | O(n) | Queue + TreeMap |
| Bottom View | O(n) | O(n) | Queue + TreeMap |
| Boundary | O(n) | O(h) | DFS (3 passes) |
| Zigzag | O(n) | O(n) | Queue + Deque |

---

### ❓ Revision Questions

1. **In Zigzag traversal, what is the advantage of using a deque over reversing?**
   <details><summary>Answer</summary>Using a deque avoids the O(k) reversal step at each level (where k is level size). Instead, we add at the front or back of the deque in O(1). Overall complexity is the same O(n), but the deque approach has a smaller constant factor.</details>

2. **What horizontal distance (HD) values does a perfect binary tree of height h have?**
   <details><summary>Answer</summary>HD ranges from -h to +h. The leftmost leaf has HD = -h and the rightmost leaf has HD = +h. This means 2h+1 distinct columns in vertical/top/bottom view.</details>

---

[← Previous: Boundary Traversal](05-boundary-traversal.md) | [Back to TOC](../README.md) | [Next: Advanced Tree Concepts →](../10-Advanced-Tree-Concepts/01-threaded-binary-tree.md)
