# Graph Terminology

[← Previous: What is a Graph?](01-what-is-a-graph.md) | [Back to TOC](../README.md) | [Next: Types of Graphs →](03-types-of-graphs.md)

---

## Vertices and Edges

| Term | Definition | Example |
|------|-----------|---------|
| **Vertex (Node)** | A fundamental unit / entity | A person, a city, a web page |
| **Edge (Link)** | A connection between two vertices | A friendship, a road, a hyperlink |
| **Adjacent** | Two vertices connected by an edge | If edge (A,B) exists, A and B are adjacent |
| **Incident** | An edge is incident on its endpoints | Edge (A,B) is incident on A and on B |

## Degree

```
    Degree of a Vertex = Number of edges connected to it

         (B)
        / | \
       /  |  \            deg(A) = 2
     (A)  |  (D)          deg(B) = 4
       \  |  /            deg(C) = 2
        \ | /             deg(D) = 2
         (C)

    Handshaking Lemma: Σ deg(v) = 2|E|
    Here: 2 + 4 + 2 + 2 = 10 = 2 × 5 edges
```

**For directed graphs:**
- **In-degree** = number of edges coming IN to a vertex
- **Out-degree** = number of edges going OUT of a vertex

```
    Directed Graph:

         (A) ──→ (B)
          ↑       │
          │       ▼
         (D) ←── (C)

    Vertex │ In-deg │ Out-deg │ Total deg
    ───────┼────────┼─────────┼──────────
      A    │   1    │    1    │    2
      B    │   1    │    1    │    2
      C    │   1    │    1    │    2
      D    │   1    │    1    │    2
```

## Paths and Cycles

| Term | Definition |
|------|-----------|
| **Path** | A sequence of vertices where each adjacent pair is connected by an edge |
| **Simple path** | A path with no repeated vertices |
| **Cycle** | A path that starts and ends at the same vertex |
| **Simple cycle** | A cycle with no repeated vertices (except start = end) |
| **Length of path** | Number of edges in the path |

```
    Path Example:
    
     (A)──(B)──(C)──(D)
      │              │
      └──────────────┘

    Path: A → B → C → D         (length = 3)
    Cycle: A → B → C → D → A    (length = 4)
    Simple path: A → B → C      (no vertex repeated)
```

## Connected vs Disconnected

```
    CONNECTED GRAPH              DISCONNECTED GRAPH
    (one piece)                  (multiple pieces)

    (A)──(B)                     (A)──(B)    (E)──(F)
     │    │                       │    │      │
    (C)──(D)                     (C)──(D)    (G)

    Every vertex can             Not every vertex
    reach every other            can reach every other
    vertex via some path         vertex

                                 Components: {A,B,C,D} and {E,F,G}
```

---

[← Previous: What is a Graph?](01-what-is-a-graph.md) | [Back to TOC](../README.md) | [Next: Types of Graphs →](03-types-of-graphs.md)
