# Chapter 8.6: Trees and Spanning Trees

[← Previous: Euler and Hamiltonian Paths](05-euler-and-hamiltonian-paths.md) | [Back to README](../README.md) | [Next: Graph Coloring →](07-graph-coloring.md)

---

## 📋 Chapter Overview

A **tree** is a connected graph with no cycles — the simplest connected structure. Trees model hierarchies, search structures, and network backbones. **Spanning trees** extract the essential connectivity from any connected graph.

---

## 1. Tree Definition

A **tree** is an undirected graph that is **connected** and **acyclic** (has no cycles).

```
  Tree:                 NOT a tree:           NOT a tree:
  
      a                    a --- b             a     b
     / \                   |     |
    b   c                  c --- d             c     d
   / \                   (has a cycle)        (not connected)
  d   e
```

---

## 2. Equivalent Characterizations

For a graph $G$ with $n$ vertices, the following are **equivalent**:

| Property | Statement |
|----------|-----------|
| (i) | $G$ is a tree (connected and acyclic) |
| (ii) | $G$ is connected and has exactly $n-1$ edges |
| (iii) | $G$ is acyclic and has exactly $n-1$ edges |
| (iv) | There is a **unique path** between every pair of vertices |
| (v) | $G$ is connected, but removing any edge disconnects it |
| (vi) | $G$ is acyclic, but adding any edge creates exactly one cycle |

All six conditions are equivalent — if any one holds, all hold.

---

## 3. Properties of Trees

| Property | Value |
|----------|:-----:|
| Edges | $n - 1$ |
| Leaves (degree 1) | At least 2 (for $n \geq 2$) |
| Cycles | 0 |
| Connected components | 1 |
| Chromatic number | 2 (bipartite) |
| Every edge is a bridge | Yes |

---

## 4. Forest

A **forest** is an acyclic graph (possibly disconnected). Each connected component is a tree.

```
  Forest with 3 trees:
  
    a       d --- e       g
   / \            |      / \
  b   c           f     h   i
  
  9 vertices, 6 edges, 3 components
  Edges = vertices - components = 9 - 3 = 6 ✓
```

**Formula:** A forest with $n$ vertices and $k$ components has $n - k$ edges.

---

## 5. Spanning Trees

A **spanning tree** of a connected graph $G$ is a subgraph that:
- Contains **all vertices** of $G$
- Is a tree (connected, acyclic)

```
  Graph G:              Spanning Trees of G:
  
  a --- b               a --- b         a     b         a --- b
  |   / |               |              |   /            |     |
  |  /  |               |             |  /             |     |
  c --- d               c --- d        c --- d          c     d
  
  (4 vertices, 5 edges)  (3 edges)     (3 edges)       (3 edges)
```

### Number of Spanning Trees

| Graph | Count |
|-------|:-----:|
| $K_3$ | 3 |
| $K_4$ | 16 |
| $K_n$ | $n^{n-2}$ (Cayley's formula) |
| $C_n$ | $n$ |

---

## 6. Cayley's Formula

The number of labeled spanning trees of $K_n$ is:

$$\boxed{\tau(K_n) = n^{n-2}}$$

| $n$ | $n^{n-2}$ |
|:---:|:---------:|
| 2 | 1 |
| 3 | 3 |
| 4 | 16 |
| 5 | 125 |
| 6 | 1296 |

### Kirchhoff's Matrix Tree Theorem

For any graph $G$, the number of spanning trees equals any cofactor of the **Laplacian matrix** $L = D - A$, where:
- $D$ = diagonal matrix of degrees
- $A$ = adjacency matrix

---

## 7. Finding Spanning Trees

### BFS Spanning Tree

```
  Graph:            BFS from a:
  
  a --- b --- e     a --- b --- e
  |   / |           |         
  |  /  |           |        
  c --- d           c --- d   
  
  BFS tree edges: {a,b}, {a,c}, {b,e}, {c,d}
```

### DFS Spanning Tree

```
  Graph:            DFS from a:
  
  a --- b --- e     a --- b --- e
  |   / |           |         
  |  /  |                    
  c --- d               c --- d
  
  DFS tree edges: {a,b}, {b,e}, {b,d}, {d,c}
  (or another valid DFS order)
```

Cross edges (non-tree edges) are called **back edges** (DFS) or **cross edges** (BFS).

---

## 8. Rooted Trees

A **rooted tree** has a designated **root** vertex. This creates parent-child relationships.

```
  Rooted tree (root = a):
  
         a          Level 0
        / \
       b   c        Level 1
      / \   \
     d   e   f      Level 2
         |
         g          Level 3
  
  Height = 3
  Internal vertices: a, b, c, e
  Leaves: d, f, g
```

| Term | Definition |
|------|------------|
| **Root** | Top vertex (no parent) |
| **Parent** | Vertex directly above |
| **Child** | Vertex directly below |
| **Leaf** | Vertex with no children |
| **Internal vertex** | Vertex with at least one child |
| **Depth of $v$** | Length of path from root to $v$ |
| **Height** | Maximum depth over all vertices |
| **Level $k$** | Set of vertices at depth $k$ |

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │           Trees in Practice                      │
  │                                                  │
  │  Spanning Trees:                                 │
  │  • Network design (minimum cost backbone)        │
  │  • Broadcast routing (reach all nodes)            │
  │                                                  │
  │  Rooted Trees:                                   │
  │  • File system directories                       │
  │  • Organization charts                           │
  │  • Decision trees (ML/AI)                        │
  │  • Parse trees (compilers)                       │
  │  • Game trees (chess, tic-tac-toe)               │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Tree | Connected + acyclic |
| $n$ vertices → $n-1$ edges | Fundamental tree property |
| Forest | Acyclic graph (disjoint trees) |
| Spanning tree | Tree containing all vertices of $G$ |
| Cayley's formula | $K_n$ has $n^{n-2}$ spanning trees |
| Rooted tree | Tree with designated root → parent/child |
| Height | Max depth in a rooted tree |

---

## ❓ Quick Revision Questions

1. **A tree has 10 vertices. How many edges does it have?**

2. **How many spanning trees does $K_5$ have?**

3. **Can a tree be bipartite? Why?**

4. **A forest has 15 vertices and 11 edges. How many connected components does it have?**

5. **What is the height of a rooted binary tree with 15 vertices (best and worst case)?**

6. **Prove that every tree with $n \geq 2$ has at least 2 leaves.** (Hint: use the Handshaking Theorem.)

---

[← Previous: Euler and Hamiltonian Paths](05-euler-and-hamiltonian-paths.md) | [Back to README](../README.md) | [Next: Graph Coloring →](07-graph-coloring.md)
