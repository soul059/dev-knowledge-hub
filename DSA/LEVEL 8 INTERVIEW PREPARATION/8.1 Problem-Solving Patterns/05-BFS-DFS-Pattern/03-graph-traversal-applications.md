# Chapter 3: Graph Traversal Applications

## 📋 Chapter Overview
Practical applications of BFS/DFS: topological sort, shortest paths, multi-source BFS, and clone graph.

---

## 🔍 Application 1: Topological Sort

### Concept

```
  Directed Acyclic Graph (DAG) — order nodes so every edge
  goes from earlier to later.
  
  Course prerequisites:
  0 → 1 → 3
  0 → 2 → 3
  
  Valid orderings: [0, 1, 2, 3] or [0, 2, 1, 3]
```

### BFS Approach (Kahn's Algorithm)

```
function topologicalSort(numNodes, edges):
    inDegree = array of 0s, size numNodes
    graph = adjacency list from edges
    
    for each edge (u → v):
        inDegree[v] += 1
    
    queue = new Queue()
    for node = 0 to numNodes-1:
        if inDegree[node] == 0:
            queue.enqueue(node)
    
    order = []
    while queue is not empty:
        node = queue.dequeue()
        order.append(node)
        
        for neighbor in graph[node]:
            inDegree[neighbor] -= 1
            if inDegree[neighbor] == 0:
                queue.enqueue(neighbor)
    
    if len(order) == numNodes:
        return order              // valid topological order
    else:
        return []                 // cycle detected!
```

### Trace

```
  Edges: 0→1, 0→2, 1→3, 2→3
  inDegree: [0:0, 1:1, 2:1, 3:2]
  
  Queue starts with: [0]
```

| Step | Dequeue | Process | inDegree update | Enqueue | Order |
|------|---------|---------|-----------------|---------|-------|
| 1 | 0 | neighbors 1,2 | 1→0, 2→0 | 1, 2 | [0] |
| 2 | 1 | neighbor 3 | 3→1 | - | [0,1] |
| 3 | 2 | neighbor 3 | 3→0 | 3 | [0,1,2] |
| 4 | 3 | no neighbors | - | - | [0,1,2,3] ✓ |

### DFS Approach (Reverse Post-order)

```
function topologicalSortDFS(numNodes, graph):
    visited = new Set()
    stack = []                    // result stack
    
    for node = 0 to numNodes-1:
        if node not in visited:
            dfs(graph, node, visited, stack)
    
    return reverse(stack)

function dfs(graph, node, visited, stack):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited, stack)
    stack.push(node)              // post-order: push AFTER processing children
```

---

## 🔍 Application 2: Multi-Source BFS

### Concept

```
  Start BFS from MULTIPLE sources simultaneously.
  All sources are at distance 0.
  
  Use case: "Find distance from each cell to nearest X"
```

### 01 Matrix (LeetCode 542)

**Given binary matrix, find distance of each cell to nearest 0.**

```
  Input:        Output:
  0 0 0         0 0 0
  0 1 0    →    0 1 0
  1 1 1         1 2 1
```

```
function updateMatrix(matrix):
    rows = len(matrix), cols = len(matrix[0])
    queue = new Queue()
    
    // Step 1: enqueue ALL 0s, mark 1s as unvisited
    for r = 0 to rows-1:
        for c = 0 to cols-1:
            if matrix[r][c] == 0:
                queue.enqueue((r, c))
            else:
                matrix[r][c] = INF    // unvisited
    
    // Step 2: BFS from all 0s simultaneously
    directions = [(-1,0), (1,0), (0,-1), (0,1)]
    
    while queue is not empty:
        (r, c) = queue.dequeue()
        for (dr, dc) in directions:
            nr, nc = r + dr, c + dc
            if inBounds(nr, nc) AND matrix[nr][nc] > matrix[r][c] + 1:
                matrix[nr][nc] = matrix[r][c] + 1
                queue.enqueue((nr, nc))
    
    return matrix
```

