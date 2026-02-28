# Chapter 1: Travelling Salesman Problem — Bitmask DP

## 📋 Overview
The **Travelling Salesman Problem (TSP)** asks for the shortest route visiting every city exactly once and returning to the origin. It is NP-hard, but **bitmask DP** solves it in $O(n^2 \cdot 2^n)$ — feasible for $n \leq 20$.

---

## 🧠 Core Idea

### Bitmask Representation
```
n cities → n bits → 2ⁿ subsets

City:    0   1   2   3
Bit:     1   2   4   8

Bitmask 1011 (binary) = 11 (decimal)
  → cities {0, 1, 3} visited

Operations:
  Check city i visited:   mask & (1 << i) != 0
  Mark city i visited:    mask | (1 << i)
  Unmark city i:          mask & ~(1 << i)
  All cities visited:     mask == (1 << n) - 1
  Count visited:          popcount(mask)
```

### State Definition
```
dp[mask][i] = minimum cost to visit exactly the cities
              in 'mask', ending at city i

mask includes city i (bit i is set)
```

### Transition
```
For each subset 'mask' and each city j in mask:
  for each city i NOT in mask:
    new_mask = mask | (1 << i)
    dp[new_mask][i] = min(dp[new_mask][i], dp[mask][j] + dist[j][i])

"Extend the tour from j to a new city i"
```

### Base Case
```
dp[1 << start][start] = 0
(only start city visited, currently at start, cost = 0)

Typically start = 0
```

### Answer
```
full = (1 << n) - 1
answer = min over all i: dp[full][i] + dist[i][start]
(return to start city)
```

---

## 🔍 Step-by-Step Trace

```
4 cities, distance matrix:
      0   1   2   3
  0 [ 0  10  15  20 ]
  1 [ 10  0  35  25 ]
  2 [ 15  35  0  30 ]
  3 [ 20  25  30  0 ]

Start at city 0.

Base: dp[0001][0] = 0

Extend from city 0:
  dp[0011][1] = dp[0001][0] + dist[0][1] = 0 + 10 = 10
  dp[0101][2] = dp[0001][0] + dist[0][2] = 0 + 15 = 15
  dp[1001][3] = dp[0001][0] + dist[0][3] = 0 + 20 = 20

Extend from city 1 (mask=0011):
  dp[0111][2] = dp[0011][1] + dist[1][2] = 10 + 35 = 45
  dp[1011][3] = dp[0011][1] + dist[1][3] = 10 + 25 = 35

Extend from city 2 (mask=0101):
  dp[0111][1] = dp[0101][2] + dist[2][1] = 15 + 35 = 50
  dp[1101][3] = dp[0101][2] + dist[2][3] = 15 + 30 = 45

Extend from city 3 (mask=1001):
  dp[1011][1] = dp[1001][3] + dist[3][1] = 20 + 25 = 45
  dp[1101][2] = dp[1001][3] + dist[3][2] = 20 + 30 = 50

Continue expanding...

dp[0111][2] = min(45, ...) = 45
dp[0111][1] = min(50, ...) = 50

dp[1111][3] from dp[0111][2]: 45 + 30 = 75
dp[1111][1] from dp[1101][2]: 50 + 35 = 85
dp[1111][3] from dp[1011][1]: 45 + 25 = 70  ← 
dp[1111][2] from dp[1011][3]: 35 + 30 = 65

... (evaluating all paths)

Full mask = 1111:
  dp[1111][1] + dist[1][0] = 65 + 10 = 75
  dp[1111][2] + dist[2][0] = 65 + 15 = 80
  dp[1111][3] + dist[3][0] = 70 + 20 = 90

Optimal tour cost = 75  (e.g., 0→1→3→2→0 or similar)
```

---

## 💻 Pseudocode

```
function tsp(dist, n):
    FULL = (1 << n) - 1
    dp = 2D array [FULL+1][n] = INF
    
    // Base case: start at city 0
    dp[1][0] = 0
    
    // Fill: iterate over all masks in increasing order
    for mask = 1 to FULL:
        for last = 0 to n-1:
            if dp[mask][last] == INF: continue
            if mask & (1 << last) == 0: continue
            
            // Try extending to each unvisited city
            for next = 0 to n-1:
                if mask & (1 << next) != 0: continue
                new_mask = mask | (1 << next)
                dp[new_mask][next] = min(
                    dp[new_mask][next],
                    dp[mask][last] + dist[last][next]
                )
    
    // Close the tour back to city 0
    answer = INF
    for last = 0 to n-1:
        answer = min(answer, dp[FULL][last] + dist[last][0])
    
    return answer
```

---

## 🔄 Path Reconstruction

```
function reconstructPath(dp, dist, n):
    FULL = (1 << n) - 1
    path = []
    mask = FULL
    last = argmin(dp[FULL][i] + dist[i][0]) for all i
    
    while mask != (1 << 0):   // until only city 0
        path.prepend(last)
        prev_mask = mask ^ (1 << last)
        
        // Find which city led to 'last'
        for prev = 0 to n-1:
            if prev_mask & (1 << prev) == 0: continue
            if dp[prev_mask][prev] + dist[prev][last] == dp[mask][last]:
                mask = prev_mask
                last = prev
                break
    
    path.prepend(0)   // start city
    path.append(0)     // return to start
    return path
```

---

## ⚡ Complexity Analysis

```
┌────────────────────────────────────────┐
│ Time:  O(n² · 2ⁿ)                     │
│   2ⁿ masks × n choices for last       │
│   × n choices for next                 │
│                                        │
│ Space: O(n · 2ⁿ)                      │
│   dp table                             │
│                                        │
│ Feasible: n ≤ 20 (2²⁰ ≈ 10⁶)         │
│ Tight:    n ≤ 23-25 with optimizations │
│                                        │
│ vs Brute force: O(n!)                  │
│   n=15: 15! ≈ 10¹² vs 2¹⁵·15² ≈ 7·10⁶│
└────────────────────────────────────────┘
```

---

## 🧩 Variants

| Variant | Modification |
|---------|-------------|
| No return to start | Answer = min(dp[FULL][i]) |
| Fixed start + end | Different base and answer extraction |
| Asymmetric distances | dist[i][j] ≠ dist[j][i], same algorithm |
| Minimum Hamiltonian Path | No return needed |
| Maximum weight tour | Use max instead of min |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Shortest tour visiting all cities exactly once |
| **State** | dp[mask][i] = min cost, visited=mask, at city i |
| **Mask** | Bitmask encodes subset of visited cities |
| **Complexity** | O(n² · 2ⁿ) time, O(n · 2ⁿ) space |
| **Limit** | n ≤ 20 typically |
| **Key ops** | Check bit, set bit, full mask = (1<<n)-1 |

---

## ❓ Quick Revision Questions

1. **What does dp[mask][i] represent in TSP bitmask DP?**
2. **How do you check if city i is already visited in a bitmask?**
3. **What is the base case for TSP starting at city 0?**
4. **Why is bitmask DP O(n²·2ⁿ) instead of O(n!)?**
5. **How do you modify the solution if you don't need to return to origin?**
6. **For what range of n is bitmask DP feasible?**

---

[← Previous Unit: State Machine DP](../11-State-Machine-DP/06-pattern-recognition.md) | [Next: Shortest Path All Nodes →](02-shortest-path-all-nodes.md)

[← Back to README](../README.md)
