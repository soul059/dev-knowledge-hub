# Chapter 4: Identify Base Cases

## 📋 Overview
**Base cases** are the smallest subproblems that can be solved directly without further recursion. They anchor the recurrence relation and ensure the DP computation terminates correctly. Getting base cases wrong is one of the most common sources of bugs in DP solutions.

---

## 🧠 Core Concept

```
┌─────────────────────────────────────────────────────────┐
│                    BASE CASES                           │
│                                                         │
│  The SIMPLEST subproblems with KNOWN answers            │
│                                                         │
│  Recurrence: dp[i] = f(dp[i-1], dp[i-2], ...)          │
│                              │         │                │
│              What happens when i-1 or i-2 goes below 0? │
│              → That's where base cases are needed!       │
│                                                         │
│  ┌─────────────────────────────────────┐                │
│  │  dp[base] = known_value             │                │
│  │           ↓                         │                │
│  │  dp[base+1] = f(dp[base], ...)      │                │
│  │           ↓                         │                │
│  │  dp[base+2] = f(dp[base+1], ...)    │                │
│  │           ↓                         │                │
│  │  ... builds up to dp[n]             │                │
│  └─────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 How to Find Base Cases

```
Method 1: Follow the recurrence until it can't recurse
  dp[i] = dp[i-1] + dp[i-2]
  → dp[1] = dp[0] + dp[-1]  ← invalid! 
  → Base: dp[0] and dp[1] must be defined directly

Method 2: Think about trivial inputs
  "Min coins for amount 0" → dp[0] = 0
  "Paths in 1×1 grid" → dp[0][0] = 1
  "LCS of empty string with anything" → dp[0][j] = 0

Method 3: What does the problem say about edge cases?
  "You start at step 0" → dp[0] = known
  "Empty subsequence has length 0" → dp[0] = 0
```

---

## 📐 Base Cases for Common Problems

### 1. Fibonacci
```
dp[i] = dp[i-1] + dp[i-2]

Base cases: dp[0] = 0, dp[1] = 1

Why? The recurrence needs two previous values.
     The smallest Fibonacci numbers are defined directly.
```

### 2. Climbing Stairs
```
dp[i] = dp[i-1] + dp[i-2]

Base cases: dp[0] = 1, dp[1] = 1

Why? dp[0] = 1: There's 1 way to stay at ground (do nothing)
     dp[1] = 1: There's 1 way to reach step 1 (one single step)
     
⚠️ Note: dp[0] = 1, NOT 0! 
   "1 way to reach step 0" = doing nothing (a valid way)
```

### 3. Coin Change (Min Coins)
```
dp[i] = min(dp[i-c] + 1) for each coin c

Base cases: dp[0] = 0
            dp[i] = ∞ for i < 0 (or handle by checking i >= c)

Why? dp[0] = 0: 0 coins needed to make amount 0
     dp[negative] = impossible, so set to infinity
```

### 4. House Robber
```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])

Base cases: dp[0] = nums[0]
            dp[1] = max(nums[0], nums[1])

Why? dp[0]: Only one house → rob it
     dp[1]: Two houses → rob the more valuable one
```

### 5. Grid Paths (Unique Paths)
```
dp[i][j] = dp[i-1][j] + dp[i][j-1]

Base cases: dp[0][j] = 1 for all j  (first row: only right)
            dp[i][0] = 1 for all i  (first col: only down)

Why? Only one way to reach any cell in first row/column
     (straight line from start)
     
┌───┬───┬───┬───┐
│ 1 │ 1 │ 1 │ 1 │  ← all 1s (base)
├───┼───┼───┼───┤
│ 1 │   │   │   │
├───┼───┼───┼───┤
│ 1 │   │   │   │
└───┴───┴───┴───┘
 ↑ all 1s (base)
```

### 6. Longest Common Subsequence
```
if s1[i-1] == s2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])

Base cases: dp[0][j] = 0 for all j  (empty s1 → LCS = 0)
            dp[i][0] = 0 for all i  (empty s2 → LCS = 0)

Why? LCS of any string with empty string is 0

      ""  A  C  E
  "" [ 0  0  0  0 ]  ← base row
  A  [ 0  .  .  . ]
  B  [ 0  .  .  . ]
  C  [ 0  .  .  . ]
       ↑ base column
