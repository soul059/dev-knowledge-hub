# Strongly Connected Components

[← Previous Unit: MST Patterns](../10-Minimum-Spanning-Tree/06-common-problem-patterns.md) | [Back to TOC](../README.md) | [Next: Kosaraju's Algorithm →](02-kosarajus-algorithm.md)

---

## Definition

```
    A Strongly Connected Component (SCC) is a maximal set
    of vertices such that there is a DIRECTED PATH from
    every vertex to every other vertex in the set.
    
    ┌──────────────────────────────────────────┐
    │  Key:                                    │
    │  • Only for DIRECTED graphs              │
    │  • "Maximal" = can't add more vertices   │
    │  • Every vertex belongs to exactly 1 SCC │
    │  • A single vertex is trivially an SCC   │
    └──────────────────────────────────────────┘
```

## Visual Example

```
    Directed graph:
    
    (0) ──→ (1) ──→ (2)
     ↑      ↙        │
     └── (3)          │
                      ↓
         (4) ←── (5) ←─┘
          │       ↑
          └──────→┘
    
    SCCs:
    ┌──────────┐   ┌───┐   ┌──────────┐
    │ {0,1,3}  │──→│{2}│──→│  {4,5}   │
    └──────────┘   └───┘   └──────────┘
    
    Within {0,1,3}: 0→1→3→0 (cycle — all reachable from each other)
    Within {4,5}:   4→5→4 (cycle)
    {2} is alone:   no cycle involving 2
```

## Why SCCs Matter

```
    1. Simplify a directed graph:
       SCC → single node = "Condensation Graph" (DAG!)
    
    2. Identify mutual dependencies:
       a depends on b AND b depends on a
    
    3. Deadlock detection in resource allocation
    
    4. Social networks: clusters of mutual followers
    
    5. Prerequisite: must understand DFS deeply
```

---

[← Previous Unit: MST Patterns](../10-Minimum-Spanning-Tree/06-common-problem-patterns.md) | [Back to TOC](../README.md) | [Next: Kosaraju's Algorithm →](02-kosarajus-algorithm.md)
