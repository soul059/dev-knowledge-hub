# Adjacency Matrix

[Back to TOC](../README.md) | [Next: Adjacency List →](02-adjacency-list.md)

---

An **adjacency matrix** is a 2D array of size **V × V** where:
- `matrix[i][j] = 1` if there is an edge from vertex i to vertex j
- `matrix[i][j] = 0` otherwise

## Reference Graph

```
       (0)────(1)
        │    ╱ │
        │   ╱  │
        │  ╱   │
       (2)────(3)
        │
        │
       (4)

    V = {0, 1, 2, 3, 4}
    E = {(0,1), (0,2), (1,2), (1,3), (2,3), (2,4)}
```

## Undirected Graph → Adjacency Matrix

```
       (0)────(1)                    0   1   2   3   4
        │    ╱ │                  ┌───┬───┬───┬───┬───┐
        │   ╱  │              0  │ 0 │ 1 │ 1 │ 0 │ 0 │
        │  ╱   │                  ├───┼───┼───┼───┼───┤
       (2)────(3)             1  │ 1 │ 0 │ 1 │ 1 │ 0 │
        │                        ├───┼───┼───┼───┼───┤
        │                    2  │ 1 │ 1 │ 0 │ 1 │ 1 │
       (4)                       ├───┼───┼───┼───┼───┤
                              3  │ 0 │ 1 │ 1 │ 0 │ 0 │
                                  ├───┼───┼───┼───┼───┤
                              4  │ 0 │ 0 │ 1 │ 0 │ 0 │
                                  └───┴───┴───┴───┴───┘
    
    ✓ Symmetric: matrix[i][j] = matrix[j][i]
    ✓ Diagonal is 0 (no self-loops)
```

## Directed Graph → Adjacency Matrix

```
       (0)───→(1)                    0   1   2   3
        │      │                  ┌───┬───┬───┬───┐
        ▼      ▼              0  │ 0 │ 1 │ 1 │ 0 │
       (2)←──(3)                 ├───┼───┼───┼───┤
                              1  │ 0 │ 0 │ 0 │ 1 │
                                  ├───┼───┼───┼───┤
                              2  │ 0 │ 0 │ 0 │ 0 │
                                  ├───┼───┼───┼───┤
                              3  │ 0 │ 0 │ 1 │ 0 │
                                  └───┴───┴───┴───┘
    
    ✗ NOT symmetric (directed edges)
    Row i shows outgoing edges from i
    Column j shows incoming edges to j
```

## Weighted Graph → Adjacency Matrix

```
       (0)─5─(1)                     0    1    2    3
        │      │                  ┌────┬────┬────┬────┐
       2│      │3             0  │  0 │  5 │  2 │  ∞ │
        │      │                  ├────┼────┼────┼────┤
       (2)─1─(3)              1  │  5 │  0 │  ∞ │  3 │
                                  ├────┼────┼────┼────┤
                              2  │  2 │  ∞ │  0 │  1 │
                                  ├────┼────┼────┼────┤
                              3  │  ∞ │  3 │  1 │  0 │
                                  └────┴────┴────┴────┘
    
    ∞ (or -1 or MAX_INT) means "no edge"
    matrix[i][j] = weight of edge (i, j)
```

## Pseudocode: Build Adjacency Matrix

```
FUNCTION buildAdjMatrix(V, edges):
    matrix = 2D array of size V × V, initialized to 0
    
    FOR each edge (u, v) in edges:
        matrix[u][v] = 1          // or weight for weighted graph
        matrix[v][u] = 1          // omit this line for directed graph
    
    RETURN matrix
```

## Operations on Adjacency Matrix

```
Check if edge (u,v) exists:     matrix[u][v] != 0     → O(1)

Get all neighbors of u:         scan row u             → O(V)

Add edge (u,v):                 matrix[u][v] = 1       → O(1)

Remove edge (u,v):              matrix[u][v] = 0       → O(1)

Get degree of u:                count non-zero in row  → O(V)

Space used:                     V × V                  → O(V²)
```

---

[Back to TOC](../README.md) | [Next: Adjacency List →](02-adjacency-list.md)
