# DFS Intuition

[Back to TOC](../README.md) | [Next: Recursive DFS →](02-recursive-dfs.md)

---

## The Maze Analogy

```
    Imagine exploring a maze:
    
    1. Pick a path and go as DEEP as possible
    2. Hit a dead end? BACKTRACK to the last junction
    3. Try the next unexplored path
    4. Repeat until you've explored everything

    ┌───────────────────────┐
    │ S ─── ● ─── ● ─── ●  │    Go deep first!
    │       │              │    Then backtrack.
    │       ● ─── ●       │
    │       │              │
    │       ● (dead end)    │
    │                       │
    └───────────────────────┘
    
    This is DFS: Depth-First Search!
```

## Key Idea

```
    DFS explores as far as possible along each branch
    BEFORE backtracking.
    
    Starting from vertex A:
    
    A → B → D → (dead end) → backtrack to B
                             → E → (dead end) → backtrack to A
                             → C → F → (dead end) → done!

    ┌──────────────────────────────────────┐
    │  DFS = Go Deep, Then Backtrack       │
    │  Data Structure: STACK (LIFO)        │
    │  (or recursion = implicit stack)     │
    └──────────────────────────────────────┘
```

## DFS vs BFS: Visual Comparison

```
    Same graph, different exploration order:
    
           (A)
          ╱   ╲
        (B)   (C)
       ╱   ╲    ╲
     (D)   (E)  (F)

    BFS: A → B → C → D → E → F   (level by level)
    DFS: A → B → D → E → C → F   (branch by branch)
```

---

[Back to TOC](../README.md) | [Next: Recursive DFS →](02-recursive-dfs.md)
