# Chapter 6: Applications of Fast Exponentiation

[← Previous: Fibonacci in O(log n)](05-fibonacci-in-log-n.md) | [Back to README](../README.md) | [Next: Unit 6 — Permutations →](../06-Combinatorics/01-permutations.md)

---

## Overview

Fast exponentiation unlocks solutions to a wide variety of problems beyond simple power computation. This chapter catalogs the most important applications across competitive programming.

---

## 6.1 Application Map

```
  ┌─────────────────────────────────────────────────────────┐
  │              FAST EXPONENTIATION APPLICATIONS            │
  │                                                         │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │   Modular    │  │   Matrix     │  │  Geometric   │  │
  │  │   Inverse    │  │   Recurrence │  │   Series     │  │
  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
  │         │                 │                  │          │
  │  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐  │
  │  │   nCr mod p  │  │  Fibonacci   │  │  Sum a^k     │  │
  │  │   Division   │  │  Tiling      │  │  Formulas    │  │
  │  │   RSA crypto │  │  Graph paths │  │              │  │
  │  └──────────────┘  └──────────────┘  └──────────────┘  │
  │                                                         │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │  Permutation │  │  Binary      │  │   Cycle      │  │
  │  │  Power       │  │  Lifting     │  │  Detection   │  │
  │  └──────────────┘  └──────────────┘  └──────────────┘  │
  └─────────────────────────────────────────────────────────┘
```

---

## 6.2 Modular Inverse (Recap)

```
  a⁻¹ mod p = modPow(a, p - 2, p)    [Fermat's theorem, p prime]

  Used everywhere:
  - Division under modulo
  - nCr mod p
  - Probability calculations mod p
  - Matrix inverse mod p
```

---

## 6.3 Geometric Series Sum

```
  S = 1 + a + a² + ... + a^(n-1)  mod M

  Formula: S = (a^n - 1) / (a - 1)
  Modular: S = (modPow(a, n, M) - 1) × modInverse(a - 1, M) % M

  ⚠️ Special case: a ≡ 1 (mod M) → S = n mod M
  
  Alternative (no division, using recursion):
  
  FUNCTION geoSum(a, n, M):
      IF n == 1: RETURN 1
      IF n is even:
          half = geoSum(a, n/2, M)
          RETURN half × (1 + modPow(a, n/2, M)) % M
      ELSE:
          RETURN (1 + a × geoSum(a, n-1, M)) % M
  
  Time: O(log² n)
```

---

## 6.4 Counting Graph Paths

```
  Number of paths of EXACTLY k edges from u to v in a graph:

  Answer = (adj_matrix^k)[u][v]

  Example:
  Graph: 0→1, 1→2, 0→2, 2→0

  Adj = ┌       ┐
        │ 0 1 1 │
        │ 0 0 1 │
        │ 1 0 0 │
        └       ┘

  Paths of length 2 = Adj²:
  ┌       ┐
  │ 1 0 1 │    paths 0→0 of length 2: 1 (0→2→0)
  │ 1 0 0 │    paths 0→2 of length 2: 1 (0→1→2)
  │ 0 1 1 │    paths 2→2 of length 2: 1 (2→0→2)
  └       ┘

  For k = 10¹⁸: use matrix fast power!
  Time: O(V³ × log k)
```

---

## 6.5 Permutation Power

```
  Apply a permutation π exactly n times.
  
  π = [2, 3, 1, 5, 4]  (1-indexed)
  Means: 1→2, 2→3, 3→1, 4→5, 5→4
  
  π² = π(π) = composition
  
  Using binary exponentiation on permutation composition:
  
  FUNCTION permuteOnce(arr, perm):
      n = len(arr)
      result = new array of size n
      FOR i = 0 TO n-1:
          result[perm[i]] = arr[i]
      RETURN result
  
  FUNCTION permPow(perm, n):
      // Binary exponentiation where "multiply" = compose
      result = identity permutation
      WHILE n > 0:
          IF n & 1:
              result = compose(result, perm)
          perm = compose(perm, perm)
          n >>= 1
      RETURN result

  Time: O(k × log n) where k = permutation size
```

---

## 6.6 Tiling Problems

