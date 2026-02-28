# Dijkstra's Algorithm

[Back to TOC](../README.md) | [Next: Bellman-Ford Algorithm →](02-bellman-ford-algorithm.md)

---

## Key Idea: Greedy + Priority Queue

```
    Dijkstra's finds shortest path from a single source
    to ALL other vertices in a WEIGHTED graph.
    
    Strategy (greedy):
    Always process the CLOSEST unfinalized vertex.
    Once a vertex is finalized, its distance is optimal.
    
    ┌────────────────────────────────────────┐
    │  Requirement: NO NEGATIVE WEIGHTS      │
    │  If negative weights exist → use       │
    │  Bellman-Ford instead!                 │
    └────────────────────────────────────────┘
```

## Algorithm (Min-Heap / Lazy Deletion)

```
FUNCTION dijkstra(graph, source, V):
    dist = array of size V, all = ∞
    dist[source] = 0
    
    minHeap = priority queue (min by distance)
    minHeap.push((0, source))           // (distance, vertex)
    
    WHILE minHeap not empty:
        (d, u) = minHeap.pop()          // closest unfinalized vertex
        
        IF d > dist[u]:                 // ★ lazy deletion
            CONTINUE                    // stale entry, skip
        
        FOR each (neighbor v, weight w) in graph[u]:
            IF dist[u] + w < dist[v]:   // relaxation
                dist[v] = dist[u] + w
                minHeap.push((dist[v], v))   // may create duplicate
    
    RETURN dist
```

## Why Lazy Deletion?

```
    When we update dist[v], we push a NEW entry to the heap
    without removing the old one. Multiple entries for same
    vertex may exist in the heap.
    
    When we pop an entry (d, u):
    - If d > dist[u] → this is an OLD entry → skip it
    - If d == dist[u] → this is the CURRENT best → process it
    
    This avoids the expensive decrease-key operation!
```

## Step-by-Step Trace

```
    Graph:
       (A) ─4─ (B)
        │ ╲      │
        2   1    3
        │     ╲  │
       (C) ─5─ (D)
    
    Dijkstra from A:
    
    Step │ Pop        │ Relax Neighbors      │ dist[]         │ Heap
    ─────┼────────────┼──────────────────────┼────────────────┼──────────
      0  │ —          │ —                    │ [0,∞,∞,∞]     │ [(0,A)]
      1  │ (0,A)      │ B:0+4=4, C:0+2=2,   │ [0,4,2,1]     │ [(1,D),
         │            │ D:0+1=1              │                │  (2,C),(4,B)]
      2  │ (1,D)      │ B:1+3=4 (=dist[B]), │ [0,4,2,1]     │ [(2,C),(4,B)]
         │            │ C:1+5=6 (>dist[C])  │                │
      3  │ (2,C)      │ no improvements     │ [0,4,2,1]     │ [(4,B)]
      4  │ (4,B)      │ no improvements     │ [0,4,2,1]     │ []
    
    Final: A=0, B=4, C=2, D=1
    Shortest path A→B: A→D→B (cost 1+3=4) or A→B (cost 4)
```

## Why Dijkstra's Fails with Negative Weights

```
    Graph:
       (A) ─1─ (B) ─(-5)─ (C)
        │                    
        3                   
        │                    
       (D) ─────2────────→ (C)
    
    Dijkstra processes B (dist=1), then C via B (dist=1-5=-4).
    But wait — C is already "finalized"!
    Later, D→C = 3+2 = 5 which is worse.
    
    Problem: Dijkstra might finalize a vertex too early
    when negative edges can improve it later.
    → Use Bellman-Ford for negative weights!
```

## Complexity

```
    Time:  O((V + E) log V)    with binary heap
    Space: O(V + E)

    With Fibonacci heap: O(V log V + E) — rarely used in practice
```

---

[Back to TOC](../README.md) | [Next: Bellman-Ford Algorithm →](02-bellman-ford-algorithm.md)
