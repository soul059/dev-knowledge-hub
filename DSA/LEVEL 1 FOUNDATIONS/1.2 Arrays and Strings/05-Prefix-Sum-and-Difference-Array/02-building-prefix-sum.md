# Chapter 2: Building Prefix Sum Array

[← Previous: Prefix Sum Concept](01-prefix-sum-concept.md) | [Back to README](../README.md) | [Next: Range Sum Queries →](03-range-sum-queries.md)

---

## Overview

This chapter focuses on the **construction** of prefix sum arrays — both 0-indexed and 1-indexed versions, in-place vs auxiliary array approaches, and extensions to 2D arrays.

---

## Building a 1D Prefix Sum Array

### 0-Indexed Version

```
FUNCTION buildPrefix(arr, n):
    prefix = new array of size n
    prefix[0] = arr[0]

    FOR i = 1 TO n-1:
        prefix[i] = prefix[i-1] + arr[i]

    RETURN prefix
```

### Trace

```
  arr = [5, 2, 8, 3, 7]

  Step 0: prefix[0] = 5
  Step 1: prefix[1] = 5 + 2 = 7
  Step 2: prefix[2] = 7 + 8 = 15
  Step 3: prefix[3] = 15 + 3 = 18
  Step 4: prefix[4] = 18 + 7 = 25

  arr:    [5,  2,  8,  3,  7]
  prefix: [5,  7, 15, 18, 25]
           │      │          │
           │      │          └─ sum of all: 5+2+8+3+7=25
           │      └─ sum of first 3: 5+2+8=15
           └─ just first element: 5
```

---

## 1-Indexed Version (Recommended)

Adding a dummy 0 at the start eliminates edge cases.

```
FUNCTION buildPrefix1Indexed(arr, n):
    prefix = new array of size n+1
    prefix[0] = 0    // dummy

    FOR i = 1 TO n:
        prefix[i] = prefix[i-1] + arr[i-1]

    RETURN prefix
```

### Trace

```
  arr = [5, 2, 8, 3, 7]    (0-indexed, size 5)

  prefix[0] = 0           (dummy)
  prefix[1] = 0 + arr[0] = 5
  prefix[2] = 5 + arr[1] = 7
  prefix[3] = 7 + arr[2] = 15
  prefix[4] = 15 + arr[3] = 18
  prefix[5] = 18 + arr[4] = 25

  prefix: [0, 5, 7, 15, 18, 25]
           ↑
           dummy zero

  Range sum formula (original 0-indexed l, r):
  sum(l, r) = prefix[r+1] - prefix[l]

  Example: sum(1, 3) = prefix[4] - prefix[1] = 18 - 5 = 13
  Verify:  arr[1]+arr[2]+arr[3] = 2+8+3 = 13  ✓
```

### Advantage of 1-Indexed

```
  0-INDEXED:
    sum(l, r) = prefix[r] - (l > 0 ? prefix[l-1] : 0)
    → Need special case for l = 0

  1-INDEXED:
    sum(l, r) = prefix[r+1] - prefix[l]
    → Works uniformly, no special cases!

  ┌──────────────────────────────────────┐
  │  The dummy zero absorbs the l=0     │
  │  edge case automatically.           │
  └──────────────────────────────────────┘
```

---

## In-Place Prefix Sum

If you don't need the original array, convert it in place to save space.

```
FUNCTION buildPrefixInPlace(arr, n):
    FOR i = 1 TO n-1:
        arr[i] = arr[i] + arr[i-1]
    // arr is now the prefix sum array
```

```
  Before: [5,  2,  8,  3,  7]
  After:  [5,  7, 15, 18, 25]

  Space: O(1) extra  (but original array is destroyed!)
```

---

## Building Suffix Sum Array

```
FUNCTION buildSuffix(arr, n):
    suffix = new array of size n
    suffix[n-1] = arr[n-1]

    FOR i = n-2 DOWN TO 0:
        suffix[i] = suffix[i+1] + arr[i]

    RETURN suffix
```

### Trace

```
  arr = [5, 2, 8, 3, 7]

  suffix[4] = 7
  suffix[3] = 7 + 3 = 10
  suffix[2] = 10 + 8 = 18
  suffix[1] = 18 + 2 = 20
  suffix[0] = 20 + 5 = 25

  arr:    [5,  2,  8,  3,  7]
  suffix: [25, 20, 18, 10, 7]
           │             │  │
           │             │  └─ just last element: 7
           │             └─ sum of last 2: 3+7=10
           └─ sum of all: 25
```

---

## Prefix + Suffix Together

Many problems need both prefix and suffix sums. Together they allow splitting the array at any index.

```
  arr:    [5,  2,  8,  3,  7]
  prefix: [5,  7, 15, 18, 25]
  suffix: [25, 20, 18, 10,  7]

  At index i, you can calculate:
  • Sum of left part  (0..i-1) = prefix[i-1]  (or 0 if i=0)
  • Sum of right part (i+1..n-1) = suffix[i+1] (or 0 if i=n-1)
  • Current element = arr[i]

  Application: Find equilibrium index where
  left sum == right sum

    Index 0: left=0,     right=20   ✗
    Index 1: left=5,     right=18   ✗
    Index 2: left=7,     right=10   ✗
    Index 3: left=15,    right=7    ✗
    Index 4: left=18,    right=0    ✗
    → No equilibrium index in this array
```

---

## 2D Prefix Sum (Summed Area Table)

For a 2D matrix, the prefix sum at (r, c) stores the sum of all elements in the rectangle from (0,0) to (r,c).

