# MST Applications and Variations

[← Previous: Kruskal's vs Prim's](04-kruskals-vs-prims.md) | [Back to TOC](../README.md) | [Next: Common Problem Patterns →](06-common-problem-patterns.md)

---

## Real-World Applications

```
    ┌──────────────────────────────────────┐
    │         MST Applications             │
    ├──────────────────────────────────────┤
    │                                      │
    │  Network Design                      │
    │  ├── Cable/fiber optic layout        │
    │  ├── Road construction              │
    │  └── Pipeline networks              │
    │                                      │
    │  Clustering                          │
    │  ├── Remove expensive edges → clusters│
    │  └── K-nearest neighbors             │
    │                                      │
    │  Approximation Algorithms            │
    │  ├── TSP approximation (2×MST)       │
    │  └── Steiner tree approximation      │
    │                                      │
    │  Image Processing                    │
    │  └── Image segmentation             │
    └──────────────────────────────────────┘
```

## Variation: Minimum Spanning Forest (MSF)

```
    If the graph is DISCONNECTED, there's no spanning tree.
    Instead, find MST for each connected component.
    
    Kruskal's handles this naturally:
    - Process all edges
    - Result may have < V-1 edges
    - Each component gets its own spanning tree
```

## Variation: Maximum Spanning Tree

```
    Want the MAXIMUM total weight instead?
    
    Option 1: Negate all weights → run MST → negate back
    Option 2: Sort edges in DESCENDING order in Kruskal's
    Option 3: Use max-heap in Prim's
```

## Variation: Second Minimum Spanning Tree

```
    Find the second-smallest spanning tree.
    
    Approach:
    1. Find MST
    2. For each MST edge:
       - Remove it
       - Find new MST
       - Track minimum among all "replacement MSTs"
    3. The smallest of these is the second MST
    
    Time: O(V × E log E)
```

---

[← Previous: Kruskal's vs Prim's](04-kruskals-vs-prims.md) | [Back to TOC](../README.md) | [Next: Common Problem Patterns →](06-common-problem-patterns.md)
