# Chapter 3: 2D DP Patterns

## 📋 Chapter Overview
2D DP uses `dp[i][j]` where two dimensions represent positions, lengths, or resources. Covers grid paths, two-string problems, and knapsack.

---

## 🔍 Pattern 1: Grid Paths

### Unique Paths

```
  m×n grid, start top-left, reach bottom-right.
  Can only move right or down.
  
  dp[i][j] = number of ways to reach (i, j)
  dp[i][j] = dp[i-1][j] + dp[i][j-1]
  
  ┌──┬──┬──┬──┐
  │ 1│ 1│ 1│ 1│    Row 0: all 1s (only right)
  ├──┼──┼──┼──┤
  │ 1│ 2│ 3│ 4│
  ├──┼──┼──┼──┤
  │ 1│ 3│ 6│10│    Answer: 10
  └──┴──┴──┴──┘
```

```
function uniquePaths(m, n):
    dp = m×n array
    for i = 0 to m-1: dp[i][0] = 1       // left column
    for j = 0 to n-1: dp[0][j] = 1       // top row
    
    for i = 1 to m-1:
        for j = 1 to n-1:
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    return dp[m-1][n-1]
```

### Minimum Path Sum

```
function minPathSum(grid):
    m = rows, n = cols
    for i = 1 to m-1: grid[i][0] += grid[i-1][0]
    for j = 1 to n-1: grid[0][j] += grid[0][j-1]
    
    for i = 1 to m-1:
        for j = 1 to n-1:
            grid[i][j] += min(grid[i-1][j], grid[i][j-1])
    
    return grid[m-1][n-1]
```

---

## 🔍 Pattern 2: Two-String DP

### Longest Common Subsequence (LCS)

```
  text1 = "abcde", text2 = "ace"
  LCS = "ace", length = 3
  
  dp[i][j] = LCS length of text1[0..i-1] and text2[0..j-1]
```

```
function longestCommonSubsequence(text1, text2):
    m = len(text1), n = len(text2)
    dp = (m+1) × (n+1) array of 0s
    
    for i = 1 to m:
        for j = 1 to n:
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1     // match
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])  // skip one
    
    return dp[m][n]
```

### Trace: text1="abcde", text2="ace"

```
      ""  a  c  e
  ""   0  0  0  0
  a    0  1  1  1
  b    0  1  1  1
  c    0  1  2  2
  d    0  1  2  2
  e    0  1  2  3  ← answer
```

---

### Edit Distance (Levenshtein)

```
  word1 = "horse", word2 = "ros"
  Operations: insert, delete, replace → minimum ops to transform
  
  dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
```

```
function minDistance(word1, word2):
    m = len(word1), n = len(word2)
    dp = (m+1) × (n+1) array
    
    for i = 0 to m: dp[i][0] = i          // delete all
    for j = 0 to n: dp[0][j] = j          // insert all
    
    for i = 1 to m:
        for j = 1 to n:
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]   // no op needed
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],            // delete
                    dp[i][j-1],            // insert
                    dp[i-1][j-1]           // replace
                )
    
    return dp[m][n]
```

### Trace: "horse" → "ros"

```
      ""  r  o  s
  ""   0  1  2  3
  h    1  1  2  3
  o    2  2  1  2
  r    3  2  2  2
  s    4  3  3  2
  e    5  4  4  3  ← answer: 3
```

---

## 🔍 Pattern 3: Knapsack

### 0/1 Knapsack

```
  Items: weights=[1,3,4,5], values=[1,4,5,7], capacity=7
  
  dp[i][w] = max value using items 0..i-1 with capacity w
```

```
function knapsack(weights, values, capacity):
    n = len(weights)
    dp = (n+1) × (capacity+1) array of 0s
    
    for i = 1 to n:
        for w = 0 to capacity:
            dp[i][w] = dp[i-1][w]                    // skip item
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w],
                    dp[i-1][w - weights[i-1]] + values[i-1])  // take item
    
    return dp[n][capacity]
```

### Space-Optimized (1D)

```
function knapsack(weights, values, capacity):
    dp = array of 0s, size capacity+1
    
    for i = 0 to n-1:
        for w = capacity down to weights[i]:    // iterate BACKWARDS
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[capacity]
```

**Why backwards?** → To ensure each item is used at most once. Forward iteration would use the same item multiple times.

---

### Unbounded Knapsack

```
  Each item can be used unlimited times.
  → Iterate w FORWARDS (allows reuse)
  
  for i = 0 to n-1:
      for w = weights[i] to capacity:          // FORWARDS
          dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
```

### Subset Sum (Special Case)

```
  "Can we pick items that sum to target?"
  → Knapsack with values = weights, check dp[target] > 0
  
  Or use boolean DP:
  dp[j] = true if subset summing to j exists
  dp[0] = true
  for num in nums:
      for j = target down to num:
          dp[j] = dp[j] OR dp[j - num]
```

---

## 📊 2D DP Pattern Summary

| Pattern | Dimensions | Examples |
|---------|-----------|----------|
| Grid (i, j) | Row × Column | Unique paths, min path sum |
| Two strings (i, j) | Length1 × Length2 | LCS, edit distance |
| Knapsack (i, w) | Items × Capacity | 0/1 knapsack, subset sum |
| Interval (i, j) | Left × Right | Matrix chain, burst balloons |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Grid paths | dp[i][j] = dp[i-1][j] + dp[i][j-1] |
| LCS | Match → diagonal+1; mismatch → max(up, left) |
| Edit distance | Match → diagonal; mismatch → 1+min(delete, insert, replace) |
| 0/1 Knapsack | Iterate weight BACKWARDS for single use |
| Unbounded Knapsack | Iterate weight FORWARDS for reuse |
| Space optimization | 2D → 1D by iterating in correct direction |

---

## ❓ Revision Questions

1. **Grid unique paths transition?** → `dp[i][j] = dp[i-1][j] + dp[i][j-1]` (from above + from left).
2. **LCS: what when characters match?** → `dp[i][j] = dp[i-1][j-1] + 1`.
3. **Edit distance: three operations in code?** → `dp[i-1][j]` (delete), `dp[i][j-1]` (insert), `dp[i-1][j-1]` (replace).
4. **0/1 Knapsack: why iterate weight backwards?** → Prevents using the same item twice in one row.
5. **How to reconstruct the LCS string?** → Backtrack from dp[m][n]: if chars match, include it and go diagonal; else go toward the larger value.

---

[← Previous: 1D DP Patterns](02-1d-dp-patterns.md) | [Next: Classic DP Problems →](04-classic-dp-problems.md)
