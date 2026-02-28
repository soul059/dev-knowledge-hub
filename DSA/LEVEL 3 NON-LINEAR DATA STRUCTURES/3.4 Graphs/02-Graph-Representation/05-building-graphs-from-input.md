# Building Graphs from Problem Input

[← Previous: Comparison of Representations](04-comparison-of-representations.md) | [Back to TOC](../README.md) | [Next: Grid as a Graph →](06-grid-as-a-graph.md)

---

## Common Input Formats

```
    FORMAT 1: Number of edges, then edge pairs
    ─────────────────────────────────────────
    Input:
    5 6          ← V=5 vertices, E=6 edges
    0 1          ← edge list
    0 2
    1 2
    1 3
    2 3
    2 4

    FORMAT 2: Adjacency for each vertex
    ─────────────────────────────────────────
    Input:
    5            ← V=5
    2 1 2        ← vertex 0 has 2 neighbors: 1 and 2
    3 0 2 3      ← vertex 1 has 3 neighbors: 0, 2, 3
    4 0 1 3 4    ← vertex 2 has 4 neighbors
    2 1 2        ← vertex 3
    1 2          ← vertex 4

    FORMAT 3: Grid (implicit graph)
    ─────────────────────────────────────────
    Input:
    1 1 0
    1 1 0       ← each cell is a vertex
    0 0 1       ← adjacent cells are neighbors
```

## Pseudocode: Reading Graph Input

```
// Format 1: Edge list input → Adjacency List
FUNCTION readGraph():
    READ V, E
    adjList = array of V empty lists
    
    FOR i = 1 to E:
        READ u, v
        adjList[u].append(v)
        adjList[v].append(u)   // if undirected
    
    RETURN adjList, V
```

---

[← Previous: Comparison of Representations](04-comparison-of-representations.md) | [Back to TOC](../README.md) | [Next: Grid as a Graph →](06-grid-as-a-graph.md)
