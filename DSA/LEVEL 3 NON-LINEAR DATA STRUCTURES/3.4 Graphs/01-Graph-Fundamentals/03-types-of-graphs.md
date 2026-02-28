# Types of Graphs

[← Previous: Graph Terminology](02-graph-terminology.md) | [Back to TOC](../README.md) | [Next: Graph Properties →](04-graph-properties.md)

---

## Directed vs Undirected

```
    UNDIRECTED GRAPH              DIRECTED GRAPH (Digraph)
    
    Edge (A,B) = Edge (B,A)       Edge (A→B) ≠ Edge (B→A)

    (A)──────(B)                  (A)──────→(B)
     │        │                    ↑          │
     │        │                    │          ▼
    (C)──────(D)                  (C)←──────(D)

    Edges have NO                 Edges have a
    direction                     DIRECTION
    
    Examples:                     Examples:
    - Facebook friends            - Twitter follows
    - Road (two-way)              - One-way street
    - Undirected edge             - Web page links
```

## Weighted vs Unweighted

```
    UNWEIGHTED GRAPH              WEIGHTED GRAPH
    
    All edges have                Each edge has a
    equal "cost"                  numeric weight/cost

    (A)──(B)                      (A)──5──(B)
     │    │                        │       │
    (C)──(D)                      2│       │3
                                   │       │
                                  (C)──1──(D)

    Edge = exists or not          Edge = exists with a cost
    
    Examples:                     Examples:
    - Is there a road?            - Distance between cities
    - Are they friends?           - Travel time
    - Boolean relationship        - Cost, bandwidth, etc.
```

## Other Important Types

```
    COMPLETE GRAPH (K₅)         BIPARTITE GRAPH          DAG (Directed Acyclic Graph)
    Every pair connected        Two groups, edges only    Directed + No cycles
                                between groups

      (A)                       Set 1    Set 2            (A)──→(B)
     /│\ \                     (1)───────(A)               │      │
    / │ \ \                    (2)───┐   (B)               ▼      ▼
   (B)│(E) \                   │    └──→(C)              (C)──→(D)
    \│/ \ /                    (3)───────(D)                     │
     │ X │                                                       ▼
    (C)─(D)                                                     (E)
    
    Edges = V(V-1)/2           No edges within            Used for:
    (for undirected)           same group                 - Task scheduling
                                                          - Dependencies
```

## Quick Type Classification

```
    ┌─────────────────────────────────────────┐
    │            Is there direction?           │
    │           ┌──────┴──────┐               │
    │          YES            NO              │
    │       Directed       Undirected         │
    │           │              │               │
    │     ┌─────┴────┐   ┌────┴────┐         │
    │   Cyclic?    Acyclic?                    │
    │     │          │                         │
    │  Directed    DAG                         │
    │   Graph                                  │
    │                                          │
    │        Are edges weighted?               │
    │       ┌──────┴──────┐                   │
    │      YES            NO                  │
    │   Weighted      Unweighted              │
    └─────────────────────────────────────────┘
```

---

[← Previous: Graph Terminology](02-graph-terminology.md) | [Back to TOC](../README.md) | [Next: Graph Properties →](04-graph-properties.md)
