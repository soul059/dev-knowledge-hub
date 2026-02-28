# Build Function

## Chapter Overview

The build function constructs the segment tree from the array in O(n) time. This chapter provides production-ready pseudocode, multiple implementation styles, and detailed correctness analysis.

---

## Standard Recursive Build

```
function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    
    mid = start + (end - start) / 2      // safe mid calculation
    build(2 * node, start, mid)           // build left half
    build(2 * node + 1, mid + 1, end)     // build right half
    tree[node] = merge(tree[2*node], tree[2*node+1])
```

**Call**: `build(1, 0, n-1)`

---

## Execution Flow Diagram

```
build(1, 0, 7) for A = [5, 8, 6, 3, 2, 7, 1, 4]

                    build(1,0,7)
                   /            \
          build(2,0,3)        build(3,4,7)
         /          \          /          \
    build(4,0,1)  build(5,2,3)  build(6,4,5)  build(7,6,7)
     /      \      /      \      /      \      /      \
  b(8,0,0) b(9,1,1) b(10,2,2) b(11,3,3) b(12,4,4) b(13,5,5) b(14,6,6) b(15,7,7)
   leaf=5   leaf=8    leaf=6    leaf=3    leaf=2    leaf=7    leaf=1    leaf=4

Merge order (post-order):
  tree[8]=5, tree[9]=8  → tree[4]=13
  tree[10]=6, tree[11]=3 → tree[5]=9
  tree[4]=13, tree[5]=9  → tree[2]=22
  tree[12]=2, tree[13]=7 → tree[6]=9
  tree[14]=1, tree[15]=4 → tree[7]=5
  tree[6]=9, tree[7]=5   → tree[3]=14
  tree[2]=22, tree[3]=14 → tree[1]=36

Final tree array:
idx:  1   2   3   4  5  6  7  8 9 10 11 12 13 14 15
val: 36  22  14  13  9  9  5  5 8  6  3  2  7  1  4
```

---

## Build for Different Operations

### Min Segment Tree Build
```
A = [5, 8, 6, 3, 2, 7, 1, 4]

merge(a, b) = min(a, b)

             1:[0-7]=1
            /          \
       2:[0-3]=3      3:[4-7]=1
       /    \          /      \
   4:[0-1]=5 5:[2-3]=3 6:[4-5]=2 7:[6-7]=1
   / \       / \       / \        / \
  5   8     6   3     2   7      1   4
```

### Max Segment Tree Build
```
merge(a, b) = max(a, b)

             1:[0-7]=8
            /          \
       2:[0-3]=8      3:[4-7]=7
       /    \          /      \
   4:[0-1]=8 5:[2-3]=6 6:[4-5]=7 7:[6-7]=4
   / \       / \       / \        / \
  5   8     6   3     2   7      1   4
```

---

## Iterative Build (Bottom-Up)

When possible (especially for power-of-2 sizes or with padding):

```
function buildIterative():
    // Step 1: Copy leaves
    for i = 0 to n-1:
        tree[n + i] = A[i]
    
    // Step 2: Build internal nodes
    for i = n-1 downto 1:
        tree[i] = merge(tree[2*i], tree[2*i+1])
```

**Comparison with recursive build**:

```
┌─────────────────┬──────────────────┬──────────────────┐
│                 │    Recursive     │    Iterative     │
├─────────────────┼──────────────────┼──────────────────┤
│  Direction      │  Top-down        │  Bottom-up       │
│  Stack space    │  O(log n)        │  O(1)            │
│  Time           │  O(n)            │  O(n)            │
│  Flexibility    │  Any n           │  Needs n = 2^k   │
│  Code length    │  Longer          │  Shorter          │
│  Handles n≠2^k  │  Yes             │  Pad to 2^k      │
└─────────────────┴──────────────────┴──────────────────┘
```

---

## Handling Non-Power-of-2 Sizes

When n is not a power of 2:

```
Method 1: Pad array to next power of 2 with identity values

A = [5, 8, 6, 3, 2]  (n=5)
Padded: A = [5, 8, 6, 3, 2, 0, 0, 0]  (padded to 8 with identity=0 for sum)

Method 2: Use recursive build (handles any n naturally)

build(1, 0, 4):
             [0-4]=24
            /        \
       [0-2]=19      [3-4]=5
       /    \        /    \
    [0-1]=13 [2]=6 [3]=3  [4]=2
    / \
   5   8
   
Works perfectly! No padding needed.
```

---

## Build with Multiple Values per Node

Sometimes nodes store more than one value:

```
// Node stores both sum and count of elements
struct Node {
    sum: integer
    count: integer
}

function merge(a, b):
    return Node(a.sum + b.sum, a.count + b.count)

function build(node, start, end):
    if start == end:
        tree[node] = Node(A[start], 1)    // sum = value, count = 1
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = merge(tree[2*node], tree[2*node+1])

// Result: each node knows the sum AND count of its segment
```

---

## Correctness Proof (Inductive)

```
CLAIM: After build(node, start, end), tree[node] = merge(A[start..end])

Base case (start == end):
  tree[node] = A[start] = merge of a single element  ✓

Inductive step:
  Assume build correctly fills tree[2*node] = merge(A[start..mid])
  Assume build correctly fills tree[2*node+1] = merge(A[mid+1..end])
  
  Then: tree[node] = merge(tree[2*node], tree[2*node+1])
                   = merge(merge(A[start..mid]), merge(A[mid+1..end]))
                   = merge(A[start..end])    (by associativity of merge)
                                              ✓
```

---

## Performance Analysis

```
Nodes created: exact count depends on n

For n = 2^k: 2n - 1 nodes
For general n: between n and 2n nodes

Each node: O(1) work (one merge operation)

Total time: O(n)

Recurrence: T(n) = 2T(n/2) + O(1)
Master theorem: a=2, b=2, c=0
  log_b(a) = 1 > c = 0
  → T(n) = O(n^1) = O(n)  ✓
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Time complexity | O(n) |
| Space complexity | O(4n) for tree array |
| Stack depth | O(log n) |
| Base case | Leaf: tree[node] = A[start] |
| Merge step | tree[node] = merge(left_child, right_child) |
| Traversal | Post-order (children before parent) |
| Iterative variant | Leaves first at [n..2n-1], fill [n-1..1] |
| Non-power-of-2 | Recursive handles naturally; iterative needs padding |
| Multi-value nodes | Store struct, merge each field |

---

## Quick Revision Questions

1. What is the time complexity of building a segment tree? *(O(n))*
2. In what traversal order does the recursive build visit nodes? *(Post-order)*
3. How does the iterative build handle array sizes that aren't powers of 2? *(Pad the array with identity elements to the next power of 2)*
4. Why does correctness rely on the merge function being associative? *(Splitting the range and merging partial results must equal merging the full range)*
5. How many merge operations are performed during build? *(n - 1, one per internal node)*
6. What is the stack depth during recursive build? *(O(log n))*

---

[← Previous: Recursive Implementation](01-recursive-implementation.md) | [Next: Query Function →](03-query-function.md)
