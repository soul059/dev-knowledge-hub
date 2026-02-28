# Bidirectional BFS

[← Previous: 0-1 BFS](06-zero-one-bfs.md) | [Back to TOC](../README.md) | [Next Unit: Shortest Path Weighted →](../09-Shortest-Path-Weighted/01-dijkstras-algorithm.md)

---

## Key Idea

```
    Regular BFS from source: explores a "circle" of radius d.
    Area ∝ d² (or branching_factor^d)
    
    Bidirectional BFS: explore from BOTH source AND target.
    Two smaller circles meet in the middle!
    
    Each searches radius d/2 instead of d.
    Area ∝ 2 × (d/2)² = d²/2 → roughly HALF the work!
    
    ┌──────────────────────────────────────┐
    │  For branching factor b and depth d: │
    │  Normal BFS:    O(b^d)              │
    │  Bidir BFS:     O(2 × b^(d/2))     │
    │  Massive speedup for large d!       │
    └──────────────────────────────────────┘
```

## Algorithm

```
FUNCTION bidirectionalBFS(graph, source, target):
    IF source == target: RETURN 0
    
    visitedS = {source: 0}     // source frontier + distances
    visitedT = {target: 0}     // target frontier + distances
    queueS = [source]
    queueT = [target]
    
    WHILE queueS not empty AND queueT not empty:
        // Always expand the SMALLER frontier
        IF queueS.size() <= queueT.size():
            result = expandLevel(graph, queueS, visitedS, visitedT)
        ELSE:
            result = expandLevel(graph, queueT, visitedT, visitedS)
        
        IF result != -1:
            RETURN result              // frontiers met!
    
    RETURN -1                          // no path

FUNCTION expandLevel(graph, queue, thisVisited, otherVisited):
    size = queue.size()
    FOR i = 1 to size:
        u = queue.dequeue()
        FOR each neighbor v in graph[u]:
            IF v in otherVisited:
                RETURN thisVisited[u] + 1 + otherVisited[v]  // FOUND!
            IF v NOT in thisVisited:
                thisVisited[v] = thisVisited[u] + 1
                queue.enqueue(v)
    RETURN -1
```

## Visual

```
    Source ●                                    ● Target

    Step 1:     ●─────→                ←─────●

    Step 2:     ●──────────→    ←──────────●

    Step 3:     ●───────────────────────────●
                            ↑
                     FRONTIERS MEET!
```

---

## Summary Table

| Technique | When to Use | Time |
|-----------|-------------|------|
| **Standard BFS** | Unweighted, single source | O(V + E) |
| **BFS + Distance** | Track shortest distances | O(V + E) |
| **Word Ladder** | String transformation | O(N × L²) |
| **State-Space BFS** | Min moves, implicit graph | O(States × Branching) |
| **Multi-Source BFS** | Multiple starting points | O(V + E) |
| **0-1 BFS** | Edge weights 0 or 1 only | O(V + E) |
| **Bidirectional BFS** | Known source AND target | O(b^(d/2)) |

## Quick Revision Questions

1. **Why does BFS guarantee shortest path in unweighted graphs?**
   <details><summary>Answer</summary> BFS explores all vertices at distance d before any at distance d+1. The first time a vertex is reached is via the shortest path (fewest edges).</details>

2. **What is the optimization trick in the Word Ladder problem?**
   <details><summary>Answer</summary> Instead of comparing all word pairs O(N²), group words by wildcard patterns (e.g., "h*t" matches "hot","hat"). Words sharing a pattern are neighbors. This reduces to O(N × L).</details>

3. **How does Multi-Source BFS work?**
   <details><summary>Answer</summary> Enqueue ALL source vertices at the start with distance 0. BFS then naturally finds the shortest distance from any source to every vertex, as if a virtual "super source" is connected to all real sources.</details>

4. **In 0-1 BFS, why do we add weight-0 neighbors to the front of the deque?**
   <details><summary>Answer</summary> Weight-0 edges don't increase the distance, so the neighbor is at the SAME distance level. Adding to the front ensures it's processed before higher-distance vertices at the back, maintaining the sorted order.</details>

5. **When is Bidirectional BFS most beneficial?**
   <details><summary>Answer</summary> When both source and target are known, and the branching factor is high. It reduces the search space from O(b^d) to O(2 × b^(d/2)), which is exponentially better for large d.</details>

---

[← Previous: 0-1 BFS](06-zero-one-bfs.md) | [Back to TOC](../README.md) | [Next Unit: Shortest Path Weighted →](../09-Shortest-Path-Weighted/01-dijkstras-algorithm.md)
