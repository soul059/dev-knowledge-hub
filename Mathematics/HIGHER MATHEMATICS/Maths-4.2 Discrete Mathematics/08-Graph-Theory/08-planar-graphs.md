# Chapter 8.8: Planar Graphs

[← Previous: Graph Coloring](07-graph-coloring.md) | [Back to README](../README.md) | [Next: Trees — Tree Properties →](../09-Trees/01-tree-properties.md)

---

## 📋 Chapter Overview

A **planar graph** can be drawn in the plane with no edge crossings. Planarity is crucial for circuit board layout, map coloring, and understanding graph structure. **Euler's formula** and **Kuratowski's theorem** provide the theoretical foundation.

---

## 1. Definition

A graph is **planar** if it can be drawn in the plane so that no two edges **cross** (except at shared endpoints).

Such a drawing is called a **planar embedding** or **plane graph**.

```
  K₄ is planar:              K₅ is NOT planar:
  
      a                       No matter how you draw
     /|\                      K₅, edges must cross.
    / | \                     
   b--+--c                    (Kuratowski's theorem)
    \ | /
     \|/
      d
  
  (This drawing has no crossings)
```

---

## 2. Faces (Regions)

A planar embedding divides the plane into **faces** (regions), including one **unbounded** (outer) face.

```
  Planar graph:
  
    a ---- b
    |      |
    |  f₁  |       f₁ = inner face (bounded)
    |      |       f₂ = outer face (unbounded)
    c ---- d
  
  Vertices = 4, Edges = 4, Faces = 2
```

```
  Another example:
  
      a ---- b
     /|      |\
    / |  f₁  | \
   e  | f₂   |  f
    \ |      | /
     \|      |/
      c ---- d
  
  V = 6, E = 9, F = 5
  (3 inner faces + 1 triangle + 1 outer)
```

---

## 3. Euler's Formula

For any **connected planar** graph:

$$\boxed{V - E + F = 2}$$

where $V$ = vertices, $E$ = edges, $F$ = faces (including the outer face).

### Verification

| Graph | $V$ | $E$ | $F$ | $V-E+F$ |
|-------|:---:|:---:|:---:|:--------:|
| Tree ($n$ vertices) | $n$ | $n-1$ | 1 | 2 ✓ |
| $K_4$ | 4 | 6 | 4 | 2 ✓ |
| $C_n$ | $n$ | $n$ | 2 | 2 ✓ |
| $K_{2,3}$ | 5 | 6 | 3 | 2 ✓ |
| Cube $Q_3$ | 8 | 12 | 6 | 2 ✓ |

### Generalization

For a planar graph with $c$ connected components:

$$V - E + F = 1 + c$$

---

## 4. Corollaries of Euler's Formula

### Corollary 1: Edge Bound

For a **simple connected planar** graph with $V \geq 3$:

$$\boxed{E \leq 3V - 6}$$

### Corollary 2: Triangle-Free Bound

If the graph is **simple, connected, planar, and has no triangles** ($C_3$):

$$\boxed{E \leq 2V - 4}$$

### Using These to Prove Non-Planarity

**$K_5$ is not planar:**
- $V = 5, E = 10$
- Check: $10 \leq 3(5) - 6 = 9$? **No!** $10 > 9$
- Therefore $K_5$ is **not planar** ✓

**$K_{3,3}$ is not planar:**
- $V = 6, E = 9$
- Check: $9 \leq 3(6) - 6 = 12$? Yes... but $K_{3,3}$ has no triangles!
- Triangle-free check: $9 \leq 2(6) - 4 = 8$? **No!** $9 > 8$
- Therefore $K_{3,3}$ is **not planar** ✓

---

## 5. Kuratowski's Theorem (1930)

A graph is **planar** if and only if it contains **no subdivision** of $K_5$ or $K_{3,3}$.

A **subdivision** of a graph replaces edges with paths (inserting vertices of degree 2).

```
  K₃,₃:                  A subdivision of K₃,₃:
  
  a   b   c              a     b     c
  |\ /|\ /|              |   / | \   |
  | X | X |              |  x  |  y  |
  |/ \|/ \|              | /   |   \ |
  d   e   f              d     e     f
  
  (x, y are newly inserted vertices on edges)
```

### Wagner's Theorem

