# Chapter 2: Fast Fourier Transform (FFT) Concept

## Overview

The **Fast Fourier Transform** is one of the most important algorithms in computer science. It multiplies two degree-n polynomials in O(n log n) time instead of O(n²). FFT is a quintessential D&C algorithm — it splits a polynomial into even- and odd-indexed coefficients, evaluates recursively, then combines using the symmetry of complex roots of unity.

---

## The Polynomial Multiplication Problem

```
Given two polynomials:
    A(x) = a₀ + a₁x + a₂x² + ... + aₙ₋₁xⁿ⁻¹
    B(x) = b₀ + b₁x + b₂x² + ... + bₙ₋₁xⁿ⁻¹

Compute:
    C(x) = A(x) · B(x)
    C(x) = c₀ + c₁x + c₂x² + ... + c₂ₙ₋₂x²ⁿ⁻²

where cₖ = Σ aᵢ · bₖ₋ᵢ  (convolution)

Example:
    A(x) = 1 + 2x + 3x²
    B(x) = 4 + 5x
    
    C(x) = 4 + 5x + 8x + 10x² + 12x² + 15x³
         = 4 + 13x + 22x² + 15x³
         
Naive: O(n²) — multiply each aᵢ by each bⱼ
FFT:   O(n log n)
```

---

## Key Insight: Coefficient vs Point-Value Representation

```
COEFFICIENT FORM:
    A(x) = [a₀, a₁, ..., aₙ₋₁]  ← We start here
    Multiplication: O(n²)

POINT-VALUE FORM:
    A = {(x₀, A(x₀)), (x₁, A(x₁)), ..., (x₂ₙ₋₁, A(x₂ₙ₋₁))}
    Multiplication: O(n)  — just multiply corresponding values!
    C(xᵢ) = A(xᵢ) · B(xᵢ)

┌─────────────────────────────────────────────────────┐
│                                                     │
│  Coefficients ──[Evaluate O(n²)]──→ Point-Values    │
│       A, B                              A, B        │
│                                          │          │
│                                    Multiply O(n)    │
│                                          │          │
│                                          ▼          │
│  Coefficients ←──[Interpolate O(n²)]── Point-Values │
│       C                                  C          │
│                                                     │
│  Problem: Evaluate & Interpolate are both O(n²)!    │
│  FFT makes both O(n log n)!                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Roots of Unity: The Magic Choice of Points

```
ω = e^(2πi/n) = nth root of unity

The n-th roots of unity:
    ω⁰ = 1, ω¹, ω², ..., ωⁿ⁻¹

They form a circle in the complex plane:
                Im
                 │
            ω¹   │
          ·      │
        ·   ·    │
       ·     ·   │
    ──·───────·──── Re
  ω²  ·     ·  ω⁰ = 1
       ·   ·
        · ·
         ω³

For n=4: ω = i (the imaginary unit)
    ω⁰ = 1, ω¹ = i, ω² = -1, ω³ = -i

KEY PROPERTIES:
    1. Cancellation:  ω^(n/2) = -1
    2. Halving:       (ωⁿₖ)² = ω^(n/2)ₖ  (squaring halves the set)
    3. Summation:     Σ ωᵏ = 0 for k=0..n-1
```

---

## The FFT Algorithm (Cooley-Tukey)

```
IDEA: Evaluate A(x) at all nth roots of unity using D&C

Split polynomial by even and odd coefficients:
    A(x) = (a₀ + a₂x² + a₄x⁴ + ...) + x(a₁ + a₃x² + a₅x⁴ + ...)
         = A_even(x²) + x · A_odd(x²)

When x = ωₙᵏ (kth root of unity):
    A(ωₙᵏ) = A_even(ωₙ²ᵏ) + ωₙᵏ · A_odd(ωₙ²ᵏ)
            = A_even(ω_{n/2}ᵏ) + ωₙᵏ · A_odd(ω_{n/2}ᵏ)

For k + n/2 (using cancellation: ωₙ^(k+n/2) = -ωₙᵏ):
    A(ωₙ^(k+n/2)) = A_even(ω_{n/2}ᵏ) - ωₙᵏ · A_odd(ω_{n/2}ᵏ)

┌──────────────────────────────────────────────────────┐
│                                                      │
│  BUTTERFLY OPERATION:                                │
│                                                      │
│  For each k = 0, 1, ..., n/2 - 1:                   │
│    y_k     = E_k + ωₙᵏ · O_k                        │
│    y_{k+n/2} = E_k - ωₙᵏ · O_k                      │
│                                                      │
│  where E_k = A_even evaluated at ω_{n/2}^k          │
│        O_k = A_odd  evaluated at ω_{n/2}^k          │
│                                                      │
│  Same E_k and O_k used for TWO outputs! → D&C works │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Visual Butterfly Diagram (n=8)

