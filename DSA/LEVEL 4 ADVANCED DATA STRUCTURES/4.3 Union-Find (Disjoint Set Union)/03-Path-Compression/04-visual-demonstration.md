# Chapter 4: Visual Demonstration

## Overview

This chapter provides detailed visual traces of path compression in action, showing how the tree structure transforms with each operation.

---

## Demo 1: Simple Chain Compression

```
Initial tree (chain of 6 nodes):

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
```

### Find(5) with Path Compression

```
Step 1: Traverse to root
  5 → 4 → 3 → 2 → 1 → 0 (root!)

Step 2: Compress — set every node's parent to 0

  parent[5] = 0  ✓
  parent[4] = 0  ✓
  parent[3] = 0  ✓
  parent[2] = 0  ✓
  parent[1] = 0  (already 0)

RESULT:
       0
    /|||||
   1 2 3 4 5

parent = [0, 0, 0, 0, 0, 0]

All future Find() calls = O(1)!
```

---

## Demo 2: Partial Compression

```
Initial tree:
          0
         / \
        1   2
       / \
      3   4
     / \
    5   6
    |
    7

parent = [0, 0, 0, 1, 1, 3, 3, 5]

═══ Find(7) ═══
Path: 7 → 5 → 3 → 1 → 0

Compress nodes {7, 5, 3, 1}:

AFTER Find(7):
           0
        / / | \
       1  2  3  5
       |     |  
       4     6  
              
       7 ← also now points to 0

Actually:
            0
       / / | | \ \
      1  2  3  5  7
      |     |
      4     6

parent = [0, 0, 0, 0, 1, 0, 3, 0]

Only nodes ON THE PATH were compressed!
Nodes 2, 4, 6 were NOT affected.
```

### Now Find(6)

```
Current tree:
            0
       / / | | \ \
      1  2  3  5  7
      |     |
      4     6

Path: 6 → 3 → 0

Compress nodes {6, 3}:
  parent[6] = 0  ✓
  parent[3] = 0  (already 0)

AFTER Find(6):
              0
       / / | | | \ \
      1  2  3  5  6  7
      |     
      4     

parent = [0, 0, 0, 0, 1, 0, 0, 0]
```

### Now Find(4)

```
Current tree:
              0
       / / | | | \ \
      1  2  3  5  6  7
      |     
      4     

Path: 4 → 1 → 0

Compress nodes {4, 1}:
  parent[4] = 0  ✓
  parent[1] = 0  (already 0)

AFTER Find(4):
                0
       / / / | | | \ \
      1  2  3  4  5  6  7

parent = [0, 0, 0, 0, 0, 0, 0, 0]

NOW the tree is COMPLETELY FLAT!
Every single Find = O(1)
```

---

## Demo 3: Two Trees Merging with Compression

```
Tree A:          Tree B:
    0                6
   / \              / \
  1   2            7   8
  |                |
  3                9
  |
  4
  |
  5

═══ Union(5, 9) ═══

Step 1: Find(5) with path compression
  Path: 5→4→3→1→0
  Compress: all point to 0

  Tree A becomes:
        0
   / / / | \
  1  2  3  4  5

Step 2: Find(9) with path compression
  Path: 9→7→6
  Compress: all point to 6

  Tree B becomes:
      6
    / | \
   7  8  9

Step 3: Union roots — parent[6] = 0

  Combined tree:
           0
     / / / | | \
    1  2  3  4  5  6
                  /|\
                 7 8 9

  Much flatter than naive union would produce!
```

---

## Demo 4: Progressive Flattening

```
Show how repeated Finds flatten the tree progressively:

Initial:
        0
        |
        1
       / \
      2   3
     / \
    4   5
   /
  6

═══ Find(6): Path 6→4→2→1→0 ═══
After:
         0
      /|||||
     1 2 4 6
     |   |
     3   5

═══ Find(5): Path 5→2→0 ═══  (2 already compressed to 0!)
Wait: parent[2] = 0 now, so:
  5→parent[5]=2→parent[2]=0 → root!
  Actually path: 5→2→0 (length 2, not 4!)

After:
         0
     /||||||
    1 2 4 5 6
    |   
    3   

═══ Find(3): Path 3→1→0 ═══
After:
          0
     /|||||||
    1 2 3 4 5 6

COMPLETELY FLAT after just 3 Find operations!
```

---

## Demo 5: Recursive Find Trace

```
parent = [0, 0, 1, 2, 3]

Tree: 0←1←2←3←4

Recursive Find(4):

  Find(4):
    parent[4] = 3 ≠ 4
    parent[4] = Find(3)           ← recurse
      │
      ├─ Find(3):
      │   parent[3] = 2 ≠ 3
      │   parent[3] = Find(2)     ← recurse
      │     │
      │     ├─ Find(2):
      │     │   parent[2] = 1 ≠ 2
      │     │   parent[2] = Find(1)  ← recurse
      │     │     │
      │     │     ├─ Find(1):
      │     │     │   parent[1] = 0 ≠ 1
      │     │     │   parent[1] = Find(0)  ← recurse
      │     │     │     │
      │     │     │     ├─ Find(0):
      │     │     │     │   parent[0] = 0 == 0
      │     │     │     │   return 0  ←── BASE CASE
      │     │     │     │
      │     │     │   parent[1] = 0    ← COMPRESS
      │     │     │   return 0
      │     │     │
      │     │   parent[2] = 0          ← COMPRESS
      │     │   return 0
      │     │
      │   parent[3] = 0                ← COMPRESS
      │   return 0
      │
    parent[4] = 0                      ← COMPRESS
    return 0

After: parent = [0, 0, 0, 0, 0]    ALL compressed!
```

---

## Animation Summary

```
Think of path compression as a SPRING:

Before:                          After:
  ┌─┐                            ┌─┐
  │R│ root                       │R│ root
  └┬┘                           └┬┘
   │                          ┌──┴──┬──┬──┐
  ┌┴┐                        ┌┴┐  ┌┴┐┌┴┐┌┴┐
  │ │                        │ │  │ ││ ││ │
  └┬┘                        └─┘  └─┘└─┘└─┘
   │                          A    B   C   D
  ┌┴┐                    
  │ │  ←── "spring"        All nodes now 
  └┬┘      compressed!     directly under root
   │
  ┌┴┐
  │ │
  └─┘
```

---

## Summary Table

| Demo | Initial Height | Nodes Compressed | Final Height | Find Calls |
|------|---------------|-----------------|--------------|------------|
| Chain of 6 | 5 | 5 | 1 | 1 |
| Partial (8 nodes) | 4 | 4 per call | 1 | 3 |
| Two trees | 5+3 | 4+2 | 2 | 2 |
| Progressive | 4 | varies | 1 | 3 |

---

## Quick Revision Questions

1. **Does path compression affect nodes that are NOT on the Find path?**
2. **After one Find(deepest_node), is the entire tree guaranteed to be flat?**
3. **How does path compression interact with Union?**
4. **In recursive Find, when does the actual compression happen — during recursion or on return?**
5. **Can path compression make a tree taller?**

---

[← Previous: Amortized Complexity](03-amortized-complexity.md) | [Next: Path Halving →](05-path-halving.md)
