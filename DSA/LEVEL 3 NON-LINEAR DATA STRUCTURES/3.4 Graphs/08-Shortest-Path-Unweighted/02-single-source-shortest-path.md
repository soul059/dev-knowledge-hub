# Single Source Shortest Path

[← Previous: BFS for Shortest Path](01-bfs-for-shortest-path.md) | [Back to TOC](../README.md) | [Next: Word Ladder →](03-word-ladder.md)

---

## BFS with Distance and Parent Tracking

```
FUNCTION shortestPath(graph, source, V):
    dist = array of size V, all = ∞
    parent = array of size V, all = -1
    dist[source] = 0
    
    queue = [source]
    
    WHILE queue is not empty:
        u = queue.dequeue()
        
        FOR each neighbor v in graph[u]:
            IF dist[v] == ∞:                // not yet discovered
                dist[v] = dist[u] + 1       // distance = parent + 1
                parent[v] = u               // track parent for path
                queue.enqueue(v)
    
    RETURN dist, parent
```

## Step-by-Step Trace

```
    Graph:
       (0)──(1)──(4)
        │    │
       (2)──(3)──(5)
    
    BFS from source = 0:
    
    Step │ Dequeue │ Process Neighbors     │ dist[]
    ─────┼─────────┼───────────────────────┼────────────────
      0  │   —     │ enqueue 0            │ [0,∞,∞,∞,∞,∞]
      1  │   0     │ 1(d=1), 2(d=1)       │ [0,1,1,∞,∞,∞]
      2  │   1     │ 3(d=2), 4(d=2)       │ [0,1,1,2,2,∞]
      3  │   2     │ 3(visited)           │ [0,1,1,2,2,∞]
      4  │   3     │ 5(d=3)              │ [0,1,1,2,2,3]
      5  │   4     │ —                   │ [0,1,1,2,2,3]
      6  │   5     │ —                   │ [0,1,1,2,2,3]
    
    Final distances from 0: [0, 1, 1, 2, 2, 3]
    Parent array: [-1, 0, 0, 1, 1, 3]
    
    Path from 0 to 5:
    5 ← parent[5]=3 ← parent[3]=1 ← parent[1]=0
    Path: 0 → 1 → 3 → 5 (length = 3) ✓
```

---

[← Previous: BFS for Shortest Path](01-bfs-for-shortest-path.md) | [Back to TOC](../README.md) | [Next: Word Ladder →](03-word-ladder.md)
