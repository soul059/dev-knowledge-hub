# Chapter 4: Memoization (Top-Down DP)

## 📋 Overview
**Memoization** is a top-down DP technique where we solve the problem recursively and store (cache) the results of subproblems to avoid recomputation. Think of it as "recursion + memory."

---

## 🧠 Core Concept

```
┌─────────────────────────────────────────────────────────┐
│                    MEMOIZATION                          │
│                                                         │
│   "Remember what you've already computed"               │
│                                                         │
│   ┌──────────┐    Already     ┌──────────────┐          │
│   │ Function │───computed?───►│  Cache/Memo   │          │
│   │  Call    │    YES         │  ┌───┬───┐    │          │
│   └──────────┘   ┌──────┐    │  │ 3 │ 2 │    │          │
│        │         │Return│◄───│  │ 4 │ 3 │    │          │
│        │ NO      │cached│    │  │ 5 │ 5 │    │          │
│        ▼         │result│    │  └───┴───┘    │          │
│   ┌──────────┐   └──────┘    └──────────────┘          │
│   │ Compute  │                     ▲                    │
│   │ Result   │─────────────────────┘                    │
│   └──────────┘     Store result                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 How Memoization Works

### Step-by-Step Process

```
1. Start with the ORIGINAL problem (top)
2. Recursively break into subproblems
3. Before computing, check if result is cached
4. If cached → return stored result (O(1) lookup)
5. If not → compute, STORE in cache, then return
6. Natural recursion handles the order automatically

┌─────────────────────────────────────────┐
│  solve(n):                              │
│    if n in memo:          ◄── CHECK     │
│        return memo[n]     ◄── REUSE     │
│    if base_case:                        │
│        return base_value                │
│    result = recurrence(n)               │
│    memo[n] = result       ◄── STORE     │
│    return result                        │
└─────────────────────────────────────────┘
```

---

## 🏗️ Building Memoized Solutions

### Example: Fibonacci

#### Step 1 — Naive Recursion
```
function fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)

Time: O(2ⁿ)  |  Space: O(n) call stack
```

#### Step 2 — Add Memoization
```
function fib(n, memo = {}):
    if n in memo:               // ← Cache check
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)  // ← Store
    return memo[n]

Time: O(n)  |  Space: O(n) cache + O(n) call stack
```

### Execution Trace for fib(5):

```
Call Stack & Memo State:

fib(5) → not cached, compute
  fib(4) → not cached, compute
    fib(3) → not cached, compute
      fib(2) → not cached, compute
        fib(1) → base case, return 1
        fib(0) → base case, return 0
      memo[2] = 1, return 1
      fib(1) → base case, return 1
    memo[3] = 2, return 2
    fib(2) → CACHED! return 1          ◄── Saved work!
  memo[4] = 3, return 3
  fib(3) → CACHED! return 2            ◄── Saved work!
memo[5] = 5, return 5

Memo table after execution:
┌─────┬───────┐
│  n  │ memo  │
├─────┼───────┤
│  0  │  0    │
│  1  │  1    │
│  2  │  1    │
│  3  │  2    │
│  4  │  3    │
│  5  │  5    │
└─────┴───────┘
```

### Call Tree Comparison:

```
WITHOUT MEMO (15 calls):            WITH MEMO (9 calls, 2 cached):

        fib(5)                              fib(5)
       /      \                            /      \
    fib(4)    fib(3)                    fib(4)    fib(3)✓
    /    \    /    \                    /    \
 fib(3) fib(2) fib(2) fib(1)       fib(3) fib(2)✓
 /   \   /  \   /  \               /    \
f(2) f(1) f(1) f(0) f(1) f(0)   fib(2)  fib(1)
/  \                              /   \
f(1) f(0)                     fib(1) fib(0)

✓ = returned from cache (no further recursion)
```

---

## 📐 Memoization Patterns

### Pattern 1: Using HashMap/Dictionary
```
memo = {}

function solve(key):
    if key in memo:
        return memo[key]
    // ... compute result ...
    memo[key] = result
    return result

Best for: Sparse state spaces, non-integer keys
```

### Pattern 2: Using Array
```
memo = array of size [n+1], initialized to -1

function solve(i):
    if memo[i] != -1:
        return memo[i]
    // ... compute result ...
    memo[i] = result
    return result

Best for: Dense state spaces, integer indices
```

### Pattern 3: Multi-dimensional
```
memo = 2D array of size [m+1][n+1], initialized to -1

