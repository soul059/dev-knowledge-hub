# Chapter 8.2: Types of Graphs

[← Previous: Basic Terminology](01-basic-terminology.md) | [Back to README](../README.md) | [Next: Graph Representation →](03-graph-representation.md)

---

## 📋 Chapter Overview

Graphs come in many varieties, each with special properties and applications. This chapter catalogs the most important types of graphs, from complete graphs to bipartite graphs, regular graphs, and more.

---

## 1. Complete Graph $K_n$

A graph where **every pair** of vertices is connected by an edge.

Properties:
- $|V| = n$
- $|E| = \binom{n}{2} = \frac{n(n-1)}{2}$
- Every vertex has degree $n-1$

```
  K₁     K₂      K₃       K₄          K₅
  
   •    •---•   •---•    •---*---•    All 10 edges
                 \ | /    |\ /\ /|    among 5 vertices
                  \|/     | X  X |
                   •      |/ \/ \|
                          •---*---•
```

| $n$ | $K_n$ Edges |
|:---:|:-----------:|
| 1 | 0 |
| 2 | 1 |
| 3 | 3 |
| 4 | 6 |
| 5 | 10 |
| 6 | 15 |
| $n$ | $n(n-1)/2$ |

---

## 2. Cycle Graph $C_n$

A single cycle through all $n$ vertices ($n \geq 3$).

- $|E| = n$
- Every vertex has degree 2

```
  C₃         C₄          C₅           C₆
  
   *          *---*       *---*        *---*
  / \         |   |      / \ / \      / |   | \
 *---*        *---*     *   *   *    *  |   |  *
                         \ / \ /      \ |   | /
                          *---*        *---*
```

---

## 3. Path Graph $P_n$

A simple path with $n$ vertices and $n-1$ edges.

```
  P₁    P₂      P₃        P₄          P₅
  
   •   •---•   •---•---•  •---•---•---•  •---•---•---•---•
```

Two endpoints have degree 1; all other vertices have degree 2.

---

## 4. Bipartite Graphs

A graph whose vertices can be divided into **two disjoint sets** $V_1$ and $V_2$ such that every edge connects a vertex in $V_1$ to one in $V_2$.

```
  Bipartite:              NOT Bipartite:
  
  V₁: a   b   c          a --- b
      |\ /|\ /|              / |
      | X | X |             /  |
      |/ \|/ \|            c---d
  V₂: d   e   f           (odd cycle)
```

### Theorem

A graph is bipartite **if and only if** it contains **no odd-length cycle**.

---

## 5. Complete Bipartite Graph $K_{m,n}$

Every vertex in $V_1$ (size $m$) is connected to every vertex in $V_2$ (size $n$).

- $|E| = m \cdot n$

```
  K₂,₃:                 K₃,₃:
  
  V₁: a     b           V₁: a   b   c
      |\   /|  \              |\ /|\ /|
      | \ / |   \             | X | X |
      | / \ |    \            |/ \|/ \|
      |/   \|     \      V₂: d   e   f
  V₂: c   d   e
  
  Edges: 2 × 3 = 6      Edges: 3 × 3 = 9
```

---

## 6. Regular Graphs

A graph where **every vertex** has the same degree $k$ is called **$k$-regular**.

| Graph | Regularity |
|-------|:----------:|
| $C_n$ | 2-regular |
| $K_n$ | $(n-1)$-regular |
| Petersen graph | 3-regular |
| Cube graph $Q_3$ | 3-regular |

```
  3-Regular (Petersen Graph - schematic):
  
       *---*
      /|   |\
     / |   | \
    *  |   |  *
    |\ |   | /|
    | \*---*/ |
    |  |   |  |
    *--*---*--*
```

---

## 7. Hypercube Graph $Q_n$

Vertices are all binary strings of length $n$. Two vertices are adjacent if they differ in exactly one bit.

```
  Q₁:  0 --- 1
  
  Q₂:  00 --- 01          Q₃:     000 ---- 001
        |      |                   /|        /|
       10 --- 11                010 ---- 011  |
                                |  100 ---|--101
                                |  /      |  /
                                110 ---- 111
```

| Property | Value |
|----------|:-----:|
| Vertices | $2^n$ |
| Edges | $n \cdot 2^{n-1}$ |
| Regular | $n$-regular |
| Bipartite | Yes (even vs. odd bit count) |

---

## 8. Wheel Graph $W_n$

A cycle $C_{n-1}$ plus a **hub** vertex connected to all cycle vertices.

```
  W₄ (wheel on 4):     W₅ (wheel on 5):
  
       b                    b
      /|\                 / | \
     / | \               /  |  \
    a--*--c             a---*---c
     \ | /               \  |  /
      \|/                 \ | /
       d                    d
  
  * = hub, edges = 2(n-1)
```

---

## 9. Complement of a Graph

The **complement** $\bar{G}$ has the same vertices as $G$, but edge $\{u,v\}$ exists in $\bar{G}$ if and only if it does **not** exist in $G$.

$$|E(\bar{G})| = \binom{n}{2} - |E(G)|$$

```
  G:                  G̅ (complement):
  
  a --- b             a     b
  |                    \   /
  c     d               c --- d
  
  G edges: {a,b}, {a,c}
  G̅ edges: {a,d}, {b,c}, {b,d}, {c,d}
```

---

## 10. Special Graph Families

| Graph | Notation | Vertices | Edges | Degree |
|-------|:--------:|:--------:|:-----:|:------:|
| Complete | $K_n$ | $n$ | $\binom{n}{2}$ | $n-1$ |
| Cycle | $C_n$ | $n$ | $n$ | $2$ |
| Path | $P_n$ | $n$ | $n-1$ | $1$ or $2$ |
| Complete bipartite | $K_{m,n}$ | $m+n$ | $mn$ | $m$ or $n$ |
| Wheel | $W_n$ | $n$ | $2(n-1)$ | $3$ or $n-1$ |
| Hypercube | $Q_n$ | $2^n$ | $n \cdot 2^{n-1}$ | $n$ |
| Star | $S_n = K_{1,n}$ | $n+1$ | $n$ | $1$ or $n$ |
| Null/Empty | $\bar{K_n}$ | $n$ | $0$ | $0$ |

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $K_n$ | Complete graph — all pairs connected |
| $C_n$ | Cycle — single loop through $n$ vertices |
| $P_n$ | Path — chain of $n$ vertices |
| Bipartite | Vertices split into two groups; edges only cross |
| $K_{m,n}$ | Complete bipartite — all cross-edges |
| $k$-Regular | Every vertex has degree $k$ |
| $Q_n$ | Hypercube — binary strings, differ by 1 bit |
| Complement $\bar{G}$ | Swap edges and non-edges |

---

## ❓ Quick Revision Questions

1. **How many edges does $K_7$ have?**

2. **Is $C_6$ bipartite? What about $C_5$?**

3. **What is the degree of every vertex in $Q_4$?**

4. **How many edges does the complement of $K_{3,3}$ have (as a graph on 6 vertices)?**

5. **Give the degree sequence of $K_{2,4}$.**

6. **Is the Petersen graph bipartite? Why or why not?**

---

[← Previous: Basic Terminology](01-basic-terminology.md) | [Back to README](../README.md) | [Next: Graph Representation →](03-graph-representation.md)
