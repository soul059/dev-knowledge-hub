# Chapter 5: Determine Order of Computation

## 📋 Overview
In tabulation (bottom-up DP), the **order of computation** determines the sequence in which we fill the DP table. The rule is simple: when computing `dp[state]`, all states it depends on must already be computed.

---

## 🧠 Core Principle

```
┌─────────────────────────────────────────────────────────┐
│          ORDER OF COMPUTATION RULE                      │
│                                                         │
│  Before computing dp[i], ALL states that dp[i]          │
│  depends on MUST already be computed.                   │
│                                                         │
│  Dependencies ──► must be computed FIRST                │
│  Current state ──► computed AFTER dependencies          │
│                                                         │
│  Example: dp[i] = dp[i-1] + dp[i-2]                    │
│                                                         │
│  dp[0] ──► dp[1] ──► dp[2] ──► dp[3] ──► dp[4]        │
│  base      base      needs     needs     needs         │
│                       0,1       1,2       2,3           │
│                                                         │
│  Fill order: LEFT to RIGHT (i = 0, 1, 2, ...)          │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Determining Order from Dependencies

### 1D Dependencies

```
Case: dp[i] depends on dp[i-1], dp[i-2]
Order: i = 0 → n  (left to right)

  ┌───┬───┬───┬───┬───┬───┐
  │ B │ B │ ← │ ← │ ← │ ? │  B = base, ← = depends on left
  └───┴───┴───┴───┴───┴───┘
  Fill: ──────────────────►

Case: dp[i] depends on dp[i+1], dp[i+2]
Order: i = n → 0  (right to left)

  ┌───┬───┬───┬───┬───┬───┐
  │ ? │ → │ → │ → │ B │ B │  → = depends on right
  └───┴───┴───┴───┴───┴───┘
  Fill: ◄──────────────────
```

### 2D Dependencies — Grid DP

```
dp[i][j] depends on dp[i-1][j] (above) and dp[i][j-1] (left)

  ┌─────┬─────┬─────┬─────┐
  │  B  │  B  │  B  │  B  │  ← base row
  ├─────┼─────┼─────┼─────┤
  │  B  │  ↑← │  ↑← │  ↑← │  ↑ = from above, ← = from left
  ├─────┼─────┼─────┼─────┤
  │  B  │  ↑← │  ↑← │  ↑← │
  ├─────┼─────┼─────┼─────┤
  │  B  │  ↑← │  ↑← │  ↑← │  ← answer here
  └─────┴─────┴─────┴─────┘
  
  Fill order: Row by row, left to right
  for i = 0 to m:
      for j = 0 to n:
          compute dp[i][j]
```

### 2D Dependencies — String DP (LCS)

```
dp[i][j] depends on dp[i-1][j-1], dp[i-1][j], dp[i][j-1]

  ┌─────┬─────┬─────┬─────┐
  │  B  │  B  │  B  │  B  │
  ├─────┼─────┼─────┼─────┤
  │  B  │ ↖↑← │ ↖↑← │ ↖↑← │  ↖ = diagonal, ↑ = above, ← = left
  ├─────┼─────┼─────┼─────┤
  │  B  │ ↖↑← │ ↖↑← │ ↖↑← │
  └─────┴─────┴─────┴─────┘
  
  Fill order: Row by row, left to right
  (Same as grid — all three dependencies are already computed)
```

### 2D Dependencies — Interval DP

```
dp[i][j] depends on dp[i][k] and dp[k+1][j] for all i ≤ k < j
(subintervals of [i,j])

  j→  0   1   2   3   4
i↓ ┌───┬───┬───┬───┬───┐
 0 │ B │ 1 │ 2 │ 3 │ 4 │  Numbers = order of computation
   ├───┼───┼───┼───┼───┤  (by diagonal / length)
 1 │   │ B │ 1 │ 2 │ 3 │
   ├───┼───┼───┼───┼───┤
 2 │   │   │ B │ 1 │ 2 │
   ├───┼───┼───┼───┼───┤
 3 │   │   │   │ B │ 1 │
   ├───┼───┼───┼───┼───┤
 4 │   │   │   │   │ B │
   └───┴───┴───┴───┴───┘
   
  Fill order: By increasing LENGTH of interval
  for len = 2 to n:
      for i = 0 to n-len:
          j = i + len - 1
          compute dp[i][j]
