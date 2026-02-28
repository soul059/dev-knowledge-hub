# Chapter 1: Space Optimization in DP

## 📋 Overview
Most DP solutions can have their **space reduced** without changing time complexity. This chapter covers the most common technique: recognizing that each row only depends on the **previous row**, allowing **1D instead of 2D** arrays.

---

## 🧠 Core Principle

```
If dp[i] depends only on dp[i-1]:
  Keep only 2 rows (current and previous)
  → O(n·m) → O(m) space

If dp[i][j] depends only on dp[i-1][j] and dp[i][j-1]:
  Keep only 1 row, update left to right
  → O(n·m) → O(m) space
```

---

## 📊 Pattern 1: Two-Row Technique

### Before (2D)
```
function lcs(A, B):
    dp = 2D array [n+1][m+1] = 0
    for i = 1 to n:
        for j = 1 to m:
            if A[i-1] == B[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[n][m]

Space: O(n · m)
```

### After (Two Rows)
```
function lcs_optimized(A, B):
    prev = array [m+1] = 0
    curr = array [m+1] = 0
    
    for i = 1 to n:
        for j = 1 to m:
            if A[i-1] == B[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        swap(prev, curr)
        fill(curr, 0)
    
    return prev[m]

Space: O(m)
```

### Visualization
```
Full table:        Two-row:
┌───────────┐      Only keep:
│ . . . . . │      prev: [. . . . .]
│ . . . . . │      curr: [. . . . .]
│ . . . . . │ →    
│ . . . . . │      Swap after each row
│ . . . . . │      
└───────────┘      Space: O(m) vs O(n·m)
```

---

## 📊 Pattern 2: Single-Row In-Place Update

### When it works
```
dp[i][j] depends on:
  dp[i-1][j]   = prev[j]   = current value BEFORE update
  dp[i][j-1]   = curr[j-1] = value just computed

Can reuse single array!
  dp[j] already holds dp[i-1][j] before update
  dp[j-1] already updated to dp[i][j-1]
```

### Example: Unique Paths
```
Before:
  dp[i][j] = dp[i-1][j] + dp[i][j-1]

After (single row):
  dp = array[m] = 1    // first row all 1s
  for i = 1 to n-1:
      for j = 1 to m-1:
          dp[j] = dp[j] + dp[j-1]
          //      ↑ old value (from row above)
          //              ↑ new value (from left)
  return dp[m-1]

Space: O(m)
```

---

## 📊 Pattern 3: Knapsack Space Optimization

### 0/1 Knapsack: Right-to-Left
```
dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])

Both terms use row i-1 → single row, iterate RIGHT to LEFT

for i = 0 to n-1:
    for w = W down to wt[i]:    // RIGHT TO LEFT
        dp[w] = max(dp[w], dp[w - wt[i]] + val[i])

Why right-to-left?
  dp[w-wt[i]] should be the i-1 value (not yet updated)
  Processing w from high to low ensures this.
  
  ← ← ← ← ← direction
  dp: [__, __, __, dp[w-wt], __, dp[w]]
                   ↑ not yet updated (row i-1 value)
```

### Unbounded Knapsack: Left-to-Right
```
dp[i][w] = max(dp[i-1][w], dp[i][w-wt[i]] + val[i])
                             ↑ same row i (can reuse item)

for i = 0 to n-1:
    for w = wt[i] to W:    // LEFT TO RIGHT
        dp[w] = max(dp[w], dp[w - wt[i]] + val[i])

Left-to-right: dp[w-wt[i]] is already updated (row i value)
  → allows multiple uses of item i ✓
```

### Comparison
```
┌─────────────────────────────────────────────────┐
│  Problem        │ Direction    │ Reason          │
│─────────────────┼──────────────┼─────────────────│
│  0/1 Knapsack   │ Right→Left  │ Use each once    │
│  Unbounded      │ Left→Right  │ Reuse allowed    │
│  Coin Change    │ Left→Right  │ Unlimited coins   │
│  Subset Sum     │ Right→Left  │ Each element once │
└─────────────────────────────────────────────────┘
```

---

## 📊 Pattern 4: Variables Instead of Arrays

### Fibonacci-Type Problems
```
dp[i] = dp[i-1] + dp[i-2]

Only need 2 variables:
  a, b = 0, 1
  for i = 2 to n:
      a, b = b, a + b
  return b

Space: O(1)
```

### Kadane's Algorithm
```
dp[i] = max(nums[i], dp[i-1] + nums[i])

Single variable:
  maxEndingHere = nums[0]
  maxSoFar = nums[0]
  for i = 1 to n-1:
      maxEndingHere = max(nums[i], maxEndingHere + nums[i])
      maxSoFar = max(maxSoFar, maxEndingHere)

Space: O(1)
```

### Stock Problems
```
dp[hold], dp[notHold] → just 2 variables

hold = -INF
notHold = 0
for price in prices:
    hold = max(hold, notHold - price)
    notHold = max(notHold, hold + price)

Space: O(1)
```

---

## ⚠️ Pitfall: Losing the Diagonal

```
LCS single-row needs care:

dp[i][j] uses dp[i-1][j-1] (diagonal)

When updating dp[j], the old dp[j-1] is already overwritten!

Fix: save the diagonal value before overwriting.

prev_diag = 0
for j = 1 to m:
    temp = dp[j]          // save dp[i-1][j]
    if A[i-1] == B[j-1]:
        dp[j] = prev_diag + 1
    else:
        dp[j] = max(dp[j], dp[j-1])
    prev_diag = temp      // for next iteration
```

---

## 📊 Space Optimization Decision Table

| Original | Dependency | Optimized | Technique |
|----------|-----------|-----------|-----------|
| O(n·m) | Previous row | O(m) | Two rows / single row |
| O(n·m) | Row + diagonal | O(m) | Save diagonal variable |
| O(n) | Last 1 element | O(1) | Single variable |
| O(n) | Last 2 elements | O(1) | Two variables |
| O(n) | Last K elements | O(K) | Circular buffer |
| O(n·m·k) | Previous k | O(m·k) | Rolling layer |

---

## ⚡ Tradeoff: Space vs Reconstruction

```
Space optimization drops old rows → can't backtrack!

Solutions:
1. Don't optimize if you need the path
2. Hirschberg's algorithm: 
   Reconstruct LCS in O(m) space + O(n·m) time
   Divide and conquer on rows
3. Store decisions separately (O(n·m) for decisions)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Two-row** | Keep prev[] and curr[], swap each iteration |
| **Single-row** | In-place update when dependencies allow |
| **Direction** | Right→left for 0/1, left→right for unbounded |
| **Variables** | O(1) space for Fibonacci-type recurrences |
| **Diagonal save** | Store dp[i-1][j-1] before overwriting |
| **Tradeoff** | Loses ability to reconstruct solution path |

---

## ❓ Quick Revision Questions

1. **When can you reduce 2D DP to a single row?**
2. **Why does 0/1 knapsack iterate right-to-left?**
3. **How do you save the diagonal value in single-row LCS?**
4. **What is the space optimization for Fibonacci-type recurrences?**
5. **What do you lose by space-optimizing?**
6. **How does Hirschberg's algorithm recover the LCS in O(m) space?**

---

[← Previous Unit: Bitmask DP](../12-Bitmask-DP/06-special-permutation.md) | [Next: Rolling Array →](02-rolling-array.md)

[← Back to README](../README.md)
