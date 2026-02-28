# Algorithm Comparison

[← Previous: Floyd-Warshall Algorithm](03-floyd-warshall-algorithm.md) | [Back to TOC](../README.md) | [Next: Path Reconstruction →](05-path-reconstruction.md)

---

## Decision Flowchart

```
    ┌─────────────────────────────────────┐
    │    Shortest Path: Which Algorithm?  │
    └──────────────┬──────────────────────┘
                   │
         ┌─────────┴──────────┐
         ▼                    ▼
    All pairs?            Single source?
         │                    │
         ▼                    │
    Floyd-Warshall       ┌────┴────┐
    O(V³)                ▼         ▼
                    Negative    No negative
                    weights?    weights?
                         │         │
                         ▼         ▼
                    Bellman-Ford  Dijkstra's
                    O(V × E)     O((V+E)log V)
                         
    Unweighted? → BFS O(V + E)
    Weights 0/1? → 0-1 BFS O(V + E)
```

## Feature Comparison Table

| Feature | BFS | Dijkstra's | Bellman-Ford | Floyd-Warshall |
|---------|-----|-----------|-------------|----------------|
| **Type** | Single source | Single source | Single source | All pairs |
| **Weights** | Unweighted | Non-negative | Any | Any |
| **Negative edges** | ✗ | ✗ | ✓ | ✓ |
| **Negative cycle detect** | ✗ | ✗ | ✓ | ✓ |
| **Time** | O(V+E) | O((V+E)log V) | O(V×E) | O(V³) |
| **Space** | O(V) | O(V+E) | O(V) | O(V²) |
| **Data Structure** | Queue | Min-Heap | Edge list | Matrix |
| **Approach** | Level-order | Greedy | Relaxation | DP |

---

[← Previous: Floyd-Warshall Algorithm](03-floyd-warshall-algorithm.md) | [Back to TOC](../README.md) | [Next: Path Reconstruction →](05-path-reconstruction.md)
