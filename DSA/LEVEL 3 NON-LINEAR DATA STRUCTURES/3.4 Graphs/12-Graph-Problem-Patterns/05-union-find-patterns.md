# Union-Find Patterns

[← Previous: Graph Coloring](04-graph-coloring.md) | [Back to TOC](../README.md) | [Next: Topological Sort Patterns →](06-topological-sort-patterns.md)

---

## When to Use Union-Find

```
    ┌──────────────────────────────────────────────┐
    │  Use Union-Find when:                        │
    │                                              │
    │  1. "Are X and Y connected?"                 │
    │  2. "Merge these groups"                     │
    │  3. "How many groups?"                       │
    │  4. Dynamic connectivity (edges added online)│
    │  5. Cycle detection in undirected graphs     │
    │  6. Kruskal's MST                            │
    └──────────────────────────────────────────────┘
    
    NOT ideal when:
    • Need to DELETE edges (Union-Find doesn't support un-union)
    • Need shortest paths
    • Need to traverse neighbors
```

## Template

```
FUNCTION find(parent, x):
    IF parent[x] != x:
        parent[x] = find(parent, parent[x])    // path compression
    RETURN parent[x]

FUNCTION union(parent, rank, x, y):
    rootX = find(parent, x)
    rootY = find(parent, y)
    IF rootX == rootY: RETURN false             // already connected
    IF rank[rootX] < rank[rootY]: swap(rootX, rootY)
    parent[rootY] = rootX                       // union by rank
    IF rank[rootX] == rank[rootY]: rank[rootX]++
    RETURN true

// Initialize
parent = [0, 1, 2, ..., n-1]
rank = [0, 0, ..., 0]
components = n
```

## Classic Problems

### 1. Number of Connected Components

```
    Given n nodes and edge list, count components.
    
    FOR each edge (u, v):
        IF union(u, v):        // merged two different components
            components--
    RETURN components
```

### 2. Redundant Connection

```
    Tree with one extra edge → find the extra edge.
    
    FOR each edge (u, v):
        IF find(u) == find(v):  // already connected → this is extra!
            RETURN (u, v)
        union(u, v)
```

### 3. Accounts Merge

```
    Group accounts that share any email.
    
    • Map each email to first account that uses it
    • If email seen again → union the two accounts
    • Collect by root → merged groups
```

### 4. Number of Islands II (Online)

```
    Grid starts empty. Add land cells one by one.
    After each addition, report number of islands.
    
    • Add cell → components++
    • Check 4 neighbors: if neighbor is land AND different root
      → union them, components--
    • Report components after each step
```

### 5. Largest Component by Size

```
    Track component sizes:
    
    size = [1, 1, ..., 1]   // each vertex starts as size 1
    
    FUNCTION union(x, y):
        ...
        size[rootX] += size[rootY]   // merge sizes
        maxSize = max(maxSize, size[rootX])
```

---

[← Previous: Graph Coloring](04-graph-coloring.md) | [Back to TOC](../README.md) | [Next: Topological Sort Patterns →](06-topological-sort-patterns.md)
