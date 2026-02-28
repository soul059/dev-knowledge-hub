# Segment Tree with Sets / Multisets

## Chapter Overview

Sometimes each node of a segment tree needs to store a collection — a set, multiset, or sorted list — instead of a single aggregate value. This chapter covers segment trees augmented with ordered containers to answer questions like "count of distinct elements" or "k-th smallest in range."

---

## When Do We Need Sets in Nodes?

```
Standard segment tree node stores ONE value (sum, min, etc.)
But some queries need MORE information:

1. "How many distinct elements in [L, R]?"
2. "How many elements in [L, R] are < X?"
3. "K-th smallest in [L, R]"
4. "What elements appear in every subarray of [L, R]?"

These require storing actual elements, not just aggregates.
```

---

## Approach: Merge Sort Tree

```
Each node stores the SORTED LIST of all elements in its range.

Example: A = [3, 1, 4, 1, 5, 9, 2, 6]

                    [1,1,2,3,4,5,6,9]          ← root [1..8]
                   /                  \
          [1,1,3,4]                [2,5,6,9]    ← [1..4], [5..8]
          /        \               /        \
       [1,3]      [1,4]        [5,9]      [2,6]
       / \        / \          / \        / \
     [3] [1]    [4] [1]     [5] [9]    [2] [6]

Build: merge children's sorted lists → O(n log n) total space
```

---

## Build

```
build(node, l, r):
    if l == r:
        tree[node] = [A[l]]    // single-element sorted list
        return
    mid = (l + r) / 2
    build(2*node, l, mid)
    build(2*node+1, mid+1, r)
    tree[node] = merge_sorted(tree[2*node], tree[2*node+1])

// Total elements across all nodes:
//   Level 0 (root): n elements
//   Level 1: n elements (split into 2 lists)
//   ...
//   Level log(n): n elements (n lists of 1)
//   Total: n × log(n) elements
//   Space: O(n log n)
```

---

## Query: Count of Elements ≤ X in Range [L, R]

```
count_leq(node, l, r, ql, qr, X):
    if ql > r or qr < l:
        return 0
    if ql <= l and r <= qr:
        // Binary search for X in sorted list at this node
        return upper_bound(tree[node], X)
    mid = (l + r) / 2
    return count_leq(2*node, l, mid, ql, qr, X)
         + count_leq(2*node+1, mid+1, r, ql, qr, X)

// Each decomposed node: O(log n) for binary search
// Number of decomposed nodes: O(log n)
// Total: O(log² n)
```

---

## Query: K-th Smallest in Range [L, R]

```
Strategy: Binary search on the answer!

kth_smallest(ql, qr, k):
    lo = min_value, hi = max_value
    while lo < hi:
        mid = (lo + hi) / 2
        count = count_leq(1, 1, n, ql, qr, mid)
        if count >= k:
            hi = mid
        else:
            lo = mid + 1
    return lo

// Outer binary search: O(log V) where V = value range
// Inner count_leq: O(log² n)
// Total: O(log V × log² n)

// Compare with persistent segment tree: O(log n) per query
// Merge sort tree is simpler but slower
```

---

## Fractional Cascading Optimization

```
Standard: O(log² n) per count query (binary search at each node)

Fractional Cascading: O(log n) per query (eliminate binary searches)

Idea: Precompute "pointers" between parent and child sorted lists
so that binary search at parent gives position in children for free.

For each element in parent's list, store:
  - Position in left child's list
  - Position in right child's list

After one binary search at the root, cascade down in O(1) per level.

Total: O(log n) per query instead of O(log² n)

In practice: The O(log² n) version is usually fast enough,
and fractional cascading adds significant code complexity.
```

---

## Segment Tree with Multisets (Dynamic)

```
For problems with updates, use actual set/multiset containers:

Each node stores a balanced BST (set/multiset):
  - Insert: O(log n) per BST × O(log n) nodes = O(log² n)
  - Delete: O(log n) per BST × O(log n) nodes = O(log² n)
  - Query:  O(log n) per BST × O(log n) nodes = O(log² n)

Pseudocode for insert at position i, value v:

update(node, l, r, i, old_val, new_val):
    tree[node].erase(old_val)    // remove from multiset
    tree[node].insert(new_val)   // insert into multiset
    if l == r: return
    mid = (l + r) / 2
    if i <= mid:
        update(2*node, l, mid, i, old_val, new_val)
    else:
        update(2*node+1, mid+1, r, i, old_val, new_val)

Space: O(n log n) — each element appears in O(log n) nodes
```

---

## Worked Example: Count Elements in Range < X

```
A = [3, 1, 4, 2]
Query: How many elements in [1..3] are less than 3?

Merge sort tree:
        [1,2,3,4]       root [1..4]
        /        \
    [1,3]       [2,4]   [1..2], [3..4]
    / \         / \
  [3] [1]     [4] [2]

Query range [1..3] decomposes to: [1..2] and [3..3]

Node [1..2]: sorted list = [1, 3]
  Elements < 3: upper_bound for 2 → position 1 → count = 1

Node [3..3]: sorted list = [4]
  Elements < 3: upper_bound for 2 → position 0 → count = 0

Total: 1 + 0 = 1

Verify: A[1..3] = [3, 1, 4], elements < 3 = {1} → count = 1 ✓
```

---

## Comparison with Alternatives

```
┌─────────────────┬──────────────┬───────────┬──────────────┐
│ Problem         │ Merge Sort   │ Persistent│ Wavelet Tree │
│                 │ Tree         │ Seg Tree  │              │
├─────────────────┼──────────────┼───────────┼──────────────┤
│ K-th smallest   │ O(log V·     │ O(log n)  │ O(log V)     │
│                 │  log² n)     │           │              │
│ Count ≤ X       │ O(log² n)   │ O(log n)  │ O(log V)     │
│ Space           │ O(n log n)  │ O(n log n)│ O(n log V)   │
│ Updates         │ O(log² n)   │ Hard      │ Hard         │
│ Code complexity │ Moderate    │ Moderate  │ Complex      │
│ Build time      │ O(n log n)  │ O(n log n)│ O(n log V)   │
└─────────────────┴──────────────┴───────────┴──────────────┘
```

---

## Summary Table

| Aspect | Merge Sort Tree | With Multisets |
|--------|----------------|---------------|
| Storage per node | Sorted array | Balanced BST |
| Build | O(n log n) | O(n log n) |
| Count ≤ X | O(log² n) | O(log² n) |
| Updates | Rebuild needed | O(log² n) |
| Space | O(n log n) | O(n log n) |
| Best for | Static data | Dynamic data |

---

## Quick Revision Questions

1. What does each node in a merge sort tree store? *(A sorted list of all elements in its range)*
2. What is the total space of a merge sort tree? *(O(n log n) — each element appears in O(log n) nodes)*
3. How does count_leq query work? *(Decompose range into O(log n) nodes, binary search within each → O(log² n))*
4. How can k-th smallest be answered with a merge sort tree? *(Binary search on the answer value, using count_leq as the decision function)*
5. What is fractional cascading? *(Precomputing position pointers between parent and child lists to eliminate per-node binary searches, reducing O(log² n) to O(log n))*

---

[← Previous: Persistent Segment Tree](02-persistent-segment-tree.md) | [Next: Merge Sort Tree →](04-merge-sort-tree.md)
