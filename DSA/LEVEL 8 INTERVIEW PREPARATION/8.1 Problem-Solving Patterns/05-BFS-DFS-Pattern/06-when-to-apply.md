# Chapter 6: When to Apply BFS/DFS

## 📋 Chapter Overview
Decision guide for choosing between BFS and DFS, and recognizing graph/tree traversal problems.

---

## 🗺️ BFS vs DFS Decision

```
  Need SHORTEST path (unweighted)?
  │
  ├── YES → BFS
  │
  └── NO
      │
      ├── Need LEVEL-by-level processing? → BFS
      │
      ├── Need to explore ALL paths / backtrack? → DFS
      │
      ├── Tree problem with top-down constraints? → DFS
      │
      ├── Need to detect CYCLES in directed graph? → DFS (3-color)
      │
      ├── Topological sort? → Either (BFS=Kahn's, DFS=reverse post)
      │
      └── Memory constrained on wide graph? → DFS (less memory)
```

---

## ✅ When BFS

| Signal | Example |
|--------|---------|
| "Shortest path" in unweighted graph | Word Ladder, Shortest Path in Grid |
| "Minimum number of steps/moves" | Min Knight Moves, Open Lock |
| Level-order / layer-by-layer | Level Order Traversal, Zigzag |
| "Nearest" / "closest" | Nearest 0 in Matrix |
| Multi-source simultaneous spread | Rotting Oranges, Walls and Gates |

## ✅ When DFS

| Signal | Example |
|--------|---------|
| "All paths" / "any path" | All Paths from Source to Target |
| Connectivity / components | Number of Islands |
| Cycle detection | Course Schedule (directed) |
| Backtracking needed | Sudoku Solver, N-Queens |
| Tree recursion (path sum, LCA) | Path Sum, Lowest Common Ancestor |
| Topological order | Course Schedule II |

---

## ❌ When Neither

| Scenario | Use Instead |
|----------|-------------|
| Weighted shortest path | Dijkstra's / Bellman-Ford |
| Minimum spanning tree | Kruskal's / Prim's |
| Dynamic connectivity | Union-Find |
| Shortest path with negative weights | Bellman-Ford |

---

## 📊 Complexity Comparison

| Aspect | BFS | DFS |
|--------|-----|-----|
| Time | O(V + E) | O(V + E) |
| Space (graph) | O(V) queue | O(V) stack |
| Space (tree) | O(width) | O(height) |
| Balanced tree | O(n) wide | O(log n) deep |
| Skewed tree | O(1) wide | O(n) deep |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Shortest path | BFS (unweighted only) |
| Level processing | BFS with levelSize snapshot |
| All paths / backtrack | DFS |
| Cycle detection | DFS (3-color for directed) |
| Tree problems | DFS for most, BFS for level-order |
| Both O(V+E) | Same time, different space profiles |

---

## ❓ Revision Questions

1. **"Find minimum moves" → BFS or DFS?** → BFS (shortest path).
2. **"Find all connected components" → BFS or DFS?** → Either works; DFS is simpler.
3. **Space usage on a balanced binary tree?** → BFS: O(n/2) ≈ O(n); DFS: O(log n). DFS wins.
4. **Space usage on a complete graph?** → Both O(V). BFS may queue many neighbors at once.
5. **Weighted shortest path → BFS?** → No. Use Dijkstra's algorithm.

---

[← Previous: Classic Problems](05-classic-problems.md)

[← Back to README](../README.md) | [Next Unit: Backtracking Pattern →](../06-Backtracking-Pattern/01-backtracking-fundamentals.md)
