# Chapter 6: Optimize Space

## 📋 Overview
Many DP solutions use more memory than necessary. **Space optimization** techniques reduce the space complexity — often from O(n²) to O(n) or from O(n) to O(1) — by observing that we only need a few previous states, not the entire table.

---

## 🧠 Core Insight

```
┌─────────────────────────────────────────────────────────┐
│              SPACE OPTIMIZATION INSIGHT                  │
│                                                         │
│  In most DP problems, dp[i] only depends on a few       │
│  previous states (dp[i-1], dp[i-2], etc.)               │
│                                                         │
│  Full table (O(n)):                                     │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┐                     │
│  │ 0 │ 1 │ 1 │ 2 │ 3 │ 5 │ 8 │ 13│  All stored        │
│  └───┴───┴───┴───┴───┴───┴───┴───┘                     │
│                                                         │
│  Optimized (O(1)):                                      │
│  Only keep last 2 values!                               │
│  ┌───┬───┐                                              │
│  │ 8 │ 13│  prev, curr                                  │
│  └───┴───┘                                              │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Optimization Levels

```
Level 0: Full Table     → O(n) or O(m×n) space
Level 1: Rolling Array  → O(n) from O(m×n), or O(1) from O(n)
Level 2: Two Variables  → O(1) space (best case)

┌─────────────────────────────────────────────────────────┐
│  LEVEL 0: Full 2D Table                                 │
│  ┌───┬───┬───┬───┐                                     │
│  │   │   │   │   │  All rows stored                     │
│  ├───┼───┼───┼───┤  Space: O(m×n)                       │
│  │   │   │   │   │                                     │
│  ├───┼───┼───┼───┤                                     │
│  │   │   │   │   │                                     │
│  └───┴───┴───┴───┘                                     │
│                                                         │
│  LEVEL 1: Two Rows (Rolling Array)                      │
│  ┌───┬───┬───┬───┐                                     │
│  │prev row       │  Only 2 rows stored                  │
│  ├───┼───┼───┼───┤  Space: O(n)                         │
│  │curr row       │                                     │
│  └───┴───┴───┴───┘                                     │
│                                                         │
│  LEVEL 2: Single Row (In-place update)                  │
│  ┌───┬───┬───┬───┐                                     │
│  │ one row       │  Space: O(n)                         │
│  └───┴───┴───┴───┘  (sometimes even better)             │
└─────────────────────────────────────────────────────────┘
```

---

## 📐 Technique 1: Two Variables (1D → O(1))

### Example: Fibonacci
```
BEFORE (O(n) space):
  dp = array[n+1]
  dp[0] = 0, dp[1] = 1
  for i = 2 to n:
      dp[i] = dp[i-1] + dp[i-2]
  return dp[n]

OBSERVATION: dp[i] only needs dp[i-1] and dp[i-2]

AFTER (O(1) space):
  prev2 = 0, prev1 = 1
  for i = 2 to n:
      curr = prev1 + prev2
      prev2 = prev1
      prev1 = curr
  return prev1

Trace for n=6:
  Step │ prev2 │ prev1 │ curr
  ─────┼───────┼───────┼──────
  init │   0   │   1   │  -
  i=2  │   1   │   1   │  1
  i=3  │   1   │   2   │  2
  i=4  │   2   │   3   │  3
  i=5  │   3   │   5   │  5
  i=6  │   5   │   8   │  8 ◄── answer
```

### Example: House Robber
```
BEFORE: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
AFTER:
  prev2 = 0, prev1 = 0
  for each num in nums:
      curr = max(prev1, prev2 + num)
      prev2 = prev1
      prev1 = curr
  return prev1
```

---

## 📐 Technique 2: Rolling Array (2D → 1D)

### Example: Grid Unique Paths
```
BEFORE (O(m×n) space):
  dp[m][n]
  for i = 0 to m-1:
      for j = 0 to n-1:
          if i==0 or j==0: dp[i][j] = 1
          else: dp[i][j] = dp[i-1][j] + dp[i][j-1]

OBSERVATION: Row i only depends on row i-1

AFTER (O(n) space):
  dp = array[n], all 1s
  for i = 1 to m-1:
      for j = 1 to n-1:
          dp[j] = dp[j] + dp[j-1]
          //       ↑         ↑
          //    from above   from left
          //    (prev row)   (curr row, already updated)

Trace for 3×4 grid:
  Initial: dp = [1, 1, 1, 1]
  
  i=1: j=1: dp[1] = 1+1 = 2
       j=2: dp[2] = 1+2 = 3
       j=3: dp[3] = 1+3 = 4
       dp = [1, 2, 3, 4]
  
  i=2: j=1: dp[1] = 2+1 = 3
       j=2: dp[2] = 3+3 = 6
       j=3: dp[3] = 4+6 = 10
       dp = [1, 3, 6, 10] ← answer: dp[3] = 10
