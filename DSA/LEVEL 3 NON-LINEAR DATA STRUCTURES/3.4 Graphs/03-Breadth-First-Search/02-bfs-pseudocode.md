# BFS Pseudocode

[← Previous: BFS Intuition](01-bfs-intuition.md) | [Back to TOC](../README.md) | [Next: Step-by-Step BFS Trace →](03-bfs-step-by-step-trace.md)

---

## Standard BFS Algorithm

```
FUNCTION BFS(graph, source):
    visited = set containing {source}      // ← Mark BEFORE enqueuing
    queue = new Queue()
    queue.enqueue(source)
    
    WHILE queue is not empty:
        vertex = queue.dequeue()
        PROCESS(vertex)                    // ← Do something with vertex
        
        FOR each neighbor in graph[vertex]:
            IF neighbor NOT in visited:
                visited.add(neighbor)      // ← Mark visited HERE
                queue.enqueue(neighbor)    //   NOT when dequeuing!
    
    // All reachable vertices have been visited
```

## Why Mark Visited When ENQUEUING (Not Dequeuing)?

```
    If we mark when dequeuing, the same vertex can be 
    added to the queue MULTIPLE times:

    WRONG:                          CORRECT:
    ┌─────────────────┐            ┌─────────────────┐
    │ Enqueue A       │            │ Enqueue A       │
    │ Dequeue A       │            │ Mark A visited  │
    │ Mark A visited  │            │ Dequeue A       │
    │                 │            │                 │
    │ Enqueue B       │            │ Enqueue B       │
    │ Enqueue C       │            │ Mark B visited  │
    │ Dequeue B       │            │ Enqueue C       │
    │ Mark B visited  │            │ Mark C visited  │
    │ Enqueue D       │            │ Dequeue B       │
    │ Enqueue C ← DUP!│           │ Enqueue D       │
    └─────────────────┘            │ Mark D visited  │
                                   │ Dequeue C       │
    C gets enqueued twice          │ D already visited│
    because it wasn't              └─────────────────┘
    marked when first              
    enqueued.                      No duplicates! ✓
```

---

[← Previous: BFS Intuition](01-bfs-intuition.md) | [Back to TOC](../README.md) | [Next: Step-by-Step BFS Trace →](03-bfs-step-by-step-trace.md)