function solve(i, j):
    if memo[i][j] != -1:
        return memo[i][j]
    // ... compute result ...
    memo[i][j] = result
    return result

Best for: Grid problems, string comparisons
```

---

## 🧪 Complete Example: Minimum Cost Climbing Stairs

```
Problem: Each step has a cost. You can climb 1 or 2 steps.
         Find minimum cost to reach the top.
         
cost = [10, 15, 20]

Step 1: Define state
  dp(i) = minimum cost to reach step i

Step 2: Recurrence
  dp(i) = cost[i] + min(dp(i-1), dp(i-2))

Step 3: Base cases
  dp(0) = cost[0] = 10
  dp(1) = cost[1] = 15

Step 4: Memoized solution
  memo = {}
  function minCost(i):
      if i in memo: return memo[i]
      if i <= 1: return cost[i]
      memo[i] = cost[i] + min(minCost(i-1), minCost(i-2))
      return memo[i]
  
  Answer = min(minCost(n-1), minCost(n-2))  // can start from last or second-last

Step 5: Trace
  minCost(2) = 20 + min(minCost(1), minCost(0))
             = 20 + min(15, 10)
             = 20 + 10
             = 30
  Answer = min(minCost(2), minCost(1)) = min(30, 15) = 15
```

---

## ⚡ Advantages & Disadvantages

```
┌───────────────────────────┐  ┌───────────────────────────┐
│      ADVANTAGES           │  │     DISADVANTAGES         │
│                           │  │                           │
│ ✓ Easy to implement       │  │ ✗ Stack overflow risk     │
│   (just add cache to      │  │   for deep recursion      │
│    recursive solution)    │  │   (n > ~10,000)           │
│                           │  │                           │
│ ✓ Only computes needed    │  │ ✗ Function call overhead  │
│   subproblems (lazy)      │  │   (slower than iterative) │
│                           │  │                           │
│ ✓ Natural for problems    │  │ ✗ Extra space for call    │
│   with complex ordering   │  │   stack + memo table      │
│                           │  │                           │
│ ✓ Preserves recursive     │  │ ✗ HashMap has higher      │
│   problem structure       │  │   constant factor         │
└───────────────────────────┘  └───────────────────────────┘
```

---

## 💡 When to Prefer Memoization

```
Choose MEMOIZATION (top-down) when:

1. Not all subproblems need to be solved
   ┌─────────────────────┐
   │  State Space (10×10) │
   │  ┌───┐              │
   │  │///│ = computed     │
   │  │   │ = never needed │
   │  └───┘              │
   │  ///...              │
   │  ///.....            │
   │  //.......           │
   │  /.........          │
   │  Only ~30% computed! │
   └─────────────────────┘

2. The recursion order is complex or non-obvious

3. You want to convert a recursive solution quickly

4. The state space is sparse (few valid states)
```

---

## 🔄 Template: General Memoization

```
// Template for memoized DP
function solveProblem(input):
    memo = {}  // or array
    
    function dp(state):
        // 1. Check cache
        if state in memo:
            return memo[state]
        
        // 2. Base case
        if is_base_case(state):
            return base_value
        
        // 3. Recursive case with all choices
        best = initial_value  // -∞ for max, +∞ for min
        for each choice in get_choices(state):
            next_state = transition(state, choice)
            candidate = combine(choice, dp(next_state))
            best = optimize(best, candidate)  // max or min
        
        // 4. Store and return
        memo[state] = best
        return best
    
    return dp(initial_state)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Memoization** | Top-down DP using recursion + caching |
| **Approach** | Start from problem, recurse down, cache results |
| **Data Structure** | HashMap (flexible) or Array (faster) |
| **Time Complexity** | O(number of unique states × work per state) |
| **Space Complexity** | O(states) for cache + O(depth) for call stack |
| **Best For** | Sparse state spaces, complex ordering, quick implementation |
| **Limitation** | Stack overflow for deep recursion |

---

## ❓ Quick Revision Questions

1. **What is the key difference between plain recursion and memoization?**
2. **When would you use a HashMap vs an Array for memoization?**
3. **What is the maximum recursion depth issue, and how can you handle it?**
4. **For fib(n) with memoization, how many function calls are made in total?**
5. **Why is memoization called a "top-down" approach?**
6. **Write memoized pseudocode for computing nCr (combinations).**

---

[← Previous: Optimal Substructure](03-optimal-substructure.md) | [Next: Tabulation →](05-tabulation.md)

[← Back to README](../README.md)
