# Graph Coloring

[← Previous: Multi-State BFS](03-multi-state-bfs.md) | [Back to TOC](../README.md) | [Next: Union-Find Patterns →](05-union-find-patterns.md)

---

## Bipartite Check (2-Coloring)

```
    Can we color all vertices with 2 colors such that
    no two adjacent vertices share the same color?
    
    YES → Graph is BIPARTITE
    NO  → Graph has an ODD-length cycle
    
    Algorithm: BFS/DFS coloring
    
    ┌────────────────────────────────────────┐
    │  Start: color source with 0           │
    │  BFS:   color all neighbors with 1    │
    │  Next:  color their neighbors with 0  │
    │  If conflict → NOT bipartite          │
    └────────────────────────────────────────┘
```

## Pseudocode: BFS 2-Coloring

```
FUNCTION isBipartite(V, graph):
    color = array of size V, all -1
    
    FOR i = 0 to V-1:                       // handle disconnected
        IF color[i] != -1: CONTINUE
        
        queue = [i]
        color[i] = 0
        
        WHILE queue not empty:
            u = queue.dequeue()
            FOR each v in graph[u]:
                IF color[v] == -1:
                    color[v] = 1 - color[u]  // flip color
                    queue.enqueue(v)
                ELSE IF color[v] == color[u]:
                    RETURN false             // conflict!
    
    RETURN true
```

## M-Coloring (Backtracking)

```
    Can we color the graph with M colors such that no 
    two adjacent vertices share the same color?
    
    This is the "Graph Coloring Problem" — NP-Complete 
    for M ≥ 3!
    
    Solved with BACKTRACKING:
    
    FUNCTION canColor(graph, V, M):
        colors = array of size V, all 0
        RETURN solve(0, graph, V, M, colors)
    
    FUNCTION solve(vertex, graph, V, M, colors):
        IF vertex == V: RETURN true          // all colored
        
        FOR c = 1 to M:
            IF isSafe(vertex, c, graph, colors):
                colors[vertex] = c
                IF solve(vertex + 1, ...): RETURN true
                colors[vertex] = 0           // backtrack
        
        RETURN false                         // no color works
    
    FUNCTION isSafe(v, c, graph, colors):
        FOR each neighbor u of v:
            IF colors[u] == c: RETURN false
        RETURN true
```

## Key Distinctions

```
    ┌───────────────────┬──────────────────────────────┐
    │  2-Coloring       │  M-Coloring (M ≥ 3)         │
    ├───────────────────┼──────────────────────────────┤
    │  Polynomial O(V+E)│  NP-Complete                 │
    │  BFS/DFS          │  Backtracking                │
    │  = Bipartite check│  = Graph Coloring Problem    │
    │  Practical        │  Exponential worst case      │
    └───────────────────┴──────────────────────────────┘
    
    Four Color Theorem: Any planar graph can be colored
    with at most 4 colors. (But FINDING the coloring 
    is still hard!)
```

---

[← Previous: Multi-State BFS](03-multi-state-bfs.md) | [Back to TOC](../README.md) | [Next: Union-Find Patterns →](05-union-find-patterns.md)
