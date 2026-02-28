# Chapter 6: Leading to Dynamic Programming

## Overview
This chapter is the **bridge between recursion and DP**. By understanding how to recognize DP-eligible problems and mechanically transform recursive solutions, you unlock one of the most powerful optimization techniques in computer science.

---

## The Journey: Recursion → Memoization → DP

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   STEP 1          STEP 2             STEP 3             │
│  ┌──────┐      ┌───────────┐     ┌──────────────┐      │
│  │Naive │ ──→  │ Top-Down  │ ──→ │ Bottom-Up    │      │
│  │Recurs│      │ Memoize   │     │ Tabulation   │      │
│  └──────┘      └───────────┘     └──────────────┘      │
│                                                         │
│  Time: O(2^n)   Time: O(n)       Time: O(n)            │
│  Space: O(n)    Space: O(n)      Space: O(n) or O(1)   │
│  stack           +memo table      no recursion!         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## The Two Required Properties

```
For DP to work, the problem MUST have BOTH:

1. OPTIMAL SUBSTRUCTURE
   ────────────────────
   The optimal solution to the problem contains
   optimal solutions to its subproblems.
   
   Example: Shortest path A→C through B
   = Shortest(A→B) + Shortest(B→C)
   
   Test: Can you express the solution recursively?
         If YES → has optimal substructure ✓

2. OVERLAPPING SUBPROBLEMS
   ──────────────────────
   The same subproblems are solved repeatedly
   in the naive recursive approach.
   
   Example: fib(5) and fib(4) both need fib(3)
   
   Test: Draw the recursion tree.
         Are there repeated nodes?
         If YES → has overlapping subproblems ✓
```

---

## Step-by-Step Transformation

### Example: Fibonacci

#### Step 1: Naive Recursion
```
FUNCTION fib(n):
    IF n <= 1: RETURN n
    RETURN fib(n - 1) + fib(n - 2)

// Time: O(2^n)  Space: O(n) call stack
```

#### Step 2: Add Memoization (Top-Down DP)
```
memo = {}

FUNCTION fib(n):
    IF n in memo: RETURN memo[n]     ← CHECK cache
    IF n <= 1: RETURN n
    
    result = fib(n - 1) + fib(n - 2)
    memo[n] = result                  ← STORE in cache
    RETURN result

// Time: O(n)  Space: O(n) memo + O(n) stack
```

#### Step 3: Tabulation (Bottom-Up DP)
```
FUNCTION fib(n):
    IF n <= 1: RETURN n
    dp[0] = 0
    dp[1] = 1
    FOR i = 2 TO n:
        dp[i] = dp[i-1] + dp[i-2]   ← Fill table bottom-up
    RETURN dp[n]

// Time: O(n)  Space: O(n) table only (no recursion!)
```

#### Step 4: Space Optimization
```
FUNCTION fib(n):
    IF n <= 1: RETURN n
    prev2 = 0     ← only need last 2 values
    prev1 = 1
    FOR i = 2 TO n:
        curr = prev1 + prev2
        prev2 = prev1
        prev1 = curr
    RETURN prev1

// Time: O(n)  Space: O(1)!
```

---

## The Transformation Pattern

```
STEP 1: Write correct recursive solution
        (don't worry about efficiency)
        
STEP 2: Identify the changing parameters
        (these become your DP state)
        
STEP 3: Add memoization
        □ Create memo table (HashMap or array)
        □ Before computing, check if answer exists
        □ After computing, store the answer
        □ Return cached answer if found
        
STEP 4: Convert to bottom-up (optional)
        □ Determine the order of evaluation
        □ Replace recursion with iteration
        □ Fill the table from smallest subproblem up
        
STEP 5: Optimize space (optional)
        □ Analyze which previous states are needed
        □ Keep only those states (sliding window)
```

---

## More Examples of the Transformation

