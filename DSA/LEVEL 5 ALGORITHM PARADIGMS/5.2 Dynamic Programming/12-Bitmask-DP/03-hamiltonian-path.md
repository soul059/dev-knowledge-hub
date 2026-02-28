# Chapter 3: Hamiltonian Path — Bitmask DP

## 📋 Overview
A **Hamiltonian Path** visits every vertex in a graph **exactly once**. Determining if one exists is NP-complete, but bitmask DP solves it in $O(n^2 \cdot 2^n)$. This chapter covers existence, counting, and reconstruction.

---

## 🧠 Core Idea

### Problem Definition
```
Given: Graph G with n vertices
Find:  Path that visits every vertex exactly once

Hamiltonian Path:   v₁ → v₂ → v₃ → ... → vₙ
  Each vertex appears exactly once
  Consecutive vertices are connected by an edge

Hamiltonian Cycle:   Same + edge from vₙ back to v₁
```

### State Definition
```
dp[mask][i] = true if there exists a path visiting
              exactly the vertices in 'mask', ending at vertex i

Transition:
  dp[mask][i] = OR over all j where:
    1. j is in mask
    2. edge(j, i) exists
    3. dp[mask ^ (1 << i)][j] is true

"There's a valid path to i through mask
 if some neighbor j had a valid path through mask\{i}"
```

---

## 🔍 Step-by-Step Trace

```
Graph (n=4):
  0 — 1
  |
  2 — 3

Edges: (0,1), (0,2), (2,3)

Base cases: dp[0001][0]=T, dp[0010][1]=T, dp[0100][2]=T, dp[1000][3]=T

2-vertex paths (2 bits set):
  dp[0011][1] = dp[0001][0] AND edge(0,1) = T ✓
  dp[0011][0] = dp[0010][1] AND edge(1,0) = T ✓
  dp[0101][2] = dp[0001][0] AND edge(0,2) = T ✓
  dp[0101][0] = dp[0100][2] AND edge(2,0) = T ✓
  dp[0110][1] = dp[0100][2] AND edge(2,1) = F ✗
  dp[0110][2] = dp[0010][1] AND edge(1,2) = F ✗
  dp[1100][3] = dp[0100][2] AND edge(2,3) = T ✓
  dp[1100][2] = dp[1000][3] AND edge(3,2) = T ✓
  ...

3-vertex paths:
  dp[0111][2] = dp[0011][0] AND edge(0,2) = T ✓  (1→0→2)
  dp[0111][2] = dp[0011][1] AND edge(1,2) = F
    → dp[0111][2] = T
  
  dp[1101][3] = dp[0101][2] AND edge(2,3) = T ✓  (0→2→3)
  dp[1101][0] = dp[1100][2] AND edge(2,0) = T ✓  (3→2→0)
  ...

4-vertex paths (full mask = 1111):
  dp[1111][3] = dp[0111][2] AND edge(2,3)
              = T AND T = T ✓  (1→0→2→3)

Hamiltonian path exists! Path: 1 → 0 → 2 → 3
```

---

## 💻 Pseudocode: Existence

```
function hamiltonianPathExists(adj, n):
    FULL = (1 << n) - 1
    dp = 2D array [FULL+1][n] = false
    
    // Base: single vertex paths
    for i = 0 to n-1:
        dp[1 << i][i] = true
    
    // Fill: increasing mask size
    for mask = 1 to FULL:
        for i = 0 to n-1:
            if not (mask & (1 << i)): continue  // i must be in mask
            if not dp[mask][i]: continue
            
            // Extend to unvisited neighbor
            for j = 0 to n-1:
                if mask & (1 << j): continue  // j not visited
                if not adj[i][j]: continue     // edge must exist
                dp[mask | (1 << j)][j] = true
    
    // Check if any ending vertex works
    for i = 0 to n-1:
        if dp[FULL][i]: return true
    return false
```

---

