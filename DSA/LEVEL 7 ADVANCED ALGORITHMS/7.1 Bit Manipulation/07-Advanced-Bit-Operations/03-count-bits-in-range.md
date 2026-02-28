# Chapter 7.3: Count Total Set Bits in a Range [1..N]

> **Unit 7 — Advanced Bit Operations**
> Count the total number of 1-bits across all integers from 1 to N in O(log N) time.

---

## Overview

Given N, compute: `popcount(1) + popcount(2) + ... + popcount(N)`. The brute-force O(N log N) approach is too slow for large N. A mathematical approach achieves O(log N).

---

## Key Insight: Count Contribution of Each Bit Position

```
  ┌──────────────────────────────────────────────────────────┐
  │  For bit position k (0-indexed), count how many numbers  │
  │  from 1 to N have bit k set.                            │
  │                                                          │
  │  Total = Σ count_of_1s_at_position_k  for all k         │
  └──────────────────────────────────────────────────────────┘
```

### Pattern for bit k

```
  Bit k cycles with period 2^(k+1):
    2^k zeros, then 2^k ones, repeating...

  Bit 0:  0 1 0 1 0 1 0 1 0 1 0 1 ...    (period 2)
  Bit 1:  0 0 1 1 0 0 1 1 0 0 1 1 ...    (period 4)
  Bit 2:  0 0 0 0 1 1 1 1 0 0 0 0 ...    (period 8)

  For numbers 0 to N:
  n = 0: 000    bit0=0 bit1=0 bit2=0
  n = 1: 001    bit0=1 bit1=0 bit2=0
  n = 2: 010    bit0=0 bit1=1 bit2=0
  n = 3: 011    bit0=1 bit1=1 bit2=0
  n = 4: 100    bit0=0 bit1=0 bit2=1
  n = 5: 101    bit0=1 bit1=0 bit2=1
  n = 6: 110    bit0=0 bit1=1 bit2=1
  n = 7: 111    bit0=1 bit1=1 bit2=1
```

---

## Formula for Bit Position k

```
  For numbers 1 to N, count of numbers with bit k set:

  full_cycles = (N + 1) / (2^(k+1))
  ones_in_full = full_cycles × 2^k

  remainder = (N + 1) % (2^(k+1))
  ones_in_partial = MAX(0, remainder - 2^k)

  count_at_k = ones_in_full + ones_in_partial
```

### Simplified code

```
FUNCTION countSetBitsUpTo(N):
    total = 0
    FOR k = 0 TO 30:          // for 32-bit numbers
        cycle = 1 << (k + 1)  // 2^(k+1)
        half = 1 << k         // 2^k
        
        full = (N + 1) / cycle * half
        remainder = (N + 1) % cycle
        partial = MAX(0, remainder - half)
        
        total += full + partial
    RETURN total
```

---

## Step-by-Step Trace: N = 7

```
  Bit 0 (k=0): cycle=2, half=1
    full = 8/2 × 1 = 4
    remainder = 8%2 = 0, partial = max(0, 0-1) = 0
    count = 4

  Bit 1 (k=1): cycle=4, half=2
    full = 8/4 × 2 = 4
    remainder = 8%4 = 0, partial = max(0, 0-2) = 0
    count = 4

  Bit 2 (k=2): cycle=8, half=4
    full = 8/8 × 4 = 4
    remainder = 8%8 = 0, partial = max(0, 0-4) = 0
    count = 4

  Total = 4 + 4 + 4 = 12

  Verify: 1(1) + 1(2) + 2(3) + 1(4) + 2(5) + 2(6) + 3(7) = 12 ✓
```

### Trace: N = 5

```
  Bit 0: cycle=2, half=1
    full = 6/2×1 = 3, rem = 0, partial = 0 → 3

  Bit 1: cycle=4, half=2
    full = 6/4×2 = 2, rem = 6%4 = 2, partial = max(0,2-2) = 0 → 2

  Bit 2: cycle=8, half=4
    full = 6/8×4 = 0, rem = 6%8 = 6, partial = max(0,6-4) = 2 → 2

  Total = 3 + 2 + 2 = 7

  Verify: 1+1+2+1+2 = 7 ✓ (popcount of 1,2,3,4,5)
```

---

## Count Bits in Range [L, R]

```
  countBitsInRange(L, R) = countSetBitsUpTo(R) - countSetBitsUpTo(L - 1)
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Bit-by-bit formula | O(log N) | O(1) |
| Brute force | O(N × log N) | O(1) |

---

## Applications

- **Digital circuit design** — counting transitions
- **Information theory** — total information content
- **Competitive programming** — bit contribution technique generalizes to sum of XOR, AND, OR across pairs

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Approach | Count contribution of each bit position independently |
| Pattern | Bit k has period 2^(k+1): 2^k zeros then 2^k ones |
| Formula | full_cycles × 2^k + max(0, remainder − 2^k) |
| Range query | f(R) − f(L−1) |
| Time | O(log N) per query |

---

## Quick Revision Questions

1. **How many total set bits are in numbers 1 to 3?**
   <details><summary>Answer</summary>popcount(1)+popcount(2)+popcount(3) = 1+1+2 = 4.</details>

2. **Why do we iterate k from 0 to 30 for 32-bit numbers?**
   <details><summary>Answer</summary>Bit positions 0 through 30 cover positive 32-bit integers (bit 31 is the sign bit for signed integers). For unsigned, go to 31.</details>

3. **What is the total set bits from 1 to 15?**
   <details><summary>Answer</summary>Each bit position 0-3 contributes 8 set bits (by symmetry of 4-bit numbers). Total = 4 × 8 = 32.</details>

4. **How would you count set bits from 10 to 20?**
   <details><summary>Answer</summary>countSetBitsUpTo(20) − countSetBitsUpTo(9).</details>

5. **Can this technique count set bits at a specific position k across [1..N]?**
   <details><summary>Answer</summary>Yes, that's exactly what the formula computes per position. Just use the formula for one specific k without summing over all positions.</details>

---

[← Previous: Leftmost Set Bit Position](02-leftmost-set-bit-position.md) | [Next: Sum of XOR of All Subsets →](04-sum-of-xor-of-all-subsets.md)
