# Chapter 1: BFS Fundamentals

## 📋 Chapter Overview
Breadth-First Search explores a graph **level by level**, using a **queue**. It's the go-to algorithm for **shortest path in unweighted graphs** and **level-order traversal** in trees.

---

## 🧠 Core Concept

### How BFS Works

```
  Start at source node, explore ALL neighbors first,
  then neighbors' neighbors, and so on.

  Graph:          BFS Order (from A):
      A               Level 0: A
     / \               Level 1: B, C
    B   C              Level 2: D, E, F
   / \   \             Level 3: G
  D   E   F
  |
  G

  Visit order: A → B → C → D → E → F → G
```

### BFS vs DFS at a Glance

```
  BFS (Queue - FIFO):        DFS (Stack - LIFO):
  
  Explores WIDE first        Explores DEEP first
  ┌─┬─┬─┬─┬─┐               │
  │1│2│3│4│5│               1─2─3─4─5
  └─┴─┴─┴─┴─┘                    │
       ↓                          6─7
  ┌─┬─┬─┬─┬─┐
  │6│7│8│9│10│
  └─┴─┴─┴─┴──┘
```

---

## 📝 BFS Template

```
function bfs(graph, source):
    queue = new Queue()
    visited = new Set()
    
    queue.enqueue(source)
    visited.add(source)
    
    while queue is not empty:
        node = queue.dequeue()
        process(node)                    // do work here
        
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)    // mark BEFORE enqueue
                queue.enqueue(neighbor)
```

### Why Mark Before Enqueue?

```
  If we mark after dequeue:
  
      A
     / \
    B   C        B and C both add D to queue!
     \ /         D gets processed TWICE.
      D
  
  Mark before enqueue → D only added once ✓
```

---

## 📝 Level-by-Level BFS Template

```
function bfsLevelOrder(graph, source):
    queue = new Queue()
    visited = new Set()
    level = 0
    
    queue.enqueue(source)
    visited.add(source)
    
    while queue is not empty:
        levelSize = queue.size()         // snapshot current level
        
        for i = 0 to levelSize - 1:
            node = queue.dequeue()
            process(node, level)
            
            for neighbor in graph[node]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.enqueue(neighbor)
        
        level += 1
```

---

## 🔍 Detailed Trace: BFS on Graph

```
  Graph adjacency list:
  A → [B, C]
  B → [A, D, E]
  C → [A, F]
  D → [B]
  E → [B, F]
  F → [C, E]
```

### Step-by-Step

| Step | Queue | Dequeue | Visited | Enqueue |
|------|-------|---------|---------|---------|
| Init | [A] | - | {A} | - |
| 1 | [] | A | {A} | B, C |
| 2 | [B, C] | B | {A,B,C} | D, E |
| 3 | [C, D, E] | C | {A,B,C,D,E} | F |
| 4 | [D, E, F] | D | {A,B,C,D,E,F} | (none new) |
| 5 | [E, F] | E | {A,B,C,D,E,F} | (F already visited) |
| 6 | [F] | F | {A,B,C,D,E,F} | (all visited) |
| 7 | [] | - | - | **DONE** |

**BFS order: A → B → C → D → E → F**

---

## 🔍 BFS on Trees: Level-Order Traversal

```
  Binary Tree:
        1
       / \
      2   3
     / \   \
    4   5   6

  Level 0: [1]
  Level 1: [2, 3]
  Level 2: [4, 5, 6]
  
  Output: [[1], [2, 3], [4, 5, 6]]
```

### Solution

```
function levelOrder(root):
    if root is null: return []
    
    result = []
    queue = new Queue()
    queue.enqueue(root)
    
    while queue is not empty:
        levelSize = queue.size()
        currentLevel = []
        
        for i = 0 to levelSize - 1:
            node = queue.dequeue()
            currentLevel.append(node.val)
            
            if node.left:  queue.enqueue(node.left)
            if node.right: queue.enqueue(node.right)
        
        result.append(currentLevel)
    
    return result
```

