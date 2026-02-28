# Chapter 4: Triangle

## 📋 Overview
**Triangle** (LeetCode #120) is a grid DP variant where the grid is triangular. Find the minimum path sum from the top to the bottom, where at each step you can move to the adjacent number on the row below. The key insight is to fill **bottom-up** to avoid complex indexing.

---

## 🧠 Core Concept

```
Triangle:
       2
      3 4
     6 5 7
    4 1 8 3

At position (i, j), you can move to:
  (i+1, j)   → directly below
  (i+1, j+1) → diagonally right below

       2
      ╱ ╲
     3   4
    ╱ ╲ ╱ ╲
   6   5   7
  ╱ ╲ ╱ ╲ ╱ ╲
 4   1   8   3

Minimum path: 2 → 3 → 5 → 1 = 11
```

---

## 🔨 Two Approaches

### Approach 1: Top-Down DP
```
dp[i][j] = min cost to reach position (i,j) from top

Recurrence:
  dp[i][j] = triangle[i][j] + min(dp[i-1][j-1], dp[i-1][j])
  (came from above-left or directly above)

Edge cases:
  j == 0:    dp[i][j] = triangle[i][j] + dp[i-1][j]     (only from above)
  j == i:    dp[i][j] = triangle[i][j] + dp[i-1][j-1]   (only from above-left)

Answer: min(dp[last_row])  ← need to check all positions in last row
```

### Approach 2: Bottom-Up DP (Preferred)
```
dp[i][j] = min cost from position (i,j) to bottom

Recurrence:
  dp[i][j] = triangle[i][j] + min(dp[i+1][j], dp[i+1][j+1])
  (choose cheaper path going down)

Answer: dp[0][0]  ← single answer, no need to scan!

This is cleaner because:
1. No edge case handling for left/right boundaries
2. Answer is a single value dp[0][0]
```

---

## 🔬 Trace: Bottom-Up Approach

```
Triangle:
       2
      3 4
     6 5 7
    4 1 8 3

Start from bottom row (dp = copy of last row):
  dp = [4, 1, 8, 3]

Row 2 (6, 5, 7):
  dp[0] = 6 + min(4, 1) = 6+1 = 7
  dp[1] = 5 + min(1, 8) = 5+1 = 6
  dp[2] = 7 + min(8, 3) = 7+3 = 10
  dp = [7, 6, 10, 3]

Row 1 (3, 4):
  dp[0] = 3 + min(7, 6) = 3+6 = 9
  dp[1] = 4 + min(6, 10) = 4+6 = 10
  dp = [9, 10, 10, 3]

Row 0 (2):
  dp[0] = 2 + min(9, 10) = 2+9 = 11
  dp = [11, 10, 10, 3]

Answer: dp[0] = 11

Path: 2 → 3 → 5 → 1 = 11 ✓
```

---

## 💻 Implementation

### Space-Optimized O(n) — Bottom-Up
```
function minimumTotal(triangle):
    n = len(triangle)
    dp = copy of triangle[n-1]  // last row
    
    for i = n-2 down to 0:
        for j = 0 to i:
            dp[j] = triangle[i][j] + min(dp[j], dp[j+1])
    
    return dp[0]

Time: O(n²) where n = number of rows
Space: O(n) — single array of last row length
```

### In-Place Modification
```
function minimumTotal(triangle):
    n = len(triangle)
    
    for i = n-2 down to 0:
        for j = 0 to i:
            triangle[i][j] += min(triangle[i+1][j], triangle[i+1][j+1])
    
    return triangle[0][0]

Time: O(n²), Space: O(1)
```

---

## 🌐 Why Bottom-Up is Better Here

```
Top-Down Issues:
  ┌─────────────────────────────────┐
  │ 1. Must handle edge cases       │
  │    (leftmost and rightmost)     │
  │ 2. Answer requires scanning     │
  │    entire last row              │
  │ 3. More complex boundary logic  │
  └─────────────────────────────────┘

Bottom-Up Advantages:
  ┌─────────────────────────────────┐
  │ 1. No boundary edge cases       │
  │ 2. Single answer at dp[0]       │
  │ 3. Cleaner code                 │
  │ 4. Natural space optimization   │
  │    (process rows bottom→top,    │
  │     shrinking the active area)  │
  └─────────────────────────────────┘

  Row sizes: 1, 2, 3, ..., n
  Bottom-up processes: n, n-1, ..., 1
  We only need dp of size n (last row), and it shrinks naturally.
```

---

## 🔬 Trace: Another Example

```
Triangle:
     -1
     2 3
    1 -1 -3

Bottom-up:
  dp = [1, -1, -3]

  Row 1 (2, 3):
    dp[0] = 2 + min(1, -1) = 2+(-1) = 1
    dp[1] = 3 + min(-1, -3) = 3+(-3) = 0
    dp = [1, 0, -3]

  Row 0 (-1):
    dp[0] = -1 + min(1, 0) = -1+0 = -1
    dp = [-1, 0, -3]

  Answer: -1
  Path: -1 → 3 → -3 = -1 ✓
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[j] = min cost from (i,j) to bottom |
| **Recurrence** | dp[j] = tri[i][j] + min(dp[j], dp[j+1]) |
| **Direction** | Bottom-up row processing |
| **Base** | dp = copy of last row |
| **Complexity** | O(n²) time, O(n) space |
| **Key Insight** | Bottom-up avoids boundary edge cases |

---

## ❓ Quick Revision Questions

1. **Why is bottom-up preferred over top-down for the Triangle problem?**
2. **What is the recurrence when processing bottom-up?**
3. **How do you achieve O(n) space complexity?**
4. **What are the adjacency options from position (i, j)?**
5. **Walk through the bottom-up trace for triangle [[1],[2,3],[4,5,6]].**
6. **How would you reconstruct the actual minimum path?**

---

[← Previous: Minimum Path Sum](03-minimum-path-sum.md) | [Next: Dungeon Game →](05-dungeon-game.md)

[← Back to README](../README.md)
