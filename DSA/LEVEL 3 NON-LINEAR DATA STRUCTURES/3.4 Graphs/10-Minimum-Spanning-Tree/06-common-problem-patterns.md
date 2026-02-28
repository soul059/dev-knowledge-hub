# Common Problem Patterns

[← Previous: MST Applications](05-mst-applications.md) | [Back to TOC](../README.md) | [Next Unit: Advanced Graph Algorithms →](../11-Advanced-Graph-Algorithms/01-strongly-connected-components.md)

---

## Pattern: Connect All with Minimum Cost

```
    "N cities, build roads to connect them all.
    Costs given. Minimize total cost."
    
    → This is EXACTLY MST!
    → Use Kruskal's or Prim's.
    
    Variations:
    - Some connections already exist (pre-union those nodes)
    - Can only build certain roads (filter edges)
    - Must use at most K edges (different problem!)
```

## Pattern: K Clusters

```
    "Divide N points into K clusters to maximize
    the minimum distance between clusters."
    
    Approach:
    1. Build complete graph of N points
    2. Run Kruskal's but STOP when you have K components
       (i.e., after adding V - K edges instead of V - 1)
    3. The next edge Kruskal would add = min distance
       between clusters
    
    Why? Kruskal's adds shortest edges first.
    Stopping early leaves K separated components.
    The remaining gap = maximum inter-cluster distance!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Spanning Tree** | Connects all V vertices with V-1 edges, no cycles |
| **MST** | Spanning tree with minimum total edge weight |
| **Kruskal's** | Sort edges + Union-Find, O(E log E) |
| **Prim's** | Grow tree + Min-Heap, O((V+E) log V) |
| **Kruskal's best for** | Sparse graphs, edge list |
| **Prim's best for** | Dense graphs, adjacency list |
| **MST uniqueness** | Not unique if edge weight ties exist |
| **Cut Property** | Lightest edge across any cut is in MST |

## Quick Revision Questions

1. **How many edges does an MST of a graph with V vertices have?**
   <details><summary>Answer</summary> Exactly V - 1 edges. A spanning tree must connect all V vertices with minimum edges and no cycles, which requires exactly V - 1 edges.</details>

2. **What data structure does Kruskal's algorithm use and why?**
   <details><summary>Answer</summary> Union-Find (Disjoint Set Union). It efficiently determines whether adding an edge would create a cycle (same component) and merges components. Operations are nearly O(1) amortized.</details>

3. **How does Prim's algorithm differ from Dijkstra's?**
   <details><summary>Answer</summary> Both use a min-heap and grow a set of vertices. Dijkstra's key = shortest total distance from source. Prim's key = minimum single edge weight connecting to the MST. They solve different problems.</details>

4. **When is Kruskal's preferred over Prim's?**
   <details><summary>Answer</summary> When the graph is sparse (E ≈ V), when edges are given as a list, or when the graph may be disconnected (Kruskal's naturally produces a Minimum Spanning Forest).</details>

5. **How do you find a Maximum Spanning Tree?**
   <details><summary>Answer</summary> Negate all edge weights and find the MST, or sort edges in descending order in Kruskal's, or use a max-heap in Prim's.</details>

6. **How does Kruskal's algorithm relate to the K-clusters problem?**
   <details><summary>Answer</summary> Run Kruskal's but stop after adding V-K edges (instead of V-1). This leaves K connected components. The edges we skipped represent the gaps between clusters, and stopping early maximizes the minimum inter-cluster distance.</details>

---

[← Previous: MST Applications](05-mst-applications.md) | [Back to TOC](../README.md) | [Next Unit: Advanced Graph Algorithms →](../11-Advanced-Graph-Algorithms/01-strongly-connected-components.md)