---

## 🔍 Application 3: Shortest Path Variants

### Shortest Path in Binary Matrix

```
  Grid of 0s and 1s. Find shortest path from top-left to
  bottom-right moving 8-directionally through 0s only.
```

```
function shortestPathBinaryMatrix(grid):
    n = len(grid)
    if grid[0][0] != 0 OR grid[n-1][n-1] != 0: return -1
    
    queue = new Queue()
    queue.enqueue((0, 0, 1))         // (row, col, distance)
    grid[0][0] = 1                    // mark visited
    
    dirs = [(-1,-1),(-1,0),(-1,1),(0,-1),(0,1),(1,-1),(1,0),(1,1)]
    
    while queue is not empty:
        (r, c, dist) = queue.dequeue()
        
        if r == n-1 AND c == n-1:
            return dist
        
        for (dr, dc) in dirs:
            nr, nc = r + dr, c + dc
            if inBounds(nr, nc) AND grid[nr][nc] == 0:
                grid[nr][nc] = 1
                queue.enqueue((nr, nc, dist + 1))
    
    return -1
```

---

## 🔍 Application 4: Clone Graph

```
function cloneGraph(node):
    if node is null: return null
    
    map = new HashMap()             // original → clone
    queue = new Queue()
    
    map[node] = new Node(node.val)
    queue.enqueue(node)
    
    while queue is not empty:
        curr = queue.dequeue()
        for neighbor in curr.neighbors:
            if neighbor not in map:
                map[neighbor] = new Node(neighbor.val)
                queue.enqueue(neighbor)
            map[curr].neighbors.add(map[neighbor])
    
    return map[node]
```

---

## 🔍 Application 5: Word Ladder

**Transform one word to another, changing one letter at a time, each intermediate word must be in dictionary.**

```
  beginWord = "hit", endWord = "cog"
  wordList = ["hot","dot","dog","lot","log","cog"]
  
  hit → hot → dot → dog → cog  (length 5)
```

```
function ladderLength(beginWord, endWord, wordList):
    wordSet = Set(wordList)
    if endWord not in wordSet: return 0
    
    queue = new Queue()
    queue.enqueue((beginWord, 1))
    visited = Set([beginWord])
    
    while queue is not empty:
        (word, level) = queue.dequeue()
        
        for i = 0 to len(word)-1:
            for c = 'a' to 'z':
                newWord = word[:i] + c + word[i+1:]
                if newWord == endWord: return level + 1
                if newWord in wordSet AND newWord not in visited:
                    visited.add(newWord)
                    queue.enqueue((newWord, level + 1))
    
    return 0
```

---

## 📋 Summary Table

| Application | Algorithm | Key Idea |
|-------------|-----------|----------|
| Topological sort | BFS (Kahn's) or DFS | Process zero-indegree nodes first |
| Multi-source BFS | BFS | Start with all sources in queue |
| Shortest path (grid) | BFS | Level = distance |
| Clone graph | BFS + HashMap | Map original → clone |
| Word ladder | BFS | Each word is a node, single-char change is edge |

---

## ❓ Revision Questions

1. **When does topological sort fail?** → When the graph has a cycle (not a DAG).
2. **Kahn's vs DFS for topological sort?** → Kahn's (BFS) naturally detects cycles (order length ≠ n); DFS gives reverse post-order.
3. **Why multi-source BFS?** → To compute distances from ALL sources simultaneously in O(V+E).
4. **Clone graph: why use a HashMap?** → To map each original node to its clone and avoid re-creating nodes.
5. **Word Ladder: why BFS not DFS?** → BFS finds shortest transformation sequence.

---

[← Previous: DFS Fundamentals](02-dfs-fundamentals.md) | [Next: Tree BFS/DFS Problems →](04-tree-bfs-dfs-problems.md)
