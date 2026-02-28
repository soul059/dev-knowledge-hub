# Chapter 3: Is-Connected Queries

## Overview

The **is-connected query** (also called **connectivity query**) checks whether two nodes belong to the same component. With Union-Find, this is simply comparing roots.

---

## Core Operation

```
connected(u, v) = (Find(u) == Find(v))

That's it! The simplest DSU operation.

Time: O(α(n)) ≈ O(1)
```

---

## Implementation

```python
def connected(self, x, y):
    return self.find(x) == self.find(y)
```

```cpp
bool connected(int x, int y) {
    return find(x) == find(y);
}
```

```java
public boolean connected(int x, int y) {
    return find(x) == find(y);
}
```

---

## Example: Graph Valid Tree

```
LeetCode 261: Graph Valid Tree

A graph is a valid tree if:
  1. It is fully connected (1 component)
  2. It has no cycles (exactly n-1 edges)

Equivalently: A Union that fails (same root) means 
              there's a cycle!

Input: n=5, edges=[[0,1],[0,2],[0,3],[1,4]]
       0
      /|\
     1 2 3
     |
     4
Output: true (connected, no cycles)

Input: n=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false (cycle: 1→2→3→1)
```

```python
def validTree(n, edges):
    if len(edges) != n - 1:
        return False      # Tree must have exactly n-1 edges
    
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u, v):
            return False  # Cycle detected!
    
    return True
    # If n-1 edges and no cycles → must be connected
```

---

## Pattern: Connectivity with Conditions

```
Many problems add conditions to connectivity:

1. "Are u and v connected BY TIME t?"
   → Process edges in time order, query at time t

2. "What is the earliest time u and v become connected?"
   → Binary search on time + Union-Find
   → Or process edges in order, record when connected

3. "Are u and v connected using only edges ≤ weight w?"
   → Sort edges by weight, process until w
```

### Example: Earliest Connection
```python
def earliestConnection(n, edges):
    """edges = [(time, u, v), ...], returns earliest time 
    when ALL nodes are connected, or -1"""
    edges.sort()  # sort by time
    uf = UnionFind(n)
    
    for time, u, v in edges:
        uf.union(u, v)
        if uf.components == 1:
            return time
    
    return -1
```

---

## Batch Connectivity Queries

```
Given a graph with n nodes and m edges, answer q queries:
  "Is u connected to v?"

Approach 1: Build graph, BFS/DFS per query → O(q × (n+m))
Approach 2: Union-Find → O((m + q) × α(n))

Example:
  n=6, edges: (0,1),(1,2),(3,4)
  Queries: (0,2)? (0,3)? (3,4)? (2,5)?

  Build DSU:
    Union(0,1), Union(1,2), Union(3,4)
    Components: {0,1,2}, {3,4}, {5}

  Answer queries:
    connected(0,2)? → Find(0)=0, Find(2)=0 → YES ✓
    connected(0,3)? → Find(0)=0, Find(3)=3 → NO  ✗
    connected(3,4)? → Find(3)=3, Find(4)=3 → YES ✓
    connected(2,5)? → Find(2)=0, Find(5)=5 → NO  ✗
```

```python
def batchConnectivity(n, edges, queries):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    
    return [uf.connected(u, v) for u, v in queries]
```

---

## Interleaved Operations

```
Most powerful pattern: Edges and queries interleaved

operations = [
    ("edge", 0, 1),
    ("edge", 2, 3),
    ("query", 0, 3),    ← NO (separate components)
    ("edge", 1, 2),
    ("query", 0, 3),    ← YES (now connected: 0-1-2-3)
    ("edge", 4, 5),
    ("query", 3, 5),    ← NO
]

Processing:
  Union(0,1) → {0,1}, {2}, {3}, {4}, {5}
  Union(2,3) → {0,1}, {2,3}, {4}, {5}
  Query(0,3) → Find(0)≠Find(3) → NO
  Union(1,2) → {0,1,2,3}, {4}, {5}
  Query(0,3) → Find(0)=Find(3) → YES
  Union(4,5) → {0,1,2,3}, {4,5}
  Query(3,5) → Find(3)≠Find(5) → NO
```

---

## Common Pitfall: Directed Edges

```
⚠ Union-Find handles UNDIRECTED connectivity only!

For directed graphs, "u can reach v" ≠ "v can reach u"

Directed graph:     Union-Find sees:
  0 → 1 → 2          0 — 1 — 2
  
  Can 2 reach 0?      connected(2,0)? → YES
  Actually: NO!       (WRONG for directed!)

For directed connectivity, use:
  - DFS/BFS from each node, or
  - Tarjan's algorithm (SCCs), or
  - Topological sort + reachability
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Operation | connected(u, v) = Find(u) == Find(v) |
| Time | O(α(n)) per query |
| Returns | Boolean |
| Pattern | Build DSU first, then query |
| Limitation | Undirected connectivity only |
| Common use | Graph valid tree, friend circles, network checks |

---

## Quick Revision Questions

1. **How do you check if two nodes are connected using Union-Find?**
2. **What is the time complexity of a connectivity query?**
3. **How can Union-Find detect cycles during edge insertion?**
4. **Why can't Union-Find handle directed connectivity?**
5. **How would you find the earliest time all nodes are connected?**

---

[← Previous: Number of Components](02-number-of-components.md) | [Next: Size of Components →](04-size-of-components.md)
