# Chapter 8.3: Counting Paths with States (Hamiltonian Path)

> **Unit 8 — Bit Manipulation in DP**
> Count (or find) paths that visit every node exactly once using bitmask DP.

---

## Overview

**Hamiltonian Path:** A path in a graph that visits every vertex exactly once. Counting or finding such paths is NP-hard, but bitmask DP solves it in O(n² × 2^n) for small n.

---

## State Definition

```
  dp[mask][i] = number of Hamiltonian paths that visit exactly
                the vertices in 'mask' and end at vertex i.

  mask: set of visited vertices
  i:    current (last) vertex in the path
```

---

## Recurrence

```
  dp[mask][i] = Σ dp[mask ^ (1<<i)][j]
                for all j where:
                  - bit j is set in mask
                  - j ≠ i
                  - edge (j, i) exists

  "The number of paths ending at i with visited set mask =
   sum of paths ending at each valid predecessor j
   with visited set mask minus i."
```

---

## Base Case

```
  dp[1 << i][i] = 1   for each vertex i
  
  (A single vertex is a valid path of length 0)
```

---

## Final Answer

```
  Hamiltonian path exists (or count) =
    Σ dp[(1<<n) - 1][i]  for all i
  
  (Sum over all possible ending vertices with all vertices visited)
```

---

## Pseudocode

```
FUNCTION countHamiltonianPaths(adj[][], n):
    FULL = (1 << n) - 1
    dp = 2D array [FULL+1][n], initialized to 0
    
    // Base: single vertices
    FOR i = 0 TO n-1:
        dp[1 << i][i] = 1
    
    // Fill DP
    FOR mask = 1 TO FULL:
        FOR i = 0 TO n-1:
            IF NOT (mask & (1<<i)): CONTINUE
            IF dp[mask][i] == 0: CONTINUE
            
            FOR j = 0 TO n-1:
                IF mask & (1<<j): CONTINUE      // j already visited
                IF NOT adj[i][j]: CONTINUE       // no edge i→j
                dp[mask | (1<<j)][j] += dp[mask][i]
    
    // Count all Hamiltonian paths
    total = 0
    FOR i = 0 TO n-1:
        total += dp[FULL][i]
    RETURN total
```

---

## Step-by-Step Trace: n = 4

```
  Graph (adjacency):
    0 — 1
    0 — 2
    1 — 2
    1 — 3
    2 — 3

  Base cases:
    dp[0001][0] = 1
    dp[0010][1] = 1
    dp[0100][2] = 1
    dp[1000][3] = 1

  mask=0001 (vertex 0):
    i=0: extend to 1 (edge exists): dp[0011][1] += 1
         extend to 2 (edge exists): dp[0101][2] += 1

  mask=0010 (vertex 1):
    i=1: extend to 0: dp[0011][0] += 1
         extend to 2: dp[0110][2] += 1
         extend to 3: dp[1010][3] += 1

  mask=0100 (vertex 2):
    i=2: extend to 0: dp[0101][0] += 1
         extend to 1: dp[0110][1] += 1
         extend to 3: dp[1100][3] += 1

  mask=1000 (vertex 3):
    i=3: extend to 1: dp[1010][1] += 1
         extend to 2: dp[1100][2] += 1

  mask=0011 ({0,1}):
    dp[0011][0]=1: ext→2: dp[0111][2]+=1
    dp[0011][1]=1: ext→2: dp[0111][2]+=1, ext→3: dp[1011][3]+=1

  mask=0101 ({0,2}):
    dp[0101][0]=1: ext→1: dp[0111][1]+=1
    dp[0101][2]=1: ext→1: dp[0111][1]+=1, ext→3: dp[1101][3]+=1

  ... (continuing through all masks)

  Final: dp[1111][i] for all i → sum = total Hamiltonian paths
```

---

## Applications

| Problem | Bitmask DP Role |
|---------|----------------|
| Hamiltonian path | Count/find paths visiting all nodes |
| Hamiltonian cycle | Paths that return to start |
| Shortest Hamiltonian path | Minimize path weight |
| Chromatic number | Color graph with minimum colors |
| Steiner tree | Connect subset of terminals |

---

## Variations

### Fixed Start Vertex

```
  Only set dp[1<<start][start] = 1 as base case.
```

### Fixed Start and End

```
  Base: dp[1<<start][start] = 1
  Answer: dp[FULL][end]
```

### Hamiltonian Cycle

```
  Answer = Σ dp[FULL][i]  for all i where edge(i, start) exists
```

---

## Complexity

| Aspect | Value |
|--------|-------|
| States | O(n × 2^n) |
| Transitions | O(n) per state |
| Total time | **O(n² × 2^n)** |
| Space | O(n × 2^n) |
| Practical limit | n ≤ 20 |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| State | dp[mask][i] = paths visiting mask, ending at i |
| Transition | Sum over predecessors j with edge j→i |
| Base | dp[1<<i][i] = 1 for each vertex |
| Answer | Sum dp[FULL][i] for all i |
| Same framework as TSP | TSP minimizes cost; this counts paths |

---

## Quick Revision Questions

1. **How does this differ from TSP bitmask DP?**
   <details><summary>Answer</summary>Same state definition. TSP minimizes cost (uses MIN); counting paths sums counts (uses +). TSP requires returning to start; Hamiltonian path doesn't.</details>

2. **Can this work on directed graphs?**
   <details><summary>Answer</summary>Yes, just check directed edges: only extend from i to j if there's a directed edge i→j.</details>

3. **What if n = 20, is this feasible?**
   <details><summary>Answer</summary>20² × 2^20 ≈ 400M operations. Tight but feasible with efficient implementation and ~2 second time limit.</details>

4. **How to find the actual path, not just count?**
   <details><summary>Answer</summary>Store parent information: parent[mask][i] = j meaning we came from vertex j. Backtrack from any dp[FULL][i] > 0.</details>

5. **Does this handle disconnected graphs?**
   <details><summary>Answer</summary>Yes. If the graph is disconnected, no Hamiltonian path exists and dp[FULL][i] = 0 for all i. The algorithm correctly returns 0.</details>

---

[← Previous: Assignment Problem](02-assignment-problem.md) | [Next: Bitmask Optimization Techniques →](04-bitmask-optimization.md)
