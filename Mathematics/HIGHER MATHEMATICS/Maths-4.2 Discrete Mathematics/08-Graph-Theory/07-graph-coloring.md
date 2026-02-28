# Chapter 8.7: Graph Coloring

[← Previous: Trees and Spanning Trees](06-trees-and-spanning-trees.md) | [Back to README](../README.md) | [Next: Planar Graphs →](08-planar-graphs.md)

---

## 📋 Chapter Overview

**Graph coloring** assigns labels (colors) to vertices such that no two adjacent vertices share the same color. It models scheduling conflicts, register allocation, map coloring, and frequency assignment.

---

## 1. Proper Vertex Coloring

A **proper $k$-coloring** of $G$ assigns one of $k$ colors to each vertex so that **no two adjacent vertices** have the same color.

```
  3-coloring of this graph:
  
    R --- B             R = Red
    |   / |             B = Blue
    |  /  |             G = Green
    G --- R
  
  Adjacent vertices always have different colors ✓
```

---

## 2. Chromatic Number $\chi(G)$

The **chromatic number** $\chi(G)$ is the **minimum** number of colors needed for a proper coloring.

| Graph | $\chi(G)$ | Reason |
|-------|:---------:|--------|
| $K_n$ | $n$ | Every pair adjacent |
| $C_n$ (even) | 2 | Bipartite |
| $C_n$ (odd) | 3 | Odd cycle needs 3 |
| Tree | 2 | Bipartite (if $n \geq 2$) |
| $K_{m,n}$ | 2 | Bipartite |
| Petersen | 3 | 3-chromatic |
| Null graph $\bar{K_n}$ | 1 | No edges |
| Wheel $W_n$ (odd $n$) | 3 | |
| Wheel $W_n$ (even $n$) | 4 | |

---

## 3. Bounds on $\chi(G)$

### Lower Bounds

$$\chi(G) \geq \omega(G)$$

where $\omega(G)$ = size of the largest **clique** (complete subgraph).

$$\chi(G) \geq \frac{n}{\alpha(G)}$$

where $\alpha(G)$ = size of the largest **independent set**.

### Upper Bounds

$$\chi(G) \leq \Delta(G) + 1$$

where $\Delta(G)$ = maximum degree. (Greedy coloring bound)

**Brooks' Theorem:** $\chi(G) \leq \Delta(G)$ unless $G$ is a complete graph or an odd cycle.

---

## 4. Greedy Coloring Algorithm

```
  Algorithm: Greedy Coloring
  ──────────────────────────────
  Input: Graph G, vertex ordering v₁, v₂, ..., vₙ
  
  For each vertex vᵢ (in order):
      Assign the SMALLEST color not used
      by any already-colored neighbor of vᵢ
  
  ──────────────────────────────
  
  Example:
  
    a --- b --- c --- d
    |         / |
    e       f   g
  
  Order: a, b, c, d, e, f, g
  
  a → color 1
  b → color 2 (neighbor a has 1)
  c → color 1 (neighbor b has 2)
  d → color 2 (neighbor c has 1)
  e → color 2 (neighbor a has 1)
  f → color 2 (neighbor c has 1)
  g → color 2 (neighbor c has 1, d has 2) → color 3?
     Wait: check neighbors of g. If g is adjacent to c and d:
     g → color 3
  
  Result: 3 colors used
```

The greedy algorithm **always** uses at most $\Delta(G) + 1$ colors, but the result depends on vertex ordering. An optimal ordering gives $\chi(G)$ colors.

---

## 5. Chromatic Polynomial $P(G, k)$

$P(G, k)$ = number of proper $k$-colorings of $G$.

| Graph | $P(G, k)$ |
|-------|:---------:|
| $K_n$ | $k(k-1)(k-2)\cdots(k-n+1) = k^{\underline{n}}$ |
| $\bar{K_n}$ (empty) | $k^n$ |
| Tree on $n$ vertices | $k(k-1)^{n-1}$ |
| $C_n$ | $(k-1)^n + (-1)^n(k-1)$ |
| Single edge | $k(k-1)$ |

