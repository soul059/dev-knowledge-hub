# Build Order, All Orderings, and Advanced Examples

[← Previous: Course Schedule Problems](05-course-schedule-problems.md) | [Back to TOC](../README.md) | [Next Unit: Shortest Path Unweighted →](../08-Shortest-Path-Unweighted/01-bfs-for-shortest-path.md)

---

## Build Order Problem

```
    Same as topological sort!
    
    Given projects and dependencies:
    Projects: a, b, c, d, e, f
    Dependencies: (a,d), (f,b), (b,d), (f,a), (d,c)
    
    Meaning: d depends on a, b depends on f, etc.
    
    Graph:
       (f) → (a) → (d) → (c)
        │              
        └──→ (b) → (d)    (e) stands alone
    
    Valid build order: f, e, a, b, d, c
    (or: e, f, a, b, d, c — e can go anywhere)
```

## Finding ALL Topological Orderings

```
    Use backtracking:
    
FUNCTION allTopoOrders(graph, V):
    inDegree = calculate in-degrees
    visited = set()
    result = []
    allResults = []
    
    backtrack(graph, inDegree, visited, result, allResults, V)
    RETURN allResults

FUNCTION backtrack(graph, inDeg, visited, result, allResults, V):
    IF result.size() == V:
        allResults.add(copy of result)
        RETURN
    
    FOR vertex = 0 to V-1:
        IF vertex NOT in visited AND inDeg[vertex] == 0:
            // Choose
            visited.add(vertex)
            result.append(vertex)
            FOR each neighbor in graph[vertex]:
                inDeg[neighbor] -= 1
            
            // Explore
            backtrack(graph, inDeg, visited, result, allResults, V)
            
            // Unchoose (backtrack)
            visited.remove(vertex)
            result.removeLast()
            FOR each neighbor in graph[vertex]:
                inDeg[neighbor] += 1
```

## Complex DAG Example

```
    Complex DAG with multiple valid orderings:
    
       (A) ──→ (C) ──→ (F)
        │        ▲
        ▼        │
       (B) ──→ (D) ──→ (G)
                 │
                 ▼
               (E)
    
    Some valid topological orderings:
    1. A, B, D, C, E, F, G
    2. A, B, D, E, C, F, G
    3. A, B, D, C, G, F, E
    4. A, B, D, C, G, E, F
    ...
    
    A must be first (no prerequisites).
    B must be after A.
    D must be after B.
    And so on...
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Topological Sort** | Linear ordering where u before v for every edge u→v |
| **Works Only On** | DAGs (Directed Acyclic Graphs) |
| **DFS Method** | Reverse postorder |
| **Kahn's Method** | BFS with in-degree reduction |
| **Cycle Detection** | Kahn's: result.size < V; DFS: back edge (gray) |
| **Time** | O(V + E) for both methods |
| **Course Schedule** | Can finish ⟺ no cycle in prerequisite graph |
| **All Orderings** | Backtracking over zero in-degree choices |

## Quick Revision Questions

1. **What is a topological sort and on what type of graph does it work?**
   <details><summary>Answer</summary> A topological sort is a linear ordering of vertices such that for every directed edge (u,v), u appears before v. It works only on DAGs (Directed Acyclic Graphs) — graphs that are directed and have no cycles.</details>

2. **How does the DFS-based topological sort work?**
   <details><summary>Answer</summary> Run DFS and add each vertex to a stack when it FINISHES (post-order). The reversed stack gives the topological order. This works because a vertex finishes after all its descendants finish.</details>

3. **How does Kahn's algorithm detect cycles?**
   <details><summary>Answer</summary> If the final result contains fewer than V vertices, a cycle exists. Vertices in a cycle never reach in-degree 0 (they depend on each other), so they're never added to the queue.</details>

4. **Can a DAG have multiple valid topological orderings?**
   <details><summary>Answer</summary> Yes. Any DAG with vertices that are not directly ordered relative to each other can have multiple valid topological orderings. For example, if A→C and B→C, both "A,B,C" and "B,A,C" are valid.</details>

5. **How do you find ALL topological orderings of a DAG?**
   <details><summary>Answer</summary> Use backtracking: at each step, try each vertex with in-degree 0, recurse, then undo the choice. This explores all possible valid orderings.</details>

---

[← Previous: Course Schedule Problems](05-course-schedule-problems.md) | [Back to TOC](../README.md) | [Next Unit: Shortest Path Unweighted →](../08-Shortest-Path-Unweighted/01-bfs-for-shortest-path.md)
