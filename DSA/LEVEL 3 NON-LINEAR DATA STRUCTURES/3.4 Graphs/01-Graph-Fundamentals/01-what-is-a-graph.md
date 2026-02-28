# What is a Graph?

[Back to TOC](../README.md) | [Next: Graph Terminology →](02-graph-terminology.md)

---

Graphs are the **most general** non-linear data structure. Unlike trees (which are hierarchical) or linked lists (which are linear), graphs can represent **arbitrary relationships** between objects.

A **graph** G is defined as a pair **(V, E)** where:
- **V** = a set of **vertices** (also called **nodes**)
- **E** = a set of **edges** (also called **links** or **arcs**)

Each edge connects two vertices, representing a relationship between them.

```
    GRAPH G = (V, E)
    ┌─────────────────────────────┐
    │                             │
    │    (A)───────(B)            │
    │     │       / │             │
    │     │      /  │             │
    │     │     /   │             │
    │    (C)───    (D)            │
    │     │         │             │
    │     │         │             │
    │    (E)───────(F)            │
    │                             │
    │  V = {A, B, C, D, E, F}    │
    │  E = {(A,B),(A,C),(B,C),   │
    │       (B,D),(C,E),(D,F),   │
    │       (E,F)}               │
    └─────────────────────────────┘
```

## Graph vs Tree vs Linked List

```
    Linked List          Tree               Graph
    (Linear)          (Hierarchical)      (Network)

    [1]→[2]→[3]          [1]           [1]───[2]
                         / \            │\   /│
                       [2] [3]          │ [3] │
                       /               [4]───[5]
                     [4]

    Each node has     Each node has     Any node can
    at most ONE       ONE parent        connect to
    successor         (except root)     ANY other node
```

**Key insight:** A tree is a special case of a graph (connected, acyclic). A linked list is a special case of a tree. Graphs are the most general form.

---

[Back to TOC](../README.md) | [Next: Graph Terminology →](02-graph-terminology.md)
