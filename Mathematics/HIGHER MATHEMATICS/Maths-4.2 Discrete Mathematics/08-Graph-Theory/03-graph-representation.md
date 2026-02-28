# Chapter 8.3: Graph Representation

[← Previous: Types of Graphs](02-types-of-graphs.md) | [Back to README](../README.md) | [Next: Paths, Cycles, Connectivity →](04-paths-cycles-connectivity.md)

---

## 📋 Chapter Overview

To store and manipulate graphs computationally, we need concrete representations. The three main approaches — **adjacency matrix**, **adjacency list**, and **incidence matrix** — each offer different trade-offs in space and time efficiency.

---

## 1. Adjacency Matrix

For a graph $G$ with $n$ vertices $v_1, v_2, \ldots, v_n$, the **adjacency matrix** $A$ is an $n \times n$ matrix where:

$$A[i][j] = \begin{cases} 1 & \text{if } \{v_i, v_j\} \in E \\ 0 & \text{otherwise} \end{cases}$$

### Example

```
  Graph:                Adjacency Matrix:
  
    a --- b                 a  b  c  d
    |   / |             a [ 0  1  1  0 ]
    |  /  |             b [ 1  0  1  1 ]
    c --- d             c [ 1  1  0  1 ]
                        d [ 0  1  1  0 ]
```

### Properties

| Property | Undirected | Directed |
|----------|:----------:|:--------:|
| Symmetry | $A = A^T$ (always symmetric) | Generally not symmetric |
| Diagonal | 0 (simple graph) | 0 (no self-loops) |
| Row sum | $\deg(v_i)$ | $\deg^+(v_i)$ |
| Column sum | $\deg(v_i)$ | $\deg^-(v_i)$ |
| Space | $O(n^2)$ | $O(n^2)$ |

### Powers of Adjacency Matrix

$A^k[i][j]$ = number of **walks of length $k$** from $v_i$ to $v_j$.

```
  A²[i][j] = number of walks of length 2 from vᵢ to vⱼ
  
  Example: A² for the graph above:
  
      a  b  c  d
  a [ 2  1  1  1 ]    A²[a][a] = 2 (walks: a-b-a, a-c-a)
  b [ 1  3  2  1 ]    A²[b][b] = 3 (walks: b-a-b, b-c-b, b-d-b)
  c [ 1  2  3  1 ]
  d [ 1  1  1  2 ]
```

---

## 2. Adjacency List

Each vertex stores a **list of its neighbors**.

```
  Graph:              Adjacency List:
  
    a --- b           a → [b, c]
    |   / |           b → [a, c, d]
    |  /  |           c → [a, b, d]
    c --- d           d → [b, c]
```

### For Directed Graphs

```
  Digraph:            Adjacency List:
  
  a → b               a → [b]
  a → c               b → [d]
  b → d               c → [a]
  c → a               d → [c]
  d → c
```

### Space Complexity

- Undirected: $O(|V| + 2|E|)$ — each edge stored twice
- Directed: $O(|V| + |E|)$ — each arc stored once

---

## 3. Incidence Matrix

For a graph with $n$ vertices and $m$ edges, the **incidence matrix** $B$ is $n \times m$:

$$B[i][j] = \begin{cases} 1 & \text{if vertex } v_i \text{ is an endpoint of edge } e_j \\ 0 & \text{otherwise} \end{cases}$$

### Example

```
  Graph:              Incidence Matrix:
                       e₁  e₂  e₃  e₄
    a --e₁-- b     a [  1   1   0   0 ]
    |        |     b [  1   0   0   1 ]
   e₂      e₄     c [  0   1   1   0 ]
    |        |     d [  0   0   1   1 ]
    c --e₃-- d
```

Properties:
- Each column has exactly **2** ones (for simple undirected graphs)
- Row sum = degree of that vertex
- Space: $O(|V| \times |E|)$

### For Directed Graphs

$$B[i][j] = \begin{cases} +1 & \text{if edge } e_j \text{ leaves } v_i \\ -1 & \text{if edge } e_j \text{ enters } v_i \\ 0 & \text{otherwise} \end{cases}$$

---

## 4. Comparison of Representations

