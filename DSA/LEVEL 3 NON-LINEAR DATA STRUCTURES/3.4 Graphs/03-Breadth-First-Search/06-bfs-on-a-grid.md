# BFS on a Grid

[← Previous: BFS Properties](05-bfs-properties.md) | [Back to TOC](../README.md) | [Next: Applications and Patterns →](07-applications-and-patterns.md)

---

Grids are implicit graphs. BFS on a grid uses the same algorithm with **direction arrays**.

## Grid BFS Pseudocode

```
FUNCTION BFS_Grid(grid, startRow, startCol):
    rows = grid.numRows
    cols = grid.numCols
    
    visited = 2D boolean array [rows][cols], all false
    visited[startRow][startCol] = true
    
    queue = new Queue()
    queue.enqueue((startRow, startCol))
    
    // Direction arrays: up, down, left, right
    dr = [-1, 1, 0, 0]
    dc = [0, 0, -1, 1]
    
    WHILE queue is not empty:
        (row, col) = queue.dequeue()
        PROCESS(row, col)
        
        FOR d = 0 to 3:                         // 4 directions
            newRow = row + dr[d]
            newCol = col + dc[d]
            
            IF newRow >= 0 AND newRow < rows
               AND newCol >= 0 AND newCol < cols
               AND NOT visited[newRow][newCol]
               AND grid[newRow][newCol] != WALL:
                
                visited[newRow][newCol] = true
                queue.enqueue((newRow, newCol))
```

## Grid BFS Trace

```
    Grid (S=start, .=open, #=wall):
    ┌───┬───┬───┬───┐
    │ S │ . │ . │ . │
    ├───┼───┼───┼───┤
    │ . │ # │ # │ . │
    ├───┼───┼───┼───┤
    │ . │ . │ . │ . │
    └───┴───┴───┴───┘

    BFS from S = (0,0), distances:
    ┌───┬───┬───┬───┐
    │ 0 │ 1 │ 2 │ 3 │
    ├───┼───┼───┼───┤
    │ 1 │ # │ # │ 4 │
    ├───┼───┼───┼───┤
    │ 2 │ 3 │ 4 │ 5 │
    └───┴───┴───┴───┘

    Shortest path from (0,0) to (2,3) = 5 steps
```

---

[← Previous: BFS Properties](05-bfs-properties.md) | [Back to TOC](../README.md) | [Next: Applications and Patterns →](07-applications-and-patterns.md)
