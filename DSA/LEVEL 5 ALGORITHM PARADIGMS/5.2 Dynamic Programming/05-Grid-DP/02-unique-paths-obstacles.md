# Chapter 2: Unique Paths with Obstacles

## 📋 Overview
**Unique Paths II** (LeetCode #63) adds obstacles to the grid. Cells marked as obstacles cannot be traversed. This teaches how to handle **invalid states** in grid DP — a critical skill for more complex problems.

---

## 🧠 Core Concept

```
Grid with obstacles (O = obstacle):
  ┌───┬───┬───┐
  │ S │   │   │
  ├───┼───┼───┤
  │   │ O │   │
  ├───┼───┼───┤
  │   │   │ E │
  └───┴───┴───┘

  Can still only move RIGHT or DOWN.
  Cannot pass through obstacles.

Key modification to recurrence:
  if grid[i][j] == obstacle:
      dp[i][j] = 0          ← no paths through here
  else:
      dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

---

## 🔨 Solution

```
function uniquePathsWithObstacles(grid):
    m = rows, n = cols
    
    // If start or end is blocked
    if grid[0][0] == 1 or grid[m-1][n-1] == 1:
        return 0
    
    dp = 2D array of size m × n, filled with 0
    dp[0][0] = 1
    
    // First row: stop at first obstacle
    for j = 1 to n-1:
        dp[0][j] = 0 if grid[0][j]==1 else dp[0][j-1]
    
    // First column: stop at first obstacle
    for i = 1 to m-1:
        dp[i][0] = 0 if grid[i][0]==1 else dp[i-1][0]
    
    // Fill rest
    for i = 1 to m-1:
        for j = 1 to n-1:
            if grid[i][j] == 1:
                dp[i][j] = 0
            else:
                dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    return dp[m-1][n-1]
```

---

## 🔬 Trace: 3×3 grid with obstacle at (1,1)

```
Grid:                    dp table:
  0  0  0                  1  1  1
  0  1  0      →           1  0  1
  0  0  0                  1  1  2

Step-by-step:
  dp[0][0] = 1 (start)
  dp[0][1] = dp[0][0] = 1
  dp[0][2] = dp[0][1] = 1
  dp[1][0] = dp[0][0] = 1
  dp[1][1] = 0  ← OBSTACLE
  dp[1][2] = dp[0][2] + dp[1][1] = 1 + 0 = 1
  dp[2][0] = dp[1][0] = 1
  dp[2][1] = dp[1][1] + dp[2][0] = 0 + 1 = 1
  dp[2][2] = dp[1][2] + dp[2][1] = 1 + 1 = 2

Answer: 2

The two paths:
  Path 1: →→↓↓  (right, right, down, down)
  Path 2: ↓↓→→  (down, down, right, right)
```

---

## 🔬 Trace: Obstacle blocks entire column

```
Grid:                    dp table:
  0  0  0                  1  1  1
  1  0  0      →           0  1  2
  0  0  0                  0  1  3

dp[1][0] = 0 (obstacle!)
dp[2][0] = dp[1][0] = 0 (blocked by obstacle above)

Entire left column below obstacle becomes 0.
This is why first row/column initialization must stop at obstacles.
```

---

## 💻 Space-Optimized Version

```
function uniquePathsWithObstacles(grid):
    m = rows, n = cols
    if grid[0][0] == 1: return 0
    
    row = array of size n, filled with 0
    row[0] = 1
    
    for i = 0 to m-1:
        for j = 0 to n-1:
            if grid[i][j] == 1:
                row[j] = 0    // obstacle: zero paths
            else if j > 0:
                row[j] += row[j-1]  // from above (row[j]) + from left (row[j-1])
        // Special: if first cell in row is obstacle
        if grid[i][0] == 1:
            row[0] = 0
    
    return row[n-1]

Time: O(m·n), Space: O(n)
```

---

## 🌐 Edge Cases

```
┌──────────────────────────────┬──────────┬─────────────┐
│ Edge Case                    │ Grid     │ Answer      │
├──────────────────────────────┼──────────┼─────────────┤
│ Start is blocked             │ [1,0]    │ 0           │
│                              │ [0,0]    │             │
├──────────────────────────────┼──────────┼─────────────┤
│ End is blocked               │ [0,0]    │ 0           │
│                              │ [0,1]    │             │
├──────────────────────────────┼──────────┼─────────────┤
│ No path exists               │ [0,1]    │ 0           │
│                              │ [1,0]    │             │
├──────────────────────────────┼──────────┼─────────────┤
│ 1×1 grid, no obstacle        │ [0]      │ 1           │
├──────────────────────────────┼──────────┼─────────────┤
│ 1×1 grid, obstacle           │ [1]      │ 0           │
├──────────────────────────────┼──────────┼─────────────┤
│ No obstacles                 │ [0,0]    │ Standard    │
│                              │ [0,0]    │ unique paths│
└──────────────────────────────┴──────────┴─────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = paths to (i,j) avoiding obstacles |
| **Recurrence** | dp[i][j] = 0 if obstacle, else dp[i-1][j]+dp[i][j-1] |
| **Key Change** | Obstacle cells get dp = 0 |
| **Base Cases** | First row/col = 1 until first obstacle, then 0 |
| **Complexity** | O(m·n) time, O(n) space |
| **Edge Cases** | Start/end blocked, no path exists |

---

## ❓ Quick Revision Questions

1. **How does the recurrence change when a cell is an obstacle?**
2. **Why does an obstacle in the first row zero out all cells to its right?**
3. **What happens if the start or end cell is an obstacle?**
4. **How do you handle the space-optimized version with obstacles?**
5. **Draw the dp table for a 3×3 grid with obstacle at (0,2).**
6. **Can you modify this approach for multiple obstacle types with different costs?**

---

[← Previous: Unique Paths](01-unique-paths.md) | [Next: Minimum Path Sum →](03-minimum-path-sum.md)

[← Back to README](../README.md)