```
  Problem: Fill a 3×n grid with 2×1 dominoes.
  
  This can be modeled as a linear recurrence on "column states".
  Each column has 2³ = 8 possible fill states.
  Transition between states → 8×8 transfer matrix.
  
  Answer = entry in (transfer_matrix)^n.
  
  Time: O(8³ × log n) = O(512 × log n)
  
  For 2×n: Fibonacci! F(n+1) tilings.
  For 3×n: Can be solved with 4×4 matrix.
  For k×n: 2^k × 2^k matrix → feasible for k ≤ 15.
```

---

## 6.7 Binary Lifting on Trees

```
  Problem: Find k-th ancestor of node v in a tree.
  
  Precompute:
  up[v][0] = parent(v)
  up[v][j] = up[up[v][j-1]][j-1]    // 2^j-th ancestor

  Query: k-th ancestor
  WHILE k > 0:
      IF k & 1:
          v = up[v][bit_position]
      k >>= 1, bit_position++

  This is binary exponentiation on the "go to parent" function!
  
  Time: O(log n) per query, O(n log n) precomputation
  Used in: LCA (Lowest Common Ancestor)
```

---

## 6.8 Modular Arithmetic in Counting Problems

```
  Common pattern in competitive programming:

  1. Compute n! mod p          → O(n) precomputation
  2. Compute inverse factorials → O(n) using modPow once
  3. Answer nCr queries in O(1)

  PRECOMPUTE:
  fact[0] = 1
  FOR i = 1 TO MAXN:
      fact[i] = fact[i-1] * i % MOD

  inv_fact[MAXN] = modPow(fact[MAXN], MOD - 2, MOD)
  FOR i = MAXN-1 DOWNTO 0:
      inv_fact[i] = inv_fact[i+1] * (i+1) % MOD

  nCr(n, r) = fact[n] * inv_fact[r] % MOD * inv_fact[n-r] % MOD
```

---

## 6.9 Solving Recurrences: Decision Guide

```
  Given a recurrence f(n) = ...

  ┌─── Is it linear with constant coefficients?
  │
  ├── YES ──┐
  │         ├── n ≤ 10⁷? → Use simple iteration O(n)
  │         └── n > 10⁷? → Use matrix exponentiation O(k³ log n)
  │
  └── NO
      ├── Depends on n? (e.g., f(n) = n×f(n-1))
      │   → Cannot use matrix exp. Use other techniques.
      │
      └── Non-linear? (e.g., f(n) = f(n-1)²)
          → Cannot use matrix exp. Problem-specific approach.
```

---

## 6.10 Performance Tips

```
  ┌──────────────────────────────────────────────────────────┐
  │  OPTIMIZATION TIPS:                                      │
  │                                                          │
  │  1. Use iterative modPow (O(1) space)                   │
  │  2. For 2×2 matrices, unroll the multiply               │
  │     (avoid generic matrix multiply overhead)             │
  │  3. Precompute inverse factorials in O(n) not O(n log n)│
  │  4. Use fast doubling for Fibonacci (3 mults/step)      │
  │  5. Batch modular inverses using prefix products         │
  │  6. Use __int128 in C++ when M ≈ 10¹⁸                   │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Application | Technique | Time |
|-------------|-----------|------|
| a^n mod M | Binary exponentiation | O(log n) |
| Modular inverse | a^(p-2) mod p | O(log p) |
| Fibonacci F(n) | Matrix / fast doubling | O(log n) |
| Linear recurrence | Matrix exponentiation | O(k³ log n) |
| Graph paths of length k | Adj matrix^k | O(V³ log k) |
| Permutation power | Compose via binary exp | O(k log n) |
| k-th ancestor | Binary lifting | O(log n) query |
| nCr mod p | Precompute + modPow | O(1) query |

---

## Quick Revision Questions

1. **List five different domains** where binary exponentiation is applied.
2. **How do you compute** sum 1 + a + a² + ... + a^(n-1) mod M?
3. **To count paths of length k** in a graph, what do you exponentiate?
4. **What is binary lifting** and how is it related to fast exponentiation?
5. **How do you precompute** inverse factorials in O(n) instead of O(n log n)?
6. **When can you NOT use** matrix exponentiation for a recurrence?

---

[← Previous: Fibonacci in O(log n)](05-fibonacci-in-log-n.md) | [Back to README](../README.md) | [Next: Unit 6 — Permutations →](../06-Combinatorics/01-permutations.md)
