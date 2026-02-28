# DFS Properties

[← Previous: Iterative DFS](03-iterative-dfs.md) | [Back to TOC](../README.md) | [Next: DFS Tree and Backtracking →](05-dfs-tree-and-backtracking.md)

---

## Discovery and Finish Times

```
    DFS assigns two timestamps to each vertex:
    
    discovery[v] = when v is FIRST visited (pre-order)
    finish[v]    = when v is FULLY processed (post-order)
    
    Example:
                    disc  fin
       (A)──(B)     A: 1   12
        │    │       B: 2    7
       (C)  (D)     C: 8   11
        │    │       D: 3    6
       (F)  (E)     E: 4    5
                     F: 9   10
    
    Key Property: For any two vertices u, v:
    - [disc[u], fin[u]] and [disc[v], fin[v]]
      are either NESTED or DISJOINT (never partial overlap)
    
    This is called the "Parenthesis Property":
    A[ B[ D[ E[] ] ] C[ F[] ] ]
```

## Edge Classification

```
    DFS classifies every edge in the graph:
    
    ┌──────────────────────────────────────────────────────┐
    │ Edge Type    │ Description           │ Detection     │
    ├──────────────┼───────────────────────┼───────────────┤
    │ Tree Edge    │ Edge used to discover │ v is new      │
    │              │ a new vertex          │ (not visited) │
    ├──────────────┼───────────────────────┼───────────────┤
    │ Back Edge    │ Edge to an ANCESTOR   │ v is in       │
    │              │ in DFS tree           │ current path  │
    │              │ ★ INDICATES CYCLE ★   │ (GRAY)        │
    ├──────────────┼───────────────────────┼───────────────┤
    │ Forward Edge │ Edge to a DESCENDANT  │ disc[u] <     │
    │              │ (non-tree)            │ disc[v]       │
    │              │ (directed graphs only)│ (BLACK)       │
    ├──────────────┼───────────────────────┼───────────────┤
    │ Cross Edge   │ Edge to a vertex in a │ disc[u] >     │
    │              │ different subtree     │ disc[v]       │
    │              │ (directed graphs only)│ (BLACK)       │
    └──────────────┴───────────────────────┴───────────────┘
    
    ★ MOST IMPORTANT: Back Edge ⟺ Cycle exists!
```

## Edge Classification Visualization

```
    Directed graph with DFS:
    
       (A) ───→ (B) ───→ (C)
        │                  │
        ▼                  │
       (D) ←───────────────┘
        │
        ▼
       (E)

    DFS from A:
    A → B → C → D (back to A? No, to D which isn't ancestor of C)
    Actually: A → B → C → D → E
    
    Tree edges:    A→B, B→C, C→D, D→E
    Back edge:     NONE in this example (no cycle to ancestor)
    
    If C pointed back to A:
    C → A would be a BACK EDGE (cycle: A→B→C→A)
```

---

[← Previous: Iterative DFS](03-iterative-dfs.md) | [Back to TOC](../README.md) | [Next: DFS Tree and Backtracking →](05-dfs-tree-and-backtracking.md)
