# Chapter 5: Applications Overview

## Overview

Union-Find is surprisingly versatile. This chapter surveys the major application areas where DSU provides elegant and efficient solutions.

---

## Application 1: Dynamic Connectivity

The most natural application — tracking connected components as edges are added to a graph.

```
Problem: Given vertices and edges added one by one,
         answer "Are u and v connected?" queries.

  Edge (0,1) added:    0─1    2    3    4
  Edge (2,3) added:    0─1    2─3  4
  Query: 0 and 3?      → NO (different components)
  Edge (1,2) added:    0─1─2─3    4
  Query: 0 and 3?      → YES (same component!)

  DSU Solution:
    Union(0,1) → Union(2,3) → Find(0)==Find(3)? NO
    → Union(1,2) → Find(0)==Find(3)? YES!
```

---

## Application 2: Network Connectivity

```
Computer Network:
  ┌────┐     ┌────┐     ┌────┐
  │ PC1├─────┤ PC2├─────┤ PC3│
  └────┘     └────┘     └────┘
                              
  ┌────┐     ┌────┐          
  │ PC4├─────┤ PC5│          
  └────┘     └────┘          

  Q: Can PC1 reach PC5?   → Find(1) ≠ Find(5) → NO
  Q: Can PC1 reach PC3?   → Find(1) == Find(3) → YES
  
  Action: Connect PC3-PC4 → Union(3, 4)
  Q: Can PC1 reach PC5?   → Find(1) == Find(5) → YES!
```

---

## Application 3: Kruskal's Minimum Spanning Tree

```
Graph with weights:
    1       4
A───────B───────C
|       |       |
|2      |3      |5
|       |       |
D───────E───────F
    6       7

Kruskal's: Sort edges by weight, add if no cycle

Edge (A,B) w=1: Union(A,B) ✓  (no cycle)
Edge (A,D) w=2: Union(A,D) ✓  (no cycle)
Edge (B,E) w=3: Union(B,E) ✓  (no cycle)
Edge (B,C) w=4: Union(B,C) ✓  (no cycle)
Edge (C,F) w=5: Union(C,F) ✓  (no cycle)
Edge (D,E) w=6: Find(D)==Find(E)? YES → SKIP (cycle!)
Edge (E,F) w=7: Find(E)==Find(F)? YES → SKIP (cycle!)

MST edges: (A,B), (A,D), (B,E), (B,C), (C,F)
```

---

## Application 4: Cycle Detection in Undirected Graphs

```
Algorithm: For each edge (u, v):
  If Find(u) == Find(v) → CYCLE FOUND!
  Else → Union(u, v)

Example:
  Edges: (0,1), (1,2), (2,0)

  Edge (0,1): Find(0)=0, Find(1)=1 → different → Union(0,1)
  Edge (1,2): Find(1)=0, Find(2)=2 → different → Union(1,2)
  Edge (2,0): Find(2)=0, Find(0)=0 → SAME! → CYCLE DETECTED!

     0───1
      \ /
       2     ← this edge creates the cycle
```

---

## Application 5: Image Processing

```
Connected Component Labeling:
  Find connected regions in a binary image

  Image:               Labels:
  1 1 0 0 1            A A . . B
  1 1 0 1 1            A A . B B
  0 0 0 1 0            . . . B .
  1 0 0 0 0            C . . . .
  1 1 0 0 0            C C . . .

  DSU groups adjacent 1-pixels into components:
  Component A: {(0,0), (0,1), (1,0), (1,1)}
  Component B: {(0,4), (1,3), (1,4), (2,3)}
  Component C: {(3,0), (4,0), (4,1)}
```

---

## Application 6: Equivalence Classes

```
Problem: Given pairs of equivalent items, 
         group all equivalent items.

  Pairs: (a,b), (b,c), (d,e), (e,f), (a,f)
  
  Union(a,b): {a,b} {c} {d} {e} {f}
  Union(b,c): {a,b,c} {d} {e} {f}
  Union(d,e): {a,b,c} {d,e} {f}
  Union(e,f): {a,b,c} {d,e,f}
  Union(a,f): {a,b,c,d,e,f}        ← all connected!
  
  Result: One equivalence class containing everything
```

---

## Application 7: Percolation

```
Problem: Does water flow from top to bottom?

  ░░░░█░░░       ░ = open site
  ░█░░░░█░       █ = blocked site
  ░░█░░█░░
  █░░░█░░░       Connected top-to-bottom
  ░░█░░░░█       path exists? → YES
  ░░░█░░░░

  DSU approach:
  - Add virtual "top" node connected to all top-row open sites
  - Add virtual "bottom" node connected to all bottom-row open sites
  - Union adjacent open sites
  - Check: Find(top) == Find(bottom)?
```

---

## Application 8: Least Common Ancestor (Offline)

```
Tarjan's Offline LCA Algorithm uses DSU:

         1
        / \
       2   3
      / \   \
     4   5   6

  Process nodes in DFS order
  Use Union-Find to efficiently answer 
  LCA queries for already-visited nodes
```

---

## Applications Summary

```
┌──────────────────────────────────────────────────────┐
│                DSU APPLICATION MAP                    │
├──────────────────────┬───────────────────────────────┤
│ Graph Theory         │ Connected components          │
│                      │ Cycle detection               │
│                      │ MST (Kruskal's)              │
│                      │ Bipartiteness checking        │
├──────────────────────┼───────────────────────────────┤
│ Network Analysis     │ Network connectivity          │
│                      │ Social network groups         │
│                      │ Computer network routing      │
├──────────────────────┼───────────────────────────────┤
│ Image Processing     │ Connected component labeling  │
│                      │ Percolation                   │
│                      │ Region merging                │
├──────────────────────┼───────────────────────────────┤
│ Other                │ Equivalence classes           │
│                      │ Account merging               │
│                      │ Offline LCA                   │
│                      │ Dynamic connectivity          │
└──────────────────────┴───────────────────────────────┘
```

---

## Summary Table

| Application | DSU Role | Key Operations |
|-------------|----------|----------------|
| Dynamic Connectivity | Track components as edges added | Union + Find |
| Kruskal's MST | Detect cycles during edge addition | Find to check, Union to add |
| Cycle Detection | Check if edge creates cycle | Find only |
| Image Labeling | Group adjacent pixels | Union neighbors |
| Percolation | Check top-to-bottom flow | Union + virtual nodes |
| Account Merging | Group accounts by shared info | Union on shared attributes |
| Network Connectivity | Track reachable nodes | Find for queries |

---

## Quick Revision Questions

1. **How does DSU detect a cycle when adding an edge (u, v)?**
2. **What role does DSU play in Kruskal's algorithm?**
3. **What are "virtual nodes" in percolation problems?**
4. **Why is DSU preferred over BFS/DFS for dynamic connectivity?**
5. **How does DSU help in account merging problems?**

---

[← Previous: Basic Operations](04-basic-operations.md) | [Next: When to Use DSU →](06-when-to-use-dsu.md)
