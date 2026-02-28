# Chapter 5: Tabulation (Bottom-Up DP)

## 📋 Overview
**Tabulation** is a bottom-up DP technique where we solve subproblems iteratively, starting from the smallest (base cases) and building up to the final answer. Results are stored in a table (array) that is filled systematically.

---

## 🧠 Core Concept

```
┌─────────────────────────────────────────────────────────┐
│                    TABULATION                           │
│                                                         │
│   "Build the solution from the ground up"               │
│                                                         │
│   Base Cases ──► Small problems ──► Larger ──► Answer   │
│                                                         │
│   ┌───┬───┬───┬───┬───┬───┬───┐                        │
│   │ 0 │ 1 │ 1 │ 2 │ 3 │ 5 │ ? │  ◄── Fill left to     │
│   └───┴───┴───┴───┴───┴───┴───┘      right             │
│   base  base  ───────────────►                          │
│   cases       computed from                             │
│               previous entries                          │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Memoization vs Tabulation

```
┌──────────────────────────┐  ┌──────────────────────────┐
│   MEMOIZATION (Top-Down) │  │   TABULATION (Bottom-Up) │
│                          │  │                          │
│   Start: Big problem     │  │   Start: Base cases      │
│   Direction: Top → Down  │  │   Direction: Bottom → Up │
│   Method: Recursion      │  │   Method: Iteration      │
│   Computes: On demand    │  │   Computes: All states   │
│   Call stack: Yes (deep) │  │   Call stack: No          │
│                          │  │                          │
│       solve(5)           │  │   dp[0] → dp[1] → dp[2] │
│       /      \           │  │     → dp[3] → dp[4]     │
│   solve(4)  solve(3)     │  │       → dp[5] ✓         │
│    /    \    (cached)     │  │                          │
│  solve(3) solve(2)       │  │   Linear, no recursion   │
│  ...                     │  │                          │
└──────────────────────────┘  └──────────────────────────┘
```

---

## 🏗️ Building Tabulated Solutions

### Example: Fibonacci

#### Step 1 — Define DP Table
```
dp[] = array of size n+1
dp[i] = i-th Fibonacci number
```

#### Step 2 — Set Base Cases
```
dp[0] = 0
dp[1] = 1
```

#### Step 3 — Fill Table (Recurrence)
```
for i = 2 to n:
    dp[i] = dp[i-1] + dp[i-2]
```

#### Step 4 — Return Answer
```
return dp[n]
```

### Complete Pseudocode:
```
function fib(n):
    dp = array of size [n+1]
    dp[0] = 0
    dp[1] = 1
    for i = 2 to n:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

Time: O(n)  |  Space: O(n)
```

### Step-by-Step Trace for fib(7):

```
i  │  dp[i-2]  │  dp[i-1]  │  dp[i] = dp[i-1] + dp[i-2]
───┼───────────┼───────────┼──────────────────────────────
0  │     -     │     -     │    0  (base)
1  │     -     │     -     │    1  (base)
2  │     0     │     1     │    1
3  │     1     │     1     │    2
4  │     1     │     2     │    3
5  │     2     │     3     │    5
6  │     3     │     5     │    8
7  │     5     │     8     │   13  ◄── Answer

Table after filling:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  0  │  1  │  1  │  2  │  3  │  5  │  8  │ 13  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
 dp[0] dp[1] dp[2] dp[3] dp[4] dp[5] dp[6] dp[7]
```

---

## 📐 Tabulation Template

### 1D Tabulation
```
function solve_1d(n):
    // Step 1: Create table
    dp = array of size [n+1], initialized to 0
    
    // Step 2: Base cases
    dp[0] = base_value_0
    dp[1] = base_value_1
    
    // Step 3: Fill table
    for i = 2 to n:
        dp[i] = recurrence(dp[i-1], dp[i-2], ...)
    
    // Step 4: Return answer
    return dp[n]
