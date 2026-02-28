# Chapter 2: Edge Addition and Cycles

## Overview

When edges are added dynamically, Union-Find can track exactly **which edge** causes the first cycle and how many edges cause cycles. This extends basic cycle detection into richer analyses.

---

## Counting Cycle-Causing Edges

```
In a graph with n nodes and E edges:
  - A spanning forest needs at most n-1 edges (if connected)
  - Every additional edge creates a cycle

  Redundant edges = E - (n - components)
  
  Or equivalently: count how many Union operations FAIL

Example:
  n = 4, edges: (0,1), (1,2), (2,0), (0,3), (3,1)
  
  (0,1): Union succeeds → tree edge
  (1,2): Union succeeds → tree edge  
  (2,0): Union FAILS   → cycle edge!  
  (0,3): Union succeeds → tree edge
  (3,1): Union FAILS   → cycle edge!

  Tree edges: 3, Cycle edges: 2
  Components: 1, so n-1 = 3 tree edges ✓
```

```python
def count_cycles(n, edges):
    uf = UnionFind(n)
    cycle_edges = 0
    
    for u, v in edges:
        if not uf.union(u, v):
            cycle_edges += 1
    
    return cycle_edges
```

---

## First Edge Causing a Cycle

```
Problem: Find the index (or the edge itself) of the 
         FIRST edge that creates a cycle.

n = 5, edges: (0,1), (2,3), (1,2), (0,3), (3,4)

  (0,1): OK → {0,1}
  (2,3): OK → {0,1}, {2,3}
  (1,2): OK → {0,1,2,3}
  (0,3): FAIL! → Find(0)=Find(3) → First cycle at index 3
         Cycle: 0→1→2→3→0

  Return: edge (0,3) at index 3
```

```python
def first_cycle_edge(n, edges):
    uf = UnionFind(n)
    for i, (u, v) in enumerate(edges):
        if not uf.union(u, v):
            return i, (u, v)
    return -1, None  # no cycle
```

---

## Problem: Redundant Connection (LeetCode 684)

```
Given a tree with one extra edge (making exactly one cycle),
find the edge that can be removed to make it a tree again.
If multiple answers, return the one appearing LAST.

Input: edges = [[1,2],[1,3],[2,3]]

  1──2
   \ |    → Triangle! One edge is redundant.
    3

Answer: [2,3] (last edge creating the cycle)
```

```
Why "last edge appearing" works with Union-Find:

Process edges in order:
  Union(1,2): OK
  Union(1,3): OK
  Union(2,3): FAIL → This is the redundant edge!

The LAST edge that creates a cycle is automatically 
the one Union-Find detects, because we process in order.
```

```python
def findRedundantConnection(edges):
    n = len(edges)
    uf = UnionFind(n + 1)  # 1-indexed nodes
    
    for u, v in edges:
        if not uf.union(u, v):
            return [u, v]
```

---

## All Edges in the Cycle

```
What if we need ALL edges that form a cycle?

Union-Find alone can identify the CLOSING edge (the one 
that completes the cycle), but not all edges in the cycle.

For that, combine with path tracking:

Approach:
  1. Build a spanning tree using Union-Find
  2. For each rejected edge (u,v), find the path u→v in tree
  3. The path u→v + edge (u,v) forms the cycle

Or use DFS after detecting that a cycle exists.
```

---

## Edge Processing Order Matters

```
Same graph, different edge order → different cycle edge detected:

Graph: Triangle 0─1─2─0

Order 1: (0,1), (1,2), (2,0) → cycle edge: (2,0)
Order 2: (2,0), (0,1), (1,2) → cycle edge: (1,2)
Order 3: (1,2), (2,0), (0,1) → cycle edge: (0,1)

The LAST edge in the order that closes the cycle is detected.

This matters for problems like "Redundant Connection" where 
the answer depends on edge order!
```

---

## Multiple Cycles

```
A graph can have multiple cycles. Union-Find detects 
them as they form:

n=4, edges: (0,1), (0,2), (1,2), (2,3), (0,3)

  (0,1): OK → tree
  (0,2): OK → tree
  (1,2): FAIL → Cycle 1: 0─1─2─0
  (2,3): OK → tree
  (0,3): FAIL → Cycle 2: 0─2─3─0 (or 0─1─2─3─0)

Two cycle-creating edges detected.
Redundant edge count = 2
To make this a tree: remove 2 edges.
```

---

## Problem: Graph Valid Tree (Revisited)

```
LeetCode 261: Graph Valid Tree

Conditions for a valid tree:
  1. Exactly n-1 edges
  2. No cycles
  3. Fully connected

With Union-Find:
  - Check edge count = n-1
  - Process all edges: if any Union fails → cycle → not a tree
  - After processing: if components = 1 → connected → tree

Actually, if n-1 edges and no cycles → must be connected!
So we only need checks 1 and 2.
```

```python
def validTree(n, edges):
    if len(edges) != n - 1:
        return False
    
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u, v):
            return False  # cycle
    
    return True  # n-1 edges, no cycle → tree
```

---

## Summary Table

| Task | Method | Time |
|------|--------|------|
| Detect any cycle | Union fails → cycle | O(E × α(n)) |
| Count cycle edges | Count failed Unions | O(E × α(n)) |
| Find first cycle edge | Return on first failure | O(E × α(n)) |
| Find last cycle edge | Process all, keep last failure | O(E × α(n)) |
| Find all cycle members | Need DFS supplement | O(V + E) |
| Check if tree | n-1 edges + no cycles | O(E × α(n)) |

---

## Quick Revision Questions

1. **How do you count the number of redundant edges?**
2. **Why does the Redundant Connection problem naturally work with Union-Find?**
3. **Can Union-Find identify ALL edges in a cycle?**
4. **How does edge processing order affect which cycle edge is detected?**
5. **What two conditions make a graph a valid tree?**

---

[← Previous: Cycle in Undirected Graph](01-cycle-in-undirected-graph.md) | [Next: Redundant Connection →](03-redundant-connection.md)
