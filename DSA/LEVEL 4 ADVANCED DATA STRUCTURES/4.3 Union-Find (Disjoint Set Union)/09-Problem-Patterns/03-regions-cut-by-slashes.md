# Chapter 3: Regions Cut by Slashes (LeetCode 959)

## Overview

Given an N×N grid where each cell contains '/', '\', or ' ' (space), determine the number of regions formed. This is a creative Union-Find problem requiring clever node modeling.

---

## Problem Statement

```
Input: grid = [
  " /",
  "/ "
]

Visual:
  ┌──┬──┐
  │  │/ │
  │ /│  │
  ├──┼──┤
  │/ │  │
  │  │  │
  └──┴──┘

Actual regions when slashes are drawn:
  ┌────────┐
  │╲      /│
  │  ╲  /  │
  │    ╳   │
  │  /  ╲  │
  │/      ╲│
  └────────┘

Hmm, let me draw it properly:
Grid (2x2):
  Cell(0,0)=' ', Cell(0,1)='/'
  Cell(1,0)='/', Cell(1,1)=' '
  
  Looks like:
    ·──·──·
    | \|/ |    →  2 regions
    ·──·──·
    |/ |\ |
    ·──·──·

Output: 2
```

---

## Approach 1: Upscale Grid (Simple)

```
Upscale each cell to 3×3:

' ' →  . . .      '/' →  . . 1      '\' →  1 . .
       . . .             . 1 .             . 1 .
       . . .             1 . .             . . 1

Then count connected components of 0s using BFS/DFS or DSU.

3N × 3N grid → standard connected components problem.
```

---

## Approach 2: Triangle Decomposition with DSU (Elegant)

```
Split each cell into 4 TRIANGLES:

     0 (top)
    /|\
   / | \
  3--+--1  (left and right)
   \ | /
    \|/
     2 (bottom)

Each cell (r, c) has 4 regions: {0, 1, 2, 3}
  0 = top triangle
  1 = right triangle
  2 = bottom triangle
  3 = left triangle

Node index: cell(r, c), triangle t → 4 * (r * N + c) + t

Rules:
  ' ' → union all 4 triangles (0,1,2,3)
  '/' → union(0,3) and union(1,2)
  '\' → union(0,1) and union(2,3)

Adjacent cells (always connected):
  cell(r,c) bottom(2) ↔ cell(r+1,c) top(0)
  cell(r,c) right(1)  ↔ cell(r,c+1) left(3)
```

---

## Visual Explanation

```
Cell with ' ' (space):
  All 4 triangles are ONE region
    ┌─────┐
    │╲ 0 /│
    │3 ╳ 1│    union(0,1), union(1,2), union(2,3)
    │/ 2 ╲│
    └─────┘

Cell with '/' :
  Top-Left and Bottom-Right are separate
    ┌─────┐
    │╲ 0 /│
    │3 ╳  │    union(0,3) and union(1,2)
    │  / 2╲│   → two groups: {0,3} and {1,2}
    └─────┘

Cell with '\' :
  Top-Right and Bottom-Left are separate
    ┌─────┐
    │╲ 0  │
    │  ╳ 1│    union(0,1) and union(2,3)
    │/ 2 ╲│    → two groups: {0,1} and {2,3}
    └─────┘

Between cells:
  Bottom of (r,c) connects to Top of (r+1,c):
    ┌─────┐
    │  2  │ cell (r,c)
    ├─────┤
    │  0  │ cell (r+1,c)
    └─────┘
    union( node(r,c,2), node(r+1,c,0) )
```

---

## Implementation

