# Chapter 8.5: Euler and Hamiltonian Paths

[← Previous: Paths, Cycles, Connectivity](04-paths-cycles-connectivity.md) | [Back to README](../README.md) | [Next: Trees and Spanning Trees →](06-trees-and-spanning-trees.md)

---

## 📋 Chapter Overview

Two classical problems in graph theory: Can we traverse **every edge** exactly once (Euler)? Can we visit **every vertex** exactly once (Hamilton)? Despite their similarity, one has an elegant complete solution and the other remains fundamentally hard.

---

## 1. Euler Paths and Circuits

| Term | Definition |
|------|------------|
| **Euler path** | A trail that visits **every edge** exactly once |
| **Euler circuit** | An Euler path that starts and ends at the same vertex |

### Königsberg Bridge Problem (1736)

```
  Can you cross all 7 bridges exactly once?
  
       ┌── North ──┐
       │     │      │
  ═══ bridge bridge ═══
       │     │      │
   ── Island A ──  
       │     │      │
  ═══ bridge bridge ═══
       │     │      │
       └── South ──┘
            │
         bridge
            │
         East
  
  Euler modeled this as a graph:
  
     N
    /|\
   / | \
  A--+--A     (multigraph)
   \ | /
    \|/
     S
     |
     E
  
  All 4 vertices have ODD degree → No Euler path exists!
```

---

## 2. Euler's Theorems

### Theorem 1: Euler Circuit

A connected graph has an **Euler circuit** if and only if **every vertex has even degree**.

### Theorem 2: Euler Path

A connected graph has an **Euler path** (but not a circuit) if and only if it has **exactly 2 vertices of odd degree**. The path starts at one odd-degree vertex and ends at the other.

| Condition | Result |
|-----------|--------|
| 0 odd-degree vertices | Euler **circuit** exists |
| 2 odd-degree vertices | Euler **path** exists (not a circuit) |
| $> 2$ odd-degree vertices | **No** Euler path or circuit |

### For Directed Graphs

An Euler circuit exists iff:
- The graph is strongly connected, **and**
- $\deg^+(v) = \deg^-(v)$ for every vertex

An Euler path exists iff:
- The graph is weakly connected, **and**
- Exactly one vertex has $\deg^+ = \deg^- + 1$ (start)
- Exactly one vertex has $\deg^- = \deg^+ + 1$ (end)
- All other vertices have $\deg^+ = \deg^-$

---

## 3. Finding an Euler Circuit: Fleury's Algorithm

