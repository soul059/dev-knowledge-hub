# Chapter 6: Grid Connectivity

## Overview

Many problems involve grids where cells are connected to their neighbors. Union-Find handles grid connectivity elegantly by mapping 2D coordinates to 1D indices.

---

## 2D-to-1D Mapping

```
Grid (rows × cols):

  (0,0) (0,1) (0,2) (0,3)
  (1,0) (1,1) (1,2) (1,3)
  (2,0) (2,1) (2,2) (2,3)

Index formula: id = row × cols + col

   0  1  2  3
   4  5  6  7
   8  9 10 11

Reverse: row = id // cols, col = id % cols

Total nodes: rows × cols
```

---

## 4-Directional Connectivity

```
Each cell connects to up to 4 neighbors:
  Up:    (r-1, c)
  Down:  (r+1, c)
  Left:  (r, c-1)
  Right: (r, c+1)

Optimization: Only check RIGHT and DOWN to avoid 
              duplicate unions:

  For each cell (r,c):
    if cell to the RIGHT (r, c+1) is valid: union
    if cell BELOW (r+1, c) is valid: union

  This covers all connections because:
  - (r,c)→RIGHT = (r, c+1)→LEFT (same edge)
  - (r,c)→DOWN = (r+1, c)→UP (same edge)
```

---

## Problem: Number of Islands

```
LeetCode 200: Number of Islands

Grid:
  1 1 1 1 0
  1 1 0 1 0
  1 1 0 0 0
  0 0 0 0 0

Islands (connected '1' regions): 1
```

```python
def numIslands(grid):
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    uf = UnionFind(rows * cols)
    
    land = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                land += 1
    
    uf.components = land  # start with land cells only
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                idx = r * cols + c
                # Check right
                if c + 1 < cols and grid[r][c+1] == '1':
                    uf.union(idx, idx + 1)
                # Check down
                if r + 1 < rows and grid[r+1][c] == '1':
                    uf.union(idx, idx + cols)
    
    return uf.components
```

---

## Problem: Number of Islands II (Online)

```
LeetCode 305: Number of Islands II

Start with all water. Add land cells one at a time.
After each addition, report the number of islands.

Grid: 3×3, initially all water
Operations: addLand(0,0), addLand(0,1), addLand(1,2), 
            addLand(2,1), addLand(1,1)

Step 1: add (0,0)     Step 2: add (0,1)
  1 0 0                 1 1 0
  0 0 0                 0 0 0
  0 0 0                 0 0 0
  islands = 1           islands = 1

Step 3: add (1,2)     Step 4: add (2,1)
  1 1 0                 1 1 0
  0 0 1                 0 0 1
  0 0 0                 0 1 0
  islands = 2           islands = 3

Step 5: add (1,1)
  1 1 0
  0 1 1           ← connects (0,1), (1,2), (2,1)
  0 1 0
  islands = 1

Output: [1, 1, 2, 3, 1]
```

```python
def numIslands2(m, n, positions):
    uf = UnionFind(m * n)
    uf.components = 0       # start with 0 land
    is_land = set()
    result = []
    
    for r, c in positions:
        if (r, c) in is_land:
            result.append(uf.components)
            continue
        
        is_land.add((r, c))
        idx = r * n + c
        uf.components += 1   # new island
        
        # Connect to existing land neighbors
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and (nr, nc) in is_land:
                uf.union(idx, nr * n + nc)
        
        result.append(uf.components)
    
    return result
```

```
This is where Union-Find SHINES over BFS/DFS!

BFS approach: After each addition, re-run BFS → O(k × m × n)
DSU approach: Each addition → O(α(m×n)) → O(k × α(m×n))

For k additions on m×n grid:
  BFS: O(k × m × n) — potentially 10^10
  DSU: O(k × α(m×n)) — essentially O(k)
```

---

## Problem: Surrounded Regions

```
LeetCode 130: Surrounded Regions

Replace all 'O' regions that are NOT connected to 
the border with 'X'.

Board:
  X X X X          X X X X
  X O O X    →     X X X X
  X X O X          X X X X
  X O X X          X O X X

The 'O' at (3,1) touches the border → stays 'O'.
The 'O' group at (1,1),(1,2),(2,2) doesn't → becomes 'X'.
```

```python
def solve(board):
    if not board:
        return
    
    rows, cols = len(board), len(board[0])
    uf = UnionFind(rows * cols + 1)  # +1 for virtual border node
    border = rows * cols  # virtual node for all border O's
    
    for r in range(rows):
        for c in range(cols):
            if board[r][c] == 'O':
                idx = r * cols + c
                
                # Border O → connect to virtual border node
                if r == 0 or r == rows-1 or c == 0 or c == cols-1:
                    uf.union(idx, border)
                
                # Connect to right/down O neighbors
                if c+1 < cols and board[r][c+1] == 'O':
                    uf.union(idx, idx + 1)
                if r+1 < rows and board[r+1][c] == 'O':
                    uf.union(idx, idx + cols)
    
    # Replace O's not connected to border
    for r in range(rows):
        for c in range(cols):
            if board[r][c] == 'O':
                if not uf.connected(r * cols + c, border):
                    board[r][c] = 'X'
```

---

## Virtual Node Pattern

```
The VIRTUAL NODE is a powerful DSU technique for grids:

Create an extra "dummy" node that represents a concept:
  - Border: Connect all border cells to it
  - Water: Connect all water cells to it
  - Source: Connect all source cells to it

Then: connected(cell, virtual) answers "Is this cell 
      connected to the border/water/source?"

Example: Surrounded Regions
  Virtual border node = n×m (index after all cells)
  Connect border 'O's to it
  Any 'O' connected to border node → don't flip
```

---

## 8-Directional Connectivity

```
Some problems use 8 directions (including diagonals):

  ↖ ↑ ↗
  ← · →
  ↙ ↓ ↘

Directions: [(-1,-1),(-1,0),(-1,1),
             (0,-1),         (0,1),
             (1,-1), (1,0), (1,1)]

Same Union-Find approach, just more neighbors to check.

Optimization: Check only right, down, down-right, down-left
  → covers all edges without duplicates
```

---

## Summary Table

| Grid Problem | DSU Technique |
|-------------|---------------|
| Number of Islands | Union adjacent land cells |
| Islands II (online) | Add cells dynamically, union neighbors |
| Surrounded Regions | Virtual border node |
| Grid connectivity | 2D→1D mapping + directional unions |
| Component sizes | Union by size + grid scan |

---

## Quick Revision Questions

1. **How do you convert (row, col) to a Union-Find index?**
2. **Why check only right and down neighbors during union?**
3. **What advantage does Union-Find have for "Number of Islands II"?**
4. **What is the virtual node technique?**
5. **How do you handle 8-directional connectivity differently from 4-directional?**

---

[← Previous: Connected Components in Graph](05-connected-components-in-graph.md) | [Next: Unit 6 - Cycle in Undirected Graph →](../06-Cycle-Detection/01-cycle-in-undirected-graph.md)

[← Back to Main README](../README.md)
