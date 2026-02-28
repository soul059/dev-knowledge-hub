# Chapter 4: Size of Components

## Overview

With **Union by Size**, querying the number of elements in any component is an O(1) operation. This is useful in many problems that ask about group sizes.

---

## How It Works

```
In Union by Size:
  size[root] = number of elements in that set

Query: "How many elements in the set containing x?"
Answer: size[Find(x)]

  Find(x) gives the root → O(α(n))
  size[root] is a direct lookup → O(1)
  Total: O(α(n))
```

---

## Trace Example

```
n = 7
Operations: Union(0,1), Union(2,3), Union(4,5), Union(0,2), Union(5,6)

After Union(0,1):    size = [2,1,1,1,1,1,1]
  {0,1} {2} {3} {4} {5} {6}
  
After Union(2,3):    size = [2,1,2,1,1,1,1]
  {0,1} {2,3} {4} {5} {6}
  
After Union(4,5):    size = [2,1,2,1,2,1,1]
  {0,1} {2,3} {4,5} {6}
  
After Union(0,2):    size = [4,1,2,1,2,1,1]
  {0,1,2,3} {4,5} {6}
  
After Union(5,6):    size = [4,1,2,1,3,1,1]
  {0,1,2,3} {4,5,6}

Queries:
  getSize(3) → Find(3)=0 → size[0] = 4
  getSize(6) → Find(6)=4 → size[4] = 3
  getSize(1) → Find(1)=0 → size[0] = 4
```

---

## Finding the Largest Component

```python
def largest_component(uf):
    """Find the size of the largest component."""
    max_size = 0
    for i in range(len(uf.parent)):
        if uf.parent[i] == i:  # only check roots
            max_size = max(max_size, uf.size[i])
    return max_size
```

Or track it dynamically:
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.max_size = 1          # ← Track max
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False
        if self.size[rx] < self.size[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        self.size[rx] += self.size[ry]
        self.max_size = max(self.max_size, self.size[rx])  # ← Update
        return True
```

---

## Problem: Largest Component Size by Common Factor

```
LeetCode 952: Largest Component Size by Common Factor

Connect numbers that share a common factor > 1.
Find the largest connected component.

Input: [4, 6, 15, 35]

  4 = 2²       → factor 2
  6 = 2 × 3    → factors 2, 3
  15 = 3 × 5   → factors 3, 5
  35 = 5 × 7   → factors 5, 7

  4 ──(2)── 6 ──(3)── 15 ──(5)── 35

  All connected! Size = 4
```

```python
def largestComponentSize(nums):
    uf = UnionFind(max(nums) + 1)
    
    for num in nums:
        # Connect num with all its prime factors
        d = 2
        temp = num
        while d * d <= temp:
            if temp % d == 0:
                uf.union(num, d)
                while temp % d == 0:
                    temp //= d
            d += 1
        if temp > 1:
            uf.union(num, temp)
    
    # Count elements in each component
    count = {}
    for num in nums:
        root = uf.find(num)
        count[root] = count.get(root, 0) + 1
    
    return max(count.values())
```

---

## Problem: Process Queries After Removing Stars

```
Given n nodes and edges, remove nodes one at a time.
After each removal, report the largest component size.

Trick: Process in REVERSE! (additions instead of removals)

Forward: Remove nodes → components split (hard!)
Reverse: Add nodes → components merge (DSU!)

Example:
  n=5, edges: (0,1),(1,2),(2,3),(3,4)
  Remove order: [2, 0, 4, 1, 3]
  
  Reverse (add order): [3, 1, 4, 0, 2]
  
  Add 3:  {3}           max_size = 1
  Add 1:  {1} {3}       max_size = 1
  Add 4:  {1} {3,4}     max_size = 2  (edge 3-4)
  Add 0:  {0,1} {3,4}   max_size = 2  (edge 0-1)
  Add 2:  {0,1,2,3,4}   max_size = 5  (edges 1-2, 2-3)
  
  Reverse result: [5, 2, 2, 1, 1] → answer for removals
```

---

## Tracking All Component Sizes

```python
class UnionFindWithSizeDistribution:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        # Distribution: how many components of each size
        self.size_count = {1: n}
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False
        
        old_size_x = self.size[rx]
        old_size_y = self.size[ry]
        
        if old_size_x < old_size_y:
            rx, ry = ry, rx
            old_size_x, old_size_y = old_size_y, old_size_x
        
        self.parent[ry] = rx
        new_size = old_size_x + old_size_y
        self.size[rx] = new_size
        
        # Update distribution
        self.size_count[old_size_x] -= 1
        if self.size_count[old_size_x] == 0:
            del self.size_count[old_size_x]
        self.size_count[old_size_y] -= 1
        if self.size_count[old_size_y] == 0:
            del self.size_count[old_size_y]
        self.size_count[new_size] = self.size_count.get(new_size, 0) + 1
        
        return True
```

---

## Summary Table

| Operation | How | Time |
|-----------|-----|------|
| Size of set containing x | size[Find(x)] | O(α(n)) |
| Largest component | Track max_size during Union | O(1) |
| All component sizes | size_count dictionary | O(1) lookup |
| Number of components of size k | size_count[k] | O(1) |

---

## Quick Revision Questions

1. **Where is the component size stored?**
2. **Why must you call Find before reading the size?**
3. **How can you track the largest component efficiently?**
4. **Describe the "reverse processing" trick for node removal problems.**
5. **What extra data structure helps track the distribution of component sizes?**

---

[← Previous: Is-Connected Queries](03-is-connected-queries.md) | [Next: Connected Components in Graph →](05-connected-components-in-graph.md)
