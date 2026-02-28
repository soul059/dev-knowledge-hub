# What is Topological Sort?

[Back to TOC](../README.md) | [Next: DFS-Based Topological Sort →](02-dfs-based-topological-sort.md)

---

## Definition

```
    A TOPOLOGICAL SORT of a Directed Acyclic Graph (DAG)
    is a linear ordering of vertices such that for every
    directed edge (u, v), vertex u comes BEFORE vertex v.

    ┌────────────────────────────────────────────────┐
    │  Topological Sort works ONLY on DAGs           │
    │  (Directed Acyclic Graphs — no cycles!)        │
    │                                                │
    │  A DAG with V vertices has AT LEAST ONE        │
    │  topological ordering (often multiple valid).  │
    └────────────────────────────────────────────────┘
```

## Real-World Examples

```
    Course Prerequisites:
    ─────────────────────
    Calc I → Calc II → Calc III
    Calc I → Linear Algebra
    
    Valid orderings:
    ✓ Calc I, Calc II, Linear Algebra, Calc III
    ✓ Calc I, Linear Algebra, Calc II, Calc III
    ✗ Calc II, Calc I, ...  (violates prerequisite)
    
    Build Dependencies:
    ─────────────────────
    source.c → object.o → executable
    header.h → object.o
    
    Must compile in dependency order!
    
    Task Scheduling:
    ─────────────────────
    Get dressed: underwear → pants → belt
                 socks → shoes
                 shirt → belt → jacket
```

## Visual Example

```
    DAG:                      Topological Orders:
       (A) ──→ (B) ──→ (D)   
        │        │            ✓ A, B, C, D, E
        ▼        ▼            ✓ A, C, B, D, E
       (C) ──→ (E)            ✓ A, B, C, E, D  
                              ✓ A, C, B, E, D
    
    All satisfy: for every edge u→v, u appears before v.
    Multiple valid orderings possible!
```

---

[Back to TOC](../README.md) | [Next: DFS-Based Topological Sort →](02-dfs-based-topological-sort.md)
