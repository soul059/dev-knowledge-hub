# Chapter 5: Dungeon Game

## 📋 Overview
**Dungeon Game** (LeetCode #174) flips grid DP on its head. Instead of finding minimum cost from start to end, you must find the **minimum initial health** to survive traveling from top-left to bottom-right, where health must stay **≥ 1** at all times. This requires **reverse (bottom-right to top-left)** DP.

---

## 🧠 Core Concept

```
Dungeon grid (negative = damage, positive = health orb):
  ┌─────┬─────┬─────┐
  │ -2  │ -3  │  3  │
  ├─────┼─────┼─────┤
  │ -5  │ -10 │  1  │
  ├─────┼─────┼─────┤
  │ 10  │ 30  │ -5  │
  └─────┴─────┴─────┘

  Knight starts at (0,0), princess at (2,2).
  Health must be ≥ 1 at EVERY cell (including start and end).

  Why forward DP fails:
  ┌─────────────────────────────────────────────┐
  │ Forward DP tracks "min health so far"        │
  │ but doesn't know future damage.              │
  │ A path with low total cost might have a      │
  │ deadly dip in the middle!                    │
  │                                              │
  │ Path A: sum=20, but dips to -100 midway  ✗  │
  │ Path B: sum=5,  never dips below -3      ✓  │
  └─────────────────────────────────────────────┘
```

---

## 🔍 Why Reverse DP Works

```
From the END, we know exactly how much health we need.

dp[i][j] = minimum health needed WHEN ENTERING cell (i,j)
           to be able to reach (m-1, n-1) alive.

At the princess cell (m-1, n-1):
  We need to have ≥ 1 health AFTER stepping on it.
  So we need: hp + dungeon[m-1][n-1] ≥ 1
  Therefore: hp ≥ 1 - dungeon[m-1][n-1]
  But hp must be ≥ 1:  hp = max(1, 1 - dungeon[m-1][n-1])

General recurrence (from bottom-right to top-left):
  dp[i][j] = max(1, min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j])
              ↑ must be alive  ↑ best direction     ↑ damage/heal here
```

---

## 🔨 Solution

```
function calculateMinimumHP(dungeon):
    m = rows, n = cols
    dp = 2D array of size m × n
    
    // Base: bottom-right corner
    dp[m-1][n-1] = max(1, 1 - dungeon[m-1][n-1])
    
    // Last row: can only go right
    for j = n-2 down to 0:
        dp[m-1][j] = max(1, dp[m-1][j+1] - dungeon[m-1][j])
    
    // Last column: can only go down
    for i = m-2 down to 0:
        dp[i][n-1] = max(1, dp[i+1][n-1] - dungeon[i][n-1])
    
    // Fill rest (bottom-right to top-left)
    for i = m-2 down to 0:
        for j = n-2 down to 0:
            minHpNeeded = min(dp[i+1][j], dp[i][j+1])
            dp[i][j] = max(1, minHpNeeded - dungeon[i][j])
    
    return dp[0][0]
```

---

## 🔬 Trace

```
Dungeon:                 dp table (min health needed):
  -2  -3   3               7   5   2
  -5 -10   1     →         6  11   5
  10  30  -5               1   1   6

Step-by-step (bottom-right to top-left):
  dp[2][2] = max(1, 1-(-5)) = max(1, 6) = 6
  dp[2][1] = max(1, 6-30) = max(1, -24) = 1
  dp[2][0] = max(1, 1-10) = max(1, -9) = 1
  dp[1][2] = max(1, 6-1) = max(1, 5) = 5
  dp[1][1] = max(1, min(1,5)-(-10)) = max(1, 1+10) = 11
  dp[1][0] = max(1, min(11,5)-(-5)) = max(1, 5+5) = 10? 
  
  Wait, let me recalculate:
  dp[1][0] = max(1, min(dp[2][0], dp[1][1]) - dungeon[1][0])
           = max(1, min(1, 11) - (-5))
           = max(1, 1 + 5) = 6
  
  dp[0][2] = max(1, dp[1][2] - dungeon[0][2])
           = max(1, 5 - 3) = max(1, 2) = 2
  dp[0][1] = max(1, min(dp[1][1], dp[0][2]) - dungeon[0][1])
           = max(1, min(11, 2) - (-3))
           = max(1, 2 + 3) = 5
  dp[0][0] = max(1, min(dp[1][0], dp[0][1]) - dungeon[0][0])
           = max(1, min(6, 5) - (-2))
           = max(1, 5 + 2) = 7

Answer: 7

Verification — Path: (0,0)→(0,1)→(0,2)→(1,2)→(2,2)
  Start with 7 health:
  Cell (0,0): 7 + (-2)  = 5  ≥ 1 ✓
  Cell (0,1): 5 + (-3)  = 2  ≥ 1 ✓
  Cell (0,2): 2 + 3     = 5  ≥ 1 ✓
  Cell (1,2): 5 + 1     = 6  ≥ 1 ✓
  Cell (2,2): 6 + (-5)  = 1  ≥ 1 ✓  Survives!
```

---

## 💻 Space-Optimized Version

```
function calculateMinimumHP(dungeon):
    m = rows, n = cols
    dp = array of size n
    
    // Last row
    dp[n-1] = max(1, 1 - dungeon[m-1][n-1])
    for j = n-2 down to 0:
        dp[j] = max(1, dp[j+1] - dungeon[m-1][j])
    
    // Remaining rows
    for i = m-2 down to 0:
        dp[n-1] = max(1, dp[n-1] - dungeon[i][n-1])
        for j = n-2 down to 0:
            dp[j] = max(1, min(dp[j], dp[j+1]) - dungeon[i][j])
            //              ↑ below   ↑ right
    
    return dp[0]

Time: O(m·n), Space: O(n)
```

---

## 💡 Key Takeaways

```
┌─────────────────────────────────────────────────────┐
│ When to use REVERSE DP:                             │
│                                                     │
│ 1. The constraint is about the JOURNEY, not just    │
│    the destination (health ≥ 1 at EVERY step)       │
│                                                     │
│ 2. Forward DP can't capture future requirements     │
│    (you'd need to track both sum and min, making    │
│    the state space too large)                       │
│                                                     │
│ 3. From the end, requirements are deterministic:    │
│    "I need at least X health to survive from here"  │
│                                                     │
│ Pattern: if question asks "minimum starting value   │
│ to survive", think REVERSE.                         │
└─────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min health to enter (i,j) and survive to end |
| **Recurrence** | dp[i][j] = max(1, min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j]) |
| **Direction** | Bottom-right to top-left (reverse) |
| **Key Constraint** | Health ≥ 1 at every step |
| **Complexity** | O(m·n) time, O(n) space |
| **Pattern** | Reverse DP for survival/threshold problems |

---

## ❓ Quick Revision Questions

1. **Why does forward DP fail for the Dungeon Game?**
2. **What does dp[i][j] represent in the reverse formulation?**
3. **Why do we use max(1, ...) in the recurrence?**
4. **What is the fill order and why?**
5. **How do you verify the answer is correct?**
6. **Name another problem that requires reverse DP.**

---

[← Previous: Triangle](04-triangle.md) | [Next: Cherry Pickup →](06-cherry-pickup.md)

[← Back to README](../README.md)
