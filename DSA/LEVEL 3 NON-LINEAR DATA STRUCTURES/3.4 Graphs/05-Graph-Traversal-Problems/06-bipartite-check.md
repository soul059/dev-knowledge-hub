# Bipartite Check

[← Previous: Clone Graph](05-clone-graph.md) | [Back to TOC](../README.md) | [Next Unit: Cycle Detection →](../06-Cycle-Detection/01-cycle-undirected-dfs.md)

---

## Problem

Determine if a graph is **bipartite** — can vertices be divided into two sets such that every edge connects a vertex in one set to a vertex in the other set?

## Approach: BFS 2-Coloring

```
    A graph is bipartite ⟺ it is 2-colorable
    ⟺ it has NO odd-length cycles.

    Try to color the graph with 2 colors such that
    no two adjacent vertices have the same color.

FUNCTION isBipartite(graph, V):
    color = array of size V, initialized to -1  (uncolored)
    
    FOR vertex = 0 to V-1:                   // handle disconnected
        IF color[vertex] == -1:
            IF NOT BFS_Color(graph, vertex, color):
                RETURN false
    
    RETURN true

FUNCTION BFS_Color(graph, source, color):
    color[source] = 0                        // color source with 0
    queue = [source]
    
    WHILE queue is not empty:
        vertex = queue.dequeue()
        
        FOR each neighbor in graph[vertex]:
            IF color[neighbor] == -1:        // uncolored
                color[neighbor] = 1 - color[vertex]   // opposite color
                queue.enqueue(neighbor)
            ELSE IF color[neighbor] == color[vertex]:  // same color!
                RETURN false                 // NOT bipartite
    
    RETURN true
```

## Trace

```
    Bipartite Graph:              Non-Bipartite (odd cycle):
    
       (0)──(1)                      (0)──(1)
        │    │                        │  ╲  │
       (2)──(3)                      (2)──(3)

    Coloring attempt:             Coloring attempt:
    0: RED                        0: RED
    1: BLUE (neighbor of 0)       1: BLUE
    2: BLUE (neighbor of 0)       2: BLUE
    3: RED  (neighbor of 1,2)     3: RED (neighbor of 1)
    ✓ No conflicts!               BUT: 3 neighbors 0 (RED=RED!)
    → BIPARTITE                   → NOT BIPARTITE (odd cycle 0-2-3-0)
```

## Key Insight

```
    ┌──────────────────────────────────────────┐
    │  Bipartite ⟺ No Odd-Length Cycle        │
    │                                          │
    │  If an odd cycle exists, 2-coloring is   │
    │  IMPOSSIBLE (pigeonhole principle:        │
    │  odd number of vertices in cycle means    │
    │  two adjacent must share a color).       │
    └──────────────────────────────────────────┘
```

---

## Problem-Solving Strategy

```
    "Is this graph bipartite?"
    "Can we divide into two groups?"
    "Two-coloring possible?"
    → ALL the same question!
    → Use BFS 2-coloring

    ★ Check ALL components (disconnected graph)
    ★ Time: O(V + E)
    ★ Space: O(V) for color array
```

## Summary Table

| Problem | Technique | Key Trick |
|---------|-----------|-----------|
| **Connected Components** | DFS loop over all vertices | Count DFS calls |
| **Number of Islands** | Grid DFS, sink visited | Modify grid in-place |
| **Flood Fill** | DFS from start pixel | Check origColor == newColor |
| **Surrounded Regions** | DFS from borders | Reverse thinking |
| **Clone Graph** | DFS + HashMap | old→new mapping handles cycles |
| **Bipartite Check** | BFS 2-coloring | odd cycle = not bipartite |

## Quick Revision Questions

1. **How do you count connected components in an undirected graph?**
   <details><summary>Answer</summary> Loop through all vertices. For each unvisited vertex, run DFS/BFS (this explores the entire component) and increment a counter. The final count is the number of components.</details>

2. **In the "Number of Islands" problem, why can we modify the grid instead of using a visited array?**
   <details><summary>Answer</summary> By changing visited '1' cells to '0', we mark them as visited without extra space. This works because we don't need the original grid after processing.</details>

3. **What edge case must you handle in Flood Fill before starting DFS?**
   <details><summary>Answer</summary> Check if origColor == newColor. If they're the same, return immediately. Otherwise, DFS would loop infinitely since painted cells would still match origColor.</details>

4. **In Surrounded Regions, why do we start DFS from the border instead of the interior?**
   <details><summary>Answer</summary> Reverse thinking — it's easier to find UNsurrounded regions (connected to border) than surrounded ones. Border-connected 'O's are safe; everything else is captured.</details>

5. **How does the HashMap in Clone Graph prevent infinite loops?**
   <details><summary>Answer</summary> The HashMap maps original→clone. Before cloning a node, check if it's already in the map. If yes, return the existing clone instead of recursing again. This breaks cycles.</details>

6. **What is the relationship between bipartiteness and odd cycles?**
   <details><summary>Answer</summary> A graph is bipartite if and only if it contains no odd-length cycles. An odd cycle makes 2-coloring impossible because adjacent vertices in the cycle must alternate colors, but an odd number of vertices means two adjacent must share a color.</details>

---

[← Previous: Clone Graph](05-clone-graph.md) | [Back to TOC](../README.md) | [Next Unit: Cycle Detection →](../06-Cycle-Detection/01-cycle-undirected-dfs.md)
