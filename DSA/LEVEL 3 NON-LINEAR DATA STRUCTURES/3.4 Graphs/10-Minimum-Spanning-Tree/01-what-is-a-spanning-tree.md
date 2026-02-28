# What is a Spanning Tree?

[Back to TOC](../README.md) | [Next: Kruskal's Algorithm →](02-kruskals-algorithm.md)

---

## Definition

```
    A SPANNING TREE of a connected graph G is a subgraph that:
    1. Includes ALL vertices of G
    2. Is a TREE (connected + acyclic)
    3. Has exactly V - 1 edges
    
    A MINIMUM SPANNING TREE (MST) is the spanning tree
    with the SMALLEST total edge weight.
    
    ┌──────────────────────────────────────────┐
    │  MST connects all vertices with          │
    │  MINIMUM total cost.                     │
    │                                          │
    │  Requirement: connected, undirected,      │
    │  weighted graph.                         │
    └──────────────────────────────────────────┘
```

## Visual Example

```
    Graph:                      MST (bold edges):
       (A)─4─(B)                  (A)   (B)
        │╲    │╲                   │     │╲
        8  2  6  3                 │     6  3
        │    ╲│  ╲                 │      │  ╲
       (C)─7─(D)─1─(E)           (C) 2 (D)─1─(E)
                                      ╲
    Total of all edges: 4+8+2+6+7+3+1 = 31
    MST edges: A─D(2), D─E(1), B─E(3), B─D(6) → Wait...
    
    Actually, pick minimum edges that connect all:
    MST: D─E(1), A─D(2), B─E(3), C─D(7) → Wait, better:
    
    Let me redo: 
    MST: D─E(1), A─D(2) [or A→through D], B─E(3), C─A(8)? 
    
    Proper MST = {D─E:1, A─D:2, B─E:3, C─D:7} 
    Total = 1+2+3+7 = 13
    (V=5 vertices, V-1=4 edges) ✓
```

## Key Properties

```
    1. MST has exactly V - 1 edges
    2. MST is NOT unique (may have ties in weight)
    3. Removing any edge disconnects the MST
    4. Adding any non-MST edge creates exactly one cycle
    5. Cut Property: the lightest edge crossing any cut
       must be in some MST
    6. MST ≠ Shortest Path Tree
```

---

[Back to TOC](../README.md) | [Next: Kruskal's Algorithm →](02-kruskals-algorithm.md)
