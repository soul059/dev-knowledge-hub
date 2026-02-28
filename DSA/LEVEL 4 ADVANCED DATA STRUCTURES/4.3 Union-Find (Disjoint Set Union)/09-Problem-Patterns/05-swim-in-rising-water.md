# Chapter 5: Swim in Rising Water (LeetCode 778)

## Overview

Given an N×N grid of elevations, find the minimum time `t` such that you can swim from (0,0) to (N-1,N-1). At time `t`, any cell with elevation ≤ t is underwater and you can swim through it. This is solved elegantly with Union-Find + sorted edge processing.

---

## Problem Statement

```
Input: grid = [
    [0,  2],
    [1,  3]
]

At time t:
  t=0: only cell(0,0)=0 is accessible
  t=1: cells 0,1 accessible → (0,0) and (1,0)
  t=2: cells 0,1,2 → (0,0),(1,0),(0,1) → not connected to (1,1)
  t=3: all cells accessible → can reach (1,1)!

Output: 3

Visualization at t=3:
  ┌───┬───┐
  │ 0 │ 2 │   All ≤ 3
  ├───┼───┤
  │ 1 │ 3 │   Path: (0,0)→(1,0)→... any path works
  └───┴───┘
```

---

## Union-Find Approach

```
Idea: Process cells in order of elevation (low → high).
At each step, add the cell and connect to adjacent 
underwater cells. Stop when (0,0) and (N-1,N-1) 
are connected.

Algorithm:
  1. Sort all cells by elevation
  2. Process cells low to high
  3. For each cell, union with adjacent cells already 
     processed
  4. Check if (0,0) and (N-1,N-1) are connected
  5. Return current elevation when connected

The answer is the MAXIMUM elevation along the optimal 
path (minimax path problem).
```

---

## Implementation

```python
def swimInWater(grid):
    N = len(grid)
    uf = UnionFind(N * N)
    
    # Sort cells by elevation
    cells = []
    for r in range(N):
        for c in range(N):
            cells.append((grid[r][c], r, c))
    cells.sort()
    
    # Track which cells are "underwater"
    underwater = [[False] * N for _ in range(N)]
    
    start = 0
    end = N * N - 1
    
    for elev, r, c in cells:
        underwater[r][c] = True
        idx = r * N + c
        
        # Connect to adjacent underwater cells
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < N and 0 <= nc < N and underwater[nr][nc]:
                uf.union(idx, nr * N + nc)
        
        # Check if start and end are connected
        if uf.connected(start, end):
            return elev
    
    return -1  # should never reach here
```

---

## Trace Example

```
grid = [[0, 1, 2, 3, 4],
        [24,23,22,21, 5],
        [12,13,14,15,16],
        [11,17,18,19,20],
        [10, 9, 8, 7, 6]]

Sorted cells: (0,0,0), (1,0,1), (2,0,2), (3,0,3), 
              (4,0,4), (5,1,4), (6,4,4), (7,4,3), ...

Processing:
  t=0: Add (0,0). No neighbors underwater.
  t=1: Add (0,1). Union with (0,0). ✓
  t=2: Add (0,2). Union with (0,1). 
  t=3: Add (0,3). Union with (0,2).
  t=4: Add (0,4). Union with (0,3).
  t=5: Add (1,4). Union with (0,4).
  t=6: Add (4,4). [no underwater neighbor yet]
  t=7: Add (4,3). Union with (4,4).
  ...
  Eventually: path connects through right side and bottom

  When (0,0) ~ (4,4) → return that elevation.
  
  Answer: 16 (the bottleneck on the optimal path)
```

---

## Alternative: Binary Search + BFS

```
Binary search on answer t:
  For a given t, can we reach (N-1,N-1) from (0,0)
  using only cells with elevation ≤ t?
  
  → BFS/DFS on cells ≤ t

Time: O(N² log(N²)) = O(N² log N)

Union-Find approach: O(N² × α(N²)) ≈ O(N²)
  → theoretically faster!
```

---

## Related: Path with Minimum Maximum (Minimax Path)

```
This is a MINIMAX PATH problem:
  Find path from s to t that minimizes the 
  MAXIMUM edge/node weight along the path.

General solution:
  Sort edges by weight, add them Kruskal-style.
  Stop when s and t are connected.
  The last edge added = the answer.

For grid problems:
  "Edges" = adjacent cells, weight = max(elev[a], elev[b])
  Or simply process cells by elevation (node-weighted version).
```

```python
def minMaxPath(grid):
    """Minimax path using sorted edges + Union-Find."""
    N = len(grid)
    uf = UnionFind(N * N)
    
    # Create edges (between adjacent cells)
    edges = []
    for r in range(N):
        for c in range(N):
            if r + 1 < N:
                w = max(grid[r][c], grid[r+1][c])
                edges.append((w, r * N + c, (r+1) * N + c))
            if c + 1 < N:
                w = max(grid[r][c], grid[r][c+1])
                edges.append((w, r * N + c, r * N + c + 1))
    
    edges.sort()
    
    start, end = 0, N * N - 1
    
    for w, u, v in edges:
        uf.union(u, v)
        if uf.connected(start, end):
            return w
    
    return -1
```

---

## Related Problems

```
┌─────────────────────────────────────┬─────────────────┐
│ Problem                             │ Pattern         │
├─────────────────────────────────────┼─────────────────┤
│ Swim in Rising Water (778)          │ Minimax node    │
│ Path with Min Effort (1631)         │ Minimax edge    │
│ Cheapest Flights within K stops     │ Different (BFS) │
│ Find Critical + Pseudo-Critical     │ MST edge check  │
│   Edges in MST (1489)               │                 │
└─────────────────────────────────────┴─────────────────┘
```

---

## Path with Minimum Effort (LeetCode 1631)

```
Similar but edge weight = |elevation difference| 
between adjacent cells.

Minimax path = path minimizing maximum edge weight.

Same approach:
  1. Create edges with weight = |diff|
  2. Sort edges
  3. Union-Find, stop when connected
```

```python
def minimumEffortPath(heights):
    rows, cols = len(heights), len(heights[0])
    uf = UnionFind(rows * cols)
    
    edges = []
    for r in range(rows):
        for c in range(cols):
            if r + 1 < rows:
                diff = abs(heights[r][c] - heights[r+1][c])
                edges.append((diff, r * cols + c, (r+1) * cols + c))
            if c + 1 < cols:
                diff = abs(heights[r][c] - heights[r][c+1])
                edges.append((diff, r * cols + c, r * cols + c + 1))
    
    edges.sort()
    
    for w, u, v in edges:
        uf.union(u, v)
        if uf.connected(0, rows * cols - 1):
            return w
    
    return 0
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem type | Minimax path |
| DSU approach | Sort cells/edges, process low→high |
| Stop condition | Source connected to destination |
| Answer | Last cell/edge elevation when connected |
| Alternatives | Binary search + BFS, Dijkstra |
| Time complexity | O(N² log N) with sorting |

---

## Quick Revision Questions

1. **What is a minimax path problem?**
2. **How does sorting cells by elevation help solve this?**
3. **When do you stop processing cells?**
4. **What's the difference between Swim in Rising Water and Path with Minimum Effort?**
5. **How does this relate to Kruskal's algorithm?**

---

[← Previous: Most Stones Removed](04-most-stones-removed.md) | [Next: Satisfiability of Equations →](06-satisfiability-of-equations.md)
