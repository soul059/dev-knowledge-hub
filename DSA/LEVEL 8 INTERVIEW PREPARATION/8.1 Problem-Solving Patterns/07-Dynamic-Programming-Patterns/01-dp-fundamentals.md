# Chapter 1: DP Fundamentals

## 📋 Chapter Overview
Dynamic Programming = **recursion + memoization** (top-down) or **iterative table-filling** (bottom-up). It applies when a problem has **optimal substructure** and **overlapping subproblems**.

---

## 🧠 The Two Properties

### 1. Optimal Substructure
The optimal solution to the problem contains optimal solutions to subproblems.

```
  Shortest path A → D = shortest path A → C + edge C → D
  (if C is on the optimal path)
```

### 2. Overlapping Subproblems
The same subproblems are solved multiple times.

```
  Fibonacci: fib(5) calls fib(4) and fib(3)
             fib(4) calls fib(3) and fib(2)
             fib(3) is computed TWICE!

  fib(5)
  ├── fib(4)
  │   ├── fib(3)        ← computed here
  │   │   ├── fib(2)
  │   │   └── fib(1)
  │   └── fib(2)
  └── fib(3)             ← and AGAIN here
      ├── fib(2)
      └── fib(1)
```

---

## 📝 Three Approaches

### 1. Plain Recursion (Exponential)

```
function fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)

// Time: O(2^n) — huge redundancy
```

### 2. Top-Down (Memoization)

```
function fib(n, memo):
    if n <= 1: return n
    if memo[n] exists: return memo[n]
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]

// Time: O(n)  Space: O(n)
```

### 3. Bottom-Up (Tabulation)

```
function fib(n):
    if n <= 1: return n
    dp = array of size n+1
    dp[0] = 0, dp[1] = 1
    for i = 2 to n:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

// Time: O(n)  Space: O(n), can optimize to O(1)
```

### Space-Optimized Bottom-Up

```
function fib(n):
    if n <= 1: return n
    prev2 = 0, prev1 = 1
    for i = 2 to n:
        curr = prev1 + prev2
        prev2 = prev1
        prev1 = curr
    return prev1

// Time: O(n)  Space: O(1)
```

---

## 🗺️ DP Problem-Solving Framework

```
  Step 1: Define STATE
    "What information do I need to describe a subproblem?"
    → dp[i], dp[i][j], dp[i][w], etc.
  
  Step 2: Define TRANSITION (recurrence)
    "How do I build dp[i] from smaller subproblems?"
    → dp[i] = dp[i-1] + dp[i-2]
  
  Step 3: Define BASE CASE
    "What are the smallest subproblems I can solve directly?"
    → dp[0] = 0, dp[1] = 1
  
  Step 4: Define ANSWER
    "Which dp cell holds the final answer?"
    → dp[n]
  
  Step 5: Determine ORDER
    "What order should I fill the table?"
    → Left to right, or bottom-up
```

---

## 🔍 Example: Climbing Stairs

**You can climb 1 or 2 steps. How many ways to reach step n?**

```
  Step 1: State → dp[i] = number of ways to reach step i
  Step 2: Transition → dp[i] = dp[i-1] + dp[i-2]
             (come from step i-1 OR step i-2)
  Step 3: Base → dp[0] = 1, dp[1] = 1
  Step 4: Answer → dp[n]
```

### Trace: n = 5

| i | dp[i-2] | dp[i-1] | dp[i] | Meaning |
|---|---------|---------|-------|---------|
| 0 | - | - | 1 | 1 way to stay at ground |
| 1 | - | - | 1 | 1 way to reach step 1 |
| 2 | 1 | 1 | 2 | {1+1, 2} |
| 3 | 1 | 2 | 3 | {1+1+1, 1+2, 2+1} |
| 4 | 2 | 3 | 5 | {1111, 112, 121, 211, 22} |
| 5 | 3 | 5 | **8** | Answer |

---

## 🔍 Example: Coin Change (Minimum Coins)

**Given coins and amount, find minimum coins to make amount.**

```
  coins = [1, 3, 4], amount = 6
  
  State: dp[a] = min coins to make amount a
  Transition: dp[a] = min(dp[a - coin] + 1) for each coin
  Base: dp[0] = 0
  Answer: dp[amount]
```

```
function coinChange(coins, amount):
    dp = array of size amount+1, filled with INF
    dp[0] = 0
    
    for a = 1 to amount:
        for coin in coins:
            if coin <= a AND dp[a - coin] + 1 < dp[a]:
                dp[a] = dp[a - coin] + 1
    
    return dp[amount] if dp[amount] != INF else -1
```

### Trace: coins = [1, 3, 4], amount = 6

| a | Using 1 | Using 3 | Using 4 | dp[a] |
|---|---------|---------|---------|-------|
| 0 | - | - | - | 0 |
| 1 | dp[0]+1=1 | - | - | 1 |
| 2 | dp[1]+1=2 | - | - | 2 |
| 3 | dp[2]+1=3 | dp[0]+1=1 | - | 1 |
| 4 | dp[3]+1=2 | dp[1]+1=2 | dp[0]+1=1 | 1 |
| 5 | dp[4]+1=2 | dp[2]+1=3 | dp[1]+1=2 | 2 |
| 6 | dp[5]+1=3 | dp[3]+1=2 | dp[2]+1=3 | **2** |

Answer: 2 (coins 3 + 3) ✓

---

## 📊 Top-Down vs Bottom-Up

| Aspect | Top-Down (Memo) | Bottom-Up (Tab) |
|--------|----------------|-----------------|
| Approach | Recursive + cache | Iterative + table |
| Ease | Often easier to write | Need to determine fill order |
| Computes | Only needed subproblems | All subproblems |
| Stack | O(n) recursion | No recursion |
| Space opt | Harder | Easier (rolling array) |
| Interview | Good starting point | Preferred final answer |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Two properties | Optimal substructure + overlapping subproblems |
| 5-step framework | State → Transition → Base → Answer → Order |
| Top-down | Recursion + memo; natural but uses stack |
| Bottom-up | Iterative table; preferred for space optimization |
| Space optimization | Keep only needed previous states (rolling variables) |
| Time complexity | Usually O(n × factors in transition) |

---

## ❓ Revision Questions

1. **Two properties required for DP?** → Optimal substructure and overlapping subproblems.
2. **Top-down vs bottom-up: when to prefer which?** → Top-down for complex state spaces (compute only what's needed); bottom-up for space optimization.
3. **What are the 5 steps of DP design?** → Define state, transition, base case, answer location, fill order.
4. **Climbing stairs: why is dp[i] = dp[i-1] + dp[i-2]?** → You can arrive at step i from step i-1 (1 step) or step i-2 (2 steps).
5. **How to optimize DP space?** → If dp[i] only depends on dp[i-1] and dp[i-2], keep only two variables instead of an array.

---

[← Back to README](../README.md) | [Next: 1D DP Patterns →](02-1d-dp-patterns.md)
