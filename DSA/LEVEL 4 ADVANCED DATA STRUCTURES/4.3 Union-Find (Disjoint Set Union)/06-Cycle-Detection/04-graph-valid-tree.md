# Chapter 4: Graph Valid Tree

## Overview

**Graph Valid Tree** (LeetCode 261) is a classic problem that combines Union-Find cycle detection with component counting. A graph is a tree if and only if it's connected and acyclic.

---

## Tree Properties

```
A graph with n nodes is a valid tree ⟺

  1. Exactly n-1 edges        (necessary for tree)
  2. No cycles                (acyclic)
  3. Fully connected          (one component)

Key insight: Any TWO of these imply the third!

  n-1 edges + no cycles → connected
  n-1 edges + connected → no cycles
  connected + no cycles → n-1 edges

So we only need to check TWO conditions.
```

---

## Solution 1: Edge Count + No Cycles

```python
def validTree(n, edges):
    # Condition 1: Must have exactly n-1 edges
    if len(edges) != n - 1:
        return False
    
    # Condition 2: No cycles
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u, v):
            return False  # cycle detected
    
    return True  # n-1 edges + no cycles = tree
```

Why this works: If there are exactly n-1 edges and no cycles, the graph must be a spanning tree (connected and acyclic).

---

## Solution 2: No Cycles + Connected

```python
def validTree(n, edges):
    uf = UnionFind(n)
    
    for u, v in edges:
        if not uf.union(u, v):
            return False  # cycle
    
    return uf.components == 1  # must be connected
```

---

## Solution 3: Edge Count + Connected

```python
def validTree(n, edges):
    if len(edges) != n - 1:
        return False
    
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    
    return uf.components == 1
```

---

## Trace Through Examples

```
Example 1: n=5, edges=[[0,1],[0,2],[0,3],[1,4]]
  
  Edge count: 4 = n-1 = 4 ✓
  
  Union(0,1): OK → {0,1}
  Union(0,2): OK → {0,1,2}
  Union(0,3): OK → {0,1,2,3}
  Union(1,4): OK → {0,1,2,3,4}
  
  No cycle ✓, Components = 1 ✓
  
  Tree:  0
        /|\
       1 2 3
       |
       4
  
  Answer: TRUE ✓

Example 2: n=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]]
  
  Edge count: 5 ≠ n-1 = 4 ✗
  
  Answer: FALSE (too many edges → must have cycle)
  
  We can also see:
  Union(0,1): OK
  Union(1,2): OK
  Union(2,3): OK
  Union(1,3): FAIL → cycle: 1─2─3─1
  
  Answer: FALSE ✓

Example 3: n=4, edges=[[0,1],[2,3]]
  
  Edge count: 2 ≠ n-1 = 3 ✗
  
  Answer: FALSE (not enough edges → disconnected)
  
  Components after processing: {0,1}, {2,3} → 2 components
  Answer: FALSE ✓
```

---

## Why Check Edge Count First?

```
Checking len(edges) == n-1 is an O(1) shortcut!

  Too few edges (< n-1):
    → Can't connect all nodes → not a tree
    
  Too many edges (> n-1):
    → Must have a cycle → not a tree
    
  Exactly n-1 edges:
    → Could be a tree — need to verify no cycle

This early check avoids processing edges unnecessarily.
```

---

## Complete Optimized Solution

```python
class Solution:
    def validTree(self, n: int, edges: list) -> bool:
        if len(edges) != n - 1:
            return False
        
        parent = list(range(n))
        
        def find(x):
            while parent[x] != x:
                parent[x] = parent[parent[x]]  # path halving
                x = parent[x]
            return x
        
        def union(x, y):
            rx, ry = find(x), find(y)
            if rx == ry:
                return False
            parent[ry] = rx
            return True
        
        return all(union(u, v) for u, v in edges)
```

```cpp
class Solution {
    vector<int> parent;
    
    int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    
public:
    bool validTree(int n, vector<vector<int>>& edges) {
        if (edges.size() != n - 1) return false;
        
        parent.resize(n);
        iota(parent.begin(), parent.end(), 0);
        
        for (auto& e : edges) {
            int rx = find(e[0]), ry = find(e[1]);
            if (rx == ry) return false;
            parent[ry] = rx;
        }
        return true;
    }
};
```

---

## Related: Minimum Edges to Make Tree

```
Problem: Given a graph with n nodes and some edges, 
what is the minimum number of edges to ADD to make 
it a tree (connected and acyclic)?

Solution:
  1. Count components using Union-Find
  2. Remove redundant edges (within components)
  3. Need (components - 1) new edges to connect all

  But: If there are redundant edges, we can "repurpose" them.
  
  Extra edges (cycles) = E - (n - components)
  Edges needed = components - 1
  
  If extra_edges >= edges_needed: possible!
  Otherwise: impossible (not enough edges total)
```

```python
def makeConnected(n, connections):
    """LeetCode 1319: Number of Operations to Make Network Connected"""
    if len(connections) < n - 1:
        return -1  # not enough cables
    
    uf = UnionFind(n)
    for u, v in connections:
        uf.union(u, v)
    
    return uf.components - 1
```

---

## Summary Table

| Check | What it Catches | Time |
|-------|----------------|------|
| len(edges) != n-1 | Wrong edge count | O(1) |
| Union fails | Cycle exists | O(E × α(n)) |
| components != 1 | Graph disconnected | O(1) |
| Any 2 of above | Complete tree validation | O(E × α(n)) |

---

## Quick Revision Questions

1. **What three properties define a valid tree?**
2. **Why is checking only two of the three conditions sufficient?**
3. **What is the O(1) shortcut before processing edges?**
4. **How do you solve "minimum operations to make connected"?**
5. **Write the complete Graph Valid Tree solution from memory.**

---

[← Previous: Redundant Connection](03-redundant-connection.md) | [Next: Implementation Approach →](05-implementation-approach.md)
