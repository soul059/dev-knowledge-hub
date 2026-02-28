# Chapter 5: MST Implementation Details

## Overview

This chapter covers practical implementation details, edge cases, and coding patterns for Kruskal's MST with Union-Find in competitive programming and real-world scenarios.

---

## Complete Production-Ready Implementation

### Python
```python
class KruskalMST:
    def __init__(self, n):
        self.n = n
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False
        if self.rank[rx] < self.rank[ry]: rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]: self.rank[rx] += 1
        self.components -= 1
        return True
    
    def solve(self, edges):
        """
        Args: edges = [(u, v, weight), ...]
        Returns: (mst_weight, mst_edges, is_connected)
        """
        edges = sorted(edges, key=lambda e: e[2])
        
        mst_edges = []
        mst_weight = 0
        
        for u, v, w in edges:
            if self.union(u, v):
                mst_edges.append((u, v, w))
                mst_weight += w
                if len(mst_edges) == self.n - 1:
                    break
        
        return mst_weight, mst_edges, self.components == 1
```

### C++ (Competitive Programming)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> p, rnk;
    DSU(int n) : p(n), rnk(n, 0) { iota(p.begin(), p.end(), 0); }
    int find(int x) { return p[x] == x ? x : p[x] = find(p[x]); }
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (rnk[x] < rnk[y]) swap(x, y);
        p[y] = x;
        if (rnk[x] == rnk[y]) rnk[x]++;
        return true;
    }
};

int main() {
    int n, m;
    cin >> n >> m;
    
    vector<tuple<int,int,int>> edges(m); // weight, u, v
    for (auto& [w, u, v] : edges) cin >> u >> v >> w;
    
    sort(edges.begin(), edges.end()); // sort by weight (first element)
    
    DSU dsu(n);
    long long mst = 0;
    int count = 0;
    
    for (auto& [w, u, v] : edges) {
        if (dsu.unite(u, v)) {
            mst += w;
            if (++count == n - 1) break;
        }
    }
    
    if (count < n - 1) cout << "IMPOSSIBLE\n";
    else cout << mst << "\n";
}
```

---

## Edge Cases

```
1. Single node (n=1):
   MST weight = 0, MST edges = [] (empty tree)
   
2. No edges (E=0, n>1):
   MST impossible (disconnected)
   
3. All edges same weight:
   Any spanning tree is an MST
   
4. Negative weights:
   Kruskal's handles them correctly — sort normally
   
5. Self-loops (u=v):
   find(u) == find(u) → always rejected → no issue
   
6. Parallel edges:
   Sorting handles naturally — lighter one picked first
   
7. Very large weights:
   Use long long / int64 for total weight

8. 1-indexed nodes:
   DSU(n+1) or subtract 1 from all indices
```

---

## Common Variations

### Maximum Spanning Tree
```python
# Just negate weights or sort in descending order!
edges.sort(key=lambda e: -e[2])  # sort descending
# Then run Kruskal's normally
```

### Second-Best MST
```
For each non-MST edge (u,v,w):
  Adding it creates a cycle in MST.
  The heaviest edge in the cycle (excluding (u,v)) 
  can be swapped.
  
  Second-best = MST_weight + w - max_edge_on_path(u,v)
  
  Take minimum over all non-MST edges.
```

### MST with Forced Edges
```python
def mst_with_required(n, edges, required_edges):
    """Some edges must be in the MST."""
    uf = UnionFind(n)
    total = 0
    
    # Step 1: Force required edges
    for u, v, w in required_edges:
        uf.union(u, v)
        total += w
    
    # Step 2: Run Kruskal's for remaining
    edges.sort(key=lambda e: e[2])
    for u, v, w in edges:
        if uf.union(u, v):
            total += w
    
    return total
```

---

## Problem: Min Cost to Connect All Points

```
LeetCode 1584: Min Cost to Connect All Points

Given n points, connect all with minimum total 
Manhattan distance.

This is MST on a complete graph!
  Edges: Every pair of points
  Weight: |x1-x2| + |y1-y2|
  E = n(n-1)/2
```

```python
def minCostConnectPoints(points):
    n = len(points)
    edges = []
    
    for i in range(n):
        for j in range(i + 1, n):
            dist = (abs(points[i][0] - points[j][0]) + 
                    abs(points[i][1] - points[j][1]))
            edges.append((dist, i, j))
    
    edges.sort()
    
    uf = UnionFind(n)
    total = 0
    count = 0
    
    for w, u, v in edges:
        if uf.union(u, v):
            total += w
            count += 1
            if count == n - 1:
                break
    
    return total
```

---

## Performance Tips

```
1. Store edges as (weight, u, v) tuples
   → Python sorts by first element automatically
   → No need for key function

2. Use iterative path compression
   → Avoids Python recursion limit

3. Early termination at n-1 edges
   → Can save 50%+ time on dense graphs

4. For complete graphs (E = O(V²)):
   → Consider Prim's instead (O(V²) with array, O(E log V) with heap)
   → Kruskal's sorting is O(V² log V) 

5. For integer weights in small range:
   → Bucket sort → O(E + W) total
```

---

## Summary Table

| Variation | Change |
|-----------|--------|
| Standard MST | Sort ascending, Kruskal's |
| Maximum spanning tree | Sort descending |
| MSF (disconnected) | Don't check n-1 |
| Forced edges | Union forced first, then Kruskal's |
| Second-best MST | Swap each non-MST edge |
| Complete graph | Consider Prim's instead |

---

## Quick Revision Questions

1. **How do you handle 1-indexed nodes in Kruskal's?**
2. **How do you find a Maximum Spanning Tree?**
3. **What data type issue can occur with large weights?**
4. **How do you handle the "Min Cost to Connect All Points" problem?**
5. **When is Prim's algorithm preferred over Kruskal's?**
6. **How do you find the second-best MST?**

---

[← Previous: Minimum Spanning Forest](04-minimum-spanning-forest.md) | [Next: Unit 8 - Weighted Union-Find →](../08-Advanced-DSU/01-weighted-union-find.md)

[← Back to Main README](../README.md)