```

### 2D Tabulation
```
function solve_2d(m, n):
    // Step 1: Create table
    dp = 2D array of size [m+1][n+1], initialized to 0
    
    // Step 2: Base cases
    dp[0][0] = base_value
    for i = 0 to m: dp[i][0] = base_row
    for j = 0 to n: dp[0][j] = base_col
    
    // Step 3: Fill table
    for i = 1 to m:
        for j = 1 to n:
            dp[i][j] = recurrence(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    
    // Step 4: Return answer
    return dp[m][n]
```

---

## 🧪 Complete Example: Coin Change

```
Problem: Minimum coins to make amount = 11
Coins: [1, 5, 6]

Step 1: Define table
  dp[i] = minimum coins to make amount i

Step 2: Base case
  dp[0] = 0  (0 coins needed for amount 0)

Step 3: Recurrence
  dp[i] = min(dp[i - coin] + 1) for each coin where i - coin >= 0

Step 4: Fill table

i    │  dp[i-1]+1  │  dp[i-5]+1  │  dp[i-6]+1  │  dp[i]
─────┼─────────────┼─────────────┼─────────────┼────────
0    │     -       │     -       │     -       │   0 (base)
1    │   0+1=1     │     -       │     -       │   1
2    │   1+1=2     │     -       │     -       │   2
3    │   2+1=3     │     -       │     -       │   3
4    │   3+1=4     │     -       │     -       │   4
5    │   4+1=5     │   0+1=1    │     -       │   1
6    │   1+1=2     │   1+1=2    │   0+1=1    │   1
7    │   1+1=2     │   2+1=3    │   1+1=2    │   2
8    │   2+1=3     │   3+1=4    │   2+1=3    │   3
9    │   3+1=4     │   4+1=5    │   3+1=4    │   4
10   │   4+1=5     │   1+1=2    │   4+1=5    │   2
11   │   2+1=3     │   1+1=2    │   1+1=2    │   2 ◄── Answer!

Table:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 1 │ 1 │ 2 │ 3 │ 4 │ 2 │ 2 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
 0   1   2   3   4   5   6   7   8   9  10  11

Answer: dp[11] = 2 (using coins 5 + 6)
```

---

## 🔄 Converting Memoization to Tabulation

```
┌─────────────────────────────────────────────────────────┐
│         CONVERSION STEPS                                │
│                                                         │
│  1. Identify all changing parameters in recursion       │
│     → These become table dimensions                     │
│                                                         │
│  2. Identify base cases in recursion                    │
│     → These become initial table values                 │
│                                                         │
│  3. Identify the recurrence relation                    │
│     → This becomes the table-filling formula            │
│                                                         │
│  4. Determine fill order                                │
│     → Ensure dependencies are computed first            │
│                                                         │
│  5. Identify where the answer is                        │
│     → Usually dp[n] or dp[m][n]                         │
└─────────────────────────────────────────────────────────┘
```

### Example Conversion: Climbing Stairs

```
MEMOIZATION:                        TABULATION:

function climb(n, memo):            function climb(n):
  if n in memo:                       dp = array[n+1]
    return memo[n]                    dp[0] = 1
  if n <= 1:                          dp[1] = 1
    return 1                          for i = 2 to n:
  memo[n] = climb(n-1)                  dp[i] = dp[i-1] + dp[i-2]
          + climb(n-2)                return dp[n]
  return memo[n]

Mapping:
  Recursive parameter n → Table index i
  Base cases n≤1 → dp[0]=1, dp[1]=1
  Recurrence → dp[i] = dp[i-1] + dp[i-2]
  Fill order → left to right (i=2 to n)
  Answer → dp[n]
```

---

## 💡 2D Tabulation Example: Grid Paths

```
Problem: Count unique paths from top-left to bottom-right
         in a 3×3 grid (can move right or down only)

Table dp[i][j] = number of paths to reach (i,j)

Base: First row and first column = 1 (only one way)

     j=0   j=1   j=2
    ┌─────┬─────┬─────┐
i=0 │  1  │  1  │  1  │  ← only right
    ├─────┼─────┼─────┤
i=1 │  1  │  2  │  3  │
    ├─────┼─────┼─────┤
i=2 │  1  │  3  │  6  │  ← Answer: 6
    └─────┴─────┴─────┘
     ↑
  only down

Fill order: row by row, left to right
  dp[i][j] = dp[i-1][j] + dp[i][j-1]
              (from top)   (from left)

Trace:
  dp[1][1] = dp[0][1] + dp[1][0] = 1 + 1 = 2
  dp[1][2] = dp[0][2] + dp[1][1] = 1 + 2 = 3
  dp[2][1] = dp[1][1] + dp[2][0] = 2 + 1 = 3
  dp[2][2] = dp[1][2] + dp[2][1] = 3 + 3 = 6
```

---

## ⚡ Advantages & Disadvantages

```
┌───────────────────────────┐  ┌───────────────────────────┐
│      ADVANTAGES           │  │     DISADVANTAGES         │
│                           │  │                           │
│ ✓ No recursion overhead   │  │ ✗ Must determine fill     │
│   (no stack overflow)     │  │   order explicitly        │
│                           │  │                           │
│ ✓ Generally faster        │  │ ✗ Computes ALL states     │
│   (no function call cost) │  │   (even unneeded ones)    │
│                           │  │                           │
│ ✓ Easy to optimize space  │  │ ✗ Harder to see the       │
│   (rolling arrays)        │  │   recursive structure     │
│                           │  │                           │
│ ✓ Better cache locality   │  │ ✗ Less intuitive for     │
│   (sequential access)     │  │   complex state spaces    │
└───────────────────────────┘  └───────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Tabulation** | Bottom-up DP using iterative table filling |
| **Direction** | Start from base cases, build to answer |
| **Method** | Iterative loops (no recursion) |
| **Time Complexity** | O(total states × work per state) |
| **Space Complexity** | O(table size), often optimizable |
| **Best For** | When all states needed, no stack overflow risk |
| **Fill Order** | Must ensure dependencies computed first |

---

## ❓ Quick Revision Questions

1. **Why is tabulation called "bottom-up"?**
2. **What determines the fill order in a tabulation approach?**
3. **Convert the following memoized recurrence to tabulation: `dp(i,j) = dp(i-1,j) + dp(i,j-1)` with base `dp(0,j) = dp(i,0) = 1`**
4. **When might tabulation compute unnecessary states that memoization wouldn't?**
5. **What is the space complexity advantage of tabulation over memoization?**
6. **Can tabulation always replace memoization? Are there cases where memoization is strictly better?**

---

[← Previous: Memoization](04-memoization.md) | [Next: When to Use DP →](06-when-to-use-dp.md)

[← Back to README](../README.md)
