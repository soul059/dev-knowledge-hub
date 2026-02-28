# Chapter 8.1: Basic Terminology

[← Previous: Solving Recurrences](../07-Recurrence-Relations/05-solving-recurrences.md) | [Back to README](../README.md) | [Next: Types of Graphs →](02-types-of-graphs.md)

---

## 📋 Chapter Overview

**Graph theory** studies relationships between objects using **vertices** (nodes) and **edges** (connections). Graphs model networks, social connections, maps, circuits, and countless other structures. This chapter introduces the fundamental vocabulary.

---

## 1. What Is a Graph?

A **graph** $G = (V, E)$ consists of:
- $V$ = a finite set of **vertices** (nodes)
- $E$ = a set of **edges**, each connecting two vertices

### Example

```
  G = (V, E)
  V = {a, b, c, d}
  E = {{a,b}, {a,c}, {b,c}, {c,d}}
  
       a -------- b
       |        / 
       |      /   
       |    /     
       c -------- d
```

---

## 2. Directed vs. Undirected Graphs

| Type | Edges | Notation | Example |
|------|-------|----------|---------|
| **Undirected** | Unordered pairs $\{u,v\}$ | $G = (V,E)$ | Friendships |
| **Directed** (digraph) | Ordered pairs $(u,v)$ | $D = (V,A)$ | Web links, one-way streets |

```
  Undirected:           Directed:
  
   a --- b              a --→ b
   |     |              ↑     |
   c --- d              c ←-- d
```

---

## 3. Key Terminology

| Term | Definition |
|------|------------|
| **Adjacent** | Two vertices connected by an edge |
| **Incident** | An edge is incident to its endpoints |
| **Degree** $\deg(v)$ | Number of edges incident to vertex $v$ |
| **Neighbor** | A vertex adjacent to $v$ |
| **Loop** | An edge from a vertex to itself |
| **Multiple edges** | Two or more edges between the same pair |
| **Simple graph** | No loops, no multiple edges |
| **Multigraph** | Allows multiple edges |
| **Pseudograph** | Allows loops and multiple edges |
| **Isolated vertex** | $\deg(v) = 0$ |
| **Pendant vertex** | $\deg(v) = 1$ (leaf) |

---

## 4. Degree of a Vertex

For undirected graphs:

$$\deg(v) = \text{number of edges incident to } v$$

A loop at $v$ contributes **2** to $\deg(v)$.

### Example

```
       a -------- b
       |  \       |
       |    \     |
       c      d --+-- e
                  ↺ (loop)
  
  deg(a) = 3  (edges to b, c, d)
  deg(b) = 2  (edges to a, e)
  deg(c) = 1  (edge to a)
  deg(d) = 4  (edges to a, e, + loop = 2)
  deg(e) = 2  (edges to b, d)
```

### For Directed Graphs

| Term | Definition |
|------|------------|
| **In-degree** $\deg^-(v)$ | Number of edges coming **into** $v$ |
| **Out-degree** $\deg^+(v)$ | Number of edges going **out of** $v$ |

---

## 5. Handshaking Theorem

$$\boxed{\sum_{v \in V} \deg(v) = 2|E|}$$

Every edge contributes 2 to the total degree (one at each endpoint).

### Corollary

The number of vertices with **odd** degree is always **even**.

### Example

Graph with $V = \{a,b,c,d\}$ and degrees $3, 2, 1, 4$:

$3 + 2 + 1 + 4 = 10 = 2|E| \implies |E| = 5$

Odd-degree vertices: $a$ (deg 3) and $c$ (deg 1) — count = 2 (even ✓)

### For Directed Graphs

$$\sum_{v \in V} \deg^+(v) = \sum_{v \in V} \deg^-(v) = |E|$$

---

## 6. Degree Sequence

The **degree sequence** of a graph is the list of all vertex degrees in non-increasing order.

| Graph | Degree Sequence |
|-------|:---------------:|
| Triangle ($K_3$) | $(2, 2, 2)$ |
| Path $P_4$ | $(2, 2, 1, 1)$ |
| Star $S_4$ | $(4, 1, 1, 1, 1)$ |

### Erdős–Gallai Theorem

A sequence $d_1 \geq d_2 \geq \cdots \geq d_n$ is a valid degree sequence of a simple graph if and only if:
1. $\sum d_i$ is even
2. $\sum_{i=1}^{k} d_i \leq k(k-1) + \sum_{i=k+1}^{n} \min(d_i, k)$ for all $1 \leq k \leq n$

---

## 7. Subgraphs

A graph $H = (V_H, E_H)$ is a **subgraph** of $G = (V, E)$ if $V_H \subseteq V$ and $E_H \subseteq E$.

| Type | Definition |
|------|------------|
| **Subgraph** | $V_H \subseteq V$, $E_H \subseteq E$ |
| **Induced subgraph** | Contains all edges of $G$ between vertices in $V_H$ |
| **Spanning subgraph** | $V_H = V$ (uses all vertices) |

```
  Original G:          Induced subgraph     Spanning subgraph
                       on {a, b, c}:        (remove edge b-d):
  
   a --- b             a --- b              a --- b
   |   / |             |   /                |   / 
   |  /  |             |  /                 |  /  
   c --- d             c                    c --- d
```

---

## 8. Graph Isomorphism

Two graphs $G_1$ and $G_2$ are **isomorphic** ($G_1 \cong G_2$) if there exists a bijection $f: V_1 \to V_2$ such that $\{u,v\} \in E_1 \iff \{f(u), f(v)\} \in E_2$.

### Necessary Conditions (not sufficient)

- Same number of vertices
- Same number of edges
- Same degree sequence
- Same number of cycles of each length

```
  These are isomorphic:
  
   1 --- 2          a --- b
   |     |          |     |
   4 --- 3          d --- c
  
  Mapping: 1→a, 2→b, 3→c, 4→d
```

---

## 9. Real-World Applications

```
  ┌──────────────────────────────────────────────┐
  │          Graphs Model Everything              │
  │                                               │
  │  • Social Networks (vertices = people,        │
  │    edges = friendships)                       │
  │  • Internet (vertices = routers,              │
  │    edges = links)                             │
  │  • Maps (vertices = cities,                   │
  │    edges = roads)                             │
  │  • Chemistry (vertices = atoms,               │
  │    edges = bonds)                             │
  │  • Computer Science (vertices = states,       │
  │    edges = transitions)                       │
  └──────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Graph $G = (V, E)$ | Set of vertices and edges |
| Degree $\deg(v)$ | Number of edges at $v$ |
| Handshaking Theorem | $\sum \deg(v) = 2|E|$ |
| Simple graph | No loops, no multiple edges |
| Directed graph | Edges have direction |
| Isomorphism | Structure-preserving bijection |
| Subgraph | Subset of vertices and edges |

---

## ❓ Quick Revision Questions

1. **A graph has 6 vertices with degrees 3, 3, 3, 3, 2, 2. How many edges does it have?**

2. **Is the sequence (3, 3, 3, 1) a valid degree sequence for a simple graph?**

3. **What is the maximum number of edges in a simple undirected graph with $n$ vertices?**

4. **In a directed graph with 5 vertices and 8 edges, what is the sum of all in-degrees?**

5. **Can a simple graph have all vertices with the same degree? Give an example or prove it's impossible.**

---

[← Previous: Solving Recurrences](../07-Recurrence-Relations/05-solving-recurrences.md) | [Back to README](../README.md) | [Next: Types of Graphs →](02-types-of-graphs.md)
