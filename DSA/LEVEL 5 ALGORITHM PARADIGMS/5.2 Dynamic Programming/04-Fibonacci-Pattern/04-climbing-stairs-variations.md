# Chapter 4: Climbing Stairs Variations

## 📋 Overview
The **Climbing Stairs** problem is the purest expression of the Fibonacci pattern. This chapter explores multiple variations that demonstrate how the pattern generalizes: variable step sizes, costs, minimum steps, and combinatorial counting.

---

## 🧠 The Base Problem Recap

```
You can climb 1 or 2 steps at a time.
How many distinct ways to reach step n?

dp[n] = dp[n-1] + dp[n-2]   ← Pure Fibonacci!

     Step 5  ←─── dp[5] = dp[4] + dp[3] = 5 + 3 = 8
     Step 4  ←─── dp[4] = dp[3] + dp[2] = 3 + 2 = 5
     Step 3  ←─── dp[3] = dp[2] + dp[1] = 2 + 1 = 3
     Step 2  ←─── dp[2] = dp[1] + dp[0] = 1 + 1 = 2
     Step 1  ←─── dp[1] = 1
     Step 0  ←─── dp[0] = 1 (base)
```

---

## 🔨 Variation 1: K-step Climbing (1 to k steps)

```
You can climb 1, 2, 3, ... up to k steps at a time.

Recurrence:
  dp[i] = dp[i-1] + dp[i-2] + ... + dp[i-k]
         = Σ dp[i-j] for j = 1 to min(i, k)

This is the K-bonacci pattern!

function climbStairs(n, k):
    dp = array of size n+1, filled with 0
    dp[0] = 1
    for i = 1 to n:
        for j = 1 to min(i, k):
            dp[i] += dp[i-j]
    return dp[n]

Example: n=4, k=3 (can take 1, 2, or 3 steps)
  dp[0] = 1
  dp[1] = dp[0] = 1
  dp[2] = dp[1] + dp[0] = 2
  dp[3] = dp[2] + dp[1] + dp[0] = 4
  dp[4] = dp[3] + dp[2] + dp[1] = 7

Time: O(n·k), Space: O(n) → optimizable to O(k)
```

---

## 🔨 Variation 2: Min Cost Climbing Stairs (LC #746)

```
Given cost[i] = cost to step on stair i.
Find minimum cost to reach the top (beyond last stair).
You can start from step 0 or step 1.

Recurrence:
  dp[i] = cost[i] + min(dp[i-1], dp[i-2])

  "Pay cost[i] to stand here, came from cheapest neighbor"

Example: cost = [10, 15, 20]

  dp[0] = 10
  dp[1] = 15
  dp[2] = 20 + min(10, 15) = 30
  Answer = min(dp[1], dp[2]) = min(15, 30) = 15

  Optimal: start at step 1 (cost 15), jump to top.
```

### Pseudocode
```
function minCostClimbing(cost):
    n = len(cost)
    prev2 = cost[0]
    prev1 = cost[1]
    for i = 2 to n-1:
        current = cost[i] + min(prev1, prev2)
        prev2 = prev1
        prev1 = current
    return min(prev1, prev2)

Time: O(n), Space: O(1)
```

---

## 🔨 Variation 3: Climbing with Specific Step Sizes

```
Given a set of allowed step sizes, e.g., steps = {1, 3, 5}

Recurrence:
  dp[i] = Σ dp[i - s]  for each s in steps where s ≤ i

Example: n=7, steps={1, 3, 5}
  dp[0] = 1
  dp[1] = dp[0] = 1
  dp[2] = dp[1] = 1
  dp[3] = dp[2] + dp[0] = 2
  dp[4] = dp[3] + dp[1] = 3
  dp[5] = dp[4] + dp[2] + dp[0] = 5
  dp[6] = dp[5] + dp[3] + dp[1] = 8
  dp[7] = dp[6] + dp[4] + dp[2] = 12

function climbStairs(n, steps):
    dp = array of size n+1, filled with 0
    dp[0] = 1
    for i = 1 to n:
        for s in steps:
            if i >= s:
                dp[i] += dp[i-s]
    return dp[n]

Time: O(n · |steps|), Space: O(n)
```

