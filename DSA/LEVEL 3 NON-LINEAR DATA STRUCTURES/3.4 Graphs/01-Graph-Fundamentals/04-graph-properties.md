# Graph Properties

[← Previous: Types of Graphs](03-types-of-graphs.md) | [Back to TOC](../README.md) | [Next: Important Formulas →](05-important-formulas.md)

---

## Property 1: Connectivity

| Property | Undirected | Directed |
|----------|-----------|----------|
| **Connected** | Path between every pair of vertices | — |
| **Strongly connected** | — | Directed path between every pair (both ways) |
| **Weakly connected** | — | Connected if we ignore edge directions |
| **Connected component** | Maximal connected subgraph | — |
| **Strongly connected component** | — | Maximal strongly connected subgraph |

## Property 2: Density

```
    SPARSE GRAPH                 DENSE GRAPH
    |E| ≈ O(V)                  |E| ≈ O(V²)
    
    (A)──(B)                     (A)──(B)
     │                           │╲  ╱│
    (C)  (D)                     │ ╲╱ │
         │                       │ ╱╲ │
    (E)──(F)                     │╱  ╲│
                                (C)──(D)

    Few edges relative           Many edges relative
    to vertices                  to vertices
    
    Prefer: Adjacency List       Prefer: Adjacency Matrix
```

**Edge count bounds** (undirected, simple graph):
- Minimum edges for connected graph: **V - 1** (a tree)
- Maximum edges: **V(V-1)/2** (complete graph)
- A graph is "sparse" when E is much closer to V than V²

## Property 3: Acyclicity

| Graph Type | Has Cycles? | Name |
|------------|------------|------|
| Tree | No | Connected acyclic undirected graph |
| Forest | No | Disjoint union of trees |
| DAG | No | Directed Acyclic Graph |
| General graph | Possibly | — |

## Property 4: Self-loops and Parallel Edges

```
    SIMPLE GRAPH                 MULTIGRAPH              GRAPH WITH SELF-LOOP
    No self-loops                Parallel edges           
    No parallel edges            allowed                 
                                                            ◄──┐
    (A)──(B)                    (A)══(B)                 (A)──(B)
     │    │                      │    │                       │
    (C)──(D)                    (C)──(D)                     (C)
                                                          Self-loop on A
```

---

[← Previous: Types of Graphs](03-types-of-graphs.md) | [Back to TOC](../README.md) | [Next: Important Formulas →](05-important-formulas.md)