```
Input:  a₀  a₁  a₂  a₃  a₄  a₅  a₆  a₇

After bit-reversal reorder:
        a₀  a₄  a₂  a₆  a₁  a₅  a₃  a₇

Stage 1 (size-2 DFTs):
        ╭───╮   ╭───╮   ╭───╮   ╭───╮
  a₀ ───┤ + ├── y₀  a₁ ──┤ + ├── y₄  ...
  a₄ ──×┤ - ├── y₁  a₅ ─×┤ - ├── y₅  ...
        ╰───╯   ╰───╯   ╰───╯   ╰───╯
        ω⁰₂         ω⁰₂         

Stage 2 (size-4 DFTs):
        ╭─────╮         ╭─────╮
  y₀ ───┤     ├── z₀    y₄ ──┤     ├── z₄
  y₁ ───┤     ├── z₁    y₅ ──┤     ├── z₅
  y₂ ──×┤     ├── z₂    y₆ ─×┤     ├── z₆
  y₃ ──×┤     ├── z₃    y₇ ─×┤     ├── z₇
        ╰─────╯         ╰─────╯

Stage 3 (size-8 DFT):
  Final combination of all 8 values
  using twiddle factors ω⁰₈, ω¹₈, ω²₈, ω³₈
```

---

## Pseudocode

```
FFT(a, n):
    // a = coefficient array, n = length (power of 2)
    if n == 1:
        return a
    
    ω_n ← e^(2πi/n)       // primitive nth root of unity
    ω ← 1                  // current twiddle factor
    
    a_even ← [a₀, a₂, a₄, ...]    // even-indexed coefficients
    a_odd  ← [a₁, a₃, a₅, ...]    // odd-indexed coefficients
    
    y_even ← FFT(a_even, n/2)
    y_odd  ← FFT(a_odd, n/2)
    
    y ← array of size n
    for k ← 0 to n/2 - 1:
        y[k]       ← y_even[k] + ω · y_odd[k]
        y[k + n/2] ← y_even[k] - ω · y_odd[k]
        ω ← ω · ω_n
    
    return y


INVERSE_FFT(y, n):
    // Same as FFT but use ω = e^(-2πi/n) and divide by n
    result ← FFT_with_inverse_roots(y, n)
    return result / n


POLYNOMIAL_MULTIPLY(A, B):
    n ← next power of 2 ≥ len(A) + len(B) - 1
    
    // Pad to length n
    pad A and B with zeros to length n
    
    // Step 1: Evaluate (FFT)
    FA ← FFT(A, n)
    FB ← FFT(B, n)
    
    // Step 2: Pointwise multiply O(n)
    FC ← [FA[i] · FB[i] for i in 0..n-1]
    
    // Step 3: Interpolate (Inverse FFT)
    C ← INVERSE_FFT(FC, n)
    
    return C
```

---

## Step-by-Step Trace (n=4)

```
Multiply A(x) = 1 + 2x, B(x) = 3 + 4x
Expected: C(x) = 3 + 10x + 8x²

Pad to n=4: A = [1, 2, 0, 0], B = [3, 4, 0, 0]

ω₄ = i (fourth root of unity)
Evaluate A at {1, i, -1, -i}:

FFT([1, 2, 0, 0], 4):
    a_even = [1, 0]
    a_odd  = [2, 0]
    
    FFT([1, 0], 2):
        a_even = [1], a_odd = [0]
        y[0] = 1 + 1·0 = 1
        y[1] = 1 - 1·0 = 1
        return [1, 1]
    
    FFT([2, 0], 2):
        return [2, 2]  (same structure)
    
    Combine with ω₄⁰=1, ω₄¹=i:
        y[0] = 1 + 1·2 = 3      → A(1)  = 1+2 = 3 ✓
        y[1] = 1 + i·2 = 1+2i   → A(i)  = 1+2i ✓
        y[2] = 1 - 1·2 = -1     → A(-1) = 1-2 = -1 ✓
        y[3] = 1 - i·2 = 1-2i   → A(-i) = 1-2i ✓

FA = [3, 1+2i, -1, 1-2i]

Similarly FFT([3, 4, 0, 0]) = [7, 3+4i, -1, 3-4i]

Pointwise multiply:
    FC[0] = 3 × 7         = 21
    FC[1] = (1+2i)(3+4i)  = 3+4i+6i+8i² = -5+10i
    FC[2] = (-1)(-1)      = 1
    FC[3] = (1-2i)(3-4i)  = 3-4i-6i+8i² = -5-10i

FC = [21, -5+10i, 1, -5-10i]

Inverse FFT (divide by n=4):
    C = [21+1+(-5-5), (21×1+(-5+10i)×(-i)+1×(-1)+(-5-10i)×i)] / 4
    
    Simpler: IFFT gives [3, 10, 8, 0]
    → c₀=3, c₁=10, c₂=8, c₃=0
    
    C(x) = 3 + 10x + 8x² ✓
```

---

## Complexity Analysis

