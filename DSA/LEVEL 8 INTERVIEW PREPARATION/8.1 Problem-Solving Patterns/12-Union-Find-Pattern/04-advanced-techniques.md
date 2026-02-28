# Chapter 4: Advanced Techniques

## 📋 Chapter Overview
Advanced Union-Find techniques: weighted UF, offline processing, dynamic graph problems, and combining UF with other data structures.

---

## 📝 Technique 1: Weighted Union-Find

**Concept:** Store extra weight/relationship along edges. Track relative differences between nodes.

```
  Used for:
  - Potential/distance between nodes
  - Ratio relationships (a/b = 2.0)
  - Relative ordering / ranking
```

### Evaluate Division (LeetCode 399)

**Problem:** Given equations a/b = 2.0, b/c = 3.0, answer queries like a/c = ?

```
  Model as weighted graph:
  a/b = 2.0  →  a ---2.0--→ b  (and b ---0.5--→ a)
  b/c = 3.0  →  b ---3.0--→ c  (and c ---0.33-→ b)
  
  Query a/c:
  a → root: weight(a→root)
  c → root: weight(c→root)
  Answer = weight(a→root) / weight(c→root)
```

### Weighted UF Template

```
class WeightedUnionFind:
    parent = [0..n-1]
    rank   = [0..n-1]
    weight = [1.0, 1.0, ..., 1.0]
    // weight[i] = node_i / parent[i]
    
    function find(x):
        if parent[x] ≠ x:
            root = find(parent[x])
            weight[x] *= weight[parent[x]]
            parent[x] = root
        return parent[x]
    
    function union(x, y, w):
        // w means x / y = w
        rootX = find(x)
        rootY = find(y)
        if rootX == rootY: return
        
        // weight[x] * ? = w * weight[y]
        // We need rootX → rootY with weight ratio
        parent[rootX] = rootY
        weight[rootX] = w * weight[y] / weight[x]
    
    function query(x, y):
        if find(x) ≠ find(y): return -1.0
        return weight[x] / weight[y]
```

### Trace

```
  a/b = 2.0  →  union(a, b, 2.0)
  b/c = 3.0  →  union(b, c, 3.0)
  
  After union(a,b,2.0):
    parent[a] = b, weight[a] = 2.0   (a/b = 2.0)
    
  After union(b,c,3.0):
    parent[b] = c, weight[b] = 3.0   (b/c = 3.0)
    
  State:  a --(2.0)-→ b --(3.0)-→ c
  
  Query a/c:
    find(a): a→b→c, weight[a] = 2.0 * 3.0 = 6.0  (path compressed)
    find(c): c is root, weight[c] = 1.0
    Result = 6.0 / 1.0 = 6.0 ✓   (a/c = 2*3 = 6)
    
  Query c/a:
    weight[c] / weight[a] = 1.0 / 6.0 ≈ 0.167 ✓
```

---

## 📝 Technique 2: UF with Coordinate Compression

**Problem:** Surrounded Regions (LeetCode 130) — Flip all 'O' not on border.

```
function solve(board):
    m, n = dimensions
    uf = UnionFind(m * n + 1)
    DUMMY = m * n   // virtual node for border-connected Os
    
    for i, j in all cells:
        if board[i][j] == 'O':
            if on border:
                uf.union(i*n + j, DUMMY)
            else:
                // union with adjacent O cells
                for each neighbor (ni, nj):
                    if board[ni][nj] == 'O':
                        uf.union(i*n + j, ni*n + nj)
    
    for i, j in all cells:
        if board[i][j] == 'O':
            if find(i*n + j) ≠ find(DUMMY):
                board[i][j] = 'X'
```

```
  2D → 1D index:  cell(i,j) → i * cols + j
  
  Board:           After:
  X X X X          X X X X
  X O O X    →     X X X X
  X X O X          X X X X
  X O X X          X O X X
  
  Border O at (3,1) → connected to DUMMY
  Interior Os at (1,1),(1,2),(2,2) → not connected to DUMMY → flip
```

---

## 📝 Technique 3: Offline Processing with Union-Find

**Concept:** When queries can be answered out of order, sort them strategically.

### Processing Queries by Threshold

```
  Problem: "Can u reach v using edges with weight ≤ threshold?"
  
  Approach:
  1. Sort edges by weight ascending
  2. Sort queries by threshold ascending
  3. Process queries in order, adding edges as threshold increases
  
  function processQueries(n, edges, queries):
      sort edges by weight
      sort queries by threshold (keep original index)
      uf = UnionFind(n)
      
      edgeIdx = 0
      answers = array of size |queries|
      
      for each (u, v, threshold, queryIdx) in sorted queries:
          while edgeIdx < |edges| and edges[edgeIdx].weight ≤ threshold:
              uf.union(edges[edgeIdx].u, edges[edgeIdx].v)
              edgeIdx++
          answers[queryIdx] = (find(u) == find(v))
      
      return answers
```

