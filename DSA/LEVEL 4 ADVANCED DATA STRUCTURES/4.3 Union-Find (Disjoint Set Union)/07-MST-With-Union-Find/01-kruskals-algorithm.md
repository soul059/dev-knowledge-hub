# Chapter 1: Kruskal's Algorithm

## Overview

**Kruskal's Algorithm** finds the **Minimum Spanning Tree (MST)** of a weighted, connected, undirected graph. It uses Union-Find as its core data structure, making it one of the most elegant applications of DSU.

---

## What is a Minimum Spanning Tree?

```
Given a connected, undirected, weighted graph:

MST = A subset of edges that:
  1. Connects all vertices (spanning)
  2. Forms a tree (no cycles, n-1 edges)
  3. Has minimum total edge weight

Example graph:
     2      3
  0────1────2
  |   /|    |
  6  8 5    7
  |/   |    |
  3────4    5
     9

MST edges (total weight = 2+3+5+6+7 = 23... let me recalculate):
  Actually, pick minimum edges avoiding cycles:
  Edge(0,1)=2, Edge(1,2)=3, Edge(1,4)=5, Edge(0,3)=6, Edge(2,5)=7
  Total = 2 + 3 + 5 + 6 + 7 = 23
```

---

## Kruskal's Algorithm

```
1. Sort all edges by weight (ascending)
2. Initialize Union-Find with n nodes
3. For each edge (u, v, weight) in sorted order:
   a. If Find(u) ≠ Find(v):      ← different components
      - Include this edge in MST
      - Union(u, v)
   b. If Find(u) == Find(v):     ← same component
      - Skip (would create cycle)
4. Stop when MST has n-1 edges

Time: O(E log E + E × α(n)) = O(E log E)
  - Sorting dominates: O(E log E)
  - Union-Find operations: O(E × α(n))
```

---

## Visual Walkthrough

```
Graph:
  Nodes: 0, 1, 2, 3, 4
  Edges (sorted by weight):
    (0,1, w=1), (2,3, w=2), (1,2, w=3), (0,3, w=4), 
    (1,3, w=5), (3,4, w=6), (2,4, w=7)

Step 1: Edge (0,1, w=1)
  Find(0)=0, Find(1)=1 → different → INCLUDE
  MST: {(0,1)}  Total: 1
  
  0──1    2   3   4

Step 2: Edge (2,3, w=2)
  Find(2)=2, Find(3)=3 → different → INCLUDE
  MST: {(0,1), (2,3)}  Total: 3
  
  0──1    2──3   4

Step 3: Edge (1,2, w=3)
  Find(1)=0, Find(2)=2 → different → INCLUDE
  MST: {(0,1), (2,3), (1,2)}  Total: 6
  
  0──1──2──3   4

Step 4: Edge (0,3, w=4)
  Find(0)=0, Find(3)=0 → SAME → SKIP (cycle!)
  
  0──1──2──3 would form cycle 0─1─2─3─0

Step 5: Edge (1,3, w=5)
  Find(1)=0, Find(3)=0 → SAME → SKIP

Step 6: Edge (3,4, w=6)
  Find(3)=0, Find(4)=4 → different → INCLUDE
  MST: {(0,1), (2,3), (1,2), (3,4)}  Total: 12
  
  0──1──2──3──4

MST has n-1 = 4 edges → DONE!

Final MST:
  0──1──2──3──4
  w=1  w=3  w=2  w=6
  Total weight: 12
```

---

## Implementation

### Python
```python
def kruskal(n, edges):
    """
    Args:
        n: number of vertices
        edges: list of (u, v, weight)
    Returns:
        mst_edges: list of edges in MST
        total_weight: sum of MST edge weights
    """
    edges.sort(key=lambda e: e[2])  # sort by weight
    
    uf = UnionFind(n)
    mst_edges = []
    total_weight = 0
    
    for u, v, w in edges:
        if uf.union(u, v):           # different components
            mst_edges.append((u, v, w))
            total_weight += w
            if len(mst_edges) == n - 1:
                break                 # MST complete
    
    return mst_edges, total_weight
```