```

### 7. 0/1 Knapsack
```
dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])

Base cases: dp[0][w] = val[0] if w >= wt[0], else 0
            dp[i][0] = 0 for all i  (capacity 0 → value 0)

Why? With 0 capacity, can't take anything
     With only item 0, take it only if it fits
```

---

## ⚠️ Common Base Case Mistakes

```
┌─────────────────────────────────────────────────────────┐
│  MISTAKE 1: Wrong initial value for counting problems   │
│                                                         │
│  Climbing stairs: dp[0] = 0 ← WRONG!                   │
│  Should be dp[0] = 1 (one way: do nothing)              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 2: Wrong initial value for min problems        │
│                                                         │
│  Coin change: dp[i] = 0 initially ← WRONG!             │
│  Should be dp[i] = ∞ (haven't found a way yet)         │
│  Only dp[0] = 0                                         │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 3: Missing boundary conditions in 2D           │
│                                                         │
│  Grid paths: Forgetting to set first row/column         │
│  Results in dp[i][j] = 0 + 0 everywhere!               │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 4: Off-by-one in state definition              │
│                                                         │
│  dp[i] = answer for first i items (1-indexed)           │
│  vs dp[i] = answer ending at index i (0-indexed)        │
│  Base cases differ!                                      │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 5: Not handling "impossible" states            │
│                                                         │
│  Coin change: what if amount can't be made?             │
│  Need to check if dp[amount] is still ∞                │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 Base Case Initialization Patterns

```
Problem Type          │  Base Case Value  │  Other States Init
──────────────────────┼───────────────────┼────────────────────
Counting (# of ways)  │  dp[0] = 1        │  dp[i] = 0
Minimization          │  dp[0] = 0        │  dp[i] = ∞
Maximization          │  dp[0] = 0        │  dp[i] = -∞ or 0
Boolean (possible?)   │  dp[0] = true     │  dp[i] = false
Sequence length       │  dp[0] = 0        │  dp[i] = 0

Key Rule: Initialize NON-base states to the 
"identity element" of your combination operator:
  min → ∞    (anything is better than infinity)
  max → -∞   (anything is better than neg infinity)
  +   → 0    (adding 0 doesn't change result)
  ||  → false (OR with false doesn't change result)
```

---

## 🧪 Verification Technique

```
Step 1: Pick the smallest valid input
Step 2: Compute answer by hand
Step 3: Verify your base case gives same answer

Example: Coin Change, coins = [1, 5], amount = 0
  Hand computation: 0 coins needed for amount 0
  Base case: dp[0] = 0 ✓

Example: Climbing Stairs, n = 0
  Hand computation: 1 way (do nothing)
  Base case: dp[0] = 1 ✓

Example: Climbing Stairs, n = 1
  Hand computation: 1 way (single step)
  Base case: dp[1] = 1 ✓
  Verify: dp[1] = dp[0] + dp[-1]? NO! dp[1] must be base.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Base Cases** | Smallest subproblems with directly known answers |
| **Purpose** | Anchor the recurrence, terminate recursion |
| **Finding Them** | Follow recurrence to smallest inputs, think of trivial cases |
| **Counting Init** | Base = 1 (or known count), others = 0 |
| **Minimization Init** | Base = 0, others = ∞ |
| **Maximization Init** | Base = 0, others = -∞ |
| **Common Errors** | Wrong initial values, missing boundaries, off-by-one |

---

## ❓ Quick Revision Questions

1. **Why is dp[0] = 1 for climbing stairs but dp[0] = 0 for coin change (min coins)?**
2. **What should you initialize non-base states to for a minimization DP problem?**
3. **In a 2D grid DP, what are the typical base cases and why?**
4. **How do you verify that your base cases are correct?**
5. **What happens if you set dp[0] = 0 instead of dp[0] = 1 for a counting problem?**
6. **For the Knapsack problem, what's the base case when capacity = 0? When there's only 1 item?**

---

[← Previous: Define Recurrence](03-define-recurrence-relation.md) | [Next: Order of Computation →](05-order-of-computation.md)

[← Back to README](../README.md)
