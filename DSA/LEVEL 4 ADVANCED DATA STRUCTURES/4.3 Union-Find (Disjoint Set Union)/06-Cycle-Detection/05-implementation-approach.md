# Chapter 5: Cycle Detection — Implementation Approach

## Overview

This chapter consolidates all cycle detection patterns with Union-Find into a decision framework, covering common variations, pitfalls, and optimization tips.

---

## Decision Framework

```
    Does the graph have cycles?
    │
    ├── Undirected Graph
    │   ├── Just detect? ──── Union-Find (this chapter)
    │   ├── Find cycle path? → DFS with parent tracking
    │   └── Count cycles? ──── E - V + components (Euler)
    │
    └── Directed Graph
        ├── Cycle detection → DFS with coloring (white/gray/black)
        ├── Find SCC ────── → Tarjan's or Kosaraju
        └── Topological sort → Kahn's (cycle = incomplete sort)

    ┌──────────────────────────────────────────────────┐
    │  Union-Find is ONLY for undirected cycle detect! │
    └──────────────────────────────────────────────────┘
```

---

## The Standard Pattern

```python
def detect_cycle_undirected(n, edges):
    """Standard undirected cycle detection with DSU."""
    parent = list(range(n))
    rank = [0] * n
    
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]  # path compression
            x = parent[x]
        return x
    
    def union(x, y):
        rx, ry = find(x), find(y)
        if rx == ry:
            return False          # CYCLE!
        if rank[rx] < rank[ry]:
            rx, ry = ry, rx
        parent[ry] = rx
        if rank[rx] == rank[ry]:
            rank[rx] += 1
        return True
    
    for u, v in edges:
        if not union(u, v):
            return True, (u, v)   # cycle found, return edge
    
    return False, None            # no cycle
```

---

## Variations

### 1. Return the Cycle-Creating Edge
```python
def find_cycle_edge(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u, v):
            return (u, v)
    return None
```

### 2. Return ALL Cycle-Creating Edges
```python
def find_all_cycle_edges(n, edges):
    uf = UnionFind(n)
    cycle_edges = []
    for u, v in edges:
        if not uf.union(u, v):
            cycle_edges.append((u, v))
    return cycle_edges
```

### 3. Minimum Edges to Remove to Make Acyclic
```python
def min_removals_acyclic(n, edges):
    uf = UnionFind(n)
    removals = 0
    for u, v in edges:
        if not uf.union(u, v):
            removals += 1
    return removals
# Same as: len(edges) - (n - uf.components)
```

### 4. Check if Adding a Specific Edge Creates a Cycle
```python
def would_create_cycle(uf, u, v):
    return uf.find(u) == uf.find(v)
# Note: Don't call union — just check!
```

---

## Common Pitfalls

```
Pitfall 1: Forgetting 1-indexed nodes
  
  ❌ uf = UnionFind(n)      # if nodes are 1 to n
  ✓  uf = UnionFind(n + 1)   # need index n

Pitfall 2: Self-loops
  
  Edge (u, u): Find(u) == Find(u) → always "cycle"
  Some problems explicitly exclude self-loops.
  Solution: if u == v: continue  (or handle specially)

Pitfall 3: Parallel edges
  
  Two edges (u, v): First union succeeds, second fails.
  This IS a cycle in multi-graph sense.
  Make sure problem allows parallel edges.

Pitfall 4: Using Union-Find for directed graphs
  
  ❌ Directed cycle: 0→1→2→0
     Union(0,1), Union(1,2), Union(2,0)
     Detects "cycle" but direction matters!
  
  ✓  Use DFS with coloring for directed graphs.

Pitfall 5: Modifying DSU when only checking
  
  ❌ uf.union(u, v)  # modifies state!
  ✓  uf.find(u) == uf.find(v)  # read-only check
```

---

## Performance Comparison

```
Cycle detection methods for undirected graph:

┌────────────────────┬─────────────┬──────────┬──────────┐
│ Method             │ Time        │ Space    │ Detects  │
├────────────────────┼─────────────┼──────────┼──────────┤
│ Union-Find         │ O(E·α(n))   │ O(V)     │ Yes/No   │
│ DFS                │ O(V + E)    │ O(V + E) │ + path   │
│ BFS                │ O(V + E)    │ O(V + E) │ + path   │
│ Edge count per     │ O(V + E)    │ O(V)     │ Yes/No   │
│  component         │             │          │          │
└────────────────────┴─────────────┴──────────┴──────────┘

Union-Find advantages:
  - No adjacency list needed
  - Online edge processing
  - Lower space usage
  - Simpler code

DFS advantages:
  - Can output the actual cycle
  - Works for directed graphs (with modification)
  - O(V + E) vs O(E × α(V)) — technically faster
```

---

## Template: Cycle Detection with Full Context

```python
class CycleDetector:
    """Complete cycle detection utility for undirected graphs."""
    
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.cycle_edges = []
        self.has_cycle = False
    
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x
    
    def add_edge(self, u, v):
        """Add edge. Returns True if edge doesn't create cycle."""
        ru, rv = self.find(u), self.find(v)
        if ru == rv:
            self.has_cycle = True
            self.cycle_edges.append((u, v))
            return False
        if self.rank[ru] < self.rank[rv]:
            ru, rv = rv, ru
        self.parent[rv] = ru
        if self.rank[ru] == self.rank[rv]:
            self.rank[ru] += 1
        return True
    
    def build(self, edges):
        """Process all edges."""
        for u, v in edges:
            self.add_edge(u, v)
    
    def get_cycle_count(self):
        return len(self.cycle_edges)
    
    def get_redundant_edges(self):
        return self.cycle_edges
```

---

## Summary Table

| Scenario | Approach |
|----------|----------|
| Undirected, just detect | Union-Find |
| Undirected, find cycle | DFS |
| Undirected, count redundant edges | Union-Find |
| Directed, detect cycle | DFS with colors |
| Directed, find all cycles | Tarjan's SCC |
| Check before adding edge | Find(u) == Find(v) |

---

## Quick Revision Questions

1. **Why can't Union-Find detect directed cycles?**
2. **What are the five common pitfalls in cycle detection with DSU?**
3. **How do you check if adding an edge would create a cycle WITHOUT modifying the DSU?**
4. **What formula gives the number of redundant edges?**
5. **When should you use DFS instead of Union-Find for cycle detection?**

---

[← Previous: Graph Valid Tree](04-graph-valid-tree.md) | [Next: Unit 7 - Kruskal's Algorithm →](../07-MST-With-Union-Find/01-kruskals-algorithm.md)

[← Back to Main README](../README.md)
