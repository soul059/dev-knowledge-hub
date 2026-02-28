# Decision Framework and Final Summary

[← Previous: Topological Sort Patterns](06-topological-sort-patterns.md) | [Back to TOC](../README.md)

---

## 3-Step Algorithm Selection Framework

```
    ┌──────────────────────────────────────────────┐
    │        STEP 1: Identify Graph Type            │
    ├──────────────────────────────────────────────┤
    │  Directed or Undirected?                      │
    │  Weighted or Unweighted?                      │
    │  Explicit (adj list) or Implicit (grid/state)?│
    │  Can have negative edges?                     │
    │  Is it a DAG?                                 │
    └──────────────────┬───────────────────────────┘
                       ↓
    ┌──────────────────────────────────────────────┐
    │        STEP 2: Identify the Question          │
    ├──────────────────────────────────────────────┤
    │  Shortest path?        → Step 3a             │
    │  Connectivity?         → Step 3b             │
    │  Cycle detection?      → Step 3c             │
    │  Ordering/dependencies?→ Step 3d             │
    │  Minimum cost network? → Step 3e             │
    └──────────────────┬───────────────────────────┘
                       ↓
    ┌──────────────────────────────────────────────┐
    │        STEP 3: Pick Algorithm                 │
    ├──────────────────────────────────────────────┤
    │                                              │
    │  3a. SHORTEST PATH:                          │
    │  ├── Unweighted → BFS                        │
    │  ├── Weighted, no negative → Dijkstra's      │
    │  ├── Negative edges → Bellman-Ford            │
    │  ├── All pairs → Floyd-Warshall              │
    │  ├── DAG → Topo-sort + relaxation            │
    │  └── 0/1 weights → 0-1 BFS (deque)           │
    │                                              │
    │  3b. CONNECTIVITY:                           │
    │  ├── Connected components → DFS/BFS/UF       │
    │  ├── Dynamic connectivity → Union-Find       │
    │  ├── Strongly connected → Tarjan/Kosaraju    │
    │  └── Cut points/bridges → Tarjan's           │
    │                                              │
    │  3c. CYCLE DETECTION:                        │
    │  ├── Undirected → DFS (parent) or UF         │
    │  └── Directed → DFS (3 colors)               │
    │                                              │
    │  3d. ORDERING:                               │
    │  ├── Topo sort → DFS or Kahn's BFS           │
    │  └── All orderings → Backtracking            │
    │                                              │
    │  3e. MIN-COST NETWORK:                       │
    │  ├── Sparse → Kruskal's                      │
    │  └── Dense → Prim's                          │
    └──────────────────────────────────────────────┘
```

## Common Mistakes Table

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| BFS on weighted graph | BFS doesn't respect weights | Use Dijkstra's |
| Dijkstra with negative edges | Greedy fails with negatives | Use Bellman-Ford |
| DFS for shortest path | DFS doesn't guarantee shortest | Use BFS |
| Forgetting visited check | Infinite loop on cycles | Always mark visited |
| Marking visited when dequeuing | Duplicate processing | Mark when enqueuing |
| Topo-sort on cyclic graph | No valid ordering exists | Check for cycle first |
| Union-Find for directed graphs | UF doesn't track direction | Use DFS with colors |
| Floyd-Warshall for single source | O(V³) overkill | Use Dijkstra O((V+E)logV) |

## Master Complexity Table

| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| **BFS** | O(V+E) | O(V) | Shortest path (unweighted), level-order |
| **DFS** | O(V+E) | O(V) | Connectivity, cycle detection, topo-sort |
| **Dijkstra's** | O((V+E)logV) | O(V) | Shortest path (non-negative weights) |
| **Bellman-Ford** | O(V×E) | O(V) | Shortest path (negative weights) |
| **Floyd-Warshall** | O(V³) | O(V²) | All-pairs shortest path |
| **Kruskal's** | O(ElogE) | O(V+E) | MST (sparse graphs) |
| **Prim's** | O((V+E)logV) | O(V) | MST (dense graphs) |
| **Kahn's (BFS)** | O(V+E) | O(V) | Topological sort + cycle detection |
| **Tarjan (SCC)** | O(V+E) | O(V) | Strongly connected components |
| **Kosaraju** | O(V+E) | O(V+E) | Strongly connected components |
| **Union-Find** | O(α(n)) per op | O(V) | Dynamic connectivity, Kruskal's |
| **0-1 BFS** | O(V+E) | O(V) | 0/1 weighted shortest path |
| **Multi-source BFS** | O(V+E) | O(V) | Simultaneous spread from multiple sources |