```python
def regionsBySlashes(grid):
    N = len(grid)
    uf = UnionFind(4 * N * N)  # 4 triangles per cell
    
    def node(r, c, t):
        """Get DSU node for cell (r,c), triangle t."""
        return 4 * (r * N + c) + t
    
    for r in range(N):
        for c in range(N):
            ch = grid[r][c]
            
            # Internal unions within cell
            if ch == ' ':
                # All 4 triangles connected
                uf.union(node(r,c,0), node(r,c,1))
                uf.union(node(r,c,1), node(r,c,2))
                uf.union(node(r,c,2), node(r,c,3))
            elif ch == '/':
                # Top-Left: {0,3}, Bottom-Right: {1,2}
                uf.union(node(r,c,0), node(r,c,3))
                uf.union(node(r,c,1), node(r,c,2))
            else:  # '\'
                # Top-Right: {0,1}, Bottom-Left: {2,3}
                uf.union(node(r,c,0), node(r,c,1))
                uf.union(node(r,c,2), node(r,c,3))
            
            # External unions between adjacent cells
            # Bottom of (r,c) ↔ Top of (r+1,c)
            if r + 1 < N:
                uf.union(node(r,c,2), node(r+1,c,0))
            
            # Right of (r,c) ↔ Left of (r,c+1)
            if c + 1 < N:
                uf.union(node(r,c,1), node(r,c+1,3))
    
    # Count distinct components
    return len(set(uf.find(i) for i in range(4 * N * N)))
```

---

## Trace Example

```
grid = [" /", "/ "]
N = 2

Cells:
  (0,0)=' '  (0,1)='/'
  (1,0)='/'  (1,1)=' '

Cell (0,0) ' ': 
  union(0,1), union(1,2), union(2,3)
  → {0,1,2,3} all one group

Cell (0,1) '/': 
  union(4,7), union(5,6)
  → {4,7} and {5,6}

Cell (1,0) '/':
  union(8,11), union(9,10)
  → {8,11} and {9,10}

Cell (1,1) ' ':
  union(12,13), union(13,14), union(14,15)
  → {12,13,14,15}

Between cells:
  (0,0) bottom ↔ (1,0) top: union(2, 8)
    → {0,1,2,3} ∪ {8,11} = {0,1,2,3,8,11}
    
  (0,1) bottom ↔ (1,1) top: union(6, 12)
    → {5,6} ∪ {12,13,14,15} = {5,6,12,13,14,15}
    
  (0,0) right ↔ (0,1) left: union(1, 7)
    → {0,1,2,3,8,11} ∪ {4,7} = {0,1,2,3,4,7,8,11}
    
  (1,0) right ↔ (1,1) left: union(9, 15)
    → {9,10} ∪ {5,6,12,13,14,15} = {5,6,9,10,12,13,14,15}

Final components:
  {0,1,2,3,4,7,8,11} ← 8 nodes
  {5,6,9,10,12,13,14,15} ← 8 nodes

Count = 2 ✓
```

---

## Approach Comparison

```
┌───────────────────┬──────────────┬──────────────────┐
│ Approach          │ Time         │ Space            │
├───────────────────┼──────────────┼──────────────────┤
│ Upscale 3x        │ O(9N²)       │ O(9N²)           │
│ Triangle DSU (4x) │ O(4N² × α)  │ O(4N²)           │
│ Point-based DSU   │ Complex      │ O((N+1)²)        │
└───────────────────┴──────────────┴──────────────────┘

Triangle DSU is the most elegant:
  - Clean abstraction
  - No coordinate math
  - Standard DSU template
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Count regions in slashed grid |
| DSU nodes | 4 triangles per cell |
| Node formula | 4(r×N + c) + t, t ∈ {0,1,2,3} |
| ' ' | Union all 4 triangles |
| '/' | Union {0,3} and {1,2} |
| '\\' | Union {0,1} and {2,3} |
| Between cells | Bottom↔Top, Right↔Left |

---

## Quick Revision Questions

1. **How do you split each cell into triangles?**
2. **What unions does '/' create within a cell?**
3. **How are adjacent cells connected?**
4. **What is the total number of DSU nodes for an N×N grid?**
5. **How do you count the final number of regions?**

---

[← Previous: Sentence Similarity](02-sentence-similarity.md) | [Next: Most Stones Removed →](04-most-stones-removed.md)
