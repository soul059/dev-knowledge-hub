# Chapter 6: When to Use DSU

## Overview

Knowing **when** to reach for Union-Find is just as important as knowing **how** to implement it. This chapter provides decision frameworks and pattern recognition for identifying DSU problems.

---

## The DSU Signal Checklist

When you see these patterns in a problem, think DSU:

```
┌─────────────────────────────────────────────────────────┐
│  ✅ STRONG SIGNALS FOR DSU                              │
├─────────────────────────────────────────────────────────┤
│  □ "Group" or "connected component" mentioned           │
│  □ Merging groups / sets dynamically                    │
│  □ "Are X and Y in the same group?"                     │
│  □ Adding edges to an undirected graph                  │
│  □ "Find the number of connected components"            │
│  □ Cycle detection in undirected graph                  │
│  □ Equivalence relation (transitive grouping)           │
│  □ Process edges/connections one by one                 │
│  □ No need to remove edges or split groups              │
└─────────────────────────────────────────────────────────┘
```

---

## When DSU is the BEST Choice

### Pattern 1: Incremental Connectivity
```
Edges are added ONE BY ONE, with connectivity queries between additions.

Example: "After adding each edge, how many components remain?"

Timeline:
  Add edge ──▶ Query ──▶ Add edge ──▶ Query ──▶ ...

Why DSU: Each Union is O(α(n)), BFS/DFS would be O(V+E) per query.
```

### Pattern 2: Grouping by Equivalence
```
Given pairs of equivalent/similar items, group them.

Example: "Given friend pairs, find all friend groups."

Input:  (A,B), (B,C), (D,E)
Output: Group1={A,B,C}, Group2={D,E}

Why DSU: Transitivity is handled automatically by Union.
```

### Pattern 3: Minimum Spanning Tree
```
Need MST of a weighted undirected graph.

Kruskal's algorithm = Edge sorting + DSU

Why DSU: Efficient cycle detection during edge selection.
```

### Pattern 4: Counting Components
```
"How many groups/components exist?"

Track: numComponents = n (initial)
Each successful Union: numComponents--
Answer: numComponents

Why DSU: O(1) to maintain count.
```

---

## When DSU is NOT the Right Choice

```
┌─────────────────────────────────────────────────────────┐
│  ❌ DO NOT USE DSU WHEN                                 │
├─────────────────────────────────────────────────────────┤
│  ✗ Need to DELETE edges / SPLIT groups                  │
│  ✗ Directed graph connectivity (use DFS/BFS/Tarjan)     │
│  ✗ Finding shortest path (use BFS/Dijkstra)             │
│  ✗ Need the actual path between nodes                   │
│  ✗ Ordered traversal needed (use BST)                   │
│  ✗ Single connectivity check (simple BFS/DFS suffices)  │
│  ✗ Dense graph operations                               │
└─────────────────────────────────────────────────────────┘
```

---

## DSU vs. Alternatives Decision Tree

```
                    Need connectivity?
                    ┌──── YES ────┐
                    │              │
              Multiple          Single
              queries?          query?
              ┌─ YES ─┐        │
              │        │      Use BFS/DFS
        Edges only   Edges    
        added?       added &
        │            removed?
       YES           │
        │           Consider
      USE DSU       Euler Tour /
                    Link-Cut Tree

              Dynamic?
              ┌── YES ──┐
              │          │
         Add only?    Add + Remove?
              │          │
           USE DSU    Use more advanced
                      structures
```

---

## Comparison Table: DSU vs. BFS/DFS

| Scenario | DSU | BFS/DFS |
|----------|-----|---------|
| Single "are connected?" query | Overkill — O(n) init | Better — O(V+E) |
| 10,000 "are connected?" queries | O(n + 10000·α(n)) | O(10000·(V+E)) |
| Edges added dynamically | Perfect | Must rebuild each time |
| Need shortest path | Cannot do | BFS gives shortest path |
| Need all nodes in a component | Need extra work | Natural with traversal |
| Cycle detection (undirected) | Efficient | Also works |
| Cycle detection (directed) | Cannot do | DFS needed |

---

## Problem Statement Keywords

```
Keywords that suggest DSU:

"connected"         → connectivity checking
"component"         → connected components
"group"             → grouping elements
"merge"             → union operation
"cluster"           → clustering
"same set"          → find operation
"equivalent"        → equivalence classes
"union"             → directly stated
"friends of friends"→ transitive closure
"cycle in undirected"→ cycle detection
"minimum spanning"  → MST (Kruskal's)
"redundant edge"    → find the cycle-creating edge
```

---

## Real Problem Examples

### ✅ Use DSU
```
1. "Given n cities and roads being built one by one, after each road,
    report the number of disconnected city groups."
    → Union for each road, track component count

2. "Given a list of accounts where each account has emails,
    merge accounts that share at least one email."
    → Union emails, group accounts by root email

3. "Find if adding a specific edge to a graph creates a cycle."
    → Find both endpoints, if same root → cycle

4. "Given n computers, initially all disconnected. Process
    connections and answer connectivity queries."
    → Classic DSU: Union for connections, Find for queries
```

### ❌ Don't Use DSU
```
1. "Find the shortest path between two nodes."
    → Use BFS (unweighted) or Dijkstra (weighted)

2. "Find strongly connected components in a directed graph."
    → Use Tarjan's or Kosaraju's algorithm

3. "Remove edges and check connectivity."
    → Process in reverse (add edges) or use advanced structures

4. "Find all paths between two nodes."
    → Use DFS with backtracking
```

---

## Decision Framework for Competitive Programming

```
Step 1: Is this a connectivity/grouping problem?
        NO  → DSU probably not needed
        YES → Continue

Step 2: Is the graph undirected?
        NO  → DSU won't work (need DFS/BFS)
        YES → Continue

Step 3: Do edges only get ADDED (never removed)?
        NO  → Consider reverse processing or Link-Cut Tree
        YES → Continue

Step 4: Are there multiple queries mixed with updates?
        NO  → Simple BFS/DFS might suffice
        YES → DSU is likely optimal ✓

Step 5: Choose optimization:
        Path compression + Union by rank/size
        → O(α(n)) per operation
```

---

## Summary Table

| Scenario | Use DSU? | Alternative |
|----------|----------|-------------|
| Dynamic edge addition + connectivity queries | ✅ Yes | BFS/DFS (slower) |
| Kruskal's MST | ✅ Yes | Prim's algorithm |
| Cycle detection (undirected) | ✅ Yes | DFS |
| Grouping equivalences | ✅ Yes | BFS on graph |
| Shortest path | ❌ No | BFS/Dijkstra |
| Edge deletion | ❌ No | Reverse + DSU |
| Directed graph connectivity | ❌ No | DFS/BFS/Tarjan |
| Single connectivity check | ⚠️ Possible | BFS simpler |

---

## Quick Revision Questions

1. **What is the strongest signal that a problem needs DSU?**
2. **Why can't DSU handle edge deletions natively?**
3. **When would BFS/DFS be preferred over DSU?**
4. **What keywords in problem statements hint at DSU?**
5. **Can DSU find the shortest path between two nodes? Why or why not?**
6. **How do you handle a problem that requires edge deletions using DSU?**

---

[← Previous: Applications Overview](05-applications-overview.md) | [Next: Unit 2 - Find Operation →](../02-Basic-Implementation/01-find-operation.md)

[← Back to Main README](../README.md)