```

---

## 📐 Common Fill Orders

```
┌──────────────┬──────────────────────────────────────────┐
│ Problem Type │ Fill Order                               │
├──────────────┼──────────────────────────────────────────┤
│ 1D (forward) │ i = 0 → n                               │
│              │ ────────────────►                         │
│              │                                          │
│ 1D (backward)│ i = n → 0                               │
│              │ ◄────────────────                         │
│              │                                          │
│ 2D Grid      │ Row by row, left to right                │
│              │ ┌──►──►──►                               │
│              │ ├──►──►──►                               │
│              │ └──►──►──►                               │
│              │                                          │
│ 2D Interval  │ By diagonal (increasing gap)             │
│              │ ╲                                        │
│              │  ╲  (gap=1, then gap=2, ...)              │
│              │   ╲                                      │
│              │                                          │
│ Knapsack     │ Row by row (items), within each:         │
│              │ Forward (unbounded) or either (0/1)      │
│              │                                          │
│ Bottom-right │ i = m → 0, j = n → 0                    │
│ to top-left  │ ◄──◄──◄──┐                              │
│              │ ◄──◄──◄──┤                              │
│              │ ◄──◄──◄──┘                              │
└──────────────┴──────────────────────────────────────────┘
```

---

## 🧪 Example: Determining Order for Edit Distance

```
Recurrence:
  if s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1]
  else:
      dp[i][j] = 1 + min(dp[i-1][j],      // delete
                         dp[i][j-1],        // insert
                         dp[i-1][j-1])      // replace

Dependencies for dp[i][j]:
  dp[i-1][j-1]  ← row above, col to left
  dp[i-1][j]    ← row above, same col
  dp[i][j-1]    ← same row, col to left

  dp[i-1][j-1]    dp[i-1][j]
       ↘              ↓
  dp[i][j-1]  →  dp[i][j]

All three are computed before (i,j) if we go row by row, left to right.

Fill order: 
  for i = 0 to m:
      for j = 0 to n:
```

---

## ⚠️ Special Case: When Order Matters

### 0/1 Knapsack vs Unbounded Knapsack

```
0/1 KNAPSACK (each item used at most once):
  dp[i][w] depends on dp[i-1][w] and dp[i-1][w-wt[i]]
  → Process items in order, use PREVIOUS row
  
  Space-optimized: iterate w from RIGHT to LEFT
    for w = W down to wt[i]:
        dp[w] = max(dp[w], dp[w-wt[i]] + val[i])
    
    Why? dp[w-wt[i]] must still be from previous item (row i-1)
    Right-to-left ensures we don't overwrite values we still need.

UNBOUNDED KNAPSACK (each item used unlimited times):
  dp[w] depends on dp[w-wt[i]] from CURRENT item
  → Iterate w from LEFT to RIGHT
  
    for w = wt[i] to W:
        dp[w] = max(dp[w], dp[w-wt[i]] + val[i])
    
    Why? dp[w-wt[i]] should include current item too.
    Left-to-right means dp[w-wt[i]] may already include item i.

Visual:
  0/1:        RIGHT to LEFT    ◄──◄──◄──◄──
  Unbounded:  LEFT to RIGHT    ──►──►──►──►
```

---

## 💡 Key Insight: Memoization Avoids This

```
┌─────────────────────────────────────────────────────────┐
│  With MEMOIZATION, you don't need to determine order!   │
│                                                         │
│  The recursion naturally computes dependencies first:    │
│                                                         │
│  solve(5:                                               │
│    needs solve(4) → computed first automatically         │
│    needs solve(3) → computed first automatically         │
│                                                         │
│  This is why memoization is often EASIER to implement.  │
│  But tabulation is typically FASTER in practice.         │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Fill Order** | Sequence in which DP table is filled |
| **Rule** | Compute dependencies before current state |
| **1D Forward** | Left to right (dp[i] depends on dp[i-1]) |
| **1D Backward** | Right to left (dp[i] depends on dp[i+1]) |
| **2D Grid** | Row by row, left to right |
| **2D Interval** | By increasing interval length (diagonal) |
| **Critical Case** | 0/1 Knapsack (right-to-left) vs Unbounded (left-to-right) |

---

## ❓ Quick Revision Questions

1. **What is the fundamental rule for determining fill order?**
2. **Why does Interval DP fill by increasing gap length?**
3. **In space-optimized 0/1 Knapsack, why must we iterate weights right-to-left?**
4. **What fill order does Edit Distance use? Draw the dependency arrows.**
5. **Why does memoization not require you to think about fill order?**
6. **Give an example where filling left-to-right gives wrong results.**

---

[← Previous: Identify Base Cases](04-identify-base-cases.md) | [Next: Optimize Space →](06-optimize-space.md)

[← Back to README](../README.md)
