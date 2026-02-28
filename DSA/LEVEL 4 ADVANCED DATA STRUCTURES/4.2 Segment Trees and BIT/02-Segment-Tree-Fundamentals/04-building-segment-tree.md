# Building a Segment Tree

## Chapter Overview

Building a segment tree is the first step before any query or update. This chapter walks through the build algorithm step by step, with full traces and complexity analysis.

---

## Build Algorithm

The build process is **recursive** and follows the divide-and-conquer pattern:

```
ALGORITHM: Build Segment Tree

Input:  Array A[0..n-1]
Output: Segment tree array tree[1..4n]

function build(node, start, end):
    if start == end:                          // BASE CASE: leaf
        tree[node] = A[start]
        return
    
    mid = (start + end) / 2
    build(2 * node, start, mid)               // build left subtree
    build(2 * node + 1, mid + 1, end)         // build right subtree
    tree[node] = merge(tree[2*node], tree[2*node+1])  // combine

// Kick off:
build(1, 0, n-1)
```

Where `merge` depends on the query type:
- Sum: `merge(a, b) = a + b`
- Min: `merge(a, b) = min(a, b)`
- Max: `merge(a, b) = max(a, b)`

---

## Complete Trace: Sum Segment Tree

```
Array A = [3, 1, 4, 1, 5, 9]   (n = 6)

build(1, 0, 5)
│
├── build(2, 0, 2)                     mid = 1
│   ├── build(4, 0, 1)                mid = 0
│   │   ├── build(8, 0, 0)  → tree[8] = A[0] = 3     LEAF
│   │   ├── build(9, 1, 1)  → tree[9] = A[1] = 1     LEAF
│   │   └── tree[4] = 3 + 1 = 4                       MERGE
│   │
│   ├── build(5, 2, 2)  → tree[5] = A[2] = 4          LEAF
│   └── tree[2] = 4 + 4 = 8                           MERGE
│
├── build(3, 3, 5)                     mid = 4
│   ├── build(6, 3, 4)                mid = 3
│   │   ├── build(12, 3, 3) → tree[12] = A[3] = 1     LEAF
│   │   ├── build(13, 4, 4) → tree[13] = A[4] = 5     LEAF
│   │   └── tree[6] = 1 + 5 = 6                       MERGE
│   │
│   ├── build(7, 5, 5)  → tree[7] = A[5] = 9          LEAF
│   └── tree[3] = 6 + 9 = 15                          MERGE
│
└── tree[1] = 8 + 15 = 23                             MERGE (ROOT)
```

**Resulting tree**:
```
              tree[1]=23
              [0-5]
             /       \
       tree[2]=8    tree[3]=15
        [0-2]        [3-5]
        /   \        /    \
   tree[4]=4 tree[5]=4 tree[6]=6 tree[7]=9
    [0-1]    [2]       [3-4]     [5]
    / \                / \
tree[8]=3 tree[9]=1  tree[12]=1 tree[13]=5
  [0]       [1]       [3]        [4]

Array: tree = [_, 23, 8, 15, 4, 4, 6, 9, 3, 1, _, _, 1, 5]
Index:         0   1  2   3  4  5  6  7  8  9 10 11 12 13
```

---

## Complete Trace: Min Segment Tree

```
Same array A = [3, 1, 4, 1, 5, 9]

Building with merge = min:

              tree[1]=1
              [0-5]
             /       \
       tree[2]=1    tree[3]=1
        [0-2]        [3-5]
        /   \        /    \
   tree[4]=1 tree[5]=4 tree[6]=1 tree[7]=9
    [0-1]    [2]       [3-4]     [5]
    / \                / \
tree[8]=3 tree[9]=1  tree[12]=1 tree[13]=5
  [0]       [1]       [3]        [4]

tree[4] = min(3, 1) = 1
tree[2] = min(1, 4) = 1
tree[6] = min(1, 5) = 1
tree[3] = min(1, 9) = 1
tree[1] = min(1, 1) = 1
```