### Deletion-Contraction Formula

$$P(G, k) = P(G-e, k) - P(G/e, k)$$

- $G - e$: delete edge $e$
- $G/e$: contract edge $e$ (merge its endpoints)

```
  Example: Triangle K₃
  
  P(K₃, k) = P(P₃, k) - P(K₂, k)
            = k(k-1)² - k(k-1)
            = k(k-1)[(k-1) - 1]
            = k(k-1)(k-2)
  
  P(K₃, 3) = 3·2·1 = 6 colorings ✓
```

---

## 6. Edge Coloring

A **proper edge coloring** assigns colors to edges so that no two edges sharing a vertex have the same color.

- **Chromatic index** $\chi'(G)$: minimum colors for edge coloring

**Vizing's Theorem:**

$$\Delta(G) \leq \chi'(G) \leq \Delta(G) + 1$$

Graphs are classified as:
- **Class 1:** $\chi'(G) = \Delta(G)$
- **Class 2:** $\chi'(G) = \Delta(G) + 1$

---

## 7. The Four Color Theorem

**Every planar graph can be properly colored with at most 4 colors.**

$$\chi(G) \leq 4 \quad \text{for all planar graphs } G$$

```
  Map coloring example (states/countries):
  
  ┌─────┬──────┐
  │  R  │  B   │
  ├─────┼──────┤       4 colors suffice for
  │  G  │  Y   │       any map!
  ├─────┴──┬───┤
  │   R    │ B │
  └────────┴───┘
```

- Proved in 1976 by Appel and Haken (computer-assisted).
- The **Five Color Theorem** has a simple proof.

---

## 8. Applications of Graph Coloring

```
  ┌─────────────────────────────────────────────────┐
  │         Graph Coloring Applications              │
  │                                                  │
  │  1. Exam Scheduling                              │
  │     Vertices = exams, edges = shared students    │
  │     Colors = time slots                          │
  │     χ(G) = minimum time slots needed             │
  │                                                  │
  │  2. Register Allocation (compilers)              │
  │     Vertices = variables, edges = live ranges    │
  │     Colors = CPU registers                       │
  │                                                  │
  │  3. Frequency Assignment                         │
  │     Vertices = transmitters, edges = interference│
  │     Colors = frequencies                         │
  │                                                  │
  │  4. Map Coloring                                 │
  │     Vertices = regions, edges = shared borders   │
  │     4 colors always suffice (4CT)                │
  │                                                  │
  │  5. Sudoku                                       │
  │     9-coloring of a specific graph               │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $\chi(G)$ | Minimum colors for proper vertex coloring |
| $K_n$ needs $n$ colors | Complete graph chromatic number |
| Trees / bipartite | $\chi = 2$ |
| Brooks' Theorem | $\chi(G) \leq \Delta(G)$ (unless $K_n$ or odd cycle) |
| Greedy bound | $\chi(G) \leq \Delta(G) + 1$ |
| Chromatic polynomial | $P(G,k)$ = number of $k$-colorings |
| Four Color Theorem | Planar graphs need at most 4 colors |
| Vizing's Theorem | $\Delta \leq \chi'(G) \leq \Delta + 1$ |

---

## ❓ Quick Revision Questions

1. **What is $\chi(C_7)$? $\chi(C_8)$?**

2. **Find $P(P_4, k)$ — the chromatic polynomial of a path on 4 vertices.**

3. **A graph has maximum degree 5. What is the maximum $\chi(G)$ by greedy bound?**

4. **Why is $\chi(K_{3,3}) = 2$ even though $K_{3,3}$ has 9 edges?**

5. **Use deletion-contraction to find the chromatic polynomial of $C_4$.**

6. **A company needs to schedule 6 exams. The conflict graph has $\chi = 3$. How many time slots are needed at minimum?**

---

[← Previous: Trees and Spanning Trees](06-trees-and-spanning-trees.md) | [Back to README](../README.md) | [Next: Planar Graphs →](08-planar-graphs.md)
