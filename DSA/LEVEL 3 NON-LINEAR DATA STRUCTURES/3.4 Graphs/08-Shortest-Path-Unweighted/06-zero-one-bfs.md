# 0-1 BFS

[← Previous: Multi-Source BFS](05-multi-source-bfs.md) | [Back to TOC](../README.md) | [Next: Bidirectional BFS →](07-bidirectional-bfs.md)

---

## The Problem

```
    Graph where edges have weight 0 or 1 only.
    Find shortest path from source.
    
    Normal BFS doesn't work (it counts edges, not weights).
    Dijkstra's works but is O((V+E) log V).
    
    0-1 BFS: O(V + E) using a DEQUE!
```

## Key Idea: Use a Deque

```
    When processing vertex u and examining neighbor v:
    
    - Weight 0 edge: v is at SAME distance level
      → Add to FRONT of deque (process sooner)
    
    - Weight 1 edge: v is at NEXT distance level
      → Add to BACK of deque (process later)
    
    This maintains the invariant that the deque
    is sorted by distance, just like BFS with a queue!
```

## Algorithm

```
FUNCTION BFS_01(graph, source, V):
    dist = array of size V, all = ∞
    dist[source] = 0
    
    deque = new Deque()
    deque.pushFront(source)
    
    WHILE deque not empty:
        u = deque.popFront()
        
        FOR each (neighbor v, weight w) in graph[u]:
            IF dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                
                IF w == 0:
                    deque.pushFront(v)     // ← same level → front
                ELSE:
                    deque.pushBack(v)      // ← next level → back
    
    RETURN dist
```

## Trace

```
    Graph (edges labeled 0 or 1):
       (A) ─1─ (B)
        │       │
        0       1
        │       │
       (C) ─0─ (D) ─1─ (E)
    
    Start from A:
    dist = [0, ∞, ∞, ∞, ∞]
    deque = [A]
    
    Pop A: process B(w=1): dist[B] = 1, pushBack(B)
           process C(w=0): dist[C] = 0, pushFront(C)
    deque = [C, B]
    
    Pop C: process D(w=0): dist[D] = 0, pushFront(D)
    deque = [D, B]
    
    Pop D: process B(w=1): dist[D]+1=1 = dist[B] → skip
           process E(w=1): dist[E] = 1, pushBack(E)
    deque = [B, E]
    
    Pop B: already optimal → skip all
    Pop E: done
    
    Final dist: A=0, B=1, C=0, D=0, E=1
```

## When to Use 0-1 BFS

```
    ✓ Edges have ONLY weights 0 and 1
    ✓ Want O(V + E) instead of Dijkstra's O((V+E) log V)
    ✓ Grid problems where some moves are "free"
    
    Example: Grid where moving on roads costs 0, 
    moving on grass costs 1.
```

---

[← Previous: Multi-Source BFS](05-multi-source-bfs.md) | [Back to TOC](../README.md) | [Next: Bidirectional BFS →](07-bidirectional-bfs.md)