| Operation | Adj. Matrix | Adj. List | Incidence Matrix |
|-----------|:-----------:|:---------:|:----------------:|
| Space | $O(n^2)$ | $O(n+m)$ | $O(nm)$ |
| Check edge $(u,v)$ | $O(1)$ | $O(\deg(u))$ | $O(m)$ |
| List neighbors of $v$ | $O(n)$ | $O(\deg(v))$ | $O(m)$ |
| Add edge | $O(1)$ | $O(1)$ | $O(nm)$ |
| Remove edge | $O(1)$ | $O(\deg)$ | $O(nm)$ |
| Add vertex | $O(n^2)^*$ | $O(1)$ | $O(nm)$ |

*Requires resizing the matrix.

### When to Use What

```
  ┌────────────────────────────────────────────────┐
  │  Dense graph (many edges)?                      │
  │  └─ YES → Adjacency Matrix                     │
  │                                                 │
  │  Sparse graph (few edges)?                      │
  │  └─ YES → Adjacency List                       │
  │                                                 │
  │  Need to reason about edges specifically?       │
  │  └─ YES → Incidence Matrix                     │
  │                                                 │
  │  Need fast edge lookup?                         │
  │  └─ YES → Adjacency Matrix or Hash Set          │
  └────────────────────────────────────────────────┘
```

---

## 5. Weighted Graphs

Replace 0/1 entries with **weights**:

```
  Weighted Graph:         Weighted Adjacency Matrix:
  
    a --5-- b                a    b    c    d
    |       |            a [ 0    5    2    ∞  ]
    2       3            b [ 5    0    ∞    3  ]
    |       |            c [ 2    ∞    0    7  ]
    c --7-- d            d [ ∞    3    7    0  ]
  
  ∞ = no edge (or use 0, -1, etc. depending on context)
```

Adjacency list for weighted graphs:

```
  a → [(b, 5), (c, 2)]
  b → [(a, 5), (d, 3)]
  c → [(a, 2), (d, 7)]
  d → [(b, 3), (c, 7)]
```

---

## 6. Converting Between Representations

```
  Adjacency Matrix → Adjacency List:
  For each row i, collect all columns j where A[i][j] ≠ 0
  
  Adjacency List → Adjacency Matrix:
  For each vertex v and each neighbor u in list[v], set A[v][u] = 1
  
  Adjacency Matrix → Incidence Matrix:
  For each (i,j) with i < j and A[i][j] = 1, create a column
  with 1s in rows i and j
```

---

## 7. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │      Choosing Graph Representations               │
  │                                                    │
  │  Social Networks (sparse, billions of nodes)       │
  │  └─ Adjacency List (saves memory)                 │
  │                                                    │
  │  Dense circuit connections                         │
  │  └─ Adjacency Matrix (fast lookup)                │
  │                                                    │
  │  Road networks with distances                      │
  │  └─ Weighted Adjacency List                        │
  │                                                    │
  │  Small graphs for algorithm demos                  │
  │  └─ Adjacency Matrix (easy to visualize)           │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Representation | Best For | Space |
|----------------|----------|:-----:|
| Adjacency Matrix | Dense graphs, fast edge check | $O(n^2)$ |
| Adjacency List | Sparse graphs, iteration | $O(n+m)$ |
| Incidence Matrix | Edge-centric analysis | $O(nm)$ |
| Weighted Matrix | Shortest-path algorithms | $O(n^2)$ |

---

## ❓ Quick Revision Questions

1. **Write the adjacency matrix for $K_4$.**

2. **Write the adjacency list for $C_5$ with vertices $\{1,2,3,4,5\}$.**

3. **What does $A^3[i][j]$ represent in the adjacency matrix?**

4. **A graph has 100 vertices and 200 edges. Which representation uses less space: adjacency matrix or adjacency list?**

5. **Write the incidence matrix for the triangle graph $K_3$.**

6. **How would you represent a multigraph (with multiple edges between two vertices) using an adjacency matrix?**

---

[← Previous: Types of Graphs](02-types-of-graphs.md) | [Back to README](../README.md) | [Next: Paths, Cycles, Connectivity →](04-paths-cycles-connectivity.md)
