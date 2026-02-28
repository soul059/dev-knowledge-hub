# Chapter 3: Redundant Connection

## Overview

**Redundant Connection** (LeetCode 684) is a classic Union-Find problem. Given a graph that would be a tree with one extra edge, find the edge to remove.

---

## Problem Statement

```
A tree of n nodes (1 to n) has exactly n-1 edges.
You're given n edges (one extra), forming exactly one cycle.
Return the edge that can be removed to make it a tree.
If multiple answers exist, return the one that appears LAST.
```

---

## Why Union-Find Works Perfectly

```
Process edges in order. The LAST edge that connects two 
already-connected nodes is the answer.

Example: n=3, edges = [[1,2],[1,3],[2,3]]

  Edge [1,2]: Find(1)=1, Find(2)=2 → Union → OK
    1──2

  Edge [1,3]: Find(1)=1, Find(3)=3 → Union → OK
    1──2
    |
    3

  Edge [2,3]: Find(2)=1, Find(3)=1 → SAME!
    ╔══════════════════════════╗
    ║ Redundant edge: [2,3]   ║
    ╚══════════════════════════╝

    1──2
    | /
    3     ← [2,3] creates the cycle
```

---

## Solution

```python
def findRedundantConnection(edges):
    n = len(edges)
    parent = list(range(n + 1))  # 1-indexed
    rank = [0] * (n + 1)
    
    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]
    
    def union(x, y):
        rx, ry = find(x), find(y)
        if rx == ry:
            return False
        if rank[rx] < rank[ry]:
            rx, ry = ry, rx
        parent[ry] = rx
        if rank[rx] == rank[ry]:
            rank[rx] += 1
        return True
    
    for u, v in edges:
        if not union(u, v):
            return [u, v]
```

```cpp
class Solution {
    vector<int> parent, rank_;
    
    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank_[rx] < rank_[ry]) swap(rx, ry);
        parent[ry] = rx;
        if (rank_[rx] == rank_[ry]) rank_[rx]++;
        return true;
    }
    
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        parent.resize(n + 1);
        rank_.resize(n + 1, 0);
        iota(parent.begin(), parent.end(), 0);
        
        for (auto& e : edges)
            if (!unite(e[0], e[1]))
                return e;
        
        return {};
    }
};
```

---

## Detailed Trace

```
Input: edges = [[1,2],[2,3],[3,4],[1,4],[1,5]]

n = 5 (5 edges for 5 nodes → 1 extra)

Step 1: Edge [1,2]
  parent: [0,1,2,3,4,5]
  Find(1)=1, Find(2)=2 → different → Union
  parent: [0,1,1,3,4,5]
  
  Tree: 1──2   3   4   5

Step 2: Edge [2,3]
  Find(2)=1, Find(3)=3 → different → Union
  parent: [0,1,1,1,4,5]
  
  Tree: 1──2   4   5
        |
        3

Step 3: Edge [3,4]
  Find(3)=1, Find(4)=4 → different → Union
  parent: [0,1,1,1,1,5]
  
  Tree: 1──2   5
       /|
      3 4

Step 4: Edge [1,4]
  Find(1)=1, Find(4)=1 → SAME!
  ╔══════════════════════════════╗
  ║ Answer: [1,4]               ║
  ║ Cycle: 1──2...3──4──1       ║
  ╚══════════════════════════════╝
```

---

## Variant: Redundant Connection II (Directed)

```
LeetCode 685: Redundant Connection II (HARDER)

Given a DIRECTED graph that would be a rooted tree with 
one extra edge. Find the edge to remove.

Extra complexity: Directed edges can cause TWO problems:
  1. A node has TWO parents (indegree = 2)
  2. A cycle exists

Three cases:
  Case 1: No node has 2 parents → the cycle-creating edge
  Case 2: A node has 2 parents + no cycle if we remove the 
           second → remove the second parent edge
  Case 3: A node has 2 parents + cycle → remove the first 
           parent edge
```

```python
def findRedundantDirectedConnection(edges):
    n = len(edges)
    parent_arr = [0] * (n + 1)  # track incoming edges
    
    cand1 = cand2 = None
    
    # Step 1: Find node with 2 parents
    for u, v in edges:
        if parent_arr[v] == 0:
            parent_arr[v] = u
        else:
            # v has two parents: parent_arr[v] and u
            cand1 = [parent_arr[v], v]  # first edge to v
            cand2 = [u, v]             # second edge to v
            break
    
    # Step 2: Build DSU, skipping cand2 if it exists
    uf_parent = list(range(n + 1))
    
    def find(x):
        while uf_parent[x] != x:
            uf_parent[x] = uf_parent[uf_parent[x]]
            x = uf_parent[x]
        return x
    
    for u, v in edges:
        if cand2 and u == cand2[0] and v == cand2[1]:
            continue  # skip candidate 2
        
        ru, rv = find(u), find(v)
        if ru == rv:
            # Cycle detected
            if cand1 is None:
                return [u, v]  # Case 1: just a cycle
            return cand1       # Case 3: remove first parent
        uf_parent[rv] = ru
    
    return cand2  # Case 2: remove second parent
```

---

## Related Problems

```
┌──────────────────────────────────────────────────────┐
│ Problem                        │ Key Difference      │
├────────────────────────────────┼─────────────────────┤
│ Redundant Connection I (684)   │ Undirected, simple  │
│ Redundant Connection II (685)  │ Directed, 3 cases   │
│ Graph Valid Tree (261)         │ Check tree property  │
│ Number of Operations to Make   │                     │
│  Network Connected (1319)      │ Count extra edges   │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Remove one edge to form a tree |
| Solution | Union-Find, return first failed union |
| "Last" requirement | Automatically satisfied by sequential processing |
| Directed variant | Much harder — 3 separate cases |
| Time | O(n × α(n)) |
| Space | O(n) |

---

## Quick Revision Questions

1. **Why does processing edges in order give the "last" redundant edge?**
2. **How many extra edges does the problem guarantee?**
3. **What makes the directed version (LeetCode 685) harder?**
4. **What are the three cases in the directed version?**
5. **Write the undirected solution from memory.**

---

[← Previous: Edge Addition and Cycles](02-edge-addition-and-cycles.md) | [Next: Graph Valid Tree →](04-graph-valid-tree.md)
