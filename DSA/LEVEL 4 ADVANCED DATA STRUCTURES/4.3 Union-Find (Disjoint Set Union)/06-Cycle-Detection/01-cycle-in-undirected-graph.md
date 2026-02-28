# Chapter 1: Cycle in Undirected Graph

## Overview

Union-Find provides an elegant O(E × α(n)) solution for detecting cycles in **undirected graphs**. If a Union operation finds that both endpoints are already in the same set, the edge creates a cycle.

---

## Core Insight

```
When adding edge (u, v):
  
  If Find(u) ≠ Find(v):
    → u and v are in different components
    → This edge connects them (no cycle)
    → Union(u, v)
  
  If Find(u) == Find(v):
    → u and v are ALREADY in the same component
    → There's already a path from u to v
    → Adding this edge creates a CYCLE!

┌──────────────────────────────────────────────────┐
│  CYCLE EXISTS ⟺ Union returns false              │
│  (both nodes already share the same root)        │
└──────────────────────────────────────────────────┘
```

---

## Visual Example

```
Graph: 4 nodes, edges: (0,1), (1,2), (2,3), (3,1)

Process edge (0,1):
  Find(0)=0, Find(1)=1 → different → Union → OK
  0──1   2   3

Process edge (1,2):
  Find(1)=0, Find(2)=2 → different → Union → OK
  0──1──2   3

Process edge (2,3):
  Find(2)=0, Find(3)=3 → different → Union → OK
  0──1──2──3

Process edge (3,1):
  Find(3)=0, Find(1)=0 → SAME! → CYCLE DETECTED!
  
  0──1──2──3
     └─────┘  ← This edge would create cycle 1→2→3→1
```

---

## Implementation

```python
def has_cycle(n, edges):
    """Detect if undirected graph has a cycle."""
    uf = UnionFind(n)
    
    for u, v in edges:
        if not uf.union(u, v):
            return True   # Cycle found!
    
    return False

# Time:  O(E × α(n))
# Space: O(n)
```

```cpp
bool hasCycle(int n, vector<pair<int,int>>& edges) {
    UnionFind uf(n);
    for (auto& [u, v] : edges) {
        if (!uf.unite(u, v))
            return true;  // cycle
    }
    return false;
}
```

```java
boolean hasCycle(int n, int[][] edges) {
    UnionFind uf = new UnionFind(n);
    for (int[] e : edges) {
        if (!uf.union(e[0], e[1]))
            return true;
    }
    return false;
}
```

---

## Complete Trace

```
n = 5, edges: (0,1), (1,2), (2,3), (3,4), (4,1)

parent = [0, 1, 2, 3, 4]
rank   = [0, 0, 0, 0, 0]

Edge (0,1): Find(0)=0, Find(1)=1 → Union
  parent = [0, 0, 2, 3, 4], rank = [1, 0, 0, 0, 0]
  
  0
  |
  1

Edge (1,2): Find(1)=0, Find(2)=2 → Union
  parent = [0, 0, 0, 3, 4], rank = [1, 0, 0, 0, 0]
  
  0
  |\ 
  1  2

Edge (2,3): Find(2)=0, Find(3)=3 → Union
  parent = [0, 0, 0, 0, 4], rank = [1, 0, 0, 0, 0]
  
   0
  /|\ 
 1 2 3

Edge (3,4): Find(3)=0, Find(4)=4 → Union
  parent = [0, 0, 0, 0, 0], rank = [1, 0, 0, 0, 0]
  
    0
  /||\\ 
 1 2 3 4

Edge (4,1): Find(4)=0, Find(1)=0 → SAME ROOT!
  ╔════════════════════════════╗
  ║  CYCLE DETECTED!          ║
  ║  Path: 1→2→3→4→1         ║
  ╚════════════════════════════╝
```

---

## Why This Works

```
Key observation about undirected graphs:

A graph with n nodes has a cycle ⟺ it has more than n-1 edges
  in any connected component

Equivalently:
  A connected graph with n nodes is a tree ⟺ exactly n-1 edges

Union-Find perspective:
  - We start with n components
  - Each successful Union merges 2 → reduces by 1
  - At most n-1 successful Unions possible
  - Any additional edge must connect nodes already connected
  → CYCLE

When we try Union(u,v) and Find(u) == Find(v):
  - u and v are in the same tree
  - There's already a unique path from u to v in the tree
  - Adding edge (u,v) creates a second path → cycle!
```

---

## Union-Find vs DFS for Cycle Detection

```
┌──────────────────┬──────────────────┬──────────────────┐
│ Aspect           │ Union-Find       │ DFS              │
├──────────────────┼──────────────────┼──────────────────┤
│ Time             │ O(E × α(n))      │ O(V + E)         │
│ Space            │ O(V)             │ O(V + E)         │
│ Implementation   │ Simple           │ Need parent track│
│ Online edges?    │ YES              │ NO               │
│ Directed graphs? │ NO               │ YES              │
│ Find cycle path? │ NO (just detect) │ YES (backtrack)  │
│ Build adj list?  │ NO               │ YES              │
└──────────────────┴──────────────────┴──────────────────┘

Choose Union-Find when:
  ✓ Undirected graph
  ✓ Only need to DETECT (not find the actual cycle)
  ✓ Edges arrive one at a time
  ✓ Don't want to build adjacency list
```

---

## Edge Cases

```
1. Self-loops: edge (u, u)
   Find(u) == Find(u) → always detects as cycle
   Some problems exclude self-loops

2. Parallel edges: e.g., two edges (u, v)
   First (u,v): Union succeeds
   Second (u,v): Find(u)==Find(v) → cycle detected

3. No edges: 
   No edges → no cycles

4. Tree (exactly n-1 edges, connected):
   All Unions succeed → no cycle
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Cycle condition | Find(u) == Find(v) when edge (u,v) is added |
| Graph type | Undirected only |
| Time | O(E × α(n)) total |
| Can find cycle edges? | Yes (the edge that triggers it) |
| Can find cycle path? | No (use DFS for that) |
| Handles self-loops? | Yes (detects as cycle) |

---

## Quick Revision Questions

1. **When does adding an edge create a cycle in Union-Find terms?**
2. **Why does Find(u) == Find(v) guarantee a cycle?**
3. **Can Union-Find detect cycles in directed graphs?**
4. **What is the relationship between n-1 edges and trees?**
5. **When would you choose DFS over Union-Find for cycle detection?**

---

[← Previous: Grid Connectivity](../05-Connected-Components/06-grid-connectivity.md) | [Next: Edge Addition and Cycles →](02-edge-addition-and-cycles.md)
