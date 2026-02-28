# Applications, DFS vs BFS, and Common Patterns

[← Previous: DFS on a Grid](06-dfs-on-a-grid.md) | [Back to TOC](../README.md) | [Next Unit: Graph Traversal Problems →](../05-Graph-Traversal-Problems/01-connected-components.md)

---

## Applications of DFS

```
    ┌──────────────────────────────────────────────────┐
    │              DFS Applications Map                │
    ├──────────────────────────────────────────────────┤
    │                                                  │
    │  Cycle Detection ──── Topological Sort           │
    │       │                    │                      │
    │       ▼                    ▼                      │
    │  Connected      ──── Path Finding                │
    │  Components           (any path, not shortest)   │
    │       │                                          │
    │       ▼                                          │
    │  Strongly       ──── Articulation Points         │
    │  Connected             and Bridges               │
    │  Components                                      │
    │       │                                          │
    │       ▼                                          │
    │  Maze Solving   ──── Backtracking Problems       │
    └──────────────────────────────────────────────────┘
```

## DFS vs BFS: Complete Comparison

| Feature | DFS | BFS |
|---------|-----|-----|
| **Data Structure** | Stack/Recursion | Queue |
| **Exploration** | Deep first | Level first |
| **Shortest Path** | No (unweighted) | Yes (unweighted) |
| **Space** | O(H) where H = max depth | O(W) where W = max width |
| **Cycle Detection** | Natural (back edges) | Possible but less natural |
| **Topological Sort** | Yes | Yes (Kahn's) |
| **Connected Components** | Yes | Yes |
| **Better for** | Deep structures, trees | Shortest paths, levels |
| **Risk** | Stack overflow | Memory (wide graphs) |

## Decision Flowchart

```
    ┌─────────────────────────────┐
    │  Need shortest path?        │
    └──────────┬──────────────────┘
           YES │           NO
               ▼           │
            ★ BFS ★        │
                           ▼
    ┌─────────────────────────────┐
    │  Need to explore all paths? │
    │  Backtracking problem?      │
    └──────────┬──────────────────┘
           YES │           NO
               ▼           │
            ★ DFS ★        │
                           ▼
              Either works! (Default: DFS for simplicity)
```

## Time and Space Complexity

```
    TIME:  O(V + E)   — same as BFS
    ─────────────────────────────────
    - Each vertex visited once    → O(V)
    - Each edge examined once     → O(E)
    
    SPACE: O(V) worst case
    ─────────────────────────────────
    - Visited set: O(V)
    - Stack/recursion depth: O(V) worst case
    
    ⚠ Stack Overflow Risk:
    Recursive DFS depth > ~10,000 may crash.
    Use iterative DFS for deep graphs.
```

## Common DFS Patterns

```
    Pattern 1: Return Value from DFS
    ────────────────────────────────
    DFS returns information from subtrees.
    Example: count nodes, find max depth, check property.
    
    FUNCTION dfs(node) → returns some value:
        result = BASE_CASE
        FOR each child:
            childResult = dfs(child)
            result = COMBINE(result, childResult)
        RETURN result
    
    Pattern 2: DFS with Backtracking
    ────────────────────────────────
    Undo changes when backtracking.
    
    FUNCTION dfs(state):
        IF goalReached(state): record solution
        FOR each choice:
            MAKE choice
            dfs(new state)
            UNDO choice          // ← backtrack
    
    Pattern 3: DFS for Connected Components
    ────────────────────────────────
    FOR each vertex:
        IF not visited:
            DFS(vertex)          // explore entire component
            componentCount += 1
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Data Structure** | Stack (explicit or implicit via recursion) |
| **Exploration Order** | As deep as possible, then backtrack |
| **Shortest Path** | NO (use BFS for unweighted shortest path) |
| **Time Complexity** | O(V + E) |
| **Space Complexity** | O(V) |
| **Cycle Detection** | Back edge = cycle |
| **Recursion Risk** | Stack overflow for deep graphs |
| **Key Advantage** | Natural for backtracking, topological sort |

## Quick Revision Questions

1. **What data structure does DFS use?**
   <details><summary>Answer</summary> Stack (explicit stack for iterative DFS, or implicit call stack for recursive DFS)</details>

2. **In DFS edge classification, which edge type indicates a cycle?**
   <details><summary>Answer</summary> Back edge — an edge from a vertex to one of its ancestors in the DFS tree indicates a cycle.</details>

3. **What is the Parenthesis Property in DFS?**
   <details><summary>Answer</summary> For any two vertices u and v, their discovery/finish intervals [disc[u], fin[u]] and [disc[v], fin[v]] are either completely nested or completely disjoint — they never partially overlap.</details>

4. **When should you prefer DFS over BFS?**
   <details><summary>Answer</summary> When you need cycle detection, topological sort, backtracking, exploring all paths, or when the graph is deep and narrow (DFS uses less memory).</details>

5. **Why might recursive DFS fail on a large graph?**
   <details><summary>Answer</summary> Stack overflow — each recursive call adds a frame to the call stack, and deep graphs (>10,000 levels) can exceed the stack size limit.</details>

---

[← Previous: DFS on a Grid](06-dfs-on-a-grid.md) | [Back to TOC](../README.md) | [Next Unit: Graph Traversal Problems →](../05-Graph-Traversal-Problems/01-connected-components.md)
