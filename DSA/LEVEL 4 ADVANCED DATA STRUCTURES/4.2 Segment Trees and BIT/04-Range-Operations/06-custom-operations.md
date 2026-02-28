# Custom Operations

## Chapter Overview

Any associative binary operation with an identity element can power a segment tree. This chapter shows how to design custom merge functions for diverse problems — multi-value nodes, composite operations, and unconventional queries.

---

## Requirements Checklist

```
To use an operation in a segment tree, verify:

  ✓ ASSOCIATIVE:   f(f(a,b), c) = f(a, f(b,c))
    → Required. Segment tree relies on combining sub-results.
  
  ✓ IDENTITY element e:   f(a, e) = f(e, a) = a
    → Required for "no overlap" return value.
  
  ○ COMMUTATIVE (optional):
    → Not required, but simplifies implementation.
  
  ○ INVERTIBLE (optional):
    → Enables prefix-based approaches and BIT.
  
  ○ IDEMPOTENT (optional):
    → Enables Sparse Table O(1) queries.
```

---

## Recipe: Designing a Custom Node

```
Step 1: Define what information EACH ELEMENT contributes
Step 2: Define what information EACH NODE stores
Step 3: Design merge(left_child, right_child)
Step 4: Verify associativity
Step 5: Find identity element

Example: Count of elements > threshold in range

Step 1: Each element either is or isn't > threshold
    → contributes 1 or 0

Step 2: Node stores count of elements > threshold
    → but threshold might change per query!
    → Need different approach (merge sort tree or offline)

Example: Maximum prefix sum in range

Step 1: Each element A[i] contributes to prefix sums
Step 2: Node stores: total_sum, max_prefix_sum
Step 3: merge(L, R):
    total = L.total + R.total
    max_prefix = max(L.max_prefix, L.total + R.max_prefix)
Step 4: Associativity: ✓ (verified by expansion)
Step 5: Identity: (total=0, max_prefix=-∞)
```

---

## Classic Custom: Maximum Subarray Sum (Kadane on Segments)

```
Problem: Find maximum subarray sum in range [L, R] with updates.

What a node needs to store:
┌────────────────────────────────────────┐
│  For segment [l, r]:                   │
│    total:   sum of all elements        │
│    prefix:  max sum starting from l    │
│    suffix:  max sum ending at r        │
│    answer:  max subarray sum anywhere  │
└────────────────────────────────────────┘

Why each field:
  - The answer might be entirely in left child
  - Or entirely in right child
  - Or it SPANS the boundary (suffix of left + prefix of right)

Merge function:
  function merge(L, R):
      node.total  = L.total + R.total
      node.prefix = max(L.prefix, L.total + R.prefix)
      node.suffix = max(R.suffix, R.total + L.suffix)
      node.answer = max(L.answer, R.answer, L.suffix + R.prefix)
      return node

Identity: total=0, prefix=-∞, suffix=-∞, answer=-∞

Leaf(val): total=val, prefix=val, suffix=val, answer=val

Trace:
  A = [2, -1, 3, -2]

  Leaf 0: (2, 2, 2, 2)
  Leaf 1: (-1, -1, -1, -1)
  Leaf 2: (3, 3, 3, 3)
  Leaf 3: (-2, -2, -2, -2)

  Node [0-1]: merge((2,2,2,2), (-1,-1,-1,-1))
    total  = 2 + (-1) = 1
    prefix = max(2, 2+(-1)) = max(2, 1) = 2
    suffix = max(-1, (-1)+2) = max(-1, 1) = 1
    answer = max(2, -1, 2+(-1)) = max(2, -1, 1) = 2
    → (1, 2, 1, 2)

  Node [2-3]: merge((3,3,3,3), (-2,-2,-2,-2))
    total  = 3 + (-2) = 1
    prefix = max(3, 3+(-2)) = 3
    suffix = max(-2, (-2)+3) = 1
    answer = max(3, -2, 3+(-2)) = 3
    → (1, 3, 1, 3)

  Node [0-3]: merge((1,2,1,2), (1,3,1,3))
    total  = 1 + 1 = 2
    prefix = max(2, 1+3) = 4
    suffix = max(1, 1+1) = 2
    answer = max(2, 3, 1+3) = 4
    → (2, 4, 2, 4)

  Max subarray = 4  (subarray [2,-1,3] = 4)  ✓
```

---

## Classic Custom: Count + Sum in Range

```
Problem: How many elements in [L,R]? What's their sum?

struct Node:
    count: integer
    sum:   integer

merge(L, R): Node(L.count + R.count, L.sum + R.sum)
identity:    Node(0, 0)
leaf(val):   Node(1, val)

Use case: Weighted queries, average = sum / count
```

