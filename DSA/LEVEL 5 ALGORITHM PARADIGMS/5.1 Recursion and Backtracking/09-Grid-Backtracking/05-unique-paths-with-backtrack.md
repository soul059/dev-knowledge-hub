# Chapter 5: Unique Paths with Backtracking

## Overview
The **Unique Paths** problem counts all distinct paths from the top-left to bottom-right of a grid, moving only right or down. While DP is the optimal approach, solving it with backtracking reveals the recursive structure and naturally leads to the DP insight. This chapter bridges grid backtracking and dynamic programming.

---

## Problem Statement

```
Given an m×n grid, find the number of unique paths
from top-left (0,0) to bottom-right (m-1, n-1).
You can only move RIGHT or DOWN.

Example: 3×3 grid
┌───┬───┬───┐
│ S │ → │ → │    S = Start
├───┼───┼───┤    G = Goal
│ ↓ │   │ ↓ │    → = Right
├───┼───┼───┤    ↓ = Down
│   │ → │ G │
└───┴───┴───┘

All 6 paths for 3×3:
Path 1: →→↓↓    Path 4: ↓→↓→
Path 2: →↓→↓    Path 5: ↓→→↓
Path 3: →↓↓→    Path 6: ↓↓→→
```

---

## Approach 1: Pure Backtracking

```
FUNCTION countPaths(r, c, m, n):
    IF r == m-1 AND c == n-1:
        RETURN 1                    ← Reached goal!
    IF r >= m OR c >= n:
        RETURN 0                    ← Out of bounds

    right = countPaths(r, c+1, m, n)   ← Go right
    down  = countPaths(r+1, c, m, n)   ← Go down

    RETURN right + down

Call: countPaths(0, 0, m, n)
```

---

## Recursion Tree for 3×3

```
                    (0,0)
                   /     \
              (0,1)       (1,0)
             /    \       /    \
         (0,2)  (1,1)  (1,1)  (2,0)
          |     /  \    /  \     |
        (1,2) (1,2)(2,1)(1,2)(2,1)(2,1)
          |     |    |    |    |    |
        (2,2) (2,2)(2,2)(2,2)(2,2)(2,2)
          1     1    1    1    1    1

Answer = 6 ✓

NOTICE: (1,1) appears TWICE → overlapping subproblems!
         (1,2) appears THREE times
         (2,1) appears THREE times
```

---

## Approach 2: With Obstacles

```
Grid with obstacles (1 = blocked):
┌───┬───┬───┐
│ 0 │ 0 │ 0 │
├───┼───┼───┤
│ 0 │ 1 │ 0 │   Cell (1,1) is blocked
├───┼───┼───┤
│ 0 │ 0 │ 0 │
└───┴───┴───┘

FUNCTION countPaths(grid, r, c):
    IF r >= m OR c >= n:
        RETURN 0                       ← Out of bounds
    IF grid[r][c] == 1:
        RETURN 0                       ← Obstacle!
    IF r == m-1 AND c == n-1:
        RETURN 1                       ← Goal reached
    
    RETURN countPaths(grid, r, c+1)    ← Right
         + countPaths(grid, r+1, c)    ← Down

With obstacle at (1,1): Answer = 2
  Path 1: →→↓↓
  Path 2: ↓↓→→
```

---

## Approach 3: With All 4 Directions

```
If movement in all 4 directions is allowed,
we NEED a visited array (to prevent cycles):

FUNCTION countPaths(grid, r, c, visited):
    IF r == m-1 AND c == n-1:
        RETURN 1
    
    visited[r][c] = TRUE              ← Mark
    count = 0
    
    FOR each (dr, dc) in [(0,1),(0,-1),(1,0),(-1,0)]:
        nr = r + dr
        nc = c + dc
        IF inBounds(nr, nc) AND NOT visited[nr][nc] 
           AND grid[nr][nc] != 1:
            count += countPaths(grid, nr, nc, visited)
    
    visited[r][c] = FALSE             ← Unmark (backtrack!)
    RETURN count

This is TRUE backtracking (explore all, undo choices).
```

---

## Backtracking → DP Transformation

```
Step 1: BACKTRACKING (exponential)
  countPaths(r, c) = countPaths(r, c+1) + countPaths(r+1, c)
  Time: O(2^(m+n))

Step 2: MEMOIZATION (top-down DP)
  Add cache: memo[r][c] = result for (r, c)
  IF memo[r][c] != -1: RETURN memo[r][c]
  Time: O(m × n)

Step 3: TABULATION (bottom-up DP)
  dp[r][c] = dp[r][c+1] + dp[r+1][c]
  Fill right-to-left, bottom-to-top
  Time: O(m × n), Space: O(m × n)

Step 4: SPACE OPTIMIZED
  Only need current row and next row
  Time: O(m × n), Space: O(n)

Visual of DP table for 3×3:
  ┌───┬───┬───┐
  │ 6 │ 3 │ 1 │  dp[0][0]=6 → answer
  ├───┼───┼───┤
  │ 3 │ 2 │ 1 │
  ├───┼───┼───┤
  │ 1 │ 1 │ 1 │  Base: last row/col = 1
  └───┴───┴───┘
```

---

## Mathematical Formula (No DP Needed)

```
For m×n grid (right/down only, no obstacles):

            (m-1+n-1)!     (m+n-2)!
Paths  =  ─────────── = ──────────────
           (m-1)!(n-1)!  (m-1)! × (n-1)!

This is C(m+n-2, m-1) — choosing which of
(m+n-2) steps are "down" (rest are "right")

3×3: C(4,2) = 6  ✓
3×7: C(8,2) = 28
```

---

## Complexity Comparison

```
┌───────────────────┬──────────────┬───────────┐
│ Approach          │ Time         │ Space     │
├───────────────────┼──────────────┼───────────┤
│ Backtracking      │ O(2^(m+n))   │ O(m+n)   │
│ Memoization       │ O(m×n)       │ O(m×n)   │
│ Tabulation        │ O(m×n)       │ O(m×n)   │
│ Space-optimized   │ O(m×n)       │ O(n)     │
│ Combinatorics     │ O(m+n)       │ O(1)     │
│ 4-dir backtrack   │ O(4^(m×n))   │ O(m×n)   │
└───────────────────┴──────────────┴───────────┘
```

---

## When to Use Backtracking vs DP

| Use Backtracking When... | Use DP When... |
|--------------------------|----------------|
| Need actual paths (list them) | Need only count |
| 4 directions allowed (cycles possible) | 2 directions (right/down) |
| Grid has complex constraints | Simple obstacle grid |
| State can't be easily memoized | Clear subproblem structure |
| Want to enumerate solutions | Want optimal efficiency |

---

## Quick Revision Questions

1. **Why does 2-direction** unique paths have overlapping subproblems?
2. **Why is a visited array** needed for 4-direction movement but not 2?
3. **How do obstacles** change the recurrence relation?
4. **What is the combinatorial formula** for an m×n grid?
5. **What's the key difference** between counting paths and listing paths?
6. **How do you transform** the backtracking solution to DP step by step?

---

[← Previous: Word Search](04-word-search.md) | [Next: Knight's Tour →](06-knights-tour.md)

[← Back to README](../README.md)
