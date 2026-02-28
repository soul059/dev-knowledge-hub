# Chapter 5: Classic BFS/DFS Problems

## 📋 Chapter Overview
High-frequency interview problems combining BFS/DFS with other techniques.

---

## 🔍 Problem 1: Surrounded Regions (LeetCode 130)

**Capture all 'O' regions not connected to the border.**

```
  Input:              Output:
  X X X X             X X X X
  X O O X       →     X X X X
  X X O X             X X X X
  X O X X             X O X X
```

### Key Insight
Instead of finding surrounded regions, find **un-surrounded** regions (connected to border) and flip everything else.

```
function solve(board):
    // Step 1: DFS from all border 'O's, mark as 'S' (safe)
    for each border cell:
        if cell == 'O': dfs(board, r, c)   // mark as 'S'
    
    // Step 2: Flip remaining 'O' to 'X' (captured)
    //         Flip 'S' back to 'O' (safe)
    for each cell:
        if cell == 'O': cell = 'X'
        if cell == 'S': cell = 'O'

function dfs(board, r, c):
    if out of bounds OR board[r][c] != 'O': return
    board[r][c] = 'S'
    dfs(board, r-1, c)
    dfs(board, r+1, c)
    dfs(board, r, c-1)
    dfs(board, r, c+1)
```

---

## 🔍 Problem 2: Rotting Oranges (LeetCode 994)

**Multi-source BFS. Fresh oranges rot from adjacent rotten ones each minute.**

```
  0 = empty, 1 = fresh, 2 = rotten
  
  Input:           After 1 min:     After 2 min:
  2 1 1            2 2 1            2 2 2
  1 1 0      →     2 2 0      →     2 2 0
  0 1 1            0 1 1            0 2 2
  
  Answer: 4 minutes (if all become rotten)
```

```
function orangesRotting(grid):
    queue = new Queue()
    fresh = 0
    
    // Enqueue all rotten oranges
    for r, c in grid:
        if grid[r][c] == 2: queue.enqueue((r, c))
        if grid[r][c] == 1: fresh += 1
    
    if fresh == 0: return 0
    
    minutes = 0
    dirs = [(-1,0),(1,0),(0,-1),(0,1)]
    
    while queue is not empty:
        levelSize = queue.size()
        for i = 0 to levelSize-1:
            (r, c) = queue.dequeue()
            for (dr, dc) in dirs:
                nr, nc = r + dr, c + dc
                if inBounds(nr, nc) AND grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    queue.enqueue((nr, nc))
        if queue is not empty: minutes += 1
    
    return minutes if fresh == 0 else -1
```

---

## 🔍 Problem 3: Course Schedule (LeetCode 207)

**Can you finish all courses? (Cycle detection in directed graph)**

```
function canFinish(numCourses, prerequisites):
    graph = build adjacency list
    inDegree = compute in-degrees
    
    queue = [all nodes with inDegree == 0]
    count = 0
    
    while queue is not empty:
        node = queue.dequeue()
        count += 1
        for neighbor in graph[node]:
            inDegree[neighbor] -= 1
            if inDegree[neighbor] == 0:
                queue.enqueue(neighbor)
    
    return count == numCourses    // true if no cycle
```

### Trace: numCourses=4, prereqs=[[1,0],[2,0],[3,1],[3,2]]

```
  Graph: 0→1, 0→2, 1→3, 2→3
  inDegree: [0:0, 1:1, 2:1, 3:2]
  
  Queue: [0] → process 0, reduce 1→0, 2→0
  Queue: [1, 2] → process 1, reduce 3→1; process 2, reduce 3→0
  Queue: [3] → process 3
  count = 4 == numCourses → true ✓
```

---

## 🔍 Problem 4: Pacific Atlantic Water Flow (LeetCode 417)

**Find cells where water can flow to both Pacific and Atlantic oceans.**

```
  Pacific ~                Atlantic
  ~  1  2  2  3  5  ~
  ~  3  2  3  4  4  ~
  ~  2  4  5  3  1  ~
  ~  6  7  1  4  5  ~
  ~  5  1  1  2  4  ~
           ~ Atlantic ~
```

### Key Insight: Reverse DFS from oceans inward

```
function pacificAtlantic(heights):
    pacific = new Set()
    atlantic = new Set()
    
    // DFS from top row and left col (Pacific)
    for each cell on top row or left col:
        dfs(heights, r, c, pacific, -INF)
    
    // DFS from bottom row and right col (Atlantic)
    for each cell on bottom row or right col:
        dfs(heights, r, c, atlantic, -INF)
    
    return intersection(pacific, atlantic)

function dfs(heights, r, c, reachable, prevHeight):
    if out of bounds: return
    if (r,c) in reachable: return
    if heights[r][c] < prevHeight: return    // water can't flow uphill
    
    reachable.add((r, c))
    dfs(heights, r-1, c, reachable, heights[r][c])
    dfs(heights, r+1, c, reachable, heights[r][c])
    dfs(heights, r, c-1, reachable, heights[r][c])
    dfs(heights, r, c+1, reachable, heights[r][c])
```

---

## 🔍 Problem 5: Walls and Gates (LeetCode 286)

**Multi-source BFS from all gates to fill shortest distances.**

```
  INF = 2147483647 (empty room), -1 = wall, 0 = gate
  
  Input:              Output:
  INF -1   0  INF     3  -1  0   1
  INF INF INF  -1  →  2   2  1  -1
  INF -1  INF  -1     1  -1  2  -1
   0  -1  INF INF     0  -1  3   4
```

```
function wallsAndGates(rooms):
    queue = new Queue()
    
    // Enqueue all gates
    for r, c in rooms:
        if rooms[r][c] == 0:
            queue.enqueue((r, c))
    
    dirs = [(-1,0),(1,0),(0,-1),(0,1)]
    
    while queue is not empty:
        (r, c) = queue.dequeue()
        for (dr, dc) in dirs:
            nr, nc = r + dr, c + dc
            if inBounds(nr, nc) AND rooms[nr][nc] == INF:
                rooms[nr][nc] = rooms[r][c] + 1
                queue.enqueue((nr, nc))
```

---

## 📊 Pattern Classification

| Pattern | Problems | Key Technique |
|---------|----------|---------------|
| Multi-source BFS | Rotting oranges, walls & gates | All sources start in queue |
| Border DFS | Surrounded regions, Pacific Atlantic | Start from edges inward |
| Topological order | Course schedule | Kahn's / in-degree BFS |
| Grid traversal | Islands, shortest path | 4-dir BFS/DFS |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Surrounded regions | Reverse thinking — mark safe from border |
| Rotting oranges | Multi-source BFS, count levels = minutes |
| Course schedule | Topological sort detects cycles |
| Pacific Atlantic | DFS from ocean inward, find intersection |
| Walls and gates | Multi-source BFS from all gates |

---

## ❓ Revision Questions

1. **Surrounded Regions: why DFS from border?** → Easier to find regions connected to border (safe), then capture everything else.
2. **Rotting Oranges: why multi-source BFS?** → All rotten oranges spread simultaneously; BFS levels = time steps.
3. **Course Schedule: how to detect cycle?** → If topological sort doesn't include all nodes, there's a cycle.
4. **Pacific Atlantic: why DFS from ocean?** → DFS from each cell is O(mn) per cell. From ocean edges, each cell visited at most twice (once per ocean).
5. **Walls and Gates: why BFS not DFS?** → BFS guarantees shortest distance from any gate.

---

[← Previous: Tree BFS/DFS Problems](04-tree-bfs-dfs-problems.md) | [Next: When to Apply →](06-when-to-apply.md)