## 💻 Pseudocode: Counting Paths

```
function countHamiltonianPaths(adj, n):
    FULL = (1 << n) - 1
    dp = 2D array [FULL+1][n] = 0
    
    for i = 0 to n-1:
        dp[1 << i][i] = 1
    
    for mask = 1 to FULL:
        for i = 0 to n-1:
            if not (mask & (1 << i)): continue
            if dp[mask][i] == 0: continue
            
            for j = 0 to n-1:
                if mask & (1 << j): continue
                if not adj[i][j]: continue
                dp[mask | (1 << j)][j] += dp[mask][i]
    
    total = 0
    for i = 0 to n-1:
        total += dp[FULL][i]
    return total
```

---

## 🔄 Path Reconstruction

```
function reconstructPath(dp, adj, n):
    FULL = (1 << n) - 1
    
    // Find ending vertex
    last = -1
    for i = 0 to n-1:
        if dp[FULL][i]:
            last = i
            break
    
    path = [last]
    mask = FULL
    
    while popcount(mask) > 1:
        prev_mask = mask ^ (1 << last)
        for prev = 0 to n-1:
            if not (prev_mask & (1 << prev)): continue
            if adj[prev][last] and dp[prev_mask][prev]:
                path.prepend(prev)
                mask = prev_mask
                last = prev
                break
    
    return path
```

---

## 🆚 Hamiltonian Path vs Cycle

```
Hamiltonian Path:
  dp[FULL][i] = true for some i
  
Hamiltonian Cycle:
  dp[FULL][i] = true AND edge(i, start) exists
  
  Fix start = 0 (WLOG for undirected):
    dp[(1<<n)-1][i] AND adj[i][0] for any i
    
    ┌─→ 0 → 1 → 2 → 3 ─┐
    └────────────────────┘
```

---

## ⚡ Complexity Analysis

```
┌────────────────────────────────────────┐
│ Time:  O(n² · 2ⁿ)                     │
│   2ⁿ masks × n endpoints × n next     │
│                                        │
│ Space: O(n · 2ⁿ)                      │
│                                        │
│ Comparison:                            │
│   Brute force: O(n!)                   │
│   n=20: 20! ≈ 2.4·10¹⁸                │
│         20²·2²⁰ ≈ 4.2·10⁸  ← feasible│
│                                        │
│ For existence only:                    │
│   Use boolean dp → bit operations      │
│   Pack dp[mask] as single integer      │
│   → O(2ⁿ · n) with bitset tricks      │
└────────────────────────────────────────┘
```

---

## 🧩 Applications

| Application | How |
|-------------|-----|
| Knight's tour | Graph = chess board, edges = knight moves |
| Gray code | Hamiltonian cycle in hypercube graph |
| DNA assembly | Overlap graph, find path through fragments |
| Circuit design | Visit all components exactly once |
| Puzzle solving | Graph of puzzle states |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Path visiting every vertex exactly once |
| **State** | dp[mask][i] = can reach i with visited = mask |
| **Transition** | Extend from neighbor in mask to unvisited vertex |
| **Complexity** | O(n² · 2ⁿ) time, O(n · 2ⁿ) space |
| **Cycle variant** | Check edge from last to start at full mask |
| **Count variant** | Replace boolean with integer count |

---

## ❓ Quick Revision Questions

1. **What is the difference between Hamiltonian Path and Hamiltonian Cycle?**
2. **What does dp[mask][i] represent?**
3. **How do you modify existence check to counting?**
4. **How do you check for a Hamiltonian Cycle using the path DP?**
5. **Why is bitmask DP exponentially faster than brute force for this problem?**
6. **What is the maximum n for which bitmask DP is feasible?**

---

[← Previous: Shortest Path All Nodes](02-shortest-path-all-nodes.md) | [Next: Partition into K Subsets →](04-partition-k-subsets.md)

[← Back to README](../README.md)
