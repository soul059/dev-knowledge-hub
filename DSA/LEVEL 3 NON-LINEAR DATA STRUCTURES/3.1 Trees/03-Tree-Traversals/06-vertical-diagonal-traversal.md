# 3.6 Vertical & Diagonal Traversal

[← Previous: Morris Traversal](05-morris-traversal.md) | [Back to TOC](../README.md) | [Next Unit: Binary Search Tree →](../04-Binary-Search-Tree/01-bst-property.md)

---

## Vertical Traversal

Group nodes by their **horizontal distance** (HD) from the root.

```
            1 (HD=0)
           / \
    (HD=-1)2   3(HD=1)
         / \    \
  (HD=-2)4  5    6(HD=2)
           (HD=0)
           
    HD = -2: [4]
    HD = -1: [2]
    HD =  0: [1, 5]
    HD =  1: [3]
    HD =  2: [6]
    
    Vertical Order: [4], [2], [1,5], [3], [6]
```

### Algorithm: Vertical Order

```
FUNCTION verticalOrder(root):
    IF root == NULL:
        RETURN []
    
    map ← empty TreeMap     // HD → list of node values
    queue ← pairs of (node, HD)
    queue.enqueue((root, 0))
    
    WHILE queue is not empty:
        (node, hd) ← queue.dequeue()
        map[hd].append(node.data)
        
        IF node.left ≠ NULL:
            queue.enqueue((node.left, hd - 1))
        IF node.right ≠ NULL:
            queue.enqueue((node.right, hd + 1))
    
    RETURN map values sorted by HD
```

---

## Diagonal Traversal

Group nodes by their **diagonal distance** from the root. Moving left increases the diagonal by 1, moving right keeps it the same.

```
            1
           / \
          2    3
         / \    \
        4   5    6

    Diagonal 0: 1, 3, 6    (keep going right)
    Diagonal 1: 2, 5       (one left turn from diagonal 0)
    Diagonal 2: 4          (two left turns)
    
    Diagonal view:
    
            1─────3─────6      ← Diagonal 0
           /     /
          2─────5               ← Diagonal 1
         /
        4                       ← Diagonal 2
```

### Algorithm: Diagonal Traversal

```
FUNCTION diagonalTraversal(root):
    IF root == NULL:
        RETURN []
    
    map ← empty map     // diagonal level → list of nodes
    queue ← pairs of (node, diagonal)
    queue.enqueue((root, 0))
    
    WHILE queue is not empty:
        (node, d) ← queue.dequeue()
        map[d].append(node.data)
        
        IF node.left ≠ NULL:
            queue.enqueue((node.left, d + 1))
        IF node.right ≠ NULL:
            queue.enqueue((node.right, d))     // Same diagonal
    
    RETURN map values sorted by diagonal
```

---

## 💡 When to Use Which Traversal?

| Traversal | Best For |
|-----------|----------|
| **Inorder** | BST → sorted order; expression trees |
| **Preorder** | Copying/cloning trees; serialization; prefix expressions |
| **Postorder** | Deleting trees; postfix expressions; bottom-up calculations |
| **Level order** | Finding level-wise info; shortest path in unweighted tree |
| **Morris** | When O(1) space is required |
| **Vertical** | Column-wise grouping |
| **Diagonal** | Diagonal grouping |

---

## DFS vs BFS Summary

```
    DFS (Depth-First Search)              BFS (Breadth-First Search)
    ─────────────────────                 ──────────────────────────
    Uses: Stack (or recursion)            Uses: Queue
    Goes: Deep before wide                Goes: Wide before deep
    Types: Inorder, Preorder,             Type: Level Order
           Postorder
    Space: O(h) where h=height           Space: O(w) where w=width
    
    Better when:                          Better when:
    • Tree is very wide                   • Tree is very deep
    • Solution is deep in tree            • Solution is near root
    • Need to explore all paths           • Need shortest path
```

---

## ⏱️ Complexity

| Traversal | Time | Space |
|-----------|------|-------|
| Vertical | O(n log n) | O(n) |
| Diagonal | O(n) | O(n) |

---

[← Previous: Morris Traversal](05-morris-traversal.md) | [Back to TOC](../README.md) | [Next Unit: Binary Search Tree →](../04-Binary-Search-Tree/01-bst-property.md)
