# Chapter 5: Path Halving

## Overview

**Path halving** is a simpler variant of path compression. Instead of making every node point directly to the root, it makes every **other** node on the path skip its parent, pointing to its grandparent instead.

---

## How Path Halving Works

```
function Find(x):
    while parent[x] ≠ x:
        parent[x] = parent[parent[x]]    // skip to grandparent
        x = parent[x]                    // move to (new) parent
    return x
```

**Key**: Only ONE line of code added compared to naive Find!

---

## Visual Demonstration

```
Initial tree:
     0
     |
     1
     |
     2
     |
     3
     |
     4
     |
     5

parent = [0, 0, 1, 2, 3, 4]

═══ Find(5) with Path Halving ═══

Step 1: x = 5
  parent[5] = 4, grandparent = parent[4] = 3
  parent[5] = 3    ← skip 4, point to grandparent
  x = 3            ← move to new parent (3)

  Tree state:
       0
       |
       1
       |
       2
       |
       3
      / \
     4   5    ← 5 now child of 3 (was child of 4)

Step 2: x = 3
  parent[3] = 2, grandparent = parent[2] = 1
  parent[3] = 1    ← skip 2, point to grandparent
  x = 1            ← move to new parent (1)

  Tree state:
       0
       |
       1
      / \
     2   3
         |
     4   5

Step 3: x = 1
  parent[1] = 0, grandparent = parent[0] = 0
  parent[1] = 0    ← already correct (parent is root)
  x = 0            ← move to root

Step 4: x = 0
  parent[0] = 0 == x → STOP! Return 0

Result:
parent = [0, 0, 1, 1, 3, 3]

       0
      / \
     1   2
    / \
   3   
  / \
 4   5

Wait, let me re-trace more carefully:
Final parent: [0, 0, 1, 1, 3, 3]

       0
       |
       1
      / \
     2   3
        / \
       4   5

Height reduced from 5 to 3 (not as flat as full compression)
```

---

## Path Halving vs Full Compression

```
Same initial tree:
     0─1─2─3─4─5 (chain, height = 5)

FULL Path Compression (Find(5)):
         0
      /|||||
     1 2 3 4 5

     Height: 5 → 1  (completely flat)

PATH HALVING (Find(5)):
       0
       |
       1
      / \
     2   3
        / \
       4   5

     Height: 5 → 3  (partially flattened)

Path halving does HALF the compression per Find.
But it achieves the same amortized complexity!
```

---

## Why Use Path Halving?

```
Advantages over full path compression:
┌─────────────────────────────────────────────────┐
│ ✓ Single pass (no second traversal)             │
│ ✓ No recursion needed (no stack overflow risk)  │
│ ✓ Same amortized complexity: O(α(n))            │
│ ✓ Simpler code (just 2 lines in the loop)       │
│ ✓ Better cache performance in practice          │
│ ✓ Compatible with Union by Rank invariants      │
└─────────────────────────────────────────────────┘

Disadvantage:
┌─────────────────────────────────────────────────┐
│ ✗ Individual Find may be slightly slower        │
│   (doesn't fully flatten like full compression) │
│   But amortized, it's the same!                 │
└─────────────────────────────────────────────────┘
```

---

## Implementation Comparison

```
Naive Find:                    Path Halving:
  while parent[x] ≠ x:          while parent[x] ≠ x:
      x = parent[x]                 parent[x] = parent[parent[x]]
  return x                          x = parent[x]
                                 return x

  Difference: ONE extra line!
```

### Full Implementations

**Python:**
```python
def find(self, x):
    while self.parent[x] != x:
        self.parent[x] = self.parent[self.parent[x]]  # path halving
        x = self.parent[x]
    return x
```

**C++:**
```cpp
int find(int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];  // path halving
        x = parent[x];
    }
    return x;
}
```

---

## Multiple Finds Show Convergence

```
Initial chain: 0─1─2─3─4─5─6─7  (height 7)

Find(7) — Round 1:
  x=7: skip 6, point to 5, move to 5    parent[7]=5
  x=5: skip 4, point to 3, move to 3    parent[5]=3
  x=3: skip 2, point to 1, move to 1    parent[3]=1
  x=1: skip 0→0, point to 0, move to 0  parent[1]=0
  
  Tree:    0
           |
           1
          / \
         2   3
             |
         4   5
             |
         6   7
  Height = 4

Find(7) — Round 2:
  x=7: parent[7]=5, parent[5]=3 → parent[7]=3, x=3
  x=3: parent[3]=1, parent[1]=0 → parent[3]=0, x=0
  
  Height = 3

Find(7) — Round 3:
  x=7: parent[7]=3, parent[3]=0 → parent[7]=0, x=0
  
  Height = 2 (nearly flat!)

After 3 rounds, even path halving produces a nearly flat tree.
```

---

## Complexity Proof Sketch

```
Path halving achieves O(α(n)) amortized because:

1. Each halving step reduces the path length by ~half
2. After log*(n) Find operations on any path, 
   it becomes constant length
3. The total "work" across all operations is bounded 
   by O(n · α(n))

Key insight: Half compression per Find × twice as many rounds
             = same total compression work
             = same amortized complexity as full compression
```

---

## Summary Table

| Aspect | Path Halving | Full Compression |
|--------|-------------|-----------------|
| Passes per Find | 1 | 1 (recursive) or 2 (iterative) |
| Compression per Find | ~50% | 100% |
| Amortized complexity | O(α(n)) | O(α(n)) |
| Stack overflow risk | None | Yes (recursive) |
| Code complexity | Very simple | Simple |
| Practical speed | Slightly faster | Slightly slower |

---

## Quick Revision Questions

1. **What does path halving do to each node during Find?**
2. **How many extra lines does path halving add to naive Find?**
3. **Is the amortized complexity different from full path compression?**
4. **Why might path halving be preferred in practice?**
5. **After how many rounds of Find does path halving fully flatten a chain?**

---

[← Previous: Visual Demonstration](04-visual-demonstration.md) | [Next: Path Splitting →](06-path-splitting.md)
