# Prim's Algorithm

[← Previous: Kruskal's Algorithm](02-kruskals-algorithm.md) | [Back to TOC](../README.md) | [Next: Kruskal's vs Prim's →](04-kruskals-vs-prims.md)

---

## Key Idea

```
    Start from ANY vertex. Grow the MST one vertex at a time.
    Always add the CHEAPEST edge that connects a new vertex
    to the growing tree.
    
    Similar to Dijkstra's but:
    - Dijkstra's: minimizes total distance from source
    - Prim's:     minimizes edge weight to tree
```

## Algorithm

```
FUNCTION prim(graph, V):
    inMST = array of size V, all false
    key = array of size V, all ∞          // minimum edge weight to MST
    parent = array of size V, all -1
    key[0] = 0                            // start from vertex 0
    
    minHeap = priority queue
    minHeap.push((0, 0))                  // (weight, vertex)
    
    mstWeight = 0
    
    WHILE minHeap not empty:
        (w, u) = minHeap.pop()
        
        IF inMST[u]: CONTINUE             // already in MST
        
        inMST[u] = true
        mstWeight += w
        
        FOR each (neighbor v, weight w) in graph[u]:
            IF NOT inMST[v] AND w < key[v]:
                key[v] = w
                parent[v] = u
                minHeap.push((w, v))
    
    RETURN mstWeight, parent
```

## Step-by-Step Trace

```
    Graph:
       (0)─4─(1)
        │╲    │
        8  2  6
        │    ╲│
       (2)─7─(3)─1─(4)
    
    Start from vertex 0:
    
    Step │ Pop      │ Add to MST │ Update Keys           │ key[]
    ─────┼──────────┼────────────┼───────────────────────┼──────────
      0  │ —        │ —          │ key[0]=0              │ [0,∞,∞,∞,∞]
      1  │ (0, 0)   │ 0          │ 1:4, 2:8, 3:2        │ [0,4,8,2,∞]
      2  │ (2, 3)   │ 3          │ 1:min(4,6)=4, 4:1    │ [0,4,8,2,1]
      3  │ (1, 4)   │ 4          │ —                     │ [0,4,8,2,1]
      4  │ (4, 1)   │ 1          │ 3:min(6,key)=4       │ [0,4,8,2,1]
      5  │ (7, 2)   │ 2          │ —                     │ [0,4,8,2,1]
    
    Wait — let me re-trace more carefully.
    
    Start: key=[0,∞,∞,∞,∞], heap=[(0,0)]
    
    Pop (0,0): MST={0}, check neighbors:
      1: w=4 < ∞ → key[1]=4, push(4,1)
      2: w=8 < ∞ → key[2]=8, push(8,2)  
      3: w=2 < ∞ → key[3]=2, push(2,3)
    Heap: [(2,3),(4,1),(8,2)]
    
    Pop (2,3): MST={0,3}, check neighbors:
      0: inMST → skip
      1: w=6 > key[1]=4 → skip
      2: w=7 < key[2]=8 → key[2]=7, push(7,2)
      4: w=1 < ∞ → key[4]=1, push(1,4)
    Heap: [(1,4),(4,1),(7,2),(8,2)]
    
    Pop (1,4): MST={0,3,4}, weight so far: 0+2+1=3
      3: inMST → skip
    Heap: [(4,1),(7,2),(8,2)]
    
    Pop (4,1): MST={0,1,3,4}, weight: 3+4=7
      0: inMST → skip
      3: inMST → skip
    Heap: [(7,2),(8,2)]
    
    Pop (7,2): MST={0,1,2,3,4}, weight: 7+7=14 ✓
    
    MST weight = 14 (same as Kruskal's!) ✓
```

---

[← Previous: Kruskal's Algorithm](02-kruskals-algorithm.md) | [Back to TOC](../README.md) | [Next: Kruskal's vs Prim's →](04-kruskals-vs-prims.md)