```

### Example: LCS (2D → 1D)
```
BEFORE: dp[i][j] uses dp[i-1][j-1], dp[i-1][j], dp[i][j-1]
  Need diagonal + above + left

AFTER: Single row + one variable for diagonal
  dp = array[n+1] of 0s
  for i = 1 to m:
      prev = 0  // stores dp[i-1][j-1]
      for j = 1 to n:
          temp = dp[j]  // save before overwrite (becomes diagonal for next j)
          if s1[i-1] == s2[j-1]:
              dp[j] = prev + 1
          else:
              dp[j] = max(dp[j], dp[j-1])
          prev = temp
```

---

## 📐 Technique 3: In-Place Update (0/1 Knapsack)

```
FULL TABLE (O(n×W)):
  for i = 1 to n:
      for w = 0 to W:
          if wt[i] <= w:
              dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])
          else:
              dp[i][w] = dp[i-1][w]

OPTIMIZED (O(W)):
  dp = array[W+1] of 0s
  for i = 1 to n:
      for w = W down to wt[i]:      ← RIGHT TO LEFT!
          dp[w] = max(dp[w], dp[w-wt[i]] + val[i])

Why right-to-left?
  ┌───┬───┬───┬───┬───┬───┬───┐
  │dp0│dp1│dp2│dp3│dp4│dp5│dp6│
  └───┴───┴───┴───┴───┴───┴───┘
                          ◄──── process direction
  
  When computing dp[6], dp[3] still has value from previous item.
  If we went left-to-right, dp[3] would already be updated
  (allowing item i to be used twice — wrong for 0/1 knapsack!)
```

---

## 🧪 Space Optimization Cheat Sheet

```
┌──────────────────┬─────────────┬────────────┬─────────────────┐
│ Problem          │ Original    │ Optimized  │ Technique       │
├──────────────────┼─────────────┼────────────┼─────────────────┤
│ Fibonacci        │ O(n)        │ O(1)       │ Two variables   │
│ Climbing Stairs  │ O(n)        │ O(1)       │ Two variables   │
│ House Robber     │ O(n)        │ O(1)       │ Two variables   │
│ Unique Paths     │ O(m×n)      │ O(n)       │ Rolling array   │
│ Min Path Sum     │ O(m×n)      │ O(n)       │ Rolling array   │
│ LCS              │ O(m×n)      │ O(n)       │ 1 row + diagonal│
│ Edit Distance    │ O(m×n)      │ O(n)       │ 1 row + diagonal│
│ 0/1 Knapsack     │ O(n×W)      │ O(W)       │ Reverse iterate │
│ Coin Change      │ O(n×amount) │ O(amount)  │ Forward iterate │
│ Triangle         │ O(n²)       │ O(n)       │ Bottom-up row   │
└──────────────────┴─────────────┴────────────┴─────────────────┘
```

---

## ⚠️ When Space Optimization is NOT Possible

```
1. When you need to RECONSTRUCT the solution (not just the value)
   → Need the full table to trace back the path
   → Example: Finding the actual LCS string, not just its length

2. When dependencies span MORE than adjacent rows
   → dp[i][j] depends on dp[i-3][j-2]
   → Need to keep 3+ rows

3. Interval DP (usually)
   → dp[i][j] depends on all dp[i][k] for i < k < j
   → Full table typically needed

4. When debugging
   → Full table is easier to inspect and verify
   → Optimize AFTER correctness is confirmed
```

---

## 📊 Summary Table

| Technique | From | To | When to Use |
|-----------|------|----|-------------|
| **Two Variables** | O(n) | O(1) | dp[i] depends on dp[i-1], dp[i-2] only |
| **Rolling Array** | O(m×n) | O(n) | Row i depends only on row i-1 |
| **Single Row + Var** | O(m×n) | O(n) | Need diagonal element too |
| **Reverse Iteration** | O(n×W) | O(W) | 0/1 Knapsack (prevent reuse) |
| **Forward Iteration** | O(n×W) | O(W) | Unbounded Knapsack (allow reuse) |

---

## ❓ Quick Revision Questions

1. **What's the key observation that enables space optimization?**
2. **Why can't we always optimize space — when is the full table needed?**
3. **For Fibonacci, show how to go from O(n) to O(1) space.**
4. **In the 0/1 Knapsack 1D optimization, why iterate right-to-left?**
5. **How do you handle the diagonal dependency in LCS when using a single row?**
6. **If dp[i] depends on dp[i-1], dp[i-2], and dp[i-3], what's the minimum space?**

---

[← Previous: Order of Computation](05-order-of-computation.md) | [Next Unit: 1D DP Problems →](../03-1D-DP-Problems/01-climbing-stairs.md)

[← Back to README](../README.md)
