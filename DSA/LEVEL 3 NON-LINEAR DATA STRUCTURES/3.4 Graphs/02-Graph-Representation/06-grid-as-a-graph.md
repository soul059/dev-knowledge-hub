# Grid as a Graph

[← Previous: Building Graphs from Input](05-building-graphs-from-input.md) | [Back to TOC](../README.md) | [Next Unit: Breadth-First Search →](../03-Breadth-First-Search/01-bfs-intuition.md)

---

Many problems present a 2D grid which is actually an **implicit graph**:

```
    Grid:                          Graph (implicit):
    ┌───┬───┬───┐
    │ 0 │ 1 │ 2 │                 (0,0)──(0,1)──(0,2)
    ├───┼───┼───┤                   │       │       │
    │ 3 │ 4 │ 5 │                 (1,0)──(1,1)──(1,2)
    ├───┼───┼───┤                   │       │       │
    │ 6 │ 7 │ 8 │                 (2,0)──(2,1)──(2,2)
    └───┴───┴───┘

    Each cell = vertex
    Adjacent cells (up/down/left/right) = edges
    
    Neighbors of (r, c):
        (r-1, c)  ← up
        (r+1, c)  ← down
        (r, c-1)  ← left
        (r, c+1)  ← right
    
    Direction array trick:
        dr = [-1, 1, 0, 0]
        dc = [0, 0, -1, 1]
```

---

## Summary Table

| Representation | Space | Edge Check | Neighbors | Best For |
|---------------|-------|-----------|-----------|----------|
| **Adjacency Matrix** | O(V²) | O(1) | O(V) | Dense, Floyd-Warshall, O(1) check |
| **Adjacency List** | O(V+E) | O(deg) | O(deg) | Most problems, BFS, DFS |
| **Edge List** | O(E) | O(E) | O(E) | Kruskal's, edge-centric |
| **Grid (implicit)** | O(R×C) | O(1) | O(1)* | Matrix/grid problems |

*4 or 8 neighbors max per cell.

## Quick Revision Questions

1. **Which representation uses O(V²) space regardless of the number of edges?**
   <details><summary>Answer</summary> Adjacency Matrix</details>

2. **In an adjacency list for an undirected graph with E edges, how many total entries are stored across all lists?**
   <details><summary>Answer</summary> 2E (each edge stored twice, once for each endpoint)</details>

3. **Which representation is best for checking if edge (u, v) exists in O(1) time?**
   <details><summary>Answer</summary> Adjacency Matrix</details>

4. **For Kruskal's MST algorithm, which representation is most natural?**
   <details><summary>Answer</summary> Edge List (because we need to sort all edges by weight)</details>

5. **What are the 4-directional neighbors of cell (r, c) in a grid?**
   <details><summary>Answer</summary> (r-1,c), (r+1,c), (r,c-1), (r,c+1) — up, down, left, right</details>

6. **When should you prefer adjacency matrix over adjacency list?**
   <details><summary>Answer</summary> When the graph is dense (E ≈ V²), when you need O(1) edge existence checks, or when using Floyd-Warshall algorithm.</details>

---

[← Previous: Building Graphs from Input](05-building-graphs-from-input.md) | [Back to TOC](../README.md) | [Next Unit: Breadth-First Search →](../03-Breadth-First-Search/01-bfs-intuition.md)
