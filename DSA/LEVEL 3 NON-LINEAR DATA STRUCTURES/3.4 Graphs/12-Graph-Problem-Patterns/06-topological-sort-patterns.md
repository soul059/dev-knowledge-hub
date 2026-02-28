# Topological Sort Patterns

[← Previous: Union-Find Patterns](05-union-find-patterns.md) | [Back to TOC](../README.md) | [Next: Decision Framework →](07-decision-framework.md)

---

## When to Use Topological Sort

```
    ┌─────────────────────────────────────────────────┐
    │  Use Topo-Sort when:                            │
    │                                                  │
    │  1. "Order tasks respecting dependencies"       │
    │  2. "Can all courses be finished?"              │
    │  3. "Find a valid build order"                  │
    │  4. Longest path in a DAG                       │
    │  5. Shortest path in a DAG                      │
    │  6. Any problem with "prerequisites"            │
    │                                                  │
    │  Requirement: DIRECTED ACYCLIC GRAPH (DAG)      │
    └─────────────────────────────────────────────────┘
```

## Pattern Table

| Problem Pattern | Approach |
|----------------|----------|
| "Can we finish all tasks?" | Kahn's — check if result has V nodes |
| "Find one valid order" | DFS topo-sort or Kahn's |
| "Find ALL valid orders" | Backtracking with Kahn's |
| "Longest path in DAG" | Topo-sort + DP relaxation |
| "Shortest path in DAG" | Topo-sort + relaxation |
| "Detect cycle in dependencies" | Kahn's — incomplete result = cycle |
| "Parallel task scheduling" | Kahn's with level tracking |

## Longest Path in a DAG

```
    Unlike general graphs (NP-hard), longest path in a
    DAG can be found in O(V + E) using topological sort!
    
    FUNCTION longestPath(V, graph):
        order = topologicalSort(graph)
        dist = array of size V, all 0
        
        FOR each u in order:
            FOR each (v, weight) in graph[u]:
                dist[v] = max(dist[v], dist[u] + weight)
        
        RETURN max(dist)
    
    Application: Critical path in project scheduling
```

## Parallel Task Scheduling

```
    "What's the minimum time to complete all tasks
    if we can run independent tasks in parallel?"
    
    → This is the LONGEST PATH in the dependency DAG!
    
    OR: Use Kahn's BFS with level tracking.
    Each "level" = tasks that can run in parallel.
    Answer = number of levels.
    
    FUNCTION minTime(V, graph):
        // Kahn's with level counting
        inDeg = compute in-degrees
        queue = all vertices with inDeg 0
        levels = 0
        
        WHILE queue not empty:
            levels++
            nextQueue = []
            FOR each u in queue:
                FOR each v in graph[u]:
                    inDeg[v]--
                    IF inDeg[v] == 0:
                        nextQueue.append(v)
            queue = nextQueue
        
        RETURN levels
```

---

[← Previous: Union-Find Patterns](05-union-find-patterns.md) | [Back to TOC](../README.md) | [Next: Decision Framework →](07-decision-framework.md)
