# Iterative DFS

[← Previous: Recursive DFS](02-recursive-dfs.md) | [Back to TOC](../README.md) | [Next: DFS Properties →](04-dfs-properties.md)

---

## Iterative DFS Pseudocode

```
FUNCTION DFS_Iterative(graph, source):
    visited = empty set
    stack = new Stack()
    stack.push(source)
    
    WHILE stack is not empty:
        vertex = stack.pop()
        
        IF vertex NOT in visited:          // ← Check AFTER popping
            visited.add(vertex)
            PROCESS(vertex)
            
            // Push neighbors (often in reverse order
            // to match recursive DFS order)
            FOR each neighbor in REVERSE(graph[vertex]):
                IF neighbor NOT in visited:
                    stack.push(neighbor)
    
    // All reachable vertices visited
```

## Iterative vs Recursive: Key Differences

```
    ┌─────────────────────┬──────────────────────┐
    │     RECURSIVE       │      ITERATIVE       │
    ├─────────────────────┼──────────────────────┤
    │ Uses call stack     │ Uses explicit stack   │
    │ Mark before calling │ Mark after popping    │
    │ Natural backtracking│ Manual stack mgmt     │
    │ Risk: stack overflow│ No stack overflow     │
    │ Cleaner code        │ More control          │
    └─────────────────────┴──────────────────────┘

    ⚠ NOTE: Iterative DFS may visit vertices in a
    slightly different order than recursive DFS,
    but both are valid DFS orderings.
    
    ⚠ Recursive DFS risks stack overflow for
    graphs with > 10,000 vertices (language-dependent).
    Use iterative DFS for large graphs.
```

## When to Use Each?

```
    ★ Default: Recursive DFS (cleaner, easier to code)
    ★ Large graphs: Iterative DFS (no stack overflow)
    ★ Need backtracking info: Recursive DFS
    ★ Need fine-grained control: Iterative DFS
```

---

[← Previous: Recursive DFS](02-recursive-dfs.md) | [Back to TOC](../README.md) | [Next: DFS Properties →](04-dfs-properties.md)
