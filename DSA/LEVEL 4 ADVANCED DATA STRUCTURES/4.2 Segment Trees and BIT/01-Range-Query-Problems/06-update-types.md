# Update Types (Point, Range)

## Chapter Overview

Updates change the array between queries. The type of update — **point** or **range** — determines which data structure and techniques are needed. This chapter covers both types in detail.

---

## Point Update

A point update modifies a **single element** in the array.

```
Point Update(index, value):
  A[index] = value        ← assignment
  OR
  A[index] += value       ← increment

Example:
  Before: A = [3, 1, 4, 1, 5, 9, 2, 6]
  
  Update: A[3] = 7
  After:  A = [3, 1, 4, 7, 5, 9, 2, 6]
                       ↑ changed
```

### How Point Update Affects the Tree

Only the **path from leaf to root** needs updating — O(log n) nodes.

```
Before update A[3] = 7 (was 1, diff = +6):

              [0-7] sum=31
             /          \
        [0-3] sum=9    [4-7] sum=22
        /    \          /     \
    [0-1]=4 [2-3]=5  [4-5]=14 [6-7]=8
     / \     / \      / \      / \
    3   1   4   1    5   9    2   6
                ↑
            Update this leaf

After update:
              [0-7] sum=37  ← +6
             /          \
        [0-3] sum=15   [4-7] sum=22  ← unchanged
        /    \          
    [0-1]=4 [2-3]=11  ← +6
     / \     / \
    3   1   4   7     ← changed to 7
                ↑

Path updated: leaf → [2-3] → [0-3] → [0-7]
Nodes changed: 4 out of 15 = O(log n)
```

### Pseudocode: Point Update

```
function pointUpdate(node, start, end, index, value):
    if start == end:                    // leaf node
        tree[node] = value
        return
    
    mid = (start + end) / 2
    if index <= mid:
        pointUpdate(2*node, start, mid, index, value)
    else:
        pointUpdate(2*node+1, mid+1, end, index, value)
    
    tree[node] = merge(tree[2*node], tree[2*node+1])
```

---

## Range Update

A range update modifies **all elements** in a subarray.

```
Range Update(L, R, value):
  For all i in [L, R]: A[i] += value    ← range increment
  OR
  For all i in [L, R]: A[i] = value     ← range assignment

Example:
  Before: A = [3, 1, 4, 1, 5, 9, 2, 6]

  Range Update: Add 10 to A[2..5]
  After:  A = [3, 1, 14, 11, 15, 19, 2, 6]
                    +10  +10  +10  +10
```

### The Naive Problem

```
Naive range update: iterate through [L, R] and update each

for i = L to R:
    pointUpdate(i, A[i] + value)

Cost: O((R-L+1) × log n)
Worst case: O(n × log n) per range update  ← TOO SLOW!
```

### The Solution: Lazy Propagation

Instead of immediately updating all nodes, we **mark nodes as "pending"** and only propagate when needed.

```
Range Update: Add 10 to A[2..5]

WITHOUT lazy (update every leaf):
  Update A[2] → O(log n)
  Update A[3] → O(log n)
  Update A[4] → O(log n)
  Update A[5] → O(log n)
  Total: O(4 × log n)

WITH lazy (mark relevant nodes):
              [0-7] 
             /      \
        [0-3]       [4-7]
        /    \       /    \
    [0-1]  [2-3]  [4-5]  [6-7]
    
  Mark [2-3] with lazy=10  → covers A[2], A[3]
  Mark [4-5] with lazy=10  → covers A[4], A[5]
  Update ancestor sums
  
  Total: O(log n) nodes visited!
```

---

## Update Type Comparison

```
┌────────────────┬──────────────┬──────────────────────┐
│                │ Point Update │   Range Update       │
├────────────────┼──────────────┼──────────────────────┤
│ Elements       │ 1            │ R - L + 1            │
│ changed        │              │                      │
├────────────────┼──────────────┼──────────────────────┤
│ Seg Tree cost  │ O(log n)     │ O(log n) with lazy   │
│                │              │ O(n log n) without   │
├────────────────┼──────────────┼──────────────────────┤
│ BIT cost       │ O(log n)     │ O(log n) with trick  │
├────────────────┼──────────────┼──────────────────────┤
│ Prefix sum     │ O(n) rebuild │ O(n) rebuild         │
│ cost           │              │                      │
├────────────────┼──────────────┼──────────────────────┤
│ Implementation │ Simple       │ Requires lazy prop   │
│ complexity     │              │ or difference array   │
└────────────────┴──────────────┴──────────────────────┘
```

