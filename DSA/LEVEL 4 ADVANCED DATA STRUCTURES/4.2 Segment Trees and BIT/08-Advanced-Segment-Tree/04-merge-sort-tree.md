# Merge Sort Tree

## Chapter Overview

A Merge Sort Tree is the most practical variant of "segment tree with sorted lists." It directly mirrors the merge sort algorithm's structure and answers order-statistic queries on arbitrary ranges efficiently. This chapter covers its construction, query operations, and applications in depth.

---

## What is a Merge Sort Tree?

```
It's a segment tree where each node stores the SORTED subarray
for its range — exactly the intermediate arrays in merge sort.

Merge sort on [3, 1, 4, 1, 5, 9, 2, 6]:

Level 3 (leaves):  [3] [1] [4] [1] [5] [9] [2] [6]
Level 2 (merge):   [1,3]  [1,4]  [5,9]  [2,6]
Level 1 (merge):   [1,1,3,4]    [2,5,6,9]
Level 0 (merge):   [1,1,2,3,4,5,6,9]

This IS the merge sort tree!
Each level's arrays are stored as segment tree nodes.
```

---

## Construction

```
build(node, l, r):
    if l == r:
        tree[node] = [A[l]]
        return
    mid = (l + r) / 2
    build(2*node, l, mid)
    build(2*node+1, mid+1, r)
    
    // Merge two sorted children (like merge sort's merge step)
    tree[node] = merge(tree[2*node], tree[2*node+1])

// Time: O(n log n) — same as merge sort
// Space: O(n log n) — each element in O(log n) nodes

Space analysis:
  Level k has ⌈n/2^k⌉ nodes, each storing 2^k elements
  Total per level = n
  Levels = log n
  Total elements = n × log n
```

---

## Core Queries

### Query 1: Count of Elements ≤ X in [L, R]

```
count_leq(node, l, r, ql, qr, X):
    if ql > r or qr < l:
        return 0
    if ql <= l and r <= qr:
        // All elements in this node are within query range
        // Binary search: how many are ≤ X?
        return upper_bound(tree[node], X)  // O(log(size))
    mid = (l + r) / 2
    return count_leq(2*node, l, mid, ql, qr, X) +
           count_leq(2*node+1, mid+1, r, ql, qr, X)

// Decomposed into O(log n) nodes
// Binary search in each: O(log n) per node
// Total: O(log² n)
```

### Query 2: Count of Elements in Value Range [lo, hi] in [L, R]

```
count_in_range(ql, qr, lo, hi):
    return count_leq(1, 1, n, ql, qr, hi) 
         - count_leq(1, 1, n, ql, qr, lo - 1)

// Two count_leq queries: O(log² n)
```

### Query 3: K-th Smallest in [L, R]

```
kth_smallest(ql, qr, k):
    lo = global_min, hi = global_max
    while lo < hi:
        mid = (lo + hi) / 2
        cnt = count_leq(1, 1, n, ql, qr, mid)
        if cnt >= k:
            hi = mid      // answer is ≤ mid
        else:
            lo = mid + 1  // answer is > mid
    return lo

// Binary search on answer: O(log V) iterations
// Each iteration: O(log² n) count query
// Total: O(log V × log² n)

// After coordinate compression (V = n):
// Total: O(log n × log² n) = O(log³ n)
```

---

## Worked Example

```
A = [5, 2, 8, 1, 4, 7, 3, 6]

Merge Sort Tree:
Level 0: [1,2,3,4,5,6,7,8]
Level 1: [1,2,5,8]  [3,4,6,7]
Level 2: [2,5]  [1,8]  [4,7]  [3,6]
Level 3: [5] [2] [8] [1] [4] [7] [3] [6]

Query: 2nd smallest in A[2..6] = {2, 8, 1, 4, 7}

Binary search on answer:
  lo=1, hi=8
  
  Iteration 1: mid=4
    count_leq([2..6], 4):
      [2..2] = [2]: upper_bound(4) = 1 ← count 1
      [3..4] = [1,8]: upper_bound(4) = 1 ← count 1  
      [5..6] = [4,7]: upper_bound(4) = 1 ← count 1
    Total = 3 ≥ 2 → hi = 4
  
  Iteration 2: mid=2
    count_leq([2..6], 2):
      [2..2] = [2]: upper_bound(2) = 1
      [3..4] = [1,8]: upper_bound(2) = 1
      [5..6] = [4,7]: upper_bound(2) = 0
    Total = 2 ≥ 2 → hi = 2
  
  Iteration 3: mid=1
    count_leq([2..6], 1):
      [2..2] = [2]: upper_bound(1) = 0
      [3..4] = [1,8]: upper_bound(1) = 1
      [5..6] = [4,7]: upper_bound(1) = 0
    Total = 1 < 2 → lo = 2
  
  lo == hi == 2 → answer = 2 ✓
  (Sorted: [1,2,4,7,8], 2nd smallest = 2)
```

