# Chapter 9.5: Minimum Spanning Tree Algorithms

[← Previous: Spanning Trees](04-spanning-trees.md) | [Back to README](../README.md) | [Next: Boolean Algebra — Boolean Operations →](../10-Boolean-Algebra/01-boolean-operations.md)

---

## 📋 Chapter Overview

Given a **weighted connected graph**, a **minimum spanning tree (MST)** is a spanning tree with the smallest total edge weight. Two classic greedy algorithms — **Kruskal's** and **Prim's** — efficiently find MSTs, with applications in network design, clustering, and approximation algorithms.

---

## 1. Minimum Spanning Tree Definition

Given $G = (V, E)$ with weight function $w: E \to \mathbb{R}$, the MST is a spanning tree $T$ that minimizes:

$$w(T) = \sum_{e \in T} w(e)$$

```
  Weighted graph:
  
      a ---4--- b
     /|         |\
   1  |         | 3
   /  8         2  \
  e   |         |   c
   \  |         |  /
    6 |         | 5
     \|         |/
      d ---7--- f
  
  One spanning tree: {a-e, a-b, b-c, a-d, d-f}
  Weight = 1 + 4 + 3 + 8 + 7 = 23
  
  MST: {a-e, b-c, a-b, d-e, b-f}? Let me work through properly...
```

---

## 2. Kruskal's Algorithm

**Strategy:** Sort all edges by weight. Add the cheapest edge that **doesn't create a cycle**.

```
  KRUSKAL(G, w):
      Sort edges by weight: e₁, e₂, ..., eₘ
      T ← empty set
      for each edge eᵢ (in increasing weight order):
          if adding eᵢ to T does not create a cycle:
              add eᵢ to T
          if |T| = |V| - 1: break
      return T
```

### Worked Example

```
  Weighted graph:
  
      a ---4--- b
     /|         |\
   1  |         | 3
   /  8         2  \
  e   |         |   c
   \  |         |  /
    6 |         | 5
     \|         |/
      d ---7--- f
  
  Sorted edges:
  Edge      Weight    Action
  ─────────────────────────────
  (a,e)       1       ADD ✓
  (b,f)       2       ADD ✓
  (b,c)       3       ADD ✓
  (a,b)       4       ADD ✓
  (c,f)       5       SKIP (cycle: b-c-f-b)
  (d,e)       6       ADD ✓   ← 5th edge, tree complete!
  (d,f)       7       (skip)
  (a,d)       8       (skip)
  
  MST edges: {a-e, b-f, b-c, a-b, d-e}
  MST weight: 1 + 2 + 3 + 4 + 6 = 16
  
  MST:
      a ---4--- b
     /           |\
   1              | 3
   /           2  \
  e               c
   \          
    6         
     \        
      d       f (connected via b-f)
```

### Time Complexity

| Step | Cost |
|------|:----:|
| Sort edges | $O(E \log E)$ |
| Cycle check (Union-Find) | $O(E \cdot \alpha(V))$ ≈ $O(E)$ |
| **Total** | **$O(E \log E)$** |

---

## 3. Prim's Algorithm

**Strategy:** Grow the MST from a starting vertex. At each step, add the cheapest edge connecting a tree vertex to a non-tree vertex.

```
  PRIM(G, w, start):
      T ← {start}
      MST_edges ← empty set
      while |T| < |V|:
          find the minimum-weight edge (u, v) where
              u ∈ T and v ∉ T
          add v to T
          add (u, v) to MST_edges
      return MST_edges
```

### Worked Example (same graph, start at $a$)

```
  Step 1: T = {a}
  Candidate edges: (a,e)=1, (a,b)=4, (a,d)=8
  Add (a,e)=1.  T = {a, e}
  
  Step 2: T = {a, e}
  Candidates: (a,b)=4, (a,d)=8, (e,d)=6
  Add (a,b)=4.  T = {a, e, b}
  
  Step 3: T = {a, e, b}
  Candidates: (b,c)=3, (b,f)=2, (a,d)=8, (e,d)=6
  Add (b,f)=2.  T = {a, e, b, f}
  
  Step 4: T = {a, e, b, f}
  Candidates: (b,c)=3, (f,c)=5, (f,d)=7, (a,d)=8, (e,d)=6
  Add (b,c)=3.  T = {a, e, b, f, c}
  
  Step 5: T = {a, e, b, f, c}
  Candidates: (f,d)=7, (a,d)=8, (e,d)=6
  Add (e,d)=6.  T = {a, e, b, f, c, d}
  
  MST edges: {a-e, a-b, b-f, b-c, e-d}
  Weight: 1 + 4 + 2 + 3 + 6 = 16 ✓ (same as Kruskal)
```

