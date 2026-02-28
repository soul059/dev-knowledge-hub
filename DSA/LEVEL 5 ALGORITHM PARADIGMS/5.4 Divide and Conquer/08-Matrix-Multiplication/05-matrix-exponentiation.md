# Chapter 5: Matrix Exponentiation

## Overview

Matrix Exponentiation computes $A^k$ (matrix A raised to power k) in O(n³ log k) time using **exponentiation by squaring** — a D&C technique. This is a powerful tool for solving linear recurrences (like Fibonacci) in O(log n) time.

---

## The Problem

```
Given: n×n matrix A, positive integer k
Compute: A^k = A × A × A × ... × A  (k times)

Naive: k-1 matrix multiplications → O(k · n³)

Can we do better? YES — using D&C!
```

---

## Exponentiation by Squaring

```
KEY IDEA (same as fast power for integers):

    A^k = ┌ A^(k/2) × A^(k/2)           if k is even
          │
          └ A^((k-1)/2) × A^((k-1)/2) × A   if k is odd

Example: A^13
    A^13 = A^6 × A^6 × A
    A^6  = A^3 × A^3
    A^3  = A^1 × A^1 × A
    A^1  = A

    Only 4-5 multiplications instead of 12!
```

---

## Pseudocode

```
MATRIX_POWER(A, k):
    if k == 1:
        return A
    if k == 0:
        return I  (identity matrix)
    
    if k is even:
        half ← MATRIX_POWER(A, k/2)
        return half × half
    else:
        half ← MATRIX_POWER(A, (k-1)/2)
        return half × half × A

// Iterative version (binary method):
MATRIX_POWER_ITERATIVE(A, k):
    result ← I  (identity matrix)
    base ← A
    
    while k > 0:
        if k is odd:
            result ← result × base
        base ← base × base
        k ← k / 2   (integer division)
    
    return result
```

---

## Trace: A^13 (Iterative)

```
k = 13 = 1101₂

Step 1: k=13 (odd)  → result = I × A = A       base = A² 
Step 2: k=6  (even) → result = A                base = A⁴
Step 3: k=3  (odd)  → result = A × A⁴ = A⁵     base = A⁸
Step 4: k=1  (odd)  → result = A⁵ × A⁸ = A¹³   base = A¹⁶
k=0 → done!

Answer: A¹³
Matrix multiplications: 5 (instead of 12)
In general: O(log k) multiplications
```

---

## Complexity

```
Number of matrix multiplications: O(log k)
Each multiplication: O(n³) for standard, O(n^2.807) for Strassen

┌────────────────────────────┬──────────────┐
│ Method                     │ Time         │
├────────────────────────────┼──────────────┤
│ Naive (k multiplications)  │ O(k · n³)    │
│ D&C with standard mult     │ O(n³ log k)  │
│ D&C with Strassen          │ O(n^2.81 log k) │
└────────────────────────────┴──────────────┘

Space: O(n²) for the result and temporary matrices
```

---

## Application 1: Fibonacci in O(log n)

```
Fibonacci recurrence:
    F(n) = F(n-1) + F(n-2)
    F(0) = 0, F(1) = 1

Matrix form:
    ┌ F(n+1) ┐   ┌ 1  1 ┐ⁿ   ┌ F(1) ┐   ┌ 1  1 ┐ⁿ   ┌ 1 ┐
    │        │ = │      │   × │      │ = │      │   × │   │
    └ F(n)   ┘   └ 1  0 ┘    └ F(0) ┘   └ 1  0 ┘    └ 0 ┘

Verification:
    ┌ 1  1 ┐¹  ┌ 1 ┐   ┌ 1 ┐   → F(1)=1, F(0)=... ✓
    │      │ × │   │ = │   │
    └ 1  0 ┘   └ 0 ┘   └ 1 ┘

    ┌ 1  1 ┐²  ┌ 1 ┐   ┌ 2  1 ┐  ┌ 1 ┐   ┌ 2 ┐   → F(2)=1, F(1)=1 ✓
    │      │ × │   │ = │      │× │   │ = │   │
    └ 1  0 ┘   └ 0 ┘   └ 1  1 ┘  └ 0 ┘   └ 1 ┘

Algorithm:
    FIBONACCI(n):
        M ← [[1,1],[1,0]]
        result ← MATRIX_POWER(M, n)
        return result[1][0]   // F(n)
    
    Time: O(log n) × O(2³) = O(log n)
    (2×2 matrix multiplication is O(1))
```

---

## Application 2: General Linear Recurrences

```
Any linear recurrence of order d:
    f(n) = c₁f(n-1) + c₂f(n-2) + ... + c_d f(n-d)

Can be expressed as:
    ┌  f(n)  ┐   ┌ c₁ c₂ ... c_(d-1) c_d ┐ⁿ⁻ᵈ⁺¹   ┌ f(d-1) ┐
    │ f(n-1) │   │  1  0 ...    0      0  │         │ f(d-2) │
    │ f(n-2) │ = │  0  1 ...    0      0  │       × │ f(d-3) │
    │  ...   │   │  :  :       :      :  │         │  ...   │
    └f(n-d+1)┘   └  0  0 ...    1      0  ┘         └  f(0)  ┘

    d×d companion matrix raised to power (n-d+1)
    Time: O(d³ log n)
```

