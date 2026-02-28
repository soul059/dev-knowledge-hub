# Chapter 3: Naive Implementation

## Overview

The **naive implementation** of Union-Find uses only the parent array — no optimizations like path compression or union by rank. Understanding this version is essential before learning the optimizations.

---

## Complete Naive Implementation

### Pseudocode
```
class NaiveUnionFind:
    parent[]
    n
    
    function Initialize(size):
        n = size
        parent = new array[n]
        for i = 0 to n-1:
            parent[i] = i          // every element is its own root
    
    function Find(x):
        while parent[x] ≠ x:      // walk up to root
            x = parent[x]
        return x
    
    function Union(x, y):
        rootX = Find(x)
        rootY = Find(y)
        if rootX ≠ rootY:
            parent[rootY] = rootX  // arbitrary direction
    
    function Connected(x, y):
        return Find(x) == Find(y)
```

---

## Python Implementation

```python
class NaiveUnionFind:
    def __init__(self, n):
        """Initialize n elements (0 to n-1), each in its own set."""
        self.parent = list(range(n))
        self.num_components = n
    
    def find(self, x):
        """Find the root/representative of x's set."""
        while self.parent[x] != x:
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        """Merge sets containing x and y. Returns True if merged."""
        root_x = self.find(x)
        root_y = self.find(y)
        if root_x == root_y:
            return False  # already in same set
        self.parent[root_y] = root_x
        self.num_components -= 1
        return True
    
    def connected(self, x, y):
        """Check if x and y are in the same set."""
        return self.find(x) == self.find(y)
    
    def count(self):
        """Return number of disjoint sets."""
        return self.num_components
```

---

## C++ Implementation

```cpp
#include <vector>
#include <numeric>

class NaiveUnionFind {
private:
    std::vector<int> parent;
    int numComponents;

public:
    NaiveUnionFind(int n) : parent(n), numComponents(n) {
        std::iota(parent.begin(), parent.end(), 0);
    }

    int find(int x) {
        while (parent[x] != x)
            x = parent[x];
        return x;
    }

    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        parent[ry] = rx;
        numComponents--;
        return true;
    }

    bool connected(int x, int y) {
        return find(x) == find(y);
    }

    int count() { return numComponents; }
};
```

---

## Java Implementation

```java
public class NaiveUnionFind {
    private int[] parent;
    private int numComponents;

    public NaiveUnionFind(int n) {
        parent = new int[n];
        numComponents = n;
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }

    public int find(int x) {
        while (parent[x] != x)
            x = parent[x];
        return x;
    }

    public boolean union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return false;
        parent[rootY] = rootX;
        numComponents--;
        return true;
    }

    public boolean connected(int x, int y) {
        return find(x) == find(y);
    }

    public int count() { return numComponents; }
}
```

---

## Full Execution Trace