### Trace

| Level | Queue Before | Process | Queue After |
|-------|-------------|---------|-------------|
| 0 | [1] | Dequeue 1 | [2, 3] |
| 1 | [2, 3] | Dequeue 2, 3 | [4, 5, 6] |
| 2 | [4, 5, 6] | Dequeue 4, 5, 6 | [] |

---

## 🔍 Shortest Path in Unweighted Graph

### Why BFS Guarantees Shortest Path

```
  BFS explores level by level.
  Level = distance from source.
  First time we reach a node = shortest path.

  Source: A     Target: F

  Level 0: A
  Level 1: B, C        ← 1 edge from A
  Level 2: D, E, F     ← 2 edges from A
  
  Shortest path A → F = 2 edges ✓
```

### With Path Tracking

```
function shortestPath(graph, source, target):
    queue = new Queue()
    visited = new Set()
    parent = new Map()
    
    queue.enqueue(source)
    visited.add(source)
    parent[source] = null
    
    while queue is not empty:
        node = queue.dequeue()
        
        if node == target:
            return reconstructPath(parent, target)
        
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                parent[neighbor] = node
                queue.enqueue(neighbor)
    
    return null                // no path

function reconstructPath(parent, target):
    path = []
    node = target
    while node is not null:
        path.prepend(node)
        node = parent[node]
    return path
```

---

## 🔍 BFS on Grid (2D Matrix)

### Template

```
function bfsGrid(grid, startRow, startCol):
    rows = len(grid)
    cols = len(grid[0])
    directions = [(-1,0), (1,0), (0,-1), (0,1)]    // up, down, left, right
    
    queue = new Queue()
    visited = boolean[rows][cols]
    
    queue.enqueue((startRow, startCol))
    visited[startRow][startCol] = true
    
    while queue is not empty:
        (r, c) = queue.dequeue()
        process(r, c)
        
        for (dr, dc) in directions:
            nr = r + dr
            nc = c + dc
            if 0 <= nr < rows AND 0 <= nc < cols
               AND not visited[nr][nc]
               AND grid[nr][nc] is valid:
                visited[nr][nc] = true
                queue.enqueue((nr, nc))
```

### Common Grid Direction Arrays

```
  4-directional: [(-1,0), (1,0), (0,-1), (0,1)]
  
  8-directional: [(-1,-1), (-1,0), (-1,1),
                  ( 0,-1),         ( 0,1),
                  ( 1,-1), ( 1,0), ( 1,1)]
```

---

## 📊 Complexity Analysis

| Aspect | Graph | Grid |
|--------|-------|------|
| Time | O(V + E) | O(rows × cols) |
| Space | O(V) queue + visited | O(rows × cols) |
| V | Number of vertices | rows × cols |
| E | Number of edges | ~4 × rows × cols |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Data structure | Queue (FIFO) |
| Exploration order | Level by level (breadth first) |
| Mark visited | BEFORE enqueue (not after dequeue) |
| Level tracking | Snapshot `queue.size()` at start of each level |
| Shortest path | Guaranteed for unweighted graphs |
| Grid BFS | Use direction arrays, check bounds |
| Time | O(V + E) |

---

## ❓ Revision Questions

1. **What data structure does BFS use?**
   → Queue (FIFO).

2. **Why does BFS find shortest path in unweighted graphs?**
   → It explores all nodes at distance d before distance d+1.

3. **Why mark visited before enqueue, not after dequeue?**
   → Prevents the same node being enqueued multiple times by different parents.

4. **How do you track levels in BFS?**
   → At each iteration, capture `levelSize = queue.size()`, process exactly that many nodes.

5. **BFS time complexity on a grid of m×n?**
   → O(m × n) — each cell visited at most once.

---

[← Back to README](../README.md) | [Next: DFS Fundamentals →](02-dfs-fundamentals.md)
