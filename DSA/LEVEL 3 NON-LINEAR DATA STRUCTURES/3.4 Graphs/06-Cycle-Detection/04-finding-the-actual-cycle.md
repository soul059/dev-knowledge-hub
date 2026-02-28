# Finding the Actual Cycle

[← Previous: Cycle in Directed Graph](03-cycle-directed-dfs-colors.md) | [Back to TOC](../README.md) | [Next: Applications and Comparison →](05-applications-and-comparison.md)

---

## Problem

Not just *detect* a cycle, but actually *find and return* the vertices in the cycle.

## Approach: Parent Array + Reconstruction

```
FUNCTION findCycle(graph, V):
    color = array of size V, all WHITE
    parent = array of size V, all -1
    
    FOR vertex = 0 to V-1:
        IF color[vertex] == WHITE:
            cycle = DFS_FindCycle(graph, vertex, color, parent)
            IF cycle is not null:
                RETURN cycle
    
    RETURN null    // no cycle

FUNCTION DFS_FindCycle(graph, vertex, color, parent):
    color[vertex] = GRAY
    
    FOR each neighbor in graph[vertex]:
        IF color[neighbor] == GRAY:
            // Found cycle! Reconstruct it.
            RETURN reconstructCycle(parent, vertex, neighbor)
        IF color[neighbor] == WHITE:
            parent[neighbor] = vertex
            result = DFS_FindCycle(graph, neighbor, color, parent)
            IF result is not null: RETURN result
    
    color[vertex] = BLACK
    RETURN null

FUNCTION reconstructCycle(parent, cycleEnd, cycleStart):
    cycle = [cycleStart]
    current = cycleEnd
    WHILE current != cycleStart:
        cycle.addFirst(current)
        current = parent[current]
    cycle.addFirst(cycleStart)        // complete the cycle
    RETURN cycle
```

## Trace

```
    Graph: A → B → C → D → B (cycle: B→C→D→B)
    
    When DFS reaches D and finds neighbor B is GRAY:
    
    cycleEnd = D, cycleStart = B
    
    Reconstruct: 
    Start with [B]
    current = D, parent[D] = C → [D, B]
    current = C, parent[C] = B → [C, D, B]
    current = B = cycleStart → stop
    Add cycleStart: [B, C, D, B]
    
    Cycle: B → C → D → B  ✓
```

---

[← Previous: Cycle in Directed Graph](03-cycle-directed-dfs-colors.md) | [Back to TOC](../README.md) | [Next: Applications and Comparison →](05-applications-and-comparison.md)
