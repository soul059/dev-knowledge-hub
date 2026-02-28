# Chapter 1: Union by Rank

## Overview

**Union by Rank** is an optimization that controls **which root becomes the child** during a Union operation. It always attaches the shorter tree under the taller tree, preventing degenerate chains.

---

## The Problem with Naive Union

```
Naive Union can attach a tall tree under a short one:

     0       5             5
    /|\      |             |
   1 2 3     |             0         ← Height increased!
             |            /|\
                          1 2 3

Height went from max(3,1) = 3 to 4.
EVERY node in the larger tree is now 1 level deeper!

Better: Attach the shorter tree under the taller one:
     0       5             0
    /|\      |            /|\ \
   1 2 3                 1 2 3 5     ← Height unchanged!

Height stays at 3. Only 1 node (5) gets deeper.
```

---

## What is Rank?

**Rank** is an upper bound on the height of a subtree rooted at a node.

```
Rules:
  1. Initially, every node has rank 0
  2. Rank only increases when TWO trees of EQUAL rank are merged:
     - rank[newRoot] = rank[oldRoot] + 1
  3. When trees of different rank merge:
     - Shorter tree goes under taller tree
     - rank stays the same

  ┌────────────────────────────────────────────────┐
  │  Rank is NOT the actual height!                │
  │  It's an UPPER BOUND on height.                │
  │  With path compression, actual height < rank.  │
  └────────────────────────────────────────────────┘
```

---

## Algorithm

```
function Union(x, y):
    rootX = Find(x)
    rootY = Find(y)
    if rootX == rootY: return     // same set
    
    if rank[rootX] < rank[rootY]:
        parent[rootX] = rootY          // shorter under taller
    else if rank[rootX] > rank[rootY]:
        parent[rootY] = rootX          // shorter under taller
    else:
        parent[rootY] = rootX          // equal rank: pick one
        rank[rootX] += 1               // increment winner's rank
```

---

## Step-by-Step Trace

```
Initialize: 8 elements (0-7)
parent = [0, 1, 2, 3, 4, 5, 6, 7]
rank   = [0, 0, 0, 0, 0, 0, 0, 0]

═══ Union(0, 1) ═══
  Find(0)=0, Find(1)=1
  rank[0]=0 == rank[1]=0 → equal!
  parent[1] = 0, rank[0] = 1

  parent = [0, 0, 2, 3, 4, 5, 6, 7]
  rank   = [1, 0, 0, 0, 0, 0, 0, 0]
  
  Tree:  0(r=1)   2  3  4  5  6  7
         |
         1(r=0)

═══ Union(2, 3) ═══
  rank[2]=0 == rank[3]=0 → equal!
  parent[3] = 2, rank[2] = 1

  parent = [0, 0, 2, 2, 4, 5, 6, 7]
  rank   = [1, 0, 1, 0, 0, 0, 0, 0]
  
  Trees:  0(r=1)  2(r=1)  4  5  6  7
          |       |
          1       3

═══ Union(4, 5) ═══
  rank[4]=0 == rank[5]=0 → equal!
  parent[5] = 4, rank[4] = 1

═══ Union(6, 7) ═══
  rank[6]=0 == rank[7]=0 → equal!
  parent[7] = 6, rank[6] = 1

  Trees:  0(r=1)  2(r=1)  4(r=1)  6(r=1)
          |       |       |       |
          1       3       5       7

═══ Union(0, 2) ═══
  Find(0)=0, Find(2)=2
  rank[0]=1 == rank[2]=1 → equal!
  parent[2] = 0, rank[0] = 2

  Tree:     0(r=2)          4(r=1)  6(r=1)
           / \              |       |
         1    2(r=1)        5       7
              |
              3

═══ Union(4, 6) ═══
  rank[4]=1 == rank[6]=1 → equal!
  parent[6] = 4, rank[4] = 2

  Trees:   0(r=2)            4(r=2)
          / \               / \
        1    2(r=1)       5    6(r=1)
             |                 |
             3                 7

═══ Union(0, 4) ═══
  rank[0]=2 == rank[4]=2 → equal!
  parent[4] = 0, rank[0] = 3

  Final tree:
              0(r=3)
           /  |  \
         1   2    4(r=2)
             |   / \
             3  5   6(r=1)
                    |
                    7

  Height = 3 = log₂(8)  ← OPTIMAL!
```

---

## Why Rank Guarantees O(log n) Height

```
Claim: A tree with rank r has at least 2^r nodes.

Proof by induction:
  Base: rank 0 tree has 1 node = 2^0 ✓
  
  Rank increases only when merging two rank-r trees.
  Each has ≥ 2^r nodes.
  Combined: ≥ 2^r + 2^r = 2^(r+1) nodes ✓

Therefore:
  n ≥ 2^rank  →  rank ≤ log₂(n)
  
  Since rank is an upper bound on height:
  height ≤ rank ≤ log₂(n) = O(log n)

  ┌────────────────────────────────────┐
  │  With Union by Rank alone:         │
  │  Find = O(log n)                   │
  │  Union = O(log n)                  │
  │                                    │
  │  Much better than O(n)!            │
  └────────────────────────────────────┘
```

---

## Implementation

### Python
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx     # ensure rx has higher rank
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        return True
```

### C++
```cpp
class UnionFind {
    vector<int> parent, rank_;
public:
    UnionFind(int n) : parent(n), rank_(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        while (parent[x] != x) x = parent[x];
        return x;
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank_[rx] < rank_[ry]) swap(rx, ry);
        parent[ry] = rx;
        if (rank_[rx] == rank_[ry]) rank_[rx]++;
        return true;
    }
};
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Purpose | Keep trees balanced during Union |
| Rank meaning | Upper bound on subtree height |
| Rank increases when | Two equal-rank trees merge |
| Direction | Shorter tree under taller tree |
| Height bound | O(log n) |
| Extra space | rank[] array — O(n) |
| Combined with PC | O(α(n)) amortized |

---

## Quick Revision Questions

1. **What is rank in Union by Rank?**
2. **When does the rank of a root increase?**
3. **Why does Union by Rank guarantee O(log n) height?**
4. **What happens when two trees of different rank are merged?**
5. **Is rank always equal to the actual tree height?**

---

[← Previous: Path Splitting](../03-Path-Compression/06-path-splitting.md) | [Next: Union by Size →](02-union-by-size.md)
