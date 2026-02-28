# Kruskal's vs Prim's

[← Previous: Prim's Algorithm](03-prims-algorithm.md) | [Back to TOC](../README.md) | [Next: MST Applications →](05-mst-applications.md)

---

## Feature Comparison

| Feature | Kruskal's | Prim's |
|---------|-----------|--------|
| **Approach** | Edge-centric (sort edges) | Vertex-centric (grow tree) |
| **Data Structure** | Union-Find | Min-Heap (Priority Queue) |
| **Graph Type** | Edge list | Adjacency list |
| **Time** | O(E log E) | O((V+E) log V) |
| **Better for** | Sparse graphs (E ≈ V) | Dense graphs (E ≈ V²) |
| **Starting point** | Any (processes all edges) | Specific vertex |
| **Disconnected graph** | Finds MSF naturally | Needs loop over components |
| **Implementation** | Simpler (sort + UF) | Similar to Dijkstra's |
| **Result** | Same MST weight | Same MST weight |

```
    ┌─────────────────────────────────────────┐
    │  Both find the SAME MST weight.         │
    │  The actual edges may differ if there   │
    │  are ties in weight.                    │
    │                                         │
    │  ★ Default choice for interviews:       │
    │    Kruskal's (simpler to implement)     │
    │                                         │
    │  ★ Dense graph or adjacency list:       │
    │    Prim's (avoid sorting all edges)     │
    └─────────────────────────────────────────┘
```

---

[← Previous: Prim's Algorithm](03-prims-algorithm.md) | [Back to TOC](../README.md) | [Next: MST Applications →](05-mst-applications.md)
