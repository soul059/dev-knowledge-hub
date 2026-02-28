# Chapter 2: Edge Sorting

## Overview

The **sorting step** is the bottleneck of Kruskal's algorithm — O(E log E). This chapter covers sorting strategies and alternatives.

---

## Standard Approach: Sort All Edges

```python
edges.sort(key=lambda e: e[2])  # sort by weight
```

```
Time: O(E log E) = O(E log V)
Space: O(E) for storing edges (often already given)

For most implementations, this is the default.
```

---

## Why Sort by Weight?

```
Kruskal's greedy choice: always pick the LIGHTEST edge 
that doesn't create a cycle.

Sorting ensures we examine edges in weight order.

Unsorted: might miss lighter edges
  Edge (A,B,10): take it
  Edge (A,C,1):  would have been better!

Sorted:
  Edge (A,C,1):  take it ← optimal
  Edge (A,B,10): take only if needed
```

---

## Handling Ties

```
When multiple edges have the same weight:

  Edges: (0,1,3), (2,3,3), (1,2,3)
  
  All weight 3 — order among them doesn't matter!
  
  Any order produces a valid MST with the same total weight.
  (The MST itself might differ, but the weight is the same.)

This is guaranteed by the MST uniqueness property:
  - MST is unique ⟺ all edge weights are distinct
  - With ties, multiple MSTs may exist (all optimal)
```

---

## Alternative: Partial Sort with Min-Heap

```
Instead of sorting ALL edges, use a min-heap:
  - Only extract the minimum edge when needed
  - Stop early when MST is complete (n-1 edges)

import heapq

def kruskal_heap(n, edges):
    heapq.heapify(edges)  # O(E) — heapify by weight
    
    uf = UnionFind(n)
    mst = []
    total = 0
    
    while edges and len(mst) < n - 1:
        w, u, v = heapq.heappop(edges)  # O(log E)
        if uf.union(u, v):
            mst.append((u, v, w))
            total += w
    
    return mst, total

Time: O(E + K log E) where K = edges examined
  Best case: K ≈ n-1 (no rejections) → O(E + n log E)
  Worst case: K = E → O(E log E) — same as sorting
```

When is this better?
```
Heap approach wins when MST is found early:
  - Dense graph: Many edges, but only n-1 needed
  - Random edge weights: Few rejections early on

Sorting approach wins when:
  - Sparse graph: E ≈ V, must examine most edges
  - Need sorted order for other purposes
```

---

## Alternative: Bucket Sort for Integer Weights

```
If edge weights are integers in range [0, W]:

  Bucket sort: O(E + W) instead of O(E log E)

  Buckets: [0] → edges with weight 0
           [1] → edges with weight 1
           ...
           [W] → edges with weight W

  Process buckets in order.

This is O(E + W) — better than O(E log E) when W is small!

Example: Road network with distances 1-1000
  E = 10^6, W = 1000
  Comparison sort: O(10^6 × 20) = 2×10^7
  Bucket sort:     O(10^6 + 1000) = ~10^6  ← 20x faster!
```

```python
def kruskal_bucket(n, edges, max_weight):
    # Bucket sort
    buckets = [[] for _ in range(max_weight + 1)]
    for u, v, w in edges:
        buckets[w].append((u, v))
    
    uf = UnionFind(n)
    mst = []
    total = 0
    
    for w in range(max_weight + 1):
        for u, v in buckets[w]:
            if uf.union(u, v):
                mst.append((u, v, w))
                total += w
                if len(mst) == n - 1:
                    return mst, total
    
    return mst, total
```

---

## Edge Storage Formats

```
Different input formats and how to sort:

1. Edge tuple list: [(u, v, w), ...]
   edges.sort(key=lambda e: e[2])

2. Edge object list:
   edges.sort(key=lambda e: e.weight)

3. Adjacency list → convert to edge list first:
   edges = []
   for u in range(n):
       for v, w in adj[u]:
           if u < v:  # avoid duplicates
               edges.append((u, v, w))
   edges.sort(key=lambda e: e[2])

4. Matrix → convert to edge list:
   for i in range(n):
       for j in range(i+1, n):
           if matrix[i][j] != INF:
               edges.append((i, j, matrix[i][j]))
```

---

## Optimization: Early Termination

```
Once we have n-1 edges in MST, we can stop immediately.
No need to process remaining edges.

for u, v, w in sorted_edges:
    if uf.union(u, v):
        mst_count += 1
        if mst_count == n - 1:
            break              ← STOP HERE

In sparse graphs, this can save significant work.
In dense graphs (E >> V), most edges are skipped.
```

---

## Summary Table

| Sorting Method | Time | Best When |
|---------------|------|-----------|
| Standard sort | O(E log E) | General case |
| Min-heap | O(E + K log E) | Dense graphs, early MST |
| Bucket sort | O(E + W) | Small integer weights |
| Counting sort | O(E + W) | Small weight range |
| Radix sort | O(E × d) | Fixed-length weight keys |

---

## Quick Revision Questions

1. **Why is sorting the bottleneck of Kruskal's algorithm?**
2. **When does a min-heap approach outperform full sorting?**
3. **When can bucket sort be used and what is its complexity?**
4. **Does the tie-breaking order among equal-weight edges affect MST weight?**
5. **How do you convert an adjacency list to an edge list for Kruskal's?**

---

[← Previous: Kruskal's Algorithm](01-kruskals-algorithm.md) | [Next: Union-Find Integration →](03-union-find-integration.md)
