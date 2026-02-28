# Kahn's Algorithm (BFS-Based)

[← Previous: DFS-Based Topological Sort](02-dfs-based-topological-sort.md) | [Back to TOC](../README.md) | [Next: DFS vs Kahn's Comparison →](04-dfs-vs-kahns-comparison.md)

---

## Key Idea: Process Zero In-Degree Vertices

```
    A vertex with in-degree 0 has NO prerequisites.
    It can go first in the topological order!
    
    1. Find all vertices with in-degree 0 → enqueue them
    2. Process (dequeue) a vertex → add to result
    3. "Remove" it by decreasing neighbors' in-degrees
    4. If any neighbor reaches in-degree 0 → enqueue it
    5. Repeat until queue is empty
    
    ★ If result has fewer than V vertices → CYCLE EXISTS!
```

## Algorithm

```
FUNCTION topologicalSort_Kahn(graph, V):
    // Step 1: Calculate in-degrees
    inDegree = array of size V, all 0
    FOR each vertex u:
        FOR each neighbor v in graph[u]:
            inDegree[v] += 1
    
    // Step 2: Enqueue all vertices with in-degree 0
    queue = new Queue()
    FOR vertex = 0 to V-1:
        IF inDegree[vertex] == 0:
            queue.enqueue(vertex)
    
    // Step 3: Process
    result = []
    WHILE queue is not empty:
        vertex = queue.dequeue()
        result.append(vertex)
        
        FOR each neighbor in graph[vertex]:
            inDegree[neighbor] -= 1
            IF inDegree[neighbor] == 0:
                queue.enqueue(neighbor)
    
    // Step 4: Check for cycle
    IF result.size() != V:
        RETURN "Cycle detected! No topological order."
    
    RETURN result
```

## Step-by-Step Trace

```
    DAG:
       (5) ──→ (0) ←── (4)         In-degrees:
        │               │           0: 2    3: 1
        ▼               ▼           1: 2    4: 0
       (2) ──→ (3) ──→ (1)         2: 1    5: 0

    Initial queue (in-degree 0): [4, 5]
    
    Step 1: Dequeue 4, result = [4]
            Decrease: inDeg[0] = 1, inDeg[1] = 1
            No new zeros.
    
    Step 2: Dequeue 5, result = [4, 5]
            Decrease: inDeg[0] = 0 → enqueue 0
                      inDeg[2] = 0 → enqueue 2
            Queue: [0, 2]
    
    Step 3: Dequeue 0, result = [4, 5, 0]
            No outgoing edges.
    
    Step 4: Dequeue 2, result = [4, 5, 0, 2]
            Decrease: inDeg[3] = 0 → enqueue 3
            Queue: [3]
    
    Step 5: Dequeue 3, result = [4, 5, 0, 2, 3]
            Decrease: inDeg[1] = 0 → enqueue 1
            Queue: [1]
    
    Step 6: Dequeue 1, result = [4, 5, 0, 2, 3, 1]
            Queue empty. result.size() = 6 = V ✓
    
    Topological Order: 4, 5, 0, 2, 3, 1
```

## Cycle Detection with Kahn's

```
    If the graph has a cycle, some vertices will
    NEVER reach in-degree 0 (they depend on each
    other circularly).
    
    Result: result.size() < V → cycle exists!
    
    Example with cycle:
       A → B → C → A    (cycle)
    
    In-degrees: A:1, B:1, C:1
    No vertex has in-degree 0!
    Queue starts empty → result = [] → size 0 ≠ 3
    → Cycle detected!
```

---

[← Previous: DFS-Based Topological Sort](02-dfs-based-topological-sort.md) | [Back to TOC](../README.md) | [Next: DFS vs Kahn's Comparison →](04-dfs-vs-kahns-comparison.md)
