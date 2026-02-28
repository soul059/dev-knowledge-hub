# Chapter 2: Union-Find with Rollback

## Overview

Standard Union-Find with path compression is **irreversible** — once you flatten paths, you can't undo it. **Union-Find with rollback** trades path compression for the ability to **undo** Union operations, supporting offline and divide-and-conquer algorithms.

---

## Why Rollback?

```
Scenarios requiring undo:
  1. Offline D&C: Process queries in special order, 
     undoing unions at subproblem boundaries
  2. Time-travel queries: "What was the state at time t?"
  3. Interactive problems: "What if we undo the last merge?"

Standard DSU: Path compression modifies the tree irreversibly
Rollback DSU: Union by rank WITHOUT path compression
              → can undo by restoring old parent and rank values
```

---

## Key Design Choice

```
┌─────────────────────────────────────────────────────┐
│  For rollback to work:                               │
│    ✓ Use Union by Rank (for O(log n) guarantee)     │
│    ✗ Do NOT use path compression                     │
│                                                      │
│  Without path compression:                           │
│    Find = O(log n) per operation (not α(n))          │
│    But rollback is possible!                         │
└─────────────────────────────────────────────────────┘
```

---

## Implementation

```python
class DSURollback:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.history = []  # stack of (node, old_parent, old_rank)
        self.components = n
    
    def find(self, x):
        """Find WITHOUT path compression."""
        while self.parent[x] != x:
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        """Union by rank, recording history for rollback."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            self.history.append(None)  # no-op marker
            return False
        
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        
        # Record old state BEFORE modifying
        self.history.append((ry, self.parent[ry], self.rank[rx]))
        
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1
        return True
    
    def rollback(self):
        """Undo the last Union operation."""
        if not self.history:
            return
        
        entry = self.history.pop()
        if entry is None:
            return  # was a no-op (same set union)
        
        ry, old_parent, old_rank_rx = entry
        rx = self.parent[ry]  # current parent
        
        self.rank[rx] = old_rank_rx
        self.parent[ry] = old_parent
        self.components += 1
    
    def save(self):
        """Save current state (returns checkpoint)."""
        return len(self.history)
    
    def restore(self, checkpoint):
        """Rollback to a saved checkpoint."""
        while len(self.history) > checkpoint:
            self.rollback()
```

---

## Trace Example

```
n = 5

Step 1: Union(0, 1)
  history: [(1, 1, 0)]  ← saved: node=1, old_parent=1, old_rank=0
  parent = [0, 0, 2, 3, 4]
  rank = [1, 0, 0, 0, 0]

Step 2: Union(2, 3)
  history: [(1,1,0), (3,3,0)]
  parent = [0, 0, 2, 2, 4]
  rank = [1, 0, 1, 0, 0]

Step 3: checkpoint = save()  → checkpoint = 2

Step 4: Union(0, 2)
  history: [(1,1,0), (3,3,0), (2,2,1)]
  parent = [0, 0, 0, 2, 4]
  rank = [1, 0, 1, 0, 0]  (rank stays 1 since rank[0]=1 > rank[2]=1? 
                            No: equal → rank[0] becomes 2)

  Actually: rank[0]=1, rank[2]=1 → equal → rank[0] = 2
  parent = [0, 0, 0, 2, 4]
  rank = [2, 0, 1, 0, 0]
  history: [(1,1,0), (3,3,0), (2,2,1)]  ← old_rank_rx = 1

Step 5: restore(checkpoint)  → undo back to checkpoint=2
  Rollback Union(0,2):
    ry=2, old_parent=2, old_rank_rx=1
    parent[2] = 2  (was 0, restore to 2)
    rank[0] = 1    (was 2, restore to 1)
    components += 1

  State restored:
  parent = [0, 0, 2, 2, 4]
  rank = [1, 0, 1, 0, 0]
  
  {0,1} {2,3} {4}  ← back to checkpoint state ✓
```

---

## Application: Offline Dynamic Connectivity (D&C)

```
Problem: Given edges with lifetimes [L, R), answer 
connectivity queries at each time step.

Approach: Divide & Conquer on time + rollback DSU

  function solve(time_L, time_R, active_edges):
      checkpoint = dsu.save()
      
      // Add edges active throughout [L, R)
      for each edge active in [L, R):
          dsu.union(edge.u, edge.v)
      
      if time_L + 1 == time_R:
          // Single time step — answer query
          answer_query(time_L)
      else:
          mid = (time_L + time_R) / 2
          solve(time_L, mid, left_edges)
          solve(mid, time_R, right_edges)
      
      dsu.restore(checkpoint)  // ← ROLLBACK!
```

---

## Complexity

```
┌─────────────────┬────────────────┬─────────────────┐
│ Operation       │ With Rollback  │ Standard DSU    │
├─────────────────┼────────────────┼─────────────────┤
│ Find            │ O(log n)       │ O(α(n))         │
│ Union           │ O(log n)       │ O(α(n))         │
│ Rollback        │ O(1)           │ N/A             │
│ Save            │ O(1)           │ N/A             │
│ Restore(k ops)  │ O(k)           │ N/A             │
│ Space           │ O(n + ops)     │ O(n)            │
└─────────────────┴────────────────┴─────────────────┘

The O(log n) instead of O(α(n)) is the price of rollback.
```

---

## C++ Implementation

```cpp
struct DSURollback {
    vector<int> parent, rnk;
    vector<pair<int,int>> history; // (node, old_value)
    // We store pairs: what was changed and its old value
    
    DSURollback(int n) : parent(n), rnk(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        while (parent[x] != x) x = parent[x];
        return x;
    }
    
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (rnk[x] < rnk[y]) swap(x, y);
        
        // Save: parent[y] changes, possibly rnk[x]
        history.push_back({y, parent[y]});
        history.push_back({~x, rnk[x]}); // ~x to distinguish rank entries
        
        parent[y] = x;
        if (rnk[x] == rnk[y]) rnk[x]++;
        return true;
    }
    
    int save() { return history.size(); }
    
    void rollback(int checkpoint) {
        while ((int)history.size() > checkpoint) {
            auto [node, val] = history.back();
            history.pop_back();
            if (node < 0) rnk[~node] = val;  // rank entry
            else parent[node] = val;          // parent entry
        }
    }
};
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| No path compression | Required for rollback |
| Union by rank | Required for O(log n) |
| History stack | Stores old parent/rank values |
| Rollback | Pop history, restore values |
| Checkpoint | Save history length |
| Time trade-off | O(log n) vs O(α(n)) |
| Use case | Offline D&C, time-travel queries |

---

## Quick Revision Questions

1. **Why can't we use path compression with rollback?**
2. **What does the history stack store?**
3. **How do save() and restore() work?**
4. **What is the time complexity per operation with rollback DSU?**
5. **Describe the offline dynamic connectivity D&C approach.**

---

[← Previous: Weighted Union-Find](01-weighted-union-find.md) | [Next: Offline Queries →](03-offline-queries.md)
