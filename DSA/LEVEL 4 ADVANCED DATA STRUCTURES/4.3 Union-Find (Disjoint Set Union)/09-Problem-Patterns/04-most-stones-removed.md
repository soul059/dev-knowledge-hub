# Chapter 4: Most Stones Removed (LeetCode 947)

## Overview

Given stones on a 2D plane, you can remove a stone if it shares a row or column with another stone (that hasn't been removed). Find the **maximum number of stones** that can be removed.

---

## Problem Statement

```
Input: stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]

Grid visualization:
  col: 0 1 2
  row 0: S S .
  row 1: S . S
  row 2: . S S

We can remove stones until only 1 per connected 
component remains.

Connected = share a row or column (directly or 
            transitively through other stones).

Answer = total stones - number of connected components
```

---

## Key Insight

```
┌────────────────────────────────────────────────────┐
│  max_stones_removed = total - num_components       │
│                                                     │
│  In each connected component of k stones,           │
│  we can remove k-1 stones (keep 1).                │
│                                                     │
│  Total removable = Σ(kᵢ - 1) = total - components │
└────────────────────────────────────────────────────┘

Connected means: stones that share a row or column,
  directly or through a chain of shared rows/columns.
```

---

## Approach 1: Union Stones by Index

```python
def removeStones(stones):
    n = len(stones)
    uf = UnionFind(n)
    
    # Group stones by row and by column
    from collections import defaultdict
    row_map = defaultdict(list)  # row → [stone indices]
    col_map = defaultdict(list)  # col → [stone indices]
    
    for i, (r, c) in enumerate(stones):
        row_map[r].append(i)
        col_map[c].append(i)
    
    # Union stones in same row
    for indices in row_map.values():
        for i in range(1, len(indices)):
            uf.union(indices[0], indices[i])
    
    # Union stones in same column
    for indices in col_map.values():
        for i in range(1, len(indices)):
            uf.union(indices[0], indices[i])
    
    # Count components
    components = len(set(uf.find(i) for i in range(n)))
    return n - components
```

---

## Approach 2: Union Rows and Columns (Clever!)

```
Instead of unioning stone indices, union ROW and COLUMN 
as abstract entities.

Each stone at (r, c) says: "row r and column c are connected"

To avoid row and column index collision:
  Row r → r
  Column c → c + offset (e.g., c + 10001)

Each stone: union(r, c + 10001)
Components = connected groups of rows and columns.
```

```python
def removeStones(stones):
    parent = {}
    
    def find(x):
        if x not in parent:
            parent[x] = x
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]
    
    def union(x, y):
        parent[find(x)] = find(y)
    
    for r, c in stones:
        union(r, c + 10001)  # offset columns
    
    # Count unique roots among stone positions
    roots = set()
    for r, c in stones:
        roots.add(find(r))
    
    return len(stones) - len(roots)
```

---

## Trace Example

```
stones = [[0,0], [0,1], [1,0], [1,2], [2,1], [2,2]]

Using row/column union (Approach 2):

Stone (0,0): union(row0, col10001) → {row0, col10001}
Stone (0,1): union(row0, col10002) → {row0, col10001, col10002}
Stone (1,0): union(row1, col10001) → {row0, row1, col10001, col10002}
Stone (1,2): union(row1, col10003) → {row0, row1, col10001, col10002, col10003}
Stone (2,1): union(row2, col10002) → {row0, row1, row2, col10001, col10002, col10003}
Stone (2,2): union(row2, col10003) → already same component

All stones in ONE component!

Components = 1
Answer = 6 - 1 = 5

We can remove 5 out of 6 stones. ✓
```

---

## Why They Connect Transitively

```
Stone(0,0) and Stone(1,0) share column 0:
  Stone(0,0) → row 0 ~ col 0
  Stone(1,0) → row 1 ~ col 0
  Therefore: row 0 ~ col 0 ~ row 1  (connected!)

Stone(0,0) and Stone(0,1) share row 0:
  Stone(0,0) → row 0 ~ col 0
  Stone(0,1) → row 0 ~ col 1
  Therefore: col 0 ~ row 0 ~ col 1  (connected!)

The chain of shared rows/columns = Union-Find path.

    row0 ─── col0
     |         |
    col1     row1
     |         |
    row2     col2
     |         
    col2  (already connected to row1 via Stone(1,2))
    
    All in one component!
```

---

## Approach Comparison

```
┌─────────────────────┬──────────────────┬────────────────┐
│ Approach            │ Time             │ Space          │
├─────────────────────┼──────────────────┼────────────────┤
│ Union stone indices │ O(n × α(n))      │ O(n + R + C)   │
│ with row/col maps   │                  │                │
├─────────────────────┼──────────────────┼────────────────┤
│ Union row/col       │ O(n × α(n))      │ O(n)           │
│ (direct union)      │                  │ (hash-based)   │
├─────────────────────┼──────────────────┼────────────────┤
│ DFS on adj graph    │ O(n²)            │ O(n²)          │
│                     │ (build adj list) │                │
└─────────────────────┴──────────────────┴────────────────┘

Row/column union is the most elegant!
```

---

## Related Problem: Similar String Groups (LeetCode 839)

```
Two strings are similar if they differ in exactly 
0 or 2 positions (i.e., one swap makes them equal).

Group similar strings and count groups.

Approach:
  For each pair of strings, check if similar.
  If similar → Union them.
  
  Answer = number of groups = total - removable? 
  No, just count groups.
  
  Time: O(n² × L) where L = string length
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Max removable stones |
| Key formula | removed = total - components |
| Approach 1 | Union stone indices via row/col maps |
| Approach 2 | Union row and column as abstract nodes |
| Column offset | c + 10001 (avoid collision with rows) |
| Component = | Group sharing rows/columns transitively |

---

## Quick Revision Questions

1. **Why is the answer `total - components`?**
2. **What is the row/column union trick?**
3. **Why do you need an offset for column indices?**
4. **How does transitivity work through shared rows/columns?**
5. **What's the advantage of the row/column approach over pairwise comparison?**

---

[← Previous: Regions Cut by Slashes](03-regions-cut-by-slashes.md) | [Next: Swim in Rising Water →](05-swim-in-rising-water.md)
