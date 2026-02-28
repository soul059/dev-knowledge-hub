# Chapter 4: Small-to-Large Merging

## Overview

**Small-to-Large Merging** (also called **DSU on tree** or **weighted merge heuristic**) extends Union-Find to maintain auxiliary data structures (like sets) per component, merging the smaller into the larger to ensure O(n log n) total work.

---

## The Idea

```
Problem: Each component maintains a SET of elements.
When merging, combine the sets.

Naive: Move all elements from one set to another → O(n) per merge

Smart: Always move from SMALLER to LARGER → O(n log n) total!

Why? Each element is moved at most O(log n) times.
  When moved, its set size at least DOUBLES.
  Starting from 1, doubling log₂(n) times → n.
  
  Total moves across all elements: n × O(log n) = O(n log n)
```

---

## Visual Example

```
Merge sets: {A,B} and {C,D,E,F}

BAD: Move larger into smaller (4 moves)
  {A,B} ← C, D, E, F → {A,B,C,D,E,F}
  4 elements moved

GOOD: Move smaller into larger (2 moves)
  {C,D,E,F} ← A, B → {A,B,C,D,E,F}
  2 elements moved

Always move the SMALLER one!
```

---

## Implementation

```python
class DSUWithSets:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        # Each root maintains a set of elements
        self.members = [set([i]) for i in range(n)]
    
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return
        
        # ALWAYS merge smaller into larger
        if len(self.members[rx]) < len(self.members[ry]):
            rx, ry = ry, rx
        
        # Move all elements from ry's set to rx's set
        for elem in self.members[ry]:
            self.members[rx].add(elem)
            # Can also process elem here (update data structures)
        
        self.members[ry] = set()  # clear smaller set
        self.parent[ry] = rx
    
    def get_members(self, x):
        return self.members[self.find(x)]
```

---

## Application: Accounts Merge

```
LeetCode 721: Accounts Merge

Each account has a name and list of emails.
Two accounts belong to the same person if they share 
any email. Merge them.

Approach:
  1. Map emails to account indices
  2. Union accounts that share emails
  3. Use small-to-large to merge email sets
  4. Output merged accounts
```

```python
def accountsMerge(accounts):
    n = len(accounts)
    parent = list(range(n))
    rank = [0] * n
    emails = [set(acc[1:]) for acc in accounts]
    
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    
    def union(x, y):
        rx, ry = find(x), find(y)
        if rx == ry: return
        # Small-to-large: merge smaller email set into larger
        if len(emails[rx]) < len(emails[ry]):
            rx, ry = ry, rx
        emails[rx] |= emails[ry]
        emails[ry] = set()
        parent[ry] = rx
    
    # Map email → first account that has it
    email_to_account = {}
    for i, acc in enumerate(accounts):
        for email in acc[1:]:
            if email in email_to_account:
                union(i, email_to_account[email])
            else:
                email_to_account[email] = i
    
    # Collect results
    result = []
    for i in range(n):
        if find(i) == i and emails[i]:
            name = accounts[i][0]
            result.append([name] + sorted(emails[i]))
    
    return result
```

---

## Application: Maintaining Sorted Elements per Component

```python
from sortedcontainers import SortedList

class DSUWithSortedData:
    def __init__(self, n, values):
        self.parent = list(range(n))
        self.data = [SortedList([values[i]]) for i in range(n)]
    
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return
        
        # Small-to-large
        if len(self.data[rx]) < len(self.data[ry]):
            rx, ry = ry, rx
        
        # Move elements one by one (O(|smaller| × log(|larger|)))
        for val in self.data[ry]:
            self.data[rx].add(val)
        
        self.data[ry] = SortedList()
        self.parent[ry] = rx
    
    def kth_in_component(self, x, k):
        """Return kth smallest element in x's component."""
        root = self.find(x)
        if k >= len(self.data[root]):
            return -1
        return self.data[root][k]
```

---

## Complexity Analysis

```
Why O(n log n) total?

Consider any single element x:
  - x is in a set of size s₁
  - After merge: x is in a set of size ≥ 2s₁
  - x is only "moved" when it's in the smaller set
  - After move: its set size DOUBLES
  
  Starting from size 1, can double at most log₂(n) times
  → x is moved at most log₂(n) times

Total moves across ALL n elements:
  n × log₂(n) = O(n log n)

Per-merge cost:
  O(|smaller set| × per-element-work)
  
Total across all merges:
  O(n log n × per-element-work)
```

---

## Comparison: Naive vs Small-to-Large

```
Merging 1024 singletons into one set:

Naive (arbitrary merge direction):
  Worst case: always move the larger set
  Total moves: 1 + 2 + 4 + ... + 512 = 1023 per merge
  But accumulated: could be O(n²)
  
Small-to-Large:
  Round 1: 512 merges of size 1+1 → 512 sets of size 2
           512 moves
  Round 2: 256 merges of size 2+2 → 256 sets of size 4  
           512 moves
  ...
  Round 10: 1 merge of 512+512 → 1 set of 1024
           512 moves
  
  Total: 512 × 10 = 5120 = O(n log n) ✓
  
  n = 1024: n² = 1,048,576 vs n log n = 10,240
  That's 100x better!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Core idea | Always merge smaller into larger |
| Total work | O(n log n) element moves |
| Per-element moves | At most O(log n) |
| Data structures | Sets, sorted lists, maps, etc. |
| Applications | Account merge, component tracking |
| Alternative name | DSU on tree, weighted merge |

---

## Quick Revision Questions

1. **Why is small-to-large merging O(n log n) total instead of O(n²)?**
2. **How many times can a single element be moved?**
3. **What auxiliary data structures can be maintained per component?**
4. **How does the Accounts Merge problem use this technique?**
5. **What happens if you accidentally merge large into small?**

---

[← Previous: Offline Queries](03-offline-queries.md) | [Next: Virtual Nodes →](05-virtual-nodes.md)
