# Bellman-Ford Algorithm

[← Previous: Dijkstra's Algorithm](01-dijkstras-algorithm.md) | [Back to TOC](../README.md) | [Next: Floyd-Warshall Algorithm →](03-floyd-warshall-algorithm.md)

---

## Key Idea

```
    Relax ALL edges, V-1 times.
    
    Why V-1? The shortest path has at most V-1 edges.
    After i iterations, all shortest paths using ≤ i edges
    are correct. After V-1 iterations, ALL are correct.
    
    ★ Works with NEGATIVE weights!
    ★ Can DETECT negative cycles!
```

## Algorithm

```
FUNCTION bellmanFord(V, edges, source):
    dist = array of size V, all = ∞
    dist[source] = 0
    
    // Step 1: Relax all edges V-1 times
    FOR i = 1 to V-1:
        FOR each edge (u, v, w) in edges:
            IF dist[u] != ∞ AND dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    
    // Step 2: Check for negative cycles
    FOR each edge (u, v, w) in edges:
        IF dist[u] != ∞ AND dist[u] + w < dist[v]:
            RETURN "Negative cycle detected!"
    
    RETURN dist
```

## Step-by-Step Trace

```
    Graph:
       (A) ─6─→ (B)
        │         │
       ↓7        ↓-3
        │         │
       (C) ─5─→ (D) ─-2─→ (E)
        │                    ↑
        └─────────8──────────┘
    
    Edges: (A,B,6), (A,C,7), (B,D,-3), (C,D,5), (C,E,8), (D,E,-2)
    
    Initial:  dist = [0, ∞, ∞, ∞, ∞]
    
    Iteration 1 (relax all edges):
    (A,B,6): dist[B] = min(∞, 0+6) = 6
    (A,C,7): dist[C] = min(∞, 0+7) = 7
    (B,D,-3): dist[D] = min(∞, 6-3) = 3
    (C,D,5): dist[D] = min(3, 7+5) = 3 (no change)
    (C,E,8): dist[E] = min(∞, 7+8) = 15
    (D,E,-2): dist[E] = min(15, 3-2) = 1
    
    dist = [0, 6, 7, 3, 1]
    
    Iterations 2-4: no changes (already optimal)
    
    Negative cycle check: no edge can be further relaxed ✓
    
    Final: A=0, B=6, C=7, D=3, E=1
```

## Negative Cycle Detection

```
    After V-1 iterations, if ANY edge can STILL be relaxed,
    a negative cycle exists (distances keep decreasing forever).
    
       A ─1→ B ─(-3)→ C ─1→ A     (cycle weight = 1-3+1 = -1)
    
    Each trip around the cycle decreases total distance by 1.
    After V-1 iterations, we can STILL find improvements
    → Negative cycle detected!
```

## Complexity

```
    Time:  O(V × E)     — V-1 iterations × E edge relaxations
    Space: O(V)          — distance array
    
    Slower than Dijkstra's but handles negative weights!
```

---

[← Previous: Dijkstra's Algorithm](01-dijkstras-algorithm.md) | [Back to TOC](../README.md) | [Next: Floyd-Warshall Algorithm →](03-floyd-warshall-algorithm.md)