```
FUNCTION build2DPrefix(matrix, rows, cols):
    prefix = new 2D array of size (rows+1) × (cols+1), filled with 0

    FOR r = 1 TO rows:
        FOR c = 1 TO cols:
            prefix[r][c] = matrix[r-1][c-1]
                         + prefix[r-1][c]
                         + prefix[r][c-1]
                         - prefix[r-1][c-1]

    RETURN prefix
```

### Inclusion-Exclusion Principle

```
  To compute prefix[r][c]:

  ┌───────────┬───────┐
  │           │       │
  │     A     │   B   │
  │           │       │
  ├───────────┼───────┤
  │     C     │ (r,c) │
  └───────────┴───────┘

  prefix[r][c] = matrix[r][c]
               + prefix[r-1][c]    (A + B)
               + prefix[r][c-1]    (A + C)
               - prefix[r-1][c-1]  (A — subtracted twice!)

  Visual:
  ┌─────────┐     ┌─────────┐     ┌─────────┐
  │/////////│     │         │     │/////////│
  │/////////│  +  │/////////│  -  │/////////│
  └─────────┘     └─────────┘     └─────────┘
  prefix[r-1][c]  prefix[r][c-1]  prefix[r-1][c-1]
  (top portion)   (left portion)  (double counted)
```

### Trace: 2D Prefix Sum

```
  matrix:        prefix (1-indexed with dummy row/col of 0):
  [1, 2, 3]     [0, 0, 0, 0]
  [4, 5, 6]     [0, 1, 3, 6]
  [7, 8, 9]     [0, 5, 12, 21]
                [0, 12, 27, 45]

  prefix[2][2] = matrix[1][1] + prefix[1][2] + prefix[2][1] - prefix[1][1]
               = 5 + 3 + 5 - 1 = 12
  Verify: 1+2+4+5 = 12  ✓

  prefix[3][3] = matrix[2][2] + prefix[2][3] + prefix[3][2] - prefix[2][2]
               = 9 + 21 + 27 - 12 = 45
  Verify: 1+2+3+4+5+6+7+8+9 = 45  ✓
```

---

## 2D Range Sum Query

```
  Sum of rectangle from (r1,c1) to (r2,c2):

  sum = prefix[r2+1][c2+1]
      - prefix[r1][c2+1]
      - prefix[r2+1][c1]
      + prefix[r1][c1]

  Using inclusion-exclusion:

  ┌───────────────────────┐
  │         r1            │
  │    ┌────┼────────┐    │
  │ c1 │    │ QUERY  │ c2 │
  │    │    │ AREA   │    │
  │    └────┼────────┘    │
  │         r2            │
  └───────────────────────┘

  Total    - Top     - Left    + TopLeft (double subtracted)
```

### Example Query

```
  prefix:
  [0,  0,  0,  0]
  [0,  1,  3,  6]
  [0,  5, 12, 21]
  [0, 12, 27, 45]

  Query: sum from (1,1) to (2,2)  (0-indexed in original)

  sum = prefix[3][3] - prefix[1][3] - prefix[3][1] + prefix[1][1]
      = 45 - 6 - 12 + 1 = 28

  Verify:  matrix[1][1] + matrix[1][2] + matrix[2][1] + matrix[2][2]
         = 5 + 6 + 8 + 9 = 28  ✓
```

---

## Building Prefix XOR

```
FUNCTION buildPrefixXOR(arr, n):
    prefix = new array of size n+1
    prefix[0] = 0

    FOR i = 1 TO n:
        prefix[i] = prefix[i-1] XOR arr[i-1]

    RETURN prefix

// Range XOR(l, r) = prefix[r+1] XOR prefix[l]
// Works because XOR is its own inverse: a XOR a = 0
```

---

## Building Prefix Product

```
FUNCTION buildPrefixProduct(arr, n):
    prefix = new array of size n
    prefix[0] = arr[0]

    FOR i = 1 TO n-1:
        prefix[i] = prefix[i-1] × arr[i]

    RETURN prefix

// Range product(l, r) = prefix[r] / prefix[l-1]
// WARNING: Division may not be exact for integers.
//          Also fails if any element is 0!
```

---

## Complexity Summary

| Variant | Build Time | Query Time | Space |
|---------|-----------|------------|-------|
| 1D Prefix Sum | O(n) | O(1) | O(n) |
| 1D In-Place | O(n) | O(1) | O(1) extra |
| 1D Suffix Sum | O(n) | O(1) | O(n) |
| 2D Prefix Sum | O(r × c) | O(1) | O(r × c) |
| Prefix XOR | O(n) | O(1) | O(n) |
| Prefix Product | O(n) | O(1) | O(n) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| 0-indexed formula | prefix[i] = prefix[i-1] + arr[i], prefix[0] = arr[0] |
| 1-indexed formula | prefix[i] = prefix[i-1] + arr[i-1], prefix[0] = 0 |
| In-place | arr[i] += arr[i-1] |
| Suffix | suffix[i] = suffix[i+1] + arr[i] |
| 2D build | Inclusion-exclusion with 4 terms |
| 2D query | Inclusion-exclusion with 4 prefix values |

---

## Quick Revision Questions

1. **What is the advantage** of using a 1-indexed prefix sum array with a dummy zero?

2. **Build the prefix sum array** for arr = [1, -2, 3, -4, 5]. Then compute sum(1, 3).

3. **How does the 2D prefix sum formula work?** Why do we subtract `prefix[r-1][c-1]`?

4. **Can you build prefix products safely** when the array contains zeros? Why or why not?

5. **What's the difference between prefix sum and suffix sum?** When would you use both together?

6. **Write the formula** for a 2D range sum query from (r1,c1) to (r2,c2).

---

[← Previous: Prefix Sum Concept](01-prefix-sum-concept.md) | [Back to README](../README.md) | [Next: Range Sum Queries →](03-range-sum-queries.md)
