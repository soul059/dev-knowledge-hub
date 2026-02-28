# Floyd-Warshall Algorithm

[← Previous: Bellman-Ford Algorithm](02-bellman-ford-algorithm.md) | [Back to TOC](../README.md) | [Next: Algorithm Comparison →](04-algorithm-comparison.md)

---

## Key Idea: All-Pairs Shortest Path

```
    Find shortest path between EVERY pair of vertices.
    
    Dynamic Programming approach:
    dist[i][j] = shortest path from i to j using 
                 only intermediate vertices {0, 1, ..., k}
    
    For each intermediate vertex k:
        dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
    
    "Is it shorter to go through k?"
```

## Algorithm

```
FUNCTION floydWarshall(V, graph):
    // Initialize distance matrix
    dist = V × V matrix, all = ∞
    FOR i = 0 to V-1:
        dist[i][i] = 0
    FOR each edge (u, v, w):
        dist[u][v] = w
    
    // ★ k MUST be the OUTERMOST loop!
    FOR k = 0 to V-1:                 // intermediate vertex
        FOR i = 0 to V-1:             // source
            FOR j = 0 to V-1:         // destination
                IF dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    
    RETURN dist
```

## Why k Must Be Outermost

```
    The recurrence builds up from smaller subproblems:
    
    Using vertices {0}     → dist after k=0
    Using vertices {0,1}   → dist after k=1
    Using vertices {0,1,2} → dist after k=2
    ...
    
    If k is not outermost, we might use vertex k
    as intermediate BEFORE we've computed paths
    through vertices {0..k-1} → WRONG results!
```

## Step-by-Step Trace

```
    Graph (directed, weighted):
       (0) ─3→ (1)
        │        │
       ↓8       ↓-2
        │        │
       (2) ←7── (3)
    
    Initial dist matrix:
         0    1    2    3
    0 [  0    3    8    ∞  ]
    1 [  ∞    0    ∞   -2  ]
    2 [  ∞    ∞    0    ∞  ]
    3 [  ∞    ∞    7    0  ]
    
    k=0 (through vertex 0):
    dist[1][2] = min(∞, dist[1][0]+dist[0][2]) = min(∞, ∞+8) = ∞
    No changes (can't reach 0 from 1,2,3).
    
    k=1 (through vertex 1):
    dist[0][3] = min(∞, 3+(-2)) = 1  ★ improved!
    
    k=2 (through vertex 2): no improvements
    
    k=3 (through vertex 3):
    dist[0][2] = min(8, dist[0][3]+dist[3][2]) = min(8, 1+7) = 8
    dist[1][2] = min(∞, dist[1][3]+dist[3][2]) = min(∞, -2+7) = 5 ★
    
    Final:
         0    1    2    3
    0 [  0    3    8    1  ]
    1 [  ∞    0    5   -2  ]
    2 [  ∞    ∞    0    ∞  ]
    3 [  ∞    ∞    7    0  ]
```

## Detecting Negative Cycles

```
    After Floyd-Warshall, check the diagonal:
    
    IF dist[i][i] < 0 for any i → NEGATIVE CYCLE exists!
    
    (A path from i back to i with negative total weight
     means a negative cycle going through i.)
```

## Complexity

```
    Time:  O(V³)     — three nested loops
    Space: O(V²)     — distance matrix
    
    Best for: dense graphs, all-pairs queries,
    small V (V ≤ 400 typically)
```

---

[← Previous: Bellman-Ford Algorithm](02-bellman-ford-algorithm.md) | [Back to TOC](../README.md) | [Next: Algorithm Comparison →](04-algorithm-comparison.md)
