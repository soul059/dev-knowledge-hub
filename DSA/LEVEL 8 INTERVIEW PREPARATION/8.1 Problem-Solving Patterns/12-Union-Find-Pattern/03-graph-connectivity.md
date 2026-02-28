# Chapter 3: Graph Connectivity

## 📋 Chapter Overview
Using Union-Find for advanced graph connectivity: MST (Kruskal's), dynamic connectivity, and graph validity checks.

---

## 📝 Problem 1: Kruskal's Minimum Spanning Tree

**Problem:** Find MST of a weighted undirected graph.

```
  Kruskal's Algorithm:
  1. Sort all edges by weight
  2. For each edge (u, v, w) in sorted order:
     - If u and v are in different components → add edge (union)
     - If same component → skip (would create cycle)
  3. Stop when n-1 edges are added
```

### Pseudocode

```
function kruskal(n, edges):
    sort edges by weight ascending
    uf = UnionFind(n)
    mst = []
    totalWeight = 0
    
    for each (u, v, w) in edges:
        if uf.union(u, v):
            mst.add((u, v, w))
            totalWeight += w
            if mst.size == n - 1: break
    
    return (totalWeight, mst)
```

### Trace

```
  Nodes: 0,1,2,3
  Edges sorted by weight:
  (0,1,1)  (1,2,2)  (0,2,3)  (2,3,4)  (1,3,5)
  
  (0,1,1): union(0,1) → true.  MST: [(0,1,1)]      weight=1
  (1,2,2): union(1,2) → true.  MST: [(0,1,1),(1,2,2)]  weight=3
  (0,2,3): union(0,2) → false! Same component. SKIP.
  (2,3,4): union(2,3) → true.  MST: [(0,1,1),(1,2,2),(2,3,4)]  weight=7
  
  n-1 = 3 edges added. Done.
  
  MST:
  0 ---1--- 1 ---2--- 2 ---4--- 3
  Total weight: 7
```

**Complexity:** O(E log E) for sorting + O(E × α(n)) for unions = **O(E log E)**

---

## 📝 Problem 2: Graph Valid Tree (LeetCode 261)

**Problem:** Given n nodes and edges, check if it forms a valid tree.

```
  Valid tree conditions:
  1. Connected (all nodes in one component)
  2. No cycles (exactly n-1 edges)
  
function validTree(n, edges):
    if len(edges) ≠ n - 1: return false  // quick check
    
    uf = UnionFind(n)
    for each [u, v] in edges:
        if not uf.union(u, v):
            return false   // cycle detected
    
    return true   // n-1 edges, no cycle → connected tree
```

### Trace

```
  n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]
  
  Edge count = 4 = n-1 ✓
  union(0,1) → true
  union(0,2) → true
  union(0,3) → true
  union(1,4) → true
  No cycle → valid tree ✓
  
  n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]
  Edge count = 5 ≠ 4 → false (too many edges for a tree)
```

---

## 📝 Problem 3: Number of Provinces (LeetCode 547)

**Problem:** Given adjacency matrix, find number of connected groups (provinces).

```
function findCircleNum(isConnected):
    n = len(isConnected)
    uf = UnionFind(n)
    
    for i = 0 to n-1:
        for j = i+1 to n-1:
            if isConnected[i][j] == 1:
                uf.union(i, j)
    
    return uf.getCount()
```

---

## 📝 Problem 4: Earliest Time Everyone Becomes Friends (LeetCode 1101)

**Problem:** Logs of friendships with timestamps. Find earliest time when all people are in one group.

```
function earliestAcq(logs, n):
    sort logs by timestamp
    uf = UnionFind(n)
    
    for each [timestamp, x, y] in logs:
        uf.union(x, y)
        if uf.getCount() == 1:
            return timestamp
    
    return -1
```

**Key insight:** Process edges chronologically. The moment component count reaches 1, everyone is connected.

---

## 📝 Problem 5: Satisfiability of Equality Equations (LeetCode 990)

**Problem:** Given equations like "a==b" and "a!=b", check if they can all be satisfied.

```
function equationsPossible(equations):
    uf = UnionFind(26)   // 26 lowercase letters
    
    // First pass: process all '==' equations
    for each eq:
        if eq[1] == '=':
            uf.union(eq[0] - 'a', eq[3] - 'a')
    
    // Second pass: check all '!=' equations
    for each eq:
        if eq[1] == '!':
            if uf.find(eq[0] - 'a') == uf.find(eq[3] - 'a'):
                return false   // contradiction!
    
    return true
```

### Trace

```
  equations = ["a==b", "b!=a"]
  
  Pass 1: union(a, b) → a and b in same group
  Pass 2: "b!=a" → find(b) == find(a) → CONTRADICTION!
  
  Answer: false
  
  equations = ["a==b", "b==c", "a!=c"]
  
  Pass 1: union(a,b), union(b,c) → {a,b,c} same group
  Pass 2: "a!=c" → find(a) == find(c) → CONTRADICTION!
  
  Answer: false
  
  equations = ["a==b", "c!=d"]
  Pass 1: union(a,b)
  Pass 2: "c!=d" → find(c) ≠ find(d) → OK
  Answer: true
```

---

## 📊 Graph Connectivity Summary

| Problem | UF Application | Time |
|---------|---------------|------|
| Kruskal's MST | Sort edges + union, skip cycles | O(E log E) |
| Valid tree | n-1 edges + no cycles | O(n) |
| Provinces | Union connected nodes, count groups | O(n²) |
| Earliest friends | Process by time, check count=1 | O(E log E) |
| Equality equations | Union equals, check not-equals | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Kruskal's MST | Sort edges, greedily add if no cycle (union succeeds) |
| Valid tree | Exactly n-1 edges + no cycles |
| Dynamic connectivity | Process edges in order; track component count |
| Equation satisfaction | Two-pass: union equalities, then verify inequalities |
| Component tracking | UF.count decreases with each successful union |

---

## ❓ Revision Questions

1. **Kruskal's vs Prim's?** → Kruskal: edge-based, sort all edges, use UF. Prim: vertex-based, grow from one node, use heap. Both O(E log E/V).
2. **Valid tree: why check edge count first?** → A tree with n nodes has exactly n-1 edges. More → cycle. Fewer → disconnected. Quick O(1) filter.
3. **Equality equations: why two passes?** → Must establish all equalities first (union), then check inequalities against the groups.
4. **Kruskal's: when to stop?** → After adding n-1 edges (MST of n nodes has n-1 edges).
5. **Dynamic connectivity: UF vs DFS?** → UF handles incremental edge additions in O(α(n)) per edge. DFS would need O(V+E) per query.

---

[← Previous: Classic Problems](02-classic-problems.md) | [Next: Advanced Techniques →](04-advanced-techniques.md)
