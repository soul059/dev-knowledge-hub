# Chapter 1: Rat in a Maze

## Overview
The Rat in a Maze problem is a **classic grid backtracking** problem: find all paths from the top-left to the bottom-right of a grid, moving only through open cells. It teaches the **visit-explore-unvisit** pattern for grid traversal.

---

## Problem Statement

```
Given a grid where 1 = open, 0 = blocked:

  ┌───┬───┬───┬───┐
  │ 1 │ 0 │ 0 │ 0 │
  ├───┼───┼───┼───┤
  │ 1 │ 1 │ 0 │ 1 │
  ├───┼───┼───┼───┤
  │ 0 │ 1 │ 0 │ 0 │
  ├───┼───┼───┼───┤
  │ 1 │ 1 │ 1 │ 1 │
  └───┴───┴───┴───┘

Find ALL paths from (0,0) to (3,3).
Allowed moves: Down (D), Left (L), Right (R), Up (U)
Cannot revisit cells.
```

---

## Pseudocode

```
DIRECTIONS = [(1,0,'D'), (0,-1,'L'), (0,1,'R'), (-1,0,'U')]

FUNCTION findPaths(grid):
    n = len(grid)
    IF grid[0][0] == 0: RETURN []   ← Start blocked
    
    visited = n×n grid of false
    result = []
    visited[0][0] = true
    backtrack(grid, 0, 0, n, visited, "", result)
    RETURN result

FUNCTION backtrack(grid, r, c, n, visited, path, result):
    IF r == n-1 AND c == n-1:       ← Reached destination
        result.add(path)
        RETURN
    
    FOR each (dr, dc, dir) in DIRECTIONS:
        nr = r + dr
        nc = c + dc
        IF isValid(nr, nc, n, grid, visited):
            visited[nr][nc] = true       ← MARK visited
            backtrack(grid, nr, nc, n, visited, path + dir, result)
            visited[nr][nc] = false      ← UNMARK (backtrack!)

FUNCTION isValid(r, c, n, grid, visited):
    RETURN 0 <= r < n AND 0 <= c < n
           AND grid[r][c] == 1
           AND NOT visited[r][c]
```

---

## Trace for the Example Grid

```
Grid:
  1 0 0 0
  1 1 0 1
  0 1 0 0
  1 1 1 1

Starting at (0,0):

(0,0) → D → (1,0) → D → blocked
                     → R → (1,1) → D → (2,1) → D → (3,1)
                                                      → R → (3,2)
                                                              → R → (3,3) ✓
                                                                    Path: "DRDDRR"

Let's trace more carefully:
Path 1: (0,0)→(1,0)→(1,1)→(2,1)→(3,1)→(3,2)→(3,3) = "DRDDRR"
Path 2: (0,0)→(1,0)→(1,1)→(2,1)→(3,1)→(3,2)→(3,3) 
  (exploring other directions might find more paths 
   depending on the grid configuration)

For this specific grid: 
  Result: ["DRDDRR"]  (only one path exists)
```

---

## Why We Need Visited Array

```
Without visited tracking, we get INFINITE LOOPS:

(0,0) → D → (1,0) → U → (0,0) → D → (1,0) → ...  INFINITE!

The visited array prevents revisiting:
  visited[0][0] = true → can't go back to (0,0) from (1,0)

CRITICAL: Must UNMARK when backtracking!
  Otherwise, valid paths through that cell from OTHER
  starting directions would be blocked.
```

---

## Grid Backtracking Template

```
This pattern applies to ALL grid backtracking problems:

FUNCTION gridBacktrack(grid, r, c, ...):
    // BASE CASE
    IF reached goal:
        record solution
        RETURN
    
    // TRY ALL DIRECTIONS
    FOR each (dr, dc) in directions:
        nr, nc = r + dr, c + dc
        IF inBounds(nr, nc) AND isOpen(nr, nc) AND NOT visited:
            visited[nr][nc] = true       ← MARK
            gridBacktrack(grid, nr, nc, ...)
            visited[nr][nc] = false      ← UNMARK
```

---

## Variations

```
Variant 1: Find ANY path (return first found)
  → Return true/false, stop on first success

Variant 2: Find SHORTEST path
  → Use BFS instead (backtracking finds all, not shortest)

Variant 3: Count paths
  → Return count instead of recording paths

Variant 4: Only Down and Right moves
  → Simpler (no cycles possible, no visited needed)
  → Can use DP: dp[r][c] = dp[r-1][c] + dp[r][c-1]
```

---

## Complexity

```
┌────────────────────────────────────────────────────┐
│ Worst case: all cells open, 4 directions           │
│                                                    │
│ Time: O(4^(n²)) theoretical worst case             │
│   (each cell has up to 4 choices)                  │
│   Practical: much less due to visited pruning      │
│                                                    │
│ Space: O(n²) for visited + O(n²) recursion depth   │
│                                                    │
│ For the "only D and R" variant:                    │
│   Total paths: C(2(n-1), n-1)                      │
│   Time: O(paths × path length)                     │
└────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Grid cell states | 1 (open) / 0 (blocked) |
| Direction order | D, L, R, U (alphabetical for sorted output) |
| Visited array | Essential to prevent cycles |
| Mark/Unmark | Core backtracking pattern |
| Base case | Reached (n-1, n-1) |

---

## Quick Revision Questions

1. **Why is a visited array** essential for grid backtracking?
2. **What happens** if you forget to unmark visited cells?
3. **In what order** should directions be tried for sorted output?
4. **Can this problem be solved** with BFS? What would BFS find instead?
5. **For a grid where only D/R** moves are allowed, why is DP better?
6. **What is the base case** and how do you record the solution?

---

[← Previous Unit: String Permutations](../08-Permutations/06-string-permutations.md) | [Next: N-Queens Problem →](02-n-queens-problem.md)

[← Back to README](../README.md)
