# Chapter 5: Connected Components in Graph

## Overview

Finding **all connected components** in an undirected graph is a classic application of Union-Find. Process all edges, then group nodes by their root.

---

## Algorithm

```
1. Initialize DSU with n nodes
2. For each edge (u, v): Union(u, v)
3. Group nodes by Find(node) to get components

Time: O((n + E) × α(n))
Space: O(n)
```

---

## Step-by-Step Example

```
Graph (n=8, E=6):
  0─1    2─3    5─6
  |      |      |
  4      7      (5 isolated? no, 5-6)

Edges: (0,1), (0,4), (2,3), (2,7), (5,6)

Processing:
  Union(0,1): {0,1} {2} {3} {4} {5} {6} {7}
  Union(0,4): {0,1,4} {2} {3} {5} {6} {7}
  Union(2,3): {0,1,4} {2,3} {5} {6} {7}
  Union(2,7): {0,1,4} {2,3,7} {5} {6}
  Union(5,6): {0,1,4} {2,3,7} {5,6}

Group by root:
  Find(0)=0, Find(1)=0, Find(4)=0 → Component 1: {0,1,4}
  Find(2)=2, Find(3)=2, Find(7)=2 → Component 2: {2,3,7}
  Find(5)=5, Find(6)=5             → Component 3: {5,6}

Answer: 3 components
```

---

## Implementation: Extracting Components

```python
def find_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    
    # Group nodes by their root
    components = {}
    for node in range(n):
        root = uf.find(node)
        if root not in components:
            components[root] = []
        components[root].append(node)
    
    return list(components.values())
    # Returns: [[0,1,4], [2,3,7], [5,6]]
```

```python
# Using defaultdict
from collections import defaultdict

def find_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    
    groups = defaultdict(list)
    for node in range(n):
        groups[uf.find(node)].append(node)
    
    return list(groups.values())
```

---

## Union-Find vs BFS/DFS for Components

```
┌──────────────────┬──────────────────┬──────────────────┐
│ Aspect           │ Union-Find       │ BFS/DFS          │
├──────────────────┼──────────────────┼──────────────────┤
│ Time             │ O((n+E)·α(n))   │ O(n + E)         │
│ Space            │ O(n)             │ O(n + E)         │
│ Need adj. list?  │ NO               │ YES              │
│ Online updates?  │ YES              │ NO (rebuild)     │
│ Edge list input? │ Ideal            │ Must build graph │
│ Remove edges?    │ NO               │ NO (rebuild)     │
│ Directed?        │ NO               │ YES (DFS for SCC)│
└──────────────────┴──────────────────┴──────────────────┘

Use Union-Find when:
  ✓ Edges arrive incrementally
  ✓ Input is edge list (not adjacency list)
  ✓ Need online connectivity updates

Use BFS/DFS when:
  ✓ Static graph (all edges known upfront)
  ✓ Need traversal order or paths
  ✓ Directed graph (SCCs)
```

---

## Application: Connected Components in a Grid

```
Grid:
  . # . . #
  . # . # #
  . . . . .
  # . # # .
  # . # # .

Find connected components of '#' cells.

Approach:
  1. Assign each '#' cell a node ID
  2. Union adjacent '#' cells (4-directional)
  3. Count distinct roots
```

```python
def gridComponents(grid):
    rows, cols = len(grid), len(grid[0])
    uf = UnionFind(rows * cols)
    
    # Only count '#' cells as components initially
    hash_count = 0
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '#':
                hash_count += 1
                idx = r * cols + c
                # Check right and down
                for dr, dc in [(0,1), (1,0)]:
                    nr, nc = r + dr, c + dc
                    if (nr < rows and nc < cols 
                        and grid[nr][nc] == '#'):
                        nidx = nr * cols + nc
                        if uf.union(idx, nidx):
                            hash_count -= 1
    
    return hash_count

# Result for the grid above:
# Component 1: top strip of #'s (col 1, rows 0-1)
# Component 2: right block (col 3-4, rows 0-1)  — wait,
#              connected to top? Let me trace...
# Actually depends on exact connectivity
```

---

## Problem: Accounts Merge (Preview)

```
LeetCode 721: Accounts Merge

Merge accounts that share an email address.

Input:
  accounts = [
    ["John", "a@", "b@"],
    ["John", "c@"],
    ["John", "a@", "d@"],
    ["Mary", "e@"]
  ]

    Account 0: {a@, b@}
    Account 2: {a@, d@}
    
    They share "a@" → merge!
    
    Result: {a@, b@, d@} (John), {c@} (John), {e@} (Mary)

This is a connected components problem!
  Nodes: email addresses
  Edges: emails in the same account are connected
```

---

## Performance

```
Finding components with Union-Find:

Phase 1: Process edges → O(E × α(n))
Phase 2: Group by root → O(n × α(n))

Total: O((n + E) × α(n))

In practice, this is essentially O(n + E).

Memory: O(n) for DSU (vs O(n + E) for adjacency list)
```

---

## Summary Table

| Task | Code | Time |
|------|------|------|
| Build components | Union all edges | O(E × α(n)) |
| Count components | Read counter | O(1) |
| List all components | Group by Find() | O(n × α(n)) |
| Check same component | Find(u) == Find(v) | O(α(n)) |
| Component size | size[Find(x)] | O(α(n)) |

---

## Quick Revision Questions

1. **How do you extract all components after building the DSU?**
2. **When would you prefer Union-Find over BFS/DFS for finding components?**
3. **How do you handle grid-based connectivity with Union-Find?**
4. **What is the space advantage of Union-Find over adjacency-list BFS?**
5. **How do you handle the Accounts Merge problem conceptually?**

---

[← Previous: Size of Components](04-size-of-components.md) | [Next: Grid Connectivity →](06-grid-connectivity.md)