```
  Visualization:
  
  Queries sorted by threshold:
  Q1(t=2) → add edges ≤ 2, answer Q1
  Q2(t=5) → add edges ≤ 5, answer Q2    (incremental!)
  Q3(t=9) → add edges ≤ 9, answer Q3
  
  Each edge added at most once → O(E α(n)) total
```

---

## 📝 Technique 4: Counting Components Properties

### Largest Component Size (LeetCode 952)

**Problem:** Group numbers sharing a common factor > 1. Find the largest group.

```
function largestComponentSize(A):
    uf = UnionFind(max(A) + 1)
    
    for each num in A:
        for factor in primeFactors(num):
            uf.union(num, factor)
    
    // Count size of each component among elements of A
    counter = HashMap
    maxSize = 0
    for each num in A:
        root = uf.find(num)
        counter[root]++
        maxSize = max(maxSize, counter[root])
    
    return maxSize

function primeFactors(n):
    factors = []
    d = 2
    while d * d ≤ n:
        if n % d == 0:
            factors.add(d)
            while n % d == 0: n /= d
        d++
    if n > 1: factors.add(n)
    return factors
```

### Trace

```
  A = [4, 6, 15, 35]
  
  4  → factors: {2}     → union(4, 2)
  6  → factors: {2, 3}  → union(6, 2), union(6, 3)
  15 → factors: {3, 5}  → union(15, 3), union(15, 5)
  35 → factors: {5, 7}  → union(35, 5), union(35, 7)
  
  Components:
  {2, 4, 6, 3, 15, 5, 35, 7}  ← all connected!
  
  Answer: 4 (all elements connected)
  
  Chain: 4-2-6-3-15-5-35
```

---

## 📝 Technique 5: Union-Find on Strings

### Similar String Groups (LeetCode 839)

```
function numSimilarGroups(strs):
    n = len(strs)
    uf = UnionFind(n)
    
    for i = 0 to n-1:
        for j = i+1 to n-1:
            if areSimilar(strs[i], strs[j]):
                uf.union(i, j)
    
    return uf.getCount()

function areSimilar(a, b):
    diff = 0
    for k = 0 to len(a)-1:
        if a[k] ≠ b[k]: diff++
        if diff > 2: return false
    return diff == 0 or diff == 2
```

```
  Similar = identical or differ in exactly 2 positions (swap).
  
  "tars" ↔ "rats"  differ at pos 0,1 → similar (swap t↔r)
  "tars" ↔ "arts"  differ at 0,1,2  → NOT similar
  "rats" ↔ "star"  differ at 0,1,2,3 → NOT similar
```

---

## 📊 Advanced Techniques Summary

| Technique | Use Case | Extra State |
|-----------|----------|-------------|
| Weighted UF | Ratio/distance between nodes | weight[] array |
| Coordinate compression | 2D grid → 1D + virtual node | dummy node |
| Offline processing | Batch queries by threshold | Sort edges + queries |
| Factor-based grouping | Numbers sharing common factor | Prime factorization |
| String similarity | Group similar strings | Pairwise comparison |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Weighted UF | Track ratios along parent edges; multiply during path compression |
| Virtual/dummy node | Connect special-case elements to one dummy node |
| Offline queries | Sort queries + edges together; process incrementally |
| Prime factor union | Union numbers with their factors; O(n√max) |
| Two-pass pattern | Build groups first, then query/classify |

---

## ❓ Revision Questions

1. **How does path compression work in weighted UF?** → When compressing path x→...→root, multiply weights along the path so weight[x] directly represents x/root.
2. **Why use a dummy node in surrounded regions?** → All border-connected 'O's unioned to one dummy node. After processing, any 'O' not in dummy's component gets flipped.
3. **Offline processing advantage?** → Avoids rebuilding UF per query. Sort queries by threshold, add edges incrementally. Each edge processed once.
4. **Prime factor grouping complexity?** → O(n × √max) for factorization + O(n × factors × α(n)) for unions.
5. **Weighted UF for equation queries: what does weight[x] represent?** → The ratio value(x) / value(root). For query x/y, compute weight[x] / weight[y].

---

[← Previous: Graph Connectivity](03-graph-connectivity.md) | [Next: When to Apply →](05-when-to-apply.md)
