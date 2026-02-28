# Real-World Graph Examples

[← Previous: Important Formulas](05-important-formulas.md) | [Back to TOC](../README.md) | [Next Unit: Graph Representation →](../02-Graph-Representation/01-adjacency-matrix.md)

---

```
    ┌────────────────────┬─────────────┬──────────┬──────────┐
    │    Domain          │   Vertices  │   Edges  │ Type     │
    ├────────────────────┼─────────────┼──────────┼──────────┤
    │ Social Network     │ People      │ Friends  │ Undir.   │
    │ Twitter            │ Users       │ Follows  │ Directed │
    │ Road Map           │ Cities      │ Roads    │ Weighted │
    │ Internet           │ Routers     │ Links    │ Weighted │
    │ Web Pages          │ Pages       │ Links    │ Directed │
    │ Course Prereqs     │ Courses     │ Prereqs  │ DAG      │
    │ Airline Routes     │ Airports    │ Flights  │ W+Dir    │
    │ Electrical Circuit │ Components  │ Wires    │ Undir.   │
    │ File System        │ Dirs/Files  │ Contains │ Tree/DAG │
    │ State Machine      │ States      │ Transit. │ Directed │
    │ Chess Positions    │ Board state │ Moves    │ Directed │
    │ Molecular Struct.  │ Atoms       │ Bonds    │ Undir.   │
    └────────────────────┴─────────────┴──────────┴──────────┘
```

## Thinking in Graphs — Problem Solving Mindset

When you see a problem, ask yourself:
1. **What are the entities?** → These become vertices
2. **What relationships exist?** → These become edges
3. **Is the relationship one-way or two-way?** → Directed or undirected
4. **Does the relationship have a cost/weight?** → Weighted or unweighted
5. **What do I need to find?** → This determines the algorithm

```
    PROBLEM: "Find if you can go from city A to city B"
    
    Step 1: Cities = Vertices    ──→  V = {A, B, C, D, E}
    Step 2: Roads = Edges        ──→  E = {(A,C), (C,D), (D,B), (A,E)}
    Step 3: Two-way roads        ──→  Undirected
    Step 4: No distances needed  ──→  Unweighted
    Step 5: Reachability         ──→  BFS or DFS
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Graph definition | G = (V, E), vertices + edges |
| Directed vs Undirected | Edges have direction or not |
| Weighted vs Unweighted | Edges have costs or not |
| Degree | Number of edges touching a vertex |
| Path | Sequence of vertices connected by edges |
| Cycle | Path that returns to start |
| Connected | Every vertex reachable from every other |
| DAG | Directed graph with no cycles |
| Sparse vs Dense | Few edges (≈V) vs many edges (≈V²) |
| Complete graph | Every pair of vertices connected |
| Bipartite | Vertices split into two groups; edges only between groups |
| Handshaking lemma | Sum of all degrees = 2 × number of edges |

## Quick Revision Questions

1. **What is the maximum number of edges in a simple undirected graph with 6 vertices?**
   <details><summary>Answer</summary> 6×5/2 = 15 edges</details>

2. **A tree with 10 nodes has how many edges?**
   <details><summary>Answer</summary> 9 edges (always V - 1 for a tree)</details>

3. **In a directed graph, the sum of all in-degrees equals what?**
   <details><summary>Answer</summary> The number of edges E (and also equals the sum of all out-degrees)</details>

4. **Can a DAG have undirected edges?**
   <details><summary>Answer</summary> No. DAG = Directed Acyclic Graph. By definition, all edges are directed.</details>

5. **What type of graph models Twitter's follow system?**
   <details><summary>Answer</summary> Directed unweighted graph (A follows B does not mean B follows A)</details>

6. **A connected undirected graph with V vertices and exactly V-1 edges is always a ____?**
   <details><summary>Answer</summary> Tree</details>

---

[← Previous: Important Formulas](05-important-formulas.md) | [Back to TOC](../README.md) | [Next Unit: Graph Representation →](../02-Graph-Representation/01-adjacency-matrix.md)