---

## 🔨 Variation 4: Count Paths with Exactly m Steps

```
How many ways to reach step n using exactly m steps?

State: dp[i][j] = ways to reach stair i using exactly j steps

Recurrence:
  dp[i][j] = dp[i-1][j-1] + dp[i-2][j-1]
  (took 1 step from i-1, or 2 steps from i-2, both in j-1 prior steps)

Base: dp[0][0] = 1

Example: n=4, m=3
  Ways using exactly 3 steps to reach stair 4:
  - (1,1,2), (1,2,1), (2,1,1), (2,2,×)... only steps summing to 4
  - Valid: {1,1,2}, {1,2,1}, {2,1,1} = 3 ways

  dp table:
  j\i   0  1  2  3  4
   0  [ 1  0  0  0  0 ]
   1  [ 0  1  1  0  0 ]
   2  [ 0  0  1  2  1 ]
   3  [ 0  0  0  1  3 ]  ← dp[4][3] = 3 ✓
```

---

## 🔨 Variation 5: Climbing Down (Bidirectional)

```
Can go up 1-2 steps OR down 1 step.
Find minimum steps from 0 to n.

This becomes a BFS/shortest path problem rather than pure DP,
but still uses the Fibonacci-like state transitions:

States: current position
Transitions: pos+1, pos+2, pos-1

BFS from 0 → n gives shortest path.
Pure DP doesn't work well here due to cycles (going down).

Lesson: Fibonacci pattern works for DAG-like dependencies.
         Cycles require graph algorithms (BFS/Dijkstra).
```

---

## 🔬 Comprehensive Trace: Min Cost, cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]

```
Index:    0    1    2    3    4    5    6    7    8    9
Cost:     1  100    1    1    1  100    1    1  100    1

dp[0] = 1
dp[1] = 100
dp[2] = 1 + min(1, 100) = 2
dp[3] = 1 + min(100, 2) = 3
dp[4] = 1 + min(2, 3) = 3
dp[5] = 100 + min(3, 3) = 103
dp[6] = 1 + min(3, 103) = 4
dp[7] = 1 + min(103, 4) = 5
dp[8] = 100 + min(4, 5) = 104
dp[9] = 1 + min(5, 104) = 6

Answer = min(dp[8], dp[9]) = min(104, 6) = 6

Path: 0→2→3→4→6→7→9→top (cost: 1+1+1+1+1+1=6) ✓
```

---

## 📊 Summary Table

| Variation | Recurrence | Time | Space |
|-----------|------------|------|-------|
| **Basic (1,2 steps)** | dp[i] = dp[i-1] + dp[i-2] | O(n) | O(1) |
| **K-step** | dp[i] = Σ dp[i-j], j=1..k | O(nk) | O(k) |
| **Min Cost** | dp[i] = cost[i] + min(dp[i-1], dp[i-2]) | O(n) | O(1) |
| **Custom steps** | dp[i] = Σ dp[i-s], s∈steps | O(n·\|S\|) | O(n) |
| **Exact m steps** | dp[i][j] = dp[i-1][j-1]+dp[i-2][j-1] | O(nm) | O(nm) |

---

## ❓ Quick Revision Questions

1. **How does climbing with k possible steps generalize to K-bonacci?**
2. **What is the recurrence for Minimum Cost Climbing Stairs?**
3. **How do you handle arbitrary step sizes like {1, 3, 5}?**
4. **Why can't pure DP handle the "climb up or down" variation?**
5. **Write the space-optimized solution for Min Cost Climbing Stairs.**
6. **How do you count paths using exactly m steps to reach stair n?**

---

[← Previous: House Robber](03-house-robber.md) | [Next: Maximum Sum Non-Adjacent →](05-max-sum-non-adjacent.md)

[← Back to README](../README.md)
