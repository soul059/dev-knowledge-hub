# Chapter 5: Virtual Nodes

## Overview

**Virtual nodes** (dummy nodes) are extra nodes added to the Union-Find that represent abstract concepts like "the boundary", "the water", "connected to source", etc. They simplify problems where you need to track membership in a special group.

---

## The Concept

```
Instead of checking a complex condition for each node,
create a VIRTUAL node that represents the condition,
and union real nodes to it.

Query: "Does node x satisfy the condition?"
Answer: connected(x, virtual_node)

┌────────────────────────────────────────────────────┐
│  Virtual node = a representative for a concept     │
│  Union a real node to it = mark it as belonging    │
│  Query = check if connected to virtual node        │
└────────────────────────────────────────────────────┘
```

---

## Example: Border Connectivity

```
Problem: In a grid, which 'O' cells are connected to 
         the border? (LeetCode 130: Surrounded Regions)

Without virtual node:
  For each 'O' cell: BFS/DFS to check if it reaches border
  → O(n × m) per cell in worst case

With virtual node:
  Create virtual node BORDER
  Union all border 'O' cells with BORDER
  Union adjacent 'O' cells with each other
  
  Query: connected(cell, BORDER)? → O(α(n))

  ┌───────────────────────┐
  │  Virtual BORDER node  │
  │  /   |   \   \        │
  │ O₁  O₂  O₃  O₄       │  ← border O cells
  │      |                 │
  │      O₅               │  ← interior O connected to border
  └───────────────────────┘
  
  Interior O cells NOT connected to BORDER → surrounded!
```

---

## Implementation Pattern

```python
def solve_with_virtual(grid):
    rows, cols = len(grid), len(grid[0])
    total_cells = rows * cols
    
    # Virtual node index: one past all real cells
    VIRTUAL = total_cells
    
    uf = UnionFind(total_cells + 1)  # +1 for virtual
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 'O':
                idx = r * cols + c
                
                # Border cell → connect to virtual
                if r == 0 or r == rows-1 or c == 0 or c == cols-1:
                    uf.union(idx, VIRTUAL)
                
                # Connect to adjacent O cells
                for dr, dc in [(0,1), (1,0)]:
                    nr, nc = r + dr, c + dc
                    if nr < rows and nc < cols and grid[nr][nc] == 'O':
                        uf.union(idx, nr * cols + nc)
    
    # Check each cell
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 'O':
                if not uf.connected(r * cols + c, VIRTUAL):
                    grid[r][c] = 'X'  # surrounded!
```

---

## Multiple Virtual Nodes

```
You can have MULTIPLE virtual nodes for different concepts:

Example: Bipartite check with DSU
  Create 2 virtual nodes per real node:
  - node_same[x]: "same group as x"
  - node_diff[x]: "different group from x"
  
  Edge (u, v): union(node_same[u], node_diff[v])
               union(node_diff[u], node_same[v])
  
  Conflict: connected(node_same[u], node_diff[u]) → not bipartite!
```

---

## Application: Bipartite Check with DSU

```
Problem: Check if a graph is bipartite using Union-Find.

For each node x, create two virtual nodes:
  x_even = 2*x     (x is in color A)
  x_odd  = 2*x + 1 (x is in color B)

Edge (u, v) means u and v must be DIFFERENT colors:
  union(u_even, v_odd)   ← if u is A, v must be B
  union(u_odd, v_even)   ← if u is B, v must be A

Not bipartite if:
  connected(u_even, u_odd) for any u
  → u must be BOTH colors → contradiction!
```

```python
def isBipartite(n, edges):
    uf = UnionFind(2 * n)  # 2 virtual nodes per real node
    
    for u, v in edges:
        u_even, u_odd = 2 * u, 2 * u + 1
        v_even, v_odd = 2 * v, 2 * v + 1
        
        uf.union(u_even, v_odd)
        uf.union(u_odd, v_even)
        
        if uf.connected(u_even, u_odd):
            return False  # contradiction!
    
    return True
```

---

## Application: Enemy of My Enemy

```
Problem: N people. Some are friends, some are enemies.
"The enemy of my enemy is my friend."
Detect contradictions.

For each person x:
  x_friend = 2*x      (friend group of x)  
  x_enemy  = 2*x + 1  (enemy group of x)

Friend(A, B):
  union(A_friend, B_friend)
  union(A_enemy, B_enemy)

Enemy(A, B):
  union(A_friend, B_enemy)
  union(A_enemy, B_friend)

Contradiction: connected(A_friend, A_enemy)
  → A is both friend and enemy with someone → impossible!
```

---

## Application: Virtual Source/Sink in Network

```
Problem: Multiple sources and sinks.
"Is any source connected to any sink?"

Virtual_Source: Connect all source nodes to it
Virtual_Sink:  Connect all sink nodes to it

Query: connected(Virtual_Source, Virtual_Sink)?

  ┌──────────────────┐
  │ Virtual Source    │
  │  /  |   \        │
  │ S₁  S₂  S₃      │  ← source nodes
  │      |    |      │
  │      A    B      │  ← intermediate
  │      |           │
  │      T₁  T₂     │  ← sink nodes
  │  \  /            │
  │ Virtual Sink     │
  └──────────────────┘
```

---

## Common Virtual Node Patterns

```
┌─────────────────────────┬──────────────────────────────┐
│ Pattern                 │ Virtual Node Represents      │
├─────────────────────────┼──────────────────────────────┤
│ Border detection        │ "Connected to border"        │
│ Source/sink             │ "Connected to source/sink"   │
│ Water/land              │ "Connected to water"         │
│ Bipartite (2 per node)  │ "Same color" / "Diff color"  │
│ Enemy relations         │ "Friend group" / "Enemy grp" │
│ Top row (grid)          │ "Connected to top"           │
│ Ground (percolation)    │ "Connected to bottom"        │
└─────────────────────────┴──────────────────────────────┘

Key: Virtual nodes convert GLOBAL conditions into 
     LOCAL connectivity checks!
```

---

## Implementation Tip: Indexing

```python
# For single virtual node:
n_real = rows * cols
VIRTUAL = n_real
uf = UnionFind(n_real + 1)

# For multiple virtual nodes:
VIRTUAL_TOP = n_real
VIRTUAL_BOTTOM = n_real + 1
uf = UnionFind(n_real + 2)

# For 2 virtual nodes per real node (bipartite):
# node x → 2*x (same), 2*x+1 (opposite)
uf = UnionFind(2 * n)

# For 3 groups per node (3-coloring):
# node x → 3*x, 3*x+1, 3*x+2
uf = UnionFind(3 * n)
```

---

## Summary Table

| Technique | Virtual Nodes | DSU Size | Use Case |
|-----------|--------------|----------|----------|
| Border check | 1 | n+1 | Surrounded regions |
| Source/Sink | 2 | n+2 | Multi-source reachability |
| Bipartite | 2n | 2n | Graph coloring |
| k-coloring | kn | kn | Generalized coloring |
| Percolation | 2 | n+2 | Top-to-bottom connectivity |

---

## Quick Revision Questions

1. **What is a virtual node and why is it useful?**
2. **How do you check bipartiteness using Union-Find?**
3. **How many virtual nodes do you need for the "surrounded regions" problem?**
4. **How does the "enemy of my enemy" pattern work?**
5. **Why are virtual nodes better than checking each node individually?**

---

[← Previous: Small-to-Large Merging](04-small-to-large-merging.md) | [Next: 2-SAT Concept →](06-two-sat-concept.md)
