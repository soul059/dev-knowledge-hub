# Chapter 6.6: Iterating Over All Subsets of a Mask

> **Unit 6 — Subsets with Bits**
> Enumerate every submask of a given bitmask — a critical technique for bitmask DP transitions.

---

## Overview

Given a bitmask `mask`, iterate over all subsets of `mask` (all submasks). A submask `sub` of `mask` satisfies: `sub & mask == sub` — every bit in `sub` is also in `mask`.

---

## The Technique

```
FUNCTION iterateSubsets(mask):
    sub = mask
    DO:
        process(sub)
        sub = (sub - 1) & mask
    WHILE sub != mask     // wraps around when sub reaches -1 & mask = mask

// Alternative with explicit 0 handling:
    sub = mask
    WHILE sub > 0:
        process(sub)
        sub = (sub - 1) & mask
    process(0)    // don't forget the empty subset!
```

---

## How (sub - 1) & mask Works

```
  ┌──────────────────────────────────────────────────────────┐
  │  sub - 1:  Flips the rightmost set bit and sets all      │
  │            lower bits. This "decrements" the submask.    │
  │                                                          │
  │  & mask:   Clears any bits that aren't in the original   │
  │            mask, keeping only valid submask bits.         │
  └──────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace: mask = 1010

```
  mask = 1010 (elements {1, 3})
  
  Subsets of {1, 3}: {}, {1}, {3}, {1,3}

  sub = 1010:  process(1010) = {1,3}
    sub-1 = 1001,  &mask = 1001 & 1010 = 1000

  sub = 1000:  process(1000) = {3}
    sub-1 = 0111,  &mask = 0111 & 1010 = 0010

  sub = 0010:  process(0010) = {1}
    sub-1 = 0001,  &mask = 0001 & 1010 = 0000

  sub = 0000:  process(0000) = {}
    sub-1 = 1111,  &mask = 1111 & 1010 = 1010 = mask → STOP

  Output: {1,3}, {3}, {1}, {} ✓ (all 4 subsets)
```

---

## Trace: mask = 1101

```
  mask = 1101 (elements {0, 2, 3})
  
  Expected 2^3 = 8 subsets:

  sub = 1101 → {0,2,3}    next: 1100 & 1101 = 1100
  sub = 1100 → {2,3}      next: 1011 & 1101 = 1001
  sub = 1001 → {0,3}      next: 1000 & 1101 = 1000
  sub = 1000 → {3}        next: 0111 & 1101 = 0101
  sub = 0101 → {0,2}      next: 0100 & 1101 = 0100
  sub = 0100 → {2}        next: 0011 & 1101 = 0001
  sub = 0001 → {0}        next: 0000 & 1101 = 0000
  sub = 0000 → {}         next: 1111 & 1101 = 1101 = mask → STOP

  All 8 subsets enumerated ✓
```

---

## Complexity Analysis

### For a single mask

```
  A mask with k set bits has 2^k submasks.
  Time: O(2^k) per mask.
```

### Sum over all masks (crucial for DP!)

```
  ┌──────────────────────────────────────────────────────────┐
  │  SUM over all masks of (number of submasks):             │
  │                                                          │
  │  = Σ(k=0 to n) C(n,k) × 2^k                            │
  │  = (1 + 2)^n                (binomial theorem)           │
  │  = 3^n                                                   │
  │                                                          │
  │  Iterating submasks of ALL masks takes O(3^n) total!    │
  └──────────────────────────────────────────────────────────┘

  n = 15: 3^15 ≈ 14M        ← fast
  n = 20: 3^20 ≈ 3.5B       ← borderline
```

---

## Application: DP Transition over Submasks

```
  Common pattern: dp[mask] depends on splitting mask into
  two parts (submask and complement).

  dp[mask] = MIN over all submasks s of mask:
               f(s) + dp[mask ^ s]

  FOR mask = 0 TO (1<<n)-1:
      sub = mask
      WHILE sub > 0:
          dp[mask] = MIN(dp[mask], f(sub) + dp[mask ^ sub])
          sub = (sub - 1) & mask
```

### Example: Set Partition

Partition a set into groups where each group has a precomputed cost `groupCost[mask]`.

```
  dp[mask] = minimum total cost to partition elements in mask

  dp[0] = 0
  FOR mask = 1 TO (1<<n)-1:
      sub = mask
      WHILE sub > 0:
          dp[mask] = MIN(dp[mask], groupCost[sub] + dp[mask ^ sub])
          sub = (sub - 1) & mask
```

---

## Important: Non-Empty Submasks Only

```
  The loop `sub = (sub-1) & mask; while sub != mask` includes sub = 0.
  
  If you only want NON-EMPTY submasks:

  sub = mask
  WHILE sub > 0:
      process(sub)
      sub = (sub - 1) & mask
  // Skip sub = 0
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Decrement trick | sub = (sub - 1) & mask |
| Why & mask | Keeps only bits that exist in the original mask |
| Termination | sub wraps to mask (via -1 & mask) |
| Single mask cost | O(2^k) for k set bits |
| All masks total | O(3^n) by binomial theorem |
| Application | DP transitions that split mask into submask + complement |

---

## Quick Revision Questions

1. **How many submasks does mask = 0b1111 have?**
   <details><summary>Answer</summary>2^4 = 16 (including the empty set and mask itself).</details>

2. **What is the first non-trivial submask of 0b1110 after itself?**
   <details><summary>Answer</summary>(1110 - 1) & 1110 = 1101 & 1110 = 1100. Submask = {2, 3}.</details>

3. **Why is the total over all masks O(3^n) and not O(4^n)?**
   <details><summary>Answer</summary>Each element has 3 states: (a) not in mask, (b) in mask but not in submask, (c) in both mask and submask. That's 3 choices per element → 3^n total.</details>

4. **Can you iterate submasks of mask in increasing order?**
   <details><summary>Answer</summary>The (sub-1) & mask trick goes in decreasing order. For increasing order, you'd need to enumerate differently (e.g., store and sort, or use complement-based tricks). Decreasing is the natural direction.</details>

5. **When would iterating submasks be used in competitive programming?**
   <details><summary>Answer</summary>Set partitioning problems, Steiner tree DP, covering problems, and any DP where a state is split into two disjoint parts.</details>

---

[← Previous: DP with Bitmasks Preview](05-dp-with-bitmasks-preview.md) | [Next: Unit 7 — Rightmost Set Bit Position →](../07-Advanced-Bit-Operations/01-rightmost-set-bit-position.md)
