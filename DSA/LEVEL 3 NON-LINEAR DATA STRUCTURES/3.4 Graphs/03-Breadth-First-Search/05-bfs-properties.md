# BFS Properties

[← Previous: Level-Order BFS](04-level-order-bfs.md) | [Back to TOC](../README.md) | [Next: BFS on a Grid →](06-bfs-on-a-grid.md)

---

## Property 1: Shortest Path Guarantee

```
    In an UNWEIGHTED graph, BFS finds the
    SHORTEST PATH (minimum number of edges)
    from the source to every reachable vertex.
    
    Why? Because BFS explores:
    - All vertices at distance 1 first
    - Then all at distance 2
    - Then all at distance 3
    - ...
    
    A vertex is discovered at the EARLIEST level
    possible = shortest distance!
```

## Property 2: BFS Tree

```
    The edges used to DISCOVER new vertices
    form a tree called the BFS Tree.
    
    Starting graph:              BFS Tree from A:
       (A)────(B)                    (A)
        │    ╱ │                    ╱   ╲
        │   ╱  │                  (B)   (C)
       (C)    (D)                  │     │
        │                         (D)   (E)
       (E)

    (A-C edge and B-C edge are "non-tree" edges)
```

## Property 3: Connected Component Exploration

```
    BFS from source visits ALL vertices in the
    same connected component.
    
    Vertices NOT visited = different component.
    
    To visit ALL vertices (even disconnected):
    
    FOR each vertex v in graph:
        IF v NOT visited:
            BFS(graph, v)         // new component found
            componentCount += 1
```

---

[← Previous: Level-Order BFS](04-level-order-bfs.md) | [Back to TOC](../README.md) | [Next: BFS on a Grid →](06-bfs-on-a-grid.md)
