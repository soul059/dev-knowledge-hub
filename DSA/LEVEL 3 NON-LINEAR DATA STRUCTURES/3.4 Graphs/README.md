# 3.4 Graphs — Complete Study Guide

## Course Overview

Graphs are one of the most versatile and powerful data structures in computer science. They model **relationships between objects** — from social networks and maps to dependency chains and state machines. Mastering graphs unlocks the ability to solve a vast array of problems in algorithms, system design, and real-world engineering.

This guide covers graph theory from the ground up: fundamental concepts, classic algorithms (BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, Kruskal, Prim, Tarjan, Kosaraju), and the **problem-solving patterns** that appear repeatedly in interviews and competitive programming.

```
    GRAPHS — THE BIG PICTURE
    =========================

    Fundamentals ──► Representation ──► Traversal (BFS/DFS)
         │                                    │
         │                              ┌─────┴──────┐
         │                              ▼            ▼
         │                        Traversal      Cycle
         │                        Problems     Detection
         │                              │            │
         │                              ▼            ▼
         │                       Topological    Shortest
         │                          Sort         Path
         │                              │       ┌──┴──┐
         │                              ▼       ▼     ▼
         │                            MST   Unwtd  Wtd
         │                              │            │
         └──────────────────────────────┴─────┬──────┘
                                              ▼
                                      Advanced Algos
                                        & Patterns
```

---

## Complete Table of Contents

### Unit 1: Graph Fundamentals
- [01 — What is a Graph?](01-Graph-Fundamentals/01-what-is-a-graph.md)
- [02 — Graph Terminology](01-Graph-Fundamentals/02-graph-terminology.md)
- [03 — Types of Graphs](01-Graph-Fundamentals/03-types-of-graphs.md)
- [04 — Graph Properties](01-Graph-Fundamentals/04-graph-properties.md)
- [05 — Important Formulas](01-Graph-Fundamentals/05-important-formulas.md)
- [06 — Real-World Examples](01-Graph-Fundamentals/06-real-world-examples.md)

### Unit 2: Graph Representation
- [01 — Adjacency Matrix](02-Graph-Representation/01-adjacency-matrix.md)
- [02 — Adjacency List](02-Graph-Representation/02-adjacency-list.md)
- [03 — Edge List](02-Graph-Representation/03-edge-list.md)
- [04 — Comparison of Representations](02-Graph-Representation/04-comparison-of-representations.md)
- [05 — Building Graphs from Input](02-Graph-Representation/05-building-graphs-from-input.md)
- [06 — Grid as a Graph](02-Graph-Representation/06-grid-as-a-graph.md)

### Unit 3: Breadth-First Search (BFS)
- [01 — BFS Intuition](03-Breadth-First-Search/01-bfs-intuition.md)
- [02 — BFS Pseudocode](03-Breadth-First-Search/02-bfs-pseudocode.md)
- [03 — BFS Step-by-Step Trace](03-Breadth-First-Search/03-bfs-step-by-step-trace.md)
- [04 — Level-Order BFS](03-Breadth-First-Search/04-level-order-bfs.md)
- [05 — BFS Properties](03-Breadth-First-Search/05-bfs-properties.md)
- [06 — BFS on a Grid](03-Breadth-First-Search/06-bfs-on-a-grid.md)
- [07 — Applications and Patterns](03-Breadth-First-Search/07-applications-and-patterns.md)

### Unit 4: Depth-First Search (DFS)
- [01 — DFS Intuition](04-Depth-First-Search/01-dfs-intuition.md)
- [02 — Recursive DFS](04-Depth-First-Search/02-recursive-dfs.md)
- [03 — Iterative DFS](04-Depth-First-Search/03-iterative-dfs.md)
- [04 — DFS Properties](04-Depth-First-Search/04-dfs-properties.md)
- [05 — DFS Tree and Backtracking](04-Depth-First-Search/05-dfs-tree-and-backtracking.md)
- [06 — DFS on a Grid](04-Depth-First-Search/06-dfs-on-a-grid.md)
- [07 — Applications and Patterns](04-Depth-First-Search/07-applications-and-patterns.md)