---

## Assignment vs Increment

Two common update semantics:

```
ASSIGNMENT: A[i] = value
  "Set element i to value"
  Previous value is discarded
  
  Example: Set price of item 3 to $50
  A = [10, 20, 30, 40] → Update(2, 50) → A = [10, 20, 50, 40]

INCREMENT: A[i] += value  
  "Add value to element i"
  Previous value is kept and added to
  
  Example: Add 5 bonus points to student 3
  A = [90, 85, 70, 95] → Update(2, +5) → A = [90, 85, 75, 95]
```

**Impact on Segment Tree**:

```
Assignment update of leaf:
  tree[leaf] = new_value
  Propagate up: parent = merge(left_child, right_child)

Increment update of leaf:
  tree[leaf] += delta
  Propagate up: parent.sum += delta  (for sum tree)
  
For range assignment with lazy:
  lazy[node] = assigned_value   (overwrites previous lazy)
  
For range increment with lazy:
  lazy[node] += delta           (accumulates with previous lazy)
```

---

## Difference Array Technique

For **range increment** on static queries (all updates first, then all queries), use a difference array:

```
Original: A = [0, 0, 0, 0, 0, 0, 0, 0]
                0  1  2  3  4  5  6  7

Add 5 to range [2, 5]:
  diff[2] += 5, diff[6] -= 5
  diff = [0, 0, 5, 0, 0, 0, -5, 0]

Add 3 to range [4, 7]:
  diff[4] += 3, diff[8] -= 3
  diff = [0, 0, 5, 0, 3, 0, -5, 0]  (index 8 out of bounds, ignore)

Compute prefix sum of diff to get final array:
  A = [0, 0, 5, 5, 8, 8, 3, 3]
  
Verify: positions 2-5 got +5, positions 4-7 got +3
  pos 0,1: 0       ✓
  pos 2,3: 5       ✓ (+5 only)
  pos 4,5: 8       ✓ (+5 and +3)
  pos 6,7: 3       ✓ (+3 only)
```

**Limitation**: O(n) to read the final state — not suitable for interleaved queries and updates.

---

## Problem Patterns by Update Type

```
┌──────────────────────────────────────────────────────────────┐
│  Pattern                           │ Update │ Structure      │
├────────────────────────────────────┼────────┼────────────────┤
│  Dynamic prefix sum                │ Point  │ BIT            │
│  Dynamic range min/max             │ Point  │ Segment Tree   │
│  Range add + range sum             │ Range  │ Seg Tree+Lazy  │
│  Range set + range min             │ Range  │ Seg Tree+Lazy  │
│  Batch range add, then query       │ Range  │ Diff Array     │
│  Range add + point query           │ Range  │ BIT (reverse)  │
│  Flip bits in range + count ones   │ Range  │ Seg Tree+Lazy  │
└──────────────────────────────────────────────┴────────────────┘
```

---

## Summary Table

| Update Type | Modifies | Seg Tree | BIT | Lazy Needed? |
|-----------|----------|:--------:|:---:|:------------:|
| Point Assignment | 1 element | O(log n) | O(log n) | No |
| Point Increment | 1 element | O(log n) | O(log n) | No |
| Range Increment | [L,R] | O(log n) | O(log n)* | Yes (ST) |
| Range Assignment | [L,R] | O(log n) | Hard | Yes |
| Batch Range (offline) | [L,R] | - | - | Diff Array |

\* BIT supports range increment + point query using the difference array idea.

---

## Quick Revision Questions

1. How many tree nodes are affected by a point update? *(O(log n) — the path from leaf to root)*
2. Why is naive range update slow on a segment tree? *(It performs O(R-L+1) point updates, each costing O(log n))*
3. What is lazy propagation in one sentence? *(Defer updates to child nodes until they're actually needed)*
4. What is the difference array technique used for? *(Batch range increments followed by a single read of the final array)*
5. Can BIT handle range assignment updates efficiently? *(Not directly — BIT is best for range increment; range assignment typically needs a segment tree)*

---

[← Previous: Query Types](05-query-types.md) | [Next Unit: Segment Tree Fundamentals →](../02-Segment-Tree-Fundamentals/01-what-is-a-segment-tree.md)
