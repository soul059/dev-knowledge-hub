# Types of Range Queries

## Chapter Overview

Range queries are operations that ask questions about a **contiguous subarray** of elements. Understanding the different types of range queries is the foundation for knowing when and why we need advanced data structures like Segment Trees and BITs.

---

## What is a Range Query?

Given an array `A[0..n-1]`, a range query asks: **"What is the answer for elements from index L to R?"**

```
Array A:  [ 3,  1,  4,  1,  5,  9,  2,  6 ]
Index:      0   1   2   3   4   5   6   7

Query(2, 5) → asks about A[2], A[3], A[4], A[5]
           → the subarray [ 4, 1, 5, 9 ]
```

---

## Categories of Range Queries

### 1. Aggregate Queries

These combine all elements in the range into a single value.

```
┌──────────────────────────────────────────────┐
│           AGGREGATE QUERY TYPES              │
├──────────────┬───────────────────────────────┤
│  Query Type  │  Example for [4, 1, 5, 9]    │
├──────────────┼───────────────────────────────┤
│  Sum         │  4 + 1 + 5 + 9 = 19          │
│  Min         │  min(4, 1, 5, 9) = 1         │
│  Max         │  max(4, 1, 5, 9) = 9         │
│  GCD         │  gcd(4, 1, 5, 9) = 1         │
│  XOR         │  4 ^ 1 ^ 5 ^ 9 = 11         │
│  Product     │  4 × 1 × 5 × 9 = 180        │
│  Count       │  count where > 3 → 3         │
└──────────────┴───────────────────────────────┘
```

### 2. Search Queries

These find specific elements or positions within a range.

- **Find k-th smallest** element in range [L, R]
- **Find first element** greater than X in range [L, R]
- **Count elements** less than X in range [L, R]

### 3. Update + Query (Dynamic)

The array changes over time, and queries must reflect current state.

```
Timeline:
  t=0: A = [3, 1, 4, 1, 5]
  t=1: Update A[2] = 7 → A = [3, 1, 7, 1, 5]
  t=2: Query sum(1, 3) → 1 + 7 + 1 = 9
  t=3: Update A[0] = 10 → A = [10, 1, 7, 1, 5]
  t=4: Query sum(0, 2) → 10 + 1 + 7 = 18
```

---

## The Core Problem Structure

Every range query problem has two dimensions:

```
                    ┌─────────────┐
                    │  OPERATION  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌────────┐  ┌─────────┐  ┌─────────┐
         │  SUM   │  │   MIN   │  │   MAX   │ ... etc
         └────────┘  └─────────┘  └─────────┘

                    ┌─────────────┐
                    │   ACCESS    │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌────────┐  ┌─────────┐  ┌─────────┐
         │ STATIC │  │  POINT  │  │  RANGE  │
         │  (no   │  │ UPDATE  │  │ UPDATE  │
         │ update)│  └─────────┘  └─────────┘
         └────────┘
```

---

## Formal Definition

Given array `A[0..n-1]`:

- **Range Query(L, R)**: Compute `f(A[L], A[L+1], ..., A[R])` where `f` is an associative function
- **Point Update(i, val)**: Set `A[i] = val` (or `A[i] += val`)
- **Range Update(L, R, val)**: For all `i` in `[L, R]`, set `A[i] = val` (or `A[i] += val`)

**Key insight**: The function `f` must be **associative** — meaning `f(f(a,b), c) = f(a, f(b,c))`. This allows us to split the range and combine partial results.

```
Associative functions:  +, ×, min, max, gcd, xor, AND, OR
Non-associative:        -, ÷, median (cannot split and merge easily)
```

---

## Real-World Examples

| Domain | Query Type | Example |
|--------|-----------|---------|
| Finance | Range Sum | Total sales from day L to day R |
| Weather | Range Min/Max | Coldest/hottest day in a period |
| Gaming | Range Update | Apply damage to all units in zone |
| Database | Count in Range | Records with timestamp between L and R |
| Image Processing | 2D Range Sum | Sum of pixel values in a rectangle |

---

## Summary Table

| Aspect | Description |
|--------|-------------|
| Range Query | Operation on a contiguous subarray A[L..R] |
| Aggregate | Combines elements into one value (sum, min, max, etc.) |
| Associativity | Required property for divide-and-conquer approach |
| Static | Array does not change; precompute answers |
| Dynamic | Array changes; need efficient update + query |
| Two Dimensions | What operation? + How does data change? |

---

## Quick Revision Questions

1. What makes a function suitable for range queries on a segment tree? *(It must be associative)*
2. Give three examples of aggregate range queries. *(Sum, Min, Max / GCD / XOR)*
3. What is the difference between a point update and a range update? *(Point changes one element; range changes a subarray)*
4. Why can't we use median as a merge function in a segment tree? *(It's not associative — can't combine partial medians)*
5. Name a real-world scenario that requires dynamic range queries. *(Stock portfolio — prices update, need range sum/min)*

---

[Next: Prefix Sum Limitations →](02-prefix-sum-limitations.md)
