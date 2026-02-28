# Chapter 1: Union-Find Fundamentals

## 📋 Chapter Overview
Disjoint Set Union (DSU / Union-Find) data structure: representing connected components with near-O(1) union and find operations.

---

## 🧠 Core Concept

```
  Union-Find tracks GROUPS (disjoint sets).
  
  Two operations:
  1. FIND(x): which group does x belong to? (return root/representative)
  2. UNION(x, y): merge the groups containing x and y
  
  Initially: each element is its own group
  
  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
  │0│ │1│ │2│ │3│ │4│     5 separate components
  └─┘ └─┘ └─┘ └─┘ └─┘
  
  After union(0,1), union(2,3), union(0,2):
  
      0               4
     / \
    1   2              1 component of {0,1,2,3}
        |              1 component of {4}
        3
```

---

## 📐 Basic Implementation

```
class UnionFind:
    parent = array [0, 1, 2, ..., n-1]  // each is own parent
    
    find(x):
        while parent[x] ≠ x:
            x = parent[x]
        return x
    
    union(x, y):
        rootX = find(x)
        rootY = find(y)
        if rootX ≠ rootY:
            parent[rootX] = rootY
```

**Problem:** Without optimization, trees can become tall → find is O(n) worst case.

---

## 📐 Optimization 1: Path Compression

```
  During find, make every node point directly to root.
  
  Before:           After find(4):
      0                  0
      |                / | \
      1               1  2  4
      |
      2
      |
      4
  
  find(x):
      if parent[x] ≠ x:
          parent[x] = find(parent[x])   // recursive compression
      return parent[x]
```

---

## 📐 Optimization 2: Union by Rank (or Size)

```
  Always attach shorter tree under taller tree.
  
  rank = array of 0s   (approximate height)
  
  union(x, y):
      rootX = find(x)
      rootY = find(y)
      if rootX == rootY: return
      
      if rank[rootX] < rank[rootY]:
          parent[rootX] = rootY
      elif rank[rootX] > rank[rootY]:
          parent[rootY] = rootX
      else:
          parent[rootY] = rootX
          rank[rootX]++
```

### Union by Size (alternative)

```
  size = array of 1s
  
  union(x, y):
      rootX = find(x)
      rootY = find(y)
      if rootX == rootY: return false
      
      if size[rootX] < size[rootY]:
          swap(rootX, rootY)
      parent[rootY] = rootX
      size[rootX] += size[rootY]
      return true    // union happened
```

---

## 📐 Complete Optimized Template

```
class UnionFind:
    parent = [0, 1, ..., n-1]
    rank = [0, 0, ..., 0]
    count = n                    // number of components
    
    find(x):
        if parent[x] ≠ x:
            parent[x] = find(parent[x])  // path compression
        return parent[x]
    
    union(x, y):
        rootX = find(x)
        rootY = find(y)
        if rootX == rootY: return false
        
        if rank[rootX] < rank[rootY]:
            parent[rootX] = rootY
        elif rank[rootX] > rank[rootY]:
            parent[rootY] = rootX
        else:
            parent[rootY] = rootX
            rank[rootX]++
        
        count--                  // one less component
        return true
    
    connected(x, y):
        return find(x) == find(y)
    
    getCount():
        return count
```

---

## 📊 Complexity Analysis

| Operation | Without optimization | Path compression only | Both optimizations |
|-----------|---------------------|----------------------|-------------------|
| Find | O(n) | O(log n) amortized | O(α(n)) ≈ O(1) |
| Union | O(n) | O(log n) amortized | O(α(n)) ≈ O(1) |

> α(n) = inverse Ackermann function. For all practical n (up to 10^80), α(n) ≤ 4. Effectively constant.

---

## 📝 Trace: Union-Find Operations

```
  n = 5, parent = [0,1,2,3,4], rank = [0,0,0,0,0]
  
  union(0,1): find(0)=0, find(1)=1. rank equal → parent[1]=0, rank[0]=1
              parent = [0,0,2,3,4], count=4
  
  union(2,3): find(2)=2, find(3)=3. rank equal → parent[3]=2, rank[2]=1
              parent = [0,0,2,2,4], count=3
  
  union(0,3): find(0)=0, find(3)→find(2)=2.
              rank[0]=1 == rank[2]=1 → parent[2]=0, rank[0]=2
              parent = [0,0,0,2,4], count=2
  
  find(3): 3→2→0. Path compress: parent[3]=0, parent[2]=0
           parent = [0,0,0,0,4]
  
  connected(1,3): find(1)=0, find(3)=0 → true ✓
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Union-Find | Track disjoint sets; near-O(1) union and find |
| Path compression | Every node points directly to root after find |
| Union by rank | Attach shorter tree under taller tree |
| Both optimizations | O(α(n)) per operation — practically O(1) |
| Component count | Decrement on each successful union |

---

## ❓ Revision Questions

1. **What is α(n)?** → Inverse Ackermann function; grows incredibly slowly. α(10^80) = 4. Makes operations effectively O(1).
2. **Path compression: what does it do?** → Makes every node on the find path point directly to the root, flattening the tree.
3. **Union by rank vs union by size?** → Rank approximates height; size tracks exact group size. Both achieve O(α(n)). Size is useful when you need group sizes.
4. **Why return true/false from union?** → Returns whether a new union happened (roots were different). Useful for cycle detection and counting components.
5. **Initial state?** → Each element is its own parent (self-loop); rank=0; count=n components.

---

[← Previous: Hash Map — When to Apply](../11-Hash-Map-Pattern/05-when-to-apply.md) | [Next: Classic Union-Find Problems →](02-classic-problems.md)
