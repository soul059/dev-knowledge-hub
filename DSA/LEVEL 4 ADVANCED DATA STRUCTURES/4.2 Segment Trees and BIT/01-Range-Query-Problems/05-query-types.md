# Query Types (Sum, Min, Max)

## Chapter Overview

Different range query operations require different merge strategies. This chapter explores the three most common query types — **Sum**, **Min**, and **Max** — and the mathematical properties that make them compatible with tree-based structures.

---

## The Merge Function

Every range query structure works by **splitting** a range into precomputed segments and **merging** partial results.

```
Query(2, 7):

     [────────────── full range ──────────────]
     [── segment A ──][── segment B ──][─ C ──]
     
Answer = merge(A, B, C)
```

The merge function determines what query type the structure supports.

---

## Range Sum Query (RSQ)

**Merge function**: `merge(a, b) = a + b`

```
Array: [3, 1, 4, 1, 5, 9, 2, 6]

           [0-7] = 31
          /          \
     [0-3] = 9     [4-7] = 22
      /    \        /     \
  [0-1]=4 [2-3]=5 [4-5]=14 [6-7]=8
   / \     / \     / \      / \
  3   1   4   1   5   9    2   6

Query sum(1, 5):
  = A[1] + sum(2,3) + sum(4,5)
  =  1   +    5     +   14
  = 20

Properties:
  ✓ Associative:  (a+b)+c = a+(b+c)
  ✓ Commutative:  a+b = b+a
  ✓ Identity:     a+0 = a  (identity element = 0)
  ✓ Invertible:   can subtract → prefix sum works too
```

**Applications**: Total sales, cumulative scores, area calculations.

---

## Range Minimum Query (RMQ)

**Merge function**: `merge(a, b) = min(a, b)`

```
Array: [3, 1, 4, 1, 5, 9, 2, 6]

           [0-7] = 1
          /          \
     [0-3] = 1     [4-7] = 2
      /    \        /     \
  [0-1]=1 [2-3]=1 [4-5]=5 [6-7]=2
   / \     / \     / \      / \
  3   1   4   1   5   9    2   6

Query min(2, 6):
  = min(min(2,3), min(4,5), A[6])
  = min(  1,       5,       2  )
  = 1

Properties:
  ✓ Associative:   min(min(a,b),c) = min(a,min(b,c))
  ✓ Commutative:   min(a,b) = min(b,a)
  ✓ Identity:      min(a, +∞) = a  (identity = +∞)
  ✗ NOT Invertible: can't "undo" a min → no prefix trick
```

**Note**: Since min is not invertible, we CANNOT use prefix sums. We need Segment Tree or Sparse Table.

**Applications**: Lowest temperature in period, cheapest price, shortest path segment.

---

## Range Maximum Query

**Merge function**: `merge(a, b) = max(a, b)`

```
Array: [3, 1, 4, 1, 5, 9, 2, 6]

           [0-7] = 9
          /          \
     [0-3] = 4     [4-7] = 9
      /    \        /     \
  [0-1]=3 [2-3]=4 [4-5]=9 [6-7]=6
   / \     / \     / \      / \
  3   1   4   1   5   9    2   6

Query max(0, 4):
  = max(max(0,3), A[4])
  = max(  4,      5  )
  = 5

Properties:
  ✓ Associative:   max(max(a,b),c) = max(a,max(b,c))
  ✓ Commutative:   max(a,b) = max(b,a)
  ✓ Identity:      max(a, -∞) = a  (identity = -∞)
  ✗ NOT Invertible: can't "undo" a max
```

**Applications**: Highest stock price, peak load, maximum altitude.

---

## Comparison of Query Types

```
┌──────────────┬───────────┬───────────┬───────────┐
│   Property   │    SUM    │    MIN    │    MAX    │
├──────────────┼───────────┼───────────┼───────────┤
│  merge(a,b)  │   a + b   │  min(a,b) │  max(a,b) │
│  Identity    │     0     │    +∞     │    -∞     │
│  Associative │    YES    │    YES    │    YES    │
│  Commutative │    YES    │    YES    │    YES    │
│  Invertible  │    YES    │    NO     │    NO     │
│  Prefix OK   │    YES    │    NO     │    NO     │
│  Sparse Tbl  │    NO*    │    YES    │    YES    │
│  Seg Tree    │    YES    │    YES    │    YES    │
│  BIT         │    YES    │  TRICKY  │  TRICKY  │
└──────────────┴───────────┴───────────┴───────────┘

* Sparse Table doesn't work for sum because sum is not idempotent
  (adding overlapping ranges double-counts)
```

