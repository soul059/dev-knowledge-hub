# Number of Islands

[← Previous: Connected Components](01-connected-components.md) | [Back to TOC](../README.md) | [Next: Flood Fill →](03-flood-fill.md)

---

## Problem

Given a 2D grid of `'1'` (land) and `'0'` (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.

## Approach: Grid DFS

```
    Key Insight: This is just "Connected Components" on a grid!
    Each island = one connected component of '1's.

FUNCTION numIslands(grid):
    count = 0
    
    FOR each cell (r, c) in grid:
        IF grid[r][c] == '1':
            DFS_Sink(grid, r, c)     // sink the entire island
            count = count + 1         // found one island
    
    RETURN count

FUNCTION DFS_Sink(grid, r, c):
    // Boundary and validity checks
    IF r < 0 OR r >= rows OR c < 0 OR c >= cols:
        RETURN
    IF grid[r][c] != '1':
        RETURN
    
    grid[r][c] = '0'                  // sink this cell (mark visited)
    
    DFS_Sink(grid, r-1, c)            // up
    DFS_Sink(grid, r+1, c)            // down
    DFS_Sink(grid, r, c-1)            // left
    DFS_Sink(grid, r, c+1)            // right
```

## Step-by-Step Trace

```
    Initial Grid:            After DFS from (0,0):
    ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
    │ 1 │ 1 │ 0 │ 0 │       │ 0 │ 0 │ 0 │ 0 │  ← island 1 sunk
    ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
    │ 1 │ 1 │ 0 │ 0 │       │ 0 │ 0 │ 0 │ 0 │
    ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
    │ 0 │ 0 │ 1 │ 1 │       │ 0 │ 0 │ 1 │ 1 │  ← island 2 remains
    └───┴───┴───┴───┘       └───┴───┴───┴───┘
    count = 1                 
    
    After DFS from (2,2):
    ┌───┬───┬───┬───┐
    │ 0 │ 0 │ 0 │ 0 │
    ├───┼───┼───┼───┤
    │ 0 │ 0 │ 0 │ 0 │
    ├───┼───┼───┼───┤
    │ 0 │ 0 │ 0 │ 0 │  ← island 2 sunk
    └───┴───┴───┴───┘
    count = 2
    
    Answer: 2 islands
```

## Complexity

```
    Time:  O(R × C)  — each cell visited once
    Space: O(R × C)  — recursion stack in worst case
    
    ★ Trick: We modify the grid itself ('1' → '0')
      so no extra visited array needed!
```

---

[← Previous: Connected Components](01-connected-components.md) | [Back to TOC](../README.md) | [Next: Flood Fill →](03-flood-fill.md)
