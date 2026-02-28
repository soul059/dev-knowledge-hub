# Chapter 5: When to Apply Union-Find

## 📋 Chapter Overview
A decision guide for recognizing Union-Find problems, choosing UF over BFS/DFS, and a complete problem-solving flowchart.

---

## 📝 Signal Checklist: When to Use Union-Find

```
  ✅ USE Union-Find when you see:
  
  □ "Group" / "connected component" / "cluster"
  □ Questions about connectivity (is A connected to B?)
  □ Merging sets / groups incrementally
  □ Cycle detection in undirected graph
  □ "Earliest time" when some connectivity condition met
  □ Equivalence relations (a==b, b==c → a==c)
  □ Minimum spanning tree problems
  □ Dynamic connectivity with only additions (no deletions)
  
  ❌ AVOID Union-Find when:
  
  □ Need shortest path (use BFS/Dijkstra)
  □ Need traversal order (use DFS/BFS)
  □ Edges can be removed (UF doesn't support efficient un-union)
  □ Need path information, not just connectivity
  □ Directed graph (UF is for undirected relationships)
```

---

## 📝 Decision Flowchart

```
  Is the problem about CONNECTIVITY?
  │
  ├── NO → Use other pattern
  │
  └── YES
       │
       ├── Need SHORTEST PATH?
       │   └── YES → BFS / Dijkstra / Bellman-Ford
       │
       ├── Need TRAVERSAL ORDER?
       │   └── YES → DFS / BFS
       │
       ├── DIRECTED graph?
       │   └── YES → DFS/BFS (or Tarjan's for SCC)
       │
       └── UNDIRECTED connectivity?
            │
            ├── Static graph?
            │   ├── YES → Either DFS/BFS or UF works
            │   └── Processing edges INCREMENTALLY?
            │       └── YES → ★ UNION-FIND ★
            │
            ├── Need CYCLE detection?
            │   └── YES → ★ UNION-FIND ★ (or DFS)
            │
            ├── Need COMPONENT COUNT?
            │   └── YES → ★ UNION-FIND ★ or DFS/BFS
            │
            ├── MERGING groups?
            │   └── YES → ★ UNION-FIND ★
            │
            └── MST construction?
                └── YES → Kruskal's with ★ UNION-FIND ★
```

---

## 📝 Union-Find vs BFS/DFS Comparison

| Criteria | Union-Find | BFS/DFS |
|----------|-----------|---------|
| Connectivity check | O(α(n)) per query | O(V+E) per query |
| Incremental edges | ✅ Excellent | ❌ Must rebuild |
| Component count | ✅ O(1) maintained | O(V+E) traversal |
| Cycle detection | ✅ O(α(n)) | ✅ O(V+E) |
| Shortest path | ❌ Cannot | ✅ BFS |
| Traversal order | ❌ Cannot | ✅ Natural |
| Edge removal | ❌ Not supported | ✅ Rebuild |
| Directed graphs | ❌ Not applicable | ✅ Supported |
| Space | O(n) | O(V+E) for adj list |

### When UF Clearly Wins

```
  1. MANY connectivity queries on same graph
     → N queries × O(α(n)) vs N queries × O(V+E)
     
  2. Edges arrive one at a time
     → UF: union each edge O(α(n))
     → BFS: rebuild from scratch each time
     
  3. Need real-time component count
     → UF: maintain counter, O(1) lookup
     → BFS: count components = O(V+E)
```

### When BFS/DFS Clearly Wins

```
  1. Need the actual PATH between nodes
  2. Need shortest distance
  3. Need topological order (directed)
  4. Need to enumerate all nodes in a component
```

---

## 📝 Problem Pattern Recognition

| Pattern | Signal Words | Example Problem |
|---------|-------------|-----------------|
| Component counting | "how many groups/islands/components" | Number of Islands, Provinces |
| Cycle detection | "redundant edge", "valid tree" | Redundant Connection, Graph Valid Tree |
| Dynamic merging | "earliest time", "after each operation" | Earliest Moment Friends Connected |
| Equivalence | "equal", "same group", "equivalent" | Equation Satisfaction, Accounts Merge |
| MST | "minimum cost to connect", "spanning tree" | Min Cost to Connect All Points |
| Weighted relation | "ratio", "relative value" | Evaluate Division |

---

## 📝 Implementation Decision Guide

