# Cycle in Undirected Graph (Union-Find)

[← Previous: Cycle Detection DFS](01-cycle-undirected-dfs.md) | [Back to TOC](../README.md) | [Next: Cycle in Directed Graph →](03-cycle-directed-dfs-colors.md)

---

## Key Idea

```
    Process edges one by one.
    For each edge (u, v):
    - If u and v are in the SAME set → adding this edge creates a CYCLE
    - If u and v are in DIFFERENT sets → union them (safe, no cycle yet)
```

## Union-Find Data Structure

```
FUNCTION find(parent, x):
    IF parent[x] != x:
        parent[x] = find(parent, parent[x])    // path compression
    RETURN parent[x]

FUNCTION union(parent, rank, x, y):
    rootX = find(parent, x)
    rootY = find(parent, y)
    
    IF rootX == rootY:
        RETURN false                             // ★ CYCLE!
    
    // Union by rank
    IF rank[rootX] < rank[rootY]:
        parent[rootX] = rootY
    ELSE IF rank[rootX] > rank[rootY]:
        parent[rootY] = rootX
    ELSE:
        parent[rootY] = rootX
        rank[rootX] += 1
    
    RETURN true                                  // successfully merged
```

## Algorithm

```
FUNCTION hasCycle_UnionFind(V, edges):
    parent = [0, 1, 2, ..., V-1]     // each vertex is its own parent
    rank = [0, 0, 0, ..., 0]
    
    FOR each edge (u, v) in edges:
        IF NOT union(parent, rank, u, v):
            RETURN true                // same set → CYCLE!
    
    RETURN false
```

## Step-by-Step Trace

```
    Edges: (0,1), (1,3), (3,2), (2,0)
    
    Initial: parent = [0, 1, 2, 3]    (each is own root)
    
    Edge (0,1): find(0)=0, find(1)=1 → different → union
                parent = [0, 0, 2, 3]
                Sets: {0,1} {2} {3}
    
    Edge (1,3): find(1)=0, find(3)=3 → different → union
                parent = [0, 0, 2, 0]
                Sets: {0,1,3} {2}
    
    Edge (3,2): find(3)=0, find(2)=2 → different → union
                parent = [0, 0, 0, 0]
                Sets: {0,1,2,3}
    
    Edge (2,0): find(2)=0, find(0)=0 → ★ SAME SET → CYCLE!
    
    The edge (2,0) connects two vertices already in the
    same component → it must create a cycle.
```

## Complexity

```
    Time:  O(E × α(V)) ≈ O(E)   — α is inverse Ackermann (nearly constant)
    Space: O(V)                   — parent and rank arrays
```

---

[← Previous: Cycle Detection DFS](01-cycle-undirected-dfs.md) | [Back to TOC](../README.md) | [Next: Cycle in Directed Graph →](03-cycle-directed-dfs-colors.md)
