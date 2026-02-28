# Chapter 6: Path Splitting

## Overview

**Path splitting** is another variant of path compression where every node on the path to the root is made to point to its **grandparent**. Unlike path halving, path splitting modifies **every** node (not every other node).

---

## How Path Splitting Works

```
function Find(x):
    while parent[x] ≠ x:
        next = parent[x]               // save parent
        parent[x] = parent[parent[x]]  // point to grandparent
        x = next                       // advance to old parent
    return x
```

**Key difference from path halving**: We advance to `next` (old parent), not to the new parent. This means we process **every** node.

---

## Visual Demonstration

```
Initial chain: 0─1─2─3─4─5 (height 5)

parent = [0, 0, 1, 2, 3, 4]

═══ Find(5) with Path Splitting ═══

Step 1: x = 5
  next = parent[5] = 4
  parent[5] = parent[4] = 3    ← 5 now points to grandparent 3
  x = next = 4

Step 2: x = 4
  next = parent[4] = 3
  parent[4] = parent[3] = 2    ← 4 now points to grandparent 2
  x = next = 3

Step 3: x = 3
  next = parent[3] = 2
  parent[3] = parent[2] = 1    ← 3 now points to grandparent 1
  x = next = 2

Step 4: x = 2
  next = parent[2] = 1
  parent[2] = parent[1] = 0    ← 2 now points to grandparent 0
  x = next = 1

Step 5: x = 1
  next = parent[1] = 0
  parent[1] = parent[0] = 0    ← 1 already points to root
  x = next = 0

Step 6: x = 0
  parent[0] = 0 == x → STOP! Return 0

Result: parent = [0, 0, 0, 1, 2, 3]

       0
      / \
     1   2
     |   |
     3   4
     |
     5

Height reduced from 5 to 3
```

---

## Path Splitting vs Path Halving vs Full Compression

```
Same initial chain: 0─1─2─3─4─5 (height 5)

After Find(5):

FULL COMPRESSION:           PATH HALVING:          PATH SPLITTING:
       0                        0                       0
    /|||||                      |                      / \
   1 2 3 4 5                    1                     1   2
                               / \                    |   |
Height: 1                     2   3                   3   4
                                  |                   |
                              4   5                   5
                            
                            Height: 3              Height: 3

All three achieve O(α(n)) amortized complexity!
```

---

## Comparison Table

```
┌──────────────────┬───────────────┬──────────────┬───────────────┐
│ Aspect           │ Full Compr.   │ Path Halving │ Path Splitting│
├──────────────────┼───────────────┼──────────────┼───────────────┤
│ Nodes modified   │ ALL on path   │ Every OTHER  │ ALL on path   │
│ Advances by      │ to root       │ grandparent  │ old parent    │
│ Compression      │ 100%          │ ~50%         │ ~50%          │
│ per operation    │               │              │               │
│ Amortized cost   │ O(α(n))       │ O(α(n))      │ O(α(n))       │
│ Recursion needed │ Yes (or 2-pass)│ No          │ No            │
│ Stack safe       │ Iterative: Yes│ Yes          │ Yes           │
│ Code complexity  │ Moderate      │ Simple       │ Simple        │
│ Practice speed   │ Good          │ Best         │ Good          │
└──────────────────┴───────────────┴──────────────┴───────────────┘
```

---

## Code Side-by-Side

```python
# NAIVE
def find(x):
    while parent[x] != x:
        x = parent[x]                          # just follow
    return x

# PATH HALVING
def find(x):
    while parent[x] != x:
        parent[x] = parent[parent[x]]          # skip to grandparent
        x = parent[x]                          # advance to NEW parent
    return x

# PATH SPLITTING
def find(x):
    while parent[x] != x:
        next_x = parent[x]                     # save old parent
        parent[x] = parent[parent[x]]          # skip to grandparent
        x = next_x                             # advance to OLD parent
    return x

# FULL COMPRESSION (recursive)
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])            # recursive compress
    return parent[x]
```

---

## When to Use Which?

```
Decision Guide:

  Need maximum compression? → Full compression
  Want simplest code?       → Path halving (1 extra line)
  Need to process all nodes?→ Path splitting
  Worried about stack?      → Path halving or splitting
  Competitive programming?  → Any of the three (same O())
  
  In practice:
  ┌─────────────────────────────────────────────┐
  │ Most competitive programmers use either:     │
  │   1. Recursive full compression (most common)│
  │   2. Path halving (fastest in practice)      │
  │                                              │
  │ Path splitting is rarely used but good       │
  │ to know for theoretical understanding.       │
  └─────────────────────────────────────────────┘
```

---

## Convergence Comparison

```
Chain of 16 nodes: 0─1─2─...─15

After repeated Find(15):

                 Full Comp.  Path Halving  Path Splitting
After 1 Find:   Height = 1   Height = 8    Height = 8
After 2 Finds:  Height = 1   Height = 4    Height = 4
After 3 Finds:  Height = 1   Height = 2    Height = 2
After 4 Finds:  Height = 1   Height = 1    Height = 1

Full compression achieves flatness in 1 Find.
Halving/Splitting need O(log n) Finds on same element.
But amortized across all elements: SAME total work!
```

---

## Summary Table

| Variant | Nodes Modified | Per-Find Reduction | Amortized | Best For |
|---------|---------------|-------------------|-----------|----------|
| Full Compression | All on path | 100% | O(α(n)) | Maximum flattening |
| Path Halving | Every other | ~50% | O(α(n)) | Simple, fast code |
| Path Splitting | All on path | ~50% | O(α(n)) | All-node processing |

---

## Quick Revision Questions

1. **How does path splitting differ from path halving?**
2. **In path splitting, where does x advance to — new parent or old parent?**
3. **Is the amortized complexity different between the three variants?**
4. **Which path compression variant requires the most code changes from naive?**
5. **Why do path halving and splitting both need O(log n) rounds to fully flatten?**

---

[← Previous: Path Halving](05-path-halving.md) | [Next: Unit 4 - Union by Rank →](../04-Union-By-Rank-Size/01-union-by-rank.md)

[← Back to Main README](../README.md)
