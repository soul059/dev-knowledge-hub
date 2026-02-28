# Step-by-Step BFS Trace

[← Previous: BFS Pseudocode](02-bfs-pseudocode.md) | [Back to TOC](../README.md) | [Next: Level-Order BFS →](04-level-order-bfs.md)

---

## Tracing BFS on a Graph

```
    Graph:            Adjacency List:
       (A)────(B)     A → [B, C]
        │    ╱ │      B → [A, C, D]
        │   ╱  │      C → [A, B, E]
        │  ╱   │      D → [B, F]
       (C)    (D)     E → [C]
        │      │      F → [D]
       (E)    (F)
    
    BFS from A:
```

| Step | Dequeue | Queue (after) | Visited | Action |
|------|---------|--------------|---------|--------|
| 0 | — | [A] | {A} | Enqueue source A |
| 1 | A | [B, C] | {A, B, C} | Visit A, enqueue B, C |
| 2 | B | [C, D] | {A, B, C, D} | Visit B, enqueue D (A,C already visited) |
| 3 | C | [D, E] | {A, B, C, D, E} | Visit C, enqueue E (A,B already visited) |
| 4 | D | [E, F] | {A, B, C, D, E, F} | Visit D, enqueue F (B already visited) |
| 5 | E | [F] | {A, B, C, D, E, F} | Visit E (C already visited) |
| 6 | F | [] | {A, B, C, D, E, F} | Visit F (D already visited) |

```
    BFS Order: A → B → C → D → E → F

    BFS Tree (edges used to discover vertices):
    
           (A)
          ╱   ╲
        (B)   (C)
         │     │
        (D)   (E)
         │
        (F)
    
    Level 0: A
    Level 1: B, C
    Level 2: D, E
    Level 3: F
```

---

[← Previous: BFS Pseudocode](02-bfs-pseudocode.md) | [Back to TOC](../README.md) | [Next: Level-Order BFS →](04-level-order-bfs.md)