---

## Classic Custom: Matrix Multiplication

```
Problem: Product of 2×2 matrices in range [L,R]

Each element A[i] is a 2×2 matrix.
merge(L, R) = L × R   (matrix multiplication)

Matrix multiplication is:
  ✓ Associative
  ✗ Not commutative (ORDER MATTERS!)
  Identity = I (identity matrix)

Application: Computing Fibonacci(n) in O(log n):
  F(n) = [[1,1],[1,0]]^n

  Range product of matrix arrays → also O(log n) per query

Caution: Since NOT commutative, query must combine left-to-right!
  query = merge(left_result, right_result)  // order matters
  NOT merge(right_result, left_result)

This naturally works in the standard segment tree because
the recursive structure preserves order: left child before right.
```

---

## Classic Custom: Count of Specific Value

```
Problem: Count occurrences of value V in range [L,R]

Node stores: count of V in range
merge(L, R): L.count + R.count
leaf(val):   1 if val == V, else 0

Extension — count of ALL distinct values:
  This doesn't fit a simple merge!
  Need Mo's algorithm or merge sort tree instead.
```

---

## Design Patterns Summary

```
Pattern 1: SINGLE VALUE NODE
  Merge is simple binary operation
  Examples: sum, min, max, gcd, xor, product

Pattern 2: MULTI-VALUE NODE
  Multiple fields needed for correct merge
  Examples: 
    - (sum, count) for average
    - (total, prefix, suffix, answer) for max subarray
    - (max1, max2) for second maximum
    - (min, max) for range spread

Pattern 3: OBJECT NODE
  Node stores a complex structure
  Examples:
    - 2×2 matrix for linear recurrence
    - Sorted list for merge sort tree (query time O(log²n))
    - Set of values for distinct counting

Rule of thumb: If merge needs information from BOTH children
that isn't captured by a single value, add more fields.
```

---

## Verifying Associativity

```
For simple operations, associativity is obvious.
For complex merges, verify with 3 segment example:

  A = [a₁, a₂, a₃]
  
  merge(merge(A₁, A₂), A₃) should equal
  merge(A₁, merge(A₂, A₃))

  where Aᵢ is the leaf node for element i.

For max subarray sum:
  Let L1 = leaf(a₁), L2 = leaf(a₂), L3 = leaf(a₃)
  
  Path 1: merge(merge(L1, L2), L3)
  Path 2: merge(L1, merge(L2, L3))
  
  Both should give same (total, prefix, suffix, answer).
  
  This works because each field's formula only depends on
  the same fields from children, and those aggregate correctly.
```

---

## Common Pitfalls

```
1. FORGETTING A FIELD
   Max subarray without suffix → boundary-spanning subarrays missed

2. WRONG IDENTITY
   Using 0 as identity for min → queries return 0 instead of actual min

3. ASSUMING COMMUTATIVITY
   Matrix merge(L, R) ≠ merge(R, L) → order matters in query

4. NON-ASSOCIATIVE OPERATION
   Subtraction: (a - b) - c ≠ a - (b - c) → can't use segment tree!
   Median: median(median(a,b), c) ≠ correct median → can't use!

5. LAZY PROPAGATION MISMATCH
   Custom nodes with lazy updates need careful compose logic
```

---

## Summary Table

| Custom Operation | Fields Stored | Merge Complexity | Identity |
|-----------------|---------------|-----------------|----------|
| Sum | sum | O(1) | 0 |
| Min/Max | value | O(1) | ±∞ |
| GCD | value | O(log V) | 0 |
| Max subarray | total, prefix, suffix, answer | O(1) | (0,-∞,-∞,-∞) |
| Count + Sum | count, sum | O(1) | (0, 0) |
| Matrix product | 2×2 matrix | O(k³) for k×k | Identity matrix |
| Min + Max (spread) | min, max | O(1) | (+∞, -∞) |

---

## Quick Revision Questions

1. What two properties must every segment tree operation have? *(Associativity and an identity element)*
2. Why does max subarray sum need 4 fields? *(The answer might span the boundary between children, requiring prefix/suffix/total to compute)*
3. Can non-commutative operations be used in a segment tree? *(Yes — matrix multiplication works because recursive structure preserves order)*
4. Can subtraction be used as a merge function? *(No — subtraction is not associative: (a-b)-c ≠ a-(b-c))*
5. How do you verify a custom merge is correct? *(Check associativity with 3 elements; verify identity element; test boundary cases)*

---

[← Previous: Range XOR Queries](05-range-xor-queries.md) | [Back to README →](../README.md)
