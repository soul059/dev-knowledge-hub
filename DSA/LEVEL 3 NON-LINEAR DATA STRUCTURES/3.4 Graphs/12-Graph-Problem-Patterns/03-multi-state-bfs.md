# Multi-State BFS

[← Previous: Implicit Graphs](02-implicit-graphs.md) | [Back to TOC](../README.md) | [Next: Graph Coloring →](04-graph-coloring.md)

---

## Key Idea

```
    Sometimes the "state" isn't just position — it includes
    EXTRA dimensions like remaining resources, collected keys, 
    or remaining obstacles to eliminate.
    
    State = (position, extra_info)
    
    This expands the search space but is still BFS!
    
    ┌─────────────────────────────────────────────────┐
    │  Standard BFS:  visited[row][col]               │
    │  Multi-state:   visited[row][col][extra_dim]    │
    │                                                  │
    │  Extra dimension examples:                       │
    │  • Obstacles eliminated so far (0..K)           │
    │  • Keys collected (bitmask)                     │
    │  • Steps remaining                              │
    └─────────────────────────────────────────────────┘
```

## Example: Shortest Path with Obstacle Elimination

```
    Grid with obstacles. You can eliminate at most K obstacles.
    Find shortest path from (0,0) to (rows-1, cols-1).
    
    State: (row, col, remaining_eliminations)
    
    Transition:
    • Move to empty cell: (nr, nc, k)
    • Move to obstacle cell: (nr, nc, k-1) if k > 0
```

## Pseudocode

```
FUNCTION shortestPath(grid, K):
    rows = grid.length, cols = grid[0].length
    
    // visited[r][c][k] = have we visited (r,c) with k remaining?
    visited = 3D array, all false
    
    queue = [(0, 0, K, 0)]    // (row, col, remaining_K, distance)
    visited[0][0][K] = true
    
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]
    
    WHILE queue not empty:
        (r, c, k, dist) = queue.dequeue()
        
        IF r == rows-1 AND c == cols-1:
            RETURN dist
        
        FOR d = 0 to 3:
            nr = r + dr[d]
            nc = c + dc[d]
            IF NOT isValid(nr, nc, rows, cols): CONTINUE
            
            newK = k - grid[nr][nc]    // subtract 1 if obstacle
            
            IF newK >= 0 AND NOT visited[nr][nc][newK]:
                visited[nr][nc][newK] = true
                queue.enqueue((nr, nc, newK, dist + 1))
    
    RETURN -1
```

## Example: Shortest Path to Get All Keys

```
    Grid with keys (a-f) and locks (A-F).
    Key 'a' opens lock 'A'.
    Find shortest path to collect ALL keys.
    
    State: (row, col, keys_bitmask)
    
    keys_bitmask: bit i = 1 means key i is collected
    
    e.g., keys "ac" → bitmask = 101 (binary) = 5
    
    Transitions:
    • Can always move to empty cells and key cells
    • Can move to lock cell ONLY if corresponding key is held
    • Picking up key: newMask = mask | (1 << keyIndex)
    
    Target: mask with all key bits set
    
    Space: O(rows × cols × 2^numKeys)
```

---

[← Previous: Implicit Graphs](02-implicit-graphs.md) | [Back to TOC](../README.md) | [Next: Graph Coloring →](04-graph-coloring.md)
