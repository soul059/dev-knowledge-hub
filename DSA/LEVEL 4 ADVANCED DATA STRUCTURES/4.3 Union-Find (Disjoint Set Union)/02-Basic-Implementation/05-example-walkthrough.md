# Chapter 5: Example Walkthrough

## Overview

This chapter provides a comprehensive, step-by-step walkthrough of Union-Find solving a real problem. We'll trace every operation with diagrams showing the evolving forest.

---

## Problem: Friend Groups

**Given**: 8 people (0-7) and the following friendships:
```
(0,1), (1,2), (3,4), (5,6), (5,7), (2,7)
```

**Find**: How many friend groups exist?

---

## Step-by-Step Solution

### Initialization

```
parent = [0, 1, 2, 3, 4, 5, 6, 7]
components = 8

Forest:
  0   1   2   3   4   5   6   7
  ↺   ↺   ↺   ↺   ↺   ↺   ↺   ↺

  8 singleton trees
```

### Step 1: Process Edge (0, 1)

```
Find(0) = 0 (root)
Find(1) = 1 (root)
0 ≠ 1 → Union! parent[1] = 0
components = 7

parent = [0, 0, 2, 3, 4, 5, 6, 7]

Forest:
  0    2   3   4   5   6   7
  |    ↺   ↺   ↺   ↺   ↺   ↺
  1
```

### Step 2: Process Edge (1, 2)

```
Find(1): 1 → parent[1]=0 → root! = 0
Find(2) = 2 (root)
0 ≠ 2 → Union! parent[2] = 0
components = 6

parent = [0, 0, 0, 3, 4, 5, 6, 7]

Forest:
    0       3   4   5   6   7
   / \      ↺   ↺   ↺   ↺   ↺
  1   2
```

### Step 3: Process Edge (3, 4)

```
Find(3) = 3 (root)
Find(4) = 4 (root)
3 ≠ 4 → Union! parent[4] = 3
components = 5

parent = [0, 0, 0, 3, 3, 5, 6, 7]

Forest:
    0       3     5   6   7
   / \      |     ↺   ↺   ↺
  1   2     4
```

### Step 4: Process Edge (5, 6)

```
Find(5) = 5 (root)
Find(6) = 6 (root)
5 ≠ 6 → Union! parent[6] = 5
components = 4

parent = [0, 0, 0, 3, 3, 5, 5, 7]

Forest:
    0       3     5     7
   / \      |     |     ↺
  1   2     4     6
```

### Step 5: Process Edge (5, 7)

```
Find(5) = 5 (root)
Find(7) = 7 (root)
5 ≠ 7 → Union! parent[7] = 5
components = 3

parent = [0, 0, 0, 3, 3, 5, 5, 5]

Forest:
    0       3       5
   / \      |      / \
  1   2     4     6   7
```

### Step 6: Process Edge (2, 7)

```
Find(2): 2 → parent[2]=0 → root! = 0
Find(7): 7 → parent[7]=5 → root! = 5
0 ≠ 5 → Union! parent[5] = 0
components = 2

parent = [0, 0, 0, 3, 3, 0, 5, 5]

Forest:
        0              3
      / | \            |
     1  2  5           4
          / \
         6   7
```

### Final State

```
parent = [0, 0, 0, 3, 3, 0, 5, 5]
components = 2

Two friend groups:
  Group 1 (root=0): {0, 1, 2, 5, 6, 7}
  Group 2 (root=3): {3, 4}

Verification:
  Find(0) = 0 ┐
  Find(1) = 0 │
  Find(2) = 0 │ All root = 0
  Find(5) = 0 │ → Same group!
  Find(6) = 0 │
  Find(7) = 0 ┘

  Find(3) = 3 ┐ Both root = 3
  Find(4) = 3 ┘ → Same group!

Answer: 2 friend groups
```

---

## Query Traces

```
After all unions, let's trace some queries:

Q: Are 1 and 7 friends?
  Find(1): 1 → 0 (root)     = 0
  Find(7): 7 → 5 → 0 (root) = 0
  0 == 0 → YES ✓

Q: Are 2 and 4 friends?
  Find(2): 2 → 0 (root) = 0
  Find(4): 4 → 3 (root) = 3
  0 ≠ 3 → NO ✗

Q: Are 6 and 7 friends?
  Find(6): 6 → 5 → 0 (root) = 0
  Find(7): 7 → 5 → 0 (root) = 0
  0 == 0 → YES ✓
```

---

## Operation Count Analysis

```
For this problem:
  n = 8 elements
  m = 6 edges + queries

  Total Find calls during Unions:
    Union(0,1): Find(0)=1 hop, Find(1)=1 hop  → 2 hops
    Union(1,2): Find(1)=2 hops, Find(2)=1 hop → 3 hops
    Union(3,4): Find(3)=1 hop, Find(4)=1 hop  → 2 hops
    Union(5,6): Find(5)=1 hop, Find(6)=1 hop  → 2 hops
    Union(5,7): Find(5)=1 hop, Find(7)=1 hop  → 2 hops
    Union(2,7): Find(2)=2 hops, Find(7)=2 hops→ 4 hops
    
  Total: 15 hops for 6 Union operations
  Average: 2.5 hops per Union

  Maximum tree height = 3 (path 6→5→0 or 7→5→0)
```

---

## Full State Table

| Step | Operation | parent[] | Components | Tree Height |
|------|-----------|----------|------------|-------------|
| Init | — | [0,1,2,3,4,5,6,7] | 8 | 0 |
| 1 | Union(0,1) | [0,0,2,3,4,5,6,7] | 7 | 1 |
| 2 | Union(1,2) | [0,0,0,3,4,5,6,7] | 6 | 1 |
| 3 | Union(3,4) | [0,0,0,3,3,5,6,7] | 5 | 1 |
| 4 | Union(5,6) | [0,0,0,3,3,5,5,7] | 4 | 1 |
| 5 | Union(5,7) | [0,0,0,3,3,5,5,5] | 3 | 1 |
| 6 | Union(2,7) | [0,0,0,3,3,0,5,5] | 2 | 3 |

---

## Alternative Problem: Island Count

```
Grid:
  1 1 0 0
  1 0 0 1
  0 0 1 1

Convert to Union-Find:
  Number each cell:
  0  1  2  3
  4  5  6  7
  8  9  10 11

  Land cells: 0,1,4,7,10,11
  
  For each land cell, union with adjacent land cells:
  
  Union(0,1): same island
  Union(0,4): same island
  Union(7,11): same island
  Union(10,11): same island
  
  Result: 
    Island 1: {0,1,4}    (top-left)
    Island 2: {7,10,11}  (bottom-right)
    Answer: 2 islands
```

---

## Summary Table

| Problem Element | DSU Mapping |
|----------------|-------------|
| Person/Node | Element in DSU |
| Friendship/Edge | Union operation |
| "Same group?" query | Connected (Find + compare) |
| Number of groups | Component counter |
| Group members | All elements with same root |

---

## Quick Revision Questions

1. **In the friend groups problem, how many Find calls are made total during all Union operations?**
2. **What is the final tree height after all operations?**
3. **If we add edge (3, 6), what happens to the number of components?**
4. **How would you list all members of a specific group?**
5. **Why is the order of edge processing important for tree shape but not for final grouping?**

---

[← Previous: Time Complexity (Naive)](04-time-complexity-naive.md) | [Next: Limitations →](06-limitations.md)
