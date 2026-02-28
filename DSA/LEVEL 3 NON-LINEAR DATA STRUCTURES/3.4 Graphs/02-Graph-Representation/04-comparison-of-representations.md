# Comparison of Representations

[← Previous: Edge List](03-edge-list.md) | [Back to TOC](../README.md) | [Next: Building Graphs from Input →](05-building-graphs-from-input.md)

---

## Visual Comparison

```
    SAME GRAPH — THREE REPRESENTATIONS
    
       (0)──(1)
        │    │
       (2)──(3)

    ┌─────────────────┬─────────────────┬─────────────────┐
    │ ADJACENCY MATRIX│ ADJACENCY LIST  │ EDGE LIST       │
    ├─────────────────┼─────────────────┼─────────────────┤
    │   0 1 2 3       │ 0 → [1, 2]     │ (0,1)           │
    │ 0 [0 1 1 0]     │ 1 → [0, 3]     │ (0,2)           │
    │ 1 [1 0 0 1]     │ 2 → [0, 3]     │ (1,3)           │
    │ 2 [1 0 0 1]     │ 3 → [1, 2]     │ (2,3)           │
    │ 3 [0 1 1 0]     │                 │                 │
    └─────────────────┴─────────────────┴─────────────────┘
```

## Complexity Comparison Table

| Operation | Adj. Matrix | Adj. List | Edge List |
|-----------|------------|-----------|-----------|
| **Space** | O(V²) | O(V + E) | O(E) |
| **Check edge (u,v)** | O(1) | O(deg(u)) | O(E) |
| **All neighbors of u** | O(V) | O(deg(u)) | O(E) |
| **Add edge** | O(1) | O(1) | O(1) |
| **Remove edge** | O(1) | O(deg(u)) | O(E) |
| **Add vertex** | O(V²)* | O(1) | O(1) |
| **Iterate all edges** | O(V²) | O(V + E) | O(E) |

*Adding a vertex to a matrix requires resizing the entire 2D array.

## Decision Guide: Which Representation to Use?

```
    ┌─────────────────────────────────┐
    │    What's your graph like?      │
    └──────────────┬──────────────────┘
                   │
         ┌─────────┴──────────┐
         ▼                    ▼
    Dense (E ≈ V²)?     Sparse (E ≈ V)?
         │                    │
         ▼                    ▼
    Adjacency Matrix     Adjacency List
    (fast edge lookup)   (space efficient)
         
    Need to sort edges?
         │
         ▼
    Edge List → then Kruskal's
    
    Most interview/contest problems:
         │
         ▼
    ★ ADJACENCY LIST ★  (default choice)
```

## Recommendation Summary

| Scenario | Best Representation |
|----------|-------------------|
| General purpose / most problems | **Adjacency List** |
| Dense graph (E ≈ V²) | Adjacency Matrix |
| Need O(1) edge lookup | Adjacency Matrix (or HashSet in adj list) |
| Kruskal's MST | Edge List |
| Very large sparse graph | Adjacency List |
| Simple implementation needed | Adjacency Matrix |
| Floyd-Warshall algorithm | Adjacency Matrix |
| BFS / DFS | Adjacency List |

---

[← Previous: Edge List](03-edge-list.md) | [Back to TOC](../README.md) | [Next: Building Graphs from Input →](05-building-graphs-from-input.md)
