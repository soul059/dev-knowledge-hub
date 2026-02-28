# DFS Tree and Backtracking

[← Previous: DFS Properties](04-dfs-properties.md) | [Back to TOC](../README.md) | [Next: DFS on a Grid →](06-dfs-on-a-grid.md)

---

## DFS Tree

```
    The tree edges from DFS form the DFS Tree:
    
    Original Graph:              DFS Tree from A:
       (A)────(B)                    (A)
        │    ╱ │                    ╱
        │   ╱  │                  (B)
        │  ╱   │                 ╱   ╲
       (C)    (D)              (D)   (C)
        │                      │      │
       (F)                    (E)    (F)
                              
    Non-tree edges (B-C, A-C) form "back edges"
    to ancestors in the DFS tree.
```

## Backtracking Visualization

```
    DFS is essentially:
    
    GO FORWARD (explore new vertex)
         │
         ▼
    Can go deeper? ──YES──→ GO FORWARD again
         │
         NO (dead end or all neighbors visited)
         │
         ▼
    BACKTRACK (return to previous vertex)
         │
         ▼
    Try next unexplored neighbor
    
    ────────────────────────────────────────
    
    Path during DFS of above graph:
    
    A ─→ B ─→ D ─→ E ─→ [backtrack] ─→ D
    ─→ [backtrack] ─→ B ─→ C ─→ F ─→ [backtrack]
    ─→ C ─→ [backtrack] ─→ B ─→ [backtrack] ─→ A
    
    The recursive call stack naturally handles
    backtracking — each return pops back to
    the previous vertex.
```

---

[← Previous: DFS Properties](04-dfs-properties.md) | [Back to TOC](../README.md) | [Next: DFS on a Grid →](06-dfs-on-a-grid.md)
