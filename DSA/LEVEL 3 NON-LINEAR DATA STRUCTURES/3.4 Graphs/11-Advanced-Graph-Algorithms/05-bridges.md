# Bridges (Cut Edges)

[← Previous: Articulation Points](04-articulation-points.md) | [Back to TOC](../README.md) | [Next: Eulerian Path and Circuit →](06-eulerian-path-and-circuit.md)

---

## Definition

```
    A BRIDGE (cut edge) is an edge whose removal
    DISCONNECTS the graph.
    
    Bridge condition for edge (u, v) where v is child of u:
    
        low[v] > disc[u]     ← STRICTLY greater!
    
    Compare with Articulation Point:
        low[v] >= disc[u]    ← Greater or equal
    
    Why strictly >?
    If low[v] == disc[u], v can reach u through a back edge,
    so removing edge (u,v) doesn't disconnect — u is still
    reachable. But u becomes an AP because v's subtree 
    can ONLY reach back to u, not above.
```

## Visual Example

```
    (0)───(1)───(3)───(4)
           │           │
          (2)         (5)
    
    Bridges: (0,1), (1,3)
    NOT bridges: (3,4), (4,5), (3,5) form a cycle — removing 
    any one doesn't disconnect.
    
    Wait — let me redraw properly:
    
    (0)───(1)───(3)───(4)
           │     │     │
          (2)   (5)───(6)
    
    Edges: 0-1, 1-2, 1-3, 3-4, 3-5, 4-6, 5-6
    
    Bridges: (0,1), (1,2), (1,3)
    Edge 3-4-6-5-3 forms a cycle → none of those are bridges
```

## Algorithm

```
FUNCTION findBridges(V, graph):
    disc = array of size V, all -1
    low = array of size V, all -1
    parent = array of size V, all -1
    bridges = []
    timer = 0
    
    FOR i = 0 to V-1:
        IF disc[i] == -1:
            dfs(i, graph, disc, low, parent, bridges, timer)
    
    RETURN bridges

FUNCTION dfs(u, graph, disc, low, parent, bridges, timer):
    disc[u] = low[u] = timer++
    
    FOR each v in graph[u]:
        IF disc[v] == -1:                    // tree edge
            parent[v] = u
            dfs(v, ...)
            low[u] = min(low[u], low[v])
            
            IF low[v] > disc[u]:             // STRICTLY greater
                bridges.append((u, v))
        
        ELSE IF v != parent[u]:              // back edge
            low[u] = min(low[u], disc[v])
```

## AP vs Bridge Condition — Summary

```
    ┌────────────────────┬─────────────────────┐
    │ Articulation Point │ Bridge              │
    ├────────────────────┼─────────────────────┤
    │ low[v] >= disc[u]  │ low[v] > disc[u]    │
    │ (can equal)        │ (strictly greater)  │
    │ Vertex removal     │ Edge removal        │
    │ Root: 2+ children  │ No special root case│
    └────────────────────┴─────────────────────┘
    
    Intuition:
    - If low[v] == disc[u]: v can reach u via back edge.
      Edge (u,v) is NOT a bridge (alternate path exists).
      But u IS an AP (v can't go ABOVE u).
    
    - If low[v] > disc[u]: v CANNOT reach u at all
      without edge (u,v). Edge IS a bridge.
```

---

[← Previous: Articulation Points](04-articulation-points.md) | [Back to TOC](../README.md) | [Next: Eulerian Path and Circuit →](06-eulerian-path-and-circuit.md)
