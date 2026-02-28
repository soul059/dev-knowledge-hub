# Connected Components

[Back to TOC](../README.md) | [Next: Number of Islands →](02-number-of-islands.md)

---

## Problem

Given an undirected graph with V vertices, find the **number of connected components**.

## Approach: DFS Loop

```
FUNCTION countComponents(V, graph):
    visited = set()
    count = 0
    
    FOR vertex = 0 to V-1:
        IF vertex NOT in visited:
            DFS(graph, vertex, visited)     // explore entire component
            count = count + 1               // found a new component!
    
    RETURN count
```

## Step-by-Step Trace

```
    Graph:
       (0)──(1)    (3)──(4)    (6)
        │              │
       (2)            (5)

    Adjacency List:
    0 → [1, 2]    3 → [4, 5]    6 → []
    1 → [0]       4 → [3]
    2 → [0]       5 → [3]

    Execution:
    ─────────────────────────────────────────────
    vertex 0: NOT visited → DFS(0)
              DFS visits: 0, 1, 2
              count = 1 ★ Component 1: {0, 1, 2}
    
    vertex 1: already visited → skip
    vertex 2: already visited → skip
    
    vertex 3: NOT visited → DFS(3)
              DFS visits: 3, 4, 5
              count = 2 ★ Component 2: {3, 4, 5}
    
    vertex 4: already visited → skip
    vertex 5: already visited → skip
    
    vertex 6: NOT visited → DFS(6)
              DFS visits: 6
              count = 3 ★ Component 3: {6}
    
    Answer: 3 connected components
```

## Complexity

```
    Time:  O(V + E)  — visit each vertex and edge once
    Space: O(V)      — visited set + recursion stack
```

---

[Back to TOC](../README.md) | [Next: Number of Islands →](02-number-of-islands.md)