### C++
```cpp
struct Edge {
    int u, v, w;
    bool operator<(const Edge& o) const { return w < o.w; }
};

pair<vector<Edge>, int> kruskal(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end());
    
    UnionFind uf(n);
    vector<Edge> mst;
    int totalWeight = 0;
    
    for (auto& e : edges) {
        if (uf.unite(e.u, e.v)) {
            mst.push_back(e);
            totalWeight += e.w;
            if (mst.size() == n - 1) break;
        }
    }
    
    return {mst, totalWeight};
}
```

---

## Correctness: Why Does This Work?

```
The GREEDY CHOICE property:

Theorem (Cut Property):
  For any cut (partition into two sets) of the graph,
  the minimum weight edge crossing the cut is in SOME MST.

Kruskal's uses this: When we pick the smallest available 
edge that connects two different components, we're choosing 
the minimum edge across the "cut" between those components.

Proof sketch:
  Suppose the smallest edge e crossing a cut is NOT in MST.
  Add e to MST → creates a cycle.
  This cycle must cross the cut at another edge e'.
  Since e has minimum weight: w(e) ≤ w(e').
  Replace e' with e → MST with same or less weight.
  Contradiction!
```

---

## Complexity Analysis

```
┌────────────────────────┬────────────────────────────┐
│ Phase                  │ Time                       │
├────────────────────────┼────────────────────────────┤
│ Sort edges             │ O(E log E)                 │
│ Initialize DSU         │ O(V)                       │
│ Process edges (Find)   │ O(E × α(V))               │
│ Process edges (Union)  │ O(V × α(V)) [at most V-1] │
├────────────────────────┼────────────────────────────┤
│ Total                  │ O(E log E)                 │
│                        │ = O(E log V)               │
│                        │ (since E ≤ V²)             │
├────────────────────────┼────────────────────────────┤
│ Space                  │ O(V) for DSU               │
└────────────────────────┴────────────────────────────┘

Note: O(E log E) = O(E log V²) = O(2E log V) = O(E log V)
```

---

## When to Use Kruskal's vs Prim's

```
┌──────────────────┬────────────────┬────────────────┐
│ Aspect           │ Kruskal's      │ Prim's         │
├──────────────────┼────────────────┼────────────────┤
│ Approach         │ Edge-centric   │ Vertex-centric │
│ Sort             │ Sort all edges │ Priority queue │
│ Data structure   │ Union-Find     │ Min-heap       │
│ Time (adj list)  │ O(E log E)     │ O(E log V)     │
│ Best for         │ Sparse graphs  │ Dense graphs   │
│ Edge list input  │ Natural        │ Need adj list  │
│ Parallelizable   │ Harder         │ Easier         │
│ Multiple MSTs    │ Can detect     │ Less obvious   │
└──────────────────┴────────────────┴────────────────┘

Rule of thumb:
  E ≈ V (sparse):  Kruskal's ≈ O(V log V)
  E ≈ V² (dense):  Kruskal's ≈ O(V² log V) — Prim's better
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Sort edges + Union-Find |
| Time | O(E log E) |
| Space | O(V) for DSU |
| Key insight | Greedy: pick lightest non-cycle edge |
| DSU role | Detect if edge creates cycle |
| Result | n-1 edges forming MST |
| Correctness | Cut property ensures optimality |

---

## Quick Revision Questions

1. **What are the three steps of Kruskal's algorithm?**
2. **Why do we skip an edge when Find(u) == Find(v)?**
3. **What is the Cut Property and why does it guarantee correctness?**
4. **What is the time complexity and which phase dominates?**
5. **When is Kruskal's preferred over Prim's?**

---

[← Previous: Cycle Detection Summary](../06-Cycle-Detection/05-implementation-approach.md) | [Next: Edge Sorting →](02-edge-sorting.md)