### Unit 5: Graph Traversal Problems
- [01 — Connected Components](05-Graph-Traversal-Problems/01-connected-components.md)
- [02 — Number of Islands](05-Graph-Traversal-Problems/02-number-of-islands.md)
- [03 — Flood Fill](05-Graph-Traversal-Problems/03-flood-fill.md)
- [04 — Surrounded Regions](05-Graph-Traversal-Problems/04-surrounded-regions.md)
- [05 — Clone Graph](05-Graph-Traversal-Problems/05-clone-graph.md)
- [06 — Bipartite Check](05-Graph-Traversal-Problems/06-bipartite-check.md)

### Unit 6: Cycle Detection
- [01 — Cycle in Undirected Graph (DFS)](06-Cycle-Detection/01-cycle-undirected-dfs.md)
- [02 — Cycle in Undirected Graph (Union-Find)](06-Cycle-Detection/02-cycle-undirected-union-find.md)
- [03 — Cycle in Directed Graph (DFS Colors)](06-Cycle-Detection/03-cycle-directed-dfs-colors.md)
- [04 — Finding the Actual Cycle](06-Cycle-Detection/04-finding-the-actual-cycle.md)
- [05 — Applications and Comparison](06-Cycle-Detection/05-applications-and-comparison.md)

### Unit 7: Topological Sort
- [01 — What is Topological Sort?](07-Topological-Sort/01-what-is-topological-sort.md)
- [02 — DFS-Based Topological Sort](07-Topological-Sort/02-dfs-based-topological-sort.md)
- [03 — Kahn's Algorithm](07-Topological-Sort/03-kahns-algorithm.md)
- [04 — DFS vs Kahn's Comparison](07-Topological-Sort/04-dfs-vs-kahns-comparison.md)
- [05 — Course Schedule Problems](07-Topological-Sort/05-course-schedule-problems.md)
- [06 — Build Order and Advanced](07-Topological-Sort/06-build-order-and-advanced.md)

### Unit 8: Shortest Path — Unweighted
- [01 — BFS for Shortest Path](08-Shortest-Path-Unweighted/01-bfs-for-shortest-path.md)
- [02 — Single Source Shortest Path](08-Shortest-Path-Unweighted/02-single-source-shortest-path.md)
- [03 — Word Ladder](08-Shortest-Path-Unweighted/03-word-ladder.md)
- [04 — Minimum Moves Problems](08-Shortest-Path-Unweighted/04-minimum-moves-problems.md)
- [05 — Multi-Source BFS](08-Shortest-Path-Unweighted/05-multi-source-bfs.md)
- [06 — Zero-One BFS](08-Shortest-Path-Unweighted/06-zero-one-bfs.md)
- [07 — Bidirectional BFS](08-Shortest-Path-Unweighted/07-bidirectional-bfs.md)

### Unit 9: Shortest Path — Weighted
- [01 — Dijkstra's Algorithm](09-Shortest-Path-Weighted/01-dijkstras-algorithm.md)
- [02 — Bellman-Ford Algorithm](09-Shortest-Path-Weighted/02-bellman-ford-algorithm.md)
- [03 — Floyd-Warshall Algorithm](09-Shortest-Path-Weighted/03-floyd-warshall-algorithm.md)
- [04 — Algorithm Comparison](09-Shortest-Path-Weighted/04-algorithm-comparison.md)
- [05 — Path Reconstruction](09-Shortest-Path-Weighted/05-path-reconstruction.md)
- [06 — Practical Tips](09-Shortest-Path-Weighted/06-practical-tips.md)

### Unit 10: Minimum Spanning Tree
- [01 — What is a Spanning Tree?](10-Minimum-Spanning-Tree/01-what-is-a-spanning-tree.md)
- [02 — Kruskal's Algorithm](10-Minimum-Spanning-Tree/02-kruskals-algorithm.md)
- [03 — Prim's Algorithm](10-Minimum-Spanning-Tree/03-prims-algorithm.md)
- [04 — Kruskal's vs Prim's](10-Minimum-Spanning-Tree/04-kruskals-vs-prims.md)
- [05 — MST Applications](10-Minimum-Spanning-Tree/05-mst-applications.md)
- [06 — Common Problem Patterns](10-Minimum-Spanning-Tree/06-common-problem-patterns.md)

### Unit 11: Advanced Graph Algorithms
- [01 — Strongly Connected Components](11-Advanced-Graph-Algorithms/01-strongly-connected-components.md)
- [02 — Kosaraju's Algorithm](11-Advanced-Graph-Algorithms/02-kosarajus-algorithm.md)
- [03 — Tarjan's Algorithm](11-Advanced-Graph-Algorithms/03-tarjans-algorithm.md)
- [04 — Articulation Points](11-Advanced-Graph-Algorithms/04-articulation-points.md)
- [05 — Bridges](11-Advanced-Graph-Algorithms/05-bridges.md)
- [06 — Eulerian Path and Circuit](11-Advanced-Graph-Algorithms/06-eulerian-path-and-circuit.md)
- [07 — Condensation Graph](11-Advanced-Graph-Algorithms/07-condensation-graph.md)

### Unit 12: Graph Problem Patterns
- [01 — Grid Problems](12-Graph-Problem-Patterns/01-grid-problems.md)
- [02 — Implicit Graphs](12-Graph-Problem-Patterns/02-implicit-graphs.md)
- [03 — Multi-State BFS](12-Graph-Problem-Patterns/03-multi-state-bfs.md)
- [04 — Graph Coloring](12-Graph-Problem-Patterns/04-graph-coloring.md)
- [05 — Union-Find Patterns](12-Graph-Problem-Patterns/05-union-find-patterns.md)
- [06 — Topological Sort Patterns](12-Graph-Problem-Patterns/06-topological-sort-patterns.md)
- [07 — Decision Framework](12-Graph-Problem-Patterns/07-decision-framework.md)

---

## How to Use This Guide

| Step | Action |
|------|--------|
| 1 | Read each unit **in order** — later units build on earlier ones |
| 2 | **Trace every algorithm by hand** on the example graphs |
| 3 | Implement each algorithm yourself before looking at solutions |
| 4 | Answer the **Quick Revision Questions** at the end of each chapter |
| 5 | Solve the suggested practice problems |
| 6 | Revisit the **Summary Tables** for quick review |

---

## Key Prerequisites

- Arrays, linked lists, stacks, queues
- Recursion and backtracking basics
- Big-O notation (time and space)
- Basic tree concepts (trees are special graphs!)

---

## Quick Reference — Algorithm Complexities

| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| BFS | O(V + E) | O(V) | Shortest path (unweighted), level-order |
| DFS | O(V + E) | O(V) | Connectivity, cycle detection, topo sort |
| Dijkstra | O((V+E) log V) | O(V) | Shortest path (non-negative weights) |
| Bellman-Ford | O(V·E) | O(V) | Shortest path (negative weights OK) |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs shortest path |
| Kruskal | O(E log E) | O(V) | MST (sparse graphs) |
| Prim | O((V+E) log V) | O(V) | MST (dense graphs) |
| Topological Sort | O(V + E) | O(V) | DAG ordering |
| Tarjan (SCC) | O(V + E) | O(V) | Strongly connected components |
| Kosaraju (SCC) | O(V + E) | O(V) | Strongly connected components |

---

> **Notation:** Throughout this guide, **V** = number of vertices, **E** = number of edges.

---

[**Start Learning → Unit 1: What is a Graph?**](01-Graph-Fundamentals/01-what-is-a-graph.md)