Equivalent: $G$ is planar iff it has no $K_5$ or $K_{3,3}$ **minor**.

(A minor is obtained by edge contractions, not just subdivisions.)

---

## 6. Dual Graphs

For a planar graph $G$, its **dual** $G^*$ has:
- One vertex per face of $G$
- An edge between two vertices if the corresponding faces share an edge in $G$

```
  Graph G:              Dual G* (vertices = faces):
  
  a --- b                 
  |     |               f₁ --- f₂
  |     |               (f₁ = inner face, f₂ = outer face)
  c --- d
  
  G has 2 faces → G* has 2 vertices
  G has 4 edges → G* has 4 edges (some may be multi-edges)
```

Properties:
- $(G^*)^* = G$ for connected planar graphs
- $\chi(G) = \chi^*(G^*)$ where $\chi^*$ is the edge-chromatic number ... actually the proper coloring of faces of $G$ corresponds to vertex coloring of $G^*$

---

## 7. Platonic Solids as Planar Graphs

The five **Platonic solids** correspond to regular convex polyhedra, each yielding a planar graph:

| Solid | $V$ | $E$ | $F$ | $V-E+F$ | Degree |
|-------|:---:|:---:|:---:|:--------:|:------:|
| Tetrahedron | 4 | 6 | 4 | 2 | 3 |
| Cube | 8 | 12 | 6 | 2 | 3 |
| Octahedron | 6 | 12 | 8 | 2 | 4 |
| Dodecahedron | 20 | 30 | 12 | 2 | 3 |
| Icosahedron | 12 | 30 | 20 | 2 | 5 |

```
  Tetrahedron (K₄):     Cube (Q₃):
  
       *                 *-----*
      /|\               /|    /|
     / | \             * |   * |
    /  |  \            | *---|-*
   *---+---*           |/    |/
    \  |  /            *-----*
     \ | /
      \|/
       *
```

---

## 8. Planarity Testing

| Method | Time | Description |
|--------|:----:|-------------|
| Edge bound | $O(1)$ | Check $E \leq 3V - 6$ |
| Kuratowski check | Varies | Look for $K_5$ or $K_{3,3}$ subdivision |
| Hopcroft-Tarjan | $O(V)$ | Linear-time planarity test |
| Boyer-Myrvold | $O(V)$ | Practical linear-time algorithm |

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │           Planar Graph Applications               │
  │                                                    │
  │  1. Circuit Board Layout (PCB)                     │
  │     Components on one layer → planar graph         │
  │     Non-planar → need multiple layers              │
  │                                                    │
  │  2. Map Coloring                                   │
  │     Countries as faces → dual graph vertex coloring│
  │     Four Color Theorem applies                     │
  │                                                    │
  │  3. Network Design                                 │
  │     Planar networks avoid cable crossings          │
  │                                                    │
  │  4. Geographic Information Systems                 │
  │     Road networks are nearly planar                │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Planar graph | Drawable without edge crossings |
| Euler's formula | $V - E + F = 2$ |
| Edge bound | $E \leq 3V - 6$ (simple, connected) |
| Triangle-free bound | $E \leq 2V - 4$ |
| Kuratowski | Planar iff no $K_5$ or $K_{3,3}$ subdivision |
| $K_5$ | Not planar ($E = 10 > 9 = 3V-6$) |
| $K_{3,3}$ | Not planar ($E = 9 > 8 = 2V-4$) |
| Four Color Theorem | $\chi(G) \leq 4$ for planar $G$ |

---

## ❓ Quick Revision Questions

1. **Verify Euler's formula for the octahedron (6 vertices, 12 edges).**

2. **Is $K_4$ planar? Prove it using the edge bound.**

3. **A connected planar graph has 10 vertices and 15 edges. How many faces does it have?**

4. **Is the Petersen graph planar? (Hint: it contains a $K_{3,3}$ subdivision.)**

5. **What is the maximum number of edges in a simple planar graph with 8 vertices?**

6. **A planar graph has all faces as triangles (triangulation). If $V = 8$, find $E$ and $F$.**

---

[← Previous: Graph Coloring](07-graph-coloring.md) | [Back to README](../README.md) | [Next: Trees — Tree Properties →](../09-Trees/01-tree-properties.md)