```
  Choosing UF Variant:
  
  Basic UF (parent + find + union)
  └── When: Simple connectivity
  
  UF with Rank/Size
  └── When: Performance matters (keeps tree balanced)
  └── Always use this by default
  
  UF with Count
  └── When: Need to track number of components
  └── Add: count variable, decrement on successful union
  
  UF with component Size
  └── When: Need largest component or component sizes
  └── Add: size[] array, merge smaller into larger
  
  Weighted UF
  └── When: Need ratio/distance between nodes
  └── Add: weight[] array, update during path compression
```

---

## 📝 Common Mistakes & Pitfalls

```
  ❌ Mistake 1: Forgetting path compression
     → Without it: O(n) per find
     → With it: O(α(n)) ≈ O(1)
  
  ❌ Mistake 2: Using UF on directed graphs
     → UF models symmetric relationships only
     → a→b does NOT mean b→a in directed graphs
  
  ❌ Mistake 3: Not handling disconnected nodes
     → Initial state: every node is its own component
     → Count starts at n, not 0
  
  ❌ Mistake 4: Off-by-one in 2D→1D mapping
     → cell(i,j) = i * COLS + j  (not ROWS)
     → Verify: cell(0,0)=0, cell(0,1)=1 ✓
  
  ❌ Mistake 5: Trying to "un-union"
     → UF does not support splitting
     → Alternative: process in reverse (offline)
```

---

## 📝 Complete UF Template (Production-Ready)

```
class UnionFind:
    // Initialize
    function init(n):
        parent = [0, 1, 2, ..., n-1]
        rank   = [0, 0, 0, ..., 0]
        count  = n
        size   = [1, 1, 1, ..., 1]
    
    // Find with path compression
    function find(x):
        if parent[x] ≠ x:
            parent[x] = find(parent[x])
        return parent[x]
    
    // Union by rank, returns true if merged
    function union(x, y):
        rootX = find(x)
        rootY = find(y)
        if rootX == rootY: return false
        
        if rank[rootX] < rank[rootY]:
            parent[rootX] = rootY
            size[rootY] += size[rootX]
        else if rank[rootX] > rank[rootY]:
            parent[rootY] = rootX
            size[rootX] += size[rootY]
        else:
            parent[rootY] = rootX
            size[rootX] += size[rootY]
            rank[rootX]++
        
        count--
        return true
    
    // Helpers
    function connected(x, y): return find(x) == find(y)
    function getCount(): return count
    function getSize(x): return size[find(x)]
```

---

## 📊 Master Problem Table

| Problem | Technique | Difficulty |
|---------|-----------|------------|
| Number of Islands | Grid UF or DFS | Medium |
| Redundant Connection | Cycle detection | Medium |
| Accounts Merge | Group merging + DFS | Medium |
| Graph Valid Tree | n-1 edges + no cycle | Medium |
| Number of Provinces | Adjacency matrix UF | Medium |
| Earliest Friends | Chronological union | Medium |
| Equation Satisfaction | Two-pass equal/not-equal | Medium |
| Evaluate Division | Weighted UF | Medium |
| Surrounded Regions | Dummy node | Medium |
| Kruskal's MST | Sort edges + UF | Medium |
| Largest Component | Prime factor union | Hard |
| Similar String Groups | Pairwise similarity | Hard |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| When to use UF | Undirected connectivity, merging, cycles, MST |
| When NOT to use | Shortest path, traversal order, directed graphs |
| UF vs BFS/DFS | UF for incremental/multi-query; BFS/DFS for paths/order |
| Default template | Always include path compression + union by rank |
| Common pitfall | Directed graphs, forgetting compression, un-union |

---

## ❓ Revision Questions

1. **UF vs BFS for "are A and B connected"?** → Single query: both O(V+E). Multiple queries: UF wins — O(α(n)) each after setup vs O(V+E) each for BFS.
2. **Can UF find shortest path?** → No. UF only answers connectivity (yes/no), not distance or path.
3. **How to handle edge deletions?** → UF doesn't support un-union. Workaround: process in reverse order (add instead of remove).
4. **When is weighted UF needed?** → When you need the ratio or relative value between two nodes (e.g., a/b = ?), not just connectivity.
5. **Why always use rank/size optimization?** → Without it, tree can degenerate to height O(n), making find O(n). With rank, height stays O(log n), and with path compression too, it's O(α(n)).
6. **UF on 2D grid: key formula?** → Map (row, col) to 1D index: row × numCols + col. Optionally add a dummy node for border/special cells.

---

[← Previous: Advanced Techniques](04-advanced-techniques.md) | [Back to Main README →](../README.md)
