# Cycle in Undirected Graph (DFS)

[Back to TOC](../README.md) | [Next: Cycle Detection with Union-Find →](02-cycle-undirected-union-find.md)

---

## Problem

Detect if an undirected graph contains a cycle.

## Key Insight

```
    In undirected graph DFS, if we visit a neighbor
    that is ALREADY VISITED and it's NOT our parent,
    then we found a CYCLE!

    Why exclude parent?
    Edge A──B: when DFS goes A→B, B sees A as neighbor.
    A is visited, but A is B's parent → NOT a cycle.
    If B sees a visited vertex that ISN'T its parent → CYCLE!
```

## Algorithm

```
FUNCTION hasCycle(graph, V):
    visited = set()
    
    FOR vertex = 0 to V-1:
        IF vertex NOT in visited:
            IF DFS_Cycle(graph, vertex, -1, visited):
                RETURN true
    
    RETURN false

FUNCTION DFS_Cycle(graph, vertex, parent, visited):
    visited.add(vertex)
    
    FOR each neighbor in graph[vertex]:
        IF neighbor NOT in visited:
            IF DFS_Cycle(graph, neighbor, vertex, visited):
                RETURN true                    // cycle found deeper
        ELSE IF neighbor != parent:
            RETURN true                        // ★ CYCLE FOUND!
    
    RETURN false
```

## Step-by-Step Trace

```
    Graph with cycle:
       (0)──(1)
        │    │
       (2)──(3)
    
    DFS from 0 (parent = -1):
    ├─ Visit 0, visited = {0}
    ├─ Neighbor 1 (not visited):
    │   ├─ DFS(1, parent=0), visited = {0,1}
    │   ├─ Neighbor 0: visited BUT parent → skip
    │   ├─ Neighbor 3 (not visited):
    │   │   ├─ DFS(3, parent=1), visited = {0,1,3}
    │   │   ├─ Neighbor 1: visited BUT parent → skip
    │   │   ├─ Neighbor 2 (not visited):
    │   │   │   ├─ DFS(2, parent=3), visited = {0,1,3,2}
    │   │   │   ├─ Neighbor 0: visited AND NOT parent(3)
    │   │   │   └─ ★ RETURN TRUE — CYCLE FOUND!
    │   │   └─ RETURN TRUE
    │   └─ RETURN TRUE
    └─ RETURN TRUE
    
    Cycle: 0 → 1 → 3 → 2 → 0
```

---

[Back to TOC](../README.md) | [Next: Cycle Detection with Union-Find →](02-cycle-undirected-union-find.md)