### Example: Tribonacci

```
T(n) = T(n-1) + T(n-2) + T(n-3)
T(0) = 0, T(1) = 0, T(2) = 1

Companion matrix:
    ┌ 1  1  1 ┐
    │ 1  0  0 │
    └ 0  1  0 ┘

Time: O(3³ log n) = O(27 log n) = O(log n)
```

---

## Application 3: Counting Paths in Graphs

```
Given adjacency matrix A of a graph:
    A^k[i][j] = number of paths of length k from i to j

Example:
    Graph:  0 ↔ 1 ↔ 2

    A = ┌ 0  1  0 ┐
        │ 1  0  1 │
        └ 0  1  0 ┘

    A² = ┌ 1  0  1 ┐    Paths of length 2:
         │ 0  2  0 │    0→1→0, 0→1→2, etc.
         └ 1  0  1 ┘

    A³ = ┌ 0  2  0 ┐    Paths of length 3:
         │ 2  0  2 │    0→1→0→1, 0→1→2→1, etc.
         └ 0  2  0 ┘

Applications:
  - Count walks of length k: A^k[i][j]
  - Shortest paths (with modification)
  - Reachability: (A + I)^n ≠ 0 → connected
```

---

## Application 4: Dynamic Programming Optimization

```
Some DP transitions can be expressed as matrix multiplication:

Example: Number of ways to tile a 2×n board

State: current column configuration
Transition matrix T encodes valid placements
Answer: T^n [start_state][end_state]

Time: O(s³ log n) where s = number of states
(Often s is small, making this very efficient)
```

---

## Implementation (Python)

```python
def matrix_mult(A, B, mod=None):
    """Multiply two matrices, optionally with modular arithmetic."""
    n = len(A)
    m = len(B[0])
    k = len(B)
    C = [[0] * m for _ in range(n)]
    for i in range(n):
        for j in range(m):
            for p in range(k):
                C[i][j] += A[i][p] * B[p][j]
            if mod:
                C[i][j] %= mod
    return C

def matrix_power(M, k, mod=None):
    """Compute M^k using exponentiation by squaring."""
    n = len(M)
    # Identity matrix
    result = [[1 if i == j else 0 for j in range(n)] for i in range(n)]
    base = [row[:] for row in M]  # Copy
    
    while k > 0:
        if k % 2 == 1:
            result = matrix_mult(result, base, mod)
        base = matrix_mult(base, base, mod)
        k //= 2
    
    return result

# Fibonacci
def fibonacci(n, mod=10**9+7):
    if n <= 1:
        return n
    M = [[1, 1], [1, 0]]
    result = matrix_power(M, n, mod)
    return result[1][0]

# Test
for i in range(10):
    print(f"F({i}) = {fibonacci(i)}")
# F(0)=0, F(1)=1, F(2)=1, F(3)=2, F(4)=3, F(5)=5, ...
```

---

## Important: Modular Arithmetic

```
For competitive programming, results are often mod 10⁹+7:

    matrix_mult with mod:
        C[i][j] = (C[i][j] + A[i][p] * B[p][j]) % mod

This prevents integer overflow and works correctly because:
    (a + b) mod m = ((a mod m) + (b mod m)) mod m
    (a × b) mod m = ((a mod m) × (b mod m)) mod m

WITHOUT mod: Fibonacci(10⁶) has ~200,000 digits!
WITH mod:    Fibonacci(10⁶) mod (10⁹+7) fits in 32 bits
```

---

## D&C Connection

```
Matrix exponentiation IS divide and conquer:

    ┌────────────────────────────────────────────┐
    │  DIVIDE:  Split exponent k → k/2          │
    │  CONQUER: Compute A^(k/2) recursively     │
    │  COMBINE: Square the result (× A if odd)  │
    │                                            │
    │  Recurrence: T(k) = T(k/2) + O(n³)        │
    │  Solution:   T(k) = O(n³ log k)           │
    │                                            │
    │  Note: D&C is on the EXPONENT, not matrix  │
    └────────────────────────────────────────────┘
```

---

## Summary Table

| Application | Matrix Size | Time |
|-------------|-------------|------|
| Fibonacci | 2×2 | O(log n) |
| k-th order recurrence | k×k | O(k³ log n) |
| Graph paths of length k | n×n | O(n³ log k) |
| DP optimization | s×s | O(s³ log n) |
| General matrix power | n×n | O(n³ log k) |

---

## Quick Revision Questions

1. **How does exponentiation by squaring work?**
2. **Write the matrix form for F(n) = F(n-1) + F(n-2).**
3. **What does A^k[i][j] represent for an adjacency matrix?**
4. **How many matrix multiplications for A^1000?**
5. **Why use modular arithmetic in competitive programming?**
6. **What is the companion matrix for a 3rd-order recurrence?**

---

[← Previous: Practical Considerations](04-practical-considerations.md) | [Back to README](../README.md)
