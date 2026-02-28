# Chapter 9.4: Spanning Trees

[← Previous: Tree Traversals](03-tree-traversals.md) | [Back to README](../README.md) | [Next: Minimum Spanning Tree Algorithms →](05-minimum-spanning-tree-algorithms.md)

---

## 📋 Chapter Overview

A **spanning tree** extracts the essential connectivity from a graph using the minimum number of edges. This chapter covers construction methods, counting, and the relationship between spanning trees and graph structure.

---

## 1. Recall: Definition

A **spanning tree** $T$ of a connected graph $G = (V, E)$ is a subgraph that:
- Contains **all vertices** of $G$
- Is a **tree** (connected and acyclic)
- Has exactly $|V| - 1$ edges

```
  Graph G:              A spanning tree of G:
  
  a --- b --- e         a --- b     e
  |   / |   / |         |     |   /
  |  /  |  /  |         |     |  /
  c --- d --- f         c     d --- f
  
  G: 6 vertices, 9 edges
  T: 6 vertices, 5 edges
```

---

## 2. Existence

**Theorem:** A graph has a spanning tree **if and only if** it is connected.

**Proof sketch:**
- If $G$ is connected, repeatedly remove edges from cycles. Eventually, no cycles remain but the graph stays connected → tree.
- If $G$ has a spanning tree, then $G$ is connected (tree is connected and spans all vertices).

---

## 3. BFS Spanning Tree

Run BFS from any vertex. The **tree edges** form a spanning tree.

```
  Graph:                BFS from a:
  
  a --- b --- e           Level 0: a
  |   / |                 Level 1: b, c
  |  /  |                 Level 2: d, e
  c --- d                 
                          BFS Tree:
                             a
                            / \
                           b   c
                          / \
                         d   e
  
  Tree edges: {a,b}, {a,c}, {b,d}, {b,e}
  Non-tree edges: {b,c}, {c,d} (cross edges)
```

**Property:** BFS tree edges connect vertices at most 1 level apart. Non-tree edges are **cross edges** (connect vertices at the same level or adjacent levels).

---

## 4. DFS Spanning Tree

Run DFS from any vertex. The **tree edges** form a spanning tree.

```
  Graph:                DFS from a:
  
  a --- b --- e           Visited: a → b → c → d → (back to b) → e
  |   / |                 
  |  /  |                 DFS Tree:
  c --- d                    a
                             |
                             b
                            / \
                           c   e
                           |
                           d
  
  Tree edges: {a,b}, {b,c}, {c,d}, {b,e}
  Non-tree edge: {a,c} (back edge)
```

**Property:** Non-tree edges in DFS are **back edges** — they connect a vertex to an ancestor in the DFS tree.

---

## 5. BFS vs. DFS Spanning Trees

| Property | BFS Tree | DFS Tree |
|----------|----------|----------|
| Shape | Short and wide | Tall and narrow |
| Non-tree edges | Cross edges | Back edges |
| Path from root | **Shortest** path in $G$ | Not necessarily shortest |
| Use in algorithms | Shortest path, connectivity | Cycle detection, topological sort |

---

## 6. Number of Spanning Trees

### Cayley's Formula (Labeled Complete Graphs)

$$\tau(K_n) = n^{n-2}$$

### Kirchhoff's Matrix Tree Theorem

For any graph $G$, the number of spanning trees equals any cofactor of the **Laplacian matrix** $L$.

$$L = D - A$$

where $D$ = degree diagonal matrix, $A$ = adjacency matrix.

$\tau(G)$ = any cofactor of $L$ = det of $L$ with any one row and column deleted.

### Example

```
  Graph K₃:   a --- b
                \   |
                 \  |
                  c
  
  A = [ 0  1  1 ]     D = [ 2  0  0 ]
      [ 1  0  1 ]         [ 0  2  0 ]
      [ 1  1  0 ]         [ 0  0  2 ]
  
  L = D - A = [ 2  -1  -1 ]
              [-1   2  -1 ]
              [-1  -1   2 ]
  
  Delete row 3, col 3:
  
  M = [ 2  -1 ]
      [-1   2 ]
  
  det(M) = 4 - 1 = 3
  
  τ(K₃) = 3 ✓ (matches Cayley: 3^(3-2) = 3)
```

---

## 7. Fundamental Cycles and Cuts

### Fundamental Cycle

Adding any non-tree edge $e$ to a spanning tree $T$ creates exactly **one cycle**. This is the **fundamental cycle** of $e$ with respect to $T$.

```
  Spanning tree T:       Add edge {a,d}:
  
    a --- b                a --- b
    |     |                |   / |
    c     d                c /   d
                           (cycle: a-b-d-c-a? No...)
  
  Actually: if T has edges {a,b}, {b,d}, {a,c}
  Adding {c,d} creates cycle: c-a-b-d-c
```

The number of fundamental cycles = $|E| - |V| + 1$ = **circuit rank**.

### Fundamental Cut

Removing any tree edge from $T$ disconnects $T$ into two components. The set of edges in $G$ crossing between these components is a **fundamental cut**.

---

## 8. Spanning Trees of Specific Graphs

| Graph | $\tau(G)$ |
|-------|:---------:|
| $K_n$ | $n^{n-2}$ |
| $C_n$ | $n$ |
| $K_{m,n}$ | $m^{n-1} \cdot n^{m-1}$ |
| $P_n$ | 1 |
| $W_n$ (wheel) | $L_n - 2$ where $L_n$ is Lucas number |

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │        Spanning Tree Applications                │
  │                                                  │
  │  1. Network Design                               │
  │     Connect all nodes with minimum wiring         │
  │                                                  │
  │  2. Broadcast Routing                            │
  │     Spanning tree protocol (STP) in Ethernet      │
  │     prevents loops in switched networks           │
  │                                                  │
  │  3. Electrical Circuits                          │
  │     Kirchhoff's theorem counts configurations    │
  │                                                  │
  │  4. Cluster Analysis                             │
  │     Remove expensive MST edges → clusters         │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Spanning tree | Tree subgraph using all $n$ vertices, $n-1$ edges |
| Exists iff | Graph is connected |
| BFS tree | Shortest paths from root |
| DFS tree | Back edges → cycle detection |
| Cayley's formula | $\tau(K_n) = n^{n-2}$ |
| Matrix Tree Theorem | $\tau(G)$ = cofactor of Laplacian |
| Fundamental cycle | Unique cycle from adding non-tree edge |
| Circuit rank | $|E| - |V| + 1$ |

---

## ❓ Quick Revision Questions

1. **A connected graph has 8 vertices and 12 edges. How many edges does a spanning tree have? How many edges must be removed?**

2. **Use the Matrix Tree Theorem to find the number of spanning trees of $C_4$.**

3. **How many spanning trees does $K_{2,3}$ have?**

4. **What is the circuit rank (number of independent cycles) of a graph with 10 vertices and 15 edges?**

5. **Explain why BFS and DFS on the same graph can produce different spanning trees.**

6. **A non-tree edge $\{u,v\}$ in a DFS tree is always a back edge. Why can't it be a cross edge?**

---

[← Previous: Tree Traversals](03-tree-traversals.md) | [Back to README](../README.md) | [Next: Minimum Spanning Tree Algorithms →](05-minimum-spanning-tree-algorithms.md)
