# Chapter 3: Minimum Path Sum

## 📋 Overview
**Minimum Path Sum** (LeetCode #64) asks for the path from top-left to bottom-right in a grid of non-negative numbers that minimizes the sum of values along the path. It transitions from counting paths to **optimizing** over paths — the same grid DP structure, but with `min` instead of `+`.

---

## 🧠 Core Concept

```
Grid:
  ┌───┬───┬───┐
  │ 1 │ 3 │ 1 │
  ├───┼───┼───┤
  │ 1 │ 5 │ 1 │
  ├───┼───┼───┤
  │ 4 │ 2 │ 1 │
  └───┴───┴───┘

Optimal path: 1→3→1→1→1 = 7?  or  1→1→5→1→1 = 9?
Actually:     1→1→4→2→1 = 9?  or  1→3→1→1→1 = 7 ✓

  1 → 3 → 1
              ↓
              1
              ↓
              1     Total: 1+3+1+1+1 = 7

Recurrence:
  dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
              ↑ current     ↑ from above   ↑ from left
              cell cost     best way to arrive here
```

---

## 🔨 Solution

```
function minPathSum(grid):
    m = rows, n = cols
    dp = 2D array of size m × n
    
    // Base case: start
    dp[0][0] = grid[0][0]
    
    // First row: can only come from left
    for j = 1 to n-1:
        dp[0][j] = dp[0][j-1] + grid[0][j]
    
    // First column: can only come from above
    for i = 1 to m-1:
        dp[i][0] = dp[i-1][0] + grid[i][0]
    
    // Fill rest
    for i = 1 to m-1:
        for j = 1 to n-1:
            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
    
    return dp[m-1][n-1]
```

---

## 🔬 Trace

```
Grid:                    dp table:
  1  3  1                  1  4  5
  1  5  1      →           2  7  6
  4  2  1                  6  8  7

Step-by-step:
  dp[0][0] = 1
  dp[0][1] = 1+3 = 4
  dp[0][2] = 4+1 = 5
  dp[1][0] = 1+1 = 2
  dp[1][1] = 5 + min(4, 2) = 5+2 = 7
  dp[1][2] = 1 + min(5, 7) = 1+5 = 6
  dp[2][0] = 2+4 = 6
  dp[2][1] = 2 + min(7, 6) = 2+6 = 8
  dp[2][2] = 1 + min(6, 8) = 1+6 = 7

Answer: 7

Path reconstruction (backtrack from dp[2][2]):
  (2,2)→ came from (1,2) since dp[1][2]=6 < dp[2][1]=8
  (1,2)→ came from (0,2) since dp[0][2]=5 < dp[1][1]=7
  (0,2)→ came from (0,1) since only left
  (0,1)→ came from (0,0) since only left
  
  Path: (0,0)→(0,1)→(0,2)→(1,2)→(2,2)
  Sum:    1  +  3  +  1  +  1  +  1  = 7 ✓
```

---

## 💻 Space-Optimized Version

```
function minPathSum(grid):
    m = rows, n = cols
    row = array of size n
    
    // Initialize first row
    row[0] = grid[0][0]
    for j = 1 to n-1:
        row[j] = row[j-1] + grid[0][j]
    
    // Process remaining rows
    for i = 1 to m-1:
        row[0] += grid[i][0]   // first column
        for j = 1 to n-1:
            row[j] = grid[i][j] + min(row[j], row[j-1])
            //                      ↑ above    ↑ left
    
    return row[n-1]

Time: O(m·n), Space: O(n)
```

### In-Place Modification — O(1) extra space
```
function minPathSum(grid):
    m = rows, n = cols
    
    for j = 1 to n-1: grid[0][j] += grid[0][j-1]
    for i = 1 to m-1: grid[i][0] += grid[i-1][0]
    
    for i = 1 to m-1:
        for j = 1 to n-1:
            grid[i][j] += min(grid[i-1][j], grid[i][j-1])
    
    return grid[m-1][n-1]

Warning: modifies input grid
```

---

## 🔄 Path Reconstruction

```
function reconstructPath(dp, grid):
    m = rows, n = cols
    path = [(m-1, n-1)]
    i, j = m-1, n-1
    
    while i > 0 or j > 0:
        if i == 0:
            j -= 1       // must go left
        else if j == 0:
            i -= 1       // must go up
        else if dp[i-1][j] < dp[i][j-1]:
            i -= 1       // came from above
        else:
            j -= 1       // came from left
        path.append((i, j))
    
    return reverse(path)
```

---

## 🌐 Variants

### Variant 1: Maximum Path Sum
```
Just change min to max:
  dp[i][j] = grid[i][j] + max(dp[i-1][j], dp[i][j-1])
```

### Variant 2: 4-Direction Movement
```
If you can move UP, DOWN, LEFT, RIGHT:
  - DP alone won't work (cycles possible)
  - Use Dijkstra's algorithm instead
  - Time: O(m·n·log(m·n))
```

### Variant 3: With Negative Values
```
Negative values: min and max both work correctly with DP
since the graph is still a DAG (right and down only).
No cycles = DP is valid.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min cost to reach (i,j) from (0,0) |
| **Recurrence** | dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1]) |
| **Base** | dp[0][0]=grid[0][0], cumulative sums on edges |
| **Complexity** | O(m·n) time, O(1) space (in-place) |
| **Reconstruction** | Backtrack from (m-1,n-1) following smaller neighbor |
| **Key Difference** | Uses `min` instead of `+` vs Unique Paths |

---

## ❓ Quick Revision Questions

1. **What is the recurrence for Minimum Path Sum?**
2. **How do base cases differ from Unique Paths?**
3. **How do you reconstruct the actual optimal path?**
4. **How do you optimize to O(1) extra space?**
5. **Why can't you use DP if movement in all 4 directions is allowed?**
6. **What changes for Maximum Path Sum?**

---

[← Previous: Unique Paths with Obstacles](02-unique-paths-obstacles.md) | [Next: Triangle →](04-triangle.md)

[← Back to README](../README.md)
