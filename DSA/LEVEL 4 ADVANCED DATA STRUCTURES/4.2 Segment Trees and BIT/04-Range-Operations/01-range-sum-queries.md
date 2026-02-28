# Range Sum Queries

## Chapter Overview

Range Sum Query (RSQ) is the most common application of segment trees. This chapter covers the complete implementation with detailed examples and discusses how sum's mathematical properties make it particularly well-suited for tree-based structures.

---

## Problem Statement

```
Given array A[0..n-1]:
  - Query(L, R): Return A[L] + A[L+1] + ... + A[R]
  - Update(i, val): Set A[i] = val

Support both operations in O(log n).
```

---

## Why Sum Works Perfectly

Sum has all the properties a segment tree merge function needs:

```
1. ASSOCIATIVE:   (a + b) + c = a + (b + c)     ✓
   → Can split range and merge partial sums

2. COMMUTATIVE:   a + b = b + a                  ✓
   → Order of merging doesn't matter

3. HAS IDENTITY:  a + 0 = a                      ✓
   → "No overlap" returns 0 without affecting result

4. INVERTIBLE:    a + b - b = a                   ✓
   → Can also use prefix sums (but not with updates)
```

---

## Complete Implementation

```
// Globals
tree[4*MAXN]
A[MAXN]
n

function merge(a, b):
    return a + b

IDENTITY = 0

function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = tree[2*node] + tree[2*node+1]

function query(node, start, end, L, R):
    if end < L or start > R:
        return 0                              // identity
    if L <= start and end <= R:
        return tree[node]
    mid = (start + end) / 2
    return query(2*node, start, mid, L, R) +
           query(2*node+1, mid+1, end, L, R)

function update(node, start, end, idx, val):
    if start == end:
        tree[node] = val
        return
    mid = (start + end) / 2
    if idx <= mid:
        update(2*node, start, mid, idx, val)
    else:
        update(2*node+1, mid+1, end, idx, val)
    tree[node] = tree[2*node] + tree[2*node+1]
```

---

## Worked Example

```
A = [1, 3, 5, 7, 9, 11]   n = 6

Build the sum segment tree:

              1:[0-5] = 36
             /            \
        2:[0-2] = 9      3:[3-5] = 27
        /      \          /       \
    4:[0-1]=4  5:[2]=5  6:[3-4]=16  7:[5]=11
    / \                 / \
  8:1  9:3           12:7  13:9

Query: sum(1, 4)
  = A[1] + A[2] + A[3] + A[4]
  = 3 + 5 + 7 + 9
  = 24

Tree traversal:
  query(1, 0, 5, 1, 4) → PARTIAL
    query(2, 0, 2, 1, 4) → PARTIAL
      query(4, 0, 1, 1, 4) → PARTIAL
        query(8, 0, 0, 1, 4) → NO OVERLAP → 0
        query(9, 1, 1, 1, 4) → COMPLETE → 3
        merge(0, 3) = 3
      query(5, 2, 2, 1, 4) → COMPLETE → 5
      merge(3, 5) = 8
    query(3, 3, 5, 1, 4) → PARTIAL
      query(6, 3, 4, 1, 4) → COMPLETE → 16
      query(7, 5, 5, 1, 4) → NO OVERLAP → 0
      merge(16, 0) = 16
    merge(8, 16) = 24  ✓

Update: A[2] = 10 (was 5)
  update(1, 0, 5, 2, 10)
    tree[5] = 10
    tree[2] = tree[4] + tree[5] = 4 + 10 = 14
    tree[1] = tree[2] + tree[3] = 14 + 27 = 41
```

---

## Common Problem Variations

### 1. Range Sum + Point Increment
```
// Instead of "set to val", add delta to element
function incrementUpdate(node, start, end, idx, delta):
    if start == end:
        tree[node] += delta
        return
    mid = (start + end) / 2
    if idx <= mid:
        incrementUpdate(2*node, start, mid, idx, delta)
    else:
        incrementUpdate(2*node+1, mid+1, end, idx, delta)
    tree[node] = tree[2*node] + tree[2*node+1]
```

### 2. Range Sum + Range Increment (needs Lazy Propagation)
```
// Add delta to all elements in [L, R]
// Covered in detail in Unit 5 (Lazy Propagation)
```

### 3. Subarray Sum with Modular Arithmetic
```
// sum(L, R) mod M
merge(a, b) = (a + b) % M
// Works because modular addition is associative
```

---

## Applications

| Application | What's Being Summed |
|------------|-------------------|
| Total revenue in date range | Daily revenue values |
| Cumulative frequency table | Counts per value |
| Inversion count | Processed element counts |
| Rectangle area (sweep line) | Active segment lengths |
| Prefix sum with updates | Element values |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Merge function | a + b |
| Identity element | 0 |
| Build time | O(n) |
| Query time | O(log n) |
| Update time | O(log n) |
| Also works with | Prefix sums (static), BIT (dynamic) |
| Lazy propagation | Supported (range add + range sum) |

---

## Quick Revision Questions

1. What is the identity element for range sum? *(0)*
2. Can range sum be solved with BIT? *(Yes — BIT is simpler and more efficient for sum)*
3. What property of sum allows prefix sum queries to work? *(Invertibility: sum(L,R) = prefix(R) - prefix(L-1))*
4. How do you handle sum with modular arithmetic? *(merge(a,b) = (a+b) % M — still associative)*
5. What is the time complexity for Q range sum queries with point updates? *(O(Q log n))*

---

[← Previous Unit: Space Complexity](../03-Segment-Tree-Implementation/06-space-complexity.md) | [Next: Range Minimum Queries →](02-range-minimum-queries.md)
