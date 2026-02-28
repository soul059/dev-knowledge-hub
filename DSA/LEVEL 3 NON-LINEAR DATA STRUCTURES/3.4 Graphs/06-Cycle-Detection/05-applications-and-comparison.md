# Applications and Comparison

[← Previous: Finding the Actual Cycle](04-finding-the-actual-cycle.md) | [Back to TOC](../README.md) | [Next Unit: Topological Sort →](../07-Topological-Sort/01-what-is-topological-sort.md)

---

## Where is Cycle Detection Used?

```
    ┌──────────────────────────────────────┐
    │         Cycle Detection Uses         │
    ├──────────────────────────────────────┤
    │                                      │
    │  Deadlock Detection    (OS)          │
    │  Dependency Cycles     (build tools) │
    │  Infinite Loop Check   (compilers)   │
    │  Valid DAG Check       (scheduling)  │
    │  Linked List Cycle     (Floyd's)     │
    │  Spreadsheet Circular Refs           │
    │                                      │
    └──────────────────────────────────────┘
```

## Method Comparison

| Method | Graph Type | Time | Space | Notes |
|--------|-----------|------|-------|-------|
| **DFS + Parent Check** | Undirected | O(V+E) | O(V) | Simple, intuitive |
| **Union-Find** | Undirected | O(E·α(V)) | O(V) | Edge-by-edge, good for streaming |
| **DFS 3-Colors** | Directed | O(V+E) | O(V) | Must use colors, not parent |
| **Kahn's (in-degree)** | Directed | O(V+E) | O(V) | If topo sort fails → cycle |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Undirected DFS** | visited neighbor ≠ parent → cycle |
| **Union-Find** | same root before union → cycle |
| **Directed DFS** | GRAY neighbor → back edge → cycle |
| **3 Colors** | WHITE=new, GRAY=in-progress, BLACK=done |
| **Finding Cycle** | Parent array + reconstruct from back edge |
| **Time** | All methods: O(V + E) |

## Quick Revision Questions

1. **Why can't we use the "parent check" method for directed graphs?**
   <details><summary>Answer</summary> In directed graphs, two paths can reach the same vertex without forming a cycle (e.g., A→B→D and A→C→D). The parent check would falsely detect a cycle because D is already visited but reached via a different path, not a back edge.</details>

2. **What do the three colors WHITE, GRAY, BLACK represent in directed cycle detection?**
   <details><summary>Answer</summary> WHITE = not yet discovered, GRAY = currently being processed (in the recursion stack / current DFS path), BLACK = fully processed (all descendants explored). An edge to a GRAY vertex is a back edge indicating a cycle.</details>

3. **In Union-Find cycle detection, when does an edge create a cycle?**
   <details><summary>Answer</summary> When both endpoints of the edge already belong to the same set (have the same root). This means they're already connected, so adding another edge between them creates a cycle.</details>

4. **How do you reconstruct the actual cycle from a back edge?**
   <details><summary>Answer</summary> When a back edge (u, v) is found (v is GRAY ancestor of u), trace the parent array from u back to v. The cycle is v → ... → u → v.</details>

---

[← Previous: Finding the Actual Cycle](04-finding-the-actual-cycle.md) | [Back to TOC](../README.md) | [Next Unit: Topological Sort →](../07-Topological-Sort/01-what-is-topological-sort.md)
