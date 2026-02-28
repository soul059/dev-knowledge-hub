# Applications, Complexity, and Common BFS Patterns

[← Previous: BFS on a Grid](06-bfs-on-a-grid.md) | [Back to TOC](../README.md) | [Next Unit: Depth-First Search →](../04-Depth-First-Search/01-dfs-intuition.md)

---

## Applications of BFS

```
    ┌──────────────────────────────────────────────────┐
    │              BFS Applications Map                │
    ├──────────────────────────────────────────────────┤
    │                                                  │
    │  Shortest Path  ────  Web Crawling               │
    │       │                    │                      │
    │       ▼                    ▼                      │
    │  Level Order   ────  Social Networks             │
    │       │              (degrees of separation)     │
    │       ▼                                          │
    │  Connected     ────  Puzzle Solving              │
    │  Components          (min moves)                 │
    │       │                                          │
    │       ▼                                          │
    │  Bipartite     ────  Network Broadcasting        │
    │  Check                                           │
    └──────────────────────────────────────────────────┘
```

## Distance Tracking with BFS

```
FUNCTION BFS_Distance(graph, source):
    dist = array of size V, initialized to -1
    dist[source] = 0
    
    queue = new Queue()
    queue.enqueue(source)
    
    WHILE queue is not empty:
        vertex = queue.dequeue()
        
        FOR each neighbor in graph[vertex]:
            IF dist[neighbor] == -1:           // not visited
                dist[neighbor] = dist[vertex] + 1
                queue.enqueue(neighbor)
    
    RETURN dist    // dist[v] = shortest distance from source to v
                   // dist[v] = -1 if v is unreachable
```

## Parent Tracking (Path Reconstruction)

```
FUNCTION BFS_Path(graph, source, target):
    parent = array of size V, initialized to -1
    visited = set containing {source}
    queue = [source]
    
    WHILE queue is not empty:
        vertex = queue.dequeue()
        
        IF vertex == target:
            RETURN reconstructPath(parent, source, target)
        
        FOR each neighbor in graph[vertex]:
            IF neighbor NOT in visited:
                visited.add(neighbor)
                parent[neighbor] = vertex
                queue.enqueue(neighbor)
    
    RETURN "no path exists"

FUNCTION reconstructPath(parent, source, target):
    path = []
    current = target
    WHILE current != -1:
        path.addFirst(current)
        current = parent[current]
    RETURN path
```

## Time and Space Complexity

```
    TIME:  O(V + E)
    ─────────────────────────────────
    - Each vertex is enqueued at most once  → O(V)
    - Each edge is examined at most twice   → O(E)
      (once from each endpoint in undirected graph)
    - Total: O(V + E)

    SPACE: O(V)
    ─────────────────────────────────
    - Visited set: O(V)
    - Queue: at most O(V) vertices
    - Total: O(V)
    
    For grid: V = R×C, E ≈ 4×R×C
    So: O(R×C) time and space
```

## Common BFS Patterns

### Pattern 1: Multi-Source BFS

```
    Start BFS from MULTIPLE sources simultaneously.
    Enqueue ALL sources at the start.
    
    Example: "Rotten Oranges" — all rotten oranges spread at once
    
    queue = all initial rotten oranges
    // BFS proceeds as normal
    // Level = time step
```

### Pattern 2: State-Space BFS

```
    When the "graph" is not explicit but defined by states.
    
    State: a configuration (e.g., puzzle state, position + keys)
    Edge:  a valid move between states
    
    visited = set of STATES (not just vertices)
    
    Example: "Open the Lock" — state is the lock combination
             Each move changes one digit by ±1
             Find minimum moves from "0000" to target
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Data Structure** | Queue (FIFO) |
| **Exploration Order** | Level by level |
| **Shortest Path** | Yes (unweighted graphs) |
| **Time Complexity** | O(V + E) |
| **Space Complexity** | O(V) |
| **Mark Visited** | When ENQUEUING (not dequeuing) |
| **Grid BFS** | Direction arrays: dr, dc |
| **Multi-Source** | Enqueue all sources initially |

## Quick Revision Questions

1. **What data structure does BFS use and why?**
   <details><summary>Answer</summary> Queue (FIFO): ensures vertices at distance d are processed before those at distance d+1, guaranteeing level-by-level exploration.</details>

2. **Why should you mark vertices as visited when enqueuing, not when dequeuing?**
   <details><summary>Answer</summary> To prevent the same vertex from being enqueued multiple times by different neighbors, which would waste time and could cause incorrect results.</details>

3. **What is the time complexity of BFS and why?**
   <details><summary>Answer</summary> O(V + E). Each vertex is enqueued/dequeued once (O(V)), and each edge is examined at most twice (O(E)).</details>

4. **How do you track the level number during BFS?**
   <details><summary>Answer</summary> Read the queue size at the start of each level, then dequeue exactly that many vertices before incrementing the level counter.</details>

5. **Can BFS find shortest paths in weighted graphs?**
   <details><summary>Answer</summary> No. BFS only finds shortest paths in unweighted graphs. For weighted graphs, use Dijkstra's or Bellman-Ford.</details>

6. **What is Multi-Source BFS?**
   <details><summary>Answer</summary> BFS starting from multiple source vertices simultaneously (enqueue all at start). Used for problems like "Rotten Oranges" where multiple spreading points exist.</details>

---

[← Previous: BFS on a Grid](06-bfs-on-a-grid.md) | [Back to TOC](../README.md) | [Next Unit: Depth-First Search →](../04-Depth-First-Search/01-dfs-intuition.md)
