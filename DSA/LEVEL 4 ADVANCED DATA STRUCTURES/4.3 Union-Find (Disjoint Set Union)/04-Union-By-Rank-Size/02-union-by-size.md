# Chapter 2: Union by Size

## Overview

**Union by Size** is an alternative to Union by Rank. Instead of tracking tree height, it tracks the **number of elements** in each tree and always attaches the smaller tree under the larger one.

---

## The Idea

```
When merging two sets, attach the SMALLER set under the LARGER set.

This minimizes the number of nodes whose depth increases.

  Small set (2 nodes):    Large set (5 nodes):
       A                       B
       |                     / | \
       C                    D  E  F
                                |
                                G

  GOOD: parent[A] = B          BAD: parent[B] = A
       B (size=7)                  A (size=7)
     / | \ \                       |
    D  E  F  A                     B         ← 5 nodes deeper!
       |     |                   / | \
       G     C                  D  E  F
                                   |
  2 nodes got 1 deeper             G
                              2+5 nodes affected worse
```

---

## Algorithm

```
function Union(x, y):
    rootX = Find(x)
    rootY = Find(y)
    if rootX == rootY: return
    
    // Attach smaller tree under larger tree
    if size[rootX] < size[rootY]:
        parent[rootX] = rootY
        size[rootY] += size[rootX]
    else:
        parent[rootY] = rootX
        size[rootX] += size[rootY]
```

---

## Step-by-Step Trace

```
Initialize: 6 elements
parent = [0, 1, 2, 3, 4, 5]
size   = [1, 1, 1, 1, 1, 1]

═══ Union(0, 1) ═══
  size[0]=1, size[1]=1 → equal, pick 0 as root
  parent[1] = 0, size[0] = 2

  parent = [0, 0, 2, 3, 4, 5]
  size   = [2, 1, 1, 1, 1, 1]
  
     0(s=2)  2  3  4  5
     |
     1

═══ Union(2, 3) ═══
  size[2]=1, size[3]=1 → equal
  parent[3] = 2, size[2] = 2

     0(s=2)  2(s=2)  4  5
     |       |
     1       3

═══ Union(0, 2) ═══
  size[0]=2, size[2]=2 → equal
  parent[2] = 0, size[0] = 4

     0(s=4)     4  5
    / \
   1   2(s=2)
       |
       3

═══ Union(4, 5) ═══
  size[4]=1, size[5]=1 → equal
  parent[5] = 4, size[4] = 2

     0(s=4)    4(s=2)
    / \        |
   1   2       5
       |
       3

═══ Union(3, 5) ═══
  Find(3)=0 (size=4), Find(5)=4 (size=2)
  size[0]=4 > size[4]=2 → attach 4 under 0
  parent[4] = 0, size[0] = 6

         0(s=6)
       / | \
      1  2  4(s=2)
         |  |
         3  5

  Height = 2 = O(log n) ✓
```

---

## Union by Size vs Union by Rank

```
┌────────────────────┬──────────────┬──────────────┐
│ Aspect             │ Union by Rank│ Union by Size│
├────────────────────┼──────────────┼──────────────┤
│ Tracks             │ Height bound │ Element count│
│ Extra array        │ rank[]       │ size[]       │
│ Initial value      │ 0            │ 1            │
│ Update rule        │ +1 on equal  │ += other size│
│ Height guarantee   │ O(log n)     │ O(log n)     │
│ Can query set size │ No           │ YES!         │
│ With path comp.    │ O(α(n))      │ O(α(n))      │
│ Practical use      │ Common       │ More useful  │
└────────────────────┴──────────────┴──────────────┘

Key advantage of size: You can query "how many elements 
are in this set?" in O(1)!
  Answer: size[Find(x)]
```

---

## Why O(log n) Height?

```
Claim: With union by size, tree height ≤ log₂(n)

Proof: When a node's depth increases by 1, its tree size 
       at least DOUBLES.

  Before merge:
    Node x is in tree A (size a)
    Tree B has size b ≥ a  (A goes under B)
    
  After merge:
    x's depth increased by 1
    New tree size = a + b ≥ 2a  (doubled!)
  
  Starting from size 1, doubling can happen at most log₂(n) times
  before reaching size n.
  
  Therefore: depth of any node ≤ log₂(n)

  n = 1:   max depth = 0 = log₂(1)
  n = 2:   max depth = 1 = log₂(2)
  n = 4:   max depth = 2 = log₂(4)
  n = 8:   max depth = 3 = log₂(8)
  n = 1000: max depth = 9 ≈ log₂(1000)
```

---

## Implementation

### Python
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n
    
    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        # Attach smaller under larger
        if self.size[rx] < self.size[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        self.size[rx] += self.size[ry]
        self.components -= 1
        return True
    
    def get_size(self, x):
        """Return size of the set containing x."""
        return self.size[self.find(x)]
```

### C++
```cpp
class UnionFind {
    vector<int> parent, sz;
    int components;
public:
    UnionFind(int n) : parent(n), sz(n, 1), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        while (parent[x] != x) x = parent[x];
        return x;
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (sz[rx] < sz[ry]) swap(rx, ry);
        parent[ry] = rx;
        sz[rx] += sz[ry];
        components--;
        return true;
    }
    
    int getSize(int x) { return sz[find(x)]; }
    int getComponents() { return components; }
};
```

---

## Practical Advantage: Size Queries

```
With Union by Size, you get set sizes for FREE:

  uf = UnionFind(10)
  uf.union(0, 1)     // size of set = 2
  uf.union(2, 3)     // size of set = 2
  uf.union(0, 2)     // size of set = 4
  
  uf.get_size(3)     // → 4  (set {0,1,2,3})
  uf.get_size(5)     // → 1  (set {5})
  
  This is very common in problems:
  - "What is the size of the largest component?"
  - "How many nodes are in the same group as x?"
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Tracks | Number of elements per set |
| Initial size | 1 (singleton) |
| Update | size[winner] += size[loser] |
| Height bound | O(log n) |
| Bonus feature | O(1) set size queries |
| Extra space | size[] array — O(n) |
| Preferred when | Need set sizes |

---

## Quick Revision Questions

1. **How does Union by Size decide which root becomes the child?**
2. **What is the initial value in the size array?**
3. **How do you update sizes during Union?**
4. **Prove that depth is O(log n) with Union by Size.**
5. **What advantage does Union by Size have over Union by Rank?**

---

[← Previous: Union by Rank](01-union-by-rank.md) | [Next: Why It Helps →](03-why-it-helps.md)
