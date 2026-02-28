# Chapter 2: Implementation Techniques

## Overview

Path compression can be implemented in several ways. This chapter covers the two main approaches — **recursive** and **iterative** — with full code and trade-offs.

---

## Technique 1: Recursive (One-Pass)

The most elegant implementation. The recursion naturally handles both finding the root and compressing the path in a single pass.

```
function Find(x):
    if parent[x] ≠ x:
        parent[x] = Find(parent[x])    // recurse to root, compress on return
    return parent[x]
```

### How It Works

```
parent = [0, 0, 1, 2, 3]

Tree:    0 ← 1 ← 2 ← 3 ← 4

Call Find(4):
  ┌─ Find(4): parent[4]=3 ≠ 4, so call Find(3)
  │  ┌─ Find(3): parent[3]=2 ≠ 3, so call Find(2)
  │  │  ┌─ Find(2): parent[2]=1 ≠ 2, so call Find(1)
  │  │  │  ┌─ Find(1): parent[1]=0 ≠ 1, so call Find(0)
  │  │  │  │  ┌─ Find(0): parent[0]=0 == 0 → return 0
  │  │  │  │  └─
  │  │  │  │  parent[1] = Find(0) = 0  ← compress!
  │  │  │  └─ return 0
  │  │  │  parent[2] = Find(1) = 0    ← compress!
  │  │  └─ return 0
  │  │  parent[3] = Find(2) = 0       ← compress!
  │  └─ return 0
  │  parent[4] = Find(3) = 0          ← compress!
  └─ return 0

After: parent = [0, 0, 0, 0, 0]

Tree:      0
        / | | \
       1  2  3  4
```

### Code Implementations

**Python:**
```python
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])
    return self.parent[x]
```

**C++:**
```cpp
int find(int x) {
    if (parent[x] != x)
        parent[x] = find(parent[x]);
    return parent[x];
}
```

**Java:**
```java
int find(int x) {
    if (parent[x] != x)
        parent[x] = find(parent[x]);
    return parent[x];
}
```

---

## Technique 2: Iterative (Two-Pass)

Two separate loops: first find the root, then compress the path.

```
function Find(x):
    // Pass 1: Find root
    root = x
    while parent[root] ≠ root:
        root = parent[root]
    
    // Pass 2: Compress path
    while parent[x] ≠ root:
        next = parent[x]
        parent[x] = root
        x = next
    
    return root
```

### How It Works

```
parent = [0, 0, 1, 2, 3]

Call Find(4):

Pass 1 — Find root:
  root = 4 → parent[4]=3 → root=3
  root = 3 → parent[3]=2 → root=2
  root = 2 → parent[2]=1 → root=1
  root = 1 → parent[1]=0 → root=0
  root = 0 → parent[0]=0 → STOP! root = 0

Pass 2 — Compress:
  x = 4: next=parent[4]=3, parent[4]=0, x=3
  x = 3: next=parent[3]=2, parent[3]=0, x=2
  x = 2: next=parent[2]=1, parent[2]=0, x=1
  x = 1: next=parent[1]=0, parent[1]=0, x=0
  x = 0: parent[0]=0==root → STOP!

After: parent = [0, 0, 0, 0, 0]    (same result!)
```

### Code Implementations

**Python:**
```python
def find(self, x):
    root = x
    while self.parent[root] != root:
        root = self.parent[root]
    while self.parent[x] != root:
        self.parent[x], x = root, self.parent[x]
    return root
```

**C++:**
```cpp
int find(int x) {
    int root = x;
    while (parent[root] != root)
        root = parent[root];
    while (parent[x] != root) {
        int next = parent[x];
        parent[x] = root;
        x = next;
    }
    return root;
}
```

---

## Comparison: Recursive vs. Iterative

```
┌─────────────────┬──────────────┬───────────────┐
│ Aspect          │  Recursive   │  Iterative    │
├─────────────────┼──────────────┼───────────────┤
│ Code length     │  3 lines     │  8-10 lines   │
│ Readability     │  Very clear  │  Moderate     │
│ Stack overflow  │  Risk (deep  │  No risk      │
│                 │  trees)      │               │
│ # of passes     │  1 pass      │  2 passes     │
│ # of lines      │  1 pass up,  │  1 pass up,   │
│ traversed       │  1 back down │  1 pass down  │
│ Performance     │  Same O()    │  Same O()     │
│ Recommended     │  Most cases  │  Very deep    │
│                 │              │  trees        │
└─────────────────┴──────────────┴───────────────┘
```

---

## Stack Overflow Considerations

```
Recursive Find on a chain of depth n:
  Creates n stack frames!

  n = 10,000  → Usually OK
  n = 100,000 → May crash (stack overflow)
  n = 1,000,000 → Will crash!

  Default stack sizes:
  ┌──────────┬────────────┬───────────────┐
  │ Language │ Stack Size │ Max Depth     │
  ├──────────┼────────────┼───────────────┤
  │ Python   │ 1000 calls │ Very limited! │
  │ Java     │ ~512 KB    │ ~10,000       │
  │ C++      │ ~1-8 MB    │ ~50,000       │
  └──────────┴────────────┴───────────────┘

  Python fix: sys.setrecursionlimit(500000)
  Better fix: Use iterative implementation
  Best fix: Use Union by Rank (limits depth to log n)
```

---

## Complete Union-Find with Path Compression

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.components = n
    
    def find(self, x):
        """Recursive Find with path compression."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union two elements (no rank optimization yet)."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        self.parent[ry] = rx
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

```cpp
class UnionFind {
    vector<int> parent;
    int components;
public:
    UnionFind(int n) : parent(n), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);
        return parent[x];
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        parent[ry] = rx;
        components--;
        return true;
    }
};
```

---

## Side Effect: Find Modifies Structure

```
IMPORTANT: With path compression, Find is NO LONGER a read-only operation!

Before Find(4):         After Find(4):
     0                       0
     |                    / / \ \
     1                   1  2  3  4
     |
     2
     |
     3
     |
     4

parent changed from [0,0,1,2,3] to [0,0,0,0,0]

This means:
  ✓ Find is still correct (returns same root)
  ✗ Find modifies parent[] (side effect)
  ✗ Cannot use in contexts requiring immutability
  ✗ Thread safety issues (concurrent Finds can conflict)
```

---

## Summary Table

| Technique | Passes | Stack Safe | Lines of Code | Best For |
|-----------|--------|------------|---------------|----------|
| Recursive | 1 (up+down) | No (deep trees) | 3 | General use |
| Iterative (two-pass) | 2 | Yes | 8-10 | Large n |
| Path halving | 1 | Yes | 4 | Compromise |
| Path splitting | 1 | Yes | 5 | Compromise |

---

## Quick Revision Questions

1. **Write the recursive path compression Find in 3 lines.**
2. **Why might the recursive version cause a stack overflow?**
3. **How many passes does the iterative version make?**
4. **Does Find with path compression modify the data structure?**
5. **Which technique would you choose for n = 1,000,000 and why?**

---

[← Previous: What is Path Compression?](01-what-is-path-compression.md) | [Next: Amortized Complexity →](03-amortized-complexity.md)
