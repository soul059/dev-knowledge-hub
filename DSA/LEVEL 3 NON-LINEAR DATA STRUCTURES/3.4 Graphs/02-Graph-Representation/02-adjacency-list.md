# Adjacency List

[← Previous: Adjacency Matrix](01-adjacency-matrix.md) | [Back to TOC](../README.md) | [Next: Edge List →](03-edge-list.md)

---

An **adjacency list** stores, for each vertex, a **list of its neighbors**. This is the most common representation.

## Undirected Graph → Adjacency List

```
       (0)────(1)          Adjacency List:
        │    ╱ │           
        │   ╱  │           0 → [1, 2]
        │  ╱   │           1 → [0, 2, 3]
       (2)────(3)          2 → [0, 1, 3, 4]
        │                  3 → [1, 2]
        │                  4 → [2]
       (4)
                           Each edge appears TWICE
                           (once for each endpoint)
```

## Internal Structure (Array of Lists)

```
    Index │ Linked List / Array
    ──────┼─────────────────────────
      0   │ ──→ [1] ──→ [2] ──→ null
      1   │ ──→ [0] ──→ [2] ──→ [3] ──→ null
      2   │ ──→ [0] ──→ [1] ──→ [3] ──→ [4] ──→ null
      3   │ ──→ [1] ──→ [2] ──→ null
      4   │ ──→ [2] ──→ null
```

## Directed Graph → Adjacency List

```
       (0)───→(1)          Adjacency List:
        │      │           
        ▼      ▼           0 → [1, 2]
       (2)←──(3)           1 → [3]
                           2 → []
                           3 → [2]

                           Each edge appears ONCE
                           (only from source)
```

## Weighted Graph → Adjacency List

```
       (0)─5─(1)          Adjacency List (with weights):
        │      │           
       2│      │3          0 → [(1,5), (2,2)]
        │      │           1 → [(0,5), (3,3)]
       (2)─1─(3)          2 → [(0,2), (3,1)]
                           3 → [(1,3), (2,1)]

                           Store (neighbor, weight) pairs
```

## Pseudocode: Build Adjacency List

```
FUNCTION buildAdjList(V, edges):
    adjList = array of V empty lists
    
    FOR each edge (u, v) in edges:
        adjList[u].append(v)
        adjList[v].append(u)      // omit for directed graph
    
    RETURN adjList

// For weighted graphs:
FUNCTION buildWeightedAdjList(V, edges):
    adjList = array of V empty lists

    FOR each edge (u, v, w) in edges:
        adjList[u].append((v, w))
        adjList[v].append((u, w))  // omit for directed graph
    
    RETURN adjList
```

## Common Implementation: HashMap of Lists

```
    For vertices labeled with strings:

    graph = {
        "NYC":     ["Boston", "DC"],
        "Boston":  ["NYC"],
        "DC":      ["NYC", "Miami"],
        "Miami":   ["DC"]
    }

    This is identical in concept — keys are vertices,
    values are lists of neighbors.
```

## Operations on Adjacency List

```
Check if edge (u,v) exists:     search adjList[u]       → O(degree(u))

Get all neighbors of u:         return adjList[u]        → O(1) to access,
                                                           O(degree(u)) to iterate

Add edge (u,v):                 adjList[u].append(v)     → O(1)

Remove edge (u,v):              remove v from adjList[u] → O(degree(u))

Get degree of u:                len(adjList[u])          → O(1)

Space used:                     V + 2E (undirected)      → O(V + E)
                                V + E  (directed)
```

---

[← Previous: Adjacency Matrix](01-adjacency-matrix.md) | [Back to TOC](../README.md) | [Next: Edge List →](03-edge-list.md)