### Example: Climbing Stairs
```
RECURSIVE:                     BOTTOM-UP DP:
ways(n) = ways(n-1)+ways(n-2)  dp[0]=1, dp[1]=1
                                for i=2..n:
                                  dp[i]=dp[i-1]+dp[i-2]

State: n (1D)
Table: 1D array of size n+1
```

### Example: Grid Paths
```
RECURSIVE:                       BOTTOM-UP DP:
paths(i,j) = paths(i-1,j)       for i=0..m:
             + paths(i,j-1)        for j=0..n:
                                     dp[i][j]=dp[i-1][j]+dp[i][j-1]

State: (i, j) (2D)
Table: 2D array of size m×n
```

### Example: Coin Change
```
RECURSIVE:                       BOTTOM-UP DP:
minCoins(amt) =                  dp[0]=0
  min(minCoins(amt-c)+1)         for a=1..amount:
  for each coin c                  for each coin c:
                                     if a>=c:
                                       dp[a]=min(dp[a],dp[a-c]+1)

State: amount (1D)
Table: 1D array of size amount+1
```

---

## When to Use Each Approach

```
┌──────────────────────────────────────────────────────┐
│  USE MEMOIZATION (TOP-DOWN) WHEN:                     │
│  • Not all subproblems need to be solved              │
│  • Recursion is natural and easy to write             │
│  • You want a quick optimization of existing code     │
│  • The subproblem space is sparse                     │
│                                                      │
│  USE TABULATION (BOTTOM-UP) WHEN:                     │
│  • All subproblems will be needed                     │
│  • You want to avoid stack overflow                   │
│  • Space optimization is important                   │
│  • The evaluation order is clear                     │
│                                                      │
│  USE SPACE-OPTIMIZED DP WHEN:                         │
│  • Current state depends on only a few prior states   │
│  • Memory constraints are tight                      │
│  • You've already verified correctness with full DP   │
└──────────────────────────────────────────────────────┘
```

---

## Decision Flowchart

```
START: You have a recursive solution
  │
  ├── Does it have overlapping subproblems?
  │     │
  │     ├── NO → Divide & Conquer (no DP needed)
  │     │
  │     └── YES → Does it have optimal substructure?
  │           │
  │           ├── NO → May need different approach
  │           │
  │           └── YES → USE DP!
  │                 │
  │                 ├── Add memo table → Top-Down DP ✓
  │                 │
  │                 ├── Determine fill order → Bottom-Up DP ✓
  │                 │
  │                 └── Reduce state → Space-Optimized DP ✓
```

---

## Summary Table

| Approach | Time | Space | Stack | Easy to Write | Easy to Optimize |
|----------|------|-------|-------|---------------|-----------------|
| Naive Recursion | Exponential | O(depth) | Yes | ✓✓✓ | ✗ |
| Memoization | Polynomial | O(states) + stack | Yes | ✓✓ | ✓ |
| Bottom-Up DP | Polynomial | O(states) | No | ✓ | ✓✓ |
| Space-Optimized | Polynomial | O(1) or O(row) | No | ✓ | ✓✓✓ |

---

## What's Next?

```
This chapter concludes our study of recursion trees.
You now understand:
  ✓ How to visualize recursion as trees
  ✓ How to identify overlapping subproblems
  ✓ How to derive complexity from tree structure
  ✓ How recursion leads naturally to DP

Coming up: BACKTRACKING — where we use recursion to
explore ALL possibilities and prune invalid ones!
```

---

## Quick Revision Questions

1. **What are the two properties** required for DP?
2. **What is the difference** between memoization and tabulation?
3. **Transform** the naive recursive coin change to bottom-up DP.
4. **How do you determine** the DP state from a recursive solution?
5. **When would you prefer** top-down over bottom-up?
6. **What does space optimization** require you to analyze?

---

[← Previous: Complexity From Tree](05-complexity-from-tree.md) | [Next Unit: Backtracking Fundamentals →](../06-Backtracking-Fundamentals/01-what-is-backtracking.md)

[← Back to README](../README.md)
