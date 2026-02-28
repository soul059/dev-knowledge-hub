# Edge List

[← Previous: Adjacency List](02-adjacency-list.md) | [Back to TOC](../README.md) | [Next: Comparison of Representations →](04-comparison-of-representations.md)

---

An **edge list** simply stores all edges as a list of pairs (or triples with weights).

```
       (0)────(1)          Edge List:
        │    ╱ │           
        │   ╱  │           edges = [
        │  ╱   │               (0, 1),
       (2)────(3)              (0, 2),
        │                      (1, 2),
        │                      (1, 3),
       (4)                     (2, 3),
                               (2, 4)
                           ]

    For weighted: edges = [(0,1,5), (0,2,2), (1,3,3), (2,3,1)]
```

## When to Use Edge List

- **Kruskal's MST algorithm** — needs to sort edges by weight
- When you need to iterate over ALL edges
- When the graph is very sparse
- Input format (many problems give edges as a list)

## Operations on Edge List

```
Check if edge (u,v) exists:     scan all edges           → O(E)

Get all neighbors of u:         scan all edges           → O(E)

Add edge (u,v):                 append to list           → O(1)

Remove edge (u,v):              find and remove          → O(E)

Sort by weight:                 sort the list            → O(E log E)

Space used:                     E entries                → O(E)
```

---

[← Previous: Adjacency List](02-adjacency-list.md) | [Back to TOC](../README.md) | [Next: Comparison of Representations →](04-comparison-of-representations.md)
