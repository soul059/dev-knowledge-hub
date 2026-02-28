# Path Reconstruction

[← Previous: Algorithm Comparison](04-algorithm-comparison.md) | [Back to TOC](../README.md) | [Next: Practical Tips →](06-practical-tips.md)

---

## Dijkstra's / Bellman-Ford: Parent Array

```
    Track parent[v] = the vertex that gave v its shortest distance.
    
    During relaxation:
    IF dist[u] + w < dist[v]:
        dist[v] = dist[u] + w
        parent[v] = u                  // ← track parent!
    
    Reconstruct path (target → source):
    path = []
    current = target
    WHILE current != -1:
        path.addFirst(current)
        current = parent[current]
    
    Example:
    parent = [-1, 0, 0, 1, 1, 3]
    Path to 5: 5 ← 3 ← 1 ← 0
    Path: [0, 1, 3, 5]
```

## Floyd-Warshall: Next Matrix

```
    For all-pairs, use next[i][j] = the next vertex 
    on the shortest path from i to j.
    
    Initialize:
    next[i][j] = j    (for each edge i→j)
    next[i][j] = -1   (if no edge)
    
    During DP:
    IF dist[i][k] + dist[k][j] < dist[i][j]:
        dist[i][j] = dist[i][k] + dist[k][j]
        next[i][j] = next[i][k]           // ← go toward k first
    
    Reconstruct path from i to j:
    path = [i]
    current = i
    WHILE current != j:
        current = next[current][j]
        path.append(current)
    RETURN path
```

## Example

```
    next matrix:
         0    1    2    3
    0 [ -1    1    2    1  ]
    1 [ -1   -1    3    3  ]
    2 [ -1   -1   -1   -1  ]
    3 [ -1   -1    2   -1  ]
    
    Path from 0 to 2:
    Start at 0, next[0][2] = 2 → go to 2
    Path: [0, 2] (direct)
    
    Path from 1 to 2:
    Start at 1, next[1][2] = 3 → go to 3
    At 3, next[3][2] = 2 → go to 2
    Path: [1, 3, 2]
```

---

[← Previous: Algorithm Comparison](04-algorithm-comparison.md) | [Back to TOC](../README.md) | [Next: Practical Tips →](06-practical-tips.md)