---

## Summary Table

| Unit | Core Concept |
|------|-------------|
| **1. Fundamentals** | Graph = vertices + edges; directed/undirected, weighted/unweighted |
| **2. Representation** | Adj matrix O(V²), adj list O(V+E), edge list O(E) |
| **3. BFS** | Queue, level-by-level, shortest path in unweighted |
| **4. DFS** | Stack/recursion, backtracking, edge classification |
| **5. Traversal Problems** | Islands, flood fill, surrounded regions, bipartite |
| **6. Cycle Detection** | Undirected: parent check / UF. Directed: 3 colors |
| **7. Topo Sort** | DFS reverse postorder or Kahn's BFS. DAGs only |
| **8. Shortest Path (Unweighted)** | BFS, word ladder, multi-source, 0-1 BFS |
| **9. Shortest Path (Weighted)** | Dijkstra, Bellman-Ford, Floyd-Warshall |
| **10. MST** | Kruskal's (sort+UF) vs Prim's (grow+heap) |
| **11. Advanced** | SCC (Kosaraju/Tarjan), APs, bridges, Euler paths |
| **12. Patterns** | Grid, implicit graphs, multi-state, coloring, UF, topo patterns |

## Quick Revision Questions

1. **When should you use BFS vs DFS?**
   <details><summary>Answer</summary> BFS: shortest path in unweighted graphs, level-order processing, finding nearest target. DFS: cycle detection, topological sorting, connected components, path existence, backtracking problems. BFS uses a queue (O(V) space for width); DFS uses recursion/stack (O(V) space for depth).</details>

2. **How do you choose between Dijkstra's and Bellman-Ford?**
   <details><summary>Answer</summary> Dijkstra's: no negative edges, faster O((V+E)logV). Bellman-Ford: handles negative edges, detects negative cycles, O(V×E). Default to Dijkstra unless negative edges exist.</details>

3. **What's the key difference between Union-Find and DFS for connectivity?**
   <details><summary>Answer</summary> Union-Find excels at DYNAMIC connectivity (edges added over time, each query near O(1)). DFS/BFS is better for static graphs and when you need to traverse/process all reachable nodes. Union-Find cannot delete edges or find paths.</details>

4. **How do you recognize an implicit graph problem?**
   <details><summary>Answer</summary> The problem talks about "states" and "moves/transitions" rather than explicit vertices and edges. Examples: puzzles, lock combinations, word transformations. Model each configuration as a vertex, each valid move as an edge, then use BFS for shortest path.</details>

5. **What makes multi-state BFS different from regular BFS?**
   <details><summary>Answer</summary> The state includes extra dimensions beyond just position (e.g., keys collected as bitmask, obstacles eliminated). The visited array becomes multi-dimensional: visited[row][col][extra]. This increases space but handles problems where the same position may be reached in meaningfully different states.</details>

6. **When does topological sort fail, and how do you detect it?**
   <details><summary>Answer</summary> Topo-sort fails when the graph has a cycle (not a DAG). Detection: with Kahn's BFS, if the result has fewer than V vertices, a cycle exists. With DFS, if a back edge is found (visiting a GRAY/in-progress node), there's a cycle.</details>

---

**Congratulations!** You've completed the comprehensive Graph theory study guide covering all 12 units — from fundamentals to advanced algorithms and problem-solving patterns.

---

[← Previous: Topological Sort Patterns](06-topological-sort-patterns.md) | [Back to TOC](../README.md)
