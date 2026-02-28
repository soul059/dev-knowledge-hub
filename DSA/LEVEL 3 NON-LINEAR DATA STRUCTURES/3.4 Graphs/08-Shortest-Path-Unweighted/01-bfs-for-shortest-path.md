# BFS for Shortest Path

[Back to TOC](../README.md) | [Next: Single Source Shortest Path →](02-single-source-shortest-path.md)

---

## Why BFS Works for Unweighted Graphs

```
    In an unweighted graph, all edges have "weight 1."
    
    BFS explores level by level:
    Level 0: source        → distance 0
    Level 1: neighbors     → distance 1
    Level 2: next ring     → distance 2
    ...
    
    The FIRST time BFS reaches a vertex, it's via
    the SHORTEST path (fewest edges).
    
    ┌──────────────────────────────────────────┐
    │  BFS guarantees shortest path in         │
    │  UNWEIGHTED graphs because it explores   │
    │  ALL vertices at distance d before       │
    │  ANY vertex at distance d+1.             │
    └──────────────────────────────────────────┘
```

## When BFS Fails (Weighted Graphs)

```
    Weighted graph:
       (A) ─10─ (B)
        │         │
        1         1
        │         │
       (C) ──2── (D)

    BFS from A: discovers B at level 1 (distance = 1 edge)
    But shortest path A→C→D→B = 1+2+1 = 4 (weight)
    vs A→B = 10 (weight)
    
    BFS says distance to B = 1 edge ✗
    Actual shortest weight = 4 via A→C→D→B ✓
    
    For weighted graphs: use Dijkstra's or Bellman-Ford!
```

---

[Back to TOC](../README.md) | [Next: Single Source Shortest Path →](02-single-source-shortest-path.md)
