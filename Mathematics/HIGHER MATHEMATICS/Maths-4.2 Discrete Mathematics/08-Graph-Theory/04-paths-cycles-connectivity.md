# Chapter 8.4: Paths, Cycles, and Connectivity

[← Previous: Graph Representation](03-graph-representation.md) | [Back to README](../README.md) | [Next: Euler and Hamiltonian Paths →](05-euler-and-hamiltonian-paths.md)

---

## 📋 Chapter Overview

Understanding how vertices are connected within a graph is fundamental. This chapter covers **walks**, **paths**, **cycles**, **connected components**, and algorithms for determining connectivity.

---

## 1. Walks, Trails, Paths, Circuits, Cycles

| Term | Definition | Vertices | Edges |
|------|------------|:--------:|:-----:|
| **Walk** | Sequence of alternating vertices and edges | May repeat | May repeat |
| **Trail** | Walk with no repeated **edges** | May repeat | All distinct |
| **Path** | Walk with no repeated **vertices** | All distinct | All distinct |
| **Circuit** | Closed trail (starts = ends) | May repeat | All distinct |
| **Cycle** | Closed path (starts = ends, $\geq 3$ edges) | All distinct (except first=last) | All distinct |

```
  Example graph:
  
    a --- b --- c
    |         / |
    d --- e    f
  
  Walk:    a, b, c, e, d, a, b     (repeats a, b)
  Trail:   a, b, c, f, c, e, d     (repeats vertex c, but no edge)
  Path:    a, b, c, e, d           (no repeats)
  Cycle:   a, b, c, e, d, a        (closed, no repeats except start)
```

---

## 2. Length of a Walk/Path

The **length** = number of **edges** traversed.

| Walk | Length |
|------|:------:|
| $a \to b$ | 1 |
| $a \to b \to c$ | 2 |
| $a \to b \to c \to d \to a$ | 4 |

**Shortest path** from $u$ to $v$ = path with minimum length = **distance** $d(u,v)$.

---

## 3. Connected Graphs

An undirected graph is **connected** if there exists a path between **every pair** of vertices.

```
  Connected:              Disconnected:
  
   a --- b               a --- b      e --- f
   |   / |                     |            |
   c --- d               c     d      g     h
```

### Connected Components

A **connected component** is a maximal connected subgraph.

The graph above has **3 connected components**: $\{a, b, d\}$, $\{c\}$, $\{e, f, g, h\}$  
(Wait, let me reconsider. Actually the graph would be:)

```
  Disconnected graph with 3 components:
  
  Component 1:    Component 2:    Component 3:
   a --- b           e              g --- h
         |                               |
         d                               i
```

---

## 4. Strongly and Weakly Connected (Directed Graphs)

| Type | Definition |
|------|------------|
| **Strongly connected** | Directed path exists from every vertex to every other vertex |
| **Weakly connected** | Connected when all edge directions are ignored |

```
  Strongly connected:     NOT strongly connected
                          (but weakly connected):
  
   a → b                  a → b
   ↑   ↓                      ↓
   d ← c                  d ← c
```

---

## 5. Cut Vertices and Bridges

| Term | Definition |
|------|------------|
| **Cut vertex** (articulation point) | Removing it **disconnects** the graph |
| **Bridge** (cut edge) | Removing it **disconnects** the graph |

```
  Example:
  
    a --- b --- c --- d
          |     |
          e --- f
  
  Cut vertices: b, c
  (removing b disconnects {a} from rest)
  (removing c disconnects {d} from rest)
  
  Bridges: {a,b}, {c,d}
  (removing {a,b} disconnects a)
```

### Theorems

- A vertex $v$ is a cut vertex if and only if there exist vertices $u, w \neq v$ such that every path from $u$ to $w$ passes through $v$.
- An edge $e$ is a bridge if and only if $e$ is not part of any cycle.

---

## 6. Vertex and Edge Connectivity

| Measure | Definition | Notation |
|---------|------------|:--------:|
| **Vertex connectivity** $\kappa(G)$ | Min vertices to remove to disconnect $G$ | $\kappa(G)$ |
| **Edge connectivity** $\lambda(G)$ | Min edges to remove to disconnect $G$ | $\lambda(G)$ |

**Whitney's Theorem:**

$$\kappa(G) \leq \lambda(G) \leq \delta(G)$$

where $\delta(G)$ is the minimum degree.

| Graph | $\kappa$ | $\lambda$ | $\delta$ |
|-------|:--------:|:---------:|:--------:|
| $K_n$ | $n-1$ | $n-1$ | $n-1$ |
| $C_n$ | 2 | 2 | 2 |
| $K_{3,3}$ | 3 | 3 | 3 |
| Tree ($n \geq 2$) | 1 | 1 | 1 |

---

## 7. BFS and DFS for Connectivity

### Breadth-First Search (BFS)

Explores vertices level by level from a source.

```
  BFS from vertex a:
  
  Graph:     a --- b --- e        Order visited:
             |     |              Level 0: a
             c --- d --- f        Level 1: b, c
                                  Level 2: d, e
                                  Level 3: f
  
  BFS Tree:
       a
      / \
     b   c
     |
     d --- e
     |
     f
```

### Depth-First Search (DFS)

Explores as deep as possible before backtracking.

```
  DFS from vertex a:
  
  Same graph.            One possible DFS order:
                         a → b → d → c → (backtrack) →
                         e → (backtrack) → f
  
  DFS Tree:
       a
       |
       b
       |
       d
      / \
     c   e
         |
         f
```

### Connectivity Test

Run BFS/DFS from any vertex. The graph is connected if and only if **all vertices are visited**.

---

## 8. Distance and Diameter

| Concept | Definition |
|---------|------------|
| **Distance** $d(u,v)$ | Length of shortest path from $u$ to $v$ |
| **Eccentricity** $\text{ecc}(v)$ | $\max_{u \in V} d(v,u)$ |
| **Diameter** | $\max_{v \in V} \text{ecc}(v) = \max_{u,v} d(u,v)$ |
| **Radius** | $\min_{v \in V} \text{ecc}(v)$ |
| **Center** | Set of vertices with eccentricity = radius |

### Example

```
  a --- b --- c --- d
  
  d(a,b) = 1, d(a,c) = 2, d(a,d) = 3
  
  ecc(a) = 3, ecc(b) = 2, ecc(c) = 2, ecc(d) = 3
  
  Diameter = 3, Radius = 2
  Center = {b, c}
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Walk | Any sequence of adjacent vertices |
| Path | Walk with no repeated vertices |
| Cycle | Closed path ($\geq 3$ edges) |
| Connected | Path exists between all pairs |
| Cut vertex | Removal disconnects graph |
| Bridge | Edge removal disconnects graph |
| $\kappa(G)$ | Vertex connectivity |
| $\lambda(G)$ | Edge connectivity |
| Diameter | Maximum shortest-path distance |

---

## ❓ Quick Revision Questions

1. **Is $a, b, c, b, d$ a path? A trail? A walk?**

2. **Find all cut vertices and bridges in: $a$-$b$-$c$-$d$ with extra edge $b$-$d$.**

3. **What is the diameter of $K_n$? Of $C_n$?**

4. **What is $\kappa(K_5)$? $\lambda(K_5)$?**

5. **Describe BFS traversal from any vertex of $K_4$. What are the levels?**

6. **A connected graph with $n$ vertices has at least how many edges?**

---

[← Previous: Graph Representation](03-graph-representation.md) | [Back to README](../README.md) | [Next: Euler and Hamiltonian Paths →](05-euler-and-hamiltonian-paths.md)
