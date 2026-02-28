# Chapter 3: Offline Queries

## Overview

**Offline processing** means reading ALL queries first, then answering them in an optimal order (not necessarily the input order). Union-Find combined with offline techniques solves problems that seem to require edge deletion.

---

## The Reverse Processing Trick

```
Problem: Edges are REMOVED one by one. After each removal, 
         answer a connectivity query.

Removing edges → components SPLIT (hard for DSU!)
Adding edges  → components MERGE (easy for DSU!)

TRICK: Process in REVERSE!
  - Start from the final state (all removals done)
  - Add edges back one by one
  - Use Union-Find normally
  - Reverse the answers at the end

┌────────────────────────────────────────────────────┐
│  Forward: Remove edges (hard)                      │
│  Reverse: Add edges (easy)                         │
│  Same answers, reversed order!                     │
└────────────────────────────────────────────────────┘
```

---

## Example: Reverse Processing

```
Graph: 5 nodes, edges: (0,1), (1,2), (2,3), (3,4)
Remove order: edge(1,2), edge(3,4), edge(0,1)

Forward (what happens):
  State 0: 0─1─2─3─4    (all connected)
  Remove (1,2): 0─1  2─3─4   2 components
  Remove (3,4): 0─1  2─3  4  3 components
  Remove (0,1): 0  1  2─3  4  4 components
  
  Query: "components after each removal?"
  Answer: [2, 3, 4]

Reverse (how we solve it):
  Start: edges remaining = {(2,3)} 
         (all removed edges are gone)
  State: 0  1  2─3  4    4 components

  Add (0,1) back:  0─1  2─3  4   → 3 components (answer for step 3)
  Add (3,4) back:  0─1  2─3─4    → 2 components (answer for step 2)
  Add (1,2) back:  0─1─2─3─4     → 1 component  (but this is before any removal)
  
  Reversed answers: [1, 2, 3] → but we need [2, 3, 4]
  Actually: answers from reverse = [3, 2, 1], reversed = [1, 2, 3]...
  
  Let me redo carefully:
  
  Reverse add order: (0,1), (3,4), (1,2)  [reverse of remove order]
  
  Start state: only edge (2,3) remains
  Components: {0}, {1}, {2,3}, {4} → 4 components
  
  Step 1 (reverse): Add (0,1) → 3 components  
  Step 2 (reverse): Add (3,4) → 2 components
  Step 3 (reverse): Add (1,2) → 1 component
  
  Answers collected in reverse: [3, 2, 1]
  
  The answer BEFORE each forward removal:
    Before remove 3 (0,1): the state had [answer from reverse step 0] = 4
    After remove 1 (1,2): 2
    After remove 2 (3,4): 3
    After remove 3 (0,1): 4
  
  Result in forward order: [2, 3, 4] ✓
  Which is: reverse answers starting from step 0 = [4, 3, 2, 1]
            Drop first (initial state), take rest: [3, 2, 1]
            Reverse: [1, 2, 3]... 
            
  Hmm, it's easier to just collect in reverse and reverse at end.
```

Let me provide a cleaner example:

```python
def solve_removals(n, all_edges, remove_order):
    """
    Remove edges in given order. After each removal,
    report number of components.
    """
    # Determine which edges remain after ALL removals
    removed_set = set(remove_order)
    remaining = [e for e in all_edges if e not in removed_set]
    
    # Build DSU with remaining edges
    uf = UnionFind(n)
    for u, v in remaining:
        uf.union(u, v)
    
    # Process removals in reverse (= additions)
    answers = []
    for u, v in reversed(remove_order):
        answers.append(uf.components)
        uf.union(u, v)
    
    answers.reverse()
    return answers
```

---

## Problem: Bricks Falling When Hit (LeetCode 803)

```
Grid with bricks. Hit bricks one at a time.
A brick falls if it's not connected to the top row.

Forward: Remove brick → some bricks fall (hard to compute!)
Reverse: Add brick → some bricks become stable (easy with DSU!)

Approach:
  1. Remove ALL hit bricks first
  2. Build DSU with remaining stable bricks
  3. Add hit bricks back in reverse order
  4. Count new stable bricks after each addition
```

