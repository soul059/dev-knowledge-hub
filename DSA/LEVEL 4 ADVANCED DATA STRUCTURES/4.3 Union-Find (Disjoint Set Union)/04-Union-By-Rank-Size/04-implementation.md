# Chapter 4: Implementation

## Overview

This chapter provides the **complete, production-ready** Union-Find implementation with both Union by Rank and Union by Size variants.

---

## Complete Implementation: Union by Rank

### Python
```python
class UnionFindRank:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        """Find with path compression (recursive)."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union by rank. Returns True if merge happened."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        
        # Attach smaller rank under larger rank
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx  # ensure rx has >= rank
        
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

### C++
```cpp
class UnionFindRank {
    vector<int> parent, rank_;
    int components;
    
public:
    UnionFindRank(int n) : parent(n), rank_(n, 0), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);
        return parent[x];
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        
        if (rank_[rx] < rank_[ry]) swap(rx, ry);
        parent[ry] = rx;
        if (rank_[rx] == rank_[ry]) rank_[rx]++;
        
        components--;
        return true;
    }
    
    bool connected(int x, int y) {
        return find(x) == find(y);
    }
    
    int count() { return components; }
};
```

---

## Complete Implementation: Union by Size

### Python
```python
class UnionFindSize:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n
    
    def find(self, x):
        """Find with path compression."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union by size. Returns True if merge happened."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        
        # Attach smaller set under larger set
        if self.size[rx] < self.size[ry]:
            rx, ry = ry, rx  # rx is the larger set
        
        self.parent[ry] = rx
        self.size[rx] += self.size[ry]
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
    
    def get_size(self, x):
        """Size of the set containing x."""
        return self.size[self.find(x)]
    
    def get_max_size(self):
        """Size of the largest set."""
        return max(self.size[self.find(i)] for i in range(len(self.parent)))
```

### C++
```cpp
class UnionFindSize {
    vector<int> parent, sz;
    int components;
    
public:
    UnionFindSize(int n) : parent(n), sz(n, 1), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);
        return parent[x];
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
    int count() { return components; }
};
```

### Java
```java
class UnionFindSize {
    private int[] parent, size;
    private int components;
    
    public UnionFindSize(int n) {
        parent = new int[n];
        size = new int[n];
        components = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }
    
    public int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);
        return parent[x];
    }
    
    public boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        
        if (size[rx] < size[ry]) {
            int temp = rx; rx = ry; ry = temp;
        }
        parent[ry] = rx;
        size[rx] += size[ry];
        components--;
        return true;
    }
    
    public int getSize(int x) { return size[find(x)]; }
    public int getComponents() { return components; }
}
```

---

## Space-Optimized: Negative Size Trick

Store **negative size** in the parent array for roots — no separate size array needed!

```python
class UnionFindNegSize:
    """Space-optimized: parent[root] = -(size of set)"""
    
    def __init__(self, n):
        self.parent = [-1] * n  # -1 = root with size 1
    
    def find(self, x):
        if self.parent[x] < 0:  # negative = root
            return x
        self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        # Larger magnitude (more negative) = more elements
        if self.parent[rx] > self.parent[ry]:  # rx has fewer
            rx, ry = ry, rx
        self.parent[rx] += self.parent[ry]  # add sizes
        self.parent[ry] = rx               # attach
        return True
    
    def get_size(self, x):
        return -self.parent[self.find(x)]
```

```
Encoding:
  parent[i] ≥ 0  → i is not a root, parent[i] is its parent
  parent[i] < 0  → i is a root, |parent[i]| is the set size

Example:
  parent = [-4, 0, 0, 2]
  
  Node 0: parent[0] = -4 → root, size = 4
  Node 1: parent[1] = 0  → parent is 0
  Node 2: parent[2] = 0  → parent is 0
  Node 3: parent[3] = 2  → parent is 2
  
  Tree:    0 (size=4)
          / \
         1   2
             |
             3
```

---

## Template for Competitive Programming

```cpp
// Minimal competitive programming template
struct DSU {
    vector<int> p, sz;
    int comps;
    
    DSU(int n) : p(n), sz(n, 1), comps(n) {
        iota(p.begin(), p.end(), 0);
    }
    
    int find(int x) {
        return p[x] == x ? x : p[x] = find(p[x]);
    }
    
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (sz[x] < sz[y]) swap(x, y);
        p[y] = x;
        sz[x] += sz[y];
        comps--;
        return true;
    }
};
```

```python
# Minimal Python template
class DSU:
    def __init__(self, n):
        self.p = list(range(n))
        self.sz = [1] * n
    
    def find(self, x):
        if self.p[x] != x:
            self.p[x] = self.find(self.p[x])
        return self.p[x]
    
    def union(self, x, y):
        x, y = self.find(x), self.find(y)
        if x == y: return False
        if self.sz[x] < self.sz[y]: x, y = y, x
        self.p[y] = x
        self.sz[x] += self.sz[y]
        return True
```

---

## Summary Table

| Implementation | Arrays | Root Check | Size Query | Space |
|---------------|--------|------------|------------|-------|
| Rank-based | parent[], rank[] | parent[x]==x | N/A | 2n |
| Size-based | parent[], size[] | parent[x]==x | size[find(x)] | 2n |
| Negative size | parent[] | parent[x]<0 | -parent[find(x)] | n |

---

## Quick Revision Questions

1. **Write the Union by Size code from memory (pseudocode).**
2. **How does the negative size trick encode both parent and size in one array?**
3. **What is the root condition in the negative size encoding?**
4. **In the competitive programming template, what does `p[x] == x ? x : p[x] = find(p[x])` do?**
5. **Which implementation would you choose if you need set sizes? Why?**

---

[← Previous: Why It Helps](03-why-it-helps.md) | [Next: Combined with Path Compression →](05-combined-with-path-compression.md)
