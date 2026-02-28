# Chapter 1: What is Dynamic Programming?

## 📋 Overview
Dynamic Programming (DP) is an algorithmic technique that solves complex problems by breaking them into smaller **overlapping subproblems**, solving each subproblem **only once**, and storing the result to avoid redundant computation.

---

## 🧠 Core Idea

```
┌─────────────────────────────────────────────────────────────┐
│                   DYNAMIC PROGRAMMING                       │
│                                                             │
│   "An optimization over plain recursion"                    │
│                                                             │
│   Recursion (Brute Force)                                   │
│        │                                                    │
│        ▼                                                    │
│   Identify overlapping subproblems                          │
│        │                                                    │
│        ├──► Memoization (Top-Down)                          │
│        │    Store results as you recurse                     │
│        │                                                    │
│        └──► Tabulation (Bottom-Up)                          │
│             Build solution from smallest subproblem          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Why Do We Need DP?

### The Problem with Plain Recursion

Consider computing Fibonacci(5):

```
                        fib(5)
                       /      \
                   fib(4)      fib(3)
                  /     \      /    \
              fib(3)  fib(2) fib(2) fib(1)
              /   \    / \    / \
          fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
          /   \
      fib(1) fib(0)
```

**Problem:** `fib(3)` is computed **2 times**, `fib(2)` is computed **3 times**!

For `fib(n)`, the recursion tree has **O(2ⁿ)** nodes — exponential!

### The DP Solution

```
┌─────────────────────────────────────────────┐
│  Store computed results in a table/cache     │
│                                              │
│  fib(0) = 0  ──► stored                     │
│  fib(1) = 1  ──► stored                     │
│  fib(2) = 1  ──► stored (= fib(1) + fib(0)) │
│  fib(3) = 2  ──► stored (= fib(2) + fib(1)) │
│  fib(4) = 3  ──► stored (= fib(3) + fib(2)) │
│  fib(5) = 5  ──► stored (= fib(4) + fib(3)) │
│                                              │
│  Time: O(n)  instead of O(2ⁿ)               │
└─────────────────────────────────────────────┘
```

---

## 🏗️ Two Key Properties for DP

A problem can be solved using DP if it has **both** of these properties:

```
┌────────────────────────┐    ┌────────────────────────┐
│  OVERLAPPING           │    │  OPTIMAL               │
│  SUBPROBLEMS           │    │  SUBSTRUCTURE          │
│                        │    │                        │
│  Same subproblems are  │    │  Optimal solution to   │
│  solved multiple times │    │  the problem contains  │
│  in the recursion tree │    │  optimal solutions to  │
│                        │    │  its subproblems       │
│  Example:              │    │  Example:              │
│  fib(3) computed       │    │  shortest path A→C     │
│  multiple times        │    │  via B uses shortest   │
│                        │    │  path A→B + B→C        │
└────────────────────────┘    └────────────────────────┘
         │                              │
         └──────────┬───────────────────┘
                    │
                    ▼
           DP is applicable!
```

---

## 📐 DP vs Other Paradigms

| Paradigm | Key Idea | Subproblems | Example |
|----------|----------|-------------|---------|
| **Brute Force** | Try everything | Independent | Permutations |
| **Greedy** | Local optimal choice | No backtracking | Activity selection |
| **Divide & Conquer** | Split, solve, merge | **Non-overlapping** | Merge sort |
| **Dynamic Programming** | Store & reuse | **Overlapping** | Fibonacci, Knapsack |

### Key Difference: D&C vs DP

```
DIVIDE & CONQUER                    DYNAMIC PROGRAMMING
(Non-overlapping)                   (Overlapping)

    merge_sort(arr)                     fib(5)
       /        \                      /     \
  sort(left)  sort(right)          fib(4)    fib(3)  ◄── shared
     /    \     /    \             /    \      /   \
  sort  sort sort  sort        fib(3) fib(2) fib(2) fib(1)
                                  ▲              ▲
                                  └── same ───────┘
```

---

## 🔄 The DP Development Process

```
Step 1: Write recursive brute-force solution
         │
         ▼
Step 2: Identify overlapping subproblems
         │
         ├──► Step 3a: Add memoization (top-down)
         │              Cache results in hash map/array
         │
         └──► Step 3b: Convert to tabulation (bottom-up)
                        Fill table iteratively
                        │
                        ▼
         Step 4: Optimize space if possible
                 Often O(n) → O(1) using rolling variables
```

---

## 💡 Simple Example: Climbing Stairs

**Problem:** You can climb 1 or 2 stairs at a time. How many distinct ways to reach step `n`?

### Step 1: Recursive Solution
```
function climbStairs(n):
    if n <= 1:
        return 1
    return climbStairs(n-1) + climbStairs(n-2)
```

### Step 2: Identify Overlap
```
                climbStairs(5)
               /              \
        climb(4)              climb(3)      ◄── computed twice
        /      \              /      \
    climb(3)  climb(2)   climb(2)  climb(1)
    /    \     /    \     /    \
climb(2) climb(1) climb(1) climb(0) climb(1) climb(0)
```

### Step 3: Add Memoization
```
function climbStairs(n, memo={}):
    if n <= 1: return 1
    if n in memo: return memo[n]
    memo[n] = climbStairs(n-1, memo) + climbStairs(n-2, memo)
    return memo[n]
```

### Step 4: Tabulation
```
function climbStairs(n):
    dp[0] = 1, dp[1] = 1
    for i = 2 to n:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

### Step 5: Space Optimization
```
function climbStairs(n):
    prev2 = 1, prev1 = 1
    for i = 2 to n:
        curr = prev1 + prev2
        prev2 = prev1
        prev1 = curr
    return prev1
```

### Trace for n = 5:
```
Step  |  prev2  |  prev1  |  curr  |  Result
──────┼─────────┼─────────┼────────┼─────────
init  |    1    |    1    |   -    |
i=2   |    1    |    2    |   2    |
i=3   |    2    |    3    |   3    |
i=4   |    3    |    5    |   5    |
i=5   |    5    |    8    |   8    |    8
```

---

## 🌍 Real-World Applications

| Domain | Application |
|--------|-------------|
| **Bioinformatics** | DNA sequence alignment (Edit Distance) |
| **Finance** | Portfolio optimization, Stock trading |
| **NLP** | Spell checkers, Text similarity |
| **Networking** | Shortest path routing |
| **Operations Research** | Resource allocation, Scheduling |
| **Computer Graphics** | Seam carving for image resizing |
| **Games** | AI decision making, Optimal play |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Dynamic Programming** | Optimization technique that stores solutions to overlapping subproblems |
| **Key Properties** | Overlapping subproblems + Optimal substructure |
| **Memoization** | Top-down approach using recursion + cache |
| **Tabulation** | Bottom-up approach using iterative table filling |
| **Time Improvement** | Exponential → Polynomial (typically) |
| **Space Trade-off** | Uses extra memory to store subproblem solutions |

---

## ❓ Quick Revision Questions

1. **What are the two essential properties a problem must have for DP to be applicable?**
2. **How does DP differ from Divide and Conquer?**
3. **What is the time complexity of naive recursive Fibonacci vs DP Fibonacci?**
4. **Explain the difference between memoization and tabulation in one sentence each.**
5. **Why does DP trade space for time? Is it always worth it?**
6. **Given a recursive solution with overlapping calls, what is the first optimization step?**

---

[Next: Overlapping Subproblems →](02-overlapping-subproblems.md)

[← Back to README](../README.md)
