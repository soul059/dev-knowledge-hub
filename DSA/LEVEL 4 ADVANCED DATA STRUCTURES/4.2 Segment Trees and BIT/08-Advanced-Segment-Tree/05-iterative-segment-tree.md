# Iterative Segment Tree

## Chapter Overview

All our segment trees so far used recursion. An iterative (bottom-up) segment tree eliminates recursion overhead, uses exactly 2n space, and is faster in practice. This chapter covers the elegant iterative approach.

---

## Why Iterative?

```
Recursive segment tree:
  ✗ Function call overhead (log n calls per operation)
  ✗ Stack frame allocation
  ✗ 4n array size (to handle rounding)
  ✗ Harder to optimize

Iterative segment tree:
  ✓ Simple loops (no recursion)
  ✓ Exactly 2n array size
  ✓ 2-5× faster in practice
  ✓ Cache-friendly memory access
  ✓ Very short code
```

---

## Array Layout

```
For n = 4 elements A[0..3]:

Array tree[0..2n-1]:    (0-indexed, tree[0] unused)

Index:  0   1   2   3   4   5   6   7
        _  root  .   .   A0  A1  A2  A3
             ↑   ↑   ↑    ↑   ↑   ↑   ↑
            [1] [2] [3]  [4] [5] [6] [7]

Tree structure:
           tree[1]          ← root
          /        \
      tree[2]     tree[3]
      /    \      /    \
  tree[4] tree[5] tree[6] tree[7]
    A[0]    A[1]    A[2]    A[3]

Key relationships:
  Leaves:       tree[n + i] = A[i]    (for i = 0..n-1)
  Parent:       tree[i / 2]
  Left child:   tree[2 * i]
  Right child:  tree[2 * i + 1]
  Sibling:      tree[i ^ 1]     (XOR with 1)
```

---

## Build

```
build(A, n):
    // Copy leaves
    for i from 0 to n-1:
        tree[n + i] = A[i]
    
    // Build internal nodes bottom-up
    for i from n-1 down to 1:
        tree[i] = merge(tree[2*i], tree[2*i+1])

// Time: O(n)
// Space: exactly 2n

Trace for A = [3, 1, 4, 2]:
  n = 4
  Leaves: tree[4]=3, tree[5]=1, tree[6]=4, tree[7]=2
  
  i=3: tree[3] = merge(tree[6], tree[7]) = merge(4, 2) = 6
  i=2: tree[2] = merge(tree[4], tree[5]) = merge(3, 1) = 4
  i=1: tree[1] = merge(tree[2], tree[3]) = merge(4, 6) = 10

  tree: [_, 10, 4, 6, 3, 1, 4, 2]
```

---

## Point Update

```
update(pos, val):
    pos += n                   // go to leaf
    tree[pos] = val            // update leaf
    while pos > 1:             // walk up to root
        pos = pos / 2          // go to parent
        tree[pos] = merge(tree[2*pos], tree[2*pos+1])

// Time: O(log n)

Alternative (slightly faster):
update(pos, val):
    pos += n
    tree[pos] = val
    while pos > 1:
        // Update parent using sibling
        tree[pos >> 1] = merge(tree[pos], tree[pos ^ 1])
        // But be careful: tree[pos ^ 1] may be left or right
        // Correct version:
        tree[pos >> 1] = merge(tree[pos & ~1], tree[pos | 1])
        pos >>= 1

Trace: update(1, 5) on tree [_, 10, 4, 6, 3, 1, 4, 2]
  pos = 1 + 4 = 5
  tree[5] = 5
  tree: [_, 10, 4, 6, 3, 5, 4, 2]
  
  pos = 5/2 = 2
  tree[2] = merge(tree[4], tree[5]) = merge(3, 5) = 8
  tree: [_, 10, 8, 6, 3, 5, 4, 2]
  
  pos = 2/2 = 1
  tree[1] = merge(tree[2], tree[3]) = merge(8, 6) = 14
  tree: [_, 14, 8, 6, 3, 5, 4, 2]
```

---

## Range Query

```
query(l, r):    // query on [l, r] (0-indexed)
    result = identity
    l += n      // go to left leaf
    r += n + 1  // go past right leaf (exclusive)
    while l < r:
        if l is odd:     // l is a right child → include it
            result = merge(result, tree[l])
            l += 1       // move to next subtree
        if r is odd:     // r is a right child → include its left sibling
            r -= 1
            result = merge(result, tree[r])
        l >>= 1         // go to parent level
        r >>= 1
    return result

// Time: O(log n)
```

---

## Query Walkthrough

```
A = [3, 1, 4, 2],  tree = [_, 10, 4, 6, 3, 1, 4, 2]
Query: sum of [1, 2] (0-indexed) = A[1] + A[2] = 1 + 4 = 5

l = 1 + 4 = 5,  r = 2 + 4 + 1 = 7
result = 0 (identity for sum)

Iteration 1: l=5, r=7
  l=5 is odd → result = merge(0, tree[5]) = 1, l = 6
  r=7 is odd → r = 6, result = merge(1, tree[6]) = 5
  l = 6>>1 = 3, r = 6>>1 = 3

l == r → STOP
return 5 ✓

Visual of which nodes are visited:
           tree[1]=10
          /           \
     tree[2]=4     tree[3]=6
      /    \        /     \
  tree[4]  tree[5] tree[6] tree[7]
    (3)     (1)✓    (4)✓    (2)

Only tree[5] and tree[6] are accessed → O(log n)
```

