# Chapter 4: Minimum Spanning Forest

## Overview

When a graph is **disconnected**, there is no spanning tree. Instead, Kruskal's algorithm produces a **Minimum Spanning Forest (MSF)** — a collection of MSTs, one for each connected component.

---

## What is a Spanning Forest?

```
A SPANNING FOREST of a graph with k components is:
  - A spanning tree for each component
  - Total edges: n - k  (where k = number of components)

Example:
  Graph with 3 components:
  
  Component 1: 0──1──2      (3 nodes, MST has 2 edges)
  Component 2: 3──4          (2 nodes, MST has 1 edge)
  Component 3: 5              (1 node, MST has 0 edges)
  
  MSF = MST₁ + MST₂ + MST₃
  Total MSF edges = 2 + 1 + 0 = 3 = 6 - 3
```

---

## Kruskal's Naturally Produces MSF

```
Kruskal's algorithm doesn't need modification for 
disconnected graphs — it naturally produces the MSF!

Why? Union-Find connects components greedily.
If two nodes can't be connected (no edge between 
their components), they simply stay separate.

After processing all edges:
  - MST edges < n-1 → disconnected (MSF)
  - MST edges = n-1 → connected (MST)
```

---

## Implementation

```python
def minimum_spanning_forest(n, edges):
    """
    Returns the MSF of a possibly disconnected graph.
    Works identically to Kruskal's.
    """
    edges.sort(key=lambda e: e[2])
    uf = UnionFind(n)
    
    msf_edges = []
    msf_weight = 0
    
    for u, v, w in edges:
        if uf.union(u, v):
            msf_edges.append((u, v, w))
            msf_weight += w
    
    num_components = uf.components
    
    return {
        'edges': msf_edges,
        'weight': msf_weight,
        'components': num_components,
        'is_connected': num_components == 1,
        'trees': len(msf_edges)  # = n - num_components
    }
```

---

## Example: Disconnected Graph

```
n = 7, edges:
  (0,1,2), (0,2,4), (1,2,1),  ← Component {0,1,2}
  (3,4,3), (3,5,5), (4,5,2),  ← Component {3,4,5}
  (6 is isolated)              ← Component {6}

Sorted: (1,2,1), (0,1,2), (4,5,2), (3,4,3), (0,2,4), (3,5,5)

Process:
  (1,2,1): Union(1,2) → OK, MSF += 1
  (0,1,2): Union(0,1) → OK, MSF += 2
  (4,5,2): Union(4,5) → OK, MSF += 2
  (3,4,3): Union(3,4) → OK, MSF += 3
  (0,2,4): Union(0,2) → SAME set → skip
  (3,5,5): Union(3,5) → SAME set → skip

MSF edges: 4 = 7 - 3 components

Forest:
  Tree 1:  1──2    (w=1)     Tree 2:  4──5   (w=2)
           |                          |
           0  (w=2)                   3   (w=3)
           
  Tree 3:  6  (isolated)
  
  Total MSF weight: 1 + 2 + 2 + 3 = 8
```

---

## Applications of MSF

```
1. Network design with multiple islands
   → Minimum cost to connect each island internally
   
2. Clustering
   → Remove the k-1 most expensive MST edges
   → Get k clusters (min-cut based)
   
3. Image segmentation
   → Pixels are nodes, edge weights = color difference
   → MSF components = segments
   
4. Reducing communication costs
   → Each component's MST = cheapest internal network
```

---

## MST-Based Clustering

```
Given n points, find k clusters minimizing inter-cluster 
distance.

Algorithm:
  1. Compute MST (or MSF)
  2. Remove the k-1 heaviest edges
  3. Remaining k trees = k clusters

Example: 6 points, k=3 clusters

MST edges (sorted by weight):
  (A,B,1), (B,C,2), (D,E,3), (C,D,8), (E,F,4)

Remove 2 heaviest: (C,D,8) and (E,F,4)

Clusters: {A,B,C}, {D,E}, {F}

  ┌─────────────────────────────────────────────┐
  │  The (k-1)th heaviest MST edge weight       │
  │  = the "spacing" of the k-clustering        │
  │  This is the OPTIMAL k-clustering!          │
  └─────────────────────────────────────────────┘
```

```python
def cluster(n, edges, k):
    """Find k clusters using MST."""
    edges.sort(key=lambda e: e[2])
    uf = UnionFind(n)
    
    for u, v, w in edges:
        if uf.components == k:
            break  # stop at k clusters
        uf.union(u, v)
    
    # Extract clusters
    clusters = {}
    for node in range(n):
        root = uf.find(node)
        if root not in clusters:
            clusters[root] = []
        clusters[root].append(node)
    
    return list(clusters.values())
```

---

## Checking If MST or MSF

```python
# After running Kruskal's:

if uf.components == 1:
    print("MST found — graph is connected")
    print(f"MST weight: {total_weight}")
else:
    print(f"MSF found — {uf.components} components")
    print(f"Cannot form MST (disconnected)")
    print(f"MSF weight: {total_weight}")
    print(f"Need {uf.components - 1} more edges to connect")
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| MSF definition | Separate MST for each component |
| MSF edges | n - k (k = components) |
| Algorithm | Same as Kruskal's |
| Modified code? | No — Kruskal's naturally handles it |
| Detection | mst_edges < n-1 → disconnected |
| Application | Clustering, network design |

---

## Quick Revision Questions

1. **What is the difference between MST and MSF?**
2. **How many edges does an MSF have?**
3. **Does Kruskal's need modification for disconnected graphs?**
4. **How do you use MST for k-clustering?**
5. **How do you detect whether the result is an MST or MSF?**

---

[← Previous: Union-Find Integration](03-union-find-integration.md) | [Next: Implementation Details →](05-implementation-details.md)
