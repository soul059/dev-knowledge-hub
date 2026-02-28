# Chapter 1: Dynamic Connectivity

## Overview

**Dynamic connectivity** is the problem of maintaining a graph where edges are added over time and answering queries about whether two nodes are connected. Union-Find is the ideal data structure for this when edges are **only added** (not removed).

---

## Problem Definition

```
Given:
  - n nodes, initially disconnected
  - A stream of operations:
    1. ADD(u, v)   — add an edge between u and v
    2. QUERY(u, v) — are u and v connected?

Goal: Answer each query efficiently.

┌────────────────────────────────────────────────────┐
│  This is the CORE problem that Union-Find solves!  │
│                                                    │
│    ADD(u, v)   → Union(u, v)                       │
│    QUERY(u, v) → Find(u) == Find(v)                │
└────────────────────────────────────────────────────┘
```

---

## Connectivity Evolution

```
Start: 5 nodes, no edges

  0   1   2   3   4     Components: {0},{1},{2},{3},{4}

ADD(0, 1):
  0──1   2   3   4     Components: {0,1},{2},{3},{4}

QUERY(0, 1) → YES (same component)
QUERY(0, 2) → NO  (different components)

ADD(2, 3):
  0──1   2──3   4     Components: {0,1},{2,3},{4}

ADD(1, 3):
  0──1
     |               Components: {0,1,2,3},{4}
  2──3         4

QUERY(0, 3) → YES (connected through 1)
QUERY(0, 4) → NO

ADD(0, 4):
  0──1
  |  |               Components: {0,1,2,3,4}
  4  3──2

QUERY(2, 4) → YES (all connected now)
```

---

## Union-Find Solution

```python
def solve_dynamic_connectivity(n, operations):
    uf = UnionFind(n)
    results = []
    
    for op, u, v in operations:
        if op == "ADD":
            uf.union(u, v)
        elif op == "QUERY":
            results.append(uf.connected(u, v))
    
    return results

# Time: O(m × α(n)) for m operations
# Space: O(n)
```

---

## Why Union-Find Excels Here

```
Alternative approaches:

1. BFS/DFS per query:
   - ADD: O(1) — just add to adjacency list
   - QUERY: O(n + m) — full traversal
   - Total: O(q × (n + m))  ← too slow!

2. Adjacency matrix with Floyd-Warshall:
   - Update after each edge: O(n²)
   - Total: O(q × n²)  ← too slow!

3. Union-Find:
   - ADD: O(α(n))
   - QUERY: O(α(n))
   - Total: O(q × α(n)) ≈ O(q) ← OPTIMAL!

Comparison for n = 10^5, q = 10^5:
┌─────────────┬──────────────┬────────────┐
│ Approach     │ Operations   │ Time       │
├─────────────┼──────────────┼────────────┤
│ BFS/DFS     │ 10^5 × 10^5  │ 10^10      │
│ Union-Find  │ 10^5 × 4     │ 4×10^5     │
└─────────────┴──────────────┴────────────┘
```

---

## Online vs Offline Processing

```
ONLINE dynamic connectivity:
  - Process operations as they arrive
  - Must answer queries immediately
  - Union-Find handles this perfectly
  
OFFLINE dynamic connectivity:
  - Read ALL operations first, then process
  - Can sometimes reorder for efficiency
  - Useful when edges are both added AND removed

┌─────────────────────────────────────────────────┐
│  Union-Find = ONLINE dynamic connectivity       │
│  (edge additions only, no deletions)            │
│                                                 │
│  For edge deletions, need more complex          │
│  structures (Link-Cut Trees, ETT)               │
└─────────────────────────────────────────────────┘
```

---

## Example: Social Network

```
Problem: Users join a social network. When two users 
         become friends, can we quickly check if two 
         users are in the same friend circle?

Users: Alice(0), Bob(1), Carol(2), Dave(3), Eve(4)

befriend(Alice, Bob):    Union(0, 1)
befriend(Carol, Dave):   Union(2, 3)
sameFriendCircle(Alice, Dave)?  → Find(0) ≠ Find(3) → NO

befriend(Bob, Carol):    Union(1, 2)
sameFriendCircle(Alice, Dave)?  → Find(0) = Find(3) → YES!

Internal state:
  parent = [0, 0, 0, 2, 4]
  
       0
      / \
     1   2
         |
         3         4

  Alice → Bob → Carol → Dave are all in one circle.
```

---

## Limitations

```
What Union-Find CANNOT do:

1. ✗ Remove edges (disconnect nodes)
2. ✗ Handle directed connectivity
3. ✗ Find shortest path between nodes
4. ✗ Enumerate all nodes in a component efficiently

For these, consider:
  - Link-Cut Trees (dynamic forests with deletion)
  - Euler Tour Trees (dynamic connectivity with deletion)
  - BFS/DFS (shortest path, enumeration)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Are u—v connected after edge additions? |
| DSU mapping | add = Union, query = connected |
| Time per op | O(α(n)) |
| Advantage over BFS | No full traversal per query |
| Limitation | No edge deletions |
| Use cases | Social networks, maze building, network monitoring |

---

## Quick Revision Questions

1. **What two operations define the dynamic connectivity problem?**
2. **How does Union-Find map to dynamic connectivity?**
3. **Why is BFS/DFS inefficient for repeated connectivity queries?**
4. **What is the difference between online and offline dynamic connectivity?**
5. **Name a scenario where Union-Find cannot handle dynamic connectivity.**

---

[← Previous: Inverse Ackermann](../04-Union-By-Rank-Size/06-inverse-ackermann-function.md) | [Next: Number of Components →](02-number-of-components.md)
