# Recursive DFS

[← Previous: DFS Intuition](01-dfs-intuition.md) | [Back to TOC](../README.md) | [Next: Iterative DFS →](03-iterative-dfs.md)

---

## Recursive DFS Pseudocode

```
FUNCTION DFS(graph, vertex, visited):
    visited.add(vertex)
    PROCESS(vertex)
    
    FOR each neighbor in graph[vertex]:
        IF neighbor NOT in visited:
            DFS(graph, neighbor, visited)    // ← Recursive call

// Usage:
visited = empty set
DFS(graph, source, visited)
```

## Step-by-Step Trace

```
    Graph:              Adjacency List:
       (A)────(B)       A → [B, C]
        │      │        B → [A, D, E]
        │      │        C → [A, F]
       (C)    (D)       D → [B]
        │      │        E → [B]
       (F)    (E)       F → [C]
```

```
    Call Stack Trace:                          Visited
    ─────────────────────────────────────────  ────────
    DFS(A)                                    {A}
     ├─ DFS(B)                                {A, B}
     │   ├─ neighbor A: already visited
     │   ├─ DFS(D)                            {A, B, D}
     │   │   └─ neighbor B: already visited
     │   │   └─ RETURN (backtrack to B)
     │   └─ DFS(E)                            {A, B, D, E}
     │       └─ neighbor B: already visited
     │       └─ RETURN (backtrack to B)
     │   └─ RETURN (backtrack to A)
     └─ DFS(C)                                {A, B, D, E, C}
         ├─ neighbor A: already visited
         └─ DFS(F)                            {A, B, D, E, C, F}
             └─ neighbor C: already visited
             └─ RETURN (backtrack to C)
         └─ RETURN (backtrack to A)
    DONE
    
    DFS Order: A → B → D → E → C → F
```

---

[← Previous: DFS Intuition](01-dfs-intuition.md) | [Back to TOC](../README.md) | [Next: Iterative DFS →](03-iterative-dfs.md)
