# Chapter 5.4: Find the Missing Number

> **Unit 5 — XOR Applications**
> Given n numbers from 0 to n with one missing, find it using XOR.

---

## Overview

Given an array of n distinct numbers taken from 0, 1, 2, ..., n, find the one number missing from the array. XOR provides an elegant O(n) time, O(1) space solution.

---

## The XOR Approach

```
  ┌────────────────────────────────────────────────────────────┐
  │  XOR all array elements with all numbers 0..n.             │
  │  Every number that EXISTS appears twice → cancels to 0.    │
  │  The MISSING number appears once → survives!               │
  └────────────────────────────────────────────────────────────┘

  missing = (0 ^ 1 ^ 2 ^ ... ^ n) ^ (arr[0] ^ arr[1] ^ ... ^ arr[n-1])
```

---

## Step-by-Step Trace

### Example: arr = [3, 0, 1], n = 3

```
  Expected:  0, 1, 2, 3
  Present:   3, 0, 1
  Missing:   2

  XOR of 0..3:      0 ^ 1 ^ 2 ^ 3
  XOR of array:     3 ^ 0 ^ 1

  Combined:
  (0 ^ 1 ^ 2 ^ 3) ^ (3 ^ 0 ^ 1)
  = (0^0) ^ (1^1) ^ (3^3) ^ 2
  =   0   ^   0   ^   0   ^ 2
  = 2  ✓
```

---

## Pseudocode

```
FUNCTION findMissing(arr[], n):
    result = n    // start with n (the largest expected number)
    FOR i = 0 TO n-1:
        result = result ^ i ^ arr[i]
    RETURN result
```

### Why start with n?

```
  The loop runs i = 0..n-1, giving us XOR of 0,1,...,n-1.
  By initializing result = n, we effectively XOR all of 0..n.
  Simultaneously, we XOR all array elements.
  
  This is a single-pass solution!
```

---

## Alternative: Sum Formula

```
  expected_sum = n × (n + 1) / 2
  actual_sum = sum of array
  missing = expected_sum - actual_sum
```

### Comparison

| Method | Time | Space | Overflow Risk |
|--------|------|-------|---------------|
| **XOR** | O(n) | O(1) | **None** |
| Sum formula | O(n) | O(1) | Yes, for large n |
| Sorting | O(n log n) | O(1) | None |
| Hash set | O(n) | O(n) | None |

> XOR wins: same efficiency as sum formula but **no overflow risk**.

---

## XOR of a Range [0..n] Quick Formula

```
  XOR from 0 to n follows a pattern based on n % 4:

  n % 4 == 0  →  n
  n % 4 == 1  →  1
  n % 4 == 2  →  n + 1
  n % 4 == 3  →  0

  Proof:
  n=0: 0                         = 0   (0%4=0 → 0 ✓)
  n=1: 0^1                       = 1   (1%4=1 → 1 ✓)
  n=2: 0^1^2                     = 3   (2%4=2 → 2+1=3 ✓)
  n=3: 0^1^2^3                   = 0   (3%4=3 → 0 ✓)
  n=4: 0^1^2^3^4                 = 4   (4%4=0 → 4 ✓)
  n=5: 0^1^2^3^4^5               = 1   (5%4=1 → 1 ✓)
  n=6: 0^1^2^3^4^5^6             = 7   (6%4=2 → 6+1=7 ✓)
  n=7: 0^1^2^3^4^5^6^7           = 0   (7%4=3 → 0 ✓)

  This allows O(1) computation of XOR(0..n)!
```

---

## Using the Range Formula

```
FUNCTION findMissing_fast(arr[], n):
    xor_range = xorUpTo(n)           // O(1) using n%4 formula
    xor_arr = 0
    FOR each x in arr:
        xor_arr = xor_arr ^ x       // O(n)
    RETURN xor_range ^ xor_arr

FUNCTION xorUpTo(n):
    SWITCH (n % 4):
        CASE 0: RETURN n
        CASE 1: RETURN 1
        CASE 2: RETURN n + 1
        CASE 3: RETURN 0
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core idea | XOR expected range with actual array; missing survives |
| Single-pass trick | Init result = n, XOR both i and arr[i] in same loop |
| No overflow | XOR can't overflow unlike sum approach |
| Range XOR formula | n%4 pattern gives XOR(0..n) in O(1) |

---

## Quick Revision Questions

1. **Find the missing number in [0, 1, 3, 4, 5], n = 5.**
   <details><summary>Answer</summary>XOR(0..5) = 1 (5%4=1). XOR(array) = 0^1^3^4^5 = 3. Missing = 1^3 = 2.</details>

2. **Why doesn't the sum formula work well for n = 10^9?**
   <details><summary>Answer</summary>Sum = n(n+1)/2 ≈ 5×10^17, which overflows 32-bit integers. XOR stays within the value range of the elements.</details>

3. **What is XOR(0..100) using the quick formula?**
   <details><summary>Answer</summary>100 % 4 = 0, so XOR(0..100) = 100.</details>

4. **Can this find two missing numbers?**
   <details><summary>Answer</summary>Not with XOR alone — you'd get a^b. You'd need the technique from Chapter 5.3 (partition by differentiating bit) or use sum + sum-of-squares equations.</details>

5. **What if the range starts at 1 instead of 0?**
   <details><summary>Answer</summary>XOR of 1..n = XOR(0..n) ^ 0 = XOR(0..n). Starting from 0 or 1 gives the same result since 0 is the XOR identity.</details>

---

[← Previous: Two Numbers Appearing Once](03-two-numbers-appearing-once.md) | [Next: Find Duplicate Number →](05-find-duplicate-number.md)