### Time Complexity

| Implementation | Cost |
|---------------|:----:|
| Simple (adjacency matrix) | $O(V^2)$ |
| Binary heap + adjacency list | $O(E \log V)$ |
| Fibonacci heap | $O(E + V \log V)$ |

---

## 4. Kruskal vs. Prim

| Aspect | Kruskal | Prim |
|--------|---------|------|
| Strategy | Edge-centric (global) | Vertex-centric (local) |
| Best for | **Sparse** graphs | **Dense** graphs |
| Time | $O(E \log E)$ | $O(V^2)$ or $O(E \log V)$ |
| Data structure | Union-Find | Priority Queue |
| Starts with | All vertices isolated | Single start vertex |

---

## 5. MST Properties

### Cut Property

For any cut of $G$, the **minimum weight** edge crossing the cut belongs to some MST.

### Cycle Property

For any cycle in $G$, the **maximum weight** edge in the cycle does **not** belong to any MST (if it's unique).

### Uniqueness

If all edge weights are **distinct**, the MST is **unique**.

If some weights are equal, there may be multiple MSTs, but all have the **same total weight**.

---

## 6. Union-Find (Disjoint Set) Data Structure

Used by Kruskal's to efficiently detect cycles.

```
  Operations:
  
  MAKE-SET(x)    Create a set containing only x
  FIND(x)        Return the representative of x's set
  UNION(x, y)    Merge the sets containing x and y
  
  With union by rank + path compression:
  Nearly O(1) per operation (amortized)
  
  Cycle detection:
  Edge (u,v) creates a cycle iff FIND(u) == FIND(v)
```

---

## 7. Borůvka's Algorithm (bonus)

The oldest MST algorithm (1926):

1. Start with each vertex as its own component
2. For each component, find the **cheapest outgoing edge**
3. Add all such edges (merging components)
4. Repeat until one component remains

Time: $O(E \log V)$. Useful for **parallel** implementations.

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │          MST Applications                         │
  │                                                    │
  │  1. Network Design                                │
  │     Connect cities with minimum total cable cost   │
  │                                                    │
  │  2. Clustering                                    │
  │     Remove k-1 most expensive MST edges            │
  │     → k clusters                                   │
  │                                                    │
  │  3. Approximation Algorithms                      │
  │     TSP ≤ 2 × MST weight (for metric TSP)         │
  │                                                    │
  │  4. Image Segmentation                            │
  │     Pixels as vertices, differences as weights     │
  │                                                    │
  │  5. Maze Generation                               │
  │     Random weight MST → random spanning tree maze  │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| MST | Spanning tree with minimum total weight |
| Kruskal's | Sort edges, add cheapest non-cycle edge |
| Prim's | Grow tree, add cheapest cut edge |
| Kruskal time | $O(E \log E)$ |
| Prim time | $O(V^2)$ or $O(E \log V)$ |
| Cut property | Min-weight cut edge is in MST |
| Cycle property | Max-weight cycle edge is not in MST |
| Unique MST | Guaranteed when all weights are distinct |

---

## ❓ Quick Revision Questions

1. **Apply Kruskal's algorithm to a graph with edges: (a,b)=3, (a,c)=1, (b,c)=7, (b,d)=5, (c,d)=2, (c,e)=4, (d,e)=6. What is the MST weight?**

2. **Apply Prim's algorithm starting from vertex $a$ on the same graph. Do you get the same result?**

3. **Why does Kruskal's algorithm work? (Explain using the cut property.)**

4. **A graph has 7 vertices. Its MST has weight 25. What is the minimum possible weight of any spanning tree?**

5. **Can a maximum-weight edge in a graph ever be in the MST? When?**

6. **A connected graph has 10 vertices, 20 edges, and all edge weights distinct. How many MSTs does it have?**

---

[← Previous: Spanning Trees](04-spanning-trees.md) | [Back to README](../README.md) | [Next: Boolean Algebra — Boolean Operations →](../10-Boolean-Algebra/01-boolean-operations.md)
