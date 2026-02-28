# Cycle in Directed Graph (DFS Colors)

[← Previous: Cycle Detection Union-Find](02-cycle-undirected-union-find.md) | [Back to TOC](../README.md) | [Next: Finding the Actual Cycle →](04-finding-the-actual-cycle.md)

---

## Why Parent-Check Won't Work

```
    In a directed graph, the "parent check" method fails.
    
       (A) ──→ (B)
        │        │
        ▼        ▼
       (C) ──→ (D)
    
    A→C→D and A→B→D both reach D, but there's NO CYCLE.
    The parent-check method would incorrectly detect one.
    
    Solution: Use 3-color system!
```

## The Three Colors

```
    WHITE (0) = Not yet discovered
    GRAY  (1) = Currently being processed (in recursion stack)
    BLACK (2) = Fully processed (all descendants explored)
    
    ★ KEY RULE: 
    If DFS finds an edge to a GRAY vertex → CYCLE!
    (Gray = ancestor in current DFS path = back edge)
```

## Algorithm

```
FUNCTION hasCycle_Directed(graph, V):
    color = array of size V, all WHITE
    
    FOR vertex = 0 to V-1:
        IF color[vertex] == WHITE:
            IF DFS_Color(graph, vertex, color):
                RETURN true
    
    RETURN false

FUNCTION DFS_Color(graph, vertex, color):
    color[vertex] = GRAY                // ← entering vertex
    
    FOR each neighbor in graph[vertex]:
        IF color[neighbor] == GRAY:     // ★ Back edge = CYCLE!
            RETURN true
        IF color[neighbor] == WHITE:
            IF DFS_Color(graph, neighbor, color):
                RETURN true
        // BLACK neighbors are fully processed → safe, skip
    
    color[vertex] = BLACK               // ← leaving vertex (fully done)
    RETURN false
```

## Step-by-Step Trace

```
    Directed graph with cycle:
       (A) ──→ (B) ──→ (C)
                ▲        │
                │        ▼
               (E) ←── (D)
    
    Cycle: B → C → D → E → B
    
    DFS from A:
    ├─ color[A] = GRAY
    ├─ Neighbor B:
    │   ├─ color[B] = GRAY
    │   ├─ Neighbor C:
    │   │   ├─ color[C] = GRAY
    │   │   ├─ Neighbor D:
    │   │   │   ├─ color[D] = GRAY
    │   │   │   ├─ Neighbor E:
    │   │   │   │   ├─ color[E] = GRAY
    │   │   │   │   ├─ Neighbor B: color = GRAY
    │   │   │   │   └─ ★ CYCLE FOUND! (back edge E→B)
    │   │   │   └─ RETURN TRUE
    │   │   └─ RETURN TRUE
    │   └─ RETURN TRUE
    └─ RETURN TRUE
```

## Why Three Colors?

```
    Two colors (visited/not) can't distinguish:
    
    1. Vertex is an ANCESTOR (in current path = GRAY)
       → back edge → CYCLE
    
    2. Vertex is fully PROCESSED (BLACK)
       → cross/forward edge → NO CYCLE
    
    Example without cycle:
       A → B → D
       A → C → D    (D reached from two paths, but no cycle)
    
    When processing C→D:
    - D is BLACK (fully processed via B) → no cycle ✓
    - If we only had "visited", we'd falsely detect a cycle ✗
```

---

[← Previous: Cycle Detection Union-Find](02-cycle-undirected-union-find.md) | [Back to TOC](../README.md) | [Next: Finding the Actual Cycle →](04-finding-the-actual-cycle.md)
