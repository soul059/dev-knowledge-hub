# Unit 9 – Topic 4: Bottom View

[← Previous: Top View](03-top-view.md) | [Back to TOC](../README.md) | [Next: Boundary Traversal →](05-boundary-traversal.md)

---

## 9.4 Bottom View

### 📖 Problem

Print nodes visible from **below** — for each horizontal distance, the bottommost (last encountered in BFS) node.

```
            1(HD=0)
           /     \
      (HD=-1)2   3(HD=1)
          / \   / \
    (HD=-2)4 5(0) 6(0) 7(HD=2)

    HD:  -2  -1   0   1   2
         4    2   5   6   7     ← wait, 6 or 5?
                   ↑
              Both at HD=0, but 5 is left and 6 is right
              In BFS, if 5 comes before 6, last one wins → 6
              (or 5 depending on exact position)
    
    Bottom View: [4, 2, 6, 3, 7]  (for this specific tree,
                                    6 overwrites 5 at HD=0)
```

### Algorithm

```
FUNCTION bottomView(root):
    IF root == NULL: RETURN []
    
    map ← TreeMap()
    queue ← [(root, 0)]
    
    WHILE queue is not empty:
        (node, hd) ← queue.dequeue()
        
        // OVERWRITE with latest node at each HD
        map[hd] ← node.data
        
        IF node.left: queue.enqueue((node.left, hd - 1))
        IF node.right: queue.enqueue((node.right, hd + 1))
    
    RETURN map values sorted by HD
```

### 💡 Top View vs Bottom View

```
    The ONLY difference:
    
    Top View:    IF hd NOT in map: map[hd] = node  (FIRST wins)
    Bottom View: map[hd] = node                     (LAST wins / overwrite)
```

---

### ❓ Revision Question

**What is the only difference between Top View and Bottom View algorithms?**
<details><summary>Answer</summary>Top View keeps the FIRST node encountered at each horizontal distance (if HD not in map, add it). Bottom View OVERWRITES with every node at each HD, keeping the last one (deepest). Both use BFS with horizontal distance tracking.</details>

---

[← Previous: Top View](03-top-view.md) | [Back to TOC](../README.md) | [Next: Boundary Traversal →](05-boundary-traversal.md)
