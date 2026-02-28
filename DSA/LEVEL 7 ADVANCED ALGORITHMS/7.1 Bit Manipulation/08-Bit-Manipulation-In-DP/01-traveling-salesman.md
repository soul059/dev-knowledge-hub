# Chapter 8.1: Traveling Salesman Problem (Bitmask DP)

> **Unit 8 — Bit Manipulation in DP**
> Solve TSP exactly in O(n² × 2^n) using bitmask DP — the most famous application of bitmask states.

---

## Overview

**TSP:** Given n cities and distances between every pair, find the shortest tour that visits every city exactly once and returns to the start. NP-hard in general, but solvable exactly for small n using bitmask DP.

---

## State Definition

```
  dp[mask][i] = minimum cost to visit exactly the cities in 'mask',
                ending at city i.

  mask: bitmask of visited cities
  i:    current (last) city
  
  Constraint: bit i must be set in mask (we're at city i)
```

---

## Recurrence

```
  dp[mask][i] = MIN over all j where:
      - bit j is set in mask
      - j ≠ i
      dp[mask ^ (1<<i)][j] + dist[j][i]

  "The best way to be at city i having visited 'mask' is:
   come from some city j, having visited mask without i,
   plus the distance j → i."
```

---

## Base Case

```
  dp[1 << start][start] = 0
  
  (Starting at city 'start' with only that city visited, cost = 0)
  Typically start = 0.
```

---

## Final Answer

```
  answer = MIN over all i:
      dp[(1<<n) - 1][i] + dist[i][start]

  "Visit all cities (full mask), end at i, then return to start."
```

---

## Pseudocode

```
FUNCTION tsp(dist[][], n):
    FULL = (1 << n) - 1
    dp = 2D array [FULL+1][n], initialized to INFINITY
    dp[1][0] = 0    // start at city 0
    
    FOR mask = 1 TO FULL:
        FOR i = 0 TO n-1:
            IF NOT (mask & (1<<i)): CONTINUE   // i must be in mask
            IF dp[mask][i] == INFINITY: CONTINUE
            
            // Try extending to city j
            FOR j = 0 TO n-1:
                IF mask & (1<<j): CONTINUE     // j must NOT be in mask
                newMask = mask | (1<<j)
                dp[newMask][j] = MIN(dp[newMask][j],
                                     dp[mask][i] + dist[i][j])
    
    // Find best complete tour
    answer = INFINITY
    FOR i = 0 TO n-1:
        answer = MIN(answer, dp[FULL][i] + dist[i][0])
    RETURN answer
```

---

## Step-by-Step Trace: n = 4

```
  Cities: 0, 1, 2, 3
  Distance matrix:
       0   1   2   3
  0  [ 0, 10, 15, 20]
  1  [10,  0, 35, 25]
  2  [15, 35,  0, 30]
  3  [20, 25, 30,  0]

  Base: dp[0001][0] = 0  (at city 0, visited {0})

  mask = 0001 (only city 0):
    From 0 → 1: dp[0011][1] = 0 + 10 = 10
    From 0 → 2: dp[0101][2] = 0 + 15 = 15
    From 0 → 3: dp[1001][3] = 0 + 20 = 20

  mask = 0011 ({0,1}):
    From 1 → 2: dp[0111][2] = 10 + 35 = 45
    From 1 → 3: dp[1011][3] = 10 + 25 = 35

  mask = 0101 ({0,2}):
    From 2 → 1: dp[0111][1] = 15 + 35 = 50
    From 2 → 3: dp[1101][3] = 15 + 30 = 45

  mask = 1001 ({0,3}):
    From 3 → 1: dp[1011][1] = 20 + 25 = 45
    From 3 → 2: dp[1101][2] = 20 + 30 = 50

  mask = 0111 ({0,1,2}):
    dp[0111][1] = 50, dp[0111][2] = 45
    From 1 → 3: dp[1111][3] = 50 + 25 = 75
    From 2 → 3: dp[1111][3] = min(75, 45+30) = 75

  mask = 1011 ({0,1,3}):
    dp[1011][1] = 45, dp[1011][3] = 35
    From 1 → 2: dp[1111][2] = 45 + 35 = 80
    From 3 → 2: dp[1111][2] = min(80, 35+30) = 65

  mask = 1101 ({0,2,3}):
    dp[1101][2] = 50, dp[1101][3] = 45
    From 2 → 1: dp[1111][1] = 50 + 35 = 85
    From 3 → 1: dp[1111][1] = min(85, 45+25) = 70

  Final (mask = 1111):
    dp[1111][1] = 70, return to 0: 70 + 10 = 80
    dp[1111][2] = 65, return to 0: 65 + 15 = 80
    dp[1111][3] = 75, return to 0: 75 + 20 = 95

  Optimal tour cost = 80
  Tour: 0 → 3 → 1 → 2 → 0 (cost 20+25+35+15=... hmm)
  Actually: 0 → 2 → 3 → 1 → 0 = 15+30+25+10 = 80  ✓
```

---

## Complexity

| Aspect | Value |
|--------|-------|
| States | O(n × 2^n) |
| Transitions per state | O(n) |
| Total time | **O(n² × 2^n)** |
| Space | O(n × 2^n) |
| Practical limit | n ≤ 20 |

```
  n    │ States     │ Approx operations
  ─────┼────────────┼───────────────────
  10   │ 10K        │ 100K
  15   │ 500K       │ 7.5M
  20   │ 20M        │ 400M (borderline)
  25   │ 800M       │ too much
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| State | dp[mask][i] = min cost visiting mask, ending at i |
| Transition | Try adding unvisited city j from current city i |
| Base | dp[1<<start][start] = 0 |
| Answer | min(dp[FULL][i] + dist[i][start]) for all i |
| Complexity | O(n² × 2^n) time, O(n × 2^n) space |

---

## Quick Revision Questions

1. **Why is mask needed instead of just tracking the last city?**
   <details><summary>Answer</summary>Knowing only the last city doesn't tell you which cities remain. Two different sets of remaining cities with the same current city need different continuations.</details>

2. **Can TSP bitmask DP solve n = 25?**
   <details><summary>Answer</summary>Barely. 25 × 2^25 ≈ 800M states, each doing O(25) work = 20B operations. Needs heavy optimization or isn't feasible in contest time limits.</details>

3. **What if the graph is not complete (some edges missing)?**
   <details><summary>Answer</summary>Set dist[i][j] = INFINITY for missing edges. The DP handles it naturally — those transitions produce INFINITY cost and are never chosen.</details>

4. **How do you reconstruct the actual tour path?**
   <details><summary>Answer</summary>Maintain a parent array: parent[mask][i] = j meaning "we came to city i in state mask from city j." Backtrack from the optimal final state.</details>

5. **What's the brute-force time for TSP?**
   <details><summary>Answer</summary>O(n!) — try all permutations. For n=20: 20! ≈ 2.4×10^18 vs bitmask DP: 20²×2^20 ≈ 400M. Exponential improvement!</details>

---

[← Previous: Maximum XOR Pair](../07-Advanced-Bit-Operations/06-maximum-xor-pair.md) | [Next: Assignment Problem →](02-assignment-problem.md)
