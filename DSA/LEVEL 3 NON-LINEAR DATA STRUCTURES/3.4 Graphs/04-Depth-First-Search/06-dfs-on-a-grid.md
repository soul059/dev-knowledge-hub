# DFS on a Grid

[← Previous: DFS Tree and Backtracking](05-dfs-tree-and-backtracking.md) | [Back to TOC](../README.md) | [Next: Applications and Patterns →](07-applications-and-patterns.md)

---

## Grid DFS Pseudocode

```
FUNCTION DFS_Grid(grid, row, col, visited):
    // Boundary and validity checks
    IF row < 0 OR row >= rows OR col < 0 OR col >= cols:
        RETURN
    IF visited[row][col] OR grid[row][col] == WALL:
        RETURN
    
    visited[row][col] = true
    PROCESS(row, col)
    
    // Explore 4 directions
    DFS_Grid(grid, row - 1, col, visited)   // up
    DFS_Grid(grid, row + 1, col, visited)   // down
    DFS_Grid(grid, row, col - 1, visited)   // left
    DFS_Grid(grid, row, col + 1, visited)   // right
```

## Grid DFS Trace

```
    Grid (1 = land, 0 = water):
    ┌───┬───┬───┬───┐
    │ 1 │ 1 │ 0 │ 0 │
    ├───┼───┼───┼───┤
    │ 1 │ 1 │ 0 │ 0 │
    ├───┼───┼───┼───┤
    │ 0 │ 0 │ 1 │ 1 │
    └───┴───┴───┴───┘

    DFS from (0,0):
    Visit (0,0) → (0,1) → (1,1) → (1,0) → backtrack all
    
    Visited after DFS from (0,0):
    ┌───┬───┬───┬───┐
    │ ✓ │ ✓ │   │   │
    ├───┼───┼───┼───┤
    │ ✓ │ ✓ │   │   │
    ├───┼───┼───┼───┤
    │   │   │   │   │
    └───┴───┴───┴───┘
    
    This found one "island" of connected 1s.
```

---

[← Previous: DFS Tree and Backtracking](05-dfs-tree-and-backtracking.md) | [Back to TOC](../README.md) | [Next: Applications and Patterns →](07-applications-and-patterns.md)
