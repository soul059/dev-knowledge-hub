# Chapter 1: Climbing Stairs

## 📋 Overview
**Problem:** You are climbing a staircase with `n` steps. Each time you can climb 1 or 2 steps. In how many distinct ways can you reach the top?

This is the quintessential introductory DP problem — simple enough to understand the concept, yet powerful enough to illustrate all DP techniques.

---

## 🧠 Problem Visualization

```
                    ┌───┐
                    │ 5 │  ← GOAL
                ┌───┤   │
                │ 4 │   │
            ┌───┤   │   │
            │ 3 │   │   │
        ┌───┤   │   │   │
        │ 2 │   │   │   │
    ┌───┤   │   │   │   │
    │ 1 │   │   │   │   │
┌───┤   │   │   │   │   │
│ 0 │   │   │   │   │   │  ← START
└───┴───┴───┴───┴───┴───┘

At each step, you can go UP 1 or UP 2.
```

---

## 🔍 Step-by-Step DP Development

### Step 1: Recursive Thinking
```
To reach step n:
  Option A: I was at step n-1 and took 1 step
  Option B: I was at step n-2 and took 2 steps

ways(n) = ways(n-1) + ways(n-2)

This is the Fibonacci recurrence!
```

### Step 2: Recursion Tree (showing overlap)
```
                    ways(5)
                   /       \
              ways(4)      ways(3)         ← ways(3) computed twice
             /      \      /      \
         ways(3)  ways(2) ways(2) ways(1)  ← ways(2) three times
         /    \
     ways(2) ways(1)

Total calls: 15 → Only 6 unique subproblems
Time: O(2ⁿ) — exponential!
```

### Step 3: Memoization (Top-Down)
```
function climbStairs(n, memo = {}):
    if n in memo: return memo[n]
    if n <= 1: return 1
    memo[n] = climbStairs(n-1, memo) + climbStairs(n-2, memo)
    return memo[n]
    
Time: O(n) | Space: O(n)
```

### Step 4: Tabulation (Bottom-Up)
```
function climbStairs(n):
    dp = array[n+1]
    dp[0] = 1    // 1 way to stand at ground
    dp[1] = 1    // 1 way to reach step 1
    for i = 2 to n:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

Time: O(n) | Space: O(n)
```

### Step 5: Space Optimized
```
function climbStairs(n):
    if n <= 1: return 1
    prev2 = 1, prev1 = 1
    for i = 2 to n:
        curr = prev1 + prev2
        prev2 = prev1
        prev1 = curr
    return prev1

Time: O(n) | Space: O(1)
```

---

## 🧪 Complete Trace for n = 5

```
Step │ prev2 │ prev1 │ curr │ Meaning
─────┼───────┼───────┼──────┼──────────────────────
init │   1   │   1   │  -   │ ways(0)=1, ways(1)=1
i=2  │   1   │   2   │  2   │ ways(2)=2  {1+1, 2}
i=3  │   2   │   3   │  3   │ ways(3)=3  {1+1+1, 1+2, 2+1}
i=4  │   3   │   5   │  5   │ ways(4)=5
i=5  │   5   │   8   │  8   │ ways(5)=8

Answer: 8 ways

All 8 ways for n=5:
1. 1+1+1+1+1
2. 1+1+1+2
3. 1+1+2+1
4. 1+2+1+1
5. 2+1+1+1
6. 1+2+2
7. 2+1+2
8. 2+2+1
```

---

## 💡 Real-World Applications
- Tiling problems (domino/tromino tiling)
- Counting binary strings without consecutive 1s
- Robot movement on a linear track
- Step-counting in fitness apps

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force Recursion | O(2ⁿ) | O(n) | Too slow |
| Memoization | O(n) | O(n) | Easy to implement |
| Tabulation | O(n) | O(n) | No stack overflow |
| Space Optimized | O(n) | O(1) | Best solution |

---

## ❓ Quick Revision Questions

1. **Why is this problem equivalent to Fibonacci?**
2. **What are the base cases and why is dp[0] = 1?**
3. **If you could take 1, 2, or 3 steps, how would the recurrence change?**
4. **What is the time complexity of the naive recursive approach?**
5. **How many distinct ways are there to climb 6 stairs?**
6. **Can you solve this using matrix exponentiation for O(log n) time?**

---

[← Previous Unit: Optimize Space](../02-DP-Approach/06-optimize-space.md) | [Next: House Robber →](02-house-robber.md)

[← Back to README](../README.md)