```
NaiveUnionFind uf = new NaiveUnionFind(7)

Initial state:
  parent = [0, 1, 2, 3, 4, 5, 6]
  components = 7
  Trees: 0  1  2  3  4  5  6

═══════════════════════════════════════
Operation: uf.union(0, 1)
  Find(0) = 0, Find(1) = 1
  0 ≠ 1 → parent[1] = 0
  components = 6

  parent = [0, 0, 2, 3, 4, 5, 6]
  Trees:  0   2  3  4  5  6
          |
          1

═══════════════════════════════════════
Operation: uf.union(2, 3)
  Find(2) = 2, Find(3) = 3
  2 ≠ 3 → parent[3] = 2
  components = 5

  parent = [0, 0, 2, 2, 4, 5, 6]
  Trees:  0   2   4  5  6
          |   |
          1   3

═══════════════════════════════════════
Operation: uf.union(4, 5)
  Find(4) = 4, Find(5) = 5
  4 ≠ 5 → parent[5] = 4
  components = 4

  parent = [0, 0, 2, 2, 4, 4, 6]
  Trees:  0   2   4   6
          |   |   |
          1   3   5

═══════════════════════════════════════
Operation: uf.union(5, 6)
  Find(5): 5→4 (root) → rootX = 4
  Find(6) = 6
  4 ≠ 6 → parent[6] = 4
  components = 3

  parent = [0, 0, 2, 2, 4, 4, 4]
  Trees:  0    2     4
          |    |    / \
          1    3   5   6

═══════════════════════════════════════
Operation: uf.union(1, 3)
  Find(1): 1→0 (root) → rootX = 0
  Find(3): 3→2 (root) → rootY = 2
  0 ≠ 2 → parent[2] = 0
  components = 2

  parent = [0, 0, 0, 2, 4, 4, 4]
  Trees:    0          4
           / \        / \
          1   2      5   6
              |
              3

═══════════════════════════════════════
Operation: uf.connected(1, 3)
  Find(1): 1→0 = 0
  Find(3): 3→2→0 = 0
  0 == 0 → TRUE ✓

Operation: uf.connected(0, 6)
  Find(0) = 0
  Find(6): 6→4 = 4
  0 ≠ 4 → FALSE ✗

═══════════════════════════════════════
Operation: uf.union(3, 6)
  Find(3): 3→2→0 → rootX = 0
  Find(6): 6→4   → rootY = 4
  0 ≠ 4 → parent[4] = 0
  components = 1

  parent = [0, 0, 0, 2, 0, 4, 4]
  Tree:        0
            / | \
           1  2  4
              | / \
              3 5  6

ALL elements in ONE set!
```

---

## Memory Layout Visualization

```
For n = 7 elements:

Memory allocation:
┌──────────────────────────────────────┐
│  parent array (7 integers)           │
│  ┌───┬───┬───┬───┬───┬───┬───┐      │
│  │ 0 │ 0 │ 0 │ 2 │ 0 │ 4 │ 4 │      │
│  └───┴───┴───┴───┴───┴───┴───┘      │
│    0   1   2   3   4   5   6         │
│                                      │
│  numComponents: 1 (integer)          │
│                                      │
│  Total space: O(n)                   │
└──────────────────────────────────────┘
```

---

## The Degenerate Case

Naive Union can create **chains** — the worst possible tree shape:

```
Sequence: Union(0,1), Union(1,2), Union(2,3), Union(3,4)

Step by step (always attach right root under left root):

Union(0,1): parent[1]=0       0
                               |
                               1

Union(1,2): Find(1)=0, Find(2)=2
            parent[2]=0        0
                              / \
                             1   2

Union(2,3): Find(2)=0, Find(3)=3
            parent[3]=0        0
                             / | \
                            1  2  3
                            
This is fine! But what if we always make the SECOND root the parent?

Alternative (parent[rootX] = rootY):
Union(0,1): parent[0]=1       1
                               |
                               0

Union(1,2): Find(1)=1, Find(2)=2
            parent[1]=2        2
                               |
                               1
                               |
                               0

Union(2,3): Find(2)=2, so Find(0) travels: 0→1→2
            parent[2]=3        3
                               |
                               2
                               |
                               1
                               |
                               0

This creates a CHAIN of length n → Find is O(n)!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Arrays used | parent[] only |
| Initialization | O(n) — set parent[i] = i |
| Find | O(h) — traverse to root |
| Union | O(h) — two Finds + one link |
| Space | O(n) |
| Worst-case height | O(n) — degenerate chain |
| Average case | O(log n) for random unions |

---

## Quick Revision Questions

1. **What is stored in the parent array initially?**
2. **What is the space complexity of naive Union-Find?**
3. **Write the three core methods from memory.**
4. **How does the component counter change during Union?**
5. **What shape of tree gives the worst-case Find performance?**
6. **Will Union-Find break if you call Union on already-connected elements?**

---

[← Previous: Union Operation](02-union-operation.md) | [Next: Time Complexity (Naive) →](04-time-complexity-naive.md)
