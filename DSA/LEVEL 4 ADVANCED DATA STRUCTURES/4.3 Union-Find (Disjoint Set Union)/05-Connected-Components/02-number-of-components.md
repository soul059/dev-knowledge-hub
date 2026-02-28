# Chapter 2: Number of Components

## Overview

Tracking the **number of connected components** is one of the most common uses of Union-Find. Starting with n components (each node is its own), the count decreases by 1 with each successful Union.

---

## Core Idea

```
Rule: Every successful Union reduces component count by 1

Initialize: n elements → n components

Union(a, b):
  If Find(a) ≠ Find(b):
    merge them → components -= 1
  If Find(a) = Find(b):
    already same set → no change

Final answer: Read the components counter

┌──────────────────────────────────────────────┐
│  components = n - (number of successful      │
│                     Union operations)         │
└──────────────────────────────────────────────┘
```

---

## Trace Example

```
n = 6, edges: (0,1), (2,3), (1,2), (4,5), (3,4)

Start: components = 6
  {0} {1} {2} {3} {4} {5}

Union(0,1): success → components = 5
  {0,1} {2} {3} {4} {5}

Union(2,3): success → components = 4
  {0,1} {2,3} {4} {5}

Union(1,2): success → components = 3
  {0,1,2,3} {4} {5}

Union(4,5): success → components = 2
  {0,1,2,3} {4,5}

Union(3,4): success → components = 1
  {0,1,2,3,4,5}

Answer: 1 component
```

---

## Classic Problem: Number of Connected Components in Undirected Graph

```
LeetCode 323: Number of Connected Components in an Undirected Graph

Given n nodes and a list of edges, find the number of 
connected components.

Input: n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2

  0──1──2    3──4
  (comp 1)  (comp 2)
```

### Solution
```python
def countComponents(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.components

# Time: O(E × α(n))
# Space: O(n)
```

---

## Classic Problem: Number of Islands (Grid)

```
LeetCode 200: Number of Islands

Given a 2D grid of '1' (land) and '0' (water), count 
the number of islands.

Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Answer: 3 islands
```

### Solution
```python
def numIslands(grid):
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    
    # Count land cells, start with that many "components"
    land_count = sum(grid[r][c] == '1' 
                     for r in range(rows) 
                     for c in range(cols))
    
    uf = UnionFind(rows * cols)
    uf.components = land_count
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                # Check right and down neighbors
                for dr, dc in [(0, 1), (1, 0)]:
                    nr, nc = r + dr, c + dc
                    if (0 <= nr < rows and 0 <= nc < cols 
                        and grid[nr][nc] == '1'):
                        uf.union(r * cols + c, nr * cols + nc)
    
    return uf.components
```

```
Grid-to-index mapping:
  (r, c) → r * cols + c

  (0,0)=0  (0,1)=1  (0,2)=2  (0,3)=3  (0,4)=4
  (1,0)=5  (1,1)=6  (1,2)=7  (1,3)=8  (1,4)=9
  (2,0)=10 (2,1)=11 (2,2)=12 (2,3)=13 (2,4)=14
  (3,0)=15 (3,1)=16 (3,2)=17 (3,3)=18 (3,4)=19

Union operations:
  Union(0,1), Union(0,5), Union(1,6), Union(5,6) → island 1
  Union(12) is alone → island 2
  Union(18,19) → island 3
  
  Components with land = 3
```

---

## Tracking Components Over Time

```
Problem: After each edge addition, report the number 
of components.

n = 5, edges added one at a time:

Step 0:  {0}{1}{2}{3}{4}         Components: 5
Step 1:  add(0,1) → merge        Components: 4
Step 2:  add(2,3) → merge        Components: 3
Step 3:  add(0,2) → merge        Components: 2
Step 4:  add(1,3) → already same Components: 2  (no change!)
Step 5:  add(3,4) → merge        Components: 1

Output: [5, 4, 3, 2, 2, 1]
```

```python
def componentHistory(n, edges):
    uf = UnionFind(n)
    history = [n]
    for u, v in edges:
        uf.union(u, v)
        history.append(uf.components)
    return history
```

---

## Implementation with Component Count

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n        # ← Track this!
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False           # No merge → no decrement
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1       # ← Decrement on merge
        return True
```

---

## Edge Cases

```
1. No edges at all:
   components = n (every node is its own island)

2. Self-loop edge (u, u):
   Find(u) = Find(u) → no merge → components unchanged

3. Duplicate edge:
   Already same component → no change

4. Fully connected graph (n-1 edges forming spanning tree):
   components = 1

5. Empty graph (n = 0):
   components = 0
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Initial count | n |
| After successful Union | count -= 1 |
| Redundant Union | No change |
| Final minimum | 1 (if graph is connected) |
| Query time | O(1) — just read counter |
| Total time | O(E × α(n)) for E edges |

---

## Quick Revision Questions

1. **How do you initialize the component count?**
2. **When does the component count NOT decrease on a Union?**
3. **How do you map a 2D grid position to a 1D Union-Find index?**
4. **What is the minimum possible number of components?**
5. **How would you find the component count at each step?**

---

[← Previous: Dynamic Connectivity](01-dynamic-connectivity.md) | [Next: Is-Connected Queries →](03-is-connected-queries.md)
