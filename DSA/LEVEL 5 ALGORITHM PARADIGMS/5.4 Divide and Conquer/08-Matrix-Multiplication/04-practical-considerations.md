# Chapter 4: Practical Considerations

## Overview

While Strassen's algorithm is asymptotically faster, practical matrix multiplication involves many trade-offs: numerical stability, cache behavior, memory overhead, and hardware-specific optimizations. This chapter covers the engineering side of matrix multiplication.

---

## When to Use Strassen in Practice

```
┌────────────────────────────────────────────────────────┐
│                Decision Guide                          │
│                                                        │
│  Matrix size n < 64?                                   │
│    → Use standard O(n³). Strassen overhead too high.   │
│                                                        │
│  64 ≤ n < 500?                                         │
│    → Use optimized standard (BLAS level 3).            │
│    → Strassen MAY help depending on hardware.          │
│                                                        │
│  n ≥ 500?                                              │
│    → Strassen likely wins.                             │
│    → Use hybrid: Strassen recursion → standard base.   │
│                                                        │
│  Need high numerical precision?                        │
│    → Prefer standard. Strassen has stability issues.   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Numerical Stability

```
Problem: Strassen uses SUBTRACTION extensively.

    M₃ = A₁₁ · (B₁₂ - B₂₂)     ← subtraction
    M₄ = A₂₂ · (B₂₁ - B₁₁)     ← subtraction
    M₆ = (A₂₁ - A₁₁) · (B₁₁ + B₁₂)   ← subtraction

Subtraction of similar values → cancellation error!

Example:
    B₁₂ = 1.00000001
    B₂₂ = 1.00000002
    B₁₂ - B₂₂ = -0.00000001   ← lost 8 digits of precision!

Standard multiplication: forward stable
    Error bound: |ΔC| ≤ n · ε · |A| · |B|

Strassen: less stable
    Error bound: |ΔC| ≤ n^(log₂ 12) · ε · |A| · |B|
                     ≈ n^3.585 · ε   (WORSE than n³ · ε)

┌────────────────────────┬───────────────────────┐
│ Algorithm              │ Stability             │
├────────────────────────┼───────────────────────┤
│ Standard               │ Forward stable        │
│ Strassen               │ Numerically weaker    │
│ Winograd variant       │ Slightly better       │
│ CW-type                │ Very unstable         │
└────────────────────────┴───────────────────────┘
```

---

## Cache Performance

```
Matrix size: n × n with n = 4096 (32 MB per matrix)
Cache: 8 MB L3

Standard (naive triple loop):
    ┌──────────────────────────────────────┐
    │  for i:                              │
    │    for j:                            │
    │      for k:                          │
    │        C[i][j] += A[i][k] * B[k][j] │
    │                                      │
    │  Row-major access pattern:            │
    │  A: stride 1 (good)                  │
    │  B: stride n (BAD — cache misses!)   │
    │                                      │
    │  Cache misses: O(n³/L) where L=line  │
    └──────────────────────────────────────┘

Standard (BLOCKED / tiled):
    ┌──────────────────────────────────────┐
    │  for i in blocks of B:               │
    │    for j in blocks of B:             │
    │      for k in blocks of B:           │
    │        multiply B×B sub-blocks        │
    │                                      │
    │  Block fits in cache → fewer misses  │
    │  Cache misses: O(n³/(L·√M))          │
    │  where M = cache size                │
    └──────────────────────────────────────┘

Strassen:
    ┌──────────────────────────────────────┐
    │  Recursive subdivision naturally     │
    │  creates cache-friendly blocks!      │
    │                                      │
    │  When subproblem fits in cache,      │
    │  all further work is cache-local.    │
    │                                      │
    │  Cache-oblivious algorithm!          │
    │  Cache misses: O(n^ω / (L·√M))      │
    └──────────────────────────────────────┘
```

---

## Memory Overhead

```
Standard: O(n²) — just the output matrix C
  In-place: can even compute C += A × B

Strassen: O(n² log n) — temporary matrices at each level

  Level 0: ~10 temporary (n/2)×(n/2) matrices → ~2.5 n² entries
  Level 1: ~10 temporary (n/4)×(n/4) matrices → ~0.6 n² entries
  Level 2: ~10 temporary (n/8)×(n/8) matrices → ~0.15 n² entries
  ...
  Total: ~2.5 n² + 0.6 n² + 0.15 n² + ... ≈ 3.3 n²

  With careful memory reuse: O(n²) extra space
  (Reuse temporary buffers across the 7 products)

Practical impact for n = 4096:
  Standard: 128 MB (A, B, C)
  Strassen: ~170 MB (with extra temporaries)
  Overhead: ~33% more memory
```

---

## BLAS: What Libraries Actually Do

```
BLAS (Basic Linear Algebra Subprograms) Level 3:
  Highly optimized matrix multiplication.

What makes BLAS fast:
  1. Cache-aware blocking (tiled multiplication)
  2. SIMD vectorization (SSE, AVX-512)
  3. Register blocking (compute small tiles in registers)
  4. Loop unrolling (reduce branch overhead)
  5. Prefetching (hide memory latency)
  6. Multi-threading (OpenMP / parallel)

