# Chapter 3: Union-Find Integration

## Overview

Union-Find is what makes Kruskal's algorithm efficient. Without it, checking "does this edge create a cycle?" would require a full graph traversal per edge. This chapter details exactly how DSU integrates into the MST algorithm.

---

## The Role of Union-Find in Kruskal's

```
For each edge (u, v, w) in sorted order:

┌─────────────────────────────────────────────┐
│  Question: Does adding (u,v) create a cycle?│
│                                             │
│  Without DSU: Run DFS/BFS from u to v       │
│               → O(V + E) per edge           │
│               → O(E × (V+E)) total          │
│                                             │
│  With DSU: Find(u) == Find(v)?              │
│            → O(α(n)) per edge               │
│            → O(E × α(n)) total              │
└─────────────────────────────────────────────┘
```

---

## Integration Pattern

```python
def kruskal(n, edges):
    # 1. Sort edges
    edges.sort(key=lambda e: e[2])
    
    # 2. Initialize DSU
    parent = list(range(n))
    rank = [0] * n
    
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    
    def union(x, y):
        rx, ry = find(x), find(y)
        if rx == ry: return False
        if rank[rx] < rank[ry]: rx, ry = ry, rx
        parent[ry] = rx
        if rank[rx] == rank[ry]: rank[rx] += 1
        return True
    
    # 3. Greedy edge selection
    mst_weight = 0
    mst_edges = []
    
    for u, v, w in edges:
        if union(u, v):           # DSU: cycle check + merge
            mst_weight += w
            mst_edges.append((u, v, w))
            if len(mst_edges) == n - 1:
                break
    
    return mst_weight, mst_edges
```

---

## What DSU Does At Each Step

```
Sorted edges: (A,B,1), (C,D,2), (B,C,3), (A,D,4), (B,D,5)

Step 1: Edge (A,B,1)
  DSU state: {A} {B} {C} {D}
  Find(A)=A, Find(B)=B → different
  Union(A,B) → {A,B} {C} {D}    ← merge components
  MST += edge (A,B,1)

Step 2: Edge (C,D,2)
  DSU state: {A,B} {C} {D}
  Find(C)=C, Find(D)=D → different
  Union(C,D) → {A,B} {C,D}     ← merge components
  MST += edge (C,D,2)

Step 3: Edge (B,C,3)
  DSU state: {A,B} {C,D}
  Find(B)=A, Find(C)=C → different
  Union(B,C) → {A,B,C,D}       ← merge components
  MST += edge (B,C,3)
  
  MST has n-1 = 3 edges → DONE!

Step 4: Edge (A,D,4) — NOT PROCESSED (early termination)
Step 5: Edge (B,D,5) — NOT PROCESSED

DSU prevented us from adding (A,D) or (B,D):
  If we had processed them: Find(A)=Find(D) → cycle!
```

---

## DSU Operations Per Edge

```
Each edge requires:
  1. Find(u)   — find root of u
  2. Find(v)   — find root of v
  3. Compare   — same root?
  4. Union     — merge if different (at most n-1 times)

Total DSU operations:
  - 2E Find operations (2 per edge)
  - At most n-1 Union operations (one per MST edge)
  - Total: O(E × α(n))

Since α(n) ≤ 4 for all practical n:
  DSU contributes effectively O(E) work
  → Sorting's O(E log E) dominates
```

---

## Handling Disconnected Graphs

```
If the graph is not connected, Kruskal's produces a 
MINIMUM SPANNING FOREST (MSF) — a separate MST for 
each component.

After processing all edges:
  if len(mst_edges) < n - 1:
      # Graph is disconnected
      # mst_edges forms a minimum spanning forest
      # Number of components = n - len(mst_edges)

Check connectivity:
```

```python
def kruskal_with_check(n, edges):
    edges.sort(key=lambda e: e[2])
    uf = UnionFind(n)
    mst_weight = 0
    mst_count = 0
    
    for u, v, w in edges:
        if uf.union(u, v):
            mst_weight += w
            mst_count += 1
    
    if mst_count < n - 1:
        return -1       # disconnected — no MST exists
    return mst_weight
```

---

## Verification: Is the MST Correct?

```
To verify a claimed MST, check these properties:

1. Has exactly n-1 edges ✓
2. Total weight matches ✓
3. No cycles (every edge connects different components) ✓
4. Adding any non-MST edge creates a cycle where it's 
   the heaviest edge ✓ (optimality)

Union-Find naturally ensures properties 1 and 3.
```

---

## Complete Implementation with Full Details

```python
class KruskalMST:
    def __init__(self, n):
        self.n = n
        self.parent = list(range(n))
        self.rank = [0] * n
        self.mst_edges = []
        self.mst_weight = 0
        self.is_connected = False
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False
        if self.rank[rx] < self.rank[ry]: rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]: self.rank[rx] += 1
        return True
    
    def compute(self, edges):
        """Run Kruskal's algorithm."""
        edges.sort(key=lambda e: e[2])
        
        for u, v, w in edges:
            if self.union(u, v):
                self.mst_edges.append((u, v, w))
                self.mst_weight += w
                if len(self.mst_edges) == self.n - 1:
                    break
        
        self.is_connected = (len(self.mst_edges) == self.n - 1)
        return self.mst_weight if self.is_connected else -1
```

---

## Summary Table

| DSU Operation | Role in Kruskal's | Calls |
|--------------|-------------------|-------|
| Find(u) | Get component of u | 2 per edge |
| Find(v) | Get component of v | 2 per edge |
| Compare roots | Check for cycle | 1 per edge |
| Union(u, v) | Merge components | At most n-1 |
| Total DSU work | | O(E × α(n)) |

---

## Quick Revision Questions

1. **What would the complexity of Kruskal's be without Union-Find?**
2. **How many Union operations can succeed at most?**
3. **What does DSU output when the graph is disconnected?**
4. **Why does the sorting step dominate the total time?**
5. **What is the significance of early termination at n-1 edges?**

---

[← Previous: Edge Sorting](02-edge-sorting.md) | [Next: Minimum Spanning Forest →](04-minimum-spanning-forest.md)
