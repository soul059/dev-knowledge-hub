# Chapter 1: What is Path Compression?

## Overview

**Path compression** is the single most important optimization for Union-Find. During a `Find(x)` operation, it flattens the tree by making every node on the path from x to the root point **directly to the root**. This dramatically speeds up future operations.

---

## The Core Idea

```
WITHOUT path compression:
  Find(5) traverses: 5 → 4 → 3 → 2 → 1 → 0
  Next Find(5):      5 → 4 → 3 → 2 → 1 → 0  (same slow path!)

WITH path compression:
  Find(5) traverses: 5 → 4 → 3 → 2 → 1 → 0
                     then FLATTENS the path
  Next Find(5):      5 → 0  (direct! just 1 hop!)
```

---

## Before and After

```
BEFORE Find(5) with path compression:

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

AFTER Find(5) with path compression:

         0
      /|||||\ 
     1 2 3 4 5

Every node on the path now points DIRECTLY to root 0!
```

---

## How It Works

During Find(x), after finding the root, go back through every node visited and set its parent directly to the root.

```
function Find(x):
    root = x
    while parent[root] ≠ root:     // Step 1: Find the root
        root = parent[root]
    
    while x ≠ root:                 // Step 2: Compress the path
        next = parent[x]            //   save current parent
        parent[x] = root            //   point directly to root
        x = next                    //   move to saved parent
    
    return root
```

### Recursive Version (More Elegant)

```
function Find(x):
    if parent[x] ≠ x:
        parent[x] = Find(parent[x])    // recurse AND compress
    return parent[x]
```

---

## Step-by-Step Trace

```
parent = [0, 0, 1, 2, 3, 4]

Tree:
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

═══ Find(5) WITH Path Compression ═══

Phase 1: Find the root
  x = 5 → parent[5]=4 → x=4
  x = 4 → parent[4]=3 → x=3
  x = 3 → parent[3]=2 → x=2
  x = 2 → parent[2]=1 → x=1
  x = 1 → parent[1]=0 → x=0
  x = 0 → parent[0]=0 → ROOT FOUND! root = 0

Phase 2: Compress the path (x starts at 5 again)
  x = 5: next = parent[5] = 4, parent[5] = 0, x = 4
  x = 4: next = parent[4] = 3, parent[4] = 0, x = 3
  x = 3: next = parent[3] = 2, parent[3] = 0, x = 2
  x = 2: next = parent[2] = 1, parent[2] = 0, x = 1
  x = 1: next = parent[1] = 0, parent[1] = 0, x = 0
  x = 0: x == root → STOP

Result: parent = [0, 0, 0, 0, 0, 0]

Tree AFTER:
        0
     /|||||
    1 2 3 4 5

Now Find(5) = O(1)!  (5 → 0 directly)
```

---

## Why It's So Powerful

```
Consider n elements in a chain, and m Find operations:

WITHOUT path compression:
  Each Find can take O(n) → Total: O(m·n)

WITH path compression:
  First Find(deepest) takes O(n) — but FLATTENS the tree
  All subsequent Finds take O(1)!
  
  Amortized: O(α(n)) per operation ≈ O(1)
  
  ┌──────────────────────────────────────┐
  │ Path compression turns trees into    │
  │ near-flat structures over time.      │
  │                                      │
  │ "Pay once, benefit forever"          │
  └──────────────────────────────────────┘
```

---

## Self-Healing Property

Path compression has a beautiful **self-healing** property — the more you use Find, the faster it gets.

```
Initial (worst case):     After several Finds:

     0                          0
     |                     /|||||||||\ 
     1                    1 2 3 4 5 6 7 8 9
     |
     2
     |
     ...
     |
     9

The structure improves AUTOMATICALLY through normal usage!
```

---

## Impact on Tree Height

```
Before path compression:
  Height can be up to n-1

After path compression on all elements:
  Height becomes 1 (all nodes directly under root)

  Maximum height after m Finds with path compression:
  h ≤ O(log n) after ANY sequence of operations
  
  Combined with Union by Rank:
  h ≤ O(α(n)) amortized — practically O(1)
```

---

## Common Analogy

Think of a corporate hierarchy:

```
WITHOUT path compression:
  Employee → Manager → Director → VP → CEO
  
  "Who's the CEO?" requires climbing 4 levels every time.

WITH path compression:
  After asking once:
  Employee → CEO  (remembers the answer!)
  Manager  → CEO
  Director → CEO
  VP       → CEO
  
  Everyone now reports directly to CEO.
  Next time: instant answer!
```

---

## Summary Table

| Aspect | Without Path Compression | With Path Compression |
|--------|-------------------------|----------------------|
| Find time (first) | O(h) | O(h) |
| Find time (subsequent) | O(h) | O(1) amortized |
| Tree shape | Can degenerate | Self-flattening |
| Memory change | None | Modifies parent[] during Find |
| m operations | O(m·n) worst case | O(m·α(n)) amortized |

---

## Quick Revision Questions

1. **What does path compression do during a Find operation?**
2. **Does the first Find with path compression run faster than without?**
3. **Why is the recursive Find implementation so elegant for path compression?**
4. **What is the "self-healing" property of path compression?**
5. **After path compression on all elements, what is the tree height?**

---

[← Previous: Limitations](../02-Basic-Implementation/06-limitations.md) | [Next: Implementation Techniques →](02-implementation-techniques.md)
