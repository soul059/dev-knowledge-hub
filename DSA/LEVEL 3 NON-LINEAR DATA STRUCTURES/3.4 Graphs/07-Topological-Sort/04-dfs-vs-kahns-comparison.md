# DFS vs Kahn's Comparison

[← Previous: Kahn's Algorithm](03-kahns-algorithm.md) | [Back to TOC](../README.md) | [Next: Course Schedule Problems →](05-course-schedule-problems.md)

---

## Feature Comparison

| Feature | DFS-Based | Kahn's (BFS) |
|---------|-----------|---------------|
| **Approach** | Post-order + reverse | In-degree reduction |
| **Data Structure** | Recursion stack | Queue |
| **Cycle Detection** | Needs 3-color check | Built-in (result.size < V) |
| **Implementation** | Recursive, elegant | Iterative, intuitive |
| **Output** | Reversed postorder | Direct order |
| **All orderings** | Yes (with backtracking) | Yes (with backtracking) |
| **Time** | O(V + E) | O(V + E) |
| **Space** | O(V) | O(V) |
| **When to prefer** | Already using DFS | Need cycle detection |

```
    ┌─────────────────────────────────────┐
    │  Both produce valid topological     │
    │  orders, but may differ since       │
    │  multiple valid orders exist.       │
    │                                     │
    │  ★ Kahn's is often preferred in     │
    │    interviews because:              │
    │    - Easier to understand           │
    │    - Built-in cycle detection       │
    │    - Iterative (no stack overflow)  │
    └─────────────────────────────────────┘
```

---

[← Previous: Kahn's Algorithm](03-kahns-algorithm.md) | [Back to TOC](../README.md) | [Next: Course Schedule Problems →](05-course-schedule-problems.md)