---

## Non-Commutative Operations

```
For commutative operations (sum, min, max, XOR, GCD):
  The basic query above works perfectly.

For non-commutative operations (matrix multiplication, etc.):
  We must collect left-side and right-side results separately.

query_noncommutative(l, r):
    res_left = identity
    res_right = identity
    l += n
    r += n + 1
    while l < r:
        if l & 1:
            res_left = merge(res_left, tree[l])  // left side
            l += 1
        if r & 1:
            r -= 1
            res_right = merge(tree[r], res_right) // right side (prepend!)
        l >>= 1
        r >>= 1
    return merge(res_left, res_right)

// res_left collects from left boundary going right
// res_right collects from right boundary going left
// Final merge combines them in correct order
```

---

## When N is Not a Power of 2

```
The iterative segment tree works even when n is NOT a power of 2!

Example: n = 5, A = [a, b, c, d, e]

tree indices: [_, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
              root                              leaves

Leaves: tree[5]=a, tree[6]=b, tree[7]=c, tree[8]=d, tree[9]=e

          tree[1]
         /       \
     tree[2]     tree[3]
     /    \      /    \
  tree[4] tree[5] tree[6] tree[7]
   /  \     a       b       c
tree[8] tree[9]
   d       e

tree[4] = merge(tree[8], tree[9]) = merge(d, e)
tree[2] = merge(tree[4], tree[5]) = merge(merge(d,e), a)  ← note: d,e before a!

Wait — the leaves are NOT in order!

For non-power-of-2, the mapping is:
  tree[n + i] = A[i]   ← this still holds
  But the tree structure may have "wrap-around"

This works correctly because the query algorithm 
collects the right nodes regardless of tree shape.

Simplification: Pad n to next power of 2 with identity values.
  n_padded = next_power_of_2(n)
  Wastes at most n extra space.
```

---

## Complete Implementation Template (Sum)

```
MAXN = 100005
tree[2 * MAXN]
n = 0

build(a[], size):
    n = size
    for i from 0 to n-1:
        tree[n + i] = a[i]
    for i from n-1 down to 1:
        tree[i] = tree[2*i] + tree[2*i+1]

update(pos, val):        // 0-indexed position
    pos += n
    tree[pos] = val
    while pos > 1:
        pos >>= 1
        tree[pos] = tree[2*pos] + tree[2*pos+1]

query(l, r):             // 0-indexed, inclusive [l, r]
    res = 0
    l += n
    r += n + 1
    while l < r:
        if l & 1: res += tree[l]; l++
        if r & 1: r--; res += tree[r]
        l >>= 1
        r >>= 1
    return res

// That's it! ~20 lines total.
```

---

## Iterative vs Recursive Comparison

```
                    Recursive          Iterative
                    ─────────          ─────────
Code length         ~40-50 lines       ~20 lines
Array size          4n                 2n
Speed               Baseline           2-5× faster
Function calls      O(log n) per op    0
Stack usage         O(log n)           O(1)
Supports lazy       Yes (standard)     Yes (but harder)
Non-power-of-2      Needs 4n padding   Works directly
Code clarity        Clear structure    Elegant but tricky
```

---

## Iterative Lazy Propagation (Advanced)

```
Iterative lazy propagation is possible but more complex.

The key challenge: in recursive, push_down happens naturally
during top-down traversal. In iterative, we go bottom-up.

Solution: Before query/update, push down from root to
the relevant leaves.

push_down_path(pos):
    pos += n
    // Push down from root to pos
    for i from log2(n) down to 1:
        ancestor = pos >> i
        if lazy[ancestor] != identity:
            push_down(ancestor)

This adds complexity. For lazy propagation, many prefer
the recursive version for clarity.
```

---

## Summary Table

| Aspect | Iterative | Recursive |
|--------|----------|----------|
| Space | 2n | 4n |
| Speed | Faster (no recursion) | Slower |
| Code length | ~20 lines | ~40-50 lines |
| Lazy support | Complex | Natural |
| Non-commutative | Needs care | Straightforward |
| Preferred for | Speed-critical, simple ops | Complex ops, lazy |

---

## Quick Revision Questions

1. How much space does an iterative segment tree use? *(Exactly 2n, compared to 4n for recursive)*
2. Where are the leaves stored? *(At tree[n + i] for element A[i])*
3. How does the query handle non-commutative operations? *(Collect left-side and right-side results separately, then merge in correct order)*
4. What is the sibling of tree[i]? *(tree[i ^ 1] — XOR with 1)*
5. Why is the iterative version faster? *(No function call overhead, no stack allocation, better branch prediction, 2× less memory = better cache)*

---

[← Previous: Merge Sort Tree](04-merge-sort-tree.md) | [Next: Dynamic Segment Tree →](06-dynamic-segment-tree.md)