1. Start at any vertex (or an odd-degree vertex for Euler path)
2. At each step, choose an edge to traverse:
   - **Prefer non-bridge edges** (don't burn bridges unless no choice)
3. Remove the traversed edge
4. Continue until all edges are used

### Hierholzer's Algorithm (more efficient)

1. Start at any vertex, follow edges until returning to start (forming a circuit)
2. If unvisited edges remain, find a vertex on the circuit with unvisited edges
3. Start a new sub-circuit from that vertex
4. Splice the sub-circuit into the main circuit
5. Repeat until all edges are included

---

## 4. Euler Circuit Example

```
  Graph:
      a ------- b
     /|         |\
    / |         | \
   f  |         |  c
    \ |         | /
     \|         |/
      e ------- d
      All degrees = 4 (even) → Euler circuit exists
  
  One Euler circuit:
  a → b → c → d → e → f → a → e → b → d → f → ? 
  (Finding the exact circuit requires careful edge tracking)
  
  Hierholzer's approach:
  Circuit 1: a → b → d → e → a
  Remaining edges form: b → c → d, and e → f → a
  Splice in at b: a → b → c → d → b → ...
  Continue splicing until all edges used.
```

---

## 5. Hamiltonian Paths and Cycles

| Term | Definition |
|------|------------|
| **Hamiltonian path** | A path that visits **every vertex** exactly once |
| **Hamiltonian cycle** | A cycle that visits every vertex exactly once and returns to start |

```
  Hamiltonian cycle         No Hamiltonian cycle
  in this graph:            in this graph (Petersen):
  
    a --- b                 (The Petersen graph has
    |     |                  a Hamiltonian path but
    |     |                  no Hamiltonian cycle)
    d --- c
  
  Cycle: a → b → c → d → a ✓
```

### Key Difference from Euler

| | Euler | Hamilton |
|---|------|----------|
| Visits | Every **edge** once | Every **vertex** once |
| Existence test | Polynomial (check degrees) | **NP-complete** |
| Always findable? | Easy criteria | No simple criterion |

---

## 6. Sufficient Conditions for Hamiltonian Cycles

### Dirac's Theorem (1952)

If $G$ is a simple graph with $n \geq 3$ vertices and $\deg(v) \geq n/2$ for every vertex $v$, then $G$ has a Hamiltonian cycle.

### Ore's Theorem (1960)

If $G$ has $n \geq 3$ vertices and for every pair of non-adjacent vertices $u, v$:

$$\deg(u) + \deg(v) \geq n$$

then $G$ has a Hamiltonian cycle.

(Dirac's theorem is a special case of Ore's.)

### Necessary Condition

If removing $k$ vertices from $G$ creates more than $k$ connected components, then $G$ has **no** Hamiltonian cycle.

---

## 7. Comparing Euler and Hamilton

| Property | Euler Path/Circuit | Hamiltonian Path/Cycle |
|----------|:------------------:|:---------------------:|
| Covers | Every edge | Every vertex |
| Existence | Polynomial check | NP-complete |
| Characterization | Complete (degree condition) | Only sufficient conditions |
| Algorithm | Hierholzer's $O(|E|)$ | Backtracking (exponential) |
| $K_n$ ($n \geq 3$) | Euler circuit iff $n$ odd | Always Hamiltonian |
| $K_{m,n}$ | Euler circuit iff $m,n$ even | Hamiltonian cycle iff $m = n$ |

---

## 8. Traveling Salesman Problem (TSP)

Finding the **shortest** Hamiltonian cycle in a weighted complete graph.

```
  Cities:     a, b, c, d
  Distances:
       a    b    c    d
  a [  -    10   15   20 ]
  b [ 10    -    35   25 ]
  c [ 15   35    -    30 ]
  d [ 20   25   30    -  ]
  
  All Hamiltonian cycles (from a):
    a→b→c→d→a: 10+35+30+20 = 95
    a→b→d→c→a: 10+25+30+15 = 80  ← Optimal
    a→c→b→d→a: 15+35+25+20 = 95
  
  TSP is NP-hard — no known polynomial algorithm.
```

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │         Euler & Hamilton Applications             │
  │                                                   │
  │  Euler Circuits:                                  │
  │  • Mail carrier route (traverse every street)     │
  │  • DNA sequencing (de Bruijn graphs)              │
  │  • Circuit board testing (every connection)       │
  │                                                   │
  │  Hamiltonian Cycles:                              │
  │  • Traveling Salesman Problem                     │
  │  • Knight's tour on a chessboard                  │
  │  • Job scheduling / circuit layout                │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Euler path | Uses every edge exactly once |
| Euler circuit | Euler path that returns to start |
| Euler exists | 0 or 2 odd-degree vertices |
| Hamiltonian path | Visits every vertex exactly once |
| Hamiltonian cycle | Hamiltonian path returning to start |
| Dirac's Theorem | $\deg(v) \geq n/2$ → Hamiltonian cycle |
| Ore's Theorem | $\deg(u)+\deg(v) \geq n$ for non-adjacent pairs |
| TSP | Shortest Hamiltonian cycle — NP-hard |

---

## ❓ Quick Revision Questions

1. **Does $K_5$ have an Euler circuit? An Euler path?**

2. **Does $K_{3,4}$ have an Euler path? A Hamiltonian cycle?**

3. **A graph has degree sequence (4, 4, 4, 4, 4). Does it have an Euler circuit?**

4. **Apply Dirac's theorem: Does a graph on 8 vertices where every vertex has degree ≥ 4 have a Hamiltonian cycle?**

5. **How many Hamiltonian cycles does $K_n$ have? (Count distinct cycles, not considering direction.)**

6. **Find an Euler circuit in the graph: $a$-$b$, $b$-$c$, $c$-$d$, $d$-$a$, $a$-$c$, $b$-$d$.**

---

[← Previous: Paths, Cycles, Connectivity](04-paths-cycles-connectivity.md) | [Back to README](../README.md) | [Next: Trees and Spanning Trees →](06-trees-and-spanning-trees.md)
