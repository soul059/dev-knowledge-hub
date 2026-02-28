# DFS-Based Topological Sort

[← Previous: What is Topological Sort?](01-what-is-topological-sort.md) | [Back to TOC](../README.md) | [Next: Kahn's Algorithm →](03-kahns-algorithm.md)

---

## Key Idea: Reverse Postorder

```
    In DFS, a vertex is "finished" AFTER all its
    descendants are finished.
    
    If we add vertices to a list when they FINISH
    (post-order), and then REVERSE the list,
    we get a valid topological order!
    
    Why? If u→v exists, v finishes BEFORE u.
    In reversed list, u appears before v. ✓
```

## Algorithm

```
FUNCTION topologicalSort_DFS(graph, V):
    visited = set()
    stack = []            // will hold the result in reverse
    
    FOR vertex = 0 to V-1:
        IF vertex NOT in visited:
            DFS(graph, vertex, visited, stack)
    
    RETURN stack.reversed()    // reverse = topological order

FUNCTION DFS(graph, vertex, visited, stack):
    visited.add(vertex)
    
    FOR each neighbor in graph[vertex]:
        IF neighbor NOT in visited:
            DFS(graph, neighbor, visited, stack)
    
    stack.push(vertex)         // ← add AFTER all descendants done
```

## Step-by-Step Trace

```
    DAG:
       (5) ──→ (0) ←── (4)
        │               │
        ▼               ▼
       (2) ──→ (3) ──→ (1)
    
    Adjacency List:
    0 → []        3 → [1]
    1 → []        4 → [0, 1]
    2 → [3]       5 → [0, 2]
    
    DFS traversal (starting from vertex 0, 1, 2, ...):
    
    vertex 0: DFS(0) → no neighbors → push 0
              stack = [0]
    
    vertex 1: DFS(1) → no neighbors → push 1
              stack = [0, 1]
    
    vertex 2: DFS(2) → DFS(3) → DFS(1, visited) → push 3 → push 2
              stack = [0, 1, 3, 2]
    
    vertex 3: already visited
    
    vertex 4: DFS(4) → DFS(0, visited) → DFS(1, visited) → push 4
              stack = [0, 1, 3, 2, 4]
    
    vertex 5: DFS(5) → DFS(0, visited) → DFS(2, visited) → push 5
              stack = [0, 1, 3, 2, 4, 5]
    
    Reverse: [5, 4, 2, 3, 1, 0]
    
    Verify: 5→0 ✓, 5→2 ✓, 4→0 ✓, 4→1 ✓, 2→3 ✓, 3→1 ✓
```

---

[← Previous: What is Topological Sort?](01-what-is-topological-sort.md) | [Back to TOC](../README.md) | [Next: Kahn's Algorithm →](03-kahns-algorithm.md)
