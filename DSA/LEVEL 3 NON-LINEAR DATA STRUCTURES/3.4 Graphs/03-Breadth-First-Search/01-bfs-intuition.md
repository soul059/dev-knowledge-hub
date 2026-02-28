# BFS Intuition

[Back to TOC](../README.md) | [Next: BFS Pseudocode →](02-bfs-pseudocode.md)

---

## The Ripple Analogy

```
    Drop a stone in a pond.

    Time 0:        Time 1:        Time 2:        Time 3:
                                   ╭───╮          ╭─────╮
                    ╭─╮           │╭─╮│         │╭───╮│
       ●           │●│           ││●││         ││╭─╮││
                    ╰─╯           │╰─╯│         ││●│││
                                   ╰───╯         ││╰─╯││
                                                  │╰───╯│
                                                   ╰─────╯
    
    Waves spread outward LEVEL BY LEVEL.
    BFS explores a graph the same way!
```

## Key Idea

```
    BFS explores ALL vertices at distance d
    BEFORE exploring ANY vertex at distance d+1.

    Starting from vertex A:

    Level 0:    A                    ← Start here
    Level 1:    B, C, D             ← All neighbors of A
    Level 2:    E, F, G, H         ← All neighbors of B, C, D
    Level 3:    I, J                ← All neighbors of E, F, G, H
    ...

    ┌──────────────────────────────────────┐
    │  BFS = Level-by-Level Exploration    │
    │  Data Structure: QUEUE (FIFO)        │
    │  Guarantee: Shortest path (unweighted)│
    └──────────────────────────────────────┘
```

## Why a Queue?

```
    FIFO = First In, First Out
    
    Enqueue level 1 vertices first.
    They get dequeued and processed first.
    Their neighbors (level 2) are enqueued.
    Level 1 finishes → Level 2 starts → ...
    
    This GUARANTEES level-by-level exploration!
```

---

[Back to TOC](../README.md) | [Next: BFS Pseudocode →](02-bfs-pseudocode.md)
