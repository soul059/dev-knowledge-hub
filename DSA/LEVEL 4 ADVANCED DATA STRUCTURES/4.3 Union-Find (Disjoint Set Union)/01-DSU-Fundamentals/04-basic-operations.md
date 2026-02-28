# Chapter 4: Basic Operations

## Overview

Union-Find has three fundamental operations: **MakeSet**, **Find**, and **Union**. This chapter explains the logic and mechanics of each operation without optimizations — the "vanilla" version.

---

## Operation 1: MakeSet(x)

Creates a new set containing only element x.

```
MakeSet(x):
    parent[x] = x      // x is its own parent (root)

Visualization:
    Before MakeSet(5): element 5 doesn't exist
    After  MakeSet(5): 
        
        5
        ↺  (points to itself)
```

**Typical initialization** — create n singleton sets:

```
function Initialize(n):
    for i = 0 to n-1:
        MakeSet(i)
    
    // Result: parent = [0, 1, 2, ..., n-1]
```

**Time Complexity**: O(1) per element, O(n) total for initialization.

---

## Operation 2: Find(x)

Returns the **root** (representative) of the set containing x. Walks up the parent pointers until reaching a node that is its own parent.

```
function Find(x):
    while parent[x] ≠ x:       // while not at root
        x = parent[x]          // move to parent
    return x                   // return root
```

### Step-by-Step Trace

```
parent[] = [0, 0, 1, 2, 3]

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

Find(4):
  Step 1: x = 4, parent[4] = 3  → x = 3
  Step 2: x = 3, parent[3] = 2  → x = 2
  Step 3: x = 2, parent[2] = 1  → x = 1
  Step 4: x = 1, parent[1] = 0  → x = 0
  Step 5: x = 0, parent[0] = 0  → STOP! Return 0

  Find(4) = 0 (traversed 4 edges)

Find(1):
  Step 1: x = 1, parent[1] = 0  → x = 0
  Step 2: x = 0, parent[0] = 0  → STOP! Return 0

  Find(1) = 0 (traversed 1 edge)
```

**Time Complexity**: O(h) where h is the height of the tree.

---

## Operation 3: Union(x, y)

Merges the sets containing x and y by linking one root to the other.

```
function Union(x, y):
    rootX = Find(x)         // find root of x's set
    rootY = Find(y)         // find root of y's set
    if rootX ≠ rootY:       // if different sets
        parent[rootY] = rootX   // make rootX the parent of rootY
```

### Step-by-Step Trace

```
Initial: parent = [0, 1, 2, 3, 4]
Trees:   0   1   2   3   4

── Union(0, 1) ──
  Find(0) = 0, Find(1) = 1
  rootX = 0, rootY = 1
  Different roots → parent[1] = 0

  parent = [0, 0, 2, 3, 4]
  Tree:  0   2   3   4
         |
         1

── Union(2, 3) ──
  Find(2) = 2, Find(3) = 3
  rootX = 2, rootY = 3
  Different roots → parent[3] = 2

  parent = [0, 0, 2, 2, 4]
  Trees:  0    2    4
          |    |
          1    3

── Union(1, 3) ──
  Find(1): 1 → 0 (root), so rootX = 0
  Find(3): 3 → 2 (root), so rootY = 2
  Different roots → parent[2] = 0

  parent = [0, 0, 0, 2, 4]
  Tree:    0         4
          / \
         1   2
             |
             3
```

**Time Complexity**: O(h) dominated by the two Find operations.

---

## Connected Check: AreConnected(x, y)

```
function AreConnected(x, y):
    return Find(x) == Find(y)

Example:
  parent = [0, 0, 0, 2, 4]
  
  AreConnected(1, 3)?
    Find(1) = 0
    Find(3) = 0
    0 == 0 → TRUE ✓
  
  AreConnected(1, 4)?
    Find(1) = 0
    Find(4) = 4
    0 ≠ 4 → FALSE ✗
```

---

## Complete Operation Flow

```
┌───────────┐
│ MakeSet(x)│──── O(1) ──── Creates singleton {x}
└───────────┘

┌───────────┐
│  Find(x)  │──── O(h) ──── Returns root of x's tree
└─────┬─────┘
      │ walks up
      │ parent chain
      ▼
   ┌──────┐
   │ root │
   └──────┘

┌────────────┐     ┌──────────┐     ┌──────────┐
│ Union(x,y) │────▶│ Find(x)  │────▶│ Find(y)  │
└────────────┘     │ = rootX  │     │ = rootY  │
                   └──────────┘     └──────────┘
                        │                │
                        ▼                ▼
                   ┌──────────────────────────┐
                   │ if rootX ≠ rootY:        │
                   │   parent[rootY] = rootX  │
                   └──────────────────────────┘
```

---

## Edge Cases

### Union of elements already in the same set
```
Union(x, y) where Find(x) == Find(y):
  → rootX == rootY
  → Condition (rootX ≠ rootY) is FALSE
  → Nothing happens (no-op)
  → This is CORRECT behavior!
```

### Find on a root element
```
Find(root):
  → parent[root] == root  
  → Loop doesn't execute
  → Returns root immediately
  → O(1) time
```

---

## Operation Comparison

```
┌────────────┬──────────────┬───────────────────┬──────────┐
│ Operation  │ What it does │ Time (no optim.)  │ Modifies │
├────────────┼──────────────┼───────────────────┼──────────┤
│ MakeSet(x) │ Create {x}   │ O(1)              │ parent[] │
│ Find(x)    │ Get root     │ O(h) ≤ O(n)       │ Nothing  │
│ Union(x,y) │ Merge sets   │ O(h) ≤ O(n)       │ parent[] │
│ Connected  │ Same set?    │ O(h) ≤ O(n)       │ Nothing  │
└────────────┴──────────────┴───────────────────┴──────────┘

h = height of tallest tree involved
n = total number of elements
Worst case: h = n (degenerate chain)
```

---

## Summary Table

| Operation | Purpose | Time | Space Change |
|-----------|---------|------|-------------|
| MakeSet(x) | Create singleton set | O(1) | +1 element |
| Find(x) | Find representative | O(h) | None |
| Union(x,y) | Merge two sets | O(h) | None (in-place) |
| Connected(x,y) | Check same set | O(h) | None |

---

## Quick Revision Questions

1. **What does MakeSet do and what is its time complexity?**
2. **How does Find determine the root of an element?**
3. **What happens when you Union two elements already in the same set?**
4. **Why does Union need to call Find first?**
5. **What determines the time complexity of Find in the naive version?**

---

[← Previous: Forest Representation](03-forest-representation.md) | [Next: Applications Overview →](05-applications-overview.md)