---

## Bottom-Up Build (Iterative)

For arrays where n is a power of 2, you can build iteratively:

```
ALGORITHM: Iterative Build (n must be power of 2)

// Step 1: Copy array elements to leaves
for i = 0 to n-1:
    tree[n + i] = A[i]

// Step 2: Build internal nodes bottom-up
for i = n-1 down to 1:
    tree[i] = merge(tree[2*i], tree[2*i+1])
```

**Trace for A = [3, 1, 4, 1, 5, 9, 2, 6] (n = 8)**:

```
Step 1 - Place leaves:
Index:  0  1  2  3  4  5  6  7  │  8  9  10 11 12 13 14 15
Value:  _  _  _  _  _  _  _  _  │  3  1   4  1  5  9  2  6
                                    └── leaves at [8..15]

Step 2 - Fill internal nodes:
i=7:  tree[7] = tree[14] + tree[15] = 2 + 6 = 8
i=6:  tree[6] = tree[12] + tree[13] = 5 + 9 = 14
i=5:  tree[5] = tree[10] + tree[11] = 4 + 1 = 5
i=4:  tree[4] = tree[8]  + tree[9]  = 3 + 1 = 4
i=3:  tree[3] = tree[6]  + tree[7]  = 14+ 8 = 22
i=2:  tree[2] = tree[4]  + tree[5]  = 4 + 5 = 9
i=1:  tree[1] = tree[2]  + tree[3]  = 9 + 22= 31

Final:
Index:  0   1   2   3   4  5   6   7  │  8  9 10 11 12 13 14 15
Value:  _  31   9  22   4  5  14   8  │  3  1  4  1  5  9  2  6
```

---

## Build Order Visualization

```
Recursive build visits nodes in POST-ORDER (children before parent):

Visit order for n = 4:
  1. tree[4] (leaf, A[0])
  2. tree[5] (leaf, A[1])
  3. tree[2] (merge 4,5)       ← left subtree done
  4. tree[6] (leaf, A[2])
  5. tree[7] (leaf, A[3])
  6. tree[3] (merge 6,7)       ← right subtree done
  7. tree[1] (merge 2,3)       ← root

Total merge operations: n - 1 = 3
Total leaf assignments: n = 4
Total operations: 2n - 1 = 7
```

---

## Time Complexity of Build

```
Each node is visited exactly once.
Total nodes ≈ 2n (for leaves) + (n-1) (for internals) = ~2n

At each node: O(1) work (merge two values)

Total: O(n)

Recurrence:
  T(n) = 2 × T(n/2) + O(1)
  By Master Theorem: T(n) = O(n)
```

---

## Space Complexity of Build

```
Tree array:      O(4n) = O(n)
Recursion stack: O(log n)  (depth of tree)

Total space: O(n)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Algorithm | Recursive divide-and-conquer |
| Base case | Leaf node: tree[node] = A[start] |
| Merge step | tree[node] = merge(left_child, right_child) |
| Traversal order | Post-order (children before parent) |
| Time | O(n) — each node visited once |
| Space | O(n) for tree, O(log n) for recursion stack |
| Iterative variant | Place leaves at [n..2n-1], fill [n-1..1] bottom-up |
| Call signature | build(node_idx, segment_start, segment_end) |

---

## Quick Revision Questions

1. What is the base case in the recursive build? *(When start == end — a leaf node stores A[start])*
2. In what order does the recursive build visit nodes? *(Post-order — left child, right child, then parent)*
3. What is the time complexity of building a segment tree? *(O(n))*
4. How does the iterative build differ from recursive? *(Places leaves directly, then fills internal nodes from n-1 down to 1)*
5. How many times is the merge function called during build? *(n - 1 times — once per internal node)*
6. What is the recursion depth during build? *(O(log n) — the height of the tree)*

---

[← Previous: Array Representation](03-array-representation.md) | [Next: Range Query →](05-range-query.md)
