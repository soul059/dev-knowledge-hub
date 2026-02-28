# Chapter 1: Unique Paths

## 📋 Overview
**Unique Paths** (LeetCode #62) is the foundational grid DP problem. Given an m×n grid, find the number of unique paths from the top-left corner to the bottom-right corner, moving only **right** or **down**. It perfectly illustrates how 1D DP extends to 2D.

---

## 🧠 Core Concept

```
Grid (3×7):
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ S │ → │ → │ → │ → │ → │ → │  S = Start
  ├───┼───┼───┼───┼───┼───┼───┤
  │ ↓ │   │   │   │   │   │   │
  ├───┼───┼───┼───┼───┼───┼───┤
  │ ↓ │   │   │   │   │   │ E │  E = End
  └───┴───┴───┴───┴───┴───┴───┘

  Can only move RIGHT (→) or DOWN (↓)
  
  Any valid path takes exactly (m-1) downs + (n-1) rights
  = (m+n-2) total moves
```

---

## 🔨 Step-by-Step Development

### Step 1: Define State
```
dp[i][j] = number of unique paths from (0,0) to (i,j)
```

### Step 2: Recurrence
```
dp[i][j] = dp[i-1][j] + dp[i][j-1]
            ↑ from above  ↑ from left

Can only arrive at (i,j) from above or from the left.
```

### Step 3: Base Cases
```
dp[0][j] = 1 for all j  (only one way along top row: all right)
dp[i][0] = 1 for all i  (only one way along left column: all down)
```

### Step 4: Fill Order
```
Left to right, top to bottom (each cell needs its top and left).
```

---

## 🔬 Trace: m=3, n=4

```
Fill dp table:

     j=0  j=1  j=2  j=3
i=0 [ 1    1    1    1  ]   ← top row all 1s
i=1 [ 1    2    3    4  ]   ← dp[1][1]=1+1=2, dp[1][2]=1+2=3, dp[1][3]=1+3=4
i=2 [ 1    3    6   10  ]   ← dp[2][1]=1+2=3, dp[2][2]=3+3=6, dp[2][3]=4+6=10

Answer: dp[2][3] = 10

Visual of all 10 paths for 3×4 grid:
Each path = sequence of R(right) and D(down):
RRRDDD... wait, it's 3 Rights + 2 Downs = C(5,2) = 10 ✓

Combinatorial verification:
  Total moves = (3-1) + (4-1) = 5
  Choose 2 downs from 5 moves = C(5,2) = 10 ✓
```

---

## 💻 Implementations

### Tabulation — O(m·n) time, O(m·n) space
```
function uniquePaths(m, n):
    dp = 2D array of size m × n
    
    // Base cases
    for i = 0 to m-1: dp[i][0] = 1
    for j = 0 to n-1: dp[0][j] = 1
    
    // Fill table
    for i = 1 to m-1:
        for j = 1 to n-1:
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    return dp[m-1][n-1]
```

### Space Optimized — O(m·n) time, O(n) space
```
function uniquePaths(m, n):
    row = array of size n, filled with 1
    
    for i = 1 to m-1:
        for j = 1 to n-1:
            row[j] = row[j] + row[j-1]
            //       ↑ from above  ↑ from left (already updated)
    
    return row[n-1]

How it works:
  row starts as [1, 1, 1, 1]  (top row)
  
  i=1: j=1: row[1]=1+1=2, j=2: row[2]=1+2=3, j=3: row[3]=1+3=4
       row = [1, 2, 3, 4]
  
  i=2: j=1: row[1]=2+1=3, j=2: row[2]=3+3=6, j=3: row[3]=4+6=10
       row = [1, 3, 6, 10]
  
  Answer: 10 ✓
```

### Combinatorial — O(min(m,n)) time, O(1) space
```
function uniquePaths(m, n):
    // Answer = C(m+n-2, m-1) = (m+n-2)! / ((m-1)! × (n-1)!)
    // Compute without overflow using incremental multiplication
    
    result = 1
    for i = 1 to min(m-1, n-1):
        result = result × (m + n - 1 - i) / i
    
    return result

Why: choosing (m-1) downs from (m+n-2) total moves
```

---

## 🌐 Pascal's Triangle Connection

```
The dp table IS Pascal's Triangle rotated 45°!

Pascal's Triangle:
         1
        1 1
       1 2 1
      1 3 3 1
     1 4 6 4 1
    1 5 10 10 5 1

Unique Paths dp table (4×4):
  1  1  1  1
  1  2  3  4
  1  3  6  10
  1  4  10 20

Both follow: entry = sum of entry above + entry to left
             (or entry above-left + entry above in Pascal's)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = paths from (0,0) to (i,j) |
| **Recurrence** | dp[i][j] = dp[i-1][j] + dp[i][j-1] |
| **Base Cases** | First row and first column are all 1 |
| **Time** | O(m·n) DP, O(min(m,n)) combinatorial |
| **Space** | O(n) with rolling array, O(1) combinatorial |
| **Math** | Answer = C(m+n-2, m-1) |

---

## ❓ Quick Revision Questions

1. **What is the recurrence relation for Unique Paths?**
2. **Why are all base cases (first row/column) equal to 1?**
3. **How do you optimize space from O(m·n) to O(n)?**
4. **What is the combinatorial formula for the answer?**
5. **How does the dp table relate to Pascal's Triangle?**
6. **What changes when obstacles are added? (Preview of next chapter)**

---

[← Previous Unit: Delete and Earn](../04-Fibonacci-Pattern/06-delete-and-earn.md) | [Next: Unique Paths with Obstacles →](02-unique-paths-obstacles.md)

[← Back to README](../README.md)
