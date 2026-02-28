# Chapter 6: Product of Array Except Self

[← Previous: Subarray Sum Equals K](05-subarray-sum-equals-k.md) | [Back to README](../README.md) | [Next Unit: Kadane's Algorithm →](../06-Kadanes-Algorithm/01-maximum-subarray-problem.md)

---

## Overview

**Problem (LeetCode 238):** Given an integer array `nums`, return an array `answer` where `answer[i]` is the product of all elements of `nums` except `nums[i]`.

**Constraint:** You must solve it in O(n) time **without using division**.

This problem is a beautiful application of prefix and suffix products.

---

## Problem Understanding

```
  nums = [1, 2, 3, 4]

  answer[0] = 2 × 3 × 4 = 24
  answer[1] = 1 × 3 × 4 = 12
  answer[2] = 1 × 2 × 4 = 8
  answer[3] = 1 × 2 × 3 = 6

  Output: [24, 12, 8, 6]
```

---

## Why No Division?

```
  OBVIOUS approach: total = product of all → answer[i] = total / nums[i]
  
  Problems:
  1. Division by zero if nums[i] = 0
  2. Problem explicitly forbids it
  3. Integer division may not give exact result

  nums = [1, 2, 0, 4]
  total = 0
  answer[0] = 0/1 = 0?   But actual = 2×0×4 = 0  ✓
  answer[2] = 0/0 = ???   UNDEFINED!               ✗
```

---

## Approach 1: Brute Force — O(n²)

```
FUNCTION productExceptSelf(nums, n):
    answer = new array of size n

    FOR i = 0 TO n-1:
        product = 1
        FOR j = 0 TO n-1:
            IF j ≠ i:
                product = product × nums[j]
        answer[i] = product

    RETURN answer
```

---

## Approach 2: Prefix and Suffix Products — O(n)

### Key Insight

```
  answer[i] = (product of all elements LEFT of i)
            × (product of all elements RIGHT of i)

  answer[i] = prefix_product[i] × suffix_product[i]

  nums:   [1,   2,   3,   4]
  
  Left products (prefix):
  L[0] = 1           (nothing to the left)
  L[1] = 1           (only nums[0])
  L[2] = 1×2 = 2
  L[3] = 1×2×3 = 6

  Right products (suffix):
  R[3] = 1           (nothing to the right)
  R[2] = 4
  R[1] = 3×4 = 12
  R[0] = 2×3×4 = 24

  answer:
  i=0: L[0] × R[0] = 1 × 24 = 24  ✓
  i=1: L[1] × R[1] = 1 × 12 = 12  ✓
  i=2: L[2] × R[2] = 2 × 4 = 8    ✓
  i=3: L[3] × R[3] = 6 × 1 = 6    ✓
```

### Visual

```
  For answer[2] (skip nums[2]=3):

  nums:  [1,    2,    3,    4]
          ├────┤      ├────┤
          left         right
          product      product
          = 1×2        = 4
          = 2          = 4

  answer[2] = 2 × 4 = 8  ✓
```

---

## Algorithm (Two Arrays)

```
FUNCTION productExceptSelf(nums, n):
    left = new array of size n
    right = new array of size n
    answer = new array of size n

    // Build left products
    left[0] = 1
    FOR i = 1 TO n-1:
        left[i] = left[i-1] × nums[i-1]

    // Build right products
    right[n-1] = 1
    FOR i = n-2 DOWN TO 0:
        right[i] = right[i+1] × nums[i+1]

    // Combine
    FOR i = 0 TO n-1:
        answer[i] = left[i] × right[i]

    RETURN answer
```

---

## Detailed Trace: nums = [2, 3, 4, 5]

```
  Build left products (product of everything BEFORE i):
  left[0] = 1                  (nothing before index 0)
  left[1] = 1 × nums[0] = 2
  left[2] = 2 × nums[1] = 6
  left[3] = 6 × nums[2] = 24

  left = [1, 2, 6, 24]

  Build right products (product of everything AFTER i):
  right[3] = 1                  (nothing after index 3)
  right[2] = 1 × nums[3] = 5
  right[1] = 5 × nums[2] = 20
  right[0] = 20 × nums[1] = 60

  right = [60, 20, 5, 1]

  Combine:
  answer[0] = left[0] × right[0] = 1 × 60 = 60
  answer[1] = left[1] × right[1] = 2 × 20 = 40
  answer[2] = left[2] × right[2] = 6 × 5 = 30
  answer[3] = left[3] × right[3] = 24 × 1 = 24

  answer = [60, 40, 30, 24]

  Verify: total product = 2×3×4×5 = 120
  120/2=60 ✓, 120/3=40 ✓, 120/4=30 ✓, 120/5=24 ✓
```

---

## Approach 3: O(1) Extra Space

Use the answer array itself for left products, then do a backward pass multiplying by right.

```
FUNCTION productExceptSelf(nums, n):
    answer = new array of size n

    // Forward pass: answer[i] = product of all left of i
    answer[0] = 1
    FOR i = 1 TO n-1:
        answer[i] = answer[i-1] × nums[i-1]

    // Backward pass: multiply by right products
    rightProduct = 1
    FOR i = n-1 DOWN TO 0:
        answer[i] = answer[i] × rightProduct
        rightProduct = rightProduct × nums[i]

    RETURN answer
```

### Trace: nums = [1, 2, 3, 4]

```
  Forward pass (left products):
  answer = [1, 1, 2, 6]

  Backward pass:
  i=3: answer[3] = 6 × 1 = 6,    rightProduct = 1 × 4 = 4
  i=2: answer[2] = 2 × 4 = 8,    rightProduct = 4 × 3 = 12
  i=1: answer[1] = 1 × 12 = 12,  rightProduct = 12 × 2 = 24
  i=0: answer[0] = 1 × 24 = 24,  rightProduct = 24 × 1 = 24

  answer = [24, 12, 8, 6]  ✓

  Space: O(1) extra (answer array doesn't count)
```

---

## Handling Zeros

```
  nums = [1, 2, 0, 4]

  Forward: answer = [1, 1, 2, 0]
  
  Backward:
  i=3: answer[3] = 0 × 1 = 0,    rightProduct = 4
  i=2: answer[2] = 2 × 4 = 8,    rightProduct = 0
  i=1: answer[1] = 1 × 0 = 0,    rightProduct = 0
  i=0: answer[0] = 1 × 0 = 0,    rightProduct = 0

  answer = [0, 0, 8, 0]

  Verify:
  answer[0] = 2×0×4 = 0   ✓
  answer[1] = 1×0×4 = 0   ✓
  answer[2] = 1×2×4 = 8   ✓ (the only non-zero!)
  answer[3] = 1×2×0 = 0   ✓
```

---

## Edge Case: Two Zeros

```
  nums = [0, 2, 0, 4]
  
  Every answer[i] will have at least one zero in its product.
  All answers are 0.

  answer = [0, 0, 0, 0]
```

---

## Connection to Prefix/Suffix Pattern

```
  ┌──────────────────────────────────────────────────┐
  │  This is the PREFIX + SUFFIX pattern:            │
  │                                                  │
  │  answer[i] = f(left of i) ⊗ f(right of i)       │
  │                                                  │
  │  Where:                                          │
  │  f = product, ⊗ = multiplication                 │
  │                                                  │
  │  Same pattern applies to:                        │
  │  • Product except self (this problem)            │
  │  • Trapping rain water (min of prefix max,       │
  │    suffix max)                                   │
  │  • Stock problems (prefix min, suffix max)       │
  └──────────────────────────────────────────────────┘
```

---

## Alternative: With Division (If Allowed)

```
FUNCTION productExceptSelfDiv(nums, n):
    zeroCount = 0
    product = 1

    FOR i = 0 TO n-1:
        IF nums[i] == 0:
            zeroCount++
        ELSE:
            product = product × nums[i]

    answer = new array of size n, all zeros

    IF zeroCount > 1:
        RETURN answer          // all zeros
    ELSE IF zeroCount == 1:
        FOR i = 0 TO n-1:
            IF nums[i] == 0:
                answer[i] = product
        RETURN answer          // only the zero position gets product
    ELSE:
        FOR i = 0 TO n-1:
            answer[i] = product / nums[i]
        RETURN answer
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force | O(n²) | O(1) extra |
| Two arrays (left + right) | O(n) | O(n) |
| Single array + running product | O(n) | O(1) extra |
| Division approach | O(n) | O(1) extra |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Key insight | answer[i] = left_product × right_product |
| Left product | Product of nums[0..i-1] |
| Right product | Product of nums[i+1..n-1] |
| Optimal time | O(n) — two passes |
| Optimal space | O(1) extra — use output array + running variable |
| Handles zeros | Yes, naturally |
| No division needed | Yes |

---

## Quick Revision Questions

1. **Why can't we simply divide the total product** by each element? What are two reasons?

2. **What does `left[i]` represent** in the prefix product approach?

3. **How does the O(1) space optimization work?** What two passes does it make?

4. **What happens when the array contains exactly one zero?** What about two zeros?

5. **Trace through nums = [5, 3, 1, 4, 2]** using the O(1) space approach.

6. **What other problems** use the same prefix + suffix product/sum pattern?

---

[← Previous: Subarray Sum Equals K](05-subarray-sum-equals-k.md) | [Back to README](../README.md) | [Next Unit: Kadane's Algorithm →](../06-Kadanes-Algorithm/01-maximum-subarray-problem.md)