---

## Idempotent vs Non-Idempotent

An operation is **idempotent** if applying it to overlapping segments gives the correct answer:

```
Idempotent: f(a, a) = a
  min(3, 3) = 3  ✓
  max(5, 5) = 5  ✓
  gcd(6, 6) = 6  ✓

Non-idempotent: f(a, a) ≠ a  (in general)
  3 + 3 = 6 ≠ 3  ✗
  3 × 3 = 9 ≠ 3  ✗
  3 ^ 3 = 0 ≠ 3  ✗
```

**Why it matters**:
```
Sparse Table allows overlapping ranges:
  query(L, R) = merge(block₁, block₂)
  where block₁ and block₂ may overlap

  For min: min(min(a,b,c), min(b,c,d)) = min(a,b,c,d)  ✓
  For sum: (a+b+c) + (b+c+d) ≠ a+b+c+d  ✗ (double counts b,c)
```

---

## Segment Tree Node Design

Each query type needs a different node structure:

```
// Sum query node
struct SumNode {
    sum: integer
    merge(left, right) → left.sum + right.sum
    identity → 0
}

// Min query node
struct MinNode {
    min_val: integer
    merge(left, right) → min(left.min_val, right.min_val)
    identity → +INFINITY
}

// Max query node
struct MaxNode {
    max_val: integer
    merge(left, right) → max(left.max_val, right.max_val)
    identity → -INFINITY
}

// Combined node (stores all three)
struct CombinedNode {
    sum: integer
    min_val: integer
    max_val: integer
    merge(left, right) →
        sum = left.sum + right.sum
        min_val = min(left.min_val, right.min_val)
        max_val = max(left.max_val, right.max_val)
}
```

---

## Step-by-Step Trace: Multi-Query

```
Array: [2, 5, 1, 8, 3]

Build tree (storing sum / min / max at each node):

              [0-4]
           sum=19, min=1, max=8
           /              \
      [0-2]              [3-4]
   sum=8, min=1, max=5  sum=11, min=3, max=8
     /       \            /       \
  [0-1]     [2]       [3]       [4]
 s=7,m=2   s=1,m=1   s=8,m=8   s=3,m=3
 M=5       M=1       M=8       M=3
  / \
[0] [1]
s=2 s=5
m=2 m=5
M=2 M=5

Query(1, 3):
  Sum: 5 + 1 + 8 = 14
  Min: min(5, 1, 8) = 1
  Max: max(5, 1, 8) = 8

Using the tree:
  Need [1] + [2] + [3]
  Sum = 5 + 1 + 8 = 14  ✓
  Min = min(5, 1, 8) = 1  ✓
  Max = max(5, 1, 8) = 8  ✓
```

---

## Summary Table

| Query Type | Merge Function | Identity | Invertible | Idempotent |
|-----------|---------------|:--------:|:----------:|:----------:|
| Sum | a + b | 0 | Yes | No |
| Min | min(a,b) | +∞ | No | Yes |
| Max | max(a,b) | -∞ | No | Yes |
| GCD | gcd(a,b) | 0 | No | Yes |
| XOR | a ⊕ b | 0 | Yes | No |
| AND | a & b | all 1s | No | Yes |
| OR | a \| b | 0 | No | Yes |
| Product | a × b | 1 | Yes* | No |

\* Product is invertible via division, but beware of zeros.

---

## Quick Revision Questions

1. Why can't we use prefix sums for range minimum queries? *(min is not invertible — we can't subtract a prefix min)*
2. What is the identity element for range maximum queries? *(-∞)*
3. What does "idempotent" mean and why does it matter? *(f(a,a)=a; idempotent ops allow overlapping ranges in Sparse Table)*
4. Can a single segment tree answer sum, min, and max simultaneously? *(Yes — store all three in each node and merge each separately)*
5. Why doesn't Sparse Table work for range sum? *(Sum is not idempotent; overlapping ranges would double-count elements)*
6. What is the merge function for range GCD queries? *(merge(a, b) = gcd(a, b))*

---

[← Previous: Static vs Dynamic Data](04-static-vs-dynamic-data.md) | [Next: Update Types →](06-update-types.md)
