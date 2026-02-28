# Chapter 5: Combined with Path Compression

## Overview

The **combination** of path compression with union by rank/size achieves the theoretically optimal **O(α(n))** amortized time per operation. This chapter shows how they work together.

---

## Why Combine Both?

```
Path Compression alone:    O(log* n) amortized
Union by Rank alone:       O(log n) worst case
Both together:             O(α(n)) amortized ← OPTIMAL!

Each addresses a DIFFERENT weakness:
┌──────────────────────────────────────────────────────┐
│ Path Compression: Fixes PAST mistakes                │
│   → Flattens trees that are already deep             │
│                                                      │
│ Union by Rank/Size: Prevents FUTURE mistakes         │
│   → Keeps trees from becoming deep in first place    │
└──────────────────────────────────────────────────────┘
```

---

## How They Interact

```
Step 1: Union by rank creates a balanced tree (height ≤ log n)

         0 (rank=2)
        / \
       1   2 (rank=1)
      / \   \
     3   4   5

Step 2: Path compression during Find(3) flattens the path

         0 (rank=2)         ← rank stays 2 (doesn't change!)
       / | \ \
      1  2  3  4             ← 3 moved directly under root
         |
         5

Note: Rank is now INACCURATE as a height measure!
      Rank = 2, but actual height = 2 (was 3)
      
This is fine! Rank is an UPPER BOUND, not exact height.
```

---

## The Rank Stays Stale (And That's OK)

```
Why rank doesn't update after path compression:

1. Updating rank would require knowing ALL node depths → O(n)
2. The rank is only used to CHOOSE direction during Union
3. As an upper bound, it still prevents worst-case behavior
4. The slight inaccuracy is what gives O(α(n)) instead of O(log n)

Think of rank as a "historical" height:
  "This tree ONCE had height rank, so it's probably large"
  → A reasonable heuristic for choosing Union direction
```

---

## Combined Trace

```
Elements: 0-7
parent = [0,1,2,3,4,5,6,7], rank = [0,0,0,0,0,0,0,0]

═══ Union(0,1) ═══  rank[0]=rank[1]=0 → parent[1]=0, rank[0]=1
═══ Union(2,3) ═══  rank[2]=rank[3]=0 → parent[3]=2, rank[2]=1
═══ Union(4,5) ═══  rank[4]=rank[5]=0 → parent[5]=4, rank[4]=1
═══ Union(6,7) ═══  rank[6]=rank[7]=0 → parent[7]=6, rank[6]=1

Trees (all height 1):
  0(r1) 2(r1) 4(r1) 6(r1)
  |     |     |     |
  1     3     5     7

═══ Union(0,2) ═══  rank[0]=rank[2]=1 → parent[2]=0, rank[0]=2
═══ Union(4,6) ═══  rank[4]=rank[6]=1 → parent[6]=4, rank[4]=2

Trees (height 2):
     0(r2)          4(r2)
    / \             / \
   1   2(r1)       5   6(r1)
       |               |
       3               7

═══ Union(0,4) ═══  rank[0]=rank[4]=2 → parent[4]=0, rank[0]=3

Tree (height 3):
           0(r3)
         / |  \
        1  2   4(r2)
           |  / \
           3 5   6(r1)
                  |
                  7

═══ Now Find(7) with path compression ═══
Path: 7 → 6 → 4 → 0 (root)
Compress: parent[7]=0, parent[6]=0, parent[4]=0

Tree becomes:
          0(r3)          ← rank still 3!
       /||||||\
      1 2 3 4 5 6 7      ← nearly flat!

Actual height = 2 (node 3 via 2→0)
Rank = 3 (stale, but still valid upper bound)

Future Find(7) = O(1)!
```

---

## Performance: The Best of Both Worlds

```
Scenario: n = 1,000,000 elements, m = 5,000,000 operations

┌──────────────────────────┬──────────────┬────────────┐
│ Configuration            │ Max Height   │ Total Hops │
├──────────────────────────┼──────────────┼────────────┤
│ Naive                    │ ~500,000     │ ~10^12     │
│ Path compression only    │ ~20 → 1     │ ~25×10^6   │
│ Union by rank only       │ 20           │ ~100×10^6  │
│ BOTH                     │ 20 → 1      │ ~20×10^6   │
├──────────────────────────┼──────────────┼────────────┤
│ Per operation (both)     │              │ ~4 hops    │
│ α(10^6) =               │              │ 4          │
└──────────────────────────┴──────────────┴────────────┘
```

---

## The Complete Optimized Implementation

```python
class DSU:
    """Optimal Union-Find: Path Compression + Union by Rank"""
    
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        # Path compression (recursive)
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        # Union by rank
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
    
    def count(self):
        return self.components
```

```cpp
struct DSU {
    vector<int> p, rnk;
    int comps;
    
    DSU(int n) : p(n), rnk(n, 0), comps(n) {
        iota(p.begin(), p.end(), 0);
    }
    
    int find(int x) {
        return p[x] == x ? x : p[x] = find(p[x]);
    }
    
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (rnk[x] < rnk[y]) swap(x, y);
        p[y] = x;
        if (rnk[x] == rnk[y]) rnk[x]++;
        comps--;
        return true;
    }
};
```

---

## Note on Rank + Path Compression

```
Important subtlety:

When using path compression WITH rank:
  - Rank becomes an UPPER BOUND, not exact height
  - Path compression changes actual heights but NOT ranks
  - This is intentional and correct
  - The amortized analysis accounts for this discrepancy

When using path compression WITH size:
  - Size remains EXACT (not affected by path compression)
  - Path compression changes tree shape but not element count
  - size[find(x)] always returns the correct set size
  
  → Union by Size + Path Compression is often preferred
    for this reason
```

---

## Summary Table

| Optimization | Alone | Combined |
|-------------|-------|----------|
| Path Compression | O(log* n) amortized | O(α(n)) amortized |
| Union by Rank | O(log n) worst case | O(α(n)) amortized |
| Union by Size | O(log n) worst case | O(α(n)) amortized |

---

## Quick Revision Questions

1. **What does each optimization address — past or future problems?**
2. **Why does rank become inaccurate with path compression?**
3. **Is it okay that rank is inaccurate? Why?**
4. **What is the amortized complexity when both optimizations are used?**
5. **Why might Union by Size be preferred over Union by Rank when combined with path compression?**

---

[← Previous: Implementation](04-implementation.md) | [Next: Inverse Ackermann Function →](06-inverse-ackermann-function.md)