```
RECURRENCE:
    T(n) = 2T(n/2) + O(n)
    
    Master Theorem: a=2, b=2, f(n)=O(n)
    log_b a = 1, f(n) = Θ(n^1) → Case 2
    
    T(n) = Θ(n log n)

POLYNOMIAL MULTIPLICATION:
    FFT(A):       O(n log n)
    FFT(B):       O(n log n)
    Pointwise:    O(n)
    Inverse FFT:  O(n log n)
    ─────────────────────────
    Total:        O(n log n)

Improvement:
    Naive polynomial multiply: O(n²)
    FFT-based:                 O(n log n)
```

---

## Number-Theoretic Transform (NTT)

```
Problem with FFT: Floating-point rounding errors!

Solution: NTT — FFT over finite fields

Instead of ω = e^(2πi/n), use a primitive root modulo a prime p.

Common NTT primes:
    p = 998244353 = 119 × 2²³ + 1
    Primitive root g = 3
    ω = g^((p-1)/n) mod p

Advantages:
    ✓ Exact integer arithmetic
    ✓ Same O(n log n) complexity
    ✓ Used in competitive programming
    
Disadvantages:
    ✗ Only works modulo prime
    ✗ Need CRT for general integer result
```

---

## Applications

```
┌────────────────────────────────────────────────────┐
│                                                    │
│  1. POLYNOMIAL MULTIPLICATION                      │
│     Direct application → O(n log n)                │
│                                                    │
│  2. BIG INTEGER MULTIPLICATION                     │
│     Treat digits as polynomial coefficients        │
│     Multiply, then propagate carries               │
│     → O(n log n) via Schönhage-Strassen            │
│                                                    │
│  3. STRING MATCHING                                │
│     Pattern matching with wildcards                 │
│                                                    │
│  4. SIGNAL PROCESSING                              │
│     Audio, image, radar, communications            │
│     Convolution = polynomial multiplication        │
│                                                    │
│  5. COMPETITIVE PROGRAMMING                        │
│     Counting problems via generating functions     │
│     Convolution of sequences                       │
│                                                    │
│  6. SCIENTIFIC COMPUTING                           │
│     PDE solvers, spectral methods                  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Python Implementation

```python
import cmath

def fft(a):
    """Compute FFT of coefficient array a (length must be power of 2)."""
    n = len(a)
    if n == 1:
        return a
    
    # Split into even and odd
    even = fft(a[0::2])
    odd  = fft(a[1::2])
    
    # Combine
    result = [0] * n
    for k in range(n // 2):
        w = cmath.exp(2j * cmath.pi * k / n)   # twiddle factor
        result[k]         = even[k] + w * odd[k]
        result[k + n // 2] = even[k] - w * odd[k]
    
    return result

def ifft(a):
    """Inverse FFT."""
    n = len(a)
    # Conjugate, FFT, conjugate, divide by n
    conjugated = [x.conjugate() for x in a]
    result = fft(conjugated)
    return [x.conjugate() / n for x in result]

def poly_multiply(A, B):
    """Multiply two polynomials using FFT."""
    # Result length
    result_len = len(A) + len(B) - 1
    
    # Pad to next power of 2
    n = 1
    while n < result_len:
        n <<= 1
    
    A = A + [0] * (n - len(A))
    B = B + [0] * (n - len(B))
    
    # FFT → Pointwise multiply → IFFT
    FA = fft(A)
    FB = fft(B)
    FC = [fa * fb for fa, fb in zip(FA, FB)]
    C = ifft(FC)
    
    # Round to integers (handle floating-point)
    return [round(c.real) for c in C[:result_len]]

# Test
A = [1, 2]       # 1 + 2x
B = [3, 4]       # 3 + 4x
print(poly_multiply(A, B))  # [3, 10, 8] → 3 + 10x + 8x²

A = [1, 2, 3]    # 1 + 2x + 3x²
B = [4, 5]        # 4 + 5x
print(poly_multiply(A, B))  # [4, 13, 22, 15]
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Multiply two polynomials of degree n |
| Key idea | Evaluate at roots of unity → multiply → interpolate |
| D&C split | Even-indexed vs odd-indexed coefficients |
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Complexity | O(n log n) |
| Improvement | O(n²) → O(n log n) |
| Butterfly | y_k = E_k + ωᵏ·O_k,  y_{k+n/2} = E_k - ωᵏ·O_k |
| Inverse | Same FFT with conjugate roots, divide by n |
| NTT variant | FFT over finite fields for exact arithmetic |

---

## Quick Revision Questions

1. **Why can't we just multiply polynomials in coefficient form efficiently?**
2. **What are roots of unity and why are they the ideal evaluation points?**
3. **Explain the butterfly operation in one sentence.**
4. **How does inverse FFT relate to forward FFT?**
5. **What is NTT and when would you use it over FFT?**
6. **What is the connection between polynomial multiplication and integer multiplication?**

---

[← Previous: Karatsuba Multiplication](01-karatsuba-multiplication.md) | [Next: Convex Hull →](03-convex-hull.md)