┌───────────────────────────────┬────────────────────┐
│ Library                       │ GFlops (n=4096)    │
├───────────────────────────────┼────────────────────┤
│ Naive triple loop             │ ~1 GFlop           │
│ Blocked/tiled                 │ ~10 GFlops         │
│ OpenBLAS / MKL (standard)     │ ~100+ GFlops       │
│ OpenBLAS + Strassen           │ ~120+ GFlops       │
│ GPU (cuBLAS)                  │ ~10,000+ GFlops    │
└───────────────────────────────┴────────────────────┘

Peak theoretical (modern CPU):
  16 cores × 3 GHz × 32 FP ops/cycle ≈ 1500 GFlops
  Libraries achieve 70-90% of peak!
```

---

## Hybrid Strategy

```
PRACTICAL_MATMUL(A, B, n):
    if n ≤ CROSSOVER:
        return OPTIMIZED_STANDARD(A, B, n)    // BLAS-like
    else:
        return STRASSEN(A, B, n)               // D&C

Choosing CROSSOVER:
    Benchmark on target hardware!
    Typical values: 64 to 512

┌──────────────────┬──────────────────────────────┐
│ CROSSOVER = 32   │ Too many recursive calls,    │
│                  │ overhead dominates            │
├──────────────────┼──────────────────────────────┤
│ CROSSOVER = 128  │ Good balance                 │
├──────────────────┼──────────────────────────────┤
│ CROSSOVER = 512  │ Strassen used only for       │
│                  │ very large matrices           │
└──────────────────┴──────────────────────────────┘
```

---

## Special Matrix Types

```
Not all matrices benefit from Strassen:

┌──────────────────────┬─────────────────────────────────┐
│ Matrix Type          │ Best Approach                    │
├──────────────────────┼─────────────────────────────────┤
│ Dense, large         │ Strassen hybrid or BLAS          │
│ Sparse               │ Sparse algorithms (CSR/CSC)      │
│ Symmetric            │ Exploit symmetry, ~half work     │
│ Triangular           │ Back-substitution, O(n²) solve   │
│ Band matrix          │ O(n · b²) where b = bandwidth   │
│ Diagonal             │ O(n) — trivial                   │
│ Small (n < 100)      │ Standard with SIMD               │
│ Rectangular (m×n·n×p)│ Depends on m, n, p               │
└──────────────────────┴─────────────────────────────────┘
```

---

## Parallelism

```
Matrix multiplication is highly parallelizable:

Standard (triple loop):
  ┌──────────────────────────────────────┐
  │  Each C[i][j] is independent!        │
  │  Embarrassingly parallel             │
  │  Parallelize outer loop: O(n) tasks  │
  │  Speedup: near-linear with cores     │
  └──────────────────────────────────────┘

Strassen:
  ┌──────────────────────────────────────────────┐
  │  7 independent multiplications per level!    │
  │  Can run M₁-M₇ in parallel                  │
  │  Parallelism: 7-way at each level            │
  │  Total parallelism: 7^(log₂ n) = n^2.807    │
  │  Deeper recursion → more parallel tasks      │
  └──────────────────────────────────────────────┘

GPU multiplication:
  ┌──────────────────────────────────────┐
  │  Thousands of cores × SIMD           │
  │  Standard tiled algorithm preferred  │
  │  CuBLAS: highly tuned for GPU arch   │
  │  Tensor cores: specialized hardware  │
  │  Mixed precision (FP16): even faster │
  └──────────────────────────────────────┘
```

---

## When NOT to Use Strassen

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  1. Small matrices (n < 100-500)                        │
│     → Overhead exceeds savings                          │
│                                                         │
│  2. Numerical precision is critical                     │
│     → Scientific computing, control systems             │
│     → Use standard with error analysis                  │
│                                                         │
│  3. Sparse matrices                                     │
│     → Strassen treats zeros as non-zeros                │
│     → Use sparse multiplication algorithms              │
│                                                         │
│  4. Memory-constrained environments                     │
│     → Strassen needs extra O(n²) space                  │
│     → Embedded systems, microcontrollers                │
│                                                         │
│  5. GPU computation                                     │
│     → Standard tiled is better suited to GPU arch       │
│     → Tensor cores handle standard multiply natively    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Factor | Standard | Strassen |
|--------|----------|----------|
| Time (asymptotic) | O(n³) | O(n^2.807) |
| Constant factor | Small | ~25× |
| Numerical stability | Excellent | Weaker |
| Cache behavior | Needs blocking | Naturally recursive |
| Memory overhead | O(n²) | O(n² log n) |
| Parallelism | Easy | Easy (7-way) |
| Implementation | Simple | Complex |
| Practical crossover | — | n ≈ 200-500 |
| Sparse matrices | N/A (use sparse) | Bad |

---

## Quick Revision Questions

1. **Why is Strassen less numerically stable?**
2. **What is the memory overhead of Strassen?**
3. **Why are BLAS libraries so fast for standard multiplication?**
4. **What is a typical crossover point for hybrid approaches?**
5. **Why is Strassen bad for sparse matrices?**
6. **How does Strassen's recursion help with cache performance?**

---

[← Previous: Complexity Improvement](03-complexity-improvement.md) | [Next: Matrix Exponentiation →](05-matrix-exponentiation.md)
