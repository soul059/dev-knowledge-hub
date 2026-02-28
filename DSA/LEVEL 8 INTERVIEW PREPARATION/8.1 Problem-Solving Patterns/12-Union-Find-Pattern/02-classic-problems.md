# Chapter 2: Classic Union-Find Problems

## 📋 Chapter Overview
Core interview problems solved with Union-Find: number of islands, redundant connection, accounts merge, and connected components.

---

## 📝 Problem 1: Number of Islands — Union-Find Approach (LeetCode 200)

**Problem:** Given a 2D grid of '1' (land) and '0' (water), count connected land regions.

```
function numIslands(grid):
    m = rows, n = cols
    uf = UnionFind(m * n)
    count = 0
    
    for i = 0 to m-1:
        for j = 0 to n-1:
            if grid[i][j] == '1':
                count++
                // Union with right and down neighbors
                if j+1 < n and grid[i][j+1] == '1':
                    if uf.union(i*n+j, i*n+j+1):
                        count--
                if i+1 < m and grid[i+1][j] == '1':
                    if uf.union(i*n+j, (i+1)*n+j):
                        count--
    
    return count
```

### Trace

```
  grid:
  1 1 0
  0 1 0
  0 0 1
  
  Count starts at 0.
  
  (0,0)='1': count=1.
  (0,1)='1': count=2. union(0,1) → count=1.
  (1,1)='1': count=2. union(1, 4) → count=1.
  (2,2)='1': count=2. No neighbors to union.
  
  Answer: 2 (one island {(0,0),(0,1),(1,1)}, another {(2,2)})
```

**Note:** BFS/DFS is often simpler for this problem, but UF shines when updates are dynamic.

---

## 📝 Problem 2: Redundant Connection (LeetCode 684)

**Problem:** Tree with one extra edge. Find the edge that, if removed, makes it a tree.

```
function findRedundantConnection(edges):
    uf = UnionFind(n + 1)
    
    for each [u, v] in edges:
        if not uf.union(u, v):
            return [u, v]   // already connected → this edge is redundant
```

### Trace

```
  edges = [[1,2],[1,3],[2,3]]
  
  Edge [1,2]: union(1,2) → true. Components: {1,2}, {3}
  Edge [1,3]: union(1,3) → true. Components: {1,2,3}
  Edge [2,3]: union(2,3) → false! Already connected.
  
  Answer: [2, 3]
```

**Why UF is perfect:** Cycle detection in undirected graph = union two nodes already in the same component.

---

## 📝 Problem 3: Accounts Merge (LeetCode 721)

**Problem:** List of accounts [name, email1, email2, ...]. Merge accounts that share any email.

```
function accountsMerge(accounts):
    uf = UnionFind(total emails count)
    emailToId = {}    // email → unique id
    emailToName = {}  // email → account name
    id = 0
    
    // Assign IDs and union emails in same account
    for each account:
        name = account[0]
        for each email in account[1:]:
            if email not in emailToId:
                emailToId[email] = id
                id++
            emailToName[email] = name
            // Union first email with all others in account
            uf.union(emailToId[account[1]], emailToId[email])
    
    // Group emails by root
    groups = {}   // root id → list of emails
    for each (email, eid) in emailToId:
        root = uf.find(eid)
        groups[root].add(email)
    
    // Format result
    result = []
    for each (root, emails) in groups:
        sort(emails)
        result.add([emailToName[emails[0]]] + emails)
    
    return result
```

### Trace

```
  accounts:
  ["John", "john@a.com", "john@b.com"]
  ["John", "john@b.com", "john@c.com"]
  ["Mary", "mary@a.com"]
  
  emailToId: john@a=0, john@b=1, john@c=2, mary@a=3
  
  Account 1: union(0,0), union(0,1) → {john@a, john@b}
  Account 2: union(1,1), union(1,2) → {john@a, john@b, john@c}
  Account 3: mary@a stays alone → {mary@a}
  
  Result: [["John","john@a.com","john@b.com","john@c.com"],
           ["Mary","mary@a.com"]]
```

---

## 📝 Problem 4: Number of Connected Components (LeetCode 323)

**Problem:** Given n nodes and edges, find number of connected components.

```
function countComponents(n, edges):
    uf = UnionFind(n)
    
    for each [u, v] in edges:
        uf.union(u, v)
    
    return uf.getCount()
```

**Simplest UF application.** Component count decreases by 1 per successful union.

---

## 📝 Problem 5: Largest Component Size by Common Factor (LeetCode 952)

**Problem:** Group numbers sharing a common factor > 1. Find the largest group.

```
function largestComponentSize(nums):
    uf = UnionFind(max(nums) + 1)
    
    for each num in nums:
        for factor = 2 to sqrt(num):
            if num % factor == 0:
                uf.union(num, factor)
                uf.union(num, num / factor)
    
    // Count group sizes by root
    count = {}
    maxSize = 0
    for each num in nums:
        root = uf.find(num)
        count[root] = count.getOrDefault(root, 0) + 1
        maxSize = max(maxSize, count[root])
    
    return maxSize
```

**Key insight:** Union numbers with their prime factors. Numbers sharing any factor end up in the same component.

---

## 📊 Classic Problems Summary

| Problem | UF Role | Complexity |
|---------|---------|-----------|
| Number of Islands | Connect adjacent land cells | O(m×n × α) |
| Redundant Connection | Detect cycle (union fails) | O(n × α) |
| Accounts Merge | Merge overlapping email sets | O(n × α) |
| Connected Components | Count after all unions | O(n + E × α) |
| Largest by Common Factor | Union via prime factors | O(n√M × α) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Cycle detection | Union returns false → edge creates cycle |
| Dynamic connectivity | UF is ideal for incremental "add edge" queries |
| Group merging | Union items sharing any common property |
| Factor-based grouping | Union number with its factors |
| Component counting | Track count; decrement on union |

---

## ❓ Revision Questions

1. **Redundant connection: why UF?** → Adding edges one by one; if union fails (same root), that edge is the cycle-creating redundant edge.
2. **Islands: UF vs DFS?** → DFS is simpler for static grids. UF is better for dynamic (adding land cells over time).
3. **Accounts merge: core idea?** → Emails in the same account get unioned. Transitively, accounts sharing any email merge.
4. **When does union return false?** → When the two elements already have the same root (already connected).
5. **Largest component by common factor: efficiency?** → Union with factors instead of checking all pairs (O(n²) → O(n√M)).

---

[← Previous: Union-Find Fundamentals](01-union-find-fundamentals.md) | [Next: Graph Connectivity →](03-graph-connectivity.md)
