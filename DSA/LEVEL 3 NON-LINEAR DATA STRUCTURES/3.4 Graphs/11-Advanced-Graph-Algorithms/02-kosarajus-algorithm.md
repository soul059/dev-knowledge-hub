# Kosaraju's Algorithm

[← Previous: Strongly Connected Components](01-strongly-connected-components.md) | [Back to TOC](../README.md) | [Next: Tarjan's Algorithm →](03-tarjans-algorithm.md)

---

## Key Idea

```
    Two-pass DFS algorithm:
    
    Pass 1: DFS on ORIGINAL graph
            → Record vertices in FINISH ORDER (like topo-sort)
    
    Pass 2: DFS on REVERSED graph
            → Process vertices in REVERSE finish order
            → Each DFS in pass 2 finds one SCC
    
    Why does this work?
    Reversing edges doesn't change SCCs (mutual reachability
    still holds). But it prevents DFS from "leaking" into 
    other SCCs when we process in reverse finish order.
```

## Algorithm

```
FUNCTION kosaraju(V, graph):
    // Pass 1: Fill order by finish time
    visited = array of size V, all false
    stack = []
    
    FOR i = 0 to V-1:
        IF NOT visited[i]:
            dfs1(i, graph, visited, stack)
    
    // Build reversed graph
    revGraph = reverse all edges of graph
    
    // Pass 2: Process in reverse finish order
    visited = array of size V, all false    // reset
    sccs = []
    
    WHILE stack not empty:
        v = stack.pop()
        IF NOT visited[v]:
            component = []
            dfs2(v, revGraph, visited, component)
            sccs.append(component)
    
    RETURN sccs

FUNCTION dfs1(u, graph, visited, stack):
    visited[u] = true
    FOR each v in graph[u]:
        IF NOT visited[v]:
            dfs1(v, graph, visited, stack)
    stack.push(u)                          // post-order push!

FUNCTION dfs2(u, revGraph, visited, component):
    visited[u] = true
    component.append(u)
    FOR each v in revGraph[u]:
        IF NOT visited[v]:
            dfs2(v, revGraph, visited, component)
```

## Step-by-Step Trace

```
    Graph: 0→1, 1→2, 2→0, 1→3, 3→4
    
    ═══ PASS 1: DFS on original graph ═══
    
    Start DFS from 0:
      Visit 0 → Visit 1 → Visit 2 → 2's neighbor 0 visited
        Push 2 (stack: [2])
      1's next neighbor: Visit 3 → Visit 4 → no unvisited
        Push 4 (stack: [2,4])
        Push 3 (stack: [2,4,3])
        Push 1 (stack: [2,4,3,1])
        Push 0 (stack: [2,4,3,1,0])
    
    ═══ BUILD REVERSE GRAPH ═══
    
    Original: 0→1, 1→2, 2→0, 1→3, 3→4
    Reversed: 1→0, 2→1, 0→2, 3→1, 4→3
    
    ═══ PASS 2: DFS on reversed graph (pop order) ═══
    
    Pop 0: DFS on reversed → 0→2→1 → SCC: {0, 2, 1} ✓
    Pop 1: already visited → skip
    Pop 3: DFS on reversed → 3→(1 visited) → SCC: {3} ✓
    Pop 4: DFS on reversed → 4→(3 visited) → SCC: {4} ✓
    Pop 2: already visited → skip
    
    Result: 3 SCCs → {0,1,2}, {3}, {4}
```

## Complexity

```
    Time:  O(V + E)  — two DFS passes + graph reversal
    Space: O(V + E)  — reversed graph storage
```

---

[← Previous: Strongly Connected Components](01-strongly-connected-components.md) | [Back to TOC](../README.md) | [Next: Tarjan's Algorithm →](03-tarjans-algorithm.md)
