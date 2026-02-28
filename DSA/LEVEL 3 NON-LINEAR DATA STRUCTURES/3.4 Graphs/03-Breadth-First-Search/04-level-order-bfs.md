# Level-Order BFS

[← Previous: Step-by-Step BFS Trace](03-bfs-step-by-step-trace.md) | [Back to TOC](../README.md) | [Next: BFS Properties →](05-bfs-properties.md)

---

Sometimes we need to process the graph **level by level** (e.g., shortest distance, tree level order). We modify BFS to track levels:

## Level-Order BFS Pseudocode

```
FUNCTION BFS_LevelOrder(graph, source):
    visited = set containing {source}
    queue = new Queue()
    queue.enqueue(source)
    level = 0
    
    WHILE queue is not empty:
        size = queue.size()        // ← Number of vertices in THIS level
        
        PRINT "Level", level, ":"
        
        FOR i = 1 to size:         // ← Process exactly 'size' vertices
            vertex = queue.dequeue()
            PRINT vertex
            
            FOR each neighbor in graph[vertex]:
                IF neighbor NOT in visited:
                    visited.add(neighbor)
                    queue.enqueue(neighbor)
        
        level = level + 1

    // ★ The key trick is using queue.size() BEFORE the inner loop
    //   to process exactly one level at a time.
```

## Why This Works

```
    At the start of each outer loop iteration,
    the queue contains EXACTLY the vertices at
    the current level.

    Queue at level start:    [B, C]        ← 2 vertices at level 1
    Process B → enqueue D
    Process C → enqueue E
    Queue after level 1:     [D, E]        ← 2 vertices at level 2
    
    By reading the size BEFORE the inner loop,
    we know how many to process for this level.
```

---

[← Previous: Step-by-Step BFS Trace](03-bfs-step-by-step-trace.md) | [Back to TOC](../README.md) | [Next: BFS Properties →](05-bfs-properties.md)
