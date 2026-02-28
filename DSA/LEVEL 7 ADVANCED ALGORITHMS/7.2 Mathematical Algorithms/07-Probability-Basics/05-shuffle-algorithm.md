# Chapter 5: Shuffle Algorithm

[← Previous: Reservoir Sampling](04-reservoir-sampling.md) | [Back to README](../README.md) | [Next: Probabilistic Data Structures →](06-probabilistic-data-structures.md)

---

## Overview

A perfect shuffle produces each permutation with equal probability $1/n!$. The **Fisher-Yates (Knuth) shuffle** is the gold-standard O(n) algorithm. This chapter covers correct shuffling, common mistakes, and bias analysis.

---

## 5.1 Fisher-Yates Shuffle (Knuth Shuffle)

```
FUNCTION shuffle(A, n):
    FOR i = n - 1 DOWNTO 1:
        j = random integer in [0, i]   // inclusive
        SWAP A[i] with A[j]

  Alternatively (forward version):
  FOR i = 0 TO n - 2:
      j = random integer in [i, n-1]
      SWAP A[i] with A[j]

Time:  O(n)
Space: O(1) in-place
```

---

## 5.2 Why It's Uniform

```
  Proof: Count the number of possible execution paths.
  
  Step n-1: choose from n positions    → n choices
  Step n-2: choose from n-1 positions  → n-1 choices
  ...
  Step 1:   choose from 2 positions    → 2 choices
  
  Total paths = n × (n-1) × ... × 2 = n!
  
  Each path produces a DISTINCT permutation.
  Each path has probability 1/n!
  → Every permutation equally likely! □

  ┌──────────────────────────────────────────────────┐
  │  Visual for n = 3, array [A, B, C]:             │
  │                                                  │
  │  i=2: j∈{0,1,2}  →  3 branches                 │
  │  i=1: j∈{0,1}    →  2 branches each             │
  │                                                  │
  │  Total leaves = 3 × 2 = 6 = 3!                  │
  │  Each leaf: one unique permutation               │
  └──────────────────────────────────────────────────┘
```

---

## 5.3 Step-by-Step Trace

```
  Array: [1, 2, 3, 4]   (n = 4)
  
  i=3: j = random(0..3), say j=1
       SWAP A[3] ↔ A[1]  →  [1, 4, 3, 2]
  
  i=2: j = random(0..2), say j=0
       SWAP A[2] ↔ A[0]  →  [3, 4, 1, 2]
  
  i=1: j = random(0..1), say j=1
       SWAP A[1] ↔ A[1]  →  [3, 4, 1, 2]   (no-op)
  
  Result: [3, 4, 1, 2]
  This is one of 4! = 24 equally likely outcomes.
```

---

## 5.4 Common WRONG Shuffles

### Mistake 1: Swap with ANY Position (Naive Shuffle)

```
  WRONG:
  FOR i = 0 TO n - 1:
      j = random(0, n-1)
      SWAP A[i] with A[j]

  Total paths = nⁿ (n choices at each of n steps)
  But nⁿ is generally NOT divisible by n!
  → Some permutations are MORE LIKELY than others!
  
  Example: n=3
  Total paths = 3³ = 27
  Permutations = 3! = 6
  27 / 6 = 4.5 — not integer!
  
  So it's impossible for all 6 permutations to have equal probability.
  
  ┌─────────────────────────────────────────────────┐
  │  For n=3, empirical distribution:               │
  │  [1,2,3] → 4/27    [1,3,2] → 5/27             │
  │  [2,1,3] → 5/27    [2,3,1] → 5/27             │
  │  [3,1,2] → 4/27    [3,2,1] → 4/27             │
  │                                                  │
  │  ⚠️ NOT UNIFORM! Bias up to 25%                 │
  └─────────────────────────────────────────────────┘
```

### Mistake 2: Sattolo's Algorithm (Derangement Only)

```
  FOR i = n - 1 DOWNTO 1:
      j = random(0, i - 1)    // ← excludes i itself!
      SWAP A[i] with A[j]

  This produces a random CYCLIC permutation (derangement).
  Every element moves — no fixed points.
  Not the same as a uniform shuffle!
  
  Total paths = (n-1)! = number of cyclic permutations ✓
```

---

## 5.5 Verifying Shuffle Correctness

```
  Test: Run shuffle 10⁶ times, count how often each element
        lands in each position. Should be ≈ 10⁶/n for all.
        
  ┌──────────────────────────────────────────────────────┐
  │  Position →  0      1      2      3                  │
  │  Element 1:  25.0%  25.0%  25.0%  25.0%  ← good     │
  │  Element 2:  25.0%  25.0%  25.0%  25.0%  ← good     │
  │  Element 3:  25.0%  25.0%  25.0%  25.0%  ← good     │
  │  Element 4:  25.0%  25.0%  25.0%  25.0%  ← good     │
  └──────────────────────────────────────────────────────┘
  
  Chi-squared test to validate statistical uniformity.
```

---

## 5.6 Partial Shuffle (Choose k Random Elements)

```
  Only need k random elements from array of n?
  
FUNCTION partialShuffle(A, n, k):
    FOR i = 0 TO k - 1:
        j = random(i, n - 1)
        SWAP A[i] with A[j]
    RETURN A[0..k-1]

  Time:  O(k) — NOT O(n)!
  Space: O(1)
  
  First k positions hold k uniformly random elements.
  This is optimal when k << n.
```

---

## 5.7 Online / Streaming Shuffle

```
  Can't hold all elements? Combine with reservoir sampling.
  
  1. Read stream into reservoir of size n
  2. Shuffle the reservoir using Fisher-Yates
  
  Or: use reservoir sampling directly (already random order).
```

---

## Summary Table

| Algorithm | Uniform? | Total Paths | Time | Space |
|-----------|----------|-------------|------|-------|
| Fisher-Yates | Yes ✓ | n! | O(n) | O(1) |
| Naive (swap any) | No ✗ | nⁿ | O(n) | O(1) |
| Sattolo | Cyclic only | (n-1)! | O(n) | O(1) |
| Partial shuffle | Yes (k items) | — | O(k) | O(1) |

---

## Quick Revision Questions

1. **Why does** the naive shuffle produce biased results for n=3?
2. **Trace** Fisher-Yates on [A, B, C, D, E] with specific random choices.
3. **How does** Sattolo's algorithm differ from Fisher-Yates?
4. **What is** the time complexity to choose 5 random elements from 10⁶?
5. **Prove** that Fisher-Yates produces each permutation with P = 1/n!.
6. **Design** a test to detect shuffle bias empirically.

---

[← Previous: Reservoir Sampling](04-reservoir-sampling.md) | [Back to README](../README.md) | [Next: Probabilistic Data Structures →](06-probabilistic-data-structures.md)
