# Chapter 1: Find Operation

## Overview

The **Find** operation is the heart of Union-Find. It answers: **"Which set does element x belong to?"** by returning the **root** (representative) of the tree containing x.

---

## Core Logic

The Find operation follows parent pointers upward until it reaches a node that is its own parent — the **root**.

```
function Find(x):
    while parent[x] ≠ x:
        x = parent[x]
    return x
```

**Invariant**: The root is the only node where `parent[x] == x`.

---

## Visual Walkthrough

```
Tree Structure:
         0           ← root (parent[0] = 0)
        / \
       1   2
      / \
     3   4
     |
     5

parent[] = [0, 0, 0, 1, 1, 3]

═══ Find(5) ═══
    
    Start: x = 5
    ┌──────────────────────────────────────┐
    │ Step 1: parent[5] = 3 ≠ 5 → x = 3  │
    │ Step 2: parent[3] = 1 ≠ 3 → x = 1  │
    │ Step 3: parent[1] = 0 ≠ 1 → x = 0  │
    │ Step 4: parent[0] = 0 == 0 → STOP!  │
    └──────────────────────────────────────┘
    Return: 0

    Path traversed: 5 → 3 → 1 → 0
                              ↑
                            root

═══ Find(2) ═══

    Start: x = 2
    ┌──────────────────────────────────────┐
    │ Step 1: parent[2] = 0 ≠ 2 → x = 0  │
    │ Step 2: parent[0] = 0 == 0 → STOP!  │
    └──────────────────────────────────────┘
    Return: 0

    Path traversed: 2 → 0
                        ↑
                      root
```

---

## Iterative vs. Recursive Implementation

### Iterative (Preferred)
```
function Find(x):
    while parent[x] ≠ x:
        x = parent[x]
    return x
```

### Recursive
```
function Find(x):
    if parent[x] == x:
        return x
    return Find(parent[x])
```

### Comparison

| Aspect | Iterative | Recursive |
|--------|-----------|-----------|
| Stack overflow risk | None | Yes (deep trees) |
| Performance | Slightly faster | Slightly slower |
| Code simplicity | Clear | Elegant |
| Path compression fit | Manual | Natural |

---

## Why Find Returns the Root

The root uniquely identifies a set. Two elements are in the same set **if and only if** they have the same root.

```
Set A: {0, 1, 3, 5}     root = 0
Set B: {2, 4}            root = 2

Find(1) = 0 ┐
Find(3) = 0 ├── All return 0 → same set!
Find(5) = 0 ┘

Find(2) = 2 ┐
Find(4) = 2 ┘── Both return 2 → same set!

Find(1) = 0 ≠ Find(4) = 2  → different sets!
```

---

## Time Complexity Analysis

```
Find traverses from node x up to the root.

Time = O(depth of x) = O(height of tree)

Best case:  O(1)  — x is the root
Worst case: O(n)  — degenerate chain

Best case tree:          Worst case tree (chain):
      0                        0
    / | \                      |
   1  2  3                     1
  /|    / \                    |
 4 5   6   7                   2
                               |
 Find(any) = O(1) or O(2)     3
                               |
                              ...
                               |
                              n-1
                          
                          Find(n-1) = O(n)
```

---

## Find in Connected Check

```
function AreConnected(x, y):
    return Find(x) == Find(y)

Trace:
  parent = [0, 0, 2, 2, 0]
  
  Tree 1:    0        Tree 2:   2
           / | \                |
          1  4                  3
  
  AreConnected(1, 4)?
    Find(1) = 0
    Find(4) = 0
    0 == 0 → TRUE
  
  AreConnected(1, 3)?
    Find(1) = 0
    Find(3) = 2
    0 ≠ 2 → FALSE
```

---

## Implementation in Code

### Python
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))  # parent[i] = i initially
    
    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x
```

### C++
```cpp
class UnionFind {
    vector<int> parent;
public:
    UnionFind(int n) : parent(n) {
        iota(parent.begin(), parent.end(), 0); // parent[i] = i
    }
    
    int find(int x) {
        while (parent[x] != x)
            x = parent[x];
        return x;
    }
};
```

### Java
```java
class UnionFind {
    int[] parent;
    
    UnionFind(int n) {
        parent = new int[n];
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }
    
    int find(int x) {
        while (parent[x] != x)
            x = parent[x];
        return x;
    }
}
```

---

## Common Mistakes

```
MISTAKE 1: Forgetting the base case
  ✗ function Find(x):
        return Find(parent[x])     // infinite recursion!
  
  ✓ function Find(x):
        if parent[x] == x: return x
        return Find(parent[x])

MISTAKE 2: Not following the chain completely
  ✗ return parent[x]              // returns parent, not root!
  
  ✓ Follow until parent[x] == x  // must reach root

MISTAKE 3: Off-by-one with 0-indexed vs 1-indexed
  ✗ parent = new int[n]   // for 1-indexed elements
  ✓ parent = new int[n+1] // extra slot for 1-indexed
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Purpose | Find the root/representative of x's set |
| Algorithm | Follow parent pointers to root |
| Root condition | `parent[x] == x` |
| Time (naive) | O(h) where h = tree height |
| Worst case | O(n) for degenerate chain |
| Best case | O(1) when x is the root |
| Returns | Root element (representative) |

---

## Quick Revision Questions

1. **How does Find know when it has reached the root?**
2. **What is the worst-case time for naive Find and when does it occur?**
3. **Why do we return the root rather than just the parent?**
4. **What is the risk of using recursive Find without path compression?**
5. **How do you use Find to check if two elements are in the same set?**

---

[← Previous: When to Use DSU](../01-DSU-Fundamentals/06-when-to-use-dsu.md) | [Next: Union Operation →](02-union-operation.md)
