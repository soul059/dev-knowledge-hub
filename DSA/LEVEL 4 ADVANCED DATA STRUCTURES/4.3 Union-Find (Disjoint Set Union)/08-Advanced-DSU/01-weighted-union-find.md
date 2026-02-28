# Chapter 1: Weighted Union-Find

## Overview

**Weighted Union-Find** (also called **Union-Find with weights** or **potentials**) extends DSU to track a **relative weight/distance** between each node and its root. This enables answering queries like "What is the relationship between x and y within the same set?"

---

## The Idea

```
Standard DSU: Tracks only "are x and y in the same set?"
Weighted DSU: Also tracks "what is the difference/distance 
              between x and y?"

Each node stores a weight relative to its parent:
  weight[x] = relationship between x and parent[x]

By summing weights along the path to root:
  potential[x] = total weight from x to root

Relationship between x and y:
  diff(x, y) = potential[x] - potential[y]
```

---

## Visual Example

```
Problem: "A is 3 heavier than B", "B is 2 heavier than C"
         → "How much heavier is A than C?"

Weighted DSU:
              root (potential = 0)
             /    \
          B(+3)   D(+1)
          |
         A(+3)  ← weight[A] = 3 (A is 3 MORE than parent B)
         
  potential[A] = weight[A] + weight[B] = 3 + 3 = 6
  potential[B] = weight[B] = 3
  potential[C] = 0 (root)
  
  A relative to B: potential[A] - potential[B] = 6 - 3 = 3
  A relative to C: potential[A] - potential[C] = 6 - 0 = 6
  
Wait, let me redo this more carefully:

  "A is 3 heavier than B": w(A) - w(B) = 3
  "B is 2 heavier than C": w(B) - w(C) = 2
  → w(A) - w(C) = 3 + 2 = 5
  
  DSU representation (C = root):
    C (rank[C] = potential 0)
    |
    B (rank = weight 2 relative to C)
    |
    A (rank = weight 3 relative to B)
    
  potential[A] = 3 + 2 = 5 (distance from A to root C)
  potential[B] = 2 (distance from B to root C)
  
  diff(A, B) = potential[A] - potential[B] = 5 - 2 = 3 ✓
  diff(A, C) = potential[A] - potential[C] = 5 - 0 = 5 ✓
```

---

## Data Structure

```python
class WeightedUnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.weight = [0] * n  # weight[x] = distance from x to parent[x]
    
    def find(self, x):
        """Returns (root, potential of x relative to root)."""
        if self.parent[x] == x:
            return x, 0
        root, w = self.find(self.parent[x])
        self.parent[x] = root
        self.weight[x] += w  # accumulate weight to root
        return root, self.weight[x]
    
    def union(self, x, y, w):
        """
        Declare: potential[x] - potential[y] = w
        (i.e., x is w more than y)
        Returns False if contradicts existing info.
        """
        rx, wx = self.find(x)  # wx = potential[x]
        ry, wy = self.find(y)  # wy = potential[y]
        
        if rx == ry:
            # Already same set — check consistency
            return (wx - wy) == w
        
        # Merge: need weight[ry] such that
        # potential[x] - potential[y] = w
        # wx + 0 - (wy + weight[ry→rx]) = w
        # We want weight from ry to rx
        if self.rank[rx] < self.rank[ry]:
            # rx goes under ry
            self.parent[rx] = ry
            self.weight[rx] = wy - wx + w
            # Verify: potential[x] = wx + weight[rx] = wx + (wy - wx + w) = wy + w
            # potential[y] = wy
            # diff = wy + w - wy = w ✓
        else:
            # ry goes under rx
            self.parent[ry] = rx
            self.weight[ry] = wx - wy - w
            # Verify: potential[y] = wy + weight[ry] = wy + (wx - wy - w) = wx - w
            # potential[x] = wx
            # diff = wx - (wx - w) = w ✓
            if self.rank[rx] == self.rank[ry]:
                self.rank[rx] += 1
        
        return True
    
    def diff(self, x, y):
        """Return potential[x] - potential[y], or None if different sets."""
        rx, wx = self.find(x)
        ry, wy = self.find(y)
        if rx != ry:
            return None  # different sets
        return wx - wy
```

---

## Classic Problem: Evaluate Division (LeetCode 399)

```
Given equations like a/b = 2.0, b/c = 3.0
Answer queries like a/c = ?, a/a = ?, x/y = ?

This is weighted Union-Find where:
  weight = log(ratio) or directly ratio
  
  a/b = 2.0 → potential[a] / potential[b] = 2.0
  b/c = 3.0 → potential[b] / potential[c] = 3.0
  
  a/c = (a/b) × (b/c) = 2.0 × 3.0 = 6.0

Using multiplicative weights:
  weight[x] = value[x] / value[parent[x]]
  potential[x] = product of weights to root
```

```python
def calcEquation(equations, values, queries):
    # Map variable names to indices
    var_map = {}
    idx = 0
    for (a, b), v in zip(equations, values):
        if a not in var_map: var_map[a] = idx; idx += 1
        if b not in var_map: var_map[b] = idx; idx += 1
    
    n = idx
    parent = list(range(n))
    weight = [1.0] * n  # multiplicative weight
    
    def find(x):
        if parent[x] != x:
            root = find(parent[x])
            weight[x] *= weight[parent[x]]
            parent[x] = root
        return parent[x]
    
    def union(x, y, w):
        rx, ry = find(x), find(y)
        if rx == ry: return
        parent[rx] = ry
        weight[rx] = w * weight[y] / weight[x]
    
    # Process equations
    for (a, b), v in zip(equations, values):
        union(var_map[a], var_map[b], v)
    
    # Answer queries
    results = []
    for a, b in queries:
        if a not in var_map or b not in var_map:
            results.append(-1.0)
        elif find(var_map[a]) != find(var_map[b]):
            results.append(-1.0)
        else:
            results.append(weight[var_map[a]] / weight[var_map[b]])
    
    return results
```

---

## Key Insight: Path Compression with Weights

```
During path compression, weights must be ACCUMULATED:

Before compression:
  A --w1--> B --w2--> C (root)
  
  potential[A] = w1 + w2

After compression:
  A ---(w1+w2)---> C (root)
  B ---w2---------> C (root)
  
  potential[A] = w1 + w2 (same!)

Code:
  def find(x):
      if parent[x] != x:
          root = find(parent[x])
          weight[x] += weight[parent[x]]  # accumulate!
          parent[x] = root
      return parent[x]
```

---

## Summary Table

| Aspect | Standard DSU | Weighted DSU |
|--------|-------------|-------------|
| Stores | Connectivity | Connectivity + relationships |
| weight[x] | N/A | Distance x → parent |
| Find returns | root | root + potential |
| Union(x,y) | Merge sets | Merge with constraint |
| Query | Same set? | Same set? + difference |
| Applications | Components | Ratios, distances, potentials |

---

## Quick Revision Questions

1. **What does `weight[x]` represent in weighted DSU?**
2. **How do you compute the difference between two nodes?**
3. **How does path compression affect weights?**
4. **How does the Evaluate Division problem map to weighted DSU?**
5. **What does a contradictory Union return?**

---

[← Previous: MST Implementation](../07-MST-With-Union-Find/05-implementation-details.md) | [Next: Union-Find with Rollback →](02-union-find-with-rollback.md)
