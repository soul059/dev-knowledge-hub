# Kruskal's Algorithm

[← Previous: What is a Spanning Tree?](01-what-is-a-spanning-tree.md) | [Back to TOC](../README.md) | [Next: Prim's Algorithm →](03-prims-algorithm.md)

---

## Key Idea

```
    1. Sort ALL edges by weight (ascending)
    2. Process edges one by one:
       - If adding this edge creates a cycle → SKIP
       - Otherwise → ADD to MST
    3. Stop when MST has V-1 edges
    
    Uses UNION-FIND to efficiently check cycles!
```

## Algorithm

```
FUNCTION kruskal(V, edges):
    SORT edges by weight (ascending)
    
    parent = [0, 1, 2, ..., V-1]
    rank = [0, 0, ..., 0]
    mstEdges = []
    mstWeight = 0
    
    FOR each edge (u, v, w) in sorted edges:
        IF find(parent, u) != find(parent, v):    // different sets?
            union(parent, rank, u, v)              // merge sets
            mstEdges.append((u, v, w))             // add to MST
            mstWeight += w
            
            IF mstEdges.size() == V - 1:           // MST complete!
                BREAK
    
    RETURN mstEdges, mstWeight
```

## Step-by-Step Trace

```
    Graph:
       (0)─4─(1)
        │╲    │
        8  2  6
        │    ╲│
       (2)─7─(3)─1─(4)
    
    Edges sorted by weight:
    (3,4,1), (0,3,2), (0,1,4), (1,3,6), (2,3,7), (0,2,8)
    
    Process:
    ┌────────┬──────────┬─────────┬────────────────────┐
    │ Edge   │ Weight   │ Action  │ Sets               │
    ├────────┼──────────┼─────────┼────────────────────┤
    │ (3,4)  │ 1        │ ADD ✓   │ {0}{1}{2}{3,4}     │
    │ (0,3)  │ 2        │ ADD ✓   │ {0,3,4}{1}{2}      │
    │ (0,1)  │ 4        │ ADD ✓   │ {0,1,3,4}{2}       │
    │ (1,3)  │ 6        │ SKIP ✗  │ same set (cycle!)  │
    │ (2,3)  │ 7        │ ADD ✓   │ {0,1,2,3,4}        │
    └────────┴──────────┴─────────┴────────────────────┘
    
    MST edges: (3,4), (0,3), (0,1), (2,3)
    MST weight: 1 + 2 + 4 + 7 = 14
    V-1 = 4 edges ✓
```

## Complexity

```
    Time:  O(E log E)   — dominated by sorting edges
           Union-Find operations: O(E × α(V)) ≈ O(E)
    Space: O(V + E)
    
    Best for: sparse graphs, edge list representation
```

---

[← Previous: What is a Spanning Tree?](01-what-is-a-spanning-tree.md) | [Back to TOC](../README.md) | [Next: Prim's Algorithm →](03-prims-algorithm.md)
