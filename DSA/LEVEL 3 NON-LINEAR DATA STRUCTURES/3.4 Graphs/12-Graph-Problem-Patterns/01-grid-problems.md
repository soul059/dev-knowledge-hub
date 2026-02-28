# Grid Problems

[← Previous Unit: Condensation Graph](../11-Advanced-Graph-Algorithms/07-condensation-graph.md) | [Back to TOC](../README.md) | [Next: Implicit Graphs →](02-implicit-graphs.md)

---

## Grid = Implicit Graph

```
    A 2D grid is a graph where:
    • Each cell (r, c) is a vertex
    • Neighbors are adjacent cells (4- or 8-directional)
    • Edges are IMPLICIT (no adjacency list needed!)
    
    ┌───┬───┬───┐        Vertex: (r,c)
    │0,0│0,1│0,2│        Neighbors of (1,1):
    ├───┼───┼───┤         ↑ (0,1)
    │1,0│1,1│1,2│         ↓ (2,1)
    ├───┼───┼───┤         ← (1,0)
    │2,0│2,1│2,2│         → (1,2)
    └───┴───┴───┘
```

## Standard Direction Arrays

```
    // 4-directional
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]
    
    // 8-directional (includes diagonals)
    dr = [-1, -1, -1, 0, 0, 1, 1, 1]
    dc = [-1, 0, 1, -1, 1, -1, 0, 1]
    
    // Bounds check
    FUNCTION isValid(r, c, rows, cols):
        RETURN 0 <= r < rows AND 0 <= c < cols
```

## Common Grid Problem Patterns

```
    ┌─────────────────────────────────────────────────┐
    │  Pattern              │  Algorithm              │
    ├───────────────────────┼─────────────────────────┤
    │  Count islands        │  DFS/BFS + mark visited │
    │  Shortest path        │  BFS (unweighted)       │
    │  Flood fill           │  DFS from start cell    │
    │  Connected region     │  DFS/BFS traversal      │
    │  Surrounded regions   │  DFS from border        │
    │  Rotten oranges       │  Multi-source BFS       │
    │  Maze solving         │  BFS for shortest path  │
    │  A* pathfinding       │  BFS + heuristic        │
    └───────────────────────┴─────────────────────────┘
```

## Template: Grid BFS

```
FUNCTION gridBFS(grid, startR, startC):
    rows = grid.length
    cols = grid[0].length
    
    visited = 2D array of booleans, all false
    queue = [(startR, startC)]
    visited[startR][startC] = true
    
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]
    
    WHILE queue not empty:
        (r, c) = queue.dequeue()
        // Process cell (r, c)
        
        FOR d = 0 to 3:
            nr = r + dr[d]
            nc = c + dc[d]
            IF isValid(nr, nc, rows, cols) 
               AND NOT visited[nr][nc]
               AND grid[nr][nc] meets condition:
                visited[nr][nc] = true
                queue.enqueue((nr, nc))
```

---

[← Previous Unit: Condensation Graph](../11-Advanced-Graph-Algorithms/07-condensation-graph.md) | [Back to TOC](../README.md) | [Next: Implicit Graphs →](02-implicit-graphs.md)