---

## Practical Implementation Notes

```
1. Storage: Use vectors/dynamic arrays per node
   tree = array of n*4 sorted arrays (or vectors)

2. Memory: Pre-allocate if possible
   Total elements = n × ceil(log2(n))
   For n = 10^5: ~1.7 × 10^6 elements ≈ 7 MB (int32)

3. Build efficiently:
   Don't use std::merge repeatedly — it creates temporaries
   Better: merge in-place or into pre-allocated buffers

4. Binary search:
   Use language-provided upper_bound / bisect_right
   
5. Coordinate compression for k-th queries:
   Compress values to [1..n] before binary search
   Reduces O(log V) factor to O(log n)
```

---

## Applications

```
1. Range rank queries
   "What is the rank of value X among elements in [L, R]?"
   → count_leq(L, R, X-1) + 1

2. Range frequency
   "How many times does value X appear in [L, R]?"
   → count_leq(L, R, X) - count_leq(L, R, X-1)

3. Range count less/greater
   "How many elements in [L, R] exceed X?"
   → (R - L + 1) - count_leq(L, R, X)

4. Range median
   → kth_smallest(L, R, (R-L+2)/2)

5. Range mode (approximate)
   Binary search on frequency using count queries
```

---

## Merge Sort Tree vs Persistent Segment Tree

```
                    Merge Sort Tree     Persistent Seg Tree
─────────────────   ───────────────     ───────────────────
Build               O(n log n)          O(n log n)
K-th query          O(log³ n)           O(log n)
Count ≤ X           O(log² n)           O(log n)
Space               O(n log n)          O(n log n)
Updates             Hard / O(n)         Hard (but possible)
Code complexity     Simple              Moderate
Constant factor     Moderate            Larger (pointers)

Verdict:
  - For speed: Persistent segment tree wins
  - For simplicity: Merge sort tree wins
  - For competitive programming: both are viable;
    merge sort tree is easier to code under pressure
```

---

## With Updates (Dynamic Merge Sort Tree)

```
Problem: Updates break sorted order in affected nodes.

Approach: Use balanced BSTs instead of sorted arrays

Each node stores a balanced BST (e.g., order-statistic tree):
  - Insert: add element to O(log n) nodes, O(log n) per BST
  - Delete: remove from O(log n) nodes
  - Count ≤ X: order_of_key in O(log n) per BST

Total per operation: O(log² n)

Alternative: Sqrt decomposition + sorted blocks
  Easier to implement, O(n√n) per rebuild
```

---

## Summary Table

| Query | Complexity | Method |
|-------|-----------|--------|
| Count ≤ X in [L,R] | O(log² n) | Binary search in O(log n) nodes |
| Count in [lo,hi] in [L,R] | O(log² n) | Two count_leq queries |
| K-th smallest in [L,R] | O(log³ n) | Binary search on answer |
| Rank of X in [L,R] | O(log² n) | count_leq(L,R,X-1) + 1 |
| Build | O(n log n) | Merge sort construction |
| Space | O(n log n) | Each element in log n nodes |

---

## Quick Revision Questions

1. What does each node in a merge sort tree store? *(The sorted array of all elements in its range — identical to merge sort's intermediate arrays)*
2. What is the time complexity of counting elements ≤ X in [L,R]? *(O(log² n) — O(log n) decomposed nodes × O(log n) binary search each)*
3. How does k-th smallest work on a merge sort tree? *(Binary search on the answer value, using count_leq as the predicate — O(log V × log² n))*
4. What is the space complexity? *(O(n log n) — each element appears in O(log n) nodes)*
5. How does merge sort tree compare to persistent segment tree for k-th queries? *(Merge sort tree: O(log³ n), simpler code. Persistent: O(log n), harder to implement)*

---

[← Previous: Segment Tree with Sets](03-segment-tree-with-sets.md) | [Next: Iterative Segment Tree →](05-iterative-segment-tree.md)
