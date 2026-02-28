# Chapter 2: Union Operation

## Overview

The **Union** operation merges two disjoint sets into one. It finds the roots of both elements and makes one root point to the other.

---

## Core Logic

```
function Union(x, y):
    rootX = Find(x)           // find representative of x
    rootY = Find(y)           // find representative of y
    if rootX ≠ rootY:         // only if in different sets
        parent[rootY] = rootX // make rootX the parent of rootY
```

**Key insight**: We always link **roots**, never arbitrary nodes.

---

## Why Link Roots?

```
CORRECT: Link root to root
  Before Union(3, 5):        After:
      0       2                 0
     / \      |               / | \
    1   3     5              1  3   2
    |   |                    |  |   |
    4   6                    4  6   5

  parent[2] = 0  ← root 2 now points to root 0

WRONG: Link node to node (DO NOT DO THIS!)
  If we set parent[5] = 3:
      0       2                0      2
     / \      |               / \
    1   3     5              1   3
    |   |                    |   |
    4   6                    4   6
                                 |
                                 5   ← 5 still has root 2!
                                     
  This breaks the invariant — Find(5) would return 2, not 0!
```

---

## Step-by-Step Trace

```
Initial: parent = [0, 1, 2, 3, 4, 5]
         6 elements, 6 sets

Trees:  0   1   2   3   4   5

═══ Union(0, 1) ═══
  Find(0) = 0, Find(1) = 1
  rootX = 0, rootY = 1
  0 ≠ 1 → parent[1] = 0
  
  parent = [0, 0, 2, 3, 4, 5]
  Trees:  0   2   3   4   5
          |
          1

═══ Union(2, 3) ═══
  Find(2) = 2, Find(3) = 3
  rootX = 2, rootY = 3
  2 ≠ 3 → parent[3] = 2
  
  parent = [0, 0, 2, 2, 4, 5]
  Trees:  0    2    4   5
          |    |
          1    3

═══ Union(4, 5) ═══
  Find(4) = 4, Find(5) = 5
  rootX = 4, rootY = 5
  4 ≠ 5 → parent[5] = 4
  
  parent = [0, 0, 2, 2, 4, 4]
  Trees:  0    2    4
          |    |    |
          1    3    5

═══ Union(1, 3) ═══
  Find(1): 1 → parent[1]=0 → root! → rootX = 0
  Find(3): 3 → parent[3]=2 → root! → rootY = 2
  0 ≠ 2 → parent[2] = 0
  
  parent = [0, 0, 0, 2, 4, 4]
  Trees:    0         4
           / \        |
          1   2       5
              |
              3

═══ Union(0, 5) ═══
  Find(0) = 0 (root)
  Find(5): 5 → parent[5]=4 → root! → rootY = 4
  0 ≠ 4 → parent[4] = 0
  
  parent = [0, 0, 0, 2, 0, 4]
  Tree:       0
            / | \
           1  2  4
              |  |
              3  5

All 6 elements now in ONE set!
```

---

## Union When Already Connected

```
Union(x, y) where x and y are already in the same set:

parent = [0, 0, 0, 2, 0, 4]

Union(1, 3):
  Find(1) = 0
  Find(3) = 0      ← same root!
  rootX == rootY   → do nothing (no-op)
  
This is correct! No structural change needed.
```

---

## Counting Components

A useful extension — track the number of distinct sets:

```
function Initialize(n):
    for i = 0 to n-1:
        parent[i] = i
    numComponents = n

function Union(x, y):
    rootX = Find(x)
    rootY = Find(y)
    if rootX ≠ rootY:
        parent[rootY] = rootX
        numComponents -= 1      // one fewer component
    
    // return whether a merge happened
    return rootX ≠ rootY

Trace:
  n = 5, numComponents = 5
  Union(0,1): merge → numComponents = 4
  Union(2,3): merge → numComponents = 3
  Union(0,2): merge → numComponents = 2
  Union(0,1): already same → numComponents = 2 (no change)
  Union(3,4): merge → numComponents = 1
```

---

## Union Direction Choice

In naive Union, the direction is arbitrary:

```
Union(x, y): parent[rootY] = rootX   (rootX becomes parent)
  OR
Union(x, y): parent[rootX] = rootY   (rootY becomes parent)

Both are valid! The choice affects tree shape:

Option A: parent[rootY] = rootX     Option B: parent[rootX] = rootY
     rootX                               rootY
      |                                    |
    rootY                               rootX

This arbitrary choice can lead to bad tree shapes (see Limitations).
Optimizations (Union by Rank/Size) make this choice strategic.
```

---

## The Union Process Diagram

```
Union(x, y) — Complete Flow:

  ┌─────────┐     ┌─────────┐
  │  x = 3  │     │  y = 7  │
  └────┬────┘     └────┬────┘
       │               │
       ▼               ▼
  ┌─────────┐     ┌─────────┐
  │ Find(3) │     │ Find(7) │
  │ = rootX │     │ = rootY │
  └────┬────┘     └────┬────┘
       │               │
       └───────┬───────┘
               │
               ▼
        ┌──────────────┐
        │rootX == rootY│
        │      ?       │
        └──────┬───────┘
          YES  │  NO
          ┌────┴────┐
          ▼         ▼
       ┌──────┐ ┌────────────────┐
       │No-op │ │parent[rootY]   │
       │      │ │  = rootX       │
       └──────┘ │numComponents-- │
                └────────────────┘
```

---

## Implementation in Code

### Python
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.components = n
    
    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        root_x = self.find(x)
        root_y = self.find(y)
        if root_x != root_y:
            self.parent[root_y] = root_x
            self.components -= 1
            return True     # merge happened
        return False        # already connected
```

### C++
```cpp
class UnionFind {
    vector<int> parent;
    int components;
public:
    UnionFind(int n) : parent(n), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        while (parent[x] != x) x = parent[x];
        return x;
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        parent[ry] = rx;
        components--;
        return true;
    }
    
    int getComponents() { return components; }
};
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Purpose | Merge two disjoint sets |
| Prerequisite | Two Find calls to locate roots |
| Action | Set one root's parent to the other root |
| Already connected | No-op (safe, no error) |
| Component tracking | Decrement counter on successful merge |
| Time (naive) | O(h) dominated by Find calls |
| Returns (optional) | Boolean indicating if merge occurred |

---

## Quick Revision Questions

1. **Why must Union link roots and not arbitrary nodes?**
2. **What happens when Union is called on two elements already in the same set?**
3. **How does Union help track the number of components?**
4. **What is the time complexity of naive Union?**
5. **Does the direction of linking (which root becomes the child) matter in naive Union?**

---

[← Previous: Find Operation](01-find-operation.md) | [Next: Naive Implementation →](03-naive-implementation.md)