```python
def hitBricks(grid, hits):
    rows, cols = len(grid), len(grid[0])
    
    # Remove all hits
    original = [row[:] for row in grid]
    for r, c in hits:
        grid[r][c] = 0
    
    # DSU with virtual top node
    uf = UnionFind(rows * cols + 1)
    top = rows * cols  # virtual node for top row
    
    # Build DSU from remaining bricks
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                idx = r * cols + c
                if r == 0:
                    uf.union(idx, top)
                if r > 0 and grid[r-1][c] == 1:
                    uf.union(idx, (r-1) * cols + c)
                if c > 0 and grid[r][c-1] == 1:
                    uf.union(idx, r * cols + c - 1)
    
    # Add hits back in reverse
    result = [0] * len(hits)
    for i in range(len(hits) - 1, -1, -1):
        r, c = hits[i]
        if original[r][c] == 0:
            continue  # wasn't a brick
        
        stable_before = uf.get_size(top)
        
        idx = r * cols + c
        grid[r][c] = 1
        
        if r == 0:
            uf.union(idx, top)
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                uf.union(idx, nr * cols + nc)
        
        stable_after = uf.get_size(top)
        result[i] = max(0, stable_after - stable_before - 1)
        # -1 because the brick itself doesn't count as "fallen"
    
    return result
```

---

## Offline + Sorting by Weight

```
Problem: Process edges by weight, answer queries at 
         specific weight thresholds.

"Are u and v connected using only edges of weight ≤ W?"

Approach:
  1. Sort edges by weight
  2. Sort queries by weight threshold
  3. Two-pointer: add edges as threshold increases
  4. Answer queries when their threshold is reached
```

```python
def solve_threshold_queries(n, edges, queries):
    """queries = [(u, v, max_weight, query_id), ...]"""
    edges.sort(key=lambda e: e[2])         # sort by weight
    queries.sort(key=lambda q: q[2])       # sort by threshold
    
    uf = UnionFind(n)
    answers = [False] * len(queries)
    edge_idx = 0
    
    for u, v, max_w, qid in queries:
        # Add all edges with weight ≤ max_w
        while edge_idx < len(edges) and edges[edge_idx][2] <= max_w:
            eu, ev, ew = edges[edge_idx]
            uf.union(eu, ev)
            edge_idx += 1
        
        answers[qid] = uf.connected(u, v)
    
    return answers
```

---

## Offline Binary Search

```
"What is the MINIMUM weight threshold such that u and v 
are connected?"

Binary search on weight + Union-Find:
  For threshold W, add all edges ≤ W and check connectivity.
  
Or process edges in weight order and record when 
u and v first become connected:
```

```python
def min_threshold(n, edges, u, v):
    edges.sort(key=lambda e: e[2])
    uf = UnionFind(n)
    
    for eu, ev, ew in edges:
        uf.union(eu, ev)
        if uf.connected(u, v):
            return ew  # first time connected
    
    return -1  # never connected
```

---

## Summary Table

| Technique | When to Use | Key Idea |
|-----------|-------------|----------|
| Reverse processing | Edge removals | Reverse → additions |
| Sort by weight | Threshold queries | Two-pointer + DSU |
| Binary search | Min threshold | Sort edges, process until connected |
| D&C + rollback | Edge lifetimes | Rollback DSU at boundaries |

---

## Quick Revision Questions

1. **What is the reverse processing trick and when do you use it?**
2. **How does the "Bricks Falling" problem use offline DSU?**
3. **How do you answer threshold connectivity queries offline?**
4. **What is the two-pointer technique with sorted edges and queries?**
5. **When do you need rollback DSU vs simple reverse processing?**

---

[← Previous: Union-Find with Rollback](02-union-find-with-rollback.md) | [Next: Small-to-Large Merging →](04-small-to-large-merging.md)
