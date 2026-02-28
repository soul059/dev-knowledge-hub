# Practical Tips

[← Previous: Path Reconstruction](05-path-reconstruction.md) | [Back to TOC](../README.md) | [Next Unit: Minimum Spanning Tree →](../10-Minimum-Spanning-Tree/01-what-is-a-spanning-tree.md)

---

## Interview Tips

```
    ┌──────────────────────────────────────────┐
    │         Shortest Path Cheat Sheet        │
    ├──────────────────────────────────────────┤
    │                                          │
    │ 1. Unweighted → BFS                     │
    │ 2. Non-negative weights → Dijkstra's     │
    │ 3. Negative weights → Bellman-Ford       │
    │ 4. All pairs → Floyd-Warshall            │
    │ 5. Weights 0/1 → 0-1 BFS (deque)        │
    │ 6. DAG → topological sort + relaxation   │
    │                                          │
    │ ★ Always ask: "Can weights be negative?" │
    │ ★ Always ask: "Single source or all?"    │
    └──────────────────────────────────────────┘
```

## Common Pitfalls

```
    1. Using Dijkstra's with negative weights → WRONG
    2. Forgetting lazy deletion in Dijkstra's → TLE
    3. Floyd-Warshall: k not outermost loop → WRONG
    4. Bellman-Ford: using V iterations instead of V-1
    5. Not handling unreachable vertices (dist = ∞)
    6. Integer overflow when adding to ∞
```

---

## Summary Table

| Algorithm | When to Use | Time | Handles Negative? |
|-----------|------------|------|-------------------|
| **BFS** | Unweighted | O(V+E) | N/A |
| **Dijkstra's** | Non-negative weights | O((V+E)log V) | No |
| **Bellman-Ford** | Negative weights | O(V×E) | Yes |
| **Floyd-Warshall** | All pairs | O(V³) | Yes |
| **0-1 BFS** | Weights 0 or 1 | O(V+E) | No |

## Quick Revision Questions

1. **Why can't Dijkstra's handle negative edge weights?**
   <details><summary>Answer</summary> Dijkstra's greedily finalizes the closest vertex, assuming its distance is optimal. With negative edges, a longer path through a negative edge might be shorter — but the vertex was already finalized and won't be updated.</details>

2. **Why does Bellman-Ford run exactly V-1 relaxation iterations?**
   <details><summary>Answer</summary> The shortest path between any two vertices has at most V-1 edges (any more would mean a repeated vertex = cycle). After i iterations, all shortest paths with ≤ i edges are correct. So V-1 iterations guarantee all shortest paths.</details>

3. **How does Floyd-Warshall detect negative cycles?**
   <details><summary>Answer</summary> After running the algorithm, check the diagonal of the distance matrix. If dist[i][i] < 0 for any i, a negative-weight cycle exists through vertex i.</details>

4. **What is the "lazy deletion" optimization in Dijkstra's?**
   <details><summary>Answer</summary> Instead of using decrease-key (expensive), we push new (distance, vertex) pairs to the heap. When popping, if the distance is greater than the current known best (d > dist[u]), we skip it — it's a stale entry.</details>

5. **For a DAG, what's the fastest shortest path algorithm?**
   <details><summary>Answer</summary> Topological sort + single-pass relaxation in topological order. Time: O(V + E). Works even with negative weights (no cycles in a DAG)!</details>

---

[← Previous: Path Reconstruction](05-path-reconstruction.md) | [Back to TOC](../README.md) | [Next Unit: Minimum Spanning Tree →](../10-Minimum-Spanning-Tree/01-what-is-a-spanning-tree.md)
