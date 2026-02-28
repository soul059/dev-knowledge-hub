# Chapter 3: Random Algorithms

[← Previous: Expected Value](02-expected-value.md) | [Back to README](../README.md) | [Next: Reservoir Sampling →](04-reservoir-sampling.md)

---

## Overview

Randomized algorithms use random choices to achieve good expected performance or simplify complex problems. They fall into two categories: **Las Vegas** (always correct, random runtime) and **Monte Carlo** (fast, possibly incorrect).

---

## 3.1 Las Vegas vs Monte Carlo

```
  ┌──────────────────────────────────────────────────────────┐
  │  Las Vegas Algorithms                                    │
  │  • Always gives CORRECT answer                          │
  │  • Running time is RANDOM                               │
  │  • Example: Randomized QuickSort                        │
  │                                                          │
  │  Monte Carlo Algorithms                                  │
  │  • Running time is FIXED/bounded                        │
  │  • Answer may be WRONG with small probability           │
  │  • Example: Miller-Rabin primality test                 │
  │                                                          │
  │  ┌──────────┬──────────────┬─────────────┐              │
  │  │          │ Correctness  │  Runtime     │              │
  │  ├──────────┼──────────────┼─────────────┤              │
  │  │ Las Vegas│ Always ✓     │ Random       │              │
  │  │ M. Carlo │ Probably ✓   │ Deterministic│              │
  │  └──────────┴──────────────┴─────────────┘              │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.2 Randomized QuickSort (Las Vegas)

```
  Choose pivot RANDOMLY instead of fixed position.
  
FUNCTION quickSort(A, lo, hi):
    IF lo >= hi: RETURN
    pivot_idx = random integer in [lo, hi]
    SWAP A[pivot_idx] with A[hi]
    p = partition(A, lo, hi)    // standard Lomuto/Hoare
    quickSort(A, lo, p - 1)
    quickSort(A, p + 1, hi)

  ┌──────────────────────────────────────────────────────────┐
  │  Why randomize?                                          │
  │  • Deterministic pivot (first/last): O(n²) worst case   │
  │    on sorted/reverse-sorted input                        │
  │  • Random pivot: E[time] = O(n log n) for ANY input     │
  │  • No adversary can construct a "bad" input!            │
  └──────────────────────────────────────────────────────────┘
  
  Expected comparisons: 2n ln n ≈ 1.39 n log₂ n
  
  Proof sketch: For elements i < j (sorted order),
  P(i and j are compared) = 2/(j-i+1)
  E[comparisons] = Σᵢ Σⱼ 2/(j-i+1) = 2n Hₙ ≈ 2n ln n
```

---

## 3.3 Randomized Selection (QuickSelect)

```
  Find k-th smallest element in expected O(n).

FUNCTION quickSelect(A, lo, hi, k):
    IF lo == hi: RETURN A[lo]
    pivot_idx = random integer in [lo, hi]
    SWAP A[pivot_idx] with A[hi]
    p = partition(A, lo, hi)
    IF k == p: RETURN A[p]
    IF k < p: RETURN quickSelect(A, lo, p-1, k)
    ELSE: RETURN quickSelect(A, p+1, hi, k)

  Expected time: O(n) — geometric series argument
  Worst case: O(n²) — but extremely unlikely with randomization
  
  T(n) = n + T(n/2) on average (lucky split half the time)
       = n + n/2 + n/4 + ... = 2n = O(n)
```

---

## 3.4 Monte Carlo: Miller-Rabin Primality

```
  Test if n is prime with k rounds.
  
  Key idea: If n is prime, then for random a:
  either aᵈ ≡ 1 (mod n)
  or aᵈ×²ⁱ ≡ -1 (mod n) for some i
  
  (where n-1 = d × 2ʳ, d is odd)
  
  P(mistake per round) ≤ 1/4
  P(mistake after k rounds) ≤ (1/4)ᵏ
  
  k=20: P(error) < 10⁻¹² — practically certain
  
  Time per round: O(log² n) — due to modular exponentiation
  Total: O(k log² n)
```

---

## 3.5 Randomized Hashing

```
  Problem: Adversary can construct inputs that all hash
  to the same bucket → O(n) per lookup.
  
  Solution: Choose hash function RANDOMLY from a family.
  
  Universal hashing:
  h(k) = ((a × k + b) mod p) mod m
  
  where a ∈ {1,...,p-1}, b ∈ {0,...,p-1} chosen randomly
  p is a prime > universe size, m is table size
  
  Property: P(h(x) = h(y)) ≤ 1/m for x ≠ y
  → Expected chain length = O(1) regardless of input!
```

---

## 3.6 Freivald's Algorithm (Matrix Multiplication Verification)

```
  Given A, B, C (n×n matrices), verify A×B = C?
  
  Naive: Compute A×B → O(n³)
  Freivald: O(n²) with randomization!
  
FUNCTION verify(A, B, C, n):
    r = random binary vector of size n
    // Compute A×(B×r) and C×r
    Br = B × r          // O(n²)
    ABr = A × Br         // O(n²)
    Cr = C × r           // O(n²)
    RETURN ABr == Cr

  If A×B = C: always returns TRUE
  If A×B ≠ C: returns FALSE with P ≥ 1/2
  
  Repeat k times: P(error) ≤ (1/2)ᵏ
  
  Total time: O(k × n²)
```

---

## 3.7 Applications Summary

```
  ┌────────────────────────────────────────────────────────────┐
  │  Algorithm          │ Type       │ Expected   │ Use Case   │
  ├─────────────────────┼────────────┼────────────┼────────────┤
  │  Randomized QSort   │ Las Vegas  │ O(n log n) │ Sorting    │
  │  QuickSelect        │ Las Vegas  │ O(n)       │ k-th elem  │
  │  Miller-Rabin       │ Monte Carlo│ O(k log²n) │ Primality  │
  │  Freivald's         │ Monte Carlo│ O(kn²)     │ Matrix chk │
  │  Random hashing     │ Las Vegas  │ O(1) amort │ Hash table │
  │  Karger's min-cut   │ Monte Carlo│ O(n² log n)│ Graph cut  │
  │  Treap / Skip List  │ Las Vegas  │ O(log n)   │ BST ops    │
  └─────────────────────┴────────────┴────────────┴────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Las Vegas | Always correct, random runtime |
| Monte Carlo | Bounded time, possibly wrong |
| Randomized QSort | E[O(n log n)] for all inputs |
| QuickSelect | E[O(n)] for k-th element |
| Error amplification | Repeat k times: error ≤ (1/c)ᵏ |
| Universal hashing | O(1) expected per operation |

---

## Quick Revision Questions

1. **Classify** randomized QuickSort and Miller-Rabin by type.
2. **Why does** randomized QuickSort avoid O(n²) on sorted input?
3. **What is** the expected number of comparisons in randomized QuickSort?
4. **How does** Freivald's algorithm verify matrix multiplication in O(n²)?
5. **How many** rounds of Miller-Rabin are needed for error < 10⁻⁶?
6. **Explain** the difference between expected time and worst-case time.

---

[← Previous: Expected Value](02-expected-value.md) | [Back to README](../README.md) | [Next: Reservoir Sampling →](04-reservoir-sampling.md)
