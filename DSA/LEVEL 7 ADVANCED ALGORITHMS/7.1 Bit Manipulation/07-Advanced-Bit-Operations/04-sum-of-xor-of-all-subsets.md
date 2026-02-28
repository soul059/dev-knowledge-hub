# Chapter 7.4: Sum of XOR of All Subsets

> **Unit 7 — Advanced Bit Operations**
> Compute the sum of XOR values of all subsets of an array — using bit contribution analysis.

---

## Overview

Given an array of n elements, find the sum of XOR of all 2^n subsets. This classic problem is solved in O(n) using the **bit contribution technique**.

---

## Problem Statement

```
  arr = [1, 5, 6]
  
  Subsets and their XOR:
  {}      → XOR = 0
  {1}     → XOR = 1
  {5}     → XOR = 5
  {6}     → XOR = 6
  {1,5}   → XOR = 1^5 = 4
  {1,6}   → XOR = 1^6 = 7
  {5,6}   → XOR = 5^6 = 3
  {1,5,6} → XOR = 1^5^6 = 2

  Sum = 0 + 1 + 5 + 6 + 4 + 7 + 3 + 2 = 28
```

---

## The Bit Contribution Technique

```
  ┌───────────────────────────────────────────────────────────┐
  │  Key Insight: Analyze each bit position independently.    │
  │                                                           │
  │  For bit position k:                                      │
  │    If AT LEAST ONE element has bit k set, then bit k      │
  │    contributes 2^k to exactly HALF of all subsets'        │
  │    XOR values. That is 2^(n-1) subsets.                   │
  │                                                           │
  │  Contribution of bit k = 2^k × 2^(n-1) = 2^(k+n-1)     │
  └───────────────────────────────────────────────────────────┘
```

### Why exactly half?

```
  Suppose c elements have bit k set (c ≥ 1).
  A subset's XOR has bit k set iff it contains an ODD number
  of these c elements.
  
  By combinatorics: exactly half of all subsets of c elements
  have odd-sized intersection → exactly 2^(n-1) subsets total
  have bit k set in their XOR.
```

---

## Algorithm

```
FUNCTION sumOfXOROfSubsets(arr[], n):
    // OR all elements to find which bits appear at least once
    orAll = 0
    FOR each x in arr:
        orAll = orAll | x
    
    // Each set bit contributes 2^k × 2^(n-1) to the sum
    RETURN orAll × (1 << (n - 1))
```

### That's it! Just OR + multiply!

---

## Step-by-Step Trace: arr = [1, 5, 6]

```
  n = 3

  Step 1: OR all elements
    1 | 5 | 6 = 001 | 101 | 110 = 111 = 7

  Step 2: Multiply by 2^(n-1) = 2^2 = 4
    7 × 4 = 28

  Answer: 28  ✓ (matches brute force above)
```

---

## Proof by Bit Position

```
  arr = [1, 5, 6]  = [001, 101, 110]

  Bit 0: present in {1, 5} (at least one) → contributes 2^0 × 2^2 = 4
  Bit 1: present in {6}    (at least one) → contributes 2^1 × 2^2 = 8
  Bit 2: present in {5, 6} (at least one) → contributes 2^2 × 2^2 = 16

  Total = 4 + 8 + 16 = 28  ✓

  Note: OR = 111 = 7, and 4 + 8 + 16 = 7 × 4 = 28
```

---

## Edge Cases

| Case | OR | Result |
|------|----|--------|
| All zeros [0,0,0] | 0 | 0 |
| Single element [x] | x | x (only subsets: {} with XOR=0, {x} with XOR=x → sum = x = x × 2^0) |
| All same [3,3,3] | 3 | 3 × 2^2 = 12 |

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Bit contribution | **O(n)** — just compute OR | O(1) |
| Brute force | O(n × 2^n) | O(1) |

---

## Generalization: Sum of XOR of All Pairs

```
  Different problem: Σ (arr[i] ^ arr[j]) for all i < j
  
  For each bit k: count how many elements have bit k SET (call it c).
  Pairs with different bit k: c × (n - c)
  Contribution = c × (n - c) × 2^k
  
  Total = Σ_k c_k × (n - c_k) × 2^k
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core trick | OR all elements, multiply by 2^(n-1) |
| Why OR | Identifies which bit positions are present |
| Why 2^(n-1) | Each present bit appears in exactly half of subsets' XORs |
| Time | O(n) to compute OR |
| Generalization | Sum of pair XOR uses count-per-bit × complement × 2^k |

---

## Quick Revision Questions

1. **What is the sum of XOR of all subsets of [3, 5]?**
   <details><summary>Answer</summary>OR = 3|5 = 7. Result = 7 × 2^1 = 14. Verify: 0+3+5+(3^5=6) = 14 ✓.</details>

2. **Why does the formula fail if n = 0 (empty array)?**
   <details><summary>Answer</summary>2^(n-1) = 2^(-1) = 0.5, which doesn't make integer sense. Empty array has one subset ({}) with XOR = 0. Handle as special case.</details>

3. **If all elements are the same value v, what's the answer?**
   <details><summary>Answer</summary>OR = v. Result = v × 2^(n-1). This is because subsets with odd count of v have XOR = v, even count have XOR = 0, and there are 2^(n-1) of each.</details>

4. **How does the "bit contribution" technique differ for Sum of AND of all subsets?**
   <details><summary>Answer</summary>For AND: bit k is set in subset's AND only if ALL elements in the subset have bit k set. If c elements have bit k, then 2^c - 1 non-empty subsets have it. Contribution = (2^c - 1) × 2^k.</details>

5. **What is the sum of XOR of all subsets of [7]?**
   <details><summary>Answer</summary>OR = 7. Result = 7 × 2^0 = 7. Subsets: {}: XOR=0, {7}: XOR=7. Sum = 7 ✓.</details>

---

[← Previous: Count Bits in Range](03-count-bits-in-range.md) | [Next: Maximum AND Pair →](05-maximum-and-pair.md)
